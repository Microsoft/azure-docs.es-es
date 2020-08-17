---
title: 'Creación de un emparejamiento de red virtual de Azure: modelos de implementación diferentes: misma suscripción'
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo crear un emparejamiento de redes virtuales entre redes virtuales creadas mediante diferentes modelos de implementación de Azure que existen en diferentes suscripciones de Azure.
services: virtual-network
documentationcenter: ''
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2020
ms.author: kumud
ms.reviewer: anavin
ms.openlocfilehash: 88e576231e0231a105cd9ec303f63307b5eaff89
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051628"
---
# <a name="create-a-virtual-network-peering---different-deployment-models-and-subscriptions"></a>Crear un emparejamiento de redes virtuales de Azure: diferentes modelos de implementación y suscripciones

En este tutorial aprenderá a crear un emparejamiento de redes virtuales entre dos redes virtuales creadas mediante diferentes modelos de implementación. Las redes virtuales se encuentran en suscripciones diferentes. Emparejar dos redes virtuales permite que los recursos en distintas redes virtuales se comuniquen entre sí con el mismo ancho de banda y latencia que tendrían los recursos si estuvieran en la misma red virtual. Obtenga más información sobre el [Emparejamiento de redes virtuales](virtual-network-peering-overview.md).

Los pasos para crear un emparejamiento de redes virtuales cambian en función de si las redes virtuales se encuentran en la misma suscripción o en suscripciones diferentes, y en función del [modelo de implementación de Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json) con el que se han creado las redes virtuales. Para más información sobre cómo crear un emparejamiento de redes virtuales en otros escenarios, haga clic en el escenario en la tabla siguiente:

|Modelo de implementación de Azure  | Suscripción de Azure  |
|--------- |---------|
|[Ambas mediante Resource Manager](tutorial-connect-virtual-networks-portal.md) |Iguales|
|[Ambas mediante Resource Manager](create-peering-different-subscriptions.md) |Diferentes|
|[Una mediante Resource Manager y la otra clásica](create-peering-different-deployment-models.md) |Iguales|

No se puede crear un emparejamiento de redes virtuales entre dos redes virtuales implementadas mediante el modelo de implementación clásico. En este tutorial se usan las redes virtuales de la misma región. En este tutorial se emparejan redes virtuales de la misma región. También puede emparejar redes virtuales en distintas [regiones compatibles](virtual-network-manage-peering.md#cross-region). Se recomienda que se familiarice con los [requisitos y restricciones de emparejamiento](virtual-network-manage-peering.md#requirements-and-constraints) antes de emparejar redes virtuales.

Al crear un emparejamiento de redes virtuales entre redes virtuales que se encuentran en suscripciones diferentes, las suscripciones pueden estar asociadas al mismo inquilino de Azure Active Directory. Si todavía no tiene un inquilino de Azure Active Directory, puede [crear uno](../active-directory/develop/quickstart-create-new-tenant.md?toc=%2fazure%2fvirtual-network%2ftoc.json#create-a-new-azure-ad-tenant) rápidamente.

Puede usar [Azure Portal](#portal), la [interfaz de la línea de comandos](#cli) (CLI) de Azure o Azure [PowerShell](#powershell) para crear un emparejamiento de redes virtuales. Haga clic en cualquiera de los vínculos anteriores de herramientas para ir directamente a los pasos para crear un emparejamiento de redes virtuales con la herramienta de su preferencia.

## <a name="create-peering---azure-portal"></a><a name="portal"></a>Creación de emparejamiento: Azure Portal

En este tutorial se usan cuentas diferentes para cada suscripción. Si está usando una cuenta que tiene permisos para ambas suscripciones puede usar la misma cuenta para todos los pasos, y omitir los pasos para cerrar sesión en el portal y para asignar a otro usuario permisos para las redes virtuales.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como UserA. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
2. Haga clic en **+ Nuevo**, **Redes** y, luego, en **Red virtual**.
3. En la hoja **Crear red virtual**, escriba o seleccione valores para la configuración siguiente y, luego, haga clic en **Crear**:
    - **Nombre**: *myVnetA*
    - **Espacio de direcciones**: *10.0.0.0/16*
    - **Nombre de subred**: *default*
    - **Rango de direcciones de subred**: *10.0.0.0/24*
    - **Suscripción**: Seleccione la suscripción A.
    - **Grupo de recursos**: seleccione **Crear nuevo** y escriba *myResourceGroupA*.
    - **Ubicación**: *Este de EE. UU.*
4. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetA*. Haga clic en **myVnetA** cuando aparezca en los resultados de la búsqueda. Aparece una hoja para la red virtual **myVnetA**.
5. En la hoja **myVnetA** que aparece, haga clic en **Control de acceso (IAM)** en la lista vertical de opciones que aparece a la izquierda de la hoja.
6. En la hoja **myVnetA - Control de acceso (IAM)** que aparece, haga clic en **+ Agregar asignación de roles**.
7. En la hoja **Agregar asignación de roles** que aparece, seleccione **Colaborador de la red** en el cuadro **Rol**.
8. En el cuadro **Seleccionar**, seleccione otro usuario (UserB) o escriba la dirección de correo de UserB para buscarlo. La lista de usuario que se muestra proviene del mismo inquilino de Azure Active Directory que la red virtual para la que configura el emparejamiento. Haga clic en UserB cuando aparezca en la lista.
9. Haga clic en **Save**(Guardar).
10. Cierre sesión en el portal como UserA e inicie sesión como UserB.
11. Haga clic en **+ Nuevo**, escriba *Red virtual* en el cuadro **Buscar en el Marketplace** y después haga clic en **Red virtual** en los resultados de la búsqueda.
12. En la hoja **Virtual Network** que aparece, seleccione **Clásica** en el cuadro **Seleccionar un modelo de implementación** y haga clic en **Crear**.
13. En el cuadro Crear red virtual (clásica) que aparece, escriba los valores siguientes:

    - **Nombre**: *myVnetB*
    - **Espacio de direcciones**: *10.1.0.0/16*
    - **Nombre de subred**: *default*
    - **Rango de direcciones de subred**: *10.1.0.0/24*
    - **Suscripción**: Seleccione la suscripción B.
    - **Grupo de recursos**: haga clic en **Crear nuevo** y escriba *myResourceGroupB*.
    - **Ubicación**: *Este de EE. UU.*

14. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetB*. Haga clic en **myVnetB** cuando aparezca en los resultados de la búsqueda. Aparece una hoja para la red virtual **myVnetB**.
15. En la hoja **myVnetB** que aparece, haga clic en **Propiedades** en la lista vertical de opciones que aparece a la izquierda de la hoja. Copie el **ID. DE RECURSO**, ya que se usará en un paso posterior. El id. de recurso es similar al del siguiente ejemplo: `/subscriptions/<Subscription ID>/resourceGroups/myResourceGroupB/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB`
16. Complete los pasos 5 a 9 para myVnetB y escriba **UserA** en el paso 8.
17. Cierre sesión en el portal como UserB e inicie sesión como UserA.
18. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetA*. Haga clic en **myVnetA** cuando aparezca en los resultados de la búsqueda. Aparece una hoja para la red virtual **myVnet**.
19. Haga clic en **myVnetA**.
20. En la hoja **myVnetA** que aparece, haga clic en **Emparejamientos** en la lista vertical de opciones que aparece a la izquierda de la hoja.
21. En la hoja **myVnetA - Emparejamientos** que aparece, haga clic en **+ Agregar**.
22. En la hoja **Agregar emparejamiento** que aparece, escriba o seleccione las opciones siguientes y, luego, haga clic en **Aceptar**:
     - **Nombre**: *myVnetAToMyVnetB*
     - **Modelo de implementación de red virtual**:  Seleccione **Clásico**.
     - **Conozco mi Id. de recurso**: Active esta casilla.
     - **Identificador de recurso**: escriba el identificador de recurso de myVnetB del paso 15.
     - **Permitir acceso a red virtual**: asegúrese de que está seleccionada la opción **Habilitado**.
    En este tutorial no se usa ninguna otra configuración. Para conocer todas las configuraciones de emparejamiento, lea [Manage virtual network peerings](virtual-network-manage-peering.md#create-a-peering) (Administración de emparejamientos de redes virtuales).
23. Una vez que haga clic en **Aceptar** en el paso anterior, se cerrará la hoja **Agregar emparejamiento** y se volverá a mostrar la hoja **myVnetA - Emparejamientos**. Unos segundos después, el emparejamiento que creó aparece en la hoja. El estado **Conectado** aparece en la columna **ESTADO DE EMPAREJAMIENTO** correspondiente al emparejamiento **myVnetAToMyVnetB** que ha creado. El emparejamiento está ahora establecido. No es necesario emparejar la red virtual (clásica) a la red virtual (Resource Manager).

    Los recursos de Azure que cree en cualquiera de las redes virtuales ahora se pueden comunicar entre sí mediante sus direcciones IP. Si usa la resolución de nombres predeterminada de Azure para las redes virtuales, los recursos de las redes virtuales no pueden resolver nombres entre las redes virtuales. Si desea resolver nombres entre las redes virtuales de un emparejamiento, debe crear su propio servidor DNS. Obtenga información sobre cómo configurar la [resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

24. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
25. **Opcional**: para eliminar los recursos que crea en este tutorial, complete los pasos que aparecen en la sección [Eliminación de recursos](#delete-portal) de este artículo.

## <a name="create-peering---azure-cli"></a><a name="cli"></a>Creación de emparejamiento: CLI de Azure

En este tutorial se usan cuentas diferentes para cada suscripción. Si está usando una cuenta que tiene permisos para ambas suscripciones, puede usar la misma cuenta para todos los pasos, omitir los pasos para cerrar sesión en Azure y quitar las líneas del script que crean las asignaciones de roles de usuario. Reemplace UserA@azure.com y UserB@azure.com en todos los scripts siguientes por los nombres de usuario que está usando para UserA y UserB. Complete los pasos siguientes mediante la CLI de Azure clásica y la CLI de Azure. Puede completar los pasos desde Azure Cloud Shell. Para ello solo tiene que seleccionar el botón **Try it** (Pruébelo) en cualquiera de los pasos siguientes o instalar la [CLI clásica](/cli/azure/install-classic-cli) y la [CLI](/cli/azure/install-azure-cli), y ejecutar los comandos en el equipo local.

1. Si usa Cloud Shell, vaya al paso 2, ya que Cloud Shell inicia automáticamente la sesión de Azure. Abra una sesión de comandos e inicie sesión en Azure mediante el comando `azure login`.
2. Ejecute la CLI clásica en modo de Administración de servicios escribiendo el comando `azure config mode asm`.
3. Escriba el siguiente comando de la CLI clásica para crear la red virtual (clásica):

    ```console
    azure network vnet create --vnet myVnetB --address-space 10.1.0.0 --cidr 16 --location "East US"
    ```

4. Los pasos restantes tienen que realizarse mediante un shell de bash con la CLI de Azure (no la CLI clásica).
5. Copie el script siguiente en un editor de texto del equipo. Reemplace `<SubscriptionB-Id>` con el Id. de suscripción. Si no conoce el Id. de suscripción, escriba el comando `az account show`. El valor de **id** en la salida es el identificador de la suscripción. Copie el script modificado, péguelo en la sesión de la CLI y, después, presione `Enter`.

    ```azurecli-interactive
    az role assignment create \
      --assignee UserA@azure.com \
      --role "Classic Network Contributor" \
      --scope /subscriptions/<SubscriptionB-Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB
    ```

    Cuando creó la red virtual (clásica) en el paso 4, Azure creó la red virtual en el grupo de recursos *Default-Networking*.
6. Cierre sesión en Azure como UserB e inicie sesión como UserA en la CLI.
7. Cree un grupo de recursos y una red virtual (Resource Manager). Copie el script siguiente, péguelo en la sesión de la CLI y después presione `Enter`.

    ```azurecli-interactive
    #!/bin/bash

    # Variables for common values used throughout the script.
    rgName="myResourceGroupA"
    location="eastus"

    # Create a resource group.
    az group create \
      --name $rgName \
      --location $location

    # Create virtual network A (Resource Manager).
    az network vnet create \
      --name myVnetA \
      --resource-group $rgName \
      --location $location \
      --address-prefix 10.0.0.0/16

    # Get the id for myVnetA.
    vNetAId=$(az network vnet show \
      --resource-group $rgName \
      --name myVnetA \
      --query id --out tsv)

    # Assign UserB permissions to myVnetA.
    az role assignment create \
      --assignee UserB@azure.com \
      --role "Network Contributor" \
      --scope $vNetAId
    ```

8. Cree un emparejamiento de redes virtuales entre las dos redes virtuales creadas mediante los modelos de implementación diferentes. Copie el script siguiente en un editor de texto del equipo. Reemplace `<SubscriptionB-id>` con el Id. de suscripción. Si no conoce el Id. de suscripción, escriba el comando `az account show`. El valor de **id** en la salida es el identificador de la suscripción. Azure creó la red virtual (clásica) que creó en el paso 4 en un grupo de recursos denominado *Default-Networking*. Pegue el script modificado en la sesión de CLI y después presione `Enter`.

    ```azurecli-interactive
    # Peer VNet1 to VNet2.
    az network vnet peering create \
      --name myVnetAToMyVnetB \
      --resource-group $rgName \
      --vnet-name myVnetA \
      --remote-vnet-id  /subscriptions/<SubscriptionB-id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB \
      --allow-vnet-access
    ```

9. Una vez que se ejecute el script, revise los emparejamientos de la red virtual (Resource Manager). Copie el script siguiente y péguelo en la sesión de la CLI:

    ```azurecli-interactive
    az network vnet peering list \
      --resource-group $rgName \
      --vnet-name myVnetA \
      --output table
    ```

    La salida muestra **Conectado** en la columna **PeeringState**.

    Los recursos de Azure que cree en cualquiera de las redes virtuales ahora se pueden comunicar entre sí mediante sus direcciones IP. Si usa la resolución de nombres predeterminada de Azure para las redes virtuales, los recursos de las redes virtuales no pueden resolver nombres entre las redes virtuales. Si desea resolver nombres entre las redes virtuales de un emparejamiento, debe crear su propio servidor DNS. Obtenga información sobre cómo configurar la [resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

10. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
11. **Opcional**: para eliminar los recursos creados en este tutorial, complete los pasos de la sección [Eliminar recursos](#delete-cli) de este artículo.

## <a name="create-peering---powershell"></a><a name="powershell"></a>Creación de emparejamiento: PowerShell

En este tutorial se usan cuentas diferentes para cada suscripción. Si está usando una cuenta que tiene permisos para ambas suscripciones, puede usar la misma cuenta para todos los pasos, omitir los pasos para cerrar sesión en Azure y quitar las líneas del script que crean las asignaciones de roles de usuario. Reemplace UserA@azure.com y UserB@azure.com en todos los scripts siguientes por los nombres de usuario que está usando para UserA y UserB. 

1. Instale la versión más reciente de los módulos [Azure](https://www.powershellgallery.com/packages/Azure) y [Az](https://www.powershellgallery.com/packages/Az) de PowerShell. Si no está familiarizado con Azure PowerShell, consulte [Introducción a Azure PowerShell](/powershell/azure/?toc=%2fazure%2fvirtual-network%2ftoc.json).
2. Inicie una sesión de PowerShell.
3. En PowerShell, inicie sesión en la suscripción de UserB como UserB escribiendo el comando `Add-AzureAccount`. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
4. Para crear una red virtual (clásica) con PowerShell, debe crear o modificar un archivo de configuración de red existente. Obtenga información sobre cómo [exportar, actualizar e importar archivos de configuración de red](virtual-networks-using-network-configuration-file.md). El archivo debe incluir el siguiente elemento **VirtualNetworkSite** para la red virtual que se usa en este tutorial:

    ```xml
    <VirtualNetworkSite name="myVnetB" Location="East US">
      <AddressSpace>
        <AddressPrefix>10.1.0.0/16</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="default">
          <AddressPrefix>10.1.0.0/24</AddressPrefix>
        </Subnet>
      </Subnets>
    </VirtualNetworkSite>
    ```

    > [!WARNING]
    > Importar un archivo de configuración de red modificada puede producir cambios en las redes virtuales (clásicas) existentes en la suscripción. Asegúrese de agregar solo la red virtual anterior y que no cambia o quita ninguna red virtual existente de la suscripción. 

5. Inicie sesión en la suscripción de UserB como UserB para usar los comandos de Resource Manager escribiendo el comando `Connect-AzAccount`.
6. Asigne los permisos de UserA a la red virtual B. Copie el script siguiente en un editor de texto en su PC y reemplace `<SubscriptionB-id>` con el Id. de suscripción B. Si no conoce el Id. de suscripción, escriba el comando `Get-AzSubscription` para verlo. El valor de **id** en la salida devuelta es el identificador de la suscripción. Azure creó la red virtual (clásica) que creó en el paso 4 en un grupo de recursos denominado *Default-Networking*. Para ejecutar el script, copie el script modificado, péguelo en PowerShell y pulse `Enter`.

    ```powershell
    New-AzRoleAssignment `
      -SignInName UserA@azure.com `
      -RoleDefinitionName "Classic Network Contributor" `
      -Scope /subscriptions/<SubscriptionB-id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB
    ```

7. Cierre sesión en Azure como UserB e inicie sesión en la suscripción de UserA como UserA escribiendo el comando `Connect-AzAccount`. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
8. Para crear la red virtual (Resource Manager) copie el siguiente script, péguelo en PowerShell y después presione `Enter`:

    ```powershell
    # Variables for common values
      $rgName='MyResourceGroupA'
      $location='eastus'

    # Create a resource group.
    New-AzResourceGroup `
      -Name $rgName `
      -Location $location

    # Create virtual network A.
    $vnetA = New-AzVirtualNetwork `
      -ResourceGroupName $rgName `
      -Name 'myVnetA' `
      -AddressPrefix '10.0.0.0/16' `
      -Location $location
    ```

9. Asigne los permisos de UserB a myVnetA. Copie el script siguiente en un editor de texto en su PC y reemplace `<SubscriptionA-Id>` con el Id. de la suscripción A. Si no conoce el Id. de la suscripción, escriba el comando `Get-AzSubscription` para verlo. El valor de **id** en la salida devuelta es el identificador de la suscripción. Pegue la versión modificada del script en PowerShell y después presione `Enter` para ejecutarlo.

    ```powershell
    New-AzRoleAssignment `
      -SignInName UserB@azure.com `
      -RoleDefinitionName "Network Contributor" `
      -Scope /subscriptions/<SubscriptionA-Id>/resourceGroups/myResourceGroupA/providers/Microsoft.Network/VirtualNetworks/myVnetA
    ```

10. Copie el script siguiente en un editor de texto de su equipo y reemplace `<SubscriptionB-id>` por el identificador de la suscripción B. Para emparejar myVnetA con myVNetB, copie el script modificado, péguelo en PowerShell y después presione `Enter`.

    ```powershell
    Add-AzVirtualNetworkPeering `
      -Name 'myVnetAToMyVnetB' `
      -VirtualNetwork $vnetA `
      -RemoteVirtualNetworkId /subscriptions/<SubscriptionB-id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB
    ```

11. Para ver el estado de emparejamiento de myVnetA, copie el siguiente script, péguelo en PowerShell y presione `Enter`.

    ```powershell
    Get-AzVirtualNetworkPeering `
      -ResourceGroupName $rgName `
      -VirtualNetworkName myVnetA `
      | Format-Table VirtualNetworkName, PeeringState
    ```

    El estado es **Conectado**. Cuando haya configurado el emparejamiento a myVnetA desde myVnetB, cambiará a **Conectado**.

    Los recursos de Azure que cree en cualquiera de las redes virtuales ahora se pueden comunicar entre sí mediante sus direcciones IP. Si usa la resolución de nombres predeterminada de Azure para las redes virtuales, los recursos de las redes virtuales no pueden resolver nombres entre las redes virtuales. Si desea resolver nombres entre las redes virtuales de un emparejamiento, debe crear su propio servidor DNS. Obtenga información sobre cómo configurar la [resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

12. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
13. **Opcional**: para eliminar los recursos creados en este tutorial, complete los pasos de la sección [Eliminar recursos](#delete-powershell) de este artículo.

## <a name="delete-resources"></a><a name="delete"></a>Eliminación de recursos
Cuando haya terminado este tutorial, es posible que quiera eliminar los recursos que creó en el tutorial, para no incurrir en gastos de uso. Al eliminar un grupo de recursos se eliminan también todos los recursos contenidos en el mismo.

### <a name="azure-portal"></a><a name="delete-portal"></a>Azure Portal

1. En el cuadro de búsqueda del portal, escriba **myResourceGroupA**. En los resultados de la búsqueda, haga clic en **myResourceGroupA**.
2. En la hoja **myResourceGroupA**, haga clic en el icono **Eliminar**.
3. Para confirmar la eliminación, en el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS**, escriba **myResourceGroupA** y, luego, haga clic en **Eliminar**.
4. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetB*. Haga clic en **myVnetB** cuando aparezca en los resultados de la búsqueda. Aparece una hoja para la red virtual **myVnetB**.
5. En la hoja **myVnetB**, haga clic en **Eliminar**.
6. Para confirmar la eliminación, haga clic en **Sí** en el cuadro **Eliminar red virtual**.

### <a name="azure-cli"></a><a name="delete-cli"></a>Azure CLI

1. Inicie sesión en Azure con la CLI para eliminar la red virtual (Resource Manager) con el siguiente comando:

   ```azurecli-interactive
   az group delete --name myResourceGroupA --yes
   ```

2. Inicie sesión en Azure con la CLI clásica para eliminar la red virtual (clásica) con los siguientes comandos:

   ```console
   azure config mode asm

   azure network vnet delete --vnet myVnetB --quiet
   ```

### <a name="powershell"></a><a name="delete-powershell"></a>PowerShell

1. En el símbolo del sistema de PowerShell, escriba el siguiente comando para eliminar la red virtual (Resource Manager):

   ```powershell
   Remove-AzResourceGroup -Name myResourceGroupA -Force
   ```

2. Para eliminar la red virtual (clásica) con PowerShell, debe modificar un archivo de configuración de red existente. Obtenga información sobre cómo [exportar, actualizar e importar archivos de configuración de red](virtual-networks-using-network-configuration-file.md). Quite el siguiente elemento VirtualNetworkSite para la red virtual que se usa en este tutorial:

   ```xml
   <VirtualNetworkSite name="myVnetB" Location="East US">
     <AddressSpace>
       <AddressPrefix>10.1.0.0/16</AddressPrefix>
     </AddressSpace>
     <Subnets>
       <Subnet name="default">
         <AddressPrefix>10.1.0.0/24</AddressPrefix>
       </Subnet>
     </Subnets>
   </VirtualNetworkSite>
   ```

   > [!WARNING]
   > Importar un archivo de configuración de red modificada puede producir cambios en las redes virtuales (clásicas) existentes en la suscripción. Asegúrese de quitar solo la red virtual anterior y que no cambia o quita ninguna red virtual existente de la suscripción. 

## <a name="next-steps"></a>Pasos siguientes

- Conozca en profundidad las [restricciones y comportamientos importantes del emparejamiento de redes virtuales](virtual-network-manage-peering.md#requirements-and-constraints) antes de crear un emparejamiento de redes virtuales para su uso en el entorno de producción.
- Conozca toda la [configuración de emparejamiento de redes virtuales](virtual-network-manage-peering.md#create-a-peering).
- Obtenga información sobre cómo [crear una topología de red de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) con emparejamiento de redes virtuales.
