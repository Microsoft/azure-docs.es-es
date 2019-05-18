---
title: 'Tutorial: Uso de Azure Database Migration Service para realizar una migración en línea de Oracle a Azure Database for PostgreSQL | Microsoft Docs'
description: Aprenda a migrar bases de datos de Oracle locales o en máquinas virtuales a Azure Database for PostgreSQL mediante Azure Database Migration Service.
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
ms.openlocfilehash: b73249a9f72e4616f6d36e16f110913278f04590
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415599"
---
# <a name="tutorial-migrate-oracle-to-azure-database-for-postgresql-online-using-dms-preview"></a>Tutorial: Migración de Oracle a Azure Database for PostgreSQL en línea mediante DMS (versión preliminar)

Puede usar Azure Database Migration Service para migrar las bases de datos de Oracle locales o en máquinas virtuales a [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/) con un tiempo de inactividad mínimo. En otras palabras, la migración se puede completar con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **HR** de una instancia local o en máquina virtual de Oracle 11g a Azure Database for PostgreSQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Evaluar el esfuerzo de migración con la herramienta ora2pg.
> * Migrar el esquema de ejemplo con la herramienta ora2pg.
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe cómo realizar una migración en línea de Oracle a Azure Database for PostgreSQL.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* Descargue e instale [Oracle 11g Release 2 (Standard Editionde Standard Edition One o Enterprise Edition)](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html).
* Descargue la base de datos de ejemplo **HR** [aquí](https://docs.oracle.com/database/121/COMSC/installation.htm#COMSC00002).
* Descargue e instale ora2pg en [Windows](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows.pdf) o [Linux](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Linux.pdf).
* [Creación de una instancia en Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal).
* Conéctese a la instancia y cree una base de datos con las instrucciones de este [documento](https://docs.microsoft.com/azure/postgresql/tutorial-design-database-using-azure-portal).
* Cree una red virtual de Azure para Azure Database Migration Service usando el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen locales mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.

  > [!NOTE]
  > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de redes con Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:
  > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
  > * Punto de conexión de Storage
  > * Punto de conexión de Service Bus
  >
  > Esta configuración es necesaria porque Azure Database Migration Service no tiene conexión a Internet.

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445, 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor Oracle de origen que, de manera predeterminada, es el puerto TCP 1521.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) en el nivel de servidor para que Azure Database for PostgreSQL permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcione el rango de subredes de la red virtual usadas para Azure Database Migration Service.
* Habilite el acceso a las bases de datos de Oracle de origen.

  > [!NOTE]
  > Los usuarios necesitan tener el rol DBA para poder conectarse al origen de Oracle.

  * Se necesitan registros de recuperación archivados para que la sincronización incremental en Azure Database Migration Service pueda capturar los cambios en los datos. Siga estos pasos para configurar el origen de Oracle:
    * Ejecute el comando siguiente para iniciar sesión con privilegios SYSDBA:

      ```
      sqlplus (user)/(password) as sysdba
      ```

    * Ejecute el comando siguiente para cerrar la instancia de la base de datos.

      ```
      SHUTDOWN IMMEDIATE;
      ```

      Espere la confirmación `'ORACLE instance shut down'`.

    * Inicie la nueva instancia y monte (sin abrir) la base de datos para habilitar o deshabilitar el archivado; para ello, ejecute el comando siguiente:

      ```
      STARTUP MOUNT;
      ```

      Se debe montar la base de datos; espere la confirmación "Instancia de Oracle iniciada".

    * Ejecute el comando siguiente para camiar el modo de archivado de la base de datos:

      ```
      ALTER DATABASE ARCHIVELOG;
      ```

    * Ejecute el comando siguiente para abrir la base de datos para las operaciones normales:

      ```
      ALTER DATABASE OPEN;
      ```

      Es posible que deba reiniciar para que se muestre el archivo ARC.

    * Para comprobarlo, ejecute el siguiente comando:

      ```
      SELECT log_mode FROM v$database;
      ```

      Recibirá una respuesta `'ARCHIVELOG'`. Si la respuesta es `'NOARCHIVELOG'`, no se cumple el requisito.

  * Habilite un registro complementario para la replicación con una de las siguientes opciones.

    * **Opción 1**.
      Cambie el registro complementario en el nivel de base de datos para que incluya todas las tablas con índice único y clave principal. La consulta de detección devolverá `'IMPLICIT'`.

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY, UNIQUE) COLUMNS;
      ```

      Cambie el registro complementario en el nivel de tabla. Ejecútelo solo para las tablas que tengan manipulación de datos y que no tengan claves principales o índices únicos.

      ```
      ALTER TABLE [TABLENAME] ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    * **Opción 2**.
      Cambie el registro complementario en el nivel de base de datos para que incluya todas las tablas, y la consulta de detección devuelva `'YES'`.

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
      ```

      Cambie el registro complementario en el nivel de tabla. Siga esta lógica para ejecutar solo una instrucción para cada tabla.

      Si la tabla tiene una clave principal:

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
      ```

      Si la tabla tiene un índice único:

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG GROUP (first unique index columns) ALWAYS;
      ```

      De lo contrario, ejecute el siguiente comando:

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    Para comprobarlo, ejecute el siguiente comando:

      ```
      SELECT supplemental_log_data_min FROM v$database;
      ```

    Recibirá una respuesta `'YES'`.

> [!IMPORTANT]
> En la versión preliminar pública de este escenario, Azure Database Migration Service admite la versión de Oracle 10g u 11g. Los clientes que ejecutan Oracle versión 12 o posteriores deben tener en cuenta que el protocolo de autenticación mínimo permitido para que el controlador ODBC se conecte a Oracle es 8. Para un origen de Oracle de la versión 12C o posteriores, debe configurar el protocolo de autenticación como sigue:
>
> * Actualice SQLNET.ORA:
>
>    ```
>    SQLNET.ALLOWED_LOGON_VERSION_CLIENT = 8
>    SQLNET.ALLOWED_LOGON_VERSION_SERVER = 8
>    ```
>
> * Reinicie el equipo para que la nueva configuración surta efecto.
> * Cambie la contraseña para los usuarios existentes:
>
>    ```
>    ALTER USER system IDENTIFIED BY {pswd}
>    ```
>
>   Consulte [esta página](http://www.dba-oracle.com/t_allowed_login_version_server.htm) para obtener más información.
>
> Por último, recuerde que cambiar el protocolo de autenticación puede afectar a la autenticación de cliente.

## <a name="assess-the-effort-for-an-oracle-to-azure-database-for-postgresql-migration"></a>Evaluación del esfuerzo de la migración de Oracle a Azure Database for PostgreSQL

Se recomienda usar ora2pg para evaluar el esfuerzo necesario para migrar desde Oracle a Azure Database for PostgreSQL. Use la directiva `ora2pg -t SHOW_REPORT` para crear un informe que enumera todos los objetos de Oracle, el costo estimado de la migración (en días de desarrollador) y ciertos objetos de base de datos que pueden necesitar atención especial como parte de la conversión.

La mayoría de los clientes pasará una cantidad de tiempo considerable revisando el informe de evaluación para determinar el esfuerzo necesario para la conversión automática y manual.

Para configurar y ejecutar ora2pg para crear un informe de evaluación, consulte la sección **Migración previa: Evaluación** de la [guía de Oracle a Azure Database for PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf). Hay disponible [un informe de evaluación de ejemplo](http://ora2pg.darold.net/report.html) de ora2pg como referencia.

## <a name="export-the-oracle-schema"></a>Exportación del esquema de Oracle

Se recomienda usar ora2pg para convertir el esquema de Oracle y otros objetos de Oracle (tipos, procedimientos, funciones, etc.) en un esquema que sea compatible con Azure Database for PostgreSQL. ora2pg incluye muchas directivas que le ayudarán a predefinir ciertos tipos de datos. Por ejemplo, puede usar la directiva `DATA_TYPE` para reemplazar NUMBER(*,0) por bigint en lugar de NUMERIC(38).

Puede ejecutar ora2pg para exportar cada uno de los objetos de base de datos en archivos .sql. Después, puede revisar los archivos .sql antes de importarlos a Azure Database for PostgreSQL mediante psql o puede ejecutar el script .SQL en PgAdmin.

```
psql -f [FILENAME] -h [AzurePostgreConnection] -p 5432 -U [AzurePostgreUser] -d database 
```

Por ejemplo: 

```
psql -f %namespace%\schema\sequences\sequence.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database
```

Para configurar y ejecutar ora2pg para la conversión del esquema, vea la sección **Migración: Esquema y datos** de la [guía de Oracle a Azure Database for PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf).

## <a name="set-up-the-schema-in-azure-database-for-postgresql"></a>Configuración del esquema en Azure Database for PostgreSQL

De forma predeterminada, Oracle mantiene esquema.tabla.columna todo en mayúsculas, mientras que PostgreSQL mantiene esquema.tabla.columna en minúsculas. Para que Azure Database Migration Service inicie el movimiento de datos de Oracle a Azure Database for PostgreSQL, esquema.tabla.columna debe tener el mismo formato de mayúsculas y minúsculas que el origen de Oracle.

Por ejemplo, si el origen de Oracle tiene el esquema “HR”.”EMPLOYEES”.”EMPLOYEE_ID”, el esquema de PostgreSQL debe usar el mismo formato.

Para asegurarse de que el formato de mayúsculas y minúsculas de esquema.tabla.columna sea el mismo para Oracle y Azure Database for PostgreSQL, se recomienda seguir estos pasos.

> [!NOTE]
> Puede usar un enfoque diferente para derivar el esquema de mayúsculas y minúsculas. Estamos trabajando para mejorar y automatizar este paso.

1. Exporte los esquemas mediante ora2pg con minúsculas. En el script sql de creación de tablas, cree manualmente un esquema con mayúsculas "SCHEMA".
2. Importe el resto de los objetos de Oracle, como desencadenadores, secuencias, procedimientos, tipos y funciones, en Azure Database for PostgreSQL.
3. Para que TABLE y COLUMN estén en mayúsculas, ejecute el siguiente script:

   ```
   -- INPUT: schema name
   set schema.var = “HR”;

   -- Generate statements to rename tables and columns
   SELECT 1, 'SET search_path = "' ||current_setting('schema.var')||'";'
   UNION ALL 
   SELECT 2, 'alter table "'||c.relname||'" rename '||a.attname||' to "'||upper(a.attname)||'";'
   FROM pg_class c
   JOIN pg_attribute a ON a.attrelid = c.oid
   JOIN pg_type t ON a.atttypid = t.oid
   LEFT JOIN pg_catalog.pg_constraint r ON c.oid = r.conrelid
    AND r.conname = a.attname
   WHERE c.relnamespace = (select oid from pg_namespace where nspname=current_setting('schema.var')) AND a.attnum > 0 AND c.relkind ='r'
   UNION ALL
   SELECT 3, 'alter table '||c.relname||' rename to "'||upper(c.relname)||'";'
   FROM pg_catalog.pg_class c
    LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
   WHERE c.relkind ='r' AND n.nspname=current_setting('schema.var')
   ORDER BY 1;
   ```

* Quite la clave externa en la base de datos de destino para que se ejecute la carga completa. Consulte la sección **Migración del esquema de ejemplo** en [este artículo](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online) para ver un script que puede usar para quitar la clave externa.
* Use Azure Database Migration Service para ejecutar la sincronización y la carga completa.
* Cuando los datos de la base de datos de Azure Database for PostgreSQL de destino estén sincronizados con la fuente, realice la migración total de la base de datos en Azure Database Migration Service.
* Para que SCHEMA, TABLE y COLUMN estén en minúsculas (si el esquema de Azure Database for PostgreSQL debe estar así para la consulta de la aplicación), ejecute el siguiente script:

  ```
  -- INPUT: schema name
  set schema.var = hr;
  
  -- Generate statements to rename tables and columns
  SELECT 1, 'SET search_path = "' ||current_setting('schema.var')||'";'
  UNION ALL
  SELECT 2, 'alter table "'||c.relname||'" rename "'||a.attname||'" to '||lower(a.attname)||';'
  FROM pg_class c
  JOIN pg_attribute a ON a.attrelid = c.oid
  JOIN pg_type t ON a.atttypid = t.oid
  LEFT JOIN pg_catalog.pg_constraint r ON c.oid = r.conrelid
     AND r.conname = a.attname
  WHERE c.relnamespace = (select oid from pg_namespace where nspname=current_setting('schema.var')) AND a.attnum > 0 AND c.relkind ='r'
  UNION ALL
  SELECT 3, 'alter table "'||c.relname||'" rename to '||lower(c.relname)||';'
  FROM pg_catalog.pg_class c
     LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
  WHERE c.relkind ='r' AND n.nspname=current_setting('schema.var')
  ORDER BY 1;
  ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-oracle-azure-postgresql-online/portal-select-subscriptions.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-oracle-azure-postgresql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-oracle-azure-postgresql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Creación de una instancia de DMS

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-oracle-azure-postgresql-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-create2.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de Oracle de origen y a la instancia de Azure Database for PostgreSQL de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

5. Seleccione un plan de tarifa.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-settings5.png)

6. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

    ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

    ![Busque la instancia de Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-instance-search.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).
4. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Tipo de servidor de origen**, seleccione **Oracle** y, en el cuadro de texto **Tipo de servidor de destino**, elija **Azure Database for PostgreSQL**.
5. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

   ![Creación de un proyecto de Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-create-project5.png)

   > [!NOTE]
   > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

6. Seleccione **Guardar**, anote los requisitos para usar Azure Database Migration Service correctamente para realizar una migración en línea y, a continuación, seleccione **Crear y ejecutar actividad**.

## <a name="specify-source-details"></a>Especificación de los detalles de origen

* En la pantalla **Agregar detalles de origen**, especifique los detalles de conexión de la instancia de Oracle de origen.

  ![Pantalla Agregar detalles de origen](media/tutorial-oracle-azure-postgresql-online/dms-add-source-details.png)

## <a name="upload-oracle-oci-driver"></a>Carga del controlador OCI de Oracle

1. Seleccione **Guardar** y, a continuación, en la pantalla **Instalar controlador OCI**, inicie sesión en su cuenta de Oracle y descargue el controlador **instantclient-basiclite-windows.x64-12.2.0.1.0.zip** (37,128,586 Byte(s)) (SHA1 Checksum: 865082268) [desde aquí](https://www.oracle.com/technetwork/topics/winx64soft-089540.html#ic_winx64_inst).
2. Descargue el controlador en una carpeta compartida.

   Asegúrese de que la carpeta se comparte con el nombre de usuario que especificó con el acceso mínimo de solo lectura. Azure Database Migration Service accede y lee el recurso compartido para cargar el controlador OCI en Azure mediante la suplantación del nombre de usuario que especifique.

   El nombre de usuario que especifique debe ser una cuenta de usuario de Windows.

   ![Instalación del controlador OCI](media/tutorial-oracle-azure-postgresql-online/dms-oci-driver-install.png)

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y, en la pantalla **Detalles del destino**, especifique los detalles de conexión del servidor de Azure Database for PostgreSQL de destino, que es la instancia de Azure Database for PostgreSQL que se aprovisionó previamente y en la que se implementó el esquema **HR**.

    ![Pantalla de detalles de destino](media/tutorial-oracle-azure-postgresql-online/dms-add-target-details1.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-oracle-azure-postgresql-online/dms-map-target-details.png)

3. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-oracle-azure-postgresql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

  Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **En ejecución**.

     ![Estado de actividad: en ejecución](media/tutorial-oracle-azure-postgresql-online/dms-activity-running.png)

2. En **Nombre de la base de datos**, seleccione una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    La carga completa de los datos muestra el estado de migración de la carga inicial mientras que la sincronización de datos incrementales muestra el estado de la captura de datos modificados (CDC).

     ![Estado de actividad: carga completa efectuada](media/tutorial-oracle-azure-postgresql-online/dms-activity-full-load-completed.png)

     ![Estado de actividad: sincronización de datos incrementales](media/tutorial-oracle-azure-postgresql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez completada la carga completa inicial, las bases de datos se marcan como **A punto para la migración total**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.

   ![Iniciar transición](media/tutorial-oracle-azure-postgresql-online/dms-start-cutover.png)

3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva instancia de Azure Database for PostgreSQL de destino.

 > [!NOTE]
 > Puesto que, de forma predeterminada, PostgreSQL tiene esquema.tabla.columna en minúsculas, puede cambiar de mayúsculas a minúsculas con el script indicado en la sección **Configuración del esquema en Azure Database for PostgreSQL**, anteriormente en este artículo.

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure Database for PostgreSQL, consulte el artículo [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problemas conocidos y soluciones alternativas con migraciones en línea de Azure Database for PostgreSQL).
* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para más información sobre Azure Database for PostgreSQL, consulte el artículo [¿Qué es Azure Database for PostgreSQL?](https://docs.microsoft.com/azure/postgresql/overview)
