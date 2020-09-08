---
title: 'Tutorial: Incorporación de nodos a un clúster de Azure FXT Edge Filer'
description: Aprenda a agregar nodos de clúster a la caché de almacenamiento de Azure FXT Edge Storage y habilitar la característica de alta disponibilidad (HA).
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.openlocfilehash: 5b62927930212fc7e59fc4329a29ceecbe2815e5
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88185339"
---
# <a name="tutorial-add-cluster-nodes-to-an-azure-fxt-edge-filer-cluster"></a>Tutorial: Incorporación de nodos de clúster a un clúster de Azure FXT Edge Filer

Se crea un clúster de Azure FXT Edge Filer con un solo nodo. Debe agregar al menos dos nodos más y habilitar la alta disponibilidad antes de realizar otra configuración. 

En este tutorial se explica cómo agregar nodos de clúster y habilitar la característica de alta disponibilidad (HA). 

En este tutorial, aprenderá a: 

> [!div class="checklist"]
> * Agregar nodos al clúster FXT
> * Habilitar la alta disponibilidad

Los pasos descritos en este tutorial tardan aproximadamente 45 minutos en completarse.

Antes de empezar este tutorial, active los nodos que quiera agregar y [establezca sus contraseñas iniciales](fxt-node-password.md). 

## <a name="1-load-the-cluster-nodes-page"></a>1. Carga de la página de nodos del clúster

Abra el Panel de control del clúster en un explorador web e inicie sesión como administrador. (Las instrucciones detalladas están en el artículo de introducción en [Apertura de las páginas de configuración](fxt-cluster-create.md#open-the-settings-pages)).

El Panel de control muestra la pestaña **Dashboard** (Panel) cuando se abre. 

![Dashboard (Panel) (primera pestaña) del Panel de control](media/fxt-cluster-config/dashboard-1-node.png)

Esta imagen muestra el panel de un clúster recién creado, con un único nodo.

## <a name="2-locate-the-node-to-add"></a>2. Búsqueda del nodo para agregar

Para agregar nodos, haga clic en la pestaña **Settings** (Configuración) y elija la página **FXT Nodes** (Nodos FXT) en la sección **Cluster** (Clúster).

![Pestaña Settings (Configuración) (segunda pestaña) del Panel de control con la página Cluster > FXT Nodes (Clúster > Nodos FXT) cargada](media/fxt-cluster-config/settings-fxt-nodes.png)

La lista **FXT Nodes - Unjoined** (Nodos FXT: no unidos) muestra todos los nodos de FXT sin asignar (la mayoría de los centros de datos solo tienen unos cuantos). Encuentre los nodos FXT que quiere agregar al clúster.

> [!Tip] 
> Si no encuentra el nodo que quiere en la lista **Unjoined** (No unido), compruebe que cumple estos requisitos:
> 
> * Está activado y ha tenido una [contraseña raíz establecida](fxt-node-password.md).
> * Está conectado a una red a la que puede acceder. Si usa redes VLAN, debe estar en la misma VLAN que el clúster.
> * Se puede detectar con el protocolo Bonjour. 
>
>   Algunas opciones de firewall bloquean los puertos TCP/UDP que usa Bonjour, lo que impide que el sistema operativo de FXT detecte automáticamente los nodos.
> 
> Si el nodo que quiere agregar no está en la lista, pruebe estas soluciones: 
> 
> * Haga clic en el botón **Manual Discover** (Detección manual) para buscarlo por dirección IP.
> 
> * Asigne manualmente direcciones IP temporales. Aunque no es algo frecuente, podría ser necesario si usa redes VLAN etiquetadas y los nodos no están en la red correcta o la red no admite direcciones IP asignadas automáticamente. Siga las instrucciones de la versión heredada de este documento para [establecer manualmente una dirección IP estática](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/static_ip.html).

En la lista se muestra el nombre del nodo, la dirección IP, la versión de software y el estado de elegibilidad. Normalmente, en la columna **Status** (Estado) se indica que quiere unirse o se describe un problema de hardware o del sistema que hace que el nodo no sea válido para unirse al clúster.

La columna **Actions** (Acciones) tiene botones que le permiten agregar el nodo al clúster o actualizar su software. El botón de actualización instala automáticamente la versión de software que coincida con los nodos que ya están en el clúster.

Todos los nodos de un clúster deben usar la misma versión del sistema operativo, pero no es necesario actualizar el software antes de agregar un nodo. Tras hacer clic en el botón **Allow To Join** (Permitir unirse), el proceso de unión al clúster comprueba e instala automáticamente el software del sistema operativo que coincide con la versión del clúster.

Para más información sobre las opciones de esta página, lea [**Cluster** > **FXT Nodes**](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_fxt_nodes.html) (Clúster > Nodos FXT) en la guía de configuración del clúster.

## <a name="3-click-the-allow-to-join-button"></a>3. Hacer clic en el botón para unirse 

Haga clic en el botón **Allow To Join*** (Permitir unirse) de la columna **Actions** (Acciones) del nodo que quiere agregar.

Tras hacer clic en el botón, el estado del nodo podría cambiar ya que su software se actualiza como preparación para agregarlo al clúster. 

En la imagen siguiente se muestra un nodo que está en proceso de unirse al clúster (probablemente esté obteniendo una actualización del sistema operativo antes de agregarse). No aparece ningún botón en la columna **Actions** (Acciones) de los nodos que están en proceso de ser agregados al clúster.

![una fila de la tabla de nodos, que muestra el nombre de un nodo, la dirección IP, la versión de software, el mensaje que indica permiso para unirse y una última columna en blanco](media/fxt-cluster-config/node-join-in-process.png)

Transcurridos unos instantes, el nuevo nodo debe aparecer en la lista de nodos del clúster en la parte superior de la página de configuración **FXT Nodes** (Nodos FXT). 

Repita este proceso para agregar los demás nodos al clúster. No es necesario esperar a que un nodo termine de unirse al clúster para iniciar otro.

## <a name="enable-high-availability"></a>Habilitación de alta disponibilidad

Después de haber agregado un segundo nodo al clúster, es posible que vea un mensaje de advertencia en el panel del Panel de control que indica que no está configurada la característica de alta disponibilidad. 

La alta disponibilidad (HA) permite que se compensen entre sí los nodos del clúster en caso de que uno deje de funcionar. De forma predeterminada, esta característica no está habilitada.

![Pestaña Dashboard (Panel) con el mensaje de que el clúster tiene más de un nodo, pero no está habilitada la alta disponibilidad en la tabla Conditions (Condiciones)](media/fxt-cluster-config/no-ha-2-nodes.png)

> [!Note] 
> No habilite la alta disponibilidad hasta que haya al menos tres nodos en el clúster.

Siga este procedimiento para activar la alta disponibilidad: 

1. Cargue la página **High Availability** (Alta disponibilidad) de la sección **Cluster** (Clúster) de la pestaña **Settings** (Configuración).

   ![Página de configuración de alta disponibilidad (Cluster > High Availability [Clúster > Alta disponibilidad]). La casilla "Enable HA" (Habilitar la alta disponibilidad) se encuentra en la parte superior y el botón de envío en la parte inferior.](media/fxt-cluster-config/enable-ha.png)

2. Haga clic en la casilla **Enable HA** (Habilitar la alta disponibilidad) y haga clic en el **Submit** (Enviar). 

Aparece una alerta en el **panel** para confirmar que está habilitada la alta disponibilidad.

![Tabla del panel que muestra el mensaje de que la alta disponibilidad está ahora completamente configurada](media/fxt-cluster-config/ha-configured-alert.png)


## <a name="next-steps"></a>Pasos siguientes

Después de agregar todos los nodos del clúster, continúe con la configuración del almacenamiento a largo plazo del clúster.

> [!div class="nextstepaction"]
> [Adición de almacenamiento back-end y configuración del espacio de nombres virtual](fxt-add-storage.md)