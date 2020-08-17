---
title: Uso de Azure Firewall para proteger las implementaciones de Azure Kubernetes Service (AKS)
description: Aprenda a usar Azure Firewall para proteger las implementaciones de Azure Kubernetes Service (AKS)
author: vhorne
ms.service: firewall
services: firewall
ms.topic: how-to
ms.date: 07/29/2020
ms.author: victorh
ms.openlocfilehash: 602671f1052de2d9446f32946271cea2f9995044
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87412956"
---
# <a name="use-azure-firewall-to-protect-azure-kubernetes-service-aks-deployments"></a>Uso de Azure Firewall para proteger las implementaciones de Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) ofrece un clúster de Kubernetes administrado en Azure. Reduce la complejidad y la sobrecarga operativa de la administración de Kubernetes al descargar gran parte de esa responsabilidad en Azure. AKS controla las tareas críticas, como la supervisión y el mantenimiento de estado, y ofrece un clúster de nivel empresarial y seguro con gobernanza facilitada.

Kubernetes organiza los clústeres de máquinas virtuales y programa los contenedores para que se ejecuten en esas máquinas virtuales en función de sus recursos de proceso disponibles y de los requisitos de recursos de cada contenedor. Los contenedores se agrupan en pods, la unidad operativa básica de Kubernetes y los pods se escalan al estado que se quiere.

Para fines operativos y de administración, los nodos de un clúster de AKS deben tener acceso a determinados puertos y nombres de dominio completo (FQDN). Estas acciones podrían consistir en la comunicación con el servidor de la API o la descarga e instalación de actualizaciones de seguridad de nodos y componentes principales del clúster de Kubernetes. Azure Firewall puede ayudarle a bloquear su entorno y a filtrar el tráfico saliente.

Siga las instrucciones de este artículo para proporcionar protección adicional para el clúster de Azure Kubernetes mediante Azure Firewall.

## <a name="prerequisites"></a>Requisitos previos

- Un clúster de Azure Kubernetes implementado con una aplicación en ejecución.

   Para más información, consulte el [Tutorial: Implementación de un clúster de Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-deploy-cluster.md) y el [Tutorial: Ejecución de aplicaciones en Azure Kubernetes Service (AKS)](../aks/tutorial-kubernetes-deploy-application.md).


## <a name="securing-aks"></a>Protección de AKS

Azure Firewall proporciona una etiqueta FQDN AKS para simplificar la configuración. Siga estos pasos para permitir el tráfico saliente de la plataforma AKS:

- Cuando use Azure Firewall para restringir el tráfico saliente y cree una ruta definida por el usuario (UDR) para dirigir todo el tráfico saliente, asegúrese de crear una regla DNAT adecuada en el firewall para permitir correctamente el tráfico entrante. 

   El uso de Azure Firewall con una UDR interrumpe la configuración entrante debido al enrutamiento asimétrico. El problema se produce si la subred de AKS tiene una ruta predeterminada que va a la dirección IP privada del firewall, pero está usando un equilibrador de carga público. Por ejemplo, servicio de entrada o Kubernetes del tipo *equilibrador de carga*.

   En este caso, el tráfico entrante del equilibrador de carga se recibe a través de su dirección IP pública, pero la ruta de vuelta pasa a través de la dirección IP privada del firewall. Dado que el firewall es con estado, quita el paquete de vuelta porque el firewall no tiene conocimiento de una sesión establecida. Para aprender a integrar Azure Firewall con el equilibrador de carga de entrada o de servicio, consulte [Integración de Azure Firewall con Azure Standard Load Balancer](integrate-lb.md).
- Cree una colección de reglas de aplicación y agregue una regla para habilitar la etiqueta FQDN *AzureKubernetesService*. El intervalo de direcciones IP de origen es la red virtual del grupo de hosts, el protocolo es https y el destino es AzureKubernetesService.
- Un clúster de AKS requiere los siguientes puertos de salida / reglas de red:

   - Puerto TCP 443
   - TCP [*IPAddrOfYourAPIServer*]:443 es necesario si tiene una aplicación que debe comunicarse con el servidor de API. Este cambio se puede establecer después de crear el clúster.
   - Puerto TCP 9000 y puerto UDP 1194 para el pod de la parte delantera del túnel para comunicarse con el extremo de túnel en el servidor de la API.

      Para información más concreta, vea las direcciones *.hcp.<location>.azmk8s.io* en la tabla siguiente.
   - Puerto UDP 123 para la sincronización de hora del protocolo de tiempo de red (NTP) (nodos de Linux).
   - También se requiere el puerto UDP 53 para DNS si tiene pods que acceden directamente al servidor de API.

   Para más información, consulte [Control del tráfico de salida de los nodos de clúster en Azure Kubernetes Service (AKS)](../aks/limit-egress-traffic.md).
- Configure AzureMonitor y las etiquetas de servicio de Storage. Azure Monitor recibe datos de análisis de registros.

   También puede permitir la dirección URL del área de trabajo individualmente: `<worksapceguid>.ods.opinsights.azure.com` y `<worksapceguid>.oms.opinsights.azure.com`. Puede abordar esto de una de las formas siguientes:

    - Permita el acceso https desde la subred del grupo host a `*. ods.opinsights.azure.com`y `*.oms. opinsights.azure.com`. Estos FQDN con caracteres comodín permiten el acceso requerido, pero son menos restrictivos.
    - Use la siguiente consulta de análisis de registros para enumerar los FQDN necesarios exactos y, a continuación, permitirlos explícitamente en las reglas de aplicación del firewall:
   ```
   AzureDiagnostics 
   | where Category == "AzureFirewallApplicationRule" 
   | search "Allow" 
   | search "*. ods.opinsights.azure.com" or "*.oms. opinsights.azure.com"
   | parse msg_s with Protocol " request from " SourceIP ":" SourcePort:int " to " FQDN ":" * 
   | project TimeGenerated,Protocol,FQDN 
   ```


## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre Azure Kubernetes Service, consulte los [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)](../aks/concepts-clusters-workloads.md).
