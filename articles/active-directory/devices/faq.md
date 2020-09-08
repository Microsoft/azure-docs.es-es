---
title: Preguntas más frecuentes sobre la administración de dispositivos de Azure Active Directory | Microsoft Docs
description: Preguntas más frecuentes sobre la administración de dispositivos de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 06/28/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: ravenn
ms.collection: M365-identity-device-management
ms.openlocfilehash: c8d03b8f13a016dc21e37b82f66abc8050ef17d8
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266789"
---
# <a name="azure-active-directory-device-management-faq"></a>Preguntas más frecuentes sobre la administración de dispositivos de Azure Active Directory

## <a name="general-faq"></a>Preguntas más frecuentes generales

### <a name="q-i-registered-the-device-recently-why-cant-i-see-the-device-under-my-user-info-in-the-azure-portal-or-why-is-the-device-owner-marked-as-na-for-hybrid-azure-active-directory-azure-ad-joined-devices"></a>P: He registrado el dispositivo hace poco. ¿Por qué no puedo ver el dispositivo en mi información del usuario de Azure Portal? O bien, ¿por qué el propietario del dispositivo está marcado como N/D para los dispositivos unidos a Azure Active Directory (Azure AD) híbrido?

**R:** los dispositivos Windows 10 que están unidos a Azure AD híbrido no aparecen bajo los **dispositivos del USUARIO**.
Use la vista **Todos los dispositivos** de Azure Portal. También puede usar un cmdlet [Get-MsolDevice](/powershell/module/msonline/get-msoldevice?view=azureadps-1.0) de PowerShell.

Los únicos que aparecen entre los **dispositivos del USUARIO** son los siguientes:

- Todos los dispositivos personales que no están unidos a Azure AD híbrido. 
- Todos los dispositivos que no tienen Windows 10 o Windows Server 2016.
- Todos los dispositivos que no son de Windows. 

---

### <a name="q-how-do-i-know-what-the-device-registration-state-of-the-client-is"></a>P: ¿Cómo puedo saber cuál es el estado de registro del dispositivo del cliente?

**R:** En Azure Portal, vaya a **Todos los dispositivos**. Use el identificador del dispositivo para buscarlo. Compruebe el valor en la columna de tipo de unión. A veces, puede haberse restablecido el dispositivo o su imagen inicial. Por lo tanto, también es esencial comprobar el estado de registro del dispositivo:

- En dispositivos Windows 10 y Windows Server 2016 o posteriores, ejecute `dsregcmd.exe /status`.
- En las versiones anteriores del sistema operativo, ejecute `%programFiles%\Microsoft Workplace Join\autoworkplace.exe`.

**R:** Para consultar información sobre solución de problemas, vea los artículos:
- [Solución de problemas de dispositivos mediante el comando dsregcmd](troubleshoot-device-dsregcmd.md)
- [Solución de problemas de dispositivos híbridos de Windows 10 y Windows Server 2016 unidos a Azure Active Directory](troubleshoot-hybrid-join-windows-current.md)
- [Solución de problemas de dispositivos híbridos de nivel inferior unidos a Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)

---

### <a name="q-i-see-the-device-record-under-the-user-info-in-the-azure-portal-and-i-see-the-state-as-registered-on-the-device-am-i-set-up-correctly-to-use-conditional-access"></a>P: Puedo ver el registro del dispositivo en la información del usuario de Azure Portal. Además, puedo ver el estado del dispositivo en "registrado". ¿He establecido la configuración correctamente para utilizar el acceso condicional?

**R:** El estado de unión del dispositivo, que se muestra en **deviceID**, debe coincidir con el de Azure AD y satisfacer los criterios de evaluación del acceso condicional. Para más información, consulte [Exigir dispositivos administrados para el acceso a aplicaciones en la nube con acceso condicional](../conditional-access/require-managed-devices.md).

---

### <a name="q-why-do-my-users-see-an-error-message-saying-your-organization-has-deleted-the-device-or-your-organization-has-disabled-the-device-on-their-windows-10-devices"></a>P: ¿Por qué los usuarios ven un mensaje de error que indica "La organización ha eliminado el dispositivo" o "La organización ha deshabilitado el dispositivo" en los dispositivos Windows 10?

**R:** En los dispositivos de Windows 10 conectados o registrados con Azure AD, se emite un [token de actualización principal (PRT)](concept-primary-refresh-token.md) que permite el inicio de sesión único. La validez de dicho token se basa en la validez del propio dispositivo. Los usuarios verán este mensaje si el dispositivo se ha eliminado o deshabilitado en Azure AD sin iniciar la acción desde el propio dispositivo. Un dispositivo se puede eliminar o deshabilitar en Azure AD uno de los siguientes escenarios: 

- El usuario deshabilita el dispositivo en el portal Mis aplicaciones. 
- Un administrador (o usuario) elimina o deshabilita el dispositivo en Azure Portal o mediante PowerShell.
- Solo unido a Azure AD híbrido: Un administrador quita la unidad organizativa de los dispositivos del ámbito de sincronización, lo que da lugar a la eliminación de los dispositivos de Azure AD.
- Actualización de Azure AD Connect a la versión 1.4.xx.x. [Descripción de Azure AD Connect 1.4.xx.x y desaparición del dispositivo](../hybrid/reference-connect-device-disappearance.md).


Vea a continuación cómo se pueden rectificar estas acciones.

---

### <a name="q-i-disabled-or-deleted-my-device-in-the-azure-portal-or-by-using-windows-powershell-but-the-local-state-on-the-device-says-its-still-registered-what-should-i-do"></a>P: He deshabilitado o eliminado mi dispositivo de Azure Portal o mediante Windows PowerShell, pero el estado local del dispositivo indica que todavía está registrado. ¿Cuál debo hacer?

**R:** Esta operación es así por diseño. En este caso, el dispositivo no tiene acceso a los recursos en la nube. Los administradores pueden realizar esta acción en dispositivos obsoletos, perdidos o robados para evitar el acceso no autorizado. Si esta acción se realiza de forma no intencionada, deberá volver a habilitar o volver a registrar el dispositivo, tal y como se describe a continuación.

- Si el dispositivo se deshabilitó en Azure AD, un administrador con suficientes privilegios puede habilitarlo desde el portal de Azure AD.  
  > [!NOTE]
  > Si va a sincronizar dispositivos mediante Azure AD Connect, los dispositivos unidos a Azure AD híbrido se volverán a habilitar automáticamente durante el siguiente ciclo de sincronización. Por lo tanto, si necesita deshabilitar un dispositivo unido a Azure AD híbrido, debe deshabilitarlo de la instancia de AD local.

 - Si el dispositivo se elimina en Azure AD, debe volver a registrar el dispositivo. Para volver a registrar el dispositivo, deberá realizar algunas acciones manualmente en él. Consulte a continuación las instrucciones para volver a registrar en función del estado del dispositivo. 

      Para volver a registrar los dispositivos unidos a Azure AD híbrido para Windows 10 y Windows Server 2016/2019, siga estos pasos:

      1. Abra el símbolo del sistema como administrador.
      1. Escriba `dsregcmd.exe /debug /leave`.
      1. Cierre la sesión y luego iníciela para desencadenar la tarea programada que registra de nuevo el dispositivo con Azure AD. 

      Para las versiones anteriores del sistema operativo Windows que estén unidas a Azure AD híbrido, realice los pasos siguientes:

      1. Abra el símbolo del sistema como administrador.
      1. Escriba `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe /l"`.
      1. Escriba `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe /j"`.

      En los dispositivos de Windows 10 para dispositivos unidos a Azure AD, siga estos pasos:

      1. Abra el símbolo del sistema como administrador.
      1. Escriba `dsregcmd /forcerecovery` (Nota: Debe ser administrador para realizar esta acción).
      1. Haga clic en "Iniciar sesión" en el cuadro de diálogo que se abre y continúe con el proceso de inicio de sesión.
      1. Cierre la sesión y vuelva a iniciarla en el dispositivo para completar la recuperación.

      En el caso de los dispositivos de Windows 10 registrados en Azure AD, realice los pasos siguientes:

      1. Vaya a **Configuración** > **Cuentas** > **Obtener acceso a trabajo o escuela**. 
      1. Seleccione la cuenta y después **Desconectar**.
      1. Haga clic en "+ Conectar" y vuelva a registrar el dispositivo. Para ello, pase por el proceso de inicio de sesión.

---

### <a name="q-why-do-i-see-duplicate-device-entries-in-the-azure-portal"></a>P: ¿Por qué veo entradas duplicadas del dispositivo en Azure Portal?

**R:**

- En Windows 10 y Windows Server 2016, si se realizan varios intentos para separar y volver a unir el mismo dispositivo, pueden aparecer entradas duplicadas. 
- Cada usuario de Windows que usa **Agregar cuenta profesional o educativa** crea un nuevo registro de dispositivo con el mismo nombre de dispositivo.
- En las versiones anteriores del sistema operativo Windows que estén unidas a un dominio de Azure Directory local, el registro automático crea un nuevo registro del dispositivo con el mismo nombre de dispositivo para cada usuario del dominio que inicie sesión en dicho dispositivo. 
- En el caso de las máquinas unidas a Azure AD que se han borrado, se han vuelto a instalar y se han vuelto a unir con el mismo nombre, aparece otro registro con el mismo nombre de dispositivo.

---

### <a name="q-does-windows-10-device-registration-in-azure-ad-support-tpms-in-fips-mode"></a>P: ¿El registro de dispositivos Windows 10 en Azure AD admite TPM en el modo FIPS?

**R:** El registro de dispositivos Windows 10 solo se admite con TPM 2.0 compatible con FIPS y no se admite en TPM 1.2. Si los dispositivos tienen TPM 1.2 compatible con FIPS, debe deshabilitarlos antes de continuar con la unión a Azure AD o la unión a Azure AD híbrido. Microsoft no proporciona ninguna herramienta para deshabilitar el modo FIPS para TPM, ya que eso depende del fabricante de TPM. Póngase en contacto con el OEM de hardware para obtener soporte técnico. 

---

**P: ¿Por qué un usuario puede tener acceso a recursos desde un dispositivo que deshabilité en Azure Portal?**

**R:** La revocación tarda hasta una hora en aplicarse desde el momento en el que el dispositivo de Azure AD se marca como deshabilitado.

>[!NOTE] 
>En el caso de los dispositivos inscritos, se recomienda que borre los datos del dispositivo para asegurarse de que los usuarios no pueden acceder a los recursos. Para obtener más información, consulte [¿Qué es la inscripción de dispositivos?](/mem/intune/user-help/use-managed-devices-to-get-work-done). 

---

### <a name="q-why-are-there-devices-marked-as-pending-under-the-registered-column-in-the-azure-portal"></a>P: ¿Por qué hay dispositivos marcados como "Pendientes" en la columna REGISTRADO en Azure Portal?

**R.** :  Pendiente indica que el dispositivo no está registrado. Este estado indica que un dispositivo se ha sincronizado con Azure AD Connect desde una instancia de AD local y está listo para el registro de dispositivos. Estos dispositivos tienen el TIPO DE UNIÓN establecido en "Unidos a Azure AD híbrido". Para más información, consulte [Instrucciones: Planeamiento de la implementación de la unión a Azure Active Directory híbrido](hybrid-azuread-join-plan.md).

>[!NOTE]
>Un dispositivo también puede pasar de tener un estado registrado a "Pendiente".
>* Si se elimina un dispositivo de Azure AD primero y se vuelve a sincronizar desde una instancia de AD local.
>* Si un dispositivo se quita de un ámbito de sincronización en Azure AD Connect y se vuelve a agregar.
>
>En ambos casos, debe volver a registrar el dispositivo manualmente en cada uno de estos dispositivos. Para revisar si el dispositivo se registró anteriormente, puede [solucionar problemas de los dispositivos mediante el comando dsregcmd](troubleshoot-device-dsregcmd.md).

---
## <a name="azure-ad-join-faq"></a>P+F de unión a Azure AD

### <a name="q-how-do-i-unjoin-an-azure-ad-joined-device-locally-on-the-device"></a>P: ¿Cómo se separa un dispositivo unido a Azure AD localmente en el dispositivo?

**R:** Para los dispositivos unidos a Azure AD puro, asegúrese de tener una cuenta de administrador local sin conexión o cree una. No podrá iniciar sesión con credenciales de usuario de Azure AD. A continuación, vaya a **Configuración** > **Cuentas** > **Obtener acceso a trabajo o escuela**. Seleccione su cuenta y luego **Desconectar**. Siga las indicaciones y proporcione las credenciales de administrador local cuando se le solicite. Reinicie el dispositivo para finalizar el proceso de separación.

---

### <a name="q-can-my-users-sign-in-to-azure-ad-joined-devices-that-are-deleted-or-disabled-in-azure-ad"></a>P: ¿Mis usuarios pueden iniciar sesión en dispositivos unidos a Azure AD que están eliminados o deshabilitados en Azure AD?

**R:** Sí. Windows tiene una funcionalidad para almacenar en caché un nombre de usuario y contraseña. Esta funcionalidad permite que los usuarios que han iniciado sesión anteriormente accedan rápidamente al escritorio, incluso sin conectividad de red. 

Cuando un dispositivo se elimina o deshabilita en Azure AD, el dispositivo Windows no lo reconoce. Por lo que los usuarios que han iniciado sesión anteriormente siguen teniendo acceso al escritorio con el nombre de usuario y contraseña almacenados en caché. Pero dado que el dispositivo se ha eliminado o deshabilitado, los usuarios no pueden acceder a los recursos protegidos con el acceso condicional basado en el dispositivo. 

Los usuarios que no han iniciado sesión anteriormente no pueden acceder al dispositivo. No se ha habilitado ningún nombre de usuario o contraseña almacenados en caché para ellos. 

---

### <a name="q-can-a-disabled-or-deleted-user-sign-in-to-an-azure-ad-joined-devices"></a>P: ¿Puede un usuario deshabilitado o eliminado iniciar sesión en dispositivos unidos a Azure AD?

**R:** Sí, pero solo durante un tiempo limitado. Cuando un usuario se elimina o deshabilita en Azure AD, el dispositivo Windows no lo reconoce inmediatamente. Por lo que los usuarios que han iniciado sesión anteriormente pueden acceder al escritorio con el nombre de usuario y contraseña almacenados en caché. 

Normalmente, el dispositivo reconoce el estado del usuario en menos de cuatro horas. Tras lo cual, Windows bloquea el acceso de esos usuarios al escritorio. Como el usuario está eliminado o deshabilitado en Azure AD, se revocan todos sus tokens, por lo que no puede tener acceso a ninguno de los recursos. 

Los usuarios eliminados o deshabilitados que no han iniciado sesión anteriormente no pueden acceder al dispositivo. No se ha habilitado ningún nombre de usuario o contraseña almacenados en caché para ellos. 

---

### <a name="q-why-do-my-users-have-issues-on-azure-ad-joined-devices-after-changing-their-upn"></a>P: ¿Por qué mis usuarios tienen problemas en los dispositivos unidos a Azure AD después de cambiar su UPN?

**R:** Actualmente, los cambios de UPN no son totalmente compatibles en dispositivos unidos a Azure AD. Por ello, su autenticación con Azure AD genera un error tras la modificación del UPN. Como resultado, los usuarios experimentarán problemas con el acceso condicional y el inicio de sesión único en sus dispositivos. Por tanto, para poder solucionar este problema, los usuarios deben iniciar sesión en Windows con el icono “Otro usuario” mediante su nuevo UPN. Estamos trabajando actualmente para poder resolver este incidente. Sin embargo, los usuarios que inicien sesión con Windows Hello para empresas no tendrán este problema. 

Los cambios de nombre principal de usuario se admiten con la actualización 2004 de Windows 10. Los usuarios de dispositivos con esta actualización no tendrán ningún problema después de cambiar su nombre principal de usuario.

---

### <a name="q-my-users-cant-search-printers-from-azure-ad-joined-devices-how-can-i-enable-printing-from-those-devices"></a>P: Mis usuarios no pueden buscar impresoras desde dispositivos unidos a Azure AD. ¿Cómo se puede habilitar la impresión desde esos dispositivos?

**R:** Con el fin de implementar impresoras para los dispositivos unidos a Azure AD, consulte [Deploy Windows Server Hybrid Cloud Print with Pre-Authentication](/windows-server/administration/hybrid-cloud-print/hybrid-cloud-print-deploy) (Implementación de impresión en nube híbrida de Windows Server mediante autenticación previa). Necesita un servidor de Windows Server local para implementar la impresión en nube híbrida. Actualmente, el servicio de impresión basado en la nube no está disponible. 

---

### <a name="q-how-do-i-connect-to-a-remote-azure-ad-joined-device"></a>P: ¿Cómo me conecto a un dispositivo remoto unido a Azure AD?

**R:** Consulte [Conectarse a un equipo remoto unido a Azure Active Directory](/windows/client-management/connect-to-remote-aadj-pc).

---

### <a name="q-why-do-my-users-see-you-cant-get-there-from-here"></a>P: ¿Por qué mis usuarios ven un mensaje que indica que *no pueden acceder desde aquí?*

**R:** ¿Ha configurado determinadas reglas de acceso condicional para requerir un estado de dispositivo específico? Si el dispositivo no cumple los criterios, se bloquea a los usuarios y se les muestra ese mensaje. Examine las reglas de la directiva de acceso condicional. Asegúrese de que el dispositivo cumple los criterios para evitar el mensaje.

---

### <a name="q-why-dont-some-of-my-users-get-azure-multi-factor-authentication-prompts-on-azure-ad-joined-devices"></a>P: ¿Por qué algunos de mis usuarios no reciben las indicaciones de Azure Multi-factor Authentication en dispositivos unidos a Azure AD?

**R:** Un usuario puede unir o registrar un dispositivo a Azure AD mediante Multi-Factor Authentication. De esta forma, el dispositivo se convierte en un segundo factor de confianza para dicho usuario. Siempre que el mismo usuario inicia sesión en el dispositivo y accede a una aplicación, Azure AD considera al dispositivo como segundo factor. Esta función permite que el usuario acceda sin problemas a las aplicaciones sin indicaciones adicionales de Multi-Factor Authentication. 

Este comportamiento:

- Es aplicable a dispositivos unidos a Azure AD y a dispositivos registrados de Azure AD, pero no a dispositivos unidos a Azure AD híbridos.
- No es aplicable a ningún otro usuario que inicie sesión en el dispositivo. Por lo que todos los demás usuarios que accedan a dicho dispositivo recibirán un desafío de Multi-Factor Authentication. Después de realizarlo, podrán tener acceso a las aplicaciones que requieren de Multi-Factor Authentication.

---

### <a name="q-why-do-i-get-a-username-or-password-is-incorrect-message-for-a-device-i-just-joined-to-azure-ad"></a>P: ¿Por qué me aparece el mensaje *El nombre de usuario o la contraseña no son correctos* para un dispositivo que acabo de unir a Azure AD?

**R:** Las razones comunes para este escenario son las siguientes:

- Las credenciales de usuario ya no son válidas.
- El equipo no puede comunicarse con Azure Active Directory. Compruebe si existen errores de conectividad de red.
- Los inicios de sesión federados requieren que el servidor de federación admita puntos de conexión WS-Trust que estén habilitados y accesibles. 
- Habilitó la autenticación de paso a través. Por tanto, debe cambiar su contraseña temporal al iniciar sesión.

---

### <a name="q-why-do-i-see-the-oops-an-error-occurred-dialog-when-i-try-to-azure-ad-join-my-pc"></a>P: ¿Por qué se muestra el cuadro de diálogo *¡Vaya!... se produjo un error* cuando intento unir mi equipo a Azure AD?

**R:** Este error se produce cuando configura la inscripción de Azure Active Directory con Intune. Asegúrese de que el usuario que intenta unirse a Azure AD tiene asignada la licencia de Intune correcta. Para obtener más información, consulte [Configuración de la inscripción de dispositivos Windows](/intune/windows-enroll).  

---

### <a name="q-why-did-my-attempt-to-azure-ad-join-a-pc-fail-although-i-didnt-get-any-error-information"></a>P: ¿Por qué se produce un error al intentar unir mi equipo a Azure AD a pesar de que no he obtenido información sobre dicho error?

**R:** Puede deberse a que inició sesión en el dispositivo con la cuenta de administrador integrada local. Cree una cuenta local distinta antes de usar la unión de Azure Active Directory para completar la instalación. 

---

### <a name="qwhat-are-the-ms-organization-p2p-access-certificates-present-on-our-windows-10-devices"></a>P: ¿Cuáles son los certificados MS-Organization-P2P-Access presentes en nuestros dispositivos Windows 10?

**R:** Azure AD emite los certificados Ms-Organization-P2P-Access para los dispositivos unidos a Azure AD y a Azure AD híbrido. Estos certificados se usan para habilitar la confianza entre los dispositivos del mismo inquilino en escenarios de escritorios remotos. Se emite un certificado al dispositivo y otro al usuario. El certificado del dispositivo está presente en `Local Computer\Personal\Certificates` y es válido durante un día. Este certificado se renueva (mediante la emisión de un nuevo certificado) si el dispositivo está activo en Azure AD. El certificado de usuario está presente en `Current User\Personal\Certificates` y dicho certificado también es válido durante un día, pero se emite a petición cuando un usuario intenta establecer una sesión de escritorio remoto con otro dispositivo unido a Azure AD. No se renueva cada vez que expira. Ambos certificados se emiten con el certificado MS-Organization-P2P-Access presente en `Local Computer\AAD Token Issuer\Certificates`. Azure AD emite este certificado durante el registro de dispositivos. 

---

### <a name="qwhy-do-i-see-multiple-expired-certificates-issued-by-ms-organization-p2p-access-on-our-windows-10-devices-how-can-i-delete-them"></a>P: ¿Por qué se muestran varios certificados expirados que emitió MS-Organization-P2P-Access en nuestros dispositivos Windows 10? ¿Cómo puedo eliminarlos?

**R:** Hubo un problema identificado en Windows 10, versión 1709 y versiones anteriores en el que los certificados expirados de MS-Organization-P2P-Access aún existían en el equipo debido a problemas de cifrado. Los usuarios se pueden encontrar con problemas de conectividad de red si está usando algún cliente VPN (por ejemplo, Cisco AnyConnect) que no pueda gestionar el gran número de certificados expirados. Este problema se corrigió en Windows 10, versión 1803 para que se eliminen automáticamente estos certificados MS-Organization-P2P-Access expirados. Puede solucionar este problema al actualizar los dispositivos a Windows 10, 1803. Si no puede actualizarlos, puede eliminar dichos certificados sin ningún impacto negativo.  

---

## <a name="hybrid-azure-ad-join-faq"></a>P+F de unión a Azure AD híbrido

### <a name="q-how-do-i-unjoin-a-hybrid-azure-ad-joined-device-locally-on-the-device"></a>P: ¿Cómo se separa un dispositivo unido a Azure AD híbrido localmente en el dispositivo?

**R:** En el caso de los dispositivos unidos a Azure AD híbrido, asegúrese de desactivar el registro automático. De esta forma, la tarea programada no registrará el dispositivo de nuevo. A continuación, abra un símbolo del sistema como administrador y escriba `dsregcmd.exe /debug /leave`. O bien ejecute este comando como un script en varios dispositivos para separarlos de forma masiva.

### <a name="q-where-can-i-find-troubleshooting-information-to-diagnose-hybrid-azure-ad-join-failures"></a>P: ¿Dónde puedo encontrar información de solución de problemas para diagnosticar errores de unión a Azure AD híbrido?

**R:** Para consultar información sobre solución de problemas, vea los artículos:

- [Solución de problemas de dispositivos híbridos de Windows 10 y Windows Server 2016 unidos a Azure Active Directory](troubleshoot-hybrid-join-windows-current.md)
- [Solución de problemas de dispositivos híbridos de nivel inferior unidos a Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)
 
### <a name="q-why-do-i-see-a-duplicate-azure-ad-registered-record-for-my-windows-10-hybrid-azure-ad-joined-device-in-the-azure-ad-devices-list"></a>P: ¿Por qué veo un registro registrado duplicado de Azure AD para mi dispositivo híbrido unido a Azure AD de Windows 10 en la lista de dispositivos de Azure AD?

**R:** Cuando los usuarios agregan sus cuentas a las aplicaciones en un dispositivo unido a un dominio, se les puede preguntar si quieren **agregarla a Windows**. Si indican que **Sí** en la ventana, el dispositivo se registra con Azure AD. El tipo de confianza se marca como registrado en Azure AD. Después de que habilite la unión de Azure AD híbrido en una organización, el dispositivo también se une con Azure AD híbrido. De esta forma, se muestran dos estados para el mismo dispositivo. 

La unión de Azure AD híbrido tiene prioridad sobre el estado registrado en Azure AD. Por tanto, el dispositivo se considera con unión a Azure AD híbrido para cualquier autenticación y evaluación de acceso condicional. Puede eliminar sin problemas el registro de dispositivos registrados en Azure AD desde el portal de Azure AD. Aprenda a [evitar o borrar los dos estados en el equipo de Windows 10](hybrid-azuread-join-plan.md#review-things-you-should-know). 

---

### <a name="q-why-do-my-users-have-issues-on-windows-10-hybrid-azure-ad-joined-devices-after-changing-their-upn"></a>P: ¿Por qué mis usuarios tienen problemas en los dispositivos híbridos unidos a Azure AD en Windows 10 después de cambiar su UPN?

**R:** Actualmente, los cambios de UPN no son totalmente compatibles en dispositivos híbridos unidos a Azure AD. Aunque los usuarios pueden iniciar sesión en el dispositivo y acceder a sus aplicaciones de forma local, se produce un error en la autenticación con Azure AD después de cambiar un UPN. Como resultado, los usuarios experimentarán problemas con el acceso condicional y el inicio de sesión único en sus dispositivos. Para solucionar el problema, el usuario debe desunir el dispositivo de Azure AD (ejecutar "dsregcmd /leave" con privilegios elevados) y volver a realizar la unión (se produce automáticamente). Estamos trabajando actualmente para poder resolver este incidente. Sin embargo, los usuarios que inicien sesión con Windows Hello para empresas no tendrán este problema. 

Los cambios de nombre principal de usuario se admiten con la actualización 2004 de Windows 10. Los usuarios de dispositivos con esta actualización no tendrán ningún problema después de cambiar su nombre principal de usuario.

---

### <a name="q-do-windows-10-hybrid-azure-ad-joined-devices-require-line-of-sight-to-the-domain-controller-to-get-access-to-cloud-resources"></a>P: ¿Requieren los dispositivos unidos a Azure AD híbrido en Windows 10 línea de visión al controlador de dominio para acceder a recursos en la nube?

**R:** No, excepto cuando se cambia la contraseña del usuario. Después de completar la unión a Azure AD híbrido en Windows 10 y de que el usuario ha iniciado sesión al menos una vez, el dispositivo no requiere línea de visión al controlador de dominio para acceder a recursos en la nube. Windows 10 puede obtener inicio de sesión único a aplicaciones de Azure AD desde cualquier lugar con conexión a internet, excepto cuando se cambia una contraseña. Los usuarios que inician sesión con Windows Hello para empresas continúan recibiendo el inicio de sesión único en las aplicaciones de Azure AD incluso después de un cambio de contraseña, aunque no tengan línea de visión a su controlador de dominio. 

---

### <a name="q-what-happens-if-a-user-changes-their-password-and-tries-to-login-to-their-windows-10-hybrid-azure-ad-joined-device-outside-the-corporate-network"></a>P: ¿Qué ocurre si un usuario cambia la contraseña e intenta iniciar sesión en su dispositivo Windows 10 unido a Azure AD híbrido fuera de la red corporativa?

**R:** Si se cambia una contraseña fuera de la red corporativa (por ejemplo, mediante el uso de SSPR de Azure AD), se producirá un error en el inicio de sesión de usuario con la nueva contraseña. Para los dispositivos unidos a Azure AD híbrido, el directorio local de Active Directory es la autoridad principal. Cuando un dispositivo no tiene línea de visión al controlador de dominio, no puede validar la nueva contraseña. Por tanto, el usuario necesita establecer conexión con el controlador de dominio (ya sea a través de VPN o desde dentro de la red corporativa) antes de poder iniciar sesión en el dispositivo con la nueva contraseña. En caso contrario, solo puede iniciar sesión con la contraseña antigua debido a la capacidad de Windows de inicio de sesión almacenado en caché. Sin embargo, Azure AD invalida la contraseña antigua durante las solicitudes de token y, por tanto, no se puede realizar el inicio de sesión único y no se cumplen las directivas de acceso condicional basado en el dispositivo. Este problema no se produce cuando se usa Windows Hello para empresas. 

---

## <a name="azure-ad-register-faq"></a>P+F de registro de Azure AD

### <a name="q-how-do-i-remove-an-azure-ad-registered-state-for-a-device-locally"></a>P: ¿Cómo se quita el estado registrado de Azure AD en un dispositivo a nivel local?

**R:** 
- Para los dispositivos registrados en Azure AD Windows 10 vaya a **Configuración** > **Cuentas** > **Obtener acceso a trabajo o escuela**. Seleccione su cuenta y luego **Desconectar**. El registro de dispositivos es por perfil de usuario en Windows 10.
- Para iOS y Android, puede usar **Configuración** > **Registro de dispositivos** la aplicación Microsoft Authenticator configuración y seleccionar **Anular registro del dispositivo** .
- En macOS, puede usar la aplicación Portal de empresa de Microsoft Intune para anular la inscripción del dispositivo en la administración y quitar cualquier registro. 

---
### <a name="q-how-can-i-block-users-from-adding-additional-work-accounts-azure-ad-registered-on-my-corporate-windows-10-devices"></a>P: ¿Cómo puedo impedir que los usuarios agreguen más cuentas de trabajo (registradas en Azure AD) en mis dispositivos corporativos de Windows 10?

**R:** Habilite el siguiente registro para evitar que los usuarios agreguen cuentas de trabajo adicionales a los dispositivos Windows 10 corporativos unidos a un dominio o a Azure AD o híbridos unidos a Azure AD. Esta directiva también se puede usar para impedir que las máquinas unidas a un dominio reciban involuntariamente la instancia de Azure AD registrada con la misma cuenta de usuario. 

`HKLM\SOFTWARE\Policies\Microsoft\Windows\WorkplaceJoin, "BlockAADWorkplaceJoin"=dword:00000001`

---
### <a name="q-can-i-register-android-or-ios-byod-devices"></a>P: ¿Puedo registrar los dispositivos BYOD de Android o iOS?

**R:** Sí, pero solo con el servicio de registro de dispositivos de Azure y para clientes híbridos. No es compatible con el servicio de registro de dispositivos local en los servicios de federación de Active Directory (AD FS).

---
### <a name="q-how-can-i-register-a-macos-device"></a>P: ¿Cómo puedo registrar un dispositivo macOS?

**R:** Siga estos pasos.

1.    [Cree una directiva de cumplimiento](/intune/compliance-policy-create-mac-os).
1.    [Defina una directiva de acceso condicional para dispositivos de Mac OS](../conditional-access/overview.md). 

**Observaciones**:

- Los usuarios incluidos en la directiva de acceso condicional necesitan una [versión admitida de Office para macOS](../conditional-access/concept-conditional-access-conditions.md) para acceder a los recursos. 
- Durante el primer intento de acceso, se pide a los usuarios que inscriban el dispositivo mediante el portal de la empresa.

---
## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre los [dispositivos registrados en Azure AD](concept-azure-ad-register.md).
- Obtenga más información sobre los [dispositivos unidos a Azure AD](concept-azure-ad-join.md).
- Obtenga más información sobre los [dispositivos híbridos unidos a Azure AD](concept-azure-ad-join-hybrid.md).