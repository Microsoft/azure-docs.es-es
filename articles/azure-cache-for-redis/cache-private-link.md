---
title: Azure Cache for Redis con Azure Private Link (versión preliminar)
description: Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a Azure Cache for Redis de Azure Private Link. En este artículo, obtendrá información sobre cómo crear una caché de Azure, una red virtual de Azure y un punto de conexión privado mediante Azure Portal.
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: 5db756b60330cdac4e43e13bfe29d9397f87af50
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421661"
---
# <a name="azure-cache-for-redis-with-azure-private-link-preview"></a>Azure Cache for Redis con Azure Private Link (versión preliminar)
Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a Azure Cache for Redis de Azure Private Link. 

En este artículo, obtendrá información sobre cómo crear una caché de Azure, una red virtual de Azure y un punto de conexión privado mediante Azure Portal.  

> [!IMPORTANT]
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Condiciones de uso complementarias de las versiones preliminares de Microsoft Azure.](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) 
> 

## <a name="prerequisites"></a>Requisitos previos
* Una suscripción a Azure:  [cree una cuenta gratuita](https://azure.microsoft.com/free/).

> [!NOTE]
> Esta característica se encuentra actualmente en versión preliminar: [póngase en contacto con nosotros](mailto:azurecache@microsoft.com) si le interesa.
>

## <a name="create-a-cache"></a>Creación de una caché
1. Para crear una instancia de caché, inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Crear un recurso**. 

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Selección de Crear un recurso.":::
   
1. En la página **Nuevo**, seleccione **Base de datos** y, a continuación, seleccione **Azure Cache for Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Selección de Azure Cache for Redis.":::
   
1. En la página **Nueva instancia de Redis Cache**, configure las opciones de la nueva caché.
   
   | Configuración      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nombre DNS** | Escriba un nombre único global. | El nombre de la memoria caché debe ser una cadena de entre 1 y 63 caracteres, y solo puede contener números, letras o guiones. El nombre debe comenzar y terminar por un número o una letra y no puede contener guiones consecutivos. El *nombre de host* de la instancia de caché será *\<DNS name>.redis.cache.windows.net*. | 
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | La suscripción en la que se creará esta nueva instancia de Azure Cache for Redis. | 
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. | 
   | **Ubicación** | Desplácese hacia abajo y seleccione una ubicación. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la memoria caché. |
   | **Plan de tarifa** | Desplácese hacia abajo y seleccione un [plan de tarifa](https://azure.microsoft.com/pricing/details/cache/). |  El plan de tarifa determina el tamaño, el rendimiento y las características disponibles para la memoria caché. Para más información, consulte la [introducción a Azure Redis Cache](cache-overview.md). |
   
1. Seleccione **Crear**. 
   
    :::image type="content" source="media/cache-private-link/3-new-cache.png" alt-text="Creación de Azure Cache for Redis.":::
   
   La caché tarda un tiempo en crearse. Puede supervisar el progreso en la página **Información general** de Azure Cache for Redis. Cuando **Estado** se muestra como **En ejecución**, la memoria caché está lista para su uso.
    
    :::image type="content" source="media/cache-private-link/4-status.png" alt-text="Instancia creada de Azure Cache for Redis.":::

## <a name="create-a-virtual-network"></a>Creación de una red virtual

En esta sección, creará una red virtual y una subred.

1. Seleccione **Crear un recurso**.

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Selección de Crear un recurso.":::

2. En la página **Nuevo**, seleccione **Redes** y seleccione **Red virtual**.

    :::image type="content" source="media/cache-private-link/5-select-vnet.png" alt-text="Creación de una red virtual.":::

3. En **Crear red virtual**, escriba o seleccione esta información en la pestaña **Conceptos básicos**:

    | **Configuración**          | **Valor**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Detalles del proyecto**  |                                                                 |
    | Suscripción     | Desplácese hacia abajo y seleccione su suscripción.                                  |
    | Grupo de recursos   | Desplácese hacia abajo y seleccione un grupo de recursos. |
    | **Detalles de instancia** |                                                                 |
    | Nombre             | Escriba **\<virtual-network-name>**                                    |
    | Region           | Selección de **\<region-name>** |

4. Seleccione la pestaña **Direcciones IP** o el botón **Siguiente: Direcciones IP** situado en la parte inferior de la página.

5. En la pestaña **Direcciones IP**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Espacio de direcciones IPv4 | Escriba **\<IPv4-address-space>** |

6. En **Nombre de subred**, seleccione la palabra **predeterminada**.

7. En **Editar subred**, especifique esta información:

    | Configuración            | Value                      |
    |--------------------|----------------------------|
    | Nombre de subred | Escriba **\<subnet-name>** |
    | Intervalo de direcciones de subred | Escriba **\<subnet-address-range>**

8. Seleccione **Guardar**.

9. Seleccione la pestaña **Revisar y crear** o el botón **Revisar y crear**.

10. Seleccione **Crear**.


## <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado 

En esta sección, creará un punto de conexión privado y lo conectará a la caché que creó anteriormente.

1. Busque **Private Link** y presione Entrar o selecciónelo en las sugerencias de búsqueda.

    :::image type="content" source="media/cache-private-link/7-create-private-link.png" alt-text="Búsqueda de Private Link.":::

2. En el lado izquierdo de la pantalla, seleccione **Puntos de conexión privados**.

    :::image type="content" source="media/cache-private-link/8-select-private-endpoint.png" alt-text="Selección de Private Link.":::

3. Seleccione el botón **+ Agregar** para crear el punto de conexión privado. 

    :::image type="content" source="media/cache-private-link/9-add-private-endpoint.png" alt-text="Adición de una instancia de Private Link.":::

4. En la **página Crear un punto de conexión privado**, configure las opciones del punto de conexión privado.

    | Configuración | Value |
    | ------- | ----- |
    | **DETALLES DEL PROYECTO** | |
    | Suscripción | Desplácese hacia abajo y seleccione su suscripción. |
    | Resource group | Desplácese hacia abajo y seleccione un grupo de recursos. |
    | **DETALLES DE INSTANCIA** |  |
    | Nombre |Escriba un nombre para el punto de conexión privado.  |
    | Region |Desplácese hacia abajo y seleccione una ubicación. |
    |||

5. Seleccione el botón **Siguiente: Recurso** en la parte inferior de la página.

6. En la pestaña **Recurso**, seleccione su suscripción, elija el tipo de recurso Microsoft.Cache/Redis y, a continuación, seleccione la caché que creó en la sección anterior.

    :::image type="content" source="media/cache-private-link/10-resource-private-endpoint.png" alt-text="Recursos para Private Link.":::

7. Seleccione el botón **Siguiente: Configuración** situado en la parte inferior de la página.

8. En la pestaña **Configuración**, seleccione la red virtual y la subred que creó en la sección anterior.

    :::image type="content" source="media/cache-private-link/11-configuration-private-endpoint.png" alt-text="Configuración de Private Link.":::

9. Seleccione el botón **Siguiente: Etiquetas** situado en la parte inferior de la página.

10. En la pestaña **Etiquetas**, escriba el nombre y el valor si desea clasificar el recurso. Este paso es opcional.

    :::image type="content" source="media/cache-private-link/12-tags-private-endpoint.png" alt-text="Etiquetas para Private Link.":::

11. Seleccione  **Revisar y crear**. Se le remite a la pestaña  **Revisar y crear** , donde Azure valida la configuración.

12. Cuando aparezca el mensaje verde **Validación superada**, seleccione **Crear**.


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Private Link, consulte la [documentación de Azure Private Link](https://docs.microsoft.com/azure/private-link/private-link-overview). 

