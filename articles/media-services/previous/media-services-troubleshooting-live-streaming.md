---
title: Guía de solución de problemas para el streaming en vivo | Microsoft Docs
description: En este artículo se ofrecen sugerencias sobre cómo solucionar problemas de streaming en vivo de Azure Media Services.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 97ad3048fccc4f8ed6baf0f1d5304f1c8d81c3f2
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269240"
---
# <a name="troubleshooting-guide-for-live-streaming"></a>Guía de solución de problemas para el streaming en vivo

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

En este artículo se ofrecen sugerencias sobre cómo solucionar algunos problemas del streaming en vivo.

## <a name="issues-related-to-on-premises-encoders"></a>Problemas relacionados con los codificadores locales
Esta sección se ofrecen sugerencias sobre cómo solucionar problemas relacionados con codificadores locales que están configurados para enviar una secuencia de velocidad de bits única a los canales de AMS que están habilitados para la codificación en directo.

### <a name="problem-would-like-to-see-logs"></a>Problema: Le gustaría ver registros
* **Problema potencial**: no puede encontrar los registros del codificador que podrían ayudar en los problemas de depuración.
  
  * **Telestream Wirecast**: normalmente encontrará los registros en C:\Users\{username}\AppData\Roaming\Wirecast\ 
  * **Elemental Live**: puede encontrar vínculos a los registros en el Portal de administración. Haga clic en **Estadísticas** y luego en **Registros**. En la página **Log Files** (Archivos de registro), verá una lista de registros para todos los elementos de LiveEvent; seleccione el que coincida con su sesión actual. 
  * **Flash Media Live Encoder**: puede encontrar el **directorio de registro...** desplazándose hasta la pestaña **Encoding Log** (Registro de codificación).

### <a name="problem-there-is-no-option-for-outputting-a-progressive-stream"></a>Problema: no hay ninguna opción para generar una secuencia progresiva
* **Problema potencial**: el codificador que se usa no deshace el entrelazado automáticamente. 
  
    **Pasos para solucionar problemas**: busque una opción de eliminación de entrelazado dentro de la interfaz del codificador. Cuando se habilite la opción de eliminación de entrelazado, compruebe de nuevo la configuración de la salida progresiva. 

### <a name="problem-tried-several-encoder-output-settings-and-still-unable-to-connect"></a>Problema: se intentaron varias configuraciones de salida del codificador y todavía no es posible conectarse.
* **Problema potencial**: el canal de codificación Azure no se restableció correctamente. 
  
    **Pasos para solucionar problemas**: asegúrese de que el codificador ya no está insertando en AMS, detenga y restablezca el canal. Cuando se vuelva a ejecutar, intente conectar su codificador con la nueva configuración. Si esto sigue sin solucionar el problema, intente crear un nuevo canal por completo; a veces, los canales se pueden dañar tras varios intentos con error.  
* **Problema potencial**: la configuración del fotograma clave o el tamaño de GOP no son óptimos. 
  
    **Pasos para solucionar problemas**: el intervalo de fotogramas clave o el tamaño de GOP recomendado es de dos segundos. Algunos codificadores calculan esta configuración en el número de fotogramas, mientras que otros usan segundos. Por ejemplo: cuando se envían 30 fps, el tamaño de GOP sería de 60 fotogramas, lo que equivale a 2 segundos.  
* **Problema potencial**: los puertos cerrados están bloqueando la secuencia. 
  
    **Pasos para solucionar problemas**: Al hacer el streaming mediante RTMP, compruebe la configuración del firewall o del proxy para confirmar que los puertos salientes 1935 y 1936 están abiertos. 

> [!NOTE]
> Si después de seguir los pasos de solución de problemas, todavía no puede realizar correctamente la transmisión, envíe una incidencia de soporte técnico mediante Azure Portal.
> 
> 

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

