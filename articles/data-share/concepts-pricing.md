---
title: Información sobre los precios de Azure Data Share
description: Obtenga información sobre cómo funcionan los precios de Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: conceptual
ms.date: 08/11/2020
ms.openlocfilehash: 1c2c58e206a70a3801c712df3b5582409712d3c8
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136687"
---
# <a name="understand-azure-data-share-pricing"></a>Información sobre los precios de Azure Data Share

Para el uso compartido basado en instantáneas, Azure Data Share se cobra por instantánea del conjunto de datos y ejecución de instantáneas. En este artículo se explica cómo calcular la instantánea del conjunto de datos y la ejecución de instantáneas mediante la información del historial de instantáneas. Actualmente, el proveedor de datos se factura por instantánea del conjunto de datos y ejecución de instantáneas.

## <a name="dataset-snapshot"></a>Instantánea del conjunto de datos

Una instantánea del conjunto de datos es la operación para mover el conjunto de datos desde su origen hasta un destino. Cuando se realiza una instantánea de un recurso compartido, la instantánea de cada conjunto de datos dentro del recurso compartido es una operación de instantánea del conjunto de datos. Siga los pasos que se indican a continuación para ver una lista de instantáneas del conjunto de datos. 

1. En Azure Portal, navegue hasta el recurso de Data Share.

1. Seleccione un recurso compartido enviado o recibido.

1. Haga clic en la pestaña **Historial**.

1. Haga clic en la hora de inicio de una instantánea.
 
    ![Historial de instantáneas](./media/concepts/concepts-pricing/pricing-snapshot-history.png "Historial de instantáneas") 

1. Vea la lista de operaciones de instantáneas del conjunto de datos. Cada elemento de línea corresponde a una operación de instantánea del conjunto de datos. En este ejemplo, hay dos instantáneas del conjunto de datos.

    ![Operación de instantánea del conjunto de datos](./media/concepts/concepts-pricing/pricing-dataset-snapshot.png "Operación de instantánea del conjunto de datos")

## <a name="snapshot-execution"></a>Ejecución de instantáneas

La ejecución de instantáneas incluye los recursos necesarios para trasladar un conjunto de datos desde el origen al destino. Para cada operación de instantánea del conjunto de datos, la ejecución de instantáneas se calcula como el número de núcleos virtuales multiplicado por la duración de la instantánea. Los cargos se prorratean por minuto y se redondean. El número de núcleo virtual se selecciona de forma dinámica en función del par de origen y destino y del patrón de datos. Siga los pasos que se indican a continuación para ver la hora de inicio y finalización de la instantánea y los núcleos virtuales utilizados para una operación de instantánea del conjunto de datos.

1. En Azure Portal, navegue hasta el recurso de Data Share.

1. Seleccione un recurso compartido enviado o recibido.

1. Haga clic en la pestaña **Historial**.

1. Haga clic en la hora de inicio de una instantánea.

    ![Historial de instantáneas](./media/concepts/concepts-pricing/pricing-snapshot-history.png "Historial de instantáneas") 

1. Haga clic en el estado de una operación de instantánea del conjunto de datos.

    ![Estado de la instantánea del conjunto de datos](./media/concepts/concepts-pricing/pricing-snapshot-status.png "Estado de la instantánea del conjunto de datos")

1. Vea la hora de inicio y finalización y los núcleos virtuales utilizados para esta operación de instantánea del conjunto de datos. 

    ![Ejecución de instantáneas](./media/concepts/concepts-pricing/pricing-snapshot-execution.png "Ejecución de instantáneas")

## <a name="next-steps"></a>Pasos siguientes

- Obtenga la información más actualizada sobre los [precios de Azure Data Share](https://azure.microsoft.com/pricing/details/data-share/).
- Use la [calculadora de preciso de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular el costo.
