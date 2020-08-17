---
title: Imágenes de tipo Traiga su propia suscripción (BYOS) de Red Hat Enterprise Linux en Azure | Microsoft Docs
description: Más información sobre las imágenes de "Bring-your-own-subscription" (BYOS) para Red Hat Enterprise Linux en Azure
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: BorisB2015
editor: ''
ms.assetid: f495f1b4-ae24-46b9-8d26-c617ce3daf3a
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 06/10/2020
ms.author: alsin
ms.openlocfilehash: 54d703b8a493610174f00844cd0736f65f3ee541
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87052170"
---
# <a name="red-hat-enterprise-linux-bring-your-own-subscription-gold-images-in-azure"></a>Imágenes Gold de tipo "Bring-your-own-subscription" (BYOS) de Red Hat Enterprise Linux en Azure

Las imágenes de Red Hat Enterprise Linux (RHEL) están disponibles en Azure a través de un modelo "Bring-your-own-subscription" (BYOS) (Red Hat Gold Image). En este artículo se proporciona información general de las imágenes de Red Hat Gold Image en Azure.

>[!NOTE]
> Las imágenes Gold de tipo BYOS de RHEL están disponibles en la nube pública (comercial) de Azure y en la nube de Azure Government. No están disponibles en las nubes de Azure China ni Azure Blackforest.

## <a name="important-points-to-consider"></a>Puntos importantes para tener en cuenta

- Las imágenes de Red Hat Gold Image proporcionadas en este programa son imágenes de RHEL preparadas para la producción similares a las imágenes de Pago por uso de RHEL de Azure Marketplace.
- Las imágenes siguen las directivas actuales descritas en [Imágenes de Red Hat Enterprise Linux en Azure](./redhat-images.md).
- Las directivas de soporte técnico estándar se aplican a las máquinas virtuales creadas a partir de estas imágenes.
- Las máquinas virtuales aprovisionadas a partir de imágenes de Red Hat Gold Image no tienen las tarifas de RHEL asociadas a las imágenes de Pago por uso de RHEL.
- Las imágenes no están autorizadas, por lo que debe usar el administrador de suscripciones de Red Hat para registrarse y suscribirse a las máquinas virtuales para obtener actualizaciones de Red Hat directamente.
- Actualmente no es posible cambiar de forma dinámica entre los modelos de facturación BYOS y Pago por uso para las imágenes de Linux. Para cambiar el modelo de facturación, es necesario volver a implementar la máquina virtual a partir la imagen correspondiente.

>[!NOTE]
> Las imágenes de BYOS de RHEL de generación 2 no están disponibles actualmente en la oferta de Marketplace. Si necesita una imagen de BYOS de RHEL de generación 2, visite el panel de acceso a la nube en la administración de suscripciones de Red Hat. Para más información, consulte la [documentación de Red Hat](https://access.redhat.com/articles/4847681).

## <a name="requirements-and-conditions-to-access-the-red-hat-gold-images"></a>Requisitos y condiciones para acceder a las imágenes de Red Hat Gold Image

1. Familiarícese con los términos del [programa de acceso a la nube de Red Hat](https://www.redhat.com/en/technologies/cloud-computing/cloud-access). Habilite las suscripciones de Red Hat para el acceso a la nube en el [administrador de suscripciones de Red Hat](https://access.redhat.com/management/cloud). Deberá tener a mano las suscripciones de Azure que se van a registrar en Cloud Access.

1. Si ha habilitado suscripciones de Red Hat para Cloud Access que cumplen los requisitos de idoneidad adecuados, las suscripciones de Azure se habilitarán automáticamente para el acceso a Gold Image.

### <a name="expected-time-for-image-access"></a>Tiempo esperado para el acceso a las imágenes

Después de finalizar los pasos de habilitación de Cloud Access, Red Hat validará su idoneidad para las imágenes de Red Hat Gold Image. Si la validación se realiza correctamente, recibirá acceso a las imágenes Gold en un plazo de tres horas.

## <a name="use-the-red-hat-gold-images-from-the-azure-portal"></a>Uso de las imágenes de Red Hat Gold Image desde Azure Portal

1. Una vez que la suscripción a Azure haya recibido el acceso a las imágenes de Red Hat Gold Image, puede buscarla en [Azure Portal](https://portal.azure.com). Haga clic en **Crear un recurso** > **Ver todo**.

1. En la parte superior de la página, verá que tiene ofertas privadas.

    ![Ofertas privadas de Marketplace](./media/rhel-byos-privateoffers.png)

1. Seleccione el vínculo púrpura o desplazarse hacia abajo hasta la parte inferior de la página para ver las ofertas privadas.

1. El resto del aprovisionamiento en la interfaz de usuario no es diferente a cualquier otra imagen de Red Hat existente. Elija la versión de RHEL y siga las indicaciones para aprovisionar la máquina virtual. Este proceso también le permite aceptar los términos de la imagen en el paso final.

>[!NOTE]
>Estos pasos hasta ahora no habilitarán la imagen de Red Hat Gold Image para su implementación mediante programación. Se requiere un paso adicional, tal como se describe en la sección "Información adicional".

El resto de este documento se centra en el método de la CLI para aprovisionar y aceptar los términos de la imagen. La interfaz de usuario y la CLI son completamente intercambiables en lo que respecta al resultado final (una VM de RHEL Gold Image aprovisionada).

## <a name="use-the-red-hat-gold-images-from-the-azure-cli"></a>Uso de las imágenes de Red Hat Gold Image desde la CLI de Azure

Las siguientes instrucciones le guiarán a través del proceso de implementación inicial de una máquina virtual con RHEL mediante la CLI de Azure. En estas instrucciones se da por supuesto que tiene [instalada la CLI de Azure](/cli/azure/install-azure-cli).

>[!IMPORTANT]
>Asegúrese de usar todas las letras minúsculas en el publicador, la oferta, el plan y las referencias de imagen para los siguientes comandos.

1. Compruebe que está en la suscripción deseada.

    ```azurecli
    az account show -o=json
    ```

1. Cree un grupo de recursos para la máquina virtual de Red Hat Gold Image.

    ```azurecli
    az group create --name <name> --location <location>
    ```

1. Acepte los términos de la imagen.

    ```azurecli
    az vm image terms accept --publisher redhat --offer rhel-byos --plan <SKU value here> -o=jsonc

    # Example:
    az vm image terms accept --publisher redhat --offer rhel-byos --plan rhel-lvm75 -o=jsonc

    OR

    az vm image terms accept --urn redhat:rhel-byos:rhel-lvm8:8.0.20190620
    ```

    >[!NOTE]
    >Estos términos se deben aceptar *una vez por cada suscripción de Azure, por SKU de imagen*.

1. (Opcional) Valide la implementación de la máquina virtual con el siguiente comando:

    ```azurecli
    az vm create -n <VM name> -g <resource group name> --image <image urn> --validate

    # Example:
    az vm create -n rhel-byos-vm -g rhel-byos-group --image redhat:rhel-byos:rhel-lvm8:latest --validate
    ```

1. Aprovisione la máquina virtual mediante la ejecución del mismo comando que se mostró en el ejemplo anterior sin el argumento `--validate`.

    ```azurecli
    az vm create -n <VM name> -g <resource group name> --image <image urn>

    # Example:
    az vm create -n rhel-byos-vm -g rhel-byos-group --image redhat:rhel-byos:rhel-lvm8:latest
    ```

1. Aplique SSH en la máquina virtual y compruebe que tiene una imagen no autorizada. Para completar este paso, ejecute `sudo yum repolist`. Para RHEL 8, use `sudo dnf repolist`. La salida le pide que use el administrador de suscripciones para registrar la máquina virtual con Red Hat.

>[!NOTE]
>En RHEL 8, `dnf` y `yum` son intercambiables. Para más información, consulte la [guía de administración de RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/packaging_and_distributing_software/index).

## <a name="use-the-red-hat-gold-images-from-powershell"></a>Uso de las imágenes de Red Hat Gold Image desde PowerShell

El siguiente script es un ejemplo. Reemplace el grupo de recursos, la ubicación, el nombre de la máquina virtual, la información de inicio de sesión y otras variables con la configuración de su elección. La información del publicador y del plan debe estar en minúscula.

```powershell-interactive
    # Variables for common values
    $resourceGroup = "testbyos"
    $location = "canadaeast"
    $vmName = "test01"

    # Define user name and blank password
    $securePassword = ConvertTo-SecureString 'TestPassword1!' -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential("azureuser",$securePassword)
    Get-AzureRmMarketplaceTerms -Publisher redhat -Product rhel-byos -Name rhel-lvm75 | SetAzureRmMarketplaceTerms -Accept

    # Create a resource group
    New-AzureRmResourceGroup -Name $resourceGroup -Location $location

    # Create a subnet configuration
    $subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

    # Create a virtual network
    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName $resourceGroup -Location
    $location `-Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

    # Create a public IP address and specify a DNS name
    $pip = New-AzureRmPublicIpAddress -ResourceGroupName $resourceGroup -Location
    $location `-Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

    # Create an inbound network security group rule for port 22
    $nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name
    myNetworkSecurityGroupRuleSSH -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -
    DestinationAddressPrefix * `-DestinationPortRange 22 -Access Allow

    # Create a network security group
    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location
    $location `-Name myNetworkSecurityGroup -SecurityRules $nsgRuleSSH

    # Create a virtual network card and associate with public IP address and NSG
    $nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -
    Location $location `-SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

    # Create a virtual machine configuration
    $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize Standard_D3_v2 |
    Set-AzureRmVMOperatingSystem -Linux -ComputerName $vmName -Credential $cred |
    Set-AzureRmVMSourceImage -PublisherName redhat -Offer rhel-byos -Skus rhel-lvm75 -Version latest | Add-     AzureRmVMNetworkInterface -Id $nic.Id
    Set-AzureRmVMPlan -VM $vmConfig -Publisher redhat -Product rhel-byos -Name "rhel-lvm75"

    # Configure SSH Keys
    $sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"
    Add-AzureRmVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

    # Create a virtual machine
    New-AzureRmVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

## <a name="encrypt-red-hat-enterprise-linux-bring-your-own-subscription-gold-images"></a>Cifrado de imágenes Gold de tipo "Bring-your-own-subscription" (BYOS) de Red Hat Enterprise Linux

Las imágenes Gold de tipo BYOS de Red Hat Enterprise Linux se pueden proteger mediante el uso de [Azure Disk Encryption](../../linux/disk-encryption-overview.md). La suscripción *debe* registrarse antes de poder habilitar el cifrado. Para más información sobre cómo registrar una imagen Gold de tipo BYOS de RHEL, consulte [Registro y suscripción de un sistema al portal de clientes de Red Hat mediante el administrador de suscripciones de Red Hat](https://access.redhat.com/solutions/253273). Si tiene una suscripción de Red Hat activa, también puede consultar el artículo [Creación de claves de activación del portal de clientes de Red Hat](https://access.redhat.com/articles/1378093).

Azure Disk Encryption no se admite en las [imágenes personalizadas de Red Hat](../../linux/redhat-create-upload-vhd.md). Los requisitos adicionales y requisitos previos de Azure Disk Encryption se documentan en [Azure Disk Encryption para máquinas virtuales Linux](../../linux/disk-encryption-overview.md#additional-vm-requirements).

Los pasos para aplicar Azure Disk Encryption están disponibles en [Escenarios de Azure Disk Encryption en máquinas virtuales Linux](../../linux/disk-encryption-linux.md) y artículos relacionados.

## <a name="additional-information"></a>Información adicional

- Si intenta aprovisionar una máquina virtual en una suscripción que no está habilitada para esta oferta, recibirá el siguiente mensaje:

    ```
    "Offer with PublisherId: redhat, OfferId: rhel-byos, PlanId: rhel-lvm75 is private and can not be purchased by subscriptionId: GUID"
    ```

    En este caso, póngase en contacto con Microsoft o con Red Hat para habilitar su suscripción.

- Si modifica una instantánea a partir de una imagen BYOS de RHEL e intenta publicarla en [Shared Image Gallery](../../linux/shared-image-galleries.md), debe proporcionar información del plan que coincida con el origen inicial de la instantánea. Por ejemplo, el comando podría tener el siguiente aspecto:

    ```azurecli
    az vm create –image \
    "/subscriptions/GUID/resourceGroups/GroupName/providers/Microsoft.Compute/galleries/GalleryName/images/ImageName/versions/1.0.0" \
    -g AnotherGroupName --location EastUS2 -n VMName \
    --plan-publisher redhat --plan-product rhel-byos --plan-name rhel-lvm75
    ```

    Observe los parámetros del plan en la línea final.

    [Azure Disk Encryption](#encrypt-red-hat-enterprise-linux-bring-your-own-subscription-gold-images) no se admite en imágenes personalizadas.

- Si usa la automatización para aprovisionar máquinas virtuales de las imágenes BYOS de RHEL, deberá proporcionar parámetros de plan similares a los mostrados en los comandos de ejemplo. Por ejemplo, si usa Terraform, proporcione la información del plan en un [bloque plan](https://www.terraform.io/docs/providers/azurerm/r/virtual_machine.html#plan).

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar más información sobre el acceso a la nube de Red Hat en la [documentación de nube pública de Red Hat](https://access.redhat.com/public-cloud).
- Para ver las guías paso a paso y los detalles del programa para Cloud Access, consulte la [documentación de Red Hat Cloud Access.](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/index)
- Para más información sobre la infraestructura de actualización de Red Hat, consulte [Infraestructura de actualización de Azure Red Hat](./redhat-rhui.md).
- Para más información sobre todas las imágenes de Red Hat en Azure, consulte la [página de documentación](./redhat-images.md).
- Puede encontrar información sobre las directivas de compatibilidad de Red Hat para todas las versiones de RHEL en la página [Ciclo de vida de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).
- Para encontrar documentación adicional sobre las imágenes de RHEL Gold Image, consulte la [documentación de Red Hat](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/using_red_hat_gold_images#con-gold-image-azure).
