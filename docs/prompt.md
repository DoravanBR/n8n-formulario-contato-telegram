# Prompt do desafio

## Contexto

Criar uma automação para registrar notificações de contato e enviar avisos pelo Telegram.

## Público

O autor do projeto e qualquer pessoa que queira implementar um formulário de contatos em uma página de portfólio.

## Ferramentas envolvidas

- Google Sheets
- Gmail
- Telegram
- n8n

## Fluxo

Receber a requisição enviada pelo formulário de contato, registrar o nome, o e-mail e a mensagem em uma planilha e enviar uma notificação pelo Telegram avisando sobre o recebimento de uma nova mensagem.

## Regras

- A planilha deve possuir uma aba para informações dos contatos, com nome e e-mail.
- A planilha deve possuir outra aba para as mensagens, com e-mail, mensagem e data.
- Contatos já cadastrados não devem ser registrados novamente.
- Mensagens novas de contatos existentes devem ser salvas como novos registros na aba de mensagens.
- O workflow deve enviar uma cópia por Gmail e uma notificação pelo Telegram.

## Resultado esperado

Um workflow que mantenha o cadastro de contatos sem duplicidades e preserve o histórico completo das mensagens recebidas.


