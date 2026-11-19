# Instruções para assistentes de IA — DashCast

Estas instruções se aplicam a todo o repositório. Baseiam-se na seção 13 do
enunciado do Trabalho Final de RAD do IFPB, nas convenções do projeto e na
exigência de desenvolvimento com TDD.

## Responsabilidade e compreensão

- O uso de IA é permitido e esperado. Cada integrante precisa saber explicar
  qualquer parte do que o grupo entregar, inclusive código gerado por IA.
- Produza soluções simples e explique as decisões, o comportamento, os testes
  executados e as limitações. O integrante deve conseguir revisar e defender a
  alteração, e não apenas reproduzir uma explicação pronta.
- Quando faltar uma regra de negócio, procure nos requisitos versionados e
  pergunte ao integrante se a dúvida afetar o comportamento. Não invente regras.
- O histórico e os registros de IA complementam a avaliação; não substituem a
  conversa prevista no enunciado para 09/12.

## Contexto e comandos do projeto

O projeto usa Python 3.14, Django, uv, Ruff, pip-audit, coverage e pre-commit.
Consulte `pyproject.toml`, `uv.lock`, `.pre-commit-config.yaml`,
`.github/workflows/ci.yml` e `README.md` antes de alterar a esteira.

```bash
# Instalar as dependências existentes sem modificar o lock
uv sync --locked

# Instalar a ferramenta e os hooks locais
uv tool install pre-commit
pre-commit install --install-hooks --hook-type pre-commit --hook-type commit-msg

# Verificações do Django
uv run manage.py check
uv run manage.py makemigrations --check

# Executar testes e medir a cobertura
uv run coverage run manage.py test
uv run coverage report

# Executar as verificações aplicáveis a todos os arquivos
pre-commit run --all-files
```

O CI roda em pushes e pull requests. Prepara Python, uv, pre-commit e
dependências, executa os hooks sobre todos os arquivos, valida separadamente
as mensagens de commit e publica o relatório de cobertura no resumo do job.
O checkout completo permite selecionar os commits do push ou pull request.
`pre-commit run --all-files` não executa o estágio `commit-msg`.

## Desenvolvimento com TDD

Para funcionalidades e correções de comportamento, siga o ciclo:

1. **Vermelho:** escreva primeiro um teste que expresse o requisito ou reproduza
   o defeito. Execute-o e confirme que falha pelo motivo esperado.
2. **Verde:** implemente o código necessário para fazer esse teste passar.
3. **Refatoração:** melhore a implementação mantendo os testes verdes e execute
   os testes relacionados para procurar regressões.

- Use os testes do Django, descobertos por `manage.py test`. Para executar um
  teste específico, use `uv run manage.py test <modulo_ou_classe_de_teste>`.
- Teste comportamentos observáveis, regras de negócio, casos de erro e limites
  relevantes. Não escreva testes que apenas repitam a implementação.
- Para mudanças apenas em documentação ou comentários, faça a revisão e as
  validações pertinentes; não crie testes artificiais para esses arquivos.
- A cobertura mínima configurada é **95%**. Não reduza o limite nem exclua código
  de negócio da medição para fazer a esteira passar.
- Cobertura alta não comprova qualidade dos testes. O estado inicial pode
  mostrar 100% sem testes porque mede apenas código de inicialização. Para
  comportamentos novos, confirme que os testes existem e realmente executam.
- Não apresente uma execução com zero testes como evidência de comportamento
  validado. Informe quando uma verificação não tiver sido executada ou falhar.

## Alterações e validação

- Respeite as convenções existentes e mantenha as mudanças focadas no pedido.
- Use uv para gerenciar dependências; quando elas mudarem, atualize também
  `uv.lock`. Não edite o lock manualmente.
- Ao alterar modelos, gere e versione as migrations correspondentes. Não
  versione `db.sqlite3`.
- Antes de concluir uma mudança de código, execute os testes relevantes e as
  verificações da esteira. Corrija falhas; não contorne os hooks com `--no-verify`
  ou altere os testes apenas para esconder defeitos.
- Revise as correções automáticas do Ruff. No CI, elas não criam um commit com
  os arquivos corrigidos; aplique e envie as correções pelo fluxo do projeto.
- Atualize o README quando alterar instalação, comandos, configuração ou CI.

## Commits e autoria

- Use Conventional Commits no formato `tipo(escopo opcional): descrição`.
  Mantenha os tipos convencionais em inglês e escreva a descrição e o corpo
  em português, de forma objetiva.
- Exemplos: `feat(inscricoes): valida capacidade da sala`,
  `fix: corrige cálculo de vagas`, `test: cobre inscrição em sala lotada` e
  `docs: documenta a esteira de validação`.
- Faça commits coesos, com mensagens que descrevam o que realmente mudou.
- A autoria do commit é do integrante que o realiza e responde por ele.
  Não substitua sua identidade pela do assistente, não invente autoria ou
  coautoria e não altere `user.name` ou `user.email` por conta própria.
- Só crie commits ou envie alterações ao remoto quando houver autorização do
  integrante na conversa. Preserve a identidade Git configurada e eventuais
  configurações de assinatura existentes.

## Artefatos de IA são código

- Mantenha versionados este `AGENTS.md` e os arquivos de instruções das
  ferramentas efetivamente usadas pelo grupo, como `CLAUDE.md`, `.claude/`,
  `.github/copilot-instructions.md`, `.cursor/rules/` e `.cursorrules`.
- Versione também prompts, scripts, agentes, skills e comandos próprios usados
  para gerar, revisar ou testar código. Não crie configurações de ferramentas
  que o grupo não usa apenas para preencher o repositório.
- Antes da entrega, confira o `.gitignore` e as regras efetivas com
  `git check-ignore -v <caminho>`. Documentação e configurações de IA usadas pelo
  grupo não podem ficar ignoradas.
- Não coloque chaves de API, tokens ou senhas nesses arquivos. Se uma
  configuração exigir credenciais, use variáveis de ambiente, mantenha o
  arquivo real de segredos fora do Git e documente apenas valores fictícios
  em `.env.example`.

## Planos e especificações

- Se um plano, especificação, divisão de tarefas ou documento de arquitetura
  gerado por IA for usado para orientar a implementação, salve-o em `docs/` e
  inclua-o no versionamento.
- Preserve o conteúdo realmente usado. Não reescreva um plano depois para
  simular decisões anteriores ou esconder diferenças em relação à entrega.
- Use nomes descritivos, por exemplo `docs/2026-11-19-plano-inscricoes.md`.
  Não é necessário criar um plano para toda alteração pequena.

## Registro de prompts

- Mantenha `PROMPTS.md` na raiz com os prompts que produziram artefatos usados
  no projeto: regras, modelos, código, testes, documentação ou configurações.
  Não copie a conversa inteira nem registre toda pergunta exploratória.
- Cada entrada deve conter data, integrante, assunto, prompt e resultado.
  Transcreva o prompt relevante; sinalize explicitamente qualquer resumo ou
  omissão. Nunca inclua segredos.
- Preencha o integrante usando o contexto Git, nesta ordem de preferência:
  primeiro o nome retornado por `git config user.name`, executado no
  repositório; se não estiver disponível, o nome de usuário da conta Git
  autenticada, quando essa informação estiver disponível e identificar o
  responsável pela sessão. Não use o dono do repositório remoto como se fosse
  o integrante nem deduza um nome de usuário a partir do e-mail.
- Registre a avaliação real: `aceito como veio`, `aceito com ajustes` ou
  `descartado`, explicando os ajustes quando houver. Enquanto o integrante não
  tiver revisado a saída, use `pendente de revisão`.
- Não invente nomes, aceites, testes ou registros retroativos. Se nem o nome
  nem o nome de usuário estiverem disponíveis, registre `integrante não informado`
  para o responsável completar. Não altere a configuração Git para preencher
  o registro.
- O registro é declaratório e deve ser coerente com o histórico de commits.

Formato de uma entrada, com campos a preencher:

```markdown
## AAAA-MM-DD — <integrante> — <assunto>

Prompt:
> <texto do prompt>

Resultado: <avaliação real e ajustes, se houver>
```

### Pergunta de aceite ao concluir a entrega

- Ao concluir um pedido que gere ou altere um artefato do projeto e tenha uma
  entrada em `PROMPTS.md`, apresente o resultado e as validações realizadas e
  solicite a avaliação do integrante. Faça isso depois de preparar a entrega
  para revisão, sem usar o aceite como condição para começar o trabalho.
- Use o sistema de perguntas do agente, quando disponível e permitido no modo
  atual, como `request_user_input_async` ou `request_user_input`. Ofereça estas
  três opções, sem selecionar ou submeter uma resposta pelo integrante:
  - `aceito como veio`;
  - `aceito com ajustes` — peça que o integrante indique os ajustes;
  - `descartado`.
- Permita informar os ajustes em texto livre. Se o integrante escolher
  `aceito com ajustes` sem descrevê-los, solicite o detalhamento pelo mesmo
  sistema de perguntas antes de completar o registro do resultado.
- Atualize a entrada correspondente em `PROMPTS.md` conforme a resposta real,
  incluindo os ajustes informados. Não interprete silêncio, tempo decorrido
  ou opção pré-selecionada pela interface como aceite. Enquanto não houver
  resposta suficiente, mantenha `pendente de revisão`.
- Se a ferramenta de perguntas estiver indisponível ou não for permitida no
  modo atual, explique a limitação e apresente as mesmas opções no chat,
  mantendo o resultado pendente até a resposta do integrante.
- A atualização do próprio registro de aceite não exige outra entrada nem uma
  nova pergunta. Não crie um ciclo de confirmação para registrar a avaliação.
