---
title: Prueba de código de Azure Data Lake Analytics
description: Aprenda a agregar casos de prueba para código de U-SQL y C# extendido para Azure Data Lake Analytics.
services: data-lake-analytics
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.assetid: 66dd58b1-0b28-46d1-aaae-43ee2739ae0a
ms.service: data-lake-analytics
ms.topic: how-to
ms.workload: big-data
ms.date: 08/30/2019
ms.openlocfilehash: 44426598daf1808ef0aee233968b04d2dc7c165f
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87129923"
---
# <a name="test-your-azure-data-lake-analytics-code"></a>Prueba del código de Azure Data Lake Analytics

Azure Data Lake proporciona el lenguaje [U-SQL](data-lake-analytics-u-sql-get-started.md). U-SQL combina SQL declarativo con C# imperativo para procesar datos a cualquier escala. En este documento, aprenderá a crear casos de prueba para código de U-SQL y C# extendido de operador definido por el usuario (UDO).

## <a name="test-u-sql-scripts"></a>Prueba de scripts U-SQL

El script U-SQL se compila y se optimiza para que el código ejecutable pueda ejecutarse en Azure o en el equipo local. El proceso de compilación y optimización trata el script de U-SQL completo como un todo. No puede realizar la tradicional prueba unitaria para cada instrucción. Sin embargo, mediante el SDK de prueba y el SDK de ejecución local de U-SQL, puede realizar pruebas de nivel de script.

### <a name="create-test-cases-for-u-sql-script"></a>Creación de casos de prueba para el script U-SQL

Las Herramientas de Azure Data Lake para Visual Studio le permiten crear casos de prueba para el script U-SQL.

1. Haga clic con el botón derecho en un script U-SQL en el Explorador de soluciones y después seleccione **Create Unit Test** (Crear prueba unitaria).

1. Cree un proyecto de prueba o inserte el caso de prueba en un proyecto de prueba existente.

   ![Herramientas de Data Lake para Visual Studio: Creación de una configuración de proyecto de prueba de U-SQL](./media/data-lake-analytics-cicd-test/data-lake-tools-create-usql-test-project-configure.png)

### <a name="manage-the-test-data-source"></a>Administración del origen de datos de prueba

Al probar scripts U-SQL, necesita los archivos de entrada de prueba. Para administrar los datos de prueba, en el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto U-SQL y seleccione **Propiedades**. Puede especificar un origen en **Origen de datos de prueba**.

![Herramientas de Data Lake para Visual Studio: Configuración del origen de datos de prueba del proyecto](./media/data-lake-analytics-cicd-test/data-lake-tools-configure-project-test-data-source.png)

Cuando llama a la interfaz `Initialize()` en el SDK de prueba de U-SQL, se crea una carpeta raíz de datos local temporal en el directorio de trabajo del proyecto de prueba. Todos los archivos y carpetas de la carpeta Origen de datos de prueba se copian en la carpeta raíz de datos local temporal antes de que ejecute los casos de prueba del script U-SQL. Puede agregar más carpetas de origen de datos de prueba si divide la ruta de la carpeta de datos de prueba con un punto y coma.

### <a name="manage-the-database-environment-for-testing"></a>Administración del entorno de base de datos para la prueba

Si, los scripts U-SQL usan objetos de base de datos U-SQL o realizan consultas con estos, debe inicializar el entorno de base de datos antes de ejecutar los casos de prueba de U-SQL. Este enfoque puede ser necesario cuando se llama a procedimientos almacenados. La interfaz `Initialize()` en el SDK de prueba de U-SQL le ayudará a implementar todas las bases de datos a las que hace referencia el proyecto de U-SQL en la carpeta raíz de datos local temporal del directorio de trabajo del proyecto de prueba.

Para obtener más información sobre cómo administrar las referencias del proyecto de base de datos U-SQL para un proyecto U-SQL, consulte [Referencia de un proyecto de base de datos U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md#reference-a-u-sql-database-project).

### <a name="verify-test-results"></a>Comprobación de los resultados de la prueba

La interfaz `Run()` devuelve el resultado de la ejecución de un trabajo. *0* significa correcto y *1* significa error. También puede usar funciones de aserción de C# para comprobar los resultados.

### <a name="run-test-cases-in-visual-studio"></a>Ejecución de casos de prueba en Visual Studio

Un proyecto de prueba del script de U-SQL se compila a partir de la plataforma de una prueba unitaria de C#. Después de compilar el proyecto, seleccione **Probar** > **Windows** > **TestExplorer**. Puede ejecutar casos de prueba desde el **Explorador de pruebas**. Como alternativa, haga clic con el botón derecho en el archivo .cs de la prueba unitaria y seleccione **Ejecutar pruebas**.

## <a name="test-c-udos"></a>Prueba de los UDO de C#

### <a name="create-test-cases-for-c-udos"></a>Creación de casos de prueba para UDO de C#

Puede usar la plataforma de una prueba unitaria de C# para probar sus operadores definidos por el usuario (UDO) de C#. Al probar los UDO, debe preparar los objetos **IRowset** correspondientes como entradas.

Hay dos maneras de crear un objeto **IRowset**:

- Cargar datos desde un archivo para crear el objeto **IRowset**:

    ```csharp
    //Schema: "a:int, b:int"
    USqlColumn<int> col1 = new USqlColumn<int>("a");
    USqlColumn<int> col2 = new USqlColumn<int>("b");
    List<IColumn> columns = new List<IColumn> { col1, col2 };
    USqlSchema schema = new USqlSchema(columns);

    //Generate one row with default values
    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Get data from file
    IRowset rowset = UnitTestHelper.GetRowsetFromFile(@"processor.txt", schema, output.AsReadOnly(), discardAdditionalColumns: true, rowDelimiter: null, columnSeparator: '\t');
    ```

- Usar datos de una colección de datos para crear el objeto **IRowset**:

    ```csharp
    //Schema: "a:int, b:int"
    USqlSchema schema = new USqlSchema(
        new USqlColumn<int>("a"),
        new USqlColumn<int>("b")
    );

    IUpdatableRow output = new USqlRow(schema, null).AsUpdatable();

    //Generate Rowset with specified values
    List<object[]> values = new List<object[]>{
        new object[2] { 2, 3 },
        new object[2] { 10, 20 }
    };

    IEnumerable<IRow> rows = UnitTestHelper.CreateRowsFromValues(schema, values);
    IRowset rowset = UnitTestHelper.GetRowsetFromCollection(rows, output.AsReadOnly());
    ```

### <a name="verify-test-results"></a>Comprobación de los resultados de la prueba

Después de llamar a las funciones de UDO, puede comprobar los valores schema y Rowset con funciones de aserción de C# para comprobar el resultado. Puede agregar un **proyecto de prueba unitaria UDO de C# de U-SQL** a la solución. Para ello, seleccione **Archivo -> Nuevo > Proyecto** en Visual Studio.

### <a name="run-test-cases-in-visual-studio"></a>Ejecución de casos de prueba en Visual Studio

Después de compilar el proyecto, seleccione **Probar** > **Windows** > **TestExplorer**. Puede ejecutar casos de prueba desde el **Explorador de pruebas**. Como alternativa, haga clic con el botón derecho en el archivo .cs de la prueba unitaria y seleccione **Ejecutar pruebas**.

## <a name="run-test-cases-in-azure-pipelines"></a>Ejecución de casos de prueba en Azure Pipelines<a name="run-test-cases-in-azure-devops"></a>

Tanto los **proyectos de prueba para el script de U-SQL** como los **proyectos de prueba de UDO de C#** heredan los proyectos de prueba unitaria de C#. La [tarea de prueba de Visual Studio](https://docs.microsoft.com/azure/devops/pipelines/test/getting-started-with-continuous-testing?view=vsts) en Azure Pipelines puede ejecutar estos casos de prueba.

### <a name="run-u-sql-test-cases-in-azure-pipelines"></a>Ejecución de casos de prueba de U-SQL en Azure Pipelines

En el caso de una prueba de U-SQL, asegúrese de cargar `CPPSDK` en el equipo de compilación y, a continuación, pase la ruta de acceso `CPPSDK` a `USqlScriptTestRunner(cppSdkFolderFullPath: @"")`.

#### <a name="what-is-cppsdk"></a>¿Qué es CPPSDK?

CPPSDK es un paquete que incluye Microsoft Visual C++ 14 y Windows SDK 10.0.10240.0. Este paquete incluye el entorno que necesita el entorno de ejecución de U-SQL. Puede obtener este paquete en la carpeta de instalación de las Herramientas de Azure Data Lake para Visual Studio:

- Para Visual Studio 2015, se encuentra en `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`
- Para Visual Studio 2017, se encuentra en `C:\Program Files (x86)\Microsoft Visual Studio\2017\<Visual Studio Edition>\SDK\ScopeCppSDK`
- Para Visual Studio 2019, se encuentra en `C:\Program Files (x86)\Microsoft Visual Studio\2019\<Visual Studio Edition>\SDK\ScopeCppSDK`.

#### <a name="prepare-cppsdk-in-the-azure-pipelines-build-agent"></a>Preparación de CPPSDK en el agente de compilación de Azure Pipelines

La forma más habitual de preparar la dependencia de CPPSDK en Azure Pipelines es la siguiente:

1. Comprima la carpeta que incluye las bibliotecas de CPPSDK.

1. Compruebe el archivo .zip en el sistema de control de código fuente. (El archivo ZIP garantiza que inserta en el repositorio todas las bibliotecas de la carpeta CPPSDK, de modo que algunos archivos no se ignoren debido a un archivo `.gitignore`).

1. Descomprima el archivo .zip en la canalización de compilación.

1. Dirija `USqlScriptTestRunner` a esta carpeta descomprimida en el equipo de compilación.

### <a name="run-c-udo-test-cases-in-azure-pipelines"></a>Ejecución de casos de prueba de UDO de C# en Azure Pipelines

Para una prueba de UDO de C#, asegúrese de hacer referencia a los ensamblados siguientes, que son necesarios para los UDO.

- Microsoft.Analytics.Interfaces
- Microsoft.Analytics.Types
- Microsoft.Analytics.UnitTest

Si hace referencia a ellos mediante [el paquete de Nuget Microsoft.Azure.DataLake.USQL.Interfaces](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.Interfaces/), asegúrese de agregar una tarea de restauración de NuGet en la canalización de compilación.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de canalizaciones de CI/CD para Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Ejecución local de scripts U-SQL](data-lake-analytics-data-lake-tools-local-run.md)
- [Uso de un proyecto de base de datos de U-SQL para desarrollar bases de datos de U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md)
