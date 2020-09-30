---
title: Solución de problemas
description: Información de solución de problemas de Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/25/2020
ms.topic: troubleshooting
ms.openlocfilehash: 14184c09cc9d5eebab7f33323cd8ce587fdf9e88
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89014598"
---
# <a name="troubleshoot"></a>Solución de problemas

En esta página se enumeran los problemas comunes que interfieren con Azure Remote Rendering y las maneras de resolverlos.

## <a name="cant-link-storage-account-to-arr-account"></a>No se puede vincular la cuenta de almacenamiento a la cuenta de ARR

A veces, durante la [vinculación de una cuenta de almacenamiento](../how-tos/create-an-account.md#link-storage-accounts) la cuenta de Remote Rendering no aparece en la lista. Para corregir este problema, vaya a la cuenta de ARR en el Azure Portal y seleccione **Identidad** en el grupo de **Configuración** de la izquierda. Asegúrese de que **estado** se estableció **Encendido**.
![Depurador de fotograma de Unity](./media/troubleshoot-portal-identity.png)

## <a name="client-cant-connect-to-server"></a>El cliente no se puede conectar al servidor

Asegúrese de que los firewalls (en el dispositivo, dentro de los enrutadores, etc.) no bloqueen los puertos siguientes:

* **50051 (TCP)** : necesario para la conexión inicial (protocolo de enlace HTTP)
* **8266 (TCP + UDP)** : necesario para la transferencia de datos
* **5000 (TCP)** , **5433 (TCP)** , **8443 (TCP)** : necesario para [ArrInspector](tools/arr-inspector.md)

## <a name="error-disconnected-videoformatnotavailable"></a>Error "`Disconnected: VideoFormatNotAvailable`"

Compruebe que la GPU admita la descodificación de vídeo de hardware. Consulte [Equipo de desarrollo](../overview/system-requirements.md#development-pc).

Si está trabajando en un equipo portátil con dos GPU, es posible que la GPU que se ejecuta de forma predeterminada no proporcione la funcionalidad de descodificación de vídeo de hardware. Si es así, intente forzar la aplicación para que use la otra GPU. Esto suele ser posible en la configuración del controlador de GPU.

## <a name="retrieve-sessionconversion-status-fails"></a>Error al recuperar el estado de la sesión o conversión

Enviar comandos de la API REST con demasiada frecuencia hará que el servidor se limite y devuelva un error con el tiempo. El código de estado HTTP en caso de limitación es 429 ("demasiadas solicitudes"). Como regla general, debería haber un retraso de entre **5 y 10 segundos entre las llamadas subsiguientes**.

Tenga en cuenta que este límite no solo afecta a las llamadas a API REST cuando se realizan directamente, sino también a sus homólogos de C#/C++, como `Session.GetPropertiesAsync`, `Session.RenewAsync` o `Frontend.GetAssetConversionStatusAsync`.

Si experimenta una limitación en el lado del servidor, cambie el código para que realice las llamadas con menos frecuencia. El servidor restablecerá el estado de limitación cada minuto, por lo que podrá volver a ejecutar el código después de un minuto.

## <a name="h265-codec-not-available"></a>Códec H265 no disponible

Hay dos razones por las que el servidor puede rechazar la conexión con un error `codec not available`.

**El códec H265 no está instalado:**

En primer lugar, asegúrese de instalar **Extensiones de vídeo HEVC** como se menciona en la sección [Software](../overview/system-requirements.md#software) de los requisitos del sistema.

Si sigue experimentando problemas, asegúrese de que la tarjeta gráfica admite H265 y de que tiene instalado el controlador de gráficos más reciente. Consulte la sección [Equipo de desarrollo](../overview/system-requirements.md#development-pc) de los requisitos del sistema para obtener información específica del proveedor.

**El códec está instalado, pero no se puede usar:**

La razón de este problema es una configuración de seguridad incorrecta en los archivos DLL. Este problema no se manifiesta al intentar ver vídeos codificados con H265. La reinstalación del códec tampoco soluciona el problema. En su lugar, realice los pasos siguientes:

1. Abra una instancia de **PowerShell con derechos de administrador** y ejecute el comando:

    ```PowerShell
    Get-AppxPackage -Name Microsoft.HEVCVideoExtension
    ```
  
    Ese comando debería generar el objeto `InstallLocation` del códec, similar al siguiente:
  
    ```cmd
    InstallLocation   : C:\Program Files\WindowsApps\Microsoft.HEVCVideoExtension_1.0.23254.0_x64__5wasdgertewe
    ```

1. Abra esa carpeta en el Explorador de Windows.
1. Debería contener las subcarpetas **x86** y **x64**. Haga clic con el botón derecho en una de las carpetas y elija **Propiedades**.
    1. Seleccione la pestaña **Seguridad** y, después, haga clic en el botón **Opciones avanzadas**.
    1. Haga clic en **Cambiar** en **Propietario**.
    1. Escriba **Administradores** en el campo de texto.
    1. Haga clic en **Comprobar nombres** y luego en **Aceptar**.
1. Repita los pasos anteriores para la otra carpeta.
1. Repita también los pasos anteriores en cada archivo DLL de ambas carpetas. Debe haber cuatro archivos DLL en total.

Para comprobar que la configuración ahora es correcta, realice el procedimiento siguiente con cada archivo DLL:

1. Seleccione **Propiedades > Seguridad > Editar**.
1. Recorra la lista de todos los **Grupos/Usuarios** y asegúrese de que cada uno tenga el derecho **Lectura y ejecución** establecido (deben tener la marca de verificación en la columna **Permitir**)

## <a name="low-video-quality"></a>Calidad de vídeo baja

La calidad de vídeo puede verse afectada por la calidad de la red o la falta del códec de vídeo H265.

* Consulte los pasos para [identificar problemas de red](#unstable-holograms).
* Consulte los [requisitos del sistema](../overview/system-requirements.md#development-pc) para instalar el controlador de gráficos más reciente.

## <a name="video-recorded-with-mrc-does-not-reflect-the-quality-of-the-live-experience"></a>El vídeo grabado con MRC no refleja la calidad de la experiencia en directo

Se puede grabar un vídeo en Hololens a través de [Captura de realidad mixta (MRC)](https://docs.microsoft.com/windows/mixed-reality/mixed-reality-capture-for-developers). Sin embargo, el vídeo resultante tiene una calidad peor que la experiencia en directo por dos motivos:
* La velocidad de fotogramas de vídeo se limita a 30 Hz en lugar de 60 Hz.
* Las imágenes de vídeo no se someten al paso de procesamiento de [reproyección de la fase final](../overview/features/late-stage-reprojection.md), por lo que el vídeo parece más entrecortado.

Ambas limitaciones son inherentes de la técnica de grabación.

## <a name="black-screen-after-successful-model-loading"></a>Pantalla negra tras cargar el modelo correctamente

Si está conectado al entorno de ejecución de representación y ha cargado un modelo correctamente, pero después solo ve una pantalla negra después, puede deberse a distintas causas.

Se recomienda realizar las comprobaciones siguientes antes de realizar un análisis más exhaustivo:

* ¿El códec H265 está instalado? Aunque debería haber una reserva para el códec H264, hemos detectado casos en los que esta reserva no funcionó correctamente. Consulte los [requisitos del sistema](../overview/system-requirements.md#development-pc) para instalar el controlador de gráficos más reciente.
* Al usar un proyecto de Unity, cierre Unity, elimine las carpetas temporales *library* y *obj* del directorio del proyecto, y cargue o cree el proyecto de nuevo. En algunos casos, los datos almacenados en caché provocaron el fracaso del ejemplo sin ningún motivo aparente.

Si estos dos pasos no ayudan, debe averiguar si el cliente recibe fotogramas de vídeo. La consulta se puede realizar mediante programación, como se explica en el capítulo [Consultas de rendimiento del lado del servidor](../overview/features/performance-queries.md). El objeto `FrameStatistics struct` tiene un miembro que indica el número de fotogramas de vídeo que se han recibido. Si este número es mayor que 0 y aumenta con el tiempo, el cliente recibe fotogramas de vídeo reales del servidor. Por lo tanto, debe tratarse de un problema en el lado cliente.

### <a name="common-client-side-issues"></a>Problemas comunes del lado cliente

**El modelo supera los límites de la VM seleccionada, específicamente el número máximo de polígonos:**

Vea los [límites de tamaño de servidor](../reference/limits.md#overall-number-of-polygons) específicos.

**El modelo no está dentro del tronco de la cámara:**

En muchos casos, el modelo se muestra correctamente pero se encuentra fuera del tronco de la cámara. Un motivo habitual es que el modelo se ha exportado con una dinamización muy descentrada y el plano de recorte lejano de la cámara lo ha recortado. Ayuda a consultar el cuadro de límite del modelo mediante programación y a visualizar el cuadro con Unity como un cuadro de línea o imprimir sus valores en el registro de depuración.

Además, el proceso de conversión genera un [archivo JSON de salida](../how-tos/conversion/get-information.md) junto con el modelo convertido. Para depurar los problemas de posicionamiento de modelos, vale la pena examinar la entrada `boundingBox` de la sección [outputStatistics](../how-tos/conversion/get-information.md#the-outputstatistics-section):

```JSON
{
    ...
    "outputStatistics": {
        ...
        "boundingBox": {
            "min": [
                -43.52,
                -61.775,
                -79.6416
            ],
            "max": [
                43.52,
                61.775,
                79.6416
            ]
        }
    }
}
```

El cuadro de límite se describe como una posición `min` y `max` en el espacio 3D, en metros. Por lo tanto, una coordenada de 1000,0 significa que está a 1 kilómetro del origen.

Puede haber dos problemas con este cuadro de límite que den lugar a una geometría invisible:
* **El cuadro puede estar muy alejado del centro**, de modo que el objeto se recorte por completo debido al recorte del plano lejano. En este caso, los valores `boundingBox` tendrían el siguiente aspecto: `min = [-2000, -5,-5], max = [-1990, 5,5]`, con un gran desplazamiento en el eje x como ejemplo aquí. Para resolver este tipo de problema, habilite la opción `recenterToOrigin` en [Configuración de la conversión de modelos](../how-tos/conversion/configure-model-conversion.md).
* **El cuadro puede centrarse, pero los órdenes de magnitud ser demasiado grandes**. Esto significa que, aunque la cámara se inicie en el centro del modelo, su geometría se recorta en todas las direcciones. En este caso, los valores de `boundingBox` típicos tendrían el siguiente aspecto: `min = [-1000,-1000,-1000], max = [1000,1000,1000]`. El motivo de este tipo de problema suele ser una discrepancia en la escala de unidades. Para compensarla, especifique un [valor de escalado durante la conversión](../how-tos/conversion/configure-model-conversion.md#geometry-parameters) o marque el modelo de origen con las unidades correctas. El escalado también se puede aplicar al nodo raíz al cargar el modelo en tiempo de ejecución.

**La canalización de representación de Unity no incluye los enlaces de representación:**

Azure Remote Rendering se enlaza a la canalización de representación de Unity para realizar la composición de fotogramas con el vídeo y para volver a proyectarla. Para comprobar que estos enlaces existen, abra el menú *:::no-loc text="Window > Analysis > Frame debugger":::* . Habilite esta opción y asegúrese de que existen dos entradas de `HolographicRemotingCallbackPass` en la canalización:

![Unity Frame Debugger](./media/troubleshoot-unity-pipeline.png)

## <a name="checkerboard-pattern-is-rendered-after-model-loading"></a>El patrón de tablero de damas se representa después de cargar el modelo

Si la imagen representada tiene el siguiente aspecto: ![Tablero de damas](../reference/media/checkerboard.png) A continuación, el representador alcanza los [límites del polígono para el tamaño de configuración estándar](../reference/vm-sizes.md). Para mitigarlo, cambie al tamaño de configuración **prémium** o reduzca el número de polígonos visibles.

## <a name="the-rendered-image-in-unity-is-upside-down"></a>La imagen representada en Unity está al revés

Asegúrese de seguir el [Tutorial de Unity: Visualización de modelos remotos](../tutorials/unity/view-remote-models/view-remote-models.md) con exactitud. Una imagen al revés indica que Unity debe crear un destino de representación fuera de la pantalla. Este comportamiento no se admite en la actualidad y ocasiona un gran impacto en el rendimiento de HoloLens 2.

Los motivos de este problema podrían ser MSAA, HDR o la habilitación del procesamiento posterior. Asegúrese de que el perfil de baja calidad esté seleccionado y establecido como predeterminado en Unity. Para ello, vaya a *Editar > Configuración del proyecto... > Calidad*.

## <a name="unity-code-using-the-remote-rendering-api-doesnt-compile"></a>El código de Unity que usa la API de Remote Rendering no se compila

### <a name="use-debug-when-compiling-for-unity-editor"></a>Uso de Depurar al compilar para el Editor de Unity

Cambie el *tipo de compilación* de la solución Unity a **Depuración**. Al probar ARR en el editor de Unity, el objeto `UNITY_EDITOR` de definición solo está disponible en las compilaciones de tipo "Depuración". Tenga en cuenta que esto no está relacionado con el tipo de compilación usado para las [aplicaciones implementadas](../quickstarts/deploy-to-hololens.md), donde debe preferirá las compilaciones de tipo "Versión".

### <a name="compile-failures-when-compiling-unity-samples-for-hololens-2"></a>Errores de compilación al compilar ejemplos de Unity para HoloLens 2

Detectamos errores falsos al intentar compilar los ejemplos de Unity (quickstart, ShowCaseApp, …) para HoloLens 2. Visual Studio notifica que no puede copiar algunos archivos, aunque están ahí. Si ve este problema:
* Quite todos los archivos temporales de Unity del proyecto y vuelva a intentarlo. Es decir, cierre Unity, elimine la *biblioteca* y carpetas *obj* temporales del directorio del proyecto, y cargue o cree el proyecto de nuevo.
* Asegúrese de que los proyectos se encuentran en un directorio en el disco con una ruta de acceso razonablemente corta, ya que el paso de copia a veces parece tener problemas con nombres de archivo largos.
* Si eso no ayuda, podría deberse a que MS Sense interfiere con el paso de copia. Para configurar una excepción, ejecute este comando del Registro desde la línea de comandos (requiere derechos de administrador):
    ```cmd
    reg.exe ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection" /v groupIds /t REG_SZ /d "Unity”
    ```
    
### <a name="arm64-builds-for-unity-projects-fail-because-audiopluginmshrtfdll-is-missing"></a>Se producen errores en las compilaciones Arm64 para proyectos de Unity porque falta AudioPluginMsHRTF.dll

`AudioPluginMsHRTF.dll` para Arm64 se agregó al paquete de *Windows Mixed Reality* *(com.unity.xr.windowsmr.metro)* en la versión 3.0.1. Asegúrese de que tiene instalada la versión 3.0.1 o una versión posterior a través del administrador de paquetes de Unity. En la barra de menús de Unity, vaya a *Ventana > Administrador de paquetes* y busque el paquete *Windows Mixed Reality*.

## <a name="unstable-holograms"></a>Hologramas inestables

En caso de que parezca que los objetos representados se mueven con movimientos de cabeza, es posible que experimente problemas de *reproyección de fases con demora* (LSR). Consulte la sección sobre la [reproyección de fases con demora](../overview/features/late-stage-reprojection.md) para obtener instrucciones sobre cómo enfocar esta situación.

Otra causa de los hologramas inestables (hologramas con tambaleo, deformación, vibración o saltos) puede ser una conectividad de red deficiente, en particular, un ancho de banda de red insuficiente o una latencia demasiado alta. Un buen indicador de la calidad de la conexión de red es el valor de [estadísticas de rendimiento](../overview/features/performance-queries.md)`ARRServiceStats.VideoFramesReused`. Los fotogramas reutilizados indican situaciones en las que un fotograma de vídeo antiguo necesitaba reutilizarse en el lado cliente porque no había ningún nuevo fotograma de vídeo disponible (por ejemplo, debido a la pérdida de paquetes o a variaciones en la latencia de red). Si `ARRServiceStats.VideoFramesReused` suele ser mayor que cero, significa que hay un problema de red.

Otro valor que se debe examinar es `ARRServiceStats.LatencyPoseToReceiveAvg`. Debe estar siempre por debajo de 100 ms. Si ve valores más altos, significa que está conectado a un centro de datos que está demasiado lejos.

Para obtener una lista de posibles mitigaciones, consulte las [instrucciones sobre la conectividad de red](../reference/network-requirements.md#guidelines-for-network-connectivity).

## <a name="z-fighting"></a>Z-fighting

Si bien ARR ofrece la [funcionalidad de mitigación de Z-fighting](../overview/features/z-fighting-mitigation.md), Z-fighting todavía puede mostrarse en la escena. Esta guía pretende solucionar estos problemas restantes.

### <a name="recommended-steps"></a>Pasos recomendados

Use el siguiente flujo de trabajo para mitigar Z-fighting:

1. Pruebe la escena con la configuración predeterminada de ARR (mitigación de Z-fighting activada).

1. Deshabilite la mitigación de Z-fighting mediante su [API](../overview/features/z-fighting-mitigation.md). 

1. Cambie el plano de cámara de cera y lejos a un intervalo más cercano.

1. Solucione los problemas de escena mediante la sección siguiente.

### <a name="investigating-remaining-z-fighting"></a>Investigación del Z-fighting restante

Si se han agotado los pasos anteriores y el Z-fighting restante es inaceptable, es necesario investigar la causa subyacente del Z-fighting. Como se indica en la [página de la característica de mitigación de Z-fighting](../overview/features/z-fighting-mitigation.md), hay dos razones principales para el Z-fighting: la pérdida de precisión de profundidad en el extremo del rango de profundidad y las superficies que forman una intersección siendo coplanarias. La pérdida de precisión de profundidad es una eventualidad matemática y solo puede mitigarse siguiendo el paso 3 anterior. Las superficies coplanarias indican un defecto del recurso de origen y se corrigen mejor en los datos de origen.

ARR tiene una característica para determinar si las superficies podrían producir Z-fighting: [Resaltado de tablero de damas](../overview/features/z-fighting-mitigation.md). También puede determinar visualmente lo que provoca el Z-fighting. En la primera animación que hay a continuación se muestra un ejemplo de pérdida de precisión de profundidad en la distancia y en la segunda, un ejemplo de superficies prácticamente coplanarias:

![Z-fighting debido a la precisión de profundidad](./media/depth-precision-z-fighting.gif)  ![Z-fighting coplanario](./media/coplanar-z-fighting.gif)

Compare estos ejemplos con su Z-fighting para determinar la causa o, si quiere, siga este flujo de trabajo paso a paso:

1. Coloque la cámara sobre las superficies de Z-fighting para mirar directamente la superficie.
1. Mueva lentamente la cámara hacia atrás, alejándose de las superficies.
1. Si el Z-fighting es visible todo el tiempo, las superficies son perfectamente coplanarias. 
1. Si el Z-fighting es visible la mayor parte del tiempo, las superficies son prácticamente coplanarias.
1. Si el Z-fighting solo es visible desde lejos, la causa es la falta de precisión de profundidad.

Las superficies coplanarias pueden tener varias causas:

* La aplicación de exportación duplicó un objeto debido a un error o a distintos enfoques de flujo de trabajo.

    Compruebe estos problemas con la aplicación correspondiente y el soporte técnico de la aplicación.

* Las superficies se duplican y se voltean para que aparezcan dos caras en los representadores que usan la selección frontal o posterior.

    La importación mediante la [conversión del modelo](../how-tos/conversion/model-conversion.md) determina la lateralidad principal del modelo. Se presupone que el valor predeterminado es la bilateralidad. La superficie se representará como una pared delgada con una iluminación físicamente correcta desde ambos lados. La unilateralidad puede estar implícita en el recurso de origen o bien puede aplicarse de forma explícita en la [conversión del modelo](../how-tos/conversion/model-conversion.md). Además, pero opcionalmente, el [modo unilateral](../overview/features/single-sided-rendering.md) se puede establecer en "normal".

* Los objetos forman una intersección en los recursos de origen.

     Los objetos transformados de forma que algunas de sus superficies se superpongan también crean Z-fighting. Las partes de transformación del árbol de la escena en la escena importada de ARR también pueden ocasionar este problema.

* Las superficies se han creado de forma intencionada para tocarse, como las marcas o el texto en las paredes.



## <a name="next-steps"></a>Pasos siguientes

* [Requisitos del sistema](../overview/system-requirements.md)
* [Requisitos de red](../reference/network-requirements.md)
