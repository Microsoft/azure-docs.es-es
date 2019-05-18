---
title: 'Tutorial: Uso de Azure Database Migration Service para realizar una migración en línea de MySQL a Azure Database for MySQL | Microsoft Docs'
description: Aprenda a realizar una migración en línea de MySQL local a Azure Database for MySQL mediante Azure Database Migration Service.
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
ms.openlocfilehash: 409fa3a501b80e225fff299980f374f73f0dbf13
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415728"
---
# <a name="tutorial-migrate-mysql-to-azure-database-for-mysql-online-using-dms"></a>Tutorial: Migración de MySQL a Azure Database for MySQL en línea mediante DMS

Puede usar Azure Database Migration Service para migrar las bases de datos de una instancia de MySQL local a [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/) con un tiempo de inactividad mínimo. En otras palabras, se puede lograr la migración con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **Employees** de una instancia local de MySQL 5.7 a Azure Database for MySQL mediante una actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Migre el esquema de ejemplo mediante la utilidad mysqldump.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* Descargue e instale [MySQL community edition](https://dev.mysql.com/downloads/mysql/) 5.6 o 5.7. La versión de MySQL local debe coincidir con la versión de Azure Database for MySQL. Por ejemplo, MySQL 5.6 solo puede migrar a Azure Database for MySQL 5.6 y no a la versión actualizada 5.7.
* [Creación de una instancia en Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal). Consulte el artículo [Uso de MySQL Workbench para conectarse y consultar datos](https://docs.microsoft.com/azure/mysql/connect-workbench) para más información acerca de cómo conectarse y crear una base de datos mediante Azure Portal.  
* Crear una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de redes con Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque la instancia de Azure Database Migration Service carece de conectividad a internet.

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445, 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor MySQL de origen que, de manera predeterminada, es el puerto TCP 3306.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) en el nivel de servidor para que Azure Database for MySQL permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcionar el rango de subred de la red virtual usada para Azure Database Migration Service.
* La versión de la instancia de MySQL de origen debe ser compatible con MySQL community edition. Para determinar la versión de la instancia de MySQL, en la utilidad de MySQL o en MySQL Workbench, ejecute el siguiente comando:

    ```
    SELECT @@version;
    ```

* Azure Database for MySQL solo es compatible con tablas de InnoDB. Para convertir tablas de MyISAM a InnoDB, consulte el artículo [Converting Tables from MyISAM to InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html) (Cómo convertir tablas de MyISAM a InnoDB)

* Habilite el registro binario en el archivo my.ini (Windows) o en el archivo my.cnf (Unix) en la base de datos de origen con la configuración siguiente:

    * **server_id** = 1 o mayor (pertinente solo para MySQL 5.6)
    * **log-bin** =\<path> (pertinente solo para MySQL 5.6)

        Por ejemplo: log-bin = E:\MySQL_logs\BinLog
    * **binlog_format** = row
    * **Expire_logs_days** = 5 (se recomienda no usar cero; pertinente solo para MySQL 5.6)
    * **Binlog_row_image** = full (pertinente solo para MySQL 5.6)
    * **log_slave_updates** = 1

* El usuario debe tener el rol ReplicationAdmin con los siguientes privilegios:

    * **REPLICATION CLIENT** (CLIENTE DE REPLICACIÓN): necesario solo para tareas de procesamiento de cambios. En otras palabras, las tareas solo de carga completa no requieren este privilegio.
    * **REPLICATION REPLICA** (RÉPLICA DE REPLICACIÓN): necesario solo para tareas de procesamiento de cambios. En otras palabras, las tareas solo de carga completa no requieren este privilegio.
    * **SUPER** (SÚPER): solo se requiere en las versiones anteriores a MySQL 5.6.6.

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo

Para completar todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados, se debe extraer el esquema de la base de datos de origen y aplicarlo a la base de datos. Para extraer el esquema, puede usar mysqldump con el parámetro `--no-data`.

Suponiendo que tiene la base de datos de ejemplo de **empleados** de MySQL en el sistema local, el comando para realizar la migración del esquema mediante mysqldump es:

```
mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
```

Por ejemplo: 

```
mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
```

Para importar el esquema en la instancia de Azure Database for MySQL de destino, ejecute el siguiente comando:

```
mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
 ```

Por ejemplo: 

```
mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
 ```

Si tiene claves externas en el esquema, se producirá un error en la carga inicial y la sincronización continua de la migración.  Ejecute el siguiente script en MySQL Workbench para extraer el script de colocación de clave externa y agregar el script de clave externa.

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
  AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries
  GROUP BY SchemaName;
 ```

Ejecute el script de colocación de clave externa (que es el de la segunda columna) en el resultado de la consulta para colocar la clave externa.

> [!IMPORTANT]
> Si importa datos mediante una copia de seguridad, quite los comandos CREATE DEFINER manualmente o mediante el comando --skip-definer durante una operación mysqldump. DEFINER requiere privilegios de superusuarios para crear y está restringido en Azure Database for MySQL.

Si tiene un desencadenador en los datos (desencadenador de inserción o de actualización), aplicará la integridad de datos en el destino antes que en los datos replicados desde el origen. Es recomendable deshabilitar los desencadenadores en todas las tablas del destino durante la migración y, a continuación, habilitar los desencadenadores una vez finalizada esta.

Para deshabilitar los desencadenadores en la base de datos de destino, use el siguiente comando:

```
SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-mysql-to-azure-mysql-online/portal-select-subscriptions.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-mysql-to-azure-mysql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-mysql-to-azure-mysql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Creación de una instancia de DMS

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-mysql-to-azure-mysql-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de SQL Server de origen y a la instancia de Azure SQL Database de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

5. Seleccione un plan de tarifa.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-settings3.png)

6. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

     ![Busque la instancia de Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-instance-search.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).
4. En la pantalla **New Migration Project** (Nuevo proyecto de migración), especifique un nombre para el proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **MySQL** y, en el cuadro de texto **Target server type** (Tipo de servidor de destino), elija **AzureDbForMySQL**.
5. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**

    ![Creación de un proyecto de Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-create-project4.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

6. Seleccione **Guardar**, anote los requisitos para usar correctamente DMS para migrar datos y, a continuación, seleccione **Crear y ejecutar una actividad**.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Agregar detalles de origen**, especifique los detalles de conexión de la instancia de MySQL de origen.

    ![Pantalla Agregar detalles de origen](media/tutorial-mysql-to-azure-mysql-online/dms-add-source-details.png)

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y después, en la pantalla **Detalles del destino**, especifique los detalles de conexión del servidor de Azure Database for MySQL de destino, que es la instancia de Azure Database for MySQL que se aprovisionó previamente en la que se implementó el esquema **Employees** mediante mysqldump.

    ![Pantalla de detalles de destino](media/tutorial-mysql-to-azure-mysql-online/dms-add-target-details.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-mysql-to-azure-mysql-online/dms-map-target-details.png)

3. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-mysql-to-azure-mysql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **Completado**.

     ![Estado de actividad Completado](media/tutorial-mysql-to-azure-mysql-online/dms-activity-completed.png)

2. En **Nombre de la base de datos**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    La carga completa de los datos muestra el estado de migración de la carga inicial mientras que la sincronización de datos incrementales muestra el estado de la captura de datos modificados (CDC).

     ![Estado de actividad: carga completa efectuada](media/tutorial-mysql-to-azure-mysql-online/dms-activity-full-load-completed.png)

     ![Estado de actividad: sincronización de datos incrementales](media/tutorial-mysql-to-azure-mysql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez completada la carga completa inicial, las bases de datos se marcan como **A punto para la migración total**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

    ![Iniciar transición](media/tutorial-mysql-to-azure-mysql-online/dms-start-cutover.png)

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.
3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure SQL de destino.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure Database for MySQL, consulte el artículo [Known issues and workarounds with Azure Database for MySQL online migrations](known-issues-azure-mysql-online.md) (Problemas conocidos y soluciones para las migraciones en línea de Azure Database for MySQL).
* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para más información sobre Azure Database for MySQL, consulte el artículo [¿Qué es el servicio Azure SQL Database?](https://docs.microsoft.com/azure/mysql/overview)
