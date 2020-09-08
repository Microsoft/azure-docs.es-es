---
title: 'Uso de las identidades administradas en una máquina virtual de Azure para el inicio de sesión: Azure ADV'
description: Instrucciones paso a paso y ejemplos de uso de entidades administradas de una máquina virtual de Azure de una entidad de servicio de recursos de Azure para el acceso a los recursos y el inicio de sesión de cliente mediante un script.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 08fe856fd43baad4189d81f7743396a71cd5de48
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266330"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-for-sign-in"></a>Uso de identidades administradas para recursos de Azure en una máquina virtual de Azure para el inicio de sesión 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  
En este artículo se proporcionan ejemplos de script de PowerShell y de la CLI para el inicio de sesión mediante identidades administradas para una entidad de servicio de recursos de Azure, así como instrucciones sobre temas importantes como el tratamiento de errores.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Si tiene previsto usar los ejemplos de Azure PowerShell o de la CLI de Azure en este artículo, asegúrese de instalar la versión más reciente de [Azure PowerShell](/powershell/azure/install-az-ps) o de la [CLI de Azure](/cli/azure/install-azure-cli). 

> [!IMPORTANT]
> - En todo el script de ejemplo de este artículo se da por supuesto que el cliente de la línea de comandos se ejecuta en una VM con identidades administradas para los recursos de Azure habilitados. Use la característica "Conectar" de la máquina virtual en Azure Portal para conectarse a la máquina virtual de forma remota. Para más información sobre la habilitación de identidades administradas de recursos de Azure en una VM, consulte [Configure managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM mediante Azure Portal) o uno de los artículos derivados (mediante PowerShell, la CLI, una plantilla o un SDK de Azure). 
> - Para evitar errores durante el acceso a los recursos, se debe otorgar a la identidad administrada de VM al menos el acceso de "Lector" en el ámbito adecuado (la VM u otro superior) para permitir las operaciones de Azure Resource Manager en la VM. Consulte [Asignación de acceso de Managed Service Identity a un recurso mediante Azure Portal](howto-assign-access-portal.md) para obtener más detalles.

## <a name="overview"></a>Información general

Las identidades administradas para recursos de Azure proporcionan un [objeto de entidad de servicio](../develop/developer-glossary.md#service-principal-object), que se [crea al habilitar las identidades administradas para recursos de Azure](overview.md) en la máquina virtual. A la entidad de servicio se le puede dar acceso a los recursos de Azure y los clientes de script o de línea de comandos pueden usar esta entidad para iniciar sesión y acceder a los recursos. Tradicionalmente, para acceder a los recursos protegidos con su propia identidad, un cliente de script debía hacer lo siguiente:  

   - Registrarse y ser aceptado en Azure AD como una aplicación cliente confidencial/web
   - Iniciar sesión con su entidad de servicio mediante las credenciales de la aplicación (que probablemente estaban insertadas en el script)

Con las identidades administradas para los recursos de Azure, el cliente de script ya no tiene que seguir ninguno de estos pasos, ya que puede iniciar sesión bajo las identidades administradas para la entidad de servicio de recursos de Azure. 

## <a name="azure-cli"></a>Azure CLI

El script siguiente muestra cómo:

1. Iniciar sesión en Azure AD bajo la identidad administrada de VM para la entidad de servicio de recursos de Azure  
2. Llamar a Azure Resource Manager y obtener el identificador de entidad de servicio de la máquina virtual. La CLI se ocupa de administrar la adquisición y el uso del token automáticamente. Asegúrese de sustituir el nombre de máquina virtual por `<VM-NAME>`.  

   ```azurecli
   az login --identity
   
   spID=$(az resource list -n <VM-NAME> --query [*].identity.principalId --out tsv)
   echo The managed identity for Azure resources service principal ID is $spID
   ```

## <a name="azure-powershell"></a>Azure PowerShell

El script siguiente muestra cómo:

1. Iniciar sesión en Azure AD bajo la identidad administrada de VM para la entidad de servicio de recursos de Azure  
2. Llamar a un cmdlet de Azure Resource Manager para obtener información acerca de la máquina virtual PowerShell se encarga de administrar el uso del token automáticamente.  

   ```azurepowershell
   Add-AzAccount -identity

   # Call Azure Resource Manager to get the service principal ID for the VM's managed identity for Azure resources. 
   $vmInfoPs = Get-AzVM -ResourceGroupName <RESOURCE-GROUP> -Name <VM-NAME>
   $spID = $vmInfoPs.Identity.PrincipalId
   echo "The managed identity for Azure resources service principal ID is $spID"
   ```

## <a name="resource-ids-for-azure-services"></a>Identificadores de recurso para los servicios de Azure

Consulte [Servicios de Azure que admiten la autenticación de Azure AD](services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) para ver una lista de recursos que admite Azure AD y que se han probado con identidades administradas de recursos de Azure y sus respectivos identificadores de recursos.

## <a name="error-handling-guidance"></a>Instrucciones de control de errores 

Respuestas como las siguientes pueden indicar que la identidad administrada de VM para los recursos de Azure no se ha configurado correctamente:

- PowerShell: *Invoke-WebRequest : No se puede conectar al servidor remoto*
- CLI: *MSI: No se pudo recuperar un token de `http://localhost:50342/oauth2/token` con un error de "HTTPConnectionPool(host='localhost', port=50342)"* 

Si recibe uno de estos errores, vuelva a la máquina virtual de Azure en [Azure Portal](https://portal.azure.com) y:

- Vaya a la página **Identidad** y asegúrese de que la opción **Asignado por el sistema** se haya definido como "Sí".
- Vaya a la página **Extensiones** y asegúrese de que las identidades administradas para la extensión de recursos de Azure **(planeada para quedar en desuso en enero de 2019)** se haya implementado correctamente.

Si alguna de las opciones no es correcta, puede que tenga que volver a implementar las identidades administradas para recursos de Azure en el recurso de nuevo o solucionar el error de implementación. Consulte [Configure Managed identities for Azure resources on a VM using the Azure portal](qs-configure-portal-windows-vm.md) (Configurar identidades administradas para recursos de Azure en una VM mediante Azure Portal) si necesita ayuda con la configuración de la VM.

## <a name="next-steps"></a>Pasos siguientes

- Para habilitar las identidades administradas para recursos de Azure en una VM de Azure, consulte [Configure managed identities for Azure resources on an Azure VM using PowerShell](qs-configure-powershell-windows-vm.md) (Configurar identidades administradas para recursos de Azure en una VM de Azure mediante PowerShell) o [Configure managed identities for Azure resources on an Azure VM using Azure CLI](qs-configure-cli-windows-vm.md) (Configurar identidades administradas para recursos de Azure en una VM de Azure mediante la CLI de Azure).