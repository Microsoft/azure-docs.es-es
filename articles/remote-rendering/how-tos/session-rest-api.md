---
title: La API REST de administración de sesión
description: Describe cómo administrar sesiones
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: article
ms.openlocfilehash: 0af9d6906e038a4b9285a2c302fc0c98345fdbd9
ms.sourcegitcommit: 70ee014d1706e903b7d1e346ba866f5e08b22761
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90023761"
---
# <a name="use-the-session-management-rest-api"></a>Uso de la API REST de administración de sesión

Para usar la funcionalidad de Azure Remote Rendering, debe crear una *sesión*. Cada sesión se corresponde con una máquina virtual (VM) que se va a asignar en Azure y espera a que se conecte un dispositivo cliente. Cuando se conecta un dispositivo, la máquina virtual representa los datos solicitados y sirve el resultado como una secuencia de vídeo. Durante la creación de la sesión, se elige el tipo de servidor para la ejecución, que determina los precios. Una vez que la sesión ya no se necesita, debe detenerse. Si no se detiene manualmente, se apagará de forma automática cuando expire el *tiempo de concesión* de la sesión.

Proporcionamos un script de PowerShell en el [repositorio de ejemplos de ARR](https://github.com/Azure/azure-remote-rendering) en la carpeta *Scripts* que se denomina *RenderingSession.ps1* y muestra el uso de nuestro servicio. El script y su configuración se describen aquí: [Scripts de PowerShell de ejemplo](../samples/powershell-example-scripts.md)

> [!TIP]
> Los comandos de PowerShell que se muestran en esta página están diseñados para complementarse entre sí. Si ejecuta todos los scripts en secuencia en el mismo símbolo del sistema de PowerShell, se crearán uno a partir del otro.

## <a name="regions"></a>Regions

Consulte en la [lista de regiones disponibles](../reference/regions.md) las direcciones URL base a las que enviar las solicitudes.

En los siguientes scripts de ejemplo, elegimos la región *westus2*.

### <a name="example-script-choose-an-endpoint"></a>Script de ejemplo: Elección de un punto de conexión

```PowerShell
$endPoint = "https://remoterendering.westus2.mixedreality.azure.com"
```

## <a name="accounts"></a>Cuentas

Si no tiene una cuenta de Remote Rendering, [cree una](create-an-account.md). Cada recurso se identifica mediante un *accountId*, que se usa en las API de sesión.

### <a name="example-script-set-accountid-and-accountkey"></a>Script de ejemplo: Establecimiento de accountId y accountKey

```PowerShell
$accountId = "********-****-****-****-************"
$accountKey = "*******************************************="
```

## <a name="common-request-headers"></a>Encabezados de solicitud comunes

* El encabezado *Authorization* debe tener el valor "`Bearer TOKEN`", donde "`TOKEN`" es el token de autenticación [devuelto por el Servicio de token seguro](tokens.md).

### <a name="example-script-request-a-token"></a>Script de ejemplo: Solicitud de un token

```PowerShell
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
$webResponse = Invoke-WebRequest -Uri "https://sts.mixedreality.azure.com/accounts/$accountId/token" -Method Get -ContentType "application/json" -Headers @{ Authorization = "Bearer ${accountId}:$accountKey" }
$response = ConvertFrom-Json -InputObject $webResponse.Content
$token = $response.AccessToken;
```

## <a name="common-response-headers"></a>Encabezados de respuesta comunes

* El equipo de producto puede usar el encabezado *MS-CV* para realizar el seguimiento de la llamada dentro del servicio.

## <a name="create-a-session"></a>Creación de una sesión

Este comando permite crear una sesión. Devuelve el identificador de la nueva sesión. Necesita el identificador de sesión para todos los demás comandos.

| URI | Método |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions/create | POST |

**Cuerpo de la solicitud:**

* maxLeaseTime (intervalo de tiempo): valor de tiempo de espera en el que la máquina virtual se retirará automáticamente.
* models (matriz): direcciones URL del contenedor de recursos para cargar previamente
* tamaño (cadena): tamaño del servidor que se va a configurar ([ **"estándar"** ](../reference/vm-sizes.md) o [ **"premium"** ](../reference/vm-sizes.md)). Consulte las [limitaciones de tamaño](../reference/limits.md#overall-number-of-polygons) específicas.

**Respuestas:**

| status code | carga de JSON | Comentarios |
|-----------|:-----------|:-----------|
| 202 | - sessionId: GUID | Correcto |

### <a name="example-script-create-a-session"></a>Script de ejemplo: Creación de una sesión

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/create" -Method Post -ContentType "application/json" -Body "{ 'maxLeaseTime': '4:0:0', 'models': [], 'size': 'standard' }" -Headers @{ Authorization = "Bearer $token" }
```

Salida de ejemplo:

```PowerShell
StatusCode        : 202
StatusDescription : Accepted
Content           : {"sessionId":"d31bddca-dab7-498e-9bc9-7594bc12862f"}
RawContent        : HTTP/1.1 202 Accepted
                    MS-CV: 5EqPJ1VdTUakDJZc6/ZhTg.0
                    Content-Length: 52
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 09 May 2019 16:17:50 GMT
                    Location: accounts/11111111-1111-1111-11...
Forms             : {}
Headers           : {[MS-CV, 5EqPJ1VdTUakDJZc6/ZhTg.0], [Content-Length, 52], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 09 May 2019 16:17:50 GMT]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 52
```

### <a name="example-script-store-sessionid"></a>Script de ejemplo: sessionId del almacén

La respuesta de la solicitud anterior incluye un **sessionId**, que necesita para todas las solicitudes de seguimiento.

```PowerShell
$sessionId = "d31bddca-dab7-498e-9bc9-7594bc12862f"
```

## <a name="modify-and-query-session-properties"></a>Modificación y consulta de las propiedades de sesión

Hay algunos comandos para consultar o modificar los parámetros de las sesiones existentes.

> [!CAUTION]
> En el caso de todas las llamadas REST, enviar estos comandos con demasiada frecuencia hará que el servidor se limite y devuelva un error con el tiempo. El código de error en este caso es 429 ("demasiadas solicitudes"). Como regla general, debería haber un retraso de entre **5 y 10 segundos entre las llamadas subsiguientes**.

### <a name="update-session-parameters"></a>Actualización de los parámetros de sesión

Este comando actualiza los parámetros de una sesión. Actualmente solo se puede ampliar el tiempo de concesión de una sesión.

> [!IMPORTANT]
> El tiempo de concesión siempre se proporciona como un tiempo total desde el inicio de la sesión. Esto significa que, si ha creado una sesión con un tiempo de concesión de una hora y desea ampliar su tiempo de concesión durante otra hora, tiene que actualizar su valor de maxLeaseTime a dos horas.

| URI | Método |
|-----------|:-----------|
| /v1/accounts/*accountID*/sessions/*sessionId* | PATCH |

**Cuerpo de la solicitud:**

* maxLeaseTime (intervalo de tiempo): valor de tiempo de espera en el que la máquina virtual se retirará automáticamente.

**Respuestas:**

| status code | carga de JSON | Comentarios |
|-----------|:-----------|:-----------|
| 200 | | Correcto |

#### <a name="example-script-update-a-session"></a>Script de ejemplo: actualización de una sesión

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId" -Method Patch -ContentType "application/json" -Body "{ 'maxLeaseTime': '5:0:0' }" -Headers @{ Authorization = "Bearer $token" }
```

Salida de ejemplo:

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : {}
RawContent        : HTTP/1.1 200 OK
                    MS-CV: Fe+yXCJumky82wuoedzDTA.0
                    Content-Length: 0
                    Date: Thu, 09 May 2019 16:27:31 GMT


Headers           : {[MS-CV, Fe+yXCJumky82wuoedzDTA.0], [Content-Length, 0], [Date, Thu, 09 May 2019 16:27:31 GMT]}
RawContentLength  : 0
```

### <a name="get-active-sessions"></a>obtención de sesiones activas

Este comando devuelve una lista de las sesiones activas.

| URI | Método |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions | GET |

**Respuestas:**

| status code | carga de JSON | Comentarios |
|-----------|:-----------|:-----------|
| 200 | - sessions: matriz de propiedades de sesión | Vea la sección "Obtención de las propiedades de sesión" para obtener una descripción de las propiedades de la sesión. |

#### <a name="example-script-query-active-sessions"></a>Script de ejemplo: consulta de las sesiones activas

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions" -Method Get -Headers @{ Authorization = "Bearer $token" }
```

Salida de ejemplo:

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : []
RawContent        : HTTP/1.1 200 OK
                    MS-CV: WfB9Cs5YeE6S28qYkp7Bhw.1
                    Content-Length: 15
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 25 Jul 2019 16:23:50 GMT

                    {"sessions":[]}
Forms             : {}
Headers           : {[MS-CV, WfB9Cs5YeE6S28qYkp7Bhw.1], [Content-Length, 2], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 25 Jul 2019 16:23:50 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 2
```

### <a name="get-sessions-properties"></a>obtención de las propiedades de la sesión

Este comando devuelve información sobre una sesión, como el nombre de host de la máquina virtual.

| URI | Método |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions/*sessionId*/properties | GET |

**Respuestas:**

| status code | carga de JSON | Comentarios |
|-----------|:-----------|:-----------|
| 200 | - message: string<br/>- sessionElapsedTime: timespan<br/>- sessionHostname: string<br/>- sessionId: string<br/>- sessionMaxLeaseTime: timespan<br/>- sessionSize: enum<br/>- sessionStatus: enum | enum sessionStatus { starting, ready, stopping, stopped, expired, error}<br/>Si el estado es "error" o "expired" (expirado), el mensaje contendrá más información |

#### <a name="example-script-get-session-properties"></a>Script de ejemplo: Obtiene propiedades de una sesión.

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId/properties" -Method Get -Headers @{ Authorization = "Bearer $token" }
```

Salida de ejemplo:

```PowerShell
StatusCode        : 200
StatusDescription : OK
Content           : {"message":null,"sessionElapsedTime":"00:00:01","sessionHostname":"5018fee8-817e-4366-9179-556af79a4240.remoterenderingvm.westeurope.mixedreality.azure.com","sessionId":"e13d2c44-63e0-4591-991e-f9e05e599a93","sessionMaxLeaseTime":"04:00:00","sessionStatus":"Ready"}
RawContent        : HTTP/1.1 200 OK
                    MS-CV: CMXegpZRMECH4pbOW2j5GA.0
                    Content-Length: 60
                    Content-Type: application/json; charset=utf-8
                    Date: Thu, 09 May 2019 16:30:38 GMT

                    {"message":null,...
Forms             : {}
Headers           : {[MS-CV, CMXegpZRMECH4pbOW2j5GA.0], [Content-Length, 60], [Content-Type, application/json;
                    charset=utf-8], [Date, Thu, 09 May 2019 16:30:38 GMT]}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 60
```

## <a name="stop-a-session"></a>detención de una sesión

Este comando permite detener una sesión. La máquina virtual asignada se recuperará poco después.

| URI | Método |
|-----------|:-----------|
| /v1/accounts/*accountId*/sessions/*sessionId* | Delete |

**Respuestas:**

| status code | carga de JSON | Comentarios |
|-----------|:-----------|:-----------|
| 204 | | Correcto |

### <a name="example-script-stop-a-session"></a>Script de ejemplo: detención de una sesión

```PowerShell
Invoke-WebRequest -Uri "$endPoint/v1/accounts/$accountId/sessions/$sessionId" -Method Delete -Headers @{ Authorization = "Bearer $token" }
```

Salida de ejemplo:

```PowerShell
StatusCode        : 204
StatusDescription : No Content
Content           : {}
RawContent        : HTTP/1.1 204 No Content
                    MS-CV: YDxR5/7+K0KstH54WG443w.0
                    Date: Thu, 09 May 2019 16:45:41 GMT


Headers           : {[MS-CV, YDxR5/7+K0KstH54WG443w.0], [Date, Thu, 09 May 2019 16:45:41 GMT]}
RawContentLength  : 0
```

## <a name="next-steps"></a>Pasos siguientes

* [Scripts de PowerShell de ejemplo](../samples/powershell-example-scripts.md)
