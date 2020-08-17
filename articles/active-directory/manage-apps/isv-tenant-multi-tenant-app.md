---
title: Crear un inquilino de Azure para una aplicación multiinquilino
description: Guía para proveedores de software independientes sobre la integración con Azure Active Directory
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 05/22/2019
ms.author: kenwith
ms.reviewer: jeeds
ms.collection: M365-identity-device-management
ms.openlocfilehash: 264d4afb50904be17624f8801b7b79fe4b232030
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386816"
---
# <a name="create-an-azure-tenant-for-a-multi-tenant-application"></a>Crear un inquilino de Azure para una aplicación multiinquilino  

Para proporcionar acceso a la aplicación multiinquilino, debe crear un inquilino de Azure Active Directory a fin de registrar la aplicación y habilitar la federación de identidades del cliente. Vea [Selección del protocolo de federación correcto para una aplicación multiinquilino](isv-choose-multi-tenant-federation.md). Este inquilino permite probar la aplicación y la federación en un entorno similar a los entornos de Azure AD de los clientes.

## <a name="costs-of-hosting-a-multi-tenant-application"></a>Costos de hospedaje de una aplicación multiinquilino

Azure Active Directory está disponible en varias ediciones. [Vea la comparación de características detallada](https://azure.microsoft.com/pricing/details/active-directory/).

Puede crear la suscripción de Azure y Azure Active Directory de forma gratuita y usar características básicas.

## <a name="create-your-tenant"></a>Crear el inquilino

1. Cree el inquilino. Vea [Configuración de un entorno de desarrollo](../develop/quickstart-create-new-tenant.md).

2. Habilite y pruebe el acceso de inicio de sesión único a la aplicación,

   a. **En aplicaciones de OIDC u Oath**, [registre la aplicación](../develop/quickstart-register-app.md) como una aplicación multiinquilino. Seleccione la opción Cuentas en cualquier directorio organizativo y cuentas Microsoft personales en Tipos de cuenta admitidos.

   b. **En aplicaciones basadas en SAML y WS-Fed**, [configure el inicio de sesión único basado en SAML](configure-saml-single-sign-on.md) mediante una plantilla genérica de SAML en Azure AD.

También puede [convertir una aplicación de un solo inquilino en multiinquilino](../develop/howto-convert-app-to-be-multi-tenant.md) si fuera necesario.

## <a name="next-steps"></a>Pasos siguientes

[Integración del inicio de sesión único en la aplicación](isv-sso-content.md)
