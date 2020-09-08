---
title: Corrección de errores HTTP 502 y HTTP 503
description: Solucionar los errores 502 Puerta de enlace no válida y 503 Servicio no disponible en su aplicación hospedada en Azure App Service.
tags: top-support-issue
keywords: 502 Puerta de enlace no válida, 503 Servicio no disponible, error 503, error 502
ms.assetid: 51cd331a-a3fa-438f-90ef-385e755e50d5
ms.topic: article
ms.date: 07/06/2016
ms.custom: seodec18
ms.openlocfilehash: 9a41c5912ec285094c1f47540caf3744ddd4d469
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958333"
---
# <a name="troubleshoot-http-errors-of-502-bad-gateway-and-503-service-unavailable-in-azure-app-service"></a>Solucionar los errores HTTP de "502 Puerta de enlace no válida" y "503 Servicio no disponible" en Azure App Service
"502 Puerta de enlace no válida" y "503 Servicio no disponible" son errores comunes de su aplicación hospedada en [Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714). Este artículo le ayuda a solucionar estos errores.

Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure y de desbordamiento de pila](https://azure.microsoft.com/support/forums/). Como alternativa, también puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y haga clic en **Obtener soporte técnico**.

## <a name="symptom"></a>Síntoma
Cuando se dirige a la aplicación, se devuelve un error HTTP "502 Puerta de enlace no válida" o un error HTTP "503 Servicio no disponible".

## <a name="cause"></a>Causa
Con frecuencia este problema se debe a problemas en el nivel de la aplicación, como, por ejemplo:

* Solicitudes que tardan mucho tiempo
* Aplicaciones que hacen un uso elevado de memoria y CPU
* Aplicaciones que se bloquean debido a una excepción.

## <a name="troubleshooting-steps-to-solve-502-bad-gateway-and-503-service-unavailable-errors"></a>Pasos de solución de problemas para resolver los errores "502 Puerta de enlace no válida" y "503 Servicio no disponible"
El procedimiento de solución de problemas se puede dividir en tres tareas distintas, en orden secuencial:

1. [Observación y supervisión del comportamiento de la aplicación](#observe)
2. [Recopilación de datos](#collect)
3. [Mitigación del problema](#mitigate)

[App Service](overview.md) ofrece diversas opciones en cada paso.

<a name="observe"></a>

### <a name="1-observe-and-monitor-application-behavior"></a>1. Observación y supervisión del comportamiento de la aplicación
#### <a name="track-service-health"></a>Seguimiento del estado del servicio
Cada vez que hay una interrupción del servicio o una degradación del rendimiento, Microsoft Azure lo anuncia. Puede realizar un seguimiento del estado del servicio en el [Azure Portal](https://portal.azure.com/). Para más información, consulte [Track service health](../service-health/service-notifications.md) (Seguimiento del estado del servicio).

#### <a name="monitor-your-app"></a>Supervise la aplicación
Esta opción le permite averiguar si la aplicación tiene problemas. En la hoja de la aplicación, haga clic en el icono **Solicitudes y errores**. La hoja **Métrica** mostrará todas las métricas que puede agregar.

Algunas de las métricas que podría querer supervisar para su aplicación son:

* Espacio de trabajo de memoria promedio
* Tiempo medio de respuesta
* Tiempo de CPU
* Espacio de trabajo de memoria
* Requests

![supervisar la aplicación para solucionar los errores HTTP de 502 Puerta de enlace no válida y 503 Servicio no disponible](./media/app-service-web-troubleshoot-HTTP-502-503/1-monitor-metrics.png)

Para más información, consulte:

* [Supervisión de aplicaciones en Azure App Service](web-sites-monitor.md)
* [Recibir notificaciones de alerta](../azure-monitor/platform/alerts-overview.md)

<a name="collect"></a>

### <a name="2-collect-data"></a>2. Recopilación de datos
#### <a name="use-the-diagnostics-tool"></a>Uso de la herramienta de diagnósticos
App Service proporciona una experiencia inteligente e interactiva que le ayuda a solucionar los problemas de su aplicación sin necesidad de configurar nada. Cuando surgen problemas con la aplicación, la herramienta de diagnóstico le indicará lo que pasa y le guiará a la información correcta de forma que pueda solucionar el problema de la manera más fácil y rápida.

Para acceder al diagnóstico de App Service, vaya a la aplicación de App Service o del entorno de App Service en [Azure Portal](https://portal.azure.com). En el panel izquierdo, haga clic en **Diagnosticar y solucionar problemas**.

#### <a name="use-the-kudu-debug-console"></a>Uso de la consola de depuración Kudu
App Service incluye una consola de depuración que puede usar para depurar, explorar o cargar archivos, e incluye también puntos de conexión JSON para obtener información sobre su entorno. A esto se le denomina *Consola Kudu* o *Panel SCM* para la aplicación.

Puede tener acceso a este panel en el vínculo **https://&lt;Nombre de la aplicación>.scm.azurewebsites.net/** .

Algunas de las cosas que proporciona Kudu son:

* Configuración del entorno de la aplicación
* transmisión de registro
* Volcado de diagnóstico
* Depuración de la consola en la que puede ejecutar cmdlets de Powershell y comandos básicos de DOS.

Otra característica útil de Kudu es que, en caso de que la aplicación inicie excepciones de primera oportunidad, puede usar Kudu y la herramienta Procdump de SysInternals para crear volcados de memoria. Estos volcados de memoria son instantáneas del proceso y a menudo pueden ayudarle solucionar problemas más complicados de su aplicación.

Para obtener más información sobre las características disponibles en Kudu, consulte [Herramientas en línea de Azure Websites que debe conocer](https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/).

<a name="mitigate"></a>

### <a name="3-mitigate-the-issue"></a>3. Mitigación del problema
#### <a name="scale-the-app"></a>Escalar la aplicación
En Azure App Service, puede ajustar la escala en la que se ejecuta la aplicación para aumentar el rendimiento y la capacidad de proceso. El escalado vertical de una aplicación implica dos acciones relacionadas: cambiar el plan de App Service por un plan de tarifa más alto y configurar determinados valores después de haber cambiado a ese plan de tarifa más alto.

Para obtener más información sobre el escalado, consulte [Escalado de una aplicación en Azure App Service](manage-scale-up.md).

Además, puede elegir ejecutar la aplicación en más de una instancia. Esto no solo le proporciona una mayor capacidad de procesamiento, sino también algo de tolerancia a errores. Si el proceso se interrumpe en una instancia, la otra instancia seguirá atendiendo las solicitudes.

Puede establecer el escalado en Manual o Automático.

#### <a name="use-autoheal"></a>Uso de AutoHeal
AutoHeal recicla el proceso de trabajo para su aplicación en función de la configuración que elija (como cambios de configuración, solicitudes, límites de memoria o el tiempo necesario para ejecutar una solicitud). Casi siempre, el proceso de reciclaje es la forma más rápida de recuperarse de un problema. Aunque siempre puede reiniciar la aplicación directamente en Azure Portal, AutoHeal lo hará automáticamente por usted. Todo lo que debe hacer es agregar algunos desencadenadores en web.config raíz de la aplicación. Tenga en cuenta que esta configuración funcionaría igual incluso si la aplicación no fuera .Net.

Para obtener más información, consulte [Recuperación automática de Sitios web de Azure](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites/).

#### <a name="restart-the-app"></a>Reiniciar la aplicación
Suele ser la manera más sencilla de recuperarse de problemas que solo tienen lugar una vez. En el [Azure Portal](https://portal.azure.com/), en la hoja de la aplicación, tiene las opciones para detener o reiniciar la aplicación.

 ![reiniciar la aplicación para solucionar los errores HTTP de 502 Puerta de enlace no válida y 503 Servicio no disponible](./media/app-service-web-troubleshoot-HTTP-502-503/2-restart.png)

También puede administrar la aplicación con Azure Powershell. Para obtener más información, vea [Uso de Azure PowerShell con el Administrador de recursos de Azure](../azure-resource-manager/management/manage-resources-powershell.md).