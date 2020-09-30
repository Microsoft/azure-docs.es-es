---
title: Uso de cuadernos con Azure Sentinel para cuestiones de seguridad
description: En este artículo se describe cómo usar cuadernos con las capacidades de búsqueda de Azure Sentinel.
services: sentinel
author: yelevin
ms.author: yelevin
ms.assetid: 1721d0da-c91e-4c96-82de-5c7458df566b
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.custom: mvc
ms.date: 09/06/2020
ms.openlocfilehash: ded332813a840892f640aa6f6e48debbfe381b4b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90889298"
---
# <a name="use-jupyter-notebooks-to-hunt-for-security-threats"></a>Uso de cuadernos de Jupyter para buscar amenazas de seguridad

La base de Azure Sentinel es el almacén de datos; combina consultas de alto rendimiento y esquemas dinámicos, y tiene capacidad para escalar a volúmenes de datos masivos. Azure Portal y todas las herramientas de Azure Sentinel emplean una API común para acceder a este almacén de datos. Esta misma API está disponible también para herramientas externas, como los cuadernos de [Jupyter](https://jupyter.org/) y Python. Mientras que muchas tareas comunes pueden llevarse a cabo en el portal, Jupyter amplía las posibilidades de lo que se puede hacer con estos datos. Combina una programación completa con una vasta colección de bibliotecas pensadas para el análisis de datos, la visualización y Machine Learning. Estas características hacen que Jupyter sea una herramienta fascinante para la búsqueda e investigación relacionadas con cuestiones de seguridad.

![Cuaderno de ejemplo](./media/notebooks/sentinel-notebooks-map.png)

Hemos integrado la experiencia de Jupyter en Azure Portal, lo que facilita el proceso para crear y ejecutar cuadernos para analizar los datos. La biblioteca *Kqlmagic* actúa como el elemento aglutinador que permite realizar consultas desde Azure Sentinel y ejecutarlas directamente en un cuaderno. En las consultas se usa el [lenguaje de consulta Kusto](https://kusto.azurewebsites.net/docs/query/index.html). Azure Sentinel viene con varios cuadernos, desarrollados por algunos de los analistas de seguridad de Microsoft. Algunos de estos cuadernos se crean para escenarios concretos y se pueden usar tal cual; otros se incluyen como ejemplos para ilustrar las técnicas y características que se pueden copiar o adaptar para usarlas en sus propios cuadernos, y también se pueden importar otros desde la comunidad de Azure Sentinel en GitHub.

La experiencia de Jupyter integrada usa [Azure Notebooks](https://notebooks.azure.com/) para almacenar, compartir y ejecutar cuadernos. Estos cuadernos también se pueden ejecutar localmente si tiene Jupyter y un entorno de Python en el equipo o en otros entornos de JupyterHub como Azure Databricks.

Los cuadernos constan de dos componentes:

- La interfaz basada en explorador donde se escriben y ejecutan las consultas y el código, y donde se muestran los resultados de la ejecución.
- Un *kernel*, responsable de analizar y ejecutar el código en sí.

El kernel del cuaderno de Azure Sentinel se ejecuta en una máquina virtual (VM) de Azure. Existen varias opciones de licencia para aprovechar máquinas virtuales más eficaces si los cuadernos incluyen modelos de Machine Learning complejos.

Los cuadernos de Azure Sentinel usan muchas bibliotecas de Python conocidas, como pandas, matplotlib y bokeh, entre otras. Existen otros muchos paquetes de Python que se pueden elegir, que cubren áreas como las siguientes:

- Visualizaciones y gráficos
- Procesamiento de datos y análisis
- Estadísticas y computación numérica
- Aprendizaje automático y aprendizaje profundo

También hemos lanzado una serie de herramientas de seguridad de Jupyter de código abierto en un paquete denominado [msticpy](https://github.com/Microsoft/msticpy/). Este paquete se usa en muchos de los cuadernos incluidos en el producto. Las herramientas de Msticpy están diseñadas expresamente para ayudar a crear cuadernos que se puedan usar en búsquedas e investigaciones, y estamos trabajando activamente en nuevas características y mejoras.

El [repositorio de la comunidad GitHub de Azure Sentinel](https://github.com/Azure/Azure-Sentinel) es la ubicación de los futuros cuadernos de Azure Sentinel creados por Microsoft o procedentes de contribuciones de la comunidad.

Para usar los cuadernos, primero debe crear un área de trabajo de Azure Machine Learning (ML).

## <a name="create-an-azure-ml-workspace"></a>Creación de un área de trabajo de Azure Machine Learning

1. Desde Azure Portal, vaya a **Azure Sentinel** > **Administración de amenazas** > **Cuadernos** y, después, seleccione **Iniciar Notebook**.

    > [!div class="mx-imgBorder"]
    > ![Iniciar Notebook para iniciar el área de trabajo de aprendizaje automático de Azure](./media/notebooks/sentinel-notebooks-launch.png)

1. En **Área de trabajo de Azure ML**, seleccione **Crear nueva**.

    > [!div class="mx-imgBorder"]
    > ![Creación del área de trabajo](./media/notebooks/sentinel-notebooks-azureml-create.png)

1. En la página **Machine Learning**, proporcione la información siguiente y, después, seleccione **Revisar y crear**.

    |Campo|Descripción|
    |--|--|
    |Subscription|Seleccione la suscripción de Azure que quiera usar.|
    |Resource group|Use un grupo de recursos existente en su suscripción o escriba un nombre para crear un nuevo grupo de recursos. Un grupo de recursos almacena los recursos relacionados con una solución de Azure. En este ejemplo se usa **AzureMLRG**.|
    |Nombre del área de trabajo|Escriba un nombre único que identifique el área de trabajo. En este ejemplo se usa **testworkspace1**. Los nombres deben ser únicos en el grupo de recursos. Utilice un nombre que sea fácil de recordar y que se diferencie del de las áreas de trabajo creadas por otros.|
    |Region|Seleccione la ubicación más cercana a los usuarios y los recursos de datos para crear el área de trabajo.|
    |Edición del área de trabajo|Seleccione **Básico** como el tipo de área de trabajo para este ejemplo. El tipo de área de trabajo (Básico o Enterprise) determina las características a las que tendrá acceso y los precios.|

    > [!div class="mx-imgBorder"]
    > ![Detalles del área de trabajo](./media/notebooks/sentinel-notebooks-azureml-basics.png)

1. Revise la información, compruebe que es correcta y, después, seleccione **Crear** para iniciar la implementación del área de trabajo.

    > [!div class="mx-imgBorder"]
    > ![Revisión de los detalles del área de trabajo](./media/notebooks/sentinel-notebooks-azureml-review.png)

    El área de trabajo puede tardar varios minutos en crearse en la nube y, durante ese tiempo, en la página **Información general** se muestra el estado de implementación actual.

    > [!div class="mx-imgBorder"]
    > ![Implementación del área de trabajo](./media/notebooks/sentinel-notebooks-azureml-deploy.png)

Una vez que se haya completado la implementación, puede iniciar cuadernos en la nueva área de trabajo de Azure ML.

> [!div class="mx-imgBorder"]
> ![Implementación correcta del área de trabajo](./media/notebooks/sentinel-notebooks-azureml-complete.png)

## <a name="launch-a-notebook-using-your-azure-ml-workspace"></a>Inicio de un cuaderno con el área de trabajo de Azure ML

1. En Azure Portal, vaya a **Azure Sentinel** > **Threat management (Administración de amenazas)**  > **Notebooks** (Cuadernos), donde puede ver los cuadernos que proporciona Azure Sentinel.

    > [!TIP]
    > Seleccione **Guías y comentarios** para abrir un panel con ayuda e instrucciones adicionales sobre los cuadernos.
    > ![Visualización de guías de cuadernos](./media/notebooks/sentinel-azure-notebooks-guides.png)

1. Seleccione cuadernos individuales para ver sus descripciones, los tipos de datos necesarios y los orígenes de datos.

    > [!div class="mx-imgBorder"]
    > ![Visualización de los detalles del cuaderno](./media/notebooks/sentinel-azure-notebooks-view.png)

1. Seleccione el cuaderno que quiera usar y, después, seleccione **Iniciar Notebook** para clonar y configurar el cuaderno en un nuevo proyecto de Azure Notebooks que se conecta al área de trabajo de Azure Sentinel. Una vez completado el proceso, el cuaderno se abre en Azure Notebooks para que pueda ejecutarlo.

    > [!div class="mx-imgBorder"]
    > ![Selección del cuaderno](./media/notebooks/sentinel-azure-notebooks-select.png)

1. En el área de trabajo AzureML, seleccione el área de trabajo de Azure ML y después **Iniciar**.

    > [!div class="mx-imgBorder"]
    > ![iniciar cuaderno](./media/notebooks/sentinel-azure-notebooks-launch.png)

1. Seleccione una instancia de proceso. Si no tiene una instancia de proceso, siga estos pasos:
    1. Seleccione el signo más (+) para iniciar el asistente **Nueva instancia de proceso**.

        > [!div class="mx-imgBorder"]
        > ![Inicio del asistente para instancia de proceso](./media/notebooks/sentinel-azure-notebooks-compute-wizard.png)

    1. En la página **Nueva instancia de proceso**, proporcione la información necesaria y, después, seleccione **Crear**.

        > [!div class="mx-imgBorder"]
        > ![Creación de la instancia de proceso](./media/notebooks/sentinel-azure-notebooks-compute-create.png)

1. Una vez que se ha creado el servidor de cuaderno, seleccione en cada celda el icono Ejecutar para ejecutar el código en los cuadernos.

    > [!div class="mx-imgBorder"]
    > ![Ejecución del cuaderno](./media/notebooks/sentinel-azure-notebooks-run.png)

Recomendaciones:

- Para obtener una introducción rápida sobre cómo consultar datos en Azure Sentinel, vea la guía [Introducción a cuadernos de Azure ML y Azure Sentinel](https://github.com/Azure/Azure-Sentinel-Notebooks/blob/master/A%20Getting%20Started%20Guide%20For%20Azure%20Sentinel%20ML%20Notebooks.ipynb).

- Encontrará cuadernos de ejemplo adicionales en la subcarpeta [**Sample-Notebooks**](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/Sample-Notebooks) de GitHub. Los cuadernos de ejemplo se han guardado con datos, así será más fácil ver los resultados previstos. Se recomienda ver estos cuadernos en [nbviewer](https://nbviewer.jupyter.org/).

- La carpeta [**HowTos**](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/HowTos) (Procedimientos) de GitHub contiene cuadernos que describen, por ejemplo, lo siguiente: configurar la versión de Python predeterminada, configurar una instancia de DSVM o crear marcadores de Azure Sentinel desde un cuaderno.

Estos cuadernos proporcionados están diseñados como herramientas útiles, pero también como ilustraciones y códigos de ejemplo que los usuarios pueden utilizar para desarrollar sus propios cuadernos.

Agradecemos cualquier comentario, sean sugerencias, peticiones de características, contribuciones de cuadernos, informes de errores o mejoras y adiciones a cuadernos ya existentes. Vaya a la [comunidad GitHub de Azure Sentinel](https://github.com/Azure/Azure-Sentinel) para abrir una incidencia, crear una bifurcación y cargar una contribución.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a usar cuadernos de Jupyter Notebooks en Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- [Búsqueda de amenazas con Azure Sentinel, versión preliminar](hunting.md)
- [Realizar un seguimiento de los datos durante una búsqueda](bookmarks.md)
