---
title: Introducción a Azure Multi-Factor Authentication
description: Aprenda de qué forma ayuda Azure Multi-Factor Authentication a proteger el acceso a los datos y a las aplicaciones, además de satisfacer la demanda de los usuarios de un proceso de inicio de sesión simple.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: b5f8d08f0814ec69719c002ea9efd39bb38b7d16
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718023"
---
# <a name="how-it-works-azure-multi-factor-authentication"></a>Funcionamiento: Azure Multi-Factor Authentication

La autenticación multifactor es un procedimiento en el que durante el proceso de inicio de sesión se solicita a un usuario una forma adicional de identificación, como especificar un código en su teléfono móvil o escanear su huella digital.

Si solo usa una contraseña para autenticar a un usuario, deja un vector desprotegido frente a los ataques. Si la contraseña es débil o se ha expuesto en otro lugar, ¿cómo saber si es el usuario quien inicia sesión realmente con el nombre de usuario y la contraseña y no un atacante? Al exigir una segunda forma de autenticación, aumenta la seguridad, ya que este factor adicional no es algo que resulte fácil de obtener o duplicar para un atacante.

![Imagen conceptual de las distintas formas de autenticación multifactor](./media/concept-mfa-howitworks/methods.png)

El funcionamiento de Azure Multi-Factor Authentication se basa en exigir uno o varios de los siguientes métodos de autenticación:

* Algo que conoce, normalmente una contraseña.
* Algo que tiene, como un dispositivo de confianza que no se puede duplicar con facilidad (por ejemplo, un teléfono o una clave de hardware).
* Algo que forma parte de usted, información biométrica como una huella digital o una detección de rostro.

Los usuarios pueden registrarse para el autoservicio de restablecimiento de contraseña y Azure Multi-Factor Authentication en un solo paso a fin de simplificar la experiencia de incorporación. Los administradores pueden definir qué formas de autenticación secundaria se pueden usar. También se puede exigir Azure Multi-Factor Authentication cuando los usuarios realizan un autoservicio de restablecimiento de contraseña para proteger mejor ese proceso.

![Métodos de autenticación en uso en la pantalla de inicio de sesión](media/concept-authentication-methods/overview-login.png)

Azure Multi-Factor Authentication ayuda a proteger el acceso a los datos y aplicaciones, y al mismo tiempo mantiene la simplicidad para los usuarios. Proporciona más seguridad, ya que requiere una segunda forma de autenticación y ofrece autenticación segura a través de una gama de [método de autenticación](concept-authentication-methods.md) fáciles de usar. Puede que los usuarios se vean desafiados por la MFA en función de las decisiones de configuración que tome un administrador.

No es necesario que sus aplicaciones o servicios realicen ningún cambio para usar Azure Multi-Factor Authentication. Los mensajes de comprobación forman parte del evento de inicio de sesión de Azure AD, que solicita y procesa automáticamente el desafío de MFA cuando es necesario.

## <a name="available-verification-methods"></a>Métodos de comprobación disponibles

Cuando un usuario inicia sesión en una aplicación o servicio, y recibe un mensaje de Multi-Factor Authentication, puede elegir uno de sus formularios registrados de comprobación adicional. Un administrador podría requerir el registro de estos métodos de comprobación de Azure Multi-Factor Authentication, o bien el usuario puede acceder a su propia instancia de [Mi perfil](https://myprofile.microsoft.com) para editar o agregar métodos de comprobación.

Con Azure Multi-Factor Authentication se pueden usar las siguientes formas adicionales de comprobación:

* Aplicación Microsoft Authenticator
* Token de hardware OATH
* sms
* Llamada de voz

## <a name="how-to-enable-and-use-azure-multi-factor-authentication"></a>Habilitación y uso de Azure Multi-Factor Authentication

Los usuarios y grupos pueden estar habilitados para Azure Multi-Factor Authentication para solicitar una comprobación adicional durante el evento de inicio de sesión. [Los valores predeterminados de seguridad](../fundamentals/concept-fundamentals-security-defaults.md) están disponibles para todos los Azure AD inquilinos para habilitar rápidamente el uso de la aplicación Microsoft Authenticator para todos los usuarios.

Para tener controles más pormenorizados, se pueden usar directivas de [acceso condicional](../conditional-access/overview.md) para definir los eventos o aplicaciones que requieren MFA. Estas directivas pueden permitir eventos de inicio de sesión regulares cuando el usuario se encuentra en la red corporativa o en un dispositivo registrado, pero solicitar más factores de comprobación cuando se encuentra en un dispositivo personal o en un entorno remoto.

![Diagrama general sobre el funcionamiento del acceso condicional para proteger el proceso de inicio de sesión](media/tutorial-enable-azure-mfa/conditional-access-overview.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre las licencias, consulte [Características y licencias de Azure Multi-Factor Authentication](concept-mfa-licensing.md).

Para ver MFA en acción, habilite Azure Multi-Factor Authentication para un conjunto de usuarios de prueba en el siguiente tutorial:

> [!div class="nextstepaction"]
> [Habilitación de Azure Multi-Factor Authentication](./tutorial-enable-azure-mfa.md)