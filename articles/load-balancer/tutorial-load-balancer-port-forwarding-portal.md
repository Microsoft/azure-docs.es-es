---
title: 'Tutorial: Configuración del reenvío de puertos: Azure Portal'
titleSuffix: Azure Load Balancer
description: En este tutorial se explica cómo configurar el reenvío de puertos mediante Azure Load Balancer para crear conexiones a las máquinas virtuales de una red virtual de Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: As an IT administrator, I want to configure port forwarding in Azure Load Balancer to remotely connect to VMs in an Azure virtual network.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/26/2019
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: dcb151c8be0ab3a2393d0659b75985a92ac60507
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "82207894"
---
# <a name="tutorial-configure-port-forwarding-in-azure-load-balancer-using-the-portal"></a>Tutorial: Configuración del enrutamiento de puerto en Azure Load Balancer mediante Azure Portal

El enrutamiento de puerto le permite conectarse a máquinas virtuales (VM) en una red virtual de Azure con el uso de un número de puerto y una dirección IP pública de Azure Load Balancer. 

En este tutorial, se va a configurar el enrutamiento de puerto en Azure Load Balancer. Aprenderá a:

> [!div class="checklist"]
> * Crear un equilibrador de carga estándar público para equilibrar el tráfico de red a través de las máquinas virtuales. 
> * Crear una red virtual y máquinas virtuales con una regla de grupo de seguridad de red (NSG). 
> * Agregar las máquinas virtuales al grupo de direcciones de back-end del equilibrador de carga.
> * Crear un sondeo de mantenimiento del equilibrador de carga y reglas de tráfico.
> * Crear reglas de enrutamiento de puerto de entrada del equilibrador de carga.
> * Instalar y configurar IIS en las máquinas virtuales para ver el equilibrio de carga y el enrutamiento de puerto en acción.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

Para realizar todos los pasos de este tutorial, inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

En primer lugar, cree un equilibrador de carga estándar público que pueda equilibrar la carga de tráfico a través de las máquinas virtuales. Un equilibrador de carga estándar solo admite una dirección IP pública estándar. Cuando se crea un equilibrador de carga estándar, también se crea una dirección IP pública estándar, que se configura como el front-end de equilibrador de carga y se denomina **LoadBalancerFrontEnd** de manera predeterminada. 

1. En la parte superior izquierda de la pantalla, haga clic en **Crear un recurso** > **Redes** > **Azure Load Balancer**.
2. En la pestaña **Datos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información, acepte los valores predeterminados del resto de la configuración y, luego, seleccione **Revisar y crear**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba *MyResourceGroupLB* en el cuadro de texto.|
    | Nombre                   | *myLoadBalancer*                                   |
    | Region         | Seleccione **Oeste de Europa**.                                        |
    | Tipo          | Seleccione **Público**.                                        |
    | SKU           | Seleccione **Estándar**.                          |
    | Dirección IP pública | Seleccione **Crear nuevo**. |
    | Nombre de la dirección IP pública              | Escriba *myPublicIP* en el cuadro de texto.   |
    |Zona de disponibilidad| Seleccione **Redundancia de zona**.    |
     
    >[!NOTE]
     >Asegúrese de crear el equilibrador de carga y todos los recursos para él en una ubicación compatible con Availability Zones. Para más información, vea [Regiones que admiten Availability Zones](../availability-zones/az-region.md). 

3. En la pestaña **Revisar y crear**, haga clic en **Crear**.  
  
## <a name="create-and-configure-back-end-servers"></a>Creación y configuración de servidores back-end

Cree una red virtual con dos máquinas virtuales y agréguelas al grupo de back-end del equilibrador de carga. 

## <a name="virtual-network-and-parameters"></a>Red virtual y parámetros

En los pasos de esta sección, tendrá que reemplazar los siguientes parámetros por la siguiente información:

| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLB (seleccione el grupo de recursos existente) |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Oeste de Europa      |
| **\<IPv4-address-space>**   | 10.3.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.3.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

### <a name="create-vms-and-add-them-to-the-load-balancer-back-end-pool"></a>Crear máquinas virtuales y agregarlas al grupo de back-end del equilibrador de carga

1. En la parte superior izquierda del portal, seleccione **Crear un recurso** > **Proceso** > **Windows Server 2016 Datacenter**. 
   
1. En **Crear una máquina virtual**, escriba o seleccione los valores siguientes en la pestaña **Básico**:
   - **Suscripción** > **Grupo de recursos**: despliegue y seleccione **MyResourceGroupLB**.
   - **Nombre de la máquina virtual**: escriba *MyVM1*.
   - **Región**: Seleccione **Oeste de Europa**. 
   - **Nombre de usuario**: escriba *azureuser*.
   - **Contraseña**: escriba *Azure1234567*. 
     Vuelva a escribir la contraseña en el campo **Confirmar contraseña**.
   
1. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**. 
   
   Asegúrese de que está seleccionado lo siguiente:
   - **Red virtual**: **MyVNet**
   - **Subred**: **MyBackendSubnet**
   
1. En **IP pública**, seleccione **Crear nueva**, seleccione **Estándar** en la página **Crear dirección IP pública** y, después, seleccione **Aceptar**. 
   
1. En **Grupo de seguridad de red**, seleccione **Avanzado** para crear un grupo de seguridad de red (NSG), un tipo de firewall. 
   1. En el campo **Configurar grupo de seguridad de red**, seleccione **Crear nuevo**. 
   1. Escriba *MyNetworkSecurityGroup* y seleccione **Aceptar**. 
   
   >[!NOTE]
   >Observe que, de forma predeterminada, el NSG ya tiene una regla de entrada para abrir el puerto 3389, el puerto de escritorio remoto (RDP).
   
1. Agregue la máquina virtual a un grupo de back-end de equilibrador de carga que cree:
   
   1. En **EQUILIBRIO DE CARGA** > **Place this virtual machine behind an existing load balancing solution?** (¿Colocar esta máquina virtual detrás de una solución de equilibrio de carga existente?), seleccione **Sí**. 
   1. En **Opciones de equilibrio de carga**, seleccione **Azure Load Balancer** en el menú desplegable. 
   1. En **Select a load balancer** (Seleccionar un equilibrador de carga), seleccione **MyLoadBalancer** en el menú desplegable. 
   1. En **Select a backend pool** (Seleccionar un grupo de back-end), seleccione **Crear nuevo**, escriba *MyBackendPool* y seleccione **Crear**. 
   
   ![Creación de una red virtual](./media/tutorial-load-balancer-port-forwarding-portal/create-vm-networking.png)
   
1. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**. En **Supervisión**, establezca **Diagnósticos de arranque** en **Desactivado**.
   
1. Seleccione **Revisar + crear**.
   
1. Revise la configuración y, cuando la validación sea correcta, seleccione **Crear**. 

1. Siga los pasos para crear una segunda máquina virtual llamada *MyVM2*, con todos los demás valores igual a MyVM1. 
   
   En **Grupo de seguridad de red**, después de seleccionar **Avanzado**, seleccione en el menú desplegable el grupo **MyNetworkSecurityGroup** que ya ha creado. 
   
   En **Select a backend pool** (Seleccionar un grupo de back-end), asegúrese de que **MyBackendPool** está seleccionado. 

### <a name="create-an-nsg-rule-for-the-vms"></a>Creación de una regla de NSG para las máquinas virtuales

Cree una regla de grupo de seguridad de red (NSG) para que las máquinas virtuales permitan conexiones entrantes de Internet (HTTP).

>[!NOTE]
>De forma predeterminada, el NSG ya tiene una regla que abre el puerto 3389, el puerto de escritorio remoto (RDP).

1. Seleccione **Todos los recursos** en el menú izquierdo. En la lista de recursos, seleccione **MyNetworkSecurityGroup** en el grupo de recursos **MyResourceGroupLB**.
   
1. En **Configuración**, seleccione **Reglas de seguridad de entrada** y, a continuación, seleccione **Agregar**.
   
1. En el cuadro de diálogo **Agregar regla de seguridad de entrada**, escriba o seleccione lo siguiente:
   
   - **Origen**: seleccione **Etiqueta de servicio**.  
   - **Etiqueta de servicio de origen**: seleccione **Internet**. 
   - **Intervalos de puerto de destino**: escriba *80*.
   - **Protocolo**: seleccione **TCP**. 
   - **Acción**: seleccione **Permitir**.  
   - **Prioridad**: escriba *100*. 
   - **Nombre**: escriba *MyHTTPRule*. 
   - **Descripción**: escriba *Permitir HTTP*. 
   
1. Seleccione **Agregar**. 
   
   ![Creación de una regla de NSG](./media/tutorial-load-balancer-port-forwarding-portal/8-load-balancer-nsg-rules.png)
   
## <a name="create-load-balancer-resources"></a>Creación de recursos del equilibrador de carga

En esta sección, va a inspeccionar el grupo de back-end del equilibrador de carga y a configurar reglas de tráfico y sondeo de mantenimiento de un equilibrador de carga.

### <a name="view-the-back-end-address-pool"></a>Visualización del grupo de direcciones de back-end

Para distribuir el tráfico a las máquinas virtuales, el equilibrador de carga usa un grupo de direcciones de back-end que contiene las direcciones IP de las interfaces de red virtual (NIC) conectadas al equilibrador de carga. 

Se ha creado el grupo de back-end del equilibrador de carga y se han agregado máquinas virtuales a él cuando se crearon las máquinas virtuales. También puede crear grupos de back-end y agregar o quitar máquinas virtuales en la página de **Grupos de back-end** del equilibrador de carga. 

1. Seleccione **Todos los recursos** en el menú de la izquierda y, a continuación, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Grupos de back-end**.
   
1. En la página **Grupos back-end**, expanda **MyBackendPool** y asegúrese de que aparecen ambas máquinas **VM1** y **VM2**.

1. Seleccione **MyBackendPool**. 
   
   En la página **MyBackendPool**, en **MÁQUINA VIRTUAL** y **DIRECCIÓN IP**, puede agregar las máquinas virtuales disponibles al grupo o quitarlas.

Puede crear grupos de back-end; para ello, seleccione **Agregar** en la página **Grupos de back-end**.

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

Para permitir que el equilibrador de carga supervise el mantenimiento de la máquina virtual, utilice un sondeo de mantenimiento. El sondeo de estado agrega o quita de forma dinámica las máquinas virtuales de la rotación del equilibrador de carga en base a su respuesta a las comprobaciones de estado. 

1. Seleccione **Todos los recursos** en el menú de la izquierda y, a continuación, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Sondeos de mantenimiento** y, a continuación, seleccione **Agregar**.
   
1. En la página **Agregar sondeo de mantenimiento**, escriba o seleccione los siguientes valores:
   
   - **Name**: escriba *MyHealthProbe*.
   - **Protocolo**: en la lista desplegable, seleccione **HTTP**. 
   - **Puerto**: escriba *80*. 
   - **Ruta de acceso**: acepte */* para el identificador URI predeterminado. Puede reemplazar este valor por cualquier otro identificador URI. 
   - **Intervalo**: escriba *15*. El valor Intervalo es el número de segundos entre los intentos de sondeo.
   - **Umbral incorrecto**: escriba *2*. Este valor es el número de errores de sondeo consecutivos que tienen que producirse para que se considere que una máquina virtual no funciona correctamente.
   
1. Seleccione **Aceptar**.
   
   ![Incorporación de un sondeo](./media/tutorial-load-balancer-port-forwarding-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla del equilibrador de carga define cómo se distribuye el tráfico a las máquinas virtuales. La regla define la configuración IP de front-end para el tráfico entrante y el grupo de direcciones IP de back-end para recibir el tráfico y los puertos de origen y destino requeridos. 

La regla del equilibrador de carga llamada **MyLoadBalancerRule** escucha en el puerto 80 en el front-end **LoadBalancerFrontEnd**. La regla envía el tráfico de red al grupo de direcciones de back-end **MyBackendPool**, también a través del puerto 80. 

1. Seleccione **Todos los recursos** en el menú de la izquierda y, a continuación, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.
   
1. En la página **Agregar regla de equilibrio de carga**, escriba o seleccione los valores siguientes:
   
   - **Name**: escriba *MyLoadBalancerRule*.
   - **Protocolo**: seleccione **TCP**.
   - **Puerto**: escriba *80*.
   - **Puerto back-end**: escriba *80*.
   - **Grupo de back-end**: seleccione **MyBackendPool**.
   - **Sondeo de mantenimiento**: seleccione **MyHealthProbe**. 
   
1. Seleccione **Aceptar**.
   
   ![Incorporación de una regla de equilibrador de carga](./media/tutorial-load-balancer-port-forwarding-portal/5-load-balancing-rules.png)

## <a name="create-an-inbound-nat-port-forwarding-rule"></a>Creación de una regla de enrutamiento de puerto NAT de entrada

Cree una regla de traducción de direcciones de red (NAT) de entrada del equilibrador de carga para enrutar el tráfico desde un puerto específico de la dirección IP front-end a un puerto específico de una máquina virtual back-end.

1. Seleccione **Todos los recursos** en el menú de la izquierda y, después, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Reglas NAT de entrada** y, después, seleccione **Agregar**. 
   
1. En la página **Agregar regla NAT de entrada**, escriba o seleccione los valores siguientes:
   
   - **Name**: escriba *MyNATRuleVM1*.
   - **Puerto**: escriba *4221*.
   - **Máquina virtual de destino**: seleccione **MyVM1** en la lista desplegable.
   - **Configuración de IP de red**: seleccione **ipconfig1** en la lista desplegable.
   - **Asignación de puertos**: seleccione **Personalizada**.
   - **Puerto de destino**: escriba *3389*.
   
1. Seleccione **Aceptar**.
   
1. Repita los pasos para agregar una regla NAT de entrada denominada *MyNATRuleVM2*, con **Puerto**: *4222* y **Máquina virtual de destino**: **MyVM2**.

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

En esta sección, se instalará Internet Information Services (IIS) en los servidores back-end y se personalizará la página web predeterminada para mostrar el nombre de la máquina. Después, se usará la dirección IP pública del equilibrador de carga para probar el equilibrador de carga. 

Cada máquina virtual de back-end sirve una versión diferente de la página web IIS predeterminada, de esta forma puede ver como el equilibrador de carga distribuye las solicitudes entre las dos máquinas virtuales.

### <a name="connect-to-the-vms-with-rdp"></a>Conexión a las máquinas virtuales con RDP

Conéctese a cada máquina virtual con Escritorio remoto (RDP). 

1. En el portal, seleccione **Todos los recursos** en el menú izquierdo. En la lista de recursos, seleccione cada máquina virtual en el grupo de recursos **MyResourceGroupLB**.
   
1. En la página **Información general**, seleccione **Conectar** y, a continuación, seleccione **Descargar archivo RDP**. 
   
1. Abra el archivo RDP que descargó y seleccione **Conectar**.
   
1. En la pantalla Seguridad de Windows, seleccione **Más opciones** y, después, **Usar otra cuenta**. 
   
   Escriba el nombre de usuario *azureuser* y la contraseña *Azure1234567* y seleccione **Aceptar**.
   
1. Responda **Sí** a cualquier solicitud de certificado. 
   
   El escritorio de la máquina virtual se abre en una nueva ventana. 

### <a name="install-iis-and-replace-the-default-iis-web-page"></a>Instalar IIS y reemplazar la página web predeterminada de IIS 

Use PowerShell para instalar IIS y reemplazar la página web predeterminada de IIS por una página en la que se muestre el nombre de la máquina virtual.

1. Iniciar **Windows PowerShell** en MyVM1 y MyVM2 desde el menú **Inicio**. 

2. Ejecute los comandos siguientes para instalar IIS y reemplazar la página web predeterminada de IIS:
   
   ```powershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    
   ```
   
1. Cierre las conexiones RDP con MyVM1 y MyVM2 seleccionando **Desconectar**. No apague las máquinas virtuales.

### <a name="test-load-balancing"></a>Prueba del equilibrio de carga

1. En el portal, en la página **Información general** de **MyLoadBalancer**, copie la dirección IP pública en **Dirección IP pública**. Mantenga el puntero sobre la dirección y seleccione el icono **Copiar** para copiarlo. En este ejemplo, es **40.67.218.235**. 
   
1. Pegue o escriba la dirección IP pública del equilibrador de carga (*40.67.218.235*) en la barra de direcciones del explorador de Internet. 
   
   La página predeterminada del servidor web de IIS personalizada aparece en el explorador. El mensaje que aparece será **Hola mundo de MyVM1**, u **Hola mundo de MyVM2**.
   
   ![Nueva página predeterminada de IIS](./media/tutorial-load-balancer-port-forwarding-portal/9-load-balancer-test.png) 
   
1. Actualice el explorador para ver cómo el equilibrador de carga distribuye el tráfico entre máquinas virtuales. A veces aparece la página **MyVM1** y otras es la página **MyVM2** la que aparece, ya que el equilibrador de carga distribuye las solicitudes a cada máquina virtual de back-end.
   
   >[!NOTE]
   >Puede que deba borrar la caché del explorador o abrir una nueva ventana del explorador entre intentos.

## <a name="test-port-forwarding"></a>Comprobación del reenvío de puertos

Con el enrutamiento de puerto, puede remitir el escritorio remoto a una máquina virtual back-end con la dirección IP del equilibrador de carga y el valor del puerto de front-end definidos en la regla NAT. 

1. En el portal, en la página **Información general** de **MyLoadBalancer**, copie su dirección IP pública. Mantenga el puntero sobre la dirección y seleccione el icono **Copiar** para copiarlo. En este ejemplo, es **40.67.218.235**. 
   
1. Abra un símbolo del sistema y use el siguiente comando para crear una sesión de escritorio remoto con MyVM2, utilizando la dirección IP pública del equilibrador de carga y el puerto de front-end que definió en la regla NAT de la máquina virtual. 
   
   ```
   mstsc /v:40.67.218.235:4222
   ```
  
1. Abra el archivo RDP descargado y seleccione **Conectar**.
   
1. En la pantalla Seguridad de Windows, seleccione **Más opciones** y, después, **Usar otra cuenta**. 
   
   Escriba el nombre de usuario *azureuser* y la contraseña *Azure1234567* y seleccione **Aceptar**.
   
1. Responda **Sí** a cualquier solicitud de certificado. 
   
   El escritorio de MyVM2 se abre en una nueva ventana. 

La conexión RDP se realiza correctamente, porque la regla NAT entrante **MyNATRuleVM2** dirige el tráfico del puerto de front-end del equilibrador de carga 4222 al puerto de MyVM2 3389 (puerto RDP).

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el equilibrador de carga y todos los recursos relacionados cuando ya no los necesite, abra el grupo de recursos **MyResourceGroupLB** y seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un equilibrador de carga público estándar. Ha creado y configurado los recursos de red, servidores back-end, un sondeo de mantenimiento y las reglas para el equilibrador de carga. Ha instalado IIS en las máquinas virtuales de back-end y ha utilizado la dirección IP pública del equilibrador de carga para probar el equilibrador de carga. Ha configurado y probado el enrutamiento de puerto desde un puerto específico del equilibrador de carga a un puerto de la máquina virtual de back-end. 

Para más información sobre Azure Load Balancer, consulte otros tutoriales sobre Load Balancer.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Load Balancer](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
