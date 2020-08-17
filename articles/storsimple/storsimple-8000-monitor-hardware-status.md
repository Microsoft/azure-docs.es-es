---
title: Componentes de hardware de la serie StorSimple 8000 y su estado | Microsoft Docs
description: Aprenda a supervisar los componentes de hardware del dispositivo StorSimple a través del servicio StorSimple Device Manager.
services: storsimple
documentationcenter: ''
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/15/2018
ms.author: alkohli
ms.openlocfilehash: 8faf4ca8e27b280a60750d5f349ab0282ed593bf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999503"
---
# <a name="use-the-storsimple-device-manager-service-to-monitor-hardware-components-and-status"></a>Usar el servicio StorSimple Device Manager para supervisar componentes y estados de hardware

## <a name="overview"></a>Información general
En este artículo se describen los distintos componentes físicos y lógicos en el dispositivo de la serie StorSimple 8000 local. También se explica cómo supervisar el estado de los componentes del dispositivo mediante la hoja **Estado y mantenimiento de hardware** del servicio StorSimple Device Manager.

En la hoja **Estado y mantenimiento de hardware** aparece el estado del hardware de todos los componentes del dispositivo StorSimple.

En la lista de componentes para 8100, se describen tres secciones:

* **Componentes compartidos**: no forman parte de los controladores, como las unidades de disco, el revestimiento, los componentes del módulo de alimentación y refrigeración (PCM) y su temperatura, la tensión de línea y los sensores de corriente de línea.
* **Componentes del Controlador 0** : componentes que residen en el Controlador 0, por ejemplo, el controlador, el ampliador y el conector de SAS, los sensores de temperatura del controlador y las diversas interfaces de red.
* **Componentes del Controlador 1** : componentes que constituyen el Controlador 1, similares a los detallados en el Controlador 0.

Un dispositivo 8600 tiene componentes adicionales que se corresponden con el revestimiento de un grupo extendido de discos (EBOD). En la lista de componentes, hay cinco secciones. De estas, hay tres que contienen los componentes en el revestimiento principal y son idénticas a las que se describen para 8100. Hay dos secciones adicionales para el revestimiento de EBOD que describen lo siguiente:

* **Componentes del Controlador 0 de EBOD** : componentes que residen en el revestimiento 0 de EBOD, como el controlador de EBOD, el ampliador y el conector de SAS y los sensores de temperatura del controlador.
* **Componentes del Controlador 1 de EBOD** : componentes que constituyen el revestimiento 1 de EBOD, similares a los detallados en el revestimiento 0 de EBOD.
* **Componentes compartidos del revestimiento de EBOD** : componentes presentes en el revestimiento de EBOD y el PCM que no forman parte del controlador de EBOD.

> [!NOTE]
> **El estado del hardware no está disponible para StorSimple Cloud Appliance (8010/8020).**


## <a name="monitor-the-hardware-status"></a>Supervisión del estado del hardware
Realice los pasos siguientes para ver el estado de hardware de un componente del dispositivo:

1. Vaya a **Dispositivos**y seleccione un dispositivo de StorSimple concreto. Vaya a **Supervisar > Mantenimiento de hardware**.

    ![Captura de pantalla de la hoja Configuración del servicio Administrador de dispositivos. En Monitor, está seleccionado Mantenimiento de hardware.](./media/storsimple-8000-monitor-hardware-status/hw-health1.png)

2. Busque la sección **Componentes de hardware** y elija entre los componentes disponibles. Simplemente haga clic en la etiqueta del componente para expandir la lista y ver el estado de los distintos componentes del dispositivo. Consulte la [lista detallada de componentes del gabinete principal](#component-list-for-primary-enclosure-of-storsimple-device) y la [lista detallada de componentes del receptáculo EBOD](#component-list-for-ebod-enclosure-of-storsimple-device).

    ![Captura de pantalla que muestra un componente seleccionado en la sección de componentes de hardware. Una hoja adyacente muestra el nombre y el estado de los elementos del componente.](./media/storsimple-8000-monitor-hardware-status/hw-health2.png)

3. Use el siguiente código de colores para interpretar el estado de los componentes:
   
   * **Marca de verificación verde**: denota un componente correcto con estado **OK**.
   * **Amarillo**: denota un componente degradado en estado de **Advertencia**.
   * **Signo de exclamación rojo**: denota un componente erróneo que tiene un estado de **Error**.
   * **Blanco con texto negro** : denota un componente que no está presente.
   
   La captura de pantalla siguiente muestra un dispositivo que tiene componentes en estados **OK**, **Advertencia** y **Error**.
       
   ![Captura de pantalla que muestra tres componentes en la sección de componentes de hardware: uno en estado correcto, otro en estado de error y el tercero en estado de advertencia.](./media/storsimple-8000-monitor-hardware-status/hw-health3.png)

   Al expandir la **lista de componentes compartidos**, podemos ver que la NVRAM y el clúster están degradados.

   ![Captura de pantalla que muestra el elemento Componentes compartidos seleccionado. En la hoja adyacente, la NVRAM y el clúster se encuentran en un estado de advertencia.](./media/storsimple-8000-monitor-hardware-status/hw-health5.png)

   Al expandir la lista de **componentes del controlador 1**, podemos ver que el nodo de clúster generó un error.  

   ![Captura de pantalla que muestra el elemento Componentes de controlador 1 seleccionado. En la hoja adyacente, el nodo de clúster está en estado de error.](./media/storsimple-8000-monitor-hardware-status/hw-health4.png)  

4. Si se encuentra un componente que no está en estado **Correcto** , póngase en contacto con el soporte técnico de Microsoft. Si las alertas están habilitadas en el dispositivo, recibirá una alerta por correo electrónico. Si necesita reemplazar un componente de hardware con errores, consulte la [Reemplazo de los componentes de hardware de StorSimple](storsimple-hardware-component-replacement.md).

## <a name="component-list-for-primary-enclosure-of-storsimple-device"></a>Lista de componentes de la caja principal del dispositivo StorSimple
En la tabla siguiente se describen los componentes físicos y lógicos que contiene la caja principal (presente en las series 8100 y 8600) del dispositivo StorSimple local.

| Componente | módulo | Tipo | Location | ¿Unidad reemplazable en campo (FRU)? | Descripción |
| --- | --- | --- | --- | --- | --- |
| Unidad en la ranura [0-11] |Unidades de disco |Física |Compartido |Sí |Hay una línea preesnte para cada una de las unidades SSD o HDD en el revestimiento principal. |
| Sensor de temperatura ambiente |Revestimiento |Física |Compartido |No |Mide la temperatura del chasis. |
| Sensor de temperatura de plano medio |Revestimiento |Física |Compartido |No |Mide la temperatura del plano medio. |
| Alarma audible |Revestimiento |Física |Compartido |No |Indica si el subsistema de alarma audible del chasis es funcional. |
| Revestimiento |Revestimiento |Física |Compartido |Sí |Indica la presencia de un chasis. |
| Configuración del revestimiento |Revestimiento |Física |Compartido |No |Hace referencia al panel frontal del chasis. |
| Sensores de voltaje de línea |PCM |Física |Compartido |No |En numerosos sensores de tensión de línea se muestra el estado, que indica si la tensión medida está dentro de la tolerancia. |
| Sensores de corriente de línea |PCM |Física |Compartido |No |En numerosos sensores de corriente de línea se muestra el estado, que indica si la corriente medida está dentro de la tolerancia. |
| Sensores de temperatura en el PCM |PCM |Física |Compartido |No |En numerosos sensores de temperatura como los sensores Inlet y Hotspot se muestra el estado, que indica si la temperatura medida está dentro de la tolerancia. |
| Fuente de alimentación [0-1] |PCM |Física |Compartido |Sí |Hay una línea presente para cada fuente de alimentación en las dos PCM situadas en la parte posterior del dispositivo. |
| Refrigeración [0-1] |PCM |Física |Compartido |Sí |Hay una línea presente para cada uno de los cuatro ventiladores de refrigeración que residen en los dos PCM. |
| Batería [0-1] |PCM |Física |Compartido |Sí |Hay una línea presente para cada uno de los módulos de batería de reserva que están alojados en el PCM. |
| Metis |N/D |Lógicos |Compartido |N/D |Muestra el estado de las baterías: si necesitan recargarse o se acercan al fin de ciclo de vida. |
| Clúster |N/D |Lógicos |Compartido |N/D |Muestra el estado del clúster que se crea entre los dos módulos de controlador integrados. |
| Nodo de clúster |N/D |Lógicos |Compartido |N/D |Indica el estado del controlador como parte del clúster. |
| Quórum de clúster |N/D |Lógicos | |N/D |Indica la presencia de la pertenencia al disco de mayoría en el grupo de almacenamiento de la unidad de disco duro. |
| Espacio de datos de la unidad de disco duro |N/D |Lógicos |Compartido |N/D |El espacio de almacenamiento que se usa para los datos en el grupo de almacenamiento de la unidad de disco duro (HDD). |
| Espacio de administración de la unidad de disco duro |N/D |Lógicos |Compartido |N/D |El espacio reservado en el grupo de almacenamiento de la unidad de disco duro para tareas de administración. |
| Espacio de quórum de la unidad de disco duro |N/D |Lógicos |Compartido |N/D |El espacio reservado en el grupo de almacenamiento de la unidad de disco duro para el quórum del clúster. |
| Espacio de reemplazo de la unidad de disco duro |N/D |Lógicos |Compartido |N/D |El espacio reservado en el grupo de almacenamiento de la unidad de disco duro para el reemplazo del controlador. |
| Espacio de datos de la unidad de estado sólido |N/D |Lógicos |Compartido |N/D |El espacio de almacenamiento para los datos en el grupo de almacenamiento de la unidad de estado sólido (SSD). |
| Espacio de NVRAM de la unidad de estado sólido |N/D |Lógicos |Compartido |N/D |El espacio de almacenamiento en el grupo de almacenamiento de la unidad de estado sólido dedicado para la lógica NVRAM. |
| Grupo de almacenamiento de la unidad de disco duro |N/D |Lógicos |Compartido |N/D |Muestra el estado del grupo de almacenamiento lógico que se crea desde las unidades de disco duro del dispositivo. |
| Grupo de almacenamiento de la unidad de estado sólido |N/D |Lógicos |Compartido |N/D |Muestra el estado del grupo de almacenamiento lógico que se crea desde las unidades de estado sólido del dispositivo. |
| Controller [0-1] [estado] |E/S |Física |Controller |Sí |Muestra el estado del controlador y si está en modo activo o en espera dentro del chasis. |
| Sensores de temperatura en el controlador |E/S |Física |Controller |No |En numerosos sensores de temperatura como los sensores del módulo de E/S, de temperatura de la CPU, de DIMM y de PCI se muestra el estado, que indica si la temperatura se encuentra dentro de la tolerancia. |
| Ampliador SAS |E/S |Física |Controller |No |Indica el estado del ampliador del SCSI acoplado en serie (SAS), que se usa para conectar el almacenamiento integrado al controlador. |
| Conector SAS [0-1] |E/S |Física |Controller |No |Indica el estado de cada conector SAS, que se usa para conectar el almacenamiento integrado al ampliador SAS. |
| Interconexión de plano medio de SBB |E/S |Física |Controller |No |Indica el estado del conector de plano medio, que se usa para conectar cada controlador al plano medio. |
| Núcleo del procesador |E/S |Física |Controller |No |Indica el estado de los núcleos del procesador en cada controlador. |
| Potencia de la electrónica del revestimiento |E/S |Física |Controller |No |Indica el estado del sistema de alimentación que usa el revestimiento. |
| Diagnóstico de la electrónica del revestimiento |E/S |Física |Controller |No |Indica el estado de los subsistemas de diagnóstico que proporciona el controlador. |
| Controlador de administración de placa base (BMC) |E/S |Física |Controller |No |Indica el estado del controlador de administración de placa base (BMC), que es un procesador de servicios especializado que supervisa el dispositivo de hardware a través de sensores y se comunica con el administrador del sistema a través de una conexión independiente. |
| Ethernet |E/S |Física |Controller |No |Indica el estado de cada una de las interfaces de red, es decir, la administración y los puertos de datos proporcionados en el controlador. |
| NVRAM |E/S |Física |Controller |No |Indica el estado de la NVRAM, una memoria de acceso aleatorio no volátil respaldada por la batería que sirve para conservar la información crítica para la aplicación en caso de error de alimentación. |

## <a name="component-list-for-ebod-enclosure-of-storsimple-device"></a>Lista de componentes de la caja EBOD del dispositivo StorSimple
En la tabla siguiente se describen los componentes físicos y lógicos que contiene la caja EBOD (presente solo en la serie 8600) del dispositivo StorSimple local.

| Componente | módulo | Tipo | Location | ¿FRU? | Descripción |
| --- | --- | --- | --- | --- | --- |
| Unidad en la ranura [0-11] |Unidades de disco |Física |Compartido |Sí |Hay una línea presente para cada una de las unidades HDD en el revestimiento de EBOD. |
| Sensor de temperatura ambiente |Revestimiento |Física |Compartido |No |Mide la temperatura del chasis. |
| Sensor de temperatura de plano medio |Revestimiento |Física |Compartido |No |Mide la temperatura del plano medio. |
| Alarma audible |Revestimiento |Física |Compartido |No |Indica si el subsistema de alarma audible del chasis es funcional. |
| Revestimiento |Revestimiento |Física |Compartido |Sí |Indica la presencia de un chasis. |
| Configuración del revestimiento |Revestimiento |Física |Compartido |No |Hace referencia al OPS o panel frontal del chasis. |
| Sensores de voltaje de línea |PCM |Física |Compartido |No |En numerosos sensores de tensión de línea se muestra el estado, que indica si la tensión medida está dentro de la tolerancia. |
| Sensores de corriente de línea |PCM |Física |Compartido |No |En numerosos sensores de corriente de línea se muestra el estado, que indica si la corriente medida está dentro de la tolerancia. |
| Sensores de temperatura en el PCM |PCM |Física |Compartido |No |En numerosos sensores de temperatura como los sensores Inlet y Hotspot se muestra el estado, que indica si la temperatura medida está dentro de la tolerancia. |
| Fuente de alimentación [0-1] |PCM |Física |Compartido |Sí |Hay una línea presente para cada fuente de alimentación en las dos PCM situadas en la parte posterior del dispositivo. |
| Refrigeración [0-1] |PCM |Física |Compartido |Sí |Hay una línea presente para cada uno de los cuatro ventiladores de refrigeración que residen en los dos PCM. |
| Almacenamiento local [HDD] |N/D |Lógicos |Compartido |N/D |Muestra el estado del grupo de almacenamiento lógico que se crea desde las unidades de disco duro del dispositivo. |
| Controller [0-1] [estado] |E/S |Física |Controller |Sí |Muestra el estado de los controladores en el módulo EBOD. |
| Sensores de temperatura en el EBOD |E/S |Física |Controller |No |En numerosos sensores de temperatura de cada controlador se muestra el estado, que indica si la temperatura está dentro de la tolerancia. |
| Ampliador SAS |E/S |Física |Controller |No |Indica el estado del ampliador SAS, que se usa para conectar el almacenamiento integrado al controlador. |
| Conector SAS [0-2] |E/S |Física |Controller |No |Indica el estado de cada conector SAS, que se usa para conectar el almacenamiento integrado al ampliador SAS. |
| Interconexión de plano medio de SBB |E/S |Física |Controller |No |Indica el estado del conector de plano medio, que se usa para conectar cada controlador al plano medio. |
| Potencia de la electrónica del revestimiento |E/S |Física |Controller |No |Indica el estado del sistema de alimentación que usa el revestimiento. |
| Diagnóstico de la electrónica del revestimiento |E/S |Física |Controller |No |Indica el estado de los subsistemas de diagnóstico que proporciona el controlador. |
| Conexión al controlador del dispositivo |E/S |Física |Controller |No |Indica el estado de la conexión entre el módulo de E/S de EBOD y el controlador del dispositivo. |

## <a name="next-steps"></a>Pasos siguientes
* Para utilizar el servicio StorSimple Device Manager para administrar el dispositivo, vaya a [Uso del servicio StorSimple Device Manager para administrar el dispositivo StorSimple](storsimple-8000-manager-service-administration.md).
* Si necesita solucionar problemas de un componente del dispositivo que tiene un estado degradado o con error, vea [Indicadores de supervisión de StorSimple](storsimple-monitoring-indicators.md).
* Para reemplazar un componente de hardware con errores, vea [Reemplazo de los componentes de hardware de StorSimple](storsimple-hardware-component-replacement.md).
* Si sigue teniendo problemas con el dispositivo, [póngase en contacto con el servicio de soporte técnico de Microsoft](storsimple-8000-contact-microsoft-support.md).

