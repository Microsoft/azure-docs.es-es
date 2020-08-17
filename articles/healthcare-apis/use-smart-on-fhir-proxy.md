---
title: Proxy de Azure Active Directory SMART on FHIR
description: En este tutorial se describe cómo usar un proxy para habilitar las aplicaciones SMART on FHIR con Azure API for FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 04/02/2019
ms.openlocfilehash: 2e13a9fc32964781dda07e5534e5cab79868ddf0
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846982"
---
# <a name="tutorial-azure-active-directory-smart-on-fhir-proxy"></a>Tutorial: Proxy de Azure Active Directory SMART on FHIR

[SMART on FHIR](https://docs.smarthealthit.org/) es un conjunto de especificaciones abiertas para integrar aplicaciones de socio con servidores FHIR y sistemas de historiales médicos electrónicos que tienen interfaces FHIR. Uno de los objetivos principales de las especificaciones es describir cómo una aplicación debe detectar los puntos de conexión de autenticación para un servidor FHIR e iniciar una secuencia de autenticación. 

La autenticación se basa en OAuth2. No obstante, dado que SMART on FHIR usa convenciones de nomenclatura de parámetros que no son compatibles de forma inmediata con Azure Active Directory (Azure AD), Azure API for FHIR tiene un proxy integrado de Azure AD SMART on FHIR que habilita un subconjunto de las secuencias de inicio de SMART on FHIR. En concreto, el proxy habilita la [secuencia de inicio de EHR](https://hl7.org/fhir/smart-app-launch/#ehr-launch-sequence).

En este tutorial se describe cómo usar el proxy para habilitar las aplicaciones SMART on FHIR con Azure API for FHIR.

## <a name="prerequisites"></a>Requisitos previos

- Una instancia de Azure API for FHIR
- [.NET Core 2.2](https://dotnet.microsoft.com/download/dotnet-core/2.2)

## <a name="configure-azure-ad-registrations"></a>Configuración de registros de Azure AD

SMART on FHIR requiere que `Audience` tenga un URI de identificador igual al URI del servicio FHIR. La configuración estándar de Azure API for FHIR usa un valor `Audience` de `https://azurehealthcareapis.com`. De todas formas, también puede establecer un valor que coincida con la dirección URL específica de su servicio de FHIR (por ejemplo `https://MYFHIRAPI.azurehealthcareapis.com`). Esto es necesario cuando se trabaja con el proxy de SMART on FHIR.

También necesitará un registro de una aplicación cliente. La mayoría de las aplicaciones SMART on FHIR son aplicaciones JavaScript de una sola página. Por lo tanto, debe seguir las instrucciones para configurar una [aplicación cliente pública en Azure AD](register-public-azure-ad-client-app.md).

Una vez completados estos pasos, debe tener:

- Un servidor FHIR con audiencia rge establecida en `https://MYFHIRAPI.azurehealthcareapis.com`, donde `MYFHIRAPI` es el nombre de la instancia de Azure API for FHIR.
- Un registro de una aplicación cliente público. Anote el identificador de esta aplicación cliente.

## <a name="enable-the-smart-on-fhir-proxy"></a>Habilitación del proxy de SMART on FHIR

Habilite el proxy de SMART on FHIR proxy en la configuración de **Autenticación** para la instancia de Azure API for FHIR; para ello, seleccione la casilla **SMART on FHIR proxy** (Proxy de SMART on FHIR):

![Selecciones para habilitar el proxy de SMART on FHIR](media/tutorial-smart-on-fhir/enable-smart-on-fhir-proxy.png)

## <a name="enable-cors"></a>Habilitación de CORS

Dado que la mayoría de las aplicaciones SMART on FHIR son aplicaciones de JavaScript de una sola página, tiene que [habilitar el uso compartido de recursos entre orígenes (CORS)](configure-cross-origin-resource-sharing.md) para Azure API for FHIR:

![Selecciones para habilitar CORS](media/tutorial-smart-on-fhir/enable-cors.png)

## <a name="configure-the-reply-url"></a>Configuración de la dirección URL de respuesta

El proxy de SMART on FHIR actúa como intermediario entre la aplicación SMART on FHIR y Azure AD. La respuesta de autenticación (el código de autenticación) tiene que ir al proxy de SMART on FHIR en lugar de a la propia aplicación. Después, el proxy reenvía la respuesta a la aplicación. 

Debido a esta retransmisión en dos pasos del código de autenticación, tiene que establecer la dirección URL de respuesta (devolución de llamada) para su aplicación cliente de Azure AD en una dirección URL que es una combinación de la dirección URL de respuesta para el proxy de SMART on FHIR y la dirección URL de respuesta de la aplicación SMART on FHIR. La dirección URL de respuesta combinada tiene este formato:

```http
https://MYFHIRAPI.azurehealthcareapis.com/AadSmartOnFhirProxy/callback/aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA
```

En esa respuesta, `aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA` es una versión codificada en Base64 y segura para direcciones URL de la dirección URL de respuesta para la aplicación SMART on FHIR. En el iniciador de aplicaciones SMART on FHIR, cuando la aplicación se ejecuta localmente, la dirección URL de respuesta es `https://localhost:5001/sampleapp/index.html`. 

Puede generar la dirección URL de respuesta combinada mediante un script como este:

```PowerShell
$replyUrl = "https://localhost:5001/sampleapp/index.html"
$fhirServerUrl = "https://MYFHIRAPI.azurewebsites.net"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($ReplyUrl)
$encodedText = [Convert]::ToBase64String($bytes)
$encodedText = $encodedText.TrimEnd('=');
$encodedText = $encodedText.Replace('/','_');
$encodedText = $encodedText.Replace('+','-');

$newReplyUrl = $FhirServerUrl.TrimEnd('/') + "/AadSmartOnFhirProxy/callback/" + $encodedText
```

Agregue la dirección URL de respuesta a la aplicación cliente pública que creó anteriormente para Azure AD:

![URL de respuesta configurada para el cliente público](media/tutorial-smart-on-fhir/configure-reply-url.png)

## <a name="get-a-test-patient"></a>Obtención de un paciente de prueba

Para probar Azure API for FHIR y el proxy de SMART on FHIR, tiene que tener al menos un paciente en la base de datos. Si aún no ha interactuado con la API y no tiene datos en la base de datos, siga el [tutorial de la API de FHIR con Postman](access-fhir-postman-tutorial.md) para cargar un paciente. Anote el identificador de un paciente específico.

## <a name="download-the-smart-on-fhir-app-launcher"></a>Descarga del iniciador de aplicaciones SMART on FHIR

El [repositorio de FHIR Server para Azure](https://github.com/Microsoft/fhir-server) incluye un sencillo iniciador de aplicaciones SMART on FHIR y un ejemplo de aplicación SMART on FHIR. En este tutorial, use este iniciador de SMART on FHIR de forma local para probar la instalación.

Puede clonar el repositorio de GitHub e ir a la aplicación mediante estos comandos:

```PowerShell
git clone https://github.com/Microsoft/fhir-server
cd fhir-server/samples/apps/SmartLauncher
```

La aplicación necesita algunos valores de configuración, que se pueden establecer en `appsettings.json`:

```json
{
    "FhirServerUrl": "https://MYFHIRAPI.azurehealthcareapis.com",
    "ClientId": "APP-ID",
    "DefaultSmartAppUrl": "/sampleapp/launch.html"
}
```

Se recomienda utilizar la característica `dotnet user-secrets`:

```PowerShell
dotnet user-secrets set FhirServerUrl https://MYFHIRAPI.azurehealthcareapis.com
dotnet user-secrets set ClientId <APP-ID>
```

Use este comando para ejecutar la aplicación:

```PowerShell
dotnet run
```

## <a name="test-the-smart-on-fhir-proxy"></a>Prueba del proxy de SMART on FHIR

Después de iniciar el iniciador de aplicaciones de SMART on FHIR, puede dirigir el explorador a `https://localhost:5001`, donde debería ver la siguiente pantalla:

![Iniciador de aplicaciones de SMART on FHIR](media/tutorial-smart-on-fhir/smart-on-fhir-app-launcher.png)

Cuando escribe la información de los campos **Patient** (Paciente), **Encounter** (Encuentro) o **Practitioner** (Personal sanitario), notará que **Launch context** (Contexto de inicio) se actualiza. Cuando se usa Azure API for FHIR, el contexto de inicio es simplemente un documento JSON que contiene información sobre el paciente, el personal sanitario, etc. Este contexto de inicio está codificado en Base64 y se pasa a la aplicación SMART on FHIR como parámetro de consulta `launch`. Según la especificación de SMART on FHIR, esta variable es opaca para la aplicación SMART on FHIR, y se pasa al proveedor de identidades. 

El proxy de SMART on FHIR usa esta información para rellenar los campos en la respuesta del token. La aplicación SMART on FHIR *puede* usar estos campos para controlar de qué paciente solicita los datos y cómo representa la aplicación en la pantalla. El proxy de SMART on FHIR admite los siguientes campos:

* `patient`
* `encounter`
* `practitioner`
* `need_patient_banner`
* `smart_style_url`

Estos campos están diseñados para proporcionar orientación a la aplicación, pero no proporcionan ninguna información de seguridad. Una aplicación SMART on FHIR puede omitirlos.

Observe que el iniciador de aplicaciones de SMART on FHIR actualiza la información del campo **Launch URL** (Dirección URL de inicio) al final de la página. Seleccione **Launch** (Lanzamiento) para iniciar la aplicación de ejemplo. Acto seguido debería ver algo similar a este ejemplo:

![Aplicación SMART on FHIR](media/tutorial-smart-on-fhir/smart-on-fhir-app.png)

Inspeccione la respuesta del token para ver cómo se pasan los campos de contexto de inicio a la aplicación.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado el proxy de Azure Active Directory SMART on FHIR. Para explorar el uso de SMART on FHIR con Azure API for FHIR y FHIR Server para Azure de código abierto, vaya al repositorio de ejemplos de servidor de FHIR en GitHub:

>[!div class="nextstepaction"]
>[Ejemplos de Servidor de FHIR](https://github.com/Microsoft/fhir-server-samples)
