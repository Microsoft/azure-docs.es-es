---
title: Habilitación de la protección con contraseña de Azure AD local
description: Aprenda a habilitar la protección con contraseña de Azure AD para un entorno de Active Directory Domain Services local.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/05/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4e4797c3e089e0ae9d15cc604e9f93605978b36c
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419774"
---
# <a name="enable-on-premises-azure-active-directory-password-protection"></a>Habilitación de la protección con contraseña de Azure Active Directory local

Los usuarios suelen crear contraseñas que usan palabras comunes locales, como una escuela, un equipo deportivo o una persona famosa. Estas contraseñas son fáciles de adivinar y poco seguras contra ataques basados en diccionarios. Para aplicar contraseñas seguras en su organización, la protección con contraseña de Azure Active Directory (Azure AD) proporciona una lista de contraseñas prohibidas global y personalizada. Se produce un error en una solicitud de cambio de contraseña si hay una coincidencia en esta lista de contraseñas prohibidas.

Para proteger el entorno de Active Directory Domain Services (AD DS) local, puede instalar y configurar la protección con contraseña de Azure AD para que funcione con el controlador de dominio local. En este artículo se muestra cómo habilitar la protección con contraseña de Azure AD para su entorno local.

Para más información sobre cómo funciona la protección con contraseña de Azure AD en un entorno local, consulte [Aplicación de Protección con contraseña de Azure AD para Windows Server Active Directory](concept-password-ban-bad-on-premises.md).

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se muestra cómo habilitar la protección con contraseña de Azure AD para su entorno local. Antes de completar este artículo, [instale y registre el servicio de proxy de protección con contraseña de Azure AD y el agente de controlador de dominio](howto-password-ban-bad-on-premises-deploy.md) en el entorno AD DS local.

## <a name="enable-on-premises-password-protection"></a>Habilitación de la protección con contraseña local

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a **Azure Active Directory**,  > **Seguridad** > **Métodos de autenticación** > **Protección con contraseña**.
1. Establezca la opción **Habilitar protección con contraseña en Windows Server Active Directory** en *Sí*.

    Cuando esta opción se establece en *No*, todos los agentes de controlador de dominio de la protección de contraseña Azure AD implementados entran en un modo inactivo, donde todas las contraseñas se aceptan tal cual. No se realizan actividades de validación ni se generan eventos de auditoría.

1. Se recomienda establecer inicialmente el **modo** en *Auditoría*. Cuando esté familiarizado con la característica y el impacto en los usuarios de su organización, puede cambiar el **modo** a *Forzado*. Para más información, consulte la sección siguiente en [modos de operación](#modes-of-operation).
1. Cuando esté preparado, seleccione **Guardar**.

    [![Habilitación de la protección con contraseña local en Métodos de autenticación en Azure Portal](media/howto-password-ban-bad-on-premises-operations/enable-configure-custom-banned-passwords-cropped.png)](media/howto-password-ban-bad-on-premises-operations/enable-configure-custom-banned-passwords.png#lightbox)

## <a name="modes-of-operation"></a>Modos de operación

Al habilitar la protección con contraseña de Azure AD local, puede usar el modo de *auditoría* o el modo de *aplicación*. Recomendamos que la implementación y las pruebas iniciales comiencen siempre en el modo auditoría. Las entradas del registro de eventos se deben supervisar después para tantear si alguno de los procesos operativos existentes se vería perturbado una vez que se habilite el modo *Forzado*.

### <a name="audit-mode"></a>Modo Auditoría

El modo *Auditoría* está previsto como una forma de ejecutar el software en un modo "what if". Cada servicio de agente de controlador de dominio de la protección con contraseña de Azure AD evalúa una contraseña de entrada según la directiva activa en el momento.

Si la directiva actual se ha configurado para estar en modo de auditoría, las contraseñas "incorrectas" se traducen en mensajes del registro de errores pero se procesan y actualizan. Este comportamiento es la única diferencia entre el modo Auditoría y el modo Forzado. Todas las demás operaciones se ejecutan de la misma forma.

### <a name="enforced-mode"></a>Modo Forzado

El modo *Forzado* está previsto como la configuración final. Como en el anterior modo de auditoría, cada servicio de agente de controlador de dominio de la protección con contraseña de Azure AD evalúa las contraseñas de entrada según la directiva activa en el momento. Cuando el modo Forzado está habilitado, se rechaza una contraseña que se considera no segura según la directiva.

Cuando un agente de controlador de dominio de la protección con contraseña de Azure AD DC rechaza una contraseña en modo Forzado, el usuario final ve un error similar al que percibiría si su contraseña fuese rechazada por la aplicación tradicional de complejidad de la contraseña local. Por ejemplo, un usuario podría ver el siguiente mensaje de error tradicional en la pantalla de inicio de sesión o cambio de contraseña de Windows:

*"No se puede actualizar la contraseña. El valor proporcionado para la nueva contraseña no cumple los requisitos de longitud, complejidad o de historial del dominio".*

Este mensaje es solo un ejemplo de varios resultados posibles. El mensaje de error específico puede variar según el escenario o software real que intenta establecer una contraseña no segura.

Puede que los usuarios finales afectados tengan que trabajar con su personal de TI para comprender los nuevos requisitos y elegir contraseñas seguras.

> [!NOTE]
> La funcionalidad Protección con contraseña de Azure AD no tiene ningún control sobre el mensaje de error específico que se muestra en la máquina cliente cuando se rechaza una contraseña débil.

## <a name="next-steps"></a>Pasos siguientes

Para personalizar la lista de contraseñas prohibidas de su organización, consulte [Tutorial: Configuración de contraseñas prohibidas personalizadas para la protección con contraseña de Azure Active Directory](tutorial-configure-custom-password-protection.md).

Para supervisar los eventos locales, consulte [Supervisión y revisión de los registros de los entornos de protección con contraseña de Azure AD local](howto-password-ban-bad-on-premises-monitor.md).
