---
title: 'Tutorial: Escritura de funciones de C# definidas por el usuario para trabajos de Azure Stream Analytics en Visual Studio (versión preliminar)'
description: En este tutorial se muestra cómo escribir funciones de C# definidas por el usuario para trabajos de Stream Analytics en Visual Studio.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: tutorial
ms.date: 12/06/2018
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: b1a361b2c1b1010c6593defa37ef92a3c36b0693
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89015618"
---
# <a name="tutorial-write-a-c-user-defined-function-for-azure-stream-analytics-job-preview"></a>Tutorial: Escritura de una función de C# definida por el usuario para un trabajo de Azure Stream Analytics (Versión preliminar)

Las funciones definidas por el usuario de C# (UDF) creadas en Visual Studio le permiten ampliar el lenguaje de consulta de Azure Stream Analytics con sus propias funciones. Puede reutilizar el código existente (incluidos los archivos DLL) y usar lógica compleja o matemática con C#. Hay tres maneras de implementar las UDF: archivos de CodeBehind en un proyecto de Stream Analytics, UDF de un proyecto de C# local o UDF de un paquete existente desde una cuenta de almacenamiento. En este tutorial se utiliza el método CodeBehind para implementar una función de C# básica. La característica UDF para trabajos de Stream Analytics está actualmente en versión preliminar y no debe usarse en las cargas de trabajo de producción.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una función de definidos por el usuario de C# mediante CodeBehind.
> * Probar el trabajo de Stream Analytics localmente.
> * Publicar un trabajo en Azure.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que cumple los siguientes requisitos previos:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Instale las [herramientas de Stream Analytics para Visual Studio](stream-analytics-tools-for-visual-studio-install.md) y las cargas de trabajo de **desarrollo de Azure** o de **almacenamiento y procesamiento de datos**.
* Eche un vistazo a la [guía de desarrollo de Stream Analytics Edge](stream-analytics-tools-for-visual-studio-edge-jobs.md) existente si va a crear un trabajo de IoT Edge.

## <a name="create-a-container-in-your-azure-storage-account"></a>Creación de un contenedor en una cuenta de Azure Storage

El contenedor que cree se usará para almacenar el paquete compilado C#. Si crea un trabajo de Edge, esta cuenta de almacenamiento también se usará para implementar el paquete en el dispositivo IoT Edge. Use un contenedor dedicado para cada trabajo de Stream Analytics. No se admite la reutilización del mismo contenedor para varios trabajos de Stream Analytics en dispositivos perimetrales. Si ya tiene una cuenta de almacenamiento con los contenedores existentes, puede usarlos. De lo contrario, tendrá que [crear un nuevo contenedor](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal). 

## <a name="create-a-stream-analytics-project-in-visual-studio"></a>Creación de un proyecto de Stream Analytics en Visual Studio

1. Inicie Visual Studio.

2. Seleccione **Archivo > Nuevo > Proyecto**.

3. En la lista de plantillas de la izquierda, seleccione **Stream Analytics** y **Aplicación de Azure Stream Analytics Edge** o **Aplicación de Azure Stream Analytics**.

4.  Escriba el **nombre** del proyecto, la **ubicación** y el **nombre de la solución** y seleccione **Aceptar**.

    ![Creación de un proyecto de Azure Stream Analytics en IoT Edge en Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-create-edge-app.png)

## <a name="configure-assembly-package-path"></a>Configuración de la ruta de acceso de un paquete de ensamblado

1. Abra Visual Studio y vaya al **Explorador de soluciones**.

2. Haga doble clic en el archivo de configuración de trabajos, `EdgeJobConfig.json`.

3. Expanda la sección **Configuración de código definido por el usuario** y rellene la configuración con los siguientes valores sugeridos:

   |**Configuración**|**Valor sugerido**|
   |-------|---------------|
   |Recurso de configuración de almacenamiento global|Selección del origen de datos desde la cuenta actual|
   |Suscripción de configuración de almacenamiento global| < su suscripción >|
   |Cuenta de almacenamiento de la configuración de almacenamiento global| < su cuenta de almacenamiento >|
   |Recurso de configuración de almacenamiento de código personalizado|Selección del origen de datos desde la cuenta actual|
   |Cuenta de almacenamiento de configuración de almacenamiento de código personalizado|< su cuenta de almacenamiento >|
   |Contenedor de configuración de almacenamiento de código personalizado|< su contenedor de almacenamiento >|


## <a name="write-a-c-udf-with-codebehind"></a>Escritura de un UDF de C# con CodeBehind
Un archivo CodeBehind es un archivo de C# asociado con un script de consulta de ASA individual. Las herramientas de Visual Studio comprimirán automáticamente el archivo CodeBehind y lo cargarán en su cuenta de Azure Storage tras el envío. Todas las clases deben definirse como *pública* y los objetos deben definirse como *público estático*.

1. En el **Explorador de soluciones**, expanda **Script.asql** para buscar el archivo CodeBehind **Script.asaql.cs**.

2. Reemplace el código por el del ejemplo siguiente:

    ```csharp
        using System; 
        using System.Collections.Generic; 
        using System.IO; 
        using System.Linq; 
        using System.Text; 
    
        namespace ASAEdgeUDFDemo 
        { 
            public class Class1 
            { 
                // Public static function 
                public static Int64 SquareFunction(Int64 a) 
                { 
                    return a * a; 
                } 
            } 
        } 
    ```

## <a name="implement-the-udf"></a>Implementación de UDF

1. En el **Explorador de soluciones**, abra el archivo **Script.asaql**.

2. Reemplace la consulta existente por lo siguiente:

    ```sql
        SELECT machine.temperature, udf.ASAEdgeUDFDemo_Class1_SquareFunction(try_cast(machine.temperature as bigint))
        INTO Output
        FROM Input 
    ```

## <a name="local-testing"></a>Pruebas locales

1. Descargue el [archivo de datos de ejemplo del simulador de temperatura](https://raw.githubusercontent.com/Azure/azure-stream-analytics/master/Sample%20Data/TemperatureSampleData.json).

2. En el **Explorador de soluciones**, expanda **Inputs**, haga clic con el botón derecho en **Input.json**y seleccione **Agregar entrada local**.

   ![Adición de una entrada local para el trabajo de Stream Analytics en Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-add-local-input.png)

3. Especifique la ruta de acceso del archivo de entrada local para los datos de ejemplo que descargó y haga clic en **Guardar**.

    ![Configuración de entrada local para el trabajo de Stream Analytics en Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-local-input-config.png)

4. Haga clic en **Ejecutar localmente** en el editor de scripts. Una vez que la ejecución local haya guardado correctamente los resultados de la salida, presione cualquier tecla para ver los resultados en formato de tabla. 

    ![Ejecutar un trabajo de Azure Stream Analytics localmente con Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-run-locally.png)

5. También puede seleccionar **Abrir la carpeta de resultados** para ver los archivos sin formato en formato CSV y JSON.

    ![Ver los resultados de un trabajo de Azure Stream Analytics local con Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-view-local-results.png)

## <a name="debug-a-udf"></a>Depuración de una UDF
Los UDF de C# se pueden depurar localmente de la misma manera que el código de C# estándar. 

1. Agregue puntos de interrupción a la función de C#.

    ![Adición de puntos de interrupción a una función definida por el usuario de Stream Analytics en Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-udf-breakpoints.png)

2. Pulse **F5** para iniciar la depuración. El programa se detendrá en los puntos de interrupción según lo previsto.

    ![Visualización de los resultados de depuración de la función definida por el usuario de Stream Analytics](./media/stream-analytics-edge-csharp-udf/stream-analytics-udf-debug.png)

## <a name="publish-your-job-to-azure"></a>Publicación de un trabajo en Azure
Una vez que haya probado una consulta localmente, seleccione **Enviar a Azure** en el editor de scripts para publicar el trabajo en Azure.

![Enviar un trabajo de Stream Analytics en dispositivos perimetrales a Azure desde Visual Studio](./media/stream-analytics-edge-csharp-udf/stream-analytics-udf-submit-job.png)

## <a name="deploy-to-iot-edge-devices"></a>Implementación en dispositivos de IoT Edge
Si decide compilar un trabajo de Stream Analytics Edge, ahora se puede implementar como un módulo de IoT Edge. Siga la [guía de inicio rápido de IoT Edge](https://docs.microsoft.com/azure/iot-edge/quickstart) para crear una instancia de IoT Hub, registrar un dispositivo de IoT Edge e instalar e iniciar el runtime de IoT Edge en el dispositivo. A continuación, siga el tutorial de [implementación del trabajo](https://docs.microsoft.com/azure/iot-edge/tutorial-deploy-stream-analytics#deploy-the-job) para implementar el trabajo de Stream Analytics como un módulo de IoT Edge. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una función simple definida por el usuario de C# mediante CodeBehind, ha publicado un trabajo en Azure y lo ha implementado en el dispositivo de IoT Edge o Azure. 

Para más información acerca de las distintas formas de usar funciones definidas por el usuario de C# para los trabajos de Stream Analytics, continúe en este artículo:

> [!div class="nextstepaction"]
> [Escritura de funciones definidas por el usuario de C# en Azure Stream Analytics](stream-analytics-edge-csharp-udf-methods.md)
