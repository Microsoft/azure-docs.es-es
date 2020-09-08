---
title: 'Proporcionar funcionalidades de Azure MFA mediante NPS: Azure Active Directory'
description: Agregue funcionalidades de verificación en dos pasos basadas en la nube a su infraestructura de autenticación existente.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 35842520b26d3a98342660244295e26e934e7d3c
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717377"
---
# <a name="integrate-your-existing-nps-infrastructure-with-azure-multi-factor-authentication"></a>Integración de la infraestructura existente de NPS con Azure Multi-Factor Authentication

La extensión de Servidor de directivas de redes (NPS) para Azure MFA agrega funcionalidades de MFA basadas en la nube a la infraestructura de autenticación mediante los servidores existentes. Con la extensión NPS, puede agregar verificación de llamadas de teléfono, mensaje de texto o de aplicaciones de teléfono al flujo de autenticación existente sin tener que instalar, configurar ni mantener servidores nuevos. 

Esta extensión se creó para las organizaciones que desean proteger las conexiones VPN sin tener que implementar el servidor de Azure MFA. La extensión de NPS actúa como un adaptador entre RADIUS y Azure MFA basada en la nube para proporcionar un segundo factor de autenticación para usuarios federados o sincronizados.

Al usar la extensión de NPS para Azure MFA, el flujo de autenticación incluye los siguientes componentes: 

1. **Servidor NAS/VPN** recibe solicitudes de los clientes VPN y las convierte en solicitudes RADIUS para servidores NPS. 
2. **Servidor NPS** se conecta a Active Directory para realizar la autenticación principal para las solicitudes RADIUS y, cuando se realiza correctamente, pasa la solicitud a todas las extensiones instaladas.  
3. **Extensión de NPS** desencadena una solicitud para Azure MFA para la autenticación secundaria. Una vez que la extensión recibe la respuesta y, si el desafío de MFA se realiza correctamente, se completa la solicitud de autenticación proporcionando al servidor NPS tokens de seguridad que incluyen una notificación de MFA, emitidos por Azure STS.  
4. **Azure MFA** se comunica con Azure Active Directory para recuperar los detalles del usuario y realiza la autenticación secundaria con un método de comprobación configurado para el usuario.

El diagrama siguiente ilustra este flujo de solicitud de autenticación de alto nivel: 

![Diagrama de flujo de autenticación](./media/howto-mfa-nps-extension/auth-flow.png)

## <a name="plan-your-deployment"></a>Planeamiento de la implementación

La extensión de NPS controla automáticamente la redundancia, por lo que no necesita ninguna configuración especial.

Puede crear tantos servidores NPS habilitados para Azure MFA como necesite. Si instala varios servidores, debe usar un certificado de cliente distinto para cada uno de ellos. La creación de un certificado para cada servidor significa que puede actualizar individualmente cada certificado y no preocuparse por el tiempo de inactividad en los servidores.

Los servidores VPN enrutan las solicitudes de autenticación, por lo que tienen que ser conscientes de los nuevos servidores NPS habilitados para MFA de Azure.

## <a name="prerequisites"></a>Requisitos previos

La extensión de NPS está diseñada para funcionar con la infraestructura existente. Asegúrese de que cumple los siguientes requisitos previos antes de empezar.

### <a name="licenses"></a>Licencias

La extensión de NPS para Azure MFA está disponible para los clientes con [licencias para Azure Multi-Factor Authentication](./concept-mfa-howitworks.md) (incluida con Azure AD Premium, EMS o una licencia independiente de MFA). Las licencias basadas en el consumo para Azure MFA, como las licencias por usuario o por autenticación no son compatibles con la extensión de NPS. 

### <a name="software"></a>Software

Windows Server 2008 R2 SP1 o superior.

### <a name="libraries"></a>Bibliotecas

Estas bibliotecas se instalan automáticamente con la extensión.

- [Paquetes redistribuibles de Visual C++ para Visual Studio 2013 (X64)](https://www.microsoft.com/download/details.aspx?id=40784)
- [Módulo Microsoft Azure Active Directory para Windows PowerShell versión1.1.166.0](https://www.powershellgallery.com/packages/MSOnline/1.1.166.0)

El módulo Microsoft Azure Active Directory para Windows PowerShell se instala, si todavía no está presente, a través de un script de configuración que se ejecuta como parte del proceso de instalación. No es necesario instalar este módulo con antelación si aún no está instalado.

Debe instalar manualmente la siguiente biblioteca:

- [Visual C++ Redistributable para Visual Studio 2015](https://www.microsoft.com/download/details.aspx?id=48145)

### <a name="azure-active-directory"></a>Azure Active Directory

Todos los usuarios que utilizan la extensión NPS deben estar sincronizados con Azure Active Directory mediante Azure AD Connect y estar registrados en MFA.

Para instalar la extensión, necesita el *identificador de inquilino* y las credenciales de administrador para el inquilino de Azure AD. Para obtener el identificador de inquilino, complete los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global del inquilino de Azure.
1. Busque y seleccione el servicio **Azure Active Directory**.
1. En la página **Información general**, se muestra la *información del inquilino*. Junto al *identificador de inquilino*, seleccione el icono **Copiar**, tal y como se muestra en la siguiente captura de pantalla de ejemplo:

   ![Obtención del identificador de inquilino en Azure Portal](./media/howto-mfa-nps-extension/azure-active-directory-tenant-id-portal.png)

### <a name="network-requirements"></a>Requisitos de red

El servidor NPT necesita poder comunicarse con las siguientes direcciones URL a través de los puertos 80 y 443.

- https:\//adnotifications.windowsazure.com
- https:\//login.microsoftonline.com
- https:\//credentials.azure.com

Además, se requiere conectividad con las direcciones URL siguientes para completar la [instalación del adaptador mediante el script de PowerShell proporcionado](#run-the-powershell-script).

- https:\//login.microsoftonline.com
- https:\//provisioningapi.microsoftonline.com
- https:\//aadcdn.msauth.net

## <a name="prepare-your-environment"></a>Preparación del entorno

Antes de instalar la extensión de NPS, quiere preparar el entorno para controlar el tráfico de autenticación.

### <a name="enable-the-nps-role-on-a-domain-joined-server"></a>Habilitar el rol NPS en un servidor unido a dominio

El servidor NPS se conecta a Azure Active Directory y autentica las solicitudes de MFA. Elija un servidor para este rol. Se recomienda elegir un servidor que no controle solicitudes de otros servicios, puesto que la extensión de NPS produce errores para cualquier solicitud que no sea RADIUS. El servidor NPS debe estar configurado como servidor de autenticación principal y secundario para su entorno; no puede redirigir mediante proxy solicitudes RADIUS a otro servidor.

1. En el servidor, abra el **Asistente para agregar roles y características** desde el menú de inicio rápido del Administrador de servidores.
2. Elija **Instalación basada en características o en roles** como tipo de instalación.
3. Seleccione el rol de servidor **Servicios de acceso y directivas de redes**. Puede emerger una ventana para informarle de las características necesarias para ejecutar este rol.
4. Continúe con los pasos del asistente hasta la página Confirmación. Seleccione **Instalar**.

Ahora que tiene un servidor designado para NPS, también debe configurarlo para administrar las solicitudes RADIUS entrantes de la solución VPN.

### <a name="configure-your-vpn-solution-to-communicate-with-the-nps-server"></a>Configurar la solución VPN para comunicarse con el servidor NPS

Según la solución VPN que use, así varían los pasos para configurar la directiva de autenticación RADIUS. Configure esta directiva de modo que apunte al servidor NPS RADIUS.

### <a name="sync-domain-users-to-the-cloud"></a>Sincronizar los usuarios del dominio en la nube

Puede que este paso ya se haya completado en el inquilino, pero es conveniente comprobar que Azure AD Connect ha sincronizado las bases de datos recientemente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador.
2. Seleccione **Azure Active Directory** > **Azure AD Connect**.
3. Compruebe que el estado de sincronización es **Habilitado** y que la última sincronización fue hace menos de una hora.

Si necesita iniciar una nueva ronda de sincronización, use las instrucciones de [Azure AD Connect Sync: Scheduler](../hybrid/how-to-connect-sync-feature-scheduler.md#start-the-scheduler).

### <a name="determine-which-authentication-methods-your-users-can-use"></a>Determinación de los métodos de autenticación que pueden utilizar los usuarios

Existen dos factores que afectan a la disponibilidad de los métodos de autenticación con una implementación de extensión NPS:

1. El algoritmo de cifrado de contraseña utilizado entre el cliente RADIUS (VPN, servidor Netscaler u otros) y los servidores NPS.
   - **PAP** es compatible con todos los métodos de autenticación de Azure MFA en la nube: llamada de teléfono, mensaje de texto unidireccional, notificación de aplicación móvil, token de hardware OATH y código de verificación de la aplicación móvil.
   - **CHAPV2** y **EAP** admiten llamadas de teléfono y notificaciones de aplicación móvil.

      > [!NOTE]
      > Al implementar la extensión NPS, use estos factores para estudiar qué métodos están disponibles para los usuarios. Si el cliente RADIUS es compatible con PAP, pero el cliente UX no tiene campos de entrada para un código de verificación, las dos opciones compatibles son la llamada de teléfono y notificación de aplicación móvil.
      >
      > Además, si la experiencia de usuario del cliente de VPN es compatible con el campo de entrada y ha configurado la directiva de acceso de red, la autenticación puede realizarse correctamente, pero ninguno de los atributos de RADIUS configurados en la directiva de red se aplicará al dispositivo de acceso a la red (como el servidor RRAS) ni al cliente de VPN. Como resultado, el cliente de VPN puede tener más acceso del deseado o menos o ninguno.
      >

2. Los métodos de entrada que la aplicación cliente (VPN, servidor Netscaler u otros) puede controlar. Por ejemplo, ¿tiene el cliente de VPN algún medio para que el usuario escriba un código de comprobación de aplicación móvil o de texto?

Puede [deshabilitar los métodos de autenticación no compatibles](howto-mfa-mfasettings.md#verification-methods) en Azure.

### <a name="register-users-for-mfa"></a>Registro de usuarios en MFA

Antes de implementar y utilizar la extensión NPS, es necesario que los usuarios a los que se les pide la verificación en dos pasos estén registrados en MFA. De forma más inmediata, para probar la extensión mientras se implementa, necesita al menos una cuenta de prueba totalmente registrada para la autenticación multifactor.

Para iniciar una cuenta de prueba, use estos pasos:

1. Inicie sesión en [https://aka.ms/mfasetup](https://aka.ms/mfasetup) con una cuenta de prueba.
2. Siga las indicaciones para configurar un método de verificación.
3. [Cree una directiva de acceso condicional](howto-mfa-getstarted.md#create-conditional-access-policy) para requerir la autenticación multifactor para la cuenta de pruebas.

> [!IMPORTANT]
>
> Asegúrese de que los usuarios se han registrado correctamente para Azure Multi-Factor Authentication. Si los usuarios solo se han registrado previamente para el autoservicio de restablecimiento de contraseña (SSPR), *StrongAuthenticationMethods* está habilitado para su cuenta. Azure Multi-Factor Authentication se aplica cuando se configura *StrongAuthenticationMethods*, aunque el usuario solo esté registrado para SSPR.
>
> Se puede habilitar el registro de seguridad combinado que configura SSPR y Azure Multi-Factor Authentication al mismo tiempo. Para obtener más información, consulte [Habilitación del registro de información de seguridad combinado en Azure Active Directory](howto-registration-mfa-sspr-combined.md).
>
> También puede [obligar a los usuarios a volver a registrar los métodos de autenticación](howto-mfa-userdevicesettings.md#manage-user-authentication-options) si previamente solo habilitaron SSPR.

## <a name="install-the-nps-extension"></a>Instalación de la extensión de NPS

> [!IMPORTANT]
> Instale la extensión NPS en un servidor diferente al punto de acceso VPN.

### <a name="download-and-install-the-nps-extension-for-azure-mfa"></a>Descargar e instalar la extensión de NPS para Azure MFA

1. [Descargue la extensión de NPS](https://aka.ms/npsmfa) desde el Centro de descarga de Microsoft.
2. Copie el archivo binario en el Servidor de directivas de redes que quiere configurar.
3. Ejecute *setup.exe* y siga las instrucciones de instalación. Si se producen errores, compruebe que las dos bibliotecas de la sección de requisitos previos se han instalado correctamente.

#### <a name="upgrade-the-nps-extension"></a>Actualización de la extensión NPS

Cuando se actualiza una instalación de una extensión NPS existente, realice los pasos siguientes para evitar un reinicio del servidor subyacente:

1. Desinstalación de la versión existente
1. Ejecución del nuevo instalador
1. Reinicio del servicio del Servidor de directivas de redes (IAS)

### <a name="run-the-powershell-script"></a>Ejecución del script de PowerShell

El instalador crea un script de PowerShell en esta ubicación: `C:\Program Files\Microsoft\AzureMfa\Config` (donde C:\ es la unidad de instalación). Este script de PowerShell realiza las siguientes acciones cada vez que se ejecuta:

- Creación de un certificado autofirmado.
- Asociación de la clave pública del certificado a la entidad de servicio en Azure AD.
- Almacenamiento del certificado en el almacén de certificados del equipo local.
- Concesión de acceso a la clave privada del certificado para el usuario de red.
- Reinicio de NPS.

A menos que desee utilizar sus propios certificados (en lugar de los certificados autofirmados que genera el script de PowerShell), ejecute el script de PowerShell para completar la instalación. Si instala la extensión en varios servidores, cada uno debe tener su propio certificado.

1. Ejecute Windows PowerShell como administrador.
2. Cambie los directorios.

   ```powershell
   cd "C:\Program Files\Microsoft\AzureMfa\Config"
   ```

3. Ejecute el script de PowerShell creado por el instalador.

   > [!IMPORTANT]
   > En el caso de los clientes que usan Azure Government o las nubes de Azure China 21Vianet, edite primero los cmdlets `Connect-MsolService` en el script *AzureMfaNpsExtnConfigSetup. PS1* para incluir los parámetros *AzureEnvironment* para la nube requerida. Por ejemplo, especifique *-AzureEnvironment USGovernment* o *-AzureEnvironment AzureChinaCloud*.
   >
   > Para más información, consulte la [referencia de parámetros de Connect-MsolService](/powershell/module/msonline/connect-msolservice#parameters).

   ```powershell
   .\AzureMfaNpsExtnConfigSetup.ps1
   ```

4. Inicie sesión como administrador en Azure AD.
5. PowerShell le pide el identificador de inquilino. Use el GUID del *identificador de inquilino* que ha copiado desde Azure Portal en la sección de requisitos previos.
6. PowerShell muestra un mensaje de operación correcta cuando finaliza el script.  

Repita estos pasos en todos los servidores NPS adicionales que desee configurar para equilibrio de carga.

Si ha expirado el certificado de equipo anterior y se ha generado un nuevo certificado, debe eliminar todos los certificados expirados. Tener los certificados expirados puede provocar incidencias con el inicio de la extensión NPS.

> [!NOTE]
> Si utiliza certificados propios en lugar de generar certificados con el script de PowerShell, asegúrese de que se ajustan a la convención de nomenclatura de NPS. El nombre del firmante debe ser **CN=\<TenantID\>,OU=Microsoft NPS Extension**.

### <a name="microsoft-azure-government-or-azure-china-21vianet-additional-steps"></a>Pasos adicionales de Microsoft Azure Government o de Azure China 21Vianet

En el caso de los clientes que usan las nubes de Azure Government o Azure China 21Vianet, se necesitan los siguientes pasos de configuración adicionales en cada servidor NPS.

> [!IMPORTANT]
> Configure estos valores del Registro solo si es cliente de Azure Government o de Azure China 21Vianet.

1. Si es cliente de Azure Government o de Azure China 21Vianet, abra el **Editor del Registro** en el servidor NPS.
1. Vaya a `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa`.
1. Para clientes de Azure Government, establezca los siguientes valores de clave:

    | Clave del Registro       | Value |
    |--------------------|-----------------------------------|
    | AZURE_MFA_HOSTNAME | adnotifications.windowsazure.us   |
    | STS_URL            | https://login.microsoftonline.us/ |

1. En el caso de los clientes de Azure China 21Vianet, establezca los siguientes valores de clave:

    | Clave del Registro       | Value |
    |--------------------|-----------------------------------|
    | AZURE_MFA_HOSTNAME | adnotifications.windowsazure.cn   |
    | STS_URL            | https://login.chinacloudapi.cn/   |

1. Repita los dos pasos anteriores para establecer los valores de clave del Registro para cada servidor NPS.
1. Reinicie el servicio NPS para cada servidor NPS.

    Para reducir el impacto, retire cada servidor NPS de la rotación de NLB de uno en uno y espere a que se purguen todas las conexiones.

### <a name="certificate-rollover"></a>Sustitución del certificado

Desde la versión 1.0.1.32 de la extensión NPS se admite la lectura de varios certificados. Esta funcionalidad le ayudará a facilitar las actualizaciones graduales de certificados antes de su expiración. Si en la organización se ejecuta una versión anterior de la extensión NPS, debe actualizar a la versión 1.0.1.32 o superior.

Los certificados creados por el script `AzureMfaNpsExtnConfigSetup.ps1` son válidos durante 2 años. Las organizaciones de TI deben supervisar la expiración de los certificados. Los certificados para la extensión NPS se encuentran en el almacén de certificados del equipo local, en la carpeta Personal y se emiten para el identificador de inquilino proporcionado al script.

Cuando un certificado se aproxima a la fecha de expiración, se debe crear un nuevo certificado para reemplazarlo.  Este proceso se realiza de nuevo mediante la ejecución del script `AzureMfaNpsExtnConfigSetup.ps1` y se debe mantener el mismo identificador de inquilino cuando se le solicite. Este proceso se debe repetir en cada servidor de NPS del entorno.

## <a name="configure-your-nps-extension"></a>Configuración de la extensión de NPS

En esta sección se incluyen consideraciones de diseño y sugerencias para las implementaciones correctas de la extensión de NPS.

### <a name="configuration-limitations"></a>Limitaciones de configuración

- La extensión de NPS para Azure MFA no incluye herramientas para migrar usuarios y configuraciones desde el servidor MFA a la nube. Por este motivo se recomienda usar la extensión para las nuevas implementaciones en lugar de la existente. Si usa la extensión en una implementación existente, los usuarios tendrán que autenticarse de nuevo para rellenar los detalles de MFA en la nube.  
- La extensión de NPS usa UPN de la instancia de Active Directory local para identificar al usuario en Azure MFA que realiza la autenticación secundaria. La extensión se puede configurar para que utilice un identificador diferente como identificador de inicio de sesión alternativo o un campo personalizado de Active Directory que no sea UPN. Consulte el artículo de [opciones de configuración avanzada para la extensión NPS para Multi-Factor Authentication](howto-mfa-nps-extension-advanced.md) para más información.
- No todos los protocolos de cifrado son compatibles con todos los métodos de comprobación.
   - **PAP** admite llamadas de teléfono, mensajes de texto unidireccionales, notificaciones de aplicación móvil y códigos de comprobación de aplicación móvil
   - **CHAPV2** y **EAP** admiten llamadas de teléfono y notificaciones de aplicación móvil.

### <a name="control-radius-clients-that-require-mfa"></a>Control de clientes RADIUS que requieren MFA

Una vez que habilite MFA para un cliente RADIUS mediante la extensión de NPS, son necesarias todas las autenticaciones de este cliente para ejecutar MFA. Si desea habilitar MFA para algunos clientes RADIUS pero no para otros, puede configurar dos servidores NPS e instalar la extensión solo en uno de ellos. Configure los clientes RADIUS para los que desea que se exija MFA para enviar solicitudes en el servidor NPS configurado con la extensión y otros clientes RADIUS en el servidor NPS no configurado con la extensión.

### <a name="prepare-for-users-that-arent-enrolled-for-mfa"></a>Preparación para los usuarios que no están inscritos en MFA

Si tiene usuarios que no están inscritos en MFA, puede determinar lo que sucederá cuando intenten realizar la autenticación. Use la configuración de registro *REQUIRE_USER_MATCH* en la ruta de acceso del registro *HKLM\Software\Microsoft\AzureMFA* para controlar el comportamiento de la característica. Esta opción tiene una única opción de configuración:

| Clave | Value | Valor predeterminado |
| --- | ----- | ------- |
| REQUIRE_USER_MATCH | TRUE/FALSE | No establecido (equivalente a TRUE) |

El propósito de esta configuración es determinar qué hacer cuando un usuario no está inscrito en MFA. Cuando la clave no existe, no está establecida o está establecida en TRUE, y el usuario no está inscrito, se produce un error en la extensión al completar el desafío de MFA. Cuando la clave está establecida en FALSE y el usuario no está inscrito, la autenticación continúa sin realizar el desafío de MFA. Si un usuario está inscrito en MFA, debe autenticarse con MFA aunque REQUIRE_USER_MATCH esté establecido en FALSE.

Puede crear esta clave y establecerla en FALSE mientras los usuarios se incorporan, y puede que no todos estén inscritos en Azure MFA todavía. No obstante, puesto que el establecimiento de la clave permite a los usuarios que no estén inscritos en MFA iniciar sesión, debe quitar esta clave antes de pasar a producción.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="nps-extension-health-check-script"></a>Script de comprobación del estado de la extensión NPS

El siguiente script está disponible para realizar los pasos básicos de comprobación del estado al momento de solucionar problemas de la extensión de NPS.

[MFA_NPS_Troubleshooter.ps1](/samples/azure-samples/azure-mfa-nps-extension-health-check/azure-mfa-nps-extension-health-check/)

---

### <a name="how-do-i-verify-that-the-client-cert-is-installed-as-expected"></a>¿Cómo se puede comprobar que el certificado de cliente está instalado según lo previsto?

Busque el certificado autofirmado creado por el instalador en el almacén de certificados y compruebe que la clave privada tenga permisos concedidos al usuario **NETWORK SERVICE**. El certificado tiene el nombre de firmante de **CN \<tenantid\>, OU = Microsoft NPS Extension**.

Los certificados autofirmados generados por el script *AzureMfaNpsExtnConfigSetup.ps1* también tienen una vigencia de validez de dos años. Al comprobar que el certificado está instalado, también debe comprobar que el certificado no ha expirado.

---

### <a name="how-can-i-verify-that-my-client-cert-is-associated-to-my-tenant-in-azure-active-directory"></a>¿Cómo se puede comprobar que el certificado de cliente está asociado a mi inquilino en Azure Active Directory?

Abra el símbolo del sistema de PowerShell y ejecute el comando siguiente:

```powershell
import-module MSOnline
Connect-MsolService
Get-MsolServicePrincipalCredential -AppPrincipalId "981f26a1-7f43-403b-a875-f8b09b8cd720" -ReturnKeyValues 1
```

Estos comandos imprimen todos los certificados que asocian a su inquilino con su instancia de la extensión de NPS en la sesión de PowerShell. Busque el certificado al exportar el certificado de cliente como un archivo "X.509 codificado base 64 (.cer)" sin la clave privada y compárelo con la lista de PowerShell.

El comando siguiente creará un archivo denominado "npscertificate" en la unidad "C:" con el formato .cer.

```powershell
import-module MSOnline
Connect-MsolService
Get-MsolServicePrincipalCredential -AppPrincipalId "981f26a1-7f43-403b-a875-f8b09b8cd720" -ReturnKeyValues 1 | select -ExpandProperty "value" | out-file c:\npscertificate.cer
```

Una vez que ejecute este comando, vaya a la unidad C, busque el archivo y haga doble clic en él. Vaya a los detalles y desplácese hacia abajo hasta llegar a la "huella digital" y compárela con la huella digital del certificado instalado en el servidor. Las huellas digitales del certificado deben coincidir.

Las marcas de tiempo válido-desde y válido-hasta, que se encuentran en un formato legible, se pueden utilizar para filtrar desajustes evidentes si el comando devuelve más de un certificado.

---

### <a name="why-cannot-i-sign-in"></a>¿Por qué no puedo iniciar sesión?

Compruebe que la contraseña no ha expirado. La extensión NPS no admite el cambio de contraseñas como parte del flujo de trabajo del inicio de sesión. Póngase en contacto con el personal de TI de la organización para obtener más ayuda.

---

### <a name="why-are-my-requests-failing-with-adal-token-error"></a>¿Por qué las solicitudes provocan un error de token de ADAL?

Este error puede deberse a varias razones. Siga estos pasos para poder resolver el problema:

1. Reinicie el servidor NPS.
2. Compruebe que el certificado de cliente esté instalado según lo previsto.
3. Compruebe que el certificado esté asociado a su inquilino en Azure AD.
4. Compruebe que `https://login.microsoftonline.com/` es accesible desde el servidor que ejecuta la extensión.

---

### <a name="why-does-authentication-fail-with-an-error-in-http-logs-stating-that-the-user-is-not-found"></a>¿Por qué produce un error de autenticación en los registros HTTP que indica que no se encuentra el usuario?

Compruebe que se esté ejecutando AD Connect y que el usuario esté presente en Windows Active Directory y Azure Active Directory.

---

### <a name="why-do-i-see-http-connect-errors-in-logs-with-all-my-authentications-failing"></a>¿Por qué aparecen errores de conexión de HTTP en los registros con errores en todas las autenticaciones?

Compruebe que https://adnotifications.windowsazure.com es accesible desde el servidor que ejecuta la extensión NPS.

---

### <a name="why-is-authentication-not-working-despite-a-valid-certificate-being-present"></a>¿Por qué la autenticación no funciona a pesar de tener un certificado válido?

Si ha expirado el certificado de equipo anterior y se ha generado un nuevo certificado, debe eliminar todos los certificados expirados. Tener los certificados expirados puede provocar incidencias con el inicio de la extensión NPS.

Para comprobar si tiene un certificado válido, compruebe el almacén de certificados de la cuenta del equipo local mediante el uso de MMC y asegúrese de que el certificado no ha superado la fecha de expiración. Para generar un nuevo certificado válido, vuelva a ejecutar los pasos descritos en la sección "[Ejecución del script de PowerShell](#run-the-powershell-script)".

## <a name="managing-the-tlsssl-protocols-and-cipher-suites"></a>Administración de los protocolos TLS/SSL y conjuntos de cifrado

Se recomienda deshabilitar o quitar los conjuntos de cifrado más antiguos o débiles a menos que los exija la organización. Se puede encontrar información sobre cómo completar esta tarea en el artículo sobre [Administración de conjuntos de cifrado y protocolos SSL/TLS de AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs).

### <a name="additional-troubleshooting"></a>Solución de problemas adicional

Puede encontrar una guía de solución de problemas adicional y posibles soluciones en el artículo [Resolución de mensajes de error de la extensión NPS para Azure Multi-Factor Authentication](howto-mfa-nps-extension-errors.md).

## <a name="next-steps"></a>Pasos siguientes

- [Información general y configuración del servidor de directivas de redes en Windows Server](/windows-server/networking/technologies/nps/nps-top)

- Configurar los identificadores alternativos de inicio de sesión o una lista de excepciones para las direcciones IP que no deben realizar la comprobación de dos pasos en [Opciones de configuración avanzada para la extensión NPS para Multi-Factor Authentication](howto-mfa-nps-extension-advanced.md)

- Obtenga información sobre cómo integrar [Puerta de enlace de Escritorio remoto](howto-mfa-nps-extension-rdg.md) y los [servidores VPN](howto-mfa-nps-extension-vpn.md) con la extensión NPS

- [Resolución de mensajes de error de la extensión de NPS para Azure Multi-Factor Authentication](howto-mfa-nps-extension-errors.md)