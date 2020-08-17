---
title: 'Conceptos: diagrama de redes de Red Hat OpenShift en Azure 4'
description: Diagrama de redes e información general de las redes de Red Hat OpenShift en Azure
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: container-service
ms.date: 06/22/2020
ms.openlocfilehash: fb81405e85d6e2653e0cf6c007c363493992161a
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419978"
---
# <a name="networking-in-azure-red-hat-on-openshift-4"></a>Redes en Red Hat OpenShift en Azure 4

En esta guía se describe una introducción a las redes en clústeres de Red Hat OpenShift en Azure 4, junto con un diagrama y una lista de puntos de conexión importantes.

Para más información sobre los conceptos básicos de las redes de OpenShift, consulte [Documentación de redes de Red Hat OpenShift en Azure 4](https://docs.openshift.com/aro/4/networking/understanding-networking.html).

## <a name="networking-concepts-in-azure-red-hat-openshift"></a>Conceptos de redes en Red Hat OpenShift en Azure

![Diagrama de redes de Red Hat OpenShift en Azure 4](./media/concepts-networking/aro4-networking-diagram.png)

Al implementar Red Hat OpenShift en Azure 4, todo el clúster se encuentra dentro de una red virtual. Dentro de esta red virtual, los nodos principales y los nodos de trabajo tienen su propia subred. Cada subred utiliza un equilibrador de carga interno y público.

## <a name="explanation-of-endpoints"></a>Explicación de los puntos de conexión

En la lista siguiente se describen los puntos de conexión más importantes de un clúster de Red Hat OpenShift en Azure.

* **aro-pls**
    * Se trata de un punto de conexión de Azure Private Link, usado por Microsoft y los ingenieros de confiabilidad de sitios de Red Hat para ayudar a administrar el clúster.
* **aro-internal-lb**
    * Este punto de conexión equilibra el tráfico al servidor de la API. Para este equilibrador de carga, los nodos maestros se encuentran en el grupo de back-end.
* **aro-public-lb**
    * Si la API es pública, este punto de conexión enruta y equilibra el tráfico al servidor de la API. Este punto de conexión asigna una dirección IP pública de salida para que los maestros puedan acceder a Azure Resource Manager e informar sobre el estado del clúster.
* **aro-internal**
    * Este punto de conexión equilibra el tráfico de servicio interno. Para este equilibrador de carga, los nodos de trabajo se encuentran en el grupo de back-end.
    * Este equilibrador de carga no se crea de forma predeterminada. Este equilibrador de carga se crea una vez que se crea un servicio de tipo equilibrador de carga con las anotaciones correctas. Por ejemplo: service.beta.kubernetes.io/azure-load-balancer-internal: "true".
* **Directivas de red (entrada)**
    * Se admiten como parte del SDN de OpenShift.
    * Habilitado de forma predeterminada, cumplimiento realizado por los clientes.
    * Compatible con NetworkPolicy v1, sin embargo, todavía no se admiten los tipos de "salida e IPBlock".
    * **aro**
    * Este punto de conexión equilibra el tráfico al servidor de la API. Para este equilibrador de carga, los nodos maestros se encuentran en el grupo de back-end.
  * **aro-internal-lb**
    * Este punto de conexión se usa para cualquier tráfico público. Al crear una aplicación y una ruta, esta es la ruta para el tráfico de entrada.
    * Este equilibrador de carga también cubre la conectividad de salida a Internet desde cualquier pod que se ejecute en los nodos de trabajo mediante reglas de salida de Azure Load Balancer.
        * Actualmente no se pueden configurar las reglas de salida. Se asignan 1024 puertos TCP a cada nodo.
        * DisableOutboundSnat no está configurada en las reglas del equilibrador de carga, por lo que los pods podrían obtener como dirección IP de salida cualquier dirección IP pública configurada en este ALB.
        * Como consecuencia de los dos puntos anteriores, la única manera de agregar puertos SNAT efímeros es agregar a ARO servicios públicos de tipo equilibrador de carga.
* **Directivas de red (salida)**
    * Las directivas de salida se admiten mediante la característica de firewall de salida en OpenShift.
    * Solo una por espacio de nombres o proyecto.
    * No se admiten directivas de salida en el espacio de nombres "default" (predeterminado).
    * Las reglas de la directiva de salida se evalúan por orden (de la primera a la última).
    * **aro-outbound-pip**
        * Este punto de conexión actúa como una dirección IP pública (PIP) para los nodos de trabajo.
        * Este punto de conexión permite a los servicios agregar a una lista de permitidos una dirección IP específica que procede de un clúster de Red Hat OpenShift en Azure.
* **aro-node-nsg**
    * Cuando se expone un servicio, la API crea una regla en este grupo de seguridad de red para que el tráfico fluya por los nodos y tenga acceso a ellos.
    * De forma predeterminada, este grupo de seguridad de red permite todo el tráfico de salida. Actualmente, el tráfico de salida solo se puede restringir al plano de control de Red Hat OpenShift en Azure.
* **aro-controlplane-nsg**
    * Este punto de conexión solo permite que el tráfico entre por el puerto 6443 para los nodos maestros.
* **Azure Container Registry**
    * Se trata de un registro de contenedor que Microsoft proporciona y utiliza internamente.
        * Este registro proporciona las imágenes de la plataforma del host y los componentes del clúster. Por ejemplo, los contenedores de supervisión o registro.
        * No está diseñado para que lo usen los clientes de Red Hat OpenShift en Azure.  
        * Solo lectura.
        * Las conexiones a este registro se producen mediante el punto de conexión de servicio (conectividad interna entre los servicios de Azure).
        * Este registro interno no está disponible fuera del clúster de forma predeterminada.
* **Private Link**
    * Permite la conectividad de red desde el plano de administración hasta un clúster para su administración.
    * Para los ingenieros de confiabilidad de sitios de Microsoft y Red Hat para ayudar a administrar el clúster.

## <a name="networking-basics-in-openshift"></a>Aspectos básicos de las redes en OpenShift

Las redes definidas por software (SDN) de OpenShift se usan para configurar una red de superposición mediante Open vSwitch (OVS), una implementación de OpenFlow basada en la especificación de la interfaz de red del contenedor (CNI). SDN admite distintos complementos y la directiva de red es el complemento que se usa en Red Hat OpenShift en Azure 4. SDN administra toda la comunicación de red, por lo que no se necesitan rutas adicionales en las redes virtuales para lograr la comunicación entre pod y pod.

## <a name="azure-red-hat-openshift-networking-specifics"></a>Características específicas de las redes en Red Hat OpenShift en Azure

Las siguientes características son específicas de Red Hat OpenShift en Azure:
* Se admite que traiga su propia red virtual.
* Se pueden configurar los CIDR de la red de servicio y los pods.
* Los nodos y los maestros se encuentran en subredes diferentes.
* Las subredes de la red virtual de los nodos y los maestros deben ser /27 como mínimo.
* El CIDR de lo pods debe tener un tamaño mínimo de /18 (la red de los pods tiene direcciones IP no enrutables y solo se usa dentro de SDN de OpenShift).
* Cada nodo está asignado a una subred /23 (512 direcciones IP) para los pods. Este valor no puede modificarse.
* No se puede conectar un pod a varias redes.
* No se puede configurar una dirección IP estática de salida. (Esta es una característica de OpenShift. Para más información, consulte [Configuración de direcciones IP de salida](https://docs.openshift.com/aro/4/networking/openshift_sdn/assigning-egress-ips.html)).

## <a name="network-settings"></a>Configuración de red

La siguiente configuración de red está disponible en Red Hat OpenShift en Azure 4:

* **Visibilidad de la API**: establezca la visibilidad de la API mediante la ejecución del [comando az aro create](tutorial-create-cluster.md#create-the-cluster).
    * "Público": el servidor de la API es accesible para redes externas.
    * "Privado": el servidor de la API asignó una dirección IP privada de la subred de los maestros, solo accesible mediante las redes conectadas (redes virtuales emparejadas, otras subredes del clúster). Se creará una zona DNS privada en nombre del cliente.
* **Visibilidad de entrada**: establezca la visibilidad de la API mediante la ejecución del [comando az aro create](tutorial-create-cluster.md#create-the-cluster).
    * Las rutas "públicas" se establecerán de forma predeterminada en una instancia pública de Azure Standard Load Balancer (se puede cambiar).
    * Las rutas "privadas" se establecerán de forma predeterminada en el equilibrador de carga interno (se puede cambiar).

## <a name="network-security-groups"></a>Grupos de seguridad de red
Los grupos de seguridad de red se crearán en el grupo de recursos del nodo, que está bloqueado. Los grupos de seguridad de red se asignan directamente a las subredes, no a las NIC del nodo. Los grupos de seguridad de red son inmutables, lo que significa que no tiene permisos para cambiarlos. 

Sin embargo, con un servidor de API visible públicamente, no puede crear grupos de seguridad de red y asignarlos a las NIC.

## <a name="domain-forwarding"></a>Reenvío de dominios
Red Hat OpenShift en Azure usa CoreDNS. Se puede configurar el reenvío de dominios (consulte la documentación sobre [Uso del reenvío de DNS](https://docs.openshift.com/aro/4/networking/dns-operator.html#nw-dns-forward_dns-operator) para más información).

Actualmente, no puede traer su propio sistema DNS a las redes virtuales.


Para más información sobre el tráfico de salida y lo que admite Red Hat OpenShift en Azure para la salida, consulte la documentación de [compatibilidad de directivas](support-policies-v4.md).
