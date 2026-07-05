# Claude WhatsApp Skill · Visto Azul API

> Envie e automatize o **WhatsApp** direto do **Claude** (Claude Code, agentes de IA) usando a **API REST do [Visto Azul](https://vistoazul.com.br)**. Sem taxa por mensagem da Meta, com **PIX nativo** e **anti-ban** de fábrica.

Uma skill enxuta que ensina o Claude (ou qualquer LLM/agente) a operar o WhatsApp pela API do Visto Azul: enviar texto, mídia e cobrança PIX, criar grupos, disparar campanhas, gerenciar contatos e receber mensagens por webhook.

**Palavras-chave:** WhatsApp API, Claude, Claude Code, n8n, automação de WhatsApp, chatbot WhatsApp, API WhatsApp Brasil, PIX no WhatsApp, webhook WhatsApp.

---

## O que dá pra fazer

- 📤 **Enviar** texto, imagem, vídeo, documento e enquete
- 💸 **Cobrar via PIX** direto na conversa
- 👥 **Criar e gerenciar grupos** (inclusive mencionar @todos)
- 📣 **Disparo em massa** com spintax e anti-ban
- 🗂️ **Contatos e tags** para segmentar
- 🔔 **Receber mensagens** por webhook (para chatbots e agentes de IA)

Tudo por uma API REST simples, ideal para plugar em **n8n, Make, Zapier, Claude Code** ou no seu sistema.

---

## Como usar

### 1. Pegue sua API key
Você precisa de uma conta no Visto Azul (a API key é o que autentica as chamadas):

1. Crie a conta em **https://dashboard.vistoazul.com.br/signup**
2. Escolha um plano e conecte seu número lendo o QR code
3. Copie sua **API key** no painel (em *Instâncias*)

> A skill é gratuita e de código aberto. Ela funciona com a sua API key do Visto Azul.

### 2. Instale a skill
Copie o arquivo [`SKILL.md`](./SKILL.md) para o contexto do seu agente:

- **Claude Code:** salve como `.claude/skills/whatsapp/SKILL.md` no seu projeto (ou cole o conteúdo no seu `CLAUDE.md`).
- **Outros agentes / n8n / código:** use o [`openapi.yaml`](./openapi.yaml) como referência da API.

Troque `SUA_API_KEY` pela sua chave.

### 3. Primeiro envio (exemplo)

```bash
curl -X POST https://dashboard.vistoazul.com.br/api/v1/messages/text \
  -H "Authorization: Bearer SUA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"instance":"vendas","number":"5511999999999","text":"Olá do Claude!"}'
```

---

## Arquivos

| Arquivo | O que é |
|---|---|
| [`SKILL.md`](./SKILL.md) | Instruções da skill (base URL, autenticação, endpoints, regras anti-ban). |
| [`openapi.yaml`](./openapi.yaml) | Spec OpenAPI 3.1 dos endpoints públicos do gateway. |
| [`LICENSE`](./LICENSE) | MIT. |

## Links

- 🌐 Site: **https://vistoazul.com.br**
- 📚 Documentação completa: **https://vistoazul.com.br/docs**
- ✍️ Blog (n8n, IA, PIX): **https://vistoazul.com.br/blog**

---

Feito com 💙 pela [Visto Azul](https://vistoazul.com.br). Contribuições e issues são bem-vindas.
