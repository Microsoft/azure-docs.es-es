---
title: Uso de Azure Firewall para inspeccionar el tráfico destinado a un punto de conexión privado
titleSuffix: Azure Private Link
description: Obtenga información sobre cómo puede inspeccionar el tráfico destinado a un punto de conexión privado mediante Azure Firewall.
services: private-link
author: jocortems
ms.service: private-link
ms.topic: how-to
ms.date: 09/02/2020
ms.author: allensu
ms.openlocfilehash: 734d52dadbb849925303febb0d3d1195bbddb0df
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89236570"
---
# <a name="use-azure-firewall-to-inspect-traffic-destined-to-a-private-endpoint"></a>Uso de Azure Firewall para inspeccionar el tráfico destinado a un punto de conexión privado

Un punto de conexión privado de Azure es el bloque de creación fundamental para Azure Private Link. Los puntos de conexión privados permiten que los recursos de Azure implementados en una red virtual se comuniquen de forma privada con recursos de Private Link.

Los puntos de conexión privados permiten a los recursos el acceso al servicio Private Link implementado en una red virtual. El acceso al punto de conexión privado a través del emparejamiento de red virtual y las conexiones de red locales amplía la conectividad.

Es posible que tenga que inspeccionar o bloquear el tráfico de los clientes a los servicios expuestos a través de puntos de conexión privados. Complete esta inspección mediante [Azure Firewall](../firewall/overview.md) o una aplicación virtual de red de terceros.

Se aplican las siguientes limitaciones:

* Los grupos de seguridad de red (NSG) no se aplican a los puntos de conexión privados
* Las rutas definidas por el usuario (UDR) no se aplican a los puntos de conexión privados
* Se puede asociar una sola tabla de rutas a una subred
* Una tabla de rutas admite hasta 400 rutas

Azure Firewall filtra el tráfico mediante:

* [FQDN en las reglas de red](../firewall/fqdn-filtering-network-rules.md) para los protocolos TCP y UDP
* [FQDN en las reglas de aplicación](../firewall/features.md#application-fqdn-filtering-rules) para HTTP, HTTPS y MSSQL. 

La mayoría de los servicios expuestos a través de puntos de conexión privados usan HTTPS. Se recomienda el uso de reglas de aplicación a través de reglas de red al utilizar Azure SQL.

> [!NOTE]
> El filtrado por nombre de dominio completo de SQL se admite solo en [modo de proxy](../azure-sql/database/connectivity-architecture.md#connection-policy) (puerto 1433). El modo de **proxy** puede traducirse en una mayor latencia en comparación con el modo de *redirección*. Si quiere seguir usando el modo de redirección, que es el valor predeterminado para los clientes que se conectan desde dentro de Azure, puede filtrar el acceso mediante FQDN en reglas de red del firewall.

## <a name="scenario-1-hub-and-spoke-architecture---dedicated-virtual-network-for-private-endpoints"></a>Escenario 1: Arquitectura en estrella tipo hub-and-spoke: red virtual dedicada para puntos de conexión privados

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/hub-and-spoke.png" alt-text="Red virtual dedicada para puntos de conexión privados" border="true":::

Este escenario es la arquitectura más expansible para conectarse de forma privada a varios servicios de Azure mediante puntos de conexión privados. Se crea una ruta que apunta al espacio de direcciones de red donde se implementan los puntos de conexión privados. Esta configuración reduce la sobrecarga administrativa y evita que se ejecute en el límite de 400 rutas.

Las conexiones desde una red virtual de cliente a Azure Firewall en una red virtual de concentrador incurrirán en cargos si las redes virtuales están emparejadas.

Para obtener más información sobre los cargos relacionados con las conexiones con redes virtuales emparejadas, consulte la sección de preguntas más frecuentes de la página de [precios](https://azure.microsoft.com/pricing/details/private-link/).

>[!NOTE]
> Este escenario se puede implementar con cualquier regla de red de Azure Firewall o NVA de terceros en lugar de reglas de aplicación.

## <a name="scenario-2-hub-and-spoke-architecture---shared-virtual-network-for-private-endpoints-and-virtual-machines"></a>Escenario 2: Arquitectura en estrella tipo hub-and-spoke: red virtual compartida para máquinas virtuales y puntos de conexión privados

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/shared-spoke.png" alt-text="Máquinas virtuales y puntos de conexión privados en la misma red virtual" border="true":::

Este escenario se implementa cuando:

* No es posible tener una red virtual dedicada para los puntos de conexión privados

* Cuando solo algunos servicios se exponen en la red virtual con puntos de conexión privados

Las máquinas virtuales tendrán rutas del sistema /32 que apunten a cada punto de conexión privado. Una ruta por punto de conexión privado está configurada para enrutar el tráfico a través de Azure Firewall. 

La sobrecarga administrativa del mantenimiento de la tabla de rutas aumenta a medida que los servicios se exponen en la red virtual. También aumenta la posibilidad de alcanzar el límite de ruta.

Según la arquitectura global, es posible ejecutar en el límite de 400 rutas. Se recomienda usar el escenario 1 siempre que sea posible.

Las conexiones desde una red virtual de cliente a Azure Firewall en una red virtual de concentrador incurrirán en cargos si las redes virtuales están emparejadas.

Para obtener más información sobre los cargos relacionados con las conexiones con redes virtuales emparejadas, consulte la sección de preguntas más frecuentes de la página de [precios](https://azure.microsoft.com/pricing/details/private-link/).

>[!NOTE]
> Este escenario se puede implementar con cualquier regla de red de Azure Firewall o NVA de terceros en lugar de reglas de aplicación.

## <a name="scenario-3-single-virtual-network"></a>Escenario 3: Red virtual única

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/single-vnet.png" alt-text="Red virtual única" border="true":::

Existen algunas limitaciones en la implementación: no es posible realizar una migración a una arquitectura en estrella tipo hub-and-spoke. Se aplican las mismas consideraciones que en el escenario 2. En este escenario, no se aplican cargos de emparejamiento de red virtual.

>[!NOTE]
> Si desea implementar este escenario mediante reglas de red de Azure Firewall o NVA de terceros, se requieren reglas de red en lugar de reglas de aplicación para el tráfico de SNAT destinado a los puntos de conexión privados. De lo contrario, se producirá un error en la comunicación entre las máquinas virtuales y los puntos de conexión privados.

## <a name="scenario-4-on-premises-traffic-to-private-endpoints"></a>Escenario 4: Tráfico local en puntos de conexión privados

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/on-premises.png" alt-text="Tráfico local en puntos de conexión privados" border="true":::

Esta arquitectura se puede implementar si ha configurado la conectividad con la red local mediante: 

* [ExpressRoute](..\expressroute\expressroute-introduction.md)
* [VPN sitio a sitio](..\vpn-gateway\vpn-gateway-howto-site-to-site-resource-manager-portal.md) 

Si los requisitos de seguridad requieren el enrutamiento del tráfico de cliente a los servicios expuestos a través de puntos de conexión privados a través de un dispositivo de seguridad, implemente este escenario.

Se aplican las mismas consideraciones que en el escenario 2. En este escenario, no hay cargos de emparejamiento de red virtual. Para obtener más información sobre cómo configurar los servidores DNS para permitir que las cargas de trabajo locales tengan acceso a puntos de conexión privados, consulte [Cargas de trabajo locales que utilizan un reenviador DNS](./private-endpoint-dns.md#on-premises-workloads-using-a-dns-forwarder).

>[!NOTE]
> Si desea implementar este escenario mediante reglas de red de Azure Firewall o NVA de terceros, se requieren reglas de red en lugar de reglas de aplicación para el tráfico de SNAT destinado a los puntos de conexión privados. De lo contrario, se producirá un error en la comunicación entre las máquinas virtuales y los puntos de conexión privados.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure.
* Un área de trabajo de Log Analytics.  

Consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace) para crear un área de trabajo si no tiene ninguna en la suscripción.


## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-vm"></a>Crear una VM

En esta sección, va a crear una red virtual y una subred para hospedar la máquina virtual que se usa para acceder al recurso de Private Link. Se utiliza una base de datos de Azure SQL como servicio de ejemplo.

## <a name="virtual-networks-and-parameters"></a>Redes virtuales y parámetros

Cree tres redes virtuales y sus subredes correspondientes para:

* Contener la instancia de Azure Firewall que se usa para restringir la comunicación entre la máquina virtual y el punto de conexión privado.
* Hospedar la máquina virtual que se usa para tener acceso al recurso de Private Link.
* Hospedar el punto de conexión privado.

Reemplazar los siguientes parámetros de los pasos por la siguiente información:

### <a name="azure-firewall-network"></a>Red de Azure Firewall
| Parámetro                   | Value                 |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroup |
| **\<virtual-network-name>** | myAzFwVNet          |
| **\<region-name>**          | Centro-sur de EE. UU.      |
| **\<IPv4-address-space>**   | 10.0.0.0/16          |
| **\<subnet-name>**          | AzureFirewallSubnet        |
| **\<subnet-address-range>** | 10.0.0.0/24          |

### <a name="virtual-machine-network"></a>Red de máquinas virtuales
| Parámetro                   | Value                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroup |
| **\<virtual-network-name>** | myVMVNet          |
| **\<region-name>**          | Centro-sur de EE. UU.      |
| **\<IPv4-address-space>**   | 10.1.0.0/16          |
| **\<subnet-name>**          | VMSubnet      |
| **\<subnet-address-range>** | 10.1.0.0/24          |

### <a name="private-endpoint-network"></a>Red de punto de conexión privado
| Parámetro                   | Value                 |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroup |
| **\<virtual-network-name>** | myPEVNet         |
| **\<region-name>**          | Centro-sur de EE. UU.      |
| **\<IPv4-address-space>**   | 10.2.0.0/16          |
| **\<subnet-name>**          | PrivateEndpointSubnet    |        |
| **\<subnet-address-range>** | 10.2.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

10. Repita los pasos del 1 al 9 para crear las redes virtuales para hospedar los recursos de punto de conexión privado y máquina virtual.

### <a name="create-virtual-machine"></a>Crear máquina virtual

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**.

2. En **Creación de una máquina virtual: conceptos básicos**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Ha creado este grupo de recursos en la sección anterior.  |
    | **Detalles de instancia** |  |
    | Nombre de la máquina virtual | Escriba **myVM**. |
    | Region | Seleccione **Centro-sur de EE. UU. (EE. UU.)** . |
    | Opciones de disponibilidad | Deje el valor predeterminado **No se requiere redundancia de la infraestructura**. |
    | Imagen | Seleccione **Ubuntu Server 18.04 LTS: Gen1**. |
    | Size | Seleccione **Standard_B2s**. |
    | **Cuenta de administrador** |  |
    | Tipo de autenticación | Seleccione **Contraseña**. |
    | Nombre de usuario | Escriba un nombre de usuario de su elección. |
    | Contraseña | Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/linux/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    | Confirm Password | Vuelva a escribir la contraseña. |
    | **Reglas de puerto de entrada** |  |
    | Puertos de entrada públicos | Seleccione **Ninguno**. |
    |||

3. Seleccione **Siguiente: Discos**.

4. En **Creación de una máquina virtual: Discos**, deje los valores predeterminados y seleccione **Siguiente: Redes**.

5. En **Creación de una máquina virtual: Redes**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Virtual network | Seleccione **myVMVNet**.  |
    | Subnet | Seleccione **VMSubnet (10.1.0.0/24)** .|
    | Dirección IP pública | Deje el valor predeterminado **(new) myVm-ip**. |
    | Puertos de entrada públicos | Seleccione **Permitir los puertos seleccionados**. |
    | Selección de puertos de entrada | Seleccione **SSH**.|
    ||

6. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

7. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

## <a name="deploy-the-firewall"></a>Implementación del firewall

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.

2. Escriba **firewall** en el cuadro de búsqueda y presione **Entrar**.

3. Seleccione **Firewall** y después **Crear**.

4. En la página **Creación de un firewall**, utilice la tabla siguiente para configurar el firewall:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**.  |
    | **Detalles de instancia** |  |
    | Nombre | Escriba **myAzureFirewall**. |
    | Region | Seleccione **Centro-sur de EE. UU.** . |
    | Zona de disponibilidad | Deje el valor predeterminado **Ninguno**. |
    | Elegir una red virtual    |    Seleccione **Usar existente**.    |
    | Virtual network    |    Seleccione **myAzFwVNet**.    |
    | Dirección IP pública    |    Seleccione **Agregar nuevo** y, en Nombre, escriba **myFirewall-ip**.    |
    | Tunelización forzada    | Deje el valor predeterminado **Deshabilitado**.    |
    |||
5. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

6. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

## <a name="enable-firewall-logs"></a>Habilitación de registros de firewall

En esta sección, habilitará los registros en el firewall.

1. En Azure Portal, seleccione **Todos los recursos** en el menú de la izquierda.

2. Seleccione el firewall **myAzureFirewall** en la lista de recursos.

3. En **Supervisión**, en la configuración del firewall, seleccione **Configuración de diagnóstico**

4. Seleccione **+ Agregar configuración de diagnóstico** en la configuración de diagnóstico.

5. En **Configuración de diagnóstico**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre de la configuración de diagnóstico | Escriba **myDiagSetting**. |
    | Detalles de categoría | |
    | log | Seleccione **AzureFirewallApplicationRule** y **AzureFirewallNetworkRule**. |
    | Detalles de destino | Seleccione **Enviar a Log Analytics**. |
    | Subscription | Seleccione su suscripción. |
    | Área de trabajo de Log Analytics | Seleccione el área de trabajo de Log Analytics. |

6. Seleccione **Guardar**.

## <a name="create-azure-sql-database"></a>Crear una base de datos de Azure SQL

En esta sección, creará una instancia de SQL Database privada.

1. En la parte superior izquierda de la pantalla en Azure Portal, seleccione **Crear un recurso** > **Bases de datos** > **SQL Database**.

2. En **Crear SQL Database: Conceptos básicos**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Ha creado este grupo de recursos en la sección anterior.|
    | **Detalles de la base de datos** |  |
    | Nombre de la base de datos  | Escriba **miBaseDeDatos**.  |
    | Server | Seleccione **Crear nuevo** y escriba la información a continuación.    |
    | Nombre de servidor | Escriba **mydbserver**. Si el nombre ya existe, escriba uno único.   |
    | Inicio de sesión de administrador de servidor | Escriba el nombre que prefiera. |
    | Contraseña    |    Escriba una contraseña de su elección.    |
    | Confirm Password | Vuelva a escribir la contraseña.    |
    | Location    | Seleccione **Centro-sur de EE. UU. (EE. UU.)** .    |
    | ¿Quiere usar un grupo elástico de SQL?    | Deje el valor predeterminado **No**. |
    | Proceso y almacenamiento | Deje el valor predeterminado **Gen5 de uso general, 2 núcleos virtuales, 32 GB de almacenamiento**. |
    |||

3. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

4. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección, creará un punto de conexión privado para la base de datos de Azure SQL en la sección anterior.

1. En Azure Portal, seleccione **Todos los recursos** en el menú de la izquierda.

2. Seleccione el servidor SQL de Azure, **mydbserver** en la lista de servicios.  Si ha usado otro nombre del servidor, elija ese nombre.

3. En la configuración del servidor, seleccione **Conexiones de punto de conexión privado** en **Seguridad**.

4. Seleccione **+ Punto de conexión privado**.

5. En **Crear un punto de conexión privado**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. |
    | **Detalles de instancia** | |
    | Nombre | Escriba **SQLPrivateEndpoint**. |
    | Region | Seleccione **Centro-sur de EE. UU. (EE. UU.)** . |

6. Seleccione la pestaña **Recurso** o seleccione **Siguiente: Recurso** en la parte inferior de la página.

7. En la pestaña **Recurso**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Método de conexión | Seleccione **Conectarse a un recurso de Azure en mi directorio**. |
    | Subscription | Seleccione su suscripción. |
    | Tipo de recurso | Seleccione **Microsoft.Sql/servers**. |
    | Resource | Seleccione **mydbserver** o el nombre del servidor que creó en el paso anterior.
    | Recurso secundario de destino | Seleccione **sqlServer**. |

8. Seleccione la pestaña **Configuración** o **Siguiente: Configuración** situado en la parte inferior de la página.

9. En la pestaña **Configuración**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **Redes** | |
    | Virtual network | Seleccione **myPEVnet**. |
    | Subnet | Seleccione **PrivateEndpointSubnet**. |
    | **Integración de DNS privado** | |
    | Integración con una zona DNS privada | Seleccione **Sí**. |
    | Subscription | Seleccione su suscripción. |
    | Zonas DNS privadas | Deje el valor predeterminado **privatelink.database.windows.net**. |

10. Seleccione la pestaña **Revisar y crear** o **Revisar y crear** en la parte inferior de la página.

11. Seleccione **Crear**.

12. Tras crearse el punto de conexión, seleccione **Firewalls y redes virtuales** en **Seguridad**.

13. En **Firewalls y redes virtuales**, seleccione **Sí** junto a **Permitir que los servicios y recursos de Azure tengan acceso a este servidor**.

14. Seleccione **Guardar**.

## <a name="connect-the-virtual-networks-using-virtual-network-peering"></a>Conexión de las redes virtuales con el emparejamiento de red virtual

En esta sección, conectaremos las redes virtuales **myVMVNet** y **myPEVNet** a **myAzFwVNet** mediante el emparejamiento. No habrá conectividad directa entre **myVMVNet** y **myPEVNet**.

1. En la barra de búsqueda del portal, escriba **myAzFwVNet**.

2. Seleccione **Emparejamientos** en el menú **Configuración** y seleccione **+ Agregar**.

3. En **Agregar emparejamiento** escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre del emparejamiento de myAzFwVNet con la red virtual remota | Escriba **myAzFwVNet-to-myVMVNet**. |
    | **Detalles de miembro del mismo nivel** |  |
    | Modelo de implementación de red virtual  | Deje el valor predeterminado **Resource Manager**.  |
    | Conozco mi Id. de recurso | Deje la opción desactivada.    |
    | Subscription | Seleccione su suscripción.    |
    | Virtual network | Seleccione **myVMVNet**. |
    | Nombre del emparejamiento de la red virtual remota con myAzFwVNet    |    Escriba **myVMVNet-to-myAzFwVNet**.    |
    | **Configuración** | |
    | **Configuración del acceso a red virtual** | |
    | Permitir acceso a red virtual de myAzFwVNet a la red virtual remota | Deje el valor predeterminado **Habilitado**.    |
    | Permitir acceso a red virtual de la red virtual remota a myAzFwVNet    | Deje el valor predeterminado **Habilitado**.    |
    | **Configuración del tráfico reenviado** | |
    | Permitir tráfico reenviado de la red virtual remota a myAzFwVNet    | Seleccione **Habilitado**. |
    | Permitir tráfico reenviado de myAzFwVNet a la red virtual remota | Seleccione **Habilitado**. |
    | **Configuración de tránsito de la puerta de enlace** | |
    | Permitir tránsito de puerta de enlace | Dejar la opción desactivada |
    |||

4. Seleccione **Aceptar**.

5. Seleccione **+Agregar**.

6. En **Agregar emparejamiento** escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre del emparejamiento de myAzFwVNet con la red virtual remota | Escriba **myAzFwVNet-to-myPEVNet**. |
    | **Detalles de miembro del mismo nivel** |  |
    | Modelo de implementación de red virtual  | Deje el valor predeterminado **Resource Manager**.  |
    | Conozco mi Id. de recurso | Deje la opción desactivada.    |
    | Subscription | Seleccione su suscripción.    |
    | Virtual network | Seleccione **myPEVNet**. |
    | Nombre del emparejamiento de la red virtual remota con myAzFwVNet    |    Escriba **myPEVNet-to-myAzFwVNet**.    |
    | **Configuración** | |
    | **Configuración del acceso a red virtual** | |
    | Permitir acceso a red virtual de myAzFwVNet a la red virtual remota | Deje el valor predeterminado **Habilitado**.    |
    | Permitir acceso a red virtual de la red virtual remota a myAzFwVNet    | Deje el valor predeterminado **Habilitado**.    |
    | **Configuración del tráfico reenviado** | |
    | Permitir tráfico reenviado de la red virtual remota a myAzFwVNet    | Seleccione **Habilitado**. |
    | Permitir tráfico reenviado de myAzFwVNet a la red virtual remota | Seleccione **Habilitado**. |
    | **Configuración de tránsito de la puerta de enlace** | |
    | Permitir tránsito de puerta de enlace | Dejar la opción desactivada |

7. Seleccione **Aceptar**.

## <a name="link-the-virtual-networks-to-the-private-dns-zone"></a>Vincular las redes virtuales a la zona DNS privada

En esta sección, vincularemos las redes virtuales **myVMVNet** y **myAzFwVNet** a la zona DNS privada **privatelink.database.windows.net**. Esta zona se creó cuando se creó el punto de conexión privado. 

El vínculo es necesario para que la máquina virtual y el firewall resuelvan el FQDN de la base de datos en su dirección del punto de conexión privado. La red virtual **myPEVNet** se vinculó automáticamente cuando se creó el punto de conexión privado.

>[!NOTE]
>Si no vincula las redes virtuales de firewall y VM a la zona DNS privada, tanto la máquina virtual como el firewall seguirán pudiendo resolver el FQDN de SQL Server. Se resolverán en su dirección IP pública.

1. En la barra de búsqueda del portal, escriba **privatelink.database**.

2. Seleccione **privatelink.database.windows.net** en los resultados de la búsqueda.

3. Seleccione **Vínculos de red virtual** en **Configuración**.

4. Seleccione **+ Agregar**.

5. En **Agregar el vínculo de red virtual**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre del vínculo | Escriba **Link-to-myVMVNet**. |
    | **Detalles de red virtual** |  |
    | Conozco el id. de recurso de la red virtual  | Deje la opción desactivada.  |
    | Subscription | Seleccione su suscripción.    |
    | Virtual network | Seleccione **myVMVNet**. |
    | **CONFIGURATION** | |
    | Habilitación del registro automático | Deje la opción desactivada.    |


6. Seleccione **Aceptar**.

## <a name="configure-an-application-rule-with-sql-fqdn-in-azure-firewall"></a>Configuración de una regla de aplicación con FQDN de SQL en Azure Firewall

En esta sección, configure una regla de aplicación para permitir la comunicación entre **myVM** y el punto de conexión privado para **mydbserver.database.windows.net** de SQL Server. 

Esta regla permite la comunicación a través del firewall que creamos en los pasos anteriores.

1. En la barra de búsqueda del portal, escriba **myAzureFirewall**.

2. Seleccione **myAzureFirewall** en los resultados de la búsqueda.

3. Seleccione **Reglas** en **Configuración** en la información general de **myAzureFirewall**.

4. Seleccione la pestaña **Recopilación de reglas de aplicación**.

5. Seleccione **+ Agregar una colección de reglas de aplicación**.

6. En **Agregar una colección de reglas de aplicación**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **SQLPrivateEndpoint**. |
    | Prioridad | Escriba **100**. |
    | Acción | Escriba **Permitir**. |
    | **Reglas** |  |
    | **Etiquetas de nombre de dominio completo** | |
    | Nombre  | déjelo en blanco.  |
    | Tipo de origen | Deje el valor predeterminado **Dirección IP**.    |
    | Source | déjelo en blanco. |
    | Etiquetas FQDN | Deje el valor predeterminado **0 seleccionado**. |
    | **FQDN de destino** | |
    | Nombre | Escriba **SQLPrivateEndpoint**.    |
    | Tipo de origen | Deje el valor predeterminado **Dirección IP**. |
    | Source | Escriba **10.1.0.0/16**. |
    | Protocolo: Port | Escriba **mssql:1433**. |
    | FQDN de destino | Escriba **mydbserver.database.windows.net**. |
    |||

7. Seleccione **Agregar**.

## <a name="route-traffic-between-the-virtual-machine-and-private-endpoint-through-azure-firewall"></a>Enrutar el tráfico entre la máquina virtual y el punto de conexión privado a través de Azure Firewall

No creamos un emparejamiento de red virtual directamente entre las redes virtuales **myVMVNet** y **myPEVNet**. La máquina virtual **myVM** no tiene una ruta al punto de conexión privado que creamos. 

En esta sección, crearemos una tabla de rutas con una ruta personalizada. 

La ruta envía el tráfico desde la subred **myVM** al espacio de direcciones de la red virtual **myPEVNet**, a través de Azure Firewall.

1. En el menú de Azure Portal o en la página **Inicio**, seleccione **Crear un recurso**.

2. Escriba **tabla de rutas** en el cuadro de búsqueda y presione **Entrar**.

3. Seleccione **Tabla de rutas** y, luego, **Crear**.

4. En la página **Crear tabla de rutas**, utilice la tabla siguiente para configurar la tabla de rutas:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**.  |
    | **Detalles de instancia** |  |
    | Region | Seleccione **Centro-sur de EE. UU.** . |
    | Nombre | Escriba **VMsubnet-to-AzureFirewall**. |
    | Propagar las rutas de la puerta de enlace | así que seleccione **No**. |

5. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

6. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

7. Cuando finalice la implementación, seleccione **Ir al recurso**.

8. Seleccione **Routes** (Rutas) en **Settings** (Configuración).

9. Seleccione **+Agregar**.

10. En la página **Agregar ruta**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre de ruta | Escriba **myVMsubnet-to-privateendpoint**. |
    | Prefijo de dirección | Escriba **10.2.0.0/16**.  |
    | Tipo de próximo salto | Seleccione **Aplicación virtual**. |
    | Siguiente dirección de salto | Escriba **10.0.0.4**. |

11. Seleccione **Aceptar**.

12. Seleccione **Subredes** en **Configuración**.

13. Seleccione **+ Asociar**.

14. En la página **Asociar subred**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Virtual network | Seleccione **myVMVNet**. |
    | Subnet | Seleccione **VMSubnet**.  |

15. Seleccione **Aceptar**.

## <a name="connect-to-the-virtual-machine-from-your-client-computer"></a>Conectarse a la máquina virtual desde su equipo cliente

Conéctese a la máquina virtual **myVm** desde Internet de la siguiente manera:

1. En la barra de búsqueda del portal, escriba **myVm-ip**.

2. Seleccione **myVM-ip** en los resultados de la búsqueda.

3. Copie o anote el valor en **Dirección IP**.

4. Si usa Windows 10, ejecute el siguiente comando mediante PowerShell. En otras versiones cliente de Windows, use un cliente SSH como [Putty](https://www.putty.org/):

* Reemplace **nombre de usuario** con el nombre de usuario administrador que escribió durante la creación de máquinas virtuales.

* Reemplace **IPaddress** por la dirección IP del paso anterior.

    ```bash
    ssh username@IPaddress
    ```

5. Escriba la contraseña que definió al crear **myVm**.

## <a name="access-sql-server-privately-from-the-virtual-machine"></a>Acceder a SQL Server de forma privada desde la máquina virtual

En esta sección, se conectará de manera privada a SQL Database mediante el punto de conexión privado.

1. Escriba `nslookup mydbserver.database.windows.net`.
    
    Recibirá un mensaje similar a este:

    ```bash
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    mydbserver.database.windows.net       canonical name = mydbserver.privatelink.database.windows.net.
    Name:   mydbserver.privatelink.database.windows.net
    Address: 10.2.0.4
    ```

2. Instale [herramientas de línea de comandos de SQL Server](https://docs.microsoft.com/sql/linux/quickstart-install-connect-ubuntu?view=sql-server-ver15#tools).

3. Ejecute el siguiente comando para conectarse a SQL Server. Use el administrador del servidor y la contraseña que definió cuando creó SQL Server en los pasos anteriores.

* Reemplace **\<ServerAdmin>** por el nombre de usuario del administrador que escribió durante la creación de SQL Server.

* Reemplace **\<YourPassword>** por la contraseña de administrador que escribió durante la creación de SQL Server.

    ```bash
    sqlcmd -S mydbserver.database.windows.net -U '<ServerAdmin>' -P '<YourPassword>'
    ```
4. Se mostrará un símbolo del sistema de SQL en un inicio de sesión correcto. Escriba **salir** para salir de la herramienta **sqlcmd**.

5. Cierre la conexión a **myVM** escribiendo **salir**.

## <a name="validate-the-traffic-in-azure-firewall-logs"></a>Validar el tráfico en registros de Azure Firewall

1. En Azure Portal, seleccione **Todos los recursos** y el área de trabajo de Log Analytics.

2. Seleccione **Registros** en **General** en la página del área de trabajo de Log Analytics.

3. Seleccione el botón **Empezar**.

4. En la ventana **Consultas de ejemplo**, seleccione **Firewalls** en **Todas las consultas**.

5. Seleccione el botón **Ejecutar** en **Datos de registro de regla de aplicación**.

6. En el resultado de la consulta de registro, compruebe que **mydbserver.database.windows.net** aparece en **FQDN** y que **SQLPrivateEndpoint** aparece en **RuleCollection**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de usar los recursos, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba **myResourceGroup** en el cuadro **Buscar** de la parte superior del portal y seleccione **myResourceGroup** en los resultados de búsqueda.

1. Seleccione **Eliminar grupo de recursos**.

1. Escriba **myResourceGroup** para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha explorado los diferentes escenarios que puede usar para restringir el tráfico entre una máquina virtual y un punto de conexión privado mediante Azure Firewall. 

Se conectó a la máquina virtual y se comunicó de forma segura a la base de datos a través de Azure Firewall mediante Private Link.

Para más información sobre el punto de conexión privado, consulte [¿Qué es un punto de conexión privado de Azure?](private-endpoint-overview.md)
