---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 09/03/2020
ms.author: msmbaldwin
ms.openlocfilehash: 5e912b76c2ef68aa01dae57d1b42abc386a8e67b
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89482150"
---
1. Uso del comando `az group create` para crear un grupo de recursos:

    ```azurecli
    az group create --name KeyVault-PythonQS-rg --location eastus
    ```

    Si lo prefiere, puede cambiar "eastus" a una ubicación más próxima.

1. Use `az keyvault create` para crear el almacén de claves:

    ```azurecli
    az keyvault create --name <your-unique-keyvault-name> --resource-group KeyVault-PythonQS-rg
    ```

    Reemplace `<your-unique-keyvault-name>` por un nombre que sea único en todo Azure. Normalmente, se usa el nombre personal o de la empresa, junto con otros números e identificadores. 

1. Cree una variable de entorno que especifique el nombre del almacén de claves al código:

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```cmd
    set KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    export KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```

    ---
