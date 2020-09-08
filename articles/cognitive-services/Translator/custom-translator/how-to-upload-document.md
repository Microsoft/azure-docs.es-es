---
title: 'Cómo cargar un documento: Custom Translator'
titleSuffix: Azure Cognitive Services
description: La característica de carga de documentos carga documentos en paralelo (dos documentos donde uno es el origen y el otro es la traducción) en el servicio.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 8597b3e4ffab44672e88374010fa829211fbc18b
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88507336"
---
# <a name="upload-a-document"></a>Cargar un documento

En [Custom Translator](https://portal.customtranslator.azure.ai), puede cargar documentos paralelos para entrenar los modelos de traducción. Los [documentos paralelos](what-are-parallel-documents.md) son pares de documentos en los que uno es la traducción del otro. Un documento en el par contiene frases en el idioma de origen y el otro documento contiene estas mismas frases traducidas al idioma de destino.

Antes de cargar los documentos, revise la [Guía sobre formatos y convenciones de nomenclatura para documentos](document-formats-naming-convention.md) para asegurarse de que el formato de archivo es compatible con Custom Translator.

## <a name="how-to-upload-document"></a>¿Cómo se carga un documento?

Desde el portal de [Custom Translator](https://portal.customtranslator.azure.ai), haga clic en la pestaña "Documentos" para ir a la página de documentos.

![Vínculo de carga de documento](media/how-to/how-to-upload-1.png)


1.  Haga clic en el botón Cargar archivos en la página de documentos.

    ![Página de carga de documentos](media/how-to/how-to-upload-2.png)

2.  En el cuadro de diálogo, rellene la información siguiente:

    a.  Tipo de documento:

    -  Aprendizaje: estos documentos se usarán para el conjunto de aprendizaje.
    -  Optimización: estos documentos se usarán para el conjunto de optimización.
    -  Prueba: estos documentos se usarán para el conjunto de pruebas.
    -  Diccionario de frases: estos documentos se usarán para el diccionario de frases.
    -  Diccionario de oraciones: estos documentos se usarán para el diccionario de oraciones.

    b.  Par de idiomas

    c.  Override document if exists (Reemplazar el documento si existe): active esta casilla si quiere sobrescribir los documentos existentes con el mismo nombre.

    d.  Rellene la sección correspondiente para los datos en paralelo o los datos combinados.

    -  Datos en paralelo:
        -  Archivo de origen: seleccione el archivo en el idioma de origen del equipo local.
        -  Archivo de destino: seleccione el archivo en el idioma de destino del equipo local.
        -  Nombre del documento: solo se usa si está cargando archivos paralelos.

    - Datos combinados:
        -  Archivo combinado: seleccione el archivo combinado del equipo local. El archivo combinado tiene las oraciones tanto en el idioma de origen como en el de destino. La [convención de nomenclatura](document-formats-naming-convention.md) es importante para los archivos combinados.

    e.  Haga clic en Cargar

    ![Cuadro de diálogo Cargar documento](media/how-to/how-to-upload-dialog.png)

3.  En este momento, estamos procesando sus documentos e intentando extraer las oraciones. Puede hacer clic en "Ver progreso de carga" para comprobar el estado de los documentos conforme se procesan.

    ![Cuadro de diálogo del proceso de carga de documentos](media/how-to/how-to-upload-processing-dialog.png)

4.  En esta página se mostrarán el estado y los errores de cada archivo durante la carga. Puede ver los estado de carga anteriores en cualquier momento si hace clic en la pestaña "Historial de carga".

    ![Cuadro de diálogo del historial de carga de documentos](media/how-to/how-to-upload-document-history.png)


## <a name="view-upload-history"></a>Ver el historial de carga

En la página del historial de carga puede ver los detalles de todas las cargas de documentos en el historial, como el tipo de documento, el par de idiomas, el estado de carga, etc.

1. Desde el portal de [Custom Translator](https://portal.customtranslator.azure.ai), haga clic en la pestaña del historial de carga para ver el historial.

    ![Pestaña del historial de carga](media/how-to/how-to-upload-history-1.png)

2. En esta página se muestra el estado de todas las cargas anteriores. Se muestran las cargas desde la más reciente hasta la más antigua. Para cada carga, se muestra el nombre del documento, el estado de la carga, la fecha de carga, el número de archivos cargados, el tipo de los archivos cargados y el par de idiomas del archivo.

    ![Página del historial de carga](media/how-to/how-to-document-history-2.png)

3. Haga clic en cualquier registro del historial de carga. En la página de detalles del historial de carga puede ver los archivos cargados como parte del proceso de carga, el estado de carga de los archivos, el idioma del archivo y los mensajes de error (si hay algún error en la carga).

## <a name="next-steps"></a>Pasos siguientes

- Use la [página de detalles del documento](how-to-view-document-details.md) para revisar la lista de las oraciones extraídas.
- [Cómo entrenar un modelo](how-to-train-model.md).
