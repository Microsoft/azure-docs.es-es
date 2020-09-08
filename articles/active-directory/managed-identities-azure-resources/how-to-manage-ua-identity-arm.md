---
title: Creación y eliminación de una identidad administrada asignada por el usuario mediante Azure Resource Manager
description: Instrucciones paso a paso sobre cómo crear y eliminar identidades administradas asignadas por el usuario mediante Azure Resource Manager.
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
ms.date: 12/10/2019
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: bd49c60a23a0ad58f0907d3c1d7c38178ac08c97
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266364"
---
# <a name="create-list-and-delete-a-user-assigned-managed-identity-using-azure-resource-manager"></a>Creación, enumeración y eliminación de una identidad administrada asignada por el usuario mediante Azure Resource Manager


Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo creará una identidad administrada asignada por el usuario mediante Azure Resource Manager.

No es posible enumerar y eliminar una identidad administrada asignada por el usuario mediante una plantilla de Azure Resource Manager.  Consulte los artículos siguientes para crear y enumerar una identidad administrada asignada por el usuario:

- [Enumeración de identidades administradas asignadas por el usuario](how-to-manage-ua-identity-cli.md#list-user-assigned-managed-identities)
- [Eliminación de identidades administradas asignadas por el usuario](how-to-manage-ua-identity-cli.md#delete-a-user-assigned-managed-identity)
  ## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="template-creation-and-editing"></a>Creación y edición de una plantilla

Como con Azure Portal y los scripts, las plantillas de Azure Resource Manager proporcionan la capacidad de implementar recursos nuevos o modificados definidos con un grupo de recursos de Azure. Existen varias opciones para la edición e implementación de plantillas, tanto localmente como basadas en el portal, incluidas:

- Usar una [plantilla personalizada de Azure Marketplace](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), que permite crear una plantilla desde cero, o bien basada en una plantilla común existente o en una [plantilla de inicio rápido](https://azure.microsoft.com/documentation/templates/).
- Derivar a partir de un grupo de recursos existente, exportando una plantilla de [la implementación original](../../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates) o del [estado actual de la implementación](../../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates).
- Usar un [editor de JSON (por ejemplo, VS Code)](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) local y, a continuación, cargarla e implementarla con PowerShell o la CLI.
- Usar el [proyecto del grupo de recursos de Azure](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) de Visual Studio tanto para crear como para implementar una plantilla. 

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para crear una identidad administrada asignada por el usuario, use la siguiente plantilla. Reemplace el valor de `<USER ASSIGNED IDENTITY NAME>` por sus propios valores:

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "resourceName": {
          "type": "string",
          "metadata": {
            "description": "<USER ASSIGNED IDENTITY NAME>"
          }
        }
  },
  "resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('resourceName')]",
      "apiVersion": "2018-11-30",
      "location": "[resourceGroup().location]"
    }
  ],
  "outputs": {
      "identityName": {
          "type": "string",
          "value": "[parameters('resourceName')]"
      }
  }
}
```
## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo asignar una identidad administrada asignada por el usuario a una VM de Azure mediante una plantilla de Azure Resource Manager, consulte [Configure managed identities for Azure resources on an Azure VM using a template](qs-configure-template-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante una plantilla).


