<p align="center">
  <img src="assets/banner.png" alt="FluvPay" width="780">
</p>

<p align="center">
  API de pagamentos PIX para desenvolvedores. Cobranças, saques, transferências internas e webhooks por uma API REST simples e previsível.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/API-v1-34d399?style=flat-square&labelColor=0a0a0a" alt="API v1">
  <img src="https://img.shields.io/badge/OpenAPI-3.1-0a0a0a?style=flat-square&logo=openapiinitiative&logoColor=34d399" alt="OpenAPI 3.1">
  <img src="https://img.shields.io/badge/PIX-instant%C3%A2neo-34d399?style=flat-square&labelColor=0a0a0a" alt="PIX instantâneo">
  <img src="https://img.shields.io/badge/licen%C3%A7a-MIT-0a0a0a?style=flat-square" alt="Licença MIT">
  <a href="https://discord.gg/5rpHRhTxJB"><img src="https://img.shields.io/badge/Discord-entrar-5865F2?style=flat-square&logo=discord&logoColor=white" alt="Discord"></a>
</p>

<p align="center">
  <a href="https://fluvpay.com">Site</a>
  ·
  <a href="https://docs.fluvpay.com">Documentação</a>
  ·
  <a href="openapi.yaml">Especificação</a>
  ·
  <a href="https://discord.gg/5rpHRhTxJB">Comunidade</a>
  ·
  <a href="https://docs.fluvpay.com">Suporte</a>
</p>

---

## Sobre

A **FluvPay** é um gateway de pagamentos PIX voltado para desenvolvedores. Este repositório
contém a **especificação OpenAPI 3.1 pública** da API (a fonte de verdade do contrato) e é a
base para gerar clientes, importar coleções e validar integrações.

Estamos na **v1**. Toda a API vive sob o prefixo `/api/v1`, e novos recursos entram aqui,
sempre na v1, de forma retrocompatível.

- **Base URL:** `https://api.fluvpay.com/api/v1`
- **Especificação:** [`openapi.yaml`](openapi.yaml) e [`openapi.json`](openapi.json)
- **Documentação:** https://docs.fluvpay.com

## Recursos

| Recurso | O que faz |
|---|---|
| **Cobranças** | Cria, consulta e lista cobranças PIX com QR Code e copia-e-cola |
| **Transações** | Extrato financeiro consolidado de entradas e saídas |
| **Saques** | Envia PIX da sua conta para uma chave PIX |
| **Transferências internas** | Move saldo entre contas FluvPay |
| **Webhooks** | Eventos assinados (HMAC) para você reagir a pagamentos em tempo real |
| **Sandbox** | Ambiente de teste completo, sem mover dinheiro de verdade |

## Início rápido

Toda chamada usa sua API Key no header `Authorization`. O ambiente é definido pelo prefixo
da chave: `fluv_live_` para produção e `fluv_test_` para o sandbox.

Criar uma cobrança PIX:

```bash
curl -X POST https://api.fluvpay.com/api/v1/charges/ \
  -H "Authorization: Bearer fluv_test_sua_chave" \
  -H "Idempotency-Key: 4f1a9c2e-1b7d-4a3e-9b2c-7e6f5d4c3b2a" \
  -H "Content-Type: application/json" \
  -d '{
    "amount_cents": 4990,
    "description": "Pedido #1042",
    "customer": { "name": "Maria Souza", "email": "maria@exemplo.com" }
  }'
```

Resposta:

```json
{
  "id": "chg_01J9X8K2P3Q4R5S6T7U8V9W0XY",
  "amount_cents": 4990,
  "currency": "BRL",
  "status": "pending",
  "payment_method": "pix",
  "pix_qr_code": "data:image/png;base64,iVBORw0KGgo...",
  "pix_copy_paste": "00020126580014BR.GOV.BCB.PIX...",
  "net_amount_cents": 4965,
  "created_at": "2026-06-08T12:00:00Z"
}
```

Mostre o `pix_copy_paste` ou renderize o `pix_qr_code`, e aguarde o evento `charge.paid`
no seu webhook para confirmar o pagamento.

## Autenticação e escopos

```
Authorization: Bearer fluv_live_sua_chave
```

As chaves possuem escopos, e cada endpoint exige o seu:

| Escopo | Permite |
|---|---|
| `payments.create` | Criar cobranças |
| `payments.read` | Ler e listar cobranças |
| `withdrawals.create` | Criar saques e transferências internas |
| `withdrawals.read` | Ler e listar saques |
| `transfers.read` | Ler e listar transferências internas |

## Idempotência

As operações de escrita (criar cobrança, saque e transferência) aceitam o header
`Idempotency-Key`. Reenviar a mesma chave devolve a resposta original, em vez de criar um
recurso duplicado. Reusar a chave com um payload diferente retorna `409 IDEMPOTENCY_CONFLICT`.

## Webhooks

A FluvPay envia cada evento ao seu endpoint com os headers `X-FluvPay-Event`,
`X-FluvPay-Timestamp`, `X-FluvPay-Delivery-Id` e `X-FluvPay-Signature`.

A assinatura vem no formato `v1=<hex>`, onde:

```
hex = HMAC_SHA256(segredo, "{timestamp}." + corpo_cru)
```

O `segredo` é o `whsec_...` exibido na criação do webhook. Sempre verifique a assinatura
(comparando em tempo constante, sobre o corpo cru) antes de processar o evento.

Eventos disponíveis:

```
charge.created   charge.paid       charge.expired   charge.cancelled   charge.refunded
payout.created   payout.completed  payout.failed
```

## Erros

Todos os erros seguem o mesmo envelope, com um `code` canônico e um `trace_id` para
correlação nos logs:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dados inválidos",
    "details": [{ "field": "amount_cents", "message": "...", "type": "..." }],
    "trace_id": "01J9X8K2P3Q4R5S6T7U8V9W0XY"
  }
}
```

## SDKs oficiais

SDKs oficiais idiomáticos, mantidos pela FluvPay, com tipagem forte, idempotência
automática, retries seguros e verificação de webhook embutida. Todos publicados nos
registries oficiais de cada linguagem.

<table align="center">
  <tr>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-node"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nodejs/nodejs-original.svg" alt="Node.js"></a><br><sub><b>Node.js</b></sub></td>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-python"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/python/python-original.svg" alt="Python"></a><br><sub><b>Python</b></sub></td>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-php"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/php/php-original.svg" alt="PHP"></a><br><sub><b>PHP</b></sub></td>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-go"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/go/go-original-wordmark.svg" alt="Go"></a><br><sub><b>Go</b></sub></td>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-java"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/java/java-original.svg" alt="Java"></a><br><sub><b>Java</b></sub></td>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-ruby"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/ruby/ruby-original.svg" alt="Ruby"></a><br><sub><b>Ruby</b></sub></td>
    <td align="center" width="115"><a href="https://github.com/fluvpay/fluvpay-dotnet"><img height="46" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/dotnetcore/dotnetcore-original.svg" alt=".NET"></a><br><sub><b>.NET</b></sub></td>
  </tr>
</table>

<p align="center"><sub>Clique no logo para abrir o repositório. O de Node.js é TypeScript-first.</sub></p>

| Linguagem | Instalação |
|---|---|
| Node.js | `npm install fluvpay` |
| Python | `pip install fluvpay` |
| PHP | `composer require fluvpay/fluvpay` |
| Go | `go get github.com/fluvpay/fluvpay-go` |
| Ruby | `gem install fluvpay` |
| .NET | `dotnet add package FluvPay` |
| Java | `com.fluvpay:fluvpay` (Maven Central) |

## Usar a especificação

A `openapi.yaml` é a fonte de verdade do contrato. Com ela você pode:

- **Visualizar** em qualquer editor OpenAPI (Swagger Editor, Redocly, Stoplight).
- **Importar** no Postman ou Insomnia para testar os endpoints.
- **Gerar um cliente** na sua linguagem com o `openapi-generator`:

```bash
npx @openapitools/openapi-generator-cli generate \
  -i https://raw.githubusercontent.com/fluvpay/OpenAPI/main/openapi.yaml \
  -g python \
  -o ./fluvpay-client
```

## Ambientes

| Ambiente | Como ativar | Move dinheiro? |
|---|---|---|
| **Sandbox** | API Key com prefixo `fluv_test_` | Não. Gera dados de teste. |
| **Produção** | API Key com prefixo `fluv_live_` | Sim. |

A mesma base URL atende os dois. O ambiente é decidido pelo prefixo da chave.

## Comunidade

Dúvidas, novidades e contato direto com o time da FluvPay no nosso Discord:

<p>
  <a href="https://discord.gg/5rpHRhTxJB"><img src="https://img.shields.io/badge/entrar%20no%20Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Entrar no Discord"></a>
</p>

## Segurança

Encontrou uma vulnerabilidade? Não abra uma issue pública. Use o canal de divulgação
responsável descrito em https://docs.fluvpay.com.

## Licença

Distribuído sob a licença MIT. Veja [`LICENSE`](LICENSE).

---

<p align="center">
  Feito pela <a href="https://fluvpay.com">FluvPay</a> · © 2026
</p>
