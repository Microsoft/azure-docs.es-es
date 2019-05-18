---
title: 'Tutorial: Uso de Azure Database Migration Service para realizar una migración en línea de RDS SQL Server a Azure SQL Database o a una instancia administrada de Azure SQL Database | Microsoft Docs'
description: Aprenda a realizar una migración en línea desde RDS SQL Server a Azure SQL Database o a una instancia administrada de Azure SQL Database mediante Azure Database Migration Service.
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
ms.openlocfilehash: 982ead69ba3d206e1aa2538597927dcbeaab70e9
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65416098"
---
# <a name="tutorial-migrate-rds-sql-server-to-azure-sql-database-or-an-azure-sql-database-managed-instance-online-using-dms"></a>Tutorial: Migración de RDS SQL Server a Azure SQL Database o a una instancia administrada de Azure SQL Database en línea mediante DMS
Azure Database Migration Service se puede usar para migrar las bases de datos de una instancia de RDS SQL Server local a [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/) o a una [instancia administrada de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-index) con un tiempo de inactividad mínimo. En este tutorial migrará la base de datos **Adventureworks2012** restaurada en una instancia de RDS SQL Server de SQL 2012 (o posterior) a Azure SQL Database o a una instancia administrada de Azure SQL Database mediante Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear una base de datos de Azure SQL o una instancia administrada de Azure SQL Database 
> * Migrar el esquema de ejemplo mediante Data Migration Assistant
> * Crear una instancia de Azure Database Migration Service.
> * Crear un proyecto de migración mediante Azure Database Migration Service.
> * Ejecutar la migración.
> * Supervisar la migración
> * Descargar un informe de migración.

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. Para más información, consulte la página de [precios](https://azure.microsoft.com/pricing/details/database-migration/) de Azure Database Migration Service.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe una migración en línea desde RDS SQL Server a Azure SQL Database o a una instancia administrada de Azure SQL Database.

## <a name="prerequisites"></a>Requisitos previos
Para completar este tutorial, necesita:

* Crear una [base datos de RDS SQL Server](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.SQLServer.html).
* Crear una base de datos de Azure SQL, para lo que debe seguir la información del artículo acerca de cómo [crear una base de datos de Azure SQL en Azure Portal](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal).

    > [!NOTE]
    > Si va a migrar a una instancia administrada de Azure SQL Database, siga los detalles que se indican en el artículo [Creación de una Instancia administrada de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-get-started) y cree una base de datos vacía llamada **AdventureWorks2012**. 
 
* Descargar e instalar [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) versión 3.3 o posterior.
* Crear una red virtual de Azure para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager. Si va a realizar la migración a una instancia administrada de Azure SQL Database, asegúrese de que la instancia de DMS se crea en la misma red virtual usada para la instancia administrada de Azure SQL Database, pero en otra subred.  Como alternativa, si usa otra red virtual para DMS, deberá crear un emparejamiento entre las dos redes virtuales. Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de redes con Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:
    >
    > * Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > * Punto de conexión de Storage
    > * Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque la instancia de Azure Database Migration Service carece de conectividad a internet. 

* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445, 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg).
* Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda a la instancia de SQL Server de origen que, de manera predeterminada, es el puerto TCP 1433.
* Crear una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) de nivel de servidor para que el servidor de Azure SQL Database permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcionar el intervalo de subred de la red virtual usada para Azure Database Migration Service.
* Asegurarse de que las credenciales usadas para la conexión a la instancia de RDS SQL Server de origen están asociadas a una cuenta que sea miembro del rol de servidor "Processadmin" y miembro de los roles de base de datos de "db_owner" en todas las bases de datos que desea migrar.
* Asegurarse de que las credenciales usadas para conectarse a la base de datos de Azure SQL de destino tengan el permiso CONTROL DATABASE en las bases de datos de Azure SQL de destino y sean miembro del rol sysadmin si se van a migrar a una instancia administrada de Azure SQL Database.
* La versión del servidor RDS SQL Server de origen debe ser SQL Server 2012 u otra posterior. Para determinar la versión que está ejecutando la instancia de SQL Server, consulte el artículo [Cómo determinar la versión, la edición y el nivel de actualización de SQL Server y sus componentes](https://support.microsoft.com/help/321185/how-to-determine-the-version-edition-and-update-level-of-sql-server-an).
* Habilitar la captura de datos modificados (CDC) en la base de datos de RDS SQL Server y todas las tablas de usuario seleccionadas para la migración.
    > [!NOTE]
    > Puede usar el siguiente script para habilitar CDC en una base de datos de RDS SQL Server.
    ```
    exec msdb.dbo.rds_cdc_enable_db 'AdventureWorks2012'
    ```
    > Puede usar el siguiente script para habilitar CDC en todas las tablas.
    ```
    use <Database name>
    go
    exec sys.sp_cdc_enable_table 
    @source_schema = N'Schema name', 
    @source_name = N'table name', 
    @role_name = NULL, 
    @supports_net_changes = 1 --for PK table 1, non PK tables 0
    GO
    ```
* Deshabilite los desencadenadores de la base de datos en la base de datos de Azure SQL de destino.
    > [!NOTE]
    > Puede encontrar los desencadenadores de la base de datos en la base de datos de Azure SQL de destino mediante la consulta siguiente:
    ```
    Use <Database name>
    select * from sys.triggers
    DISABLE TRIGGER (Transact-SQL)
    ```
    Para más información, consulte el artículo [DISABLE TRIGGER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017).

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo
Usar DMA para migrar el esquema a Azure SQL Database.

> [!NOTE]
> Antes de crear un proyecto de migración en Data Migration Assistant, asegúrese de que ya ha proporcionado una base de datos de Azure SQL Database, tal y como se mencionó en los requisitos previos. Para los fines de este tutorial, se da por hecho que el nombre de Azure SQL Database es **AdventureWorks2012**, pero puede asignarle otro si lo desea.

Para migrar el esquema de **AdventureWorks2012** a Azure SQL Database, siga estos pasos:

1. En Data Migration Assistant, seleccione el icono de Nuevo (+) y, en **Tipo de proyecto**, seleccione el tipo de proyecto **Migración**.
2. Especifique un nombre de proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **SQL Server** y, luego, en el cuadro de texto **Target server type** (Tipo de servidor de destino), elija **Azure SQL Database**.
3. En **Migration Scope** (Ámbito de la migración), seleccione **Solo esquema**.

    Después de realizar los pasos anteriores, la interfaz de Data Migration Assistant debe aparecer como se muestra en el siguiente gráfico:

    ![Creación de proyecto en Data Migration Assistant](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-create-project.png)

4. Seleccione **Crear** para crear el proyecto.
5. En Data Migration Assistant, especifique los detalles de conexión de origen de SQL Server, seleccione **Conectar** y, luego, elija la base de datos **AdventureWorks2012**.

    ![Detalles de conexión de origen en Data Migration Assistant](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-source-connect.png)

6. Seleccione **Siguiente** en **Connect to target server** (Conectar a servidor de destino), especifique los detalles de conexión de destino de la base de datos de Azure SQL Database, seleccione **Conectar** y elija la base de datos **AdventureWorksAzure** que había aprovisionado previamente en Azure SQL Database.

    ![Detalles de conexión de destino en Data Migration Assistant](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-target-connect.png)

7. Seleccione **Siguiente** para avanzar a la pantalla **Seleccionar objetos**, en la que puede especificar los objetos de esquema de la base de datos **AdventureWorks2012** que deben implementarse en Azure SQL Database.

    De forma predeterminada, se seleccionan todos los objetos.

    ![Generación de scripts SQL](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-assessment-source.png)

8. Seleccione **Generar script SQL** para crear los scripts SQL y, luego, revíselos para ver si hay errores.

    ![Script de esquema](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-schema-script.png)

9. Seleccione **Deploy schema** (Implementar esquema) para implementar el esquema en Azure SQL Database. Luego, una vez implementado, compruebe el servidor de destino por si hay cualquier anomalía.

    ![Implementación del esquema](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-sql-server-to-azure-sql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Creación de una instancia

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service. 

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de SQL Server de origen y a la instancia de Azure SQL Database de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

6. Seleccione un plan de tarifa. Para la migración en línea, asegúrese de seleccionar el plan de tarifa prémium.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

     ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-settings3.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

      ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

     ![Busque la instancia de Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-instance-search.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).
4. En la pantalla **Nuevo proyecto de migración**, especifique un nombre para el proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **AWS RDS for SQL Server** y, en el cuadro de texto **Target server type** (Tipo de servidor de destino), elija **Azure SQL Database**.

    > [!NOTE]
    > Para el tipo de servidor de destino, seleccione **Azure SQL Database** para migrar tanto a una base de datos singleton de Azure SQL Database como a una instancia administrada de Azure SQL Database.

5. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

    > [!IMPORTANT]
    > Asegúrese de seleccionar **Migración de datos en línea**; las migraciones sin conexión no se admiten en este escenario.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create-project4.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

6. Seleccione **Guardar**.

7. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

    ![Creación y ejecución de una actividad de Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create-and-run-activity1.png)

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Detalles del origen de la migración**, especifique los detalles de conexión de la instancia de SQL Server de origen.

    Asegúrese de usar un nombre de dominio completo (FQDN) para el nombre de la instancia de SQL Server de origen.

2. Si no ha instalado ningún certificado de confianza en el servidor de origen, seleccione la casilla de verificación **Certificado de servidor de confianza**.

    Si no hay ningún certificado de confianza instalado, SQL Server genera un certificado autofirmado cuando se inicia la instancia. Este certificado se usa para cifrar las credenciales de las conexiones del cliente.

    > [!CAUTION]
    > Las conexiones SSL cifradas mediante un certificado autofirmado no proporcionan una gran seguridad. Son susceptibles de sufrir ataques de tipo "Man in the middle". No se debe confiar en SSL mediante certificados autofirmados en un entorno de producción o en servidores que están conectados a Internet.

   ![Detalles del origen](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-source-details3.png)

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y después, en la pantalla **Detalles del destino de la migración**, especifique los detalles de conexión del servidor de Azure SQL Database de destino, que es la base de datos de Azure SQL que se aprovisionó previamente y en la que se implementó el esquema de **AdventureWorks2012** mediante Data Migration Assistant.

    ![Selección del destino](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-select-target3.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-map-targets-activity4.png)

3. Seleccione **Guardar** y, en la pantalla **Seleccionar tablas**, expanda la lista de tablas y revise la lista de campos afectados.

    Azure Database Migration Service selecciona automáticamente todas las tablas de origen vacías que existen en la base de datos de Azure SQL de destino. Si quiere volver a migrar tablas que ya contienen datos, deberá seleccionar de forma explícita las tablas en esta pantalla.

    ![Selección de tablas](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-configure-setting-activity4.png)

4. Seleccione **Guardar** después de establecer la siguiente **Configuración avanzada de la migración en línea**.

    | Configuración | DESCRIPCIÓN |
    | ------------- | ------------- |
    | **Número máximo de tablas que se va a cargar en paralelo** | Especifica el número de tablas que DMS se ejecuta en paralelo durante la migración. El valor predeterminado es 5, pero se puede establecer en un valor óptimo para satisfacer necesidades de migración específicas en función de las migraciones de prueba de concepto. |
    | **Cuando se trunca la tabla de origen** | Especifica si DMS trunca la tabla de destino durante la migración. Este valor puede resultar útil si una o más tablas se truncan como parte del proceso de migración. |
    | **Configurar ajustes de los datos de objetos grandes (LOB)** | Especifica si DMS migra los datos de LOB ilimitados o limita los datos de LOB migrados a un tamaño específico.  Cuando hay un límite en los datos de LOB migrados, se truncan los datos de LOB más allá de ese límite. Para las migraciones de producción, se recomienda seleccionar **Permitir tamaño ilimitado de LOB** para evitar la pérdida de datos. Al especificar esta opción de permitir un tamaño ilimitado de LOB, seleccione la casilla **Migrate LOB data in a single block when the LOB size is less than (KB) specified** (Migrar datos de LOB en un solo bloque cuando el tamaño de LOB es menor que los [KB] especificados) para mejorar el rendimiento. |

    ![Configuración avanzada de la migración en línea](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-advanced-online-migration-settings.png)

5. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

* Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

    ![Estado de la actividad: Inicializando](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-status2.png)

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **En ejecución**.

2. Haga clic en una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    ![Estado de la actividad: En curso](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-in-progress.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez completada la carga completa inicial, las bases de datos se marcan como **A punto para la migración total**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

    ![Iniciar transición](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-start-cutover.png)

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.
3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure SQL de destino.

    ![Estado de actividad Completado](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-completed.png)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure SQL Database, consulte el artículo [Problemas conocidos y soluciones para las migraciones en línea a Azure SQL Database](known-issues-azure-sql-online.md).
* Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
* Para más información sobre Azure SQL Database, consulte el artículo [¿Qué es el servicio Azure SQL Database?](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
* Para información sobre las instancias administradas de Azure SQL Database, consulte la página [Instancia administrada de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-index).
