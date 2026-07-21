---
name: whatsapp-visto-azul
description: Operar e automatizar o WhatsApp pela Visto Azul — por linguagem natural via o servidor MCP, ou pela API REST (texto, mídia, PIX, enquete, botões, grupos, campanhas em massa, contatos, LER conversas e webhook). Use sempre que o usuário quiser mandar mensagem no WhatsApp, cobrar via PIX, disparar campanha, ler conversas, criar grupo, validar números ou receber mensagens para um chatbot.
---

# WhatsApp via Visto Azul

Duas formas de operar o WhatsApp: pelo **servidor MCP** (mais fácil, linguagem natural) ou pela **API REST** (programático). As duas usam a sua conta Visto Azul.

## Opção 1 (recomendada): servidor MCP

O jeito mais rápido. Conecta a Visto Azul direto no seu assistente de IA e você opera o WhatsApp conversando: *"manda um PIX de R$97 pro João"*, *"o que o cliente respondeu?"*, *"dispara a campanha pra essa lista"*.

- **URL:** `https://mcp.vistoazul.com.br/mcp`
- **Claude Code:** `claude mcp add --transport http vistoazul https://mcp.vistoazul.com.br/mcp` (na 1ª vez abre o navegador pra login)
- **Claude Desktop / ChatGPT:** Configurações → Conectores → adicionar conector personalizado → cole a URL e faça login.
- **Cursor / Windsurf / VS Code:** no `mcp.json`, `"vistoazul": { "url": "https://mcp.vistoazul.com.br/mcp" }`
- Autenticação por **OAuth** (login com a sua conta Visto Azul). Sem API key manual.

**Ferramentas do MCP:** conectar número (QR), listar/ler conversas ao vivo, enviar texto / mídia / PIX / enquete-botões-menu, mostrar "digitando", reagir, marcar como lida, campanha (criar / listar / status / pausar / retomar / cancelar), contatos e funil (kanban), criar grupo, templates, catálogo e status da conta.

Guia completo: https://vistoazul.com.br/docs/mcp.html

## Opção 2: API REST

Você fala só com esta API, usando a sua API key.

### Autenticação
- Base URL: `https://dashboard.vistoazul.com.br/api/v1`
- Header em toda chamada: `Authorization: Bearer SUA_API_KEY`
- Pegue a API key no painel: `https://dashboard.vistoazul.com.br` (em Instâncias). Ela é por conta.
- Com mais de um número, indique a instância no campo `instance` (endpoints diretos) ou no header `X-Instance` (endpoints de proxy `/wa`).

### Regras (anti-ban, importante)
- Aguarde de 3 a 5 segundos entre chamadas sequenciais.
- Personalize e varie o texto. Nunca mande a mesma mensagem idêntica para muita gente.
- Para volume alto, use `POST /v1/campaigns` (já traz rotação de número e delays aleatórios).
- Valide números com `POST /v1/wa/chat/check` antes de disparar.

### Instâncias
- `GET /v1/instances` · lista os números conectados e o status de cada um.
- `POST /v1/instances` `{ "name": "vendas" }` · cria uma instância (devolve o QR code).
- `GET /v1/instances/{instance}/connect` · gera um novo QR para reconectar.
- `GET /v1/instances/{instance}/status` · status da conexão.

### Enviar mensagem de texto
- `POST /v1/messages/text` `{ "instance": "vendas", "number": "5511999999999", "text": "Olá!" }`

### Enviar enquete, botões e menu
- Enquete: `POST /v1/messages/menu` `{ "instance", "number", "type": "poll", "text": "Qual horário?", "choices": ["Manhã","Tarde"], "selectableCount": 1 }`
- Botões: `POST /v1/messages/menu` `{ "instance", "number", "type": "button", "text": "Confirma?", "choices": ["Sim","Não"] }`
- Menu/lista: `{ "type": "list", "buttonText": "Ver opções", "choices": ["Item 1","Item 2"] }`
- **Aviso:** a enquete aparece para todo mundo. Botão e lista podem não renderizar em alguns aparelhos (limite do WhatsApp). Quando precisar de garantia, use enquete ou um menu numerado em texto.

### Enviar mídia e PIX (proxy `/v1/wa`, use o header `X-Instance`)
- Mídia: `POST /v1/wa/send/media` `{ "number": "...", "type": "image", "file": "https://...jpg", "text": "legenda" }` (type: `image` | `video` | `document`)
- Cobrança PIX: `POST /v1/wa/send/request-payment` `{ "number": "...", "amount": 97.00, "pixKey": "sua-chave", "pixType": "email", "title": "Pedido", "text": "Segue seu PIX", "itemName": "Pedido" }`
- Validar quem tem WhatsApp: `POST /v1/wa/chat/check` `{ "numbers": ["5511999999999"] }`
- Criar grupo: `POST /v1/wa/group/create` `{ "name": "Turma 01", "participants": [] }`

### Ler conversas (histórico, leitura ao vivo)
- Listar conversas recentes: `GET /v1/instances/{instance}/chats` → nome, telefone, não lidas, última mensagem.
- Ler mensagens de uma conversa: `GET /v1/instances/{instance}/chats/{chatid}/messages` (chatid = o número, ou o id de conversa vindo de `/chats`).
- Nada das conversas é armazenado nos nossos servidores; a leitura é ao vivo, na hora do pedido.

### Campanhas (disparo em massa)
- `POST /v1/campaigns` `{ "instances": ["vendas"], "template": "{Oi|Olá} {{nome}}!", "recipients": [{"number":"5511999999999","vars":{"nome":"Ana"}}], "minDelayMs": 8000, "maxDelayMs": 40000, "scheduledForMs": 0 }`
  - `template` aceita spintax `{a|b}` e variáveis `{{nome}}`. `scheduledForMs` (epoch ms) agenda para o futuro.
- `GET /v1/campaigns/{id}` · status (`sent`, `failed`, `total`, `state`).
- Controlar: `POST /v1/campaigns/{id}/pause` · `/resume` · `/cancel`.

### Contatos (CRM leve)
- `GET /v1/contacts?tag=vip` · lista (filtra por tag).
- `POST /v1/contacts` `{ "number": "...", "name": "Ana", "tags": ["cliente","vip"] }`
- `POST /v1/contacts/import` `{ "text": "5511...,Ana\n5521...,Bruno", "tags": ["lista"] }`

### Receber mensagens (webhook, para chatbot / agente de IA)
- `POST /v1/instances/{instance}/webhook` `{ "url": "https://seu-sistema.com/webhook", "enabled": true }`
- A partir daí, cada mensagem recebida chega como POST na sua URL. Ideal para responder com IA (ex.: Claude).

### Atendimento no Chatwoot
- Ligue seu Chatwoot para ler e responder as conversas por lá: `POST /v1/instances/{instance}/chatwoot/connect` `{ "baseUrl": "https://seu-chatwoot", "apiToken": "..." }`. Ou pelo painel, em Integrações. A Visto Azul cria o inbox e o webhook sozinha.

### Baixar mídia recebida e ações na conversa
O `id` da mensagem vem no webhook. Todas levam `{ "instance", … }` no corpo.
- Baixar mídia: `POST /v1/messages/media` `{ "instance", "id", "transcribe": true }` → `{ "base64", "mimetype", "transcription" }` (áudio pode vir transcrito).
- Marcar como lida: `POST /v1/messages/read` `{ "instance", "id" }`.
- Presença: `POST /v1/messages/presence` `{ "instance", "number", "presence": "composing" }` (composing/recording/paused).
- Reagir: `POST /v1/messages/react` `{ "instance", "number", "id", "emoji": "..." }`.
- Editar: `POST /v1/messages/edit` `{ "instance", "id", "text" }`.
- Apagar: `POST /v1/messages/delete` `{ "instance", "id" }`.

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
