---
title: Información general de las alertas clásicas en Azure Monitor
description: Las alertas clásicas están en desuso. Las alertas permiten supervisar los registros, los eventos o las métricas de recursos de Azure y recibir una notificación cuando se cumple una condición especificada.
ms.topic: conceptual
ms.date: 05/19/2018
ms.subservice: alerts
ms.openlocfilehash: c7dfbd60b7a668b849c3150bb16e40285c345634
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844120"
---
# <a name="what-are-classic-alerts-in-microsoft-azure"></a>¿Qué son las alertas clásicas en Microsoft Azure?

> [!NOTE]
> En este artículo se describe cómo crear alertas de métrica clásicas más antiguas. Azure Monitor ahora es compatible con [una nueva experiencia de alertas y las más recientes alertas de métrica casi en tiempo real](./alerts-overview.md). Las alertas clásicas se [retiran](./monitoring-classic-retirement.md), aunque siguen con un uso limitado para los recursos que aún no admiten las nuevas alertas.
>

Las alertas le permiten configurar las condiciones en los datos y recibir notificaciones cuando las condiciones coinciden con los datos de supervisión más actualizados.

## <a name="old-and-new-alerting-capabilities"></a>Funcionalidades de alertas antiguas y nuevas

Anteriormente, Azure Monitor, Application Insights, Log Analytics y Service Health tenían funcionalidades independientes de generación de alertas. Con el tiempo, Azure ha mejorado y combinado la interfaz de usuario y los distintos métodos de alerta. La consolidación aún está en proceso.

Puede ver las alertas clásicas solo en la pantalla de usuario de alertas clásicas en Azure Portal. Para acceder a esta pantalla, debe seleccionar el botón **Ver alertas clásicas** en la pantalla de alertas. 

 ![Opciones de alertas en Azure Portal](media/alerts-classic.overview/monitor-alert-screen2.png)

La nueva experiencia de usuario para alertas presenta las siguientes ventajas en comparación con la experiencia de alertas clásicas:
- **Mejor sistema de notificaciones**: todas las nuevas alertas usan grupos de acciones, que se denominan grupos de notificaciones y acciones que pueden reutilizarse en varias alertas. Las alertas de métricas clásicas y las alertas de Log Analytics más antiguas no usan grupos de acciones.
- **Una experiencia de creación unificada**: la creación de todas las alertas para las métricas, los registros y el registro de actividad en Azure Monitor, Log Analytics y Application Insights está disponible desde un solo lugar.
- **Visualización de las alertas desencadenadas de Log Analytics en Azure Portal**: ahora también puede ver las alertas desencadenadas de Log Analytics en su suscripción. Anteriormente, estas se encontraban en un portal independiente.
- **Separación de las alertas desencadenadas y las reglas de alertas**: las reglas de alertas (la definición de la condición que desencadena una alerta) y las alertas desencadenadas (una instancia de la activación de regla de alertas) están diferenciadas, es decir, que las vistas operativas y de configuración son independientes.
- **Mejor flujo de trabajo**: la nueva experiencia de creación de Alertas guía al usuario en el proceso de configuración de una regla de alertas, lo que facilita la detección de las condiciones correctas para que se active una alerta.
- **Consolidación de alertas inteligentes** y **configuración del estado de alerta**: las alertas más recientes incluyen la funcionalidad de agrupación automática, que muestra las alertas similares juntas para reducir la sobrecarga de la interfaz de usuario. 

Las alertas de métricas más recientes presentan estas ventajas en comparación con las alertas de métricas clásicas:
- **Latencia mejorada**: las nuevas alertas de métricas se pueden ejecutar con una frecuencia de cada minuto. Las alertas de métricas anteriores siempre se ejecutan con una frecuencia de 5 minutos. Las alertas más recientes tienen un retraso cada vez menor desde que tiene lugar el problema hasta la notificación o acción (de 3 a 5 minutos). Las alertas anteriores tardan entre 5 y 15 minutos, según el tipo.  Las alertas de registro suelen tener un retraso de 10 a 15 minutos debido al tiempo que se tarda en ingerir los registros, pero los métodos de procesamiento más recientes reducen ese tiempo. 
- **Compatibilidad con métricas multidimensionales**: puede enviar alertas sobre las métricas dimensionales que le permiten supervisar un segmento interesante de la métrica.
- **Más control sobre las condiciones de la métrica**: Puede definir reglas de alertas más sofisticadas. Las nuevas alertas admiten la supervisión de los valores máximos, mínimos, promedio y totales de las métricas.
- **Supervisión combinada de varias métricas**: se pueden supervisar varias métricas (actualmente hasta dos métricas) con una sola regla. Si ambas métricas incumplen sus respectivos umbrales durante el período especificado, se desencadena una alerta.
- **Mejor sistema de notificaciones**: todas las nuevas alertas usan [grupos de acciones](./action-groups.md), que se denominan grupos de notificaciones y acciones que pueden reutilizarse en varias alertas.  Las alertas de métricas clásicas y las alertas de Log Analytics más antiguas no usan grupos de acciones. 
- **Métricas de registros** (versión preliminar pública): los datos de registro que van a Log Analytics ahora se pueden extraer y convertir en métricas de Azure Monitor y luego en alertas como cualquier otra métrica. Consulte el artículo sobre [alertas (clásicas)]() para conocer la terminología específica de las alertas clásicas. 


## <a name="classic-alerts-on-azure-monitor-data"></a>Alertas clásicas sobre datos de Azure Monitor
Hay dos tipos de alertas clásicas disponibles: alertas de métricas y alertas de registro de actividad.

* **Alertas de métricas clásicas**: la alerta se desencadena cuando el valor de una métrica específica cruza un umbral que se le haya asignado. La alerta en cuestión genera una notificación cuando se sobrepasa el umbral y se cumple la condición de alerta. En ese momento, la alerta se considera "Activada". Genera otra notificación cuando se "resuelve" (esto es, cuando se sobrepasa el umbral de nuevo y ya no se cumple la condición).

* **Alertas clásicas de registro de actividad**: una alerta de registro de streaming que se desencadena en un evento de registro de actividad que coincide con los criterios de filtro. Estas alertas tienen solo un estado "Activado". El motor de alertas simplemente aplica los criterios del filtro a cualquier evento nuevo. No busca las entradas más antiguas. Se pueden utilizar estas alertas para recibir notificaciones cuando se produzca un incidente de Service Health o cuando un usuario o una aplicación realice una operación en su suscripción, por ejemplo, "Eliminar máquina virtual".

Para los datos del registro de recursos disponibles mediante Azure Monitor, enrute los datos a Log Analytics y use una alerta de consulta de registros. Log Analytics utiliza ahora el [nuevo método de alertas](./alerts-overview.md) 

El diagrama siguiente resume los orígenes de datos en Azure Monitor y, de forma conceptual, la manera en la que se puede se puede alertar sobre esos datos.

![Alertas explicadas](media/alerts-classic.overview/Alerts_Overview_Resource_v5.png)

## <a name="taxonomy-of-alerts-classic"></a>Taxonomía de alertas (clásicas)
Azure utiliza los siguientes términos para describir las alertas clásicas y sus funciones:
* **Alerta**: una definición de criterios (una o más reglas o condiciones) que se activa cuando se cumplen.
* **Activo**: estado en el que se cumplen los criterios definidos por una alerta clásica.
* **Resuelto**: estado en el que ya no se cumplen los criterios definidos por una alerta clásica, después de que se cumplieran con anterioridad.
* **Notificación**: acción que se realiza a partir de la activación de una alerta clásica.
* **Acción**: una llamada específica que se envían al receptor de una notificación (por ejemplo, envío de un correo electrónico a una dirección o una publicación en una dirección URL de webhook). Las notificaciones normalmente pueden desencadenar varias acciones.

## <a name="how-do-i-receive-a-notification-from-an-azure-monitor-classic-alert"></a>¿Cómo se puede recibir una notificación de una alerta clásica de Azure Monitor?
Históricamente, las alertas de Azure de distintos servicios usan sus propios métodos de notificación integrados. 

Azure Monitor creó una agrupación de notificaciones reutilizables denominada *grupos de acciones*. Los grupos de acciones especifican un conjunto de receptores para una notificación. Cada vez que se activa una alerta que hace referencia al grupo de acción, todos los receptores reciben esa notificación. Los grupos de acciones permiten reutilizar una agrupación de receptores (por ejemplo, la lista de ingenieros de guardia) a través de muchos objetos de alerta. Los grupos de acciones admiten la notificación mediante la publicación en una URL de webhook, además de direcciones de correo electrónico, números de SMS y otras acciones.  Para más información, consulte [Grupos de acciones](./action-groups.md). 

Las alertas clásicas anteriores del registro de actividad usan grupos de acciones.

Pero las anteriores alertas de métricas aún no los utilizan. En cambio, se pueden configurar las siguientes acciones: 
- Enviar notificaciones de correo electrónico al administrador de servicios o a los coadministradores, o a las direcciones de correo electrónico adicionales que especifique.
- Llamar a un webhook, que permite iniciar acciones de automatización adicionales.

Los webhooks permiten la automatización y corrección, por ejemplo, mediante:
- Runbook de Azure Automation
- Función de Azure
- Aplicación lógica de Azure
- un servicio de terceros

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre las reglas de alertas y su configuración mediante:

* Más información sobre las [métricas](data-platform.md)
* Configuración de [alertas de métricas clásicas a través de Azure Portal](alerts-classic-portal.md)
* Configuración de [alertas de métricas clásicas a través de PowerShell](alerts-classic-portal.md)
* Configuración de [alertas de métricas clásicas con la interfaz de línea de comandos (CLI)](alerts-classic-portal.md)
* Configuración de [alertas de métricas clásicas con la API de REST de Azure Monitor](/rest/api/monitor/alertrules)
* Más información sobre el [registro de actividad](platform-logs-overview.md)
* Configuración de [alertas del registro de actividad a través de Azure Portal](activity-log-alerts.md)
* Configuración de [alertas del registro de actividad a través de Resource Manager](alerts-activity-log.md)
* Revisión del [esquema de webhook de alertas del registro de actividad](activity-log-alerts-webhook.md)
* Obtenga más información sobre los [grupos de acciones](action-groups.md)
* Configuración de [alertas más recientes](alerts-metric.md)

