---
title: Solución de problemas de configuración de Azure Front Door
description: En este tutorial, aprenderá a solucionar por sí mismo algunos de los problemas comunes que pueden aparecer en Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 09/22/2018
ms.author: duau
ms.openlocfilehash: babe24d0c934cffac00a5100d1da7ee252d147da
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399062"
---
# <a name="troubleshooting-common-routing-issues"></a>Solución de problemas comunes de enrutamiento

En este artículo se describe cómo solucionar algunos de los problemas comunes de enrutamiento que pueden surgir con la configuración de Azure Front Door.

## <a name="503-response-from-front-door-after-a-few-seconds"></a>Respuesta 503 de Front Door después de unos segundos

### <a name="symptom"></a>Síntoma

- Las solicitudes normales enviadas al back-end sin pasar por Front Door se realizan correctamente, pero el paso por Front Door produce respuestas de error 503.

- El error de Front Door se muestra después de unos segundos (normalmente, al cabo de unos 30 segundos).

### <a name="cause"></a>Causa

Este síntoma se produce cuando el back-end tarda más del tiempo de espera configurado (el valor predeterminado es de 30 segundos) en recibir la solicitud de Front Door o si tarda más de este valor de tiempo de espera en enviar una respuesta a la solicitud de Front Door. 

### <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

- Envíe la solicitud al back-end directamente (sin pasar por Front Door) y vea cuál es el tiempo habitual que tarda el back-end en responder.
- Envíe la solicitud a través de Front Door y vea si se muestran respuestas 503. Si no es así, es posible que no se trate de un problema de tiempo de espera. Póngase en contacto con el soporte técnico.
- Si el paso por Front Door tiene como resultado un código de respuesta de error 503, configure el campo sendReceiveTimeout para que Front Door extienda el tiempo de espera predeterminado a un máximo de 4 minutos (240 segundos). La opción de configuración está en `backendPoolSettings` y se denomina `sendRecvTimeoutSeconds`. 

## <a name="requests-sent-to-the-custom-domain-returns-400-status-code"></a>Las solicitudes enviadas al dominio personalizado devuelven el código de estado 400

### <a name="symptom"></a>Síntoma

- Ha creado una instancia de Front Door, pero una solicitud al dominio o el host de front-end devuelve un código de estado HTTP 400.

- Ha creado una asignación de DNS a partir de un dominio personalizado para el host de front-end que configuró. Sin embargo, al enviar una solicitud al nombre de host de dominio personalizado, se devuelve un código de estado HTTP 400 y no parece redirigirse a los back-ends que ha configurado.

### <a name="cause"></a>Causa

Este síntoma puede ocurrir si no ha configurado una regla de enrutamiento para el dominio personalizado que agregó como host de front-end. Debe agregarse de manera explícita una regla de enrutamiento para ese host de front-end, incluso si ya se ha configurado una para el host de front-end en el subdominio de Front Door (*.azurefd.net) al que el dominio personalizado tiene una asignación de DNS.

### <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

Agregue una regla de enrutamiento desde el dominio personalizado para el grupo de back-end deseado.

## <a name="front-door-is-not-redirecting-http-to-https"></a>Front Door no redirige HTTP a HTTPS

### <a name="symptom"></a>Síntoma

Su instancia de Front Door tiene una regla de enrutamiento que indica redirigir HTTP a HTTPS, pero el acceso al dominio sigue manteniendo HTTP como el protocolo.

### <a name="cause"></a>Causa

Este comportamiento puede producirse si no se han configurado correctamente las reglas de enrutamiento para Front Door. Básicamente, la configuración actual no se ha especificado y puede tener reglas en conflicto.

### <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

## <a name="request-to-frontend-hostname-returns-404-status-code"></a>La solicitud al nombre de host de front-end devuelve el código de estado 404

### <a name="symptom"></a>Síntoma

- Ha creado una instancia de Front Door y configurado un host de front-end, un grupo de back-end con al menos un back-end y una regla de enrutamiento que conecta el host de front-end con el grupo de back-end. El contenido parece no estar disponible al enviar una solicitud al host de front-end configurado, ya que se devuelve un código de estado HTTP 404.

### <a name="cause"></a>Causa

Hay varias causas posibles para este síntoma:

- El back-end no es un back-end orientado al público y no es visible para Front Door.
- El back-end está mal configurado, lo que hace que Front Door envíe la solicitud incorrecta (es decir, el back-end solo acepta HTTP, pero no ha desmarcado la opción de permitir HTTPS, por lo que Front Door está intentando reenviar solicitudes HTTPS).
- El back-end rechaza el encabezado de host que se ha reenviado con la solicitud al back-end.
- La configuración para el back-end aún no se ha implementado por completo.

### <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

1. Tiempo de implementación
   - Asegúrese de haber esperado unos 10 minutos para que se implemente la configuración.

2. Compruebe la configuración de back-end
    - Navegue hasta el grupo de back-ends al que debería enrutarse la solicitud (depende de cómo haya configurado la regla de enrutamiento) y compruebe que el _tipo de host de back-end_ y el nombre de host de back-end sean correctos. Si el back-end es un host personalizado, asegúrese de haberlo escrito correctamente. 

    - Compruebe sus puertos HTTP y HTTPS. En la mayoría de los casos, 80 y 443 (respectivamente) son correctos y no será necesario realizar ningún cambio. Sin embargo, existe la posibilidad de que el back-end no esté configurado de este modo y esté escuchando en otro puerto.

        - Compruebe el _Encabezado host de back-end_ configurado para los back-ends a los que debería enrutarse el host de front-end. En la mayoría de los casos, este encabezado debería ser el mismo que el _Nombre de host de back-end_. Sin embargo, un valor incorrecto puede provocar distintos códigos de estado HTTP 4xx si el back-end espera algo diferente. Si escribe la dirección IP de su back-end, es posible que deba establecer el _Encabezado host de back-end_ en el nombre de host del backend.


3. Compruebe la configuración de la regla de enrutamiento
    - Vaya a la regla de enrutamiento que debe enrutar del nombre de host de front-end en cuestión a un grupo de back-end. Asegúrese de que los protocolos aceptados estén configurados correctamente o, si no, asegúrese de que el protocolo que usará Front Door al reenviar la solicitud esté configurado correctamente. El campo de _protocolos aceptados_ determina qué solicitudes debe aceptar Front Door, y el _Protocolo de desvío_ determina qué protocolo debe usar Front Door para reenviar la solicitud al back-end.
         - Por ejemplo, si el back-end solo acepta solicitudes HTTP, las siguientes configuraciones serían válidas:
            - Los _protocolos aceptados_ son HTTP y HTTPS. El _protocolo de reenvío_ es HTTP. Confrontar solicitud no funcionará, ya que HTTPS es un protocolo permitido y, si llegó una solicitud como HTTPS, Front Door intentaría reenviarla mediante HTTPS.

            - Los _protocolos aceptados_ son HTTP. El _protocolo de reenvío_ es Confrontar solicitud o HTTPS.

    - La opción _Reescritura de dirección URL_ está deshabilitada de manera predeterminada, y solo debería usar este campo si quiere restringir el ámbito de los recursos hospedados en back-end que quiera que estén disponibles. Cuando esté deshabilitado, Front Door reenviará la misma ruta de acceso de solicitud que reciba. Es posible que este campo esté mal configurado y Front Door esté solicitando un recurso del back-end que no está disponible; por tanto, devuelve un código de estado HTTP 404.

