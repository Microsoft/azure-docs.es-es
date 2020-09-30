---
title: Consideraciones relativas a la implementación de Azure Multi-Factor Authentication
description: Obtenga información sobre las consideraciones relativas a la implementación de Azure Multi-Factor Authentication y la estrategia para realizarla correctamente
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: da9a0fe30721a2f55db2c4771f54716868f9972f
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055080"
---
# <a name="plan-an-azure-multi-factor-authentication-deployment"></a>Planificación de una implementación de Azure Multi-Factor Authentication

Las personas se conectan con recursos de la organización en escenarios cada vez más complicados. Los usuarios se conectan desde dispositivos propiedad de la organización, personales y públicos, desde redes corporativas y no corporativas mediante smartphones, tabletas, PC y equipos portátiles, a menudo en varias plataformas. En este mundo conectado siempre, multidispositivo y multiplataforma, la seguridad de las cuentas de usuario es más importante que nunca. Las contraseñas, independientemente de su complejidad, que se usan en varios dispositivos, redes y plataformas ya no son suficientes para garantizar la seguridad de la cuenta de usuario, especialmente cuando los usuarios tienden a reutilizar contraseñas entre cuentas. El "phishing" sofisticado y otros ataques de ingeniería social pueden dar lugar a que los nombres de usuario y contraseñas se publiquen y vendan en la web oscura.

[Azure Multi-Factor Authentication (MFA)](concept-mfa-howitworks.md) ayuda a proteger el acceso a los datos y las aplicaciones. Proporciona una capa adicional de seguridad mediante una segunda forma de autenticación. Las organizaciones pueden usar el [acceso condicional](../conditional-access/overview.md) para que la solución se ajuste a sus necesidades específicas.

En esta guía de implementación se muestra cómo planear y probar un lanzamiento de Azure Multi-Factor Authentication.

Para ver rápidamente Azure Multi-Factor Authentication en acción y, después, volver para conocer otras consideraciones acerca de la implementación:

> [!div class="nextstepaction"]
> [Habilitación de Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md)

## <a name="prerequisites"></a>Prerrequisitos

Antes de iniciar una implementación de Azure Multi-Factor Authentication, existen requisitos previos que deben tenerse en cuenta.

| Escenario | Requisito previo |
| --- | --- |
| Entorno de identidades **solo en la nube** con autenticación moderna. | **Sin tareas de requisitos previos adicionales**. |
| Escenarios de identidad **híbridos**. | [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) está implementado y las identidades de usuario están sincronizadas o federadas con la instancia local de Active Directory Domain Services con Azure Active Directory. |
| Aplicaciones heredadas locales que se publican para el acceso a la nube. | Azure AD [Application Proxy](../manage-apps/application-proxy.md) está implementado. |
| Se usa Azure MFA con la autenticación RADIUS | Está implementado un [servidor de directivas de redes (NPS)](howto-mfa-nps-extension.md). |
| Los usuarios tienen Microsoft Office 2010 o versiones anteriores, o bien Apple Mail para iOS 11 o versiones anteriores. | Actualización a [Microsoft Office 2013 o versiones posteriores](https://support.microsoft.com/help/4041439/modern-authentication-configuration-requirements-for-transition-from-o), o bien a Apple Mail para iOS 12 o versiones posteriores. Los protocolos de autenticación heredada no admiten el acceso condicional. |

## <a name="plan-user-rollout"></a>Planificación del lanzamiento para el usuario

El plan de lanzamiento de MFA debe incluir un lanzamiento piloto seguido de fases de implementación dentro de su capacidad admitida. Comience el lanzamiento al aplicar las directivas de acceso condicional a un grupo reducido de usuarios piloto. Después de evaluar el efecto en los usuarios piloto, el proceso que se utiliza y los comportamientos de registro, puede agregar más grupos a la directiva o agregar más usuarios a los grupos existentes.

### <a name="user-communications"></a>Comunicaciones de los usuarios

Es fundamental informar a los usuarios, en comunicaciones planificadas, acerca de los próximos cambios, los requisitos de registro para Azure MFA y las acciones que debe realizar el usuario. Se recomienda que las comunicaciones se desarrollen con ayuda de los representantes de su organización; por ejemplo, los departamentos de recursos humanos, administración de cambios o comunicaciones.

Microsoft proporciona [plantillas de comunicación](https://aka.ms/mfatemplates) y [documentación de usuario final](../user-help/security-info-setup-signin.md) para ayudarle a crear borradores de sus comunicaciones. Puede enviar a los usuarios a [https://myprofile.microsoft.com](https://myprofile.microsoft.com) para registrarse directamente si seleccionan los vínculos **Información de seguridad** en la página.

## <a name="deployment-considerations"></a>Consideraciones de la implementación

Azure Multi-Factor Authentication se implementa mediante la aplicación de directivas de acceso condicional. Una directiva de acceso condicional puede requerir que los usuarios realicen la autenticación multifactor cuando se cumplen ciertos criterios, por ejemplo:

* Todos los usuarios, un usuario específico, el miembro de un grupo o un rol asignado
* Se accede a una aplicación en la nube específica
* Plataforma de dispositivo
* Estado del dispositivo
* Ubicación de red o ubicación geográfica de dirección IP
* Aplicaciones cliente
* Riesgo de inicio de sesión (requiere Identity Protection)
* Dispositivos compatible
* Dispositivo unido a Azure AD híbrido
* Aplicación cliente aprobada

Utilice los pósteres y plantillas de correo electrónico personalizables de los [materiales de lanzamiento de la autenticación multifactor](https://www.microsoft.com/download/details.aspx?id=57600&WT.mc_id=rss_alldownloads_all) para lanzar la autenticación multifactor en su organización.

## <a name="enable-multi-factor-authentication-with-conditional-access"></a>Habilitación de Multi-Factor Authentication con acceso condicional

Las directivas de acceso condicional fomentan el registro, ya que los usuarios no registrados deben completar su registro en el primer inicio de sesión, una consideración de seguridad importante.

[Azure AD Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md) contribuye con directivas de registro y directivas de detección y corrección automatizada de riesgos para el caso de Azure Multi-Factor Authentication. Las directivas se pueden crear para forzar los cambios de contraseña cuando hay una amenaza de identidad en peligro o pueden requerir MFA cuando un inicio de sesión se considera de riesgo según los siguientes [eventos](../identity-protection/overview-identity-protection.md):

* Credenciales con fugas
* Inicios de sesión desde direcciones IP anónimas
* Viaje imposible a ubicaciones inusuales
* Inicios de sesión desde ubicaciones desconocidas
* Inicios de sesión desde dispositivos infectados
* Inicios de sesión desde direcciones IP con actividad sospechosa

Algunas de las detecciones de riesgos observadas por Azure Active Directory Identity Protection se producen en tiempo real y algunas requieren un procesamiento sin conexión. Los administradores pueden bloquear a los usuarios que exhiben comportamientos de riesgo y corregir la situación manualmente, o pueden solicitar una autenticación multifactor como parte de sus directivas de acceso condicional.

## <a name="define-network-locations"></a>Definición de las ubicaciones de red

Se recomienda que las organizaciones usen acceso condicional para definir su red mediante [ubicaciones con nombre](../conditional-access/location-condition.md#named-locations). Si su organización usa Identity Protection, considere el uso de directivas basadas en riesgos en lugar de ubicaciones con nombre.

### <a name="configuring-a-named-location"></a>Configuración de una ubicación con nombre

1. Vaya a **Azure Active Directory** en Azure Portal.
2. seleccione **Seguridad**.
3. En **Administrar**, elija **Ubicaciones con nombre**.
4. Seleccione **Nueva ubicación**.
5. En el campo **Nombre** escriba un nombre descriptivo.
6. Seleccione si va a definir la ubicación mediante *Intervalos IP* o bien *Países o regiones*.
   1. Si usa *Intervalos IP*
      1. Decida si quiere utilizar la opción *Marcar como ubicación de confianza*. Iniciar sesión desde una ubicación de confianza reduce el riesgo de inicio de sesión del usuario. Solo marque esta ubicación como de confianza si sabe que los intervalos de direcciones IP especificados están establecidos y son confiables en su organización.
      2. Especifique los intervalos de direcciones IP
   2. Si usa *Países o regiones*
      1. Expanda el menú desplegable y seleccione los países o regiones que quiere definir para esta ubicación con nombre.
      2. Decida si quiere utiliza la opción *Incluir áreas desconocidas*. Las áreas desconocidas son direcciones IP que no pueden asignarse a un país o región.
7. Seleccione **Crear**

## <a name="plan-authentication-methods"></a>Planificación de métodos de autenticación

Los administradores pueden elegir los [métodos de autenticación](../authentication/concept-authentication-methods.md) que quieren que estén disponibles para los usuarios. Es importante habilitar más de un método de autenticación para que los usuarios tengan disponible un método alternativo en caso de que su método principal no esté disponible. Los métodos siguientes están disponibles para que los administradores los habiliten:

> [!TIP]
> Microsoft recomienda el uso de Microsoft Authenticator (aplicación móvil) como método principal para Azure Multi-Factor Authentication, para obtener una experiencia de usuario mejorada y más segura. La aplicación Microsoft Authenticator también [cumple](https://azure.microsoft.com/resources/microsoft-nist/) los niveles de seguridad de Authenticator del National Institute of Standards and Technology. 

### <a name="notification-through-mobile-app"></a>Notificación a través de aplicación móvil

Se envía una notificación push a la aplicación Microsoft Authenticator del dispositivo móvil. El usuario ve la notificación y selecciona **Aprobar** para completar la comprobación. Las notificaciones push a través de una aplicación móvil proporcionan la opción menos intrusiva para los usuarios. También son la opción más confiable y segura porque usan una conexión de datos en lugar de una de telefonía.

> [!NOTE]
> Si su organización tiene personal que trabaja en China o que va a viajar allí, el método **Notificación a través de aplicación móvil** en **dispositivos Android** no funciona en ese país o región. Para esos usuarios tiene que haber métodos alternativos disponibles.

### <a name="verification-code-from-mobile-app"></a>Código de verificación desde aplicación móvil

Una aplicación móvil como la de Microsoft Authenticator genera un nuevo código de verificación de OATH cada 30 segundos. El usuario escribe el código de verificación en la interfaz de inicio de sesión. La opción de aplicación móvil puede utilizarse independientemente de si el teléfono tiene una señal de telefonía móvil o datos.

### <a name="call-to-phone"></a>Llamada al teléfono

Se realiza una llamada de voz automática al usuario. El usuario responde a la llamada y pulsa **#** en el teclado del teléfono para aprobar su autenticación. La llamada a teléfono es un método alternativo excelente para los códigos de verificación o notificación de una aplicación móvil.

### <a name="text-message-to-phone"></a>Mensaje de texto al teléfono

Se envía al usuario un mensaje de texto que contiene un código de verificación; después, se le pide al usuario que escriba el código de verificación en la interfaz de inicio de sesión.

### <a name="choose-verification-options"></a>Elección de opciones de comprobación

1. Vaya a **Azure Active Directory**, **Usuarios**, **Multi-Factor Authentication**.

   ![Acceso al portal de Multi-factor Authentication desde la hoja de usuarios de Azure AD en Azure Portal](media/howto-mfa-getstarted/users-mfa.png)

1. En la nueva pestaña que se abre, vaya a **valor de configuración del servicio**.
1. En **opciones de comprobación**, active todas las casillas para los métodos disponibles para los usuarios.

   ![Configuración de métodos de verificación en la pestaña de configuración del servicio Multi-Factor Authentication](media/howto-mfa-getstarted/mfa-servicesettings-verificationoptions.png)

1. Haga clic en **Guardar**.
1. Cierre la pestaña **configuración del servicio**.

## <a name="plan-registration-policy"></a>Planificación de la directiva de registro

Los administradores deben determinar la forma en que los usuarios registrarán sus métodos. Las organizaciones deben [habilitar la nueva experiencia de registro combinado](howto-registration-mfa-sspr-combined.md) para Azure MFA y el autoservicio de restablecimiento de contraseña (SSPR). Dicho servicio permite que los usuarios restablezcan su contraseña de forma segura con los mismos métodos que se usan para la autenticación multifactor. Se recomienda usar este registro combinado, porque es una excelente experiencia para los usuarios, con la capacidad de registrarse una vez en ambos servicios. Si habilita los mismos métodos para SSPR y Azure MFA, permitirá a los usuarios a registrarse para utilizar ambas características.

### <a name="registration-with-identity-protection"></a>Registro con Identity Protection

Si su organización usa Azure Active Directory Identity Protection, [configure la directiva de registro de MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md) para pedir a los usuarios que se registren la próxima vez que inicien sesión de forma interactiva.

### <a name="registration-without-identity-protection"></a>Registro sin Identity Protection

Si su organización no tiene licencias que habiliten Identity Protection, se pedirá a los usuarios que se registren la próxima vez que se solicite MFA para iniciar de sesión. Los usuarios no se pueden registrar para MFA si no usan aplicaciones protegidas con MFA. Es importante que todos los usuarios se registren, para que los individuos malintencionados no puedan adivinar la contraseña de un usuario y registrarse para MFA en su nombre, ya que tomarían efectivamente el control de la cuenta.

#### <a name="enforcing-registration"></a>Aplicación del registro

Mediante los siguientes pasos, una directiva de acceso condicional puede obligar a los usuarios a registrarse para Multi-Factor Authentication.

1. Cree un grupo y agregue a todos los usuarios que no está registrados.
2. Mediante el acceso condicional, exija la autenticación multifactor para este grupo a fin de acceder a todos los recursos.
3. Periódicamente, vuelva a evaluar la pertenencia al grupo y quite a los usuarios del grupo que ya están registrados.

Puede identificar a los usuarios de Azure MFA registrados y no registrados con comandos de PowerShell que se basan en el [módulo MSOnline de PowerShell](/powershell/azure/active-directory/install-msonlinev1?view=azureadps-1.0).

#### <a name="identify-registered-users"></a>Identificación de usuarios registrados

```PowerShell
Get-MsolUser -All | where {$_.StrongAuthenticationMethods -ne $null} | Select-Object -Property UserPrincipalName | Sort-Object userprincipalname 
```

#### <a name="identify-non-registered-users"></a>Identificación de usuarios no registrados

```PowerShell
Get-MsolUser -All | where {$_.StrongAuthenticationMethods.Count -eq 0} | Select-Object -Property UserPrincipalName | Sort-Object userprincipalname 
```

### <a name="convert-users-from-per-user-mfa-to-conditional-access-based-mfa"></a>Convertir a los usuarios de MFA por usuario a MFA basado en el acceso condicional

Si los usuarios se habilitaron con Azure Multi-Factor Authentication habilitado y aplicado por usuario, el siguiente comando de PowerShell pueden ayudarle a realizar la conversión a Azure Multi-Factor Authentication basado en acceso condicional.

Ejecútelo en una ventana de ISE o guárdelo como un archivo `.PS1` para ejecutarlo localmente.

```PowerShell
# Sets the MFA requirement state
function Set-MfaState {

    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipelineByPropertyName=$True)]
        $ObjectId,
        [Parameter(ValueFromPipelineByPropertyName=$True)]
        $UserPrincipalName,
        [ValidateSet("Disabled","Enabled","Enforced")]
        $State
    )

    Process {
        Write-Verbose ("Setting MFA state for user '{0}' to '{1}'." -f $ObjectId, $State)
        $Requirements = @()
        if ($State -ne "Disabled") {
            $Requirement =
                [Microsoft.Online.Administration.StrongAuthenticationRequirement]::new()
            $Requirement.RelyingParty = "*"
            $Requirement.State = $State
            $Requirements += $Requirement
        }

        Set-MsolUser -ObjectId $ObjectId -UserPrincipalName $UserPrincipalName `
                     -StrongAuthenticationRequirements $Requirements
    }
}

# Disable MFA for all users
Get-MsolUser -All | Set-MfaState -State Disabled
```

> [!NOTE]
> Recientemente hemos cambiado el comportamiento y el script de PowerShell anterior en consecuencia. Anteriormente, el script se guardaba fuera de los métodos de MFA, se deshabilitó MFA y restauraba los métodos. Ahora, ya no es necesario que el comportamiento predeterminado para deshabilitar no borre los métodos.

## <a name="plan-conditional-access-policies"></a>Planificación de directivas de acceso condicional

Para planear la estrategia de directiva de acceso condicional, que determinará cuándo se solicitan MFA y otros controles, consulte [Directivas de acceso condicional habituales](../conditional-access/concept-conditional-access-policy-common.md).

Es importante que evite bloquearse accidentalmente del inquilino de Azure AD. Puede mitigar el efecto de esta falta involuntaria de acceso administrativo si [crea de dos o más cuentas de acceso de emergencia en su inquilino](../users-groups-roles/directory-emergency-access.md) y no las incluye en su directiva de acceso condicional.

### <a name="create-conditional-access-policy"></a>Creación de una directiva de acceso condicional

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador global.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
   ![Creación de una directiva de acceso condicional que habilite MFA para los usuarios de Azure Portal en el grupo piloto](media/howto-mfa-getstarted/conditionalaccess-newpolicy.png)
1. Escriba un nombre descriptivo para la directiva.
1. En **usuarios y grupos**:
   * En la pestaña **Incluir**, seleccione el botón de radio **Todos los usuarios**.
   * En la pestaña **Excluir**, active la casilla para **Usuarios y grupos** y elija las cuentas de acceso de emergencia.
   * Haga clic en **Done**(Listo).
1. En **Aplicaciones en la nube**, seleccione el botón de selección **Todas las aplicaciones en la nube**.
   * OPCIONAL: En la pestaña **Excluir**, elija aplicaciones en la nube para las que su organización no requiere MFA.
   * Haga clic en **Done**(Listo).
1. En la sección **Condiciones**:
   * OPCIONAL: Si ha habilitado Azure Identity Protection, puede optar por integrar la evaluación de riesgos de inicio de sesión en la directiva.
   * OPCIONAL: Si ha configurado ubicaciones de confianza o ubicaciones con nombre, puede especificar que se incluyan o excluyan esas ubicaciones de la directiva.
1. En **Conceder**, asegúrese de que el botón de selección **Conceder acceso** está seleccionado.
    * Active la casilla **Requerir autenticación multifactor**.
    * Haga clic en **Seleccionar**.
1. Omita la sección **Sesión**.
1. Establezca la opción **Habilitar directiva** en **Activada**.
1. Haga clic en **Crear**.

## <a name="plan-integration-with-on-premises-systems"></a>Planificación de la integración con sistemas locales

Algunas aplicaciones heredadas y locales que no se autentican directamente en Azure AD requieren pasos adicionales para usar MFA, incluidas:

* Aplicaciones heredadas locales, que deben usar Application Proxy.
* Aplicaciones locales de RADIUS, que deben usar el adaptador de MFA con el servidor NPS.
* Aplicaciones locales de AD FS, que deben usar el adaptador de MFA con AD FS 2016 o posterior.

Las aplicaciones que se autentican directamente con Azure AD y tienen autenticación moderna (WS-Fed, SAML, OAuth, OpenID Connect) pueden hacer uso directo de las directivas de acceso condicional.

### <a name="use-azure-mfa-with-azure-ad-application-proxy"></a>Uso de Azure Multi-Factor Authentication con Azure Active Directory Application Proxy

Las aplicaciones que residen en el entorno local se puede publicar en el inquilino de Azure AD a través de [Azure AD Application Proxy](../manage-apps/application-proxy.md) y pueden aprovechar las ventajas de Azure Multi-Factor Authentication si se configuran para usar la autenticación previa de Azure AD.

Estas aplicaciones están sujetas a las directivas de acceso condicional que exigen Azure Multi-Factor Authentication, al igual que cualquier otra aplicación con Azure AD integrado.

Del mismo modo, si se exige el uso de Azure Multi-Factor Authentication para todos los inicios de sesión de los usuarios, las aplicaciones locales que se publiquen con Azure Active Directory Application Proxy estarán protegidas.

### <a name="integrating-azure-multi-factor-authentication-with-network-policy-server"></a>Integración de Azure Multi-Factor Authentication con Servidor de directivas de redes

La extensión de Servidor de directivas de redes (NPS) para Azure MFA agrega funcionalidades de MFA basadas en la nube a la infraestructura de autenticación mediante los servidores existentes. Con la extensión NPS, podrá agregar mecanismos de verificación mediante llamadas de teléfono, mensajes de texto o aplicaciones de teléfono al flujo de autenticación existente. Esta integración tiene las siguientes limitaciones:

* Con el protocolo CHAPv2, solo se admiten las notificaciones push y llamadas de voz de la aplicación autenticadora.
* No se pueden aplicar las directivas de acceso condicional.

La extensión NPS actúa como un adaptador entre RADIUS y Azure MFA en la nube para proporcionar un segundo factor de autenticación a fin de proteger la [VPN](howto-mfa-nps-extension-vpn.md), las [conexiones de puerta de enlace de Escritorio remoto](howto-mfa-nps-extension-rdg.md) u otras aplicaciones con funcionalidad RADIUS. Los usuarios que se registren para Azure MFA en este entorno se enfrentarán al desafío de todos los intentos de autenticación, ya que la falta de directivas de acceso condicional implica que siempre se requiere la autenticación multifactor.

#### <a name="implementing-your-nps-server"></a>Implementación del servidor de NPS

Si tiene una instancia de NPS implementada y está en uso, consulte [Integración de la infraestructura existente de NPS con Azure Multi-Factor Authentication](howto-mfa-nps-extension.md). Si está configurando NPS por primera vez, consulte el artículo [Servidor de directivas de redes (NPS)](/windows-server/networking/technologies/nps/nps-top) para obtener instrucciones. Puede encontrar una guía de solución de problemas en el artículo [Resolución de mensajes de error de la extensión de NPS para Azure Multi-Factor Authentication](howto-mfa-nps-extension-errors.md).

#### <a name="prepare-nps-for-users-that-arent-enrolled-for-mfa"></a>Preparación de los usuarios NPS que no están inscritos en MFA

Elija lo que sucede cuando los usuarios que no están inscritos en MFA intentan autenticarse. Use la configuración del registro `REQUIRE_USER_MATCH` en la ruta de acceso del registro `HKLM\Software\Microsoft\AzureMFA` para controlar el comportamiento de la característica. Esta opción tiene una única opción de configuración.

| Clave | Value | Valor predeterminado |
| --- | --- | --- |
| `REQUIRE_USER_MATCH` | TRUE/FALSE | No establecido (equivalente a TRUE) |

El propósito de esta configuración es determinar qué hacer cuando un usuario no está inscrito en MFA. Los efectos de modificar esta configuración se muestran en la tabla siguiente.

| Configuración | Estado de MFA del usuario | Efectos |
| --- | --- | --- |
| La clave no existe | No inscrito | El desafío de MFA no se realizó correctamente |
| Valor establecido en True o sin establecer | No inscrito | El desafío de MFA no se realizó correctamente |
| Clave establecida en False | No inscrito | Autenticación con MFA |
| Clave establecida en False o True | Inscrito | Debe autenticarse con MFA |

### <a name="integrate-with-active-directory-federation-services"></a>Integración con Servicios de federación de Active Directory (AD FS)

Si su organización está federada con Azure AD, puede usar [Azure Multi-Factor Authentication para proteger los recursos de AD FS](multi-factor-authentication-get-started-adfs.md), tanto en entornos locales como en la nube. Azure MFA le permite reducir las contraseñas y proporciona una manera más segura de autenticarse. A partir de Windows Server 2016, ahora puede configurar Azure MFA como autenticación principal.

A diferencia de AD FS en Windows Server 2012 R2, el adaptador de Azure MFA para AD FS 2016 se integra directamente con Azure AD y no requiere un servidor local de Azure MFA. El adaptador de Azure MFA está integrado en Windows Server 2016 y no es necesario para una instalación adicional.

Cuando se usa Azure MFA con AD FS 2016 y la aplicación de destino está sujeta a la directiva de acceso condicional, debe tener en cuenta consideraciones adicionales:

* El acceso condicional está disponible cuando la aplicación es un usuario de confianza de Azure AD y está federada con AD FS 2016 o posterior.
* El acceso condicional no está disponible cuando la aplicación es un usuario de confianza de AD FS 2016 o AD FS 2019 y está federada o administrada con AD FS 2016 o AD FS 2019.
* Además, el acceso condicional no está disponible cuando AD FS 2016 o AD FS 2019 está configurado para usar Azure MFA como método de autenticación principal.

#### <a name="ad-fs-logging"></a>Registro de AD FS

La escritura de registros estándar de AD FS 2016 y 2019 en el registro de seguridad de Windows y el registro de administración de AD FS contiene información sobre las solicitudes de autenticación y si se realizaron correctamente o no. Los datos de registro de eventos dentro de estos eventos indican si se usó Azure MFA. Por ejemplo, un evento de auditoría de AD FS con ID 1200 puede contener:

```
<MfaPerformed>true</MfaPerformed>
<MfaMethod>MFA</MfaMethod>
```

#### <a name="renew-and-manage-certificates"></a>Renovación y administración de certificados

En todos los servidores de AD FS del equipo local My Store habrá un certificado de Azure MFA autofirmado con el título OU=Microsoft AD FS Azure MFA, que contiene la fecha de expiración del certificado. Compruebe el período de validez de este certificado en cada servidor de AD FS para determinar la fecha de expiración.

Si el período de validez de los certificados está próximo a expirar, [genere y compruebe un nuevo certificado MFA en cada servidor de AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-and-azure-mfa#configure-the-ad-fs-servers).

Las instrucciones siguientes detallan cómo administrar los certificados de Azure MFA en los servidores de AD FS. Cuando se configura AD FS con Azure MFA, los certificados que se generan a través del cmdlet `New-AdfsAzureMfaTenantCertificate` de PowerShell son válidos durante dos años. Renueve e instale los certificados renovados antes de que expiren para evitar interrupciones en el servicio de MFA.

## <a name="implement-your-plan"></a>Implementación del plan

Ahora que ha planeado la solución, puede implementarla mediante los siguientes pasos:

1. Cumpla con los requisitos previos necesarios.
   1. Implemente [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) para los escenarios híbridos.
   1. Implemente [Azure AD Application Proxy](../manage-apps/application-proxy.md) en todas las aplicaciones locales publicadas para el acceso a la nube.
   1. Implemente [NPS](/windows-server/networking/technologies/nps/nps-top) para todas las autenticaciones de RADIUS.
   1. Asegúrese de que los usuarios están actualizados a las versiones compatibles de Microsoft Office con la autenticación moderna habilitada.
1. Configure los [métodos de autenticación](#choose-verification-options) que eligió.
1. Defina las [ubicaciones de red con nombre](../conditional-access/location-condition.md#named-locations).
1. Seleccione grupos para comenzar el lanzamiento de MFA.
1. Configure las [directivas de acceso condicional](#create-conditional-access-policy).
1. Configure la directiva de registro MFA.
   1. [MFA y SSPR combinados](howto-registration-mfa-sspr-combined.md)
   1. Con [Identity Protection](../identity-protection/howto-identity-protection-configure-mfa-policy.md)
1. Envíe las comunicaciones de usuario y haga que los usuarios se inscriban en [https://aka.ms/mfasetup](https://aka.ms/mfasetup).
1. [Realice un seguimiento de quién está inscrito](#identify-non-registered-users).

> [!TIP]
> Los usuarios de la nube de administración pública pueden inscribirse en [https://aka.ms/GovtMFASetup](https://aka.ms/GovtMFASetup)

## <a name="manage-your-solution"></a>Administración de la solución

Informes para Azure MFA

Azure Multi-Factor Authentication proporciona informes a través de Azure Portal:

| Informe | Location | Descripción |
| --- | --- | --- |
| Alertas de fraude y de uso | Azure AD > Inicios de sesión | Proporciona información sobre el uso general, el resumen del usuario, detalles del usuario; así como un historial de alertas de fraude enviadas durante el intervalo de fechas especificado. |

## <a name="troubleshoot-mfa-issues"></a>Solución de problemas de MFA

Busque las soluciones para problemas comunes con Azure MFA en el [artículo de solución de problemas de Azure Multi-Factor Authentication](https://support.microsoft.com/help/2937344/troubleshooting-azure-multi-factor-authentication-issues) en el Centro de soporte de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

Para ver Multi-Factor Authentication de Azure en acción, complete el siguiente tutorial:

> [!div class="nextstepaction"]
> [Habilitación de Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md)
