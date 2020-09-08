---
title: Procedimientos de autodiagnóstico y solución de problemas en Azure Spring Cloud
description: Más información sobre el autodiagnóstico y la solución de problemas en Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 05/29/2020
ms.custom: devx-track-java
ms.openlocfilehash: d8f3b4d57f00ef33bbf566c36babafd83be9a5df
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299633"
---
# <a name="how-to-self-diagnose-and-solve-problems-in-azure-spring-cloud"></a>Procedimientos de autodiagnóstico y solución de problemas en Azure Spring Cloud
Los diagnósticos de Azure Spring Cloud constituyen una experiencia interactiva que le ayuda a solucionar problemas de la aplicación. No se requiere ninguna configuración. Cuando encuentre problemas, el diagnóstico de Azure Spring Cloud indica lo que es incorrecto y le guía por la información que ayuda a solucionar el problema.

## <a name="prerequisites"></a>Requisitos previos
Para completar este tutorial, necesita:

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Una instancia del servicio Azure Spring Cloud implementada. Para comenzar, siga nuestro [inicio rápido sobre la implementación de una aplicación mediante la CLI de Azure](spring-cloud-quickstart.md).
* Al menos una aplicación ya creada en su instancia de servicio.

## <a name="navigate-to-the-diagnostics-page"></a>Vaya a la página de diagnósticos.
1. Inicie sesión en Azure Portal.
2. Vaya a la página **Información general** de Azure Spring Cloud.
3. Abra **Diagnosticar y solucionar problemas** en el menú de la izquierda de la página.

 ![Diagnosticar y solucionar (cuadro de diálogo)](media/spring-cloud-diagnose/diagnose-solve-dialog.png)

 ## <a name="search-logged-issues"></a>Búsqueda de problemas registrados
Para encontrar un problema, puede buscar escribiendo una palabra clave o haciendo clic en grupo de soluciones para explorar todo en esa categoría.

 ![Búsqueda de problemas](media/spring-cloud-diagnose/search-detectors.png)

La selección de **comprobación de estado de Config Server**, el **estado de mantenimiento de Config Server** o el **historial de actualizaciones de Config Server** mostrará varios resultados.

![Opciones de problemas](media/spring-cloud-diagnose/detectors-options.png)

Busque el detector de destino y haga clic en él para ejecutarlo. Después de ejecutar el detector, se mostrará un resumen de los diagnósticos. Puede seleccionar **Ver informe completo** para comprobar los detalles de diagnóstico o haga clic en el botón **Mostrar icono** para volver a la lista de detectores.

 ![Resumen de diagnósticos](media/spring-cloud-diagnose/summary-diagnostics.png)

En la página Detalles de diagnóstico, puede cambiar el intervalo de tiempo de diagnóstico con el controlador en la esquina superior derecha. Para ver más métricas o registros, alterne cada diagnóstico. Puede haber un retraso de 15 minutos para las métricas y los registros.

 ![Detalles de diagnóstico](media/spring-cloud-diagnose/diagnostics-details.png)

Algunos resultados contienen documentación relacionada.

 ![Detalles relacionados](media/spring-cloud-diagnose/related-details.png)

## <a name="next-steps"></a>Pasos siguientes
* [Supervisión de recursos de Spring Cloud mediante alertas y grupos de acciones](spring-cloud-tutorial-alerts-action-groups.md).
* [Controles de seguridad para el servicio Azure Spring Cloud](spring-cloud-concept-security-controls.md)