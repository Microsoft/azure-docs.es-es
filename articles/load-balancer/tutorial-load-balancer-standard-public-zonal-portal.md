---
title: 'Tutorial: Máquinas virtuales de Load Balancer en una zona: Azure Portal'
titleSuffix: Azure Load Balancer
description: Este tutorial muestra cómo crear una instancia de Standard Load Balancer con un front-end de zona para equilibrar la carga de las máquinas virtuales en una zona de disponibilidad mediante Azure Portal
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines within a specific zone in a region.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2019
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: 940636a5e368a84aaaf0d4490bf874d56d3ddb6e
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "78251904"
---
# <a name="tutorial-load-balance-vms-within-an-availability-zone-with-standard-load-balancer-by-using-the-azure-portal"></a>Tutorial: Equilibrio de carga de máquinas virtuales en una zona de disponibilidad con Standard Load Balancer mediante Azure Portal

En este tutorial se crea una instancia de [Azure Standard Load Balancer](https://aka.ms/azureloadbalancerstandard) pública con un front-end de zona que usa una dirección IP pública estándar mediante Azure Portal. En este escenario, puede especificar una zona determinada para las instancias de front-end y back-end para alinear la ruta de acceso a los datos y los recursos con una zona específica. Aprenderá a realizar las siguientes funciones:

> [!div class="checklist"]
> * Crear una instancia de Standard Load Balancer con un front-end de zona.
> * Crear grupos de seguridad de red para definir las reglas de tráfico de entrada.
> * Crear máquinas virtuales de zona y conectarlas a un equilibrador de carga.
> * Crear un sondeo de mantenimiento del equilibrador de carga.
> * Crear reglas de tráfico del equilibrador de carga.
> * Crear un sitio de Internet Information Services (IIS) básico.
> * Visualizar un equilibrador de carga en acción.

Para más información sobre cómo usar las zonas de disponibilidad con Standard Load Balancer, consulte [Standard Load Balancer y Availability Zones](load-balancer-standard-availability-zones.md).

Si lo prefiere, utilice la [CLI de Azure](load-balancer-standard-public-zonal-cli.md) para completar este tutorial.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-public-standard-load-balancer-instance"></a>Creación de una instancia de Standard Load Balancer pública

Standard Load Balancer solo admite direcciones IP públicas estándar. Cuando se crea una dirección IP pública nueva al crear el equilibrador de carga, se configura automáticamente como una versión de la SKU Estándar. También tiene automáticamente redundancia de zona.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.
2. En la pestaña **Datos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información, acepte los valores predeterminados del resto de la configuración y, luego, seleccione **Revisar y crear**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba *MyResourceGroupZLB* en el cuadro de texto.|
    | Nombre                   | *myLoadBalancer*                                   |
    | Region         | Seleccione **Oeste de Europa**.                                        |
    | Tipo          | Seleccione **Público**.                                        |
    | SKU           | Seleccione **Estándar**.                          |
    | Dirección IP pública | Seleccione **Crear nuevo**. |
    | Nombre de la dirección IP pública              | Escriba *myPublicIP* en el cuadro de texto.   |
    |Zona de disponibilidad| Seleccione **1**.    |
3. En la pestaña **Revisar y crear**, haga clic en **Crear**.   

## <a name="create-backend-servers"></a>Creación de servidores back-end

En esta sección, creará una red virtual. También creará dos máquinas virtuales en la misma zona (es decir, la zona 1) para la región que desea agregar al grupo de back-end del equilibrador de carga. A continuación, se instala IIS en las máquinas virtuales para ayudar a probar el equilibrador de carga con redundancia de zona. Si se produce un error en una máquina virtual, se produce un error en el sondeo de mantenimiento de la máquina virtual en la misma zona. El tráfico continúa en otras máquinas virtuales dentro de la misma zona.

## <a name="virtual-network-and-parameters"></a>Red virtual y parámetros

En los pasos de esta sección, tendrá que reemplazar los siguientes parámetros por la siguiente información:

| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupZLB (seleccione el grupo de recursos existente) |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Oeste de Europa      |
| **\<IPv4-address-space>**   | 10.0.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.0.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso**. En el cuadro de búsqueda, escriba **Grupo de seguridad de red**. En la página Grupo de seguridad de red, seleccione **Crear**.
2. En la página **Crear grupo de seguridad de red**, escriba estos valores:
   - **myNetworkSecurityGroup** como nombre del grupo de seguridad de red.
   - **myResourceGroupLBAZ** como nombre del grupo de recursos existente.
   
     ![Crear un grupo de seguridad de red](./media/tutorial-load-balancer-standard-zonal-portal/create-network-security-group.png)

### <a name="create-nsg-rules"></a>Creación de reglas de NSG

En esta sección, se crean reglas de NSG para permitir conexiones entrantes que usen HTTP y Microsoft Remote Desktop Protocol (RDP) mediante Azure Portal.

1. Seleccione **Todos los recursos** en el menú de la izquierda de Azure Portal. A continuación, busque y seleccione **myNetworkSecurityGroup**. Se encuentra en el grupo de recursos **myResourceGroupZLB**.
2. En **Configuración**, seleccione **Reglas de seguridad de entrada**. A continuación, seleccione **Agregar**.
3. Especifique los siguientes valores para la regla de seguridad de entrada llamada **myHTTPRule** para permitir que las conexiones HTTP entrantes usen el puerto 80:
    - **Etiqueta de servicio** en **Origen**.
    - **Internet** en **Etiqueta de servicio de origen**.
    - **80** en **Intervalos de puerto de destino**.
    - **vTCP** en **Protocolo**.
    - **Permitir** en **Acción**.
    - **100** en **Prioridad**.
    - **myHTTPRule** en **Nombre**.
    - **Permitir HTTP** en **Descripción**.
4. Seleccione **Aceptar**.
 
   ![Creación de reglas de NSG](./media/load-balancer-standard-public-availability-zones-portal/8-load-balancer-nsg-rules.png)

5. Repita los pasos 2 a 4 para crear otra regla llamada **myRDPRule**. Esta regla permite una conexión RDP entrante que usa el puerto 3389, con los valores siguientes:
    - **Etiqueta de servicio** en **Origen**.
    - **Internet** en **Etiqueta de servicio de origen**.
    - **3389** en **Intervalos de puerto de destino**.
    - **TCP** en **Protocolo**.
    - **Permitir** en **Acción**.
    - **200** en **Prioridad**.
    - **myRDPRule** en **Nombre**.
    - **Permitir RDP** en **Descripción**.

      ![Creación de la regla de RDP](./media/tutorial-load-balancer-standard-zonal-portal/create-rdp-rule.png)

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

1. En la esquina superior izquierda de la pantalla, seleccione **Crear un recurso** > **Compute** > **Windows Server 2016 Datacenter**. Especifique estos valores para la máquina virtual:
    - **myVM1** como nombre de la máquina virtual.        
    - **azureuser** como nombre del usuario administrador.    
    - **myResourceGroupZLB** como **Grupo de recursos**. Seleccione **Usar existente** y, a continuación, seleccione **myResourceGroupZLB**.
2. Seleccione **Aceptar**.
3. Seleccione **DS1_V2** como tamaño de la máquina virtual. Elija **Seleccionar**.
4. Especifique estos valores para la configuración de la máquina virtual:
    - **Zona 1** como Zona de disponibilidad donde va a situar la máquina virtual.
    -  **myVNet**. Asegúrese de que está seleccionada como la red virtual.
    - **myVM1PIP** como la dirección IP pública estándar que crea. Seleccione **Crear nuevo**. A continuación, en nombre, seleccione **myVM1PIP**. Para **Zona**, seleccione **1**. La SKU de la dirección IP es estándar de manera predeterminada.
    - **myBackendSubnet**. Asegúrese de que está seleccionada como la subred.
    - **myNetworkSecurityGroup** como nombre del firewall del grupo de seguridad de red existente.
5. Seleccione **Deshabilitado** para deshabilitar los diagnósticos de arranque.
6. Seleccione **Aceptar**. Revise la configuración en la página Resumen. Seleccione **Crear**.
7. Repita los pasos 1 a 6 para crear una segunda máquina virtual, llamada **myVM2** en la Zona 1. Especifique **myVnet** como la red virtual. Especifique **myVM2PIP** como la dirección IP pública estándar. Especifique **myBackendSubnet** como la subred. Y especifique **myNetworkSecurityGroup** como el grupo de seguridad de red.

    ![Creación de máquinas virtuales](./media/tutorial-load-balancer-standard-zonal-portal/create-virtual-machine.png) 

### <a name="install-iis-on-vms"></a>Instalación de IIS en las máquinas virtuales

1. Seleccione **Todos los recursos** en el menú de la izquierda. A continuación, en la lista de recursos, seleccione **myVM1**. Se encuentra en el grupo de recursos **myResourceGroupZLB**.
2. En la página **Información general**, seleccione **Conectar** para conectar mediante RDP con la máquina virtual.
3. Inicie sesión en la máquina virtual con el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Para especificar las credenciales que especificó cuando creó la máquina virtual, tendrá que seleccionar **Más opciones**. Después, seleccione **Usar otra cuenta**. Y, a continuación, seleccione **Aceptar**. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Seleccione **Sí** para continuar con la conexión.
4. En el escritorio del servidor, vaya a **Herramientas administrativas de Windows** > **Windows PowerShell**.
6. En la ventana de **PowerShell**, ejecute los comandos siguientes para instalar el servidor IIS. Estos comandos también eliminan el archivo iisstart.htm predeterminado y, a continuación, agregan uno nuevo que muestra el nombre de la máquina virtual:

   ```azurepowershell-interactive
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from" + $env:computername)
   ```
7. Cierre la sesión de RDP con **myVM1**.
8. Repita los pasos del 1 al 7 para instalar IIS en **myVM2**.

## <a name="create-load-balancer-resources"></a>Creación de recursos del equilibrador de carga

En esta sección se configuran los valores del equilibrador de carga para un grupo de direcciones de back-end y un sondeo de mantenimiento. También se especifican reglas de traducción de direcciones de red y el equilibrador de carga.


### <a name="create-a-backend-address-pool"></a>Creación de un grupo de direcciones de back-end

Para distribuir el tráfico a las máquinas virtuales, un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red virtual conectadas al equilibrador de carga. Cree el grupo de direcciones de back-end **myBackendPool** para incluir **VM1** y **VM2**.

1. Seleccione **Todos los recursos** en el menú de la izquierda. A continuación, seleccione **myLoadBalancer** en la lista de recursos.
2. En **Configuración**, seleccione **Grupos de back-end**. A continuación, seleccione **Agregar**.
3. En la página **Agregar grupo back-end**, realice lo siguiente:
    - En el espacio para el nombre, escriba **myBackEndPool** como nombre del grupo de back-end.
    - Para **Red virtual**, en el menú desplegable, seleccione **myVNet**. 
    - Para **Máquina Virtual** y **Dirección IP**, agregue **myVM1** y **myVM2** y sus direcciones IP públicas correspondientes.
4. Seleccione **Agregar**.
5. Asegúrese de que la configuración del grupo de back-end del equilibrador de carga muestra las dos máquinas virtuales, **myVM1** y **myVM2**.
 
    ![Creación de un grupo de back-end](./media/tutorial-load-balancer-standard-zonal-portal/create-backend-pool.png) 

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

Para permitir que el equilibrador de carga supervise el estado de la aplicación, utilice un sondeo de mantenimiento. El sondeo de estado agrega o quita de forma dinámica las máquinas virtuales de la rotación del equilibrador de carga en base a su respuesta a las comprobaciones de estado. Cree un sondeo de estado, **myHealthProbe**, para supervisar el estado de las máquinas virtuales.

1. Seleccione **Todos los recursos** en el menú de la izquierda. A continuación, seleccione **myLoadBalancer** en la lista de recursos.
2. En **Configuración**, seleccione **Sondeos de estado**. A continuación, seleccione **Agregar**.
3. Use estos valores para crear el sondeo de estado:
    - **myHealthProbe** como nombre del sondeo de mantenimiento.
    - **HTTP** en tipo de protocolo.
    - **80** en número de puerto.
    - **15** como número de **Intervalo**, en segundos, entre los intentos de sondeo.
    - **2** como número de **Umbral incorrecto** o errores de sondeo consecutivos que deben producirse para que una máquina virtual se considere que no funciona de manera correcta.
4. Seleccione **Aceptar**.

   ![Agregar un sondeo de estado](./media/load-balancer-standard-public-availability-zones-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla del equilibrador de carga define cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración de la IP de front-end para el tráfico entrante y el grupo de IP de back-end para el tráfico entrante, junto con los puertos de origen y destino requeridos. Cree una regla del equilibrador de carga llamada **myLoadBalancerRuleWeb** para escuchar en el puerto 80 en el front-end **FrontendLoadBalancer**. La regla envía tráfico de red con equilibrio de carga al grupo de direcciones de back-end **myBackEndPool**, también a través del puerto 80. 

1. Seleccione **Todos los recursos** en el menú de la izquierda. A continuación, seleccione **myLoadBalancer** en la lista de recursos.
2. En **Configuración**, seleccione **Reglas de equilibrio de carga**. A continuación, seleccione **Agregar**.
3. Use estos valores para configurar la regla de equilibrio de carga:
    - **myHTTPRule** como nombre de la regla de equilibrio de carga.
    - **TCP** en tipo de protocolo.
    - **80** en número de puerto.
    - **80** como puerto de back-end.
    - **myBackendPool** como nombre del grupo back-end.
    - **myHealthProbe** como nombre del sondeo de mantenimiento.
4. Seleccione **Aceptar**.
    
    ![Agregar una regla de equilibrio de carga](./media/tutorial-load-balancer-standard-zonal-portal/load-balancing-rule.png)

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga
1. Busque la dirección IP pública de Load Balancer en la pantalla **Información general**. Seleccione **Todos los recursos**. A continuación, seleccione **myPublicIP**. 

2. Copie la dirección IP pública. A continuación, péguela en la barra de direcciones del explorador. La página predeterminada que incluye el nombre de la página del servidor web se muestra en el explorador.

      ![Servidor web IIS](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)
3. Para ver el equilibrador de carga en acción, fuerce la detención de la máquina virtual que se muestra. Actualice el explorador para ver el otro nombre de servidor que aparece en el explorador.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine el grupo de recursos, el equilibrador de carga y todos los recursos relacionados. Seleccione el grupo de recursos que contiene el equilibrador de carga. A continuación, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Load Balancer Estándar](load-balancer-standard-overview.md).
- [Equilibrio de carga de máquinas virtuales entre zonas de disponibilidad](tutorial-load-balancer-standard-public-zone-redundant-portal.md).
