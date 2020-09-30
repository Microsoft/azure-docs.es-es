---
title: 'Tutorial: Configuración del enrutamiento geográfico del tráfico con Azure Traffic Manager'
description: En este tutorial se explica cómo configurar el método de enrutamiento geográfico del tráfico mediante Azure Traffic Manager
services: traffic-manager
author: duongau
manager: kumudD
ms.service: traffic-manager
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2017
ms.author: duau
ms.openlocfilehash: 53773d7c616edec067e1ed1778b7ce6b500ee936
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462622"
---
# <a name="tutorial-configure-the-geographic-traffic-routing-method-using-traffic-manager"></a>Tutorial: Configuración del método de enrutamiento de tráfico geográfico con Traffic Manager

El método de enrutamiento de tráfico geográfico le permite dirigir el tráfico a puntos de conexión específicos según la ubicación geográfica en que se originan las solicitudes. En este tutorial se muestra cómo crear un perfil de Traffic Manager con este método de enrutamiento y configurar los puntos de conexión para recibir el tráfico proveniente de geografías específicas.

## <a name="create-a-traffic-manager-profile"></a>Creación de un perfil del Administrador de tráfico

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com). Si aún no tiene cuenta, puede registrarse para obtener [una evaluación gratuita durante un mes](https://azure.microsoft.com/free/).
2. Haga clic en **Crear un recurso** > **Redes** > **Perfil del Traffic Manager** > **Crear**.
4. En **Crear perfil de Traffic Manager**:
    1. Proporcione un nombre para el perfil. Este nombre debe ser único dentro de la zona trafficmanager.net. Para acceder a su perfil de Traffic Manager, use el nombre DNS `<profilename>.trafficmanager.net`.
    2. Seleccione el método de enrutamiento **geográfico**.
    3. Seleccione la suscripción en la que desea crear este perfil.
    4. Use un grupo de recursos existente o cree un grupo de recursos nuevo en el cual colocar este perfil. Si decide crear un nuevo grupo de recursos, use la lista desplegable **Ubicación del grupo de recursos** para especificar la ubicación del grupo de recursos. Esta configuración hace referencia a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementará globalmente.
    5. Una vez que haga clic en **Crear**, se creará el perfil de Traffic Manager y se implementará globalmente.

![Crear un perfil de Traffic Manager](./media/traffic-manager-geographic-routing-method/create-traffic-manager-profile.png)

## <a name="add-endpoints"></a>Incorporación de puntos de conexión

1. Busque el nombre del perfil de Traffic Manager que acaba de crear en la barra de búsqueda del portal y haga clic en el resultado cuando aparezca.
2. Vaya a **Configuración** -> **Puntos de conexión** en Traffic Manager.
3. Haga clic en **Agregar** y, en el panel **Agregar punto de conexión** que aparece, siga estos pasos:
4. Seleccione **Tipo** según el tipo de punto de conexión que agrega. En el caso de los perfiles de enrutamiento geográfico que se usan en producción, se recomienda usar tipos de punto de conexión anidado que contengan un perfil secundario con más de un punto de conexión. Para más detalles, consulte [Preguntas más frecuentes sobre los métodos de enrutamiento de tráfico geográfico](traffic-manager-FAQs.md).
5. Proporcione un **nombre** con el que desea reconocer este punto de conexión.
6. Algunos de los campos de esta página dependen del tipo de punto de conexión que agrega:
    1. Si agrega un punto de conexión de Azure, seleccione el **tipo de recurso de destino** y el **destino** según el recurso al que desea dirigir el tráfico.
    2. Si agrega un punto de conexión **externo**, proporcione el **nombre de dominio completo (FQDN)** del punto de conexión.
    3. Si agrega un **punto de conexión anidado**, seleccione el **recurso de destino** que corresponde al perfil secundario que desea usar y especifique el **recuento mínimo de puntos de conexión secundarios**.
7. En la sección Asignación geográfica, use la lista desplegable para agregar las regiones desde las que desea enviar el tráfico a este punto de conexión. Debe agregar al menos una región y puede tener asignadas varias regiones.
8. Repita este proceso para todos los puntos de conexión que desea agregar en este perfil

![Incorporación de un punto de conexión de Traffic Manager](./media/traffic-manager-geographic-routing-method/add-traffic-manager-endpoint.png)

## <a name="use-the-traffic-manager-profile"></a>Uso del perfil de Traffic Manager
1.  En la barra de búsqueda del portal, busque el nombre del **perfil de Traffic Manager** que creó en la sección anterior y haga clic en el perfil de Traffic Manager en los resultados que aparecen.
2. Haga clic en **Descripción general**.
3. La hoja **Perfil de Traffic Manager** muestra el nombre DNS del perfil de Traffic Manager que acaba de crear. Cualquier cliente puede usar este perfil (por ejemplo, mediante un explorador web) para enrutar el tráfico al punto de conexión correcto según el tipo de enrutamiento.  En el caso del enrutamiento geográfico, Traffic Manager busca la dirección IP de origen de la solicitud entrante y determina la región desde la que se origina. Si esa región está asignada a un punto de conexión, el tráfico se enruta ahí. Si esta región no está asignada a un punto de conexión, Traffic Manager devuelve una respuesta de consulta NODATA.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [método de enrutamiento del tráfico geográfico](traffic-manager-routing-methods.md#geographic).
- Información sobre cómo [probar la configuración de Traffic Manager](traffic-manager-testing-settings.md).
