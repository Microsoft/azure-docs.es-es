---
title: Cómo usar PerfInsights en Microsoft Azure | Microsoft Docs
description: Obtenga información sobre cómo usar PerfInsights para solucionar problemas de rendimiento de máquinas virtuales con Windows.
services: virtual-machines-windows'
documentationcenter: ''
author: anandhms
manager: dcscontentpm
editor: na
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 7abff8c33ea276b8b8aaeffd010d5f5e09440d9b
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86526646"
---
# <a name="how-to-use-perfinsights-in-azure"></a>Uso de PerfInsights en Azure

[PerfInsights](https://aka.ms/perfinsightsdownload) es una herramienta de diagnóstico de autoayuda que recopila y analiza los datos de diagnóstico y proporciona un informe para ayudar a solucionar los problemas de rendimiento de las máquinas virtuales Windows en Azure. Se puede ejecutar en máquinas virtuales como una herramienta independiente o directamente desde el portal mediante [Diagnóstico de rendimiento para máquinas virtuales de Azure](performance-diagnostics.md) o mediante la instalación de la [extensión Azure Performance Diagnostics VM](performance-diagnostics-vm-extension.md).

Si experimenta problemas de rendimiento con las máquinas virtuales, ejecute esta herramienta antes de ponerse en contacto con el servicio de soporte técnico.

## <a name="supported-troubleshooting-scenarios"></a>Escenarios admitidos de solución de problemas

PerfInsights puede recopilar y analizar varios tipos de información. En las siguientes secciones se tratan escenarios comunes.

### <a name="quick-performance-analysis"></a>Análisis rápido del rendimiento

En este escenario se recopila la configuración de disco y otra información importante, lo que incluye:

-   Registros de eventos

-   Estado de la red para todas las conexiones entrantes y salientes

-   Opciones de configuración de red y de firewall

-   Lista de tareas para todas las aplicaciones que se ejecutan actualmente en el sistema

-   Opciones de configuración de base de datos de Microsoft SQL Server (si la máquina virtual se identifica como un servidor que ejecuta SQL Server)

-   Contadores de confiabilidad de almacenamiento

-   Revisiones importantes de Windows

-   Controladores de filtro instalados

Se trata de un conjunto de información pasiva que no debería afectar al sistema. 

>[!Note]
>Este escenario se incluye automáticamente en cada uno de los escenarios siguientes:

### <a name="benchmarking"></a>Pruebas comparativas

En este escenario se ejecuta el banco de pruebas [Diskspd](https://github.com/Microsoft/diskspd) (IOPS y MBPS) para todas las unidades conectadas a la máquina virtual. 

> [!Note]
> Este escenario puede afectar al sistema y no se debe ejecutar en un sistema de producción activo. Si es necesario, ejecute este escenario en una ventana de mantenimiento dedicada para evitar cualquier problema. Un aumento de la carga de trabajo debido a una prueba de seguimiento o a un banco de pruebas puede afectar negativamente al rendimiento de la máquina virtual.
>

### <a name="performance-analysis"></a>Análisis de rendimiento

En este escenario se ejecuta un seguimiento de [contador de rendimiento](/windows/win32/perfctrs/performance-counters-portal) mediante los contadores que se especifican en el archivo RuleEngineConfig.json. Si la máquina virtual se identifica como un servidor en el que se ejecuta SQL Server, se ejecuta un seguimiento del contador de rendimiento. Para ello, se usan los contadores que se encuentran en el archivo RuleEngineConfig.json. Este escenario también incluye datos de diagnóstico de rendimiento.

### <a name="azure-files-analysis"></a>Análisis de Azure Files

En este escenario se ejecuta una captura especial del contador de rendimiento junto con un seguimiento de red. Dicha captura incluye todos los contadores de recursos compartidos de cliente de Bloque de mensajes del servidor (SMB). Los siguientes son algunos contadores de rendimiento de recursos compartidos de cliente de SMB clave que forman parte de la captura:

| **Tipo**     | **Contador de recursos compartidos de cliente de SMB** |
|--------------|-------------------------------|
| E/S         | Solicitudes de datos por segundo             |
|              | Solicitudes de lectura por segundo             |
|              | Solicitudes de escritura por segundo            |
| Latencia      | Media de solicitud de datos por segundo         |
|              | Media de lectura por segundo                 |
|              | Media de escritura por segundo                |
| Tamaño de E/S      | Prom. de bytes/solicitud de datos       |
|              | Prom. de bytes/lectura               |
|              | Prom. de bytes/escritura              |
| Throughput   | Bytes de datos por segundo                |
|              | Bytes de lectura por segundo                |
|              | Bytes de escritura por segundo               |
| Longitud de la cola | Prom. Longitud de la cola de lectura        |
|              | Prom. Longitud de la cola de escritura       |
|              | Prom. Longitud de la cola de datos        |

### <a name="advanced-performance-analysis"></a>Análisis avanzado del rendimiento

Al ejecutar un análisis avanzado del rendimiento, seleccione seguimientos que se ejecuten en paralelo. Si lo desea, puede ejecutarlos todos (Contador de rendimiento, Xperf, Red y StorPort).  

> [!Note]
> Este escenario puede afectar al sistema y no se debe ejecutar en un sistema de producción activo. Si es necesario, ejecute este escenario en una ventana de mantenimiento dedicada para evitar cualquier problema. Un aumento de la carga de trabajo debido a una prueba de seguimiento o a un banco de pruebas puede afectar negativamente al rendimiento de la máquina virtual.
>

## <a name="what-kind-of-information-is-collected-by-perfinsights"></a>¿Qué tipo de información recopila PerfInsights?

Se recopila información acerca de la máquina virtual Windows, la configuración de discos o bloques de almacenamiento, contadores de rendimiento, registros y diferentes seguimientos. Depende del escenario de rendimiento que se use. En la tabla siguiente se dan los detalles:

| Datos recopilados | Análisis rápido del rendimiento | Pruebas comparativas | Análisis de rendimiento | Análisis de Azure Files | Análisis avanzado del rendimiento |
|----------------------------------|----------------------------|------------------------------------|--------------------------|--------------------------------|----------------------|
| Información de los registros de eventos       | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Información del sistema                | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Asignación de volúmenes                        | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Asignación de discos                          | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Tareas en ejecución                     | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Contadores de confiabilidad de almacenamiento      | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Información de almacenamiento               | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Salida de fsutil                     | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Información del controlador del filtro                | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Salida de netstat                    | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Network configuration (Configuración de red)             | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Configuración de firewall            | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Configuración de SQL Server          | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Seguimientos de diagnóstico de rendimiento *  | Sí                        | Sí                                | Sí                      | Sí                  | Sí                  |
| Seguimiento de contadores de rendimiento **      |                            |                                    | Sí                      |                      | Sí                  |
| Seguimiento del contador de SMB **              |                            |                                    |                          | Sí                  |                      |
| Seguimiento del contador de SQL Server **       |                            |                                    | Sí                      |                      | Sí                  |
| Seguimiento de Xperf                       |                            |                                    |                          |                      | Sí                  |
| Seguimiento de StorPort                    |                            |                                    |                          |                      | Sí                  |
| Seguimiento de la red                     |                            |                                    |                          | Sí                  | Sí                  |
| Seguimiento del banco de pruebas Diskspd ***       |                            | Sí                                |                          |                      |                      |
|       |                            |                         |                                                   |                      |                      |

### <a name="performance-diagnostics-trace-"></a>Seguimiento del diagnóstico de rendimiento (*)

Se ejecuta un motor basado en reglas en segundo plano para recopilar datos y diagnosticar problemas de rendimiento en curso. Actualmente se admiten las reglas siguientes:

- Regla de HighCpuUsage: detecta períodos de uso elevado de la CPU y muestra los principales consumidores de uso de la CPU durante dichos períodos.
- Regla de HighDiskUsage: detecta períodos de uso elevado de los discos físicos y muestra los principales consumidores de uso de los discos en dichos períodos.
- Regla de HighResolutionDiskMetric: muestra métricas de IOPS, rendimiento y latencia de E/S por 50 milisegundos de cada disco físico. Ayuda a identificar rápidamente los períodos de limitación de disco.
- Regla de HighMemoryUsage: detecta períodos de uso de memoria elevados y muestra los principales consumidores de uso de memoria durante esos períodos.

> [!NOTE] 
> Actualmente, se admiten las versiones de Windows que incluyen .NET Framework 4.5 o versiones posteriores.

### <a name="performance-counter-trace-"></a>Seguimiento de contadores de rendimiento (\*\*)

Recopila los siguientes contadores de rendimiento:

- \Process, \Processor, \Memory, \Thread, \PhysicalDisk y \LogicalDisk
- \Cache\Dirty Pages, \Cache\Lazy Write Flushes/sec, \Server\Pool Nonpaged, Failures y \Server\Pool Paged Failures
- Varios contadores de \Network Interface, \IPv4\Datagrams, \IPv6\Datagrams, \TCPv4\Segments, \TCPv6\Segments,  \Network Adapter, \WFPv4\Packets, \WFPv6\Packets, \UDPv4\Datagrams, \UDPv6\Datagrams, \TCPv4\Connection, \TCPv6\Connection, \Network QoS Policy\Packets, \Per Processor Network Interface Card Activity, \Microsoft Winsock BSP

#### <a name="for-sql-server-instances"></a>Para instancias de SQL Server
- \SQL Server:Buffer Manager, \SQLServer:Resource Pool Stats y \SQLServer:SQL Statistics\
- \SQLServer:Locks, \SQLServer:General, Statistics
- \SQLServer:Access Methods

#### <a name="for-azure-files"></a>Para Azure Files
\Recursos compartidos de cliente de SMB

### <a name="diskspd-benchmark-trace-"></a>Seguimiento del banco de pruebas Diskspd (***)
Pruebas de carga de trabajo de E/S de Diskspd [disco del sistema operativo (escritura) y unidades de grupo (lectura/escritura)]

## <a name="run-the-perfinsights-tool-on-your-vm"></a>Ejecución de la herramienta PerfInsights en la máquina virtual

### <a name="what-do-i-have-to-know-before-i-run-the-tool"></a>¿Qué es necesario saber antes de ejecutar la herramienta? 

#### <a name="tool-requirements"></a>Requisitos de la herramienta

-  Esta herramienta se debe ejecutar en la máquina virtual que tiene el problema de rendimiento. 

-  Se admiten los siguientes sistemas operativos: Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 y Windows Server 2016; Windows 8.1 y Windows 10.

#### <a name="possible-problems-when-you-run-the-tool-on-production-vms"></a>Posibles problemas al ejecutar la herramienta en máquinas virtuales de producción

-  Tanto en los escenarios de pruebas comparativas como en el de "Análisis avanzado del rendimiento" que está configurado para usar Xperf o Diskspd, la herramienta podría afectar de manera negativa al rendimiento de la VM. Estos escenarios no se deben ejecutar en un entorno de producción en vivo.

-  Tanto en los escenarios de pruebas comparativas como en el de "Análisis avanzado del rendimiento" que está configurado para usar Diskspd, asegúrese de que ninguna otra actividad en segundo plano interfiera con la carga de trabajo de E/S.

-  De forma predeterminada, la herramienta usa la unidad de almacenamiento temporal para recopilar los datos. Si el seguimiento permanece habilitado durante más tiempo, es posible que la cantidad de datos que se recopilan sea relevante. Esto puede reducir la disponibilidad de espacio en el disco temporal, lo que puede afectar a las aplicaciones que usen esta unidad.

### <a name="how-do-i-run-perfinsights"></a>¿Cómo se puede ejecutar PerfInsights? 

Para ejecutar PerfInsights en una máquina virtual es preciso instalar la [extensión de máquina virtual de diagnóstico de rendimiento de Azure](performance-diagnostics-vm-extension.md). También puede ejecutarla como una herramienta independiente. 

**Instalación y ejecución de PerfInsights desde Azure Portal**

Para más información acerca de esta opción, consulte [Instalación de la extensión de diagnóstico de rendimiento](performance-diagnostics-vm-extension.md#install-the-extension).  

**Ejecución de PerfInsights en modo independiente**

Para ejecutar la herramienta PerfInsights, siga estos pasos:


1. Descargue [PerfInsights.zip](https://aka.ms/perfinsightsdownload).

2. Desbloquee el archivo PerfInsights.zip. Para ello, haga clic con el botón derecho en el archivo PerfInsights.zip y seleccione **Propiedades**. En la pestaña **General**, seleccione **Desbloquear** y haga clic en **Aceptar**. Así se asegura de que la herramienta se ejecuta sin advertencias de seguridad adicionales.  

    ![Captura de pantalla de las propiedades de PerfInsights con Desbloquear resaltado](media/how-to-use-perfInsights/pi-unlock-file.png)

3.  Expanda el archivo PerfInsights.zip comprimido en una unidad temporal (de forma predeterminada, suele ser la unidad D). 

4.  Abra el símbolo del sistema de Windows como administrador y, a continuación, ejecute PerfInsights.exe para ver los parámetros de línea de comandos disponibles.

    ```
    cd <the path of PerfInsights folder>
    PerfInsights
    ```
    ![Captura de pantalla de la salida de la línea de comandos de PerfInsights](media/how-to-use-perfInsights/pi-commandline.png)
    
    La sintaxis básica para ejecutar los escenarios de PerfInsights es:
    
    ```
    PerfInsights /run <ScenarioName> [AdditionalOptions]
    ```

    Puede usar el siguiente ejemplo para ejecutar el escenario de análisis del rendimiento durante 5 minutos:
    
    ```
    PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics
    ```

    Puede usar el ejemplo siguiente para ejecutar el escenario avanzado con seguimientos de contador Xperf y de rendimiento durante 5 minutos:
    
    ```
    PerfInsights /run advanced xp /d 300 /AcceptDisclaimerAndShareDiagnostics
    ```

    Puede usar el siguiente ejemplo para ejecutar el escenario de análisis del rendimiento durante 5 minutos y cargar el archivo zip en la cuenta de almacenamiento:
    
    ```
    PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics /sa <StorageAccountName> /sk <StorageAccountKey>
    ```

    Puede buscar todos los escenarios y opciones disponibles mediante el comando **/list**:
    
    ```
    PerfInsights /list
    ```

    >[!Note]
    >Antes de ejecutar un escenario, PerfInsights pide al usuario que acepte compartir información de diagnóstico y que acepte los términos de licencia. Use la opción **/AcceptDisclaimerAndShareDiagnostics** para omitir estos mensajes. 
    >
    >Si tiene una incidencia de soporte técnico activa con Microsoft y ejecuta PerfInsights a petición del ingeniero de soporte técnico con el que trabaja, asegúrese de proporcionar el número de la incidencia mediante la opción **/sr**.
    >
    >De forma predeterminada, PerfInsights intenta actualizarse a la versión más reciente, si está disponible. Use el parámetro **/SkipAutoUpdate** o **/sau** para omitir la actualización automática.  
    >
    >Si no se especifica el modificador de duración **/d**, PerfInsights le pide que reproduzca el problema mientras se ejecutan los escenarios vmslow, azurefiles y advanced. 

Cuando las operaciones o los seguimientos se completan, aparece un nuevo archivo en la misma carpeta que PerfInsights. El nombre del archivo es **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.zip.** Puede enviar este archivo al agente de soporte técnico para que lo analice o abrir el informe dentro del archivo ZIP para revisar las conclusiones y recomendaciones.

## <a name="review-the-diagnostics-report"></a>Revisión del informe de diagnóstico

En el archivo **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.zip.** , puede encontrar un informe HTML en el que se detallan las conclusiones de PerfInsights. Para revisar el informe, expanda el archivo **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.zip.** y después abra el archivo **PerfInsights Report.html**.

Haga clic en la pestaña **Conclusiones**.

![Captura de pantalla del informe de PerfInsights](media/how-to-use-perfInsights/pi-finding-tab.png)
![Screenshot of PerfInsights Report](media/how-to-use-perfInsights/pi-findings.png)

> [!NOTE] 
> Las conclusiones clasificadas como altas son problemas conocidos que pueden provocar problemas de rendimiento. Las conclusiones clasificadas como medias representan configuraciones no óptimas que no provocan necesariamente problemas de rendimiento. Las conclusiones clasificadas como bajas son solo declaraciones informativas.

Revise las recomendaciones y los vínculos de todas las conclusiones altas y medias. Aprenda cómo pueden afectar al rendimiento, así como los procedimientos recomendados para las configuraciones con optimización del rendimiento.

### <a name="storage-tab"></a>Pestaña Almacenamiento

La sección **Conclusiones** muestra diversas conclusiones y recomendaciones relacionadas con el almacenamiento.

En las secciones **Disk Map** (Mapa de discos) y **Volume Map** (Mapa de volúmenes) se describe la relación que los volúmenes lógicos y los discos físicos tienen entre sí.

En la perspectiva de los discos físicos (Disk Map), la tabla muestra todos los volúmenes lógicos que se ejecutan en el disco. En el ejemplo siguiente, **PhysicalDrive2** ejecuta dos volúmenes lógicos creados en varias particiones (J y H):

![Captura de la pestaña de discos](media/how-to-use-perfInsights/pi-disk-tab.png)

En la perspectiva de los volúmenes (Volume Map), las tablas muestran todos los discos físicos en cada volumen lógico. Observe que en el caso de los discos RAID y dinámicos, un volumen lógico se puede ejecutar en varios discos físicos. En el ejemplo siguiente *C:\\mount* es un punto de montaje configurado como *SpannedDisk* en los discos físicos 2 y 3:

![Captura de pantalla de la ficha de volúmenes](media/how-to-use-perfInsights/pi-volume-tab.png)

### <a name="sql-tab"></a>Pestaña SQL

Si la máquina virtual de destino hospeda alguna instancia de SQL Server, en el informe verá una pestaña adicional denominada **SQL**:

![Captura de pantalla de pestaña SQL](media/how-to-use-perfInsights/pi-sql-tab.png)

Esta sección contiene una pestaña **Findings** (Conclusiones) y pestañas adicionales para cada una de las instancias de SQL Server hospedadas en la máquina virtual.

La pestaña **Conclusiones** contiene una lista de todos los problemas de rendimiento relacionados con SQL encontrados, junto con las recomendaciones.

En el ejemplo siguiente, se muestra **PhysicalDrive0** (que se ejecuta en la unidad C). En el ejemplo siguiente, los archivos **modeldev** y **modellog** se encuentran en la unidad C y son de tipos diferentes (archivo de datos y registro de transacciones, respectivamente).

![Captura de pantalla de información de registro](media/how-to-use-perfInsights/pi-log-info.png)

Las pestañas de las instancias concretas de SQL Server contienen una sección general que muestra información básica acerca de la instancia seleccionada. También contienen secciones adicionales para información avanzada, lo que incluye los valores, las configuraciones y las opciones de usuario.

### <a name="diagnostic-tab"></a>Pestaña Diagnóstico
La pestaña **Diagnostic** (Diagnóstico) contiene información acerca de los mayores consumidores de CPU, disco y memoria mientras se ejecuta PerfInsights. También puede encontrar información acerca de las revisiones críticas que podría necesitar el sistema, la lista de tareas y los eventos importantes del sistema. 

## <a name="references-to-the-external-tools-used"></a>Referencias a las herramientas externas usadas

### <a name="diskspd"></a>Diskspd

Diskspd es un generador de carga de almacenamiento y una herramienta de prueba de rendimiento de Microsoft. Para más información, consulte [Diskspd](https://github.com/Microsoft/diskspd).

### <a name="xperf"></a>Xperf

Xperf es una herramienta de línea de comandos para capturar seguimientos desde Windows Performance Toolkit. Para más información, consulte [Windows Performance Toolkit -Xperf](/archive/blogs/ntdebugging/windows-performance-toolkit-xperf).

## <a name="next-steps"></a>Pasos siguientes

Puede cargar los registros e informes de diagnóstico en el Soporte técnico de Microsoft para que examinen con mayor profundidad. En el servicio de soporte técnico pueden solicitarle que transmita el resultado que genera PerfInsights para facilitarles el proceso de solución de los problemas.

La siguiente captura de pantalla muestra un mensaje similar al que puede recibir:

![Captura de pantalla de mensaje de ejemplo del Soporte técnico de Microsoft](media/how-to-use-perfInsights/pi-support-email.png)

Siga las instrucciones del mensaje para acceder al área de trabajo de transferencia de archivos. Para mayor seguridad, tiene que cambiar la contraseña la primera vez que la use.

Después de iniciar sesión, encontrará un cuadro de diálogo para cargar el archivo **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.zip** que recopiló PerfInsights.
