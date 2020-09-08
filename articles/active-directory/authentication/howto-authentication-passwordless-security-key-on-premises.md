---
title: 'Inicio de sesión con una clave de seguridad sin contraseña en recursos locales (versión preliminar): Azure Active Directory'
description: Información sobre cómo habilitar el inicio de sesión con una clave de seguridad sin contraseña en recursos locales con Azure Active Directory (versión preliminar)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/09/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9f20da2d2ecb4426c0deb1c01591ead5933090f6
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717003"
---
# <a name="enable-passwordless-security-key-sign-in-to-on-premises-resources-with-azure-active-directory-preview"></a>Habilitar el inicio de sesión con una clave de seguridad sin contraseña en recursos locales con Azure Active Directory (versión preliminar)

Este documento gira en torno a la habilitación de la autenticación sin contraseña en los recursos locales en entornos con dispositivos Windows 10 **unidos a Azure AD** y **unidos a Azure AD híbrido**. Esta funcionalidad proporciona inicio de sesión único (SSO) de conexión directa a recursos locales mediante claves de seguridad compatibles con Microsoft.

> [!NOTE]
> Las claves de seguridad FIDO2 son una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="sso-to-on-premises-resources-using-fido2-keys"></a>Inicio de sesión único en los recursos locales mediante claves de FIDO2

Azure Active Directory (AD) puede emitir vales de concesión de vales (TGT) de Kerberos para uno o varios de los dominios de Active Directory. Esta funcionalidad permite que los usuarios inicien sesión en Windows con credenciales modernas, como las claves de seguridad de FIDO2, y acceder a los recursos tradicionales basados en Active Directory. Los vales de servicio de Kerberos y la autorización se siguen controlando mediante los controladores de dominio de Active Directory local.

Se crea un objeto de servidor Kerberos de Azure AD en Active Directory local y, luego, se publica de forma segura en Azure Active Directory. El objeto no está asociado a ningún servidor físico. Simplemente es un recurso que se puede usar en Azure Active Directory para generar TGT de Kerberos para el dominio de Active Directory.

![Obtener un vale de concesión de vales (TGT) de Azure AD y AD DS](./media/howto-authentication-passwordless-on-premises/fido2-ticket-granting-ticket-exchange-process.png)

1. El usuario inicia sesión en su dispositivo Windows 10 con una clave de seguridad de FIDO2 y se autentica en Azure AD.
1. Azure AD comprueba el directorio en busca de una clave de servidor Kerberos que coincida con el dominio de AD local del usuario.
   1. Azure AD genera un TGT de Kerberos para el dominio de AD local del usuario. El TGT solo incluye el SID del usuario. No incluye datos de autorización.
1. El TGT se devuelve al cliente junto con su token de actualización principal (PRT) de Azure AD.
1. La máquina cliente se pone en contacto con un controlador de dominio de AD local e intercambia el TGT parcial por un TGT completo.
1. La máquina cliente tiene ahora un PRT de Azure AD y un TGT de Active Directory completo y puede acceder a recursos locales y en la nube.

## <a name="requirements"></a>Requisitos

Las organizaciones deben completar los pasos de [Habilitar el inicio de sesión con clave de seguridad sin contraseña en dispositivos Windows 10 (versión preliminar)](howto-authentication-passwordless-security-key.md) antes de realizar los pasos de este artículo.

Las organizaciones también deben cumplir los siguientes requisitos de software.

- Los dispositivos deben ejecutar Windows 10 Insider Build 18945 o una versión posterior.
- Debe tener la versión 1.4.32.0 o una versión posterior de [Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md#install-azure-ad-connect).
  - Para más información sobre las opciones de autenticación híbrida de Azure AD disponibles, consulte el artículo sobre cómo [seleccionar el método de autenticación adecuado para la solución de identidad híbrida de Azure Active Directory](../hybrid/choose-ad-authn.md) y cómo [seleccionar el tipo de instalación que se va a usar para Azure AD Connect](../hybrid/how-to-connect-install-select-installation.md).
- Los controladores de dominio de Windows Server deben tener instaladas las siguientes revisiones:
    - Para Windows Server 2016: https://support.microsoft.com/help/4534307/windows-10-update-kb4534307
    - Para Windows Server 2019: https://support.microsoft.com/help/4534321/windows-10-update-kb4534321

### <a name="supported-scenarios"></a>Escenarios admitidos

El escenario admite el inicio de sesión único (SSO) en los dos escenarios siguientes:

- Recursos de nube como Office 365 y otras aplicaciones habilitadas para SAML.
- Recursos locales y autenticación integrada de Windows en sitios web. Los recursos pueden incluir sitios web y sitios de SharePoint que requieran autenticación de IIS o recursos que usan la autenticación NTLM.

### <a name="unsupported-scenarios"></a>Escenarios no admitidos

No se admiten los escenarios siguientes:

- Implementación de Windows Server Active Directory Domain Services (AD DS) unido a un dominio (solo en dispositivos locales).
- Escenarios de RDP, VDI y Citrix con una clave de seguridad.
- S/MIME con una clave de seguridad.
- "Ejecutar como" con una clave de seguridad.
- Inicio de sesión en un servidor con una clave de seguridad.

## <a name="create-kerberos-server-object"></a>Creación de objetos de servidor Kerberos

Los administradores usan herramientas de PowerShell de su servidor de Azure AD Connect para crear un objeto de servidor Kerberos de Azure AD en su directorio local. Estos pasos deben ejecutarse en todos los dominios y bosques de la organización que contengan usuarios de Azure AD:

1. Actualice a la versión más reciente de Azure AD Connect. En las instrucciones se supone que ya ha configurado Azure AD Connect para admitir su entorno híbrido.
1. En el servidor de Azure AD Connect, abra un símbolo del sistema de PowerShell con privilegios elevados y vaya a `C:\Program Files\Microsoft Azure Active Directory Connect\AzureADKerberos\`.
1. Ejecute los siguientes comandos de PowerShell para crear un objeto de servidor Kerberos de Azure AD en el dominio de Active Directory local y en el inquilino de Azure Active Directory.

> [!NOTE]
> Reemplace `contoso.corp.com` del ejemplo siguiente por el nombre de dominio de Active Directory local.

```powerShell
Import-Module ".\AzureAdKerberos.psd1"

# Specify the on-premises Active Directory domain. A new Azure AD
# Kerberos Server object will be created in this Active Directory domain.
$domain = "contoso.corp.com"

# Enter an Azure Active Directory global administrator username and password.
$cloudCred = Get-Credential

# Enter a domain administrator username and password.
$domainCred = Get-Credential

# Create the new Azure AD Kerberos Server object in Active Directory
# and then publish it to Azure Active Directory.
Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

### <a name="viewing-and-verifying-the-azure-ad-kerberos-server"></a>Visualización y comprobación del servidor Kerberos de Azure AD

Para ver y comprobar el servidor Kerberos de Azure AD recién creado, use el siguiente comando:

```powerShell
Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

Este comando genera las propiedades del servidor Kerberos de Azure AD. Puede revisar las propiedades para comprobar que todo está en orden.

| Propiedad | Descripción |
| --- | --- |
| ID | Identificador único del objeto controlador de dominio de AD DS. En ocasiones, a este identificador se le conoce como "ranura" o "id. de rama". |
| DomainDnsName | Nombre de dominio DNS del dominio de Active Directory. |
| ComputerAccount | Objeto de cuenta de equipo del objeto de servidor Kerberos de Azure AD (controlador de dominio). |
| UserAccount | Objeto de cuenta de usuario deshabilitado que contiene la clave de cifrado del TGT del servidor Kerberos de Azure AD. El DN de esta cuenta es `CN=krbtgt_AzureAD,CN=Users,<Domain-DN>`. |
| KeyVersion | Versión de la clave de cifrado del TGT del servidor Kerberos de Azure AD. La versión se asigna cuando se crea la clave. Luego se incrementa cada vez que se rota la clave. Los incrementos se basan en los metadatos de replicación y probablemente serán mayores que uno. Por ejemplo, el valor inicial de *KeyVersion* podría ser *192272*. La primera vez que se gira la clave, la versión podría avanzar hasta *212621*. Lo importante es comprobar que el valor de *KeyVersion* del objeto local y el valor de *CloudKeyVersion* del objeto de nube sean los mismos. |
| KeyUpdatedOn | Fecha y hora en que se actualizó o creó la clave de cifrado del TGT del servidor Kerberos de Azure AD. |
| KeyUpdatedFrom | Controlador de dominio en el que se actualizó por última vez la clave de cifrado del TGT del servidor Kerberos de Azure AD. |
| CloudId | Identificador del objeto de Azure AD. Debe coincidir con el identificador anterior. |
| CloudDomainDnsName | Valor de *DomainDnsName* del objeto de Azure AD. Debe coincidir con el valor de *DomainDnsName* anterior. |
| CloudKeyVersion | Valor de *KeyVersion* del objeto de Azure AD. Debe coincidir con el valor de *KeyVersion* anterior. |
| CloudKeyUpdatedOn | Valor de *KeyUpdatedOn* del objeto de Azure AD. Debe coincidir con el valor de *KeyUpdatedOn* anterior. |

### <a name="rotating-the-azure-ad-kerberos-server-key"></a>Rotación de la clave de servidor Kerberos de Azure AD

Las claves krbtgt de cifrado del servidor Kerberos de Azure AD se deben rotar periódicamente. Se recomienda seguir la misma programación que para girar todas las demás claves krbtgt del controlador de dominio de Active Directory.

> [!WARNING]
> Aunque hay otras herramientas que podrían rotar las claves krbtgt, debe usar las que se mencionan en este documento para rotar las claves krbtgt de su servidor Kerberos de Azure AD. De esta forma, se garantiza que las claves se actualizan en AD local y en Azure AD.

```powerShell
Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred -RotateServerKey
```

### <a name="removing-the-azure-ad-kerberos-server"></a>Eliminación del servidor Kerberos de Azure AD

Si quiere revertir el escenario y quitar el servidor Kerberos de Azure AD de Active Directory local y Azure Active Directory, ejecute el siguiente comando:

```powerShell
Remove-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

### <a name="multi-forest-and-multi-domain-scenarios"></a>Escenarios de varios bosques y dominios

El objeto de servidor Kerberos de Azure AD se representa en Azure AD como un objeto *KerberosDomain*. Cada dominio de Active Directory local se representa como un único objeto *KerberosDomain* en Azure AD.

Por ejemplo, su organización tiene un bosque de Active Directory con dos dominios: `contoso.com` y `fabrikam.com`. Si opta por permitir que Azure AD emita vales de concesión de vales (TGT) de Kerberos para todo el bosque, habrá dos objetos *KerberosDomain* en Azure AD. Un objeto *KerberosDomain* para `contoso.com` y otro para `fabrikam.com`. Si tiene varios bosques de Active Directory, tendrá un objeto *KerberosDomain* para cada dominio de cada bosque.

Debe ejecutar los pasos descritos en [Creación de objetos de servidor Kerberos](#create-kerberos-server-object) en cada dominio y bosque de su organización que contenga usuarios de Azure AD.

## <a name="known-behavior"></a>Comportamiento conocido

Si la contraseña ha expirado, se bloquea el inicio de sesión con FIDO. La expectativa es que el usuario restablezca su contraseña antes de poder iniciar sesión con FIDO.

## <a name="troubleshooting-and-feedback"></a>Solución de problemas y comentarios

Si quiere compartir comentarios o detectar problemas mientras usa la versión preliminar de esta característica, compártalos mediante la aplicación Centro de opiniones sobre Windows. Para ello, realice los pasos siguientes:

1. Abra el **Centro de opiniones** y asegúrese de que ha iniciado sesión.
1. Envíe los comentarios bajo la categorización siguiente:
   - Categoría: Seguridad y privacidad
   - Subcategoría: FIDO
1. Para capturar registros, use la opción **Recreate my Problem** (Recrear mi problema).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="does-this-work-in-my-on-premises-environment"></a>¿Esto funciona en mi entorno local?

Esta característica no funciona para un entorno de Active Directory Domain Services (AD DS) local puro.

### <a name="my-organization-requires-two-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement"></a>Mi organización requiere la autenticación en dos fases para acceder a los recursos. ¿Qué puedo hacer para admitir este requisito?

Las claves de seguridad tienen varios factores de forma. Póngase en contacto con el fabricante del dispositivo concreto para analizar cómo se pueden habilitar sus dispositivos con un PIN o reconocimiento biométrico como segundo factor.

### <a name="can-admins-set-up-security-keys"></a>¿Los administradores pueden configurar claves de seguridad?

Estamos trabajando en esta funcionalidad para la disponibilidad general (GA) de esta característica.

### <a name="where-can-i-go-to-find-compliant-security-keys"></a>¿Dónde puedo encontrar claves de seguridad compatibles?

[Llaves de seguridad FIDO2](concept-authentication-passwordless.md#fido2-security-keys)

### <a name="what-do-i-do-if-i-lose-my-security-key"></a>¿Qué hago si pierdo mi clave de seguridad?

Para quitar las claves de Azure Portal, navegue a la página de **información de seguridad** y quite la clave de seguridad.

### <a name="im-not-able-to-use-fido-immediately-after-i-create-a-hybrid-azure-ad-joined-machine"></a>No puedo usar FIDO inmediatamente después de crear una máquina unida a Azure AD híbrido

Si realiza una instalación limpia de una máquina unida a Azure AD híbrido después del proceso de unión a un dominio y reinicio, debe iniciar sesión con una contraseña y esperar a que se sincronice la directiva para poder usar FIDO para iniciar sesión.

- Para comprobar el estado actual, escriba `dsregcmd /status` en una ventana de comandos y compruebe que *AzureAdJoined* y *DomainJoined* muestran *YES*.
- Este retraso es una limitación conocida de los dispositivos unidos a un dominio y no es específico de FIDO.

### <a name="im-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-fido-and-get-a-credential-prompt"></a>No puedo obtener el SSO en el recurso de red NTLM después de iniciar sesión con FIDO y obtener una petición de credenciales

Asegúrese de que se han revisado suficientes controladores de dominio para responder a tiempo para atender la solicitud de recursos. Para comprobar si puede ver un controlador de dominio que ejecute la característica, revise la salida de `nltest /dsgetdc:contoso /keylist /kdc`.

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre la ausencia de contraseñas](concept-authentication-passwordless.md)