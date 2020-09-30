---
title: 'Cambio de la configuración de solicitud para un paquete de acceso en la administración de derechos de Azure AD: Azure Active Directory'
description: Aprenda a cambiar la configuración de solicitud de un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 09/16/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 160137ca2d14e0012a524ee0818b7fb269fa7984
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90980292"
---
# <a name="change-request-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Cambio de la configuración de solicitud de un paquete de acceso en la administración de derechos de Azure AD

Como administrador de paquetes de acceso, puede cambiar los usuarios que pueden solicitar un paquete de acceso en cualquier momento si edita la directiva o agrega una nueva. En este artículo, se explica cómo cambiar la configuración de solicitud de un paquete de acceso existente.

## <a name="choose-between-one-or-multiple-policies"></a>Elección entre una o varias directivas

Para especificar quién puede solicitar un paquete de acceso, debe usar una directiva. Antes de crear una directiva o editar una directiva existente en un paquete de acceso, debe determinar cuántas directivas necesita el paquete de acceso. 

Cuando se crea un paquete de acceso, se especifica la configuración de solicitud que crea una directiva. La mayoría de los paquetes de acceso tendrán una sola directiva, pero un único paquete de acceso puede tener varias. Se crearían varias directivas para un único paquete de acceso si deseara permitir que se concedan asignaciones a los distintos conjuntos de usuarios con una configuración de solicitudes y aprobación diferente. 

Por ejemplo, no se puede usar una sola directiva para asignar usuarios internos y externos al mismo paquete de acceso. Sin embargo, puede crear dos directivas en el mismo paquete de acceso: una para los usuarios internos y otra para los usuarios externos. Si hay varias directivas que se aplican a un usuario, se le pedirá en el momento de su solicitud que seleccione la directiva que le gustaría que se le asignara. En el diagrama siguiente se muestra un paquete de acceso con dos directivas.

![Varias directivas en un paquete de acceso](./media/entitlement-management-access-package-request-policy/access-package-policy.png)

### <a name="how-many-policies-will-i-need"></a>¿Cuántas directivas necesito?

| Escenario | Número de directivas |
| --- | --- |
| Quiero que todos los usuarios de mi directorio tengan la misma configuración de solicitudes y aprobación para un paquete de acceso | Uno |
| Quiero que todos los usuarios de ciertas organizaciones conectadas puedan solicitar un paquete de acceso | Uno |
| Deseo permitir que los usuarios de mi directorio y también los de fuera de mi directorio soliciten un paquete de acceso | Múltiple |
| Deseo especificar una configuración de aprobación diferente para algunos usuarios | Múltiple |
| Quiero que algunos usuarios accedan a las asignaciones de paquetes que van a expirar mientras que otros usuarios puedan extender su acceso | Múltiple |

Para información sobre la lógica de prioridad que se usa cuando se aplican varias directivas, consulte [Varias directivas](entitlement-management-troubleshoot.md#multiple-policies
).

## <a name="open-an-existing-access-package-and-add-a-new-policy-of-request-settings"></a>Apertura de un paquete de acceso existente e incorporación de una nueva directiva de configuración de solicitud

Si tiene un conjunto de usuarios que deben tener una configuración de solicitudes y aprobación diferente, probablemente tendrá que crear una directiva. Para empezar a agregar una nueva directiva a un paquete de acceso existente, siga estos pasos:

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Directivas ** y luego en **Agregar directiva**.

1. Comenzará en la pestaña **Aspectos básicos**. Escriba un nombre y una descripción para la directiva.

    ![Crear directiva con nombre y descripción](./media/entitlement-management-access-package-request-policy/policy-name-description.png)

1. Haga clic en **Siguiente** para abrir la pestaña **Solicitudes**.

1. Cambie la opción **Usuarios que pueden solicitar acceso**. Siga los pasos de las secciones siguientes para cambiar la configuración por una de las siguientes opciones: 
    - [para los usuarios del directorio](#for-users-in-your-directory) 
    - [Para los usuarios que no están en el directorio](#for-users-not-in-your-directory)
    - [ninguno (solo para las asignaciones directas del administrador)](#none-administrator-direct-assignments-only)

## <a name="for-users-in-your-directory"></a>para los usuarios del directorio

Siga estos pasos si quiere que los usuarios que están en el directorio puedan solicitar este paquete de acceso. Al definir la directiva de solicitud, puede especificar usuarios individuales o, más comúnmente, grupos de usuarios. Por ejemplo, es posible que su organización ya tenga un grupo, como **Todos los empleados**.  Si ese grupo se agrega a la directiva para los usuarios que pueden solicitar acceso, cualquier miembro de ese grupo podrá solicitar acceso.

1. En la sección **Users who can request access** (Usuarios que pueden solicitar acceso), haga clic en **For users in your directory** (Para usuarios del directorio).

    Cuando se selecciona esta opción, aparecen nuevas opciones para restringir aún más quién en el directorio puede solicitar este paquete de acceso.

    ![Paquete de acceso - Solicitudes - Para los usuarios que están en el directorio](./media/entitlement-management-access-package-request-policy/for-users-in-your-directory.png)

1. Seleccione una de las siguientes opciones:

    |  |  |
    | --- | --- |
    | **Usuarios y grupos específicos** | Elija esta opción si desea que solo los usuarios y grupos del directorio que especifique puedan solicitar este paquete de acceso. |
    | **Todos los miembros (excepto invitados)** | Elija esta opción si desea que todos los usuarios miembros del directorio puedan solicitar este paquete de acceso. Esta opción no incluye ningún usuario invitado al que pueda haber invitado en su directorio. |
    | **Todos los usuarios (incluidos invitados)** | Elija esta opción si desea que todos los usuarios miembros y los usuarios invitados del directorio puedan solicitar este paquete de acceso. |

    Los usuarios invitados hacen referencia a usuarios externos que han sido invitados al directorio con [Azure AD B2B](../external-identities/what-is-b2b.md). Para más información sobre las diferencias entre los usuarios miembros y los usuarios invitados, consulte [¿Cuáles son los permisos de usuario predeterminados en Azure Active Directory?](../fundamentals/users-default-permissions.md)

1. Si seleccionó **Usuarios y grupos específicos**, haga clic en **Agregar usuarios y grupos**.

1. En el panel Seleccionar usuarios y grupos, seleccione los usuarios y grupos que quiera agregar.

    ![Paquete de acceso - Solicitudes - Seleccionar usuarios y grupos](./media/entitlement-management-access-package-request-policy/select-users-groups.png)

1. Haga clic en **Seleccionar** para agregar los usuarios y grupos.

1. Si quiere solicitar aprobación, siga los pasos descritos en [Cambio de la configuración de aprobación de un paquete de acceso en la administración de derechos de Azure AD](entitlement-management-access-package-approval-policy.md) para configurar los valores de aprobación.

1. Vaya a la sección [Habilitación de solicitudes](#enable-requests).
 
## <a name="for-users-not-in-your-directory"></a>para los usuarios que no están en el directorio

 Los **Usuarios que no están en el directorio** se refieren a los usuarios que están en otro dominio o directorio de Azure AD. Es posible que estos usuarios no hayan sido invitados todavía al directorio. Los directorios de Azure AD deben configurarse para permitir invitaciones en **Restricciones de colaboración**. Para más información, consulte [Habilitación de la colaboración externa B2B y administración de quién puede invitar a otros usuarios](../external-identities/delegate-invitations.md).

> [!NOTE]
> Se creará una cuenta de usuario invitado para un usuario que aún no esté en el directorio cuya solicitud se haya aprobado o aprobado automáticamente. Se invitará al invitado, pero no recibirá un correo electrónico de invitación. En su lugar, recibirá un correo electrónico cuando se entregue su asignación de paquete de acceso. De forma predeterminada, más adelante cuando ese usuario invitado ya no tenga ninguna asignación de paquete de acceso porque su última asignación ha expirado o se ha cancelado, se bloqueará el inicio de sesión de esa cuenta de usuario invitado y se eliminará posteriormente. Si quiere que los usuarios invitados permanezcan en el directorio de manera indefinida, incluso si no tienen ninguna asignación de paquete de acceso, puede cambiar la configuración de la administración de derechos. Para más información acerca del objeto de usuario de invitado, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](../external-identities/user-properties.md).

Siga estos pasos si quiere que los usuarios que no están en el directorio soliciten este paquete de acceso:

1. En la sección **Usuarios que pueden solicitar acceso**, haga clic en **Para los usuarios que no están en el directorio**.

    Cuando se selecciona esta opción, aparecen nuevas opciones.

    ![Paquete de acceso - Solicitudes - Para los usuarios que no están en el directorio](./media/entitlement-management-access-package-request-policy/for-users-not-in-your-directory.png)

1. Seleccione una de las siguientes opciones:

    |  |  |
    | --- | --- |
    | **Organizaciones conectadas específicas** | Elija esta opción si desea seleccionar de una lista de organizaciones que el administrador haya agregado anteriormente. Todos los usuarios de las organizaciones seleccionadas pueden solicitar este paquete de acceso. |
    | **Todas las organizaciones conectadas** | Elija esta opción si todos los usuarios de las organizaciones conectadas pueden solicitar este paquete de acceso. |
    | **Todos los usuarios (todas las organizaciones conectadas y todos los nuevos usuarios externos)** | Elija esta opción si todos los usuarios de todas las organizaciones conectadas pueden solicitar este paquete de acceso y que la configuración de la lista de permitidos o denegados de B2B tenga prioridad para todos los usuarios externos nuevos. |

    Una organización conectada es un dominio o directorio externo de Azure AD con el que tiene relación.

1. Si seleccionó **Organizaciones conectadas específicas**, haga clic en **Agregar directorios** para seleccionar en una lista de las organizaciones conectadas que el administrador haya agregado anteriormente.

1. Escriba el nombre o el nombre de dominio para buscar una organización anteriormente conectada.

    ![Paquete de acceso - Solicitudes - Seleccionar directorios](./media/entitlement-management-access-package-request-policy/select-directories.png)

    Si la organización con la que desea colaborar no está en la lista, puede pedir al administrador que la agregue como una organización conectada. Para más información, consulte [Incorporación de una organización conectada](entitlement-management-organization.md).

1. Una vez que haya seleccionado todas las organizaciones conectadas, haga clic en **Seleccionar**.

    > [!NOTE]
    > Todos los usuarios de las organizaciones conectadas seleccionadas podrán solicitar este paquete de acceso. Esto incluye a los usuarios de Azure AD de todos los subdominios asociados a la organización, a menos que los dominios estén bloqueados por la lista de permitidos o denegados de Azure B2B. Para obtener más información, consulte [Allow or block invitations to B2B users from specific organizations](../external-identities/allow-deny-list.md) (Permitir o bloquear invitaciones a usuarios de B2B procedentes de determinadas organizaciones).

1. Si quiere solicitar aprobación, siga los pasos descritos en [Cambio de la configuración de aprobación de un paquete de acceso en la administración de derechos de Azure AD](entitlement-management-access-package-approval-policy.md) para configurar los valores de aprobación.
 
1. Vaya a la sección [Habilitación de solicitudes](#enable-requests).

## <a name="none-administrator-direct-assignments-only"></a>ninguno (solo para las asignaciones directas del administrador)

Siga estos pasos si quiere omitir las solicitudes de acceso y permitir a los administradores asignar directamente a usuarios específicos al paquete de acceso. Los usuarios no tendrán que solicitar el paquete de acceso. Aún podrá establecer la configuración del ciclo de vida, pero no hay ninguna configuración de solicitud.

1. En la sección **Usuarios que pueden solicitar acceso**, haga clic en **Ninguno (solo para las asignaciones directas del administrador)** .

    ![Paquete de acceso - Solicitudes - Ninguno (solo para las asignaciones directas del administrador)](./media/entitlement-management-access-package-request-policy/none-admin-direct-assignments-only.png)

    Después de crear el paquete de acceso, puede asignar directamente usuarios internos y externos específicos al paquete de acceso. Si especifica un usuario externo, se creará una cuenta de usuario invitado en su directorio. Para información sobre la asignación directa de un usuario, consulte [Visualización, incorporación y eliminación de asignaciones en un paquete de acceso](entitlement-management-access-package-assignments.md).

1. Vaya a la sección [Habilitación de solicitudes](#enable-requests).


## <a name="open-and-edit-an-existing-policy-of-request-settings"></a>Apertura y edición de una directiva existente de configuración de solicitud

Para cambiar la configuración de solicitudes y aprobación de un paquete de acceso, debe abrir la directiva correspondiente. Siga estos pasos para abrir y editar la configuración de solicitud de un paquete de acceso:

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Directivas** y luego en la directiva que quiere editar.

    Se abre el panel Detalles de directiva en la parte inferior de la página.

    ![Paquete de acceso - Panel Detalles de directiva](./media/entitlement-management-shared/policy-details.png)

1. Haga clic en **Editar** para editar la directiva.

    ![Paquete de acceso - Editar directiva](./media/entitlement-management-shared/policy-edit.png)

1. Haga clic en la pestaña **Solicitudes** para abrir la configuración de solicitud.

1. Siga los pasos descritos en las secciones anteriores para cambiar la configuración de solicitud según sea necesario.

1. Vaya a la sección [Habilitación de solicitudes](#enable-requests).

## <a name="enable-requests"></a>Habilitación de solicitudes

1. Si quiere que el paquete de acceso esté disponible de inmediato para los usuarios de la directiva de solicitud para que puedan solicitarlo, cambie el conmutador de alternancia Habilitar a **Sí**.

    Podrá habilitarla en todo momento cuando haya terminado de crear el paquete de acceso.

    Si seleccionó **Ninguno (solo para las asignaciones directas del administrador)** y establece la habilitación en **No**, los administradores no podrán asignar directamente este paquete de acceso.

    ![Paquete de acceso: directiva; configuración de habilitación de la directiva](./media/entitlement-management-access-package-approval-policy/enable-requests.png)

1. Haga clic en **Next**.

1. Si quiere exigir que los solicitantes proporcionen información adicional al solicitar acceso a un paquete de acceso, siga los pasos descritos en []() para configurar la información del solicitante (versión preliminar).

1. Configure los valores del ciclo de vida.

1. Si va a editar una directiva, haga clic en **Actualizar**. Si va a agregar una directiva nueva, haga clic en **Crear**.

## <a name="next-steps"></a>Pasos siguientes

- [Cambio de la configuración de aprobación de un paquete de acceso](entitlement-management-access-package-approval-policy.md)
- [Cambiar la configuración del ciclo de vida de un paquete de acceso](entitlement-management-access-package-lifecycle-policy.md)
- [Ver las solicitudes de un paquete de acceso](entitlement-management-access-package-requests.md)