---
title: 'Tutorial: Uso de Azure Database Migration Service para una migración en línea de RDS MySQL a Azure Database for MySQL | Microsoft Docs'
description: Aprenda a realizar una migración en línea de RDS MySQL a Azure Database for MySQL mediante Azure Database Migration Service.
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
ms.openlocfilehash: e971fd160a43be088f6d3c4a9fb6fddc7dd769b0
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415672"
---
# <a name="tutorial-migrate-rds-mysql-to-azure-database-for-mysql-online-using-dms"></a>Tutorial: Migración de RDS MySQL a Azure Database for MySQL en línea mediante DMS

Puede utilizar Azure Database Migration Service para migrar bases de datos de una instancia de RDS MySQL a [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/) mientras la base de datos de origen permanece en línea durante la migración. En otras palabras, la migración se puede completar con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **Employees** de una instancia de RDS MySQL a Azure Database for MySQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Migrar el esquema de ejemplo mediante las utilidades mysqldump y mysql.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. Para más información, consulte la página de [precios](https://azure.microsoft.com/pricing/details/database-migration/) de Azure Database Migration Service.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe cómo realizar una migración en línea de una instancia de RDS MySQL a Azure Database for MySQL.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* Asegurarse de que el servidor de origen de MySQL ejecuta una edición de comunidad de MySQL admitida. Para determinar la versión de la instancia de MySQL, en la utilidad mysql o en MySQL Workbench, ejecute el comando:

    ```
    SELECT @@version;
    ```

    Para más información, consulte [Versiones admitidas de servidores de Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-supported-versions).

* Descargue e instale la base de datos de ejemplo de [MySQL **Employees**](https://dev.mysql.com/doc/employee/en/employees-installation.html).
* Cree una instancia de [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal).
* Cree una red virtual de Azure para Azure Database Migration Service usando el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen locales mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.
* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445 y 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor MySQL de origen que, de manera predeterminada, es el puerto TCP 3306.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) en el nivel de servidor para que el servidor de Azure Database for MySQL permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcione el rango de subredes de la red virtual usadas para Azure Database Migration Service.

> [!NOTE]
> Azure Database for MySQL solo es compatible con tablas de InnoDB. Para convertir tablas de MyISAM a InnoDB, consulte el artículo [Converting Tables from MyISAM to InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html) (Conversión de tablas de MyISAM a InnoDB).

### <a name="set-up-aws-rds-mysql-for-replication"></a>Configuración de AWS RDS MySQL para la replicación

1. Para crear un nuevo grupo de parámetros, siga las instrucciones proporcionadas por AWS en el artículo [MySQL Database Log Files](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.MySQL.html) (Archivos de registro de la base de datos MySQL), en la sección **Binary Logging Format** (Formato de registro binario).
2. Cree un nuevo grupo de parámetros con la siguiente configuración:
    * binlog_format = row
    * binlog_checksum = NONE
3. Guarde el nuevo grupo de parámetros.

## <a name="migrate-the-schema"></a>Migración del esquema

1. Extraiga el esquema de la base de datos de origen y aplíquelo a la base de datos de destino para completar la migración de todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados.

    La forma más fácil de migrar solo el esquema es usar mysqldump con el parámetro --no-data. El comando para migrar el esquema es:

    ```
    mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
    ```
    
    Por ejemplo, para volcar un archivo de esquema de la base de datos **Employees**, use el siguiente comando:
    
    ```
    mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
    ```

2. Importe el esquema en el servicio de destino, que es Azure Database for MySQL. Para restaurar el archivo de volcado del esquema, ejecute el siguiente comando:

    ```
    mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
    ```

    Por ejemplo, para importar el esquema de la base de datos **Employees**:

    ```
    mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
    ```

3. Si tiene claves externas en el esquema, se producirá un error en la carga inicial y la sincronización continua de la migración. Para extraer el script de clave externa que desea eliminar y agregar el script de clave externa en el destino (Azure Database for MySQL), ejecute el siguiente script en MySQL Workbench:

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
      AND KCU.REFERENCED_TABLE_SCHEMA = ['SchemaName') Queries
      GROUP BY SchemaName;
    ```

4. Ejecute la clave externa que desea eliminar (que es la segunda columna) en el resultado de la consulta para eliminar la clave externa.

5. Si tiene desencadenadores (desencadenador de inserción o de actualización) en los datos, aplicará la integridad de datos en el destino antes de la replicación de los datos desde el origen. Es recomendable deshabilitar los desencadenadores en todas las tablas *del destino* durante la migración y, a continuación, habilitar los desencadenadores una vez completada esta.

    Para deshabilitar los desencadenadores en la base de datos de destino:

    ```
    select concat ('alter table ', event_object_table, ' disable trigger ', trigger_name)
    from information_schema.triggers;
    ```

6. Si hay instancias del tipo de datos ENUM en las tablas, se recomienda actualizar temporalmente al tipo de datos "character varying" en la tabla de destino. Cuando finalice la replicación de datos, revierta el tipo de datos a ENUM.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Creación de una instancia de Azure Database Migration Service

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service.

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de MySQL de origen y a la instancia de Azure Database for MySQL de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

6. Seleccione un plan de tarifa. Para la migración en línea, asegúrese de seleccionar el plan de tarifa Premium: 4vCores.

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-settings3.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

     ![Busque la instancia de Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-instance-search.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).
4. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Tipo de servidor de origen**, seleccione **MySQL** y, a continuación, en el cuadro de texto **Tipo de servidor de destino**, elija **AzureDbForMySQL**.
5. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

    > [!IMPORTANT]
    > Asegúrese de seleccionar **Migración de datos en línea**; las migraciones sin conexión no se admiten en este escenario.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-create-project6.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

6. Seleccione **Guardar**.

7. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

    > [!NOTE]
    > Tome nota de los requisitos previos necesarios para configurar la migración en línea en la hoja de creación de proyecto.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

* En la pantalla **Detalles del origen de la migración**, especifique los detalles de conexión de la instancia de MySQL de origen.

   ![Detalles del origen](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-source-details5.png)

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y, a continuación, en la pantalla **Detalles de destino**, especifique los detalles de conexión para el servidor de Azure Database for MySQL de destino, que se ha aprovisionado previamente y tiene el esquema **Employees** implementado mediante MySQLDump.

    ![Selección del destino](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-select-target5.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-map-targets-activity5.png)

3. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-migration-summary2.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **En ejecución**.

    ![Estado de actividad: en ejecución](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-activity-status4.png)

2. En **NOMBRE DE LA BASE DE DATOS**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    **Carga completa de los datos** muestra el estado de migración de la carga inicial mientras que **Sincronización de datos incrementales** muestra el estado de la captura de datos modificados (CDC).

    ![Pantalla de inventario: carga completa de los datos](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-full-load.png)

    ![Pantalla de inventario: sincronización de datos incrementales](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez finalizada la carga completa inicial, las bases de datos se marcan como **Lista para la transición**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

    ![Iniciar transición](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-start-cutover.png)

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.
3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure Database for MySQL de destino.

La migración en línea de una instancia local de MySQL a Azure Database for MySQL ya está completa.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para información sobre Azure Database for MySQL, consulte el artículo [¿Qué es Azure Database for MySQL?](https://docs.microsoft.com/azure/mysql/overview)
* Para otras preguntas, envíe un correo electrónico al alias [Ask Azure Database Migrations](mailto:AskAzureDatabaseMigrations@service.microsoft.com).
