---
title: Introducción a la autenticación de Azure Active Directory
description: Conozca los diferentes métodos de autenticación y características de seguridad para los inicios de sesión de usuario con Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: overview
ms.date: 07/13/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: sahenry, michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5e65149b2524d2785039be78caad4ce84924e7e3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035118"
---
# <a name="what-is-azure-active-directory-authentication"></a>¿Qué es la autenticación de Azure Active Directory?

Una de las principales características de una plataforma de identidad es comprobar, o *autenticar*, las credenciales cuando un usuario inicia sesión en un dispositivo, una aplicación o un servicio. En Azure Active Directory (Azure AD), la autenticación supone algo más que comprobar un nombre de usuario y una contraseña. Para mejorar la seguridad y reducir la necesidad de asistencia del departamento de soporte técnico, la autenticación de Azure AD incluye los siguientes componentes:

* Restablecimiento de la contraseña de autoservicio
* Azure Multi-Factor Authentication
* Integración híbrida para reescribir los cambios de contraseña en el entorno local
* Integración híbrida para aplicar directivas de protección de contraseñas en un entorno local
* Autenticación sin contraseñas

## <a name="improve-the-end-user-experience"></a>Mejora de la experiencia del usuario final

Azure AD ayuda a proteger la identidad de un usuario y a simplificar su experiencia de inicio de sesión. Características como el autoservicio de restablecimiento de contraseña permiten a los usuarios actualizar o cambiar sus contraseñas mediante un explorador web desde cualquier dispositivo. Esta característica es especialmente útil cuando el usuario ha olvidado su contraseña o se ha bloqueado su cuenta, ya que puede desbloquearse y seguir trabajando sin tener que esperar a que el departamento de soporte técnico o el administrador le ayuden.

Azure Multi-Factor Authentication permite a los usuarios elegir una forma adicional de autenticación durante el inicio de sesión, como una llamada de teléfono o una notificación de la aplicación móvil. Esta capacidad reduce el requisito de una única forma fija de autenticación secundaria, como un token de hardware. Si el usuario no dispone actualmente de una forma de autenticación adicional, puede elegir un método diferente y seguir trabajando.

![Métodos de autenticación en uso en la pantalla de inicio de sesión](media/concept-authentication-methods/overview-login.png)

La autenticación sin contraseña elimina la necesidad de que el usuario cree y recuerde una contraseña segura. Funcionalidades como las claves de seguridad de Windows Hello para empresas o FIDO2 permiten a los usuarios iniciar sesión en un dispositivo o una aplicación sin necesidad de una contraseña. Esta capacidad puede reducir la complejidad de administrar contraseñas en diferentes entornos.

## <a name="self-service-password-reset"></a>Restablecimiento de la contraseña de autoservicio

El autoservicio de restablecimiento de contraseña ofrece a los usuarios la posibilidad de cambiar o restablecer su contraseña, sin necesidad de que intervenga el administrador o el departamento de soporte técnico. Si la cuenta de un usuario está bloqueada o se ha olvidado su contraseña, puede seguir las indicaciones para desbloquearse y volver al trabajo. Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación.

El autoservicio de restablecimiento de contraseña funciona en los siguientes escenarios:

* **Cambio de contraseña**: el usuario conoce la contraseña pero desea cambiarla por una nueva.
* **Restablecimiento de contraseña:** el usuario no puede iniciar sesión, por ejemplo, ha olvidado la contraseña y quiere restablecerla.
* **Desbloqueo de cuenta:** el usuario no puede iniciar sesión porque su cuenta está bloqueada y quiere desbloquearla.

Cuando un usuario actualiza o restablece su contraseña mediante el autoservicio de restablecimiento de contraseña, esa contraseña también puede reescribirse en un entorno de Active Directory local. La escritura diferida de contraseñas garantiza que un usuario puede usar inmediatamente sus credenciales actualizadas en aplicaciones y dispositivos locales.

## <a name="azure-multi-factor-authentication"></a>Azure Multi-Factor Authentication

La autenticación multifactor es un procedimiento en el que durante el proceso de inicio de sesión se solicita a un usuario una forma adicional de identificación, como especificar un código en su teléfono móvil o escanear su huella digital.

Si solo usa una contraseña para autenticar a un usuario, deja un vector desprotegido frente a los ataques. Si la contraseña es débil o se ha expuesto en otro lugar, ¿cómo saber si es el usuario quien inicia sesión realmente con el nombre de usuario y la contraseña y no un atacante? Al exigir una segunda forma de autenticación, aumenta la seguridad, ya que este factor adicional no es algo que resulte fácil de obtener o duplicar para un atacante.

![Imagen conceptual de las distintas formas de autenticación multifactor](./media/concept-mfa-howitworks/methods.png)

El funcionamiento de Azure Multi-Factor Authentication se basa en exigir uno o varios de los siguientes métodos de autenticación:

* Algo que conoce, normalmente una contraseña.
* Algo que tiene, como un dispositivo de confianza que no se puede duplicar con facilidad (por ejemplo, un teléfono o una clave de hardware).
* Algo que forma parte de usted, información biométrica como una huella digital o una detección de rostro.

Los usuarios pueden registrarse para el autoservicio de restablecimiento de contraseña y Azure Multi-Factor Authentication en un solo paso a fin de simplificar la experiencia de incorporación. Los administradores pueden definir qué formas de autenticación secundaria se pueden usar. También se puede exigir Azure Multi-Factor Authentication cuando los usuarios realizan un autoservicio de restablecimiento de contraseña para proteger mejor ese proceso.

## <a name="password-protection"></a>Protección con contraseña

De forma predeterminada, Azure AD bloquea las contraseñas no seguras, como *Contraseña1*. Se actualiza y se aplica automáticamente una lista global de contraseñas prohibidas que incluye contraseñas no seguras conocidas. Si un usuario de Azure AD intenta utilizar como contraseña una de estas contraseñas no seguras, recibirá una notificación para elegir otra más segura.

Para aumentar la seguridad, puede definir directivas de protección con contraseña personalizadas. Estas directivas pueden usar filtros para bloquear cualquier variación de una contraseña que contenga, por ejemplo, un nombre como *Contoso* o una ubicación como *Londres*.

Si busca seguridad híbrida, puede integrar la protección con contraseña de Azure AD con un entorno de Active Directory local. Un componente instalado en el entorno local recibe la lista global de contraseñas prohibidas y las directivas de protección con contraseña personalizadas de Azure AD, y los controladores de dominio las usan para procesar los eventos de cambio de contraseña. Este enfoque híbrido garantiza que, con independencia de cómo o dónde los usuarios cambien sus credenciales, se aplicará el uso de contraseñas seguras.

## <a name="passwordless-authentication"></a>Autenticación sin contraseñas

En muchos entornos el objetivo final es eliminar el uso de contraseñas como parte de los eventos de inicio de sesión. Características como la protección con contraseña de Azure o Azure Multi-Factor Authentication ayudan a mejorar la seguridad, pero un nombre de usuario y una contraseña siguen siendo una forma débil de autenticación que se puede exponer o atacar por la fuerza bruta.

![Seguridad frente a comodidad con el proceso de autenticación que conduce a la autenticación sin contraseña](./media/concept-authentication-passwordless/passwordless-convenience-security.png)

Al iniciar sesión con un método sin contraseña, las credenciales se proporcionan mediante el uso de métodos como información biométrica en Windows Hello para empresas o una clave de seguridad FIDO2. Un atacante no puede duplicar fácilmente estos métodos de autenticación.

Azure AD ofrece formas de autenticación nativa sin contraseña con el fin de simplificar la experiencia de inicio de sesión de los usuarios y reducir el riesgo de ataques.

## <a name="next-steps"></a>Pasos siguientes

Para comenzar, consulte el [tutorial del autoservicio de restablecimiento de contraseña][tutorial-sspr] y [Azure Multi-Factor Authentication][tutorial-azure-mfa].

Para más información sobre los conceptos del autoservicio de restablecimiento de contraseña, consulte [Funcionamiento del autoservicio de restablecimiento de contraseña de Azure AD][concept-sspr].

Para más información sobre los conceptos de autenticación multifactor, consulte [Funcionamiento de Azure Multi-Factor Authentication][concept-mfa].

<!-- INTERNAL LINKS -->
[tutorial-sspr]: tutorial-enable-sspr.md
[tutorial-azure-mfa]: tutorial-enable-azure-mfa.md
[concept-sspr]: concept-sspr-howitworks.md
[concept-mfa]: concept-mfa-howitworks.md
