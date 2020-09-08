---
title: 'Configuración de una escucha de grupo de disponibilidad para SQL Server en máquinas virtuales de Red Hat Enterprise Linux en Azure: Linux Virtual Machines | Microsoft Docs'
description: Información acerca de la configuración de una escucha de grupo de disponibilidad en SQL Server en máquinas virtuales RHEL en Azure
ms.service: virtual-machines-linux
ms.subservice: ''
ms.topic: tutorial
author: VanMSFT
ms.author: vanto
ms.reviewer: jroth
ms.date: 03/11/2020
ms.openlocfilehash: 01501b99d5d7c42af98d0397cf6ff8cbca14b07b
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89485811"
---
# <a name="tutorial-configure-an-availability-group-listener-for-sql-server-on-rhel-virtual-machines-in-azure"></a>Tutorial: Configuración de una escucha de grupo de disponibilidad para SQL Server en máquinas virtuales con Red Hat Enterprise Linux en Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!NOTE]
> El tutorial que se presenta está en **versión preliminar pública**. 
>
> En este tutorial, vamos a usar SQL Server 2017 con RHEL 7.6, pero es posible usar SQL Server 2019 en RHEL 7 o RHEL 8 para configurar una alta disponibilidad. Los comandos para configurar los recursos del grupo de disponibilidad han cambiado en RHEL 8 y le interesará consultar el artículo [Crear un recurso de grupo de disponibilidad](/sql/linux/sql-server-linux-availability-group-cluster-rhel#create-availability-group-resource) y los recursos de RHEL 8 para más información sobre los comandos correctos.

En este tutorial se explican los pasos para crear un escucha de grupo de disponibilidad para los servidores SQL Server en las máquinas virtuales RHEL en Azure. Aprenderá a:

> [!div class="checklist"]
> - Crear un equilibrador de carga en Azure Portal
> - Configurar el grupo de back-end para el equilibrador de carga
> - Crear un sondeo para el equilibrador de carga
> - Configuración de las reglas de equilibrio de carga
> - Crear el recurso del equilibrador de carga en el clúster
> - Creación del agente de escucha de grupo de disponibilidad
> - Probar la conexión a la escucha
> - Probar una conmutación por error

## <a name="prerequisite"></a>Requisito previo

Que se haya completado [Tutorial: Configuración de grupos de disponibilidad para SQL Server en máquinas virtuales de Red Hat Enterprise Linux en Azure](rhel-high-availability-stonith-tutorial.md)

## <a name="create-the-load-balancer-in-the-azure-portal"></a>Creación del equilibrador de carga en Azure Portal

Las instrucciones siguientes le guiarán por los pasos del 1 al 4 de la sección [Creación y configuración del equilibrador de carga en Azure Portal](../windows/availability-group-load-balancer-portal-configure.md#create--configure-load-balancer) del artículo sobre [Equilibrador de carga: Azure Portal](../windows/availability-group-load-balancer-portal-configure.md).

### <a name="create-the-load-balancer"></a>Creación del equilibrador de carga

1. En el Portal de Azure, abra el grupo de recursos que contiene las máquinas virtuales de SQL Server. 

2. En el grupo de recursos, haga clic en **Agregar**.

3. Busque el **equilibrador de carga** y, a continuación, en los resultados de la búsqueda, seleccione el **equilibrador de carga** publicado por **Microsoft**.

4. En la hoja **Load Balancer**, haga clic en **Crear**.

5. En el cuadro de diálogo **Crear equilibrador de carga**, configure el equilibrador de carga tal y como se explica a continuación:

   | Configuración | Value |
   | --- | --- |
   | **Nombre** |Nombre de texto que representa el equilibrador de carga; Por ejemplo, **sqlLB**. |
   | **Tipo** |**Interno** |
   | **Red virtual** |La red virtual predeterminada que se creó debe denominarse **VM1VNET**. |
   | **Subred** |Seleccione la subred en la que se encuentran las instancias de SQL Server. El valor predeterminado debe ser **VM1Subnet**.|
   | **Asignación de dirección IP** |**Estática** |
   | **Dirección IP privada** |Use la dirección IP `virtualip` que se creó en el clúster. |
   | **Suscripción** |Use la suscripción que se usó para el grupo de recursos. |
   | **Grupos de recursos** |Seleccione el grupo de recursos en el que se encuentran las instancias de SQL Server. |
   | **Ubicación** |Seleccione la ubicación de Azure en la que se encuentran las instancias de SQL Server. |

### <a name="configure-the-back-end-pool"></a>Configuración del grupo back-end
En Azure, el grupo de direcciones de back-end se denomina *grupo de back-end*. En este caso, el grupo de back-end contiene las direcciones de las tres instancias de SQL Server del grupo de disponibilidad. 

1. En el grupo de recursos, haga clic en el equilibrador de carga que ha creado. 

2. En **Configuración**, haga clic en **Grupos de back-end**.

3. En **Grupos de back-end**, haga clic en **Agregar** para crear un grupo de direcciones de back-end. 

4. En **Agregar grupo back-end** en **Nombre**, especifique un nombre para el grupo de back-end.

5. En **Asociado a**, seleccione **Máquina virtual**. 

6. Seleccione cada una de las máquinas virtuales del entorno y asocie la dirección IP adecuada a cada selección.

    :::image type="content" source="media/rhel-high-availability-listener-tutorial/add-backend-pool.png" alt-text="Agregar grupo back-end":::

7. Haga clic en **Agregar**. 

### <a name="create-a-probe"></a>Elaboración de un sondeo

Este sondeo establece el modo en que Azure va a comprobar cuál de las instancias de SQL Server es el propietario actual del agente de escucha del grupo de disponibilidad. Azure analiza el servició con arreglo a la dirección IP de un puerto que estableció al crear el sondeo.

1. En la hoja **Configuración** del equilibrador de carga, haga clic en **Sondeos de estado**. 

2. En la hoja **Sondeos de estado**, haga clic en **Agregar**.

3. Configure el sondeo en la hoja **Agregar sondeo** . Utilice los valores siguientes para configurar el sondeo.

   | Configuración | Value |
   | --- | --- |
   | **Nombre** |Nombre de texto que representa el sondeo. Por ejemplo, **SQLAlwaysOnEndPointProbe**. |
   | **Protocolo** |**TCP** |
   | **Puerto** |Puede usar cualquier puerto que esté disponible. Por ejemplo, *59999*. |
   | **Intervalo** |*5* |
   | **Umbral incorrecto** |*2* |

4.  Haga clic en **OK**. 

5. Inicie sesión en todas las máquinas virtuales y abra el puerto de sondeo con los siguientes comandos:

    ```bash
    sudo firewall-cmd --zone=public --add-port=59999/tcp --permanent
    sudo firewall-cmd --reload
    ```

Azure crea el sondeo y, a continuación, lo usa para comprobar qué instancia de SQL Server tiene el agente de escucha del grupo de disponibilidad.

### <a name="set-the-load-balancing-rules"></a>Establecimiento de las reglas de equilibrio de carga

Las reglas de equilibrio de carga determinan cómo el equilibrador de carga enruta el tráfico a las instancias de SQL Server. En este equilibrador de carga, habilite Direct Server Return, ya que solo una de las tres instancias de SQL Server puede ser el propietario del recurso de la escucha de grupo de disponibilidad simultáneamente.

1. En la hoja **Configuración** del equilibrador de carga, haga clic en **Reglas de equilibrio de carga**. 

2. En la hoja **Reglas de equilibrio de carga**, haga clic en **Agregar**.

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

   :::image type="content" source="media/rhel-high-availability-listener-tutorial/add-load-balancing-rule.png" alt-text="Agregar regla de equilibrio de carga":::

4. Haga clic en **OK**. 
5. Azure configura la regla de equilibrio de carga. Ahora, el equilibrador de carga está configurado para enrutar el tráfico a la instancia de SQL Server que hospeda el agente de escucha del grupo de disponibilidad. 

En este punto, el grupo de recursos dispone de un equilibrador de carga que se conecta con todas las máquinas de SQL Server. El equilibrador de carga también contiene una dirección IP de la escucha de grupo de disponibilidad AlwaysOn de SQL Server, por lo que cualquier máquina puede responder a las solicitudes de los grupos de disponibilidad.

## <a name="create-the-load-balancer-resource-in-the-cluster"></a>Creación del recurso del equilibrador de carga en el clúster

1. Inicie sesión en la máquina virtual principal. Tenemos que crear el recurso para habilitar el puerto de sondeo del equilibrador de carga de Azure (en nuestro ejemplo se usa 59999). Ejecute el siguiente comando:

    ```bash
    sudo pcs resource create azure_load_balancer azure-lb port=59999
    ```

1. Cree un grupo que contenga los recursos `virtualip` y `azure_load_balancer`:

    ```bash
    sudo pcs resource group add virtualip_group azure_load_balancer virtualip
    ```

### <a name="add-constraints"></a>Incorporación de restricciones

1. Tiene que configurarse una restricción de ubicación para asegurarse de que el recurso de grupo de disponibilidad y la dirección IP del equilibrador de carga de Azure se están ejecutando en el mismo nodo. Ejecute el siguiente comando:

    ```bash
    sudo pcs constraint colocation add azure_load_balancer ag_cluster-master INFINITY with-rsc-role=Master
    ```
1. Cree una restricción de orden para asegurarse de que el recurso de grupo de disponibilidad está en funcionamiento antes que la dirección IP del equilibrador de carga de Azure. Aunque la restricción de ubicación implica una restricción de orden, esto la aplica.

    ```bash
    sudo pcs constraint order promote ag_cluster-master then start azure_load_balancer
    ```

1. Para comprobar las restricciones, ejecute el siguiente comando:

    ```bash
    sudo pcs constraint list --full
    ```

    Debería ver la siguiente salida:

    ```output
    Location Constraints:
    Ordering Constraints:
      promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
      promote ag_cluster-master then start azure_load_balancer (kind:Mandatory) (id:order-ag_cluster-master-azure_load_balancer-mandatory)
    Colocation Constraints:
      virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
      azure_load_balancer with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-azure_load_balancer-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```

## <a name="create-the-availability-group-listener"></a>Creación del agente de escucha de grupo de disponibilidad

1. En el nodo principal, ejecute el siguiente comando en SQLCMD o SSMS:

    - Reemplace la dirección IP que se usa a continuación con la dirección IP `virtualip`.

    ```sql
    ALTER AVAILABILITY
    GROUP [ag1] ADD LISTENER 'ag1-listener' (
            WITH IP(('10.0.0.7'    ,'255.255.255.0'))
                ,PORT = 1433
            );
    GO
    ```

1. Inicie sesión en cada nodo de la máquina virtual. Use el siguiente comando para abrir el archivo de hosts y configurar la resolución de nombres de host de `ag1-listener` en cada máquina.

    ```
    sudo vi /etc/hosts
    ```

    En el editor de **vi**, escriba `i` para insertar texto y, en una línea en blanco, agregue la dirección IP de `ag1-listener`. A continuación, agregue `ag1-listener` después de un espacio junto a la dirección IP.

    ```output
    <IP of ag1-listener> ag1-listener
    ```

    Para salir del editor de **vi**, primero presione la tecla **Esc** y, a continuación, escriba el comando `:wq` para escribir en el archivo y salir. Haga esto en cada nodo.

## <a name="test-the-listener-and-a-failover"></a>Prueba de la escucha y de una conmutación por error

### <a name="test-logging-in-to-sql-server-using-the-availability-group-listener"></a>Prueba del inicio de sesión en SQL Server mediante la escucha de grupo de disponibilidad

1. Use SQLCMD para iniciar sesión en el nodo principal de SQL Server mediante el nombre de la escucha de grupo de disponibilidad:

    - Use un inicio de sesión que se haya creado previamente y reemplace `<YourPassword>` por la contraseña correcta. En el ejemplo siguiente se usa el inicio de sesión `sa` que se creó con la instancia de SQL Server.

    ```bash
    sqlcmd -S ag1-listener -U sa -P <YourPassword>
    ```

1. Compruebe el nombre del servidor al que está conectado. Ejecute el comando siguiente en SQLCMD:

    ```sql
    SELECT @@SERVERNAME
    ```

    El resultado debería mostrar el nodo principal actual. Este debe ser `VM1` si nunca ha probado una conmutación por error.

    Salga de la sesión de SQL Server escribiendo el comando `exit`.

### <a name="test-a-failover"></a>Prueba de una conmutación por error

1. Ejecute el siguiente comando para realizar manualmente la conmutación por error de la réplica principal en `<VM2>` u otra réplica. Reemplace `<VM2>` por el valor del nombre del servidor.

    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master
    ```

1. Si comprueba las restricciones, verá que se ha agregado otra restricción debido a la conmutación por error manual:

    ```bash
    sudo pcs constraint list --full
    ```

    Verá que se ha agregado una restricción con el identificador `cli-prefer-ag_cluster-master`.

1. Para quitar la restricción con el identificador `cli-prefer-ag_cluster-master`, ejecute el comando siguiente:

    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-master
    ```

1. Compruebe los recursos del clúster mediante el comando `sudo pcs resource` y observará que la instancia principal es ahora `<VM2>`.

    ```output
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
        Masters: [ <VM2> ]
        Slaves: [ <VM1> <VM3> ]
    Resource Group: virtualip_group
        azure_load_balancer        (ocf::heartbeat:azure-lb):      Started <VM2>
        virtualip  (ocf::heartbeat:IPaddr2):       Started <VM2>
    ```

1. Use SQLCMD para iniciar sesión en la réplica principal mediante el nombre de la escucha:

    - Use un inicio de sesión que se haya creado previamente y reemplace `<YourPassword>` por la contraseña correcta. En el ejemplo siguiente se usa el inicio de sesión `sa` que se creó con la instancia de SQL Server.

    ```bash
    sqlcmd -S ag1-listener -U sa -P <YourPassword>
     ```

1. Compruebe el servidor con el que está conectado. Ejecute el comando siguiente en SQLCMD:

    ```sql
    SELECT @@SERVERNAME
    ```

    Verá que ahora está conectado a la máquina virtual a la que ha conmutado por error.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los equilibradores de carga de Azure consulte:

> [!div class="nextstepaction"]
> [Configuración de un equilibrador de carga para un grupo de disponibilidad de SQL Server en máquinas virtuales de Azure](../windows/availability-group-load-balancer-portal-configure.md)
