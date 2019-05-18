---
title: 'Directivas: Azure Active Directory de restablecimiento de contraseña de autoservicio de AD Azure'
description: Opciones de directiva de autoservicio de restablecimiento de contraseña de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahenry
ms.collection: M365-identity-device-management
ms.openlocfilehash: f32952dff8f09db5b790818a5f98c527a04c2ef5
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823404"
---
# <a name="password-policies-and-restrictions-in-azure-active-directory"></a>Restricciones y directivas de contraseñas en Azure Active Directory

En este artículo se describen las directivas de contraseña y los requisitos de complejidad asociados a las cuentas de usuario del inquilino de Azure Active Directory (Azure AD).

## <a name="administrator-reset-policy-differences"></a>Diferencias entre directivas de restablecimiento de administrador

**Microsoft aplica una directiva de restablecimiento de contraseña de *dos puertas* predeterminada y segura para cualquier rol de administrador de Azure**: esta directiva puede ser diferente de la que haya definido para los usuarios y no se puede cambiar. Siempre debe probar la funcionalidad de restablecimiento de contraseña como usuario sin los roles de administrador de Azure asignados.

Con una directiva de dos puertas, los **administradores no tienen posibilidad de usar preguntas de seguridad**.

Una directiva de dos puertas requiere dos elementos de datos de autenticación, como una **dirección de correo electrónico**, una **aplicación de autenticador** o un **número de teléfono**. Se aplica una directiva de dos puertas en las siguientes circunstancias:

* Todos los roles de administrador siguientes se ven afectados:
  * Administrador del departamento de soporte técnico
  * Administrador del soporte técnico del servicio
  * Administrador de facturación
  * Soporte para asociados de nivel 1
  * Soporte para asociados de nivel 2
  * Administrador de Exchange
  * Administrador de Skype Empresarial
  * Administrador de usuarios
  * Escritores de directorios
  * Administrador global y administrador de la compañía
  * Administrador de SharePoint
  * Administrador de cumplimiento
  * Administrador de aplicaciones
  * Administrador de seguridad
  * Administrador de roles con privilegios
  * Administrador de Intune
  * Administrador del servicio de proxy de la aplicación
  * Administrador de Dynamics 365
  * Administrador de servicios de Power BI
  * Administrador de autenticación
  * Administrador de autenticación con privilegios

* Una vez transcurridos 30 días en una suscripción de prueba
* Con un dominio individualizado presente, como contoso.com
* Azure AD Connect sincroniza identidades desde el directorio local

### <a name="exceptions"></a>Excepciones

Una directiva de una puerta requiere un elemento de datos de autenticación, como una dirección de correo electrónico *o* un número de teléfono. Se aplica una directiva de una puerta en las siguientes circunstancias:

* Se encuentra en los primeros 30 días de una suscripción de prueba; o
* No hay un dominio individualizado presente (*.onmicrosoft.com); y
* Azure AD Connect no sincroniza identidades

## <a name="userprincipalname-policies-that-apply-to-all-user-accounts"></a>Directivas de UserPrincipalName que se aplican a todas las cuentas de usuario

Cada cuenta de usuario que necesita iniciar sesión en Azure AD debe tener un valor de atributo de nombre principal de usuario (UPN) único asociado a esa cuenta. En la siguiente tabla se describen las directivas que se aplican tanto a cuentas de usuario de Active Directory locales (sincronizadas con la nube) como a cuentas de usuario solo en la nube:

| Propiedad | Requisitos de UserPrincipalName |
| --- | --- |
| Caracteres permitidos |<ul> <li>A – Z</li> <li>a - z</li><li>0 – 9</li> <li> ' \. - \_ ! \# ^ \~</li></ul> |
| Caracteres no permitidos |<ul> <li>Cualquier carácter "\@\" que no separa el nombre de usuario del dominio.</li> <li>No puede contener un carácter de punto "." inmediatamente antes del símbolo "\@\".</li></ul> |
| Restricciones de longitud |<ul> <li>La longitud total no debe superar los 113 caracteres.</li><li>Puede haber hasta 64 caracteres antes del símbolo "\@\"</li><li>Puede haber hasta 48 caracteres después del símbolo "\@\"</li></ul> |

## <a name="password-policies-that-only-apply-to-cloud-user-accounts"></a>Directivas de contraseña que solo se aplican a cuentas de usuario en la nube

En la tabla siguiente se describe la configuración de directiva de contraseña que se aplican a las cuentas de usuario que se crean y administran en Azure AD:

| Propiedad | Requisitos |
| --- | --- |
| Caracteres permitidos |<ul><li>A – Z</li><li>a - z</li><li>0 – 9</li> <li>@ # $ % ^ & * - _ ! + = [ ] { } &#124; \ : ‘ , . ? / \` ~ " ( ) ;</li> <li>espacio en blanco</li></ul> |
| Caracteres no permitidos |<ul><li>Caracteres Unicode.</li><li> No puede contener un carácter de punto "." inmediatamente anterior a la "\@ \" símbolos".</li></ul> |
| Restricciones de contraseña |<ul><li>Un mínimo de 8 caracteres y un máximo de 256 caracteres.</li><li>requiere al menos tres de los cuatro requisitos siguientes:<ul><li>Caracteres en minúsculas.</li><li>Caracteres en mayúsculas.</li><li>Números (0-9).</li><li>Símbolos (vea las anteriores restricciones de contraseña).</li></ul></li></ul> |
| Duración de las contraseñas |<ul><li>Valor predeterminado: **90** días.</li><li>El valor se puede configurar con el cmdlet `Set-MsolPasswordPolicy`del módulo Active Directory para Windows PowerShell.</li></ul> |
| Notificación de la expiración de contraseñas |<ul><li>Valor predeterminado: **14** días (antes de que expire la contraseña).</li><li>El valor se puede configurar con el cmdlet `Set-MsolPasswordPolicy`.</li></ul> |
| Expiración de las contraseñas |<ul><li>Valor predeterminado: **false** días (indica que la expiración de la contraseña está habilitada).</li><li>El valor se puede configurar para cuentas de usuario individuales mediante el cmdlet `Set-MsolUser`.</li></ul> |
| Historial de cambios de contraseña |La última contraseña *no* puede usarse de nuevo cuando el usuario cambia una contraseña. |
| Historial de restablecimientos de contraseña | La última contraseña *puede* usarse de nuevo cuando el usuario restablece una contraseña olvidada. |
| Bloqueo de cuenta |Después de 10 intentos de inicio de sesión incorrectos con una contraseña incorrecta, el usuario se bloquea durante un minuto. Más intentos de inicio de sesión incorrectos bloquean el usuario durante mayor cantidad de tiempo. El [bloqueo inteligente](howto-password-smart-lockout.md) realiza un seguimiento de los últimos tres códigos hash de contraseña incorrecta para evitar que aumente el contador de bloqueo con la misma contraseña. Si alguien escribe la misma contraseña incorrecta varias veces, este comportamiento no hará que la cuenta se bloquee. |

## <a name="set-password-expiration-policies-in-azure-ad"></a>Establecer directivas de expiración de contraseñas en Azure AD

Un administrador global o administrador de usuario para un servicio de nube de Microsoft puede usar el módulo Microsoft Azure AD para Windows PowerShell para establecer las contraseñas de usuario no caducan. También puede usar cmdlets de Windows PowerShell para quitar la configuración de no expirar nunca o ver qué contraseñas de usuario están configuradas para que no expiren nunca. 

Esta guía se aplica a otros proveedores (como Intune y Office 365) que también dependen de Azure AD para los servicios de identidad y directorio. La expiración de contraseñas es la única parte de la directiva que se puede cambiar.

> [!NOTE]
> Solo las contraseñas de cuentas de usuario que no están sincronizadas a través de la sincronización de directorios pueden configurarse para que no caduquen. Para obtener más información sobre la sincronización de directorios, vea el artículo sobre cómo [conectar AD con Azure AD](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect).

## <a name="set-or-check-the-password-policies-by-using-powershell"></a>Establecer o comprobar directivas de contraseña mediante PowerShell

Para empezar, debe [descargar e instalar el módulo de Azure AD PowerShell](https://docs.microsoft.com/powershell/module/Azuread/?view=azureadps-2.0). Una vez instalado, puede realizar los pasos siguientes para configurar cada campo.

### <a name="check-the-expiration-policy-for-a-password"></a>Comprobación de la directiva de expiración de una contraseña

1. Conectarse a Windows PowerShell mediante el Administrador de usuario o las credenciales de administrador de empresa.
1. Ejecute uno de los siguientes comandos:

   * Para ver si se establece la contraseña de un usuario único que no expire nunca, ejecute el siguiente cmdlet con el UPN (por ejemplo, *aprilr\@contoso.onmicrosoft.com*) o el identificador de usuario del usuario que desea comprobar:

   ```powershell
   Get-AzureADUser -ObjectId <user ID> | Select-Object @{N="PasswordNeverExpires";E={$_.PasswordPolicies -contains "DisablePasswordExpiration"}}
   ```

   * Para ver el **contraseña nunca expira** establecer para todos los usuarios, ejecute el siguiente cmdlet:

   ```powershell
   Get-AzureADUser -All $true | Select-Object UserPrincipalName, @{N="PasswordNeverExpires";E={$_.PasswordPolicies -contains "DisablePasswordExpiration"}}
   ```

### <a name="set-a-password-to-expire"></a>Configuración de una contraseña para que caduque

1. Conectarse a Windows PowerShell mediante el Administrador de usuario o las credenciales de administrador de empresa.
1. Ejecute uno de los siguientes comandos:

   * Para establecer la contraseña de un usuario para que la contraseña caduque, ejecute el siguiente cmdlet con el UPN o el identificador de usuario del usuario:

   ```powershell
   Set-AzureADUser -ObjectId <user ID> -PasswordPolicies None
   ```

   * Para establecer las contraseñas de todos los usuarios de la organización de modo que caduquen, use el siguiente cmdlet:

   ```powershell
   Get-AzureADUser -All $true | Set-AzureADUser -PasswordPolicies None
   ```

### <a name="set-a-password-to-never-expire"></a>Configure una contraseña para que no caduque nunca

1. Conectarse a Windows PowerShell mediante el Administrador de usuario o las credenciales de administrador de empresa.
1. Ejecute uno de los siguientes comandos:

   * Para establecer la contraseña de un usuario que no expire nunca, ejecute el siguiente cmdlet con el UPN o el identificador de usuario del usuario:

   ```powershell
   Set-AzureADUser -ObjectId <user ID> -PasswordPolicies DisablePasswordExpiration
   ```

   * Para establecer las contraseñas de todos los usuarios de una organización que no expire nunca, ejecute el siguiente cmdlet:

   ```powershell
   Get-AzureADUser -All $true | Set-AzureADUser -PasswordPolicies DisablePasswordExpiration
   ```

   > [!WARNING]
   > La antigüedad de las contraseñas establecidas en `-PasswordPolicies DisablePasswordExpiration` sigue basándose en el atributo `pwdLastSet`. Si establece las contraseñas de usuario para que no expiren nunca y pasan más de 90 días, las contraseñas expiran. En función del `pwdLastSet` atributo, si cambia la expiración a `-PasswordPolicies None`, todas las contraseñas que tengan una `pwdLastSet` cuya antigüedad sea superior a 90 días requieren que el usuario las cambie la próxima vez que se inicie sesión. Este cambio puede afectar a gran cantidad de usuarios.

## <a name="next-steps"></a>Pasos siguientes

En los siguientes artículos se proporciona información adicional sobre el restablecimiento de contraseña con Azure AD:

* [¿Cómo se realiza un lanzamiento correcto de SSPR?](howto-sspr-deployment.md)
* [Restablecimiento o modificación de la contraseña](../user-help/active-directory-passwords-update-your-own-password.md).
* [Registro para el autoservicio de restablecimiento de contraseñas](../user-help/active-directory-passwords-reset-register.md).
* [¿Tiene alguna pregunta acerca de las licencias?](concept-sspr-licensing.md)
* [¿Qué datos usa SSPR y cuáles se deben rellenar en lugar de los usuarios?](howto-sspr-authenticationdata.md)
* [¿Qué métodos de autenticación están disponibles para los usuarios?](concept-sspr-howitworks.md#authentication-methods)
* [¿Qué es la escritura diferida de contraseñas y por qué nos interesa?](howto-sspr-writeback.md)
* [¿Cómo se informa sobre la actividad de SSPR?](howto-sspr-reporting.md)
* [¿Cuáles son todas las opciones en SSPR y qué significan?](concept-sspr-howitworks.md)
* [Creo que algo se ha roto. ¿Cómo se solucionan problemas en SSPR?](active-directory-passwords-troubleshoot.md)
* [Tengo una pregunta que no se ha comentado en ningún otro sitio](active-directory-passwords-faq.md)
