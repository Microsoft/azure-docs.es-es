---
title: Solución de supervisión de contenedores en Azure Monitor | Microsoft Docs
description: La solución de supervisión de contenedores de Azure Monitor le ayuda a ver y administrar los hosts de contenedores de Docker y Windows en una sola ubicación.
ms.subservice: logs
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 07/06/2020
ms.openlocfilehash: b681e3fa4963a8fe899ccbad8dbf1bbdfbe452ce
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326909"
---
# <a name="container-monitoring-solution-in-azure-monitor"></a>Solución de supervisión de contenedores en Azure Monitor

![Símbolo de Containers](./media/containers/containers-symbol.png)

En este artículo se describe cómo configurar y usar la solución de supervisión de contenedores de Azure Monitor, que le permite ver y administrar los hosts de contenedores de Docker y Windows en una sola ubicación. Docker es un sistema de virtualización de software que usa para crear contenedores que automatizan la implementación de software en su infraestructura de TI.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

La solución muestra qué contenedores están en ejecución, qué imagen de contenedor están ejecutando y dónde se ejecutan los contenedores. Puede ver información de auditoría detallada que muestra los comandos que se usan con los contenedores. Y, para solucionar los problemas de los contenedores, puede ver y buscar registros centralizados sin tener que ver los hosts de Docker o Windows de forma remota. Puede encontrar los contenedores que están causando ruido o realizando un consumo excesivo de recursos en un host. También puede ver la información centralizada acerca de la CPU, la memoria, el almacenamiento y el uso y el rendimiento de la red en relación con los contenedores. En equipos con Windows, puede centralizar y comparar registros de Windows Server, Hyper-V y contenedores de Docker. La solución es compatible con los siguientes orquestadores de contenedores:

- Docker Swarm
- DC/OS
- Kubernetes
- Service Fabric
- Red Hat OpenShift

Si tiene contenedores implementados en [Azure Service Fabric](../../service-fabric/service-fabric-overview.md), le recomendamos habilitar la [solución de Service Fabric](../../service-fabric/service-fabric-diagnostics-oms-setup.md) y esta solución para incluir la supervisión de eventos del clúster. Antes de habilitar la solución de Service Fabric, consulte [Uso de la solución de Service Fabric](../../service-fabric/service-fabric-diagnostics-event-analysis-oms.md) para comprender qué proporciona y cómo se usa.

Si está interesado en supervisar el rendimiento de las cargas de trabajo implementadas en entornos de Kubernetes hospedado en Azure Kubernetes Service (AKS), consulte [Monitor Azure Kubernetes Service](./container-insights-overview.md) (Supervisión de Azure Kubernetes Service). La solución de supervisión de contenedores no es compatible con la supervisión de esa plataforma.  

En el siguiente diagrama se muestran las relaciones entre varios hosts y agentes de contenedor con Azure Monitor.

![Diagrama de contenedores](./media/containers/containers-diagram.png)

## <a name="system-requirements-and-supported-platforms"></a>Requisitos del sistema y plataformas compatibles

Antes de comenzar, revise los detalles siguientes para comprobar que cumple los requisitos previos.

### <a name="container-monitoring-solution-support-for-docker-orchestrator-and-os-platform"></a>Compatibilidad con solución de supervisión de contenedores para el orquestador y la plataforma de sistema operativo de Docker

En este artículo, los espacios de direcciones de red virtual de radio son 10.0.2.0/24 y 10.0.3.0/24.En la tabla siguiente se hace un resumen de la compatibilidad de supervisión del sistema operativo y la orquestación de Docker con el inventario, el rendimiento y el registro de contenedores con Azure Monitor.   

|Orquestación de Docker | ACS | Linux | Windows | Contenedor<br>Inventario | Imagen<br>Inventario | Nodo<br>Inventario | Contenedor<br>Rendimiento | Contenedor<br>Evento | Evento<br>Log | Contenedor<br>Log |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Kubernetes | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; |
| Mesosphere<br>DC/OS | &#8226; | &#8226; | | &#8226; | &#8226; | &#8226; | &#8226;| &#8226; | &#8226; | &#8226; |
| Docker<br>Swarm | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |
| Servicio<br>Fabric | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; |
| Red Hat Open<br>Shift | | &#8226; | | &#8226; | &#8226;| &#8226; | &#8226; | &#8226; | | &#8226; |
| Windows Server<br>(independiente) | | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |
| Linux Server<br>(independiente) | | &#8226; | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |

### <a name="docker-versions-supported-on-linux"></a>Versiones de docker admitidas en Linux

- Docker de 1.11 a 1.13
- Docker CE y EE v17.06

### <a name="x64-linux-distributions-supported-as-container-hosts"></a>Las distribuciones x64 de Linux se admiten como hosts de contenedor

- Ubuntu 14.04 LTS y 16.04 LTS
- CoreOS(stable)
- Amazon Linux 2016.09.0
- openSUSE 13.2
- openSUSE LEAP 42.2
- CentOS 7.2 y 7.3
- SLES 12
- RHEL 7.2 y 7.3
- Red Hat OpenShift Container Platform (OCP) 3.4 y 3.5
- ACS Mesosphere DC/OS 1.7.3 a 1.8.8
- ACS Kubernetes 1.4.5 a 1.6
    - Los eventos de Kubernetes, el inventario de Kubernetes y los procesos de contenedor solo son compatibles con la versión 1.4.1-45 y posteriores del agente de Log Analytics para Linux.
- ACS Docker Swarm

[!INCLUDE [log-analytics-agent-note.md](../../../includes/log-analytics-agent-note.md)] 

### <a name="supported-windows-operating-system"></a>Sistemas operativos Windows compatibles

- Windows Server 2016
- Windows 10 Anniversary Edition (Professional o Enterprise)

### <a name="docker-versions-supported-on-windows"></a>Versiones de docker admitidas en Windows

- Docker de 1.12 y 1.13
- Docker 17.03.0 y versiones posteriores

## <a name="installing-and-configuring-the-solution"></a>Instalación y configuración de la solución

Utilice la siguiente información para instalar y configurar la solución.

1. Agregue la solución de supervisión de contenedores al área de trabajo de Log Analytics desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.ContainersOMS?tab=Overview) o mediante el proceso descrito en el artículo sobre [adición de soluciones desde la Galería de soluciones](./solutions.md).

2. Instale y use Docker con un agente de Log Analytics. En función de su sistema operativo y del orquestador de Docker, puede usar los métodos siguientes para configurar el agente.
   - Para hosts independientes:
     - En sistemas operativos compatibles con Linux, instale y ejecute Docker y, después, instale y configure el [Agente de Log Analytics para Linux](../learn/quick-collect-linux-computer.md).  
     - En CoreOS, no se puede ejecutar el agente de Log Analytics para Linux. En su lugar, ejecute una versión en contenedores del agente de Log Analytics para Linux. Examine los hosts de contenedores de Linux, incluido CoreOS, o los hosts de contenedores de Azure Government Linux, incluido CoreOS, si va a trabajar con contenedores en la nube de Azure Government.
     - En Windows Server 2016 y Windows 10, instale el cliente de Docker y Docker Engine y luego conecte un agente para recopilar información y enviarla a Azure Monitor. Consulte [Instalación y configuración de hosts de contenedor de Windows](#install-and-configure-windows-container-hosts) si tiene un entorno Windows.
   - Para la orquestación de múltiples hosts de Docker:
     - Si tiene un entorno Red Hat OpenShift, vea Configuración de un agente de Log Analytics para Red Hat OpenShift.
     - Si tiene un clúster de Kubernetes con Azure Container Service:
       - Vea [Configurar un agente de Linux de Log Analytics para Kubernetes](#configure-a-log-analytics-linux-agent-for-kubernetes).
       - Vea [Configurar un agente de Windows de Log Analytics para Kubernetes](#configure-a-log-analytics-windows-agent-for-kubernetes).
       - Vea Uso de Helm para implementar un agente de Log Analytics en Linux Kubernetes.
     - Si tiene un clúster DC/OS de Azure Container Service, obtenga más información en el artículo sobre [supervisión de un clúster de DC/OS de Azure Container Service con Azure Monitor](/previous-versions/azure/container-service/dcos-swarm/container-service-monitoring-oms).
     - Si tiene un entorno en modo Docker Swarm, obtenga más información en Configuración de un agente de Log Analytics para Docker Swarm.
     - Si tiene un clúster de Service Fabric, obtenga más información en el artículo sobre [supervisión de contenedores con Azure Monitor](../../service-fabric/service-fabric-diagnostics-oms-containers.md).

Revise el artículo [Docker Engine en Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon) para obtener información adicional sobre cómo instalar y configurar Docker Engine en equipos con Windows.

> [!IMPORTANT]
> Docker tiene que ejecutarse **antes de** instalar el [agente de Log Analytics para Linux](../learn/quick-collect-linux-computer.md) en los hosts de contenedores. Si ya ha instalado el agente antes de instalar Docker, necesita reinstalar el agente de Log Analytics para Linux. Para más información acerca de Docker, consulte el [sitio web de Docker](https://www.docker.com).

### <a name="install-and-configure-linux-container-hosts"></a>Instalación y configuración de hosts de contenedor de Linux

Después de instalar Docker, use la siguientes opciones para el host de contenedores para configurar el agente que se usará con Docker. En primer lugar necesita el identificador y la clave del área de trabajo de Log Analytics, que puede encontrar en Azure Portal. En el área de trabajo, haga clic en **Inicio rápido** > **Equipos** para ver el **Id. de área de trabajo** y la **Clave principal**.  Copie y pegue ambos valores en el editor que prefiera.

**Para todos los hosts de contenedores de Linux excepto CoreOS:**

- Para obtener más información y conocer los pasos para instalar el agente de Log Analytics para Linux, consulte [Log Analytics agent overview](../platform/log-analytics-agent.md) (Información general sobre el agente de Log Analytics).

**Para todos los hosts de contenedores de Linux incluido CoreOS:**

Inicie el contenedor que quiere supervisar. Modifique y use el ejemplo siguiente:

```
sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/containers:/var/lib/docker/containers -e WSID="your workspace id" -e KEY="your key" -h=`hostname` -p 127.0.0.1:25225:25225 --name="omsagent" --restart=always microsoft/oms
```

**Para todos los hosts de contenedores de Linux para Azure Government incluido CoreOS:**

Inicie el contenedor que quiere supervisar. Modifique y use el ejemplo siguiente:

```
sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/log:/var/log -v /var/lib/docker/containers:/var/lib/docker/containers -e WSID="your workspace id" -e KEY="your key" -e DOMAIN="opinsights.azure.us" -p 127.0.0.1:25225:25225 -p 127.0.0.1:25224:25224/udp --name="omsagent" -h=`hostname` --restart=always microsoft/oms
```

**Cambio de un agente de Linux instalado a un agente en un contenedor**

Si antes usaba el agente instalado directamente y ahora quiere usar un agente que se ejecuta en un contenedor, primero necesita desinstalar el Agente de Log Analytics para Linux. Para obtener información sobre cómo desinstalar correctamente el agente, vea [Desinstalar el agente de Log Analytics para Linux](../learn/quick-collect-linux-computer.md).  

#### <a name="configure-a-log-analytics-agent-for-docker-swarm"></a>Configurar un agente de Log Analytics para Docker Swarm

Puede ejecutar el agente de Log Analytics como un servicio global en Docker Swarm. Use la información siguiente para crear un servicio del agente de Log Analytics. Debe proporcionar el identificador y la clave principal del área de trabajo de Log Analytics.

- Ejecute lo siguiente en el nodo principal.

    ```
    sudo docker service create  --name omsagent --mode global  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --mount type=bind,source=/var/lib/docker/containers,destination=/var/lib/docker/containers -e WSID="<WORKSPACE ID>" -e KEY="<PRIMARY KEY>" -p 25225:25225 -p 25224:25224/udp  --restart-condition=on-failure microsoft/oms
    ```

##### <a name="secure-secrets-for-docker-swarm"></a>Protección de secretos de Docker Swarm

En Docker Swarm, una vez que se crea el secreto del identificador de área de trabajo y de la clave principal, es preciso usar la información siguiente para crear la información del secreto.

1. Ejecute lo siguiente en el nodo principal.

    ```
    echo "WSID" | docker secret create WSID -
    echo "KEY" | docker secret create KEY -
    ```

2. Compruebe que los secretos se crearon correctamente.

    ```
    keiko@swarmm-master-13957614-0:/run# sudo docker secret ls
    ```

    ```
    ID                          NAME                CREATED             UPDATED
    j2fj153zxy91j8zbcitnjxjiv   WSID                43 minutes ago      43 minutes ago
    l9rh3n987g9c45zffuxdxetd9   KEY                 38 minutes ago      38 minutes ago
    ```

3. Ejecute el comando siguiente para montar los secretos en el agente de Log Analytics en contenedores.

    ```
    sudo docker service create  --name omsagent --mode global  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --mount type=bind,source=/var/lib/docker/containers,destination=/var/lib/docker/containers --secret source=WSID,target=WSID --secret source=KEY,target=KEY  -p 25225:25225 -p 25224:25224/udp --restart-condition=on-failure microsoft/oms
    ```

#### <a name="configure-a-log-analytics-agent-for-red-hat-openshift"></a>Configurar un agente de Log Analytics para Red Hat OpenShift

Hay tres formas de agregar el agente de Log Analytics a Red Hat OpenShift para empezar a recopilar datos de supervisión de contenedores.

* [Instalar el agente de Log Analytics para Linux](../learn/quick-collect-linux-computer.md) directamente en cada nodo de OpenShift  
* [Habilitar la extensión de máquina virtual de Log Analytics](../learn/quick-collect-azurevm.md) en cada nodo de OpenShift que reside en Azure  
* Instalar el agente de Log Analytics como daemon-set de OpenShift  

En esta sección, se describen los pasos necesarios para instalar al agente de Log Analytics como daemon-set de OpenShift.  

1. Inicie sesión en el nodo maestro de OpenShift y copie el archivo yaml [ocp-omsagent.yaml](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-omsagent.yaml) de GitHub en el nodo maestro y modifique el valor con el identificador del área de trabajo de Log Analytics y con la clave principal.
2. Ejecute los comandos siguientes para crear un proyecto de Azure Monitor y establecer la cuenta de usuario.

    ```
    oc adm new-project omslogging --node-selector='zone=default'
    oc project omslogging  
    oc create serviceaccount omsagent  
    oc adm policy add-cluster-role-to-user cluster-reader   system:serviceaccount:omslogging:omsagent  
    oc adm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent  
    ```

3. Para implementar daemon-set, ejecute lo siguiente:

    `oc create -f ocp-omsagent.yaml`

4. Para comprobar que se ha configurado y funciona correctamente, escriba lo siguiente:

    `oc describe daemonset omsagent`  

    y la salida debería ser similar a la siguiente:

    ```
    [ocpadmin@khm-0 ~]$ oc describe ds oms  
    Name:           oms  
    Image(s):       microsoft/oms  
    Selector:       name=omsagent  
    Node-Selector:  zone=default  
    Labels:         agentVersion=1.4.0-12  
                    dockerProviderVersion=10.0.0-25  
                    name=omsagent  
    Desired Number of Nodes Scheduled: 3  
    Current Number of Nodes Scheduled: 3  
    Number of Nodes Misscheduled: 0  
    Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed  
    No events.  
    ```

Siga este procedimiento si quiere usar secretos para proteger el identificador y la clave principal del área de trabajo de Log Analytics al usar el archivo yaml de daemon-set del agente de Log Analytics.

1. Inicie sesión en el nodo principal de OpenShift y copie el archivo yaml [ocp-ds-omsagent.yaml](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-ds-omsagent.yaml) y el script de generación de secretos [ocp-secretgen.sh](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-secretgen.sh) de GitHub.  Este script genera el archivo yaml de secretos para el identificador y la clave principal del área de trabajo de Log Analytics para proteger su información secreta.  
2. Ejecute los comandos siguientes para crear un proyecto de Azure Monitor y establecer la cuenta de usuario. El script de generación de secretos le pide el identificador `<WSID>` y la clave principal `<KEY>` del área de trabajo de Log Analytics y, al terminar, crea el archivo ocp-secret.yaml.  

    ```
    oc adm new-project omslogging --node-selector='zone=default'  
    oc project omslogging  
    oc create serviceaccount omsagent  
    oc adm policy add-cluster-role-to-user cluster-reader   system:serviceaccount:omslogging:omsagent  
    oc adm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent  
    ```

3. Implemente el archivo de secretos ejecutando lo siguiente:

    `oc create -f ocp-secret.yaml`

4. Compruebe la implementación ejecutando lo siguiente:

    `oc describe secret omsagent-secret`  

    y la salida debería ser similar a la siguiente:  

    ```
    [ocpadmin@khocp-master-0 ~]$ oc describe secret omsagent-secret  
    Name:           omsagent-secret  
    Namespace:      omslogging  
    Labels:         <none>  
    Annotations:    <none>  

    Type:   Opaque  

    Data  
    ====  
    KEY:    89 bytes  
    WSID:   37 bytes  
    ```

5. Para implementar el archivo yaml de daemon-set del agente de Log Analytics, ejecute lo siguiente:

    `oc create -f ocp-ds-omsagent.yaml`  

6. Compruebe la implementación ejecutando lo siguiente:

    `oc describe ds oms`

    y la salida debería ser similar a la siguiente:

    ```
    [ocpadmin@khocp-master-0 ~]$ oc describe ds oms  
    Name:           oms  
    Image(s):       microsoft/oms  
    Selector:       name=omsagent  
    Node-Selector:  zone=default  
    Labels:         agentVersion=1.4.0-12  
                    dockerProviderVersion=10.0.0-25  
                    name=omsagent  
    Desired Number of Nodes Scheduled: 3  
    Current Number of Nodes Scheduled: 3  
    Number of Nodes Misscheduled: 0  
    Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed  
    No events.  
    ```

#### <a name="configure-a-log-analytics-linux-agent-for-kubernetes"></a>Configurar un agente de Log Analytics para Linux para Kubernetes

En Kubernetes, se usa un script para generar el archivo yaml de secretos del identificador del área de trabajo y la clave principal para instalar el agente de Log Analytics para Linux. En la página de [GitHub de Log Analytics Docker Kubernetes](https://github.com/Microsoft/OMS-docker/tree/master/Kubernetes), hay archivos que se pueden usar con o sin la información de secretos.

- El DaemonSet predeterminado del agente de Log Analytics para Linux no tiene información de secretos (omsagent.yaml)
- El archivo yaml del DaemonSet del agente de Log Analytics para Linux usa información de secretos (omsagent-ds-secrets.yaml) con scripts de generación de secretos para generar el archivo yaml (omsagentsecret.yaml) de secretos.

Puede elegir crear DaemonSets de omsagent con o sin secretos.

**Archivo yaml de DaemonSet de OMSagent predeterminado sin secretos**

- En el archivo yaml de DaemonSet del agente de Log Analytics predeterminado, reemplace `<WSID>` y `<KEY>` por su WSID y CLAVE. Copie el archivo en el nodo principal y ejecute lo siguiente:

    ```
    sudo kubectl create -f omsagent.yaml
    ```

**Archivo yaml de DaemonSet de OMSagent predeterminado con secretos**

1. Para usar el DaemonSet del agente de Log Analytics con información de secretos, primero cree los secretos.
    1. Copie el script y el archivo de plantilla secreto y asegúrese de que estén en el mismo directorio.
        - Script de generación de secretos: secret-gen.sh
        - Plantilla de secretos: secret-template.yaml
    2. Ejecute el script, como en el ejemplo siguiente. El script pide el identificador y la clave principal del área de trabajo de Log Analytics y, una vez que los escribe, se crea un archivo yaml secreto para que pueda ejecutarlo.   

        ```
        #> sudo bash ./secret-gen.sh
        ```

    3. Ejecute lo siguiente para crear el pod de secretos:
        ```
        sudo kubectl create -f omsagentsecret.yaml
        ```

    4. Para realizar una comprobación, ejecute lo siguiente:

        ```
        keiko@ubuntu16-13db:~# sudo kubectl get secrets
        ```

        La salida debe ser similar a lo siguiente:

        ```
        NAME                  TYPE                                  DATA      AGE
        default-token-gvl91   kubernetes.io/service-account-token   3         50d
        omsagent-secret       Opaque                                2         1d
        ```

        ```
        keiko@ubuntu16-13db:~# sudo kubectl describe secrets omsagent-secret
        ```

        La salida debe ser similar a lo siguiente:

        ```
        Name:           omsagent-secret
        Namespace:      default
        Labels:         <none>
        Annotations:    <none>

        Type:   Opaque

        Data
        ====
        WSID:   36 bytes
        KEY:    88 bytes
        ```

    5. Ejecute ```sudo kubectl create -f omsagent-ds-secrets.yaml``` para crear el daemon-set de omsagent

2. Compruebe que el DaemonSet del agente de Log Analytics esté ejecutándose, de forma similar a la siguiente:

    ```
    keiko@ubuntu16-13db:~# sudo kubectl get ds omsagent
    ```

    ```
    NAME       DESIRED   CURRENT   NODE-SELECTOR   AGE
    omsagent   3         3         <none>          1h
    ```

En Kubernetes, use un script para generar el archivo yaml de secretos del identificador de área de trabajo y la clave principal para el agente de Log Analytics para Linux. Use la información de ejemplo siguiente con el [archivo yaml omsagent](https://github.com/Microsoft/OMS-docker/blob/master/Kubernetes/omsagent.yaml) para proteger la información secreta.

```
keiko@ubuntu16-13db:~# sudo kubectl describe secrets omsagent-secret
Name:           omsagent-secret
Namespace:      default
Labels:         <none>
Annotations:    <none>

Type:   Opaque

Data
====
WSID:   36 bytes
KEY:    88 bytes
```

#### <a name="configure-a-log-analytics-windows-agent-for-kubernetes"></a>Configurar un agente de Log Analytics para Windows para Kubernetes

En Windows Kubernetes, se usa un script para generar el archivo yaml de secretos del identificador del área de trabajo y la clave principal para instalar el agente de Log Analytics. En la página de [GitHub de Log Analytics Docker Kubernetes](https://github.com/Microsoft/OMS-docker/tree/master/Kubernetes/windows), hay archivos que se pueden usar con la información de secretos.  Es necesario instalar el agente de Log Analytics por separado en los nodos principal y de agente.  

1. Para usar el DaemonSet del agente de Log Analytics mediante la información de secretos en el nodo maestro, primero inicie sesión y cree los secretos.
    1. Copie el script y el archivo de plantilla secreto y asegúrese de que estén en el mismo directorio.
        - Script de generación de secretos: secret-gen.sh
        - Plantilla de secretos: secret-template.yaml

    2. Ejecute el script, como en el ejemplo siguiente. El script pide el identificador y la clave principal del área de trabajo de Log Analytics y, una vez que los escribe, se crea un archivo yaml secreto para que pueda ejecutarlo.

        ```
        #> sudo bash ./secret-gen.sh
        ```
    3. Ejecute ```kubectl create -f omsagentsecret.yaml``` para crear el daemon-set de omsagent
    4. Para comprobarlo, ejecute lo siguiente:

        ```
        root@ubuntu16-13db:~# kubectl get secrets
        ```

        La salida debe ser similar a lo siguiente:

        ```
        NAME                  TYPE                                  DATA      AGE
        default-token-gvl91   kubernetes.io/service-account-token   3         50d
        omsagent-secret       Opaque                                2         1d
        root@ubuntu16-13db:~# kubectl describe secrets omsagent-secret
        Name:           omsagent-secret
        Namespace:      default
        Labels:         <none>
        Annotations:    <none>

        Type:   Opaque

        Data
        ====
        WSID:   36 bytes
        KEY:    88 bytes
        ```

    5. Ejecute ```kubectl create -f ws-omsagent-de-secrets.yaml``` para crear el daemon-set de omsagent

2. Compruebe que el DaemonSet del agente de Log Analytics esté ejecutándose, de forma similar a la siguiente:

    ```
    root@ubuntu16-13db:~# kubectl get deployment omsagent
    NAME       DESIRED   CURRENT   NODE-SELECTOR   AGE
    omsagent   1         1         <none>          1h
    ```

3. Para instalar el agente en el nodo de trabajo, en el que se ejecuta Windows, siga los pasos descritos en la sección [Instalación y configuración de hosts de contenedor de Windows](#install-and-configure-windows-container-hosts).

#### <a name="use-helm-to-deploy-log-analytics-agent-on-linux-kubernetes"></a>Usar Helm para implementar un agente de Log Analytics en Linux Kubernetes

Para usar Helm con el fin de implementar el agente de Log Analytics en un entorno de Linux Kubernetes, siga este procedimiento.

1. Ejecute ```helm install --name omsagent --set omsagent.secret.wsid=<WSID>,omsagent.secret.key=<KEY> stable/msoms``` para crear el daemon-set de omsagent
2. El resultado será similar al siguiente:

    ```
    NAME:   omsagent
    LAST DEPLOYED: Tue Sep 19 20:37:46 2017
    NAMESPACE: default
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1/Secret
    NAME            TYPE    DATA  AGE
    omsagent-msoms  Opaque  3     3s

    ==> v1beta1/DaemonSet
    NAME            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
    omsagent-msoms  3        3        3      3           3          <none>         3s
    ```

3. Puede comprobar el estado de la omsagent. Para ello, debe ejecutar ```helm status "omsagent"``` y el resultado será similar al siguiente:

    ```
    keiko@k8s-master-3814F33-0:~$ helm status omsagent
    LAST DEPLOYED: Tue Sep 19 20:37:46 2017
    NAMESPACE: default
    STATUS: DEPLOYED
 
    RESOURCES:
    ==> v1/Secret
    NAME            TYPE    DATA  AGE
    omsagent-msoms  Opaque  3     17m
 
    ==> v1beta1/DaemonSet
    NAME            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
    omsagent-msoms  3        3        3      3           3          <none>         17m
    ```
   
    Para más información, visite [Container Solution Helm Chart](https://aka.ms/omscontainerhelm).

### <a name="install-and-configure-windows-container-hosts"></a>Instalación y configuración de hosts de contenedor de Windows

Use la información de la sección para instalar y configurar hosts de contenedor de Windows.

#### <a name="preparation-before-installing-windows-agents"></a>Preparación antes de instalar agentes de Windows

Antes de instalar agentes en equipos con Windows, debe configurar el servicio Docker. La configuración permite que el agente de Windows o la extensión de máquina virtual de Azure Monitor utilice el socket TCP de Docker para que los agentes puedan acceder al demonio de Docker de forma remota y capturar datos para supervisión.

##### <a name="to-configure-the-docker-service"></a>Para configurar el servicio Docker  

Ejecute los siguientes comandos de PowerShell para habilitar la canalización TCP y canalizaciones con nombre en Windows Server:

```
Stop-Service docker
dockerd --unregister-service
dockerd --register-service -H npipe:// -H 0.0.0.0:2375  
Start-Service docker
```

Para más información sobre la configuración del demonio de Docker usada con contenedores Windows, consulte [Docker Engine on Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon) (Docker Engine en Windows).

#### <a name="install-windows-agents"></a>Instalación de agentes de Windows

Para habilitar la supervisión de contenedores de Hyper-V y Windows, instale Microsoft Monitoring Agent (MMA) en equipos Windows que sean hosts de contenedores. Para equipos con Windows en su entorno local, consulte el artículo sobre [conexión de equipos Windows a Azure Monitor](../platform/agent-windows.md). En máquinas virtuales que se ejecutan en Azure, conéctelas a Azure Monitor mediante la [extensión de máquina virtual](../learn/quick-collect-azurevm.md).

Puede supervisar los contenedores de Windows que se ejecutan en Service Fabric. Sin embargo, actualmente solo las [máquinas virtuales que se ejecutan en Azure](../learn/quick-collect-azurevm.md) y en [equipos Windows de su entorno local](../platform/agent-windows.md) son compatibles con Service Fabric.

Puede comprobar que la solución de Supervisión de contenedores está configurada correctamente para Windows. Para comprobar si el módulo de administración se descargó correctamente, busque *ContainerManagement.xxx*. Los archivos deben estar en la carpeta C:\Archivos de programa\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs.

## <a name="solution-components"></a>Componentes de soluciones

En Azure Portal, vaya a la *Galería de soluciones* y agregue la **solución de supervisión de contenedores**. Si está utilizando agentes de Windows, el siguiente módulo de administración se instala en cada equipo con un agente cuando se agrega esta solución. No es necesario realizar tareas de configuración o mantenimiento del módulo de administración.

- *ContainerManagement.xxx* instalado en C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs

## <a name="container-data-collection-details"></a>Información detallada sobre la recopilación de datos en contenedores

La solución de Supervisión de contenedores recopila diversos datos de registro y métricas de rendimiento de los hosts de contenedores y de los contenedores mediante agentes que el usuario habilita.

Los siguientes tipos de agente recopilan los datos cada tres minutos.

- [Agente de Log Analytics para Linux](../learn/quick-collect-linux-computer.md)
- [Agente de Windows](../platform/agent-windows.md)
- [Extensión de VM de Log Analytics](../learn/quick-collect-azurevm.md)

### <a name="container-records"></a>Registros de contenedor

En la tabla siguiente se muestran ejemplos de registros recopilados por la solución de Supervisión de contenedores y los tipos de datos que aparecen en los resultados de las búsquedas de registros.

| Tipo de datos | Tipo de datos en la búsqueda de registros | Fields |
| --- | --- | --- |
| Rendimiento de hosts y contenedores | `Perf` | Computer, ObjectName, CounterName &#40;%Processor Time, Disk Reads MB, Disk Writes MB, Memory Usage MB, Network Receive Bytes, Network Send Bytes, Processor Usage sec, Network&#41;, CounterValue,TimeGenerated, CounterPath, SourceSystem |
| Inventario de contenedor | `ContainerInventory` | TimeGenerated, Computer, container name, ContainerHostname, Image, ImageTag, ContainerState, ExitCode, EnvironmentVar, Command, CreatedTime, StartedTime, FinishedTime, SourceSystem, ContainerID, ImageID |
| Inventario de imágenes de contenedor | `ContainerImageInventory` | TimeGenerated, Computer, Image, ImageTag, ImageSize, VirtualSize, Running, Paused, Stopped, Failed, SourceSystem, ImageID, TotalContainer |
| Registro de contenedor | `ContainerLog` | TimeGenerated, Computer, image ID, container name, LogEntrySource, LogEntry, SourceSystem, ContainerID |
| Registro del servicio de contenedores | `ContainerServiceLog`  | TimeGenerated, Computer, TimeOfCommand, Image, Command, SourceSystem, ContainerID |
| Inventario de nodo de contenedor | `ContainerNodeInventory_CL`| TimeGenerated, Computer, ClassName_s, DockerVersion_s, OperatingSystem_s, Volume_s, Network_s, NodeRole_s, OrchestratorType_s, InstanceID_g, SourceSystem|
| Inventario de Kubernetes | `KubePodInventory_CL` | TimeGenerated, Computer, PodLabel_deployment_s, PodLabel_deploymentconfig_s, PodLabel_docker_registry_s, Name_s, Namespace_s, PodStatus_s, PodIp_s, PodUid_g, PodCreationTimeStamp_t, SourceSystem |
| Proceso del contenedor | `ContainerProcess_CL` | TimeGenerated, Computer, Pod_s, Namespace_s, ClassName_s, InstanceID_s, Uid_s, PID_s, PPID_s, C_s, STIME_s, Tty_s, TIME_s, Cmd_s, Id_s, Name_s, SourceSystem |
| Eventos de Kubernetes | `KubeEvents_CL` | TimeGenerated, Computer, Name_s, ObjectKind_s, Namespace_s, Reason_s, Type_s, SourceComponent_s, SourceSystem, Message |

Las etiquetas que se anexan a los tipos de datos *PodLabel* son sus propias etiquetas personalizadas. Las etiquetas de PodLabel anexadas que se muestran en la tabla son ejemplos. Por lo tanto, `PodLabel_deployment_s`, `PodLabel_deploymentconfig_s`, `PodLabel_docker_registry_s` se diferencian en el conjunto de datos de su entorno y genéricamente son similares a `PodLabel_yourlabel_s`.

## <a name="monitor-containers"></a>Supervisión de contenedores
Una vez habilitada la solución en Azure Portal, el icono **Contenedores** muestra la información de resumen de los hosts de contenedores y de los contenedores en ejecución en los hosts.

![Icono de Containers](./media/containers/containers-title.png)

El icono muestra información general de cuántos contenedores hay en el entorno y si su estado es de error, en ejecución o detenido.

### <a name="using-the-containers-dashboard"></a>Uso del panel Containers

Haga clic en el icono **Containers**. Desde allí verá las vistas organizadas por:

- **Eventos de contenedor**: muestra el estado de contenedor y los equipos con contenedores en estado de error.
- **Registros de contenedor**: muestra un gráfico de archivos de registro de contenedor generados durante el tiempo y una lista de equipos con el mayor número de archivos de registro.
- **Eventos de Kubernetes**: muestra un gráfico de eventos de Kubernetes generados durante el tiempo y una lista de los motivos por los que los pods generaron los eventos. *Este conjunto de datos solo se usa en entornos Linux.*
- **Inventario de espacios de nombres de Kubernetes**: muestra el número de espacios de nombres y pods y muestra su jerarquía. *Este conjunto de datos solo se usa en entornos Linux.*
- **Inventario de nodo de contenedor**: muestra el número de tipos de orquestación usados en los nodos o hosts del contenedor. Los nodos de equipo y hosts también se enumeran según el número de contenedores. *Este conjunto de datos solo se usa en entornos Linux.*
- **Inventario de imágenes de contenedor**: muestra el número total de imágenes de contenedor que se utilizan y el número de tipos de imagen. También se muestra el número de imágenes según la etiqueta de imagen.
- **Estado de contenedores**: muestra el número total de nodos de contenedor y equipos host que tienen contenedores en ejecución. También se enumeran los equipos según el número de hosts en ejecución.
- **Procesos del contenedor**: muestra un gráfico de líneas de procesos de contenedor en ejecución a lo largo del tiempo. También se enumeran los contenedores según los comandos o procesos en ejecución en los contenedores. *Este conjunto de datos solo se usa en entornos Linux.*
- **Rendimiento de la CPU del contenedor**: muestra un gráfico de líneas del promedio de uso de CPU a lo largo del tiempo para los nodos y hosts. También enumera los nodos y hosts según el uso medio de CPU.
- **Rendimiento de la memoria del contenedor**: muestra un gráfico de líneas del uso de memoria a lo largo del tiempo. También muestra el uso de memoria de los equipos según el nombre de la instancia.
- **Rendimiento del equipo**: muestra gráficos de líneas del porcentaje de rendimiento de la CPU, del porcentaje de uso de memoria y de los megabytes de espacio libre en disco a lo largo del tiempo. Puede mantener el ratón sobre cualquier línea de un gráfico para ver más detalles.

Cada área del panel es una representación visual de una búsqueda que se ejecuta en los datos recopilados.

![Panel Containers](./media/containers/containers-dash01.png)

![Panel Containers](./media/containers/containers-dash02.png)

En el área **Estado del contenedor**, haga clic en la parte superior, como se muestra a continuación.

![Estado de los contenedores](./media/containers/containers-status.png)

Se abre Log Analytics, con información sobre el estado de los contenedores.

![Log Analytics para contenedores](./media/containers/containers-log-search.png)

Desde aquí, puede editar la consulta de búsqueda para modificarla y encontrar la información específica que le interesa. Para obtener más información sobre las consultas de registro, consulte el artículo sobre [consultas de registro en Azure Monitor](../log-query/log-query-overview.md).

## <a name="troubleshoot-by-finding-a-failed-container"></a>Solución de problemas mediante la búsqueda de un contenedor con error

Log Analytics marca un contenedor como **Failed** (Error) si ha terminado con un código de salida distinto de cero. Puede ver un resumen de los errores del entorno en el área **Contenedores con errores**.

### <a name="to-find-failed-containers"></a>Para buscar contenedores con errores

1. Haga clic en el área **Estado del contenedor**.  
   ![estado de los contenedores](./media/containers/containers-status.png)
2. Log Analytics se abre y muestra el estado de los contenedores, de forma similar a lo siguiente.  
   ![estado de los contenedores](./media/containers/containers-log-search.png)
3. Expanda la línea Erróneo y haga clic en + para agregar sus criterios a la consulta. Marque como comentario la línea Resumir de la consulta.
   ![contenedores con errores](./media/containers/containers-state-failed-select.png)  
1. Ejecute la consulta y expanda una línea de los resultados para ver el identificador de la imagen.  
   ![contenedores con errores](./media/containers/containers-state-failed.png)  
1. Escriba lo siguiente en la consulta de registro. `ContainerImageInventory | where ImageID == <ImageID>` para ver detalles de la imagen como tamaño de la imagen y el número de imágenes detenidas y con error.  
   ![contenedores con errores](./media/containers/containers-failed04.png)

## <a name="query-logs-for-container-data"></a>Consulta de registros de datos de contenedor

Cuando está solucionando un error específico, puede resultar de ayuda ver dónde se está produciendo en su entorno. Los siguientes tipos de registro le ayudarán a crear consultas que devuelven la información que desea.

- **ContainerImageInventory**: use este tipo si desea encontrar información organizada por imagen, y para ver información sobre la imagen, por ejemplo, identificadores o tamaños de imagen.
- **ContainerInventory**: use este tipo si desea obtener información sobre la ubicación del contenedor, cuáles son sus nombres y qué imágenes se están ejecutando.
- **ContainerLog**: use este tipo si desea buscar la información de registro de un error específico y sus entradas.
- **ContainerNodeInventory_CL** use este tipo cuando desee ver la información sobre el nodo o host donde se encuentran los contenedores. Proporciona información sobre la versión de Docker, el tipo de orquestación, almacenamiento y red.
- **ContainerProcess_CL** use este tipo para ver rápidamente los procesos en ejecución en el contenedor.
- **ContainerServiceLog**: use este tipo si desea encontrar información de seguimiento de auditoría para el daemon de Docker, tales como los comandos de inicio, detención, eliminación o extracción.
- **KubeEvents_CL** use este tipo para ver los eventos de Kubernetes.
- **KubePodInventory_CL** use este tipo cuando desee conocer la información de la jerarquía del clúster.

### <a name="to-query-logs-for-container-data"></a>Para consultar registros de datos de contenedor

* Elija una imagen que sabe que ha tenido errores recientemente y busque los registros de errores de ella. Para empezar, busque un nombre de contenedor que esté ejecutando esa imagen con una búsqueda **ContainerInventory**. Por ejemplo, busque `ContainerInventory | where Image == "ubuntu" and ContainerState == "Failed"`.  
    ![Búsqueda de contenedores de Ubuntu](./media/containers/search-ubuntu.png)

  Expanda cualquier fila de los resultados para ver los detalles del contenedor.

## <a name="example-log-queries"></a>Consultas de registro de ejemplo

Suele resultar útil crear consultas a partir de un ejemplo o dos y modificarlas para adaptarlas a su entorno. Para empezar, puede experimentar con el área **Consultas de ejemplo** situada en el extremo derecho de la página de la solución para ayudarle a crear consultas más avanzadas.

![Consultas de contenedores](./media/containers/containers-queries.png)

## <a name="saving-log-queries"></a>Guardado de consultas de registro

El guardado de consultas es una característica estándar de Azure Monitor. Guárdelas para volver a usar en el futuro aquellas que haya encontrado más útiles.

Después de crear una consulta que encuentre útil, guárdela haciendo clic en **Favoritos** en la parte superior de la página Búsqueda de registros. Podrá volver a acceder fácilmente a ella más adelante desde la página **Mi panel**.

## <a name="next-steps"></a>Pasos siguientes

[Consulta de registros](../log-query/log-query-overview.md), para ver los registros de datos detallados de los contenedores.

