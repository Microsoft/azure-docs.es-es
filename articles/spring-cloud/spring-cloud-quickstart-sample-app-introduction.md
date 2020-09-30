---
title: 'Inicio rápido: Introducción a la aplicación de ejemplo (Azure Spring Cloud)'
description: Describe la aplicación de ejemplo que se usa en esta serie de guías de inicio rápido para la implementación en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: dd36bb18e84ea299195b77286887a3b279f81469
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903598"
---
# <a name="introduction-to-the-sample-app"></a>Introducción a la aplicación de ejemplo

::: zone pivot="programming-language-csharp"
En esta serie de guías de inicio rápido se usa una aplicación de ejemplo compuesta por dos microservicios para mostrar cómo implementar una aplicación .NET Core de Steeltoe en el servicio Azure Spring Cloud. Usará las funcionalidades de Azure Spring Cloud, como detección de servicios, servidor de configuración, registros, métricas y seguimiento distribuido.

## <a name="functional-services"></a>Servicios funcionales

La aplicación de ejemplo se compone de dos microservicios:

* El servicio `planet-weather-provider` devuelve el texto meteorológico en respuesta a una solicitud HTTP que especifica el nombre del planeta. Por ejemplo, puede devolver "muy cálido" para el planeta Mercurio. Obtiene los datos meteorológicos del servidor de configuración. El servidor de configuración obtiene los datos meteorológicos de un archivo YAML en un repositorio Git, por ejemplo:

  ```yaml
  MercuryWeather: very warm
  VenusWeather: quite unpleasant
  MarsWeather: very cool
  SaturnWeather: a little bit sandy
  ```

* El servicio `solar-system-weather` devuelve datos de cuatro planetas en respuesta a una solicitud HTTP. Obtiene los datos de realizar cuatro solicitudes HTTP a `planet-weather-provider`. Usa el servicio de detección de servidores Eureka para llamar a `planet-weather-provider`. Devuelve un archivo JSON, por ejemplo:

  ```json
  [{
      "Key": "Mercury",
      "Value": "very warm"
  }, {
      "Key": "Venus",
      "Value": "quite unpleasant"
  }, {
      "Key": "Mars",
      "Value": "very cool"
  }, {
      "Key": "Saturn",
      "Value": "a little bit sandy"
  }]
  ```

En el diagrama siguiente se muestra la arquitectura de la aplicación de ejemplo:

:::image type="content" source="media/spring-cloud-quickstart-sample-app-introduction/sample-app-diagram.png" alt-text="Diagrama de la aplicación de ejemplo":::

## <a name="code-repository"></a>Repositorio de código

La aplicación de ejemplo se encuentra en la carpeta [steeltoe-sample](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample) del repositorio Azure-Samples/Azure-Spring-Cloud-Samples en GitHub.

Las instrucciones de las siguientes guías de inicio rápido hacen referencia al código fuente según sea necesario.

::: zone-end

::: zone pivot="programming-language-java"
En esta guía de inicio rápido, usamos un ejemplo de finanzas personales denominado PiggyMetrics para mostrarle cómo implementar una aplicación en el servicio Azure Spring Cloud. PiggyMetrics muestra el patrón de arquitectura de microservicios y resalta el desglose de los servicios. Verá cómo se implementa en Azure con las potentes funcionalidades de Azure Spring Cloud, como la detección de servicios, el servidor de configuración, los registros, las métricas y el seguimiento distribuido.

Para seguir los ejemplos de implementación de Azure Spring Cloud, solo necesita la ubicación del código fuente, que se proporciona si es necesario.

## <a name="functional-services"></a>Servicios funcionales

PiggyMetrics está compuesto de tres microservicios principales, y todos ellos son aplicaciones que se pueden implementar de forma independiente y organizadas por dominios empresariales.

* **Servicio de cuenta (para implementar)** : contiene la validación y la lógica de entrada del usuario general: elementos de ingresos o gastos, ahorros y configuración de la cuenta.
* **Servicio de estadísticas (no se usa en este inicio rápido)** : realiza cálculos en los parámetros de las estadísticas principales y captura series temporales para cada cuenta. El punto de datos contiene valores, normalizados tomando como referencia la moneda base y al período. Estos datos se usan para realizar un seguimiento de la dinámica del flujo de efectivo durante la vigencia de la cuenta.
* **Servicio de notificación (no se usa en este inicio rápido)** : almacena la información de contacto de los usuarios y la configuración de notificaciones como, por ejemplo, la frecuencia de copia de seguridad y la de los recordatorios. El trabajo programado recopila la información necesaria de otros servicios y envía mensajes de correo electrónico a los clientes suscritos.

## <a name="infrastructure-services"></a>Servicios de infraestructura

En los sistemas distribuidos hay varios patrones comunes que facilitan el funcionamiento de los principales servicios. Azure Spring Cloud proporciona herramientas eficaces que mejoran el comportamiento de las aplicaciones de Spring Boot para implementar esos patrones: 

* **Servicio de configuración (hospedado en Azure Spring Cloud)** : Azure Spring Cloud Config es un servicio de configuración centralizado escalable horizontalmente para sistemas distribuidos. Usa un repositorio conectable que actualmente admite el almacenamiento local, Git y Subversion.
* **Servicio de detección (hospedado en Azure Spring Cloud)** : Permite la detección automática de ubicaciones de red para las instancias de servicio que podrían tener direcciones asignadas dinámicamente debido a la escalabilidad automática, los errores y las actualizaciones.
* **Servicio de autenticación (se implementará)** : las responsabilidades de autorización se extraen completamente en un servidor independiente, el cual concede tokens de OAuth2 a los servicios de recursos de back-end. El servidor de autorización realiza la autorización de usuarios y protege la comunicación entre máquinas dentro de un perímetro.
* **API Gateway (se implementará)** : los tres servicios principales exponen una API externa al cliente. En los sistemas reales, el número de funciones puede aumentar rápidamente con la complejidad del sistema. Cuando se representa una página web compleja puede haber cientos de servicios implicados. API Gateway es un punto de entrada único en el sistema que se usa para controlar las solicitudes y enrutarlas al servicio back-end adecuado, o para invocar varios servicios back-end, agregando los resultados. 

## <a name="sample-usage-of-piggymetrics"></a>Ejemplo de uso de PiggyMetrics

Para obtener todos los detalles sobre la implementación, consulte [PiggyMetrics](https://github.com/Azure-Samples/piggymetrics). Los ejemplos hacen referencia al código fuente cuando se necesita.
::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aprovisionamiento de una instancia de Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md)
