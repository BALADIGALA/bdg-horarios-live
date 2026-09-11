# Mapa de rutas — bdghq.com

Reorganización del 11/09/2026. Toda ruta vieja redirige (308) a la nueva: ningún
enlace guardado se rompe.

## Estructura

| Ruta nueva | Antes | Fuente de datos |
|---|---|---|
| `/` | `/` | estático |
| **VENTA** | | |
| `/venta/vivo` | `/live` | Meta + Shopify |
| `/venta/pulso` | `/pulse` | Shopify + RTDB |
| `/venta/metas` | `/metas`, `/meta` | estático |
| `/venta/anual` | `/50m` | Shopify |
| **STOCK** | | |
| `/stock/inventario` | `/inventario` | Shopify |
| `/stock/restock` | `/restock` | Shopify + RTDB |
| `/stock/ordenes` | `/oc` | RTDB |
| `/stock/curvas` | `/curvas` | estático |
| **CANAL** | | |
| `/canal/tiktok` | `/tiktok` | Shopify + RTDB |
| `/canal/palacio/inventario` | `/pdh` | RTDB |
| `/canal/palacio/catalogo` | `/catalogo` | Shopify + RTDB |
| **DISEÑO** | | |
| `/diseno` | `/diseno` | Shopify + RTDB |
| `/diseno/penny` | `/penny` | Shopify + RTDB |
| **OPERACIÓN** | | |
| `/op/logistica` | `/logistica` | Shopify |
| **CUENTA** | | |
| `/cuenta/entrar` | `/login` | estático |
| `/cuenta/registro` | `/register` | estático |
| `/cuenta/perfil` | `/perfil` | estático |
| `/cuenta/admin` | `/admin` | estático |

## Qué cambió y por qué

1. **Agrupado por la pregunta que responde**, no por orden de construcción. Se cayó
   la numeración "Módulo 0X" —que no decía nada— y la etiqueta de cada tarjeta ahora
   es su grupo.
2. **Un solo idioma.** `/login`→`/cuenta/entrar`, `/register`→`/cuenta/registro`,
   `/live`→`/venta/vivo`, `/pulse`→`/venta/pulso`, `/oc`→`/stock/ordenes`.
3. **Las dos páginas de Palacio quedaron juntas.** `/pdh` y `/catalogo` no tenían
   relación en la URL aunque las dos son Palacio de Hierro.
4. **`/penny` pasó a ser hija de `/diseno`**, que es lo que ya era: su botón de
   regreso siempre apuntó a Diseño.
5. **`/meta` eliminada.** Era un placeholder "Próximamente" que duplicaba `/metas`.
   Redirige a `/venta/metas`.
6. **Seis módulos dejaron de estar huérfanos.** `/pulse`, `/logistica`, `/catalogo`,
   `/curvas`, `/oc` y `/penny` existían pero el hub no los enlazaba: solo se llegaba
   escribiendo la URL. Ahora están en el hub.

## Rescatado de producción

`/penny` y `/oc` vivían solo en el servidor, nunca estuvieron en git. Se bajaron y
se incorporaron al repo como `public/diseno/penny/` y `public/stock/ordenes/`.

## Alias de API

Se agregaron nombres semánticos como *rewrites*; los viejos siguen funcionando, así
que no hubo que tocar ni un `fetch` de las páginas:

| Alias nuevo | Endpoint real |
|---|---|
| `/api/shopify/inventario` | `/api/inventory` |
| `/api/shopify/ordenes` | `/api/orders` |
| `/api/shopify/canales` | `/api/channels` |
| `/api/shopify/fulfillment` | `/api/fulfillment` |
| `/api/shopify/imagenes` | `/api/images` |
| `/api/shopify/catalogo` | `/api/shopify` |
| `/api/meta/vivo` | `/api/live` |
| `/api/stock/restock` | `/api/restock` |

Los archivos en `api/` **no se movieron**: renombrarlos obliga a editar el `fetch`
de cada página y a corregir los `require('./_shopify')`, y eso no se puede verificar
sin poder desplegar.

## Variables de entorno que necesita el proyecto

| Variable | Para qué | Estado |
|---|---|---|
| `SHOPIFY_TOKEN` | todo lo de Shopify | disponible |
| `META_TOKEN`, `META_ACCOUNT_ID` | `/venta/vivo` | disponible |
| `PIN_HQ`, `PIN_PULSE` | candado de acceso | hay que fijarlos |
| `KV_REST_API_URL`, `KV_REST_API_TOKEN` | `api/state.js` | no disponible |
| `WHATSAPP_TOKEN`, `WA_PHONE_ID` | recordatorios de `api/remind.js` | no disponible |
| `CRON_SECRET` | protege los dos crons | hay que fijarlo |

La base Firebase RTDB **no necesita credencial**: sus reglas son de lectura pública.
Ver la nota de seguridad abajo.

## ⚠️ Nota de seguridad (hallazgo al auditar las fuentes)

`database.rules.json` tiene `".read": true` en la raíz, y `restock_snapshot`,
`restock_pos` y `bdg_horarios_v2/pulse_channels` tienen además `".write": true`.
Cualquiera que conozca la URL de la base puede **leer** inventario, datos de Palacio
y horarios, y **escribir** en los nodos de restock. La URL viaja en el HTML de las
páginas, así que es pública de hecho. No se tocó nada: es decisión de dirección.
