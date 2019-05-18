---
title: 'Guía: Azure Active Directory de licencias de colaboración de B2B | Microsoft Docs'
description: La colaboración B2B de Azure Active Directory no necesita licencias de Azure AD de pago, pero también puede obtener características de pago para los usuarios invitados de B2B
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 10/04/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0d208e981f46bd5cc156dd7f764de64638c91efb
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65768174"
---
# <a name="azure-active-directory-b2b-collaboration-licensing-guidance"></a>Guía de concesión de licencias de colaboración B2B de Azure Active Directory

Con la colaboración negocio a negocio (B2B) de Azure Active Directory (Azure AD), puede invitar a usuarios externos (o "usuarios invitados") a usar sus servicios de Azure AD de pago. Por cada licencia de Azure AD de pago que asigne a un usuario, puede invitar hasta a cinco usuarios invitados con arreglo a la concesión de usuarios externos.

La licencia de los usuarios invitados B2B se calcula automáticamente y se notifica según la relación 1:5. Actualmente, no es posible asignar licencias de usuario invitado B2B directamente a los usuarios invitados.

Además, los usuarios invitados pueden usar características de Azure AD gratuitas sin requisitos de licencia adicionales. Los usuarios invitados tienen acceso a características gratuitas de Azure AD, incluso sin una licencia de Azure AD de pago. 

## <a name="examples-calculating-guest-user-licenses"></a>Ejemplos: Cálculo de licencias de usuario invitado
Una vez que determine cuántos usuarios invitados necesitan acceso a los servicios de Azure AD de pago, asegúrese de tener suficientes licencias de pago de Azure AD para los usuarios invitados en la proporción 1:5 requerida. A continuación se muestran algunos ejemplos:

- Quiere invitar a 100 usuarios invitados a las aplicaciones o servicios de Azure AD, y quiere asignar a todos los usuarios invitados administración de acceso y aprovisionamiento. También quiere exigir MFA y acceso condicional para 50 de esos usuarios invitados. Para satisfacer esta combinación, necesitará 10 licencias de Azure AD Basic y 10 licencias de Azure AD Premium P1. Si tiene previsto usar características de Identity Protection con los usuarios invitados, necesitará licencias de Azure AD Premium P2 en la misma proporción 1:5 para incluir a los usuarios invitados.
- Quiere invitar a 60 usuarios y todos ellos necesitan MFA, por lo que debe tener al menos 12 licencias de Azure AD Premium P1. Tiene 10 empleados con licencias de Azure AD Premium P1, lo que permitiría hasta 50 usuarios invitados en la proporción de licencia 1:5. Necesitará comprar dos licencias Premium P1 adicionales para incluir 10 usuarios invitados adicionales.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes recursos sobre la colaboración B2B de Azure AD:

* [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/)
* [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
* [Preguntas frecuentes sobre la colaboración B2B de Azure Active Directory (P+F)](faq.md)
