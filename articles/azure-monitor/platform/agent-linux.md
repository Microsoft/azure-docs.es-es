---
title: Instalación del agente de Log Analytics en equipos Linux
description: En este artículo, se explica cómo puede conectar equipos Linux hospedados en otras nubes o en el entorno local con Azure Monitor utilizando el agente de Log Analytics para Linux.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/21/2020
ms.openlocfilehash: 997064ad030d22531277f1c412add6916eb7733f
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230473"
---
# <a name="install-log-analytics-agent-on-linux-computers"></a>Instalación del agente de Log Analytics en equipos Linux
En este artículo, se proporcionan detalles sobre la instalación del agente de Log Analytics en equipos Linux con los métodos siguientes:

* [Instale al agente para Linux con un script contenedor](#install-the-agent-using-wrapper-script) hospedado en GitHub. Este es el método recomendado para instalar y actualizar el agente cuando el equipo tiene conectividad a Internet directamente o a través de un servidor proxy.
* [Descargue e instale manualmente](#install-the-agent-manually) el agente. Esto es necesario cuando el equipo Linux no tiene acceso a Internet y se va a comunicar con Azure Monitor o Azure Automation utilizando la [puerta de enlace de Log Analytics](gateway.md). 

>[!IMPORTANT]
> Los métodos de instalación descritos en este artículo se usan normalmente con máquinas virtuales locales o en otras nubes. Consulte [Opciones de instalación](log-analytics-agent.md#installation-options) para ver las opciones más eficaces que se pueden utilizar con Azure Virtual Machines.



## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Consulte [Información general de los agentes de Azure Monitor](agents-overview.md#supported-operating-systems) para obtener una lista de las distribuciones de Linux admitidas por el agente de Log Analytics.

>[!NOTE]
>OpenSSL 1.1.0 solo se admite en las plataformas x86_x64 (64 bits) y OpenSSL anterior a 1.x no se admite en ninguna plataforma.
>
A partir de las versiones publicadas después de agosto de 2018, hemos realizado los siguientes cambios en nuestro modelo de soporte técnico:  

* Solo se admiten las versiones de servidor, no de cliente.  
* La compatibilidad se centra en cualquiera de las [distribuciones de Linux aprobadas en Azure](../../virtual-machines/linux/endorsed-distros.md). Tenga en cuenta que puede haber cierto retraso entre la aprobación de una distribución o versión de Linux en Azure y su compatibilidad con el agente de Linux de Log Analytics.
* Se admiten todas las versiones secundarias de cada versión principal de la lista.
* No se admiten las versiones que han superado la fecha de finalización de soporte técnico de su fabricante.  
* No se admiten las nuevas versiones de AMI.  
* Solo se admiten las versiones que ejecutan SSL 1.x de forma predeterminada.

>[!NOTE]
>Si utiliza una distribución o versión que no se admite actualmente y no se alinea con nuestro modelo de soporte técnico, se recomienda bifurcar este repositorio, confirmar que el soporte técnico de Microsoft no prestará asistencia con versiones de agente con bifurcación.

### <a name="python-2-requirement"></a>Requisito de Python 2

 El agente de Log Analytics requiere Python 2. Si la máquina virtual usa un distribución que no incluye Python 2 de forma predeterminada, debe instalarlo. Los siguientes comandos de ejemplo instalarán Python 2 en diferentes distribuciones.

 - Red Hat, CentOS, Oracle: `yum install -y python2`
 - Ubuntu, Debian: `apt-get install -y python2`
 - SUSE: `zypper install -y python2`

El ejecutable python2 debe tener un alias para *python*. A continuación se ofrece un método que puede usar para establecer este alias:

1. Ejecute el siguiente comando para quitar los alias existentes.
 
    ```
    sudo update-alternatives --remove-all python
    ```

2. Para crear el alias, ejecute el siguiente comando.

    ```
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    ```

## <a name="supported-linux-hardening"></a>Protección de Linux admitida
El agente de OMS tiene compatibilidad de personalización limitada para Linux. 

Actualmente se admiten los siguientes: 
- FIPs

Los siguientes están planeados, pero no se admiten actualmente:
- CIS
- SELINUX

Otros métodos de protección y personalización no se admiten ni están planeados para el agente de OMS.  

## <a name="agent-prerequisites"></a>Requisitos del agente

En la tabla siguiente se indican los paquetes necesarios para las [distribuciones de Linux compatibles](#supported-operating-systems) en las que se instalará el agente.

|Paquete necesario |Descripción |Versión mínima |
|-----------------|------------|----------------|
|Glibc |    Biblioteca GNU C | 2.5-12 
|Openssl    | Bibliotecas OpenSSL | 1.0.x o 1.1.x |
|Curl | Cliente web de cURL | 7.15.5 |
|Python | | 2.6+ o 3.3+
|Python ctypes | | 
|PAM | Módulos de autenticación conectables | | 

>[!NOTE]
>Rsyslog o Syslog son necesarios para recopilar mensajes de Syslog. El demonio predeterminado de Syslog en la versión 5 de Red Hat Enterprise Linux, CentOS y Oracle Linux (Sysklog) no se admite para la recopilación de eventos de Syslog. Para recopilar datos de Syslog de esta versión de estas distribuciones, es necesario instalar configurar el demonio de Rsyslog para reemplazar Sysklog.

## <a name="network-requirements"></a>Requisitos de red
Consulte [Introducción al agente de Log Analytics](log-analytics-agent.md#network-requirements) para conocer los requisitos de red para el agente de Linux.

## <a name="agent-install-package"></a>Paquete de instalación del agente

El agente de Log Analytics para Linux se compone de varios paquetes. El archivo de lanzamiento contiene los siguientes paquetes, que estarán disponibles si el conjunto de paquetes de shell se ejecuta con el parámetro `--extract`:

**Package** | **Versión** | **Descripción**
----------- | ----------- | --------------
omsagent | 1.12.15 | Agente de Log Analytics para Linux
omsconfig | 1.1.1 | Agente para configurar el agente de Log Analytics
omi | 1.6.3 | Infraestructura de administración abierta (OMI), servidor de CIM ligero *Tenga en cuenta que OMI debe tener acceso raíz para poder ejecutar los trabajos de Cron que son necesarios para que el servicio funcione.*
scx | 1.6.3 | Proveedores de CIM OMI para métricas de rendimiento del sistema operativo
apache-cimprov | 1.0.1 | Proveedor de supervisión de rendimiento de servidor HTTP de Apache para OMI. Solo se instala si se detecta un servidor HTTP de Apache.
mysql-cimprov | 1.0.1 | Proveedor de supervisión de rendimiento de servidor MySQL Server para OMI. Solo se instala si se detecta un servidor MySQL/MariaDB de Apache.
docker-cimprov | 1.0.0 | Proveedor de Docker para OMI. Solo se instala si se detecta Docker.

### <a name="agent-installation-details"></a>Detalles de instalación del agente

Una vez instalado el agente de Log Analytics para paquetes Linux, se aplicarán los siguientes cambios de configuración adicionales en todo el sistema. Estos artefactos se quitan cuando se desinstala el paquete omsagent.

* Se crea un usuario sin privilegios llamado: `omsagent` . El demonio se ejecuta con esta credencial. 
* Se crea un archivo *include* de sudoers en `/etc/sudoers.d/omsagent`. Este archivo permite a `omsagent` reiniciar los demonios syslog y omsagent. Si la versión de sudo instalada no es compatible con las directivas *include* de sudo, estas entradas se escribirán en `/etc/sudoers`.
* Se modifica la configuración de Syslog para reenviar un subconjunto de eventos al agente. Para más información, consulte este artículo sobre la [configuración de colecciones de datos de Syslog](data-sources-syslog.md).

En los equipos supervisados con Linux, el agente aparece como `omsagent`. `omsconfig` es el agente de configuración del agente de Log Analytics para Linux que busca la nueva configuración del portal cada cinco minutos. Esta nueva configuración actualizada se aplica a los archivos de configuración del agente que se encuentran en `/etc/opt/microsoft/omsagent/conf/omsagent.conf`.

## <a name="install-the-agent-using-wrapper-script"></a>Instalación del agente con un script contenedor

Los pasos siguientes permiten configurar el agente de Log Analytics en Azure y la nube de Azure Government utilizando el script contenedor en aquellos equipos Linux que puedan conectarse directamente o a través de un servidor proxy para descargar el agente hospedado en GitHub e instalar el agente.  

Si el equipo Linux necesita un servidor proxy para comunicarse con Log Analytics, esta configuración puede especificarse en la línea de comandos incluyendo `-p [protocol://][user:password@]proxyhost[:port]`. La propiedad *protocol* acepta `http` o `https`, y la propiedad *proxyhost* acepta el nombre de dominio completo o la dirección IP del servidor proxy. 

Por ejemplo: `https://proxy01.contoso.com:30443`

Si se requiere autenticación en alguno de los dos casos, deberá especificar el nombre de usuario y la contraseña. Por ejemplo: `https://user01:password@proxy01.contoso.com:30443`

1. Si desea configurar el equipo Linux para que se conecte a un área de trabajo de Log Analytics, ejecute el comando siguiente con el identificador del área de trabajo y la clave principal. El siguiente comando descarga el agente, valida su suma de comprobación y lo instala.
    
    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

    El siguiente comando incluye el parámetro de proxy `-p` y la sintaxis de ejemplo necesaria cuando el servidor proxy requiere autenticación:

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://]<proxy user>:<proxy password>@<proxyhost>[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY>
    ```

2. Para configurar el equipo Linux de modo que se conecte con un área de trabajo de Log Analytics en la nube de Azure Government, ejecute el comando siguiente con el identificador del área de trabajo y la clave principal que ha copiado anteriormente. El siguiente comando descarga el agente, valida su suma de comprobación y lo instala. 

    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ``` 

    El siguiente comando incluye el parámetro de proxy `-p` y la sintaxis de ejemplo necesaria cuando el servidor proxy requiere autenticación:

   ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -p [protocol://]<proxy user>:<proxy password>@<proxyhost>[:port] -w <YOUR WORKSPACE ID> -s <YOUR WORKSPACE PRIMARY KEY> -d opinsights.azure.us
    ```
2. Ejecute el comando siguiente para reiniciar el agente: 

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ``` 


## <a name="install-the-agent-manually"></a>Instalación manual del agente

El agente de Log Analytics para Linux viene en un conjunto de scripts de shell autoextraíbles e instalables. Este paquete contiene los paquetes Debian y RPM para cada uno de los componentes del agente y se pueden instalar directamente o extraerse para recuperar los paquetes individuales. Hay dos paquetes, uno para arquitecturas x64 y otro para arquitecturas x86. 

> [!NOTE]
> En el caso de las máquinas virtuales de Azure, le recomendamos que instale el agente utilizando la [extensión de máquina virtual de Log Analytics](../../virtual-machines/extensions/oms-linux.md) para Linux. 

1. [Descargue](https://github.com/microsoft/OMS-Agent-for-Linux#azure-install-guide) y transfiera el conjunto adecuado (x64 o x86) a la máquina virtual o al equipo físico Linux mediante scp o sftp.

2. Instale el paquete utilizando el argumento `--install`. Para incluir un área de trabajo de Log Analytics durante la instalación, proporcione los parámetros `-w <WorkspaceID>` y `-s <workspaceKey>` que copió anteriormente.

    >[!NOTE]
    >Si los paquetes dependientes (por ejemplo, omi, scx, omsconfig o sus versiones anteriores) están instalados, como ocurriría en el caso de que el agente de System Center Operations Manager para Linux estuviera instalado, es necesario utilizar el argumento `--upgrade`. 

    ```
    sudo sh ./omsagent-*.universal.x64.sh --install -w <workspace id> -s <shared key>
    ```

3. Si desea configurar el agente de Linux para instalar un área de trabajo de Log Analytics y conectarse a ella utilizando una puerta de enlace de Log Analytics, ejecute el siguiente comando con el proxy, el identificador del área de trabajo y los parámetros de clave del área de trabajo. Esta configuración se puede especificar en la línea de comandos incluyendo `-p [protocol://][user:password@]proxyhost[:port]`. La propiedad *proxyhost* acepta un nombre de dominio completo o una dirección IP del servidor de puerta de enlace de Log Analytics.  

    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -p https://<proxy address>:<proxy port> -w <workspace id> -s <shared key>
    ```

    Si se requiere autenticación, deberá especificar el nombre de usuario y la contraseña. Por ejemplo: 
    
    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -p https://<proxy user>:<proxy password>@<proxy address>:<proxy port> -w <workspace id> -s <shared key>
    ```

4. Si desea configurar el equipo Linux para que se conecte a un área de trabajo de Log Analytics situada en la nube de Azure Government, ejecute el comando siguiente con el identificador del área de trabajo y la clave principal que copió anteriormente.

    ```
    sudo sh ./omsagent-*.universal.x64.sh --upgrade -w <workspace id> -s <shared key> -d opinsights.azure.us
    ```

Si desea instalar los paquetes del agente y configurarlos para que posteriormente envíen información a un área de trabajo de Log Analytics, ejecute el siguiente comando:

```
sudo sh ./omsagent-*.universal.x64.sh --upgrade
```

Para extraer los paquetes de agente del conjunto de paquetes sin instalarlos, ejecute el siguiente comando:

```
sudo sh ./omsagent-*.universal.x64.sh --extract
```

## <a name="upgrade-from-a-previous-release"></a>Actualización desde una versión anterior

A partir de la versión 1.0.0-47, todos los lanzamientos permiten actualizar desde la versión anterior. Realice la instalación con el parámetro `--upgrade` para actualizar todos los componentes del agente a la última versión.

## <a name="cache-information"></a>Información de caché
Los datos del agente de Log Analytics para Linux se almacenan en la caché del equipo local en *% STATE_DIR_WS%/out_oms_common*. buffer * antes de enviarse a Azure Monitor. Los datos de registro personalizados se almacenan en búfer en *%STATE_DIR_WS%/out_oms_blob*.buffer*. La ruta de acceso puede ser diferente para algunas [soluciones y tipos de datos](https://github.com/microsoft/OMS-Agent-for-Linux/search?utf8=%E2%9C%93&q=+buffer_path&type=).

El agente intenta realizar la carga cada 20 segundos. Si no lo consigue, el tiempo de espera aumentará de forma exponencial hasta que la operación se ejecute correctamente. Esperará 30 segundos antes del segundo intento; 60 segundos antes del siguiente; 120 segundos a continuación y así sucesivamente hasta un máximo de 9 minutos entre reintentos hasta que se vuelva a conectar correctamente. El agente solo volverá a intentar la operación 10 veces para un fragmento determinado de datos antes de descartarlo y pasar al siguiente. Esto continuará hasta que el agente pueda volver a efectuar la carga correctamente. Por esta razón, los datos se pueden almacenar en búfer hasta 8,5 horas antes de ser descartados.

El tamaño predeterminado de la caché es 10 MB, pero se puede modificar en el archivo [omsagent.conf](https://github.com/microsoft/OMS-Agent-for-Linux/blob/e2239a0714ae5ab5feddcc48aa7a4c4f971417d4/installer/conf/omsagent.conf).


## <a name="next-steps"></a>Pasos siguientes

- Revise [Administrar y mantener el agente de Log Analytics para Windows y Linux](agent-manage.md) para obtener información sobre cómo volver a configurar, actualizar o quitar el agente de la máquina virtual.

- Consulte [Troubleshooting the Linux agent](agent-linux-troubleshoot.md) (Solución de problemas del agente Linux) si encuentra problemas durante la instalación del agente o al administrarlo.
