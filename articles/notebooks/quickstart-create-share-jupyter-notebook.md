---
title: Creación y uso compartido de un cuaderno de Jupyter Notebook en Azure Notebooks (versión preliminar)
description: Cree y ejecute rápidamente un cuaderno de Jupyter Notebook en Azure Notebooks (versión preliminar) y, luego, comparta ese cuaderno con otros usuarios.
ms.topic: quickstart
ms.date: 12/04/2018
ms.custom: devx-track-python
ms.openlocfilehash: 19773a6857d58f5b6af2a02ac28a3414c51fc917
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87849409"
---
# <a name="quickstart-create-and-share-a-notebook-in-azure-notebooks-preview"></a>Inicio rápido: Creación y uso compartido de un cuaderno en Azure Notebooks (versión preliminar)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

En este inicio rápido, se crear y ejecuta un cuaderno de Jupyter Notebook en Azure Notebooks y, después, se comparte con otros usuarios. Jupyter permite combinar fácilmente texto Markdown, código ejecutable, datos persistentes, gráficos y visualizaciones en un solo lienzo que puede compartirse, el cuaderno. Azure Notebooks es un servicio hospedado gratuito para desarrollar y ejecutar cuadernos de Jupyter Notebook en la nube sin necesidad de instalación.

## <a name="prerequisites"></a>Prerrequisitos
Ninguno.

## <a name="create-a-new-project-and-notebook"></a>Creación de un nuevo proyecto y un cuaderno

1. Vaya al [sitio de Azure Notebooks (https://notebooks.azure.com)](https://notebooks.azure.com) e inicie sesión. Para más información, consulte [Inicio rápido: Inicio de sesión en Azure Notebooks](quickstart-sign-in-azure-notebooks.md).

1. En la página del perfil público, seleccione **My Projects** (Mis proyectos) en la parte superior de la página:

    ![Vínculo Mis proyectos en la parte superior de la ventana del explorador](media/quickstarts/my-projects-link.png)

1. En la página **My Projects** (Mis proyectos), seleccione **+ New Project** (+ Nuevo proyecto) (método abreviado de teclado: n). Si la ventana del explorador es estrecha, el botón puede aparecer solo como **+** :

    ![Comando New Project (Nuevo proyecto) en la página My Projects (Mis proyectos)](media/quickstarts/new-project-command.png)

1. En la ventana emergente **Create New Project** (Crear nuevo proyecto) que aparece, escriba o establezca estos detalles y, luego, seleccione **Create** (Crear):

   - **Project name** (Nombre del proyecto): Hola mundo en Python
   - **Identificador del proyecto**: hello-world-python
   - **Public project** (Proyecto público): desactivada
   - **Create a README.md** (Crear un Léame.md): desactivada

     ![Ventana emergente Nuevo proyecto con los detalles completados](media/quickstarts/new-project-popup.png)

1. Después de unos instantes, Azure Notebooks lo lleva al proyecto nuevo. Para agregar un cuaderno al proyecto, seleccione el menú desplegable **+ Nuevo** (que puede aparecer simplemente como **+** ) y seleccione **Cuaderno**:

    :::image type="content" source="media/quickstarts/empty-project-new-notebook-button.png" alt-text="Un nuevo proyecto vacío y agregar un comando de cuaderno." lightbox="media/quickstarts/empty-project-new-notebook-button.png":::

1. En la ventaja emergente **Create New Notebook** (Crear nuevo cuaderno) que aparece, escriba un nombre de archivo para el cuaderno, como *HelloWorldInPython.ipynb* (*.ipynb* se refiere a IronPython (Jupyter) Notebook) y seleccione **Python 3.6** en el lenguaje (que también se denomina como el *kernel*):

    ![La ventana emergente para crear un cuaderno](media/quickstarts/new-notebook-popup.png)

1. Seleccione **Nuevo** para terminar de crear el cuaderno, el que aparecerá en la lista de archivos del proyecto en cuestión:

    ![El cuaderno nuevo aparece en la lista de archivos del proyecto](media/quickstarts/new-notebook-created.png)

## <a name="run-the-notebook"></a>Ejecución del cuaderno

1. Seleccione el cuaderno nuevo para ejecutarlo en el editor; el kernel que seleccionó (en este ejemplo, Python 3.6) se activa automáticamente para este cuaderno:

    ![Vista de un cuaderno nuevo en Azure Notebooks](media/quickstarts/create-notebook-first-open.png)

1. De manera predeterminada, el cuaderno tiene una celda de código vacía. Para cambiar el tipo de celda a **Markdown**, use el menú desplegable de tipos de celda para seleccionar **Markdown**:

    ![Cambio del tipo de celda de un cuaderno nuevo](media/quickstarts/create-notebook-cell-type.png)

1. Escriba o pegue el texto de encabezado siguiente en la celda:

    ```markdown
    # Hello World in Python
    ```

1. Como se edita Markdown, el texto aparece como encabezado con el carácter "#". Para representar el archivo Markdown en HTML, seleccione el botón **Ejecutar**. Luego, Azure Notebooks crea automáticamente una celda de código nueva:

    ![El botón de ejecución de una celda y la representación del archivo Markdown](media/quickstarts/run-cell-markdown-render.png)

1. En la celda de código, escriba el código de Python siguiente:

    ```python
    from datetime import datetime

    now = datetime.now()
    msg = "Hello, Azure Notebooks! Today is %s"  % now.strftime("%A, %d %B, %Y")
    print(msg)
    ```

1. Seleccione **Ejecutar** (método abreviado de teclado: Mayús + ENTRAR) para ejecutar el código. Debajo de la celda debería aparecer una salida correcta similar al texto siguiente:

    ```output
    Hello, Azure Notebooks! Today is Thursday, 15 November, 2018
    ```

1. Seleccione el icono Guardar para guardar el trabajo:

    ![Icono Guardar en la barra de herramientas de Jupyter Notebook](media/quickstarts/hello-results-save-icon.png)

1. Seleccione el comando de menú **Archivo** > **Close and Halt** (Cerrar y detener) para detener el servidor y cerrar la ventana del explorador.

## <a name="share-the-notebook"></a>Uso compartido del cuaderno

Para compartir el cuaderno, vuelva a la página del proyecto si es necesario, haga clic con el botón derecho en el archivo del cuaderno, seleccione **Copiar vínculo** (método abreviado de teclado: y) y pegue el vínculo en un mensaje adecuado (correo electrónico, mensaje instantáneo, etc.).

En la página del proyecto, también puede usar el menú **Compartir** para obtener un vínculo, crear un mensaje de correo electrónico con el vínculo u obtener el código HTML y Markdown para insertar:

![Comando para compartir el proyecto](media/quickstarts/share-project-command.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Creación y ejecución de un cuaderno de Jupyter Notebook para hacer regresión lineal](tutorial-create-run-jupyter-notebook.md)
