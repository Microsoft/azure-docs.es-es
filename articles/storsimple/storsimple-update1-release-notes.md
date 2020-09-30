---
title: Notas de la versión de la actualización 1.2 de la serie StorSimple 8000 | Microsoft Docs
description: Describe las nuevas características, problemas y soluciones alternativas de la actualización 1.2 de la serie StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 6c9aae87-6f77-44b8-b7fa-ebbdc9d8517c
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 17e20048c4cb4de2be6fe36be100b472f0b8ee73
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89459970"
---
# <a name="update-12-release-notes-for-your-storsimple-8000-series-device"></a>Actualización de las notas de la versión Update 1.2 del dispositivo StorSimple serie 8000

## <a name="overview"></a>Información general
Las siguientes notas de la versión describen las características nuevas e identifican los problemas críticos abiertos de la actualización 1.2 de la serie StorSimple 8000. También contienen una lista de las actualizaciones de software, controlador y firmware de StorSimple incluidas en esta versión. 

La actualización 1.2 se puede aplicar a cualquier dispositivo de StorSimple que ejecute software de versión (GA), actualización 0.1, actualización 0.2 o actualización 0.3. La actualización 1.2 no está disponible si el dispositivo ejecuta la actualización 1 o la actualización 1.1. Si el dispositivo está ejecutando la versión (GA), por favor [póngase en contacto con Soporte técnico de Microsoft](storsimple-contact-microsoft-support.md) para que le ayude a instalar esta actualización.

En la siguiente tabla se muestran las versiones de software de dispositivo correspondientes a las actualizaciones de 1, 1.1 y 1.2.

| Si ejecuta la actualización... | esta es su versión de software de dispositivo. |
| --- | --- |
| Actualización 1.2 |6.3.9600.17584 |
| Actualización 1.1 |6.3.9600.17521 |
| Actualización 1.0 |6.3.9600.17491 |

Revise la información contenida en las notas de la versión antes de implementar la actualización de la solución de StorSimple. Para obtener más información, consulte cómo instalar [la actualización 1.2 en un dispositivo de StorSimple](storsimple-install-update-1.md). 

> [!IMPORTANT]
> * Esta actualización tarda aproximadamente 5-10 horas en instalarse (incluidas las actualizaciones de Windows). 
> * La actualización 1.2 tiene actualizaciones de software, controlador LSI y firmware de disco. Para instalar, siga las instrucciones para [instalar la actualización 1.2 en el dispositivo StorSimple](storsimple-install-update-1.md).
> * Para las nuevas versiones, no podrán ver las actualizaciones de inmediato porque hacemos una implementación por fases de las actualizaciones. Busque actualizaciones de nuevo en unos días ya que estas estarán disponible pronto.
> 
> 

## <a name="whats-new-in-update-12"></a>Novedades de la actualización 1.2
Estas características aparecieron por primera vez con la actualización 1 que se puso a disposición para un conjunto limitado de usuarios. Con la versión de la actualización 1.2, la mayoría de los usuarios de StorSimple verían las siguientes características nuevas y mejoras:

* **La migración de dispositivos de las series 5000-7000 a la serie 8000** : esta versión incluye una nueva característica de migración que permite a los usuarios de dispositivos de las series 5000-7000 migrar sus datos a un dispositivo StorSimple físico de la serie 8000 o a un dispositivo virtual. La característica de migración tiene dos propuestas de valor claves:                                                                  
  
  * **Continuidad del negocio**, mediante la habilitación de la migración de los datos existentes en los dispositivos de las series 5000-7000 a dispositivos de la serie 8000.
  * **Mejores ofertas de características de los dispositivos de la serie 8000**, como la administración centralizada eficaz de varios dispositivos a través del servicio StorSimple Manager, mejor clase de hardware y firmware actualizado, dispositivos virtuales, movilidad de datos y las características de la estrategia futura.
    
    Para obtener más información sobre cómo migrar un dispositivo de las series 5000-7000 de StorSimple a la serie 8000, consulte la [Guía de migración](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b) . 
* **Disponibilidad en el Portal de Azure Government** : StorSimple está disponible en el Portal de Azure Government. Consulte cómo [implementar un dispositivo StorSimple en el Portal de Azure Government](storsimple-deployment-walkthrough-gov.md).
* **Compatibilidad con otros proveedores de servicios en la nube** : los restantes proveedores de servicios en la nube que se admiten son Amazon S3, Amazon S3 con RRS, HP y OpenStack (beta).
* **Actualización a las API de almacenamiento más recientes** : con esta versión, StorSimple se ha actualizado a las API de los servicios de Azure Storage más recientes. Los dispositivos de la serie StorSimple 8000 que ejecutan las versiones de software anteriores a la actualización 1 (versión, 0.1, 0.2 y 0.3) usan versiones de las API del servicio Azure Storage anteriores al 17 de julio de 2009. Como se indicó en la actualización del [anuncio acerca de la eliminación de versiones del servicio Storage](https://docs.microsoft.com/archive/blogs/windowsazurestorage/microsoft-azure-storage-service-version-removal-update-extension-to-2016), el 1 de agosto de 2016 estas API dejarán de utilizarse. La actualización 1 de la serie StorSimple 8000 debe aplicarse antes del 1 de agosto de 2016. Si no lo hace, los dispositivos de StorSimple dejará de funcionar correctamente.
* **Soporte para el almacenamiento con redundancia de zona (ZRS)** : con la actualización a la versión más reciente de las API de almacenamiento, la serie StorSimple 8000 será compatible con el almacenamiento con redundancia de zona (ZRS), además del almacenamiento con redundancia local (LRS) y el almacenamiento con redundancia geográfica (GRS). Para obtener información detallada sobre ZRS, consulte este [artículo sobre las opciones de redundancia de Azure Storage](../storage/common/storage-redundancy.md) .
* **Mejora de la experiencia de implementación inicial y actualización** : en esta versión, se han mejorado los procesos de instalación y actualización. La instalación a través del Asistente para la instalación se ha mejorado para proporcionar comentarios al usuario si la configuración de red y la configuración del firewall son incorrectas. Se han proporcionado cmdlets de diagnósticos adicionales para ayudarle a solucionar los problemas de red del dispositivo. Para obtener más información acerca de los nuevos cmdlets de diagnóstico utilizados para solucionar problemas, consulte el [artículo sobre la solución de problemas en la implementación](storsimple-troubleshoot-deployment.md) .

## <a name="issues-fixed-in-update-12"></a>Problemas corregidos en la actualización 1.2
En las tablas siguientes se ofrece un resumen de los problemas corregidos en Update 1.2, 1.1 y 1.    

| No. | Característica | Problema | Se ha corregido en la actualización | Se aplica a un dispositivo físico | Se aplica a un dispositivo virtual |
| --- | --- | --- | --- | --- | --- |
| 1 |Windows PowerShell para StorSimple |Cuando un usuario accedió de manera remota al dispositivo StorSimple mediante Windows PowerShell para StorSimple y, a continuación, inició al Asistente para la instalación, se produjo un bloqueo en cuanto se introdujo la IP de Data 0. El error se ha corregido en la actualización 1. |Actualización 1 |Sí |Sí |
| 2 |Restablecimiento de fábrica |En algunos casos, al realizar un restablecimiento de fábrica, el dispositivo StorSimple se bloqueó y apareció este mensaje: **El restablecimiento a los valores de fábrica está en curso (fase 8)** . Esto sucedía si se presionaba CTRL+C mientras el cmdlet estaba en curso. El error se ha corregido. |Actualización 1 |Sí |No |
| 3 |Restablecimiento de fábrica |Después de un error en el restablecimiento de fábrica de un controlador doble, se permitió continuar con el registro del dispositivo. Esto provocó una configuración no compatible del sistema. En la actualización 1, se muestra un mensaje de error y el registro se bloquea en los dispositivos en que se haya producido algún error al realizar un restablecimiento de fábrica. |Actualización 1 |Sí |No |
| 4 |Restablecimiento de fábrica |En algunos casos, se han generado alertas de discrepancia que son falsos positivos. Dejarán de generarse alertas de falta de coincidencia incorrectas en los dispositivos con la actualización 1. |Actualización 1 |Sí |No |
| 5 |Restablecimiento de fábrica |Si un restablecimiento de fábrica se interrumpió antes de finalizar, el dispositivo entró en modo de recuperación, lo que no permitió al usuario tener acceso a Windows PowerShell para StorSimple. El error se ha corregido. |Actualización 1 |Sí |No |
| 6 |Recuperación ante desastres |Se corrigió un error de recuperación ante desastres (DR) en el que se produciría un error de DR durante la detección de las copias de seguridad en el dispositivo de destino. |Actualización 1 |Sí |Sí |
| 7 |LED de supervisión |En determinados casos, los LED de supervisión de la parte trasera del dispositivo no indicaron que el estado es correcto. Se ha desactivado el LED azul. Los LED de DATA 0 y DATA 1 parpadeaban aun cuando las interfaces no se habían configurado. El problema se ha corregido y los LED de supervisión indican el estado correcto. |Actualización 1 |Sí |No |
| 8 |LED de supervisión |En determinados casos, después de aplicar la actualización 1, la luz azul del controlador activo está desactivada, por lo que resulta difícil identificar el controlador activo. Este problema se ha corregido en esta versión de revisión. |Actualización 1.2 |Sí |No |
| 9 |Interfaces de red |En las versiones anteriores, los dispositivos de StorSimple configurados con una puerta de enlace no enrutable podían quedarse sin conexión. En esta versión, la métrica de enrutamiento para Data 0 se ha realizado en los valores más bajos; por lo tanto, aunque otras interfaces de red están habilitadas para la nube, todo el tráfico de nube del dispositivo se enrutará a través de Data 0. |Actualización 1 |Sí |Sí |
| 10 |Copias de seguridad |Se ha corregido un error que se daba en la actualización 1 que causaba el fallo de las copias de seguridad a los 24 días en la versión de revisión de la actualización 1.1. |Actualización 1.1 |Sí |Sí |
| 11 |Copias de seguridad |Un error en las versiones anteriores tuvo como resultado un rendimiento deficiente de las instantáneas en la nube con frecuencias de cambio bajas. Este error se ha corregido en esta versión de revisión. |Actualización 1.2 |Sí |Sí |
| 12 |Actualizaciones |En esta versión de revisión se ha corregido un error en la actualización 1 que notificó una actualización con error y que hizo que los controladores entraran en modo de recuperación. |Actualización 1.2 |Sí |Sí |

## <a name="known-issues-in-update-12"></a>Problemas conocidos de la actualización 1.2
En la tabla siguiente se proporciona un resumen de los problemas conocidos de esta versión.

| No. | Característica | Problema | Comentarios/solución alternativa | Se aplica a un dispositivo físico | Se aplica a un dispositivo virtual |
| --- | --- | --- | --- | --- | --- |
| 1 |Cuórum de disco |En raras ocasiones, si se desconecta la mayoría de los discos en el revestimiento de EBOD de un dispositivo 8600 y no se produce un cuórum de disco, el bloque de almacenamiento se desconectará. Seguirá desconectado incluso si se vuelven a conectar los discos. |Necesitará reiniciar el dispositivo. Si el problema persiste, póngase en contacto con el soporte técnico de Microsoft para conocer los pasos siguientes. |Sí |No |
| 2 |Identificador de controlador incorrecto |Cuando se realiza un reemplazo de controlador, el controlador 0 puede aparecer como controlador 1. Durante el reemplazo de controlador, cuando se carga la imagen desde el nodo del mismo nivel, el identificador de controlador puede mostrarse inicialmente como el identificador del controlador del mismo nivel. En raras ocasiones, este comportamiento también puede aparecer después del reinicio del sistema. |No se requiere ninguna acción del usuario. La situación se solucionará una vez completado el reemplazo del controlador. |Sí |No |
| 3 |Cuentas de almacenamiento |El uso del servicio de almacenamiento para eliminar la cuenta de almacenamiento es un escenario no admitido. Esto provocará una situación en la que no se pueden recuperar los datos de usuario. |Sí |Sí | |
| 4 |Conmutación por error del dispositivo |No se admiten varias conmutaciones por error de un contenedor de volúmenes del mismo dispositivo de origen a diferentes dispositivos de destino. La conmutación por error de dispositivo desde un único dispositivo inactivo a varios dispositivos hará que los contenedores de volúmenes del primer dispositivo conmutado por error pierdan la propiedad de los datos. Después de este tipo de conmutación por error, estos contenedores de volúmenes aparecerán o se comportarán de forma diferente cuando se visualicen en el Portal de Azure clásico. | |Sí |No |
| 5 |Instalación |Durante la instalación del adaptador de StorSimple para SharePoint, deberá proporcionar una dirección IP de dispositivo para que la instalación finalice correctamente. | |Sí |No |
| 6 |Proxy web |Si la configuración de proxy web tiene HTTPS como protocolo especificado, la comunicación de dispositivo a servicio se verá afectada y el dispositivo se desconectará. También se generarán paquetes de compatibilidad en el proceso, que consumen muchos recursos en el dispositivo. |Asegúrese de que la dirección URL del proxy web tiene HTTP como protocolo especificado. Para obtener más información, vaya a [Configurar el proxy web para el dispositivo](storsimple-configure-web-proxy.md). |Sí |No |
| 7 |Proxy web |Si configura y habilita el proxy web en un dispositivo registrado, será necesario reiniciar el controlador activo en el dispositivo. | |Sí |No |
| 8 |Latencia alta de la nube y alta carga de trabajo de E/S |Cuando el dispositivo StorSimple encuentra una combinación de latencias muy altas de la nube (del orden de segundos) y alta carga de trabajo de E/S, los volúmenes del dispositivo pasan a un estado degradado y las operaciones de E/S pueden fallar con el error «el dispositivo no está listo». |Necesitará reiniciar los controladores de dispositivo de forma manual o realizar una conmutación por error del dispositivo para recuperarse de esta situación. |Sí |No |
| 9 |Azure PowerShell |Cuando se usa el cmdlet de StorSimple **Get-AzureStorSimpleStorageAccountCredential &#124; Select-Object -First 1 -Wait** para seleccionar el primer objeto y crear un nuevo objeto **VolumeContainer**, el cmdlet devuelve todos los objetos. |Escriba el cmdlet entre paréntesis, como se indica a continuación: **(Get-Azure-StorSimpleStorageAccountCredential) &#124; Select-Object -First 1 -Wait**. |Sí |Sí |
| 10 |Migración |Cuando se pasan varios contenedores de volúmenes para la migración, el ETA de la copia de seguridad más reciente solo es preciso en el primer contenedor de volúmenes. Además, la migración paralela se iniciará después de que se hayan migrado las cuatro primeras copias de seguridad del primer contenedor de volúmenes. |Se recomienda migrar los contenedores de volúmenes de uno en uno. |Sí |No |
| 11 |Migración |Después de la restauración, los volúmenes no se agregan a la directiva de copia de seguridad ni al grupo de discos virtuales. |Para crear copias de seguridad, será preciso agregar estos volúmenes a una directiva de copia de seguridad. |Sí |Sí |
| 12 |Migración |Una vez completada la migración, el dispositivo de las series 5000/7000 no debe tener acceso a los contenedores de datos migrados. |Cuando la migración finaliza y se envía, se recomienda eliminar los contenedores de datos migrados. |Sí |No |
| 13 |Clonación y recuperación ante desastres |Los dispositivos StorSimple con la actualización 1 no se pueden clonar. Tampoco se puede realizar la recuperación ante desastres en un dispositivo que ejecute el software anterior a la actualización 1. |Para que se puedan realizar estas operaciones, tendrá que implementar la actualización 1 en el dispositivo de destino |Sí |Sí |
| 14 |Migración |En los dispositivos de las series 5000-7000, se puede producir un error en la copia de seguridad de la configuración para la migración cuando haya grupos de volúmenes sin volúmenes asociados. |Elimine todos los grupos de volúmenes vacíos sin volúmenes asociados y vuelva a intentar realizar la copia de seguridad de la configuración. |Sí |No |

## <a name="physical-device-updates-in-update-12"></a>Actualizaciones del dispositivo físico en la actualización 1.2
Si se aplica la actualización 1.2 de la revisión a un dispositivo físico (con las versiones anteriores a Update 1), la versión del software cambiará a 6.3.9600.17584.

## <a name="controller-and-firmware-updates-in-update-12"></a>Actualizaciones de firmware y de controlador en Update 1.2
Esta versión actualiza el controlador y el firmware del disco en el dispositivo.

* Para obtener más información sobre la actualización del controlador SAS, consulte [Actualización 1 para los controladores SAS LSI del dispositivo Microsoft Azure StorSimple](https://support.microsoft.com/kb/3043005). 
* Para obtener más información sobre la actualización del firmware del disco, consulte [Actualización 1 del firmware del disco para el dispositivo de Microsoft Azure StorSimple](https://support.microsoft.com/kb/3063416).

## <a name="virtual-device-updates-in-update-12"></a>Actualizaciones de dispositivo virtual en la actualización 1.2
No se puede aplicar esta actualización al dispositivo virtual. Deben crearse nuevos dispositivos virtuales. 

## <a name="next-steps"></a>Pasos siguientes
* [Instalación de la actualización 1.2 en el dispositivo](storsimple-install-update-1.md).

