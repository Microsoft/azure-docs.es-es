---
title: Tutorial de la aplicación SaaS de inquilino único
description: Implemente y explore una aplicación SaaS de inquilino único independiente que usa Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 11/07/2018
ms.openlocfilehash: 52dfc285deaa84792e37d0f012abd7702d856113
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87544088"
---
# <a name="deploy-and-explore-a-standalone-single-tenant-application-that-uses-azure-sql-database"></a>Implementación y dexploración de una aplicación de inquilino único independiente que usa Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial, implementará y explorará la aplicación SaaS Wingtip Tickets de ejemplo desarrollada con el patrón de aplicación independiente (o aplicación por inquilino).  La aplicación está diseñada para exhibir las características de Azure SQL Database que simplifican escenarios de SaaS multiinquilino.

El patrón de aplicación independiente (o aplicación por inquilino) implementa una instancia de la aplicación para cada inquilino.  Cada aplicación se configura para un inquilino determinado y se implementa en un grupo de recursos de Azure independiente. Varias instancias de la aplicación se aprovisionan para proporcionar una solución multiinquilino. Este patrón se adapta mejor a una cantidad reducida de inquilinos, cuando el aislamiento de inquilinos es una prioridad. Azure tiene programas para socios que permiten implementar recursos en la suscripción de un inquilino y que un proveedor de servicios los administre en nombre del inquilino. 

En este tutorial, implementará tres aplicaciones independientes para tres inquilinos en su suscripción de Azure.  Tiene acceso completo para explorar y trabajar con los componentes individuales de la aplicación.

El código fuente y los scripts de administración de la aplicación están disponibles en el repositorio [WingtipTicketsSaaS-StandaloneApp](https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp) de GitHub. La aplicación se creó con Visual Studio 2015 y no se abre ni se compila correctamente Visual Studio 2019 si no se actualiza.


En este tutorial, obtendrá información:

> [!div class="checklist"]
> * Implementación de la aplicación independiente SaaS Wingtip Tickets.
> * Dónde obtener el código fuente de la aplicación y los scripts de administración.
> * Información sobre los servidores y las bases de datos que componen la aplicación.

Se publicarán más tutoriales. Estos le permitirán explorar una variedad de escenarios de administración basados en este patrón de aplicación.   

## <a name="deploy-the-wingtip-tickets-saas-standalone-application"></a>Implementación de la aplicación independiente SaaS Wingtip Tickets

Implemente la aplicación para los tres inquilinos proporcionados:

1. Haga clic en cada botón azul **Implementar en Azure** para abrir la plantilla de implementación en [Azure Portal](https://portal.azure.com). Cada plantilla requiere dos valores de parámetro: un nombre para un grupo de recursos nuevo y un nombre de usuario que distingue esta implementación de otras implementaciones de la aplicación. El paso siguiente proporciona detalles para establecer estos valores.

   **Contoso Concert Hall**   
   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](media/saas-standaloneapp-get-started-deploy/deploy.png)](https://aka.ms/deploywingtipsa-contoso)

   **Dogwood Dojo**   
   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](media/saas-standaloneapp-get-started-deploy/deploy.png)](https://aka.ms/deploywingtipsa-dogwood)

   **Fabrikam Jazz Club**   
   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](media/saas-standaloneapp-get-started-deploy/deploy.png)](https://aka.ms/deploywingtipsa-fabrikam)
 
2. Especifique los valores necesarios de los parámetros para cada implementación.

    > [!IMPORTANT]
    > Para realizar la demostración, se ha eliminado intencionadamente la protección de varios firewalls de autenticación y del servidor. **Cree un nuevo grupo de recursos** para cada implementación de aplicación.  No use un grupo de recursos existente. No use esta aplicación, ni ninguno de los recursos que se crean, para producción. Elimine todos los grupos de recursos cuando haya terminado con la aplicación para detener la facturación relacionada con ellos.

    Se recomienda usar solo letras minúsculas, números y guiones en los nombres de recurso.
    * En **Grupo de recursos**, seleccione Crear nuevo y, luego, proporcione un nombre en minúsculas para el grupo de recursos. **wingtip-sa-\<venueName\>-\<user\>** es el patrón recomendado.  Para \<venueName\>, reemplace el nombre del lugar sin espacios. Para \<user\>, reemplace el valor de usuario a continuación.  Con este patrón, los nombres de grupos de recursos podrían ser *wingtip-sa-contosoconcerthall-af1*, *wingtip-sa-dogwooddojo-af1* y *wingtip-sa-fabrikamjazzclub-af1*.
    * Seleccione una **ubicación** en la lista desplegable.

    * En **Usuario**, se recomienda elegir un valor de usuario corto, como sus iniciales más un dígito, por ejemplo: *af1*.


3. **Implemente la aplicación**.

    * Haga clic para aceptar los términos y condiciones.
    * Haga clic en **Comprar**.

4. Supervise el estado de las tres implementaciones, para lo que debe hacer clic en **Notificaciones** (el icono de la campana de la derecha del cuadro de búsqueda). La implementación de las aplicaciones tarda aproximadamente cinco minutos.


## <a name="run-the-applications"></a>Ejecución de las aplicaciones

La aplicación presenta lugares que albergan eventos.  Los lugares son los inquilinos de la aplicación. Cada lugar obtiene un sitio web personalizado para mostrar sus eventos y vender entradas. Entre los tipos de lugares hay salas de concierto, clubs de jazz y clubs deportivos. En el ejemplo, el tipo de lugar determina la fotografía de fondo que se muestra en el sitio web del lugar.   En el modelo de aplicación independiente, cada lugar tiene una instancia de aplicación independiente con su propia instancia de Azure SQL Database independiente.

1. Abra la página de eventos para cada uno de los tres inquilinos en pestañas independientes del explorador:

   - http://events.contosoconcerthall.&lt;usuario&gt;.trafficmanager.net
   - http://events.dogwooddojo.&lt;usuario&gt;.trafficmanager.net
   - http://events.fabrikamjazzclub.&lt;usuario&gt;.trafficmanager.net

     (En cada dirección URL, reemplace &lt;usuario&gt; por el valor de usuario de la implementación).

   ![Eventos](./media/saas-standaloneapp-get-started-deploy/fabrikam.png)

Para controlar la distribución de las solicitudes entrantes, la aplicación usa [*Azure Traffic Manager*](../../traffic-manager/traffic-manager-overview.md). Cada instancia de aplicación específica del inquilino incluye el nombre del inquilino como parte del nombre de dominio de la dirección URL. Todas las direcciones URL de inquilino incluyen el valor específico de **usuario**. Las direcciones URL tienen el formato siguiente:
- http://events.&lt;nombreDeInstalación&gt;.&lt; usuario&gt;.trafficmanager.net

La **ubicación** de la base de datos de cada inquilino se incluye en la configuración de la aplicación implementada correspondiente.

En un entorno de producción, lo habitual sería crear un registro DNS CNAME que [*apunte a un dominio de empresa de Internet*](../../traffic-manager/traffic-manager-point-internet-domain.md) para la dirección URL del perfil de Traffic Manager.


## <a name="explore-the-servers-and-tenant-databases"></a>Exploración de los servidores y las bases de datos de inquilino

Veamos algunos de los recursos que se implementaron:

1. En [Azure Portal](https://portal.azure.com), navegue a la lista de los grupos de recursos.
2. Debería ver los tres grupos de recursos de inquilino.
3. Abra el grupo de recursos **wingtip-sa-fabrikam-&lt;usuario&gt;** que contiene los recursos para la implementación Fabrikam Jazz Club.  El servidor **fabrikamjazzclub-&lt;usuario&gt;** contiene la base de datos **fabrikamjazzclub**.

Cada base de datos de inquilino es una base de datos *independiente* de 50 DTU.

## <a name="additional-resources"></a>Recursos adicionales

<!--
* Additional [tutorials that build on the Wingtip SaaS application](../../sql-database/saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* To learn about elastic pools, see [*What is an Azure SQL elastic pool*](elastic-pool-overview.md)
* To learn about elastic jobs, see [*Managing scaled-out cloud databases*](../../sql-database/elastic-jobs-overview.md)
-->

- Para más información acerca de aplicaciones SaaS multiinquilino, consulte [Modelos de diseño para las aplicaciones SaaS multiinquilino](saas-tenancy-app-design-patterns.md).

 
## <a name="delete-resource-groups-to-stop-billing"></a>Eliminación de grupos de recursos para detener la facturación ##

Cuando haya terminado de utilizar el ejemplo, elimine todos los grupos de recursos que ha creado para detener la facturación asociada.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha obtenido información:

> [!div class="checklist"]
> * Implementación de la aplicación independiente SaaS Wingtip Tickets.
> * Información sobre los servidores y las bases de datos que componen la aplicación.
> * Información sobre cómo eliminar los recursos de ejemplo para detener la facturación relacionada con ellos.

A continuación, pruebe el tutorial de [aprovisionamiento y catálogo](saas-standaloneapp-provision-and-catalog.md) en el que explorará el uso de un catálogo de inquilinos que permite una variedad de escenarios entre inquilinos, como la administración de esquemas y el análisis de inquilinos.
 

