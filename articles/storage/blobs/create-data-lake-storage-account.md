---
title: Creación de una cuenta de almacenamiento para Azure Data Lake Storage Gen2
description: Obtenga información sobre cómo crear una cuenta de almacenamiento para su uso con Azure Data Lake Storage Gen2.
author: normesta
ms.topic: how-to
ms.author: normesta
ms.date: 08/31/2020
ms.service: storage
ms.reviewer: stewu
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: 6b63933fc625079bb490942cf3a32232a484fe38
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269934"
---
# <a name="create-a-storage-account-to-use-with-azure-data-lake-storage-gen2"></a>Creación de una cuenta de almacenamiento para su uso con Azure Data Lake Storage Gen2 habilitado

Para usar las funcionalidades de Data Lake Storage Gen2, cree una cuenta de almacenamiento que tenga un espacio de nombres jerárquico.

## <a name="choose-a-storage-account-type"></a>Selección de un tipo de cuenta de almacenamiento

Las funcionalidades de Data Lake Storage se admiten en los siguientes tipos de cuentas de almacenamiento:

- Uso general v2
- BlockBlobStorage

Para más información sobre cómo elegir entre ellos, consulte la [introducción a la cuenta de almacenamiento](../common/storage-account-overview.md).

## <a name="create-a-storage-account-with-a-hierarchical-namespace"></a>Creación de una cuenta de almacenamiento con un espacio de nombres jerárquico

Cree una [cuenta de uso general V2](../common/storage-account-create.md) o una cuenta de [BlockBlobStorage](storage-blob-create-account-block-blob.md) con la configuración **Espacio de nombres jerárquico** habilitada.

Desbloquee las funcionalidades de Data Lake Storage al crear la cuenta mediante la habilitación de la opción **Espacio de nombres jerárquico** en la pestaña **Avanzadas** de la página **Crear cuenta de almacenamiento**. Debe habilitar esta configuración al crear la cuenta. No se puede habilitar después.

En la siguiente imagen se muestra esta configuración en la página **Crear cuenta de almacenamiento**.

> [!div class="mx-imgBorder"]
> ![Opción Servicio de espacio de nombres jerárquico](./media/create-data-lake-storage-account/hierarchical-namespace-feature.png)

Si tiene una cuenta de almacenamiento existente que quiere usar con Data Lake Storage y la opción Espacio de nombres jerárquico está deshabilitada, debe migrar los datos a una nueva cuenta de almacenamiento que tenga habilitada esta opción.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las cuentas de almacenamiento](../common/storage-account-overview.md)
- [Uso de Azure Data Lake Storage Gen2 para requisitos de macrodatos](data-lake-storage-data-scenarios.md)
- [Control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)