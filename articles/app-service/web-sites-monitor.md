---
title: Supervisión de aplicaciones
description: Aprenda a supervisar aplicaciones en Azure App Service mediante Azure Portal. Descripción de las cuotas y las métricas que se indican.
author: btardif
ms.assetid: d273da4e-07de-48e0-b99d-4020d84a425e
ms.topic: article
ms.date: 04/23/2020
ms.author: byvinyal
ms.custom: seodec18
ms.openlocfilehash: 888118d227529110c209d7c8d5a3bb79cfcf3a9a
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88959798"
---
# <a name="monitor-apps-in-azure-app-service"></a>Supervisión de aplicaciones en Azure App Service
[Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714) proporciona funciones de supervisión integradas para aplicaciones Web, móviles y aplicaciones de API en el [Azure Portal](https://portal.azure.com).

En Azure Portal, puede repasar las *cuotas* y *métricas* de una aplicación y un plan de App Service, así como configurar *alertas* y *escalado* automático a partir de métricas.

## <a name="understand-quotas"></a>Información sobre las cuotas

Las aplicaciones que se hospedan en App Service están sujetas a ciertos límites en lo que respecta a los recursos que pueden utilizar. Los límites se definen mediante el plan de App Service asociado a la aplicación.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Si la aplicación se hospeda en un plan *gratis* o *compartido*, los límites de los recursos que la aplicación puede usar vienen definidos por las cuotas.

Si la aplicación está hospedada en un plan *Básico*, *Estándar* o *Premium*, los límites de los recursos que se pueden utilizar vendrán definidos por el *tamaño* (pequeño, mediano o grande) y el *recuento de instancias* (1, 2, 3, etc.) del plan de App Service.

Las cuotas de las aplicaciones gratis o compartidas son:

| Quota | Descripción |
| --- | --- |
| **CPU (breve)** | Cantidad de CPU permitida para esta aplicación en un intervalo de cinco minutos. Esta cuota se restablece cada cinco minutos. |
| **CPU (día)** | Cantidad total de CPU permitida para esta aplicación en un día. Esta cuota se restablece cada 24 horas a medianoche (UTC). |
| **Memoria** | Cantidad total de memoria permitida para esta aplicación. |
| **Ancho de banda** | Cantidad total de ancho de banda saliente permitido para esta aplicación en un día. Esta cuota se restablece cada 24 horas a medianoche (UTC). |
| **Sistema de archivos** | Cantidad total de almacenamiento permitido. |

La única cuota aplicable a las aplicaciones que se hospedan en un plan *Básico*, *Estándar* o *Premium* es la del sistema de archivos.

Para más información sobre cuotas específicas, límites y características disponibles para las distintas SKU de App Service, consulte los [límites del servicio de suscripción de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).

### <a name="quota-enforcement"></a>Aplicación de cuotas

Si una aplicación supera las cuotas *CPU (breve)* , *CPU (día)* o *Ancho de banda*, se detiene hasta que vuelva a restablecerse la cuota. Durante este tiempo, todas las solicitudes entrantes dan como resultado un error HTTP 403.

![Mensaje de error 403][http403]

Si se supera la cuota de memoria de la aplicación, esta se detiene temporalmente.

Si se supera la cuota del sistema de archivos, se producirá un error en la operación de escritura. Entre los errores de la operación de escritura se incluyen cualquier escritura en los registros.

Se pueden incrementar o quitar de la aplicación mediante la actualización del plan de App Service.

## <a name="understand-metrics"></a>Información sobre las métricas

> [!NOTE]
> **Uso del sistema de archivos** es una nueva métrica que se está implementando globalmente; no se espera ningún dato a menos que se haya incluido en la lista blanca de la versión preliminar privada.
> 

> [!IMPORTANT]
> **Tiempo medio de respuesta** pasará a estar en desuso para evitar confusiones con las agregaciones de métricas. Use **Tiempo de respuesta** a modo de reemplazo.

> [!NOTE]
> Las métricas de una aplicación incluyen las solicitudes al sitio de SCM de la aplicación (Kudu).  Esto incluye las solicitudes para ver el LogStream del sitio con Kudu.  Las solicitudes de LogStream pueden abarcar varios minutos, lo que afectará a las métricas de tiempo de solicitud.  Los usuarios deben tener en cuenta esta relación al usar estas métricas con la lógica de escalado automático.
> 

Las métricas proporcionan información acerca de la aplicación o el comportamiento del plan de App Service.

Para una aplicación, estas son las métricas disponibles:

| Métrica | Descripción |
| --- | --- |
| **Tiempo de respuesta** | Tiempo en segundos necesario para que la aplicación atienda solicitudes. |
| **Tiempo medio de respuesta (en desuso)** | Tiempo promedio en segundos necesario para que la aplicación atienda solicitudes. |
| **Espacio de trabajo de memoria promedio** | Cantidad media de memoria que utiliza la aplicación, en megabytes (MiB). |
| **Conexiones** | Número de sockets enlazados existente en el espacio aislado (w3wp.exe y sus procesos secundarios).  Un socket enlazado se crea mediante una llamada a las API bind()/connect() y permanece hasta que el socket se cierra con CloseHandle()/closesocket(). |
| **Tiempo de CPU** | Cantidad de CPU consumida por la aplicación, en segundos. Para más información sobre esta métrica, consulte [Tiempo de CPU y porcentaje de CPU](#cpu-time-vs-cpu-percentage). |
| **Ensamblados actuales** | Número actual de los ensamblados cargados en todos los dominios de aplicación de esta aplicación. |
| **Entrada de datos** | Cantidad de ancho de banda entrante consumido por la aplicación, en MiB. |
| **Salida de datos** | Cantidad de ancho de banda saliente consumido por la aplicación, en MiB. |
| **Uso del sistema de archivos** | Porcentaje de la cuota de sistema de archivos consumida por la aplicación. |
| **Recolección de elementos no utilizados de gen. 0** | Número de veces que los objetos de generación 0 son elementos no utilizados recopilados desde el inicio del proceso de la aplicación. Los elementos no utilizados de última generación incluyen los de generaciones anteriores.|
| **Recolección de elementos no utilizados de gen. 1** | Número de veces que los objetos de generación 1 son elementos no utilizados recopilados desde el inicio del proceso de la aplicación. Los elementos no utilizados de última generación incluyen los de generaciones anteriores.|
| **Recolección de elementos no utilizados de gen. 2** | Número de veces que los objetos de generación 2 son elementos no utilizados recopilados desde el inicio del proceso de la aplicación.|
| **Número de identificadores** | Total de identificadores abiertos actualmente por el proceso de la aplicación.|
| **Http 2xx** | Cantidad total de solicitudes que devuelven un código de estado HTTP >= 200, pero < 300. |
| **Http 3xx** | Cantidad total de solicitudes que devuelven un código de estado HTTP >= 300, pero < 400. |
| **Http 401** | Cantidad total de solicitudes que devuelven el código de estado HTTP 401. |
| **Http 403** | Cantidad total de solicitudes que devuelven el código de estado HTTP 403. |
| **Http 404** | Cantidad total de solicitudes que devuelven el código de estado HTTP 404. |
| **Http 406** | Cantidad total de solicitudes que devuelven el código de estado HTTP 406. |
| **Http 4xx** | Cantidad total de solicitudes que devuelven un código de estado HTTP >= 400, pero < 500. |
| **Errores de servidor HTTP** | Cantidad total de solicitudes que devuelven un código de estado HTTP >= 500, pero < 600. |
| **Otros bytes de E/S por segundo** | La velocidad a la que el proceso de la aplicación emite bytes en las operaciones de E/S que no implican datos, como las operaciones de control.|
| **Otras operaciones de E/S por segundo** | La velocidad a la que el proceso de la aplicación emite operaciones de E/S que no son operaciones de lectura o escritura.|
| **Bytes de lectura de E/S por segundo** | Velocidad a la que el proceso de la aplicación está leyendo bytes de las operaciones de E/S.|
| **Operaciones de lectura de E/S por segundo** | Velocidad a la que el proceso de la aplicación está realizando operaciones de lectura de E/S.|
| **Bytes de escritura de E/S por segundo** | Velocidad a la que el proceso de la aplicación está escribiendo bytes de las operaciones de E/S.|
| **Operaciones de escritura de E/S por segundo** | Velocidad a la que el proceso de la aplicación está realizando operaciones de escritura de E/S.|
| **Espacio de trabajo de memoria** | Cantidad actual de memoria utilizada por la aplicación, en MiB. |
| **Bytes privados** | Bytes privados es el tamaño actual, en bytes, de la memoria asignada por el proceso de la aplicación que no se puede compartir con otros procesos.|
| **Solicitudes** | Número total de solicitudes, independientemente de su código de estado HTTP resultante. |
| **Solicitudes en la cola de la aplicación** | Número de solicitudes en la cola de solicitudes de la aplicación.|
| **Número de subprocesos** | Número de subprocesos activos actualmente en el proceso de la aplicación.|
| **Dominios de aplicación totales** | Número actual de dominios de aplicación cargados en esta aplicación.|
| **Dominios de aplicación totales descargados** | Número total de dominios de aplicación descargados desde el inicio de la aplicación.|


Para un plan de App Service, estas son las métricas disponibles:

> [!NOTE]
> Las métricas de plan de App Service solo están disponibles para planes *Básico*, *Estándar* o *Premium*.
> 

| Métrica | Descripción |
| --- | --- |
| **Porcentaje de CPU** | Uso promedio de CPU de todas las instancias del plan. |
| **Porcentaje de memoria** | Uso promedio de memoria entre todas las instancias del plan. |
| **Entrada de datos** | Uso promedio de ancho de banda entrante entre todas las instancias del plan. |
| **Salida de datos** | Uso promedio de ancho de banda saliente entre todas las instancias del plan. |
| **Longitud de la cola de disco** | Cantidad media de solicitudes de lectura y escritura en cola en Storage. Una longitud de cola de disco alta es una indicación de que una aplicación podría ralentizarse debido a una excesiva E/S de disco. |
| **Longitud de la cola HTTP** | Promedio de solicitudes HTTP que estuvieron en cola antes de realizarse. Una longitud de la cola HTTP elevada o creciente indica que un plan está sobrecargado. |

### <a name="cpu-time-vs-cpu-percentage"></a>Tiempo de CPU y porcentaje de CPU
<!-- To do: Fix Anchor (#CPU-time-vs.-CPU-percentage) -->

Hay dos métricas que reflejan el uso de CPU:

**Tiempo de CPU**: es útil para las aplicaciones hospedadas en planes gratis o compartidos, porque una de sus cuotas está definida en minutos de CPU utilizados por la aplicación.

**Porcentaje de CPU**: es útil para las aplicaciones hospedadas en los planes de tipo Básico, Estándar y Premium, dado que se pueden escalar horizontalmente. Porcentaje de CPU es una buena indicación del uso general en todas las instancias.

## <a name="metrics-granularity-and-retention-policy"></a>Directiva de retención y granularidad de métricas
El servicio registra y agrega las métricas de una aplicación y de un plan del servicio de aplicación, y [estas se conservan según estas reglas](../azure-monitor/platform/data-platform-metrics.md#retention-of-metrics).

## <a name="monitoring-quotas-and-metrics-in-the-azure-portal"></a>Supervisión de cuotas y métricas en Azure Portal
Para consultar el estado de las diversas cuotas y métricas que afectan a una aplicación, vaya a [Azure Portal](https://portal.azure.com).

![Gráfico de cuotas en Azure Portal][quotas]

Para buscar las cuotas, seleccione **Configuración** > **Cuotas**. En el gráfico, puede consultar: 
1. El nombre de la cuota.
1. Su intervalo de restablecimiento.
1. Su límite actual.
1. Su valor actual.

![Gráfico de métricas en el Azure Portal][metrics] Puede acceder a las métricas directamente desde la **página de información general** de recursos. Aquí verá los gráficos que representan algunas de las métricas de aplicaciones.

Al hacer clic en cualquiera de estos gráficos, irá a la vista de métricas, donde puede crear gráficos personalizados, consultar diferentes métricas y mucho más. 

Para más información sobre las métricas, consulte [Supervisión de las métricas del servicio](../azure-monitor/platform/data-platform.md).

## <a name="alerts-and-autoscale"></a>Alertas y escalabilidad automática
Las métricas para una aplicación o un plan de App Service pueden enlazarse con las alertas. Para más información, consulte [Recibir notificaciones de alerta](../azure-monitor/platform/alerts-classic-portal.md).

Las aplicaciones de App Service hospedadas en los planes de App Service básico o superior admiten el escalado automático. Con la escalabilidad automática puede configurar reglas que supervisan las métricas del plan de App Service. Las reglas pueden aumentar o disminuir el recuento de instancias que pueden proporcionar recursos adicionales, según sea necesario. Las reglas también le ayudan a ahorrar dinero cuando la aplicación se aprovisiona en exceso.

Para más información acerca de la escalabilidad automática, consulte [Escalado](../azure-monitor/platform/autoscale-get-started.md) y [Procedimientos recomendados de escalado automático en Azure Monitor](../azure-monitor/platform/autoscale-best-practices.md).

[fzilla]:https://go.microsoft.com/fwlink/?LinkId=247914
[vmsizes]:https://go.microsoft.com/fwlink/?LinkID=309169

<!-- Images. -->
[http403]: ./media/web-sites-monitor/http403.png
[quotas]: ./media/web-sites-monitor/quotas.png
[metrics]: ./media/web-sites-monitor/metrics.png