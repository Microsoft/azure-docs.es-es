---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/09/2021
ms.author: alkohli
ms.openlocfilehash: 1298673c475a0308f1db27641aae93837e6b5df3
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305574"
---
Asegúrese de que se pueden usar los pasos siguientes para acceder al dispositivo desde el cliente.

Compruebe que el cliente puede conectarse a Azure Resource Manager local. 

1. Llame a las API del dispositivo local para autenticarse:

    ```powershell
    login-AzureRMAccount -EnvironmentName <Environment Name> -TenantId c0257de7-538f-415c-993a-1b87a031879d  
    ```

1. Proporcione el nombre de usuario `EdgeArmUser` y la contraseña para conectarse mediante Azure Resource Manager. Si no recuerda la contraseña, consulte [Restablecimiento de la contraseña de Azure Resource Manager](../articles/databox-online/azure-stack-edge-gpu-set-azure-resource-manager-password.md) y use esta contraseña para iniciar la sesión.