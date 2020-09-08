---
title: Puntos de conexión de seguridad del servicio de aprovisionamiento de dispositivos IoT | Microsoft Docs
description: Conceptos sobre cómo controlar el acceso de las aplicaciones back-end a IoT Device Provisioning Service (DPS). Incluye información sobre los tokens de seguridad.
author: wesmc7777
manager: philmea
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: wesmc
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 5190ff2575f80a7c1c8c4fc9b386441497de23fb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020327"
---
# <a name="control-access-to-azure-iot-hub-device-provisioning-service"></a>Control de acceso al servicio Azure IoT Hub Device Provisioning

En este artículo se describen las opciones para proteger el servicio de aprovisionamiento de dispositivos IoT. El servicio de aprovisionamiento usa *permisos* para conceder acceso a cada punto de conexión. Los permisos limitan el acceso a una instancia de servicio según la funcionalidad.

En este artículo se describe:

* Los distintos permisos que puede conceder a una aplicación back-end para acceder al servicio de aprovisionamiento.
* El proceso de autenticación y los tokens que utiliza para comprobar los permisos.

### <a name="when-to-use"></a>Cuándo se usa

Debe tener los permisos adecuados para acceder a cualquiera de los puntos de conexión del servicio de aprovisionamiento. Por ejemplo, una aplicación back-end tiene que incluir un token que contenga las credenciales de seguridad junto con cada mensaje que se envía al servicio.

## <a name="access-control-and-permissions"></a>Permisos y control del acceso

Puede conceder los [permisos](#device-provisioning-service-permissions) de las maneras siguientes:

* **Directivas de autorización de acceso compartido**. Las directivas de acceso compartido pueden conceder cualquier combinación de los [permisos](#device-provisioning-service-permissions). Las directivas se pueden definir en [Azure Portal][lnk-management-portal] o mediante programación, para lo que se usan las [API REST de Device Provisioning Service][lnk-resource-provider-apis]. Un servicio de aprovisionamiento recién creado tiene la siguiente directiva predeterminada:

* **provisioningserviceowner**: directiva con todos los permisos.

> [!NOTE]
> Para más detalles, vea [Permisos](#device-provisioning-service-permissions).

## <a name="authentication"></a>Authentication

El servicio Azure IoT Hub Device Provisioning concede acceso a los puntos de conexión mediante la verificación de un token con las directivas de acceso compartido. Las credenciales de seguridad, como las claves simétricas, nunca se envían en la conexión.

> [!NOTE]
> El proveedor de recursos de Device Provisioning Service se protege mediante la suscripción de Azure, igual que todos los proveedores en [Azure Resource Manager][lnk-azure-resource-manager].

Para más información sobre cómo crear y utilizar tokens de seguridad, vea la sección siguiente.

HTTP es el único protocolo compatible e implementa la autenticación mediante la inclusión de un token válido en el encabezado de solicitud **Authorization**.

#### <a name="example"></a>Ejemplo
```csharp
SharedAccessSignature sr = 
   mydps.azure-devices-provisioning.net&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501&skn=provisioningserviceowner`\
```

> [!NOTE]
> Los [SDK de Azure IoT Device Provisioning Service][lnk-sdks] generan tokens automáticamente cuando se conectan al servicio.

## <a name="security-tokens"></a>Tokens de seguridad

El servicio de aprovisionamiento de dispositivos usa tokens de seguridad para autenticar servicios para evitar el envío de claves a través de la red. Además, los tokens de seguridad están limitados en cuanto al ámbito y el período de validez. Los [SDK de Azure IoT Device Provisioning Service][lnk-sdks] generan automáticamente tokens sin necesidad de ninguna configuración especial. Algunos escenarios, requieren que el usuario genere y utilice directamente los tokens de seguridad. Estos escenarios incluyen el uso directo de la superficie HTTP.

### <a name="security-token-structure"></a>Estructura del token de seguridad

Utilice tokens de seguridad para conceder acceso por tiempo limitado a los servicios en la funcionalidad específica del servicio de aprovisionamiento de dispositivos IoT. Para obtener autorización para conectarse al servicio de aprovisionamiento, los servicios deben enviar tokens de seguridad firmados con un acceso compartido o una clave simétrica.

Un token firmado con una clave de acceso compartido concede acceso a toda la funcionalidad asociada con los permisos de la directiva de acceso compartido. 

El token de seguridad tiene el formato siguiente:

`SharedAccessSignature sig={signature}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

Estos son los valores esperados:

| Value | Descripción |
| --- | --- |
| {signature} |Una cadena de firma HMAC-SHA256 con el formato: `{URL-encoded-resourceURI} + "\n" + expiry`. **Importante**: La clave se descodifica en base64 y se utiliza para realizar el cálculo de HMAC-SHA256.|
| {expiry} |Cadenas UTF8 para el número de segundos transcurridos desde el tiempo 00:00:00 UTC el 1 de enero de 1970. |
| {URL-encoded-resourceURI} | Codificación de dirección URL en minúsculas del identificador URI del recurso en minúsculas. Prefijo del identificador URI (por segmento) de los puntos de conexión a los que se puede acceder con este token, que comienza por un nombre de host del servicio de aprovisionamiento de dispositivos IoT (sin protocolo). Por ejemplo, `mydps.azure-devices-provisioning.net`. |
| {policyName} |El nombre de la directiva de acceso compartido a la que hace referencia este token. |

**Nota sobre el prefijo**: el prefijo URI se calcula por segmento y no por carácter. Por ejemplo `/a/b` es un prefijo de `/a/b/c` pero `/a/bc`.

El siguiente fragmento de Node.js muestra una función denominada **generateSasToken** que calcula el token de las entradas `resourceUri, signingKey, policyName, expiresInMins`. Las secciones siguientes detallan cómo inicializar las entradas diferentes para los distintos casos de uso de token.

```javascript
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', new Buffer(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires + "&skn="+ policyName;
    return token;
};
```

Como comparación, el código de Python equivalente para generar un token de seguridad es:

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import quote_plus, urlencode
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((quote_plus(uri)), int(ttl))
    print sign_key
    signature = b64encode(HMAC(b64decode(key), sign_key, sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl)),
        'skn' : policy_name
    }

    return 'SharedAccessSignature ' + urlencode(rawtoken)
```

> [!NOTE]
> Puesto que el período de validez del token se valida en equipos que disponen del servicio de aprovisionamiento de dispositivos IoT, el desfase del reloj del equipo que genera el token debe ser mínimo.

### <a name="use-security-tokens-from-service-components"></a>Uso de tokens de seguridad de los componentes de servicio

Los componentes de servicio solo pueden generar tokens de seguridad mediante directivas de acceso compartido que conceden los permisos apropiados, tal y como se explicó anteriormente.

Estas son las funciones de servicio expuestas en los puntos de conexión:

| Punto de conexión | Funcionalidad |
| --- | --- |
| `{your-service}.azure-devices-provisioning.net/enrollments` |Ofrece operaciones de inscripción de dispositivos con el servicio de aprovisionamiento de dispositivos. |
| `{your-service}.azure-devices-provisioning.net/enrollmentGroups` |Proporciona operaciones para administrar grupos de inscripción de dispositivos. |
| `{your-service}.azure-devices-provisioning.net/registrations/{id}` |Proporciona operaciones para recuperar y administrar el estado de los registros de dispositivos. |


Por ejemplo, un servicio generado con el uso de la directiva de acceso compartido creada previamente denominada **enrollmentread** crearía un token con los parámetros siguientes:

* URI de recurso: `{mydps}.azure-devices-provisioning.net`,
* clave de firma: una de las claves de la directiva `enrollmentread` ,
* nombre de la directiva: `enrollmentread`,
* cualquier fecha de expiración.backn

![Creación de una directiva de acceso compartido para la instancia de servicio de aprovisionamiento de dispositivos en el portal][img-add-shared-access-policy]

```javascript
var endpoint ="mydps.azure-devices-provisioning.net";
var policyName = 'enrollmentread'; 
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

El resultado, que concedería acceso para leer todos los registros de inscripción, sería:

`SharedAccessSignature sr=mydps.azure-devices-provisioning.net&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=enrollmentread`

## <a name="reference-topics"></a>Temas de referencia:

Los siguientes temas de referencia proporcionan más información sobre el control de acceso al servicio de aprovisionamiento de dispositivos IoT.

### <a name="device-provisioning-service-permissions"></a>Permisos para el servicio de aprovisionamiento de dispositivos

En la tabla siguiente se enumeran los permisos que se pueden utilizar para controlar el acceso al servicio de aprovisionamiento de dispositivos IoT.

| Permiso | Notas |
| --- | --- |
| **ServiceConfig** |Concede acceso para cambiar las configuraciones del servicio. <br/>Este permiso se usa en servicios de back-end en la nube. |
| **EnrollmentRead** |Concede acceso de lectura de los grupos de inscripción y las inscripciones de dispositivos. <br/>Este permiso se usa en servicios de back-end en la nube. |
| **EnrollmentWrite** |Concede acceso de escritura de los grupos de inscripción y las inscripciones de dispositivos. <br/>Este permiso se usa en servicios de back-end en la nube. |
| **RegistrationStatusRead** |Concede acceso de lectura del estado de registro de los dispositivos. <br/>Este permiso se usa en servicios de back-end en la nube. |
| **RegistrationStatusWrite**  |Concede acceso de eliminación del estado de registro de los dispositivos. <br/>Este permiso se usa en servicios de back-end en la nube. |

<!-- links and images -->

[img-add-shared-access-policy]: ./media/how-to-control-access/how-to-add-shared-access-policy.PNG
[lnk-sdks]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-management-portal]: https://portal.azure.com
[lnk-azure-resource-manager]: ../azure-resource-manager/management/overview.md
[lnk-resource-provider-apis]: https://docs.microsoft.com/rest/api/iot-dps/
