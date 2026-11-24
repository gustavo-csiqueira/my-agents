# 🌅 01 · Resumo matinal

## O que faz

Todo dia às 7h (horário de Brasília) junta os eventos do dia no Google Calendar, o clima da minha cidade (OpenWeatherMap) e as tarefas pendentes do meu banco de tarefas no Notion numa única mensagem no Telegram.

```
☀️ Bom dia! Hoje é terça-feira, 24 de novembro.

🌡️ Tempo em São Paulo: 22°C, nublado
   mín 18°C · máx 27°C · umidade 70%

📅 Agenda (2)
   • 09:30 — Academia
   • 14:00 — Dentista (Rua Exemplo, 123)

✅ Tarefas (3) · ⚠️ 1 atrasada(s)
   ▶️ Responder a mensagem de um amigo
      🔴 Urgent · ⭐ Important · 🧩 QUICK WIN · ⚠️ venceu 15/11
   • Pagar a conta de luz
      ⭐ Important · 🧩 EASY · ⏰ vence hoje
   ▶️ Organizar as fotos da viagem
      🧩 LONG TERM · 📅 até 05/12
```

Cada título é um link para a página da tarefa no Notion, e ▶️ marca o que já está em andamento.

## Como meu dia ficou melhor

Eu começava o dia abrindo três apps diferentes ainda na cama: agenda, previsão do tempo e lista de tarefas. Às vezes pulava algum e só lembrava do compromisso das 9h às 8h50. Agora leio uma mensagem só enquanto tomo café e já sei se preciso de guarda-chuva, que horas tenho compromisso e o que ficou atrasado de ontem. Parece pouco, mas o início do dia ficou bem mais tranquilo e parei de ser pego de surpresa pela agenda.

## Como funciona

1. **Schedule Trigger**: dispara todo dia às 7h (fuso `America/Sao_Paulo`).
2. **Configuração** (Set): guarda a cidade e o `chat_id`.
3. **Configuração do Notion** (Set): define qual banco usar, os nomes das colunas, os valores que significam "concluído", "em andamento", "urgente" e "importante", e os valores padrão (veja abaixo).
4. **Google Calendar**: busca os eventos entre hoje 00h e amanhã 00h.
5. **Aggregate**: junta os eventos num único item.
6. **HTTP Request**: consulta a API do OpenWeatherMap (`/data/2.5/weather`, em °C e português). Se falhar, o fluxo segue sem o clima.
7. **Notion**: busca no banco as tarefas que **não** estão concluídas e que têm prazo até hoje **ou** são urgentes. O filtro é montado automaticamente a partir da configuração.
8. **Aggregate**: junta as tarefas num único item.
9. **Code**: monta a mensagem em HTML do Telegram. Os eventos saem ordenados por horário. As tarefas aparecem nesta ordem: atrasadas, urgentes, importantes e depois pelo prazo mais próximo, cada uma com urgência, relevância, complexidade e situação do prazo.
10. **Telegram**: envia a mensagem.

## O banco de tarefas no Notion

O workflow vem configurado para um banco com estas colunas:

| Coluna | Tipo no Notion | Valores |
|---|---|---|
| `Task Name` | Título | |
| `Status` | Status | `To Do` · `Doing` · `Complete` · `Archieved` |
| `Deadline` | Data (pode ser um intervalo) | o prazo é a data final, ou a inicial se não houver final |
| `Urgency` | Select | `Urgent` · … |
| `Relevance` | Select | `Important` · … |
| `Complexity` | Select | `QUICK WIN` · `EASY` · `MEDIUM` · `HARD` · `VERY HARD` · `LONG TERM` |

Se o seu banco for diferente, é só ajustar o nó **Configuração do Notion**. Não precisa mexer no filtro nem no código.

| Campo | Para que serve | Valor de exemplo |
|---|---|---|
| `banco_id` | Qual banco (tabela) ler | `ID_DO_BANCO_NOTION` |
| `colunas` | Nome de cada coluna no seu banco. Use `""` para ignorar uma coluna que você não tem | `{"titulo": "Task Name", "status": "Status", "prazo": "Deadline", "urgencia": "Urgency", "relevancia": "Relevance", "complexidade": "Complexity"}` |
| `status_concluidos` | Status que **não** entram no resumo | `["Complete", "Archieved"]` |
| `status_em_andamento` | Status marcado com ▶️ | `Doing` |
| `valor_urgente` | Opção de `urgencia` que conta como urgente | `Urgent` |
| `valor_importante` | Opção de `relevancia` que conta como importante | `Important` |
| `valores_padrao` | Valor usado quando a coluna da tarefa está **vazia** | `{"status": "To Do", "urgencia": "", "relevancia": "", "complexidade": ""}` |
| `max_tarefas` | Quantas tarefas mostrar, no máximo | `15` |

**Como funcionam os valores padrão:** se uma tarefa não tem `Complexity` preenchida e `valores_padrao.complexidade` é `"MEDIUM"`, ela aparece como `🧩 MEDIUM`. Se `valores_padrao.urgencia` for igual a `valor_urgente` (ex.: `"Urgent"`), tarefas sem urgência preenchida também são tratadas como urgentes e passam a entrar no resumo. Deixe `""` para não aplicar nenhum padrão.

> Os nomes precisam ser **exatamente** iguais aos do Notion, com maiúsculas e grafia iguais (por exemplo, `Archieved`).

## Pré-requisitos

- Conta Google com Google Calendar (credencial OAuth2 no n8n)
- Conta gratuita no [OpenWeatherMap](https://openweathermap.org/api) com uma API key
- Um banco de tarefas no Notion e uma [integração interna](https://www.notion.so/my-integrations) com acesso a ele
- Um bot do Telegram (criado pelo [@BotFather](https://t.me/BotFather)) e o seu chat ID

## Como importar e configurar

1. Importe o `workflow.json` no n8n (veja o [passo a passo no README principal](../README.md#como-importar-um-workflow-no-n8n)).
2. Crie as credenciais:
   | Credencial | Tipo no n8n | Observação |
   |---|---|---|
   | `Google Calendar account` | Google Calendar OAuth2 API | |
   | `OpenWeatherMap API` | **Query Auth** | Name: `appid` · Value: sua API key |
   | `Notion account` | Notion API | token da integração interna |
   | `Telegram Bot` | Telegram API | token do BotFather |
3. No Notion, abra o banco de tarefas → **⋯ → Conexões** e adicione a sua integração.
4. Substitua os placeholders:
   | Onde | Placeholder | Trocar por |
   |---|---|---|
   | Nó **Configuração** | `SEU_CHAT_ID` | seu chat ID do Telegram |
   | Nó **Configuração** | `Sao Paulo,BR` | sua cidade no formato `Cidade,PAÍS` |
   | Nó **Configuração do Notion** | `ID_DO_BANCO_NOTION` | ID do banco (o trecho de 32 caracteres na URL) |
   | Nó **Configuração do Notion** | `colunas`, status e valores | os nomes do seu banco, se forem diferentes |
   | Nó **Eventos de hoje** | `primary` | opcional: ID de outra agenda |
5. Execute manualmente para testar e ative o workflow.

## Ideias de personalização

- Trocar o Notion pelo nó do **Todoist** (*Get Many* com o filtro `today | overdue`) se você usa o Todoist.
- Incluir também as tarefas da semana que vem numa seção "No radar".
- Usar o endpoint `/forecast` para mostrar a chance de chuva ao longo do dia.
- Incluir mais de uma agenda (ex.: pessoal + família) duplicando o nó do Calendar.
- Mandar uma versão mais curta no fim de semana, ou não mandar nada.
- Acrescentar a cotação do dólar ou as manchetes do dia com mais um HTTP Request.
