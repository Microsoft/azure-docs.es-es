---
title: Probar las consultas de Azure Stream Analytics localmente con Visual Studio Code (versión preliminar)
description: En este artículo se describe cómo probar las consultas de forma local con herramientas de Azure Stream Analytics para Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 05/15/2019
ms.topic: conceptual
ms.openlocfilehash: f477a0f99c3eaa82568d8188bfaae03818fb72dc
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827956"
---
# <a name="test-stream-analytics-queries-locally-with-visual-studio-code"></a>Probar las consultas de Stream Analytics localmente con Visual Studio Code

Puede usar herramientas de Azure Stream Analytics para Visual Studio Code para probar los trabajos de Stream Analytics localmente con datos de ejemplo.

Use esta [quickstart](quick-create-vs-code.md) para aprender a crear un trabajo de Stream Analytics mediante Visual Studio Code.

## <a name="run-queries-locally"></a>Ejecutar consultas de forma local

Puede usar la extensión de Azure Stream Analytics para Visual Studio Code para probar los trabajos de Stream Analytics localmente con datos de ejemplo.

1. Una vez que ha creado el trabajo de Stream Analytics, use **Ctrl + Mayús + P** para abrir la paleta de comandos. A continuación, escriba y seleccione **ASA: Agregar entrada**.

    ![Agregar entrada de ASA en Visual Studio code](./media/vscode-local-run/add-input.png)

2. Seleccione **entrada Local**.

    ![Agregar entrada local de ASA en Visual Studio code](./media/vscode-local-run/add-local-input.png)

3. Seleccione **+ nueva entrada Local**.

    ![Agregar un nuevo ASA local de entrada en el código de Visual Studio](./media/vscode-local-run/add-new-local-input.png)

4. Escriba el mismo alias de entrada que usó en la consulta.

    ![Agregar un nuevo alias de entrada local de ASA](./media/vscode-local-run/new-local-input-alias.png)

5. En el **LocalInput_DefaultLocalStream.json** de archivo, escriba la ruta de acceso del archivo donde se encuentra el archivo de datos local.

    ![Escriba la ruta de acceso local en Visual Studio](./media/vscode-local-run/local-file-path.png)

6. Vuelva al editor de consultas y seleccione **ejecutar localmente**.

    ![Seleccione ejecutar localmente en el editor de consultas](./media/vscode-local-run/run-locally.png)

## <a name="next-steps"></a>Pasos siguientes

* [Crear un trabajo en la nube de Azure Stream Analytics en Visual Studio Code (versión preliminar)](quick-create-vs-code.md)

* [Explorar trabajos de Azure Stream Analytics con Visual Studio Code (versión preliminar)](vscode-explore-jobs.md)
