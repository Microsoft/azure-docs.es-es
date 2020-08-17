---
title: 'Preguntas más frecuentes sobre la colaboración B2B: Azure Active Directory | Microsoft Docs'
description: Vea las respuestas a las preguntas más frecuentes acerca de la colaboración B2B de Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: reference
ms.date: 03/19/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 92c3e0d77a26db406e24d6d2fa07e96349613634
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907715"
---
# <a name="azure-active-directory-b2b-collaboration-faqs"></a>Preguntas más frecuentes acerca de la colaboración B2B de Azure Active Directory

Las preguntas más frecuentes (P+F) acerca de la colaboración negocio a negocio (B2B) de Azure Active Directory (Azure AD) se actualizan periódicamente para incluir nuevos temas.

   > [!IMPORTANT]
   > **A partir del 31 de marzo de 2021**, Microsoft dejará de admitir el canje de invitaciones mediante la creación de cuentas de Azure AD no administradas e inquilinos para escenarios de colaboración B2B. Como preparación, se recomienda a los clientes que opten por la [autenticación de código de acceso de un solo uso por correo electrónico](one-time-passcode.md). Agradecemos sus comentarios sobre esta característica en vista previa pública. Nos alegra poder crear más formas de colaborar.

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
Actualmente, esta característica no se admite. Si el acceso a los recursos de la organización requiere la autenticación multifactor, la organización del partner tendrá que registrarse para la autenticación multifactor en la organización que invita.

### <a name="how-can-i-use-delayed-invitations"></a>¿Cómo se usan las invitaciones diferidas?
Algunas organizaciones pueden desear agregar usuarios de colaboración B2B, aprovisionarlos en las aplicaciones cuando sea necesario y, luego, enviar las invitaciones. Puede usar la API de invitación de colaboración B2B para personalizar el flujo de trabajo de incorporación.

### <a name="can-i-make-guest-users-visible-in-the-exchange-global-address-list"></a>¿Puedo hacer visibles a los usuarios invitados en la lista global de direcciones de Exchange?
Sí. De manera predeterminada, los objetos de invitado no aparecen en la lista global de direcciones de la organización (GAL), pero puede usar Azure Active Directory PowerShell para que figuren. Consulte [¿Puedo hacer que los objetos de invitado sean visibles en la lista global de direcciones?](https://docs.microsoft.com/office365/admin/create-groups/manage-guest-access-in-groups#add-guests-to-the-global-address-list).

### <a name="can-i-make-a-guest-user-a-limited-administrator"></a>¿Puedo hacer que un usuario invitado sea un administrador limitado?
Totalmente. Para más información, consulte [Asignación de roles de administrador en la versión preliminar de Azure Active Directory](add-guest-to-role.md).

### <a name="does-azure-ad-b2b-collaboration-allow-b2b-users-to-access-the-azure-portal"></a>¿Permite la colaboración B2B de Azure AD que los usuarios B2B accedan a Azure Portal?
Salvo que a un usuario se le asigne el rol de administrador limitado, los usuarios de colaboración B2B no necesitarán acceso a Azure Portal. Sin embargo, los usuarios de colaboración B2B a los que se asigna el rol de administrador limitado pueden acceder al portal. Además, si un usuario invitado al que no se le han asignado estos roles de administrador tuviera acceso al portal, podría acceder a determinadas partes de la experiencia. El rol de usuario invitado tiene algunos permisos en el directorio.

### <a name="can-i-block-access-to-the-azure-portal-for-guest-users"></a>¿Puedo bloquear el acceso a Azure Portal a los usuarios invitados?

Sí. Puede crear una directiva de acceso condicional que impida a todos los usuarios invitados y externos tener acceso a Azure Portal. Cuando configure esta directiva tenga cuidado de evitar que se bloquee accidentalmente el acceso a los administradores y miembros.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de seguridad o administrador de acceso condicional.
2. En Azure Portal, seleccione **Azure Active Directory**. 
3. En **Administrar**, seleccione **Seguridad**.
4. En **Proteger**, seleccione **Acceso condicional**. Seleccione **Nueva directiva**.
5. En la página **Nuevo**, en el cuadro de texto **Nombre**, escriba un nombre para la directiva (por ejemplo, "Impedir que los invitados accedan al portal").
6. En **Asignaciones**, seleccione **Usuarios y grupos**.
7. En la pestaña **Incluir**, elija **Seleccionar usuarios y grupos** y, a continuación, seleccione **Todos los usuarios externos e invitados (versión preliminar)** .
9. Seleccione **Listo**.
10. En la página **Nuevo**, en la sección **Asignaciones**, seleccione **Aplicaciones en la nube o acciones**.
11. En la página **Aplicaciones en la nube o acciones**, haga clic en **Seleccionar aplicaciones** y después haga clic en **Seleccionar**.
12. En la página **Seleccionar**, elija **Administración de Microsoft Azure** y después haga clic en **Seleccionar**.
13. En la página **Aplicaciones en la nube o acciones**, seleccione **Listo**.

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
* Cuenta Microsoft
* Facebook
* Servicios de federación de Active Directory
* Otro inquilino de nube (para la colaboración B2B)

En el caso de las cuentas federadas, la directiva de contraseñas depende de la directiva que se aplica en el inquilino local y la configuración de la cuenta Microsoft del usuario.

### <a name="an-organization-might-want-to-have-different-experiences-in-their-applications-for-tenant-users-and-guest-users-is-there-standard-guidance-for-this-is-the-presence-of-the-identity-provider-claim-the-correct-model-to-use"></a>Una organización puede tener distintas experiencias en sus aplicaciones para los usuarios inquilinos y los usuarios invitados. ¿Hay instrucciones estándar para esto? ¿La presencia de la notificación del proveedor de identidades es el modelo más adecuado para usarlo?
Un usuario invitado puede utilizar cualquier proveedor de identidades para realizar la autenticación. Para más información, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](user-properties.md). Use la propiedad **UserType** para determinar la experiencia del usuario. La notificación **UserType** no está incluida actualmente en el token. Las aplicaciones deben usar Microsoft Graph API para consultar el usuario en el directorio y obtener UserType.

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
Todas las aplicaciones integradas en Azure AD pueden admitir usuarios invitados de B2B de Azure, pero deben usar un punto de conexión configurado como inquilino para autenticar a los usuarios invitados. También es posible que necesite [personalizar las notificaciones](claims-mapping.md) del token SAML que se emite cuando un usuario invitado se autentica en la aplicación. 

### <a name="can-we-force-multi-factor-authentication-for-b2b-guest-users-if-our-partners-dont-have-multi-factor-authentication"></a>¿Podemos forzar la autenticación multifactor para usuarios invitados de B2B si nuestros asociados no la tienen?
Sí. Para más información, consulte [Acceso condicional para usuarios de colaboración B2B](conditional-access.md).

### <a name="in-sharepoint-you-can-define-an-allow-or-deny-list-for-external-users-can-we-do-this-in-azure"></a>En SharePoint puede definir una lista de "permitidos" o "denegados" para usuarios externos. ¿Se puede hacer esto en Azure?
Sí. La colaboración B2B de Azure AD admite listas de permitidos y de denegados. 

### <a name="what-licenses-do-we-need-to-use-azure-ad-b2b"></a>¿Qué licencias se necesitan para usar B2B de Azure AD?
Para obtener información acerca de las licencias que necesita tener una organización para usar B2B de Azure AD, consulte [Guía de concesión de licencias de colaboración B2B de Azure Active Directory](licensing-guidance.md).

### <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)

