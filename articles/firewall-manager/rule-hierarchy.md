---
title: Uso de la directiva de Azure Firewall para definir una jerarquía de reglas
description: Obtenga información sobre cómo usar la directiva de Azure Firewall para definir una jerarquía de reglas y exigir el cumplimiento.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: how-to
ms.date: 08/26/2020
ms.author: victorh
ms.openlocfilehash: c290904c9f4bc7dba70dad9351dc45b676e0c236
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88893632"
---
# <a name="use-azure-firewall-policy-to-define-a-rule-hierarchy"></a>Uso de la directiva de Azure Firewall para definir una jerarquía de reglas

Los administradores de seguridad tienen que administrar los firewalls y garantizar el cumplimiento en las implementaciones locales y en la nube. Un componente clave es la posibilidad de proporcionar a los equipos de la aplicación la flexibilidad necesaria para implementar canalizaciones de CI/CD a fin de crear las reglas de firewall de forma automática.

La directiva de Azure Firewall le permite definir una jerarquía de reglas y exigir el cumplimiento:

- Proporciona una estructura jerárquica para superponer una directiva base central a partir de una directiva de equipo de aplicación secundaria. La directiva base tiene una prioridad más alta y se ejecuta antes que la secundaria.
- Use una definición de control de acceso basado en rol para evitar la eliminación involuntaria de las directivas base y proporcionar acceso selectivo a los grupos de la colección de reglas dentro de una suscripción o un grupo de recursos. 

## <a name="solution-overview"></a>Información general de la solución

Los pasos generales para este ejemplo son:

1. Cree una directiva de firewall base en el grupo de recursos del equipo de seguridad. 
3. Defina reglas específicas de seguridad de TI en la directiva base. De esta forma, agrega un conjunto común de reglas para permitir o denegar el tráfico.
4. Cree directivas de equipo de la aplicación que hereden la directiva base. 
5. Defina reglas específicas del equipo de la aplicación en la directiva. También puede migrar las reglas de firewalls ya existentes.
6. Cree roles personalizados de Azure Active Directory para proporcionar acceso específico al grupo de recopilación de reglas y agregar roles en un ámbito de directiva de firewall. En el ejemplo siguiente, los miembros del equipo de ventas pueden editar los grupos de recopilación de reglas para la directiva de firewall de los equipos de ventas. Lo mismo se aplica a los equipos de ingeniería y de base de datos.
7. Asocie la directiva al firewall correspondiente. Un firewall de Azure solo puede tener asignada una directiva. Esto requiere que cada equipo de aplicación tenga su propio firewall.



:::image type="content" source="media/rule-hierarchy/contoso-teams.png" alt-text="Equipos y requisitos" border="false":::

### <a name="create-the-firewall-policies"></a>Creación de las directivas de firewall

- Una directiva base de firewall.

Cree directivas para cada uno de los equipos de la aplicación:

- Una directiva de firewall de ventas. La directiva de firewall de ventas hereda la directiva de firewall base.
- Una directiva de firewall de base de datos. La directiva de firewall de base de datos hereda la directiva de firewall base.
- Una directiva de firewall de ingeniería. La directiva de firewall de ingeniería también hereda la directiva de firewall base.

:::image type="content" source="media/rule-hierarchy/policy-hierarchy.png" alt-text="Jerarquía de directivas" border="false":::

### <a name="create-custom-roles-to-access-the-rule-collection-groups"></a>Creación de roles personalizados para acceder a los grupos de la colección de reglas 

Los roles personalizados se definen para cada equipo de la aplicación. El rol define las operaciones y el ámbito. Los equipos de la aplicación pueden editar los grupos de la colección de reglas para sus aplicaciones respectivas.

Use el siguiente procedimiento general para definir roles personalizados:

1. Obtenga la suscripción:

   `Select-AzSubscription -SubscriptionId xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx`
2. Ejecute el siguiente comando:

   `Get-AzProviderOperation "Microsoft.Support/*" | FT Operation, Description -AutoSize`
3. Use el comando Get-AzRoleDefinition para obtener la salida del rol Lector en formato JSON. 

   `Get-AzRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\CustomRoles\ReaderSupportRole.json`
4. Abra el archivo ReaderSupportRole.json en un editor.

   A continuación, se muestra la salida JSON. Para más información sobre las distintas propiedades, consulte  [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).

```json
   { 
     "Name": "Reader", 
     "Id": "acdd72a7-3385-48ef-bd42-f606fba81ae7", 
     "IsCustom": false, 
     "Description": "Lets you view everything, but not make any changes.", 
     "Actions": [ 
      "*/read" 
     ], 
     "NotActions": [], 
     "DataActions": [], 
     "NotDataActions": [], 
     "AssignableScopes": [ 
       "/" 
     ] 
   } 
```
5. Edite el archivo JSON para agregar la 

   `*/read", "Microsoft.Network/*/read", "Microsoft.Network/firewallPolicies/ruleCollectionGroups/write` 

   operación a la propiedad  **Actions** . Asegúrese de incluir una coma después de la operación de lectura. Esta acción permite al usuario crear y actualizar los grupos de la colección de reglas.
6. En  **AssignableScopes**, agregue el identificador de suscripción con el formato siguiente: 

   `/subscriptions/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx`

   Debe agregar identificadores de suscripción explícitos; en caso contrario, no podrá importar el rol en su suscripción.
7. Elimine la línea de la propiedad  **Id**  y cambie la propiedad  **IsCustom** a true.
8. Cambie las propiedades  **Name**  y  **Description** a *AZFM Rule Collection Group Author* (Autor del grupo de la colección de reglas AZFM) y *Users in this role can edit Firewall Policy rule collection groups* (Los usuarios de este rol pueden editar los grupos de colecciones de reglas de la directiva de firewall).

El archivo JSON debe tener una apariencia similar a la del ejemplo siguiente:

```
{ 

    "Name":  "AZFM Rule Collection Group Author", 
    "IsCustom":  true, 
    "Description":  "Users in this role can edit Firewall Policy rule collection groups", 
    "Actions":  [ 
                    "*/read", 
                    "Microsoft.Network/*/read", 
                     "Microsoft.Network/firewallPolicies/ruleCollectionGroups/write" 
                ], 
    "NotActions":  [ 
                   ], 
    "DataActions":  [ 
                    ], 
    "NotDataActions":  [ 
                       ], 
    "AssignableScopes":  [ 
                             "/subscriptions/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxx"] 
} 
```
9. Para crear el nuevo rol personalizado, use el comando New-AzRoleDefinition y especifique el archivo de definición de rol JSON. 

   `New-AzRoleDefinition -InputFile "C:\CustomRoles\RuleCollectionGroupRole.json`

### <a name="list-custom-roles"></a>Lista de roles personalizados

Para enumerar todos los roles personalizados, use el comando Get-AzRoleDefinition:

   `Get-AzRoleDefinition | ? {$_.IsCustom -eq $true} | FT Name, IsCustom`

También puede ver el rol personalizado en Azure Portal. Vaya a su suscripción, seleccione **Control de acceso (IAM)** , **Roles**.

:::image type="content" source="media/rule-hierarchy/sales-app-policy.png" alt-text="SalesAppPolicy":::

:::image type="content" source="media/rule-hierarchy/sales-app-policy-read.png" alt-text="Permiso de lectura SalesAppPolicy":::

Para más información, consulte el [Tutorial: Creación de un rol personalizado para los recursos de Azure con Azure PowerShell](../role-based-access-control/tutorial-custom-role-powershell.md).

### <a name="add-users-to-the-custom-role"></a>Adición de usuarios al rol personalizado

En el portal, puede agregar usuarios al rol AZFM Rule Collection Group Authors (Autores del grupo de la colección de reglas AZFM) y proporcionar acceso a las directivas de firewall.

1. En el portal, seleccione la directiva de firewall del equipo de la aplicación (por ejemplo, SalesAppPolicy).
2. Seleccione **Control de acceso**.
3. Seleccione **Agregar asignación de roles**.
4. Agregue usuarios o grupos de usuarios (por ejemplo, el equipo de ventas) al rol.

Repita este procedimiento para las demás directivas de firewall.

### <a name="summary"></a>Resumen

La directiva de firewall con RBAC personalizado ahora proporciona acceso selectivo a los grupos de recopilación de reglas de directivas de firewall.

Los usuarios no tienen permisos para:
- Eliminar la directiva de firewall o Azure Firewall.
- Actualizar la jerarquía de directivas de firewall, la configuración DNS ni la inteligencia sobre amenazas.
- Actualizar la directiva de firewall en la que no sean miembros del grupo de autores de grupos de recopilación de reglas AZFM.

Los administradores de seguridad pueden usar la directiva base para aplicar barreras y bloquear determinados tipos de tráfico (por ejemplo, ICMP) según lo requiera su empresa. 

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [directiva de Azure Firewall](policy-overview.md).

