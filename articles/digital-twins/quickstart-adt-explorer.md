---
title: Exploración de un escenario de ejemplo
titleSuffix: Azure Digital Twins
description: Use el ejemplo Explorador de ADT para visualizar y explorar un escenario creado previamente.
author: baanders
ms.author: baanders
ms.date: 8/12/2020
ms.topic: quickstart
ms.service: digital-twins
ms.openlocfilehash: bbb1378419c68db07be5ca14ca6834810f2fc9f5
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055471"
---
# <a name="explore-a-sample-azure-digital-twins-scenario-using-adt-explorer"></a>Exploración de un escenario de Azure Digital Twins de ejemplo con Explorador de ADT

Con Azure Digital Twins, puede crear modelos en directo de los entornos del mundo real e interactuar con ellos. Para ello, se modelan los elementos individuales como **gemelos digitales** y, a continuación, se conectan a un **grafo** de conocimiento que puede responder a eventos en directo y al que se puede consultar información.

En esta guía de inicio rápido, explorará un grafo de Azure Digital Twins con la ayuda de una aplicación de ejemplo llamada [**Explorador de Azure Digital Twins (ADT)** ](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Explorador de ADT le permite cargar un escenario, ver representaciones visuales de los gemelos y los grafos y realizar otras actividades de administración mediante una experiencia visual basada en el explorador web.

La guía de inicio rápido contiene los siguientes pasos principales:

1. Configurar Explorador de ADT y una instancia de Azure Digital Twins.
1. Cargar modelos creados previamente y datos de grafos para construir el escenario de ejemplo.
1. Explorar el grafo de escenario que se crea.
1. Realizar cambios en el grafo.

El grafo de ejemplo con el que va a trabajar representa un edificio con dos plantas y dos habitaciones. El grafo será así:

:::image type="content" source="media/quickstart-adt-explorer/graph-view-full.png" alt-text="Vista de un grafo compuesto por cuatro nodos circulares conectados por flechas. Un círculo con la etiqueta "Floor1" está conectado mediante una flecha con la etiqueta "contains" (contiene) a un círculo con la etiqueta "Room1" y un círculo con la etiqueta "Floor0" está conectado mediante una flecha con la etiqueta "contains" (contiene) a un círculo con la etiqueta "Room0". "Floor1" y "Floor0" no están conectados.":::

## <a name="prerequisites"></a>Requisitos previos

Necesitará una suscripción de Azure para completar esta guía de inicio rápido. Si aún no tiene una, **[cree una gratis](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)** .

Antes de iniciar la guía de inicio rápido, también deberá descargar dos ejemplos:
* La aplicación de ejemplo **Explorador de ADT**. Este ejemplo contiene la aplicación principal que se usa en la guía de inicio rápido para cargar y explorar un escenario de Azure Digital Twins. Para obtener la aplicación, vaya aquí: [Explorador de Azure Digital Twins (ADT)](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Presione el botón *Descargar archivo ZIP* para descargar el archivo *.ZIP* de este código de ejemplo en la máquina como _**ADT_Explorer.zip**_.
* El **escenario de Azure Digital Twins de ejemplo**. Incluye un grafo de Azure Digital Twins creado previamente que cargará en Explorador de ADT para trabajar con él. Para obtener el escenario, vaya aquí: [Ejemplos de Azure Digital Twins](https://docs.microsoft.com/samples/azure-samples/digital-twins-samples/digital-twins-samples). Presione el botón *Descargar archivo ZIP* para descargar el archivo *.ZIP* de este código de ejemplo en la máquina como _**Azure_Digital_Twins_samples.zip**_.

## <a name="set-up-azure-digital-twins-and-adt-explorer"></a>Configuración de Azure Digital Twins y Explorador de ADT

El primer paso para trabajar con Azure Digital Twins es configurar una **instancia de Azure Digital Twins**. Después de crear una instancia del servicio, podrá rellenarla con los datos de ejemplo más adelante en la guía de inicio rápido.

También configurará permisos para que Explorador de ADT se ejecute en el equipo y acceda a la instancia de Azure Digital Twins. Esto le permitirá usar la aplicación de ejemplo para explorar su instancia y sus datos.

### <a name="set-up-azure-digital-twins-instance"></a>Configuración de la instancia de Azure Digital Twins

La manera más sencilla de configurar una instancia y la autenticación necesaria es ejecutar un ejemplo de script de implementación automatizada. Siga las instrucciones de [*Procedimiento: Configuración de una instancia y autenticación (con scripts)* ](how-to-set-up-instance-scripted.md). Las instrucciones también contienen pasos para comprobar que ha completado cada paso correctamente y está listo para pasar a usar la nueva instancia.

En esta guía de inicio rápido, necesitará los valores siguientes que obtuvo al configurar la instancia. Si tiene que volver a recopilar estos valores, use los vínculos siguientes a las secciones correspondientes del artículo de instalación para encontrarlos en [Azure Portal](https://portal.azure.com).
* El **_nombre de host_** de la instancia de Azure Digital Twins ([que se encuentra en el portal](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values))
* El **_identificador de la aplicación (cliente)_** del registro de aplicación de Azure AD ([que se encuentra en el portal](how-to-set-up-instance-portal.md#collect-important-values))
* El **_identificador de directorio (inquilino)_**  del registro de aplicación de Azure AD ([que se encuentra en el portal](how-to-set-up-instance-portal.md#collect-important-values))

### <a name="set-adt-explorer-permissions"></a>Establecimiento de los permisos de Explorador de ADT

A continuación, va a preparar la instancia de Azure Digital Twins que creó para trabajar con Explorador de ADT, que es una aplicación web hospedada localmente. Vaya a la página [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) de Azure Portal y seleccione el nombre del registro de la aplicación en la lista.

Seleccione *Autenticación* en el menú del registro y presione *+ Agregar una plataforma*.

:::image type="content" source="media/quickstart-adt-explorer/authentication-pre.png" alt-text="Página de Azure Portal de los detalles de autenticación de un registro de aplicaciones. Hay un resaltado alrededor del botón "Agregar una plataforma"." lightbox="media/quickstart-adt-explorer/authentication-pre.png":::

En la página *Configurar plataformas* que aparece a continuación, seleccione *Web*.
Rellene los detalles de configuración como se indica a continuación:
* **URI de redirección**: agregue el identificador URI de redirección *http://localhost:3000* .
* **Concesión implícita**: active la casilla *Tokens de acceso*.

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/authentication-configure-web.png" alt-text="Página Configurar plataformas, en la que se resalta la información descrita anteriormente en la pantalla":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Presione *Configurar* para finalizar.

Ahora tiene una configuración web que Explorador de ADT podrá utilizar. La pestaña Autenticación en Azure Portal debe reflejar esto.

:::image type="content" source="media/quickstart-adt-explorer/authentication-post.png" alt-text="Página de Azure Portal de los detalles de autenticación de un registro de aplicaciones. Hay resaltados en la sección de plataforma web con el identificador URI de redirección http://localhost:3000 y en la opción Concesión implícita habilitada para los tokens de acceso.":::

### <a name="run-and-configure-adt-explorer"></a>Ejecución y configuración de Explorador de ADT

A continuación, ejecute la aplicación Explorador de ADT y configúrela para la instancia de Azure Digital Twins.

Vaya a la carpeta _**ADT_Explorer.zip**_ descargada y descomprima el archivo. Abra un símbolo del sistema en la ubicación de la carpeta *ADT_explorer/client/src*.

Ejecute `npm install` para descargar todas las dependencias necesarias.

A continuación, inicie la aplicación; para ello, ejecute `npm run start`.

Después de unos segundos, se abrirá una ventana del explorador y la aplicación aparecerá en el explorador.

:::image type="content" source="media/quickstart-adt-explorer/explorer-blank.png" alt-text="Ventana del explorador que muestra una aplicación que se ejecuta en localhost:3000. La aplicación se llama Explorador de ADT y contiene los cuadros Query Explorer (Explorador de consultas), Model View (Vista de modelo), Graph View (Vista de grafo) y Property Explorer (Explorador de propiedades). Todavía no hay datos en pantalla." lightbox="media/quickstart-adt-explorer/explorer-blank.png":::

Presione el botón *Sign in* (Iniciar sesión) de la parte superior de la ventana para configurar Explorador de ADT para que trabaje con la instancia que ha configurado. 

:::image type="content" source="media/quickstart-adt-explorer/sign-in.png" alt-text="Explorador de ADT con el icono de inicio de sesión resaltado cerca de la parte superior de la ventana. El icono muestra una silueta simple de una persona superpuesta con la silueta de una llave." lightbox="media/quickstart-adt-explorer/sign-in.png":::

Escriba la información importante que recopiló anteriormente en la sección [Requisitos previos](#prerequisites):
* Id. de aplicación (cliente)
* Id. de directorio (inquilino)
* Dirección URL de ADT, con el formato *https://{nombre de host de la instancia}*

>[!NOTE]
> Puede volver a visitar o editar esta información en cualquier momento; para ello, seleccione el mismo icono para volver a mostrar el cuadro de inicio de sesión. Mantendrá los valores que ha especificado.

Si ve la ventana emergente *Permisos solicitados* de Microsoft, conceda el consentimiento para esta aplicación y acepte para continuar.

## <a name="add-the-sample-data"></a>Adición de los datos de ejemplo

A continuación, importará el escenario de ejemplo y el grafo en Explorador de ADT.

El escenario de ejemplo se encuentra en la carpeta _**Azure_Digital_Twins_samples.zip**_ descargada, por lo que debe ir a la carpeta y descomprimirla.

### <a name="models"></a>Modelos

El primer paso de una solución de Azure Digital Twins es definir el vocabulario para el entorno. Esto se hace mediante la creación de [**modelos**](concepts-models.md), personalizados que describen los tipos de entidades que existen en el entorno. 

Los modelos se escriben en un lenguaje de tipo JSON-LD llamado **Digital Twin Definition Language (DTDL)** , que describe un único tipo de entidad en términos de sus *propiedades*, *telemetría*, *relaciones* y *componentes*. Más adelante, usará estos modelos como base para gemelos digitales que representan instancias específicas de estos tipos.

Normalmente, al crear un modelo realizará tres pasos:
1. Escribir la definición del modelo (en la guía de inicio rápido ya viene realizado como parte de la solución de ejemplo)
2. Validarla para asegurarse de que la sintaxis es correcta (en la guía de inicio rápido ya viene realizado como parte de la solución de ejemplo)
3. Cargarla en la instancia de Azure Digital Twins
 
En esta guía de inicio rápido, los archivos de modelo ya se han escrito y validado y se incluyen con la solución descargada. En esta sección, se cargarán los dos modelos escritos previamente en la instancia para definir estos componentes del entorno de un edificio:
* Floor
* Sala

#### <a name="upload-models"></a>Carga de modelos

En el cuadro *MODEL VIEW* (Vista de modelo), presione el icono *Upload a Model* (Cargar un modelo).

:::image type="content" source="media/quickstart-adt-explorer/upload-model.png" alt-text="Cuadro Vista de modelo con el icono central resaltado. Muestra una flecha que apunta a una nube." lightbox="media/quickstart-adt-explorer/upload-model.png":::
 
1. En el cuadro del selector de archivos que aparece, vaya a la carpeta *Azure_Digital_Twins_samples/AdtSampleApp/SampleClientApp/models* en el repositorio descargado.
2. Seleccione *Room.json* y *Floor.json* y presione Aceptar. (Puede cargar los otros modelos si lo desea, pero no se usarán en esta guía de inicio rápido).
3. Siga el cuadro de diálogo emergente que le pide que inicie sesión en su cuenta de Azure.

>[!NOTE]
>Si aparece el siguiente mensaje de error: :::image type="content" source="media/quickstart-adt-explorer/error-models-popup.png" alt-text="Un elemento emergente que muestra "Error: Error al recuperar modelos: ClientAuthError: Error al abrir la ventana emergente. Esto puede ocurrir si usa Internet Explorer o si los elementos emergentes están bloqueados en el explorador" con el botón Cerrar en la parte inferior" border="false"::: 
> Intente deshabilitar el bloqueador de elementos emergentes o usar otro explorador.

Explorador de ADT cargará ahora estos archivos de modelo en la instancia de Azure Digital Twins. Deberían aparecer en el cuadro *MODEL VIEW* (Vista de modelo), que muestra los nombres descriptivos y los identificadores de modelo completos. Puede hacer clic en las burbujas de información de la *Vista de modelo* para ver el código DTDL subyacente.

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/model-info.png" alt-text="Cuadro de vista de modelos con dos definiciones de modelos en su interior, Floor (dtmi:example:Floor;1) y Room (dtmi:example:Room;1). El icono "Ver modelo" muestra una letra "i" en un círculo resaltada para cada modelo." lightbox="media/quickstart-adt-explorer/model-info.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

### <a name="twins-and-the-twin-graph"></a>Gemelos y grafo de gemelos

Ahora que se han cargado algunos modelos en la instancia de Azure Digital Twins, puede crear [**gemelos digitales**](concepts-twins-graph.md) basados en las definiciones de modelo. 

Los gemelos digitales representan las entidades reales del entorno empresarial, como los sensores de una granja, las luces de un coche o, en esta guía de inicio rápido, las salas de la planta de un edificio. Puede crear muchos gemelos de un tipo de modelo determinado (por ejemplo, varias salas que usan el modelo *Room*) y conectarlos con relaciones en un **grafo de gemelos** que representa el entorno completo.

En esta sección, cargará los gemelos creados previamente que están conectados en un grafo creado previamente. El grafo contiene dos plantas y dos salas, conectadas en el siguiente diseño:
* *Floor0*
    - contiene *Room0*
* *Floor1*
    - contiene *Room1*

#### <a name="import-the-graph"></a>Importación de grafo

En el cuadro *GRAPH VIEW* (Vista de grafo), presione el icono *Import Graph* (Importar grafo).

:::image type="content" source="media/quickstart-adt-explorer/import-graph.png" alt-text="Hay un icono resaltado en el cuadro de texto Vista de grafo. Muestra una flecha que apunta a una nube." lightbox="media/quickstart-adt-explorer/import-graph.png":::

En el cuadro del selector de archivos, vaya a la carpeta *Azure_Digital_Twins_samples/AdtSampleApp/SampleClientApp* y elija el archivo de hoja de cálculo _**buildingScenario.xlsx**_. Este archivo contiene una descripción del grafo de ejemplo. Presione OK (Aceptar).

Después de unos segundos, Explorador de ADT abrirá un vista de *importación* que muestra una vista previa del grafo que se va a cargar.

Para confirmar la carga del grafo, haga clic en el icono *Save* (Guardar) situado en la esquina superior derecha de la *Vista de grafo*:

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/graph-preview-save.png" alt-text="Resaltado del icono Guardar en el panel de vista previa del grafo" lightbox="media/quickstart-adt-explorer/graph-preview-save.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Explorador de ADT usará ahora el archivo cargado para crear los gemelos solicitados y las relaciones entre ellos. Aparecerá un cuadro de diálogo para indicar que ha finalizado. Presione *Close* (Cerrar).

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/import-success.png" alt-text="Cuadro de diálogo que indica que el grafo se ha importado correctamente. Indica "Importación correcta. 49 gemelos importados. 50 relaciones importadas."" lightbox="media/quickstart-adt-explorer/import-success.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

El grafo ya se ha cargado en Explorador de ADT. Para ver el grafo, presione el botón *Run Query* (Ejecutar consulta) en el cuadro *QUERY EXPLORER* (Explorador de consultas), cerca de la parte superior de la ventana de Explorador de ADT. 

:::image type="content" source="media/quickstart-adt-explorer/run-query.png" alt-text="El botón Ejecutar consulta cerca de la parte superior de la ventana aparece resaltado" lightbox="media/quickstart-adt-explorer/run-query.png":::

Se ejecutará la consulta predeterminada para seleccionar y mostrar todos los gemelos digitales. Explorador de ADT recuperará todos los gemelos y las relaciones del servicio y dibujará el grafo que definen en el cuadro *GRAPH VIEW* (Vista de grafo).

## <a name="explore-the-graph"></a>Exploración del grafo

Ahora puede ver el grafo cargado del escenario de ejemplo:

:::image type="content" source="media/quickstart-adt-explorer/graph-view-full.png" alt-text="Vista de grafo con un grafo de gemelos en su interior. Un círculo con la etiqueta "floor1" está conectado mediante una flecha con la etiqueta "contains" (contiene) a un círculo con la etiqueta "room1"; un círculo con la etiqueta "floor0" está conectado mediante una flecha con la etiqueta "contains" (contiene) a un círculo con la etiqueta "room0".":::

Los círculos ("nodos" del grafo) representan los gemelos digitales y las líneas representan las relaciones. Puede ver que el gemelo *Floor0* contiene a *Room0* y que el gemelo *Floor1* contiene a *Room1*.

Si usa un ratón, puede hacer clic y arrastrar los elementos del grafo para desplazarlos.

### <a name="view-twin-properties"></a>Visualización de las propiedades de un gemelo 

Puede seleccionar un gemelo para ver una lista de sus propiedades y sus valores en el cuadro *PROPERTY EXPLORER* (Explorador de propiedades). 

Estas son las propiedades de *Room0*:

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/properties-room0.png" alt-text="Resaltado del cuadro Explorador de propiedades que muestra las propiedades de Room0, incluidos (entre otros) el campo $dtId con el valor "Room0", el campo Temperature con el valor 70 y el campo Humidity con el valor 30." lightbox="media/quickstart-adt-explorer/properties-room0.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Observe que *Room0* tiene una temperatura de **70**.

Estas son las propiedades de *Room1*:

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/properties-room1.png" alt-text="Resaltado del cuadro Explorador de propiedades que muestra las propiedades de Room1, incluidos (entre otros) el campo $dtId con el valor "Room1", el campo Temperature con el valor 80 y el campo Humidity con el valor 60." lightbox="media/quickstart-adt-explorer/properties-room1.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Observe que *Room1* tiene una temperatura de **80**.

### <a name="query-the-graph"></a>Consulta del grafo

Una de las principales características de Azure Digital Twins es la posibilidad de [consultar](concepts-query-language.md) el gráfico de gemelos de forma fácil y eficaz para responder a las preguntas sobre el entorno. 

Una forma de consultar los gemelos en el grafo es mediante sus *propiedades*. La consulta basada en propiedades puede ayudar a responder diversas preguntas, como buscar valores atípicos en el entorno que puedan necesitar atención.

En esta sección, ejecutará una consulta para responder a la siguiente pregunta: _**¿Cuáles son todos los gemelos de mi entorno con una temperatura superior a 75?**_

Para ver la respuesta, ejecute la siguiente consulta en el cuadro *QUERY EXPLORER* (Explorador de consultas):

```SQL
SELECT * FROM DigitalTwins T WHERE T.Temperature > 75
```

Recuerde que al ver las propiedades gemelas anteriores *Room0* tenía una temperatura de **70** y *Room1* tenía una temperatura de **80**. Por lo tanto, solo aparece _**Room1**_ en los resultados.
    
:::image type="content" source="media/quickstart-adt-explorer/result-query-property-before.png" alt-text="Resultados de la consulta de propiedades, que muestran solo Room1" lightbox="media/quickstart-adt-explorer/result-query-property-before.png":::

>[!TIP]
> También se admiten otros operadores de comparación ( *<* , *>* , *=* o *!=* ) en la consulta anterior. Puede intentar conectar estos valores, valores diferentes o propiedades del gemelo diferentes en la consulta para intentar responder a sus propias preguntas.

## <a name="edit-data-in-the-graph"></a>Edición de los datos del grafo

Puede usar Explorador de ADT para editar las propiedades de los gemelos representados en el grafo. En esta sección, vamos a **_elevar la temperatura de_ Room0 a 76**.

Para ello, seleccione *Room0*, mostrando su lista de propiedades en el cuadro *PROPERTY EXPLORER* (Explorador de propiedades).

Las propiedades de esta lista son editables. Seleccione el valor de temperatura **70** para habilitar la entrada de un nuevo valor. Escriba **76** y presione el icono *Save* (Guardar) para actualizar la temperatura a **76**.

:::row:::
    :::column:::
        :::image type="content" source="media/quickstart-adt-explorer/new-properties-room0.png" alt-text="Cuadro "Explorador de propiedades" que muestra las propiedades de Room0. El valor de temperatura es un cuadro editable que muestra 76 y hay un resaltado alrededor del icono de guardar." lightbox="media/quickstart-adt-explorer/new-properties-room0.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Después de guardarlo correctamente, verá la ventana de *información de revisión* que muestra el código de revisión que se usó en segundo plano con las [API](how-to-use-apis-sdks.md) de Azure Digital Twins para realizar la actualización. Presione *Close* (Cerrar).

### <a name="query-to-see-the-result"></a>Ejecución de la consulta para ver el resultado

Para comprobar que el grafo ha registrado correctamente la actualización de la temperatura de *Room0*, vuelva a ejecutar la consulta anterior para **obtener todos los gemelos del entorno con una temperatura superior a 75**:

```SQL
SELECT * FROM DigitalTwins T WHERE T.Temperature > 75
```

Ahora que se ha cambiado la temperatura de *Room0* de **70** a **76**, ambos gemelos deberían aparecer en el resultado.

:::image type="content" source="media/quickstart-adt-explorer/result-query-property-after.png" alt-text="Resultados de la consulta de propiedades, que muestra Room0 y Room1" lightbox="media/quickstart-adt-explorer/result-query-property-after.png":::

## <a name="review-and-contextualize-learnings"></a>Revisión y contextualización del aprendizaje

En esta guía de inicio rápido, ha creado una instancia de Azure Digital Twins, la ha conectado a Explorador de ADT y la ha rellenado con un escenario de ejemplo. 

Después, ha explorado el grafo mediante:
1. El uso de una consulta para responder a una pregunta sobre el escenario.
2. La edición de una propiedad en un gemelo digital.
    * La ejecución de la consulta de nuevo para ver cómo ha cambiado la respuesta como resultado de la actualización.

La intención de este ejercicio es demostrar cómo puede usar el grafo de Azure Digital Twins para responder a preguntas sobre el entorno, incluso a medida que el entorno cambia. 

Aunque en esta guía de inicio rápido ha realizado la actualización de la temperatura manualmente, es habitual en Azure Digital Twins conectar gemelos digitales a dispositivos IoT reales para que reciban actualizaciones automáticamente en función de los datos de telemetría. Esto le permite crear un grafo dinámico que siempre refleja el estado real del entorno y usar consultas para obtener información sobre lo que ocurre en el entorno en tiempo real.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para finalizar el trabajo de esta guía de inicio rápido, primero finalice la aplicación de consola en ejecución. Se cerrará la conexión a la aplicación Explorador de ADT en el explorador web y ya no podrá ver los datos en directo en el explorador. Puede cerrar la pestaña del explorador.

Si tiene previsto seguir con los tutoriales de Azure Digital Twins, la instancia que se usa en esta guía de inicio rápido se puede volver a usar para esos artículos y no es necesario eliminarla.
 
[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

Finalmente, elimine las carpetas de ejemplo del proyecto que descargó en el equipo local (_**ADT_Explorer.zip**_ y _**Azure_Digital_Twins_samples.zip**_).

## <a name="next-steps"></a>Pasos siguientes 

A continuación, siga con los tutoriales de Azure Digital Twins para crear su propio escenario de Azure Digital Twins y las herramientas de interacción.

> [!div class="nextstepaction"]
> [*Tutorial: Programación de una aplicación cliente*](tutorial-code.md)