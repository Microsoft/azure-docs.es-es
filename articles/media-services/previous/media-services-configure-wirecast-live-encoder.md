---
title: Uso del codificador Telestream Wirecast para enviar una transmisión en vivo con velocidad de bits única | Microsoft Docs
description: 'En este tema se muestra cómo configurar el codificador en directo Wirecast para enviar una transmisión con velocidad de bits única a canales AMS habilitados para la codificación en directo. '
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 0d2f1e81-51a6-4ca9-894a-6dfa51ce4c70
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.reviewer: cenkdin;anilmur
ms.openlocfilehash: 7b11d7f577f0a2ceb7284d9f78ccf83a64c72fd3
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89258119"
---
# <a name="use-the-wirecast-encoder-to-send-a-single-bitrate-live-stream"></a>Uso del codificador Wirecast para enviar una transmisión por secuencias en directo de velocidad de bits única

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
> [!div class="op_single_selector"]
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)
>

En este artículo se muestra cómo configurar el codificador en directo [Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm) para enviar una transmisión con velocidad de bits única a canales AMS habilitados para la codificación en directo. Para obtener más información, consulte [Uso de canales habilitados para Live Encoding con Azure Media Services](media-services-manage-live-encoder-enabled-channels.md).

En este tutorial se muestra cómo administrar Azure Media Services (AMS) con la herramienta Explorador de Azure Media Services (AMSE). Esta herramienta solo se ejecuta en Windows PC. Si se encuentra en Mac o Linux, use Azure Portal para crear [canales](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel) y [programas](media-services-portal-creating-live-encoder-enabled-channel.md).

> [!NOTE]
> Los codificadores deben admitir TLS 1.2 al usar protocolos RTMP. Use la versión de Wirecast 13.0.2 o posterior debido al requisito de TLS 1.2.

## <a name="prerequisites"></a>Requisitos previos
* [Creación de una cuenta de Azure Media Services](media-services-portal-create-account.md)
* Asegúrese de que hay un punto de conexión de streaming en ejecución. Para obtener más información, consulte [Administración de extremos de streaming en una cuenta de Media Services](media-services-portal-manage-streaming-endpoints.md)
* Debe instalar la última versión de la herramienta [AMSE](https://github.com/Azure/Azure-Media-Services-Explorer) .
* Inicie la herramienta y conéctese a la cuenta de AMS.

## <a name="tips"></a>Sugerencias
* Siempre que sea posible, use una conexión a Internet por cable.
* Una buena regla general al determinar los requisitos de ancho de banda consiste en duplicar las velocidades de bits de streaming. Aunque no se trata de un requisito obligatorio, contribuye a mitigar el impacto de la congestión de la red.
* Cuando se usen codificadores por software, cierre todos los programas innecesarios.

## <a name="create-a-channel"></a>Crear un canal
1. En la herramienta AMSE, navegue a la pestaña **Live** (Live) y haga clic con el botón derecho dentro del área de canales. Seleccione **Crear canal...** en el menú.

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast1.png)

2. Especifique un nombre de canal (el campo de descripción es opcional). En Configuración de canal, seleccione **Estándar** para la opción Live Encoding, con el protocolo de entrada establecido en **RTMP**. Puede dejar todas las demás opciones como están.

    Asegúrese de que la opción **Iniciar el nuevo canal ahora** esté seleccionada.

3. Haga clic en **Crear canal**.

   ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast2.png)

> [!NOTE]
> El canal puede tardar hasta 20 minutos en iniciarse.
>
>

Mientras se inicia el canal puede [configurar el codificador](media-services-configure-wirecast-live-encoder.md#configure_wirecast_rtmp).

> [!IMPORTANT]
> La facturación comienza tan pronto como el canal entra en un estado de preparado. Para obtener más información, consulte [Estados del canal](media-services-manage-live-encoder-enabled-channels.md#states).
>
>

## <a name="configure-the-telestream-wirecast-encoder"></a><a id="configure_wirecast_rtmp" />Configuración del codificador Telestream Wirecast
En este tutorial se usa la siguiente configuración de salida. En el resto de esta sección se describen los pasos de configuración con más detalle.

**Vídeo**:

* Códec: H.264
* Perfil: alto (nivel 4.0)
* Velocidad de bits: 5000 Kbps
* Fotograma clave: 2 segundos (60 segundos)
* Velocidad de fotogramas: 30

**Audio**:

* Códec: AAC (LC)
* Velocidad de bits: 192 Kbps
* Velocidad de muestreo: 44,1 kHz

### <a name="configuration-steps"></a>Pasos de configuración
1. Abra la aplicación de Telestream Wirecast en el equipo que está usando y configúrela para el streaming de RTMP.
2. Para configurar la salida, vaya a la pestaña **Output** (Salida) y seleccione **Output Settings** (Configuración de salida).

    Asegúrese de que **Output Destination** (Destino de salida) esté establecido en **RTMP Server** (Servidor RTMP).
3. Haga clic en **OK**.
4. En la página de configuración, establezca el campo **Destination** (Destino) en **Azure Media Services**.

    El perfil de Encoding está previamente seleccionado como **Azure H.264 720 p 16:9 (1280 x 720)** . Para personalizar esta configuración, seleccione el icono de engranaje a la derecha de la lista desplegable y luego elija **New Preset**(Nuevo valor preestablecido).

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast3.png)
5. Configure los valores preestablecidos del codificador.

    Asigne un nombre al valor preestablecido y busque la siguiente configuración recomendada:

    **Vídeo**

   * Codificador: MainConcept H.264
   * Fotogramas por segundo: 30
   * Velocidad de bits media: 5000 kbits/s (se puede ajustar en función de las limitaciones de red)
   * Perfil: Principal
   * Fotograma clave cada: 60 fotogramas

     **Audio**

   * Velocidad de bits de destino: 192 kbits/s
   * Velocidad de muestreo: 44,100 kHz

     ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast4.png)
6. Presione **Save**(Guardar).

    El campo de codificación tiene ahora el perfil recién creado disponible para la selección.

    Asegúrese de que esté seleccionado el nuevo perfil.
7. Obtenga la dirección URL de entrada del canal para asignarla al **punto de conexión de RTMP**de Wirecast.

    Navegue de nuevo a la herramienta AMSE y compruebe el estado de finalización del canal. Una vez que ha cambiado el estado de **Iniciando** a **En ejecución**, puede obtener la dirección URL de entrada.

    Mientras se ejecuta el canal, haga clic con el botón derecho en el nombre del canal, desplácese hacia abajo y mantenga el puntero sobre **Copy Input URL to clipboard** (Copiar dirección URL de entrada en el Portapapeles) y seleccione **Primary Input URL** (Dirección URL de entrada principal).  

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast6.png)
8. En la ventana **Output Settings** (Configuración de salida) de Wirecast, pegue esta información en el campo **Address** (Dirección) de la sección de salida y asigne un nombre de transmisión.

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast5.png)

1. Seleccione **Aceptar**.
2. En la pantalla principal de **Wirecast**, confirme que los orígenes de audio y vídeo están listos y haga clic en **Stream** (Transmitir) en la esquina superior izquierda.

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast7.png)

> [!IMPORTANT]
> Antes de hacer clic en **Stream** (Transmitir), **debe** asegurarse de que el canal esté listo.
> Además, asegúrese de no dejar el canal en un estado Listo sin una fuente de contribución de entrada durante más de 15 minutos.
>
>

## <a name="test-playback"></a>Prueba de reproducción

Vaya a la herramienta AMSE y haga clic con el botón derecho en el canal que se va a probar. En el menú, mantenga el puntero sobre **Playback the Preview** (Reproducir la vista previa) y seleccione **with Azure Media Player** (con Azure Media Player).  

![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast8.png)

Si la transmisión aparece en el reproductor, entonces el codificador se configuró correctamente para conectarse a AMS.

Si se recibe un error, debe restablecer el canal y ajustar la configuración del codificador. Para más información, consulte el artículo de [solución de problemas](media-services-troubleshooting-live-streaming.md).  

## <a name="create-a-program"></a>Creación de un programa
1. Una vez confirmada la reproducción de canales, cree un programa. En la pestaña **Live** (Directo) de la herramienta AMSE, haga clic con el botón derecho dentro del área de programas y seleccione **Create New Program** (Crear programa).  

    ![Wirecast](./media/media-services-wirecast-live-encoder/media-services-wirecast9.png)
2. Dé nombre al programa y, si es necesario, ajuste el valor de **Archive Window Length** (Duración de la ventana de archivo), que de forma predeterminada es cuatro horas. También puede especificar una ubicación de almacenamiento o dejar el valor predeterminado.  
3. Active la casilla **Iniciar el programa ahora** .
4. Haga clic en **Crear programa**.  

   >[!NOTE]
   >La creación de programas tarda menos que la creación de canales.
       
5. Cuando el programa esté en ejecución, confirme la reproducción. Para ello, haga clic con el botón derecho en el programa y vaya a **Playback the program(s)** (Reproducir los programas). Luego, seleccione **with Azure Media Player** (con Azure Media Player).  
6. Una vez confirmada, haga clic con el botón derecho de nuevo en el programa y seleccione **Copy the Output URL to Clipboard** (Copiar la dirección URL de salida en el Portapapeles) o recupere esta información con la opción **Program information and settings**(Información y configuración del programa) en el menú.

La transmisión está ahora preparada para insertarse en un reproductor o distribuirse a una audiencia para su visualización en directo.  

## <a name="troubleshooting"></a>Solución de problemas
Para más información, consulte el artículo de [solución de problemas](media-services-troubleshooting-live-streaming.md).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
