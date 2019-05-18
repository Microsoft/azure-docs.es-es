---
title: 'Tutorial: Uso de Azure Database Migration Service para migrar de SQL Server en línea a una base de datos única o agrupada en Azure SQL Database | Microsoft Docs'
description: Aprenda a migrar de SQL Server local en línea a una base de datos única o agrupada en Azure SQL Database mediante Azure Database Migration Service.
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
ms.openlocfilehash: 266e4a16a69d7200fbe8b58bc20339b6979db877
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415919"
---
# <a name="tutorial-migrate-sql-server-to-a-single-database-or-pooled-database-in-azure-sql-database-online-using-dms"></a>Tutorial: Migración de SQL Server a una base de datos única o agrupada en Azure SQL Database en línea mediante DMS

Puede usar Azure Database Migration Service para migrar las bases de datos de una instancia de SQL Server local a [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/) con un tiempo de inactividad mínimo. En este tutorial migrará la base de datos **Adventureworks2012** restaurada en una instancia local de SQL Server 2016 (o posterior) a una base de datos única o agrupada de Azure SQL Database mediante Azure Database Migration Service.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Evaluar la base de datos local mediante Data Migration Assistant
> - Migrar el esquema de ejemplo mediante Data Migration Assistant
> - Crear una instancia de Azure Database Migration Service.
> - Crear un proyecto de migración mediante Azure Database Migration Service.
> - Ejecutar la migración.
> - Supervisar la migración
> - Descargar un informe de migración.

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. Para más información, consulte la página de [precios](https://azure.microsoft.com/pricing/details/database-migration/) de Azure Database Migration Service.

> [!IMPORTANT]
> Para disfrutar de una experiencia de migración óptima, Microsoft recomienda crear una instancia de Azure Database Migration Service en la misma región de Azure que la base de datos de destino. Si los datos se transfieren entre diferentes regiones o ubicaciones geográficas, el proceso de migración puede verse afectado y pueden producirse errores.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

En este artículo se describe una migración en línea desde SQL Server a una base de datos única o agrupada de Azure SQL Database. Para migraciones sin conexión, consulte [Migración de SQL Server a Azure SQL Database sin conexión mediante DMS](tutorial-sql-server-to-azure-sql.md).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

- Descargar e instalar [SQL Server 2012 o una versión posterior](https://www.microsoft.com/sql-server/sql-server-downloads) (cualquier edición).
- Habilitar el protocolo TCP/IP, que se deshabilita de forma predeterminada durante la instalación de SQL Server Express, siguiendo las instrucciones del artículo [Habilitar o deshabilitar un protocolo de red de servidor](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
- Crear una base de datos única o agrupada de Azure SQL Database, para lo que debe seguir la información del artículo acerca de cómo [crear una base de datos única de Azure SQL Database en Azure Portal](https://docs.microsoft.com/azure/sql-database/sql-database-single-database-get-started).

    > [!NOTE]
    > Si usa SQL Server Integration Services (SSIS) y desea migrar la base de datos de catálogo para los proyectos y paquetes SSIS (SSISDB) de SQL Server a Azure SQL Database, la SSISDB de destino se creará y administrará automáticamente en su nombre cuando aprovisione SSIS en Azure Data Factory (ADF). Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages).

- Descargar e instalar [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) versión 3.3 o posterior.
- Crear una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways). Para más información acerca de cómo crear una red virtual, consulte la [documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/) y, concretamente, los artículos de inicio rápido que proporcionan instrucciones paso a paso.

    > [!NOTE]
    > Durante la configuración de la red virtual, si usa ExpressRoute con emparejamiento de redes con Microsoft, agregue los siguientes [puntos de conexión](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) de servicio a la subred en la que se aprovisionará el servicio:
    > - Punto de conexión de base de datos de destino (por ejemplo, punto de conexión de SQL, punto de conexión de Cosmos DB, etc.)
    > - Punto de conexión de Storage
    > - Punto de conexión de Service Bus
    >
    > Esta configuración es necesaria porque la instancia de Azure Database Migration Service carece de conectividad a internet.

- Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean los siguientes puertos de comunicación de entrada a Azure Database Migration Service: 443, 53, 9354, 445, 12000. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales de Azure, consulte el artículo [Planeamiento de redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg).
- Configurar su [Firewall de Windows para acceder al motor de base de datos](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
- Abra el Firewall de Windows para permitir que Azure Database Migration Service acceda a la instancia de SQL Server de origen que, de manera predeterminada, es el puerto TCP 1433.
- Si se ejecutan varias instancias con nombre de SQL Server con puertos dinámicos, puede ser conveniente habilitar el servicio SQL Browser y permitir el acceso al puerto UDP 1434 mediante los firewalls para que Azure Database Migration Service pueda conectarse a una instancia con nombre en el servidor de origen.
- Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.
- Crear una [regla de firewall](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) de nivel de servidor para que el servidor de Azure SQL Database permita a Azure Database Migration Service acceder a las bases de datos de destino. Proporcionar el rango de subred de la red virtual usada para Azure Database Migration Service.
- Asegurarse de que las credenciales usadas para conectarse a la instancia de SQL Server de origen tenga permisos [CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql).
- Asegurarse de que las credenciales usadas para conectarse a la base de datos de Azure SQL de destino tengan permisos CONTROL DATABASE en las bases de datos de Azure SQL de destino.
- La versión del servidor SQL Server de origen debe ser SQL Server 2005 u otra posterior. Para determinar la versión que está ejecutando la instancia de SQL Server, consulte el artículo [Cómo determinar la versión, la edición y el nivel de actualización de SQL Server y sus componentes](https://support.microsoft.com/help/321185/how-to-determine-the-version-edition-and-update-level-of-sql-server-an).
- Las bases de datos deben estar en el modelo de recuperación completa o en el optimizado para cargas masivas de registros. Para determinar el modelo de recuperación configurado para la instancia de SQL Server, consulte el artículo [Ver o cambiar el modelo de recuperación de una base de datos (SQL Server)](https://docs.microsoft.com/sql/relational-databases/backup-restore/view-or-change-the-recovery-model-of-a-database-sql-server?view=sql-server-2017).
- Asegúrese de realizar copias de seguridad completas de las bases de datos. Para crear una copia de seguridad de base de datos completa, consulte el artículo [Cómo crear una copia de seguridad de la base de datos completa (Transact-SQL)](https://docs.microsoft.com/previous-versions/sql/sql-server-2008-r2/ms191304(v=sql.105)).
- Si alguna de las tablas no tiene una clave principal, habilite la captura de datos modificados (CDC) en la base de datos y en las tablas concretas.
    > [!NOTE]
    > Puede usar el script siguiente para encontrar todas las tablas que no tengan clave principal.

    ```sql
    USE <DBName>;
    go
    SELECT is_tracked_by_cdc, name AS TableName
    FROM sys.tables WHERE type = 'U' and is_ms_shipped = 0 AND
    OBJECTPROPERTY(OBJECT_ID, 'TableHasPrimaryKey') = 0;
     ```

    >Si los resultados muestran una o varias tablas con "is_tracked_by_cdc" como "0", habilite la captura de datos modificados para la base de datos y para las tablas específicas mediante el proceso descrito en el artículo [Habilitar y deshabilitar la captura de datos modificados (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server?view=sql-server-2017).

- Configure el rol de distribuidor para el servidor SQL Server de origen.

    >[!NOTE]
    > Puede determinar si los componentes de replicación están instalados mediante la consulta siguiente.

    ```sql
    USE master;
    DECLARE @installed int;
    EXEC @installed = sys.sp_MS_replication_installed;
    SELECT @installed as installed;
    ```

    Si el resultado devuelve un mensaje de error en el que se sugiere instalar los componentes de replicación de SQL Server, hágalo siguiendo el proceso que se indica en el artículo [Instalar la replicación de SQL Server](https://docs.microsoft.com/sql/database-engine/install-windows/install-sql-server-replication?view=sql-server-2017).

    Si la replicación ya está instalada, compruebe si el rol de distribución está configurado en el servidor SQL Server de origen mediante el comando T-SQL siguiente.

    ```sql
    EXEC sp_get_distributor;
    ```

    Si no está configurada la distribución y el servidor de distribución muestra NULL para la salida del comando anterior, configure la distribución mediante la guía que se proporciona en el artículo [Configurar la publicación y la distribución](https://docs.microsoft.com/sql/relational-databases/replication/configure-publishing-and-distribution?view=sql-server-2017).

- Deshabilite los desencadenadores de la base de datos en la base de datos de Azure SQL de destino.
    >[!NOTE]
    > Puede encontrar los desencadenadores de la base de datos en la base de datos de Azure SQL de destino mediante la consulta siguiente:

    ```sql
    Use <Database name>
    select * from sys.triggers
    DISABLE TRIGGER (Transact-SQL)
    ```

    Para más información, consulte el artículo [DISABLE TRIGGER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017). 

## <a name="assess-your-on-premises-database"></a>Evaluación de una base de datos local

Antes de poder migrar datos de una instancia de SQL Server local a una base de datos única o agrupada de Azure SQL Database, debe evaluar la base de datos de SQL Server por si hay cualquier error de bloqueo que impida que se realice la migración. Con Data Migration Assistant v3.3 o una versión posterior, siga los pasos descritos en el artículo sobre cómo [llevar a cabo una evaluación de migración de SQL Server](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem) para completar la evaluación de la base de datos local.

Para evaluar una base de datos de local, realice los pasos siguientes:

1. En Data Migration Assistant, seleccione el icono de Nuevo (+) y, luego, seleccione el tipo de proyecto **Evaluación**.
2. Especifique un nombre de proyecto. Después, en el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **SQL Server**; en el cuadro de texto **Target server type** (Tipo de servidor de destino), seleccione **Azure SQL Database** y, después, seleccione **Create** (Crear) para crear el proyecto.

    Cuando esté evaluando la base de datos de SQL Server de origen que se migrará a una base de datos única o agrupada de Azure SQL Database, puede elegir uno o ambos de los siguientes tipos de informes de evaluación:

   - Check database compatibility (Comprobar compatibilidad de bases de datos)
   - Check feature parity (Comprobar paridad de características)

     De forma predeterminada, se seleccionan los dos tipos de informes.

3. En DMA, en la pantalla **Opciones**, seleccione **Siguiente**.
4. En la pantalla **Seleccionar orígenes**, cuadro de diálogo **Conectar a un servidor**, proporcione los detalles de conexión en su instancia de SQL Server y, luego, seleccione **Conectar**.
5. En el cuadro de diálogo **Agregar orígenes**, seleccione **AdventureWorks2012**, seleccione **Agregar** y, por último, seleccione **Iniciar evaluación**.

    > [!NOTE]
    > Si usa SSIS, DMA no admite actualmente la valoración de la SSISDB de origen. De todas formas, los proyectos y paquetes SSIS se evaluarán/validarán a medida que se vuelven a implementar en la SSISDB de destino hospedada por Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages).

    Una vez completada la evaluación, los resultados se muestran tal y como se muestra en el siguiente gráfico:

    ![Evaluación de la migración de datos](media/tutorial-sql-server-to-azure-sql-online/dma-assessments.png)

    Para bases de datos únicas o agrupadas de Azure SQL Database, las evaluaciones identifican problemas de paridad de características y problemas de bloqueo de migración para la implementación en una base de datos única o agrupada.

    - La categoría de **paridad de características de SQL Server** proporciona un conjunto completo de recomendaciones, alternativas disponibles en Azure y pasos de mitigación para ayudarlo a planear el trabajo en los proyectos de migración.
    - La categoría de **problemas de compatibilidad** proporciona características no compatibles o parcialmente compatibles que reflejan los problemas de compatibilidad que podrían bloquear la migración de bases de datos de SQL Server locales a bases de datos de Azure SQL Database. También se proporcionan recomendaciones para ayudarlo a resolver esos problemas.

6. Revise los resultados de evaluación correspondientes a los problemas de bloqueo de migración y los de paridad de características seleccionando las opciones correspondientes.

## <a name="migrate-the-sample-schema"></a>Migración del esquema de ejemplo

Cuando se haya familiarizado con la evaluación y piense que la base de datos seleccionada es una candidata viable para la migración a una base de datos única o agrupada de Azure SQL Database, use Data Migration Assistant para migrar el esquema a Azure SQL Database.

> [!NOTE]
> Antes de crear un proyecto de migración en Data Migration Assistant, asegúrese de que ya ha proporcionado una base de datos de Azure SQL Database, tal y como se mencionó en los requisitos previos. Para los fines de este tutorial, se da por hecho que el nombre de Azure SQL Database es **AdventureWorksAzure**, pero puede asignarle otro si lo desea.

> [!IMPORTANT]
> Si usa SSIS, DMA no admite actualmente la migración de la SSISDB de origen, pero puede volver a implementar sus proyectos y paquetes SSIS en la SSISDB de destino hospedada por Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages).

Para migrar el esquema de **AdventureWorks2012** a una base de datos única o agrupada de Azure SQL Database, siga estos pasos:

1. En Data Migration Assistant, seleccione el icono de Nuevo (+) y, en **Tipo de proyecto**, seleccione el tipo de proyecto **Migración**.
2. Especifique un nombre de proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **SQL Server** y, luego, en el cuadro de texto **Target server type** (Tipo de servidor de destino), elija **Azure SQL Database**.
3. En **Migration Scope** (Ámbito de la migración), seleccione **Solo esquema**.

    Después de realizar los pasos anteriores, la interfaz de Data Migration Assistant debe aparecer como se muestra en el siguiente gráfico:

    ![Creación de proyecto en Data Migration Assistant](media/tutorial-sql-server-to-azure-sql-online/dma-create-project.png)

4. Seleccione **Crear** para crear el proyecto.
5. En Data Migration Assistant, especifique los detalles de conexión de origen de SQL Server, seleccione **Conectar** y, luego, elija la base de datos **AdventureWorks2012**.

    ![Detalles de conexión de origen en Data Migration Assistant](media/tutorial-sql-server-to-azure-sql-online/dma-source-connect.png)

6. Seleccione **Siguiente** en **Connect to target server** (Conectar a servidor de destino), especifique los detalles de conexión de destino de la base de datos de Azure SQL Database, seleccione **Conectar** y, luego, elija la base de datos **AdventureWorksAzure** que había aprovisionado previamente en Azure SQL Database.

    ![Detalles de conexión de destino en Data Migration Assistant](media/tutorial-sql-server-to-azure-sql-online/dma-target-connect.png)

7. Seleccione **Siguiente** para avanzar a la pantalla **Seleccionar objetos**, en la que puede especificar los objetos de esquema de la base de datos **AdventureWorks2012** que deben implementarse en Azure SQL Database.

    De forma predeterminada, se seleccionan todos los objetos.

    ![Generación de scripts SQL](media/tutorial-sql-server-to-azure-sql-online/dma-assessment-source.png)

8. Seleccione **Generar script SQL** para crear los scripts SQL y, luego, revíselos para ver si hay errores.

    ![Script de esquema](media/tutorial-sql-server-to-azure-sql-online/dma-schema-script.png)

9. Seleccione **Deploy schema** (Implementar esquema) para implementar el esquema en Azure SQL Database. Luego, una vez implementado, compruebe el servidor de destino por si hay cualquier anomalía.

    ![Implementación del esquema](media/tutorial-sql-server-to-azure-sql-online/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Registro del proveedor de recursos Microsoft.DataMigration

1. Inicie sesión en Azure Portal, seleccione **Todos los servicios** y, después, **Suscripciones**.

   ![Mostrar suscripciones en el portal](media/tutorial-sql-server-to-azure-sql-online/portal-select-subscription1.png)

2. Seleccione la suscripción en la que quiere crear la instancia de Azure Database Migration Service y después seleccione **Proveedores de recursos**.

    ![Mostrar los proveedores de recursos](media/tutorial-sql-server-to-azure-sql-online/portal-select-resource-provider.png)

3. Busque la migración y después, a la derecha de **Microsoft.DataMigration**, seleccione **Registrar**.

    ![Registro del proveedor de recursos](media/tutorial-sql-server-to-azure-sql-online/portal-register-resource-provider.png)

## <a name="create-an-instance"></a>Creación de una instancia

1. En Azure Portal, seleccione + **Crear un recurso**, busque Azure Database Migration Service y, después, seleccione **Azure Database Migration Service** en la lista desplegable.

    ![Azure Marketplace](media/tutorial-sql-server-to-azure-sql-online/portal-marketplace.png)

2. En la pantalla **Azure Database Migration Service**, seleccione **Crear**.

    ![Creación de una instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-create1.png)
  
3. En la pantalla **Crear el servicio de migración**, especifique un nombre para el servicio, la suscripción y un grupo de recursos nuevo o existente.

4. Seleccione la ubicación en la que quiere crear la instancia de Azure Database Migration Service. 

5. Seleccione una red virtual existente o cree una nueva.

    La red virtual proporciona a Azure Database Migration Service acceso a la instancia de SQL Server de origen y a la instancia de Azure SQL Database de destino.

    Para más información sobre cómo crear una red virtual en Azure Portal, consulte el artículo [Creación de una red virtual mediante Azure Portal](https://aka.ms/DMSVnet).

6. Seleccione un plan de tarifa.

    Para más información sobre los costos y planes de tarifa, vea la [página de precios](https://aka.ms/dms-pricing).

    ![Configuración de la instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-settings2.png)

7. Seleccione **Crear** para crear el servicio.

## <a name="create-a-migration-project"></a>Creación de un proyecto de migración

Después de crear el servicio, búsquelo en Azure Portal, ábralo y cree un proyecto de migración.

1. En Azure Portal, seleccione **Todos los servicios**, busque Azure Database Migration Service y, luego, elija **Azure Database Migration Services**.

    ![Búsqueda de todas las instancias de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-search.png)

2. En la pantalla **Azure Database Migration Services**, busque el nombre de la instancia de Azure Database Migration Service que creó y, después, seleccione la instancia.

    ![Busque la instancia de Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-instance-search.png)

3. Seleccione **+ New Migration Project** (+ Nuevo proyecto de migración).
4. En la pantalla **New Migration Project** (Nuevo proyecto de migración), especifique un nombre para el proyecto. En el cuadro de texto **Source server type** (Tipo de servidor de origen), seleccione **SQL Server** y, en el cuadro de texto **Target server type** (Tipo de servidor de destino), elija **Azure SQL Database**.
5. En la sección **Elegir el tipo de actividad**, seleccione **Migración de datos en línea**.

    ![Creación de un proyecto de Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-create-project3.png)

    > [!NOTE]
    > Como alternativa, puede elegir **Crear solo un proyecto** para crear el proyecto de migración ahora y ejecutar la migración más adelante.

6. Seleccione **Guardar**.

7. Seleccione **Crear y ejecutar una actividad** para crear el proyecto y ejecutar la actividad de migración.

    ![Creación y ejecución de una actividad de Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-create-and-run-activity.png)

## <a name="specify-source-details"></a>Especificación de los detalles de origen

1. En la pantalla **Detalles del origen de la migración**, especifique los detalles de conexión de la instancia de SQL Server de origen.

    Asegúrese de usar un nombre de dominio completo (FQDN) para el nombre de la instancia de SQL Server de origen. También puede usar la dirección IP en los casos en que no sea posible la resolución de nombres de DNS.

2. Si no ha instalado ningún certificado de confianza en el servidor de origen, seleccione la casilla de verificación **Certificado de servidor de confianza**.

    Si no hay ningún certificado de confianza instalado, SQL Server genera un certificado autofirmado cuando se inicia la instancia. Este certificado se usa para cifrar las credenciales de las conexiones del cliente.

    > [!CAUTION]
    > Las conexiones SSL cifradas mediante un certificado autofirmado no proporcionan una gran seguridad. Son susceptibles de sufrir ataques de tipo "Man in the middle". No se debe confiar en SSL mediante certificados autofirmados en un entorno de producción o en servidores que están conectados a Internet.

   ![Detalles del origen](media/tutorial-sql-server-to-azure-sql-online/dms-source-details3.png)

    > [!IMPORTANT]
    > Si usa SSIS, DMS no admite actualmente la migración de la SSISDB de origen, pero puede volver a implementar sus proyectos y paquetes SSIS en la SSISDB de destino hospedada por Azure SQL Database. Para más información acerca de la migración de paquetes de SSIS, consulte el artículo [Migración de paquetes de SQL Server Integration Services a Azure](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages).

## <a name="specify-target-details"></a>Especificación de los detalles de destino

1. Seleccione **Guardar** y después, en la pantalla **Detalles del destino de la migración**, especifique los detalles de conexión del servidor de Azure SQL Database de destino, que es la base de datos de Azure SQL que se aprovisionó previamente y en la que se implementó el esquema de **AdventureWorks2012** mediante Data Migration Assistant.

    ![Selección del destino](media/tutorial-sql-server-to-azure-sql-online/dms-select-target3.png)

2. Seleccione **Guardar** y, en la pantalla **Asignar a las bases de datos de destino**, asigne la base de datos de origen y de destino para la migración.

    Si la base de datos de destino contiene el mismo nombre de base de datos que la de origen, Azure Database Migration Service selecciona la base de datos de destino de forma predeterminada.

    ![Asignación a bases de datos de destino](media/tutorial-sql-server-to-azure-sql-online/dms-map-targets-activity3.png)

3. Seleccione **Guardar** y, en la pantalla **Seleccionar tablas**, expanda la lista de tablas y revise la lista de campos afectados.

    Azure Database Migration Service selecciona automáticamente todas las tablas de origen vacías que existen en la base de datos de Azure SQL de destino. Si quiere volver a migrar tablas que ya contienen datos, deberá seleccionar de forma explícita las tablas de esta hoja.

    ![Selección de tablas](media/tutorial-sql-server-to-azure-sql-online/dms-configure-setting-activity3.png)

4. Seleccione **Guardar**, en la pantalla **Resumen de migración**, en el cuadro de texto **Nombre de la actividad**, especifique un nombre para la actividad de migración y, a continuación, revise el resumen para asegurarse de que los detalles de origen y destino coinciden con lo que especificó anteriormente.

    ![Resumen de migración](media/tutorial-sql-server-to-azure-sql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Ejecución de la migración

- Seleccione **Ejecutar migración**.

    Aparecerá la ventana de actividad de migración. El **estado** de la actividad es **Inicializando**.

    ![Estado de la actividad: Inicializando](media/tutorial-sql-server-to-azure-sql-online/dms-activity-status2.png)

## <a name="monitor-the-migration"></a>Supervisión de la migración

1. En la pantalla de la actividad de migración, seleccione **Actualizar** para actualizar la vista hasta que el valor de **Estado** de la migración sea **En ejecución**.

2. Haga clic en una base de datos específica para obtener el estado de migración para las operaciones **Carga completa de los datos** y **Sincronización de datos incrementales**.

    ![Estado de la actividad: En curso](media/tutorial-sql-server-to-azure-sql-online/dms-activity-in-progress.png)

## <a name="perform-migration-cutover"></a>Realización de migración total

Una vez completada la carga completa inicial, las bases de datos se marcan como **A punto para la migración total**.

1. Cuando esté listo para completar la migración de la base de datos, seleccione **Iniciar transición**.

    ![Iniciar transición](media/tutorial-sql-server-to-azure-sql-online/dms-start-cutover.png)

2. Asegúrese de detener todas las transacciones entrantes a la base de datos de origen. Espere hasta que el contador **Cambios pendientes** muestre **0**.
3. Seleccione **Confirmar** y, después, **Aplicar**.
4. Cuando aparezca el estado **Completado** de la migración de base de datos, conecte las aplicaciones a la nueva base de datos de Azure SQL de destino.

    ![Estado de actividad Completado](media/tutorial-sql-server-to-azure-sql-online/dms-activity-completed.png)

## <a name="next-steps"></a>Pasos siguientes

- Laboratorio práctico sobre la [migración de SQL mediante Azure Data Migration Service (DMS)](https://www.microsoft.com/handsonlabs/SelfPacedLabs/?storyGuid=3b671509-c3cd-4495-8e8f-354acfa09587).
- Para más información sobre problemas y limitaciones conocidos al realizar migraciones en línea a Azure SQL Database, consulte el artículo [Problemas conocidos y soluciones para las migraciones en línea a Azure SQL Database](known-issues-azure-sql-online.md).
- Para más información sobre Azure Database Migration Service, consulte el artículo [¿Qué es Azure Database Migration Service?](https://docs.microsoft.com/azure/dms/dms-overview)
- Para más información sobre Azure SQL Database, consulte el artículo [¿Qué es el servicio Azure SQL Database?](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
