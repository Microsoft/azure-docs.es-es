---
title: Tutorial de aplicaciones SaaS con una base de datos por inquilino
description: Implemente y explore la aplicación SaaS multiinquilino Wingtip Tickets, que muestra el patrón de base de datos por inquilino y otros patrones SaaS con Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 8d71f6e71a7cd6033e570742a23016df0c25dd27
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542253"
---
# <a name="deploy-and-explore-a-multitenant-saas-app-that-uses-the-database-per-tenant-pattern-with-azure-sql-database"></a>Implementación y exploración de una aplicación SaaS multiinquilino que usa el patrón de base de datos por inquilino con Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, se implementa y explora la aplicación de base de datos por inquilino de SaaS Wingtip Tickets (Wingtip). La aplicación usa un patrón de base de datos por inquilino para almacenar los datos de varios inquilinos. La aplicación está diseñada para presentar las características de Azure SQL Database que simplifican la habilitación de escenarios de SaaS.

Cinco minutos después de que seleccione **Implementar en Azure**, tendrá una aplicación de SaaS multiinquilino. La aplicación incluye una base de datos que se ejecuta en Azure SQL Database. La aplicación se implementa con tres inquilinos de ejemplo, cada uno con su propia base de datos. Todas las bases de datos se implementan en un grupo elástico de SQL. La aplicación se implementa en la suscripción de Azure. Tiene acceso completo para explorar y trabajar con los componentes individuales de la aplicación. El código fuente C# de la aplicación y los scripts de administración están disponibles en el [repositorio de GitHub WingtipTicketsSaaS-DbPerTenant][github-wingtip-dpt].

En este tutorial, obtendrá información:

> [!div class="checklist"]
> - Cómo implementar la aplicación SaaS de Wingtip.
> - Dónde obtener el código fuente de la aplicación y los scripts de administración.
> - Acerca de los servidores, los grupos y las bases de datos que componen la aplicación.
> - Cómo se asignan los inquilinos a sus datos con el *catálogo*.
> - Cómo aprovisionar un nuevo inquilino.
> - Cómo supervisar la actividad de los inquilinos en la aplicación.

Una [serie de tutoriales relacionados](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials) ofrece la posibilidad de explorar varios patrones de diseño y administración de SaaS. Los tutoriales se basan en esta implementación inicial. Cuando use los tutoriales, podrá examinar los scripts que se proporcionan para ver cómo se implementan los distintos patrones de SaaS. Los scripts muestran cómo las características de SQL Database simplifican el desarrollo de aplicaciones de SaaS.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de que Azure PowerShell está instalado. Para más información, consulte el artículo de [introducción a Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps).

## <a name="deploy-the-wingtip-tickets-saas-application"></a>Implementación de la aplicación SaaS Wingtip Tickets

### <a name="plan-the-names"></a>Planeamiento de los nombres

En los pasos de esta sección, se proporciona un valor de usuario que se usa para garantizar que los nombres de los recursos son globalmente únicos. También se proporcionará un nombre para el grupo de recursos con todos los recursos que se crean a partir de una implementación de la aplicación. Para una persona ficticia llamada Ann Finley, sugerimos:

- **Usuario**: *af1*, con las iniciales de Ann Finley más un dígito. Si implementa la aplicación una segunda vez, use otro valor. Por ejemplo, af2.
- **Grupo de recursos:** *wingtip-dpt-af1* indica que esta es la aplicación de base de datos por inquilino. Anexe el nombre de usuario af1 para correlacionar el nombre del grupo de recursos con los nombres de los recursos que contiene.

Elija los nombres ahora y escríbalos.

### <a name="steps"></a>Pasos

1. Para abrir la plantilla de implementación de base de datos por inquilino de la aplicación SAS Wingtip Tickets en Azure Portal, seleccione **Implementar en Azure**.

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://azuredeploy.net/deploybutton.png)](https://aka.ms/deploywingtipdpt)

1. Escriba valores para los parámetros requeridos en la plantilla.

    > [!IMPORTANT]
    > Para realizar la demostración, se ha eliminado intencionadamente la protección de varios firewalls de autenticación y del servidor. Es recomendable crear un grupo de recursos nuevo. No use grupos de recursos, servidores o grupos existentes. No use esta aplicación, los scripts ni ninguno de los recursos de implementación para producción. Elimine este grupo de recursos cuando haya terminado con la aplicación para detener la facturación relacionada con él.

    - **Grupo de recursos**: seleccione **Crear nuevo** y proporcione el nombre único que eligió antes para el grupo de recursos.
    - **Ubicación**: seleccione una ubicación en la lista desplegable.
    - **Usuario**: use el valor de nombre de usuario que eligió antes.

1. Implemente la aplicación.

    a. Seleccione para aceptar los términos y condiciones.

    b. Seleccione **Comprar**.

1. Para supervisar el estado de la implementación, seleccione **Notificaciones** (el icono de campana que se encuentra a la derecha del cuadro de búsqueda). La implementación de la aplicación SaaS Wingtip Tickets tarda aproximadamente cinco minutos.

   ![Implementación correcta](./media/saas-dbpertenant-get-started-deploy/succeeded.png)

## <a name="download-and-unblock-the-wingtip-tickets-management-scripts"></a>Descarga y desbloqueo de los scripts de administración de Wingtip Tickets

Mientras la aplicación se implementa, descargue el código fuente y los scripts de administración.

> [!IMPORTANT]
> Es posible que Windows bloquee el contenido ejecutable (scripts y archivos DLL) al descargar y extraer archivos ZIP de un origen externo. Siga los pasos para desbloquear el archivo ZIP antes de extraer los scripts. El desbloqueo garantiza que se puedan ejecutar los scripts.

1. Vaya al [repositorio de GitHub WingtipTicketsSaaS-DbPerTenant][github-wingtip-dpt].
1. Seleccione **Clone or download**(Clonar o descargar).
1. Seleccione **Download ZIP** (Descargar archivo ZIP) y guarde el archivo.
1. Haga clic con el botón derecho en el archivo **WingtipTicketsSaaS-DbPerTenant-master.zip** y seleccione **Properties** (Propiedades).
1. En la pestaña **General**, seleccione **Desbloquear** > **Aplicar**.
1. Seleccione **Aceptar** y extraiga los archivos.

Los scripts se encuentran en la carpeta ...\\WingtipTicketsSaaS-DbPerTenant-master\\Learning Modules.

## <a name="update-the-user-configuration-file-for-this-deployment"></a>Actualización del archivo de configuración de usuario para esta implementación

Antes de ejecutar cualquier script, actualice los valores del grupo de recursos y del usuario en el archivo de configuración del usuario. Establezca estas variables en los valores usados durante la implementación.

1. En PowerShell ISE, abra ...\\Learning Modules\\**UserConfig.psm1**
1. Actualice **ResourceGroupName** y **Name** con los valores específicos para la implementación (solo en las líneas 10 y 11).
1. Guarde los cambios.

A estos valores se hacen referencia en casi todos los scripts.

## <a name="run-the-application"></a>Ejecución de la aplicación

La aplicación presenta lugares que albergan eventos. Entre los tipos de lugares hay salas de concierto, clubs de jazz y clubs deportivos. En Wingtip Tickets, los lugares se registran como inquilinos. El hecho de ser un inquilino ofrece a un lugar una manera sencilla de enumerar eventos y vender entradas para estos a sus clientes. Cada lugar obtiene un sitio web personalizado para mostrar sus eventos y vender entradas.

Dentro de la aplicación, cada inquilino obtiene una base de datos implementada en un grupo elástico.

Una página de **Event Hubs** central proporciona una lista de vínculos a los inquilinos en su implementación.

1. Use la dirección URL para abrir Event Hubs en el explorador web: http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net. Sustituya &lt;usuario&gt; por el valor de usuario de la implementación.

    ![Centro de eventos](./media/saas-dbpertenant-get-started-deploy/events-hub.png)

2. Seleccione **Fabrikam Jazz Club** en el centro de eventos.

    ![Eventos](./media/saas-dbpertenant-get-started-deploy/fabrikam.png)

### <a name="azure-traffic-manager"></a>Administrador de tráfico de Azure

La aplicación Wingtip usa  [*Azure Traffic Manager*](../../traffic-manager/traffic-manager-overview.md) para controlar la distribución de solicitudes entrantes. La dirección URL para acceder a la página de eventos para un inquilino específico utiliza el formato siguiente:

- http://events.wingtip-dpt.&lt;usuario&gt;.trafficmanager.net/fabrikamjazzclub

    En la tabla siguiente se explican los elementos del formato anterior.

    | Elemento de URL        | Descripción       |
    | :-------------- | :---------------- |
    | events.wingtip-dpt | Elementos de eventos de la aplicación Wingtip.<br /><br /> *-dpt* distingue la implementación de *base de datos por inquilino* de Wingtip Tickets de otras implementaciones. Algunos ejemplos incluyen la implementación *única* de aplicación por inquilino ( *-sa*) o de *base de datos multiinquilino* ( *-mt*). |
    | . *&lt;usuario&gt;* | *af1* en el ejemplo. |
    | .trafficmanager.NET/ | Traffic Manager, URL base. |
    | fabrikamjazzclub | Identifica el inquilino denominado Fabrikam Jazz Club. |
    | &nbsp; | &nbsp; |

- La aplicación de eventos analiza el nombre del inquilino a partir de la dirección URL.
- El nombre del inquilino se usa para crear una clave.
- La clave se usa para acceder al catálogo y obtener la ubicación de la base de datos del inquilino.
  - El catálogo se implementa mediante la *administración de asignaciones de particiones*.
- Events Hub usa metadatos extendidos en el catálogo para generar las direcciones URL de las páginas de lista de eventos para cada inquilino.

En un entorno de producción, lo habitual sería crear un registro de DNS CNAME que  [*apunte un dominio de Internet de empresa*](../../traffic-manager/traffic-manager-point-internet-domain.md)  al nombre DNS de Traffic Manager.

> [!NOTE]
> Puede no ser obvio de inmediato cuál es el uso del administrador de tráfico en este tutorial. El objetivo de esta serie de tutoriales es mostrar patrones que pueden administrar la escala de un entorno de producción complejo. En tal caso, por ejemplo, podría tener varias aplicaciones web que se distribuyen en todo el mundo y que comparten ubicación con bases de datos, por lo que necesitaría el administrador de tráfico para enrutar entre estas instancias.
Otro conjunto de tutoriales que ilustra el uso del administrador de tráfico son los tutoriales de [restauración geográfica](../../sql-database/saas-dbpertenant-dr-geo-restore.md) y de [replicación geográfica](../../sql-database/saas-dbpertenant-dr-geo-replication.md). En estos tutoriales, el administrador de tráfico se utiliza para ayudar a cambiar a una instancia de recuperación de la aplicación SaaS si se produce una interrupción regional.

## <a name="start-generating-load-on-the-tenant-databases"></a>Empiece a generar carga en las bases de datos de los inquilinos

Ahora que se implementó la aplicación, vamos a ponerla en marcha.

El script de PowerShell *Demo-LoadGenerator* inicia una carga de trabajo que se ejecuta en las bases de datos de todos los inquilinos. La carga real en muchas aplicaciones de SaaS es esporádica e imprevisible. Para simular este tipo de carga, el generador produce una carga con picos o ráfagas de actividad en cada inquilino. Las ráfagas tienen lugar a intervalos aleatorios. El modelo de carga tarda varios minutos en emerger. Permita que el generador se ejecute durante al menos tres o cuatro minutos antes de supervisar la carga.

1. En PowerShell ISE, abra el script ...\\Learning Modules\\Utilities\\*Demo-LoadGenerator.ps1*.
2. Presione F5 para ejecutar el script e iniciar el generador de carga. Deje los valores predeterminados de parámetro por ahora.
3. Inicie sesión en su cuenta de Azure y, si es necesario, seleccione la suscripción que desea usar.

El script del generador de carga inicia un trabajo en segundo plano para cada base de datos en el catálogo y, a continuación, se detiene. Si vuelve a ejecutar el script del generador de carga, detiene los trabajos en segundo plano que se estén ejecutando antes de iniciar los nuevos.

### <a name="monitor-the-background-jobs"></a>Supervisar los trabajos en segundo plano

Si quiere controlar y supervisar los trabajos en segundo plano, use los cmdlets siguientes:

- `Get-Job`
- `Receive-Job`
- `Stop-Job`

### <a name="demo-loadgeneratorps1-actions"></a>Acciones de Demo-LoadGenerator.ps1

*Demo-LoadGenerator.ps1* imita una carga de trabajo activa de transacciones de cliente. Los pasos siguientes describen la secuencia de acciones que inicia *Demo-LoadGenerator.ps1*:

1. *Demo-LoadGenerator.ps1* inicia *LoadGenerator.ps1* en primer plano.

    - Ambos archivos. ps1 se almacenan en las carpetas Learning Modules\\Utilities\\.

2. *LoadGenerator.ps1* recorre en bucle todas las bases de datos de inquilino del catálogo.

3. *LoadGenerator.ps1* inicia un trabajo de PowerShell en segundo plano para cada base de datos de inquilinos:

    - De manera predeterminada, los trabajos en segundo plano se ejecutan durante 120 minutos.
    - Cada trabajo provoca una carga basada en CPU en una base de datos de inquilinos al ejecutar *sp_CpuLoadGenerator*. La intensidad y la duración de la carga varía según `$DemoScenario`.
    - *sp_CpuLoadGenerator* recorre en bucle una instrucción SQL SELECT que provoca una carga elevada de CPU. El intervalo de tiempo entre las emisiones de SELECT varía en función de los valores de los parámetros para crear una carga de CPU controlable. Los niveles de carga y los intervalos son aleatorios para simular cargas más realistas.
    - Este archivo .sql está almacenado en *WingtipTenantDB\\dbo\\StoredProcedures\\* .

4. Si `$OneTime = $false`, el generador de carga inicia los trabajos en segundo plano y sigue ejecutándose. Cada 10 segundos, supervisa si se aprovisionó algún inquilino nuevo. Si establece `$OneTime = $true`, LoadGenerator inicia los trabajos en segundo plano y luego detiene la ejecución en primer plano. Para este tutorial, deje `$OneTime = $false`.

   Use Ctrl-C o detener la operación Ctrl-Interrumpir si desea detener o reiniciar el generador de carga.

   Si deja el generador de carga ejecutándose en primer plano, utilice otra instancia de PowerShell ISE para ejecutar otros scripts de PowerShell.

&nbsp;

Antes de continuar con la siguiente sección, deje el generador de carga en ejecución en el estado de invocación de trabajo.

## <a name="provision-a-new-tenant"></a>Aprovisionamiento de un nuevo inquilino

La implementación inicial crea tres inquilinos de ejemplo. Ahora creará otro inquilino para ver cómo afecta esto a la aplicación implementada. En la aplicación Wingtip, se explica el flujo de trabajo para aprovisionar nuevos inquilinos en el [tutorial de catalogación y aprovisionamiento](saas-dbpertenant-provision-and-catalog.md). En esta fase, se crea a un nuevo inquilino, lo que lleva menos de un minuto.

1. Abra un nuevo PowerShell ISE.
2. Abra ...\\Learning Modules\Provision and Catalog\\*Demo-ProvisionAndCatalog.ps1*.
3. Presione F5 para ejecutar el script. Deje los valores predeterminados por ahora.

   > [!NOTE]
   > Muchos scripts SaaS de Wingtip usan *$PSScriptRoot* para examinar las carpetas para llamar a funciones en otros scripts. Esta variable solo se evalúa cuando se ejecuta el script completo presionando F5. Resaltar y ejecutar una selección con F8 puede provocar errores. Para ejecutar los scripts, presione F5.

La base de datos de inquilino nueva se:

- Crea en un grupo elástico de SQL.
- Inicializa.
- Registra en el catálogo.

Una vez completado el aprovisionamiento correctamente, el sitio *Events* del nuevo inquilino aparece en el explorador.

![Nuevo inquilino](./media/saas-dbpertenant-get-started-deploy/red-maple-racing.png)

Actualice el centro de eventos para que el nuevo inquilino aparezca en la lista.

## <a name="explore-the-servers-pools-and-tenant-databases"></a>Explore los servidores, los grupos y las bases de datos de los inquilinos

Ahora que ha empezado a ejecutar una carga en la colección de inquilinos, veamos algunos de los recursos que se han implementado.

1. En  [Azure Portal](https://portal.azure.com), vaya a la lista de los servidores SQL. Luego abra el servidor **catalog-dpt-&lt;USUARIO&gt;** .
    - El servidor de catálogos contiene dos bases de datos, **tenantcatalog** y **basetenantdb** (una base de datos de plantilla que se copia para crear nuevos inquilinos).

   ![Bases de datos](./media/saas-dbpertenant-get-started-deploy/databases.png)

2. Vuelva a la lista de servidores SQL Server.

3. Abra el servidor **tenants1-dpt-&lt;USUARIO&gt;** que contiene las bases de datos de los inquilinos.

4. Observe los siguientes elementos:

    - Todas las bases de datos de inquilinos son **elásticas estándar** en un conjunto estándar de 50 eDTU.
    - La base de datos Red Maple Racing es la base de datos de inquilino que aprovisionó anteriormente.

   ![Servidor con bases de datos](./media/saas-dbpertenant-get-started-deploy/server.png)

## <a name="monitor-the-pool"></a>Supervisión del grupo

Después de ejecutar *LoadGenerator.ps1* durante varios minutos, debería haber suficientes datos disponibles para empezar a observar ciertas funciones de supervisión. Estas funciones se integran en grupos y bases de datos.

Vaya al servidor **tenants1-dpt-&lt;usuario&gt;** y seleccione **Pool1** para ver el uso de recursos del grupo. En los siguientes gráficos, el generador de carga se ejecutó durante una hora.

   ![Supervisión del grupo](./media/saas-dbpertenant-get-started-deploy/monitor-pool.png)

- El primer gráfico, con la etiqueta **Uso de recursos** muestra el uso de eDTU del grupo.
- El segundo gráfico muestra el uso de eDTU de las cinco bases de datos más activas del grupo.

Estos dos gráficos ilustran que los grupos elásticos y SQL Database funcionan bien para las cargas de trabajo impredecibles de una aplicación SaaS. Los gráficos muestran 4 bases de datos que emiten ráfagas de hasta 40 eDTU por base de datos y, aún así, un grupo de 50 eDTU las admite cómodamente. El grupo de 50 eDTU puede admitir cargas de trabajo incluso superiores. Si las bases de datos se aprovisionaron como únicas, cada una debe ser de tipo S2 (50 DTU) para admitir las ráfagas. El costo de 4 bases de datos S2 únicas es casi tres veces superior al del grupo. En situaciones reales, los clientes de SQL Database ejecutan hasta 500 bases de datos en grupos de 200 eDTU. Para más información consulte el [tutorial de supervisión de rendimiento](saas-dbpertenant-performance-monitoring.md).

## <a name="additional-resources"></a>Recursos adicionales

- Para más información, consulte los [tutoriales adicionales que se basan en el patrón de base de datos por inquilino de la aplicación SaaS Wingtip Tickets](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials).
- Para información sobre los grupos elásticos, consulte  [¿Qué son los grupos elásticos de Azure SQL?](elastic-pool-overview.md)
- Para información sobre los trabajos elásticos, consulte  [Administración de bases de datos escaladas horizontalmente en la nube](../../sql-database/elastic-jobs-overview.md).
- Para más información sobre las aplicaciones SaaS multiinquilino, consulte  [Patrones de inquilinato de base de datos SaaS multiinquilino](saas-tenancy-app-design-patterns.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha obtenido información:

> [!div class="checklist"]
> - Implementación de la aplicación SaaS Wingtip Tickets.
> - Acerca de los servidores, los grupos y las bases de datos que componen la aplicación.
> - Cómo se asignan los inquilinos a sus datos con el *catálogo*.
> - Acerca de cómo aprovisionar nuevos inquilinos.
> - Visualización del uso de grupos para supervisar la actividad de los inquilinos
> - Información sobre cómo eliminar los recursos de ejemplo para detener la facturación relacionada con ellos.

A continuación, pruebe el [tutorial de aprovisionamiento y catalogación](saas-dbpertenant-provision-and-catalog.md).

<!-- Link references. -->

[github-wingtip-dpt]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant
