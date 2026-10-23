# Documentação do workflow

## 1. Webhook

O nó **Webhook** é o gatilho do workflow.

Configuração sugerida:

- método HTTP: `POST`;
- caminho: `contato-portfolio`;
- resposta: imediata;
- código de resposta: `200`.

Durante o desenvolvimento, deve-se utilizar a Test URL com a opção **Listen for Test Event**. Depois da publicação do workflow, o formulário deve usar a Production URL.

Dados esperados:

```json
{
  "nome": "Ana Souza",
  "email": "ana@email.com",
  "mensagem": "Gostaria de conversar sobre um projeto."
}
```

## 2. Edit Fields — Preparar contato

O nó **Edit Fields**, anteriormente chamado Set, prepara uma estrutura consistente para os próximos nós.

| Campo | Expressão |
|---|---|
| `nome` | `{{ $json.body.nome.trim() }}` |
| `email` | `{{ $json.body.email.trim().toLowerCase() }}` |
| `mensagem` | `{{ $json.body.mensagem.trim() }}` |
| `recebidoEm` | `{{ $now }}` |

A normalização impede que variações como `Pessoa@Email.com` e `pessoa@email.com` gerem contatos duplicados.

## 3. Google Sheets — Buscar contato

Esse nó consulta a aba `Contatos`.

- recurso: `Sheet Within Document`;
- operação: `Get Row(s)`;
- coluna do filtro: `email`;
- valor do filtro: `{{ $('Preparar contato').item.json.email }}`.

Nas configurações do nó, habilite **Always Output Data**. Assim, o fluxo continua quando a busca não encontra nenhuma linha.

## 4. IF — Contato existe?

O nó **IF** verifica se a busca retornou um e-mail.

- tipo: `String`;
- valor: `{{ $json.email }}`;
- operação: `is not empty`.

Interpretação das saídas:

- `true`: o contato já existe;
- `false`: o contato é novo.

## 5. Google Sheets — Cadastrar contato

Somente a saída `false` do IF passa por este nó.

- operação: `Append Row`;
- aba: `Contatos`.

| Coluna | Expressão |
|---|---|
| `nome` | `{{ $('Preparar contato').item.json.nome }}` |
| `email` | `{{ $('Preparar contato').item.json.email }}` |
| `cadastradoEm` | `{{ $('Preparar contato').item.json.recebidoEm }}` |

## 6. Google Sheets — Registrar mensagem

As duas saídas lógicas chegam a este passo:

```text
IF true ───────────────────→ Registrar mensagem
IF false → Cadastrar contato → Registrar mensagem
```

- operação: `Append Row`;
- aba: `Mensagens`.

| Coluna | Expressão |
|---|---|
| `email` | `{{ $('Preparar contato').item.json.email }}` |
| `mensagem` | `{{ $('Preparar contato').item.json.mensagem }}` |
| `recebidaEm` | `{{ $('Preparar contato').item.json.recebidoEm }}` |

Como a operação é Append Row, cada envio gera um registro próprio no histórico.

## 7. Gmail — Enviar e-mail

Use o nó **Gmail** com o recurso `Message` e a operação `Send`.

Assunto sugerido:

```text
Novo contato do portfólio: {{ $('Preparar contato').item.json.nome }}
```

Mensagem sugerida:

```text
Uma nova mensagem foi enviada pelo portfólio.

Nome: {{ $('Preparar contato').item.json.nome }}
E-mail: {{ $('Preparar contato').item.json.email }}
Data: {{ $('Preparar contato').item.json.recebidoEm }}

Mensagem:
{{ $('Preparar contato').item.json.mensagem }}
```

Quando disponível, configure **Send Replies To** com o e-mail do contato.

## 8. Telegram — Enviar notificação

Use o nó **Telegram** com o recurso `Message` e a operação `Send Message`.

Mensagem sugerida:

```text
📨 Nova mensagem no portfólio

👤 Nome: {{ $('Preparar contato').item.json.nome }}
✉️ E-mail: {{ $('Preparar contato').item.json.email }}
🕒 Data: {{ $('Preparar contato').item.json.recebidoEm }}

💬 Mensagem:
{{ $('Preparar contato').item.json.mensagem }}
```

O bot deve ser criado pelo BotFather e seu token deve ficar armazenado somente nas credenciais do n8n.

## Testes recomendados

1. Enviar uma mensagem com um e-mail ainda não cadastrado.
2. Confirmar que o contato apareceu na aba `Contatos`.
3. Confirmar que a mensagem apareceu na aba `Mensagens`.
4. Enviar uma segunda mensagem usando o mesmo e-mail.
5. Confirmar que não surgiu outro contato.
6. Confirmar que a segunda mensagem foi adicionada como uma nova linha.
7. Verificar o recebimento do e-mail e da notificação no Telegram.

## Segurança

Nunca exporte ou publique credenciais do Gmail, Google Sheets ou Telegram. Antes de publicar um workflow exportado, revise o arquivo para garantir que tokens, identificadores privados e dados pessoais foram removidos.


