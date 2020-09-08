---
title: Configuración de la unión a Azure Active Directory híbrido para dominios federados | Microsoft Docs
description: Aprenda a configurar la unión a Azure Active Directory híbrido para dominios federados.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 05/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3a37353615e35cd75c126c268de71d10077a9071
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268441"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-federated-domains"></a>Tutorial: Configuración de dispositivos híbridos unidos a Azure Active Directory para dominios federados

Al igual que un usuario de su organización, un dispositivo es una identidad principal que desea proteger. Puede usar una identidad de dispositivo para proteger los recursos en cualquier momento y ubicación. Para lograr este objetivo, puede llevar las identidades de los dispositivos a Azure Active Directory (Azure AD) y administrarlas ahí mediante uno de los métodos siguientes:

- Unión a Azure AD
- Unión a Azure AD híbrido
- Registro de Azure AD

Al traer sus dispositivos a Azure AD, está maximizando la productividad de los usuarios mediante un inicio de sesión único (SSO) en los recursos de nube y del entorno local. Puede proteger el acceso a los recursos de nube y del entorno local con [acceso condicional](../conditional-access/howto-conditional-access-policy-compliant-device.md) al mismo tiempo.

Un entorno federado debe tener un proveedor de identidades que admita los requisitos siguientes. Si tiene un entorno federado en que se utilizan los Servicios de federación de Active Directory (AD FS), ya se admiten los requisitos anteriores.

- **Notificación WIAORMULTIAUTHN:** Esta notificación es necesaria para realizar la unión a Azure AD híbrido para dispositivos Windows de nivel inferior.
- **Protocolo WS-Trust:** Este protocolo es necesario para autenticar en Azure AD los dispositivos Windows actuales unidos a Azure AD híbrido.
  Cuando use AD FS, debe habilitar los siguientes puntos de conexión de WS-Trust: `/adfs/services/trust/2005/windowstransport`
   `/adfs/services/trust/13/windowstransport`
   `/adfs/services/trust/2005/usernamemixed`
   `/adfs/services/trust/13/usernamemixed`
   `/adfs/services/trust/2005/certificatemixed`
   `/adfs/services/trust/13/certificatemixed` 

> [!WARNING] 
> Tanto **adfs/services/trust/2005/windowstransport** como **adfs/services/trust/13/windowstransport** se deben habilitar como puntos de conexión accesibles desde la intranet y NO deben exponerse como accesible desde la extranet mediante el Proxy de aplicación web. Para más información sobre cómo deshabilitar los puntos de conexión de Windows de WS-Trust, consulte [Deshabilitar los puntos de conexión de Windows de WS-Trust en el proxy](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#disable-ws-trust-windows-endpoints-on-the-proxy-ie-from-extranet). Para ver qué puntos de conexión están habilitados, vaya a **Servicio** > **Puntos de conexión** en la consola de administración de AD FS.

En este tutorial, aprenderá a configurar la unión de Azure AD híbrido para dispositivos de equipos unidos a un dominio de Active Directory en un entorno federado mediante AD FS.

Aprenderá a:

> [!div class="checklist"]
> * Configuración de la unión a Azure AD híbrido
> * Habilitación de dispositivos de Windows de nivel inferior
> * Comprobación del registro
> * Solución de problemas

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial se da por supuesto que está familiarizado con estos artículos:

- [¿Qué es una identidad de dispositivo?](overview.md)
- [Planeación de la implementación de la unión a Azure AD híbrido](hybrid-azuread-join-plan.md)
- [Validación controlada de la unión a Azure AD híbrido](hybrid-azuread-join-control.md)

Para configurar el escenario de este tutorial necesita:

- Windows Server 2012 R2 con AD FS
- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594), versión 1.1.819.0 o posterior

Desde la versión 1.1.819.0, Azure AD Connect incluye un asistente que puede utilizar para configurar la unión a Azure AD híbrido. El asistente simplifica considerablemente el proceso de configuración. El asistente relacionado:

- Configura los puntos de conexión de servicio (SCP) para el registro de dispositivos
- Hace una copia de seguridad de la relación de confianza para usuario autenticado de Azure AD existente
- Actualiza las reglas de notificación en la confianza de Azure AD

Los pasos de configuración en este artículo se basan en el uso del asistente de Azure AD Connect. Si tiene una versión anterior de Azure AD Connect instalada, tiene que actualizarla a 1.1.819 o posterior para utilizar el asistente. Si no le es posible instalar la versión más reciente de Azure AD Connect, consulte [cómo configurar manualmente la unión a Azure AD híbrido](hybrid-azuread-join-manual.md).

La unión a Azure AD híbrido requiere que los dispositivos tengan acceso a los siguientes recursos de Microsoft desde dentro de la red de su organización:  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- El servicio de token de seguridad (STS) de su organización (para dominios federados)
- `https://autologon.microsoftazuread-sso.com` (si usa o planea usar SSO de conexión directa)

> [!WARNING]
> Si su organización usa servidores proxy que interceptan el tráfico SSL en escenarios como la prevención de pérdida de datos o las restricciones de inquilino de Azure AD, asegúrese de que el tráfico a "https://device.login.microsoftonline.com" se excluya de la interrupción e inspección de TLS. Si no excluye "https://device.login.microsoftonline.com", pueden surgir interferencias con la autenticación de certificados de cliente, lo que causaría problemas con el registro de dispositivos y el acceso condicional basado en dispositivos.

Desde la versión Windows 10 1803, si se produce un error en la unión instantánea a Azure AD híbrido en un entorno federado mediante AD FS, contamos con Azure AD Connect para sincronizar el objeto de equipo en Azure AD que se usa posteriormente para completar el registro de dispositivos para la unión a Azure AD híbrido. Compruebe que Azure AD Connect ha sincronizado con Azure AD los objetos de equipo de los dispositivos que desea que estén unidos a Azure AD híbrido. Si los objetos de equipo pertenecen a unidades organizativas (OU) específicas, debe configurarlas también para su sincronización en Azure AD Connect. Para más información acerca de cómo sincronizar objetos de equipo con Azure AD Connect, consulte el artículo [Configuración del filtrado con Azure AD Connect](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

Si su organización necesita acceso a Internet mediante un proxy de salida, Microsoft recomienda [implementar la detección automática de proxy web (WPAD)](/previous-versions/tn-archive/cc995261(v%3dtechnet.10)) para permitir que los equipos con Windows 10 realicen el registro de dispositivos con Azure AD. Si tiene problemas de configuración y administración de WPAD, consulte [Troubleshoot automatic detection](/previous-versions/tn-archive/cc302643(v=technet.10)) (Solución de problemas de detección automática). 

Si no usa WPAD y desea configurar el proxy en el equipo, puede hacerlo desde la versión Windows 10 1709. Para más información, consulte [Configure WinHTTP settings using a group policy object (GPO)](/archive/blogs/netgeeks/winhttp-proxy-settings-deployed-by-gpo) [Configuración de WinHTTP mediante un objeto de directiva de grupo (GPO)].

> [!NOTE]
> Si configura el proxy en el equipo mediante WinHTTP, todos los equipos que no se puedan conectar al proxy configurado no podrán conectarse a Internet.

Si su organización requiere acceso a Internet mediante un servidor proxy saliente autenticado, tiene que asegurarse de que los equipos de Windows 10 pueden autenticarse correctamente en el proxy de salida. Debido a que los equipos de Windows 10 ejecutan el registro de dispositivos utilizando el contexto del equipo, debe configurar la autenticación del proxy de salida mediante el contexto del equipo. Realice un seguimiento con su proveedor de proxy de salida en relación a los requisitos de configuración.

Para comprobar si el dispositivo puede acceder a los recursos de Microsoft anteriores en la cuenta del sistema, puede usar el script para [probar la conectividad del Registro de dispositivos](https://gallery.technet.microsoft.com/Test-Device-Registration-3dc944c0).

## <a name="configure-hybrid-azure-ad-join"></a>Configuración de la unión a Azure AD híbrido

Para configurar una unión a Azure AD híbrido mediante Azure AD Connect, necesita:

- Las credenciales de administrador global para el inquilino de Azure AD  
- Las credenciales de administrador de empresa para cada uno de los bosques
- Las credenciales del administrador de AD FS

**Para configurar una unión a Azure AD híbrido mediante Azure AD Connect:**

1. Inicie Azure AD Connect y, a continuación, seleccione **Configurar**.

   ![Bienvenido](./media/hybrid-azuread-join-federated-domains/11.png)

1. En la página **Tareas adicionales**, seleccione **Configurar opciones de dispositivo** y luego **Siguiente**.

   ![Tareas adicionales](./media/hybrid-azuread-join-federated-domains/12.png)

1. En la página **Información general**, seleccione **Siguiente**.

   ![Información general](./media/hybrid-azuread-join-federated-domains/13.png)

1. En la página **Conectar con Azure AD**, proporcione las credenciales de administrador global para el inquilino de Azure AD y seleccione **Siguiente**.

   ![Conectarse a Azure](./media/hybrid-azuread-join-federated-domains/14.png)

1. En la página **Opciones de dispositivos**, seleccione **Configurar la combinación de Azure AD híbrido** y después **Siguiente**.

   ![Opciones de dispositivos](./media/hybrid-azuread-join-federated-domains/15.png)

1. En la página **SCP**, complete los pasos siguientes y seleccione **Siguiente**:

   ![SCP](./media/hybrid-azuread-join-federated-domains/16.png)

   1. Seleccione el bosque.
   1. Seleccione el servicio de autenticación. Debe seleccionar un **servidor de AD FS**, salvo que su organización tenga exclusivamente clientes de Windows 10 y haya configurado la sincronización de equipos o dispositivos, o bien su organización utilice SSO de conexión directa.
   1. Seleccione **Agregar** para especificar las credenciales de administrador de empresa.

1. En la página **Sistemas operativos de los dispositivos**, seleccione los sistemas operativos que los dispositivos en el entorno de Active Directory utilizan y luego **Siguiente**.

   ![Sistema operativos del dispositivo](./media/hybrid-azuread-join-federated-domains/17.png)

1. En la página **Configuración de federación**, especifique las credenciales de su administrador de AD FS y después seleccione **Siguiente**.

   ![Configuración de federación](./media/hybrid-azuread-join-federated-domains/18.png)

1. En la página **Listo para configurar**, seleccione **Configurar**.

   ![Listo para configurar](./media/hybrid-azuread-join-federated-domains/19.png)

1. En la página **Configuración completa**, seleccione **Salir**.

   ![Configuración completada](./media/hybrid-azuread-join-federated-domains/20.png)

## <a name="enable-windows-downlevel-devices"></a>Habilitación de dispositivos de Windows de nivel inferior

Si algunos de los dispositivos unidos a un dominio son dispositivos de Windows de nivel inferior, tiene que:

- Configurar los valores de la intranet local para el registro de dispositivos
- Instalar Microsoft Workplace Join for Windows en equipos de nivel inferior

> [!NOTE]
> La compatibilidad con Windows 7 finalizó el 14 de enero de 2020. Para más información, consulte el artículo sobre el [fin de la compatibilidad con Windows 7](https://support.microsoft.com/en-us/help/4057281/windows-7-support-ended-on-january-14-2020).

### <a name="configure-the-local-intranet-settings-for-device-registration"></a>Configurar los valores de la intranet local para el registro de dispositivos

Para completar con éxito la unión de Azure AD híbrido de los dispositivos de Windows de nivel inferior y para evitar las peticiones de certificados cuando los dispositivos se autentican en Azure AD, puede insertar una directiva en los dispositivos unidos a un dominio para agregar las siguientes direcciones URL a la zona de intranet local en Internet Explorer:

- `https://device.login.microsoftonline.com`
- STS de su organización (para dominios federados)
- `https://autologon.microsoftazuread-sso.com` (para SSO de conexión directa)

También tiene que habilitar **Permitir actualizaciones en la barra de estado a través de script** en la zona de intranet local del usuario.

### <a name="install-microsoft-workplace-join-for-windows-downlevel-computers"></a>Instalar Microsoft Workplace Join for Windows en equipos de nivel inferior

Para registrar dispositivos de nivel inferior de Windows, las organizaciones deben instalar [Microsoft Workplace Join para equipos sin Windows 10](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join para equipos sin Windows 10 está disponible en el Centro de descarga de Microsoft.

El paquete se puede implementar mediante un sistema de distribución de software como  [Microsoft Endpoint Configuration Manager](/configmgr/). El paquete admite las opciones de instalación silenciosa estándar mediante el parámetro `quiet`. La rama actual de Configuration Manager ofrece ventajas adicionales sobre las versiones anteriores, como la posibilidad de realizar el seguimiento de los registros completados.

El instalador crea una tarea programada en el sistema que se ejecuta en el contexto del usuario. La tarea se desencadena cuando el usuario inicia sesión en Windows. La tarea une de forma silenciosa el dispositivo con Azure AD mediante las credenciales de usuario después de que se autentique con Azure AD.

## <a name="verify-the-registration"></a>Comprobación del registro

A continuación se muestran tres maneras de buscar y comprobar el estado del dispositivo:

### <a name="locally-on-the-device"></a>De forma local en el dispositivo

1. Abra Windows PowerShell.
2. Escriba `dsregcmd /status`.
3. Compruebe que tanto **AzureAdJoined** como **DomainJoined** estén establecidos en **SÍ**.
4. Puede utilizar **DeviceId** y comparar el estado del servicio mediante Azure Portal o PowerShell.

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

1. Vaya a la página de dispositivos mediante un [vínculo directo](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices).
2. Puede encontrar información acerca de cómo buscar un dispositivo en el artículo sobre [cómo administrar identidades de dispositivo mediante Azure Portal](./device-management-azure-portal.md).
3. Si la columna **Registrado** indica **Pendiente**, entonces Unión a Azure AD híbrido no se ha completado. En entornos federados, esto solo puede ocurrir si no se ha podido realizar el registro y AAD Connect está configurado para sincronizar los dispositivos.
4. Si la columna **Registrado** contiene una **fecha y hora**, entonces Unión a Azure AD híbrido se ha completado.

### <a name="using-powershell"></a>Usar PowerShell

Compruebe el estado de registro del dispositivo en el inquilino de Azure mediante **[Get-MsolDevice](/powershell/module/msonline/get-msoldevice)** . Este cmdlet se encuentra en el [módulo de PowerShell de Azure Active Directory](/powershell/azure/active-directory/install-msonlinev1?view=azureadps-2.0).

Cuando utiliza el cmdlet **Get-MSolDevice** para comprobar los detalles de servicio:

- Tiene que existir un objeto con el **Identificador de dispositivo** que coincida con el identificador en el cliente de Windows.
- El valor de **DeviceTrustType** es **Unido a dominio**. Este valor es equivalente al estado **Hybrid Azure AD joined** en la página **Dispositivos** del portal de Azure AD.
- En el caso de los dispositivos que se usan en el acceso condicional, el valor de **Habilitado** es **True** y el de **DeviceTrustLevel** es **Administrado**.

1. Abra Windows PowerShell como administrador.
2. Escriba `Connect-MsolService` para conectarse a su inquilino de Azure.

#### <a name="count-all-hybrid-azure-ad-joined-devices-excluding-pending-state"></a>Recuento de todos los dispositivos unidos a Azure AD híbrido (excepto el estado **Pendiente**)

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="count-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Recuento de todos los dispositivos unidos a Azure AD híbrido con el estado **Pendiente**

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="list-all-hybrid-azure-ad-joined-devices"></a>Lista de todos los dispositivos unidos a Azure AD híbrido

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Lista de todos los dispositivos unidos a Azure AD híbrido con el estado **Pendiente**

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-details-of-a-single-device"></a>Lista de detalles de un solo dispositivo:

1. Escriba `get-msoldevice -deviceId <deviceId>` (es el valor **DeviceId** obtenido localmente en el dispositivo).
2. Compruebe que **Habilitado** está establecido en **True**.

## <a name="troubleshoot-your-implementation"></a>Solución de problemas de la implementación

Si tiene problemas para completar la unión a Azure AD híbrido para los dispositivos de Windows unidos a un dominio, consulte:

- [Solución de problemas de dispositivos mediante el comando dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Solución de problemas de la unión a Azure AD híbrido para dispositivos actuales de Windows](troubleshoot-hybrid-join-windows-current.md)
- [Solución de problemas de la unión a Azure AD híbrido para dispositivos de nivel inferior de Windows](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [administrar identidades de dispositivos con Azure Portal](device-management-azure-portal.md).

<!--Image references-->
[1]: ./media/active-directory-conditional-access-automatic-device-registration-setup/12.png