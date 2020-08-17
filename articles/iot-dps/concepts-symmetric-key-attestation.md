---
title: 'Azure IoT Hub Device Provisioning Service: atestación de clave simétrica'
description: En este artículo se proporciona información general y conceptual del flujo de atestación de calve simétrica usando el servicio IoT Device Provisioning Service (DPS).
author: wesmc7777
ms.author: wesmc
ms.date: 04/04/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: philmea
ms.openlocfilehash: c0a8680dc5c06bd1527b2cca732b58f484101a96
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286502"
---
# <a name="symmetric-key-attestation"></a>Atestación de clave simétrica

En este artículo se describe el proceso de atestación de identidad al usar claves simétricas con el servicio Device Provisioning. 

La atestación de clave simétrica es un enfoque sencillo para autenticar un dispositivo con una instancia del servicio Device Provisioning. Este método de atestación representa una experiencia de "Hola mundo" para los desarrolladores que no estén familiarizados con el aprovisionamiento de dispositivos, o no tengan estrictos requisitos de seguridad. La atestación de dispositivo mediante un [TPM](concepts-tpm-attestation.md) o un [certificado X.509](concepts-security.md#x509-certificates) es más segura y se debe usar cuando los requisitos de seguridad son más estrictos.

Las inscripciones de claves simétricas ofrecen también una excelente forma para que los dispositivos antiguos con funcionalidad de seguridad limitada arranquen en la nube mediante Azure IoT. Para más información acerca de la atestación de clave simétrica con dispositivos antiguos, consulte [Cómo usar las claves simétricas con dispositivos antiguos](how-to-legacy-device-symm-key.md).


## <a name="symmetric-key-creation"></a>Creación de una clave simétrica

De forma predeterminada, Device Provisioning Service crea nuevas claves simétricas con una longitud predeterminada de 64 bytes cuando se guardan nuevas inscripciones con la opción **Generar claves automáticamente** habilitada.

![Generación automática de claves simétricas](./media/concepts-symmetric-key-attestation/auto-generate-keys.png)

También puede proporcionar sus propias claves simétricas para las inscripciones deshabilitando esta opción. Cuando especifique sus propias claves simétricas, estas tienen que tener una longitud de clave entre 16 y 64 bytes. Además, las claves simétricas se tienen que proporcionar en un formato Base64 válido.



## <a name="detailed-attestation-process"></a>Proceso de atestación detallado

La atestación de clave simétrica con el servicio Device Provisioning se realiza con los mismos [tokens de seguridad](../iot-hub/iot-hub-devguide-security.md#security-token-structure) que utilizan los centros de IoT para identificar los dispositivos. Estos tokens de seguridad son [tokens de firma de acceso compartido (SAS)](../service-bus-messaging/service-bus-sas.md). 

Los tokens de SAS tienen una *firma* hash que se crea mediante la clave simétrica. El servicio Device Provisioning vuelve a crear la firma para comprobar si un token de seguridad presentado durante la atestación es auténtico o no.

Los tokens de SAS tendrán este formato:

`SharedAccessSignature sig={signature}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

Presentamos a continuación los componentes de cada token:

| Value | Descripción |
| --- | --- |
| {signature} |Una cadena de firma HMAC-SHA256. Para las inscripciones individuales, esta firma se genera mediante la clave simétrica (principal o secundaria) para realizar el hash. Para los grupos de inscripción, se usa una clave derivada de la clave de grupo de inscripción para realizar el hash. El hash se realiza en un mensaje del formulario: `URL-encoded-resourceURI + "\n" + expiry`. **Importante**: La clave se tiene que descodificar en base64 antes de utilizarla para realizar el cálculo de HMAC-SHA256. Además, el resultado de la firma tiene que codificarse como URL. |
| {resourceURI} |EL URI del punto de conexión de registro al que se puede acceder con este token, empezando por el identificador de ámbito para la instancia del servicio Device Provisioning. Por ejemplo: `{Scope ID}/registrations/{Registration ID}` |
| {expiry} |Cadenas UTF8 para el número de segundos transcurridos desde el tiempo 00:00:00 UTC el 1 de enero de 1970. |
| {URL-encoded-resourceURI} |Codificación de dirección URL en minúsculas del URI del recurso en minúsculas |
| {policyName} |El nombre de la directiva de acceso compartido a la que hace referencia este token. El nombre de directiva que se usó cuando se aprovisionó con la atestación de clave simétrica es **registration**. |

Cuando un dispositivo se atesta con una inscripción individual, el dispositivo usa la clave simétrica que se define en la entrada de inscripción individual para crear la firma hash para el token de SAS.

Para ver ejemplos de código que crea un token de SAS, consulte [Tokens de seguridad](../iot-hub/iot-hub-devguide-security.md#security-token-structure).

La creación de tokens de seguridad para la atestación de clave simétrica es compatible con el SDK de Azure IoT para C. Para obtener un ejemplo del uso del SDK de Azure IoT para C para realizar la atestación con una inscripción individual, consulte [Aprovisionamiento de un dispositivo simulado con claves simétricas](quick-create-simulated-device-symm-key.md).


## <a name="group-enrollments"></a>Inscripciones de grupo

Los dispositivos no usan directamente las claves simétricas para las inscripciones de grupos durante el aprovisionamiento. En su lugar los dispositivos que pertenecen a una inscripción de grupo se aprovisionan mediante una clave de dispositivo derivada. 

En primer lugar, se define un identificador de registro único para la atestación de cada dispositivo con un grupo de inscripción. Los caracteres válidos para el identificador de registro son caracteres alfanuméricos en minúsculas y los guiones ('-'). Este identificador de registro debe ser un valor exclusivo que identifica al dispositivo. Por ejemplo, un dispositivo antiguo puede no admitir muchas características de seguridad. Los dispositivos antiguos solo pueden tener una dirección MAC o un número de serie disponible para identificar de forma exclusiva a ese dispositivo. En ese caso, un identificador de registro puede estar formado por la dirección MAC y el número de serie de forma similar al siguiente:

```
sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6
```

Este ejemplo exacto se usa en el artículo [Aprovisionamiento de dispositivos antiguos mediante claves simétricas](how-to-legacy-device-symm-key.md).

Una vez que se ha definido un identificador de registro para el dispositivo, la clave simétrica para el grupo de inscripción se usa para calcular un hash [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) del identificador de registro para generar una clave de dispositivo derivada. El hash del identificador de registro se puede realizar con el siguiente código de C#:

```csharp
using System; 
using System.Security.Cryptography; 
using System.Text;  

public static class Utils 
{ 
    public static string ComputeDerivedSymmetricKey(byte[] masterKey, string registrationId) 
    { 
        using (var hmac = new HMACSHA256(masterKey)) 
        { 
            return Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(registrationId))); 
        } 
    } 
} 
```

```csharp
String deviceKey = Utils.ComputeDerivedSymmetricKey(Convert.FromBase64String(masterKey), registrationId);
```

La clave del dispositivo resultante se usa para generar un token SAS que se usará para la atestación. A cada dispositivo en un grupo de inscripción se le requiere que realice la atestación usando un token de seguridad generado a partir de una única clave derivada. No se puede usar directamente la clave simétrica del grupo de inscripción para la atestación.

#### <a name="installation-of-the-derived-device-key"></a>Instalación de la clave derivada de dispositivo

Lo ideal es que las claves de dispositivo se deriven e instalen en la fábrica. Este método garantiza que la clave de grupo no se incluirá nunca en cualquier software implementado en el dispositivo. Cuando al dispositivo se le asigna una dirección MAC o un número de serie, la clave se puede derivar e insertar en el dispositivo de la forma en la que el fabricante la almacene.

Considere el siguiente diagrama que muestra una tabla de claves de dispositivo generadas en una fábrica aplicando un hash a cada identificador de registro de dispositivo con la clave de inscripción de grupo (**K**). 

![Claves de dispositivo asignadas en una fábrica](./media/concepts-symmetric-key-attestation/key-diversification.png)

La identidad de cada dispositivo se representa mediante el identificador de registro y la clave de dispositivo derivada que se instala en fábrica. La clave de dispositivo nunca se copia en otra ubicación y la clave de grupo nunca se almacena en un dispositivo.

Si las claves de dispositivo no se instalan en la fábrica, se debe utilizar un [módulo de seguridad de hardware (HSM)](concepts-security.md#hardware-security-module) para almacenar de forma segura la identidad del dispositivo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un conocimiento de la atestación de clave simétrica, consulte los artículos siguientes para aprender más sobre el tema:

* [Guía de inicio rápido: Aprovisionamiento de un dispositivo simulado con claves simétricas](quick-create-simulated-device-symm-key.md)
* [Conceptos de aprovisionamiento automático](./concepts-auto-provisioning.md)
* [Introducción al uso del aprovisionamiento automático](./quick-setup-auto-provision.md) 
