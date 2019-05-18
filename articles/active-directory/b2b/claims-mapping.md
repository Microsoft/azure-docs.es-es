---
title: Asignación - de notificaciones de usuario de colaboración B2B de Azure Active Directory | Microsoft Docs
description: Personalice las notificaciones de usuario que se emiten en el token SAML para los usuarios B2B de Azure Active Directory (Azure AD).
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/06/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: edb18c31f40de3358ad987be4a9c67ed3a5079e8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65811926"
---
# <a name="b2b-collaboration-user-claims-mapping-in-azure-active-directory"></a>Asignación de notificaciones de usuario de colaboración B2B de Azure Active Directory

Azure Active Directory (Azure AD) admite la personalización de las notificaciones emitidas en el token SAML para los usuarios de colaboración B2B. Cuando un usuario se autentique en la aplicación, Azure AD emitirá un token SAML a la aplicación que contiene información (o notificaciones) sobre el usuario que lo identifica de forma única. De forma predeterminada, dicha información incluye el nombre de usuario, la dirección de correo electrónico, el nombre y los apellidos del usuario.

En [Azure Portal](https://portal.azure.com) puede ver o editar las notificaciones que se envían en el token SAML a la aplicación. Para acceder a la configuración, seleccione **Azure Active Directory** > **Aplicaciones empresariales** > la aplicación que está configurada para el inicio de sesión único > **Inicio de sesión único**. Consulte la configuración del token SAML en la sección **Atributos del usuario**.

![Muestra los atributos del token SAML en la interfaz de usuario](media/claims-mapping/view-claims-in-saml-token.png)

Hay dos razones posibles por las que puede que tenga que editar las notificaciones que se emiten en el token SAML:

1. La aplicación requiere un conjunto diferente de URI o valores de notificación.

2. La aplicación requiere que la notificación NameIdentifier tenga un valor que no sea el del nombre principal de usuario (UPN) almacenado en Azure AD.

Para más información acerca de cómo agregar y editar notificaciones, consulte [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](../develop/active-directory-saml-claims-customization.md).

Para los usuarios de colaboración B2B, por motivos de seguridad, se evita realizar la asignación de NameID y UPD entre inquilinos.

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de las propiedades de usuario de colaboración B2B, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](user-properties.md).
- Para más información acerca de los tokens de usuario para los usuarios de colaboración B2B, consulte [Información sobre los tokens de usuario de la colaboración B2B de Azure Active Directory](user-token.md).

