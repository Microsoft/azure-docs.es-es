---
title: Esquema de metadatos de entrada de Azure Media Services | Microsoft Docs
description: En este artículo se proporciona información general sobre el esquema de metadatos de entrada de Azure Media Services.
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
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 57a91622bef401d946a383e3be39f2e566fa50b4
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267843"
---
# <a name="input-metadata"></a>Metadatos de entrada

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Un trabajo de codificación está asociado a un recurso (o recursos) de entrada donde desea realizar algunas tareas de codificación.  Tras la finalización de una tarea, se produce un recurso de salida.  El recurso de salida contiene vídeo, audio, miniaturas, un manifiesto, etc. El recurso de salida también contiene un archivo con metadatos sobre el recurso de entrada. El nombre del archivo XML de metadatos tiene el formato siguiente: &lt;id_de_recurso&gt;_metadata.xml (por ejemplo, 41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml), donde &lt;id_de_recurso&gt; es el valor AssetId del recurso de entrada.  

Media Services no realiza un examen preventivo de los recursos de entrada para generar metadatos. Los metadatos de entrada se generan únicamente como artefactos cuando un recurso de entrada se procesa en un trabajo. Por tanto, estos artefactos se escriben en el recurso de salida. Para generar metadatos de los recursos de entrada y salida, se utilizan diferentes herramientas. Por tanto, los metadatos de entrada tienen un esquema ligeramente diferente al de los metadatos de salida.

Si desea examinar el archivo de metadatos, puede crear un localizador **SAS** y descargar el archivo en el equipo local. Puede encontrar un ejemplo de cómo crear un localizador SAS y descargar un archivo [con extensiones del SDK de .NET de Media Services](media-services-dotnet-get-started.md).  

En este artículo se describen los elementos y los tipos del esquema XML en que se basan los metadatos de entrada (&lt;id_de_recurso&gt;_metadata.xml).  Para información acerca del archivo que contiene metadatos sobre el recurso de salida, consulte [Output Metadata](media-services-output-metadata-schema.md) (Metadatos de salida).  

Puede encontrar el [código del esquema](media-services-input-metadata-schema.md#code) en un [ejemplo de XML](media-services-input-metadata-schema.md#xml) al final de este artículo.  
 

## <a name="assetfiles-element-root-element"></a><a name="AssetFiles"></a> Elemento AssetFiles (elemento raíz)
Contiene una colección de [elementos AssetFile](media-services-input-metadata-schema.md#AssetFile) para el trabajo de codificación.  

Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

| Nombre | Descripción |
| --- | --- |
| **AssetFile**<br /><br /> minOccurs="1" maxOccurs="unbounded" |Un único elemento secundario. Para más información, consulte [Elemento AssetFile](media-services-input-metadata-schema.md#AssetFile). |

## <a name="assetfile-element"></a><a name="AssetFile"></a> Elemento AssetFile
 Contiene atributos y elementos que describen un archivo de recursos.  

 Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Nombre**<br /><br /> Obligatorio |**xs:string** |Nombre del archivo de recursos. |
| **Tamaño**<br /><br /> Obligatorio |**xs:long** |Tamaño del archivo de recursos en bytes. |
| **Duration**<br /><br /> Obligatorio |**xs:duration** |Duración de la reproducción del contenido. Ejemplo: Duration="PT25M37.757S". |
| **NumberOfStreams**<br /><br /> Obligatorio |**xs:int** |Número de transmisiones en el archivo de recursos. |
| **FormatNames**<br /><br /> Obligatorio |**xs: string** |Nombres de formatos. |
| **FormatVerboseNames**<br /><br /> Obligatorio |**xs: string** |Nombres detallados de formatos. |
| **StartTime** |**xs:duration** |Hora de inicio del contenido. Ejemplo: StartTime="PT2.669S". |
| **OverallBitRate** |**xs: int** |Velocidad de bits media en Kbps del archivo de recursos. |

> [!NOTE]
> Los cuatro elementos secundarios siguientes deben aparecer en secuencia.  
> 
> 

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Programs**<br /><br /> minOccurs="0" | |Colección de todos los [elementos Programs](media-services-input-metadata-schema.md#Programs) cuando el archivo de recursos está en formato MPEG-TS. |
| **VideoTracks**<br /><br /> minOccurs="0" | |Cada archivo de recursos físico puede contener cero o más pistas de vídeos intercaladas en un formato de contenedor adecuado. Este elemento contiene una colección de todos los [VideoTracks](media-services-input-metadata-schema.md#VideoTracks) que forman parte del archivo de recursos. |
| **AudioTracks**<br /><br /> minOccurs="0" | |Cada archivo de recursos físico puede contener cero o más pistas de audio intercaladas en un formato de contenedor adecuado. Este elemento contiene una colección de todos los [AudioTracks](media-services-input-metadata-schema.md#AudioTracks) que forman parte del archivo de recursos. |
| **Metadata**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[MetadataType](media-services-input-metadata-schema.md#MetadataType) |Metadatos del archivo de recursos representados como cadenas de clave-valor. Por ejemplo:<br /><br /> **&lt;Metadata key="language" value="eng" /&gt;** |

## <a name="tracktype"></a><a name="TrackType"></a> TrackType
Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Id**<br /><br /> Obligatorio |**xs:int** |Índice de base cero de esta pista de audio o vídeo.<br /><br /> No es necesariamente un elemento TrackID como se usa en un archivo MP4. |
| **Codec** |**xs:string** |Cadena de códec de pista de vídeo. |
| **CodecLongName** |**xs: string** |Nombre largo del códec de pista de vídeo o audio. |
| **TimeBase**<br /><br /> Obligatorio |**xs:string** |Base de tiempo. Ejemplo: TimeBase="1/48000" |
| **NumberOfFrames** |**xs:int** |Número de fotogramas (presente para pistas de vídeo). |
| **StartTime** |**xs: duration** |Hora de inicio de la pista. Ejemplo: StartTime="PT2.669S" |
| **Duration** |**xs:duration** |Duración de la pista. Ejemplo: Duration="PTSampleFormat M37.757S". |

> [!NOTE]
> Los dos elementos secundarios siguientes deben aparecer en secuencia.  
> 
> 

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Disposition**<br /><br /> minOccurs="0" maxOccurs="1" |[StreamDispositionType](media-services-input-metadata-schema.md#StreamDispositionType) |Contiene información de presentación (por ejemplo, si una determinada pista de audio es para espectadores con discapacidad visual). |
| **Metadata**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[MetadataType](media-services-input-metadata-schema.md#MetadataType) |Cadenas genéricas de clave-valor que sirven para almacenar información de diversa índole. Por ejemplo, key=”language” y value=”eng”. |

## <a name="audiotracktype-inherits-from-tracktype"></a><a name="AudioTrackType"></a> AudioTrackType (hereda de TrackType)
 **AudioTrackType** es un tipo complejo global que hereda de [TrackType](media-services-input-metadata-schema.md#TrackType).  

 El tipo representa una determinada pista de audio en el archivo de recursos.  

 Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **SampleFormat** |**xs:string** |Formato de ejemplo. |
| **ChannelLayout** |**xs: string** |Distribución de canales. |
| **Channels**<br /><br /> Obligatorio |**xs:int** |Número de canales de audio (0 o más). |
| **SamplingRate**<br /><br /> Obligatorio |**xs:int** |Velocidad de muestreo de audio en muestras/s o Hz. |
| **Bitrate** |**xs:int** |Velocidad de bits de audio media en bits por segundo, según los cálculos en el archivo de recursos. Solo se cuenta la carga de transmisión básica; la sobrecarga de empaquetado no se incluye en este recuento. |
| **BitsPerSample** |**xs:int** |Bits por muestra para el tipo de formato wFormatTag. |

## <a name="videotracktype-inherits-from-tracktype"></a><a name="VideoTrackType"></a> VideoTrackType (hereda de TrackType)
**VideoTrackType** es un tipo complejo global que hereda de [TrackType](media-services-input-metadata-schema.md#TrackType).  

El tipo representa una determinada pista de vídeo en el archivo de recursos.  

Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **FourCC**<br /><br /> Obligatorio |**xs:string** |Código FourCC de códec de vídeo. |
| **Perfil** |**xs: string** |Perfil de la pista de vídeo. |
| **Level** |**xs: string** |Nivel de la pista de vídeo. |
| **PixelFormat** |**xs: string** |Formato de píxel de la pista de vídeo. |
| **Width**<br /><br /> Obligatorio |**xs:int** |Ancho del vídeo codificado en píxeles. |
| **Height**<br /><br /> Obligatorio |**xs:int** |Alto del vídeo codificado en píxeles. |
| **DisplayAspectRatioNumerator**<br /><br /> Obligatorio |**xs: double** |Numerador de la relación de aspecto de pantalla de vídeo. |
| **DisplayAspectRatioDenominator**<br /><br /> Obligatorio |**xs:double** |Denominador de la relación de aspecto de pantalla de vídeo. |
| **DisplayAspectRatioDenominator**<br /><br /> Obligatorio |**xs: double** |Numerador de la relación de aspecto de muestra de vídeo. |
| **SampleAspectRatioNumerator** |**xs: double** |Numerador de la relación de aspecto de muestra de vídeo. |
| **SampleAspectRatioNumerator** |**xs:double** |Denominador de la relación de aspecto de muestra de vídeo. |
| **FrameRate**<br /><br /> Obligatorio |**xs:decimal** |Velocidad de fotogramas de vídeo medida en formato .3f. |
| **Bitrate** |**xs:int** |Velocidad de bits de vídeo media en kilobits por segundo, según los cálculos en el archivo de recursos. Solo se cuenta la carga de transmisión básica; la sobrecarga de empaquetado no se incluye. |
| **MaxGOPBitrate** |**xs: int** |Velocidad de bits media máxima para GOP, en kilobits por segundo. |
| **HasBFrames** |**xs:int** |Número de pista de vídeo de fotogramas B. |

## <a name="metadatatype"></a><a name="MetadataType"></a> MetadataType
**MetadataType** es un tipo complejo global que describe los metadatos de un archivo de recursos como cadenas de clave-valor. Por ejemplo, key=”language” y value=”eng”.  

Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **key**<br /><br /> Obligatorio |**xs:string** |La clave del par clave-valor. |
| **value**<br /><br /> Obligatorio |**xs:string** |El valor del par clave-valor. |

## <a name="programtype"></a><a name="ProgramType"></a> ProgramType
**ProgramType** es un tipo complejo global que describe un programa.  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **ProgramId**<br /><br /> Obligatorio |**xs:int** |Id. del programa. |
| **NumberOfPrograms**<br /><br /> Obligatorio |**xs:int** |Número de programas. |
| **PmtPid**<br /><br /> Obligatorio |**xs:int** |Las tablas de mapa de programa (Pmt) contienen información acerca de los programas.  Para más información, consulte [PMt](https://en.wikipedia.org/wiki/MPEG_transport_stream#PMT). |
| **PcrPid**<br /><br /> Obligatorio |**xs: int** |Utilizado por el descodificador. Para más información, consulte [PCR](https://en.wikipedia.org/wiki/MPEG_transport_stream#PCR). |
| **StartPTS** |**xs: long** |Marca de tiempo de inicio de la presentación. |
| **EndPTS** |**xs: long** |Marca de tiempo de finalización de la presentación. |

## <a name="streamdispositiontype"></a><a name="StreamDispositionType"></a> StreamDispositionType
**StreamDispositionType** es un tipo complejo global que describe la transmisión.  

Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Valor predeterminado**<br /><br /> Obligatorio |**xs: int** |Establezca este atributo en 1 para indicar que se trata de la presentación predeterminada. |
| **Dub**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que se trata de la presentación doblada. |
| **Original**<br /><br /> Obligatorio |**xs: int** |Establezca este atributo en 1 para indicar que se trata de la presentación original. |
| **Comment**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que esta pista contiene el comentario. |
| **Lyrics**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que esta pista contiene letras. |
| **Karaoke**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que esto representa la pista de karaoke (música de fondo, sin voz). |
| **Forced**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que se trata de la presentación forzada. |
| **HearingImpaired**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que esta pista es para personas con discapacidad auditiva. |
| **VisualImpaired**<br /><br /> Obligatorio |**xs:int** |Establezca este atributo en 1 para indicar que esta pista es para personas con discapacidad visual. |
| **CleanEffects**<br /><br /> Obligatorio |**xs: int** |Establezca este atributo en 1 para indicar que esta pista contiene efectos limpios. |
| **AttachedPic**<br /><br /> Obligatorio |**xs: int** |Establezca este atributo en 1 para indicar que esta pista contiene imágenes. |

## <a name="programs-element"></a><a name="Programs"></a> Elemento Programs
Elemento contenedor con varios elementos **Program**.  

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Program**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[ProgramType](media-services-input-metadata-schema.md#ProgramType) |Para los archivos de recursos que están en formato MPEG-TS, contiene información acerca de los programas en el archivo de recursos. |

## <a name="videotracks-element"></a><a name="VideoTracks"></a> Elemento VideoTracks
 Elemento contenedor con varios elementos **VideoTrack**.  

 Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **VideoTrack**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[VideoTrackType (hereda de TrackType)](media-services-input-metadata-schema.md#VideoTrackType) |Contiene información sobre las pistas de vídeo en el archivo de recursos. |

## <a name="audiotracks-element"></a><a name="AudioTracks"></a> Elemento AudioTracks
 Elemento contenedor con varios elementos **AudioTrack**.  

 Consulte un ejemplo de XML al final de este artículo: [Ejemplo de XML](media-services-input-metadata-schema.md#xml).  

### <a name="elements"></a>Elementos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **AudioTrack**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[AudioTrackType (hereda de TrackType)](media-services-input-metadata-schema.md#AudioTrackType) |Contiene información sobre las pistas de audio en el archivo de recursos. |

## <a name="schema-code"></a><a name="code"></a> Código del esquema
```xml
<?xml version="1.0" encoding="utf-8"?>  
<xs:schema xmlns:xs="https://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata" version="1.0"  
            xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2014/07/mediaencoder/inputmetadata"  
            targetNamespace="http://schemas.microsoft.com/windowsazure/mediaservices/2014/07/mediaencoder/inputmetadata"  
            elementFormDefault="qualified">  

  <xs:complexType name="MetadataType">  
    <xs:attribute name="key"   type="xs:string" use="required"/>  
    <xs:attribute name="value" type="xs:string" use="required"/>  
  </xs:complexType>  

  <xs:complexType name="ProgramType">  
    <xs:attribute name="ProgramId" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>Program Id</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="NumberOfPrograms" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>Number of programs</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="PmtPid" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>pmt pid</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="PcrPid" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>pcr pid</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="StartPTS" type="xs:long">  
      <xs:annotation>  
        <xs:documentation>start pts</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="EndPTS" type="xs:long">  
      <xs:annotation>  
        <xs:documentation>end pts</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
  </xs:complexType>  

  <xs:complexType name="StreamDispositionType">  
    <xs:attribute name="Default"          type="xs:int" use="required" />  
    <xs:attribute name="Dub"              type="xs:int" use="required" />  
    <xs:attribute name="Original"         type="xs:int" use="required" />  
    <xs:attribute name="Comment"          type="xs:int" use="required" />  
    <xs:attribute name="Lyrics"           type="xs:int" use="required" />  
    <xs:attribute name="Karaoke"          type="xs:int" use="required" />  
    <xs:attribute name="Forced"           type="xs:int" use="required" />  
    <xs:attribute name="HearingImpaired"  type="xs:int" use="required" />  
    <xs:attribute name="VisualImpaired"   type="xs:int" use="required" />  
    <xs:attribute name="CleanEffects"     type="xs:int" use="required" />  
    <xs:attribute name="AttachedPic"      type="xs:int" use="required" />  
  </xs:complexType>  

  <xs:complexType name="TrackType" abstract="true">  
    <xs:sequence>  
      <xs:element name="Disposition" type="StreamDispositionType" minOccurs="0" maxOccurs="1"/>  
      <xs:element name="Metadata" type="MetadataType" minOccurs="0" maxOccurs="unbounded"/>  
    </xs:sequence>  
    <xs:attribute name="Id" use="required">  
      <xs:annotation>  
        <xs:documentation>zero-based index of this video track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
      </xs:annotation>  
      <xs:simpleType>  
        <xs:restriction base="xs:int">  
          <xs:minInclusive value="0"/>  
        </xs:restriction>  
      </xs:simpleType>  
    </xs:attribute>  
    <xs:attribute name="Codec" type="xs:string">  
      <xs:annotation>  
        <xs:documentation>video track codec string</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="CodecLongName" type="xs:string">  
      <xs:annotation>  
        <xs:documentation>video track codec long name</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="TimeBase"  type="xs:string" use="required">  
      <xs:annotation>  
        <xs:documentation>Time base. Example: TimeBase="1/48000"</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="NumberOfFrames">  
      <xs:annotation>  
        <xs:documentation>number of frames</xs:documentation>  
      </xs:annotation>  
      <xs:simpleType>  
        <xs:restriction base="xs:int">  
          <xs:minInclusive value="0"/>  
        </xs:restriction>  
      </xs:simpleType>  
    </xs:attribute>  
    <xs:attribute name="StartTime" type="xs:duration">  
      <xs:annotation>  
        <xs:documentation>Track start time. Example: StartTime="PT2.669S"</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="Duration" type="xs:duration">  
      <xs:annotation>  
        <xs:documentation>Track duration. Example: Duration="PT25M37.757S"</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
  </xs:complexType>  

  <xs:complexType name="VideoTrackType">  
    <xs:annotation>  
      <xs:documentation>A specific video track in the parent AssetFile</xs:documentation>  
    </xs:annotation>  
    <xs:complexContent>  
      <xs:extension base="TrackType">  
        <xs:attribute name="FourCC" type="xs:string" use="required">  
          <xs:annotation>  
            <xs:documentation>video codec FourCC code</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Profile" type="xs:string">  
          <xs:annotation>  
            <xs:documentation>profile</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Level" type="xs:string">  
          <xs:annotation>  
            <xs:documentation>level</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="PixelFormat" type="xs:string">  
          <xs:annotation>  
            <xs:documentation>Video track's pixel format</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Width" use="required">  
          <xs:annotation>  
            <xs:documentation>encoded video width in pixels</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="Height" use="required">  
          <xs:annotation>  
            <xs:documentation>encoded video height in pixels</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="DisplayAspectRatioNumerator" use="required">  
          <xs:annotation>  
            <xs:documentation>video display aspect ratio numerator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="DisplayAspectRatioDenominator" use="required">  
          <xs:annotation>  
            <xs:documentation>video display aspect ratio denominator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="SampleAspectRatioNumerator">  
          <xs:annotation>  
            <xs:documentation>video sample aspect ratio numerator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="SampleAspectRatioDenominator">  
          <xs:annotation>  
            <xs:documentation>video sample aspect ratio denominator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="FrameRate" use="required">  
          <xs:annotation>  
            <xs:documentation>measured video frame rate in .3f format</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:decimal">  
              <xs:minInclusive value="0"/>  
              <xs:fractionDigits value="3"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="Bitrate">  
          <xs:annotation>  
            <xs:documentation>average video bit rate in kilobits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="MaxGOPBitrate">  
          <xs:annotation>  
            <xs:documentation>Max GOP average bitrate for this video track, in kilobits per second</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="HasBFrames" type="xs:int">  
          <xs:annotation>  
            <xs:documentation>video track number of B frames</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
      </xs:extension>  
    </xs:complexContent>  
  </xs:complexType>  

  <xs:complexType name="AudioTrackType">  
    <xs:annotation>  
      <xs:documentation>a specific audio track in the parent AssetFile</xs:documentation>  
    </xs:annotation>  
    <xs:complexContent>  
      <xs:extension base="TrackType">  
        <xs:attribute name="SampleFormat"  type="xs:string">  
          <xs:annotation>  
            <xs:documentation>sample format</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="ChannelLayout"  type="xs:string">  
          <xs:annotation>  
            <xs:documentation>channel layout</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Channels" use="required">  
          <xs:annotation>  
            <xs:documentation>number of audio channels</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="SamplingRate" use="required">  
          <xs:annotation>  
            <xs:documentation>audio sampling rate in samples/sec or Hz</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="Bitrate">  
          <xs:annotation>  
            <xs:documentation>average audio bit rate in bits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="BitsPerSample">  
          <xs:annotation>  
            <xs:documentation>Bits per sample for the wFormatTag format type</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
      </xs:extension>  
    </xs:complexContent>  
  </xs:complexType>  

  <xs:element name="AssetFiles">  
    <xs:annotation>  
      <xs:documentation>Collection of AssetFile entries for the encoding job</xs:documentation>  
    </xs:annotation>  
    <xs:complexType>  
      <xs:sequence>  
        <xs:element name="AssetFile" minOccurs="1" maxOccurs="unbounded">  
          <xs:annotation>  
            <xs:documentation>asset file</xs:documentation>  
          </xs:annotation>  
          <xs:complexType>  
            <xs:sequence>  
              <xs:element name="Programs" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>This is the collection of all programs when file is MPEG-TS</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="Program" type="ProgramType" minOccurs="0" maxOccurs="unbounded" />  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="VideoTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>Each physical AssetFile can contain in it zero or more video tracks interleaved into an appropriate container format. This is the collection of all those video tracks</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="VideoTrack" type="VideoTrackType" minOccurs="0" maxOccurs="unbounded" />  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="AudioTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>each physical AssetFile can contain in it zero or more audio tracks interleaved into an appropriate container format. This is the collection of all those audio tracks</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="AudioTrack" type="AudioTrackType" minOccurs="0" maxOccurs="unbounded" />  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="Metadata" type="MetadataType" minOccurs="0" maxOccurs="unbounded" />  
            </xs:sequence>  
            <xs:attribute name="Name" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>the media asset file name</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="Size" use="required">  
              <xs:annotation>  
                <xs:documentation>size of file in bytes</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:long">  
                  <xs:minInclusive value="0"/>  
                </xs:restriction>  
              </xs:simpleType>  
            </xs:attribute>  
            <xs:attribute name="Duration" type="xs:duration" use="required">  
              <xs:annotation>  
                <xs:documentation>content play back duration. Example: Duration="PT25M37.757S"</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="NumberOfStreams" type="xs:int" use="required">  
              <xs:annotation>  
                <xs:documentation>number of streams in asset file</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="FormatNames" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>format names</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="FormatVerboseName" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>format verbose names</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="StartTime" type="xs:duration">  
              <xs:annotation>  
                <xs:documentation>content start time. Example: StartTime="PT2.669S"</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="OverallBitRate">  
              <xs:annotation>  
                <xs:documentation>average bitrate of the asset file in kbps</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:int">  
                  <xs:minInclusive value="0"/>  
                </xs:restriction>  
              </xs:simpleType>  
            </xs:attribute>  
          </xs:complexType>  
        </xs:element>  
      </xs:sequence>  
    </xs:complexType>  
  </xs:element>  
</xs:schema>  
```


## <a name="xml-example"></a><a name="xml"></a> Ejemplo de XML
A continuación, sigue un ejemplo del archivo de metadatos de entrada.  

```xml
<?xml version="1.0" encoding="utf-8"?>  
<AssetFiles xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2014/07/mediaencoder/inputmetadata">  
  <AssetFile Name="bear.mp4" Size="1973733" Duration="PT12.678S" NumberOfStreams="2" FormatNames="mov,mp4,m4a,3gp,3g2,mj2" FormatVerboseName="QuickTime / MOV" StartTime="PT0S" OverallBitRate="1245">  
    <VideoTracks>  
      <VideoTrack Id="1" Codec="h264" CodecLongName="H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10" TimeBase="1/29970" NumberOfFrames="375" StartTime="PT0.034S" Duration="PT12.645S" FourCC="avc1" Profile="High" Level="4.1" PixelFormat="yuv420p" Width="512" Height="384" DisplayAspectRatioNumerator="4" DisplayAspectRatioDenominator="3" SampleAspectRatioNumerator="1" SampleAspectRatioDenominator="1" FrameRate="29.656" Bitrate="1043" HasBFrames="1">  
        <Disposition Default="1" Dub="0" Original="0" Comment="0" Lyrics="0" Karaoke="0" Forced="0" HearingImpaired="0" VisualImpaired="0" CleanEffects="0" AttachedPic="0" />  
        <Metadata key="creation_time" value="2010-03-10 16:11:56" />  
        <Metadata key="language" value="eng" />  
        <Metadata key="handler_name" value="Mainconcept MP4 Video Media Handler" />  
      </VideoTrack>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="aac" CodecLongName="AAC (Advanced Audio Coding)" TimeBase="1/44100" NumberOfFrames="546" StartTime="PT0S" Duration="PT12.678S" SampleFormat="fltp" ChannelLayout="stereo" Channels="2" SamplingRate="44100" Bitrate="156" BitsPerSample="0">  
        <Disposition Default="1" Dub="0" Original="0" Comment="0" Lyrics="0" Karaoke="0" Forced="0" HearingImpaired="0" VisualImpaired="0" CleanEffects="0" AttachedPic="0" />  
        <Metadata key="creation_time" value="2010-03-10 16:11:56" />  
        <Metadata key="language" value="eng" />  
        <Metadata key="handler_name" value="Mainconcept MP4 Sound Media Handler" />  
      </AudioTrack>  
    </AudioTracks>  
    <Metadata key="major_brand" value="mp42" />  
    <Metadata key="minor_version" value="0" />  
    <Metadata key="compatible_brands" value="mp42mp41" />  
    <Metadata key="creation_time" value="2010-03-10 16:11:53" />  
    <Metadata key="comment" value="Courtesy of National Geographic.  Used by Permission." />  
  </AssetFile>  
</AssetFiles>  
```

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

