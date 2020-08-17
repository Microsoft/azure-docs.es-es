---
title: Depurar los trabajos recurrentes en Azure Data Lake Analytics
description: Aprenda a usar las herramientas de Azure Data Lake para Visual Studio para la depuración de trabajos periódicos anómalos.
services: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 05/20/2018
ms.openlocfilehash: 86d5134e257d2dae642eceb933a78047773b25a9
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87129957"
---
# <a name="troubleshoot-an-abnormal-recurring-job"></a>Solución de problemas de un trabajo periódico anómalo

En este artículo se explica cómo usar las [Herramientas de Azure Data Lake para Visual Studio](https://aka.ms/adltoolsvs) para solucionar problemas de trabajos periódicos. Obtenga más información sobre la canalización y los trabajos periódicos en el [blog de Azure Data Lake y Azure HDInsight](https://blogs.msdn.microsoft.com/azuredatalake/2017/09/19/managing-pipeline-recurring-jobs-in-azure-data-lake-analytics-made-easy/).

Normalmente, los trabajos periódicos comparten la misma lógica de consulta y datos de entrada similares. Por ejemplo, imagine que tiene un trabajo periódico que se ejecuta todos los lunes por la mañana a las 8 a. m. para contar el usuario activo semanal de la semana pasada. Los scripts de estos trabajos comparten una plantilla de script que contiene la lógica de consulta. Las entradas de estos trabajos son los datos de uso de la semana pasada. El uso compartido de la misma lógica de consulta y de entradas afines normalmente significa que el rendimiento de estos trabajos es estable y similar. Si uno de los trabajos periódicos de repente se lleva a cabo de forma anómala, genera un error o se ralentiza mucho, debe hacer lo siguiente:

- Consulte los informes de las ejecuciones anteriores del trabajo periódico para ver lo que ha sucedido.
- Compare el trabajo anómalo con uno normal para descubrir lo que ha cambiado.

La **vista de trabajos relacionados** de las herramientas de Azure Data Lake para Visual Studio le ayuda a acelerar el progreso de la solución de problemas en ambos casos.

## <a name="step-1-find-recurring-jobs-and-open-related-job-view"></a>Paso 1: Búsqueda de trabajos periódicos y apertura de la vista de trabajos relacionados

Para usar la vista de trabajos relacionados para solucionar un problema de un trabajo periódico, primero es preciso buscar el trabajo periódico en Visual Studio y, después, abrir la vista de trabajos relacionados.

### <a name="case-1-you-have-the-url-for-the-recurring-job"></a>Caso 1: Tiene la dirección URL del trabajo periódico

En **Herramientas** > **Data Lake** > **Vista Trabajo**, puede copiar la dirección URL del trabajo para abrir la vista Trabajo en Visual Studio. Seleccione **Ver trabajos relacionados** para abrir la vista de trabajos relacionados.

![Vínculo a Ver trabajos relacionados en las Herramientas de Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/view-related-job.png)
 
### <a name="case-2-you-have-the-pipeline-for-the-recurring-job-but-not-the-url"></a>Caso 2: Tiene la canalización del trabajo periódico, pero no la dirección URL

En Visual Studio, puede abrir el Explorador de canalizaciones a través del Explorador de servidores > su cuenta de Azure Data Lake Analytics > **Canalizaciones**. (Si no se encuentra este nodo en el Explorador de servidores, [descargue el último complemento](https://aka.ms/adltoolsvs)). 

![Selección del nodo Canalizaciones](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/pipeline-browser.png)

En el Explorador de canalizaciones, se muestran todas las canalizaciones de la cuenta de Data Lake Analytics a la izquierda. Puede expandir las canalizaciones para encontrar todos los trabajos periódicos y después seleccionar el que plantea problemas. La vista de trabajos relacionados se abre a la derecha.

![Selección de una canalización y apertura de la vista de trabajos relacionados](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-view.png)

## <a name="step-2-analyze-a-statistics-report"></a>Paso 2: Análisis de un informe de estadísticas

Un resumen y un informe de estadísticas se muestran en la parte superior de la vista de trabajos relacionados. Allí, puede encontrar la posible causa principal del problema. 

1.  En el informe, el eje X muestra la hora de envío del trabajo. Úsela para buscar el trabajo anómalo.
2.  Utilice el proceso del diagrama siguiente para comprobar las estadísticas y obtener información sobre el problema y las posibles soluciones.

![Diagrama del proceso de comprobación de estadísticas](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-metrics-debugging-flow.png)

## <a name="step-3-compare-the-abnormal-job-to-a-normal-job"></a>Paso 3: Comparación del trabajo anómalo con un trabajo normal

Todos los trabajos periódicos enviados se pueden encontrar en la lista de trabajos en la parte inferior de la vista de trabajos relacionados. Para obtener más información y posibles soluciones, haga clic con el botón derecho en el trabajo anómalo. Utilice la vista Diferencias de trabajo para comparar el trabajo anómalo con uno normal anterior.

![Menú contextual para comparar trabajos](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/compare-job.png)

Preste atención a las grandes diferencias entre estos dos trabajos. Esas diferencias probablemente están causando los problemas de rendimiento. Para realizar otras comprobaciones, siga los pasos de este diagrama:

![Diagrama del proceso de comprobación de diferencias entre los trabajos](./media/data-lake-analytics-data-lake-tools-debug-recurring-job/recurring-job-diff-debugging-flow.png)

## <a name="next-steps"></a>Pasos siguientes

* [Resolución de problemas de asimetría de datos](data-lake-analytics-data-lake-tools-data-skew-solutions.md)
* [Depuración de código C# definido por el usuario para trabajos de U-SQL con errores](data-lake-analytics-debug-u-sql-jobs.md)
