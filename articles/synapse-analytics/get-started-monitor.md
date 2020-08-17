---
title: 'Tutorial: Introducción a Azure Synapse Analytics: supervisión del área de trabajo de Synapse'
description: En este tutorial, aprenderá a supervisar actividades en el área de trabajo de Synapse.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 07/27/2020
ms.openlocfilehash: 43ed46f56be1ad4fa2aa643a6ed06f8fbe094a5b
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87351945"
---
# <a name="monitor-your-synapse-workspace"></a>Supervisión del área de trabajo de Synapse

En este tutorial, aprenderá a supervisar actividades en el área de trabajo de Synapse. Puede supervisar las actividades actuales e históricas de SQL, Apache Spark y canalizaciones. 

## <a name="introduction-to-the-monitor-hub"></a>Introducción al centro Supervisar

Abra Synapse Studio y vaya al centro **Supervisar**. Aquí, puede ver un historial de todas las actividades que se realizan en el área de trabajo y las que están activas ahora. 

* En **Orquestación**, puede supervisar canalizaciones, desencadenadores y entornos de ejecución de integración.
* En **Actividades**, puede supervisar las actividades de Spark y SQL. 

## <a name="orchestration"></a>Orquestación

1. Vaya a **Orquestación > Canalización**. En esta vista, puede ver cada vez que se ha ejecutado una canalización en el área de trabajo. 
1. Busque la canalización que ejecutó en el paso anterior y haga clic en el **Nombre de canalización**.
1. Ahora puede ver cómo se ejecutan las actividades individuales dentro de esa canalización.
1. Haga clic en **Barra de ruta de navegación** cerca de la parte superior de Synapse Studio, haga clic en **Todas las ejecuciones de la canalización** para volver a la vista anterior.

## <a name="apache-spark-activities"></a>Actividades de Apache Spark

1. Vaya a **Orquestación > Actividades > Aplicaciones de Apache Spark**. Ahora puede ver todas las aplicaciones de Spark que se están ejecutando o que se han ejecutado en el área de trabajo.
1. Busque una aplicación que ya no esté en ejecución y haga clic en el **Nombre de aplicación**. Ahora puede ver los detalles de la aplicación de Spark.
1. Si está familiarizado con Apache Spark, puede encontrar la interfaz de usuario del servidor de historial de Apache Spark estándar haciendo clic en **Servidor de historial de Spark**.

## <a name="sql-activities"></a>Actividades de SQL

1. Vaya a **Orquestación > Actividades > Solicitudes de SQL**.
1. En esta vista puede ver las solicitudes de SQL.
1. Seleccione un **Grupo** al cual supervisar. Ahora puede ver todas las aplicaciones de SQL que se están ejecutando o que se han ejecutado en el área de trabajo de ese grupo.
1. Busque una solicitud de SQL específica y mantenga el puntero sobre ese elemento. Una vez que mantenga el puntero, verá aparecer un icono de script SQL.
1. Haga clic en el icono de script SQL para ver el texto completo de la solicitud SQL.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Azure Synapse Analytics (versión preliminar de las áreas de trabajo)](overview-what-is.md)

