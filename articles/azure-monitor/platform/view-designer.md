---
title: Creación de vistas para analizar los datos de registro en Azure Monitor | Microsoft Docs
description: Con el uso del Diseñador de vistas en Azure Monitor se pueden crear vistas personalizadas que se muestran en Azure Portal y que contienen diferentes visualizaciones de datos en el área de trabajo de Log Analytics. En este artículo encontrará información general del Diseñador de vistas, y procedimientos para crear y editar vistas personalizadas.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/04/2020
ms.openlocfilehash: f3ae081c8657a3ee6a27b0f9208c3eb4e4e745cc
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87759740"
---
# <a name="create-custom-views-by-using-view-designer-in-azure-monitor"></a>Creación de vistas personalizadas mediante el Diseñador de vistas en Azure Monitor
Con el uso del Diseñador de vistas en Azure Monitor, puede crear una variedad de vistas personalizadas en Azure Portal que pueden ayudarle a visualizar datos en el área de trabajo de Log Analytics. Este artículo presenta una información general sobre Diseñador de vistas y los procedimientos para crear y editar vistas personalizadas.

> [!IMPORTANT]
> Se ha realizado la transición de las vistas de Azure Monitor a [libros](workbooks-overview.md), que proporcionan funcionalidad adicional. Consulte [guía de transición del diseñador de vistas de Azure Monitor a libros](view-designer-conversion-overview.md) para obtener más información sobre cómo convertir las vistas existentes en libros.
 


Para más información acerca del Diseñador de vistas, consulte:

* [Referencia de los iconos](view-designer-tiles.md): proporciona una guía de referencia de la configuración de cada uno de los iconos disponibles en las vistas personalizadas.
* [Referencia de los elementos de visualización](view-designer-parts.md): proporciona una guía de referencia de la configuración de los elementos de visualización que están disponibles en las vistas personalizadas.


## <a name="concepts"></a>Conceptos
Las vistas se muestran en la página **Información general** de Azure Monitor en Azure Portal. Abra esta página desde el menú **Azure Monitor** haciendo clic en **Más** en la sección **Conocimiento**. Los iconos de cada vista personalizada se muestran en orden alfabético, y los iconos para supervisar las soluciones se instalan en la misma área de trabajo.

![Página de información general](media/view-designer/overview-page.png)

Las vistas que se crean con el Diseñador de vistas contienen los elementos que se describen en la tabla siguiente:

| Parte | Descripción |
|:--- |:--- |
| Iconos | Se muestran en la página **Información general** de Azure Monitor. Cada icono muestra un resumen visual de la vista personalizada que representa. Cada tipo de icono proporciona una visualización distinta de los registros. Seleccione un icono para mostrar una vista personalizada. |
| Vista personalizada | Se muestra al seleccionar un icono. Cada vista contiene uno o varios elementos de visualización. |
| Elementos de visualización | Presentan una visualización de los datos del área de trabajo de Log Analytics que se basa en una o varias [consultas de registros](../log-query/log-query-overview.md). Muchos de los elementos incluyen un encabezado que proporciona una visualización de alto nivel y una lista que muestra los principales resultados. Cada tipo de elemento proporciona una visualización diferente de los registros del área de trabajo de Log Analytics. Seleccione los elementos para realizar una consulta de registro que proporcione registros detallados. |

## <a name="required-permissions"></a>Permisos necesarios
Requiere como mínimo [permisos de nivel de colaborador](manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics para crear o modificar las vistas. Si no tiene este permiso, la opción Diseñador de vistas no se mostrará en el menú.


## <a name="work-with-an-existing-view"></a>Uso de una vista existente
Las vistas que se crearon con el Diseñador de vistas muestran las siguientes opciones:

![Menú de información general](media/view-designer/overview-menu.png)

Las opciones se describen en la tabla siguiente:

| Opción | Descripción |
|:--|:--|
| Actualizar   | Actualiza la vista con los datos más recientes. | 
| Registros      | Abre [Log Analytics](../log-query/log-query-overview.md) para analizar los datos con consultas de registro. |
| Editar       | Abre la vista en el Diseñador de vistas para editar su contenido y configuración.  |
| Clonar      | Crea una vista nueva y la abre en el Diseñador de vistas. La vista nueva tiene el mismo nombre que la original con la palabra *Copy* agregada al final. |
| Intervalo de fechas | Establezca un filtro de tiempo y fechas para los datos que se incluye en la vista. Este intervalo de fechas se aplica antes de cualquier conjunto de intervalo de fechas en consultas de la vista.  |
| +          | Defina un filtro personalizado definido para la vista. |


## <a name="create-a-new-view"></a>Creación de vistas
Puede crear una vista nueva en el Diseñador de vistas seleccionando **Diseñador de vistas** en el menú del área de trabajo de Log Analytics.

![Icono del Diseñador de vistas](media/view-designer/view-designer-tile.png)


## <a name="work-with-view-designer"></a>Uso del Diseñador de vistas
Utilice el Diseñador de vistas para crear vistas nuevas o modificar las existentes. 

El Diseñador de vistas tiene tres paneles: 
* **Diseño**: contiene la vista personalizada que crea o edita. 
* **Controles**: contiene los iconos y los elementos que se agregan al panel **Diseño**. 
* **Propiedades**: muestra las propiedades de los iconos o elementos seleccionados.

![Diseñador de vistas](media/view-designer/view-designer-screenshot.png)

### <a name="configure-the-view-tile"></a>Configuración del icono de vista
Una vista personalizada solo puede tener un único icono. Para ver el icono actual o seleccionar otro, seleccione la pestaña **Tile** (icono) del panel de **control**. El panel **Properties** (Propiedades) muestra las propiedades del icono actual. 

Puede configurar las propiedades de los iconos según la información del artículo sobre [Referencia de los iconos](view-designer-tiles.md) y luego hacer clic en **Apply** (Aplicar) para guardar los cambios.

### <a name="configure-the-visualization-parts"></a>Configuración de los elementos de visualización
Una vista puede incluir cualquier número de elementos de visualización. Para agregar elementos a una vista, seleccione la pestaña **Vista** y luego un elemento de visualización. El panel **Propiedades** muestra las propiedades del elemento seleccionado. 

Puede configurar las propiedades de las vistas según la información detallada del artículo sobre [Referencia de los elementos de visualización](view-designer-parts.md) y luego hacer clic en **Aplicar** para guardar los cambios.

Las vistas solo tienen una fila de elementos de visualización. Puede reorganizar los elementos existentes arrastrándolos a una nueva ubicación.

Puede quitar un elemento de visualización de la vista seleccionando **X** en la esquina superior derecha del elemento.


### <a name="menu-options"></a>Opciones del menú
Las opciones para trabajar con vistas en modo de edición se describen en la tabla siguiente.

![Menú Edición](media/view-designer/edit-menu.png)

| Opción | Descripción |
|:--|:--|
| Save        | Guarda los cambios y cierra la vista. |
| Cancelar      | Descarta los cambios y cierra la vista. |
| Eliminar vista | Elimina la vista. |
| Exportación      | Exporta la vista a una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) que puede importar en otra área de trabajo. El nombre del archivo es el nombre de la vista con la extensión *omsview*. |
| Importar      | Importa el archivo *omsview* que haya exportado de otra área de trabajo. Esta acción sobrescribe la configuración de la vista existente. |
| Clonar       | Crea una vista nueva y la abre en el Diseñador de vistas. La vista nueva tiene el mismo nombre que la original con la palabra *Copy* agregada al final. |

## <a name="next-steps"></a>Pasos siguientes
* Agregue [iconos](view-designer-tiles.md) a la vista personalizada.
* Agregue [elementos de visualización](view-designer-parts.md) a la vista personalizada.
