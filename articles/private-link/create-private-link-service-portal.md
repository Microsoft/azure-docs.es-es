---
title: 'Inicio rápido: Creación de un servicio Private Link mediante Azure Portal'
titlesuffix: Azure Private Link
description: En este inicio rápido, aprenderá a crear un servicio Private Link desde Azure Portal
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: quickstart
ms.date: 02/03/2020
ms.author: allensu
ms.openlocfilehash: 0d873401d377a03581a319769604f3d976f365be
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927245"
---
# <a name="quickstart-create-a-private-link-service-by-using-the-azure-portal"></a>Inicio rápido: Creación de un servicio Private Link mediante Azure Portal

El servicio Azure Private Link hace referencia a su propio servicio que se gestiona mediante Private Link. Puede asignar a Private Link acceso al servicio o al recurso que opera detrás de Azure Standard Load Balancer. Los consumidores de su servicio pueden acceder a él de forma privada desde sus propias redes virtuales. En este inicio rápido aprenderá a crear un servicio Private Link desde Azure Portal.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-an-internal-load-balancer"></a>Creación de un conjunto de equilibrador de carga interno

En primer lugar, cree una red virtual. Luego, cree un equilibrador de carga interno para usarlo con el servicio Private Link.

## <a name="virtual-network-and-parameters"></a>Red virtual y parámetros

En esta sección, creará una red virtual. También puede crear la subred que va a hospedar el equilibrador de carga que accede a su servicio Private Link.

En los pasos de esta sección, tendrá que reemplazar los siguientes parámetros por la información a continuación:

| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLB |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Este de EE. UU. 2      |
| **\<IPv4-address-space>**   | 10.3.0.0/16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.3.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

### <a name="create-a-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

Use el portal para crear un equilibrador de carga interno estándar. El nombre y la dirección IP que especifique se configuran automáticamente como front-end del equilibrador de carga.

1. En el lado superior izquierdo del portal, seleccione **Crear un recurso** > **Redes** > **Load Balancer**.

1. En la pestaña **Datos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | **Suscripción**               | Seleccione su suscripción.    |
    | **Grupos de recursos**         | Seleccione **myResourceGroupLB** en el cuadro.|
    | **Nombre**                   | Escriba **myLoadBalancer**.                                   |
    | **Región**         | Seleccione **Este de EE. UU. 2**.                                        |
    | **Tipo**          | seleccione **Interno**.                                        |
    | **SKU**           | Seleccione **Estándar**.                          |
    | **Red virtual**           | Seleccione **myVNet**.                          |
    | **Asignación de dirección IP**              | Seleccione **Estática**.   |
    | **Dirección IP privada**|Escriba una dirección que esté en el espacio de direcciones de la red virtual y la subred. Por ejemplo, 10.3.0.7.  |

1. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**

1. En la pestaña **Revisar y crear**, seleccione **Crear**.

### <a name="create-standard-load-balancer-resources"></a>Creación de recursos del equilibrador de carga estándar

En esta sección se configuran los valores del equilibrador de carga para un grupo de direcciones de back-end y un sondeo de mantenimiento. También se especifican las reglas del equilibrador de carga.

#### <a name="create-a-back-end-pool"></a>Creación de un grupo de back-end

Un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red conectadas al equilibrador de carga. Este grupo le permite distribuir el tráfico a los recursos. Cree el grupo de direcciones de back-end llamado **myBackendPool** para incluir recursos que equilibran la carga del tráfico.

1. Seleccione **Todos los servicios** en el menú de la izquierda.
1. Seleccione **Todos los recursos** y, en la lista de recursos, seleccione **myLoadBalancer**.
1. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.
1. En la página **Agregar grupo back-end**, escriba **myBackendPool** como nombre del grupo de back-end y, después, seleccione **Agregar**.

#### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

Use un sondeo de estado para que el equilibrador de carga supervise el estado de los recursos. En función de la respuesta de los recursos a las comprobaciones del estado, el sondeo de estado agrega o quita recursos de forma dinámica en la rotación del equilibrador de carga.

Para crear un sondeo de estado para supervisar el estado de los recursos:

1. Seleccione **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, seleccione **myLoadBalancer**.

1. En **Configuración**, seleccione **Sondeos de mantenimiento** y, a continuación, seleccione **Agregar**.

1. En la página **Agregar sondeo de estado**, escriba o seleccione los siguientes valores:

   - **Name**: Escriba **myHealthProbe**.
   - **Protocolo**: seleccione **TCP**.
   - **Puerto**: Escriba **80**.
   - **Intervalo**: Escriba **15**. Este valor es el número de segundos entre los intentos de sondeo.
   - **Umbral incorrecto**: Escriba **2**. Este valor es el número de errores de sondeo consecutivos que tienen que producirse para que se considere que una máquina virtual no funciona correctamente.

1. Seleccione **Aceptar**.

#### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla del equilibrador de carga define cómo se distribuye el tráfico a los recursos. La regla define:

- La configuración de IP del front-end para el tráfico entrante.
- El grupo de IP de back-end para recibir el tráfico.
- Los puertos de origen y de destino requeridos.

La regla del equilibrador de carga llamada **myLoadBalancerRule** escucha el puerto 80 en el front-end **LoadBalancerFrontEnd**. La regla envía el tráfico de red al grupo de direcciones de back-end **myBackendPool** a través del mismo puerto 80.

Para crear una regla del equilibrador de carga:

1. Seleccione **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, seleccione **myLoadBalancer**.

1. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.

1. En la página **Agregar regla de equilibrio de carga**, escriba o seleccione los valores siguientes, en caso de que no estén presentes aún:

   - **Name**: escriba **myLoadBalancerRule**.
   - **Dirección IP de front-end**: escriba **LoadBalancerFrontEnd**.
   - **Protocolo**: seleccione **TCP**.
   - **Puerto**: Escriba **80**.
   - **Puerto back-end**: Escriba **80**.
   - **Grupo de back-end**: Seleccione **MyBackendPool**.
   - **Sondeo de mantenimiento**: Seleccione **myHealthProbe**. 

1. Seleccione **Aceptar**.

## <a name="create-a-private-link-service"></a>Creación de un servicio Private Link

En esta sección, creará un servicio Private Link detrás de un equilibrador de carga estándar.

1. En la parte superior izquierda de la página en Azure Portal, seleccione **Crear un recurso** > **Redes** > **Private Link Center (versión preliminar)** . También puede usar el cuadro de búsqueda del portal para buscar Private Link.

1. En **Private Link Center: Información general** > **Exponga su servicio propio para que otros usuarios puedan conectarse**, seleccione **Iniciar**.

1. En **Crear una instancia del servicio Private Link: Datos básicos**, escriba o seleccione esta información:

    | Configuración           | Value                                                                        |
    |-------------------|------------------------------------------------------------------------------|
    | Detalles del proyecto:  |                                                                              |
    | **Suscripción**      | Seleccione su suscripción.                                                     |
    | **Grupo de recursos**    | Seleccione **myResourceGroupLB**.                                                    |
    | Detalles de la instancia: |                                                                              |
    | **Nombre**              | Escriba **myPrivateLinkService**. |
    | **Región**            | Seleccione **Este de EE. UU. 2**.                                                        |

1. Seleccione **Siguiente: Configuración de salida**

1. En **Crear una instancia del servicio Private Link: Configuración de salida**, escriba o seleccione esta información:

    | Configuración                           | Value                                                                           |
    |-----------------------------------|---------------------------------------------------------------------------------|
    | **Equilibrador de carga**                     | Seleccione **myLoadBalancer**.                                                           |
    | **Dirección IP de front-end del equilibrador de carga** | Seleccione la dirección IP del front-end de **myLoadBalancer**.                                |
    | **Red virtual NAT de origen**        | Seleccione **myVNet**.                                                                   |
    | **Subred NAT de origen**                 | Seleccione **myBackendSubnet**.                                                          |
    | **Habilitar proxy TCP V2**               | Seleccione **Sí** o **No** en función de si la aplicación espera un encabezado proxy TCP versión 2. |
    | **Configuración de dirección IP privada**       | Configure el método de asignación y la dirección IP para cada dirección IP de NAT.                  |

1. Seleccione **Siguiente: Seguridad de acceso**

1. En **Crear una instancia del servicio Private Link: Seguridad de acceso**, seleccione **Visibilidad** y, después, elija **Solo control de acceso basado en rol**.
  
1. Seleccione **Siguiente: Etiquetas** > **Revisar y crear**, o bien elija la pestaña **Revisar y crear** en la parte superior de la página.

1. Revise la información y seleccione **Crear**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de usar el servicio Private Link, elimine el grupo de recursos para limpiar los recursos que se han usado en este inicio rápido.

1. Escriba **myResourceGroupLB** en el cuadro de búsqueda de la parte superior del portal y seleccione **myResourceGroupLB** en los resultados de la búsqueda.
1. Seleccione **Eliminar grupo de recursos**.
1. En **Escriba el nombre del grupo de recursos**, escriba **myResourceGroup**.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un equilibrador de carga interno de Azure y un servicio Private Link. También puede aprender a [crear un punto de conexión privado desde Azure Portal](https://docs.microsoft.com/azure/private-link/create-private-endpoint-portal).
