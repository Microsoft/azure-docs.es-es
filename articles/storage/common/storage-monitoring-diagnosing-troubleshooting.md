---
title: Supervisión, diagnóstico y solución de problemas de Azure Storage | Microsoft Docs
description: Use características como análisis de almacenamiento, registro del lado cliente y otras herramientas de terceros para identificar, diagnosticar y solucionar problemas relacionados con Azure Storage.
author: normesta
ms.service: storage
ms.topic: troubleshooting
ms.date: 09/23/2019
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring, devx-track-csharp
ms.openlocfilehash: 93015427dddfe2b311783c20587792e34c098ce8
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011045"
---
# <a name="monitor-diagnose-and-troubleshoot-microsoft-azure-storage"></a>Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage
[!INCLUDE [storage-selector-portal-monitoring-diagnosing-troubleshooting](../../../includes/storage-selector-portal-monitoring-diagnosing-troubleshooting.md)]

## <a name="overview"></a>Información general
Diagnosticar y solucionar problemas en una aplicación distribuida que está hospedada en un entorno de nube puede ser más complicado que en los entornos tradicionales. Las aplicaciones se pueden implementar en una infraestructura de PaaS o IaaS, en una ubicación local, en un dispositivo móvil o en varios de estos entornos combinados. Normalmente, el tráfico de red de la aplicación puede atravesar redes públicas y privadas, y la aplicación puede usar varias tecnologías de almacenamiento, como tablas, blobs, colas o archivos de Microsoft Azure Storage, además de otros almacenes de datos como, por ejemplo, bases de datos relacionales y de documentos.

Para administrar correctamente esas aplicaciones, debe supervisarlas de forma proactiva y comprender cómo diagnosticar y solucionar problemas relacionados con todos sus aspectos y con las tecnologías de las que dependen. Como usuario de los servicios de Azure Storage, debe supervisar continuamente los servicios de Almacenamiento que utiliza la aplicación para observar si se produce algún cambio inesperado en el comportamiento (por ejemplo, tiempos de respuesta más lentos de lo habitual), y usar el registro para recopilar datos más detallados y poder analizar los problemas en profundidad. Con la información de diagnóstico que obtenga de la supervisión y el registro, le será más fácil averiguar la causa raíz del problema que apareció en la aplicación. Entonces, podrá solucionar el problema y determinar los pasos más adecuados que puede tomar para remediarlo. Azure Storage es uno de los servicios principales de Azure y constituye una parte importante de la mayoría de las soluciones que implementan los clientes en la infraestructura de Azure. Azure Storage incluye funcionalidades que permiten simplificar la supervisión, el diagnóstico y la solución de problemas de almacenamiento de las aplicaciones basadas en la nube.

Para obtener una guía práctica para la solución de problemas integral en aplicaciones de almacenamiento de Azure, consulte [End-to-End Troubleshooting using Azure Storage Metrics and Logging, AzCopy, and Message Analyzer](../storage-e2e-troubleshooting.md)(Solución de problemas integral con el analizador de mensajes, AzCopy y las métricas y el registro del almacenamiento de Azure).

* [Introducción]
  * [Organización de la guía]
* [Supervisión del servicio de almacenamiento]
  * [Supervisión del estado del servicio]
  * [Supervisión de la capacidad]
  * [Supervisión de la disponibilidad]
  * [Supervisión del rendimiento]
* [Diagnóstico de problemas de almacenamiento]
  * [Problemas de estado del servicio]
  * [Problemas de rendimiento]
  * [Diagnóstico de errores]
  * [Problemas del emulador de almacenamiento]
  * [Herramientas de registro de almacenamiento]
  * [Uso de herramientas de registro de red]
* [Seguimiento integral]
  * [Correlación de datos de registro]
  * [Id. de solicitud de cliente]
  * [Identificador de solicitud de servidor]
  * [Marcas de tiempo]
* [Guía de solución de problemas]
  * [Las métricas muestran una AverageE2ELatency alta y una AverageServerLatency baja]
  * [Las métricas muestran una AverageE2ELatency baja y una AverageServerLatency baja, pero el cliente tiene latencia alta]
  * [Las métricas muestran una AverageServerLatency alta]
  * [Observa retrasos inesperados en la entrega de mensajes de una cola]
  * [Las métricas muestran un aumento de PercentThrottlingError]
  * [Las métricas muestran un aumento de PercentTimeoutError]
  * [Las métricas muestran un aumento de PercentNetworkError]
  * [El cliente recibe mensajes HTTP 403 (prohibido)]
  * [El cliente recibe mensajes HTTP 404 (no encontrado)]
  * [El cliente recibe mensajes HTTP 409 (conflicto)]
  * [Las métricas muestran un PercentSuccess bajo o las entradas de registro de análisis tienen operaciones con el estado de transacción ClientOtherErrors]
  * [Las métricas de capacidad muestran un aumento inesperado en el uso de la capacidad de almacenamiento]
  * [El problema se presenta al usar el emulador de almacenamiento para realizar tareas de desarrollo o pruebas]
  * [Se producen problemas al instalar el SDK de Azure para .NET]
  * [Tiene otro problema distinto relacionado con un servicio de almacenamiento]
  * [Solución de problemas de discos duros virtuales con máquinas virtuales Windows](../../virtual-machines/windows/troubleshoot-vhds.md)   
  * [Solución de problemas de discos duros virtuales con máquinas virtuales Linux](../../virtual-machines/linux/troubleshoot-vhds.md)
  * [Solución de problemas de Azure Files con Windows](../files/storage-troubleshoot-windows-file-connection-problems.md)   
  * [Solución de problemas de Azure Files con Linux](../files/storage-troubleshoot-linux-file-connection-problems.md)
* [Apéndices]
  * [Apéndice 1: Uso de Fiddler para capturar tráfico HTTP y HTTPS]
  * [Apéndice 2: Uso de Wireshark para capturar tráfico de red]
  * [Apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red]
  * [Apéndice 4: Uso de Excel para ver métricas y datos de registro]
  * [Apéndice 5: Supervisión mediante Application Insights para Azure DevOps]

## <a name="introduction"></a><a name="introduction"></a>Introducción
En esta guía se explica cómo usar algunas características, como el análisis de Azure Storage, el registro del lado cliente de la biblioteca de cliente de Azure Storage y otras herramientas de terceros para identificar, diagnosticar y solucionar problemas relacionados con Azure Storage.

![Diagrama que muestra el flujo de información entre las aplicaciones cliente y los servicios de Azure Storage.][1]

Esta guía está dirigida, principalmente, a los desarrolladores de servicios en línea que usan los servicios de Azure Storage y a los profesionales de TI responsables de administrar esos servicios en línea. Los objetivos de esta guía son:

* Ayudarle a mantener el estado y el rendimiento de las cuentas de Azure Storage.
* Proporcionarle las herramientas y los procesos necesarios para que le resulte más fácil averiguar si un problema de una aplicación está relacionado con Azure Storage.
* Proporcionarle una guía para que pueda actuar y resolver problemas relacionados con Azure Storage.

### <a name="how-this-guide-is-organized"></a><a name="how-this-guide-is-organized"></a>Organización de la guía
La sección "[Supervisión del servicio de almacenamiento]" describe cómo supervisar el estado y el rendimiento de los servicios de Azure Storage con las métricas de análisis de Azure Storage (métricas de Almacenamiento).

La sección “[Diagnóstico de problemas de almacenamiento]” describe cómo diagnosticar problemas con el registro de análisis de Azure Storage (registro de Almacenamiento). También explica cómo habilitar el registro del lado cliente con los recursos de una de las bibliotecas de cliente como, por ejemplo, la biblioteca de cliente de Almacenamiento para .NET o el SDK de Azure para Java.

La sección “[Seguimiento integral]” describe cómo poner en correlación la información que contienen diversos archivos de registro y datos de métricas.

La sección “[Guía de solución de problemas]” proporciona una guía de solución de problemas para algunos de los problemas habituales relacionados con el almacenamiento que puede encontrarse.

Los "[Apéndices]" contienen información sobre el uso de otras herramientas, como Wireshark y Netmon para analizar datos de paquetes de red, Fiddler para analizar mensajes HTTP/HTTPS y el Analizador de mensajes de Microsoft para poner en correlación los datos de registro.

## <a name="monitoring-your-storage-service"></a><a name="monitoring-your-storage-service"></a>Supervisión del servicio de almacenamiento
Si conoce la supervisión de rendimiento de Windows, las métricas de Almacenamiento son algo así como los contadores del monitor de rendimiento de Windows, pero en Azure Storage. En las métricas de almacenamiento encontrará un conjunto completo de métricas (o "contadores", según la terminología del Monitor de rendimiento de Windows), tales como la disponibilidad del servicio, el número total de solicitudes realizadas al servicio o el porcentaje de solicitudes correctas realizadas al servicio. Si quiere obtener la lista completa de métricas disponibles, consulte [Esquema de tabla de métricas de Storage Analytics](https://msdn.microsoft.com/library/azure/hh343264.aspx). Puede especificar si quiere que el servicio de almacenamiento recopile y agregue métricas cada hora o cada minuto. Para más información sobre cómo habilitar las métricas y supervisar las cuentas de almacenamiento, consulte el tema sobre cómo [Habilitar las métricas de almacenamiento y ver los datos de métricas](https://go.microsoft.com/fwlink/?LinkId=510865).

Puede elegir las métricas horarias que quiere mostrar en [Azure Portal](https://portal.azure.com) y configurar reglas que notifiquen a los administradores por correo electrónico cada vez que una métrica horaria supere un umbral determinado. Para más información, consulte [Recibir notificaciones de alerta](/azure/monitoring-and-diagnostics/monitoring-overview-alerts).

Se recomienda consultar [Azure Monitor para Storage](../../azure-monitor/insights/storage-insights-overview.md) (versión preliminar). Es una característica de Azure Monitor que ofrece una supervisión completa de las cuentas de Azure Storage, ya que ofrece una vista unificada del rendimiento, la capacidad y la disponibilidad de los servicios de Azure Storage. No es preciso habilitar o configurar nada, y puede ver inmediatamente estas métricas desde los gráficos interactivos predefinidos y otras visualizaciones incluidas.

El servicio de almacenamiento recopila métricas con un modelo de mejor esfuerzo, pero es posible que no registre todas las operaciones de almacenamiento.

En Azure Portal, puede ver métricas con cifras sobre la disponibilidad, el número total de solicitudes y la latencia media de una cuenta de almacenamiento. También se configuró una regla de notificación para que se envíe una alerta a un administrador si la disponibilidad desciende por debajo de un nivel determinado. Al observar estos datos, uno de los aspectos que podría examinar es que el porcentaje de operaciones correctas del servicio Tabla sea inferior al 100 % (para obtener más información, consulte la sección "[Las métricas muestran un PercentSuccess bajo o las entradas de registro de análisis tienen operaciones con el estado de transacción ClientOtherErrors]").

Debe supervisar continuamente las aplicaciones de Azure para asegurarse de que el estado es correcto y el rendimiento es el esperado, haciendo lo siguiente:

* Establecer varias métricas de línea base para la aplicación que le permitan comparar los datos actuales e identificar todos los cambios significativos en el comportamiento del almacenamiento de Azure y la aplicación. En muchos casos, los valores de las métricas de línea base serán específicos de cada aplicación y deberá establecerlos al probar el rendimiento de la aplicación.
* Registrar métricas por minuto y usarlas para supervisar activamente si se producen errores inesperados y anomalías como, por ejemplo, picos en el número de errores o la tasa de solicitudes.
* Registrar métricas horarias y usarlas para supervisar los valores medios, como el número medio de errores y las tasas medias de solicitudes.
* Investigar problemas potenciales con herramientas de diagnóstico, como se examina más adelante en la sección “[Diagnóstico de problemas de almacenamiento]”.

Los gráficos de la siguiente imagen ilustran cómo el promedio resultante de las métricas horarias puede ocultar picos de actividad. Las métricas horarias parecen mostrar una tasa de solicitudes regular, mientras que las métricas por minuto revelan las fluctuaciones que se están produciendo en realidad.

![Gráficos muestran la forma en que el promedio resultante de las métricas horarias puede ocultar picos de actividad.][3]

En el resto de esta sección, se explica qué métricas debe supervisar y por qué.

### <a name="monitoring-service-health"></a><a name="monitoring-service-health"></a>Supervisión del estado del servicio
Puede usar [Azure Portal](https://portal.azure.com) para ver el estado del servicio Storage (y de otros servicios de Azure) en todas las regiones de Azure del mundo. La supervisión le permite ver al momento si hay algún problema que no controle y que esté afectando al servicio de almacenamiento en la región que use para la aplicación.

[Azure Portal](https://portal.azure.com) también puede proporcionar notificaciones sobre los incidentes que afectan a los diversos servicios de Azure.
Nota: Anteriormente esta información estaba disponible, junto con los datos históricos, en el [Panel de servicios de Azure](https://status.azure.com).

Mientras que [Azure Portal](https://portal.azure.com) recopila información sobre el estado del servicio desde el interior de los centros de datos de Azure (esto es, una supervisión desde el interior al exterior), también puede adoptar un método que vaya del exterior al interior para generar transacciones sintéticas que accedan periódicamente a la aplicación web hospedada por Azure desde varias ubicaciones. Los servicios que ofrecen [Dynatrace](https://www.dynatrace.com/en/synthetic-monitoring) y Application Insights para Azure DevOps son algunos de ejemplos de este enfoque. Para más información sobre Application Insights para Azure DevOps, consulte el "[Apéndice 5: Supervisión mediante Application Insights para Azure DevOps](#appendix-5)".

### <a name="monitoring-capacity"></a><a name="monitoring-capacity"></a>Supervisión de la capacidad
Las métricas de Storage solo almacenan las métricas de capacidad de Blob service, porque los blob suelen representar la mayor parte de los datos almacenados (en el momento de la escritura, no se pueden usar las métricas de Storage para supervisar la capacidad de las tablas y las colas). Puede encontrar estos datos en la tabla **$MetricsCapacityBlob** si habilitó la supervisión de Blob service. Las métricas de Almacenamiento registran estos datos una vez al día, y puede usar el valor de **RowKey** para saber si la fila contiene una entidad relacionada con datos de usuarios (valor **data**) o con datos de análisis (valor **analytics**). Cada entidad almacenada contiene información sobre la cantidad de almacenamiento utilizada (**Capacity**, medida en bytes) y el número actual de contenedores (**ContainerCount**) y BLOB (**ObjectCount**) que se están usando en la cuenta de almacenamiento. Para más información sobre las métricas de capacidad almacenadas en la tabla **$MetricsCapacityBlob** , consulte [Esquema de tabla de métricas de Storage Analytics](https://msdn.microsoft.com/library/azure/hh343264.aspx).

> [!NOTE]
> Debe supervisar estos valores para saber con antelación si se está acercando a los límites de capacidad de la cuenta de almacenamiento. En Azure Portal puede agregar reglas de alertas para que le notifiquen si el uso del almacenamiento agregado es superior o inferior a los umbrales que usted mismo especifique.
>
>

Para que le resulte más fácil estimar el tamaño de diversos objetos de almacenamiento, como los BLOB, consulte la entrada de blog sobre facturación del [Azure Storage: ancho de banda, transacciones y capacidad](https://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx).

### <a name="monitoring-availability"></a><a name="monitoring-availability"></a>Supervisión de la disponibilidad
Debe supervisar la disponibilidad de los servicios de almacenamiento de la cuenta de almacenamiento. Para ello, supervise el valor de la columna **Availability** (Disponibilidad) en las tablas de métricas horarias o por minuto: **$MetricsHourPrimaryTransactionsBlob**, **$MetricsHourPrimaryTransactionsTable**, **$MetricsHourPrimaryTransactionsQueue**, **$MetricsMinutePrimaryTransactionsBlob**, **$MetricsMinutePrimaryTransactionsTable**, **$MetricsMinutePrimaryTransactionsQueue**, **$MetricsCapacityBlob**. La columna **Availability** (Disponibilidad) contiene un valor de porcentaje que indica la disponibilidad del servicio o de la operación de API representada por la fila (**RowKey** señala si la fila contiene métricas del servicio completo o de una operación de API específica).

Si el valor es inferior al 100%, significa que algunas solicitudes de almacenamiento no se están realizando correctamente. Puede ver a qué se deben los errores examinando las demás columnas de los datos de métricas, que muestran el número de solicitudes con diferentes tipos de errores, como **ServerTimeoutError**. Normalmente, observará que la **disponibilidad** desciende temporalmente por debajo del 100% por motivos como, por ejemplo, que se agotA de forma transitoria el tiempo de espera del servidor mientras el servicio mueve las particiones para equilibrar mejor la carga de las solicitudes. La lógica de reintento de la aplicación cliente debería administrar estas condiciones intermitentes. En el artículo [Storage Analytics Logged Operations and Status Messages](https://msdn.microsoft.com/library/azure/hh343260.aspx) (Operaciones registradas y mensajes de estado de análisis de almacenamiento) se enumeran los tipos de transacciones que las métricas de almacenamiento incluyen en su cálculo de **disponibilidad**.

En [Azure Portal](https://portal.azure.com), puede agregar reglas de alertas para que le notifiquen si la **disponibilidad** de un servicio desciende por debajo del umbral que especifique.

La sección “[Guía de solución de problemas]” de esta guía describe varios problemas habituales de los servicios de almacenamiento relacionados con la disponibilidad.

### <a name="monitoring-performance"></a><a name="monitoring-performance"></a>Supervisión del rendimiento
Para supervisar el rendimiento de los servicios de almacenamiento, puede usar las siguientes métricas de las tablas de métricas horarias y por minuto.

* Los valores de las columnas **AverageE2ELatency** y **AverageServerLatency** muestran el tiempo medio que tarda el tipo de operación de API o servicio de almacenamiento en procesar las solicitudes. **AverageE2ELatency** es una medida de la latencia de un extremo a otro que incluye el tiempo que se tarda en leer la solicitud y enviar la respuesta, además del tiempo que lleva procesar la solicitud (por lo tanto, incluye la latencia de red una vez que la solicitud alcanza el servicio de almacenamiento). **AverageServerLatency** es una medida que incluye solamente el tiempo de procesamiento, es decir, que excluye toda la latencia de red relacionada con la comunicación con el cliente. Consulte la sección "[Las métricas muestran una AverageE2ELatency alta y una AverageServerLatency baja]", más adelante en esta guía, para ver una explicación de por qué puede haber una diferencia significativa entre estos dos valores.
* Los valores de las columnas **TotalIngress** y **TotalEgress** muestran la cantidad total de datos, en bytes, que entran en el servicio de almacenamiento o salen de él, o que atraviesan un tipo de operación de API específico.
* Los valores de la columna **TotalRequests** muestran el número total de solicitudes que está recibiendo la operación de API o el servicio de almacenamiento. **TotalRequests** es el número total de solicitudes que recibe el servicio de almacenamiento.

Normalmente, deberá supervisar estos valores para ver si se produce algún cambio inesperado: son indicadores de que existe un error que se tiene que investigar.

[Azure Portal](https://portal.azure.com), puede agregar reglas de alertas para que le notifiquen si alguna de las métricas de rendimiento de este servicio llega a ser inferior o superior al umbral que especifique.

La sección “[Guía de solución de problemas]” de esta guía describe varios problemas habituales de los servicios de almacenamiento relacionados con el rendimiento.

## <a name="diagnosing-storage-issues"></a><a name="diagnosing-storage-issues"></a>Diagnóstico de problemas de almacenamiento
Hay varias maneras de descubrir que existe un problema en la aplicación, entre las que se incluyen las siguientes:

* Un error grave que hace que la aplicación se bloquee o deje de funcionar.
* Cambios significativos respecto de los valores de línea base que está supervisando, como se describe en la sección anterior, “[Supervisión del servicio de almacenamiento]”.
* Informes de usuarios de la aplicación que comuniquen que alguna operación concreta no se completó como se esperaba o que alguna característica no está funcionando.
* Errores generados dentro de la aplicación que aparecen en los archivos de registro o en algún otro método de notificación.

Normalmente, los problemas relacionados con los servicios de almacenamiento de Azure entran en una de estas cuatro categorías generales:

* La aplicación tiene un problema de rendimiento, del que informan los usuarios o revelado por cambios en las métricas de rendimiento.
* Hay un problema relacionado con la infraestructura de Azure Storage en una o varias regiones.
* Se produce un error en la aplicación, del que informan los usuarios o revelado por un aumento en una de las métricas de recuento de errores que supervisa.
* Durante el desarrollo y las pruebas, puede que esté usando el emulador de almacenamiento local. Es posible que observe algunos problemas relacionados específicamente con el uso del emulador de almacenamiento.

En las siguientes secciones, se describen los pasos que debe seguir para diagnosticar y resolver los problemas de cada una de estas cuatro categorías. La sección “[Guía de solución de problemas]”, más adelante en esta misma guía, proporciona más detalles sobre algunos problemas habituales que puede encontrarse.

### <a name="service-health-issues"></a><a name="service-health-issues"></a>Problemas de estado del servicio
Los problemas de estado del servicio suelen estar fuera de su control. [Azure Portal](https://portal.azure.com) proporciona información sobre todos los problemas que se estén produciendo en los servicios de Azure, incluidos los servicios de almacenamiento. Si eligió el almacenamiento con redundancia geográfica con acceso de lectura al crear la cuenta de almacenamiento, si los datos dejaran de estar disponibles en la ubicación principal, la aplicación puede cambiar temporalmente a la copia de solo lectura de la ubicación secundaria. Para leer de la ubicación secundaria, la aplicación debe ser capaz de cambiar entre las ubicaciones principal y secundaria, y debe poder funcionar en un modo de funcionalidad reducida con datos de solo lectura. Las bibliotecas de cliente de Azure Storage le permiten definir una directiva de reintentos que pueda leer el almacenamiento secundario si se produce un error al leer el almacenamiento principal. Además, la aplicación tiene que saber que los datos de la ubicación secundaria tienen coherencia final. Para más información, consulte la entrada de blog sobre [Azure Storage Redundancy Options and Read Access Geo Redundant Storage](https://blogs.msdn.microsoft.com/windowsazurestorage/2013/12/11/windows-azure-storage-redundancy-options-and-read-access-geo-redundant-storage/).

### <a name="performance-issues"></a><a name="performance-issues"></a>Problemas de rendimiento
El rendimiento de una aplicación puede ser subjetivo, especialmente desde el punto de vista de los usuarios. Por ello, es importante tener métricas de línea base disponibles con las que le resulte más fácil identificar los casos en los que puede haber un problema de rendimiento. Hay muchos factores que pueden afectar al rendimiento de un servicio de almacenamiento de Azure desde la perspectiva de la aplicación cliente. Estos factores pueden operar en el servicio de almacenamiento, en el cliente o en la infraestructura de red: por eso, es importante disponer de una estrategia para identificar el origen del problema de rendimiento.

Después de identificar, a partir de las métricas, la ubicación donde es más probable que se encuentre el motivo del problema de rendimiento, puede usar los archivos de registro para buscar información detallada que le permita seguir diagnosticando y solucionando el problema.

La sección “[Guía de solución de problemas]”, más adelante en esta misma guía, proporciona más información sobre algunos problemas habituales relacionados con el rendimiento que puede encontrarse.

### <a name="diagnosing-errors"></a><a name="diagnosing-errors"></a>Diagnóstico de errores
Puede que los usuarios de la aplicación le notifiquen los errores de los que informa la aplicación cliente. Las métricas de Almacenamiento también registran los recuentos de diferentes tipos de errores de los servicios de almacenamiento, como **NetworkError**, **ClientTimeoutError** o **AuthorizationError**. Las métricas de Almacenamiento solamente registran los recuentos de diversos tipos de errores, pero puede obtener más detalles sobre cada una de las solicitudes examinando los registros del lado servidor, del lado cliente y de la red. Normalmente, el código de estado HTTP que devuelva el servicio de almacenamiento aportará una indicación del motivo de error en la solicitud.

> [!NOTE]
> Recuerde que cabe esperar algunos errores intermitentes: por ejemplo, errores debidos a condiciones transitorias de la red o errores de aplicación.
>
>

Los siguientes recursos resultan útiles para entender los códigos de error y estado relacionados con el almacenamiento:

* [Códigos de error comunes de la API de REST](https://msdn.microsoft.com/library/azure/dd179357.aspx)
* [Códigos de error del servicio BLOB](https://msdn.microsoft.com/library/azure/dd179439.aspx)
* [Códigos de error del servicio Cola](https://msdn.microsoft.com/library/azure/dd179446.aspx)
* [Códigos de error del servicio Tabla](https://msdn.microsoft.com/library/azure/dd179438.aspx)
* [Códigos de error del servicio Archivos](https://msdn.microsoft.com/library/azure/dn690119.aspx)

### <a name="storage-emulator-issues"></a><a name="storage-emulator-issues"></a>Problemas del emulador de almacenamiento
El SDK de Azure incluye un emulador de almacenamiento que puede ejecutar en una estación de trabajo de desarrollo. Este emulador simula la mayor parte del comportamiento de los servicios de Azure Storage y resulta útil durante las fases de desarrollo y prueba. Le permite ejecutar aplicaciones que usan servicios de Azure Storage sin necesidad de una suscripción de Azure ni una cuenta de almacenamiento de Azure.

La sección "[Guía de solución de problemas]" de esta guía describe varios problemas habituales que se producen con el emulador de almacenamiento.

### <a name="storage-logging-tools"></a><a name="storage-logging-tools"></a>Herramientas de registro de almacenamiento
El registro de Storage proporciona el registro del lado servidor de las solicitudes de almacenamiento de la cuenta de Azure Storage. Para más información sobre cómo habilitar el registro del lado servidor y acceder a los datos de registro, consulte [Habilitación del registro de almacenamiento y acceso a los datos de registro](https://go.microsoft.com/fwlink/?LinkId=510867).

La biblioteca de cliente de Almacenamiento para .NET le permite recopilar datos de registro del lado cliente relacionados con las operaciones de almacenamiento que realiza la aplicación. Para obtener más información, consulte [Registro de cliente con biblioteca de cliente de almacenamiento .NET](https://go.microsoft.com/fwlink/?LinkId=510868).

> [!NOTE]
> En algunas circunstancias (por ejemplo, en el caso de los errores de autorización de SAS), es posible que un usuario informe de un error y que usted no encuentre datos de la solicitud correspondiente en los registros de Almacenamiento del lado servidor. Puede utilizar las funcionalidades de registro de la biblioteca de cliente de Almacenamiento para investigar si la causa del problema se encuentra en el cliente, o usar herramientas de supervisión de red para investigar la red.
>
>

### <a name="using-network-logging-tools"></a><a name="using-network-logging-tools"></a>Uso de herramientas de registro de red
Puede capturar el tráfico entre el cliente y el servidor para proporcionar información detallada sobre los datos que están intercambiando el cliente y el servidor y sobre las condiciones de la red subyacente. Algunas herramientas útiles de redes son:

* [Fiddler](https://www.telerik.com/fiddler) es un proxy de depuración web gratuito que permite examinar los encabezados y los datos de carga útil de los mensajes de solicitud y respuesta HTTP y HTTPS. Para más información, consulte el [Apéndice 1: Uso de Fiddler para capturar tráfico HTTP y HTTPS](#appendix-1).
* [Microsoft Network Monitor (Netmon)](https://www.microsoft.com/download/details.aspx?id=4865) y [Wireshark](https://www.wireshark.org/) son analizadores de protocolos de red gratuitos que le permiten ver información detallada del paquete de una amplia variedad de protocolos de red. Para más información sobre Wireshark, consulte el "[Apéndice 2: Uso de Wireshark para capturar tráfico de red](#appendix-2)".
* El Analizador de mensajes de Microsoft es una herramienta de Microsoft que sustituye a Netmon y que, además de capturar los datos de los paquetes de red, lo ayuda a ver y analizar los datos de registro capturados con otras herramientas. Para más información, consulte el "[Apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red](#appendix-3)".
* Si quiere realizar una prueba de conectividad básica para comprobar que el equipo cliente puede conectarse al servicio de almacenamiento de Azure a través de la red, no puede hacerlo con la herramienta de **ping** estándar que hay en el cliente. Sin embargo, puede usar la herramienta [**tcping**](https://www.elifulkerson.com/projects/tcping.php) para comprobar la conectividad.

En muchos casos, los datos de registro de Almacenamiento y de la biblioteca de cliente de Almacenamiento bastarán para diagnosticar un problema, pero en algunos escenarios, puede que necesite información más detallada de la que pueden proporcionar estas herramientas de registro de red. Por ejemplo, si usa Fiddler para ver los mensajes HTTP y HTTPS, puede ver los datos de encabezados y cargas que se envían a los servicios de almacenamiento y desde ellos. Con esta información, puede examinar la manera en la que una aplicación cliente reintenta las operaciones de almacenamiento. Los analizadores de protocolos, como Wireshark, operan en el nivel de paquetes, por lo que le permiten ver datos de TCP. Con esta información, podría solucionar los problemas de conectividad y pérdida de paquetes. El Analizador de mensajes puede operar en los niveles HTTP y TCP.

## <a name="end-to-end-tracing"></a><a name="end-to-end-tracing"></a>Seguimiento integral
La traza de un extremo a otro con diversos archivos de registro es una técnica útil para investigar problemas potenciales. Puede utilizar la información de fecha y hora de los datos de las métricas como una indicación para saber por dónde empezar a buscar en los archivos de registro la información detallada para solucionar el problema.

### <a name="correlating-log-data"></a><a name="correlating-log-data"></a>Correlación de datos de registro
Al ver registros de aplicaciones cliente, seguimientos de red y registros de almacenamiento del lado servidor, es fundamental poder poner en correlación las solicitudes de los diferentes archivos de registro. Los archivos de registro contienen varios campos distintos que resultan útiles como identificadores de correlación. El identificador de solicitud de cliente es el campo más útil que puede usar para poner en correlación las entradas de los diferentes registros. Sin embargo, a veces puede resultar útil emplear el identificador de solicitud de servidor o las marcas de tiempo. En las siguientes secciones, se proporcionan más detalles de estas opciones.

### <a name="client-request-id"></a><a name="client-request-id"></a>Id. de solicitud de cliente
La biblioteca de cliente de Storage genera automáticamente un identificador de solicitud de cliente único para cada solicitud.

* En el registro del lado cliente que crea la biblioteca de cliente de Storage, el identificador de solicitud de cliente aparece en el campo **Id. de solicitud de cliente** de cada registro relacionado con la solicitud.
* En el seguimiento de red como el que captura Fiddler, el identificador de solicitud de cliente se puede ver en los mensajes de solicitud como el valor del encabezado HTTP **x-ms-client-request-id** .
* En el registro del registro de almacenamiento del lado servidor, el identificador de solicitud de cliente aparece en la columna Id. de solicitud de cliente.

> [!NOTE]
> Es posible que varias solicitudes compartan el mismo identificador de solicitud de cliente, porque el cliente puede asignar ese valor (aunque la biblioteca de cliente de Storage asigne un valor nuevo automáticamente). Si el cliente realiza reintentos, todos los intentos comparten el mismo identificador de solicitud de cliente. En el caso de un lote enviado desde el cliente, el lote tiene un identificador de solicitud de cliente único.
>
>

### <a name="server-request-id"></a><a name="server-request-id"></a>Identificador de solicitud de servidor
El servicio de almacenamiento genera automáticamente identificadores de solicitud de servidor.

* En el registro del registro de Storage del lado servidor, el identificador de solicitud de servidor aparece en la columna **Encabezado de id. de solicitud** .
* En el seguimiento de red como el que captura Fiddler, el identificador de solicitud de servidor aparece en los mensajes de respuesta como el valor del encabezado HTTP **x-ms-request-id** .
* En el registro del lado cliente que crea la biblioteca de cliente de Storage, el identificador de solicitud de servidor aparece en la columna **Texto de operación** de la entrada de registro y muestra detalles de la respuesta del servidor.

> [!NOTE]
> El servicio de almacenamiento siempre asigna un identificador de solicitud de servidor único a cada solicitud que recibe, de modo que cada reintento del cliente y cada operación incluida en un lote tiene un identificador de solicitud de servidor único.
>
>

Si la biblioteca de cliente de Almacenamiento inicia una **StorageException** en el cliente, la propiedad **RequestInformation** contendrá un objeto **RequestResult** que incluirá una propiedad **ServiceRequestID**. También puede acceder a los objetos **RequestResult** desde una instancia de **OperationContext**.

El ejemplo de código siguiente muestra cómo establecer un valor de **ClientRequestId** personalizado adjuntando un objeto **OperationContext** a la solicitud que se realiza al servicio de almacenamiento. También muestra cómo recuperar el valor de **ServerRequestId** a partir del mensaje de respuesta.

```csharp
//Parse the connection string for the storage account.
const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Create an Operation Context that includes custom ClientRequestId string based on constants defined within the application along with a Guid.
OperationContext oc = new OperationContext();
oc.ClientRequestID = String.Format("{0} {1} {2} {3}", HOSTNAME, APPNAME, USERID, Guid.NewGuid().ToString());

try
{
    CloudBlobContainer container = blobClient.GetContainerReference("democontainer");
    ICloudBlob blob = container.GetBlobReferenceFromServer("testImage.jpg", null, null, oc);  
    var downloadToPath = string.Format("./{0}", blob.Name);
    using (var fs = File.OpenWrite(downloadToPath))
    {
        blob.DownloadToStream(fs, null, null, oc);
        Console.WriteLine("\t Blob downloaded to file: {0}", downloadToPath);
    }
}
catch (StorageException storageException)
{
    Console.WriteLine("Storage exception {0} occurred", storageException.Message);
    // Multiple results may exist due to client side retry logic - each retried operation will have a unique ServiceRequestId
    foreach (var result in oc.RequestResults)
    {
            Console.WriteLine("HttpStatus: {0}, ServiceRequestId {1}", result.HttpStatusCode, result.ServiceRequestID);
    }
}
```

### <a name="timestamps"></a><a name="timestamps"></a>Marcas de tiempo
También puede usar marcas de tiempo para buscar entradas de registro relacionadas, pero tenga cuidado con los posibles sesgos del reloj que pueda haber entre el cliente y el servidor. Busque en un intervalo de entre 15 minutos menos y 15 minutos más para encontrar entradas coincidentes del lado servidor a partir de la marca de tiempo del cliente. Recuerde que los metadatos de blob de los blobs que contienen métricas indican el intervalo de tiempo de las métricas almacenadas en el blob. Este intervalo de tiempo resulta útil si tiene muchos blobs de métricas para el mismo minuto u hora.

## <a name="troubleshooting-guidance"></a><a name="troubleshooting-guidance"></a>Guía de solución de problemas
Con esta sección, le resultará más fácil encargarse del diagnóstico y la solución de algunos de los problemas habituales que pueden producirse en la aplicación al usar los servicios de almacenamiento de Azure. Utilice la siguiente lista para buscar la información relacionada con el problema específico.

**Árbol de decisión de solución de problemas**

---
¿El problema está relacionado con el rendimiento de uno de los servicios de almacenamiento?

* [Las métricas muestran una AverageE2ELatency alta y una AverageServerLatency baja]
* [Las métricas muestran una AverageE2ELatency baja y una AverageServerLatency baja, pero el cliente tiene latencia alta]
* [Las métricas muestran una AverageServerLatency alta]
* [Observa retrasos inesperados en la entrega de mensajes de una cola]

---
¿El problema está relacionado con la disponibilidad de uno de los servicios de almacenamiento?

* [Las métricas muestran un aumento de PercentThrottlingError]
* [Las métricas muestran un aumento de PercentTimeoutError]
* [Las métricas muestran un aumento de PercentNetworkError]

---
 ¿La aplicación cliente recibe una respuesta HTTP 4XX (por ejemplo, 404) de un servicio de almacenamiento?

* [El cliente recibe mensajes HTTP 403 (prohibido)]
* [El cliente recibe mensajes HTTP 404 (no encontrado)]
* [El cliente recibe mensajes HTTP 409 (conflicto)]

---
[Las métricas muestran un PercentSuccess bajo o las entradas de registro de análisis tienen operaciones con el estado de transacción ClientOtherErrors]

---
[Las métricas de capacidad muestran un aumento inesperado en el uso de la capacidad de almacenamiento]

---
[Se producen reinicios inesperados de instancias de Virtual Machines que tienen muchos VHD adjuntos]

---
[El problema se presenta al usar el emulador de almacenamiento para realizar tareas de desarrollo o pruebas]

---
[Se producen problemas al instalar el SDK de Azure para .NET]

---
[Tiene otro problema distinto relacionado con un servicio de almacenamiento]

---
### <a name="metrics-show-high-averagee2elatency-and-low-averageserverlatency"></a><a name="metrics-show-high-AverageE2ELatency-and-low-AverageServerLatency"></a>Las métricas muestran una AverageE2ELatency alta y una AverageServerLatency baja
La siguiente ilustración de la herramienta de supervisión de [Azure Portal](https://portal.azure.com) muestra un ejemplo en el que el valor de **AverageE2ELatency** es significativamente más alto que el valor de **AverageServerLatency**.

![Ilustración de Azure Portal que muestra un ejemplo en el que el valor de AverageE2ELatency es considerablemente mayor que el de AverageServerLatency.][4]

El servicio de almacenamiento solamente calcula la métrica **AverageE2ELatency** de las solicitudes que se llevan a cabo correctamente y, a diferencia de **AverageServerLatency**, incluye el tiempo que tarda el cliente en enviar los datos y recibir la confirmación del servicio de almacenamiento. Por lo tanto, una diferencia entre **AverageE2ELatency** y **AverageServerLatency** podría deberse a que la aplicación cliente responde con lentitud o al estado de la red.

> [!NOTE]
> También puede ver el valor de **E2ELatency** y **ServerLatency** de cada operación de almacenamiento individual en los datos de registro del registro de almacenamiento.
>
>

#### <a name="investigating-client-performance-issues"></a>Investigación de problemas de rendimiento del cliente
Uno de los posibles motivos de que el cliente responda lentamente es que haya un número limitado de subprocesos o conexiones disponibles o por la lentitud de los recursos, como la CPU, la memoria o el ancho de banda de red. Tal vez pueda resolver el problema modificando el código de cliente para que sea más eficiente (por ejemplo, con llamadas asincrónicas al servicio de almacenamiento) o con una máquina virtual mayor (con más núcleos y más memoria).

Para los servicios de tablas y colas, el algoritmo de Nagle también puede provocar un valor alto de **AverageE2ELatency** en comparación con el de **AverageServerLatency**: para más información, consulte la entrada [Nagle’s Algorithm is Not Friendly towards Small Requests](https://docs.microsoft.com/archive/blogs/windowsazurestorage/nagles-algorithm-is-not-friendly-towards-small-requests) (El algoritmo de Nagle no responde bien a las solicitudes pequeñas). Puede deshabilitar el algoritmo Nagle en el código utilizando la clase **ServicePointManager** en el espacio de nombres **System.Net**. Esto debe hacerlo antes de realizar ninguna llamada a los servicios Tabla o Cola en la aplicación, dado que esto no afecta a las conexiones que ya están abiertas. El siguiente ejemplo proviene del método **Application_Start** de un rol de trabajo.

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);
ServicePoint tableServicePoint = ServicePointManager.FindServicePoint(storageAccount.TableEndpoint);
tableServicePoint.UseNagleAlgorithm = false;
ServicePoint queueServicePoint = ServicePointManager.FindServicePoint(storageAccount.QueueEndpoint);
queueServicePoint.UseNagleAlgorithm = false;
```

Para ver cuántas solicitudes envía su aplicación cliente y comprobar la existencia de cuellos de botella relacionados con el rendimiento general de .NET en el cliente, como CPU, recolección de elementos no utilizados de .NET, utilización de red o memoria, debe comprobar los registros del lado cliente. Como punto de partida para solucionar los problemas de las aplicaciones cliente .NET, consulte [Debugging, Tracing, and Profiling](https://msdn.microsoft.com/library/7fe0dd2y)(Depuración, seguimiento y generación de perfiles).

#### <a name="investigating-network-latency-issues"></a>Investigación de problemas de latencia de red
Normalmente, la latencia de un extremo a otro provocada por la red se debe a estados transitorios. Puede investigar los problemas de red tanto transitorios como persistentes, como los paquetes descartados, con herramientas como Wireshark o el Analizador de mensajes de Microsoft.

Para más información sobre el uso de Wireshark para solucionar problemas de red, consulte el "[Apéndice 2: Uso de Wireshark para capturar tráfico de red]".

Para más información sobre cómo usar Microsoft Message Analyzer para solucionar problemas de red, consulte el "[Apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red]".

### <a name="metrics-show-low-averagee2elatency-and-low-averageserverlatency-but-the-client-is-experiencing-high-latency"></a><a name="metrics-show-low-AverageE2ELatency-and-low-AverageServerLatency"></a>Las métricas muestran una AverageE2ELatency baja y una AverageServerLatency baja, pero el cliente tiene latencia alta
En este escenario, la causa más probable es un retraso en las solicitudes de almacenamiento que llegan al servicio de almacenamiento. Debe investigar por qué las solicitudes del cliente no llegan hasta el servicio BLOB.

Un posible motivo de que el cliente envíe las solicitudes con retraso es que haya un número limitado de subprocesos o conexiones disponibles.

Compruebe también si el cliente está realizando varios reintentos y, si es así, investigue la causa. Para determinar si el cliente está realizando varios reintentos, puede:

* Examinar los registros de Storage Analytics. Si se producen varios reintentos, verá varias operaciones con el mismo identificador de solicitud de cliente, pero con identificadores de solicitud de servidor diferentes.
* Examinar los registros de cliente. Los registros detallados indicarán que se ha producido un reintento.
* Depurar el código y comprobar las propiedades del objeto **OperationContext** asociado a la solicitud. Si se vuelve a intentar la operación, la propiedad **RequestResults** incluirá varios identificadores de solicitud de servidor exclusivos. También puede comprobar los tiempos de inicio y finalización de cada solicitud. Para más información, consulte el ejemplo de código de la sección [Identificador de solicitud de servidor].

Si no hay ningún problema en el cliente, debe investigar los problemas de red potenciales, como la pérdida de paquetes. Puede emplear herramientas como Wireshark o el Analizador de mensajes de Microsoft para investigar los problemas de red.

Para más información sobre el uso de Wireshark para solucionar problemas de red, consulte el "[Apéndice 2: Uso de Wireshark para capturar tráfico de red]".

Para más información sobre cómo usar Microsoft Message Analyzer para solucionar problemas de red, consulte el "[Apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red]".

### <a name="metrics-show-high-averageserverlatency"></a><a name="metrics-show-high-AverageServerLatency"></a>Las métricas muestran una AverageServerLatency alta
En el caso de que la **AverageServerLatency** de las solicitudes de descarga de BLOB sea alta, debe usar los registros del registro de Almacenamiento para ver si se repiten varias solicitudes del mismo BLOB (o conjunto de BLOB). Con las solicitudes de carga de blobs, debe investigar qué tamaño de bloques está usando el cliente (por ejemplo, los bloques con un tamaño inferior a 64 K pueden provocar sobrecargas, salvo que las lecturas se realicen también en fragmentos de menos de 64 K), y si hay varios clientes que están cargando bloques al mismo blob en paralelo. Además, también debe comprobar las métricas por minuto para ver si, en el número de solicitudes, hay picos que hacen que se superen los objetivos de escalabilidad por segundo. Consulte también "[Las métricas muestran un aumento de PercentTimeoutError]".

Si observa que la **AverageServerLatency** de las solicitudes de descarga de blobs es alta cuando se repiten las solicitudes del mismo blob o conjunto de blobs, debe plantearse la posibilidad de copiar en caché esos blobs con Azure Cache o Azure Content Delivery Network (CDN). Con las solicitudes de carga, puede mejorar el rendimiento utilizando un tamaño de bloques mayor. Para realizar consultas a tablas, también se puede implementar el almacenamiento en caché del lado cliente en los clientes que realizan las mismas operaciones de consulta y cuando los datos no suelen cambiar.

Si los valores de **AverageServerLatency** son altos, esto también puede ser síntoma de un diseño poco adecuado de las tablas o las consultas que genere operaciones de examen o que siga el antipatrón anexar/anteponer. Consulte “[Las métricas muestran un aumento de PercentThrottlingError]” para más información.

> [!NOTE]
> Puede encontrar aquí una lista de comprobación completa del rendimiento: [Lista de comprobación de rendimiento y escalabilidad de Microsoft Azure Storage](storage-performance-checklist.md).
>
>

### <a name="you-are-experiencing-unexpected-delays-in-message-delivery-on-a-queue"></a><a name="you-are-experiencing-unexpected-delays-in-message-delivery"></a>Observa retrasos inesperados en la entrega de mensajes de una cola
Si observa un retraso entre el momento en el que una aplicación agrega un mensaje a una cola y el momento en el que pasa a estar disponible para su lectura en la cola, debe seguir estos pasos para diagnosticar el problema:

* Compruebe que la aplicación está agregando los mensajes a la cola correctamente. Compruebe que la aplicación no reintenta el método **AddMessage** varias veces antes de llevar a cabo la operación correctamente. Los registros de la biblioteca de cliente de Almacenamiento muestran todos los reintentos repetidos de las operaciones de almacenamiento.
* Compruebe que no hay ningún sesgo del reloj entre el rol de trabajo que agrega el mensaje a la cola y el rol de trabajo que lee el mensaje de la cola. Este sesgo del reloj podría hacer que pareciera que hay un retraso en el procesamiento.
* Compruebe si el rol de trabajo que lee los mensajes de la cola no está funcionando correctamente. Si el cliente de una cola llama al método **GetMessage** pero no responde correctamente con una confirmación, el mensaje seguirá sin aparecer en la cola hasta que expire el período de **invisibilityTimeout**. En ese momento, el mensaje volverá a estar disponible para el procesamiento.
* Compruebe si la longitud de la cola aumenta con el tiempo. Esto puede ocurrir si no tiene suficientes trabajadores disponibles para procesar todos los mensajes que colocan en la cola los demás trabajadores. Además, compruebe las métricas para observar si las solicitudes de eliminación no se están realizando correctamente, y el recuento de elementos quitados de la cola de los mensajes, que puede señalar que se realizaron varios intentos incorrectos de eliminar el mensaje.
* Examine los registros del registro de Almacenamiento para ver si hay operaciones de la cola que tengan valores de **E2ELatency** y **ServerLatency** superiores a los que se esperaban durante un período de tiempo más largo de lo habitual.

### <a name="metrics-show-an-increase-in-percentthrottlingerror"></a><a name="metrics-show-an-increase-in-PercentThrottlingError"></a>Las métricas muestran un aumento de PercentThrottlingError
Los errores de limitación se producen cuando supera los objetivos de escalabilidad de un servicio de almacenamiento. El servicio de almacenamiento aplica la limitación para asegurarse de que ningún cliente o inquilino pueda usar el servicio a expensas de otros. Para obtener más información, consulte [Objetivos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](scalability-targets-standard-account.md) para obtener detalles sobre los objetivos de escalabilidad de las cuentas de almacenamiento y los objetivos de rendimiento de las particiones que hay dentro de las cuentas de almacenamiento.

Si la métrica **PercentThrottlingError** muestra un aumento en el porcentaje de solicitudes que tienen errores de limitación, debe investigar uno de estos dos escenarios:

* [Aumento transitorio de PercentThrottlingError]
* [Aumento permanente del error PercentThrottlingError]

Los aumentos de **PercentThrottlingError** suelen ocurrir al mismo tiempo que los aumentos en el número de solicitudes de almacenamiento, o al realizar las primeras pruebas de carga de la aplicación. Esto también puede manifestarse en el cliente como los mensajes de estado HTTP “503 Servidor ocupado” o “500 Se agotó el tiempo de espera de la operación” de las operaciones de almacenamiento.

#### <a name="transient-increase-in-percentthrottlingerror"></a><a name="transient-increase-in-PercentThrottlingError"></a>Aumento transitorio de PercentThrottlingError
Si observa picos en el valor de **PercentThrottlingError** que coinciden con los períodos de gran actividad de la aplicación, debe implementar una estrategia de retroceso exponencial (no lineal) de reintentos en el cliente. Esto reducirá la carga inmediata de la partición y ayudará a la aplicación a suavizar los picos de tráfico. Para más información sobre cómo implementar directivas de reintentos mediante la biblioteca cliente de almacenamiento, consulte el tema sobre el espacio de nombres [Microsoft.Azure.Storage.RetryPolicies](/dotnet/api/microsoft.azure.storage.retrypolicies).

> [!NOTE]
> Puede que también observe picos en el valor de **PercentThrottlingError** que no coincidan con períodos de gran actividad de la aplicación: en ese caso, la causa más probable es que el servicio de almacenamiento desplace particiones para mejorar el equilibrio de carga.
>
>

#### <a name="permanent-increase-in-percentthrottlingerror-error"></a><a name="permanent-increase-in-PercentThrottlingError"></a>Aumento permanente del error PercentThrottlingError
Si observa que el valor de **PercentThrottlingError** es alto de forma constante después de un aumento permanente en los volúmenes de transacciones, o al realizar las primeras pruebas de carga de la aplicación, tiene que evaluar cómo está usando la aplicación las particiones de almacenamiento, y si se está acercando a los objetivos de escalabilidad de una cuenta de almacenamiento. Por ejemplo, si observa errores de limitación en una cola (que cuenta como una sola partición), debe considerar la posibilidad de usar colas adicionales para repartir las transacciones por varias particiones. Si observa errores de limitación en una tabla, tiene que plantearse la opción de usar otro esquema de particiones para repartir las transacciones por varias particiones con una variedad más amplia de valores de clave de partición. Una causa habitual de este problema es el antipatrón anexar/anteponer, con el que se selecciona la fecha como la clave de partición y, luego, todos los datos de un determinado día se escriben en una sola partición: en condiciones de carga, esto puede provocar un cuello de botella de escritura. Considere la posibilidad de usar otro diseño de particiones o evalúe si usar el almacenamiento de blobs puede ser una solución más adecuada. Compruebe también si la limitación se produce debido a picos en el tráfico e investigue posibles formas de suavizar el patrón de solicitudes.

Si distribuye las transacciones por varias particiones, aun así, debe ser consciente de los límites de escalabilidad establecidos para la cuenta de almacenamiento. Por ejemplo, si utilizó diez colas y cada una de ellas procesa la cantidad máxima (2.000) de mensajes de 1 KB por segundo, alcanzará el límite global de 20.000 mensajes por segundo en la cuenta de almacenamiento. Si necesita procesar más de 20.000 entidades por segundo, debe plantearse la posibilidad de usar varias cuentas de almacenamiento. También debe recordar que el tamaño de las solicitudes y las entidades influye sobre el momento en el que el servicio de almacenamiento limita los clientes: si tiene solicitudes y entidades de gran tamaño, es posible que se le limite antes.

Un diseño ineficiente de las consultas también puede hacer que alcance los límites de escalabilidad de las particiones de tabla. Por ejemplo, una consulta con un filtro que solo selecciona un uno por ciento de las entidades de una partición, pero que examina todas las entidades de una partición, tendrá que acceder a cada una de las entidades. Cada lectura de una entidad contará de cara al número total de transacciones de esa partición: por este motivo, puede alcanzar fácilmente los objetivos de escalabilidad.

> [!NOTE]
> Las pruebas de rendimiento deberían revelar todos los diseños de consulta ineficientes de la aplicación.
>
>

### <a name="metrics-show-an-increase-in-percenttimeouterror"></a><a name="metrics-show-an-increase-in-PercentTimeoutError"></a>Las métricas muestran un aumento de PercentTimeoutError
Las métricas muestran un aumento de **PercentTimeoutError** en uno de los servicios de almacenamiento. Al mismo tiempo, el cliente recibe un gran volumen de mensajes de estado HTTP “500 Se agotó el tiempo de espera de la operación” de las operaciones de almacenamiento.

> [!NOTE]
> Puede que, temporalmente, vea errores de tiempo de espera, dado que el servicio de almacenamiento equilibra la carga de las solicitudes moviendo particiones a servidores nuevos.
>
>

La métrica **PercentTimeoutError** es una agregación de las siguientes métricas: **ClientTimeoutError**, **AnonymousClientTimeoutError**, **SASClientTimeoutError**, **ServerTimeoutError**, **AnonymousServerTimeoutError** y **SASServerTimeoutError**.

Los errores de tiempo de espera del servidor son errores del servidor. Los errores de tiempo de espera del cliente suceden porque una operación que se lleva a cabo en el servidor superó el tiempo de espera especificado por el cliente. Por ejemplo, un cliente que utiliza la biblioteca de cliente de Almacenamiento puede establecer un tiempo de espera de una operación con la propiedad **ServerTimeout** de la clase **QueueRequestOptions**.

Los errores de tiempo de espera del servidor indican que existe un problema con el servicio de almacenamiento que se debe investigar más. Puede utilizar las métricas para ver si está alcanzando los límites de escalabilidad del servicio y para identificar los picos de tráfico que puedan estar provocando este problema. Si el problema es intermitente, puede que se deba a la actividad de equilibrio de carga del servicio. Si el problema es persistente y no se debe a que la aplicación alcance los límites de escalabilidad del servicio, debe informar al soporte técnico de que existe un problema. Si se producen errores de tiempo de espera del cliente, debe decidir si el tiempo de espera está configurado con un valor adecuado en el cliente y cambiar el valor del tiempo de espera establecido en el cliente o investigar cómo puede mejorar el rendimiento de las operaciones en el servicio de almacenamiento. Por ejemplo, puede optimizar las consultas de tabla o reducir el tamaño de los mensajes.

### <a name="metrics-show-an-increase-in-percentnetworkerror"></a><a name="metrics-show-an-increase-in-PercentNetworkError"></a>Las métricas muestran un aumento de PercentNetworkError
Las métricas muestran un aumento de **PercentNetworkError** en uno de los servicios de almacenamiento. La métrica **PercentNetworkError** es la suma de las siguientes métricas: **NetworkError**, **AnonymousNetworkError** y **SASNetworkError**. Se producen cuando el servicio de almacenamiento detecta un error de red al realizar una solicitud de almacenamiento el cliente.

El motivo más común de este error es que un cliente se desconecte antes de que expire un tiempo de espera en el servicio de almacenamiento. Investigue el código en el cliente para comprender por qué y cuándo se desconecta el cliente del servicio de almacenamiento. También puede utilizar Wireshark, el Analizador de mensajes de Microsoft o Tcping para investigar los problemas de conectividad de red desde el cliente. Estas herramientas se describen en los [Apéndices].

### <a name="the-client-is-receiving-http-403-forbidden-messages"></a><a name="the-client-is-receiving-403-messages"></a>El cliente recibe mensajes HTTP 403 (prohibido)
Si la aplicación cliente inicia errores HTTP 403 (prohibido), uno de los motivos más probables es que el cliente esté usando una Firma de acceso compartido (SAS) expirada al enviar una solicitud de almacenamiento (aunque hay otras causas posibles, como un sesgo del reloj, que las claves no sean válidas o que los encabezados estén vacíos). Si el motivo es que la clave SAS expiró, no verá ninguna entrada en los datos de registro del registro de Almacenamiento del lado servidor. En la siguiente tabla, puede ver una muestra del registro del lado cliente generado por la biblioteca de cliente de Almacenamiento que ilustra este problema:

| Source | Nivel de detalle | Nivel de detalle | Id. de solicitud de cliente | Texto de operación |
| --- | --- | --- | --- | --- |
| Microsoft.Azure.Storage |Información |3 |85d077ab-… |Iniciando operación con ubicación Primary según modo de ubicación PrimaryOnly. |
| Microsoft.Azure.Storage |Información |3 |85d077ab -… |Iniciando solicitud sincrónica a <https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Synchronous_and_Asynchronous_Requests#Synchronous_request>. |
| Microsoft.Azure.Storage |Información |3 |85d077ab -… |Esperando respuesta. |
| Microsoft.Azure.Storage |Advertencia |2 |85d077ab -… |Excepción que se produce mientras se espera una respuesta: Error en el servidor remoto: 403 Prohibido. |
| Microsoft.Azure.Storage |Información |3 |85d077ab -… |Respuesta recibida. Código de estado = 403, Id. de solicitud = 9d67c64a-64ed-4b0d-9515-3b14bbcdc63d, Content-MD5 = , ETag = . |
| Microsoft.Azure.Storage |Advertencia |2 |85d077ab -… |Excepción producida durante la operación: Error en el servidor remoto: 403 Prohibido. |
| Microsoft.Azure.Storage |Información |3 |85d077ab -… |Comprobando si se debe reintentar la operación. Número de reintentos = 0, Código de estado HTTP = 403, Excepción = El servidor remoto devolvió un error: 403 Prohibido. |
| Microsoft.Azure.Storage |Información |3 |85d077ab -… |La siguiente ubicación se estableció como Primary, de acuerdo con el modo de ubicación. |
| Microsoft.Azure.Storage |Error |1 |85d077ab -… |La directiva de reintentos no permitió un reintento. Error: El servidor remoto devolvió un error: 403 Prohibido. |

En este escenario, debe investigar el motivo por el que el token de SAS expira antes de que el cliente envíe el token al servidor:

* Normalmente, cuando crea una SAS para que la use un cliente inmediatamente, no debe establecer la hora de inicio. Si existen pequeñas diferencias entre el reloj del host que genera la SAS y que usa la hora actual, por una parte, y el del servicio de almacenamiento, por otra, es posible que el servicio de almacenamiento reciba una SAS que todavía no es válida.
* En las SAS, no establezca tiempos de expiración muy breves. También en este caso, si existen pequeñas diferencias entre el reloj del host que genera la SAS y el del servicio de almacenamiento, puede que la SAS expire, aparentemente, antes de lo que se esperaba.
* ¿El parámetro de versión de la clave SAS (por ejemplo, **sv=2015-04-05**) coincide con la versión de la biblioteca de cliente de Almacenamiento que está utilizando? Recomendamos usar la última versión de la [biblioteca de cliente de Almacenamiento](https://www.nuget.org/packages/WindowsAzure.Storage/).
* Si vuelve a generar las claves de acceso de almacenamiento, esto puede invalidar todos los token de SAS existentes. Este problema puede producirse si genera tokens de SAS con un tiempo de expiración duradero para que los copien en caché las aplicaciones cliente.

Si utiliza la biblioteca de cliente de Almacenamiento para generar tokens de SAS, es fácil generar un token válido. Sin embargo, si usa la API de REST de Storage y construye manualmente los tokens de SAS, consulte [Delegar el acceso con una firma de acceso compartido](https://msdn.microsoft.com/library/azure/ee395415.aspx).

### <a name="the-client-is-receiving-http-404-not-found-messages"></a><a name="the-client-is-receiving-404-messages"></a>El cliente recibe mensajes HTTP 404 (no encontrado)
Si la aplicación cliente recibe un mensaje HTTP 404 (no encontrado) del servidor, esto significa que el objeto que estaba tratando de usar el cliente (por ejemplo, una entidad, una tabla, un BLOB, un contenedor o una cola) no existe en el servicio de almacenamiento. Hay varios motivos posibles, por ejemplo:

* [El cliente u otro proceso eliminaron anteriormente el objeto]
* [Un problema de autorización de Firma de acceso compartido (SAS)]
* [El código JavaScript del lado cliente no tiene permiso para acceder al objeto]
* [Error de red]

#### <a name="the-client-or-another-process-previously-deleted-the-object"></a><a name="client-previously-deleted-the-object"></a>El cliente u otro proceso eliminaron anteriormente el objeto
En los escenarios en los que el cliente está tratando de leer, actualizar o eliminar datos de un servicio de almacenamiento, suele ser fácil identificar en los registros del lado servidor una operación anterior que eliminara el objeto en cuestión del servicio de almacenamiento. A menudo, los datos de registro indican que otro usuario u otro proceso eliminaron el objeto. En el registro de Storage del lado servidor, las columnas operation-type y requested-object-key muestran el momento en el que un cliente eliminó un objeto.

En el escenario en el que un cliente está tratando de insertar un objeto, puede que no sea evidente de inmediato por qué esto provoca una respuesta HTTP 404 (no encontrado), dado que el cliente está creando un objeto. Sin embargo, si el cliente está creando un BLOB, debe poder encontrar el contenedor de BLOB. Si el cliente está creando un mensaje, debe poder encontrar una cola. Y si el cliente está agregando una fila, debe poder encontrar la tabla.

Puede usar el registro del lado cliente de la biblioteca de cliente de Almacenamiento para comprender de forma más detallada cuándo envía el cliente determinadas solicitudes al servicio de almacenamiento.

El siguiente registro del lado cliente generado por la biblioteca de cliente de Almacenamiento ilustra el problema que aparece cuando el cliente no puede encontrar el contenedor del BLOB que está creando. Este registro incluye detalles de las siguientes operaciones de almacenamiento:

| Id. de solicitud | Operación |
| --- | --- |
| 07b26a5d-... |**DeleteIfExists** para eliminar el contenedor de BLOB. Tenga en cuenta que esta operación incluye una solicitud **HEAD** para comprobar la existencia del contenedor. |
| e2d06d78… |**CreateIfNotExists** para crear el contenedor de BLOB. Tenga en cuenta que esta operación incluye una solicitud **HEAD** que comprueba la existencia del contenedor. **HEAD** devuelve un mensaje 404 pero continúa. |
| de8b1c3c-... |**UploadFromStream** para crear el blob. La solicitud **PUT** devuelve un error con un mensaje 404 |

Entradas del registro:

| Id. de solicitud | Texto de operación |
| --- | --- |
| 07b26a5d-... |A partir de una solicitud sincrónica a `https://domemaildist.blob.core.windows.net/azuremmblobcontainer`. |
| 07b26a5d-... |StringToSign = HEAD............x-ms-client-request-id:07b26a5d-....x-ms-date:Tue, 03 Jun 2014 10:33:11 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container. |
| 07b26a5d-... |Esperando respuesta. |
| 07b26a5d-... |Respuesta recibida. Status code = 200, Request ID = eeead849-...Content-MD5 = , ETag =    &quot;0x8D14D2DC63D059B&quot;. |
| 07b26a5d-... |Los encabezados de respuesta se procesaron correctamente y se continuó con el resto de la operación. |
| 07b26a5d-... |Descargando el cuerpo de respuesta. |
| 07b26a5d-... |Operación completada correctamente. |
| 07b26a5d-... |A partir de una solicitud sincrónica a `https://domemaildist.blob.core.windows.net/azuremmblobcontainer`. |
| 07b26a5d-... |StringToSign = DELETE............x-ms-client-request-id:07b26a5d-....x-ms-date:Tue, 03 Jun 2014 10:33:12    GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container. |
| 07b26a5d-... |Esperando respuesta. |
| 07b26a5d-... |Respuesta recibida. Código de estado = 202, Id. de solicitud = 6ab2a4cf-..., Content-MD5 = , ETag = . |
| 07b26a5d-... |Los encabezados de respuesta se procesaron correctamente y se continuó con el resto de la operación. |
| 07b26a5d-... |Descargando el cuerpo de respuesta. |
| 07b26a5d-... |Operación completada correctamente. |
| e2d06d78-... |A partir de una solicitud asincrónica a `https://domemaildist.blob.core.windows.net/azuremmblobcontainer`.</td> |
| e2d06d78-... |StringToSign = HEAD............x-ms-client-request-id:e2d06d78-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container. |
| e2d06d78-... |Esperando respuesta. |
| de8b1c3c-... |A partir de una solicitud sincrónica a `https://domemaildist.blob.core.windows.net/azuremmblobcontainer/blobCreated.txt`. |
| de8b1c3c-... |StringToSign = PUT...64.qCmF+TQLPhq/YYK50mP9ZQ==........x-ms-blob-type:BlockBlob.x-ms-client-request-id:de8b1c3c-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer/blobCreated.txt. |
| de8b1c3c-... |Preparándose para escribir datos de solicitud. |
| e2d06d78-... |Excepción que se produce mientras se espera una respuesta: Error en el servidor remoto: 404 No encontrado. |
| e2d06d78-... |Respuesta recibida. Código de estado = 404, Id. de solicitud = 353ae3bc-..., Content-MD5 = , ETag = . |
| e2d06d78-... |Los encabezados de respuesta se procesaron correctamente y se continuó con el resto de la operación. |
| e2d06d78-... |Descargando el cuerpo de respuesta. |
| e2d06d78-... |Operación completada correctamente. |
| e2d06d78-... |A partir de una solicitud asincrónica a `https://domemaildist.blob.core.windows.net/azuremmblobcontainer`. |
| e2d06d78-... |StringToSign = PUT...0.........x-ms-client-request-id:e2d06d78-....x-ms-date:Tue, 03 Jun 2014 10:33:12 GMT.x-ms-version:2014-02-14./domemaildist/azuremmblobcontainer.restype:container. |
| e2d06d78-... |Esperando respuesta. |
| de8b1c3c-... |Escribiendo datos de solicitud. |
| de8b1c3c-... |Esperando respuesta. |
| e2d06d78-... |Excepción que se produce mientras se espera una respuesta: Error en el servidor remoto: 409 Conflicto. |
| e2d06d78-... |Respuesta recibida. Código de estado = 409, Id. de solicitud = c27da20e-..., Content-MD5 = , ETag = . |
| e2d06d78-... |Descargando el cuerpo de respuesta del error. |
| de8b1c3c-... |Excepción que se produce mientras se espera una respuesta: Error en el servidor remoto: 404 No encontrado. |
| de8b1c3c-... |Respuesta recibida. Código de estado = 404, Id. de solicitud = 0eaeab3e-..., Content-MD5 = , ETag = . |
| de8b1c3c-... |Excepción producida durante la operación: Error en el servidor remoto: 404 No encontrado. |
| de8b1c3c-... |La directiva de reintentos no permitió un reintento. Error: El servidor remoto devolvió un error: 404 No encontrado. |
| e2d06d78-... |La directiva de reintentos no permitió un reintento. Error: El servidor remoto devolvió un error: 409 Conflicto. |

En este ejemplo, el registro muestra que el cliente está intercalando las solicitudes del método **CreateIfNotExists** (identificador de solicitud e2d06d78…) con las solicitudes del método **UploadFromStream** (de8b1c3c-...). Esta intercalación sucede porque la aplicación cliente está invocando a estos métodos de forma asincrónica. Modifique el código asincrónico del cliente para que cree el contenedor antes de tratar de cargar datos en un blob de ese contenedor. Lo ideal es que cree todos los contenedores de antemano.

#### <a name="a-shared-access-signature-sas-authorization-issue"></a><a name="SAS-authorization-issue"></a>Un problema de autorización de Firma de acceso compartido (SAS)
Si la aplicación cliente trata de usar una clave SAS que no incluye los permisos necesarios para realizar la operación, el servicio de almacenamiento devuelve un mensaje HTTP 404 (no encontrado) al cliente. Al mismo tiempo, verá también un valor distinto de cero en **SASAuthorizationError** , en las métricas.

En la siguiente tabla, puede ver una muestra de un mensaje de registro del lado servidor del archivo de registro del registro de Almacenamiento:

| Nombre | Value |
| --- | --- |
| Hora de inicio de la solicitud | 2014-05-30T06:17:48.4473697Z |
| Tipo de operación     | GetBlobProperties            |
| Estado de la solicitud     | SASAuthorizationError        |
| Código de estado HTTP   | 404                            |
| Tipo de autenticación| Sas                          |
| Tipo de servicio       | Blob                         |
| URL de la solicitud         | `https://domemaildist.blob.core.windows.net/azureimblobcontainer/blobCreatedViaSAS.txt` |
| &nbsp;                 |   ?sv=2014-02-14&sr=c&si=mypolicy&sig=XXXXX&;api-version=2014-02-14 |
| Encabezado de identificador de solicitud  | a1f348d5-8032-4912-93ef-b393e5252a3b |
| Id. de solicitud de cliente  | 2d064953-8436-4ee0-aa0c-65cb874f7929 |


Investigue el motivo por el que la aplicación cliente está tratando de realizar una operación para la que no se le concedieron permisos.

#### <a name="client-side-javascript-code-does-not-have-permission-to-access-the-object"></a><a name="JavaScript-code-does-not-have-permission"></a>El código JavaScript del lado cliente no tiene permiso para acceder al objeto
Si utiliza un cliente de JavaScript y el servicio de almacenamiento devuelve mensajes HTTP 404, compruebe si están los siguientes errores de JavaScript en el explorador:

```
SEC7120: Origin http://localhost:56309 not found in Access-Control-Allow-Origin header.
SCRIPT7002: XMLHttpRequest: Network Error 0x80070005, Access is denied.
```

> [!NOTE]
> Puede usar las Herramientas de desarrollo F12 de Internet Explorer para seguir los mensajes que se intercambian entre el explorador y el servicio de almacenamiento al solucionar problemas de JavaScript del lado cliente.
>
>

Estos errores se producen porque el explorador web implementa la restricción de seguridad [directiva del mismo origen](https://www.w3.org/Security/wiki/Same_Origin_Policy) , que impide que una página web llame a una API de un dominio que no sea el dominio del que proviene la página.

Una solución alternativa al problema de JavaScript consiste en configurar el Uso compartido de recursos entre orígenes (CORS) para el servicio de almacenamiento al que está accediendo el cliente. Para más información, consulte [Compatibilidad con Uso compartido de recursos entre orígenes (CORS) para los Servicios de Azure Storage](https://msdn.microsoft.com/library/azure/dn535601.aspx).

El siguiente ejemplo de código muestra cómo configurar el servicio BLOB para permitir que el JavaScript que se ejecuta en el dominio Contoso acceda a un BLOB de su servicio de almacenamiento de BLOB:

```csharp
CloudBlobClient client = new CloudBlobClient(blobEndpoint, new StorageCredentials(accountName, accountKey));
// Set the service properties.
ServiceProperties sp = client.GetServiceProperties();
sp.DefaultServiceVersion = "2013-08-15";
CorsRule cr = new CorsRule();
cr.AllowedHeaders.Add("*");
cr.AllowedMethods = CorsHttpMethods.Get | CorsHttpMethods.Put;
cr.AllowedOrigins.Add("http://www.contoso.com");
cr.ExposedHeaders.Add("x-ms-*");
cr.MaxAgeInSeconds = 5;
sp.Cors.CorsRules.Clear();
sp.Cors.CorsRules.Add(cr);
client.SetServiceProperties(sp);
```

#### <a name="network-failure"></a><a name="network-failure"></a>Error de red
En algunas circunstancias, la pérdida de paquetes de red puede hacer que el servicio de almacenamiento devuelva mensajes HTTP 404 al cliente. Por ejemplo, cuando la aplicación cliente está eliminando una entidad del servicio Tabla, ve que el cliente inicia una excepción de almacenamiento que informa de un mensaje de estado “HTTP 404 (no encontrado)” desde el servicio Tabla. Al investigar la tabla del servicio de almacenamiento de tabla, ve que el servicio sí que eliminó la entidad, como se solicitó.

Los detalles de la excepción del cliente incluyen el identificador de solicitud (7e84f12d...) que asigna Table service para la solicitud: puede usar esta información para localizar los detalles de solicitud en los registros de almacenamiento del lado servidor mediante la búsqueda en la columna **request-id-header** del archivo de registro. También podría utilizar las métricas para identificar cuándo se producen errores como este y, luego, buscar los archivos de registro teniendo en cuenta el momento en el que las métricas registraron este error. Esta entrada de registro muestra que la eliminación no se realizó correctamente y produjo el mensaje de estado “HTTP (404) Otro error del cliente”. La misma entrada de registro incluye también el identificador de solicitud generado por el cliente en la columna **client-request-id** (813ea74f…).

Además, el registro del lado servidor incluye otra entrada con el mismo valor de **client-request-id** (813ea74f…) correspondiente a una operación de eliminación realizada correctamente con la misma entidad y del mismo cliente. Esta operación de eliminación correcta tuvo lugar justo antes de la solicitud de eliminación que no se realizó correctamente.

El motivo más probable de este escenario es que el cliente envió una solicitud de eliminación de la entidad al servicio Tabla, que se llevó a cabo correctamente, pero no recibió ninguna confirmación del servidor (tal vez, debido a un problema de red temporal). Entonces, el cliente reintentó automáticamente la operación (con el mismo **client-request-id**), y este reintento no se realizó correctamente porque la entidad ya estaba eliminada.

Si este problema se produce a menudo, debe investigar por qué el cliente no recibe correctamente las confirmaciones del servicio Tabla. Si el problema es intermitente, debe interceptar el error “HTTP (404) No encontrado” y registrarlo en el cliente, pero permitir que el cliente continúe.

### <a name="the-client-is-receiving-http-409-conflict-messages"></a><a name="the-client-is-receiving-409-messages"></a>El cliente recibe mensajes HTTP 409 (conflicto)
En la tabla siguiente se muestra un extracto del registro del servidor de dos operaciones de cliente: **DeleteIfExists** seguido inmediatamente de **CreateIfNotExists** con el mismo nombre de contenedor de blobs. Cada operación de cliente hace que se envíen dos solicitudes al servidor: primero, una solicitud **GetContainerProperties** para comprobar si existe el contenedor y, luego, la solicitud **DeleteContainer** o **CreateContainer**.

| Timestamp | Operación | Resultado | Nombre del contenedor | Id. de solicitud de cliente |
| --- | --- | --- | --- | --- |
| 05:10:13.7167225 |GetContainerProperties |200 |mmcont |c9f52c89-… |
| 05:10:13.8167325 |DeleteContainer |202 |mmcont |c9f52c89-… |
| 05:10:13.8987407 |GetContainerProperties |404 |mmcont |bc881924-… |
| 05:10:14.2147723 |CreateContainer |409 |mmcont |bc881924-… |

El código de la aplicación cliente elimina y vuelve a crear inmediatamente un contenedor de blobs con el mismo nombre: el método **CreateIfNotExists** (Identificador de solicitud de cliente bc881924-...) devuelve el error HTTP 409 (conflicto). Cuando un cliente elimina contenedores de BLOB, tablas o colas, transcurre un breve período hasta que el nombre vuelve a estar disponible.

La aplicación cliente debe usar nombres de contenedor únicos siempre que cree nuevos contenedores si el patrón eliminar / volver a crear es común.

### <a name="metrics-show-low-percentsuccess-or-analytics-log-entries-have-operations-with-transaction-status-of-clientothererrors"></a><a name="metrics-show-low-percent-success"></a>Las métricas muestran un PercentSuccess bajo o las entradas de registro de análisis tienen operaciones con el estado de transacción ClientOtherErrors
La métrica **PercentSuccess** captura el porcentaje de operaciones que se realizaron correctamente de acuerdo con su código de estado HTTP. Las operaciones con códigos de estado 2XX se cuentan como correctas, mientras que las operaciones con códigos de estado en los intervalos 3XX, 4XX y 5XX se cuentan como incorrectas y disminuyen el valor de la métrica **PercentSuccess** . En los archivos de registro de almacenamiento del servidor, estas operaciones se registran con el estado de transacción **ClientOtherErrors**.

Es importante señalar que estas operaciones se completaron correctamente y, por lo tanto, no afectan a otras métricas como las de disponibilidad. Algunos ejemplos de operaciones que se ejecutan correctamente, pero que pueden provocar códigos de estado HTTP incorrectos:

* **ResourceNotFound** (no encontrado 404), por ejemplo, de una solicitud GET a un BLOB que no existe.
* **ResouceAlreadyExists** (conflicto 409), por ejemplo, de una operación **CreateIfNotExist** en la que el recurso ya existe.
* **ResouceAlreadyExists** (no modificado 304), por ejemplo, de una operación condicional como, entre otras, cuando un cliente envía un valor de **ETag** y un encabezado HTTP **If-None-Match** para solicitar una imagen solamente si se actualizó desde la última operación.

Puede encontrar una lista de códigos de error habituales de la API de REST que devuelven los servicios de almacenamiento en la página [Códigos de error comunes de la API de REST](https://msdn.microsoft.com/library/azure/dd179357.aspx).

### <a name="capacity-metrics-show-an-unexpected-increase-in-storage-capacity-usage"></a><a name="capacity-metrics-show-an-unexpected-increase"></a>Las métricas de capacidad muestran un aumento inesperado en el uso de la capacidad de almacenamiento
Si observa cambios repentinos e inesperados en el uso de la capacidad de su cuenta de almacenamiento, puede investigar los motivos consultando, primero, las métricas de disponibilidad: por ejemplo, un aumento en el número de solicitudes de eliminación que no se realizaron correctamente puede provocar un aumento en la cantidad de almacenamiento de blobs que está usando. Esto se debe a que, posiblemente, las operaciones de limpieza específicas de aplicaciones, que esperaba que estuvieran liberando espacio, no funcionen como se esperaba (por ejemplo, porque los tokens de SAS que se usan para liberar espacio expiraron).

### <a name="your-issue-arises-from-using-the-storage-emulator-for-development-or-test"></a><a name="your-issue-arises-from-using-the-storage-emulator"></a>El problema se presenta al usar el emulador de almacenamiento para realizar tareas de desarrollo o pruebas
Normalmente, lo que ocurre es que utiliza el emulador de almacenamiento durante el desarrollo y las pruebas para no necesitar una cuenta de almacenamiento de Azure. Estos son los problemas más habituales que pueden producirse al utilizar el emulador de almacenamiento:

* [La característica "X" no funciona en el emulador de almacenamiento]
* [Error “El valor de uno de los encabezados HTTP no está en el formato correcto” al usar el emulador de almacenamiento]
* [La ejecución del emulador de almacenamiento requiere privilegios administrativos]

#### <a name="feature-x-is-not-working-in-the-storage-emulator"></a><a name="feature-X-is-not-working"></a>La característica "X" no funciona en el emulador de almacenamiento
El emulador de almacenamiento no admite todas las características de los servicios de almacenamiento de Azure como, por ejemplo, el servicio de archivos. Para más información, consulte [Uso del emulador de Azure Storage para desarrollo y pruebas](storage-use-emulator.md).

Para las características que no admita el emulador de almacenamiento, utilice el servicio de almacenamiento de Azure en la nube.

#### <a name="error-the-value-for-one-of-the-http-headers-is-not-in-the-correct-format-when-using-the-storage-emulator"></a><a name="error-HTTP-header-not-correct-format"></a>Error “El valor de uno de los encabezados HTTP no está en el formato correcto” al usar el emulador de almacenamiento
Está probando la aplicación que utiliza la biblioteca de cliente de Storage con el emulador de almacenamiento local, y las llamadas de métodos como **CreateIfNotExists** producen un error con el mensaje “El valor de uno de los encabezados HTTP no está en el formato correcto”. Esto indica que la versión del emulador de almacenamiento que está usando no admite la versión de la biblioteca de cliente de Almacenamiento que está usando. La biblioteca de cliente de Storage agrega el encabezado **x-ms-version** a todas las solicitudes que realiza. Si el emulador de almacenamiento no reconoce el valor del encabezado **x-ms-version** , rechaza la solicitud.

Puede usar los registros de la biblioteca de cliente de Almacenamiento para ver el valor del **encabezado x-ms-version** que está enviando. También puede ver el valor del **encabezado x-ms-version** si utiliza Fiddler para seguir paso a paso las solicitudes desde la aplicación cliente.

Normalmente, este escenario se produce si instala y usa la última versión de la biblioteca de cliente de Almacenamiento sin actualizar el emulador de almacenamiento. Debe instalar la última versión del emulador de almacenamiento o utilizar almacenamiento en la nube, en lugar del emulador, para el desarrollo y las pruebas.

#### <a name="running-the-storage-emulator-requires-administrative-privileges"></a><a name="storage-emulator-requires-administrative-privileges"></a>La ejecución del emulador de almacenamiento requiere privilegios administrativos
El sistema le pregunta por las credenciales de administrador al ejecutar el emulador de almacenamiento. Esto solo sucede al inicializar el emulador de almacenamiento por primera vez. Una vez inicializado el emulador de almacenamiento, no necesita tener privilegios administrativos para volver a ejecutarlo.

Para más información, consulte [Uso del emulador de Azure Storage para desarrollo y pruebas](storage-use-emulator.md). También puede inicializar el emulador de almacenamiento en Visual Studio, lo que también requiere privilegios administrativos.

### <a name="you-are-encountering-problems-installing-the-azure-sdk-for-net"></a><a name="you-are-encountering-problems-installing-the-Windows-Azure-SDK"></a>Se producen problemas al instalar el SDK de Azure para .NET
Cuando trata de instalar el SDK, se produce un error al tratar de instalar el emulador de almacenamiento en la máquina local. El registro de instalación contiene uno de los siguientes mensajes:

* CAQuietExec:  Error: No se puede acceder a la instancia de SQL
* CAQuietExec:  Error: No se puede crear la base de datos

El motivo es un problema relacionado con la instalación de LocalDB existente. De forma predeterminada, el emulador de almacenamiento utiliza LocalDB para hacer que los datos sean persistentes al simular los servicios de almacenamiento de Azure. Puede restablecer la instancia de LocalDB ejecutando los siguientes comandos en una ventana del símbolo del sistema antes de tratar de instalar el SDK.

```
sqllocaldb stop v11.0
sqllocaldb delete v11.0
delete %USERPROFILE%\WAStorageEmulatorDb3*.*
sqllocaldb create v11.0
```

El comando **delete** quita todos los archivos de base de datos antiguos de las instalaciones anteriores del emulador de almacenamiento.

### <a name="you-have-a-different-issue-with-a-storage-service"></a><a name="you-have-a-different-issue-with-a-storage-service"></a>Tiene otro problema distinto relacionado con un servicio de almacenamiento
Si las secciones de solución de problemas anteriores no contienen el problema que tiene con un servicio de almacenamiento, debe adoptar el siguiente método para diagnosticar y solucionar el problema.

* Compruebe las métricas para ver si hay algún cambio respecto del comportamiento de línea base que espera. A partir de las métricas, es posible que pueda averiguar si el problema es transitorio o permanente y a qué operaciones de almacenamiento afecta el problema.
* Puede utilizar la información de las métricas para buscar en los datos de registro del lado servidor información más detallada sobre los errores que se producen. Con esta información, puede que le resulte más fácil solucionar el problema.
* Si la información de los registros del lado servidor no basta para solucionar el problema correctamente, puede utilizar los registros del lado cliente de la biblioteca de cliente de Almacenamiento para investigar el comportamiento de la aplicación cliente, y herramientas como Fiddler, Wireshark o el Analizador de mensajes de Microsoft para investigar la red.

Para más información sobre el uso de Fiddler, consulte el "[Apéndice 1: Uso de Fiddler para capturar tráfico HTTP y HTTPS]".

Para más información sobre el uso de Wireshark, consulte el "[Apéndice 2: Uso de Wireshark para capturar tráfico de red]".

Para obtener más información sobre el uso de Microsoft Message Analyzer, vea "[apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red]".

## <a name="appendices"></a><a name="appendices"></a>Apéndices
En los apéndices, se describen varias herramientas que pueden resultarle útiles al diagnosticar y solucionar problemas relacionados con Azure Storage (y otros servicios). Estas herramientas no forman parte de Azure Storage y algunas son productos de terceros. Por eso, las herramientas de las que se habla en estos apéndices no se incluyen en ningún contrato de asistencia que pueda tener con Microsoft Azure o Azure Storage. Por este motivo, dentro de su proceso de evaluación, deberá examinar las opciones de asistencia y licencias disponibles que ofrecen los proveedores de estas herramientas.

### <a name="appendix-1-using-fiddler-to-capture-http-and-https-traffic"></a><a name="appendix-1"></a>Apéndice 1: Uso de Fiddler para capturar tráfico HTTP y HTTPS
[Fiddler](https://www.telerik.com/fiddler) es una herramienta útil para analizar el tráfico HTTP y HTTPS entre la aplicación cliente y el servicio de almacenamiento de Azure que está utilizando.

> [!NOTE]
> Fiddler puede descodificar el tráfico HTTPS. Para entender cómo lo hace y comprender las implicaciones para la seguridad, debe leer atentamente la documentación de Fiddler.
>
>

Este apéndice proporciona un breve tutorial que explica cómo configurar Fiddler para capturar el tráfico entre la máquina local donde tiene instalado Fiddler y los servicios de almacenamiento de Azure.

Una vez iniciado Fiddler, empezará a capturar el tráfico HTTP y HTTPS de la máquina local. Estos son algunos comandos útiles para controlar Fiddler:

* Dejar de capturar tráfico y empezar a capturarlo. En el menú principal, vaya a **File** (Archivo) y, luego, haga clic en **Capture Traffic** (Capturar tráfico) para activar o desactivar la captura.
* Guardar los datos de tráfico capturados. Desde el menú principal, vaya a **Archivo**, haga clic en **Guardar** y, a continuación, en **Todas las sesiones**: esto le permitirá guardar el tráfico en un archivo de sesión. Más adelante, puede recargar un archivo de sesión para analizarlo o enviárselo al soporte técnico de Microsoft si se lo solicita.

Para limitar la cantidad de tráfico que captura Fiddler, puede utilizar filtros, que se configuran en la pestaña **Filters** (Filtros). En la siguiente captura de pantalla, se muestra un filtro que captura solamente el tráfico enviado al extremo de almacenamiento **contosoemaildist.table.core.windows.net** :

![Captura de pantalla que muestra un filtro que captura solamente el tráfico enviado al punto de conexión de almacenamiento contosoemaildist.table.core.windows.net.][5]

### <a name="appendix-2-using-wireshark-to-capture-network-traffic"></a><a name="appendix-2"></a>Apéndice 2: Uso de Wireshark para capturar tráfico de red
[Wireshark](https://www.wireshark.org/) es un analizador de protocolos de red que le permite ver información detallada sobre los paquetes de una gran variedad de protocolos de red.

En el siguiente procedimiento, se muestra cómo capturar información detallada sobre los paquetes del tráfico de la máquina local donde instaló Wireshark al servicio Tabla de la cuenta de almacenamiento de Azure.

1. Inicie Wireshark en la máquina local.
2. En la sección **Start** (Inicio), seleccione la interfaz o las interfaces de red locales que están conectadas a Internet.
3. Haga clic en **Capture Options**(Opciones de captura).
4. Agregue un filtro al cuadro de texto **Capture Filter** (Filtro de captura). Por ejemplo, **host contosoemaildist.table.core.windows.net** configurará Wireshark para que capture solamente los paquetes enviados al punto de conexión del servicio Tabla de la cuenta de almacenamiento **contosoemaildist** o desde dicho punto de conexión. Consulte la [lista completa de filtros de captura](https://wiki.wireshark.org/CaptureFilters).

   ![Captura de pantalla que muestra cómo agregar un filtro al cuadro de texto Capture Filter (Filtro de captura).][6]
5. Haga clic en **Iniciar**. Ahora, Wireshark capturará todos los paquetes que se envíen al extremo del servicio Tabla o desde él mientras utiliza la aplicación cliente en la máquina local.
6. Cuando termine, en el menú principal, haga clic en **Capture** (Captura) y, luego, en **Stop** (Detener).
7. Para guardar los datos capturados en un archivo de captura de Wireshark, en el menú principal, haga clic en **File** (Archivo) y, luego, en **Save** (Guardar).

WireShark resaltará todos los errores que haya en la ventana **packetlist** (lista de paquetes). También puede usar la ventana **Expert Info** (Información para expertos) (haga clic en **Analyze** (Analizar) y, después, en **Expert Info** (Información para expertos)) para ver un resumen de los errores y las advertencias.

![Captura de pantalla que muestra la ventana Expert Info (Información experta), donde se puede ver un resumen de los errores y advertencias.][7]

Aparte de esto, puede optar por ver los datos de TCP como los ve el nivel de aplicación: para ello, haga clic con el botón derecho en los datos de TCP y elija **Follow TCP Stream**(Seguir secuencia TCP). Resulta útil si capturó el volcado sin un filtro de captura. Para más información, consulte [Following TCP Streams](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvFollowTCPSection.html)(Siguientes secuencias TCP).

![Captura de pantalla que muestra cómo ver los datos del TCP a medida que los ve la capa de aplicación.][8]

> [!NOTE]
> Para más información sobre el uso de Wireshark, consulte la [Guía del usuario de Wireshark](https://www.wireshark.org/docs/wsug_html_chunked).
>
>

### <a name="appendix-3-using-microsoft-message-analyzer-to-capture-network-traffic"></a><a name="appendix-3"></a>Apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red
Puede usar el Analizador de mensajes de Microsoft para capturar el tráfico HTTP y HTTPS de manera similar a como se hace en Fiddler, y capturar el tráfico de red de forma similar a como lo hace Wireshark.

#### <a name="configure-a-web-tracing-session-using-microsoft-message-analyzer"></a>Configurar una sesión de traza web con el Analizador de mensajes de Microsoft
Para configurar una sesión de traza web del tráfico HTTP y HTTPS con el Analizador de mensajes de Microsoft, ejecute la aplicación Analizador de mensajes de Microsoft y, luego, en el menú **Archivo**, haga clic en **Captura/seguimiento**. En la lista de escenarios de seguimiento disponibles, seleccione **Proxy web**. A continuación, en el panel **Configuración de escenario de seguimiento**, en el cuadro de texto **HostnameFilter**, agregue los nombres de los puntos de conexión de almacenamiento (puede buscar estos nombres en [Azure Portal](https://portal.azure.com)). Por ejemplo, si el nombre de su cuenta de almacenamiento de Azure es **contosodata**, debe agregar lo siguiente al cuadro de texto **HostnameFilter**:

```
contosodata.blob.core.windows.net contosodata.table.core.windows.net contosodata.queue.core.windows.net
```

> [!NOTE]
> Un carácter de espacio separa los nombres de host.
>
>

Cuando esté listo para empezar a recopilar datos de seguimiento, haga clic en el botón **Iniciar con** .

Para más información sobre el seguimiento de **Proxy web** del Analizador de mensajes, consulte [Microsoft-PEF-WebProxy Provider](https://technet.microsoft.com/library/jj674814.aspx)(Proveedor de Microsoft-PEF-WebProxy).

El seguimiento de **Proxy web** integrado en el Analizador de mensajes de Microsoft está basado en Fiddler. Puede capturar el tráfico HTTPS del lado cliente y mostrar mensajes HTTPS sin cifrar. El seguimiento de **Proxy web** funciona configurando un proxy local para todo el tráfico HTTP y HTTPS que le permite acceder a los mensajes no cifrados.

#### <a name="diagnosing-network-issues-using-microsoft-message-analyzer"></a>Diagnóstico de problemas de red con el Analizador de mensajes de Microsoft
Además de usar el seguimiento de **Proxy web** del Analizador de mensajes de Microsoft para capturar detalles del tráfico HTTP/HTTPs entre la aplicación cliente y el servicio de almacenamiento, también puede utilizar el seguimiento del **Nivel de vínculo local** para capturar la información de los paquetes de red. Esto le permite capturar datos similares a los que puede capturar con Wireshark y diagnosticar problemas de red como, por ejemplo, paquetes descartados.

A continuación se muestra un ejemplo de seguimiento del **Nivel de vínculo local** con algunos mensajes de **Información** en la columna **DiagnosisTypes**. Al hacer clic en un icono de la columna **DiagnosisTypes** , se muestran los detalles del mensaje. En este ejemplo, el servidor retransmitió el mensaje número 305 porque no recibió una confirmación del cliente:

![Captura de pantalla que muestra un ejemplo de seguimiento del Nivel de vínculo local con algunos mensajes informativos en la columna DiagnosisTypes][9]

Al crear la sesión de seguimiento en el Analizador de mensajes de Microsoft, puede especificar filtros para reducir la cantidad de ruido del seguimiento. En la página **Captura/seguimiento** donde define el seguimiento, haga clic en el vínculo **Configurar** situado junto a **Microsoft-Windows-NDIS-PacketCapture**. En la siguiente captura de pantalla, se muestra una configuración que filtra el tráfico TCP de las direcciones IP de tres servicios de almacenamiento:

![Captura de pantalla que muestra una configuración que filtra el tráfico TCP de las direcciones IP de tres servicios de almacenamiento.][10]

Para más información sobre el seguimiento del Nivel de vínculo local del Analizador de mensajes de Microsoft, consulte [Microsoft-PEF-NDIS-PacketCapture Provider](https://technet.microsoft.com/library/jj659264.aspx)(Proveedor de Microsoft-PEF-NDIS-PacketCapture).

### <a name="appendix-4-using-excel-to-view-metrics-and-log-data"></a><a name="appendix-4"></a>Apéndice 4: Uso de Excel para ver métricas y datos de registro
Hay muchas herramientas que le permiten descargar los datos de métricas de Azure  Table Storage en un formato delimitado con el que es fácil cargar los datos en Excel para verlos y analizarlos. El almacenamiento de los datos de registro de Azure Blob Storage ya está en un formato delimitado que puede cargar en Excel. Sin embargo, tendrá que agregar los encabezados de columna correspondientes de acuerdo con la información de [Formato del registro de Storage Analytics](https://msdn.microsoft.com/library/azure/hh343259.aspx) y [ Esquema de las tablas de métricas de Storage Analytics](https://msdn.microsoft.com/library/azure/hh343264.aspx).

Para importar los datos del registro de Storage en Excel después de descargarlos del almacenamiento de blobs:

* En el menú **Datos**, haga clic en **Desde texto**.
* Vaya al archivo de registro que quiere ver y haga clic en **Importar**.
* En el paso 1 del **Asistente para importar texto**, seleccione **Delimitado**.

En el paso 1 del **Asistente para importar texto**, seleccione **Punto y coma** como el único delimitador y elija la comilla doble como **Calificador de texto**. Luego, haga clic en **Finalizar** y elija dónde quiere colocar los datos del libro.

### <a name="appendix-5-monitoring-with-application-insights-for-azure-devops"></a><a name="appendix-5"></a>Apéndice 5: Supervisión mediante Application Insights para Azure DevOps
Asimismo, también puede usar la característica Application Insights para Azure DevOps como parte de la supervisión del rendimiento y la disponibilidad. Esta herramienta puede:

* Comprobar que el servicio web está disponible y responde adecuadamente. Tanto si la aplicación es un sitio web como si es una aplicación para dispositivo que utiliza un servicio web, puede probar la URL cada pocos minutos desde ubicaciones de todo el mundo e informar si hay algún problema.
* Diagnosticar rápidamente los problemas de rendimiento o las excepciones del servicio web. Averigüe si la CPU u otros recursos se están ampliando, observe el seguimiento de la pila de las excepciones y busque fácilmente en los seguimientos de registros. Si el rendimiento de la aplicación desciende por debajo de los límites aceptables, Microsoft puede enviarle un correo electrónico. Puede supervisar servicios web de .NET y de Java.

Puede encontrar más información en [¿Qué es Application Insights?](../../azure-monitor/app/app-insights-overview.md)

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de los análisis en Azure Storage, consulte estos recursos:

* [Supervisión de una cuenta de almacenamiento en Azure Portal](storage-monitor-storage-account.md)
* [Storage Analytics](storage-analytics.md)
* [Métricas de Storage Analytics](storage-analytics-metrics.md)
* [Esquema de las tablas de métricas de Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema)
* [Registros de Storage Analytics](storage-analytics-logging.md)
* [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format)

<!--Anchors-->
[Introducción]: #introduction
[Organización de la guía]: #how-this-guide-is-organized

[Supervisión del servicio de almacenamiento]: #monitoring-your-storage-service
[Supervisión del estado del servicio]: #monitoring-service-health
[Supervisión de la capacidad]: #monitoring-capacity
[Supervisión de la disponibilidad]: #monitoring-availability
[Supervisión del rendimiento]: #monitoring-performance

[Diagnóstico de problemas de almacenamiento]: #diagnosing-storage-issues
[Problemas de estado del servicio]: #service-health-issues
[Problemas de rendimiento]: #performance-issues
[Diagnóstico de errores]: #diagnosing-errors
[Problemas del emulador de almacenamiento]: #storage-emulator-issues
[Herramientas de registro de almacenamiento]: #storage-logging-tools
[Uso de herramientas de registro de red]: #using-network-logging-tools

[Seguimiento integral]: #end-to-end-tracing
[Correlación de datos de registro]: #correlating-log-data
[Id. de solicitud de cliente]: #client-request-id
[Identificador de solicitud de servidor]: #server-request-id
[Marcas de tiempo]: #timestamps

[Guía de solución de problemas]: #troubleshooting-guidance
[Las métricas muestran una AverageE2ELatency alta y una AverageServerLatency baja]: #metrics-show-high-AverageE2ELatency-and-low-AverageServerLatency
[Las métricas muestran una AverageE2ELatency baja y una AverageServerLatency baja, pero el cliente tiene latencia alta]: #metrics-show-low-AverageE2ELatency-and-low-AverageServerLatency
[Las métricas muestran una AverageServerLatency alta]: #metrics-show-high-AverageServerLatency
[Observa retrasos inesperados en la entrega de mensajes de una cola]: #you-are-experiencing-unexpected-delays-in-message-delivery

[Las métricas muestran un aumento de PercentThrottlingError]: #metrics-show-an-increase-in-PercentThrottlingError
[Aumento transitorio de PercentThrottlingError]: #transient-increase-in-PercentThrottlingError
[Aumento permanente del error PercentThrottlingError]: #permanent-increase-in-PercentThrottlingError
[Las métricas muestran un aumento de PercentTimeoutError]: #metrics-show-an-increase-in-PercentTimeoutError
[Las métricas muestran un aumento de PercentNetworkError]: #metrics-show-an-increase-in-PercentNetworkError

[El cliente recibe mensajes HTTP 403 (prohibido)]: #the-client-is-receiving-403-messages
[El cliente recibe mensajes HTTP 404 (no encontrado)]: #the-client-is-receiving-404-messages
[El cliente u otro proceso eliminaron anteriormente el objeto]: #client-previously-deleted-the-object
[Un problema de autorización de Firma de acceso compartido (SAS)]: #SAS-authorization-issue
[El código JavaScript del lado cliente no tiene permiso para acceder al objeto]: #JavaScript-code-does-not-have-permission
[Error de red]: #network-failure
[El cliente recibe mensajes HTTP 409 (conflicto)]: #the-client-is-receiving-409-messages

[Las métricas muestran un PercentSuccess bajo o las entradas de registro de análisis tienen operaciones con el estado de transacción ClientOtherErrors]: #metrics-show-low-percent-success
[Las métricas de capacidad muestran un aumento inesperado en el uso de la capacidad de almacenamiento]: #capacity-metrics-show-an-unexpected-increase
[El problema se presenta al usar el emulador de almacenamiento para realizar tareas de desarrollo o pruebas]: #your-issue-arises-from-using-the-storage-emulator
[La característica "X" no funciona en el emulador de almacenamiento]: #feature-X-is-not-working
[Error “El valor de uno de los encabezados HTTP no está en el formato correcto” al usar el emulador de almacenamiento]: #error-HTTP-header-not-correct-format
[La ejecución del emulador de almacenamiento requiere privilegios administrativos]: #storage-emulator-requires-administrative-privileges
[Se producen problemas al instalar el SDK de Azure para .NET]: #you-are-encountering-problems-installing-the-Windows-Azure-SDK
[Tiene otro problema distinto relacionado con un servicio de almacenamiento]: #you-have-a-different-issue-with-a-storage-service

[Apéndices]: #appendices
[Apéndice 1: Uso de Fiddler para capturar tráfico HTTP y HTTPS]: #appendix-1
[Apéndice 2: Uso de Wireshark para capturar tráfico de red]: #appendix-2
[Apéndice 3: Uso de Microsoft Message Analyzer para capturar tráfico de red]: #appendix-3
[Apéndice 4: Uso de Excel para ver métricas y datos de registro]: #appendix-4
[Apéndice 5: Supervisión mediante Application Insights para Azure DevOps]: #appendix-5

<!--Image references-->
[1]: ./media/storage-monitoring-diagnosing-troubleshooting/overview.png
[3]: ./media/storage-monitoring-diagnosing-troubleshooting/hour-minute-metrics.png
[4]: ./media/storage-monitoring-diagnosing-troubleshooting/high-e2e-latency.png
[5]: ./media/storage-monitoring-diagnosing-troubleshooting/fiddler-screenshot.png
[6]: ./media/storage-monitoring-diagnosing-troubleshooting/wireshark-screenshot-1.png
[7]: ./media/storage-monitoring-diagnosing-troubleshooting/wireshark-screenshot-2.png
[8]: ./media/storage-monitoring-diagnosing-troubleshooting/wireshark-screenshot-3.png
[9]: ./media/storage-monitoring-diagnosing-troubleshooting/mma-screenshot-1.png
[10]: ./media/storage-monitoring-diagnosing-troubleshooting/mma-screenshot-2.png
