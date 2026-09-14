# Revisão de código — validador de CPF

## Ferramenta e modelo utilizado
Claude Sonnet 5, via claude.ai (skill de revisão criada com o skill-creator da Anthropic).

## Uma pergunta que não tinha pensado
Ao criar a skill, a ferramenta perguntou o que ela conseguiria checar de forma
executável além de comparar o diff com a main — se ela podia só ler o código,
abrir o HTML no navegador, ou rodar comandos de lint/teste. Isso me fez perceber
que "revisar" não é só apontar problema: a skill também precisa deixar claro
o que ela verificou de fato rodando algo e o que ela só leu, sem confundir os dois.

## Achado
O código do validador (`validador/index.html`) calcula os dois dígitos
verificadores do CPF em dois blocos de código quase idênticos (linhas 89–110),
mudando só os pesos usados na soma. A skill, na primeira rodada, não apontou
essa duplicação. Ajustei o `SKILL.md` para incluir explicitamente a checagem
de lógica duplicada entre blocos do diff, e na segunda rodada ela passou a
identificar o problema.
