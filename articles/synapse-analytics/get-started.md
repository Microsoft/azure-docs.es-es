---
title: 'Tutorial: Introducción a Azure Synapse Analytics'
description: En este tutorial, aprenderá los pasos básicos para configurar y usar Azure Synapse Analytics.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: 7affc5aad89fd79e6ba6480f7bf10d37f90dc5e3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075859"
---
# <a name="get-started-with-azure-synapse-analytics"></a>Introducción a Azure Synapse Analytics

Este tutorial es una guía paso a paso de las principales áreas de características de Azure Synapse Analytics. El tutorial es un punto de partida ideal para alguien que desee un paseo guiado por los escenarios principales de Azure Synapse Analytics. Después de seguir los pasos del tutorial, tendrá un área de trabajo de Synapse totalmente funcional en la que podrá empezar a analizar los datos mediante SQL, SQL a petición y Apache Spark.

Aprenderá a:
* Aprovisionar un área de trabajo de Synapse en una suscripción de Azure
* Configurar el control de acceso en una cuenta de ADLSGEN2 para que funcione sin problemas con el área de trabajo de Synapse
* Cargar los datos de ejemplo de NYCTaxi en el área de trabajo de Synapse para que se puedan usar desde SQL, SQL a petición y Spark
* Editar y ejecutar scripts de SQL y cuadernos de Synapse con Synapse Studio
* Consultar tablas de SQL y tablas de Spark
* Cargar datos desde tablas de SQL en dataframes de Spark
* Cargar datos en tablas de SQL desde dataframes de Spark
* Explorar el contenido de una cuenta de ADLSGEN2
* Analizar archivos de Parquet en cuentas de ADLSGEN2 con Spark y SQL a petición 
* Crear una canalización de datos que ejecuta automáticamente un cuaderno de Synapse cada hora

Siga los pasos *por orden* como se muestra a continuación para realizar un paseo guiado por muchas de las funcionalidades y aprender a ejercitar las características principales.

* [Paso 1: creación y configuración de un área de trabajo de Synapse](get-started-create-workspace.md)
* [Paso 2: análisis mediante un grupo de SQL](get-started-analyze-sql-pool.md)
* [Paso 3: análisis mediante Spark](get-started-analyze-spark.md)
* [Paso 4: análisis mediante SQL a petición](get-started-analyze-sql-on-demand.md)
* [Paso 5: análisis de los datos de una cuenta de almacenamiento](get-started-analyze-storage.md)
* [Paso 6: orquestación con canalizaciones](get-started-pipelines.md)
* [Paso 7: visualización de los datos con Power BI](get-started-visualize-power-bi.md)
