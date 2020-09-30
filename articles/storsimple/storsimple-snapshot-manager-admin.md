---
title: Administración de StorSimple Snapshot Manager | Microsoft Docs
description: Proporciona infomación general y vínculos para obtener más información acerca de las tareas de administración y flujos de trabajo de la solución Administrador de instantáneas StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 1cdbb61d-bd16-4be4-ade2-ceab11508acb
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2016
ms.author: alkohli
ms.openlocfilehash: 16d782c6a25d8d2d8e1323b7cb683f41d8c99649
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90054983"
---
# <a name="use-storsimple-snapshot-manager-to-administer-your-storsimple-solution"></a>Uso del Administrador de instantáneas StorSimple para administrar la solución de StorSimple

## <a name="overview"></a>Información general
Administrador de instantáneas StorSimple es un complemento de Microsoft Management Console (MMC) que simplifica la protección de datos y la administración de las copias de seguridad en un entorno de Microsoft Azure StorSimple. Con Administrador de instantáneas StorSimple, puede administrar los datos de Microsoft Azure StorSimple en el centro de datos y en la nube como una solución de almacenamiento integrada y, por consiguiente, simplificar los procesos de copia de seguridad y reducir los costos.

La consola de administración central de Administrador de instantáneas StorSimple le permite crear copias de seguridad de un momento dado y coherentes de datos locales y en la nube. Por ejemplo, la consola se puede usar para:

* Configurar, crear copias de seguridad y eliminar volúmenes.
* Configurar grupos de volúmenes para asegurarse de que los datos con copia de seguridad son coherentes con la aplicación.
* Administrar directivas de copia de seguridad, con el fin de que las copias de seguridad de los datos se realicen según una programación predeterminada.
* Crear copias independientes de los datos que se pueden almacenar en la nube y se pueden usar para la recuperación ante desastres.

Este artículo proporciona vínculos a tutoriales que describen Administrador de instantáneas StorSimple y cómo usarlo para completar las tareas de administración del sistema y flujos de trabajo.

* Para obtener más información acerca de la arquitectura y los componentes de Administrador de instantáneas StorSimple, consulte [¿Qué es el Administrador de instantáneas StorSimple?](storsimple-what-is-snapshot-manager.md) 
* Para descargar Administrador de instantáneas StorSimple, vaya a [la página de descarga de Administrador de instantáneas StorSimple](https://www.microsoft.com/download/details.aspx?id=44220).
* Para los procedimientos de implementación de Administrador de instantáneas StorSimple, vaya a [Implementación de Administrador de instantáneas StorSimple](storsimple-snapshot-manager-deployment.md).

> [!NOTE]
> No puede utilizar StorSimple Snapshot Manager para administrar las matrices virtuales de Microsoft Azure StorSimple (también conocidas como dispositivos virtuales locales de StorSimple).


## <a name="storsimple-snapshot-manager-tasks-and-workflows"></a>Tareas y flujos de trabajo de Administrador de instantáneas StorSimple
Administrador de instantáneas StorSimple se puede usar para supervisar y administrar trabajos de copia de seguridad próximos, programados y completados. Además, Administrador de instantáneas StorSimple proporciona un catálogo de hasta 64 copias de seguridad completadas. El catálogo se puede usar para buscar y restaurar volúmenes o archivos individuales. 

| SI DESEA HACER ESTO... | USE ESTE TUTORIAL... |
|:--- |:--- |
| Más información sobre Administrador de instantáneas StorSimple |[¿Qué es StorSimple Snapshot Manager?](storsimple-what-is-snapshot-manager.md) |
| Instalar Administrador de instantáneas StorSimple<br>Reinstalar StorSimple Snapshot Manager<br>Quitar Administrador de instantáneas StorSimple |[Implementación de Administrador de instantáneas StorSimple](storsimple-snapshot-manager-deployment.md) |
| Usar los menús y las características de Administrador de instantáneas StorSimple:<ul><li>Barra de menús</li><li>Barra de herramientas</li><li>Panel de Ámbito</li><li>Panel de Resultados</li><li>Panel de Acciones</li><li>Métodos abreviados y navegación mediante el teclado</li></ul> |[Interfaz de usuario de Administrador de instantáneas StorSimple](storsimple-use-snapshot-manager.md) |
| Usar las características comunes de MMC incluidas en Administrador de instantáneas StorSimple:<ul><li>Ver</li><li>Nueva ventana desde aquí</li><li>Actualizar</li><li>Exportar lista</li><li>Ayuda</li></ul> |[Uso de las acciones del menú de MMC en el Administrador de instantáneas StorSimple](storsimple-snapshot-manager-mmc-menu.md) |
| Incorporación o remplazo de un dispositivo<br>Conexión de un dispositivo<br>Comprobación de los grupos de volúmenes importados<br>Actualización de los dispositivos conectados<br>Autenticar un dispositivo<br>Ver los detalles de dispositivo<br>Eliminación de una configuración de dispositivo<br>Cambiar una contraseña de dispositivo<br>Reemplazo de un dispositivo con errores<br> |[Uso de Administrador de instantáneas StorSimple para conectar y administrar dispositivos StorSimple](storsimple-snapshot-manager-manage-devices.md) |
| Montaje de volúmenes<br>Ver información acerca de volúmenes<br>Eliminar un volumen<br>Volver a examinar volúmenes<br>Configuración y copia de seguridad de un volumen básico<br>Configurar y realizar copia de seguridad de un volumen reflejado dinámico |[Uso de Administrador de instantáneas para ver y administrar volúmenes StorSimple](storsimple-snapshot-manager-manage-volumes.md) |
| Visualización de grupos de volúmenes<br>Crear un grupo de volúmenes<br>Realizar una copia de seguridad de un grupo de volúmenes<br>Editar un grupo de volúmenes<br>Eliminar un grupo de volúmenes |[Uso de Administrador de instantáneas StorSimple para crear y administrar grupos de volúmenes](storsimple-snapshot-manager-manage-volume-groups.md) |
| Crear una directiva de copia de seguridad <br>Edición de una directiva de copia de seguridad<br>Eliminación de una directiva de copia de seguridad |[Use Administrador de instantáneas StorSimple para crear y administrar directivas de copia de seguridad](storsimple-snapshot-manager-manage-backup-policies.md) |
| Ver y administrar los trabajos de copia de seguridad programados<br>Ver y administrar los trabajos de copia de seguridad recientes<br>Ver y administrar trabajos de copia de seguridad en ejecución en ese momento |[Uso de Administrador de instantáneas StorSimple para ver y administrar trabajos de copia de seguridad](storsimple-snapshot-manager-manage-backup-jobs.md) |
| Restauración de un volumen<br>Clonación de un volumen o un grupo de volúmenes<br>Eliminación de una copia de seguridad<br>Recuperación de un archivo<br>Restauración de la base de datos de Administrador de instantáneas Storsimple |[Uso de Administrador de instantáneas StorSimple para administrar el catálogo de copia de seguridad](storsimple-snapshot-manager-manage-backup-catalog.md) |

## <a name="next-steps"></a>Pasos siguientes
[Descarga de Administrador de instantáneas StorSimple](https://www.microsoft.com/download/details.aspx?id=44220).

