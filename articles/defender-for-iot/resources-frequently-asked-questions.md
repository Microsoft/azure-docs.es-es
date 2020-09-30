---
title: Preguntas más frecuentes sobre Defender para IoT
description: Encuentre respuestas a las preguntas más frecuentes sobre las características y servicios de Azure Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 97dd4e13754175e9c81ae308b86faae811e4d554
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932349"
---
# <a name="azure-defender-for-iot-frequently-asked-questions"></a>Preguntas más frecuentes sobre Azure Defender para IoT

En este artículo se proporciona una lista de las preguntas más frecuentes y las respuestas sobre Defender para IoT.

## <a name="does-azure-provide-support-for-iot-security"></a>¿Ofrece Azure compatibilidad para seguridad de IoT?

Azure proporciona una vista integrada para supervisar y administrar la seguridad de IoT como parte de la solución general de seguridad a través de Azure Security Center. Si es desarrollador de aplicaciones, puede usar la vista de IoT Hub para administrar la seguridad de su aplicación para IoT.

## <a name="what-is-azures-unique-value-proposition-for-iot-security"></a>¿Cuál es la propuesta de valor propia de Azure para la seguridad de IoT?

Defender para IoT permite a las empresas ampliar su vista existente de la seguridad cibernética a toda la solución de IoT. Azure proporciona una vista integral de su solución empresarial, lo que le permite realizar acciones relacionadas con la empresa y tomar decisiones según su posición de seguridad empresarial y los datos recopilados. La seguridad combinada mediante Azure IoT, Azure IoT Edge y Azure Security Center le permite crear la solución que quiere con la seguridad que necesita.

## <a name="who-is-defender-for-iot-made-for"></a>¿Para quién es Defender para IoT?

Defender para IoT está integrado en la seguridad de Azure IoT Hub y permite la administración de las operaciones diarias de seguridad para las soluciones empresariales. Defender para IoT también está integrado en las funcionalidades de Azure Security Center y proporciona una vista integrada para supervisar y administrar la seguridad de IoT como parte de la solución general de seguridad.

## <a name="how-does-defender-for-iot-compare-to-the-competition"></a>¿Cómo es Defender para IoT en relación con la competencia?

Mientras que otras soluciones proporcionan un conjunto de funcionalidades que permiten a los clientes crear sus propias soluciones, Defender para IoT proporciona una solución exclusiva e integral de seguridad de IoT que ofrece una amplia visión de la seguridad de todos los recursos de Azure relacionados. Azure permite una implementación rápida y la integración completa en los módulos gemelos de IoT Hub para una fácil integración con las herramientas existentes de administración de dispositivos.

## <a name="do-i-have-to-be-an-azure-security-center-customer-to-use-this-service"></a>¿Tengo que ser cliente de Azure Security Center para usar el servicio?

No, pero se recomienda. Sin Azure Security Center, Defender para IoT recibe datos limitados de los recursos conectados y brinda un análisis limitado de la superficie expuesta a ataques potenciales, de las amenazas y de los posibles ataques.

## <a name="do-i-have-to-be-an-azure-iot-customer"></a>¿Tengo que ser cliente de Azure IoT?

Sí. Defender para IoT se basa en la infraestructura y la conectividad de Azure IoT.

## <a name="do-i-have-to-install-an-agent"></a>¿Es necesario instalar a un agente?

La instalación de un agente en los dispositivos de IoT no es obligatoria para habilitar Defender para IoT. Puede elegir entre las tres opciones siguientes, con lo que obtiene distintos niveles de funcionalidad de supervisión y administración de la seguridad según la selección:

1. Instale el agente de seguridad de Defender para IoT con o sin modificaciones. Esta opción proporciona el máximo nivel de información de seguridad mejorada relativa al comportamiento y el acceso al dispositivo.

1. Cree su propio agente e implemente el esquema de mensajes de seguridad de Defender para IoT. Esta opción permite usar las herramientas de análisis de Defender para IoT, además del agente de seguridad de los dispositivos.

1. Sin instalación de ningún agente de seguridad en los dispositivos de IoT. Esta opción permite la supervisión de la comunicación de IoT Hub, con funcionalidades reducidas de supervisión y administración de seguridad.

## <a name="what-does-the-defender-for-iot-agent-do"></a>¿Qué hace el agente de Defender para IoT?

El agente de Defender para IoT proporciona cobertura de amenazas a nivel del dispositivo para la configuración, el comportamiento, el acceso (mediante el examen de la configuración), los procesos y la conectividad de los dispositivos. El agente de seguridad de Defender para IoT no examina los datos ni actividades relacionados con el negocio.

## <a name="where-can-i-get-the-defender-for-iot-security-agent"></a>¿Dónde puedo encontrar el agente de seguridad de Defender para IoT?

El agente de seguridad de Defender para IoT es de código abierto y está disponible en GitHub para Windows y Linux en versiones de 32 y 64 bits: https://github.com/Azure/Azure-IoT-Security

## <a name="where-does-the-defender-for-iot-agent-get-installed"></a>¿Donde se instala el agente de Defender para IoT?

En GitHub, encontrará información detallada sobre la instalación y la implementación del agente: https://github.com/Azure/Azure-IoT-Security.

## <a name="what-are-the-dependencies-and-prerequisites-of-the-agent"></a>¿Cuáles son las dependencias y los requisitos previos del agente?

Defender para IoT admite una amplia variedad de plataformas. Consulte [Plataformas compatibles](how-to-deploy-agent.md) para verificar la compatibilidad para sus dispositivos en concreto.

## <a name="which-data-is-collected-by-the-agent"></a>¿Qué datos recopila el agente?

El agente recopila datos de conectividad, acceso, configuración del firewall, lista de procesos y línea base del SO.

## <a name="how-much-data-will-the-agent-generate"></a>¿Cuántos datos generará el agente?

La generación de datos del agente depende del dispositivo, la aplicación, el tipo de conectividad y la configuración del agente del cliente. Debido a la gran variabilidad entre dispositivos y soluciones de IoT, se recomienda implementar primero el agente en un entorno de laboratorio o prueba para observar, aprender y establecer la configuración específica que se adapte a sus necesidades, a la vez que se mide la cantidad de datos generados. Después de iniciar el servicio, el agente de Defender para IoT proporciona recomendaciones operativas para optimizar el rendimiento del agente, lo que le ayudará en el proceso de configuración y personalización.

## <a name="how-can-i-control-my-billing"></a>¿Cómo puedo controlar la facturación?

Defender para IoT proporciona exámenes de agente configurables, búferes de datos y la capacidad de crear alertas personalizadas que aumentan o reducen la cantidad de datos que genera el agente.

## <a name="do-agent-messages-use-up-quota-from-iot-hub"></a>¿Se cuentan los mensajes del agente en la cuota de IoT Hub?

Sí. Los datos transmitidos por el agente se contabilizan en su cuota de IoT Hub.

## <a name="what-next-ive-installed-an-agent-and-dont-see-any-activities-or-logs"></a>¿Qué más? He instalado un agente y no veo ninguna actividad ni registros...

1. Compruebe que el [tipo de agente se ajusta a la plataforma del SO designada de su dispositivo](how-to-deploy-agent.md).

1. Confirme que el [agente se esté ejecutando en el dispositivo](how-to-agent-configuration.md).

1. Compruebe que el [servicio se haya habilitado correctamente](quickstart-onboard-iot-hub.md) en **Seguridad** en IoT Hub.

1. Compruebe que el dispositivo esté [configurado en IoT Hub con el módulo de Defender para IoT](quickstart-create-security-twin.md).

Si las actividades o los registros aún no están disponibles, póngase en contacto con su asociado de Defender para IoT para obtener ayuda adicional.

## <a name="what-happens-when-the-internet-connection-stops-working"></a>¿Qué ocurre cuando la conexión a Internet deja de funcionar?

El agente sigue ejecutándose y almacena los datos, siempre y cuando el dispositivo esté en ejecución. Los datos se almacenan en la caché de mensajes de seguridad según la configuración de tamaño. Cuando el dispositivo vuelve a tener conectividad, se reanuda el envío de los mensajes de seguridad.

## <a name="if-the-device-is-restarted-will-the-security-agent-self-recover"></a>Si el dispositivo se reinicia, ¿se recupera automáticamente el agente de seguridad?

El agente de seguridad está diseñado para volver a ejecutarse automáticamente con cada reinicio del dispositivo.

## <a name="can-the-agent-affect-the-performance-of-the-device-or-other-installed-software"></a>¿Puede el agente afectar al rendimiento del dispositivo o de otro software instalado?

El agente consume recursos de la máquina como cualquier otra aplicación o proceso y no debería interrumpir la actividad normal del dispositivo. El consumo de recursos en el dispositivo donde se ejecuta el agente está condicionado por la instalación y la configuración. Se recomienda probar la configuración de agente en un entorno independiente, junto con la interoperabilidad con las otras aplicaciones y funcionalidades de IoT, antes de intentar implementar en un entorno de producción.

## <a name="im-making-some-maintenance-on-the-device-can-i-turn-off-the-agent"></a>Estoy realizando mantenimiento en el dispositivo. ¿Puedo desactivar el agente?

El agente no puede desactivarse.

## <a name="is-there-a-way-to-test-if-the-agent-is-working-correctly"></a>¿Hay alguna manera de probar si el agente funciona correctamente?

Si el agente deja de comunicarse o no envía mensajes de seguridad, se genera una alerta **Device is silent** (El dispositivo está silencioso).

## <a name="can-i-create-my-own-alerts"></a>¿Puedo crear mis propias alertas?

Sí. Puede establecer una alerta personalizada ante un conjunto predeterminado de comportamientos, como direcciones IP y puertos abiertos. Consulte [Creación de alertas personalizadas](quickstart-create-custom-alerts.md) para más información sobre las alertas personalizadas y cómo crearlas.

## <a name="where-can-i-see-logs-can-i-customize-logs"></a>¿Dónde se pueden ver los registros? ¿Se pueden personalizan los registros?

- Consulte las alertas y recomendaciones desde el área de trabajo de Log Analytics conectada. Configure el tamaño y la duración de almacenamiento en el área de trabajo.

- Los datos sin procesar del agente de seguridad también pueden almacenarse en su cuenta de Log Analytics. Tenga en cuenta el tamaño, la duración, los requisitos de almacenamiento y los costos asociados antes de cambiar la configuración de esta opción.

## <a name="why-should-i-add-defender-for-iot-to-the-module-identity-what-is-it-used-for"></a>¿Por qué debería agregar Defender para IoT a la identidad de módulo? ¿Para qué se usa?

El módulo Defender para IoT se usa para la configuración y administración del agente.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los primeros pasos con Defender para IoT, consulte los siguientes artículos:

- Lea la [información general](overview.md) de Defender para IoT.
- Verifique los [requisitos previos del servicio](service-prerequisites.md)
- Más información sobre cómo [empezar a usar ASC para IoT](getting-started.md)
- Información sobre las [alertas de seguridad de Defender para IoT](concept-security-alerts.md)
