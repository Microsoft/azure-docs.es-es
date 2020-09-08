---
title: Configuración de la unión a Azure Active Directory híbrido para dominios administrados | Microsoft Docs
description: Aprenda a configurar la unión a Azure Active Directory híbrido para dominios administrados.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 03/06/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 56b0685dee518399ae8328ddac18f03e82918a38
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89268424"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-managed-domains"></a>Tutorial: Configuración de dispositivos híbridos unidos a Azure Active Directory para dominios administrados

En este tutorial, aprenderá a configurar la unión a Azure Active Directory (Azure AD) híbrido para dispositivos unidos a un dominio de Active Directory. Este método admite un entorno administrado que incluye Active Directory local y Azure AD.

Al igual que un usuario de su organización, un dispositivo es una identidad principal que desea proteger. Puede usar una identidad de dispositivo para proteger los recursos en cualquier momento y ubicación. Puede lograr este objetivo mediante la administración de identidades de dispositivo en Azure AD. Utilice uno de los métodos siguientes:

- Unión a Azure AD
- Unión a Azure AD híbrido
- Registro de Azure AD

Este artículo se centra en la unión a Azure AD híbrido.

Al traer sus dispositivos a Azure AD, está maximizando la productividad de los usuarios mediante un inicio de sesión único (SSO) en los recursos de nube y del entorno local. Puede proteger el acceso a los recursos de nube y del entorno local con [acceso condicional](../conditional-access/howto-conditional-access-policy-compliant-device.md) al mismo tiempo.

Un entorno administrado se puede implementar mediante la [sincronización de hash de contraseña (PHS)](../hybrid/whatis-phs.md) o la [autenticación de paso a través (PTA)](../hybrid/how-to-connect-pta.md) con el [inicio de sesión único de conexión directa](../hybrid/how-to-connect-sso.md). Estos escenarios no requieren que se configure un servidor de federación para la autenticación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configuración de la unión a Azure AD híbrido
> * Habilitación de dispositivos de Windows de nivel inferior
> * Comprobación dispositivos unidos
> * Solución de problemas

## <a name="prerequisites"></a>Requisitos previos

- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) (versión 1.1.819.0 o posterior)
- Las credenciales de administrador global para el inquilino de Azure AD
- Las credenciales de administrador de empresa para cada uno de los bosques

Familiarícese con estos artículos:

- [¿Qué es una identidad de dispositivo?](overview.md)
- [Cómo: Planificar la implementación de la combinación a Azure Active Directory híbrido](hybrid-azuread-join-plan.md)
- [Validación controlada de la unión a Azure AD híbrido](hybrid-azuread-join-control.md)

> [!NOTE]
> Azure AD no admite tarjetas inteligentes ni certificados en dominios administrados.

Compruebe que Azure AD Connect ha sincronizado con Azure AD los objetos de equipo de los dispositivos que desea que estén unidos a Azure AD híbrido. Si los objetos de equipo pertenecen a unidades organizativas (OU) específicas, configure las OU para su sincronización en Azure AD Connect. Para más información acerca de cómo sincronizar objetos de equipo con Azure AD Connect, consulte [Filtrado basado en la unidad organizativa](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

Desde la versión 1.1.819.0, Azure AD Connect incluye un asistente para configurar la unión a Azure AD híbrido. El asistente simplifica considerablemente el proceso de configuración. El asistente configura los puntos de conexión de servicio (SCP) para el registro de dispositivos.

Los pasos de configuración en este artículo se basan en el uso del asistente de Azure AD Connect.

La unión a Azure AD híbrido requiere que los dispositivos tengan acceso a los siguientes recursos de Microsoft desde dentro de la red de su organización:  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- `https://autologon.microsoftazuread-sso.com` (si usa o planea usar SSO de conexión directa)

> [!WARNING]
> Si su organización usa servidores proxy que interceptan el tráfico SSL en escenarios como la prevención de pérdida de datos o las restricciones de inquilino de Azure AD, asegúrese de que el tráfico a "https://device.login.microsoftonline.com" se excluya de la interrupción e inspección de TLS. Si no excluye "https://device.login.microsoftonline.com", pueden surgir interferencias con la autenticación de certificados de cliente, lo que causaría problemas con el registro de dispositivos y el acceso condicional basado en dispositivos.

Si la organización necesita acceso a Internet mediante un proxy de salida, puede [implementar la detección automática de proxy web (WPAD)](/previous-versions/tn-archive/cc995261(v=technet.10)) para permitir que los equipos con Windows 10 realicen el registro de dispositivos con Azure AD. Para solucionar problemas de configuración y administración de WPAD, consulte [Solución de problemas de la detección automática](/previous-versions/tn-archive/cc302643(v=technet.10)). En los dispositivos Windows 10 anteriores a la actualización 1709, WPAD es la única opción disponible para configurar un proxy con el fin de que funcione con una combinación de Azure AD híbrido. 

Si no usa WPAD, puede configurar el proxy en WinHTTP a partir de la versión 1709 de Windows 10. Para más información, consulte [Configuración del servidor proxy WinHTTP implementado por GPO](/archive/blogs/netgeeks/winhttp-proxy-settings-deployed-by-gpo).

> [!NOTE]
> Si configura el proxy en el equipo mediante WinHTTP, todos los equipos que no se puedan conectar al proxy configurado no podrán conectarse a Internet.

Si la organización requiere acceso a Internet mediante un servidor proxy saliente autenticado, asegúrese de que los equipos con Windows 10 pueden autenticarse correctamente en el proxy de salida. Debido a que los equipos con Windows 10 ejecutan el registro de dispositivos mediante el contexto del equipo, configure la autenticación del proxy de salida mediante el contexto del equipo. Realice un seguimiento con su proveedor de proxy de salida en relación a los requisitos de configuración.

Compruebe si el dispositivo puede acceder a los recursos de Microsoft anteriores con la cuenta del sistema; para ello, utilice el script para [Probar la conectividad del registro de dispositivos](https://gallery.technet.microsoft.com/Test-Device-Registration-3dc944c0).

## <a name="configure-hybrid-azure-ad-join"></a>Configuración de la unión a Azure AD híbrido

Para configurar una unión a Azure AD híbrido mediante Azure AD Connect:

1. Inicie Azure AD Connect y, a continuación, seleccione **Configurar**.

   ![Bienvenido](./media/hybrid-azuread-join-managed-domains/welcome-azure-ad-connect.png)

1. En **Tareas adicionales**, seleccione **Configurar opciones de dispositivo** y, a continuación, seleccione **Siguiente**.

   ![Tareas adicionales](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-additional-tasks.png)

1. En **Información general**, seleccione **Siguiente**.

   ![Información general](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-overview.png)

1. En **Conectar con Azure AD**, proporcione las credenciales de administrador global para el inquilino de Azure AD.  

   ![Conectarse a Azure](./media/hybrid-azuread-join-managed-domains/connect-to-azure-ad-username-password.png)

1. En **Opciones de dispositivos**, seleccione **Configurar la unión a Azure AD híbrido** y, a continuación, seleccione **Siguiente**.

   ![Opciones de dispositivos](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-device-options.png)

1. En **Configuración del SCP**, para cada bosque en el que desee que Azure AD Connect configure el SCP, complete los pasos siguientes y, a continuación, seleccione **Siguiente**.

   1. Seleccione el **Bosque**.
   1. Seleccione un **Servicio de autenticación**.
   1. Seleccione **Agregar** para especificar las credenciales de administrador de empresa.

   ![SCP](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-scp-configuration.png)

1. En **Sistemas operativos de los dispositivos**, seleccione los sistemas operativos que usan los dispositivos del entorno de Active Directory y, a continuación, seleccione **Siguiente**.

   ![Sistema operativos del dispositivo](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-device-operating-systems.png)

1. En **Listo para configurar**, seleccione **Configurar**.

   ![Listo para configurar](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-ready-to-configure.png)

1. En **Configuración completada**, seleccione **Salir**.

   ![Configuración completada](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-configuration-complete.png)

## <a name="enable-windows-down-level-devices"></a>Habilitación de dispositivos de Windows de nivel inferior

Si algunos de los dispositivos unidos a un dominio son dispositivos Windows de nivel inferior, tendrá que:

- Configurar los valores de la intranet local para el registro de dispositivos
- Configurar SSO de conexión directa
- Instalación de Microsoft Workplace Join for Windows en equipos de nivel inferior

> [!NOTE]
> La compatibilidad con Windows 7 finalizó el 14 de enero de 2020. Para más información, consulte [Compatibilidad con Windows 7 finalizada](https://support.microsoft.com/help/4057281/windows-7-support-ended-on-january-14-2020).

### <a name="configure-the-local-intranet-settings-for-device-registration"></a>Configurar los valores de la intranet local para el registro de dispositivos

Para completar la unión a Azure AD híbrido de los dispositivos de Windows de nivel inferior y para evitar las peticiones de certificados cuando los dispositivos se autentican en Azure AD, puede insertar una directiva en los dispositivos unidos a un dominio para agregar las siguientes direcciones URL a la zona de intranet local en Internet Explorer:

- `https://device.login.microsoftonline.com`
- `https://autologon.microsoftazuread-sso.com`

También debe habilitar **Permitir actualizaciones en la barra de estado a través de script** en la zona de intranet local del usuario.

### <a name="configure-seamless-sso"></a>Configurar SSO de conexión directa

Para completar la unión a Azure AD híbrido de los dispositivos Windows de nivel inferior en un dominio administrado que usa la [sincronización de hash de contraseña](../hybrid/whatis-phs.md) o la [autenticación de paso a través](../hybrid/how-to-connect-pta.md) como el método de autenticación en la nube de Azure AD, también debe [configurar el inicio de sesión único de conexión directa](../hybrid/how-to-connect-sso-quick-start.md#step-2-enable-the-feature).

### <a name="install-microsoft-workplace-join-for-windows-down-level-computers"></a>Instalación de Microsoft Workplace Join for Windows en equipos de nivel inferior

Para registrar los dispositivos Windows de nivel inferior, las organizaciones deben instalar [Microsoft Workplace Join para equipos sin Windows 10](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join para equipos sin Windows 10 está disponible en el Centro de descarga de Microsoft.

El paquete se puede implementar mediante un sistema de distribución de software como  [Microsoft Endpoint Configuration Manager](/configmgr/). El paquete admite las opciones de instalación silenciosa estándar mediante el parámetro `quiet`. La versión actual de Configuration Manager ofrece ventajas adicionales sobre las versiones anteriores, como la posibilidad de realizar el seguimiento de los registros completados.

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
3. Si la columna **Registrado** indica **Pendiente**, entonces Unión a Azure AD híbrido no se ha completado.
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

Si tiene problemas para completar la unión a Azure AD híbrido para los dispositivos Windows unidos a un dominio, consulte:

- [Solución de problemas de dispositivos mediante el comando dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Solución de problemas de dispositivos unidos a Azure Active Directory híbrido](troubleshoot-hybrid-join-windows-current.md)
- [Solución de problemas de dispositivos híbridos de nivel inferior unidos a Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo para aprender a administrar las identidades de dispositivo mediante Azure Portal.
> [!div class="nextstepaction"]
> [Administración de identidades de dispositivo](device-management-azure-portal.md)