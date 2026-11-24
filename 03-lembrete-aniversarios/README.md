# 🎂 03 · Lembrete de aniversários

## O que faz

Todo dia às 9h lê uma planilha do Google Sheets com nomes e datas de nascimento e, se alguém faz aniversário **amanhã**, manda um aviso no Telegram com a idade e uma observação (ex.: "gosta de vinho tinto").

```
🎉 Aniversários de amanhã

🎂 Maria faz 34 anos amanhã (25/11)
   📝 prima, mora em BH

Não esquece de mandar uma mensagem! 😉
```

## Como meu dia ficou melhor

Eu dependia das notificações das redes sociais para lembrar de aniversário, e várias pessoas importantes nem estão lá. Resultado: mais de uma vez mandei parabéns atrasado e pedindo desculpa. Com o aviso chegando um dia antes, dá tempo de ligar, comprar um presente ou pelo menos escrever algo com calma, em vez de um "parabéns!!" às 23h. A planilha também virou um lugar para anotar detalhes que eu acabaria esquecendo.

## Como funciona

1. **Schedule Trigger**: todo dia às 9h (`America/Sao_Paulo`).
2. **Google Sheets**: lê todas as linhas da aba `Aniversarios`.
3. **Filter**: mantém só quem tem dia/mês igual ao de amanhã.
4. **Set**: monta uma linha formatada por pessoa e calcula a idade (se o ano estiver preenchido).
5. **Aggregate**: junta todas as linhas num único item.
6. **Telegram**: envia uma mensagem só com todos os aniversariantes.

Se ninguém faz aniversário amanhã, o fluxo termina no filtro e nada é enviado.

## Pré-requisitos

- Conta Google com Google Sheets (credencial OAuth2 no n8n)
- Bot do Telegram e o seu chat ID
- Uma planilha com uma aba chamada `Aniversarios` e estas colunas na primeira linha:

| nome | data_nascimento | observacao |
|------|-----------------|------------|
| Maria | 25/11/1992 | prima, mora em BH |
| João | 03/02 | amigo de infância |

> Use o formato **dd/mm/aaaa** (ou **dd/mm** quando não souber o ano). Para evitar conversões de data, formate a coluna `data_nascimento` como **Texto simples** (*Formatar → Número → Texto simples*).

## Como importar e configurar

1. Importe o `workflow.json` no n8n.
2. Crie as credenciais `Google Sheets account` (Google Sheets OAuth2 API) e `Telegram Bot` (Telegram API).
3. Substitua os placeholders:
   | Onde | Placeholder | Trocar por |
   |---|---|---|
   | **Ler planilha de aniversários** | `ID_DA_PLANILHA` | ID da planilha (trecho da URL entre `/d/` e `/edit`) |
   | **Ler planilha de aniversários** | `Aniversarios` | nome da aba, se for diferente |
   | **Avisar no Telegram** | `SEU_CHAT_ID` | seu chat ID |
4. Para testar, coloque na planilha alguém com a data de amanhã e execute manualmente.

## Ideias de personalização

- Avisar com 7 dias de antecedência também (duplicar o filtro com `days: 7`) para dar tempo de comprar presente.
- Mandar também no dia, às 8h, com um lembrete de "hoje é o dia".
- Adicionar uma coluna `telefone` e gerar um link `https://wa.me/...` para mandar mensagem com um toque.
- Usar um LLM para sugerir uma mensagem de parabéns a partir da observação.
- Colocar datas comemorativas na mesma planilha (aniversário de casamento, de namoro etc.).
