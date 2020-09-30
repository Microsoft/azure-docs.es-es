---
title: Solución de problemas con la experiencia de usuario de Azure Data Factory
description: Obtenga información acerca de la solución de problemas relacionados con la experiencia de usuario de Azure Data Factory.
services: data-factory
author: ceespino
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 09/02/2020
ms.author: ceespino
ms.reviewer: daperlov
ms.openlocfilehash: e0c5ab4f6d36a24ff9f59ec373291e00669e1e92
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89320126"
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
1. Seleccione la opción **Permitir todas las cookies**. 

    ![Permitir todas las cookies en Chrome](media/data-factory-ux-troubleshoot-guide/chrome-allow-all-cookies.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

#### <a name="only-allow-adf-ux-to-use-cookies"></a>Permitir que solo la experiencia de usuario de ADF utilice cookies
Si no desea permitir todas las cookies, puede permitir que solo las utilice la experiencia de usuario de ADF:
1. Vaya a **chrome://settings/cookies**.
1. Seleccione **Añadir** en la opción **Sitios web que pueden usar cookies siempre**. 

    ![Agregar la experiencia de usuario de ADF a sitios permitidos en Chrome](media/data-factory-ux-troubleshoot-guide/chrome-only-adf-cookies-1.png)
1. Agregue el sitio **adf.azure.com**, active la opción **Todas las cookies** y guarde. 

    ![Permitir todas las cookies del sitio de la experiencia de usuario de ADF](media/data-factory-ux-troubleshoot-guide/chrome-only-adf-cookies-2.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

### <a name="microsoft-edge"></a>Microsoft Edge

1. Vaya a **edge://settings/content/cookies** en el explorador.
1. Asegúrese de que **Permitir que los sitios guarden y lean datos de cookies** está habilitado y que la opción **Bloquear las cookies de terceros** está deshabilitada. 

    ![Permitir todas las cookies en Edge](media/data-factory-ux-troubleshoot-guide/edge-allow-all-cookies.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

#### <a name="only-allow-adf-ux-to-use-cookies"></a>Permitir que solo la experiencia de usuario de ADF utilice cookies

Si no desea permitir todas las cookies, puede permitir que solo las utilice la experiencia de usuario de ADF:

1. Vaya a **edge://settings/content/cookies**.
1. En la sección **Permitir**, seleccione **Agregar** y agregue el sitio **adf.azure.com**. 

    ![Agregar la experiencia de usuario de ADF a sitios permitidos en Edge](media/data-factory-ux-troubleshoot-guide/edge-allow-adf-cookies.png)
1. Actualice la experiencia de usuario de ADF e inténtelo de nuevo.

## <a name="connection-failed-on-adf-ux"></a>Error en la conexión de la experiencia de usuario de ADF

A veces aparecerán "errores de conexión" en la experiencia de usuario de ADF parecidos a los de la siguiente captura de pantalla después de hacer clic en **Prueba de conexión**, **Vista previa**, etc.

![Error en la conexión](media/data-factory-ux-troubleshoot-guide/connection-failed.png)

En este caso, puede intentar primero la misma operación con el modo de exploración InPrivate del explorador.

Si sigue sin funcionar, en el explorador, presione F12 para abrir **Herramientas de desarrollo**. Vaya a la pestaña **Red**, active **Disable Cache** (Deshabilitar caché), vuelva a intentar la operación con el error y busque la solicitud con error (en rojo).

![Solicitud con error](media/data-factory-ux-troubleshoot-guide/failed-request.png)

A continuación, busque el **nombre de host** (en este caso, **dpnortheurope.svc.datafactory.azure.com**) de la **dirección URL de la solicitud** de la solicitud con error.

Escriba directamente el **nombre de host** en la barra de direcciones del explorador. Si ve 404 en el explorador, esto normalmente significa que el lado cliente es correcto y que el problema está en el lado del servicio ADF. Abra una incidencia de soporte técnico con el **identificador de actividad** del mensaje de error de la experiencia del usuario de ADF.

![Recurso no encontrado](media/data-factory-ux-troubleshoot-guide/status-code-404.png)

Si no es así, o si ve un error similar a continuación en el explorador, esto suele significar que hay algún problema del lado cliente. Siga los pasos de solución de problemas.

![Error del lado cliente](media/data-factory-ux-troubleshoot-guide/client-side-error.png)

Abra el **símbolo del sistema** y escriba **nslookup dpnortheurope.svc.datafactory.azure.com**. Una respuesta normal debería tener este aspecto:

![Respuesta del comando 1](media/data-factory-ux-troubleshoot-guide/command-response-1.png)

Si ve una respuesta DNS normal, póngase en contacto con el equipo de soporte técnico de TI local para comprobar la configuración del firewall para ver si la conexión HTTPS a este nombre de host está bloqueada o no. Si el problema no se pudo resolver, abra una incidencia de soporte técnico con el **identificador de actividad** del mensaje de error de la experiencia del usuario de ADF.

Si esto es todo lo que puede ver, esto normalmente significa que hay algún problema con el servidor DNS al resolver el nombre DNS. Normalmente, el cambio de ISP (proveedor de servicios de Internet) o del DNS (por ejemplo, a Google DNS 8.8.8.8) podría ser una posible solución. Si el problema persiste, podría probar con **nslookup datafactory.azure.com** y **nslookup azure.com** para ver en qué nivel se produjo el error en la resolución de DNS y enviar toda la información al equipo de soporte técnico de TI local o a su ISP para solucionar el problema. Si considera que el problema está todavía en Microsoft, abra una incidencia de soporte técnico con el **identificador de actividad** del mensaje de error de la experiencia de usuario de ADF.

![Respuesta del comando 2](media/data-factory-ux-troubleshoot-guide/command-response-2.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, pruebe estos recursos:

* [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
* [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
* [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
* [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
* [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/)
* [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-data-factory.html)
