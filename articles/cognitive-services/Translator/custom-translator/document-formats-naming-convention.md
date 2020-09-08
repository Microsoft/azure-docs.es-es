---
title: 'Formatos y convenciones de nomenclatura para documentos: Custom Translator'
titleSuffix: Azure Cognitive Services
description: A continuación se presenta una guía sobre los formatos y convenciones de nomenclatura de documentos en Custom Translator. Este concepto ayuda a administrar mejor los nombres de los documentos y a evitar los conflictos relacionados con la nomenclatura.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 46a19cd4396e598a74f9099727ac80d44c0a44f5
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88507253"
---
# <a name="document-formats-and-naming-convention-guidance"></a>Guía sobre formatos y convenciones de nomenclatura para documentos

Cualquier archivo que se utilice para la traducción personalizada debe tener al menos **cuatro** caracteres de longitud.

Esta tabla incluye todos los formatos de archivo compatibles que puede usar para crear el sistema de traducción:

| Formato            | Extensiones   | Descripción                                                                                                                                                                                                                                                                    |
|-------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| XLIFF             | .XLF, .XLIFF | Formato de documento paralelo que se exporta desde los sistemas de memoria de traducción. Los idiomas utilizados se definen dentro del archivo.                                                                                                                                                              |
| TMX               | .TMX         | Formato de documento paralelo que se exporta desde los sistemas de memoria de traducción. Los idiomas utilizados se definen dentro del archivo.                                                                                                                                                              |
| ZIP               | .ZIP         | ZIP es un formato de archivo.                                                                                                                                                                                                        |
| Locstudio         | .LCL         | Formato de Microsoft para los documentos paralelos                                                                                                                                                                                                                                      |
| Microsoft Word    | .DOCX        | Documento de Microsoft Word                                                                                                                                                                                                                                                        |
| Adobe Acrobat     | .PDF         | Documento portátil de Adobe Acrobat                                                                                                                                                                                                                                                |
| HTML              | .HTML, .HTM  | Documento HTML                                                                                                                                                                                                                                                                  |
| Archivo de texto         | .TXT         | Archivos de texto codificados en UTF-16 o UTF-8. El nombre de archivo no debe contener caracteres japoneses.                                                                                                                                                                                        |
| Archivo de texto alineado | .ALIGN       | La extensión `.ALIGN` es una extensión especial que puede usar si sabe que las oraciones del par de documentos están perfectamente alineadas. Si proporciona un archivo `.ALIGN`, Custom Translator no alineará las oraciones por usted. |
| Archivo de Excel        | .XLSX        | Archivo de Excel (2013 o posterior). La primera línea o fila de la hoja de cálculo debe ser el código de idioma.                                                                                                                                                                                                                                                      |

## <a name="dictionary-formats"></a>Formatos de diccionario

Para los diccionarios, Traductor personalizado es compatible con todos los formatos de archivo que se admiten para los conjuntos de entrenamiento. Si está utilizando un diccionario de Excel, la primera línea o fila de la hoja de cálculo debe contener códigos de idioma.

## <a name="zip-file-formats"></a>Formatos de archivo ZIP

Los documentos se pueden agrupar en un único archivo ZIP y cargarse. Custom Translator es compatible con los formatos de archivo ZIP (ZIP, GZ y TGZ).

Todos los documentos del archivo zip con la extensión TXT, HTM, HTML, PDF, DOCX o ALIGN deben seguir esta convención de nomenclatura:

{nombre del documento}\_{código de idioma} donde {nombre del documento} es el nombre del documento y {código de idioma} es el identificador de idioma ISO (dos caracteres), que indica que el documento contiene oraciones en ese idioma. Debe haber un guion bajo (_) antes del código de idioma.

Por ejemplo, para cargar dos documentos paralelos dentro de un archivo ZIP para un sistema de inglés a español, los archivos deben tener los nombres "data_en" y "data_es".

No es necesario que los archivos de la memoria de traducción (TMX, XLF, XLIFF, LCL y XLSX) sigan la convención de nomenclatura de un idioma específico.  

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre los [proyectos](workspace-and-project.md#what-is-a-custom-translator-project) para crearlos y administrarlos.
