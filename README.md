# Requerimientos - Integración de Proveedores por API - Construplaza

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Cómo usa Construplaza la API](#cómo-usa-construplaza-la-api)
3. [Requisitos generales](#requisitos-generales)
4. [Fases del Proyecto](#-fases-del-proyecto)
   - [Fase 1 - Consultar Existencia y Costos](#-fase-1---consultar-existencia-y-costos)
   - [Fase 2 - Cargar Pedidos u Órdenes](#-fase-2---cargar-pedidos-u-órdenes)
     - [2.1 - Consultar el pedido por nuestra orden de compra](#21---consultar-el-pedido-por-nuestra-orden-de-compra)
     - [2.2 - Obtener Factura](#22---obtener-factura)
5. [Manejo de Errores](#manejo-de-errores)
6. [Ambiente de Pruebas](#-ambiente-de-pruebas)
7. [Seguridad](#seguridad)
8. [Lista de verificación para el proveedor](#lista-de-verificación-para-el-proveedor)
9. [Autor](#autor)
10. [Bitácora de Documentación](#bitácora-de-documentación)

## Introducción

Construplaza compra a sus proveedores de forma automática: valida existencias y precios contra la API del proveedor, coloca la orden de compra por API y la confirma cuando el proveedor la factura.

Este documento describe lo que la API del proveedor debe ofrecer para integrarse a ese proceso. La integración tiene dos fases:

- **Fase 1:** consultar existencia y costos de los artículos.
- **Fase 2:** cargar el pedido, consultarlo y obtener su factura.

Las dos son necesarias para que un proveedor quede integrado.

## Cómo usa Construplaza la API

Conocer el uso esperado ayuda al proveedor a dimensionar su API:

| Proceso | Qué consulta | Frecuencia aproximada |
| :--- | :--- | :--- |
| Actualización del catálogo | Existencia y precio de los artículos homologados (Fase 1) | Varias veces al día |
| Envío de órdenes | Existencia y precio de los artículos de cada orden (Fase 1), consulta de si el pedido ya existe (2.1) y creación del pedido (Fase 2) | Cada 15 a 20 minutos, solo si hay órdenes nuevas |
| Confirmación de órdenes | Facturas de cada pedido enviado y todavía sin facturar (2.2) | Cada hora |

Las órdenes se procesan de una en una. Si la API tiene límites de solicitudes por minuto, el proveedor debe indicarlos.

## Requisitos generales

- **HTTPS y JSON** en todas las solicitudes y respuestas.
- **Un mismo código de artículo (SKU) en todos los endpoints.** Construplaza homologa sus artículos contra ese código. Si el proveedor maneja más de un código por artículo, debe indicar cuál es el que se usa en la API.
- **Cantidades enteras.** Construplaza nunca envía cantidades con decimales; la API debe expresar existencias, cantidades de pedido y cantidades facturadas en una unidad que permita cantidades enteras.
- **La misma unidad en todos los endpoints.** Existencia, precio, cantidad pedida y cantidad facturada deben estar en la misma unidad de venta del artículo. Si un artículo se vende por empaque (caja, rollo, carrete), la API debe decir cuántas unidades trae el empaque y si el precio es por unidad o por empaque.
- **Fechas** en formato ISO 8601 (`2026-11-24T10:30:00-06:00`).
- **Montos** con punto decimal y sin separador de miles.

## 🔄 Fases del Proyecto

### 🔵 Fase 1 - Consultar Existencia y Costos

**Descripción:**
Consultar la existencia y el costo actualizados de los artículos, tanto para mantener el catálogo al día como para validar cada orden antes de colocarla.

**Datos Requeridos por artículo:**

| Parámetro | Tipo | Requerido | Ejemplo | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `sku` | `string` | Sí | `"PRU01"` | Código del artículo en el proveedor |
| `descripcion` | `string` | Sí | `"Cable THHN 12 AWG negro"` | Descripción del artículo |
| `precio` | `decimal` | Sí | `21843.00` | Precio unitario **sin impuestos**, ya con descuentos aplicados |
| `moneda` | `string` | Sí | `"CRC"` o `"USD"` | Moneda del precio |
| `impuesto` | `string` | Sí | `"13%"` | Tarifa de IVA del artículo |
| `unidadVenta` | `string` | Sí | `"UND"`, `"M"`, `"CAJA"` | Unidad en que se expresan precio, existencia y cantidad |
| `unidadesPorEmpaque` | `int` | Sí | `1`, `20`, `500` | Cantidad mínima que despacha el proveedor; los pedidos deben ser múltiplo de este valor |
| `existencia` | `int` o `null` | Sí | `0`, `1`, `20` | Existencia disponible en `unidadVenta`. `null` solo si el proveedor no reporta existencias; `0` significa que no hay |
| `bodega` | `string` | Si aplica | `"1"` | Bodega de la existencia, si el proveedor tiene varias |
| `activo` | `boolean` | Sí | `true` | `false` si el artículo está descontinuado o no se puede pedir |

**Reglas:**
- Un artículo sin precio debe venir con `precio` en `null` y no en `0`.
- Si la existencia es por bodega, debe venir un registro por bodega, o una lista de bodegas por artículo.
- La consulta debe permitir **una lista de SKUs** (para validar una orden) y **el catálogo completo**, paginado si es grande. Si es paginado, hay que documentar cómo se pide la página siguiente.

**Ejemplo de respuesta:**

```json
[
  {
    "sku": "PRUEBA01",
    "descripcion": "ARTICULO PRUEBA",
    "precio": 3239.00,
    "moneda": "CRC",
    "impuesto": "13%",
    "unidadVenta": "UND",
    "unidadesPorEmpaque": 1,
    "existencia": 10,
    "bodega": "4",
    "activo": true
  },
  {
    "sku": "PRUEBA02",
    "descripcion": "CABLE PRUEBA CARRETE 500 M",
    "precio": 0.4977,
    "moneda": "USD",
    "impuesto": "13%",
    "unidadVenta": "M",
    "unidadesPorEmpaque": 500,
    "existencia": 2000,
    "bodega": "2",
    "activo": true
  }
]
```

**Descripción gráfica:**

![Fase 1 - Descripción](Documentacion/fase1.png)

### 🔵 Fase 2 - Cargar Pedidos u Órdenes

**Descripción:**
Colocar por API el pedido de una orden de compra de Construplaza, con las cantidades y precios ya validados en la Fase 1.

**Datos a Enviar:**

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `ordenCompra` | `string` | Sí | Número de orden de compra de Construplaza. Es único: el proveedor debe rechazar una segunda orden con el mismo número |
| `fecha` | `string` | Sí | Fecha de la orden de compra |
| `entrega` | `object` | Sí | Bodega o dirección de destino y modalidad (`"entrega"` o `"retira"`) |
| `lineas` | `array` | Sí | Una por artículo |
| `lineas[].sku` | `string` | Sí | Código del artículo, igual al de la Fase 1 |
| `lineas[].cantidad` | `int` | Sí | Cantidad entera en `unidadVenta`, múltiplo de `unidadesPorEmpaque` |
| `lineas[].precio` | `decimal` | Sí | Precio unitario sin impuestos que espera Construplaza |
| `lineas[].bodega` | `string` | Si aplica | Bodega del proveedor de la que sale el artículo |
| `observaciones` | `string` | No | Instrucciones especiales de entrega o manejo |

**Ejemplo de Cuerpo de Solicitud:**

```json
{
  "ordenCompra": "P0001",
  "fecha": "2026-11-24",
  "entrega": { "destino": "Bodega central", "modalidad": "entrega" },
  "lineas": [
    { "sku": "PRUEBA01", "cantidad": 2, "precio": 3239.00 },
    { "sku": "PRUEBA02", "cantidad": 1000, "precio": 0.4977 }
  ],
  "observaciones": ""
}
```

**Respuesta:**
La respuesta debe traer el **número de pedido del proveedor**, que Construplaza guarda para consultar la factura:

```json
{
  "numeroPedido": "1020192",
  "ordenCompra": "P0001",
  "estado": "recibido",
  "fecha": "2026-11-24T10:30:00-06:00"
}
```

Si el proveedor asigna el número de pedido más tarde (procesamiento asíncrono), la respuesta puede venir con `numeroPedido` en `null`. En ese caso el endpoint 2.1 es obligatorio, y el proveedor debe indicar en cuánto tiempo, normalmente, queda asignado el número.

**Descripción gráfica:**

![Fase 2 - Descripción](Documentacion/fase2.png)

#### 2.1 - Consultar el pedido por nuestra orden de compra

Antes de colocar un pedido, Construplaza consulta si el proveedor ya lo tiene, para no comprar dos veces si una solicitud anterior se cortó. También lo usa para obtener el número de pedido cuando se asigna de forma asíncrona.

| Parámetro | Tipo | Descripción |
| :--- | :--- | :--- |
| `ordenCompra` | `string` | **Requerido**. Número de orden de compra de Construplaza |

**Ejemplo de respuesta:**

```json
{
  "ordenCompra": "P0001",
  "numeroPedido": "1020192",
  "estado": "procesado",
  "motivo": null,
  "lineas": [
    { "sku": "PRUEBA01", "cantidad": 2, "precio": 3239.00 }
  ]
}
```

- Si la orden no existe, debe responder `404` o una lista vacía, nunca un error `500`.
- Los valores posibles de `estado` deben estar documentados, por ejemplo `recibido`, `procesado`, `rechazado` y `anulado`. Si un pedido se rechaza o queda en revisión, `motivo` debe decir por qué.

#### 2.2 - Obtener Factura

La facturación no es inmediata, por lo que Construplaza consulta periódicamente las facturas de cada pedido enviado. Con ellas compara lo facturado contra lo pedido y confirma la orden de compra.

| Parámetro | Tipo | Descripción |
| :--- | :--- | :--- |
| `pedido` | `string` | **Requerido**. Número de pedido del proveedor o número de orden de compra de Construplaza (el proveedor indica cuál) |

**Reglas:**
- Un pedido puede tener **varias facturas** si se entrega por partes. La respuesta debe traerlas todas.
- El número de factura debe ser el **consecutivo de la factura electrónica** (20 dígitos).
- Las cantidades deben venir en la misma `unidadVenta` de la Fase 1 y en el mismo SKU del pedido.
- Un pedido todavía sin facturar no es un error: responde con la lista de facturas vacía.

**Ejemplo de respuesta con factura disponible:**

```json
{
  "pedido": "1020192",
  "facturas": [
    {
      "numero": "00100001010000088888",
      "fecha": "2026-11-25T09:00:00-06:00",
      "moneda": "CRC",
      "total": 7320.14,
      "lineas": [
        { "sku": "PRUEBA01", "cantidad": 2, "precioUnitario": 3239.00 }
      ]
    }
  ]
}
```

**Ejemplo de respuesta sin factura:**

```json
{
  "pedido": "1020192",
  "facturas": []
}
```

## Manejo de Errores

Construplaza decide si reintenta una solicitud según el código HTTP. Por eso la API debe respetar esta regla:

| Código HTTP | Significado | Qué hace Construplaza |
| :--- | :--- | :--- |
| `2xx` | Solicitud correcta | Continúa |
| `4xx` | Error en la solicitud (datos, artículo, cantidad) | No reintenta hasta corregir la orden |
| `409` | La orden de compra ya existe | No reintenta; consulta el pedido con 2.1 |
| `5xx` | Falla temporal del proveedor | Reintenta más tarde |

Un error de datos nunca debe responder `5xx`, porque se reintentaría sin sentido.

**Formato de Respuesta en Caso de Error:**

```json
{
  "codigo": 23,
  "mensaje": "Stock disponible insuficiente para el SKU PRUEBA01.",
  "sku": "PRUEBA01"
}
```

**Catálogo de errores sugerido:**

| Código | Mensaje |
| :--- | :--- |
| `1` | No se envió un parámetro requerido. |
| `23` | Stock disponible insuficiente para el SKU. |
| `24` | La cantidad no es múltiplo del empaque del SKU. |
| `26` | Se solicitan demasiados productos. Debe indicar el máximo permitido. |
| `27` | El producto está desactivado. |
| `28` | Producto no autorizado o no existe. |
| `29` | Lista de productos vacía. |
| `30` | La orden de compra ya existe. |
| `31` | El pedido no tiene el formato correcto. |

Si el proveedor usa otros códigos, debe entregar su catálogo.

## 🔧 Ambiente de Pruebas

El proveedor debe ofrecer un ambiente de pruebas con estas condiciones:

- **No genera pedidos ni facturas reales.**
- Tiene la **misma estructura de API** que producción, con credenciales propias.
- Tiene **SKUs de prueba con existencia y precio**, incluidos artículos por empaque y artículos sin existencia.
- Permite **generar una factura de prueba** para un pedido de prueba, porque sin ella no se puede probar la confirmación.
- Se mantiene disponible hasta terminar todas las pruebas.

Si el ambiente de pruebas se elige con un parámetro o encabezado, por ejemplo `X-Environment: test`, debe estar documentado.

## Seguridad

- **Autenticación:** preferiblemente OAuth 2.0 (*client credentials*) o un token *Bearer*. Otros esquemas se pueden evaluar, pero deben estar documentados por completo.
- **Credenciales separadas** para pruebas y producción, entregadas por un canal seguro y nunca por correo en texto plano. El proveedor debe indicar cómo se rotan.
- **Lista blanca de IPs:** si el proveedor restringe el acceso por IP, Construplaza le entregará las IPs de origen al iniciar la integración.

## Lista de verificación para el proveedor

Antes de pasar a producción, el proveedor debe entregar:

- [ ] Documentación de los endpoints de la Fase 1, la Fase 2, 2.1 y 2.2
- [ ] Credenciales de pruebas y de producción
- [ ] Lista de SKUs de prueba
- [ ] Unidad de venta y empaque de los artículos que maneja
- [ ] Si reporta existencias, y por cuál bodega
- [ ] Si el número de pedido es inmediato o asíncrono, y el tiempo esperado
- [ ] Catálogo de errores y valores de `estado`
- [ ] Límites de solicitudes, si existen
- [ ] Contacto técnico para incidentes

## Autor

Esto es una propuesta de trabajo desarrollada por CONSTRUPLAZA S.A

<img src="Documentacion/construplaza_logo.png" alt="Logo Construplaza" width="200"/>


## Bitácora de Documentación

- 2024-08-12 - Andrei Calderón Molina - Creación de la documentación necesaria para el proyecto de APIs

- 2024-08-16 - Andrei Calderón Molina - Revisión y ajuste de documentación

- 2024-10-11 - Andrei Calderón Molina - Agregación del endpoint necesario para la consulta de facturación

- 2024-11-05 - Andrei Calderón Molina - Agregación de ajuste de seguridad, lista blanca de ip

- 2026-11-24 - Andrei Calderón Molina - Corrección de ejemplos JSON, catálogo de errores, ortografía e índice

- 2026-11-24 - Erick Abarca - Actualización a la integración actual: se quita la Fase 3; se agregan unidad de venta, empaque y bodega en la Fase 1; consulta del pedido por orden de compra, estados y varias facturas por pedido en la Fase 2; regla de reintentos por código HTTP; requisitos del ambiente de pruebas y lista de verificación
