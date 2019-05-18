---
title: 'Acceso condicional para usuarios de colaboración B2B: Azure Active Directory | Microsoft Docs'
description: La colaboración B2B de Azure Active Directory admite Multi-Factor Authentication (MFA) para poder acceder de manera selectiva a las aplicaciones corporativas.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 09/11/2017
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1f3bfe067b7a927f800f88958ee2ffca09711c10
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812807"
---
# <a name="conditional-access-for-b2b-collaboration-users"></a>Acceso condicional para usuarios de colaboración B2B

## <a name="multi-factor-authentication-for-b2b-users"></a>Multi-Factor Authentication para usuarios B2B
Con la colaboración B2B de Azure AD, las organizaciones pueden exigir directivas de autenticación multifactor (MFA) para usuarios de B2B. Estas directivas se pueden exigir en el nivel de inquilino, aplicación o usuario individual, del mismo modo que pueden habilitarse para empleados a tiempo completo y miembros de la organización. Se aplican las directivas MFA en la organización de recursos.

Ejemplo:
1. El administrador o el trabajador de la información de la empresa A invita a un usuario de la empresa B a una aplicación *Foo* de la empresa A.
2. La aplicación *Foo* de la empresa A se configura para requerir MFA en el acceso.
3. Cuando el usuario de la empresa B intenta acceder a la aplicación *Foo* en el inquilino de la empresa A, se le pide que realice un desafío de MFA.
4. El usuario puede configurar su MFA con la empresa A y elegir su opción de MFA.
5. Este escenario funciona con cualquier identidad (Azure AD o MSA, por ejemplo, si los usuarios de la empresa B se autentican con un identificador social).
6. La empresa A debe tener suficientes licencias de Azure AD Premium que admitan MFA. El usuario de la empresa B consume esta licencia de la empresa A.

El inquilino que realiza la invitación siempre es responsable de MFA en los usuarios de la organización asociada, incluso si esta tiene funcionalidades MFA.

### <a name="setting-up-mfa-for-b2b-collaboration-users"></a>Configuración de Multi-Factor Authentication para los usuarios de colaboración B2B
Para descubrir lo fácil que es configurar MFA para los usuarios de colaboración B2B, vea el siguiente vídeo:

>[!VIDEO https://channel9.msdn.com/Blogs/Azure/b2b-conditional-access-setup/Player]

### <a name="b2b-users-mfa-experience-for-offer-redemption"></a>Experiencia de MFA de los usuarios B2B para el canje de ofertas
Consulte la siguiente animación para ver la experiencia de canje:

>[!VIDEO https://channel9.msdn.com/Blogs/Azure/MFA-redemption/Player]

### <a name="mfa-reset-for-b2b-collaboration-users"></a>Restablecimiento de la MFA para los usuarios de colaboración B2B
Actualmente, el administrador puede requerir que se vuelvan a probar los usuarios de colaboración B2B solo mediante los siguientes cmdlets de PowerShell.

1. Conectar con Azure AD

   ```
   $cred = Get-Credential
   Connect-MsolService -Credential $cred
   ```
2. Obtenga todos los usuarios con los métodos de prueba.

   ```
   Get-MsolUser | where { $_.StrongAuthenticationMethods} | select UserPrincipalName, @{n="Methods";e={($_.StrongAuthenticationMethods).MethodType}}
   ```
   Este es un ejemplo:

   ```
   Get-MsolUser | where { $_.StrongAuthenticationMethods} | select UserPrincipalName, @{n="Methods";e={($_.StrongAuthenticationMethods).MethodType}}
   ```

3. Restablezca el método MFA en un usuario específico para exigir que el usuario de colaboración B2B establezca de nuevo los métodos de prueba. Ejemplo:

   ```
   Reset-MsolStrongAuthenticationMethodByUpn -UserPrincipalName gsamoogle_gmail.com#EXT#@ WoodGroveAzureAD.onmicrosoft.com
   ```

### <a name="why-do-we-perform-mfa-at-the-resource-tenancy"></a>¿Por qué se realiza el MFA en el arrendamiento de recursos?

En la versión actual, MFA se encuentra siempre en el inquilino de recursos, por motivos de previsión. Por ejemplo, supongamos que un usuario de Contoso (Sally) está invitado a Fabrikam y Fabrikam ha habilitado MFA para usuarios de B2B.

Si Contoso tiene habilitada la directiva MFA para App1 pero no para App2, si examinamos la notificación de MFA de Contoso, podríamos ver el siguiente problema:

* Día 1: un usuario tiene MFA en Contoso y accede a App1; así que no se muestra ningún mensaje adicional de MFA en Fabrikam.

* Día 2: el usuario ha accedido a App 2 en Contoso, así que ahora, al acceder a Fabrikam, debe registrarse ahí en MFA.

Este proceso puede resultar confuso y podría dar lugar a que los inicios de sesión se queden sin terminar.

Además, aunque Contoso tenga la funcionalidad de MFA, no siempre Fabrikam confiará en la directiva de MFA de Contoso.

Por último, el MFA del inquilino de recursos también funciona en las MSA y los identificadores sociales, así como en las organizaciones asociadas que no tengan una configuración de MFA.

Por lo tanto, la recomendación de MFA en usuarios de B2B es exigir siempre MFA en el inquilino que invita. Este requisito podría dar lugar a MFA doble en algunos casos, pero cada vez que se accede al inquilino que invita, la experiencia de los usuarios finales es predecible: Sally debe registrarse en MFA con el inquilino que invita.

### <a name="device-based-location-based-and-risk-based-conditional-access-for-b2b-users"></a>Acceso condicional basado en el dispositivo, la ubicación y el riesgo para usuarios de B2B

Cuando Contoso habilita las directivas de acceso condicional basado en el dispositivo para sus datos corporativos, se impide el acceso desde dispositivos no administrados por Contoso y que no cumplen estas directivas de dispositivo.

Si el dispositivo del usuario de B2B no está administrado por Contoso, el acceso de usuarios de B2B de las organizaciones asociadas se bloqueará en el contexto en que se apliquen estas directivas. Sin embargo, Contoso puede crear listas de exclusión que contengan usuarios asociados específicos para excluirlos de la directiva de acceso condicional basado en el dispositivo.

#### <a name="location-based-conditional-access-for-b2b"></a>Acceso condicional basado en ubicación para B2B

Las directivas de acceso condicional basado en la ubicación se pueden aplicar a usuarios de B2B si la organización que invita no puede crear un intervalo de direcciones IP de confianza que defina sus organizaciones asociadas.

#### <a name="risk-based-conditional-access-for-b2b"></a>Acceso condicional basado en riesgos para B2B

Actualmente, no se pueden aplicar directivas de inicio de sesión basadas en el riesgo a los usuarios de B2B ya que la evaluación del riesgo se realiza en la organización principal del usuario de B2B.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

* [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
* [Concesión de licencias de colaboración B2B de Azure AD](licensing-guidance.md)
* [Preguntas frecuentes sobre la colaboración B2B de Azure Active Directory (P+F)](faq.md)
