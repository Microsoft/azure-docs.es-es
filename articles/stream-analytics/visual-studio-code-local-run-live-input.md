---
title: Prueba de las consultas de Azure Stream Analytics localmente en una entrada de streaming en vivo con Visual Studio Code
description: En este artículo se describe cómo probar las consultas localmente en una entrada de streaming en vivo mediante las herramientas de Azure Stream Analytics para Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/14/2019
ms.topic: how-to
ms.openlocfilehash: 944c1a8cc4606c5e614f17cd0590331826527a7c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90892071"
---
# <a name="test-stream-analytics-queries-locally-against-live-stream-input-by-using-visual-studio-code"></a>Prueba de las consultas de Stream Analytics localmente en una entrada de streaming en vivo con Visual Studio Code

Puede usar las herramientas de Azure Stream Analytics para Visual Studio Code para probar los trabajos de Stream Analytics localmente en una entrada de streaming en vivo. La entrada puede proceder de un origen como Azure Event Hubs o Azure IoT Hub. Los resultados de salida se envían como archivos JSON a una carpeta de su proyecto denominada **LocalRunOutputs**.

## <a name="prerequisites"></a>Prerrequisitos

* Instale el [SDK de .NET Core](https://dotnet.microsoft.com/download) y reinicie Visual Studio Code.

* Use [este inicio rápido](quick-create-visual-studio-code.md) para aprender a crear un trabajo de Stream Analytics mediante Visual Studio Code.

## <a name="define-a-live-stream-input"></a>Definición de una entrada de streaming en vivo

1. Haga clic con el botón derecho en la carpeta **Entradas** del proyecto de Stream Analytics. Después, seleccione **ASA: Agregar entrada** en el menú contextual.

   ![Adición de una entrada desde la carpeta Entradas](./media/quick-create-visual-studio-code/add-input-from-inputs-folder.png)

   También puede seleccionar **Ctrl+Mayús+P** para abrir la paleta de comandos y escribir **ASA: Agregar entrada**.

   ![Adición de una entrada de Stream Analytics en Visual Studio Code](./media/quick-create-visual-studio-code/add-input.png)

2. Elija un tipo de origen de entrada en la lista desplegable.

   ![Selección de un centro de IoT como opción de entrada](./media/quick-create-visual-studio-code/iot-hub.png)

3. Si agregó la entrada desde la paleta de comandos, elija el script de consulta de Stream Analytics que va a utilizar la entrada. Debe rellenarse automáticamente con la ruta de acceso del archivo a **myASAproj.asaql**.

   ![Selección de un script de Stream Analytics en Visual Studio Code](./media/quick-create-visual-studio-code/asa-script.png)

4. Elija **Select from your Azure Subscriptions** (Seleccionar entre las suscripciones de Azure) en el menú desplegable.

    ![Selección en las suscripciones](./media/quick-create-visual-studio-code/add-input-select-subscription.png)

5. Configure el archivo JSON recién generado. Puede usar la característica CodeLens para ayudarle a escribir una cadena, seleccionar en una lista desplegable o cambiar el texto directamente en el archivo. En la captura de pantalla siguiente se muestra **Select from your Subscriptions** (Seleccionar entre las suscripciones) como ejemplo.

   ![Configuración de una entrada en Visual Studio Code](./media/quick-create-visual-studio-code/configure-input.png)

## <a name="preview-input"></a>Vista previa de la entrada

Para asegurarse de que los datos de entrada están disponibles, seleccione **Vista previa de los datos** en su archivo de configuración de entrada en vivo de la línea superior. Algunos datos de entrada proceden de un centro de IoT y se muestran en la ventana de vista previa. La vista previa podría tardar unos segundos en aparecer.

 ![Vista previa de la entrada activa](./media/quick-create-visual-studio-code/preview-live-input.png)

## <a name="run-queries-locally"></a>Ejecutar consultas de forma local

Vuelva al editor de consultas y seleccione **Ejecutar localmente**. Luego, seleccione **Use Live Input** (Usar entrada activa) en la lista desplegable.

![Selección de la opción "Ejecutar localmente" en el editor de consultas](./media/vscode-local-run/run-locally.png)

![Selección de "Use Live Input" (Usar entrada activa)](./media/vscode-local-run-live-input/run-locally-use-live-input.png)

El resultado se muestra en la ventana derecha y se actualiza cada 3 segundos. Puede seleccionar **Ejecutar** para volver a probar. También puede seleccionar **Open in folder** (Abrir en carpeta) para ver los archivos de resultados en el Explorador de archivos y abrirlos con Visual Studio Code o una herramienta como Excel. Tenga en cuenta que los archivos de resultados solo están disponibles en formato JSON.

La hora predeterminada para que el trabajo empiece a crear la salida se establece en **Ahora**. Puede personalizar la hora seleccionando el botón **Hora de inicio de salida** en la ventana de resultados.

![Ver resultado de ejecución local](./media/vscode-local-run-live-input/vscode-livetesting.gif)

## <a name="next-steps"></a>Pasos siguientes

* [Exploración de trabajos de Azure Stream Analytics con Visual Studio Code (versión preliminar)](visual-studio-code-explore-jobs.md)

* [Configuración de canalizaciones de CI/CD mediante el paquete npm](setup-cicd-vs-code.md)
