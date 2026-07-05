---
name: whatsapp-visto-azul
description: Enviar e automatizar WhatsApp pela API REST do Visto Azul (texto, mídia, PIX, enquete, grupos, campanhas em massa, contatos e webhook). Use sempre que o usuário quiser mandar mensagem no WhatsApp, cobrar via PIX, disparar campanha, criar grupo, validar números ou receber mensagens para um chatbot.
---

# WhatsApp via Visto Azul

Skill para operar o WhatsApp pela API REST do Visto Azul. Você fala só com esta API, usando a sua API key.

## Autenticação
- Base URL: `https://dashboard.vistoazul.com.br/api/v1`
- Header em toda chamada: `Authorization: Bearer SUA_API_KEY`
- Pegue a API key no painel: `https://dashboard.vistoazul.com.br` (em Instâncias). Ela é por conta.
- Com mais de um número, indique a instância no campo `instance` (endpoints diretos) ou no header `X-Instance` (endpoints de proxy `/wa`).

## Regras (anti-ban, importante)
- Aguarde de 3 a 5 segundos entre chamadas sequenciais.
- Personalize e varie o texto. Nunca mande a mesma mensagem idêntica para muita gente.
- Para volume alto, use `POST /v1/campaigns` (já traz rotação de número e delays aleatórios).
- Valide números com `POST /v1/wa/chat/check` antes de disparar.

## Endpoints

### Instâncias
- `GET /v1/instances` · lista os números conectados e o status de cada um.
- `POST /v1/instances` `{ "name": "vendas" }` · cria uma instância (devolve o QR code).
- `GET /v1/instances/{instance}/connect` · gera um novo QR para reconectar.

### Enviar mensagem de texto
- `POST /v1/messages/text` `{ "instance": "vendas", "number": "5511999999999", "text": "Olá!" }`

### Enviar mídia, PIX e enquete (proxy `/v1/wa`, use o header `X-Instance`)
- Mídia: `POST /v1/wa/send/media` `{ "number": "...", "type": "image", "file": "https://...jpg", "text": "legenda" }` (type: `image` | `video` | `document`)
- Cobrança PIX: `POST /v1/wa/send/request-payment` `{ "number": "...", "amount": 97.00, "pixKey": "sua-chave", "pixType": "email", "title": "Pedido", "text": "Segue seu PIX", "itemName": "Pedido" }`
- Enquete: `POST /v1/wa/send/menu` `{ "number": "...", "type": "poll", "text": "Qual horário?", "choices": ["Manhã","Tarde"], "selectableCount": 1 }`
- Validar quem tem WhatsApp: `POST /v1/wa/chat/check` `{ "numbers": ["5511999999999"] }`
- Criar grupo: `POST /v1/wa/group/create` `{ "name": "Turma 01", "participants": [] }`

### Campanhas (disparo em massa)
- `POST /v1/campaigns` `{ "instances": ["vendas"], "template": "{Oi|Olá} {{nome}}!", "recipients": [{"number":"5511999999999","vars":{"nome":"Ana"}}], "minDelayMs": 8000, "maxDelayMs": 40000, "scheduledForMs": 0 }`
  - `template` aceita spintax `{a|b}` e variáveis `{{nome}}`. `scheduledForMs` (epoch ms) agenda para o futuro.
- `GET /v1/campaigns/{id}` · status (`sent`, `failed`, `total`, `state`).

### Contatos (CRM leve)
- `GET /v1/contacts?tag=vip` · lista (filtra por tag).
- `POST /v1/contacts` `{ "number": "...", "name": "Ana", "tags": ["cliente","vip"] }`
- `POST /v1/contacts/import` `{ "text": "5511...,Ana\n5521...,Bruno", "tags": ["lista"] }`

### Receber mensagens (webhook, para chatbot / agente de IA)
- `POST /v1/instances/{instance}/webhook` `{ "url": "https://seu-sistema.com/webhook", "enabled": true }`
- A partir daí, cada mensagem recebida chega como POST na sua URL. Ideal para responder com IA (ex.: Claude).

## Respostas de erro
`401` API key ausente/inválida · `402` assinatura inativa · `403` sem permissão / limite do plano · `400` parâmetro faltando.

## Exemplo (curl)
```bash
curl -X POST https://dashboard.vistoazul.com.br/api/v1/messages/text \
  -H "Authorization: Bearer SUA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"instance":"vendas","number":"5511999999999","text":"Olá do Claude!"}'
```

Documentação completa e mais exemplos: https://vistoazul.com.br/docs
