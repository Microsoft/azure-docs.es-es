---
title: Configuración de escuchas del grupo de disponibilidad y del equilibrador de carga (Azure Portal)
description: Instrucciones paso a paso para crear un agente de escucha en un grupo de disponibilidad AlwaysOn para SQL Server en Azure Virtual Machines
services: virtual-machines
documentationcenter: na
author: MashaMSFT
editor: monicar
ms.assetid: d1f291e9-9af2-41ba-9d29-9541e3adcfcf
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 02/16/2017
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 9cf6fa26cec0abbc52a990d71c1c2fcc5d6023e4
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88612561"
---
# <a name="configure-a-load-balancer-for-a-sql-server-always-on-availability-group-in-azure-virtual-machines"></a>Configuración de un equilibrador de carga para un grupo de disponibilidad de SQL Server Always On en Azure Virtual Machines

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


En este artículo se explica cómo crear un equilibrador de carga para un grupo de disponibilidad de SQL Server Always On en instancias de Azure Virtual Machines que se ejecutan con Azure Resource Manager. Cuando las instancias de SQL Server están implementadas en Azure Virtual Machines, los grupos de disponibilidad necesitan un equilibrador de carga. El equilibrador de carga almacena la dirección IP del agente de escucha del grupo de disponibilidad. Si un grupo de disponibilidad abarca varias regiones, cada región necesitará su propio equilibrador de carga.

Para completar esta tarea, debe tener un grupo de disponibilidad de SQL Server Always On implementado en máquinas virtuales de Azure que se ejecutan con Resource Manager. Las dos máquinas virtuales de SQL Server deben pertenecer al mismo conjunto de disponibilidad. Puede usar la [plantilla de Microsoft](availability-group-azure-marketplace-template-configure.md) para crear automáticamente el grupo de disponibilidad en Resource Manager. Esta plantilla crea automáticamente un equilibrador de carga interno. 

Si lo prefiere, puede [configurar manualmente un grupo de disponibilidad](availability-group-manually-configure-tutorial.md).

En este artículo, es necesario que los grupos de disponibilidad ya estén configurados.  

Artículos relacionados:

* [Configuración de Grupos de disponibilidad AlwaysOn en la máquina virtual de Azure (GUI)](availability-group-manually-configure-tutorial.md)   
* [Configuración de una conexión entre dos redes virtuales mediante Azure Resource Manager y PowerShell](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

A lo largo de este artículo, va a crear y configurar un equilibrador de carga en Azure Portal. Una vez hecho esto, configurará el clúster para que utilice la dirección IP del equilibrador de carga para el agente de escucha del grupo de disponibilidad.

## <a name="create-and-configure-the-load-balancer-in-the-azure-portal"></a>Creación y configuración del equilibrador de carga en el Portal de Azure

En esta parte de la tarea, realice los pasos siguientes:

1. En Azure Portal, cree el equilibrador de carga y configure la dirección IP.
2. Configure el grupo back-end.
3. Cree el sondeo. 
4. Establezca las reglas de equilibrio de carga.

> [!NOTE]
> Si las instancias de SQL Server se encuentran en varios grupos de recursos y regiones, realice cada paso dos veces, una vez en cada grupo de recursos.
> 

### <a name="step-1-create-the-load-balancer-and-configure-the-ip-address"></a>Paso 1: Creación del equilibrador de carga y configuración de la dirección IP

En primer lugar, cree el equilibrador de carga. 

1. En el Portal de Azure, abra el grupo de recursos que contiene las máquinas virtuales de SQL Server. 

2. En el grupo de recursos, seleccione **Agregar**.

3. Busque el **equilibrador de carga**. En los resultados de la búsqueda, seleccione **Load Balancer** (publicado por **Microsoft**).

4. En la hoja **Load Balancer**, haga clic en **Crear**.

5. En el cuadro de diálogo **Crear equilibrador de carga**, configure el equilibrador de carga tal y como se explica a continuación:

   | Configuración | Value |
   | --- | --- |
   | **Nombre** |Nombre de texto que representa el equilibrador de carga; Por ejemplo, **sqlLB**. |
   | **Tipo** |**Internas**: en la mayoría de las implementaciones se usa un equilibrador de carga interno que permite que las aplicaciones dentro de la misma red virtual se conecten al grupo de disponibilidad.  </br> **Externas**: permite que las aplicaciones se conecten al grupo de disponibilidad mediante una conexión a Internet pública. |
   | **SKU** |**Estándar**: Es obligatorio si las instancias de SQL se encuentran en un conjunto de disponibilidad diferente al del equilibrador de carga. </br> **Básico**: Opción predeterminada. |
   | **Red virtual** |Seleccione la red virtual en la que se encuentran las instancias de SQL Server. |
   | **Subred** |Seleccione la subred en la que se encuentran las instancias de SQL Server. |
   | **Asignación de dirección IP** |**Estática** |
   | **Dirección IP privada** |Especifique una dirección IP disponible de la subred. Utilícela cuando cree un agente de escucha en el clúster. Más adelante en este mismo artículo, hay un script de PowerShell en el que tendrá que utilizar esta dirección para la variable `$ILBIP`. |
   | **Suscripción** |Si tiene varias suscripciones, puede aparecer este campo. Seleccione la suscripción que quiere asociar con este recurso. Normalmente, será la misma suscripción que se utiliza con todos los recursos del grupo de disponibilidad. |
   | **Grupos de recursos** |Seleccione el grupo de recursos en el que se encuentran las instancias de SQL Server. |
   | **Ubicación** |Seleccione la ubicación de Azure en la que se encuentran las instancias de SQL Server. |

6. Seleccione **Crear**. 

Azure crea el equilibrador de carga. El equilibrador de carga pertenece a una red, una subred, una ubicación y un grupo de recursos específicos. Cuando Azure haya completado la tarea, compruebe la configuración del equilibrador de carga en Azure. 

### <a name="step-2-configure-the-back-end-pool"></a>Paso 2: Configuración del grupo back-end

En Azure, el grupo de direcciones de back-end se denomina *grupo de back-end*. En este caso, el grupo de back-end contiene las direcciones de las dos instancias de SQL Server del grupo de disponibilidad. 

1. En el grupo de recursos, seleccione el equilibrador de carga que ha creado. 

2. En **Configuración**, seleccione **Grupos de back-end**.

3. En **Grupos de back-end**, seleccione **Agregar** para crear un grupo de direcciones de back-end. 

4. En **Agregar grupo back-end** en **Nombre**, especifique un nombre para el grupo de back-end.

5. En **Máquinas virtuales**, seleccione **Agregar una máquina virtual**. 

6. En **Elegir máquinas virtuales**, seleccione **Elegir un conjunto de disponibilidad** y, a continuación, especifique el conjunto de disponibilidad al que pertenecen las máquinas virtuales de SQL Server.

7. Después de elegir el conjunto de disponibilidad, seleccione **Elegir máquinas virtuales**, seleccione las dos máquinas virtuales que hospedan las instancias de SQL Server en el grupo de disponibilidad y, finalmente, elija **Seleccionar**. 

8. Seleccione **Aceptar** para cerrar las hojas **Elegir máquinas virtuales** y **Agregar grupo back-end**. 

Azure actualiza la configuración del grupo de direcciones de back-end. Ahora, el conjunto de disponibilidad tiene un grupo de dos instancias de SQL Server.

### <a name="step-3-create-a-probe"></a>Paso 3: Elaboración de un sondeo

Este sondeo establece el modo en que Azure va a comprobar cuál de las instancias de SQL Server es el propietario actual del agente de escucha del grupo de disponibilidad. Azure analiza el servició con arreglo a la dirección IP de un puerto que estableció al crear el sondeo.

1. En la hoja **Configuración** del equilibrador de carga, seleccione **Sondeos de estado**. 

2. En la hoja **Sondeos de estado**, seleccione **Agregar**.

3. Configure el sondeo en la hoja **Agregar sondeo** . Utilice los valores siguientes para configurar el sondeo.

   | Configuración | Value |
   | --- | --- |
   | **Nombre** |Nombre de texto que representa el sondeo. Por ejemplo, **SQLAlwaysOnEndPointProbe**. |
   | **Protocolo** |**TCP** |
   | **Puerto** |Puede usar cualquier puerto que esté disponible. Por ejemplo, *59999*. |
   | **Intervalo** |*5* |
   | **Umbral incorrecto** |*2* |

4.  Seleccione **Aceptar**. 

> [!NOTE]
> Asegúrese de que el puerto especificado esté abierto en el firewall de las dos instancias de SQL Server. En estas dos instancias, es necesario definir una regla de entrada para el puerto TCP. Consulte [Agregar o editar regla de firewall](https://technet.microsoft.com/library/cc753558.aspx) para más información. 
> 

Azure crea el sondeo y, a continuación, lo usa para comprobar qué instancia de SQL Server tiene el agente de escucha del grupo de disponibilidad.

### <a name="step-4-set-the-load-balancing-rules"></a>Paso 4: Establecimiento de las reglas de equilibrio de carga

Las reglas de equilibrio de carga determinan cómo el equilibrador de carga enruta el tráfico a las instancias de SQL Server. En este equilibrador de carga, habilite Direct Server Return, ya que solo una de las dos instancias de SQL Server puede ser el propietario del recurso del agente de escucha del grupo de disponibilidad simultáneamente.

1. En la hoja **Configuración** del equilibrador de carga, seleccione **Reglas de equilibrio de carga**. 

2. En la hoja **Reglas de equilibrio de carga**, seleccione **Agregar**.

3. Utilice la hoja **Add load balancing rules** (Agregar reglas de equilibrio de carga) para configurar la regla de equilibrio de carga. Use la configuración siguiente: 

   | Configuración | Value |
   | --- | --- |
   | **Nombre** |Nombre de texto que representa las reglas de equilibrio de carga. Por ejemplo, **SQLAlwaysOnEndPointListener**. |
   | **Protocolo** |**TCP** |
   | **Puerto** |*1433* |
   | **Puerto back-end** |*1433*. Este valor se ignorará porque la regla usa **IP flotante (Direct Server Return)** . |
   | **Sondeo** |Utilice el nombre del sondeo que creó para este equilibrador de carga. |
   | **Persistencia de la sesión** |**None** |
   | **Tiempo de espera de inactividad (minutos)** |*4* |
   | **IP flotante (Direct Server Return)** |**Enabled** |

   > [!NOTE]
   > Tal vez necesite desplazarse hacia abajo en la hoja para ver todas las opciones.
   > 

4. Seleccione **Aceptar**. 

5. Azure configura la regla de equilibrio de carga. Ahora, el equilibrador de carga está configurado para enrutar el tráfico a la instancia de SQL Server que hospeda el agente de escucha del grupo de disponibilidad. 

En este punto, el grupo de recursos dispone de un equilibrador de carga que se conecta con las dos máquinas de SQL Server. El equilibrador de carga también contiene una dirección IP del agente de escucha del grupo de disponibilidad de SQL Server AlwaysOn, por lo que cualquier máquina puede responder a las solicitudes de los grupos de disponibilidad.

> [!NOTE]
> Si las instancias de SQL Server se encuentran en dos regiones diferentes, repita los pasos en la otra región. Cada una de las regiones necesita su propio equilibrador de carga. 
> 

## <a name="configure-the-cluster-to-use-the-load-balancer-ip-address"></a>Configure el clúster para que utilice la dirección IP del equilibrador de carga.

El siguiente paso consiste en configurar el agente de escucha del clúster y conectarlo. Siga estos pasos: 

1. Cree el agente de escucha del grupo de disponibilidad en el clúster de conmutación por error. 

2. Ponga el agente de escucha en línea.

### <a name="step-5-create-the-availability-group-listener-on-the-failover-cluster"></a>Paso 5: Creación del agente de escucha del grupo de disponibilidad en el clúster de conmutación por error

En este paso, se crea manualmente el agente de escucha del grupo de disponibilidad en el Administrador de clústeres de conmutación por error y SQL Server Management Studio.

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

### <a name="verify-the-configuration-of-the-listener"></a>Comprobación de la configuración del agente de escucha

Si los recursos y las dependencias de clúster están configurados correctamente, tendría que ver el agente de escucha en SQL Server Management Studio. Para establecer el puerto del cliente de escucha, realice los pasos siguientes:

1. Abra SQL Server Management Studio y conéctese a la réplica principal.

2. Vaya a **Alta disponibilidad de AlwaysOn** > **Grupos de disponibilidad** > **Agentes de escucha del grupo de disponibilidad**.  

    Ahora tienes que ver el nombre del agente de escucha que creaste en el Administrador de clústeres de conmutación por error. 

3. Haga clic con el botón derecho en el nombre del cliente de escucha y, a continuación, seleccione **Propiedades**.

4. En el cuadro **Puerto**, especifique el número de puerto del cliente de escucha del grupo de disponibilidad mediante el valor de $EndpointPort que ha utilizado antes (1433 era el valor predeterminado) y, a continuación, seleccione **Aceptar**.

Ahora tiene un grupo de disponibilidad en máquinas virtuales de Azure que se ejecutan con el modelo de Resource Manager. 

## <a name="test-the-connection-to-the-listener"></a>Comprobación de la conexión con el agente de escucha

Realice los pasos siguientes para probar la conexión:

1. Use el Protocolo de escritorio remoto (RDP) para conectarse a una instancia de SQL Server que esté en la misma red virtual, pero que no sea la propietaria de la réplica. Este servidor puede ser otra instancia de SQL Server del clúster.

2. Use la utilidad **sqlcmd** para probar la conexión. Por ejemplo, el siguiente script establece una conexión **sqlcmd** con la réplica principal por medio del agente de escucha con autenticación de Windows:

    ```console
    sqlcmd -S <listenerName> -E
    ```

La conexión SQLCMD se establece automáticamente con la instancia de SQL Server en la que se hospede la réplica principal. 

## <a name="create-an-ip-address-for-an-additional-availability-group"></a>Creación de una dirección IP para un grupo de disponibilidad adicional

Cada grupo de disponibilidad usa un agente de escucha independiente. Cada agente de escucha tiene su propia dirección IP. Use el mismo equilibrador de carga para contener la dirección IP de los agentes de escucha adicionales. Después de crear un grupo de disponibilidad, agregue la dirección IP al equilibrador de carga y luego configure el agente de escucha.

Para agregar una dirección IP a un equilibrador de carga con Azure Portal, siga estos pasos:

1. En Azure Portal, abra el grupo de recursos que contiene el equilibrador de carga y seleccione el equilibrador de carga. 

2. En **Configuración**, seleccione **Grupo de direcciones IP de front-end** y, a continuación, seleccione **Agregar**. 

3. En **Agregar dirección IP de front-end**, asigne un nombre para el front-end. 

4. Compruebe que la **red Virtual** y la **subred** sean las mismas que en las instancias de SQL Server.

5. Establezca la dirección IP del agente de escucha. 
   
   >[!TIP]
   >Puede configurar la dirección IP como estática y escribir una dirección que no se use actualmente en la subred. También puede configurar la dirección IP como dinámica y guardar el nuevo grupo de direcciones IP de front-end. Al hacerlo, Azure Portal asigna automáticamente una dirección IP disponible al grupo. Luego puede volver a abrir el grupo de direcciones IP de front-end y cambiar la asignación a estática. 

6. Guarde la dirección IP del agente de escucha. 

7. Agregue un sondeo de estado mediante las siguientes opciones:

   |Configuración |Value
   |:-----|:----
   |**Nombre** |Un nombre para identificar el sondeo.
   |**Protocolo** |TCP
   |**Puerto** |Un puerto TCP sin usar, que debe estar disponible en todas las máquinas virtuales. No se puede usar para ningún otro fin. Dos agentes de escucha no pueden usar el mismo puerto de sondeo. 
   |**Intervalo** |Cantidad de tiempo entre los intentos de sondeo. Use el valor predeterminado (5).
   |**Umbral incorrecto** |El número de umbrales consecutivos que deben generar un error antes de que se considere que una máquina virtual tiene un estado incorrecto.

8. Seleccione **Aceptar** para guardar el sondeo. 

9. Cree una regla de equilibrio de carga. Seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.

10. Configure la nueva regla de equilibrio de carga con los siguientes valores:

    |Configuración |Value
    |:-----|:----
    |**Nombre** |Nombre para identificar la regla de equilibrio de carga. 
    |**Frontend IP address** (Dirección IP de front-end) |Seleccione la dirección IP que creó. 
    |**Protocolo** |TCP
    |**Puerto** |Use el puerto que usan las instancias de SQL Server. Una instancia predeterminada usa el puerto 1433, a menos que lo haya modificado. 
    |**Puerto back-end** |Use el mismo valor que en **Puerto**.
    |**Grupo de back-end** |El grupo que contiene las máquinas virtuales con las instancias de SQL Server. 
    |**Sondeo de estado** |Elija el sondeo que creó.
    |**Persistencia de la sesión** |None
    |**Tiempo de espera de inactividad (minutos)** |Valor predeterminado (4)
    |**IP flotante (Direct Server Return)** | habilitado

### <a name="configure-the-availability-group-to-use-the-new-ip-address"></a>Configuración del grupo de disponibilidad para usar la nueva dirección IP

Para terminar de configurar el clúster, repita los pasos realizados cuando creó el primer grupo de disponibilidad. Es decir, configure el [clúster para usar la nueva dirección IP](#configure-the-cluster-to-use-the-load-balancer-ip-address). 

Después de agregar una dirección IP para el cliente de escucha, configure el grupo de disponibilidad adicional mediante los pasos siguientes: 

1. Compruebe que el puerto de sondeo de la nueva dirección IP esté abierto en ambas máquinas virtuales SQL Server. 

2. [En el Administrador de clústeres, agregue el punto de acceso cliente](#addcap).

3. [Configurar el recurso IP para el grupo de disponibilidad](#congroup).

   >[!IMPORTANT]
   >Cuando cree la dirección IP, use la dirección IP que agregó al equilibrador de carga.  

4. [Haga que el recurso del grupo de disponibilidad de SQL Server dependa del punto de acceso cliente](#dependencyGroup).

5. [Hacer que el recurso de punto de acceso cliente dependa de la dirección IP](#listname).
 
6. [Establecer los parámetros de clúster en PowerShell](#setparam).

Después de configurar el grupo de disponibilidad para usar la nueva dirección IP, configure la conexión al agente de escucha. 

## <a name="add-load-balancing-rule-for-distributed-availability-group"></a>Adición de una regla de equilibrio de carga para un grupo de disponibilidad distribuido

Si un grupo de disponibilidad forma parte de un grupo de disponibilidad distribuido, el equilibrador de carga necesita una regla adicional. Esta regla almacena el puerto usado por el escucha del grupo de disponibilidad distribuido.

>[!IMPORTANT]
>Este paso solo se aplica si el grupo de disponibilidad forma parte de un [grupo de disponibilidad distribuido](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/configure-distributed-availability-groups). 

1. En cada servidor que forma parte del grupo de disponibilidad distribuido, cree una regla de entrada en el puerto TCP del escucha del grupo de disponibilidad distribuido. En muchos ejemplos, la documentación usa el puerto 5022. 

1. En Azure Portal, seleccione el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **+Agregar**. 

1. Cree la regla de equilibrio de carga con la configuración siguiente:

   |Configuración |Value
   |:-----|:----
   |**Nombre** |Nombre para identificar la regla de equilibrio de carga para el grupo de disponibilidad distribuido. 
   |**Frontend IP address** (Dirección IP de front-end) |Use la misma dirección IP de front-end que en el grupo de disponibilidad.
   |**Protocolo** |TCP
   |**Puerto** |5022: el puerto para el [escucha de punto de conexión del grupo de disponibilidad distribuido](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/configure-distributed-availability-groups).</br> Puede ser cualquier puerto disponible.  
   |**Puerto back-end** | 5022: use el mismo valor que en **Puerto**.
   |**Grupo de back-end** |El grupo que contiene las máquinas virtuales con las instancias de SQL Server. 
   |**Sondeo de estado** |Elija el sondeo que creó.
   |**Persistencia de la sesión** |None
   |**Tiempo de espera de inactividad (minutos)** |Valor predeterminado (4)
   |**IP flotante (Direct Server Return)** | habilitado

Repita estos pasos para el equilibrador de carga de los otros grupos de disponibilidad que forman parte de los grupos de disponibilidad distribuidos.

Si tiene un grupo de seguridad de red de Azure para restringir el acceso, asegúrese de que las reglas de tipo permitir incluyen:
- Las direcciones IP de la máquina virtual de SQL Server de back-end
- Las direcciones IP flotantes del equilibrador de carga para el cliente de escucha de AG
- La dirección IP principal del clúster, si procede.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de un grupo de disponibilidad de SQL Server AlwaysOn en Azure Virtual Machines en regiones distintas](availability-group-manually-configure-multiple-regions.md)
