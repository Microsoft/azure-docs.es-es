---
title: Administración y supervisión de aplicaciones con el accionador de Spring Boot de Azure
description: Aprenda a administrar y supervisar aplicaciones con el accionador de Spring Boot.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: devx-track-java
ms.openlocfilehash: 93fd286aa76a0409a515abbf8c9dabd88a9a65c4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904277"
---
# <a name="manage-and-monitor-app-with-azure-spring-boot-actuator"></a>Administración y supervisión de aplicaciones con el accionador de Spring Boot de Azure

**Este artículo se aplica a:** ✔️ Java

Después de implementar el nuevo archivo binario en la aplicación, recomendamos comprobar la característica y ver información sobre la aplicación en ejecución. En este artículo se explica cómo acceder a la API desde un punto de conexión de prueba proporcionado por Azure Spring Cloud y exponer las características listas para producción de la aplicación.

## <a name="prerequisites"></a>Requisitos previos
En este artículo se presupone que tiene una aplicación Spring Boot 2.x que se puede implementar y arrancar correctamente en el servicio Azure Spring Cloud.  Consulte [Quickstart: Inicio de una aplicación de Azure Spring Cloud existente desde Azure Portal](spring-cloud-quickstart.md)

## <a name="verify-app-through-test-endpoint"></a>Comprobación de la aplicación a través del punto de conexión de prueba
1. Vaya al **Panel de la aplicación** y haga clic en la aplicación para acceder a la página de información general de la aplicación.

1. En el panel **Información general**, debería ver un **Punto de conexión de prueba**.  Acceda a este punto de conexión desde la línea de comandos o el explorador y observe la respuesta de la API.

1. Tenga en cuenta el URI del **extremo de prueba** que se usará en la sección próxima.

>[!TIP]
> * Si la aplicación devuelve una página front-end y hace referencia a otros archivos a través de una ruta de acceso relativa, confirme que el punto de conexión de prueba termina con una barra diagonal (/). Así se asegurará de que el archivo CSS se carga correctamente.
> * Si ve la API de la aplicación desde un explorador y este le pide que escriba las credenciales de inicio de sesión para ver la página, use [Descodificar como dirección URL ](https://www.urldecoder.org/) para descodificar el punto de conexión de prueba. La descodificación de URL devuelve una dirección URL con el formato "https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/\<app-name>/\<deployment-name>".  Use este formulario para acceder al punto de conexión.

## <a name="add-actuator-dependency"></a>Adición de dependencia del actuador

Para agregar el actuador a un proyecto basado en Maven, agregue la dependencia 'Starter':

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

Compile el nuevo archivo binario e impleméntelo en la aplicación.

## <a name="enable-production-ready-features"></a>Habilitación de características listas para producción
Los puntos de conexión del actuador permiten supervisar e interactuar con la aplicación. De forma predeterminada, la aplicación Spring Boot expone los puntos de conexión `health` y `info` para mostrar información de la aplicación arbitraria e información de estado.

Para observar la configuración y el entorno configurable, necesitamos habilitar también los puntos de conexión `env` y `configgrops`.

1. Vaya al panel **Información general** de la aplicación, haga clic en **Configuración** en el menú de configuración y vaya a la página de configuración **Variables de entorno**.
1. Agregue las siguientes propiedades como en el formulario "clave: valor". Este entorno abrirá los puntos de conexión "env", "Health" e "info" del accionador de Spring Boot.

   ```
   management.endpoints.web.exposure.include: env,health,info
   ```
1. Haga clic en el botón **Guardar**; la aplicación se reiniciará automáticamente y cargará las nuevas variables de entorno.

Ahora puede volver al panel de información general de la aplicación y esperar a que el estado del aprovisionamiento cambie a "Correcto".  Habrá más de una instancia en ejecución.

> [!Note] 
> Después de exponer la aplicación al público, estos puntos de conexión del accionador también se exponen al público. Puede ocultar todos los puntos de conexión eliminando las variables de entorno `management.endpoints.web.exposure.include` y estableciendo `management.endpoints.web.exposure.exclude=*`.

## <a name="view-the-actuator-endpoint-to-view-application-information"></a>Consulta del punto de conexión del accionador para ver información de la aplicación
1. Ahora puede acceder a la URL `"<test-endpoint>/actuator/"` para ver todos los puntos de conexión expuestos por el accionador de Spring Boot.
1. Vaya a la URL `"<test-endpoint>/actuator/env"` y verá los perfiles activos usados por la aplicación y todas las variables de entorno cargadas.
1. Si quiere buscar en un entorno específico, puede acceder a la URL `"<test-endpoint>/actuator/env/{toMatch}"` para verlo.

Para ver todos los puntos de conexión integrados, lea [Exponer puntos de conexión](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints).

## <a name="next-steps"></a>Pasos siguientes

* [Descripción de las métricas en Azure Spring Cloud](spring-cloud-concept-metrics.md)
* [Descripción del estado de las aplicaciones en Azure Spring Cloud](spring-cloud-concept-app-status.md)

