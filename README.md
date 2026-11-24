# Requerimientos - Proyecto Construplaza APIs

## Tabla de Contenidos
1. [Introducción](#introducción)
   - [Descripción del Proyecto](#descripción-del-proyecto)
2. [Objetivos del Proyecto](#objetivos-del-proyecto)
   - [Objetivos Específicos](#objetivos-específicos)
3. [Alcance del Proyecto](#alcance-del-proyecto)
   - [Funcionalidades](#funcionalidades)
4. [Requisitos del Proyecto](#requisitos-del-proyecto)
   - [Requisitos Funcionales](#requisitos-funcionales)
   - [Requisitos No Funcionales](#requisitos-no-funcionales)
5. [Fases del Proyecto](#-fases-del-proyecto)
   - [Fase 1 - Consultar Existencia y Costos](#-fase-1---consultar-existencia-y-costos)
   - [Fase 2 - Cargar Pedidos u Órdenes](#-fase-2---cargar-pedidos-u-órdenes)
     - [Fase 2.1 - Obtener Factura](#21---obtener-factura)
   - [Fase 3 - Consultar Productos de Pedido Especial](#-fase-3---consultar-productos-de-pedido-especial)
6. [Flujo de Trabajo General para Nuestra Implementación](#flujo-de-trabajo-general-para-nuestra-implementación)
7. [Ambiente de Pruebas](#-ambiente-de-pruebas)
8. [Seguridad IP](#seguridad-ip)
9. [Ejemplos de Integraciones formato API - Fase 1](#-ejemplos-de-integraciones-formato-api---fase-1)
10. [Autor](#autor)
11. [Bitácora de Documentación](#bitácora-de-documentación)

## Introducción

### Descripción del Proyecto

Construplaza está buscando establecer una integración API con nuestros proveedores para obtener datos valiosos que se mostrarán en nuestra página web y para validar información antes de formalizar pedidos u órdenes.

## Objetivos del Proyecto

### Objetivos Específicos

- Establecer una conexión segura con la API externa.
- Obtener datos específicos y relevantes para nuestras operaciones.
- Procesar y mostrar estos datos en tiempo real en nuestra página web.
- Validar la información obtenida antes de formalizar cualquier pedido.
- Generar órdenes automatizadas por medio de API.
## Alcance del Proyecto

### Funcionalidades

- **Conexión a la API:** Configurar y establecer una conexión segura con la API externa.
- **Obtención de Datos:** Obtener datos específicos en los llamados a los endpoints detallados en cada fase.
- **Procesamiento:** Asegurar que la información se muestre en tiempo real, es decir, cuando se consulte la API, se obtenga la información real sobre el stock.
- **Envío de Pedidos:** Garantizar una correcta conexión entre el stock validado y el stock del endpoint de la fase 1.
- **Obtención del Catálogo Completo:** Obtener toda la información adicional de los artículos.
- **Habilitar espacio de desarrollo:** Garantizar un espacio de desarrollo para realizar todas las pruebas necesarias.
- **Desplegar a producción:** Después de completar las pruebas en el entorno de desarrollo, proceder al despliegue en producción, asegurando la obtención de todas las credenciales necesarias y que los cambios se implementen de manera segura y sin interrupciones.

## Requisitos del Proyecto

### Requisitos Funcionales

- Establecer una conexión segura con la API del proveedor.
- Ejecutar métodos de HTTP para obtener o enviar todos los datos necesarios.

### Requisitos No Funcionales

- Seguridad en la conexión y manejo de datos.
- Alta disponibilidad y tiempo de respuesta rápido.
- Escalabilidad para manejar un gran volumen de solicitudes.

## 🔄 Fases del Proyecto

### 🔵 Fase 1 - Consultar Existencia y Costos

**Descripción:**
Implementar una funcionalidad que permita consultar la API del proveedor en tiempo real para obtener información actualizada sobre la existencia y costos de los productos.

**Datos Requeridos:**


  | Parámetro                           | Tipo      | Ejemplo                  |
  | :---------------------------------- | :-------- | :----------------------- |
  | `Sku (Código artículo)`             | `string`  | `"PRU01"`                |
  | `Impuestos`                         | `string`  | `"13%"` o `"IVA 13%"`    |
  | `Bodega`                            | `string`  | `"1"`                    |
  | `Cantidad`                          | `int`     | `0`, `1`, `20`           |
  | `Precio`                            | `decimal` | `21843.00`               |
  | `Descuentos`                        | `string`  | `"0%"`                   |
  | `Inactivo`                          | `boolean` | `false` o `true`         |
  | `Moneda (en caso de ser necesario)` | `string`  | `"USD"` o `"CRC"`        |


**Alcance:**
Configurar la conexión a la API, implementar solicitudes GET para obtener los datos necesarios, procesar estos datos y presentarlos en la página web de manera precisa y en tiempo real.

**Descripción gráfica:**

![Fase 1 - Descripción](Documentacion/fase1.png)

### 🔵 Fase 2 - Cargar Pedidos u Órdenes

**Descripción:**
Integrar la funcionalidad para cargar y gestionar pedidos directamente a través de la API proporcionada, permitiendo la automatización del proceso.

**Datos a Enviar:**
- **Información del Pedido:** Detalles completos del pedido, incluyendo número de orden, fecha de creación, y cualquier otra información relevante.

#### Ejemplo

  | Parámetro       | Tipo     | Descripción                  |
  | :-------------- | :------- | :--------------------------- |
  | `idOrden`       | `string` | **Requerido**                |
  | `fechaCreacion` | `string` | Fecha de creación del pedido |


- **Detalles del Producto:** Listado de productos incluidos en el pedido, especificando ID y cantidad.

#### Ejemplo de Cuerpo de Solicitud para la API

El ejemplo muestra solo el listado de productos. El `idOrden` y la `fechaCreacion` acompañan a este listado, ya sea en la ruta del endpoint o en el cuerpo de la solicitud, según lo defina la API del proveedor.

```json
[
    {
        "Sku": "ArticuloPrueba1",
        "Quantity": 2
    },
    {
        "Sku": "ArticuloPrueba2",
        "Quantity": 1
    }
]
```
- **Instrucciones Especiales:** Cualquier instrucción especial relacionada con la entrega o manejo del pedido (de ser necesario).

**Proceso de Integración:**
- **Establecer Conexión Segura:** Configurar y establecer una conexión segura con la API del proveedor, utilizando protocolos de seguridad como HTTPS.
- **Autenticación y Autorización:** Implementar mecanismos de autenticación y autorización para el API.
- **Confirmación y Validación:** Validar que el pedido ha sido recibido y procesado correctamente, y recibir número de pedido, número de orden o factura (respuesta de la API).
- **Manejo de Errores:** Implementar el manejo de errores y excepciones, asegurando que cualquier problema en el envío del pedido sea detectado.

#### Formato de Respuesta en Caso de Error
```json
{
    "Message": "Stock disponible insuficiente para agregar SKU al pedido.",
    "ErrorCode": 23
}
```

#### Catálogo de Errores Comunes de la API

| Código de error | Mensaje                                                                                                                    |
| --------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `1`             | No se envió un parámetro requerido.                                                                                        |
| `23`            | Stock disponible insuficiente para agregar SKU al pedido.                                                                  |
| `26`            | Se solicitan demasiados productos. Se excedió el número máximo de productos permitidos por pedido.                         |
| `27`            | No se puede agregar un producto a un pedido. Este se encuentra desactivado.                                                |
| `28`            | Producto de pedido no autorizado. Por favor consulta el catálogo de productos autorizados.                                 |
| `29`            | Lista de productos vacía.                                                                                                  |
| `30`            | Pedido inválido.                                                                                                           |
| `31`            | Se ha especificado un pedido no válido. Asegúrese de que el pedido tenga el formato correcto en el cuerpo de la solicitud. |
| `33`            | Se ha especificado un pedido no válido. Asegúrese de que el pedido tenga el formato correcto en el cuerpo de la solicitud. |


**Alcance:**
Configurar la conexión a la API, implementar solicitudes POST para enviar los datos de los pedidos u órdenes, manejar confirmaciones y errores, y presentar el estado del pedido u orden en nuestro sistema interno.

**Descripción gráfica:**

![Fase 2 - Descripción](Documentacion/fase2.png)


**Formato de Respuesta Deseado al Crear el Pedido**

```json
{
    "OrderNumber": "1020192",
    "OrderDate": "2024-08-08T00:00:00",
    "Customer": {
        "CustomerId": "012",
        "Name": "CONSTRUPLAZA S.A",
        "Company": {
            "Id": "XCR",
            "Name": "Costa Rica"
        },
        "Country": {
            "Id": "cr",
            "Name": "Costa Rica"
        }
    },
    "CustomerOrderNumber": "PRPRE09"
}
```

| Campo                 | Descripción                         |
| :-------------------- | :---------------------------------- |
| `OrderNumber`         | Número de orden del proveedor.      |
| `CustomerOrderNumber` | Nuestro número de orden de compra.  |

#### 2.1 - Obtener Factura

Es importante destacar que la facturación no es un proceso inmediato. Por lo tanto, necesitamos un endpoint disponible para consultar la factura cuando esté lista. A continuación, se presenta un ejemplo de un endpoint, pero estamos abiertos a adaptarnos según sea necesario. Es fundamental obtener las líneas de la factura para poder comparar los detalles de nuestra orden de compra con los de la factura.

#### Parámetros

| Parámetro        | Tipo     | Descripción                                                |
| :--------------- | :------- | :--------------------------------------------------------- |
| `Pedido u Orden` | `string` | **Requerido**. Identificador del pedido u orden de compra. |

#### Ejemplo de respuesta de la API con factura disponible
```json
{
    "facturapedido": "00100001010000088888",
    "existe": true,
    "articulos": [
        {
            "articulo": "PRUEBA01",
            "cantidad": 48.00000000,
            "precio_unitario": 5550.00000000
        },
        {
            "articulo": "PRUEBA02",
            "cantidad": 24.00000000,
            "precio_unitario": 6250.00000000
        }
    ]
}
```

#### Ejemplo de respuesta de la API sin factura
```json
{
    "facturapedido": "No existe factura",
    "existe": false,
    "articulos": []
}
```






###  🔵 Fase 3 - Consultar Productos de Pedido Especial

**Descripción:**
Habilitar consultas para productos que no se manejan regularmente en nuestro inventario y que requieren pedidos especiales, obteniendo y almacenando información detallada sobre estos artículos.

**Datos Requeridos:**
- Sku
- Descripción del artículo
- Descripción del Fabricante
- Fotos
- Categorías
- Tipos
- Marca
- ID del Artículo
- Toda información extra del artículo que pueda ser importante

**Descripción Detallada:**
Implementar una funcionalidad que permita obtener un catálogo completo de los artículos que manejan para guardarlos y mostrarlos como artículos especiales.

**Alcance:**
Configurar la conexión a la API, implementar solicitudes GET para obtener datos sobre productos, manejar la información en múltiples endpoints o en un único endpoint, y almacenar y actualizar esta información en nuestro sistema.

## Flujo de Trabajo General para Nuestra Implementación

### Descripción del Flujo

Al iniciarse el sistema, se establece una conexión segura con la API del proveedor utilizando credenciales almacenadas de manera segura. El sistema realiza solicitudes GET para obtener los datos necesarios, como información de productos, disponibilidad de inventario y precios actualizados. Una vez obtenidos los datos, estos se procesan y se formatean adecuadamente para ser presentados en tiempo real en nuestra página web. La información se actualiza periódicamente para asegurar que siempre se muestra la información más reciente y precisa. Antes de que un pedido sea formalizado, el sistema realiza una validación de los datos obtenidos mediante la API para asegurar su precisión y veracidad.

## 🔧 Ambiente de Pruebas

### Infraestructura

Configurar un entorno que replique el entorno de producción en cuanto a software y a nivel de API.

### Datos de Prueba

Cargar datos de prueba que imiten el volumen y la variedad de datos en producción para validar el comportamiento bajo condiciones similares.

### Seguridad y Accesos

Implementar medidas de seguridad y configuración de accesos adecuados.

Habilitar este entorno hasta finalizar todas las pruebas respectivas.

## Seguridad IP

Para asegurar un acceso seguro y controlado entre ambos desarrollos, recomendamos implementar una lista blanca de IPs, limitando las invocaciones a las direcciones IP específicas que proveemos. Esta medida permitirá a su equipo restringir el acceso únicamente a fuentes confiables, mejorando la protección general del sistema.

### Beneficios de la lista blanca de IPs
- Mayor Control de Seguridad: Limitar el acceso a direcciones IP específicas reduce el riesgo de accesos no autorizados.

- Cumplimiento de Normativas: La lista blanca contribuye a cumplir con estándares de seguridad y privacidad al restringir conexiones únicamente a IPs aprobadas.

- Prevención de Accesos Involuntarios: Al configurar la lista blanca en cada ambiente, su equipo puede asegurarse de que solo nuestras IPs autorizadas puedan realizar invocaciones en sus sistemas.

### Procedimiento recomendado para implementar la lista blanca
- Agregar Nuestras IPs Autorizadas: Recomendamos añadir nuestras IPs a una lista blanca en sus ambientes de desarrollo y producción.

- Configurar Monitoreo: Configuren alertas para detectar cualquier intento de acceso desde IPs no autorizadas.

- Proceso de Actualización de IPs: Si en el futuro es necesario cambiar alguna IP, les notificaremos oportunamente para que actualicen la lista.


### Lista Blanca de IPs

#### Producción

| IP |
| :-------- |
| `34.200.99.163`      |
| `3.225.20.67`      | 


#### Desarrollo

| IP  |
| :-------- |
| `44.213.198.105`      |




## 💡 Ejemplos de Integraciones formato API - Fase 1

### Información en un mismo endpoint

```json
[
    {
        "idProducto": "PRUEBA01",
        "nombrearticulo": "ARTICULO PRUEBA",
        "impuesto": "IVA 13%",
        "bodega": "4",
        "cantidad": 10,
        "price": 3239.00,
        "descuento": "0%",
        "inactivo": false,
        "moneda": "CRC"
    },
    {
        "idProducto": "PRUEBA02",
        "nombrearticulo": "ARTICULO PRUEBA 2",
        "impuesto": "IVA 13%",
        "bodega": "2",
        "cantidad": 10,
        "price": 5039.00,
        "descuento": "0%",
        "inactivo": false,
        "moneda": "CRC"
    }
]
```


### Información en diferentes endpoints
#### Endpoint de catálogo

```json
[
    {
        "Sku": "ArticuloPrueba1",
        "Description": "Cable potencia 1",
        "Type": "Physical",
        "Manufacturer": {
            "ManufacturerId": "acv",
            "Description": "Accvent"
        },
        "Brand": {
            "ManufacturerId": "acv",
            "BrandId": "xtk",
            "Description": "Xtech"
        },
        "Category": {
            "CategoryId": "cac",
            "Description": "Accesorios para Computadores",
            "Subcategories": [
                {
                    "CategoryId": "cac.cable",
                    "Description": "Cables",
                    "Subcategories": []
                }
            ]
        }
    },
    {
        "Sku": "ArticuloPrueba2",
        "Description": "Cable potencia 2",
        "Type": "Physical",
        "Manufacturer": {
            "ManufacturerId": "acv",
            "Description": "Accvent"
        },
        "Brand": {
            "ManufacturerId": "acv",
            "BrandId": "xtk",
            "Description": "Xtech"
        },
        "Category": {
            "CategoryId": "cac",
            "Description": "Accesorios para Computadores",
            "Subcategories": [
                {
                    "CategoryId": "cac.cable",
                    "Description": "Cables",
                    "Subcategories": []
                }
            ]
        }
    }
]
```

#### Endpoint de precios

```json
[
    {
        "Sku": "ArticuloPrueba1",
        "Price": {
            "CurrencyId": "USD",
            "UnitPrice": 5.6598
        }
    },
    {
        "Sku": "ArticuloPrueba2",
        "Price": {
            "CurrencyId": "USD",
            "UnitPrice": 174.2268
        }
    }
]
```

#### Endpoint de stock

```json
[
    {
        "Sku": "ArticuloPrueba1",
        "InStock": 10
    },
    {
        "Sku": "ArticuloPrueba2",
        "InStock": 10
    }
]
```

## Autor

Esto es una propuesta de trabajo desarrollada por CONSTRUPLAZA S.A

<img src="Documentacion/construplaza_logo.png" alt="Logo Construplaza" width="200"/>


## Bitácora de Documentación

- 2024-08-12 - Andrei Calderón Molina - Creación de la documentación necesaria para el proyecto de APIs

- 2024-08-16 - Andrei Calderón Molina - Revisión y ajuste de documentación
  
- 2024-10-11 - Andrei Calderón Molina - Agregación del endpoint necesario para la consulta de facturación

- 2024-11-05 - Andrei Calderón Molina - Agregación de ajuste de seguridad, lista blanca de ip

- 2026-11-24 - Andrei Calderón Molina - Corrección de ejemplos JSON, catálogo de errores, ortografía e índice
