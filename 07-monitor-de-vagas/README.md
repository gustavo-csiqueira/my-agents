# 🔎 07 · Monitor de vagas

## O que faz

A cada 6 horas busca vagas em feeds RSS de sites de emprego e na API pública da [Remotive](https://remotive.com/), filtra pelas palavras-chave que me interessam e manda no Telegram **só as vagas que eu ainda não vi**. As vagas já avisadas ficam guardadas no próprio n8n, então nada se repete.

```
🔔 3 vaga(s) nova(s)

💼 Senior Python Developer
🏢 Empresa Exemplo · Worldwide
🔗 ver vaga (Remotive)

💼 Automation Engineer (n8n)
🏢 Outra Empresa
🔗 ver vaga (We Work Remotely)
```

## Como meu dia ficou melhor

Quando estava de olho no mercado, eu entrava nos mesmos sites todo dia, digitava as mesmas buscas e passava boa parte do tempo relendo vagas que já tinha visto. Era cansativo e fácil de largar no meio. Com o monitor, recebo umas poucas mensagens por dia só com novidades e consigo me candidatar logo nas primeiras horas, quando a vaga ainda tem poucos candidatos. Parei de gastar tempo procurando e passei a gastar com o que interessa: currículo e carta de apresentação.

## Como funciona

1. **Schedule Trigger**: a cada 6 horas.
2. **Configuração** (Set): palavras-chave, lista de feeds, termo de busca da Remotive, chat ID e o máximo de vagas por alerta.
3. Duas fontes em paralelo, cada uma resistente a falhas (com **On Error → Continue**, uma fonte fora do ar não derruba a outra):
   - **RSS**: *Split Out* da lista de feeds → *RSS Read* → *Set* para padronizar os campos.
   - **API Remotive**: *HTTP Request* para `https://remotive.com/api/remote-jobs?search=...` → *Split Out* das vagas → *Set* para padronizar.
4. **Merge**: junta as duas fontes no mesmo formato (`id`, `titulo`, `empresa`, `local`, `link`, `descricao`, `fonte`).
5. **Filter**: mantém só vagas cujo título ou descrição contém alguma palavra-chave.
6. **Remove Duplicates** (entrada atual): tira repetidas dentro da mesma execução.
7. **Remove Duplicates** (*Remove Items Processed in Previous Executions*): compara o `id` com os IDs guardados nas execuções anteriores e deixa passar só as vagas novas. É o armazenamento de dados do próprio n8n, com escopo do workflow e até 10.000 IDs.
8. **Limit**: no máximo `max_por_alerta` vagas por mensagem (padrão 15).
9. **Set + Aggregate**: formata cada vaga e junta tudo numa mensagem.
10. **Telegram**: envia o alerta. Se não houver vaga nova, nada é enviado.

> **Primeira execução:** como nenhuma vaga foi vista ainda, todas são "novas". O alerta vem limitado a 15 e as demais ficam marcadas como vistas. A partir da segunda execução chegam só as novidades.

## Pré-requisitos

- Bot do Telegram e o seu chat ID
- URLs de feeds RSS de vagas. Muitos sites e agregadores oferecem, como o We Work Remotely (já incluído como exemplo) ou alertas de busca de outros portais
- Nenhuma chave é necessária para a API da Remotive.

## Como importar e configurar

1. Importe o `workflow.json` no n8n.
2. Crie a credencial `Telegram Bot` (Telegram API).
3. No nó **Configuração**, substitua os placeholders:
   | Campo | Placeholder | Trocar por |
   |---|---|---|
   | `feeds` | `SEU_FEED_RSS` | URL de um feed de vagas (ou remova o item) |
   | `palavras_chave` | lista de exemplo | termos que te interessam (sem diferenciar maiúsculas) |
   | `busca_remotive` | `python` | termo de busca na Remotive |
   | `chat_id` | `SEU_CHAT_ID` | seu chat ID |
4. Rode manualmente uma vez (isso já "aprende" as vagas atuais) e ative o workflow.

Para recomeçar do zero, crie temporariamente um nó **Remove Duplicates** com a operação *Clear Deduplication History* e escopo *Workflow*, execute-o uma vez e depois apague.

## Ideias de personalização

- Guardar as vagas vistas no **Google Sheets** em vez da memória do n8n. Assim você tem um histórico e pode marcar "candidatei / descartei".
- Usar um LLM para dar uma nota de aderência a cada vaga comparando com o seu currículo e mandar só as melhores.
- Palavras de exclusão (ex.: "estágio", "senior" ou "pleno", conforme o seu nível).
- Filtrar por localização ou faixa salarial quando a fonte informar.
- Adicionar outras APIs públicas de vagas com mais ramos de HTTP Request ligados ao Merge.
