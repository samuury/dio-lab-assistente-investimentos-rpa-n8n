# Workflow n8n: e-mails personalizados de investimento

**Fluxo final:** Webhook → leitura do CSV → Code 1 → Code 2 → Nó de IA → Code 3 → IF → Gmail

**Objetivo:** receber clientes e produtos de investimento e enviar a cada cliente um e-mail personalizado conforme perfil (Conservador, Moderado, Arrojado) e saldo.

## Desafios e soluções

| # | Desafio | Causa | Solução |
|---|---------|-------|---------|
| 1 | Saldo vinha como texto (`"R$ 12.500,00"`) | Formato brasileiro com ponto, vírgula e símbolo | Função `parseSaldo` que converte para número |
| 2 | Produtos acima do saldo apareciam na mensagem | Faltava filtro por valor mínimo | Filtro por perfil **e** `minimo <= saldo` |
| 3 | `\n` e `•` literais na mensagem | Quebras de linha escapadas no texto | Mensagem em linha única, sem quebras |
| 4 | IA devolvia `[Nome do cliente]`, `[Produto 1]` | Dados não chegavam ao prompt | User prompt em modo **Expression** com `{{ $json.campo }}` |
| 5 | IA ignorava as regras | System prompt colocado no role errado | Regras em **System**, dados em **User**, Assistant vazio |
| 6 | Observações para o remetente dentro do e-mail | Faltava instrução explícita | Regra: responder só com o e-mail pronto, assinar "Equipe de Investimentos" |
| 7 | Saída em texto/markdown, difícil de enviar | Formato livre | IA responde só JSON: `{"email","assunto","corpo"}` com HTML simples |
| 8 | `Referenced node doesn't exist` | Nome do nó citado em `$('Code')` não batia | Usar o nome exato do nó ou pedir o `email` no próprio JSON da IA |
| 9 | Campo `detalhes` inexistente no prompt | Code 1 era outro (só preparava `investimentos`) | Separar papéis: Code 1 lê o CSV, Code 2 cruza clientes e produtos |
| 10 | Clientes não encontrados no Code 2 | O input era o CSV; clientes estavam no Webhook | Buscar clientes com `$('Webhook').first().json` |
| 11 | JSON cortado no meio (5 de 10 e-mails) | Limite de tokens baixo | Max Tokens maior, texto mais curto, Code 3 com `try/catch` e nó **IF** (`ok = true`) |

## Lições aprendidas

- **Cada Code com uma responsabilidade** evita confusão sobre quais campos existem em cada etapa.
- **Prompt de sistema e de usuário são coisas diferentes**, e os dados do cliente só funcionam em modo Expression.
- **Peça JSON à IA** e valide antes de enviar. Nunca envie direto a saída do modelo.
- **Proteja o fluxo:** o IF barra e-mails incompletos, e *Retry On Fail* tenta de novo.
- **Teste com o seu e-mail** antes de ativar o envio real.

## Antes de ir para produção

- Trocar a URL `webhook-test` por `/webhook/Clientes` e ativar o workflow.
- Substituir os e-mails fictícios por dados reais, conferindo a lista antes do primeiro disparo.
