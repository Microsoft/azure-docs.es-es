---
title: Explorar trabajos de Azure Stream Analytics con Visual Studio Code (versión preliminar)
description: Este artículo muestra cómo exportar un trabajo de Azure Stream Analytics a un proyecto local, la lista de trabajos y las entidades job de la vista.
ms.service: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.date: 05/15/2019
ms.topic: conceptual
ms.openlocfilehash: 8674d478646c8f9be6b32521c6624752ac6df052
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827806"
---
# <a name="explore-azure-stream-analytics-with-visual-studio-code-preview"></a>Explorar Azure Stream Analytics con Visual Studio Code (versión preliminar)

Azure Stream Analytics para la extensión de Visual Studio Code ofrece a los desarrolladores una experiencia ligera para administrar sus trabajos de Stream Analytics. Se puede usar en Windows, Mac y Linux. Con la extensión de Azure Stream Analytics, hacer lo siguiente:

- [Crear](quick-create-vs-code.md), iniciar y detener trabajos
- Exportar los trabajos existentes a un proyecto local
- Enumerar los trabajos y ver las entidades job

## <a name="export-a-job-to-a-local-project"></a>Exportar un trabajo a un proyecto local

Para exportar un trabajo a un proyecto local, busque el trabajo que desea exportar en el **Stream Analytics Explorer** en Visual Studio Code. A continuación, seleccione una carpeta para el proyecto. El proyecto se exporta a la carpeta que seleccione, y puede seguir administrar el trabajo desde Visual Studio Code. Para obtener más información sobre el uso de código de Visual Studio para administrar los trabajos de Stream Analytics, consulte el código de Visual Studio [quickstart](quick-create-vs-code.md).

![Exportar trabajo de ASA en Visual Studio Code](./media/vscode-explore-jobs/export-job.png)

## <a name="list-job-and-view-job-entities"></a>Lista de trabajo y ver las entidades job

Puede usar la vista de trabajos para interactuar con trabajos de Azure Stream Analytics desde Visual Studio.


1. Haga clic en el **Azure** icono en la barra de actividades de código de Visual Studio y, a continuación, expanda **nodo Stream Analytics**. Los trabajos deben aparecer en las suscripciones.

   ![Abra Stream Analytics Explorer](./media/vscode-explore-jobs/open-explorer.png)

2. Expanda el nodo de trabajos, puede abrir y ver la consulta del trabajo, configuración, entradas, salidas y funciones. 

3. A la derecha, haga clic en el nodo de trabajos y elija el **abrir la vista de trabajos en el Portal** el nodo para abrir la vista de trabajos en el portal de Azure.

   ![Vista de trabajo abierto en el portal](./media/vscode-explore-jobs/open-job-view.png)

## <a name="next-steps"></a>Pasos siguientes

* [Crear un trabajo en la nube de Azure Stream Analytics en Visual Studio Code (versión preliminar)](quick-create-vs-code.md)
