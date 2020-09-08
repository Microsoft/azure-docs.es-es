---
title: StorSimple serie 8000 como destino de copia de seguridad con NetBackup | Microsoft Docs
description: Obtenga información sobre la configuración del destino de copia de seguridad de StorSimple con Veritas NetBackup y los procedimientos recomendados para la integración de ambas soluciones.
services: storsimple
documentationcenter: ''
author: harshakirank
manager: matd
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/15/2017
ms.author: matd
ms.openlocfilehash: b186fadcc99c6cc538b61eaa94d5d84d649c233f
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184013"
---
# <a name="storsimple-as-a-backup-target-with-netbackup"></a>StorSimple como destino de copia de seguridad con NetBackup

## <a name="overview"></a>Información general

Azure StorSimple es una solución de almacenamiento en nube híbrida de Microsoft. StorSimple aborda las complejidades del crecimiento exponencial de datos mediante una cuenta Azure Storage como una extensión de la solución local y la organización en capas automática de datos en almacenamiento local y almacenamiento en nube.

En este artículo se describen la integración de StorSimple con NetBackup y los procedimientos recomendados para la integración de ambas soluciones. También se ofrecen recomendaciones sobre cómo configurar Veritas NetBackup para que se integre perfectamente con StorSimple. Nos remitimos a los procedimientos recomendados de Veritas, los arquitectos de copias de seguridad y los administradores en cuanto a la mejor manera de configurar Veritas NetBackup con el objetivo de cumplir los requisitos de copia de seguridad individual y los Acuerdos de Nivel de Servicio (SLA).

Aunque se muestran tanto los pasos de configuración como los conceptos clave, este artículo no es una guía detallada de configuración o instalación. Se supone que la infraestructura y los componentes básicos funcionan perfectamente y están listos para admitir los conceptos que se describen.

### <a name="who-should-read-this"></a>¿A quiénes está dirigido este documento?

La información de este artículo será de especial utilidad para administradores de copia de seguridad, administradores de almacenamiento y arquitectos de almacenamiento con conocimientos de almacenamiento, Windows Server 2012 R2, Ethernet, servicios en la nube y Veritas NetBackup.

### <a name="supported-versions"></a>Versiones compatibles

-   NetBackup 7.7.x y versiones posteriores
-   [StorSimple Update 3 y versiones posteriores](storsimple-overview.md#storsimple-workload-summary)


## <a name="why-storsimple-as-a-backup-target"></a>¿Por qué StorSimple como destino de copia de seguridad?

StorSimple es una buena elección para un destino de copia de seguridad porque:

-   Proporciona almacenamiento local estándar que las aplicaciones de copia de seguridad pueden usar con el fin de proporcionar un destino de copias de seguridad rápidas sin realizar ningún cambio. También puede usar StorSimple para la restauración rápida de copias de seguridad recientes.
-   Sus niveles en la nube se integran perfectamente con una cuenta de almacenamiento en nube de Azure para usar Azure Storage de forma rentable.
-   Proporciona automáticamente almacenamiento externo para la recuperación ante desastres.

## <a name="key-concepts"></a>Conceptos clave

Al igual que con cualquier solución de almacenamiento, para que sea un éxito son fundamentales una evaluación minuciosa del rendimiento del almacenamiento de la solución, los SLA, la tasa de cambio y las necesidades de aumento de capacidad. La idea principal es que mediante la introducción de un nivel de nube, los tiempos de acceso y el rendimiento en la nube juegan un papel fundamental en la capacidad de StorSimple para hacer su trabajo.

StorSimple está diseñado para proporcionar almacenamiento a las aplicaciones que funcionan en un conjunto de datos activo bien definido (datos activos). En este modelo, el conjunto de datos activo se almacena en las capas locales y el conjunto de datos no activos o archivados restantes está en capas en la nube. Este modelo se representa en la siguiente ilustración. La línea verde casi plana representa los datos almacenados en los niveles locales del dispositivo de StorSimple. La línea roja representa la cantidad total de datos almacenados en la solución de StorSimple a través de todos los niveles. El espacio entre la línea verde plana y la curva exponencial roja representa la cantidad total de datos almacenados en la nube.

**Niveles de StorSimple**
![Diagrama de niveles de Storsimple](./media/storsimple-configure-backup-target-using-netbackup/image1.jpg)

Con esta arquitectura en mente, StorSimple es ideal para operar como destino de copia de seguridad. Puede usar StorSimple para:
-   Realizar las restauraciones más frecuentes desde el conjunto de datos activo local.
-   Usar la nube para la recuperación ante desastres fuera del sitio y para datos antiguos, donde las restauraciones son menos frecuentes.

## <a name="storsimple-benefits"></a>Ventajas de StorSimple

StorSimple ofrece una solución local que se integra perfectamente con Microsoft Azure, para lo que saca provecho del acceso ininterrumpido al almacenamiento local y en la nube.

StorSimple usa niveles automáticos entre el dispositivo local, que tiene almacenamiento SCSI conectado en serie (SAS) y en dispositivo en estado sólido (SSD), y Azure Storage. Los niveles automáticos conservan los datos de acceso frecuente en local, en los niveles SAS y SSD. Mueve los datos de acceso poco frecuente a Azure Storage.

StorSimple ofrece las siguientes ventajas:

-   Algoritmos de desduplicación y compresión únicos que usan la nube para lograr unos niveles de desduplicación sin precedentes
-   Alta disponibilidad
-   Replicación geográfica mediante el uso de la replicación geográfica de Azure
-   Integración con Azure
-   Cifrado de datos en la nube
-   Mejor recuperación ante desastres y cumplimiento normativo

Aunque StorSimple presenta dos escenarios de implementación principales (destino de copia de seguridad principal y secundario), es fundamentalmente un dispositivo de almacenamiento de bloques sin formato. StorSimple realiza toda la compresión y desduplicación. Envía y recupera datos perfectamente entre la nube y el sistema de archivos y de la aplicación.

Para obtener más información sobre StorSimple, consulte [Serie StorSimple 8000: una solución de almacenamiento en la nube híbrida](storsimple-overview.md). También puede consultar las [especificaciones técnicas para Serie StorSimple 8000](storsimple-technical-specifications-and-compliance.md).

> [!IMPORTANT]
> El uso del dispositivo StorSimple como destino de copia de seguridad es compatible solo con StorSimple 8000 Update 3 y versiones posteriores.

## <a name="architecture-overview"></a>Información general sobre la arquitectura

Las tablas siguientes muestran la guía inicial de la relación entre el modelo del dispositivo y la arquitectura.

**Capacidades de StorSimple para el almacenamiento local y en la nube**

| Capacidad de almacenamiento       | 8100          | 8600            |
|------------------------|---------------|-----------------|
| Capacidad de almacenamiento local | &lt; 10 TiB\*  | &lt; 20 TiB\*  |
| Capacidad de almacenamiento en la nube | &gt; 200 TiB\* | &gt; 500 TiB\* |

\* En el tamaño de almacenamiento, se asume que no hay desduplicación ni compresión.

**Capacidades de StorSimple para copias de seguridad principales y secundarias**

| Escenario de copia de seguridad  | Capacidad de almacenamiento local  | Capacidad de almacenamiento en la nube  |
|---|---|---|
| Copia de seguridad principal  | Copias de seguridad recientes almacenadas localmente para una recuperación rápida que cumpla el objetivo de punto de recuperación (RPO) | El historial de copias de seguridad (RPO) se ajusta a la capacidad de la nube |
| Copia de seguridad secundaria | La copia secundaria de los datos de las copias de seguridad se puede almacenar en la capacidad de la nube  | N/D  |

## <a name="storsimple-as-a-primary-backup-target"></a>StorSimple como destino de copia de seguridad principal

En este escenario, los volúmenes de StorSimple se presentan a la aplicación de copia de seguridad como el único repositorio para copias de seguridad. La siguiente ilustración muestra la arquitectura de una solución en la que todas las copias de seguridad usan los volúmenes en capas de StorSimple para las copias de seguridad y las restauraciones.

![Diagrama lógico de StorSimple como destino de copia de seguridad principal](./media/storsimple-configure-backup-target-using-netbackup/primarybackuptargetlogicaldiagram.png)

### <a name="primary-target-backup-logical-steps"></a>Pasos lógicos de copias de seguridad en el destino principal

1.  El servidor de copia de seguridad se pone en contacto con el agente de copia de seguridad de destino y este transmite los datos al primero.
2.  El servidor de copia de seguridad escribe los datos en los volúmenes en capas de StorSimple.
3.  El servidor de copia de seguridad actualiza la base de datos del catálogo y después finaliza el trabajo de copia de seguridad.
4.  Un script de instantánea desencadena el administrador de instantáneas de StorSimple (iniciar o eliminar).
5.  El servidor de copia de seguridad elimina las copias de seguridad que han expirado en función de lo que establezca una directiva de retención.

### <a name="primary-target-restore-logical-steps"></a>Pasos lógicos de restauración del destino principal

1.  El servidor de copia de seguridad empieza a restaurar los datos apropiados del repositorio de almacenamiento.
2.  El agente de copia de seguridad recibe los datos del servidor de copia de seguridad.
3.  El servidor de copia de seguridad completa el trabajo de restauración.

## <a name="storsimple-as-a-secondary-backup-target"></a>StorSimple como destino de copia de seguridad secundario

En este escenario, los volúmenes de StorSimple se utilizan principalmente para la retención o el archivado a largo plazo.

En la siguiente ilustración se muestra una arquitectura en la que las copias de seguridad y restauraciones iniciales tienen como destino un volumen de alto rendimiento. Estas copias de seguridad se realizan y se archivan en un volumen en capas de StorSimple con una programación establecida.

Es importante asignar al volumen de alto rendimiento un tamaño adecuado para que pueda controlar la capacidad de la directiva de retención y los requisitos de rendimiento.

![Diagrama lógico de StorSimple como destino de copia de seguridad secundario](./media/storsimple-configure-backup-target-using-netbackup/secondarybackuptargetlogicaldiagram.png)

### <a name="secondary-target-backup-logical-steps"></a>Pasos lógicos de copias de seguridad en el destino secundario

1.  El servidor de copia de seguridad se pone en contacto con el agente de copia de seguridad de destino y este transmite los datos al primero.
2.  El servidor de copia de seguridad escribe datos en el almacenamiento de alto rendimiento.
3.  El servidor de copia de seguridad actualiza la base de datos del catálogo y después finaliza el trabajo de copia de seguridad.
4.  El servidor de copia de seguridad copia las copias de seguridad en StorSimple en función de lo que establezca una directiva de retención.
5.  Un script de instantánea desencadena el administrador de instantáneas de StorSimple (iniciar o eliminar).
6.  El servidor de copia de seguridad elimina las copias de seguridad que han expirado en función de lo que establezca una directiva de retención.

### <a name="secondary-target-restore-logical-steps"></a>Pasos lógicos de restauración del destino secundario

1.  El servidor de copia de seguridad empieza a restaurar los datos apropiados del repositorio de almacenamiento.
2.  El agente de copia de seguridad recibe los datos del servidor de copia de seguridad.
3.  El servidor de copia de seguridad completa el trabajo de restauración.

## <a name="deploy-the-solution"></a>Implementación de la solución

La implementación de esta solución requiere tres pasos:
1. Preparación de la infraestructura de red
2. Implementación del dispositivo StorSimple como destino de copia de seguridad
3. Implementación de Veritas NetBackup

En las siguientes secciones se detallan cada uno de estos pasos.

### <a name="set-up-the-network"></a>Configuración de la red

Puesto que StorSimple es una solución integrada con la nube de Azure, requiere una conexión activa con esta. Esta conexión se utiliza para operaciones tales como instantáneas en la nube, administración de datos y transferencia de metadatos, así como para almacenar en capas datos antiguos con menos acceso en el almacenamiento en la nube de Azure.

Para que la solución funcione de manera óptima, se recomienda seguir estos procedimientos de red recomendados:

-   El vínculo que conecta los niveles de StorSimple con Azure debe cumplir sus requisitos de ancho de banda. Para lograrlo, debe aplicar el nivel adecuado de Calidad de servicio (QoS) a los conmutadores de su infraestructura para que cumplan sus Acuerdos de Nivel de Servicio de RPO y Objetivo de tiempo de recuperación (RTO).

-   Las latencias de acceso máximas de Azure Blob Storage deben rondar los 80 ms.

### <a name="deploy-storsimple"></a>Implementación de StorSimple

Para ver instrucciones detalladas para la implementación de StorSimple, consulte [Implementar el dispositivo StorSimple local](storsimple-deployment-walkthrough-u2.md).

### <a name="deploy-netbackup"></a>Implementación de NetBackup

En la [documentación de NetBackup 7.7.x](http://www.veritas.com/docs/000094423) puede consultar una guía de implementación paso a paso de NetBackup 7.7.x.

## <a name="set-up-the-solution"></a>Configuración de la solución

En esta sección se muestran algunos ejemplos de configuración. Las siguientes recomendaciones y ejemplos ilustran la implementación más básica y fundamental. Es posible que esta implementación no se aplique directamente a sus requisitos de copia de seguridad específicos.

### <a name="set-up-storsimple"></a>Configuración de StorSimple

| Tareas de implementación de StorSimple  | Comentarios adicionales |
|---|---|
| Implementar un dispositivo de StorSimple local. | Versiones compatibles: Update 3 y versiones posteriores. |
| Active el destino de copia de seguridad. | Utilice estos comandos para activar o desactivar el modo de destino de copia de seguridad y para obtener el estado. Para obtener más información, vaya a [Conectarse de forma remota al dispositivo StorSimple](storsimple-remote-connect.md).</br> Para activar el modo de copia de seguridad: `Set-HCSBackupApplianceMode -enable`. </br> Para desactivar el modo de copia de seguridad: `Set-HCSBackupApplianceMode -disable`. </br> Para obtener el estado actual de la configuración del modo de copia de seguridad: `Get-HCSBackupApplianceMode`. |
| Cree un contenedor de volúmenes común para el volumen que almacena los datos de copia de seguridad. Todos los datos de un contenedor de volumen se desduplican. | Los contenedores de volúmenes de StorSimple definen dominios de desduplicación.  |
| Cree volúmenes de StorSimple. | Cree volúmenes cuyos tamaños se ajusten lo máximo posible al uso previsto, ya que el tamaño del volumen afecta a la duración de la instantánea de la nube. Para obtener información sobre cómo cambiar el tamaño de un volumen, consulte las [directivas de retención](#retention-policies).</br> </br> Use los volúmenes en capas de StorSimple y seleccione la casilla **Usar este volumen para los datos de archivo a los que accede con menos frecuencia**. </br> No se admite que solo se usen volúmenes anclados localmente. |
| Cree una directiva de copia de seguridad de StorSimple única para todos los volúmenes de destino de copia de seguridad. | Una directiva de copia de seguridad de StorSimple define el grupo de coherencia del volumen. |
| Deshabilite la programación a medida que las instantáneas expiran. | Las instantáneas se desencadenan como operación posterior al procesamiento. |

### <a name="set-up-the-host-backup-server-storage"></a>Configuración del almacenamiento del servidor de copia de seguridad de host

Configure el almacenamiento del servidor de copia de seguridad de host siguiendo estas instrucciones:  

- No use volúmenes distribuidos (creados por el administrador de discos de Windows), ya que no se admiten.
- Dé formato a los volúmenes mediante NTFS con un tamaño de asignación de 64 kB.
- Asigne los volúmenes de StorSimple directamente al servidor NetBackup.
    - Use iSCSI para servidores físicos.
    - Use discos de acceso directo para servidores virtuales.


## <a name="best-practices-for-storsimple-and-netbackup"></a>Procedimientos recomendados para StorSimple y NetBackup

Configure su solución aplicando las instrucciones de las siguientes secciones.

### <a name="operating-system-best-practices"></a>Procedimientos recomendados para un sistema operativo

- Deshabilite el cifrado y la desduplicación de Windows Server para el sistema de archivos NTFS.
- Deshabilite la desfragmentación de Windows Server en los volúmenes de StorSimple.
- Deshabilite la indexación de Windows Server en los volúmenes de StorSimple.
- Realice una detección de virus en el host de origen (no en los volúmenes de StorSimple).
- Desactive el [mantenimiento de Windows Server](https://msdn.microsoft.com/library/windows/desktop/hh848037.aspx) predeterminado en el Administrador de tareas. Para ello, siga uno de estos pasos:
  - Desactive el configurador de mantenimiento en el Programador de tareas de Windows.
  - Descargue [PsExec](https://technet.microsoft.com/sysinternals/bb897553.aspx) de Windows Sysinternals. Después de descargar PsExec, ejecute Windows PowerShell como administrador y escriba:
    ```powershell
    psexec \\%computername% -s schtasks /change /tn “MicrosoftWindowsTaskSchedulerMaintenance Configurator" /disable
    ```

### <a name="storsimple-best-practices"></a>Procedimientos recomendados para StorSimple

-   Asegúrese de que el dispositivo de StorSimple se ha actualizado a [Update 3 o cualquier versión posterior](storsimple-install-update-3.md).
-   Aísle tráfico de iSCSI y de la nube. Use conexiones iSCSI dedicadas para el tráfico entre StorSimple y el servidor de copia de seguridad.
-   Asegúrese de que su dispositivo de StorSimple sea un destino de copia de seguridad dedicado. No se admiten cargas de trabajo mixtas porque afectan a su RTO y RPO.

### <a name="netbackup-best-practices"></a>Procedimientos recomendados para NetBackup

-   La base de datos de NetBackup debe ser local para el servidor y no residir en un volumen de StorSimple.
-   Para recuperación ante desastres, realice una copia de la base de datos de NetBackup en un volumen de StorSimple.
-   En esta solución se admiten copias de seguridad completas e incrementales de NetBackup (que también se conocen como copias de seguridad incrementales diferenciales en NetBackup). Se recomienda no usar copias de seguridad incrementales sintéticas y acumulativas.
-   Los archivos de datos de copia de seguridad solo deben contener los datos de un trabajo concreto. Por ejemplo, no se permiten anexos de medios entre distintos trabajos.

En la documentación de NetBackup que se encuentra en [www.veritas.com](https://www.veritas.com) puede consultar la configuración más reciente de NetBackup y los procedimientos recomendados para implementar estos requisitos.


## <a name="retention-policies"></a>Directivas de retención

Una de las directivas de retención de copias de seguridad más habituales es Grandfather, Father, and Son (GFS). En una directiva GFS, se realiza una copia de seguridad incremental diaria y se realizan copias de seguridad completas semanales y mensuales. Esta directiva da lugar a seis volúmenes de StorSimple en capas: un volumen contiene las copias de seguridad completas semanales, mensuales y anuales; los otros cinco volúmenes almacenan copias de seguridad incrementales diarias.

En el siguiente ejemplo se usa una rotación de GFS. En dicho ejemplo se dan por supuestos los siguientes hechos:

-   Se usan datos no desduplicados o comprimidos.
-   Cada copia de seguridad completa ocupa 1 TiB.
-   Las copias de seguridad incrementales diarias ocupan 500 GiB.
-   Se conservan cuatro copias de seguridad semanales durante un mes.
-   Se conservan doce copias de seguridad mensuales durante un año.
-   Se conserva una copia de seguridad anual durante diez años.

De acuerdo con los supuestos anteriores, cree un volumen en capas de StorSimple de 26 TiB para las copias de seguridad completas mensuales y anuales. Cree un volumen en capas de StorSimple de 5 TiB para cada una de las copias de seguridad incrementales diarias.

| Retención de tipo de copia de seguridad | Tamaño (TiB) | Multiplicador de GFS\* | Capacidad total (TiB)  |
|---|---|---|---|
| Completa semanal | 1 | 4  | 4 |
| Incremental diaria | 0.5 | 20 (los ciclos equivalen al número de semanas al mes) | 12 (2 para cuota adicional) |
| Completa mensual | 1 | 12 | 12 |
| Completa anual | 1  | 10 | 10 |
| Requisito de GFS |   | 38 |   |
| Cuota adicional  | 4  |   | Requisito de GFS, un total de 42  |

\* El multiplicador de GFS es el número de copias que es preciso proteger y retener para cumplir los requisitos de las directivas de copia de seguridad.

## <a name="set-up-netbackup-storage"></a>Configuración del almacenamiento de NetBackup

### <a name="to-set-up-netbackup-storage"></a>Para configurar el almacenamiento de NetBackup

1.  En la consola de administración NetBackup, seleccione **Media and Device Management** > **Devices** > **Disk Pools** (Administración de dispositivos y medios > Dispositivos > Grupos de discos). En el Asistente para la configuración del grupo de discos, seleccione el tipo de servidor de almacenamiento **AdvancedDisk** (Disco avanzado) y después **Next** (Siguiente).

    ![Consola de administración de NetBackup, Asistente para configuración de grupo de discos](./media/storsimple-configure-backup-target-using-netbackup/nbimage1.png)

2.  Busque su servidor y seleccione **Next** (Siguiente).

    ![Consola de administración de NetBackup, seleccione el servidor](./media/storsimple-configure-backup-target-using-netbackup/nbimage2.png)

3.  Seleccione su volumen de StorSimple.

    ![Consola de administración de NetBackup, seleccione el disco de volumen de StorSimple](./media/storsimple-configure-backup-target-using-netbackup/nbimage3.png)

4.  Escriba un nombre para el destino de copia de seguridad y después seleccione **Next** > **Next** (Siguiente > Siguiente) para finalizar el asistente.

5.  Revise la configuración y, a continuación, seleccione **Finish** (Finalizar).

6.  Al final de cada asignación de volumen, cambie la configuración del dispositivo de almacenamiento para que coincida con las recomendaciones de [Procedimientos recomendados para StorSimple y NetBackup](#best-practices-for-storsimple-and-netbackup).

7. Repita los pasos del 1 al 6 hasta que haya terminado de asignar los volúmenes de StorSimple.

    ![Consola de administración de NetBackup, configuración de discos](./media/storsimple-configure-backup-target-using-netbackup/nbimage5.png)

## <a name="set-up-storsimple-as-a-primary-backup-target"></a>Configuración de StorSimple como destino de copia de seguridad principal

> [!NOTE]
> La restauración de datos de una copia de seguridad que se ha organizado en niveles en la nube se realiza a la velocidad de la nube.

En la ilustración siguiente, se muestra la asignación de un volumen normal a un trabajo de copia de seguridad. En este caso, todas las copias de seguridad semanales se asignan al disco Saturday Full (Completa sábado) y las copias de seguridad incrementales se asignan a los discos Monday-Friday Incremental (Incremental lunes a viernes). Todas las copias de seguridad y restauraciones proceden de un volumen en capas de StorSimple.

![Diagrama lógico de la configuración de destino de copia de seguridad principal](./media/storsimple-configure-backup-target-using-netbackup/primarybackuptargetdiagram.png)

### <a name="storsimple-as-a-primary-backup-target-gfs-schedule-example"></a>Ejemplo de programación GFS de StorSimple como destino de copia de seguridad principal

A continuación se muestra un ejemplo de programación de rotación GFS para cuatro semanas, mensual, y anual:

| Frecuencia o tipo de copia de seguridad | Completo | Incremental (días 1-5)  |   
|---|---|---|
| Semanal (semanas 1-4) | Sábado | Lunes-viernes |
| Mensual  | Sábado  |   |
| Anual | Sábado  |   |

## <a name="assigning-storsimple-volumes-to-a-netbackup-backup-job"></a>Asignación de volúmenes de StorSimple a un trabajo de copia de seguridad de NetBackup

En la secuencia siguiente se asume que NetBackup y el host de destino están configurados según la guía del agente de NetBackup.

### <a name="to-assign-storsimple-volumes-to-a-netbackup-backup-job"></a>Para asignar volúmenes de StorSimple a un trabajo de copia de seguridad de NetBackup

1. En la consola de administración NetBackup, seleccione **NetBackup administración** (Administración de NetBackup), haga clic con el botón derecho en **Policies** (Directivas) y después seleccione **New Policy** (Nueva directiva).

   ![Consola de administración de NetBackup, creación de una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage6.png)

2. En el cuadro de diálogo **Add a New Policy** (Agregar una nueva directiva), escriba un nombre para esta y después seleccione la casilla **Use Policy Configuration Wizard** (Usar Asistente para la configuración de directivas). Seleccione **Aceptar**.

   ![Consola de administración de NetBackup, cuadro de diálogo para agregar una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage7.png)

3. En el Asistente para la configuración de directivas de copia de seguridad, seleccione el tipo de copia de seguridad que desee y después seleccione **Next** (Siguiente).

   ![Consola de administración de NetBackup, selección del tipo de copia de seguridad](./media/storsimple-configure-backup-target-using-netbackup/nbimage8.png)

4. Para establecer el tipo de directiva, seleccione **Standard** (Estándar) y luego seleccione **Next** (Siguiente).

   ![Consola de administración de NetBackup, selección del tipo de directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage9.png)

5. Seleccione el host, seleccione la casilla **Detect client operating system** (Detectar el sistema operativo cliente) y después seleccione **Add** (Agregar). Seleccione **Next** (Siguiente).

   ![Consola de administración de NetBackup, enumeración de clientes en una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage10.png)

6. Seleccione las unidades de las que desea realizar una copia de seguridad.

   ![Consola de administración de NetBackup, selecciones de copia de seguridad para una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage11.png)

7. Seleccione los valores de frecuencia y retención que cumplen los requisitos de rotación de su copia de seguridad.

   ![Consola de administración de NetBackup, frecuencia y rotación de copias de seguridad para una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage12.png)

8. Select **Next** > **Next** > **Finish** (Siguiente > Siguiente > Finalizar).  Puede modificar la programación después de crear la directiva.

9. Seleccione expandir la directiva que ha creado y después seleccione **Schedules** (Programaciones).

   ![Consola de administración de NetBackup, programaciones para una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage13.png)

10. Haga clic con el botón derecho **Differential-Inc** (Incremento diferencial, seleccione **Copy to new** (Copiar en nueva) y después seleccione **OK** (Aceptar).

    ![Consola de administración de NetBackup, copia de programaciones a una nueva directiva](./media/storsimple-configure-backup-target-using-netbackup/nbimage14.png)

11. A continuación, haga clic con el botón derecho en la programación recién creada y seleccione **Change** (Cambiar).

12. En la pestaña **Attributes** (Atributos), seleccione la casilla **Override policy storage selection** (Invalidar selección de almacenamiento de directiva) y seleccione el volumen donde van las copias de seguridad incrementales del lunes.

    ![Consola de administración de NetBackup, cambio de programación](./media/storsimple-configure-backup-target-using-netbackup/nbimage15.png)

13. En la pestaña **Start Window** (Ventana de inicio), seleccione la ventana de tiempo de las copias de seguridad.

    ![Consola de administración de NetBackup, cambio de ventana de inicio](./media/storsimple-configure-backup-target-using-netbackup/nbimage16.png)

14. Seleccione **Aceptar**.

15. Repita los pasos del 10 al 14 para cada copia de seguridad incremental. Seleccione el volumen y la programación apropiados para cada copia de seguridad que cree.

16. A continuación, haga clic con el botón derecho en la programación **Differential-inc** (Incremento diferencial) y elimínela.

17. Modifique la programación Completa para ajustarla a las necesidades de su copia de seguridad.

    ![Consola de administración de NetBackup, cambio de programación completa](./media/storsimple-configure-backup-target-using-netbackup/nbimage17.png)

18. Cambie la ventana de inicio.

    ![Consola de administración de NetBackup, cambio de la ventana de inicio](./media/storsimple-configure-backup-target-using-netbackup/nbimage18.png)

19. La programación final tiene este aspecto:

    ![Consola de administración de NetBackup, programación final](./media/storsimple-configure-backup-target-using-netbackup/nbimage19.png)

## <a name="set-up-storsimple-as-a-secondary-backup-target"></a>Configuración de StorSimple como destino de copia de seguridad secundario

> [!NOTE]
>La restauración de datos de una copia de seguridad que se ha organizado en niveles en la nube se realiza a la velocidad de la nube.

En este modelo, es preciso un soporte físico de almacenamiento (que no sea StorSimple) que actúe como una memoria caché temporal. Por ejemplo, puede usar un volumen de matriz redundante de discos independientes (RAID) para ofrecer espacio, entrada y salida (E y S) y ancho de banda. Se recomienda usar RAID 5, 50 y 10.

En la ilustración siguiente aparecen tanto volúmenes locales (en el servidor) para la retención a corto plazo como volúmenes de archivos a largo plazo. En este escenario, todas las copias de seguridad se ejecutan en el volumen RAID local (en el servidor). Estas copias de seguridad se duplican periódicamente y se archivan en un volumen de archivos. Es importante ajustar el tamaño del volumen RAID local (en el servidor) de modo que pueda controlar los requisitos de rendimiento y capacidad de retención a corto plazo.

### <a name="storsimple-as-a-secondary-backup-target-gfs-example"></a>Ejemplo de GFS de StorSimple como destino de copia de seguridad secundario

![Diagrama lógico de StorSimple como destino de copia de seguridad secundario](./media/storsimple-configure-backup-target-using-netbackup/secondarybackuptargetdiagram.png)

En la tabla siguiente se muestra cómo configurar copias de seguridad para que se ejecuten en los discos locales y de StorSimple. Incluye requisitos de capacidad individual y total.

### <a name="backup-configuration-and-capacity-requirements"></a>Configuración de copia de seguridad y requisitos de capacidad

| Tipo de copia de seguridad y retención | Almacenamiento configurado | Tamaño (TiB) | Multiplicador de GFS | Capacidad total \* (TiB) |
|---|---|---|---|---|
| Semana 1 (completa e incremental) |Disco local (a corto plazo)| 1 | 1 | 1 |
| StorSimple semanas 2-4 |Disco de StorSimple (largo plazo) | 1 | 4 | 4 |
| Completa mensual |Disco de StorSimple (largo plazo) | 1 | 12 | 12 |
| Completa anual |Disco de StorSimple (largo plazo) | 1 | 1 | 1 |
|Requisito de tamaño de volúmenes de GFS |  |  |  | 18*|

\* La capacidad total incluye 17 TiB de discos de StorSimple y 1 TiB de volumen RAID local.


### <a name="gfs-example-schedule-gfs-rotation-weekly-monthly-and-yearly-schedule"></a>Ejemplo de programación GFS: programación semanal, mensual y anual de rotación de GFS

| Semana | Completo | Incremental día 1 | Incremental día 2 | Incremental día 3 | Incremental día 4 | Incremental día 5 |
|---|---|---|---|---|---|---|
| Semana 1 | Volumen RAID local  | Volumen RAID local | Volumen RAID local | Volumen RAID local | Volumen RAID local | Volumen RAID local |
| Semana 2 | StorSimple semanas 2-4 |   |   |   |   |   |
| Semana 3 | StorSimple semanas 2-4 |   |   |   |   |   |
| Semana 4 | StorSimple semanas 2-4 |   |   |   |   |   |
| Mensual | StorSimple mensual |   |   |   |   |   |
| Anual | StorSimple anual  |   |   |   |   |   |


## <a name="assign-storsimple-volumes-to-a-netbackup-archive-and-duplication-job"></a>Asignación de volúmenes de StorSimple a un trabajo de archivo y duplicación de NetBackup

Como NetBackup ofrece una amplia gama de opciones para la administración de almacenamiento y soportes físicos, le recomendamos que se comunique con Veritas o con el arquitecto de NetBackup para evaluar correctamente los requisitos de la directiva del ciclo de vida de almacenamiento (SLP).

Después de definir los grupos de disco inicial, debe definir tres directivas de ciclo de vida de almacenamiento adicionales, es decir, un total de cuatro directivas:
* LocalRAIDVolume
* StorSimpleWeek2-4
* StorSimpleMonthlyFulls
* StorSimpleYearlyFulls

### <a name="to-assign-storsimple-volumes-to-a-netbackup-archive-and-duplication-job"></a>Para asignar volúmenes de StorSimple a un trabajo de archivo y duplicación de NetBackup

1. En la consola de administración NetBackup, seleccione **Storage** > **Storage Lifecycle Policies** > **New Storage Lifecycle Policy** (Almacenamiento > Directivas de ciclo de vida de almacenamiento > Nueva directiva de ciclo de vida de almacenamiento).

   ![Consola de administración de NetBackup, nueva directiva de ciclo de vida de almacenamiento](./media/storsimple-configure-backup-target-using-netbackup/nbimage20.png)

2. Escriba un nombre para la instantánea y luego seleccione **Add** (Agregar).

3. En el cuadro de diálogo **New Operation** (Nueva operación), en la pestaña **Properties** (Propiedades), en **Operation** (Operación), seleccione **Backup** (Copia de seguridad). Seleccione los valores que desee para **Destination storage** (Almacenamiento de destino), **Retention type** (Tipo de retención), y **Retention period** (Periodo de retención). Seleccione **Aceptar**.

   ![Consola de administración de NetBackup, cuadro de diálogo para agregar una nueva operación](./media/storsimple-configure-backup-target-using-netbackup/nbimage22.png)

   Esto define la operación y el repositorio de la primera copia de seguridad.

4. Seleccione esta opción para resaltar la operación anterior y después seleccione **Add** (Agregar). En el cuadro de diálogo **Change Storage Operation** (Cambiar operación de almacenamiento), seleccione los valores que desee para **Destination storage** (Almacenamiento de destino), **Retention type** (Tipo de retención), y **Retention period** (Periodo de retención).

   ![Consola de administración de NetBackup, cuadro de diálogo para cambiar una operación de almacenamiento](./media/storsimple-configure-backup-target-using-netbackup/nbimage23.png)

5. Seleccione esta opción para resaltar la operación anterior y después seleccione **Add** (Agregar). En el cuadro de diálogo **New Storage Lifecycle Policy** (Nueva directiva de ciclo de vida de almacenamiento, agregue copias de seguridad mensuales durante un año.

   ![Consola de administración de NetBackup, cuadro de diálogo de nueva directiva de ciclo de vida de almacenamiento](./media/storsimple-configure-backup-target-using-netbackup/nbimage24.png)

6. Repita los pasos 4 y 5 hasta que haya creado la directiva de retención SLP integral que necesita.

   ![Consola de administración de NetBackup, agregación de directivas en el cuadro de diálogo de directiva de ciclo de vida de almacenamiento](./media/storsimple-configure-backup-target-using-netbackup/nbimage25.png)

7. Cuando haya terminado de definir la directiva de retención SLP, en **Policy** (Directiva), defina una directiva de copia de seguridad siguiendo los pasos indicados en [Assigning StorSimple volumes to a NetBackup backup job](#assigning-storsimple-volumes-to-a-netbackup-backup-job) (Asignación de volúmenes de StorSimple a un trabajo de copia de seguridad de NetBackup).

8. En **Schedules** (Programaciones), en el cuadro de diálogo **Change Schedule** (Cambiar programación), haga clic con el botón derecho en **Full** (Completa) y seleccione **Change** (Cambiar).

   ![Consola de administración de NetBackup, cuadro de diálogo de cambio de programación](./media/storsimple-configure-backup-target-using-netbackup/nbimage26.png)

9. Seleccione la casilla de verificación **Override policy storage selection** (Invalidar selección de almacenamiento de directiva) y después seleccione la directiva de retención SLP que creó en los pasos 1-6.

   ![Consola de administración de NetBackup, invalidación de selección de almacenamiento de directivas](./media/storsimple-configure-backup-target-using-netbackup/nbimage27.png)

10. Seleccione **OK** (Aceptar) y luego repita los mismos pasos para la programación de copia de seguridad incremental.

    ![Consola de administración de NetBackup, cuadro de diálogo de cambio de programación para copias de seguridad incrementales](./media/storsimple-configure-backup-target-using-netbackup/nbimage28.png)


| Retención de tipo de copia de seguridad | Tamaño (TiB) | Multiplicador de GFS\* | Capacidad total (TiB)  |
|---|---|---|---|
| Completa semanal |  1  |  4 | 4  |
| Incremental diaria  | 0.5  | 20 (los ciclos son iguales al número de semanas por mes) | 12 (2 para cuota adicional) |
| Completa mensual  | 1 | 12 | 12 |
| Completa anual | 1  | 10 | 10 |
| Requisito de GFS  |     |     | 38 |
| Cuota adicional  | 4  |    | Requisito de GFS, un total de 42 |

\* El multiplicador de GFS es el número de copias que es preciso proteger y retener para cumplir los requisitos de las directivas de copia de seguridad.

## <a name="storsimple-cloud-snapshots"></a>Instantáneas de nube de StorSimple

Las instantáneas de nube de StorSimple protegen los datos que residen en su dispositivo de StorSimple. Crear una instantánea en la nube es equivalente al envío de cintas de copia de seguridad locales a una instalación externa. Si utiliza el almacenamiento con redundancia geográfica de Azure, crear una instantánea en la nube es equivalente a enviar las cintas de copia de seguridad a varios sitios. Si necesita restaurar un dispositivo en caso de desastre, puede poner en línea otro dispositivo de StorSimple y realizar una conmutación por error. Después de dicha conmutación, se podría acceder a los datos (a velocidades de nube) desde la instantánea en la nube más reciente.

En la sección siguiente se describe cómo crear un script breve para iniciar y eliminar instantáneas en la nube de StorSimple durante el procesamiento posterior a la copia de seguridad.

> [!NOTE]
> Las instantáneas que se crean manualmente o mediante programación no siguen la directiva de expiración de instantáneas de StorSimple. En otras palabras, se deben eliminar manualmente o mediante programación.

### <a name="start-and-delete-cloud-snapshots-by-using-a-script"></a>Inicio y eliminación de instantáneas en la nube con un script

> [!NOTE]
> Evalúe cuidadosamente las repercusiones tanto en lo relativo al cumplimiento como a la retención de datos antes de eliminar una instantánea de StorSimple. Para obtener más información sobre cómo ejecutar un script posterior a la copia de seguridad, consulte la [documentación de NetBackup](http://www.veritas.com/docs/000094423).

### <a name="backup-lifecycle"></a>Ciclo de vida de copia de seguridad

![Diagrama del ciclo de vida de copia de seguridad](./media/storsimple-configure-backup-target-using-netbackup/backuplifecycle.png)

### <a name="requirements"></a>Requisitos

-   El servidor que ejecute el script debe tener acceso a los recursos de Azure en la nube.
-   La cuenta de usuario debe tener los permisos necesarios.
-   Debe haber configurada, pero no activada, una directiva de copia de seguridad de StorSimple con los volúmenes de StorSimple asociados.
-   Necesitará el nombre del recurso de StorSimple, la clave de registro, el nombre de dispositivo y el identificador de directiva de copia de seguridad.

### <a name="to-start-or-delete-a-cloud-snapshot"></a>Para iniciar o eliminar una instantánea en la nube

1. [Instale Azure PowerShell](/powershell/azure/).
2. Descargue y configure el script [Manage-CloudSnapshots.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Manage-CloudSnapshots.ps1) de PowerShell.
3. En el servidor que ejecuta el script, ejecute PowerShell como administrador. Asegúrese de ejecutar el script con `-WhatIf $true` para ver cuáles son los cambios que hará el script. Una vez que se complete la validación, pase `-WhatIf $false`. Ejecute el comando siguiente:
   ```powershell
   .\Manage-CloudSnapshots.ps1 -SubscriptionId [Subscription Id] -TenantId [Tenant ID] -ResourceGroupName [Resource Group Name] -ManagerName [StorSimple Device Manager Name] -DeviceName [device name] -BackupPolicyName [backup policyname] -RetentionInDays [Retention days] -WhatIf [$true or $false]
   ```
4. Agregue el script a su trabajo de copia de seguridad en NetBackup. Para ello, edite los comandos de pre- y posprocesamiento de sus opciones de trabajo de NetBackup.

> [!NOTE]
> Se recomienda ejecutar la directiva de copia de seguridad de instantáneas en la nube de StorSimple como script posterior al proceso al final del trabajo de copia de seguridad diaria. Para obtener más información sobre cómo realizar las operaciones de copia de seguridad y restauración en el entorno de aplicaciones de copia de seguridad para ayudarle a cumplir los valores de RPO y RTO, consulte a su arquitecto de copia de seguridad.

## <a name="storsimple-as-a-restore-source"></a>StorSimple como origen de restauración

Las restauraciones desde un dispositivo de StorSimple funcionan como las de cualquier dispositivo de almacenamiento en bloque. Las restauraciones de los datos que están en capas en la nube se producen a velocidades de nube. En el caso de los datos locales, las restauraciones se producen a la velocidad del disco local del dispositivo. Para obtener información sobre cómo realizar una restauración, consulte la [documentación de NetBackup](http://www.veritas.com/docs/000094423). Le recomendamos que siga los procedimientos recomendados de restauración de NetBackup.

## <a name="storsimple-failover-and-disaster-recovery"></a>Conmutación por error y recuperación ante desastres de StorSimple

> [!NOTE]
> En los escenarios de destino de copia de seguridad, StorSimple Cloud Appliance no se admite como destino de restauración.

Un desastre puede deberse a una serie de factores. En la tabla siguiente encontrará escenarios comunes de recuperación ante desastres.

| Escenario | Impacto | Cómo realizar la recuperación | Notas |
|---|---|---|---|
| Error de dispositivo de StorSimple | Se interrumpen las operaciones de copia de seguridad y restauración. | Reemplace el dispositivo con error y realice las operaciones de [conmutación por error y recuperación ante desastres de StorSimple](storsimple-device-failover-disaster-recovery.md). | Si es preciso realizar una operación de restauración inmediatamente después de la recuperación del dispositivo, los espacios de trabajo completos se recuperan de la nube al nuevo dispositivo. Todas las operaciones se realizan a velocidades de la nube. Este proceso de examen repetido del índice y el catálogo podría provocar el examen de todos los conjuntos de copia de seguridad y su extracción de la capa de nube a la capa de dispositivo local, lo que podría consumir mucho tiempo. |
| Error en el servidor de NetBackup | Se interrumpen las operaciones de copia de seguridad y restauración. | Recompile el servidor de copia de seguridad y realice la restauración de base de datos. | Debe volver a generar o restaurar el servidor de NetBackup en el sitio de recuperación ante desastres. Restaure la base de datos al punto más reciente. Si la base de datos de NetBackup restaurada no está sincronizada con los trabajos de copia de seguridad más recientes, será necesario realizar la indexación y la catalogación. Este proceso de examen repetido del índice y el catálogo puede provocar el examen de todos los conjuntos de copia de seguridad y su extracción de la capa de nube a la capa de dispositivo local. Esto hace que requiera mucho tiempo. |
| Error del sitio que da lugar a la pérdida del servidor de copia de seguridad y de StorSimple | Se interrumpen las operaciones de copia de seguridad y restauración. | Restaure primero StorSimple y después NetBackup. | Restaure primero StorSimple y después NetBackup. Si es preciso realizar una operación de restauración inmediatamente después de la recuperación del dispositivo, los espacios de trabajo completos se recuperan de la nube al nuevo dispositivo. Todas las operaciones se realizan a velocidades de la nube. |

## <a name="references"></a>Referencias

En este artículo se ha hecho referencia a los siguientes documentos:

- [Configurar E/S de múltiples rutas para el dispositivo StorSimple](storsimple-configure-mpio-windows-server.md)
- [Escenarios de almacenamiento: el aprovisionamiento fino](https://msdn.microsoft.com/library/windows/hardware/dn265487.aspx)
- [Using GPT drives](https://msdn.microsoft.com/windows/hardware/gg463524.aspx#EHD) (Uso de unidades de GPT)
- [Habilitar y configurar las instantáneas de carpetas compartidas](https://technet.microsoft.com/library/cc771893.aspx)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo [restaurar a partir de un conjunto de copia de seguridad](storsimple-restore-from-backup-set-u2.md).
- Obtenga más información sobre cómo realizar [la conmutación por error y la recuperación ante desastres en los dispositivos](storsimple-device-failover-disaster-recovery.md).
