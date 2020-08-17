---
title: 'Identificación de escenarios y planeamiento del proceso de análisis: proceso de ciencia de datos en equipos | Azure Machine Learning'
description: Identificación de escenarios y planeamiento del procesamiento de datos analítico avanzado teniendo en cuenta una serie de preguntas.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 29190cee748db17caee32497de5512bee736bae0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87047723"
---
# <a name="how-to-identify-scenarios-and-plan-for-advanced-analytics-data-processing"></a>Identificación de escenarios y planeamiento del procesamiento analítico avanzado de datos

¿Qué recursos son necesarios para crear un entorno que pueda realizar el procesamiento de análisis avanzado en un conjunto de datos? En este artículo se sugiere una serie de preguntas que le pueden ayudar a identificar las tareas y los recursos pertinentes para su escenario.

Para aprender sobre el orden de los pasos de alto nivel para el análisis predictivo, consulte [¿Qué es el Proceso de ciencia de datos en equipo (TDSP)?](overview.md). Cada paso requiere recursos específicos para las tareas pertinentes para su escenario concreto.

Responder preguntas clave en las áreas siguientes para identificar su escenario:

* logística de datos
* características de datos
* calidad del conjunto de datos
* herramientas y lenguajes preferidos

## <a name="logistic-questions-data-locations-and-movement"></a>Cuestiones de logística: movimiento y ubicaciones de los datos

Las cuestiones logísticas tratan los siguientes elementos:

* ubicación de origen de datos
* destino de destino en Azure
* requisitos para mover los datos, incluida la programación, la cantidad y los recursos que están involucrados

Es posible que tenga que mover los datos varias veces durante el proceso de análisis. Un escenario común es mover datos locales a algún tipo de almacenamiento en Azure y, a continuación, en Machine Learning Studio.

### <a name="what-is-your-data-source"></a>¿Cuál es el origen de los datos?

¿Están los datos en local o en la nube? Entre las posibles ubicaciones se incluyen:

* una dirección HTTP disponible públicamente
* una ubicación de archivo local o de red
* una base de datos de SQL Server
* un contenedor de Azure Storage

### <a name="what-is-the-azure-destination"></a>¿Cuál es el destino en Azure?

¿Dónde tienen que estar los datos para procesarse o modelarse? 

* Azure Blob Storage
* Bases de datos SQL Azure
* SQL Server en máquina virtual de Azure
* HDInsight (Hadoop en Azure) o tablas de Hive
* Azure Machine Learning
* Discos duros virtuales de Azure que se pueden montar

### <a name="how-are-you-going-to-move-the-data"></a>¿Cómo va a mover los datos?

Para los procedimientos y los recursos para ingerir o cargar datos en una variedad de entornos de almacenamiento y procesamiento diferentes, consulte:

* [Carga de datos en entornos de almacenamiento para el análisis](ingest-data.md)
* [Importación de datos de entrenamiento en Azure Machine Learning Studio (clásico) desde varios orígenes de datos](../studio/import-data.md)

### <a name="does-the-data-need-to-be-moved-on-a-regular-schedule-or-modified-during-migration"></a>¿Necesitan moverse los datos siguiendo una programación regular o modificarse durante la migración?

Considere el uso de Azure Data Factory (ADF) cuando los datos deban migrarse continuamente. ADF puede ser útil para:

* un escenario híbrido que implique recursos locales y en la nube
* un escenario donde los datos se transfieran, modifiquen o cambien por lógica de negocios mientras se migran

Para más información, consulte [Movimiento de datos desde una base de datos de SQL Server hasta SQL Azure con Azure Data Factory](move-sql-azure-adf.md).

### <a name="how-much-of-the-data-is-to-be-moved-to-azure"></a>¿Qué cantidad de datos se va a mover a Azure?

Los conjuntos de datos grandes pueden superar la capacidad de almacenamiento de ciertos entornos. Para ver un ejemplo, consulte la explicación de los límites de tamaño para Machine Learning Studio (clásico) en la sección siguiente. En tales casos, podría usarse una muestra de los datos durante el análisis. Para obtener más información sobre cómo reducir la muestra de un conjunto de datos en diversos entornos de Azure, consulte [Muestreo de datos del proceso de ciencia de datos en equipos](sample-data.md).

## <a name="data-characteristics-questions-type-format-and-size"></a>Cuestiones sobre las características de los datos: tipo, formato y tamaño

Estas cuestiones son clave para planear los entornos de almacenamiento y procesamiento. Le ayudarán a elegir el escenario adecuado para su tipo de datos y conocer todas las restricciones.

### <a name="what-are-the-data-types"></a>¿Cuáles son los tipos de datos?

* Numérico
* Categorías
* Cadenas
* Binary

### <a name="how-is-your-data-formatted"></a>¿Qué formato tienen los datos?

* Archivos sin formato separados por comas (CSV) o separados por tabulaciones (TSV)
* Comprimidos o sin comprimir
* Blobs de Azure
* Tablas de Hadoop Hive
* Tablas de SQL Server

### <a name="how-large-is-your-data"></a>¿Qué tamaño tienen los datos?

* Pequeño: menos de 2 GB
* Media: más de 2 GB y menos de 10 GB
* Grande: más de 10 GB

Veamos por ejemplo Azure Machine Learning Studio (clásico):

* Para obtener una lista de los formatos de datos y los tipos admitidos por Azure Machine Learning Studio, consulte la sección [Tipos y formatos de datos admitidos](../studio/import-data.md#supported-data-formats-and-data-types) .
* Para obtener más información sobre las limitaciones de otros servicios de Azure usados en el proceso de análisis, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="data-quality-questions-exploration-and-pre-processing"></a>Cuestiones sobre la calidad de los datos: exploración y procesamiento previo

### <a name="what-do-you-know-about-your-data"></a>¿Qué sabe acerca de los datos?

Comprenda las características básicas de los datos:

* Qué patrones o tendencias muestran
* Qué valores atípicos tienen
* Cuántos valores faltan

Este paso es importante para ayudarle a:

* Determinar cuánto procesamiento previo es necesario
* Formular hipótesis que sugieren las características o el tipo de análisis más adecuado
* Formular planes para recopilar datos adicionales

Entre las técnicas útiles para la inspección de datos podemos citar el cálculo de estadísticas descriptivas y los trazados de visualizaciones. Para obtener más información sobre cómo explorar un conjunto de datos en diversos entornos de Azure, consulte [Exploración de datos en el proceso de ciencia de datos en equipos](explore-data.md).

### <a name="does-the-data-require-preprocessing-or-cleaning"></a>¿Los datos requieren un procesamiento previo o una limpieza?

Es posible que deba preprocesar y limpiar los datos antes de poder usar eficazmente el conjunto de datos para el aprendizaje automático. Los datos sin procesar suelen ser ruidosos y no confiables. Les podrían faltar valores. El uso de estos datos para el modelado puede producir resultados engañosos. Para ver una descripción, consulte [Tareas para preparar los datos para el aprendizaje automático mejorado](prepare-data.md).

## <a name="tools-and-languages-questions"></a>Cuestiones sobre herramientas y lenguajes

Hay muchas opciones para lenguajes, entornos de desarrollo y herramientas. Sea consiente de sus necesidades y preferencias.

### <a name="what-languages-do-you-prefer-to-use-for-analysis"></a>¿Qué lenguajes prefiere usar para el análisis?

* R
* Python
* SQL

### <a name="what-tools-should-you-use-for-data-analysis"></a>¿Qué herramientas debe usar para analizar los datos?

* [Microsoft Azure PowerShell](/powershell/azure/): un lenguaje de script que se usa para administrar los recursos de Azure en un lenguaje de script
* [Azure Machine Learning Studio](../studio/what-is-ml-studio.md)
* [Revolution Analytics](https://www.microsoft.com/sql-server/machinelearningserver)
* [RStudio](https://www.rstudio.com)
* [Herramientas de Python para Visual Studio](https://aka.ms/ptvsdocs)
* [Anaconda](https://www.anaconda.com/)
* [Jupyter Notebooks](https://jupyter.org/)
* [Microsoft Power BI](https://powerbi.microsoft.com)

## <a name="identify-your-advanced-analytics-scenario"></a>Identificación del escenario de análisis avanzado

Después de haber respondido a las preguntas de la sección anterior, estará listo para determinar qué escenario se adapta mejor a su caso. Los escenarios de ejemplo que se describen en [Escenarios para análisis avanzado en Azure Machine Learning](plan-sample-scenarios.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [¿Qué es el proceso de ciencia de datos en equipos (TDSP)?](overview.md)
