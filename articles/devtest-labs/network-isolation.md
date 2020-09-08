---
title: Aislamiento de red en Azure DevTest Labs
description: Obtenga información sobre el aislamiento de red en Azure DevTest Labs.
ms.topic: article
ms.date: 08/25/2020
ms.openlocfilehash: fbd2725cd3677e03cadbe0ae1f060b141f5d212b
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875542"
---
# <a name="network-isolation-in-devtest-labs"></a>Aislamiento de red en DevTest Labs

Una [red virtual de Azure](../virtual-network/virtual-networks-overview.md) actúa como un límite de seguridad, aislando los recursos de Azure de la red pública. También puede unir una red virtual de Azure a la red local para poder conectarse de forma segura a los recursos locales. En DevTest Labs, puede optar por [aislar todas las máquinas virtuales de laboratorio](devtest-lab-configure-vnet.md) y [entornos a su red](connect-environment-lab-virtual-network.md) para asegurarse de que los recursos de laboratorio siguen las directivas de red de la organización. 

Como propietario de un laboratorio, también puede optar por aislar completamente el laboratorio, lo que significa que, además de aislar las máquinas virtuales y los entornos a la red seleccionada, también puede aislar la cuenta de almacenamiento de laboratorio y los almacenes de claves creados en la suscripción. Este artículo lo guiará a través de la creación de un laboratorio con aislamiento de red. 

Consulte también los artículos siguientes:

- [Uso de cuenta de almacenamiento de laboratorio por DevTest Labs](encrypt-storage.md)
- [Uso de almacenes de claves por DevTest Labs](devtest-lab-store-secrets-in-key-vault.md)
 
> [!NOTE]
> Actualmente, el aislamiento de red solo se admite para nuevas creaciones de laboratorios.

## <a name="steps-to-enable-network-isolation-during-lab-creation"></a>Pasos para habilitar el aislamiento de red durante la creación del laboratorio

1. Durante la creación del laboratorio, vaya a la pestaña **Redes**.
1. Puede seleccionar una red **predeterminada** que el laboratorio creará automáticamente o seleccionar una red existente en la lista desplegable. Solo podrá seleccionar redes que estén en la misma región y suscripción que el laboratorio. 

    > [!div class="mx-imgBorder"]
    > ![Creación de laboratorio](./media/network-isolation/create-lab.png)
1. Seleccione una subred.

    > [!div class="mx-imgBorder"]
    > ![Creación de una subred](./media/network-isolation/create-lab-subnet.png)
1. Si decide aislar los recursos de laboratorio (la cuenta de almacenamiento y el almacén de claves del laboratorio) a la red predeterminada, no es necesario realizar ninguna otra acción y el laboratorio se ocupará del aislamiento de los recursos en el futuro.
 
    > [!div class="mx-imgBorder"]
    > ![Aislamiento de red](./media/network-isolation/isolate-lab-resources.png)
1. Si decide aislar los recursos de laboratorio (la cuenta de almacenamiento y el almacén de claves del laboratorio ) a una red existente seleccionada, debe seguir estos pasos después de la creación del laboratorio para asegurarse de que el laboratorio sigue funcionando en modo aislado. 
 
    > [!div class="mx-imgBorder"]
    > ![Aislamiento de recurso](./media/network-isolation/isolate-my-vnet.png)

    > [!IMPORTANT]
    > El propietario del laboratorio debe completar estos pasos antes de configurar o crear recursos en el laboratorio.

### <a name="steps-to-follow-post-lab-creation"></a>Pasos posteriores a la creación del laboratorio

1. En la página principal del laboratorio, seleccione el **grupo de recursos** en la página **Información general**. Debería ver la página **Grupo de recursos** del grupo de recursos que contiene el laboratorio. 
 
   > [!div class="mx-imgBorder"]
   > ![Laboratorio de Contoso](./media/network-isolation/contoso-lab.png)
1. Seleccione la cuenta de almacenamiento de Azure del laboratorio. La convención de nomenclatura para la cuenta de almacenamiento de laboratorio es: a<*nombreLaboratorioSinCaracteresNoVálidos*> *<número 4 dígitos*>. Por ejemplo, si el nombre del laboratorio es contosolab, el nombre de la cuenta de almacenamiento podría ser acontosolab1234.
 
   > [!div class="mx-imgBorder"]
   > ![Prueba de Contoso](./media/network-isolation/contoso-test.png)
1. En la cuenta de almacenamiento, vaya a Firewalls y redes virtuales y asegúrese de que la casilla "Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento" está activada. Como [DevTest Labs es un servicio de confianza de Microsoft](https://docs.microsoft.com/azure/storage/common/storage-network-security#trusted-microsoft-services), esta opción permitirá que el laboratorio funcione normalmente en un modo aislado de red. 

   > [!div class="mx-imgBorder"]
   > ![Firewalls de laboratorio de Contoso](./media/network-isolation/contoso-lab-firewalls-vnets.png)
1. A continuación, haga clic en **+ Agregar red virtual existente**, seleccione la red virtual y la subred que eligió al crear el laboratorio y haga clic en **Habilitar**. 

   > [!div class="mx-imgBorder"]
   > ![Mi red virtual de Contoso](./media/network-isolation/contoso-lab-my-vnet.png)
5.  Una vez que el punto de conexión de servicio se haya habilitado correctamente para la red virtual seleccionada, haga clic en **Agregar**. 

   > [!div class="mx-imgBorder"]
   > ![Add (Agregar)](./media/network-isolation/contoso-firewall-add.png)
 
Con esto, Azure Storage permitirá conexiones entrantes de la red virtual agregada y permitirá que el laboratorio funcione correctamente en un modo de aislamiento de red. 

También puede elegir automatizar estos pasos para configurar esta opción para varios laboratorios. 

[Más información sobre la administración de reglas de acceso de red predeterminadas para Azure Storage mediante PowerShell y la CLI](https://docs.microsoft.com/azure/storage/common/storage-network-security?toc=/azure/virtual-network/toc.json#powershell)

## <a name="things-to-remember-while-using-a-lab-in-a-network-isolated-mode"></a>Elementos que hay que recordar al usar un laboratorio en un modo aislado de red

### <a name="accessing-labs-storage-account-outside-the-lab"></a>Acceso a la cuenta de almacenamiento del laboratorio fuera del laboratorio 

Dentro de un laboratorio con aislamiento de red, para acciones como cargar un VHD a la cuenta de almacenamiento del laboratorio para crear imágenes personalizadas, el propietario del laboratorio deberá habilitar explícitamente el acceso a la cuenta de almacenamiento desde un punto de conexión permitido. Para ello, puede crear una máquina virtual y tener acceso seguro a la cuenta de almacenamiento del laboratorio desde esa máquina virtual. 

[Más información sobre el acceso a una cuenta de almacenamiento de forma privada desde una máquina virtual](../private-link/create-private-endpoint-storage-portal.md)

### <a name="exporting-usage-data-from-the-lab"></a>Exportación de datos de uso del laboratorio 

Dentro de un laboratorio con aislamiento de red, para [exportar los datos de uso personales del laboratorio](personal-data-delete-export.md), el propietario del laboratorio debe proporcionar la cuenta de almacenamiento de forma explícita y generar un blob en la cuenta para almacenar los datos. 

Si no se proporciona una cuenta de almacenamiento, esta operación producirá un error en el modo de aislamiento de red porque el laboratorio no podrá acceder a su cuenta de almacenamiento si el cliente no hubiera proporcionado una. 

[Más información sobre la exportación de datos de uso del laboratorio en una cuenta de almacenamiento especificada](personal-data-delete-export.md#azure-powershell)

## <a name="next-steps"></a>Pasos siguientes

[Creación o modificación de laboratorios automáticamente con plantillas de Azure Resource Manager y PowerShell](devtest-lab-use-arm-and-powershell-for-lab-resources.md)
