---
title: Configuración del método de enrutamiento del tráfico de rendimiento con Azure Traffic Manager | Microsoft Docs
description: En este artículo se explica cómo configurar Traffic Manager para enrutar el tráfico al punto de conexión con latencia más baja.
services: traffic-manager
manager: twooley
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2017
ms.author: duau
ms.openlocfilehash: 46d71a38b79f449084f7353527f2dfb05d5b92a4
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401034"
---
# <a name="configure-the-performance-traffic-routing-method"></a>Configuración del método de enrutamiento del tráfico de rendimiento

El método de enrutamiento de tráfico Rendimiento permite dirigir el tráfico al punto de conexión con la menor latencia de red del cliente. Normalmente, el centro de datos con la latencia más baja corresponde a la distancia geográfica más cercana. Este método de enrutamiento de tráfico no cuenta los cambios en tiempo real en la carga o configuración de red.

##  <a name="to-configure-performance-routing-method"></a>Pasos para configurar el método de enrutamiento de rendimiento

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com). Si aún no tiene cuenta, puede registrarse para obtener [una evaluación gratuita durante un mes](https://azure.microsoft.com/free/). 
2. En la barra de búsqueda del Portal, busque los **perfiles de Traffic Manager** y, luego, haga clic en el nombre del perfil para el que desea configurar el método de enrutamiento.
3. En la hoja **Perfil de Traffic Manager**, compruebe que existen los servicios en la nube y los sitios web que desea incluir en la configuración.
4. En la sección **Configuración**, haga clic en **Configuración** y, en la hoja **Configuración**, proceda de la siguiente forma:
    1. En la **configuración del método de enrutamiento de tráfico**, seleccione **Rendimiento** en **Método de enrutamiento**.
    2. Establezca la misma **configuración de supervisión de puntos de conexión** para todos los puntos de conexión dentro de este perfil de esta forma:
        1. Seleccione el **protocolo** adecuado y especifique el número de **puerto**. 
        2. En el tipo de **ruta de acceso**, escriba una barra diagonal */* . Para supervisar los puntos de conexión, debe especificar una ruta de acceso y un nombre de archivo. Una barra diagonal "/" es una entrada válida para la ruta de acceso relativa e implica que el archivo se encuentra en el directorio raíz (valor predeterminado).
        3. Haga clic en **Guardar** en la parte superior de la página.
5.  Pruebe los cambios de la configuración de la siguiente forma:
    1.  En la barra de búsqueda del Portal, busque el nombre del perfil de Traffic Manager y haga clic en él en los resultados que se muestran.
    2.  En la hoja del perfil de **Traffic Manager**, haga clic en **Información general**.
    3.  La hoja **Perfil de Traffic Manager** muestra el nombre DNS del perfil de Traffic Manager que acaba de crear. Cualquier cliente puede usar este perfil (por ejemplo, mediante un explorador web) para enrutar el tráfico al punto de conexión correcto según el tipo de enrutamiento. En este caso, todas las solicitudes se enrutan al punto de conexión con la menor latencia de red del cliente.
6. Una vez que el perfil de Traffic Manager esté en funcionamiento, edite el registro DNS en el servidor DNS relevante para redireccionar el nombre de dominio de la empresa al nombre de dominio del Administrador de tráfico.

![Configuración del método de enrutamiento del tráfico de rendimiento con Traffic Manager][1]

## <a name="next-steps"></a>Pasos siguientes

- Información sobre el [método de enrutamiento de tráfico ponderado](traffic-manager-configure-weighted-routing-method.md).
- Sepa cómo funciona el [método de enrutamiento por prioridad](traffic-manager-configure-priority-routing-method.md).
- Información sobre el [método de enrutamiento geográfico](traffic-manager-configure-geographic-routing-method.md).
- Información sobre cómo [probar la configuración de Traffic Manager](traffic-manager-testing-settings.md).

<!--Image references-->
[1]: ./media/traffic-manager-performance-routing-method/traffic-manager-performance-routing-method.png