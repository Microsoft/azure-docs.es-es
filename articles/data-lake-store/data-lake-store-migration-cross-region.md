---
title: Migración entre regiones para Azure Data Lake Storage Gen1 | Microsoft Docs
description: Obtenga información sobre lo que debe tener en cuenta al planear y completar una migración a Azure Data Lake Storage Gen1 a medida que esté disponible en nuevas regiones.
services: data-lake-store
documentationcenter: ''
author: swums
manager: amitkul
editor: swums
ms.assetid: ebde7b9f-2e51-4d43-b7ab-566417221335
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 01/27/2017
ms.author: stewu
ms.openlocfilehash: ba28d767c11c15b2dd70eeed2b39e13b084a7500
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88191357"
---
# <a name="migrate-azure-data-lake-storage-gen1-across-regions"></a>Migración de Azure Data Lake Storage Gen1 entre regiones

A medida que Azure Data Lake Storage Gen1 está disponible en nuevas regiones, puede optar por realizar una migración de una sola vez para aprovechar las ventajas de la nueva región. Obtenga información sobre qué se debe tener en cuenta mientras planea y completa la migración.

## <a name="prerequisites"></a>Prerequisites

* **Una suscripción de Azure**. Para más información, consulte [Cree su cuenta gratuita de Azure hoy mismo](https://azure.microsoft.com/pricing/free-trial/).
* **Una cuenta de Data Lake Storage Gen1 en dos regiones diferentes**. Para más información, consulte [Introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Azure Data Factory**. Para más información, consulte [Introducción a Azure Data Factory](../data-factory/introduction.md).


## <a name="migration-considerations"></a>Consideraciones sobre la migración

En primer lugar, identifique la estrategia de migración que mejor se adapte a la aplicación que escribe, lee y procesa los datos en Data Lake Storage Gen1. Cuando elija una estrategia, tenga en cuenta los requisitos de disponibilidad de la aplicación y el tiempo de inactividad que se produce durante una migración. Por ejemplo, el enfoque más sencillo podría ser usar el modelo de migración a nube "lift-and-shift". En este enfoque, la aplicación se pausa en la región existente mientras todos los datos se copian en la nueva región. Una vez finalizado el proceso de copia, puede reanudar la aplicación en la nueva región y eliminar después la cuenta antigua de Data Lake Storage Gen1. Se requiere un tiempo de inactividad durante la migración.

Para reducir el tiempo de inactividad, puede empezar a ingerir inmediatamente nuevos datos en la nueva región. Cuando tenga los datos mínimos necesarios, ejecute la aplicación en la nueva región. En segundo plano, continúe copiando los datos más antiguos de la cuenta de Azure Data Lake Storage Gen1 existente en la nueva cuenta de Data Lake Storage Gen1 en la nueva región. Mediante este enfoque, puede hacer el cambio a la nueva región con poco tiempo de inactividad. Una vez que se han copiado todos los datos antiguos, elimine la cuenta antigua de Data Lake Storage Gen1.

Otros detalles importantes a tener en cuenta al planear la migración son:

* **Volumen de datos**. El volumen de datos (en gigabytes, el número de archivos y carpetas, etc.) afecta al tiempo y los recursos que necesita para la migración.

* **Nombre de la cuenta de Data Lake Storage Gen1**. El nombre de la nueva cuenta en la nueva región debe ser único globalmente. Por ejemplo, el nombre de la cuenta antigua de Data Lake Storage Gen1 en Este de EE. UU. 2 podría ser contosoeastus2.azuredatalakestore.net. Podría denominar a la nueva cuenta de Data Lake Storage Gen1 en Norte UE contosonortheu.azuredatalakestore.net.

* **Herramientas**. Para copiar los archivos de Data Lake Storage Gen1, le recomendamos usar la [actividad de copia de Azure Data Factory](../data-factory/connector-azure-data-lake-store.md). Data Factory admite el movimiento de datos con alto rendimiento y confiabilidad. Tenga en cuenta que Data Factory solo copia la jerarquía de carpetas y el contenido de los archivos. Debe aplicar manualmente todas las listas de control de acceso (ACL) que utiliza en la cuenta antigua a la nueva cuenta. Para más información, incluidos los objetivos de rendimiento para los mejores escenarios, consulte [Guía de optimización y rendimiento de la actividad de copia](../data-factory/copy-activity-performance.md). Si desea que los datos se copien más rápido, puede que tenga que utilizar unidades adicionales de movimiento de datos de nube. Otras herramientas, como AdlCopy, no admiten la copia de datos entre regiones.  

* **Cobros por ancho de banda**. [Los cobros por ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/) se aplican porque los datos se transfieren fuera de una región de Azure.

* **ACL de los datos**. Proteja los datos en la nueva región aplicando ACL a archivos y carpetas. Para más información, consulte [Protección de los datos almacenados en Azure Data Lake Storage Gen1](data-lake-store-secure-data.md). Se recomienda que realice la migración para actualizar y ajustar las ACL. Puede que le interese usar una configuración similar a la configuración actual. Puede ver las ACL que se aplican a cualquier archivo mediante Azure Portal, [cmdlets de PowerShell](/powershell/module/az.datalakestore/get-azdatalakestoreitempermission) o SDK.  

* **Ubicación de los servicios de análisis**. Para obtener el mejor rendimiento, los servicios de análisis, como Azure Data Lake Analytics o Azure HDInsight, deben estar en la misma región que los datos.  

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Data Lake Storage Gen1](data-lake-store-overview.md)
