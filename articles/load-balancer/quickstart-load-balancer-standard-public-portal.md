---
title: 'Inicio rápido: Creación de una instancia pública de Load Balancer: Azure Portal'
titleSuffix: Azure Load Balancer
description: En esta guía de inicio rápido se muestra cómo crear una instancia de Load Balancer mediante Azure Portal.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a load balancer so that I can load balance internet traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/17/2020
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 262c12b1fb1d5c768b178d6e56c2964527f34495
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88607219"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-the-azure-portal"></a>Inicio rápido: Uso de Azure Portal para crear un equilibrador de carga público para equilibrar la carga de máquinas virtuales

Comience a usar Azure Load Balancer mediante Azure Portal para crear un equilibrador de carga público y tres máquinas virtuales.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

---

# <a name="standard-sku"></a>[**SKU Estándar**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción.  Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

En esta sección, va a crear un equilibrador de carga que equilibra la carga de las máquinas virtuales. 

Cuando se crea una instancia pública de Load Balancer, también se debe crear una nueva dirección IP pública que se configura como front-end (llamada **LoadBalancerFrontend** de forma predeterminada) para dicha instancia.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba **MyResourceGroupLB** en el cuadro de texto.|
    | Nombre                   | Escriba **myLoadBalancer**.                                   |
    | Region         | Seleccione **Oeste de Europa**.                                        |
    | Tipo          | Seleccione **Público**.                                        |
    | SKU           | Seleccione **Estándar**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. Si tiene una dirección IP pública que le gustaría usar, seleccione **Utilizar existente**. |
    | Nombre de la dirección IP pública | Escriba **myPublicIP** en el cuadro de texto.|
    | Zona de disponibilidad | Seleccione **Con redundancia de zona** para crear un equilibrador de carga resistente. Para crear una instancia de Load Balancer de zona, seleccione una zona específica entre 1, 2 o 3. |
    | Adición de una dirección IPv6 pública | así que seleccione **No**. </br> Para más información sobre las direcciones IPv6 y el equilibrador de carga, consulte [¿Qué es IPv6 para Azure Virtual Network?](https://docs.microsoft.com/azure/virtual-network/ipv6-overview)  |

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   
    
    :::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/create-standard-load-balancer.png" alt-text="Creación de un equilibrador de carga estándar" border="true":::
 
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

* Cree una red virtual.
* Creará tres máquinas virtuales para el grupo de back-end del equilibrador de carga.
* Instalará IIS en las máquinas virtuales para probar el equilibrador de carga.

## <a name="create-the-virtual-network"></a>Crear la red virtual

En esta sección, creará una red virtual y una subred.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso > Redes > Red virtual** o busque **Red virtual** en el cuadro de búsqueda.

2. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | **Configuración**          | **Valor**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Detalles del proyecto**  |                                                                 |
    | Suscripción     | Selección de su suscripción a Azure                                  |
    | Grupo de recursos   | Seleccione **myResourceGroupLB**. |
    | **Detalles de instancia** |                                                                 |
    | Nombre             | Escriba **myVNet**.                                    |
    | Region           | Seleccione **Oeste de Europa**. |

3. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

4. En la pestaña **Direcciones IP**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Espacio de direcciones IPv4 | Escriba **10.1.0.0/16**. |

5. En **Nombre de subred**, seleccione la palabra **predeterminada**.

6. En **Editar subred**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre de subred | Escriba **myBackendSubnet**. |
    | Intervalo de direcciones de subred | Escriba **10.1.0.0/24**. |

7. Seleccione **Guardar**.

8. Seleccione la pestaña **Seguridad** .

9. En **BastionHost**, seleccione **Habilitar**. Escriba esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre del bastión | Escriba **myBastionHost**. |
    | Espacio de direcciones de AzureBastionSubnet | Escriba **10.1.1.0/24**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. </br> En **Nombre**, escriba **myBastionIP**. </br> Seleccione **Aceptar**. |


8. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

9. Seleccione **Crear**.

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

En esta sección, creará tres máquinas virtuales (**myVM1**, **myVM2** y **myVM3**) en tres zonas diferentes (**Zona 1**, **Zona 2**y **Zona 3**). 

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
    | **Reglas de puerto de entrada** |  |
    | Puertos de entrada públicos | Seleccione **Ninguno**. |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-|-|
    | **Interfaz de red** |  |
    | Virtual network | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Avanzado**.|
    | Configuración del grupo de seguridad de red | Seleccione **Crear nuevo**. </br> En la página **Crear grupo de seguridad de red**, escriba **myNSG** en **Nombre**. </br> En **Reglas de entrada**, seleccione **+Agregar una regla de entrada**. </br> En **Intervalos de puertos de destino**, escriba **80**. </br> En **Prioridad**, escriba **100**. </br> En **Nombre**, escriba **myHTTPRule**. </br> Seleccione **Agregar**. </br> Seleccione **Aceptar**. |
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

9. Siga los pasos 1 a 8 para crear dos máquinas virtuales adicionales con los siguientes valores y todos los demás valores deben coincidir con los de **myVM1**:

    | Configuración | VM 2| VM 3|
    | ------- | ----- |---|
    | Nombre |  **myVM2** |**myVM3**|
    | Zona de disponibilidad | **2** |**3**|
    | Grupo de seguridad de red | Seleccione el grupo **myNSG** existente.| Seleccione el grupo **myNSG** existente.|

## <a name="create-outbound-rule-configuration"></a>Creación de la configuración de regla de salida
Las reglas de salida del equilibrador de carga configuran SNAT saliente para las máquinas virtuales del grupo de back-end. 

Para más información sobre las conexiones salientes, consulte [Conexiones salientes en Azure](load-balancer-outbound-connections.md).

### <a name="create-outbound-rule"></a>Creación de una regla de salida

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Reglas de salida** y, a continuación, seleccione **Agregar**.

3. Use estos valores para configurar las reglas de salida:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myOutboundRule**. |
    | Dirección IP del front-end | Seleccione **Crear nuevo**. </br> En **Nombre**, escriba **LoadBalancerFrontEndOutbound**. </br> Seleccione **Dirección IP** o **Prefijo IP**. </br> Seleccione **Crear nuevo** en **Dirección IP pública** o **Prefijo de dirección IP pública**. </br> En Nombre, escriba **myPublicIPOutbound** o **myPublicIPPrefixOutbound**. </br> Seleccione **Agregar**.|
    | Tiempo de espera de inactividad (minutos) | Mueva el control deslizante a **15 minutos**.|
    | Restablecimiento de TCP | Seleccione **Habilitado**.|
    | Grupo back-end | Seleccione **Crear nuevo**. </br> Escriba **myBackendPoolOutbound** en **Name**. </br> Seleccione **Agregar**. |
    | Asignación de puertos: > Asignación de puertos | Seleccione **Elegir manualmente el número de puertos de salida**. |
    | Puertos de salida: > Elegir por | Seleccione **Puertos por instancia**. |
    | Puertos de salida -> Puertos por instancia | Escriba **10000**. |

4. Seleccione **Agregar**.

### <a name="add-virtual-machines-to-outbound-pool"></a>Adición de máquinas virtuales al grupo de salida

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Grupos de back-end**.

3. Seleccione **myBackendPoolOutbound**.

4. En **Red virtual**, seleccione **myVNet**.

5. En **Máquinas virtuales**, seleccione **+ Agregar**.

6. Active las casillas situadas junto a **myVM1**, **myVM2** y **myVM3**. 

7. Seleccione **Agregar**.

8. Seleccione **Guardar**.

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción.  Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

En esta sección, va a crear un equilibrador de carga que equilibra la carga de las máquinas virtuales. 

Cuando se crea una instancia pública de Load Balancer, también se debe crear una nueva dirección IP pública que se configura como front-end (llamada **LoadBalancerFrontend** de forma predeterminada) para dicha instancia.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba **myResourceGroupLB** en el cuadro de texto.|
    | Nombre                   | Escriba **myLoadBalancer**.                                   |
    | Region         | Seleccione **Oeste de Europa**.                                        |
    | Tipo          | Seleccione **Público**.                                        |
    | SKU           | Seleccione **Básica**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. Si tiene una dirección IP pública que le gustaría usar, seleccione **Utilizar existente**. |
    | Nombre de la dirección IP pública | Escriba **myPublicIP** en el cuadro de texto.|
    | Asignación | Seleccione **Dinámica**. |
    | Adición de una dirección IPv6 pública | así que seleccione **No**. </br> Para más información sobre las direcciones IPv6 y el equilibrador de carga, consulte [¿Qué es IPv6 para Azure Virtual Network?](https://docs.microsoft.com/azure/virtual-network/ipv6-overview)  |

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   

    :::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/create-basic-load-balancer.png" alt-text="Creación de una instancia de Load Balancer Básico" border="true":::

## <a name="create-load-balancer-resources"></a>Creación de recursos del equilibrador de carga

En esta sección, va a configurar:

* Cree una red virtual.
* Las opciones del equilibrador de carga para un grupo de direcciones de back-end.
* Un sondeo de estado.
* Una regla de equilibrador de carga.

## <a name="create-the-virtual-network"></a>Crear la red virtual

En esta sección, creará una red virtual y una subred.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso > Redes > Red virtual** o busque **Red virtual** en el cuadro de búsqueda.

2. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | **Configuración**          | **Valor**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Detalles del proyecto**  |                                                                 |
    | Suscripción     | Selección de su suscripción a Azure                                  |
    | Grupo de recursos   | Seleccione **myResourceGroupLB**. |
    | **Detalles de instancia** |                                                                 |
    | Nombre             | Escriba **myVNet**.                                    |
    | Region           | Seleccione **Oeste de Europa**. |

3. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

4. En la pestaña **Direcciones IP**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Espacio de direcciones IPv4 | Escriba **10.1.0.0/16**. |

5. En **Nombre de subred**, seleccione la palabra **predeterminada**.

6. En **Editar subred**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre de subred | Escriba **myBackendSubnet**. |
    | Intervalo de direcciones de subred | Escriba **10.1.0.0/24**. |

7. Seleccione **Guardar**.

8. Seleccione la pestaña **Seguridad** .

9. En **BastionHost**, seleccione **Habilitar**. Escriba esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre del bastión | Escriba **myBastionHost**. |
    | Espacio de direcciones de AzureBastionSubnet | Escriba **10.1.1.0/24**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. </br> En **Nombre**, escriba **myBastionIP**. </br> Seleccione **Aceptar**. |


8. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

9. Seleccione **Crear**.
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

* Creará tres máquinas virtuales para el grupo de back-end del equilibrador de carga.
* Creará un conjunto de disponibilidad para las máquinas virtuales.
* Instalará IIS en las máquinas virtuales para probar el equilibrador de carga.

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

En esta sección, creará tres máquinas virtuales (**myVM1**, **myVM2**y **myVM3**) con una dirección IP pública básica.  

Las tres máquinas virtuales se agregarán a un conjunto de disponibilidad llamado **myAvailabilitySet**.

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
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Avanzado**.|
    | Configuración del grupo de seguridad de red | Seleccione **Crear nuevo**. </br> En la página **Crear grupo de seguridad de red**, escriba **myNSG** en **Nombre**. </br> En **Reglas de entrada**, seleccione **+Agregar una regla de entrada**. </br> En **Intervalos de puertos de destino**, escriba **80**. </br> En **Prioridad**, escriba **100**. </br> En **Nombre**, escriba **myHTTPRule**. </br> Seleccione **Agregar**. </br> Seleccione **Aceptar**. |
    | **Equilibrio de carga**  |
    | ¿Quiere colocar esta máquina virtual como subyacente respecto a una solución de equilibrio de carga existente? | Seleccione **No**. |
 
5. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**.

6. En la pestaña **Administración**, seleccione o escriba:
    
    | Configuración | Value |
    |---|---|
    | **Supervisión** | |
    | Diagnósticos de arranque | Seleccione **Desactivado**. |

7. Seleccione **Revisar + crear**. 
  
8. Revise la configuración y, a continuación, seleccione **Crear**.

9. Siga los pasos 1 a 8 para crear dos máquinas virtuales adicionales con los siguientes valores y todos los demás valores deben coincidir con los de **myVM1**:

    | Configuración | VM 2| VM 3|
    | ------- | ----- |---|
    | Nombre |  **myVM2** |**myVM3**|
    | Conjunto de disponibilidad| Seleccione **myAvailabilitySet**. | Seleccione **myAvailabilitySet**.|
    | Grupo de seguridad de red | Seleccione el grupo **myNSG** existente.| Seleccione el grupo **myNSG** existente.|

### <a name="add-virtual-machines-to-the-backend-pool"></a>Adición de máquinas virtuales al grupo de back-end

Las máquinas virtuales creadas en los pasos anteriores se deben agregar al grupo de back-end de **myLoadBalancer**.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.

2. En **Configuración**, seleccione **Grupos de back-end** y, después, seleccione **myBackendPool**.

3. Seleccione **Máquinas virtuales** en **Asociado a**.

4. En la sección **Máquinas virtuales**, seleccione **+ Agregar**.

5. Active las casillas que se encuentran al lado de **myVM1**, **myVM2** y **myVM3**.

6. Seleccione **Agregar**.

7. Seleccione **Guardar**.

---

## <a name="install-iis"></a>Instalación de IIS

1. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, en la lista de recursos, seleccione **myVM1**, que se encuentra en el grupo de recursos **myResourceGroupLB**.

2. En la página **Introducción**, seleccione **Conectar** y después **Instancia de Bastion**.

4. Escriba el nombre de usuario y la contraseña especificados durante la creación de la máquina virtual.

5. Seleccione **Conectar**.

6. En el escritorio del servidor, vaya a **Herramientas administrativas de Windows** > **Windows PowerShell**.

7. Ejecute los siguientes comandos en la ventana de PowerShell para:

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
8. Cierre la sesión de Bastion con **myVM1**.

9. Repita los pasos 1 a 6 para instalar IIS y el archivo iisstart.htm actualizado en **myVM2** y **myVM3**.

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

1. Busque la dirección IP pública de Load Balancer en la pantalla **Información general**. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, **myPublicIP**.

2. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. La página predeterminada del servidor web de IIS se muestra en el explorador.

   ![Servidor web de IIS](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)

Para ver el tráfico distribuido por Load Balancer entre las tres máquinas virtuales, puede personalizar la página predeterminada de cada servidor web IIS de las máquinas virtuales y luego forzar una actualización del explorador web desde el equipo cliente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, la instancia de Load Balancer y todos los recursos relacionados. Para ello, seleccione el grupo de recursos **myResourceGroupLB**, que contiene los recursos y, a continuación, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido:

* Ha creado una instancia Estándar o Básica de Azure Load Balancer.
* Ha conectado 3 máquinas virtuales al equilibrador de carga.
* Ha configurado la regla de tráfico del equilibrador de carga, el sondeo de estado y, a continuación, ha probado el equilibrador de carga. 

Para más información sobre Azure Load Balancer, consulte [¿Qué es Azure Load Balancer?](load-balancer-overview.md) y [Preguntas frecuentes de Load Balancer](load-balancer-faqs.md).

Más información sobre [Load Balancer y zonas de disponibilidad](load-balancer-standard-availability-zones.md).
