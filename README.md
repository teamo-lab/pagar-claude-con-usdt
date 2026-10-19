# Pagar la API de Claude y OpenAI con USDT desde Argentina: Claude Code, Codex y el SDK de OpenAI sin tarjeta

> Mantenido por el equipo de TeamoRouter. El gateway de esta guía es nuestro.

¿Te rechazan la tarjeta en Anthropic u OpenAI, o no tenés una tarjeta para pagos en dólares? Podés usar Claude y los modelos GPT a través de [TeamoRouter](https://teamorouter.com/?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt): Claude Code, Codex y los SDK oficiales se conectan a `api.teamorouter.com`, y el saldo se recarga con USDT. Si ya tenés una tarjeta que Anthropic u OpenAI aceptan, también podés usar el camino oficial; esta guía es para el resto de los casos.

## Respuesta corta

- **Qué cambia en tu código:** la base URL y la key (`sk-teamo-...`). Para Claude, `https://api.teamorouter.com` (sin `/v1`) con el protocolo nativo de Anthropic; para GPT, `https://api.teamorouter.com/v1` con el formato estándar de OpenAI. Requests, respuestas y streaming siguen el formato de cada protocolo.
- **Cómo pagás:** USDT en la red X Layer desde la extensión OKX Wallet del navegador. Firmás una autorización, sin gas. Mínimo US$5. La ventana de recarga también acepta tarjeta a través de Stripe.
- **Qué obtenés:** una sola key para Claude, GPT (`gpt-6-sol`, `gpt-6-luna`, `gpt-6-astra` y otros), Gemini, DeepSeek y más de 30 modelos, con cobro por token y sin suscripción.
- **Qué no necesitás:** cuenta de Anthropic u OpenAI, login de ChatGPT ni tarjeta de crédito.

## ¿Por qué USDT en Argentina?

- Según el informe de Chainalysis sobre Latinoamérica publicado en octubre de 2025, Argentina es el segundo país de la región por volumen de transacciones cripto (US$93.900 millones).
- El mismo informe indica que, entre julio de 2024 y junio de 2025, las stablecoins fueron más de la mitad de las compras en exchanges hechas con pesos argentinos (según datos de order books de CryptoCompare).
- La Ley 27.739 designa a la Comisión Nacional de Valores (CNV) como regulador de los proveedores de servicios de activos virtuales (PSAV), y la CNV publica un registro de PSAV que se puede consultar en línea.
- Los precios y el saldo de TeamoRouter están en dólares. Si ya tenés USDT, recargás sin pasar por una tarjeta.

Las fuentes están al final de este archivo.

## Claude Code

Necesitás [Node.js](https://nodejs.org/en/download/) y una API key de TeamoRouter ([creala en la consola](https://teamorouter.com/dashboard?tab=api-keys&utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt)).

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

**macOS y Linux** (el script elige `~/.zshrc` o `~/.bashrc` según tu shell):

```bash
SHELL_RC="$HOME/.zshrc"; case "$SHELL" in *bash*) SHELL_RC="$HOME/.bashrc";; esac
cat >> "$SHELL_RC" <<'EOF'
export ANTHROPIC_BASE_URL=https://api.teamorouter.com
export ANTHROPIC_AUTH_TOKEN="<tu-key-de-TeamoRouter>"
export ANTHROPIC_MODEL=claude-opus-5-5
export ANTHROPIC_DEFAULT_FABLE_MODEL=claude-fable-5-1
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-5-5
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-5
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5
export CLAUDE_CODE_SUBAGENT_MODEL=claude-sonnet-5
export CLAUDE_CODE_EFFORT_LEVEL=xhigh
EOF
source "$SHELL_RC"
```

**Windows (PowerShell)**: las mismas variables con `setx`; después cerrá y volvé a abrir la terminal.

```powershell
setx ANTHROPIC_BASE_URL "https://api.teamorouter.com"
setx ANTHROPIC_AUTH_TOKEN "<tu-key-de-TeamoRouter>"
setx ANTHROPIC_MODEL "claude-opus-5-5"
setx ANTHROPIC_DEFAULT_FABLE_MODEL "claude-fable-5-1"
setx ANTHROPIC_DEFAULT_OPUS_MODEL "claude-opus-5-5"
setx ANTHROPIC_DEFAULT_SONNET_MODEL "claude-sonnet-5"
setx ANTHROPIC_DEFAULT_HAIKU_MODEL "claude-haiku-4-5"
setx CLAUDE_CODE_SUBAGENT_MODEL "claude-sonnet-5"
setx CLAUDE_CODE_EFFORT_LEVEL "xhigh"
```

Ejecutá `claude` en la carpeta del proyecto. Guía completa con capturas: [instalar Claude Code](https://teamorouter.com/docs/install-claude-code?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés).

- `ANTHROPIC_BASE_URL` va **sin** `/v1` al final.
- ¿Ves `Not logged in · Please run /login`? No inicies sesión: en esa terminal no está definida `ANTHROPIC_AUTH_TOKEN`. Ejecutá `source ~/.zshrc` (o `~/.bashrc`), o abrí una terminal nueva después de `setx`.

## SDK de Anthropic

**Python** (`pip install anthropic`):

```python
from anthropic import Anthropic

client = Anthropic(
    api_key="sk-teamo-xxxxxx",
    base_url="https://api.teamorouter.com",
)

resp = client.messages.create(
    model="claude-fable-5-1",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hola"}],
)
print(resp.content[0].text)
```

## SDK de OpenAI

**Python** (`pip install openai`):

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-teamo-xxxxxx",
    base_url="https://api.teamorouter.com/v1",
)

resp = client.chat.completions.create(
    model="gpt-6-sol",
    messages=[{"role": "user", "content": "Hola"}],
)
print(resp.choices[0].message.content)
```

**Node.js** (`npm install openai`), Responses API:

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: "sk-teamo-xxxxxx",
  baseURL: "https://api.teamorouter.com/v1",
});

const resp = await client.responses.create({
  model: "gpt-6-sol",
  input: "Presentá TeamoRouter en una oración",
});
console.log(resp.output_text);
```

**Sin cambiar código**, para herramientas que leen las variables de entorno estándar:

```bash
export OPENAI_BASE_URL="https://api.teamorouter.com/v1"
export OPENAI_API_KEY="sk-teamo-xxxxxx"
```

Para tener en cuenta:

- Funcionan `/v1/chat/completions` y `/v1/responses`; `/v1/responses` es solo para modelos GPT.
- Para ver los modelos que puede usar tu key: `curl https://api.teamorouter.com/v1/models -H "Authorization: Bearer sk-teamo-xxxxxx"`.
- Modo Fast opcional: `service_tier: "fast"` en los modelos que lo ofrecen (por ejemplo, `gpt-6-astra`); se factura al doble.

Referencia completa: [guía de integración de la API](https://teamorouter.com/docs/api-integration?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés).

## Codex CLI

Instalá Codex (Node.js 22 o superior) con `npm install -g @openai/codex` y después agregá el proveedor y la key. macOS / Linux:

```bash
mkdir -p ~/.codex && cat > ~/.codex/config.toml <<'EOF'
model_provider = "teamorouter"
model = "gpt-6-sol"
model_reasoning_effort = "high"
[model_providers.teamorouter]
name = "TeamoRouter"
base_url = "https://api.teamorouter.com/v1"
env_key = "OPENAI_API_KEY"
wire_api = "responses"
EOF
echo 'export OPENAI_API_KEY="<tu-key-de-TeamoRouter>"' >> ~/.zshrc   # ~/.bashrc en Linux
source ~/.zshrc
```

Ejecutá `codex`. Los pasos para Windows (PowerShell) están en la [guía de Codex CLI](https://teamorouter.com/docs/install-codex?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés).

- `base_url` va **con** `/v1` al final (al revés que en Claude Code).
- Modelos: `gpt-6-sol` por defecto; `gpt-6-luna` cuesta bastante menos por token; `gpt-6-astra` es el de mayor precio de los tres. Dentro de la sesión, `/model` cambia de modelo y `/fast` activa el modo Fast (en los modelos que lo ofrecen, se factura al doble).

## Recargar con USDT, paso a paso

Necesitás un navegador de escritorio (Chrome, Edge o Brave) con la [extensión OKX Wallet](https://chromewebstore.google.com/detail/okx-wallet/mcohilncbfahbmgdjkbpemcciiolgcge) desbloqueada, y USDT en la red **X Layer** dentro de esa wallet (on-chain aparece como USD₮0, 1:1 con USDT). El saldo en un exchange no cuenta hasta que lo retirás a la wallet.

### 1. Llevar USDT a X Layer

| Tus USDT están en | Ruta |
|---|---|
| OKX | Activos → Retirar → USDT → red **X Layer** → dirección de tu OKX Wallet |
| Binance | Retirá por **Arbitrum** a la dirección de tu OKX Wallet (ese USDT es USDT0) y después pasalo a X Layer en [usdt0.to](https://usdt0.to). Necesitás un poco de ETH en Arbitrum para el gas |
| Otro exchange | Enviá a tu cuenta de OKX por una red que soporten los dos (por ejemplo, TRC20) y después retirá a X Layer |
| Una wallet en Tron, Ethereum, Arbitrum… | Página Transfer de [usdt0.to](https://usdt0.to): origen = tu red, destino = X Layer, destinatario = la dirección de tu OKX Wallet. La página muestra si la ruta funciona; el gas se paga en la red de origen |

Mandá primero un monto chico de prueba: la red equivocada es el error más común.

### 2. Pagar

1. Iniciá sesión en TeamoRouter y abrí la ventana de recarga en la consola.
2. Elegí un monto (mínimo US$5) y hacé clic en **Pay with crypto**.
3. Aprobá la conexión de la wallet y después el cambio a X Layer (la primera vez, agregar la red).
4. Firmá la solicitud: es una autorización EIP-3009 `TransferWithAuthorization` solo por este monto, de un solo uso dentro de su período de validez y sin acceso a nada más de tu wallet. La transferencia se envía por vos, así que no necesitás OKB para el gas. Algunas wallets muestran unidades con 6 decimales: `5000000` = 5 USDT.
5. Cuando se acredite el saldo, [creá una API key](https://teamorouter.com/dashboard?tab=api-keys&utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt).

Guía completa: [Pay for Claude and GPT APIs with USDT on X Layer](https://teamorouter.com/blogs/pay-claude-with-usdt?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés).

## Errores comunes

| Ves | Solución |
|---|---|
| No wallet found | Instalá y desbloqueá OKX Wallet y recargá la página. Usá un navegador de escritorio; los del celular normalmente no tienen extensión de wallet |
| `insufficient_balance` al pagar | No alcanza el USD₮0 **en X Layer**. Los USDT en otras redes o en un exchange no cuentan |
| Código `-32603` | Desbloqueá OKX Wallet, desactivá otras extensiones de wallet como MetaMask y recargá la página |
| Claude Code: `Not logged in · Please run /login` | `ANTHROPIC_AUTH_TOKEN` no está definida en esa terminal; no inicies sesión, cargá las variables (ver arriba) |
| API `401` | La key se copió incompleta o se borró en la consola |
| API `402 insufficient_balance` | El saldo no cubre la request; las requests de contexto largo reservan primero su costo máximo |
| Codex `404 … url: https://api.teamorouter.com/responses` | Falta `/v1` en `base_url` |

## Preguntas frecuentes

**¿Necesito cuenta de Anthropic u OpenAI, o ChatGPT Plus?** No. Llamás a `api.teamorouter.com` con una key de TeamoRouter y pagás solo por tokens.

**¿Se aplican los límites de uso de Codex?** Los límites que se reinician cada pocas horas o cada semana son de los planes de ChatGPT. Con una API key pagás por token mientras tengas saldo.

**¿La misma key sirve para Claude y para GPT?** Sí. Claude funciona con el protocolo nativo de Anthropic: el SDK de Anthropic con `base_url="https://api.teamorouter.com"` (sin `/v1`) o [Claude Code](https://teamorouter.com/docs/install-claude-code?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) con `ANTHROPIC_BASE_URL`. Por el formato de OpenAI, Claude puede perder el prompt caching y el thinking.

**¿Puedo pagar desde el celular?** Para USDT necesitás un navegador de escritorio con OKX Wallet: los navegadores del celular normalmente no tienen extensión de wallet.

**¿Puedo pagar con tarjeta?** Sí, la ventana de recarga también acepta tarjeta a través de Stripe.

**¿Cuánto cuesta?** Los precios por token de cada modelo están en la [página de precios](https://teamorouter.com/?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt#pricing). Sin suscripción.

## Links

- [Inicio de TeamoRouter](https://teamorouter.com/?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt)
- [Pagar con USDT: guía completa](https://teamorouter.com/blogs/pay-claude-with-usdt?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés)
- [Instalar Claude Code](https://teamorouter.com/docs/install-claude-code?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés)
- [Instalar Codex CLI](https://teamorouter.com/docs/install-codex?utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt) (en inglés)
- [Crear una API key](https://teamorouter.com/dashboard?tab=api-keys&utm_source=github&utm_medium=readme&utm_campaign=pagar-claude-con-usdt)

<!--
Fuentes de "¿Por qué USDT en Argentina?" (consultadas el 2026-10-19):
- Chainalysis, informe de adopción cripto en Latinoamérica (publicado el 2025-10-02): https://www.chainalysis.com/blog/latin-america-crypto-adoption-2026/
  "Argentina ranks second regionally with $93.9 billion in transaction volume"; stablecoins = más de la mitad de las compras en exchanges con pesos argentinos entre julio de 2024 y junio de 2025 (datos de order books de CryptoCompare).
- Argentina.gob.ar, "Regulación de los Proveedores de Servicios de Activos Virtuales (PSAV)" (2025-03-13): https://www.argentina.gob.ar/noticias/regulacion-de-los-proveedores-de-servicios-de-activos-virtuales-psav
  La Ley 27.739 designa a la CNV como regulador de los PSAV; RG CNV 1058/2025.
- CNV, Registro de Proveedores de Servicios de Activos Virtuales: https://www.argentina.gob.ar/cnv/registro-de-proveedores-de-servicios-de-activos-virtuales
  Consulta del registro: https://www.cnv.gov.ar/SitioWeb/ProveedoresServiciosActivosVirtuales/RegistrosPSAV
- Precios en USD: https://api.teamorouter.com/v1/models/pricing (list_cost.currency = "USD").
-->
