---
title: Solución de problemas con la experiencia de usuario de Azure Data Factory
description: Obtenga información acerca de la solución de problemas relacionados con la experiencia de usuario de Azure Data Factory.
services: data-factory
author: ceespino
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 8/03/2020
ms.author: ceespino
ms.reviewer: daperlov
ms.openlocfilehash: 9a96c385c25eea1c5e217665f162de402a73e558
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87567841"
---
# <a name="troubleshoot-azure-data-factory-ux-issues"></a>Solución de problemas con la experiencia de usuario de Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran métodos comunes de solución de problemas con la experiencia de usuario de Azure Data Factory.

## <a name="adf-ux-not-loading"></a>No carga la experiencia de usuario de Azure Data Factory

> [!NOTE]
> La experiencia de usuario de Azure Data Factory admite oficialmente Microsoft Edge y Google Chrome. El uso de otros exploradores web puede dar lugar a un comportamiento inesperado o no documentado.

### <a name="third-party-cookies-blocked"></a>Cookies de terceros bloqueadas

La experiencia de usuario de ADF usa cookies del explorador para conservar la sesión del usuario y habilitar las experiencias interactivas de desarrollo y supervisión. Es posible que el explorador bloquee las cookies de terceros si usa una sesión de incógnito o tiene habilitado un bloqueador de anuncios. El bloqueo de las cookies de terceros puede provocar problemas al cargar el portal, como que se le redirija a una página en blanco, https://adf.azure.com/accesstoken.html, o que se muestre un mensaje de advertencia que indique que las cookies de terceros están bloqueadas. Para solucionar este problema, habilite las opciones de cookies de terceros en el explorador mediante estos pasos:

### <a name="google-chrome"></a>Google Chrome

#### <a name="allow-all-cookies"></a>Permitir todas las cookies

1. Vaya a **chrome://settings/cookies** en el explorador.
1. Seleccione la opción **Permitir todas las cookies**. ![Chrome-Permitir-todas-las-cookies](media/data-factory-ux-troubleshoot-guide/chrome-allow-all-cookies.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

#### <a name="only-allow-adf-ux-to-use-cookies"></a>Permitir que solo la experiencia de usuario de ADF utilice cookies
Si no desea permitir todas las cookies, puede permitir que solo las utilice la experiencia de usuario de ADF:
1. Vaya a **chrome://settings/cookies**.
1. Haga clic en **Añadir** en la opción **Sitios web que pueden usar cookies siempre**. ![Agregar la experiencia de usuario de ADF a los sitios permitidos](media/data-factory-ux-troubleshoot-guide/chrome-only-adf-cookies-1.png)
1. Agregue el sitio **adf.azure.com**, active la opción **Todas las cookies** y guarde. ![Permitir todas las cookies del sitio de la experiencia de usuario de ADF](media/data-factory-ux-troubleshoot-guide/chrome-only-adf-cookies-2.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.


### <a name="microsoft-edge"></a>Microsoft Edge

1. Vaya a **edge://settings/content/cookies** en el explorador.
1. Asegúrese de que **Permitir que los sitios guarden y lean datos de cookies** está habilitado y que la opción **Bloquear las cookies de terceros** está deshabilitada. ![Permitir todas las cookies en Edge](media/data-factory-ux-troubleshoot-guide/edge-allow-all-cookies.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

#### <a name="only-allow-adf-ux-to-use-cookies"></a>Permitir que solo la experiencia de usuario de ADF utilice cookies

Si no desea permitir todas las cookies, puede permitir que solo las utilice la experiencia de usuario de ADF:

1. Vaya a **edge://settings/content/cookies**.
1. En la sección **Permitir**, haga clic en **Agregar** y agregue el sitio **adf.azure.com**. ![Agregar la experiencia de usuario de ADF a sitios permitidos](media/data-factory-ux-troubleshoot-guide/edge-allow-adf-cookies.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, pruebe estos recursos:

* [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
* [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
* [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
* [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
* [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/)
* [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-data-factory.html)
