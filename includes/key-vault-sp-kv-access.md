---
author: msmbaldwin
ms.service: key-vault
ms.subservice: B2C
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: 826d5c2183ecc70908192695927890dcf06137b1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87013148"
---
Cree una directiva de acceso para su almacén de claves que conceda permiso a su entidad de servicio, para lo que se debe usar `clientId` en el comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy). Proporcione a la entidad de servicio los permisos get, list y set para las claves y los secretos.

```azurecli
az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions delete get list set --key-permissions create decrypt delete encrypt get list unwrapKey wrapKey
```
