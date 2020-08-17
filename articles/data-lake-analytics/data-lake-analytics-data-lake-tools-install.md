---
title: Instalación de Herramientas de Azure Data Lake para Visual Studio
description: En este artículo se describe cómo instalar Herramientas de Azure Data Lake para Visual Studio.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: ad8a6992-02c7-47d4-a108-62fc5a0777a3
ms.topic: how-to
ms.date: 08/30/2019
ms.openlocfilehash: df3bf715f8571659d0e622729562055428ab5c97
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125945"
---
# <a name="install-data-lake-tools-for-visual-studio"></a>Instalación de Data Lake Tools para Visual Studio

Aprenda a usar Visual Studio para crear cuentas de Azure Data Lake Analytics. Puede definir trabajos en [U-SQL](data-lake-analytics-u-sql-get-started.md) y enviarlos al servicio de Data Lake Analytics. Para obtener más información acerca de Análisis de Data Lake, consulte [Información general sobre Análisis de Azure Data Lake](data-lake-analytics-overview.md).

## <a name="prerequisites"></a>Prerequisites

* **Visual Studio**: se admiten todas las ediciones, excepto Express.

  * Visual Studio 2019
  * Visual Studio 2017
  * Visual Studio 2015
  * Visual Studio 2013

* **Microsoft Azure SDK para .NET** versión 2.7.1, o posterior. Instálelo con el [Instalador de plataforma web](https://www.microsoft.com/web/downloads/platform.aspx).
* Una cuenta de **Data Lake Analytics**. Para crear una cuenta, consulte [Introducción a Azure Data Lake Analytics mediante Azure Portal](data-lake-analytics-get-started-portal.md).

## <a name="install-azure-data-lake-tools-for-visual-studio-2017-or-visual-studio-2019"></a>Instalación de Herramientas de Azure Data Lake para Visual Studio 2017 o Visual Studio 2019

El complemento Herramientas de Azure Data Lake para Visual Studio se puede usar en Visual Studio 2017 15.3 o versiones posteriores. La herramienta forma parte de las cargas de trabajo **Almacenamiento y procesamiento de datos** y **Desarrollo de Azure**. Habilite cualquiera de estas dos cargas como parte de la instalación de Visual Studio.

Habilite la carga de trabajo de **procesamiento y almacenamiento de datos** como se muestra:

![Habilitar la carga de trabajo de procesamiento y almacenamiento de datos](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-tools-for-vs-2019-install-01.png)

Habilite la carga de trabajo de **desarrollo de Azure** como se muestra:

![Seleccionar la carga de trabajo de desarrollo de Azure](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-tools-for-vs-2019-install-02.png)

## <a name="install-azure-data-lake-tools-for-visual-studio-2013-and-2015"></a>Instalación de Herramientas de Azure Data Lake para Visual Studio 2013 y 2015

Descargue e instale [las herramientas de Stream Analytics y Microsoft Azure Data Lake para Visual Studio](https://aka.ms/adltoolsvs). Después de la instalación, Visual Studio presenta los siguientes cambios:

* El nodo de **Explorador de servidores** > **Azure** contiene un nodo de **Data Lake Analytics**.
* El menú **Herramientas** tiene un elemento de **Data Lake**.

## <a name="next-steps"></a>Pasos siguientes

* Para registrar información de diagnóstico, consulte [Acceso a los registros de diagnóstico de Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md).
* Para ver una consulta más compleja, consulte la página sobre el [análisis de registros de sitio web mediante Análisis de Azure Data Lake](data-lake-analytics-analyze-weblogs.md).
* Para usar la vista de ejecución de vértices, vea [Uso de la vista de ejecución de vértices de Data Lake Tools para Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md).
