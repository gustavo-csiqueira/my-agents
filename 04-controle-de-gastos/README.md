# 💸 04 · Controle de gastos

## O que faz

Um bot do Telegram que funciona como caderninho de gastos. Mando `mercado 85,90` e ele registra no Google Sheets a data, a descrição, a categoria (definida por palavras-chave) e o valor, e responde confirmando. O comando `/resumo` devolve o total do mês por categoria.

```
eu:  mercado 85,90
bot: ✅ Anotado: mercado — R$ 85,90 (Alimentação)

eu:  /resumo
bot: 📊 Gastos de novembro/2026
     • Alimentação: R$ 1.230,40 (48%)
     • Transporte: R$ 610,00 (24%)
     • Moradia: R$ 520,00 (20%)
     • Outros: R$ 198,50 (8%)

     💰 Total: R$ 2.558,90
```

## Como meu dia ficou melhor

Já tentei vários apps de finanças e sempre abandonava depois de umas semanas: abrir o app, escolher categoria, preencher campo, salvar… dava preguiça e eu deixava para "anotar depois", o que nunca acontecia. O Telegram já está aberto o dia inteiro, então mandar `uber 23,50` leva dois segundos. Faz alguns meses que registro praticamente tudo, e o `/resumo` no meio do mês me ajuda a perceber quando estou exagerando no delivery antes da fatura chegar. E como os dados ficam numa planilha minha, dá para montar os gráficos que eu quiser.

## Como funciona

1. **Telegram Trigger**: recebe as mensagens enviadas ao bot.
2. **Só o meu chat** (Filter): ignora mensagens de qualquer chat que não seja o seu (o bot é público, então isso é importante).
3. **Switch**: decide o que fazer:
   - começa com `/resumo` → ramo de resumo;
   - termina com um número (ex.: `mercado 85,90`, `uber 23.50`, `aluguel R$ 1.500,00`) → ramo de registro;
   - qualquer outra coisa → responde explicando o formato.
4. **Registro**
   1. **Code**: separa descrição e valor (aceita vírgula, ponto e separador de milhar) e escolhe a categoria pelas palavras-chave.
   2. **Google Sheets**: adiciona uma linha `data | descricao | categoria | valor` na aba `Gastos`.
   3. **Telegram**: confirma o registro.
5. **Resumo**
   1. **Google Sheets**: lê todas as linhas da aba `Gastos`.
   2. **Code**: filtra o mês atual, soma por categoria e calcula percentuais.
   3. **Telegram**: envia o resumo.

## Pré-requisitos

- Bot do Telegram (criado no [@BotFather](https://t.me/BotFather)) e o seu chat ID
- Conta Google com Google Sheets
- Uma planilha com uma aba `Gastos` e o cabeçalho na primeira linha:

| data | descricao | categoria | valor |
|------|-----------|-----------|-------|

- O n8n precisa estar acessível pela internet (HTTPS) para o Telegram entregar as mensagens via webhook. O n8n Cloud já é; no self-hosted, configure a variável `WEBHOOK_URL` ou use um túnel.

## Como importar e configurar

1. Importe o `workflow.json` no n8n.
2. Crie as credenciais `Telegram Bot` (Telegram API) e `Google Sheets account` (Google Sheets OAuth2 API).
3. Substitua os placeholders:
   | Onde | Placeholder | Trocar por |
   |---|---|---|
   | **Só o meu chat** | `SEU_CHAT_ID` | seu chat ID |
   | **Adicionar linha na planilha** | `ID_DA_PLANILHA` | ID da planilha |
   | **Ler gastos** | `ID_DA_PLANILHA` | o mesmo ID |
   | Os dois nós do Sheets | `Gastos` | nome da aba, se for diferente |
4. Ajuste as categorias e palavras-chave no início do nó **Extrair e categorizar** (objeto `CATEGORIAS`).
5. Ative o workflow (o Telegram Trigger só registra o webhook com o workflow ativo) e mande uma mensagem para o bot.

> Os valores são gravados como dados "crus" (*cellFormat RAW*): a data fica como texto `dd/mm/aaaa` e o valor como número, o que evita a planilha converter formatos sozinha.

## Ideias de personalização

- Comando `/categorias` listando as categorias e palavras-chave.
- Permitir informar a categoria manualmente com `#`: `presente mãe 120 #família`.
- Comando `/desfazer` para apagar a última linha registrada.
- Alerta automático quando uma categoria passar de um limite mensal.
- Mandar foto do cupom fiscal e usar um LLM com visão para extrair o valor.
- Um resumo automático no último dia do mês, sem precisar pedir.
