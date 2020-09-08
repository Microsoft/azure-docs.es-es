---
title: Acerca del servicio Mobility para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con Azure Site Recovery | Microsoft Docs
description: Aprenda acerca del agente del servicio Mobility para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con el servicio Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: how-to
ms.date: 04/10/2020
ms.author: ramamill
ms.openlocfilehash: 19d6933497b469148687dc9c5012513ab7d0314b
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89047214"
---
# <a name="about-the-mobility-service-for-vmware-vms-and-physical-servers"></a>Acerca de Mobility Service para máquinas virtuales VMware y servidores físicos

Al configurar la recuperación ante desastres para máquinas virtuales de VMware y servidores físicos con [Azure Site Recovery](site-recovery-overview.md), instala el servicio Mobility de Site Recovery en cada máquina virtual de VMware y servidor físico local. Mobility Service captura las escrituras de datos en la máquina y las reenvía al servidor de procesos de Site Recovery. El servicio Mobility se instala mediante el software del agente correspondiente que puede implementar mediante los siguientes métodos:

- [Instalación de inserción](#push-installation): cuando se habilita la protección mediante Azure Portal, Site Recovery instala el servicio Mobility en el servidor.
- Instalación manual: puede instalar el servicio Mobility manualmente en cada máquina mediante la [interfaz de usuario](#install-the-mobility-service-using-ui) o del [símbolo del sistema](#install-the-mobility-service-using-command-prompt).
- [Implementación automatizada](vmware-azure-mobility-install-configuration-mgr.md): puede automatizar la instalación del servicio Mobility con herramientas de implementación de software como Configuration Manager.

> [!NOTE]
> El servicio Mobility usa aproximadamente entre un 6 y un 10 % de memoria en máquinas de origen para máquinas físicas o máquinas virtuales de VMware.

## <a name="antivirus-on-replicated-machines"></a>Antivirus en máquinas replicadas

Si las máquinas que desea replicar están ejecutando un software antivirus, excluya la carpeta de instalación del servicio Mobility, _C:\ProgramData\ASR\agent_, de las operaciones del antivirus. Esta exclusión garantiza que la replicación funcionará según lo esperado.

## <a name="push-installation"></a>Instalación de inserción

La instalación de inserción es una parte integral del trabajo que se ejecuta desde Azure Portal para [habilitar la replicación](vmware-azure-enable-replication.md#enable-replication). Después de elegir el conjunto de máquinas virtuales que desea proteger y de habilitar la replicación, el servidor de configuración inserta el agente del servicio Mobility en los servidores, lo instala y completa su registro con el servidor de configuración.

### <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de que se cumplen todos los [requisitos previos](vmware-azure-install-mobility-service.md).
- Asegúrese de que todas las configuraciones de servidor cumplen los criterios que se indican en [Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure](vmware-physical-azure-support-matrix.md).
- A partir de la [versión 9.36](https://support.microsoft.com/help/4578241/), en el caso de SUSE Linux Enterprise Server 11 SP4, asegúrese de que el instalador más reciente está [disponible en el servidor de configuración y que se realiza la escalabilidad horizontal del servidor de procesos](#download-latest-mobility-agent-installer-for-suse-11-sp3-server)

El flujo de trabajo de instalación de inserción se describe en las siguientes secciones:

### <a name="mobility-service-agent-version-923-and-higher"></a>Versión 9.23 y posteriores del agente del servicio Mobility

Para más información acerca de la versión 9.23, consulte el [paquete acumulativo de actualizaciones 35 para Azure Site Recovery](https://support.microsoft.com/help/4494485/update-rollup-35-for-azure-site-recovery).

Durante una instalación de inserción del servicio Mobility, se realizan los siguientes pasos:

1. El agente se inserta en la máquina de origen. Es posible que no se pueda copiar el agente en la máquina de origen debido a varios errores de entorno. Visite [nuestra guía](vmware-azure-troubleshoot-push-install.md) para solucionar los errores de instalación de inserción.
1. Tras copiarse correctamente el agente en el servidor, se realiza una comprobación de requisitos previos en este último.
   - Si se cumplen todos los requisitos previos, empezará la instalación.
   - Si uno o varios de los [requisitos previos](vmware-physical-azure-support-matrix.md) no se cumplen, se producirá un error en la instalación.
1. Como parte de la instalación del agente, se instala el proveedor del Servicio de instantáneas de volumen (VSS) para Azure Site Recovery. Este proveedor se usa para generar puntos de recuperación coherentes con la aplicación. Si se produce un error en la instalación de VSS, este paso se omitirá y la instalación del agente continuará.
1. Si la instalación del agente se realiza correctamente, pero se produce un error en la del proveedor de VSS, el estado del trabajo se marca como **Advertencia**. Esto no afecta a la generación de puntos de recuperación coherentes frente a bloqueos.

    - Para generar puntos coherentes con la aplicación, consulte [nuestra guía](vmware-physical-manage-mobility-service.md#install-site-recovery-vss-provider-on-source-machine) para completar una instalación manual del proveedor de VSS de Site Recovery.
    - Si no desea que se generen puntos de recuperación coherentes con la aplicación, [modifique la directiva de replicación](vmware-azure-set-up-replication.md#create-a-policy) para desactivarlos.

### <a name="mobility-service-agent-version-922-and-below"></a>Versión 9.22 y anteriores del agente del servicio Mobility

1. El agente se inserta en la máquina de origen. Es posible que no se pueda copiar el agente en la máquina de origen debido a varios errores de entorno. Consulte [nuestra guía](vmware-azure-troubleshoot-push-install.md) para solucionar los errores de instalación de inserción.
1. Tras copiarse correctamente el agente en el servidor, se realiza una comprobación de requisitos previos en este último.
   - Si se cumplen todos los requisitos previos, empezará la instalación.
   - Si uno o varios de los [requisitos previos](vmware-physical-azure-support-matrix.md) no se cumplen, se producirá un error en la instalación.

1. Como parte de la instalación del agente, se instala el proveedor del Servicio de instantáneas de volumen (VSS) para Azure Site Recovery. Este proveedor se usa para generar puntos de recuperación coherentes con la aplicación.
   - Si se produce un error en la instalación del proveedor de VSS, no se podrá realizar la instalación del agente. Para evitar errores en la instalación del agente, use la [versión 9.23](https://support.microsoft.com/help/4494485/update-rollup-35-for-azure-site-recovery) o versiones posteriores para generar puntos de recuperación coherentes frente a bloqueos e instalar el proveedor de VSS manualmente.

## <a name="install-the-mobility-service-using-ui"></a>Instalación del servicio Mobility mediante una interfaz de usuario

### <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de que todas las configuraciones de servidor cumplen los criterios que se indican en [Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure](vmware-physical-azure-support-matrix.md).
- [Busque el instalador](#locate-installer-files) correspondiente según el sistema operativo del servidor.

>[!IMPORTANT]
> No use el método de instalación con interfaz de usuario si va a replicar una máquina virtual de infraestructura como servicio (IaaS) de Azure de una región de Azure a otra. Use la instalación desde el [símbolo del sistema](#install-the-mobility-service-using-command-prompt).

1. Copie el archivo de instalación en el equipo y ejecútelo.
1. En **Opción de instalación**, seleccione **Instalar Mobility Service**.
1. Elija la ubicación de instalación y seleccione **Instalar**.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility1.png" alt-text="Página Opción de instalación del servicio Mobility":::.

1. Supervise la instalación en **Progreso de la instalación**. Una vez completada la instalación, seleccione **Continuar con la configuración** para registrar el servicio en el servidor de configuración.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility3.png" alt-text="Página de registro del servicio Mobility":::.

1. En **Configuration Server Details** (Detalles del servidor de configuración), especifique la dirección IP y la frase de contraseña que ha configurado.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility4.png" alt-text="Página de registro del servicio Mobility":::.

1. Seleccione **Registrar** para finalizar el registro.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility5.png" alt-text="Página de registro final del servicio Mobility":::.

## <a name="install-the-mobility-service-using-command-prompt"></a>Instalación del servicio Mobility mediante el símbolo del sistema

### <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de que todas las configuraciones de servidor cumplen los criterios que se indican en [Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure](vmware-physical-azure-support-matrix.md).
- [Busque el instalador](#locate-installer-files) correspondiente según el sistema operativo del servidor.

### <a name="windows-machine"></a>Un equipo Windows

- En un símbolo del sistema, ejecute los siguientes comandos para copiar el instalador en una carpeta local como, por ejemplo, _C:\Temp_, en el servidor que desea proteger. Reemplace el nombre de archivo del instalador por el nombre de archivo real.

  ```cmd
  cd C:\Temp
  ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
  MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted
  ```

- Ejecute este comando para instalar el agente.

  ```cmd
  UnifiedAgent.exe /Role "MS" /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
  ```

- Ejecute estos comandos para registrar el agente en el servidor de configuración.

  ```cmd
  cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
  UnifiedAgentConfigurator.exe  /CSEndPoint <CSIP> /PassphraseFilePath <PassphraseFilePath>
  ```

#### <a name="installation-settings"></a>Configuración de la instalación

Configuración | Detalles
--- | ---
Sintaxis | `UnifiedAgent.exe /Role \<MS/MT> /InstallLocation \<Install Location> /Platform "VmWare" /Silent`
Registros de configuración | `%ProgramData%\ASRSetupLogs\ASRUnifiedAgentInstaller.log`
`/Role` | Parámetro de instalación obligatorio. Especifica si se debe instalar Mobility Service (MS) o el destino maestro (MT).
`/InstallLocation`| Parámetro opcional. Especifique la ubicación (cualquier carpeta) de la instalación de Mobility Service.
`/Platform` | Mandatory. Especifica la plataforma en la que se instala el servicio Mobility: <br/> **VMware** para las máquinas virtuales y los servidores físicos de VMware. <br/> **Azure** para las máquinas virtuales de Azure.<br/><br/> Si está tratando máquinas virtuales de Azure como máquinas físicas, especifique **VMware**.
`/Silent`| Opcional. Especifica si se debe ejecutar el instalador en modo silencioso.

#### <a name="registration-settings"></a>Configuración de registro

Configuración | Detalles
--- | ---
Sintaxis | `UnifiedAgentConfigurator.exe  /CSEndPoint \<CSIP> /PassphraseFilePath \<PassphraseFilePath>`
Registros de configuración del agente | `%ProgramData%\ASRSetupLogs\ASRUnifiedAgentConfigurator.log`
`/CSEndPoint` | Parámetro obligatorio. `<CSIP>` especifica la dirección IP del servidor de configuración. Use una dirección IP válida.
`/PassphraseFilePath` |  Mandatory. Ubicación de la frase de contraseña. Use cualquier ruta de acceso de archivo local o UNC válida.

### <a name="linux-machine"></a>En un equipo Linux

1. Desde una sesión terminal, copie el instalador en una carpeta local (por ejemplo, _/tmp_) del servidor que desea proteger. Reemplace el nombre de archivo del instalador por el nombre de archivo real de la distribución de Linux y, a continuación, ejecute los comandos.

   ```shell
   cd /tmp ;
   tar -xvf Microsoft-ASR_UA_version_LinuxVersion_GA_date_release.tar.gz
   ```

2. Haga la instalación de la siguiente manera:

   ```shell
   sudo ./install -d <Install Location> -r MS -v VmWare -q
   ```

3. Una vez completada la instalación, el servicio Mobility debe registrarse en el servidor de configuración. Ejecute el siguiente comando para registrar el servicio Mobility en el servidor de configuración.

   ```shell
   /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <CSIP> -P /var/passphrase.txt
   ```

#### <a name="installation-settings"></a>Configuración de la instalación

Configuración | Detalles
--- | ---
Sintaxis | `./install -d \<Install Location> -r \<MS/MT> -v VmWare -q`
`-r` | Parámetro de instalación obligatorio. Especifica si se debe instalar Mobility Service (MS) o el destino maestro (MT).
`-d` | Parámetro opcional. Especifica la ubicación de la instalación del servicio Mobility: `/usr/local/ASR`.
`-v` | Mandatory. Especifica la plataforma en la que se instala el servicio Mobility. <br/> **VMware** para las máquinas virtuales y los servidores físicos de VMware. <br/> **Azure** para las máquinas virtuales de Azure.
`-q` | Opcional. Especifica si se debe ejecutar el instalador en modo silencioso.

#### <a name="registration-settings"></a>Configuración de registro

Configuración | Detalles
--- | ---
Sintaxis | `cd /usr/local/ASR/Vx/bin<br/><br/> UnifiedAgentConfigurator.sh -i \<CSIP> -P \<PassphraseFilePath>`
`-i` | Parámetro obligatorio. `<CSIP>` especifica la dirección IP del servidor de configuración. Use una dirección IP válida.
`-P` |  Mandatory. Ruta de acceso completa al archivo en que se guarda la frase de contraseña. Uso de cualquier carpeta válida.

## <a name="azure-virtual-machine-agent"></a>Agente de máquina virtual de Azure

- **Máquinas virtuales Windows**: desde la versión 9.7.0.0 de Mobility Service, el instalador de Mobility Service instala el [agente de máquina virtual de Azure](../virtual-machines/extensions/features-windows.md#azure-vm-agent). De este forma se garantiza que, cuando un equipo conmuta por error a Azure, la máquina virtual de Azure cumple el requisito previo de instalación del agente para usar cualquier extensión de máquina virtual.
- **Máquinas virtuales Linux**: [WALinuxAgent](../virtual-machines/extensions/update-linux-agent.md) debe instalarse manualmente en la máquina virtual Azure después de la conmutación por error.

## <a name="locate-installer-files"></a>Búsqueda de archivos del instalador

En el servidor de configuración vaya a la carpeta _%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository_. Compruebe qué instalador necesita en función del sistema operativo. En la siguiente tabla se resumen los archivos del instalador para cada máquina virtual VMware y el sistema operativo del servidor físico. Antes de empezar, puede revisar los [sistemas operativos admitidos](vmware-physical-azure-support-matrix.md#replicated-machines).

> [!NOTE]
> Los nombres de archivo utilizan la sintaxis que se muestra en la tabla siguiente con _version_ y _date_ como marcadores de posición de valores reales. Los nombres de archivo reales tendrán un aspecto similar al de estos ejemplos:
> - `Microsoft-ASR_UA_9.30.0.0_Windows_GA_22Oct2019_release.exe`
> - `Microsoft-ASR_UA_9.30.0.0_UBUNTU-16.04-64_GA_22Oct2019_release.tar.gz`

Archivo de instalador | Sistema operativo (solo de 64 bits)
--- | ---
`Microsoft-ASR_UA_version_Windows_GA_date_release.exe` | Windows Server 2016 </br> Windows Server 2012 R2 </br> Windows Server 2012 </br> Windows Server 2008 R2 SP1
`Microsoft-ASR_UA_version_RHEL6-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 6 </br> CentOS 6
`Microsoft-ASR_UA_version_RHEL7-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 7 </br> CentOS 7
`Microsoft-ASR_UA_version_SLES12-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 12 SP1 </br> Incluye SP2 y SP3.
[Para descargarlo y colocarlo en esta carpeta de forma manual](#download-latest-mobility-agent-installer-for-suse-11-sp3-server). | SUSE Linux Enterprise Server 11 SP3
`Microsoft-ASR_UA_version_SLES11-SP4-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 11 SP4
`Microsoft-ASR_UA_version_OL6-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 6.4 </br> Oracle Enterprise Linux 6.5
`Microsoft-ASR_UA_version_UBUNTU-14.04-64_GA_date_release.tar.gz` | Ubuntu Linux 14.04
`Microsoft-ASR_UA_version_UBUNTU-16.04-64_GA_date_release.tar.gz` | Servidor Ubuntu Linux 16.04 LTS
`Microsoft-ASR_UA_version_DEBIAN7-64_GA_date_release.tar.gz` | Debian 7
`Microsoft-ASR_UA_version_DEBIAN8-64_GA_date_release.tar.gz` | Debian 8

### <a name="download-latest-mobility-agent-installer-for-suse-11-sp3-server"></a>Descarga del instalador del agente de movilidad más reciente para el servidor de SUSE 11 SP3

Como **requisito previo para actualizar o proteger las máquinas con SUSE Linux Enterprise Server 11 SP3**  a partir de la versión [9.36](https://support.microsoft.com/help/4578241/):

1. Asegúrese de que el instalador del agente de movilidad más reciente se descarga desde el Centro de descarga de Microsoft y se coloca en el repositorio del instalador de inserciones del servidor de configuración y en todos los servidores de procesos de escalado horizontal
2. [Descargue](https://download.microsoft.com/download/0/3/4/0341b388-1ff5-4ead-b197-7cf6d2bb3e40/Microsoft-ASR_UA_9.36.0.0_SLES11-SP3-64_GA_06Aug2020_release.tar.gz) el instalador del agente de SUSE Linux Enterprise Server 11 SP3.
3. Vaya al servidor de configuración y copie el instalador del agente de SUSE Linux Enterprise Server 11 SP3 en las siguientes rutas de acceso:
    1. INSTALL_DIR\home\svsystems\pushinstallsvc\repository
    1.  INSTALL_DIR\home\svsystems\admin\web\sw folders
4. Ahora, vaya a los servidores de procesos de escalabilidad horizontal asociados y copie el instalador en las dos rutas de acceso mencionadas en el tercer paso.
5. **Por ejemplo**, si la ruta de acceso de instalación es C:\Archivos de programa (x86)\Microsoft Azure Site Recovery, los directorios mencionados anteriormente serán
    1. C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc\repository
    1. C:\Archivos de programa (x86)\Microsoft Azure Site Recovery\home\svsystems\admin\web\sw path

## <a name="next-steps"></a>Pasos siguientes

[Configurar la instalación de Mobility Service](vmware-azure-install-mobility-service.md).
