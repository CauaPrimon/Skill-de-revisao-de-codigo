---
name: revisao-de-codigo
description: Revisa apenas o que mudou entre a branch atual e a main de um repositório Git, apontando arquivo, linha e forma de reproduzir cada achado. Use esta skill sempre antes de abrir um Pull Request (autorrevisão) e sempre que for revisar o PR de um colega. Não use para revisar o repositório inteiro, nem para comentar sobre formatação/estilo.
---

# Revisão de código (diff vs main)

## Quando usar
- Antes de abrir um Pull Request, para revisar o próprio código antes de mandar.
- Ao revisar o PR de um colega, apontando o branch dele.

## O que ela não faz
- Não reclama de formatação/indentação/espaçamento.
- Não aponta um problema sem dizer o arquivo e a linha exata.
- Não sugere refatoração sem dizer qual problema concreto aquilo resolve.
- Não revisa código que não mudou em relação à main.

## Passo a passo

### 1. Descobrir o que mudou
```bash
git fetch origin main
git diff origin/main...HEAD --name-only
git diff origin/main...HEAD
```
Se o branch a revisar não é o atual (ex: revisando o PR de um colega), primeiro faça checkout nele:
```bash
git fetch origin <branch-do-colega>
git checkout <branch-do-colega>
git diff origin/main...HEAD --name-only
git diff origin/main...HEAD
```
Revise **somente** os trechos que aparecem nesse diff. Arquivos ou linhas fora do diff estão fora do escopo, mesmo que pareçam problemáticos.

### 2. Tentar verificar de forma executável (não só ler)
Antes de dar qualquer veredito, procure por ferramentas de verificação já configuradas no repositório:
```bash
ls package.json pyproject.toml .eslintrc* .flake8 Makefile 2>/dev/null
```
- Se houver `package.json` com scripts de `lint` ou `test`, rode:
  ```bash
  npm install --silent && npm run lint 2>&1
  npm test 2>&1
  ```
- Se houver linter/formatter Python configurado (flake8, ruff, pytest), rode o equivalente.
- Se o diff for HTML/JS puro sem tooling nenhum (ex: `validador/index.html`), não existe comando de lint/teste para rodar — nesse caso, isso vai para a seção "Apenas lido" no final, e não invente verificação que não rodou.
- Guarde a saída bruta de cada comando executado (vai ser citada no relatório).

### 3. Analisar cada achado do diff
Além de bugs e comportamento incorreto, procure também por **lógica duplicada dentro do próprio diff**: dois ou mais blocos que fazem cálculos/validações quase idênticos (mesma estrutura de loop, condição ou fórmula, mudando só um valor). Isso conta como achado de categoria `preferência` — aponte os dois trechos (arquivo:linha de cada um) e sugira extrair a parte repetida em uma função, explicando o problema concreto que isso evita (ex: corrigir um bug em uma cópia e esquecer a outra).

Para cada problema encontrado nos trechos alterados, registre:
- **Arquivo e linha** (ex: `validador/index.html:47`).
- **O que está errado**, em uma frase objetiva.
- **Como reproduzir**: passo a passo mínimo para o colega ver o problema acontecer (ex: "digitar `111.111.111-11` no campo e clicar em Validar — o algoritmo aceita porque a checagem de dígitos repetidos só cobre o formato sem pontuação").
- **Categoria**: `quebra` (o programa não funciona, dá erro, ou o comportamento está incorreto) ou `preferência` (funciona, mas poderia ser mais claro/seguro/manutenível). Nunca misture as duas no mesmo item.

### 4. Montar o comentário pronto para colar no PR
Gere a saída neste formato, pronta para copiar e colar como comentário do PR:

```markdown
## Revisão do diff (branch vs main)

### 🔴 Quebra o programa
- **arquivo:linha** — descrição do problema.
  Como reproduzir: ...

### 🟡 Preferência (não bloqueia)
- **arquivo:linha** — descrição do problema.
  Por que mudar: qual problema concreto isso evita.

### ✅ Verificação
**Rodei:**
- `comando` → resultado resumido (ok / falhou, com trecho relevante da saída)

**Apenas li (sem comando disponível):**
- arquivo X — motivo pelo qual não havia como rodar/testar automaticamente
```

Se uma seção ficar vazia (ex: nenhum achado de "quebra"), mantenha o cabeçalho e escreva "Nenhum achado nesta categoria." — não omita a seção.

### 5. Antes de entregar, confira
- Todo achado tem arquivo + linha + forma de reproduzir?
- Nenhum item de formatação/estilo entrou como achado?
- Toda sugestão de refatoração explica o problema que resolve?
- A seção de verificação separa claramente o que foi rodado do que foi só lido?
