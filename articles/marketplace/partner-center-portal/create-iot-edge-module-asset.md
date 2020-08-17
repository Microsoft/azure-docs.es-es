---
title: Preparar los recursos técnicos del módulo IoT Edge | Azure Marketplace
description: Obtenga información sobre los requisitos técnicos y de configuración que deben cumplir sus recursos técnicos de un módulo IoT (Internet de las cosas) Edge para poder publicarlos en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 04/03/2020
ms.openlocfilehash: dec0711c4763983e520d247fd8b775b1810e0479
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324648"
---
# <a name="prepare-your-iot-edge-module-technical-assets"></a>Preparar los recursos técnicos del módulo IoT Edge

En este artículo se describen los requisitos que deben cumplir los recursos técnicos del módulo IoT (Internet de las cosas) Edge para que se pueda publicar en Azure Marketplace.

## <a name="get-started"></a>Introducción

Un módulo IoT Edge es un contenedor compatible con Docker que se ejecuta en un dispositivo IoT Edge.

- Para obtener más información sobre los módulos de IoT Edge, vea [Información sobre los módulos de Azure IoT Edge](../../iot-edge/iot-edge-modules.md).
- Para empezar a trabajar en el desarrollo del módulo IoT Edge, consulte [Desarrollar sus propios módulos de IoT Edge](../../iot-edge/module-development.md).

## <a name="technical-requirements"></a>Requisitos técnicos

El módulo IoT Edge debe cumplir los siguientes requisitos técnicos para certificarse y publicarse en Azure Marketplace.

### <a name="platform-support"></a>Compatibilidad con plataformas

El módulo IoT Edge debe admitir una de las siguientes opciones de plataforma:

#### <a name="tier-1-platforms-supported-by-iot-edge"></a>Plataformas de nivel 1 compatibles con IoT Edge

El módulo debe admitir todas las plataformas de nivel 1 compatibles con IoT Edge (según el registro de [Compatibilidad de Azure IoT Edge](../../iot-edge/support.md)). Se recomienda esta opción porque proporciona una mejor experiencia de cliente. Se presentarán los módulos que cumplan este criterio. Un módulo que use esta opción de plataforma debe:

- Proporcionar una etiqueta más reciente y una etiqueta de versión (por ejemplo, 1.0.1) que sean etiquetas de manifiesto compiladas con la [herramienta de manifiestos de GitHub](https://github.com/estesp/manifest-tool).

- Use la pestaña Descripción de la oferta del [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace) para agregar un vínculo en la sección **Vínculos útiles** al [catálogo de dispositivos certificados Azure IoT Edge](https://catalog.azureiotsolutions.com/alldevices?filters={%2218%22:[%221%22]}/).

#### <a name="a-subset-of-tier-1-platforms-supported-by-iot-edge"></a>Un subconjunto de plataformas de nivel 1 compatibles con IoT Edge

El módulo debe admitir un subconjunto (al menos uno) de plataformas de nivel 1 compatibles con IoT Edge (según el registro de [Compatibilidad de Azure IoT Edge](../../iot-edge/support.md)). Un módulo que use esta opción de plataforma debe:

- Proporcionar una etiqueta más reciente y una etiqueta de versión (por ejemplo, 1.0.1) que sean etiquetas de manifiesto compiladas con la [herramienta de manifiestos](https://github.com/estesp/manifest-tool) de GitHub, si hay más de una plataforma compatible. Las etiquetas de manifiesto son opcionales solo cuando hay una plataforma compatible.
- Use la pestaña Descripción de la oferta del [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace) para agregar un vínculo en la sección **Vínculos útiles** a al menos un dispositivo IoT Edge del [catálogo de dispositivos certificados Azure IoT Edge](https://catalog.azureiotsolutions.com/).

:::image type="content" source="media/iot-edge-module-technical-assets-offer-listing.png" alt-text="Se trata de una imagen de la sección Descripción de la oferta del Centro de partners":::

### <a name="device-dimensions"></a>Dimensiones de dispositivos

Las dimensiones de un módulo IoT Edge (como, por ejemplo, CPU, RAM, almacenamiento y GPU) en dispositivos IoT Edge de destino deben cumplir los siguientes requisitos:

- El módulo debe trabajar con al menos un dispositivo IoT Edge del [catálogo de dispositivos certificados Azure IoT Edge](https://catalog.azureiotsolutions.com/).

- Los requisitos mínimos de hardware deben quedar documentados como último párrafo de la descripción de la oferta (en la pestaña Descripción de la oferta del [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace)). También puede indicar los requisitos de hardware recomendados si difieren considerablemente. Por ejemplo, agregue la siguiente sección al final de la descripción de la oferta:

Copie este texto HTML o use las funciones de texto enriquecido correspondientes de la ventana de edición.

```html
<p><u>Minimum hardware requirements:</u> Linux x64 and arm32 OS, 1GB of RAM, 500 Mb of storage</p>
```

### <a name="configuration"></a>Configuración

El módulo debe incluir la configuración predeterminada para que la implementación en un dispositivo IoT Edge se realice de la manera más simple posible. Esta información se puede proporcionar en la página **Configuración técnica** del plan en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace). Es posible que el contenedor también incluya el SDK del módulo IoT Edge para habilitar la comunicación con EdgeHub e IoT Hub.

#### <a name="default-configuration"></a>Configuración predeterminada

Los módulos IoT Edge deben poder iniciarse con la configuración predeterminada proporcionada en la página **Configuración técnica** del plan en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace). Están disponibles los siguientes valores predeterminados:

- **Rutas** predeterminadas
- **Propiedades deseadas del módulo gemelo** predeterminado
- **Variables de entorno** predeterminadas
- **Opciones de creación del contenedor** predeterminadas

En un escenario donde un parámetro necesario para un valor predeterminado no tiene sentido (por ejemplo, la dirección IP del servidor de un cliente), agregue un parámetro como valor predeterminado. Este valor se escribe en mayúsculas y entre corchetes. En este ejemplo, se configura la variable de entorno predeterminada siguiente:

```
ServerIPAddress = <MY_SERVER_IP_ADDRESS>
```

#### <a name="configuration-documentation"></a>Documentación de configuración

Toda la configuración de un módulo IoT Edge debe documentarse con claridad. Por ejemplo, debe documentar cómo usar sus rutas, las propiedades deseadas de dispositivos gemelos, las variables de entorno, createOptions, etc. Debe proporcionar un vínculo a la documentación o convertirla en parte de la descripción del plan o la oferta. Puede proporcionar esta información en la página **Descripción de la oferta** y **Lista del plan** del [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace).

#### <a name="tags-and-versioning"></a>Etiquetas y control de versiones

Los clientes deben poder implementar fácilmente un módulo y obtener automáticamente actualizaciones desde Marketplace (en un escenario de desarrollador). También deben poder usar y congelar una versión exacta que hayan probado (en un escenario de producción).

Para satisfacer las expectativas de estos clientes y publicarse en Marketplace, los módulos IoT Edge deben cumplir los siguientes requisitos:

- Incluir una etiqueta más reciente de manifiesto que apunte a la versión más reciente en todas las plataformas compatibles.
- Crear etiquetas de versión del tipo X.Y.Z, donde X, Y y Z son números enteros.
- Incluir una etiqueta de "versión", como 1.0.1, que apunte a una versión concreta en todas las plataformas compatibles.
- No actualice las etiquetas de "versión", como 1.0.1, ya que no se deben cambiar.

> [!NOTE]
> Opcionalmente, el control de versiones puede incluir etiquetas "revertir versión", como 2.0 y 1.0. Esto permite mantener varias versiones principales en paralelo.

### <a name="telemetry"></a>Telemetría

Los módulos que usan el SDK de módulo IoT deben establecer el identificador de módulo único en PublisherId.OfferId.SkuId para fines de telemetría. Un identificador único ayuda a Azure Marketplace a identificar la cantidad de instancias de módulos que se están ejecutando.

Use uno de los métodos siguientes de los SDK de módulo IoT para establecer ProductInfo en este identificador:

- [C#](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.deviceclient.productinfo?view=azure-dotnet#Microsoft_Azure_Devices_Client_DeviceClient_ProductInfo)
- [C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/Iothub_sdk_options.md)
- [Python](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/Iothub_sdk_options.md)
- [Java](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.productinfo?view=azure-java-stable)

En cuanto a los módulos que no usan el SDK de módulo IoT, puede encontrar detalles menos precisos en el Centro de partners, por ejemplo el número de descargas.

### <a name="security"></a>Seguridad

Los módulos IoT Edge deben evitar los [módulos con privilegios](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities). En su lugar, pida el acceso con menos privilegios como sea posible al host.

### <a name="module-iot-sdk"></a>SDK de módulo IoT

Incluir el SDK de módulo IoT no es un requisito previo para realizar la certificación. Pero, si decide incluir el SDK de módulo IoT, puede proporcionar una experiencia de usuario mejor. Por ejemplo, admitir el enrutamiento o el envío de mensajes a la nube.

El SDK de módulo IoT es necesario para obtener datos de telemetría sobre la cantidad de instancias de módulos en ejecución.

## <a name="recertification-process"></a>Proceso de recertificación

Los asociados reciben una notificación siempre que hay un cambio importante que afecta a sus módulos, como:

- Matriz de compatibilidad de sistema operativo o arquitectura de nivel 1 compatible con IoT Edge
- SDK de módulo IoT
- Entorno de tiempo de ejecución de IoT Edge
- Directrices de certificación de módulo IoT Edge

Los asociados deben actualizar y volver a certificar sus ofertas volviendo a publicarlas en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace).

La oferta también se volverá a certificar si se actualiza, por ejemplo, al agregar nuevas etiquetas de imagen.

## <a name="host-module-in-azure-container-registry"></a>Módulo Host en Azure Container Registry

Para cargar el módulo IoT Edge en Azure Marketplace, primero debe hospedarlo en [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) (ACR). El módulo debe incluir todas las etiquetas que quiere publicar, incluidas las etiquetas de imagen a las que hace referencia una etiqueta de manifiesto. Para obtener más información, consulte el tutorial [Creación de una instancia de Azure Container Registry e inserción de una imagen de contenedor](../../container-instances/container-instances-tutorial-prepare-acr.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de módulo de IoT Edge](azure-iot-edge-module-creation.md)