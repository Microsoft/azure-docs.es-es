---
title: Esquema de metadatos de salida de Azure Media Services | Microsoft Docs
description: En este artículo se proporciona información general sobre el esquema de metadatos de salida de Azure Media Services.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1ed84c88-eea5-4a24-9c4f-f2428157d08a
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: cd81ef78ecc5ef9cea71adb387597681460d50c8
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89261332"
---
# <a name="output-metadata"></a>Metadatos de salida

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general
Un trabajo de codificación está asociado a un recurso (o recursos) de entrada donde desea realizar algunas tareas de codificación. Por ejemplo, codificar un archivo MP4 en conjuntos MP4 de velocidad de bits adaptable H.264; crear una miniatura; crear superposiciones. Tras la finalización de una tarea, se produce un recurso de salida.  El recurso de salida contiene vídeo, audio, miniaturas, etc. El recurso de salida también contiene un archivo con metadatos sobre el recurso de salida. El nombre del archivo XML de metadatos tiene el formato siguiente: &lt;nombre_de_archivo_de_origen&gt;_manifest.xml (por ejemplo, BigBuckBunny_manifest.xml).  

Media Services no realiza un examen preventivo de los recursos de entrada para generar metadatos. Los metadatos de entrada se generan únicamente como artefactos cuando un recurso de entrada se procesa en un trabajo. Por tanto, estos artefactos se escriben en el recurso de salida. Para generar metadatos de los recursos de entrada y salida, se utilizan diferentes herramientas. Por tanto, los metadatos de entrada tienen un esquema ligeramente diferente al de los metadatos de salida.

Si desea examinar el archivo de metadatos, puede crear un localizador **SAS** y descargar el archivo en el equipo local.  

En este artículo se describen los elementos y los tipos del esquema XML en que se basan los metadatos de salida (&lt;nombre_de_archivo_de_origen&gt;_manifest.xml). Para información acerca del archivo que contiene los metadatos sobre el recurso de entrada, consulte Metadatos de entrada.  

Puede encontrar el código del esquema completo y un ejemplo de XML al final de este artículo.  

## <a name="assetfiles-root-element"></a><a name="AssetFiles"></a> Elemento raíz AssetFiles
Colección de entradas AssetFile para el trabajo de codificación.  

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Descripción |
| --- | --- |
| **AssetFile**<br/><br/> minOccurs="0" maxOccurs="1" |Un elemento AssetFile que forma parte de la colección de AssetFiles. |

## <a name="assetfile-element"></a><a name="AssetFile"></a> Elemento AssetFile
Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Nombre**<br/><br/> Obligatorio |**xs:string** |Nombre del archivo de recursos multimedia. |
| **Tamaño**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:long** |Tamaño del archivo de recursos en bytes. |
| **Duration**<br/><br/> Obligatorio |**xs:duration** |Duración de la reproducción del contenido. |

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Descripción |
| --- | --- |
| **Sources** |Colección de archivos multimedia de entrada/origen, que se procesa para producir este AssetFile. Para más información, consulte el elemento Source. |
| **VideoTracks**<br/><br/> minOccurs="0" maxOccurs="1" |Cada AssetFile físico puede contener cero o más pistas de vídeo intercaladas en un formato de contenedor adecuado. Para más información, consulte el elemento VideoTracks. |
| **AudioTracks**<br/><br/> minOccurs="0" maxOccurs="1" |Cada AssetFile físico puede contener cero o más pistas de audio intercaladas en un formato de contenedor adecuado. Se trata de la colección de todas esas pistas de audio. Para más información, consulte el elemento AudioTracks. |

## <a name="sources-element"></a><a name="Sources"></a> Elemento Sources
Colección de archivos multimedia de entrada/origen, que se procesa para producir este AssetFile.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Descripción |
| --- | --- |
| **Origen**<br/><br/> minOccurs="1" maxOccurs="unbounded" |Un archivo de entrada/origen que se usa al generar este recurso. Para más información, consulte el elemento Source. |

## <a name="source-element"></a><a name="Source"></a> Elemento Source
Un archivo de entrada/origen que se usa al generar este recurso.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Nombre**<br/><br/> Obligatorio |**xs:string** |Nombre de archivo de origen de entrada. |

## <a name="videotracks-element"></a><a name="VideoTracks"></a> Elemento VideoTracks
Cada AssetFile físico puede contener cero o más pistas de vídeo intercaladas en un formato de contenedor adecuado. El elemento **VideoTracks** representa una colección de todas las pistas de vídeo.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Descripción |
| --- | --- |
| **VideoTrack**<br/><br/> minOccurs="1" maxOccurs="unbounded" |Una determinada pista de vídeo en el AssetFile primario. Para más información, consulte el elemento VideoTrack. |

## <a name="videotrack-element"></a><a name="VideoTrack"></a> Elemento VideoTrack
Una determinada pista de vídeo en el AssetFile primario.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Id**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Índice de base cero de esta pista de vídeo. **Nota:**  este **identificador** no es necesariamente el objeto TrackID que se usa en un archivo MP4. |
| **FourCC**<br/><br/> Obligatorio |**xs:string** |Código FourCC de códec de vídeo. |
| **Perfil** |**xs:string** |Perfil de H264 (solo es aplicable al códec H264). |
| **Level** |**xs:string** |Nivel de H264 (solo es aplicable al códec H264). |
| **Width**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Ancho del vídeo codificado en píxeles. |
| **Height**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Alto del vídeo codificado en píxeles. |
| **DisplayAspectRatioNumerator**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:double** |Numerador de la relación de aspecto de pantalla de vídeo. |
| **DisplayAspectRatioDenominator**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:double** |Denominador de la relación de aspecto de pantalla de vídeo. |
| **Framerate**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:decimal** |Velocidad de fotogramas de vídeo medida en formato .3f. |
| **TargetFramerate**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:decimal** |Velocidad de fotogramas de vídeo de destino preestablecida en formato .3f. |
| **Bitrate**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Velocidad de bits de vídeo media en kilobits por segundo, según los cálculos de AssetFile. Solo cuenta la carga de transmisión básica y no incluye la sobrecarga de empaquetado. |
| **TargetBitrate**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Velocidad de bits media de destino para esta pista de vídeo, tal como se ha solicitado mediante la codificación preestablecida, en kilobits por segundo. |
| **MaxGOPBitrate**<br/><br/> minInclusive ="0" |**xs:int** |Velocidad de bits media máxima para GOP, en kilobits por segundo. |

## <a name="audiotracks-element"></a><a name="AudioTracks"></a> Elemento AudioTracks
Cada AssetFile físico puede contener cero o más pistas de audio intercaladas en un formato de contenedor adecuado. El elemento **AudioTracks** representa una colección de todas las pistas de audio.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Descripción |
| --- | --- |
| **AudioTrack**<br/><br/> minOccurs="1" maxOccurs="unbounded" |Una determinada pista de audio en el AssetFile primario. Para más información, consulte el elemento AudioTrack. |

## <a name="audiotrack-element"></a><a name="AudioTrack"></a> Elemento AudioTrack
Una determinada pista de audio en el AssetFile primario.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **Id**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Índice de base cero de esta pista de audio. **Nota:**  no es necesariamente el objeto TrackID que se usa en un archivo MP4. |
| **Codec** |**xs:string** |Cadena de códec de pista de audio. |
| **EncoderVersion** |**xs:string** |Cadena de versión de codificador opcional, requerida para EAC3. |
| **Channels**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Número de canales de audio. |
| **SamplingRate**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Velocidad de muestreo de audio en muestras/s o Hz. |
| **Bitrate**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Velocidad de bits de audio media en bits por segundo, según los cálculos de AssetFile. Solo cuenta la carga de transmisión básica y no incluye la sobrecarga de empaquetado. |
| **BitsPerSample**<br/><br/> minInclusive ="0"<br/><br/> Obligatorio |**xs:int** |Bits por muestra para el tipo de formato wFormatTag. |

### <a name="child-elements"></a>Elementos secundarios
| Nombre | Descripción |
| --- | --- |
| **LoudnessMeteringResultParameters**<br/><br/> minOccurs="0" maxOccurs="1" |Parámetros de resultado de medición de la sonoridad. Para más información, consulte el elemento LoudnessMeteringResultParameters. |

## <a name="loudnessmeteringresultparameters-element"></a><a name="LoudnessMeteringResultParameters"></a> Elemento LoudnessMeteringResultParameters
Parámetros de resultado de medición de la sonoridad.  

Puede encontrar un ejemplo de XML en [Ejemplo de XML](#xml).  

### <a name="attributes"></a>Atributos
| Nombre | Tipo | Descripción |
| --- | --- | --- |
| **DPLMVersionInformation** |**xs:string** |Versión del kit de desarrollo de medición de sonoridad profesional **Dolby**. |
| **DialogNormalization**<br/><br/> minInclusive="-31" maxInclusive="-1"<br/><br/> Obligatorio |**xs:int** |DialogNormalization generado mediante DPLM, requerido si LoudnessMetering está establecido. |
| **IntegratedLoudness**<br/><br/> minInclusive="-70" maxInclusive="10"<br/><br/> Obligatorio |**xs:float** |Sonoridad integrada. |
| **IntegratedLoudnessUnit**<br/><br/> Obligatorio |**xs:string** |Unidad de sonoridad integrada. |
| **IntegratedLoudnessGatingMethod**<br/><br/> Obligatorio |**xs:string** |Identificador de acceso. |
| **IntegratedLoudnessSpeechPercentage**<br/><br/> minInclusive ="0" maxInclusive="100" |**xs:float** |Contenido de voz sobre el programa, en forma de porcentaje. |
| **SamplePeak**<br/><br/> Obligatorio |**xs:float** |Valor pico de ejemplo absoluto, desde el restablecimiento o desde la última vez que se desactivó, por canal.  Las unidades son dBFS. |
| **SamplePeakUnit**<br/><br/> fixed="dBFS"<br/><br/> Obligatorio |**xs:anySimpleType** |Unidad pico de ejemplo. |
| **TruePeak**<br/><br/> Obligatorio |**xs:float** |Valor pico verdadero máximo, según ITU-R BS.1770-2, desde el restablecimiento o desde la última vez que se desactivó, por canal. Las unidades son dBTP. |
| **TruePeakUnit**<br/><br/> fixed="dBTP"<br/><br/> Obligatorio |**xs:anySimpleType** |Unidad pico verdadera. |

## <a name="schema-code"></a>Código del esquema
```xml
<?xml version="1.0" encoding="utf-8"?>  
<xs:schema xmlns:xs="https://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata" version="1.2"  
            xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
            targetNamespace="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
            elementFormDefault="qualified">  
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
              <xs:element name="Sources">  
                <xs:annotation>  
                  <xs:documentation>Collection of input/source media files, that was processed in order to produce this AssetFile</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="Source" minOccurs="1" maxOccurs="unbounded">  
                      <xs:annotation>  
                        <xs:documentation>An input/source file used when generating this asset</xs:documentation>  
                      </xs:annotation>  
                      <xs:complexType>  
                        <xs:attribute name="Name" type="xs:string" use="required">  
                          <xs:annotation>  
                            <xs:documentation>input source file name</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                      </xs:complexType>  
                    </xs:element>  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="VideoTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>Each physical AssetFile can contain in it zero or more video tracks interleaved into an appropriate container format. This is the collection of all those video tracks</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="VideoTrack" maxOccurs="unbounded">  
                      <xs:annotation>  
                        <xs:documentation>A specific video track in the parent AssetFile</xs:documentation>  
                      </xs:annotation>  
                      <xs:complexType>  
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
                        <xs:attribute name="FourCC" type="xs:string" use="required">  
                          <xs:annotation>  
                            <xs:documentation>video codec FourCC code</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="Profile" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>H264 profile (only appliable for H264 codec)</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="Level" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>H264 level (only appliable for H264 codec)</xs:documentation>  
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
                        <xs:attribute name="Framerate" use="required">  
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
                        <xs:attribute name="TargetFramerate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>preset target video frame rate in .3f format</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:decimal">  
                              <xs:minInclusive value="0"/>  
                              <xs:fractionDigits value="3"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Bitrate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>average video bit rate in kilobits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="TargetBitrate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>target average bitrate for this video track, as requested via the encoding preset, in kilobits per second</xs:documentation>  
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
                      </xs:complexType>  
                    </xs:element>  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="AudioTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>each physical AssetFile can contain in it zero or more audio tracks interleaved into an appropriate container format. This is the collection of all those audio tracks</xs:documentation>  
                </xs:annotation>  
                 <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="AudioTrack" maxOccurs="unbounded">  
                      <xs:annotation>  
                        <xs:documentation>a specific audio track in the parent AssetFile</xs:documentation>  
                      </xs:annotation>  
                      <xs:complexType>  
                        <xs:sequence>  
                          <xs:element name="LoudnessMeteringResultParameters" minOccurs="0" maxOccurs="1">  
                            <xs:annotation>  
                              <xs:documentation>Loudness Metering Result Parameters</xs:documentation>  
                            </xs:annotation>  
                            <xs:complexType>  
                              <xs:attribute name="DPLMVersionInformation" type="xs:string">  
                                <xs:annotation>  
                                  <xs:documentation>Dolby Professional Loudness Metering Development Kit Version</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="DialogNormalization" use="required">  
                                <xs:annotation>  
                                  <xs:documentation> DialogNormalization generated through DPLM, required when LoudnessMetering is set</xs:documentation>  
                                </xs:annotation>  
                                <xs:simpleType>  
                                  <xs:restriction base="xs:int">  
                                    <xs:minInclusive value="-31"/>  
                                    <xs:maxInclusive value="-1"/>  
                                  </xs:restriction>  
                                </xs:simpleType>  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudness" use="required">  
                                <xs:annotation>  
                                  <xs:documentation>Integrated loudness</xs:documentation>  
                                </xs:annotation>  
                                <xs:simpleType>  
                                  <xs:restriction base="xs:float">  
                                    <xs:minInclusive value="-70"/>  
                                    <xs:maxInclusive value="10"/>  
                                  </xs:restriction>  
                                </xs:simpleType>  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudnessUnit" use="required" type="xs:string">  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudnessGatingMethod" use="required" type="xs:string">  
                                <xs:annotation>  
                                  <xs:documentation>Gating identifier</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudnessSpeechPercentage">  
                                <xs:annotation>  
                                  <xs:documentation>Speech content over the program, as a percentage.</xs:documentation>  
                                </xs:annotation>  
                                <xs:simpleType>  
                                  <xs:restriction base="xs:float">  
                                    <xs:minInclusive value="0"/>  
                                    <xs:maxInclusive value="100"/>  
                                  </xs:restriction>  
                                </xs:simpleType>  
                              </xs:attribute>  
                              <xs:attribute name="SamplePeak" use="required" type="xs:float">  
                                <xs:annotation>  
                                  <xs:documentation>Peak absolute sample value, since reset or since it was last cleared, per channel.  Units are dBFS.</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="SamplePeakUnit" use="required" fixed="dBFS">  
                              </xs:attribute>  
                              <xs:attribute name="TruePeak" use="required" type="xs:float">  
                                <xs:annotation>  
                                  <xs:documentation>Maximum True Peak value, as per ITU-R BS.1770-2, since reset or since it was last cleared, per channel.  Units are dBTP.</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="TruePeakUnit" use="required" fixed="dBTP">  
                              </xs:attribute>  
                            </xs:complexType>  
                          </xs:element>  
                        </xs:sequence>  
                        <xs:attribute name="Id" use="required">  
                          <xs:annotation>  
                            <xs:documentation>zero-based index of this audio track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Codec" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>audio track codec string</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="EncoderVersion" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>optional encoder version string, required for EAC3</xs:documentation>  
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
                        <xs:attribute name="Bitrate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>average audio bit rate in bits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="BitsPerSample" use="required">  
                          <xs:annotation>  
                            <xs:documentation>Bits per sample for the wFormatTag format type</xs:documentation>  
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
            <xs:attribute name="Duration" use="required">  
              <xs:annotation>  
                <xs:documentation>content play back duration</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:duration"/>  
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

El siguiente código XML es un ejemplo del archivo de metadatos de salida.  

```xml
<AssetFiles xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema"   
            xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata">  
  <AssetFile Name="BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4" Size="4646283" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.2" Width="1280" Height="720" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="4250" TargetBitrate="3400" MaxGOPBitrate="5514"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4" Size="3166728" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
      <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="2846" TargetBitrate="2250" MaxGOPBitrate="3630"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4" Size="2205095" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1932" TargetBitrate="1500" MaxGOPBitrate="2513"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4" Size="1508567" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1271" TargetBitrate="1000" MaxGOPBitrate="1527"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4" Size="1057155" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="843" TargetBitrate="650" MaxGOPBitrate="1086"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4" Size="699262" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="1.3" Width="320" Height="180" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="503" TargetBitrate="400" MaxGOPBitrate="661"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_AAC_und_ch2_96kbps.mp4" Size="166780" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_AAC_und_ch2_56kbps.mp4" Size="124576" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="53" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
</AssetFiles>  
```

## <a name="next-steps"></a>Pasos siguientes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
