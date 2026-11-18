# DashCast

Projeto em fase inicial, desenvolvido com Python e Django. O ambiente e as
dependências são gerenciados com [uv](https://docs.astral.sh/uv/).

## Preparação do ambiente

Pré-requisitos: Git, Python 3.14 e uv instalado. Execute os comandos abaixo na
raiz do repositório.

Instale as dependências nas versões registradas em `uv.lock`:

```bash
uv sync --locked
```

Para preparar o banco de dados local e iniciar o servidor de desenvolvimento:

```bash
uv run manage.py migrate
uv run manage.py runserver
```

## Visão geral da esteira

A validação acontece localmente, nos hooks do Git, e no GitHub Actions, em pushes
e pull requests. Ambos usam [.pre-commit-config.yaml](.pre-commit-config.yaml)
como definição das verificações.

| Momento | Execução | Escopo |
| --- | --- | --- |
| Antes do commit local | Hook `pre-commit` | Arquivos adicionados ao commit, conforme os filtros de cada hook; as checagens Django e cobertura executam sobre o projeto. |
| Ao validar a mensagem local | Hook `commit-msg` | Mensagem do commit no padrão Conventional Commits. |
| CI: verificações do projeto | `uv run pre-commit run --all-files` | Todos os arquivos aplicáveis aos hooks e as checagens do projeto. |
| CI: mensagens de commit | Chamada explícita ao estágio `commit-msg` | Cada commit do intervalo do push ou pull request. |

## Pre-commit local

### Instalação e ativação

Após clonar o repositório, cada pessoa deve instalar a ferramenta e ativar os
hooks na sua cópia local:

```bash
uv tool install pre-commit
pre-commit install --install-hooks --hook-type pre-commit --hook-type commit-msg
```

O primeiro comando instala o pre-commit como ferramenta do uv, separado das
dependências do projeto. O segundo instala os dois hooks no Git e prepara seus
ambientes. Os mesmos estágios estão definidos em `default_install_hook_types`
na configuração.

É necessário acesso à internet para baixar as ferramentas e os ambientes dos
hooks. Se o comando `pre-commit` não for encontrado após a instalação, execute
`uv tool update-shell` e abra um novo terminal. Os hooks locais também precisam
do uv disponível no `PATH`.

### Verificações configuradas

| Hook | Versão/configuração | O que verifica |
| --- | --- | --- |
| `ruff-check` | Ruff `v0.17.5`, com `--fix` | Analisa o código Python e aplica as correções automáticas disponíveis. |
| `ruff-format` | Ruff `v0.17.5` | Formata o código Python. |
| `pip-audit` | `v2.10.1`, com argumento `.` | Audita as dependências do projeto em busca de vulnerabilidades conhecidas. |
| `django-check` | `uv run manage.py check` | Executa as verificações de configuração do Django. |
| `django-migrations` | `uv run manage.py makemigrations --check` | Detecta alterações nos modelos que exigem novas migrations. |
| `coverage` | `uv run coverage run manage.py test` seguido de `uv run coverage report` | Executa os testes Django e exige cobertura mínima de 95%. |
| `conventional-pre-commit` | `v4.5.0`, estágio `commit-msg` | Valida a mensagem conforme Conventional Commits. |

As verificações Django e de cobertura usam `language: system` e
`pass_filenames: false`: executam os comandos do projeto via uv sem receber uma
lista de arquivos. Como não há `always_run: true`, hooks podem ser ignorados
quando não houver arquivos aplicáveis.

A configuração de cobertura fica em [pyproject.toml](pyproject.toml), com
`source = ["."]`, `fail_under = 95` e `show_missing = true`. A medição exclui os
padrões de migrations, testes, `manage.py`, settings, WSGI, ASGI e `venv`
declarados em `omit`.

### Fluxo de trabalho

```bash
git add caminho/do/arquivo.py
git commit -m "feat: adiciona dashboard"
```

Com os hooks instalados, as verificações rodam durante o commit. Uma falha
interrompe a criação do commit. Quando Ruff alterar arquivos, revise as
correções, execute `git add` nos arquivos alterados e tente novamente.

As mensagens devem seguir Conventional Commits, por exemplo:

```text
feat: adiciona dashboard
fix: corrige configuração do banco
docs: documenta a esteira de CI
```

### Execução manual

Para verificar todos os arquivos ou executar um hook específico:

```bash
pre-commit run --all-files
pre-commit run django-check --all-files
pre-commit run coverage --all-files
```

O comando `--all-files` não executa o estágio `commit-msg`. Para verificar
manualmente a mensagem do último commit, usando Bash:

```bash
message_file=$(mktemp)
trap 'rm -f "$message_file"' EXIT
git log -1 --format=%B > "$message_file"
pre-commit run --hook-stage commit-msg --commit-msg-filename "$message_file"
```

## CI no GitHub Actions

O workflow [ci.yml](.github/workflows/ci.yml), chamado `CI`, é disparado por
`push` e `pull_request`. O job `checks` executa em `ubuntu-latest`, nesta ordem:

1. **Checkout:** usa `actions/checkout@v6` com `fetch-depth: 0`, obtendo o
   histórico completo necessário para selecionar os commits a validar.
2. **Python:** usa `actions/setup-python@v5` para configurar Python 3.14.
3. **uv:** instala o gerenciador com `python -m pip install uv`.
4. **Pre-commit:** instala a ferramenta com `uv tool install pre-commit`.
   Essa instalação é necessária porque ela não está nas dependências do projeto.
5. **Dependências:** executa `uv sync --locked`, respeitando `uv.lock` e
   falhando se o lock estiver desatualizado em relação ao projeto.
6. **Verificações:** executa `uv run pre-commit run --all-files`, incluindo
   Ruff, auditoria de dependências, checagens Django e testes com cobertura.
7. **Mensagens:** executa explicitamente o hook `commit-msg` para cada commit
   selecionado, conforme o procedimento abaixo.

As etapas seguem em sequência: uma falha interrompe as próximas etapas e deixa
o job com erro. Correções automáticas do Ruff devem ser aplicadas localmente e
enviadas em um novo commit. Os logs ficam na aba **Actions** do repositório,
na execução do workflow `CI`, dentro do job `checks`.

### Solução para executar o hook de mensagens no GitHub

O runner verifica commits que já existem; por isso, a validação usa uma chamada
explícita ao estágio `commit-msg`. Rodar apenas `pre-commit run --all-files`
não valida as mensagens.

A etapa `Validar mensagens de commit` define as referências conforme o evento:

| Evento | Base (`BASE_SHA`) | Head (`HEAD_SHA`) |
| --- | --- | --- |
| Pull request | `github.event.pull_request.base.sha` | `github.event.pull_request.head.sha` |
| Push | `github.event.before` | `github.event.after` |

O script usa `git rev-list --reverse "$BASE_SHA..$HEAD_SHA"` para percorrer os
commits do mais antigo ao mais recente. Em pushes de criação de branch, o SHA
anterior contém apenas zeros. Nesse caso, compara o head com
`refs/remotes/origin/<branch padrão>`, se essa referência existir e for outra
branch. Caso contrário, percorre todo o histórico alcançável pelo head.
Tags e exclusões de branches não executam a etapa de mensagens.

Para cada SHA, o script salva a mensagem completa em um arquivo temporário e
chama o hook usando esse arquivo:

```bash
git show -s --format=%B "$sha" > "$message_file"
uv run pre-commit run \
  --hook-stage commit-msg \
  --commit-msg-filename "$message_file"
```

Essa chamada reutiliza o hook `conventional-pre-commit` da configuração local.
Uma mensagem inválida faz a etapa falhar. O arquivo temporário é removido ao
encerrar o script por meio de `trap`, inclusive em caso de falha. A implementação
completa, com os comentários de cada tratamento, está no workflow.
