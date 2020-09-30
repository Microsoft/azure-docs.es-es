---
title: Notas de la versión de la actualización 2 de la serie StorSimple 8000 | Microsoft Docs
description: Describe las nuevas características, problemas y soluciones alternativas de la actualización 2 de la serie StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: e2c8bffd-7fc5-4b77-b632-a4f59edacc3a
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: alkohli
ms.openlocfilehash: 53be638c46685dd46a5afb5ed56f9a76ed7b5fd6
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90055867"
---
# <a name="storsimple-8000-series-update-2-release-notes"></a>Notas de la versión de la actualización 2 de la serie StorSimple 8000

## <a name="overview"></a>Información general
Las siguientes notas de la versión describen las características nuevas e identifican los problemas críticos abiertos de la actualización 2 de la serie StorSimple 8000. También contienen una lista de las actualizaciones de software, controlador y firmware de StorSimple incluidas en esta versión. 

La actualización 2 puede aplicarse a cualquier dispositivo de StorSimple que ejecute software de versión (GA) o las actualizaciones de la 0.1 a la 1.2. La versión de dispositivo asociada a la actualización 2 es 6.3.9600.17673.

Revise la información contenida en las notas de la versión antes de implementar la actualización de la solución de StorSimple.

> [!IMPORTANT]
> * Esta actualización tarda aproximadamente 4-7 horas en instalarse (incluidas las actualizaciones de Windows). 
> * La actualización 2 tiene actualizaciones de software, controlador LSI y firmware de SSD.
> * Para las nuevas versiones, no podrán ver las actualizaciones de inmediato porque hacemos una implementación por fases de las actualizaciones. Espere unos días y luego busque actualizaciones de nuevo ya que estas estarán disponibles pronto.
> 
> 

## <a name="whats-new-in-update-2"></a>Novedades de la actualización 2
La actualización 2 presenta las siguientes características nuevas.

* **Volúmenes anclados localmente** : en las versiones anteriores de la serie StorSimple 8000, los bloques de datos se almacenan en capas en la nube en función de su uso. No había ninguna forma de garantizar que los bloques permanecerían en el equipo local. En la actualización 2, cuando se crea un volumen, puede designar un volumen como anclado localmente, y los datos principales de dicho volumen no se almacenarán en capas en la nube. Las instantáneas de los volúmenes anclados localmente aún se copiarán en la nube para las copias de seguridad, a fin de que la nube pueda utilizarse a efectos de movilidad de datos y de recuperación ante desastres. Además, puede cambiar el tipo de volumen (es decir, convertir los volúmenes en capas en volúmenes anclados localmente, y viceversa). 
* **Mejoras del dispositivo virtual de StorSimple** : anteriormente, la serie StorSimple 8000 colocaba el dispositivo virtual como una solución de desarrollo y pruebas o de recuperación ante desastres. Solo había un modelo de dispositivo virtual (modelo 1100). La actualización 2 presenta dos modelos de dispositivo virtual: 
  
  * 8010 (anteriormente denominado 1100): sin cambios; tiene una capacidad de 30 TB y usa el almacenamiento estándar de Azure.
  * 8020: tiene una capacidad de 64 TB y usa el almacenamiento premium de Azure para mejorar el rendimiento.
    
    Hay un solo VHD para ambos modelos de dispositivo virtual (8010/8020). Cuando se inicia por primera vez el dispositivo virtual, detecta los parámetros de la plataforma y se aplica a la versión correcta del modelo.
* **Mejoras de red** : la actualización 2 contiene las siguientes mejoras de red:
  
  * Varias NIC pueden habilitarse para la nube para que la conmutación por error pueda realizarse si se produce un error en una tarjeta NIC.
  * Mejoras de enrutamiento con métricas fijas para bloques habilitados para la nube.
  * Reintento en línea de recursos con errores antes de una conmutación por error.
  * Nuevas alertas para errores de servicio.
* **Mejoras en las actualizaciones** : en la actualización 1.2 y en las versiones anteriores, la serie StorSimple 8000 se ha actualizado a través de dos canales: Windows Update para la agrupación en clústeres, iSCSI, y así sucesivamente, y Microsoft Update para los archivos binarios y el firmware.
    La actualización 2 usa Microsoft Update para todos los paquetes de actualizaciones. Esto debería reducir el tiempo de la aplicación de revisiones o de la realización de conmutaciones por error. 
* **Actualizaciones de firmware** : se incluyen las siguientes actualizaciones del firmware:
  
  * LSI: versión del producto 2.00.72.10 lsi_sas2.sys
  * SSD solo (ninguna actualización de la unidad de disco duro): XMGG, XGEG, KZ50, F6C2 y VR08
* **Soporte técnico proactivo** : la actualización 2 permite a Microsoft extraer información de diagnóstico adicional del dispositivo. Cuando nuestro equipo de operaciones identifica dispositivos que están teniendo problemas, estamos mejor equipados para recopilar información del dispositivo y diagnosticar problemas. **Al aceptar la actualización 2, nos permite proporcionar este soporte técnico proactivo**.    

## <a name="issues-fixed-in-update-2"></a>Problemas corregidos en la actualización 2
En las tablas siguientes se ofrece un resumen de los problemas corregidos en la actualización 2.    

| No. | Característica | Problema | Se aplica a un dispositivo físico | Se aplica a un dispositivo virtual |
| --- | --- | --- | --- | --- |
| 1 |Interfaces de red |Después de actualizar a la actualización 1, el servicio del administrador de StorSimple informó que se habían producido errores en los puertos Data2 y Data3 de un controlador. Ahora se ha corregido. |Sí |No |
| 2 |Actualizaciones |Una vez realizada la actualización 1, se generaron alertas audibles en el Portal de Azure clásico en varios dispositivos. Ahora se ha corregido. |Sí |No |
| 3 |Autenticación de Openstack |Al utilizar Openstack como proveedor de servicios en la nube, podría recibir un error que indica que la cadena de autenticación de la nube es demasiado larga. Esto se ha solucionado. |Sí |No |

## <a name="known-issues-in-update-2"></a>Problemas conocidos de la actualización 2
En la tabla siguiente se proporciona un resumen de los problemas conocidos de esta versión.

| No. | Característica | Problema | Comentarios / solución alternativa | Se aplica a un dispositivo físico | Se aplica a un dispositivo virtual |
| --- | --- | --- | --- | --- | --- |
| 1 |Cuórum de disco |En raras ocasiones, si se desconecta la mayoría de los discos en el revestimiento de EBOD de un dispositivo 8600 y no se produce un cuórum de disco, el grupo de almacenamiento se desconectará. Seguirá desconectado incluso si se vuelven a conectar los discos. |Necesitará reiniciar el dispositivo. Si el problema persiste, póngase en contacto con el soporte técnico de Microsoft para conocer los pasos siguientes. |Sí |No |
| 2 |Identificador de controlador incorrecto |Cuando se realiza un reemplazo de controlador, el controlador 0 puede aparecer como controlador 1. Durante el reemplazo de controlador, cuando se carga la imagen desde el nodo del mismo nivel, el identificador de controlador puede mostrarse inicialmente como el identificador del controlador del mismo nivel. En raras ocasiones, este comportamiento también puede aparecer después del reinicio del sistema. |No se requiere ninguna acción del usuario. La situación se solucionará una vez completado el reemplazo del controlador. |Sí |No |
| 3 |Cuentas de almacenamiento |El uso del servicio de almacenamiento para eliminar la cuenta de almacenamiento es un escenario no admitido. Esto provocará una situación en la que no se pueden recuperar los datos de usuario. | |Sí |Sí |
| 4 |Conmutación por error del dispositivo |No se admiten varias conmutaciones por error de un contenedor de volúmenes del mismo dispositivo de origen a diferentes dispositivos de destino. La conmutación por error de un único dispositivo inactivo a varios dispositivos hará que los contenedores de volúmenes del primer dispositivo conmutado por error pierdan la propiedad de los datos. Después de este tipo de conmutación por error, estos contenedores de volúmenes aparecerán o se comportarán de forma diferente cuando se visualicen en el Portal de Azure clásico. | |Sí |No |
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
| 15 |Cmdlets de Azure PowerShell y volúmenes anclados localmente |No se puede crear un volumen anclado localmente mediante los cmdlets de Azure PowerShell. (Se puede almacenar en capas cualquier volumen creado con Azure PowerShell). |Utilice siempre el servicio del administrador de StorSimple para configurar volúmenes anclados localmente. |Sí |No |
| 16 |Espacio disponible para los volúmenes anclados localmente |Si elimina un volumen anclado localmente, puede que el espacio disponible para los volúmenes nuevos no se actualice inmediatamente. El servicio del administrador de StorSimple actualiza el espacio local disponible aproximadamente cada hora. |Espere una hora antes de intentar crear el nuevo volumen. |Sí |No |
| 17 |Volúmenes anclados localmente |El trabajo de restauración expone la copia de seguridad de instantánea temporal en el catálogo de copia de seguridad, pero solo para la duración de la tarea de restauración. Además, expone un grupo de discos virtuales con el prefijo **tmpCollection** en la página **Directivas de copia de seguridad**, pero solo para la duración del trabajo de restauración. |Este comportamiento puede producirse si el trabajo de restauración ha anclado solo localmente volúmenes o una mezcla de volúmenes nivelados y anclados localmente. Si el trabajo de restauración incluye solo volúmenes nivelados, este comportamiento no se producirá. No se requiere ninguna intervención del usuario. |Sí |No |
| 18 |Volúmenes anclados localmente |Si cancela un trabajo de restauración y se produce una conmutación por error de controlador inmediatamente después, el trabajo de restauración mostrará **Cancelado** en lugar de **Erróneo**. Si se produce un error en un trabajo de restauración y se produce una conmutación por error de controlador inmediatamente después, el trabajo de restauración mostrará **Cancelado** en lugar de **Erróneo**. |Este comportamiento puede producirse si el trabajo de restauración ha anclado solo localmente volúmenes o una mezcla de volúmenes nivelados y anclados localmente. Si el trabajo de restauración incluye solo volúmenes nivelados, este comportamiento no se producirá. No se requiere ninguna intervención del usuario. |Sí |No |
| 19 |Volúmenes anclados localmente |Si se cancela un trabajo de restauración o si se produce un error en una restauración y, a continuación, se produce una conmutación por error de controlador, aparece un trabajo de restauración adicional en la página **Trabajos** . |Este comportamiento puede producirse si el trabajo de restauración ha anclado solo localmente volúmenes o una mezcla de volúmenes nivelados y anclados localmente. Si el trabajo de restauración incluye solo volúmenes nivelados, este comportamiento no se producirá. No se requiere ninguna intervención del usuario. |Sí |No |
| 20 |Volúmenes anclados localmente |Si intenta convertir un volumen en niveles (creado y clonado con la actualización 1.2 o anterior) a un volumen anclado localmente y el dispositivo ya no tiene espacio o si hay una interrupción en la nube, es posible que los clones resulten dañados. |Este problema solo se produce con los volúmenes que se crearon y clonaron con software anterior a la actualización 2. Este debería ser un escenario poco frecuente. | | |
| 21 |Conversión de volumen |No actualice los ACR anexados a un volumen mientras se realiza una conversión de volumen (de un volumen en niveles a un volumen anclado localmente o viceversa). Actualizar los ACR podría dañar los datos. |En caso de ser necesario, actualice los ACR antes de realizar la conversión de volumen y no realice ninguna actualización ACR adicional mientras la conversión esté en progreso. | | |

## <a name="controller-and-firmware-updates-in-update-2"></a>Actualizaciones del firmware y del controlador en la actualización 2
Esta versión actualiza el controlador y el firmware del disco en el dispositivo.

* Para obtener más información sobre la actualización del firmware de LSI, consulte el artículo 3121900 de Microsoft Knowledge Base. 
* Para obtener más información sobre la actualización del firmware del disco, consulte el artículo 3121899 de Microsoft Knowledge Base.

## <a name="virtual-device-updates-in-update-2"></a>Actualizaciones del dispositivo virtual en la actualización 2
No se puede aplicar esta actualización al dispositivo virtual. Deben crearse nuevos dispositivos virtuales. 

## <a name="next-step"></a>Paso siguiente
Obtenga información sobre cómo [instalar la actualización 2](storsimple-install-update-2.md) en el dispositivo de StorSimple.

