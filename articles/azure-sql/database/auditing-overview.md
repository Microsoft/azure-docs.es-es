---
title: Auditoría de Azure SQL para Azure SQL Database y Azure Synapse Analytics
description: Use la auditoría de Azure SQL Database para realizar el seguimiento de eventos de base de datos en un registro de auditoría.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 04/28/2020
ms.custom: azure-synapse, sqldbrb=1
ms.openlocfilehash: 24c3ec1ee16123cef0c4e2bd230bfdb66915fc9f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040581"
---
# <a name="auditing-for-azure-sql-database-and-azure-synapse-analytics"></a>Auditoría para Azure SQL Database y Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

La auditoría para [Azure SQL Database](sql-database-paas-overview.md) y [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) realiza el seguimiento de eventos de base de datos y los escribe en un registro de auditoría en la cuenta de Azure Storage, el área de trabajo de Log Analytics o Event Hubs.

La auditoría también puede hacer lo siguiente:

- Ayudar a mantener el cumplimiento de normativas, comprender la actividad de las bases de datos y conocer las discrepancias y anomalías que pueden indicar problemas en el negocio o infracciones de seguridad sospechosas.

- Posibilitar y facilitar la observancia de estándares reguladores aunque no garantiza el cumplimiento. Para obtener más información sobre los programas de Azure que se adhieren al cumplimiento normativo, vea el [Centro de confianza de Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942), donde podrá encontrar la lista más reciente de certificaciones de cumplimiento de Azure SQL.

> [!NOTE]
> Para obtener información sobre la auditoría de Instancia administrada de Azure SQL, vea el siguiente artículo [Introducción a la auditoría de Instancia administrada de SQL](../managed-instance/auditing-configure.md).

## <a name="overview"></a><a id="overview"></a>Información general

Puede usar la auditoría de SQL Database para:

- **Conservar** una traza de auditoría de eventos seleccionados. Puede definir categorías de acciones de base de datos para auditar.
- **Informar** sobre la actividad de la base de datos. Puede usar informes preconfigurados y un panel para dar los primeros pasos más rápido con el informe de actividades y eventos.
- **Analizar** informes. Puede buscar eventos sospechosos, actividades inusuales y tendencias.

> [!IMPORTANT]
> La auditoría de Azure SQL Database está optimizada para el rendimiento y la disponibilidad. Durante periodos de actividad muy elevada, Azure SQL Database o Azure Synapse permite que las operaciones continúen y es posible que no registre algunos eventos auditados.

### <a name="auditing-limitations"></a>Limitaciones de auditoría

- **Premium Storage** actualmente **no se admite**.
- **El espacio de nombres jerárquico** para la **cuenta de almacenamiento de Azure Data Lake Storage Gen2** actualmente **no se admite**.
- No es posible habilitar la auditoría en una instancia de **Azure Synapse** en pausa. Para habilitar la auditoría, reanude Azure Synapse.

#### <a name="define-server-level-vs-database-level-auditing-policy"></a><a id="server-vs-database-level"></a>Definir la directiva de auditoría de nivel de servidor frente la de nivel de base de datos

Se puede definir una directiva de auditoría para una base de datos específica o como directiva de [servidor](logical-servers.md) predeterminada en Azure (que aloja SQL Database o Azure Synapse):

- Una directiva de servidor se aplica a todas las bases de datos recién creadas en el servidor.

- Si la *auditoría de servidor está habilitada*, *se aplica siempre a la base de datos*. La base de datos se auditará, independientemente de la configuración de auditoría de la base de datos.

- Al habilitar la auditoría en la base de datos, además de en el servidor, *no* se invalida ni cambia ninguna de las opciones de la auditoría del servidor. Ambas auditorías existirán en paralelo. En otras palabras, la base de datos se auditará dos veces en paralelo; una vez por la directiva de servidor y otra vez por la directiva de base de datos.

   > [!NOTE]
   > Debe evitar habilitar tanto la auditoría de servidor como la auditoría de blobs de base de datos, a menos que:
    >
    > - Quiera usar una *cuenta de almacenamiento*, un *período de retención* o un *área de trabajo de Log Analytics* diferentes para una base de datos específica.
    > - Quiera auditar tipos de eventos o categorías para una base de datos específica que difieren del resto de las bases de datos del servidor. Por ejemplo, es posible que tenga inserciones de tabla que solo tengan que deban auditarse para una base de datos concreta.
   >
   > En caso contrario, se recomienda habilitar solo la auditoría de nivel de servidor y dejar que la auditoría de nivel de base de datos esté deshabilitada para todas las bases de datos.

## <a name="set-up-auditing-for-your-server"></a><a id="setup-auditing"></a>Configuración de la auditoría para su servidor

La directiva de auditoría predeterminada incluye todas las acciones y el siguiente conjunto de grupos de acciones, que auditarán todas las consultas y almacenarán los procedimientos ejecutados en la base de datos, así como los inicios de sesión correctos y erróneos:
  
- BATCH_COMPLETED_GROUP
- SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP
- FAILED_DATABASE_AUTHENTICATION_GROUP
  
Puede configurar la auditoría de los distintos tipos de acciones y grupos de acciones con PowerShell, como se describe en la sección [Administración de auditorías de SQL Database mediante Azure PowerShell](#manage-auditing).

La auditoría de Azure SQL Database y Azure Synapse almacena 4000 caracteres de datos para los campos de caracteres en un registro de auditoría. Cuando los valores de **statement** o **data_sensitivity_information** devueltos por una acción auditable contienen más de 4000 caracteres, todos los datos a partir de los primeros 4000 caracteres **se truncan y no se auditan**.
En la sección siguiente se describe la configuración de auditoría mediante Azure Portal.

  > [!NOTE]
  > No es posible habilitar la auditoría en un grupo de Synapse SQL en pausa. Para habilitar la auditoría, quite la pausa del grupo de Synapse SQL. Más información acerca del [grupo de Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql/best-practices-sql-pool).

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Vaya a **Auditoría** bajo el encabezado Seguridad en el panel de la **base de datos SQL** o el **servidor SQL**.
3. Si prefiere configurar una directiva de auditoría de servidor, puede seleccionar el vínculo **Ver configuración del servidor** en la hoja de auditoría de base de datos. Después, puede ver o modificar la configuración de auditoría del servidor. Las directivas de auditoría de servidor se aplican a todas las bases de datos existentes y recién creadas en este servidor.

    ![Panel de navegación](./media/auditing-overview/2_auditing_get_started_server_inherit.png)

4. Si prefiere habilitar la auditoría en el nivel de base de datos, cambie **Auditoría** a **Activado**. Si está habilitada la auditoría del servidor, la auditoría configurada de base de datos se producirá de forma paralela a la auditoría del servidor.

5. Tiene varias opciones para configurar dónde se escribirán los registros de auditoría. Puede escribir registros en una cuenta de almacenamiento de Azure, en un área de trabajo de Log Analytics para su consumo en registros de Azure Monitor (versión preliminar), o en un centro de eventos para consumirlos mediante el centro de eventos (versión preliminar). Puede configurar cualquier combinación de estas opciones, y los registros de auditoría se escribirán en cada una.
  
   ![opciones de almacenamiento](./media/auditing-overview/auditing-select-destination.png)

### <a name="audit-to-storage-destination"></a><a id="audit-storage-destination"></a>Auditoría para el destino de almacenamiento

Para configurar la escritura de registros de auditoría en una cuenta de almacenamiento, seleccione **Almacenamiento** y abra **Detalles de almacenamiento**. Seleccione la cuenta de almacenamiento de Azure donde se guardarán los registros y, después, seleccione el período de retención. A continuación, haga clic en **Aceptar**. Los registros anteriores al período de retención se eliminarán.

- El valor predeterminado para el período de retención es 0 (retención ilimitada). Puede cambiar este valor moviendo el control deslizante **Retención (días)** de **Configuración de almacenamiento** al configurar la cuenta de almacenamiento para la auditoría.
  - Si cambia el período de retención de 0 (retención ilimitada) a cualquier otro valor, tenga en cuenta que la retención solo se aplicará a los registros escritos una vez cambiado el valor de retención (los registros escritos durante el período en el que la retención se estableció en ilimitada se conservan, incluso después de habilitarse la retención).

  ![Cuenta de almacenamiento](./media/auditing-overview/auditing_select_storage.png)

#### <a name="remarks"></a>Observaciones

- Los registros de auditoría se escriben en **blobs en anexos** en Azure Blob Storage en su suscripción a Azure.
- Para configurar un almacén de registros inmutable para los eventos de auditoría de nivel de servidor o base de datos, siga las [instrucciones proporcionadas por Azure Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blob-immutability-policies-manage#enabling-allow-protected-append-blobs-writes). Asegúrese de que ha seleccionado **Permitir anexiones adicionales** al configurar el almacenamiento de blobs inmutable.
- Puede escribir registros de auditoría en una cuenta de Azure Storage detrás de un firewall o una red virtual. Para obtener instrucciones específicas, consulte cómo [escribir auditorías en una cuenta de almacenamiento detrás de una red virtual y un firewall](audit-write-storage-account-behind-vnet-firewall.md).
- Después de configurar los valores de auditoría, puede activar la nueva característica de detección de amenazas y configurar los mensajes de correo para recibir alertas de seguridad. Cuando se usa la detección de amenazas, se reciben alertas proactivas sobre actividades anómalas de la base de datos que pueden indicar posibles amenazas de seguridad. Para más información, vea [Introducción a la detección de amenazas](threat-detection-overview.md).
- Para obtener más información sobre el formato de registro, la jerarquía de la carpeta de almacenamiento y las convenciones de nomenclatura, vea la [referencia del formato de registro de auditoría de blobs](https://go.microsoft.com/fwlink/?linkid=829599).
- Cuando se usa la autenticación de Azure AD, los registros de inicios de sesión con error *no* aparecerán en el registro de auditoría SQL. Para ver los registros de auditoría de inicio de sesión con error, debe visitar el [portal de Azure Active Directory](../../active-directory/reports-monitoring/reference-sign-ins-error-codes.md), que registra los detalles de estos eventos.
- La auditoría en las [réplicas de solo lectura](read-scale-out.md) se habilita automáticamente. Para obtener más información sobre la jerarquía de las carpetas de almacenamiento, las convenciones de nomenclatura y el formato del registro, consulte el artículo sobre el [formato del registro de auditoría de SQL Database](audit-log-format.md).

### <a name="audit-to-log-analytics-destination"></a><a id="audit-log-analytics-destination"></a>Auditoría para el destino de Log Analytics
  
Para configurar la escritura de registros de auditoría en un área de trabajo de Log Analytics, seleccione **Log Analytics (versión preliminar)** y abra **Detalles de Log Analytics**. Seleccione o cree el área de trabajo de Log Analytics donde se escribirán los registros y, luego, haga clic en **Aceptar**.

   ![LogAnalyticsworkspace](./media/auditing-overview/auditing_select_oms.png)

Para información más detallada sobre las áreas de trabajo de los registros de Azure Monitor, consulte [Diseño de la implementación de registros de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/design-logs-deployment).
   
### <a name="audit-to-event-hub-destination"></a><a id="audit-event-hub-destination"></a>Auditoría para un destino del centro de eventos

Para configurar la escritura de registros de auditoría en un centro de eventos, seleccione **Centro de eventos (versión preliminar)** y abra **Detalles del centro de eventos**. Seleccione el centro de eventos donde se escribirán los registros y, luego, haga clic en **Aceptar**. Asegúrese de que el centro de eventos esté en la misma región que la base de datos y el servidor.

   ![Eventhub](./media/auditing-overview/auditing_select_event_hub.png)

## <a name="analyze-audit-logs-and-reports"></a><a id="subheading-3"></a>Análisis de registros e informes de auditoría

Si eligió escribir registros de auditoría en registros de Azure Monitor:

- Usar [Azure Portal](https://portal.azure.com). Abra la base de datos pertinente. En la parte superior de la página **Auditoría** de la base de datos, seleccione **Ver registros de auditoría**.

    ![ver registros de auditoría](./media/auditing-overview/auditing-view-audit-logs.png)

- A continuación, dispone de dos formas de ver los registros:

    Al hacer clic en **Log Analytics** en la parte superior de la página **Registros de auditoría**, se abrirá la vista de registros en el área de trabajo de Log Analytics, donde puede personalizar el intervalo de tiempo y la consulta de búsqueda.

    ![abrir en el área de trabajo de Log Analytics](./media/auditing-overview/auditing-log-analytics.png)

    Al hacer clic en **Ver panel** en la parte superior de la página **Registros de auditoría**, se abrirá un panel con información sobre los registros de auditoría, donde puede explorar en profundidad la información de seguridad, el acceso a datos confidenciales y mucho más. Este panel se ha diseñado para ayudarle a obtener información de seguridad para sus datos.
    También puede personalizar el intervalo de tiempo y la consulta de búsqueda.
    ![Ver panel de Log Analytics](media/auditing-overview/auditing-view-dashboard.png)

    ![Panel de Log Analytics](media/auditing-overview/auditing-log-analytics-dashboard.png)

    ![Información de seguridad de Log Analytics](media/auditing-overview/auditing-log-analytics-dashboard-data.png)

- Como alternativa, también puede acceder a los registros de auditoría desde la hoja Log Analytics. Abra el área de trabajo de Log Analytics y, en la sección **General**, haga clic en **Registros**. Puede comenzar con una consulta simple, como: *buscar "SQLSecurityAuditEvents"* para ver los registros de auditoría.
    Desde aquí, también puede usar los [registros de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) para ejecutar búsquedas avanzadas en los datos de registro de auditoría. Los registros de Azure Monitor proporcionan conclusiones operativas en tiempo real gracias a uso de paneles personalizados y de búsqueda integrados para analizar fácilmente millones de registros en todas las cargas de trabajo y servidores. Para información útil adicional sobre los comandos y el lenguaje de búsqueda de registros de Azure Monitor, consulte la [referencia de búsqueda de registros de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

Si eligió escribir registros de auditoría en el centro de eventos:

- Para consumir datos de registros de auditoría desde el centro de eventos, deberá configurar una secuencia que consuma eventos y los escriba en un destino. Para más información, consulte la [documentación de Azure Event Hubs](../index.yml).
- Los registros de auditoría del Centro de eventos se capturan en el cuerpo de los eventos de [Apache Avro](https://avro.apache.org/) y se almacenan con el formato JSON con codificación UTF-8. Para leer los registros de auditoría, puede usar [Avro Tools](../../event-hubs/event-hubs-capture-overview.md#use-avro-tools) o herramientas similares que procesen este formato.

Si eligió escribir los registros de auditoría en una cuenta de almacenamiento de Azure, hay varios métodos que puede usar para ver los registros:

- Los registros de auditoría se agregan a la cuenta que eligió durante la configuración. Puede explorar los registros de auditoría con una herramienta como el [Explorador de Azure Storage](https://storageexplorer.com/). En Azure Storage, los registros de auditoría se guardan como una colección de archivos de blob dentro de un contenedor llamado **sqldbauditlogs**. Para obtener más información sobre la jerarquía de las carpetas de almacenamiento, las convenciones de nomenclatura y el formato del registro, consulte el artículo sobre el [formato del registro de auditoría de SQL Database](https://go.microsoft.com/fwlink/?linkid=829599).

- Usar [Azure Portal](https://portal.azure.com).  Abra la base de datos pertinente. En la parte superior de la página **Auditoría** de la base de datos, haga clic en **Ver registros de auditoría**.

    ![Panel de navegación](./media/auditing-overview/7_auditing_get_started_blob_view_audit_logs.png)

    Se abre la hoja **Registros de auditoría**, desde la que podrá ver los registros.

  - Puede elegir ver fechas específicas si hace clic en **Filtrar** en la parte superior de la página **Registros de auditoría**.
  - Puede cambiar entre los registros de auditoría que se crearon con la *directiva de auditoría de servidor* y la *directiva de auditoría de base de datos* alternando **Origen de auditoría**.
  - Para ver solo los registros de auditoría relacionados con la inyección de código SQL, puede activar la casilla **Mostrar solo los registros de auditoría de inyección de código SQL**.

       ![Panel de navegación]( ./media/auditing-overview/8_auditing_get_started_blob_audit_records.png)

- Use la función del sistema **sys.fn_get_audit_file** (T-SQL) para devolver los datos de registro de auditoría en formato tabular. Para más información sobre el uso de esta función, consulte [sys.fn_get_audit_file](/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql).

- Use **Combinar archivos de auditoría** en SQL Server Management Studio (a partir de SSMS 17):
    1. En el menú SSMS, seleccione **Archivo** > **Abrir** > **Combinar archivos de auditoría**.

        ![Panel de navegación](./media/auditing-overview/9_auditing_get_started_ssms_1.png)
    2. Se abre el cuadro de diálogo **Agregar archivos de auditoría**. Seleccione una de las opciones **Agregar** para elegir si quiere combinar los archivos de auditoría desde un disco local o importarlos desde Azure Storage. Debe proporcionar los detalles de Azure Storage y la clave de cuenta.

    3. Una vez agregados todos los archivos que se van a combinar, haga clic en **Aceptar** para completar la operación de combinación.

    4. El archivo combinado se abre en SSMS, donde puede verlo y analizarlo, y también exportarlo a un archivo XEL o CSV, o a una tabla.

- Use Power BI. Puede ver y analizar los datos de registro de auditoría en Power BI. Para obtener más información y para acceder a una plantilla que se puede descargar, consulte el [análisis de datos de registro de auditoría en Power BI](https://techcommunity.microsoft.com/t5/azure-database-support-blog/sql-azure-blob-auditing-basic-power-bi-dashboard/ba-p/368895).
- Descargue los archivos de registro del contenedor de blobs de Azure Storage mediante el portal o con una herramienta como el [Explorador de Azure Storage](https://storageexplorer.com/).
  - Después de descargar localmente un archivo de registro, haga doble clic en él para abrir, ver y analizar los registros en SSMS.
  - También puede descargar varios archivos al mismo tiempo a través del Explorador de Azure Storage. Para ello, haga clic con el botón derecho en una subcarpeta específica y seleccione **Guardar como** para guardarlos en una carpeta local.

- Otros métodos:

  - Después de descargar varios archivos o una subcarpeta que contenga archivos de registro, puede combinarlos localmente como se describe en las instrucciones anteriores para combinar archivos de auditoría de SSMS.
  - Ver los registros de auditoría de blobs mediante programación:

    - [Consulta de archivos de eventos extendidos](https://sqlscope.wordpress.com/2014/11/15/reading-extended-event-files-using-client-side-tools-only/) mediante PowerShell.

## <a name="production-practices"></a><a id="production-practices"></a>Prácticas de producción


### <a name="auditing-geo-replicated-databases"></a>Auditoría de bases de datos con replicación geográfica

Con bases de datos con replicación geográfica, cuando se habilita la auditoría en la base de datos principal, la base de datos secundaria tendrá una directiva de auditoría idéntica. También es posible configurar la auditoría en la base de datos secundaria habilitando la auditoría en el **servidor secundario**, independientemente de la base de datos principal.

- Nivel de servidor (**recomendado**): active la auditoría en el **servidor principal** así como en el **servidor secundario**. Las bases de datos principal y secundaria se auditarán de forma independiente en función de la directiva de nivel de servidor respectiva.
- Nivel de base de datos: la auditoría en el nivel de base de datos para las bases de datos secundarias solo se puede configurar desde la configuración de auditoría de la base de datos principal.
  - La auditoría debe estar habilitada en la *propia base de datos principal*, no en el servidor.
  - Después de habilitar la auditoría en la base de datos principal, también se habilitará en la base de datos secundaria.

    > [!IMPORTANT]
    > Con la auditoría en el nivel de base de datos, la configuración de almacenamiento de la base de datos secundaria será idéntica a la de la base de datos principal, lo que provocará tráfico interregional. Se recomienda habilitar solo la auditoría de nivel de servidor y dejar que la auditoría de nivel de base de datos esté deshabilitada para todas las bases de datos.

### <a name="storage-key-regeneration"></a>Regeneración de clave de almacenamiento

En el entorno de producción, es probable que actualice periódicamente las claves de almacenamiento. Al escribir registros de auditoría en Azure Storage, debe volver a guardar la directiva de auditoría cuando se actualicen las claves. El proceso es el siguiente:

1. Abra **Detalles de almacenamiento**. En el cuadro **Clave de acceso de almacenamiento**, seleccione **Secundaria** y haga clic en **Aceptar**. Después, haga clic en **Guardar** en la parte superior de la página de configuración de auditoría.

    ![Panel de navegación](./media/auditing-overview/5_auditing_get_started_storage_key_regeneration.png)
2. Vaya al panel de configuración de almacenamiento y vuelva a generar la clave de acceso principal.

    ![Panel de navegación](./media/auditing-overview/6_auditing_get_started_regenerate_key.png)
3. Vuelva a la página de configuración de auditoría, cambie el valor de la clave de acceso de almacenamiento de secundaria a principal y haga clic en **Aceptar**. Después, haga clic en **Guardar** en la parte superior de la página de configuración de auditoría.
4. Vuelva a la página de configuración de almacenamiento y vuelva a generar la clave de acceso secundaria (como preparación para el siguiente ciclo de actualización de claves).

## <a name="manage-azure-sql-database-auditing"></a><a id="manage-auditing"></a>Administración de la auditoría de Azure SQL Database

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

**Cmdlets de PowerShell (incluye compatibilidad con la cláusula WHERE para filtrado adicional)** :

- [Crear o actualizar la directiva de auditoría de base de datos (Set-AzSqlDatabaseAudit)](/powershell/module/az.sql/set-azsqldatabaseaudit)
- [Crear o actualizar la directiva de auditoría de servidor (Set-AzSqlServerAudit)](/powershell/module/az.sql/set-azsqlserveraudit)
- [Obtener la directiva de auditoría de base de datos (Get-AzSqlDatabaseAudit)](/powershell/module/az.sql/get-azsqldatabaseaudit)
- [Obtener la directiva de auditoría de servidor (Get-AzSqlServerAudit)](/powershell/module/az.sql/get-azsqlserveraudit)
- [Quitar la directiva de auditoría de base de datos (Remove-AzSqlDatabaseAudit)](/powershell/module/az.sql/remove-azsqldatabaseaudit)
- [Quitar la directiva de auditoría de servidor (Remove-AzSqlServerAudit)](/powershell/module/az.sql/remove-azsqlserveraudit)

Para ver un script de ejemplo, consulte [Configuración de la auditoría y detección de amenazas mediante PowerShell](scripts/auditing-threat-detection-powershell-configure.md).

### <a name="using-rest-api"></a>Uso de la API de REST

**API REST**:

- [Create or Update Database Auditing Policy](/rest/api/sql/database%20auditing%20settings/createorupdate) (Creación o actualización de la directiva de auditoría de la base de datos)
- [Create or Update Server Auditing Policy](/rest/api/sql/server%20auditing%20settings/createorupdate) (Creación o actualización de la directiva de auditoría del servidor)
- [Get Database Auditing Policy](/rest/api/sql/database%20auditing%20settings/get) (Obtención de la directiva de auditoría de la base de datos)
- [Get Server Auditing Policy](/rest/api/sql/server%20auditing%20settings/get) (Obtención de la directiva de auditoría del servidor)

Directiva extendida compatible con la cláusula WHERE para filtrado adicional:

- [Crear o actualizar la directiva de auditoría de base de datos *extendida*](/rest/api/sql/database%20extended%20auditing%20settings/createorupdate)
- [Crear o actualizar la directiva de auditoría de servidor *extendida*](/rest/api/sql/server%20auditing%20settings/createorupdate)
- [Obtener la directiva de auditoría de base de datos *extendida*](/rest/api/sql/database%20extended%20auditing%20settings/get)
- [Obtener la directiva de auditoría de servidor *extendida*](/rest/api/sql/server%20auditing%20settings/get)

### <a name="using-azure-resource-manager-templates"></a>Uso de plantillas del Administrador de recursos de Azure

Puede administrar auditorías de Azure SQL Database mediante plantillas de [Azure Resource Manager](../../azure-resource-manager/management/overview.md), como se muestra en estos ejemplos:

- [Implementación de una instancia de Azure SQL Database con la auditoría habilitada para escribir los registros de auditoría en la cuenta de Azure Blob Storage](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-blob-storage)
- [Implementación de una instancia de Azure SQL Database con la auditoría habilitada para escribir los registros de auditoría en Log Analytics](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-oms)
- [Implementación de una instancia de Azure SQL Database con la auditoría habilitada para escribir los registros de auditoría en Event Hubs](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-eventhub)

> [!NOTE]
> Los ejemplos vinculados se encuentran en un repositorio público externo y se proporcionan "tal cual", sin ninguna garantía y no se admiten en todos los programas o servicios de soporte técnico de Microsoft.
