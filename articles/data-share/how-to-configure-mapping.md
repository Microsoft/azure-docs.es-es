---
title: Configuración de una asignación de conjunto de datos en Azure Data Share
description: Aprenda a configurar una asignación de conjunto de datos para un recurso compartido recibido en Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 08/14/2020
ms.openlocfilehash: eed3e8275400a3e677df53b9d62cf0e0bc70271c
ms.sourcegitcommit: ef055468d1cb0de4433e1403d6617fede7f5d00e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2020
ms.locfileid: "88257832"
---
# <a name="how-to-configure-a-dataset-mapping-for-a-received-share-in-azure-data-share"></a>Configuración de una asignación de conjunto de datos para un recurso compartido recibido en Azure Data Share

En este artículo se explica cómo configurar una asignación de conjunto de datos para un recurso compartido recibido en Azure Data Share. Es posible configurar una asignación de conjunto de datos para especificar un almacén de datos de destino en el que quiere recibir datos o si necesita cambiar el almacén de datos de destino.

## <a name="navigate-to-a-received-data-share"></a>Vaya a un recurso compartido de datos recibido.

En el servicio Azure Data Share, vaya al recurso compartido recibido y seleccione la pestaña **Conjuntos de datos**. 

![Asignación de conjuntos de datos](./media/dataset-mapping.png "Asignación de conjuntos de datos") 

Active la casilla situada junto al conjunto de datos al que desearía asignar un destino. Seleccione **+ Map to target** (+ Asignar a destino) para elegir un nuevo almacén de destino. Seleccione **Desasociar** primero si el conjunto de datos ya está asignado y quiere cambiar el almacén de datos de destino.

![Asignación a destino](./media/dataset-map-target.png "Asignar a destino") 

## <a name="select-a-target-store"></a>Selección de un almacén de destino

Seleccione un tipo de datos de destino en el que quiera colocar los datos. En el uso compartido basado en instantáneas, todos los datos que ya existan en cualquier cuenta de almacenamiento ya asignada no se moverán automáticamente al almacén de destino nuevo. Para el uso compartido en contexto, seleccione un almacén de datos en la ubicación especificada. La ubicación es el centro de datos de Azure donde se encuentra el almacén de datos de origen del proveedor de datos.

![Cuenta de almacenamiento de destino](./media/dataset-map-target-sql.png "Almacenamiento de destino") 

## <a name="select-a-file-format-sql-sources-only"></a>Selección de un formato de archivo (solo orígenes de SQL)

Si los datos de origen proceden de un origen basado en SQL y quiere recibirlos como un archivo, puede elegir el formato en el que se reciben. 

![Elegir formato](./media/sql-file-formats.png "Formatos de archivo SQL")

## <a name="next-steps"></a>Pasos siguientes

Para obtener información acerca de cómo empezar a compartir datos, vaya al tutorial que cubre cómo [compartir sus datos](share-your-data.md).



