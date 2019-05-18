---
title: 'Agregar invitados de B2B sin un vínculo de invitación o de correo electrónico: Azure Active Directory | Microsoft Docs'
description: Puede permitir que un usuario invitado agregue otros usuarios invitados a Azure AD sin canjear una invitación en Colaboración de Azure Active Directory B2B.
services: active-directory
documentationcenter: ''
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 05/21/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: sasubram
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5db5eb9c0e0493d906345892fcc5f2872a3e0e14
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812454"
---
# <a name="add-b2b-collaboration-guest-users-without-an-invitation-link-or-email"></a>Agregar usuarios invitados de colaboración B2B sin un correo electrónico o un vínculo de invitación

Ahora puede invitar a usuarios mediante el envío de un vínculo directo a una aplicación compartida. Con este método, los usuarios invitados ya no necesitan usar el correo electrónico de invitación, excepto en algunos casos especiales. Un usuario invitado hace clic en el vínculo de la aplicación, revisa y acepta los términos de privacidad y, después, accede sin problemas a la aplicación. Para más información, consulte [Canje de invitación de colaboración B2B](redemption-experience.md).   

Antes de que este nuevo método estuviera disponible, podía invitar a los usuarios sin necesidad del correo electrónico de invitación al agregar un invitador (de su organización o de una organización asociada) al rol de directorio **Invitador de usuarios** y, después, hacer que el invitador agregara a los usuarios al directorio, los grupos o las aplicaciones a través de la interfaz de usuario o de PowerShell. (Si usa PowerShell, puede suprimir por completo el correo electrónico de invitación). Por ejemplo:

1. Un usuario de la organización anfitriona (por ejemplo, WoodGrove) invita a un usuario de la organización asociada (por ejemplo, Sam@litware.com) como invitado.
2. El administrador de la organización anfitriona [configura directivas](delegate-invitations.md) que permiten a Sam identificar y agregar otros usuarios de la organización asociada (Litware). (Debe agregarse a Sam al rol **Invitador de usuarios**).
3. Ahora, Sam puede agregar otros usuarios de Litware al directorio, los grupos o las aplicaciones de WoodGrove sin necesidad de canjear invitaciones. Si Sam tiene los privilegios de enumeración adecuados en Litware, se hará automáticamente.
 
Este método original sigue funcionando, pero hay una pequeña diferencia en el comportamiento. Si usa PowerShell, observará que una cuenta invitada ahora tiene un estado **PendingAcceptance** en lugar de mostrar inmediatamente como **Accepted**. Aunque el estado sea pendiente, el usuario invitado todavía puede iniciar sesión y acceder a la aplicación sin hacer clic en un vínculo de invitación por correo electrónico. El estado pendiente significa que el usuario aún no ha pasado la [experiencia de consentimiento](redemption-experience.md#privacy-policy-agreement), donde se aceptan los términos de privacidad de la organización que invita. El usuario invitado ve esta pantalla de consentimiento cuando inicia sesión por primera vez. 

Si invita a un usuario al directorio, el usuario invitado debe acceder directamente a la URL de Azure Portal específica de su inquilino de recursos (como https://portal.azure.com/*inquilinoderecursos*.onmicrosoft.com) para ver y aceptar los términos de privacidad.

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Canje de invitación de colaboración B2B](redemption-experience.md)
- [Delegación de invitaciones de la colaboración B2B de Azure Active Directory](delegate-invitations.md)
- [¿Cómo agregan los trabajadores de la información usuarios de colaboración B2B?](add-users-information-worker.md)

