---
title: Ejecución de una carga de trabajo en paralelo mediante la API para Python
description: 'Tutorial: Procesamiento de archivos multimedia en paralelo con ffmpeg en Azure Batch con la biblioteca cliente de Python de Batch'
ms.devlang: python
ms.topic: tutorial
ms.date: 11/29/2018
ms.custom: mvc, devx-track-python
ms.openlocfilehash: 34cea5e0b60c0a7ee8c3d31dd02a6f7e975de738
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853337"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-python-api"></a>Tutorial: Ejecución de una carga de trabajo paralela con Azure Batch mediante Python API

Use Azure Batch para ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala, de manera eficaz en Azure. Este tutorial le guía a través de un ejemplo de Python de ejecución de carga de trabajo paralela con Batch. Aprenderá lo que es un flujo de trabajo de aplicación de Batch común y a interactuar con los recursos de Batch y Storage mediante programación. Aprenderá a:

> [!div class="checklist"]
> * Autenticarse en las cuentas de Batch y Storage
> * Cargar archivos de entrada en Storage
> * Crear un grupo de nodos de proceso para ejecutar una aplicación
> * Crear un trabajo y tareas para procesar archivos de entrada
> * Supervisar la ejecución de las tareas
> * Recuperación de archivos de salida

En este tutorial, convertiremos archivos multimedia MP4 a formato MP3 en paralelo con la herramienta de código de abierto [ffmpeg](https://ffmpeg.org/). 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* [Python 2.7 o 3.3, o cualquier versión posterior](https://www.python.org/downloads/)

* Administrador de paquetes [pip](https://pip.pypa.io/en/stable/installing/)

* Una cuenta de Azure Batch y una cuenta de Azure Storage vinculada. Para crear estas cuentas, consulte las guías de inicio rápido de Batch con [Azure Portal](quick-create-portal.md) o la [CLI de Azure](quick-create-cli.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)] 

## <a name="download-and-run-the-sample"></a>Descarga y ejecución del ejemplo

### <a name="download-the-sample"></a>Descarga del ejemplo

[Descargue o clone la aplicación de ejemplo](https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial) desde GitHub. Para clonar el repositorio de la aplicación de ejemplo con un cliente de Git, use el siguiente comando:

```
git clone https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial.git
```

Vaya al directorio que contiene el archivo `batch_python_tutorial_ffmpeg.py`.

En el entorno de Python, instale los paquetes necesarios mediante `pip`.

```bash
pip install -r requirements.txt
```

Abra el archivo `config.py`. Actualice las cadenas de credenciales de la cuenta de Storage y de Batch con los valores únicos correspondientes. Por ejemplo:


```Python
_BATCH_ACCOUNT_NAME = 'mybatchaccount'
_BATCH_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ=='
_BATCH_ACCOUNT_URL = 'https://mybatchaccount.mybatchregion.batch.azure.com'
_STORAGE_ACCOUNT_NAME = 'mystorageaccount'
_STORAGE_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=='
```

### <a name="run-the-app"></a>Ejecución la aplicación

Para ejecutar el script:

```
python batch_python_tutorial_ffmpeg.py
```

Al ejecutar la aplicación de ejemplo, la salida de la consola es similar a la siguiente. Durante la ejecución, se experimenta una pausa en `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` mientras se inician los nodos de proceso del grupo. 
   
```
Sample start: 11/28/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [LinuxFFmpegPool]...
Creating job [LinuxFFmpegJob]...
Adding 5 tasks to job [LinuxFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]....

Sample end: 11/28/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

Vaya a la cuenta de Batch de Azure Portal para supervisar el grupo, los nodos de proceso, el trabajo y las tareas. Por ejemplo, para ver un mapa térmico de los nodos de proceso del grupo, haga clic en **Grupos** > *LinuxFFmpegPool*.

Con las tareas en ejecución, el mapa térmico es similar al siguiente:

![Mapa térmico de grupo](./media/tutorial-parallel-python/pool.png)

El tiempo de ejecución es de aproximadamente **5 minutos** al ejecutar la aplicación con la configuración predeterminada. La creación del grupo es lo que más tarda. 

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>Revisión del código

En las secciones siguientes se desglosan los pasos que lleva a cabo la aplicación de ejemplo para procesar una carga de trabajo en el servicio Batch. Consulte el código de Python mientras lee el resto de este artículo, ya que no se explican todas las líneas de código del ejemplo.

### <a name="authenticate-blob-and-batch-clients"></a>Autenticación de los clientes de Blob y Batch

Para interactuar con una cuenta de almacenamiento, la aplicación usa el paquete [azure-storage-blob](https://pypi.python.org/pypi/azure-storage-blob) para crear un objeto [BlockBlobService](/python/api/azure-storage-blob/azure.storage.blob.blockblobservice.blockblobservice).

```python
blob_client = azureblob.BlockBlobService(
    account_name=_STORAGE_ACCOUNT_NAME,
    account_key=_STORAGE_ACCOUNT_KEY)
```

La aplicación crea un objeto [BatchServiceClient](/python/api/azure.batch.batchserviceclient) para crear y administrar los grupos, los trabajos y las tareas en el servicio Batch. El cliente de Batch del ejemplo utiliza la autenticación de clave compartida. Batch también admite la autenticación a través de [Azure Active Directory](batch-aad-auth.md) para usuarios individuales o una aplicaciones desatendidas.

```python
credentials = batchauth.SharedKeyCredentials(_BATCH_ACCOUNT_NAME,
                                             _BATCH_ACCOUNT_KEY)

batch_client = batch.BatchServiceClient(
    credentials,
    base_url=_BATCH_ACCOUNT_URL)
```

### <a name="upload-input-files"></a>Carga de archivos de entrada

La aplicación usa la referencia a `blob_client` para crear un contenedor de almacenamiento para los archivos MP4 de entrada y uno para la salida de la tarea. A continuación, llama a la función `upload_file_to_container` para cargar los archivos MP4 del directorio `InputFiles` local en el contenedor. Los archivos de almacenamiento se definen como objetos [ResourceFile](/python/api/azure-batch/azure.batch.models.resourcefile) de Batch para que el servicio los descargue después en nodos de proceso.

```python
blob_client.create_container(input_container_name, fail_on_exist=False)
blob_client.create_container(output_container_name, fail_on_exist=False)
input_file_paths = []

for folder, subs, files in os.walk(os.path.join(sys.path[0], './InputFiles/')):
    for filename in files:
        if filename.endswith(".mp4"):
            input_file_paths.append(os.path.abspath(
                os.path.join(folder, filename)))

# Upload the input files. This is the collection of files that are to be processed by the tasks.
input_files = [
    upload_file_to_container(blob_client, input_container_name, file_path)
    for file_path in input_file_paths]
```

### <a name="create-a-pool-of-compute-nodes"></a>Creación de un grupo de nodos de proceso

Después, en el ejemplo se crea un grupo de nodos de proceso en la cuenta de Batch con una llamada a `create_pool`. Esta función definida usa la clase [PoolAddParameter](/python/api/azure-batch/azure.batch.models.pooladdparameter) de Batch para establecer el número de nodos, el tamaño de la máquina virtual y una configuración de grupo. En este caso, un objeto [VirtualMachineConfiguration](/python/api/azure-batch/azure.batch.models.virtualmachineconfiguration) especifica un valor de [ImageReference](/python/api/azure-batch/azure.batch.models.imagereference) a una imagen de Ubuntu Server 18.04 LTS publicada en Azure Marketplace. Batch es compatible con una amplia gama de imágenes de máquina virtual de Azure Marketplace, así como con las imágenes de máquina virtual personalizadas.

El número de nodos y el tamaño de la máquina virtual se establecen mediante constantes definidas. Batch admite nodos especializados y [nodos de prioridad baja](batch-low-pri-vms.md), y en los grupos puede utilizar ambos. Los nodos dedicados están reservados para el grupo. Los nodos de prioridad baja se ofrecen a precio reducido por la capacidad sobrante de las máquinas virtuales de Azure. Los nodos de prioridad baja dejan de estar disponibles si Azure no tiene capacidad suficiente. En el ejemplo, de forma predeterminada se crea un grupo que contiene solo 5 nodos de baja prioridad con el tamaño *Standard_A1_v2*. 

Además de las propiedades de nodo físico, esta configuración de grupo incluye un objeto [StartTask](/python/api/azure-batch/azure.batch.models.starttask). StartTask se ejecutará en cada nodo cuando este se una al grupo, así como cada vez que se reinicie. En este ejemplo, StartTask ejecuta comandos de shell de Bash para instalar el paquete de ffmpeg y las dependencias en los nodos.

El método [pool.add](/python/api/azure-batch/azure.batch.operations.pooloperations) envía el grupo al servicio Batch.

```python
new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=batchmodels.ImageReference(
            publisher="Canonical",
            offer="UbuntuServer",
            sku="18.04-LTS",
            version="latest"
        ),
        node_agent_sku_id="batch.node.ubuntu 18.04"),
    vm_size=_POOL_VM_SIZE,
    target_dedicated_nodes=_DEDICATED_POOL_NODE_COUNT,
    target_low_priority_nodes=_LOW_PRIORITY_POOL_NODE_COUNT,
    start_task=batchmodels.StartTask(
        command_line="/bin/bash -c \"apt-get update && apt-get install -y ffmpeg\"",
        wait_for_success=True,
        user_identity=batchmodels.UserIdentity(
            auto_user=batchmodels.AutoUserSpecification(
                scope=batchmodels.AutoUserScope.pool,
                elevation_level=batchmodels.ElevationLevel.admin)),
    )
)
batch_service_client.pool.add(new_pool)
```

### <a name="create-a-job"></a>Creación de un trabajo

Un trabajo de Batch especifica un grupo en el que ejecutar tareas y valores de configuración opcionales, como la prioridad y la programación del trabajo. En el ejemplo se crea un trabajo con una llamada a `create_job`. Esta función definida usa la clase [JobAddParameter](/python/api/azure-batch/azure.batch.models.jobaddparameter) para crear un trabajo en el grupo. El método [job.add](/python/api/azure-batch/azure.batch.operations.joboperations) envía el grupo al servicio Batch. Inicialmente, el trabajo no tiene tareas.

```python
job = batch.models.JobAddParameter(
    id=job_id,
    pool_info=batch.models.PoolInformation(pool_id=pool_id))

batch_service_client.job.add(job)
```

### <a name="create-tasks"></a>Creación de tareas

La aplicación crea tareas en el trabajo con una llamada a `add_tasks`. Esta función definida crea una lista de objetos de tarea mediante la clase [TaskAddParameter](/python/api/azure-batch/azure.batch.models.taskaddparameter). Las tareas ejecutan ffmpeg para procesar un objeto `resource_files` de entrada con un parámetro `command_line`. Al crear el grupo, ffmpeg se instaló previamente en todos los nodos. En este caso, la línea de comandos ejecuta ffmpeg para convertir los archivos MP4 de entrada (vídeo) en archivos MP3 (audio).

En el ejemplo se crea un objeto [OutputFile](/python/api/azure-batch/azure.batch.models.outputfile) para el archivo MP3 después de ejecutar la línea de comandos. Los archivos de salida de la tarea (en este caso, uno) se cargan en un contenedor en la cuenta de Storage vinculada mediante la propiedad `output_files` de la tarea.

A continuación, la aplicación agrega tareas al trabajo con el método [task.add_collection](/python/api/azure-batch/azure.batch.operations.taskoperations), que las pone en cola para que se ejecuten en los nodos de proceso. 

```python
tasks = list()

for idx, input_file in enumerate(input_files):
    input_file_path = input_file.file_path
    output_file_path = "".join((input_file_path).split('.')[:-1]) + '.mp3'
    command = "/bin/bash -c \"ffmpeg -i {} {} \"".format(
        input_file_path, output_file_path)
    tasks.append(batch.models.TaskAddParameter(
        id='Task{}'.format(idx),
        command_line=command,
        resource_files=[input_file],
        output_files=[batchmodels.OutputFile(
            file_pattern=output_file_path,
            destination=batchmodels.OutputFileDestination(
                container=batchmodels.OutputFileBlobContainerDestination(
                    container_url=output_container_sas_url)),
            upload_options=batchmodels.OutputFileUploadOptions(
                upload_condition=batchmodels.OutputFileUploadCondition.task_success))]
    )
    )
batch_service_client.task.add_collection(job_id, tasks)
```

### <a name="monitor-tasks"></a>Supervisión de las tareas

Al agregar tareas a un trabajo, Batch las pone en cola automáticamente y las programa para que se ejecuten en nodos de proceso del grupo asociado. Según la configuración que especifique, Batch controla la administración de las colas, la programación, los reintentos y otras labores de administración de tareas. 

Existen varios enfoques para supervisar la ejecución de tareas. La función `wait_for_tasks_to_complete` de este ejemplo usa el objeto [TaskState](/python/api/azure-batch/azure.batch.models.taskstate) para supervisar un estado concreto de las tareas, en este caso, el estado completado, en un tiempo determinado.

```python
while datetime.datetime.now() < timeout_expiration:
    print('.', end='')
    sys.stdout.flush()
    tasks = batch_service_client.task.list(job_id)

    incomplete_tasks = [task for task in tasks if
                        task.state != batchmodels.TaskState.completed]
    if not incomplete_tasks:
        print()
        return True
    else:
        time.sleep(1)
...
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de ejecutar las tareas, la aplicación elimina automáticamente el contenedor de almacenamiento de entrada que creó y ofrece la opción de eliminar el grupo y el trabajo de Batch. Las clases [JobOperations](/python/api/azure-batch/azure.batch.operations.joboperations) y [Pool Operations](/python/api/azure-batch/azure.batch.operations.pooloperations) tienen sus métodos de eliminación correspondientes, a los que se llama si el usuario confirma la eliminación. Aunque no se cobran los trabajos y las tareas, sí se cobran los nodos de proceso. Por consiguiente, se recomienda asignar solo los grupos necesarios. Al eliminar el grupo, las salidas de tarea de los nodos también se eliminan. Sin embargo, los archivos de entrada y salida permanecen en la cuenta de Storage.

Cuando ya no los necesite, elimine el grupo de recursos, la cuenta de Batch y la de Storage. Para hacerlo desde Azure Portal, seleccione el grupo de recursos de la cuenta de Batch y haga clic en **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Autenticarse en las cuentas de Batch y Storage
> * Cargar archivos de entrada en Storage
> * Crear un grupo de nodos de proceso para ejecutar una aplicación
> * Crear un trabajo y tareas para procesar archivos de entrada
> * Supervisar la ejecución de las tareas
> * Recuperación de archivos de salida

Para más ejemplos de uso de la API de Python para programar y procesar cargas de trabajo de Batch, consulte los ejemplos de GitHub.

> [!div class="nextstepaction"]
> [Ejemplos de Python de Batch](https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch)

