---
title: 'ML Studio (clásico): Ejecución de scripts de Python: Azure'
description: Obtenga información sobre cómo usar el módulo Ejecutar script de Python para usar código Python en servicios web y experimentos de Machine Learning Studio (clásico).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: devx-track-python, previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 03/12/2019
ms.openlocfilehash: d2f2e01bda22c348d53ce2a1dd25d8d11e86ff3e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87843545"
---
# <a name="execute-python-machine-learning-scripts-in-azure-machine-learning-studio-classic"></a>Ejecución de scripts de Python en Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Python es una valiosa herramienta de la caja de herramientas de muchos científicos de datos. Se utiliza en cada etapa de flujos de trabajo típicos de aprendizaje automático, incluida la exploración de datos, la extracción de características, el entrenamiento y la validación del modelo, y la implementación.

En este artículo se describe cómo puede usar el módulo Ejecutar script de Python para usar código Python en los experimentos de Azure Machine Learning Studio (clásico) y servicios web.

## <a name="using-the-execute-python-script-module"></a>Uso del módulo Ejecutar script de Python

La interfaz principal para Python en Studio (clásico) se presenta a través del módulo [Ejecutar script de Python][execute-python-script]. Acepta hasta tres entradas y genera hasta dos salidas, y es similar al módulo [Ejecutar script R][execute-r-script]. El código Python se especifica en el cuadro de parámetro mediante una función de punto de entrada especialmente denominada llamada `azureml_main`.

![Módulo Ejecutar script de Python](./media/execute-python-scripts/execute-machine-learning-python-scripts-module.png)

![Código Python de ejemplo en el cuadro de parámetro del módulo](./media/execute-python-scripts/embedded-machine-learning-python-script.png)

### <a name="input-parameters"></a>Parámetros de entrada

Las entradas en el módulo Python se exponen como DataFrames de Pandas. La función `azureml_main` acepta hasta dos DataFrames de Pandas opcionales como parámetros.

La asignación entre los puertos de entrada y los parámetros de función es posicional:

- El primer puerto de entrada conectado se asigna al primer parámetro de la función.
- La segunda entrada (si está conectada) se asigna al segundo parámetro de la función.
- La tercera entrada se utiliza para [importar módulos de Python adicionales](#import-modules).

Puede ver una semántica más detallada de cómo se asignan los puertos de entrada a los parámetros de la función `azureml_main`.

![Tabla de configuraciones de puertos de entrada y firma resultante de Python](./media/execute-python-scripts/python-script-inputs-mapped-to-parameters.png)

### <a name="output-return-values"></a>Valores devueltos de salida

La función `azureml_main` debe devolver un solo DataFrame de Pandas empaquetado dentro de una [secuencia](https://docs.python.org/2/c-api/sequence.html) de Python, como una tupla, una lista o una matriz de NumPy. El primer elemento de esta secuencia se devuelve al primer puerto de salida del módulo. El segundo puerto de salida del módulo se usa para [visualizaciones](#visualizations) y no requiere un valor devuelto. Este esquema se muestra a continuación.

![Asignación de puertos de entrada a parámetros y valor devuelto a puerto de salida](./media/execute-python-scripts/map-of-python-script-inputs-outputs.png)

## <a name="translation-of-input-and-output-data-types"></a>Conversión de tipos de datos de entrada y salida

Los conjuntos de datos de Studio no son lo mismo que los DataFrames de Pandas. En consecuencia, los conjuntos de datos de entrada de Studio (clásico) se convierten en DataFrames de Pandas y los DataFrames de salida se convierten en conjuntos de datos de Studio (clásico). Durante este proceso de conversión, también se realizan las siguientes conversiones:

 **Tipo de datos de Python** | **Procedimiento de conversión de Studio** |
| --- | --- |
| Cadenas y valores numéricos| Se dejan como están |
| Valores no disponibles 'NA' de Pandas | Se convierten a 'Falta el valor' |
| Vectores de índice | No admitidos* |
| Nombres de columna que no son cadenas | Llamada `str` en nombres de columna |
| Nombres de columna duplicados | Agregar sufijo numérico: (1), (2), (3), etc.

**Todas las tramas de datos de entrada de la función de Python siempre tienen un índice numérico de 64 bits entre 0 y el número de filas menos 1*

## <a name="importing-existing-python-script-modules"></a><a id="import-modules"></a>Importación de módulos de scripts existentes de Python

El back-end que se usa para ejecutar Python se basa en [Anaconda](https://www.anaconda.com/distribution/), una distribución científica de Python ampliamente usada. Incluye cerca de 200 de los paquetes más comunes de Python que se utilizan en las cargas de trabajo centradas en datos. Studio (clásico) no admite actualmente el uso de sistemas de administración de paquetes, como Pip o Conda, para instalar y administrar bibliotecas externas.  Si encuentra la necesidad de incorporar bibliotecas adicionales, use el siguiente escenario como guía.

Un caso de uso común es incorporar scripts existentes de Python a experimentos de Studio (clásico). El modulo [Ejecutar script de Python][execute-python-script] acepta un archivo ZIP que contiene los módulos de Python en el tercer puerto de entrada. El marco de ejecución descomprime el archivo en runtime y el contenido se agrega a la ruta de acceso de la biblioteca del intérprete de Python. La función de punto de entrada `azureml_main` luego puede importar directamente estos módulos. 

Por ejemplo, considere el archivo Hello.py que contiene una función "Hello, World".

![Función definida por el usuario en el archivo Hello.py.](./media/execute-python-scripts/figure4.png)

A continuación, creamos un archivo Hello.zip que contiene Hello.py:

![Archivo ZIP que contiene código Python definido por el usuario.](./media/execute-python-scripts/figure5.png)

Cargue el archivo ZIP como un conjunto de datos en Studio (clásico). Luego cree y ejecute un experimento que use el código Python del archivo Hello.zip al asociarlo al tercer puerto de entrada del módulo **Ejecutar script de Python**, como se muestra en la siguiente imagen.

![Experimento de ejemplo con Hello.zip como entrada a un módulo Ejecutar script de Python](./media/execute-python-scripts/figure6a.png)

![Código Python definido por el usuario cargado como archivo ZIP](./media/execute-python-scripts/figure6b.png)

La salida del módulo muestra que el archivo ZIP se ha desempaquetado y que la función `print_hello` se ha ejecutado.

![Salida del módulo que muestra una función definida por el usuario](./media/execute-python-scripts/figure7.png)

## <a name="accessing-azure-storage-blobs"></a>Acceso a blobs de Azure Storage

Puede tener acceso a datos almacenados en una cuenta de Azure Blob Storage siguiendo estos pasos:

1. Descargue el [paquete de Azure Blob Storage para Python](https://azuremlpackagesupport.blob.core.windows.net/python/azure.zip) localmente.
1. Cargue el archivo ZIP en el área de trabajo de Studio (clásico) como un conjunto de datos.
1. Cree el objeto BlobService con `protocol='http'`

```
from azure.storage.blob import BlockBlobService

# Create the BlockBlockService that is used to call the Blob service for the storage account
block_blob_service = BlockBlobService(account_name='account_name', account_key='account_key', protocol='http')
```

1. Deshabilite **Se requiere transferencia segura** en la pestaña **Configuración** de Storage.

![Deshabilite Se requiere transferencia segura en Azure Portal](./media/execute-python-scripts/disable-secure-transfer-required.png)

## <a name="operationalizing-python-scripts"></a>Funcionamiento de los scripts de Python

Cuando un experimento de puntuación se publica como servicio web, se llama a todos los módulos [Ejecutar script de Python][execute-python-script] usados en él. Por ejemplo, la imagen siguiente muestra un experimento de puntuación que contiene el código para evaluar una única expresión de Python.

![Área de trabajo de Studio para un servicio web](./media/execute-python-scripts/figure3a.png)

![Expresión de Pandas de Python](./media/execute-python-scripts/python-script-with-python-pandas.png)

Un servicio web creado a partir de este experimento tendría las siguientes acciones:

1. Tomar como entrada una expresión de Python (como una cadena).
1. Enviar la expresión de Python para el intérprete de Python.
1. Devuelve una tabla que contiene la expresión y el resultado evaluado.

## <a name="working-with-visualizations"></a><a id="visualizations"></a>Trabajo con visualizaciones

El módulo [Ejecutar script de Python][execute-python-script] puede devolver los trazados creados con MatplotLib. Sin embargo, los trazados no se redirigen automáticamente a imágenes como sucede cuando se usa R. Por lo tanto, el usuario debe guardar explícitamente todo trazado como archivo PNG.

Para generar imágenes desde MatplotLib, debe seguir estos pasos:

1. Cambie el back-end a "AGG" desde el representador basado en Qt predeterminado.
1. Cree un nuevo objeto de figura.
1. Obtenga el eje y genere todos los trazados en él.
1. Guarde la figura en un archivo PNG.

Este proceso se muestra en las siguientes imágenes que crean una matriz de trazado de dispersión con la función scatter_matrix de Pandas.

![Código para guardar figuras de MatplotLib en imágenes.](./media/execute-python-scripts/figure-v1-8.png)

![Haga clic en visualizar en un módulo Ejecutar script de Python para ver las cifras](./media/execute-python-scripts/figure-v2-9a.png)

![Visualización de los trazados para un experimento de ejemplo con código Python](./media/execute-python-scripts/figure-v2-9b.png)

Es posible devolver varias figuras si se guardan en diferentes imágenes. El tiempo de ejecución de Studio (clásico) selecciona todas las imágenes y las concatena para su visualización.

## <a name="advanced-examples"></a>Ejemplos avanzados

El entorno Anaconda instalado en Studio (clásico) contiene paquetes comunes como NumPy, SciPy y Scikits-Learn. Estos paquetes se pueden usar de forma efectiva para procesamiento de datos en una canalización de aprendizaje automático.

Por ejemplo, el siguiente experimento y script muestran el uso de sistemas aprendices de conjunto de Scikits-Learn para calcular las puntuaciones de importancia de características de un conjunto de datos. Las puntuaciones se pueden usar para realizar una selección supervisada de características antes de transmitirlas a otro modelo.

Aquí se muestra la función de Python usada para calcular las puntuaciones de importancia y ordenar las características según esas puntuaciones:

![Función para clasificar características por puntuaciones](./media/execute-python-scripts/figure8.png)

El siguiente experimento calcula y devuelve las puntuaciones de importancia de características en el conjunto de datos "Pima Indian Diabetes" en Azure Machine Learning Studio (clásico):

![Experimento para clasificar las características del conjunto de datos Diabetes mediante Python](./media/execute-python-scripts/figure9a.png)

![Visualización de la salida del módulo Ejecutar script de Python](./media/execute-python-scripts/figure9b.png)

## <a name="limitations"></a>Limitaciones

El módulo [Ejecutar script de Python][execute-python-script] tiene actualmente las siguientes limitaciones:

### <a name="sandboxed-execution"></a>Ejecución en espacio aislado

El tiempo de ejecución de Python se realiza actualmente en un espacio aislado y no permite el acceso a la red o al sistema de archivos local de manera persistente. Todos los archivos guardados localmente son aislados y eliminados una vez que se finaliza el módulo. El código Python no puede tener acceso a la mayoría de los directorios de la máquina en que se ejecuta, con la excepción del directorio actual y sus subdirectorios.

### <a name="lack-of-sophisticated-development-and-debugging-support"></a>Falta de compatibilidad con la depuración y desarrollo sofisticado

El módulo Python actualmente no es compatible con características de IDE, como IntelliSense y depuración. Además, si se produce un error en el módulo en runtime, el seguimiento de la pila completo de Python está disponible. Pero debe verse en el registro de salida del módulo. De momento se recomienda que los usuarios desarrollen y depuren scripts de Python en un entorno como IPython y luego importen el código al módulo.

### <a name="single-data-frame-output"></a>Salida de una trama de datos

El punto de entrada de Python solo tiene permitido devolver una trama de datos como salida. Actualmente no es posible devolver objetos arbitrarios de Python, como modelos entrenados, directamente de vuelta al tiempo de ejecución de Studio (clásico). Al igual que [Ejecutar script R][execute-r-script], que tiene la misma limitación, en muchos casos es posible incluir objetos en una matriz de bytes y luego devolverla dentro de una trama de datos.

### <a name="inability-to-customize-python-installation"></a>Incapacidad para personalizar la instalación de Python

Actualmente, la única manera de agregar módulos personalizados de Python es a través del mecanismo de archivo ZIP descrito anteriormente. A pesar de que esto es viable cuando se trata de módulos pequeños, es complicado para módulos de gran tamaño (especialmente para los módulos con DLL nativas) o para un gran número de módulos.

## <a name="next-steps"></a>Pasos siguientes

Para más información, vea el [Centro para desarrolladores de Python](https://azure.microsoft.com/develop/python/).

<!-- Module References -->
[execute-python-script]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/execute-python-script
[execute-r-script]: https://docs.microsoft.com/azure/machine-learning/studio-module-reference/execute-r-script
