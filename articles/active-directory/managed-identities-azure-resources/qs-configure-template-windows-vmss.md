---
title: 'Configuración dela plantilla para usar identidades administradas en conjuntos de escalado de máquinas virtuales: Azure AD'
description: Instrucciones detalladas para configurar identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales mediante una plantilla de Azure Resource Manager.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/20/2018
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 30185733a89ecf078bd2077cdcb51d6817f889d2
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266755"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-virtual-machine-scale-using-a-template"></a>Configuración de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales de Azure mediante una plantilla

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código.

En este artículo, aprenderá a realizar las siguientes operaciones de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales de Azure mediante la plantilla de implementación de Azure Resource Manager:
- Habilitación y deshabilitación de la identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales de Azure
- Adición y eliminación de una identidad asignada por el usuario un conjunto de escalado de máquinas virtuales de Azure

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para realizar las operaciones de administración de este artículo, su cuenta debe tener las siguientes asignaciones de control de acceso basado en rol:

    > [!NOTE]
    > No se requieren asignaciones de roles del directorio de Azure AD.

    - [Colaborador de máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) para crear un conjunto de escalado de máquinas virtuales y habilitar y quitar la identidad administrada asignada por el usuario o por el sistema desde un conjunto de escalado de máquinas virtuales.
    - Rol [Colaborador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-contributor) para crear una identidad administrada asignada por el usuario.
    - Rol [Operador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-operator) para asignar y quitar una identidad administrada asignada por el usuario en un conjunto de escalado de máquinas virtuales.

## <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure

Al igual que con Azure Portal y los scripts, las plantillas de [Azure Resource Manager](../../azure-resource-manager/management/overview.md) proporcionan la capacidad de implementar recursos nuevos o modificados definidos por un grupo de recursos de Azure. Existen varias opciones para la edición e implementación de plantillas, tanto localmente como basadas en el portal, incluidas:

   - Usar una [plantilla personalizada de Azure Marketplace](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), que permite crear una plantilla desde cero, o bien basada en una plantilla común existente o en una [plantilla de inicio rápido](https://azure.microsoft.com/documentation/templates/).
   - Derivar a partir de un grupo de recursos existente, exportando una plantilla de [la implementación original](../../azure-resource-manager/templates/export-template-portal.md) o del [estado actual de la implementación](../../azure-resource-manager/templates/export-template-portal.md).
   - Usar un [editor de JSON (por ejemplo, VS Code)](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) local y, a continuación, cargarla e implementarla con PowerShell o la CLI.
   - Usar el [proyecto del grupo de recursos de Azure](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) de Visual Studio tanto para crear como para implementar una plantilla.  

Independientemente de la opción que elija, la sintaxis de la plantilla es la misma durante la implementación inicial y posteriores implementaciones. La habilitación de identidades administradas de recursos de Azure en una máquina virtual existente o nueva se realiza de la misma forma. Además, de forma predeterminada, Azure Resource Manager realiza una [actualización incremental](../../azure-resource-manager/templates/deployment-modes.md) en las implementaciones.

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, se habilita y deshabilita la identidad administrada asignada por el sistema con una plantilla de Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-the-creation-of-a-virtual-machines-scale-set-or-an-existing-virtual-machine-scale-set"></a>Habilitación de la identidad administrada asignada por el sistema durante la creación de un conjunto de escalado de máquinas virtuales o un conjunto de escalado de máquinas virtuales existente

1. Independientemente de que inicie sesión localmente en Azure o mediante Azure Portal, use una cuenta que esté asociada a la suscripción de Azure que contiene el conjunto de escalado de máquinas virtuales.
2. Para habilitar la identidad administrada asignada por el sistema, cargue la plantilla en un editor, busque el recurso de interés `Microsoft.Compute/virtualMachinesScaleSets` en la sección de recursos y agregue la propiedad `identity` en el mismo nivel que la propiedad `"type": "Microsoft.Compute/virtualMachinesScaleSets"`. Use la sintaxis siguiente:

   ```JSON
   "identity": {
       "type": "SystemAssigned"
   }
   ```

> [!NOTE]
> De forma opcional, puede aprovisionar las identidades administradas de extensión del conjunto de escalado de máquinas virtuales de los recursos de Azure si lo especifica en el elemento `extensionProfile` de la plantilla. Este paso es opcional, ya que puede usar el punto de conexión de identidad de Azure Instance Metadata Service (IMDS) para recuperar tokens.  Para obtener más información, consulte [Migración de la extensión de máquina virtual a IMDS de Azure para la autenticación](howto-migrate-vm-extension.md).


4. Cuando haya terminado, deben agregarse las siguientes secciones a la sección de recursos de la plantilla para que se parezca a la siguiente:

   ```json
    "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "name": "[variables('vmssName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "SystemAssigned",
            },
           "properties": {
                //other resource provider properties...
                "virtualMachineProfile": {
                    //other virtual machine profile properties...
                    //The following appears only if you provisioned the optional virtual machine scale set extension (to be deprecated)
                    "extensionProfile": {
                        "extensions": [
                            {
                                "name": "ManagedIdentityWindowsExtension",
                                "properties": {
                                  "publisher": "Microsoft.ManagedIdentity",
                                  "type": "ManagedIdentityExtensionForWindows",
                                  "typeHandlerVersion": "1.0",
                                  "autoUpgradeMinorVersion": true,
                                  "settings": {
                                      "port": 50342
                                  }
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
   ```

### <a name="disable-a-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Deshabilitación de una identidad administrada asignada por el sistema de un conjunto de escalado de máquinas virtuales de Azure

Si tiene un conjunto de escalado de máquinas virtuales que ya no necesita una identidad administrada asignada por el sistema, haga lo siguiente:

1. Independientemente de que inicie sesión localmente en Azure o mediante Azure Portal, use una cuenta que esté asociada a la suscripción de Azure que contiene el conjunto de escalado de máquinas virtuales.

2. Cargue la plantilla en un [editor](#azure-resource-manager-templates) y busque el `Microsoft.Compute/virtualMachineScaleSets`recurso de interés dentro de la sección `resources`. Si dispone de una VM que solo tenga una identidad administrada asignada por el sistema, puede deshabilitarla cambiando el tipo de identidad a `None`.

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2018-06-01**

   Si apiVersion es `2018-06-01` y la VM tiene identidades administradas asignadas tanto por el usuario como por el sistema, quite `SystemAssigned` del tipo de identidad y mantenga `UserAssigned` junto con los valores del diccionario userAssignedIdentities.

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2018-06-01**

   Si apiVersion es `2017-12-01` y el conjunto de escalado de máquinas virtuales tiene identidades administradas asignadas tanto por el usuario como por el sistema, quite `SystemAssigned` del tipo de identidad y mantenga `UserAssigned` junto con la matriz `identityIds` de las identidades administradas asignadas por el usuario.



   En el ejemplo siguiente se muestra cómo quitar una identidad administrada asignada por el sistema de un conjunto de escalado de máquinas virtuales sin identidades administradas asignadas por el usuario:

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2018-06-01",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "None"
        }

   }
   ```

## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

En esta sección, asignará una identidad administrada asignada por el usuario a un conjunto de escalado de máquinas virtuales mediante la plantilla de Azure Resource Manager.

> [!Note]
> Para crear una identidad administrada asignada por el usuario mediante una plantilla de Azure Resource Manager, consulte [Create a user-assigned managed identity](how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity) (Creación de una identidad administrada asignada por el usuario).

### <a name="assign-a-user-assigned-managed-identity-to-a-virtual-machine-scale-set"></a>Asignación de una identidad administrada asignada por el usuario a un conjunto de escalado de máquinas virtuales

1. En el elemento `resources`, agregue la siguiente entrada para asignar una identidad administrada asignada por el usuario al conjunto de escalado de máquinas virtuales.  No olvide reemplazar `<USERASSIGNEDIDENTITY>` con el nombre de la identidad administrada asignada por el usuario que ha creado.

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2018-06-01**

   Si apiVersion es `2018-06-01`, las identidades administradas asignadas por el usuario se almacenan en el formato de diccionario `userAssignedIdentities` y el valor `<USERASSIGNEDIDENTITYNAME>` debe almacenarse en una variable definida en la sección `variables` de la plantilla.

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2018-06-01",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "userAssigned",
           "userAssignedIdentities": {
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
           }
       }

   }
   ```   

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2017-12-01**

   Si `apiVersion` es `2017-12-01` o anterior, las identidades administradas asignadas por el usuario se almacenan en la matriz `identityIds` y el valor `<USERASSIGNEDIDENTITYNAME>` debe almacenarse en una variable definida en la sección de variables de la plantilla.

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2017-03-30",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "userAssigned",
           "identityIds": [
               "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITY>'))]"
           ]
       }

   }
   ```
> [!NOTE]
> De forma opcional, puede aprovisionar las identidades administradas de extensión del conjunto de escalado de máquinas virtuales de los recursos de Azure si lo especifica en el elemento `extensionProfile` de la plantilla. Este paso es opcional, ya que puede usar el punto de conexión de identidad de Azure Instance Metadata Service (IMDS) para recuperar tokens.  Para obtener más información, consulte [Migración de la extensión de máquina virtual a IMDS de Azure para la autenticación](howto-migrate-vm-extension.md).

3. Cuando haya terminado, la plantilla debería tener un aspecto similar al siguiente:

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2018-06-01**   

   ```json
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2018-06-01",
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "name": "[variables('vmssName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "UserAssigned",
                "userAssignedIdentities": {
                    "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]": {}
                }
            },
           "properties": {
                //other virtual machine properties...
                "virtualMachineProfile": {
                    //other virtual machine profile properties...
                    //The following appears only if you provisioned the optional virtual machine scale set extension (to be deprecated)
                    "extensionProfile": {
                        "extensions": [
                            {
                                "name": "ManagedIdentityWindowsExtension",
                                "properties": {
                                  "publisher": "Microsoft.ManagedIdentity",
                                  "type": "ManagedIdentityExtensionForWindows",
                                  "typeHandlerVersion": "1.0",
                                  "autoUpgradeMinorVersion": true,
                                  "settings": {
                                      "port": 50342
                                  }
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
   ```

   **Microsoft.Compute/virtualMachines versión 2017-12-01 de la API**

   ```json
   "resources": [
        {
            //other resource provider properties...
            "apiVersion": "2017-12-01",
            "type": "Microsoft.Compute/virtualMachineScaleSets",
            "name": "[variables('vmssName')]",
            "location": "[resourceGroup().location]",
            "identity": {
                "type": "UserAssigned",
                "identityIds": [
                    "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('<USERASSIGNEDIDENTITYNAME>'))]"
                ]
            },
           "properties": {
                //other virtual machine properties...
                "virtualMachineProfile": {
                    //other virtual machine profile properties...
                    //The following appears only if you provisioned the optional virtual machine scale set extension (to be deprecated)    
                    "extensionProfile": {
                        "extensions": [
                            {
                                "name": "ManagedIdentityWindowsExtension",
                                "properties": {
                                  "publisher": "Microsoft.ManagedIdentity",
                                  "type": "ManagedIdentityExtensionForWindows",
                                  "typeHandlerVersion": "1.0",
                                  "autoUpgradeMinorVersion": true,
                                  "settings": {
                                      "port": 50342
                                  }
                                }
                            }
                        ]
                    }
                }
            }
        }
    ]
   ```
   ### <a name="remove-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Eliminación de una identidad administrada asignada por el usuario de un conjunto de escalado de máquinas virtuales de Azure

Si tiene un conjunto de escalado de máquinas virtuales que ya no necesita una identidad administrada asignada por el usuario, haga lo siguiente:

1. Independientemente de que inicie sesión localmente en Azure o mediante Azure Portal, use una cuenta que esté asociada a la suscripción de Azure que contiene el conjunto de escalado de máquinas virtuales.

2. Cargue la plantilla en un [editor](#azure-resource-manager-templates) y busque el `Microsoft.Compute/virtualMachineScaleSets`recurso de interés dentro de la sección `resources`. Si dispone de un conjunto de escalado de máquinas virtuales que solo tenga una identidad administrada asignada por el usuario, puede deshabilitarla cambiando el tipo de identidad a `None`.

   En el ejemplo siguiente se muestra cómo quitar todas las identidades administradas asignadas por un usuario de una VM sin identidades administradas asignadas por el sistema:

   ```json
   {
       "name": "[variables('vmssName')]",
       "apiVersion": "2018-06-01",
       "location": "[parameters(Location')]",
       "identity": {
           "type": "None"
        }
   }
   ```

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2018-06-01**

   Para quitar una identidad administrada asignada por un usuario único desde un conjunto de escalado de máquinas virtuales, quítela del diccionario `userAssignedIdentities`.

   Si tiene una identidad asignada por el sistema, consérvela en el valor `type` de `identity`.

   **Microsoft.Compute/virtualMachineScaleSets versión de API 2017-12-01**

   Para quitar una identidad administrada asignada por un usuario único desde un conjunto de escalado de máquinas virtuales, quítela de la matriz `identityIds`.

   Si tiene una identidad administrada asignada por el sistema, consérvela en el valor `type` de `identity`.

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre las identidades administradas de recursos de Azure](overview.md).