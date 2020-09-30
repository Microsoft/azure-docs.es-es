---
title: Conversión de modelos
description: Describe el proceso de conversión de un modelo para su representación
author: jakrams
ms.author: jakras
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: 49d3fd953d069f4368d28e26265114e574e8100a
ms.sourcegitcommit: 655e4b75fa6d7881a0a410679ec25c77de196ea3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89506668"
---
# <a name="convert-models"></a>Conversión de modelos

Azure Remote Rendering le permite representar modelos muy complejos. Para lograr el máximo rendimiento, los datos deben preprocesarse para tener un formato óptimo. En función de la cantidad de datos, este paso puede tardar unos minutos. No sería práctico que este tiempo se gastara durante la carga de modelos. Asimismo, sería una pérdida de tiempo repetir este proceso para varias sesiones. Por estos motivos, el servicio ARR proporciona un *servicio de conversión* dedicado, que puede ejecutar de antemano.
Una vez convertido, se puede cargar un modelo desde una cuenta de Azure Storage.

## <a name="supported-source-formats"></a>Formatos de origen admitidos

El servicio de conversión admite estos formatos:

- **FBX**  (versión 2011 y posteriores)
- **GLTF** (versión 2.x)
- **GLB**  (versión 2.x)

Existen pequeñas diferencias entre los formatos con respecto a la conversión de propiedades de material, como se muestra en el capítulo [Asignación de materiales para formatos de modelos](../../reference/material-mapping.md).

## <a name="the-conversion-process"></a>El proceso de conversión

1. [Prepare dos contenedores de Azure Blob Storage](blob-storage.md): uno para la entrada y el otro para la salida.
1. Cargue el modelo en el contenedor de entrada (opcionalmente en un subtrazado).
1. Desencadene el proceso de conversión a través de [la API de REST de conversión de modelos](conversion-rest-api.md).
1. Sondee el servicio para el progreso de la conversión.
1. Una vez finalizado, cargue un modelo
    - desde una cuenta de almacenamiento vinculada (consulte los pasos de "Vinculación de cuentas de almacenamiento" en [Crear una cuenta](../create-an-account.md#link-storage-accounts) para vincular su cuenta de almacenamiento)
    - o proporcionando una *firma de acceso compartido (SAS)* .

Todos los datos del modelo (entrada y salida) se almacenan en la instancia de Azure Blob Storage proporcionada por el usuario. Azure Remote Rendering proporciona un control total sobre la administración de recursos.

## <a name="pricing"></a>Precios

Para obtener información sobre los precios de conversión, consulte la página de [Precios de Remote Rendering](https://azure.microsoft.com/pricing/details/remote-rendering).


## <a name="conversion-parameters"></a>Parámetros de conversión

Para ver las diferentes opciones de conversión, consulte [este capítulo](configure-model-conversion.md).

## <a name="examples"></a>Ejemplos

- [Inicio rápido: Conversión de un modelo para su representación](../../quickstarts/convert-model.md) es una introducción paso a paso de cómo convertir un modelo.
- Encontrará [scripts de PowerShell de ejemplo](../../samples/powershell-example-scripts.md), que muestran el uso del servicio de conversión, en el [repositorio de ejemplos de ARR](https://github.com/Azure/azure-remote-rendering) en la carpeta *Scripts*.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de Azure Blob Storage para la conversión de modelos](blob-storage.md)
- [La API de REST de conversión de modelos](conversion-rest-api.md)
- [Configuración de la conversión de modelos](configure-model-conversion.md)
- [Diseño de archivos para la conversión](layout-files-for-conversion.md)
- [Asignación de materiales para formatos de modelos](../../reference/material-mapping.md)
