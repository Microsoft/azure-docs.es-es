---
title: ¿Qué es Azure Static Web Apps?
description: Características clave y funcionalidad de Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 086feb3aa7c9a38756fded58538eb22e094a5960
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906805"
---
# <a name="what-is-azure-static-web-apps-preview"></a>¿Qué es la versión preliminar de Azure Static Web Apps?

Azure Static Web Apps es un servicio que compila e implementa automáticamente aplicaciones web de pila completa en Azure desde un repositorio de GitHub.

:::image type="content" source="media/overview/static-apps-overview.png" alt-text="Información general de Static Web Apps":::

El flujo de trabajo de Azure Static Web Apps se adapta al flujo de trabajo diario de un desarrollador. Las aplicaciones se compilan e implementan en función de las interacciones de GitHub.

Cuando se crea un recurso de Azure Static Web Apps, Azure configura un flujo de trabajo de Acciones de GitHub en el repositorio de código fuente de la aplicación que supervisa una rama de su elección. Cada vez que inserte confirmaciones o acepte solicitudes de incorporación de cambios en la rama inspeccionada, la acción de GitHub compilará e implementará automáticamente la aplicación y su API en Azure.

Las aplicaciones web estáticas se suelen compilar con bibliotecas y marcos como Angular, React, Svelte, Vue o Blazor. Estas aplicaciones incluyen recursos HTML, CSS, JavaScript y de imagen que componen la aplicación. Con un servidor web tradicional, estos recursos se ofrecen desde un único servidor junto con los puntos de conexión de la API necesarios.

Con Static Web Apps, los recursos estáticos se separan de un servidor web tradicional y, en su lugar, se ofrecen puntos distribuidos geográficamente en todo el mundo. Esta distribución permite ofrecer archivos de forma más rápida, dado que se encuentran físicamente más cerca de los usuarios finales. Además, los puntos de conexión de la API se hospedan mediante una [arquitectura sin servidor](../azure-functions/functions-overview.md), lo que evita la necesidad de usar un servidor back-end completo.

## <a name="key-features"></a>Principales características

- **Hospedaje web** para contenido estático como HTML, CSS, JavaScript e imágenes.
- Compatibilidad con la **API integrada** proporcionada por Azure Functions.
- **Integración de GitHub de primera clase** donde los cambios de repositorio desencadenan compilaciones e implementaciones.
- Contenido estático **distribuido globalmente**, lo que permite que el contenido esté más cerca de los usuarios.
- **Certificados SSL gratuitos**, que se renuevan automáticamente.
- **Dominios personalizados** para proporcionar personalizaciones de marca a la aplicación.
- **Modelo de seguridad ágil** con un proxy inverso al llamar a las API, lo que no requiere ninguna configuración de CORS.
- **Integraciones del proveedor de autenticación** con Azure Active Directory, Facebook, Google, GitHub y Twitter.
- **Definición de roles de autorización personalizables** y asignaciones.
- **Reglas de enrutamiento de back-end** que permiten tener control total sobre el contenido y las rutas que atiende.
- **Versiones de almacenamiento provisional generadas** que se basan en las solicitudes de incorporación de cambios y permiten obtener versiones preliminares del sitio antes de su publicación.

## <a name="what-you-can-do-with-static-web-apps"></a>Qué puede hacer con Static Web Apps

- **Crear aplicaciones web modernas** con los marcos y bibliotecas de JavaScript, como [Angular](getting-started.md?tabs=angular), [React](getting-started.md?tabs=react), [Svelte](https://docs.microsoft.com/learn/modules/publish-app-service-static-web-app-api/), [Vue](getting-started.md?tabs=react) o con[Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) para crear aplicaciones WebAssembly con un servidor back-end de [Azure Functions](https://azure.microsoft.com/es-mx/services/functions/).
- **Publicar sitios estáticos** con marcos como [Gatsby](publish-gatsby.md), [Hugo](publish-hugo.md) y [VuePress](publish-vuepress.md).
- **Implementar aplicaciones web** con marcos como [Next.js](deploy-nextjs.md) y [Nuxt.js](deploy-nuxtjs.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de la primera aplicación estática](getting-started.md)
