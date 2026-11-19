# Registro de prompts

Registre aqui os prompts que produziram artefatos usados no projeto, conforme
`AGENTS.md`. A identidade deve seguir a ordem de preferência do contexto Git;
o aceite deve refletir a revisão do integrante responsável. Uma saída gerada
não implica aprovação.

## 2026-11-19 — Arthur Araujo — instruções para assistentes

Prompt (trechos literais; foi omitida a transcrição da seção 13 do enunciado
fornecida entre os dois trechos):
> crie um AGENTS.md com base em:
>
> e tambem leve em consideracao TDD, conventional com mensagem em portugues

Contexto fornecido: seção 13, “Uso de assistentes de inteligência artificial”,
do Trabalho Final de RAD do IFPB. Exige compreensão da entrega pelos integrantes,
autoria humana dos commits, versionamento das configurações e artefatos de IA,
planos e especificações em `docs/`, proteção de credenciais e registro dos
prompts relevantes com seu resultado real.

Resultado: aceito com ajustes — instruções adaptadas ao projeto, incluindo TDD
e Conventional Commits com mensagens em português; gerados `AGENTS.md` e este
registro; removida a regra que ignorava `docs/`, para permitir versionar os
planos e especificações. Integrante preenchido a partir de `git config user.name`
e aceite registrado conforme orientação do responsável.

## 2026-11-19 — Arthur Araujo — identificação do integrante pelo Git

Prompt:
> adicione ao agents.md essa configuracao de pegar do contexto git o integrante, ou nome ou nome de usuario, o que estiver disponivel, mas com essa ordem de preferencia

Resultado: pendente de revisão — definida a preferência pelo nome configurado
no Git, seguido do nome de usuário disponível da conta autenticada. Na ausência
dos dois, o registro deve indicar `integrante não informado`.

## 2026-11-19 — Arthur Araujo — pergunta de aceite pelo sistema do agente

Prompt:
> adicione uma regra ao agents.md em que seja adicionado ao final de cada prompt que for gerar um artefato no projeto e um registro no prompts.md uma pergunta sobre o aceite com as seguintes opcoes: aceito como veio, aceito com ajustes (indicando os ajustes) e descartado, pergunta essa que deve utilizar o sistema de pergunta do agente, nao apenas escrever no chat e esperar mais um prompt, se possivel

Resultado: aceito como veio — adicionada a regra de perguntar o aceite pelo
sistema de perguntas do agente, com as três opções, detalhamento dos ajustes
e atualização do resultado após a resposta. Previsto o uso do chat apenas se
a ferramenta estiver indisponível ou não for permitida no modo atual.
