---
title: Ejecución de paquetes SSIS mediante el agente de Instancia administrada de Azure SQL
description: Aprenda a ejecutar paquetes SSIS mediante el agente de Instancia administrada de Azure SQL.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: lle
author: lle
ms.date: 04/14/2020
ms.openlocfilehash: cf1bf9e05f83610fd43146cf4c99c5006fdc97b3
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171440"
---
# <a name="run-ssis-packages-by-using-azure-sql-managed-instance-agent"></a>Ejecución de paquetes SSIS con el agente de Instancia administrada de Azure SQL

En este artículo se describe cómo ejecutar un paquete de SQL Server Integration Services (SSIS) mediante el agente de Instancia administrada de Azure SQL. Esta característica proporciona comportamientos similares a cuando se programan paquetes SSIS mediante el Agente SQL Server en el entorno local.

Con esta característica, puede ejecutar paquetes SSIS que se almacenan en SSISDB en una instancia de SQL Managed Instance; un sistema de archivos, como Azure Files; o un almacén de paquetes de entorno de ejecución de integración de Azure-SSIS.

## <a name="prerequisites"></a>Requisitos previos

Para usar esta característica, [descargue](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017) e instale la versión más reciente de SQL Server Management Studio (SSMS). A continuación encontrará los detalles de la compatibilidad de versiones:

- Para ejecutar paquetes en SSISDB o en el sistema de archivos, instale la versión 18.5 o posterior de SSMS.
- Para ejecutar paquetes en el almacén de paquetes, instale la versión 18.6 o posterior de SSMS.

También tiene que [aprovisionar una instancia de Integration Runtime para la integración de SSIS en Azure](tutorial-create-azure-ssis-runtime-portal.md) en Azure Data Factory. Usa una instancia administrada de Azure SQL como servidor de punto de conexión.

## <a name="run-an-ssis-package-in-ssisdb"></a>Ejecución de un paquete SSIS en SSISDB

En este procedimiento, se usa el agente de Instancia administrada de SQL para invocar un paquete SSIS que está almacenado en SSISDB.

1. En la versión más reciente de SSMS, conéctese a la Instancia administrada de SQL.
1. Cree un trabajo del agente nuevo y un paso de trabajo nuevo. En **Agente SQL Server**, haga clic con el botón derecho en la carpeta **Trabajos** y seleccione **Nuevo trabajo**.

   ![Selecciones para crear un nuevo trabajo del agente](./media/how-to-invoke-ssis-package-managed-instance-agent/new-agent-job.png)

1. En la página **Nuevo paso de trabajo**, seleccione el tipo **Paquete de SQL Server Integration Services**.

   ![Selecciones para crear un nuevo paso de trabajo de SSIS](./media/how-to-invoke-ssis-package-managed-instance-agent/new-ssis-job-step.png)

1. En la pestaña **Paquete**, seleccione **Catálogo de SSIS** como la ubicación del paquete.
1. Como SSISDB está en una instancia administrada de SQL, no necesita especificar la autenticación.
1. Especifique un paquete SSIS del SSISDB.

   ![Pestaña Paquete con selecciones para el tipo de origen del paquete](./media/how-to-invoke-ssis-package-managed-instance-agent/package-source-ssisdb.png)

1. En la pestaña **Configuración**, puede:
  
   - Especifique los valores de parámetro en **Parámetros**.
   - Invalidar los valores en **Administradores de conexión**.
   - Invalidar la propiedad y elegir el nivel de registro en **Avanzado**.

   ![Pestaña Configuración con selecciones para el tipo de origen del paquete](./media/how-to-invoke-ssis-package-managed-instance-agent/package-source-ssisdb-configuration.png)

1. Seleccione **Aceptar** para guardar la configuración del trabajo del agente.
1. Inicie el trabajo del Agente para ejecutar el paquete SSIS.

## <a name="run-an-ssis-package-in-the-file-system"></a>Ejecución de un paquete SSIS en el sistema de archivos

En este procedimiento, se usa el agente de Instancia administrada de SQL para ejecutar un paquete SSIS que está almacenado en el sistema de archivos.

1. En la versión más reciente de SSMS, conéctese a la Instancia administrada de SQL.
1. Cree un trabajo del agente nuevo y un paso de trabajo nuevo. En **Agente SQL Server**, haga clic con el botón derecho en la carpeta **Trabajos** y seleccione **Nuevo trabajo**.

   ![Selecciones para crear un nuevo trabajo del agente](./media/how-to-invoke-ssis-package-managed-instance-agent/new-agent-job.png)

1. En la página **Nuevo paso de trabajo**, seleccione el tipo **Paquete de SQL Server Integration Services**.

   ![Selecciones para crear un nuevo paso de trabajo de SSIS](./media/how-to-invoke-ssis-package-managed-instance-agent/new-ssis-job-step.png)

1. En la pestaña **Paquete**:

   1. En **Ubicación del paquete**, seleccione **Sistema de archivos**.

   1. En **Tipo de origen del archivo**:

      - Si el paquete se carga en Azure Files, seleccione **Recurso compartido de archivos de Azure**.

        ![Opciones del tipo de origen del archivo](./media/how-to-invoke-ssis-package-managed-instance-agent/package-source-file-system.png)

        La ruta de acceso del paquete es **`\\<storage account name>.file.core.windows.net\<file share name>\<package name>.dtsx`** .

        En **Credencial de acceso al archivo del paquete**, escriba el nombre de la cuenta del archivo de Azure y la clave de cuenta para acceder al archivo de Azure. El dominio se establece como **Azure**.

      - Si el paquete se carga en un recurso compartido de red, seleccione **Recurso compartido de red**.

        La ruta de acceso al paquete es la ruta de acceso UNC del archivo del paquete con su extensión .dtsx.

        Escriba el dominio, el nombre de usuario y la contraseña correspondientes para acceder al archivo de paquete del recurso compartido de red.
   1. Si el archivo de paquete está cifrado con una contraseña, seleccione **Contraseña de cifrado** y escriba la contraseña.
1. En la pestaña **Configuraciones**, escriba la ruta de acceso al archivo de configuración si necesita un archivo de configuración para ejecutar el paquete SSIS.
   Si almacena la configuración en Azure Files, su ruta de acceso de configuración será **`\\<storage account name>.file.core.windows.net\<file share name>\<configuration name>.dtsConfig`** .
1. En la pestaña **Opciones de ejecución**, puede elegir si usar la **autenticación de Windows** o **tiempo de ejecución de 32 bits** para ejecutar el paquete SSIS.
1. En la pestaña **Registro**, puede elegir la ruta de acceso al registro y la credencial de acceso al registro correspondiente para almacenar los archivos de registro. 
   De forma predeterminada, la ruta de acceso al registro es la misma que la ruta de acceso a la carpeta del paquete y la credencial de acceso al registro es igual que la credencial de acceso al paquete.
   Si almacena los registros en Azure Files, su ruta de acceso de registro será **`\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`** .
1. En la pestaña **Establecer valores**, puede escribir la ruta de acceso a la propiedad y el valor para invalidar las propiedades del paquete.

   Por ejemplo, para invalidar el valor de la variable de usuario, escriba su ruta de acceso en el formato **`\Package.Variables[User::<variable name>].Value`** .
1. Seleccione **Aceptar** para guardar la configuración del trabajo del agente.
1. Inicie el trabajo del Agente para ejecutar el paquete SSIS.

## <a name="run-an-ssis-package-in-the-package-store"></a>Ejecución de un paquete SSIS en el almacén de paquetes

En este procedimiento, se usa el agente de SQL Managed Instance para ejecutar un paquete de SSIS que está almacenado en el almacén de paquetes de Azure-SSIS IR.

1. En la versión más reciente de SSMS, conéctese a la Instancia administrada de SQL.
1. Cree un trabajo del agente nuevo y un paso de trabajo nuevo. En **Agente SQL Server**, haga clic con el botón derecho en la carpeta **Trabajos** y seleccione **Nuevo trabajo**.

   ![Selecciones para crear un nuevo trabajo del agente](./media/how-to-invoke-ssis-package-managed-instance-agent/new-agent-job.png)

1. En la página **Nuevo paso de trabajo**, seleccione el tipo **Paquete de SQL Server Integration Services**.

   ![Selecciones para crear un nuevo paso de trabajo de SSIS](./media/how-to-invoke-ssis-package-managed-instance-agent/new-ssis-job-step.png)

1. En la pestaña **Paquete**:

   1. En **Ubicación del paquete**, seleccione **Almacén de paquetes**.

   1. En **Ruta de acceso del paquete**:

      La ruta de acceso del paquete es **`<package store name>\<folder name>\<package name>`** .

      ![Opciones del tipo de almacén de paquetes](./media/how-to-invoke-ssis-package-managed-instance-agent/package-source-package-store.png)

   1. Si el archivo de paquete está cifrado con una contraseña, seleccione **Contraseña de cifrado** y escriba la contraseña.
1. En la pestaña **Configuraciones**, escriba la ruta de acceso al archivo de configuración si necesita un archivo de configuración para ejecutar el paquete SSIS.
   Si almacena la configuración en Azure Files, su ruta de acceso de configuración será **`\\<storage account name>.file.core.windows.net\<file share name>\<configuration name>.dtsConfig`** .
1. En la pestaña **Opciones de ejecución**, puede elegir si usar la **autenticación de Windows** o **tiempo de ejecución de 32 bits** para ejecutar el paquete SSIS.
1. En la pestaña **Registro**, puede elegir la ruta de acceso al registro y la credencial de acceso al registro correspondiente para almacenar los archivos de registro.
   De forma predeterminada, la ruta de acceso al registro es la misma que la ruta de acceso a la carpeta del paquete y la credencial de acceso al registro es igual que la credencial de acceso al paquete.
   Si almacena los registros en Azure Files, su ruta de acceso de registro será **`\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`** .
1. En la pestaña **Establecer valores**, puede escribir la ruta de acceso a la propiedad y el valor para invalidar las propiedades del paquete.

   Por ejemplo, para invalidar el valor de la variable de usuario, escriba su ruta de acceso en el formato **`\Package.Variables[User::<variable name>].Value`** .
1. Seleccione **Aceptar** para guardar la configuración del trabajo del agente.
1. Inicie el trabajo del Agente para ejecutar el paquete SSIS.

## <a name="cancel-ssis-package-execution"></a>Cancelación de la ejecución de paquetes SSIS

Para cancelar la ejecución del paquete desde un trabajo del agente de Instancia administrada de SQL, siga estos pasos en lugar de detener directamente el trabajo del agente:

1. Busque el valor de **jobId** del Agente de SQL en **msdb.dbo.sysjobs**.
1. Busque el valor de **executionId** de SSIS correspondiente en función del identificador del trabajo con esta consulta:
   ```sql
   select * from '{table for job execution}' where  parameter_value = 'SQL_Agent_Job_{jobId}' order by execution_id desc
   ```
   Si los paquetes SSIS están en SSISDB, use **ssisdb.internal.execution_parameter_values** como tabla para la ejecución del trabajo. Si los paquetes SSIS están en el sistema de archivos, use **ssisdb.internal.execution_parameter_values_noncatalog**.
1. Haga clic con el botón derecho en el catálogo de SSISDB y, a continuación, seleccione **Operaciones activas**.

   !["Operaciones activas" en el menú contextual del catálogo de SSISDB](./media/how-to-invoke-ssis-package-managed-instance-agent/catalog-active-operations.png)

1. Detenga la operación correspondiente según **executionId**.

## <a name="next-steps"></a>Pasos siguientes
También puede programar los paquetes SSIS con Azure Data Factory. Para instrucciones paso a paso, consulte [Desencadenador de eventos de Azure Data Factory](how-to-create-event-trigger.md). 
