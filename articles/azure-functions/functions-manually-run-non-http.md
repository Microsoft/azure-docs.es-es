---
title: Ejecución manual de una instancia de Azure Functions no desencadenada por HTTP
description: Uso de una solicitud HTTP para ejecutar una instancia de Azure Functions no desencadenada por HTTP
author: craigshoemaker
ms.topic: article
ms.date: 04/23/2020
ms.author: cshoe
ms.openlocfilehash: 37f79d717b7ea0e26717e7b51f9e66b908b96521
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640977"
---
# <a name="manually-run-a-non-http-triggered-function"></a>Ejecución manual de una función no desencadenada por HTTP

En este artículo se muestra cómo ejecutar manualmente una función no desencadenada por HTTP a través de una solicitud HTTP con formato especial.

En algunos contexto, es posible que tenga que ejecutar "a petición" una instancia de Azure Functions desencadenada de manera indirecta.  Algunos ejemplos de desencadenadores indirectos son las [funciones según una programación](./functions-create-scheduled-function.md) o las funciones que se ejecutan como resultado de la [acción de otro recurso](./functions-create-storage-blob-triggered-function.md). 

En el ejemplo siguiente se usa [Postman](https://www.getpostman.com/), pero puede usar [cURL](https://curl.haxx.se/), [Fiddler](https://www.telerik.com/fiddler) o cualquier otra herramienta similar para enviar solicitudes HTTP.

## <a name="define-the-request-location"></a>Definición de la ubicación de la solicitud

Para ejecutar una función no desencadenada por HTTP, necesita una manera de enviar una solicitud a Azure para que ejecute la función. La dirección URL que se usa para hacer esta solicitud tiene un formato específico.

![Definición de la ubicación de la solicitud: nombre de host + ruta de acceso a la carpeta + nombre de la función](./media/functions-manually-run-non-http/azure-functions-admin-url-anatomy.png)

- **Nombre de host:** La ubicación pública de la aplicación de función que consta del nombre de la aplicación de función más *azurewebsites.net* o el dominio personalizado.
- **Ruta de acceso a la carpeta:** Para acceso a las funciones no desencadenadas por HTTP a través de una solicitud HTTP, debe enviar la solicitud a través de las carpetas *admin/functions*.
- **Nombre de la función:** El nombre de la función que quiere ejecutar.

Puede usar la ubicación de esta solicitud en Postman junto con la clave maestra de la función en la solicitud para que Azure ejecute la función.

> [!NOTE]
> Cuando se ejecuta localmente, la clave maestra de la función no es necesaria. Puede [llamar a la función](#call-the-function) directamente al omitir el encabezado `x-functions-key`.

## <a name="get-the-functions-master-key"></a>Obtención de la clave maestra de la función

1. Vaya a la aplicación de función en [Azure Portal](https://portal.azure.com), seleccione **Claves de la aplicación** y, luego, la clave `_master`. 

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key.png" alt-text="Busque la clave maestra que se va a copiar." border="true":::

1. En la sección **Editar clave**, copie el valor de la clave en el portapapeles y, a continuación, seleccione **Aceptar**.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key-copy.png" alt-text="Copie la clave maestra en el portapapeles." border="true":::

1. Después de copiar la clave *_master*, seleccione **Código y prueba** y, a continuación, seleccione **Registros**. Aquí verá mensajes de la función que se registraron cuando ejecutó manualmente la función desde Postman.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-function-log.png" alt-text="Vea los registros para ver los resultados de la prueba de clave maestra." border="true":::

> [!CAUTION]  
> Debido a los permisos elevados de la aplicación de función otorgados por la clave maestra, no debe compartir esta clave con terceros ni distribuirla en una aplicación. La clave solo se debe enviar a un punto de conexión HTTPS.

## <a name="call-the-function"></a>Llamada a la función

Abra Postman y siga estos pasos:

1. Escriba la **ubicación de la solicitud en el cuadro de texto de la dirección URL**.
1. Asegúrese de que el método HTTP está establecido en **POST**.
1. Seleccione la pestaña **Encabezados**.
1. Escriba **x-functions-key** como la primera clave y pegue la clave maestra (que está en el portapapeles) como el valor.
1. Escriba **Content-Type** como la segunda clave y **application/json** como el valor.

    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-headers.png" alt-text="Configuración de los encabezados de Postman." border="true":::

1. Seleccione la pestaña **Cuerpo**.
1. Escriba **{ "input": "test" }** como el cuerpo de la solicitud.

    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-body.png" alt-text="Configuración del cuerpo de Postman." border="true":::

1. Seleccione **Enviar**.
        
    :::image type="content" source="./media/functions-manually-run-non-http/functions-manually-run-non-http-send.png" alt-text="Envíe una solicitud con Postman." border="true":::

    Postman luego notifica un estado de **202 - Aceptado**.

1. A continuación, vuelva a la función en Azure Portal. Revise los registros y verá mensajes procedentes de la llamada manual a la función.

    :::image type="content" source="./media/functions-manually-run-non-http/azure-portal-functions-master-key-logs.png" alt-text="Vea los registros para ver los resultados de la prueba de clave maestra." border="true":::

## <a name="next-steps"></a>Pasos siguientes

- [Estrategias para probar el código en Azure Functions](./functions-test-a-function.md)
- [Depuración local de funciones de Azure desencadenadas por Event Grid](./functions-debug-event-grid-trigger-local.md)
