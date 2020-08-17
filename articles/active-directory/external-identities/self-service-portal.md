---
title: 'Portal de registro de autoservicio para la colaboración B2B: Azure AD'
description: La colaboración con Azure Active Directory B2B posibilita las relaciones entre empresas al permitir que los asociados empresariales accedan de forma selectiva a las aplicaciones corporativas.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: ff18d3d9cae6e887abbd9fb1845de62386062b2b
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907652"
---
# <a name="self-service-for-azure-ad-b2b-collaboration-sign-up"></a>Autoservicio para el registro en la colaboración B2B de Azure AD

Los clientes pueden hacer muchas cosas con las características integradas que se exponen mediante [Azure Portal](https://portal.azure.com) y el [panel de acceso a las aplicaciones](https://myapps.microsoft.com) para los usuarios finales. Sin embargo, puede que necesite personalizar el flujo de trabajo de incorporación para los usuarios de B2B para ajustarse a las necesidades de su organización.

## <a name="azure-ad-entitlement-management-for-b2b-guest-user-sign-up"></a>Administración de derechos de Azure AD para el registro de usuarios invitados de B2B

Como una organización que invita, puede que no sepa de antemano qué colaboradores externos individuales necesitan acceso a sus recursos. Necesita contar con una forma de que los usuarios de empresas asociadas se registren ellos mismos con unas directivas que usted controle. Si desea permitir que los usuarios de otras organizaciones soliciten acceso y, tras la aprobación, se aprovisionen con cuentas de invitado y se asignen a grupos, aplicaciones y sitios de SharePoint Online, puede usar la [administración de derechos de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-overview) para configurar directivas que [administren el acceso para usuarios externos](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-external-users#how-access-works-for-external-users).

## <a name="azure-active-directory-b2b-invitation-api"></a>API de invitación de Azure Active Directory B2B

Las organizaciones pueden usar la [API del administrador de invitaciones de Microsoft Graph](https://docs.microsoft.com/graph/api/resources/invitation?view=graph-rest-1.0) para crear sus propias experiencias de incorporación para los usuarios invitados de B2B. Si desea ofrecer el autoservicio de registro de usuarios invitados de B2B, le recomendamos que use la [administración de derechos de Azure AD](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-overview). Sin embargo, si desea crear su propia experiencia, puede usar la [API de creación de invitaciones](https://docs.microsoft.com/graph/api/invitation-post?view=graph-rest-1.0&tabs=http) para enviar de forma automática el correo electrónico con la invitación personalizada directamente al usuario de B2B, por ejemplo. O bien, la aplicación puede usar el valor de inviteRedeemUrl devuelto en la respuesta de creación para elaborar su propia invitación (a través del mecanismo de comunicación elegido) al usuario invitado.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
* [Concesión de licencias de colaboración B2B de Azure AD](licensing-guidance.md)
* [Preguntas frecuentes sobre la colaboración B2B de Azure Active Directory (P+F)](faq.md)
