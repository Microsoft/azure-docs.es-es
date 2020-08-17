---
title: Incorporación de conjuntos de datos a una instancia de Azure Data Share existente
description: Aprenda a agregar conjuntos de datos a un recurso compartido de datos existente en Azure Data Share con los mismos destinatarios.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 07/30/2020
ms.openlocfilehash: 540cdc2948cbcd97bf7c8872f3aace2862280434
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513496"
---
# <a name="how-to-add-datasets-to-an-existing-share-in-azure-data-share"></a>Incorporación de conjuntos de datos a un recurso compartido existente en Azure Data Share

En este artículo se explica cómo agregar conjuntos de datos a un recurso compartido de datos existente mediante Azure Data Share. Esto le permite compartir más datos con los mismos destinatarios sin tener que crear un nuevo recurso compartido.

Para información sobre cómo agregar conjuntos de datos a medida que crea un recurso compartido, consulte el tutorial [Uso compartido de datos](share-your-data.md).

## <a name="navigate-to-a-sent-data-share"></a>Desplazamiento a un recurso compartido de datos enviado

En Azure Data Share, vaya a su recurso compartido enviado y seleccione la pestaña **Conjuntos de datos**. Haga clic en el botón **+ Add Datasets** (Agregar conjuntos de datos) para agregar más conjuntos de valores.

![Incorporación de conjuntos de datos](./media/how-to/how-to-add-datasets/add-datasets.png)

En el panel de la derecha, seleccione el tipo de conjunto de datos que quiere agregar y haga clic en **Siguiente**. Seleccione la suscripción y el grupo de recursos de los datos que desea agregar. Con las flechas desplegables, busque y active la casilla situada junto a los datos que se van a agregar.

![Incorporación de conjuntos de datos](./media/how-to/how-to-add-datasets/add-datasets-side.png)

Una vez que haga clic en **Add Datasets** (Agregar conjuntos de datos), los conjuntos de datos se agregarán al recurso compartido. Nota: Los consumidores deben desencadenar una instantánea para que puedan ver los nuevos conjuntos de datos. Si hay configuraciones de instantánea definidas, los consumidores verán los nuevos conjuntos de datos una vez que se complete la siguiente instantánea programada. Sin una configuración de instantánea definida, el consumidor debe desencadenar manualmente una copia completa o incremental de los datos para recibir las actualizaciones. Para más información sobre las instantáneas, consulte [Instantáneas](terminology.md).

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre cómo [agregar destinatarios a un recurso compartido de datos existente](how-to-add-recipients.md).