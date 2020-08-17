---
title: Recuperación de datos de imagen de Azure Kinect
description: Aprenda a recuperar datos de imagen de Azure Kinect mediante el SDK del sensor Kinect.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, recuperación, sensor, cámara, sdk, profundidad, rgb, imágenes, color, captura, resolución, búfer
ms.openlocfilehash: fed5c7340d287d9103ba35f0fd3d80c0fff6e3ec
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87538921"
---
# <a name="retrieve-azure-kinect-image-data"></a>Recuperación de datos de imagen de Azure Kinect

En esta página se proporcionan detalles sobre cómo recuperar imágenes de Azure Kinect. En el artículo se muestra cómo capturar imágenes coordinadas entre las cámaras de color y de profundidad del dispositivo y cómo acceder a ellas. Para acceder a las imágenes, primero debe abrir y configurar el dispositivo y luego capturar imágenes.
Para configurar y capturar una imagen, debe [buscar y abrir un dispositivo](find-then-open-device.md).

También puede ver el [Ejemplo de transmisión del SDK](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/streaming), donde se muestra cómo usar las funciones de este artículo.

Se explican las siguientes funciones:

- [`k4a_device_start_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaad7a85e1e5471810262442fc4a8e217a.html#gaad7a85e1e5471810262442fc4a8e217a)
- [`k4a_device_get_capture()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4dac757a33657f4d3dbf1ae8b21c158a.html#ga4dac757a33657f4d3dbf1ae8b21c158a)
- [`k4a_capture_get_depth_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafa03513da96bf6b8c254fc11a04ee6d6.html#gafa03513da96bf6b8c254fc11a04ee6d6)
- [`k4a_image_get_buffer()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ef070cf4e543cd0f726478af332546e.html#ga2ef070cf4e543cd0f726478af332546e)
- [`k4a_image_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga27c81863b13fafc3934a32935a014e9f.html#ga27c81863b13fafc3934a32935a014e9f)
- [`k4a_capture_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6)
- [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8)
  
## <a name="configure-and-start-the-device"></a>Configuración e inicio del dispositivo

Las dos cámaras disponibles en el dispositivo Kinect admiten varios modos, resoluciones y formatos de salida. Para obtener una lista completa, vea las [especificaciones de hardware](hardware-specification.md) del kit de desarrollo de Azure Kinect.

La configuración de transmisión se establece mediante valores de la estructura [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

```C
k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
config.camera_fps = K4A_FRAMES_PER_SECOND_30;
config.color_format = K4A_IMAGE_FORMAT_COLOR_MJPG;
config.color_resolution = K4A_COLOR_RESOLUTION_2160P;
config.depth_mode = K4A_DEPTH_MODE_NFOV_UNBINNED;

if (K4A_RESULT_SUCCEEDED != k4a_device_start_cameras(device, &config))
{
    printf("Failed to start device\n");
    goto Exit;
}
```

Una vez iniciadas las cámaras, siguen capturando datos hasta que se llama a [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8) o se cierra el dispositivo.

## <a name="stabilization"></a>Estabilización

Al iniciar dispositivos mediante la característica de sincronización de varios dispositivos, se recomienda encarecidamente hacerlo mediante una configuración de exposición fija.
Con un conjunto de exposición manual, pueden ser necesarias hasta ocho capturas del dispositivo para que las imágenes y la velocidad de fotogramas se estabilicen. Con la exposición automática, pueden ser necesarias hasta 20 capturas para que las imágenes y la velocidad de fotogramas se estabilicen.

## <a name="get-a-capture-from-the-device"></a>Obtención de una captura del dispositivo

Las imágenes se capturan del dispositivo de una manera correlacionada. Cada imagen capturada contiene una imagen de profundidad, una imagen de IR, una imagen de color o una combinación de imágenes.

De forma predeterminada, la API solo devuelve una captura una vez que ha recibido todas las imágenes solicitadas para el modo de transmisión. Puede configurar la API de modo que devuelva capturas parciales con solo imágenes de profundidad o de color en cuanto estén disponibles si borra el parámetro [`synchronized_images_only`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t_a8208974f05d89fc1362c6a0900bdef4d.html#a8208974f05d89fc1362c6a0900bdef4d) de [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

```C
// Capture a depth frame
switch (k4a_device_get_capture(device, &capture, TIMEOUT_IN_MS))
{
case K4A_WAIT_RESULT_SUCCEEDED:
    break;
case K4A_WAIT_RESULT_TIMEOUT:
    printf("Timed out waiting for a capture\n");
    continue;
    break;
case K4A_WAIT_RESULT_FAILED:
    printf("Failed to read a capture\n");
    goto Exit;
}
```

Una vez que la API ha devuelto correctamente una captura, debe llamar a [`k4a_capture_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6) si ha terminado de usar el objeto de captura.

## <a name="get-an-image-from-the-capture"></a>Obtención de una imagen de la captura

Para recuperar una imagen capturada, llame a la función adecuada para cada tipo de imagen. Uno de los valores siguientes:

- [`k4a_capture_get_color_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga683e440b5f22215a2de58d7fa140488c.html#ga683e440b5f22215a2de58d7fa140488c)
- [`k4a_capture_get_depth_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafa03513da96bf6b8c254fc11a04ee6d6.html#gafa03513da96bf6b8c254fc11a04ee6d6)
- [`k4a_capture_get_ir_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga1531c3fa76a7c174b8f2eab24de91794.html#ga1531c3fa76a7c174b8f2eab24de91794)

Debe llamar a [`k4a_image_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga27c81863b13fafc3934a32935a014e9f.html#ga27c81863b13fafc3934a32935a014e9f) en cualquier identificador [`k4a_image_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__image__t.html) devuelto por estas funciones una vez que haya terminado de usar la imagen.

## <a name="access-image-buffers"></a>Acceso a los búferes de imagen

[`k4a_image_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__image__t.html) tiene muchas funciones de descriptores de acceso para obtener propiedades de la imagen.

Para acceder al búfer de memoria de la imagen, use [k4a_image_get_buffer](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ef070cf4e543cd0f726478af332546e.html#ga2ef070cf4e543cd0f726478af332546e).

En el ejemplo siguiente se muestra cómo acceder a una imagen de profundidad capturada. Este mismo principio se aplica a otros tipos de imagen. Pero asegúrese de reemplazar la variable de tipo de imagen por el tipo de imagen correcto, como IR o color.

```C
// Access the depth16 image
k4a_image_t image = k4a_capture_get_depth_image(capture);
if (image != NULL)
{
    printf(" | Depth16 res:%4dx%4d stride:%5d\n",
            k4a_image_get_height_pixels(image),
            k4a_image_get_width_pixels(image),
            k4a_image_get_stride_bytes(image));

    // Release the image
    k4a_image_release(image);
}

// Release the capture
k4a_capture_release(capture);
```

## <a name="next-steps"></a>Pasos siguientes

Ya sabe cómo capturar y coordinar las imágenes de las cámaras entre el color y la profundidad con el dispositivo Azure Kinect. También puede llevar a cabo:

>[!div class="nextstepaction"]
>[Recuperación de muestras del IMU](retrieve-imu-samples.md)

>[!div class="nextstepaction"]
>[Acceso al micrófono](access-mics.md)
