---
title: Solución de problemas de la herramienta Copiar datos en Azure Data Factory
description: Obtenga información acerca de la solución de problemas relacionados con la herramienta Copiar datos en Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: troubleshooting
ms.date: 07/28/2020
ms.author: yexu
ms.openlocfilehash: 428b8956b4a86009c946d9d8e9e9ba1c58d31403
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87388328"
---
# <a name="troubleshoot-copy-data-tool-in-azure-data-factory"></a>Solución de problemas de la herramienta Copiar datos en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran métodos comunes de solución de problemas de la herramienta Copiar datos en Azure Data Factory.

## <a name="common-errors-and-messages"></a>Errores habituales y mensajes

### <a name="error-code-unable-to-validate-in-copy-data-tool"></a>Código de error: No se puede validar en la herramienta Copiar datos

- **Síntomas**: En el primer paso de la herramienta Copiar datos, aparece el mensaje de advertencia "No se puede validar".
- **Causas**: Esto puede ocurrir si todas las cookies de terceros están deshabilitadas.
- **Solución:** 
    - Use Internet Explorer o Microsoft Edge.
    - Si usa el explorador Chrome, siga las instrucciones que se indican a continuación para agregar una excepción de cookies para *microsoftonline.com* y *windows.net*.
        1.  Abra el explorador Chrome.
        2.  Haga clic en la llave inglesa o en las tres líneas a la derecha (Personalizar y controlar Google Chrome).
        3.  Haga clic en **Configuración**.
        4.  Busque **Cookies** o vaya a **Privacidad** en Configuración avanzada.
        5.  Seleccione **Configuración de contenido**.    
        6.  Las cookies deben estar establecidas en **Permitir que se almacenen datos locales (recomendado)** .
        7.  Haga clic en **Administrar excepciones**. En **Patrón de nombre de host** escriba lo siguiente y asegúrese de que **Permitir** sea el comportamiento establecido.
            - login.microsoftonline.com
            - login.windows.net
        8.  Cierre el explorador y vuelva a abrirlo.
    - Si usa el explorador Firefox, siga las instrucciones que se indican a continuación para agregar excepciones de cookies.
        1. En el menú de Firefox, vaya a **Herramientas** > **Opciones**.
        2. En **Privacidad** > **Historial**, es posible que el valor actual sea **Usar una configuración personalizada para el historial**.
        3. En **Aceptar cookies de terceros**, es posible que el valor actual sea **Nunca**. En tal caso, debe hacer clic en **Excepciones** a la derecha para agregar los sitios siguientes.
            - https://login.microsoftonline.com
            - https://login.windows.net
        4.  Cierre el explorador y vuelva a abrirlo. 


### <a name="error-code-unable-to-open-login-page-and-enter-password"></a>Código de error: No se puede abrir la página de inicio de sesión e introducir la contraseña

- **Síntomas**: La herramienta Copiar datos le redirige a la página de inicio de sesión, pero la página no se muestra correctamente.
- **Causas**: Este problema puede producirse si ha cambiado el entorno de red de una red de oficina a una red doméstica. Hay algunas cachés en exploradores. 
- **Solución:** 
    1.  Cierre el explorador e inténtelo de nuevo. Si el problema persiste, vaya al paso siguiente.   
    2.  Si usa Internet Explorer, intente abrirla en modo privado (presione "Ctrl" + "Mayús" + "P"). Si usa Chrome, intente abrirla en modo de incógnito (presione "Ctrl" + "Mayús" + "N"). Si el problema persiste, vaya al paso siguiente. 
    3.  Pruebe con otro explorador. 


## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, pruebe estos recursos:
*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-data-factory.html)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
*  [Guía de optimización y rendimiento de Mapping Data Flows](concepts-data-flow-performance.md)
