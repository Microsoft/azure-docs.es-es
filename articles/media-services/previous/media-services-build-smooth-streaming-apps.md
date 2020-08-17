---
title: Tutorial de aplicación de la Tienda Windows de Smooth Streaming | Microsoft Docs
description: Aprenda a usar Azure Media Services para crear una aplicación de la Tienda Windows de C# con un control MediaElement de XML para reproducir contenido de Smooth Streaming.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 0fa5d8c5-3d5f-4886-ae55-fb6de4f5256d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.openlocfilehash: 44f10bd49a768004fc63a3287799e6b79dd5bae1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071900"
---
# <a name="how-to-build-a-smooth-streaming-windows-store-application"></a>Generación de una aplicación de la Tienda Windows de Smooth Streaming  

El SDK de cliente Smooth Streaming para Windows 8 permite a los desarrolladores crear aplicaciones de la Tienda Windows que reproduzcan contenido de Smooth Streaming en directo y bajo demanda. Además de la reproducción básica de contenido de Smooth Streaming, el SDK también ofrece funciones enriquecidas como la protección de Microsoft PlayReady, la restricción del nivel de calidad, Live DVR, la conmutación de la secuencia de audio, la escucha de las actualizaciones de estado (como los cambios del nivel de calidad) y los eventos de error, entre otras. Para obtener más información acerca de las funciones compatibles, consulte las [notas de la versión](https://www.iis.net/learn/media/smooth-streaming/smooth-streaming-client-sdk-for-windows-8-release-notes). Para más información, vea [Player Framework para Windows 8](https://playerframework.codeplex.com/). 

Este tutorial contiene cuatro lecciones:

1. Creación de una aplicación básica de Tienda de Smooth Streaming
2. Incorporación de una barra deslizante para controlar el progreso multimedia
3. Selección de secuencias de Smooth Streaming
4. Selección de pistas de Smooth Streaming

## <a name="prerequisites"></a>Prerrequisitos
> [!NOTE]
> Los proyectos de la Tienda Windows versión 8.1 y versiones anteriores no se admiten en Visual Studio 2017.  Para más información, consulte [Compatibilidad y destinatarios de la plataforma de Visual Studio 2017](https://www.visualstudio.com/en-us/productinfo/vs2017-compatibility-vs).

* Windows 8 de 32 o 64 bits.
* Versiones de 2012 a 2015 de Visual Studio.
* [SDK de cliente Smooth Streaming de Microsoft para Windows 8](https://visualstudiogallery.msdn.microsoft.com/04423d13-3b3e-4741-a01c-1ae29e84fea6?SRC=Home).

La solución completa de cada lección se puede descargar en las muestras de código de desarrollador de MSDN (Galería de código) (información en inglés): 

* [Lección 1](https://code.msdn.microsoft.com/Smooth-Streaming-Client-0bb1471f) : Un sencillo reproductor multimedia de Smooth Streaming para Windows 8 
* [Lección 2](https://code.msdn.microsoft.com/A-simple-Windows-8-Smooth-ee98f63a) : Un sencillo reproductor multimedia de Smooth Streaming para Windows 8 con un control de barra deslizante 
* [Lección 3](https://code.msdn.microsoft.com/A-Windows-8-Smooth-883c3b44) : Un reproductor multimedia de Smooth Streaming para Windows 8 con selección de secuencias  
* [Lección 4](https://code.msdn.microsoft.com/A-Windows-8-Smooth-aa9e4907): Un reproductor multimedia de Smooth Streaming para Windows 8 con selección de pistas

## <a name="lesson-1-create-a-basic-smooth-streaming-store-application"></a>Lección 1: Creación de una aplicación básica de Tienda de Smooth Streaming

En esta lección, creará una aplicación de Tienda Windows con un control MediaElement para reproducir contenido de Smooth Streaming.  La aplicación en ejecución tiene este aspecto:

![Ejemplo de aplicación de Tienda Windows de Smooth Streaming][PlayerApplication]

Para obtener más información acerca del desarrollo de la aplicación de la Tienda Windows, consulte [Desarrollo de magníficas aplicaciones para Windows 8](https://developer.microsoft.com/en-us/windows/). Esta lección contiene los procedimientos siguientes:

1. Creación de un proyecto de Tienda Windows
2. Diseño de la interfaz de usuario (XAML)
3. Modificación del archivo de código subyacente
4. Compilación y prueba de la aplicación

### <a name="to-create-a-windows-store-project"></a>Para crear un proyecto de Tienda Windows

1. Ejecute Visual Studio; se admiten las versiones 2012 a 2015.
1. En el menú **ARCHIVO**, haga clic en **Nuevo** y en **Proyecto**.
1. En el diálogo Nuevo proyecto, escriba o seleccione los valores siguientes:

    | Nombre | Value |
    | --- | --- |
    | Grupo de plantillas |Instalado/Plantillas/Visual C#/Tienda Windows |
    | Plantilla |Aplicación vacía (XAML) |
    | Nombre |SSPlayer |
    | Location |C:\SSTutorials |
    | Nombre de solución |SSPlayer |
    | Crear directorio para la solución |(seleccionado) |

1. Haga clic en **OK**.

### <a name="to-add-a-reference-to-the-smooth-streaming-client-sdk"></a>Para agregar una referencia al SDK de cliente Smooth Streaming

1. En el Explorador de soluciones, haga clic con el botón derecho en **SSPlayer** y haga clic en **Agregar referencia**.
1. Escriba o seleccione los valores siguientes:

    | Nombre | Value |
    | --- | --- |
    | Grupo de referencia |Windows/Extensiones |
    | Referencia |Seleccione el SDK de cliente Smooth Streaming de Microsoft para Windows 8 y el paquete en tiempo de ejecución de Microsoft Visual C++ |

1. Haga clic en **OK**. 

Después de agregar las referencias, debe seleccionar la plataforma de destino (x64 o x86), ya que en ninguna configuración de la plataforma de la CPU funciona la incorporación de las referencias.  En el Explorador de soluciones, verá una marca de advertencia amarilla en estas referencias agregadas.

### <a name="to-design-the-player-user-interface"></a>Para diseñar la interfaz de usuario del reproductor

1. En el Explorador de soluciones, haga doble clic en **MainPage.xaml** para abrirlo en la vista de diseño.
2. Busque las etiquetas **&lt;Grid&gt;** y **&lt;/Grid&gt;** del archivo XAML y pegue el código siguiente entre ellas:

   ```xml
         <Grid.RowDefinitions>

            <RowDefinition Height="20"/>    <!-- spacer -->
            <RowDefinition Height="50"/>    <!-- media controls -->
            <RowDefinition Height="100*"/>  <!-- media element -->
            <RowDefinition Height="80*"/>   <!-- media stream and track selection -->
            <RowDefinition Height="50"/>    <!-- status bar -->
         </Grid.RowDefinitions>

         <StackPanel Name="spMediaControl" Grid.Row="1" Orientation="Horizontal">
            <TextBlock x:Name="tbSource" Text="Source :  " FontSize="16" FontWeight="Bold" VerticalAlignment="Center" />
            <TextBox x:Name="txtMediaSource" Text="https://ecn.channel9.msdn.com/o9/content/smf/smoothcontent/elephantsdream/Elephants_Dream_1024-h264-st-aac.ism/manifest" FontSize="10" Width="700" Margin="0,4,0,10" />
            <Button x:Name="btnSetSource" Content="Set Source" Width="111" Height="43" Click="btnSetSource_Click"/>
            <Button x:Name="btnPlay" Content="Play" Width="111" Height="43" Click="btnPlay_Click"/>
            <Button x:Name="btnPause" Content="Pause"  Width="111" Height="43" Click="btnPause_Click"/>
            <Button x:Name="btnStop" Content="Stop"  Width="111" Height="43" Click="btnStop_Click"/>
            <CheckBox x:Name="chkAutoPlay" Content="Auto Play" Height="55" Width="Auto" IsChecked="{Binding AutoPlay, ElementName=mediaElement, Mode=TwoWay}"/>
            <CheckBox x:Name="chkMute" Content="Mute" Height="55" Width="67" IsChecked="{Binding IsMuted, ElementName=mediaElement, Mode=TwoWay}"/>
         </StackPanel>

         <StackPanel Name="spMediaElement" Grid.Row="2" Height="435" Width="1072"
                    HorizontalAlignment="Center" VerticalAlignment="Center">
            <MediaElement x:Name="mediaElement" Height="356" Width="924" MinHeight="225"
                          HorizontalAlignment="Center" VerticalAlignment="Center" 
                          AudioCategory="BackgroundCapableMedia" />
            <StackPanel Orientation="Horizontal">
                <Slider x:Name="sliderProgress" Width="924" Height="44"
                        HorizontalAlignment="Center" VerticalAlignment="Center"
                        PointerPressed="sliderProgress_PointerPressed"/>
                <Slider x:Name="sliderVolume" 
                        HorizontalAlignment="Right" VerticalAlignment="Center" Orientation="Vertical" 
                        Height="79" Width="148" Minimum="0" Maximum="1" StepFrequency="0.1" 
                        Value="{Binding Volume, ElementName=mediaElement, Mode=TwoWay}" 
                        ToolTipService.ToolTip="{Binding Value, RelativeSource={RelativeSource Mode=Self}}"/>
            </StackPanel>
         </StackPanel>

         <StackPanel Name="spStatus" Grid.Row="4" Orientation="Horizontal">
            <TextBlock x:Name="tbStatus" Text="Status :  " 
               FontSize="16" FontWeight="Bold" VerticalAlignment="Center" HorizontalAlignment="Center" />
            <TextBox x:Name="txtStatus" FontSize="10" Width="700" VerticalAlignment="Center"/>
         </StackPanel>
   ```
   Se usa el control MediaElement para reproducir contenido multimedia. En la lección siguiente se usará el control deslizante llamado sliderProgress para controlar el progreso multimedia.
3. Presione **CTRL+S** para guardar el archivo.

El control MediaElement no admite contenido Smooth Streaming directamente. Para habilitar la compatibilidad con Smooth Streaming, debe registrar el controlador de esquema de bytes de Smooth Streaming por la extensión del nombre del archivo y el tipo MIME.  Para el registro, debe usar el método MediaExtensionManager.RegisterByteStreamHandler del espacio de nombres Windows.Media.

En este archivo XAML, algunos controladores de eventos están asociados a los controles.  Debe definir dichos controladores de eventos.

### <a name="to-modify-the-code-behind-file"></a>Para modificar el archivo de código subyacente

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. En la parte superior del archivo, agregue las siguientes instrucciones using:

    ```csharp
        using Windows.Media;
    ```

3. Al principio de la clase **MainPage** , agregue el miembro de datos siguiente:

    ```csharp
        private MediaExtensionManager extensions = new MediaExtensionManager();
    ```

4. Al principio del constructor **MainPage** , agregue las dos líneas siguientes:

    ```csharp
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "text/xml");
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "application/vnd.ms-sstr+xml");
    ```

5. Al final de la clase **MainPage**, pegue el código siguiente:
   ```csharp
         # region UI Button Click Events
         private void btnPlay_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Play();
         txtStatus.Text = "MediaElement is playing ...";
         }
         private void btnPause_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Pause();
         txtStatus.Text = "MediaElement is paused";
         }
         private void btnSetSource_Click(object sender, RoutedEventArgs e)
         {

         sliderProgress.Value = 0;
         mediaElement.Source = new Uri(txtMediaSource.Text);

         if (chkAutoPlay.IsChecked == true)
         {
             txtStatus.Text = "MediaElement is playing ...";
         }
         else
         {
             txtStatus.Text = "Click the Play button to play the media source.";
         }
         }
         private void btnStop_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Stop();
         txtStatus.Text = "MediaElement is stopped";
         }
         private void sliderProgress_PointerPressed(object sender, PointerRoutedEventArgs e)
         {

         txtStatus.Text = "Seek to position " + sliderProgress.Value;
         mediaElement.Position = new TimeSpan(0, 0, (int)(sliderProgress.Value));
         }
         # endregion
   ```
   Aquí se define el controlador de eventos sliderProgress_PointerPressed.  Hay algo más que hacer para que empiece a funcionar, pero ello se tratará en la lección siguiente del tutorial.
6. Presione **CTRL+S** para guardar el archivo.

El archivo de código subyacente finalizado tendrá un aspecto similar al siguiente:

![Vista del código en Visual Studio de la aplicación de Tienda Windows de Smooth Streaming][CodeViewPic]

### <a name="to-compile-and-test-the-application"></a>Para compilar y probar la aplicación

1. En el menú **COMPILAR**, haga clic en **Administrador de configuración**.
2. Cambie **Plataforma de soluciones activa** para que coincida con la plataforma de desarrollo.
3. Presione **F6** para compilar el proyecto. 
4. Presione **F5** para ejecutar la aplicación.
5. En la parte superior de la aplicación, puede usar la URL de Smooth Streaming predeterminada o escribir una diferente. 
6. Haga clic en **Establecer origen**. Como la opción **Reproducción automática** está habilitada de forma predeterminada, el contenido multimedia se reproducirá automáticamente.  Puede controlar el contenido multimedia con los botones **Reproducir**, **Pausa** y **Detener**.  Puede controlar el volumen multimedia usando el control deslizante vertical.  No obstante, el control deslizante horizontal que controla el progreso multimedia no está completamente implementado todavía. 

Ha terminado la lección 1.  En esta lección se usa el control MediaElement para reproducir contenido de Smooth Streaming.  En la lección siguiente, agregará un control deslizante para el progreso del contenido de Smooth Streaming.

## <a name="lesson-2-add-a-slider-bar-to-control-the-media-progress"></a>Lección 2: Incorporación de una barra deslizante para controlar el progreso multimedia

En la lección 1 ha creado una aplicación de Tienda Windows con un control XAML de MediaElement para reproducir contenido multimedia de Smooth Streaming.  Esta incorpora algunas funciones multimedia básicas como iniciar, detener y poner en pausa.  En esta lección, agregará un control de barra deslizante a la aplicación.

En este tutorial usaremos un temporizador para actualizar la posición del control deslizante según la posición actual del control MediaElement.  Hay que actualizar las horas de inicio y fin del control deslizante en caso de que se use contenido en directo.  Esto se puede controlar mejor en el evento de actualización del origen adaptivo.

Los orígenes multimedia son objetos que generan datos multimedia.  La resolución del origen usa una URL o una secuencia de bytes y crea el origen multimedia adecuado para el contenido en cuestión.  La resolución del origen es la forma estándar que tienen las aplicaciones para crear los orígenes multimedia. 

Esta lección contiene los procedimientos siguientes:

1. Registro del controlador de Smooth Streaming 
2. Incorporación de los controladores de eventos de nivel de administrador de origen adaptativo
3. Incorporación de los controladores de eventos de nivel de origen adaptativo
4. Incorporación de controladores de eventos MediaElement
5. Incorporación de código relacionado con la barra deslizante
6. Compilación y prueba de la aplicación

### <a name="to-register-the-smooth-streaming-byte-stream-handler-and-pass-the-propertyset"></a>Para registrar el controlador de esquema de bytes de Smooth Streaming y pasar el conjunto de propiedades

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Al principio del archivo, agregue las siguientes instrucciones using:

   ```csharp
        using Microsoft.Media.AdaptiveStreaming;
   ```
3. Al principio de la clase MainPage, agregue los miembros de datos siguientes:

   ```csharp
         private Windows.Foundation.Collections.PropertySet propertySet = new Windows.Foundation.Collections.PropertySet();             
         private IAdaptiveSourceManager adaptiveSourceManager;
   ```
4. Dentro del constructor **MainPage**, agregue el código siguiente después de la línea **this.Initialize Components();** y las líneas del código de registro escritas en la lección anterior:

   ```csharp
        // Gets the default instance of AdaptiveSourceManager which manages Smooth 
        //Streaming media sources.
        adaptiveSourceManager = AdaptiveSourceManager.GetDefault();
        // Sets property key value to AdaptiveSourceManager default instance.
        // {A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}" must be hardcoded.
        propertySet["{A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}"] = adaptiveSourceManager;
   ```
5. Dentro del constructor **MainPage** , modifique los dos métodos RegisterByteStreamHandler para agregar los parámetros forth:

   ```csharp
         // Registers Smooth Streaming byte-stream handler for ".ism" extension and, 
         // "text/xml" and "application/vnd.ms-ss" mime-types and pass the propertyset. 
         // http://*.ism/manifest URI resources will be resolved by Byte-stream handler.
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "text/xml", 
            propertySet );
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "application/vnd.ms-sstr+xml", 
         propertySet);
   ```
6. Presione **CTRL+S** para guardar el archivo.

### <a name="to-add-the-adaptive-source-manager-level-event-handler"></a>Para agregar el controlador de eventos de nivel de administrador de orígenes adaptativo

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Dentro de la clase **MainPage** , agregue el siguiente miembro de datos:

   ```csharp
     private AdaptiveSource adaptiveSource = null;
   ```
3. Al final de la clase **MainPage** , agregue el siguiente controlador de eventos:

   ```csharp
         # region Adaptive Source Manager Level Events
         private void mediaElement_AdaptiveSourceOpened(AdaptiveSource sender, AdaptiveSourceOpenedEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
         }

         # endregion Adaptive Source Manager Level Events
   ```
4. Al final del constructor **MainPage** , agregue la línea siguiente para suscribirse al evento abierto de origen adaptativo:

   ```csharp
         adaptiveSourceManager.AdaptiveSourceOpenedEvent += 
           new AdaptiveSourceOpenedEventHandler(mediaElement_AdaptiveSourceOpened);
   ```
5. Presione **CTRL+S** para guardar el archivo.

### <a name="to-add-adaptive-source-level-event-handlers"></a>Para agregar los controladores de eventos de nivel de origen adaptativo

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Dentro de la clase **MainPage** , agregue el siguiente miembro de datos:

   ```csharp
     private AdaptiveSourceStatusUpdatedEventArgs adaptiveSourceStatusUpdate; 
     private Manifest manifestObject;
   ```
3. Al final de la clase **MainPage** , agregue los siguientes controladores de eventos:

   ```csharp
         # region Adaptive Source Level Events
         private void mediaElement_ManifestReady(AdaptiveSource sender, ManifestReadyEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
            manifestObject = args.AdaptiveSource.Manifest;
         }

         private void mediaElement_AdaptiveSourceStatusUpdated(AdaptiveSource sender, AdaptiveSourceStatusUpdatedEventArgs args)
         {

            adaptiveSourceStatusUpdate = args;
         }

         private void mediaElement_AdaptiveSourceFailed(AdaptiveSource sender, AdaptiveSourceFailedEventArgs args)
         {

            txtStatus.Text = "Error: " + args.HttpResponse;
         }

         # endregion Adaptive Source Level Events
   ```
4. Al final del método **mediaElement AdaptiveSourceOpened** , agregue el código siguiente para suscribirse a los eventos:

   ```csharp
         adaptiveSource.ManifestReadyEvent +=

                    mediaElement_ManifestReady;
         adaptiveSource.AdaptiveSourceStatusUpdatedEvent += 

            mediaElement_AdaptiveSourceStatusUpdated;
         adaptiveSource.AdaptiveSourceFailedEvent += 

            mediaElement_AdaptiveSourceFailed;
   ```
5. Presione **CTRL+S** para guardar el archivo.

También están disponibles los mismos eventos en el nivel de administrador de origen adaptativo, que se pueden usar para administrar la funcionalidad común a todos los elementos multimedia de la aplicación. Cada AdaptiveSource incluye sus propios eventos y todos los eventos AdaptiveSource se aplicarán en cascada debajo de AdaptiveSourceManager.

### <a name="to-add-media-element-event-handlers"></a>Para agregar controladores de eventos de elementos multimedia

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Al final de la clase **MainPage** , agregue los siguientes controladores de eventos:

   ```csharp
         # region Media Element Event Handlers
         private void MediaOpened(object sender, RoutedEventArgs e)
         {

            txtStatus.Text = "MediaElement opened";
         }

         private void MediaFailed(object sender, ExceptionRoutedEventArgs e)
         {

            txtStatus.Text= "MediaElement failed: " + e.ErrorMessage;
         }

         private void MediaEnded(object sender, RoutedEventArgs e)
         {

            txtStatus.Text ="MediaElement ended.";
         }

         # endregion Media Element Event Handlers
   ```
3. Al final del constructor **MainPage** , agregue el código siguiente para suscribirse a los eventos:

   ```csharp
         mediaElement.MediaOpened += MediaOpened;
         mediaElement.MediaEnded += MediaEnded;
         mediaElement.MediaFailed += MediaFailed;
   ```
4. Presione **CTRL+S** para guardar el archivo.

### <a name="to-add-slider-bar-related-code"></a>Para agregar código relacionado con la barra deslizante

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Al principio del archivo, agregue las siguientes instrucciones using:

   ```csharp
        using Windows.UI.Core;
   ```
3. Dentro de la clase **MainPage** , agregue los siguientes miembros de datos:

   ```csharp
         public static CoreDispatcher _dispatcher;
         private DispatcherTimer sliderPositionUpdateDispatcher;
   ```
4. Al final del constructor **MainPage** , agregue el código siguiente:

   ```csharp
         _dispatcher = Window.Current.Dispatcher;
         PointerEventHandler pointerpressedhandler = new PointerEventHandler(sliderProgress_PointerPressed);
         sliderProgress.AddHandler(Control.PointerPressedEvent, pointerpressedhandler, true);    
   ```
5. Al final de la clase **MainPage** , agregue el código siguiente:

   ```csharp
         # region sliderMediaPlayer
         private double SliderFrequency(TimeSpan timevalue)
         {

            long absvalue = 0;
            double stepfrequency = -1;

            if (manifestObject != null)
            {
                absvalue = manifestObject.Duration - (long)manifestObject.StartTime;
            }
            else
            {
                absvalue = mediaElement.NaturalDuration.TimeSpan.Ticks;
            }

            TimeSpan totalDVRDuration = new TimeSpan(absvalue);

            if (totalDVRDuration.TotalMinutes >= 10 && totalDVRDuration.TotalMinutes < 30)
            {
               stepfrequency = 10;
            }
            else if (totalDVRDuration.TotalMinutes >= 30 
                     && totalDVRDuration.TotalMinutes < 60)
            {
                stepfrequency = 30;
            }
            else if (totalDVRDuration.TotalHours >= 1)
            {
                stepfrequency = 60;
            }

            return stepfrequency;
         }

         void updateSliderPositionoNTicks(object sender, object e)
         {

            sliderProgress.Value = mediaElement.Position.TotalSeconds;
         }

         public void setupTimer()
         {

            sliderPositionUpdateDispatcher = new DispatcherTimer();
            sliderPositionUpdateDispatcher.Interval = new TimeSpan(0, 0, 0, 0, 300);
            startTimer();
         }

         public void startTimer()
         {

            sliderPositionUpdateDispatcher.Tick += updateSliderPositionoNTicks;
            sliderPositionUpdateDispatcher.Start();
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderStartTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.StartTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Minimum = absvalue;
            });
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderEndTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Maximum = absvalue;
            });
         }

         # endregion sliderMediaPlayer
   ```

   > [!NOTE]
   > Se usa CoreDispatcher para realizar cambios en el subproceso de interfaz de usuario desde un subproceso no perteneciente a la interfaz de usuario. En caso de que se produzca un cuello de botella en el subproceso de distribuidor, el desarrollador puede optar por usar el distribuidor proporcionado por el elemento de interfaz de usuario que pretende actualizar.  Por ejemplo:

   ```csharp
         await sliderProgress.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => { TimeSpan 

         timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime); 
         double absvalue  = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero); 

         sliderProgress.Maximum = absvalue; }); 
   ```
6. Al final del método **mediaElement_AdaptiveSourceStatusUpdated**, agregue el código siguiente:

   ```csharp
         setSliderStartTime(args.StartTime);
         setSliderEndTime(args.EndTime);
   ```
7. Al final del método **MediaOpened** , agregue el código siguiente:

   ```csharp
         sliderProgress.StepFrequency = SliderFrequency(mediaElement.NaturalDuration.TimeSpan);
         sliderProgress.Width = mediaElement.Width;
         setupTimer();
   ```
8. Presione **CTRL+S** para guardar el archivo.

### <a name="to-compile-and-test-the-application"></a>Para compilar y probar la aplicación

1. Presione **F6** para compilar el proyecto. 
2. Presione **F5** para ejecutar la aplicación.
3. En la parte superior de la aplicación, puede usar la URL de Smooth Streaming predeterminada o escribir una diferente. 
4. Haga clic en **Establecer origen**. 
5. Pruebe la barra deslizante.

Ha completado la lección 2.  En esta lección ha agregado un control deslizante a la aplicación. 

## <a name="lesson-3-select-smooth-streaming-streams"></a>Lección 3: Selección de secuencias de Smooth Streaming
Smooth Streaming es capaz de transmitir contenido con pistas de audio de varios lenguajes que los usuarios pueden seleccionar.  En esta lección, podrá habilitar a los usuarios para que seleccionen las transmisiones. Esta lección contiene los procedimientos siguientes:

1. Modificación del archivo XAML
2. Modificación del archivo de código subyacente
3. Compilación y prueba de la aplicación

### <a name="to-modify-the-xaml-file"></a>Para modificar el archivo XAML

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Diseñador de vistas**.
2. Busque &lt;Grid.RowDefinitions&gt; y modifique RowDefinitions para que tenga el aspecto siguiente:

   ```xml
         <Grid.RowDefinitions>            
            <RowDefinition Height="20"/>
            <RowDefinition Height="50"/>
            <RowDefinition Height="100"/>
            <RowDefinition Height="80"/>
            <RowDefinition Height="50"/>
         </Grid.RowDefinitions>
   ```
3. Dentro de las etiquetas &lt;Grid&gt;&lt;/Grid&gt;, agregue el código siguiente para definir un control de cuadro de lista, para que los usuarios vean la lista de transmisiones disponibles, y seleccione transmisiones:

   ```xml
         <Grid Name="gridStreamAndBitrateSelection" Grid.Row="3">
            <Grid.RowDefinitions>
                <RowDefinition Height="300"/>
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="250*"/>
                <ColumnDefinition Width="250*"/>
            </Grid.ColumnDefinitions>
            <StackPanel Name="spStreamSelection" Grid.Row="1" Grid.Column="0">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Name="tbAvailableStreams" Text="Available Streams:" FontSize="16" VerticalAlignment="Center"></TextBlock>
                    <Button Name="btnChangeStreams" Content="Submit" Click="btnChangeStream_Click"/>
                </StackPanel>
                <ListBox x:Name="lbAvailableStreams" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                    ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
                    <ListBox.ItemTemplate>
                        <DataTemplate>
                            <CheckBox Content="{Binding Name}" IsChecked="{Binding isChecked, Mode=TwoWay}" />
                        </DataTemplate>
                    </ListBox.ItemTemplate>
                </ListBox>
            </StackPanel>
         </Grid>
   ```
4. Presione **CTRL+S** para guardar los cambios.

### <a name="to-modify-the-code-behind-file"></a>Para modificar el archivo de código subyacente

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Dentro del espacio de nombres SSPlayer, agregue la nueva clase:

   ```csharp
        #region class Stream
   
        public class Stream
        {
            private IManifestStream stream;
            public bool isCheckedValue;
            public string name;
   
            public string Name
            {
                get { return name; }
                set { name = value; }
            }
   
            public IManifestStream ManifestStream
            {
                get { return stream; }
                set { stream = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    // mMke the video stream always checked.
                    if (stream.Type == MediaStreamType.Video)
                    {
                        isCheckedValue = true;
                    }
                    else
                    {
                        isCheckedValue = value;
                    }
                }
            }
   
            public Stream(IManifestStream streamIn)
            {
                stream = streamIn;
                name = stream.Name;
            }
        }
        #endregion class Stream
   ```
3. Al principio de la clase MainPage, agregue las siguientes definiciones de variable:

   ```csharp
         private List<Stream> availableStreams;
         private List<Stream> availableAudioStreams;
         private List<Stream> availableTextStreams;
         private List<Stream> availableVideoStreams;
   ```
4. Dentro de la clase MainPage, agregue la región siguiente:
   ```csharp
        #region stream selection
        ///<summary>
        ///Functionality to select streams from IManifestStream available streams
        /// </summary>
   
        // This function is called from the mediaElement_ManifestReady event handler 
        // to retrieve the streams and populate them to the local data members.
        public void getStreams(Manifest manifestObject)
        {
            availableStreams = new List<Stream>();
            availableVideoStreams = new List<Stream>();
            availableAudioStreams = new List<Stream>();
            availableTextStreams = new List<Stream>();
   
            try
            {
                for (int i = 0; i<manifestObject.AvailableStreams.Count; i++)
                {
                    Stream newStream = new Stream(manifestObject.AvailableStreams[i]);
                    newStream.isChecked = false;
   
                    //populate the stream lists based on the types
                    availableStreams.Add(newStream);
   
                    switch (newStream.ManifestStream.Type)
                    {
                        case MediaStreamType.Video:
                            availableVideoStreams.Add(newStream);
                            break;
                        case MediaStreamType.Audio:
                            availableAudioStreams.Add(newStream);
                            break;
                        case MediaStreamType.Text:
                            availableTextStreams.Add(newStream);
                            break;
                    }
   
                    // Select the default selected streams from the manifest.
                    for (int j = 0; j<manifestObject.SelectedStreams.Count; j++)
                    {
                        string selectedStreamName = manifestObject.SelectedStreams[j].Name;
                        if (selectedStreamName.Equals(newStream.Name))
                        {
                            newStream.isChecked = true;
                            break;
                        }
                    }
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function set the list box ItemSource
        private async void refreshAvailableStreamsListBoxItemSource()
        {
            try
            {
                //update the stream check box list on the UI
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableStreams.ItemsSource = availableStreams; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function creates a selected streams list
        private void createSelectedStreamsList(List<IManifestStream> selectedStreams)
        {
            bool isOneVideoSelected = false;
            bool isOneAudioSelected = false;
   
            // Only one video stream can be selected
            for (int j = 0; j<availableVideoStreams.Count; j++)
            {
                if (availableVideoStreams[j].isChecked && (!isOneVideoSelected))
                {
                    selectedStreams.Add(availableVideoStreams[j].ManifestStream);
                    isOneVideoSelected = true;
                }
            }
   
            // Select the first video stream from the list if no video stream is selected
            if (!isOneVideoSelected)
            {
                availableVideoStreams[0].isChecked = true;
                selectedStreams.Add(availableVideoStreams[0].ManifestStream);
            }
   
            // Only one audio stream can be selected
            for (int j = 0; j<availableAudioStreams.Count; j++)
            {
                if (availableAudioStreams[j].isChecked && (!isOneAudioSelected))
                {
                    selectedStreams.Add(availableAudioStreams[j].ManifestStream);
                    isOneAudioSelected = true;
                    txtStatus.Text = "The audio stream is changed to " + availableAudioStreams[j].ManifestStream.Name;
                }
            }
   
            // Select the first audio stream from the list if no audio steam is selected.
            if (!isOneAudioSelected)
            {
                availableAudioStreams[0].isChecked = true;
                selectedStreams.Add(availableAudioStreams[0].ManifestStream);
            }
   
            // Multiple text streams are supported.
            for (int j = 0; j < availableTextStreams.Count; j++)
            {
                if (availableTextStreams[j].isChecked)
                {
                    selectedStreams.Add(availableTextStreams[j].ManifestStream);
                }
            }
        }
   
        // Change streams on a smooth streaming presentation with multiple video streams.
        private async void changeStreams(List<IManifestStream> selectStreams)
        {
            try
            {
                IReadOnlyList<IStreamChangedResult> returnArgs =
                    await manifestObject.SelectStreamsAsync(selectStreams);
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
        #endregion stream selection
   ```
5. Busque el método mediaElement_ManifestReady y anexe el código siguiente al final de la función:
   ```csharp
        getStreams(manifestObject);
        refreshAvailableStreamsListBoxItemSource();
   ```
    Así, cuando el manifiesto MediaElement está listo, el código obtiene una lista de secuencias disponibles y rellena el cuadro de lista de la interfaz de usuario con la lista.
6. Dentro de la clase MainPage, busque la región de eventos de clic de los botones de interfaz de usuario y, a continuación, agregue la siguiente definición de función:
   ```csharp
        private void btnChangeStream_Click(object sender, RoutedEventArgs e)
        {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();
   
            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);
   
            // Change streams on the presentation
            changeStreams(selectedStreams);
        }
   ```

### <a name="to-compile-and-test-the-application"></a>Para compilar y probar la aplicación

1. Presione **F6** para compilar el proyecto. 
2. Presione **F5** para ejecutar la aplicación.
3. En la parte superior de la aplicación, puede usar la URL de Smooth Streaming predeterminada o escribir una diferente. 
4. Haga clic en **Establecer origen**. 
5. El idioma predeterminado es audio_eng. Intente cambiar entre audio_eng y audio_es. Cada vez que seleccione una nueva secuencia, debe hacer clic en el botón Enviar.

Ha completado la lección 3.  En esta lección ha agregado la funcionalidad para elegir secuencias.

## <a name="lesson-4-select-smooth-streaming-tracks"></a>Lección 4: Selección de pistas de Smooth Streaming

Una presentación de Smooth Streaming puede contener varios archivos de vídeo codificados con diferentes niveles de calidad (velocidad de bits) y resoluciones. En esta lección habilitará a los usuarios para que puedan seleccionar las pistas. Esta lección contiene los procedimientos siguientes:

1. Modificación del archivo XAML
2. Modificación del archivo de código subyacente
3. Compilación y prueba de la aplicación

### <a name="to-modify-the-xaml-file"></a>Para modificar el archivo XAML

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Diseñador de vistas**.
2. Busque la etiqueta &lt;Grid&gt; cuyo nombre es **gridStreamAndBitrateSelection** y anexe el código siguiente al final de la etiqueta:
   ```xml
         <StackPanel Name="spBitRateSelection" Grid.Row="1" Grid.Column="1">
         <StackPanel Orientation="Horizontal">
             <TextBlock Name="tbBitRate" Text="Available Bitrates:" FontSize="16" VerticalAlignment="Center"/>
             <Button Name="btnChangeTracks" Content="Submit" Click="btnChangeTrack_Click" />
         </StackPanel>
         <ListBox x:Name="lbAvailableVideoTracks" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                  ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
             <ListBox.ItemTemplate>
                 <DataTemplate>
                     <CheckBox Content="{Binding Bitrate}" IsChecked="{Binding isChecked, Mode=TwoWay}"/>
                 </DataTemplate>
             </ListBox.ItemTemplate>
         </ListBox>
         </StackPanel>
   ```
3. Presione **CTRL+S** para guardar los cambios

### <a name="to-modify-the-code-behind-file"></a>Para modificar el archivo de código subyacente

1. En el Explorador de soluciones, haga clic con el botón derecho en **MainPage.xaml** y haga clic en **Ver código**.
2. Dentro del espacio de nombres SSPlayer, agregue la nueva clase:
   ```csharp
        #region class Track
        public class Track
        {
            private IManifestTrack trackInfo;
            public string _bitrate;
            public bool isCheckedValue;
   
            public IManifestTrack TrackInfo
            {
                get { return trackInfo; }
                set { trackInfo = value; }
            }
   
            public string Bitrate
            {
                get { return _bitrate; }
                set { _bitrate = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    isCheckedValue = value;
                }
            }
   
            public Track(IManifestTrack trackInfoIn)
            {
                trackInfo = trackInfoIn;
                _bitrate = trackInfoIn.Bitrate.ToString();
            }
            //public Track() { }
        }
        #endregion class Track
   ```
3. Al principio de la clase MainPage, agregue las siguientes definiciones de variable:
   ```csharp
        private List<Track> availableTracks;
   ```
4. Dentro de la clase MainPage, agregue la región siguiente:
   ```csharp
        #region track selection
        /// <summary>
        /// Functionality to select video streams
        /// </summary>
   
        /// This Function gets the tracks for the selected video stream
        public void getTracks(Manifest manifestObject)
        {
            availableTracks = new List<Track>();
   
            IManifestStream videoStream = getVideoStream();
            IReadOnlyList<IManifestTrack> availableTracksLocal = videoStream.AvailableTracks;
            IReadOnlyList<IManifestTrack> selectedTracksLocal = videoStream.SelectedTracks;
   
            try
            {
                for (int i = 0; i < availableTracksLocal.Count; i++)
                {
                    Track thisTrack = new Track(availableTracksLocal[i]);
                    thisTrack.isChecked = true;
   
                    for (int j = 0; j < selectedTracksLocal.Count; j++)
                    {
                        string selectedTrackName = selectedTracksLocal[j].Bitrate.ToString();
                        if (selectedTrackName.Equals(thisTrack.Bitrate))
                        {
                            thisTrack.isChecked = true;
                            break;
                        }
                    }
                    availableTracks.Add(thisTrack);
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = e.Message;
            }
        }
   
        // This function gets the video stream that is playing
        private IManifestStream getVideoStream()
        {
            IManifestStream videoStream = null;
            for (int i = 0; i < manifestObject.SelectedStreams.Count; i++)
            {
                if (manifestObject.SelectedStreams[i].Type == MediaStreamType.Video)
                {
                    videoStream = manifestObject.SelectedStreams[i];
                    break;
                }
            }
            return videoStream;
        }
   
        // This function set the UI list box control ItemSource
        private async void refreshAvailableTracksListBoxItemSource()
        {
            try
            {
                // Update the track check box list on the UI 
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableVideoTracks.ItemsSource = availableTracks; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }        
        }
   
        // This function creates a list of the selected tracks.
        private void createSelectedTracksList(List<IManifestTrack> selectedTracks)
        {
            // Create a list of selected tracks
            for (int j = 0; j < availableTracks.Count; j++)
            {
                if (availableTracks[j].isCheckedValue == true)
                {
                    selectedTracks.Add(availableTracks[j].TrackInfo);
                }
            }
        }
   
        // This function selects the tracks based on user selection 
        private void changeTracks(List<IManifestTrack> selectedTracks)
        {
            IManifestStream videoStream = getVideoStream();
            try
            {
                videoStream.SelectTracks(selectedTracks);
            }
            catch (Exception ex)
            {
                txtStatus.Text = ex.Message;
            }
        }
        #endregion track selection
   ```
5. Busque el método mediaElement_ManifestReady y anexe el código siguiente al final de la función:
   ```csharp
         getTracks(manifestObject);
         refreshAvailableTracksListBoxItemSource();
   ```
6. Dentro de la clase MainPage, busque la región de eventos de clic de los botones de interfaz de usuario y, a continuación, agregue la siguiente definición de función:
   ```csharp
         private void btnChangeStream_Click(object sender, RoutedEventArgs e)
         {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();

            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);

            // Change streams on the presentation
            changeStreams(selectedStreams);
         }
   ```
   
### <a name="to-compile-and-test-the-application"></a>Para compilar y probar la aplicación

1. Presione **F6** para compilar el proyecto. 
2. Presione **F5** para ejecutar la aplicación.
3. En la parte superior de la aplicación, puede usar la URL de Smooth Streaming predeterminada o escribir una diferente. 
4. Haga clic en **Establecer origen**. 
5. De forma predeterminada, todas las pistas de la secuencia de vídeo están seleccionadas. Para ver el cambio de velocidad de bits, puede seleccionar la menor velocidad de bits disponible y, a continuación, la mayor. Debe hacer clic en Submit después de realizar cada uno de los cambios.  Puede ver los cambios de calidad del vídeo.

Ha completado la lección 4.  En esta lección ha agregado la funcionalidad para elegir pistas.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="other-resources"></a>Otros recursos:
* [Creación de una aplicación JavaScript de Smooth Streaming para Windows 8 con características avanzadas](https://blogs.iis.net/cenkd/archive/2012/08/10/how-to-build-a-smooth-streaming-windows-8-javascript-application-with-advanced-features.aspx)
* [Información técnica sobre Smooth Streaming](https://www.iis.net/learn/media/on-demand-smooth-streaming/smooth-streaming-technical-overview)

[PlayerApplication]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-1.png
[CodeViewPic]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-2.png
