---
title: Depuración de código en C# para trabajos de U-SQL en Azure Data Lake
description: En este artículo se describe cómo depurar vértices U-SQL con error mediante Herramientas de Azure Data Lake para Visual Studio.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: bcd0b01e-1755-4112-8e8a-a5cabdca4df2
ms.topic: how-to
ms.date: 11/30/2017
ms.openlocfilehash: ca6fe4ad35e59472e8cf8f3b8476417e01c2668f
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131878"
---
# <a name="debug-user-defined-c-code-for-failed-u-sql-jobs"></a>Depuración de código C# definido por el usuario para trabajos de U-SQL con errores

U-SQL proporciona un modelo de extensibilidad mediante C#. En los scripts de U-SQL es fácil llamar a funciones de C# y realizar funciones de análisis que no admitiría un lenguaje declarativo similar a SQL. Para más información sobre la extensibilidad de U-SQL, consulte la [Guía de programación de U-SQL](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide#use-user-defined-functions-udf). 

En la práctica, cualquier código puede necesitar depuración, pero es difícil depurar un trabajo distribuido con código personalizado en la nube con unos archivos de registro limitados. Las [Herramientas de Azure Data Lake para Visual Studio](https://aka.ms/adltoolsvs) incluyen una característica denominada **depuración de vértice con errores**, que ayuda a que sea más fácil depurar los errores que se producen en el código personalizado. Cuando se produce un error en el trabajo de U-SQL, el servicio conserva el estado de error y la herramienta le ayuda a descargar el entorno con error de la nube en la máquina local para su depuración. La descarga local captura todo el entorno en la nube, incluidos el código de usuario y los datos de entrada.

En el siguiente vídeo se demuestra la depuración de vértice con errores en las Herramientas de Azure Data Lake para Visual Studio.

> [!VIDEO https://www.youtube.com/embed/3enkNvprfm4]
>

> [!IMPORTANT]
> Visual Studio requiere las siguientes dos actualizaciones para poder usar esta característica: [Microsoft Visual C++ 2015 Redistributable Update 3](https://www.microsoft.com/en-us/download/details.aspx?id=53840) y [Universal C Runtime en Windows](https://www.microsoft.com/download/details.aspx?id=50410).
>

## <a name="download-failed-vertex-to-local-machine"></a>Error al descargar el vértice en la máquina local

Al abrir un trabajo erróneo en Herramientas de Azure Data Lake para Visual Studio, verá una barra de alerta amarilla con mensajes de error detallados en la pestaña Error.

1. Haga clic en **Descargar** para descargar todos los flujos de entrada y recursos necesarios. Si la descarga no se completa, haga clic en **Reintentar**.

2. Haga clic en **Abrir** después de que finalice la descarga para generar el entorno de depuración local. Se abrirá una nueva solución de depuración y, si ya tiene una solución abierta en Visual Studio, asegúrese de guardarla y cerrarla antes de depurar.

![Descarga de vértices para depuración de trabajos U-SQL de Azure Data Lake Analytics en Visual Studio](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-download-vertex.png)

## <a name="configure-the-debugging-environment"></a>Configuración del entorno de depuración

> [!NOTE]
> Antes de realizar la depuración, asegúrese de que ha activado **Common Language Runtime Exceptions** (Excepciones de Common Language Runtime) en la ventana de configuración de excepciones (**Ctrl + Alt + E**).

![Configuración de depuración de trabajos U-SQL de Azure Data Lake Analytics en Visual Studio](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-clr-exception-setting.png)

En la nueva instancia de Visual Studio iniciada puede que encuentre, o no, el código fuente en C# definido por el usuario:

1. [Puedo encontrar el código fuente en la solución](#source-code-is-included-in-debugging-solution)

2. [No puedo encontrar el código fuente en la solución](#source-code-is-not-included-in-debugging-solution)

### <a name="source-code-is-included-in-debugging-solution"></a>El código fuente está incluido en la solución de depuración

Hay dos casos en los que se captura el código fuente en C#:

1. El código de usuario se define en el archivo de código subyacente (normalmente denominado `Script.usql.cs` en un proyecto de U-SQL).

2. El código de usuario se define en el proyecto de la biblioteca de clases C# de la aplicación de U-SQL y se registra como ensamblado con **información de depuración**.

Si el código fuente se importa en la solución, puede usar las herramientas de depuración de Visual Studio (visualización, variables, etc.) para solucionar el problema:

1. Pulse **F5** para iniciar la depuración. El código se ejecutará hasta que una excepción lo detenga.

2. Abra el archivo de código fuente y establezca los puntos de interrupción. A continuación, pulse **F5** para depurar el código paso por paso.

    ![Excepción de depuración de U-SQL de Azure Data Lake Analytics](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-debug-exception.png)

### <a name="source-code-is-not-included-in-debugging-solution"></a>El código fuente no está incluido en la solución de depuración

Si el código de usuario no se incluye en el archivo de código subyacente, o no se registró el ensamblado con **información de depuración**, el código fuente no se incluirá automáticamente en la solución de depuración. En este caso, deberá realizar pasos adicionales para agregar el código fuente:

1. Haga clic con el botón derecho en **Solution 'VertexDebug' > Add > Existing Project...** (Solución 'VertexDebug' > Agregar > proyecto existente) para buscar el código fuente del ensamblado y agregar el proyecto a la solución de depuración.

    ![Depuración de U-SQL de Azure Data Lake Analytics: agregar proyecto](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-add-project-to-debug-solution.png)

2. Obtenga la ruta de la carpeta del proyecto **FailedVertexDebugHost**. 

3. Haga clic con el botón derecho en el **proyecto de código fuente del ensamblado > Properties** (Propiedades), seleccione la pestaña **Build** (Compilar) a la izquierda y pegue la ruta de acceso que termina en \bin\debug en **Output > Output path** (Salida > Ruta de salida). La ruta de acceso de salida final es como `<DataLakeTemp path>\fd91dd21-776e-4729-a78b-81ad85a4fba6\loiu0t1y.mfo\FailedVertexDebug\FailedVertexDebugHost\bin\Debug\`.

    ![Depuración de U-SQL de Azure Data Lake Analytics: definición de la ruta de acceso de pdb](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-set-pdb-path.png)

Después de esta configuración, inicie la depuración con **F5** y los puntos de interrupción. También puede usar las herramientas de depuración de Visual Studio (visualización, variables, etc.) para solucionar el problema.

> [!NOTE]
> Recompile el proyecto de código fuente de ensamblado cada vez que modifique el código para generar archivos .pdb actualizados.

## <a name="resubmit-the-job"></a>Reenviar el trabajo

Tras la depuración, si el proyecto se completa correctamente, la ventana de salida muestra el siguiente mensaje:

`The Program 'LocalVertexHost.exe' has exited with code 0 (0x0).`

![Azure Data Lake Analytics U-SQL debug succeed (La depuración de U-SQL de Azure Data Lake Analytics se realizó correctamente)](./media/data-lake-analytics-debug-u-sql-jobs/data-lake-analytics-debug-succeed.png)

Para volver a enviar el trabajo con error, haga lo siguiente:

1. Para los trabajos con soluciones con código subyacente, copie el código de C# en el archivo de origen de código subyacente (normalmente, `Script.usql.cs`).

2. Para los trabajos con ensamblados, haga clic con el botón derecho en el proyecto de código fuente del ensamblado en la solución de depuración y registre los ensamblados .dll actualizados en el catálogo de Azure Data Lake.

3. Vuelva a enviar el trabajo de U-SQL.

## <a name="next-steps"></a>Pasos siguientes

- [Guía de programación de U-SQL](data-lake-analytics-u-sql-programmability-guide.md)
- [Desarrollo de operadores U-SQL definidos por el usuario para trabajos de Azure Data Lake Analytics](data-lake-analytics-u-sql-develop-user-defined-operators.md)
- [Prueba y depuración de trabajos U-SQL mediante la ejecución local y el SDK de U-SQL para Azure Data Lake](data-lake-analytics-data-lake-tools-local-run.md)
- [Solución de problemas de un trabajo periódico anómalo](data-lake-analytics-data-lake-tools-debug-recurring-job.md)
