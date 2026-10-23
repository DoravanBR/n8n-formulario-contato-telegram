# Automação de contatos com n8n

Projeto desenvolvido como desafio de automação para receber mensagens de um formulário de portfólio, organizar contatos e mensagens no Google Sheets e enviar notificações por Gmail e Telegram.

## Objetivo

Construir um workflow no n8n capaz de:

- receber os dados enviados por um formulário de contato;
- cadastrar cada contato apenas uma vez;
- armazenar todas as mensagens recebidas em um histórico separado;
- enviar uma cópia da mensagem por e-mail;
- avisar pelo Telegram quando uma nova mensagem chegar.

## Tecnologias utilizadas

- n8n
- Google Sheets
- Gmail
- Telegram Bot API
- Webhooks HTTP

## Arquitetura do workflow

```text
Webhook
   ↓
Edit Fields — normalização dos dados
   ↓
Google Sheets — busca do contato pelo e-mail
   ↓
IF — o contato já existe?
   ├─ Não → Google Sheets — cadastra o contato ─┐
   └─ Sim ──────────────────────────────────────┤
                                                ↓
                               Google Sheets — registra a mensagem
                                                ↓
                                       Gmail — envia o e-mail
                                                ↓
                               Telegram — envia a notificação
```

## Organização da planilha

### Aba `Contatos`

| nome | email | cadastradoEm |
|---|---|---|
| Ana Souza | ana@email.com | 2026-10-23T10:00:00-03:00 |

Cada endereço de e-mail aparece apenas uma vez nessa aba.

### Aba `Mensagens`

| email | mensagem | recebidaEm |
|---|---|---|
| ana@email.com | Gostaria de conhecer seu trabalho. | 2026-10-23T10:00:00-03:00 |
| ana@email.com | Podemos marcar uma conversa? | 2026-10-24T14:30:00-03:00 |

Cada envio do formulário cria uma nova linha. Dessa forma, contatos não são duplicados e o histórico de mensagens é preservado.

## Funcionamento

1. O formulário envia `nome`, `email` e `mensagem` por uma requisição `POST`.
2. O Webhook inicia o workflow.
3. O nó Edit Fields remove espaços extras e converte o e-mail para letras minúsculas.
4. O Google Sheets procura o e-mail na aba `Contatos`.
5. O nó IF identifica se o contato já existe.
6. Contatos novos são adicionados à aba `Contatos`.
7. Toda mensagem, nova ou recorrente, é adicionada à aba `Mensagens`.
8. O Gmail envia uma cópia da mensagem para o responsável pelo portfólio.
9. O Telegram envia uma notificação de nova mensagem.

O detalhamento de cada nó está em [docs/workflow.md](docs/workflow.md). O enunciado utilizado para planejar a solução está em [docs/prompt.md](docs/prompt.md).

## Exemplo de requisição

```json
{
  "nome": "Ana Souza",
  "email": "ana@email.com",
  "mensagem": "Gostaria de conversar sobre um projeto."
}
```

## Cuidados de segurança

- Não publique tokens, credenciais ou IDs privados no repositório.
- Armazene as credenciais usando o gerenciador de credenciais do n8n.
- Valide os campos recebidos antes de gravá-los.
- Em um projeto público, proteja o formulário contra spam e abuso.
- Use a URL de produção do Webhook apenas depois de publicar o workflow.

## Possíveis melhorias

- resposta automática para o contato;
- validação mais rigorosa do endereço de e-mail;
- proteção com CAPTCHA ou rate limit;
- tratamento de erros com Error Trigger;
- painel para acompanhar quantidade de contatos e mensagens;
- exportação segura do workflow do n8n, sem credenciais.

## Autor

Desenvolvido por [DoravanBR](https://github.com/DoravanBR) como projeto de aprendizagem em automação com n8n.


