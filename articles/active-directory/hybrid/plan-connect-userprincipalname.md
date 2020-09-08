---
title: Rellenado de userPrincipalName de Azure AD
description: En el siguiente documento se describe cómo rellenar el atributo UserPrincipalName.
author: billmath
ms.subservice: hybrid
ms.author: billmath
ms.date: 06/26/2018
ms.topic: conceptual
ms.workload: identity
ms.service: active-directory
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: e105d4909705622a931c51bcb7cf0a9db4179525
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279608"
---
# <a name="azure-ad-userprincipalname-population"></a>Rellenado de userPrincipalName de Azure AD

En este artículo se describe cómo rellenar el atributo UserPrincipalName en Azure Active Directory (Azure AD).
El valor del atributo UserPrincipalName es el nombre de usuario de Azure AD que se usa para las cuentas de usuario.

## <a name="upn-terminology"></a>Terminología de UPN
En este artículo se usa la siguiente terminología:

|Término|Descripción|
|-----|-----|
|Dominio inicial|El dominio predeterminado (onmicrosoft.com) del inquilino de Azure AD. Por ejemplo, contoso.onmicrosoft.com.|
|Dirección de enrutamiento de correo electrónico en línea de Microsoft (MOERA)|Azure AD calcula la dirección MOERA del atributo MailNickName de Azure AD y el dominio inicial de Azure AD como &lt;MailNickName&gt;&#64;&lt;dominio inicial&gt;.|
|Atributo mailNickName local|Atributo de Active Directory, cuyo valor representa el alias de un usuario de una organización de Exchange.|
|Atributo mail local|Atributo de Active Directory, cuyo valor representa la dirección de correo electrónico de un usuario.|
|Dirección SMTP principal|La dirección de correo electrónico principal de un objeto de destinatario de Exchange. Por ejemplo, SMTP:user\@contoso.com.|
|Identificador de inicio de sesión alternativo|Atributo local que no sea UserPrincipalName, como el atributo mail, utilizado para iniciar sesión.|

## <a name="what-is-userprincipalname"></a>¿Qué es UserPrincipalName?
UserPrincipalName es un atributo de inicio de sesión del estilo de Internet para un usuario basado en el estándar de Internet [RFC 822](https://www.ietf.org/rfc/rfc0822.txt). 

### <a name="upn-format"></a>Formato de UPN
Un UPN consta de un prefijo de UPN (el nombre de la cuenta de usuario) y un sufijo de UPN (un nombre de dominio DNS). El prefijo se une con el sufijo mediante el símbolo "\@". Por ejemplo, "alguien\@ejemplo.com". Un UPN debe ser único entre todos los objetos de entidad de seguridad dentro de un bosque de directorio. 

## <a name="upn-in-azure-ad"></a>UPN en Azure AD 
Azure AD usa el UPN para permitir que los usuarios inicien sesión.  El UPN que un usuario puede usar depende de si el dominio se ha verificado o no.  Si se ha verificado, un usuario con dicho sufijo podrá iniciar sesión en Azure AD.  

El atributo lo sincroniza Azure AD Connect.  Durante la instalación, puede ver los dominios verificados y los no verificados.

   ![Dominios sin verificar](./media/plan-connect-userprincipalname/unverifieddomain.png) 

## <a name="alternate-login-id"></a>Identificador de inicio de sesión alternativo
En algunos entornos, es posible que los usuarios finales solo conozcan su dirección de correo electrónico y no su UPN.  El uso de la dirección de correo electrónico puede deberse a una directiva corporativa o a una dependencia de una aplicación de línea de negocio local.

El identificador de inicio de sesión alternativo le permite configurar una experiencia de inicio de sesión donde los usuarios puedan iniciar sesión con un atributo que no sea su UPN, por ejemplo, el correo electrónico.

Para habilitar el identificador de inicio de sesión alternativo en Azure AD, no se necesitan pasos de configuración adicionales para usar Azure AD Connect. El identificador alternativo se puede configurar directamente con el asistente. Vea la sección de sincronización para consultar la configuración de inicio de sesión de los usuarios en Azure AD. En el menú desplegable **Nombre principal de usuario**, seleccione el atributo para el identificador de inicio de sesión alternativo.

![Dominios sin verificar](./media/plan-connect-userprincipalname/altloginid.png)  

Para más información, consulte [Configurar un identificador de inicio de sesión alternativo](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id) y [Configuración de inicio de sesión de Azure AD](how-to-connect-install-custom.md#azure-ad-sign-in-configuration).

## <a name="non-verified-upn-suffix"></a>Sufijo de UPN no verificado
Si el sufijo del identificador de inicio de sesión alternativo o del atributo UserPrincipalName local no se verifica en el inquilino de Azure AD, el valor del atributo UserPrincipalName de Azure AD se establece en MOERA. Azure AD calcula la dirección MOERA del atributo MailNickName de Azure AD y el dominio inicial de Azure AD como &lt;MailNickName&gt;&#64;&lt;dominio inicial&gt;.

## <a name="verified-upn-suffix"></a>Sufijo de UPN verificado
Si el sufijo del identificador de inicio de sesión alternativo o del atributo UserPrincipalName local se verifica en el inquilino de Azure AD, el valor del atributo UserPrincipalName de Azure AD será el mismo que el del identificador de inicio de sesión alternativo o del atributo UserPrincipalName local.

## <a name="azure-ad-mailnickname-attribute-value-calculation"></a>Cálculo del valor del atributo MailNickName de Azure AD
Dado que el valor del atributo UserPrincipalName de Azure AD se puede establecer en MOERA, es importante saber cómo se calcula el valor del atributo MailNickName de Azure AD, que es el prefijo MOERA.

Cuando un objeto de usuario se sincroniza con un inquilino de Azure AD por primera vez, Azure AD comprueba los elementos siguientes en el orden indicado y establece el valor del atributo MailNickName en el primer elemento que exista:

- Atributo mailNickName local
- Prefijo de la dirección SMTP principal
- Prefijo del atributo mail local
- Prefijo del identificador de inicio de sesión alternativo o del atributo userPrincipalName local
- Prefijo de la dirección SMTP secundaria

Cuando las actualizaciones de un objeto de usuario se sincronizan con el inquilino de Azure AD, este último actualiza el valor del atributo MailNickName solo en caso de que haya una actualización del valor del atributo mailNickName local.

>[!IMPORTANT]
>Azure AD recalcula el valor del atributo UserPrincipalName solo en caso de que una actualización del valor del identificador de inicio de sesión alternativo o del atributo UserPrincipalName local se sincronice con el inquilino de Azure AD. 
>
>Cada vez que Azure AD recalcula el atributo UserPrincipalName, también recalcula la dirección MOERA. 

## <a name="upn-scenarios"></a>Escenarios de UPN
A continuación se indican escenarios de ejemplo de cómo se calcula el UPN en función del escenario de que se trate.

### <a name="scenario-1-non-verified-upn-suffix--initial-synchronization"></a>Escenario 1: Sufijo de UPN no comprobado (sincronización inicial)

![Escenario 1](./media/plan-connect-userprincipalname/example1.png)

Objeto de usuario local:
- mailNickName      : &lt;sin establecer&gt;
- proxyAddresses        : {SMTP:us1@contoso.com}
- mail          : us2@contoso.com
- userPrincipalName : us3@contoso.com

Se sincronizó el objeto de usuario en el inquilino de Azure AD por primera vez
- Establezca el atributo MailNickName de Azure AD en el prefijo de la dirección SMTP principal.
- Establezca MOERA como &lt;MailNickName&gt;&#64;&lt;dominio inicial&gt;.
- Establezca el atributo UserPrincipalName de Azure AD como MOERA.

Objeto de usuario del inquilino de AD Azure:
- MailNickName      : us1           
- UserPrincipalName : us1@contoso.onmicrosoft.com


### <a name="scenario-2-non-verified-upn-suffix--set-on-premises-mailnickname-attribute"></a>Escenario 2: Sufijo de UPN no comprobado (definición del atributo mailNickName local)

![Escenario 2](./media/plan-connect-userprincipalname/example2.png)

Objeto de usuario local:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us1@contoso.com}
- mail          : us2@contoso.com
- userPrincipalName : us3@contoso.com

Sincronizar la actualización del atributo mailNickName local en el inquilino de Azure AD
- Actualice el atributo MailNickName de Azure AD con el atributo mailNickName local.
- Dado que no hay ninguna actualización para el atributo userPrincipalName local, no se realiza ningún cambio del atributo UserPrincipalName de Azure AD.

Objeto de usuario del inquilino de AD Azure:
- MailNickName      : us4
- UserPrincipalName : us1@contoso.onmicrosoft.com

### <a name="scenario-3-non-verified-upn-suffix--update-on-premises-userprincipalname-attribute"></a>Escenario 3: Sufijo de UPN no comprobado (actualización del atributo userPrincipalName local)

![Escenario 3](./media/plan-connect-userprincipalname/example3.png)

Objeto de usuario local:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us1@contoso.com}
- mail          : us2@contoso.com
- userPrincipalName : us5@contoso.com

Sincronizar la actualización del atributo userPrincipalName local en el inquilino de Azure AD
- La actualización del atributo userPrincipalName local desencadena el recálculo de MOERA y del atributo UserPrincipalName de Azure AD.
- Establezca MOERA como &lt;MailNickName&gt;&#64;&lt;dominio inicial&gt;.
- Establezca el atributo UserPrincipalName de Azure AD como MOERA.

Objeto de usuario del inquilino de AD Azure:
- MailNickName      : us4
- UserPrincipalName : us4@contoso.onmicrosoft.com

### <a name="scenario-4-non-verified-upn-suffix--update-primary-smtp-address-and-on-premises-mail-attribute"></a>Escenario 4: Sufijo de UPN no comprobado (actualización del atributo "mail" local y de la dirección SMTP principal)

![Escenario 4](./media/plan-connect-userprincipalname/example4.png)

Objeto de usuario local:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us6@contoso.com}
- mail          : us7@contoso.com
- userPrincipalName : us5@contoso.com

Sincronizar la actualización del atributo mail local y la dirección SMTP principal en el inquilino de Azure AD
- Después de la sincronización inicial del objeto de usuario, las actualizaciones del atributo mail local y la dirección SMTP principal no afectarán a los atributos MailNickName o UserPrincipalName de Azure AD.

Objeto de usuario del inquilino de AD Azure:
- MailNickName      : us4
- UserPrincipalName : us4@contoso.onmicrosoft.com

### <a name="scenario-5-verified-upn-suffix--update-on-premises-userprincipalname-attribute-suffix"></a>Escenario 5: Sufijo de UPN comprobado (actualización del sufijo del atributo userPrincipalName)

![Escenario 5](./media/plan-connect-userprincipalname/example5.png)

Objeto de usuario local:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us6@contoso.com}
- mail          : us7@contoso.com
- userPrincipalName : us5@verified.contoso.com

Sincronizar la actualización del atributo userPrincipalName local en el inquilino de Azure AD
- La actualización del atributo userPrincipalName local desencadena el recálculo del atributo UserPrincipalName de Azure AD.
- Establezca el atributo UserPrincipalName de Azure como el atributo userPrincipalName local, ya que el sufijo de UPN se verifica con el inquilino de Azure AD.

Objeto de usuario del inquilino de AD Azure:
- MailNickName      : us4     
- UserPrincipalName : us5@verified.contoso.com

## <a name="next-steps"></a>Pasos siguientes
- [Integración de los directorios locales con Azure Active Directory](whatis-hybrid-identity.md)
- [Instalación personalizada de Azure AD Connect](how-to-connect-install-custom.md)