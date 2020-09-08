---
author: IngridAtMicrosoft
ms.service: media-services
ms.topic: include
ms.date: 08/17/2020
ms.author: inhenkel
ms.custom: CLI
ms.openlocfilehash: aa76f7b85302651f6874747610c3355f0572a7ee
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88556176"
---
<!-- List and set subscriptions -->

1. Obtenga la lista de suscripciones con el comando [az account list](/cli/azure/account#az-account-list):

    ```
    az account list --output table
    ```

2. Use `az account set` con el identificador de suscripción o el nombre al que desee cambiar.

    ```
    az account set --subscription "My Demos"
    ```
