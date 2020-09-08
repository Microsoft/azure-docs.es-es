---
title: Unir un nuevo dispositivo Windows 10 con Azure AD durante la primera vista de Windows| Microsoft Docs
description: Cómo los usuarios pueden configurar Azure AD Join durante la configuración rápida.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 06/28/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jairoc
ms.collection: M365-identity-device-management
ms.openlocfilehash: da37316724bf6ef166f08faa7208ad196000bb00
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85253110"
---
# <a name="tutorial-join-a-new-windows-10-device-with-azure-ad-during-a-first-run"></a>Tutorial: Unión de un nuevo dispositivo Windows 10 con Azure AD durante la primera vista de Windows

Con la administración de dispositivos en Azure Active Directory (Azure AD), puede asegurarse de que los usuarios tienen acceso a los recursos desde dispositivos que cumplen los estándares de seguridad y cumplimiento. Para más información, consulte [Introducción a la administración de dispositivos en Azure Active Directory](overview.md).

Con Windows 10, puede unir un dispositivo nuevo a Azure AD durante la primera vista de Windows (FRX).  
Esto le permite distribuir los dispositivos empaquetados a los empleados o alumnos.

Si tiene Windows 10 Professional o Windows 10 Enterprise instalado en un dispositivo, la experiencia predeterminada es el proceso de configuración de los dispositivos de la empresa.

En la *configuración rápida* de Windows, no se puede unir a un dominio local de Active Directory (AD). Si planea unir un equipo a un dominio de AD, durante la configuración, debe seleccionar el vínculo **Configurar Windows con una cuenta local**. Luego, puede unirse al dominio desde la configuración del equipo.
 
En este tutorial, aprenderá a unir un dispositivo a Azure AD durante FRX:
 > [!div class="checklist"]
> * Prerequisites
> * Unión de un dispositivo
> * Comprobación

## <a name="prerequisites"></a>Prerequisites

Para unir un dispositivo Windows 10, el servicio de registro de dispositivos debe configurarse para que pueda registrar dispositivos. Además de tener permiso para unir dispositivos en su inquilino de Azure AD, debe tener menos dispositivos registrados que el máximo configurado. Para obtener más información, consulte [Configure device settings](device-management-azure-portal.md#configure-device-settings) (Configuración del dispositivo).

Además, si el inquilino es federado, el proveedor de identidad DEBE admitir puntos de conexión de nombre de usuario y contraseña WS-Fed y WS-Trust. Puede ser la versión 1.3 o 2005. Esta compatibilidad con el protocolo se requiere tanto para unir el dispositivo a Azure AD como para iniciar sesión en el dispositivo con una contraseña.

## <a name="joining-a-device"></a>Unión de un dispositivo

**Para unir un dispositivo Windows 10 a Azure AD durante FRX:**

1. Al activar el nuevo dispositivo e iniciar el proceso de configuración, debería ver el mensaje **Preparación** . Siga las indicaciones para configurar el dispositivo.
1. Para comenzar, personalice su región e idioma. A continuación, acepte los términos de licencia del software de Microsoft.
 
    ![Personalizar para la región](./media/azuread-joined-devices-frx/01.png)

1. Seleccione la red que desea usar para conectarse a Internet.
1. Haga clic en **Este dispositivo pertenece a mi organización**. 

    ![Pantalla ¿A quién pertenece el equipo?](./media/azuread-joined-devices-frx/02.png)

1. Escriba las credenciales que le ha proporcionado la organización y después haga clic en **Iniciar sesión**.

    ![Pantalla de inicio de sesión](./media/azuread-joined-devices-frx/03.png)

1. El dispositivo busca un inquilino que coincida en Azure AD. Si se encuentra en un dominio federado, se le redirige al servidor del servicio de token seguro (STS) local; por ejemplo, Servicios de federación de Active Directory (AD FS).
1. Si es un usuario en un dominio no federado, deberá escribir las credenciales directamente en la página hospedada en Azure AD. 
1. Encontrará un desafío de autenticación multifactor. 
1. Azure AD comprobará si se requiere una inscripción en la administración de dispositivos móviles.
1. Windows registra el dispositivo en el directorio de la organización en Azure AD y lo inscribe en la administración de dispositivos móviles, si procede.
1. Si es:
   - un usuario administrado, Windows le llevará al escritorio mediante el proceso de inicio de sesión automático;
   - un usuario federado, verá la pantalla de inicio de sesión de Windows y deberá escribir sus credenciales.

## <a name="verification"></a>Comprobación

Para comprobar si un dispositivo está unido a Azure AD, consulte el cuadro de diálogo **Obtener acceso a trabajo o escuela** en el dispositivo Windows. El cuadro de diálogo debe indicar que está conectado a su directorio de Azure AD.

![Obtener acceso a trabajo o escuela](./media/azuread-joined-devices-frx/13.png)

## <a name="next-steps"></a>Pasos siguientes

- Para más información, consulte [Introducción a la administración de dispositivos en Azure Active Directory](overview.md).
- Para más información sobre cómo administrar dispositivos en el portal de Azure AD, consulte [Administración de dispositivos con Azure Portal](device-management-azure-portal.md).
