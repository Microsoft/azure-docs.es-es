---
title: 'Tutorial: Configuración del enrutamiento del tráfico prioritario con Azure Traffic Manager'
description: En este tutorial se explica cómo configurar el método de enrutamiento de tráfico prioritario en Traffic Manager
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2017
ms.author: duau
ms.openlocfilehash: 404338c3e36216833d39c3551ae2dee0be304d24
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401017"
---
# <a name="tutorial-configure-priority-traffic-routing-method-in-traffic-manager"></a>Tutorial: Configuración del método de enrutamiento de tráfico por prioridad en Traffic Manager

Con independencia del modo del sitio web, Azure Websites ya proporciona la funcionalidad de conmutación por error para sitios web en un centro de datos (también conocido como región). Traffic Manager proporciona conmutación por error para sitios web en distintos centros de datos.

Un patrón común para la conmutación por error del servicio es proporcionar un conjunto de servicios de copia de seguridad idénticos y enviar tráfico a un servicio principal. Los siguientes pasos explican cómo configurar esta conmutación por error con prioridad con Azure Cloud Services y Azure Websites:

## <a name="to-configure-the-priority-traffic-routing-method"></a>Para configurar el método de enrutamiento de tráfico por prioridad

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com). Si aún no tiene cuenta, puede registrarse para obtener [una evaluación gratuita durante un mes](https://azure.microsoft.com/free/). 
2. En la barra de búsqueda del Portal, busque los **perfiles de Traffic Manager** y, luego, haga clic en el nombre del perfil para el que desea configurar el método de enrutamiento.
3. En la hoja **Perfil de Traffic Manager**, compruebe que existen los servicios en la nube y los sitios web que desea incluir en la configuración.
4. En la sección **Configuración**, haga clic en **Configuración** y, en la hoja **Configuración**, proceda de la siguiente forma:
    1. En **Configuración del método de enrutamiento de tráfico**, compruebe que dicho método sea **Prioridad**. De no ser así, haga clic en **Prioridad** en la lista desplegable.
    2. Establezca la misma **configuración de supervisión de puntos de conexión** para todos los puntos de conexión dentro de este perfil de esta forma:
        1. Seleccione el **protocolo** adecuado y especifique el número de **puerto**. 
        2. En el tipo de **ruta de acceso**, escriba una barra diagonal */* . Para supervisar los puntos de conexión, debe especificar una ruta de acceso y un nombre de archivo. Una barra diagonal "/" es una entrada válida para la ruta de acceso relativa e implica que el archivo se encuentra en el directorio raíz (valor predeterminado).
        3. Haga clic en **Guardar** en la parte superior de la página.
5. En la sección **Configuración**, haga clic en **Puntos de conexión**.
6. En la hoja **Puntos de conexión**, revise el orden de prioridad de los puntos de conexión. Cuando selecciona el método de enrutamiento de tráfico de **Prioridad**, el orden de los extremos seleccionados es importante. Compruebe el orden de prioridad de los puntos de conexión.  El extremo principal se encuentra en primer lugar. Vuelva a comprobar el orden en que se muestra. En este caso, todas las solicitudes se enrutarán al primer punto de conexión y si Traffic Manager detecta que no funciona correctamente, se hace conmutación por error del tráfico automáticamente al siguiente punto de conexión. 
7. Para cambiar el orden de prioridad de los puntos de conexión, haga clic en el punto de conexión y, en la hoja **Punto de conexión** que aparece, haga clic en **Editar** y cambie el valor de **prioridad** según sea necesario. 
8. Haga clic en **Guardar** para guardas los cambios en la configuración de puntos de conexión.
9. Una vez que haya terminado de cambiar la configuración, haga clic en **Guardar** en la parte inferior de la página.
10. Pruebe los cambios de la configuración de la siguiente forma:
    1.  En la barra de búsqueda del Portal, busque el nombre del perfil de Traffic Manager y haga clic en él en los resultados que se muestran.
    2.  En la hoja del perfil de **Traffic Manager**, haga clic en **Información general**.
    3.  La hoja **Perfil de Traffic Manager** muestra el nombre DNS del perfil de Traffic Manager que acaba de crear. Cualquier cliente puede usar este perfil (por ejemplo, mediante un explorador web) para enrutar el tráfico al punto de conexión correcto según el tipo de enrutamiento. En este caso, todas las solicitudes se enrutan al primer punto de conexión y si Traffic Manager detecta que no funciona correctamente, se hace conmutación por error del tráfico automáticamente al siguiente punto de conexión.
11. Una vez que el perfil de Traffic Manager esté en funcionamiento, edite el registro DNS en el servidor DNS relevante para redireccionar el nombre de dominio de la empresa al nombre de dominio del Administrador de tráfico.

![Configuración del método de enrutamiento de tráfico por prioridad con Traffic Manager][1]

## <a name="next-steps"></a>Pasos siguientes


- Información sobre el [método de enrutamiento de tráfico ponderado](traffic-manager-configure-weighted-routing-method.md).
- Información sobre el [método de enrutamiento de rendimiento](traffic-manager-configure-performance-routing-method.md).
- Información sobre el [método de enrutamiento geográfico](traffic-manager-configure-geographic-routing-method.md).
- Información sobre cómo [probar la configuración de Traffic Manager](traffic-manager-testing-settings.md).

<!--Image references-->
[1]: ./media/traffic-manager-priority-routing-method/traffic-manager-priority-routing-method.png