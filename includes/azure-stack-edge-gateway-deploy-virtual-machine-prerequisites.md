---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/31/2020
ms.author: alkohli
ms.openlocfilehash: 14d1e466a7e84ff3454284c596a09ebcabf9d603
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89303937"
---
Para poder implementar VM en el dispositivo Azure Stack Edge, debe configurar el cliente para que se conecte al dispositivo mediante Azure Resource Manager con Azure PowerShell. Para consultar los pasos detallados, vaya a [Conectar con Azure Resource Manager desde el dispositivo Azure Stack Edge](../articles/databox-online/azure-stack-edge-j-series-connect-resource-manager.md).


Asegúrese de que se pueden usar los siguientes pasos para acceder al dispositivo desde el cliente (completó esta configuración al conectarse a Azure Resource Manager; ahora solo está comprobando que se realizó correctamente): 

1. Compruebe que la comunicación de Azure Resource Manager funciona. Escriba:     

    ```powershell
    Add-AzureRmEnvironment -Name <Environment Name> -ARMEndpoint "https://management.<appliance name>.<DNSDomain>"
    ```

1. Llame a las API del dispositivo local para autenticarse. Escriba: 

    `login-AzureRMAccount -EnvironmentName <Environment Name>`

    Proporcione el nombre de usuario *EdgeARMuser* y la contraseña para conectarse mediante Azure Resource Manager.

1. Si configuró **Proceso** para Kubernetes, puede omitir este paso. Continúe para asegurarse de que ha habilitado una interfaz de red para el proceso. En la interfaz de usuario local, vaya a la configuración de **Proceso**. Seleccione la interfaz de red que usará para crear un conmutador virtual. Las VM que cree se adjuntarán a un conmutador virtual conectado a este puerto y a la red asociada. Asegúrese de elegir una red que coincida con la dirección IP estática que usará para la VM.  

    ![Habilitar la configuración del proceso 1](../articles/databox-online/media/azure-stack-edge-gpu-deploy-virtual-machine-templates/enable-compute-setting.png)

    Habilite el proceso en la interfaz de red. Azure Stack Edge creará y administrará un conmutador virtual correspondiente a esa interfaz de red. No especifique direcciones IP específicas para Kubernetes en este momento. La habilitación del proceso puede tardar varios minutos.

    <!--If you decide to use another network interface for compute, make sure that you:
    
    - Delete all the VMs that you have deployed using Azure Resource Manager.
    
    - Delete all virtual network interfaces and the virtual network associated with this network interface. 
    
    - You can now enable another network interface for compute.-->

<!--1. You may also need to configure TLS 1.2 on your client machine if running older versions of AzCopy.--> 

