---
title: Uso de Python y TensorFlow para el aprendizaje automático en Azure
description: Uso de Python, TensorFlow y Azure Functions con un modelo de aprendizaje automático para clasificar una imagen en función de su contenido
author: anthonychu
ms.topic: tutorial
ms.date: 01/15/2020
ms.author: antchu
ms.custom: mvc, devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: e9bbfd311d6a05d0dd328a63c7d11e14ab0d7e4a
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069619"
---
# <a name="tutorial-apply-machine-learning-models-in-azure-functions-with-python-and-tensorflow"></a>Tutorial: Aplicación de modelos de aprendizaje automático en Azure Functions con Python y TensorFlow

En este artículo, aprenderá a usar Python, TensorFlow y Azure Functions con un modelo de aprendizaje automático para clasificar una imagen en función de su contenido. Como todo el trabajo se realiza localmente y no se crean recursos de Azure en la nube, completar este tutorial no conlleva ningún costo.

> [!div class="checklist"]
> * Inicialice un entorno local para el desarrollo de Azure Functions en Python.
> * Importe un modelo de aprendizaje automático de TensorFlow personalizado en una aplicación de funciones.
> * Cree una API HTTP sin servidor para clasificar una imagen que contenga un perro o un gato.
> * Utilice la API desde una aplicación web.

## <a name="prerequisites"></a>Prerrequisitos 

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Python 3.7.4](https://www.python.org/downloads/release/python-374/) (Python 3.7.4 y Python 3.6.x se han comprobado con Azure Functions; Python 3.8 y las versiones posteriores aún no son compatibles).
- [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools).
- Un editor de código como [Visual Studio Code](https://code.visualstudio.com/)

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

1. En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es 2.7.1846 u otra posterior.
1. Ejecute `python --version` (Linux/MacOS) o `py --version` (Windows) para comprobar los informes de la versión de Python 3.7.x.

## <a name="clone-the-tutorial-repository"></a>Clonación del repositorio de tutoriales

1. En una ventana de terminal o de comandos, clone el siguiente repositorio mediante Git:

    ```
    git clone https://github.com/Azure-Samples/functions-python-tensorflow-tutorial.git
    ```

1. Vaya a la carpeta y examine su contenido.

    ```
    cd functions-python-tensorflow-tutorial
    ```

    - *start* es la carpeta de trabajo del tutorial.
    - *end* es el resultado final y la implementación completa como referencia.
    - *resources* contiene el modelo de aprendizaje automático y las bibliotecas auxiliares.
    - *frontend* es un sitio web que llama a la aplicación de funciones.
    
## <a name="create-and-activate-a-python-virtual-environment"></a>Creación y activación de un entorno virtual de Python

Vaya a la carpeta *start* y ejecute los siguientes comandos para crear y activar un entorno virtual llamado `.venv`. Asegúrese de usar Python 3.7, que es compatible con Azure Functions.


# <a name="bash"></a>[bash](#tab/bash)

```bash
cd start
```

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

Si Python no instaló el paquete venv en la distribución de Linux, ejecute el siguiente comando:

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
cd start
```

```powershell
py -3.7 -m venv .venv
```

```powershell
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd start
```

```cmd
py -3.7 -m venv .venv
```

```cmd
.venv\scripts\activate
```

---

Ejecute todos los comandos siguientes en este entorno virtual activado. (Para salir del entorno virtual, ejecute `deactivate`).


## <a name="create-a-local-functions-project"></a>Creación de un proyecto local de Functions

En Azure Functions, un proyecto de función es un contenedor para una o varias funciones individuales que responden a un desencadenador específico. Todas las funciones de un proyecto comparten las mismas configuraciones locales y de hospedaje. En esta sección, creará un proyecto de función que contiene una sola función reutilizable llamada `classify`, que proporciona un punto de conexión HTTP. Agregará código más específico en una sección posterior.

1. En la carpeta *start*, use Azure Functions Core Tools para inicializar una aplicación de funciones de Python:

    ```
    func init --worker-runtime python
    ```

    Después de la inicialización, la carpeta *start* contiene varios archivos del proyecto, incluidos los archivos de configuración [local.settings.json](functions-run-local.md#local-settings-file) y [host.json](functions-host-json.md). Como *local.settings.json* puede contener secretos descargados de Azure, el archivo se excluye del control de código fuente de forma predeterminada en el archivo *.gitignore*.

    > [!TIP]
    > Como un proyecto de función está asociado a un entorno de ejecución específico, todas las funciones del proyecto deben estar escritas con el mismo lenguaje.

1. Agregue una función al proyecto mediante el comando siguiente, donde el argumento `--name` es el nombre único de la función y el argumento `--template` especifica el desencadenador de esta. `func new` crea una subcarpeta que coincide con el nombre de la función y que contiene un archivo de código apropiado para el lenguaje elegido del proyecto y un archivo de configuración denominado *function.json*.

    ```
    func new --name classify --template "HTTP trigger"
    ```

    Este comando crea una carpeta que coincide con el nombre de la función *classify*. En esa carpeta hay dos archivos: *\_\_init\_\_.py*, que contiene el código de la función, y *function.json*, que describe el desencadenador de la función y sus enlaces de entrada y salida. Para obtener información detallada sobre el contenido de estos archivos, consulte [Examen del contenido del archivo](./functions-create-first-azure-function-azure-cli.md?pivots=programming-language-python#optional-examine-the-file-contents) en la guía de inicio rápido de Python.


## <a name="run-the-function-locally"></a>Ejecución local de la función

1. Inicie la función, iniciando para ello el host del entorno de ejecución de Azure Functions local en la carpeta *start*:

    ```
    func start
    ```
    
1. Cuando vea que el punto de conexión `classify` aparece en la salida, vaya a la dirección URL ```http://localhost:7071/api/classify?name=Azure```. El mensaje "Hello Azure!" aparecerá en la salida.

1. Use **Ctrl**-**C** para detener el host.


## <a name="import-the-tensorflow-model-and-add-helper-code"></a>Importación del modelo de TensorFlow y adición del código auxiliar

Para modificar la función `classify` para clasificar una imagen en función de su contenido, se usa un modelo de TensorFlow predefinido que se ha entrenado y se ha exportado desde Azure Custom Vision Service. El modelo, que se encuentra en la carpeta *resources* del ejemplo clonado anteriormente, clasifica una imagen en función de si contiene un perro o un gato. A continuación, agregue el código auxiliar y las dependencias al proyecto.

Para crear su propio modelo con el nivel gratuito de Custom Vision Service, siga las instrucciones del [repositorio del proyecto de ejemplo](https://github.com/Azure-Samples/functions-python-tensorflow-tutorial/blob/master/train-custom-vision-model.md).

> [!TIP]
> Si desea hospedar el modelo de TensorFlow de modo independiente respecto de la aplicación de función, puede montar un recurso compartido de archivos que contenga el modelo en la aplicación de funciones de Linux. Para más información, consulte [Montaje de un recurso compartido de archivos en una aplicación de funciones de Python mediante la CLI de Azure](./scripts/functions-cli-mount-files-storage-linux.md).

1. En la carpeta *start*, ejecute el siguiente comando para copiar los archivos del modelo en la carpeta *classify*. Asegúrese de incluir `\*` en el comando. 

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    cp ../resources/model/* classify
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    copy ..\resources\model\* classify
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    copy ..\resources\model\* classify
    ```
    
    ---
    
1. Compruebe que la carpeta *classify* contiene los archivos llamados *model.pb* y *labels.txt*. Si no es así, compruebe que ejecutó el comando en la carpeta *start*.

1. En la carpeta *start*, ejecute el siguiente comando para copiar un archivo con el código auxiliar en la carpeta *classify*:

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    cp ../resources/predict.py classify
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    copy ..\resources\predict.py classify
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    copy ..\resources\predict.py classify
    ```
    
    ---

1. Compruebe que la carpeta *classify* contiene ahora un archivo llamado *predict.py*.

1. Abra *start/requirements.txt* en un editor de texto y agregue las siguientes dependencias requeridas por el código auxiliar:

    ```txt
    tensorflow==1.14
    Pillow
    requests
    ```
    
1. Guarde *requirements.txt*.

1. Ejecute el siguiente comando en la carpeta *start* para instalar las dependencias. La instalación puede tardar unos minutos. Durante este tiempo, puede continuar modificando la función en la sección siguiente.

    ```
    pip install --no-cache-dir -r requirements.txt
    ```
    
    En Windows, quizás se produzca el error "No se pudieron instalar los paquetes debido a un error de entorno: [Errno 2]. No existe tal archivo o directorio:", seguido de una ruta de acceso larga a un archivo como *sharded_mutable_dense_hashtable.cpython-37.pyc*. Normalmente, este error se produce porque la ruta de acceso de la carpeta es demasiado larga. En este caso, establezca la clave del Registro `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem@LongPathsEnabled` en `1` para habilitar rutas de acceso largas. Como alternativa, compruebe dónde está instalado el intérprete de Python. Si esa ubicación tiene una ruta de acceso larga, intente volver a instalarlo en una carpeta con una ruta de acceso más corta.

> [!TIP]
> Cuando se llama a *predict.py* para realizar la primera predicción, una función llamada `_initialize` carga el modelo de TensorFlow desde el disco y lo almacena en memoria caché, en variables globales. Este almacenamiento en caché acelera las predicciones posteriores. Para más información sobre el uso de variables globales, consulte la [guía de Azure Functions para desarrolladores de Python](functions-reference-python.md#global-variables).

## <a name="update-the-function-to-run-predictions"></a>Actualización de la función para ejecutar predicciones

1. Abra *classify/\_\_init\_\_.py* en un editor de texto y agregue las siguientes líneas después de las instrucciones `import` existentes para importar la biblioteca JSON estándar y las aplicaciones auxiliares *predict*:

    :::code language="python" source="~/functions-python-tensorflow-tutorial/end/classify/__init__.py" range="1-6" highlight="5-6":::

1. Reemplace todo el contenido de la función `main` por el código siguiente:

    :::code language="python" source="~/functions-python-tensorflow-tutorial/end/classify/__init__.py" range="8-19":::

    Esta función recibe una dirección URL de imagen en un parámetro de cadena de consulta denominado `img`. A continuación, llama a `predict_image_from_url` desde la biblioteca auxiliar para descargar y clasificar la imagen mediante el modelo de TensorFlow. A continuación, la función devuelve una respuesta HTTP con los resultados. 

    > [!IMPORTANT]
    > Como una página web hospedada en otro dominio llama a este punto de conexión HTTP, la respuesta incluye un encabezado `Access-Control-Allow-Origin` para satisfacer los requisitos de uso compartido de recursos entre orígenes (CORS) del explorador.
    >
    > En una aplicación de producción, cambie `*` al origen específico de la página web para mayor seguridad.

1. Guarde los cambios y, si las dependencias han terminado de instalarse, vuelva a iniciar el host de la función local con `func start`. Asegúrese de ejecutar el host en la carpeta *start* con el entorno virtual activado. De lo contrario, se iniciará el host, pero verá errores al invocar la función.

    ```
    func start
    ```

1. En un explorador, abra la siguiente dirección URL para invocar la función con la dirección URL de una imagen de gato y confirmar que el código JSON devuelto clasifica la imagen como un gato.

    ```
    http://localhost:7071/api/classify?img=https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat1.png
    ```
    
1. Mantenga el host en ejecución porque lo usará en el paso siguiente. 

### <a name="run-the-local-web-app-front-end-to-test-the-function"></a>Ejecución del front-end de la aplicación web local para probar la función

Para probar la invocación del punto de conexión de la función desde otra aplicación web, hay una aplicación simple en la carpeta *frontend* del repositorio.

1. Abra un nuevo terminal o símbolo del sistema y active el entorno virtual (tal y como se describió anteriormente en [Creación y activación de un entorno virtual de Python](#create-and-activate-a-python-virtual-environment)).

1. Vaya a la carpeta *frontend* del repositorio.

1. Inicie un servidor HTTP con Python:

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash 
    python -m http.server
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    py -m http.server
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    py -m http.server
    ```

1. En un explorador, vaya a `localhost:8000`, escriba una de las siguientes direcciones URL de foto en el cuadro de texto o use la dirección URL de cualquier imagen accesible públicamente.

    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat1.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat2.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/dog1.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/dog2.png`
    
1. Seleccione **Submit** para invocar el punto de conexión de función para clasificar la imagen.

    ![Captura de pantalla del proyecto terminado](media/functions-machine-learning-tensorflow/functions-machine-learning-tensorflow-screenshot.png)

    Si el explorador informa de un error al enviar la dirección URL de la imagen, compruebe el terminal en el que está ejecutando la aplicación de funciones. Si ve un error similar a "No se encontró ningún módulo PIL", es posible que haya iniciado la aplicación de funciones en la carpeta *start* sin activar primero el entorno virtual que creó anteriormente. Si sigue viendo errores, ejecute de nuevo `pip install -r requirements.txt` con el entorno virtual activado y busque errores.

> [!NOTE]
> El modelo siempre clasifica el contenido de la imagen como un gato o un perro, con independencia de si la imagen contiene alguno. El valor predeterminado es perro. Las imágenes de tigres y panteras, por ejemplo, normalmente se clasifican como gato, pero las imágenes de elefantes, zanahorias o aviones se clasifican como perro.

## <a name="clean-up-resources"></a>Limpieza de recursos

La totalidad de este tutorial se ejecuta localmente en la máquina, por lo que no hay que limpiar ningún recurso o servicio de Azure.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió a crear y personalizar un punto de conexión de API HTTP con Azure Functions para clasificar mediante un modelo de TensorFlow. También ha aprendido a llamar a la API desde una aplicación web. Puede utilizar estas técnicas del tutorial para crear API de cualquier complejidad, todo ello mientras ejecuta el modelo de proceso sin servidor proporcionado por Azure Functions.

> [!div class="nextstepaction"]
> [Implementación de la función en Azure Functions mediante la guía de la CLI de Azure](./functions-run-local.md#publish)

Consulte también:

- [Implementación de la función en Azure mediante Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-azure-functions).
- [Guía de Azure Functions para desarrolladores de Python](./functions-reference-python.md)
- [Montaje de un recurso compartido de archivos en una aplicación de funciones de Python mediante la CLI de Azure](./scripts/functions-cli-mount-files-storage-linux.md)
