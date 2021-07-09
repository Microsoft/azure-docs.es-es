---
title: 'Tutorial: Migración de PostgreSQL a Azure Database for PostgreSQL en línea a través de la CLI de Azure'
titleSuffix: Azure Database Migration Service
description: Aprenda a realizar una migración en línea de PostgreSQL local a Azure Database for PostgreSQL mediante Azure Database Migration Service a través de la CLI.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019, devx-track-azurecli
ms.topic: tutorial
ms.date: 04/11/2020
ms.openlocfilehash: 6384962e0591c4bc3245d0f204f9ad0cec0d1177
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069911"
---
# <a name="tutorial-migrate-postgresql-to-azure-db-for-postgresql-online-using-dms-via-the-azure-cli"></a>Tutorial: Migración de PostgreSQL a Azure DB for PostgreSQL en línea mediante DMS a través de la CLI de Azure

Puede usar Azure Database Migration Service para migrar las bases de datos de una instancia de PostgreSQL local a [Azure Database for PostgreSQL](../postgresql/index.yml) con un tiempo de inactividad mínimo. En otras palabras, la migración se puede completar con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **DVD Rental** de una instancia local de PostgreSQL 9.6 a Azure Database for PostgreSQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
> * Migre el esquema de ejemplo mediante la utilidad pg_dump.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. El disco se cifra para impedir el robo de datos durante el proceso de migración.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* Descargue e instale [PostgreSQL community edition](https://www.postgresql.org/download/) 9.5, 9.6 o 10. La versión del servidor PostgreSQL Server de origen debe ser la 9.5.11, 9.6.7, 10 o una posterior. Para más información, consulte el artículo acerca de las [versiones de base de datos admitidas de PostgreSQL](../postgresql/concepts-supported-versions.md).

    Tenga en cuenta también que la versión de Azure Database for PostgreSQL de destino debe ser igual o posterior a la versión local de PostgreSQL. Por ejemplo, PostgreSQL 9.6 solo puede migrarse a Azure Database for PostgreSQL 9.6, 10 u 11, pero no a Azure Database for PostgreSQL 9.5.

* [Cree una instancia de Azure Database for PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) o [cree una instancia de Azure Database for PostgreSQL: servidor de Hiperescala (Citus)](../postgresql/quickstart-create-hyperscale-portal.md).
* Cree una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](../virtual-network/index.yml)y, especialmente, los artículos de inicio rápido con detalles paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de red a Microsoft, agregue los siguientes [puntos de conexión](../virtual-network/virtual-network-service-endpoints-overview.md) de servicio a la subred en la que se aprovisionará el servicio:
    >
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red (NSG) de la red virtual no bloqueen el puerto 443 de salida de ServiceTag para ServiceBus, Storage y AzureMonitor. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service tenga acceso al servidor PostgreSQL de origen que, de manera predeterminada, es el puerto TCP 5432.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](../postgresql/concepts-firewall-rules.md) en el nivel de servidor para que Azure Database for PostgreSQL permita a Azure Database Migration Service tener acceso a las bases de datos de destino. Proporcione el rango de subred de la red virtual que se usa para Azure Database Migration Service.
* Hay dos métodos para invocar la CLI:

  * En la esquina superior derecha de Azure Portal, seleccione el botón Cloud Shell:

       ![Botón Cloud Shell en Azure Portal](media/tutorial-postgresql-to-azure-postgresql-online/cloud-shell-button.png)

  * Instale y ejecute la CLI localmente. CLI 2.0 es una herramienta de línea de comandos para administrar recursos de Azure.

       Para descargar la CLI, siga las instrucciones del artículo [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). En el artículo también se enumeran las plataformas que admiten CLI 2.0.

       Para configurar el subsistema Windows para Linux (WSL), siga las instrucciones de la [guía de instalación de Windows 10](/windows/wsl/install-win10)

* Habilite la replicación lógica en el archivo postgresql.config y establezca los parámetros siguientes:

  * wal_level = **logical**
  * max_replication_slots = [número de ranuras], se recomienda establecer en **cinco ranuras**
  * max_wal_senders = [número de tareas simultáneas]; el parámetro max_wal_senders establece el número de tareas simultáneas que puede ejecutar; se recomienda establecerlo en **10 tareas**

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo

Para completar todos los objetos de base de datos como esquemas de tabla, índices y procedimientos almacenados, se debe extraer el esquema de la base de datos de origen y aplicarlo a la base de datos.

1. Use el comando pg_dump -s para crear un archivo de volcado de esquema para una base de datos. 

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Por ejemplo, para volcar una base de datos dvdrental de archivo de esquema:
    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

    Para más información sobre el uso de la utilidad pg_dump, vea los ejemplos del tutorial [pg volcado](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES).

2. Cree una base de datos vacía en el entorno de destino, que es Azure Database for PostgreSQL.

    Para más información sobre cómo crear una base de datos y conectarse a ella, consulte el artículo [Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL en Azure Portal](../postgresql/quickstart-create-server-database-portal.md) o [Inicio rápido: Creación de una instancia de Hiperescala (Citus) de Azure Database for PostgreSQL en Azure Portal](../postgresql/quickstart-create-hyperscale-portal.md).

3. Importe el esquema en la base de datos de destino que creó restaurando el archivo de volcado de esquema.

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql 
    ```

    Por ejemplo:

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

  > [!NOTE]
   > El servicio de migración controla internamente la habilitación o deshabilitación de claves externas y desencadenadores para garantizar una migración de datos confiable y sólida. Como resultado, no tiene que preocuparse por realizar modificaciones en el esquema de la base de datos de destino.

## <a name="provisioning-an-instance-of-dms-using-the-azure-cli"></a>Aprovisionamiento de una instancia de DMS mediante la CLI de Azure

1. Instale la extensión de sincronización de dms:
   * Inicie sesión en Azure ejecutando el siguiente comando:
       ```azurecli
       az login
       ```

   * Cuando se le pida, abra un explorador web y escriba un código para autenticar el dispositivo. Siga las instrucciones según se indican.
   * Agregue la extensión de dms:
       * Para enumerar las extensiones disponibles, ejecute el siguiente comando:

           ```azurecli
           az extension list-available –otable
           ```

       * Para instalar la extensión, ejecute el comando siguiente:

           ```azurecli
           az extension add –n dms-preview
           ```

   * Para comprobar que ha instalado la extensión de dms correcta, ejecute el siguiente comando:

       ```azurecli
       az extension list -otable
       ```
       Debería ver la siguiente salida:

       ```output
       ExtensionType    Name
       ---------------  ------
       whl              dms
       ```

      > [!IMPORTANT]
      > Asegúrese de que la versión de la extensión es posterior a la 0.11.0.

   * En cualquier momento, vea todos los comandos admitidos en DMS ejecutando:

       ```azurecli
       az dms -h
       ```

   * Si tiene varias suscripciones de Azure, ejecute el siguiente comando para establecer la suscripción que desea usar para aprovisionar una instancia del servicio DMS.

        ```azurecli
       az account set -s 97181df2-909d-420b-ab93-1bff15acb6b7
        ```

2. Aprovisionar una instancia de DMS, ejecute el comando siguiente:

   ```azurecli
   az dms create -l [location] -n <newServiceName> -g <yourResourceGroupName> --sku-name Premium_4vCores --subnet/subscriptions/{vnet subscription id}/resourceGroups/{vnet resource group}/providers/Microsoft.Network/virtualNetworks/{vnet name}/subnets/{subnet name} –tags tagName1=tagValue1 tagWithNoValue
   ```

   Por ejemplo, el comando siguiente creará un servicio en:

   * Ubicación: Este de EE. UU. 2
   * Suscripción: 97181df2-909d-420b-ab93-1bff15acb6b7
   * Nombre del grupo de recursos: PostgresDemo
   * Nombre del servicio DMS: PostgresCLI

   ```azurecli
   az dms create -l eastus2 -g PostgresDemo -n PostgresCLI --subnet /subscriptions/97181df2-909d-420b-ab93-1bff15acb6b7/resourceGroups/ERNetwork/providers/Microsoft.Network/virtualNetworks/AzureDMS-CORP-USC-VNET-5044/subnets/Subnet-1 --sku-name Premium_4vCores
   ```

   La instancia del servicio DMS tarda en crearse entre 10 y 12 minutos aproximadamente.

3. Para identificar la dirección IP del agente DMS para que se pueda agregar al archivo pg_hba.conf de Postgres, ejecute el siguiente comando:

    ```azurecli
    az network nic list -g <ResourceGroupName>--query '[].ipConfigurations | [].privateIpAddress'
    ```

    Por ejemplo:

    ```azurecli
    az network nic list -g PostgresDemo --query '[].ipConfigurations | [].privateIpAddress'
    ```

    Debe obtener un resultado similar a la siguiente dirección: 

    ```output
    [
      "172.16.136.18"
    ]
    ```

4. Agregue la dirección IP del agente DMS al archivo pg_hba.conf de Postgres.

    * Anote la dirección IP de DMS cuando termine el aprovisionamiento en DMS.
    * Agregue la dirección IP al archivo pg_hba.conf en el origen, de forma similar a la entrada siguiente:

        ```
        host     all            all        172.16.136.18/10    md5
        host     replication    postgres   172.16.136.18/10    md5
        ```

5. A continuación, cree un proyecto de migración de PostgreSQL ejecutando el comando siguiente:
    
    ```azurecli
    az dms project create -l <location> -g <ResourceGroupName> --service-name <yourServiceName> --source-platform PostgreSQL --target-platform AzureDbforPostgreSQL -n <newProjectName>
    ```

    Por ejemplo, el comando siguiente crea un proyecto mediante estos parámetros:

   * Ubicación: Centro-Oeste de EE. UU.
   * Nombre del grupo de recursos: PostgresDemo
   * Nombre del servicio: PostgresCLI
   * Nombre del proyecto: PGMigration
   * Plataforma de origen: PostgreSQL
   * Plataforma de destino: AzureDbForPostgreSql

     ```azurecli
     az dms project create -l westcentralus -n PGMigration -g PostgresDemo --service-name PostgresCLI --source-platform PostgreSQL --target-platform AzureDbForPostgreSql
     ```

6. Cree una tarea de migración de PostgreSQL mediante los siguientes pasos.

    Este paso incluye el uso de la dirección IP de origen, el identificador de usuario y la contraseña, la dirección IP de destino, el identificador de usuario, la contraseña y el tipo de tarea para establecer la conectividad.

   * Para ver una lista completa de opciones, ejecute el comando:

       ```azurecli
       az dms project task create -h
       ```

       Para la conexión de origen y destino, el parámetro de entrada hace referencia a un archivo json que tiene la lista de objetos.

       El formato del objeto JSON de conexión para las conexiones de PostgreSQL.
        
       ```json
       {
                   "userName": "user name",    // if this is missing or null, you will be prompted
                   "password": null,           // if this is missing or null (highly recommended) you will
               be prompted
                   "serverName": "server name",
                   "databaseName": "database name", // if this is missing, it will default to the 'postgres'
               server
                   "port": 5432                // if this is missing, it will default to 5432
               }
       ```

   * También hay un archivo json de opción de base de datos que enumera los objetos json. Para PostgreSQL, el formato del objeto JSON de las opciones de base de datos se muestra a continuación:

       ```json
       [
           {
               "name": "source database",
               "target_database_name": "target database",
           },
           ...n
       ]
       ```

   * Cree un archivo json con el Bloc de notas, copie los comandos siguientes y péguelos en el archivo y luego guarde el archivo en C:\DMS\source.json.

        ```json
       {
                   "userName": "postgres",    
                   "password": null,           
               be prompted
                   "serverName": "13.51.14.222",
                   "databaseName": "dvdrental", 
                   "port": 5432                
               }
        ```

   * Cree otro archivo denominado target.json y guarde como C:\DMS\target.json. Incluya los comandos siguientes:

       ```json
       {
               "userName": " dms@builddemotarget",    
               "password": null,           
               "serverName": " builddemotarget.postgres.database.azure.com",
               "databaseName": "inventory", 
               "port": 5432                
           }
       ```

   * Cree un archivo json de opciones de base de datos que enumere el inventario como la base de datos para migrar:

       ```json
       [
           {
               "name": "dvdrental",
               "target_database_name": "dvdrental",
           }
       ]
       ```

   * Ejecute el siguiente comando, que toma el origen, el destino y los archivos json de opción de base de datos.

       ```azurecli
       az dms project task create -g PostgresDemo --project-name PGMigration --source-platform postgresql --target-platform azuredbforpostgresql --source-connection-json c:\DMS\source.json --database-options-json C:\DMS\option.json --service-name PostgresCLI --target-connection-json c:\DMS\target.json –task-type OnlineMigration -n runnowtask    
       ```

     Llegados a este punto, ha enviado correctamente una tarea de migración.

7. Para mostrar el progreso de la tarea, ejecute el siguiente comando:

   ```azurecli
   az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
   ```

   O BIEN

    ```azurecli
   az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output
    ```

8. También puede consultar migrationState desde la salida de expansión:

    ```azurecli
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output --query 'properties.output[].migrationState | [0]' "READY_TO_COMPLETE"
    ```

En el archivo de salida, hay varios parámetros que indican el progreso de la migración. Por ejemplo, vea el archivo de salida siguiente:

  ```output
    "output": [                                 // Database Level
          {
            "appliedChanges": 0,         // Total incremental sync applied after full load
            "cdcDeleteCounter": 0        // Total delete operation  applied after full load
            "cdcInsertCounter": 0,       // Total insert operation applied after full load
            "cdcUpdateCounter": 0,       // Total update operation applied after full load
            "databaseName": "inventory",
            "endedOn": null,
            "fullLoadCompletedTables": 2,   //Number of tables completed full load
            "fullLoadErroredTables": 0,     //Number of tables that contain migration error
            "fullLoadLoadingTables": 0,     //Number of tables that are in loading status
            "fullLoadQueuedTables": 0,      //Number of tables that are in queued status
            "id": "db|inventory",
            "incomingChanges": 0,           //Number of changes after full load
            "initializationCompleted": true,
            "latency": 0,
            "migrationState": "READY_TO_COMPLETE",    //Status of migration task. READY_TO_COMPLETE means the database is ready for cutover
            "resultType": "DatabaseLevelOutput",
            "startedOn": "2018-07-05T23:36:02.27839+00:00"
          },
          {
            "databaseCount": 1,
            "endedOn": null,
            "id": "dd27aa3a-ed71-4bff-ab34-77db4261101c",
            "resultType": "MigrationLevelOutput",
            "sourceServer": "138.91.123.10",
            "sourceVersion": "PostgreSQL",
            "startedOn": "2018-07-05T23:36:02.27839+00:00",
            "state": "PENDING",
            "targetServer": "builddemotarget.postgres.database.azure.com",
            "targetVersion": "Azure Database for PostgreSQL"
          },
          {                                        // Table 1
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 0,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:20.740701+00:00",    //Full load completed time
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:15.864552+00:00",    //Full load started time
            "fullLoadTotalRows": 10,                     //Number of rows loaded in full load
            "fullLoadTotalVolumeBytes": 7056,            //Volume in Bytes in full load
            "id": "or|inventory|public|actor",
            "lastModifiedTime": "2018-07-05T23:36:16.880174+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",                       //State of migration for this table
            "tableName": "public.catalog",              //Table name
            "totalChangesApplied": 0                    //Total sync changes that applied after full load
          },
          {                                            //Table 2
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 50,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:23.963138+00:00",
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:19.302013+00:00",
            "fullLoadTotalRows": 112,
            "fullLoadTotalVolumeBytes": 46592,
            "id": "or|inventory|public|address",
            "lastModifiedTime": "2018-07-05T23:36:20.308646+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",
            "tableName": "public.orders",
            "totalChangesApplied": 0
          }
        ],                                      // DMS migration task state
        "state": "Running",    //Migration task state – Running means it is still listening to any changes that might come in
        "taskType": null
      },
      "resourceGroup": "PostgresDemo",
      "type": "Microsoft.DataMigration/services/projects/tasks"
  ```

## <a name="cutover-migration-task"></a>Tarea de migración total

La base de datos está lista para la migración total cuando se complete la carga total. Dependiendo de lo ocupado que esté el servidor de origen con las nuevas transacciones, la tarea DMS podría seguir aplicando cambios una vez completada la carga total.

Para garantizar que todos los datos estén al día, valide los recuentos de filas entre las bases de datos de origen y destino. Por ejemplo, puede usar el siguiente comando:

```
"migrationState": "READY_TO_COMPLETE", //Status of migration task. READY_TO_COMPLETE means database is ready for cutover
 "incomingChanges": 0, //continue to check for a period of 5-10 minutes to make sure no new incoming changes that need to be applied to the target server
   "fullLoadTotalRows": 10, //full load for table 1
    "cdcDeleteCounter": 0, //delete, insert and update counter on incremental sync after full load
    "cdcInsertCounter": 50,
    "cdcUpdateCounter": 0,
     "fullLoadTotalRows": 112, //full load for table 2
```

1. Realice la tarea de migración de base de datos de transición mediante el comando siguiente:

    ```azurecli
    az dms project task cutover -h
    ```

    Por ejemplo:

    ```azurecli
    az dms project task cutover --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask  --object-name Inventory
    ```

2. Para supervisar el progreso de transición, ejecute el comando siguiente:

    ```azurecli
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```
3. Cuando aparezca el estado **Completado** de la migración de base de datos, [recree las secuencias](https://wiki.postgresql.org/wiki/Fixing_Sequences) (si procede) y conecte las aplicaciones a la nueva instancia de destino de Azure Database for PostgreSQL.

## <a name="service-project-task-cleanup"></a>Limpieza de servicios, proyectos y tareas

Si necesita cancelar o eliminar cualquier tarea, proyecto o servicio de DMS, realice la cancelación en la siguiente secuencia:

* Cancelar todas las tareas en ejecución
* Eliminar la tarea
* Eliminar el proyecto
* Eliminar el servicio DMS

1. Para cancelar una tarea en ejecución, utilice el siguiente comando:

    ```azurecli
    az dms project task cancel --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
     ```

2. Para eliminar una tarea en ejecución, utilice el siguiente comando:
    ```azurecli
    az dms project task delete --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

3. Para cancelar un proyecto en ejecución, utilice el siguiente comando:
     ```azurecli
    az dms project task cancel -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
     ```

4. Para eliminar un proyecto en ejecución, utilice el siguiente comando:
    ```azurecli
    az dms project task delete -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
    ```

5. Para eliminar un servicio DMS, utilice el siguiente comando:

     ```azurecli
    az dms delete -g ProgresDemo -n PostgresCLI
     ```

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure Database for PostgreSQL, consulte el artículo [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problemas conocidos y soluciones alternativas con migraciones en línea de Azure Database for PostgreSQL).
* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](./dms-overview.md)
* Para más información sobre Azure Database for PostgreSQL, consulte el artículo [¿Qué es Azure Database for PostgreSQL?](../postgresql/overview.md)