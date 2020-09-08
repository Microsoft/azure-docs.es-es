---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: d567a4f7d9b9429887d6396cb2b03dfc34c6ac93
ms.sourcegitcommit: 37afde27ac137ab2e675b2b0492559287822fded
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88602354"
---
<!-- Create a resource group -->

Use el siguiente comando para crear un grupo de recursos. Seleccione la región geográfica que se usará para almacenar los registros de elementos multimedia y de metadatos para la cuenta de Media Services. Esta región se utilizará para procesar y transmitir contenido multimedia.

```azurecli
az group create --name amsResourceGroup --location westus2
```
