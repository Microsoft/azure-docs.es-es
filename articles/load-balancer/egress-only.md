---
title: Configuración del equilibrador de carga solo de salida
titleSuffix: Azure Load Balancer
description: En este artículo, aprenderá a crear un equilibrador de carga interno con NAT de salida.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/07/2020
ms.author: allensu
ms.openlocfilehash: b44f626546b313299701687157b37b7df021bd61
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88038145"
---
# <a name="outbound-only-load-balancer-configuration"></a>Configuración del equilibrador de carga solo de salida

Use una combinación de equilibradores de carga estándar internos y externos para crear conectividad de salida para las máquinas virtuales situadas detrás de un equilibrador de carga interno. 

Esta configuración proporciona NAT de salida en los escenarios de equilibradores de carga internos, lo que produce una configuración de "solo salida" para el grupo de back-end.


<p align="center">
  <img src="./media/egress-only/load-balancer-egress-only.png" alt="Figure depicts a egress only load balancer configuration." width="500" title="Virtual Network NAT">
</p>


<!-- 
:::image type="content" source="./media/egress-only/load-balancer-egress-only.png" alt-text="Figure depicts a egress only load balancer configuration" border="true":::
-->


*Ilustración: Configuración de un equilibrador de carga solo de salida*

Esto son los pasos que se deben realizar:

1. Crear una red virtual con un host bastión.
2. Crear una máquina virtual solo con una dirección IP privada.
3. Crear equilibradores de carga estándar internos y públicos.
4. Agregar grupos de back-end a ambos equilibradores de carga y colocar la máquina virtual en cada grupo.
5. Conectarse a la máquina virtual a través del host bastión y:
    1. Probar la conectividad de salida. 
    2. Configurar una regla de salida en el equilibrador de carga público.
    3. Probar de nuevo la conectividad de salida.

## <a name="create-virtual-network-and-virtual-machine"></a>Creación de la red virtual y la máquina virtual

Cree una red virtual con dos subredes:

* Subred principal
* Subred bastión

Cree una máquina virtual en la nueva red virtual.

### <a name="create-the-virtual-network"></a>Crear la red virtual

1. [Inicie sesión](https://portal.azure.com) en Azure Portal.

2. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso > Redes > Red virtual** o busque **Red virtual** en el cuadro de búsqueda.

2. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | **Configuración**          | **Valor**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Detalles del proyecto**  |                                                                 |
    | Suscripción     | Selección de su suscripción a Azure                                  |
    | Grupo de recursos   | Seleccione **Crear nuevo**. </br> Escriba **myResourceGroupLB**. </br> Seleccione **Aceptar**. |
    | **Detalles de instancia** |                                                                 |
    | Nombre             | Escriba **myVNet**.                                    |
    | Region           | Seleccione **Este de EE. UU. 2**. |

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

### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**. 
   
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    | Configuración | Value                                          |
    |-----------------------|----------------------------------|
    | **Detalles del proyecto** |  |
    | Suscripción | Selección de su suscripción a Azure |
    | Grupo de recursos | Seleccione **myResourceGroupLB**. |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVM**. |
    | Region | Seleccione **Este de EE. UU. 2**. |
    | Opciones de disponibilidad | Seleccione **No se requiere redundancia de la infraestructura** |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Instancia de Azure Spot | Seleccione **No**. |
    | Size | Elija el tamaño de la máquina virtual o acepte la configuración predeterminada. |
    | **Cuenta de administrador** |  |
    | Nombre de usuario | Escriba un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmar contraseña | Vuelva a escribir la contraseña. |
    | **Reglas de puerto de entrada** |  |
    | Puertos de entrada públicos | Seleccione **Allow selected ports** (Permitir puertos seleccionados). |
    | Selección de puertos de entrada | Seleccione **RDP (3389)** . |

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
  
4. En la pestaña Redes, seleccione o escriba:

    | Configuración | Value |
    |-----|------------|
    | **Interfaz de red** |  |
    | Virtual network | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Dirección IP pública | Seleccione **Ninguno**. |
    | Grupo de seguridad de red de NIC | Seleccione **Ninguno**.|
    | ¿Quiere colocar esta máquina virtual detrás de una solución de equilibrio de carga existente? | Seleccione **No**. |
   
5. Seleccione la pestaña **Administración** o seleccione **Siguiente** > **Administración**.

6. En la pestaña **Administración**, seleccione o escriba:
    
    | Configuración | Value |
    |-|-|
    | **Supervisión** |  |
    | Diagnósticos de arranque | Seleccione **Desactivado**. |
   
7. Seleccione **Revisar + crear**. 
  
8. Revise la configuración y, a continuación, seleccione **Crear**.

## <a name="create-load-balancers-and-test-connectivity"></a>Creación de equilibradores de carga y prueba de la conectividad

Use Azure Portal para crear los siguientes elementos:

* Equilibrador de carga interno
* Equilibrador de carga público
 
Agregue la máquina virtual creada al grupo de back-end de cada uno.  A continuación, realizará una configuración para permitir solo la conectividad saliente desde la máquina virtual, que probará antes y después.

### <a name="create-internal-load-balancer"></a>Creación del equilibrador de carga interno

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Suscripción               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **myResourceGroupLB**, que creó en el paso anterior.|
    | Nombre                   | Escriba **myInternalLoadBalancer**.                                   |
    | Region         | Seleccione **Este de EE. UU. 2**.                                        |
    | Tipo          | seleccione **Interno**.                                        |
    | SKU           | Seleccione **Estándar**. |
    | Virtual network | Seleccione **myVNet**, que creó en el paso anterior. |
    | Subnet  | Seleccione **myBackendSubnet**, que creó en el paso anterior. |
    | Asignación de dirección IP | seleccione **Dinámico**. |

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   

### <a name="create-public-load-balancer"></a>Creación de un equilibrador de carga público

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Suscripción               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba **MyResourceGroupLB** en el cuadro de texto.|
    | Nombre                   | Escriba **myPublicLoadBalancer**.                                   |
    | Region         | Seleccione **Este de EE. UU. 2**.                                        |
    | Tipo          | Seleccione **Público**.                                        |
    | SKU           | Seleccione **Estándar**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. |
    | Nombre de la dirección IP pública | Escriba **myFrontendIP** en el cuadro de texto.|
    | Zona de disponibilidad | Seleccione **Con redundancia de zona**. |
    | Adición de una dirección IPv6 pública | así que seleccione **No**. |

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   
   
### <a name="create-internal-backend-address-pool"></a>Creación de un grupo de direcciones de back-end interno

Cree el grupo de direcciones de back-end **myInternalBackendPool**:

1. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, después, seleccione **myLoadBalancer** en la lista de recursos.

2. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.

3. En la página **Add a backend pool** (Agregar un grupo de back-end), escriba **myBackEndPool** como nombre del grupo de back-end.
 
4. En **Red virtual**, seleccione **myVNet**.

5. En **Máquinas virtuales**, seleccione **Agregar** y elija **myVM**.
 
6. Seleccione **Agregar**.

### <a name="create-public-backend-address-pool"></a>Creación de un grupo de direcciones de back-end público

Cree el grupo de direcciones de back-end **myPublicBackendPool**:

1. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, después, seleccione **myPublicLoadBalancer** en la lista de recursos.

2. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.

3. En la página **Add a backend pool** (Agregar un grupo de back-end), escriba **myPublicBackEndPool** como nombre del grupo de back-end.

4. En **Red virtual**, seleccione **myVNet**.
 
5. En **Máquinas virtuales**, seleccione **Agregar** y elija **myVM**.
 
6. Seleccione **Agregar**.

### <a name="test-connectivity-before-outbound-rule"></a>Prueba de la conectividad antes de la regla de salida

1. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, en la lista de recursos, seleccione **myVM**, que se encuentra en el grupo de recursos **myResourceGroupLB**.

2. En la página **Información general**, seleccione **Conectar** y, luego, **Bastion**.

4. Escriba el nombre de usuario y la contraseña especificados durante la creación de la máquina virtual.

5. Seleccione **Conectar**.

6. Abra Internet Explorer.

7. Escriba **https://whatsmyip.org** en la barra de direcciones.

8. Se producirá un error en la conexión. De forma predeterminada, el equilibrador de carga público estándar [no permite el tráfico saliente sin una regla de salida definida](load-balancer-overview.md#securebydefault).
 
### <a name="create-a-public-load-balancer-outbound-rule"></a>Creación de una regla de salida del equilibrador de carga público

1. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, después, seleccione **myPublicLoadBalancer** en la lista de recursos.

2. En **Configuración**, seleccione **Reglas de salida** y, a continuación, seleccione **Agregar**.

3. Use estos valores para configurar las reglas de salida:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myOutboundRule**. |
    | Dirección IP del front-end | Seleccione **LoadBalancerFrontEnd**.|
    | Tiempo de espera de inactividad (minutos) | Mueva el control deslizante a **15 minutos**.|
    | Restablecimiento de TCP | Seleccione **Habilitado**.|
    | Grupo back-end | Seleccione **myPublicBackendPool**.| |
    | Asignación de puertos: > Asignación de puertos | Seleccione **Use the default number of outbound ports** (Usar el número predeterminado de puertos de salida). |

4. Seleccione **Agregar**.

### <a name="test-connectivity-after-outbound-rule"></a>Prueba de la conectividad después de la regla de salida

1. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, en la lista de recursos, seleccione **myVM**, que se encuentra en el grupo de recursos **myResourceGroupLB**.

2. En la página **Información general**, seleccione **Conectar** y, luego, **Bastion**.

4. Escriba el nombre de usuario y la contraseña especificados durante la creación de la máquina virtual.

5. Seleccione **Conectar**.

6. Abra Internet Explorer.

7. Escriba **https://whatsmyip.org** en la barra de direcciones.

8. La conexión funcionará correctamente.

9. La dirección IP mostrada debe ser la dirección IP de front-end de **myPublicLoadBalancer**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, los equilibradores de carga, la máquina virtual y todos los recursos relacionados. 

Para ello, seleccione el grupo de recursos **myResourceGroupLB** y, luego, elija **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se ha creado una configuración de "solo salida" con una combinación de equilibradores de carga públicos e internos.  

Esta configuración permite equilibrar la carga del tráfico interno entrante en el grupo de back-end y, al mismo tiempo, evitar las conexiones entrantes públicas.

- Más información sobre [Azure Load Balancer](load-balancer-overview.md).
- Aprenda sobre las [conexiones salientes en Azure](load-balancer-outbound-connections.md).
- [Preguntas frecuentes sobre Load Balancer](load-balancer-faqs.md).
- Más información sobre [Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview).
