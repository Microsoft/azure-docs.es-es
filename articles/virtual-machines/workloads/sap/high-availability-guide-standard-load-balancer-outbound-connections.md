---
title: Conectividad de punto de conexión público para VM de Azure y Standard ILB en escenarios de alta disponibilidad de SAP
description: Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP
services: virtual-machines-windows,virtual-network,storage,
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/16/2020
ms.author: radeltch
ms.openlocfilehash: a0dc9f673abcac549fffc7291b8ac376c297da6b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836129"
---
# <a name="public-endpoint-connectivity-for-virtual-machines-using-azure-standard-load-balancer-in-sap-high-availability-scenarios"></a>Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP

El objetivo de este artículo es describir las configuraciones que permitirán la conectividad saliente hacia los puntos de conexión públicos. Las configuraciones giran principalmente en torno al contexto de alta disponibilidad con Pacemaker para SUSE/RHEL.  

Si usa Pacemaker con el agente de barrera de Azure en la solución de alta disponibilidad, las máquinas virtuales deben tener conectividad de salida a la API de administración de Azure.  
El artículo presenta varias opciones para que pueda seleccionar la opción más adecuada para su escenario.  

## <a name="overview"></a>Información general

Al implementar la alta disponibilidad para las soluciones de SAP a través de la agrupación en clústeres, uno de los componentes necesarios es [Azure Load Balancer](../../../load-balancer/load-balancer-overview.md). Azure ofrece dos SKU de equilibrador de carga: estándar y básica.

El equilibrador de carga estándar de Azure ofrece algunas ventajas en comparación con el equilibrador de carga básico. Por ejemplo, funciona en todas las zonas de disponibilidad de Azure, tiene mejores funcionalidades de supervisión y registro para solucionar problemas más fácilmente y una latencia reducida. La característica "puertos de alta disponibilidad" cubre todos los puertos; es decir, ya no es necesario enumerar todos los puertos individuales.  

Hay algunas diferencias importantes entre las SKU básica y estándar del equilibrador de carga de Azure. Una de ellas es el control del tráfico saliente al punto de conexión público. Para obtener una comparación completa entre las SKU básica y estándar de Load Balancer, consulte [Comparación de las SKU de Load Balancer](../../../load-balancer/load-balancer-overview.md).  
 
Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno de Azure (sin dirección IP pública), no hay conectividad saliente hacia los puntos de conexión públicos, a menos que se realice una configuración adicional.  

Si se asigna una dirección IP pública a una máquina virtual, o si la máquina virtual está en el grupo de back-end de un equilibrador de carga con una dirección IP pública, tendrá conectividad saliente hacia los puntos de conexión públicos.  

Los sistemas SAP suelen contener datos empresariales confidenciales. Rara vez es aceptable que las máquinas virtuales que hospedan sistemas SAP tengan direcciones IP públicas. Al mismo tiempo, hay escenarios en los que es necesaria la conectividad saliente desde la máquina virtual hasta los puntos de conexión públicos.  

Algunos ejemplos de escenarios que requieren acceso al punto de conexión público de Azure, son:  
- Usar el agente de barrera de Azure como un mecanismo de barrera en los clústeres de Pacemaker.
- Azure Backup
- Azure Site Recovery  
- Usar el repositorio público para aplicar revisiones al sistema operativo.
- El flujo de datos de la aplicación de SAP puede requerir conectividad saliente al punto de conexión público

Si la implementación de SAP no requiere conectividad saliente a los puntos de conexión públicos, no es necesario implementar la configuración adicional. Basta con crear una instancia de Azure Load Balancer de SKU estándar interna para su escenario de alta disponibilidad, suponiendo que tampoco hay necesidad de conectividad entrante desde los puntos de conexión públicos.  

> [!Note]
> Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos.  
>Si las máquinas virtuales tienen direcciones IP públicas o ya están en el grupo de back-end del equilibrador de carga de Azure con una dirección IP pública, la máquina virtual ya tendrá conectividad saliente a los puntos de conexión públicos.


Consulte primero las siguientes notas:

* Azure Standard Load Balancer
  * [Información general sobre Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md): información general completa sobre Azure Standard Load Balancer, así como principios, conceptos y tutoriales importantes. 
  * [Conexiones salientes en Azure](../../../load-balancer/load-balancer-outbound-connections.md#scenarios): escenarios sobre cómo lograr la conectividad saliente en Azure.
  * [Reglas de salida del equilibrador de carga](../../../load-balancer/load-balancer-outbound-connections.md#outboundrules): se explican los conceptos de las reglas de salida del equilibrador de carga y cómo crear reglas de salida.
* Azure Firewall
  * [Información general de Azure Firewall](../../../firewall/overview.md): información general sobre Azure Firewall.
  * [Tutorial: implementación y configuración de Azure Firewall](../../../firewall/tutorial-firewall-deploy-portal.md): instrucciones sobre cómo configurar Azure Firewall a través de Azure Portal.
* [Redes virtuales: reglas definidas por el usuario](../../../virtual-network/virtual-networks-udr-overview.md#user-defined): conceptos y reglas de enrutamiento de Azure.  
* [Etiquetas de servicio de grupos de seguridad](../../../virtual-network/security-overview.md#service-tags): cómo simplificar los grupos de seguridad de red y la configuración del firewall con etiquetas de servicio.

## <a name="additional-external-azure-standard-load-balancer-for-outbound-connections-to-internet"></a>Standard Load Balancer externo de Azure adicional para conexiones salientes a Internet

Una opción para lograr la conectividad saliente a los puntos de conexión públicos, sin permitir la conectividad entrante a la máquina virtual desde el punto de conexión público, consiste en crear un segundo equilibrador de carga con una dirección IP pública, agregar las máquinas virtuales al grupo de back-end del segundo equilibrador de carga y definir solo [reglas de salida](../../../load-balancer/load-balancer-outbound-connections.md#outboundrules).  
Use los [grupos de seguridad de red](../../../virtual-network/security-overview.md) para controlar los puntos de conexión públicos, que están disponibles para las llamadas salientes desde la máquina virtual.  
Para obtener más información, consulte el segundo escenario del documento [Conexiones salientes](../../../load-balancer/load-balancer-outbound-connections.md#scenarios).  
La configuración sería similar a la siguiente:  

![Control de la conectividad a los puntos de conexión públicos con grupos de seguridad de red](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-public.png)

### <a name="important-considerations"></a>Consideraciones importantes

- Puede usar una instancia pública de Load Balancer adicional para varias máquinas virtuales en la misma subred. Así, logrará la conectividad saliente al punto de conexión público y optimizará los costos.  
- Use los [grupos de seguridad de red](../../../virtual-network/security-overview.md) para administrar qué puntos de conexión públicos están disponibles desde las máquinas virtuales. Puede asignar el grupo de seguridad de red a la subred o a cada máquina virtual. Siempre que sea posible, use las [etiquetas de servicio](../../../virtual-network/security-overview.md#service-tags) para reducir la complejidad de las reglas de seguridad.  
- Standard Load Balancer de Azure con una dirección IP pública y las reglas de salida permite el acceso directo a un punto de conexión público. Si uno de sus requisitos de seguridad empresariales requiere que todo el tráfico saliente pase a través de una solución corporativa centralizada para la auditoría y el registro, quizá no pueda cumplirlo con este escenario.  

>[!TIP]
>Siempre que sea posible, use las [etiquetas de servicio](../../../virtual-network/security-overview.md#service-tags) para reducir la complejidad del grupo de seguridad de red. 

### <a name="deployment-steps"></a>Pasos de implementación

1. Cree un equilibrador de carga  
   1. En [Azure Portal](https://portal.azure.com), haga clic en Todos los recursos, Agregar y después busque **Load Balancer**.  
   1. Haga clic en **Crear** 
   1. Nombre del equilibrador de carga **MyPublicILB**.  
   1. Seleccione **Público** como tipo y **estándar** como SKU.  
   1. Seleccione **Crear dirección IP pública** y especifique como nombre **MyPublicILBFrondEndIP**.  
   1. Seleccione **Con redundancia de zona** como Zona de disponibilidad.  
   1. Haga clic en Revisar y crear y después en Crear.  
2. Cree el grupo de back-end **MyBackendPoolOfPublicILB** y agregue las máquinas virtuales.  
   1. Seleccione la red virtual.  
   1. Seleccione las máquinas virtuales y sus direcciones IP y agréguelas al grupo de back-end.  
3. [Cree reglas de salida](../../../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard%3ftabs%3doption-1-create-load-balancer-standard#create-outbound-rule-configuration). Actualmente no es posible crear reglas de salida desde Azure Portal. Puede crear reglas de salida con la [CLI de Azure](../../../cloud-shell/overview.md?view=azure-cli-latest).  

   ```azurecli
    az network lb outbound-rule create --address-pool MyBackendPoolOfPublicILB --frontend-ip-configs MyPublicILBFrondEndIP --idle-timeout 30 --lb-name MyPublicILB --name MyOutBoundRules  --outbound-ports 10000 --enable-tcp-reset true --protocol All --resource-group MyResourceGroup
   ```

4. Cree reglas de grupo de seguridad de red para restringir el acceso a puntos de conexión públicos específicos. Si hay un grupo de seguridad de red existente, puede ajustarlo. En el ejemplo siguiente se muestra cómo habilitar el acceso a la API de administración de Azure: 
   1. Navegue al grupo de seguridad de red.
   1. Haga clic en Reglas de seguridad de salida.
   1. Agregue una regla a **Denegar** todo el acceso saliente a **Internet**.
   1. Agregue una regla a **Permitir** el acceso a **AzureCloud**, con una prioridad inferior a la prioridad de la regla para denegar todo el acceso a Internet.


   Las reglas de seguridad de salida tendrían el siguiente aspecto: 

   ![Conexión de salida con una segunda instancia de Load Balancer con IP pública](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-network-security-groups.png)

   Para obtener más información sobre los grupos de seguridad de red de Azure, consulte [Grupos de seguridad](../../../virtual-network/security-overview.md). 

## <a name="azure-firewall-for-outbound-connections-to-internet"></a>Azure Firewall para conexiones salientes a Internet

Otra opción para lograr la conectividad saliente a los puntos de conexión públicos (sin permitir la conectividad entrante a la máquina virtual desde puntos de conexión públicos) es con Azure Firewall. Azure Firewall es un servicio administrado, con alta disponibilidad integrada y puede abarcar varias Availability Zones.  
También tendrá que implementar una [Ruta definida por el usuario](../../../virtual-network/virtual-networks-udr-overview.md#custom-routes) asociada a la subred donde se implementan las máquinas virtuales y el equilibrador de carga de Azure. Dicha ruta debe apuntar al firewall de Azure para enrutar el tráfico a través de Azure Firewall.  
Para obtener más detalles sobre cómo implementar Azure Firewall, consulte [Implementación y configuración de Azure Firewall](../../../firewall/tutorial-firewall-deploy-portal.md).  

La arquitectura tendría este aspecto:

![Conexión saliente con Azure Firewall](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-firewall.png)

### <a name="important-considerations"></a>Consideraciones importantes

- Azure Firewall es un servicio nativo en la nube, con alta disponibilidad integrada que admite la implementación por zonas.
- Requiere una subred adicional que se debe denominar AzureFirewallSubnet. 
- Si se transfieren grandes conjuntos de datos de salida desde la red virtual donde se encuentran las máquinas virtuales de SAP a una máquina virtual en otra red virtual o a un punto de conexión público, es posible que no resulte ser una solución rentable. Un ejemplo de esto es el proceso de copiar copias de seguridad de gran tamaño entre redes virtuales. Para obtener más información, consulte Precios de Azure Firewall.  
- Si la solución de firewall corporativa no es Azure Firewall y tiene requisitos de seguridad para que todo el tráfico saliente pase a través de una solución corporativa centralizada, quizá esta solución no sea práctica.  

>[!TIP]
>Siempre que sea posible, use las [etiquetas de servicio](../../../virtual-network/security-overview.md#service-tags) para reducir la complejidad de las reglas de Azure Firewall.  

### <a name="deployment-steps"></a>Pasos de implementación

1. En los pasos de implementación se supone que ya definió la red virtual y la subred para las máquinas virtuales.  
2. Cree una subred **AzureFirewallSubnet** en la misma red virtual, donde se implementan las máquinas virtuales y el Standard Load Balancer.  
   1. En Azure Portal, vaya a la red virtual: Haga clic en Todos los recursos, busque la red virtual, haga clic en ella y seleccione Subredes.  
   1. Haga clic en Agregar subred. Escriba **AzureFirewallSubnet** como nombre. Escriba el intervalo de direcciones adecuado. Haga clic en Save (Guardar).  
3. Cree una instancia de Azure Firewall.  
   1. En Azure Portal seleccione Todos los recursos y haga clic en Agregar, Firewall, Crear. Seleccione Grupo de recursos (seleccione el mismo grupo de recursos en el que está la red virtual).  
   1. Escriba el nombre del recurso de Azure Firewall. Por ejemplo, **MyAzureFirewall**.  
   1. Seleccione la región y seleccione al menos dos zonas de disponibilidad, alineadas con las zonas de disponibilidad en las que están implementadas las máquinas virtuales.  
   1. Seleccione la red virtual en la que están implementadas las máquinas virtuales SAP y Standard Load Balancer de Azure.  
   1. Dirección IP pública: Haga clic en crear y escriba un nombre. Por ejemplo, **MyFirewallPublicIP**.  
4. Cree una regla de Azure Firewall para permitir la conectividad saliente a los puntos de conexión públicos especificados. En el ejemplo se muestra cómo permitir el acceso al punto de conexión público de la API de administración de Azure.  
   1. Haga clic en Reglas, Colección de reglas de red y después en Agregar colección de reglas de red.  
   1. Nombre: **MyOutboundRule**, escriba Prioridad y seleccione Acción **Permitir**.  
   1. Servicio: Nombre **ToAzureAPI**.  Protocolo: Seleccione **Todo**. Dirección de origen: especifique el intervalo de la subred en la que están implementadas las máquinas virtuales y la instancia de Standard Load Balancer. Por ejemplo: **11.97.0.0/24**. Puertos de destino: escriba <b>*</b>.  
   1. Save
   1. Mientras está en Azure Firewall, seleccione Información general. Escriba la dirección IP privada de Azure Firewall.  
5. Cree una ruta a Azure Firewall.  
   1. En Azure Portal, seleccione Todos los recursos y haga clic en Agregar, Tabla de rutas, Crear.  
   1. Escriba el nombre MyRouteTable, seleccione Suscripción, Grupo de recursos y Ubicación (que coincida con la ubicación de la red virtual y el firewall).  
   1. Save  

   La regla de firewall tendrá el siguiente aspecto: ![Conexión saliente con Azure Firewall](./media/high-availability-guide-standard-load-balancer/high-availability-guide-standard-load-balancer-firewall-rule.png)

6. Cree una ruta definida por el usuario desde la subred de las máquinas virtuales a la dirección IP privada de **MyAzureFirewall**.
   1. Cuando esté situado en la tabla de rutas, haga clic en Rutas. Seleccione Agregar. 
   1. Nombre de ruta: ToMyAzureFirewall, Prefijo de dirección: **0.0.0.0/0**. Tipo de próximo salto: Seleccione Aplicación virtual. Dirección de próximo salto: escriba la dirección IP privada del firewall que configuró: **11.97.1.4**.  
   1. Save

## <a name="using-proxy-for-pacemaker-calls-to-azure-management-api"></a>Uso de proxy para las llamadas de Pacemaker a la API de administración de Azure

Puede usar el proxy para permitir las llamadas de Pacemaker al punto de conexión público de la API de administración de Azure.  

### <a name="important-considerations"></a>Consideraciones importantes

  - Si ya existe un proxy corporativo, puede enrutar las llamadas salientes a los puntos de conexión públicos a través de él. Las llamadas salientes a puntos de conexión públicos pasarán por el punto de control corporativo.  
  - Asegúrese de que la configuración del proxy permite la conectividad de salida a la API de administración de Azure: `https://management.azure.com` y `https://login.microsoftonline.com`.  
  - Asegúrese de que hay una ruta de las máquinas virtuales al proxy.  
  - El proxy solo controlará las llamadas HTTP/HTTPS. Si se necesita realizar llamadas salientes adicionales a un punto de conexión público a través de diferentes protocolos (como RFC), será necesaria otra solución.  
  - La solución de proxy debe tener una alta disponibilidad para evitar la inestabilidad en el clúster de Pacemaker.  
  - En función de la ubicación del proxy, puede introducir latencias adicionales en las llamadas del agente de barrera de Azure a la API de administración de Azure. Si el proxy corporativo sigue en el entorno local, mientras que el clúster de Pacemaker se encuentra en Azure, mida la latencia y analice si esta solución es adecuada.  
  - Si aún no existe un proxy corporativo de alta disponibilidad, no se recomienda esta opción, ya que el cliente incurrirá en costos y complejidad adicionales. No obstante, si decide implementar una solución de proxy adicional con el fin de permitir la conectividad saliente de Pacemaker a la API pública de administración de Azure, asegúrese de que el proxy tiene alta disponibilidad y que la latencia de las máquinas virtuales al proxy es baja.  

### <a name="pacemaker-configuration-with-proxy"></a>Configuración de Pacemaker con proxy 

En el sector hay muchas opciones de proxy distintas disponibles. Las instrucciones paso a paso para la implementación de un proxy se encuentran fuera del ámbito de este documento. En el ejemplo siguiente, se supone que el proxy responde a **MyProxyService** y está en escucha del puerto **MyProxyPort**.  
Para permitir que Pacemaker se comunique con la API de administración de Azure, realice los pasos siguientes en todos los nodos del clúster:  

1. Edite el archivo de configuración /etc/sysconfig/pacemaker y agregue las líneas siguientes (en todos los nodos del clúster):

   ```console
   sudo vi /etc/sysconfig/pacemaker
   # Add the following lines
   http_proxy=http://MyProxyService:MyProxyPort
   https_proxy=http://MyProxyService:MyProxyPort
   ```

2. Reinicie el servicio Pacemaker en **todos** los nodos del clúster.  
  - SUSE
 
     ```console
     # Place the cluster in maintenance mode
     sudo crm configure property maintenance-mode=true
     #Restart on all nodes
     sudo systemctl restart pacemaker
     # Take the cluster out of maintenance mode
     sudo crm configure property maintenance-mode=true
     ```

  - Red Hat  

     ```console
     # Place the cluster in maintenance mode
     sudo pcs property set maintenance-mode=true
     #Restart on all nodes
     sudo systemctl restart pacemaker
     # Take the cluster out of maintenance mode
     sudo pcs property set maintenance-mode=false
     ```

## <a name="other-solutions"></a>Otras soluciones

Si el tráfico saliente se enruta a través de un firewall de terceros:

- Si se usa un agente de barrera de Azure, asegúrese de que la configuración del firewall permita la conectividad de salida a la API de administración de Azure: `https://management.azure.com` y `https://login.microsoftonline.com`.   
- Si se usa la infraestructura de actualización de la nube pública de Azure de SUSE para aplicar actualizaciones y revisiones, vea [Infraestructura de actualización de la nube pública de Azure 101](https://suse.com/c/azure-public-cloud-update-infrastructure-101/).

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a configurar Pacemaker en SUSE en Azure](./high-availability-guide-suse-pacemaker.md)
* [Aprenda a configurar Pacemaker en Red Hat en Azure](./high-availability-guide-rhel-pacemaker.md)