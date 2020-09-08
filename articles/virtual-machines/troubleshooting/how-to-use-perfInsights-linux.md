---
title: Uso de PerfInsights Linux en Microsoft Azure | Microsoft Docs
description: Aprenda a usar PerfInsights para solucionar problemas de rendimiento de máquinas virtuales Linux.
services: virtual-machines-linux'
documentationcenter: ''
author: anandhms
manager: cshepard
editor: na
tags: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 7/10/2019
ms.author: genli
ms.openlocfilehash: 1bf080ad4c4dc665e61d1075cf22c84d4cd66648
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121398"
---
# <a name="how-to-use-perfinsights"></a>Cómo usar PerfInsights

[PerfInsights Linux](https://aka.ms/perfinsightslinuxdownload) es una herramienta de diagnóstico de autoayuda que recopila y analiza los datos de diagnóstico y proporciona un informe para ayudar a solucionar los problemas de rendimiento de las máquinas virtuales Linux en Azure. Se puede ejecutar en máquinas virtuales compatibles como una herramienta independiente o directamente desde el portal mediante [Performance Diagnostics para máquinas virtuales de Azure](performance-diagnostics.md).

Si experimenta problemas de rendimiento con las máquinas virtuales, ejecute esta herramienta antes de ponerse en contacto con el servicio de soporte técnico.

## <a name="supported-troubleshooting-scenarios"></a>Escenarios admitidos de solución de problemas

PerfInsights puede recopilar y analizar varios tipos de información. En las siguientes secciones se tratan escenarios comunes.

### <a name="quick-performance-analysis"></a>Análisis rápido del rendimiento

En este escenario se recopila información básica, como la configuración de almacenamiento y hardware de la máquina virtual o diversos registros; esta información incluye:

- Información del sistema operativo

- Información del dispositivo PCI

- Registros generales del SO invitado

- Archivos de configuración

- Información de almacenamiento

- Configuración de la máquina virtual de Azure [(recopilada con Azure Instance Metadata Service)](../windows/instance-metadata-service.md)

- Lista de procesos en ejecución, disco, memoria y uso de CPU

- Información de redes

Se trata de un conjunto de información pasiva que no debería afectar al sistema.

>[!Note]
>El escenario del análisis rápido de rendimiento se incluye automáticamente en cada uno de los escenarios siguientes:

### <a name="performance-analysis"></a>Análisis de rendimiento

Este escenario es similar al análisis de rendimiento rápido, pero permite capturar información de diagnóstico durante más tiempo.

## <a name="what-kind-of-information-is-collected-by-perfinsights"></a>¿Qué tipo de información recopila PerfInsights?

Se recopila información sobre la máquina virtual Linux, el sistema operativo, los dispositivos de bloqueo, los grandes consumidores de recursos, la configuración, así como diversos registros. A continuación se muestran más detalles:

- Sistema operativo
  - Distribución y versión de Linux
  - Información del kernel
  - Información de controlador

- Hardware
  - Dispositivos PCI [`*`]

- Procesos y memoria
  - Lista de procesos (nombre de tarea, memoria usada, archivos abiertos)
  - Memoria física total, disponible y libre
  - Memoria de intercambio total, disponible y libre
  - Captura de generación de perfiles de CPU y uso de CPU por parte de los procesos a intervalos de 5 segundos
  - Captura de generación de perfiles de uso de memoria por parte de los procesos a intervalos de 5 segundos

- Redes  
  - Lista de adaptadores de red con estadísticas de adaptadores
  - Tabla de enrutamiento de red
  - Puertos abiertos y estado

- Storage
  - Lista de dispositivos de bloqueo
  - Lista de particiones
  - Lista de puntos de montaje
  - Información de volumen de MDADM
  - Información de volumen de LVM
  - Captura de generación de perfiles en todos los discos a intervalos de 5 segundos

- Registros
  - /var/log/messages
  - /var/log/syslog
  - /var/log/kern.log
  - /var/log/cron.log
  - /var/log/boot.log
  - /var/log/yum.log
  - /var/log/dpkg.log
  - /var/log/sysstat o /var/log/sa [`**`]
  - /var/log/cloud-init.log
  - /var/log/cloud-init-output.log
  - /var/log/gpu-manager.log
  - /var/log/waagent.log
  - /var/log/azure/[carpeta de extensión]/\*log\*
  - /var/opt/microsoft/omsconfig/omsconfig.log
  - /var/opt/microsoft/omsagent/log/omsagent.log
  - /etc/waagent.config
  - Salida de journalctl de los últimos cinco días

- [Metadatos de la instancia de máquina virtual de Azure](../windows/instance-metadata-service.md)

>[!Note]
>[`*`] La información de PCI todavía no se recopila en las distribuciones Debian y SLES.
> 
>[`**`] /var/log/sysstat o/var/log/sa contiene los archivos de informe de actividad del sistema (SAR) recopilados por el paquete sysstat. Si el paquete sysstat no está instalado en la VM, la herramienta PerfInsights proporciona una recomendación para instalarlo.

## <a name="run-the-perfinsights-linux-on-your-vm"></a>Ejecución de PerfInsights Linux en la máquina virtual

### <a name="what-do-i-have-to-know-before-i-run-the-tool"></a>Qué es necesario saber antes de ejecutar la herramienta

#### <a name="tool-requirements"></a>Requisitos de la herramienta

- Esta herramienta se debe ejecutar en la máquina virtual que tiene el problema de rendimiento.
- Python 3.x o Python 2.7 debe estar instalado en la VM.

- Actualmente se admiten las siguientes distribuciones:

    | Distribución               | Versión                                         |
    |----------------------------|-------------------------------------------------|
    | Servidor de Oracle Linux        | 6.10 [`*`], 7.3, 7.6, 7.5 (imagen de Marketplace Oracle-Database-Ee 13.8)|
    | CentOS                     | 6.5 [`*`], 7.6                                    |
    | RHEL                       | 7.2, 7.5, 8.0 [`*`]                               |
    | Ubuntu                     | 14.04, 16.04, 18.04                               |
    | Debian                     | 8, 9, 10 [`*`]                                    |
    | SLES                       | 12 SP4 [`*`]                                      |
    |                            |                                                   |

>[!Note]
>[`*`] Consulte la sección [Problemas conocidos](#known-issues).

### <a name="known-issues"></a>Problemas conocidos

- RHEL 8 no tiene Python instalado de forma predeterminada. Para ejecutar PerfInsights Linux, antes debe instalar Python 2.7.

- Puede producirse un error en la recopilación de información del agente invitado en CentOS 6.x.

- En distribuciones basadas en Debian no se recopila información sobre los dispositivos PCI.

- La información de LVM se recopila parcialmente en algunas distribuciones.

### <a name="how-do-i-run-perfinsights"></a>Ejecución de PerfInsights

Para ejecutar PerfInsights en una máquina virtual es preciso instalar Azure Performance Diagnostics desde Azure Portal. También puede ejecutarla como una herramienta independiente.

>[!Note]
>PerfInsights simplemente recopila y analiza los datos. No realiza ninguna modificación en el sistema.

#### <a name="install-and-run-perfinsights-from-the-azure-portal"></a>Instalación y ejecución de PerfInsights desde Azure Portal

Para más información acerca de esta opción, consulte [Azure Performance Diagnostics](performance-diagnostics.md).  

#### <a name="run-perfinsights-in-standalone-mode"></a>Ejecución de PerfInsights en modo independiente

Para ejecutar la herramienta PerfInsights, siga estos pasos:

1. Descargue [PerfInsights.tar.gz](https://aka.ms/perfinsightslinuxdownload) en una carpeta de la máquina virtual y extraiga el contenido mediante los siguientes comandos desde el terminal.

   ```bash
   wget https://download.microsoft.com/download/9/F/8/9F80419C-D60D-45F1-8A98-718855F25722/PerfInsights.tar.gz
   ```

   ```bash
   tar xzvf PerfInsights.tar.gz
   ```

2. Vaya a la carpeta que contiene el archivo `perfinsights.py` y, a continuación, ejecute `perfinsights.py` para ver los parámetros de línea de comandos disponibles.

    ```bash
    cd <the path of PerfInsights folder>
    sudo python perfinsights.py
    ```

    ![Captura de pantalla de la salida de la línea de comandos de PerfInsights Linux](media/how-to-use-perfinsights-linux/perfinsights-linux-commandline.png)

    La sintaxis básica para ejecutar los escenarios de PerfInsights es:

    ```bash
    sudo python perfinsights.py -r <ScenarioName> -d [duration]<H | M | S> [AdditionalOptions]
    ```

    Puede usar el ejemplo siguiente para ejecutar un escenario de análisis de rendimiento rápido durante 1 minuto y crear los resultados en la carpeta /tmp/output:

    ```bash
    sudo python perfinsights.py -r quick -d 1M -a -o /tmp/output
    ```

    Puede usar el siguiente ejemplo para ejecutar el escenario de análisis de rendimiento durante 5 minutos y cargar el archivo tarball de resultado en la cuenta de almacenamiento:

    ```bash
    sudo python perfinsights.py -r vmslow -d 300S -a -t <StorageAccountName> -k <StorageAccountKey> -i <full resource Uri of the current VM>
    ```

    >[!Note]
    >Antes de ejecutar un escenario, PerfInsights pide al usuario que acepte compartir información de diagnóstico y que acepte los términos de licencia. Use la opción **-a o --accept-disclaimer-and-share-diagnostics** para omitir estas preguntas.
    >
    >Si tiene una incidencia de soporte técnico activa con Microsoft y ejecuta PerfInsights a petición del ingeniero de soporte técnico con el que trabaja, asegúrese de indicar el número de la incidencia mediante la opción **-s o --support-request**.

Una vez completada la ejecución, aparecerá un nuevo archivo tar en la misma carpeta que PerfInsights, a menos que no se especifique ninguna carpeta de salida. El nombre del archivo es **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.tar.gz**. Puede enviar este archivo al agente de soporte técnico para que lo analice o abrir el informe dentro del archivo para revisar las conclusiones y recomendaciones.

## <a name="review-the-diagnostics-report"></a>Revisión del informe de diagnóstico

En el archivo **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.tar.gz** puede encontrar un informe HTML en el que se detallan las conclusiones de PerfInsights. Para revisar el informe, expanda el archivo **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.tar.gz** y después abra el archivo **PerfInsights Report.html**.

### <a name="overview-tab"></a>Pestaña Información general

La pestaña **Overview** (Información general) proporciona detalles básicos sobre la ejecución e información sobre la máquina virtual. La pestaña **Findings** (Resultados) muestra un resumen de las recomendaciones de las distintas secciones del informe de PerfInsights.

![Captura de pantalla del informe de PerfInsights](media/how-to-use-perfinsights-linux/perfinsights-linux-overview.png)  
![Captura de pantalla del informe de PerfInsights](media/how-to-use-perfinsights-linux/perfinsights-linux-findings-tab.png)

> [!NOTE]
> Las conclusiones clasificadas como altas son problemas conocidos que pueden provocar problemas de rendimiento. Las conclusiones clasificadas como medias representan configuraciones no óptimas que no provocan necesariamente problemas de rendimiento. Las conclusiones clasificadas como bajas son solo declaraciones informativas.

Revise las recomendaciones y los vínculos de todas las conclusiones altas y medias. Aprenda cómo pueden afectar al rendimiento, así como los procedimientos recomendados para las configuraciones con optimización del rendimiento.

### <a name="cpu-tab"></a>Pestaña CPU

La pestaña **CPU** proporciona información sobre el consumo de CPU en todo el sistema durante la ejecución de PerfInsights. La información sobre los períodos de uso intensivo de la CPU y los principales consumidores de CPU de larga duración resultará útil para solucionar problemas de uso elevado de la CPU.

![Captura de pantalla de la pestaña CPU del informe de PerfInsights](media/how-to-use-perfinsights-linux/perfinsights-linux-cpu-tab.png)

### <a name="storage-tab"></a>Pestaña Almacenamiento

La sección **Conclusiones** muestra diversas conclusiones y recomendaciones relacionadas con el almacenamiento.

**Block Devices** (Dispositivos de bloqueo) y otras secciones relacionadas, como las pestañas **Partitions** (Particiones), **LVM** y **MDADM**, describen el modo en que están configurados los dispositivos de bloqueo y cómo se relacionan entre sí.

![Captura de pantalla de la pestaña Storage (Almacenamiento)](media/how-to-use-perfinsights-linux/perfinsights-linux-storage-tab.png)  
![Captura de pantalla de la pestaña MDADM](media/how-to-use-perfinsights-linux/perfinsights-linux-mdadm-config.png)

### <a name="linux-tab"></a>Pestaña Linux

La pestaña **Linux** contiene información sobre el hardware y el sistema operativo que se ejecuta en la máquina virtual. Entre los detalles se incluye una lista de procesos en ejecución e información sobre el agente invitado, PCI, CPU, controladores y controladores LIS.

![Captura de pantalla de la pestaña Linux](media/how-to-use-perfinsights-linux/perfinsights-linux-tab.png)

## <a name="next-steps"></a>Pasos siguientes

Puede cargar los registros e informes de diagnóstico en el Soporte técnico de Microsoft para que examinen con mayor profundidad. Cuando trabaja con el personal de Soporte técnico de Microsoft, puede que le soliciten enviar la salida generada por PerfInsights para ayudarle en el proceso de solución de problemas.

La siguiente captura de pantalla muestra un mensaje similar al que puede recibir:

![Captura de pantalla de mensaje de ejemplo del Soporte técnico de Microsoft](media/how-to-use-perfinsights-linux/support-email.png)

Siga las instrucciones del mensaje para acceder al área de trabajo de transferencia de archivos. Para mayor seguridad, tiene que cambiar la contraseña la primera vez que la use.

Después de iniciar sesión, encontrará un cuadro de diálogo para cargar el archivo **PerformanceDiagnostics\_aaaa-MM-dd\_hh-mm-ss-fff.tar.gz** que recopiló PerfInsights.
