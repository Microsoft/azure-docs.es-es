---
title: 'Tutorial: Creación del clúster de caché de Azure FXT Edge Filer'
description: Aprenda a crear un clúster de caché de almacenamiento híbrido con Azure FXT Edge Filer y a iniciar sesión en el panel de control del clúster para configurar los valores del clúster.
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 07/01/2019
ms.openlocfilehash: 7a471868bac8f5e0623942c0cc1dc4af4e3881e7
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88185356"
---
# <a name="tutorial-create-the-azure-fxt-edge-filer-cluster"></a>Tutorial: Creación de clústeres de Azure FXT Edge Filer

Después de instalar e inicializar los nodos de hardware de Azure FXT Edge Filer de su caché, use el software del clúster de FXT para crear el clúster de caché. 

Este tutorial le guía por los pasos para configurar los nodos de hardware como un clúster. 

En este tutorial, aprenderá a: 

> [!div class="checklist"]
> * La información que es necesaria antes de comenzar a crear el clúster
> * La diferencia entre la red de administración del clúster, la red en clúster y la red orientada al cliente
> * Cómo conectarse a un nodo de clúster 
> * Cómo crear un clúster inicial con un nodo de Azure FXT Edge Filer
> * Cómo iniciar sesión en el panel de control del clúster para configurar los valores del clúster

Este procedimiento tarda entre 15 y 45 minutos, según cuánta investigación se deba llevar a cabo para identificar las direcciones IP y los recursos de red.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, complete estos requisitos previos:

* Instale sus sistemas de hardware de Azure FXT Edge Filer en su centro de datos. 

  Aunque solo necesita un nodo para crear el clúster, debe [agregar al menos dos nodos más](fxt-add-nodes.md) para poder configurar el clúster y prepararlo para su uso. 

* Conecte los cables de alimentación y de red adecuados al sistema.  
* Encienda al menos un nodo de Azure FXT Edge Filer y [establezca su contraseña raíz](fxt-node-password.md).

## <a name="gather-information-for-the-cluster"></a>Recopilación de información del clúster 

Para crear el clúster de Azure FXT Edge Filer, necesita la siguiente información:

* Nombre del clúster

* Contraseña administrativa establecida para el clúster

* Direcciones IP:

  * Una única dirección IP para la administración del clúster, y la máscara de red y el enrutador que se usarán para la red de administración
  * Las direcciones IP primera y última de un intervalo contiguo de direcciones IP para la comunicación (nodo a nodo) del clúster. Consulte a continuación [Distribución de direcciones IP](#ip-address-distribution) para más información. 
  * (Las direcciones IP orientadas al cliente se establecen después de la creación del clúster).

* Información de la infraestructura de red:

  * La dirección IP de un servidor DNS del clúster
  * El nombre del dominio DNS del clúster
  * El nombre o la dirección IP de los servidores NTP del clúster (uno o tres servidores o más) 
  * Si quiere habilitar la adición de enlaces IEEE 802.1AX-2008 en las interfaces del clúster
  * Si habilita la adición de enlaces, si se usará o no la adición dinámica IEEE 802.3ad (LACP)

Puede configurar estos elementos de la infraestructura de red después de crear el clúster, pero es preferible en el momento de la creación. 

### <a name="ip-address-distribution"></a>Distribución de direcciones IP

El clúster de caché de almacenamiento híbrido de Azure FXT Edge Filer usa direcciones IP en tres categorías:

* IP de administración: una única dirección IP para la administración del clúster

  Esta dirección sirve de punto de entrada a las utilidades de configuración del clúster (el panel de control basado en web o la API de XML-RPC). Esta dirección se asigna automáticamente al nodo principal del clúster y se mueve automáticamente si cambia el nodo principal.

  Aunque se pueden usar otras direcciones IP para acceder al panel de control, la dirección IP de administración está diseñada para proporcionar acceso incluso si algunos nodos conmutan por error.

* Red en clúster: un intervalo de direcciones IP para la comunicación del clúster

  La red en clúster se usa para la comunicación entre los nodos del clúster y para recuperar archivos del almacenamiento back-end (archivadores principales).

  **Procedimiento recomendado:** Asigne una dirección IP por puerto físico usado para la comunicación del clúster a cada nodo de Azure FXT Edge Filer. El clúster asigna automáticamente las direcciones del intervalo especificado a cada nodo.

* Red orientada al cliente: el intervalo de direcciones IP que usan los clientes para solicitar y escribir archivos

  Los clientes usan las direcciones de red del cliente para acceder a los datos del archivador principal mediante el clúster. Por ejemplo, un cliente NFS podría montar una de estas direcciones.

  **Procedimiento recomendado:** asigne una dirección IP por puerto físico usado para la comunicación del cliente a cada nodo de la serie FXT.

  El clúster distribuye las direcciones IP orientadas al cliente entre sus nodos constituyentes lo más uniformemente posible.

  Por motivos de simplicidad, muchos administradores configuran un solo nombre DNS con configuración de DNS (RRDNS) round-robin para que sean más fáciles de distribuir las solicitudes de cliente entre el intervalo de direcciones. Esta configuración también permite que todos los clientes usen el mismo comando de montaje para acceder al clúster. Para más información, lea [Configuración de DNS](fxt-configure-network.md#configure-dns-for-load-balancing).

Para crear un clúster, se debe especificar la dirección IP de administración y un intervalo de direcciones de red del clúster. Las direcciones orientadas al cliente se especifican tras la creación del clúster.

## <a name="connect-to-the-first-node"></a>Conexión al primer nodo

Puede conectarse a cualquiera de los nodos FXT instalados y usar su software del sistema operativo para configurar el clúster.

Si aún no lo ha hecho, encienda al menos uno de los nodos FXT del clúster y asegúrese de que tenga una conexión de red y una dirección IP. También, si aún no lo ha hecho, debe establecer una nueva contraseña raíz para activar el nodo, así que siga los pasos que se describen en [Set hardware passwords](fxt-node-password.md) (Establecimiento de las contraseñas de hardware).

Para comprobar la conexión de red, asegúrese de que los LED del vínculo de red del nodo se iluminan (y, si es necesario, que los indicadores de la red cambian para reflejar lo que está conectado). Los indicadores LED se describen en [Monitor Azure FXT Edge Filer hardware status](fxt-monitor.md) (Supervisión del estado de hardware de Azure FXT Edge Filer).

Cuando arranque el nodo, solicitará una dirección IP. Si el nodo está conectado a un servidor DHCP, acepta la dirección IP que le proporciona DHCP. (Esta dirección IP es temporal. Cambiará al crear el clúster).

Si no está conectado a un servidor DHCP o no recibe una respuesta, el nodo usará el software Bonjour para establecer una dirección IP asignada automáticamente con el formato 169.254. \*. \*. Sin embargo, debe establecer una dirección IP estática temporal en una de las tarjetas de red del nodo para poder usarlo en la creación de un clúster. Se pueden encontrar instrucciones en este documento heredado, pero para obtener información actualizada, póngase en contacto con el servicio de soporte técnico de Microsoft: [Appendix A: Setting a Static IP Address on an FXT Node](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/static_ip.html) (Apéndice A: Establecimiento de una dirección IP estática en un nodo FXT).

### <a name="find-the-ip-address"></a>Búsqueda de la dirección IP

Conéctese al nodo de Azure FXT Edge Filer para encontrar su dirección IP. Puede usar un cable serie, la conexión directa a los puertos USB y VGA o conectarse mediante un conmutador KVM. (Para información sobre la conexión a los puertos, consulte [Set initial passwords](fxt-node-password.md) [Establecimiento de las contraseñas iniciales]).

Después de conectarse, inicie sesión con el nombre de usuario `root` y la contraseña que estableció al arrancar el nodo por primera vez.  

Después de iniciar sesión, debe determinar la dirección IP del nodo.

Use el comando `ifconfig` para ver las direcciones asignadas a este sistema.

Por ejemplo, el comando `ifconfig | grep -B5 inet` busca puertos con direcciones de Internet y proporciona cinco líneas de contexto para mostrar el identificador del puerto.

Anote las direcciones IP que se muestran en el informe de ifconfig. Las direcciones indicadas con nombres de puerto, como e0a o e0b, son buenas opciones. No use ninguna dirección IP que aparezca con los nombres e7 *, ya que esos nombres solo se usan para los puertos de servicio iDRAC/IPMI.  

## <a name="load-the-cluster-configuration-wizard"></a>Carga del asistente para configuración del clúster

Use la herramienta de configuración del clúster basada en explorador para crear el clúster. 

Escriba la dirección IP del nodo en un explorador web. Si el explorador muestra un mensaje que indica que el sitio no es de confianza, continúe con el sitio de todos modos. (Los nodos individuales de Azure FXT Edge Filer no tienen certificados de seguridad emitidos por una entidad de seguridad [CA]).

![Página de inicio de sesión del panel de control en la ventana del explorador](media/fxt-cluster-create/unconfigured-node-gui.png)

Deje en blanco los campos **Username** (Nombre de usuario) y **Password** (Contraseña). Haga clic en **Login** (Inicio de sesión) para cargar la página de creación del clúster.

![Pantalla de la configuración inicial de un nodo no configurado en el panel de control de la interfaz gráfica de usuario basada en explorador. Muestra opciones para actualizar el software, configurar un clúster de forma manual o configurar un clúster a partir de un archivo de configuración.](media/fxt-cluster-create/setup-first-screen.png)

## <a name="create-the-cluster"></a>Creación del clúster

La herramienta de configuración del clúster le guía por un conjunto de pantallas para crear el clúster de Azure FXT Edge Filer. Asegúrese de tener preparada la [información necesaria](#gather-information-for-the-cluster) antes de comenzar. 

### <a name="creation-options"></a>Opciones de creación

La primera pantalla ofrece tres opciones. Use la opción de configuración manual a menos que tenga instrucciones especiales del personal de soporte técnico.

Haga clic en **I will configure the cluster manually** (Configuraré el clúster manualmente) para cargar la nueva pantalla de opciones de configuración del clúster. 

Las demás opciones apenas se usan:

* "Update the system image" (Actualizar la imagen del sistema) le pregunta si desea instalar nuevo software del sistema operativo antes de crear el clúster. (La versión del software instalada actualmente se muestra en la parte superior de la pantalla). Debe proporcionar el archivo de paquete de software: bien una dirección URL y el nombre de usuario y la contraseña, o cargar un archivo desde el equipo. 

* En ocasiones el Servicio de soporte técnico y Atención al cliente de Microsoft usa la opción de archivo de configuración del clúster. 

## <a name="cluster-options"></a>Opciones del clúster

La pantalla siguiente le pide que configure opciones para el nuevo clúster.

La página se divide en dos secciones principales, **Basic Configuration** (Configuración básica) y **Networking Configuration** (Configuración de red). La sección de configuración de red también tiene subsecciones: una para la red de **administración** y otra para la red **en clúster**.

### <a name="basic-configuration"></a>Configuración básica

En la sección superior, rellene la información básica del nuevo clúster.

![Detalles de la sección de configuración básica en la página de la interfaz gráfica de usuario del explorador. Muestra tres campos (nombre del clúster, contraseña de administrador y confirmación de contraseña).](media/fxt-cluster-create/basic-configuration.png) 

* **Cluster Name** (Nombre del clúster): escriba un nombre único para el clúster.

  El nombre del clúster debe cumplir estos criterios:
  
  * Longitud de 1 a 16 caracteres.
  * Puede incluir letras, números, guiones (-) y caracteres de subrayado (_). 
  * No debe contener otros signos de puntuación o caracteres especiales.
  
  Puede cambiar este nombre más adelante en la página de configuración **Cluster** > **General Setup** (Clúster > Configuración general). (Para más información sobre la configuración del clúster, lea la [guía de configuración del clúster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/ops_conf_index.html), que no forma parte de este conjunto de documentación).

  > [!NOTE] 
  > El nombre del clúster se usa para identificar la información del sistema cargada para permitir la supervisión o la solución de problemas, por lo que es útil incluir el nombre de la empresa.

* **Admin password** (Contraseña de administrador): establezca la contraseña del usuario administrativo predeterminado, `admin`.
  
  Puede configurar cuentas de usuario individuales para cada persona que administre el clúster, pero no puede quitar el usuario `admin`. Inicie sesión como `admin` si necesita crear usuarios adicionales.
 
  Puede cambiar la contraseña del usuario `admin` en la página de configuración **Administration** > **Users** (Administración > Usuarios) del Panel de control del clúster. Para más información, lea la documentación de **Users** (Usuarios) en la [guía de configuración del clúster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_users.html).

<!-- to do: update "legacy" URLs when docs are ported to Microsoft site -->

### <a name="network-configuration"></a>Network configuration (Configuración de red)

La sección **Networking** (Redes) le pide que especifique la infraestructura de red que usará el clúster. 

Hay dos redes diferentes para configurar:

* La *red de administración* proporciona al administrador acceso al clúster para tareas de configuración y supervisión. La dirección IP especificada aquí se usa al conectarse al Panel de control o para el acceso SSH. 

  La mayoría de los clústeres usan una sola dirección IP de administración, pero si quiere agregar interfaces puede hacerlo después de crear el clúster.

* La *red en clúster* se usa para la comunicación entre los nodos del clúster y entre los nodos del clúster y el almacenamiento back-end (archivadores principales).

La red orientada al cliente se configura más adelante, después de crear el clúster.

En esta sección también se incluye la configuración de los servidores DNS y NTP que se usan en ambas redes.

### <a name="configure-the-management-network"></a>Configuración de la red de administración

La configuración de la sección **Management** (Administración) es para la red que proporciona al administrador acceso al clúster.

![Detalles de la sección de administración, con los campos de cinco opciones y una casilla para la red de administración de 1 Gb](media/fxt-cluster-create/management-network-filled.png)

* **Management IP** (Administración de IP): especifique la dirección IP que usará para acceder al Panel de control del clúster. El nodo principal del clúster solicitará esta dirección, que pasa automáticamente a un nodo en buen estado en caso de que el nodo principal original no se encuentre disponible.

  La mayoría de clústeres usan solo una dirección IP de administración. Si quiere más de una, puede agregarlas después de crear el clúster mediante la página de configuración **Cluster** > **Administrative Network** (Clúster > Red administrativa). Más información en la [guía de configuración del clúster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_admin_network.html).

* **Netmask** (Máscara de red): especifique la máscara de red de la red de administración.

* **Router** (Enrutador): escriba la dirección de puerta de enlace predeterminada usada en la red de administración.

* **VLAN tag (optional)** (Etiqueta VLAN [opcional]): si el clúster usa etiquetas VLAN, especifique la etiqueta de la red de administración.

  La configuración de VLAN adicional se realiza en la página de configuración **Cluster** > **VLAN** (Clúster > VLAN). Lea [Working with VLANs](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/network_overview.html#vlan-overview) (Uso de redes VLAN) y [Cluster > VLAN](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_vlan.html) (Clúster > VLAN) en la guía de configuración del clúster para más información.

* **MTU**: si es necesario, ajuste la unidad de transmisión máxima (MTU) de la red de administración del clúster.

* **Use 1Gb mgmt network** (Usar red de administración de 1 GB): active esta casilla si quiere asignar los dos puertos de red de 1 GbE de los nodos FXT solo a la red de administración. (Debe tener puertos de 25 GbE/10 GbE disponibles para todo el tráfico). Si no activa esta casilla, la red de administración usa el puerto de velocidad más alto disponible. 

### <a name="configure-the-cluster-network"></a>Configuración de la red en clúster 

La configuración de la red en clúster se aplica al tráfico entre los nodos del clúster y entre los nodos del clúster y los archivadores principales.

![detalles de la sección del clúster, con campos para escribir seis valores](media/fxt-cluster-create/cluster-network-filled.png)

* **First IP** (Primera IP) y **Last IP** (Última IP): escriba las direcciones IP que definen el intervalo que se usará para la comunicación interna del clúster. Las direcciones IP usadas aquí deben ser contiguas y no se deben asignar mediante DHCP.

  Puede agregar más direcciones IP después de crear el clúster. Use la página de configuración **Cluster** > **Cluster Networks** (Clúster > Redes en clúster) ([documentación de la guía de configuración del clúster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cluster_networks.html#gui-cluster-networks)).

  El valor de **Number of IPs in Range** (Número de direcciones IP del intervalo) se calcula y se muestra automáticamente.

* **Non-mgmt netmask (optional)** (Sin máscara de red de administración [opcional]): especifique la máscara de red de la red en clúster. 

  El sistema sugiere automáticamente el valor de máscara de red que especificó para la red de administración; cámbielo si es necesario.

* **Cluster router (optional)** (Enrutador del clúster [opcional]): especifique la dirección de puerta de enlace predeterminada que usa la red en clúster. 

  El sistema sugiere automáticamente la misma dirección de puerta de enlace que ha proporcionado para la red de administración.

* **Cluster VLAN tag (optional)** (Etiqueta VLAN del clúster [opcional]): si el clúster usa etiquetas VLAN, especifique la etiqueta de la red en clúster.

* **Non-mgmt MTU (optional)** (Sin MTU de administración [opcional]): si es necesario, ajuste la unidad de transmisión máxima (MTU) de la red en clúster.

### <a name="configure-cluster-dns-and-ntp"></a>Configuración de DNS y NTP del clúster 

Debajo de la sección **Cluster** (Clúster), hay campos para especificar los servidores DNS y NTP y para habilitar la adición de enlaces. Esta configuración se aplica a todas las redes que usa el clúster.

![Detalles de la sección de configuración de DNS y NTP, con tres campos para los servidores DNS, campos para el dominio DNS y la búsqueda de DNS, tres campos para los servidores NTP y un menú desplegable para las opciones de adición de enlaces](media/fxt-cluster-create/dns-ntp-filled.png)

* **DNS server(s)** (Servidores DNS): escriba la dirección IP de uno o varios servidores del sistema de nombres de dominio (DNS).

  DNS se recomienda para todos los clústeres y es necesario si quiere usar SMB, AD o Kerberos. 
  
  Para obtener un rendimiento óptimo, configure el servidor DNS del clúster para el equilibrio de carga round robin, como se describe en [Configuración de DNS para el clúster de Azure FXT Edge Filer](fxt-configure-network.md#configure-dns-for-load-balancing).

* **DNS domain** (Dominio DNS): escriba el nombre de dominio de red que usará el clúster.

* **DNS search** (Búsqueda DNS): opcionalmente, escriba nombres de dominio adicionales que el sistema debe buscar para resolver consultas de DNS. Puede agregar hasta seis nombres de dominio, separados por espacios.

* **NTP server(s)** (Servidores NTP): especifique uno o tres servidores de protocolo de tiempo de red (NTP) en los campos proporcionados. Puede usar nombres de host o direcciones IP.

* **Link aggregation** (Adición de enlaces): la adición de enlaces le permite personalizar cómo los puertos ethernet de los nodos del clúster FXT controlan varios tipos de tráfico. Para más información, lea [Link Aggregation](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cluster_general_setup.html#link-aggregation) (Adición de enlaces) en la guía de configuración del clúster.

### <a name="click-the-create-button"></a>Hacer clic en el botón de creación

Después de proporcionar toda la configuración necesaria en el formulario, haga clic en el botón **Create Cluster** (Crear clúster).

![parte inferior del formulario completado con el cursor sobre el botón de creación en la esquina inferior derecha](media/fxt-cluster-create/create-cluster.png)

El sistema muestra un mensaje mientras se crea el clúster.

![mensaje de estado de configuración del clúster en el explorador: "The FXT node is now creating the cluster. Esto tardará varios minutos. When the cluster is created, visit this link to complete the configuration." (El nodo FXT está creando el clúster. Esta operación tardará varios minutos. Cuando se cree el clúster, visite este vínculo para finalizar la configuración) con un hipervínculo en "visit this link" (visite este vínculo)](media/fxt-cluster-create/creating-message.png)

Al cabo de un rato, puede hacer clic en el vínculo del mensaje para ir al Panel de control del clúster. (Este vínculo le lleva a la dirección IP que especificó en **Management IP** [IP de administración]). El vínculo tarda entre 15 segundos y 1 minuto en activarse después de hacer clic en el botón Create (Crear). Si no se carga la interfaz web, espere varios segundos más y, luego, vuelva a hacer clic en el vínculo. 

La creación del clúster tarda un minuto o más, pero puede iniciar sesión en el Panel de control mientras el proceso está en marcha. Es normal que la página del panel del Panel de control muestre advertencias hasta que finalice el proceso de creación del clúster. 

## <a name="open-the-settings-pages"></a>Apertura de las páginas de configuración 

Después de crear el clúster, debe personalizar su configuración para la red y el flujo de trabajo. 

Use la interfaz web del Panel de control para configurar el nuevo clúster. Siga el vínculo de la pantalla de estado de creación del clúster o vaya a la dirección IP de administración que definió en el clúster.

Inicie sesión en la interfaz web con el nombre de usuario `admin` y la contraseña que estableció al crear el clúster.

![explorador web que muestra los campos de inicio de sesión del Panel de control](media/fxt-cluster-create/admin-login.png)

El Panel de control se abre y muestra la página **Dashboard** (Panel). Cuando finalice la creación del clúster, los mensajes de advertencia se borrarán de la pantalla.

Haga clic en **Settings** (Configuración) para configurar el clúster.

En la pestaña **Settings** (Configuración), en la barra lateral izquierda se muestra un menú de páginas de configuración. Las páginas se organizan por categoría. Haga clic en el control + o - en la parte superior del nombre de la categoría para expandir u ocultar cada página.

![Pestaña de configuración del Panel de control (en el explorador) con la página Cluster > General Setup (Clúster > Configuración general) cargada](media/fxt-cluster-create/settings-tab-populated.png)

## <a name="cluster-setup-steps"></a>Pasos de configuración del clúster

En esta fase del proceso, el clúster existe, pero solo tiene un nodo, no hay direcciones IP orientadas al cliente y no hay almacenamiento back-end. Se necesitan pasos de configuración adicionales para pasar de un clúster recién creado a un sistema de caché que esté preparado para controlar el flujo de trabajo.

### <a name="required-configuration"></a>Configuración necesaria

Estos pasos son necesarios para todos los clústeres o la mayoría de ellos. 

* Adición de nodos al clúster 

  Aunque lo normal son tres nodos, muchos clústeres de producción tienen más, hasta 24 nodos como máximo.

  Lea [Agregar nodos al clúster](fxt-add-nodes.md) para aprender a agregar otras unidades de Azure FXT Edge Filer al clúster y para habilitar la alta disponibilidad.

* Especificación del almacenamiento back-end

  Agregue definiciones de *archivador principal* para cada sistema de almacenamiento back-end que usará el clúster. Lea [Adición de almacenamiento de back-end y configuración del espacio de nombres virtual](fxt-add-storage.md#about-back-end-storage) para más información.

* Configuración del acceso de cliente y del espacio de nombres virtual 

  Cree al menos un servidor virtual (vserver) y asígnele un intervalo de direcciones IP para su uso por las máquinas cliente. También debe configurar el espacio de nombres del clúster (a veces denominado espacio de nombres global o GNS), una característica del sistema de archivos virtual que le permite asignar exportaciones del almacenamiento back-end a rutas de acceso virtuales. El espacio de nombres del clúster ofrece a los clientes una estructura de sistema de archivos coherente y accesible incluso si se cambia de medios de almacenamiento back-end. Asimismo, el espacio de nombres puede proporcionar también una jerarquía de almacenamiento virtual fácil de usar para contenedores de blobs de Azure u otro almacenamiento de objetos en la nube admitido.

  Lea [Configure the namespace](fxt-add-storage.md#configure-the-namespace) (Configuración del espacio de nombres) para más información. Este paso incluye:
  * Crear servidores virtuales
  * Configurar puntos de unión entre la vista de red del cliente y el almacenamiento back-end 
  * Definir qué direcciones IP de cliente atiende cada servidor virtual

  > [!Note] 
  > Antes de empezar a configurar el GNS del clúster, se recomienda planear bien las cosas. Lea las secciones [Using a Global Namespace](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gns_overview.html) (Uso de un espacio de nombres global) y [Creating and Working with VServers](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#creating-and-working-with-vservers) (Creación y uso de servidores virtuales) de la guía de configuración del clúster para obtener ayuda.

* [Ajuste de la configuración de red](fxt-configure-network.md)

  Hay varias configuraciones relacionadas con la red que se deben comprobar o personalizar cuando se crea un clúster. Lea [Ajuste de la configuración de red](fxt-configure-network.md) para más información sobre estos elementos:

  * Comprobación de la configuración de DNS y NTP 
  * Configuración de los servicios de directorio, si es necesario 
  * Configuración de las redes VLAN
  * Configuración de los servidores proxy
  * Adición de direcciones IP a la red en clúster
  * Almacenamiento de los certificados de cifrado

* [Configuración de la supervisión del soporte técnico](#enable-support)

  Debe aceptar la directiva de privacidad de la herramienta de configuración y debe configurar al mismo tiempo los valores de carga de soporte técnico.

  El clúster puede cargar automáticamente los datos de solución de problemas sobre el clúster, incluidas las estadísticas y los archivos de depuración. Estas operaciones de carga permiten al Servicio de soporte técnico y Atención al cliente de Microsoft proporcionar el mejor servicio posible. Puede personalizar lo que se supervisa y, opcionalmente, habilitar el soporte técnico proactivo y el servicio de solución de problemas remoto.  

### <a name="optional-configuration"></a>Configuración opcional

Estos pasos no son necesarios para todos los clústeres. Son necesarios para algunos tipos de flujos de trabajo o para algunos estilos de administración del clúster. 

* Personalización de la configuración de los nodos

  Puede establecer los nombres de los nodos y configurar sus puertos IPMI a nivel global en todo el clúster o de forma individual. Si configura estas opciones antes de agregar nodos al clúster, los nuevos nodos pueden seleccionar automáticamente la configuración cuando se unen. Las opciones se describen en el documento heredado de creación del clúster [Customizing Node Settings](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_node.html) (Personalización de la configuración del nodo).

  > [!TIP]
  > Parte de la documentación relacionada con este producto no se encuentra aún disponible en el sitio de documentación de Microsoft Azure. Los vínculos a la [guía de configuración del clúster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/ops_conf_index.html) y a la versión heredada de la [guía de creación del clúster](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/create_index.html) le llevarán a un sitio web independiente hospedado en GitHub. 

* Configuración de SMB

  Si quiere permitir acceso SMB al clúster, así como NFS, debe configurar SMB y activarlo. SMB (a veces denominado CIFS) se usa normalmente para admitir clientes de Microsoft Windows.

  El planeamiento y la configuración de SMB suponen algo más que hacer clic en unos cuantos botones del Panel de control. Según los requisitos del sistema, SMB puede influir en cómo se definen los archivadores principales, cuántos servidores virtuales se crean, cómo se configurar los puntos de unión, el espacio de nombres y los permisos de acceso, y otras configuraciones.

  Para más información, lea la sección [Configuring SMB Access](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/smb_overview.html) (Configuración del acceso SMB) de la guía de configuración del clúster.

* Instalación de licencias adicionales

  Si quiere usar un almacenamiento en la nube distinto de los blobs de Azure, debe instalar una licencia de característica adicional. Póngase en contacto con su representante de Microsoft para más información sobre la compra de una licencia FlashCloud<sup>TM</sup>. En [Adición de almacenamiento de back-end y configuración del espacio de nombres virtual](fxt-add-storage.md#about-back-end-storage) se explican los detalles.


### <a name="enable-support"></a>Habilitación del soporte técnico

El clúster de Azure FXT Edge Filer puede cargar automáticamente datos de soporte técnico sobre el clúster. Estas cargas hacen posible que el personal proporcione el mejor servicio posible al cliente.

Siga estos pasos para configurar las cargas de soporte técnico.

1. Vaya a la página de configuración **Cluster** > **Support** (Clúster > Soporte técnico). Acepte la directiva de privacidad. 

   ![Captura de pantalla que muestra el Panel de control y la ventana emergente con el botón de confirmación para aceptar la directiva de privacidad](media/fxt-cluster-create/fxt-privacy-policy.png)

1. Haga clic en el triángulo situado a la izquierda de **Información del cliente** para expandir la sección.
1. Haga clic en el botón **Revalidate upload information** (Revalidar información de carga).
1. Establezca el nombre de soporte del clúster en **Nombre de clúster único**, asegúrese de que identifica de manera única a su clúster para el personal de apoyo.
1. Active las casillas de **Statistics Monitoring** (Supervisión de estadísticas), **General Information Upload** (Carga de información general) y **Crash Information Upload** (Carga de información de bloqueo).
1. Haga clic en **Enviar**.  

   ![Captura de pantalla que contiene la sección de información del cliente completada en la página de configuración de soporte técnico](media/fxt-cluster-create/fxt-support-info.png)

1. Haga clic en el triángulo situado a la izquierda de **Secure Proactive Support (SPS)** (Soporte técnico proactivo seguro (SPS)) para expandir la sección.
1. Active la casilla **Enable SPS Link** (Habilitar vínculo SPS).
1. Haga clic en **Enviar**.

   ![Captura de pantalla que contiene la sección de soporte técnico proactivo completada en la página de configuración de soporte técnico](media/fxt-cluster-create/fxt-support-sps.png)

## <a name="next-steps"></a>Pasos siguientes

Después de haber creado el clúster básico y de haber aceptado la directiva de privacidad, agregue el resto de los nodos del clúster. 

> [!div class="nextstepaction"]
> [Adición de nodos de clúster](fxt-add-nodes.md)
