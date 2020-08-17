---
title: Introducción a la autenticación de Azure AD mediante Azure Portal | Microsoft Docs
description: Obtenga información sobre cómo usar Azure Portal para acceder a la autenticación de Azure Active Directory (Azure AD) para utilizar la API de Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 208019bbc86b22443c7d012400d4e4324679aee9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084473"
---
# <a name="get-started-with-azure-ad-authentication-by-using-the-azure-portal"></a>Introducción a la autenticación de Azure AD mediante Azure Portal

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Obtenga información sobre cómo usar Azure Portal para acceder a la autenticación de Azure Active Directory (Azure AD) para tener acceso a la API de Azure Media Services.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure. Si no tiene cuenta, comience con una [evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 
- Una cuenta de Media Services. Para más información, vea [Creación de una cuenta de Azure Media Services mediante Azure Portal](media-services-portal-create-account.md).

Al utilizar la autenticación de Azure AD con Azure Media Services, tiene dos opciones de autenticación:

- **Autenticación de entidad de servicio**. Autenticar un servicio. Las aplicaciones que normalmente utilizan este método de autenticación son aplicaciones que ejecutan servicios de demonio, servicios de nivel intermedio o trabajos programados: Web Apps, Function Apps, Logic Apps, API o un microservicio.
- **Autenticación de usuario**. Autenticar a alguien que usa la aplicación para interactuar con los recursos de Media Services. La aplicación interactiva en primer lugar debe solicitar al usuario las credenciales. Un ejemplo es una aplicación de consola de administración que usan los usuarios autorizados para supervisar trabajos de codificación o streaming en vivo. 

## <a name="access-the-media-services-api"></a>Acceso a la API de Media Services

Esta página le permite seleccionar el método de autenticación que desea usar para conectarse a la API. La página también proporciona los valores que necesita para conectarse a la API.

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Media Services.
2. Seleccione cómo conectarse a la API de Media Services.
3. En **Conectar con la API de Media Services**, seleccione la versión de Media Services API a la que desea conectarse.

## <a name="service-principal-authentication--recommended"></a>Autenticación de la entidad de servicio (recomendada)

Autentica un servicio usando una aplicación de Azure Active Directory (Azure AD) y un secreto. Esto se recomienda para cualquier servicio de nivel medio que llame a Media Services API. Algunos ejemplos son Web Apps, Functions, Logic Apps, las API y los microservicios. Este es el método de autenticación recomendado.

### <a name="manage-your-azure-ad-app-and-secret"></a>Administración del secreto y la aplicación de Azure AD

La sección **Administración del secreto y la aplicación de AAD** le permite seleccionar o crear una nueva aplicación de Azure AD y generar un secreto. Por motivos de seguridad, el secreto no se puede mostrar una vez cerrada la hoja. La aplicación usa el identificador y el secreto de la aplicación para la autenticación con el fin de obtener un token válido para Media Services.

Asegúrese de que tiene permisos suficientes para registrar una aplicación con el inquilino de Azure AD y de que la asigna a un rol en la suscripción de Azure. Para más información, consulte los [permisos necesarios](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

### <a name="connect-to-media-services-api"></a>Conexión con Media Services API

La opción **Conectar con la API de Media Services** proporciona los valores que se usan para conectar la aplicación de la entidad de servicio. Puede obtener valores de texto o copiar los bloques JSON o XML.

## <a name="user-authentication"></a>Autenticación de usuarios

Esta opción puede usarse para autenticar a un empleado o miembro de una instancia de Azure Active Directory que use una aplicación para interactuar con los recursos de Media Services. La aplicación interactiva, en primer lugar, debe solicitar al usuario las credenciales. Este método de autenticación solo debe usarse para las aplicaciones de administración.

### <a name="connect-to-media-services-api"></a>Conexión con Media Services API

Copie sus credenciales para conectar la aplicación de usuario desde la sección **Conectar con la API de Media Services**. Puede obtener valores de texto o copiar los bloques JSON o XML.

## <a name="next-steps"></a>Pasos siguientes

Empiece a trabajar en la [carga de archivos en la cuenta](media-services-portal-upload-files.md).
