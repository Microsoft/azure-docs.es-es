---
title: 'Tutorial: Conexión de cables a Azure FXT Edge Filer'
description: Cómo cablear los puertos de red y aplicar energía al hardware de Azure FXT Edge Filer
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 07/01/2019
ms.openlocfilehash: 1e3c60fd955bd899955c46b7670acd3f088d0f86
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "84692978"
---
# <a name="tutorial-make-network-connections-and-supply-power-to-the-azure-fxt-edge-filer-node"></a>Tutorial: Establecimiento de las conexiones de red y fuente de alimentación para Azure FXT Edge Filer

En este tutorial le enseñamos a cablear las conexiones de red para un nodo de hardware de sistema de Azure FXT Edge Filer.

En este tutorial, aprenderá a: 

> [!div class="checklist"]
> * Elegir el tipo de cable de red para el entorno
> * Conectar un nodo de Azure FXT Edge Filer a la red del centro de datos
> * Orientar los cables mediante el soporte de orientación de cables (CMA)
> * Conectar la alimentación al dispositivo en bastidor y encenderlo

## <a name="prerequisites"></a>Prerrequisitos

Antes de iniciar este tutorial, se debe instalar Azure FXT Edge Filer en un bastidor de equipamiento estándar. El CMA se debe instalar en el nodo del archivador. 

## <a name="identify-ports"></a>Identificación de los puertos

Identifique los distintos puertos de la parte posterior de Azure FXT Edge Filer. 
 
![Parte posterior de un dispositivo cableado](media/fxt-back-annotated.png)

## <a name="cable-the-device"></a>Cableado del dispositivo

* Conecte los puertos RJ-45 a la fuente de red del centro de datos como se describe en [Puertos de red](#network-ports).  
* Conecte de forma segura el [puerto iDRAC](#idrac-port) a una red independiente con un servidor DHCP seguro. 
* Use los puertos USB y el puerto VGA para conectar un teclado y un monitor con el nodo para la configuración inicial. Debe iniciar el nodo y [establecer una contraseña inicial](fxt-node-password.md) para activar los otros puertos del nodo. Lea [Establecer contraseñas iniciales](fxt-node-password.md) para obtener más información. 

En este artículo también se describe cómo [Conectar la corriente alterna](#connect-power-cables) para el nodo. 

En este artículo también se explica cómo conectarse al [puerto serie](#serial-port-only-when-necessary) del nodo en caso de que sea necesario para la solución de problemas especializada. 

### <a name="network-ports"></a>Puertos de red 

Cada nodo de Azure FXT Edge Filer incluye los siguientes puertos de red: 

* Seis puertos de datos de transferencia dual de alta velocidad de 25 GbE o 10 GbE: 

  * Cuatro puertos proporcionados por dos adaptadores de red de complemento de puerto dual.
  * Dos puertos proporcionados por el adaptador de red intermedio de placa base. 

* Dos puertos de 1 GbE proporcionados por el adaptador de red intermedio de placa base 

Los puertos de datos de alta velocidad de 25 GbE o 10 GbE tienen recipientes estándar compatibles con SFP28. Para usar cables ópticos, debe instalar los módulos del transceptor óptico SFP28 (no proporcionados).

Los puertos de 1 GbE tienen conectores estándar RJ45.

Para una lista completa de los cables, enchufes y transceptores compatibles, consulte [Cavium FastlinQ 41000 Series Interoperability Matrix](https://www.marvell.com/documents/xalflardzafh32cfvi0z/) (Matriz de interoperabilidad de la serie Cavium FastlinQ 41000).

El tipo de conexiones que se usará para el sistema depende del entorno del centro de datos.

* Si se conecta a una red de 25 GbE, cablee cada uno de los puertos de datos de alta velocidad con uno de los siguientes tipos de cable:

  * Cable óptico y transceptor óptico SFP28 con 25 GbE o capacidad de transferencia dual de 25 GbE o 10 GbE
  * Cable biaxial de conexión directa compatible con 25 GbE de tipo SFP28

* Si se conecta a una red de 10 GbE, cablee cada uno de los puertos de datos de alta velocidad con uno de los siguientes tipos de cable: 

  * Cable óptico y transceptor óptico SFP28 con 10 GbE o capacidad de transferencia dual de 25 GbE o 10 GbE
  * Cable biaxial de conexión directa compatible con 25 GbE de tipo SFP28
  * cable biaxial de conexión directa compatible con 10 GbE de tipo SFP28

* Los puertos de red de 1 GbE se usan para el tráfico de administración del clúster. Compruebe la opción **Use 1Gb mgmt network** (Usar red de administración de 1 GB) al crear el clúster si quiere crear una red físicamente independiente para la configuración del clúster [se describe en [Configure the management network](fxt-cluster-create.md#configure-the-management-network) (Configuración de la red de administración)]. Cablee los puertos con Cat5 estándar o un cable mejor tal como se describe en la lista de cables compatibles.

  Puede dejar los puertos de 1 GbE sin cablear si va a usar los puertos de alta velocidad para todo el tráfico. De manera predeterminada, los puertos de red de 1 GbE no se usan si está disponible un puerto de datos de mayor velocidad.  

### <a name="idrac-port"></a>Puerto iDRAC  

El puerto con la etiqueta iDRAC es una conexión de 1GB que permite la comunicación con un controlador de acceso remoto que se usa para la supervisión y administración de hardware. El software FXT usa la interfaz de administración de plataforma inteligente (IPMI) con este controlador para la recuperación y la solución de problemas. Puede usar la [interfaz iDRAC](https://www.dell.com/support/manuals/idrac9-lifecycle-controller-v3.30.30.30/idrac_3.30.30.30_ug/) integrada para supervisar el hardware a través de este puerto. El acceso a IPMI e iDRAC está habilitado de manera predeterminada. 

> [!Note]
> El puerto iDRAC puede omitir el sistema operativo e interactuar directamente con el hardware en el nodo. 

Use estas estrategias de seguridad al conectar y configurar el puerto iDRAC:

* Solo puede conectar puertos iDRAC a una red que esté separada físicamente de la red de datos usada para tener acceso al clúster.
* Establezca una contraseña de administrador de iDRAC segura en cada nodo. Debe establecer esta contraseña para activar el hardware: siga las instrucciones que se indican en [Set hardware passwords](fxt-node-password.md) (Establecimiento de contraseñas de hardware).
* La configuración predeterminada del puerto iDRAC usa DHCP e IPv4 para la asignación de direcciones IP. Asegúrese de que el entorno de DHCP esté bien protegido y que las conexiones estén restringidas entre los clientes DHCP y el servidor DHCP. (El panel de control del clúster incluye opciones de configuración para cambiar el método de configuración de dirección de los nodos después de crear el clúster).
* Deje el puerto iDRAC establecido en "modo dedicado" (valor predeterminado), lo que restringe el tráfico de red iDRAC/IPMI al puerto RJ-45 dedicado.

El puerto iDRAC no requiere una conexión de red de alta velocidad.
  
### <a name="serial-port-only-when-necessary"></a>Puerto serie (solo cuando sea necesario)

En algunas situaciones, el servicio de soporte técnico y atención al cliente de Microsoft puede indicarle que conecte un terminal al puerto serie de un nodo para diagnosticar un problema.  

Para vincular la consola:

1. Busque el puerto serie (COM1) en la parte posterior del nodo de FXT Edge Filer.
1. Use un cable de módem nulo para conectar el puerto serie a un terminal configurado para ANSI-115200-8N1.
1. Inicie sesión en la consola y realice otros pasos tal como indique el personal de soporte técnico.

## <a name="route-cables-in-the-cable-management-arm-cma"></a>Orientar los cables mediante el soporte de orientación de cables (CMA)

Cada nodo de Azure FXT Edge Filer viene con un brazo de administración de cables opcional. El CMA simplifica el enrutamiento de cables y proporciona un acceso más fácil a la parte trasera del chasis sin necesidad de desconectar los cables. 

Siga estas instrucciones para orientar los cables a través del CMA: 

1. Con las bridas proporcionadas, agrupe los cables de forma que entren y salgan de las cestas para que no interfieran en los sistemas adyacentes (1).
1. Con el CMA en la posición de servicio, oriente la agrupación de cables a través de las cestas internas y externas (2).
1. Use las correas anchas con cierre de velcro preinstaladas en cualquier extremo de las cestas para proteger los cables (3).
1. Gire el CMA para que vuelva a su posición en la bandeja (4).
1. Instale el cable del indicador de estado en la parte posterior del sistema y proteja dicho cable mediante su enrutamiento a través del CMA. Conecte el otro extremo del cable a la esquina de la cesta externa del CMA (5). 

   > [!CAUTION]
   > Para evitar posibles daños de cables que sobresalen, proteja las partes sueltas del cable del indicador de estado después de enrutar este cable a través del CMA. 

![Ilustración del CMA con los cables instalados](media/fxt-install/cma-cabling-400.png)

> [!NOTE]
>  Si no instaló el CMA, use las dos correas anchas con cierre de velcro suministradas con el kit del raíl para orientar los cables en la parte posterior del sistema.
> 
>  1. Ubique los soportes del CMA externos en los laterales interiores de las pestañas de los bastidores.
>  2. Agrupe los cables con cuidado, bien extendidos a partir de los conectores del sistema de los laterales izquierdo y derecho.
>  3. Enrosque las correas anchas con cierre de velcro en las ranuras mecanizadas de los soportes del CMA externos de cada lateral del sistema para fijar los cables.
> 
>     ![Cables enrutados sin un CMA](media/fxt-install/fxt-route-cables-no-cma-400.png)

## <a name="about-ip-address-requirements"></a>Acerca de los requisitos de las direcciones IP

Para los nodos de hardware de una caché de almacenamiento híbrido de Azure FXT Edge Filer, el software del clúster administra las direcciones IP.

Cada nodo requiere al menos una dirección IP, pero las direcciones del nodo se asignan cuando los nodos se agregan al clúster o se quitan de este. 

El número total de direcciones IP requeridas depende del número de nodos que componen la memoria caché. 

Configure el intervalo de direcciones IP mediante el software del panel de control una vez instalados los nodos. Para obtener más información, consulte [Recopilación de información del clúster](fxt-cluster-create.md#gather-information-for-the-cluster).  

## <a name="connect-power-cables"></a>Conexión de los cables de alimentación

Cada nodo de Azure FXT Edge Filer usa dos unidades de fuente de alimentación (PSU). 

> [!TIP] 
> Para aprovechar las dos PSU redundantes, conecte cada cable de alimentación de CA a una unidad de distribución de energía (PDU) en un circuito de rama independiente.  
> 
> Puede usar un SAI (UPS) para potenciar las PDU para obtener protección adicional. 

1. Conecte los cables de alimentación incluidos a las PSU del chasis. Asegúrese de que los cables y las PSU estén completamente asentados. 
1. Conecte los cables de alimentación a las unidades de distribución de energía en el bastidor del equipo. Si es posible, use dos fuentes de alimentación independientes para los dos cables. 
 
### <a name="power-on-an-azure-fxt-edge-filer-node"></a>Encendido de un nodo de Azure FXT Edge Filer

Para encender el nodo, presione el botón de encendido en la parte frontal del sistema. El botón está en el lado derecho del panel de control. 

### <a name="power-off-an-azure-fxt-edge-filer-node"></a>Apagado de un nodo de Azure FXT Edge Filer

El botón de encendido puede usarse para apagar el sistema durante las pruebas y antes de agregarlo a un clúster. Sin embargo, después de que un nodo de Azure FXT Edge Filer esté en uso como parte de un clúster, debe usar el software de panel de control del clúster para apagar el hardware. Consulte [How to safely power off Azure FXT Edge Filer hardware](fxt-power-off.md) (Cómo apagar de forma segura el hardware de Azure FXT Edge Filer) para obtener más información. 

## <a name="next-steps"></a>Pasos siguientes

Después de finalizar el cableado del hardware, encienda cada uno de los nodos e inicialícelos mediante el establecimiento de sus contraseñas raíz. 
> [!div class="nextstepaction"]
> [Establecimiento de las contraseñas iniciales](fxt-node-password.md)
