---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: efa380ac243338a91354f948fdc77b2da8dd5406
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87512825"
---
Vamos a crear un secreto llamado **mySecret** cuyo valor sea **¡Correcto!** . Un secreto puede ser una contraseña, una cadena de conexión SQL o cualquier otra información que necesite mantener segura y disponible para la aplicación. 

Para agregar un secreto al almacén de claves recién creado, use el comando [az keyvault secret set](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-set) de la CLI de Azure:

```azurecli
az keyvault secret set --vault-name "<your-unique-keyvault-name>" --name "mySecret" --value "Success!"
```