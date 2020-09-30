---
title: 'Registro de una aplicación web que llama a las API web: Plataforma de identidad de Microsoft | Azure'
description: Aprenda a registrar una aplicación web que llama a las API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: f94a3da96243e30faa90277ce86efec037f54672
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89436480"
---
# <a name="a-web-app-that-calls-web-apis-app-registration"></a>Aplicación web que llama a las API web: Registro de aplicación

Una aplicación web que llama a las API web que tiene el mismo registro que una aplicación web que permite iniciar sesión a los usuarios. Por lo tanto, siga las instrucciones de [Aplicación web que permite iniciar sesión a los usuarios: registro de aplicación](scenario-web-app-sign-user-app-registration.md).

Sin embargo, dado que la aplicación web ahora llama también a las API web, se convierte en una aplicación cliente confidencial. Este es el motivo por el que se requiere algún registro adicional. La aplicación debe compartir las credenciales del cliente o los *secretos*, con la Plataforma de identidad de Microsoft.

[!INCLUDE [Registration of client secrets](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>Permisos de API

Las aplicaciones web llaman a las API en nombre del usuario con sesión iniciada. Para ello, deben solicitar *permisos delegados*. Para más información, consulte [Adición de permisos para acceder a la API web](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aplicación web que llama a las API web: configuración del código](scenario-web-app-call-api-app-configuration.md)
