---
title: Guía sobre datos personales almacenados en Azure Log Analytics | Microsoft Docs
description: En este artículo se describe cómo administrar los datos personales almacenados en Azure Log Analytics, así como los métodos para identificarlos y quitarlos.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/18/2018
ms.openlocfilehash: 64c461c5d3e1bb34f480e5173621f8753eadbbd8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318324"
---
# <a name="guidance-for-personal-data-stored-in-log-analytics-and-application-insights"></a>Guía sobre datos personales almacenados en Log Analytics y Application Insights

Log Analytics es un almacén de datos donde es probable que haya datos personales. Application Insights almacena sus datos en una partición de Log Analytics. En este artículo se describe en qué parte de Log Analytics y Application Insights se encuentran estos datos normalmente, así como las funcionalidades disponibles para administrarlos.

> [!NOTE]
> Para los fines de este artículo, _registrar datos_ hace referencia a los datos enviados a un área de trabajo de Log Analytics, mientras que _datos de la aplicación_ hace referencia a los datos recopilados por Application Insights.

[!INCLUDE [gdpr-dsr-and-stp-note](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="strategy-for-personal-data-handling"></a>Estrategia de tratamiento de datos personales

Mientras que usted y su empresa sean los responsables en última instancia de determinar la estrategia de tratamiento de los datos privados (si los hubiera), pueden adoptarse los siguientes enfoques. Aparecen en orden de preferencia (de más a menos) desde un punto de vista técnico:

* Siempre que sea posible, detener la recopilación de los datos, ofuscarlos o anonimizarlos, o bien ajustar la recopilación de los datos para excluirlos por ser de carácter "privado". Este método es el preferido _con diferencia_, ya que evita la necesidad de crear una estrategia de tratamiento de datos rápida y costosa.
* Si no es posible, trate de normalizar los datos para reducir el impacto en el rendimiento y la plataforma de datos. Por ejemplo, en lugar de registrar un identificador de usuario explícita, cree una búsqueda para los datos que se correlacionarán el nombre de usuario y sus detalles con un identificador interno que luego puede registrarse en otra parte. De este modo, si uno de los usuarios le pide eliminar su información personal, se puede hacer suprimiendo únicamente la fila en la tabla de búsqueda correspondientes al usuario. 
* Por último, si se deben recopilar datos privados, cree un proceso en la ruta de acceso de la API de depuración y en la de la API de consulta existente para cumplir con las obligaciones relativas a exportar y eliminar cualquier dato privado asociado con un usuario. 

## <a name="where-to-look-for-private-data-in-log-analytics"></a>¿Dónde se pueden buscar datos privados en Log Analytics?

Log Analytics es un almacén completamente flexible que, además de recomendar un esquema a los datos, permite reemplazar todos los campos con valores predeterminados. Además, se puede ingerir cualquier esquema personalizado. Por lo tanto, no podemos saber exactamente dónde se encuentran datos privados en su área de trabajo concreta. Sin embargo, las siguientes ubicaciones son buenos puntos de partida en su inventario:

### <a name="log-data"></a>Datos de registro

* *Direcciones IP*: Log Analytics recopila una gran variedad de información de IP en numerosas tablas diferentes. Por ejemplo, la siguiente consulta muestra todas las tablas donde se han recopilado direcciones IPv4 en las últimas 24 horas:
    ```
    search * 
    | where * matches regex @'\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.|$)){4}\b' //RegEx originally provided on https://stackoverflow.com/questions/5284147/validating-ipv4-addresses-with-regexp
    | summarize count() by $table
    ```
* *Identificadores de usuario*: los identificadores de usuario se encuentran en una gran variedad de soluciones y tablas. Puede buscar un nombre de usuario determinado en el conjunto de datos completo con el comando de búsqueda:
    ```
    search "[username goes here]"
    ```
  Recuerde que debe buscar no solo los nombres de usuario legible para el usuario, sino también los GUID que se pueden rastrear directamente hasta un usuario determinado.
* *Identificadores de dispositivo*: al igual que los identificadores de usuario, los identificadores de dispositivo a veces se consideran "privados". Use el mismo enfoque que se indicó para los identificadores de usuario para identificar las tablas en las que esto podría suponer un problema. 
* *Datos personalizados*: Log Analytics permite recopilar de varias maneras: registros personalizados y campos personalizados, [API de recopilador de datos HTTP](./data-collector-api.md) y datos personalizados recopilados como parte de los registros de eventos del sistema. Todos estos son susceptibles de contener datos privados y se deben examinar para comprobar si es así.
* *Datos capturados por la solución*: como el mecanismo de la solución es de extremo abierto, se recomienda revisar todas las tablas generadas por las soluciones para garantizar el cumplimiento.

### <a name="application-data"></a>Datos de aplicación

* *Direcciones IP*: aunque Application Insights ofuscará de forma predeterminada todos los campos de direcciones IP a "0.0.0.0", es un patrón bastante común reemplazar este valor con la dirección IP de usuario real con el fin de mantener la información de la sesión. La siguiente consulta de Analytics puede utilizarse para buscar cualquier tabla que contenga valores en la columna de dirección IP que no sea "0.0.0.0" en las últimas 24 horas:
    ```
    search client_IP != "0.0.0.0"
    | where timestamp > ago(1d)
    | summarize numNonObfuscatedIPs_24h = count() by $table
    ```
* *Identificadores de usuario*: de forma predeterminada, Application Insights usará identificadores generados de forma aleatoria para el usuario y el seguimiento de sesión. Sin embargo, es habitual ver estos campos reemplazos para almacenar un identificador más relevante para la aplicación. Por ejemplo: nombres de usuario, GUID de AAD, etc. Estos identificadores suelen considerarse como datos personales y, por lo tanto, deben tratarse de la forma adecuada. Siempre recomendamos tratar de ofuscar o anonimizar estos identificadores. Algunos campos donde suelen encontrarse estos valores son session_Id, user_Id, user_AuthenticatedId, user_AccountId y customDimensions.
* *Datos personalizados*: Application Insights permite anexar un conjunto de dimensiones personalizadas a cualquier tipo de datos. Estas dimensiones pueden ser *cualquier* dato. Utilice la siguiente consulta para identificar las dimensiones personalizadas recopiladas durante las últimas 24 horas:
    ```
    search * 
    | where isnotempty(customDimensions)
    | where timestamp > ago(1d)
    | project $table, timestamp, name, customDimensions 
    ```
* *Datos en memoria y en tránsito*: Application Insights hará un seguimiento de las excepciones, las solicitudes, las llamadas de dependencia y los seguimientos. A menudo se pueden recopilar los datos privados en el código y en las llamadas HTTP. Revise las excepciones, las solicitudes, las dependencias y las tablas de seguimiento para identificar estos datos. Use [inicializadores de telemetría](../app/api-filtering-sampling.md) siempre que sea posible para ofuscar estos datos.
* *Capturas de Snapshot Debugger*: la característica [Snapshot Debugger](../app/snapshot-debugger.md) de Application Insights permite recopilar las instantáneas de depuración cuando se detecta una excepción en la instancia de producción de la aplicación. Las instantáneas expondrán el seguimiento de la pila completo que provoca las excepciones, así como los valores para las variables locales en cada etapa de la pila. Por desgracia, esta característica no permite eliminar de forma selectiva los puntos de ajuste ni acceder mediante programación a los datos dentro de la instantánea. Por lo tanto, si la velocidad de retención de instantáneas predeterminada no satisface sus requisitos de cumplimiento, recomendamos desactivar la característica.

## <a name="how-to-export-and-delete-private-data"></a>Cómo exportar y eliminar datos privados

Tal y como se mencionó en la sección sobre la [estrategia de tratamiento de datos personales](#strategy-for-personal-data-handling), se recomienda __encarecidamente__, siempre que sea posible, reestructurar la directiva de recopilación de datos para deshabilitar la recopilación, ofuscación y anonimización de datos, o bien modificarlos para excluirlos por considerarse de carácter "privado". Tanto usted como su equipo tendrán que asumir costos relacionados con la definición y automatización de una estrategia, la creación de una interfaz para que los clientes interactúen con sus datos y el mantenimiento continuo. Además, esto requiere que Log Analytics y Application Insights funcionen de forma intensiva, y debido al gran volumen de llamadas simultáneas de API de purga o consulta, es posible el resto de las interacciones con la funcionalidad de Log Analytics se vean afectadas de forma negativa. Dicho esto, hay realmente algunos escenarios válidos en los que se deben recopilar datos privados. En estos casos, los datos deben tratarse como se describe en esta sección.

[!INCLUDE [gdpr-intro-sentence](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-and-export"></a>Visualización y exportación

Para ver y exportar las solicitudes de datos, debe usarse la [API de consulta de Log Analytics](https://dev.loganalytics.io/) o la [API de consulta de Application Insights](https://dev.applicationinsights.io/quickstart). Tendrá que decidir qué lógica implementará para convertir la forma de los datos a uno adecuado para los usuarios. [Azure Functions](https://azure.microsoft.com/services/functions/) es un buen lugar para hospedar esta lógica.

> [!IMPORTANT]
>  Aunque la mayoría de las operaciones de purga pueden completarse mucho más rápido que lo establecido en el Acuerdo de Nivel de Servicio (SLA), **el SLA formal para la realización de operaciones de purga es de 30 días** debido a su gran impacto en la plataforma de datos utilizada. Se trata de un proceso automatizado; no hay ninguna forma de solicitar que una operación se controle con mayor rapidez.

### <a name="delete"></a>Eliminar

> [!WARNING]
> Las eliminaciones en Log Analytics son destructivas y no reversibles. Por tanto, extreme las precauciones cuando las ejecute.

Como parte de un caso de tratamiento de privacidad, hemos publicado una ruta de acceso a una API de *purga*. Esta ruta de acceso debe usarse con moderación debido al riesgo asociado inherente, al posible impacto en el rendimiento y a la posibilidad de sesgar completamente agregaciones, medidas y otros aspectos de los datos de Log Analytics. Consulte la sección [Estrategia de tratamiento de datos personales](#strategy-for-personal-data-handling) para conocer otras maneras de administrar los datos privados.

La purga es una operación con privilegios elevados que ninguna aplicación ni usuario de Azure (incluido incluso el propietario del recurso) tiene permisos para ejecutar sin que se le haya concedido expresamente un rol en Azure Resource Manager. Este rol es _Purgador de datos_ y se debe delegar con cuidado debido a la posibilidad de pérdida de datos. 

> [!IMPORTANT]
> Con el fin de administrar los recursos del sistema, las solicitudes de purga se limitan a 50 solicitudes por hora. Debe procesar por lotes la ejecución de las solicitudes de purga mediante el envío de un único comando cuyo predicado incluya todas las identidades de usuario que requieran purga. Use el [operador in](/azure/kusto/query/inoperator) para especificar varias identidades. Debe ejecutar la consulta antes de ejecutar la solicitud de purga para comprobar que se esperan los resultados. 



Una vez que se ha asignado el rol de Azure Resource Manager, habrá disponibles dos nuevas rutas de acceso de API: 

#### <a name="log-data"></a>Datos de registro

* [Purga POST](/rest/api/loganalytics/workspacepurge/purge): toma un objeto que especifica los parámetros de datos que se van a eliminar y devuelve un GUID de referencia. 
* GET purge status: la llamada POST purge devolverá un encabezado "x-ms-status-location" que incluye una dirección URL a la que se puede llamar para determinar el estado de la API de purga. Por ejemplo:

    ```
    x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/Microsoft.OperationalInsights/workspaces/[WorkspaceName]/operations/purge-[PurgeOperationId]?api-version=2015-03-20
    ```

> [!IMPORTANT]
>  Aunque se espera que la mayoría de las operaciones de purga se completen mucho más rápido que lo establecido en el Acuerdo de Nivel de Servicio (SLA), debido a su gran impacto en la plataforma de datos utilizada por Log Analytics, **el SLA formal para la realización de operaciones de purga es de 30 días**. 

#### <a name="application-data"></a>Datos de aplicación

* [Purga POST](/rest/api/application-insights/components/purge): toma un objeto que especifica los parámetros de datos que se van a eliminar y devuelve un GUID de referencia.
* GET purge status: la llamada POST purge devolverá un encabezado "x-ms-status-location" que incluye una dirección URL a la que se puede llamar para determinar el estado de la API de purga. Por ejemplo:

   ```
   x-ms-status-location: https://management.azure.com/subscriptions/[SubscriptionId]/resourceGroups/[ResourceGroupName]/providers/microsoft.insights/components/[ComponentName]/operations/purge-[PurgeOperationId]?api-version=2015-05-01
   ```

> [!IMPORTANT]
>  Aunque la mayoría de las operaciones de purga pueden completarse mucho más rápido que lo establecido en el Acuerdo de Nivel de Servicio (SLA), debido a su gran impacto en la plataforma de datos que utiliza Application Insights, **el SLA formal para la realización de operaciones de purga es de 30 días**.

## <a name="next-steps"></a>Pasos siguientes
- Para obtener más información sobre cómo se recopilan, procesan y protegen los datos de Log Analytics, consulte [Seguridad de datos de Log Analytics](./data-security.md).
- Para obtener más información sobre cómo se recopilan, procesan y protegen los datos de Application Insights, vea [Seguridad de datos de Application Insights](../app/data-retention-privacy.md).

