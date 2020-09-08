---
title: 'Varios archivos de entrada y propiedades de los componentes con Encoder Premium: Azure | Microsoft Docs'
description: En este tema se explica cómo usar setRuntimeProperties para usar varios archivos de entrada y pasar datos personalizados al procesador de multimedia del flujo de trabajo del Codificador multimedia Premium.
services: media-services
documentationcenter: ''
author: xpouyat
manager: femila
editor: ''
ms.assetid: 7fb35bdd-9891-4401-a65b-ef3cc8190e8a
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: xpouyat
ms.reviewer: anilmur;juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 36a1a198f4bd2279a9d9b8b9700b1ddd038a0900
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89256674"
---
# <a name="using-multiple-input-files-and-component-properties-with-premium-encoder"></a>Uso del codificador Premium con varios archivos de entrada y propiedades de los componentes

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general
Hay escenarios en los que podría ser necesario personalizar las propiedades de los componentes, especificar el contenido XML de lista de clips o enviar varios archivos de entrada al enviar una tarea con el procesador de multimedia del **flujo de trabajo de Codificador multimedia Premium** . Ejemplos:

* Superposición de texto en vídeo y definición del valor de texto (por ejemplo, la fecha actual) en tiempo de ejecución para cada vídeo de entrada
* Personalización del XML de la lista de clips (para especificar uno o varios archivos de código fuente, con o sin recorte, etc.)
* Superposición de una imagen de logotipo en la entrada de vídeo durante la codificación
* Codificación de varios idiomas de audio.

Para permitir que el **flujo de trabajo de Codificador multimedia Premium** sepa que va a cambiar algunas propiedades del flujo de trabajo al crear la tarea o enviar varios archivos de entrada, tiene que usar una cadena de configuración que contenga **setRuntimeProperties** o **transcodeSource**. En este tema se explica cómo usarlos.

## <a name="configuration-string-syntax"></a>Sintaxis de la cadena de configuración
La cadena de configuración para establecer en la tarea de codificación usa un documento XML con el siguiente aspecto:

```xml
<?xml version="1.0" encoding="utf-8"?>
<transcodeRequest>
  <transcodeSource>
  </transcodeSource>
  <setRuntimeProperties>
    <property propertyPath="Media File Input/filename" value="VideoFileName.mp4" />
  </setRuntimeProperties>
</transcodeRequest>
```

El código de C# que aparece a continuación lee la configuración XML de un archivo, la actualiza con el nombre de archivo del vídeo correspondiente y pasa la información a la tarea de un trabajo:

```csharp
string premiumConfiguration = ReadAllText(@"D:\home\site\wwwroot\Presets\SetRuntime.xml").Replace("VideoFileName", myVideoFileName);

// Declare a new job.
IJob job = _context.Jobs.Create("Premium Workflow encoding job");

// Get a media processor reference, and pass to it the name of the 
// processor to use for the specific task.
IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Premium Workflow");

// Create a task with the encoding details, using a string preset.
ITask task = job.Tasks.AddNew("Premium Workflow encoding task",
                              processor,
                              premiumConfiguration,
                              TaskOptions.None);

// Specify the input assets
task.InputAssets.Add(workflow); // workflow asset
task.InputAssets.Add(video); // video asset with multiple files

// Add an output asset to contain the results of the job. 
// This output is specified as AssetCreationOptions.None, which 
// means the output asset is not encrypted. 
task.OutputAssets.AddNew("Output asset", AssetCreationOptions.None);
```

## <a name="customizing-component-properties"></a>Personalización de las propiedades de los componentes
### <a name="property-with-a-simple-value"></a>Propiedad con un valor simple
En algunos casos resulta útil personalizar una propiedad de componente junto con el archivo de flujo de trabajo que va a ejecutar el flujo de trabajo de Codificador multimedia Premium.

Supongamos que ha diseñado un flujo de trabajo que superpone texto en los vídeos y este (por ejemplo, la fecha actual) debe establecerse en tiempo de ejecución. Puede hacerlo si envía el texto como valor nuevo para la propiedad de texto del componente de superposición de la tarea de codificación. Puede utilizar este mecanismo para cambiar otras propiedades de un componente en el flujo de trabajo (por ejemplo, la posición o color de la superposición, la velocidad de bits del codificador AVC, etc.).

**setRuntimeProperties** se usa para reemplazar una propiedad de los componentes del flujo de trabajo.

Ejemplo:

```xml
<?xml version="1.0" encoding="utf-8"?>
  <transcodeRequest>
    <setRuntimeProperties>
      <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
      <property propertyPath="/primarySourceFile" value="MyInputVideo.mp4" />
      <property propertyPath="Optional Text Overlay/Text To Image Converter/text" value="Today is Friday the 13th of May, 2016"/>
  </setRuntimeProperties>
</transcodeRequest>
```

### <a name="property-with-an-xml-value"></a>Propiedad con un valor XML
Para establecer una propiedad que espere un valor XML, enciérrela con `<![CDATA[ and ]]>`.

Ejemplo:

```xml
<?xml version="1.0" encoding="utf-8"?>
  <transcodeRequest>
    <setRuntimeProperties>
      <property propertyPath="/primarySourceFile" value="start.mxf" />
      <property propertyPath="/inactiveTimeout" value="65" />
      <property propertyPath="clipListXml" value="xxx">
      <extendedValue><![CDATA[<clipList>
        <clip>
          <videoSource>
            <mediaFile>
              <file>start.mxf</file>
            </mediaFile>
          </videoSource>
          <audioSource>
            <mediaFile>
              <file>start.mxf</file>
            </mediaFile>
          </audioSource>
        </clip>
        <primaryClipIndex>0</primaryClipIndex>
        </clipList>]]>
      </extendedValue>
      </property>
      <property propertyPath="Media File Input Logo/filename" value="logo.png" />
    </setRuntimeProperties>
  </transcodeRequest>
```

> [!NOTE]
> Asegúrese de no colocar un retorno de carro justo después de `<![CDATA[`.

### <a name="propertypath-value"></a>Valor de propertyPath
En los ejemplos anteriores, el valor de propertyPath era "/Media File Input/filename", "/inactiveTimeout" o "clipListXml".
En general, es el nombre del componente seguido del nombre de la propiedad. La ruta de acceso puede tener más o menos niveles, como /primarySourceFile (si la propiedad se encuentra en la raíz del flujo de trabajo) o /Video Processing/Graphic Overlay/Opacity (si la superposición se encuentra en un grupo).    

Para comprobar la ruta de acceso y el nombre de propiedad, utilice el botón de acción justo al lado de cada propiedad. Puede hacer clic en este botón de acción y seleccione **Editar**. Esto le mostrará el nombre de la propiedad y, justo encima, el espacio de nombres.

![Acción o edición](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture6_actionedit.png)

![Propiedad](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture7_viewproperty.png)

## <a name="multiple-input-files"></a>Varios archivos de entrada
Cada tarea que se envía al **flujo de trabajo de Codificador multimedia Premium** requiere dos recursos:

* El primero es un *recurso de flujo de trabajo* que contiene un archivo de flujo de trabajo. Los archivos de flujo de trabajo pueden crearse con [Diseñador de flujo de trabajo](media-services-workflow-designer.md).
* El segundo es un *recurso multimedia* que contiene los archivos multimedia para codificar.

Al enviar varios archivos multimedia al codificador del **flujo de trabajo de Codificador multimedia Premium** se aplican las restricciones siguientes:

* Todos los archivos multimedia deben encontrarse en el mismo *recurso multimedia*. No se admite el uso de varios recursos multimedia.
* En el recurso multimedia debe definir el archivo principal (idealmente, el archivo de vídeo principal que se pide procesar al codificador).
* Es necesario pasar al procesador los datos de configuración que incluyen el elemento **setRuntimeProperties** o **transcodeSource**.
  * **setRuntimeProperties** se usa para reemplazar el nombre de archivo u otra propiedad de los componentes del flujo de trabajo.
  * **transcodeSource** sirve para especificar el contenido XML de la lista de clips.

Conexiones en el flujo de trabajo:

* Si utiliza uno o varios componentes de entrada de archivos multimedia y planea usar **setRuntimeProperties** para especificar el nombre de archivo, no conecte a estos la clavija del componente del archivo principal. Asegúrese de que no hay ninguna conexión entre el objeto del archivo principal y las entradas del archivo multimedia.
* Si prefiere usar XML de lista de clips y un componente de origen multimedia, puede conectarlos entre sí.

![Sin conexión desde el archivo de origen principal para a la entrada del archivo multimedia](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture0_nopin.png)

*Si utiliza setRuntimeProperties para establecer el nombre de archivo, el archivo principal no se conecta a los componentes de entrada del archivo multimedia.*

![Conexión de la opción de XML de la lista de clips con el origen de lista de clips](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture1_pincliplist.png)

*Puede conectar el XML de la lista de clips al origen multimedia y usar transcodeSource*

### <a name="clip-list-xml-customization"></a>Personalización del XML de lista de clips
Puede especificar el XML de la lista de clips en el flujo de trabajo en tiempo de ejecución con **transcodeSource** en la configuración de la cadena XML. Esto requiere la conexión de la clavija de XML de la lista de clips al componente de origen multimedia en el flujo de trabajo.

```xml
<?xml version="1.0" encoding="utf-16"?>
  <transcodeRequest>
    <transcodeSource>
      <clipList>
        <clip>
          <videoSource>
            <mediaFile>
              <file>video-part1.mp4</file>
            </mediaFile>
          </videoSource>
          <audioSource>
            <mediaFile>
              <file>video-part1.mp4</file>
            </mediaFile>
          </audioSource>
        </clip>
        <primaryClipIndex>0</primaryClipIndex>
      </clipList>
    </transcodeSource>
    <setRuntimeProperties>
      <property propertyPath="Media File Input Logo/filename" value="logo.png" />
    </setRuntimeProperties>
  </transcodeRequest>
```

Si quiere especificar /primarySourceFile para usar esta propiedad para denominar los archivos de salida mediante "expresiones", se recomienda pasar el XML de la lista de clips como propiedad *después* de la propiedad /primarySourceFile, para evitar que la lista de clips se reemplace por el valor de /primarySourceFile.

```xml
<?xml version="1.0" encoding="utf-8"?>
  <transcodeRequest>
    <setRuntimeProperties>
      <property propertyPath="/primarySourceFile" value="c:\temp\start.mxf" />
      <property propertyPath="/inactiveTimeout" value="65" />
      <property propertyPath="clipListXml" value="xxx">
      <extendedValue><![CDATA[<clipList>
        <clip>
          <videoSource>
            <mediaFile>
              <file>c:\temp\start.mxf</file>
            </mediaFile>
          </videoSource>
          <audioSource>
            <mediaFile>
              <file>c:\temp\start.mxf</file>
            </mediaFile>
          </audioSource>
        </clip>
        <primaryClipIndex>0</primaryClipIndex>
        </clipList>]]>
      </extendedValue>
      </property>
      <property propertyPath="Media File Input Logo/filename" value="logo.png" />
    </setRuntimeProperties>
  </transcodeRequest>
```

Con un recorte preciso de marco adicional:

```xml
<?xml version="1.0" encoding="utf-8"?>
  <transcodeRequest>
    <setRuntimeProperties>
      <property propertyPath="/primarySourceFile" value="start.mxf" />
      <property propertyPath="/inactiveTimeout" value="65" />
      <property propertyPath="clipListXml" value="xxx">
      <extendedValue><![CDATA[<clipList>
        <clip>
          <videoSource>
            <trim>
              <inPoint fps="25">00:00:05:24</inPoint>
              <outPoint fps="25">00:00:10:24</outPoint>
            </trim>
            <mediaFile>
              <file>start.mxf</file>
            </mediaFile>
          </videoSource>
          <audioSource>
            <trim>
              <inPoint fps="25">00:00:05:24</inPoint>
              <outPoint fps="25">00:00:10:24</outPoint>
            </trim>
            <mediaFile>
              <file>start.mxf</file>
            </mediaFile>
          </audioSource>
        </clip>
        <primaryClipIndex>0</primaryClipIndex>
        </clipList>]]>
      </extendedValue>
      </property>
      <property propertyPath="Media File Input Logo/filename" value="logo.png" />
    </setRuntimeProperties>
  </transcodeRequest>
```

## <a name="example-1--overlay-an-image-on-top-of-the-video"></a>Ejemplo 1: Superposición de una imagen en un vídeo

### <a name="presentation"></a>Presentación
Considere un ejemplo en el que desea superponer una imagen de logotipo en el vídeo de entrada durante la codificación. En este ejemplo, el vídeo de entrada se llama "Microsoft_HoloLens_Possibilities_816p24.mp4" y el logotipo, "logo.png". Debe realizar los pasos siguientes:

* Creación de un recurso de flujo de trabajo con el archivo de flujo de trabajo (consulte el siguiente ejemplo)
* Cree un recurso multimedia, que contenga dos archivos: MyInputVideo.mp4 como archivo principal y MyLogo.png.
* Envío de una tarea al procesador multimedia del flujo de trabajo premium de Codificador multimedia con los recursos de entrada anteriores y especificación de la siguiente cadena de configuración

Configuración:

```xml
<?xml version="1.0" encoding="utf-16"?>
  <transcodeRequest>
    <setRuntimeProperties>
      <property propertyPath="Media File Input/filename" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
      <property propertyPath="/primarySourceFile" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
      <property propertyPath="Media File Input Logo/filename" value="logo.png" />
    </setRuntimeProperties>
  </transcodeRequest>
```

En el ejemplo anterior, el nombre del archivo de vídeo se envía al componente de entrada de archivos multimedia y a la propiedad primarySourceFile. El nombre del archivo de logotipo se envía a otra entrada de archivos multimedia que se conecta al componente de superposición gráfica.

> [!NOTE]
> El nombre de archivo de vídeo se envía a la propiedad primarySourceFile. Esto se hace para usar esta propiedad en el flujo de trabajo para crear el nombre de archivo de salida correcto mediante expresiones, por ejemplo.

### <a name="step-by-step-workflow-creation"></a>Creación paso a paso de un flujo de trabajo
Estos son los pasos para crear un flujo de trabajo que toma como entrada dos archivos: un vídeo y una imagen. Añadirá la imagen en la parte superior del vídeo.

Abra el **Diseñador de flujo de trabajo** y seleccione **File** > **New Workspace** > **Transcode Blueprint** (Archivo > Nueva área de trabajo > Transcodificar plano).

El nuevo flujo de trabajo muestra tres elementos:

* Primary Source File (Archivo de origen principal)
* Clip List XML (XML de la lista de clips)
* Output File/Asset (Recurso/Archivo de salida)  

![Nuevo flujo de trabajo de Encoding](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture9_empty.png)

*Nuevo flujo de trabajo de Encoding*

Para aceptar el archivo multimedia de entrada, debe empezar con la incorporación de un componente de entrada de archivo multimedia. Para agregar un componente al flujo de trabajo, busque en el cuadro de búsqueda del repositorio y arrastre la entrada deseada al panel del diseñador.

A continuación, agregue el archivo de vídeo que se usará para diseñar el flujo de trabajo. Para ello, haga clic en el panel de fondo de Diseñador de flujo de trabajo y busque la propiedad de archivo de origen principal en el panel de propiedades de la derecha. Haga clic en el icono de carpeta y seleccione el archivo de vídeo adecuado.

![Primary Source File (Archivo de origen principal)](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture10_primaryfile.png)

*Primary Source File (Archivo de origen principal)*

A continuación, especifique el archivo de vídeo en el componente de entrada de archivo multimedia.   

![Origen de entrada de archivo multimedia](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture11_mediafileinput.png)

*Origen de entrada de archivo multimedia*

En cuanto lo haya hecho, el componente de entrada de archivos multimedia inspeccionará el archivo y rellenará sus clavijas de salida para reflejar el archivo que inspeccionó.

El siguiente paso es agregar un "Video Data Type Updater" (Actualizador de tipo de datos de vídeo) para especificar el espacio de color a Rec.709. Agregue un convertidor de formato de vídeo establecido en las opciones Data Layout/Layout type = Configurable Planar (Diseño de datos/Tipo de diseño = Planar configurable). Esto convertirá la secuencia de vídeo a un formato que puede considerarse el origen del componente de superposición.

![Actualizador de tipo de datos de vídeo y convertidor de formato](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture12_formatconverter.png)

*Actualizador de tipo de datos de vídeo y convertidor de formato*

![Layout type = Configurable Planar (Tipo de diseño = Planar configurable)](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture12_formatconverter2.png)

*El valor de Layout type es Configurable Planar (Planar configurable)*

A continuación, agregue un componente de superposición de vídeo y conecte la clavija de vídeo (sin comprimir) a la clavija de vídeo (sin comprimir) de la entrada de archivo multimedia.

Agregue otra entrada de archivo multimedia (para cargar el archivo de logotipo), haga clic en este componente y cambie el nombre a Media File Input Logo (Logotipo de entrada de archivo multimedia), seleccione una imagen (por ejemplo, un archivo .png) de la propiedad de archivo. Conecte la clavija de la imagen sin comprimir al de la imagen sin comprimir de la superposición.

![Componente de superposición y origen del archivo de imagen](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture13_overlay.png)

*Componente de superposición y origen del archivo de imagen*

Si quiere modificar la posición del logotipo en el vídeo (por ejemplo, ponerlo en el 10 % de la esquina superior izquierda del vídeo), desactive la opción Manual Input (Entrada manual). Puede desactivar esta opción porque se está usando una entrada de archivo multimedia para proporcionar el archivo de logotipo en el componente de superposición.

![Posición de superposición](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture14_overlay_position.png)

*Posición de superposición*

Para codificar la secuencia de vídeo en H.264, agregue los componentes AVC Video Encoder (Codificador de vídeo AVC) y el codificador AAC a la superficie del diseñador. Conecte las clavijas.
Configure el codificador AAC y seleccione Audio Format Conversion/Preset: 2.0 (L, R) (Conversión a formato de audio/predefinido: 2.0 [I, D]).

![Codificadores de audio y vídeo](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture15_encoders.png)

*Codificadores de audio y vídeo*

Ahora agregue los componentes **ISO Mpeg-4 Multiplexer** (Multiplexor ISO Mpeg-4) y **File Output** (Salida de archivo) y conecte las clavijas tal y como se muestra.

![Multiplexor MP4 y salida de archivo](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture16_mp4output.png)

*Multiplexor MP4 y salida de archivo*

Debe establecer el nombre del archivo de salida. Haga clic en el componente **File Output** (Salida de archivo) y edite la expresión del archivo:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_withoverlay.mp4`

![Nombre de salida de archivo](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture17_filenameoutput.png)

*Nombre de salida de archivo*

Puede ejecutar el flujo de trabajo en la máquina para comprobar que lo hace correctamente.

Al finalizar, puede ejecutarlo en Azure Media Services.

Primero, prepare un recurso de Azure Media Services con dos archivos: el archivo de vídeo y el logotipo. Puede hacerlo mediante .NET o la API de REST. También puede hacerlo con Azure Portal o el [Explorador de Azure Media Services](https://github.com/Azure/Azure-Media-Services-Explorer) (AMSE).

Este tutorial muestra cómo administrar recursos con AMSE. Hay dos maneras de agregar archivos a un recurso:

* Cree una carpeta local, copie en ella los dos archivos y arrastre y coloque la carpeta en la pestaña **Recurso** .
* Cargue el archivo de vídeo como un recurso, muestre la información del recurso, vaya a la pestaña Archivos y cargar un archivo adicional (logotipo).

> [!NOTE]
> Asegúrese de establecer un archivo principal en el recurso (el archivo de vídeo principal).

![Archivos de recursos en AMSE](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture18_assetinamse.png)

*Archivos de recursos en AMSE*

Seleccione el recurso y elija codificarlo con el codificador Premium. Cargue el flujo de trabajo y selecciónelo.

Haga clic en el botón para pasar datos al procesador y agregue el siguiente código XML para establecer las propiedades del sistema de tiempo de ejecución:

![Codificador Premium en AMSE](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture19_amsepremium.png)

*Codificador Premium en AMSE*

A continuación, pegue los datos XML siguientes. Debe especificar el nombre del archivo de vídeo tanto para la entrada del archivo multimedia como para el archivo de origen principal. Especifique el nombre del nombre del archivo también para el logotipo.

```xml
<?xml version="1.0" encoding="utf-16"?>
  <transcodeRequest>
    <setRuntimeProperties>
      <property propertyPath="Media File Input/filename" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
      <property propertyPath="/primarySourceFile" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
      <property propertyPath="Media File Input Logo/filename" value="logo.png" />
    </setRuntimeProperties>
  </transcodeRequest>
```

![setRuntimeProperties](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture20_amsexmldata.png)

*setRuntimeProperties*

Si utiliza el SDK de .Net para crear y ejecutar la tarea, estos datos XML deben pasarse como la cadena de configuración.

```csharp
public ITask AddNew(string taskName, IMediaProcessor mediaProcessor, string configuration, TaskOptions options);
```

Cuando se complete el trabajo, el archivo MP4 en el recurso de salida mostrará la superposición.

![Superposición en el vídeo](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture21_resultoverlay.png)

*Superposición en el vídeo*

Puede descargar el flujo de trabajo de ejemplo desde [GitHub](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows/).

## <a name="example-2--multiple-audio-language-encoding"></a>Ejemplo 2: Codificación de varios idiomas de audio

En [GitHub](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows/MultilanguageAudioEncoding), hay un ejemplo de un flujo de trabajo para la codificación de varios idiomas de audio.

Esta carpeta contiene un flujo de trabajo de ejemplo que puede utilizarse para codificar un archivo MXF en un recurso con varios archivos MP4 y varias pistas de audio.

El flujo de trabajo presupone que el archivo MXF contiene una sola pista de audio, por lo que las otras pistas deben pasarse en otros archivos de audio (WAV, MP4, etc.).

Para realizar la codificación, siga estos pasos:

* Cree un recurso de Media Services con el archivo MXF y los archivos de audio (de 0 a 18 archivos de audio).
* Asegúrese de que el archivo MXF está establecido como archivo principal.
* Cree un trabajo y una tarea mediante el procesador del codificador de flujos de trabajo premium. Utilice el flujo de trabajo proporcionado (MultiMP4-1080p-19audio-v1.workflow).
* Pase los datos de setruntime.xml a la tarea (si utiliza el Explorador de Azure Media Services, use el botón “pass xml data to the workflow” [pasar datos xml al flujo de trabajo]).
  * Actualice los datos XML para especificar las etiquetas de idioma y los nombres de archivo correctos.
  * El flujo de trabajo tiene unos componentes de audio cuyo nombre está comprendido entre Audio 1 y Audio 18.
  * RFC5646 es compatible con la etiqueta de idioma.

```xml
<?xml version="1.0" encoding="utf-16"?>
<transcodeRequest>
  <setRuntimeProperties>
    <property propertyPath="Media File Input Video/filename" value="MainVideo.mxf" />
    <property propertyPath="Language/language_code" value="en" />
    <property propertyPath="/primarySourceFile" value="MainVideo.mxf" />
    <property propertyPath="Audio 1/Media File Input/filename" value="french-audio.wav" />
    <property propertyPath="Audio 1/Language/language_code" value="fr" />
    <property propertyPath="Audio 2/Media File Input/filename" value="german-audio.wav" />
    <property propertyPath="Audio 2/Language/language_code" value="de" />
    <property propertyPath="Audio 3/Media File Input/filename" value="japanese-audio.wav" />
    <property propertyPath="Audio 3/Language/language_code" value="ja" />
  </setRuntimeProperties>
</transcodeRequest>
```

* El recurso codificado contendrá pistas de audio en varios idiomas que podrán seleccionarse en Azure Media Player.

## <a name="see-also"></a>Consulte también
* [Introducción de la codificación Premium en Azure Media Services](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services)
* [Uso de la codificación Premium en Azure Media Services](https://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services)
* [Codificación de contenido a petición con Azure Media Services](media-services-encode-asset.md#media-encoder-premium-workflow)
* [Códecs y formatos de flujo de trabajo del Codificador multimedia Premium](media-services-premium-workflow-encoder-formats.md)
* [Archivos de flujo de trabajo de ejemplo](https://github.com/Azure/azure-media-services-samples)
* [Explorador de Azure Media Services](https://aka.ms/amse)

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
