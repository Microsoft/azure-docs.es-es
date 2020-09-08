---
title: 'Tutorial: Creación de una aplicación de procesamiento de formularios con AI Builder: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: En este tutorial, usará AI Builder para crear y entrenar una aplicación de procesamiento de formularios.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: tutorial
ms.date: 07/01/2020
ms.author: pafarley
ms.openlocfilehash: 981c6f6bb2b0eb597b32ce8e428ef0aa7d19929b
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88003358"
---
# <a name="tutorial-create-a-form-processing-app-with-ai-builder"></a>Tutorial: Creación de una aplicación de procesamiento de formularios con AI Builder

[AI Builder](https://docs.microsoft.com/ai-builder/overview) es una funcionalidad de Power Platform que permite automatizar los procesos y predecir los resultados para mejorar el rendimiento empresarial. Puede usar el procesamiento de formularios de AI Builder para crear modelos de inteligencia artificial que identifiquen y extraigan pares clave-valor y datos de tabla de documentos de formulario.

> [!NOTE]
> Este proyecto también está disponible como [módulo de Microsoft Learn](https://docs.microsoft.com/learn/modules/get-started-with-form-processing/).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un modelo de inteligencia artificial de procesamiento de formularios
> * Entrenamiento de un modelo
> * Publicar el modelo para usarlo en Azure Power Apps o Power Automate

## <a name="prerequisites"></a>Requisitos previos

* Un conjunto de al menos cinco formularios del mismo tipo para entrenar y probar los datos. Consulte [Creación de un conjunto de datos de aprendizaje](./build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Entrenar** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2128080).
* Una licencia de Power Apps o Power Automate (consulte la [guía de licencia](https://go.microsoft.com/fwlink/?linkid=2085130)). La licencia debe incluir [Common Data Service](https://powerplatform.microsoft.com/en-us/common-data-service/).
* Un [complemento o una evaluación gratuita](https://go.microsoft.com/fwlink/?LinkId=2113956&clcid=0x409) de AI Builder.


## <a name="create-a-form-processing-project"></a>Creación de un proyecto de procesamiento de formularios

1. Vaya a [Power Apps](https://make.powerapps.com/) o [Power Automate](https://flow.microsoft.com/signin) e inicie sesión con la cuenta de la organización.
1. En el panel izquierdo, seleccione **AI Builder** > **Compilar**.
1. Seleccione la tarjeta **Procesamiento de formularios**.
1. Escriba un nombre para el modelo.
1. Seleccione **Crear**.

## <a name="upload-and-analyze-documents"></a>Carga y análisis de documentos

En la página **Agregar documentos**, debe proporcionar documentos de ejemplo para entrenar el modelo según el tipo de formulario del que quiera extraer información. Después de cargar los documentos, AI Builder los analiza para comprobar que son suficientes para entrenar un modelo.

> [!NOTE]
> AI Builder no admite actualmente los siguientes tipos de datos de entrada de procesamiento de formularios:
>
> - Tablas complejas (tablas anidadas, celdas o encabezados combinados, etc.)
> - Casillas o botones de radio
> - Documentos PDF de más de 50 páginas
> - PDF rellenables
>
> Para más información sobre los requisitos de los documentos de entrada, consulte los [requisitos de entrada](./overview.md#input-requirements).

### <a name="upload-your-documents"></a>Carga de los documentos

1. Seleccione **Agregar documentos**, seleccione cinco documentos como mínimo y, luego, elija **Cargar**.
1. Una vez finalizada la carga, seleccione **Cerrar**.
1. Luego, seleccione **Analizar**.

> [!NOTE] 
> Después de cargar estos documentos, puede quitar algunos de los documentos o cargar otros adicionales.

> [!div class="mx-imgBorder"]
> ![página Agregar documentos](./media/tutorial-ai-builder/add-documents-page.png)

### <a name="analyze-your-documents"></a>Análisis de los documentos

Durante el paso de análisis, AI Builder examina los documentos cargados y detecta los campos y las tablas. El tiempo necesario para completar esta operación depende del número de documentos proporcionados, pero en la mayoría de los casos será solo de unos minutos.

Cuando el análisis haya finalizado, seleccione la miniatura para abrir la experiencia de selección de campos.

> [!IMPORTANT]
> Si se produce un error en el análisis, es probable que AI Builder no pueda detectar texto estructurado en los documentos. Compruebe que los documentos que ha actualizado siguen los [requisitos de entrada](./overview.md#input-requirements).

## <a name="select-your-form-fields"></a>Selección de los campos de formulario

En la página de selección de campos, elija los campos que le interesan:

1. Para seleccionar un campo, haga clic en un rectángulo que indique un campo detectado en el documento o haga clic y arrastre para seleccionar varios campos. También puede seleccionar campos en el panel derecho.
1. Si quiere cambiar un nombre de campo para que se ajuste a sus necesidades o normalizar las etiquetas extraídas, haga clic en el nombre del campo seleccionado.

    Al hacer clic en un campo detectado, aparece la siguiente información:

    - **Nombre del campo**: nombre de la etiqueta del campo detectado.
    - **Valor de campo**: valor del campo detectado.

> [!div class="mx-imgBorder"]
> ![página Agregar documentos](./media/tutorial-ai-builder/select-fields-page.png)

### <a name="label-undetected-fields"></a>Etiquetado de campos no detectados

Si el modelo no ha detectado automáticamente el campo que quiere etiquetar, puede dibujar un rectángulo alrededor de su valor y escribir un nombre de etiqueta en el cuadro de diálogo que aparece.

## <a name="train-your-model"></a>Entrenamiento de un modelo

1. Seleccione **Siguiente** para comprobar los campos de formulario seleccionados. Si todo está bien, seleccione **Entrenar** para entrenar el modelo.

    > [!div class="mx-imgBorder"]
    > ![página Agregar documentos](./media/tutorial-ai-builder/summary-train-page.png)
1. Cuando finalice el entrenamiento, seleccione **Ir a la página Detalles** en la pantalla **Entrenamiento finalizado**.
## <a name="quick-test-your-model"></a>Prueba rápida del modelo

La página Detalles le permite probar el modelo antes de publicarlo o usarlo:

1. En la página Detalles, seleccione **Prueba rápida**.
2. Puede arrastrar y colocar un documento o seleccionar **Cargar desde mi dispositivo** para cargar el archivo de prueba. La prueba rápida tardará solo unos segundos en mostrar los resultados.
3. Puede seleccionar **Empezar de nuevo** para ejecutar otra prueba, o **Cerrar** si ha finalizado.

### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

Si obtiene resultados incorrectos o puntuaciones de confianza baja en determinados campos, pruebe las siguientes sugerencias:

- Vuelva a realizar el entrenamiento usando formularios con distintos valores en cada campo.
- Vuelva a realizar el entrenamiento con un conjunto mayor de documentos de entrenamiento. Cuantos más documentos etiquete, mejor aprenderá AI Builder a reconocer los campos.
- Puede optimizar los archivos PDF seleccionando solo determinadas páginas con las que realizar el entrenamiento. Use la opción **Imprimir** > **Print to PDF** (Imprimir en PDF) para seleccionar determinadas páginas del documento.

## <a name="publish-your-model"></a>Publicación del modelo

Si está satisfecho con el modelo, seleccione **Publicar** para publicarlo. Cuando finalice la publicación, el estado del modelo cambiará a **Publicado** y estará listo para usarse.

> [!div class="mx-imgBorder"]
> ![página Agregar documentos](./media/tutorial-ai-builder/model-page.png)

Después de haber publicado el modelo de procesamiento de formularios, puede usarlo en una [aplicación de lienzo de Power Apps](https://docs.microsoft.com/ai-builder/form-processor-component-in-powerapps) o en [Power Automate](https://docs.microsoft.com/ai-builder/form-processing-model-in-flow).

## <a name="next-steps"></a>Pasos siguientes

Siga la documentación de AI Builder para usar un modelo de procesamiento de formularios.

* [Usar el componente del procesador de formularios en Power Apps](https://docs.microsoft.com/ai-builder/form-processor-component-in-powerapps)
* [Usar un modelo de procesamiento de formularios en Power Automate](https://docs.microsoft.com/ai-builder/form-processing-model-in-flow)