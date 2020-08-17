---
title: 'Creación de un emparejamiento de redes virtuales: diferentes suscripciones'
titlesuffix: Azure Virtual Network
description: Aprenda a crear un emparejamiento de redes virtuales entre redes virtuales creadas mediante Resource Manager que existen en diferentes suscripciones de Azure, en el mismo inquilino de Azure Active Directory o en otro.
services: virtual-network
documentationcenter: ''
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/09/2019
ms.author: kumud
ms.openlocfilehash: 79062ae45f04b290f6e4120906b98590ce95dbe1
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87833273"
---
# <a name="create-a-virtual-network-peering---resource-manager-different-subscriptions-and-azure-active-directory-tenants"></a>Creación de un emparejamiento de redes virtuales: Resource Manager, diferentes suscripciones e inquilinos de Azure Active Directory

En este tutorial aprenderá a crear un emparejamiento de redes virtuales entre dos redes virtuales creadas mediante Resource Manager. Las redes virtuales existen en distintas suscripciones que pueden pertenecer a distintos inquilinos de Azure Active Directory (Azure AD). Emparejar dos redes virtuales permite que los recursos en distintas redes virtuales se comuniquen entre sí con el mismo ancho de banda y latencia que tendrían los recursos si estuvieran en la misma red virtual. Obtenga más información sobre el [Emparejamiento de redes virtuales](virtual-network-peering-overview.md).

Los pasos para crear un emparejamiento de redes virtuales cambian en función de si las redes virtuales se encuentran en la misma suscripción o en suscripciones diferentes, y en función del [modelo de implementación de Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json) con el que se han creado las redes virtuales. Para más información sobre cómo crear un emparejamiento de redes virtuales en otros escenarios, seleccione el escenario en la tabla siguiente:

|Modelo de implementación de Azure  | Suscripción de Azure  |
|--------- |---------|
|[Ambas mediante Resource Manager](tutorial-connect-virtual-networks-portal.md) |Iguales|
|[Una mediante Resource Manager y la otra clásica](create-peering-different-deployment-models.md) |Iguales|
|[Una mediante Resource Manager y la otra clásica](create-peering-different-deployment-models-subscriptions.md) |Diferentes|

No se puede crear un emparejamiento de redes virtuales entre dos redes virtuales implementadas mediante el modelo de implementación clásico. Si necesita conectar redes virtuales que se crearon a través del modelo de implementación clásica, puede usar una instancia de [Azure VPN Gateway](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para conectar las redes virtuales.

En este tutorial se emparejan redes virtuales de la misma región. También puede emparejar redes virtuales en distintas [regiones compatibles](virtual-network-manage-peering.md#cross-region). Se recomienda que se familiarice con los [requisitos y restricciones de emparejamiento](virtual-network-manage-peering.md#requirements-and-constraints) antes de emparejar redes virtuales.

Puede usar [Azure Portal](#portal), Azure [PowerShell](#cli), la [interfaz de la línea de comandos](#powershell) (CLI) de Azure o la [plantilla de Azure Resource Manager](#template) para crear un emparejamiento de redes virtuales. Seleccione cualquiera de los vínculos anteriores de herramientas para ir directamente a los pasos para crear un emparejamiento de redes virtuales con la herramienta de su preferencia.

Si las redes virtuales están en diferentes suscripciones y las suscripciones están asociadas a diferentes inquilinos de Azure Active Directory, complete los siguientes pasos antes de continuar:
1. Agregue el usuario de cada inquilino de Active Directory como un [usuario invitado](../active-directory/b2b/add-users-administrator.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-guest-users-to-the-directory) en el inquilino de Azure Active Directory opuesto.
1. Cada usuario debe aceptar la invitación del usuario invitado del inquilino de Azure Active Directory opuesto.

## <a name="create-peering---azure-portal"></a><a name="portal"></a>Creación de emparejamiento: Azure Portal

Los pasos siguientes usan cuentas diferentes para cada suscripción. Si está usando una cuenta que tiene permisos para ambas suscripciones puede usar la misma cuenta para todos los pasos, y omitir los pasos para cerrar sesión en el portal y para asignar a otro usuario permisos para las redes virtuales.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como *UserA*. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
2. Seleccione **+ Crear un recurso**, seleccione **Redes** y, luego, **Red virtual**.
3. Seleccione o escriba los valores de ejemplo siguientes para las siguientes opciones y, luego, seleccione **Crear**:
    - **Nombre**: *myVnetA*
    - **Espacio de direcciones**: *10.0.0.0/16*
    - **Nombre de subred**: *default*
    - **Rango de direcciones de subred**: *10.0.0.0/24*
    - **Suscripción**: Seleccione la suscripción A.
    - **Grupo de recursos**: seleccione **Crear nuevo** y escriba *myResourceGroupA*.
    - **Ubicación**: *Este de EE. UU.*
4. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetA*. Seleccione **myVnetA** cuando aparezca en los resultados de la búsqueda. 
5. Seleccione **Control de acceso (IAM)** en la lista de opciones vertical que aparece a la izquierda.
6. En **myVnetA: control de acceso (IAM)** , seleccione **+ Agregar asignación de roles**.
7. Seleccione **Colaborador de la red** en la casilla **Rol**.
8. En el cuadro **Seleccionar**, seleccione *UserB* o escriba la dirección de correo electrónico de UserB para buscarlo.
9. Seleccione **Guardar**.
10. En **myVnetA: control de acceso (IAM)** , seleccione **Propiedades** en la lista de opciones vertical que aparece a la izquierda. Copie el **ID. DE RECURSO**, ya que se usará en un paso posterior. El ID de respuesta es similar al siguiente ejemplo: `/subscriptions/<Subscription Id>/resourceGroups/myResourceGroupA/providers/Microsoft.Network/virtualNetworks/myVnetA`.
11. Cierre sesión en el portal como UserA e inicie sesión como UserB.
12. Complete los pasos del 2 al 3 y especifique o seleccione los valores siguientes en el paso 3:

    - **Nombre**: *myVnetB*
    - **Espacio de direcciones**: *10.1.0.0/16*
    - **Nombre de subred**: *default*
    - **Rango de direcciones de subred**: *10.1.0.0/24*
    - **Suscripción**: Seleccione la suscripción B.
    - **Grupo de recursos**: haga clic en **Crear nuevo** y escriba *myResourceGroupB*.
    - **Ubicación**: *Este de EE. UU.*

13. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetB*. Seleccione **myVnetB** cuando aparezca en los resultados de la búsqueda.
14. En **myVnetB**, seleccione **Propiedades** en la lista de opciones vertical que aparece a la izquierda. Copie el **ID. DE RECURSO**, ya que se usará en un paso posterior. El ID de respuesta es similar al siguiente ejemplo: `/subscriptions/<Subscription ID>/resourceGroups/myResourceGroupB/providers/Microsoft.ClassicNetwork/virtualNetworks/myVnetB`.
15. Seleccione **Control de acceso (IAM)** en **myVnetB** y complete los pasos del 5 al 10 para myVnetB (en el paso 8, escriba **UserA**).
16. Cierre sesión en el portal como UserB e inicie sesión como UserA.
17. En el cuadro **Buscar recursos** en la parte superior del portal, escriba *myVnetA*. Seleccione **myVnetA** cuando aparezca en los resultados de la búsqueda.
18. Seleccione **myVnetA**.
19. En **CONFIGURACIÓN**, seleccione **Emparejamientos**.
20. En **myVnetA : emparejamientos**, seleccione **+ Agregar**.
21. En **Agregar emparejamiento**, escriba o seleccione las opciones siguientes y, luego, seleccione **Aceptar**:
     - **Nombre**: *myVnetAToMyVnetB*
     - **Modelo de implementación de red virtual**:  Select **Administrador de recursos**.
     - **Conozco mi Id. de recurso**: Active esta casilla.
     - **Identificador de recurso**: escriba el identificador de recurso del paso 14.
     - **Permitir acceso a red virtual**: asegúrese de que está seleccionada la opción **Habilitado**.
    En este tutorial no se usa ninguna otra configuración. Para conocer todas las configuraciones de emparejamiento, lea [Manage virtual network peerings](virtual-network-manage-peering.md#create-a-peering) (Administración de emparejamientos de redes virtuales).
22. El emparejamiento que creó aparece poco después de seleccionar **Aceptar** en el paso anterior. El estado **Iniciado** aparece en la columna **ESTADO DE EMPAREJAMIENTO** correspondiente al emparejamiento **myVnetAToMyVnetB** que ha creado. Ha emparejado myVnetA con myVnetB, pero ahora debe emparejar myVnetB con myVnetA. Debe crear el emparejamiento en ambas direcciones para permitir que los recursos de las redes virtuales se comuniquen entre sí.
23. Cierre sesión en el portal como UserA e inicie sesión como UserB.
24. Complete nuevamente los pasos del 17 al 21 para MyVnetB. En el paso 21, asigne al emparejamiento el nombre *myVnetBToMyVnetA*, seleccione *myVnetA* para **Red Virtual** y escriba el identificador del paso 10 en el cuadro **Id. de recurso**.
25. Unos segundos después de seleccionar **Aceptar** para crear el emparejamiento de myVnetB, el emparejamiento **myVnetBToMyVnetA** que acaba de crear aparece con el estado **Conectado** en la columna **ESTADO DE EMPAREJAMIENTO**.
26. Cierre sesión en el portal como UserB e inicie sesión como UserA.
27. Complete nuevamente los pasos del 17 al 19. El **ESTADO DE EMPAREJAMIENTO** correspondiente al emparejamiento **myVnetAToVNetB** ahora también aparece como **Conectado**. El emparejamiento se establece correctamente una vez que ve el estado **Conectado** en la columna **ESTADO DE EMPAREJAMIENTO** para las dos redes virtuales del emparejamiento. Los recursos de Azure que cree en cualquiera de las redes virtuales ahora se pueden comunicar entre sí mediante sus direcciones IP. Si usa la resolución de nombres predeterminada de Azure para las redes virtuales, los recursos de las redes virtuales no pueden resolver nombres entre las redes virtuales. Si desea resolver nombres entre las redes virtuales de un emparejamiento, debe crear su propio servidor DNS. Obtenga información sobre cómo configurar la [resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).
28. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
29. **Opcional**: para eliminar los recursos que crea en este tutorial, complete los pasos que aparecen en la sección [Eliminación de recursos](#delete-portal) de este artículo.

## <a name="create-peering---azure-cli"></a><a name="cli"></a>Creación de emparejamiento: CLI de Azure

En este tutorial se usan cuentas diferentes para cada suscripción. Si está usando una cuenta que tiene permisos para ambas suscripciones, puede usar la misma cuenta para todos los pasos, omitir los pasos para cerrar sesión en Azure y quitar las líneas del script que crean las asignaciones de roles de usuario. Reemplace UserA@azure.com y UserB@azure.com en todos los scripts siguientes por los nombres de usuario que está usando para UserA y UserB. 

Los scripts siguientes:

- Requiere la versión 2.0.4 o posterior de la CLI de Azure. Para encontrar la versión, ejecute `az --version`. Si debe actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Funciona en un shell de Bash. Para ver las opciones de ejecución de scripts de la CLI de Azure en un cliente Windows, consulte [Instalación de la CLI de Azure 2.0 en Windows](/cli/azure/install-azure-cli-windows).

En lugar de instalar la CLI y sus dependencias, puede usar Azure Cloud Shell. Azure Cloud Shell es un shell de Bash gratuito que se puede ejecutar directamente en Azure Portal. Tiene la CLI de Azure preinstalada y configurada para utilizarla con la cuenta. Seleccione el botón **Pruébelo** en el script siguiente, que invoca un Cloud Shell en el que puede iniciar sesión con su cuenta de Azure.

1. Abra una sesión de la CLI e inicie sesión en Azure como UserA mediante el comando `azure login`. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
2. Copie el script siguiente en un editor de texto del equipo, reemplace `<SubscriptionA-Id>` por el identificador de SubscriptionA, copie el script modificado, péguelo en la sesión de la CLI y pulse `Enter`. Si no conoce el Id. de suscripción, escriba el comando `az account show`. El valor de **id** en la salida es el identificador de la suscripción.

    ```azurecli-interactive
    # Create a resource group.
    az group create \
      --name myResourceGroupA \
      --location eastus

    # Create virtual network A.
    az network vnet create \
      --name myVnetA \
      --resource-group myResourceGroupA \
      --location eastus \
      --address-prefix 10.0.0.0/16

    # Assign UserB permissions to virtual network A.
    az role assignment create \
      --assignee UserB@azure.com \
      --role "Network Contributor" \
      --scope /subscriptions/<SubscriptionA-Id>/resourceGroups/myResourceGroupA/providers/Microsoft.Network/VirtualNetworks/myVnetA
    ```

3. Cierre sesión en Azure como UserA mediante el comando `az logout` e inicie sesión en Azure como UserB. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
4. Cree myVnetB. Copie el contenido del script del paso 2 en un editor de texto del equipo. Reemplace `<SubscriptionA-Id>` por el identificador de SubscriptionB. Cambie 10.0.0.0/16 a 10.1.0.0/16, y reemplace todas las apariciones de A por B, y todas las apariciones de B por A. Copie el script modificado, péguelo en la sesión de la CLI y pulse `Enter`.
5. Cierre sesión en Azure como UserB e inicie sesión en Azure como UserA.
6. Cree un emparejamiento de redes virtuales myVnetA a myVnetB. Copie el contenido del script siguiente en un editor de texto del equipo. Reemplace `<SubscriptionB-Id>` por el identificador de SubscriptionB. Para ejecutar el script, copie el script modificado, péguelo en la sesión de la CLI y pulse ENTRAR.

    ```azurecli-interactive
        # Get the id for myVnetA.
        vnetAId=$(az network vnet show \
          --resource-group myResourceGroupA \
          --name myVnetA \
          --query id --out tsv)

        # Peer myVNetA to myVNetB.
        az network vnet peering create \
          --name myVnetAToMyVnetB \
          --resource-group myResourceGroupA \
          --vnet-name myVnetA \
          --remote-vnet /subscriptions/<SubscriptionB-Id>/resourceGroups/myResourceGroupB/providers/Microsoft.Network/VirtualNetworks/myVnetB \
          --allow-vnet-access
    ```

7. Compruebe el estado de emparejamiento de myVnetA.

    ```azurecli-interactive
    az network vnet peering list \
      --resource-group myResourceGroupA \
      --vnet-name myVnetA \
      --output table
    ```

    El estado es **Iniciado**. Cuando haya creado el emparejamiento a myVnetA desde myVnetB, cambiará a **Conectado**.

8. Cierre sesión en Azure como UserA e inicie sesión en Azure como UserB.
9. Cree el emparejamiento de myVnetB a myVnetA. Copie el contenido del script del paso 6 en un editor de texto del equipo. Reemplace `<SubscriptionB-Id>` por el identificador de SubscriptionA, todas las apariciones de A por B, y todas las apariciones de B por A. Una vez realizados los cambios, copie el script modificado, péguelo en la sesión de la CLI y pulse `Enter`.
10. Compruebe el estado de emparejamiento de myVnetB. Copie el contenido del script del paso 7 en un editor de texto del equipo. Reemplace A por B en el nombre del grupo de recursos y de la red virtual, copie el script, pegue el script modificado en la sesión de la CLI y pulse `Enter`. El estado de emparejamiento es **Conectado**. El estado de emparejamiento de myVnetA cambiará a **Conectado** después de que haya creado el emparejamiento de myVnetB a myVnetA. Puede volver a iniciar sesión como UserA en Azure y realizar el paso 7 de nuevo para comprobar el estado de emparejamiento de myVnetA. 

    > [!NOTE]
    > El emparejamiento no se habrá establecido mientras el estado de emparejamiento de ambas redes virtuales no sea **Conectado**.

11. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
12. **Opcional**: para eliminar los recursos creados en este tutorial, complete los pasos de la sección [Eliminar recursos](#delete-cli) de este artículo.

Los recursos de Azure que cree en cualquiera de las redes virtuales ahora se pueden comunicar entre sí mediante sus direcciones IP. Si usa la resolución de nombres predeterminada de Azure para las redes virtuales, los recursos de las redes virtuales no pueden resolver nombres entre las redes virtuales. Si desea resolver nombres entre las redes virtuales de un emparejamiento, debe crear su propio servidor DNS. Obtenga información sobre cómo configurar la [resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

## <a name="create-peering---powershell"></a><a name="powershell"></a>Creación de emparejamiento: PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este tutorial se usan cuentas diferentes para cada suscripción. Si está usando una cuenta que tiene permisos para ambas suscripciones, puede usar la misma cuenta para todos los pasos, omitir los pasos para cerrar sesión en Azure y quitar las líneas del script que crean las asignaciones de roles de usuario. Reemplace UserA@azure.com y UserB@azure.com en todos los scripts siguientes por los nombres de usuario que está usando para UserA y UserB.

1. Confirme que tiene Azure PowerShell versión 1.0.0 o superior. Puede hacerlo mediante la ejecución de `Get-Module -Name Az`. Recomendamos instalar la última versión del [módulo Az](/powershell/azure/install-az-ps) de PowerShell. Si no está familiarizado con Azure PowerShell, consulte [Introducción a Azure PowerShell](/powershell/azure/?toc=%2fazure%2fvirtual-network%2ftoc.json). 
2. Inicie una sesión de PowerShell.
3. En PowerShell, inicie sesión en Azure como UserA. Para ello, escriba el comando `Connect-AzAccount`. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
4. Cree un grupo de recursos y una red virtual A. Copie el script siguiente en un editor de texto del equipo. Reemplace `<SubscriptionA-Id>` por el identificador de SubscriptionA. Si no conoce el identificador de la suscripción, escriba el comando `Get-AzSubscription` para verlo. El valor de **id** en la salida devuelta es el identificador de la suscripción. Para ejecutar el script, copie el script modificado, péguelo en PowerShell y pulse `Enter`.

    ```powershell
    # Create a resource group.
    New-AzResourceGroup `
      -Name MyResourceGroupA `
      -Location eastus

    # Create virtual network A.
    $vNetA = New-AzVirtualNetwork `
      -ResourceGroupName MyResourceGroupA `
      -Name 'myVnetA' `
      -AddressPrefix '10.0.0.0/16' `
      -Location eastus

    # Assign UserB permissions to myVnetA.
    New-AzRoleAssignment `
      -SignInName UserB@azure.com `
      -RoleDefinitionName "Network Contributor" `
      -Scope /subscriptions/<SubscriptionA-Id>/resourceGroups/myResourceGroupA/providers/Microsoft.Network/VirtualNetworks/myVnetA
    ```

5. Cierre sesión en Azure como UserA e inicie sesión como UserB. La cuenta con la que inicie sesión debe tener todos los permisos necesarios para crear un emparejamiento de redes virtuales. Para ver una lista de permisos, consulte [Permisos de emparejamiento de red virtual](virtual-network-manage-peering.md#permissions).
6. Copie el contenido del script del paso 4 en un editor de texto del equipo. Reemplace `<SubscriptionA-Id>` por el identificador de SubscriptionB. Cambie 10.0.0.0/16 a 10.1.0.0/16. Reemplace todas las apariciones de A por B, y todas las apariciones de B por A. Para ejecutar el script, copie el script modificado, péguelo en PowerShell y pulse `Enter`.
7. Cierre sesión como UserB en Azure e inicie sesión como UserA.
8. Cree el emparejamiento de myVnetA a myVnetB. Copie el script siguiente en un editor de texto del equipo. Reemplace `<SubscriptionB-Id>` por el identificador de SubscriptionB. Para ejecutar el script, copie el script modificado, péguelo en PowerShell y pulse `Enter`.

   ```powershell
   # Peer myVnetA to myVnetB.
   $vNetA=Get-AzVirtualNetwork -Name myVnetA -ResourceGroupName myResourceGroupA
   Add-AzVirtualNetworkPeering `
     -Name 'myVnetAToMyVnetB' `
     -VirtualNetwork $vNetA `
     -RemoteVirtualNetworkId "/subscriptions/<SubscriptionB-Id>/resourceGroups/myResourceGroupB/providers/Microsoft.Network/virtualNetworks/myVnetB"
   ```

9. Compruebe el estado de emparejamiento de myVnetA.

    ```powershell
    Get-AzVirtualNetworkPeering `
      -ResourceGroupName myResourceGroupA `
      -VirtualNetworkName myVnetA `
      | Format-Table VirtualNetworkName, PeeringState
    ```

    El estado es **Iniciado**. Cuando haya configurado el emparejamiento a myVnetA desde myVnetB, cambiará a **Conectado**.

10. Cierre sesión en Azure como UserA e inicie sesión como UserB.
11. Cree el emparejamiento de myVnetB a myVnetA. Copie el contenido del script del paso 8 en un editor de texto del equipo. Reemplace `<SubscriptionB-Id>` por el identificador de la suscripción A y cambie todas las A por B y todas las B por A. Para ejecutar el script, copie el script modificado, péguelo en PowerShell y luego presione `Enter`.
12. Compruebe el estado de emparejamiento de myVnetB. Copie el contenido del script del paso 9 en un editor de texto del equipo. Reemplace A por B en el nombre del grupo de recursos y de la red virtual. Para ejecutar el script, pegue el script modificado en PowerShell y pulse `Enter`. El estado es **Conectado**. El estado de emparejamiento de **myVnetA** cambiará a **Conectado** después de que haya creado el emparejamiento de **myVnetB** a **myVnetA**. Puede volver a iniciar sesión como UserA en Azure y realizar el paso 9 de nuevo para comprobar el estado de emparejamiento de myVnetA.

    > [!NOTE]
    > El emparejamiento no se habrá establecido mientras el estado de emparejamiento de ambas redes virtuales no sea **Conectado**.

    Los recursos de Azure que cree en cualquiera de las redes virtuales ahora se pueden comunicar entre sí mediante sus direcciones IP. Si usa la resolución de nombres predeterminada de Azure para las redes virtuales, los recursos de las redes virtuales no pueden resolver nombres entre las redes virtuales. Si desea resolver nombres entre las redes virtuales de un emparejamiento, debe crear su propio servidor DNS. Obtenga información sobre cómo configurar la [resolución de nombres mediante su propio servidor DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

13. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
14. **Opcional**: para eliminar los recursos creados en este tutorial, complete los pasos de la sección [Eliminar recursos](#delete-powershell) de este artículo.

## <a name="create-peering---resource-manager-template"></a><a name="template"></a>Creación de emparejamiento: plantilla de Resource Manager

1. Para crear una red virtual y asignar los [permisos](virtual-network-manage-peering.md#permissions) adecuados, complete los pasos que aparecen en las secciones [Portal](#portal), [CLI de Azure](#cli) o [PowerShell](#powershell) de este artículo.
2. Guarde el texto que sigue a un archivo en el equipo local. Reemplace `<subscription ID>` por el identificador de suscripción de UserA. Puede guardar el archivo como vnetpeeringA.json, por ejemplo.

   ```json
   {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        },
        "variables": {
        },
    "resources": [
            {
            "apiVersion": "2016-06-01",
            "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
            "name": "myVnetA/myVnetAToMyVnetB",
            "location": "[resourceGroup().location]",
            "properties": {
            "allowVirtualNetworkAccess": true,
            "allowForwardedTraffic": false,
            "allowGatewayTransit": false,
            "useRemoteGateways": false,
                "remoteVirtualNetwork": {
                "id": "/subscriptions/<subscription ID>/resourceGroups/PeeringTest/providers/Microsoft.Network/virtualNetworks/myVnetB"
                }
            }
            }
        ]
   }
   ```

3. Inicie sesión en Azure como UserA e implemente la plantilla mediante el [portal](../azure-resource-manager/templates/deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-resources-from-custom-template), [PowerShell](../azure-resource-manager/templates/deploy-powershell.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-local-template) o la [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-local-template). Especifique el nombre de archivo que ha guardado en el paso 2 para el texto del archivo json de ejemplo.
4. Copie el archivo json de ejemplo del paso 2 en un archivo del equipo y realice cambios en las líneas que comienzan por:
   - **name**: cambie *myVnetA/myVnetAToMyVnetB* por *myVnetB/myVnetBToMyVnetA*.
   - **id**: reemplace `<subscription ID>` por el identificador de suscripción de UserB y cambie *myVnetB* por *myVnetA*.
5. Vuelva a realizar el paso 3, con la sesión de Azure iniciada como UserB.
6. **Opcional**: Si bien este tutorial no aborda la creación de máquinas virtuales, puede crear una máquina virtual en cada red virtual y conectar de una máquina virtual a la otra para así validar la conectividad.
7. **Opcional**: para eliminar los recursos que crea en este tutorial, complete los pasos que aparecen en la sección [Eliminación de recursos](#delete) de este artículo, ya sea mediante Azure Portal, PowerShell o la CLI de Azure.

## <a name="delete-resources"></a><a name="delete"></a>Eliminación de recursos
Cuando haya terminado este tutorial, es posible que quiera eliminar los recursos que creó en el tutorial, para no incurrir en gastos de uso. Al eliminar un grupo de recursos se eliminan también todos los recursos contenidos en el mismo.

### <a name="azure-portal"></a><a name="delete-portal"></a>Azure Portal

1. Inicie sesión en Azure Portal como UserA.
2. En el cuadro de búsqueda del portal, escriba **myResourceGroupA**. En los resultados de la búsqueda, seleccione **myResourceGroupA**.
3. Seleccione **Eliminar**.
4. Para confirmar la eliminación, en el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS**, escriba **myResourceGroupA** y, luego, seleccione **Eliminar**.
5. Cierre sesión en el portal como UserA e inicie sesión como UserB.
6. Complete los pasos del 2 al 4 para myResourceGroupB.

### <a name="azure-cli"></a><a name="delete-cli"></a>Azure CLI

1. Inicie sesión en Azure como UserA y ejecute el comando siguiente:

   ```azurecli-interactive
   az group delete --name myResourceGroupA --yes
   ```

2. Cierre sesión en Azure como UserA e inicie sesión como UserB.
3. Ejecute el comando siguiente:

   ```azurecli-interactive
   az group delete --name myResourceGroupB --yes
   ```

### <a name="powershell"></a><a name="delete-powershell"></a>PowerShell

1. Inicie sesión en Azure como UserA y ejecute el comando siguiente:

   ```powershell
   Remove-AzResourceGroup -Name myResourceGroupA -force
   ```

2. Cierre sesión en Azure como UserA e inicie sesión como UserB.
3. Ejecute el comando siguiente:

   ```powershell
   Remove-AzResourceGroup -Name myResourceGroupB -force
   ```

## <a name="next-steps"></a>Pasos siguientes

- Conozca en profundidad las [restricciones y comportamientos importantes del emparejamiento de redes virtuales](virtual-network-manage-peering.md#requirements-and-constraints) antes de crear un emparejamiento de redes virtuales para su uso en el entorno de producción.
- Conozca toda la [configuración de emparejamiento de redes virtuales](virtual-network-manage-peering.md#create-a-peering).
- Obtenga información sobre cómo [crear una topología de red de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) con emparejamiento de redes virtuales.
