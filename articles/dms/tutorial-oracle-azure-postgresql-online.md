---
title: 'Tutorial: Migración de Oracle en línea a Azure Database for PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Aprenda a migrar bases de datos de Oracle locales o en máquinas virtuales a Azure Database for PostgreSQL mediante Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: article
ms.date: 01/24/2020
ms.openlocfilehash: 37bf610d1ecc6a05e8dd142ff3d72dd9ed72e51f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087771"
---
# <a name="tutorial-migrate-oracle-to-azure-database-for-postgresql-online-using-dms-preview"></a>Tutorial: Migración de Oracle a Azure Database for PostgreSQL en línea mediante DMS (versión preliminar)

Puede usar Azure Database Migration Service para migrar las bases de datos de Oracle locales o en máquinas virtuales a [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/) con un tiempo de inactividad mínimo. En otras palabras, la migración se puede completar con un tiempo de inactividad mínimo para la aplicación. En este tutorial, va a migrar la base de datos de ejemplo **HR** de una instancia local o en máquina virtual de Oracle 11g a Azure Database for PostgreSQL mediante la actividad de migración en línea de Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
>
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

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

* Descargue e instale [Oracle 11g Release 2 (Standard Editionde Standard Edition One o Enterprise Edition)](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html).
* Descargue la base de datos de ejemplo **HR**[aquí](https://docs.oracle.com/database/121/COMSC/installation.htm#COMSC00002).
* Descargue e [instale ora2pg en Windows o Linux](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf).
* [Creación de una instancia en Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal).
* Conéctese a la instancia y cree una base de datos con las instrucciones de este [documento](https://docs.microsoft.com/azure/postgresql/tutorial-design-database-using-azure-portal).
* Cree una instancia de Microsoft Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](https://docs.microsoft.com/azure/virtual-network/)y, especialmente, los artículos de inicio rápido con detalles paso a paso.

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
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda al servidor Oracle de origen que, de manera predeterminada, es el puerto TCP 1521.
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
* Cree una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) en el nivel de servidor para que Azure Database for PostgreSQL permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcione el rango de subred de la red virtual que se usa para Azure Database Migration Service.
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

## <a name="assess-the-effort-for-an-oracle-to-azure-database-for-postgresql-migration"></a>Evaluación del esfuerzo de la migración de Oracle a Azure Database for PostgreSQL

Se recomienda usar ora2pg para evaluar el esfuerzo necesario para migrar desde Oracle a Azure Database for PostgreSQL. Use la directiva `ora2pg -t SHOW_REPORT` para crear un informe que enumera todos los objetos de Oracle, el costo estimado de la migración (en días de desarrollador) y ciertos objetos de base de datos que pueden necesitar atención especial como parte de la conversión.

La mayoría de los clientes pasará una cantidad de tiempo considerable revisando el informe de evaluación para determinar el esfuerzo necesario para la conversión automática y manual.

Para configurar y ejecutar ora2pg para crear un informe de evaluación, consulte la sección **Migración previa: Evaluación** de la [guía de Oracle a Azure Database for PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf). Hay disponible [un informe de evaluación de ejemplo](https://ora2pg.darold.net/report.html) de ora2pg como referencia.

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

Puede optar por convertir esquemas de tabla, procedimientos almacenados, paquetes y otros objetos de base de datos de Oracle para que sean compatibles con Postgres mediante ora2pg antes de iniciar una canalización de migración en Azure Database Migration Service. Consulte los vínculos siguientes para ver cómo trabajar con ora2pg:

* [Instalación de ora2pg en Windows](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf)
* [Oracle to Azure PostgreSQL Migration Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf) (Instrucciones de migración de Oracle a Azure PostgreSQL)

Azure Database Migration Service también puede crear el esquema de tabla en PostgreSQL. El servicio accede al esquema de tabla del origen de Oracle conectado y crea un esquema de tabla compatible en Azure Database for PostgreSQL. Asegúrese de validar y comprobar el formato del esquema en Azure Database for PostgreSQL cuando Azure Database Migration Service haya terminado de crear el esquema y mover los datos.

> [!IMPORTANT]
> Azure Database Migration Service solo crea el esquema de tabla; no se crean otros objetos de base de datos, como procedimientos almacenados, paquetes, índices, etc.

Asegúrese también de anular la clave externa de la base de datos de destino para que se ejecute la carga completa. Consulte la sección **Migración del esquema de ejemplo** en [este artículo](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online) para ver un script que puede usar para quitar la clave externa. Use Azure Database Migration Service para ejecutar la sincronización y la carga completa.

### <a name="when-the-postgresql-table-schema-already-exists"></a>Cuando el esquema de tabla ya existe en PostgreSQL

Si crea un esquema de PostgreSQL mediante herramientas como ora2pg antes de iniciar el movimiento de datos con Azure Database Migration Service, asigne las tablas de origen a las de destino en Azure Database Migration Service.

1. Al crear un nuevo proyecto de migración de Oracle a Azure Database for PostgreSQL, se le pedirá que seleccione la base de datos de destino y el esquema de destino en el paso Seleccionar esquemas. Rellene la base de datos de destino y el esquema de destino.

   ![Mostrar suscripciones en el portal](media/tutorial-oracle-azure-postgresql-online/dms-map-to-target-databases.png)

2. En la pantalla **Configuración de migración** se presenta una lista de las tablas del origen de Oracle. Azure Database Migration Service intenta hacer coincidir las tablas de origen y de destino en función de su nombre. Si existen varias tablas de destino coincidentes con distintas mayúsculas y minúsculas, puede seleccionar la tabla de destino para la asignación.

    ![Mostrar suscripciones en el portal](media/tutorial-oracle-azure-postgresql-online/dms-migration-settings.png)

> [!NOTE]
> Si necesita asignar nombres de tabla de origen a tablas con nombres distintos, envíe un correo electrónico a [dmsfeedback@microsoft.com](mailto:dmsfeedbac@microsoft.com) y le proporcionaremos un script para automatizar el proceso.

### <a name="when-the-postgresql-table-schema-doesnt-exist"></a>Cuando el esquema de tabla no existe aún en PostgreSQL

Si la base de datos de PostgreSQL de destino no contiene ninguna información de esquema de tabla, Azure Database Migration Service convierte el esquema de origen y lo vuelve a crear en la base de datos de destino. Recuerde que Azure Database Migration Service solo crea el esquema de tabla; no otros objetos de base de datos, como procedimientos almacenados, paquetes o índices.
Para que Azure Database Migration Service cree el esquema automáticamente, asegúrese de que el entorno de destino incluye un esquema sin tablas. Si Azure Database Migration Service detecta alguna tabla, supondrá que el esquema se creó con una herramienta externa, como ora2pg.

> [!IMPORTANT]
> Azure Database Migration Service necesita que todas las tablas se creen de la misma manera, con Azure Database Migration Service o una herramienta como ora2pg, pero no con ambos.

Primeros pasos:

1. Cree un esquema en la base de datos de destino en función de los requisitos de la aplicación. De forma predeterminada, los nombres de las columnas y el esquema de tabla de PostgreSQL están en minúsculas. En cambio, de forma predeterminada, las columnas y el esquema de tabla de Oracle están en mayúsculas.
2. En el paso Seleccionar esquemas, especifique la base de datos y el esquema de destino.
3. Según el esquema que cree en Azure Database for PostgreSQL, Azure Database Migration Service utilizará las siguientes reglas de transformación:

    Si el nombre del esquema del origen de Oracle coincide con el de Azure Database for PostgreSQL, Azure Database Migration Service *crea el esquema de tabla con el mismo formato de mayúsculas y minúsculas que el destino*.

    Por ejemplo:

    | Origen: esquema de Oracle | Destino: Database.Schema de PostgreSQL | schema.table.column creado por DMS |
    | ------------- | ------------- | ------------- |
    | HR | targetHR.public | public.countries.country_id |
    | HR | targetHR.trgthr | trgthr.countries.country_id |
    | HR | targetHR.TARGETHR | "TARGETHR"."COUNTRIES"."COUNTRY_ID" |
    | HR | targetHR.HR | "HR"."COUNTRIES"."COUNTRY_ID" |
    | HR | targetHR.Hr | *No se puede asignar una mezcla de mayúsculas y minúsculas |

    *Para crear nombres de tablas y esquemas con mezcla de mayúsculas y minúsculas en PostgreSQL de destino, póngase en contacto con [dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com). Podemos proporcionarle un script para configurar un esquema de tabla de con mezcla de mayúsculas y minúsculas en la base de datos de PostgreSQL de destino.

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

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual con Azure Portal](https://aka.ms/DMSVnet).

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
* Para más información acerca de Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para más información sobre Azure Database for PostgreSQL, consulte el artículo [¿Qué es Azure Database for PostgreSQL?](https://docs.microsoft.com/azure/postgresql/overview)
