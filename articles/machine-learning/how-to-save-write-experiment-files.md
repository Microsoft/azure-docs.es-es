---
title: Dónde se guardan y escriben los archivos de experimentos
titleSuffix: Azure Machine Learning
description: Aprenda dónde se guardan los archivos de entrada de los experimentos y dónde se escriben los archivos de salida para evitar errores de limitación del almacenamiento y la latencia de los experimentos.
services: machine-learning
author: rastala
ms.author: roastala
manager: danielsc
ms.reviewer: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.date: 03/10/2020
ms.openlocfilehash: 27d56958120d0eddebe30dc410805909fe507f7c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319582"
---
# <a name="where-to-save-and-write-files-for-azure-machine-learning-experiments"></a>Dónde guardar y escribir los archivos para los experimentos de Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, obtendrá información sobre dónde guardar los archivos de entrada y dónde escribir los archivos de salida de los experimentos para evitar errores de limitación del almacenamiento y la latencia de los experimentos.

Al iniciar las ejecuciones de entrenamiento en un [destino de proceso](how-to-set-up-training-targets.md), están aisladas de los entornos exteriores. El propósito de este diseño es garantizar la reproducibilidad y la portabilidad del experimento. Si ejecuta dos veces el mismo script, en el mismo destino de proceso o en otro, recibirá los mismos resultados. Con este diseño, puede tratar los destinos de proceso como recursos de cálculo sin estado y ninguno de ellos tendrá afinidad con los trabajos que se ejecutan una vez finalizados.

## <a name="where-to-save-input-files"></a>Dónde guardar los archivos de entrada

Para poder iniciar un experimento en la máquina local o en un destino de proceso, debe asegurarse de que los archivos necesarios están disponibles para ese destino de proceso, por ejemplo, los archivos de dependencia y los de datos que su código necesita para ejecutarse.

Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [almacén de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.data?view=azure-ml-py).

El límite de almacenamiento para las instantáneas del experimento es de 300 MB y 2000 archivos.

Por este motivo, se recomienda:

* **Almacenar los archivos en un [almacén de datos](https://docs.microsoft.com/python/api/azureml-core/azureml.data?view=azure-ml-py) de Azure Machine Learning.** Así se evitan los problemas de latencia del experimento y tiene las ventajas de acceder a los datos desde un destino de proceso remoto, lo que significa que la autenticación y el montaje son administrados por Azure Machine Learning. Más información sobre cómo especificar un almacén de datos como el directorio de origen y cómo cargar los archivos en el almacén de datos, en el artículo [Datos de acceso desde almacenes de datos](how-to-access-data.md).

* **Si solo necesita dos archivos de datos y scripts de dependencias, y no puede usar un almacén de datos,** coloque los archivos en el mismo directorio de carpetas que el script de entrenamiento. Especifique esta carpeta como su `source_directory` directamente en el script de entrenamiento o en el código que lo llama.

<a name="limits"></a>

### <a name="storage-limits-of-experiment-snapshots"></a>Límites de almacenamiento de las instantáneas del experimento

Para los experimentos, Azure Machine Learning realiza automáticamente una instantánea del experimento de su código basada en el directorio que se sugiera al configurar la ejecución. El límite total es de 300 MB o de 2000 archivos. Si se supera, verá el siguiente error:

```Python
While attempting to take snapshot of .
Your total snapshot size exceeds the limit of 300.0 MB
```

Para resolver este error, almacene los archivos del experimento en un almacén de datos. Si no puede usar un almacén de datos, la tabla siguiente ofrece algunas soluciones alternativas posibles.

Experimento &nbsp; descripción|Solución de límite de almacenamiento
---|---
Menos de 2000 archivos y no se puede usar un almacén de datos| Invalide el límite de tamaño de instantánea con <br> `azureml._restclient.snapshots_client.SNAPSHOT_MAX_SIZE_BYTES = 'insert_desired_size'`<br> Puede tardar varios minutos según el número y el tamaño de los archivos.
Debe usar el directorio de script específico| [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
Canalización|Use un subdirectorio diferente en cada paso
Jupyter Notebook| Cree un archivo `.amlignore` o mueva el Bloc de notas a un subdirectorio nuevo y vacío, y vuelva a ejecutar el código.

## <a name="where-to-write-files"></a>Dónde escribir los archivos

Debido al aislamiento de los experimentos de entrenamiento, los cambios en los archivos que se producen durante las ejecuciones no necesariamente persisten fuera de su entorno. Si el script modifica los archivos de forma local al proceso, los cambios no se conservan para la siguiente ejecución del experimento ni se propagan a la máquina cliente automáticamente. Por tanto, los cambios realizados durante la primera ejecución del experimento no afectan a los realizados durante la segunda, ni deberían afectar.

Al escribir los cambios, se recomienda escribir los archivos en un almacén de datos de Azure Machine Learning. Consulte [Datos de acceso desde almacenes de datos](how-to-access-data.md).

Si no necesita un almacén de datos, escriba los archivos en la carpeta `./outputs` o `./logs`.

>[!Important]
> Dos carpetas, *outputs* y *logs*, reciben un tratamiento especial por parte de Azure Machine Learning. Durante el entrenamiento, si escribe archivos en las carpetas llamadas `./outputs` y `./logs`, los archivos se cargarán automáticamente en su historial de ejecución, por lo que tendrá acceso a ellos una vez completada su ejecución.

* **Si la salida son los mensajes de estado o los resultados de puntuación,** escriba los archivos en la carpeta `./outputs`, de modo que se conserven como artefactos en el historial de ejecución. Esté atento al número y tamaño de los archivos escritos en esta carpeta, ya que puede producirse cierta latencia cuando el contenido se cargue en el historial de ejecución. Si la latencia es un problema, se recomienda escribir los archivos en un almacén de datos.

* **Para guardar el archivo escrito como registros en el historial de ejecución** escriba los archivos en la carpeta `./logs`. Los registros se cargan en tiempo real, por lo que este método es adecuado para el streaming de las actualizaciones directas desde una ejecución remota.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [acceso a los datos desde almacenes de datos](how-to-access-data.md).

* Más información sobre [cómo configurar los destinos de entrenamiento](how-to-set-up-training-targets.md).
