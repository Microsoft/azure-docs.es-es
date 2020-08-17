---
title: Creación de un inquilino de Azure Active Directory
description: Obtenga más información acerca de cómo crear un inquilino de Azure AD para el registro y la creación de aplicaciones.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: quickstart
ms.date: 03/12/2020
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev, identityplatformtop40, fasttrack-edit
ms.openlocfilehash: df873e49e1bc581a32bcb4e4b40b2b24db98cee5
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120820"
---
# <a name="quickstart-set-up-a-tenant"></a>Inicio rápido: Configuración de un inquilino

La plataforma de identidad de Microsoft permite a los desarrolladores crear aplicaciones destinadas a una amplia variedad de identidades y entornos de Microsoft 365 personalizados. Para comenzar a usar la plataforma de identidad de Microsoft, deberá acceder a un entorno, también denominado inquilino de Azure AD, que pueda registrar y administrar aplicaciones, tendrá acceso a datos de Microsoft 365 e implementará restricciones personalizadas de inquilino y acceso condicional.

Un inquilino es la representación de una organización. Se trata de una instancia dedicada de Azure AD que una organización o el desarrollador de la aplicación reciben cuando crean una relación con Microsoft, como al registrarse en Azure, Microsoft Intune o Microsoft 365.

Cada inquilino de Azure AD es distinto e independiente de los demás inquilinos de Azure AD y tiene su propia representación de identidades profesionales y educativas, identidades de consumidor (si es un inquilino de Azure AD B2C) y registros de aplicaciones. Un registro de aplicación dentro del inquilino puede permitir autenticaciones de cuentas solo dentro de su inquilino o en todos los inquilinos.

## <a name="determining-environment-type"></a>Determinación del tipo de entorno

Hay dos tipos de entornos que puede crear. Para decidir cuál necesita, debe basarse únicamente en los tipos de usuarios que autenticará la aplicación.

* Cuentas profesionales o educativas (cuentas de Azure AD) o cuentas de Microsoft (como outlook.com y live.com)
* Cuentas locales y sociales (Azure AD B2C)

La guía de inicio rápido se divide en dos escenarios según el tipo de aplicación que desea crear.

## <a name="work-and-school-accounts-or-personal-microsoft-accounts"></a>Cuentas profesionales o educativas o cuentas personales de Microsoft

### <a name="use-an-existing-tenant"></a>Uso de un inquilino existente

Muchos desarrolladores ya tienen inquilinos mediante servicios o suscripciones que están vinculados a inquilinos de Azure AD, como suscripciones de Microsoft 365 o Azure.

1. Para comprobar el inquilino, inicie sesión en [Azure Portal](https://portal.azure.com) con la cuenta que desea usar para administrar la aplicación.
1. Compruebe la esquina superior derecha. Si tiene un inquilino, iniciará sesión de forma automática y verá el nombre de este directamente bajo el nombre de la cuenta.
   * Mantenga el puntero sobre el nombre de su cuenta en el lado superior derecho de Azure Portal para ver el nombre, el correo electrónico, el directorio e identificador de inquilino (un GUID) y el dominio.
   * Si la cuenta está asociada a varios inquilinos, puede seleccionar el nombre de la cuenta para abrir un menú donde puede cambiar entre los inquilinos. Cada inquilino tiene su propio identificador de inquilino.

> [!TIP]
> Si necesita encontrar el identificador del inquilino, puede:
> * Mantener el puntero sobre el nombre de cuenta para obtener el directorio o el identificador de inquilino.
> * Seleccionar **Azure Active Directory > Propiedades > Id. de directorio** en Azure Portal.

Si no tiene un inquilino existente asociado a la cuenta, verá un GUID bajo el nombre de la cuenta y no podrá realizar acciones como el registro de aplicaciones hasta que siga los pasos de la siguiente sección.

### <a name="create-a-new-azure-ad-tenant"></a>Creación de un nuevo inquilino de Azure AD

Si aún no tiene un inquilino de Azure AD o si desea crear uno nuevo para su desarrollo, consulte el [inicio rápido](../fundamentals/active-directory-access-create-new-tenant.md) o simplemente siga la [experiencia de creación de directorio](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory). Tendrá que proporcionar la información siguiente para crear el nuevo inquilino:

- **Nombre de la organización**
- **Dominio inicial**: formará parte de *.onmicrosoft.com. El dominio se puede personalizar más adelante.
- **País o región**

> [!NOTE]
> Al asignar un nombre a su inquilino, utilice caracteres alfanuméricos. No se aceptan caracteres especiales. El nombre no puede tener más de 256 caracteres.

## <a name="social-and-local-accounts"></a>Cuentas locales y sociales

Para empezar a crear aplicaciones que inician sesión en cuentas sociales y locales, deberá crear a un inquilino de Azure AD B2C. Para empezar, siga los pasos que se indican en [Creación de un inquilino de Azure AD B2C](../../active-directory-b2c/tutorial-create-tenant.md).

## <a name="next-steps"></a>Pasos siguientes

* [Registre una aplicación](quickstart-register-app.md) e integre con la Plataforma de identidad de Microsoft. 
* Conozca los [fundamentos de la autenticación](./authentication-vs-authorization.md).
* Consulte [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](../fundamentals/active-directory-how-subscriptions-associated-directory.md) para más información sobre la relación entre las suscripciones y un inquilino de Azure AD.