---
title: 'Información general de implementación: Avere vFXT para Azure'
description: Obtenga información sobre cómo implementar un clúster de Avere vFXT for Azure con esta información general. Los artículos relacionados tienen instrucciones de implementación específicas.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 4c63fdf2164dd4dce12912669eec29c79755cc2a
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271233"
---
<!-- filename is linked to in the marketplace template, make sure it gets a redirect if we rename it -->

# <a name="avere-vfxt-for-azure---deployment-overview"></a>Avere vFXT para Azure: información general de la implementación

En este artículo se proporciona información general sobre los pasos necesarios para obtener un clúster funcional de Avere vFXT para Azure.

Se necesitan varias tareas antes y después de crear el clúster de vFXT desde Azure Marketplace. Tener una idea clara de los pasos que debe realizar le ayudará a calcular el esfuerzo necesario para completar el proceso de principio a fin.

## <a name="deployment-steps"></a>Pasos de implementación

Después de [diseñar el sistema](avere-vfxt-deploy-plan.md), puede empezar a crear el clúster de Avere vFXT.

Una plantilla de Azure Resource Manager en Azure Marketplace recopila la información necesaria e implementa automáticamente todo el clúster.

Después de que el clúster de vFXT esté en funcionamiento, quedan algunos pasos de configuración que se deben completar antes de usarlo. Si ha creado un contenedor de almacenamiento de blobs, se recomienda que traslade allí los datos. Si usa un sistema de almacenamiento NAS, deberá agregarlo una vez creado el clúster. Deseará conectar los clientes al clúster.

Aquí encontrará información general de todos los pasos.

1. Configuración de los requisitos previos

   Antes de crear una máquina virtual, debe crear una nueva suscripción para el proyecto de Avere vFXT, configurar la propiedad de la suscripción, comprobar las cuotas y solicitar un aumento si fuera necesario y aceptar los términos para usar el software de Avere vFXT. Lea [Prepare to create the Avere vFXT](avere-vfxt-prereqs.md) (Prepararse para crear una instancia de Avere vFXT) para obtener instrucciones detalladas.

1. Cree el clúster de Avere vFXT

   Use Azure Marketplace para crear el clúster de Avere vFXT for Azure. Una plantilla recopila la información necesaria y ejecuta scripts para crear el producto final.

   La creación del clúster conlleva estos pasos, que se realizan con la plantilla de Marketplace:

   * Crear una nueva infraestructura de red y grupos de recursos, si fuera necesario.
   * Creación de un controlador de clúster

     El controlador de clúster es una máquina virtual sencilla que se encuentra en la misma red virtual que el clúster de Avere vFXT y tiene el software personalizado necesario para crear y administrar el clúster. El controlador crea los nodos de vFXT y da forma al clúster; asimismo, también se encarga de proporcionar una interfaz de línea de comandos para administrar el clúster durante su duración.

     Si crea una red virtual o una subred durante la implementación, el controlador tendrá una dirección IP pública. Esto significa que el controlador puede actuar como host de salto para conectarse al clúster desde fuera de la red virtual.

   * Crear las VM del nodo de clúster.

   * Creación del clúster desde los nodos individuales

   * Opcionalmente, crear un contenedor de blobs y configurarlo como un almacenamiento back-end para el clúster.

   La creación del clúster se describe en detalle en [Implementación del clúster de vFXT](avere-vfxt-deploy.md).

1. Configuración del clúster

   Conéctese a la interfaz de configuración de Avere vFXT (panel de control de Avere) para personalizar la configuración del clúster. Puede participar en la opción de supervisión del soporte técnico y agregar el sistema de almacenamiento si usa almacenamiento de hardware o contenedores de blobs adicionales.

   * [Acceso al clúster de vFXT](avere-vfxt-cluster-gui.md)
   * [Habilitar el soporte técnico](avere-vfxt-enable-support.md)
   * [Configurar el almacenamiento](avere-vfxt-add-storage.md) (si fuera necesario)

1. Agregar clientes

   Siga las instrucciones para [montar el clúster de Avere vFXT ](avere-vfxt-mount-clients.md) si quiere obtener información sobre el equilibrio de carga y cómo los equipos cliente deben usar el clúster.

1. Agregar datos (si fuera necesario)

   Dado que Avere vFXT es una caché multicliente escalable, la mejor manera de mover datos a un nuevo contenedor de almacenamiento de back-end es usar una estrategia de copia de archivos multicliente y multiproceso.

   Si necesita mover los datos del espacio de trabajo a un nuevo contenedor de blobs u otro sistema de almacenamiento de back-end, siga las instrucciones de [Mover datos al clúster de vFXT](avere-vfxt-data-ingest.md).

## <a name="next-steps"></a>Pasos siguientes

Continúe en [Preparación para la creación de Avere vFXT](avere-vfxt-prereqs.md) para completar las tareas de requisitos previos.
