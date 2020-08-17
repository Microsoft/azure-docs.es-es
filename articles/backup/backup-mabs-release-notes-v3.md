---
title: Notas de la versión de Microsoft Azure Backup Server v3
description: En este artículo se proporciona la información sobre los problemas conocidos y las soluciones alternativas para Microsoft Azure Backup Server (MABS) v3.
ms.topic: conceptual
ms.date: 06/03/2020
ms.asset: 0c4127f2-d936-48ef-b430-a9198e425d81
ms.openlocfilehash: 1f4900bb129ee67cd75d2b793f4179e3135569a6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87032568"
---
# <a name="release-notes-for-microsoft-azure-backup-server"></a>Notas de la versión de Microsoft Azure Backup Server

Este artículo describe los problemas conocidos y las soluciones alternativas para Microsoft Azure Backup Server (MABS) v3.

## <a name="backup-and-recovery-fails-for-clustered-workloads"></a>Errores de copia de seguridad y recuperación para cargas de trabajo agrupadas

**Descripción:** Errores de copia de seguridad y restauración de orígenes de datos agrupados como un clúster de Hyper-V, un clúster de SQL (SQL AlwaysOn) o Exchange en un grupo de disponibilidad de base de datos (DAG) después de actualizar MABS v2 a MABS v3.

**Solución alternativa:** Para evitar estos errores, abra SQL Server Management Studio (SSMS) y ejecute el siguiente script SQL en la BD de DPM:

```sql
    IF EXISTS (SELECT * FROM dbo.sysobjects
        WHERE id = OBJECT_ID(N'[dbo].[tbl_PRM_DatasourceLastActiveServerMap]')
        AND OBJECTPROPERTY(id, N'IsUserTable') = 1)
        DROP TABLE [dbo].[tbl_PRM_DatasourceLastActiveServerMap]
        GO

        CREATE TABLE [dbo].[tbl_PRM_DatasourceLastActiveServerMap] (
            [DatasourceId]          [GUID]          NOT NULL,
            [ActiveNode]            [nvarchar](256) NULL,
            [IsGCed]                [bit]           NOT NULL
            ) ON [PRIMARY]
        GO

        ALTER TABLE [dbo].[tbl_PRM_DatasourceLastActiveServerMap] ADD
    CONSTRAINT [pk__tbl_PRM_DatasourceLastActiveServerMap__DatasourceId] PRIMARY KEY NONCLUSTERED
        (
            [DatasourceId]
        )  ON [PRIMARY],

    CONSTRAINT [DF_tbl_PRM_DatasourceLastActiveServerMap_IsGCed] DEFAULT
        (
            0
        ) FOR [IsGCed]
    GO
```

## <a name="upgrade-to-mabs-v3-fails-in-russian-locale"></a>Error en la actualización a MABS v3 en la configuración regional en ruso

**Descripción:** Error al actualizar de MABS v2 a MABS v3 en la configuración regional en ruso con el código **4387**.

**Solución alternativa:** Realice los pasos siguientes para actualizar a MABS v3 con el paquete de instalación de ruso:

1. [Realice una copia de seguridad](/sql/relational-databases/backup-restore/create-a-full-database-backup-sql-server#SSMSProcedure) de su base de datos SQL y desinstale MABS v2 (seleccione conservar los datos protegidos durante la desinstalación).
2. Actualice a SQL 2017 (Enterprise) y desinstale Reporting como parte de la actualización.
3. [Instale](/sql/reporting-services/install-windows/install-reporting-services#install-your-report-server) SQL Server Reporting Services (SSRS).
4. [Instale](/sql/ssms/download-sql-server-management-studio-ssms) SQL Server Management Studio (SSMS).
5. Configure Reporting mediante los parámetros tal como se describe en [Configuración de SSRS con SQL 2017](./backup-azure-microsoft-azure-backup.md#upgrade-mabs).
6. [Instale](backup-azure-microsoft-azure-backup.md) MABS v3.
7. [Restaure](/sql/relational-databases/backup-restore/restore-a-database-backup-using-ssms) SQL con SSMS y ejecute la herramienta DPM-Sync tal como se describe [aquí](/system-center/dpm/back-up-the-dpm-server?view=sc-dpm-2019#using-dpmsync).
8. Actualice la propiedad “DataBaseVersion” en la tabla dbo.tbl_DLS_GlobalSetting con el comando siguiente:

    ```sql
            UPDATE dbo.tbl_DLS_GlobalSetting
            set PropertyValue = '13.0.415.0'
            where PropertyName = 'DatabaseVersion'
    ```

9. Inicie el servicio MSDPM.

## <a name="after-installing-ur1-the-mabs-reports-arent-updated-with-new-rdl-files"></a>Después de instalar UR1, los informes de MABS no se actualizan con los nuevos archivos RDL

**Descripción**: Con UR1, el problema de formato del informe de MABS se corrige con los archivos RDL actualizados. Los nuevos archivos RDL no se reemplazan automáticamente con los archivos existentes.

**Solución alternativa**: Para reemplazar los archivos RDL, siga estos pasos:

1. En el equipo de MABS, abra la dirección URL del portal web de SQL Reporting Services.
1. En la dirección URL del portal web, la carpeta DPMReports tiene el formato **`DPMReports_<GUID>`**

    >[!NOTE]
    >Siempre hay solamente una carpeta con esta convención de nomenclatura. Si MABS se actualiza a partir de una versión anterior, es posible que también haya otra carpeta anterior, pero no podrá abrirla.

    ![Carpeta DPMReports](./media/backup-mabs-release-notes-v3/dpm-reports-folder.png)

1. Seleccione y abra la carpeta **`DPMReports_<GUID>`** . Los archivos de informe individuales se enumerarán como se muestra a continuación.

    ![Lista de archivos de informe individuales](./media/backup-mabs-release-notes-v3/individual-report-files.png)

1. Seleccione los archivos de informe que no terminan con **Report**, haga clic con el botón derecho en **Opción** y seleccione **Administrar**.

    ![Selección de Administrar para archivos de informe](./media/backup-mabs-release-notes-v3/manage-files.png)

1. En la página nuevo, seleccione la opción **Reemplazar** para reemplazar los archivos por los archivos de informe más recientes.

    Los archivos de informe más recientes pueden encontrarse en la ruta de acceso `<MABS Installation Directory>\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\DpmReports`.

    Por ejemplo: `C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\DpmReports`

    ![Reemplazo de los archivos por los archivos de informe más recientes](./media/backup-mabs-release-notes-v3/replace-files.png)

    Una vez reemplazados los archivos, asegúrese de que los campos **Nombre** y **Descripción** están intactos y no están vacíos.

1. Una vez reemplazados los archivos, reinicie los servicios de MABS y use los archivos de informe.

## <a name="next-steps"></a>Pasos siguientes

[Novedades de MABS](backup-mabs-whats-new-mabs.md)
