---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 09/03/2020
ms.author: msmbaldwin
ms.openlocfilehash: 9440a33858cb70389e86621ae76be9c08ec56d61
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89482149"
---
1. Asegúrese de que tiene una [cuenta de Azure con una suscripción activa](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

1. Instale [Python 2.7+ o 3.5.3+](https://www.python.org/downloads).

1. Instale la [CLI de Azure](/cli/azure/install-azure-cli).

1. Siga las instrucciones que encontrará en el apartado sobre la [configuración de la autenticación para el desarrollo local](/azure/developer/python/configure-local-development-environment?tabs=cmd#configure-authentication), con las que crea una entidad de servicio local y la pone a disposición del cliente de Azure Key Vault para Python a través de variables de entorno. 

    Cuando se ejecuta código directamente en Azure, si la aplicación usa una identidad administrada no se necesita una entidad de servicio independiente.

1. En un símbolo del sistema o en un terminal, cree una carpeta de proyecto adecuada y, después, cree y active un entorno virtual de Python, como se describe en el apartado [Uso de entornos virtuales de Python](/azure/developer/python/configure-local-development-environment?tabs=cmd#use-python-virtual-environments)

1. Instale la biblioteca de identidades de Azure Active Directory:

    ```terminal
    pip install azure.identity
    ```
