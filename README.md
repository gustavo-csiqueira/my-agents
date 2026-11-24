# 🤖 Minhas automações com n8n

Faz um tempo que comecei a usar o [n8n](https://n8n.io) para tirar do meu caminho aquelas pequenas tarefas repetitivas do dia a dia: conferir agenda, transformar e-mail em tarefa, lembrar aniversário, anotar gasto… Nada muito sofisticado, mas somado faz bastante diferença.

Resolvi juntar tudo aqui e compartilhar, caso alguém também ache útil. Cada pasta é um "agente": tem o workflow exportado em JSON (é só importar no n8n) e um README contando o que ele faz, como configurar e como ele melhorou um pouco a minha rotina.

## Os agentes

| # | Agente | O que faz |
|---|--------|-----------|
| 01 | [Resumo matinal](01-resumo-matinal/) | Às 7h manda no Telegram a agenda do dia, o clima e as tarefas pendentes. |
| 02 | [E-mail para tarefa](02-email-para-tarefa/) | E-mail com a etiqueta "fazer" vira tarefa no Notion, com link para o e-mail original. |
| 03 | [Lembrete de aniversários](03-lembrete-aniversarios/) | Lê uma planilha e avisa no Telegram quem faz aniversário amanhã. |
| 04 | [Controle de gastos](04-controle-de-gastos/) | Bot do Telegram: mando "mercado 85,90" e ele registra no Google Sheets, já categorizado. `/resumo` mostra o mês. |
| 05 | [Triagem de e-mails com IA](05-triagem-de-emails-ia/) | Um LLM classifica cada e-mail novo, aplica a etiqueta certa e me avisa dos urgentes. |
| 06 | [Digest RSS](06-digest-rss/) ⭐ | Todo domingo, um digest com os artigos da semana dos meus feeds, resumidos em português e agrupados por tema. |
| 07 | [Monitor de vagas](07-monitor-de-vagas/) | A cada 6h procura vagas por palavras-chave e avisa só as novas no Telegram. |

## Como importar um workflow no n8n

1. Abra o n8n (self-hosted ou n8n Cloud).
2. Crie um workflow novo (**+ → Workflow**).
3. No menu **⋯** do canto superior direito, clique em **Import from File…** e escolha o `workflow.json` da pasta do agente.
   *Atalho:* abra o JSON, copie tudo e cole (Ctrl+V) direto no canvas.
4. Os nós que precisam de credencial vão aparecer com um alerta. Abra cada um, em **Credential** escolha **Create new credential** e siga o assistente. Os nomes usados nos workflows são genéricos (`Telegram Bot`, `Gmail account`, `Google Sheets account`…), então dá para reaproveitar uma credencial entre vários agentes.
5. Procure os **placeholders** (`SEU_CHAT_ID`, `ID_DA_PLANILHA`, `SEU_FEED_RSS`…) e troque pelos seus valores. As notas amarelas no canvas e o README de cada agente dizem exatamente onde estão.
6. Clique em **Execute workflow** para testar manualmente.
7. Quando estiver tudo certo, ative o workflow no botão **Active** (gatilhos de agendamento, Gmail e Telegram só rodam sozinhos com o workflow ativo).

> Todos os workflows usam o fuso **America/Sao_Paulo** (em *Workflow settings → Timezone*). Ajuste se você estiver em outro fuso.

### Dica: descobrindo seu `chat_id` do Telegram

1. Crie um bot com o [@BotFather](https://t.me/BotFather) e guarde o token (ele vai na credencial `Telegram Bot`, nunca no JSON).
2. Mande qualquer mensagem para o seu bot.
3. Acesse `https://api.telegram.org/bot<SEU_TOKEN>/getUpdates` e procure `"chat":{"id": ...}`. Esse número é o seu `SEU_CHAT_ID`.

## ⚠️ Nunca commite credenciais

Os workflows deste repositório **não têm nenhum token, senha ou ID real**: só referências a credenciais com nomes genéricos e placeholders. Se você fizer um fork e exportar os seus workflows depois de configurados:

- O n8n **não** exporta o conteúdo das credenciais, mas exporta IDs de planilhas, bancos do Notion, chat IDs, e-mails etc. Revise o JSON antes de commitar.
- Nunca coloque chaves de API direto em parâmetros de nós (por exemplo, numa URL ou num campo de texto). Use sempre o sistema de credenciais do n8n.
- O `.gitignore` já ignora `.env` e arquivos de credenciais comuns, mas ele não substitui uma revisão antes do `git push`.
- Se algo vazar, **revogue a chave imediatamente**. Apagar o commit não basta.

## Quer adaptar ou contribuir?

Fique à vontade para usar, adaptar e mudar o que quiser. Essas automações foram feitas para a minha rotina, então certamente tem coisa que faz mais sentido de outro jeito para você.

Se você melhorou algum fluxo, encontrou um bug ou tem uma ideia de agente novo, abre uma [issue](../../issues) ou manda um pull request. Vou gostar de ver!

## Licença

Este projeto está sob a [GNU General Public License v3.0](LICENSE) (GPL-3.0). Você pode usar, copiar, modificar e redistribuir à vontade. Se distribuir uma versão modificada, ela também precisa ser publicada sob a GPL-3.0, com o código-fonte disponível.
