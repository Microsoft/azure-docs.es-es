---
title: Introducción a la autenticación de Azure AD
description: Obtenga información sobre cómo acceder a la autenticación de Azure Active Directory (Azure AD) para usar la API de Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 00808c25ac84da852cce6169fb210767ee2b56cf
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89265888"
---
# <a name="get-credentials-to-access-media-services-api"></a>Obtención de credenciales para acceder a la API de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Al utilizar la autenticación de Azure AD para acceder a Azure Media Services API, tiene dos opciones de autenticación:

- **Autenticación de la entidad de servicio** (recomendada)

    Autenticar un servicio. Las aplicaciones que normalmente utilizan este método de autenticación son aplicaciones que ejecutan servicios de demonio, servicios de nivel intermedio o trabajos programados: Web Apps, Function Apps, Logic Apps, API o un microservicio.
- **Autenticación de usuarios**

    Autenticar a alguien que usa la aplicación para interactuar con los recursos de Media Services. La aplicación interactiva en primer lugar debe solicitar al usuario las credenciales. Un ejemplo es una aplicación de consola de administración que usan los usuarios autorizados para supervisar trabajos de codificación o streaming en vivo. 

En este artículo se describen los pasos para obtener las credenciales para acceder a la API de Media Services. Elija entre las pestañas siguientes.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure. Si no tiene cuenta, comience con una [evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 
- Una cuenta de Media Services. Para más información, vea [Creación de una cuenta de Azure Media Services mediante Azure Portal](create-account-howto.md).

## <a name="portal"></a>[Portal](#tab/portal/)

### <a name="api-access"></a>Acceso de API

La página **Acceso de API** le permite seleccionar el método de autenticación que desee usar para conectarse a la API. La página también proporciona los valores que necesita para conectarse a la API.

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Media Services.
2. Seleccione la hoja **API access (new)** (Acceso a API [nuevo]) en la barra de navegación izquierda.
3. En **Conectar con la API de Media Services**, seleccione la versión de Media Services API a la que desea conectarse (la versión 3 es la más reciente del servicio).

### <a name="service-principal-authentication--recommended"></a>Autenticación de la entidad de servicio (recomendada)

Autentica un servicio usando una aplicación de Azure Active Directory (Azure AD) y un secreto. Esto se recomienda para cualquier servicio de nivel medio que llame a Media Services API. Algunos ejemplos son Web Apps, Functions, Logic Apps, las API y los microservicios. Este es el método de autenticación recomendado.

#### <a name="manage-your-azure-ad-app-and-secret"></a>Administración del secreto y la aplicación de Azure AD

La sección **Administración del secreto y la aplicación de AAD** le permite seleccionar o crear una nueva aplicación de Azure AD y generar un secreto. Por motivos de seguridad, el secreto no se puede mostrar una vez cerrada la hoja. La aplicación usa el identificador y el secreto de la aplicación para la autenticación con el fin de obtener un token válido para Media Services.

Asegúrese de que tiene permisos suficientes para registrar una aplicación con el inquilino de Azure AD y de que la asigna a un rol en la suscripción de Azure. Para más información, consulte los [permisos necesarios](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

#### <a name="connect-to-media-services-api"></a>Conexión con Media Services API

La opción **Conectar con la API de Media Services** proporciona los valores que se usan para conectar la aplicación de la entidad de servicio. Puede obtener valores de texto o copiar los bloques JSON o XML.

### <a name="user-authentication"></a>Autenticación de usuarios

Esta opción puede usarse para autenticar a un empleado o miembro de una instancia de Azure Active Directory que use una aplicación para interactuar con los recursos de Media Services. La aplicación interactiva, en primer lugar, debe solicitar al usuario las credenciales. Este método de autenticación solo debe usarse para las aplicaciones de administración.

#### <a name="connect-to-media-services-api"></a>Conexión con Media Services API

Copie sus credenciales para conectar la aplicación de usuario desde la sección **Conectar con la API de Media Services**. Puede obtener valores de texto o copiar los bloques JSON o XML.

## <a name="cli"></a>[CLI](#tab/cli/)

[!INCLUDE [media-services-v3-cli-access-api-include](../../../includes/media-services-v3-cli-access-api-include.md)]

---

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: Carga, codificación y streaming de vídeos con la versión 3 de Media Services](stream-files-tutorial-with-api.md).
