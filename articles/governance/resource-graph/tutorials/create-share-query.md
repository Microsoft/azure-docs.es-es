---
title: 'Tutorial: Administración de consultas en Azure Portal'
description: En este tutorial, creará una consulta de Resource Graph y la compartirá con otras personas en Azure Portal.
ms.date: 06/29/2020
ms.topic: tutorial
ms.openlocfilehash: f602a6ed26f467f2d3395bdf5e3346c83684b108
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005172"
---
# <a name="tutorial-create-and-share-an-azure-resource-graph-query-in-the-azure-portal"></a>Tutorial: Creación y uso compartido de una consulta de Azure Resource Graph en Azure Portal

Azure Resource Graph Explorer le permite guardar consultas de Resource Graph directamente en Azure Portal. Existen dos tipos de consultas: _Privadas_ y _compartidas_. Una consulta privada se guarda en la configuración de Azure Portal. Una consulta compartida es un recurso de Azure Resource Manager que se puede administrar con el control de acceso basado en roles (RBAC) y se puede proteger con bloqueos de recursos. Ambos tipos de consultas se cifran en reposo.

Al guardar las consultas en Azure Portal, ahorrará el tiempo que, de otro modo, habría pasado buscando sus consultas favoritas o usadas con frecuencia. Al compartir consultas, ayudará al equipo a lograr los objetivos de coherencia y eficacia a través de la repetición.

En este tutorial, va a completar las siguientes tareas:

> [!div class="checklist"]
> - Creación y eliminación de una consulta privada.
> - Creación de una consulta compartida.
> - Detección de consultas compartidas.
> - Eliminación de una consulta compartida.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará una suscripción de Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-and-delete-a-private-query"></a>Creación y eliminación de una consulta privada.

Puede visualizar y tener acceso a las consultas privadas solo desde la cuenta que las crea. Dado que se guardan en la configuración de Azure Portal de una cuenta, se pueden crear, usar y eliminar solo desde dentro de Azure Portal. Una consulta privada no es un recurso de Resource Manager. Para crear una nueva consulta privada, siga estos pasos:

1. En el menú del portal, seleccione **Todos los servicios** o use el cuadro de búsqueda de Azure en la parte superior de todas las páginas. Busque y después seleccione **Resource Graph Explorer**.

1. En la pestaña **Consulta 1** de la página de Azure Resource Graph Explorer, escriba la siguiente consulta:

   ```kusto
   Resources
   | where type =~ 'Microsoft.Compute/virtualMachines'
   | summarize count() by tostring(properties.storageProfile.osDisk.osType)
   ```

   Seleccione **Ejecutar consulta** para ver los resultados de la consulta en el panel inferior.

   Para obtener más información acerca de esta consulta, consulte [Ejemplos: conteo de máquinas virtuales por tipo de sistema operativo](../samples/starter.md#count-os).

1. Seleccione **Guardar** o **Guardar como**, escriba **Count VMs by OS** como nombre, deje el tipo como **Consulta privada** y después seleccione **Guardar** en la parte inferior del panel **Guardar consulta**. El título de la pestaña cambia de **Consulta 1** a **Count VMs by OS**.

1. Salga de Azure Resource Graph Explorer en Azure Portal y vuelva a entrar. Observe que la consulta guardada ya no se muestra y que ha vuelto la pestaña **Consulta 1**.

1. Seleccione **Abrir una consulta**. Asegúrese de que el tipo es **Consulta privada**. El nombre guardado **Count VMs by OS** aparece ahora en la lista de **Nombre de consulta**. Al seleccionar el vínculo de título de la consulta guardada, se carga una nueva pestaña con el nombre de esa consulta.

   > [!NOTE] 
   > Cuando una consulta guardada está abierta y la pestaña muestra su nombre, puede seleccionar el botón **Guardar** para actualizarla con los cambios que se hayan realizado. Para crear una nueva consulta guardada a partir de esta consulta abierta, seleccione **Guardar como** y continúe como si estuviera guardando una consulta totalmente nueva.

1. Para eliminar la consulta guardada, seleccione nuevamente **Abrir una consulta** y compruebe que el campo **Tipo** está establecido en **Consulta privada**. En la fila de la consulta de `Count VMs by OS` guardada, seleccione **Eliminar** (icono de papelera de reciclaje). En el cuadro de diálogo de confirmación, seleccione **Sí** para terminar de eliminar la consulta.
   A continuación, cierre el panel **Abrir una consulta**.

## <a name="create-a-shared-query"></a>Creación de una consulta compartida.

A diferencia de una consulta privada, una consulta compartida es un recurso de Resource Manager. Esto significa que la consulta se guarda en un grupo de recursos, se puede administrar y controlar con Azure RBAC e incluso se puede proteger con bloqueos de recursos. Ya que es un recurso, cualquier usuario que tenga los permisos adecuados podrá verla y usarla.
Para crear una nueva consulta compartida, siga estos pasos:

1. En el menú del portal, seleccione **Todos los servicios** o use el cuadro de búsqueda de Azure en la parte superior de todas las páginas y seleccione **Resource Graph Explorer**.

1. En la pestaña **Consulta 1** de la página de Azure Resource Graph Explorer, escriba la siguiente consulta:

   ```kusto
   Resources
   | where type =~ 'Microsoft.Compute/virtualMachines'
   | summarize count() by tostring(properties.storageProfile.osDisk.osType)
   ```
    
   Seleccione **Ejecutar consulta** para ver los resultados de la consulta en el panel inferior.

   Para obtener más información acerca de esta consulta, consulte [Ejemplos: conteo de máquinas virtuales por tipo de sistema operativo](../samples/starter.md#count-os).

1. Seleccione **Guardar** o **Guardar como**.

   :::image type="content" source="../media/create-share-query/save-shared-query-buttons.png" alt-text="Guardar la nueva consulta con el botón Guardar" border="false":::

1. En el panel **Guardar consultas**, escriba **Count VMs by OS** para el nombre.

1. Cambie el tipo a **Consulta compartida**, establezca la descripción en **Recuento de máquinas virtuales por tipo de sistema operativo** y establezca la **Suscripción** para especificar dónde se crea el recurso de consulta.

1. Deje activada la casilla de verificación **Publicar en el grupo de recursos resource-graph-queries** y el campo **Ubicación del grupo de recursos** establecido en **(EE. UU.) Centro-oeste de EE. UU.** .

1. Seleccione **Guardar** en la parte inferior del panel **Guardar consulta**. El título de la pestaña cambia de **Consulta 1** a **Count VMs by OS**. La primera vez que se usa el grupo de recursos **resource-graph-queries**, la operación de guardar tarda más tiempo del esperado ya que se crea el grupo de recursos.
   
   :::image type="content" source="../media/create-share-query/save-shared-query-window.png" alt-text="Guardar la nueva consulta como una consulta compartida" border="false":::

   > [!NOTE] 
   > Puede desactivar la casilla **Publicar en el grupo de recursos resource-graph-queries** si quiere proporcionar el nombre de un grupo de recursos existente para guardar en él la consulta compartida. Si usa el grupo de recursos con nombre predeterminado para las consultas, las consultas compartidas serán más fáciles de detectar. También hace que el propósito de ese grupo de recursos sea más claro. Sin embargo, puede optar por seleccionar un grupo de recursos existente por motivos de seguridad en función de los permisos existentes.

1. Salga de Azure Resource Graph Explorer en Azure Portal y vuelva a entrar. Observe que la consulta guardada ya no se muestra y que ha vuelto la pestaña **Consulta 1**.

1. Seleccione **Abrir una consulta**. Compruebe que el tipo está establecido en **Consulta compartida** y la combinación de **Suscripción** y **Grupo de recursos** coincide con el lugar en el que guardó la consulta. El elemento guardado **Count VMs by OS** aparece ahora en la lista de **Nombre de consulta**. Seleccione el vínculo de título de la consulta guardada para cargarla en una nueva pestaña con el nombre de esa consulta. Ya que es una consulta compartida, se muestra un icono en la pestaña situada junto al título, que indica que es compartida.

   :::image type="content" source="../media/create-share-query/show-saved-shared-query.png" alt-text="Mostrar la consulta compartida con el icono" border="false":::

   > [!NOTE] 
   > Cuando una consulta guardada está abierta y la pestaña muestra su nombre, el botón **Guardar** la actualiza con los cambios que se hayan realizado. Para crear una nueva consulta guardada, seleccione **Guardar como** y continúe como si estuviera guardando una consulta totalmente nueva.

## <a name="discover-shared-queries"></a>Detección de consultas compartidas.

Dado que una consulta compartida es un recurso de Resource Manager, hay varias maneras de buscar una:

- En Resource Graph Explorer, seleccione **Abrir una consulta** y establezca el tipo en **Consulta compartida**.
- En la página del portal de consultas de Resource Graph.
- En el grupo de recursos en el que se guardó la consulta compartida.
- A través de una consulta a Resource Graph.

### <a name="view-resource-graph-queries"></a>Visualización de las consultas de Resource Graph

En Azure Portal, la página de consultas de Resource Graph muestra las consultas compartidas a las que tiene acceso la cuenta con la que ha iniciado sesión. Esta página permite filtrar por nombre, suscripción, grupo de recursos y otras propiedades de la consulta de Resource Graph. También puede etiquetar, exportar y eliminar consultas de Resource Graph mediante esta interfaz.

Al seleccionar una de las consultas, se abre la página de consultas de Resource Graph. Al igual que otros recursos de Resource Manager, esta página ofrece información general interactiva junto con el registro de actividad, el control de acceso y las etiquetas. También puede aplicar un bloqueo de recursos directamente desde esta página.

Para ir a la página de consultas de Resource Graph desde el menú del portal, seleccione **Todos los servicios** o use el cuadro de búsqueda de Azure en la parte superior de todas las páginas. Busque y seleccione **Resource Graph Explorer**.

### <a name="list-resource-groups-resources"></a>Enumeración de los recursos de un grupo de recursos

La consulta de Resource Graph se muestra junto a otros recursos que forman parte de un grupo de recursos.
Al seleccionar la consulta de Resource Graph, se abre la página de esa consulta. Los puntos suspensivos y las opciones del menú contextual (desencadenadas al hacer clic con el botón derecho) funcionan igual que en la página de consultas de Resource Graph.

### <a name="query-resource-graph"></a>Envío de una consulta a Resource Graph

Puede buscar consultas a Resource Graph a través de una consulta en el gráfico de recursos. La siguiente consulta de Resource Graph estaba bien fuerte el clima. La siguiente límite de consulta de Resource Graph por tipo `Microsoft.ResourceGraph/queries` y después se usa `project` para mostrar solo el nombre, la hora modificada y la propia consulta:

```kusto
Resources
| where type == "microsoft.resourcegraph/queries"
| project name, properties.timeModified, properties.query
```

## <a name="run-a-shared-query"></a>Ejecución de una consulta compartida

Se puede ejecutar una consulta compartida de Resource Graph con la sintaxis `{{shared-query-uri}}` (versión preliminar). Para más información, consulte [Sintaxis de consulta compartida](../concepts/query-language.md#shared-query-syntax).

## <a name="delete-a-shared-query"></a>Eliminación de una consulta compartida.

Si ya cuenta con una consulta compartida y no se necesita una consulta compartida simplemente se debe delegar. Al eliminar una consulta compartida, se quita el recurso de Resource Manager. Los paneles a los que estaba anclado el gráfico de resultados ahora muestran un mensaje de error. Cuando aparezca el mensaje de error, use el botón **Quitar del panel** para limpiar el panel.

Puede eliminar una consulta compartida a través de las interfaces siguientes:
- Página de consultas de Resource Graph.
- Página de consulta de Resource Graph.
- La página **Abrir una consulta** en Resource Graph Explorer
- Página de grupos de recursos

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado este tutorial, elimine las consultas privadas y compartidas que creó si ya no las quiere

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado consultas privadas y compartidas. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](../concepts/query-language.md)