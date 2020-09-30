---
title: 'Tutorial: Implementación de Azure Spring Cloud en una red virtual'
description: Implemente Azure Spring Cloud en una red virtual (inserción de red virtual).
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/21/2020
ms.custom: devx-track-java
ms.openlocfilehash: 979ecf77fe53238dfd377c5fd2baf394de985c2f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90892899"
---
# <a name="tutorial-deploy-azure-spring-cloud-in-azure-virtual-network-vnet-injection"></a>Tutorial: Implementación de Azure Spring Cloud en una red virtual de Azure (inserción de red virtual)

**Este artículo se aplica a:** ✔️ Java ✔️ C#

En este tutorial se explica cómo implementar una instancia de servicio de Azure Spring Cloud en la red virtual. Esto se llama en ocasiones inserción de red virtual.  

La implementación habilita:

* El aislamiento de Internet en la red corporativa de las aplicaciones y el entorno de ejecución del servicio Azure Spring Cloud
* La interacción de Azure Spring Cloud con los sistemas de los centros de datos locales y los servicios de Azure de otras redes virtuales
* Permite a los clientes controlar las comunicaciones de red entrantes y salientes para Azure Spring Cloud

## <a name="prerequisites"></a>Requisitos previos
Debe registrar el proveedor de recursos de Azure Spring Cloud `Microsoft.AppPlatform` de acuerdo con las instrucciones descritas en [Registro del proveedor de recursos en Azure Portal](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types#azure-portal) o mediante la ejecución del siguiente comando az de la CLI:

```azurecli
az provider register --namespace Microsoft.AppPlatform
```
## <a name="virtual-network-requirements"></a>Requisitos de red virtual
La red virtual en la que se implementa la instancia de servicio de Azure Spring Cloud debe cumplir los siguientes requisitos:

* **Ubicación**: la red virtual debe residir en la misma ubicación que la instancia de servicio de Azure Spring Cloud.
* **Suscripción**: la red virtual debe estar en la misma suscripción que la instancia de servicio de Azure Spring Cloud.
* **Subredes**: la red virtual debe incluir dos subredes dedicadas para una instancia de servicio de Azure Spring Cloud: 
    * Una para el entorno de ejecución del servicio
    * Una para las aplicaciones de microservicios de Spring Boot. 
    * Hay una relación uno a uno entre estas subredes y una instancia de servicio de Azure Spring Cloud. Debe usar una nueva subred para cada instancia de servicio que implemente, y cada subred solo puede incluir una única instancia de servicio.
* **Espacio de direcciones**: un bloque CIDR de hasta /28 para la subred del entorno de ejecución del servicio y otro bloque CIDR de hasta /24 para la subred de aplicaciones de microservicios de Spring Boot.
* **Tabla de rutas**: las subredes no deben tener asociada una tabla de rutas existente.

Los procedimientos siguientes describen la configuración de la red virtual para que contenga la instancia de Azure Spring Cloud.

## <a name="create-a-virtual-network"></a>Creación de una red virtual
Si ya tiene una red virtual para hospedar la instancia de servicio de Azure Spring Cloud, omita los pasos 1, 2 y 3. Puede comenzar en el paso 4 para preparar las subredes de la red virtual.

1. En el menú de Azure Portal, seleccione **Crear un recurso**. En Azure Marketplace, seleccione **Redes** > **Red virtual**.

1. En el cuadro de diálogo **Crear red virtual**, escriba o seleccione la siguiente información:

    |Configuración          |Valor                                             |
    |-----------------|--------------------------------------------------|
    |Suscripción     |Seleccione su suscripción.                         |
    |Resource group   |Seleccione el grupo de recursos o cree uno nuevo.  |
    |Nombre             |Escriba *azure-spring-cloud-vnet*.                   |
    |Location         |Seleccione **Este de EE. UU**.                                |

1. Haga clic en **Siguiente: Direcciones IP >** . 
 
1. En Espacio de direcciones IPv4, escriba 10.1.0.0/16.

1. Seleccione **Agregar subred** y, a continuación, escriba *service-runtime-subnet* para **Nombre de subred** y 10.1.0.0/24 para **Intervalo de direcciones de subred**. A continuación, haga clic en **Agregar**.

1. Seleccione de nuevo **Agregar subred** y escriba *apps-subnet* para **Nombre de subred** y 10.1.1.0/24 para **Intervalo de direcciones de subred**.  Haga clic en **Agregar**.

1. Haga clic en **Revisar + crear**. Deje el resto de campos con los valores predeterminados y haga clic en **Aceptar**.

## <a name="grant-service-permission-to-the-virtual-network"></a>Concesión de permisos de servicio a la red virtual

Seleccione la red virtual *azure-spring-cloud-vnet* que creó anteriormente.

1. Seleccione **Control de acceso (IAM)** y, a continuación, seleccione **Agregar > Agregar asignación de roles**.

    ![Control de acceso para la red virtual](./media/spring-cloud-v-net-injection/access-control.png)

2. En el cuadro de diálogo **Agregar asignación de roles**, escriba o seleccione esta información:

    |Configuración  |Valor                                             |
    |---------|--------------------------------------------------|
    |Role     |Seleccione **Propietario**.                                  |
    |Seleccionar   |Escriba *Azure Spring Cloud Resource Provider*.      |

    A continuación, seleccione *Azure Spring Cloud Resource Provider* y haga clic en **Guardar**.

    ![Concesión de permisos del proveedor de recursos de Azure Spring Cloud en la red virtual](./media/spring-cloud-v-net-injection/grant-azure-spring-cloud-resource-provider-to-vnet.png)

También puede conseguirlo mediante la ejecución del siguiente comando az de la CLI.

```azurecli
VIRTUAL_NETWORK_RESOURCE_ID=`az network vnet show \
    --name ${NAME_OF_VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP_OF_VIRTUAL_NETWORK} \
    --query "id" \
    --output tsv`

az role assignment create \
    --role "Owner" \
    --scope ${VIRTUAL_NETWORK_RESOURCE_ID} \
    --assignee e8de9221-a19c-4c81-b814-fd37c6caf9d2
```

## <a name="deploy-azure-spring-cloud-service-instance-in-the-virtual-network"></a>Implementación de la instancia de servicio de Azure Spring Cloud en la red virtual

1. Abra Azure Portal en https://ms.portal.azure.com.

1. En el cuadro de búsqueda de la parte superior, busque **Azure Spring Cloud** y seleccione **Azure Spring Cloud** en el resultado.

1. En la página **Azure Spring Cloud**, seleccione **+ Add** (+ Agregar).

1. Rellene el formulario en la página **Crear** de Azure Spring Cloud. 

1. Seleccione el mismo grupo de recursos y la misma región que la red virtual.

1. En el campo **Name** (Nombre) situado en **Service Details** (Detalles de servicio), seleccione *azure-spring-cloud-vnet*.

1. Seleccione la pestaña **Networking** (Redes) y seleccione lo siguiente:

    |Configuración                                |Valor                                             |
    |---------------------------------------|--------------------------------------------------|
    |Deploy in your own virtual network (Implementar en su propia red virtual)     |Seleccione **Sí**.                                    |
    |Virtual network                        |Seleccione *azure-spring-cloud-vnet*.                  |
    |Service runtime subnet (Subred del entorno de ejecución del servicio)                 |Seleccione *service-runtime-subnet*.                   |
    |Spring Boot microservice apps subnet (Subred de aplicaciones de microservicios de Spring Boot)   |Seleccione *apps-subnet*.                              |

    ![Pestaña de redes de la página de creación](./media/spring-cloud-v-net-injection/creation-blade-networking-tab.png)

1. Haga clic en **Revisar y crear**.

1. Verifique sus especificaciones y haga clic en **Crear**.

Después de la implementación, se crearán dos grupos de recursos adicionales en la suscripción para hospedar los recursos de red de la instancia de servicio de Azure Spring Cloud.  Vaya a **Inicio** y seleccione **Grupos de recursos** en los elementos del menú superior para buscar los siguientes nuevos grupos de recursos.

El grupo de recursos llamado *azure-spring-cloud-service-runtime_{nombre de la instancia de servicio}_{región de la instancia de servicio}* contiene los recursos de red del entorno de ejecución del servicio de la instancia de servicio.

  ![Entorno de ejecución del servicio](./media/spring-cloud-v-net-injection/service-runtime-resource-group.png)

El grupo de recursos llamado *azure-spring-cloud-service-runtime_{nombre de la instancia de servicio}_{región de la instancia de servicio}* contiene los recursos de red de las aplicaciones de microservicios de Spring Boot de la instancia de servicio.

  ![Grupo de recursos de aplicaciones](./media/spring-cloud-v-net-injection/apps-resource-group.png)

Esos recursos de red están conectados a la red virtual creada anteriormente.

  ![Red virtual con dispositivo conectado](./media/spring-cloud-v-net-injection/vnet-with-connected-device.png)

   > [!Important]
   > El servicio Azure Spring Cloud administra completamente los grupos de recursos. NO elimine ni modifique manualmente ningún recurso dentro de estos grupos de recursos.

## <a name="next-steps"></a>Pasos siguientes

[Implementación de la aplicación en Azure Spring Cloud en la red virtual](https://github.com/microsoft/vnet-in-azure-spring-cloud/blob/master/02-deploy-application-to-azure-spring-cloud-in-your-vnet.md)

## <a name="see-also"></a>Consulte también

- [Solución de problemas de Azure Spring Cloud en una red virtual](https://github.com/microsoft/vnet-in-azure-spring-cloud/blob/master/05-troubleshooting-azure-spring-cloud-in-vnet.md)
- [Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual](https://github.com/microsoft/vnet-in-azure-spring-cloud/blob/master/06-customer-responsibilities-for-running-azure-spring-cloud-in-vnet.md)
