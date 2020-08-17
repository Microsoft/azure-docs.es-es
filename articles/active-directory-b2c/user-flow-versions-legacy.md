---
title: Versiones heredadas de flujos de usuario de Azure Active Directory B2C | Microsoft Docs
description: Obtenga información sobre las versiones de los flujos de usuario disponibles en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 67ce6ab611e8d32c320320976ff2eba932f761a2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505556"
---
# <a name="legacy-user-flow-versions-in-azure-active-directory-b2c"></a>Versiones heredadas de flujos de usuario de Azure Active Directory B2C

> [!IMPORTANT]
> En este artículo, se describe el método que se utiliza para controlar las versiones heredadas de los flujos de usuario, que ofrece las versiones V1 (listas para producción) y las versiones V1.1 y V2 (versiones preliminares) de los flujos de usuario. Los entornos que no son de nube pública de Azure seguirán usando este método para controlar las versiones heredadas. En la nube pública de Azure, este método se está reemplazando por las [nuevas versiones **recomendadas** y las **versiones preliminares**](user-flow-versions.md).
> 
Los flujos de usuario de Azure Active Directory B2C (Azure AD B2C) le permiten configurar [directivas](user-flow-overview.md) que describan de forma exhaustiva la experiencia de los clientes con las identidades. Estas experiencias incluyen las operaciones de registro, inicio de sesión, restablecimiento de contraseña y edición de perfiles.

En la tabla siguiente, a menos que un flujo de usuario se identifique como **recomendado**, se considera que está en *versión preliminar*. Solo debe usar flujos de usuario recomendados en las aplicaciones de producción.

## <a name="v1"></a>V1

| Flujo de usuario | Recomendado | Descripción |
| --------- | ----------- | ----------- |
| Restablecimiento de contraseña | Sí | Permite a los usuarios elegir una contraseña nueva después de verificar el correo electrónico. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>Configuración de compatibilidad de tokens</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul> |
| Edición de perfiles | Sí | Permite al usuario configurar los atributos de usuario. Con este flujo de usuario, puede configurar: <ul><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li></ul> |
| Iniciar sesión mediante ROPC | No | Permite a los usuarios con una cuenta local iniciar sesión directamente en las aplicaciones nativas (no se necesita un explorador). Con este flujo de usuario, puede configurar: <ul><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li></ul> |
| Iniciar sesión | No | Permite a los usuarios iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>Bloqueo de inicios de sesión</li><li>Forzar el restablecimiento de contraseñas</li><li>Mantener la sesión iniciada (KMSI)</ul><br>No se puede personalizar la interfaz de usuario con este flujo de usuario. |
| Suscripción | No | Permite a los usuarios crear una cuenta. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul> |
| Registrarse e iniciar sesión | Sí | Permite a los usuarios crear una cuenta o iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul>|

## <a name="v11"></a>V1.1

| Flujo de usuario | Recomendado | Descripción |
| --------- | ----------- | ----------- |
| Restablecimiento de contraseña v1.1 | No | Permite al usuario elegir una nueva contraseña después de comprobar su correo electrónico (nuevo diseño de página disponible). Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>Configuración de compatibilidad de tokens</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul> |

## <a name="v2"></a>V2

| Flujo de usuario | Recomendado | Descripción |
| --------- | ----------- | ----------- |
| Restablecimiento de contraseña v2 | No | Permite a los usuarios elegir una contraseña nueva después de verificar el correo electrónico. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>Configuración de compatibilidad de tokens</li><li>[Restricción de acceso por edad](basic-age-gating.md)</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul> |
| Edición de perfil v2 | Sí | Permite al usuario configurar los atributos de usuario. Con este flujo de usuario, puede configurar: <ul><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li></ul> |
| Inicio de sesión v2 | No | Permite a los usuarios iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Restricción de acceso por edad](basic-age-gating.md)</li><li>Personalización de la página de inicio de sesión</li></ul> |
| Registrarse v2 | No | Permite a los usuarios crear una cuenta. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Restricción de acceso por edad](basic-age-gating.md)</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul> |
| Registrarse e iniciar sesión v2 | No | Permite a los usuarios crear una cuenta o iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](custom-policy-multi-factor-authentication.md)</li><li>[Restricción de acceso por edad](basic-age-gating.md)</li><li>[Requisitos de complejidad de la contraseña](user-flow-password-complexity.md)</li></ul> |