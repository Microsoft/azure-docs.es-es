---
title: Solución de errores intermitentes en la conexión de salida en Azure App Service
description: Solucione errores de conexión intermitentes y problemas de rendimiento relacionados en Azure App Service
author: v-miegge
manager: barbkess
ms.topic: troubleshooting
ms.date: 07/24/2020
ms.author: ramakoni
ms.custom: security-recommendations,fasttrack-edit
ms.openlocfilehash: 467f7b3525883e16e57a06ff97cf4fd386279d22
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958242"
---
# <a name="troubleshooting-intermittent-outbound-connection-errors-in-azure-app-service"></a>Solución de errores intermitentes en la conexión de salida en Azure App Service

Este artículo le ayuda a solucionar errores de conexión intermitentes y problemas de rendimiento relacionados en [Azure App Service](./overview.md). En este tema se proporciona más información sobre las metodologías y la solución de problemas de agotamiento de los puertos de traducción de red de direcciones (SNAT) de origen. Si necesita más ayuda con cualquier aspecto de este artículo, póngase en contacto con los expertos de Azure en los [foros de MSDN Azure y Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Soporte técnico**.

## <a name="symptoms"></a>Síntomas

Las aplicaciones y las funciones que se hospedan en Azure App Service pueden presentar uno o varios de los síntomas siguientes:

* Tiempos de respuesta lentos en todas o algunas de las instancias de un plan de servicio.
* Errores 5xx o de **Puerta de enlace incorrecta** intermitentes.
* Mensaje de error relativo al tiempo de expiración.
* No se pudo conectar a puntos de conexión externos (como SQLDB, Service Fabric, otros servicios de aplicaciones, etc.).

## <a name="cause"></a>Causa

Una causa principal de estos síntomas es que la instancia de la aplicación no puede abrir una conexión nueva con el punto de conexión externo porque ha alcanzado uno de los límites siguientes:

* Conexiones TCP: existe un límite en cuanto al número de conexiones salientes que se pueden realizar. Esto se asocia con el tamaño del trabajo que se usa.
* Puertos SNAT: como se describe en [Conexiones salientes en Azure](../load-balancer/load-balancer-outbound-connections.md), Azure usa la traducción de direcciones de red de origen (SNAT) y el equilibrador de carga (no expuesto a los clientes) para comunicarse con los puntos de conexión externos a Azure en el espacio de direcciones IP públicas. A cada instancia de Azure App Service se le asigna inicialmente un número predefinido de **128** puertos SNAT. Ese límite afecta a la apertura de conexiones a la misma combinación de host y puerto. Si la aplicación crea conexiones a una mezcla de combinaciones de direcciones y puertos, no se usarán los puertos SNAT. Los puertos SNAT se usan cuando hay llamadas repetidas a la misma combinación de dirección y puerto. Una vez que se ha liberado un puerto, está disponible para reutilizarse según sea necesario. El equilibrador de carga de red de Azure solo reclama el puerto SNAT de las conexiones cerradas después de esperar 4 minutos.

Si las aplicaciones o las funciones abren rápidamente una conexión nueva, pueden agotar con rapidez la cuota preasignada de 128 puertos. Después, se bloquean hasta que un nuevo puerto SNAT esté disponible, ya sea a través de la asignación dinámica de puertos SNAT adicionales o mediante la reutilización de un puerto SNAT reclamado. Las aplicaciones o funciones que estén bloqueadas debido a esta imposibilidad de crear conexiones comenzarán a experimentar uno o varios de los problemas descritos en la sección **Síntomas** de este artículo.

## <a name="avoiding-the-problem"></a>Cómo evitar el problema

Si el destino es un servicio de Azure que admite puntos de conexión de servicio, podrá evitar los problemas de agotamiento de puertos SNAT utilizando la [integración con red virtual regional](./web-sites-integrate-with-vnet.md) junto con puntos de conexión privados o de servicio. Cuando se usa la integración con red virtual regional y se colocan puntos de conexión de servicio en la subred de integración, el tráfico que sale de la aplicación hacia esos servicios no tiene restricciones sobre los puertos SNAT de salida. Del mismo modo, si usa la integración con red virtual regional junto con puntos de conexión privados, no tendrá ningún problema con los puertos SNAT de salida en ese destino. 

Evitar el problema del puerto SNAT se traduce en evitar la creación de conexiones repetidas a los mismos host y puerto.

Las estrategias generales para la mitigación del agotamiento de puertos SNAT se describen en la sección [Solución de problemas](../load-balancer/load-balancer-outbound-connections.md) de la documentación **Conexiones salientes de Azure**. De estas estrategias, las siguientes se aplican a las funciones y aplicaciones hospedadas en el servicio Azure App Service.

### <a name="modify-the-application-to-use-connection-pooling"></a>Modificación de la aplicación para usar la agrupación de conexiones

* Para agrupar las conexiones HTTP, revise [Agrupación de conexiones HTTP con HttpClientFactory](/aspnet/core/performance/performance-best-practices#pool-http-connections-with-httpclientfactory).
* Para obtener información sobre la agrupación de conexiones de SQL Server, consulte [Agrupación de conexiones de SQL Server (ADO.NET)](/dotnet/framework/data/adonet/sql-server-connection-pooling).
* Para implementar la agrupación con aplicaciones de Entity Framework, revise [Agrupación de DbContext](/ef/core/what-is-new/ef-core-2.0#dbcontext-pooling).

Esta es una colección de vínculos para implementar la agrupación de conexiones por otra pila de soluciones.

#### <a name="node"></a>Nodo

De forma predeterminada, las conexiones de NodeJS no se mantienen activas. A continuación se muestran las bases de datos y los paquetes más populares para la agrupación de conexiones, con ejemplos de cómo implementarlos.

* [MySQL](https://github.com/mysqljs/mysql#pooling-connections)
* [MongoDB](https://blog.mlab.com/2017/05/mongodb-connection-pooling-for-express-applications/)
* [PostgreSQL](https://node-postgres.com/features/pooling)
* [SQL Server](https://github.com/tediousjs/node-mssql#connection-pools)

Mantenimiento de conexiones HTTP

* [agentkeepalive](https://www.npmjs.com/package/agentkeepalive)
* [Documentación de Node.js v13.9.0](https://nodejs.org/api/http.html)

#### <a name="java"></a>Java

A continuación se muestran las bibliotecas más populares usadas para la agrupación de conexiones JDBC, con ejemplos de cómo implementarlas: Agrupación de conexiones JDBC.

* [Tomcat 8](https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html)
* [C3p0](https://github.com/swaldman/c3p0)
* [HikariCP](https://github.com/brettwooldridge/HikariCP)
* [Apache DBCP](https://commons.apache.org/proper/commons-dbcp/)

Agrupación de conexiones HTTP

* [Administración de conexiones de Apache](https://hc.apache.org/httpcomponents-client-ga/tutorial/html/connmgmt.html)
* [Clase PoolingHttpClientConnectionManager](http://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/impl/conn/PoolingHttpClientConnectionManager.html)

#### <a name="php"></a>PHP

Aunque PHP no admite la agrupación de conexiones, puede intentar usar conexiones de base de datos persistentes en el servidor de back-end.
 
* Servidor MySQL

   * [Conexiones de MySQLi](https://www.php.net/manual/mysqli.quickstart.connections.php) para las versiones más recientes
   * [mysql_pconnect](https://www.php.net/manual/function.mysql-pconnect.php) para versiones anteriores de PHP

* Otros orígenes de datos

   * [Administración de conexiones de PHP](https://www.php.net/manual/en/pdo.connections.php)

#### <a name="python"></a>Python

* [MySQL](https://github.com/mysqljs/mysql#pooling-connections)
* [MongoDB](https://blog.mlab.com/2017/05/mongodb-connection-pooling-for-express-applications/)
* [PostgreSQL](https://node-postgres.com/features/pooling)
* [SQL Server](https://github.com/tediousjs/node-mssql#connection-pools) (Nota: SQLAlchemy se puede usar con otras bases de datos además de Microsoft SQL Server)
* [HTTP Keep-alive](https://requests.readthedocs.io/en/master/user/advanced/#keep-alive) (Keep-alive es automático al usar objetos [session-objects](https://requests.readthedocs.io/en/master/user/advanced/#keep-alive) de sesiones).

En el caso de otros entornos, revise la documentación específica del proveedor o del controlador en cuestión para implementar la agrupación de conexiones en las aplicaciones.

### <a name="modify-the-application-to-reuse-connections"></a>Modificación de la aplicación para reutilizar las conexiones

*  Para obtener más punteros y ejemplos sobre la administración de conexiones en funciones de Azure, revise [Administración de conexiones en Azure Functions](../azure-functions/manage-connections.md).

### <a name="modify-the-application-to-use-less-aggressive-retry-logic"></a>Modificación de la aplicación para usar lógica de reintento menos agresiva

* Para obtener instrucciones y ejemplos adicionales, revise [Patrón de reintento](/azure/architecture/patterns/retry).

### <a name="use-keepalives-to-reset-the-outbound-idle-timeout"></a>Uso de conexiones persistentes para restablecer el tiempo de expiración de inactividad saliente

* Para implementar conexiones persistentes para aplicaciones de Node.js, revise [Mi aplicación Node realiza demasiadas llamadas salientes](./app-service-web-nodejs-best-practices-and-troubleshoot-guide.md#my-node-application-is-making-excessive-outbound-calls).

### <a name="additional-guidance-specific-to-app-service"></a>Instrucciones adicionales específicas para App Service:

* Una [prueba de carga](/azure/devops/test/load-test/app-service-web-app-performance-test) debe simular datos reales a una velocidad de alimentación estable. La prueba de las aplicaciones y funciones sometidas a esfuerzo real permite identificar y resolver los problemas de agotamiento de puertos SNAT con anterioridad.
* Asegúrese de que los servicios back-end puedan devolver respuestas con rapidez. Para solucionar problemas de rendimiento de base de datos de Azure SQL Database, revise [Solución de problemas de rendimiento de Azure SQL Database con Intelligent Insights](../azure-sql/database/intelligent-insights-troubleshoot-performance.md#recommended-troubleshooting-flow).
* Escale horizontalmente el plan de App Service a más instancias. Para obtener más información sobre el escalado, consulte [Escalado de una aplicación en Azure App Service](./manage-scale-up.md). A cada instancia de trabajo de un plan de App Service se le asigna un número de puertos SNAT. Si distribuye el uso entre más instancias, es posible que consiga reducir el uso de puertos SNAT por instancia por debajo del límite recomendado de 100 conexiones salientes, por punto de conexión remoto único.
* Considere la posibilidad de cambiar a [App Service Environment (ASE)](./environment/using-an-ase.md), donde se le asigna una sola dirección IP de salida, y los límites de conexiones y puertos SNAT son mucho mayores. En un ASE, el número de puertos SNAT por instancia se basa en la [tabla de asignación previa del equilibrador de carga de Azure](../load-balancer/load-balancer-outbound-connections.md#snatporttable): por ejemplo, un ASE con 1 a 50 instancias de trabajo tiene 1024 puertos preasignados por instancia, mientras que un ASE con 51 a 100 instancias de trabajo tiene 512 puertos preasignados por instancia.

Evitar los límites de TCP salientes es algo más fácil de resolver, ya que los límites se establecen en función del tamaño del trabajo. Puede ver los límites en [Límites numéricos de máquina virtual entre espacios aislados: conexiones TCP](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#cross-vm-numerical-limits).

|Nombre del límite|Descripción|Pequeño (A1)|Medio (A2)|Grande (A3)|Nivel aislado (ASE)|
|---|---|---|---|---|---|
|Conexiones|Número de conexiones en toda la máquina virtual|1920|3968|8064|16 000|

Para evitar los límites de TCP salientes, puede aumentar el tamaño de los trabajos o realizar un escalado horizontal.

## <a name="troubleshooting"></a>Solución de problemas

Si conoce los dos tipos de límites de conexión de salida y lo que hace la aplicación, la solución de problemas debería ser más fácil. Si sabe que la aplicación realiza muchas llamadas a la misma cuenta de almacenamiento, podría sospechar de un límite de SNAT. Si la aplicación crea una gran cantidad de llamadas a los puntos de conexión a través de Internet, sospecharía que está llegando al límite de la máquina virtual.

Si no tiene suficiente información sobre el comportamiento de la aplicación para determinar la causa con rapidez, hay algunas herramientas y técnicas disponibles en App Service para ayudarle a determinarlo.

### <a name="find-snat-port-allocation-information"></a>Búsqueda de información de asignación de puertos SNAT

Puede usar [Diagnósticos de App Service](./overview-diagnostics.md) para buscar información sobre la asignación de puertos SNAT y observar la métrica de asignación de puertos SNAT de un sitio de App Service. Para buscar información sobre la asignación de puertos SNAT, siga estos pasos:

1. Para acceder al diagnóstico de App Service, vaya a la aplicación web de App Service o a App Service Environment en [Azure Portal](https://portal.azure.com/). En el panel de navegación de la izquierda, seleccione **Diagnosticar y solucionar problemas**.
2. Seleccione la categoría Disponibilidad y rendimiento.
3. Seleccione el icono de agotamiento del puerto SNAT en la lista de iconos disponibles en la categoría. El procedimiento consiste en mantenerlo por debajo de 128.
Si lo necesita, puede abrir una incidencia y el ingeniero de soporte técnico obtendrá de forma automática la métrica de back-end.

Tenga en cuenta que, como el uso de puertos SNAT no está disponible como una métrica, no es posible escalar automáticamente en función del uso de los puertos SNAT, o bien configurar el escalado automático en función de la métrica de asignación de puertos SNAT.

### <a name="tcp-connections-and-snat-ports"></a>Conexiones TCP y puertos SNAT

Las conexiones TCP y los puertos SNAT no están directamente relacionados. Un detector de uso de conexiones TCP se incluye en la hoja Diagnosticar y solucionar problemas de cualquier sitio de App Service. Busque la frase "Conexiones TCP" para encontrarla.

* Los puertos SNAT solo se usan para flujos de red externos, mientras que el número total de conexiones TCP incluye conexiones de bucle invertido local.
* Un puerto SNAT se puede compartir entre distintos flujos, si difieren en el protocolo, la dirección IP o el puerto. La métrica de conexiones TCP cuenta todas las conexiones TCP.
* El límite de conexiones TCP se produce en el nivel de instancia de trabajo. El equilibrio de carga de salida de red de Azure no usa la métrica de conexiones TCP para la limitación de puertos SNAT.
* Los límites de conexiones TCP se describen en [Límites numéricos de máquinas virtuales entre espacios aislados: conexiones TCP](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#cross-vm-numerical-limits).

|Nombre del límite|Descripción|Pequeño (A1)|Medio (A2)|Grande (A3)|Nivel aislado (ASE)|
|---|---|---|---|---|---|
|Conexiones|Número de conexiones en toda la máquina virtual|1920|3968|8064|16 000|

### <a name="webjobs-and-database-connections"></a>Trabajos web y conexiones de base de datos
 
Si se agotan los puertos SNAT y los trabajos web no se pueden conectar a SQL Database, no hay ninguna métrica para mostrar el número de conexiones abiertas por cada proceso de aplicación web individual. Para encontrar el trabajo web con el problema, puede transferir varios trabajos web a otro plan de App Service para ver si la situación mejora, o bien si un problema se mantiene en uno de los planes. Repita el proceso hasta que encuentre el trabajo web con el problema.

### <a name="using-snat-ports-sooner"></a>Uso temprano de puertos SNAT

No se puede cambiar la configuración de Azure para liberar los puertos SNAT usados antes, ya que todos los puertos SNAT se liberarán según las condiciones siguientes y el comportamiento es por diseño.
 
* Si el servidor o el cliente envía FINACK, el [puerto SNAT se liberará](../load-balancer/load-balancer-outbound-connections.md) después de 240 segundos.
* Si se ve un RST, el puerto SNAT se liberará después de 15 segundos.
* Si se ha alcanzado del tiempo de expiración, el puerto se libera.
 
## <a name="additional-information"></a>Información adicional

* [SNAT con App Service](https://4lowtherabbit.github.io/blogs/2019/10/SNAT/)
* [Solución de problemas de rendimiento reducido de aplicaciones web en Azure App Service](./troubleshoot-performance-degradation.md)