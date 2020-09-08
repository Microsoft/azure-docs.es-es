---
title: 'Azure AD Connect: autenticación de paso a través, actualización de los agentes de autenticación | Microsoft Docs'
description: En este artículo se describe cómo actualizar la configuración de la Autenticación de paso a través de Azure Active Directory (Azure AD).
services: active-directory
keywords: Autenticación de paso a través de Azure AD Connect, instalación de Active Directory, componentes necesarios para Azure AD, SSO, inicio de sesión único
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/27/2018
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1d0507f7751305af5e626cbd7dd6e0dfd1a63a74
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279489"
---
# <a name="azure-active-directory-pass-through-authentication-upgrade-preview-authentication-agents"></a>Autenticación de paso a través de Azure Active Directory: Actualización de la versión preliminar de los agentes de autenticación

## <a name="overview"></a>Información general

Este artículo está dirigido a los clientes que usan la Autenticación de paso a través de Azure AD en versión preliminar. Se acaba de actualizar el software del agente de autenticación y, en consecuencia, también se ha cambiado el nombre. Debe actualizar _manualmente_ la versión preliminar de los agentes de autenticación instalados en los servidores locales. Esta actualización manual requiere una única acción. Todas las actualizaciones futuras de los agentes de autenticación son automáticas. Los motivos para actualizar son los siguientes:

- Las versiones preliminares de los agentes de autenticación no recibirán más correcciones de seguridad o de errores.
-   Las versiones preliminares de los agentes de autenticación no se pueden instalar en otros servidores para lograr alta disponibilidad.

## <a name="check-versions-of-your-authentication-agents"></a>Comprobación de las versiones de los agentes de autenticación

### <a name="step-1-check-where-your-authentication-agents-are-installed"></a>Paso 1: Comprobación de dónde están instalados los agentes de autenticación

Siga estos pasos para comprobar si los agentes de autenticación están instalados:

1. Inicie sesión en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) con las credenciales de administrador global del inquilino.
2. Seleccione **Azure Active Directory** en la barra de navegación de la izquierda.
3. Seleccione **Azure AD Connect**. 
4. Seleccione **Autenticación de paso a través**. En esta hoja se enumeran los servidores donde están instalados los agentes de autenticación.

![Centro de administración de Azure Active Directory: hoja de Autenticación de paso a través](./media/how-to-connect-pta-upgrade-preview-authentication-agents/pta8.png)

### <a name="step-2-check-the-versions-of-your-authentication-agents"></a>Paso 2: Comprobación de las versiones de los agentes de autenticación

Para comprobar las versiones de los agentes de autenticación, en cada servidor identificado en el paso anterior, siga estas instrucciones:

1. Vaya a **Panel de control -> Programas -> Programas y características** en el servidor local.
2. Si hay una entrada de "**Microsoft Azure AD Connect Authentication Agent**" (Agente de autenticación de Microsoft Azure AD Connect), no es necesario realizar ninguna acción en este servidor.
3. Si hay una entrada de "**Microsoft Azure AD Application Proxy Connector**" (Conector proxy de aplicación de Microsoft Azure AD), debe actualizar manualmente en este servidor.

![Versión preliminar del agente de autenticación](./media/how-to-connect-pta-upgrade-preview-authentication-agents/pta6.png)

## <a name="best-practices-to-follow-before-starting-the-upgrade"></a>Procedimientos recomendados antes de iniciar la actualización

Antes de actualizar, asegúrese de que existen los siguientes elementos:

1. **Crear una cuenta de administrador global solo en la nube**: no actualice sin tener una cuenta de administrador global solo en la nube para usarla en situaciones de emergencia en las que los agentes de autenticación de paso a través no funcionen correctamente. Información acerca de la [incorporación de una cuenta de administrador global que está solo en la nube](../fundamentals/add-users-azure-active-directory.md). Este paso es esencial y se asegura de no quedar bloqueado fuera de su inquilino.
2.  **Asegurar la alta disponibilidad**: si no lo ha hecho anteriormente, instale un segundo agente de autenticación independiente a fin de proporcionar alta disponibilidad para las solicitudes de inicio de sesión; para ello, siga estas [instrucciones](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

## <a name="upgrading-the-authentication-agent-on-your-azure-ad-connect-server"></a>Actualización del agente de autenticación en el servidor de Azure AD Connect

Necesita actualizar Azure AD Connect antes de actualizar el agente de autenticación en el mismo servidor. Siga estos pasos tanto en el servidor principal como en el servidor provisional de Azure AD Connect:

1. **Actualice Azure AD Connect**: siga este [artículo](how-to-upgrade-previous-version.md) y actualice a la versión más reciente de Azure AD Connect.
2. **Desinstale la versión preliminar del agente de autenticación**: Descargue [este script de PowerShell](https://aka.ms/rmpreviewagent) y ejecútelo como administrador en el servidor.
3. **Descargue la última versión del agente de autenticación (versión 1.5.389.0 o posterior)** : inicie sesión en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) con las credenciales de administrador global del inquilino. Seleccione **Azure Active Directory -> Azure AD Connect -> Autenticación de paso a través -> Descargar agente**. Acepte las [condiciones del servicio](https://aka.ms/authagenteula) y descargue la última versión del agente de autenticación. También puede descargar el agente de autenticación desde [aquí](https://aka.ms/getauthagent).
4. **Instale la versión más reciente del agente de autenticación**: ejecute el archivo ejecutable descargado en el paso 3. Proporcione las credenciales de administrador global de su inquilino cuando se le solicite.
5. **Compruebe que se ha instalado la versión más reciente**: como se indicó anteriormente, vaya a **Panel de control -> Programas -> Programas y características** y compruebe que haya una entrada para "**Microsoft Azure AD Connect Authentication Agent**" (Agente de autenticación de Microsoft Azure AD Connect).

>[!NOTE]
>Si activa la hoja Autenticación de paso a través en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) después de completar los pasos anteriores, verá dos entradas del agente de autenticación por servidor; una entrada en la que aparece el agente de autenticación como **activo** y otra, como **inactivo**. Se _espera_ que esto sea así. La entrada **Inactivo** se quita automáticamente después de unos días.

## <a name="upgrading-the-authentication-agent-on-other-servers"></a>Actualización del agente de autenticación en otros servidores

Siga estos pasos para actualizar los agentes de autenticación en otros servidores (donde no se instala Azure AD Connect):

1. **Desinstale la versión preliminar del agente de autenticación**: Descargue [este script de PowerShell](https://aka.ms/rmpreviewagent) y ejecútelo como administrador en el servidor.
2. **Descargue la última versión del agente de autenticación (versión 1.5.389.0 o posterior)** : inicie sesión en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) con las credenciales de administrador global del inquilino. Seleccione **Azure Active Directory -> Azure AD Connect -> Autenticación de paso a través -> Descargar agente**. Acepte las condiciones del servicio y descargue la última versión.
3. **Instale la versión más reciente del agente de autenticación**: Ejecute el archivo ejecutable descargado en el paso 2. Proporcione las credenciales de administrador global de su inquilino cuando se le solicite.
4. **Compruebe que se ha instalado la versión más reciente**: como se indicó anteriormente, vaya a **Panel de control -> Programas -> Programas y características** y compruebe que haya una entrada llamada "**Microsoft Azure AD Connect Authentication Agent**" (Agente de autenticación de Microsoft Azure AD Connect).

>[!NOTE]
>Si activa la hoja Autenticación de paso a través en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) después de completar los pasos anteriores, verá dos entradas del agente de autenticación por servidor; una entrada en la que aparece el agente de autenticación como **activo** y otra, como **inactivo**. Se _espera_ que esto sea así. La entrada **Inactivo** se quita automáticamente después de unos días.

## <a name="next-steps"></a>Pasos siguientes
- [**Solución de problemas**](tshoot-connect-pass-through-authentication.md): información para resolver problemas habituales de esta característica.