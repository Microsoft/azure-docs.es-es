---
title: ¿Qué son las condiciones en el acceso condicional de Azure Active Directory? | Microsoft Docs
description: Obtenga información sobre cómo se usan las condiciones en el acceso condicional de Azure Active Directory para desencadenar una directiva.
services: active-directory
keywords: acceso condicional a aplicaciones, acceso condicional con Azure AD, acceso seguro a recursos de empresa, directivas de acceso condicional
documentationcenter: ''
author: MicrosoftGuyJFlo
manager: daveba
editor: ''
ms.assetid: 8c1d978f-e80b-420e-853a-8bbddc4bcdad
ms.service: active-directory
ms.subservice: conditional-access
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/14/2018
ms.author: joflore
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9da23b0c0b0b0c0bfc238b1504811a9c1c55a9ef
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785377"
---
# <a name="what-are-conditions-in-azure-active-directory-conditional-access"></a>¿Qué son las condiciones en el acceso condicional de Azure Active Directory? 

Puede controlar el modo en que los usuarios acceden a las aplicaciones en la nube con el [acceso condicional de Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal). En una directiva de acceso condicional, definirá la respuesta ("haga esto") al motivo para desencadenarla ("cuando esto suceda"). 

![Motivo y respuesta](./media/conditions/10.png)


En el contexto del acceso condicional, **Cuando esto sucede** se denomina una **condición**. **Entonces haga esto** es un **control de acceso**. La combinación de las condiciones y los controles de acceso representa una directiva de acceso condicional.

![Directiva de acceso condicional](./media/conditions/61.png)


Las condiciones que no haya configurado en una directiva de acceso condicional no se aplican. Algunas condiciones son [obligatorias](best-practices.md) para poder aplicar una directiva de acceso condicional a un entorno. 

En este artículo se ofrece información general de las condiciones y de cómo se utilizan en una directiva de acceso condicional. 

## <a name="users-and-groups"></a>Usuarios y grupos

La condición de usuarios y grupos es obligatoria en una directiva de acceso condicional. En la directiva, puede seleccionar **Todos los usuarios** o bien usuarios y grupos específicos.

![Usuarios y grupos](./media/conditions/111.png)

Si selecciona **Todos los usuarios**, la directiva se aplica a todos los usuarios del directorio, incluidos los usuarios invitados.

Al **Seleccionar usuarios y grupos**, puede establecer las siguientes opciones:

* **Todos los usuarios invitados** permite tener como destino una directiva para usuarios invitados de B2B. Esta condición coincide con cualquier cuenta de usuario que tenga el atributo **userType** establecido en **Invitado**. Use esta configuración en caso de que tenga que aplicar una directiva tan pronto como se cree una cuenta en un flujo de invitación de Azure AD.

* **Roles de directorio** permite tener como destino una directiva según la asignación de roles del usuario. Esta condición admite roles de directorio como **Administrador global** o **Administrador de contraseñas**.

* Los **Usuarios y grupos** tienen como destino conjuntos específicos de usuarios. Por ejemplo, puede seleccionar un grupo que contenga todos los miembros del departamento de Recursos Humanos cuando tenga una aplicación de Recursos Humanos seleccionada como aplicación en la nube. Un grupo puede ser cualquier tipo de grupo en Azure AD, como grupos de distribución y de seguridad dinámicos o asignados.

También puede excluir determinados usuarios o grupos de una directiva. Un caso de uso común son las cuentas de servicio si su directiva aplica la autenticación multifactor (MFA).

Dirigirse a grupos específicos de usuarios es útil para la implementación de una nueva directiva. En una nueva directiva, debería dirigir solo un conjunto inicial de usuarios para validar el comportamiento de la directiva.

## <a name="cloud-apps-and-actions"></a>Aplicaciones en la nube y acciones

Una aplicación de nube es un sitio Web, servicio o punto de conexión protegida por Azure AD Application Proxy. Para obtener una descripción detallada de las aplicaciones en la nube admitidas, consulte la sección [Asignaciones de aplicaciones en la nube](technical-reference.md#cloud-apps-assignments). El **en la nube aplicaciones o acciones** condición es obligatoria en una directiva de acceso condicional. En la directiva, puede seleccionar **todas las aplicaciones en la nube** o especificar las aplicaciones con **seleccionar aplicaciones**.

Las organizaciones pueden elegir entre las siguientes opciones:

* **Todas las aplicaciones en la nube** al aplicar las directivas de línea de base se aplican a toda la organización. Use esta selección para las directivas que requieren la autenticación multifactor cuando se detecta el riesgo de inicio de sesión para cualquier aplicación en la nube. Se aplica una directiva se aplica a todas las aplicaciones de nube para el acceso a todos los sitios Web y servicios. Esta configuración no se limita a las aplicaciones de nube que aparecen en la lista de aplicaciones seleccione.
* **Seleccione aplicaciones** para dirigir servicios específicos por su directiva. Por ejemplo, puede requerir que los usuarios tengan un dispositivo compatible para tener acceso a SharePoint Online. Esta directiva también se aplica a otros servicios cuando se accede al contenido de SharePoint. Un ejemplo es Microsoft Teams.

> [!NOTE]
> Puede excluir aplicaciones específicas de una directiva. Sin embargo, estas aplicaciones siguen estando sujetas a las directivas aplicadas a los servicios a los que acceden.

**Las acciones del usuario** son tareas que se pueden realizar por el usuario. La única acción admitida actualmente es **registrar información de seguridad (versión preliminar)**, lo que permite la directiva de acceso condicional exigir cuando un usuario registra su información de seguridad.

## <a name="sign-in-risk"></a>Riesgo de inicio de sesión

El riesgo del inicio de sesión es un indicador de la probabilidad (alta, media o baja) que hay de que el legítimo propietario de una cuenta de usuario no haya realizado un inicio de sesión. Azure AD calcula el nivel de riesgo de inicio de sesión durante el inicio de sesión de un usuario. Puede utilizar el nivel calculado de riesgo de inicio de sesión como condición en una directiva de acceso condicional.

![Niveles de riesgo de inicio de sesión](./media/conditions/22.png)

Para usar esta condición, debe tener [Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection-enable) habilitado.
 
Los casos de uso comunes para esta condición son directivas que tienen las protecciones siguientes: 

- Bloquear a los usuarios con un riesgo alto de inicio de sesión. Esta protección impide que usuarios potencialmente no legítimos accedan a sus aplicaciones en la nube. 
- Requieren la autenticación multifactor para aquellos usuarios con un riesgo medio de inicio de sesión. Al aplicar la autenticación multifactor, puede proporcionar más confianza de que el inicio de sesión lo realiza el propietario legítimo de una cuenta.

Para obtener más información, consulte [Bloqueo del acceso si se detecta un riesgo de sesión](app-sign-in-risk.md).  

## <a name="device-platforms"></a>Plataformas de dispositivo

La plataforma de dispositivo se caracteriza por el sistema operativo que se ejecuta en el dispositivo. Azure AD identifica la plataforma mediante el uso de la información proporcionada por el dispositivo, como el agente de usuario. Esta información no está verificada. Se recomienda que todas las plataformas tengan una directiva aplicada a ellas. La directiva debe bloquear el acceso, requerir el cumplimiento de las directivas de Microsoft Intune o requerir que el dispositivo esté unido a un dominio. El valor predeterminado es aplicar una directiva a todas las plataformas de dispositivo. 


![Configuración de plataformas de dispositivos](./media/conditions/24.png)

Para obtener una lista de las plataformas de dispositivos compatibles, vea [Condición de plataforma de dispositivos](technical-reference.md#device-platform-condition).


Un caso de uso común para esta condición es una directiva que restringe el acceso a las aplicaciones en la nube a [dispositivos administrados](require-managed-devices.md). Para conocer más escenarios, como la condición de la plataforma de dispositivo, consulte [Acceso condicional basado en aplicaciones de Azure Active Directory](app-based-conditional-access.md).



## <a name="device-state"></a>Estado del dispositivo

La condición de estado del dispositivo excluye de una política de acceso condicional los dispositivos híbridos unidos a Azure AD y aquellos que estén marcados como compatibles. 


![Configuración del estado del dispositivo](./media/conditions/112.png)

Esta condición es útil cuando la directiva solo debe aplicarse a un dispositivo no administrado para proporcionar seguridad adicional a la sesión. Por ejemplo, aplique el control de sesión de Microsoft Cloud App Security únicamente cuando un dispositivo no esté administrado. 

## <a name="locations"></a>Ubicaciones

Mediante el uso de ubicaciones, puede definir condiciones en función de dónde se intentó realizar una conexión. 

![Configuración de ubicaciones](./media/conditions/25.png)

Los casos de uso comunes para esta condición son directivas con las protecciones siguientes:

- Requieren la autenticación multifactor a los usuarios que accedan a un servicio desde fuera de la red corporativa.  

- Bloquean el acceso a los usuarios que accedan a un servicio desde determinados países o regiones. 

Para más información, vea [¿Qué es la condición de ubicación en el acceso condicional de Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-locations)


## <a name="client-apps"></a>Aplicaciones cliente

De forma predeterminada, una directiva de acceso condicional se aplica a las siguientes aplicaciones:

- **[Aplicaciones del explorador](technical-reference.md#supported-browsers)**: entre las aplicaciones del explorador se incluyen sitios web que usan los protocolos de inicio de sesión único web de SAML, WS-Federation u OpenID Connect. Esto también se aplicará a cualquier sitio web o servicio web que se haya registrado como un cliente confidencial de OAuth. Por ejemplo, el sitio web de Office 365 SharePoint. 

- **[Aplicaciones móviles y de escritorio que usan la autenticación moderna](technical-reference.md#supported-mobile-applications-and-desktop-clients)**: entre estas aplicaciones se incluyen las aplicaciones de escritorio de Office y aplicaciones telefónicas. 


Además, puede destinar una directiva a aplicaciones cliente específicas que no usan la autenticación moderna, por ejemplo:

- **[Clientes de Exchange ActiveSync](conditions.md#exchange-activesync-clients)**: cuando una directiva bloquea el uso de Exchange ActiveSync, los usuarios afectados reciben un solo correo electrónico de cuarentena con información sobre la razón de dicho bloqueo. Si es necesario, en el correo electrónico se incluyen instrucciones para inscribir su dispositivo con Intune.

- **[Otros clientes](block-legacy-authentication.md)**: estas aplicaciones incluyen clientes que usan la autenticación básica con protocolos de correo electrónico como IMAP, MAPI, POP, SMTP y aplicaciones de Office más antiguas que no usan la autenticación moderna. Para obtener más información, consulte [Cómo funciona la autenticación moderna para las aplicaciones de cliente de Office 2013 y Office 2016](https://docs.microsoft.com/office365/enterprise/modern-auth-for-office-2013-and-2016).

![Aplicaciones cliente](./media/conditions/41.png)

Los casos de uso comunes para esta condición son directivas con los requisitos siguientes:

- **[Requieren un dispositivo administrado](require-managed-devices.md)** para aplicaciones móviles y de escritorio que descarguen datos en un dispositivo. Al mismo tiempo, permiten el acceso a través del explorador desde cualquier dispositivo. Este escenario evita que se guarden y sincronicen documentos en un dispositivo no administrado. Con este método, puede reducir la probabilidad de pérdida de datos en caso de que se pierda o se robe el dispositivo.

- **[Requieren un dispositivo administrado](require-managed-devices.md)** para aplicaciones que usan ActiveSync para tener acceso a Exchange Online.

- **[Bloquean la autenticación heredada](block-legacy-authentication.md)** a Azure AD (otros clientes)

- Bloquean el acceso de aplicaciones web, pero permiten el acceso desde aplicaciones de escritorio y móviles.



### <a name="exchange-activesync-clients"></a>Clientes de Exchange ActiveSync

Solo puede seleccionar **clientes de Exchange ActiveSync** si:


- Microsoft Office 365 Exchange Online es la única aplicación en la nube que ha seleccionado.

    ![Aplicaciones en la nube](./media/conditions/32.png)

- No tiene otras condiciones configuradas en una directiva. Sin embargo, puede restringir el ámbito de esta condición para que solo se aplique a las [plataformas compatibles](technical-reference.md#device-platform-condition).
 
    ![Aplicar directiva solo en las plataformas compatibles](./media/conditions/33.png)


Al bloquearse el acceso porque se requiere un [dispositivo administrado](require-managed-devices.md), los usuarios afectados reciben un solo correo electrónico que les guía en el uso de Intune. 

Si se requiere una aplicación aprobada, los usuarios afectados obtienen directrices para instalar y usar el cliente móvil de Outlook.

En otros casos, por ejemplo, si se requiere MFA, se bloquean los usuarios afectados, ya que los clientes que usan la autenticación básica no admiten MFA.

Solo puede dirigir esta configuración a usuarios y grupos. No admite invitados ni roles. Si se configura la condición de invitado o de rol, todos los usuarios se bloquearán, ya que el acceso condicional no puede determinar si la directiva debe aplicarse al usuario, o no.


 Para más información, consulte:

- [Configuración de SharePoint Online y Exchange Online para el acceso condicional de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-no-modern-authentication)
 
- [Acceso condicional basado en aplicaciones de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/conditional-access/app-based-conditional-access) 



## <a name="next-steps"></a>Pasos siguientes

- Para averiguar cómo configurar una directiva de acceso condicional, consulte [Guía de inicio rápido: Requerir MFA para aplicaciones específicas con acceso condicional a Azure Active Directory](app-based-mfa.md).

- Para configurar directivas de acceso condicional para su entorno, vea [Procedimientos recomendados para el acceso condicional en Azure Active Directory](best-practices.md). 

