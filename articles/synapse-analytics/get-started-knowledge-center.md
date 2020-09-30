---
title: 'Tutorial: Introducción a la exploración del Centro de conocimientos de Synapse'
description: En este tutorial, aprenderá a usar el Centro de conocimientos de Synapse.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 09/15/2020
ms.openlocfilehash: c01d1bcb682a5f711dcba3cc7b32ef69b2642ef6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900768"
---
# <a name="explore-the-synapse-knowledge-center"></a>Exploración del Centro de conocimientos de Synapse

En este tutorial, aprenderá a usar el Centro de conocimientos de Synapse Studio.

## <a name="getting-to-the-knowledge-center"></a>Búsqueda del Centro de conocimientos

Hay dos formas de buscar el Centro de conocimientos en Synapse Studio:

  1. En la central de inicio, en Vínculos útiles, haga clic en el primer vínculo denominado **Centro de conocimientos**.
  2. En la barra de menú de la parte superior, haga clic en **?** y, a continuación, en **Centro de conocimientos**.

Elija cualquier método y abra el **Centro de conocimientos**.

## <a name="overview"></a>Información general

El **Centro de conocimientos** le permite hacer tres cosas:
* **Usar ejemplos inmediatamente**. Esta opción está optimizada para ver el análisis en acción lo más rápido posible. Si quiere un ejemplo rápido de cómo funciona Synapse, elija esta opción.
* **Examinar ejemplo disponible**. Esta opción le permite vincular conjuntos de datos de ejemplo y agregar código de ejemplo en forma de scripts de SQL, cuadernos y canalizaciones.
* **Visita por Synapse Studio**. Esta opción le permite realizar una breve visita por las partes básicas de Synapse Studio. Resulta útil si nunca ha usado Synapse Studio.

## <a name="exploring-blob-storage-with-sql-on-demand"></a>Exploración de Blob Storage con SQL a petición

1. En el **Centro de conocimientos**, haga clic en **Usar ejemplos inmediatamente**.
1. Seleccione **Consultar datos con SQL**. 
1. Haga clic en **Usar ejemplos inmediatamente**.
1. Se creará un nuevo script de SQL.
1. Desplácese hasta la primera consulta (líneas 28 a 32) y seleccione el texto de la consulta.
1. Haga clic en Ejecutar. Se ejecutará el texto seleccionado.

## <a name="loading-more-nyc-taxi-data"></a>Carga de más datos de taxis de Nueva York

1. En el **Centro de conocimientos**, haga clic en **Examinar ejemplos disponibles**. 
1. Seleccione la pestaña **Scripts de SQL** en la parte superior.
1. Seleccione **Carga de conjunto de datos de taxis de Nueva York**
1. En **Entradas**, elija **Seleccione un grupo existente** y seleccione **SQLDB1**.
1. Haga clic en **Abrir script**.
1. Aparecerá un nuevo script de SQL.
1. Haga clic en **Ejecutar**
1. Se crearán varias tablas para todos los datos de taxi de Nueva York y se cargarán con el comando T-SQL COPY.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis mediante un grupo de SQL](get-started-analyze-sql-pool.md)
