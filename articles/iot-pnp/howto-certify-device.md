---
title: Certificación de dispositivos IoT Plug and Play | Microsoft Docs
description: Como generador de dispositivos, obtenga información sobre cómo ejecutar pruebas y enviar un dispositivo para su certificación.
author: konichi3
ms.author: koichih
ms.date: 08/21/2020
ms.topic: how-to
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: bdb6bf166e84bb9134bbd14454899bcefbf0a887
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949905"
---
# <a name="how-to-certify-iot-plug-and-play-devices"></a>Certificación de dispositivos IoT Plug and Play

El programa de certificación de dispositivos IoT Plug and Play incluye herramientas para comprobar que un dispositivo cumple los requisitos de certificación de IoT Plug and Play. Las herramientas también ayudan a las organizaciones a impulsar el reconocimiento de la disponibilidad de sus dispositivos IoT Plug and Play. Estos dispositivos certificados se adaptan a las soluciones de IoT y ayudan a reducir el tiempo de comercialización.

En este artículo aprenderá a:

- Instalar la extensión de la herramienta de línea de comandos de Azure IoT para la CLI de Azure
- Ejecutar las pruebas de IoT Plug and Play para validar la aplicación de dispositivo durante la fase de desarrollo  
- Usar el portal Azure Certified Device para validar la aplicación de dispositivo

## <a name="prepare-your-device"></a>Preparación del dispositivo

El código de aplicación que se ejecuta en IoT Plug and Play debe:

- Conectarse a Azure IoT Hub mediante [Device Provisioning Service (DPS)](../iot-dps/about-iot-dps.md).
- Seguir las [convenciones de IoT Plug and Play](concepts-convention.md) para la implementación de telemetría, propiedades y comandos.

La aplicación es un software que se instala de forma independiente del sistema operativo o se empaqueta con el sistema operativo en una imagen de firmware guardada en memoria flash en el dispositivo.

El proceso de certificación comprueba que el dispositivo es compatible con IoT Plug and Play mediante la validación de que la implementación del dispositivo coincide con la telemetría, las propiedades y los comandos definidos en el modelo de dispositivo de [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl) y que el modelo está disponible en el [repositorio de modelos públicos de Azure IoT](concepts-model-repository.md).

Para cumplir los requisitos de certificación, el dispositivo debe:

- Conectarse a Azure IoT Hub mediante [DPS](../iot-dps/about-iot-dps.md).
- Implementar la telemetría, las propiedades o los comandos siguiendo la convención de IoT Plug and Play.
- Describir las interacciones de dispositivos con un modelo de [DTDL v2](https://aka.ms/dtdl).
- Publicar el modelo y todas las interfaces necesarias, en el [repositorio de modelos públicos de Azure IoT](https://devicemodels.azureiotsolutions.com/).
- Enviar el id. de modelo durante el [registro de DPS](concepts-developer-guide.md#dps-payload) en la carga de aprovisionamiento de DPS.
- Anunciar el id. del modelo durante la [conexión de MQTT](concepts-developer-guide.md#model-id-announcement).

## <a name="test-with-the-azure-iot-extension-cli"></a>Prueba con la CLI de la extensión de Azure IoT

La [extensión de la CLI de Azure IoT](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/product?view=azure-cli-latest) permite validar que la implementación del dispositivo coincide con el modelo, antes de enviar el dispositivo para la certificación a través del portal Azure Certified Device.

En los pasos siguientes se muestra cómo preparar y ejecutar las pruebas de certificación mediante la CLI:

### <a name="install-the-azure-iot-extension-for-the-azure-cli"></a>Instalación de la extensión de Azure IoT para la CLI de Azure

Siga las instrucciones de instalación para instalar la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) en su entorno.

Para instalar la extensión de Azure IoT, ejecute el comando siguiente:

```azurecli
az extension add --name azure-iot
```

Para más información, consulte [CLI de Azure para Azure IoT](https://docs.microsoft.com/cli/azure/azure-cli-reference-for-iot?view=azure-cli-latest).

### <a name="create-a-new-product-test"></a>Creación de una nueva prueba de producto

El siguiente comando crea una prueba mediante DPS con un método de atestación de clave simétrica:

- Crea un nuevo producto para probar y genera una configuración de prueba. La salida muestra la información de DPS que el dispositivo debe usar para el aprovisionamiento: la clave principal, el id. de dispositivo y el ámbito del id.
- Especifica la carpeta con los archivos DTDL que describen el modelo.

```azurecli
az iot product test create --badge-type Pnp --at SymmetricKey --device-type FinishedProduct --models {local folder name}
```

> [!NOTE]
> Debe [iniciar sesión](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest) en su suscripción al usar la CLI.

La salida JSON del comando contiene los elementos `primaryKey`, `registrationId` y `scopeID` que se usarán al conectar el dispositivo.

Resultado esperado:

```json
"deviceType": "FinishedProduct",
"id": "d45d53d9-656d-4be7-9bbf-140bc87e98dc",
"provisioningConfiguration": {
  "symmetricKeyEnrollmentInformation": {
    "primaryKey":"Ci/Ghpqp0n4m8FD5PTicr6dEikIfM3AtVWzAhObU7yc=",
    "registrationId": "d45d53d9-656d-4be7-9bbf-140bc87e98dc",
    "scopeId": "0ne000FFA42"
  }
}
```

### <a name="connect-your-device"></a>Conexión del dispositivo

Use la salida de información de DPS del comando anterior para conectar el dispositivo a la instancia de IoT Hub de pruebas.

### <a name="manage-and-configure-the-product-tests"></a>Administración y configuración de las pruebas de productos

Cuando el dispositivo esté conectado y listo para interactuar con IoT Hub, genere un archivo de configuración de prueba del producto. Para crear el archivo:

- Use el elemento `id` de prueba de la salida del comando anterior.
- Use el parámetro `--wait` para obtener el caso de prueba.

```azurecli
az iot product test task create --type GenerateTestCases --test-id d45d53d9-656d-4be7-9bbf-140bc87e98dc --wait
```

Resultado esperado:

```json
{
  "deviceTestId": "d45d53d9-656d-4be7-9bbf-140bc87e98dc",
  "error": null,
  "id": "526da38e-91fc-4e20-a761-4f04b392c42b",
  "resultLink": "/deviceTests/d45d53d9-656d-4be7-9bbf-140bc87e98dc/TestCases",
  "status": "Completed",
  "type": "GenerateTestCases"
}
```

Puede usar el comando `az iot product test case update` para modificar el archivo de configuración de prueba.

### <a name="run-the-tests"></a>Ejecución de las pruebas

Después de generar la configuración de prueba, el paso siguiente es ejecutar las pruebas. Use el mismo elemento `devicetestId` de los comandos anteriores como parámetro para ejecutar las pruebas. Compruebe los resultados de la prueba para asegurarse de que todas las pruebas tienen un estado `Passed`.

```azurecli
az iot product test task create --type QueueTestRun --test-id d45d53d9-656d-4be7-9bbf-140bc87e98dc --wait
```

Ejemplo de salida de la serie de pruebas

```json
      "validationTasks": [
        {
          "componentName": "Default component",
          "endTime": "2020-08-25T05:18:49.5224772+00:00",
          "interfaceId": "dtmi:com:example:TemperatureController;1",
          "logs": [
            {
              "message": "Waiting for telemetry from the device",
              "time": "2020-08-25T05:18:37.3862586+00:00"
            },
            {
              "message": "Validating PnP properties",
              "time": "2020-08-25T05:18:37.3875168+00:00"
            },
            {
              "message": "Validating PnP commands",
              "time": "2020-08-25T05:18:37.3894343+00:00"
            },
            {
              "message": "{\"propertyName\":\"serialNumber\",\"expectedSchemaType\":null,\"actualSchemaType\":null,\"message\":\"Property is successfully validated\",\"passed\":true,\"time\":\"2020-08-25T05:18:37.4205985+00:00\"}",
              "time": "2020-08-25T05:18:37.4205985+00:00"
            },
            {
              "message": "PnP interface properties validation passed",
              "time": "2020-08-25T05:18:37.4206964+00:00"
            },
```

## <a name="test-using-the-azure-certified-device-portal"></a>Prueba mediante el portal Azure Certified Device

En los pasos siguientes se muestra cómo usar el [portal Azure Certified Device](https://aka.ms/acdp) para incorporar, registrar los detalles de los productos, enviar una guía de introducción y ejecutar las pruebas de certificación.

> [!NOTE]
> En el momento de escribir este artículo, el portal no admite la publicación en el [Catálogo de dispositivos Azure Certified for IoT](https://aka.ms/devicecatalog).

### <a name="onboarding"></a>Incorporación

Para usar el [portal de certificación](https://aka.ms/acdp), debe usar una instancia de Azure Active Directory del inquilino profesional o educativo.

Para publicar los modelos en el repositorio de modelos públicos de Azure IoT, su cuenta debe ser miembro de [Microsoft Partner Network](https://partner.microsoft.com). El sistema comprueba que el id. de Microsoft Partner Network existe y que la cuenta se ha examinado completamente antes de publicarlo en el catálogo de dispositivos.

### <a name="company-profile"></a>Perfil de empresa

Puede administrar el perfil de su empresa en el menú de navegación izquierdo. El perfil de empresa incluye la dirección URL de la empresa, la dirección de correo electrónico y el logotipo de la empresa. Debe aceptarse el contrato de programa en esta página, antes de ejecutar cualquier operación de certificación.

La información del perfil de empresa se usa en la descripción del dispositivo que se presenta en el catálogo de dispositivos.

### <a name="create-new-project"></a>Creación de un proyecto

Para certificar un dispositivo, primero debe crear un nuevo proyecto.

Vaya al [portal de certificación](https://aka.ms/acdp). En la página **Proyectos**, seleccione *+ Crear nuevo proyecto*. A continuación, escriba un nombre para el proyecto, el nombre del dispositivo y seleccione una clase de dispositivo.

La información del producto que se proporciona durante el proceso de certificación se divide en cuatro categorías:

- Información del dispositivo. Recopila información sobre el dispositivo, como su nombre, descripción, certificaciones y sistema operativo.
- La guía **Introducción**. Debe enviar la guía como documento PDF para que el administrador del sistema la apruebe antes de publicar el dispositivo.
- Detalles de marketing. Proporcione información de marketing preparada para el cliente del dispositivo. La información de marketing incluye una descripción, una foto y los distribuidores.
- Certificaciones adicionales del sector. Esta sección opcional le permite proporcionar información adicional sobre cualquier otra certificación que el dispositivo haya obtenido.

### <a name="connect-and-test"></a>Conexión y prueba

El paso de conexión y prueba comprueba que el dispositivo cumple los requisitos de certificación de IoT Plug and Play.

Hay tres pasos que se deben completar:

1. Conexión y detección de interfaces. El dispositivo debe conectarse al servicio de certificación de IoT de Azure a través de DPS. Elija el método de autenticación (certificado X.509, claves simétricas o módulo de plataforma de confianza) para usar y actualizar la aplicación de dispositivo con la información de DPS.
1. Revisión de las interfaces. Revise la interfaz y asegúrese de que cada una de ellas tiene entradas de carga con sentido para las pruebas.
1. Hacer pruebas. El sistema prueba cada modelo de dispositivo para comprobar que la telemetría, las propiedades y los comandos descritos en el modelo siguen las convenciones de IoT Plug and Play. Una vez finalizada la prueba, seleccione el vínculo **ver los registros** para ver la telemetría del dispositivo y los datos sin procesar que se envían a las propiedades del dispositivo gemelo de IoT Hub.

## <a name="next-steps"></a>Pasos siguientes

Ahora que el envío del dispositivo se ha completado, puede ponerse en contacto con el equipo de certificación de dispositivos en [iotcert@microsoft.com](mailto:iotcert@microsoft.com) para continuar con los pasos siguientes, entre los que se incluyen la validación de la pertenencia a Microsoft Partner Network y una revisión de las guías de introducción. Una vez que se cumplan todos los requisitos, puede elegir que el dispositivo se incluya en el [Catálogo de dispositivos Azure Certified for IoT](https://aka.ms/devicecatalog).
