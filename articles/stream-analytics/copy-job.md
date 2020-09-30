---
title: Copia o copia de seguridad de trabajos de Azure Stream Analytics
description: En este artículo se describe como copiar o hacer una copia de seguridad de un trabajo de Azure Stream Analytics.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/11/2019
ms.openlocfilehash: e0214cb9515a5974f1de9a27760dc037f7593f30
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90892293"
---
# <a name="copy-or-back-up-azure-stream-analytics-jobs"></a>Copia o copia de seguridad de trabajos de Azure Stream Analytics

Puede copiar o hacer una copia de seguridad de los trabajos de Azure Stream Analytics implementados mediante Visual Studio Code o Visual Studio. Al copiar un trabajo en otra región, no se copia la hora de la última salida. Por lo tanto, no se puede usar la opción [**Última detención**](https://docs.microsoft.com/azure/stream-analytics/start-job#start-options) al iniciar el trabajo copiado.

## <a name="before-you-begin"></a>Antes de empezar
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/).

* Inicie sesión en [Azure Portal](https://portal.azure.com/).

* Instale la [extensión Azure Stream Analytics para Visual Studio Code](quick-create-visual-studio-code.md#install-the-azure-stream-analytics-tools-extension) o las [herramientas de Azure Stream Analytics para Visual Studio](quick-create-visual-studio-code.md#install-the-azure-stream-analytics-tools-extension).  

## <a name="visual-studio-code"></a>Visual Studio Code

1. Haga clic en el icono de **Azure** en la barra de actividades de código de Visual Studio Code y, después, expanda el nodo **Stream Analytics**. Los trabajos deben aparecer en las suscripciones.

   ![Apertura del explorador de Stream Analytics](./media/vscode-explore-jobs/open-explorer.png)

2. Para exportar un trabajo a un proyecto local, busque el trabajo que desea exportar en el **explorador de Stream Analytics** en Visual Studio Code. Después seleccione una carpeta para el proyecto.

    ![Búsqueda de un trabajo de ASA en Visual Studio Code](./media/vscode-explore-jobs/export-job.png)

    El proyecto se exporta a la carpeta que seleccione y se agrega al área de trabajo actual.

    ![Exportación de un trabajo de ASA en Visual Studio Code](./media/stream-analytics-manage-job/copy-backup-stream-analytics-jobs.png)

3. Para publicar el trabajo en otra región o copia de seguridad con otro nombre, seleccione **Seleccione entre las suscripciones para publicar** en el editor de consultas (\*.asaql) y siga las instrucciones.

    ![Publicación en Azure en Visual Studio Code](./media/quick-create-visual-studio-code/submit-job.png)

## <a name="visual-studio"></a>Visual Studio

1. Siga las [instrucciones para exportar a un proyecto un trabajo de Azure Stream Analytics implementado](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-vs-tools#export-jobs-to-a-project).

2. Abra el archivo \*.asaql en el editor de consultas, seleccione **Enviar a Azure** en el editor de scripts y siga las instrucciones para publicar el trabajo en otra región o copia de seguridad con un nombre nuevo.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Visual Studio Code](quick-create-visual-studio-code.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Visual Studio](stream-analytics-quick-create-vs.md)
* [Implementación de un trabajo de Azure Stream Analytics con CI/CD mediante Azure Pipelines](stream-analytics-tools-visual-studio-cicd-vsts.md)
