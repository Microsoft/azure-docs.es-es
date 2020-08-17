---
title: Glosario de términos de Azure IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: glosario en el que se explican algunos de los términos comunes empleados en los artículos de Azure IoT Hub.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/15/2019
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 2b7a81357cc6ee2608d2acd3051dcac3e5d0e07e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87023473"
---
# <a name="glossary-of-iot-hub-terms"></a>Glosario de términos de IoT Hub

En este artículo se enumeran algunos de los términos comunes que se utilizan en los artículos de IoT Hub.

## <a name="advanced-message-queueing-protocol"></a>Advanced Message Queueing Protocol

[Advanced Message Queueing Protocol (AMQP)](https://www.amqp.org/) es uno de los protocolos de mensajería que [IoT Hub](#iot-hub) admite para la comunicación con dispositivos. Para obtener más información acerca de los protocolos de mensajería que admite IoT Hub, consulte [Enviar y recibir mensajes con IoT Hub](iot-hub-devguide-messaging.md).

## <a name="automatic-device-management"></a>Administración automática de dispositivos

La administración automática de dispositivos de Azure IoT Hub automatiza muchas de las tareas repetitivas y complejas de administración de grandes flotas de dispositivos durante su ciclo de vida completo. Con la administración automática de dispositivos, puede tener como destino un conjunto de dispositivos según sus propiedades, definir una configuración deseada y permitir que IoT Hub actualice los dispositivos en cuanto estén dentro del alcance.  Consta de [configuraciones automáticas de dispositivos](iot-hub-auto-device-config.md) e [implementaciones automáticas de IoT Edge](../iot-edge/how-to-deploy-at-scale.md).

## <a name="automatic-device-configuration"></a>Configuración automática de dispositivos

El back-end de la solución puede usar las [configuraciones automáticas de dispositivos](iot-hub-auto-device-config.md) para asignar las propiedades deseadas a un conjunto de [dispositivos gemelos](#device-twin) e informar del estado mediante métricas del sistema y métricas personalizadas. 

## <a name="azure-classic-cli"></a>CLI de Azure clásica

La [CLI de Azure clásica](../cli-install-nodejs.md) es una herramienta de comandos multiplataforma, de código abierto y basada en shell que se utiliza para crear y administrar recursos en Microsoft Azure. Esta versión de la CLI solo debe usarse para las implementaciones clásicas.

## <a name="azure-cli"></a>Azure CLI

La [CLI de Azure](https://docs.microsoft.com/cli/azure/install-az-cli2) es una herramienta de comandos multiplataforma, de código abierto y basada en shell que se utiliza para crear y administrar recursos en Microsoft Azure.

## <a name="azure-iot-device-sdks"></a>SDK de dispositivos IoT de Azure

Hay _SDK de dispositivos_ disponibles para varios idiomas que permiten crear [aplicaciones para dispositivo](#device-app) que interactúan con un centro de IoT. Los tutoriales de IoT Hub muestran cómo se utilizan estos SDK de dispositivos. Puede encontrar el código fuente y más información sobre los SDK de dispositivos en este [repositorio](https://github.com/Azure/azure-iot-sdks) de GitHub.

## <a name="azure-iot-explorer"></a>Azure IoT Explorer

[Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer) se usa para ver la telemetría que el dispositivo envía, trabajar con propiedades de dispositivo y llamar a comandos. También puede usar el explorador para interactuar con los dispositivos y probarlos, y para administrar dispositivos Plug and Play.

## <a name="azure-iot-service-sdks"></a>SDK de servicios IoT de Azure

Hay _SDK de servicios_ disponibles para varios idiomas que permiten crear [aplicaciones de back-end](#back-end-app) que interactúan con un centro de IoT. Los tutoriales de IoT Hub muestran cómo se utilizan estos SDK de servicios. Puede encontrar el código fuente y más información sobre los SDK de servicios en este [repositorio](https://github.com/Azure/azure-iot-sdks) de GitHub.

## <a name="azure-iot-tools"></a>Herramientas de Azure IoT

Las [herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) son una extensión de Visual Studio Code multiplataforma y de código abierto que lo ayuda a administrar Azure IoT Hub y dispositivos en VS Code. Con las herramientas de Azure IoT, los desarrolladores de IoT podrían desarrollar proyectos de IoT en VS Code con facilidad.

## <a name="azure-portal"></a>Azure portal

[Microsoft Azure Portal](https://portal.azure.com) es una ubicación central desde donde se pueden aprovisionar y administrar los recursos de Azure. Organiza su contenido mediante _hojas_.

## <a name="azure-powershell"></a>Azure PowerShell

[Azure PowerShell](/powershell/azure/) es un conjunto de cmdlets que puede usar para administrar Azure con Windows PowerShell. Puede utilizar los cmdlets para crear, probar, implementar y administrar soluciones y servicios ofrecidos a través de la plataforma de Azure.

## <a name="azure-resource-manager"></a>Azure Resource Manager

[Azure Resource Manager](../azure-resource-manager/management/overview.md) permite trabajar con los recursos de la solución como grupo. Los recursos de la solución se pueden implementar, actualizar o eliminar en una sola operación coordinada.

## <a name="azure-service-bus"></a>Azure Service Bus

[Service Bus](../service-bus/index.md) proporciona comunicación habilitada para la nube, con mensajería empresarial y comunicación retransmitida que ayuda a conectar las soluciones locales con la nube. Algunos tutoriales de IoT Hub usan las [colas](../service-bus-messaging/service-bus-messaging-overview.md) de Service Bus.

## <a name="azure-storage"></a>Azure Storage

[Azure Storage](../storage/common/storage-introduction.md) es una solución de almacenamiento en la nube. Incluye el servicio Blob Storage que se puede usar para almacenar datos de objetos no estructurados. Algunos tutoriales de IoT Hub utilizan Blob Storage.

## <a name="back-end-app"></a>Aplicación de back-end

En el contexto de [IoT Hub](#iot-hub), una aplicación de back-end es una aplicación que se conecta a uno de los puntos de conexión orientados al servicio en un centro de IoT. Por ejemplo, una aplicación de back-end podría recuperar mensajes [del dispositivo a la nube](#device-to-cloud) o administrar el [registro de identidades](#identity-registry). Normalmente, una aplicación de back-end se ejecuta en la nube pero, en muchos de los tutoriales, las aplicaciones de back-end son aplicaciones de consola que se ejecutan en el equipo de desarrollo local.

## <a name="built-in-endpoints"></a>Puntos de conexión integrados

Todas las instancias de IoT Hub incluyen un [punto de conexión](iot-hub-devguide-endpoints.md) integrado que es compatible con Event Hubs. Puede utilizar cualquier mecanismo que funcione con Event Hubs para leer mensajes del dispositivo a la nube desde este punto de conexión.

## <a name="cloud-gateway"></a>Puerta de enlace en la nube

Una puerta de enlace en la nube habilita la conectividad para dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub). Una puerta de enlace en la nube se hospeda en la nube a diferencia de una [puerta de enlace de campo](#field-gateway) que se ejecuta localmente en los dispositivos. Un caso de uso típico de una puerta de enlace en la nube es la implementación de la traducción de protocolos para los dispositivos.

## <a name="cloud-to-device"></a>De la nube al dispositivo

Hace referencia a los mensajes enviados desde un centro de IoT a un dispositivo conectado. A menudo, estos mensajes son comandos que indican al dispositivo que realice una acción. Para más información, vea [Enviar y recibir mensajes con IoT Hub](iot-hub-devguide-messaging.md).

## <a name="configuration"></a>Configuración

En el contexto de la [configuración automática de dispositivos](iot-hub-auto-device-config.md), una configuración en IoT Hub define la configuración deseada de un conjunto de dispositivos gemelos y ofrece una serie de métricas para informar del estado y el progreso.

## <a name="connection-string"></a>Cadena de conexión

Las cadenas de conexión se usan en el código de aplicación a fin de encapsular la información necesaria para conectarse a un punto de conexión. Normalmente, una cadena de conexión incluye la dirección de la información del punto de conexión y de seguridad, pero los formatos de la cadena de conexión varían en función de los servicios. Hay dos tipos de cadena de conexión asociadas con el servicio de IoT Hub:

- Las *cadenas de conexión de dispositivo* permiten que los dispositivos se conecten a puntos de conexión orientados a dispositivos de un centro de IoT Hub.

- Las *cadenas de conexión de dispositivo* permiten que las aplicaciones de back-end se conecten a puntos de conexión orientados a servicios de un centro de IoT Hub.

## <a name="custom-endpoints"></a>Puntos de conexión personalizados

Puede crear [puntos de conexión](iot-hub-devguide-endpoints.md) personalizados en una instancia de IoT Hub para enviar mensajes que distribuye una [regla de enrutamiento](#routing-rules). Los puntos de conexión personalizados se conectan directamente con una instancia de Event Hubs, una cola de Service Bus o un tema de Service Bus.

## <a name="custom-gateway"></a>Puerta de enlace personalizada

Una puerta de enlace habilita la conectividad para dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub). Puede usar Azure IoT Edge con el fin de crear puertas de enlace personalizadas que implementan lógica personalizada para controlar mensajes, conversiones de protocolo personalizadas y otros procesos del nodo perimetral.

## <a name="data-point-message"></a>Mensaje de punto de datos

Un mensaje de punto de datos es un mensaje [del dispositivo a la nube](#device-to-cloud) que contiene datos de [telemetría](#telemetry), como la velocidad del viento o la temperatura.

## <a name="desired-configuration"></a>Configuración deseada

En el contexto de un [dispositivo gemelo](iot-hub-devguide-device-twins.md), la configuración deseada hace referencia al conjunto completo de propiedades y metadatos en el dispositivo gemelo que se debe sincronizar con el dispositivo.

## <a name="desired-properties"></a>Propiedades deseadas

En el contexto de un [dispositivo gemelo](iot-hub-devguide-device-twins.md), las propiedades deseadas son una subsección del dispositivo gemelo que se usa con las [propiedades notificadas](#reported-properties) para sincronizar la configuración o la condición del dispositivo. Solo una [aplicación de back-end](#back-end-app) puede establecer las propiedades deseadas y solo la [aplicación para dispositivo](#device-app) las puede observar.

## <a name="device-to-cloud"></a>Del dispositivo a la nube

Hace referencia a los mensajes enviados desde un dispositivo conectado a [IoT Hub](#iot-hub). Estos mensajes pueden ser [puntos de datos](#data-point-message) o [mensajes](#interactive-message) interactivos. Para más información, vea [Enviar y recibir mensajes con IoT Hub](iot-hub-devguide-messaging.md).

## <a name="device"></a>Dispositivo

En el contexto de IoT, un dispositivo suele ser un dispositivo de computación independiente a pequeña escala que puede recopilar datos o controlar otros dispositivos. Por ejemplo, un dispositivo podría ser un dispositivo de supervisión ambiental o un controlador para los sistemas de riego y ventilación en un invernadero. El [catálogo de dispositivos](https://catalog.azureiotsolutions.com/) proporciona una lista de dispositivos de hardware certificados que funcionan con [IoT Hub](#iot-hub).

## <a name="device-app"></a>Aplicación para dispositivo

Una aplicación para dispositivo se ejecuta en un [dispositivo](#device) y esta aplicación administra la comunicación con el [centro de IoT](#iot-hub). Por lo general, se suele utilizar uno de los [SDK de dispositivos IoT de Azure](#azure-iot-device-sdks) al implementar una aplicación para dispositivo. En muchos de los tutoriales de IoT, se usa un [dispositivo simulado](#simulated-device) por motivos de comodidad.

## <a name="device-condition"></a>Condición de dispositivo

Hace referencia a información de estado del dispositivo, como el método de conectividad actualmente en uso, según notifica una [aplicación para dispositivo](#device-app). Las [aplicaciones para dispositivo](#device-app) también pueden notificar sus funcionalidades. Se puede consultar la información de condición y funcionalidad mediante dispositivos gemelos.

## <a name="device-data"></a>Datos del dispositivo

Los datos del dispositivo hacen referencia a los datos por dispositivo almacenados en el [registro de identidades](#identity-registry) del centro de IoT. Es posible importar y exportar estos datos.

## <a name="device-explorer"></a>Explorador de dispositivos

El explorador de dispositivos se ha reemplazado por [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer), que se usa para ver la telemetría que el dispositivo envía, trabajar con las propiedades del dispositivo y llamar a comandos. También puede usar el explorador para interactuar con los dispositivos y probarlos, y para administrar dispositivos Plug and Play.

## <a name="device-identity"></a>Identidad del dispositivo

La identidad del dispositivo es el identificador único asignado a cada dispositivo registrado del [registro de identidad](#identity-registry).

## <a name="device-management"></a>Administración de dispositivos

La administración de dispositivos abarca el ciclo de vida completo asociado a la administración de dispositivos de la solución de IoT, incluidos la planeación, el aprovisionamiento, la configuración, la supervisión y la retirada.

## <a name="device-management-patterns"></a>Patrones de administración de dispositivos

[IoT Hub](#iot-hub) permite patrones comunes de administración de dispositivos incluidos el reinicio, el restablecimiento de fábrica y las actualizaciones de firmware de los dispositivos.

## <a name="device-rest-api"></a>API REST de dispositivos

Puede usar la [API REST de dispositivos](https://docs.microsoft.com/rest/api/iothub/device) desde un dispositivo para enviar mensajes del dispositivo a la nube a IoT Hub y recibir mensajes [de la nube al dispositivo](#cloud-to-device) desde una instancia de IoT Hub. Por lo general, debería usar uno de los [SDK de dispositivos](#azure-iot-device-sdks) de nivel superior, tal como se muestra en los tutoriales de IoT Hub.

## <a name="device-provisioning"></a>Aprovisionamiento de dispositivos

El aprovisionamiento de dispositivos es el proceso de agregar los [datos del dispositivo](#device-data) iniciales a los almacenes de la solución. Para habilitar un nuevo dispositivo y que se conecte al centro, debe agregar un nuevo identificador de dispositivo y claves al [registro de identidades](#identity-registry) del centro de IoT. Como parte del proceso de aprovisionamiento, es posible que tenga que inicializar datos específicos del dispositivo en otros almacenes de la solución.

## <a name="device-twin"></a>Dispositivo gemelo

Un [dispositivo gemelo](iot-hub-devguide-device-twins.md) es un documento JSON que almacena información sobre el estado de los dispositivos, como metadatos, configuraciones y condiciones. [IoT Hub](#iot-hub) conserva un dispositivo gemelo por cada dispositivo que se aprovisiona en el centro de IoT. Los dispositivos gemelos permiten sincronizar las [condiciones de dispositivo](#device-condition) y las configuraciones entre el dispositivo y el back-end de soluciones. Puede consultar los dispositivos gemelos para buscar dispositivos específicos y consultar el estado de las operaciones de larga duración.

## <a name="direct-method"></a>Método directo

Un [método directo](iot-hub-devguide-direct-methods.md) es una manera de desencadenar un método para ejecutarse en un dispositivo mediante la invocación de una API en su IoT Hub.

## <a name="endpoint"></a>Punto de conexión

Un centro de IoT expone varios [puntos de conexión](iot-hub-devguide-endpoints.md) que permiten a las aplicaciones conectarse con el centro de IoT. Hay puntos de conexión accesibles desde los dispositivos que permiten a los dispositivos realizar operaciones tales como envío de mensajes [del dispositivo a la nube](#device-to-cloud) y recepción de mensajes [de la nube al dispositivo](#cloud-to-device). Hay puntos de conexión de administración accesibles desde el servicio que permiten a las [aplicaciones de back-end](#back-end-app) realizar operaciones como la administración de [identidades de dispositivo](#device-identity) y la administración de dispositivos gemelos. Hay [puntos de conexión integrados](#built-in-endpoints) orientados al servicio para leer mensajes del dispositivo a la nube. Puede crear [puntos de conexión](#custom-endpoints) personalizados para recibir mensajes del dispositivo a la nube que distribuye una [regla de enrutamiento](#routing-rules).

## <a name="event-hubs-service"></a>Servicio Event Hubs

[Event Hubs](../event-hubs/event-hubs-what-is-event-hubs.md) es un servicio de entrada de datos altamente escalable que puede ingerir millones de eventos por segundo. El servicio permite procesar y analizar la gran cantidad de datos que generan las aplicaciones y los dispositivos conectados. Para ver una comparación con el servicio IoT Hub, consulte [Comparación de Azure IoT Hub y Azure Event Hubs](iot-hub-compare-event-hubs.md).

## <a name="event-hub-compatible-endpoint"></a>Punto de conexión compatible con Event Hub

Para leer mensajes [del dispositivo a la nube](#device-to-cloud) enviados a IoT Hub, puede conectarse a un punto de conexión en el centro y utilizar cualquier método compatible con Event Hub para leer esos mensajes. Los métodos compatibles con Event Hub incluyen el uso de los [SDK de Event Hubs](../event-hubs/event-hubs-programming-guide.md) y [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md).

## <a name="field-gateway"></a>Puerta de enlace de campo

Una puerta de enlace de campo habilita la conectividad de dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub) y se suele implementar localmente con los dispositivos. Para más información, consulte [¿Qué es Azure IoT Hub?](about-iot-hub.md)

## <a name="free-account"></a>Cuenta gratuita

Puede crear una [cuenta de Azure libre](https://azure.microsoft.com/pricing/free-trial/) para completar los tutoriales de IoT Hub y experimentar con el servicio IoT Hub (y otros servicios de Azure).

## <a name="gateway"></a>Puerta de enlace

Una puerta de enlace habilita la conectividad para dispositivos que no se pueden conectar directamente a [IoT Hub](#iot-hub). Consulte también [Puerta de enlace de campo](#field-gateway), [Puerta de enlace en la nube](#cloud-gateway) y [Puerta de enlace personalizada](#custom-gateway).

## <a name="identity-registry"></a>Registro de identidad

El [registro de identidades](iot-hub-devguide-identity-registry.md) es el componente integrado de un centro de IoT que almacena información acerca de los dispositivos individuales permitidos para conectarse a un centro de IoT.

## <a name="interactive-message"></a>Mensaje interactivo

Un mensaje interactivo es un mensaje [de nube a dispositivo](#cloud-to-device) que desencadena una acción inmediata en el back-end de solución. Por ejemplo, un dispositivo puede enviar una alarma sobre un error que se debe registrar automáticamente en un sistema CRM.

[!INCLUDE [azure-iot-hub-edge-glossary-includes](../../includes/azure-iot-hub-edge-glossary-includes.md)]

## <a name="iot-hub"></a>IoT Hub

IoT Hub es un servicio de Azure totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y un back-end de soluciones. Para más información, consulte [¿Qué es Azure IoT Hub?](about-iot-hub.md) Con su [suscripción de Azure](#subscription), puede crear centros de IoT para controlar las cargas de trabajo de mensajería de IoT.

## <a name="iot-hub-metrics"></a>Métricas de IoT Hub

Las [métricas de IoT Hub](iot-hub-metrics.md) ofrecen datos sobre el estado de los centros de IoT en la [suscripción de Azure](#subscription). Las métricas de IoT Hub permiten evaluar el estado general del servicio y de los dispositivos conectados a él. Las métricas de IoT Hub pueden ayudar a ver lo que está ocurriendo con el centro de IoT y a investigar la causa raíz de los problemas sin necesidad de ponerse en contacto con el soporte técnico de Azure.

## <a name="iot-hub-query-language"></a>Lenguaje de consulta de IoT Hub

El [lenguaje de consulta de IoT Hub](iot-hub-devguide-query-language.md) es un lenguaje de tipo SQL que permite consultar el [trabajo](#job) y los dispositivos gemelos.

## <a name="iot-hub-resource-rest-api"></a>API REST de recursos de IoT Hub

Puede usar la [API REST de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource) para administrar los centros de IoT Hub en la [suscripción de Azure](#subscription); por ejemplo, puede realizar operaciones como crear, actualizar y eliminar centros.

## <a name="iot-solution-accelerators"></a>Aceleradores de soluciones de IoT

Los aceleradores de soluciones de Azure IoT agrupan varios servicios de Azure en soluciones. Estas soluciones le permiten iniciar rápidamente implementaciones de un extremo a otro de escenarios comunes de IoT. Para más información, vea [¿Cuáles son los aceleradores de soluciones de Azure IoT?](../iot-accelerators/about-iot-accelerators.md)

## <a name="the-iot-extension-for-azure-cli"></a>La extensión de IoT para la CLI de Azure 

[La extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension) es una herramienta de la línea de comandos multiplataforma. La herramienta permite administrar los dispositivos en el [registro de identidades](#identity-registry), enviar y recibir mensajes y archivos de los dispositivos, así como supervisar las operaciones del centro de IoT.

## <a name="job"></a>Trabajo

El back-end de solución puede utilizar [trabajos](iot-hub-devguide-jobs.md) para programar y realizar el seguimiento de actividades en un conjunto de dispositivos registrados con IoT Hub. Las actividades incluyen actualizar las [propiedades deseadas](#desired-properties) del dispositivo gemelo, actualizar las [etiquetas](#tags) del dispositivo gemelo e invocar [métodos directos](#direct-method). [IoT Hub](#iot-hub) también usa trabajos para [importar y exportar](iot-hub-devguide-identity-registry.md#import-and-export-device-identities) elementos desde el [registro de identidades](#identity-registry).

## <a name="modules"></a>Módulos

En el lado del dispositivo, los SDK de dispositivo de IoT Hub le permiten crear [módulos](iot-hub-devguide-module-twins.md), donde cada uno abre una conexión independiente a IoT Hub. Esta funcionalidad le permite usar espacios de nombres distintos para distintos componentes del dispositivo.

La identidad del módulo y el módulo gemelo proporcionan las mismas funcionalidades que la [identidad del dispositivo](#device-identity) y el [dispositivo gemelo](#device-twin), pero con una mayor granularidad. Esto permite a los dispositivos compatibles, como dispositivos basados en el sistema operativo o dispositivos de firmware que administran varios componentes, aislar la configuración y las condiciones de cada uno de esos componentes.

## <a name="module-identity"></a>Identidad de módulo

La identidad de módulo es el identificador único asignado a cada módulo que pertenece a un dispositivo. La identidad de módulo también se registra en el [registro de identidades](#identity-registry).

## <a name="module-twin"></a>Módulo gemelo

De forma similar al dispositivo gemelo, un módulo gemelo es un documento JSON que almacena información sobre el estado de los módulos, como metadatos, configuraciones y condiciones. IoT Hub persiste un módulo gemelo para cada identidad de módulo aprovisionado en una identidad del dispositivo del centro de IoT. Los módulos gemelos permiten sincronizar las condiciones de módulo y las configuraciones entre el módulo y el back-end de soluciones. Puede consultar los módulos gemelos para buscar módulos específicos y consultar el estado de las operaciones de larga duración.

## <a name="mqtt"></a>MQTT

[MQTT](https://mqtt.org/) es uno de los protocolos de mensajería que admite [IoT Hub](#iot-hub) para la comunicación con dispositivos. Para obtener más información acerca de los protocolos de mensajería que admite IoT Hub, consulte [Enviar y recibir mensajes con IoT Hub](iot-hub-devguide-messaging.md).

## <a name="operations-monitoring"></a>Supervisión de operaciones

La [supervisión de operaciones](iot-hub-operations-monitoring.md) de IoT Hub permite supervisar el estado de las operaciones del centro de IoT en tiempo real. [IoT Hub](#iot-hub) realiza el seguimiento de eventos a través de varias categorías de operaciones. Se puede optar por que los eventos de una o varias categorías se envíen a un punto de conexión del IoT Hub para su procesamiento. Los usuarios pueden supervisar los datos en busca de errores o configurar un procesamiento más complejo basado en patrones de datos.

## <a name="physical-device"></a>Dispositivo físico

Un dispositivo físico es un dispositivo real, como un Raspberry Pi que se conecta a un centro de IoT. Para mayor comodidad, muchos de los tutoriales de IoT Hub usan [dispositivos simulados](#simulated-device) para que se puedan ejecutar los ejemplos en el equipo local.

## <a name="primary-and-secondary-keys"></a>Claves principales y secundarias

Al conectarse a un punto de conexión accesible desde el dispositivo o el servicio en un centro de IoT, la [cadena de conexión](#connection-string) incluye la clave para el acceso. Al agregar un dispositivo al [registro de identidad](#identity-registry) o una [directiva de acceso compartido](#shared-access-policy) al centro, el servicio genera una clave principal y una secundaria. Tener dos claves permite pasar de una clave a otra cuando se actualiza una clave, sin perder acceso al centro de IoT.

## <a name="protocol-gateway"></a>Puerta de enlace de protocolo

Una puerta de enlace de protocolo se implementa normalmente en la nube y proporciona servicios de traducción de protocolo para dispositivos que se conectan a [IoT Hub](#iot-hub). Para más información, consulte [¿Qué es Azure IoT Hub?](about-iot-hub.md)

## <a name="quotas-and-throttling"></a>Cuotas y limitación

Hay distintas [cuotas](iot-hub-devguide-quotas-throttling.md) que se aplican al uso de [IoT Hub](#iot-hub), muchas de ellas varían según el nivel del centro de IoT. [IoT Hub](#iot-hub) también aplica [límites](iot-hub-devguide-quotas-throttling.md) al uso del servicio en tiempo de ejecución.

## <a name="reported-configuration"></a>Configuración notificada

En el contexto de un [dispositivo gemelo](iot-hub-devguide-device-twins.md), la configuración notificada hace referencia al conjunto completo de propiedades y metadatos en el dispositivo gemelo que deben notificarse al back-end de la solución.

## <a name="reported-properties"></a>Propiedades notificadas

En el contexto de un [dispositivo gemelo](iot-hub-devguide-device-twins.md), las propiedades notificadas son una subsección del dispositivo gemelo que se usa con las [propiedades deseadas](#desired-properties) para sincronizar la configuración o la condición del dispositivo. Solo la [aplicación para dispositivo](#device-app) puede establecer las propiedades notificadas y solo la [aplicación de back-end](#back-end-app) las puede leer y consultar.

## <a name="resource-group"></a>Resource group

[Azure Resource Manager](#azure-resource-manager) usa grupos de recursos para agrupar recursos relacionados. Se puede usar un grupo de recursos para realizar operaciones en todos los recursos del grupo al mismo tiempo.

## <a name="retry-policy"></a>Directiva de reintentos

Una directiva de reintentos se utiliza para controlar los [errores transitorios](/azure/architecture/best-practices/transient-faults) al conectarse a un servicio en la nube.

## <a name="routing-rules"></a>Reglas de enrutamiento

Configure [reglas de enrutamiento](iot-hub-devguide-messages-read-custom.md) en su instancia de IoT Hub para enrutar mensajes del dispositivo a la nube a un [punto de conexión integrado](#built-in-endpoints) o a [puntos de conexión personalizados](#custom-endpoints) para que los procese el back-end de su solución.

## <a name="sasl-plain"></a>SASL PLAIN

SASL PLAIN es un protocolo que el protocolo AMQP utiliza para transferir tokens de seguridad.

## <a name="service-rest-api"></a>API de REST de servicio

Puede usar la [API REST de servicio](https://docs.microsoft.com/rest/api/iothub/service/configuration) en el back-end de la solución para administrar sus dispositivos. La API le permite recuperar y actualizar las propiedades del [dispositivo gemelo](#device-twin), invocar [métodos directos](#direct-method) y programar [trabajos](#job). Por lo general, debería usar uno de los [SDK de servicios](#azure-iot-service-sdks) de nivel superior, tal como se muestra en los tutoriales de IoT Hub.

## <a name="shared-access-signature"></a>Firma de acceso compartido

Las firmas de acceso compartido (SAS) son un mecanismo de autenticación basado en valores hash seguros SHA-256 o en URI. La autenticación de SAS tiene dos componentes: una _directiva de acceso compartido_ y una _firma de acceso compartido_ (a menudo denominada token). Un dispositivo utiliza SAS para autenticarse con un centro de IoT. Las [aplicaciones de back-end](#back-end-app) también utilizan SAS para autenticarse con los puntos de conexión accesibles desde servicios en un centro de IoT. Normalmente, se incluye el token de SAS en la [cadena de conexión](#connection-string) que una aplicación usa para establecer una conexión con un centro de IoT.

## <a name="shared-access-policy"></a>Directiva de acceso compartido

Una directiva de acceso compartido define los permisos concedidos a cualquier persona que tenga una [clave principal o secundaria](#primary-and-secondary-keys) válida asociada a esa directiva. Se pueden administrar las directivas de acceso compartido y las claves para el centro en el [portal](#azure-portal).

## <a name="simulated-device"></a>Dispositivo simulado

Para mayor comodidad, muchos de los tutoriales de IoT Hub usan dispositivos simulados para que se puedan ejecutar los ejemplos en el equipo local. En cambio, un [dispositivo físico](#physical-device) es un dispositivo real, como un Raspberry Pi que se conecta a un centro de IoT.

## <a name="solution"></a>Solución
Una _solución_ puede hacer referencia a una solución de Visual Studio que incluye uno o más proyectos. Una _solución_ también puede hacer referencia a una solución de IoT que incluye elementos como dispositivos, [aplicaciones para dispositivo](#device-app), un centro de IoT, otros servicios de Azure y [aplicaciones de back-end](#back-end-app).

## <a name="subscription"></a>Suscripción

Una suscripción de Azure es donde se realiza la facturación. Cada recurso de Azure que se crea o servicio de Azure que se utiliza está asociado a una única suscripción. Muchas cuotas también se aplican en el nivel de suscripción.

## <a name="system-properties"></a>Propiedades del sistema

En el contexto de un [dispositivo gemelo](iot-hub-devguide-device-twins.md), las propiedades del sistema son de solo lectura e incluyen información sobre el uso de dispositivos, como el último estado de conexión y tiempo de actividad.

## <a name="tags"></a>Etiquetas

En el contexto de un [dispositivo gemelo](iot-hub-devguide-device-twins.md), las etiquetas son metadatos de dispositivos almacenados y recuperados por el back-end de solución en forma de documento JSON. Las etiquetas no son visibles para las aplicaciones en un dispositivo.

## <a name="telemetry"></a>Telemetría

Los dispositivos recopilan datos de telemetría, como la velocidad del viento o la temperatura, y usan mensajes de punto de datos para enviar la telemetría a un centro de IoT.

## <a name="token-service"></a>Servicio de token

Se puede utilizar un servicio de token a fin de implementar un mecanismo de autenticación para los dispositivos. Usa una [directiva de acceso compartido](#shared-access-policy) de IoT Hub con permisos **DeviceConnect** para crear *tokens centrados en el dispositivo*. Estos tokens permiten que un dispositivo se conecte al centro de IoT. Un dispositivo utiliza un mecanismo de autenticación personalizado para autenticarse con el servicio de token. Si el dispositivo se autentica correctamente, el servicio de token emite un token de SAS para que el dispositivo pueda tener acceso al centro de IoT.

## <a name="twin-queries"></a>Consultas gemelas

Las [consultas de dispositivos y módulos gemelos](iot-hub-devguide-query-language.md) usan el lenguaje de consulta de IoT Hub de tipo SQL para recuperar información de los dispositivos o módulos gemelos. Puede usar el mismo lenguaje de consulta de IoT Hub para recuperar información sobre un [trabajo](#job) que se ejecuta en IoT Hub.

## <a name="twin-synchronization"></a>Sincronización de gemelos

La sincronización de gemelos usa las [propiedades deseadas](#desired-properties) en los dispositivos o módulos gemelos para configurar los dispositivos o módulos y recuperar las [propiedades notificadas](#reported-properties) de los dispositivos para almacenarlas en el gemelo.

## <a name="x509-client-certificate"></a>Certificado de cliente X.509

Un dispositivo puede usar un certificado X.509 para autenticarse con [IoT Hub](#iot-hub). Un certificado X.509 es una alternativa al uso de un [token de SAS](#shared-access-signature).
