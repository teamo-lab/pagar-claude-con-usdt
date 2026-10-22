# Pagar la API de Claude y OpenAI con USDT desde Argentina: Claude Code, Codex y el SDK de OpenAI sin tarjeta

> Mantenido por el equipo de TeamoRouter. El gateway de esta guía es nuestro.

¿Te rechazan la tarjeta en Anthropic u OpenAI, o no tenés una tarjeta para pagos en dólares? Podés usar Claude y los modelos GPT a través de [TeamoRouter](https://teamorouter.com/?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt): Claude Code, Codex y los SDK oficiales se conectan a `api.teamorouter.com`, y el saldo se recarga con USDT. Si ya tenés una tarjeta que Anthropic u OpenAI aceptan, también podés usar el camino oficial; esta guía es para el resto de los casos.

## Respuesta corta

- **Qué cambia en tu código:** la base URL y la key (`sk-teamo-...`). Para Claude, `https://api.teamorouter.com` (sin `/v1`) con el protocolo nativo de Anthropic; para GPT, `https://api.teamorouter.com/v1` con el formato estándar de OpenAI. Requests, respuestas y streaming siguen el formato de cada protocolo.
- **Cómo pagás:** en la ventana de recarga hacés clic en **Pay with crypto**, en la página de pago elegís la moneda (USDT, USDC o ETH) y después la red, y transferís a la dirección que te muestra. La página de pago muestra qué redes admite tu monto (BEP20 funciona desde US$5). Podés mandar desde la app de tu exchange (Binance, OKX u otros) o desde cualquier wallet. No necesitás extensión del navegador y funciona desde el celular. Mínimo US$5. La ventana de recarga también acepta tarjeta a través de Stripe.
- **Qué obtenés:** una sola key para Claude, GPT (`gpt-6-sol`, `gpt-6-luna`, `gpt-6-astra` y otros), Gemini, DeepSeek y más de 60 modelos, con cobro por token y sin suscripción.
- **Qué no necesitás:** cuenta de Anthropic u OpenAI, login de ChatGPT ni tarjeta de crédito.

## ¿Por qué USDT en Argentina?

- Según el informe de Chainalysis sobre Latinoamérica publicado en octubre de 2025, Argentina es el segundo país de la región por volumen de transacciones cripto (US$93.900 millones).
- El mismo informe indica que, entre julio de 2024 y junio de 2025, las stablecoins fueron más de la mitad de las compras en exchanges hechas con pesos argentinos (según datos de order books de CryptoCompare).
- La Ley 27.739 designa a la Comisión Nacional de Valores (CNV) como regulador de los proveedores de servicios de activos virtuales (PSAV), y la CNV publica un registro de PSAV que se puede consultar en línea.
- Los precios y el saldo de TeamoRouter están en dólares. Si ya tenés USDT, recargás sin pasar por una tarjeta.

Las fuentes están al final de este archivo.

## Claude Code

Necesitás [Node.js](https://nodejs.org/en/download/) y una API key de TeamoRouter ([creala en la consola](https://teamorouter.com/dashboard?tab=api-keys&utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt)).

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

Ejecutá `claude` en la carpeta del proyecto. Guía completa con capturas: [instalar Claude Code](https://teamorouter.com/docs/install-claude-code?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés).

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

Referencia completa: [guía de integración de la API](https://teamorouter.com/docs/api-integration?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés).

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

Ejecutá `codex`. Los pasos para Windows (PowerShell) están en la [guía de Codex CLI](https://teamorouter.com/docs/install-codex?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés).

- `base_url` va **con** `/v1` al final (al revés que en Claude Code).
- Modelos: `gpt-6-sol` por defecto; `gpt-6-luna` cuesta bastante menos por token; `gpt-6-astra` es el de mayor precio de los tres. Dentro de la sesión, `/model` cambia de modelo y `/fast` activa el modo Fast (en los modelos que lo ofrecen, se factura al doble).

## Recargar con USDT, paso a paso

Necesitás una cuenta de TeamoRouter y USDT, USDC o ETH en un exchange o en una wallet, que alcance para el monto de la recarga más la comisión de retiro. Nada más: la página de pago es una página web común, así que con el celular y la app de tu exchange alcanza.

### Monedas y redes aceptadas

La página de pago acepta USDT, USDC y ETH. Qué monedas y redes ofrece depende del monto de la orden. Al 2026-10-20, la página de pago muestra lo que está disponible para tu monto:

| Monto de la orden | Qué ofrece la página de pago |
|---|---|
| Desde US$5 | ETH, USDC en BNB Smart Chain (BEP20), USDT en BNB Smart Chain (BEP20) |
| Desde US$10 | Además USDC en Base y USDT en Ethereum (ERC20) |
| Desde US$30 | Además USDT en Tron (TRC20) |

Para montos chicos, elegí **USDT en BEP20**: está disponible desde US$5, y retirar desde Binance por BEP20 cuesta alrededor de 0,01 USDT, contra alrededor de 1,5 USDT por TRC20. OKX no permite retirar USDT por BEP20; desde OKX, recargá US$10 o más y usá ERC20, que ahí cuesta menos que TRC20.

Mandá solo la moneda y la red que elegiste; cualquier otra cosa que mandes a la dirección de pago no cuenta como pago. Las comisiones de retiro las fija cada exchange y cambian con el tiempo, así que fijate cuánto cobran en la pantalla de retiro.

### Pasos

1. Iniciá sesión en TeamoRouter, abrí la ventana de recarga en la consola y elegí un monto (US$5 o más). El monto define qué monedas y redes ofrece la página de pago.
2. Hacé clic en **Pay with crypto** (en pantallas angostas el botón dice **Crypto**). Te lleva a la página de pago.
3. La página de pago primero te pide elegir la moneda (USDT, USDC o ETH) y después la red por la que vas a mandar. Solo aparecen las redes disponibles para tu monto. La página te muestra cuánto transferir y una dirección de depósito. La orden vale **60 minutos**.
4. En la app de tu exchange, iniciá un retiro (en una wallet, un envío): elegí **la misma moneda en la misma red**, pegá la dirección de la página de pago y poné el monto de forma que lo que te llegue **no sea menor que el monto de la página**.
5. Esperá a que la red confirme la transferencia. El saldo se acredita cuando se confirma el pago, aunque cierres la página de pago. Si todavía no aparece, actualizá la consola en un rato y no vuelvas a pagar.
6. Cuando se acredite el saldo, [creá una API key](https://teamorouter.com/dashboard?tab=api-keys&utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt).

### Por qué puede fallar un pago cripto

- **Mandar otra moneda o por otra red.** Retirá la misma moneda, por la misma red que elegiste en la página de pago. Las direcciones BEP20 y ERC20 empiezan las dos con `0x`, así que el exchange te va a dejar mandar USDT por ERC20 a una dirección BEP20, pero esa transferencia no cuenta como pago de la orden. Cada exchange llama a las redes a su manera: BEP20 puede aparecer como BSC o BNB Smart Chain, TRC20 como Tron y ERC20 como Ethereum. Si tu exchange no permite retirar esa moneda por esa red, volvé a la página de pago y elegí una que sí ofrezca.
- **La comisión de retiro se come parte del monto.** Muchos exchanges descuentan la comisión del monto que escribís. Antes de confirmar, mirá la línea de "monto a recibir" en la pantalla de retiro: tiene que ser por lo menos el monto de la página de pago; si la comisión sale de tu monto, sumala arriba. Una orden a la que le falta hasta un 1% igual cuenta como pagada. Si falta más, no mandes una segunda transferencia: contactá a soporte con tu TxID. Si mandás de más, el saldo se acredita por el monto de la orden y podés contactar a soporte por la diferencia.
- **La orden vence.** A veces el exchange retiene el retiro para una revisión manual, que puede tardar más de 60 minutos. Antes de hacer clic en **Pay with crypto**, asegurate de que los USDT ya estén en la cuenta del exchange desde la que vas a retirar. Si venció antes de que mandaras nada: renová la orden en la página de pago o empezá una recarga nueva, y mandá a la dirección vigente, no a una que copiaste antes. Si venció después de que ya mandaste: no vuelvas a mandar; renová la orden en la página de pago y, si con eso no se resuelve, contactá a soporte con tu TxID.

### Dónde encontrar el TxID

El TxID (también llamado hash de la transacción o Txn Hash) identifica tu transferencia en la blockchain, y soporte lo necesita para rastrear un pago.

- **App del exchange:** abrí el historial de retiros, tocá este retiro y copiá el TxID del detalle. Aparece recién cuando el exchange efectivamente mandó la transferencia; un retiro que sigue en revisión todavía no lo tiene.
- **Wallet:** abrí la transacción en el historial de la wallet; el hash está en el detalle.

Cuando contactes a soporte, pasá el TxID, la red, el monto, la hora en que mandaste y la cuenta de TeamoRouter que recargaste.

Guía completa: [Pay for Claude and GPT APIs with Crypto (USDT)](https://teamorouter.com/blogs/pay-claude-with-usdt?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés).

## ¿Ya tenés USDT en X Layer? Pagá con OKX Wallet

OKX Wallet funciona como cualquier otra wallet en la página de pago: elegís la moneda y la red ahí y mandás desde la wallet. Para mandar desde una wallet necesitás la moneda de esa red para el gas, por ejemplo BNB en BEP20.

Si tus USDT ya están en X Layer (on-chain aparecen como USD₮0), podés saltearte la página de pago. Con la [extensión OKX Wallet](https://chromewebstore.google.com/detail/okx-wallet/mcohilncbfahbmgdjkbpemcciiolgcge) instalada y desbloqueada en un navegador de escritorio (Chrome, Edge o Brave), la ventana de recarga muestra un link extra, **Have USDT on X Layer? Pay with OKX Wallet**. Usá ese link, no el botón **Pay with crypto**:

1. Elegí un monto (mínimo US$5) y hacé clic en **Have USDT on X Layer? Pay with OKX Wallet**.
2. Aprobá la conexión de la wallet y después el cambio a X Layer (la primera vez, agregar la red).
3. Firmá la solicitud: es una autorización EIP-3009 `TransferWithAuthorization` solo por este monto, de un solo uso dentro de su período de validez y sin acceso a nada más de tu wallet. La transferencia se envía por vos, así que no necesitás OKB para el gas. Algunas wallets muestran unidades con 6 decimales: `5000000` = 5 USDT.

Los USD₮0 tienen que estar en la wallet, en X Layer; el saldo en un exchange y los USDT en otras redes no cuentan. Para llevar USDT a X Layer:

| Tus USDT están en | Ruta |
|---|---|
| OKX | Activos → Retirar → USDT → red **X Layer** → dirección de tu OKX Wallet |
| Binance | Retirá por **Arbitrum** a la dirección de tu OKX Wallet (ese USDT es USDT0) y después pasalo a X Layer en [usdt0.to](https://usdt0.to). Necesitás un poco de ETH en Arbitrum para el gas |
| Otro exchange | Enviá a tu cuenta de OKX por una red que soporten los dos (por ejemplo, TRC20) y después retirá a X Layer |
| Una wallet en Tron, Ethereum, Arbitrum… | Página Transfer de [usdt0.to](https://usdt0.to): origen = tu red, destino = X Layer, destinatario = la dirección de tu OKX Wallet. La página muestra si la ruta funciona; el gas se paga en la red de origen |

Errores de esta ruta:

| Ves | Solución |
|---|---|
| No wallet found | Instalá y desbloqueá OKX Wallet y recargá la página. Usá un navegador de escritorio; los del celular normalmente no tienen extensión de wallet |
| `insufficient_balance` al pagar | No alcanza el USD₮0 **en X Layer**. Los USDT en otras redes o en un exchange no cuentan |
| Código `-32603` | Desbloqueá OKX Wallet, desactivá otras extensiones de wallet como MetaMask y recargá la página |
| Falla el cambio de red | Cambiá a X Layer a mano en la wallet y probá de nuevo |

## Errores comunes

| Ves | Solución |
|---|---|
| Pagué pero el saldo no cambió | No vuelvas a pagar; puede que la red todavía esté confirmando. Actualizá la consola en un rato. Si la orden venció o faltó monto, mirá "Por qué puede fallar un pago cripto" más arriba y, si sigue sin resolverse, contactá a soporte con tu TxID |
| Claude Code: `Not logged in · Please run /login` | `ANTHROPIC_AUTH_TOKEN` no está definida en esa terminal; no inicies sesión, cargá las variables (ver arriba) |
| API `401` | La key se copió incompleta o se borró en la consola |
| API `402 insufficient_balance` | El saldo no cubre la request; las requests de contexto largo reservan primero su costo máximo |
| Codex `404 … url: https://api.teamorouter.com/responses` | Falta `/v1` en `base_url` |

## Preguntas frecuentes

**¿Necesito cuenta de Anthropic u OpenAI, o ChatGPT Plus?** No. Llamás a `api.teamorouter.com` con una key de TeamoRouter y pagás solo por tokens.

**¿Se aplican los límites de uso de Codex?** Los límites que se reinician cada pocas horas o cada semana son de los planes de ChatGPT. Con una API key pagás por token mientras tengas saldo.

**¿La misma key sirve para Claude y para GPT?** Sí. Claude funciona con el protocolo nativo de Anthropic: el SDK de Anthropic con `base_url="https://api.teamorouter.com"` (sin `/v1`) o [Claude Code](https://teamorouter.com/docs/install-claude-code?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) con `ANTHROPIC_BASE_URL`. Por el formato de OpenAI, Claude puede perder el prompt caching y el thinking.

**¿Puedo pagar desde el celular?** Sí. Abrí la página de pago en el celular y copiá la dirección y el monto en la app de tu exchange.

**¿Hay comisiones por pagar con cripto?** La página de pago no agrega comisión de procesamiento. Pagás la comisión de retiro de tu exchange, o el gas de la red si mandás desde una wallet.

**¿Puedo pagar con tarjeta?** Sí, la ventana de recarga también acepta tarjeta a través de Stripe.

**¿Cuánto cuesta?** Los precios por token de cada modelo están en la [página de precios](https://teamorouter.com/?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt#pricing). Sin suscripción.

## Links

- [Inicio de TeamoRouter](https://teamorouter.com/?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt)
- [Pagar con cripto: guía completa](https://teamorouter.com/blogs/pay-claude-with-usdt?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés)
- [Instalar Claude Code](https://teamorouter.com/docs/install-claude-code?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés)
- [Instalar Codex CLI](https://teamorouter.com/docs/install-codex?utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt) (en inglés)
- [Crear una API key](https://teamorouter.com/dashboard?tab=api-keys&utm_source=github&utm_medium=readme&utm_campaign=b6_pagar-claude-con-usdt)
- [teamo-lab/claude-api-card-declined](https://github.com/teamo-lab/claude-api-card-declined): si Anthropic rechaza tu tarjeta (en inglés)
- [teamo-lab/openrouter-usdt](https://github.com/teamo-lab/openrouter-usdt): OpenRouter solo acepta USDC; qué hacer si tenés USDT (en inglés)

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
