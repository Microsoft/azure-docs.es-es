---
title: Introducción a la administración de dispositivos con Azure IoT Hub | Microsoft Docs
description: 'Información general sobre la administración de dispositivos con Azure IoT Hub: ciclo de vida del dispositivo empresarial y patrones de administración de dispositivos como, reinicio, restablecimiento de fábrica, actualización de firmware, configuración, dispositivos gemelos, consultas y trabajos.'
author: bzurcher
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/24/2017
ms.author: briz
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: System Architecture'
ms.openlocfilehash: 63497553f7cec2edf3c77ba8690dda2b15d104be
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324733"
---
# <a name="overview-of-device-management-with-iot-hub"></a>Información general sobre la administración de dispositivos con IoT Hub

Azure IoT Hub proporciona características y un modelo de extensibilidad que permiten a los desarrolladores de back-end y de dispositivos generar soluciones sólidas de administración de dispositivos. Los dispositivos incluyen sensores restringidos y microcontroladores para un solo fin, así como puertas de enlace eficaces que enrutan las comunicaciones para grupos de dispositivos.  Además, las condiciones de uso y los requisitos de los operadores de IoT varían considerablemente de un sector a otro.  A pesar de esta variación, la administración de dispositivos con Azure IoT Hub proporciona las funcionalidades, las bibliotecas de código y los patrones adecuados para un conjunto diverso de dispositivos y usuarios finales.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Una parte fundamental de la creación de una solución de IoT empresarial adecuada es contar con una estrategia relativa al modo en que los operadores controlan la administración continua de su conjunto de dispositivos. Los operadores de IoT requieren herramientas y aplicaciones sencillas y confiables que les permitan centrarse en los aspectos más estratégicos de sus trabajos. Este artículo ofrece:

* Descripción general del método de administración de dispositivos con Azure IoT Hub.
* Descripción de los principios de administración de dispositivos comunes.
* Descripción del ciclo de vida de los dispositivos.
* Información general sobre los patrones comunes de administración de dispositivos.

## <a name="device-management-principles"></a>Principios de la administración de dispositivos

La administración de dispositivos con IoT cuenta con sus propios desafíos y todas las soluciones empresariales deben tener en cuenta los siguientes principios:

![Gráfico de los principios de la administración de dispositivos](media/iot-hub-device-management-overview/image4.png)

* **Escala y automatización**: las soluciones de IoT requieren herramientas sencillas que puedan automatizar las tareas rutinarias y que permitan que un personal de operaciones relativamente reducido administre millones de dispositivos. Los operadores diarios esperan controlar las operaciones de dispositivos de forma remota y masiva, y que solo se les alerte cuando surjan problemas que requieran su atención directa.

* **Versatilidad y compatibilidad**: el ecosistema de dispositivos es sumamente variado. Se deben personalizar las herramientas de administración para que se adapten a una gran variedad de clases de dispositivos, plataformas y protocolos. Los operadores deben ser capaces de admitir todo tipo de dispositivos, desde los chips de proceso único insertados más limitados hasta equipos potentes y totalmente funcionales.

* **Reconocimiento del contexto**: los entornos de IoT son dinámicos y cambiantes. La confiabilidad del servicio es primordial. Las operaciones de administración de dispositivos deben tener en cuenta los siguientes factores para asegurarse de que el tiempo de inactividad del mantenimiento no afecte a operaciones críticas del negocio o cree condiciones peligrosas:

    * Ventanas de mantenimiento de Acuerdo de Nivel de Servicio
    * Estados de energía y red
    * Condiciones en uso
    * Ubicación geográfica del dispositivo

* **Dar servicio a numerosos roles**: la compatibilidad con los flujos de trabajo y los procesos únicos de los roles de operaciones de IoT es fundamental. El personal de operaciones debe trabajar conforme a las restricciones especificadas de los departamentos de TI internos.  También debe encontrar modos sostenibles de exponer la información sobre las operaciones de los dispositivos en tiempo real a los supervisores y a otros roles de administración empresarial.

## <a name="device-lifecycle"></a>Ciclo de vida de dispositivo
Hay un conjunto de fases generales de administración de dispositivos que son comunes a todos los proyectos de IoT de empresa. En IoT de Azure, hay cinco fases en el ciclo de vida del dispositivo:

![Las cinco fases del ciclo de vida del dispositivo de Azure IoT son: planear, aprovisionar, configurar, supervisar, retirar.](./media/iot-hub-device-management-overview/image5.png)

Dentro de cada una de estas cinco fases, hay que cumplir varios requisitos de operador de dispositivo para proporcionar una solución completa:

* **Planeamiento**: permitir a los operadores que creen un esquema de metadatos de los dispositivos que les permita consultar y seleccionar, de forma fácil y rápida, un grupo de dispositivos para operaciones de administración masiva. Puede usar al dispositivo gemelo para almacenar los metadatos de este dispositivo en forma de etiquetas y propiedades.
  
    *Lecturas adicionales*: 
    * [Introducción a los dispositivos gemelos](iot-hub-node-node-twin-getstarted.md)
    * [Descripción de dispositivo gemelos](iot-hub-devguide-device-twins.md)
    * [Uso de propiedades de dispositivos gemelos](tutorial-device-twins.md)
    * [Procedimientos recomendados para la configuración de dispositivos en una solución de IoT](iot-hub-configuration-best-practices.md)

* **Aprovisionamiento**: aprovisionar de forma segura nuevos dispositivos en IoT Hub y permitir que los operadores puedan detectar de inmediato las funcionalidades de los dispositivos.  Use el registro de identidad de IoT Hub para crear credenciales e identidades de dispositivo flexibles, y para realizar esta operación de forma masiva mediante un trabajo. Cree dispositivos para informar de sus funcionalidades y condiciones mediante las propiedades de dispositivo en el dispositivo gemelo.
  
    *Lecturas adicionales*: 
    * [Administración de identidades de dispositivo](iot-hub-devguide-identity-registry.md)
    * [Administración de identidades de dispositivos de forma masiva](iot-hub-bulk-identity-mgmt.md)
    * [Uso de propiedades de dispositivos gemelos](tutorial-device-twins.md)
    * [Procedimientos recomendados para la configuración de dispositivos en una solución de IoT](iot-hub-configuration-best-practices.md)
    * [Servicio Azure IoT Hub Device Provisioning](https://azure.microsoft.com/documentation/services/iot-dps)

* **Configuración**: facilitar los cambios de configuración de forma masiva y las actualizaciones de firmware en dispositivos, a la vez que se mantienen el estado y la seguridad. Realice estas operaciones de administración de dispositivos de forma masiva usando las propiedades que desee o con métodos directos y trabajos de difusión.
  
    *Lecturas adicionales*:
    * [Uso de propiedades de dispositivos gemelos](tutorial-device-twins.md)
    * [Configuración y supervisión de dispositivos IoT a escala](iot-hub-auto-device-config.md)
    * [Procedimientos recomendados para la configuración de dispositivos en una solución de IoT](iot-hub-configuration-best-practices.md)

* **Supervisión**: supervisar la situación general del conjunto de dispositivos y el estado de las operaciones en curso, y alertar a los operadores sobre los problemas que puedan necesitar su atención.  Aplique dispositivos gemelos para que los dispositivos puedan notificar en tiempo real las condiciones de funcionamiento y el estado de las operaciones de actualización. Genere informes de panel eficaces que expongan los problemas más inmediatos mediante el uso de dispositivos gemelos.
  
    *Lecturas adicionales*: 
    * [Uso de propiedades de dispositivos gemelos](tutorial-device-twins.md)
    * [Lenguaje de consulta de IoT Hub para dispositivos gemelos, trabajos y enrutamiento de mensajes](iot-hub-devguide-query-language.md)
    * [Configuración y supervisión de dispositivos IoT a escala](iot-hub-auto-device-config.md)
    * [Procedimientos recomendados para la configuración de dispositivos en una solución de IoT](iot-hub-configuration-best-practices.md)

* **Retirada**: reemplazar o retirar dispositivos después de un error, ciclo de actualización o al final de la duración del servicio.  Use dispositivos gemelos para conservar la información del dispositivo si el dispositivo físico se va a reemplazar, o para archivarla si se va a retirar. Use el registro de identidades de IoT Hub para revocar de forma segura las credenciales y las identidades de los dispositivos.
  
    *Lecturas adicionales*: 
    * [Uso de propiedades de dispositivos gemelos](tutorial-device-twins.md)
    * [Administración de identidades de dispositivo](iot-hub-devguide-identity-registry.md)

## <a name="device-management-patterns"></a>Patrones de administración de dispositivos

IoT Hub habilita el siguiente conjunto de patrones de administración de dispositivos. Los [tutoriales de administración de dispositivos](iot-hub-node-node-device-management-get-started.md) muestran con más detalle cómo ampliar estos patrones para que se adapten a su escenario exacto y cómo diseñar nuevos patrones basados en estas plantillas centrales.

* **Reinicio**: la aplicación back-end usa un método directo para informar al dispositivo de que se ha iniciado un reinicio.  El dispositivo usa las propiedades notificadas para actualizar el estado de reinicio del dispositivo.
  
    ![Gráfico de los patrones de reinicio de la administración de dispositivos](./media/iot-hub-device-management-overview/reboot-pattern.png)

* **Restablecimiento de fábrica**: la aplicación back-end usa un método directo para informar al dispositivo de que se ha iniciado un restablecimiento de fábrica. El dispositivo usa las propiedades notificadas para actualizar el estado de restablecimiento de fábrica del dispositivo.
  
    ![Gráfico de los patrones de restablecimiento de fábrica de la administración de dispositivos](./media/iot-hub-device-management-overview/facreset-pattern.png)

* **Configuración**: la aplicación back-end usa las propiedades deseadas para configurar el software que se ejecuta en el dispositivo. El dispositivo usa las propiedades notificadas para actualizar el estado de configuración del dispositivo.
  
    ![Gráfico de los patrones de configuración de la administración de dispositivos](./media/iot-hub-device-management-overview/configuration-pattern.png)

* **Actualización de firmware**: la aplicación de back-end utiliza una configuración de administración de dispositivos automática para seleccionar los dispositivos que recibirán la actualización, para indicarles dónde encontrar la actualización y para supervisar el proceso de actualización. El dispositivo inicia un proceso de varios pasos para descargar, verificar y aplicar la imagen de firmware y después reiniciar el dispositivo antes de volver a conectarse al servicio IoT Hub. A lo largo del proceso de varios pasos, el dispositivo usa las propiedades notificadas para actualizar su progreso y estado.
  
    ![Gráfico del patrón de actualización del firmware de la administración de dispositivos](media/iot-hub-device-management-overview/fwupdate-pattern.png)

* **Informes de progreso y estado**: la solución back-end ejecuta consultas de dispositivos gemelos, en un conjunto de dispositivos, para informar sobre el estado y el progreso de las acciones que se ejecutan en el dispositivo.
  
    ![Gráfico del patrón del estado y progreso de la administración de dispositivos](./media/iot-hub-device-management-overview/report-progress-pattern.png)

## <a name="next-steps"></a>Pasos siguientes

Las funcionalidades, los patrones y las bibliotecas de código que IoT Hub proporciona para la administración de dispositivos le permiten crear aplicaciones de IoT que cumplan los requisitos del operador de IoT empresarial dentro de cada fase del ciclo de vida de dispositivo.

Para más información acerca de las características de administración de dispositivos en IoT Hub, consulte el tutorial [Introducción a la administración de dispositivos](iot-hub-node-node-device-management-get-started.md).