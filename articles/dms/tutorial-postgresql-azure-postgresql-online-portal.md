---
title: 'Tutorial: Migración de PostgreSQL a Azure DB for PostgreSQL en línea mediante Azure Portal'
titleSuffix: Azure Database Migration Service
description: Aprenda a realizar una migración en línea de PostgreSQL local a Azure Database for PostgreSQL mediante Azure Database Migration Service a través de Azure Portal.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: article
ms.date: 04/11/2020
ms.openlocfilehash: 9ccbd22a44f0dfb05e425bff45cdc18803391812
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087652"
---
# <a name="tutorial-migrate-postgresql-to-azure-db-for-postgresql-online-using-dms-via-the-azure-portal"></a>Tutorial: Migración de PostgreSQL a Azure DB for PostgreSQL en línea mediante DMS a través de Azure Portal

Puede usar Azure Database Migration Service para migrar las bases de datos de una instancia de PostgreSQL local a [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/) con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **DVD Rental** de una instancia local de PostgreSQL 9.6 a Azure Database for PostgreSQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Migrar el esquema de ejemplo mediante la utilidad pg_dump.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración en una instancia de Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración
> * Realizar la migración total.

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. El disco se cifra para impedir el robo de datos durante el proceso de migración

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* Descargue e instale [PostgreSQL community edition](https://www.postgresql.org/download/) 9.4, 9.5, 9.6 o 10. La versión del servidor PostgreSQL Server de origen debe ser la 9.4, 9.5, 9.6, 10 u 11. Para más información, consulte el artículo acerca de las [versiones de base de datos admitidas de PostgreSQL](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions).

    Tenga en cuenta también que la versión de Azure Database for PostgreSQL de destino debe ser igual o posterior a la versión local de PostgreSQL. Por ejemplo, PostgreSQL 9.6 puede migrarse a Azure Database for PostgreSQL 9.6, 10 u 11, pero no a Azure Database for PostgreSQL 9.5.

* [Cree un servidor de Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal) o [cree una instancia de Azure Database for PostgreSQL: servidor de Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/quickstart-create-hyperscale-portal).
* Cree una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](https://docs.microsoft.com/azure/virtual-network/)y, especialmente, los artículos de inicio rápido con detalles paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:
    >
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red (NSG) de la red virtual no bloqueen los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445, 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service tenga acceso al servidor PostgreSQL de origen que, de manera predeterminada, es el puerto TCP 5432.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) en el nivel de servidor para que Azure Database for PostgreSQL permita a Azure Database Migration Service tener acceso a las bases de datos de destino. Proporcione el rango de subred de la red virtual que se usa para Azure Database Migration Service.
* Habilite la replicación lógica en el archivo postgresql.config y establezca los parámetros siguientes:

  * wal_level = **logical**
  * max_replication_slots = [número de ranuras], se recomienda establecer en **cinco ranuras**
  * max_wal_senders = [número de tareas simultáneas]; el parámetro max_wal_senders establece el número de tareas simultáneas que puede ejecutar; se recomienda establecerlo en **10 tareas**

> [!IMPORTANT]
> Todas las tablas de la base de datos existente necesitan una clave principal para asegurarse de que todos los cambios se puedan sincronizar en la base de datos de destino.

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo

Para completar todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados, se debe extraer el esquema de la base de datos de origen y aplicarlo a la base de datos.

1. Use el comando pg_dump -s para crear un archivo de volcado de esquema para una base de datos.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Por ejemplo, para crear un archivo de copia de seguridad del esquema para la base de datos **dvdrental**:

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s -O -x > dvdrentalSchema.sql
    ```

    Para más información sobre el uso de la utilidad pg_dump, vea los ejemplos del tutorial [pg volcado](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES).

2. Cree una base de datos vacía en el entorno de destino, que es Azure Database for PostgreSQL.

    Para más información sobre cómo crear una base de datos y conectarse a ella, consulte el artículo [Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL en Azure Portal](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal) o [Inicio rápido: Creación de una instancia de Hiperescala (Citus) de Azure Database for PostgreSQL en Azure Portal](https://docs.microsoft.com/azure/postgresql/quickstart-create-hyperscale-portal).

    > [!NOTE]
    > Una instancia de Azure Database for PostgreSQL: Hiperescala (Citus) solo tiene una base de datos única: **citus**.

3. Importe el esquema en la base de datos de destino que creó restaurando el archivo de volcado de esquema.

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Por ejemplo:

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental citus < dvdrentalSchema.sql
    ```

4. Para extraer el script de clave externa que quiere eliminar y agregarlo en el destino (Azure Database for PostgreSQL), ejecute el siguiente script en PgAdmin o en psql.

   > [!IMPORTANT]
   > Las claves externas en el esquema provocarán un error en la carga inicial y la sincronización continua de la migración.

    ```
    SELECT Q.table_name
        ,CONCAT('ALTER TABLE ', table_schema, '.', table_name, STRING_AGG(DISTINCT CONCAT(' DROP CONSTRAINT ', foreignkey), ','), ';') as DropQuery
            ,CONCAT('ALTER TABLE ', table_schema, '.', table_name, STRING_AGG(DISTINCT CONCAT(' ADD CONSTRAINT ', foreignkey, ' FOREIGN KEY (', column_name, ')', ' REFERENCES ', foreign_table_schema, '.', foreign_table_name, '(', foreign_column_name, ')' ), ','), ';') as AddQuery
    FROM
        (SELECT
        S.table_schema,
        S.foreignkey,
        S.table_name,
        STRING_AGG(DISTINCT S.column_name, ',') AS column_name,
        S.foreign_table_schema,
        S.foreign_table_name,
        STRING_AGG(DISTINCT S.foreign_column_name, ',') AS foreign_column_name
    FROM
        (SELECT DISTINCT
        tc.table_schema,
        tc.constraint_name AS foreignkey,
        tc.table_name,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
        FROM information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu ON tc.constraint_name = kcu.constraint_name AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu ON ccu.constraint_name = tc.constraint_name AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY'
        ) S
        GROUP BY S.table_schema, S.foreignkey, S.table_name, S.foreign_table_schema, S.foreign_table_name
        ) Q
        GROUP BY Q.table_schema, Q.table_name;
    ```

5. Ejecute la clave externa que desea eliminar (que es la segunda columna) en el resultado de la consulta.

6. Para deshabilitar los desencadenadores en la base de datos de destino, ejecute el siguiente script.

   > [!IMPORTANT]
   > Los desencadenadores (de inserción o de actualización) de los datos aplican la integridad de datos en el destino antes de que los datos se repliquen desde el origen. Como resultado, se recomienda deshabilitar los desencadenadores en todas las tablas **del destino** durante la migración y, cuando esta haya terminado, volver a habilitar los desencadenadores.

    ```
    SELECT DISTINCT CONCAT('ALTER TABLE ', event_object_schema, '.', event_object_table, ' DISABLE TRIGGER ', trigger_name, ';')
    FROM information_schema.triggers
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-select-subscriptions.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Creación de una instancia de DMS

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre, la suscripción, un grupo de recursos nuevo o existente y la ubicación del servicio.

4. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso al servidor de PostgreSQL de origen y a la instancia de Azure Database for PostgreSQL de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](https://aka.ms/DMSVnet).

5. Seleccione un plan de tarifa.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-settings4.png)

6. Seleccione **Revisar y crear** para crear el servicio.

   La creación del servicio se completará en un plazo de 10 a 15 minutos.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que ha creado, selecciónela y, luego, elija **+ Nuevo proyecto de migración**.

3. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Tipo de servidor de origen**, seleccione **PostgresSQL** y, en el cuadro de texto **Tipo de servidor de destino**, elija **Azure Database for PostgreSQL**.

4. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

    ![Creación de un proyecto de Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-create-project.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

5. Seleccione **Guardar**, anote los requisitos para usar Azure Database Migration Service correctamente para la migración de los datos y, a continuación, seleccione **Crear y ejecutar actividad**.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Agregar detalles de origen**, especifique los detalles de conexión de la instancia de PostgreSQL de origen.

    ![Pantalla Agregar detalles de origen](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-add-source-details.png)

2. Seleccione **Guardar**.

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. En la pantalla **Detalles de destino**, especifique los detalles de conexión para el servidor de Hiperescala (Citus) de destino, que es la instancia aprovisionada previamente de Hiperescala (Citus) y tiene el esquema **DVD Rentals** implementado mediante pg_dump.

    ![Pantalla de detalles de destino](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-add-target-details.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Pantalla Asignación a bases de datos de destino](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-map-target-databases.png)

3. Seleccione **Guardar** y, a continuación, en la pantalla **Configuración de la migración**, acepte los valores predeterminados.

    ![Pantalla Configuración de la migración](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-migration-settings.png)

4. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Pantalla Resumen de la migración](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración y el **estado** de la actividad debería actualizarse para mostrar **Copia de seguridad en curso**.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **Completado**.

     ![Proceso de supervisión de la migración](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-monitor-migration.png)

2. Cuando se complete la migración, en **Nombre de la base de datos**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

   > [!NOTE]
   > **Carga completa de los datos** muestra el estado de migración de la carga inicial mientras que **Sincronización de datos incrementales** muestra el estado de la captura de datos modificados (CDC).

     ![Detalles de la carga completa de los datos](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-full-data-load-details.png)

     ![Detalles de la sincronización de datos incrementales](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-incremental-data-sync-details.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez completada la carga completa inicial, las bases de datos se marcan como **A punto para la migración total**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

2. Espere a que el contador **Cambios pendientes** muestre **0** para garantizar que todas las transacciones entrantes a la base de datos de origen están detenidas, active la casilla **Confirmar** y, luego, seleccione **Aplicar**.

    ![Pantalla de migración total completa](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-complete-cutover.png)

3. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva instancia de destino de Azure Database for PostgreSQL.

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure Database for PostgreSQL, consulte el artículo [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problemas conocidos y soluciones alternativas con migraciones en línea de Azure Database for PostgreSQL).
* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para más información sobre Azure Database for PostgreSQL, consulte el artículo [¿Qué es Azure Database for PostgreSQL?](https://docs.microsoft.com/azure/postgresql/overview)
