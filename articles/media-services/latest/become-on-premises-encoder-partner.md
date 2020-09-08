---
title: 'Conviértase en un asociado de codificador local: Azure Media Services'
description: En este artículo se explica cómo comprobar sus codificadores de streaming en vivo locales.
services: media-services
author: johndeu
manager: johndeu
ms.author: johndeu
ms.date: 08/31/2020
ms.topic: how-to
ms.service: media-services
ms.openlocfilehash: 1383b6afac9ad2332169328ef71981e23e86bdcc
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297457"
---
# <a name="how-to-verify-your-on-premises-live-streaming-encoder"></a>Cómo comprobar su codificador de streaming en vivo local

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Como asociado de codificador local de Azure Media Services, Media Services promueve su producto al recomendar su codificador a clientes empresariales. Para convertirse en un asociado de codificador local, debe comprobar la compatibilidad del codificador local con Media Services. Para ello, complete las comprobaciones siguientes:

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="pass-through-live-event-verification"></a>Verificación del Evento en directo con paso a través

1. En la cuenta de Media Services, asegúrese de que el **Punto de conexión de streaming** esté en ejecución. 
2. Cree e inicie el Evento en directo de **paso a través**. <br/> Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).
3. Obtenga las direcciones URL de ingesta y configure el codificador local de forma que utilice la dirección URL para enviar una secuencia en directo de velocidad de bits múltiple a Media Services.
4. Obtenga la dirección URL de versión preliminar y úsela para verificar que la entrada del codificador se está recibiendo realmente.
5. Cree un nuevo objeto de **recurso**.
6. Cree un objeto de **salida en directo** y use el nombre del recurso que ha creado.
7. Cree un objeto **Streaming Locator** con los tipos del objeto **Streaming Policy** integrados.
8. Enumere las rutas de acceso en el **localizador de streaming** para recuperar las direcciones URL que se van a usar.
9. Obtenga el nombre de host del **punto de conexión de streaming** desde el que quiere hacer el streaming.
10. Combine la dirección URL del paso 8 con el nombre de host del paso 9 para obtener la dirección URL completa.
11. Ejecute el codificador en directo durante aproximadamente diez minutos.
12. Detenga el objeto LiveEvent. 
13. Use un reproductor, como [Azure Media Player](https://aka.ms/azuremediaplayer), para ver el recurso archivado y asegurarse de que la reproducción no tenga problemas visibles. También, puede ver el recurso y validarlo utilizando la dirección URL de vista previa durante la sesión activa.
14. Registre el identificador del recurso, la dirección URL de streaming publicada en el archivo dinámico y la configuración y la versión que utilizó en el codificador en directo.
15. Restablezca el estado del Evento en directo después de crear cada ejemplo.
16. Repita los pasos 5 a 15 con todas las configuraciones admitidas por el codificador (con y sin señalización, subtítulos o velocidades de codificación diferentes).

## <a name="live-encoding-live-event-verification"></a>Verificación del Evento en directo con codificación en directo

1. En la cuenta de Media Services, asegúrese de que el **Punto de conexión de streaming** esté en ejecución. 
2. Cree e inicie el Evento en directo de **codificación en directo**. <br/> Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).
3. Obtenga las direcciones URL de ingesta y configure el codificador para insertar una secuencia en directo de velocidad de bits única en Media Services.
4. Obtenga la dirección URL de versión preliminar y úsela para verificar que la entrada del codificador se está recibiendo realmente.
5. Cree un nuevo objeto de **recurso**.
6. Cree un objeto de **salida en directo** y use el nombre del recurso que ha creado.
7. Cree un objeto **Streaming Locator** con los tipos del objeto **Streaming Policy** integrados.
8. Enumere las rutas de acceso en el **localizador de streaming** para recuperar las direcciones URL que se van a usar.
9. Obtenga el nombre de host del **punto de conexión de streaming** desde el que quiere hacer el streaming.
10. Combine la dirección URL del paso 8 con el nombre de host del paso 9 para obtener la dirección URL completa.
11. Ejecute el codificador en directo durante aproximadamente diez minutos.
12. Detenga el objeto LiveEvent.
13. Use un reproductor, como [Azure Media Player](https://aka.ms/azuremediaplayer), para ver el recurso archivado y asegurarse de que la reproducción no tenga problemas visibles en todos los niveles de calidad. También, puede ver el recurso y validarlo utilizando la dirección URL de vista previa durante la sesión activa.
14. Registre el identificador del recurso, la dirección URL de streaming publicada en el archivo dinámico y la configuración y la versión que utilizó en el codificador en directo.
15. Restablezca el estado del Evento en directo después de crear cada ejemplo.
16. Repita los pasos 5 a 15 con todas las configuraciones admitidas por el codificador (con y sin señalización, subtítulos o velocidades de codificación diferentes).

## <a name="longevity-verification"></a>Comprobación de duración

Siga los mismos pasos que se indican en [Verificación del evento en directo con paso a través](#pass-through-live-event-verification), salvo el paso 11. <br/>En lugar de 10 minutos, ejecute el codificador en directo durante una semana o más. Use un reproductor, como [Azure Media Player](https://aka.ms/azuremediaplayer), para ver el streaming en vivo cada cierto tiempo (o un recurso archivado) y asegurarse de que la reproducción no tenga problemas visibles.

## <a name="email-your-recorded-settings"></a>Envío de la configuración anotada por correo electrónico

Por último, envíe la configuración registrada y los parámetros archivados por correo electrónico a Azure Media Services (amshelp@microsoft.com) para notificar que se han superado todos los pasos de la comprobación automática. Incluya también sus datos de contacto para que pueda recibir las actualizaciones que haya. Puede ponerse en contacto con el equipo de Azure Media Services si tiene alguna pregunta sobre este proceso.

## <a name="see-also"></a>Consulte también

[Codificadores locales probados](recommended-on-premises-live-encoders.md)

## <a name="next-steps"></a>Pasos siguientes

[Streaming en vivo con Media Services v3](live-streaming-overview.md)
