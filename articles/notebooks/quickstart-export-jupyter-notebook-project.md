---
title: Exportación de un proyecto de Jupyter Notebooks desde Azure Notebooks (versión preliminar)
description: Exporte rápidamente un proyecto de Jupyter Notebooks.
ms.topic: quickstart
ms.date: 06/29/2020
ms.openlocfilehash: a2e76f2c1956df4d0de59cfb429dc1c9dce85a62
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892106"
---
# <a name="quickstart-export-a-jupyter-notebook-project-in-azure-notebooks-preview"></a>Inicio rápido: Exportación de un proyecto de un cuaderno de Jupyter Notebook en Azure Notebooks (versión preliminar)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

En este inicio rápido descargará un proyecto de Azure Notebooks para usarlo en otras soluciones de Jupyter Notebooks. 

## <a name="prerequisites"></a>Requisitos previos

Un proyecto de Azure Notebooks existente.

## <a name="export-an-azure-notebooks-project"></a>Exportación de un proyecto de Azure Notebooks

1. Vaya a [Azure Notebooks](https://notebooks.azure.com) e inicie sesión. Para más información, consulte [Inicio rápido: Inicio de sesión en Azure Notebooks](quickstart-sign-in-azure-notebooks.md).

1. En la página del perfil público, seleccione **My Projects** (Mis proyectos) en la parte superior de la página:

    ![Vínculo Mis proyectos en la parte superior de la ventana del explorador](media/quickstarts/my-projects-link.png)

1. Seleccione un proyecto.
1. Haga clic en el botón "Descargar" para desencadenar la descarga de un archivo ZIP que contiene todos los archivos del proyecto.
1. O también, en una página específica del proyecto, haga clic en el botón "Descargar proyecto" para descargar todos los archivos de un proyecto dado.

Después de descargar los archivos del proyecto puede usarlos con otras soluciones de Jupyter Notebook. Estas son algunas de las opciones que se describen en las secciones siguientes: 
- [Visual Studio Code](#use-notebooks-in-visual-studio-code)
- [Visual Studio Codespaces](#use-notebooks-in-visual-studio-codespaces)
- [Azure Machine Learning](#use-notebooks-with-azure-machine-learning)
- [Azure Lab Services](#use-azure-lab-services)
- [GitHub](#use-github)

## <a name="create-an-environment-for-notebooks"></a>Creación de un entorno para Notebooks

Si desea crear un entorno que coincida con el de la versión preliminar de Azure Notebooks, puede usar el archivo de script que se proporciona en GitHub.

1. Vaya al [repositorio de GitHub](https://github.com/microsoft/AzureNotebooks) de Azure Notebooks o [acceda directamente a la carpeta del entorno](https://aka.ms/aznbrequirementstxt).
1. En un símbolo del sistema, vaya al directorio que desea usar para los proyectos.
1. Descargue el contenido de la carpeta del entorno y siga las instrucciones del archivo Léame para instalar las dependencias del paquete de Azure Notebooks.


## <a name="use-notebooks-in-visual-studio-code"></a>Uso de Notebooks en Visual Studio Code

[VS Code](https://code.visualstudio.com/) es un editor de código gratuito que se puede usar tanto localmente como conectado a un proceso remoto. Si se combina con la extensión de Python, ofrece un entorno completo para el desarrollo de Python, lo que incluye una experiencia nativa enriquecida para trabajar con cuadernos de Jupyter Notebook. 

![Compatibilidad de VS Code y Jupyter Notebook](media/vs-code-jupyter-notebook.png)

Después de [descargar](#export-an-azure-notebooks-project) los archivos del proyecto puede usarlos con VS Code. Para obtener instrucciones sobre cómo usar VS Code con Jupyter Notebooks, consulte los tutoriales sobre cómo [trabajar con Jupyter Notebooks en Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support) y sobre la [ciencia de datos en Visual Studio Code](https://code.visualstudio.com/docs/python/data-science-tutorial).

También puede usar el [script de entorno de Azure Notebooks](#create-an-environment-for-notebooks) con Visual Studio Code para crear un entorno que coincida con la versión preliminar de Azure Notebooks.

## <a name="use-notebooks-in-visual-studio-codespaces"></a>Uso de Notebooks en Visual Studio Codespaces

Visual Studio Codespaces proporciona entornos hospedados en los que se pueden editar cuadernos mediante Visual Studio Code o un explorador web. Ofrece la misma experiencia de Jupyter que VS Code, pero sin necesidad de instalar nada en el dispositivo. Si no desea configurar un entorno local y prefiere una solución con el respaldo de la nube, una solución excelente es crear un espacio de código. Primeros pasos:

1. [Descargue](#export-an-azure-notebooks-project) los archivos del proyecto.
1. [Cree un repositorio de GitHub](https://help.github.com/github/getting-started-with-github/create-a-repo) para almacenar los cuadernos. 
1. [Agregue los archivos](https://help.github.com/github/managing-files-in-a-repository/adding-a-file-to-a-repository) al repositorio.
1. Configure Visual Studio Codespaces en el [explorador](https://docs.microsoft.com/visualstudio/online/how-to/browser), [Visual Studio](https://docs.microsoft.com/visualstudio/online/how-to/vside) o [Visual Studio Code](https://docs.microsoft.com/visualstudio/online/how-to/vscode).

## <a name="use-notebooks-with-azure-machine-learning"></a>Uso de Notebooks con Azure Machine Learning

Azure Machine Learning proporciona una plataforma de aprendizaje automático de un extremo a otro que permite a los usuarios crear e implementar modelos más rápidamente en Azure. Azure Machine Learning permite ejecutar cuadernos de Jupyter Notebook en una máquina virtual o en un entorno de computación de clúster compartido. Si necesita una solución en la nube para su carga de trabajo de Machine Learning con seguimiento de experimentos, administración de conjuntos de datos, etc., es aconsejable que use Azure Machine Learning. Para empezar a utilizar Azure Machine Learning:

1. [Descargue](#export-an-azure-notebooks-project) los archivos del proyecto.
1. [Cree un área de trabajo](../machine-learning/how-to-manage-workspace.md) en Azure Portal.

   ![Crear un área de trabajo](../machine-learning/media/how-to-manage-workspace/create-workspace.gif)
 
1. Abra [Azure Studio (versión preliminar)](https://ml.azure.com/).
1. Use la barra de navegación izquierda para seleccionar **Notebooks**.
1. Haga clic en el botón **Cargar archivos** y cargue los archivos del proyecto que descargó de Azure Notebooks.

Para más información sobre Azure Machine Learning y la ejecución de Jupyter Notebooks, puede consultar la [documentación](../machine-learning/how-to-run-jupyter-notebooks.md) o probar el módulo [Introducción a Azure Machine Learning](https://docs.microsoft.com/learn/modules/intro-to-azure-machine-learning-service/) de Microsoft Learn.


## <a name="use-azure-lab-services"></a>Uso de Azure Lab Services

[Azure Lab Services](https://azure.microsoft.com/services/lab-services/) permite a los formadores configurar y proporcionar fácilmente a toda una clase acceso a petición a las máquinas virtuales preconfiguradas. Si busca una manera de trabajar con Jupyter Notebooks y con procesos en la nube en un entorno docente adaptado, Lab Services es una excelente opción.

![imagen](../lab-services/media/tutorial-setup-classroom-lab/new-lab-button.png)

 Tras [descargar](#export-an-azure-notebooks-project) los archivos del proyecto puede usarlos con Azure Lab Services. Si desea una guía para configurar un laboratorio, consulte [Configuración de un laboratorio para enseñar ciencia de datos con Python y Jupyter Notebooks](../lab-services/class-type-jupyter-notebook.md)

## <a name="use-github"></a>Uso de GitHub

GitHub ofrece una forma gratuita y con el respaldo del control del origen de almacenar cuadernos (y otros archivos), compartir cuadernos con otros usuarios y trabajar de forma colaborativa. Si busca una forma de compartir proyectos y colaborar con otros usuarios, GitHub es una opción magnífica que, además, se puede combinar con [Visual Studio Codespaces](#use-notebooks-in-visual-studio-codespaces) para que la experiencia de desarrollo sea estupenda. Para empezar a usar GitHub

1. [Descargue](#export-an-azure-notebooks-project) los archivos del proyecto.
1. [Cree un repositorio de GitHub](https://help.github.com/github/getting-started-with-github/create-a-repo) para almacenar los cuadernos. 
1. [Agregue los archivos](https://help.github.com/github/managing-files-in-a-repository/adding-a-file-to-a-repository) al repositorio.

## <a name="next-steps"></a>Pasos siguientes

- [Información acerca de Python en Visual Studio Code](https://code.visualstudio.com/docs/python/python-tutorial)
- [Información acerca de Azure Machine Learning y cuadernos de Jupyter Notebook](../machine-learning/how-to-run-jupyter-notebooks.md)
- [Información sobre Visual Studio Codespaces](https://visualstudio.microsoft.com/services/visual-studio-codespaces/)
- [Información sobre Azure Lab Services](https://azure.microsoft.com/services/lab-services/)
- [Información acerca de GitHub](https://help.github.com/github/getting-started-with-github/)
