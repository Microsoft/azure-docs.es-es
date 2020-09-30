---
title: 'Problemas conocidos y solución de problemas con la implementación híbrida de llaves de seguridad FIDO2: Azure Active Directory'
description: Más información sobre algunos de los problemas conocidos (y cómo solucionarlos) relacionados con la implementación híbrida del inicio de sesión sin contraseña con llaves de seguridad FIDO2 a través de Azure Active Directory (versión preliminar)
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
ms.openlocfilehash: ecab82e43bff6c0d1d83c9c1cdc38cafd809e277
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89236558"
---
# <a name="troubleshooting-for-hybrid-deployments-of-fido2-security-keys-in-azure-ad-preview"></a>Solución de problemas de implementaciones híbridas de llaves de seguridad FIDO2 en Azure AD (versión preliminar)

En este artículo se tratan las preguntas más frecuentes sobre los dispositivos unidos a Azure AD híbrido y el inicio de sesión sin contraseña en recursos locales. Con esta característica sin contraseña, puede habilitar la autenticación de Azure AD en dispositivos con Windows 10 unidos a Azure AD híbrido mediante llaves de seguridad FIDO2. Los usuarios pueden iniciar sesión en Windows en sus dispositivos usando credenciales modernas como las llaves FIDO2, y acceder a recursos tradicionales basados en Active Directory Domain Services (AD DS) con una experiencia de inicio de sesión único (SSO) que permite acceder directamente a sus recursos locales.

Se admiten los siguientes escenarios para los usuarios de un entorno híbrido:

* Inicio de sesión en dispositivos unidos a Azure AD híbrido mediante llaves de seguridad FIDO2 para acceder con SSO a recursos locales
* Inicio de sesión en dispositivos unidos a Azure AD con llaves de seguridad FIDO2 para acceder con SSO a recursos locales

Para ver una introducción de las llaves de seguridad FIDO2 y del acceso híbrido a recursos locales, consulte los siguientes artículos:

* [Claves de seguridad sin contraseñas](howto-authentication-passwordless-security-key.md)
* [Windows 10 sin contraseña](howto-authentication-passwordless-security-key-windows.md)
* [Recursos locales sin contraseña](howto-authentication-passwordless-security-key-on-premises.md)

> [!NOTE]
> Las claves de seguridad FIDO2 son una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="known-issues"></a>Problemas conocidos

* [Los usuarios no pueden iniciar sesión con las llaves de seguridad FIDO2, ya que la autenticación facial de Windows Hello es demasiado rápida y es el método de inicio de sesión predeterminado](#users-are-unable-to-sign-in-using-fido2-security-keys-as-windows-hello-face-is-too-quick-and-is-the-default-sign-in-mechanism)
* [Los usuarios no pueden usar llaves de seguridad FIDO2 inmediatamente después de crear una máquina unida a Azure AD híbrido](#users-arent-able-to-use-fido2-security-keys-immediately-after-they-create-a-hybrid-azure-ad-joined-machine)
* [Los usuarios no pueden acceder mediante SSO al recurso de red NTLM después de iniciar sesión con una llave de seguridad FIDO2 y recibir una petición de credenciales](#users-are-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-a-fido2-security-key-and-receiving-a-credential-prompt)

### <a name="users-are-unable-to-sign-in-using-fido2-security-keys-as-windows-hello-face-is-too-quick-and-is-the-default-sign-in-mechanism"></a>Los usuarios no pueden iniciar sesión con las llaves de seguridad FIDO2, ya que la autenticación facial de Windows Hello es demasiado rápida y es el método de inicio de sesión predeterminado

La autenticación facial de Windows Hello es la mejor experiencia en aquellos dispositivos en los que un usuario está inscrito. Las llaves de seguridad FIDO2 están diseñadas para usarse en dispositivos compartidos o donde la inscripción de Windows Hello para empresas supone una barrera.

Si la autenticación facial de Windows Hello impide que los usuarios prueben el escenario de inicio de sesión con llaves de seguridad FIDO2, los usuarios pueden desactivar el inicio de sesión facial de Hello quitando la inscripción facial en **Configuración > Opciones de inicio de sesión**.

### <a name="users-arent-able-to-use-fido2-security-keys-immediately-after-they-create-a-hybrid-azure-ad-joined-machine"></a>Los usuarios no pueden usar llaves de seguridad FIDO2 inmediatamente después de crear una máquina unida a Azure AD híbrido

Si realiza una instalación limpia de una máquina unida a Azure AD híbrido después del proceso de unión a un dominio y reinicio, debe iniciar sesión con una contraseña y esperar a que se sincronice la directiva para poder usar una llave de seguridad FIDO2 para iniciar sesión.

Este comportamiento es una limitación conocida de los dispositivos unidos a un dominio y no es específico de las llaves de seguridad FIDO2.

Para comprobar el estado actual, use el comando `dsregcmd /status`. Compruebe que los valores de *AzureAdJoined* y *DomainJoined* estén establecidos en *YES*.

### <a name="users-are-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-a-fido2-security-key-and-receiving-a-credential-prompt"></a>Los usuarios no pueden acceder mediante SSO al recurso de red NTLM después de iniciar sesión con una llave de seguridad FIDO2 y recibir una petición de credenciales

Asegúrese de que se hayan revisado suficientes controladores de dominio para responder a tiempo y atender la solicitud de recursos. Para comprobar si puede ver un controlador de dominio que ejecute la característica, revise la salida de `nltest /dsgetdc:<dc name> /keylist /kdc`.

Si puede ver un controlador de dominio con esta característica, es posible que la contraseña del usuario haya cambiado desde que se inició sesión o que haya otro problema. Recopile los registros como se detalla en la sección siguiente para que el equipo de soporte técnico de Microsoft los depure.

## <a name="troubleshoot"></a>Solución de problemas

Hay dos áreas en las que solucionar problemas: [Problemas del cliente Windows](#windows-client-issues)o [Problemas de implementación](#deployment-issues).

### <a name="windows-client-issues"></a>Problemas del cliente Windows

Para recopilar datos que ayuden a solucionar problemas al iniciar sesión en Windows o acceder a recursos locales desde dispositivos con Windows 10, siga estos pasos:

1. Abra la aplicación **Centro de opiniones**. Asegúrese de que su nombre se muestra en la parte inferior izquierda de la aplicación y, a continuación, seleccione **Create a new feedback item (Crear un elemento de comentarios)** .

    En el tipo de elemento de comentarios, elija *Problema*.

1. Seleccione la categoría *Seguridad y privacidad* y, después, la subcategoría *FIDO*.
1. Active la casilla *Send attached files and diagnostics to Microsoft along with my feedback* (Enviar archivos y diagnósticos adjuntos a Microsoft junto con mis comentarios).
1. Seleccione *Recreate my problems (Volver a crear mis problemas)* y, a continuación, *Iniciar captura*.
1. Bloquee y desbloquee la máquina con la llave de seguridad FIDO2. Si se produce el problema, intente desbloquearlo con otras credenciales.
1. Vuelva a **Centro de opiniones**, seleccione **Detener captura** y envíe sus comentarios.
1. Vaya a la página *Comentarios* y, después, a la pestaña *Mis comentarios*. Seleccione los comentarios enviados recientemente.
1. Seleccione el botón *Compartir* en la esquina superior derecha para obtener un vínculo a los comentarios. Incluya este vínculo al abrir un caso de soporte técnico o responda al ingeniero asignado a un caso de soporte técnico.

Se recopila la siguiente información de claves del Registro y registros de eventos:

**Claves del Registro**

* *HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\FIDO [\*]*
* *HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\PasswordForWork\* [\*]*
* *HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Policies\PasswordForWork\* [\*]*

**Información de diagnóstico**

* Volcado de Kernel activo
* Información de los paquetes AppX recopilados
* Archivos de contexto UIF

**Registros de eventos**

* *%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-AAD%40Operational.evtx*
* *%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-WebAuthN%40Operational.evtx*
* *%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-HelloForBusiness%40Operational.evtx*

### <a name="deployment-issues"></a>Problemas de implementación

Para solucionar problemas relacionados con la implementación del servidor Kerberos de Azure AD, use el nuevo módulo de PowerShell que se incluye con Azure AD Connect.

#### <a name="viewing-the-logs"></a>Visualización de los registros

Los cmdlets de PowerShell del servidor Kerberos de Azure AD usan el mismo registro que el Asistente de Azure AD Connect estándar. Para ver información o detalles de errores de los cmdlets, siga estos pasos:

1. En el servidor de Azure AD Connect, vaya a `C:\ProgramData\AADConnect\`. Esta carpeta está oculta de forma predeterminada.
1. Abra y vea el archivo `trace-*.log` más reciente que se encuentra en el directorio.

#### <a name="viewing-the-azure-ad-kerberos-server-objects"></a>Visualización de objetos del servidor Kerberos de Azure AD

Para ver los objetos del servidor Kerberos de Azure AD y comprobar que están en buen estado, siga estos pasos:

1. En el servidor de Azure AD Connect, abra un símbolo del sistema de PowerShell con privilegios elevados y vaya a `C:\Program Files\Microsoft Azure Active Directory Connect\AzureADKerberos\`.
1. Ejecute los siguientes comandos de PowerShell para ver el servidor Kerberos desde Azure AD y de AD DS local.

    Reemplace *corp.contoso.com* por el nombre de su dominio de AD DS local.

    ```powershell
    Import-Module ".\AzureAdKerberos.psd1"

    # Specify the on-premises AD DS domain.
    $domain = "corp.contoso.com"

    # Enter an Azure Active Directory Global Admin username and password.
    $cloudCred = Get-Credential

    # Enter a Domain Admin username and password.
    $domainCred = Get-Credential

    # Get the Azure AD Kerberos Server Object
    Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential
    $domainCred
    ```

El comando genera las propiedades del servidor Kerberos de Azure AD desde Azure AD y AD DS local. Puede revisar las propiedades para comprobar que todo está en orden. Use la tabla siguiente para comprobar las propiedades.

El primer conjunto de propiedades procede de los objetos del entorno de AD DS local. La segunda mitad (las propiedades que comienzan por *Cloud**) proceden del objeto del servidor Kerberos de Azure AD:

| Propiedad           | Descripción  |
|--------------------|--------------|
| Identificador                 | *Identificador* único del objeto de controlador de dominio de AD DS. |
| DomainDnsName      | El nombre de dominio DNS del dominio de AD DS. |
| ComputerAccount    | Objeto de cuenta de equipo del objeto de servidor Kerberos de Azure AD (controlador de dominio). |
| UserAccount        | Objeto de cuenta de usuario deshabilitado que contiene la clave de cifrado del TGT del servidor Kerberos de Azure AD. El nombre de dominio de esta cuenta es *CN=krbtgt_AzureAD,CN=Users,<Domain-DN>* . |
| KeyVersion         | Versión de la clave de cifrado del TGT del servidor Kerberos de Azure AD. La versión se asigna cuando se crea la clave. Luego se incrementa cada vez que se rota la clave. Los incrementos se basan en los metadatos de replicación y probablemente serán mayores que uno.<br /><br /> Por ejemplo, el valor inicial de *KeyVersion* podría ser *192272*. La primera vez que se gira la clave, la versión podría avanzar hasta *212621*.<br /><br /> Lo importante es comprobar que el valor de *KeyVersion* del objeto local y el valor de *CloudKeyVersion* del objeto de nube sean los mismos. |
| KeyUpdatedOn       | Fecha y hora en que se actualizó o creó la clave de cifrado del TGT del servidor Kerberos de Azure AD. |
| KeyUpdatedFrom     | Controlador de dominio en el que se actualizó por última vez la clave de cifrado del TGT del servidor Kerberos de Azure AD. |
| CloudId            | El *identificador* del objeto de Azure AD. Debe coincidir con el *identificador* anterior. |
| CloudDomainDnsName | Valor de *DomainDnsName* del objeto de Azure AD. Debe coincidir con el valor de *DomainDnsName* anterior. |
| CloudKeyVersion    | Valor de *KeyVersion* del objeto de Azure AD. Debe coincidir con el valor de *KeyVersion* anterior. |
| CloudKeyUpdatedOn  | Valor de *KeyUpdatedOn* del objeto de Azure AD. Debe coincidir con el valor de *KeyUpdatedOn* anterior. |

## <a name="next-steps"></a>Pasos siguientes

Para ver una introducción de las llaves de seguridad FIDO2 y del acceso híbrido a recursos locales, consulte los siguientes artículos:

* [Claves de seguridad FIDO2 sin contraseñas](howto-authentication-passwordless-security-key.md)
* [Windows 10 sin contraseña](howto-authentication-passwordless-security-key-windows.md)
* [Entorno local sin contraseña](howto-authentication-passwordless-security-key-on-premises.md)
