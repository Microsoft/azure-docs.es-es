---
title: Programación de trabajos U-SQL de Azure Data Lake Analytics mediante SSIS
description: Aprenda a usar SQL Server Integration Services para programar trabajos de U-SQL con script en línea o desde archivos de consulta de U-SQL.
services: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: how-to
ms.workload: big-data
ms.date: 07/17/2018
ms.openlocfilehash: ac747b87cf1a0f2d7c85d05975a31f953bfa5aae
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132507"
---
# <a name="schedule-u-sql-jobs-using-sql-server-integration-services-ssis"></a>Programación de trabajos U-SQL mediante SQL Server Integration Services (SSIS)

En este documento, aprenda a organizar y crear trabajos U-SQL mediante SQL Server Integration Service (SSIS). 

## <a name="prerequisites"></a>Requisitos previos

[Azure Feature Pack para Integration Services](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis?view=sql-server-2017#scenario-managing-data-in-the-cloud) proporciona la [tarea de Azure Data Lake Analytics](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017) y el [Administrador de conexiones de Azure Data Lake Analytics](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager?view=sql-server-2017) que ayuda a conectarse con el servicio de Azure Data Lake Analytics. Para usar esta tarea, asegúrese de instalar lo siguiente:

- [Descargar e instala SQL Server Data Tools (SSDT) para Visual Studio](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017)
- [Instalar Azure Feature Pack para Integration Services (SSIS)](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis?view=sql-server-2017)

## <a name="azure-data-lake-analytics-task"></a>Tarea de Azure Data Lake Analytics

La tarea de Azure Data Lake Analytics permite a los usuarios enviar trabajos U-SQL a la cuenta de Azure Data Lake Analytics. 

[Aprenda a configurar la tarea de Azure Data Lake Analytics](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017).

![Tarea de Azure Data Lake Analytics en SSIS](./media/data-lake-analytics-schedule-jobs-ssis/data-lake-analytics-azure-data-lake-analytics-task-in-ssis.png)

Puede obtener el script U-SQL desde distintos lugares mediante el uso de tareas y funciones integradas de SSIS. Los escenarios que aparecen a continuación muestran cómo se pueden configurar los scripts U-SQL para distintos casos de uso.

## <a name="scenario-1-use-inline-script-call-tvfs-and-stored-procs"></a>Escenario 1: Usar funciones TFV de llamada de script alineado y procedimientos almacenados

En el Editor de tareas de Azure Data Lake Analytics, configure **SourceType** como **DirectInput** y coloque las instrucciones U-SQL en **USQLStatement**.

Para simplificar la administración del código y el mantenimiento, solo coloque un script U-SQL breve como scripts alineados. Por ejemplo, puede llamar a funciones con valores de tabla existentes y procedimientos almacenados en las bases de datos U-SQL. 

![Edición de un script U-SQL alineado en la tarea de SSIS](./media/data-lake-analytics-schedule-jobs-ssis/edit-inline-usql-script-in-ssis.png)

Artículo relacionado: [How to pass parameter to stored procedures](#scenario-6-pass-parameters-to-u-sql-script) (Uso de parámetros en los procedimientos almacenados)

## <a name="scenario-2-use-u-sql-files-in-azure-data-lake-store"></a>Escenario 2: Usar archivos U-SQL en Azure Data Lake Store

También puede usar archivos U-SQL en Azure Data Lake Store con la **tarea del sistema de archivos de Azure Data Lake Store** en Azure Feature Pack. Este enfoque le permite usar los scripts almacenados en la nube.

Siga estos pasos para configurar la conexión entre la tarea del sistema de archivos de Azure Data Lake Store y la tarea de Azure Data Lake Analytics.

### <a name="set-task-control-flow"></a>Establecimiento del flujo de control de las tareas

En la vista de diseño del paquete SSIS, agregue una **tarea del sistema de archivos de Azure Data Lake Store**, un **contenedor de bucles Foreach** y una **tarea de Azure Data Lake Analytics** en el contenedor de bucles Foreach. La tarea del sistema de archivos de Azure Data Lake Store ayuda a descargar los archivos U-SQL de la cuenta de ADLS a una carpeta temporal. El contenedor de bucles Foreach y la tarea de Azure Data Lake Analytics ayudan a enviar cada archivo U-SQL de la carpeta temporal a la cuenta de Azure Data Lake Analytics como un trabajo U-SQL.

![Uso de archivos U-SQL en Azure Data Lake Store](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-data-lake-store.png)

### <a name="configure-azure-data-lake-store-file-system-task"></a>Configuración de la tarea del sistema de archivos de Azure Data Lake Store

1. Establezca **Operación** en **CopyFromADLS**.
2. Configure **AzureDataLakeConnection** y obtenga más información sobre el [Administrador de conexiones de Azure Data Lake Store](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-store-connection-manager?view=sql-server-2017).
3. Establezca **AzureDataLakeDirectory**. Indique la carpeta que almacena los scripts U-SQL. Use una ruta de acceso relativa que esté relacionada con la carpeta raíz de la cuenta de Azure Data Lake Store.
4. Establezca **Destino** en una carpeta que almacena en caché los scripts U-SQL descargados. Esta ruta de acceso a la carpeta se usará en el contenedor de bucles Foreach para enviar el trabajo U-SQL. 

![Configuración de la tarea del sistema de archivos de Azure Data Lake Store](./media/data-lake-analytics-schedule-jobs-ssis/configure-azure-data-lake-store-file-system-task.png)

[Obtenga más información sobre la tarea del sistema de archivos de Azure Data Lake Store](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-store-file-system-task?view=sql-server-2017).

### <a name="configure-foreach-loop-container"></a>Configuración del contenedor de bucles Foreach

1. En la página **Colección**, establezca **Enumerador** en **Foreach File Enumerator** (Enumerador de archivos Foreach).

2. Establezca **Carpeta** del grupo **Enumerator configuration** (Configuración del enumerador) en la carpeta temporal que incluye los scripts U-SQL que se descargaron.

3. Establezca **Archivos** de **Enumerator configuration** (Configuración del enumerador) en `*.usql` para que el contenedor de bucles solo capture los archivos que terminan en `.usql`.

    ![Configuración del contenedor de bucles Foreach](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-collection.png)

4. En la página **Variable Mappings** (Asignaciones de variables), agregue una variable definida por el usuario para obtener el nombre de archivo de cada archivo U-SQL. Establezca el **índice** en 0 para obtener el nombre de archivo. En este ejemplo, defina una variable denominada `User::FileName`. Esta variable se usará para obtener de manera dinámica la conexión del archivo de script U-SQL y establezca el nombre del trabajo U-SQL de la tarea de Azure Data Lake Analytics.

    ![Configuración del contenedor de bucles Foreach para obtener el nombre de archivo](./media/data-lake-analytics-schedule-jobs-ssis/configure-foreach-loop-container-variable-mapping.png)

### <a name="configure-azure-data-lake-analytics-task"></a>Configuración de la tarea de Azure Data Lake Analytics 

1. Establezca **SourceType** en **FileConnection**.

2. Establezca **FileConnection** en la conexión de archivos que apunta a los objetos de archivo devueltos desde el contenedor de bucles Foreach.
    
    Para crear esta conexión de archivos:

   1. Elija **\<New Connection...>** en la configuración FileConnection.
   2. Establezca **Usage type** (Tipo de uso) en **Existing file** (Archivo existente) y establezca **File** (Archivo) en la ruta de acceso de archivo de cualquier archivo existente.

       ![Configuración del contenedor de bucles Foreach](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

   3. En la vista **Administradores de conexiones**, haga clic con el botón derecho en la conexión de archivo que acaba de crear y elija **Propiedades**.

   4. En la ventana **Propiedades**, expanda **Expressions** (Expresiones) y establezca **ConnectionString** en la variable definida en el contenedor de bucles Foreach, por ejemplo, `@[User::FileName]`.

       ![Configuración del contenedor de bucles Foreach](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-property-for-foreach-loop-container.png)

3. Establezca **AzureDataLakeAnalyticsConnection** en la cuenta de Azure Data Lake Analytics a la que quiere enviar los trabajos. Obtenga más información sobre el [Administrador de conexiones de Azure Data Lake Analytics](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-data-lake-analytics-connection-manager?view=sql-server-2017).

4. Establezca las demás configuraciones del trabajo. [Más información](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017).

5. Use **Expressions** (Expresiones) para establecer el nombre del trabajo U-SQL de manera dinámica:

    1. En la página **Expressions** (Expresiones), agregue el par clave-valor de una expresión nueva para **JobName**.
    2. Establezca el valor de JobName en la variable definida en el contenedor de bucles Foreach, por ejemplo, `@[User::FileName]`.
    
        ![Configuración de la expresión de SSIS para el nombre del trabajo U-SQL](./media/data-lake-analytics-schedule-jobs-ssis/configure-expression-for-u-sql-job-name.png)

## <a name="scenario-3-use-u-sql-files-in-azure-blob-storage"></a>Escenario 3: Usar archivos U-SQL en Azure Blob Storage

Puede usar archivos U-SQL en Azure Blob Storage mediante la **tarea de descarga de Azure Blob** de Azure Feature Pack. Este enfoque le permite usar los scripts en la nube.

Los pasos son similares a los de [Escenario 2: Uso de archivos U-SQL en Azure Data Lake Store](#scenario-2-use-u-sql-files-in-azure-data-lake-store). Cambie la tarea del sistema de archivos de Azure Data Lake Store a la tarea de descarga de Azure Blob. [Obtenga más información sobre la tarea de descarga de Azure Blob](https://docs.microsoft.com/sql/integration-services/control-flow/azure-blob-download-task?view=sql-server-2017).

El flujo de control es el siguiente.

![Uso de archivos U-SQL en Azure Data Lake Store](./media/data-lake-analytics-schedule-jobs-ssis/use-u-sql-files-in-azure-blob-storage.png)

## <a name="scenario-4-use-u-sql-files-on-the-local-machine"></a>Escenario 4: Usar archivos U-SQL en la máquina local

Además de usar archivos U-SQL almacenados en la nube, también puede usar archivos de la máquina local o archivos implementados con los paquetes SSIS.

1. Haga clic con el botón derecho en **Administradores de conexiones** del proyecto SSIS y elija **New Connection Manager** (Nuevo administrador de conexiones).

2. Seleccione el tipo de **archivo** y haga clic en **Add...** (Agregar...).

3. Establezca **Usage type** (Tipo de uso) en **Existing file** (Archivo existente) y establezca **File** (Archivo) en el archivo de la máquina local.

    ![Incorporación de conexión de archivos al archivo local](./media/data-lake-analytics-schedule-jobs-ssis/configure-file-connection-for-foreach-loop-container.png)

4. Agregue la tarea **Azure Data Lake Analytics** y haga lo siguiente:
    1. Establezca **SourceType** en **FileConnection**.
    2. Establezca **FileConnection** en la conexión de archivos que acaba de crear.

5. Finalice las demás configuraciones de la tarea de Azure Data Lake Analytics.

## <a name="scenario-5-use-u-sql-statement-in-ssis-variable"></a>Escenario 5: Usar instrucciones U-SQL en una variable de SSIS

En algunos casos, puede que sea necesario generar las instrucciones U-SQL de manera dinámica. Puede usar **SSIS Variable** (Variable de SSIS) con **SSIS Expression** (Expresión SSIS) y otras tareas de SSIS, como la tarea Script, para ayudarlo a generar la instrucción U-SQL de manera dinámica.

1. Abra la ventana de herramientas Variables en el menú de nivel superior **SSIS > Variables**.

2. Agregue una variable de SSIS y establezca el valor directamente o use **Expression** (Expresión) para generar el valor.

3. Agregue la **tarea de Azure Data Lake Analytics** y haga lo siguiente:
    1. Establezca **SourceType** en **Variable**.
    2. Establezca **SourceVariable** en la variable de SSIS que acaba de crear.

4. Finalice las demás configuraciones de la tarea de Azure Data Lake Analytics.

## <a name="scenario-6-pass-parameters-to-u-sql-script"></a>Escenario 6: Pasar parámetros a un script U-SQL

En algunos casos, es posible que quiera establecer de manera dinámica el valor de la variable U-SQL del script U-SQL. La característica **Asignación de parámetros** de la tarea de Azure Data Lake Analytics resulta útil en este escenario. Habitualmente hay dos casos de uso típicos:

- Establezca las variables de ruta de acceso de archivo de entrada y de salida de manera dinámica en función de la fecha y hora actual.
- Establezca el parámetro para los procedimientos almacenados.

[Obtenga más información sobre cómo establecer los parámetros del script U-SQL](https://docs.microsoft.com/sql/integration-services/control-flow/azure-data-lake-analytics-task?view=sql-server-2017#parameter-mapping-page-configuration).

## <a name="next-steps"></a>Pasos siguientes

- [Ejecución de paquetes SSIS en Azure](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity)
- [Azure Feature Pack para Integration Services (SSIS)](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis?view=sql-server-2017#scenario-managing-data-in-the-cloud)
- [Programación de trabajos U-SQL con Azure Data Factory](https://docs.microsoft.com/azure/data-factory/transform-data-using-data-lake-analytics)
