# 📧➡️✅ 02 · E-mail para tarefa

## O que faz

Quando aplico a etiqueta **fazer** num e-mail do Gmail, ele vira uma tarefa no Notion: o assunto vira o título, e o remetente, a data, um trecho do texto e o link para abrir o e-mail vão no corpo. Depois a etiqueta é trocada por **processado**.

## Como meu dia ficou melhor

Minha caixa de entrada funcionava como lista de tarefas, o que é péssimo: e-mail importante ia descendo, ficava esquecido e eu só lembrava quando alguém cobrava. Também deixava e-mails marcados como não lidos para "lembrar depois", e eles se misturavam com o resto. Agora, quando bato o olho num e-mail que pede alguma ação, só aplico a etiqueta e arquivo. Em poucos minutos ele aparece no Notion junto com o resto das minhas tarefas, com o link para voltar ao e-mail se precisar. A caixa de entrada ficou mais limpa e parei de me perder no meio do caminho.

## Como funciona

1. **Gmail Trigger**: a cada 5 minutos verifica e-mails com a etiqueta `fazer`.
2. **Preparar dados** (Set): extrai título, remetente, data, um trecho do texto e monta o link `https://mail.google.com/mail/u/0/#all/<id>`.
3. **Notion**: cria uma página no banco de tarefas com o assunto como título e as informações como parágrafos.
4. **Gmail – Remover etiqueta**: tira a etiqueta `fazer`.
5. **Gmail – Adicionar etiqueta**: aplica a etiqueta `processado`, para eu saber que já virou tarefa.

## Pré-requisitos

- Conta Google com Gmail (credencial OAuth2 no n8n)
- Duas etiquetas criadas no Gmail: `fazer` e `processado`
- Um banco de dados (database) de tarefas no Notion
- Uma [integração interna do Notion](https://www.notion.so/my-integrations) com acesso a esse banco

## Como importar e configurar

1. Importe o `workflow.json` no n8n.
2. Crie as credenciais:
   | Credencial | Tipo no n8n |
   |---|---|
   | `Gmail account` | Gmail OAuth2 API |
   | `Notion account` | Notion API (token da integração interna) |
3. No Notion, abra o banco de tarefas → **⋯ → Conexões** e adicione a sua integração.
4. Substitua os placeholders:
   | Onde | Placeholder | Trocar por |
   |---|---|---|
   | **E-mail com etiqueta fazer** (filtro *Label Names or IDs*) | `ID_DA_ETIQUETA_FAZER` | selecione a etiqueta `fazer` na lista |
   | **Remover etiqueta fazer** | `ID_DA_ETIQUETA_FAZER` | selecione a etiqueta `fazer` |
   | **Marcar como processado** | `ID_DA_ETIQUETA_PROCESSADO` | selecione a etiqueta `processado` |
   | **Criar tarefa no Notion** | `ID_DO_BANCO_NOTION` | ID do banco (o trecho de 32 caracteres na URL) ou escolha na lista |
5. Se o seu banco tiver propriedades obrigatórias (ex.: *Status*), adicione-as em **Properties** no nó do Notion.
6. Teste aplicando a etiqueta `fazer` num e-mail e ative o workflow.

## Ideias de personalização

- Usar **Todoist** no lugar do Notion (nó Todoist → *Create Task*), colocando o link na descrição.
- Preencher uma data de vencimento automática (ex.: daqui a 2 dias úteis).
- Usar etiquetas diferentes para áreas diferentes (`fazer/casa`, `fazer/estudos`) e mandar para bancos diferentes com um nó **Switch**.
- Pedir a um LLM para reescrever o assunto como uma ação ("Responder um amigo sobre a viagem").
