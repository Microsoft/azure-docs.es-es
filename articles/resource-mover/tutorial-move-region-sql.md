---
title: Traslado de recursos de Azure SQL entre regiones mediante Azure Resource Mover
description: Aprenda a trasladar recursos de Azure SQL a otra región mediante Azure Resource Mover
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: e3e2c9aa42ff3189e90f57d7c6e92b2a71f46639
ms.sourcegitcommit: 51df05f27adb8f3ce67ad11d75cb0ee0b016dc5d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90061618"
---
# <a name="tutorial-move-azure-sql-database-resources-to-another-region"></a>Tutorial: Traslado de recursos de Azure SQL Database a otra región

En este tutorial, aprenderá a trasladar grupos elásticos y bases de datos de Azure SQL a otra región de Azure mediante [Azure Resource Mover](overview.md).

> [!NOTE]
> Azure Resource Mover se encuentra actualmente en versión preliminar.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar los requisitos previos y otras exigencias.
> * Seleccionar los recursos que quiere trasladar.
> * Resolver las dependencias de recursos.
> * Preparar el servidor SQL Server y trasladarlo a la región de destino.
> * Preparar y trasladar las bases de datos y los grupos elásticos.
> * Elegir si desea descartar o confirmar el traslado. 
> * Opcionalmente, quitar los recursos de la región después del traslado. 

> [!NOTE]
> Los tutoriales muestran la manera más rápida de probar un escenario y utilizar las opciones predeterminadas. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar. Luego, inicie sesión en el [Portal de Azure](https://portal.azure.com).

## <a name="prerequisites"></a>Requisitos previos

-  Compruebe que tiene acceso como *propietario* a la suscripción que contiene los recursos que desea trasladar.
    - La primera vez que agregue un recurso de un par de origen y destino específicos a una suscripción de Azure, Azure Resource Mover creará una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types), anteriormente llamada identidad de servicio administrado (MSI), en la que confiará la suscripción.
    - Para crear la identidad y asignarle el rol requerido (Colaborador o Administrador de acceso de usuario en la suscripción de origen), la cuenta que utilice para agregar recursos necesita permisos de *propietario* para la suscripción. [Obtenga más información](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) sobre los roles de Azure.
- La suscripción necesita tener cuota suficiente para crear los recursos que va a trasladar a la región de destino. Si no dispone de cuota suficiente, [solicite límites adicionales](/azure/azure-resource-manager/management/azure-subscription-service-limits).
- Compruebe los precios y los cargos asociados con la región de destino a la que va a trasladar los recursos. El uso de la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) le resultará útil.
    

## <a name="check-sql-requirements"></a>Comprobación de los requisitos de SQL

1. [Compruebe](support-matrix-move-region-sql.md) qué características de base de datos o de grupo elástico se admiten para un traslado a otra región.
2. En la región de destino, cree un servidor de destino para cada servidor de origen. [Más información](/azure/azure-sql/database/active-geo-replication-security-configure#how-to-configure-logins-and-users).
4. Si las bases de datos se cifran mediante el cifrado de datos transparente (TDE) y el usuario usa su propia clave de cifrado en Azure Key Vault, [consulte cómo](../key-vault/general/move-region.md) trasladar almacenes de claves a otra región.
5. Si SQL Data Sync está habilitado, es posible trasladar bases de datos miembro. Tras el traslado, debe configurar SQL Data Sync en la nueva base de datos de destino.
6. Antes del traslado, quite la configuración de seguridad de datos avanzada. Después del traslado, [configure los valores](/azure/sql-database/sql-database-advanced-data-security) en el nivel de SQL Server de la región de destino.
7. Si la auditoría está habilitada, las directivas se restablecerán al valor predeterminado después del traslado. [Vuelva a configurar](/azure/sql-database/sql-database-auditing) la auditoría después del traslado.
7. Las directivas de retención de copias de seguridad para la base de datos de origen se transfieren a la base de datos de destino. [Obtenga más información](/azure/sql-database/sql-database-long-term-backup-retention-configure ) sobre cómo modificar los valores después del traslado.
8. Quite las reglas de firewall del nivel de servidor antes del traslado. Las reglas de firewall del nivel de base de datos se copiarán desde el servidor de origen al servidor de destino durante la operación. Después del traslado, [configure las reglas de firewall](/azure/sql-database/sql-database-server-level-firewall-rule) para el servidor SQL Server en la región de destino.
9. Quite la configuración de ajuste automático antes de la operación. [Vuelva a configurar el ajuste automático](/azure/sql-database/sql-database-automatic-tuning-enable) tras la operación.
10. Quite la configuración de alertas de base de datos antes del traslado. [Restablezca](/azure/sql-database/sql-database-insights-alerts-portal) la configuración después de la operación.
    
## <a name="select-resources"></a>Selección de recursos

Seleccione los recursos que desea trasladar.

- Puede seleccionar cualquier tipo de recurso admitido para los grupos de recursos de la región de origen seleccionada.
- Los recursos se trasladarán a una región de destino en la misma suscripción que la región de origen. Si desea cambiar la suscripción, puede hacerlo después de trasladar los recursos.

1. En Azure Portal, busque *resource mover*. A continuación, en **Servicios**, seleccione **Azure Resource Mover**.

     ![Resultados de la búsqueda de "resource mover" en Azure Portal](./media/tutorial-move-region-sql/search.png)

2. En **Información general**, haga clic en **Introducción**.

    ![Botón que permite agregar recursos para trasladarlos a otra región](./media/tutorial-move-region-sql/get-started.png)

3. En **Mover recursos** > **Origen + destino**, seleccione la suscripción y la región de origen.
4. En **Destino**, seleccione la región a la que desea trasladar los recursos. A continuación, haga clic en **Siguiente**.

    ![Página para seleccionar las regiones de origen y de destino](./media/tutorial-move-region-sql/source-target.png)

6. En **Recursos que se van a mover** , haga clic en **Seleccionar recursos**.
7. En **Seleccionar recursos**, seleccione los recursos. Solo es posible agregar recursos que se puedan mover. A continuación, haga clic en **Hecho**.

    ![Página para seleccionar los recursos de SQL que se van a mover](./media/tutorial-move-region-sql/select-resources.png)

8. En **Recursos que se van a mover**, haga clic en **Siguiente**.

9. En **Revisar + agregar**, compruebe la configuración de origen y de destino. Es importante que comprenda que los metadatos de los recursos que se vayan a trasladar se almacenarán en un grupo de recursos creado con esta finalidad en la región de metadatos.


    ![Página para revisar la configuración y realizar el traslado de los recursos](./media/tutorial-move-region-sql/review.png)

10. Haga clic en **Continuar** para empezar a agregar recursos.
11. Una vez que haya terminado de agregarlos correctamente, haga clic en **Adding resources for move** (Agregar recursos para mover) en el icono de notificación.
12. Después de hacer clic en la notificación, revise los recursos de la página **Entre regiones**.


> [!NOTE]
> 
> - El servidor SQL Server tiene ahora el estado *Manual assignment pending* (Asignación manual pendiente).
> - Otros recursos agregados tienen el estado *Prepare pending* (Preparación pendiente).
> - Si desea quitar un recurso de una colección de traslado, el método para hacerlo dependerá del punto en el que se encuentre en dicha operación. [Más información](remove-move-resources.md).

## <a name="resolve-dependencies"></a>Resolución de dependencias


1. En **Entre regiones**, si los recursos muestran el mensaje *Validar dependencias* en la columna **Problemas**, haga clic en el botón **Validar dependencias**. Se iniciará el proceso de validación.
2. Si se encuentran dependencias, haga clic en **Agregar dependencias**.

    ![Botón para agregar dependencias](./media/tutorial-move-region-sql/add-dependencies.png)
   
3. En **Agregar dependencias**, seleccione los recursos dependientes > **Agregar dependencias**. Compruebe el progreso en el área de notificaciones.

4. Agregue más dependencias si es necesario y vuelva a validarlas. 

5. En la página **Entre regiones**, compruebe que los recursos tienen ahora el estado *Prepare pending* (Preparación pendiente) y que no presentan problemas.

    ![Página que muestra los recursos que tienen el estado de preparación pendiente](./media/tutorial-move-region-sql/prepare-pending.png)



## <a name="move-the-sql-server"></a>Traslado del servidor SQL Server

Asigne un servidor SQ Server de destino en la región de destino y confirme el traslado.

### <a name="assign-a-target-sql-server"></a>Asignación de un servidor SQL Server de destino

1. En **Entre regiones**, en la columna **Destination configuration** (Configuración de destino) para el recurso de SQL Server, haga clic en **Recurso no asignado**.
2. Seleccione un recurso de SQL Server existente en la región de destino. 
    
    ![Entrada que muestra el estado de SQL Server para confirmar la operación de traslado pendiente](./media/tutorial-move-region-sql/sql-server-commit-move-pending.png) 

    
> [!NOTE]
> El estado del servidor SQL Server de origen cambia a *Commit move pending* (Confirmar movimiento pendiente). 

### <a name="commit-the-sql-server-move"></a>Confirmación de la operación de movimiento del servidor SQL Server

1. En **Entre regiones**, seleccione el servidor SQL Server y haga clic en **Confirmar movimiento**.
2. En **Confirmar recursos**, haga clic en **Confirmar**.

    ![Página para confirmar la operación de traslado del servidor SQL Server](./media/tutorial-move-region-sql/commit-sql-server.png)

3. Haga un seguimiento del progreso del traslado en la barra de notificaciones.

> [!NOTE]
> Tras la confirmación, el servidor SQL Server tendrá el estado *Delete source pending* (Eliminar origen pendiente).


## <a name="prepare-resources-to-move"></a>Preparación de los recursos que se van a trasladar

Después de haber trasladado el servidor SQL Server de origen, podrá preparar los demás recursos para su traslado.

## <a name="prepare-an-elastic-pool"></a>Preparación de un grupo elástico

1. En **Entre regiones**, seleccione el grupo elástico de origen (demo-test1-elasticpool en este tutorial) y haga clic en **Preparar**.

    ![Botón para preparar los recursos](./media/tutorial-move-region-sql/prepare-elastic.png)

2. En **Preparar recursos**, haga clic en **Preparar**.
3. Cuando las notificaciones indiquen que el proceso de preparación se ha realizado correctamente, haga clic en **Actualizar**.

> [!NOTE]
> El grupo elástico tendrá ahora el estado *Initiate move pending* (Iniciar movimiento pendiente).

## <a name="prepare-a-single-database"></a>Preparación de una base de datos única

1. En **Entre regiones**, seleccione la base de datos única (que no sea parte de un grupo elástico) y haga clic en **Preparar**.

    ![Botón para preparar los recursos seleccionados](./media/tutorial-move-region-sql/prepare-db.png)

2. En **Preparar recursos**, haga clic en **Preparar**.
3. Cuando las notificaciones indiquen que el proceso de preparación se ha realizado correctamente, haga clic en **Actualizar**.

> [!NOTE]
> La base de datos tendrá ahora el estado *Initiate move pending* (Iniciar movimiento pendiente) y se ha creado en la región de destino.


## <a name="move-the-pool-and-prepare-pool-databases"></a>Traslado del grupo y preparación de las bases de datos de este

Para preparar las bases de datos de un grupo elástico, este debe tener el estado *Commit move pending* (Confirmar movimiento pendiente). Para pasar a este estado, inicie el traslado del grupo.

#### <a name="initiate-move---elastic-pool"></a>Iniciar movimiento: grupo elástico

1. En **Entre regiones**, seleccione el grupo elástico de origen (demo-test1-elasticpool en este tutorial) y haga clic en **Iniciar movimiento**.
2. En **Mover recursos**, haga clic en **Iniciar movimiento**.

    
    ![Botón para iniciar el traslado del grupo elástico](./media/tutorial-move-region-sql/initiate-elastic.png)

1. Haga un seguimiento del progreso del traslado en la barra de notificaciones.
1. Cuando las notificaciones indiquen que el traslado se ha realizado correctamente, haga clic en **Actualizar**.

> [!NOTE]
> El grupo elástico se encontrará ahora en el estado *Commit move pending* (Confirmar movimiento pendiente).

#### <a name="prepare-database"></a>Preparación de la base de datos

1. En **Entre regiones**, seleccione la base de datos (demo-test2-SQLdb en este tutorial) y haga clic en **Preparar**.
2. En **Preparar recursos**, haga clic en **Preparar**.

    ![Botón para preparar la base de datos de un grupo elástico](./media/tutorial-move-region-sql/prepare-database-elastic.png) 

Durante la preparación, se crea la base de datos de destino en la región de destino y se inicia la replicación de datos. Tras la preparación, la base de datos tendrá el estado *Initiate move pending* (Iniciar estado pendiente). 

![Botón para preparar la base de datos seleccionada en el grupo elástico](./media/tutorial-move-region-sql/initiate-move-pending.png) 

## <a name="move-databases"></a>Mover bases de datos

Comience a trasladar las bases de datos.
1. En **Entre regiones**, seleccione los recursos que tengan el estado **Initiate move pending** (Iniciar movimiento pendiente). A continuación, haga clic en **Iniciar movimiento**.
2. En **Mover recursos**, haga clic en **Iniciar movimiento**.

    ![Página para iniciar la operación de movimiento](./media/tutorial-move-region-sql/initiate-move.png)

3. Haga un seguimiento del progreso del traslado en la barra de notificaciones.

> [!NOTE]
> Las bases de datos tendrán ahora el estado *Commit move pending* (Confirmar movimiento pendiente).


## <a name="discard-or-commit"></a>¿Descartar o confirmar?

Tras la operación inicial de traslado, puede decidir si desea confirmarla o descartarla. 

- **Descartar**: puede descartar un traslado cuando esté haciendo pruebas y no quiera trasladar verdaderamente el recurso de origen. Si el traslado se descarta, el recurso volverá a tener el estado **Initiate move pending** (Iniciar movimiento pendiente).
- **Confirmación**: la confirmación completa la operación de traslado a la región de destino. Tras la confirmación, un recurso de origen tendrá el estado **Delete source pending** (Eliminar origen pendiente) y el usuario puede decidir si desea eliminarlo.


## <a name="discard-the-move"></a>Descartar la operación de traslado 

Para descartar la operación de traslado, siga estos pasos:

1. En **Entre regiones**, seleccione los recursos que tengan el estado **Commit move pending** (Confirmar movimiento pendiente) y haga clic en **Descartar movimiento**.
2. En **Descartar movimiento**, haga clic en **Descartar**.
3. Haga un seguimiento del progreso del traslado en la barra de notificaciones.


> [!NOTE]
> - Después de haber descartado recursos, las máquinas virtuales tendrán el estado *Initiate move pending* (Iniciar movimiento pendiente).
> - Si solo hay un grupo elástico, la operación de descarte avanzará y se eliminará el grupo elástico creado en la región de destino.
> - Si hay un grupo elástico con bases de datos asociadas que tengan el estado *Commit move pending* (Confirmar movimiento pendiente), no podrá descartar el grupo elástico.
> - Si descarta una base de datos SQL, no se eliminarán los recursos de la región de destino. 

Si desea volver a empezar la operación de traslado después de descartar los recursos, seleccione la base de datos SQL o el grupo elástico y empiece la operación de nuevo.


## <a name="commit-the-move"></a>Confirmación de la operación de traslado

Para finalizar el traslado de bases de datos y grupos elásticos, siga estos pasos:


1. Compruebe que el servidor SQL Server tiene el estado *Delete source pending* (Eliminar origen pendiente).
2. Actualice las cadenas de conexión de la base de datos para que muestren la región de destino antes de confirmar.
3. En **Entre regiones**, seleccione los recursos de SQL y haga clic en **Confirmar movimiento**.
4. En **Confirmar recursos**, haga clic en **Confirmar**.

    ![Confirmar movimiento](./media/tutorial-move-region-sql/commit-sql-resources.png)

5. Haga un seguimiento del progreso de la operación de confirmación en la barra de notificaciones.


> [!NOTE]
> Durante el proceso de confirmación, se produce cierto tiempo de inactividad para las bases de datos SQL
> Las bases de datos y los grupos elásticos que se hayan confirmado tendrán ahora el estado *Delete source pending* (Eliminar origen pendiente).
> Tras la confirmación, actualice los parámetros relacionados con la base de datos, incluidas las reglas de firewall, las directivas y las alertas, en la base de datos de destino.


## <a name="delete-source-resources-after-commit"></a>Eliminación de los recursos de origen después de la confirmación

Después del traslado, puede eliminar los recursos de la región de origen si lo desea. 

1. En **Entre regiones**, haga clic en el nombre de los recursos de origen que desee eliminar.
2. En la página Propiedades de cada recurso, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Trasladar bases de datos de Azure SQL a otra región de Azure.
> * Trasladar grupos elásticos de Azure SQL a otra región.

Ahora veremos cómo trasladar máquinas virtuales de Azure a otra región.

> [!div class="nextstepaction"]
> [Traslado de máquinas virtuales de Azure](./tutorial-move-region-virtual-machines.md)
