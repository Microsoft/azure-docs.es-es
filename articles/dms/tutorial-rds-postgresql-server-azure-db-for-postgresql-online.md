---
title: 'Tutorial: Uso de Azure Database Migration Service para una migración en línea de PostgreSQL de RDS a Azure Database for PostgreSQL | Microsoft Docs'
description: Aprenda a realizar una migración en línea de PostgreSQL de RDS a Azure Database for PostgreSQL mediante Azure Database Migration Service.
services: dms
author: HJToland3
ms.author: jtoland
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: article
ms.date: 05/08/2019
ms.openlocfilehash: 1ee4d546ce823f48a597331276813b42d91e01e4
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415980"
---
# <a name="tutorial-migrate-rds-postgresql-to-azure-database-for-postgresql-online-using-dms"></a>Tutorial: Migración de PostgreSQL de RDS a Azure Database for PostgreSQL en línea mediante DMS

Puede utilizar Azure Database Migration Service para migrar bases de datos de una instancia de PostgreSQL de RDS a [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/) mientras la base de datos de origen permanece en línea durante la migración. En otras palabras, se puede lograr la migración con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **DVD Rental** de una instancia de PostgreSQL 9.6 de RDS a Azure Database for PostgreSQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Migre el esquema de ejemplo mediante la utilidad pg_dump.
> * Cree una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. Para más información, consulte la página de [precios](https://azure.microsoft.com/pricing/details/database-migration/) de Azure Database Migration Service.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe cómo realizar una migración en línea de una instancia local de PostgreSQL a Azure Database for PostgreSQL.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* Descargue e instale [PostgreSQL community edition](https://www.postgresql.org/download/) 9.5, 9.6 o 10. La versión del servidor PostgreSQL Server de origen debe ser la 9.5.11, 9.6.7, 10 o una posterior. Para más información, consulte el artículo acerca de las [versiones de base de datos admitidas de PostgreSQL](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions).

    Además, la versión de PostgreSQL de RDS debe coincidir con la versión de Azure Database for PostgreSQL. Por ejemplo, PostgreSQL 9.5.11.5 de RDS solo puede migrarse a Azure Database for PostgreSQL 9.5.11 y no a la versión 9.6.7.

    > [!NOTE]
    > Para PostgreSQL versión 10, DMS solo admite en la actualidad la migración de la versión 10.3 a Azure Database for PostgreSQL.

* Cree una instancia de [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal). Consulte esta [sección](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal#connect-to-the-postgresql-server-using-pgadmin) del documento para obtener detalles sobre cómo conectarse al servidor PostgreSQL mediante pgAdmin.
* Cree una red virtual de Azure para Azure Database Migration Service usando el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen locales mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.
* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445 y 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service tenga acceso al servidor PostgreSQL de origen que, de manera predeterminada, es el puerto TCP 5432.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) en el nivel de servidor para que Azure Database for PostgreSQL permita a Azure Database Migration Service tener acceso a las bases de datos de destino. Proporcione el intervalo de subred de la red virtual usada para Azure Database Migration Service.

### <a name="set-up-aws-rds-postgresql-for-replication"></a>Configuración de AWS RDS PostgreSQL para la replicación

1. Para crear un nuevo grupo de parámetros, siga las instrucciones proporcionadas por AWS en el artículo [Trabajo con los grupos de parámetros de base de datos](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html).
2. Use el nombre de usuario maestro para conectarse al origen de Azure Database Migration Service. Si usa una cuenta distinta de la cuenta de usuario maestro, la cuenta debe contar con los roles rds_superuser y rds_replication. El rol rds_replication concede permisos para administrar ranuras lógicas y transmitir datos mediante ranuras lógicas.
3. Cree un nuevo grupo de parámetros con la siguiente configuración: a. Establezca el parámetro rds.logical_replication de su grupo de parámetros de base de datos en 1.
    b. max_wal_senders = [número de tareas simultáneas]; el parámetro max_wal_senders establece el número de tareas simultáneas que puede ejecutar; se recomiendan 10 tareas.
    c. max_replication_slots = [número de ranuras]; se recomienda establecer en 5 ranuras.
4. Asocie el grupo de parámetros que creó a la instancia de PostgreSQL de RDS.

## <a name="migrate-the-schema"></a>Migración del esquema

1. Extraiga el esquema de la base de datos de origen y aplíquelo a la base de datos de destino para completar la migración de todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados.

    La forma más fácil de migrar solo el esquema es usar pg_dump con la opción -s. Para obtener más información, consulte los [ejemplos](https://www.postgresql.org/docs/9.6/app-pgdump.html#PG-DUMP-EXAMPLES) en el tutorial de Postgres pg_dump.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Por ejemplo, para volcar un archivo de esquema para la base de datos **dvdrental**, use el siguiente comando:

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

2. Cree una base de datos vacía en el servicio de destino, que es Azure Database for PostgreSQL. Para conectar y crear una base de datos, consulte uno de los siguientes artículos:

    * [Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL en Azure Portal](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)
    * [Create an Azure Database for PostgreSQL using the Azure CLI](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-azure-cli) (Creación de una base de datos Azure Database for PostgreSQL con la CLI de Azure)

3. Importe el esquema al servicio de destino, que es Azure Database for PostgreSQL. Para restaurar el archivo de volcado de esquema, ejecute el siguiente comando:

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Por ejemplo: 

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

4. Si tiene claves externas en el esquema, se producirá un error en la carga inicial y la sincronización continua de la migración. Para extraer el script de clave externa que desea eliminar y agregar el script de clave externa en el destino (Azure Database for PostgreSQL), ejecute el siguiente script en PgAdmin o en psql:

    ```
    SET group_concat_max_len = 8192;
        SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
        FROM
        (SELECT
        KCU.REFERENCED_TABLE_SCHEMA as SchemaName,
        KCU.TABLE_NAME,
        KCU.COLUMN_NAME,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
        WHERE
          KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
          AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
      AND KCU.REFERENCED_TABLE_SCHEMA = 'sakila') Queries
      GROUP BY SchemaName;
    ```

5. Ejecute la clave externa que desea eliminar (que es la de la segunda columna) en el resultado de la consulta para eliminar la clave externa.

6. Si tiene desencadenadores (desencadenador de inserción o de actualización) en los datos, aplicará la integridad de datos en el destino antes de la replicación de los datos desde el origen. Es recomendable deshabilitar los desencadenadores en todas las tablas *del destino* durante la migración y, a continuación, habilitar los desencadenadores una vez completada esta.

    Para deshabilitar los desencadenadores en la base de datos de destino:

    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Creación de una instancia de Azure Database Migration Service

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service.

5. Seleccione una red virtual existente o cree una nueva.

    La VNET proporciona a Azure Database Migration Service acceso a la instancia de PostgreSQL de origen y a la instancia de Azure Database for PostgreSQL de destino.

    Para más información sobre cómo crear una VNET en Azure Portal, consulte el artículo [Guía de inicio rápido: Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

6. Seleccione un plan de tarifa. Para la migración en línea, asegúrese de seleccionar el plan de tarifa Premium: 4vCores.

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-settings4.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

     ![Busque la instancia de Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-instance-search.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).
4. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Tipo de servidor de origen**, seleccione **AWS RDS for PostgreSQL** y, en el cuadro de texto **Tipo de servidor de destino**, elija **Azure Database for PostgreSQL**.
5. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

    > [!IMPORTANT]
    > Asegúrese de seleccionar **Migración de datos en línea**; las migraciones sin conexión no se admiten en este escenario.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-create-project5.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

6. Seleccione **Guardar**.

7. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

    > [!NOTE]
    > Tome nota de los requisitos previos necesarios para configurar la migración en línea en la hoja de creación de proyecto.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

* En la pantalla **Detalles del origen de la migración**, especifique los detalles de conexión de la instancia de PostgreSQL de origen.

   ![Detalles del origen](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-source-details4.png)

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y, a continuación, en la pantalla **Detalles de destino**, especifique los detalles de conexión para el servidor de Azure Database for PostgreSQL de destino, que se ha aprovisionado previamente y tiene el esquema **DVD Rentals** implementado mediante pg_dump.

    ![Selección del destino](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-select-target4.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-map-targets-activity5.png)

3. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-migration-summary1.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **En ejecución**.

    ![Estado de actividad: en ejecución](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-activity-status3.png)

2. En **NOMBRE DE LA BASE DE DATOS**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    **Carga completa de los datos** muestra el estado de migración de la carga inicial mientras que **Sincronización de datos incrementales** muestra el estado de la captura de datos modificados (CDC).

    ![Pantalla de inventario: carga completa de los datos](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-full-load.png)

    ![Pantalla de inventario: sincronización de datos incrementales](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez finalizada la carga completa inicial, las bases de datos se marcan como **Lista para la transición**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

    ![Iniciar transición](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-start-cutover.png)

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.
3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure Database for PostgreSQL de destino.

Su migración en línea de una instancia local de PostgreSQL a Azure Database for PostgreSQL ya está completa.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para información sobre Azure Database for PostgreSQL, consulte el artículo [¿Qué es Azure Database for PostgreSQL?](https://docs.microsoft.com/azure/postgresql/overview)
* Para otras preguntas, envíe un correo electrónico al alias [Ask Azure Database Migrations](mailto:AskAzureDatabaseMigrations@service.microsoft.com).
