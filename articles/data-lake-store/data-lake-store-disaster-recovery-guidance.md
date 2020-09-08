---
title: Guía de recuperación ante desastres para Azure Data Lake Storage Gen1 | Microsoft Docs
description: Obtenga información sobre cómo proteger aún más los datos ante interrupciones en toda la región o eliminaciones accidentales más allá del almacenamiento con redundancia local de Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: twooley
ms.openlocfilehash: b124f828c4a6a019c45243528ed2d957e3f781f3
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88191410"
---
# <a name="high-availability-and-disaster-recovery-guidance-for-data-lake-storage-gen1"></a>Guía de alta disponibilidad y recuperación ante desastres para Data Lake Storage Gen1

Data Lake Storage Gen1 proporciona almacenamiento con redundancia local (LRS). Por lo tanto, los datos de su cuenta de Data Lake Storage Gen1 resisten los errores transitorios de hardware dentro de un centro de datos a través de las réplicas automatizadas. Esto garantiza la durabilidad y alta disponibilidad, en cumplimiento con el Acuerdo de Nivel de Servicio de Data Lake Storage Gen1. En este artículo, se proporciona una guía sobre cómo proteger aún más los datos en caso de las poco frecuentes interrupciones de toda la región o de eliminaciones accidentales.

## <a name="disaster-recovery-guidance"></a>Guía de recuperación ante desastres

Es fundamental preparar un plan de recuperación ante desastres. Revise la información de este artículo y estos recursos adicionales que le ayudarán a crear su propio plan.

* [Recuperación ante desastres y alta disponibilidad para aplicaciones de Azure](../resiliency/resiliency-disaster-recovery-high-availability-azure-applications.md)
* [Guía técnica sobre resistencia en Azure](../resiliency/resiliency-technical-guidance.md)

### <a name="best-practice-recommendations"></a>Mejores prácticas recomendadas

Se recomienda que copie los datos críticos en otra cuenta de Data Lake Storage Gen1 en otra región con una frecuencia que esté en consonancia con las necesidades de su plan de recuperación ante desastres. Hay una variedad de métodos para copiar datos entre los que se incluyen [ADLCopy](data-lake-store-copy-data-azure-storage-blob.md), [Azure PowerShell](data-lake-store-get-started-powershell.md) o [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md). Azure Data Factory es un servicio útil para crear e implementar las canalizaciones de movimiento de datos de forma periódica.

Si se produce una interrupción regional, puede acceder a sus datos en la región donde estos se copiaron. Puede supervisar el [panel de Azure Service Health](https://azure.microsoft.com/status/) para determinar el estado del servicio de Azure en todo el mundo.

## <a name="data-corruption-or-accidental-deletion-recovery-guidance"></a>Guía para la recuperación ante un problema de datos dañados o eliminación accidental

Tenga en cuenta que aunque Data Lake Storage Gen1 ofrece resistencia de datos mediante réplicas automatizadas, esto no evita que su aplicación (o los desarrolladores o usuarios) puedan dañar o eliminar de forma no intencionada los datos.

Para evitar la eliminación por error, se recomienda establecer primero las directivas de acceso correctas para su cuenta de Data Lake Storage Gen1. Esto incluye la aplicación de [bloqueos de recursos de Azure](../azure-resource-manager/management/lock-resources.md) para recursos importantes, así como la aplicación de control de acceso de nivel de cuenta y archivo mediante las [características de seguridad de Data Lake Storage Gen1](data-lake-store-security-overview.md) disponibles. También se recomienda que establezca una rutina de creación de copias de los datos críticos mediante [ADLCopy](data-lake-store-copy-data-azure-storage-blob.md), [Azure PowerShell](data-lake-store-get-started-powershell.md) o [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md) en otra cuenta de Data Lake Storage Gen1, otra carpeta o suscripción de Azure. Esto se puede usar para recuperar después de un incidente de daños o eliminación de datos. Azure Data Factory es un servicio útil para crear e implementar las canalizaciones de movimiento de datos de forma periódica.

También puede habilitar el [registro de diagnósticos](data-lake-store-diagnostic-logs.md) para una cuenta de Data Lake Storage Gen1 para recopilar registros de auditoría de acceso a datos. Los registros de auditoría proporcionan información sobre quién puede haber eliminado o actualizado un archivo.

Puede intentar restaurar un elemento eliminado mediante el módulo de Azure PowerShell para Data Lake Storage Gen 1 [Az.DataLakeStore](https://docs.microsoft.com/powershell/module/az.datalakestore/). En concreto, consulte el comando [Restore-AzDataLakeStoreDeletedItem](https://docs.microsoft.com/powershell/module/az.datalakestore/restore-azdatalakestoredeleteditem). Asegúrese de revisar la sección [Descripción](https://docs.microsoft.com/powershell/module/az.datalakestore/restore-azdatalakestoredeleteditem#description) antes de intentar utilizar este comando.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)
