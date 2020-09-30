---
title: ¿Qué es Administrador de instantáneas StorSimple? | Microsoft Docs
description: Describe Administrador de instantáneas StorSimple, su arquitectura y sus características.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 6094c31e-e2d9-4592-8a15-76bdcf60a754
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d7e9d73fa94acd2f31452de05ef3caee92d5cf3d
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90055510"
---
# <a name="an-introduction-to-storsimple-snapshot-manager"></a>Una introducción a StorSimple Snapshot Manager

## <a name="overview"></a>Información general
Administrador de instantáneas StorSimple es un complemento de Microsoft Management Console (MMC) que simplifica la protección de datos y la administración de las copias de seguridad en un entorno de Microsoft Azure StorSimple. Con Administrador de instantáneas StorSimple, puede administrar los datos de Microsoft Azure StorSimple en el centro de datos y en la nube como una solución de almacenamiento integrada y, por consiguiente, simplificar los procesos de copia de seguridad y reducir los costos.

Esta información general presenta el Administrador de instantáneas StorSimple, describe sus características y explica su rol en Microsoft Azure StorSimple. 

Para obtener información general de todo el sistema de Microsoft Azure StorSimple, incluido el dispositivo de StorSimple, el servicio StorSimple Manager, el Administrador de instantáneas StorSimple y el Adaptador de StorSimple para SharePoint, consulte [Serie StorSimple 8000: una solución híbrida de almacenamiento en la nube](storsimple-overview.md). 

> [!NOTE]
> * No puede utilizar StorSimple Snapshot Manager para administrar las matrices virtuales de Microsoft Azure StorSimple (también conocidas como dispositivos virtuales locales de StorSimple).
> * Si tiene previsto instalar la actualización 2 de StorSimple en el dispositivo de StorSimple, asegúrese de descargar la última versión de StorSimple Snapshot Manager e instalarlo **antes de instalar la actualización 2 de StorSimple**. La última versión de StorSimple Snapshot Manager es compatible con versiones anteriores y funciona con todas las versiones publicadas de Microsoft Azure StorSimple. Si usa la versión anterior de StorSimple Snapshot Manager, debe actualizarlo (no es necesario desinstalar la versión anterior antes de instalar la nueva versión).
> 
> 

## <a name="storsimple-snapshot-manager-purpose-and-architecture"></a>Arquitectura y propósito de Administrador de instantáneas StorSimple
Administrador de instantáneas StorSimple proporciona una consola de administración central que se puede usar para crear copias de seguridad de un momento dado y coherentes de datos locales y en la nube. Por ejemplo, la consola se puede usar para:

* Configurar, crear copias de seguridad y eliminar volúmenes.
* Configurar grupos de volúmenes para asegurarse de que los datos con copia de seguridad son coherentes con la aplicación.
* Administrar directivas de copia de seguridad, con el fin de que las copias de seguridad de los datos se realicen según una programación predeterminada.
* Cree instantáneas locales y en la nube, que se pueden almacenar en la nube y utilizarse para la recuperación ante desastres.

Snapshot Manager de StorSimple recupera la lista de aplicaciones registradas con el proveedor VSS en el host. Así, para crear copias de seguridad coherentes con la aplicación, comprueba los volúmenes utilizados por una aplicación y sugiere grupos de volúmenes para configurar. Administrador de instantáneas StorSimple usa estos grupos de volúmenes para generar copias de seguridad coherentes con la aplicación. (Existe coherencia de aplicaciones cuando todos los archivos y bases de datos relacionados están sincronizados y representan el estado real de la aplicación en un momento dado específico.) 

Las copias de seguridad de Administrador de instantáneas StorSimple adoptan la forma de instantáneas incrementales, que capturan únicamente los cambios desde la última copia de seguridad. Como consecuencia, las copias de seguridad requieren menos espacio de almacenamiento y se pueden crear y restaurar rápidamente. Administrador de instantáneas StorSimple usa el Servicio de instantáneas de volumen  de Windows (VSS) para garantizar que las instantáneas capturan datos coherentes con la aplicación. (Para obtener más información, vaya a la sección Integración con el Servicio de instantáneas de volumen  de Windows.) Con Administrador de instantáneas StorSimple, puede crear programaciones de copia de seguridad o hacer copias inmediatas, según sea necesario. Si necesita restaurar datos desde una copia de seguridad, Administrador de instantáneas StorSimple le permite seleccionarla en un catálogo de instantáneas locales o en la nube. Azure StorSimple restaura solo los datos necesarios, y en el momento en que se necesitan, lo que evita retrasos en la disponibilidad de los datos durante las operaciones de restauración.

![Arquitectura de Administrador de instantáneas StorSimple](./media/storsimple-what-is-snapshot-manager/HCS_SSM_Overview.png)

**Arquitectura de Administrador de instantáneas StorSimple** 

## <a name="support-for-multiple-volume-types"></a>Compatibilidad con varios tipos de volumen
Administrador de instantáneas StorSimple se puede usar para configurar y realizar copias de seguridad de los siguientes tipos de volúmenes: 

* **Volúmenes básicos** : un volumen básico es una sola partición en un disco básico. 
* **Volúmenes simples**: un volumen simple es un volumen dinámico que contiene el espacio en disco de un único disco dinámico. Un volumen simple se compone de una única región de un disco o varias regiones vinculadas entre sí en el mismo disco. (Los volúmenes simples solo se pueden crear en discos dinámicos.) Los volúmenes simples no son tolerantes a errores.
* **Volúmenes dinámicos** : un volumen dinámico es un volumen creado en un disco dinámico. Los discos dinámicos usan una base de datos para controlar la información acerca de los volúmenes que se encuentran en los discos dinámicos de un equipo. 
* **Volúmenes dinámicos con creación de reflejo** : los volúmenes dinámicos con creación de reflejo se basan en la arquitectura RAID 1. Con RAID 1, se escriben datos idénticos en dos o más discos, lo que genera un conjunto reflejado. De esa forma, las solicitudes de lectura las puede administrar cualquier disco que contenga los datos solicitados.
* **Volúmenes compartidos de clúster** : con los volúmenes compartidos de clúster (CSV), varios nodos de un clúster de conmutación por error pueden leer el mismo disco de forma simultánea o escribir en él. La conmutación por error de un nodo a otro puede producirse rápidamente, sin necesidad de un cambio en la propiedad de la unidad ni de montar, desmontar y quitar un volumen. 

> [!IMPORTANT]
> No mezcle CSV y no CSV en la misma instantánea. No se admite la mezcla de CSV y no CSV en una instantánea. 
> 
> 

Puede usar Administrador de instantáneas StorSimple para restaurar grupos de volúmenes completos o clonar volúmenes individuales y recuperar archivos individuales.

* [Volúmenes y grupos de volúmenes](#volumes-and-volume-groups) 
* [Tipos de copia de seguridad y directivas de copia de seguridad](#backup-types-and-backup-policies) 

Para obtener más información acerca de las funciones del Administrador de instantáneas StorSimple y cómo usarlas, consulte la [Interfaz de usuario del Administrador de instantáneas StorSimple](storsimple-use-snapshot-manager.md).

## <a name="volumes-and-volume-groups"></a>Volúmenes y grupos de volúmenes
Con Administrador de instantáneas StorSimple, es posible crear volúmenes y, a continuación, configurarlos en grupos de volúmenes. 

Administrador de instantáneas StorSimple usa grupos de volúmenes para crear copias de seguridad coherentes con la aplicación. Existe coherencia de aplicaciones cuando todos los archivos y bases de datos relacionados están sincronizados y representan el estado real de una aplicación en un momento dado específico. Los grupos de volúmenes (también conocidos como *grupos de consistencia*) constituyen la base de los trabajos de copia de seguridad y de restauración.

Los grupos de volúmenes no son lo mismo que los contenedores de volúmenes. Un contenedor de volúmenes contiene uno o varios volúmenes que comparten una cuenta de almacenamiento de nube y otros atributos, como el consumo de ancho de banda y el cifrado. Un contenedor de volúmenes individual puede contener hasta 256 volúmenes de StorSimple con aprovisionamiento fino. Para obtener más información acerca de los contenedores de volúmenes, vaya a [Administración de contenedores de volúmenes](storsimple-manage-volume-containers.md). Los grupos de volúmenes son colecciones de volúmenes que se pueden configurar para facilitar las operaciones de copia de seguridad. Si selecciona dos volúmenes que pertenecen a distintos contenedores de volúmenes, colóquelos en un único grupo de volúmenes y, a continuación, cree una directiva de copia de seguridad para dicho grupo de volúmenes, se realizará una copia de seguridad de cada volumen en el contenedor de volumen apropiado mediante la cuenta de almacenamiento adecuada.

> [!NOTE]
> Todos los volúmenes de un grupo de volúmenes deben provenir de un solo proveedor de servicios en la nube.
> 
> 

## <a name="integration-with-windows-volume-shadow-copy-service"></a>Integración con el Servicio de instantáneas de volumen de Windows
Administrador de instantáneas StorSimple usa el Servicio de instantáneas de volumen de Windows (VSS) para capturar datos coherentes con la aplicación. VSS facilita la consistencia de aplicaciones, ya que se comunica con las aplicaciones para VSS para coordinar la creación de instantáneas incrementales. VSS garantiza que las aplicaciones están temporalmente o completamente inactivas cuando se realizan las instantáneas. 

La implementación de Administrador de instantáneas StorSimple de VSS funciona con SQL Server y volúmenes NTFS genéricos. El proceso es el siguiente: 

1. Un solicitante, que normalmente es una solución de protección y administración de datos (como Administrador de instantáneas StorSimple) o una aplicación de copia de seguridad, invoca a VSS y le pide que recopile información del software de escritura de la aplicación de destino.
2. VSS contacta con el componente escritor para recuperar una descripción de los datos. El escrituro devuelve la descripción de los datos de los que se va a realizar la copia de seguridad. 
3. VSS indica al escritor que prepare la aplicación para la copia de seguridad. El escritor prepara los datos para la copia de seguridad completando, para lo que completa las transacciones abiertas, actualiza los registros de transacciones etc. y, a continuación, notifica a VSS.
4. VSS indica al escritor que detenga temporalmente los almacenes de datos de la aplicación y se asegure de que no se escriben datos en el volumen mientras se crea la instantánea. Este paso garantiza la coherencia de datos y no tarda más de 60 segundos en completarse.
5. VSS envía al proveedor la orden de que cree la instantánea. Los proveedores, que pueden basarse en software o en hardware, administran los volúmenes que se ejecutan actualmente y crean instantáneas de ellos a petición. El proveedor crea la instantánea y notifica a VSS cuando se complete.
6. VSS contacta con el escritor para notificar a la aplicación que se puede reanudar la E/S y también para confirmar que la E/S se pausó correctamente durante la creación de la instantánea. 
7. Si la copia se realizó correctamente, VSS devuelve la ubicación de la copia al solicitante. 
8. Si se escribieron datos mientras se creaba la instantánea, la copia de seguridad será incoherente. VSS eliminará la instantánea y notificará al solicitante. El solicitante puede repetir el proceso de copia de seguridad automáticamente, o bien notificar al administrador que lo reintente más tarde.

Vea la ilustración siguiente.

![Proceso VSS](./media/storsimple-what-is-snapshot-manager/HCS_SSM_VSS_process.png)

**Proceso del Servicio de instantáneas de volumen de Windows** 

## <a name="backup-types-and-backup-policies"></a>Tipos de copia de seguridad y directivas de copia de seguridad
Con Administrador de instantáneas StorSimple puede realizar copias de seguridad de los datos y almacenarlas localmente y en la nube. Puede usar Administrador de instantáneas StorSimple para realizar copias de seguridad de los datos inmediatamente o puede usar una directiva de copia de seguridad para crear una programación para realizar copias de seguridad de forma automática. Las directivas de copia de seguridad también permiten especificar el número de instantáneas que se conservan. 

### <a name="backup-types"></a>Tipos de copia de seguridad
Puede usar Administrador de instantáneas StorSimple para crear los siguientes tipos de copias de seguridad:

* **Instantáneas locales** : las instantáneas locales son copias de los datos de un volumen que se realizan en un momento dado y que se almacenan en el dispositivo StorSimple. Normalmente, este tipo de copia de seguridad se puede crear y restaurar rápidamente. Puede usar una instantánea local como lo haría con una copia de seguridad local.
* **Instantáneas en la nube** : las instantáneas en la nube son copias de los datos de un volumen que se realizan en un momento dado y que se almacenan en la nube. Una instantánea de nube es equivalente a una instantánea replicada en un sistema de almacenamiento externo. Las instantáneas en la nube son especialmente útiles en escenarios de recuperación ante desastres.

### <a name="on-demand-and-scheduled-backups"></a>Copias de seguridad a petición y programadas
Con Administrador de instantáneas StorSimple, puede iniciar una copia de seguridad única que se creará de inmediato, o bien puede usar una directiva de copia de seguridad para programar operaciones de copia de seguridad periódicas.

Una directiva de copia de seguridad es un conjunto de reglas automatizadas que puede usar para programar copias de seguridad periódicas. Una directiva de copia de seguridad permite definir la frecuencia y los parámetros necesarios para tomar instantáneas de un grupo de volúmenes específico. Las directivas se pueden usar para especificar las fechas y horas de inicio y finalización, así como las frecuencias y los requisitos de retención de instantáneas tanto locales como en la nube. Las directivas se aplican inmediatamente después de que se definen. 

Puede usar Administrador de instantáneas StorSimple para configurar o volver a configurar las directivas de copia de seguridad siempre que sea necesario. 

Configure la siguiente información en cada directiva de copia de seguridad que cree:

* **Nombre** : es el nombre único de la directiva de copia de seguridad seleccionada.
* **Tipo** : es el tipo de directiva de copia de seguridad; bien instantánea local, o bien instantánea en la nube.
* **Grupo de volúmenes** : es el grupo de volúmenes al que se asigna la directiva de copia de seguridad seleccionada.
* **Retención** : es el número de copias de seguridad que se conservan. Si activa el cuadro **Todas** , todas las copias de seguridad se conservan hasta que se alcance el número máximo de copias de seguridad por volumen; en ese momento, se producirá un error en la directiva y se generará un mensaje de error. También puede especificar el número de copias de seguridad que se conservarán (entre 1 y 64).
* **Fecha** : es la fecha en la que se creó la directiva de copia de seguridad.

Para obtener información acerca de la configuración de directivas de copia de seguridad, vaya a [Uso del Administrador de instantáneas StorSimple para crear y administrar directivas de copia de seguridad](storsimple-snapshot-manager-manage-backup-policies.md).

### <a name="backup-job-monitoring-and-management"></a>Supervisión y administración de trabajos de copia de seguridad
Administrador de instantáneas de StorSimple se puede usar para supervisar y administrar trabajos de copia de seguridad próximos, programados y completados. Además, Administrador de instantáneas StorSimple proporciona un catálogo de hasta 64 copias de seguridad completadas. El catálogo se puede usar para buscar y restaurar volúmenes o archivos individuales. 

Para obtener información acerca de la supervisión de trabajos de copia de seguridad, vaya a [Uso de Administrador de instantáneas StorSimple para ver y administrar trabajos de copia de seguridad](storsimple-snapshot-manager-manage-backup-jobs.md).

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [uso de Snapshot Manager de StorSimple para administrar la solución de StorSimple](storsimple-snapshot-manager-admin.md).
* Descargue el [Administrador de instantáneas StorSimple](https://www.microsoft.com/download/details.aspx?id=44220).

