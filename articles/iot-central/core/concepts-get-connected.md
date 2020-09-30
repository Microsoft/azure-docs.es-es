---
title: Conectividad de dispositivos en Azure IoT Central | Microsoft Docs
description: En este artículo se presentan conceptos clave relacionados con la conectividad de dispositivos en Azure IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: philmea
ms.custom:
- amqp
- mqtt
- device-developer
ms.openlocfilehash: 834d3bd3e41be0487a3d05f00846bcb58bfe00a8
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90018203"
---
# <a name="get-connected-to-azure-iot-central"></a>Conexión a Azure IoT Central

*Este artículo se aplica a los operadores y desarrolladores de dispositivos.*

En este artículo se describen las opciones para conectar los dispositivos a una aplicación de Azure IoT Central.

Normalmente, debe registrar un dispositivo en la aplicación antes de conectarlo. Sin embargo, IoT Central admite escenarios en los que los [dispositivos pueden conectarse sin necesidad de registrarlos primero](#connect-without-registering-devices).

IoT Central usa el [servicio Azure IoT Hub Device Provisioning (DPS)](../../iot-dps/about-iot-dps.md) para administrar el proceso de conexión. Un dispositivo se conecta primero a un punto de conexión de DPS para recuperar la información necesaria para conectarse a la aplicación. De manera interna, la aplicación de IoT Central usa un centro de IoT para administrar la conectividad de los dispositivos. El uso de DPS permite:

- Que IoT Central admite la incorporación y la conexión de dispositivos a escala.
- Que usted pueda generar credenciales de dispositivo y configurar los dispositivos sin conexión sin tener que registrarlos primero en la interfaz de usuario de IoT Central.
- Que usted pueda usar identificadores de dispositivo propios para registrar estos en IoT Central. Esto simplifica la integración con sistemas del área de operaciones existentes.
- Una manera sencilla y coherente de conectar dispositivos a IoT Central.

Para proteger la comunicación entre un dispositivo y la aplicación, IoT Central admite firmas de acceso compartido (SAS) y certificados X.509. Los certificados X.509 se recomiendan en entornos de producción.

En este artículo se describen los siguientes casos de uso:

- [Conexión rápida a un solo dispositivo con SAS](#connect-a-single-device)
- [Conexión de dispositivos a escala con SAS](#connect-devices-at-scale-using-sas)
- [Conexión de dispositivos a escala con certificados X.509](#connect-devices-using-x509-certificates). Este es el enfoque recomendado para entornos de producción.
- [Conexión de dispositivos sin registrarlos primero](#connect-without-registering-devices)
- [Conexión de dispositivos que usan inscripciones de DPS individuales](#individual-enrollment-based-device-connectivity)
- [Asociación automática de un dispositivo a una plantilla de dispositivo](#automatically-associate-with-a-device-template)

## <a name="connect-a-single-device"></a>Conectar un solo dispositivo

Este enfoque es útil cuando está experimentando con IoT Central o probado dispositivos. Puede usar las claves SAS de conexión de dispositivos de la aplicación de IoT Central para conectar un dispositivo a dicha aplicación. Copie la _clave SAS del dispositivo_ de la información de conexión de un dispositivo registrado:

![Claves SAS de un dispositivo individual](./media/concepts-get-connected/single-device-sas.png)

Para más información, consulte el tutorial [Creación y conexión de una aplicación cliente Node.js a una aplicación de Azure IoT Central](./tutorial-connect-device-nodejs.md).

## <a name="connect-devices-at-scale-using-sas"></a>Conexión de dispositivos a escala con SAS

Para conectar dispositivos a IoT Central a escala con claves SAS, es preciso registrarlos y configurarlos:

### <a name="register-devices-in-bulk"></a>Registro de dispositivos de forma masiva

Para registrar un gran número de dispositivos con la aplicación de IoT Central, use un archivo CSV para [importar identificadores y nombres de dispositivo](howto-manage-devices.md#import-devices).

Para recuperar la información de conexión de los dispositivos importados, [exporte un archivo CSV desde la aplicación de IoT Central](howto-manage-devices.md#export-devices). El archivo CSV exportado incluye los identificadores de dispositivo y las claves SAS.

### <a name="set-up-your-devices"></a>Configuración de los dispositivos

Utilice la información de conexión del archivo de exportación del código del dispositivo para permitir que los dispositivos se conecten a la aplicación de IoT Central y envíen datos. También necesita el **ámbito de identificador** de DPS de la aplicación. Puede encontrar este valor en **Administration > Device connection** (Administración > Conexión de dispositivos).

> [!NOTE]
> Para aprender a conectar dispositivos sin registrarlos primero en IoT Central, consulte [Conectarse sin registro de dispositivos](#connect-without-registering-devices).

## <a name="connect-devices-using-x509-certificates"></a>Conexión de dispositivos con certificados X.509

En un entorno de producción, se recomienda el uso de certificados X.509 como mecanismo de autenticación de dispositivos para IoT Central. Para más información consulte [Autenticación de dispositivos mediante certificados de entidades de certificación X.509](../../iot-hub/iot-hub-x509ca-overview.md).

Para conectar un dispositivo con un certificado X.509 a la aplicación, haga lo siguiente:

1. Cree un *grupo de inscripción* que use el tipo de atestación **Certificados (X.509)** .
2. Agregue y verifique un certificado X.509 intermedio o raíz en el grupo de inscripción.
3. Registre y conecte dispositivos que usen certificados X.509 de hoja generados a partir del certificado raíz o intermedio en el grupo de inscripción.

### <a name="create-an-enrollment-group"></a>Creación de un grupo de inscripción

Un [grupo de inscripción](../../iot-dps/concepts-service.md#enrollment) es un grupo de dispositivos que comparten el mismo tipo de atestación. Los dos tipos de atestación admitidos son certificados X.509 y SAS:

- En un grupo de inscripción X.509, todos los dispositivos que se conectan a IoT Central utilizan certificados X.509 de hoja generados a partir del certificado raíz o intermedio en el grupo de inscripción.
- En un grupo de inscripción SAS, todos los dispositivos que se conectan a IoT Central utilizan un token SAS generado a partir del token SAS en el grupo de inscripción.

Los dos grupos de inscripción predeterminados de cada aplicación de IoT Central son grupos de inscripción SAS: uno para dispositivos IoT y otro para dispositivos Azure IoT Edge. Para crear un grupo de inscripción X.509, vaya a la página **Conexión del dispositivo** y seleccione **+ Agregar grupo de inscripciones**:

:::image type="content" source="media/concepts-get-connected/add-enrollment-group.png" alt-text="Captura de pantalla de adición de un grupo de inscripción X.509":::

### <a name="add-and-verify-a-root-or-intermediate-x509-certificate"></a>Adición y verificación de un certificado X.509 raíz o intermedio

Para agregar y verificar un certificado raíz o intermedio en su grupo de inscripción, haga lo siguiente:

1. Vaya al grupo de inscripción X.509 que acaba de crear. Tiene la opción de agregar certificados X.509 principales y secundarios. Seleccione **+ Administrar principal**.

1. En la página **Certificado principal**, cargue el certificado X.509 principal. Este es su certificado raíz o intermedio:

    :::image type="content" source="media/concepts-get-connected/upload-primary-certificate.png" alt-text="Captura de pantalla del certificado principal":::

1. Use el **código de verificación** para generar un código de verificación en la herramienta que usa. A continuación, seleccione **Verificar** para cargar el certificado de verificación.

1. Cuando la verificación se haya realizado correctamente, verá la confirmación siguiente:

    :::image type="content" source="media/concepts-get-connected/verified-primary-certificate.png" alt-text="Captura de pantalla de certificado principal verificado":::

La verificación de la titularidad del certificado garantiza que la persona que carga el certificado está en posesión de la clave privada.

Si sufre una infracción de seguridad o si el certificado principal está configurado para expirar, use el certificado secundario para reducir el tiempo de inactividad. Puede continuar con el aprovisionamiento de los dispositivos mediante el certificado secundario al actualizar el principal.

### <a name="register-and-connect-devices"></a>Registro y conexión de dispositivos

Para conectar dispositivos de forma masiva mediante certificados X.509, registre primero los dispositivos en la aplicación mediante un archivo CSV para [importar los identificadores y los nombres de los dispositivos](howto-manage-devices.md#import-devices). Todos los identificadores de dispositivo deben estar en minúsculas.

Genere certificados X.509 de hoja para los dispositivos con el certificado raíz o intermedio que cargó en el grupo de inscripción X.509. Use el valor de **Device ID** (Id. de dispositivo) como valor `CNAME` en los certificados de hoja. El código del dispositivo necesita el valor de **ID scope** (Id. de ámbito) de la aplicación, el valor de **device ID** (Id. de dispositivo) y el certificado de dispositivo correspondiente.

#### <a name="sample-device-code"></a>Código del dispositivo de ejemplo

En el ejemplo siguiente del [SDK de Azure IoT para Node.JS](https://github.com/Azure/azure-iot-sdk-node/blob/master/provisioning/device/samples/register_x509.js) se muestra cómo un cliente de dispositivo de Node.js usa un certificado de hoja X.509 y un DPS para registrarse en una aplicación de IoT Central:

:::code language="nodejs" source="~/azure-iot-sdk-node/provisioning/device/samples/register_x509.js":::

Para obtener un ejemplo equivalente de C, consulte [prov_dev_client_sample.c](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/samples/prov_dev_client_sample/prov_dev_client_sample.c) en el [SDK de cliente de Provisioning Device de C de Azure IoT](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client/devdoc/using_provisioning_client.md).

### <a name="for-testing-purposes-only"></a>Solo para fines de prueba

Solo para pruebas, puede usar las siguientes utilidades para generar certificados raíz, intermedio y de dispositivo:

- [Herramientas del SDK de dispositivo de Azure IoT Device Provisioning](https://github.com/Azure/azure-iot-sdk-node/blob/master/provisioning/tools/readme.md): colección de herramientas de Node.js que se puede usar para generar y verificar claves y certificados X.509.
- Si usa un dispositivo DevKit, esta [herramienta de la línea de comandos](https://aka.ms/iotcentral-docs-dicetool) genera un certificado de entidad de certificación que puede agregar a la aplicación de IoT Central para verificar los certificados.
- [Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md): una colección de scripts de PowerShell y Bash para:
  - Crear una cadena de certificados.
  - Guardar los certificados como archivos .cer para cargarlos en la aplicación de IoT Central.
  - Usar el código de verificación de la aplicación de IoT Central para generar el certificado de verificación.
  - Crear certificados de hoja para los dispositivos mediante sus identificadores de dispositivo como parámetro en la herramienta.

## <a name="connect-without-registering-devices"></a>Conexión sin registro de los dispositivos

En los escenarios descritos anteriormente, es necesario registrar los dispositivos de la aplicación antes de conectarlos. IoT Central también permite a los OEM fabricar dispositivos de forma masiva que puedan conectarse sin registrarse primero. Un OEM genera las credenciales de dispositivo adecuadas y configura los dispositivos en la fábrica. Cuando un cliente enciende un dispositivo por primera vez, se conecta a DPS, que, a continuación, conecta automáticamente el dispositivo a la aplicación de IoT Central correcta. Un operador de IoT Central debe aprobar el dispositivo antes de empezar a enviar datos a la aplicación.

El flujo es ligeramente diferente en función de si los dispositivos usan tokens de SAS o certificados X.509:

### <a name="connect-devices-that-use-sas-tokens-without-registering"></a>Conexión de los dispositivos que usan tokens de SAS sin registrarse

1. Copie la clave principal del grupo desde el grupo de inscripción **SAS-IoT-Devices**:

    :::image type="content" source="media/concepts-get-connected/group-primary-key.png" alt-text="Clave principal del grupo desde el grupo de inscripción SAS-IoT-Devices":::

1. Use el comando `az iot central device compute-device-key` para generar las claves SAS del dispositivo. Use la clave principal del grupo del paso anterior. El identificador del dispositivo debe estar en minúsculas:

    ```azurecli
    az iot central device compute-device-key --primary-key <enrollment group primary key> --device-id <device ID>
    ```

1. El OEM carga cada dispositivo con un identificador de dispositivo, una clave SAS de dispositivo generada y el valor de **ID scope** (Ámbito de id.) de la aplicación.

1. Al encender un dispositivo, primero se conecta a DPS para recuperar la información de registro de IoT Central.

    Inicialmente, el dispositivo tiene un estado **Unassociated** (No asociado) en la página **Devices** (Dispositivos) y no está asignado a una plantilla de dispositivo. En la página **Devices** (Dispositivos), seleccione **Migrate** (Migrar) para migrar el dispositivo a la plantilla de dispositivo adecuada. El aprovisionamiento de dispositivos ya está completo, el estado del dispositivo es ahora **Provisioned** (Aprovisionado) y el dispositivo puede empezar a enviar datos.

    En la página **Administration > Device connection** (Administración > Conexión de dispositivos), la opción **Auto approve** (Aprobar automáticamente) controla si debe aprobar el dispositivo manualmente antes de que pueda empezar a enviar datos.

    > [!NOTE]
    > Para aprender a asociar automáticamente un dispositivo a una plantilla de dispositivo, consulte [Asociación automática de un dispositivo a una plantilla de dispositivo](#automatically-associate-with-a-device-template).

### <a name="connect-devices-that-use-x509-certificates-without-registering"></a>Conexión de dispositivos que usan certificados X.509 sin registrarse

1. [Cree un grupo de inscripción](#create-an-enrollment-group) y, luego, [agregue y verifique un certificado X.509 raíz o intermedio](#add-and-verify-a-root-or-intermediate-x509-certificate) a la aplicación de IoT Central.

1. genere los certificados de hoja para sus dispositivos mediante el certificado raíz o intermedio que agregó a la aplicación de IoT Central. Use los identificadores de dispositivo en minúsculas como `CNAME` en los certificados de hoja.

1. El OEM carga cada dispositivo con un identificador de dispositivo, un certificado X.509 de hoja generado y el valor de **ID scope** (Ámbito de id.) de la aplicación.

1. Al encender un dispositivo, primero se conecta a DPS para recuperar la información de registro de IoT Central.

    Inicialmente, el dispositivo tiene un estado **Unassociated** (No asociado) en la página **Devices** (Dispositivos) y no está asignado a una plantilla de dispositivo. En la página **Devices** (Dispositivos), seleccione **Migrate** (Migrar) para migrar el dispositivo a la plantilla de dispositivo adecuada. El aprovisionamiento de dispositivos ya está completo, el estado del dispositivo es ahora **Provisioned** (Aprovisionado) y el dispositivo puede empezar a enviar datos.

    En la página **Administration > Device connection** (Administración > Conexión de dispositivos), la opción **Auto approve** (Aprobar automáticamente) controla si debe aprobar el dispositivo manualmente antes de que pueda empezar a enviar datos.

    > [!NOTE]
    > Para aprender a asociar automáticamente un dispositivo a una plantilla de dispositivo, consulte [Asociación automática de un dispositivo a una plantilla de dispositivo](#automatically-associate-with-a-device-template).

## <a name="individual-enrollment-based-device-connectivity"></a>Conectividad de dispositivos basada en inscripciones individuales

Para los clientes que conectan dispositivos que tienen sus propias credenciales de autenticación, use inscripciones individuales. Una inscripción individual es una entrada para un único dispositivo que tiene permiso para conectarse. Las inscripciones individuales pueden usar certificados X.509 de hoja o tokens de SAS (de un módulo de plataforma segura físico o virtual) como mecanismos de atestación. El identidad del dispositivo (también conocido como identificador de registro) de una inscripción individual está formado por caracteres alfanuméricos y en minúscula, y puede contener guiones. Para más información, consulte [Inscripción individual de DPS](https://docs.microsoft.com/azure/iot-dps/concepts-service#individual-enrollment).

> [!NOTE]
> Cuando se crea una inscripción individual para un dispositivo, tiene prioridad sobre las opciones de inscripción de grupo predeterminadas de la aplicación de IoT Central.

### <a name="create-individual-enrollments"></a>Creación de inscripciones individuales

IoT Central admite los siguientes mecanismos de atestación para las inscripciones individuales:

- **Atestación de clave simétrica:** la atestación de clave simétrica es un enfoque sencillo para autenticar un dispositivo con una instancia de DPS. Para crear una inscripción individual que use claves simétricas, abra la página **Device Connection** (Conexión de dispositivos), seleccione **Individual enrollment** (Inscripción individual) como método de conexión y **Shared access signature (SAS)** (Firma de acceso compartido [SAS]) como mecanismo. Escriba las claves principal y secundaria codificadas en Base64 y guarde los cambios. Use el valor de **ID scope** (Ámbito de id.), **Device ID** (Id. de dispositivo) y la clave principal o secundaria para conectarse al dispositivo.

    > [!TIP]
    > Con fines de prueba, puede usar **OpenSSL** para generar claves codificadas en Base64: `openssl rand -base64 64`

- **Certificados X.509:** Para crear una inscripción individual con certificados X.509, abra la página **Device Connection** (Conexión de dispositivos), seleccione **Individual enrollment** (Inscripción individual) como método de conexión y **Certificates (X.509)** (Certificados [X.509]) como mecanismo. Los certificados de dispositivo usados con una entrada de inscripción individual tienen el requisito de que el emisor y el CN del firmante están establecidos en el identificador de dispositivo.

    > [!TIP]
    > Con fines de prueba, puede usar las [herramientas para el SDK de dispositivo de Azure IoT Device Provisioning para Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/tools) para generar un certificado autofirmado: `node create_test_cert.js device "mytestdevice"`

- **Atestación del Módulo de plataforma segura (TPM):** Un [TPM](https://docs.microsoft.com/azure/iot-dps/concepts-tpm-attestation) es un tipo de módulo de seguridad de hardware. El uso de un TPM es una de las formas más seguras de conectar un dispositivo. En este artículo se supone que usa un TPM discreto, de firmware o integrado. Los TPM emulados por software son adecuados para prototipos o pruebas, pero no brindan el mismo nivel de seguridad que los TPM discretos, de firmware o integrados. No use TPM de software en producción. Para crear una inscripción individual que use un TPM, abra la página **Device Connection** (Conexión de dispositivos), seleccione **Individual enrollment** (Inscripción individual) como método de conexión y **TPM** como mecanismo. Escriba la clave de aprobación de TPM y guarde la información de conexión del dispositivo.

## <a name="automatically-associate-with-a-device-template"></a>Asociación automática a una plantilla de dispositivo

Una de las características clave de IoT Central es la posibilidad de asociar plantillas de dispositivo automáticamente en la conexión del dispositivo. Junto con las credenciales del dispositivo, los dispositivos pueden enviar el valor de **CapabilityModelId** como parte de la llamada del registro del dispositivo. **CapabilityModelID** es un URN que identifica el modelo de funcionalidad que el dispositivo implementa. La aplicación IoT Central puede usar **CapabilityModelID** para identificar la plantilla de dispositivo que se va a usar y, a continuación, asociarle el dispositivo de forma automática. El proceso de detección funciona de la siguiente manera:

1. Si la plantilla de dispositivo ya está publicada en la aplicación IoT Central, el dispositivo se le asocia.
1. En el caso de los dispositivos IoT Plug and Play certificados previamente, si la plantilla de dispositivo no se ha publicado en la aplicación IoT Central, se captura desde el repositorio público.

En los fragmentos de código siguientes se muestra el formato de la carga adicional que el dispositivo debe enviar durante la llamada de registro de DPS para que funcione la asociación automática.

Este es el formato para los dispositivos que usan el SDK de dispositivo disponible con carácter general:

```javascript
    iotcModelId: '< this is the URN for the capability model>';
```

Este es el formato de los dispositivos que usan el SDK de dispositivo de versión preliminar pública:

```javascript
'__iot:interfaces': {
    CapabilityModelId: <this is the URN for the capability model>
}
```

> [!NOTE]
> La opción **Auto approve** (Aprobación automática) de **Administration > Device connection** (Administración > Conexión de dispositivos) debe estar habilitada para que los dispositivos se conecten automáticamente, detecten la plantilla de dispositivo y comiencen a enviar datos.

## <a name="device-status-values"></a>Valores de estado del dispositivo

Cuando un dispositivo real se conecta a la aplicación de IoT Central, su estado de dispositivo cambia como sigue:

1. Primero, el estado del dispositivo es **Registrado**. Este estado significa que el dispositivo se ha creado en IoT Central y tiene un identificador. El dispositivo está registrado cuando:
    - Se agrega un nuevo dispositivo real en la página **Dispositivos**.
    - Se agrega un conjunto de dispositivos mediante **Importar** en la página **Dispositivos**.

1. El estado del dispositivo cambia a **Aprovisionado** cuando el dispositivo conectado a la aplicación de IoT Central con credenciales válidas completa el paso de aprovisionamiento. En este paso, el dispositivo usa DPS para recuperar automáticamente una cadena de conexión de IoT Hub que usa la aplicación de IoT Central. El dispositivo puede conectarse ahora a IoT Hub y empezar a enviar datos.

1. Los dispositivos los puede bloquear un operador. Cuando están bloqueados, no pueden enviar datos a la aplicación de IoT Central. Los dispositivos bloqueados tienen un estado de **Bloqueado**. Un operador debe restablecer el dispositivo para que pueda volver a enviar datos. Cuando un operador desbloquea un dispositivo, el estado vuelve a su valor anterior, **Registrado** o **Aprovisionado**.

1. Si el estado del dispositivo es **Waiting for Approval** (Esperando aprobación), significa que la opción **Auto approve** (Aprobación automática) está deshabilitada. Un operador debe aprobar explícitamente un dispositivo antes de que empiece a enviar datos. Los dispositivos no registrados manualmente en la página **Dispositivos**, pero conectados con credenciales válidas, tendrán el estado del dispositivo **En espera de aprobación**. Los operadores pueden aprobar estos dispositivos desde la página **Dispositivos** mediante el botón **Aprobar**.

1. Si el estado del dispositivo es **Unassociated** (No asociado), significa que el dispositivo que se conecta a IoT Central no tiene una plantilla de dispositivo asociada. Esta situación suele darse en los escenarios siguientes:

    - Se agrega un conjunto de dispositivos mediante la opción **Import** (Importar) de la página **Devices** (Dispositivos) sin especificar la plantilla de dispositivo.
    - Se registró un dispositivo manualmente en la página **Devices** (Dispositivos) sin especificar la plantilla de dispositivo. Después, el dispositivo se conecta con credenciales válidas.  

    El operador puede asociar un dispositivo a una plantilla de dispositivo desde la página **Devices** (Dispositivos) mediante el botón **Migrate** (Migrar).

## <a name="best-practices"></a>Procedimientos recomendados

No guarde ni almacene en caché la cadena de conexión de dispositivo que DPS devuelve cuando conecta el dispositivo por primera vez. Para volver a conectar un dispositivo, recorra el flujo de registro de dispositivos estándar para obtener la cadena de conexión de dispositivo correcta. Si el dispositivo almacena en caché la cadena de conexión, el software del dispositivo corre el riesgo de tener una cadena de conexión obsoleta si IoT Central actualiza el centro de IoT de Azure subyacente que usa.

## <a name="sdk-support"></a>Compatibilidad con SDK

La oferta de SDK de dispositivos de Azure es la manera más fácil de implementar el código del dispositivo. En la actualidad, están disponibles los siguientes SDK:

- [SDK de Azure IoT para C](https://github.com/azure/azure-iot-sdk-c)
- [SDK de Azure IoT para Python](https://github.com/azure/azure-iot-sdk-python)
- [SDK de Azure IoT para Node.js](https://github.com/azure/azure-iot-sdk-node)
- [SDK de Azure IoT para Java](https://github.com/azure/azure-iot-sdk-java)
- [SDK de Azure IoT para .NET](https://github.com/azure/azure-iot-sdk-csharp)

### <a name="sdk-features-and-iot-hub-connectivity"></a>Características del SDK y la conectividad de IoT Hub

Toda comunicación del dispositivo con la instancia de IoT Hub utiliza las siguientes opciones de conectividad:

- [Mensajería del dispositivo a la nube](../../iot-hub/iot-hub-devguide-messages-d2c.md)
- [Dispositivos gemelos](../../iot-hub/iot-hub-devguide-device-twins.md)

En la tabla siguiente se resume cómo las características de los dispositivos de Azure IoT Central se asignan a las características de IoT Hub:

| Azure IoT Central | Azure IoT Hub |
| ----------- | ------- |
| Telemetría | Mensajería de un dispositivo a la nube |
| Propiedad | Propiedades notificadas de dispositivos gemelos |
| Propiedad (grabable) | Propiedades deseadas y notificadas de dispositivos gemelos |
| Get-Help | Métodos directos |

### <a name="protocols"></a>Protocolos

Los SDK de dispositivo admiten los siguientes protocolos de red para conectarse a un centro de IoT:

- MQTT
- AMQP
- HTTPS

Para obtener información sobre estos protocolos de diferencia y guía sobre cómo elegir uno, consulte [Elección de un protocolo de comunicación](../../iot-hub/iot-hub-devguide-protocols.md).

Si el dispositivo no puede usar ninguno de los protocolos admitidos, utilice Azure IoT Edge para realizar la conversión de protocolos. IoT Edge es compatible con otros escenarios de inteligencia perimetral para descargar el procesamiento al perímetro desde la aplicación Azure IoT Central.

## <a name="security"></a>Seguridad

Todos los datos intercambiados entre los dispositivos y la instancia de Azure IoT Central están cifrados. IoT Hub autentica todas las solicitudes de un dispositivo que se conecta a cualquiera de los puntos de conexión de IoT Hub orientados al dispositivo. Para evitar el intercambio de credenciales por la red, el dispositivo utiliza tokens firmados para autenticarse. Para más información, consulte [Control del acceso a IoT Hub](../../iot-hub/iot-hub-devguide-security.md).

## <a name="next-steps"></a>Pasos siguientes

Si es desarrollador de dispositivos, algunos de los pasos sugeridos son los siguientes:

- Aprender a [supervisar la conectividad de dispositivos mediante la CLI de Azure](./howto-monitor-devices-azure-cli.md)
- Aprender a [definir un nuevo tipo de dispositivo IoT en la aplicación de Azure IoT Central](./howto-set-up-template.md)
- Leer sobre los [dispositivos de Azure IoT Edge y Azure IoT Central](./concepts-iot-edge.md)
