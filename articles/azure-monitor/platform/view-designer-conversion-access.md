---
title: Acceso y resumen de conversión del diseñador de vistas de Azure Monitor en libros
description: ''
author: austonli
ms.author: aul
ms.subservice: ''
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 7614dd1d60bad5c124269ae2af02d30a5aacfe3e
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90564039"
---
# <a name="view-designer-to-workbooks-conversion-summary-and-access"></a>Acceso y resumen de conversión del diseñador de vistas en libros
El [diseñador de vistas](view-designer.md) es una característica de Azure Monitor que permite crear vistas personalizadas para ayudar a visualizar datos en el área de trabajo de Log Analytics, con gráficos, listas y escalas de tiempo. Estas vistas se están retirando paulatinamente y están siendo reemplazadas por libros, que proporcionan funcionalidad adicional. En este artículo se describe cómo puede crear un resumen de información general y los permisos necesarios para acceder a los libros.

## <a name="creating-your-workspace-summary-from-azure-dashboard"></a>Creación del resumen del área de trabajo desde el panel de Azure
Los usuarios del diseñador de vistas pueden estar familiarizados con un icono de información general para representar un conjunto de vistas. Para mantener una información general visual similar al resumen del área de trabajo del diseñador de vistas, los libros ofrecen pasos anclados, que se pueden anclar al [panel de Azure Portal](../../azure-portal/azure-portal-dashboards.md). Al igual que los iconos de información general del resumen del área de trabajo, los elementos del libro anclados se vincularán directamente a la vista del libro.

Puede aprovechar el alto nivel de las características de personalización que se proporcionan con los paneles de Azure, que permiten la actualización automática, el movimiento, el ajuste de tamaño y el filtrado adicional para los elementos anclados y las visualizaciones. 

![Captura de pantalla que muestra un panel de Azure personalizado titulado Resumen del área de trabajo.](media/view-designer-conversion-access/dashboard.png)

Cree un nuevo panel de Azure o seleccione un panel existente para empezar a anclar los elementos de los libros.

Para anclar un elemento individual, deberá habilitar el icono de anclaje para el paso específico. Para ello, seleccione el botón **Editar** correspondiente al paso y, a continuación, seleccione el icono de engranaje para abrir **Configuración avanzada**. Active la opción **Mostrar siempre el icono de anclaje en este paso** y aparecerá un icono de anclaje en la esquina superior derecha del paso. Este icono le permite anclar visualizaciones específicas al panel, como los iconos de información general.

![Paso de anclaje](media/view-designer-conversion-access/pin-step.png)


Es posible que también desee anclar varias visualizaciones del libro o el contenido completo del libro a un panel. Para anclar todo el libro, seleccione **Editar** en la barra de herramientas superior para alternar la opción **Modo de edición**. Aparecerá un icono de anclaje que le permite anclar el elemento libro completo o todos los pasos y visualizaciones individuales del libro.

![Anclar todos](media/view-designer-conversion-access/pin-all.png)



## <a name="sharing-and-viewing-permissions"></a>Compartición y visualización de permisos 
Los libros tienen la ventaja de que pueden ser un documento privado o compartido. De forma predeterminada, los libros guardados se guardarán en **Mis informes**, lo que significa que solo el creador puede ver este libro.

Puede compartir los libros seleccionando el icono **Compartir** de la barra de herramientas superior, mientras se encuentra en **Modo de edición**. Se le pedirá que mueva el libro a **Informes compartidos**, lo que generará un vínculo que proporciona acceso directo al libro.

Para que un usuario pueda ver un libro compartido, debe tener acceso a la suscripción y al grupo de recursos en el que se guarda el libro.

![Acceso basado en suscripciones](media/view-designer-conversion-access/subscription-access.png)

## <a name="next-steps"></a>Pasos siguientes

- [Tareas comunes](view-designer-conversion-tasks.md)