---
title: 'Ejemplos de código de conectores de API para flujos de usuario: Azure AD'
description: Ejemplos de código para conectores de API en flujos de registro de autoservicio para identidades externas de Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: sample
ms.date: 06/16/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a3995395cfd6256689bedc7a4a3c83effc65c0b4
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907049"
---
# <a name="samples-for-external-identities-self-service-sign-up"></a>Ejemplos de registro de autoservicio de identidades externas

En las tablas siguientes se proporcionan vínculos a ejemplos de código para aprovechar las API web en los flujos de usuario de registro de autoservicio mediante [conectores de API](api-connectors-overview.md).

## <a name="api-connector-azure-function-quickstarts"></a>Guías de inicio rápido de Azure Functions para conectores de API

| Muestra                                                                                                                          | Descripción                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-api-connector-azure-function-validate) | Este ejemplo de Azure Functions con .NET Core muestra cómo limitar los registros a dominios de inquilino específicos y validar la información proporcionada por el usuario. |
| [Node.js](https://github.com/Azure-Samples/active-directory-nodejs-external-identities-api-connector-azure-function-validate)   | Este ejemplo de Azure Functions con Node.js muestra cómo limitar los registros a dominios de inquilino específicos y validar la información proporcionada por el usuario.  |
| [Python](https://github.com/Azure-Samples/active-directory-python-external-identities-api-connector-azure-function-validate)    | Este ejemplo de Azure Functions con Python muestra cómo limitar los inicios de sesión a dominios de inquilino específicos y validar la información proporcionada por el usuario.    |

<!-- \| [Java](../../azure-docs-pr/articles/active-directory/b2b/invite-internal-users.md#use-the-invitation-api-to-send-a-b2b-invitation) |  The sample below illustrates how to call the invitation API to invite an internal user as a B2B user. | -->

## <a name="custom-approval-workflows"></a>Flujos de trabajo de aprobación personalizados

| Muestra | Descripción |
|--------| ----------- |
| [Flujo de trabajo de aprobación manual](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-api-connectors-approvals) | Este ejemplo muestra un flujo de trabajo de aprobación de un extremo a otro para administrar la creación de cuentas de usuario invitado en el registro de autoservicio |

## <a name="identity-verification"></a>Verificación de identidad

| Muestra                                                                                                            | Descripción                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [IDology](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-idology-identity-verification) | En este ejemplo se muestra cómo comprobar una identidad de usuario como parte de su suscripción de autoservicio mediante un conector de API para integrarse con IDology. |
| [Experian](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-experian-identity-verification) | En este ejemplo se muestra cómo comprobar una identidad de usuario como parte de su registro de autoservicio mediante un conector de API para integrarse con Experian. |
