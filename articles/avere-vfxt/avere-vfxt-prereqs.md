---
title: Requisitos previos de Avere vFXT - Azure
description: Requisitos previos de Avere vFXT para Azure
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/21/2020
ms.author: rohogue
ms.openlocfilehash: 538f3f18e1c8889834b9ee6588b85c9094dfbebe
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496719"
---
# <a name="prepare-to-create-the-avere-vfxt"></a>Preparación para la creación de Avere vFXT

En este artículo se explican las tareas previas necesarias para crear un clúster de Avere vFXT.

## <a name="create-a-new-subscription"></a>Crear una nueva suscripción

Comience creando nueva suscripción de Azure. Use una suscripción independiente para cada proyecto de Avere vFXT con el fin de simplificar la limpieza y el seguimiento de los gastos, y para asegurarse de que otros proyectos no se vean afectados por las cuotas o por la limitación de recursos al aprovisionar el flujo de trabajo del clúster.

Para crear una nueva suscripción de Azure en Azure Portal:

1. Vaya a la [hoja de suscripciones](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).
1. Haga clic en el botón **+ Agregar** de la parte superior.
1. Inicie sesión si es necesario.
1. Seleccione una oferta y siga los pasos para crear una nueva suscripción.

## <a name="configure-subscription-owner-permissions"></a>Configurar los permisos de propietario de la suscripción

Un usuario con permisos de propietario de la suscripción debe encargarse de crear el clúster de vFXT. La creación del clúster requiere que un propietario acepte los términos de servicio del software y autorice los cambios en los recursos de red y de almacenamiento.

Hay algunas soluciones alternativas que permiten a quien no sea propietario crear una instancia de Avere vFXT para el clúster de Azure. Estos escenarios implican restringir los recursos y asignar roles de Azure adicionales al creador. En todos los casos, el propietario de una suscripción también debe [aceptar los términos del software de Avere vFXT](#accept-software-terms) con antelación.

| Escenario | Restricciones | Roles de acceso necesarios para crear el clúster de vFXT Avere |
|----------|--------|-------|
| El administrador del grupo de recursos crea el vFXT | La red virtual, el controlador de clúster y los nodos de clúster deben crearse en el grupo de recursos. | Roles [Administrador de acceso de usuario](../role-based-access-control/built-in-roles.md#user-access-administrator) y [Colaborador](../role-based-access-control/built-in-roles.md#contributor), ambos con ámbito en el grupo de recursos de destino. |
| Uso de una red virtual externa existente | El controlador de clúster y los nodos de clúster se crean dentro del grupo de recursos de vFXT, pero se usa una red virtual existente de otro grupo de recursos. | (1) Roles [Administrador de acceso de usuario](../role-based-access-control/built-in-roles.md#user-access-administrator) y [Colaborador](../role-based-access-control/built-in-roles.md#contributor) con ámbito en el grupo de recursos de vFXT; y (2) Roles [Colaborador de la máquina virtual](../role-based-access-control/built-in-roles.md#virtual-machine-contributor), [Administrador de acceso de usuario](../role-based-access-control/built-in-roles.md#user-access-administrator) y [Colaborador de Avere](../role-based-access-control/built-in-roles.md#avere-contributor) con ámbito en el grupo de recursos de la red virtual. |
| Rol personalizado para los creadores de clústeres | No hay restricciones en la ubicación de los recursos. Este método proporciona privilegios significativos a quienes no son propietarios. | El propietario de la suscripción crea un rol personalizado de Azure, tal como se explica en [este artículo](avere-vfxt-non-owner.md). |

## <a name="quota-for-the-vfxt-cluster"></a>Cuota del clúster de vFXT

Compruebe que tiene una cuota suficiente para los siguientes componentes de Azure. Si fuera necesario, [solicite un aumento de la cuota](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request).

> [!NOTE]
> Las máquinas virtuales y los componentes SSD que se enumeran aquí son para el propio clúster de vFXT. Recuerde que también necesita una cuota para las máquinas virtuales y las unidades de estado sólido (SSD) que usará para la granja de servidores de proceso.
>
> Asegúrese de que la cuota está habilitada para la región donde se vaya a ejecutar el flujo de trabajo.

|Componente de Azure|Quota|
|----------|-----------|
|Máquinas virtuales|3 o más E32s_v3 (uno por nodo de clúster) |
|Almacenamiento SSD Premium|200 GB de espacio de sistema operativo y de 1 TB a 4 TB de espacio en caché por nodo |
|Cuenta de almacenamiento (opcional) |v2|
|Almacenamiento back-end de datos (opcional) |Un contenedor de blobs LRS nuevo |
<!-- this table also appears in the overview - update it there if updating here -->

## <a name="accept-software-terms"></a>Aceptar los términos de software

> [!TIP]
> Omita este paso, si el propietario de una suscripción va a crear el clúster de Avere vFXT.

Durante la creación del clúster, debe aceptar los términos del servicio para el software de Avere vFXT. Si no es el propietario de la suscripción, tiene que pedir a un propietario de la suscripción que acepte los términos antes de realizar cualquier operación.

Este paso solo se debe hacer una vez por suscripción.

Para aceptar los términos de software de antemano:

1. Abra una instancia de Cloud Shell en Azure Portal o vaya a <https://shell.azure.com>. Inicie sesión con el id. de la suscripción.

   ```azurecli
    az login
    az account set --subscription <subscription ID>
   ```

1. Use este comando para aceptar los términos de servicio y habilitar el acceso mediante programación para Avere vFXT y así obtener la imagen de software de Azure:

   ```azurecli
   az vm image accept-terms --urn microsoft-avere:vfxt:avere-vfxt-controller:latest
   ```

## <a name="create-a-storage-service-endpoint-in-your-virtual-network-if-needed"></a>Creación de un punto de conexión de servicio de almacenamiento en la red virtual (si es necesario)

Un [punto de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) mantiene el tráfico del blob de Azure de manera local, en lugar de enrutarlo fuera de la red virtual. Se recomienda para cualquier clúster de Avere vFXT for Azure que utilice el blob de Azure para el almacenamiento de datos de back-end.

Si crea una nueva red virtual durante la creación del clúster, se crea automáticamente un punto de conexión. Si proporciona una red virtual existente, debe tener un punto de conexión del servicio de almacenamiento de Microsoft si desea crear un nuevo contenedor de almacenamiento de blobs durante la creación del clúster.<!-- if there is no endpoint in that situation, the cluster creation will fail -->

> [!TIP]
>
>* Omita este paso si va a crear una red virtual como parte de la creación del clúster.
>* Un punto de conexión es opcional, si no va a crear el almacenamiento de blobs durante la creación del clúster. En ese caso, puede crear el punto de conexión de servicio más adelante si decide utilizar un blob de Azure.

Cree el punto de conexión de servicio de almacenamiento desde Azure Portal.

1. En el portal, abra la lista de redes virtuales.
1. Seleccione la red virtual para el clúster.
1. Haga clic en **Puntos de conexión de servicio** en el menú de la izquierda.
1. Haga clic en **Agregar** en la parte superior.
1. Elija el servicio ``Microsoft.Storage``.
1. Seleccione la subred del clúster.
1. Haga clic en **Agregar**en la parte inferior.

   ![Captura de pantalla de Azure Portal con anotaciones de los pasos para crear el punto de conexión de servicio](media/avere-vfxt-service-endpoint.png)

## <a name="next-steps"></a>Pasos siguientes

Después de completar estos requisitos previos, puede crear el clúster. Lea [Deploy the vFXT cluster](avere-vfxt-deploy.md) (Implementar el clúster de vFXT) para obtener más instrucciones.
