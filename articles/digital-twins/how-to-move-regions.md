---
title: Migración de una instancia a otra región de Azure
titleSuffix: Azure Digital Twins
description: Vea cómo migrar una instancia de Azure Digital Twins de una región de Azure a otra.
author: baanders
ms.author: baanders
ms.date: 08/26/2020
ms.topic: how-to
ms.custom: subject-moving-resources
ms.service: digital-twins
ms.openlocfilehash: c8f78af8753de0eadc26585adacf04f54c2eb750
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300999"
---
# <a name="move-an-azure-digital-twins-instance-to-a-different-azure-region"></a>Migración de una instancia de Azure Digital Twins a otra región de Azure

Si necesita migrar la instancia de Azure Digital Twins de una región a otra, el proceso actual es **volver a crear los recursos en la nueva región** y, luego, opcionalmente, eliminar los recursos originales. Al final de este proceso, trabajará con una nueva instancia de Azure Digital Twins que es idéntica a la primera, excepto por la ubicación actualizada.

En este artículo se proporcionan instrucciones sobre cómo realizar una migración completa, de forma que se copie todo lo necesario para hacer que la nueva instancia coincida con la original.

Este proceso incluye los siguientes pasos:
1. Preparación: Descargue los modelos, gemelos y grafos originales.
2. Migración: Cree una instancia de Azure Digital Twins en una nueva región.
3. Migración: Vuelva a rellenar la nueva instancia de Azure Digital Twins.
    - Cargue los modelos, gemelos y grafos originales.
    - Vuelva a crear los puntos de conexión y las rutas.
    - Vuelva a vincular los recursos conectados.
4. Limpie los recursos de origen (opcional): elimine la instancia original.

## <a name="prerequisites"></a>Requisitos previos

Antes de intentar volver a crear la instancia de Azure Digital Twins, es una buena idea revisar los componentes de la instancia original y hacerse una idea clara de todas las partes que deben volver a crearse.

Estas son algunas preguntas que puede plantearse:
* ¿Cuáles son los modelos **cargados** en mi instancia? ¿Cuántos hay?
* ¿Cuáles son los **gemelos** de mi instancia? ¿Cuántos hay?
* ¿Cuál es la forma general del **grafo** en mi instancia? ¿Cuántas relaciones hay?
* ¿Qué **puntos de conexión** tengo en mi instancia?
* ¿Qué **rutas** tengo en mi instancia? ¿Tienen filtros?
* ¿Dónde la instancia se **conecta a otros servicios de Azure**? Algunos puntos de integración comunes incluyen...
    - Event Grid, Event Hub o Service Bus
    - Azure Functions
    - Logic Apps
    - Time Series Insights
    - Azure Maps
    - Device Provisioning Service (DPS)
* ¿Qué otras **aplicaciones personales o empresariales** tengo que conectar a mi instancia?

Esta información puede recopilarla mediante el ejemplo de [Azure Portal](https://portal.azure.com), las [API y los SDK de Azure Digital Twins](how-to-use-apis-sdks.md), los [comandos de la CLI de Azure Digital Twins](how-to-use-cli.md) o el [Explorador de Azure Digital Twins (ADT)](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/).

## <a name="prepare"></a>Preparación

En esta sección, se preparará para volver a crear la instancia mediante la **descarga de los modelos, gemelos y grafos originales** de la instancia original. Para ello, use el ejemplo del [Explorador de Azure Digital Twins (ADT)](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/).

>[!NOTE]
>Es posible que ya tenga archivos que contengan los modelos o el grafo de la instancia. Si es así, no es necesario descargar todo de nuevo, solo las partes que faltan o cosas que pueden haber cambiado desde que se cargaron originalmente estos archivos (por ejemplo, los gemelos que pueden haberse actualizado con nuevos datos).

### <a name="set-up-adt-explorer-application"></a>Configuración de la aplicación del Explorador de ADT

En primer lugar, descargue el código de aplicación de ejemplo y prepárelo ejecutarlo en su máquina. 

Vaya al ejemplo este: [Explorador de Azure Digital Twins (ADT)](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Presione el botón *Descargar archivo ZIP* para descargar el archivo *.ZIP* de este código de ejemplo en la máquina como _**ADT_Explorer.zip**_. Descomprima el archivo.

A continuación, configure los permisos para que el Explorador de ADT se ejecute en la máquina. Para ello, siga los pasos descritos en la sección [*Establecimiento de los permisos de Explorador de ADT*](quickstart-adt-explorer.md#set-adt-explorer-permissions) del inicio rápido de Azure Digital Twins.

Por último, ejecute y configure el Explorador de ADT para conectarse a la instancia original de Azure Digital Twins. Siga los pasos de la sección [*Ejecución y configuración de Explorador de ADT*](quickstart-adt-explorer.md#run-and-configure-adt-explorer) del inicio rápido.

Ahora tendrá en ejecución la aplicación de ejemplo del Explorador de ADT en un explorador en la máquina. El ejemplo debe estar conectado a la instancia original de Azure Digital Twins.

:::image type="content" source="media/how-to-move-regions/explorer-blank.png" alt-text="Ventana del explorador que muestra una aplicación que se ejecuta en localhost:3000. La aplicación se llama Explorador de ADT y contiene los cuadros Query Explorer (Explorador de consultas), Model View (Vista de modelo), Graph View (Vista de grafo) y Property Explorer (Explorador de propiedades). Todavía no hay datos en pantalla." lightbox="media/how-to-move-regions/explorer-blank.png":::

Para comprobar la conexión, puede presionar el botón *Ejecutar consulta* para ejecutar la consulta predeterminada que muestra todos los gemelos y relaciones en el grafo en el cuadro *PROBADOR DE GRAPH*.

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="El botón Ejecutar consulta cerca de la parte superior de la ventana aparece resaltado" lightbox="media/how-to-move-regions/run-query.png":::

Puede dejar el Explorador de ADT en ejecución, ya que lo usará más adelante en este artículo para volver a cargar estos elementos en la nueva instancia de la región de destino.

### <a name="download-models-twins-and-graph"></a>Descarga de modelos, gemelos y grafo

A continuación, descargue los modelos, los gemelos y el grafo de la solución en la máquina.

Para descargar todos ellos a la vez, asegúrese primero de que el grafo completo se muestra en el cuadro *GRAPH VIEW* (VISTA DE GRAFO) (puede hacerlo volviendo a ejecutar la consulta predeterminada de `SELECT * FROM digitaltwins` en el cuadro *EXPLORADOR DE CONSULTAS*).
 
A continuación, presione el icono *Export graph* (Exportar grafo) en el cuadro *GRAPH VIEW* (VISTA DE GRAFO).

:::image type="content" source="media/how-to-move-regions/export-graph.png" alt-text="Hay un icono resaltado en el cuadro de texto Vista de grafo. Se muestra una flecha que apunta hacia fuera de una nube." lightbox="media/how-to-move-regions/export-graph.png":::

Esta acción habilitará un vínculo *Descargar* en *GRAPH VIEW* (VISTA DE GRAFO). Selecciónelo para descargar una representación basada en JSON del resultado de la consulta, incluidos los modelos, los gemelos y las relaciones. Se descargará un archivo *.json* en la máquina.

>[!NOTE]
>Si el archivo descargado parece tener una extensión de archivo diferente, pruebe a editar la extensión directamente y cámbiela por *.json*.

## <a name="move"></a>Mover

A continuación, creará una instancia en la región de destino y la rellenará con los datos y los componentes de la instancia original. Así finalizará el paso de migración.

### <a name="create-a-new-instance"></a>Creación de una instancia

En primer lugar, **crear una instancia de Azure Digital Twins en la región de destino**. Para ello, siga los pasos descritos en [*Procedimiento: Configuración de una instancia y autenticación*](how-to-set-up-instance-scripted.md), teniendo en cuenta estas indicaciones:
* Puede mantener el mismo nombre para la nueva instancia **si** está en otro grupo de recursos. Si necesita usar el mismo grupo de recursos que contiene la instancia original, la nueva instancia necesitará su propio nombre distintivo.
* Cuando se le solicite una ubicación, escriba la nueva región de destino.
* No **necesita** volver a crear el registro de aplicaciones. La nueva instancia puede volver a usar el que ya tiene.
    - Si usa las instrucciones del artículo de configuración [con script](how-to-set-up-instance-scripted.md), puede volver a escribir los detalles del registro de aplicaciones existente en lugar de especificar un nuevo nombre cuando se le solicite.
    - Si usa las instruccioes de los artículos de configuración manual del [portal](how-to-set-up-instance-portal.md) o de la [CLI](how-to-set-up-instance-cli.md), puede detenerse después de los pasos *Creación de la instancia de Azure Digital Twins* y *Configuración de los permisos de acceso de usuario*. No es necesario continuar con *Configuración de los permisos de acceso para las aplicaciones cliente*.

Cuando haya finalizado, necesitará el **nombre de host** de la nueva instancia para continuar con la configuración de los datos. Si no ha tomado nota de esta información durante la configuración, puede seguir [estas instrucciones](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values) para obtenerla ahora desde Azure Portal.

### <a name="repopulate-old-instance"></a>Volver a rellenar la instancia anterior

A continuación, configurará la nueva instancia para que sea una copia de la original.

#### <a name="upload-original-models-twins-and-graph-using-adt-explorer"></a>Carga de los modelos, los gemelos y el grafo originales con el Explorador de ADT

En esta sección, puede volver a cargar los modelos, los gemelos y el grafo en la nueva instancia. Si no tiene ninguno de estos en la instancia original o no desea migrarlos a la nueva instancia, puede ir directamente a la [siguiente sección](#recreate-endpoints-and-routes).

De lo contrario, para continuar, ejecute el **Explorador de ADT** para volver a la ventana del explorador y siga los pasos que se indican a continuación.

##### <a name="connect-to-the-new-instance"></a>Conexión a la nueva instancia

Actualmente, el Explorador de ADT está conectado a la instancia original de Azure Digital Twins. Cambie la conexión para que apunte a la nueva instancia; para ello, presione el botón *Iniciar sesión* en la parte superior de la ventana. 

:::image type="content" source="media/how-to-move-regions/sign-in.png" alt-text="Explorador de ADT con el icono de inicio de sesión resaltado cerca de la parte superior de la ventana. El icono muestra una silueta simple de una persona superpuesta con la silueta de una llave." lightbox="media/how-to-move-regions/sign-in.png":::

Puesto que va a reutilizar el registro de aplicaciones, solo debe reemplazar la *dirección URL de ADT*. Cambie este valor por el que pone *https://{new instance hostname}* .

Presione *Conectar*. Es posible que se le pida que vuelva a iniciar sesión con sus credenciales de Azure o que conceda a esta aplicación el consentimiento para la instancia.

##### <a name="upload-models-twins-and-graph"></a>Carga de modelos, gemelos y grafo

A continuación, cargue los componentes de la solución que descargó anteriormente en la nueva instancia.

Para cargar los **modelos, los gemelos y el grafo**, presione el icono *Import Graph* (Importar grafo) en el cuadro *GRAPH VIEW* (VISTA DE GRAFO). Esta opción cargará los tres componentes a la vez (incluso los modelos que no se usen en ese momento en el grafo).

:::image type="content" source="media/how-to-move-regions/import-graph.png" alt-text="Hay un icono resaltado en el cuadro de texto Vista de grafo. Muestra una flecha que apunta a una nube." lightbox="media/how-to-move-regions/import-graph.png":::

En el cuadro se selección de archivos, desplácese hasta el grafo descargado. Seleccione el archivo *.json* del grafo y presione *Abrir*.

Después de unos segundos, Explorador de ADT abrirá un vista de *importación* que muestra una vista previa del grafo que se va a cargar.

Para confirmar la carga del grafo, haga clic en el icono *Save* (Guardar) situado en la esquina superior derecha de la *Vista de grafo*:

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-move-regions/graph-preview-save.png" alt-text="Resaltado del icono Guardar en el panel de vista previa del grafo" lightbox="media/how-to-move-regions/graph-preview-save.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

El Explorador de ADT cargará ahora los modelos y el grafo (incluidos los gemelos y las relaciones) en la nueva instancia de Azure Digital Twins. Verá un mensaje de confirmación que indica cuántos modelos, gemelos y relaciones se cargaron:

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-move-regions/import-success.png" alt-text="Cuadro de diálogo que indica que el grafo se ha importado correctamente. Indica Importación correcta. 2 modelos importados. 4 gemelos importados. 2 relaciones importadas." lightbox="media/how-to-move-regions/import-success.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Para comprobar que todo se cargó correctamente, presione el botón *Ejecutar consulta* en el cuadro *PROBADOR DE GRAPH* para ejecutar la consulta predeterminada que muestra todos los gemelos y las relaciones del grafo. También se actualizará la lista de modelos en *VISTA DE MODELO*.

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="Resaltado alrededor del mismo botón Ejecutar consulta de antes, cerca de la parte superior de la ventana" lightbox="media/how-to-move-regions/run-query.png":::

Verá el grafo con todos sus gemelos y relaciones mostrados en el cuadro *PROBADOR DE GRAPH*. También verá los modelos enumerados en el cuadro *MODELO DE VISTA*.

:::image type="content" source="media/how-to-move-regions/post-upload.png" alt-text="Una vista del Explorador de ADT que muestra 2 modelos resaltados en el cuadro Vista de modelo y un grafo resaltado en el cuadro Probador de Graph" lightbox="media/how-to-move-regions/post-upload.png":::

Esto confirma que los modelos, los gemelos y el grafo se han vuelto a cargar en la nueva instancia de la región de destino.

#### <a name="recreate-endpoints-and-routes"></a>Nueva creación de puntos de conexión y rutas

Si tiene **puntos de conexión o rutas** en la instancia original, deberá volver a crearlos en la nueva instancia. Si no tiene ningún punto de conexión o ruta en la instancia original o no quiere moverlos a la nueva instancia, puede ir directamente a la [siguiente sección](#re-link-connected-resources).

De lo contrario, siga los pasos descritos en [*Procedimientos: Administración de puntos de conexión y rutas*](how-to-manage-routes-portal.md) con la nueva instancia, teniendo en cuenta estas indicaciones: 
* No **necesita** volver a crear el recurso de Event Grid, Event Hub o Service Bus que usa para el punto de conexión (secciones de *Requisitos previos* de las instrucciones del punto de conexión). Solo tiene que volver a crear el punto de conexión en la instancia de Azure Digital Twins.
* Puede reutilizar los **nombres** de punto de conexión y ruta, ya que tienen como ámbito una instancia diferente.
* No olvide agregar los **filtros** necesarios a las rutas que cree.

#### <a name="re-link-connected-resources"></a>Nueva vinculación de los recursos conectados

Si tiene otras aplicaciones o recursos de Azure que están conectados a la instancia original de Azure Digital Twins, deberá editar la conexión para que se comuniquen con la nueva instancia. Este puede ser el caso de otros servicios de Azure, o aplicaciones personales o empresariales que haya configurado para trabajar con Azure Digital Twins.

Si no tiene ningún otro recurso conectado a la instancia original o no desea migrarlos a la nueva instancia, puede ir directamente a la [siguiente sección](#verify).

De lo contrario, para continuar, tenga en cuenta los recursos conectados en su escenario. No es necesario eliminar y volver a crear los recursos conectados; solo tiene que editar los puntos en los que se conectan a una instancia de Azure Digital Twins mediante su **nombre de host** y actualizar esta para usar el nombre de host de la nueva instancia en lugar del de la instancia original.

Los recursos exactos que necesite editar dependen del escenario, pero estos son algunos puntos de integración comunes:
* Azure Functions. Si tiene una función de Azure cuyo código incluye el nombre de host de la instancia original, debe actualizar este valor al nombre de host de la nueva instancia y volver a publicar la función.
* Event Grid, Event Hubs o Service Bus
* Logic Apps
* Time Series Insights
* Azure Maps
* Device Provisioning Service (DPS)
* Aplicaciones personales o empresariales fuera de Azure, como la **aplicación cliente** creada en el [*Tutorial: Programación de una aplicación cliente*](tutorial-code.md), que se conecta a la instancia y llama a las API de Azure Digital Twins

Después de completar este paso, la nueva instancia de la región de destino debe ser una copia de la instancia original.

## <a name="verify"></a>Comprobar

Para comprobar que la nueva instancia se ha configurado correctamente, puede usar las siguientes herramientas:
* [**Azure Portal**](https://portal.azure.com) (adecuado para comprobar que la nueva instancia existe y que se encuentra en la región de destino correcta; también para comprobar los puntos de conexión y las rutas, y las conexiones a otros servicios de Azure)
* Los [**comandos de la CLI** de Azure Digital Twins](how-to-use-cli.md) (adecuados para comprobar que la nueva instancia existe y que se encuentra en la región de destino correcta; también se pueden usar para comprobar los datos de la instancia)
* [**Explorador de ADT**](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) (adecuado para comprobar los datos de la instancia, como modelos, gemelos y grafos)
* Las [API y los SDK de Azure Digital Twins](how-to-use-apis-sdks.md) (adecuados para comprobar los datos de la instancia, como los modelos, gemelos y grafos; también para comprobar los puntos de conexión y las rutas).

También puede intentar ejecutar cualquier aplicación personalizada o flujo de un extremo a otro que haya ejecutado con la instancia original, a fin de ayudarle a comprobar que están trabajando con la nueva instancia de forma correcta.

## <a name="clean-up-source-resources-optional"></a>Limpieza de los recursos de origen (opcional)

Ahora que la nueva instancia está configurada en la región de destino con una copia de los datos y las conexiones de la instancia original, puede **eliminar la instancia original** si así lo desea.

Para ello, puede usar [Azure Portal](https://portal.azure.com), la [CLI](how-to-use-cli.md) o las [API del panel de control](how-to-use-apis-sdks.md#overview-control-plane-apis).

Para eliminar la instancia mediante Azure Portal, [abra el portal](https://portal.azure.com) en una ventana del explorador y vaya a la instancia original de Azure Digital Twins; para ello, busque su nombre en la barra de búsqueda del portal.

Presione el botón *Eliminar* y siga las indicaciones para finalizar la eliminación.

:::image type="content" source="media/how-to-move-regions/delete-instance.png" alt-text="Vista de los detalles de la instancia de Azure Digital Twins en Azure Portal, en la pestaña Información general. El botón Guardar está resaltado.":::
