---
title: Prueba de un trabajo de Azure Stream Analytics localmente con datos de ejemplo mediante Visual Studio Code
description: En este artículo se describe cómo probar las consultas localmente con datos de ejemplo mediante las herramientas de Azure Stream Analytics para Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/10/2019
ms.topic: how-to
ms.openlocfilehash: 0122e67714efb201584a52f1cb763528dbb0b925
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903814"
---
# <a name="test-stream-analytics-queries-locally-with-sample-data-using-visual-studio-code"></a>Prueba de consultas de Stream Analytics localmente con datos de ejemplo mediante Visual Studio Code

Puede usar las herramientas de Azure Stream Analytics para Visual Studio Code para probar los trabajos de Stream Analytics localmente con datos de ejemplo. Puede encontrar los resultados de la consulta en archivos JSON en la carpeta **LocalRunOutputs** del proyecto.

## <a name="prerequisites"></a>Prerrequisitos

* Instale el [SDK de .NET Core](https://dotnet.microsoft.com/download) y reinicie Visual Studio Code.

* Use este [inicio rápido](quick-create-visual-studio-code.md) para aprender a crear un trabajo de Stream Analytics mediante Visual Studio Code.

## <a name="prepare-sample-data"></a>Preparación de datos de ejemplo

Primero debe preparar los archivos de datos de entrada de ejemplo. Si ya tiene algunos archivos de datos de ejemplo en el equipo, puede omitir este paso y pasar al siguiente.

1. Haga clic en **Vista previa de los datos** en el archivo de configuración de entrada desde la línea superior. Algunos datos de entrada se capturarán desde IoT Hub y se mostrarán en la ventana de vista previa. Tenga en cuenta que esto puede tardar cierto tiempo.

2. Una vez que se muestren los datos, haga clic en **Guardar como** para guardar los datos en un archivo local.

 ![Vista previa de la entrada activa](./media/quick-create-visual-studio-code/preview-live-input.png)

## <a name="define-a-local-input"></a>Definición de una entrada local

1. Haga clic en **input.json** en la carpeta Entradas del proyecto de Stream Analytics. Luego, seleccione **Agregar entrada local** en la línea superior.

    ![Agregar entrada local desde el proyecto](./media/quick-create-visual-studio-code/add-input-from-project.png)

    También puede usar **Ctrl+Mayús+P** para abrir la paleta de comandos y escribir **ASA: Agregar entrada**.

   ![Adición de entrada de Stream Analytics en VS Code](./media/quick-create-visual-studio-code/add-input.png)

2. Seleccione una **entrada local**.

    ![Adición de una entrada local de ASA en Visual Studio Code](./media/vscode-local-run/add-local-input.png)

3. Seleccione **+ Nueva entrada local**.

    ![Adición de una nueva entrada local de ASA en Visual Studio Code](./media/vscode-local-run/add-new-local-input.png)

4. Escriba el mismo alias de entrada que usó en la consulta.

    ![Adición de un nuevo alias de entrada local de ASA](./media/vscode-local-run/new-local-input-alias.png)

5. En el archivo**LocalInput_Input.json** recién generado, escriba la ruta de acceso del archivo donde se encuentra el archivo de datos local.

    ![Entrada de la ruta de acceso de archivo local en Visual Studio](./media/vscode-local-run/local-file-path.png)

6. Seleccione **Vista previa de los datos**  para obtener una vista previa de los datos de entrada. El tipo de serialización de los datos se detecta automáticamente si es JSON o CSV. Use el selector para ver los datos en los formatos de **tabla** o **sin procesar**. La siguiente tabla es un ejemplo de datos en **formato de tabla**:

     ![Obtención de una vista previa de los datos locales en formato de tabla](./media/vscode-local-run/local-file-preview-table.png)

    La tabla siguiente es un ejemplo de datos en **formato sin procesar**:

    ![Obtención de una vista previa de los datos locales en formato sin procesar](./media/vscode-local-run/local-file-preview-raw.png)

## <a name="run-queries-locally"></a>Ejecutar consultas de forma local

Vuelva al editor de consultas y seleccione **Ejecutar localmente**. Luego, seleccione **Use local input** (Usar entrada local) en la lista desplegable.

![Selección de la opción Ejecutar localmente en el editor de consultas](./media/vscode-local-run/run-locally.png)

![Usar entrada local](./media/vscode-local-run/run-locally-use-local-input.png)

El resultado se muestra en la ventana derecha. Puede hacer clic en **Ejecutar** para volver a probar. También puede seleccionar **Open in folder** (Abrir en carpeta) para ver los archivos de resultados en el explorador de archivos y abrirlos con otras herramientas. Tenga en cuenta que los archivos de resultados solo están disponibles en formato JSON.

![Ver resultado de ejecución local](./media/vscode-local-run/run-locally-result.png)

## <a name="next-steps"></a>Pasos siguientes

* [Prueba de los trabajos de Azure Stream Analytics localmente con una entrada activa mediante Visual Studio Code](visual-studio-code-local-run-live-input.md)

* [Exploración de trabajos de Azure Stream Analytics con Visual Studio Code (versión preliminar)](visual-studio-code-explore-jobs.md)
