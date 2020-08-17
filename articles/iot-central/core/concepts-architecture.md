---
title: Conceptos de arquitectura en Azure IoT Central | Microsoft Docs
description: En este artículo se presentan conceptos clave relacionados con la arquitectura de Azure IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 11/27/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: 1e4b81f2350795b1244289119d714e99bc06ba6f
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337150"
---
# <a name="azure-iot-central-architecture"></a>Arquitectura de Azure IoT Central



En este artículo se proporciona información general sobre la arquitectura de Microsoft Azure IoT Central.

![Arquitectura de nivel superior](media/concepts-architecture/architecture.png)

## <a name="devices"></a>Dispositivos

Los dispositivos intercambian datos con la aplicación Azure IoT Central. Un dispositivo puede:

- Enviar las mediciones como telemetría.
- Sincronizar la configuración con la aplicación.

En Azure IoT Central, los datos que un dispositivo puede intercambiar con la aplicación se especifican en una plantilla de dispositivo. Para obtener más información acerca de las plantillas de dispositivo, consulte [Metadata management](#metadata-management) (Administración de metadatos).

Para obtener más información sobre cómo se conectan los dispositivos a la aplicación de Azure IoT Central, consulte [Device connectivity](concepts-get-connected.md) (Conectividad de dispositivos).

## <a name="azure-iot-edge-devices"></a>Dispositivos de Azure IoT Edge

Además de los dispositivos creados con el [SDK de IoT de Azure](https://github.com/Azure/azure-iot-sdks), también puede conectar [dispositivos Azure IoT Edge](../../iot-edge/about-iot-edge.md) a una aplicación IoT Central. IoT Edge permite ejecutar la inteligencia de nube y la lógica personalizada directamente en dispositivos IoT administrados por IoT Central. El tiempo de ejecución de IoT Edge permite:

- Instalación y actualización de las cargas de trabajo en el dispositivo.
- Mantenimiento de los estándares de seguridad de IoT Edge en el dispositivo.
- Garantía de que los módulos de IoT Edge están siempre en ejecución.
- Notificación del mantenimiento del módulo a la nube para la supervisión remota.
- Administración de la comunicación entre los dispositivos hoja descendentes y un dispositivo IoT Edge, entre los módulos de un dispositivo IoT Edge y entre un dispositivo IoT Edge y la nube.

![Azure IoT Central con Azure IoT Edge](./media/concepts-architecture/iotedge.png)

IoT Central habilita las funcionalidades siguientes para los dispositivos IoT Edge:

- Plantillas de dispositivos para describir las funcionalidades de un dispositivo IoT Edge, como:
  - Funcionalidad de carga del manifiesto de implementación, que ayuda a administrar un manifiesto para una flota de dispositivos.
  - Módulos que se ejecutan en el dispositivo IoT Edge.
  - Telemetría que envía cada módulo.
  - Propiedades que notifica cada módulo.
  - Comandos a los que responde cada módulo.
  - Relaciones entre el modelo de funcionalidad del dispositivo de puerta de enlace IoT Edge y el modelo de funcionalidad del dispositivo de nivel inferior.
  - Propiedades de la nube que no están almacenadas en el dispositivo IoT Edge.
  - Personalizaciones, paneles y formularios que forman parte de la aplicación IoT Central.

  Para obtener más información, consulte el artículo [Conexión de dispositivos de Azure IoT Edge a una aplicación de Azure IoT Central](./concepts-iot-edge.md).

- La capacidad de aprovisionar dispositivos IoT Edge a escala mediante el servicio de aprovisionamiento de dispositivos Azure IoT.
- Reglas y acciones.
- Análisis y paneles personalizados.
- Exportación de datos continua de telemetría desde dispositivos IoT Edge.

### <a name="iot-edge-device-types"></a>Tipos de dispositivos IoT Edge

IoT Central clasifica los tipos de dispositivo IoT Edge de la manera siguiente:

- Dispositivos hoja. Un dispositivo IoT Edge puede tener dispositivos hoja de nivel inferior, pero estos dispositivos no se aprovisionan en IoT Central.
- Dispositivos de puerta de enlace con dispositivos de nivel inferior. Tanto el dispositivo de puerta de enlace como los dispositivos de nivel inferior se aprovisionan en IoT Central.

![Introducción a IoT Central con IoT Edge](./media/concepts-architecture/gatewayedge.png)

### <a name="iot-edge-patterns"></a>Patrones de IoT Edge

IoT Central admite estos patrones de dispositivos IoT Edge:

#### <a name="iot-edge-as-leaf-device"></a>IoT Edge como dispositivo hoja

![IoT Edge como dispositivo hoja](./media/concepts-architecture/edgeasleafdevice.png)

El dispositivo IoT Edge se aprovisiona en IoT Central y todos los dispositivos de nivel inferior y su telemetría se representan como provenientes del dispositivo IoT Edge. Los dispositivos de nivel inferior conectados al dispositivo IoT Edge no se aprovisionan en IoT Central.

#### <a name="iot-edge-gateway-device-connected-to-downstream-devices-with-identity"></a>Dispositivo de puerta de enlace IoT Edge conectado a dispositivos de nivel inferior con identidad

![IoT Edge con identidad del dispositivo de nivel inferior](./media/concepts-architecture/edgewithdownstreamdeviceidentity.png)

El dispositivo IoT Edge se aprovisiona en IoT Central junto con los dispositivos de nivel inferior conectados al dispositivo IoT Edge. Actualmente, no se admite la compatibilidad en tiempo de ejecución para aprovisionar dispositivos de nivel inferior a través de la puerta de enlace.

#### <a name="iot-edge-gateway-device-connected-to-downstream-devices-with-identity-provided-by-the-iot-edge-gateway"></a>Dispositivo de puerta de enlace de IoT Edge conectado a dispositivos de nivel inferior con la identidad proporcionada por la puerta de enlace IoT Edge

![IoT Edge con dispositivo de nivel inferior sin identidad](./media/concepts-architecture/edgewithoutdownstreamdeviceidentity.png)

El dispositivo IoT Edge se aprovisiona en IoT Central junto con los dispositivos de nivel inferior conectados al dispositivo IoT Edge. La compatibilidad en tiempo de ejecución de la puerta de enlace que proporciona la identidad a los dispositivos de nivel inferior y el aprovisionamiento de dispositivos de nivel inferior no se admite actualmente. Si aporta su propio módulo de traducción de identidades, IoT Central puede admitir este patrón.

## <a name="cloud-gateway"></a>Puerta de enlace en la nube

Azure IoT Central usa Azure IoT Hub como una puerta de enlace en la nube que permite la conectividad del dispositivo. IoT Hub permite lo siguiente:

- Ingesta de datos a escala en la nube.
- Administración de dispositivos.
- Conectividad de dispositivos segura.

Para más información acerca de IoT Hub, consulte [Azure IoT Hub](https://docs.microsoft.com/azure/iot-hub/).

Para obtener más información sobre la conectividad de dispositivos en Azure IoT Central, consulte [Device connectivity](concepts-get-connected.md) (Conectividad de dispositivos).

## <a name="data-stores"></a>Almacenes de datos

Azure IoT Central almacena los datos de la aplicación en la nube. Entre los datos de aplicación almacenados están los siguientes:

- Plantillas del dispositivo.
- Identidades del dispositivo.
- Metadatos del dispositivo.
- Datos de usuario y rol.

Azure IoT Central utiliza un almacén en serie temporal para los datos de medición enviados desde los dispositivos. Los datos de serie temporal de dispositivos usados por el servicio de análisis.

## <a name="analytics"></a>Análisis

El servicio de análisis es responsable de generar los datos de informes personalizados que muestra la aplicación. Un operador puede [personalizar el análisis](howto-create-analytics.md) mostrado en la aplicación. El servicio de análisis se basa en [Azure Time Series Insights](https://azure.microsoft.com/services/time-series-insights/) y procesa los datos de medición enviados desde los dispositivos.

## <a name="rules-and-actions"></a>Reglas y acciones

Las [reglas y acciones](tutorial-create-telemetry-rules.md) trabajan conjuntamente para automatizar las tareas dentro de la aplicación. Un generador puede definir reglas basadas en la telemetría de dispositivo, como la temperatura que supera un umbral definido. Azure IoT Central usa un procesador de secuencias para determinar cuando se cumplen las condiciones de la regla. Si se cumple una condición de regla, desencadena una acción definida por el generador. Por ejemplo, una acción puede enviar un correo electrónico para notificar a un ingeniero que la temperatura en un dispositivo es demasiado alta.

## <a name="metadata-management"></a>Administración de metadatos

En una aplicación de Azure IoT Central, las plantillas de dispositivo definen el comportamiento y la capacidad de los tipos de dispositivo. Por ejemplo, una plantilla de dispositivo de un refrigerador especifica la telemetría que un refrigerador envía a la aplicación.

![Arquitectura de plantillas](media/concepts-architecture/template-architecture.png)

Una plantilla de dispositivo de aplicación de IoT Central contiene:

- **Los modelos de funcionalidad del dispositivo** especifican las capacidades de un dispositivo, como la telemetría que envía, las propiedades que definen el estado del dispositivo y los comandos a los que responde el dispositivo. Las funcionalidades del dispositivo están organizadas en una o varias interfaces.
- **Las propiedades de la nube** especifican las propiedades que IoT Central almacena para un dispositivo. Estas propiedades solo se almacenan en IoT Central y nunca se envían a un dispositivo.
- Las **vistas** especifican los paneles y los formularios que crea el generador para permitir que el operador supervise y administre los dispositivos.
- Las **personalizaciones** permiten que el generador invalide algunas de las definiciones en el modelo de funcionalidad del dispositivo para que sean más relevantes para la aplicación de IoT Central.

Una aplicación puede tener uno o varios dispositivos simulados y reales basados en cada plantilla de dispositivo.

## <a name="data-export"></a>Exportación de datos

En una aplicación de Azure IoT Central, puede [exportar continuamente los datos](howto-export-data.md) a sus propias instancias de Azure Event Hubs y de Azure Service Bus. También puede exportar los datos periódicamente a la cuenta de Azure Blob Storage. IoT Central puede exportar medidas, dispositivos y plantillas de dispositivo.

## <a name="batch-device-updates"></a>Actualizaciones de dispositivos en lote

En una aplicación de Azure IoT Central, puede [crear y ejecutar trabajos](howto-run-a-job.md) para administrar los dispositivos conectados. Estos trabajos le permiten realizar actualizaciones masivas de los comandos, las configuraciones y propiedades de los dispositivos. Por ejemplo, puede crear un trabajo para aumentar la velocidad del ventilador de varias máquinas expendedoras refrigeradas.

## <a name="role-based-access-control-rbac"></a>Control de acceso basado en roles (RBAC)

Un [administrador puede definir reglas de acceso](howto-manage-users-roles.md) para una aplicación de Azure IoT Central con uno de los roles predefinidos o creando un rol personalizado. Los roles determinan las áreas de la aplicación a las que los usuarios tienen acceso y las acciones que pueden realizar.

## <a name="security"></a>Seguridad

Entre las características de seguridad de Azure IoT Central están las siguientes:

- Los datos se cifran en tránsito y en reposo.
- La autenticación se proporciona por Azure Active Directory o la cuenta de Microsoft. Se admite la autenticación de dos factores.
- Aislamiento de inquilino completo.
- Seguridad a nivel de dispositivo.

## <a name="ui-shell"></a>Shell de IU

El shell de la interfaz de usuario es una a aplicación moderna, con capacidad de respuesta y basada en un explorador HTML5.
Un administrador puede personalizar la interfaz de usuario de la aplicación mediante la aplicación de temas personalizados y la modificación de los vínculos de ayuda para que apunten a sus propios recursos de ayuda personalizados. Para más información sobre la interfaz de usuario personalizada, consulte el artículo [Personalizar la interfaz de usuario de Azure IoT Central](howto-customize-ui.md).

Un operador puede crear paneles de aplicaciones personalizados. Puede tener varios paneles que muestren datos diferentes y cambiar entre ellos.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido sobre la arquitectura de Azure IoT Central, el siguiente paso sugerido es obtener información sobre [la conectividad de dispositivos](concepts-get-connected.md) en Azure IoT Central.
