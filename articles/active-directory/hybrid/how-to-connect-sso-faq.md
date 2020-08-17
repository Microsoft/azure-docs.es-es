---
title: 'Azure AD Connect: preguntas más frecuentes sobre el inicio de sesión único de conexión directa | Microsoft Docs'
description: Respuestas a las preguntas más frecuentes sobre el inicio de sesión único de conexión directa de Azure Active Directory.
services: active-directory
keywords: qué es Azure AD Connect, instalar Active Directory, componentes necesarios para Azure AD, SSO, inicio de sesión único
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/07/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: f621ed1342928b7f05fc8b84bfc2fceadf494fb5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87019738"
---
# <a name="azure-active-directory-seamless-single-sign-on-frequently-asked-questions"></a>Inicio de sesión único de conexión directa de Azure Active Directory: Preguntas más frecuentes

En este artículo se ofrece respuesta a las preguntas más frecuentes sobre el inicio de sesión único de conexión directa (SSO de conexión directa) de Azure Active Directory. Siga comprobando si hay contenido nuevo.

**P: ¿Con qué métodos de inicio de sesión funciona el inicio de sesión único de conexión directa?**

SSO de conexión directa se puede combinar con los métodos de inicio de sesión mediante [sincronización de hash de contraseñas](how-to-connect-password-hash-synchronization.md) o [autenticación de paso a través](how-to-connect-pta.md). Pero esta característica no se puede usar con los Servicios de federación de Active Directory (AD FS).

**P: ¿El inicio de sesión único de conexión directa es una característica gratuita?**

SSO de conexión directa es una característica gratuita y no es necesario usar ninguna versión de pago de Azure AD para usarla.

**P: ¿Está disponible el inicio de sesión único de conexión directa en la [nube de Microsoft Azure Alemania](https://www.microsoft.de/cloud-deutschland) y en la [nube de Microsoft Azure Government?](https://azure.microsoft.com/features/gov/)**

El SSO de conexión directa está disponible para la nube de Azure Government. Para más información, consulte [Consideraciones al respecto de la identidad híbrida para Azure Government](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-government-cloud).

**P: ¿Qué aplicaciones aprovechan la funcionalidad de parámetro `domain_hint` o `login_hint` del inicio de sesión único de conexión directa?**

La siguiente es una lista parcial de las aplicaciones que envían estos parámetros a Azure AD y, por lo tanto, proporciona a los usuarios una experiencia de inicio de sesión silenciosa con SSO de conexión directa (es decir, los usuarios no tendrán que especificar su nombre de usuario o contraseñas):

| Nombre de la aplicación | La dirección URL de la aplicación que se va a usar |
| -- | -- |
| Panel de acceso | https:\//myapps.microsoft.com/contoso.com |
| Outlook en la Web | https:\//outlook.office365.com/contoso.com |
| Portales de Office 365 | https:\//portal.office.com?domain_hint=contoso.com, https:\//www.office.com?domain_hint=contoso.com |

Además, no es necesaria la intervención del usuario para iniciar sesión cuando una aplicación envía solicitudes de inicio de sesión a puntos de conexión de Azure AD configurados como inquilinos; es decir, https:\//login.microsoftonline.com/contoso.com/<..> o https:\//login.microsoftonline.com/<tenant_ID>/<..> en lugar del punto de conexión común de Azure AD; es decir, https:\//login.microsoftonline.com/common/<...>. A continuación se incluye una lista no exhaustiva de aplicaciones que hacen estos tipos de solicitudes de inicio de sesión.

| Nombre de la aplicación | La dirección URL de la aplicación que se va a usar |
| -- | -- |
| SharePoint Online | https:\//contoso.sharepoint.com |
| Azure Portal | https:\//portal.azure.com/contoso.com |

En las tablas anteriores, reemplace "contoso.com" por el nombre de dominio para obtener las direcciones URL de aplicación correctas para el inquilino.

Si quiere que otras aplicaciones utilicen la experiencia de inicio de sesión silenciosa, comuníquenoslo en la sección de comentarios.

**P: ¿El inicio de sesión único de conexión directa admite `Alternate ID` como nombre de usuario en lugar de `userPrincipalName`?**

Sí. SSO de conexión directa admite `Alternate ID` como el nombre de usuario cuando se configura en Azure AD Connect, tal como se muestra [aquí](how-to-connect-install-custom.md). No todas las aplicaciones de Office 365 admiten `Alternate ID`. Consulte la documentación de la aplicación específica para conocer la declaración de compatibilidad.

**P: ¿Cuál es la diferencia entre la experiencia de inicio de sesión único que proporciona [Azure AD Join](../active-directory-azureadjoin-overview.md) y el inicio de sesión único de conexión directa?**

[Azure AD Join](../active-directory-azureadjoin-overview.md) permite el inicio de sesión único a los usuarios si sus dispositivos se han registrado con Azure AD. Estos dispositivos no deben estar unidos al dominio necesariamente. SSO se proporciona mediante *tokens de actualización principal* o *PRT*, y no con Kerberos. La experiencia del usuario es más adecuada en dispositivos Windows 10. SSO se realiza automáticamente en el explorador Microsoft Edge. También funciona en Chrome mediante una extensión de explorador.

Puede usar tanto Azure AD Join como SSO de conexión directa en el inquilino. Estas dos características son complementarias. Si ambas están activadas, SSO de Azure AD Join tiene prioridad sobre SSO de conexión directa.

**P: Quiero registrar dispositivos no de Windows 10 con Azure AD, sin usar AD FS. ¿Puedo usar el inicio de sesión único de conexión directa en su lugar?**

Sí, en este escenario se necesita la versión 2.1 o posterior del [cliente para unirse al área de trabajo](https://www.microsoft.com/download/details.aspx?id=53554).

**P: ¿Cómo puedo implementar la clave de descifrado de Kerberos de la cuenta de equipo `AZUREADSSO`?**

Es importante implementar con frecuencia la clave de descifrado de Kerberos de la cuenta de equipo `AZUREADSSO` (que representa a Azure AD) creada en el bosque local de AD.

>[!IMPORTANT]
>Se recomienda implementar la clave de descifrado de Kerberos al menos cada 30 días.

Siga estos pasos en el servidor local donde se ejecuta Azure AD Connect:

   **Paso 1. Obtención de la lista de bosques de AD en los que se habilitó SSO de conexión directa**

   1. Primero, descargue e instale [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/overview).
   2. Vaya a la carpeta `%programfiles%\Microsoft Azure Active Directory Connect`.
   3. Importe el módulo de PowerShell de SSO de conexión directa mediante este comando: `Import-Module .\AzureADSSO.psd1`.
   4. Ejecute PowerShell como administrador. En PowerShell, llame a `New-AzureADSSOAuthenticationContext`. Este comando debería mostrar un cuadro emergente para escribir las credenciales de administrador global del inquilino.
   5. Llame a `Get-AzureADSSOStatus | ConvertFrom-Json`. Aparecerá la lista de bosques de AD (examine la lista "Dominios") en la que se ha habilitado esta característica.

   **Paso 2. Actualización de la clave de descifrado de Kerberos en cada bosque de AD en el que se haya configurado**

   1. Llame a `$creds = Get-Credential`. Cuando se le pida, escriba las credenciales del administrador de dominio para el bosque de AD deseado.

   > [!NOTE]
   >El nombre de usuario de las credenciales de administrador de dominio debe especificarse con el formato de nombre de cuenta SAM (contoso\johndoe o contoso.com\johndoe). La parte de dominio del nombre de usuario se usa para localizar el controlador de dominio del administrador de dominio con DNS.

   >[!NOTE]
   >La cuenta de administrador de dominio usada no puede ser miembro del grupo Usuarios protegidos. De lo contrario, la operación presentará un error.

   2. Llame a `Update-AzureADSSOForest -OnPremCredentials $creds`. Este comando actualiza la clave de descifrado de Kerberos de la cuenta de equipo `AZUREADSSO` en este bosque de AD concreto y la actualiza en Azure AD.
   
   >[!NOTE]
   >Si no es administrador de dominio pero el administrador del dominio le ha asignado permisos, debe llamar a `Update-AzureADSSOForest -OnPremCredentials $creds -PreserveCustomPermissionsOnDesktopSsoAccount`
   
   3. Repita los pasos anteriores para cada bosque de AD en el que haya configurado la característica.

   >[!IMPORTANT]
   >Asegúrese de _no_ ejecutar el comando `Update-AzureADSSOForest` más de una vez. De lo contrario, la característica deja de funcionar hasta que expira la hora de los vales Kerberos de los usuarios y se vuelven a emitir por parte de la instancia local de Active Directory.

**P: ¿Cómo se deshabilita el inicio de sesión único de conexión directa?**

   **Paso 1. Deshabilitación de la función en el inquilino**

   **Opción A: deshabilitación mediante Azure AD Connect**
    
   1. Ejecute Azure AD Connect, elija **Change user sign-in page** (Cambiar página de inicio de sesión de usuario) y haga clic en **Siguiente**.
   2. Desactive la opción **Enable single sign on** (Habilitar el inicio de sesión único). Continúe con el asistente.

   Cuando haya finalizado con el asistente, el SSO de conexión directa estará deshabilitado en su inquilino. Sin embargo, verá un mensaje en pantalla en el que se anuncia lo siguiente:

   "El inicio de sesión único ya está deshabilitado, pero es necesario completar otros pasos manualmente para finalizar el proceso. Más información".

   Para completar el proceso de limpieza, siga los pasos 2 y 3 en el servidor local donde se ejecuta Azure AD Connect.

   **Opción B: deshabilitación mediante PowerShell**

   Ejecute los pasos siguientes en el servidor local donde se ejecuta Azure AD Connect:

   1. Primero, descargue e instale [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/overview).
   2. Vaya a la carpeta `%programfiles%\Microsoft Azure Active Directory Connect`.
   3. Importe el módulo de PowerShell de SSO de conexión directa mediante este comando: `Import-Module .\AzureADSSO.psd1`.
   4. Ejecute PowerShell como administrador. En PowerShell, llame a `New-AzureADSSOAuthenticationContext`. Este comando debería mostrar un cuadro emergente para escribir las credenciales de administrador global del inquilino.
   5. Llame a `Enable-AzureADSSO -Enable $false`.

   >[!IMPORTANT]
   >Si deshabilita SSO de conexión directa con PowerShell, no cambiará el estado en Azure AD Connect. SSO de conexión directa se mostrará como habilitado en la página **Change user sign-in** (Cambiar inicio de sesión de usuario).

   **Paso 2. Obtención de la lista de bosques de AD en los que se habilitó SSO de conexión directa**

   Realice las tareas 1 a 4 a continuación si ha deshabilitado el inicio de sesión único de conexión directa con Azure AD Connect. Si, en cambio, ha deshabilitado el inicio de sesión único de conexión directa con PowerShell, avance a la tarea 5 a continuación.

   1. Primero, descargue e instale [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/overview).
   2. Vaya a la carpeta `%programfiles%\Microsoft Azure Active Directory Connect`.
   3. Importe el módulo de PowerShell de SSO de conexión directa mediante este comando: `Import-Module .\AzureADSSO.psd1`.
   4. Ejecute PowerShell como administrador. En PowerShell, llame a `New-AzureADSSOAuthenticationContext`. Este comando debería mostrar un cuadro emergente para escribir las credenciales de administrador global del inquilino.
   5. Llame a `Get-AzureADSSOStatus | ConvertFrom-Json`. Aparecerá la lista de bosques de AD (examine la lista "Dominios") en la que se ha habilitado esta característica.

   **Paso 3. Elimine manualmente la cuenta de equipo `AZUREADSSO` de cada bosque de AD que encuentre en la lista.**

## <a name="next-steps"></a>Pasos siguientes

- [**Inicio rápido**](how-to-connect-sso-quick-start.md): desarrollo y ejecución de SSO de conexión directa de Azure AD.
- [**Profundización técnica**](how-to-connect-sso-how-it-works.md): descripción del funcionamiento de esta característica.
- [**Solución de problemas**](tshoot-connect-sso.md): información para resolver problemas habituales de esta característica.
- [**UserVoice**](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect): para la tramitación de solicitudes de nuevas características.
