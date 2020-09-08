---
title: 'Cambio de la pertenencia a un grupo estático a dinámico: Azure AD | Microsoft Docs'
description: Creación de reglas de pertenencia para rellenar automáticamente grupos y creación de una referencia de regla.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 04/29/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: e6ee5af5d7489835f125319050d291a89ab252a6
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797129"
---
# <a name="change-static-group-membership-to-dynamic-in-azure-active-directory"></a>Cambio de la pertenencia de un grupo estático a dinámico en Azure Active Directory

Puede cambiar la pertenencia de un grupo de estático a dinámico (o viceversa) en Azure Active Directory (Azure AD). Azure AD mantiene el mismo nombre e identificador del grupo en el sistema, por lo que todas las referencias existentes al grupo siguen siendo válidas. Si crea un nuevo grupo en su lugar, deberá actualizar esas referencias. La pertenencia a grupos dinámicos elimina la sobrecarga de administración al agregar y quitar usuarios. Este artículo explica cómo convertir los grupos existentes de pertenencia estática a dinámica mediante el centro de administración de Azure AD o cmdlets de PowerShell.

> [!WARNING]
> Al cambiar un grupo estático existente a uno dinámico, se eliminarán todos sus miembros y, a continuación, se procesará la regla de pertenencia para agregar nuevos miembros. Si el grupo se usa para controlar el acceso a aplicaciones o recursos, tenga en cuenta que los miembros originales podrían perder el acceso hasta que se procese por completo la regla de pertenencia.
>
> Es recomendable que pruebe la nueva regla de pertenencia con antelación para asegurarse de que la nueva pertenencia al grupo es la que se preveía.

## <a name="change-the-membership-type-for-a-group"></a>Cambio del tipo de pertenencia de un grupo

1. Iniciar sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta que sea administrador global o administrador de usuarios de la organización de Azure AD.
2. Seleccione **Grupos**.
3. En la lista **Todos los grupos**, abra el grupo que desea cambiar.
4. Seleccione **Propiedades**.
5. En la página **Propiedades** del grupo, seleccione un **Tipo de pertenencia**, ya sea Asignada (estática), Usuario dinámico o Dispositivo dinámico, según el tipo de pertenencia que desee. Para la pertenencia dinámica, puede usar el generador de reglas para seleccionar opciones para una regla sencilla o escribir usted mismo una regla de pertenencia. 

Los pasos siguientes son un ejemplo de cómo cambiar un grupo de usuarios para que pase de una pertenencia estática a una dinámica.

1. En la página **Propiedades** del grupo seleccionado, elija un **tipo de pertenencia** de **Usuario dinámico** y, a continuación, seleccione Sí en el cuadro de diálogo que explica los cambios en la pertenencia del grupo para continuar. 
  
   ![seleccionar tipo de pertenencia de usuario dinámico](./media/groups-change-type/select-group-to-convert.png)
  
2. Seleccione **Agregar una consulta dinámica** y, a continuación, proporcione la regla.
  
   ![Especificar la regla del grupo dinámico](./media/groups-change-type/enter-rule.png)
  
3. Después de crear la regla, seleccione **Agregar consulta** en la parte inferior de la página.
4. Seleccione **Guardar** en la página **Propiedades** del grupo para guardar los cambios. El **tipo de pertenencia** del grupo se actualiza inmediatamente en la lista de grupos.

> [!TIP]
> Es posible que la conversión del grupo produzca un error si la regla de pertenencia que escribió no era correcta. Aparecerá una notificación en la esquina superior derecha del portal, con una explicación de por qué el sistema no pudo aceptar la regla. Léala con cuidado para saber cómo debe ajustar la regla para que sea válida. Para ver ejemplos de sintaxis de reglas y una lista completa de las propiedades, operadores y valores admitidos para una regla de pertenencia, consulte [Reglas de pertenencia a grupos dinámicos en Azure Active Directory](groups-dynamic-membership.md).

## <a name="change-membership-type-for-a-group-powershell"></a>Cambio del tipo de pertenencia de un grupo (PowerShell)

> [!NOTE]
> Para cambiar las propiedades de un grupo dinámico, es preciso usar los cmdlets de **la versión preliminar de** [Azure AD PowerShell, versión 2](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0). Puede instalar la versión preliminar desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureADPreview).

Este es un ejemplo e las funciones que cambian la administración de la pertenencia a un grupo existente. En este ejemplo, se actúa con precaución para manipular correctamente la propiedad GroupTypes y preservar todos los valores que no tienen relación con la pertenencia a un grupo dinámico.

```powershell
#The moniker for dynamic groups as used in the GroupTypes property of a group object
$dynamicGroupTypeString = "DynamicMembership"

function ConvertDynamicGroupToStatic
{
    Param([string]$groupId)

    #existing group types
    [System.Collections.ArrayList]$groupTypes = (Get-AzureAdMsGroup -Id $groupId).GroupTypes

    if($groupTypes -eq $null -or !$groupTypes.Contains($dynamicGroupTypeString))
    {
        throw "This group is already a static group. Aborting conversion.";
    }


    #remove the type for dynamic groups, but keep the other type values
    $groupTypes.Remove($dynamicGroupTypeString)

    #modify the group properties to make it a static group: i) change GroupTypes to remove the dynamic type, ii) pause execution of the current rule
    Set-AzureAdMsGroup -Id $groupId -GroupTypes $groupTypes.ToArray() -MembershipRuleProcessingState "Paused"
}

function ConvertStaticGroupToDynamic
{
    Param([string]$groupId, [string]$dynamicMembershipRule)

    #existing group types
    [System.Collections.ArrayList]$groupTypes = (Get-AzureAdMsGroup -Id $groupId).GroupTypes

    if($groupTypes -ne $null -and $groupTypes.Contains($dynamicGroupTypeString))
    {
        throw "This group is already a dynamic group. Aborting conversion.";
    }
    #add the dynamic group type to existing types
    $groupTypes.Add($dynamicGroupTypeString)

    #modify the group properties to make it a static group: i) change GroupTypes to add the dynamic type, ii) start execution of the rule, iii) set the rule
    Set-AzureAdMsGroup -Id $groupId -GroupTypes $groupTypes.ToArray() -MembershipRuleProcessingState "On" -MembershipRule $dynamicMembershipRule
}
```
Para convertir un grupo en estático:

```powershell
ConvertDynamicGroupToStatic "a58913b2-eee4-44f9-beb2-e381c375058f"
```

Para convertir un grupo en dinámico:

```powershell
ConvertStaticGroupToDynamic "a58913b2-eee4-44f9-beb2-e381c375058f" "user.displayName -startsWith ""Peter"""
```

## <a name="next-steps"></a>Pasos siguientes

En estos artículos se proporciona información adicional sobre los grupos en Azure Active Directory.

* [Consulta de los grupos existentes](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Crear un nuevo grupo y agregar miembros](../fundamentals/active-directory-groups-create-azure-portal.md)
* [Administración de la configuración de un grupo](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [Administrar la pertenencia a grupos](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [Administrar reglas dinámicas de los usuarios de un grupo](groups-dynamic-membership.md)