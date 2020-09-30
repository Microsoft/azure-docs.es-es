---
title: 'Preguntas más frecuentes sobre la implementación híbrida de llaves de seguridad FIDO2: Azure Active Directory'
description: Más información sobre algunas de las preguntas más frecuentes sobre la implementación híbrida del inicio de sesión sin contraseña con llaves de seguridad FIDO2 a través de Azure Active Directory (versión preliminar)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 08/19/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 16e232cedb13dc246bf7a568adfad401c1fe3eb8
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89236554"
---
# <a name="deployment-frequently-asked-questions-faqs-for-hybrid-fido2-security-keys-in-azure-ad-preview"></a>Preguntas más frecuentes sobre la implementación híbrida de llaves de seguridad FIDO2 en Azure AD (versión preliminar)

En este artículo se tratan las preguntas más frecuentes sobre la implementación de dispositivos unidos a Azure AD híbrido y el inicio de sesión sin contraseña en recursos locales. Con esta característica sin contraseña, puede habilitar la autenticación de Azure AD en dispositivos con Windows 10 unidos a Azure AD híbrido mediante llaves de seguridad FIDO2. Los usuarios pueden iniciar sesión en Windows en sus dispositivos usando credenciales modernas como las llaves FIDO2, y acceder a recursos tradicionales basados en Active Directory Domain Services (AD DS) con una experiencia de inicio de sesión único (SSO) que permite acceder directamente a sus recursos locales.

Se admiten los siguientes escenarios para los usuarios de un entorno híbrido:

* Inicio de sesión en dispositivos unidos a Azure AD híbrido mediante llaves de seguridad FIDO2 para acceder con SSO a recursos locales
* Inicio de sesión en dispositivos unidos a Azure AD con llaves de seguridad FIDO2 para acceder con SSO a recursos locales

Para ver una introducción de las llaves de seguridad FIDO2 y del acceso híbrido a recursos locales, consulte los siguientes artículos:

* [Claves de seguridad FIDO2 sin contraseñas](howto-authentication-passwordless-security-key.md)
* [Windows 10 sin contraseña](howto-authentication-passwordless-security-key-windows.md)
* [Entorno local sin contraseña](howto-authentication-passwordless-security-key-on-premises.md)

> [!NOTE]
> Las claves de seguridad FIDO2 son una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="security-keys"></a>Llaves de seguridad

* [Mi organización requiere la autenticación en dos fases para acceder a los recursos. ¿Qué puedo hacer para admitir este requisito?](#my-organization-requires-multi-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement)
* [¿Dónde puedo encontrar llaves de seguridad de FIDO2 compatibles?](#where-can-i-find-compliant-fido2-security-keys)
* [¿Qué hago si pierdo mi clave de seguridad?](#what-if-i-lose-my-security-key)
* [¿Cómo se protegen los datos de las llaves de seguridad FIDO2?](#how-is-the-data-protected-on-the-fido2-security-key)
* [¿Cómo funciona el registro de las llaves de seguridad FIDO2?](#how-does-the-registering-of-fido2-security-keys-work)
* [¿Hay alguna manera de que los administradores aprovisionen directamente las claves para los usuarios?](#is-there-a-way-for-admins-to-provision-the-keys-for-the-users-directly)

### <a name="my-organization-requires-multi-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement"></a>Mi organización requiere la autenticación multifactor para acceder a los recursos. ¿Qué puedo hacer para admitir este requisito?

Las llaves de seguridad FIDO2 tienen varios factores de forma. Póngase en contacto con el fabricante del dispositivo concreto para analizar cómo se pueden habilitar sus dispositivos con un PIN o reconocimiento biométrico como segundo factor. Para ver una lista de proveedores disponibles, vea esta sección sobre [proveedores de llaves de seguridad FIDO2](concept-authentication-passwordless.md#fido2-security-key-providers).

### <a name="where-can-i-find-compliant-fido2-security-keys"></a>¿Dónde puedo encontrar llaves de seguridad de FIDO2 compatibles?

Para ver una lista de proveedores disponibles, vea esta sección sobre [proveedores de llaves de seguridad FIDO2](concept-authentication-passwordless.md#fido2-security-key-providers).

### <a name="what-if-i-lose-my-security-key"></a>¿Qué ocurre si pierdo mi clave de seguridad?

Para quitar claves de Azure Portal, vaya a la página **Información de seguridad** y quite la clave de seguridad FIDO2.

### <a name="how-is-the-data-protected-on-the-fido2-security-key"></a>¿Cómo se protegen los datos de las llaves de seguridad FIDO2?

Las llaves de seguridad FIDO2 tienen enclaves seguros que protegen las claves privadas almacenadas en ellas. Una llave de seguridad FIDO2 también tiene protección contra ataques de repetición (hammering) integrada, como en Windows Hello, donde no se puede extraer la clave privada.

### <a name="how-does-the-registering-of-fido2-security-keys-work"></a>¿Cómo funciona el registro de las llaves de seguridad FIDO2?

Para obtener más información sobre cómo registrar y usar las llaves de seguridad FIDO2, consulte [Habilitación del inicio de sesión sin contraseña con llaves de seguridad](howto-authentication-passwordless-security-key.md).

### <a name="is-there-a-way-for-admins-to-provision-the-keys-for-the-users-directly"></a>¿Hay alguna manera de que los administradores aprovisionen directamente las claves para los usuarios?

No, en este momento todavía no.

## <a name="prerequisites"></a>Requisitos previos

* [¿Funciona esta característica si no hay conectividad a Internet?](#does-this-feature-work-if-theres-no-internet-connectivity)
* [¿Cuáles son los puntos de conexión específicos que se deben abrir para Azure AD?](#what-are-the-specific-end-points-that-are-required-to-be-open-to-azure-ad)
* [¿Cómo puedo identificar el tipo de unión a un dominio (unido a Azure AD o unido a Azure AD híbrido) en mi dispositivo con Windows 10?](#how-do-i-identify-the-domain-join-type-azure-ad-joined-or-hybrid-azure-ad-joined-for-my-windows-10-device)
* [¿Cuál es la recomendación sobre el número de controladores de dominio que se deben revisar?](#whats-the-recommendation-on-the-number-of-dcs-that-should-be-patched)
* [¿Puedo implementar el proveedor de credenciales FIDO2 en un dispositivo exclusivamente local?](#can-i-deploy-the-fido2-credential-provider-on-an-on-premises-only-device)
* [El inicio de sesión con llaves de seguridad FIDO2 no funciona en mi administrador de dominio ni en otras cuentas con privilegios elevados. ¿Por qué?](#fido2-security-key-sign-in-isnt-working-for-my-domain-admin-or-other-high-privilege-accounts-why)

### <a name="does-this-feature-work-if-theres-no-internet-connectivity"></a>¿Funciona esta característica si no hay conectividad a Internet?

La conectividad a Internet es un requisito previo para poder habilitar esta característica. La primera vez que un usuario inicia sesión con llaves de seguridad FIDO2 debe tener conectividad a Internet. En el caso de los eventos de inicio de sesión posteriores, el inicio de sesión almacenado en caché debería funcionar y permitir que el usuario se autentique sin conectividad a Internet.

Para disfrutar de una experiencia coherente, asegúrese de que los dispositivos tengan acceso a Internet y una línea de visión a los controladores de dominio.

### <a name="what-are-the-specific-end-points-that-are-required-to-be-open-to-azure-ad"></a>¿Cuáles son los puntos de conexión específicos que se deben abrir para Azure AD?

Se necesitan los siguientes puntos de conexión para el registro y la autenticación:

* * *.microsoftonline.com*
* * *.microsoftonline-p.com*
* * *.msauth.net*
* * *.msauthimages.net*
* * *.msecnd.net*
* * *.msftauth.net*
* * *.msftauthimages.net*
* * *.phonefactor.net*
* *enterpriseregistration.windows.net*
* *management.azure.com*
* *policykeyservice.dc.ad.msft.net*
* *secure.aadcdn.microsoftonline-p.com*

Para ver una lista completa de los puntos de conexión necesarios para usar productos en línea de Microsoft, consulte [Direcciones URL e intervalos de direcciones IP de Office 365](/microsoft-365/enterprise/urls-and-ip-address-ranges).

### <a name="how-do-i-identify-the-domain-join-type-azure-ad-joined-or-hybrid-azure-ad-joined-for-my-windows-10-device"></a>¿Cómo puedo identificar el tipo de unión a un dominio (unido a Azure AD o unido a Azure AD híbrido) en mi dispositivo con Windows 10?

Para comprobar si el dispositivo cliente con Windows 10 tiene el tipo de unión a un dominio correcto, use el siguiente comando:

```console
Dsregcmd/status
```

La siguiente salida de ejemplo muestra que el dispositivo está unido a Azure AD, ya que el valor de *AzureADJoined* está establecido en *YES*:

```output
+---------------------+
| Device State        |
+---------------------+

AzureADJoined: YES
EnterpriseJoined: NO
DomainedJoined: NO
```

La siguiente salida de ejemplo muestra que el dispositivo está unido a Azure AD híbrido, ya que el valor de *DomainedJoined* está establecido en *YES*. También se muestra *DomainName*:

```output
+---------------------+
| Device State        |
+---------------------+

AzureADJoined: YES
EnterpriseJoined: NO
DomainedJoined: YES
DomainName: CONTOSO
```

En un controlador de dominio de Windows Server 2016 o 2019, compruebe que se han aplicado las siguientes revisiones. Si es necesario, ejecute Windows Update para instalarlas:

* Windows Server 2016: [KB4534307](https://support.microsoft.com/help/4534307/windows-10-update-kb4534307)
* Windows Server 2019: [KB4534321](https://support.microsoft.com/help/4534321/windows-10-update-kb4534321)

Desde un dispositivo cliente, ejecute el siguiente comando para comprobar la conectividad a un controlador de dominio adecuado con las revisiones instaladas:

```console
nltest /dsgetdc:<domain> /keylist /kdc
```

### <a name="whats-the-recommendation-on-the-number-of-dcs-that-should-be-patched"></a>¿Cuál es la recomendación sobre el número de controladores de dominio que se deben revisar?

Se recomienda aplicar una revisión a la mayoría de los controladores de dominio de Windows Server 2016 o 2019 con la revisión para asegurarse de que puedan controlar la carga de solicitudes de autenticación de la organización.

En un controlador de dominio de Windows Server 2016 o 2019, compruebe que se han aplicado las siguientes revisiones. Si es necesario, ejecute Windows Update para instalarlas:

* Windows Server 2016: [KB4534307](https://support.microsoft.com/help/4534307/windows-10-update-kb4534307)
* Windows Server 2019: [KB4534321](https://support.microsoft.com/help/4534321/windows-10-update-kb4534321)

### <a name="can-i-deploy-the-fido2-credential-provider-on-an-on-premises-only-device"></a>¿Puedo implementar el proveedor de credenciales FIDO2 en un dispositivo exclusivamente local?

No, esta característica no es compatible con el dispositivo exclusivamente local. El proveedor de credenciales FIDO2 no se mostraría.

### <a name="fido2-security-key-sign-in-isnt-working-for-my-domain-admin-or-other-high-privilege-accounts-why"></a>El inicio de sesión con llaves de seguridad FIDO2 no funciona en mi administrador de dominio ni en otras cuentas con privilegios elevados. ¿Por qué?

La directiva de seguridad predeterminada no concede permiso de Azure AD para firmar cuentas con privilegios elevados en recursos locales.

Para desbloquear las cuentas, use **Usuarios y equipos de Active Directory** para modificar la propiedad *msDS-NeverRevealGroup* del *objeto Computer del servidor Kerberos de Azure AD (CN=AzureADKerberos,OU=Domain Controllers,\<domain-DN>)* .

## <a name="under-the-hood"></a>En segundo plano

* [¿Cómo se vincula el servidor Kerberos de Azure AD a mi entorno de Active Directory Domain Services local?](#how-is-azure-ad-kerberos-linked-to-my-on-premises-active-directory-domain-services-environment)
* [¿Dónde puedo ver estos objetos de servidor Kerberos creados en AD DS y publicados en Azure AD?](#where-can-i-view-these-kerberos-server-objects-that-are-created-in-ad-ds-and-published-in-azure-ad)
* [¿Por qué no puedo registrar la clave pública en la instancia de AD DS local para no tener que depender de Internet?](#why-cant-we-have-the-public-key-registered-to-on-premises-ad-ds-so-there-is-no-dependency-on-the-internet)
* [¿Cómo se rotan las claves en el objeto de servidor Kerberos?](#how-are-the-keys-rotated-on-the-kerberos-server-object)
* [¿Por qué necesitamos Azure AD Connect? ¿Se vuelve a escribir información en AD DS desde Azure AD?](#why-do-we-need-azure-ad-connect-does-it-write-any-info-back-to-ad-ds-from-azure-ad)
* [¿Qué aspecto tiene la solicitud/respuesta HTTP al solicitar el TGT parcial de PRT+?](#what-does-the-http-requestresponse-look-like-when-requesting-prt-partial-tgt)

### <a name="how-is-azure-ad-kerberos-linked-to-my-on-premises-active-directory-domain-services-environment"></a>¿Cómo se vincula el servidor Kerberos de Azure AD a mi entorno de Active Directory Domain Services local?

Hay dos partes: el entorno de AD DS local y el inquilino de Azure AD.

**Active Directory Domain Services (AD DS)**

El servidor Kerberos de Azure AD se representa en un entorno de AD DS local como un objeto de controlador de dominio (DC). Este objeto de controlador de dominio se compone a su vez de varios objetos:

* *CN=AzureADKerberos,OU=Domain Controllers,\<domain-DN>*
    
    Un objeto *Computer* que representa un controlador de dominio de solo lectura (RODC) en AD DS. No hay ningún equipo asociado a este objeto. En su lugar, es una representación lógica de un controlador de dominio.

* *CN=krbtgt_AzureAD,CN=Users,\<domain-DN>*

    Un objeto *User* que representa una clave de cifrado del vale de concesión de vales (TGT) de Kerberos del controlador de dominio de solo lectura.

* *CN=900274c4-b7d2-43c8-90ee-00a9f650e335,CN=AzureAD,CN=System,\<domain-DN>*

    Un objeto *ServiceConnectionPoint* que almacena metadatos de los objetos de servidor Kerberos de Azure AD. Las herramientas administrativas utilizan este objeto para identificar y ubicar los objetos del servidor Kerberos de Azure AD.

**Azure Active Directory**

El objeto del servidor Kerberos de Azure AD se representa en Azure AD como un objeto *KerberosDomain*. Cada entorno de AD DS local se representa como un único objeto *KerberosDomain* en el inquilino de Azure AD.

Por ejemplo, puede tener un bosque de AD DS con dos dominios como *contoso.com* y *fabrikam.com*. Si permite que Azure AD emita vales de concesión de vales (TGT) de Kerberos para todo el bosque, habrá dos objetos *KerberosDomain* en Azure AD: un objeto para *contoso.com* y otro para *fabrikam.com*.

Si tiene varios bosques de AD DS, tendrá un objeto *KerberosDomain* para cada dominio de cada bosque.

### <a name="where-can-i-view-these-kerberos-server-objects-that-are-created-in-ad-ds-and-published-in-azure-ad"></a>¿Dónde puedo ver estos objetos de servidor Kerberos creados en AD DS y publicados en Azure AD?

Para ver todos los objetos, use los cmdlets de PowerShell del servidor Kerberos de Azure AD que se incluyen con la versión más reciente de Azure AD Connect.

Para obtener más información, incluidas las instrucciones para ver los objetos, consulte la sección sobre cómo [crear objetos del servidor Kerberos](howto-authentication-passwordless-security-key-on-premises.md#create-kerberos-server-object).

### <a name="why-cant-we-have-the-public-key-registered-to-on-premises-ad-ds-so-there-is-no-dependency-on-the-internet"></a>¿Por qué no puedo registrar la clave pública en la instancia de AD DS local para no tener que depender de Internet?

Hemos recibido comentarios sobre la complejidad del modelo de implementación de Windows Hello para empresas, por lo que queríamos simplificarlo sin tener que usar certificados ni PKI (FIDO2 no usa certificados).

### <a name="how-are-the-keys-rotated-on-the-kerberos-server-object"></a>¿Cómo se rotan las claves en el objeto de servidor Kerberos?

Las claves *krbtgt* de cifrado del servidor Kerberos de Azure AD se deben rotar periódicamente, al igual que con cualquier otro controlador de dominio. Se recomienda seguir la misma programación que para girar las demás claves *krbtgt* de AD DS.

> [!NOTE]
> Aunque hay otras herramientas para rotar las claves *krbtgt*, debe [usar los cmdlets de PowerShell para rotar las claves *krbtgt*](howto-authentication-passwordless-security-key-on-premises.md#rotating-the-azure-ad-kerberos-server-key) del servidor Kerberos de Azure AD. Este método garantiza que las claves se actualicen en el entorno de AD DS local y en Azure AD.

### <a name="why-do-we-need-azure-ad-connect-does-it-write-any-info-back-to-ad-ds-from-azure-ad"></a>¿Por qué necesitamos Azure AD Connect? ¿Se vuelve a escribir información en AD DS desde Azure AD?

Azure AD Connect no vuelve a escribir información en AD DS desde Azure AD. La utilidad incluye el módulo de PowerShell para crear el objeto del servidor Kerberos en AD DS y publicarlo en Azure AD.

### <a name="what-does-the-http-requestresponse-look-like-when-requesting-prt-partial-tgt"></a>¿Qué aspecto tiene la solicitud/respuesta HTTP al solicitar el TGT parcial de PRT+?

La solicitud HTTP es una solicitud de token de actualización principal (PRT) estándar. Esta solicitud de PRT incluye una notificación que indica que se necesita un vale de concesión de vales (TGT) de Kerberos.

| Notificación | Value | Descripción                             |
|-------|-------|-----------------------------------------|
| tgt   | true  | La notificación indica que el cliente necesita un TGT. |

Azure AD combina la clave de cliente cifrada y el búfer de mensajes en la respuesta de PRT como propiedades adicionales. La carga útil se cifra mediante la clave de sesión de dispositivos de Azure AD.

| Campo              | Tipo   | Descripción  |
|--------------------|--------|--------------|
| tgt_client_key     | string | Clave de cliente codificada en Base64 (secreto). Esta clave es el secreto de cliente que se usa para proteger el TGT. En este escenario sin contraseña, el servidor genera el secreto de cliente como parte de cada solicitud TGT y, a continuación, se devuelve al cliente en la respuesta. |
| tgt_key_type       | int    | El tipo de clave de AD DS local utilizado para la clave de cliente y la clave de sesión de Kerberos incluidos en KERB_MESSAGE_BUFFER. |
| tgt_message_buffer | string | KERB_MESSAGE_BUFFER codificado en Base64. |

## <a name="next-steps"></a>Pasos siguientes

Para ver una introducción de las llaves de seguridad FIDO2 y del acceso híbrido a recursos locales, consulte los siguientes artículos:

* [Claves de seguridad FIDO2 sin contraseñas](howto-authentication-passwordless-security-key.md)
* [Windows 10 sin contraseña](howto-authentication-passwordless-security-key-windows.md)
* [Entorno local sin contraseña](howto-authentication-passwordless-security-key-on-premises.md)
