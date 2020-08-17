---
title: HTTP Data Collector API de Azure Monitor | Microsoft Docs
description: Puede usar HTTP Data Collector API de Azure Monitor para agregar datos POST JSON en un área de trabajo de Log Analytics desde cualquier cliente que pueda llamar a la API REST. Este artículo describe cómo utilizar la API y contiene algunos ejemplos de cómo publicar datos mediante diferentes lenguajes de programación.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/14/2020
ms.openlocfilehash: 40f688d6acd1714999210e67567d25faa14c5d6e
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87384861"
---
# <a name="send-log-data-to-azure-monitor-with-the-http-data-collector-api-public-preview"></a>Envío de datos de registro a Azure Monitor con HTTP Data Collector API (versión preliminar pública)
En este artículo se muestra cómo utilizar HTTP Data Collector API para enviar datos de registro a Azure Monitor desde un cliente de API REST.  Describe cómo dar formato a los datos recopilados por el script o la aplicación, incluirlos en una solicitud y hacer que esa solicitud la autorice Azure Monitor.  Se proporcionan ejemplos de PowerShell, C# y Python.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

> [!NOTE]
> HTTP Data Collector API de Azure Monitor se encuentra en versión preliminar pública.

## <a name="concepts"></a>Conceptos
Puede usar HTTP Data Collector API para enviar datos a un área de trabajo de Log Analytics en Azure Monitor desde cualquier cliente que pueda llamar a una API REST.  Podría tratarse de un runbook en Azure Automation que recopila datos de administración de Azure u otra nube, o podría ser un sistema de administración alternativo que usa Azure Monitor para consolidar y analizar los datos de registro.

Todos los datos del área de trabajo de Log Analytics se almacenan como un registro con un tipo de registro concreto.  Se da formato a los datos para enviarlos a la API del recopilador de datos HTTP como varios registros en JSON.  Al enviar los datos, se crea un registro individual en el repositorio para cada registro en la carga de solicitud.


![Información general del recopilador de datos HTTP](media/data-collector-api/overview.png)



## <a name="create-a-request"></a>Creación de una solicitud
Para usar la API de recopilador de datos de HTTP, cree una solicitud POST que incluya los datos que se van a enviar en notación de objetos JavaScript (JSON).  Las siguientes tres tablas enumeran los atributos que son necesarios para cada solicitud. Se describirá cada atributo con más detalle más adelante en el artículo.

### <a name="request-uri"></a>URI de solicitud
| Atributo | Propiedad |
|:--- |:--- |
| Método |POST |
| URI |https://\<CustomerId\>.ods.opinsights.azure.com/api/logs?api-version=2016-04-01 |
| Tipo de contenido |application/json |

### <a name="request-uri-parameters"></a>Parámetros de URI de solicitud
| Parámetro | Descripción |
|:--- |:--- |
| CustomerID |El identificador único del área de trabajo de Log Analytics. |
| Recurso |Nombre de recurso de la API: /api/logs. |
| Versión de API |Versión de la API que se usará con esta solicitud. Actualmente, es 2016-04-01. |

### <a name="request-headers"></a>Encabezados de solicitud
| Encabezado | Descripción |
|:--- |:--- |
| Authorization |Firma de la autorización. Más adelante en este artículo, encontrará información acerca de cómo crear un encabezado HMAC-SHA256. |
| Log-Type |Especifica el tipo de registro de los datos que se envían. Solo puede contener letras, números y guiones bajos (_) y no puede superar los 100 caracteres. |
| x-ms-date |Fecha en que se procesó la solicitud, en formato RFC 1123. |
| x-ms-AzureResourceId | Identificador de recurso del recurso de Azure con el que se deben asociar los datos. Esto rellena la propiedad [_ResourceId](log-standard-properties.md#_resourceid) y permite que los datos se incluyan en las consultas de [contexto del recurso](design-logs-deployment.md#access-mode). Si no se especifica este campo, los datos no se incluirán en las consultas de contexto del recurso. |
| time-generated-field | Nombre de un campo en los datos que contiene la marca de tiempo del elemento de datos. Si especifica un campo, su contenido se usa para **TimeGenerated**. Si no se especifica este campo, el valor predeterminado de **TimeGenerated** es el tiempo que el mensaje se ingiere. El contenido del campo de mensaje debe seguir el formato ISO 8601 AAAA-MM-DDThh:mm:ssZ. |

## <a name="authorization"></a>Authorization
Cualquier solicitud a HTTP Data Collector API de Azure Monitor debe incluir un encabezado de autorización. Para autenticar una solicitud, debe firmar la solicitud con la clave principal o secundaria del área de trabajo que realiza la solicitud. Después, pase esa firma como parte de la solicitud.   

Este es el formato del encabezado de autorización:

```
Authorization: SharedKey <WorkspaceID>:<Signature>
```

*WorkspaceID* es el identificador único del área de trabajo de Log Analytics. *Signature* es un [código de autenticación de mensajes basado en hash (HMAC)](/dotnet/api/system.security.cryptography.hmacsha256?view=netcore-3.1) que se construye a partir de la solicitud y después se procesa mediante el [algoritmo SHA256](/dotnet/api/system.security.cryptography.sha256?view=netcore-3.1). Luego se codifica con la codificación Base64.

Use este formato para codificar la cadena de firma **SharedKey**:

```
StringToSign = VERB + "\n" +
                  Content-Length + "\n" +
               Content-Type + "\n" +
                  x-ms-date + "\n" +
                  "/api/logs";
```

Este es un ejemplo de una cadena de firma:

```
POST\n1024\napplication/json\nx-ms-date:Mon, 04 Apr 2016 08:00:00 GMT\n/api/logs
```

Cuando tenga la cadena de firma, codifíquela usando un algoritmo HMAC-SHA256 en la cadena codificada mediante UTF-8 y, después, codifique el resultado como Base64. Use este formato:

```
Signature=Base64(HMAC-SHA256(UTF8(StringToSign)))
```

Los ejemplos de las secciones siguientes tienen código de ejemplo que le ayudarán a crear un encabezado de autorización.

## <a name="request-body"></a>Cuerpo de la solicitud
El cuerpo del mensaje debe estar en formato JSON. Debe incluir uno o varios registros con los pares de nombre y valor de propiedad en el siguiente formato. El nombre de la propiedad solo puede contener letras, números y guiones bajos (_).

```json
[
    {
        "property 1": "value1",
        "property 2": "value2",
        "property 3": "value3",
        "property 4": "value4"
    }
]
```

Puede procesar por lotes varios registros en una sola solicitud con el formato siguiente. Todos los registros deben ser del mismo tipo de registro.

```json
[
    {
        "property 1": "value1",
        "property 2": "value2",
        "property 3": "value3",
        "property 4": "value4"
    },
    {
        "property 1": "value1",
        "property 2": "value2",
        "property 3": "value3",
        "property 4": "value4"
    }
]
```

## <a name="record-type-and-properties"></a>Propiedades y tipo de registro
Cuando se envían datos a través de HTTP Data Collector API de Azure Monitor, se define un tipo de registro personalizado. Actualmente, no se pueden escribir datos en tipos de registros existentes creados por otros tipos de datos y soluciones. Azure Monitor lee los datos entrantes y después crea las propiedades que coinciden con los tipos de datos de los valores que especifique.

Cada solicitud a Data Collector API debe incluir un encabezado **Log-Type** con el nombre del tipo de registro. El sufijo **_CL** se anexa automáticamente al nombre que especifique para distinguirlo de otros tipos de registros como un registro personalizado. Por ejemplo, si escribe el nombre **MyNewRecordType**, Azure Monitor crea un registro con el tipo de **MyNewRecordType_CL**. Esto ayuda a garantizar que no haya conflictos entre los nombres de tipo creados por el usuario y los incluidos en las soluciones de Microsoft actuales o futuras.

Para identificar el tipo de datos de una propiedad, Azure Monitor agrega un sufijo al nombre de la propiedad. Si una propiedad contiene un valor null, la propiedad no se incluye en ese registro. Esta tabla enumera el tipo de datos de la propiedad y el sufijo correspondiente:

| Tipo de datos de la propiedad | Sufijo |
|:--- |:--- |
| String |_s |
| Boolean |_b |
| Double |_d |
| Fecha/hora |_t |
| GUID (almacenado como una cadena) |_g |

> [!NOTE]
> A los valores de cadena que parecen ser GUID se les asigna el sufijo _g y se les aplica formato de GUID, incluso si el valor entrante no incluye guiones. Por ejemplo, "8145d822-13a7-44ad-859c-36f31a84f6dd" y "8145d82213a744ad859c36f31a84f6dd" se almacenarán como "8145d822-13a7-44ad-859c-36f31a84f6dd". Las únicas diferencias entre esta y otra cadena es la _g en el nombre y la inserción de guiones si no se proporcionan en la entrada. 

El tipo de datos que utiliza Azure Monitor para cada propiedad depende de si ya existe el tipo de registro para el nuevo registro.

* Si el tipo de registro no existe, Azure Monitor crea uno nuevo mediante la inferencia de tipos JSON para determinar el tipo de datos para cada propiedad del nuevo registro.
* Si el tipo de registro existe, Azure Monitor intenta crear un nuevo registro en función de las propiedades existentes. Si el tipo de datos de una propiedad en el nuevo registro no coincide y no se puede convertir al tipo existente, o si el registro incluye una propiedad que no existe, Azure Monitor crea una nueva propiedad que tiene el sufijo pertinente.

Por ejemplo, esta entrada de envío crearía un registro con tres propiedades **number_d**, **boolean_b** y **string_s**:

![Registro de ejemplo 1](media/data-collector-api/record-01.png)

Si después se envía la entrada siguiente, con todos los valores con formato de cadena, las propiedades no cambiarían. Estos valores se pueden convertir a los tipos de datos existentes:

![Registro de ejemplo 2](media/data-collector-api/record-02.png)

Pero, si después se realiza el siguiente envío, Azure Monitor crearía las nuevas propiedades **boolean_d** y **string_d**. Estos valores no se pueden convertir:

![Registro de ejemplo 3](media/data-collector-api/record-03.png)

Si después se envía la entrada siguiente, antes de que se cree el tipo de registro, Azure Monitor crearía un registro con tres propiedades **number_s**, **boolean_s** y **string_s**. En esta entrada, se da un formato de cadena a cada uno de los valores iniciales:

![Registro de ejemplo 4](media/data-collector-api/record-04.png)

## <a name="reserved-properties"></a>Propiedades reservadas
Las propiedades siguientes están reservadas y no deben usarse en un registro de tipo personalizado. Recibirá un error si la carga útil incluye alguno de estos nombres de propiedad.

- tenant

## <a name="data-limits"></a>Límites de datos
Existen algunas restricciones en cuando a los datos enviados a Data Collector API de Azure Monitor.

* Máximo de 30 MB por publicación en Data Collector API de Azure Monitor. Se trata de un límite de tamaño para una sola publicación. Si los datos de una única publicación superan los 30 MB, debe dividir los datos en fragmentos más pequeños y enviarlos al mismo tiempo.
* Límite de 32 kB para los valores de campo. Si el valor del campo es mayor que 32 kB, se truncarán los datos.
* El número máximo recomendado de campos para un tipo determinado es 50. Se trata de un límite práctico desde una perspectiva de la experiencia de búsqueda y la facilidad de uso.  
* Una tabla en un área de trabajo de Log Analytics solo admite hasta 500 columnas (denominada campo en este artículo). 
* El número máximo de caracteres para el nombre de columna es 500.

## <a name="return-codes"></a>Códigos de retorno
El código de estado HTTP 200 significa que se ha recibido la solicitud para su procesamiento. Esto indica que el trabajo se ha completado correctamente.

Esta tabla muestra el conjunto completo de códigos de estado que el servicio puede devolver:

| Código | Estado | Código de error | Descripción |
|:--- |:--- |:--- |:--- |
| 200 |Aceptar | |La solicitud se aceptó correctamente. |
| 400 |Solicitud incorrecta |InactiveCustomer |El área de trabajo se cerró. |
| 400 |Solicitud incorrecta |InvalidApiVersion |El servicio no reconoció la versión de API especificada. |
| 400 |Solicitud incorrecta |InvalidCustomerId |El identificador de área de trabajo especificado no es válido. |
| 400 |Solicitud incorrecta |InvalidDataFormat |No envió un formato JSON no válido. El cuerpo de la respuesta puede contener más información acerca de cómo resolver el error. |
| 400 |Solicitud incorrecta |InvalidLogType |El tipo de registro especificado contenía caracteres especiales o valores numéricos. |
| 400 |Solicitud incorrecta |MissingApiVersion |No se especificó la versión de API. |
| 400 |Solicitud incorrecta |MissingContentType |No se especificó el tipo de contenido. |
| 400 |Solicitud incorrecta |MissingLogType |No se especificó el tipo de registro de valor requerido. |
| 400 |Solicitud incorrecta |UnsupportedContentType |No se estableció el tipo de contenido en **application/json**. |
| 403 |Prohibido |InvalidAuthorization |El servicio no pudo autenticar la solicitud. Compruebe que el identificador del área de trabajo y la clave de conexión sean válidas. |
| 404 |No encontrado | | La dirección URL proporcionada no es correcta, bien o la solicitud es demasiado grande. |
| 429 |Demasiadas solicitudes | | El servicio está experimentando un gran volumen de datos de su cuenta. Vuelva a intentar realizar la solicitud más tarde. |
| 500 |Internal Server Error |UnspecifiedError |El servicio detectó un error interno. Vuelva a intentar realizar la solicitud. |
| 503 |Servicio no disponible |ServiceUnavailable |El servicio actualmente no está disponible para recibir solicitudes. Vuelva a intentar realizar la solicitud. |

## <a name="query-data"></a>Consultar datos
Para consultar los datos enviados por HTTP Data Collector API de Azure Monitor, busque los registros cuyo **Type** sea igual al valor de **LogType** que especificó, con el sufijo **_CL**. Por ejemplo, si utilizó **MyCustomLog**, se devolverán todos los registros con `MyCustomLog_CL`.

## <a name="sample-requests"></a>Solicitudes de ejemplo
En las secciones siguientes, encontrará ejemplos de cómo enviar datos a HTTP Data Collector API de Azure Monitor mediante el uso de diferentes lenguajes de programación.

Para cada ejemplo, siga estos pasos para establecer las variables para el encabezado de autorización:

1. En Azure Portal, seleccione el área de trabajo de Log Analytics.
2. Seleccione **Administración de agentes**.
2. A la derecha de **Id. de área de trabajo**, seleccione el icono de copia y pegue el identificador como valor de la variable **Id. de cliente**.
3. A la derecha de **Clave principal**, seleccione el icono de copia y pegue el identificador como valor de la variable **Clave compartida**.

También puede cambiar las variables del tipo de registro y de los datos JSON.

### <a name="powershell-sample"></a>Ejemplo de PowerShell
```powershell
# Replace with your Workspace ID
$CustomerId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  

# Replace with your Primary Key
$SharedKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Specify the name of the record type that you'll be creating
$LogType = "MyRecordType"

# You can use an optional field to specify the timestamp from the data. If the time field is not specified, Azure Monitor assumes the time is the message ingestion time
$TimeStampField = ""


# Create two records with the same set of properties to create
$json = @"
[{  "StringValue": "MyString1",
    "NumberValue": 42,
    "BooleanValue": true,
    "DateValue": "2019-09-12T20:00:00.625Z",
    "GUIDValue": "9909ED01-A74C-4874-8ABF-D2678E3AE23D"
},
{   "StringValue": "MyString2",
    "NumberValue": 43,
    "BooleanValue": false,
    "DateValue": "2019-09-12T20:00:00.625Z",
    "GUIDValue": "8809ED01-A74C-4874-8ABF-D2678E3AE23D"
}]
"@

# Create the function to create the authorization signature
Function Build-Signature ($customerId, $sharedKey, $date, $contentLength, $method, $contentType, $resource)
{
    $xHeaders = "x-ms-date:" + $date
    $stringToHash = $method + "`n" + $contentLength + "`n" + $contentType + "`n" + $xHeaders + "`n" + $resource

    $bytesToHash = [Text.Encoding]::UTF8.GetBytes($stringToHash)
    $keyBytes = [Convert]::FromBase64String($sharedKey)

    $sha256 = New-Object System.Security.Cryptography.HMACSHA256
    $sha256.Key = $keyBytes
    $calculatedHash = $sha256.ComputeHash($bytesToHash)
    $encodedHash = [Convert]::ToBase64String($calculatedHash)
    $authorization = 'SharedKey {0}:{1}' -f $customerId,$encodedHash
    return $authorization
}


# Create the function to create and post the request
Function Post-LogAnalyticsData($customerId, $sharedKey, $body, $logType)
{
    $method = "POST"
    $contentType = "application/json"
    $resource = "/api/logs"
    $rfc1123date = [DateTime]::UtcNow.ToString("r")
    $contentLength = $body.Length
    $signature = Build-Signature `
        -customerId $customerId `
        -sharedKey $sharedKey `
        -date $rfc1123date `
        -contentLength $contentLength `
        -method $method `
        -contentType $contentType `
        -resource $resource
    $uri = "https://" + $customerId + ".ods.opinsights.azure.com" + $resource + "?api-version=2016-04-01"

    $headers = @{
        "Authorization" = $signature;
        "Log-Type" = $logType;
        "x-ms-date" = $rfc1123date;
        "time-generated-field" = $TimeStampField;
    }

    $response = Invoke-WebRequest -Uri $uri -Method $method -ContentType $contentType -Headers $headers -Body $body -UseBasicParsing
    return $response.StatusCode

}

# Submit the data to the API endpoint
Post-LogAnalyticsData -customerId $customerId -sharedKey $sharedKey -body ([System.Text.Encoding]::UTF8.GetBytes($json)) -logType $logType  
```

### <a name="c-sample"></a>Ejemplo de C#
```csharp
using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Security.Cryptography;
using System.Text;
using System.Threading.Tasks;

namespace OIAPIExample
{
    class ApiExample
    {
        // An example JSON object, with key/value pairs
        static string json = @"[{""DemoField1"":""DemoValue1"",""DemoField2"":""DemoValue2""},{""DemoField3"":""DemoValue3"",""DemoField4"":""DemoValue4""}]";

        // Update customerId to your Log Analytics workspace ID
        static string customerId = "xxxxxxxx-xxx-xxx-xxx-xxxxxxxxxxxx";

        // For sharedKey, use either the primary or the secondary Connected Sources client authentication key   
        static string sharedKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";

        // LogName is name of the event type that is being submitted to Azure Monitor
        static string LogName = "DemoExample";

        // You can use an optional field to specify the timestamp from the data. If the time field is not specified, Azure Monitor assumes the time is the message ingestion time
        static string TimeStampField = "";

        static void Main()
        {
            // Create a hash for the API signature
            var datestring = DateTime.UtcNow.ToString("r");
            var jsonBytes = Encoding.UTF8.GetBytes(json);
            string stringToHash = "POST\n" + jsonBytes.Length + "\napplication/json\n" + "x-ms-date:" + datestring + "\n/api/logs";
            string hashedString = BuildSignature(stringToHash, sharedKey);
            string signature = "SharedKey " + customerId + ":" + hashedString;

            PostData(signature, datestring, json);
        }

        // Build the API signature
        public static string BuildSignature(string message, string secret)
        {
            var encoding = new System.Text.ASCIIEncoding();
            byte[] keyByte = Convert.FromBase64String(secret);
            byte[] messageBytes = encoding.GetBytes(message);
            using (var hmacsha256 = new HMACSHA256(keyByte))
            {
                byte[] hash = hmacsha256.ComputeHash(messageBytes);
                return Convert.ToBase64String(hash);
            }
        }

        // Send a request to the POST API endpoint
        public static void PostData(string signature, string date, string json)
        {
            try
            {
                string url = "https://" + customerId + ".ods.opinsights.azure.com/api/logs?api-version=2016-04-01";

                System.Net.Http.HttpClient client = new System.Net.Http.HttpClient();
                client.DefaultRequestHeaders.Add("Accept", "application/json");
                client.DefaultRequestHeaders.Add("Log-Type", LogName);
                client.DefaultRequestHeaders.Add("Authorization", signature);
                client.DefaultRequestHeaders.Add("x-ms-date", date);
                client.DefaultRequestHeaders.Add("time-generated-field", TimeStampField);

                System.Net.Http.HttpContent httpContent = new StringContent(json, Encoding.UTF8);
                httpContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                Task<System.Net.Http.HttpResponseMessage> response = client.PostAsync(new Uri(url), httpContent);

                System.Net.Http.HttpContent responseContent = response.Result.Content;
                string result = responseContent.ReadAsStringAsync().Result;
                Console.WriteLine("Return Result: " + result);
            }
            catch (Exception excep)
            {
                Console.WriteLine("API Post Exception: " + excep.Message);
            }
        }
    }
}

```

### <a name="python-2-sample"></a>Ejemplo de Python 2
```python
import json
import requests
import datetime
import hashlib
import hmac
import base64

# Update the customer ID to your Log Analytics workspace ID
customer_id = 'xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

# For the shared key, use either the primary or the secondary Connected Sources client authentication key   
shared_key = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# The log type is the name of the event that is being submitted
log_type = 'WebMonitorTest'

# An example JSON web monitor object
json_data = [{
   "slot_ID": 12345,
    "ID": "5cdad72f-c848-4df0-8aaa-ffe033e75d57",
    "availability_Value": 100,
    "performance_Value": 6.954,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "true"
},
{   
    "slot_ID": 67890,
    "ID": "b6bee458-fb65-492e-996d-61c4d7fbb942",
    "availability_Value": 100,
    "performance_Value": 3.379,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "false"
}]
body = json.dumps(json_data)

#####################
######Functions######  
#####################

# Build the API signature
def build_signature(customer_id, shared_key, date, content_length, method, content_type, resource):
    x_headers = 'x-ms-date:' + date
    string_to_hash = method + "\n" + str(content_length) + "\n" + content_type + "\n" + x_headers + "\n" + resource
    bytes_to_hash = bytes(string_to_hash).encode('utf-8')  
    decoded_key = base64.b64decode(shared_key)
    encoded_hash = base64.b64encode(hmac.new(decoded_key, bytes_to_hash, digestmod=hashlib.sha256).digest())
    authorization = "SharedKey {}:{}".format(customer_id,encoded_hash)
    return authorization

# Build and send a request to the POST API
def post_data(customer_id, shared_key, body, log_type):
    method = 'POST'
    content_type = 'application/json'
    resource = '/api/logs'
    rfc1123date = datetime.datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
    content_length = len(body)
    signature = build_signature(customer_id, shared_key, rfc1123date, content_length, method, content_type, resource)
    uri = 'https://' + customer_id + '.ods.opinsights.azure.com' + resource + '?api-version=2016-04-01'

    headers = {
        'content-type': content_type,
        'Authorization': signature,
        'Log-Type': log_type,
        'x-ms-date': rfc1123date
    }

    response = requests.post(uri,data=body, headers=headers)
    if (response.status_code >= 200 and response.status_code <= 299):
        print 'Accepted'
    else:
        print "Response code: {}".format(response.status_code)

post_data(customer_id, shared_key, body, log_type)
```

### <a name="python-3-sample"></a>Ejemplo de Python 3
```python
import json
import requests
import datetime
import hashlib
import hmac
import base64

# Update the customer ID to your Log Analytics workspace ID
customer_id = 'xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

# For the shared key, use either the primary or the secondary Connected Sources client authentication key   
shared_key = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# The log type is the name of the event that is being submitted
log_type = 'WebMonitorTest'

# An example JSON web monitor object
json_data = [{
   "slot_ID": 12345,
    "ID": "5cdad72f-c848-4df0-8aaa-ffe033e75d57",
    "availability_Value": 100,
    "performance_Value": 6.954,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "true"
},
{   
    "slot_ID": 67890,
    "ID": "b6bee458-fb65-492e-996d-61c4d7fbb942",
    "availability_Value": 100,
    "performance_Value": 3.379,
    "measurement_Name": "last_one_hour",
    "duration": 3600,
    "warning_Threshold": 0,
    "critical_Threshold": 0,
    "IsActive": "false"
}]
body = json.dumps(json_data)

#####################
######Functions######  
#####################

# Build the API signature
def build_signature(customer_id, shared_key, date, content_length, method, content_type, resource):
    x_headers = 'x-ms-date:' + date
    string_to_hash = method + "\n" + str(content_length) + "\n" + content_type + "\n" + x_headers + "\n" + resource
    bytes_to_hash = bytes(string_to_hash, encoding="utf-8")  
    decoded_key = base64.b64decode(shared_key)
    encoded_hash = base64.b64encode(hmac.new(decoded_key, bytes_to_hash, digestmod=hashlib.sha256).digest()).decode()
    authorization = "SharedKey {}:{}".format(customer_id,encoded_hash)
    return authorization

# Build and send a request to the POST API
def post_data(customer_id, shared_key, body, log_type):
    method = 'POST'
    content_type = 'application/json'
    resource = '/api/logs'
    rfc1123date = datetime.datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
    content_length = len(body)
    signature = build_signature(customer_id, shared_key, rfc1123date, content_length, method, content_type, resource)
    uri = 'https://' + customer_id + '.ods.opinsights.azure.com' + resource + '?api-version=2016-04-01'

    headers = {
        'content-type': content_type,
        'Authorization': signature,
        'Log-Type': log_type,
        'x-ms-date': rfc1123date
    }

    response = requests.post(uri,data=body, headers=headers)
    if (response.status_code >= 200 and response.status_code <= 299):
        print('Accepted')
    else:
        print("Response code: {}".format(response.status_code))

post_data(customer_id, shared_key, body, log_type)
```


## <a name="alternatives-and-considerations"></a>Alternativas y consideraciones
Aunque la API del recopilador de datos debe cubrir la mayor parte de sus necesidades para recopilar datos de formato libre en los registros de Azure, hay casos en los que podría ser necesaria una alternativa para solucionar algunas de las limitaciones de la API. Las siguientes son todas las opciones, junto con las consideraciones principales:

| Alternativa | Descripción | Idónea para |
|---|---|---|
| [Eventos personalizados](../app/api-custom-events-metrics.md?toc=%2Fazure%2Fazure-monitor%2Ftoc.json#properties): Ingesta basada en SDK nativa de Application Insights | Application Insights, instrumentado normalmente a través de un SDK dentro de la aplicación, ofrece la capacidad de enviar datos personalizados a través de los eventos personalizados. | <ul><li> Los datos se generan dentro de la aplicación, pero el SDK no los recibe a través de uno de los tipos de datos predeterminados (solicitudes, dependencias, excepciones, etc.).</li><li> Datos que está correlacionados con mayor frecuencia con otros datos de aplicación en Application Insights </li></ul> |
| API del recopilador de datos en registros de Azure Monitor | La API del recopilador de datos en registros de Azure Monitor es una forma completamente libre para la ingesta de datos. Pueden enviársele todos los datos cuyo formato sea un objeto JSON. Una vez enviados, se procesarán y estarán disponibles en los registros, para que se correlacionen con otros datos en los registros u otros datos de Application Insights. <br/><br/> Es bastante fácil cargar los datos como archivos en un blob de Azure Blob, desde donde se procesarán y cargarán estos archivos en Log Analytics. Consulte [este](./create-pipeline-datacollector-api.md) artículo para ver un ejemplo de implementación de una canalización de este tipo. | <ul><li> Datos que no necesariamente se generaron dentro de una aplicación instrumentada en Application Insights.</li><li> Algunos ejemplos son tablas de hechos y de búsqueda, datos de referencia, estadísticas agregadas previamente, etc. </li><li> Diseñado para datos a los que se harán referencia cruzadas con otros datos de Azure Monitor (Application Insights, otros tipos de datos de registros, Security Center, Azure Monitor para contenedores y máquinas virtuales, etc.). </li></ul> |
| [Azure Data Explorer](/azure/data-explorer/ingest-data-overview) | Azure Data Explorer (ADX) es la plataforma de datos que se utiliza en Application Insights Analytics y los registros de Azure Monitor. Ahora con disponibilidad general ("GA"), cuando usa la plataforma de datos sin procesar obtiene completa flexibilidad (que requiere la sobrecarga de administración) en el clúster (RBAC, tasa de retención, esquema, etc). Azure Data Explorer proporciona muchas [opciones de ingestión](/azure/data-explorer/ingest-data-overview#ingestion-methods), incluidos los archivos [JSON, CSV y TSV](/azure/kusto/management/mappings?branch=master). | <ul><li> Datos que no se correlacionan con ningún otro dato en Application Insights o los registros. </li><li> Datos que requieren ingesta o procesamiento avanzados que no están disponibles actualmente en los registros de Azure Monitor. </li></ul> |


## <a name="next-steps"></a>Pasos siguientes
- Use [Log Search API](../log-query/log-query-overview.md) para recuperar datos desde el área de trabajo de Log Analytics.

- Más información sobre cómo [crear una canalización de datos con Data Collector API](create-pipeline-datacollector-api.md) mediante el flujo de trabajo Logic Apps a Azure Monitor.
