---
title: Solución de problemas de inicio de sesión en Azure AD Domain Services | Microsoft Docs
description: Aprenda a solucionar problemas y errores comunes del inicio de sesión en Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 91c42e391d1a23e9ac494af9271c92e3e9b5d6c4
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722660"
---
# <a name="troubleshoot-account-sign-in-problems-with-an-azure-active-directory-domain-services-managed-domain"></a>Solución de problemas del inicio de sesión de cuentas con un dominio administrado de Azure Active Directory Domain Services

Los motivos más comunes por los que una cuenta de usuario no puede iniciar sesión en un dominio administrado de Azure Active Directory Domain Services (Azure AD DS) incluyen los siguientes escenarios:

* [La cuenta no se ha sincronizado todavía en Azure AD DS.](#account-isnt-synchronized-into-azure-ad-ds-yet)
* [Azure AD DS no tiene los valores hash de contraseña para permitir que la cuenta inicie sesión.](#azure-ad-ds-doesnt-have-the-password-hashes)
* [La cuenta está bloqueada.](#the-account-is-locked-out)

> [!TIP]
> Azure AD DS no puede sincronizar las credenciales en las cuentas que son externas al inquilino de Azure AD. Los usuarios externos no pueden iniciar sesión en el dominio administrado de Azure AD DS.

## <a name="account-isnt-synchronized-into-azure-ad-ds-yet"></a>La cuenta no se ha sincronizado todavía en Azure AD DS.

En función del tamaño de su directorio, es posible que las cuentas de usuario y los hash de credenciales tarden algo en estar disponibles en un dominio administrado. En el caso de directorios grandes, esta sincronización unidireccional inicial desde Azure AD puede tardar unas horas y hasta uno o dos días. Asegúrese de esperar lo suficiente antes de volver a intentar la autenticación.

En el caso de entornos híbridos en los que el usuario de Azure AD Connect sincronice los datos de los directorios en el entorno local en Azure AD, asegúrese de ejecutar la versión más reciente de Azure AD Connect y de haber [configurado Azure AD Connect para realizar una sincronización completa después de habilitar Azure AD DS][azure-ad-connect-phs]. Si deshabilita Azure AD DS y, a continuación, vuelve a habilitarlo, tendrá que seguir estos pasos de nuevo.

Si sigue teniendo problemas con la sincronización de las cuentas mediante Azure AD Connect, reinicie los Servicios de sincronización de Azure AD. En el equipo que tiene instalado Azure AD Connect, abra una ventana del símbolo del sistema y ejecute los siguientes comandos:

```console
net stop 'Microsoft Azure AD Sync'
net start 'Microsoft Azure AD Sync'
```

## <a name="azure-ad-ds-doesnt-have-the-password-hashes"></a>Azure AD DS no tiene los hash de contraseña

A menos que habilite Azure AD DS para el inquilino, Azure AD no genera ni almacena los hash de las contraseñas en el formato necesario para la autenticación NTLM o Kerberos. Por motivos de seguridad, Azure AD tampoco almacena las credenciales de contraseñas en forma de texto sin cifrar. Por consiguiente, Azure AD no tiene forma de generar automáticamente estos hash de las contraseñas de NTLM o Kerberos basándose en las credenciales existentes de los usuarios.

### <a name="hybrid-environments-with-on-premises-synchronization"></a>Entornos híbridos con sincronización en el entorno local

En el caso de entornos híbridos que usan Azure AD Connect para sincronizar desde un entorno de AD DS local, puede generar y sincronizar localmente los valores hash de contraseña NTLM o Kerberos necesarios en Azure AD. Tras crear el dominio administrado, [habilite la sincronización de hash de contraseña para Azure Active Directory Domain Services][azure-ad-connect-phs]. Sin completar este paso de sincronización de hash de contraseña, no puede iniciar sesión en una cuenta mediante el dominio administrado. Si deshabilita Azure AD DS y, a continuación, vuelve a habilitarlo, tendrá que seguir estos pasos de nuevo.

Para más información, consulte [Implementación de la sincronización de contraseña mediante la sincronización de Azure AD DS][phs-process].

### <a name="cloud-only-environments-with-no-on-premises-synchronization"></a>Entornos solo en la nube sin sincronización en el entorno local

Los dominios administrados sin sincronización en el entorno local (solo cuentas de Azure AD), también deben generar los valores hash de contraseña NTLM o Kerberos necesarios. Si una cuenta solo en la nube no puede iniciar sesión, ¿se ha completado correctamente un proceso de cambio de contraseña para la cuenta después de habilitar Azure AD DS?

* **No, la contraseña no se ha cambiado.**
    * [Cambie la contraseña de la cuenta][enable-user-accounts] para generar los valores hash de contraseña necesarios y espere 15 minutos antes de intentar volver a iniciar sesión.
    * Si deshabilita Azure AD DS y, a continuación, vuelve a habilitarlo, cada cuenta debe seguir los pasos de nuevo para cambiar su contraseña y generar los valores hash de contraseña necesarios.
* **Sí, la contraseña se ha cambiado.**
    * Pruebe a iniciar sesión con el formato *UPN*, por ejemplo `driley@aaddscontoso.com`, en lugar de *SAMAccountName* como `AADDSCONTOSO\deeriley`.
    * El atributo *SAMAccountName* puede generarse automáticamente para los usuarios cuyo prefijo UPN sea demasiado largo o coincida con el de otro usuario del dominio administrado. El formato *UPN* garantiza que sea único dentro de un inquilino de Azure AD.

## <a name="the-account-is-locked-out"></a>La cuenta está bloqueada.

Una cuenta de usuario en un dominio administrado se bloquea cuando alcanza un umbral establecido para los intentos de inicio de sesión incorrectos. Este comportamiento está diseñado para protegerle en caso de que se intente iniciar sesión por fuerza bruta repetidamente, lo que podría ser un síntoma de un ataque digital automatizado.

De forma predeterminada, si hay cinco intentos de contraseña incorrectos en un plazo de 2 minutos, la cuenta se bloqueará durante 30 minutos.

Para más información sobre cómo resolver los problemas de bloqueo de cuentas, consulte [Solucionar los problemas de bloqueo de cuenta en Azure AD DS][troubleshoot-account-lockout].

## <a name="next-steps"></a>Pasos siguientes

Si sigue teniendo problemas para unir la máquina virtual al dominio administrado, [busque ayuda y abra una incidencia de soporte técnico para Azure Active Directory][azure-ad-support].

<!-- INTERNAL LINKS -->
[troubleshoot-account-lockout]: troubleshoot-account-lockout.md
[azure-ad-connect-phs]: ./tutorial-configure-password-hash-sync.md
[enable-user-accounts]:  tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds
[phs-process]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md#password-hash-sync-process-for-azure-ad-domain-services
[azure-ad-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md