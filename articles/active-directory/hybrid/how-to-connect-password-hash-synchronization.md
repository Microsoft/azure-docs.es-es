---
title: Implementación de la sincronización de hash de contraseñas mediante la sincronización de Azure AD Connect | Microsoft Docs
description: Ofrece información sobre cómo funciona la sincronización de hash de contraseñas y cómo configurarla.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 05f16c3e-9d23-45dc-afca-3d0fa9dbf501
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/26/2020
ms.subservice: hybrid
ms.author: billmath
search.appverid:
- MET150
ms.collection: M365-identity-device-management
ms.openlocfilehash: c16882f35c9ca79644cd2b51ce4cd88bba516ed2
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89652074"
---
# <a name="implement-password-hash-synchronization-with-azure-ad-connect-sync"></a>Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect
En este artículo se ofrece información que se necesita para sincronizar las contraseñas de usuario desde una instancia de Active Directory local con otra de Azure Active Directory (Azure AD) basado en la nube.

## <a name="how-password-hash-synchronization-works"></a>Funcionamiento de la sincronización de hash de contraseñas
El servicio de dominio de Active Directory almacena contraseñas en forma de representación de valor hash de la contraseña de usuario real. Un valor hash es el resultado de una función matemática unidireccional (el *algoritmo hash*). No hay ningún método para volver del resultado de una función unidireccional a la versión de texto sin formato de una contraseña. 

Para sincronizar la contraseña, la sincronización de Azure AD Connect extrae el hash de la contraseña de la instancia de Active Directory local. Se aplica un procesamiento de seguridad adicional al hash de contraseña antes de que se sincronice con el servicio de autenticación de Azure Active Directory. Las contraseñas se sincronizan usuario por usuario y en orden cronológico.

El flujo de datos reales del proceso de sincronización de hash de contraseñas es similar al de sincronización de datos de usuario. De todas formas, las contraseñas se sincronizan con más frecuencia que la ventana de sincronización de directorios estándar para otros atributos. El proceso de sincronización de hash de contraseñas se ejecuta cada 2 minutos. La frecuencia de este proceso no se puede modificar. Al sincronizar una contraseña, esta sobrescribe la contraseña existente en la nube.

La primera vez que se habilita la característica de sincronización de hash de contraseñas, se realiza una sincronización inicial de las contraseñas de todos los usuarios dentro del ámbito. No puede definir explícitamente un subconjunto de contraseñas de usuario que quiera sincronizar. Pero si hay varios conectores, es posible deshabilitar la sincronización de hash de contraseña en algunos conectores, pero no en otros, mediante el cmdlet [Set-ADSyncAADPasswordSyncConfiguration](../../active-directory-domain-services/tutorial-configure-password-hash-sync.md).

Al cambiar una contraseña local, la contraseña actualizada se sincroniza más a menudo en cuestión de minutos.
La característica de sincronización de hash de contraseñas reintenta automáticamente los intentos fallidos de sincronización. Si se produce un error al intentar sincronizar una contraseña, se registra un error en el visor de eventos.

La sincronización de una contraseña no influye en el usuario que actualmente ha iniciado sesión.
Si se sincroniza un cambio de contraseña mientras tiene iniciada sesión en un servicio en la nube, este cambio no afectará de inmediato a la sesión actual del servicio en la nube. Sin embargo, cuando el servicio en la nube requiera de nuevo su autenticación, deberá proporcionar la nueva contraseña.

Un usuario debe escribir sus credenciales corporativas una segunda vez para autenticarse en Azure AD, independientemente de que haya iniciado sesión en su red corporativa. Pero este patrón puede reducirse si el usuario activa la casilla "Mantener la sesión iniciada" (KMSI) en el inicio de sesión. Esta opción establece una cookie de sesión que omite la autenticación durante 180 días. El comportamiento de KMSI lo puede habilitar o deshabilitar el administrador de Azure AD. Además, para reducir los mensajes de petición de contraseña, puede activar [SSO de conexión directa](how-to-connect-sso.md), que permite iniciar sesión automáticamente a los usuarios en dispositivos corporativos conectados a la red de la empresa.

> [!NOTE]
> Solo se admite la sincronización de la contraseña para el usuario del tipo de objeto de Active Directory. No se admite para el tipo de objeto iNetOrgPerson.

### <a name="detailed-description-of-how-password-hash-synchronization-works"></a>Descripción detallada de cómo funciona la sincronización de hash de contraseñas

En la sección siguiente se describe con detalle cómo funciona la sincronización de hash de contraseñas entre Active Directory y Azure AD.

![Flujo detallado de contraseñas](./media/how-to-connect-password-hash-synchronization/arch3b.png)

1. Cada dos minutos, el agente de sincronización de hash de contraseñas en el servidor de AD Connect solicita hashes de contraseña almacenados (el atributo unicodePwd) desde un controlador de dominio.  Esta solicitud es a través del protocolo de replicación [MS-DRSR](/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) estándar que se utiliza para sincronizar datos entre controladores de dominio. La cuenta de servicio debe tener los permisos para replicar cambios de directorio y para replicar cambios de directorio en todos los AD (se conceden de manera predeterminada en la instalación) para obtener los hashes de contraseña.
2. Antes del envío, el controlador de dominio cifra el hash de contraseña MD4 mediante una clave que es un hash de [MD5](https://www.rfc-editor.org/rfc/rfc1321.txt) de la clave de sesión RPC y valor sal. Después envía el resultado al agente de sincronización de hash de contraseñas a través de RPC. El controlador de dominio también pasa el valor sal al agente de sincronización mediante el protocolo de replicación del controlador de dominio, por lo que el agente podrá descifrar el sobre.
3. Cuando el agente de sincronización de hash de contraseñas tiene el sobre cifrado, usa [MD5CryptoServiceProvider](/dotnet/api/system.security.cryptography.md5cryptoserviceprovider?view=netcore-3.1) y el valor sal para generar una clave para descifrar los datos recibidos en su formato original de MD4. El agente de sincronización de hash de contraseñas nunca tiene acceso a la contraseña de texto no cifrado. El uso del agente de sincronización de hash de contraseñas de MD5 es estrictamente para compatibilidad del protocolo de replicación con el controlador de dominio y solo se usa en local entre el controlador de dominio y el agente de sincronización de hash de contraseñas.
4. El agente de sincronización de hash de contraseñas amplía el hash de contraseña binario de 16 bits a 64 bytes al convertir en primer lugar el hash con una cadena hexadecimal de 32 bytes y, después, convertir esta cadena de nuevo en binario con codificación UTF-16.
5. El agente de sincronización de hash de contraseñas agrega un valor sal por usuario, que consta de un valor sal de longitud de 10 bytes, al archivo binario de 64 bits para proteger aún más el valor hash original.
6. El agente de sincronización de hash de contraseñas combina el hash MD4 con el valor sal por usuario y los introduce en la función [PBKDF2](https://www.ietf.org/rfc/rfc2898.txt). Se usan 1000 iteraciones del algoritmo hash con clave [HMAC-SHA256](/dotnet/api/system.security.cryptography.hmacsha256?view=netcore-3.1). 
7. El agente de sincronización de hash de contraseñas toma el hash de 32 bits resultante, concatena el valor sal por usuario y el número de iteraciones de SHA256 con él (para su uso con Azure AD) y después transmite la cadena desde Azure AD Connect a Azure AD a través de TLS.</br> 
8. Cuando un usuario intenta iniciar sesión en Azure AD y escribe su contraseña, esta se ejecuta a través del mismo proceso MD4 + sal + PBKDF2 + HMAC-SHA256. Si el hash resultante coincide con el valor hash almacenado en Azure AD, el usuario ha especificado la contraseña correcta y se autentica.

> [!NOTE]
> El hash MD4 original no se transmite a Azure AD. En su lugar, se transmite el hash SHA256 del algoritmo hash MD4 original. Por consiguiente, si se obtiene el hash almacenado en Azure AD, no se puede usar en un ataque pass-the-hash en local.

### <a name="security-considerations"></a>Consideraciones sobre la seguridad

Al sincronizar contraseñas, la versión de texto sin formato de su contraseña no se expone a la característica de sincronización de hash de contraseñas ni a Azure AD ni a ninguno de los servicios asociados.

La autenticación del usuario tiene lugar en Azure AD, y no en la propia instancia de Active Directory de la organización. Los datos de la contraseña SHA256 almacenados en Azure AD, un hash del hash MD4 original, son más seguros que los que se almacenan en Active Directory. Además, como no se puede descifrar este hash SHA256, no se vuelve al entorno de Active Directory de la organización y se presenta como una contraseña de usuario válida en un ataque pass-the-hash.

### <a name="password-policy-considerations"></a>Consideraciones de la directiva de contraseña

Existen dos tipos de directivas de contraseña que se ven afectados por la habilitación de la sincronización de hash de contraseñas:

* Directiva de complejidad de contraseñas
* Directiva de expiración de contraseñas

#### <a name="password-complexity-policy"></a>Directiva de complejidad de contraseñas

Cuando se habilita la sincronización de hash de contraseñas, las directivas de complejidad de contraseñas en su instancia local de Active Directory reemplazan a las directivas de complejidad en la nube para los usuarios sincronizados. Puede usar todas las contraseñas válidas de su instancia de Active Directory local para acceder a servicios de Azure AD.

> [!NOTE]
> Las contraseñas para los usuarios que se crean directamente en la nube siguen estando sujetas a las directivas de contraseñas tal como se definen en la nube.

#### <a name="password-expiration-policy"></a>Directiva de expiración de contraseñas

Si un usuario está en el ámbito de sincronización de hash de contraseñas, de forma predeterminada la contraseña de cuenta en la nube se establece en *No caduca nunca*.

Puede seguir iniciando sesión en los servicios en la nube con una contraseña sincronizada que haya expirado en su entorno local. La contraseña en la nube se actualizará la próxima vez que cambie la contraseña en el entorno local.

##### <a name="enforcecloudpasswordpolicyforpasswordsyncedusers"></a>EnforceCloudPasswordPolicyForPasswordSyncedUsers

Si hay usuarios sincronizados que solo interactúan con servicios integrados de Azure AD y también deben cumplir una directiva de expiración de contraseñas, puede obligarlos a cumplir con la directiva de expiración de contraseñas de Azure AD mediante la habilitación de la característica *EnforceCloudPasswordPolicyForPasswordSyncedUsers*.

Cuando *EnforceCloudPasswordPolicyForPasswordSyncedUsers* está deshabilitado (que es la configuración predeterminada), Azure AD Connect establece el atributo PasswordPolicies de los usuarios sincronizados en "DisablePasswordExpiration". Esto se hace cada vez que se sincroniza la contraseña de un usuario e indica a Azure AD que omita la directiva de expiración de contraseñas en la nube para ese usuario. Puede comprobar el valor del atributo mediante el módulo de Azure AD PowerShell con el siguiente comando:

`(Get-AzureADUser -objectID <User Object ID>).passwordpolicies`

Para habilitar la característica EnforceCloudPasswordPolicyForPasswordSyncedUsers, ejecute el siguiente comando con el módulo MSOnline de PowerShell, tal como se muestra a continuación. Tendrá que escribir yes en el parámetro Enable, como se muestra a continuación:

```
Set-MsolDirSyncFeature -Feature EnforceCloudPasswordPolicyForPasswordSyncedUsers
cmdlet Set-MsolDirSyncFeature at command pipeline position 1
Supply values for the following parameters:
Enable: yes
Confirm
Continue with this operation?
[Y] Yes [N] No [S] Suspend [?] Help (default is "Y"): y
```

Una vez habilitado, Azure AD no va a cada usuario sincronizado para quitar el valor `DisablePasswordExpiration` del atributo PasswordPolicies. En su lugar, el valor se establece en `None` durante la siguiente sincronización de contraseñas para cada usuario la próxima vez que cambie su contraseña en AD local.  

Se recomienda habilitar EnforceCloudPasswordPolicyForPasswordSyncedUsers, antes de habilitar la sincronización de hash de contraseñas, para que la sincronización inicial de los hashes de contraseñas no agregue el valor `DisablePasswordExpiration` al atributo PasswordPolicies para los usuarios.

La directiva de contraseñas de Azure AD predeterminada requiere que los usuarios cambien sus contraseñas cada 90 días. Si la directiva de AD es también de 90 días, las dos directivas deben coincidir. Sin embargo, si la directiva de AD no es de 90 días, puede actualizar la directiva de contraseñas de Azure AD para que coincida con el comando de PowerShell Set-MsolPasswordPolicy.

Azure AD admite una directiva de expiración de contraseñas independiente por dominio registrado.

Advertencia: Si hay cuentas sincronizadas que necesiten contraseñas que no expiren en Azure AD, debe agregar explícitamente el valor `DisablePasswordExpiration` al atributo PasswordPolicies del objeto de usuario en Azure AD.  Para ello, puede ejecutar el siguiente comando.

`Set-AzureADUser -ObjectID <User Object ID> -PasswordPolicies "DisablePasswordExpiration"`

> [!NOTE]
> El comando de PowerShell Set-MsolPasswordPolicy no funcionará en dominios federados. 

#### <a name="synchronizing-temporary-passwords-and-force-password-change-on-next-logon"></a>Sincronización de contraseñas temporales y "Forzar cambio de contraseña en el siguiente inicio de sesión"

Es habitual exigir al usuario a cambiar la contraseña durante el primer inicio de sesión, especialmente después de que se produzca un restablecimiento de la contraseña de administrador.  Normalmente se conoce como establecer una contraseña "temporal" y se completa mediante la comprobación de la marca "El usuario debe cambiar la contraseña en el siguiente inicio de sesión" en un objeto de usuario de Active Directory (AD).
  
La funcionalidad de contraseña temporal ayuda a garantizar que la transferencia de propiedad de la credencial se complete al usarse por primera vez, con el fin de minimizar la duración del tiempo en el que más de un usuario tiene conocimiento de esa credencial.

Para admitir contraseñas temporales en Azure AD para usuarios sincronizados, puede habilitar la característica *ForcePasswordChangeOnLogOn* mediante la ejecución del siguiente comando en el servidor de Azure AD Connect:

`Set-ADSyncAADCompanyFeature -ForcePasswordChangeOnLogOn $true`

> [!NOTE]
> Forzar a un usuario a cambiar su contraseña en el siguiente inicio de sesión requiere un cambio de contraseña al mismo tiempo.  Azure AD Connect no recoge la marca de forzar el cambio de contraseña por sí misma; es complementaria al cambio de contraseña detectado que se produce durante la sincronización de hash de contraseñas.

> [!CAUTION]
> Solo debe usar esta característica cuando SSPR y la escritura diferida de contraseñas estén habilitados en el inquilino.  De este modo, si un usuario cambia la contraseña a través de SSPR, se sincroniza con Active Directory.

#### <a name="account-expiration"></a>Expiración de la cuenta

Si su organización usa el atributo accountExpires como parte de la administración de cuentas de usuario, este atributo no se sincroniza con Azure AD. Por consiguiente, una cuenta de Active Directory expirada en un entorno configurado para la sincronización de hash de contraseñas seguirá activa en Azure AD. Se recomienda que, si la cuenta ha expirado, una acción de flujo de trabajo debe desencadenar un script de PowerShell que deshabilite la cuenta de Azure AD del usuario (use el cmdlet [Set-AzureADUser](/powershell/module/azuread/set-azureaduser?view=azureadps-2.0)). Por el contrario, cuando la cuenta está activada, la instancia de Azure AD debe estar activada.

### <a name="overwrite-synchronized-passwords"></a>Sobrescritura de las contraseñas sincronizadas

Un administrador puede restablecer manualmente la contraseña mediante Windows PowerShell.

En este caso, la nueva contraseña reemplaza a la contraseña sincronizada y todas las directivas de contraseñas definidas en la nube se aplican a la nueva contraseña.

Si vuelve a cambiar la contraseña local, la nueva contraseña se sincroniza con la nube y reemplaza a la contraseña actualizada manualmente.

La sincronización de una contraseña no influye en el usuario de Azure que ha iniciado sesión. Si se sincroniza un cambio de contraseña mientras tiene iniciada sesión en un servicio en la nube, este cambio no afectará de inmediato a la sesión actual del servicio en la nube. KMSI prolonga la duración de esta diferencia. Cuando el servicio en la nube requiera de nuevo su autenticación, deberá proporcionar la nueva contraseña.

### <a name="additional-advantages"></a>Ventajas adicionales

- Por lo general, la sincronización de hash de contraseñas es más fácil de implementar que un servicio de federación. No requiere ningún servidor adicional y elimina la dependencia en un servicio de federación de alta disponibilidad para autenticar a los usuarios.
- También se puede habilitar la sincronización de hash de contraseñas además de la federación. Puede usarse como reserva en caso de que el servicio de federación experimente una interrupción del servicio.

## <a name="password-hash-sync-process-for-azure-ad-domain-services"></a>Proceso de sincronización de hash de contraseña para Azure AD Domain Services

Si usa Azure AD Domain Services para proporcionar autenticación heredada para las aplicaciones y los servicios que necesitan usar Keberos, LDAP o NTLM, algunos procesos adicionales forman parte del flujo de sincronización de hash de contraseña. Azure AD Connect usa el siguiente proceso adicional para sincronizar los hash de contraseña con Azure AD para su uso en Azure AD Domain Services:

> [!IMPORTANT]
> Azure AD Connect solo debe instalarse y configurarse para la sincronización con entornos de AD DS locales. No se admite la instalación de Azure AD Connect en un dominio administrado de Azure AD DS para volver a sincronizar los objetos con Azure AD.
>
> Azure AD Connect solo sincroniza los hashes de contraseña heredados al habilitar Azure AD DS para su inquilino de Azure AD. Los siguientes pasos no se usan si solo utiliza Azure AD Connect para sincronizar un entorno de AD DS local con Azure AD.
>
> Si las aplicaciones heredadas no utilizan la autenticación NTLM o los enlaces simples LDAP, se recomienda deshabilitar la sincronización de hash de contraseña de NTLM para Azure AD DS. Para más información, consulte [Deshabilitación de la sincronización de hash de credenciales NTLM y de conjuntos de cifrado débil](../../active-directory-domain-services/secure-your-domain.md).

1. Azure AD Connect recupera la clave pública de la instancia del inquilino de Azure AD Domain Services.
1. Cuando un usuario cambia la contraseña, el controlador de dominio local almacena el resultado del cambio de contraseña (hashes) en dos atributos:
    * *unicodePwd* para el hash de contraseña de NTLM.
    * *supplementalCredentials* para el hash de contraseña de Kerberos.
1. Azure AD Connect detecta los cambios de contraseña a través del canal de replicación de directorios (los cambios de atributo deben replicarse en otros controladores de dominio).
1. Para cada usuario cuya contraseña ha cambiado, Azure AD Connect realiza los pasos siguientes:
    * Genera una clave simétrica AES de 256 bits aleatoria.
    * Genera un vector de inicialización aleatorio necesario para la primera ronda de cifrado.
    * Extrae los valores hash de contraseña de Kerberos de los atributos *supplementalCredentials*.
    * Comprueba el valor *SyncNtlmPasswords* de configuración de seguridad de Azure AD Domain Services.
        * Si esta configuración está deshabilitada, genera un hash de NTLM aleatorio y de alta entropía (diferente de la contraseña del usuario). Después, este hash se combina con los hashes de contraseña de Kerberos extraídos del atributo *supplementalCrendetials* en una estructura de datos.
        * Si está habilitada, combina el valor del atributo *unicodePwd* con los hash de contraseña de Kerberos extraídos del atributo *supplementalCredentials* en una estructura de datos.
    * Cifra la estructura de datos única mediante la clave simétrica AES.
    * Cifra la clave simétrica AES mediante la clave pública de Azure AD Domain Services del inquilino.
1. Azure AD Connect transmite la clave simétrica AES cifrada, la estructura de datos cifrada que contiene los valores hash de contraseña y el vector de inicialización que se va a Azure AD.
1. Azure AD almacena la clave simétrica AES cifrada, la estructura de datos cifrada y el vector de inicialización para el usuario.
1. Azure AD envía la clave simétrica AES cifrada, la estructura de datos cifrada y el vector de inicialización mediante un mecanismo de sincronización interno a través de una sesión HTTP cifrada para Azure AD Domain Services.
1. Azure AD Domain Services recupera la clave privada de la instancia del inquilino desde Azure Key Vault.
1. Para cada conjunto de datos cifrado (que representa el cambio de contraseña de un solo usuario), Azure AD Domain Services realiza los siguientes pasos:
    * Usa su clave privada para descifrar la clave simétrica AES.
    * Usa la clave simétrica AES con el vector de inicialización para descifrar la estructura de datos cifrados que contiene los hash de contraseña.
    * Escribe los hash de contraseña de Kerberos que recibe en el controlador de dominio de Azure AD Domain Services. Los hashes se guardan en el atributo *supplementalCredentials* del objeto de usuario que se cifra con la clave pública del controlador de dominio de Azure AD Domain Services.
    * Azure AD Domain Services escribe el hash de contraseña de NTLM que se recibe en el controlador de dominio de Azure AD Domain Services. El hash se guarda en el atributo *unicodePwd* del objeto de usuario que se cifra con la clave pública del controlador de dominio de Azure AD Domain Services.

## <a name="enable-password-hash-synchronization"></a>Habilitación de la sincronización de hash de contraseñas

>[!IMPORTANT]
>Si va a migrar desde AD FS (u otra tecnología de federación) a la sincronización de hash de contraseña, es muy recomendable que siga nuestra guía de implementación detallada publicada [aquí](https://aka.ms/adfstophsdpdownload).

Cuando se instala Azure AD Connect mediante la opción **Configuración rápida**, la sincronización de hash de contraseñas se habilita automáticamente. Para más información, vea [Introducción a Azure AD Connect mediante la configuración rápida](how-to-connect-install-express.md).

Si usa una configuración personalizada al instalar Azure AD Connect, la sincronización de hash de contraseñas está disponible en la página de inicio de sesión del usuario. Para más información, consulte [Instalación personalizada de Azure AD Connect](how-to-connect-install-custom.md).

![Habilitación de la sincronización de hash de contraseñas](./media/how-to-connect-password-hash-synchronization/usersignin2.png)

### <a name="password-hash-synchronization-and-fips"></a>Sincronización de hash de contraseñas y FIPS
Si el servidor se bloquea según el Estándar Federal de Procesamiento de la Información (FIPS), se deshabilita el hash MD5.

**Para habilitar el MD5 para la sincronización de hash de contraseñas, realice los pasos siguientes:**

1. Vaya a %programfiles%\Azure AD Sync\Bin.
2. Abra miiserver.exe.config.
3. Vaya al nodo configuration/runtime (al final del archivo).
4. Agregue el siguiente nodo: `<enforceFIPSPolicy enabled="false"/>`
5. Guarde los cambios.

Como referencia, este fragmento de código debe ser similar al siguiente:

```
    <configuration>
        <runtime>
            <enforceFIPSPolicy enabled="false"/>
        </runtime>
    </configuration>
```

Para obtener información sobre la seguridad y FIPS, vea [Azure AD password hash sync, encryption, and FIPS compliance](https://blogs.technet.microsoft.com/enterprisemobility/2014/06/28/aad-password-sync-encryption-and-fips-compliance/) (Cumplimiento de sincronización de hash de contraseña, cifrado y FIPS de Azure AD).

## <a name="troubleshoot-password-hash-synchronization"></a>Solución de problemas de sincronización de hash de contraseñas
Si tiene problemas con la sincronización de hash de contraseñas, vea [Solución de problemas de sincronización de hash de contraseñas](tshoot-connect-password-hash-synchronization.md).

## <a name="next-steps"></a>Pasos siguientes
* [Sincronización de Azure AD Connect: personalización de las opciones de sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)
* [Obtención de un plan de implementación paso a paso para migrar desde AD FS para la sincronización de hash de contraseña](https://aka.ms/authenticationDeploymentPlan)
