# 🤖 05 · Triagem de e-mails com IA

## O que faz

Cada e-mail novo que chega na caixa de entrada é classificado por um LLM em uma de cinco categorias (**urgente**, **pessoal**, **financeiro/cobrança**, **newsletter** ou **outros**) e recebe a etiqueta correspondente no Gmail. Se for urgente, também chega um alerta no Telegram com remetente, assunto e um trecho do texto.

## Como meu dia ficou melhor

Eu recebo muito e-mail que não precisa de atenção nenhuma (newsletter, promoção, notificação), e no meio disso às vezes vinha um boleto ou um pedido que precisava de resposta no mesmo dia. Filtros do Gmail resolviam parte, mas cada remetente novo exigia uma regra nova. Com a classificação por IA, as etiquetas se organizam sozinhas: abro "IA/Financeiro" uma vez por semana para pagar as contas, leio as newsletters quando tenho tempo e só sou interrompido pelo celular quando é realmente urgente. Ainda erra de vez em quando, mas bem menos do que eu imaginava, e ajustar as descrições das categorias resolve a maioria dos casos.

## Como funciona

1. **Gmail Trigger**: a cada 2 minutos busca e-mails novos na caixa de entrada (`INBOX`).
2. **Text Classifier** (nó de IA nativo do n8n): recebe remetente, assunto e os primeiros 2.500 caracteres do corpo e escolhe uma das categorias. Cada categoria tem uma descrição que orienta o modelo, e o nó tenta de novo automaticamente se a API falhar.
3. **Anthropic Chat Model**: o modelo usado pelo classificador (Claude Haiku 5.5, `claude-haiku-5-5`: rápido e barato, bom para classificar cada e-mail que chega).
4. **Gmail – Add Labels**: um nó por categoria aplica a etiqueta certa.
5. **Telegram**: só no ramo **urgente**, envia um alerta com link para abrir o e-mail.

## Pré-requisitos

- Conta Google com Gmail (credencial OAuth2 no n8n)
- Chave de API da [Anthropic](https://console.anthropic.com/) (dá para usar outro provedor, veja abaixo)
- Bot do Telegram e o seu chat ID
- Cinco etiquetas criadas no Gmail, por exemplo: `IA/Urgente`, `IA/Pessoal`, `IA/Financeiro`, `IA/Newsletter`, `IA/Outros`

> **Privacidade:** o conteúdo dos e-mails é enviado para o provedor do LLM. Se isso for um problema, troque o modelo por um local (ex.: nó *Ollama Chat Model*) ou envie só remetente e assunto.

## Como importar e configurar

1. Importe o `workflow.json` no n8n.
2. Crie as credenciais:
   | Credencial | Tipo no n8n |
   |---|---|
   | `Gmail account` | Gmail OAuth2 API |
   | `Anthropic account` | Anthropic API |
   | `Telegram Bot` | Telegram API |
3. Substitua os placeholders (nos nós do Gmail, abra o campo *Label Names or IDs* e selecione a etiqueta na lista):
   | Onde | Placeholder | Trocar por |
   |---|---|---|
   | **Etiqueta urgente** | `ID_ETIQUETA_URGENTE` | etiqueta `IA/Urgente` |
   | **Etiqueta pessoal** | `ID_ETIQUETA_PESSOAL` | etiqueta `IA/Pessoal` |
   | **Etiqueta financeiro** | `ID_ETIQUETA_FINANCEIRO` | etiqueta `IA/Financeiro` |
   | **Etiqueta newsletter** | `ID_ETIQUETA_NEWSLETTER` | etiqueta `IA/Newsletter` |
   | **Etiqueta outros** | `ID_ETIQUETA_OUTROS` | etiqueta `IA/Outros` |
   | **Avisar urgente no Telegram** | `SEU_CHAT_ID` | seu chat ID |
4. Revise as descrições das categorias no nó **Classificar e-mail**: é ali que você "ensina" o que é urgente para você.
5. Ative o workflow.

### Trocando o modelo

- **Para mais precisão:** se a classificação errar muito no seu caso, troque o ID `claude-haiku-5-5` por `claude-opus-5-5` (Claude Opus 5.5) no nó **Anthropic Chat Model**. Ele é mais capaz, mas bem mais caro, e o fluxo roda para *todo* e-mail que chega.
- **Outro provedor:** apague o nó **Anthropic Chat Model**, adicione outro nó de modelo (ex.: *OpenAI Chat Model*) e conecte-o na entrada *Model* do **Classificar e-mail**. O resto do fluxo não muda.
- Os modelos Claude atuais (Haiku 5.5 e Opus 5.5) não aceitam valores customizados de `temperature`, `top_p` e `top_k`. Se você mexer nas opções do nó, deixe esses campos em branco.

## Ideias de personalização

- Arquivar automaticamente as newsletters (tirar a etiqueta `INBOX`) para a caixa de entrada ficar só com o que importa.
- No ramo financeiro, extrair valor e vencimento do boleto e criar um lembrete no calendário.
- Criar uma lista de remetentes VIP que sempre caem em "urgente", sem passar pela IA.
- Gerar um rascunho de resposta para e-mails pessoais (nó Gmail → *Create Draft*).
- Mandar um resumo diário no Telegram com quantos e-mails chegaram em cada categoria.
