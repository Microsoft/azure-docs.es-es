---
title: 'Azure AD Connect: tareas posteriores a la configuración de la combinación de Azure AD híbrido | Microsoft Docs'
description: En este documento se detallan las tareas posteriores a la configuración necesarias para completa la combinación de Azure AD híbrido
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: billmath
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/10/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: da5cefbacbd3851d2609a687c1948d9bcba5ffae
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88612476"
---
# <a name="post-configuration-tasks-for-hybrid-azure-ad-join"></a>Tareas posteriores a la configuración de la combinación de Azure AD híbrido

Después de ejecutar Azure AD Connect para configurar la organización para la combinación de Azure AD híbrido, hay algunos pasos adicionales que debe completar para finalizar la instalación.  Lleve a cabo solo los pasos correspondientes a sus dispositivos.

## <a name="1-configure-controlled-rollout-optional"></a>1. Configuración del lanzamiento controlado (opcional)
Todos los dispositivos unidos a un dominio que ejecuten Windows 10 y Windows Server 2016 se registran automáticamente en Azure AD una vez que se han completado todos los pasos de configuración. Si prefiere realizar un lanzamiento controlada en lugar de este registro automático, puede usar la directiva de grupo para habilitar o deshabilitar el lanzamiento automático de forma selectiva.  Esta directiva de grupo debe establecerse antes de iniciar otros pasos de configuración:
* Cree un objeto de directiva de grupo en Active Directory.
* Póngale nombre (por ejemplo, combinación de Azure AD híbrido).
* Edítelo y vaya a:  Configuración del equipo > Directivas > Plantillas administrativas > Componentes de Windows > Registro de dispositivos.

>[!NOTE]
>Para 2012R2, la configuración de directivas se encuentra en **Configuración del equipo > Directivas > Plantillas administrativas > Componentes de Windows > Workplace Join > Automatically workplace join client computers** (Unión automática al área de trabajo de equipos cliente).

* Habilite esta configuración:  Registro de equipos unidos a un dominio como dispositivos.
* Aplique las opciones y haga clic en Aceptar.
* Vincule GPO a la ubicación de su elección (unidad organizativa, grupo de seguridad o dominio para todos los dispositivos).

## <a name="2-configure-network-with-device-registration-endpoints"></a>2. Configurar la red con los puntos de conexión del registro de dispositivos
Asegúrese de que las direcciones URL siguientes son accesibles desde equipos dentro de la red de su organización para el registro en Azure AD:

* `https://enterpriseregistration.windows.net`
* `https://login.microsoftonline.com`
* `https://device.login.microsoftonline.com` 

## <a name="3-implement-wpad-for-windows-10-devices"></a>3. Implementar WPAD para dispositivos Windows 10
Si la organización accede a Internet a través de un proxy de salida, implemente la Detección automática de proxy web (WPAD) para permitir que los equipos de Windows 10 se registren en Azure AD.

## <a name="4-configure-the-scp-in-any-forests-that-were-not-configured-by-azure-ad-connect"></a>4. Configurar SCP en los bosques que no configuró Azure AD Connect 

El punto de conexión de servicio (SCP) contiene la información del inquilino de Azure AD que usarán los dispositivos durante el registro automático.  Ejecute el script de PowerShell, ConfigureSCP.ps1, que ha descargado de Azure AD Connect.

## <a name="5-configure-any-federation-service-that-was-not-configured-by-azure-ad-connect"></a>5. Configurar cualquier servicio de federación que no haya configurado Azure AD Connect

Si su organización utiliza un servicio de federación para iniciar sesión en Azure AD, las reglas de notificación en la relación de confianza para usuario autenticado de Azure AD deben permitir la autenticación de dispositivos. Si utiliza la federación con AD FS, vaya a la [ayuda de AD FS](https://aka.ms/aadrptclaimrules) para generar las reglas de notificación. Si utiliza una solución de federación que no sea de Microsoft, póngase en contacto con ese proveedor para obtener instrucciones.  

>[!NOTE]
>Si tiene dispositivos de nivel inferior de Windows, el servicio debe admitir la emisión de notificaciones authenticationmethod y wiaormultiauthn al recibir las solicitudes para la relación de confianza de Azure AD. En AD FS, debería agregar una regla de transformación de emisión que atraviese el método de autenticación.

## <a name="6-enable-azure-ad-seamless-sso-for-windows-down-level-devices"></a>6. Habilitar el SSO de conexión directa de Azure AD para dispositivos de nivel inferior de Windows

Si su organización usa la sincronización de hash de contraseñas o la autenticación de paso a través para iniciar sesión en Azure AD, habilite el SSO de conexión directa de Azure AD con ese método de inicio de sesión para autenticar los dispositivos de nivel inferior de Windows: https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso. 

## <a name="7-set-azure-ad-policy-for-windows-down-level-devices"></a>7. Configurar una directiva de Azure AD para dispositivos de nivel inferior de Windows

Para registrar dispositivos de nivel inferior de Windows, es preciso asegurarse de que la directiva de Azure AD permite a los usuarios registrar dispositivos. 

* Inicie sesión en su cuenta en Azure Portal.
* Vaya a:  Azure Active Directory > Dispositivos > Configuración de dispositivo.
* Establezca "Los usuarios pueden registrar sus dispositivos con Azure AD" en TODOS.
* Haga clic en Guardar

## <a name="8-add-azure-ad-endpoint-to-windows-down-level-devices"></a>8. Agregar un punto de conexión de Azure AD a dispositivos de nivel inferior de Windows

Agregue el punto de conexión de autenticación de dispositivos de Azure AD a las zonas de la intranet locales en los dispositivos de nivel inferior de Windows para evitar las solicitudes de certificado al autenticar los dispositivos: `https://device.login.microsoftonline.com`. 

Si utiliza un [SSO de conexión directa](how-to-connect-sso.md), habilite también la opción "Permitir actualizaciones de barra de estado a través de scripts" en esa zona y agregue el siguiente punto de conexión: `https://autologon.microsoftazuread-sso.com`. 

## <a name="9-install-microsoft-workplace-join-on-windows-down-level-devices"></a>9. Instalar Microsoft Workplace Join en dispositivos de nivel inferior de Windows

El instalador crea una tarea programada en el sistema del dispositivo que se ejecuta en el contexto del usuario. La tarea se desencadena cuando el usuario inicia sesión en Windows. La tarea registra de forma silenciosa el dispositivo en Azure AD con las credenciales de usuario después de autenticar mediante la autenticación integrada de Windows. El centro de descarga se encuentra en https://www.microsoft.com/download/details.aspx?id=53554. 

## <a name="10-configure-group-policy-to-allow-device-registration"></a>10. Configurar la directiva de grupo para permitir el registro de dispositivos

Para obtener información sobre cómo permitir la unión a Azure AD híbrido para dispositivos individuales, consulte [Validación controlada de la unión a Azure AD híbrido](../devices/hybrid-azuread-join-control.md).

## <a name="next-steps"></a>Pasos siguientes
[Configurar la escritura diferida de dispositivos](how-to-connect-device-writeback.md)
