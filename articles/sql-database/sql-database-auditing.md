---
title: Introducción a la auditoría de Azure SQL Database | Microsoft Docs
description: Utilice la auditoría de base de datos de Azure SQL para realizar el seguimiento de eventos de base de datos en un registro de auditoría.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: vainolo
ms.author: arib
ms.reviewer: vanto
manager: craigg
ms.date: 04/16/2019
ms.openlocfilehash: 3efdf5c256a22529c9d19e9ae1dce5d2db9516a5
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827770"
---
# <a name="get-started-with-sql-database-auditing"></a>Introducción a la auditoría de bases de datos SQL

La auditoría de Azure [SQL Database](sql-database-technical-overview.md) y [SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md) realiza un seguimiento de eventos de base de datos y los escribe en un registro de auditoría de la cuenta de Azure Storage, área de trabajo OMS o Event Hubs. La auditoría también:

- Puede ayudarle a mantener el cumplimiento de normativas, comprender la actividad de las bases de datos y conocer las discrepancias y anomalías que pueden indicar problemas en el negocio o infracciones de seguridad sospechosas.

- Posibilita y facilita la observancia de estándares reguladores aunque no garantiza el cumplimiento. Para obtener más información sobre Azure programas ese cumplimiento de estándares de soporte técnico, consulte el [Azure Trust Center](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) donde puede encontrar la lista más actualizada de certificaciones de cumplimiento de la base de datos SQL.


> [!NOTE] 
> Este tema se aplica al servidor de Azure SQL y tanto a las bases de datos de SQL Database como a SQL Data Warehouse que se crean en el servidor de Azure SQL. Para simplificar, SQL Database se utiliza cuando se hace referencia tanto a SQL Database como a SQL Data Warehouse.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]


## <a id="subheading-1"></a>Información general sobre auditoría en Azure SQL Database

Puede usar la auditoría de base de datos SQL para:

- **Conservar** una traza de auditoría de eventos seleccionados. Puede definir categorías de acciones de base de datos para auditar.
- **Informar** sobre la actividad de la base de datos. Puede usar informes preconfigurados y un panel para dar los primeros pasos más rápido con el informe de actividades y eventos.
- **Analizar** informes. Puede buscar eventos sospechosos, actividades inusuales y tendencias.

> [!IMPORTANT]
> Los registros de auditoría se escriben en **Anexar blobs** en Azure Blob Storage en la suscripción de Azure.
>
> - Se admiten todos los tipos de almacenamiento (v1, v2 y blob).
> - Se admiten todas las configuraciones de replicación de almacenamiento.
> - **Premium Storage** actualmente **no se admite**.
> - **El almacenamiento en VNet** actualmente **no se admite**.
> - **El almacenamiento detrás de un firewall** actualmente **no se admite**.

## <a id="subheading-8"></a>Definir la directiva de auditoría de nivel de servidor frente la de nivel de base de datos

Puede definirse una directiva de auditoría para una base de datos específica o como directiva de servidor predeterminada:

- Una directiva de servidor se aplica a todas las bases de datos recién creadas en el servidor.

- Si la *auditoría de blobs de servidor* está habilitada, *se aplica siempre a la base de datos*. La base de datos se auditará, independientemente de la configuración de auditoría de la base de datos.

- Habilitar la auditoría de blobs en la base de datos o el almacenamiento de datos, además de en el servidor, *no* invalida ni cambia ninguna de las opciones de la auditoría del servidor de blobs. Ambas auditorías existirán en paralelo. En otras palabras, la base de datos se auditará dos veces en paralelo; una vez por la directiva de servidor y otra vez por la directiva de base de datos.

   > [!NOTE]
   > Debe habilitar tanto la auditoría de blobs de servidor como la auditoría de blobs de base de datos, a menos que:
    > - Quiera usar una *cuenta de almacenamiento* o un *período de retención* diferentes para una base de datos específica.
    > - Quiera auditar tipos de eventos o categorías para una base de datos específica que difieren del resto de las bases de datos del servidor. Por ejemplo, es posible que tenga inserciones de tabla que solo tengan que deban auditarse para una base de datos concreta.
   >
   > En caso contrario, se recomienda habilitar solo la auditoría de blobs de nivel de servidor y dejar que la auditoría de nivel de base de datos esté deshabilitada para todas las bases de datos.

## <a id="subheading-2"></a>Configuración de la auditoría para su base de datos

En la sección siguiente se describe la configuración de auditoría mediante Azure Portal.

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Vaya a **Auditoría** bajo el encabezado Seguridad en el panel de la base de datos/servidor SQL.

    <a id="auditing-screenshot"></a>![Panel de navegación][1]

3. Si prefiere configurar una directiva de auditoría de servidor, puede seleccionar el vínculo **Ver configuración del servidor** en la hoja de auditoría de base de datos. Después, puede ver o modificar la configuración de auditoría del servidor. Las directivas de auditoría de servidor se aplican a todas las bases de datos existentes y recién creadas en este servidor.

    ![Panel de navegación][2]

4. Si prefiere habilitar la auditoría en el nivel de base de datos, cambie **Auditoría** a **Activado**.

    Si está habilitada la auditoría del servidor, la auditoría configurada de base de datos se producirá de forma paralela a la auditoría del servidor.

    ![Panel de navegación][3]

5. **Nuevo**: ahora tiene varias opciones para configurar dónde se escribirán los registros de auditoría. Puede escribir registros en una cuenta de almacenamiento de Azure, a un área de trabajo de Log Analytics para su uso en los registros de Azure Monitor o al centro de eventos para usarlos con el centro de eventos. Puede configurar cualquier combinación de estas opciones, y los registros de auditoría se escribirán en cada una.

   > [!WARNING]
   > Habilitar la auditoría a Log Analytics incurrirá en costos en función de las tasas de ingesta. Tenga en cuenta el costo asociado con el uso de este [opción](https://azure.microsoft.com/pricing/details/monitor/), o considere la posibilidad de almacenar la auditoría registra en una cuenta de almacenamiento de Azure.

    ![opciones de almacenamiento](./media/sql-database-auditing-get-started/auditing-select-destination.png)

6. Para configurar la escritura de registros de auditoría en una cuenta de almacenamiento, seleccione **Almacenamiento** y abra **Detalles de almacenamiento**. Seleccione la cuenta de almacenamiento de Azure donde se guardarán los registros y, después, seleccione el período de retención. Se eliminarán los registros antiguos. A continuación, haga clic en **Aceptar**.

    ![cuenta de almacenamiento](./media/sql-database-auditing-get-started/auditing_select_storage.png)

7. Para configurar la escritura de registros de auditoría en un área de trabajo de Log Analytics, seleccione **Log Analytics (versión preliminar)** y abra **Detalles de Log Analytics**. Seleccione o cree el área de trabajo de Log Analytics donde se escribirán los registros y, luego, haga clic en **Aceptar**.

    ![Área de trabajo de Log Analytics](./media/sql-database-auditing-get-started/auditing_select_oms.png)

8. Para configurar la escritura de registros de auditoría en un centro de eventos, seleccione **Centro de eventos (versión preliminar)** y abra **Detalles del centro de eventos**. Seleccione el centro de eventos donde se escribirán los registros y, luego, haga clic en **Aceptar**. Asegúrese de que el centro de eventos esté en la misma región que la base de datos y el servidor.

    ![Centro de eventos](./media/sql-database-auditing-get-started/auditing_select_event_hub.png)

9. Haga clic en **Save**(Guardar).
10. Si quiere personalizar los eventos auditados, puede hacerlo a través de los [cmdlets de PowerShell](#subheading-7) o de la [API de REST](#subheading-9).
11. Después de configurar los valores de auditoría, puede activar la nueva característica de detección de amenazas y configurar los mensajes de correo para recibir alertas de seguridad. Cuando se usa la detección de amenazas, se reciben alertas proactivas sobre actividades anómalas de la base de datos que pueden indicar posibles amenazas de seguridad. Para más información, vea [Introducción a la detección de amenazas](sql-database-threat-detection-get-started.md).

> [!IMPORTANT]
> No es posible habilitar la auditoría en Azure SQL Data Warehouse en pausa. Para habilitarla, finaliza la pausa del almacenamiento de datos.

> [!WARNING]
> Habilitar la auditoría en un servidor que tenga una Azure SQL Data Warehouse en él **dará como resultado en el almacén de datos que se reanude y volver a en pausa nuevo** que incurran en cargos de facturación.

## <a id="subheading-3"></a>Análisis de registros e informes de auditoría

Si decide escribir los registros de auditoría en los registros de Azure Monitor:

- Usar [Azure Portal](https://portal.azure.com).  Abra la base de datos pertinente. En la parte superior de la página **Auditoría** de la base de datos, haga clic en **Ver registros de auditoría**.

    ![ver registros de auditoría](./media/sql-database-auditing-get-started/7_auditing_get_started_blob_view_audit_logs.png)

- A continuación, al hacer clic en **Open in OMS** (Abrir en OMS) en la parte superior de la página **Registros de auditoría**, se abrirá la vista de registros en Log Analytics, donde puede personalizar el intervalo de tiempo y la consulta de búsqueda.

    ![abrir en Log Analytics](./media/sql-database-auditing-get-started/auditing_open_in_oms.png)

- Como alternativa, también puede acceder a los registros de auditoría desde la hoja Log Analytics. Abra el área de trabajo de Log Analytics y, en la sección **General**, haga clic en **Registros**. Puede comenzar con una consulta simple, como: *buscar "SQLSecurityAuditEvents"* para ver los registros de auditoría.
    Desde aquí, también puede usar [registros de Azure Monitor](../log-analytics/log-analytics-log-search.md) para ejecutar las búsquedas avanzadas en los datos de registro de auditoría. Registros de Azure Monitor le ofrece visión operativa en tiempo real mediante búsqueda integrada y paneles personalizados para analizar fácilmente millones de registros en todas las cargas de trabajo y servidores. Para obtener información útil adicional sobre los comandos y lenguaje de búsqueda de registros de Azure Monitor, consulte [referencia sobre búsqueda de registros de Azure Monitor](../log-analytics/log-analytics-log-search.md).

Si eligió escribir registros de auditoría en el centro de eventos:

- Para consumir datos de registros de auditoría desde el centro de eventos, deberá configurar una secuencia que consuma eventos y los escriba en un destino. Para más información, consulte la [documentación de Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/).
- Los registros de auditoría del Centro de eventos se capturan en el cuerpo de los eventos de [Apache Avro](https://avro.apache.org/) y se almacenan con el formato JSON con codificación UTF-8. Para leer los registros de auditoría, puede usar [Avro Tools](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview#use-avro-tools) o herramientas similares que procesen este formato.

Si eligió escribir los registros de auditoría en una cuenta de almacenamiento de Azure, hay varios métodos que puede usar para ver los registros:

- Los registros de auditoría se agregan a la cuenta que eligió durante la configuración. Puede explorar los registros de auditoría con una herramienta como el [Explorador de Azure Storage](https://storageexplorer.com/). En Azure Storage, los registros de auditoría se guardan como una colección de archivos de blob dentro de un contenedor llamado **sqldbauditlogs**. Para obtener más información sobre la jerarquía de la carpeta de almacenamiento, las convenciones de nomenclatura y el formato del registro, vea la [referencia del formato de registro de auditoría de blobs](https://go.microsoft.com/fwlink/?linkid=829599).

- Usar [Azure Portal](https://portal.azure.com).  Abra la base de datos pertinente. En la parte superior de la página **Auditoría** de la base de datos, haga clic en **Ver registros de auditoría**.

    ![Panel de navegación][7]

    Se abre la hoja **Registros de auditoría**, desde la que podrá ver los registros.

  - Puede elegir ver fechas específicas si hace clic en **Filtrar** en la parte superior de la página **Registros de auditoría**.
  - Puede cambiar entre los registros de auditoría que se crearon con la *directiva de auditoría de servidor* y la *directiva de auditoría de base de datos* alternando **Origen de auditoría**.
  - Para ver solo los registros de auditoría relacionados con la inyección de código SQL, puede activar la casilla **Mostrar solo los registros de auditoría de inyección de código SQL**.

       ![Panel de navegación][8]

- Use la función del sistema **sys.fn_get_audit_file** (T-SQL) para devolver los datos de registro de auditoría en formato tabular. Para más información sobre el uso de esta función, consulte [sys.fn_get_audit_file](https://docs.microsoft.com/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql).

- Use **Combinar archivos de auditoría** en SQL Server Management Studio (a partir de SSMS 17):
    1. En el menú SSMS, seleccione **Archivo** > **Abrir** > **Combinar archivos de auditoría**.

        ![Panel de navegación][9]
    2. Se abre el cuadro de diálogo **Agregar archivos de auditoría**. Seleccione una de las opciones **Agregar** para elegir si quiere combinar los archivos de auditoría desde un disco local o importarlos desde Azure Storage. Debe proporcionar los detalles de Azure Storage y la clave de cuenta.

    3. Una vez agregados todos los archivos que se van a combinar, haga clic en **Aceptar** para completar la operación de combinación.

    4. El archivo combinado se abre en SSMS, donde puede verlo y analizarlo, y también exportarlo a un archivo XEL o CSV, o a una tabla.

- Use Power BI. Puede ver y analizar los datos de registro de auditoría en Power BI. Para obtener más información y para acceder a una plantilla que se puede descargar, consulte el [análisis de datos de registro de auditoría en Power BI](https://blogs.msdn.microsoft.com/azuresqldbsupport/20../../sql-azure-blob-auditing-basic-power-bi-dashboard/).
- Descargue los archivos de registro del contenedor de blobs de Azure Storage mediante el portal o con una herramienta como el [Explorador de Azure Storage](https://storageexplorer.com/).
  - Después de descargar localmente un archivo de registro, haga doble clic en él para abrir, ver y analizar los registros en SSMS.
  - También puede descargar varios archivos al mismo tiempo a través del Explorador de Azure Storage. Para ello, haga clic con el botón derecho en una subcarpeta específica y seleccione **Guardar como** para guardarlos en una carpeta local.

- Otros métodos:

  - Después de descargar varios archivos o una subcarpeta que contenga archivos de registro, puede combinarlos localmente como se describe en las instrucciones anteriores para combinar archivos de auditoría de SSMS.
  - Ver los registros de auditoría de blobs mediante programación:

    - [Consulta de archivos de eventos extendidos](https://sqlscope.wordpress.com/20../../reading-extended-event-files-using-client-side-tools-only/) mediante PowerShell.

## <a id="subheading-5"></a>Prácticas de producción

<!--The description in this section refers to preceding screen captures.-->

### <a id="subheading-6">Auditoría de bases de datos con replicación geográfica</a>

Con bases de datos con replicación geográfica, cuando se habilita la auditoría en la base de datos principal, la base de datos secundaria tendrá una directiva de auditoría idéntica. También es posible configurar la auditoría en la base de datos secundaria habilitando la auditoría en el **servidor secundario**, independientemente de la base de datos principal.

- Nivel de servidor (**recomendado**): active la auditoría en el **servidor principal** así como en el **servidor secundario**. Las bases de datos principal y secundaria se auditarán de forma independiente en función de la directiva de nivel de servidor respectiva.
- Nivel de base de datos: la auditoría en el nivel de base de datos para las bases de datos secundarias solo se puede configurar desde la configuración de auditoría de la base de datos principal.
  - La auditoría debe estar habilitada en la *propia base de datos principal*, no en el servidor.
  - Después de habilitar la auditoría en la base de datos principal, también se habilitará en la base de datos secundaria.

    >[!IMPORTANT]
    >Con la auditoría en el nivel de base de datos, la configuración de almacenamiento de la base de datos secundaria será idéntica a la de la base de datos principal, lo que provocará tráfico interregional. Se recomienda habilitar solo la auditoría de nivel de servidor y dejar que la auditoría de nivel de base de datos esté deshabilitada para todas las bases de datos.
    > [!WARNING]
    > Uso de centro de eventos o registros de Azure Monitor como destinos para los registros de auditoría en el nivel de servidor no se admite para bases de datos secundaria con replicación geográfica.

### <a id="subheading-6">Regeneración de clave de almacenamiento</a>

En el entorno de producción, es probable que actualice periódicamente las claves de almacenamiento. Al escribir registros de auditoría en Azure Storage, debe volver a guardar la directiva de auditoría cuando se actualicen las claves. El proceso es el siguiente:

1. Abra **Detalles de almacenamiento**. En el cuadro **Clave de acceso de almacenamiento**, seleccione **Secundaria** y haga clic en **Aceptar**. Después, haga clic en **Guardar** en la parte superior de la página de configuración de auditoría.

    ![Panel de navegación][5]
2. Vaya al panel de configuración de almacenamiento y vuelva a generar la clave de acceso principal.

    ![Panel de navegación][6]
3. Vuelva a la página de configuración de auditoría, cambie el valor de la clave de acceso de almacenamiento de secundaria a principal y haga clic en **Aceptar**. Después, haga clic en **Guardar** en la parte superior de la página de configuración de auditoría.
4. Vuelva a la página de configuración de almacenamiento y vuelva a generar la clave de acceso secundaria (como preparación para el siguiente ciclo de actualización de claves).

## <a name="additional-information"></a>Información adicional

- Para obtener más información sobre el formato de registro, la jerarquía de la carpeta de almacenamiento y las convenciones de nomenclatura, vea la [referencia del formato de registro de auditoría de blobs](https://go.microsoft.com/fwlink/?linkid=829599).

    > [!IMPORTANT]
    > La auditoría de Azure SQL Database almacena 4000 caracteres de datos para los campos de caracteres en un registro de auditoría. Cuando los valores de **statement** o **data_sensitivity_information** devueltos por una acción auditable contienen más de 4000 caracteres, todos los datos a partir de los primeros 4000 caracteres **se truncan y no se auditan**.

- Los registros de auditoría se escriben en **Anexar blobs** en Azure Blob Storage en la suscripción de Azure:
  - **Premium Storage** actualmente **no es compatible** con Append Blobs.
  - **El almacenamiento en VNet** actualmente **no se admite**.

- La directiva de auditoría predeterminada incluye todas las acciones y el siguiente conjunto de grupos de acciones, que auditarán todas las consultas y almacenarán los procedimientos ejecutados en la base de datos, así como los inicios de sesión correctos y erróneos:

    BATCH_COMPLETED_GROUP<br>
    SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP<br>
    FAILED_DATABASE_AUTHENTICATION_GROUP

    Puede configurar la auditoría de los distintos tipos de acciones y grupos de acciones con PowerShell, según se describe en la sección [Administración de auditorías de SQL Database mediante Azure PowerShell](#subheading-7).

- Cuando se usa la autenticación de Azure AD, los registros de inicios de sesión con error *no* aparecerán en el registro de auditoría SQL. Para ver los registros de auditoría de inicio de sesión con error, debe visitar el [portal de Azure Active Directory]( ../active-directory/reports-monitoring/reference-sign-ins-error-codes.md), que registra los detalles de estos eventos.


## <a id="subheading-7"></a>Administración de auditorías de SQL Database mediante Azure PowerShell

**Cmdlets de PowerShell (incluye compatibilidad con la cláusula WHERE para filtrado adicional)**:

- [Crear o actualizar la base de datos (Set-AzSqlDatabaseAuditing) de la directiva de auditoría](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabaseauditing)
- [Crear o actualizar la directiva de auditoría de servidor (Set-AzSqlServerAuditing)](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlserverauditing)
- [Obtener directiva de auditoría de base de datos (Get-AzSqlDatabaseAuditing)](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabaseauditing)
- [Get Server Auditing Policy (Get-AzSqlServerAuditing)](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlserverauditing)

Para ver un script de ejemplo, consulte [Configuración de la auditoría y detección de amenazas mediante PowerShell](scripts/sql-database-auditing-and-threat-detection-powershell.md).

## <a id="subheading-9"></a>Administración de auditorías de SQL Database mediante API de REST

**API REST**:

- [Create or Update Database Auditing Policy](https://docs.microsoft.com/rest/api/sql/database%20auditing%20settings/createorupdate) (Creación o actualización de la directiva de auditoría de la base de datos)
- [Create or Update Server Auditing Policy](https://docs.microsoft.com/rest/api/sql/server%20auditing%20settings/createorupdate) (Creación o actualización de la directiva de auditoría del servidor)
- [Get Database Auditing Policy](https://docs.microsoft.com/rest/api/sql/database%20auditing%20settings/get) (Obtención de la directiva de auditoría de la base de datos)
- [Get Server Auditing Policy](https://docs.microsoft.com/rest/api/sql/server%20auditing%20settings/get) (Obtención de la directiva de auditoría del servidor)

Directiva extendida compatible con la cláusula WHERE para filtrado adicional:

- [Crear o actualizar la base de datos *extendidos* la directiva de auditoría](https://docs.microsoft.com/rest/api/sql/database%20extended%20auditing%20settings/createorupdate)
- [Crear o actualizar el servidor *extendidos* la directiva de auditoría](https://docs.microsoft.com/rest/api/sql/server%20auditing%20settings/createorupdate)
- [Obtener base de datos *extendidos* la directiva de auditoría](https://docs.microsoft.com/rest/api/sql/database%20extended%20auditing%20settings/get)
- [Obtener servidor *extendidos* la directiva de auditoría](https://docs.microsoft.com/rest/api/sql/server%20auditing%20settings/get)

## <a id="subheading-10"></a>Administración de auditorías de SQL Database mediante plantillas de Azure Resource Manager

Puede administrar auditorías de Azure SQL Database mediante plantillas de [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview), tal como se muestra en estos ejemplos:

- [Implementar un servidor de Azure SQL Server con la auditoría habilitada para escribir los registros de auditoría en la cuenta de Azure Blob Storage](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-blob-storage)
- [Implementación de un servidor de Azure SQL Server con la auditoría habilitada para escribir los registros de auditoría en Log Analytics](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-oms)
- [Implementación de un servidor de Azure SQL Server con la auditoría habilitada para escribir los registros de auditoría en Event Hubs](https://github.com/Azure/azure-quickstart-templates/tree/master/201-sql-auditing-server-policy-to-eventhub)

> [!NOTE]
> Los ejemplos vinculados están en un repositorio externo público y se proporcionan "como está", sin ninguna garantía y no se admiten en cualquier programa o servicio de soporte técnico de Microsoft.

<!--Anchors-->
[Azure SQL Database Auditing overview]: #subheading-1
[Set up auditing for your database]: #subheading-2
[Analyze audit logs and reports]: #subheading-3
[Practices for usage in production]: #subheading-5
[Storage Key Regeneration]: #subheading-6
[Manage SQL database auditing using Azure PowerShell]: #subheading-7
[Blob/Table differences in Server auditing policy inheritance]: (#subheading-8)
[Manage SQL database auditing using REST API]: #subheading-9
[Manage SQL database auditing using ARM templates]: #subheading-10

<!--Image references-->
[1]: ./media/sql-database-auditing-get-started/1_auditing_get_started_settings.png
[2]: ./media/sql-database-auditing-get-started/2_auditing_get_started_server_inherit.png
[3]: ./media/sql-database-auditing-get-started/3_auditing_get_started_turn_on.png
[4]: ./media/sql-database-auditing-get-started/4_auditing_get_started_storage_details.png
[5]: ./media/sql-database-auditing-get-started/5_auditing_get_started_storage_key_regeneration.png
[6]: ./media/sql-database-auditing-get-started/6_auditing_get_started_regenerate_key.png
[7]: ./media/sql-database-auditing-get-started/7_auditing_get_started_blob_view_audit_logs.png
[8]: ./media/sql-database-auditing-get-started/8_auditing_get_started_blob_audit_records.png
[9]: ./media/sql-database-auditing-get-started/9_auditing_get_started_ssms_1.png
[10]: ./media/sql-database-auditing-get-started/10_auditing_get_started_ssms_2.png