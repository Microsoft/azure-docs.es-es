---
title: 'Incorporación de una suscripción de Azure existente al inquilino: Azure AD'
description: Instrucciones sobre cómo agregar una suscripción de Azure existente al inquilino de Azure Active Directory.
services: active-directory
author: msaburnley
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 06/01/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18, contperfq4
ms.collection: M365-identity-device-management
ms.openlocfilehash: 347232d452b5d309da7cdb7c8fe34f0e41abde00
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87025345"
---
# <a name="associate-or-add-an-azure-subscription-to-your-azure-active-directory-tenant"></a>Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory

Una suscripción de Azure tiene una relación de confianza con Azure Active Directory (Azure AD). Una suscripción confía en Azure AD para autenticar usuarios, servicios y dispositivos.

Varias suscripciones pueden confiar en el mismo directorio de Azure AD. Cada suscripción solo puede confiar en un único directorio.

Si su suscripción expira, se pierde el acceso a los otros recursos asociados a la suscripción. Sin embargo, el directorio de Azure AD permanece en Azure. Puede asociar y administrar el directorio con una suscripción de Azure diferente.

Todos los usuarios tienen un único directorio *particular* para la autenticación. Los usuarios también pueden ser invitados en otros directorios. Puede ver los directorios principales e invitados para cada usuario en Azure AD.

> [!Important]
> Al asociar una suscripción a un directorio diferente, los usuarios que tengan roles asignados mediante el [control de acceso basado en rol (RBAC)](../../role-based-access-control/role-assignments-portal.md) pierden el acceso. Los administradores de suscripciones clásicas, incluidos el administrador y los coadministradores del servicio, también pierden el acceso.
>
> También se quitan las asignaciones de directivas de una suscripción cuando dicha suscripción está asociada a un directorio diferente.
>
> El traslado del clúster de Azure Kubernetes Service (AKS) a otra suscripción o el traslado de la suscripción propietaria del clúster a un nuevo inquilino, provoca que el clúster pierda funcionalidad debido a la pérdida de asignaciones de roles y derechos de las entidades de servicio. Para obtener más información sobre AKS, consulte [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/).


## <a name="before-you-begin"></a>Antes de empezar

Antes de poder asociar o agregar la suscripción, realice las siguientes tareas:

- Revise la siguiente lista de cambios que se producirán después de asociar o agregar su suscripción e infórmese sobre cómo podría verse afectado:

  - Los usuarios que tienen roles asignados mediante RBAC perderán el acceso.
  - El administrador y los coadministradores del servicio perderán el acceso.
  - Si tiene almacenes de claves, no se podrá acceder a ellos y tendrá que corregirlos después de la asociación.
  - Si tiene identidades administradas para recursos, como Virtual Machines o Logic Apps, debe volver a habilitarlas o crearlas después de la asociación.
  - Si tiene una instancia de Azure Stack registrada, tendrá que volver a registrarla después de la asociación.

- Iniciar sesión con una cuenta que:

  - Tiene una asignación de rol [Propietario](../../role-based-access-control/built-in-roles.md#owner) para la suscripción. Para obtener más información sobre cómo asignar el rol Propietario, consulte [Administración del acceso a los recursos de Azure mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).
  - Exista en el directorio actual y en el nuevo directorio. El directorio actual está asociado a la suscripción. Va a asociar el nuevo directorio a la suscripción. Para más información sobre cómo obtener acceso a otro directorio, consulte [Adición de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](../b2b/add-users-administrator.md).

- Asegúrese de que no usa una suscripción de proveedores de servicios en la nube de Azure (CSP) (MS-AZR-0145P, MS-AZR-0146P, MS-AZR-159P), una suscripción interna de Microsoft (MS-AZR-0015P) o una suscripción a Microsoft Imagine (MS-AZR-0144P).

## <a name="associate-a-subscription-to-a-directory"></a>Asociación de una suscripción a un directorio<a name="to-associate-an-existing-subscription-to-your-azure-ad-directory"></a>

Para asociar una suscripción existente al directorio de Azure AD, siga estos pasos:

1. Inicie sesión y seleccione la suscripción que quiere usar en la [Página de suscripciones de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

1. Seleccione **Cambiar directorio**.

    ![Página de suscripciones, con la opción Cambiar directorio resaltada](media/active-directory-how-subscriptions-associated-directory/change-directory-in-azure-subscriptions.png)

1. Revise las advertencias que aparecen y, a continuación, seleccione **Cambiar**.

    ![Página de cambio de directorio que muestra el directorio al que se va a cambiar](media/active-directory-how-subscriptions-associated-directory/edit-directory-ui.png)

    Una vez que se cambie el directorio para la suscripción, se mostrará un mensaje de confirmación.

  1. Seleccione **Cambiar directorios** en la página suscripción para ir al nuevo directorio. 

      ![Página del conmutador de directorios con información de ejemplo](media/active-directory-how-subscriptions-associated-directory/directory-switcher.png)

      Puede tardar varias horas hasta que todo se muestre correctamente. Si parece que tarda demasiado tiempo, compruebe el **filtro de suscripción global**. Asegúrese de que la suscripción que se ha trasladado no esté oculta. Es posible que tenga que cerrar la sesión de Azure Portal y volver a iniciarla para ver el directorio nuevo.

Cambiar el directorio de suscripción es una operación de nivel de servicio, por lo que no afecta a la propiedad de facturación de suscripción. El administrador de cuenta todavía puede cambiar al administrador de servicio desde el [centro de cuentas](https://account.azure.com/subscriptions). Para eliminar el directorio original, debe transferir la propiedad de facturación de suscripción a un nuevo administrador de cuenta. Para más información acerca de cómo transferir la propiedad de facturación, vea [Transferencia de la propiedad de una suscripción de Azure a otra cuenta](../../cost-management-billing/manage/billing-subscription-transfer.md).

## <a name="post-association-steps"></a>Pasos posteriores a la asociación

Después de asociar una suscripción a un directorio diferente, puede que tenga que realizar las siguientes tareas para reanudar las operaciones:

- Si tiene almacenes de claves, debe cambiar el Id. de inquilino del almacén de claves. Para más información, consulte [Cambio del identificador de inquilino de Key Vault después de mover una suscripción](../../key-vault/general/move-subscription.md).

- Si usaba identidades administradas asignadas por el sistema para los recursos, debe volver a habilitar estas identidades. Si usaba identidades administradas asignadas por el usuario, debe volver a crear estas identidades. Después de volver a habilitar o crear las identidades administradas, debe volver a restablecer los permisos asignados a esas identidades. Para más información, consulte [¿Qué es Managed Identities for Azure Resources?](../managed-identities-azure-resources/overview.md)

- Si ha registrado una instancia de Azure Stack que usa esta suscripción, debe volver a registrarla. Para obtener más información, consulte [Registro de Azure Stack con Azure](/azure-stack/operator/azure-stack-registration).

## <a name="next-steps"></a>Pasos siguientes

- Para crear un nuevo inquilino de Azure AD, consulte [Inicio rápido: Creación de un inquilino en Azure Active Directory](active-directory-access-create-new-tenant.md).

- Para más información sobre cómo controla Microsoft Azure el acceso a los recursos, consulte [Roles de administrador de suscripciones clásicas, roles de RBAC de Azure y roles de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

- Para más información sobre cómo asignar roles en Azure AD, consulte [Asignación de roles de administrador y no administrador a los usuarios con Azure Active Directory](active-directory-users-assign-role-azure-portal.md).
