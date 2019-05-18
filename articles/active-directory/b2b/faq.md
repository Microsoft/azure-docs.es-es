---
title: Colaboración B2B preguntas más frecuentes - Azure Active Directory | Microsoft Docs
description: Vea las respuestas a las preguntas más frecuentes acerca de la colaboración B2B de Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: reference
ms.date: 04/10/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3b40fc631e84336b684f981a2fef1515a1febec0
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65811903"
---
# <a name="azure-active-directory-b2b-collaboration-faqs"></a>Preguntas más frecuentes acerca de la colaboración B2B de Azure Active Directory

Las preguntas más frecuentes (P+F) acerca de la colaboración negocio a negocio (B2B) de Azure Active Directory (Azure AD) se actualizan periódicamente para incluir nuevos temas.

### <a name="can-we-customize-our-sign-in-page-so-its-more-intuitive-for-our-b2b-collaboration-guest-users"></a>¿Se puede personalizar la página de inicio de sesión de forma que resulte más intuitiva para los usuarios invitados a la colaboración B2B?
Por supuesto. Consulte nuestra [entrada del blog relativa a esta característica](https://blogs.technet.microsoft.com/enterprisemobility/2017/04/07/improving-the-branding-logic-of-azure-ad-login-pages/). Para más información acerca de cómo personalizar la página de inicio de sesión de una organización, consulte [Incorporación de la personalización de marca de empresa a sus páginas de inicio de sesión y panel de acceso](../fundamentals/customize-branding.md).

### <a name="can-b2b-collaboration-users-access-sharepoint-online-and-onedrive"></a>¿Pueden acceder los usuarios de colaboración B2B a SharePoint Online y OneDrive?
Sí. Sin embargo, la capacidad de buscar usuarios invitados existentes en SharePoint Online mediante el uso del selector de personas está **desactivada**. Para activar la opción de búsqueda de usuarios invitados existentes, establezca **ShowPeoplePickerSuggestionsForGuestUsers** en **Activado**. Esta configuración se puede activar en el nivel del inquilino o en el nivel de colección de sitios. Esta configuración se puede cambiar mediante los cmdlets Set-SPOTenant y SPOSite. Con estos cmdlets, los miembros pueden buscar todos los usuarios invitados existentes en el directorio. Los cambios en el ámbito del inquilino no afectan a los sitios de SharePoint Online que ya se han aprovisionado.

### <a name="is-the-csv-upload-feature-still-supported"></a>¿Aún se admite la característica de carga de CSV?
Sí. Para más información acerca de cómo utilizar la característica de carga de archivos .csv, vea [este ejemplo de PowerShell](code-samples.md).

### <a name="how-can-i-customize-my-invitation-emails"></a>¿Cómo puedo personalizar mis correos electrónicos de invitación?
Mediante las [API de invitación de B2B](customize-invitation-api.md) puede personalizar casi todos los elementos del proceso del invitador.

### <a name="can-guest-users-reset-their-multi-factor-authentication-method"></a>¿Pueden restablecer los usuarios invitados su método de Multi-Factor Authentication?
Sí. Los usuarios invitados pueden restablecer su método de Multi-Factor Authentication de la misma manera que los usuarios normales.

### <a name="which-organization-is-responsible-for-multi-factor-authentication-licenses"></a>¿Qué organización es la responsable de las licencias de Multi-Factor Authentication?
La organización que invita realiza la autenticación multifactor. La organización que invita debe asegurarse de que la organización tiene suficientes licencias para sus usuarios B2B que utilizan Multi-Factor Authentication.

### <a name="what-if-a-partner-organization-already-has-multi-factor-authentication-set-up-can-we-trust-their-multi-factor-authentication-and-not-use-our-own-multi-factor-authentication"></a>¿Qué ocurre si una organización asociada ya tiene Multi-Factor Authentication configurado? ¿Podemos confiar en su Multi-Factor Authentication y no utilizar nuestro propio Multi-Factor Authentication?
Esta característica está prevista para una versión futura, de modo que pueda seleccionar asociados específicos para excluir de su autenticación multifactor (la organización que invita).

### <a name="how-can-i-use-delayed-invitations"></a>¿Cómo se usan las invitaciones diferidas?
Algunas organizaciones pueden desear agregar usuarios de colaboración B2B, aprovisionarlos en las aplicaciones cuando sea necesario y, luego, enviar las invitaciones. Puede usar la API de invitación de colaboración B2B para personalizar el flujo de trabajo de incorporación.

### <a name="can-i-make-guest-users-visible-in-the-exchange-global-address-list"></a>¿Puedo hacer visibles a los usuarios invitados en la lista global de direcciones de Exchange?
Sí. Objetos de invitado no son visibles en la lista de su organización global de direcciones (GAL) de forma predeterminada, pero puede usar PowerShell de Azure Active Directory para que sean visibles. Consulte [¿puedo hacer que los objetos de invitado visible en la lista global de direcciones?](https://docs.microsoft.com/office365/admin/create-groups/manage-guest-access-in-groups#can-i-make-guest-objects-visible-in-the-global-address-list)

### <a name="can-i-make-a-guest-user-a-limited-administrator"></a>¿Puedo hacer que un usuario invitado sea un administrador limitado?
Totalmente. Para más información, consulte [Asignación de roles de administrador en la versión preliminar de Azure Active Directory](add-guest-to-role.md).

### <a name="does-azure-ad-b2b-collaboration-allow-b2b-users-to-access-the-azure-portal"></a>¿Permite la colaboración B2B de Azure AD que los usuarios B2B accedan a Azure Portal?
A menos que un usuario está asignado el rol de administrador limitado, los usuarios de colaboración B2B no necesitarán acceso al portal de Azure. Sin embargo, los usuarios de colaboración de B2B que están asignados al rol de administrador limitado pueden acceder al portal. Además, si un usuario invitado al que no se le han asignado estos roles de administrador tuviera acceso al portal, podría acceder a determinadas partes de la experiencia. El rol de usuario invitado tiene algunos permisos en el directorio.

### <a name="can-i-block-access-to-the-azure-portal-for-guest-users"></a>¿Puedo bloquear el acceso a Azure Portal a los usuarios invitados?
¡Sí! Cuando configure esta directiva tenga cuidado de evitar que se bloquee accidentalmente el acceso a los administradores y miembros.
Para bloquear el acceso de un usuario invitado a [Azure Portal](https://portal.azure.com), use una directiva de acceso condicional en la API del modelo de implementación clásica de Azure:
1. Modifique el grupo **Todos los usuarios** para que contenga solo miembros.
   ![Captura de pantalla muestra el grupo todos los usuarios que no sea invitado igual UserType](media/faq/modify-all-users-group.png)
2. Cree un grupo dinámico que contenga usuarios invitados.
   ![Captura de pantalla muestra un nuevo grupo de todos los usuarios invitados](media/faq/group-with-guest-users.png)
3. Configure una directiva de acceso condicional para bloquear el acceso de los usuarios invitados al portal, tal como se muestra en el siguiente vídeo:
  
   > [!VIDEO https://channel9.msdn.com/Blogs/Azure/b2b-block-guest-user/Player] 

### <a name="does-azure-ad-b2b-collaboration-support-multi-factor-authentication-and-consumer-email-accounts"></a>¿Admite la colaboración B2B de Azure AD admite la autenticación multifactor y las cuentas de correo electrónico de consumidor?
Sí. Tanto la autenticación multifactor como las cuentas de correo electrónico de consumidor admiten para la colaboración B2B de Azure AD.

### <a name="do-you-support-password-reset-for-azure-ad-b2b-collaboration-users"></a>¿Va a admitir el restablecimiento de contraseñas con los usuarios de la colaboración B2B de Azure AD?
Si el inquilino de Azure AD es el directorio principal de un usuario, puede [restablecer la contraseña del usuario](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-users-reset-password-azure-portal) en Azure Portal. Sin embargo, no puede restablecer directamente la contraseña de los usuarios invitados que inician sesión con una cuenta administrada por otro proveedor de identidades externo o por otro directorio de Azure AD. En ese caso, solo podrá restablecer la contraseña el usuario invitado o un administrador del directorio principal del usuario. A continuación, se incluyen algunos ejemplos de cómo funciona el restablecimiento de contraseñas con los usuarios invitados:
 
* Los usuarios invitados que inicien sesión con una cuenta Microsoft (por ejemplo guestuser@live.com) podrán restablecer sus propias contraseñas con el autoservicio de restablecimiento de contraseñas (SSPR) de la cuenta Microsoft. Consulte [Cómo restablecer la contraseña de tu cuenta Microsoft](https://support.microsoft.com/help/4026971/microsoft-account-how-to-reset-your-password).
* Los usuarios invitados que inicien sesión con una cuenta de Google u otro proveedor de identidades externo podrán restablecer sus propias contraseñas mediante el método de SSPR de su proveedor de identidades. Por ejemplo, un usuario invitado con la cuenta de Google guestuser@gmail.com puede restablecer su contraseña siguiendo las instrucciones acerca de [cómo cambiar o restablecer la contraseña](https://support.google.com/accounts/answer/41078).
* Si el inquilino de la identidad es un inquilino Just-In-Time (JIT) o un inquilino "viral" (es decir, un inquilino de Azure que es independiente y no está administrado), solamente el usuario invitado podrá restablecer su contraseña. A veces, una organización [asumirá la administración de los inquilinos virales](https://docs.microsoft.com/azure/active-directory/users-groups-roles/domains-admin-takeover) que se crean cuando los empleados usan sus direcciones de correo electrónico del trabajo para registrarse en los servicios. Después de que la organización se haga cargo de un inquilino viral, solo un administrador de dicha organización puede restablecer la contraseña del usuario o habilitar SSPR. Si es necesario, como la organización que invita, puede quitar la cuenta del usuario invitado del directorio y volver a enviar una invitación.
* Si el directorio principal del usuario invitado es su inquilino de Azure AD, usted podrá restablecer la contraseña del usuario. Por ejemplo, es posible que haya creado o sincronizado un usuario desde la instancia local de Active Directory y que haya establecido UserType en Guest (invitado). Como este usuario estará alojado en su directorio, podrá restablecer su contraseña desde Azure Portal.

### <a name="does-microsoft-dynamics-365-provide-online-support-for-azure-ad-b2b-collaboration"></a>¿Proporciona Microsoft Dynamics 365 compatibilidad en línea con la colaboración B2B de Azure AD?
Sí, Dynamics 365 (en línea) admite la colaboración B2B de Azure AD. Para más información, consulte el artículo de Dynamics 365 [Invitar a usuarios con colaboración B2B de Azure Active Directory](https://docs.microsoft.com/dynamics365/customer-engagement/admin/invite-users-azure-active-directory-b2b-collaboration).

### <a name="what-is-the-lifetime-of-an-initial-password-for-a-newly-created-b2b-collaboration-user"></a>¿Cuál es la duración de una contraseña inicial para un usuario de colaboración B2B recién creado?
Azure AD tiene un conjunto fijo de requisitos de bloqueo de cuentas, seguridad de la contraseña y caracteres que se aplican igualmente a todas las cuentas de usuario en la nube de Azure AD. Las cuentas de usuario en la nube son cuentas que no están federadas con otro proveedor de identidades, como: 
* Cuenta de Microsoft
* Facebook
* Servicios de federación de Active Directory
* Otro inquilino de nube (para la colaboración B2B)

En el caso de las cuentas federadas, la directiva de contraseñas depende de la directiva que se aplica en el inquilino local y la configuración de la cuenta Microsoft del usuario.

### <a name="an-organization-might-want-to-have-different-experiences-in-their-applications-for-tenant-users-and-guest-users-is-there-standard-guidance-for-this-is-the-presence-of-the-identity-provider-claim-the-correct-model-to-use"></a>Una organización puede tener distintas experiencias en sus aplicaciones para los usuarios inquilinos y los usuarios invitados. ¿Hay instrucciones estándar para esto? ¿La presencia de la notificación del proveedor de identidades es el modelo más adecuado para usarlo?
Un usuario invitado puede utilizar cualquier proveedor de identidades para realizar la autenticación. Para más información, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](user-properties.md). Use la propiedad **UserType** para determinar la experiencia del usuario. La notificación **UserType** no está incluida actualmente en el token. Las aplicaciones deben usar Graph API para consultar el usuario en el directorio y obtener UserType.

### <a name="where-can-i-find-a-b2b-collaboration-community-to-share-solutions-and-to-submit-ideas"></a>¿Dónde puedo encontrar una comunidad de colaboración B2B para compartir soluciones y enviar ideas?
Escuchamos constantemente sus comentarios para mejorar la colaboración B2B. Comparta escenarios de usuario, procedimientos recomendados y todo lo que le guste de la colaboración B2B de Azure AD. Únase al debate en [Microsoft Tech Community](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-B2B/bd-p/AzureAD_B2b).
 
También le invitamos a enviar sus ideas y a votar las características futuras en el sitio de [ideas para la colaboración B2B](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-B2B-Ideas/idb-p/AzureAD_B2B_Ideas).

### <a name="can-we-send-an-invitation-that-is-automatically-redeemed-so-that-the-user-is-just-ready-to-go-or-does-the-user-always-have-to-click-through-to-the-redemption-url"></a>¿Podemos enviarle una invitación que se canjee automáticamente para que el usuario pueda empezar en cualquier momento? ¿O bien, el usuario siempre tiene que hacer clic para desplazarse a la dirección URL de canje?
Puede invitar a otros usuarios de la organización asociada utilizando la interfaz de usuario, los scripts de PowerShell o las API. Después, puede enviar al usuario invitado un vínculo directo a una aplicación compartida. En la mayoría de los casos, ya no es necesario abrir la invitación de correo electrónico y hacer clic en una dirección URL de canje. Consulte [Experiencia de invitación de colaboración B2B de Azure Active Directory](redemption-experience.md).

### <a name="how-does-b2b-collaboration-work-when-the-invited-partner-is-using-federation-to-add-their-own-on-premises-authentication"></a>¿Cómo funciona la colaboración B2B cuando el asociado invitado utiliza la federación para agregar su propia autenticación local?
Si el asociado tiene un inquilino de Azure AD que está federado en la infraestructura de autenticación local, se consigue automáticamente el inicio de sesión único (SSO) local. Si el asociado no tiene ningún inquilino de Azure AD, se crea una cuenta de Azure AD para los usuarios nuevos. 

### <a name="i-thought-azure-ad-b2b-didnt-accept-gmailcom-and-outlookcom-email-addresses-and-that-b2c-was-used-for-those-kinds-of-accounts"></a>Creía que B2B de Azure AD no aceptaba direcciones de correo electrónico gmail.com y outlook.com, y que se usaba B2C para esos tipos de cuentas
Estamos eliminando las diferencias entre B2B y colaboración de negocio al consumidor (B2C) en cuanto a las identidades que se admiten. La identidad utilizada no es una buena razón para decidir si se va a utilizar B2B o B2C. Para obtener información acerca de cómo elegir la opción de colaboración, consulte [Comparación de la colaboración B2B y B2C de Azure Active Directory](compare-with-b2c.md).

### <a name="what-applications-and-services-support-azure-b2b-guest-users"></a>¿Qué aplicaciones y los servicios admiten los usuarios invitados de Azure B2B?
Todas las aplicaciones integradas en AD de Azure pueden admitir usuarios invitados de B2B de Azure, pero debe usar un punto de conexión configurado como un inquilino para autenticar a los usuarios invitados. También es posible que necesite [personalizar las notificaciones](claims-mapping.md) del token SAML que se emite cuando un usuario invitado se autentica en la aplicación. 

### <a name="can-we-force-multi-factor-authentication-for-b2b-guest-users-if-our-partners-dont-have-multi-factor-authentication"></a>¿Podemos forzar la autenticación multifactor para usuarios invitados de B2B si nuestros asociados no la tienen?
Sí. Puede encontrar información en [Acceso condicional para usuarios de colaboración B2B](conditional-access.md).

### <a name="in-sharepoint-you-can-define-an-allow-or-deny-list-for-external-users-can-we-do-this-in-azure"></a>En SharePoint puede definir una lista de "permitidos" o "denegados" para usuarios externos. ¿Se puede hacer esto en Azure?
Sí. La colaboración B2B de Azure AD admite listas de permitidos y de denegados. 

### <a name="what-licenses-do-we-need-to-use-azure-ad-b2b"></a>¿Qué licencias se necesitan para usar B2B de Azure AD?
Para obtener información acerca de las licencias que necesita tener una organización para usar B2B de Azure AD, consulte [Guía de concesión de licencias de colaboración B2B de Azure Active Directory](licensing-guidance.md).

### <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)

