---
title: 'Inicio rápido: Creación de un grupo de administración con Azure Portal'
description: En este inicio rápido, se usa Azure Portal para crear un grupo de administración para organizar los recursos en una jerarquía de recursos.
ms.date: 08/31/2020
ms.topic: quickstart
ms.openlocfilehash: 28b530e1da6b75dcc20e4a5dfebc494a959040b3
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237096"
---
# <a name="quickstart-create-a-management-group"></a>Inicio rápido: Creación de un grupo de administración

Los grupos de administración son contenedores que ayudan a administran el acceso, las directivas y el cumplimiento de varias suscripciones. Cree estos contenedores para construir una jerarquía eficaz y eficiente que pueda usarse con [Azure Policy](../policy/overview.md) y los [controles de acceso basados en roles de Azure](../../role-based-access-control/overview.md). Para obtener más información sobre los grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](overview.md).

El primer grupo de administración creado en el directorio puede tardar hasta 15 minutos en completarse. Hay procesos que se ejecutan la primera vez para configurar el servicio de grupos de administración dentro de Azure para su directorio. Recibirá una notificación cuando finalice el proceso. Para obtener más información, vea [Instalación inicial de los grupos de administración](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Si la [protección de jerarquías](./how-to/protect-resource-hierarchy.md#setting---require-authorization) no está habilitada, cualquier usuario de Azure AD del inquilino puede crear un grupo de administración sin tener el permiso de escritura del grupo de administración asignado. Este nuevo grupo de administración pasa a ser un elemento secundario del grupo de administración raíz o del [grupo de administración predeterminado](./how-to/protect-resource-hierarchy.md#setting---default-management-group) y al creador se le asigna el rol de "Propietario". El servicio del grupo de administración ofrece esta funcionalidad para que las asignaciones de roles no sean necesarias en el nivel raíz. Ningún usuario tiene acceso al grupo de administración raíz cuando se crea. Para sortear el obstáculo de buscar administradores globales de Azure AD para empezar a usar grupos de administración, se permite la creación de grupos de administración iniciales en el nivel raíz.

### <a name="create-in-portal"></a>Crear en el portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Todos los servicios** > **Administración y gobernanza**.

1. Seleccione **Grupos de administración**.

1. Seleccione **+ Agregar grupo de administración**.

   :::image type="content" source="./media/main.png" alt-text="Página para trabajar con grupos de administración" border="false":::

1. Deje la opción **Crear nuevo** seleccionada y rellene el campo del identificador del grupo de administración.

   - El **identificador de grupo de administración** es el identificador único de directorio que se usa para enviar comandos en este grupo de administración. Este identificador no es puede editar después de la creación, ya que se usa en todo el sistema de Azure para identificar este grupo. El [grupo de administración raíz](./overview.md#root-management-group-for-each-directory) se crea automáticamente con un ID que es el identificador de Azure Active Directory. Para el resto de grupos de administración, asigne un identificador único.
   - El campo de nombre para mostrar es el nombre que se muestra en Azure Portal. Un nombre para mostrar independiente es un campo opcional al crear el grupo de administración y se puede cambiar en cualquier momento.

   :::image type="content" source="./media/create_context_menu.png" alt-text="Panel de opciones para crear un nuevo grupo de administración":::

1. Seleccione **Guardar**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el grupo de administración que se ha creado, siga estos pasos:

1. Seleccione **Todos los servicios** > **Administración y gobernanza**.

1. Seleccione **Grupos de administración**.

1. Busque el grupo de administración creado anteriormente, selecciónelo y, después, seleccione **Detalles** junto al nombre.
   Luego, seleccione **Eliminar** y confirme la solicitud.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de administración para organizar una jerarquía de recursos. El grupo de administración puede contener suscripciones u otros grupos de administración.

Para más información acerca de los grupos de administración y cómo administrar la jerarquía de recursos, vaya a:

> [!div class="nextstepaction"]
> [Administración de los recursos con grupos de administración](./manage.md)