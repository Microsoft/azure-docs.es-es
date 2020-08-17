---
title: Creación y administración de recursos con la extensión de VS Code (versión preliminar)
titleSuffix: Azure Machine Learning
description: Creación y administración de recursos con la extensión de VS Code
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.date: 07/09/2020
ms.openlocfilehash: 5ec6af894f434f8d1f7df34b2ed0c2514ab88cc4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87306152"
---
# <a name="manage-azure-machine-learning-resources-with-the-vs-code-extension-preview"></a>Administración de recursos de Azure Machine Learning con la extensión de VS Code (versión preliminar)

Aprenda a administrar los recursos de Azure Machine Learning con la extensión de VS Code.

![Extensión Azure Machine Learning para VS Code](media/how-to-manage-resources-vscode/azure-machine-learning-vscode-extension.png)

## <a name="prerequisites"></a>Requisitos previos

- Suscripción de Azure. Si no tiene una, regístrese para probar la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
- Código de Visual Studio. Si no lo tiene, [instálelo](https://code.visualstudio.com/docs/setup/setup-overview).
- Extensión Azure Machine Learning para VS Code. Siga la [guía de instalación de la extensión Azure Machine Learning para VS Code](tutorial-setup-vscode-extension.md#install-the-extension) para instalar la extensión.

Todos los procesos siguientes suponen que se encuentra en la vista Azure Machine Learning de Visual Studio Code. Para iniciar la extensión, seleccione el icono **Azure** en la barra de actividades de VS Code.

## <a name="workspaces"></a>Áreas de trabajo

Para más información, consulte el artículo sobre las [áreas de trabajo](concept-workspace.md).

### <a name="create-a-workspace"></a>Crear un área de trabajo

1. En la vista Azure Machine Learning, haga clic con el botón derecho en el nodo de suscripción y seleccione **Crear área de trabajo**.
1. En la solicitud:
    1. Escriba un nombre para el área de trabajo.
    1. Elegir la suscripción de Azure
    1. Elija o cree un grupo de recursos donde aprovisionar el área de trabajo.
    1. Seleccione la ubicación donde aprovisionar el área de trabajo.
    1. Elija entre la edición *Basic* y *Enterprise*. Obtenga más información sobre las distintas [ediciones de Azure Machine Learning](concept-editions.md).

Los métodos alternativos para crear un área de trabajo incluyen:

- Hacer clic en el icono `+` que aparece en la parte superior de la vista Azure Machine Learning.
- Crear un área de trabajo nueva cuando se le pida que seleccione un área de trabajo durante el aprovisionamiento de otros recursos.

### <a name="remove-a-workspace"></a>Eliminación de un área de trabajo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Haga clic con el botón derecho en el área de trabajo que quiere quitar.
1. Seleccione si quiere quitar:
    - *Solo el área de trabajo*: esta opción **solo** elimina el recurso de área de trabajo de Azure. El grupo de recursos, las cuentas de almacenamiento y otros recursos a los que se adjuntó el área de trabajo siguen en Azure.
    - *Con recursos asociados*: esta opción elimina el área de trabajo **y** todos los recursos que tiene asociados.

## <a name="datastores"></a>Almacenes de datos

Actualmente, la extensión de VS Code admite estos tipos de almacenes de datos:

- Recurso compartido de archivos de Azure
- Azure Blob Storage

Al crear un área de trabajo, se crea un almacén de datos para cada uno de estos tipos.

Para más información, consulte la sección sobre los [almacenes de datos](concept-data.md#datastores).

### <a name="create-a-datastore"></a>Creación de un almacén de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde quiere crear el almacén de datos.
1. Haga clic con el botón derecho en el nodo **Almacenes de datos** y seleccione **Register datastore** (Registrar almacén de datos).
1. En la solicitud:
    1. Escriba un nombre para el almacén de datos.
    1. Elija el tipo de almacén de datos.
    1. Seleccione el recurso de almacenamiento. Puede elegir un recurso de almacenamiento asociado con el área de trabajo o seleccionar uno de los recursos de almacenamiento válidos en las suscripciones de Azure.
    1. Elija el contenedor en el que se encuentran los datos en el recurso de almacenamiento seleccionado anteriormente.
1. Un archivo de configuración aparece en VS Code. Si está satisfecho con el archivo de configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

### <a name="manage-a-datastore"></a>Administración de un almacén de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Almacenes de datos** dentro del área de trabajo.
1. Seleccione el almacén de datos en el que quiere hacer lo siguiente:
    - *Establecer como predeterminado*. Este será el almacén de datos que se usará cada vez que ejecute un experimento.
    - *Inspeccionar la configuración de solo lectura*.
    - *Modificar*. Cambie el tipo de autenticación y las credenciales. Entre los tipos de autenticación admitidos se incluyen la clave de cuenta y el token de SAS.

## <a name="datasets"></a>Conjuntos de datos

Actualmente, la extensión admite estos tipos de conjunto de datos:

- *Tabular*: permite materializar los datos en una estructura DataFrame (Pandas o PySpark).
- *Archivo*: un archivo o una colección de archivos. Permite descargar o montar archivos en el proceso.

Para más información, consulte la sección sobre [conjuntos de datos](concept-data.md#datasets).

### <a name="create-dataset"></a>Creación de un conjunto de datos

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde quiere crear el almacén de datos.
1. Haga clic con el botón derecho en el nodo **Conjuntos de datos** y seleccione **Crear conjunto de datos**.
1. En la solicitud:
    1. Elija el tipo de conjunto de datos.
    1. Defina si los datos están ubicados en el equipo o en la Web.
    1. Indique la ubicación de los datos. Puede ser un solo archivo o un directorio que contenga los archivos de datos.
    1. Elija el almacén de datos al que quiere cargar los datos.
    1. Proporcione un prefijo que ayude a identificar el conjunto de datos en el almacén de datos.

### <a name="version-datasets"></a>Conjuntos de datos de versión

Al crear modelos de aprendizaje automático, a medida que cambian los datos, puede que quiera crear una versión del conjunto de datos. Para ello, en la extensión de VS Code:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Conjuntos de datos**.
1. Haga clic con el botón derecho en el conjunto de datos cuya versión quiere crear y seleccione **Crear nueva versión**.
1. En la solicitud:
    1. Seleccione el tipo de conjunto de datos.
    1. Defina si los datos están ubicados en el equipo o en la Web.
    1. Indique la ubicación de los datos. Puede ser un solo archivo o un directorio que contenga los archivos de datos.
    1. Elija el almacén de datos al que quiere cargar los datos.
    1. Proporcione un prefijo que ayude a identificar el conjunto de datos en el almacén de datos.

### <a name="view-dataset-properties"></a>Visualización de las propiedades del conjunto de datos

Esta opción permite ver los metadatos asociados a un conjunto de datos específico. Para ello, en la extensión de VS Code:

1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Conjuntos de datos**.
1. Haga clic con el botón derecho en el conjunto de datos que quiere inspeccionar y seleccione **View Dataset Properties** (Ver propiedades del conjunto de datos). Se mostrará un archivo de configuración con las propiedades de la versión del conjunto de datos más reciente.

> [!NOTE]
> Si tiene varias versiones del conjunto de datos, puede optar por ver solo las propiedades del conjunto de datos de una versión específica al expandir el nodo del conjunto de datos y realizar los mismos pasos descrito en esta sección sobre la versión correspondiente.

### <a name="unregister-datasets"></a>Anulación del registro de los conjuntos de datos

Para quitar un conjunto de datos y todas sus versiones, anule su registro. Para ello, en la extensión de VS Code:

1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Conjuntos de datos**.
1. Haga clic con el botón derecho en el conjunto de datos cuyo registro quiere anular y seleccione **Unregister dataset** (Anular registro del conjunto de datos).

## <a name="environments"></a>Entornos

Para más información, consulte la sección sobre los [entornos](concept-environments.md).

### <a name="create-environment"></a>Creación del entorno

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde quiere crear el almacén de datos.
1. Haga clic con el botón derecho en el nodo **Entornos** y seleccione **Crear entorno**.
1. En la solicitud:
    1. Proporcione un nombre para el entorno.
    1. Defina la configuración del entorno:
        - *Entornos mantenidos*: entornos configurados previamente en Azure Machine Learning. Para personalizar aún más el entorno, modifique la propiedad `dependencies` en el archivo JSON. Obtenga más información sobre los [entornos mantenidos](resource-curated-environments.md).
        - *Archivo de dependencias de Conda*: en los entornos Anaconda, se puede proporcionar el archivo que contiene la definición del entorno.
        - *Archivo de requisitos de pip*: en los entornos de pip, se puede proporcionar el archivo que contiene la definición del entorno.
        - *Entorno de Conda existente*: esta opción busca los entornos de Conda en el equipo local e intenta crear un entorno desde el entorno seleccionado.
        - *Personalizado*: defina sus propios canales y dependencias.
    1. Se abrirá un archivo de configuración en el editor. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

### <a name="view-environment-configurations"></a>Visualización de configuraciones de entorno

Para ver las dependencias y configuraciones de un entorno específico en la extensión:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo **Entornos**.
1. Haga clic con el botón derecho en el entorno que quiere ver y seleccione **Ver entorno**.

### <a name="edit-environment-configurations"></a>Edición de configuraciones de entorno

Para editar las dependencias y configuraciones de un entorno específico en la extensión:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Entornos** en el área de trabajo.
1. Haga clic con el botón derecho en el entorno que quiere ver y seleccione **Editar entorno**.
1. Después de realizar las modificaciones, si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

## <a name="experiments"></a>Experimentos

Para más información, consulte la sección sobre los [experimentos](concept-azure-machine-learning-architecture.md#experiments).

### <a name="create-experiment"></a>Crear un experimento

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Experimentos** en el área de trabajo y seleccione **Crear experimento**.
1. En la solicitud, escriba un nombre para el experimento.

### <a name="run-experiment"></a>Ejecutar experimento

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Haga clic con el botón derecho en el experimento que quiere ejecutar.
1. Seleccione el icono **Ejecutar experimento** en la barra de actividades.
1. Elija su suscripción.
1. Elija el área de trabajo de Azure ML donde ejecutar el experimento.
1. Elija el experimento.
1. Elija o cree un proceso donde ejecutar el experimento.
1. Elija o cree una configuración de ejecución para el experimento.

También puede seleccionar el botón **Ejecutar experimento** en la parte superior de la extensión y configurar la ejecución del experimento en la solicitud.

### <a name="view-experiment"></a>Visualización de un experimento

Para ver el experimento en Azure Machine Learning Studio:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Haga clic con el botón derecho en el experimento que quiere ver y seleccione **Ver experimento**. 
1. Aparece una solicitud para que abra la dirección URL del experimento en Azure Machine Learning Studio. seleccione **Open**(Abrir).

### <a name="track-run-progress"></a>Seguimiento del progreso de la ejecución

Mientras ejecuta el experimento, puede que quiera ver el progreso. Para hacer un seguimiento del progreso de una ejecución en Azure Machine Learning Studio desde la extensión:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Expanda el nodo del experimento para el que quiere hacer el seguimiento.
1. Haga clic con el botón derecho en la ejecución y seleccione **View Run in Azure portal** (Ver ejecución en Azure Portal).
1. Aparece una solicitud para que abra la dirección URL de la ejecución en Azure Machine Learning Studio. seleccione **Open**(Abrir).

### <a name="download-run-logs--outputs"></a>Descarga de registros y salidas de la ejecución

Una vez que se completa una ejecución, puede que quiera descargar los registros y recursos, como el modelo generado como parte de la ejecución de un experimento.

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Experimentos** en el área de trabajo.
1. Expanda el nodo del experimento para el que quiere hacer el seguimiento.
1. Haga clic con el botón derecho en la ejecución:
    - Para descargar las salidas, seleccione **Download outputs** (Descargar salidas).
    - Para descargar los registros, seleccione **Download logs** (Descargar registros).

### <a name="view-run-metadata"></a>Visualización de metadatos de la ejecución

En la extensión, puede inspeccionar los metadatos, como la configuración de ejecución que se usa para la ejecución, así como los detalles de la ejecución.

## <a name="compute-clusters"></a>Clústeres de proceso

La extensión admite estos tipos de proceso:

- Clúster de proceso de Azure Machine Learning
- Azure Kubernetes Service

Para más información, consulte la sección sobre los [destinos de proceso](concept-compute-target.md#train).

### <a name="create-compute"></a>Creación del proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo donde quiere crear el clúster de proceso.
1. Haga clic con el botón derecho en el nodo **Clústeres de proceso** y seleccione **Crear proceso**.
1. En la solicitud:
    1. Elija un tipo de proceso.
    1. Elija el tamaño de la máquina virtual. Obtenga más información sobre los [tamaños de VM](https://docs.microsoft.com/azure/virtual-machines/linux/sizes).
    1. Proporcione un nombre para el proceso.

### <a name="view-compute-configuration"></a>Vista de la configuración de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Clústeres de proceso** en el área de trabajo.
1. Haga clic con el botón derecho en el proceso que quiere ver y seleccione **View Compute Properties** (Ver propiedades del proceso).

### <a name="edit-compute-scale-settings"></a>Edición de configuración de escalado de proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Clústeres de proceso** en el área de trabajo.
1. Haga clic con el botón derecho en el proceso que quiere editar y seleccione **Editar proceso**.
1. Se abre un archivo de configuración para el proceso en el editor. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

### <a name="delete-compute"></a>Eliminación del proceso

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Clústeres de proceso** en el área de trabajo.
1. Haga clic con el botón derecho en el proceso que quiere eliminar y seleccione **Eliminar proceso**.

### <a name="create-run-configuration"></a>Creación de una configuración de ejecución

Para crear una configuración de ejecución en la extensión:

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Clústeres de proceso** en el área de trabajo.
1. Haga clic con el botón derecho en el destino de proceso donde quiere crear la configuración de ejecución y seleccione **Crear configuración de ejecución**.
1. En la solicitud:
    1. Proporcione un nombre para el destino de proceso.
    1. Elija o cree un entorno nuevo.
    1. Escriba el nombre del script que quiere ejecutar o presione **ENTRAR** para buscar el script en la máquina local.
    1. (Opcional) Elija si quiere crear una referencia de datos para la ejecución de entrenamiento. Si lo hace, se le pedirá que defina un conjunto de datos en la configuración de ejecución.
        1. Seleccione uno de los conjuntos de datos registrados para vincularlo con la configuración de ejecución. Se abre un archivo de configuración para el conjunto de datos en el editor. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.
    1. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

### <a name="edit-run-configuration"></a>Edición de una configuración de ejecución

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo de clúster de proceso en el nodo **Clústeres de proceso** del área de trabajo.
1. Haga clic con el botón derecho en la configuración de ejecución que quiere editar y seleccione **Editar configuración de ejecución**.
1. Se abre un archivo de configuración para la configuración de ejecución en el editor. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

### <a name="delete-run-configuration"></a>Eliminación de la configuración de ejecución

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Expanda el nodo de clúster de proceso de interés dentro del nodo **Clústeres de proceso**.
1. Haga clic con el botón derecho en la configuración de ejecución que quiere editar y seleccione **Delete Run Configuration** (Eliminar configuración de ejecución).

## <a name="models"></a>Modelos

Para más información, consulte la sección sobre los [modelos](concept-azure-machine-learning-architecture.md#models).

### <a name="register-model"></a>Registro del modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Modelos** y seleccione **Register Model** (Registrar modelo).
1. En la solicitud:
    1. Proporcione un nombre para el modelo.
    1. Elija si el modelo es un archivo o una carpeta.
    1. Busque el modelo en el equipo local.
    1. Un archivo de configuración para el modelo en el editor. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

### <a name="view-model-properties"></a>Visualización de las propiedades del modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Modelos** en el área de trabajo.
1. Haga clic con el botón derecho en el modelo cuyas propiedades quiere ver y seleccione **View Model Properties** (Ver propiedades del modelo). En el editor se abre un archivo que contiene las propiedades del modelo.

### <a name="download-model"></a>Descarga del modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Modelos** en el área de trabajo.
1. Haga clic con el botón derecho en el modelo que va a descargar y seleccione **Download Model File** (Descargar archivo del modelo).

### <a name="delete-a-model"></a>Eliminación de un modelo

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Modelos** en el área de trabajo.
1. Haga clic con el botón derecho en el modelo que quiere eliminar y seleccione **Eliminar modelo**.

## <a name="endpoints"></a>Puntos de conexión

La extensión de VS Code admite estos destinos de implementación:

- Azure Container Instances
- Azure Kubernetes Service

Para más información, consulte la sección sobre los [puntos de conexión de servicio web](concept-azure-machine-learning-architecture.md#web-service-endpoint).

### <a name="create-deployments"></a>Creación de implementaciones

> [!NOTE]
> Actualmente, la creación de la implementación solo funciona con entornos de Conda.

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo del área de trabajo.
1. Haga clic con el botón derecho en el nodo **Puntos de conexión** y seleccione **Implementar servicio**.
1. En la solicitud:
    1. Elija si quiere usar un modelo ya registrado o un archivo de modelo local.
    1. Selección del modelo
    1. Elija el destino de implementación donde quiere implementar el modelo.
    1. Proporcione un nombre para el modelo.
    1. Proporcione el script que se ejecutará al puntuar el modelo.
    1. Proporcione un archivo de dependencias de Conda.
    1. En el editor aparece un archivo de configuración para la implementación. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.

> [!NOTE]
> También puede hacer clic con el botón derecho en un modelo registrado en el nodo *Modelos* y seleccionar **Deploy Service From Registered Model** (Implementar el servicio a partir del modelo registrado).

### <a name="delete-deployments"></a>Eliminación de implementaciones

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Puntos de conexión** en el área de trabajo.
1. Haga clic con el botón derecho en la implementación que quiere quitar y seleccione **Quitar servicio**.
1. Aparece un mensaje para confirmar que quiere quitar el servicio. Seleccione **Aceptar**.

### <a name="manage-deployments"></a>Administración de implementaciones

Además de crear y eliminar implementaciones, también puede ver y editar la configuración asociada con la implementación.

1. Expanda el nodo de suscripción que contiene el área de trabajo.
1. Expanda el nodo **Puntos de conexión** en el área de trabajo.
1. Haga clic con el botón derecho en la implementación que quiere administrar:
    - Para editar la configuración, seleccione **Editar servicio**.
        - En el editor aparece un archivo de configuración para la implementación. Si está satisfecho con la configuración, seleccione **Guardar y continuar** o abra la paleta de comandos de VS Code (**Ver > Paleta de comandos**) y escriba **Azure ML: Guardar y continuar**.
    - Para ver la configuración de implementación, seleccione **View service properties** (Ver propiedades del servicio).

## <a name="next-steps"></a>Pasos siguientes

[Entrenamiento de un modelo de clasificación de imágenes](tutorial-train-deploy-image-classification-model-vscode.md) con la extensión de VS Code.