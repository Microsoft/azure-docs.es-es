---
title: 'Inicio rápido: Creación de un equilibrador de carga interno: Azure Portal'
titleSuffix: Azure Load Balancer
description: En este inicio rápido se muestra cómo crear un equilibrador de carga interno mediante Azure Portal.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a internal load balancer so that I can load balance internal traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/30/2020
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: fb0a5c06c8b7bdbb62e937e865e6bb2fd4000f2d
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87462451"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-using-the-azure-portal"></a>Inicio rápido: Creación de un equilibrador de carga interno para equilibrar la carga de las máquinas virtuales mediante Azure Portal

Para empezar a usar Azure Load Balancer diríjase a Azure Portal para crear un equilibrador de carga interno y dos máquinas virtuales.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

---

# <a name="option-1-default-create-a-internal-load-balancer-standard-sku"></a>[Opción 1 (valor predeterminado): crear un equilibrador de carga interno (SKU Estándar)](#tab/option-1-create-internal-load-balancer-standard)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción.  Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

En esta sección, va a crear un equilibrador de carga que equilibra la carga de las máquinas virtuales. 

Puede crear un equilibrador de carga público o interno. 

Cuando se crea un equilibrador de carga interno, se configura una red virtual como red para el equilibrador de carga. 

Una dirección IP privada de la red virtual se configura como front-end (denominado **LoadBalancerFrontend** de manera predeterminada) para el equilibrador de carga. 

La dirección IP de front-end puede ser **estática** o **dinámica**.

## <a name="virtual-network-and-parameters"></a>Red virtual y parámetros

En esta sección, reemplazará los parámetros de los pasos con la siguiente información:

| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLB |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Oeste de Europa      |
| **\<IPv4-address-space>**   | 10.1.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.1.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-load-balancer"></a>Creación de un equilibrador de carga

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **myResourceGroupLB**, que creó en el paso anterior.|
    | Nombre                   | Escriba **myLoadBalancer**.                                   |
    | Region         | Seleccione **Oeste de Europa**.                                        |
    | Tipo          | seleccione **Interno**.                                        |
    | SKU           | Seleccione **Estándar**. |
    | Virtual network | Select **myVNet**, que creó en el paso anterior. |
    | Subnet  | Select **myBackendSubnet**, que creó en el paso anterior. |
    | Asignación de dirección IP | seleccione **Dinámico**. |
    | Zona de disponibilidad | Seleccione **Con redundancia de zona**. |

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   
    
    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/create-standard-internal-load-balancer.png" alt-text="Creación de un equilibrador de carga interno estándar" border="true":::
 
## <a name="create-load-balancer-resources"></a>Creación de recursos del equilibrador de carga

En esta sección, va a configurar:

* Las opciones del equilibrador de carga para un grupo de direcciones de back-end.
* Un sondeo de estado.
* Una regla de equilibrador de carga.

### <a name="create-a-backend-pool"></a>Creación de un grupo de back-end

Un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red virtuales conectadas al equilibrador de carga. 

Cree el grupo de direcciones de back-end **myBackendPool** para incluir máquinas virtuales para el tráfico de Internet de equilibrio de carga.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.

3. En la página **Agregar un grupo back-end**, en nombre, escriba **myBackEndPool**, como el nombre del grupo de back-end y, a continuación, seleccione **Aceptar**.

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

El equilibrador de carga supervisa el estado de la aplicación con un sondeo de estado. 

El sondeo de estado agrega o quita las máquinas virtuales del equilibrador de carga a partir de su respuesta a las comprobaciones de estado. 

Cree un sondeo de mantenimiento llamado **myHealthProbe** para supervisar el mantenimiento de las máquinas virtuales.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Sondeos de estado** y, a continuación, seleccione **Agregar**.
    
    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myHealthProbe**. |
    | Protocolo | Seleccione **HTTP**. |
    | Port | Escriba **80**.|
    | Intervalo | Escriba **15** como número de **Intervalo**, en segundos, entre los intentos de sondeo. |
    | Umbral incorrecto | Seleccione **2** como número de **Umbral incorrecto** o errores de sondeo consecutivos que deben producirse para que una máquina virtual se considere que no funciona de manera correcta.|
    | | |

3. Deje el resto de valores predeterminados y seleccione **Aceptar**.

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración IP del front-end para el tráfico entrante y el grupo de direcciones IP de back-end para recibir el tráfico. Los puertos de origen y de destino se definen en la regla. 

En esta sección va a crear una regla de equilibrador de carga:

* Llamada **myHTTPRule**.
* En el front-end llamado **LoadBalancerFrontEnd**.
* A la escucha en el **puerto 80**.
* Dirige el tráfico con equilibrio de carga al back-end llamado **myBackendPool** en el **puerto 80**.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.

3. Use estos valores para configurar la regla de equilibrio de carga:
    
    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myHTTPRule**. |
    | Versión de la dirección IP | Seleccione **IPv4**. |
    | Dirección IP del front-end | Seleccione **LoadBalancerFrontEnd**. |
    | Protocolo | seleccione **TCP**. |
    | Port | Escriba **80**.|
    | Puerto back-end | Escriba **80**. |
    | Grupo back-end | Seleccione **MyBackendPool**.|
    | Sondeo de mantenimiento | Seleccione **myHealthProbe**. |
    | Creación de reglas de salida implícitas | así que seleccione **No**.

4. Deje el resto de valores predeterminados y después seleccione **Aceptar**.

## <a name="create-backend-servers"></a>Creación de servidores back-end

En esta sección:

* Creará dos máquinas virtuales para el grupo de back-end del equilibrador de carga.
* Instalará IIS en las máquinas virtuales para probar el equilibrador de carga.

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

En esta sección, creará dos máquinas virtuales (**myVM1** y **myVM2**) con una dirección IP pública estándar en dos zonas (**Zona 1** y **Zona 2**). 

Estas máquinas virtuales se agregan al grupo de back-end del equilibrador de carga que se creó anteriormente.

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Value                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroupLB**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVM1**. |
    | Region | Seleccione **Oeste de Europa**. |
    | Opciones de disponibilidad | Seleccione **Zonas de disponibilidad**. |
    | Zona de disponibilidad | Seleccione **1**. |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Dirección IP pública | Acepte el valor predeterminado de **myVM-ip**. </br> La dirección IP será automáticamente una dirección IP de la SKU estándar en la Zona 1. |
    | Grupo de seguridad de red de NIC | Seleccione **Avanzado**.|
    | Configuración del grupo de seguridad de red | Seleccione **Crear nuevo**. </br> En la página **Crear grupo de seguridad de red**, escriba **myNSG** en **Nombre**. </br> Seleccione **Aceptar**. |
    | **Equilibrio de carga**  |
    | ¿Quiere colocar esta máquina virtual como subyacente respecto a una solución de equilibrio de carga existente? | Seleccione **Sí**. |
    | **Configuración de equilibrio de carga** |
    | Opciones de equilibrio de carga | Seleccione **Equilibrio de carga de Azure**. |
    | Seleccionar un equilibrador de carga | Seleccione **myLoadBalancer**.  |
    | Seleccionar un grupo de back-end | Seleccione **MyBackendPool**. |

5. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**.

6. En la pestaña **Administración**, seleccione o escriba:
    
    | Configuración | Value |
    |-|-|
    | **Supervisión** |  |
    | Diagnósticos de arranque | Seleccione **Desactivado**. |
   
7. Seleccione **Revisar + crear**. 
  
8. Revise la configuración y, a continuación, seleccione **Crear**.

9. Siga los pasos 1 a 8 para crear otra máquina virtual con los siguientes valores, mientras que el resto de la configuración es la misma que la de la máquina virtual **myVM1**:

    | Configuración | VM 2|
    | ------- | ----- |
    | Nombre |  **myVM2** |
    | Zona de disponibilidad | **2** |
    | Grupo de seguridad de red | Seleccione el grupo **myNSG** existente.|


# <a name="option-2-create-a-internal-load-balancer-basic-sku"></a>[Opción 2: crear un equilibrador de carga interno (SKU Básica)](#tab/option-1-create-internal-load-balancer-basic)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción.  Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

En esta sección, va a crear un equilibrador de carga que equilibra la carga de las máquinas virtuales. 

Puede crear un equilibrador de carga público o interno. 

Cuando se crea un equilibrador de carga interno, se configura una red virtual como red para él. 

Una dirección IP privada de la red virtual se configura como front-end (denominado **LoadBalancerFrontend** de manera predeterminada) para el equilibrador de carga. 

La dirección IP de front-end puede ser **estática** o **dinámica**.

## <a name="virtual-network-and-parameters"></a>Red virtual y parámetros

En esta sección, reemplazará los parámetros de los pasos con la siguiente información:

| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLB |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Oeste de Europa      |
| **\<IPv4-address-space>**   | 10.1.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.1.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-load-balancer"></a>Creación de un equilibrador de carga

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **myResourceGroupLB**, que creó en el paso anterior.|
    | Nombre                   | Escriba **myLoadBalancer**.                                   |
    | Region         | Seleccione **Oeste de Europa**.                                        |
    | Tipo          | seleccione **Interno**.                                        |
    | SKU           | Seleccione **Básica**. |
    | Virtual network | Seleccione **myVNet**, que creó en el paso anterior. |
    | Subnet  | Seleccione **myBackendSubnet**, que creó en el paso anterior. |
    | Asignación de dirección IP | seleccione **Dinámico**. |

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/create-basic-internal-load-balancer.png" alt-text="Creación de un equilibrador de carga interno estándar" border="true":::

## <a name="create-load-balancer-resources"></a>Creación de recursos del equilibrador de carga

En esta sección, va a configurar:

* Las opciones del equilibrador de carga para un grupo de direcciones de back-end.
* Un sondeo de estado.
* Una regla de equilibrador de carga.

### <a name="create-a-backend-pool"></a>Creación de un grupo de back-end

Un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red virtuales conectadas al equilibrador de carga. 

Cree el grupo de direcciones de back-end **myBackendPool** para incluir máquinas virtuales para el tráfico de Internet de equilibrio de carga.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.

3. En la página **Agregar un grupo de back-end**, escriba o seleccione:
    
    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myBackendPool**. |
    | Virtual network | Seleccione **myVNet**. |
    | Asociado a | Seleccione **Máquinas virtuales**. |

4. Seleccione **Agregar**.

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

El equilibrador de carga supervisa el estado de la aplicación con un sondeo de estado. 

El sondeo de estado agrega o quita las máquinas virtuales del equilibrador de carga a partir de su respuesta a las comprobaciones de estado. 

Cree un sondeo de mantenimiento llamado **myHealthProbe** para supervisar el mantenimiento de las máquinas virtuales.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Sondeos de estado** y, a continuación, seleccione **Agregar**.
    
    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myHealthProbe**. |
    | Protocolo | Seleccione **HTTP**. |
    | Port | Escriba **80**.|
    | Path | Escriba **/** |
    | Intervalo | Escriba **15** como número de **Intervalo**, en segundos, entre los intentos de sondeo. |
    | Umbral incorrecto | Seleccione **2** como número de **Umbral incorrecto** o errores de sondeo consecutivos que deben producirse para que una máquina virtual se considere que no funciona de manera correcta.|

3. Seleccione **Aceptar**.

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración IP del front-end para el tráfico entrante y el grupo de direcciones IP de back-end para recibir el tráfico. Los puertos de origen y de destino se definen en la regla. 

En esta sección va a crear una regla de equilibrador de carga:

* Llamada **myHTTPRule**.
* En el front-end llamado **LoadBalancerFrontEnd**.
* A la escucha en el **puerto 80**.
* Dirige el tráfico con equilibrio de carga al back-end llamado **myBackendPool** en el **puerto 80**.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.

3. Use estos valores para configurar la regla de equilibrio de carga:
    
    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myHTTPRule**. |
    | Versión de la dirección IP | Seleccione **IPv4**. |
    | Dirección IP del front-end | Seleccione **LoadBalancerFrontEnd**. |
    | Protocolo | seleccione **TCP**. |
    | Port | Escriba **80**.|
    | Puerto back-end | Escriba **80**. |
    | Grupo back-end | Seleccione **MyBackendPool**.|
    | Sondeo de mantenimiento | Seleccione **myHealthProbe**. |
 
4. Deje el resto de valores predeterminados y después seleccione **Aceptar**.

## <a name="create-backend-servers"></a>Creación de servidores back-end

En esta sección:

* Creará dos máquinas virtuales para el grupo de back-end del equilibrador de carga.
* Creará un conjunto de disponibilidad para las máquinas virtuales.
* Instalará IIS en las máquinas virtuales para probar el equilibrador de carga.

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Las SKU de la dirección IP pública y las SKU de Load Balancer deben coincidir. Para la instancia de Load Balancer básico, utilice máquinas virtuales con direcciones IP básicas en el grupo de back-end. 

En esta sección, creará dos máquinas virtuales (**myVM1**, **myVM2**) con una dirección IP pública básica.  

Las dos máquinas virtuales se agregarán a un conjunto de disponibilidad llamado **myAvailabilitySet**.

Estas máquinas virtuales se agregan al grupo de back-end del equilibrador de carga que se creó anteriormente.

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Value                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroupLB**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVM1**. |
    | Region | Seleccione **Oeste de Europa**. |
    | Opciones de disponibilidad | Seleccione **Conjunto de disponibilidad**. |
    | Conjunto de disponibilidad | Seleccione **Crear nuevo**. </br> Escriba **myAvailabilitySet** en **Nombre**. </br> Seleccione **Aceptar**. |
    | Imagen | **Windows Server 2019 Datacenter** |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | Seleccione **myVNet**. |
    | Subnet | Seleccione **myBackendSubnet**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. </br> Escriba **myVM-ip** en el nombre. </br> Seleccione **Aceptar**. |
    | Grupo de seguridad de red de NIC | Seleccione **Avanzado**.|
    | Configuración del grupo de seguridad de red | Seleccione **Crear nuevo**. </br> En la página **Crear grupo de seguridad de red**, escriba **myNSG** en **Nombre**. </br> Seleccione **Aceptar**. |
    | **Equilibrio de carga**  |
    | ¿Quiere colocar esta máquina virtual como subyacente respecto a una solución de equilibrio de carga existente? | Seleccione **No**. |
 
5. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**.

6. En la pestaña **Administración**, seleccione o escriba:
    | Configuración | Value |
    |-|-|
    | **Supervisión** | |
    | Diagnósticos de arranque | Seleccione **Desactivado**. |

7. Seleccione **Revisar + crear**. 
  
8. Revise la configuración y, a continuación, seleccione **Crear**.

9. Siga los pasos 1 a 8 para crear otra máquina virtual con los siguientes valores, mientras que el resto de la configuración es la misma que la de la máquina virtual **myVM1**:

    | Configuración | VM 2 |
    | ------- | ----- |
    | Nombre |  **myVM2** |
    | Conjunto de disponibilidad| Seleccione **myAvailabilitySet**. |
    | Grupo de seguridad de red | Seleccione el grupo **myNSG** existente.|

### <a name="add-virtual-machines-to-the-backend-pool"></a>Adición de máquinas virtuales al grupo de back-end

Las máquinas virtuales creadas en los pasos anteriores se deben agregar al grupo de back-end de **myLoadBalancer**.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Grupos de back-end** y, después, seleccione **myBackendPool**.

3. Seleccione **Máquinas virtuales** en **Asociado a**.

4. En la sección **Máquinas virtuales**, seleccione **+ Agregar**.

5. Active las casillas situadas junto a **myVM1** y **myVM2**.

6. Seleccione **Agregar**.

7. Seleccione **Guardar**.
---

## <a name="create-test-virtual-machine"></a>Creación de una máquina virtual de prueba

En esta sección va a crear una máquina virtual denominada **myTestVM**.  Esta máquina virtual se usará para probar la configuración del equilibrador de carga.

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Value                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroupLB**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myTestVM** |
    | Region | Seleccione **Oeste de Europa**. |
    | Opciones de disponibilidad | Seleccione **No se requiere redundancia de la infraestructura** |
    | Zona de disponibilidad | Seleccione **Redundancia de zona** |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Dirección IP pública | Acepte el valor predeterminado de **myTestVM-ip**. |
    | Grupo de seguridad de red de NIC | Seleccione **Avanzado**.|
    | Configuración del grupo de seguridad de red | Seleccione **MyNSG**, que creó en el paso anterior.|
    
5. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**.

6. En la pestaña **Administración**, seleccione o escriba:
    
    | Configuración | Value |
    |-|-|
    | **Supervisión** |  |
    | Diagnósticos de arranque | Seleccione **Desactivado**. |
   
7. Seleccione **Revisar + crear**. 
  
8. Revise la configuración y, a continuación, seleccione **Crear**.

## <a name="install-iis"></a>Instalación de IIS

1. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, en la lista de recursos, seleccione **myVM1**, que se encuentra en el grupo de recursos **myResourceGroupLB**.

2. En la página **Información general**, seleccione **Conectar** para descargar el archivo RDP de la máquina virtual.

3. Abra el archivo RDP.

4. Inicie sesión en la máquina virtual con las credenciales que proporcionó durante la creación de esta máquina virtual.

5. En el escritorio del servidor, vaya a **Herramientas administrativas de Windows**>**Windows PowerShell**.

6. Ejecute los siguientes comandos en la ventana de PowerShell para:

    * Instalar el servidor IIS
    * Eliminar el archivo predeterminado iisstart.htm
    * Agregue un nuevo archivo iisstart.htm que muestre el nombre de la máquina virtual:

   ```powershell
    
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # remove default htm file
    remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add a new htm file that displays server name
    Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
   ```
7. Cierre la sesión de RDP con **myVM1**.

8. Repita los pasos 1 a 6 para instalar IIS y el archivo iisstart.htm actualizado en **myVM2**.

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

1. Busque la dirección IP privada del equilibrador de carga en la pantalla **Información general**. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, **myLoadBalancer**.

2. Tome nota o copie la dirección que encontrará junto a **Dirección IP privada**, en la pestaña **Información general** de **myLoadBalancer**.

3. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, en la lista de recursos, seleccione **myTestVM**, que se encuentra en el grupo de recursos **myResourceGroupLB**.

4. En la página **Información general**, seleccione **Conectar** para descargar el archivo RDP de la máquina virtual.

5. Abra el archivo RDP.

6. Inicie sesión en la máquina virtual con las credenciales que proporcionó durante la creación de esta máquina virtual.

7. Abra **Internet Explorer** en **myTestVM**.

4. Copie la dirección IP privada y péguela en la barra de direcciones del explorador. La página predeterminada del servidor web de IIS se muestra en el explorador.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="Crear un equilibrador de carga interno estándar" border="true":::
   
Para ver el tráfico distribuido por Load Balancer entre las tres máquinas virtuales, puede personalizar la página predeterminada de cada servidor web IIS de las máquinas virtuales y luego forzar una actualización del explorador web desde el equipo cliente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, la instancia de Load Balancer y todos los recursos relacionados. Para ello, seleccione el grupo de recursos **myResourceGroupLB**, que contiene los recursos y, a continuación, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido:

* Ha creado un equilibrador de carga Básico o Estándar en Azure.
* Ha conectado dos máquinas virtuales al equilibrador de carga.
* Ha configurado la regla de tráfico del equilibrador de carga, el sondeo de estado y, a continuación, ha probado el equilibrador de carga. 

Para más información sobre Azure Load Balancer, consulte [¿Qué es Azure Load Balancer?](load-balancer-overview.md) y [Preguntas frecuentes de Load Balancer](load-balancer-faqs.md).

Más información sobre [Load Balancer y zonas de disponibilidad](load-balancer-standard-availability-zones.md).
