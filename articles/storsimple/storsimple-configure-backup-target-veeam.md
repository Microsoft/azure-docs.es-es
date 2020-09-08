---
title: StorSimple serie 8000 como destino de copia de seguridad con Veeam | Microsoft Docs
description: Obtenga información acerca de la configuración del destino de copia de seguridad de StorSimple con Veeam y los procedimientos recomendados para la integración de ambas soluciones.
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
ms.date: 12/06/2016
ms.author: matd
ms.openlocfilehash: 71a5434d985aad4033e4392dd31e6b7d112692de
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183989"
---
# <a name="storsimple-as-a-backup-target-with-veeam"></a>StorSimple como destino de copia de seguridad con Veeam

## <a name="overview"></a>Información general

Azure StorSimple es una solución de almacenamiento en nube híbrida de Microsoft. StorSimple aborda las complejidades del crecimiento exponencial de datos mediante una cuenta Azure Storage como una extensión de la solución local y la organización en capas automática de datos en almacenamiento local y almacenamiento en nube.

En este artículo se describen la integración de StorSimple con Veeam y los procedimientos recomendados para la integración de ambas soluciones. También se ofrecen recomendaciones sobre cómo configurar Veeam para que se integre perfectamente con StorSimple. Nos remitimos a los procedimientos recomendados, los arquitectos de copias de seguridad y los administradores en cuanto a la mejor manera de configurar Veeam con el objetivo de cumplir los requisitos de copia de seguridad individual y los Acuerdos de Nivel de Servicio (SLA).

Aunque se muestran tanto los pasos de configuración como los conceptos clave, este artículo no es una guía detallada de configuración o instalación. Se supone que la infraestructura y los componentes básicos funcionan perfectamente y están listos para admitir los conceptos que se describen.

### <a name="who-should-read-this"></a>¿A quiénes está dirigido este documento?

La información de este artículo será de especial utilidad para administradores de copia de seguridad, administradores de almacenamiento y arquitectos de almacenamiento con conocimientos de almacenamiento, Windows Server 2012 R2, Ethernet, servicios en la nube y Veeam.

### <a name="supported-versions"></a>Versiones compatibles

-   Veeam 9 y versiones posteriores
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
![Diagrama de niveles de Storsimple](./media/storsimple-configure-backup-target-using-veeam/image1.jpg)

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

| Capacidad de almacenamiento | 8100 | 8600 |
|---|---|---|
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

![Diagrama lógico de StorSimple como destino de copia de seguridad principal](./media/storsimple-configure-backup-target-using-veeam/primarybackuptargetlogicaldiagram.png)

### <a name="primary-target-backup-logical-steps"></a>Pasos lógicos de copias de seguridad en el destino principal

1.  El servidor de copia de seguridad se pone en contacto con el agente de copia de seguridad de destino y este transmite los datos al primero.
2.  El servidor de copia de seguridad escribe los datos en los volúmenes en capas de StorSimple.
3.  El servidor de copia de seguridad actualiza la base de datos del catálogo y después finaliza el trabajo de copia de seguridad.
4.  Un script de instantánea desencadena el administrador de instantáneas en la nube de StorSimple (iniciar o eliminar).
5.  El servidor de copia de seguridad elimina las copias de seguridad que han expirado en función de lo que establezca una directiva de retención.

### <a name="primary-target-restore-logical-steps"></a>Pasos lógicos de restauración del destino principal

1.  El servidor de copia de seguridad empieza a restaurar los datos apropiados del repositorio de almacenamiento.
2.  El agente de copia de seguridad recibe los datos del servidor de copia de seguridad.
3.  El servidor de copia de seguridad completa el trabajo de restauración.

## <a name="storsimple-as-a-secondary-backup-target"></a>StorSimple como destino de copia de seguridad secundario

En este escenario, los volúmenes de StorSimple se utilizan principalmente para la retención o el archivado a largo plazo.

En la siguiente ilustración se muestra una arquitectura en la que las copias de seguridad y restauraciones iniciales tienen como destino un volumen de alto rendimiento. Estas copias de seguridad se realizan y se archivan en un volumen en capas de StorSimple con una programación establecida.

Es importante asignar al volumen de alto rendimiento un tamaño adecuado para que pueda controlar la capacidad de la directiva de retención y los requisitos de rendimiento.

![Diagrama lógico de StorSimple como destino de copia de seguridad secundario](./media/storsimple-configure-backup-target-using-veeam/secondarybackuptargetlogicaldiagram.png)

### <a name="secondary-target-backup-logical-steps"></a>Pasos lógicos de copias de seguridad en el destino secundario

1.  El servidor de copia de seguridad se pone en contacto con el agente de copia de seguridad de destino y este transmite los datos al primero.
2.  El servidor de copia de seguridad escribe datos en el almacenamiento de alto rendimiento.
3.  El servidor de copia de seguridad actualiza la base de datos del catálogo y después finaliza el trabajo de copia de seguridad.
4.  El servidor de copia de seguridad copia las copias de seguridad en StorSimple en función de lo que establezca una directiva de retención.
5.  Un script de instantánea desencadena el administrador de instantáneas en la nube de StorSimple (iniciar o eliminar).
6.  El servidor de copia de seguridad elimina las copias de seguridad que han expirado en función de lo que establezca una directiva de retención.

### <a name="secondary-target-restore-logical-steps"></a>Pasos lógicos de restauración del destino secundario

1.  El servidor de copia de seguridad empieza a restaurar los datos apropiados del repositorio de almacenamiento.
2.  El agente de copia de seguridad recibe los datos del servidor de copia de seguridad.
3.  El servidor de copia de seguridad completa el trabajo de restauración.

## <a name="deploy-the-solution"></a>Implementación de la solución

La implementación de la solución requiere tres pasos:

1. Preparación de la infraestructura de red
2. Implementación del dispositivo StorSimple como destino de copia de seguridad
3. Implementación de Veeam

En las siguientes secciones se detallan cada uno de estos pasos.

### <a name="set-up-the-network"></a>Configuración de la red

Puesto que StorSimple es una solución integrada con la nube de Azure, requiere una conexión activa con la nube de Azure. Esta conexión se utiliza para operaciones tales como instantáneas en la nube, administración de datos y transferencia de metadatos, así como para almacenar en capas datos antiguos con menos acceso en el almacenamiento en la nube de Azure.

Para que la solución funcione de manera óptima, se recomienda seguir estos procedimientos de red recomendados:

-   El vínculo que conecta los niveles de StorSimple con Azure debe cumplir sus requisitos de ancho de banda. Esto se logra aplicando el nivel necesario de Calidad de servicio (QoS) a los conmutadores de su infraestructura para que cumplan sus Acuerdos de Nivel de Servicio de RPO y Objetivo de tiempo de recuperación (RTO).
-   Las latencias de acceso máximas de Azure Blob Storage deben rondar los 80 ms.

### <a name="deploy-storsimple"></a>Implementación de StorSimple

Para ver instrucciones detalladas para la implementación de StorSimple, consulte [Implementar el dispositivo StorSimple local](storsimple-deployment-walkthrough-u2.md).

### <a name="deploy-veeam"></a>Implementación de Veeam

Para ver los procedimientos recomendados para la instalación de Veeam, vaya a [Veeam Backup & Replication Best Practices](https://bp.veeam.expert/) (Procedimientos recomendados de copia de seguridad y replicación de Veeam), y lea la guía de usuario en [Veeam Help Center (Technical Documentation)](https://www.veeam.com/documentation-guides-datasheets.html) (Centro de ayuda de Veeam [Documentación técnica]).

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

- No use volúmenes distribuidos (creados por Windows Disk Management). Los volúmenes distribuidos no son compatibles.
- Dé formato a los volúmenes mediante NTFS con un tamaño de la unidad de asignación de 64 kB.
- Asigne los volúmenes de StorSimple directamente en el servidor de Veeam.
    - Use iSCSI para servidores físicos.


## <a name="best-practices-for-storsimple-and-veeam"></a>Procedimientos recomendados para StorSimple y Veeam

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

### <a name="veeam-best-practices"></a>Procedimientos recomendados para Veeam

-   La base de datos de Veeam debe ser encontrarse en el servidor, no puede residir en un volumen de StorSimple.
-   Para recuperación ante desastres, realice una copia de la base de datos de Veeam en un volumen de StorSimple.
-   Para esta solución se admiten copias de seguridad completas e incrementales de Veeam. Se recomienda no usar copias de seguridad sintéticas y diferenciales.
-   Los archivos de datos de copia de seguridad solo deben contener los datos de un trabajo concreto. Por ejemplo, no se permiten anexos de medios entre distintos trabajos.
-   Desactive la comprobación de trabajos. Si es necesaria, se debe programar que se realice una comprobación después del último trabajo de copia de seguridad. Es importante saber que este trabajo afecta a la ventana de copia de seguridad.
-   Active la asignación previa del soporte físico.
-   Asegúrese de que está activado el procesamiento en paralelo.
-   Desactive la compresión.
-   Desactive la desduplicación en el trabajo de copia de seguridad.
-   Establezca la optimización en **LAN Target** (LAN destino).
-   Active **Create active full backup** (Crear copia de seguridad completa activa) (cada 2 semanas).
-   En el repositorio de la copia de seguridad, configure **Use per-VM backup files** (Usar archivos de copia de seguridad por VM).
-   En **Use multiple upload streams per job** (Usar varias transmisiones de carga por trabajo), seleccione **8** (se permite un máximo de 16). Suba o baje este número en función del uso de la CPU en el dispositivo de StorSimple.

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

## <a name="set-up-veeam-storage"></a>Configuración del almacenamiento de Veeam

### <a name="to-set-up-veeam-storage"></a>Para configurar el almacenamiento de Veeam

1.  En la consola de Veeam Backup and Replication, en **Repository Tools** (Herramientas de repositorio), vaya a **Backup Infrastructure** (Infraestructura de copia de seguridad). Haga clic con el botón derecho en **Backup Repositories** (Repositorios de copia de seguridad) y seleccione **Add Backup Repository** (Agregar repositorio de copia de seguridad).

    ![consola de administración de Veeam, pantalla de repositorio de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage1.png)

2.  En el cuadro de diálogo **New Backup Repository** (Nuevo repositorio de copia de seguridad), escriba un nombre y una descripción para el repositorio. Seleccione **Next** (Siguiente).

    ![consola de administración de Veeam, página de nombre y descripción](./media/storsimple-configure-backup-target-using-veeam/veeamimage2.png)

3.  Para el tipo, seleccione **Microsoft Windows server**. Seleccione el servidor de Veeam. Seleccione **Next** (Siguiente).

    ![consola de administración de Veeam, selección del tipo de repositorio de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage3.png)

4.  Para especificar el valor de **Location** (Ubicación), busque y seleccione el volumen. Seleccione la casilla **Limit maximum concurrent tasks to:** (Limitar el número máximo de tareas simultáneas a:) y establezca el valor en **4**. Esto garantiza que solo se procesen cuatro discos virtuales simultáneamente mientras se procesa cada máquina virtual (VM). Seleccione el botón **Advanced** (Opciones avanzadas).

    ![consola de administración de Veeam, selección de volumen](./media/storsimple-configure-backup-target-using-veeam/veeamimage4.png)


5.  En el cuadro de diálogo **Storage Compatibility Settings** (Configuración de compatibilidad de almacenamiento), seleccione la casilla **Use per-VM backup files** (Usar por archivo de copia de seguridad de VM).

    ![consola de administración de Veeam, configuración de compatibilidad de almacenamiento](./media/storsimple-configure-backup-target-using-veeam/veeamimage5.png)

6.  En el cuadro de diálogo **New Backup Repository** (Nuevo repositorio de copia de seguridad), seleccione la casilla **Enable vPower NFS service on the mount server (recommended)** (Habilitar el servicio vPower NFS en el servidor de montaje [recomendado]). Seleccione **Next** (Siguiente).

    ![consola de administración de Veeam, pantalla de repositorio de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage6.png)

7.  Revise la configuración y, a continuación, seleccione **Next** (Siguiente).

    ![consola de administración de Veeam, pantalla de repositorio de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage7.png)

    Se agrega un repositorio al servidor de Veeam.

## <a name="set-up-storsimple-as-a-primary-backup-target"></a>Configuración de StorSimple como destino de copia de seguridad principal

> [!IMPORTANT]
> La restauración de datos de una copia de seguridad que se ha organizado en niveles en la nube se realiza a la velocidad de la nube.

En la ilustración siguiente, se muestra la asignación de un volumen normal a un trabajo de copia de seguridad. En este caso, todas las copias de seguridad semanales se asignan al disco Saturday Full (Completa sábado) y las copias de seguridad incrementales se asignan a los discos Monday-Friday Incremental (Incremental lunes a viernes). Todas las copias de seguridad y restauraciones proceden de un volumen en capas de StorSimple.

![Diagrama lógico de la configuración de destino de copia de seguridad principal](./media/storsimple-configure-backup-target-using-veeam/primarybackuptargetdiagram.png)

### <a name="storsimple-as-a-primary-backup-target-gfs-schedule-example"></a>Ejemplo de programación GFS de StorSimple como destino de copia de seguridad principal

A continuación se muestra un ejemplo de programación de rotación GFS para cuatro semanas, mensual, y anual:

| Frecuencia o tipo de copia de seguridad | Completo | Incremental (días 1-5)  |   
|---|---|---|
| Semanal (semanas 1-4) | Sábado | Lunes-viernes |
| Mensual  | Sábado  |   |
| Anual | Sábado  |   |


### <a name="assign-storsimple-volumes-to-a-veeam-backup-job"></a>Asignación de volúmenes de StorSimple a un trabajo de copia de seguridad de Veeam

Para el escenario de destino de copia de seguridad principal, cree un trabajo diario con el volumen de Veeam StorSimple principal. Para un escenario de destino de copia de seguridad secundaria, cree un trabajo diario mediante almacenamiento conectado directamente (DAS), almacenamiento conectado a red (NAS) o un grupo de discos sin más (JBOD).

#### <a name="to-assign-storsimple-volumes-to-a-veeam-backup-job"></a>Para asignar volúmenes de StorSimple a un trabajo de copia de seguridad de Veeam

1.  En la consola de Veeam Backup and Replication, seleccione **Backup & Replication** (Copia de seguridad y replicación). Haga clic con el botón derecho en **Backup** (Copia de seguridad) y luego seleccione **VMware** o **Hyper-V**, en función del entorno.

    ![consola de administración de Veeam, nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage8.png)

2.  En el cuadro de diálogo **New Backup Job** (Nuevo trabajo de copia de seguridad), escriba un nombre y una descripción para el trabajo de copia de seguridad.

    ![consola de administración de Veeam, nueva página de trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage9.png)

3.  Seleccione la máquina virtual en la que se va a realizar la copia de seguridad.

    ![consola de administración de Veeam, nueva página de trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage10.png)

4.  Seleccione los valores que desee para **Backup proxy** (Proxy de copia de seguridad) y **Backup repository** (Repositorio de copia de seguridad). Seleccione un valor para **Restore points to keep on disk** (Restaurar puntos que se mantienen en disco) según las definiciones de RPO/RTO de su entorno en el almacenamiento conectado localmente. Seleccione **Advanced** (Avanzadas).

    ![consola de administración de Veeam, nueva página de trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage11.png)

5. En el cuadro de diálogo **Advanced Settings** (Configuración avanzada), en la pestaña **Backup** (Copia de seguridad), seleccione **Incremental**. Compruebe que no esté seleccionada la casilla **Create synthetic full backups periodically** (Crear copias de seguridad completas y sintéticas periódicamente). Seleccione la casilla **Create active full backups periodically** (Crear copias de seguridad completas y activas periódicamente). En **Active full backup** (Copia de seguridad completa y activa), seleccione la casilla **Weekly on selected days** (Semanalmente en los días seleccionados) para el sábado.

    ![consola de administración de Veeam, página de configuración avanzada de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage12.png)

6. En la pestaña **Storage** (Almacenamiento), compruebe que no esté marcada la casilla **Enable inline data deduplication** (Habilitar la desduplicación de datos en línea). Seleccione la casilla **Exclude swap file blocks** (Excluir bloques de archivos de intercambio) y la casilla **Exclude deleted file blocks** (Excluir bloques de archivos eliminados). Establezca **Compression level** (Nivel de compresión) en **None** (Ninguno). En **Storage optimization** (Optimización de almacenamiento), seleccione **LAN Target** (LAN destino) si desea que el rendimiento sea equilibrado y que haya desduplicación. Seleccione **Aceptar**.

    ![consola de administración de Veeam, página de configuración avanzada de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage13.png)

    Para obtener información sobre la configuración de desduplicación y de compresión de Veeam, consulte [Data Compression and Deduplication](https://helpcenter.veeam.com/backup/vsphere/compression_deduplication.html) (Compresión y desduplicación de datos).

7.  En el cuadro de diálogo **Edit Backup Job** (Editar trabajo de copia de seguridad), puede seleccionar la casilla (opcional) **Enable application-aware processing** (Habilitar procesamiento con reconocimiento de la aplicación).

    ![consola de administración de Veeam, página de procesamiento de invitado de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage14.png)

8.  Configure la programación para que se ejecute una vez al día a una hora que puede especificar.

    ![consola de administración de Veeam, página de programación de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage15.png)

## <a name="set-up-storsimple-as-a-secondary-backup-target"></a>Configuración de StorSimple como destino de copia de seguridad secundario

> [!NOTE]
> La restauración de datos de una copia de seguridad que se ha organizado en niveles en la nube se realiza a la velocidad de la nube.

En este modelo, es preciso un soporte físico de almacenamiento (que no sea StorSimple) que actúe como una memoria caché temporal. Por ejemplo, puede usar un volumen de matriz redundante de discos independientes (RAID) para ofrecer espacio, entrada y salida (E y S) y ancho de banda. Se recomienda usar RAID 5, 50 y 10.

En la ilustración siguiente aparecen tanto volúmenes locales (en el servidor) para la retención a corto plazo como volúmenes de archivo a largo plazo. En este escenario, todas las copias de seguridad se ejecutan en el volumen RAID local (en el servidor). Estas copias de seguridad se duplican periódicamente y se archivan en un volumen de archivo. Es importante ajustar el tamaño del volumen RAID local (en el servidor) de modo que pueda controlar los requisitos de rendimiento y capacidad de retención a corto plazo.

![Diagrama lógico de StorSimple como destino de copia de seguridad secundario](./media/storsimple-configure-backup-target-using-veeam/secondarybackuptargetdiagram.png)

### <a name="storsimple-as-a-secondary-backup-target-gfs-example"></a>Ejemplo de GFS de StorSimple como destino de copia de seguridad secundario

En la tabla siguiente se muestra cómo configurar copias de seguridad para que se ejecuten en los discos locales y de StorSimple. Incluye requisitos de capacidad individual y total.

| Tipo de copia de seguridad y retención | Almacenamiento configurado | Tamaño (TiB) | Multiplicador de GFS | Capacidad total \* (TiB) |
|---|---|---|---|---|
| Semana 1 (completa e incremental) |Disco local (a corto plazo)| 1 | 1 | 1 |
| StorSimple semanas 2-4 |Disco de StorSimple (largo plazo) | 1 | 4 | 4 |
| Completa mensual |Disco de StorSimple (largo plazo) | 1 | 12 | 12 |
| Completa anual |Disco de StorSimple (largo plazo) | 1 | 1 | 1 |
|Requisito de tamaño de volúmenes de GFS |  |  |  | 18*|

\* La capacidad total incluye 17 TiB de discos de StorSimple y 1 TiB de volumen RAID local.


### <a name="gfs-example-schedule"></a>Programación de ejemplo de GFS

Programación semanal, mensual y anual de rotación de GFS

| Semana | Completo | Incremental día 1 | Incremental día 2 | Incremental día 3 | Incremental día 4 | Incremental día 5 |
|---|---|---|---|---|---|---|
| Semana 1 | Volumen RAID local  | Volumen RAID local | Volumen RAID local | Volumen RAID local | Volumen RAID local | Volumen RAID local |
| Semana 2 | StorSimple semanas 2-4 |   |   |   |   |   |
| Semana 3 | StorSimple semanas 2-4 |   |   |   |   |   |
| Semana 4 | StorSimple semanas 2-4 |   |   |   |   |   |
| Mensual | StorSimple mensual |   |   |   |   |   |
| Anual | StorSimple anual  |   |   |   |   |   |

### <a name="assign-storsimple-volumes-to-a-veeam-copy-job"></a>Asignación de volúmenes de StorSimple a un trabajo de copia de Veeam

#### <a name="to-assign-storsimple-volumes-to-a-veeam-copy-job"></a>Para asignar volúmenes de StorSimple a un trabajo de copia de Veeam

1.  En la consola de Veeam Backup and Replication, seleccione **Backup & Replication** (Copia de seguridad y replicación). Haga clic con el botón derecho en **Backup** (Copia de seguridad) y luego seleccione **VMware** o **Hyper-V**, en función del entorno.

    ![consola de administración de Veeam, página de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage16.png)

2.  En el cuadro de diálogo **New Backup Copy Job** (Nuevo trabajo de copia de copia de seguridad), escriba un nombre y una descripción para el trabajo.

    ![consola de administración de Veeam, página de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage17.png)

3.  Seleccione las máquinas virtuales que desea procesar. Seleccione desde copias de seguridad y después seleccione la copia de seguridad diaria que creó anteriormente.

    ![consola de administración de Veeam, página de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage18.png)

4.  Excluya objetos del trabajo de copia de seguridad si fuera necesario.

5.  Seleccione el repositorio de copia de seguridad y establezca un valor para **Restore points to keep** (Puntos de restauración que se conservarán). Asegúrese de seleccionar la casilla **Keep the following restore points for archival purposes** (Conservar los siguientes puntos de restauración para fines de archivado). Defina la frecuencia de copia de seguridad y seleccione **Advanced** (Avanzado).

    ![consola de administración de Veeam, página de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage19.png)

6.  Especifique las siguientes opciones avanzadas:

    * En la pestaña **Maintenance** (Mantenimiento), desactive la protección contra daño en el nivel de almacenamiento.

    ![consola de administración de Veeam, página de configuración avanzada de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage20.png)

    * En la pestaña **Storage** (Almacenamiento), asegúrese de que la desduplicación y la compresión están desactivadas.

    ![consola de administración de Veeam, página de configuración avanzada de nuevo trabajo de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/veeamimage21.png)

7.  Especifique que la transferencia de datos es directa.

8.  Defina la programación de la ventana de copia de seguridad según sus necesidades y después finalice al asistente.

Para obtener más información, consulte [Create backup copy jobs](https://helpcenter.veeam.com/backup/hyperv/backup_copy_create.html) (Creación de trabajos de copia de seguridad).

## <a name="storsimple-cloud-snapshots"></a>Instantáneas de nube de StorSimple

Las instantáneas de nube de StorSimple protegen los datos que residen en su dispositivo de StorSimple. Crear una instantánea en la nube es equivalente al envío de cintas de copia de seguridad locales a una instalación externa. Si utiliza el almacenamiento con redundancia geográfica de Azure, crear una instantánea en la nube es equivalente a enviar las cintas de copia de seguridad a varios sitios. Si necesita restaurar un dispositivo en caso de desastre, puede poner en línea otro dispositivo de StorSimple y realizar una conmutación por error. Después de dicha conmutación, se podría acceder a los datos (a velocidades de nube) desde la instantánea en la nube más reciente.

En la sección siguiente se describe cómo crear un script breve para iniciar y eliminar instantáneas en la nube de StorSimple durante el procesamiento posterior a la copia de seguridad.

> [!NOTE]
> Las instantáneas que se crean manualmente o mediante programación no siguen la directiva de expiración de instantáneas de StorSimple. En otras palabras, se deben eliminar manualmente o mediante programación.

### <a name="start-and-delete-cloud-snapshots-by-using-a-script"></a>Inicio y eliminación de instantáneas en la nube con un script

> [!NOTE]
> Evalúe cuidadosamente las repercusiones tanto en lo relativo al cumplimiento como a la retención de datos antes de eliminar una instantánea de StorSimple. Para obtener más información sobre cómo ejecutar un script posterior a la copia de seguridad, consulte la documentación de Veeam.


### <a name="backup-lifecycle"></a>Ciclo de vida de copia de seguridad

![Diagrama del ciclo de vida de copia de seguridad](./media/storsimple-configure-backup-target-using-veeam/backuplifecycle.png)

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
4. Para agregar el script a un trabajo de copia de seguridad, edite las opciones avanzadas del trabajo de Veeam.

    ![Pestaña de script de configuración avanzada de copia de seguridad de Veeam](./media/storsimple-configure-backup-target-using-veeam/veeamimage22.png)

Se recomienda ejecutar la directiva de copia de seguridad de instantáneas en la nube de StorSimple como script posterior al proceso al final del trabajo de copia de seguridad diaria. Para obtener más información sobre cómo realizar las operaciones de copia de seguridad y restauración en el entorno de aplicaciones de copia de seguridad para ayudarle a cumplir los valores de RPO y RTO, consulte a su arquitecto de copia de seguridad.

## <a name="storsimple-as-a-restore-source"></a>StorSimple como origen de restauración

Las restauraciones desde un dispositivo de StorSimple funcionan como las de cualquier dispositivo de almacenamiento en bloque. Las restauraciones de los datos que están en capas en la nube se producen a velocidades de nube. En el caso de los datos locales, las restauraciones se producen a la velocidad del disco local del dispositivo.

Con Veeam, logra una recuperación rápida pormenorizada a nivel de archivo a través de StorSimple mediante los exploradores integrados de la consola de Veeam. Use los exploradores de Veeam para recuperar elementos individuales, como mensajes de correo electrónico, objetos de Active Directory y elementos de SharePoint de las copias de seguridad. La recuperación puede realizarse sin que se produzca ninguna interrupción en la máquina virtual local. También puede hacer recuperaciones a partir de un momento específico de Azure SQL Database y bases de datos de Oracle. Veeam y StorSimple facilitan y agilizan el proceso de recuperación a nivel de elemento desde Azure. Para obtener información sobre cómo realizar una restauración, consulte la documentación de Veeam:

- Para [Exchange Server](https://www.veeam.com/microsoft-exchange-recovery.html)
- Para [Active Directory](https://www.veeam.com/microsoft-active-directory-explorer.html)
- Para [SQL Server](https://www.veeam.com/microsoft-sql-server-explorer.html)
- Para [SharePoint](https://www.veeam.com/microsoft-sharepoint-recovery-explorer.html)
- Para [Oracle](https://www.veeam.com/oracle-backup-recovery-explorer.html)


## <a name="storsimple-failover-and-disaster-recovery"></a>Conmutación por error y recuperación ante desastres de StorSimple

> [!NOTE]
> En los escenarios de destino de copia de seguridad, StorSimple Cloud Appliance no se admite como destino de restauración.

Un desastre puede deberse a una serie de factores. En la tabla siguiente encontrará escenarios comunes de recuperación ante desastres.

| Escenario | Impacto | Cómo realizar la recuperación | Notas |
|---|---|---|---|
| Error de dispositivo de StorSimple | Se interrumpen las operaciones de copia de seguridad y restauración. | Reemplace el dispositivo con error y realice las operaciones de [conmutación por error y recuperación ante desastres de StorSimple](storsimple-device-failover-disaster-recovery.md). | Si es preciso realizar una operación de restauración inmediatamente después de la recuperación del dispositivo, los espacios de trabajo completos se recuperan de la nube al nuevo dispositivo. Todas las operaciones se realizan a velocidades de la nube. Este proceso de examen repetido del índice y el catálogo podría provocar el examen de todos los conjuntos de copia de seguridad y su extracción de la capa de nube a la capa de dispositivo local, lo que podría consumir mucho tiempo. |
| Error de servidor de Veeam | Se interrumpen las operaciones de copia de seguridad y restauración. | Vuelva a compilar el servidor de copia de seguridad y realice la restauración de la base de datos como se detalla en [Veeam Help Center (Technical Documentation)](https://www.veeam.com/documentation-guides-datasheets.html) [Centro de ayuda de Veeam (documentación técnica)].  | Debe volver a generar o restaurar el servidor de Veeam en el sitio de recuperación ante desastres. Restaure la base de datos al punto más reciente. Si la base de datos de Veeam restaurada no está sincronizada con los trabajos de copia de seguridad más recientes, se requieren la indexación y la catalogación. Este proceso de examen repetido del índice y el catálogo puede provocar el examen de todos los conjuntos de copia de seguridad y su extracción de la capa de nube a la capa de dispositivo local. Esto hace que requiera mucho tiempo. |
| Error del sitio que da lugar a la pérdida del servidor de copia de seguridad y de StorSimple | Se interrumpen las operaciones de copia de seguridad y restauración. | Restaure primero StorSimple y después Veeam. | Restaure primero StorSimple y después Veeam. Si es preciso realizar una operación de restauración inmediatamente después de la recuperación del dispositivo, los espacios de trabajo completos se recuperan de la nube al nuevo dispositivo. Todas las operaciones se realizan a velocidades de la nube. |


## <a name="references"></a>Referencias

En este artículo se ha hecho referencia a los siguientes documentos:

- [Configurar E/S de múltiples rutas para el dispositivo StorSimple](storsimple-configure-mpio-windows-server.md)
- [Escenarios de almacenamiento: el aprovisionamiento fino](https://msdn.microsoft.com/library/windows/hardware/dn265487.aspx)
- [Using GPT drives](https://msdn.microsoft.com/windows/hardware/gg463524.aspx#EHD) (Uso de unidades de GPT)
- [Habilitar y configurar las instantáneas de carpetas compartidas](https://technet.microsoft.com/library/cc771893.aspx)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo [restaurar a partir de un conjunto de copia de seguridad](storsimple-restore-from-backup-set-u2.md).
- Obtenga más información sobre cómo realizar [la conmutación por error y la recuperación ante desastres en los dispositivos](storsimple-device-failover-disaster-recovery.md).
