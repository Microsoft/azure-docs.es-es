---
title: Compatibilidad de API en Azure Static Web Apps con Azure Functions
description: Obtenga información sobre las API que admiten Azure Static Web Apps
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 09daffa74ccd279c8187391ba3b86063aed7d204
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88607032"
---
# <a name="api-support-in-azure-static-web-apps-preview-with-azure-functions"></a>Compatibilidad de API en Azure Static Web Apps (versión preliminar) con Azure Functions

Azure Static Web Apps proporciona puntos de conexión de API sin servidor a través de [Azure Functions](../azure-functions/functions-overview.md). Al aprovechar Azure Functions, las API se escalan dinámicamente según la demanda e incluyen las características siguientes:

- **Seguridad integrada** con acceso directo a los datos de [autenticación y autorización basada en roles](user-information.md) del usuario.
- **Enrutamiento de conexión directa** que hace que la ruta de _api_ esté disponible para la aplicación web de forma segura sin necesidad de reglas de CORS personalizadas.
- **Azure Functions** v3 compatible con Node.js 12.
- **Desencadenadores HTTP** y enlaces de salida.

## <a name="configuration"></a>Configuración

Los puntos de conexión de API están a disposición de la aplicación web a través de la ruta de _api_. Aunque esta ruta es fija, usted tiene control sobre la carpeta donde se encuentra la aplicación de Azure Functions asociada. Para cambiar esta ubicación, [edite el archivo YAML del flujo de trabajo](github-actions-workflow.md#build-and-deploy), que se encuentra en la carpeta _.github/workflows_ de su repositorio.

## <a name="constraints"></a>Restricciones

Azure Static Web Apps proporciona una API a través de Azure Functions. Las funcionalidades de Azure Functions se centran en un conjunto específico de características que le permiten crear una API para una aplicación web, y permitir que la aplicación web se conecte a la API de forma segura. Estas características presentan algunas restricciones, entre otras:

- El prefijo de la ruta de la API debe ser _api_.
- La aplicación de funciones de la API debe estar en JavaScript.
- Las reglas de enrutamiento para las funciones de API solo admiten [redirigir](routes.md#redirects) y [proteger las rutas con roles](routes.md#securing-routes-with-roles).
- Los desencadenadores se limitan a [HTTP](../azure-functions/functions-bindings-http-webhook.md).
  - Se admiten los [enlaces](../azure-functions/functions-triggers-bindings.md#supported-bindings) de entrada y salida.
- Los registros solo están disponibles si agrega [Application Insights](../azure-functions/functions-monitoring.md) a la aplicación de Functions.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adición de una API](add-api.md)
