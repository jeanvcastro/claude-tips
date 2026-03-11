# Prompt: Ralph Loop para POC

## Contexto
Demonstra o modo go-horse: loop autônomo com contexto fresco a cada iteração.
Cria uma CLI em Go de forma totalmente autônoma usando Ralph.

## Pré-requisito
- Ralph instalado: https://github.com/snarktank/ralph

## Prompt 1 — Gerar PRD

```
Crie um PRD em output/04-ralph-loop/prd.md para uma CLI Go que converte Markdown para HTML:

User stories (cada uma pequena e testável):
1. Setup: go mod init, main que lê path do .md como argumento CLI, exit 1 se arquivo não existe
2. Parser básico: converter headings, paragraphs, bold, italic, links para HTML
3. Code blocks: detectar blocos com linguagem (```go, ```js) e aplicar classes CSS
4. Output: flag --output/-o para arquivo, sem flag vai pra stdout, CSS inline
5. Template HTML5: head, meta charset, title, tema dark para code blocks, responsivo

Cada story deve ter critérios de aceitação claros e um teste automatizado.
```

## Prompt 2 — Converter para prd.json

```
Converta output/04-ralph-loop/prd.md para output/04-ralph-loop/prd.json no formato Ralph:

{
  "branchName": "poc-md2html",
  "userStories": [
    {
      "id": "US-001",
      "title": "...",
      "acceptanceCriteria": ["..."],
      "passes": false,
      "priority": 1
    }
  ]
}
```

## Prompt 3 — Rodar o loop

```bash
cd output/04-ralph-loop
git init && git add -A && git commit -m "initial prd"
./scripts/ralph/ralph.sh --tool claude 10
```

## O que vai acontecer
1. Ralph cria branch `poc-md2html`
2. Pega US-001 (maior prioridade, passes: false)
3. Implementa, roda `go build` + `go test`
4. Passa → commit automático, marca passes: true, pega US-002
5. Falha → registra em progress.txt, tenta de novo com contexto fresco
6. Todas passes: true → loop encerra

## Resultado esperado em output/04-ralph-loop/
- Código Go funcional da CLI
- prd.json com todas as stories passes: true
- progress.txt com aprendizados
- Git log com commits incrementais por story
