---
title: Conexión de equipos Linux a Azure Monitor | Microsoft Docs
description: En este artículo, se explica cómo puede conectar equipos Linux hospedados en otras nubes o en el entorno local con Azure Monitor utilizando el agente de Log Analytics para Linux.
ms.subservice: logs
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 01/21/2020
ms.openlocfilehash: 965d5dd558d0da7a758db77330c9129ea0e8247c
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543867"
---
# <a name="connect-linux-computers-to-azure-monitor"></a>Conexión de equipos Linux a Azure Monitor

Para poder supervisar y administrar máquinas virtuales o equipos físicos del centro de datos local o de otro entorno en la nube con Azure Monitor, debe implementar el agente de Log Analytics y configurarlo para que envíe información a un área de trabajo de Log Analytics. Asimismo, el agente admite el rol de Hybrid Runbook Worker de Azure Automation.

Si desea instalar el agente de Log Analytics para Linux, siga uno de los métodos siguientes. Más adelante en el artículo se proporcionan detalles sobre el uso de cada método.

* [Descargue e instale manualmente](#install-the-agent-manually) el agente. Esto es necesario cuando el equipo Linux no tiene acceso a Internet y se va a comunicar con Azure Monitor o Azure Automation utilizando la [puerta de enlace de Log Analytics](gateway.md). 
* [Instale al agente para Linux con un script contenedor](#install-the-agent-using-wrapper-script) hospedado en GitHub. Este es el método recomendado para instalar y actualizar el agente cuando el equipo tiene conectividad a Internet directamente o a través de un servidor proxy.

Para comprender la configuración compatible, revise los [sistemas operativos Linux admitidos](log-analytics-agent.md#supported-linux-operating-systems) y la [configuración del firewall de red](log-analytics-agent.md#network-requirements).

>[!NOTE]
>No se puede configurar el agente de Log Analytics para Linux para informar a varias áreas de trabajo de Log Analytics. Solo se puede configurar para enviar informes simultánea o individualmente a un grupo de administración de System Center Operations Manager y a un área de trabajo de Log Analytics.

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

## <a name="obtain-workspace-id-and-key"></a>Obtención de la clave y el identificador de área de trabajo

Antes de instalar el agente Log Analytics para Linux, necesita la clave y el identificador de área de trabajo para el área de trabajo de Log Analytics. Esta información es necesaria para configurar el agente de la forma adecuada y asegurarse de que puede comunicarse correctamente con Azure Monitor.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]  

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**. En el cuadro de búsqueda, escriba **Log Analytics**. La lista se filtra en función de lo que escriba. Seleccione **Áreas de trabajo de Log Analytics**.

2. En la lista de áreas de trabajo de Log Analytics, seleccione la que creó anteriormente. (Puede que le haya asignado el nombre **DefaultLAWorkspace**).

3. Seleccione **Configuración avanzada**.

    ![Menú Opciones avanzadas de Log Analytics en Azure Portal](../learn/media/quick-collect-azurevm/log-analytics-advanced-settings-azure-portal.png) 
 
4. Seleccione **Orígenes conectados** y **Servidores Linux**.

5. Encontrará los valores a la derecha de **Id. del área de trabajo** y **Clave principal**. Copie y pegue ambos valores en el editor que prefiera.

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

## <a name="supported-linux-hardening"></a>Protección de Linux admitida
El agente de OMS tiene compatibilidad de personalización limitada para Linux. 

Actualmente se admiten los siguientes: 
- FIPs

Los siguientes están planeados, pero no se admiten actualmente:
- CIS
- SELINUX

Otros métodos de protección y personalización no se admiten ni están planeados para el agente de OMS.  


## <a name="upgrade-from-a-previous-release"></a>Actualización desde una versión anterior

A partir de la versión 1.0.0-47, todos los lanzamientos permiten actualizar desde la versión anterior. Realice la instalación con el parámetro `--upgrade` para actualizar todos los componentes del agente a la última versión.

## <a name="next-steps"></a>Pasos siguientes

- Revise [Administrar y mantener el agente de Log Analytics para Windows y Linux](agent-manage.md) para obtener información sobre cómo volver a configurar, actualizar o quitar el agente de la máquina virtual.

- Consulte [Troubleshooting the Linux agent](agent-linux-troubleshoot.md) (Solución de problemas del agente Linux) si encuentra problemas durante la instalación del agente o al administrarlo.
