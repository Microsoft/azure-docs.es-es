---
title: 'Inicio rápido: creación de un grupo de administración con la CLI de Azure'
description: En este inicio rápido, se usa la CLI de Azure para crear un grupo de administración para organizar los recursos en una jerarquía de recursos.
ms.date: 08/31/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: f07ae46c95f9ab9cc1ad973204ac5c50320fdf46
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237136"
---
# <a name="quickstart-create-a-management-group-with-the-azure-cli"></a>Inicio rápido: creación de un grupo de administración con la CLI de Azure

Los grupos de administración son contenedores que ayudan a administran el acceso, las directivas y el cumplimiento de varias suscripciones. Cree estos contenedores para construir una jerarquía eficaz y eficiente que pueda usarse con [Azure Policy](../policy/overview.md) y los [controles de acceso basados en roles de Azure](../../role-based-access-control/overview.md). Para obtener más información sobre los grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](overview.md).

El primer grupo de administración creado en el directorio puede tardar hasta 15 minutos en completarse. Hay procesos que se ejecutan la primera vez para configurar el servicio de grupos de administración dentro de Azure para su directorio. Recibirá una notificación cuando finalice el proceso. Para obtener más información, vea [Instalación inicial de los grupos de administración](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Este inicio rápido requiere la ejecución de la versión 2.0.76 de la CLI de Azure, o cualquier versión posterior, para instalar y usar la CLI en un entorno local. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

- Si la [protección de jerarquías](./how-to/protect-resource-hierarchy.md#setting---require-authorization) no está habilitada, cualquier usuario de Azure AD del inquilino puede crear un grupo de administración sin tener el permiso de escritura del grupo de administración asignado. Este nuevo grupo de administración pasa a ser un elemento secundario del grupo de administración raíz o del [grupo de administración predeterminado](./how-to/protect-resource-hierarchy.md#setting---default-management-group) y al creador se le asigna el rol de "Propietario". El servicio del grupo de administración ofrece esta funcionalidad para que las asignaciones de roles no sean necesarias en el nivel raíz. Ningún usuario tiene acceso al grupo de administración raíz cuando se crea. Para sortear el obstáculo de buscar administradores globales de Azure AD para empezar a usar grupos de administración, se permite la creación de grupos de administración iniciales en el nivel raíz.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-in-the-azure-cli"></a>Creación en la CLI de Azure

Para la CLI de Azure, use el comando [az account management-group create](/cli/azure/account/management-group#az-account-management-group-create) para crear un nuevo grupo de administración. En este ejemplo, el **nombre** del grupo de administración es _Contoso_.

```azurecli-interactive
az account management-group create --name 'Contoso'
```

El **nombre** es un identificador único que se crea. Este identificador lo usan otros comandos para hacer referencia a este grupo y no se puede cambiar más adelante.

Si quiere que el grupo de administración muestre un nombre diferente en Azure Portal, agregue el parámetro **display-name**. Por ejemplo, para crear un grupo de administración con el nombre de grupo de Contoso y el nombre para mostrar de "Grupo de Contoso", use el siguiente comando:

```azurecli-interactive
az account management-group create --name 'Contoso' --display-name 'Contoso Group'
```

En los ejemplos anteriores, se crea el nuevo grupo de administración en el grupo de administración raíz. Para especificar otro grupo de administración como el elemento primario, use el parámetro **parent** y proporcione el nombre del grupo principal.

```azurecli-interactive
az account management-group create --name 'ContosoSubGroup' --parent 'Contoso'
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar el grupo de administración que ha creado anteriormente, use el comando [az account management-group delete](/cli/azure/account/management-group#az-account-management-group-delete):

```azurecli-interactive
az account management-group delete --name 'Contoso'
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de administración para organizar una jerarquía de recursos. El grupo de administración puede contener suscripciones u otros grupos de administración.

Para más información acerca de los grupos de administración y cómo administrar la jerarquía de recursos, vaya a:

> [!div class="nextstepaction"]
> [Administración de los recursos con grupos de administración](./manage.md)