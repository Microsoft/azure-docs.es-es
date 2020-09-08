---
title: 'Inicio rápido: Etiquetado de formularios, entrenamiento de un modelo y análisis de un formulario mediante la herramienta de etiquetado de ejemplo: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, utilizará la herramienta de etiquetado de ejemplo Form Recognizer para etiquetar manualmente documentos de formularios. A continuación, entrenará un modelo personalizado con los documentos etiquetados y lo empleará para extraer pares clave-valor.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: quickstart
ms.date: 08/25/2020
ms.author: pafarley
ms.openlocfilehash: 6118f8109f44081c797cb09a6157abaf4044965e
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89377819"
---
# <a name="train-a-form-recognizer-model-with-labels-using-the-sample-labeling-tool"></a>Entrenamiento de un modelo de Form Recognizer con etiquetas mediante la herramienta de etiquetado de ejemplo

En este inicio rápido, usará la API REST de Form Recognizer con la herramienta de etiquetado de ejemplo para entrenar un modelo personalizado con datos etiquetados manualmente. Consulte la sección [Entrenamiento con etiquetas](../overview.md#train-with-labels) de la introducción para más información acerca de esta característica.

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Azure-Form-Recognizer/player]

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, debe cumplir los siguientes requisitos:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Un conjunto de al menos seis formularios del mismo tipo. Usará estos datos para entrenar el modelo y probar un formulario. En este inicio rápido puede usar un [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451). Cargue los archivos de entrenamiento en la raíz de un contenedor de almacenamiento de blobs de una cuenta de Azure Storage de nivel de rendimiento estándar.

## <a name="create-a-form-recognizer-resource"></a>Creación de un recurso de Form Recognizer

[!INCLUDE [create resource](../includes/create-resource.md)]

## <a name="set-up-the-sample-labeling-tool"></a>Configuración de la herramienta de etiquetado de ejemplo

Usará el motor de Docker para ejecutar la herramienta de etiquetado de ejemplo. Siga estos pasos para configurar el contenedor de Docker. Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).

> [!TIP]
> OCR Form Labeling Tool también está disponible como proyecto de código abierto en GitHub. La herramienta es una aplicación web en TypeScript compilada mediante React + Redux. Para más información o para realizar una contribución, consulte el repositorio [OCR Form Labeling Tool](https://github.com/microsoft/OCR-Form-Tools/blob/master/README.md#run-as-web-application). Para probar la herramienta en línea, vaya al [sitio web de FOTT](https://fott.azurewebsites.net/).   

1. En primer lugar, instale Docker en un equipo host. En esta guía se muestra cómo usar el equipo local como un host. Si desea usar un servicio de hospedaje de Docker en Azure, consulte la guía paso a paso [Implementación de la herramienta de etiquetado de ejemplo](../deploy-label-tool.md). 

   El equipo host debe cumplir los siguientes requisitos de hardware:

    | Contenedor | Mínima | Recomendado|
    |:--|:--|:--|
    |Herramienta de etiquetado de ejemplo|2 núcleos, 4 GB de memoria|4 núcleos, 8 GB de memoria|

   Instale Docker en la máquina siguiendo las instrucciones adecuadas para su sistema operativo: 
   * [Windows](https://docs.docker.com/docker-for-windows/)
   * [macOS](https://docs.docker.com/docker-for-mac/)
   * [Linux](https://docs.docker.com/install/)





1. Obtenga el contenedor de la herramienta de etiquetado de ejemplo con el comando `docker pull`.

    # <a name="v20"></a>[v2.0](#tab/v2-0)    
    ```
    docker pull mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool
    ```
    # <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1)    
    ```
    docker pull mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:2.1.012970002-amd64-preview
    ```

    ---

1. Ahora ya está listo para ejecutar el contenedor con `docker run`.

    # <a name="v20"></a>[v2.0](#tab/v2-0)    
    ```
    docker run -it -p 3000:80 mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool eula=accept
    ```
    # <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1)    
    ```
    docker run -it -p 3000:80 mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:2.1.012970002-amd64-preview eula=accept    
    ```

    --- 

   Este comando hará que la herramienta de etiquetado de ejemplo esté disponible mediante un explorador web. Ir a `http://localhost:3000`.

> [!NOTE]
> También puede etiquetar documentos y entrenar modelos mediante la API REST de Form Recognizer. Para entrenar y analizar con la API REST, consulte [Entrenamiento con etiquetas mediante la API REST y Python](./python-labeled-data.md).

## <a name="set-up-input-data"></a>Configuración de datos de entrada

En primer lugar, asegúrese de que todos los documentos de entrenamiento tienen el mismo formato. Si tiene formularios en varios formatos, organícelos en subcarpetas basadas en un formato común. Al entrenar, deberá dirigir la API a una subcarpeta.

### <a name="configure-cross-domain-resource-sharing-cors"></a>Configuración del uso compartido de recursos entre dominios (CORS)

Habilite CORS en la cuenta de almacenamiento. Seleccione la cuenta de almacenamiento en Azure Portal y haga clic en la pestaña **CORS** del panel izquierdo. En la línea inferior, rellene los valores siguientes. Después, haga clic en **Guardar** en la parte superior.

* Orígenes permitidos = * 
* Métodos permitidos = \[seleccionar todos\]
* Encabezados permitidos = *
* Encabezados expuestos = * 
* Antigüedad máxima = 200

> [!div class="mx-imgBorder"]
> ![Configuración de CORS en Azure Portal](../media/label-tool/cors-setup.png)

## <a name="connect-to-the-sample-labeling-tool"></a>Conexión a la herramienta de etiquetado de ejemplo

La herramienta de etiquetado de ejemplo se conecta a un origen (en el que se encuentran los formularios originales) y a un destino (donde exporta las etiquetas creadas y los datos de salida).

Las conexiones se pueden configurar y compartir entre proyectos. Usan un modelo extensible de proveedores, por lo que puede agregar fácilmente nuevos proveedores de origen y destino.

Para crear una nueva conexión, haga clic en el icono **Nuevas conexiones** (un enchufe) en la barra de navegación izquierda.

Rellene los campos con los siguientes valores:

* **Nombre para mostrar**: el nombre para mostrar de la conexión.
* **Descripción**: la descripción del proyecto.
* **Dirección URL de SAS**: la dirección URL de la firma de acceso compartido (SAS) del contenedor de Azure Blob Storage. Para recuperar la dirección URL de SAS, abra el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor y seleccione **Get shared access signature** (Obtener firma de acceso compartido). Establezca la hora de expiración en alguna hora posterior a la hora en la que utilizará el servicio. Asegúrese de que están seleccionados los permisos de **lectura**, **escritura**, **eliminación** y **enumeración** y haga clic en **Crear**. A continuación, copie el valor en la sección **URL**. Debe tener el formato `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.

:::image type="content" source="../media/label-tool/connections.png" alt-text="Configuración de la conexión de la herramienta de etiquetado de ejemplo.":::


## <a name="create-a-new-project"></a>Creación de un nuevo proyecto

En la herramienta de etiquetado de ejemplo, los proyectos almacenan las configuraciones y los valores. Cree un nuevo proyecto y rellene los campos con los siguientes valores:

* **Nombre para mostrar**: el nombre para mostrar del proyecto
* **Token de seguridad**: algunos valores de configuración del proyecto pueden incluir valores confidenciales, como claves de API u otros secretos compartidos. Cada proyecto generará un token de seguridad que se puede usar para cifrar o descifrar los valores de configuración confidenciales del proyecto. Puede buscar los tokens de seguridad en la configuración de la aplicación. Para ello, haga clic en el icono de engranaje situado en la esquina inferior de la barra de navegación izquierda.
* **Conexión de origen**: la conexión de Azure Blob Storage que creó en el paso anterior que le gustaría usar para este proyecto.
* **Ruta de acceso a la carpeta** (opcional): si los formularios de origen se encuentran en una carpeta en el contenedor de blobs, especifique aquí el nombre de la carpeta.
* **URI de servicio del servicio Form Recognizer**: la dirección URL del punto de conexión de Form Recognizer.
* **Clave de API**: la clave de suscripción de Form Recognizer.
* **Descripción** (opcional): descripción del proyecto.

:::image type="content" source="../media/label-tool/new-project.png" alt-text="Página Nuevo proyecto en la herramienta de etiquetado de ejemplo.":::

## <a name="label-your-forms"></a>Etiquetado de formularios

Al crear o abrir un proyecto, se abrirá la ventana principal del editor de etiquetas. El editor de etiquetas consta de tres partes:

* Un panel de vista previa de tamaño variable que contiene una lista desplazable de formularios de la conexión de origen.
* El panel principal del editor que permite aplicar etiquetas.
* El panel del editor de etiquetas que permite a los usuarios modificar, bloquear, reordenar y eliminar etiquetas. 

### <a name="identify-text-elements"></a>Identificación de elementos de texto

Haga clic en **Run OCR on all files** (Ejecutar OCR en todos los archivos) en el panel izquierdo para obtener la información de diseño del texto de cada documento. La herramienta de etiquetado dibujará los cuadros de límite alrededor de cada elemento de texto.

También se mostrarán las tablas que se hayan extraído automáticamente. Haga clic en el icono de tabla o cuadrícula en la parte izquierda del documento para ver la tabla extraída. En esta guía de inicio rápido, dado que el contenido de la tabla se extrae automáticamente, este no se etiquetará, sino que se basará en la extracción automatizada.

:::image type="content" source="../media/label-tool/table-extraction.png" alt-text="Visualización de tablas mediante la herramienta de etiquetado de ejemplo.":::

### <a name="apply-labels-to-text"></a>Aplicación de etiquetas a texto

A continuación, creará etiquetas y las aplicará a los elementos de texto que desea que reconozca el modelo.

# <a name="v20"></a>[v2.0](#tab/v2-0)  
1. Primero, use el panel del editor de etiquetas para crear las etiquetas que le gustaría identificar.
   1. Haga clic en **+** para crear una nueva etiqueta.
   1. Escriba el nombre de la etiqueta.
   1. Presione Entrar para guardar la etiqueta.
1. En el editor principal, haga clic para seleccionar las palabras en los elementos de texto resaltados.
1. Haga clic en la etiqueta que desea aplicar o presione la tecla correspondiente del teclado. Las teclas numéricas se asignan como teclas de acceso rápido para las diez primeras etiquetas. Puede volver a ordenar las etiquetas con los iconos de flecha arriba y abajo del panel del editor de etiquetas.
    > [!Tip]
    > Tenga en cuenta las siguientes sugerencias cuando vaya a etiquetar los formularios.
    > * Solo se puede aplicar una etiqueta a cada elemento de texto seleccionado.
    > * Cada etiqueta solo se puede aplicar una vez por página. Si un valor aparece varias veces en el mismo formulario, cree etiquetas diferentes para cada instancia. Por ejemplo, "factura n.º 1", "factura n.º 2", etc.
    > * Las etiquetas no pueden abarcar varias páginas.
    > * Etiquete los valores tal como aparecen en el formulario; no intente dividir un valor en dos partes con dos etiquetas diferentes. Por ejemplo, un campo de dirección debe etiquetarse con una sola etiqueta incluso si abarca varias líneas.
    > * No incluya claves en los campos etiquetados, solo los valores.
    > * Los datos de la tabla se deben detectar automáticamente y estarán disponibles en el archivo JSON de salida final. Sin embargo, si el modelo no detecta todos los datos de la tabla, también puede etiquetar manualmente estos campos. Etiquete cada celda de la tabla con una etiqueta diferente. Si los formularios tienen tablas con un número variable de filas, asegúrese de etiquetar al menos un formulario con la tabla más grande posible.
    > * Use los botones situados a la derecha de **+** para buscar, reordenar y eliminar las etiquetas, así como cambiarles el nombre.
    > * Para quitar una etiqueta aplicada sin eliminar la etiqueta en sí, seleccione el rectángulo etiquetado en la vista de documento y presione la tecla Supr.


# <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1) 
1. Primero, use el panel del editor de etiquetas para crear las etiquetas que le gustaría identificar.
   1. Haga clic en **+** para crear una nueva etiqueta.
   1. Escriba el nombre de la etiqueta.
   1. Presione Entrar para guardar la etiqueta.
1. En el editor principal, haga clic para seleccionar las palabras en los elementos de texto resaltados. En la _versión preliminar v2.1_, también puede hacer clic para seleccionar _marcas de selección_ como botones de radio y casillas de verificación como pares clave-valor. Form Recognizer identificará si la marca de selección está "activada" o "desactivada" como el valor.
1. Haga clic en la etiqueta que desea aplicar o presione la tecla correspondiente del teclado. Las teclas numéricas se asignan como teclas de acceso rápido para las diez primeras etiquetas. Puede volver a ordenar las etiquetas con los iconos de flecha arriba y abajo del panel del editor de etiquetas.
    > [!Tip]
    > Tenga en cuenta las siguientes sugerencias cuando vaya a etiquetar los formularios.
    > * Solo se puede aplicar una etiqueta a cada elemento de texto seleccionado.
    > * Cada etiqueta solo se puede aplicar una vez por página. Si un valor aparece varias veces en el mismo formulario, cree etiquetas diferentes para cada instancia. Por ejemplo, "factura n.º 1", "factura n.º 2", etc.
    > * Las etiquetas no pueden abarcar varias páginas.
    > * Etiquete los valores tal como aparecen en el formulario; no intente dividir un valor en dos partes con dos etiquetas diferentes. Por ejemplo, un campo de dirección debe etiquetarse con una sola etiqueta incluso si abarca varias líneas.
    > * No incluya claves en los campos etiquetados, solo los valores.
    > * Los datos de la tabla se deben detectar automáticamente y estarán disponibles en el archivo JSON de salida final. Sin embargo, si el modelo no detecta todos los datos de la tabla, también puede etiquetar manualmente estos campos. Etiquete cada celda de la tabla con una etiqueta diferente. Si los formularios tienen tablas con un número variable de filas, asegúrese de etiquetar al menos un formulario con la tabla más grande posible.
    > * Use los botones situados a la derecha de **+** para buscar, reordenar y eliminar las etiquetas, así como cambiarles el nombre.
    > * Para quitar una etiqueta aplicada sin eliminar la etiqueta en sí, seleccione el rectángulo etiquetado en la vista de documento y presione la tecla Supr.


---

:::image type="content" source="../media/label-tool/main-editor-2-1.png" alt-text="Ventana principal del editor de la herramienta de etiquetado de ejemplo.":::


Siga los pasos anteriores para etiquetar al menos cinco de sus formularios.

### <a name="specify-tag-value-types"></a>Especificación de los tipos de valores de etiqueta

Opcionalmente, puede establecer el tipo de datos esperado para cada etiqueta. Abra el menú contextual a la derecha de una etiqueta y seleccione un tipo del menú. Esta característica permite que el algoritmo de detección realice determinadas suposiciones que mejorarán la precisión de la detección de texto. También garantiza que los valores detectados se devuelvan en un formato estandarizado en la salida JSON final. 

> [!div class="mx-imgBorder"]
> ![Selección del tipo de valor con la herramienta de etiquetado de ejemplo](../media/whats-new/formre-value-type.png)

Actualmente se admiten los siguientes tipos de valor y variaciones:
* `string`
    * predeterminado, `no-whitespaces`, `alphanumeric`
* `number`
    * predeterminado, `currency`
* `date` 
    * predeterminado, `dmy`, `mdy`, `ymd`
* `time`
* `integer`
* `selectionMark`: _nuevo en v2.1-preview.1!_

> [!NOTE]
> Consulte estas reglas para el formato de fecha:
> 
> Los siguientes caracteres se pueden usar como delimitadores de fecha de DMA: `, - / . \`. No se puede usar un espacio en blanco como delimitador. Por ejemplo:
> * 01,01,2020
> * 01-01-2020
> * 01/01/2020
>
> El día y el mes se pueden escribir con uno o dos dígitos y el año puede tener dos o cuatro dígitos:
> * 1-1-2020
> * 1-01-20
>
> Si una cadena de fecha de DMA tiene ocho dígitos, el delimitador es opcional:
> * 01012020
> * 01 01 2020
>
> El mes también se puede escribir con su nombre completo o abreviado. Si se usa el nombre, los caracteres delimitadores son opcionales:
> * 01/Ene/2020
> * 01Ene2020
> * 01 de enero de 2020

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

Haga clic en el icono Train (Entrenar) en el panel izquierdo para abrir la página de entrenamiento. A continuación, haga clic en el botón **Entrenar** para empezar a entrenar el modelo. Una vez completado el proceso de entrenamiento, verá la siguiente información:

* **Id. del modelo**: el identificador del modelo que se ha creado y entrenado. Cada llamada de entrenamiento crea un nuevo modelo con su propio identificador. Copie esta cadena en una ubicación segura; la necesitará si desea realizar llamadas de predicción mediante la [API REST](./curl-train-extract.md) o la [biblioteca cliente](./client-library.md).
* **Precisión media**: el promedio de precisión del modelo. Puede mejorar la precisión del modelo mediante el etiquetado de formularios adicionales y el entrenamiento de nuevo para crear un nuevo modelo. Se recomienda empezar por etiquetar cinco formularios y agregar más formularios según sea necesario.
* La lista de etiquetas y la precisión estimada por etiqueta.


:::image type="content" source="../media/label-tool/train-screen.png" alt-text="Vista de entrenamiento.":::

Una vez finalizado el entrenamiento, examine el valor de **Precisión media**. Si es bajo, debe agregar más documentos de entrada y repetir los pasos anteriores. Los documentos que ya ha etiquetado permanecerán en el índice del proyecto.

> [!TIP]
> También puede ejecutar el proceso de entrenamiento con una llamada a la API REST. Para aprender a hacerlo, consulte [Entrenamiento con etiquetas mediante Python](./python-labeled-data.md).

## <a name="compose-trained-models"></a>Creación de modelos entrenados

# <a name="v20"></a>[v2.0](#tab/v2-0)  

Esta característica está disponible actualmente en la versión preliminar v2.1. 

# <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1) 

Con el modelo Compose, puede crear hasta 100 modelos en un único identificador de modelo. Cuando llame a Analyze con este identificador de modelo compuesto, Form Recognizer clasificará primero el formulario que envió, lo hará coincidir con el mejor modelo de coincidencia y, a continuación, devolverá los resultados de ese modelo. Esto resulta útil cuando los formularios de entrada pueden pertenecer a una de varias plantillas.

Para crear modelos en la herramienta de etiquetado de ejemplo, haga clic en el icono de composición de modelo (flecha de combinación) de la izquierda. A la izquierda, seleccione los modelos que desee unir. Los modelos con el icono de flechas ya son modelos compuestos. Haga clic en el botón "Compose" (Redactar). En el elemento emergente, asigne un nombre al nuevo modelo compuesto y haga clic en "Compose" (Redactar). Una vez finalizada la operación, el nuevo modelo compuesto debe aparecer en la lista. 

:::image type="content" source="../media/label-tool/model-compose.png" alt-text="Vista de experiencia de usuario para la creación de modelos.":::

---

## <a name="analyze-a-form"></a>Análisis de un formulario 

Haga clic en el icono Predict (Predecir) (bombilla) de la izquierda para probar el modelo. Cargue un documento de formulario que no haya utilizado en el proceso de entrenamiento. A continuación, haga clic en el botón **Predecir** de la derecha para obtener las predicciones de clave y valor del formulario. La herramienta aplicará etiquetas en los cuadros de límite e informará de la confianza de cada etiqueta.

> [!TIP]
> También puede ejecutar la API de análisis con una llamada a REST. Para aprender a hacerlo, consulte [Entrenamiento con etiquetas mediante Python](./python-labeled-data.md).

## <a name="improve-results"></a>Mejora de los resultados

Según la precisión notificada, puede que desee realizar un entrenamiento adicional para mejorar el modelo. Después de haber realizado una predicción, examine los valores de confianza de cada una de las etiquetas aplicadas. Si el valor de precisión media del entrenamiento es alto, pero las puntuaciones de confianza son bajas (o los resultados son inexactos), debe agregar el archivo que se utilizó para la predicción al conjunto de entrenamiento, etiquetarlo y realizar el entrenamiento de nuevo.

La precisión media notificada, las puntuaciones de confianza y la precisión real pueden ser incoherentes cuando los documentos que se analizan son diferentes de los usados en el entrenamiento. Tenga en cuenta que algunos documentos tienen un aspecto similar cuando los ven los usuarios, pero pueden parecer distintos para el modelo de IA. Por ejemplo, puede realizar el entrenamiento con un tipo de formulario que tiene dos variantes, en el que el conjunto de entrenamiento consta de un 20 % perteneciente a la variante A y un 80 % a la variante B. Durante la predicción, es probable que las puntuaciones de confianza de los documentos de la variante A sean menores.

## <a name="save-a-project-and-resume-later"></a>Guardar un proyecto y reanudarlo más tarde

Para reanudar el proyecto en otro momento o en otro explorador, debe guardar el token de seguridad del proyecto y volver a escribirlo más tarde. 

### <a name="get-project-credentials"></a>Obtención de las credenciales del proyecto
Vaya a la página de configuración del proyecto (icono de control deslizante) y anote el nombre del token de seguridad. A continuación, vaya a la configuración de la aplicación (icono de engranaje), que muestra todos los tokens de seguridad de la instancia actual del explorador. Busque el token de seguridad del proyecto y copie su nombre y valor de clave en una ubicación segura.

### <a name="restore-project-credentials"></a>Restauración de las credenciales del proyecto
Si desea reanudar el proyecto, primero debe crear una conexión al mismo contenedor de almacenamiento de blobs. Para ello, repita los pasos anteriores. A continuación, vaya a la página de configuración de la aplicación (icono de engranaje) y compruebe si el token de seguridad del proyecto está allí. Si no es así, agregue un nuevo token de seguridad y copie el nombre y la clave del token del paso anterior. Después, haga clic en Guardar configuración. 

### <a name="resume-a-project"></a>Reanudación de un proyecto

Por último, vaya a la página principal (icono de casa) y haga clic en Open Cloud Project (Abrir proyecto en la nube). Después, seleccione la conexión al almacenamiento de blobs y seleccione el archivo *.fott* del proyecto. La aplicación cargará todos los valores del proyecto ya que tiene el token de seguridad.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a utilizar la herramienta de etiquetado de ejemplo de Form Recognizer para entrenar un modelo con datos etiquetados manualmente. Si desea integrar la herramienta de etiquetado en su propia aplicación, use las API REST que se ocupan del entrenamiento de datos etiquetados.

> [!div class="nextstepaction"]
> [Entrenamiento con etiquetas mediante Python](./python-labeled-data.md)
