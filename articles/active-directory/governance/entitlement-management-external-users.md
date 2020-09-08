---
title: 'Gobernanza del acceso de los usuarios externos en la administración de derechos de Azure AD: Azure Active Directory'
description: Obtenga información acerca de la configuración que puede especificar para controlar el acceso de los usuarios externos en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: ef88a15286389c98bb77f982afbc54358897eef4
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783865"
---
# <a name="govern-access-for-external-users-in-azure-ad-entitlement-management"></a>Gobernanza del acceso de los usuarios externos en la administración de derechos de Azure AD

La administración de derechos de Azure AD emplea [Azure AD de negocio a negocio (B2B)](../external-identities/what-is-b2b.md) para colaborar con personas ajenas a su organización en otro directorio. Con Azure AD B2B, los usuarios externos se autentican en su directorio particular, pero están representados en el directorio. La representación en el directorio permite asignar al usuario acceso a los recursos.

En este artículo se describen los valores que puede especificar para controlar el acceso de los usuarios externos.

## <a name="how-entitlement-management-can-help"></a>Cómo puede ayudar la administración de derechos

Cuando utilice la experiencia de invitación de [Azure AD B2B](../external-identities/what-is-b2b.md), ya debe conocer las direcciones de correo electrónico de los usuarios invitados externos que desea incorporar al directorio de recursos y con los que desea trabajar. Esto funciona muy bien cuando está trabajando en un proyecto más pequeño o a corto plazo y ya conoce a todos los participantes, pero es más difícil de administrar si tiene muchos usuarios con los que quiere trabajar o si los participantes cambian con el tiempo.  Por ejemplo, puede estar trabajando con otra organización y tener un punto de contacto con ella, pero con el tiempo otros usuarios de esa organización también necesitarán acceso.

Con la administración de derechos, puede definir una directiva que permita a los usuarios de las organizaciones que especifique poder solicitar un paquete de acceso. Puede especificar si se requiere aprobación y una fecha de expiración para el acceso. Si se requiere aprobación, también puede invitar a uno o varios usuarios de la organización externa a su directorio y designarlos como aprobadores, ya que es probable que sepan qué usuarios externos de su organización necesitan tener acceso. Una vez configurado el paquete de acceso, puede enviar el vínculo a su persona de contacto (patrocinador) en la organización externa. Ese contacto puede compartirlo con otros usuarios de la organización externa y pueden usar este vínculo para solicitar el paquete de acceso. Los usuarios de esa organización que ya han recibido la invitación a su directorio también pueden usar ese vínculo.

Cuando se aprueba una solicitud, la administración de derechos proporciona al usuario el acceso necesario, que puede incluir invitar al usuario si aún no está en el directorio. Azure AD creará automáticamente una cuenta de invitado B2B para ellos. Tenga en cuenta que un administrador puede haber limitado previamente qué organizaciones están permitidas para la colaboración, mediante el establecimiento de una [lista de permitidos y denegados de B2B](../external-identities/allow-deny-list.md) para permitir o bloquear las invitaciones a otras organizaciones.  Si el usuario no está autorizado por la lista de permitidos o bloqueados, no se le invitará.

Puesto que no desea que el acceso del usuario externo dure para siempre, especifique una fecha de expiración en la directiva, por ejemplo 180 días. Después de 180 días, si no se amplía el acceso, la administración de derechos eliminará todo el acceso asociado con ese paquete de acceso. De forma predeterminada, si el usuario invitado a través de la administración de derechos no tiene ninguna otra asignación de paquetes de acceso, cuando pierda la última asignación, la cuenta de invitado se bloqueará durante 30 días y posteriormente se eliminará. Esto evita la proliferación de cuentas innecesarias. Tal y como se describe en las secciones siguientes, estos valores se pueden configurar.

## <a name="how-access-works-for-external-users"></a>Cómo funciona el acceso para los usuarios externos

En el diagrama y los pasos siguientes se proporciona información general sobre cómo conceder acceso a los usuarios externos a un paquete de acceso.

![Diagrama que muestra el ciclo de vida de los usuarios externos](./media/entitlement-management-external-users/external-users-lifecycle.png)

1. [Agrega una organización conectada](entitlement-management-organization.md) para el directorio de Azure AD o el dominio con el que desea colaborar.

1. Se crea un paquete de acceso en el directorio que incluye una directiva [Para usuarios que no están en el directorio](entitlement-management-access-package-create.md#for-users-not-in-your-directory).

1. Envía un [vínculo al portal Mi acceso](entitlement-management-access-package-settings.md) a su contacto en la organización externa que pueden compartir con sus usuarios para solicitar el paquete de acceso.

1. Un usuario externo (**Solicitante A** en este ejemplo) usa el vínculo al portal Mi acceso para [solicitar acceso](entitlement-management-request-access.md) al paquete de acceso. La forma en que el usuario inicia sesión depende del tipo de autenticación del directorio o dominio definido en la organización conectada.

1. Un aprobador [aprueba la solicitud](entitlement-management-request-approve.md) (o se aprueba automáticamente).

1. La solicitud entra en el estado [en entrega](entitlement-management-process.md).

1. El proceso de invitación B2B crea una cuenta de usuario invitado en el directorio, **Solicitante A (invitado)** en este ejemplo. Si se define una [lista de permitidos o denegados](../external-identities/allow-deny-list.md), se aplicará la configuración de la lista.

1. Al usuario invitado se le asigna acceso a todos los recursos del paquete de acceso. Los cambios pueden tardar algún tiempo en realizarse en Azure AD y en otros servicios en línea de Microsoft o en aplicaciones de SaaS conectadas. Para más información, consulte [Cuándo se aplican los cambios](entitlement-management-access-package-resources.md#when-changes-are-applied).

1. El usuario externo recibe un correo electrónico que indica que el acceso se ha [entregado](entitlement-management-process.md).

1. Para tener acceso a los recursos, el usuario externo puede hacer clic en el vínculo del correo electrónico o intentar acceder a cualquiera de los recursos del directorio directamente para completar el proceso de invitación.

1. En función de la configuración de la directiva, la asignación de paquetes de acceso para el usuario externo expira con el tiempo y se quita el acceso del usuario externo.

1. En función del ciclo de vida de la configuración de usuarios externos, cuando el usuario externo ya no tenga ninguna asignación de paquete de acceso, no podrá iniciar sesión y la cuenta de usuario invitado se quitará de su directorio.

## <a name="settings-for-external-users"></a>Configuración de usuarios externos

Para asegurarse de que los usuarios ajenos a la organización puedan solicitar paquetes de acceso y obtener acceso a los recursos de estos, hay algunas opciones de configuración que debe comprobar que se han configurado correctamente.

### <a name="enable-catalog-for-external-users"></a>Habilitar un catálogo para usuarios externos

- De forma predeterminada, al crear un [nuevo catálogo](entitlement-management-catalog-create.md), este se habilita para permitir que los usuarios externos soliciten los paquetes de acceso que contiene. Asegúrese de que la opción **Habilitado para los usuarios externos** se haya establecido en **Sí**.

    ![Edición de la configuración del catálogo](./media/entitlement-management-shared/catalog-edit.png)

### <a name="configure-your-azure-ad-b2b-external-collaboration-settings"></a>Configurar las opciones de colaboración externa de Azure AD B2B

- Permitir a los invitados invitar a otros invitados a su directorio significa que las invitaciones se pueden producir fuera de la administración de derechos. Se recomienda establecer la opción **Los invitados pueden invitar** en **No** para permitir solo invitaciones controladas correctamente.
- Si usa la lista de elementos permitidos de B2B, debe asegurarse de que todos los dominios con los que desee asociarse mediante la administración de derechos se agreguen a la lista. Como alternativa, si usa la lista de denegación de elementos de B2B, debe asegurarse de que el dominio con el que desee asociar no se agregue a la lista.
- Si crea una directiva de administración de derechos para **todos los usuarios** (todas las organizaciones conectadas + cualquier usuario externo nuevo), tendrá prioridad la configuración de la lista de elementos permitidos o denegados de B2B. Por lo tanto, asegúrese de incluir en la lista de elementos permitidos los dominios que desea incluir en esta directiva si usa uno y excluirlos de la lista de elementos denegados si usa este tipo de lista.
- Si desea crear una directiva de administración de derechos que incluya **todos los usuarios** (todas las organizaciones conectadas y cualquier usuario externo nuevo), primero debe habilitar la autenticación de código de acceso de un solo uso de correo electrónico para su directorio. Para obtener más información, consulte [Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)](../external-identities/one-time-passcode.md#opting-in-to-the-preview).
- Para obtener más información sobre la configuración de colaboración externa de Azure AD B2B, consulte [Habilitación de la colaboración externa B2B y administración de quién puede invitar a otros usuarios](../external-identities/delegate-invitations.md).

    ![Comprobación de la configuración de colaboración externa de Azure AD](./media/entitlement-management-external-users/collaboration-settings.png)

### <a name="review-your-conditional-access-policies"></a>Revisar las directivas de acceso condicional

- Asegúrese de excluir los invitados de las directivas de acceso condicional que los nuevos usuarios invitados no podrán cumplir, ya que esto les impedirá que puedan iniciar sesión en su directorio. Por ejemplo, es probable que los invitados no tengan un dispositivo registrado, que no estén en una ubicación conocida y no deseen volver a registrarse para la autenticación multifactor (MFA), por lo que agregar estos requisitos en una directiva de acceso condicional impedirá que los invitados usen la administración de derechos de administración. Para más información, consulte [¿Qué son las condiciones en el acceso condicional de Azure Active Directory?](../conditional-access/concept-conditional-access-conditions.md).

    ![Configuración de la exclusión de directiva de acceso condicional de Azure AD](./media/entitlement-management-external-users/conditional-access-exclude.png)

### <a name="review-your-sharepoint-online-external-sharing-settings"></a>Revisar la configuración de uso compartido externo de SharePoint Online

- Si desea incluir sitios de SharePoint Online en los paquetes de acceso para usuarios externos, asegúrese de que la configuración de uso compartido externo en el nivel de la organización se haya establecido en **Cualquiera** (los usuarios no necesitan iniciar sesión) o **Invitados nuevos y existentes** (los invitados deben iniciar sesión o proporcionar un código de verificación). Para más información, consulta [Activar o desactivar el uso compartido externo](/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting).

- Si desea restringir el uso compartido externo fuera de la administración de derechos, puede establecer la configuración de uso compartido externo en **Invitados existentes**. Luego, solo los nuevos usuarios que se invitan a través de la administración de derechos podrán obtener acceso a estos sitios. Para más información, consulta [Activar o desactivar el uso compartido externo](/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting).

- Asegúrese de que la configuración de nivel de sitio habilite el acceso de invitado (las mismas selecciones que se muestran anteriormente). Para más información, consulte [Activar o desactivar el uso compartido externo de un sitio](/sharepoint/change-external-sharing-site).

### <a name="review-your-microsoft-365-group-sharing-settings"></a>Revisión de la configuración de uso compartido de grupos de Microsoft 365

- Si quiere incluir grupos de Microsoft 365 en los paquetes de acceso para usuarios externos, asegúrese de que la opción **Permitir que los usuarios agreguen nuevos invitados a la organización** se haya establecido en **Activado** para permitir el acceso de invitado. Para más información, consulte [Administrar el acceso de invitado a grupos de Microsoft 365](/office365/admin/create-groups/manage-guest-access-in-groups?view=o365-worldwide#manage-groups-guest-access).

- Si quiere que los usuarios externos puedan acceder al sitio de SharePoint Online y a los recursos asociados a un grupo de Microsoft 365, asegúrese de activar el uso compartido externo de SharePoint Online. Para más información, consulta [Activar o desactivar el uso compartido externo](/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting).

- Para obtener información sobre cómo establecer la directiva de invitado para los grupos de Microsoft 365 en el nivel de directorio de PowerShell, consulte [Ejemplo: Configuración de la directiva de invitado para grupos en el nivel de directorio](../users-groups-roles/groups-settings-cmdlets.md#example-configure-guest-policy-for-groups-at-the-directory-level).

### <a name="review-your-teams-sharing-settings"></a>Revisar la configuración de uso compartido de Teams

- Si quiere incluir Teams en los paquetes de acceso para usuarios externos, asegúrese de que la opción **Permitir el acceso de invitado en Teams** se haya establecido en **Activado** para permitir el acceso de invitado. Para más información, consulte [Configurar el acceso de invitado en el centro de administración de Microsoft Teams](/microsoftteams/set-up-guests#configure-guest-access-in-the-teams-admin-center).

## <a name="manage-the-lifecycle-of-external-users"></a>Administración del ciclo de vida de los usuarios externos

Puede seleccionar lo que ocurre cuando un usuario externo, invitado a su directorio mediante una solicitud de paquete de acceso aprobada, ya no tiene ningún paquete de acceso asignado. Esto puede ocurrir si el usuario renuncia a todas sus asignaciones de paquetes de acceso o la asignación del último paquete de acceso expira. De forma predeterminada, cuando un usuario externo ya no tiene un paquete de acceso asignado, se le impide iniciar sesión en el directorio. Después de 30 días, la cuenta de usuario invitado se quitará del directorio.

**Rol necesario:** administrador global o administrador de usuarios.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú izquierdo, en la sección **Administración de derechos**, haga clic en **Configuración**.

1. Haga clic en **Editar**.

    ![Configuración para administrar el ciclo de vida de los usuarios externos](./media/entitlement-management-external-users/settings-external-users.png)

1. En la sección **Administración del ciclo de vida de los usuarios externos**, seleccione las diferentes opciones para los usuarios externos.

1. Cuando un usuario externo pierde la última asignación de un paquete de acceso, si desea bloquearlos para que no inicien sesión en este directorio, establezca **Impedir que los usuarios externos inicien sesión en este directorio** en **Sí**.

    > [!NOTE]
    > Si un usuario tiene bloqueada la capacidad de iniciar sesión en este directorio, el usuario no podrá volver a solicitar el paquete de acceso ni solicitar acceso adicional en este directorio. No configure el bloqueo de inicio de sesión si posteriormente necesitará solicitar acceso a otros paquetes de acceso.

1. Cuando un usuario externo pierde la última asignación de un paquete de acceso, si quiere quitar su cuenta de usuario invitado de este directorio, establezca **Quitar usuario externo** en **Sí**.

    > [!NOTE]
    > La administración de derechos solo quita las cuentas a las que se ha invitado mediante la administración de derechos. Tenga en cuenta también que se impedirá que el usuario inicie sesión y se eliminará del directorio aunque dicho usuario se haya agregado a recursos del directorio que no eran asignaciones de paquetes de acceso. Si el invitado estaba presente en este directorio antes de recibir las asignaciones de paquetes de acceso, se mantendrá. Sin embargo, si se le invitó mediante una asignación de paquete de acceso y, después de invitarlo, se ha asignado a un sitio de OneDrive para la Empresa o SharePoint Online, se quitará.

1. Si quiere quitar la cuenta de usuario invitado de este directorio, puede establecer el número de días antes de que se quite. Si desea quitar la cuenta de usuario invitado en cuanto pierda la última asignación a cualquier paquete de acceso, establezca **Número de días antes de que se quite el usuario externo de este directorio** en **0**.

1. Haga clic en **Save**(Guardar).

## <a name="next-steps"></a>Pasos siguientes

- [Adición de una organización conectada](entitlement-management-organization.md)
- [Para los usuarios que no están en el directorio](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
- [Solución de problemas](entitlement-management-troubleshoot.md)