---
title: Configuración del control de acceso basado en roles (RBAC) local para Azure API for FHIR
description: En este artículo se explica cómo configurar Azure API for FHIR para usar un inquilino de Azure AD externo para el plano de datos
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 03/15/2020
ms.author: matjazl
ms.openlocfilehash: c62593251cb0e19c91d1c4877d1b33fe407ebd06
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87847013"
---
# <a name="configure-local-rbac-for-fhir"></a>Configuración de RBAC local para FHIR 

En este artículo se explica cómo configurar Azure API for FHIR para usar un inquilino de Azure Active Directory secundario externo para la administración del acceso al plano de datos. Use este modo solo si no es posible usar el inquilino de Azure Active Directory asociado a la suscripción.

> [!NOTE]
> Si el plano de datos del servicio FHIR está configurado para usar el inquilino de Azure Active Directory principal asociado a la suscripción, [use RBAC de Azure para asignar roles de plano de datos](configure-azure-rbac.md).

## <a name="add-service-principal"></a>Agregar entidad de servicio

RBAC local permite usar un inquilino de Azure Active Directory externo con el servidor de FHIR. Para que el sistema RBAC pueda comprobar las pertenencias a grupos en este inquilino, Azure API for FHIR debe tener una entidad de servicio en el inquilino. Esta entidad de servicio se crea automáticamente en los inquilinos vinculados a las suscripciones que han implementado Azure API for FHIR, pero si el inquilino no tiene ninguna suscripción vinculada, un administrador de inquilinos tiene que crear esta entidad de servicio con uno de los siguientes comandos:

Con el módulo de PowerShell `Az`:

```azurepowershell-interactive
New-AzADServicePrincipal -ApplicationId 3274406e-4e0a-4852-ba4f-d7226630abb7
```

O bien puede usar el módulo `AzureAd` de PowerShell:

```azurepowershell-interactive
New-AzureADServicePrincipal -AppId 3274406e-4e0a-4852-ba4f-d7226630abb7
```

O bien puede usar la CLI de Azure:

```azurecli-interactive
az ad sp create --id 3274406e-4e0a-4852-ba4f-d7226630abb7
```

## <a name="configure-local-rbac"></a>Configuración de RBAC local

Puede configurar Azure API for FHIR para que use un inquilino de Azure Active Directory externo o secundario en la hoja **Autenticación**:

![Asignaciones de RBAC local](media/rbac/local-rbac-guids.png).

En el cuadro Autoridad, escriba un inquilino de Azure Active Directory válido. Una vez validado el inquilino, el cuadro **Identificadores de objeto permitidos** debe estar activado y se puede especificar una lista de identificadores de objeto de identidad. Estos identificadores pueden ser los identificadores de objeto de identidad de:

* Un usuario de Azure Active Directory.
* Una entidad de servicio de Azure Active Directory.
* Un grupo de seguridad de Azure Active Directory.

Puede leer el artículo sobre la [búsqueda de identificadores de objetos de identidad](find-identity-object-ids.md) para obtener más detalles.

Después de especificar los identificadores de objeto necesarios, haga clic en **Guardar** y espere a que se guarden los cambios antes de intentar acceder al plano de datos mediante los usuarios asignados, las entidades de servicio o los grupos.

## <a name="caching-behavior"></a>Comportamiento del almacenamiento en caché

Azure API for FHIR almacena en caché las decisiones durante un máximo de 5 minutos. Si concede a un usuario acceso al servidor de FHIR al agregarlo a la lista de identificadores de objeto permitidos, o lo quita de la lista, debe dejar hasta cinco minutos para que se propaguen los cambios de permisos.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido a asignar acceso al plano de datos de FHIR mediante un inquilino externo (secundario) de Azure Active Directory. A continuación obtenga más información sobre la configuración adicional para Azure API for FHIR:
 
>[!div class="nextstepaction"]
>[Configuración adicional de Azure API for FHIR](azure-api-for-fhir-additional-settings.md)
