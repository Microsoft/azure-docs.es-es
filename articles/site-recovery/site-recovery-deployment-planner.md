---
title: Azure Site Recovery Deployment Planner para la recuperación ante desastres de VMware
description: Obtenga información sobre Azure Site Recovery Deployment Planner para la recuperación ante desastres de máquinas virtuales de VMware en Azure.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: mayg
ms.openlocfilehash: f930fbb9cad893363db2b1a6b9b6ea8acade5a54
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083793"
---
# <a name="about-the-azure-site-recovery-deployment-planner-for-vmware-to-azure"></a>Información sobre Azure Site Recovery Deployment Planner para VMware en Azure
Este artículo es la guía del usuario de Azure Site Recovery Deployment Planner para implementaciones de producción de VMware en Azure.

## <a name="overview"></a>Información general

Antes de empezar a proteger cualquier máquina virtual (VM) de VMware mediante Azure Site Recovery, asigne suficiente ancho de banda, según la frecuencia diaria de cambio de datos, para cumplir el objetivo de punto de recuperación (RPO) deseado. Asegúrese de implementar localmente el número correcto de servidores de configuración y de proceso.

También es preciso que cree el tipo y número correctos de cuentas de Azure Storage de destino. Cree cuentas de almacenamiento Estándar o Premium, para lo que debe tener en cuenta el crecimiento en los servidores de producción de origen debido al aumento del uso con el paso del tiempo. Elija el tipo de almacenamiento por máquina virtual, en función de las características de la carga de trabajo (por ejemplo, operaciones de E/S por segundo [IOPS] de lectura/escritura o actividad de datos) y de los límites de Site Recovery.

 Site Recovery Deployment Planner es una herramienta de línea de comandos para escenarios de recuperación ante desastres tanto de Hyper-V a Azure como de VMware a Azure. Con esta herramienta es posible generar de forma remota un perfil para las máquinas virtuales de VMware (sin ningún efecto en la producción) para conocer el ancho de banda y los requisitos de almacenamiento necesarios para realizar una correcta replicación y conmutación por error de prueba. La herramienta se puede ejecutar sin que sea preciso instalar localmente ningún componente de Site Recovery. Para obtener unos resultados de rendimiento adecuados, ejecute el organizador en una instancia de Windows Server que cumpla los requisitos mínimos del servidor de configuración de Site Recovery que debería implementarse como uno de los primeros pasos de la implementación en producción.

La herramienta proporciona los detalles siguientes:

**Evaluación de compatibilidad**

* Evaluación de la idoneidad de una máquina virtual, en función del número de discos, su tamaño, IOPS, la renovación, el tipo de arranque (EFI/BIOS) y la versión del sistema operativo

**Necesidad de ancho de banda de red frente a evaluación de RPO**

* El ancho de banda de red necesario para la replicación diferencial
* El rendimiento que Site Recovery puede obtener del paso de local a Azure
* El número de máquinas virtuales que se agrupan en lotes, en función del ancho de banda estimado para completar la replicación inicial en un período determinado de tiempo
* RPO que se puede conseguir para un ancho de banda determinado
* Impacto en el RPO deseado si se aprovisiona un ancho de banda inferior

**Requisitos de infraestructura de Azure**

* Tipo de almacenamiento (almacenamiento estándar o premium) de cada máquina virtual
* El número total de cuentas de almacenamiento estándar y premium que se van a configurar para la replicación (se incluyen las cuentas de almacenamiento en caché)
* Sugerencias de nomenclatura de las cuentas de almacenamiento, según la guía de Storage
* Número de núcleos de Azure que se deben configurar antes de realizar una conmutación por error, de prueba o real, en la suscripción
* Tamaño de máquina virtual de Azure que se recomienda para cada máquina virtual local

**Requisitos de la infraestructura local**
* Número necesario de servidores de configuración y de proceso que se van a implementar localmente

**Costo estimado de recuperación ante desastres en Azure**
* Costo total estimado de recuperación ante desastres en Azure: costo de licencia de Site Recovery, almacenamiento, red y proceso
* Detalles del análisis del costo por máquina virtual


>[!IMPORTANT]
>
>Dado que es probable que el uso aumente con el tiempo, al realizar todos los cálculos anteriores de la herramienta se asume un factor de crecimiento de las características de carga de trabajo de un 30 por ciento. Los cálculos también usan un valor del percentil 95 de todas las métricas de la generación de perfiles como IOPS de lectura y escritura y actividad de datos. Tanto el cálculo de percentil como el factor de crecimiento son configurables. Para más información acerca del factor de crecimiento, consulte la sección "Consideraciones acerca del factor de crecimiento". Para más información sobre el valor de percentil, consulte la sección "Valor del percentil usado para el cálculo".
>

## <a name="support-matrix"></a>Matrices compatibles

| **Categoría** | **VMware a Azure** |**Hyper-V en Azure**|**De Azure a Azure**|**De Hyper-V a un sitio secundario**|**Sitio VMware en un sitio secundario**
--|--|--|--|--|--
Escenarios admitidos |Sí|Sí|No|Sí*|No
Versión admitida | vCenter 6.7, 6.5, 6.0 o 5.5| Windows Server 2016, Windows Server 2012 R2 | N/D |Windows Server 2016, Windows Server 2012 R2|N/D
Configuración admitida|vCenter, ESXi| Clúster de Hyper-V, host de Hyper-V|N/D|Clúster de Hyper-V, host de Hyper-V|N/D|
Número de servidores cuyo perfil puede generarse por instancia en ejecución de Site Recovery Deployment Planner |Único (los perfiles de las máquinas virtuales que pertenecen a una instancia de vCenter Server o a un servidor ESXi se pueden generar a la vez)|Varios (los perfiles de las máquinas virtuales en varios hosts o clústeres de hosts se pueden generar a la vez)| N/D |Varios (los perfiles de las máquinas virtuales en varios hosts o clústeres de hosts se pueden generar a la vez)| N/D

*La herramienta es principalmente para un escenario de recuperación ante desastres de Hyper-V en Azure. En un escenario de recuperación ante desastres de Hyper-V en un sitio secundario, solo se puede usar para conocer recomendaciones sobre el origen como el ancho de banda de red requerido, el espacio de almacenamiento disponible necesario en cada uno de los servidores Hyper-V de origen y las cifras referentes al procesamiento por lotes de replicación inicial y a las definiciones de los lotes. Omita las recomendaciones de Azure y los costos del informe. Además, la operación Get Throughput (Obtención de rendimiento) no es aplicable al escenario de recuperación ante desastres de Hyper-V a un sitio secundario.

## <a name="prerequisites"></a>Prerrequisitos
La herramienta tiene dos fases principales: la generación de perfiles y la generación de informes. También hay una tercera opción para calcular solo el rendimiento. Los requisitos del servidor desde el que se inician la medición del rendimiento y la generación de perfiles se presentan en la tabla siguiente.

| Requisito del servidor | Descripción|
|---|---|
|Generación de perfiles y medición de rendimiento| <ul><li>Sistema operativo: Windows Server 2016 o Windows Server 2012 R2<br>(lo ideal es que coincida al menos con las [recomendaciones de tamaño del servidor de configuración](https://aka.ms/asr-v2a-on-prem-components)).</li><li>Configuración de la máquina: 8 vCPUs, 16 GB de RAM y disco duro de 300 GB</li><li>[.NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[VMware vSphere PowerCLI 6.0 R3](https://aka.ms/download_powercli)</li><li>[Microsoft Visual C++ Redistributable para Visual Studio 2012](https://aka.ms/vcplusplus-redistributable)</li><li>Acceso a Internet en Azure (*.blob.core.windows.net) desde este servidor, puerto 443<br>[Es opcional. Puede elegir especificar el ancho de banda disponible durante la generación de informes de forma manual.]</li><li>Cuenta de almacenamiento de Azure</li><li>Acceso de administrador en el servidor</li><li>Mínimo de 100 GB de espacio libre en disco (asumiendo 1.000 máquinas virtuales con un promedio de tres discos cada una, con perfil para 30 días)</li><li>La configuración del nivel de las estadísticas de VMware vCenter puede ser 1 o un nivel superior</li><li>Permitir el puerto de vCenter (443 de forma predeterminada): Site Recovery Deployment Planner utiliza este puerto para conectarse al servidor de vCenter o al host ESXi.</ul></ul>|
| Generación de informes | Un equipo con Windows o Windows Server con Excel 2013, o cualquier versión posterior.<li>[.NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[Microsoft Visual C++ Redistributable para Visual Studio 2012](https://aka.ms/vcplusplus-redistributable)</li><li>[VMware vSphere PowerCLI 6.0 R3](https://aka.ms/download_powercli) solo se necesita cuando pasa la opción -User en el comando de generación de informes para capturar la información de configuración de máquina virtual más reciente de las máquinas virtuales. Deployment Planner se conecta a vCenter Server. Permita el puerto vCenter (el valor predeterminado es 443) para conectarse a vCenter Server.</li>|
| Permisos de usuario | El permiso de solo lectura para la cuenta de usuario que se utiliza para acceder al servidor de VMware vCenter o host de VMware vSphere ESXi durante la generación de perfiles |

> [!NOTE]
>
>La herramienta puede generar perfiles solo de las máquinas virtuales con discos VMDK y RDM. No se pueden generar perfiles de máquinas virtuales con discos iSCSI o NFS. Site Recovery es compatible con discos NFS e iSCSI para los servidores de VMware. Como Deployment Planner no se encuentra dentro del invitado y solo genera perfiles mediante el uso de contadores de rendimiento de vCenter, la herramienta no tiene visibilidad sobre estos tipos de disco.
>

## <a name="download-and-extract-the-deployment-planner-tool"></a>Descarga y extracción de Deployment Planner Tool
1. Descargue la versión más reciente de [Site Recovery Deployment Planner](https://aka.ms/asr-deployment-planner).
La herramienta está empaquetada en una carpeta en formato zip. La versión actual de la herramienta admite únicamente el escenario de VMware a Azure.

2. Copie la carpeta .zip en la instancia de Windows Server desde la que desea ejecutar la herramienta.
La herramienta se puede ejecutar desde Windows Server 2012 R2 si el servidor tiene acceso a la red para conectarse al servidor de vCenter/host de vSphere ESXi que contiene las máquinas virtuales cuyo perfil se va a generar. Sin embargo, se recomienda ejecutar la herramienta en un servidor cuya configuración de hardware cumpla las [directrices de tamaño del servidor de configuración](https://aka.ms/asr-v2a-on-prem-components). Si ya ha implementado los componentes de Site Recovery de forma local, ejecute la herramienta desde el servidor de configuración.

    Se recomienda que tenga la misma configuración de hardware sea la misma que el servidor de configuración (que tiene un servidor de procesos incorporado) en el servidor en el que se ejecuta la herramienta. Dicha configuración garantiza que el rendimiento obtenido que indica la herramienta coincide con el rendimiento real que Site Recovery puede lograr durante la replicación. El cálculo del rendimiento depende del ancho de banda de red disponible en el servidor y de la configuración del hardware (como CPU y almacenamiento) del servidor. Si ejecuta la herramienta desde cualquier otro servidor, el rendimiento se calcula desde dicho servidor en Azure. Además, dado que la configuración del hardware del servidor puede diferir de la del servidor de configuración, el rendimiento obtenido del que informa la herramienta puede ser imprecisa.

3. Extraiga la carpeta .zip.
La carpeta contiene varios archivos y subcarpetas. El archivo ejecutable es ASRDeploymentPlanner.exe y está en la carpeta primaria.

    Ejemplo: Copie el archivo .zip en la unidad E:\ y extráigalo.
    E:\ASR Deployment Planner_v2.3.zip

    E:\ASR Deployment Planner_v2.3\ASRDeploymentPlanner.exe

### <a name="update-to-the-latest-version-of-deployment-planner"></a>Actualización a la versión más reciente de Deployment Planner

En el [historial de versiones](site-recovery-deployment-planner-history.md) se resumen las actualizaciones más recientes de Deployment Planner.

Si tiene una versión anterior de Deployment Planner, realice una de las siguientes acciones:
 * Si la versión más reciente no contiene una corrección de la generación de perfiles y la generación de perfiles ya está en curso en la versión actual del programador, continúe con la generación de perfiles.
 * Si la versión más reciente contiene una corrección de la generación de perfiles, se recomienda detener la versión actual de la generación de perfiles y reiniciar la generación de perfiles con la nueva versión.


 >[!NOTE]
 >
 >Al iniciar la generación de perfiles con la nueva versión, pase la misma ruta de acceso del directorio de salida, con el fin de que la herramienta anexe los datos del perfil a los archivos existentes. Para generar el informe, se usará un conjunto completo de datos de la generación de perfiles. Si pasa otro directorio de salida, se crean nuevos archivos y los datos anteriores de generación de perfiles no se usan al generar el informe.
 >
 >Cada nueva versión de Deployment Planner es una actualización acumulativa del archivo zip. No es preciso copiar los archivos más recientes en la carpeta anterior. Se puede crear y usar una carpeta nueva.


## <a name="version-history"></a>Historial de versiones
La versión más reciente de la herramienta Site Recovery Deployment Planner es la 2.5.
Para saber qué correcciones se agregan en cada actualización, consulte [ASR Deployment Planner Version History](./site-recovery-deployment-planner-history.md) (Historial de versiones de ASR Deployment Planner).

## <a name="next-steps"></a>Pasos siguientes
[Ejecución de Site Recovery Deployment Planner](site-recovery-vmware-deployment-planner-run.md)
