---
title: 'Guía de concesión de licencias de colaboración B2B: Azure Active Directory | Microsoft Docs'
description: La colaboración B2B de Azure Active Directory no necesita licencias de Azure AD de pago, pero también puede obtener características de pago para los usuarios invitados de B2B
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 08/29/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 28d02160163da4081ad8adbe233b27fee970a0df
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907684"
---
# <a name="azure-active-directory-b2b-collaboration-licensing-guidance"></a>Guía de concesión de licencias de colaboración B2B de Azure Active Directory

Con la colaboración negocio a negocio (B2B) de Azure Active Directory (Azure AD), puede invitar a usuarios externos (o "usuarios invitados") a usar sus servicios de Azure AD de pago. Algunas características son gratuitas, pero para las características de Azure AD de pago, puede invitar hasta cinco usuarios invitados por cada licencia de la edición Azure AD que posea para un empleado o un usuario que no sea invitado en el inquilino.

> [!NOTE]
> Consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/) para más información sobre los precios de Azure AD y las características de colaboración B2B.

La licencia de los usuarios invitados B2B se calcula automáticamente y se notifica según la relación 1:5. 

Además, los usuarios invitados pueden usar características de Azure AD gratuitas sin requisitos de licencia adicionales. Los usuarios invitados tienen acceso a características gratuitas de Azure AD, incluso sin una licencia de Azure AD de pago. 

## <a name="examples-calculating-guest-user-licenses"></a>Ejemplos: Cálculo de licencias de usuario invitado
Una vez que determine cuántos usuarios invitados necesitan acceso a los servicios de Azure AD de pago, asegúrese de tener suficientes licencias de pago de Azure AD para los usuarios invitados en la proporción 1:5 requerida. Estos son algunos ejemplos:

- Quiere invitar a 100 usuarios a sus aplicaciones o servicios de Azure AD y proporcionarles administración del acceso y aprovisionamiento. Asimismo, para 50 de esos usuarios invitados quiere solicitar MFA y acceso condicional. Por lo tanto, para poder usar esas características necesitará 10 licencias de Azure AD Premium P1. Si tiene previsto usar características de Identity Protection con los usuarios invitados, necesitará licencias de Azure AD Premium P2 en la misma proporción 1:5 para incluir a los usuarios invitados.
- Quiere invitar a 60 usuarios y todos ellos necesitan MFA, por lo que debe tener al menos 12 licencias de Azure AD Premium P1. Tiene 10 empleados con licencias de Azure AD Premium P1, lo que permitiría hasta 50 usuarios invitados en la proporción de licencia 1:5. Necesitará comprar dos licencias Premium P1 adicionales para incluir 10 usuarios invitados adicionales.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes recursos sobre la colaboración B2B de Azure AD:

* [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/)
* [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
* [Preguntas frecuentes sobre la colaboración B2B de Azure Active Directory (P+F)](faq.md)
