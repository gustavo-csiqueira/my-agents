# 📰 06 · Digest RSS semanal

## O que faz

Todo **domingo às 9h** lê uma lista de feeds RSS, pega os artigos publicados nos últimos 7 dias, remove os repetidos, pede a um LLM um resumo de 2–3 frases em português para cada um e agrupa tudo por tema. O resultado chega como um e-mail em HTML bem formatado e também numa versão curta no Telegram, só com os títulos e links.

Se algum feed estiver fora do ar ou quebrado, o digest sai do mesmo jeito, com um aviso no final dizendo qual feed falhou.

```
📰 Digest semanal · 17 de nov a 24 de nov
23 artigos

Inteligência Artificial
• Novo modelo open source supera…
• Como avaliar agentes em produção

Programação
• O que muda no Python 3.15
…
⚠️ 1 feed(s) com erro: Blog X
```

## Como meu dia ficou melhor

Eu assinava uns vinte blogs e newsletters e vivia com dezenas de abas abertas "para ler depois". Na prática eu não lia quase nada e ainda ficava com a sensação de estar sempre atrasado. Agora deixo os feeds trabalharem durante a semana e no domingo de manhã recebo um e-mail só, organizado por assunto, com um resumo curto de cada artigo. Em uns 15 minutos de café eu sei o que aconteceu e abro só os dois ou três que realmente valem a leitura completa. Desinscrevi de quase todas as newsletters e a caixa de entrada agradeceu. Dos agentes daqui, é o que eu mais uso.

## Como funciona

```
Domingo 9h → Configuração → Separar feeds → Ler feed RSS → Normalizar itens
   → Só da última semana → Remover duplicados → Mais recentes primeiro → Limitar quantidade
   → Resumir artigo (LLM + parser JSON) → Juntar resumo e artigo → Montar digest
   → Enviar por e-mail (HTML) + Enviar no Telegram
```

1. **Schedule Trigger**: domingo às 9h (`America/Sao_Paulo`).
2. **Configuração** (Set): tudo que você precisa editar fica aqui:
   - `feeds`: lista de `{ "nome": "...", "url": "..." }`
   - `temas`: temas usados para agrupar (o LLM escolhe um por artigo)
   - `dias`: janela de tempo (padrão 7)
   - `max_artigos`: teto de artigos por digest, para controlar o custo do LLM (padrão 40)
   - `email_destino` e `chat_id`
3. **Split Out**: transforma a lista de feeds em um item por feed.
4. **RSS Read**: lê cada feed separadamente, com 2 tentativas. Está configurado com **On Error → Continue**: um feed com problema vira um item com o campo `error` em vez de interromper a execução.
5. **Normalizar itens** (Set): padroniza `titulo`, `link` (sem parâmetros `utm_`), `data`, `conteudo` e `fonte`, e registra o erro quando houver.
6. **Filter**: descarta itens com erro, sem link ou mais antigos que `dias`.
7. **Remove Duplicates**: remove artigos com o mesmo link (comum quando dois feeds publicam a mesma coisa).
8. **Sort + Limit**: mais recentes primeiro, até `max_artigos`.
9. **Basic LLM Chain + Structured Output Parser**: para cada artigo, o modelo devolve um JSON `{ titulo_pt, resumo, tema }`. O resumo tem 2 a 3 frases em português, o título é traduzido quando necessário e o tema vem da lista configurada. Tem 3 tentativas e, se mesmo assim falhar, o artigo não se perde (veja o próximo passo).
10. **Juntar resumo e artigo** (Set): combina a resposta da IA com os dados originais. Se a IA falhou ou inventou um tema fora da lista, usa um trecho do conteúdo original e o tema "Outros".
11. **Montar digest** (Code): agrupa por tema na ordem da lista, gera o HTML do e-mail (com estilos inline, compatível com Gmail/Outlook), gera a versão curta do Telegram respeitando o limite de 4.096 caracteres e adiciona o aviso dos feeds que falharam.
12. **Gmail** e **Telegram**: enviam as duas versões. Se quiser só uma, desative o outro nó.

### Tratamento de falhas, em resumo

| Problema | O que acontece |
|---|---|
| Feed fora do ar / URL errada / XML inválido | 2 tentativas; se continuar falhando, o feed é ignorado e listado como "com problema" no fim do digest |
| Artigo sem data ou sem link | Descartado no filtro |
| Mesmo artigo em dois feeds | Fica só um |
| LLM falha ou retorna algo inválido | 3 tentativas; depois usa um trecho do texto original como resumo |
| LLM escolhe um tema que não está na lista | Vai para "Outros" |
| Nenhum artigo novo na semana | Nada é enviado |

## Pré-requisitos

- Chave de API da [Anthropic](https://console.anthropic.com/) (dá para usar outro provedor, veja as ideias abaixo)
- Conta Google com Gmail, para enviar o e-mail
- Bot do Telegram e o seu chat ID (opcional, se quiser a versão no Telegram)
- As URLs dos feeds RSS/Atom que você acompanha (a maioria dos blogs tem `/feed`, `/rss` ou `/atom.xml`)

**Modelo e custo:** o padrão é o Claude Haiku 5.5 (`claude-haiku-5-5`), rápido e barato. Com o teto de 40 artigos, um digest custa frações de centavo de dólar. Se quiser resumos mais caprichados, troque o ID no nó **Anthropic Chat Model** por `claude-opus-5-5` (Claude Opus 5.5).

## Como importar e configurar

1. Importe o `workflow.json` no n8n.
2. Crie as credenciais:
   | Credencial | Tipo no n8n |
   |---|---|
   | `Anthropic account` | Anthropic API |
   | `Gmail account` | Gmail OAuth2 API |
   | `Telegram Bot` | Telegram API |
3. No nó **Configuração**, substitua os placeholders:
   | Campo | Placeholder | Trocar por |
   |---|---|---|
   | `feeds` | `SEU_FEED_RSS_1`, `SEU_FEED_RSS_2`, `SEU_FEED_RSS_3` | URLs dos seus feeds (adicione ou remova itens à vontade) |
   | `email_destino` | `SEU_EMAIL@exemplo.com` | e-mail que vai receber o digest |
   | `chat_id` | `SEU_CHAT_ID` | seu chat ID do Telegram |
   | `temas` | lista de exemplo | os temas que fazem sentido para você |
4. Rode manualmente uma vez para conferir o resultado. Uma dica: no primeiro teste, deixe `max_artigos` em 5.
5. Ative o workflow.

### Exemplo de lista de feeds

```json
[
  { "nome": "Blog X",            "url": "https://exemplo.com/feed" },
  { "nome": "Newsletter Y",      "url": "https://exemplo.substack.com/feed" },
  { "nome": "Canal Z (YouTube)", "url": "https://www.youtube.com/feeds/videos.xml?channel_id=ID_DO_CANAL" }
]
```

### Preferindo uma planilha para a lista de feeds?

Troque o nó **Separar feeds** por um **Google Sheets → Get Row(s)** lendo uma aba com as colunas `nome` e `url`. Como cada linha já vira um item, o resto do fluxo funciona sem mudanças. Só ajuste as referências `$('Separar feeds')` no nó **Normalizar itens** para o nome do novo nó.

## Ideias de personalização

- Usar outro provedor (ex.: *OpenAI Chat Model*): apague o nó **Anthropic Chat Model** e conecte o novo na entrada *Model* do **Resumir artigo**. Não preencha `temperature`, `top_p` ou `top_k` no nó da Anthropic, porque os modelos Claude atuais recusam valores customizados nesses parâmetros.
- Pedir ao LLM uma nota de relevância de 1 a 5 e mostrar só os artigos 4+ (ou destacar o "artigo da semana" no topo).
- Guardar os artigos numa planilha ou no Notion para ter um histórico pesquisável.
- Criar um digest diário mais curto com `dias: 1` e outro schedule.
- Adicionar feeds de canais do YouTube ou de subreddits (`https://www.reddit.com/r/NOME/.rss`).
- Mandar para mais de uma pessoa, cada uma com a sua lista de temas.
