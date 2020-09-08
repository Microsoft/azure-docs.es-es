---
title: 'Tutorial: Filtrado del tráfico de entrada de Internet con la DNAT de Azure Firewall mediante el portal'
description: En este tutorial, aprenderá a implementar y configurar la DNAT de Azure Firewall mediante Azure Portal.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 08/28/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 8f528c6be68258400cb3e29582943f1d657c557d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069279"
---
# <a name="tutorial-filter-inbound-internet-traffic-with-azure-firewall-dnat-using-the-azure-portal"></a>Tutorial: Filtrado del tráfico de entrada de Internet con la DNAT de Azure Firewall mediante Azure Portal

La traducción de direcciones de red de destino (DNAT) de Azure Firewall se puede configurar para traducir y filtrar el tráfico de Internet que entra a las subredes. Al configurar DNAT, en la acción de colección de la regla NAT se selecciona **DNAT**. Luego, todas las reglas de la colección de reglas NAT pueden usarse para traducir la dirección IP y el puerto públicos del firewall a una dirección IP y puerto privados. Las reglas DNAT agregan implícitamente una regla de red correspondiente implícita para permitir el tráfico traducido. Para invalidar este comportamiento, agregue explícitamente una colección de reglas de red con reglas de denegación que coinciden con el tráfico traducido. Para más información acerca de la lógica de procesamiento de reglas Azure Firewall, consulte [Lógica de procesamiento de reglas de Azure Firewall](rule-processing.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar un entorno de red de prueba
> * Implementar un firewall
> * Crear una ruta predeterminada
> * Configurar una regla de DNAT
> * Probar el firewall

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.



## <a name="create-a-resource-group"></a>Crear un grupo de recursos

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En la página principal de Azure Portal, seleccione **Grupos de recursos** y, después, **Agregar**.
3. En **Nombre del grupo de recursos**, escriba **RG-DNAT-Test**.
4. En **Suscripción**, seleccione la suscripción.
5. En **Ubicación del grupo de recursos**: seleccione una ubicación. Todos los recursos posteriores que cree deben estar en la misma ubicación.
6. Seleccione **Crear**.

## <a name="set-up-the-network-environment"></a>Configuración del entorno de red

En este tutorial se crearán dos redes virtuales emparejadas:

- **VN-Hub**: el firewall está en esta red virtual.
- **VN-Spoke**: el servidor de carga de trabajo está en esta red virtual.

En primer lugar, cree las redes virtuales y, después, emparéjelas.

### <a name="create-the-hub-vnet"></a>Creación de la red virtual Hub

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En **Redes**, seleccione **Redes virtuales**.
3. Seleccione **Agregar**.
4. En **Nombre**, escriba **VN-Hub**.
5. En **Espacio de direcciones**, escriba **10.0.0.0/16**.
6. En **Suscripción**, seleccione la suscripción.
7. En **Grupo de recursos**, seleccione **Usar existente** y, después, **RG-DNAT-Test**.
8. En **Ubicación**, seleccione la misma ubicación que usó anteriormente.
9. En **Subred**, como **Nombre** escriba **AzureFirewallSubnet**.

     El firewall estará en esta subred y el nombre de la subred **debe** ser AzureFirewallSubnet.
     > [!NOTE]
     > El tamaño de la subred AzureFirewallSubnet es /26. Para más información sobre el tamaño de la subred, consulte [Preguntas más frecuentes sobre Azure Firewall](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

10. En **Intervalo de direcciones**, escriba **10.0.1.0/26**.
11. Use el resto de valores predeterminados y seleccione **Crear**.

### <a name="create-a-spoke-vnet"></a>Creación de la red virtual Spoke

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En **Redes**, seleccione **Redes virtuales**.
3. Seleccione **Agregar**.
4. En **Nombre**, escriba **VN-Spoke**.
5. En **Espacio de direcciones**, escriba **192.168.0.0/16**.
6. En **Suscripción**, seleccione la suscripción.
7. En **Grupo de recursos**, seleccione **Usar existente** y, después, **RG-DNAT-Test**.
8. En **Ubicación**, seleccione la misma ubicación que usó anteriormente.
9. En **Subred**, en **Nombre** escriba **SN-Workload**.

    El servidor estará en esta subred.
10. En **Intervalo de direcciones**, escriba **192.168.1.0/24**.
11. Use el resto de valores predeterminados y seleccione **Crear**.

### <a name="peer-the-vnets"></a>Emparejamiento de las redes virtuales

Ahora empareje las dos redes virtuales.

1. Seleccione la red virtual **VN-Hub**.
2. En **Configuración**, seleccione **Emparejamientos**.
3. Seleccione **Agregar**.
4. Escriba **Peer-HubSpoke** como **Name of the peering from VN-Hub to VN-Spoke** (Nombre del emparejamiento de centro de VN a radio de VN).
5. Seleccione **VN-Spoke** como red virtual.
6. Escriba **Peer-SpokeHub** como **Name of peering from VN-Spoke to VN-Hub** (Nombre del emparejamiento de radio de VN a centro de VN).
7. En **Allow forwarded traffic from VN-Spoke to VN-Hub** (Permitir el tráfico reenviado de centro de VN a radio de VN), seleccione **Habilitado**.
8. Seleccione **Aceptar**.

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Crear una máquina virtual de cargas de trabajo y colóquela en la subred **SN-Workload**.

1. En el menú de Azure Portal, seleccione **Crear un recurso**.
2. En la página **Popular**, seleccione **Windows Server 2016 Datacenter**.

**Conceptos básicos**

1. En **Suscripción**, seleccione la suscripción.
1. En **Grupo de recursos**, seleccione **Usar existente** y, después, **RG-DNAT-Test**.
1. En **Nombre de máquina virtual**, escriba **Srv-Workload**.
1. En **Región**, seleccione la misma ubicación que usó anteriormente.
1. Escriba un nombre de usuario y una contraseña.
1. Seleccione **Siguiente: Discos**.

**Discos**
1. Seleccione **Siguiente: Redes**.

**Redes**

1. En **Red virtual**, seleccione **VN-Spoke** (Radio de VN).
2. En **Subred**, seleccione **SN-Workload**.
3. En **Dirección IP pública**, seleccione **Ninguna**.
4. En **Puertos de entrada públicos**, seleccione **Ninguno**. 
2. Deje los demás valores predeterminados y seleccione **Siguiente: Administración**.

**Administración**

1. En **Diagnósticos de arranque**, seleccione **Desactivado**.
1. Seleccione **Revisar + crear**.

**Revisar y crear**

Revise el resumen y luego seleccione **Crear**. Esta operación tardará algunos minutos en completarse.

Al finalizar la implementación, anote la dirección IP privada de la máquina virtual, ya que tendrá que usarla más adelante, al configurar el firewall. Seleccione el nombre de la máquina virtual y, en **Configuración**, seleccione **Redes** para buscar la dirección IP privada.

## <a name="deploy-the-firewall"></a>Implementación del firewall

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. Seleccione **Redes** y, después, en **Destacados**, seleccione **Ver todo**.
3. Seleccione **Firewall** y después **Crear**. 
4. En la página **Creación de un firewall**, utilice la tabla siguiente para configurar el firewall:

   |Configuración  |Value  |
   |---------|---------|
   |Nombre     |FW-DNAT-test|
   |Subscription     |\<your subscription\>|
   |Resource group     |**Usar existente**: RG-DNAT-Test |
   |Location     |Seleccione la misma ubicación que usó anteriormente.|
   |Elegir una red virtual     |**Usar existente**: VN-Hub|
   |Dirección IP pública     |**Cree uno nuevo**. La dirección IP pública tiene que ser del tipo de SKU estándar.|

5. Seleccione **Revisar + crear**.
6. Revise el resumen y seleccione **Crear** para crear el firewall.

   La implementación tardará varios minutos.
7. Tras finalizar la implementación, vaya al grupo de recursos **RG-DNAT-Test** y seleccione el firewall **FW-DNAT-test**.
8. Anote la dirección IP privada. Se usará más adelante al crear la ruta predeterminada.

## <a name="create-a-default-route"></a>Crear una ruta predeterminada

En la subred **SN-Workload**, configure la ruta predeterminada de salida para que cruce el firewall.

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En **Redes**, seleccione **Tablas de rutas**.
3. Seleccione **Agregar**.
4. En **Nombre**, escriba **RT-FWroute**.
5. En **Suscripción**, seleccione la suscripción.
6. En **Grupo de recursos**, seleccione **Usar existente** y, después, **RG-DNAT-Test**.
7. En **Ubicación**, seleccione la misma ubicación que usó anteriormente.
8. Seleccione **Crear**.
9. Seleccione **Actualizar** y, después, seleccione la tabla de rutas **RT-FWroute**.
10. Seleccione **Subredes** y, después, seleccione **Asociar**.
11. Seleccione **Red virtual** y, después, seleccione **VN-Spoke**.
12. En **Subred**, seleccione **SN-Workload**.
13. Seleccione **Aceptar**.
14. Seleccione **Rutas** y después **Agregar**.
15. En **Nombre de ruta**, escriba **FW-DG**.
16. En **Prefijo de dirección** escriba **0.0.0.0/0**.
17. En **Tipo del próximo salto**, seleccione **Aplicación virtual**.

    Azure Firewall es realmente un servicio administrado, pero una aplicación virtual funciona en esta situación.
18. En **Dirección del próximo salto**, escriba la dirección IP privada del firewall que anotó anteriormente.
19. Seleccione **Aceptar**.

## <a name="configure-a-nat-rule"></a>Configuración de una regla de NAT

1. Abra **RG-DNAT-Test** y seleccione el firewall **FW-DNAT-test**. 
2. En la página **FW-DNAT-test**, en **Configuración**, seleccione **Reglas**. 
3. Seleccione **Agregar una colección de reglas NAT**. 
4. En **Nombre**, escriba **RC-DNAT-01**. 
5. En **Priority**, escriba **200**. 
6. En **Reglas**, en **Nombre**, escriba **RL-01**.
7. En **Protocolo**, seleccione **TCP**.
8. En **Direcciones de origen**, escriba *. 
9. En **Direcciones de destino** escriba la dirección IP pública del firewall. 
10. En **Puertos de destino**, escriba **3389**. 
11. En **Dirección traducida**, escriba la dirección IP privada de la máquina virtual Srv-Workload. 
12. En **Puerto traducido**, escriba **3389**. 
13. Seleccione **Agregar**. 

## <a name="test-the-firewall"></a>Probar el firewall

1. Conecte un Escritorio remoto a la dirección IP pública del firewall. Debe estar conectado a la máquina virtual **Srv-Workload**.
2. Cierre el Escritorio remoto.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos relacionados con el firewall para el siguiente tutorial o, si ya no los necesita, eliminar el grupo de recursos **RG-DNAT-Test** para eliminarlos todos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Configurar un entorno de red de prueba
> * Implementar un firewall
> * Crear una ruta predeterminada
> * Configurar una regla de DNAT
> * Probar el firewall

A continuación, puede supervisar los registros de Azure Firewall.

> [!div class="nextstepaction"]
> [Tutorial: Supervisión de los registros de Azure Firewall](./tutorial-diagnostics.md)
