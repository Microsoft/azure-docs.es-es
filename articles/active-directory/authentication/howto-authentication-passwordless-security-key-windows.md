---
title: 'Inicio de sesión con clave de seguridad sin contraseña en Windows: Azure Active Directory'
description: Información sobre cómo habilitar el inicio de sesión con clave de seguridad sin contraseña para Azure Active Directory mediante llaves de seguridad FIDO2 (versión preliminar)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 01/30/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: d70fe8a1fbaee285843bfd76ad2a8076df96b49b
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717972"
---
# <a name="enable-passwordless-security-key-sign-in-to-windows-10-devices-with-azure-active-directory-preview"></a>Habilitar el inicio de sesión con una clave de seguridad sin contraseña en dispositivos Windows 10 con Azure Active Directory (versión preliminar)

Este documento se centra en la habilitación de la autenticación sin contraseña basada en claves de seguridad FIDO2 para dispositivos Windows 10. Al final de este artículo, será capaz de iniciar sesión en los dispositivos Windows 10 unidos a Azure AD y a Azure AD híbrido con la cuenta de Azure AD mediante una clave de seguridad FIDO2.

> [!NOTE]
> Las claves de seguridad FIDO2 son una característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Requisitos

| Tipo de dispositivo | Unido a Azure AD | Híbrido unido a Azure AD |
| --- | --- | --- |
| [Azure Multi-Factor Authentication](howto-mfa-getstarted.md) | X | X |
| [Registro de información de seguridad combinado (vista preliminar)](concept-registration-mfa-sspr-combined.md) | X | X |
| [Claves de seguridad FIDO2](concept-authentication-passwordless.md#fido2-security-keys) compatibles | X | X |
| WebAuthN requiere Windows 10 versión 1809 o superior | X | X |
| [Los dispositivos unidos a Azure AD](../devices/concept-azure-ad-join.md) requieren Windows 10 versión 1903 o superior | X |   |
| [Los dispositivos unidos a Azure AD híbrido](../devices/concept-azure-ad-join-hybrid.md) requieren Windows 10 versión 2004 o superior |   | X |
| Controladores de dominio de Windows Server 2016/2019 totalmente revisados |   | X |
| [Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md#install-azure-ad-connect), versión 1.4.32.0 o posterior |   | X |
| [Microsoft Intune](/intune/fundamentals/what-is-intune) (opcional) | X | X |
| Paquete de aprovisionamiento (opcional) | X | X |
| Directiva de grupo (opcional) |   | X |

### <a name="unsupported-scenarios"></a>Escenarios no admitidos

No se admiten los escenarios siguientes:

- Implementación de Windows Server Active Directory Domain Services (AD DS) unido a un dominio (solo en dispositivos locales).
- Escenarios de RDP, VDI y Citrix con una clave de seguridad.
- S/MIME con una clave de seguridad.
- "Ejecutar como" con una clave de seguridad.
- Inicio de sesión en un servidor con una clave de seguridad.
- Si no ha usado la clave de seguridad para iniciar sesión en el dispositivo mientras está conectado, no podrá usarla para iniciar sesión o desbloquearlo sin conexión.
- Inicie sesión o desbloquee un dispositivo de Windows 10 con una clave de seguridad que contenga varias cuentas de Azure AD. En este escenario se utiliza la última cuenta agregada a la clave de seguridad. WebAuthN permite a los usuarios elegir la cuenta que desean usar.
- Desbloquee un dispositivo que ejecute Windows 10, versión 1809. Para disfrutar de la mejor experiencia, utilice Windows 10, versión 1903 o superior.

## <a name="prepare-devices-for-preview"></a>Preparar dispositivos para la versión preliminar

Los dispositivos unidos a Azure AD de los que realiza una prueba piloto durante la versión preliminar de características deben ejecutar Windows 10, versión 1809 o posterior. La mejor experiencia se logra con Windows 10 versión 1903 o superior.

Los dispositivos unidos a Azure AD híbrido deben ejecutar Windows 10, versión 2004 o una versión más reciente.

## <a name="enable-security-keys-for-windows-sign-in"></a>Habilitación de claves de seguridad para el inicio de sesión de Windows

Las organizaciones pueden optar por usar uno o varios de los métodos siguientes para habilitar el uso de claves de seguridad para el inicio de sesión de Windows en función de sus requisitos:

- [Habilitación con Intune](#enable-with-intune)
- [Implementación de Intune dirigida](#targeted-intune-deployment)
- [Habilitación con un paquete de aprovisionamiento](#enable-with-a-provisioning-package)
- [Habilitación con directiva de grupo (solo dispositivos unidos a Azure AD híbrido)](#enable-with-group-policy)

> [!IMPORTANT]
> Las organizaciones con dispositivos **híbridos Unidos a Azure AD** deben **también** completar los pasos del artículo [Habilitación de la autenticación de FIDO2 en recursos locales](howto-authentication-passwordless-security-key-on-premises.md) para que funcione la autenticación de claves de seguridad FIDO2 de Windows 10.
>
> Las organizaciones con **dispositivos unidos a Azure AD** deben hacerlo para que los dispositivos puedan autenticarse en recursos locales con claves de seguridad FIDO2.

### <a name="enable-with-intune"></a>Habilitación con Intune

Para habilitar el uso de claves de seguridad con Intune, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a **Microsoft Intune** > **Inscripción de dispositivos** > **Inscripción de Windows** > **Windows Hello para empresas** > **Propiedades**.
1. En **Configuración**, establezca **Utilice las claves de seguridad para el inicio de sesión** en **Habilitadas**.

La configuración de claves de seguridad para el inicio de sesión no depende de la configuración de Windows Hello para empresas.

### <a name="targeted-intune-deployment"></a>Implementación de Intune dirigida

Para dirigirse a grupos de dispositivos específicos a fin de habilitar el proveedor de credenciales, use la siguiente configuración personalizada mediante Intune:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a **Microsoft Intune** > **Configuración del dispositivo** > **Perfiles** > **Crear perfil**.
1. Configure el nuevo perfil con los valores siguientes:
   - Nombre: Claves de seguridad para inicio de sesión de Windows
   - Descripción: Habilita claves de seguridad FIDO que se van a usar durante el inicio de sesión de Windows
   - Plataforma: Windows 10 y versiones posteriores
   - Tipo de perfil: Personalizado
   - Configuración OMA-URI personalizada:
      - Nombre: Habilitar claves de seguridad FIDO para inicio de sesión de Windows
      - OMA-URI: ./Device/Vendor/MSFT/PassportForWork/SecurityKey/UseSecurityKeyForSignin
      - Tipo de datos: Entero
      - Valor: 1
1. Esta directiva se puede asignar a usuarios, dispositivos o grupos específicos. Para más información, consulte [Asignación de perfiles de usuario y de dispositivo en Microsoft Intune](/intune/device-profile-assign).

![Creación de directiva de configuración de dispositivos personalizada de Intune](./media/howto-authentication-passwordless-security-key/intune-custom-profile.png)

### <a name="enable-with-a-provisioning-package"></a>Habilitación con un paquete de aprovisionamiento

En el caso de los dispositivos no administrados por Intune, se puede instalar un paquete de aprovisionamiento para habilitar la funcionalidad. La aplicación Diseñador de configuración de Windows puede instalarse desde [Microsoft Store](https://www.microsoft.com/p/windows-configuration-designer/9nblggh4tx22). Complete los pasos siguientes para crear un paquete de aprovisionamiento:

1. Inicie el Diseñador de configuración de Windows.
1. Seleccione **Archivo** > **Nuevo proyecto**.
1. Asigne un nombre al proyecto, tome nota de la ruta de acceso donde se ha creado y, a continuación, seleccione **Siguiente**.
1. Deje *Paquete de aprovisionamiento* seleccionado como **Flujo de trabajo de proyecto seleccionados** y seleccione **Siguiente**.
1. Seleccione *Todas las ediciones del escritorio de Windows* en **Elige los valores que se deben ver y configurar** y, a continuación, seleccione **Siguiente**.
1. Seleccione **Finalizar**.
1. En el proyecto recién creado, vaya a **Configuración de tiempo de ejecución** > **WindowsHelloForBusiness** > **SecurityKeys** > **UseSecurityKeyForSignIn**.
1. Establezca **UseSecurityKeyForSignIn** en *Habilitado*.
1. Seleccione **Exportar** > **Paquete de aprovisionamiento**.
1. Deje los valores predeterminados en la ventana **Compilar** en **Describe el paquete de aprovisionamiento** y seleccione **Siguiente**.
1. Deje los valores predeterminados en la ventana **Compilar** en **Selecciona los detalles de seguridad del paquete de aprovisionamiento** y seleccione **Siguiente**.
1. Anote o cambie la ruta de acceso de la ventana **Compilar** en **Selecciona el lugar donde guardar el paquete de aprovisionamiento** y seleccione **Siguiente**.
1. Seleccione **Compilar** en la página **Compilar el paquete de aprovisionamiento**.
1. Guarde los dos archivos creados (*ppkg* y *cat*) en una ubicación donde pueda aplicarlos a los equipos más adelante.
1. Consulte [Aplicación de un paquete de aprovisionamiento](/windows/configuration/provisioning-packages/provisioning-apply-package) para aplicar el paquete de aprovisionamiento que ha creado.

> [!NOTE]
> Los dispositivos que ejecutan Windows 10 versión 1809 también deben habilitar el modo de PC compartido (*EnableSharedPCMode*). Para más información sobre cómo habilitar esta funcionalidad, consulte [Configuración de un equipo compartido o invitado con Windows 10](/windows/configuration/set-up-shared-or-guest-pc).

### <a name="enable-with-group-policy"></a>Habilitación con directiva de grupo

En el caso de los **dispositivos unidos a Azure AD híbrido**, las organizaciones pueden definir la siguiente configuración de directiva de grupo para habilitar el inicio de sesión con clave de seguridad FIDO. La configuración se puede encontrar en **Configuración del equipo** > **Plantillas administrativas** > **Sistema** > **Inicio de sesión** > **Turn on security key sign-in** (Activar inicio de sesión con clave de seguridad):

- Si se establece esta directiva en **Habilitada**, los usuarios podrán iniciar sesión con claves de seguridad.
- Si se establece en **Deshabilitada** o **No configurada**, los usuarios no podrán iniciar sesión con claves de seguridad.

Esta configuración de directiva de grupo requiere una versión actualizada de la plantilla de directiva de grupo `credentialprovider.admx`. Esta nueva plantilla está disponible con la siguiente versión de Windows Server y con Windows 10 20H1. Esta configuración se puede administrar con un dispositivo que ejecute una de estas versiones más recientes de Windows o de forma centralizada siguiendo las instrucciones del tema de soporte técnico [Creación y administración del almacén central de plantillas administrativas de la directiva de grupo en Windows](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra).

## <a name="sign-in-with-fido2-security-key"></a>Inicio de sesión con una clave de seguridad FIDO2

En el ejemplo siguiente, un usuario llamado Bala Sandhu ya ha aprovisionado su clave de seguridad FIDO2 mediante los pasos del artículo anterior, [Habilitación del inicio de sesión con la clave de seguridad sin contraseña](howto-authentication-passwordless-security-key.md#user-registration-and-management-of-fido2-security-keys). En el caso de los dispositivos unidos a Azure AD híbrido, asegúrese de que también ha [habilitado el inicio de sesión con clave de seguridad sin contraseña en los recursos locales](howto-authentication-passwordless-security-key-on-premises.md). Bala puede elegir el proveedor de credenciales de clave de seguridad desde la pantalla de bloqueo de Windows 10 e insertar la clave de seguridad para iniciar sesión en Windows.

![Inicio de sesión con clave de seguridad en la pantalla de bloqueo de Windows 10](./media/howto-authentication-passwordless-security-key/fido2-windows-10-1903-sign-in-lock-screen.png)

### <a name="manage-security-key-biometric-pin-or-reset-security-key"></a>Administrar biométrica, PIN o restablecer clave de seguridad

* Windows 10 versión 1903 o posterior
   * Los usuarios pueden abrir **Configuración de Windows** en el dispositivo > **Cuentas** > **Clave de seguridad**
   * Los usuarios pueden cambiar el PIN, actualizar la biométrica o restablecer la clave de seguridad

## <a name="troubleshooting-and-feedback"></a>Solución de problemas y comentarios

Si quiere compartir comentarios o detectar problemas mientras usa la versión preliminar de esta característica, compártalos mediante la aplicación Centro de opiniones sobre Windows. Para ello, realice los pasos siguientes:

1. Abra el **Centro de opiniones** y asegúrese de que ha iniciado sesión.
1. Envíe los comentarios bajo la categorización siguiente:
   - Categoría: Seguridad y privacidad
   - Subcategoría: FIDO
1. Para capturar registros, use la opción **Volver a crear mi problema**.

## <a name="next-steps"></a>Pasos siguientes

[Habilitación del acceso a recursos locales para dispositivos unidos a Azure AD y Azure AD híbrido](howto-authentication-passwordless-security-key-on-premises.md)

[Más información sobre el registro de dispositivos](../devices/overview.md)

[Más información sobre Azure Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)