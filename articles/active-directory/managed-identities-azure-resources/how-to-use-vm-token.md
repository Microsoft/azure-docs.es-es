---
title: 'Uso de usar identidades administradas en una máquina virtual para adquirir un token de acceso: Azure AD'
description: Paso a paso las instrucciones y ejemplos de uso de administran identidades para los recursos de Azure en una máquina virtual para adquirir un token de acceso de OAuth.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4683a77b9467775fbe368e2017416e0fbff9718c
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266296"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-to-acquire-an-access-token"></a>Cómo usar identidades administradas de recursos de Azure en una máquina virtual de Azure para adquirir un token de acceso 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo se proporcionan diversos ejemplos de códigos y scripts para obtener tokens, así como instrucciones sobre temas importantes como el control de errores HTTP y la expiración de tokens. 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Si tiene previsto usar los ejemplos de Azure PowerShell de este artículo, no se olvide de instalar la última versión de [Azure PowerShell](/powershell/azure/install-az-ps).


> [!IMPORTANT]
> - En todo el código y scripts de ejemplo de este artículo, se da por supuesto que el cliente se ejecuta en una máquina virtual con identidades administradas de recursos de Azure. Use la característica "Conectar" de la máquina virtual en Azure Portal para conectarse a la máquina virtual de forma remota. Para más información sobre la habilitación de identidades administradas de recursos de Azure en una VM, consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal) o uno de los artículos derivados (mediante PowerShell, la CLI, una plantilla o un SDK de Azure). 

> [!IMPORTANT]
> - El límite de seguridad de las identidades administradas de recursos de Azure es el recurso en el que se usa. Todo el código o scripts que se ejecutan en una máquina virtual pueden solicitar y recuperar tokens para cualquier identidad administrada disponible. 

## <a name="overview"></a>Información general

Una aplicación cliente puede solicitar un [token de acceso de solo aplicación](../develop/developer-glossary.md#access-token) de identidades administradas de recursos de Azure para acceder a un recurso determinado. El token [se basa en la entidad de servicio de identidades administradas de recursos de Azure](overview.md#managed-identity-types). Por lo tanto, no es necesario que el cliente se registre automáticamente para obtener un token de acceso en su propia entidad de servicio. El token es adecuado para utilizarse como un token de portador en [llamadas de servicio a servicio que requieren credenciales de cliente](../develop/v2-oauth2-client-creds-grant-flow.md).

| Vínculo | Descripción |
| -------------- | -------------------- |
| [Obtención de un token con HTTP](#get-a-token-using-http) | Detalles del protocolo del punto de conexión del token de las identidades administradas de recursos de Azure |
| [Obtención de un token con la biblioteca Microsoft.Azure.Services.AppAuthentication para .NET](#get-a-token-using-the-microsoftazureservicesappauthentication-library-for-net) | Ejemplo del uso de la biblioteca Microsoft.Azure.Services.AppAuthentication desde un cliente .NET
| [Obtención de un token con C#](#get-a-token-using-c) | Ejemplo del uso del punto de conexión REST de identidades administradas de recursos de Azure desde un cliente de C# |
| [Obtención de un token con Java](#get-a-token-using-java) | Ejemplo del uso del punto de conexión REST de identidades administradas de recursos de Azure desde un cliente de Java |
| [Obtención de un token con Go](#get-a-token-using-go) | Ejemplo del uso del punto de conexión REST de identidades administradas de recursos de Azure desde un cliente de Go |
| [Obtención de un token con Azure PowerShell](#get-a-token-using-azure-powershell) | Ejemplo del uso del punto de conexión REST de identidades administradas de recursos de Azure desde un cliente de PowerShell |
| [Obtención de un token con CURL](#get-a-token-using-curl) | Ejemplo del uso del punto de conexión REST de identidades administradas de recursos de Azure desde un cliente de Bash o CURL |
| Control del almacenamiento en caché de tokens | Instrucciones para controlar los tokens de acceso expirados |
| [Control de errores](#error-handling) | Instrucciones para controlar los errores HTTP devueltos desde el punto de conexión del token de las identidades administradas de recursos de Azure |
| [Identificadores de recurso de servicios de Azure](#resource-ids-for-azure-services) | Dónde obtener identificadores de recurso de los servicios de Azure admitidos |

## <a name="get-a-token-using-http"></a>Obtención de un token con HTTP 

La interfaz básica para obtener un token de acceso se basa en REST, de modo que podrá acceder a cualquier aplicación cliente que se ejecute en la máquina virtual que es capaz de realizar llamadas REST de HTTP. Algo parecido ocurre con el modelo de programación de Azure AD, solo que el cliente usa un punto de conexión en la máquina virtual (y no un punto de conexión de Azure AD).

Solicitud de ejemplo con el punto de conexión de Azure Instance Metadata Service (IMDS) *(opción recomendada)* :

```
GET 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' HTTP/1.1 Metadata: true
```

| Elemento | Descripción |
| ------- | ----------- |
| `GET` | El verbo HTTP, indicando que se van a recuperar datos desde el punto de conexión. En este caso, el token de acceso de OAuth. | 
| `http://169.254.169.254/metadata/identity/oauth2/token` | Punto de conexión del token de las identidades administradas de recursos de Azure para Instance Metadata Service. |
| `api-version`  | Un parámetro de cadena de consulta, que indica la versión de API del punto de conexión de IMDS. Use una versión de API `2018-02-01` o superior. |
| `resource` | Un parámetro de cadena de consulta, que indica el URI del identificador de aplicación del recurso de destino. También aparece en la notificación `aud` (audiencia) del token emitido. En este ejemplo, se solicita un token para acceder a Azure Resource Manager, que tiene un URI de identificador de aplicación de `https://management.azure.com/`. |
| `Metadata` | Un campo de encabezado de la solicitud HTTP, requerido por las identidades administradas de recursos de Azure como mitigación frente a ataques de falsificación de solicitud de lado del servidor (SSRF). Este valor debe establecerse en "true", todo en minúsculas. |
| `object_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de object_id de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario.|
| `client_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de client_id de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario.|
| `mi_res_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de mi_res_id (Azure Resource AD) de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario. |

Solicitud de ejemplo con el punto de conexión de extensión de máquina virtual de identidades administradas de recursos de Azure *(plan de desuso previsto para enero de 2019)* :

```http
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| Elemento | Descripción |
| ------- | ----------- |
| `GET` | El verbo HTTP, indicando que se van a recuperar datos desde el punto de conexión. En este caso, el token de acceso de OAuth. | 
| `http://localhost:50342/oauth2/token` | Punto de conexión de identidades administradas de recursos de Azure, donde 50342 es el puerto predeterminado y configurable. |
| `resource` | Un parámetro de cadena de consulta, que indica el URI del identificador de aplicación del recurso de destino. También aparece en la notificación `aud` (audiencia) del token emitido. En este ejemplo, se solicita un token para acceder a Azure Resource Manager, que tiene un URI de identificador de aplicación de `https://management.azure.com/`. |
| `Metadata` | Un campo de encabezado de la solicitud HTTP, requerido por las identidades administradas de recursos de Azure como mitigación frente a ataques de falsificación de solicitud de lado del servidor (SSRF). Este valor debe establecerse en "true", todo en minúsculas.|
| `object_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de object_id de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario.|
| `client_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de client_id de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario.|

Respuesta de ejemplo:

```json
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| Elemento | Descripción |
| ------- | ----------- |
| `access_token` | El token de acceso solicitado. Al llamar a una API de REST protegida, el token se inserta en el campo `Authorization` del encabezado de la solicitud como un token de "portador", lo que permite a la API autenticar el llamador. | 
| `refresh_token` | Las identidades administradas de recursos de Azure no lo usan. |
| `expires_in` | El número de segundos que el token de acceso sigue siendo válido, antes de expirar, desde la hora de emisión. La hora de emisión puede encontrarse en la notificación `iat` del token. |
| `expires_on` | La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde "1970-01-01T0:0:0Z UTC" (se corresponde con la notificación `exp` del token). |
| `not_before` | El intervalo de tiempo cuando el token de acceso tiene efecto y se puede aceptar. La fecha se representa como el número de segundos desde "1970-01-01T0:0:0Z UTC" (se corresponde con la notificación `nbf` del token). |
| `resource` | El recurso para el que se solicitó el token de acceso, que coincide con el parámetro de la cadena de consulta `resource` de la solicitud. |
| `token_type` | El tipo de token, que es un token de acceso al "Portador", lo que significa que el recurso puede permitir el acceso al portador del token. |

## <a name="get-a-token-using-the-microsoftazureservicesappauthentication-library-for-net"></a>Obtención de un token con la biblioteca Microsoft.Azure.Services.AppAuthentication para .NET

En el caso de las aplicaciones y las funciones de .NET, la manera más sencilla de trabajar con identidades administradas de recursos de Azure es a través del paquete Microsoft.Azure.Services.AppAuthentication. Esta biblioteca también le permite probar el código localmente en la máquina de desarrollo, con su cuenta de usuario de Visual Studio, la [CLI de Azure](/cli/azure?view=azure-cli-latest) o la autenticación integrada de Active Directory. Para obtener más información sobre las opciones de desarrollo local con esta biblioteca, consulte la [referencia de Microsoft.Azure.Services.AppAuthentication](../../key-vault/general/service-to-service-authentication.md). En esta sección se muestra cómo empezar a usar la biblioteca en su código.

1. Agregue referencias a los paquetes de NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) y [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) para la aplicación.

2.  Agregue el siguiente código a su aplicación:

    ```csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;
    // ...
    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/");
    // OR
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
    ```
    
Para obtener más información sobre Microsoft.Azure.Services.AppAuthentication y las operaciones que expone, consulte la [referencia de Microsoft.Azure.Services.AppAuthentication](../../key-vault/general/service-to-service-authentication.md) y el [ejemplo de .NET sobre App Service y KeyVault con identidades administradas de recursos de Azure](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

## <a name="get-a-token-using-c"></a>Obtención de un token con C#

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Web.Script.Serialization; 

// Build request to acquire managed identities for Azure resources token
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
request.Headers["Metadata"] = "true";
request.Method = "GET";

try
{
    // Call /token endpoint
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader, and extract access token
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    string accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

```

## <a name="get-a-token-using-java"></a>Obtención de un token con Java

Use esta [biblioteca JSON](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core/2.9.4) para recuperar un token con Java.

```Java
import java.io.*;
import java.net.*;
import com.fasterxml.jackson.core.*;
 
class GetMSIToken {
    public static void main(String[] args) throws Exception {
 
        URL msiEndpoint = new URL("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
        HttpURLConnection con = (HttpURLConnection) msiEndpoint.openConnection();
        con.setRequestMethod("GET");
        con.setRequestProperty("Metadata", "true");
 
        if (con.getResponseCode()!=200) {
            throw new Exception("Error calling managed identity token endpoint.");
        }
 
        InputStream responseStream = con.getInputStream();
 
        JsonFactory factory = new JsonFactory();
        JsonParser parser = factory.createParser(responseStream);
 
        while(!parser.isClosed()){
            JsonToken jsonToken = parser.nextToken();
 
            if(JsonToken.FIELD_NAME.equals(jsonToken)){
                String fieldName = parser.getCurrentName();
                jsonToken = parser.nextToken();
 
                if("access_token".equals(fieldName)){
                    String accesstoken = parser.getValueAsString();
                    System.out.println("Access Token: " + accesstoken.substring(0,5)+ "..." + accesstoken.substring(accesstoken.length()-5));
                    return;
                }
            }
        }
    }
}
```

## <a name="get-a-token-using-go"></a>Obtención de un token con Go

```
package main

import (
  "fmt"
  "io/ioutil"
  "net/http"
  "net/url"
  "encoding/json"
)

type responseJson struct {
  AccessToken string `json:"access_token"`
  RefreshToken string `json:"refresh_token"`
  ExpiresIn string `json:"expires_in"`
  ExpiresOn string `json:"expires_on"`
  NotBefore string `json:"not_before"`
  Resource string `json:"resource"`
  TokenType string `json:"token_type"`
}

func main() {
    
    // Create HTTP request for a managed services for Azure resources token to access Azure Resource Manager
    var msi_endpoint *url.URL
    msi_endpoint, err := url.Parse("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01")
    if err != nil {
      fmt.Println("Error creating URL: ", err)
      return 
    }
    msi_parameters := url.Values{}
    msi_parameters.Add("resource", "https://management.azure.com/")
    msi_endpoint.RawQuery = msi_parameters.Encode()
    req, err := http.NewRequest("GET", msi_endpoint.String(), nil)
    if err != nil {
      fmt.Println("Error creating HTTP request: ", err)
      return 
    }
    req.Header.Add("Metadata", "true")

    // Call managed services for Azure resources token endpoint
    client := &http.Client{}
    resp, err := client.Do(req) 
    if err != nil{
      fmt.Println("Error calling token endpoint: ", err)
      return
    }

    // Pull out response body
    responseBytes,err := ioutil.ReadAll(resp.Body)
    defer resp.Body.Close()
    if err != nil {
      fmt.Println("Error reading response body : ", err)
      return
    }

    // Unmarshall response body into struct
    var r responseJson
    err = json.Unmarshal(responseBytes, &r)
    if err != nil {
      fmt.Println("Error unmarshalling the response:", err)
      return
    }

    // Print HTTP response and marshalled response body elements to console
    fmt.Println("Response status:", resp.Status)
    fmt.Println("access_token: ", r.AccessToken)
    fmt.Println("refresh_token: ", r.RefreshToken)
    fmt.Println("expires_in: ", r.ExpiresIn)
    fmt.Println("expires_on: ", r.ExpiresOn)
    fmt.Println("not_before: ", r.NotBefore)
    fmt.Println("resource: ", r.Resource)
    fmt.Println("token_type: ", r.TokenType)
}
```

## <a name="get-a-token-using-azure-powershell"></a>Obtención de un token con Azure PowerShell

En el ejemplo siguiente se muestra cómo utilizar el punto de conexión REST de identidades administradas de recursos de Azure desde un cliente de PowerShell para las siguientes tareas:

1. Obtener un token de acceso.
2. Usar el token de acceso para llamar a una API de REST de Azure Resource Manager y consultar información sobre la máquina virtual. No olvide sustituir el identificador de suscripción, el nombre del grupo de recursos y el nombre de máquina virtual en `<SUBSCRIPTION-ID>`, `<RESOURCE-GROUP>` y `<VM-NAME>` respectivamente.

```azurepowershell
Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Headers @{Metadata="true"}
```

Ejemplo de cómo analizar el token de acceso de la respuesta:
```azurepowershell
# Get an access token for managed identities for Azure resources
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' `
                              -Headers @{Metadata="true"}
$content =$response.Content | ConvertFrom-Json
$access_token = $content.access_token
echo "The managed identities for Azure resources access token is $access_token"

# Use the access token to get resource information for the VM
$vmInfoRest = (Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Compute/virtualMachines/<VM-NAME>?api-version=2017-12-01' -Method GET -ContentType "application/json" -Headers @{ Authorization ="Bearer $access_token"}).content
echo "JSON returned from call to get VM info:"
echo $vmInfoRest

```

## <a name="get-a-token-using-curl"></a>Obtención de un token con CURL

```bash
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true -s
```


Ejemplo de cómo analizar el token de acceso de la respuesta:

```bash
response=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true -s)
access_token=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["access_token"])')
echo The managed identities for Azure resources access token is $access_token
```

## <a name="token-caching"></a>Almacenamiento en caché de tokens

Aunque el subsistema de identidades administradas de recursos de Azure que se usa (extensión de VM de IMDS o identidades administradas de recursos de Azure) almacena tokens en caché, también se recomienda implementar el almacenamiento en caché de tokens en el código. Como resultado, debe prepararse para escenarios en que el recurso indica que el token ha caducado. 

Las llamadas por cable a Azure AD solo se generan cuando:
- el error de caché se produce debido a que no hay ningún token en la caché del subsistema de identidades administradas de recursos de Azure
- el token almacenado en caché ha expirado

## <a name="error-handling"></a>Control de errores

El punto de conexión de identidades administradas de recursos de Azure señala los errores a través del campo de código de estado del encabezado del mensaje de respuesta HTTP; por ejemplo, errores 4xx o 5xx:

| Código de estado | Motivo del error | Realización del control |
| ----------- | ------------ | ------------- |
| 404 No encontrado. | El punto de conexión de IMDS se está actualizando. | Vuelva a intentarlo con retroceso exponencial. Consulte las instrucciones siguientes. |
| 429 Demasiadas solicitudes. |  Se ha alcanzado la limitación de IMDS. | Vuelva a intentarlo con retroceso exponencial. Consulte las instrucciones siguientes. |
| Error 4xx en la solicitud. | Uno o varios de los parámetros de solicitud eran incorrectos. | No vuelva a intentarlo.  Consulte los detalles del error para obtener más información.  Los errores 4xx son de tiempo de diseño.|
| Error transitorio 5xx del servicio. | El subsistema de identidades administradas de recursos de Azure o Azure Active Directory devolvió un error transitorio. | Es seguro volver a intentarlo después de esperar al menos 1 segundo.  Si vuelve a intentarlo demasiado pronto o demasiadas veces, IMDS y Azure AD pueden devolver un error de límite de frecuencia (429).|
| timeout | El punto de conexión de IMDS se está actualizando. | Vuelva a intentarlo con retroceso exponencial. Consulte las instrucciones siguientes. |

Si se produce un error, el cuerpo de respuesta HTTP correspondiente contiene los detalles del error en formato JSON:

| Elemento | Descripción |
| ------- | ----------- |
| error   | Identificador del error. |
| error_description | Descripción detallada del error. **Las descripciones de error pueden cambiar en cualquier momento. No escriba código que cree ramas en función de los valores de la descripción del error.**|

### <a name="http-response-reference"></a>Referencia de la respuesta HTTP

En esta sección se documentan las posibles respuestas de error. Un estado de "200 OK" es una respuesta correcta y el token de acceso se encuentra en el cuerpo de respuesta JSON; en el elemento access_token.

| status code | Error | Descripción del error | Solución |
| ----------- | ----- | ----------------- | -------- |
| 400 - Solicitud incorrecta | invalid_resource | AADSTS50001: la aplicación denominada *\<URI\>* no se encontró en el inquilino llamado  *\<TENANT-ID\>* . Esto puede pasar si el administrador del inquilino no es el que ha instalado el administrador del inquilino o no ha recibido el consentimiento de ningún usuario del inquilino. Es posible que haya enviado la solicitud de autenticación al inquilino incorrecto. | (Solo Linux). |
| 400 - Solicitud incorrecta | bad_request_102 | Encabezado de metadatos necesario no especificado. | El campo `Metadata` del encabezado de la solicitud no está en la solicitud o tiene un formato incorrecto. El valor debe especificarse como `true`, todo en minúsculas. Consulte "Solicitud de ejemplo" en la sección REST anterior para obtener un ejemplo.|
| 401 No autorizado | unknown_source | Origen desconocido *\<URI\>* | Compruebe que el identificador URI de la solicitud HTTP GET tiene el formato correcto. La parte `scheme:host/resource-path` debe especificarse como `http://localhost:50342/oauth2/token`. Consulte "Solicitud de ejemplo" en la sección REST anterior para obtener un ejemplo.|
|           | invalid_request | En la solicitud falta un parámetro necesario, incluye un valor de parámetro no válido o un parámetro repetido, o bien no tiene el formato correcto. |  |
|           | unauthorized_client | El cliente no está autorizado a solicitar un token de acceso con este método. | Esto se debe a que una solicitud no usó el bucle invertido local para llamar a la extensión, o bien que una máquina virtual no tiene identidades administradas de recursos de Azure configuradas correctamente. Consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal) si necesita ayuda con la configuración de la VM. |
|           | access_denied | El propietario del recurso o el servidor de consentimiento rechazaron la solicitud. |  |
|           | unsupported_response_type | El servidor de consentimiento no admite la obtención de un token de acceso con este método. |  |
|           | invalid_scope | El ámbito solicitado no es válido, es desconocido o tiene un formato incorrecto. |  |
| 500 Error interno del servidor | unknown | No se pudo recuperar el token de Active Directory. Para más información, vea los registros en *\<file path\>* . | Compruebe que las identidades administradas de recursos de Azure se han habilitado en la máquina virtual. Consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal) si necesita ayuda con la configuración de la VM.<br><br>Compruebe que el URI de la solicitud HTTP GET tiene el formato correcto, especialmente el del recurso especificado en la cadena de consulta. Consulte "Solicitud de ejemplo" en la sección REST anterior para obtener un ejemplo o [Servicios de Azure que admiten la autenticación de Azure AD](./services-support-managed-identities.md) para obtener una lista de servicios y sus respectivos identificadores de recurso.

## <a name="retry-guidance"></a>Instrucciones de reintento 

Se recomienda volver a intentarlo si recibe un código de error 404, 429 o 5xx (consulte [Control de errores](#error-handling) más arriba).

Se aplican límites al número de llamadas realizadas al punto de conexión de IMDS. Cuando se supera el umbral de limitación, el punto de conexión de IMDS limita las solicitudes sucesivas mientras la limitación está en vigor. Durante este período, el punto de conexión de IMDS devolverá el código de estado HTTP 429 ("Demasiadas solicitudes") y se producirá un error en las solicitudes. 

Para volver a intentarlo, se recomienda la estrategia siguiente: 

| **Estrategia de reintento** | **Configuración** | **Valores** | **Funcionamiento** |
| --- | --- | --- | --- |
|ExponentialBackoff |Número de reintentos<br />Interrupción mínima<br />Interrupción máxima<br />Interrupción delta<br />Primer reintento rápido |5<br />0 segundos<br />60 segundos<br />2 segundos<br />false |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de ~2 segundos<br />Intento 3 - retraso de ~6 segundos<br />Intento 4 - retraso de ~14 segundos<br />Intento 5 - retraso de ~30 segundos |

## <a name="resource-ids-for-azure-services"></a>Identificadores de recurso para los servicios de Azure

Consulte [Servicios de Azure que admiten la autenticación de Azure AD](./services-support-managed-identities.md) para ver una lista de recursos que admite Azure AD y que se han probado con identidades administradas de recursos de Azure y sus respectivos identificadores de recursos.


## <a name="next-steps"></a>Pasos siguientes

- Para habilitar las identidades administradas de recursos de Azure en una máquina virtual de Azure, consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal).