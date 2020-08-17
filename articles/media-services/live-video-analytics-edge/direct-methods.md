---
title: 'Uso de métodos directos en Live Video Analytics on IoT Edge: Azure'
description: Live Video Analytics on IoT Edge expone varios métodos directos. Los métodos directos se basan en las convenciones descritas en este tema.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: ed7cec7b8513044c2bf9b24600b8d9f42a485aae
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091834"
---
# <a name="direct-methods"></a>Métodos directos

Live Video Analytics on IoT Edge expone varios métodos directos que se pueden invocar desde IoT Hub. Los métodos directos representan una interacción solicitud-respuesta con un dispositivo similar a una llamada HTTP en la cual se completan correctamente o generan un error de inmediato (tras un tiempo de espera que especifica el usuario). Este enfoque es útil en escenarios donde el curso de una acción inmediata es distinto en función de si el dispositivo pudo responder. Para obtener más información, vea el artículo [Conocimiento e invocación de los métodos directos de IoT Hub](../../iot-hub/iot-hub-devguide-direct-methods.md).

En este tema se describen estos métodos y convenciones.

## <a name="conventions"></a>Convenciones

Los métodos directos se basan en las siguientes convenciones:

1. Los métodos directos tienen una versión especificada en el formato MAJOR.MINOR (como se muestra en el ejemplo siguiente):

    ```
    {
        "methodName": "GraphTopologySet",
        "payload": {
            // API version matches matches module MAJOR.MINOR version
            "@apiVersion": "1.0"
            //payload here…
        }
    }
    ```

2. Cada una de las versiones del módulo Live Video Analytics on IoT Edge admitirá todos los métodos directos hasta su versión actual. Por ejemplo, la versión 1.3 del módulo admitirá los métodos directos con las versiones 1.3, 1.2, 1.1 y 1.0.
3. Todos los métodos directos son sincrónicos.
4. Los resultados de error siguen el esquema de errores de OData.
5. Los nombres presentan las restricciones siguientes:
    
    * Solo pueden estar formados por caracteres alfanuméricos y guiones, siempre y cuando no empiecen y acaben con un guion.
    * No deben tener espacios.
    * Pueden ser de 32 caracteres como máximo.

### <a name="example"></a>Ejemplo

```
{
  "status": 400,
  "payload": {
    "error": {
      "code": "BadRequest",
      "message": "Graph instance is not found."
    }
  }
}
```    

### <a name="top-level-error-codes"></a>Códigos de error de nivel superior     

|Estado |Código   |Message|
|---|---|---|
|400|   BadRequest| La solicitud no es válida.|
|400|   InvalidResource|    El recurso no es válido|
|400|   InvalidVersion| La versión de la API no es válida|
|402|   ConnectivityRequired    |El módulo de Edge requiere conectividad en la nube para funcionar correctamente|
|404|   NotFound    |No se encontró el recurso|
|409|   Conflicto|   Conflicto de operación|
|500|   InternalServerError|    Error interno del servidor|
|503|   ServerBusy| Servidor ocupado|

### <a name="detailed-error-codes"></a>Códigos de error detallados

Los errores de validación detallados, por ejemplo, las validaciones de módulos de grafos, se agregan como detalles del error:

```
{
  "status": 400,
  "payload": {
    "error": {
      "code": "InvalidResource",
      "message": "The resource format is invalid.",
      "details": [
        {
          "code": "RtspEndpointUrlInvalidScheme",
          "target": "$.Properties.Sources[0]",
          "message": "Uri scheme 'http' is not valid for an RTSP source endpoint. Valid values are: rtsp"
        },
        {
          "code": "PropertyValueInvalidPattern",
          "target": "$.Properties.Sinks[0].AssetNamePattern",
          "message": "The value must match the regular expression '^[^<>%&:\\\\\\/?*+.']{1,260}$'"
        }
      ]
    }
  }
}
}
```

|Estado|    Código detallado   |Descripción|
|---|---|---|
|400|   GraphValidationError|   Errores generales de grafos, como ciclos, particiones, etc.|
|400|   ModuleValidationError|  Errores de validación específicos del módulo.|
|409|   GraphTopologyInUse| Una o varias instancias de grafo todavía hacen referencia a la topología de grafo.|
|409|   OperationNotAllowedInState| La operación solicitada no se puede realizar en el estado actual.|
|409|   ResourceValidationError|    El recurso al que se hace referencia no tiene un estado válido.|

## <a name="details"></a>Detalles  

### <a name="graphtopologyget"></a>GraphTopologyGet

Este método directo recupera una topología de un solo grafo.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphTopologyGet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "CaptureAndRecord"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": {
        "name": "CaptureAndRecord",
        "properties": {
            // Complete graph topology
        }
    }
}
```

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Entidad encontrada|  200 |N/D
|Errores de usuario generales    |Intervalo 400  ||
|No se ha encontrado la entidad   |404        ||
|Errores generales del servidor| Intervalo 500       ||

### <a name="graphtopologyset"></a>GraphTopologySet

Crea una topología única si no hay ninguna con el nombre especificado o actualiza una que ya existe con el mismo nombre.

Aspectos clave:

* La topología se puede actualizar libremente si no hay ningún grafo que haga referencia a ella.
* La topología se puede actualizar libremente si todos los grafos de referencia se desactivan, siempre y cuando se cumpla lo siguiente:

    * Los parámetros recién agregados tienen valores predeterminados.
    * Ningún grafo hace referencia a los parámetros quitados.
* No se permite la actualización de topologías si hay grafos activos.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphTopologySet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "CaptureAndRecord",
        "properties": {
            // Desired graph topology properties
        }
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 201,
    "payload": {
        "name": " CaptureAndRecord ",
        "properties": {
            // Complete graph topology
        }
    }
}
```

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
Entidad existente actualizada |200|   N/D|
Nueva entidad creada  |201|   N/D|
Errores de usuario generales |Intervalo 400  ||
Errores de validación de grafos |400    |GraphValidationError|
Errores de validación de módulos|   400 |ModuleValidationError|
Errores generales del servidor   |Intervalo 500  ||

### <a name="graphtopologydelete"></a>GraphTopologyDelete

Elimina la topología de un solo grafo.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphTopologyDelete",
    "payload": {
        "@apiVersion": "1.0",
        "name": "CaptureAndRecord"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": null
}
```

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Entidad eliminada|    200|    N/D|
|No se ha encontrado la entidad|  204|    N/D|
|Errores de usuario generales|   Intervalo 400   ||
|Una o varias instancias de grafo hacen referencia a la topología de grafo| 409 |GraphTopologyInUse|
|Errores generales del servidor| Intervalo 500   ||

### <a name="graphtopologylist"></a>GraphTopologyList

Recupera una lista de todas las topologías de grafo que coinciden con los criterios de filtro.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphTopologyList",
    "payload": {
        // Required
        "@apiVersion": "1.0",
        
        // Optional
        "@continuationToken": "xxxxx",    
        "@query": "$orderby=name asc"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": {
        "@continuationToken": "xxxx",
        "value": [
            {
                "name": "Capture & Record",
                // ...
            },
            {
                "name": "Capture, Analyze & Record",
                // ...
            }
        ]
    }
}
```

#### <a name="filter-support"></a>Compatibilidad con filtros

|Operación      |Campo   |Operadores|
|---|---|---|
|$orderby|name  |asc|


#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Correcto|   200 |N/D|
|Errores de usuario generales|   Intervalo 400   ||
|Errores generales del servidor| Intervalo 500   ||

### <a name="graphinstanceget"></a>GraphInstanceGet

Recupera una única instancia de grafo.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphInstanceGet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": {
        "name": "Camera001",
        "properties": {
            // Complete graph instance
        }
    }
}
```

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Entidad encontrada   |200|   N/D|
|Errores de usuario generales|   Intervalo 400   ||
|No se ha encontrado la entidad|  404 ||
|Errores generales del servidor| Intervalo 500   ||

### <a name="graphinstanceset"></a>GraphInstanceSet

Crea una única instancia de grafo si no hay ninguna con el nombre especificado, o bien actualiza una que ya existe con el mismo nombre.

Aspectos clave:

* La instancia de grafo se puede actualizar libremente si su estado es "Desactivado".

    * El grafo se vuelve a validar con cada actualización.
* Las actualizaciones de instancias de grafo están parcialmente restringidas si el estado del grafo no es "Inactivo".
* No se permiten actualizaciones de instancias de grafo en los grafos activos.

#### <a name="request"></a>Solicitud

```
{
"methodName": "GraphInstanceSet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001",
        "properties": {
            // Desired graph instance properties
        }
    }
}
```

#### <a name="response"></a>Response

````
{
    "status": 201,
    "payload": {
        "name": " Camera001",
        "properties": {
            // Complete graph instance
        }   
    }
}
````

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Entidad existente actualizada    |200    |N/D|
|Nueva entidad creada|    201 |N/D|
|Errores de usuario generales|   Intervalo 400   ||
|Errores de validación de grafos    |400|   GraphValidationError|
|Errores de validación de módulos|  400 |ModuleValidationError|
|Errores de validación de recursos |409    |ResourceValidationError|
|Errores generales del servidor  |Intervalo 500||    

### <a name="graphinstancedelete"></a>GraphInstanceDelete

Elimina una única instancia de grafo.

Aspectos clave:

* Solo se pueden eliminar los grafos desactivados.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphInstanceDelete",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": null
}
```

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Grafo eliminado correctamente|    200|    N/D|
|No se ha encontrado el grafo|   204|    N/D|
|Errores de usuario generales    |Intervalo 400  ||
|El estado del grafo no es "Detenido"    |409    |OperationNotAllowedInState|
|Errores generales del servidor| Intervalo 500   ||

### <a name="graphinstancelist"></a>GraphInstanceList

Es similar a GraphTopologyList. Se puede usar para enumerar las instancias de grafo.
Recupera una lista de todas las instancias de grafo que coinciden con los criterios de filtro.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphInstanceList",
    "payload": {
        // Required
        "@apiVersion": "1.0",
        
        // Optional
        "@continuationToken": "xxxxx",    
        "@query": "$orderby=name asc"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": {
        "@continuationToken": "xxxx",
        "value": [
            {
                "name": "Capture & Record",
                // ...
            },
            {
                "name": "Capture, Analyze & Record",
                // ...
            }
        ]
    }
}
```

#### <a name="filter-support"></a>Compatibilidad con filtros

|Operación  |   Campo|   Operadores|
|---|---|---|
|$orderby|  name|   asc|

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Correcto    |200    |N/D|
|Errores de usuario generales|   Intervalo 400   ||
|Errores generales del servidor| Intervalo 500   ||

### <a name="graphinstanceactivate"></a>GraphInstanceActivate

Activa una única instancia de grafo. 

Aspectos clave

* El método solo se devuelve cuando el grafo está activado. 
* El grafo asume el estado "Activando" mientras se activa.

    * Una operación List o Get en el grafo lo devolverá en el estado adecuado.
* Idempotencia:

    * Al iniciar un grafo con el estado "Activando", se produce el mismo comportamiento que si el grafo estuviera desactivado (es decir, se bloquea la llamada hasta que el grafo esté activado).
    * La activación de un grafo con el estado "Activo" se devuelve correctamente de inmediato.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphInstanceActivate",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": null
}
```

#### <a name="status-codes"></a>Códigos de estado

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Grafo activado correctamente   |200    |N/D|
|Nueva entidad creada |201|   N/D|
|Errores de usuario generales    |Intervalo 400  ||
|Errores de validación de módulos   |400|   ModuleValidationError|
|Errores de validación de recursos|    409|    ResourceValidationError|
|El grafo tiene el estado Desactivando |409    |OperationNotAllowedInState|
|Errores generales del servidor| Intervalo 500   ||

### <a name="graphinstancedeactivate"></a>GraphInstanceDeactivate

Desactiva una única instancia de grafo. Al desactivar un grafo, se desactiva correctamente el procesamiento de medios y se garantiza que todos los eventos y medios almacenados de forma transitoria en Edge se confirmen en la nube, si procede.

Aspectos clave:

* El método solo se devuelve cuando el grafo está desactivado.
* El grafo asume el estado "Desactivando" mientras se desactiva.

    * Una operación List o Get en el grafo lo devolverá en el estado adecuado.
    * Solo se detiene el método cuando todos los medios están cargados en la nube.
* Idempotencia:

    * Al desactivar un grafo con el estado "Desactivando", se produce el mismo comportamiento que si el grafo estuviera activado (es decir, se bloquea la llamada hasta que el grafo esté desactivado).
    * La desactivación de un grafo con el estado "Inactivo" se devuelve correctamente de inmediato.

#### <a name="request"></a>Solicitud

```
{
    "methodName": "GraphInstanceDeactivate",
    // A high response timeout is recommended here since there might be data movement    // being executed as part of this operation
    "responseTimeoutInSeconds": 300,
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>Response

```
{
    "status": 200,
    "payload": null
}
```

|Condición  |status code    |Código de error detallado|
|---|---|---|
|Grafo activado correctamente   |200|   N/D|
|Nueva entidad creada |201|   N/D|
|Errores de usuario generales    |Intervalo 400  ||
|El grafo tiene el estado Activando   |409|   OperationNotAllowedInState|
|Errores generales del servidor  |Intervalo 500  ||

## <a name="next-steps"></a>Pasos siguientes

[Esquema de configuración de módulos gemelos](module-twin-configuration-schema.md)
