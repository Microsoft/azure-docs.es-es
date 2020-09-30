---
title: Establecimiento de la contraseña de Azure Resource Manager en el dispositivo Azure Stack Edge Pro con GPU
description: En este artículo se explica cómo conectarse a la instancia de Azure Resource Manager que se ejecuta en Azure Stack Edge Pro con GPU a través de Azure PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: df5fea8101834dae089ab97354c438363321a707
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904485"
---
# <a name="set-azure-resource-manager-password-on-azure-stack-edge-pro-gpu-device"></a>Establecimiento de la contraseña de Azure Resource Manager en el dispositivo Azure Stack Edge Pro con GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este artículo se describe cómo establecer la contraseña de Azure Resource Manager. Debe establecer esta contraseña cuando se conecte a las API locales del dispositivo a través de Azure Resource Manager.

El procedimiento para establecer la contraseña puede variar en función de si usa Azure Portal o los cmdlets de PowerShell. En las secciones siguientes, se describe cada uno de estos procedimientos.


## <a name="reset-password-via-the-azure-portal"></a>Restablecimiento de la contraseña mediante Azure Portal

1. En Azure Portal, vaya al recurso de Azure Stack Edge que ha creado para administrar el dispositivo. Vaya a **Proceso perimetral > Comenzar**.

2. En el panel derecho, en la barra de comandos, seleccione **Restablecer contraseña de ARM de Edge**. 

    ![Restablecimiento de la contraseña de usuario de EdgeARM 1](media/azure-stack-edge-j-series-set-azure-resource-manager-password/set-edgearm-password-1.png)

3. En la hoja **Restablecer contraseña de ARM de Edge**, proporcione una contraseña para conectarse a las API locales del dispositivo a través de Azure Resource Manager. Confirme la contraseña y seleccione **Restablecer**.

    ![Restablecimiento de la contraseña de usuario de EdgeARM 2](media/azure-stack-edge-j-series-set-azure-resource-manager-password/set-edgearm-password-2.png)



## <a name="reset-password-via-powershell"></a>Restablecimiento de la contraseña mediante PowerShell

1. En Azure Portal, vaya al recurso de Azure Stack Edge que ha creado para administrar el dispositivo. Tome nota de los siguientes parámetros de la página **Información general**.

    - Nombre de recurso de Azure Stack Edge
    - Id. de suscripción

2. Vaya a **Configuración > Propiedades**. Tome nota de los siguientes parámetros del panel **Propiedades**.

    - Resource group
    - Clave de cifrado CIK: Seleccione la vista y copie la **Clave de cifrado**.

    ![Obtención de la clave de cifrado CIK](media/azure-stack-edge-j-series-set-azure-resource-manager-password/get-cik-portal.png)
 
3. Identifique la contraseña que utilizará para conectarse a Azure Resource Manager.

4. Inicie Cloud Shell. Seleccione el icono de la esquina superior derecha:

    ![Inicio de Cloud Shell](media/azure-stack-edge-j-series-set-azure-resource-manager-password/start-cloud-shell.png) 

    Una vez que se haya iniciado Cloud Shell, puede que tenga que cambiar a PowerShell.

    ![Cloud Shell](media/azure-stack-edge-j-series-set-azure-resource-manager-password/cloud-shell.png)   


5. Establecimiento del contexto. Escriba:

    `Set-AzContext -SubscriptionId <Subscription ID>`

    Este es una salida de ejemplo:

    
    ```azurepowershell
    PS Azure:\> Set-AzContext -SubscriptionId 8eb87630-972c-4c36-a270-f330e6c063df
    
        Name        Account   SubscriptionName   Environment  TenantId
        ----       -------    ----------------   -----------  --------
        DataBox_Edge_Test (8eb87630-972c-4c36-a… MSI@50342 DataBox_Edge_Tes AzureCloud           72f988bf-86f1-41af-91ab-2d7…
    
        PS Azure:/
    ```
    
5.  Si tiene algún módulo PS antiguo, deberá instalarlo.

    `Remove-Module  Az.DataBoxEdge -force`

    Esta es una salida de ejemplo. En este ejemplo, no había módulos antiguos para instalar.

    
    ```azurepowershell
        PS Azure:\> Remove-Module  Az.DataBoxEdge -force
        Remove-Module : No modules were removed. Verify that the specification of modules to remove is correct and those modules exist in the runspace.
        At line:1 char:1
        + Remove-Module  Az.DataBoxEdge -force
        + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : ResourceUnavailable: (:) [Remove-Module], InvalidOperationException
        + FullyQualifiedErrorId : Modules_NoModulesRemoved,Microsoft.PowerShell.Commands.RemoveModuleCommand
    
        PS Azure:\
    ```

6. El siguiente conjunto de comandos descargará y ejecutará un script para instalar los módulos de PowerShell.
    
    ```azurepowershell
        cd ~/clouddrive
        wget https://aka.ms/dbe-cmdlet-beta -O Az.DataBoxEdge.zip
        unzip ./Az.DataBoxEdge.zip
        Import-Module ~/clouddrive/Az.DataBoxEdge/Az.DataBoxEdge.psd1 -Force
    ```

7. En el siguiente conjunto de comandos, deberá proporcionar el nombre del recurso, el nombre del grupo de recursos, la clave de cifrado y la contraseña que identificó en el paso anterior.

    ```azurepowershell
    $devicename = "<Azure Stack Edge resource name>"
    $resourceGroup = "<Resource group name>"
    $cik = "<Encryption key>"
    $password = "<Password>"
    ```
    Los parámetros de clave de cifrado y contraseña deben pasarse como cadenas seguras. Use los siguientes cmdlets para convertir la contraseña y la clave de cifrado en cadenas seguras.

    ```azurepowershell
    $pass = ConvertTo-SecureString $password -AsPlainText -Force
    $key = ConvertTo-SecureString $cik -AsPlainText -Force
    ```
    Use las cadenas seguras generadas anteriormente como parámetros en el cmdlet Set-AzDataBoxEdgeUser para restablecer la contraseña. Use el mismo grupo de recursos que usó al crear el recurso de Azure Stack Edge Pro o Data Box Gateway.

    ```azurepowershell
    Set-AzDataBoxEdgeUser -ResourceGroupName $resourceGroup -DeviceName $devicename -Name EdgeARMUser  -Password $pass -EncryptionKey $key
    ```
    Esta es la salida de ejemplo.
    
    ```azurepowershell
    PS /home/aseuser/clouddrive> $devicename = "myaseresource"
    PS /home/aseuser/clouddrive> $resourceGroup = "myaserg"
    PS /home/aseuser/clouddrive> $cik = "54a7450fd7b3c506e10efea4e0c88a9390f37e299fbf43e01fb5dfe483ac036b6d0f85a6246e1926e297f98c0ff84c20a57348c689eff179ce31571fc787ac0a"
    PS /home/aseuser/clouddrive> $password = "Password2"
    PS /home/aseuser/clouddrive> $pass = ConvertTo-SecureString $password -AsPlainText -Force
    PS /home/aseuser/clouddrive> $key = ConvertTo-SecureString $cik -AsPlainText -Force
    PS /home/aseuser/clouddrive> Set-AzDataBoxEdgeUser -ResourceGroupName $resourceGroup -DeviceName $devicename -Name EdgeARMUser  -Password $pass -EncryptionKey $key
    
        User name   Type ResourceGroupName DeviceName
    ---------   ---- ----------------- ----------
        EdgeARMUser ARM  myaserg        myaseresource
    
        PS /home/aseuser/clouddrive>
    ```
Use la nueva contraseña para conectarse a Azure Resource Manager.

## <a name="next-steps"></a>Pasos siguientes

[Conexión a Azure Resource Manager](azure-stack-edge-j-series-connect-resource-manager.md)
