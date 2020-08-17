---
title: 'Propiedades de usuario invitado B2B: Azure Active Directory | Microsoft Docs'
description: Estados y propiedades de usuario invitado B2B de Azure Active Directory antes y después del canje de invitación
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 06/19/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan, seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: e7271c4de6d5c186c9e561aa37a140eaa04cbc0a
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907613"
---
# <a name="properties-of-an-azure-active-directory-b2b-collaboration-user"></a>Propiedades de un usuario de colaboración B2B de Azure Active Directory

En este artículo se describen las propiedades y los estados del objeto de usuario invitado B2B en Azure Active Directory (Azure AD) antes y después del canje de invitación. Un usuario de colaboración de negocio a negocio (B2B) de Azure AD es un usuario con UserType = Guest. Dicho usuario suele ser de una organización asociada y tiene, de forma predeterminada, privilegios limitados en el directorio de la invitación.

En función de las necesidades de la organización invitadora, un usuario de colaboración de B2B de Azure AD puede tener cualquiera de los siguientes estados de cuenta:

- Estado 1: alojado en una instancia externa de Azure AD y representado como un usuario invitado en la organización que invita. En este caso, el usuario de B2B inicia sesión con una cuenta de Azure AD que pertenece al inquilino invitado. Aunque la organización asociada no use Azure AD, se crea el usuario invitado en Azure AD. Los requisitos son que el usuario canjea su invitación y Azure AD comprueba su dirección de correo electrónico. Esta solución también se denomina inquilino Just-In-Time (JIT) o inquilino "viral".

   > [!IMPORTANT]
   > **A partir del 31 de marzo de 2021**, Microsoft dejará de admitir el canje de invitaciones mediante la creación de cuentas de Azure AD no administradas e inquilinos para escenarios de colaboración B2B. Como preparación, se recomienda a los clientes que opten por la [autenticación de código de acceso de un solo uso por correo electrónico](one-time-passcode.md). Agradecemos sus comentarios sobre esta característica en vista previa pública. Nos alegra poder crear más formas de colaborar.

- Estado 2: alojado en una cuenta Microsoft u otra cuenta y representado como usuario invitado en la organización host. En este caso, el usuario invitado inicia sesión con una cuenta de Microsoft o una cuenta social (google.com o similar). La identidad del usuario invitado se crea como una cuenta de Microsoft en el directorio de la organización que invita durante el canje de la oferta.

- Estado 3: alojado en la instancia de Active Directory local de la organización host y sincronizado con la instancia de Azure AD de la organización host. Puede usar Azure AD Connect para sincronizar las cuentas de asociado con la nube como usuarios B2B de Azure AD con UserType = Invitado. Vea [Conceder a las cuentas de asociado administradas localmente acceso a los recursos en la nube](hybrid-on-premises-to-cloud.md).

- Estado 4: alojado en la instancia de Azure AD de la organización host con UserType = Invitado y credenciales que administra dicha organización.

  ![Diagrama que ilustra los cuatro estados del usuario](media/user-properties/redemption-diagram.png)


Ahora, veamos cómo es un usuario de colaboración de B2B de Azure AD en Azure AD.

### <a name="before-invitation-redemption"></a>Antes del canje de la invitación

Las cuentas de estado 1 y estado 2 resultan de los usuarios invitados que invitan a colaborar con el uso de las propias credenciales de los usuarios invitados. Cuando se envía inicialmente la invitación al usuario invitado, se crea una cuenta en el directorio. Esta cuenta no tiene ninguna credencial asociada, ya que la autenticación la realiza el proveedor de identidades del usuario invitado. La propiedad **Origen** de la cuenta de usuario invitado del directorio se establece en **Usuario invitado**. 

![Captura de pantalla que muestra las propiedades del usuario antes del canje de la oferta](media/user-properties/before-redemption.png)

### <a name="after-invitation-redemption"></a>Después del canje de la invitación

Una vez que el usuario invitado acepta la invitación, la propiedad **Origen** se actualiza según determine el proveedor de identidades del usuario invitado.

Para los usuarios invitados en estado 1, el **origen** es **Azure Active Directory externo**.

![Usuario invitado en estado 1 después de canjear la oferta](media/user-properties/after-redemption-state1.png)

Para los usuarios invitados en estado 2, el **origen** es **Cuenta Microsoft**.

![Usuario invitado en estado 2 después de canjear la oferta](media/user-properties/after-redemption-state2.png)

Para los usuarios invitados en estado 3 y estado 4, la propiedad **Origen** se establece en **Azure Active Directory** o **Windows Server Active Directory**, como se describe en la siguiente sección.

## <a name="key-properties-of-the-azure-ad-b2b-collaboration-user"></a>Propiedades clave del usuario de colaboración de B2B de Azure AD
### <a name="usertype"></a>UserType
Esta propiedad indica la relación del usuario con el espacio host. Esta propiedad puede tener dos valores:
- Miembro: este valor indica un empleado de la organización host y un usuario en la plantilla de dicha organización. Por ejemplo, este usuario espera tener acceso solo a sitios internos. Este usuario no se considera como un colaborador externo.

- Invitado: Este valor indica un usuario que no se considera interno de la empresa, como un colaborador externo, un asociado o un cliente. No se espera que dicho usuario reciba una comunicación interna del CEO o que reciba beneficios de la empresa, por ejemplo.

  > [!NOTE]
  > UserType no tiene relación alguna con la forma en que el usuario inicia sesión, el rol de directorio del usuario, etc. Esta propiedad simplemente indica la relación del usuario con la organización host y permite a la organización exigir directivas que dependan de esta propiedad.

### <a name="source"></a>Source
Esta propiedad indica la forma en que el usuario inicia sesión.

- Usuario invitado: este usuario ha recibido la invitación, pero aún no la ha canjeado.

- Azure Active Directory externo: este usuario está alojado en una organización externa y se autentica mediante una cuenta de Azure AD que pertenece a la otra organización. Este tipo de inicio de sesión corresponde al estado 1.

- Cuenta de Microsoft: el usuario está alojado en una cuenta de Microsoft y se autentica mediante una cuenta de Microsoft. Este tipo de inicio de sesión corresponde al estado 2.

- Windows Server Active Directory: este usuario inicia sesión desde una instancia de Active Directory local que pertenece a esta organización. Este tipo de inicio de sesión corresponde al estado 3.

- Azure Active Directory: este usuario se autentica mediante una cuenta de Azure AD que pertenece a esta organización. Este tipo de inicio de sesión corresponde al estado 4.
  > [!NOTE]
  > Source y UserType son propiedades independientes. Un valor de Source no implica un valor concreto de UserType.

## <a name="can-azure-ad-b2b-users-be-added-as-members-instead-of-guests"></a>¿Se pueden agregar usuarios de B2B de Azure AD como miembros, en lugar de como invitados?
Normalmente, un usuario invitado y uno de B2B de Azure AD son sinónimos. Por tanto, de manera predeterminada los usuarios de colaboración de B2B de Azure AD se agregan como usuario con UserType = Guest. Sin embargo, en algunos casos, la organización asociada forma parte de una organización mayor a la que también pertenece la organización host. En ese caso, la organización host puede tratar a los usuarios de la organización asociada como miembros, en lugar de como invitados. Use las API del Administrador de invitaciones de B2B de Azure AD para agregar un usuario de la organización asociada a la organización host como miembro, o para invitarlo.

## <a name="filter-for-guest-users-in-the-directory"></a>Filtro de usuarios invitados en el directorio

![Captura de pantalla muestra el filtro para los usuarios invitados](media/user-properties/filter-guest-users.png)

## <a name="convert-usertype"></a>Conversión de UserType
Es posible convertir UserType de miembro a invitado, y viceversa, mediante PowerShell. Sin embargo, la propiedad UserType representa la relación del usuario con la organización. Por tanto, debe cambiar esta propiedad solo si cambia la relación del usuario con la organización. Si cambia la relación del usuario, ¿se debe cambiar el nombre principal de usuario (UPN)? ¿Debe el usuario seguir teniendo acceso a los mismos recursos? ¿Debe asignarse un buzón de correo? No se recomienda cambiar el valor de UserType mediante el uso de PowerShell como una actividad atómica. Además, en caso de que esta propiedad se vuelva inmutable mediante PowerShell, no se recomienda depender de este valor.

## <a name="remove-guest-user-limitations"></a>Eliminación de limitaciones de usuarios invitados
Puede haber casos en los que desee ofrecer a los usuarios invitados privilegios más altos. Puede agregar un usuario invitado a cualquier rol e, incluso, eliminar las restricciones de usuario invitado predeterminadas en el directorio para concederle los mismos privilegios que a los miembros.

Se pueden desactivar las limitaciones predeterminadas para que un usuario invitado del directorio de la empresa tenga los mismos permisos que un usuario que sea miembro.

![Captura de pantalla que muestra la opción de usuarios externos en la configuración del usuario](media/user-properties/remove-guest-limitations.png)

## <a name="can-i-make-guest-users-visible-in-the-exchange-global-address-list"></a>¿Puedo hacer visibles a los usuarios invitados en la lista global de direcciones de Exchange?
Sí. De forma predeterminada, los objetos de invitado no aparecen en la lista global de direcciones de la organización, pero puede usar Azure Active Directory PowerShell para que figuren. Para obtener más información, consulte **Can I make guest objects visible in the global address list?** (¿Puedo hacer visibles los objetos de invitado de la lista global de direcciones?) en [Manage guest access in Office 365 Groups](https://docs.microsoft.com/office365/admin/create-groups/manage-guest-access-in-groups) (Administrar el acceso de invitados en Grupos de Office 365).

## <a name="can-i-update-a-guest-users-email-address"></a>¿Puedo actualizar la dirección de correo electrónico de un usuario invitado?

Si un usuario invitado acepta su invitación y cambia posteriormente su dirección de correo electrónico, el nuevo correo electrónico no se sincroniza automáticamente con el objeto de usuario invitado en el directorio. La propiedad mail se crea a través de [Microsoft Graph API](https://docs.microsoft.com/graph/api/resources/user?view=graph-rest-1.0). Puede actualizar la propiedad mail en el centro de administración de Exchange o [Exchange Online PowerShell](https://docs.microsoft.com/powershell/module/exchange/users-and-groups/set-mailuser?view=exchange-ps), y el cambio se reflejará en el objeto de usuario invitado de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
* [Tokens de usuario de colaboración B2B](user-token.md)
* [Asignación de notificaciones de usuario de colaboración B2B](claims-mapping.md)
