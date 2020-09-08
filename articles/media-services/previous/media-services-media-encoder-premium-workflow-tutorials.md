---
title: Tutoriales avanzados sobre el flujo de trabajo premium del codificador multimedia
description: Este documento contiene tutoriales que muestran cómo realizar tareas avanzadas con el flujo de trabajo premium del codificador multimedia y también cómo crear flujos de trabajo complejos con el Diseñador de flujo de trabajo.
services: media-services
documentationcenter: ''
author: xstof
manager: femila
editor: ''
ms.assetid: 1ba52865-b4a8-4ca0-ac96-920d55b9d15b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: christoc
ms.reviewer: xpouyat; juliako
ms.openlocfilehash: 55d9d9191dc62d7397c5de944985d5653eff4e13
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269835"
---
# <a name="advanced-media-encoder-premium-workflow-tutorials"></a>Tutoriales avanzados sobre el flujo de trabajo premium del codificador multimedia

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Información general
Este documento contiene tutoriales que muestran cómo personalizar los flujos de trabajo con el **Diseñador de flujo de trabajo**. Encontrará los archivos de flujo de trabajo reales [aquí](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows/PremiumEncoderWorkflowSamples).  

## <a name="toc"></a>TABLA DE CONTENIDO
Se tratan los siguientes temas:

* [Codificación de MXF en un MP4 de velocidad de bits única](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4)
  * [Inicio de un nuevo flujo de trabajo](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_start_new)
  * [Utilización de la entrada de archivo multimedia](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_file_input)
  * [Inspección de transmisiones multimedia](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_streams)
  * [Incorporación de un codificador de vídeo para la generación de archivos .MP4](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_file_generation)
  * [Codificación de la secuencia de audio](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_audio)
  * [Multiplexación de secuencias de audio y vídeo en un contenedor MP4](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_audio_and_fideo)
  * [Escritura del archivo MP4](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_writing_mp4)
  * [Creación de un recurso de Media Services desde el archivo de salida](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_asset_from_output)
  * [Comprobación local del flujo de trabajo terminado](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_test)
* [Codificación de MXF en archivos MP4 de varias velocidades de bits: empaquetado dinámico habilitado](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging)
  * [Incorporación de una o más salidas MP4 adicionales](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_more_outputs)
  * [Configuración de los nombres de salida de archivo](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_conf_output_names)
  * [Incorporación de una pista de audio independiente](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_audio_tracks)
  * [Incorporación del archivo SMIL ".ISM"](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging_ism_file)
* [Codificación de MXF en archivos MP4 de varias velocidades de bits: plano mejorado](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4)
  * Información general del flujo de trabajo a mejorar
  * [Convenciones de nomenclatura de los archivos](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4_file_naming)
  * [Publicación de propiedades de componente en la raíz del flujo de trabajo](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4_publishing)
  * [Se han generado nombres de archivo de salida que se basan en los valores de propiedad publicados](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to__multibitrate_MP4_output_files)
* [Incorporación de miniaturas a la salida MP4 de varias velocidades de bits](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4)
  * Información general del flujo de trabajo al que agregar miniaturas
  * [Incorporación de codificación JPG](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4__with_jpg)
  * [Relación con la conversión de espacio de colores](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_color_space)
  * [Escritura de las miniaturas](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_writing_thumbnails)
  * [Detección de errores en un flujo de trabajo](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_errors)
  * [Flujo de trabajo finalizado](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4_finish)
* [Recorte basado en tiempo de salida MP4 de varias velocidades de bits](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim)
  * [Información general del flujo de trabajo al que empezar a agregar el recorte](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim_start)
  * [Utilización del recortador de transmisiones](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim_use_stream_trimmer)
  * [Flujo de trabajo finalizado](media-services-media-encoder-premium-workflow-tutorials.md#time_based_trim_finish)
* [Introducción al componente generado por script](media-services-media-encoder-premium-workflow-tutorials.md#scripting)
  * [Scripting en un flujo de trabajo: hola a todos](media-services-media-encoder-premium-workflow-tutorials.md#scripting_hello_world)
* [Recorte basado en fotogramas de salida MP4 de varias velocidades de bits](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim)
  * [Información general del plano al que empezar a agregar el recorte](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_start)
  * [Utilización del XML de la lista de clips](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_clip_list)
  * [Modificación de la lista de clips desde un componente generado por script](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_modify_clip_list)
  * [Incorporación de una propiedad de conveniencia ClippingEnabled](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim_clippingenabled_prop)

## <a name="encoding-mxf-into-a-single-bitrate-mp4"></a><a id="MXF_to_MP4"></a>Codificación de MXF en un MP4 de velocidad de bits única
En esta sección se demuestra cómo crear un archivo .MP4 de velocidad de bits única con audio codificado en AAC-HE a partir de un archivo de entrada .MXF.

### <a name="starting-a-new-workflow"></a><a id="MXF_to_MP4_start_new"></a>Inicio de un nuevo flujo de trabajo
Abra el Diseñador de flujo de trabajo y seleccione File (Archivo) > New Workspace (Nueva área de trabajo) > Transcode Blueprint (Transcodificar plano).

El nuevo flujo de trabajo muestra tres elementos:

* Primary Source File (Archivo de origen principal)
* Clip List XML (XML de la lista de clips)
* Output File/Asset (Recurso/Archivo de salida)  

![Nuevo flujo de trabajo de Encoding](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-transcode-blueprint.png)

*Nuevo flujo de trabajo de Encoding*

### <a name="using-the-media-file-input"></a><a id="MXF_to_MP4_with_file_input"></a>Utilización de la entrada de archivo multimedia
Para aceptar el archivo multimedia de entrada, debe empezar con la incorporación de un componente Media File Input (Entrada de archivo multimedia). Para agregar un componente al flujo de trabajo, busque en el cuadro de búsqueda del repositorio y arrastre la entrada deseada al panel del diseñador. Repita la acción para la entrada de archivo multimedia y conecte el componente Primary Source File (Archivo de origen principal) a la clavija de entrada de Filename (Nombre de archivo) de la entrada de archivo multimedia.

![Entrada de archivo multimedia conectada](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-file-input.png)

*Entrada de archivo multimedia conectada*

Inicialmente, identifique un archivo de ejemplo adecuado para usar al diseñar un flujo de trabajo personalizado. Para ello, haga clic en el fondo del panel del diseñador y busque la propiedad de archivo de origen principal en el panel de propiedades de la derecha. Haga clic en el icono de carpeta y seleccione el archivo con el que quiere probar el flujo de trabajo. El componente de entrada de archivo multimedia inspecciona el archivo y rellena sus clavijas de salida para reflejar los detalles del archivo de ejemplo que inspeccionó.

![Entrada de archivo multimedia rellena](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-populated-media-file-input.png)

*Entrada de archivo multimedia rellena*

Ahora que la entrada está rellena, el siguiente paso es configurar la codificación de salida. De forma similar a cómo se configuró el archivo de origen principal, configure ahora la propiedad Output Folder Variable (Variable de carpeta de salida), justo debajo de él.

![Propiedades de entrada y salida configuradas](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-configured-io-properties.png)

*Propiedades de entrada y salida configuradas*

### <a name="inspecting-media-streams"></a><a id="MXF_to_MP4_streams"></a>Inspección de transmisiones multimedia
A menudo se quiere saber qué aspecto tendrá la secuencia que se ejecuta mediante el flujo de trabajo. Para inspeccionar una transmisión en cualquier punto del flujo de trabajo, solo tiene que hacer clic en una clavija de entrada o salida en cualquiera de los componentes. En este caso, intente hacer clic en la clavija de salida Uncompressed Video (Vídeo sin comprimir) de la entrada de archivo multimedia. Se abre un cuadro de diálogo que permite inspeccionar el vídeo saliente.

![Inspección de la clavija de salida de vídeo sin comprimir](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-inspecting-uncompressed-video-output.png)

*Inspección de la clavija de salida de vídeo sin comprimir*

En este caso, muestra que el vídeo contiene una entrada de 1920 x 1080 a 24 fotogramas por segundo en un muestreo de 4:2:2 en un vídeo de casi 2 minutos.

### <a name="adding-a-video-encoder-for-mp4-file-generation"></a><a id="MXF_to_MP4_file_generation"></a>Incorporación de un codificador de vídeo para la generación de archivos .MP4
Ahora, una clavija de salida de vídeo sin comprimir y varias de audio sin comprimir están disponibles para su utilización en la entrada de archivo multimedia. Para codificar el vídeo entrante, se debe agregar un componente de codificación al flujo de trabajo; en este caso, para generar archivos MP4.

Para codificar la secuencia de vídeo en H.264, agregue el componente AVC Video Encoder (Codificador de vídeo AVC) a la superficie del diseñador. Este componente toma una secuencia de vídeo sin comprimir como entrada y entrega una secuencia de vídeo comprimida AVC en su clavija de salida.

![Codificador de AVC no conectado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-unconnected-avc-encoder.png)

*Codificador de AVC no conectado*

Sus propiedades determinan cómo ocurre exactamente la codificación. Echemos un vistazo a algunas de las opciones más importantes:

* Ancho de salida y alto de salida: determinan la resolución del vídeo codificado. En este caso, 640 x 360 es un buen valor.
* Velocidad de fotogramas: cuando se establece en transferencia, simplemente adoptará la velocidad de fotogramas de origen, aunque es posible reemplazar esta opción. Esa conversión de velocidad de fotogramas no está compensada por el movimiento.
* Perfil y nivel: determinan el nivel y el perfil de AVC. Para obtener fácilmente más información acerca de los diferentes niveles y perfiles, haga clic en el icono de signo de interrogación en el componente AVC Video Encoder (Codificador de vídeo AVC) y la página de ayuda mostrará más detalles acerca de cada uno de los niveles. En este ejemplo, use el perfil principal en el nivel 3.2 (valor predeterminado).
* Velocidad de bits (kbps) y modo de control de velocidad: en nuestro escenario, optamos por una salida de velocidad de bits constante (CBR) a 1200 kbps.
* Formato de vídeo: proporciona información sobre la VUI (información de facilidad de uso de vídeo) que se escribe en la secuencia H.264 (información secundaria que un descodificador podría utilizar para mejorar la presentación, pero que no es esencial para descodificar correctamente):
* NTSC (habitual en Estados Unidos o Japón, utiliza 30 fps)
* PAL (típica en Europa, utiliza 25 fps)
* Modo de tamaño de GOP: se configura un tamaño fijo de GOP para nuestros fines con un intervalo de clave de 2 segundos con GOP cerrados. Este valor de 2 segundos garantiza la compatibilidad con el empaquetado dinámico que proporciona Azure Media Services.

Para alimentar el codificador AVC, conecte la clavija de salida Uncompressed Video (Vídeo sin comprimir) del componente de entrada del archivo multimedia a la clavija de entrada Uncompressed Video (Vídeo sin comprimir) del codificador AVC.

![Codificador AVC conectado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-avc-encoder.png)

*Codificador principal AVC conectado*

### <a name="encoding-the-audio-stream"></a><a id="MXF_to_MP4_audio"></a>Codificación de la secuencia de audio
En este punto, aún es necesario codificar la secuencia de audio sin comprimir original. Para la compresión de la secuencia de audio, agregue un componente de codificador AAC (Dolby) al flujo de trabajo.

![Codificador de AVC no conectado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-unconnected-aac-encoder.png)

*Codificador de AAC no conectado*

Ahora se produce una incompatibilidad: solo hay una clavija de entrada de audio sin comprimir en el codificador AAC, mientras que es muy probable que la entrada de archivo multimedia tenga dos secuencias diferentes de audio sin comprimir disponibles: una para el canal de audio izquierdo y otra para el derecho. (Si está trabajando con sonido envolvente, eso suma seis canales). Por lo tanto, no es posible conectar directamente el audio desde el origen de entrada de archivo multimedia en el codificador de audio AAC. El componente AAC espera una secuencia de audio llamada "entrelazada": una única transmisión que tiene los canales de la izquierda y los de la derecha entrelazados entre sí. Una vez que sepamos a partir de nuestro archivo de origen multimedia qué pistas de audio están en qué posición del origen, podemos generar dicha secuencia de audio entrelazada con las posiciones de altavoces correctamente asignadas para el canal izquierdo y el derecho.

Lo primero que queremos es generar una secuencia entrelazada a partir de los canales necesarios de audio del origen. El componente Audio Stream Interleaver (Entrelazador de secuencias de audio) controla esto por nosotros. Agréguelo al flujo de trabajo y conecte en él las salidas de audio desde la entrada de archivo multimedia.

![Entrelazador de secuencias de audio conectado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-audio-stream-interleaver.png)

*Entrelazador de secuencias de audio conectado*

Ahora que tenemos una secuencia de audio entrelazada, todavía no hemos especificado a dónde asignar las posiciones de los altavoces izquierdo o derecho. Para ello, podemos aprovechar el componente Speaker Position Assigner (Asignador de posiciones de altavoz).

![Incorporación de un asignador de posiciones de altavoz](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-adding-speaker-position-assigner.png)

*Incorporación de un asignador de posiciones de altavoz*

Configure el asignador de posiciones de altavoz para la utilización de un flujo de entrada en estéreo mediante un filtro preestablecido del codificador denominado "Custom" (Personalizado) y del canal preestablecido denominado "2.0 (L,R)". (Esta configuración asigna la posición del altavoz izquierdo al canal 1 y la posición del derecho al 2).

Conecte la salida del asignador de posiciones de altavoz a la entrada del codificador AAC. A continuación, indique al codificador AAC que debe trabajar con un canal preestablecido "2.0 (L, R)" para que sepa que va a tratar con audio estéreo como entrada.

### <a name="multiplexing-audio-and-video-streams-into-an-mp4-container"></a><a id="MXF_to_MP4_audio_and_fideo"></a>Multiplexación de secuencias de audio y vídeo en un contenedor MP4
Una vez que tenemos nuestra secuencia de vídeo codificado AVC y nuestra secuencia de audio codificada AAC, podemos capturar ambas en un contenedor .MP4. El proceso de mezclar transmisiones diferentes en una sola se denomina "multiplexación" (o "muxing"). En este caso, estamos entrelazando las secuencias de audio y de vídeo en un solo paquete coherente .MP4. El componente encargado de coordinar esta operación para un contenedor .MP4 se denomina ISO MPEG-4 Multiplexer (Multiplexor ISO MPEG-4). Agregue uno a la superficie del diseñador y conecte el codificador de vídeo AVC y el codificador AAC a sus entradas.

![Multiplexor MPEG4 conectado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-mpeg4-multiplexer.png)

*Multiplexor MPEG4 conectado*

### <a name="writing-the-mp4-file"></a><a id="MXF_to_MP4_writing_mp4"></a>Escritura del archivo MP4
Al escribir un archivo de salida, se utiliza el componente de salida de archivo. Podemos conectar este a la salida del multiplexor ISO MPEG-4 para que su salida se grabe en el disco. Para ello, conecte la clavija de salida Container (MPEG-4) (Contenedor (MPEG-4)) a la clavija de entrada Write (Grabar) del componente File Output (Salida de archivo).

![Salida de archivo conectada](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connected-file-output.png)

*Salida de archivo conectada*

El nombre de archivo que va a utilizar se determina mediante la propiedad File (Archivo). Aunque esa propiedad puede ser codificada con un valor determinado, es probable que desee establecerla mediante una expresión.

Para que el flujo de trabajo determine automáticamente la propiedad de nombre de archivo de salida a partir de una expresión, haga clic en el botón junto al nombre de archivo (junto al icono de carpeta). A continuación, en el menú desplegable seleccione "Expression" (Expresión). Se abre el editor de expresiones. Borre el contenido del editor en primer lugar.

![Editor de expresiones vacío](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-empty-expression-editor.png)

*Editor de expresiones vacío*

El editor de expresiones permite escribir cualquier valor literal y combinarlo con una o más variables. Las variables comienzan con un signo de dólar. Al pulsar la tecla $, el editor muestra un cuadro de lista desplegable con las opciones de variables disponibles. En nuestro caso, vamos a utilizar una combinación de la variable del directorio de salida y la variable de nombre de archivo de entrada de base:

`${ROOT_outputWriteDirectory}\\${ROOT_sourceFileBaseName}.MP4`

![Editor de expresiones relleno](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-expression-editor.png)

*Editor de expresiones relleno*

> [!NOTE]
> Para poder ver un archivo de salida del trabajo de codificación en Azure, debe proporcionar un valor en el editor de expresiones.
>
>

Después de hacer clic en Aceptar para confirmar la expresión, la ventana de propiedades muestra una vista previa del valor que resuelve la propiedad de archivo en este momento.

![La expresión de archivo resuelve el directorio de salida](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-file-expression-resolves-output-dir.png)

*La expresión de archivo resuelve el directorio de salida*

### <a name="creating-a-media-services-asset-from-the-output-file"></a><a id="MXF_to_MP4_asset_from_output"></a>Creación de un recurso de Media Services desde el archivo de salida
Aunque ya hemos grabado un archivo de salida MP4, todavía es necesario indicar que este archivo pertenece al recurso de salida que generaran los servicios multimedia como resultado de ejecutar este flujo de trabajo. Para ello, se utiliza el nodo Output File/Asset (Recurso/Archivo de salida) en el lienzo del flujo de trabajo. Todos los archivos que entran en este nodo forman parte del recurso resultante de Azure Media Services.

Conecte el componente File Output (Salida de archivo) al componente Output File/Asset (Recurso/Archivo de salida) para finalizar el flujo de trabajo.

![Flujo de trabajo finalizado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-workflow.png)

*Flujo de trabajo finalizado*

### <a name="test-the-finished-workflow-locally"></a><a id="MXF_to_MP4_test"></a>Comprobación local del flujo de trabajo terminado
Para probar el flujo de trabajo de forma local, haga clic en el botón Reproducir en la barra de herramientas de la parte superior. Cuando el flujo de trabajo haya terminado de ejecutarse, inspeccione la salida generada en la carpeta de salida configurada. Allí podrá ver el archivo de salida MP4 terminado que se codificó desde el archivo de origen de entrada MXF.

## <a name="encoding-mxf-into-mp4---multibitrate-dynamic-packaging-enabled"></a><a id="MXF_to_MP4_with_dyn_packaging"></a>Codificación de MXF en archivos MP4: empaquetado dinámico de varias velocidades de bits habilitado
En este tutorial crearemos un conjunto de archivos .MP4 de varias velocidades de bits con audio codificado en AAC a partir de un archivo de entrada .MXF.

Cuando se desea una salida de recurso de varias velocidades de bits para su utilización en combinación con las características de empaquetado dinámico que ofrece Azure Media Services, deberán generarse varios archivos MP4 alineados con GOP, cada uno de ellos con velocidad de bits y resolución diferentes. Para ello, el tutorial [Codificación de MXF en un MP4 de velocidad de bits única](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4) nos proporciona un buen punto de partida.

![Inicio del flujo de trabajo](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-starting-workflow.png)

*Inicio del flujo de trabajo*

### <a name="adding-one-or-more-additional-mp4-outputs"></a><a id="MXF_to_MP4_with_dyn_packaging_more_outputs"></a>Incorporación de una o más salidas MP4 adicionales
Cada archivo MP4 en el recurso resultante de Azure Media Services admite una velocidad de bits y una resolución diferentes. Vamos a agregar uno o más archivos de salida MP4 al flujo de trabajo.

Para estar seguros de que tenemos todos nuestros codificadores de vídeo creados con la misma configuración, resulta más conveniente duplicar el codificador de vídeo AVC ya existente y configurar otra combinación de resolución y velocidad de bits (agreguemos una de 960 x 540 a 25 fotogramas por segundo a 2,5 Mbps). Para duplicar el codificador existente, cópielo y péguelo en la superficie del diseñador.

Conecte la clavija de salida de Uncompressed Video (Vídeo sin comprimir) de la entrada de archivo multimedia en nuestro nuevo componente AVC.

![Segundo codificador AVC conectado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-second-avc-encoder-connected.png)

*Segundo codificador AVC conectado*

Ahora, adapte la configuración de nuestro nuevo codificador AVC a la salida de 960 x 540 a 2.5 Mbps. (Utilice sus propiedades "Output width" (ancho de salida), "Output height" (Altura de salida) y "Bitrate (kbps)" (Velocidad de bits (kbps) para esto).

Dado que deseamos utilizar el recurso resultante junto con el empaquetado dinámico de Azure Media Services, el punto de conexión del streaming debe ser capaz de generar, a partir de estos archivos MP4, fragmentos de HLS/Fragmented MP4/DASH exactamente alineados entre sí de forma que los clientes que cambian entre distintas velocidades de bits obtengan una experiencia única de audio y vídeo continuo sin cortes. Para que esto suceda, debemos asegurarnos de que, en las propiedades de ambos codificadores AVC el tamaño del GOP ( o "grupo de imágenes") de ambos archivos MP4 esté establecido en 2 segundos, lo cual puede hacerse mediante:

* el establecimiento de la opción GOP Size Mode (Modo de cambiar el tamaño de GOP) en Fixed GOP size (Tamaño fijo de GOP) y
* el Key Frame Interval (intervalo de fotogramas clave) en dos segundos.
* el establecimiento del control de IDR de GOP en Closed GOP (GOP cerrado) para tener la seguridad de que todos los GOP permanecen por sí mismos sin dependencias.

Para facilitar la comprensión de nuestro flujo de trabajo, cambie el nombre del primer codificador AVC por "Codificador de vídeo AVC 640 x 360 1200 kbps" y el segundo codificador AVC por "Codificador de vídeo AVC 960 x 540 2500 kbps".

Ahora, agregue un segundo multiplexor ISO MPEG-4 y una segunda salida de archivo. Conecte el multiplexor al nuevo codificador AVC y asegúrese de que su salida está dirigida a la salida del archivo. A continuación, conecte también la salida del codificador de audio AAC a la nueva entrada de multiplexor. La salida del archivo a su vez puede conectarse al nodo Output File/Asset (Recurso/Archivo de salida) para agregarlo al recurso de Media Services que se va a crear.

![Segundo multiplexor y salida de archivo conectados](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-second-muxer-file-output-connected.png)

*Segundo multiplexor y salida de archivo conectados*

Para la compatibilidad con el empaquetado dinámico de Azure Media Services, configure la opción Chunk Mode (Modo de fragmento) del multiplexor en GOP count or duration (Recuento o duración del GOP) y establezca los GOP por fragmento en 1. (Este debería ser el valor predeterminado).

![Modos de fragmento del multiplexor](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-muxer-chunk-modes.png)

*Modos de fragmento del multiplexor*

Nota: puede repetir este proceso para todas las combinaciones de velocidad de bits y resolución que desee agregar a la salida del recurso.

### <a name="configuring-the-file-output-names"></a><a id="MXF_to_MP4_with_dyn_packaging_conf_output_names"></a>Configuración de los nombres de salida de archivo
Tenemos más de un único archivo agregado al recurso de salida. Esto hace que sea necesario asegurarse de que los nombres de archivo para cada uno de los archivos de salida son diferentes entre sí y quizás incluso aplicar una convención de nomenclatura de archivos para que resulte evidente viendo el nombre del archivo aquello con lo que está tratando.

La nomenclatura de archivos de salida se puede controlar mediante expresiones en el diseñador. Abra el panel de propiedades para uno de los componentes de salida de archivo y abra el editor de expresiones para la propiedad del archivo. Nuestro primer archivo de salida se configuró mediante la siguiente expresión (consulte el tutorial para pasar de [MXF a una salida MP4 de velocidad de bits única](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4)):

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}.MP4`

Esto significa que el nombre de archivo viene determinado por dos variables: el directorio de salida en el que escribir y el nombre base del archivo de origen. El primero se expone como una propiedad en la raíz del flujo de trabajo mientras que el segundo viene determinado por el archivo entrante. El directorio de salida es el que se usa para las pruebas locales. El motor del flujo de trabajo invalidará esta propiedad cuando el procesador de multimedia basado en la nube ejecute el flujo de trabajo en Azure Media Services.
Para dar a todos nuestros archivos de salida un nombre de salida coherente, cambie la primera expresión de nomenclatura de archivos a:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_640x360_1.MP4`

y la segunda a:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_960x540_2.MP4`

Ejecute una prueba intermedia para asegurarse de que ambos archivos MP4 de salida se generan correctamente.

### <a name="adding-a-separate-audio-track"></a><a id="MXF_to_MP4_with_dyn_packaging_audio_tracks"></a>Incorporación de una pista de audio independiente
Como veremos más adelante, cuando se genera un archivo .ism para que vaya con los archivos MP4 de salida, necesitaremos también un archivo MP4 de solo audio como la pista de audio para el streaming adaptable. Para crear este archivo, agregue un multiplexor adicional al flujo de trabajo (multiplexor ISO-MPEG-4) y conecte la clavija de salida del codificador AAC con su clavija de entrada para la pista 1.

![Multiplexor de audio agregado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-audio-muxer-added.png)

*Multiplexor de audio agregado*

Cree un tercer componente de salida de archivo para generar la transmisión saliente del multiplexor y configure la expresión de nomenclatura de archivos de la siguiente manera:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_128kbps_audio.MP4`

![Multiplexor de audio creando una salida de archivo](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-audio-muxer-creating-file-output.png)

*Multiplexor de audio creando una salida de archivo*

### <a name="adding-the-ism-smil-file"></a><a id="MXF_to_MP4_with_dyn_packaging_ism_file"></a>Incorporación del archivo SMIL .ISM
Para que el empaquetado dinámico funcione en combinación con ambos archivos MP4 (y con los archivos MP4 de solo audio) en nuestro recurso de Media Services, también es necesario un archivo de manifiesto (también denominado archivo "SMIL": es decir, lenguaje de integración multimedia sincronizada). Este archivo indica a Azure Media Services qué archivos MP4 están disponibles para el empaquetado dinámico y cuáles de ellos debe tener en cuenta para el streaming de audio. Un archivo de manifiesto típico para un conjunto de archivos MP4 con una única secuencia de audio tiene el siguiente aspecto:

```xml
    <?xml version="1.0" encoding="utf-8" standalone="yes"?>
    <smil xmlns="https://www.w3.org/2001/SMIL20/Language">
      <head>
        <meta name="formats" content="mp4" />
      </head>
      <body>
        <switch>
          <video src="H264_1900kbps_AAC_und_ch2_96kbps.mp4" />
          <video src="H264_1300kbps_AAC_und_ch2_96kbps.mp4" />
          <video src="H264_900kbps_AAC_und_ch2_96kbps.mp4" />
          <audio src="AAC_ch2_96kbps.mp4" title="AAC_und_ch2_96kbps" />
        </switch>
      </body>
    </smil>
```

El archivo .ism contiene una instrucción switch con una referencia a cada uno de los archivos de vídeo MP4 y además de dichas referencias, contiene una (o más) referencias de archivo de audio a un MP4 que solo contiene el audio.

Se puede generar el archivo de manifiesto para nuestro conjunto de archivos MP4 mediante un componente llamado "AMS Manifest Writer" (Escritor de manifiesto AMS). Para utilizarlo, arrástrelo a la superficie y conecte las clavijas de salida de "Write Complete" (Escritura completa) de los tres componentes de salida de archivo a la entrada del escritor de manifiesto AMS. A continuación, asegúrese de conectar la salida del escritor de manifiesto AMS al recurso o archivo de salida.

Al igual que con nuestros otros componentes de salida archivo, configure el nombre de salida del archivo .ism con una expresión:

`${ROOT_outputWriteDirectory}\\${ROOT_sourceFileBaseName}_manifest.ism`

El flujo de trabajo finalizado tiene el siguiente aspecto:

![Flujo de trabajo finalizado de MXF a MP4 con varias velocidades de bits](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-mxf-to-multibitrate-mp4-workflow.png)

*Flujo de trabajo finalizado de MXF a MP4 con varias velocidades de bits*

## <a name="encoding-mxf-into-multibitrate-mp4---enhanced-blueprint"></a><a id="MXF_to__multibitrate_MP4"></a>Codificación de MXF en archivos MP4 de varias velocidades de bits: plano mejorado
En el [tutorial anterior sobre flujos de trabajo](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging) hemos visto cómo se puede convertir un solo recurso de entrada MXF en un recurso de salida con archivos MP4 de varias velocidades de bits, un archivo MP4 de solo audio y un archivo de manifiesto para su utilización junto con el empaquetado dinámico de Azure Media Services.

En este tutorial se muestra cómo se pueden mejorar algunos de los aspectos y simplificarlos aún más.

### <a name="workflow-overview-to-enhance"></a><a id="MXF_to_multibitrate_MP4_overview"></a>Información general del flujo de trabajo a mejorar
![Flujo de trabajo a mejorar para archivos MP4 con varias velocidades de bits](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-multibitrate-mp4-workflow-to-enhance.png)

*Flujo de trabajo a mejorar para archivos MP4 con varias velocidades de bits*

### <a name="file-naming-conventions"></a><a id="MXF_to__multibitrate_MP4_file_naming"></a>Convenciones de nomenclatura de los archivos
En el flujo de trabajo anterior se especificó una expresión simple como base para generar nombres de archivo de salida. No obstante, tenemos algunas duplicaciones: todos los componentes de archivo de salida individuales especificaron dicha expresión.

Por ejemplo, el componente de salida de archivo del primer archivo de vídeo se configura con esta expresión:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_640x360_1.MP4`

Mientras que para la segunda salida de vídeo, tenemos la siguiente expresión:

`${ROOT_outputWriteDirectory}\\${ROOT_sourceFileBaseName}_960x540_2.MP4`

¿No sería más limpio, menos propenso a errores y más fácil que pudiéramos eliminar parte de esta duplicación y hacer las cosas más configurables? Afortunadamente, podemos: las funcionalidades de expresión del diseñador en combinación con la posibilidad de crear propiedades personalizadas en la raíz del flujo de trabajo proporcionará un nivel adicional de comodidad.

Supongamos que definimos la configuración de nombres de archivo a partir de las velocidades de bits de los archivos MP4 individuales. Queremos configurar estas velocidades de bits en un lugar central (en la raíz de nuestro gráfico), desde el que podrá obtener acceso para configurar e impulsar la generación de nombres de archivo. Para ello, empezaremos por publicar la propiedad de velocidad de bits de ambos codificadores AVC en la raíz de nuestro flujo de trabajo para que sea accesible desde la raíz y desde los codificadores AVC. (Incluso aunque se muestren en dos lugares diferentes, hay solo un valor subyacente).

### <a name="publishing-component-properties-onto-the-workflow-root"></a><a id="MXF_to__multibitrate_MP4_publishing"></a>Publicación de propiedades de componente en la raíz del flujo de trabajo
Abra el primer codificador AVC, vaya a la propiedad de velocidad de bits (kbps) y en la lista desplegable, elija Publicar.

![Publicación de la propiedad de velocidad de bits](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publishing-bitrate-property.png)

*Publicación de la propiedad de velocidad de bits*

Configure el cuadro de diálogo de publicación para publicar en la raíz de nuestro gráfico de flujo de trabajo, con un nombre publicado como "video1bitrate" y un nombre para mostrar legible de "Velocidad de bits de vídeo 1". Configure un grupo personalizado denominado "Velocidades de bits de streaming" y haga clic en Publish (Publicar).

![Publicación de la propiedad de velocidad de bits](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publishing-dialog-for-bitrate-property.png)

*Cuadro de diálogo de publicación para la propiedad de velocidad de bits*

Repita el mismo procedimiento con la propiedad de velocidad de bits del segundo codificador AVC y asígnele el nombre "video2bitrate" con un nombre para mostrar que sea "Velocidad de bits de vídeo 2", en el mismo grupo personalizado "Velocidades de bits de streaming".

Si ahora inspeccionamos las propiedades de la raíz del flujo de trabajo, veremos el grupo personalizado mostrando las dos propiedades publicadas. Ambas reflejan el valor de sus respectivas velocidades de bits del codificador AVC.

![Propiedades de velocidad de bits publicadas en la raíz del flujo de trabajo](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-published-bitrate-props-on-workflow-root.png)

Siempre que desee tener acceso a estas propiedades desde el código o desde una expresión, podemos hacerlo así:

* desde el código alineado desde un componente justo debajo de la raíz: node.getPropertyAsString('../video1bitrate',null)
* dentro de una expresión: ${ROOT_video1bitrate}

Vamos a completar el grupo "Velocidades de bits de streaming" publicando también la velocidad de bits de nuestra pista de audio en él. Dentro de las propiedades del codificador AAC, busque la opción Bitrate (Velocidad de bits) y seleccione Publish (Publicar) en la lista desplegable situada junto a él. Publique en la raíz del gráfico con el nombre "audio1bitrate" y el nombre para mostrar "Velocidad de bits de audio 1" dentro del grupo personalizado "Velocidades de bits de streaming".

![Cuadro de diálogo de publicación para la velocidad de bits de archivos de audio](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publishing-dialog-for-audio-bitrate.png)

*Cuadro de diálogo de publicación para la velocidad de bits de archivos de audio*

![Propiedades resultantes de audio y vídeo en la raíz](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-resulting-video-and-audio-props-on-root.png)

*Propiedades resultantes de audio y vídeo en la raíz*

Si se cambia cualquiera de estos tres valores también se volverán a configurar y a cambiar los valores de los respectivos componentes con los que están vinculados (y desde donde se han publicado).

### <a name="have-generated-output-file-names-rely-on-published-property-values"></a><a id="MXF_to__multibitrate_MP4_output_files"></a>Se han generado nombres de archivo de salida que se basan en los valores de propiedad publicados
En vez de codificar los nombres de archivo generados, es posible cambiar ahora la expresión de nombre de archivo en cada uno de los componentes de salida de archivo que se basan en las propiedades de velocidad de bits que acabamos de publicar en la raíz del grafo. A partir de nuestra primera salida de archivo, busque la propiedad de archivo y edite la expresión de la siguiente forma:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_${ROOT_video1bitrate}kbps.MP4`

Se puede obtener acceso a los distintos parámetros de esta expresión y modificarlos pulsando el signo de dólar en el teclado mientras está en la ventana de la expresión. Uno de los parámetros disponibles es la propiedad video1bitrate que hemos publicado anteriormente.

![Acceso a los parámetros de una expresión](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-accessing-parameters-within-an-expression.png)

*Acceso a los parámetros de una expresión*

Haga lo mismo para la salida de archivo del segundo vídeo:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_${ROOT_video2bitrate}kbps.MP4`

y para la salida de archivo de solo audio:

`${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_${ROOT_audio1bitrate}bps_audio.MP4`

Si ahora cambiamos la velocidad de bits de cualquiera de los archivos de audio o vídeo, se volverá a configurar el codificador respectivo y la convención de nomenclatura de archivo basada en la velocidad de bits se respetará de forma automática.

## <a name="adding-thumbnails-to-multibitrate-mp4-output"></a><a id="thumbnails_to__multibitrate_MP4"></a>Incorporación de miniaturas a la salida MP4 de varias velocidades de bits
A partir de un flujo de trabajo que genera [una salida MP4 de varias velocidades de bits a partir de una entrada MXF](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging), veremos ahora la incorporación de miniaturas a la salida.

### <a name="workflow-overview-to-add-thumbnails-to"></a><a id="thumbnails_to__multibitrate_MP4_overview"></a>Información general del flujo de trabajo al que agregar miniaturas
![Flujo de trabajo de MP4 con varias velocidades de bits con el que empezar a trabajar](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-multibitrate-mp4-workflow-to-start-from.png)

*Flujo de trabajo de MP4 con varias velocidades de bits con el que empezar a trabajar*

### <a name="adding-jpg-encoding"></a><a id="thumbnails_to__multibitrate_MP4__with_jpg"></a>Incorporación de codificación JPG
El núcleo de la generación de miniaturas será el componente de codificador JPG, capaz de generar archivos JPG.

![Codificador JPG](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-jpg-encoder.png)

*Codificador JPG*

No obstante, no podemos conectar directamente la secuencia de vídeo sin comprimir desde la entrada de archivo multimedia en el codificador JPG. En su lugar, se espera que se manejen los fotogramas individualmente. Esto lo podemos hacer mediante el componente Video Frame Gate (Puerta de fotogramas de vídeo).

![Conexión de una puerta de fotogramas con el codificador JPG](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connect-frame-gate-to-jpg-encoder.png)

*Conexión de una puerta de fotogramas con el codificador JPG*

La puerta de fotogramas permite pasar un fotograma de vídeo cada cierto número de segundos o fotogramas. El intervalo y la frecuencia con que esto ocurre es configurable en las propiedades.

![Propiedades de puerta de fotogramas de vídeo](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-video-frame-gate-properties.png)

*Propiedades de puerta de fotogramas de vídeo*

Vamos a crear una miniatura cada minuto estableciendo el modo en Time (seconds) (Tiempo (segundos)) y la opción Interval (Intervalo) en 60.

### <a name="dealing-with-color-space-conversion"></a><a id="thumbnails_to__multibitrate_MP4_color_space"></a>Relación con la conversión de espacio de colores
Aunque podría parecer lógico que ahora se puedan conectar las dos clavijas de vídeo sin comprimir de la puerta de fotogramas y la entrada del archivo multimedia, obtendríamos una advertencia si lo hacemos.

![Error de espacio de colores de entrada](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-input-color-space-error.png)

*Error de espacio de colores de entrada*

Esto se debe a que la forma en que se representa la información de color en nuestra secuencia de vídeo original sin comprimir y sin procesar procedente de nuestro MXF, es diferente de la que espera el codificador JPG. Más concretamente, se espera un "espacio de colores" denominado "RGB" o "Escala de grises". Esto significa que será necesario aplicar primero una conversión relacionada con el espacio de colores a la secuencia de vídeo entrante de la puerta de fotogramas de vídeo.

Arrastre al flujo de trabajo el componente Color Space Converter - Intel (Convertidor de espacio de colores - Intel) y conéctelo a la puerta de fotogramas.

![Conexión con un convertidor de espacio de colores](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-connect-color-space-convertor.png)

*Conexión con un convertidor de espacio de colores*

En la ventana de propiedades, seleccione la entrada BGR 24 de la lista de valores preestablecidos.

### <a name="writing-the-thumbnails"></a><a id="thumbnails_to__multibitrate_MP4_writing_thumbnails"></a>Escritura de las miniaturas
A diferencia del vídeo MP4, el componente de codificador JPG genera más de un archivo. Para afrontar esto, se puede usar el componente Scene Search JPG File Writer (Escritura de archivos JPG de búsqueda de escenas), que toma las miniaturas JPG entrantes y escribe en ellas, y proporciona un sufijo para cada nombre de archivo mediante un número diferente. (Este número indica normalmente el número de segundos o unidades en la secuencia de la que se ha extraído la miniatura).

![Presentación de la escritura de archivos JPG de búsqueda de escenas](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scene-search-jpg-file-writer.png)

*Presentación de la escritura de archivos JPG de búsqueda de escenas*

Configure la propiedad de ruta de acceso de la carpeta de salida con la expresión: `${ROOT_outputWriteDirectory}`

y la propiedad del prefijo del nombre de archivo con:

`${ROOT_sourceFileBaseName}_thumb_`

El prefijo determina cómo se denominan los archivos de miniaturas. Se les agrega como sufijo un número que indica la posición de la miniatura en la secuencia.

![Propiedades de la escritura de archivos JPG de búsqueda de escenas](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scene-search-jpg-file-writer-properties.png)

*Propiedades de la escritura de archivos JPG de búsqueda de escenas*

Conecte el componente de escritura de archivos JPG de búsqueda de escenas en el nodo Output File/Asset (Recurso/Archivo de salida).

### <a name="detecting-errors-in-a-workflow"></a><a id="thumbnails_to__multibitrate_MP4_errors"></a>Detección de errores en un flujo de trabajo
Conecte la entrada del convertidor de espacio de colores a la salida de vídeo sin comprimir y sin procesar. Ahora, ejecute una prueba del flujo de trabajo de forma local. Es muy probable que el flujo de trabajo deje de ejecutarse repentinamente e indique con un contorno rojo el componente en que se encontró un error:

![Error de convertidor de espacio de colores](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-color-space-converter-error.png)

*Error de convertidor de espacio de colores*

Haga clic en el pequeño icono rojo "E" en la esquina superior derecha del componente del convertidor de espacio de colores para ver el motivo por el que el intento de codificación dio error.

![Cuadro de diálogo de error del convertidor de espacio de colores](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-color-space-converter-error-dialog.png)

*Cuadro de diálogo de error del convertidor de espacio de colores*

Como puede observar, el espacio de colores estándar entrante para el convertidor tiene que ser rec601 para la conversión que solicitamos de YUV a RGB. Aparentemente, nuestra secuencia no indica que fuera rec601. (Rec 601 es un estándar para codificar señales de vídeo analógicas entrelazadas en formato de vídeo digital. Especifica una región activa que abarcan 720 muestras de luminancia y 360 muestras cromáticas por línea. El sistema de codificación de colores se conoce como YCbCr 4:2:2).

Para solucionar este problema, indicaremos en los metadatos de la secuencia que estamos trabajando con contenido de rec601. Para ello, utilizaremos el componente Video Data Type Updater (Actualizador de tipo de datos de vídeo) y lo colocaremos entre el origen sin formato y el componente de conversión de espacio de colores. Este actualizador permite la actualización manual de determinadas propiedades de tipos de datos de vídeo. Configúrelo para que indique un espacio de colores estándar "Rec 601". Esto hace que el actualizador de tipo de datos de vídeo etiquete la secuencia con el espacio de colores "Rec 601" si no hay ningún espacio de colores definido todavía. (No se invalidarán los metadatos existentes, a menos que se active la casilla Override (Invalidar)).

![Actualización del espacio de colores estándar en el actualizador de tipo de datos](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-update-color-space-standard-on-data-type.png)

*Actualización del espacio de colores estándar en el actualizador de tipo de datos*

### <a name="finished-workflow"></a><a id="thumbnails_to__multibitrate_MP4_finish"></a>Flujo de trabajo finalizado
Ahora que el flujo de trabajo está acabado, realice otra prueba para comprobar su ejecución.

![Flujo de trabajo terminado para varias salidas MP4 con miniaturas](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-workflow-for-multi-mp4-thumbnails.png)

*Flujo de trabajo terminado para varias salidas MP4 con miniaturas*

## <a name="time-based-trimming-of-multibitrate-mp4-output"></a><a id="time_based_trim"></a>Recorte basado en tiempo de salida MP4 de varias velocidades de bits
A partir de un flujo de trabajo que genera [una salida MP4 de varias velocidades de bits a partir de una entrada MXF](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging), veremos ahora el recorte del vídeo de origen basado en marcas de tiempo.

### <a name="workflow-overview-to-start-adding-trimming-to"></a><a id="time_based_trim_start"></a>Información general del flujo de trabajo al que empezar a agregar el recorte
![Flujo de trabajo inicial al que agregar el recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-starting-workflow-to-add-trimming.png)

*Flujo de trabajo inicial al que agregar el recorte*

### <a name="using-the-stream-trimmer"></a><a id="time_based_trim_use_stream_trimmer"></a>Utilización del recortador de transmisiones
El componente Stream Trimmer (Recortador de secuencias) permite recortar el principio y el final de una secuencia de entrada según la información de tiempo (segundos, minutos, etc.). El recortador no admite el recorte basado en fotogramas.

![Recortador de transmisiones](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-stream-trimmer.png)

*Recortador de transmisiones*

En lugar de vincular directamente los codificadores AVC y el asignador de posiciones de altavoz con la entrada de archivo multimedia, pondremos el recortador de transmisiones entre ellos. (Uno para la señal de vídeo y otro para la señal de audio entrelazada).

![Coloque el recortador de transmisiones entre ellos](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-put-stream-trimmer-in-between.png)

*Coloque el recortador de transmisiones entre ellos*

Vamos a configurar el recortador para que solo se procese el vídeo y el audio entre los segundos 15 y 60 del vídeo.

Vaya a las propiedades del recortador de secuencias de vídeo y configure las propiedades de tiempo de inicio (15 seg.) y las de tiempo de finalización (60 seg.). Para asegurarse de que tanto el recortador de audio como el de vídeo siempre están configurados con los mismos valores de inicio y finalización, dichos valores se publican en la raíz del flujo de trabajo.

![Publique la propiedad de tiempo de inicio del recortador de transmisiones](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publish-start-time-from-stream-trimmer.png)

*Publique la propiedad de tiempo de inicio del recortador de transmisiones*

![Cuadro de diálogo de propiedades de publicación para el tiempo de inicio](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publish-dialog-for-start-time.png)

*Cuadro de diálogo de propiedades de publicación para el tiempo de inicio*

![Cuadro de diálogo de propiedades de publicación para el tiempo de finalización](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-publish-dialog-for-end-time.png)

*Cuadro de diálogo de propiedades de publicación para el tiempo de finalización*

Si ahora inspeccionamos la raíz de nuestro flujo de trabajo, ambas propiedades se muestran claramente y se pueden configurar desde allí.

![Propiedades publicadas disponibles en la raíz](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-published-properties-available-on-root.png)

*Propiedades publicadas disponibles en la raíz*

Ahora, abra las propiedades de recorte del recortador de audio y configure los tiempos de inicio y finalización con una expresión que haga referencia a las propiedades publicadas en la raíz de nuestro flujo de trabajo.

Para el tiempo de inicio del recorte de audio:

`${ROOT_TrimmingStartTime}`

y para el tiempo de finalización:

`${ROOT_TrimmingEndTime}`

### <a name="finished-workflow"></a><a id="time_based_trim_finish"></a>Flujo de trabajo finalizado
![Flujo de trabajo finalizado](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-finished-workflow-time-base-trimming.png)

*Flujo de trabajo finalizado*

## <a name="introducing-the-scripted-component"></a><a id="scripting"></a>Introducción al componente generado por script
Los componentes generados por script pueden ejecutar scripts arbitrarios durante las fases de ejecución de nuestro flujo de trabajo. Hay cuatro scripts diferentes que se pueden ejecutar, cada uno con características específicas y su propio lugar en el ciclo de vida del flujo de trabajo:

* **commandScript**
* **realizeScript**
* **processInputScript**
* **lifeCycleScript**

La documentación del componente generado por script ofrece más detalles de cada uno de los scripts anteriores. En [la siguiente sección](media-services-media-encoder-premium-workflow-tutorials.md#frame_based_trim), el componente generado por script **realizeScript** se utiliza para generar un archivo xml de la lista de clips directamente cuando se inicia el flujo de trabajo. Se llama a este script durante la configuración del componente, lo que sucede una sola vez en su ciclo de vida.

### <a name="scripting-within-a-workflow-hello-world"></a><a id="scripting_hello_world"></a>Scripting en un flujo de trabajo: hola a todos
Arrastre un componente generado por script a la superficie del diseñador y cámbiele el nombre (por ejemplo, "SetClipListXML").

![Incorporación de un componente generado por script](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-add-scripted-comp.png)

*Incorporación de un componente generado por script*

Cuando inspeccione las propiedades del componente generado por script, se mostrarán los cuatro tipos diferentes de scripts y cada uno de ellos podrá configurarse con un script diferente.

![Propiedades de componente generado por script](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scripted-comp-properties.png)

*Propiedades de componente generado por script*

Borre el script processInputScript y abra el editor del script realizeScript. Ahora ya estamos listos para iniciar el scripting.

Los scripts se escriben en Groovy, un lenguaje de scripting compilado dinámicamente para la plataforma Java que conserva la compatibilidad con Java. En realidad, la mayoría del código Java es también código Groovy válido.

Vamos a escribir un script de Groovy simple de “hola a todos” en el contexto de nuestro script realizeScript. En el editor, escriba lo siguiente:

`node.log("hello world");`

A continuación, ejecute una prueba de forma local. Después de esta ejecución, inspeccione la propiedad Logs (Registros) (a través de la pestaña System (Sistema) en el componente generado por scripts.

![Salida de registro Hola a todos](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-log-output.png)

*Salida de registro Hola a todos*

El objeto de nodo en el que llamamos al método de registro, hace referencia a nuestro "nodo" actual o al componente en el que estamos haciendo scripting. Todos los componentes tienen por sí mismos la capacidad de generar datos de registro que están disponibles a través de la pestaña de sistema. En este caso, generamos el literal de cadena "hola, mundo". Es importante comprender aquí que esto puede resultar una valiosa herramienta de depuración, que le proporciona información sobre lo que realmente está haciendo el script.

Desde dentro del entorno de scripting, también tenemos acceso a las propiedades de otros componentes. Pruebe esto:

```java
    //inspect current node:
    def nodepath = node.getNodePath();
    node.log("this node path: " + nodepath);

    //walking up to other nodes:
    def parentnode = node.getParentNode();
    def parentnodepath = parentnode.getNodePath();
    node.log("parent node path: " + parentnodepath);

    //read properties from a node:
    def sourceFileExt = parentnode.getPropertyAsString( "sourceFileExtension", null );
    def sourceFileName = parentnode.getPropertyAsString("sourceFileBaseName", null);
    node.log("source file name with extension " + sourceFileExt + " is: " + sourceFileName);
```

Nuestra ventana de registro nos muestra lo siguiente:

![Salida de registro para obtener acceso a las rutas de acceso del nodo](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-log-output2.png)

*Salida de registro para obtener acceso a las rutas de acceso del nodo*

## <a name="frame-based-trimming-of-multibitrate-mp4-output"></a><a id="frame_based_trim"></a>Recorte basado en fotogramas de salida MP4 de varias velocidades de bits
A partir de un flujo de trabajo que genera [una salida MP4 de varias velocidades de bits a partir de una entrada MXF](media-services-media-encoder-premium-workflow-tutorials.md#MXF_to_MP4_with_dyn_packaging), veremos ahora el recorte del vídeo de origen basado en el recuento de fotogramas.

### <a name="blueprint-overview-to-start-adding-trimming-to"></a><a id="frame_based_trim_start"></a>Información general del plano al que empezar a agregar el recorte
![Flujo de trabajo al que empezar a agregar el recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-workflow-start-adding-trimming-to.png)

*Flujo de trabajo al que empezar a agregar el recorte*

### <a name="using-the-clip-list-xml"></a><a id="frame_based_trim_clip_list"></a>Utilización del XML de la lista de clips
En todos los tutoriales sobre flujos de trabajo anteriores, hemos utilizado el componente de entrada de archivo multimedia como origen de entrada del vídeo. No obstante, para este escenario concreto, vamos a utilizar el componente Clip List Source (Origen de lista de clips) en su lugar. Esta no debería ser la forma preferida de trabajar. Use únicamente el origen de lista de clips cuando haya un motivo real para hacerlo (como en el siguiente caso, donde estamos haciendo uso de las funcionalidades de recorte de la lista de clips).

Para cambiar de nuestra entrada de archivo multimedia al origen de lista de clips, arrastre este último componente a la superficie de diseño y conecte la clavija Clip List XML (XML de la lista de clips) con el nodo correspondiente del diseñador de flujo de trabajo. Esto rellena el origen de la lista de clips con clavijas de salida según nuestro vídeo de entrada. Conecte ahora las clavijas de vídeo y audio sin comprimir desde el origen de la lista de clips a los respectivos codificadores AVC y al entrelazador de secuencias de audio. Quite ahora la entrada de archivo multimedia.

![Reemplace la entrada de archivo multimedia con el origen de la lista de clips](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-replaced-media-file-with-clip-source.png)

*Reemplace la entrada de archivo multimedia con el origen de la lista de clips*

El componente de origen de la lista de clips toma como entrada un "XML de la lista de clips". Al seleccionar el archivo de origen con el que realizar pruebas localmente, este xml de la lista de clips se rellenará automáticamente.

![Propiedad del XML de la lista de clips rellenado de forma automática](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-auto-populated-clip-list-xml-property.png)

*Propiedad del XML de la lista de clips rellenado de forma automática*

Si examinamos un poco más detalladamente el xml, este es su aspecto:

![Cuadro de diálogo Edit clip list (Editar lista de clips)](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-edit-clip-list-dialog.png)

*Cuadro de diálogo Edit clip list (Editar lista de clips)*

Esto no obstante no refleja las funcionalidades del xml de la lista de clips. Una opción que tenemos es agregar un elemento de "recorte" en el origen de audio y en el de vídeo, similar al siguiente:

![Incorporación de un elemento de recorte a la lista de clips](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-adding-trim-element-to-clip-list.png)

*Incorporación de un elemento de recorte a la lista de clips*

Si modifica el xml de la lista de clips de la forma anterior y ejecuta una prueba local, verá que se han recortado correctamente del vídeo entre 10 y 20 segundos.

No obstante, al contrario de lo que ocurre cuando realiza una ejecución local, este mismo xml de la lista de clips no tendría el mismo efecto que cuando se aplica en un flujo de trabajo que se ejecute en Azure Media Services. Cuando se inicia el Codificador premium de Azure, se genera de nuevo el xml de la lista de clips según el archivo de entrada que se ha asignado al trabajo de codificación. Esto significa que todos los cambios que realicemos en el xml se reemplazarán.

Para contrarrestar el borrado del xml de la lista de clips al iniciar un trabajo de codificación, lo podemos volver a generar sobre la marcha justo después del inicio del flujo de trabajo. Tales acciones personalizadas se pueden realizar mediante lo que se denomina un "componente generado por script". Para más información, consulte la sección [Introducción al componente generado por script](media-services-media-encoder-premium-workflow-tutorials.md#scripting).

Arrastre un componente generado por script a la superficie del diseñador y cámbiele el nombre a "SetClipListXML".

![Incorporación de un componente generado por script](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-add-scripted-comp.png)

*Incorporación de un componente generado por script*

Al inspeccionar las propiedades del componente generado por script, se muestran los cuatro tipos diferentes de scripts y cada uno de ellos podrá configurarse con un script diferente.

![Propiedades de componente generado por script](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-scripted-comp-properties.png)

*Propiedades de componente generado por script*

### <a name="modifying-the-clip-list-from-a-scripted-component"></a><a id="frame_based_trim_modify_clip_list"></a>Modificación de la lista de clips desde un componente generado por script
Antes de que podamos volver a escribir el xml de la lista de clips que se genera durante el inicio del flujo de trabajo, necesitaremos tener acceso a la propiedad y al contenido del xml de la lista de clips. Para ello podemos hacer lo siguiente:

```java
    // get cliplist xml:
    def clipListXML = node.getProperty("../clipListXml");
    node.log("clip list xml coming in: " + clipListXML);
```

![Lista de clips entrantes que se están registrando](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-incoming-clip-list-logged.png)

*Lista de clips entrantes que se están registrando*

En primer lugar, necesitamos una manera de determinar desde dónde y hasta dónde desea recortar el vídeo. Para facilitar esto al usuario menos técnico del flujo de trabajo, publique dos propiedades en la raíz del gráfico. Para ello, haga clic con el botón derecho en la superficie del diseñador y seleccione "Add property" (Agregar propiedad):

* Primera propiedad: "ClippingTimeStart" de tipo: "TIMECODE"
* Segunda propiedad: "ClippingTimeEnd" de tipo: "TIMECODE"

![Cuadro de diálogo Add Property (Agregar propiedad) para el tiempo de inicio del recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-clip-start-time.png)

*Cuadro de diálogo Add Property (Agregar propiedad) para el tiempo de inicio del recorte*

![Propiedades del tiempo de recorte publicadas en la raíz del flujo de trabajo](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-clip-time-props.png)

*Propiedades del tiempo de recorte publicadas en la raíz del flujo de trabajo*

Configure ambas propiedades con un valor adecuado:

![Configuración de las propiedades de inicio y finalización del recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-configure-clip-start-end-prop.png)

*Configuración de las propiedades de inicio y finalización del recorte*

Ahora, desde dentro del script, podemos acceder a ambas propiedades, de la forma siguiente:

```java
    // get start and end of clipping:
    def clipstart = node.getProperty("../ClippingTimeStart").toString();
    def clipend = node.getProperty("../ClippingTimeEnd").toString();

    node.log("clipping start: " + clipstart);
    node.log("clipping end: " + clipend);
```

![Ventana de registro que muestra el inicio y fin del recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-show-start-end-clip.png)

*Ventana de registro que muestra el inicio y fin del recorte*

Vamos a redistribuir las cadenas de código de tiempo de una forma más fácil de utilizar, mediante una expresión regular simple:

```java
    //parse the start timing:
    def startregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipstart);
    startregresult.matches();
    def starttimecode = startregresult.group(1);
    node.log("timecode start is: " + starttimecode);
    def startframerate = startregresult.group(2);
    node.log("framerate start is: " + startframerate);

    //parse the end timing:
    def endregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipend);
    endregresult.matches();
    def endtimecode = endregresult.group(1);
    node.log("timecode end is: " + endtimecode);
    def endframerate = endregresult.group(2);
    node.log("framerate end is: " + endframerate);
```

![Ventana de registro con salida de código de tiempo redistribuida](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-output-parsed-timecode.png)

*Ventana de registro con salida de código de tiempo redistribuida*

Con esta información a mano, ahora podemos modificar el xml de la lista de clips para reflejar los tiempos de inicio y finalización para el recorte del fotograma adecuado de la película.

![Código de script para agregar elementos de recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-add-trim-elements.png)

*Código de script para agregar elementos de recorte*

Esto se realizó mediante operaciones de manipulación de cadenas normales. El xml modificado resultante de la lista de clips se vuelve a escribir en la propiedad clipListXML en la raíz del flujo de trabajo a través del método "setProperty". La ventana de registro debería mostrar lo siguiente después de ejecutar otra prueba:

![Registro de la lista resultante de clips](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-log-result-clip-list.png)

*Registro de la lista resultante de clips*

Realice una ejecución de prueba para ver cómo se han recortado las secuencias de audio y vídeo. Como llevará a cabo la ejecución de más de una prueba con valores diferentes para los puntos de recorte, observará que, sin embargo, estos no se tienen en cuenta. La razón de esto es que el diseñador, a diferencia del tiempo de ejecución de Azure, NO reemplaza el xml de la lista de clips en cada ejecución. Esto significa que solo se transformará el XML la primera vez que se hayan configurado los puntos de inicio y de finalización; el resto de las veces, nuestra cláusula de restricción (if(`clipListXML.indexOf("<trim>") == -1`)) impedirá que el flujo de trabajo agregue otro elemento de recorte cuando ya haya uno presente.

Para facilitar que el flujo de trabajo realice la prueba localmente, es conveniente agregar algún código de mantenimiento interno que inspeccione si ya existe un elemento de recorte. Si es así, podemos quitarlo antes de continuar, modificando el xml con los nuevos valores. En lugar de usar manipulaciones de cadenas simples, probablemente es más seguro hacerlo mediante la redistribución de modelos de objeto xml reales.

Sin embargo, antes de que podamos agregar dicho código, primero necesitaremos agregar un número de instrucciones de importación al principio del script:

```java
    import javax.xml.parsers.*;
    import org.xml.sax.*;
    import org.w3c.dom.*;
    import javax.xml.*;
    import javax.xml.xpath.*;
    import javax.xml.transform.*;
    import javax.xml.transform.stream.*;
    import javax.xml.transform.dom.*;
```

Después de esto, podemos agregar el código de limpieza necesario:

```java
    //for local testing: delete any pre-existing trim elements from the clip list xml by parsing the xml into a DOM:
    DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
    DocumentBuilder builder=factory.newDocumentBuilder();
    InputSource is=new InputSource(new StringReader(clipListXML));
    Document dom=builder.parse(is);

    //find the trim element inside videoSource and audioSource and remove it if it exists already:
    XPath xpath = XPathFactory.newInstance().newXPath();
    String findAllTrimElements = "//trim";
    NodeList trimelems = xpath.evaluate(findAllTrimElements,dom,XPathConstants.NODESET);

    //copy trim nodes into a "to-be-deleted" collection
    Set<Element> elementsToDelete = new HashSet<Element>();
    for (int i = 0; i < trimelems.getLength(); i++) {
        Element e = (Element)trimelems.item(i);
        elementsToDelete.add(e);
    }

    node.log("about to delete any existing trim nodes");
     //delete the trim nodes:
    elementsToDelete.each{
        e -> e.getParentNode().removeChild(e);
    };
    node.log("deleted any existing trim nodes");

    //serialize the modified clip list xml dom into a string:
    def transformer = TransformerFactory.newInstance().newTransformer();
    StreamResult result = new StreamResult(new StringWriter());
    DOMSource source = new DOMSource(dom);
    transformer.transform(source, result);
    clipListXML = result.getWriter().toString();
```

Este código va justo sobre el punto en el que agregamos el elemento de recorte al XML de la lista de clips.

En este punto, podemos ejecutar y modificar el flujo de trabajo tanto como sea necesario y al mismo tiempo lograr que los cambios se apliquen siempre.    

### <a name="adding-a-clippingenabled-convenience-property"></a><a id="frame_based_trim_clippingenabled_prop"></a>Incorporación de una propiedad de conveniencia ClippingEnabled
Como puede que no siempre desee realizar un recorte, vamos a terminar nuestro flujo de trabajo agregando una sencilla marca booleana que indica si queremos habilitar o no el recorte.

Al igual que antes, publique una nueva propiedad en la raíz del flujo de trabajo denominada "ClippingEnabled" de tipo "BOOLEAN".

![Publicación de una propiedad para habilitar el recorte](./media/media-services-media-encoder-premium-workflow-tutorials/media-services-enable-clip.png)

*Publicación de una propiedad para habilitar el recorte*

Con la siguiente cláusula de restricción simple, podemos comprobar si es necesario el recorte y decidir si nuestra lista de clips como tal debe modificarse o no.

```java
    //check if clipping is required:
    def clippingrequired = node.getProperty("../ClippingEnabled");
    node.log("clipping required: " + clippingrequired.toString());
    if(clippingrequired == null || clippingrequired == false)
    {
        node.setProperty("../clipListXml",clipListXML);
        node.log("no clipping required");
        return;
    }
```

### <a name="complete-code"></a><a id="code"></a>Código completo

```java
    import javax.xml.parsers.*;
    import org.xml.sax.*;
    import org.w3c.dom.*;
    import javax.xml.*;
    import javax.xml.xpath.*;
    import javax.xml.transform.*;
    import javax.xml.transform.stream.*;
    import javax.xml.transform.dom.*;

    // get cliplist xml:
    def clipListXML = node.getProperty("../clipListXml");
    node.log("clip list xml coming in: \n" + clipListXML);
    // get start and end of clipping:
    def clipstart = node.getProperty("../ClippingTimeStart").toString();
    def clipend = node.getProperty("../ClippingTimeEnd").toString();

    //parse the start timing:
    def startregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipstart);
    startregresult.matches();
    def starttimecode = startregresult.group(1);
    node.log("timecode start is: " + starttimecode);
    def startframerate = startregresult.group(2);
    node.log("framerate start is: " + startframerate);

    //parse the end timing:
    def endregresult = (~/(\d\d:\d\d:\d\d:\d\d)\/(\d\d)/).matcher(clipend);
    endregresult.matches();
    def endtimecode = endregresult.group(1);
    node.log("timecode end is: " + endtimecode);
    def endframerate = endregresult.group(2);

    node.log("framerate end is: " + endframerate);

    //for local testing: delete any pre-existing trim elements
    //from the clip list xml by parsing the xml into a DOM:

    DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
    DocumentBuilder builder=factory.newDocumentBuilder();
    InputSource is=new InputSource(new StringReader(clipListXML));
    Document dom=builder.parse(is);

    //find the trim element inside videoSource and audioSource and remove it if it exists already:
    XPath xpath = XPathFactory.newInstance().newXPath();
    String findAllTrimElements = "//trim";
    NodeList trimelems = xpath.evaluate(findAllTrimElements, dom, XPathConstants.NODESET);

    //copy trim nodes into a "to-be-deleted" collection
    Set<Element> elementsToDelete = new HashSet<Element>();
    for (int i = 0; i < trimelems.getLength(); i++) {
        Element e = (Element)trimelems.item(i);
        elementsToDelete.add(e);
    }

    node.log("about to delete any existing trim nodes");
    //delete the trim nodes:
    elementsToDelete.each{ e ->
        e.getParentNode().removeChild(e);
    };
    node.log("deleted any existing trim nodes");

    //serialize the modified clip list xml dom into a string:
    def transformer = TransformerFactory.newInstance().newTransformer();
    StreamResult result = new StreamResult(new StringWriter());
    DOMSource source = new DOMSource(dom);
    transformer.transform(source, result);
    clipListXML = result.getWriter().toString();

    //check if clipping is required:
    def clippingrequired = node.getProperty("../ClippingEnabled");
    node.log("clipping required: " + clippingrequired.toString());
    if(clippingrequired == null || clippingrequired == false)
    {
        node.setProperty("../clipListXml",clipListXML);
        node.log("no clipping required");
        return;
    }

    //add trim elements to cliplist xml
    if ( clipListXML.indexOf("<trim>") == -1 )
    {
        //trim video
        clipListXML = clipListXML.replace("<videoSource>","<videoSource>\n <trim>\n <inPoint fps=\""+
            startframerate +"\">" + starttimecode +
            "</inPoint>\n" + "<outPoint fps=\"" + endframerate +"\"> " + endtimecode +
            " </outPoint>\n </trim> \n");
        //trim audio
        clipListXML = clipListXML.replace("<audioSource>","<audioSource>\n <trim>\n <inPoint fps=\""+
            startframerate +"\">" + starttimecode +
            "</inPoint>\n" + "<outPoint fps=\""+ endframerate +"\">" +
            endtimecode + "</outPoint>\n </trim>\n");
        node.log( "clip list going out: \n" +clipListXML );
        node.setProperty("../clipListXml",clipListXML);
    }
```

## <a name="also-see"></a>Consulte también:
[Introducción de la codificación Premium en Azure Media Services](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services)

[Uso de la codificación Premium en Azure Media Services](https://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services)

[Codificación de contenido a petición con Azure Media Services](media-services-encode-asset.md#media-encoder-premium-workflow)

[Códecs y formatos de flujo de trabajo del Codificador multimedia Premium](media-services-premium-workflow-encoder-formats.md)

[Archivos de flujo de trabajo de ejemplo](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows)

[Explorador de Azure Media Services](https://aka.ms/amse)

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
