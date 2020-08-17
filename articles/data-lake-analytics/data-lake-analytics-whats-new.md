---
title: Cambios recientes en Data Lake Analytics
description: En este artículo se proporciona una lista de los últimos cambios realizados en Data Lake Analytics.
services: data-lake-analytics
author: xujiang1
ms.service: data-lake-analytics
ms.devlang: na
ms.topic: overview
ms.author: xujiang1
ms.date: 07/31/2020
ms.openlocfilehash: e78389ffc06f1b4cd4e39c15ac66215d514e9bc1
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87476324"
---
# <a name="whats-new-in-data-lake-analytics"></a>Novedades de Data Lake Analytics

Azure Data Lake Analytics se actualiza de forma periódica en determinados componentes. Para mantenerse al día con las actualizaciones más recientes, este artículo proporciona información acerca de:

- La notificación de la versión preliminar beta del componente clave.
- La información importante sobre la versión del componente, por ejemplo: la lista de versiones disponibles del componente, la versión predeterminada actual, etc.


## <a name="notification-of-key-component-beta-preview"></a>Notificación de la versión preliminar beta del componente clave

No hay ninguna versión beta del componente clave disponible para la versión preliminar. 

## <a name="u-sql-runtime"></a>Runtime de U-SQL

El runtime de U-SQL de Azure Data Lake, incluidos el compilador, el optimizador y el administrador de trabajos, es lo que procesa el código de U-SQL.

Cuando envíe el trabajo de análisis de Azure Data Lake desde cualquier herramienta, el trabajo usará el runtime predeterminado disponible actualmente en el entorno de producción. 

La versión del runtime se actualizará de forma periódica. Y el runtime anterior se mantendrá disponible durante algún tiempo. Cuando esté disponible una nueva versión beta preparada para ser versión preliminar.

A continuación se muestran las versiones de runtime actualmente disponibles.

- release-20200124live_adl_16283022_2 --> **la versión predeterminada actual**
- release_20200124live_adl_16283022
- release_20200124_adl_14480125
- release_20190904_adl_10236248_1
- release_20190904_adl_10236248
- release_20190904_adl_9225818

Para aprender a solucionar los errores de,l runtime de U-SQL, consulte [Solución de problemas del runtime de U-SQL](runtime-troubleshoot.md).

## <a name="net-framework"></a>.NET Framework

Azure Data Lake Analytics usa **.NET Framework v4.7.2**. 

Si el código del script de U-SQL de Azure Data Lake Analytics usa ensamblados personalizados y dichos ensamblados usan bibliotecas de .NET, valide el código para comprobar si hay alguna interrupción.

Para aprender a solucionar problemas de cualquier actualización de .NET, consulte [Solución de problemas de las actualizaciones de .NET](runtime-troubleshoot.md).

## <a name="release-note"></a>Nota de la versión

Para obtener información sobre la actualización más recientes, consulte la [nota de la versión de Azure Data Lake Analytics](https://github.com/Azure/AzureDataLake/tree/master/docs/Release_Notes).


## <a name="next-steps"></a>Pasos siguientes

* Introducción a Data Lake Analytics mediante [Azure Portal](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli.md)

