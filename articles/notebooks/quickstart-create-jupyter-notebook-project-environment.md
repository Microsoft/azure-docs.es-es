---
title: Creación de un proyecto de Azure Notebooks (versión preliminar) con un entorno personalizado
description: Cree un proyecto en Azure Notebooks (versión preliminar) que esté configurado con un conjunto específico de scripts de inicio y paquetes instalados.
ms.topic: quickstart
ms.date: 12/04/2018
ms.custom: devx-track-python
ms.openlocfilehash: 655c016b55abdcf4b6f546a1fe16348ec4c83724
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853371"
---
# <a name="quickstart-create-a-project-with-a-custom-environment-in-azure-notebooks-preview"></a>Inicio rápido: Creación de un proyecto con un entorno personalizado en de Azure Notebooks (versión preliminar)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Un proyecto de Azure Notebooks es una colección de archivos, como cuadernos, archivos de datos, documentación, imágenes, etc. junto con un entorno que se puede configurar con comandos de instalación específicos. Al definir el entorno con el proyecto, cualquier usuario que clone el proyecto en su propia cuenta de Azure Notebooks tiene toda la información necesaria para volver a crear el entorno necesario.

## <a name="create-a-project"></a>Crear un proyecto

1. Vaya a [Azure Notebooks](https://notebooks.azure.com) e inicie sesión. (Para más información, consulte [Quickstart - Sign in to Azure Notebooks](quickstart-sign-in-azure-notebooks.md) [Inicio rápido: Inicio de sesión en Azure Notebooks]).

1. En la página del perfil público, seleccione **My Projects** (Mis proyectos) en la parte superior de la página:

    ![Vínculo Mis proyectos en la parte superior de la ventana del explorador](media/quickstarts/my-projects-link.png)

1. En la página **My Projects** (Mis proyectos), seleccione **+New Project** (+Nuevo proyecto) [método abreviado de teclado: n]; el botón puede aparecer solo como **+** si la ventana del explorador es estrecha:

    ![Comando New Project (Nuevo proyecto) en la página My Projects (Mis proyectos)](media/quickstarts/new-project-command.png)

1. En la ventana emergente **Create New Project** (Crear nuevo proyecto) que aparece, escriba o establezca estos detalles y, luego, seleccione **Create** (Crear):

    - **Project name** (Nombre del proyecto): Proyecto con un entorno personalizado
    - **Identificador del proyecto**: project-custom-environment
    - **Public project** (Proyecto público): desactivada
    - **Create a README.md** (Crear un Léame.md): desactivada

1. Después de unos instantes, Azure Notebooks lo lleva al proyecto nuevo. Para agregar un cuaderno al proyecto, seleccione el menú desplegable **+ Nuevo** (que puede aparecer simplemente como **+** ) y seleccione **Cuaderno**.

1. Asigne al cuaderno un nombre similar a *Entorno personalizado.ipynb*, seleccione **Python 3.6** como el lenguaje y, luego, seleccione **Nuevo**.

## <a name="add-a-custom-setup-step"></a>Incorporación de un paso de configuración personalizado

1. En la página del proyecto, seleccione **Configuración del proyecto**.

    ![Comando de configuración del proyecto](media/quickstarts/project-settings-command.png)

1. En la ventana emergente **Configuración del proyecto**, seleccione la pestaña **Entorno** y luego, en **Environment Setup Steps** (Pasos para la configuración del entorno), seleccione **+ Agregar**:

    ![Comando para agregar un nuevo paso para la configuración del entorno](media/quickstarts/environment-add-command.png)

1. El comando **+ Agregar** crea un paso definido por una operación y un archivo de destino que se selecciona de los archivos del proyecto. Se admiten las operaciones siguientes:

   | Operación | Descripción |
   | --- | --- |
   | Requirements.txt | Los proyectos de Python definen sus dependencias en un archivo llamado requirements.txt. Con esta opción, seleccione el archivo adecuado en la lista de archivos del proyecto y seleccione también la versión de Python en la otra lista desplegable que aparece. Si es necesario, seleccione **Cancelar** para volver al proyecto, cargue o cree el archivo y, luego, vuelva a la pestaña **Configuración del proyecto** > **Entorno** para crear un paso. Una vez que se crea este paso, al ejecutar un cuaderno en el proyecto, automáticamente se ejecuta `pip install -r <file>` |
   | Script de shell | Úselo para indicar un script de shell de bash (por lo general, un archivo con extensión *.sh*) que contiene cualquier comando que quiere ejecutar para inicializar el entorno. |
   | Environment.yml | Un proyecto de Python que usa conda para administrar un entorno usa un archivo *environments.yml* para describir las dependencias. Con esta opción, seleccione el archivo correspondiente en la lista de archivos del proyecto. |

   > [!WARNING]
   > Dado que se trata de un servicio en versión preliminar en desarrollo, actualmente hay un problema conocido en el que el valor `Environment.yml` no se aplica al proyecto según lo esperado. En este momento, el proyecto y los cuadernos de Jupyter que contiene no cargan el archivo de entorno especificado.

1. Para quitar cualquier paso de configuración, seleccione la **X** que se encuentra a la derecha del paso.

1. Una vez que estén todos los pasos de configuración, seleccione **Guardar**. (Seleccione **Cancelar** para descartar los cambios).

1. Para probar el entorno, cree y ejecute un cuaderno nuevo y, luego, cree una celda de código con instrucciones que dependan de un paquete del entorno, como usar una instrucción `import` de Python. Si la instrucción se realiza correctamente, el paquete necesario se instaló correctamente en el entorno.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Manage and configure projects in Azure Notebooks](configure-manage-azure-notebooks-projects.md) (Administración y configuración de proyectos en Azure Notebooks)

> [!div class="nextstepaction"]
> [Tutorial: Creación y ejecución de un cuaderno de Jupyter Notebook para realizar la regresión lineal](tutorial-create-run-jupyter-notebook.md)
