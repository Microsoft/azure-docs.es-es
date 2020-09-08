---
title: ¿En qué consiste la federación con Azure AD? | Microsoft Docs
description: Describe la federación con Azure AD.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 11/28/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: eb1c3bc9f89db3f4b694803a63293a5537d4b98b
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278759"
---
# <a name="what-is-federation-with-azure-ad"></a>¿En qué consiste la federación con Azure AD?

La federación es una colección de dominios que han establecido confianza. El nivel de confianza puede variar, pero normalmente incluye la autenticación y casi siempre la autorización. Una federación típica podría incluir un número de organizaciones que han establecido confianza para el acceso compartido a un conjunto de recursos.

Puede federar el entorno local con Azure AD y usar esta federación para la autenticación y autorización.  Este método de inicio de sesión garantiza que toda la autenticación de usuarios tiene lugar de forma local.  Este método permite a los administradores implementar niveles más rigurosos de control de acceso. Está disponible la federación con AD FS y PingFederate.

![Identidad federada](./media/whatis-hybrid-identity/federated-identity.png)


> [!TIP]
> Si opta por usar Federación con Servicios de federación de Active Directory (AD FS), tiene la posibilidad de configurar la sincronización de hash de contraseñas como copia de seguridad en caso de error en la infraestructura de AD FS.


## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la identidad híbrida?](./whatis-hybrid-identity.md)
- [¿Qué es Azure AD Connect y Connect Health?](whatis-azure-ad-connect.md)
- [¿Qué es la sincronización de hash de contraseñas?](whatis-phs.md)
- [¿Qué es la federación?](whatis-fed.md)
- [¿Qué es el inicio de sesión único?](how-to-connect-sso.md)
- [Cómo funciona la federación](how-to-connect-fed-whatis.md)
- [Federación con PingFederate](how-to-connect-install-custom.md#configuring-federation-with-pingfederate)