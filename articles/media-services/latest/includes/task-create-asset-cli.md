---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/18/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: 9f079be76d4d75f05b4a8e132ac425255eed8558
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605964"
---
<!--Create a media services asset CLI-->

El siguiente comando de la CLI de Azure crea un nuevo recurso de Media Services. Reemplace los valores `assetName`, `amsAccountName` y `resourceGroup` por los valores con los que esté trabajando actualmente.

```azurecli
az ams asset create -n assetName -a amsAccountName -g resourceGroup
```
