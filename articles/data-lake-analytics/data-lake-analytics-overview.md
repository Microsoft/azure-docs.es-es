---
title: Introducción a Azure Data Lake Analytics
description: Data Lake Analytics le permite controlar su negocio con los detalles obtenidos de los datos en la nube a cualquier escala.
services: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 1e1d443a-48a2-47fb-bc00-bf88274222de
ms.service: data-lake-analytics
ms.topic: overview
ms.date: 06/23/2017
ms.openlocfilehash: b86b569b1b14b943cdc6dcc805a7b7a208d03447
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475126"
---
# <a name="what-is-azure-data-lake-analytics"></a>¿Qué es Azure Data Lake Analytics?

Azure Data Lake Analytics es un servicio de trabajos de análisis a petición que simplifican los macrodatos. En lugar de implementar, configurar y ajustar el hardware, escribirá consultas para transformar los datos y extraer ideas valiosas. El servicio de análisis puede administrar trabajos de cualquier escala al instante, simplemente estableciendo el ajuste adecuado. Solo tiene que pagar por su trabajo cuando se está ejecutando, lo que hace que sea una solución económica. 

## <a name="azure-data-lake-analytics-recent-update-information"></a>Información de la actualización reciente de Azure Data Lake Analytics

El servicio Azure Data Lake Analytics se actualiza de forma periódica con un fin determinado. Seguimos proporcionando compatibilidad con este servicio con la actualización de componentes, la versión preliminar de los componentes, etc. 

- Para obtener información general sobre la actualización más reciente, consulte [Novedades de Data Lake Analytics](data-lake-analytics-whats-new.md).
- Para obtener información sobre todas las actualizaciones, consulte la [nota de la versión de Azure Data Lake Analytics](https://github.com/Azure/AzureDataLake/tree/master/docs/Release_Notes).

## <a name="dynamic-scaling"></a>Escalado dinámico
  
Data Lake Analytics aprovisiona los recursos de forma dinámica y permite analizar desde terabytes hasta petabytes de datos. Solo paga por la capacidad de procesamiento que se utiliza. Cuando aumente o disminuya el tamaño de los datos almacenados o la cantidad de recursos de proceso utilizados, no tendrá que reescribir código. 

## <a name="develop-faster-debug-and-optimize-smarter-using-familiar-tools"></a>Desarrollo más rápido, depuración y optimización inteligentes mediante herramientas que ya conoce
  
Data Lake Analytics se integra en profundidad con Visual Studio. Puede utilizar herramientas conocidas para ejecutar, depurar y optimizar el código. Las visualizaciones de sus trabajos de U-SQL le permiten ver cómo se ejecuta el código a escala. De este modo, puede identificar fácilmente cuellos de botella en el rendimiento y optimizar los costos.

## <a name="u-sql-simple-and-familiar-powerful-and-extensible"></a>U-SQL: sencillo y familiar, eficiente y extensible
  
Análisis de Data Lake incluye U-SQL, un lenguaje de consulta que amplia la sencilla y familiar naturaleza declarativa de SQL con la capacidad expresiva de C#. El lenguaje U-SQL utiliza el mismo entorno de ejecución distribuido que alimenta el lago de datos interno de Microsoft, a una escala de exabytes. Ahora, los desarrolladores de SQL y .NET pueden procesar y analizar los datos con los conocimientos que ya tienen.

## <a name="integrates-seamlessly-with-your-it-investments"></a>Se integra sin problemas con los elementos de TI en los que ha invertido
  
Data Lake Analytics utiliza las inversiones existentes en TI para identidad, administración y seguridad. De esta forma se simplifica la gobernanza de los datos y se facilita la ampliación de sus aplicaciones de datos actuales. Data Lake Analytics se integra con Active Directory para la administración de usuarios y permisos, y viene con funciones de supervisión y auditoría integradas.

## <a name="affordable-and-cost-effective"></a>Asequible y rentable

Análisis de Data Lake es una solución muy rentable para ejecutar cargas de trabajo de macrodatos. Usted paga por trabajo cuando se procesan los datos. No se requieren licencias, hardware ni contratos de soporte específicos para el servicio. El sistema se escala o reduce verticalmente de forma automática cuando el trabajo comienza y finaliza, por lo que nunca se paga por recursos que no sean estrictamente necesarios. [Más información sobre cómo controlar los costos y ahorrar dinero](https://aka.ms/adlasavemoney).

## <a name="works-with-all-your-azure-data"></a>Funciona con todos los datos de Azure
  
Data Lake Analytics funciona con Azure Data Lake Storage para proporcionar el máximo rendimiento, capacidad de proceso y paralelización, y funciona con los blobs de Azure Storage, Azure SQL Database y Azure Synapse Analytics (antes Azure SQL Data Warehouse).


## <a name="next-steps"></a>Pasos siguientes

* Consulte la actualización más reciente de Azure Data Lake Analytics en [Novedades de Azure Data Lake Analytics](data-lake-analytics-whats-new.md).
* Introducción a Data Lake Analytics mediante [Azure Portal](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli.md)
* Administración de Azure Data Lake Analytics mediante [Azure Portal](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) | [SDK de Azure .NET](data-lake-analytics-manage-use-dotnet-sdk.md) | [Node.js](data-lake-analytics-manage-use-nodejs.md)
* [Control de costos y ahorro de dinero con Data Lake Analytics](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c)
