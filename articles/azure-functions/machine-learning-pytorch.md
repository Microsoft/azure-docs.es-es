---
title: Implementación de un modelo PyTorch como una aplicación Azure Functions
description: Use una red neuronal profunda de ResNet 18 previamente entrenada de PyTorch con Azure Functions para asignar 1 de 1000 ImageNet etiquetas a una imagen.
author: gvashishtha
ms.topic: tutorial
ms.date: 02/28/2020
ms.author: gopalv
ms.custom: devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: dd8de1b56927e158a181df952ce0dbeed140d6b7
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078663"
---
# <a name="tutorial-deploy-a-pre-trained-image-classification-model-to-azure-functions-with-pytorch"></a>Tutorial: Implementación de un modelo de clasificación de imágenes previamente entrenado en Azure Functions con PyTorch

En este artículo, aprenderá a usar Python, PyTorch y Azure Functions para cargar un modelo previamente entrenado para clasificar una imagen en función de su contenido. Como todo el trabajo se realiza localmente y no se crean recursos de Azure en la nube, completar este tutorial no conlleva ningún costo.

> [!div class="checklist"]
> * Inicialice un entorno local para el desarrollo de Azure Functions en Python.
> * Importe un modelo de aprendizaje automático de PyTorch previamente entrenado en una aplicación de función.
> * Cree una API HTTP sin servidor para clasificar una imagen como una de las clases 1000 [ImageNet](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a).
> * Utilice la API desde una aplicación web.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Python 3.7.4 o versiones posteriores](https://www.python.org/downloads/release/python-374/). (Python 3.8. x y Python 3.6. x también se comprueban con Azure Functions).
- [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools).
- Un editor de código como [Visual Studio Code](https://code.visualstudio.com/)

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

1. En una ventana de terminal o de comandos, ejecute `func --version` para comprobar que la versión de Azure Functions Core Tools es 2.7.1846 u otra posterior.
1. Ejecute `python --version` (Linux/MacOS) o `py --version` (Windows) para comprobar los informes de la versión de Python 3.7.x.

## <a name="clone-the-tutorial-repository"></a>Clonación del repositorio de tutoriales

1. En una ventana de terminal o de comandos, clone el siguiente repositorio mediante Git:

    ```
    git clone https://github.com/Azure-Samples/functions-python-pytorch-tutorial.git
    ```

1. Vaya a la carpeta y examine su contenido.

    ```
    cd functions-python-pytorch-tutorial
    ```

    - *start* es la carpeta de trabajo del tutorial.
    - *end* es el resultado final y la implementación completa como referencia.
    - *resources* contiene el modelo de aprendizaje automático y las bibliotecas auxiliares.
    - *frontend* es un sitio web que llama a la aplicación de funciones.

## <a name="create-and-activate-a-python-virtual-environment"></a>Creación y activación de un entorno virtual de Python

Vaya a la carpeta *start* y ejecute los siguientes comandos para crear y activar un entorno virtual llamado `.venv`.


# <a name="bash"></a>[bash](#tab/bash)

```bash
cd start
python -m venv .venv
source .venv/bin/activate
```

Si Python no instaló el paquete venv en la distribución de Linux, ejecute el siguiente comando:

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
cd start
py -m venv .venv
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd start
py -m venv .venv
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


## <a name="import-the-pytorch-model-and-add-helper-code"></a>Importación del modelo PyTorch e incorporación del código auxiliar

Para modificar la función `classify` para clasificar una imagen en base a su contenido, se usa un modelo previamente entrenado [ResNet](https://arxiv.org/abs/1512.03385). El modelo previamente entrenado, que procede de [PyTorch](https://pytorch.org/hub/pytorch_vision_resnet/), clasifica una imagen en 1 de 1000 [clases ImageNet](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a). A continuación, agregue el código auxiliar y las dependencias al proyecto.

1. En la carpeta *start*, ejecute el siguiente comando para copiar el código de predicción y las etiquetas en la carpeta de *classify*.

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    cp ../resources/predict.py classify
    cp ../resources/labels.txt classify
    ```

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    copy ..\resources\predict.py classify
    copy ..\resources\labels.txt classify
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    copy ..\resources\predict.py classify
    copy ..\resources\labels.txt classify
    ```

    ---

1. Compruebe que la carpeta *classify* contiene los archivos llamados *predict.py* y *labels.txt*. Si no es así, compruebe que ejecutó el comando en la carpeta *start*.

1. Abra *start/requirements.txt* en un editor de texto y agregue las dependencias requeridas por el código auxiliar, todo ello debería tener este aspecto:

    ```txt
    azure-functions
    requests
    -f https://download.pytorch.org/whl/torch_stable.html
    torch==1.5.0+cpu
    torchvision==0.6.0+cpu
    ```

1. Guarde *requirements. txt*y, después, ejecute el siguiente comando desde la carpeta *start* para instalar las dependencias.


    ```
    pip install --no-cache-dir -r requirements.txt
    ```

La instalación puede tardar unos minutos. Durante este tiempo, puede continuar modificando la función en la sección siguiente.
> [!TIP]
> >En Windows, quizás se produzca el error "No se pudieron instalar los paquetes debido a un error de entorno: [Errno 2]. No existe tal archivo o directorio:", seguido de una ruta de acceso larga a un archivo como *sharded_mutable_dense_hashtable.cpython-37.pyc*. Normalmente, este error se produce porque la ruta de acceso de la carpeta es demasiado larga. En este caso, establezca la clave del Registro `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem@LongPathsEnabled` en `1` para habilitar rutas de acceso largas. Como alternativa, compruebe dónde está instalado el intérprete de Python. Si esa ubicación tiene una ruta de acceso larga, intente volver a instalarlo en una carpeta con una ruta de acceso más corta.

## <a name="update-the-function-to-run-predictions"></a>Actualización de la función para ejecutar predicciones

1. Abra *classify/\_\_init\_\_.py* en un editor de texto y agregue las siguientes líneas después de las instrucciones `import` existentes para importar la biblioteca JSON estándar y las aplicaciones auxiliares *predict*:

    :::code language="python" source="~/functions-pytorch/end/classify/__init__.py" range="1-6" highlight="5-6":::

1. Reemplace todo el contenido de la función `main` por el código siguiente:

    :::code language="python" source="~/functions-pytorch/end/classify/__init__.py" range="8-19":::

    Esta función recibe una dirección URL de imagen en un parámetro de cadena de consulta denominado `img`. A continuación, llama a `predict_image_from_url` desde la biblioteca auxiliar para descargar y clasificar la imagen mediante el modelo PyTorch. A continuación, la función devuelve una respuesta HTTP con los resultados.

    > [!IMPORTANT]
    > Como una página web hospedada en otro dominio llama a este punto de conexión HTTP, la respuesta incluye un encabezado `Access-Control-Allow-Origin` para satisfacer los requisitos de uso compartido de recursos entre orígenes (CORS) del explorador.
    >
    > En una aplicación de producción, cambie `*` al origen específico de la página web para mayor seguridad.

1. Guarde los cambios y, si las dependencias han terminado de instalarse, vuelva a iniciar el host de la función local con `func start`. Asegúrese de ejecutar el host en la carpeta *start* con el entorno virtual activado. De lo contrario, se iniciará el host, pero verá errores al invocar la función.

    ```
    func start
    ```

1. En un explorador, abra la siguiente dirección URL para invocar la función con la dirección URL de una imagen de un perro boyero de Berna y confirmar que el código JSON devuelto clasifica la imagen como un perro boyero de Berna.

    ```
    http://localhost:7071/api/classify?img=https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/Bernese-Mountain-Dog-Temperament-long.jpg
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

    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/Bernese-Mountain-Dog-Temperament-long.jpg`
    - `https://github.com/Azure-Samples/functions-python-pytorch-tutorial/blob/master/resources/assets/bald-eagle.jpg?raw=true`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/penguin.jpg`

1. Seleccione **Submit** para invocar el punto de conexión de función para clasificar la imagen.

    ![Captura de pantalla del proyecto terminado](media/machine-learning-pytorch/screenshot.png)

    Si el explorador informa de un error al enviar la dirección URL de la imagen, compruebe el terminal en el que está ejecutando la aplicación de funciones. Si ve un error similar a "No se encontró ningún módulo PIL", es posible que haya iniciado la aplicación de funciones en la carpeta *start* sin activar primero el entorno virtual que creó anteriormente. Si sigue viendo errores, ejecute de nuevo `pip install -r requirements.txt` con el entorno virtual activado y busque errores.

## <a name="clean-up-resources"></a>Limpieza de recursos

La totalidad de este tutorial se ejecuta localmente en la máquina, por lo que no hay que limpiar ningún recurso o servicio de Azure.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió a crear y personalizar un punto de conexión de API HTTP con Azure Functions para clasificar mediante un modelo PyTorch. También ha aprendido a llamar a la API desde una aplicación web. Puede utilizar estas técnicas del tutorial para crear API de cualquier complejidad, todo ello mientras ejecuta el modelo de proceso sin servidor proporcionado por Azure Functions.

Consulte también:

- [Implementación de la función en Azure mediante Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-azure-functions).
- [Guía de Azure Functions para desarrolladores de Python](./functions-reference-python.md)


> [!div class="nextstepaction"]
> [Implementación de la función en Azure Functions mediante la guía de la CLI de Azure](./functions-run-local.md#publish)
