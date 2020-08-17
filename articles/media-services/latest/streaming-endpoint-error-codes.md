---
title: Errores de empaquetado y origen de Azure Media Services | Microsoft Docs
description: En este tema se describen los errores que podría recibir desde el servicio de punto de conexión de streaming de Azure Media Services (origen).
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2019
ms.author: juliako
ms.openlocfilehash: 1fc201cc1f3d4d26ca7b9e949d2917688e6fea8c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091987"
---
# <a name="streaming-endpoint-origin-errors"></a>Errores de punto de conexión de streaming (origen) 

En este tema se describen los errores que podría recibir desde el [servicio de punto de conexión de streaming](streaming-endpoint-concept.md) de Azure Media Services.

## <a name="400-bad-request"></a>400 - Solicitud incorrecta

La solicitud contiene información no válida y se rechazará con estos códigos de error debido a uno de los siguientes motivos:

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_BAD_URL_SYNTAX |0x80890201|Un error de formato o sintaxis de dirección URL. Algunos ejemplos incluyen solicitudes de un tipo no válido, un fragmento no válido o una pista no válida. |
|MPE_ENC_ENCRYPTION_NOT_SPECIFIED_IN_URL |0x8088024C|La solicitud no tiene ninguna etiqueta de cifrado en la dirección URL. Las solicitudes CMAF requieren una etiqueta de cifrado en la dirección URL. Otros protocolos configurados con más de un tipo de cifrado también requieren la etiqueta de cifrado para anular las ambigüedades. |
|MPE_STORAGE_BAD_URL_SYNTAX |0x808900E9|La solicitud al almacenamiento para que realice la solicitud devolvió un error de solicitud incorrecta. |

## <a name="403-forbidden"></a>403 Prohibido

No se aceptó la solicitud debido a una de las siguientes razones:

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_STORAGE_AUTHENTICATION_FAILED |0x808900EA|La solicitud al almacenamiento para que realice la solicitud devolvió un error de autenticación. Esto puede ocurrir si se giraron las claves de almacenamiento y el servicio no pudo sincronizar las claves de almacenamiento. <br/><br/>Póngase en contacto con el equipo de soporte técnico de Azure, acudiendo a [Ayuda y soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.|
|MPE_STORAGE_INSUFFICIENT_ACCOUNT_PERMISSIONS |0x808900EB |Error en la operación de almacenamiento, no se pudo acceder debido a permisos de cuenta insuficientes. |
|MPE_STORAGE_ACCOUNT_IS_DISABLED |0x808900EC |Error al solicitar al almacenamiento que realice la solicitud porque la cuenta de almacenamiento está deshabilitada. |
|MPE_STORAGE_AUTHENTICATION_FAILURE |0x808900F3 |Error de operación de almacenamiento, no se pudo acceder debido a errores genéricos. |
|MPE_OUTPUT_FORMAT_BLOCKED |0x80890207 |El formato de salida está bloqueado debido a la configuración en el objeto StreamingPolicy. |
|MPE_ENC_ENCRYPTION_REQUIRED |0x8088021E |El contenido debe estar cifrado, es necesaria una directiva de entrega para el formato de salida. |
|MPE_ENC_ENCRYPTION_NOT_SET_IN_DELIVERY_POLICY |0x8088024D |No ha establecido un cifrado en la configuración de directiva de entrega. |

## <a name="404-not-found"></a>404 No encontrado

La operación está intentando actuar en un recurso que ya no existe. Por ejemplo, puede que el recurso ya se haya eliminado.

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_EGRESS_TRACK_NOT_FOUND |0x80890209 |No se encuentra la pista solicitada. |
|MPE_RESOURCE_NOT_FOUND |0x808901F9 |No se encuentra el recurso solicitado. |
|MPE_UNAUTHORIZED |0x80890244 |El acceso no está autorizado. |
|MPE_EGRESS_TIMESTAMP_NOT_FOUND |0x8089020A |No se encuentra la marca de tiempo solicitada. |
|MPE_EGRESS_FILTER_NOT_FOUND |0x8089020C |No se encuentra el filtro de manifiesto dinámico solicitado. |
|MPE_FRAGMENT_BY_INDEX_NOT_FOUND |0x80890252 |El índice de fragmentos solicitado está fuera del intervalo válido. |
|MPE_LIVE_MEDIA_ENTRIES_NOT_FOUND |0x80890254 |No se encuentran las entradas de medios en directo para obtener el búfer moov. |
|MPE_FRAGMENT_TIMESTAMP_NOT_FOUND |0x80890255 |No se encuentra el fragmento en el momento solicitado de una determinada pista.<br/><br/>Podría ser que el fragmento no esté en el almacenamiento. Pruebe con otra capa de la presentación que podría tener un fragmento. |
|MPE_MANIFEST_MEDIA_ENTRY_NOT_FOUND |0x80890256 |No se encuentra la entrada de multimedia para la velocidad de bits solicitada en el manifiesto. <br/><br/>Podría ser que el reproductor pidiera una pista de vídeo de una determinada velocidad de bits que no se encontró en el manifiesto.|
|MPE_METADATA_NOT_FOUND |0x80890257 |No se pueden encontrar ciertos metadatos en el manifiesto o no se puede encontrar la fusión mediante cambio de base del almacenamiento. |
|MPE_STORAGE_RESOURCE_NOT_FOUND |0x808900ED |Error de operación de almacenamiento, no se encuentra el recurso. |

## <a name="409-conflict"></a>409 Conflicto

El identificador proporcionado para un recurso en una operación `PUT` o `POST` lo ha tomado un recurso existente. Use otro identificador para el recurso para resolver este problema.

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_STORAGE_CONFLICT  |0x808900EE  |Error de operación de almacenamiento, error de conflicto.  |

## <a name="410"></a>410

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_FILTER_FORCE_END_LEFT_EDGE_CROSSED_DVR_WINDOW|0x80890263|Para streaming en vivo, cuando el filtro que tiene forceEndTimestamp establecido en true, el inicio o el final de la marca de tiempo está fuera de la ventana de DVR actual.|

## <a name="412-precondition-failure"></a>412 Error en la condición previa

La operación especificó un valor eTag que es diferente de la versión disponible en el servidor, es decir, un error de simultaneidad optimista. Vuelva a intentar la solicitud después de leer la versión más reciente del recurso y de actualizar el valor eTag en la solicitud.

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_FRAGMENT_NOT_READY |0x80890200 |El fragmento solicitado no está listo.|
|MPE_STORAGE_PRECONDITION_FAILED| 0x808900EF|Error de operación de almacenamiento, un error de la condición previa.|

## <a name="415-unsupported-media-type"></a>415 Tipo de medio no compatible

El formato de carga enviado por el cliente está en un formato no admitido.

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_ENC_ALREADY_ENCRYPTED| 0x8088021F| No se debe cifrar el contenido ya cifrado.|
|MPE_ENC_INVALID_INPUT_ENCRYPTION_FORMAT|0x8088021D |El cifrado no es válido para el formato de entrada.|
|MPE_INVALID_ASSET_DELIVERY_POLICY_TYPE|0x8088021C| El tipo de directiva de entrega no es válido.|
|MPE_ENC_MULTIPLE_SAME_DELIVERY_TYPE|0x8088024E |La configuración original se pudo compartir con varios formatos de salida.|
|MPE_FORMAT_NOT_SUPPORTED|0x80890205|El tipo o el formato multimedia no es compatible. Por ejemplo, Media Services no admite un número de nivel de calidad mayor de 64. En la etiqueta de vídeo FLV, Media Services no admite un fotograma de vídeo con varios SPS y varios PPS.|
|MPE_INPUT_FORMAT_NOT_SUPPORTED|0x80890218| El formato de entrada del recurso solicitado no es compatible. Media Services admite Smooth (en directo), MP4 (VoD) y formatos de descarga progresiva.|
|MPE_OUTPUT_FORMAT_NOT_SUPPORTED|0x8089020D|El formato de salida solicitado no es compatible. Media Services admite Smooth, DASH (CSF, CMAF), HLS (v3, v4, CMAF) y formatos de descarga progresiva.|
|MPE_ENCRYPTION_NOT_SUPPORTED|0x80890208|Se detectó un tipo de cifrado no admitido.|
|MPE_MEDIA_TYPE_NOT_SUPPORTED|0x8089020E|El tipo de medio solicitado no es compatible con el formato de salida. Los tipos admitidos son vídeo, audio o subtítulo "SUBT".|
|MPE_MEDIA_ENCODING_NOT_SUPPORTED|0x8089020F|Los medios del recurso de origen se codificaron con un formato multimedia que no es compatible con el formato de salida.|
|MPE_VIDEO_ENCODING_NOT_SUPPORTED|0x80890210|El recurso de origen se codificó con un formato de vídeo que no es compatible con el formato de salida. Se admite H.264 AVC, H.265 (HEVC, hev1 o hvc1).|
|MPE_AUDIO_ENCODING_NOT_SUPPORTED|0x80890211|El recurso de origen se codificó con un formato de audio que no es compatible con el formato de salida. Los formatos de audio admitidos son AAC, E-AC3 (DD +), Dolby DTS.|
|MPE_SOURCE_PROTECTION_CONVERSION_NOT_SUPPORTED|0x80890212|El recurso protegido de origen no se puede convertir al formato de salida.|
|MPE_OUTPUT_PROTECTION_FORMAT_NOT_SUPPORTED|0x80890213|El formato de protección no es compatible con el formato de salida.|
|MPE_INPUT_PROTECTION_FORMAT_NOT_SUPPORTED|0x80890219|El formato de protección no es compatible con el formato de entrada.|
|MPE_INVALID_VIDEO_NAL_UNIT|0x80890231|Unidad NAL de vídeo no válida, por ejemplo, solo la primera NAL del ejemplo puede ser un AUD.|
|MPE_INVALID_NALU_SIZE|0x80890260|Tamaño de unidad NAL no válido.|
|MPE_INVALID_NALU_LENGTH_FIELD|0x80890261|Valor de longitud de unidad NAL no válido.|
|MPE_FILTER_INVALID|0x80890236|Filtros de manifiesto dinámico no válidos.|
|MPE_FILTER_VERSION_INVALID|0x80890237|Versiones de filtro no válidas o incompatibles.|
|MPE_FILTER_TYPE_INVALID|0x80890238|Tipo de filtro no válido.|
|MPE_FILTER_RANGE_ATTRIBUTE_INVALID|0x80890239|El filtro ha especificado un intervalo no válido.|
|MPE_FILTER_TRACK_ATTRIBUTE_INVALID|0x8089023A|El filtro ha especificado un atributo de pista no válido.|
|MPE_FILTER_PRESENTATION_WINDOW_INVALID|0x8089023B|El filtro ha especificado una longitud de ventana de presentación no válida.|
|MPE_FILTER_LIVE_BACKOFF_INVALID|0x8089023C|El filtro ha especificado una interrupción en vivo no válida.|
|MPE_FILTER_MULTIPLE_SAME_TYPE_FILTERS|0x8089023D|Solo se admite un elemento absTimeInHNS en filtros heredados.|
|MPE_FILTER_REMOVED_ALL_STREAMS|0x8089023E|No hay más secuencias después de aplicar los filtros.|
|MPE_FILTER_LIVE_BACKOFF_OVER_DVRWINDOW|0x8089023F|La interrupción en vivo está fuera de la ventana de DVR.|
|MPE_FILTER_LIVE_BACKOFF_OVER_PRESENTATION_WINDOW|0x80890262|La interrupción en vivo es mayor que la ventana de presentación.|
|MPE_FILTER_COMPOSITION_FILTER_COUNT_OVER_LIMIT|0x80890246|Se ha superado el máximo de diez (10) filtros predeterminados permitido.|
|MPE_FILTER_COMPOSITION_MULTIPLE_FIRST_QUALITY_OPERATOR_NOT_ALLOWED|0x80890248|No se permite el operador de calidad de varios primeros vídeos en los filtros de solicitudes combinadas.|
|MPE_FILTER_FIRST_QUALITY_ATTRIBUTE_INVALID|0x80890249|El número de atributos de velocidad de bits de primera calidad debe ser uno (1).|
|MPE_HLS_SEGMENT_TOO_LARGE|0x80890243|La duración del segmento HLS debe ser menor de un tercio de la ventana DVR y la interrupción HLS.|
|MPE_KEY_FRAME_INTERVAL_TOO_LARGE|0x808901FE|Las duraciones de los fragmentos deben ser menor o igual a aproximadamente 20 segundos o los niveles de calidad de entrada no se alinearán con el tiempo.|
|MPE_DTS_RESERVEDBOX_EXPECTED|0x80890105|Error específico de DTS; no se encuentra el elemento ReservedBox cuando debería estar presente en el elemento DTSSpecficBox durante el análisis del cuadro DTS.|
|MPE_DTS_INVALID_CHANNEL_COUNT|0x80890106|Error específico de DTS; no se ha encontrado ningún canal en el elemento DTSSpecficBox durante el análisis del cuadro DTS.|
|MPE_DTS_SAMPLETYPE_MISMATCH|0x80890107|Error específico de DTS; error de coincidencia de tipos de ejemplo en el DTSSpecficBox.|
|MPE_DTS_MULTIASSET_DTSH_MISMATCH|0x80890108|Error específico de DTS; se han establecido varios recursos pero hay un error de coincidencia de tipos de ejemplo de DTSH.|
|MPE_DTS_INVALID_CORESTREAM_SIZE|0x80890109|Error específico de DTS; el tamaño de la secuencia principal no es válido.|
|MPE_DTS_INVALID_SAMPLE_RESOLUTION|0x8089010A|Error específico de DTS; la resolución de ejemplo no es válida.|
|MPE_DTS_INVALID_SUBSTREAM_INDEX|0x8089010B|Error específico de DTS; el índice de la extensión de secuencia secundaria no es válido.|
|MPE_DTS_INVALID_BLOCK_NUM|0x8089010C|Error específico de DTS; el número de bloque de secuencia secundaria no es válido.|
|MPE_DTS_INVALID_SAMPLING_FREQUENCE|0x8089010D|Error específico de DTS; la frecuencia de muestreo no es válida.|
|MPE_DTS_INVALID_REFCLOCKCODE|0x8089010E|Error específico de DTS; el código de reloj de referencia en la extensión de secuencia secundaria no es válido.|
|MPE_DTS_INVALID_SPEAKERS_REMAP|0x8089010F|Error específico de DTS; el número de conjunto de reasignación de los altavoces no es válido.|

Para obtener artículos y ejemplos de cifrado, vea:

- [Concepto: Protección de contenido](content-protection-overview.md)
- [Concepto: Directivas de claves de contenido](content-key-policy-concept.md)
- [Concepto: Directivas de streaming](streaming-policy-concept.md)
- [Ejemplo: Protección con el cifrado AES](protect-with-aes128.md)
- [Ejemplo: Protección con DRM](protect-with-drm.md)

Para obtener instrucciones de filtro, vea:

- [Concepto: manifiestos dinámicos](filters-dynamic-manifest-overview.md)
- [Concepto: filtros](filters-concept.md)
- [Ejemplo: Creación de filtros con las API REST](filters-dynamic-manifest-rest-howto.md)
- [Ejemplo: Creación de filtros con .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Ejemplo: Creación de filtros con la CLI](filters-dynamic-manifest-cli-howto.md)

Para obtener ejemplos y artículos en directo, vea:

- [Concepto: Introducción al streaming en vivo](live-streaming-overview.md)
- [Concepto: Objetos LiveEvent y LiveOutput](live-events-outputs-concept.md)
- [Ejemplo: Tutorial de streaming en vivo](stream-live-tutorial-with-api.md)

## <a name="416-range-not-satisfiable"></a>416 No se puede satisfacer el intervalo

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_STORAGE_INVALID_RANGE|0x808900F1|Error de operación de almacenamiento; devolvió un error http 416, intervalo no válido.|

## <a name="500-internal-server-error"></a>Error de servidor interno 500

Durante el procesamiento de la solicitud, Media Services encuentra algún error que impide que el procesamiento continúe.  

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_STORAGE_SOCKET_TIMEOUT|0x808900F4|Recibido y trasladado del código de error de Winhttp ERROR_WINHTTP_TIMEOUT (0x00002ee2).|
|MPE_STORAGE_SOCKET_CONNECTION_ERROR|0x808900F5|Recibido y trasladado del código de error de Winhttp ERROR_WINHTTP_CONNECTION_ERROR (0x00002efe).|
|MPE_STORAGE_SOCKET_NAME_NOT_RESOLVED|0x808900F6|Recibido y trasladado del código de error de Winhttp ERROR_WINHTTP_NAME_NOT_RESOLVED (0x00002ee7).|
|MPE_STORAGE_INTERNAL_ERROR|0x808900E6|Error de operación de almacenamiento; InternalError general de uno de los errores HTTP 500.|
|MPE_STORAGE_OPERATION_TIMED_OUT|0x808900E7|Error de operación de almacenamiento; OperationTimedOut general de uno de los errores HTTP 500.|
|MPE_STORAGE_FAILURE|0x808900F2|Error de operación de almacenamiento; otros errores de HTTP 500 distintos de InternalError u OperationTimedOut.|

## <a name="503-service-unavailable"></a>Servicio no disponible 503

El servidor actualmente no puede recibir solicitudes. Este error puede deberse a solicitudes excesivas en el servicio. El mecanismo de limitación de Media Services restringe el uso de recursos en las aplicaciones que realizan un número excesivo de solicitudes al servicio.

> [!NOTE]
> Compruebe el mensaje de error y la cadena de error para obtener más información detallada sobre la razón por la que tiene el error 503. Este error no siempre implica limitación.
> 

|Código de error|Valor hexadecimal |Descripción del error|
|---|---|---|
|MPE_STORAGE_SERVER_BUSY|0x808900E8|Error de operación de almacenamiento; se recibió el error de servidor ocupado HTTP 503.|

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="see-also"></a>Consulte también

- [Códigos de error de codificación](/rest/api/media/jobs/get#joberrorcode)
- [Conceptos de Azure Media Services](concepts-overview.md)
- [Cuotas y límites](limits-quotas-constraints.md)

## <a name="next-steps"></a>Pasos siguientes

[Ejemplo: acceder a ErrorCode y Message desde ApiException con .NET](configure-connect-dotnet-howto.md#connect-to-the-net-client)
