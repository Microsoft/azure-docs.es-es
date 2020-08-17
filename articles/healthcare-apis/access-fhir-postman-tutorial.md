---
title: 'Servidor FHIR con Postman en Azure: Azure API for FHIR'
description: Este tutorial le guiará por los pasos necesarios para usar Postman con el fin de acceder a un servidor de FHIR. Postman es útil para depurar aplicaciones que tienen acceso a las API.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 02/07/2019
ms.openlocfilehash: f8b5e344fc963d466571e75ff16f17367dc32971
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844854"
---
# <a name="access-azure-api-for-fhir-with-postman"></a>Acceso a Azure API for FHIR con Postman

Una aplicación cliente accede normalmente a una API de FHIR mediante una [API REST](https://www.hl7.org/fhir/http.html). Puede que también desee interactuar directamente con el servidor de FHIR mientras crea aplicaciones, por ejemplo, con fines de depuración. Este tutorial le guiará por los pasos necesarios para usar [Postman](https://www.getpostman.com/) para acceder a un servidor de FHIR. Postman es una herramienta que se suele usar para la depuración al compilar aplicaciones que tienen acceso a las API.

## <a name="prerequisites"></a>Requisitos previos

- Un punto de conexión de FHIR en Azure. Puede configurarlo mediante Azure API for FHIR administrada o el servidor FHIR de código abierto para Azure. Configure Azure API for FHIR administrada para FHIR mediante [Azure Portal](fhir-paas-portal-quickstart.md), [PowerShell](fhir-paas-powershell-quickstart.md) o la [CLI de Azure](fhir-paas-cli-quickstart.md).
- Una [aplicación cliente](register-confidential-azure-ad-client-app.md) que va a usar para acceder al servicio FHIR
- Postman instalado. Puede obtenerlo en [https://www.getpostman.com](https://www.getpostman.com)

## <a name="fhir-server-and-authentication-details"></a>Detalles de autenticación y servidor de FHIR

Para usar Postman, se necesitan los siguientes detalles:

- La dirección URL del servidor de FHIR, por ejemplo `https://MYACCOUNT.azurehealthcareapis.com`.
- El valor `Authority` del proveedor de identidades para el servidor de FHIR, por ejemplo `https://login.microsoftonline.com/{TENANT-ID}`
- El valor `audience` configurado. Este suele ser la dirección URL del servidor de FHIR, por ejemplo `https://MYACCOUNT.azurehealthcareapis.com`, o simplemente `https://azurehealthcareapis.com`.
- El valor `client_id` (o el identificador de aplicación) de la [aplicación cliente](register-confidential-azure-ad-client-app.md) que va a usar para acceder al servicio FHIR.
- El valor `client_secret` (o el secreto de la aplicación) de la aplicación cliente.

Por último, debe comprobar que `https://www.getpostman.com/oauth2/callback` es una dirección URL de respuesta registrada para la aplicación cliente.

## <a name="connect-to-fhir-server"></a>Conexión al servidor de FHIR

Con Postman, realice una solicitud `GET` para `https://fhir-server-url/metadata`:

![Instrucción de funcionalidad de metadatos de Postman](media/tutorial-postman/postman-metadata.png)

La dirección URL de metadatos para Azure API for FHIR es `https://MYACCOUNT.azurehealthcareapis.com/metadata`. En este ejemplo, la dirección URL del servidor FHIR es `https://fhirdocsmsft.azurewebsites.net` y la instrucción de funcionalidad del servidor está disponible en `https://fhirdocsmsft.azurewebsites.net/metadata`. Dicho punto de conexión debe ser accesible sin autenticación.

Si intenta acceder a recursos restringidos, debería obtener una respuesta de "error de autenticación":

![Error de autenticación](media/tutorial-postman/postman-authentication-failed.png)

## <a name="obtaining-an-access-token"></a>Obtención de un token de acceso

Para obtener un token de acceso válido, seleccione "Authorization" (Autorización) y en TYPE (Tipo) seleccione "OAuth 2.0":

![Establecimiento de OAuth 2.0](media/tutorial-postman/postman-select-oauth2.png)

Presione "Get New Access Token" (Obtener nuevo token de acceso) y aparecerá un cuadro de diálogo:

![Solicitar nuevo token de acceso](media/tutorial-postman/postman-request-token.png)

Necesitará algunos detalles:

| Campo                 | Valor de ejemplo                                                                                                   | Comentario                    |
|-----------------------|-----------------------------------------------------------------------------------------------------------------|----------------------------|
| Nombre del token            | MYTOKEN                                                                                                         | Un nombre que elija          |
| Tipo de concesión            | Código de autorización                                                                                              |                            |
| Dirección URL de devolución de llamadas          | `https://www.getpostman.com/oauth2/callback`                                                                      |                            |
| Dirección URL de autenticación              | `https://login.microsoftonline.com/{TENANT-ID}/oauth2/authorize?resource=<audience>` | El valor`audience` es `https://MYACCOUNT.azurehealthcareapis.com` para Azure API for FHIR |
| Dirección URL del token de acceso      | `https://login.microsoftonline.com/{TENANT ID}/oauth2/token`                                                      |                            |
| Id. de cliente             | `XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXXXXX`                                                                            | Identificador de aplicación             |
| Secreto del cliente         | `XXXXXXXX`                                                                                                        | Clave de cliente secreta          |
| Ámbito | `<Leave Blank>` |
| State                 | `1234`                                                                                                            |                            |
| Autenticación de clientes | Enviar credenciales de cliente en el cuerpo                                                                                 |                 

Presione "Request Token" (Solicitar token) y se le guiará por el flujo de autenticación de Azure Active Directory y se devolverá un token a Postman. Si surgen problemas, abra la consola de Postman desde el elemento de menú "View->Show Postman Console" (Ver > Mostrar consola de Postman).

Desplácese hacia abajo en la pantalla de token devuelto y presione "Use Token" (Usar token):

![Usar token](media/tutorial-postman/postman-use-token.png)

El token se debe rellenar ahora en el campo "Access Token" (Token de acceso), y puede seleccionar tokens en "Available Tokens" (Tokens disponibles). Si vuelve a presionar "Send" (Enviar) para repetir la búsqueda del recurso `Patient`, debe obtener un estado `200 OK`:

![200 OK](media/tutorial-postman/postman-200-OK.png)

En este caso, no hay ningún paciente en la base de datos y la búsqueda está vacía.

Si inspecciona el token de acceso con una herramienta como [https://jwt.ms](https://jwt.ms), debería ver contenido similar a este:

```jsonc
{
  "aud": "https://MYACCOUNT.azurehealthcareapis.com",
  "iss": "https://sts.windows.net/{TENANT-ID}/",
  "iat": 1545343803,
  "nbf": 1545343803,
  "exp": 1545347703,
  "acr": "1",
  "aio": "AUQAu/8JXXXXXXXXXdQxcxn1eis459j70Kf9DwcUjlKY3I2G/9aOnSbw==",
  "amr": [
    "pwd"
  ],
  "appid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "oid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "appidacr": "1",

  ...// Truncated
}
```

En situaciones de solución de problemas, un buen punto de partida es validar que tiene la audiencia correcta (notificación `aud`). Si el token proviene del emisor correcto (notificación `iss`) y tiene la audiencia correcta (notificación `aud`), pero sigue sin poder acceder a la API de FHIR, es probable que el usuario o la entidad de servicio (notificación `oid`) no tengan acceso al plano de datos FHIR. Se recomienda [usar el control de acceso basado en roles de Azure (Azure RBAC)](configure-azure-rbac.md) para asignar roles del plano de datos a los usuarios. Si usa un inquilino externo secundario de Azure Active Directory para su plano de datos, tendrá que [configurar asignaciones de RBAC locales](configure-local-rbac.md).

También puede [obtener un token para Azure API for FHIR mediante la CLI de Azure](get-healthcare-apis-access-token-cli.md). Si usa un token obtenido con la CLI de Azure, debe usar el tipo de autorización "Bearer Token" (Token de portador) y pegar el token directamente.

## <a name="inserting-a-patient"></a>Inserción de un paciente

Ahora ya tiene un token de acceso válido. Con ello puede insertar un nuevo paciente. Cambie al método "POST" y agregue el siguiente documento JSON en el cuerpo de la solicitud:

[!code-json[](samples/sample-patient.json)]

Presione "Send" (Enviar) y verá que el paciente se ha creado correctamente:

![Paciente creado](media/tutorial-postman/postman-patient-created.png)

Si repite la búsqueda de pacientes, ahora debería ver el registro del paciente:

![Paciente creado](media/tutorial-postman/postman-patient-found.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha accedido a una API de FHIR mediante Postman. Lea acerca de las características de API admitidas en nuestra sección de características admitidas.
 
>[!div class="nextstepaction"]
>[Características compatibles](fhir-features-supported.md)
