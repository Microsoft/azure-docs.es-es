---
title: 'Tutorial: Creación de un equilibrador de carga interno: Azure Portal'
titleSuffix: Azure Load Balancer
description: En este tutorial se muestra cómo crear un equilibrador de carga interno estándar mediante Azure Portal.
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internal traffic to virtual machines within a specific zone in a region.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/08/2020
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: f7f16093074b48610c1db8fec7f05ee01e7ab1ed
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078766"
---
# <a name="tutorial-balance-internal-traffic-load-with-a-standard-load-balancer-in-the-azure-portal"></a>Tutorial: Equilibrio de la carga de tráfico interno con un equilibrador de carga estándar en Azure Portal

El equilibrio de carga proporciona un mayor nivel de disponibilidad y escala, ya que distribuye las solicitudes entrantes entre varias máquinas virtuales (VM). Puede usar Azure Portal para crear un equilibrador de carga estándar y así equilibrar la carga del tráfico interno distribuyéndolo entre las máquinas virtuales. En este tutorial se muestra cómo crear recursos y configurar un equilibrador de carga interno, servidores back-end y recursos de red en el plan de tarifa estándar.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

Si lo prefiere, puede seguir estos pasos con la [CLI de Azure](load-balancer-get-started-ilb-arm-cli.md) o [Azure PowerShell](load-balancer-get-started-ilb-arm-ps.md) en lugar del portal.

Para realizar los pasos con este tutorial, inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="virtual-network-and-parameters"></a>Red virtual y parámetros
En los pasos de esta sección, tendrá que reemplazar los siguientes parámetros por la siguiente información:

| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupSLB |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Este de EE. UU. 2      |
| **\<IPv4-address-space>**   | 10.3.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.3.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]
   


## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

1. En la parte superior izquierda del portal, seleccione **Crear un recurso** > **Proceso** > **Windows Server 2016 Datacenter**. 
   
1. En **Crear una máquina virtual**, escriba o seleccione los valores siguientes en la pestaña **Básico**:
   - **Suscripción** > **Grupo de recursos**: despliegue y seleccione **MyResourceGroupLB**.
   - **Detalles de instancia** > **Nombre de máquina virtual**: escriba **MyVM1**.
   - **Detalles de instancia** > **Región**: Seleccione **Este de EE. UU. 2**.
  
   
1. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**. 
   
   Asegúrese de que está seleccionado lo siguiente:
   - **Red virtual**: **MyVNet**
   - **Subred**: **MyBackendSubnet**
   - **Grupo de seguridad de red de NIC**: Seleccione **Básica**.
   - **IP pública** > Seleccione **Crear**, escriba los valores siguientes y seleccione **Aceptar**:
       - **Name**: **MyVM1-IP**.
       - **SKU**: Seleccione **Estándar**.
   - **Puertos de entrada públicos**: Seleccione **Permitir los puertos seleccionados**.
   - **Seleccionar puertos de entrada**: en la lista desplegable, seleccione **RDP (3389)** .

   
   
1. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**. En **Supervisión**, establezca **Diagnósticos de arranque** en **Desactivado**.
   
1. Seleccione **Revisar + crear**.
   
1. Revise la configuración y, a continuación, seleccione **Crear**. 

1. Siga los pasos para crear una segunda máquina virtual llamada **MyVM2**, con todos los demás valores igual a MyVM1. 

1. Vuelva a seguir los pasos para crear una tercera máquina virtual llamada **MyTestVM**. 

## <a name="create-a-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

Cree un equilibrador de carga interno estándar mediante el portal. El nombre y dirección IP que cree se configuran automáticamente como front-end del equilibrador de carga.

1. En el lado superior izquierdo del portal, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.
   
2. En la pestaña **Datos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información, acepte los valores predeterminados del resto de la configuración y, luego, seleccione **Revisar y crear**:

    | Configuración                 | Valor                                              |
    | ---                     | ---                                                |
    | Suscripción               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba *MyResourceGroupLB* en el cuadro de texto.|
    | Nombre                   | *myLoadBalancer*                                   |
    | Region         | Seleccione **Este de EE. UU. 2**.                                        |
    | Tipo          | seleccione **Interno**.                                        |
    | SKU           | Seleccione **Estándar**.                          |
    | Virtual network           | Seleccione *MyVNet*.                          |    
    | Asignación de dirección IP              | Seleccione **Estática**.   |
    | Dirección IP privada|escriba una dirección que esté en el espacio de direcciones de la red virtual y subred, por ejemplo *10.3.0.7*.  |

3. En la pestaña **Revisar y crear**, haga clic en **Crear**. 
   

## <a name="create-standard-load-balancer-resources"></a>Creación de recursos del equilibrador de carga estándar

En esta sección, va a configurar el equilibrador de carga para un grupo de direcciones de back-end y un sondeo de mantenimiento, y a especificar las reglas del equilibrador de carga.

### <a name="create-a-back-end-address-pool"></a>Creación del grupo de direcciones de back-end

Para distribuir el tráfico a las máquinas virtuales, el equilibrador de carga utiliza un grupo de direcciones de back-end. El grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red (NIC) virtuales conectadas al equilibrador de carga. 

**Para crear un grupo de direcciones de back-end que incluya VM1 y VM2:**

1. Seleccione **Todos los recursos** en el menú de la izquierda y, a continuación, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.
   
1. En la página **Agregar un grupo back-end**, escriba o seleccione los siguientes valores:
   
   - **Name**: escriba **MyBackendPool**.
   
1. En **Máquinas virtuales**: 
   1. Agregar **MyVM1** y **MyVM2** al grupo de back-end.
   2. Después de agregar cada máquina, en la lista desplegable seleccione su **Configuración IP de red**. 
     
1. Seleccione **Agregar**.
   
   ![Incorporación del grupo de direcciones de back-end](./media/tutorial-load-balancer-standard-internal-portal/3-load-balancer-backend-02.png)
   
1. En la página **Grupos back-end**, expanda **MyBackendPool** y asegúrese de que aparecen ambas máquinas **VM1** y **VM2**.

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

Para permitir que el equilibrador de carga supervise el mantenimiento de la máquina virtual, utilice un sondeo de mantenimiento. El sondeo de estado agrega o quita de forma dinámica las máquinas virtuales de la rotación del equilibrador de carga en base a su respuesta a las comprobaciones de estado. 

**Para crear un sondeo de mantenimiento para supervisar el mantenimiento de las máquinas virtuales:**

1. Seleccione **Todos los recursos** en el menú de la izquierda y, a continuación, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Sondeos de mantenimiento** y, a continuación, seleccione **Agregar**.
   
1. En la página **Agregar sondeo de mantenimiento**, escriba o seleccione los siguientes valores:
   
   - **Name**: escriba **MyHealthProbe**.
   - **Protocolo**: en la lista desplegable, seleccione **HTTP**. 
   - **Puerto**: escriba **80**. 
   - **Ruta de acceso**: acepte **/** para el identificador URI predeterminado. Puede reemplazar este valor por cualquier otro identificador URI. 
   - **Intervalo**: escriba **15**. El valor Intervalo es el número de segundos entre los intentos de sondeo.
   - **Umbral incorrecto**: escriba **2**. Este valor es el número de errores de sondeo consecutivos que tienen que producirse para que se considere que una máquina virtual no funciona correctamente.
   
1. Seleccione **Aceptar**.
   
   ![Incorporación de un sondeo](./media/tutorial-load-balancer-basic-internal-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla del equilibrador de carga define cómo se distribuye el tráfico a las máquinas virtuales. La regla define la configuración IP de front-end para el tráfico entrante y el grupo de direcciones IP de back-end para recibir el tráfico y los puertos de origen y destino requeridos. 

La regla del equilibrador de carga llamada **MyLoadBalancerRule** escucha en el puerto 80 en el front-end **LoadBalancerFrontEnd**. La regla envía el tráfico de red al grupo de direcciones de back-end **MyBackendPool**, también a través del puerto 80. 

**Para crear la regla de equilibrador de carga:**

1. Seleccione **Todos los recursos** en el menú de la izquierda y, a continuación, en la lista de recursos seleccione **MyLoadBalancer**.
   
1. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.
   
1. En la página **Agregar regla de equilibrio de carga**, escriba o seleccione los valores siguientes, si todavía no están presentes:
   
   - **Name**: escriba **MyLoadBalancerRule**.
   - **Dirección IP de front-end**: escriba **LoadBalancerFrontEnd** si no está ya presente.
   - **Protocolo**: seleccione **TCP**.
   - **Puerto**: escriba **80**.
   - **Puerto back-end**: escriba **80**.
   - **Grupo de back-end**: seleccione **MyBackendPool**.
   - **Sondeo de mantenimiento**: seleccione **MyHealthProbe**. 
   
Para configurar los [puertos de alta disponibilidad](load-balancer-ha-ports-overview.md) mediante Azure Portal, seleccione la casilla de verificación **HA Ports** (Puertos de alta disponibilidad). Al seleccionarla, la configuración del protocolo y el puerto relacionados se rellena automáticamente. 

1. Seleccione **Aceptar**.
   
   ![Incorporación de una regla de equilibrador de carga](./media/tutorial-load-balancer-basic-internal-portal/5-load-balancing-rules.png)

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

Instale Internet Information Services (IIS) en los servidores back-end, y luego use MyTestVM para probar el equilibrador de carga utilizando su dirección IP privada. Cada máquina virtual de back-end sirve una versión diferente de la página web IIS predeterminada, de esta forma puede ver como el equilibrador de carga distribuye las solicitudes entre las dos máquinas virtuales.

En el portal, en la página **Introducción** de **MyLoadBalancer**, encuentre su dirección IP en **Dirección IP privada**. Mantenga el puntero sobre la dirección y seleccione el icono **Copiar** para copiarlo. En este ejemplo, es **10.3.0.7**. 

### <a name="connect-to-the-vms-with-rdp"></a>Conexión a las máquinas virtuales con RDP

En primer lugar, conéctese a las tres máquinas virtuales con Escritorio remoto (RDP). 

>[!NOTE]
>De forma predeterminada, las máquinas virtuales ya tienen el puerto de **RDP** (Escritorio remoto) abierto para permitir el acceso remoto al escritorio. 

**Para acceder mediante Escritorio remoto (RDP) a las máquinas virtuales:**

1. En el portal, seleccione **Todos los recursos** en el menú izquierdo. En la lista de recursos, seleccione cada máquina virtual en el grupo de recursos **MyResourceGroupLB**.
   
1. En la página **Información general**, seleccione **Conectar** y, a continuación, seleccione **Descargar archivo RDP**. 
   
1. Abra el archivo RDP que descargó y seleccione **Conectar**.
   
1. En la pantalla Seguridad de Windows, seleccione **Más opciones** y, después, **Usar otra cuenta**. 
   
   Escriba el nombre de usuario y la contraseña y, después, seleccione **Aceptar**.
   
1. Responda **Sí** a cualquier solicitud de certificado. 
   
   El escritorio de la máquina virtual se abre en una nueva ventana. 

### <a name="install-iis-and-replace-the-default-iis-page-on-the-back-end-vms"></a>Instale IIS y reemplace la página predeterminada de IIS en las máquinas virtuales de back-end

En cada servidor back-end, use PowerShell para instalar IIS y reemplazar la página web predeterminada de IIS con una página personalizada.

>[!NOTE]
>También puede usar el **Asistente para agregar roles y características** en el **Administrador del servidor** para instalar IIS. 

**Para instalar IIS y actualizar la página web predeterminada con PowerShell:**

1. Iniciar **Windows PowerShell** en MyVM1 y MyVM2 desde el menú **Inicio**. 

2. Ejecute los comandos siguientes para instalar IIS y reemplazar la página web predeterminada de IIS:
   
   ```powershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
      remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add custom htm file
      Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```
1. Cierre las conexiones RDP con MyVM1 y MyVM2 seleccionando **Desconectar**. No apague las máquinas virtuales.

### <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

1. En MyTestVM, abra **Internet Explorer** y responda **Aceptar** a cualquier pregunta que aparezca durante la configuración. 
   
1. Pegue o escriba la dirección IP privada del equilibrador de carga (*10.3.0.7*) en la barra de direcciones del explorador. 
   
   La página predeterminada del servidor web de IIS personalizada aparece en el explorador. El mensaje que aparece será **Hola mundo de MyVM1**, u **Hola mundo de MyVM2**.
   
1. Actualice el explorador para ver cómo el equilibrador de carga distribuye el tráfico entre máquinas virtuales. Es posible que también tenga que borrar la caché del explorador entre intentos.

   A veces aparece la página **MyVM1** y otras es la página **MyVM2** la que aparece, ya que el equilibrador de carga distribuye las solicitudes a cada máquina virtual de back-end. 

   ![Nueva página predeterminada de IIS](./media/tutorial-load-balancer-basic-internal-portal/9-load-balancer-test.png) 
   
## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el equilibrador de carga y todos los recursos relacionados cuando ya no los necesite, abra el grupo de recursos **MyResourceGroupLB** y seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado un equilibrador de carga interno estándar. Ha creado y configurado los recursos de red, servidores back-end, un sondeo de mantenimiento y las reglas para el equilibrador de carga. Ha instalado IIS en las máquinas virtuales de back-end y ha utilizado una máquina virtual de prueba para probar el equilibrador de carga en el explorador. 

A continuación, aprenda a equilibrar la carga de las máquinas virtuales entre las zonas de disponibilidad.

> [!div class="nextstepaction"]
> [Equilibrio de carga de máquinas virtuales en distintas zonas de disponibilidad](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
