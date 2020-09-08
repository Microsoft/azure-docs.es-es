---
title: 'Inicio rápido: Creación de un grupo de administración con la API REST'
description: En este inicio rápido, se usa la API REST para crear un grupo de administración para organizar los recursos en una jerarquía de recursos.
ms.date: 08/31/2020
ms.topic: quickstart
ms.openlocfilehash: b19fddf8215a1b133254c2a31bbea568a315f721
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237144"
---
# <a name="quickstart-create-a-management-group-with-rest-api"></a>Inicio rápido: Creación de un grupo de administración con la API REST

Los grupos de administración son contenedores que ayudan a administran el acceso, las directivas y el cumplimiento de varias suscripciones. Cree estos contenedores para construir una jerarquía eficaz y eficiente que pueda usarse con [Azure Policy](../policy/overview.md) y los [controles de acceso basados en roles de Azure](../../role-based-access-control/overview.md). Para obtener más información sobre los grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](overview.md).

El primer grupo de administración creado en el directorio puede tardar hasta 15 minutos en completarse. Hay procesos que se ejecutan la primera vez para configurar el servicio de grupos de administración dentro de Azure para su directorio. Recibirá una notificación cuando finalice el proceso. Para obtener más información, vea [Instalación inicial de los grupos de administración](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Si aún no lo ha hecho, instale [ARMClient](https://github.com/projectkudu/ARMClient). Se trata de una herramienta que envía solicitudes HTTP a las API REST basadas en Azure Resource Manager. Como alternativa, puede usar la característica "Pruébelo" de la documentación de REST o herramientas como [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) o [Postman](https://www.postman.com)de PowerShell.

- Si la [protección de jerarquías](./how-to/protect-resource-hierarchy.md#setting---require-authorization) no está habilitada, cualquier usuario de Azure AD del inquilino puede crear un grupo de administración sin tener el permiso de escritura del grupo de administración asignado. Este nuevo grupo de administración pasa a ser un elemento secundario del grupo de administración raíz o del [grupo de administración predeterminado](./how-to/protect-resource-hierarchy.md#setting---default-management-group) y al creador se le asigna el rol de "Propietario". El servicio del grupo de administración ofrece esta funcionalidad para que las asignaciones de roles no sean necesarias en el nivel raíz. Ningún usuario tiene acceso al grupo de administración raíz cuando se crea. Para sortear el obstáculo de buscar administradores globales de Azure AD para empezar a usar grupos de administración, se permite la creación de grupos de administración iniciales en el nivel raíz.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-in-rest-api"></a>Creación en la API REST

En el caso de la API REST, use el punto de conexión [Grupos de administración: crear o actualizar](/rest/api/resources/managementgroups/createorupdate) para crear un grupo de administración. En este ejemplo, el valor de **groupId** del grupo de administración es _Contoso_.

- URI DE LA API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Sin cuerpo de la solicitud

**groupId** es un identificador único que se crea. Este identificador lo usan otros comandos para hacer referencia a este grupo y no se puede cambiar más adelante.

Si desea que el grupo de administración muestre un nombre diferente en Azure Portal, agregue la propiedad **properties.displayName** al cuerpo de la solicitud. Por ejemplo, para crear un grupo de administración con el valor de **groupId** _Contoso_ y el nombre para mostrar de _Grupo de Contoso_, use el siguiente punto de conexión y el siguiente cuerpo de la solicitud:

- URI DE LA API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Cuerpo de la solicitud

  ```json
  {
    "properties": {
      "displayName": "Contoso Group"
    }
  }
  ```

En los ejemplos anteriores, se crea el nuevo grupo de administración en el grupo de administración raíz. Para especificar otro grupo de administración como primario, use la propiedad **properties.parent.id**.

- URI DE LA API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Cuerpo de la solicitud

  ```json
  {
    "properties": {
      "displayName": "Contoso Group",
      "parent": {
        "id": "/providers/Microsoft.Management/managementGroups/HoldingGroup"
      }
    }
  }
  ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar el grupo de administración que ha creado anteriormente, use el punto de conexión[Grupos de administración: eliminar](/rest/api/resources/managementgroups/delete):

- URI DE LA API REST

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/Contoso?api-version=2020-02-01
  ```

- Sin cuerpo de la solicitud

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de administración para organizar una jerarquía de recursos. El grupo de administración puede contener suscripciones u otros grupos de administración.

Para más información acerca de los grupos de administración y cómo administrar la jerarquía de recursos, vaya a:

> [!div class="nextstepaction"]
> [Administración de los recursos con grupos de administración](./manage.md)