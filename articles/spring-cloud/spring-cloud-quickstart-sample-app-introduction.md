---
title: 'Inicio rápido: Introducción a la aplicación de ejemplo Piggy Metrics (Azure Spring Cloud)'
description: Describe la aplicación de ejemplo Piggy Metrics que se usa en la implementación de Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/03/2020
ms.custom: devx-track-java
ms.openlocfilehash: d8eee715a3ac2256838b37c1c6722aa1307f6dd0
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89046789"
---
# <a name="introduction-to-piggy-metrics-sample-app"></a>Introducción a la aplicación de ejemplo Piggy Metrics

En este inicio rápido, usamos un ejemplo de finanzas personales denominado Piggy Metrics para mostrarle cómo implementar una aplicación en el servicio en Azure Spring Cloud. Piggy Metrics muestra el patrón de arquitectura de microservicios y resalta el desglose de los servicios. Verá cómo se implementa en Azure con las potentes funcionalidades de Azure Spring Cloud, como la detección de servicios, el servidor de configuración, los registros, las métricas y el seguimiento distribuido.

Para seguir los ejemplos de implementación de Azure Spring Cloud, solo necesita la ubicación del código fuente, que se proporciona si es necesario.

## <a name="functional-services"></a>Servicios funcionales
Piggy Metrics está compuesto de tres microservicios principales, y todos ellos son aplicaciones que se pueden implementar de forma independiente y organizadas por dominios empresariales.

* **Servicio de cuenta (para implementar)** : contiene la validación y la lógica de entrada del usuario general: elementos de ingresos o gastos, ahorros y configuración de la cuenta.
* **Servicio de estadísticas (no se usa en este inicio rápido)** : realiza cálculos en los parámetros de las estadísticas principales y captura series temporales para cada cuenta. El punto de datos contiene valores, normalizados tomando como referencia la moneda base y al período. Estos datos se usan para realizar un seguimiento de la dinámica del flujo de efectivo durante la vigencia de la cuenta.
* **Servicio de notificación (no se usa en este inicio rápido)** : almacena la información de contacto de los usuarios y la configuración de notificaciones como, por ejemplo, la frecuencia de copia de seguridad y la de los recordatorios. El trabajo programado recopila la información necesaria de otros servicios y envía mensajes de correo electrónico a los clientes suscritos.

## <a name="infrastructure-services"></a>Servicios de infraestructura
En los sistemas distribuidos hay varios patrones comunes que facilitan el funcionamiento de los principales servicios. Azure Spring Cloud proporciona herramientas eficaces que mejoran el comportamiento de las aplicaciones de Spring Boot para implementar esos patrones: 

* **Servicio de configuración (hospedado en Azure Spring Cloud)** : Azure Spring Cloud Config es un servicio de configuración centralizado escalable horizontalmente para sistemas distribuidos. Usa un repositorio conectable que actualmente admite el almacenamiento local, Git y Subversion.
* **Servicio de detección (hospedado en Azure Spring Cloud)** : Permite la detección automática de ubicaciones de red para las instancias de servicio que podrían tener direcciones asignadas dinámicamente debido a la escalabilidad automática, los errores y las actualizaciones.
* **Servicio de autenticación (se implementará)** : las responsabilidades de autorización se extraen completamente en un servidor independiente, el cual concede tokens de OAuth2 a los servicios de recursos de back-end. El servidor de autorización realiza la autorización de usuarios y protege la comunicación entre máquinas dentro de un perímetro.
* **API Gateway (se implementará)** : los tres servicios principales exponen una API externa al cliente. En los sistemas reales, el número de funciones puede aumentar rápidamente con la complejidad del sistema. Cuando se representa una página web compleja puede haber cientos de servicios implicados. API Gateway es un punto de entrada único en el sistema que se usa para controlar las solicitudes y enrutarlas al servicio back-end adecuado, o para invocar varios servicios back-end, agregando los resultados. 

## <a name="sample-usage-of-piggy-metrics"></a>Ejemplo de uso de Piggy Metrics
Para obtener todos los detalles sobre la implementación, consulte [Piggy Metrics](https://github.com/Azure-Samples/piggymetrics). Los ejemplos hacen referencia al código fuente cuando se necesita.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Aprovisionamiento de una instancia de Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md)
