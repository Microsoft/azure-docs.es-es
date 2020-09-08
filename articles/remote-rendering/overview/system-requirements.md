---
title: Requisitos del sistema
description: Enumera los requisitos del sistema para Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/03/2020
ms.topic: article
ms.openlocfilehash: 81480bea735017d3fc59e9c6cf126c2146a0c968
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798472"
---
# <a name="system-requirements"></a>Requisitos del sistema

> [!IMPORTANT]
> **Azure Remote Rendering** está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este capítulo se enumeran los requisitos mínimos del sistema para trabajar con *Azure Remote Rendering* (ARR).

## <a name="development-pc"></a>PC de desarrollo

* Windows 10 versión 1903 o posterior.
* Controladores gráficos actualizados.
* Opcional: [Descodificador de vídeo de hardware de H265](https://www.microsoft.com/p/hevc-video-extensions/9nmzlz57r3t7), si desea usar la versión preliminar local del contenido representado de forma remota (por ejemplo, en Unity).

> [!IMPORTANT]
> Windows Update no siempre entrega los controladores de GPU más recientes; consulte el sitio web del fabricante de la GPU para obtenerlos:
>
> * [Controladores AMD](https://www.amd.com/en/support)
> * [Controladores Intel](https://www.intel.com/content/www/us/en/support/detect.html)
> * [Controladores NVIDIA](https://www.nvidia.com/Download/index.aspx)

En la tabla siguiente se enumeran las GPU que admiten la descodificación de vídeo de hardware H265.

| Fabricante de la GPU | Modelos admitidos |
|-----------|:-----------|
| NVIDIA | Compruebe la **matriz de compatibilidad de NVDEC** [en la parte inferior de esta página](https://developer.nvidia.com/video-encode-decode-gpu-support-matrix). La GPU necesita un sí (YES) en la columna **H.265 4:2:0 8 bits**. |
| AMD | GPU con al menos la versión 6 del [descodificador de vídeo unificado](https://en.wikipedia.org/wiki/Unified_Video_Decoder#UVD_6) de AMD. |
| Intel | Skylake y CPU más recientes |

Aunque es posible que esté instalado el códec H265 correcto, las propiedades de seguridad de los archivos DLL del códec pueden producir errores de inicialización del códec. En la [guía de solución de problemas](../resources/troubleshoot.md#h265-codec-not-available) se describen los pasos para resolver este problema. El problema de DLL solo puede producirse cuando se usa el servicio en una aplicación de escritorio, por ejemplo, en Unity.

## <a name="devices"></a>Dispositivos

Actualmente, Azure Remote Rendering solo admite **HoloLens 2** y el Escritorio de Windows como dispositivos de destino. Consulte la sección sobre las [limitaciones de la plataforma](../reference/limits.md#platform-limitations).

Es importante usar el códec HEVC más reciente, ya que las versiones más recientes tienen mejoras significativas en la latencia. Para comprobar qué versión está instalada en el dispositivo:

1. Inicie **Microsoft Store**.
1. Haga clic en el botón **"..."** en la esquina superior derecha.
1. Seleccione **Descargas y actualizaciones**.
1. Busque en la lista las **extensiones de vídeo HEVC del fabricante del dispositivo**. Si este elemento no aparece en actualizaciones, la versión más reciente ya está instalada.
1. Asegúrese de que el códec de la lista tiene al menos la versión **1.0.21821.0**.
1. Haga clic en el botón **Obtener actualizaciones** y espere a que se instalen.

## <a name="network"></a>Red

Una conexión de red estable y de baja latencia es fundamental para una buena experiencia del usuario.

Consulte el capítulo dedicado a los [requisitos de red](../reference/network-requirements.md).

Para solucionar problemas de red, consulte la [Guía de solución de problemas](../resources/troubleshoot.md#unstable-holograms).

## <a name="software"></a>Software

El software siguiente debe estar instalado:

* La versión más reciente de **Visual Studio 2019** [(descargar)](https://visualstudio.microsoft.com/vs/older-downloads/)
* [Visual Studio Tools para Mixed Reality](https://docs.microsoft.com/windows/mixed-reality/install-the-tools). En concreto, las instalaciones de *carga de trabajo* siguientes son obligatorias:
  * **Desarrollo para el escritorio con C++**
  * **Desarrollo de Plataforma universal de Windows (UWP)**
* **Windows SDK 10.0.18362.0** [(descargar)](https://developer.microsoft.com/windows/downloads/windows-10-sdk)
* **GIT** [(descargar)](https://git-scm.com/downloads)
* Opcional: para ver la secuencia de vídeo desde el servidor en un equipo de escritorio, necesita las **extensiones de vídeo de HEVC** [(vínculo de Microsoft Store)](https://www.microsoft.com/p/hevc-video-extensions/9nmzlz57r3t7). Asegúrese de que se ha instalado la última versión comprobando si hay actualizaciones en el almacén.

## <a name="unity"></a>Unity

Para el desarrollo con Unity, instale

* Unity 2019.3.1 [(descargar)](https://unity3d.com/get-unity/download)
* Instale estos módulos en Unity:
  * **UWP**: compatibilidad con la compilación de la Plataforma universal de Windows
  * **IL2CPP**: compatibilidad con la compilación de Windows (IL2CPP)

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Representación de un modelo con Unity](../quickstarts/render-model.md)
