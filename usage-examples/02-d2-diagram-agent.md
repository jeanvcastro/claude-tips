# Prompt: Agente D2 Diagram Architect

## Contexto
Demonstra o uso de agentes especialistas para tarefas que não são codificação direta.
Após rodar o cenário 01, usamos o agente D2 para gerar um diagrama da arquitetura da API.

## Pré-requisito
- Ter rodado o cenário 01 (ou ter qualquer código Go em output/)
- Ter d2 instalado (`brew install d2` ou https://d2lang.com)

## Prompt

```
Use o agente d2-diagram-architect para criar um diagrama da arquitetura da API em output/01-workflow-basico/.

O diagrama deve mostrar:
- O HTTP server com os endpoints (/health, /users, /users/{id})
- Os middlewares (logging, recovery) como camada intermediária
- O UserStore como camada de dados
- O fluxo de uma request POST /users passando por cada camada
- Métodos HTTP como labels nas conexões

Salve o .d2 e o .png em output/02-agente-d2/
```
