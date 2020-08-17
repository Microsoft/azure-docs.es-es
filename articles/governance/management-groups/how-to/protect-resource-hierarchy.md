---
title: 'Procedimiento para proteger la jerarquía de recursos: Gobernanza en Azure'
description: Obtenga información sobre cómo proteger la jerarquía de recursos con configuraciones de jerarquía que incluyen la configuración del grupo de administración predeterminado.
ms.date: 05/21/2020
ms.topic: conceptual
ms.openlocfilehash: cdaad59d136e89c595a6a42a9760c73523a977c0
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422834"
---
# <a name="how-to-protect-your-resource-hierarchy"></a>Procedimiento para proteger la jerarquía de recursos

Los recursos, grupos de recursos, suscripciones, grupos de administración y el inquilino forman la jerarquía de recursos. Los valores en el grupo de administración raíz, como roles de Azure personalizados o asignaciones de directivas de Azure Policy, pueden afectar a todos los recursos de la jerarquía. Es importante proteger la jerarquía de recursos de los cambios que podrían afectar negativamente a todos los recursos.

Ahora los grupos de administración tienen una configuración de jerarquía que permite al administrador de inquilinos controlar estos comportamientos. En este artículo se describe cada una de las configuraciones de jerarquía disponibles y cómo establecerlas.

## <a name="rbac-permissions-for-hierarchy-settings"></a>Permisos de RBAC para la configuración de jerarquía

La configuración de cualquiera de las opciones de jerarquía requiere las dos operaciones de RBAC siguientes en el grupo de administración raíz:

- `Microsoft.Management/managementgroups/settings/write`
- `Microsoft.Management/managementgroups/settings/read`

Estas operaciones solo permiten que un usuario lea y actualice la configuración de la jerarquía. Las operaciones no proporcionan ningún otro acceso a la jerarquía de grupos de administración ni a los recursos de la jerarquía. Las dos operaciones están disponibles en el rol de Azure integrado **Hierarchy Settings Administrator** (Administrador de configuración de jerarquía).

## <a name="setting---default-management-group"></a>Configuración: Grupo de administración predeterminado

De forma predeterminada, una nueva suscripción agregada dentro de un inquilino se agrega como miembro del grupo de administración raíz. Si las asignaciones de directivas, el control de acceso basado en rol (RBAC) y otras construcciones de gobernanza se asignan al grupo de administración raíz, afectan inmediatamente a estas nuevas suscripciones. Por este motivo, muchas organizaciones no aplican estas construcciones en el grupo de administración raíz, aunque es el lugar deseado para asignarlas. En otros casos, se recomienda un conjunto de controles más restrictivos para las suscripciones nuevas, pero no se deben asignar a todas las suscripciones. Este valor admite los dos casos de uso.

Al permitir que se defina el grupo de administración predeterminado para las suscripciones nuevas, se pueden aplicar construcciones de gobernanza para toda la organización en el grupo de administración raíz y definir un grupo de administración independiente con asignaciones de directivas o de roles de Azure más adecuadas para una suscripción nueva.

Para configurar este valor, se llama al punto de conexión de la API REST de [Configuración de jerarquía](/rest/api/resources/hierarchysettings). Para ello, use el siguiente formato de URI de la API REST y de cuerpo. Reemplace `{rootMgID}` por el identificador del grupo de administración raíz y `{defaultGroupID}` por el identificador del grupo de administración para que se convierta en el grupo de administración predeterminado:

- URI DE LA API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{rootMgID}/settings/default?api-version=2020-02-01
  ```

- Cuerpo de la solicitud

  ```json
  {
      "properties": {
          "defaultManagementGroup": "/providers/Microsoft.Management/managementGroups/{defaultGroupID}"
      }
  }
  ```

Para volver a establecer el grupo de administración predeterminado en el grupo de administración raíz, use el mismo punto de conexión y establezca **defaultManagementGroup** en un valor de `/providers/Microsoft.Management/managementGroups/{rootMgID}`.

## <a name="setting---require-authorization"></a>Configuración: Requerir autorización

De forma predeterminada, cualquier usuario puede crear grupos de administración dentro de un inquilino. Es posible que los administradores de un inquilino solo quieran proporcionar estos permisos a usuarios específicos para mantener la coherencia y la conformidad en la jerarquía de grupos de administración. Si está habilitada, un usuario requiere la operación `Microsoft.Management/managementGroups/write` en el grupo de administración raíz para crear grupos de administración secundarios.

Para configurar este valor, se llama al punto de conexión de la API REST de [Configuración de jerarquía](/rest/api/resources/hierarchysettings). Para ello, use el siguiente formato de URI de la API REST y de cuerpo. Este es un valor _booleano_, por lo que debe proporcionarle **true** o **false**. Un valor de **true** habilita este método de protección de la jerarquía de grupos de administración:

- URI DE LA API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{rootMgID}/settings/default?api-version=2020-02-01
  ```

- Cuerpo de la solicitud

  ```json
  {
      "properties": {
          "requireAuthorizationForGroupCreation": true
      }
  }
  ```

Para volver a desactivar la configuración, use el mismo punto de conexión y establezca **requireAuthorizationForGroupCreation** en un valor de **false**.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los grupos de administración, consulte:

- [Creación de grupos de administración para organizar los recursos de Azure](../create.md)
- [Cambio, eliminación y administración de los grupos de administración](../manage.md)
