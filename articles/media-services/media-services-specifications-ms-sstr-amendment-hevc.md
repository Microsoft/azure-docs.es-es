---
title: 'Modificación del protocolo Smooth Streaming (MS-SSTR) para HEVC: Azure'
description: En esta especificación se describe el protocolo y el formato para streaming en vivo basado en MP4 fragmentado para Azure Media Services. En este artículo solo se especifican los cambios necesarios para entregar HEVC, excepto cuando se incluya "(Sin cambios)" para indicar que el texto se copia solo con fines de aclaración.
services: media-services
documentationcenter: ''
author: johndeu
manager: femila
editor: ''
ms.assetid: f27d85de-2cb8-4269-8eed-2efb566ca2c6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/19/2019
ms.author: johndeu
ms.openlocfilehash: 6454bc863cb5fd628d581fff380c5ab61354f762
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87053052"
---
# <a name="smooth-streaming-protocol-ms-sstr-amendment-for-hevc"></a>Modificación del protocolo Smooth Streaming (MS-SSTR) para HEVC 

## <a name="1-introduction"></a>1 Introducción 

En este artículo se proporcionan las enmiendas detalladas que se aplican a la especificación del protocolo Smooth Streaming [MS-SSTR] para habilitar el vídeo de Smooth Streaming codificado en HEVC. En esta especificación, se describen solo los cambios necesarios para entregar el códec de vídeo HEVC. El artículo sigue el mismo esquema de numeración que la especificación [MS-SSTR]. Los títulos vacíos que se presentan en este artículo se proporcionan para orientar al lector sobre su posición en la especificación [MS-SSTR].  "(Sin cambios)" indica que el texto se copia únicamente con fines de aclaración.

El artículo proporciona los requisitos de implementación técnicos para la señal del códec de vídeo HEVC (mediante las pistas de formato "hev1" o "hvc1") en un manifiesto de Smooth Streaming y las referencias de la normativa se actualizan para hacer referencia a los estándares MPEG actuales que incluyen HEVC; el cifrado común de HEVC y los nombres de cuadro del formato de archivo multimedia básico ISO se han actualizado por coherencia con las especificaciones más recientes. 

La especificación del protocolo Smooth Streaming [MS-SSTR] describe el formato de conexión usado para entregar medios digitales en directo y bajo petición, como audio y vídeo, de las maneras siguientes: desde un codificador a un servidor web, desde un servidor a otro servidor y desde un servidor a un cliente HTTP.
El uso de una entrega de estructura de datos basada en MPEG-4 ([[MPEG4-RA])](https://go.microsoft.com/fwlink/?LinkId=327787) sobre HTTP permite una conmutación perfecta casi en tiempo real entre los distintos niveles de calidad del contenido multimedia comprimido. El resultado es una experiencia de reproducción constante para el usuario final del cliente HTTP, aunque cambien las condiciones de representación de vídeo y de red para el dispositivo o equipo cliente.

## <a name="11-glossary"></a>1.1 Glosario 

Los siguientes términos se definen en *[MS-GLOS]* :

>   **identificador único global (GUID) identificador único universal (UUID)**

Los siguientes términos son específicos de este documento:

>  **hora de composición:** la hora a la que se presenta una muestra en el cliente, tal como se define en [[ISO/IEC-14496-12]](https://go.microsoft.com/fwlink/?LinkId=183695).
> 
>   **CENC**: Common Encryption, tal como se define en [ISO/IEC 23001-7] Segunda edición.
> 
>   **hora de descodificación:** la hora a la que se solicita la descodificación de una muestra en el cliente, tal como se define en [[ISO/IEC 14496-12:2008]](https://go.microsoft.com/fwlink/?LinkId=183695).

**fragmento:** una unidad descargable de forma independiente de **elementos multimedia** formada por una o varias **muestras**.

>   **HEVC:** High Efficiency Video Coding (Codificación de vídeo de alta eficacia), tal como se define en [ISO/IEC 23008-2]
> 
>   **manifiesto:** metadatos sobre la **presentación** que permiten a un cliente realizar solicitudes de **elementos multimedia**. **Elementos multimedia:** datos de audio, vídeo y texto comprimidos utilizados por el cliente para reproducir una **presentación**. **formato multimedia:** un formato bien definido para representar audio o vídeo como una **muestra** comprimida.
> 
>   **presentación:** el conjunto de todas las **secuencias** y metadatos relacionados necesarios para reproducir una sola película. **solicitud:** un mensaje HTTP enviado desde el cliente al servidor, como se define en [[RFC2616]](https://go.microsoft.com/fwlink/?LinkId=90372) **respuesta:** un mensaje HTTP enviado desde el servidor al cliente, como se define en [[RFC2616]](https://go.microsoft.com/fwlink/?LinkId=90372)
> 
>   **muestra:** la unidad fundamental más pequeña (por ejemplo, un fotograma) en la que los **elementos multimedia** se almacenan y procesan.
> 
>   **MAY, SHOULD, MUST, SHOULD NOT, MUST NOT:** Estos términos (en mayúsculas) se utilizan como se describe en [[RFC2119]](https://go.microsoft.com/fwlink/?LinkId=90317). Todas las instrucciones de comportamiento opcional utilizan MAY, SHOULD o SHOULD NOT.

## <a name="12-references"></a>1.2 Referencias

>   Las referencias a la documentación de Microsoft Open Specifications no incluyen el año de publicación porque los vínculos llevan a la versión más reciente de los documentos, que se actualizan con frecuencia. Las referencias a otros documentos incluyen el año de publicación cuando está disponible.

### <a name="121-normative-references"></a>1.2.1 Referencias de normativa 

>  [MS-SSTR] Protocolo Smooth Streaming *v20140502* [https://msdn.microsoft.com/library/ff469518.aspx](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251)
> 
>   [ISO/IEC 14496-12] International Organization for Standardization, "Tecnologías de la información: codificación de objetos audiovisuales, parte 12: formato de archivo multimedia básico ISO", ISO/IEC 14496-12:2014, Edición 4, además de la rectificación 1 y las enmiendas 1 y 2.
>   <https://standards.iso.org/ittf/PubliclyAvailableStandards/c061988_ISO_IEC_14496-12_2012.zip>
> 
>   [ISO/IEC 14496-15] International Organization for Standardization, "Tecnologías de la información: codificación de objetos audiovisuales, parte 15: transporte de vídeo estructurado de unidad NAL en el formato de archivo multimedia básico ISO", ISO 14496-15:2015, Edición 3.
>   <https://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=65216>
> 
>   [ISO/IEC 23008-2] Tecnologías de la información: entrega multimedia y codificación de alta eficacia en entornos heterogéneos: parte 2: codificación de vídeo de alta eficacia: 2013 o más reciente   <https://standards.iso.org/ittf/PubliclyAvailableStandards/c035424_ISO_IEC_23008-2_2013.zip>
> 
>   [ISO/IEC 23001-7] Tecnologías de la información, tecnologías de sistemas MPEG, parte 7: cifrado común en archivos del formato de archivo multimedia básico ISO, CENC Edición 2:2015 <https://www.iso.org/iso/catalogue_detail.htm?csnumber=65271>
> 
>   [RFC-6381] IETF RFC-6381, Los parámetros "Codecs" y "Profiles" en tipos multimedia Bucket <https://tools.ietf.org/html/rfc6381>
> 
>   [MPEG4-RA] La entidad de registro de MP4, "MP4REG", [http://www.mp4ra.org](https://go.microsoft.com/fwlink/?LinkId=327787)
> 
>   [RFC2119] Bradner, S., "Palabras clave para su uso en RFC para indicar niveles de requisitos", BCP 14, RFC 2119, marzo de 1997,   [https://www.rfc-editor.org/rfc/rfc2119.txt](https://go.microsoft.com/fwlink/?LinkId=90317)

### <a name="122-informative-references"></a>1.2.2 Referencias informativas 

>   [MS-GLOS] Microsoft Corporation, "*Glosario maestro de protocolos de Windows*".
> 
>   [RFC3548] Josefsson, S., Ed., "Codificaciones de datos Base 16, Base 32 y Base 64", RFC 3548, julio de 2003, [https://www.ietf.org/rfc/rfc3548.txt](https://go.microsoft.com/fwlink/?LinkId=90432)
> 
>   [RFC5234] Crocker, D., Ed. y Overell, P., "BNF ampliado para especificaciones de sintaxis: ABNF", STD 68, RFC 5234, enero de 2008,   [https://www.rfc-editor.org/rfc/rfc5234.txt](https://go.microsoft.com/fwlink/?LinkId=123096)


## <a name="13-overview"></a>1.3 Introducción 

>   Solo se indican a continuación los cambios de la especificación de Smooth Streaming necesarios para la entrega de HEVC. Los encabezados de las secciones sin modificaciones se enumeran para mantener la ubicación en la referida especificación de Smooth Streaming [MS-SSTR].

## <a name="14-relationship-to-other-protocols"></a>1.4 Relación con otros protocolos 

## <a name="15-prerequisitespreconditions"></a>1.5 Requisitos previos y condiciones previas 

## <a name="16-applicability-statement"></a>1.6 Declaración de aplicabilidad 

## <a name="17-versioning-and-capability-negotiation"></a>1.7 Control de versiones y negociación de funcionalidad 

## <a name="18-vendor-extensible-fields"></a>1.8 Campos extensibles por el proveedor 

>   Se usará el método siguiente para identificar secuencias con el formato de vídeo HEVC:
> 
>   * **Códigos descriptivos personalizados para formatos multimedia:** esta funcionalidad la proporciona el campo **FourCC**, como se especifica en la sección *2.2.2.5*.
>   Los implementadores pueden asegurarse de que las extensiones no entran en conflicto mediante el registro de códigos de extensión en MPEG4-RA, como se especifica en [[ISO/IEC-14496-12]](https://go.microsoft.com/fwlink/?LinkId=183695)

## <a name="19-standards-assignments"></a>1.9 Asignaciones de estándares 

## <a name="2-messages"></a>2 Mensajes 

## <a name="21-transport"></a>2.1 Transporte

## <a name="22-message-syntax"></a>2.2 Sintaxis de mensajes 

### <a name="221-manifest-request"></a>2.2.1 Solicitud de manifiesto 

### <a name="222-manifest-response"></a>2.2.2 Respuesta del manifiesto 

#### <a name="2221-smoothstreamingmedia"></a>2.2.2.1 SmoothStreamingMedia 

>   **MinorVersion (variable):** la versión secundaria del mensaje de respuesta del manifiesto. DEBE establecerse en 2. (Sin cambios)
> 
>   **TimeScale (variable):** la escala de tiempo del atributo Duration, especificada como el número de incrementos en un segundo. El valor predeterminado es
> 1. (Sin cambios)
> 
>    El valor recomendado es 90000 para representar la duración exacta de los marcos y fragmentos que contienen vídeo de velocidad en fracciones (por ejemplo, 30/1,001 Hz).

#### <a name="2222-protectionelement"></a>2.2.2.2 ProtectionElement 

ProtectionElement DEBERÁ estar presente cuando se ha aplicado cifrado común (CENC) a las secuencias de vídeo o audio. Las secuencias cifradas HEVC DEBERÁN cumplir con la especificación de cifrado común 2ª Edición [ISO/IEC 23001-7]. Solo DEBERÁN cifrarse los datos del segmento en unidades NAL VCL.

#### <a name="2223-streamelement"></a>2.2.2.3 StreamElement 

>   **StreamTimeScale (variable):** la escala de tiempo de los valores de duración y hora de esta secuencia, especificada como el número de incrementos en un segundo. Para las secuencias HEVC se recomienda un valor de 90 000. Para las secuencias de audio se recomienda un valor que coincida con la frecuencia de muestreo de la forma de onda (por ejemplo, 48 000 o 44 100).

##### <a name="22231-streamprotectionelement"></a>2.2.2.3.1 StreamProtectionElement

#### <a name="2224-urlpattern"></a>2.2.2.4 UrlPattern 

#### <a name="225-trackelement"></a>2.2.5 TrackElement 

>   **FourCC (variable):** un código de cuatro caracteres que identifica el formato multimedia que se utiliza para cada muestra. El siguiente intervalo de valores está reservado con el significado semántico siguiente:
> 
> * "hev1": las muestras de vídeo de esta pista utilizan vídeo HEVC, con el formato de descripción de muestras "hev1" especificado en [ISO/IEC-14496-15].
>
> * "hvc1”: las muestras de vídeo de esta pista utilizan vídeo HEVC, con el formato de descripción de muestras "hvc1" especificado en [ISO/IEC-14496-15].
> 
>   **CodecPrivateData (variable):** datos que especifican los parámetros específicos del formato multimedia y son comunes a todas las muestras de la pista, representados como una cadena de bytes codificada en hexadecimal. El formato y el significado semántico de la secuencia de bytes varía según el valor del campo **FourCC**, como se indica a continuación:
> 
>   * Cuando un elemento TrackElement describe vídeo HEVC, el campo **FourCC** deberá ser igual a **"hev1"** o **"hvc1"** .
> 
>   El campo **CodecPrivateData** deberá contener una representación de cadena codificada en hexadecimal de la siguiente secuencia de bytes, como se especifica en ABNF [[RFC5234]:](https://go.microsoft.com/fwlink/?LinkId=123096) (ningún cambio respecto a MS-SSTR)
> 
>   * %x00 %x00 %x00 %x01 SPSField %x00 %x00 %x00 %x01 PPSField
> 
>   * SPSField contiene el conjunto de parámetros de la secuencia (SPS).
> 
>   * PPSField contiene el conjunto de parámetros del segmento (PPS).
> 
>   Nota: El conjunto de parámetros de vídeo (VPS) no se encuentra en CodecPrivateData, pero debe incluirse en el encabezado del archivo de los archivos almacenados en el cuadro "hvcC". Los sistemas que utilizan el protocolo Smooth Streaming deben señalar los parámetros de decodificación adicionales (por ejemplo, el nivel de HEVC) mediante el atributo personalizado "codecs".

##### <a name="22251-customattributeselement"></a>2.2.2.5.1 CustomAttributesElement 

#### <a name="226-streamfragmentelement"></a>2.2.6 StreamFragmentElement 

>   El campo **MajorVersion de SmoothStreamingMedia** DEBE establecerse en 2 y el campo **MinorVersion** DEBE establecerse en 2. (Sin cambios)

##### <a name="22261-trackfragmentelement"></a>2.2.2.6.1 TrackFragmentElement 

### <a name="223-fragment-request"></a>2.2.3 Solicitud de fragmento 

>   **Nota**: El formato multimedia solicitado para **MinorVersion** 2 y "hev1" o "hvc1" es "iso8" según el formato de archivo multimedia básico ISO especificado en [ISO/IEC 14496-12] Formato de archivo multimedia básico ISO Cuarta edición e [ISO/IEC 23001-7] Cifrado común, segunda edición.

### <a name="224-fragment-response"></a>2.2.4 Respuesta de fragmento 

#### <a name="2241-moofbox"></a>2.2.4.1 MoofBox 

#### <a name="2242-mfhdbox"></a>2.2.4.2 MfhdBox 

#### <a name="2243-trafbox"></a>2.2.4.3 TrafBox 

#### <a name="2244-tfxdbox"></a>2.2.4.4 TfxdBox 

>   **TfxdBox** está en desuso y su función ha sido reemplazada por el cuadro Track Fragment Decode Time ("tfdt") (especificado en la sección 8.8.12 de [ISO/IEC 14496-12]).
> 
>   **Nota**: Un cliente puede calcular la duración de un fragmento sumando las duraciones de las muestras enumeradas en el cuadro Run Box ("trun") o multiplicando el número de muestras por la duración de muestra predeterminada. El valor de baseMediaDecodeTime en "tfdt" más la duración del fragmento es igual al parámetro time de la dirección URL del siguiente fragmento.
> 
>   Se debería insertar un cuadro Producer Reference Time ("prft") antes de un cuadro Movie Fragment ("moof") según sea necesario, para indicar la hora UTC correspondiente al elemento Track Fragment Decode Time de la primera muestra a la que hace referencia el cuadro Movie Fragment, como se especifica en la sección 8.16.5 de [ISO/IEC 14496 -12].

#### <a name="2245-tfrfbox"></a>2.2.4.5 TfrfBox 

>   **TfrfBox** está en desuso y su función ha sido reemplazada por el cuadro Track Fragment Decode Time ("tfdt") (especificado en la sección 8.8.12 de [ISO/IEC 14496-12]).
> 
>   **Nota**: Un cliente puede calcular la duración de un fragmento sumando las duraciones de las muestras enumeradas en el cuadro Run Box ("trun") o multiplicando el número de muestras por la duración de muestra predeterminada. El valor de baseMediaDecodeTime en "tfdt" más la duración del fragmento es igual al parámetro time de la dirección URL del siguiente fragmento. Las direcciones de búsqueda hacia delante están en desuso porque retrasan el streaming en vivo.

#### <a name="2246-tfhdbox"></a>2.2.4.6 TfhdBox 

>   **TfhdBox** y los campos relacionados encapsulan los valores predeterminados de los metadatos de cada muestra del fragmento. La sintaxis del campo **TfhdBox** es un subconjunto estricto de la sintaxis del cuadro Track Fragment Header definido en la sección 8.8.7 de [[ISO/IEC-14496-12]](https://go.microsoft.com/fwlink/?LinkId=183695).
> 
>   **BaseDataOffset (8 bytes):** el desplazamiento, en bytes, desde el principio del campo **MdatBox** hasta el campo de la muestra en el campo **MdatBox**. Para indicar esta restricción, se debe establecer la marca default-base-is-moof (0x020000).

#### <a name="2247-trunbox"></a>2.2.4.7 TrunBox 

>   **TrunBox** y los campos relacionados encapsulan los metadatos de cada muestra del fragmento solicitado. La sintaxis de **TrunBox** es un subconjunto estricto de la versión 1 del cuadro Track Fragment Run definido en la sección 8.8.8 de [[ISO/IEC-14496-](https://go.microsoft.com/fwlink/?LinkId=183695)*12]* .
> 
>   **SampleCompositionTimeOffset (4 bytes):** el desplazamiento de la hora de composición de la muestra de cada muestra ajustado para que la hora de presentación de la primera muestra presentada en el fragmento sea igual a la hora de decodificación de la primera muestra decodificada. Se DEBERÁN utilizar desplazamientos de composición de la muestra de vídeo negativos,
> 
>   tal como se define en [[ISO/IEC-14496-12].](https://go.microsoft.com/fwlink/?LinkId=183695)
> 
>   Nota: Esto evita un error de sincronización de vídeo provocado por un retraso de vídeo respecto al audio igual al retraso por eliminación del búfer de imagen descodificada más grande y mantiene los tiempos de presentación entre fragmentos alternativos que pueden tener retrasos de eliminación diferentes.
> 
>   La sintaxis de los campos definidos en esta sección, tal como se especifica en ABNF [[RFC5234],](https://go.microsoft.com/fwlink/?LinkId=123096) sigue siendo la misma, excepto en lo que se indica a continuación:
> 
>   SampleCompositionTimeOffset = SIGNED_INT32

#### <a name="2248-mdatbox"></a>2.2.4.8 MdatBox 

#### <a name="2249-fragment-response-common-fields"></a>2.2.4.9 Campos comunes de la respuesta de fragmento 

### <a name="225-sparse-stream-pointer"></a>2.2.5 Puntero de secuencia dispersa 

### <a name="226-fragment-not-yet-available"></a>2.2.6 Fragmento no disponible aún 

### <a name="227-live-ingest"></a>2.2.7 Ingestión en vivo 

#### <a name="2271-filetype"></a>2.2.7.1 FileType 

>   **FileType (variable):** especifica el subtipo y el uso previsto del archivo MPEG-4 ([[MPEG4-RA])](https://go.microsoft.com/fwlink/?LinkId=327787) y los atributos de alto nivel.
> 
>   **MajorBrand (variable):** la marca principal del archivo multimedia. DEBE establecerse en "isml".
> 
>   **MinorVersion (variable):** la versión secundaria del archivo multimedia. DEBE establecerse en 1.
> 
>   **CompatibleBrands (variable):** especifica las marcas compatibles de MPEG-4.
>   DEBE incluir "ccff" e "iso8".
> 
>   La sintaxis de los campos definidos en esta sección, tal como se especifica en ABNF [[RFC5234],](https://go.microsoft.com/fwlink/?LinkId=123096) es la siguiente:

```properties
FileType = MajorBrand MinorVersion CompatibleBrands
MajorBrand = STRING_UINT32
MinorVersion = STRING_UINT32
CompatibleBrands = "ccff" "iso8" 0\*(STRING_UINT32)
```

**Nota**: Las marcas de compatibilidad "ccff" e "iso8" indican que los fragmentos se ajustan al "Common Container File Format" (Formato de archivo de contenedor común) con cifrado común [ISO/IEC 23001-7] y al formato de archivo multimedia básico ISO Edición 4 [ISO/IEC 14496-12].

#### <a name="2272-streammanifestbox"></a>2.2.7.2 StreamManifestBox 

##### <a name="22721-streamsmil"></a>2.2.7.2.1 StreamSMIL 

#### <a name="2273-liveservermanifestbox"></a>2.2.7.3 LiveServerManifestBox 

##### <a name="22731-livesmil"></a>2.2.7.3.1 LiveSMIL 

#### <a name="2274-moovbox"></a>2.2.7.4 MoovBox 

#### <a name="2275-fragment"></a>2.2.7.5 Fragmento 

##### <a name="22751-track-fragment-extended-header"></a>2.2.7.5.1 Encabezado extendido del fragmento de pista 

### <a name="228-server-to-server-ingest"></a>2.2.8 Ingestión de servidor a servidor 

## <a name="3-protocol-details"></a>3 Detalles del protocolo 


## <a name="31-client-details"></a>3.1 Detalles del cliente 

### <a name="311-abstract-data-model"></a>3.1.1 Modelo de datos abstracto 

#### <a name="3111-presentation-description"></a>3.1.1.1 Presentation Description 

>   El elemento de datos Presentation Description (Descripción de la presentación) encapsula todos los metadatos de la presentación.
> 
>   Metadatos de presentación: un conjunto de metadatos que es común a todas las secuencias de la presentación. Presentation Metadata consta de los siguientes campos, especificados en la sección *2.2.2.1*:
> 
> * **MajorVersion**
> * **MinorVersion**
> * **TimeScale**
> * **Duration**
> * **IsLive**
> * **LookaheadCount**
> * **DVRWindowLength**
> 
>   Las presentaciones que contienen secuencias HEVC DEBERÁN establecer:

```properties
MajorVersion = 2
MinorVersion = 2
```

>   LookaheadCount = 0 (Nota: Cuadros en desuso)
> 
>   Las presentaciones también DEBERÍAN establecer:

```properties
TimeScale = 90000
```

>   Colección de secuencia: una colección de elementos de datos Stream Description, como se especifica en la sección *3.1.1.1.2*.
> 
>   Descripción de la protección: una colección de elementos de datos Protection System Metadata Description, como se especifica en la sección *3.1.1.1.1*.

##### <a name="31111-protection-system-metadata-description"></a>3.1.1.1.1 Descripción de metadatos del sistema de protección 

>   El elemento de datos Protection System Metadata Description (Descripción de metadatos del sistema de protección) encapsula los metadatos específicos de un único sistema de protección de contenido. (Sin cambios)
> 
>   Descripción del encabezado de protección: metadatos de protección de contenido que pertenecen a un único sistema de Content Protection. Protection Header Description consta de los siguientes campos, especificados en la sección *2.2.2.2*:
> 
>   * **SystemID**
>   * **ProtectionHeaderContent**

##### <a name="31112-stream-description"></a>3.1.1.1.2 Descripción de la secuencia 

###### <a name="311121-track-description"></a>3.1.1.1.2.1 Descripción de la pista 

###### <a name="3111211-custom-attribute-description"></a>3.1.1.1.2.1.1 Descripción de atributos personalizados 

##### <a name="3113-fragment-reference-description"></a>3.1.1.3 Descripción de la referencia de fragmento 

###### <a name="31131-track-specific-fragment-reference-description"></a>3.1.1.3.1 Descripción de la referencia de fragmento específico de una pista 

#### <a name="3112-fragment-description"></a>3.1.1.2 Descripción del fragmento 

##### <a name="31121-sample-description"></a>3.1.1.2.1 Descripción de la muestra 

### <a name="312-timers"></a>3.1.2 Temporizadores 

### <a name="313-initialization"></a>3.1.3 Inicialización 

### <a name="314-higher-layer-triggered-events"></a>3.1.4 Eventos desencadenados al más alto nivel 

#### <a name="3141-open-presentation"></a>3.1.4.1 Abrir presentación 

#### <a name="3142-get-fragment"></a>3.1.4.2 Obtener fragmento 

#### <a name="3143-close-presentation"></a>3.1.4.3 Cerrar presentación 

### <a name="315-processing-events-and-sequencing-rules"></a>3.1.5 Procesamiento de eventos y reglas de secuenciación 

#### <a name="3151-manifest-request-and-manifest-response"></a>3.1.5.1 Solicitud de manifiesto y respuesta del manifiesto 

#### <a name="3152-fragment-request-and-fragment-response"></a>3.1.5.2 Solicitud de fragmento y respuesta de fragmento

## <a name="32-server-details"></a>3.2 Detalles del servidor

## <a name="33-live-encoder-details"></a>3.3 Detalles de codificador en directo 

## <a name="4-protocol-examples"></a>4 Ejemplos de protocolo 

## <a name="5-security"></a>5 Seguridad 

## <a name="51-security-considerations-for-implementers"></a>5.1 Consideraciones de seguridad para los implementadores

>   Si el contenido que se transporta mediante este protocolo tiene un alto valor comercial, debe utilizarse un sistema de protección de contenido para impedir el uso no autorizado del contenido. Se puede utilizar **ProtectionElement** para transportar los metadatos relacionados con el uso de un sistema de protección de contenido. El contenido de vídeo y audio protegido DEBERÁ cifrarse tal y como se especifica en Cifrado común de MPEG Segunda edición: 2015 [ISO/IEC 23001-7].
> 
>   **Nota**: Para vídeo HEVC, solo se cifran los datos de segmento en los elementos NAL de VCL. Los encabezados de segmento y otros NAL son accesibles a las aplicaciones de presentación antes del descifrado. En una ruta de acceso de vídeo segura, la información cifrada no está disponible para las aplicaciones de presentación.

## <a name="52-index-of-security-parameters"></a>5.2 Índice de parámetros de seguridad 


| **Parámetro de seguridad**  | **Sección**         |
|-------------------------|---------------------|
| ProtectionElement       | *2.2.2.2*           |
| Cuadros de cifrado común | *[ISO/IEC 23001-7]* |

## <a name="53-common-encryption-boxes"></a>5.3 Cuadros de cifrado común

Los cuadros siguientes pueden estar presentes en las respuestas de fragmento cuando se aplica el cifrado común y se especifican en [ISO/IEC 23001-7] o [ISO/IEC 14496-12]:

1.  Protection System Specific Header Box ('pssh') (Cuadro de encabezado específico del sistema de protección)

2.  Sample Encryption Box (‘senc’) (Cuadro de cifrado de la muestra)

3.  Sample Auxiliary Information Offset Box (‘saio’) (Cuadro de desplazamiento de información auxiliar de la muestra)

4.  Sample Auxiliary Information Size Box (‘saiz’) (Cuadro de tamaño de información auxiliar de la muestra)

5.  Sample Group Description Box (‘sgpd’) (Cuadro de descripción del grupo de la muestra)

6.  Sample to Group Box (‘sbgp’) (Cuadro muestra a grupo)

---

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

[image1]: ./media/media-services-fmp4-live-ingest-overview/media-services-image1.png
[image2]: ./media/media-services-fmp4-live-ingest-overview/media-services-image2.png
[image3]: ./media/media-services-fmp4-live-ingest-overview/media-services-image3.png
[image4]: ./media/media-services-fmp4-live-ingest-overview/media-services-image4.png
[image5]: ./media/media-services-fmp4-live-ingest-overview/media-services-image5.png
[image6]: ./media/media-services-fmp4-live-ingest-overview/media-services-image6.png
[image7]: ./media/media-services-fmp4-live-ingest-overview/media-services-image7.png
