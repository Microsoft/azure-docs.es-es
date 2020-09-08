---
title: Paso de tokens de autenticación a Azure Media Services | Microsoft Docs
description: Obtenga información sobre cómo enviar tokens de autenticación desde el cliente al servicio de entrega de claves de Azure Media Services.
services: media-services
keywords: content protection, DRM, token authentication
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 7c3b35d9-1269-4c83-8c91-490ae65b0817
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 498e6d9e38468b126dbe10e0d0f59f54f1816ff0
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89261050"
---
# <a name="learn-how-clients-pass-tokens-to-the-azure-media-services-key-delivery-service"></a>Obtenga información sobre cómo pasan los clientes tokens al servicio de entrega de claves de Azure Media Services.

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Los clientes suelen preguntar cómo un reproductor puede pasar tokens al servicio de entrega de claves de Azure Media Services para la verificación, a fin de que el reproductor pueda obtener la clave. Media Services admite los formatos de token web simple (SWT) y de JSON Web Token (JWT). La autenticación de token puede aplicarse a cualquier tipo de clave, con independencia de que use el cifrado común o el cifrado de sobre Estándar de cifrado avanzado (AES) en el sistema.

 En función del reproductor y de la plataforma de destino, puede pasar el token con su reproductor de las siguientes formas:

- A través del encabezado de autorización HTTP.
    > [!NOTE]
    > Las especificaciones de OAuth 2.0 esperan el prefijo "Portador". Hay un reproductor de ejemplo con la configuración de token hospedado en la [página de demostración](https://ampdemo.azureedge.net/) de Azure Media Player. Para definir el origen de vídeo, elija **AES (token de JWT)** o **AES (token de SWT)** . El token se pasa a través del encabezado de autorización.

- Mediante la adición de un parámetro de consulta de URL con "token=tokenvalue".  
    > [!NOTE]
    > No se espera el prefijo "Portador". Puesto que el token se envía a través de una dirección URL, debe proteger la cadena de token. Este es un código de ejemplo de C# que muestra cómo hacerlo:

    ```csharp
    string armoredAuthToken = System.Web.HttpUtility.UrlEncode(authToken);
    string uriWithTokenParameter = string.Format("{0}&token={1}", keyDeliveryServiceUri.AbsoluteUri, armoredAuthToken);
    Uri keyDeliveryUrlWithTokenParameter = new Uri(uriWithTokenParameter);
    ```

- Con el campo CustomData.
Esta opción se usa para la adquisición de la licencia de PlayReady, mediante el campo CustomData del desafío de adquisición de licencias de PlayReady. En este caso, el token debe estar dentro del documento XML que se describe a continuación:

    ```xml
    <?xml version="1.0"?>
    <CustomData xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyCustomData/v1"> 
        <Token></Token> 
    </CustomData>
    ```
    Coloque el token de autenticación en el elemento Token.

- Mediante una lista de reproducción alternativa de HTTP Live Streaming (HLS). Si necesita configurar la autenticación de token para la reproducción AES + HLS en iOS/Safari, no hay una manera de enviar directamente el token. Para más información sobre cómo alternar la lista de reproducción para habilitar este escenario, vea esta [entrada de blog](https://azure.microsoft.com/blog/2015/03/06/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
