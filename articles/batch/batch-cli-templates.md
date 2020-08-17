---
title: Ejecución de trabajos de un extremo a otro mediante plantillas
description: Con solo los comandos de CLI, puede crear un grupo, cargar datos de entrada, crear trabajos y tareas asociadas y descargar los datos de salida resultantes.
ms.topic: how-to
ms.date: 12/07/2018
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: e9b4930dee1b28bb4ec71690cbfcef88e0365b9e
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494934"
---
# <a name="use-azure-batch-cli-templates-and-file-transfer"></a>Uso de plantillas y transferencia de archivos de la CLI de Azure Batch

Con una extensión de Batch para la CLI de Azure, es posible ejecutar trabajos de Batch sin escribir código.

Cree y use archivos de plantilla JSON con la CLI de Azure para crear grupos, trabajos y tareas de Batch. Use comandos de la CLI para cargar fácilmente archivos de entrada de trabajos en la cuenta de almacenamiento asociada con la cuenta de Batch, y descargue archivos de salida de trabajos.

> [!NOTE]
> Los archivos JSON no admiten la misma funcionalidad que las [plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md). Están diseñados para que tengan el formato del cuerpo de la solicitud de REST sin procesar. La extensión de la CLI no cambia ningún comando existente, pero tiene una opción de plantilla similar que agrega funcionalidad de plantilla de Azure Resource Manager parcial. Consulte [Extensiones de la CLI de Azure Batch para Windows, Mac y Linux](https://github.com/Azure/azure-batch-cli-extensions).

## <a name="overview"></a>Información general

Una extensión de la CLI de Azure permite que los usuarios que no son desarrolladores usen Batch de un extremo a otro. Con solo los comandos de CLI, puede crear un grupo, cargar datos de entrada, crear trabajos y tareas asociadas y descargar los datos de salida resultantes. No se requiere ningún código adicional. Ejecute los comandos de la CLI directamente o intégrelos en los scripts.

Las plantillas de Batch se basan en el [soporte de Batch existente en la CLI de Azure](batch-cli-get-started.md#json-files-for-resource-creation) para que los archivos JSON especifiquen los valores de propiedad para la creación de grupos, trabajos, tareas y otros elementos. Las plantillas de Batch agregan las siguientes funcionalidades:

-   Se pueden definir los parámetros. Cuando se usa la plantilla, solo se especifican los valores de parámetro para crear el elemento, junto con otros valores de propiedad del elemento especificados en el cuerpo de la plantilla. Un usuario que entiende Batch y las aplicaciones que Batch ejecuta puede crear plantillas, especificar grupos, trabajos y valores de propiedad de la tarea. Un usuario menos familiarizado con Batch y/o las aplicaciones solo tiene que especificar los valores para los parámetros definidos.

-   Los generadores de tareas de trabajo crean una o varias tareas asociadas a un trabajo, evitando la necesidad de crear muchas definiciones de tareas y simplificando de manera significativa el envío de trabajos.


Los trabajos normalmente usan archivos de datos de entrada y generan archivos de datos de salida. Una cuenta de almacenamiento se asocia, de manera predeterminada, con cada cuenta de Batch. Transfiera archivos a y desde esta cuenta de almacenamiento mediante la CLI, sin codificación ni credenciales de almacenamiento.

Por ejemplo, [ffmpeg](https://ffmpeg.org/) es una aplicación popular que procesa archivos de audio y video. Estos son los pasos para usar la CLI de Azure Batch a fin de invocar ffmpeg para transcodificar los archivos de vídeo de origen para diferentes resoluciones.

-   Cree una plantilla de grupos. El usuario que crea la plantilla sabe cómo llamar a la aplicación ffmpeg y sus requisitos. Especifica el sistema operativo adecuado, el tamaño de la máquina virtual, cómo está instalado ffmpeg (por ejemplo, de un paquete de aplicación o mediante un administrador de paquetes) y otros valores de propiedad del grupo. Los parámetros se crean para que cuando se use la plantilla, solo deban especificarse el identificador del grupo y el número de máquinas virtuales.

-   Cree una plantilla de trabajos. El usuario que crea la plantilla sabe cómo debe invocarse ffmpeg para transcodificar el vídeo de origen a una resolución diferente y especifica la línea de comandos de la tarea. También sabe que hay una carpeta que contiene los archivos de vídeo de origen, con una tarea obligatoria por cada archivo de entrada.

-   Si un usuario final necesita transcodificar un conjunto de archivos de vídeo, primero crea un grupo con la plantilla de grupo, donde solo se especifican el identificador del grupo y el número de máquinas virtuales necesarias. A continuación, puede cargar los archivos de origen para transcodificarlos. Luego, se puede enviar un trabajo mediante la plantilla de trabajo, especificando solo el identificador del grupo y la ubicación de los archivos de origen cargados. Se crea el trabajo de Batch, con una tarea por cada archivo de entrada que se esté generando. Por último, se pueden descargar los archivos de salida transcodificados.

## <a name="installation"></a>Instalación

Para instalar la extensión de la CLI de Azure Batch, primero [instale la CLI de Azure 2.0](/cli/azure/install-azure-cli) o ejecute la CLI de Azure en [Azure Cloud Shell](../cloud-shell/overview.md).

Instale la versión más reciente de la extensión de Batch mediante el siguiente comando de la CLI de Azure:

```azurecli
az extension add --name azure-batch-cli-extensions
```

Para más información sobre la extensión de la CLI de Batch y opciones de instalación adicionales, visite el [repositorio de GitHub](https://github.com/Azure/azure-batch-cli-extensions).


Para usar las características de la extensión de la CLI, necesita una cuenta de Azure Batch y una cuenta de almacenamiento vinculada para los comandos que transfieren archivos a y desde el almacenamiento.

Para iniciar sesión en una cuenta de Batch con la CLI de Azure, consulte [Administración de recursos de Batch con la CLI de Azure](batch-cli-get-started.md).

## <a name="templates"></a>Plantillas

Las plantillas de Azure Batch son similares a las plantillas de Azure Resource Manager, en la funcionalidad y la sintaxis. Son archivos JSON que contienen los nombres y valores de la propiedad del elemento, pero agregan los siguientes conceptos principales:

-   **Parámetros**

    -   Permiten que se especifiquen valores de propiedad en una sección del cuerpo, donde solo es necesario proporcionar los valores de parámetro cuando se usa la plantilla. Por ejemplo, se podría colocar la definición completa de un grupo en el cuerpo y solo un parámetro definido para `poolId`. Por lo tanto, solo tiene que proporcionarse una cadena del identificador del grupo para crearlo.
        
    -   El cuerpo de la plantilla lo puede crear alguien con conocimientos de Batch y las aplicaciones que Batch ejecuta. Solo se deben proporcionar los valores para los parámetros definidos por el autor cuando se use la plantilla. Un usuario sin conocimientos detallados de las aplicaciones y/o Batch, por tanto, puede usar las plantillas.

-   **Variables**

    -   Permiten que se especifiquen valores de parámetro complejos o simples en un solo lugar y se usen en uno o más lugares en el cuerpo de la plantilla. Las variables pueden simplificar y reducir el tamaño de la plantilla, así como también facilitar el mantenimiento al tener una ubicación para cambiar las propiedades.

-   **Construcciones de nivel superiores**

    -   Algunas construcciones de niveles superiores están disponibles en la plantilla que todavía no están disponibles en las API de Batch. Por ejemplo, se puede definir un generador de tareas en una plantilla de trabajo que crea varias tareas para el trabajo mediante una definición de tarea común. Estas construcciones evitan la necesidad de programar para crear de forma dinámica varios archivos JSON, como un archivo por cada tarea, así como crear archivos de script para instalar aplicaciones a través de un administrador de paquetes.

    -   En algún momento, estas construcciones pueden agregarse al servicio de Batch y estar disponibles en las API, las interfaces de usuario y otros elementos de Batch.

### <a name="pool-templates"></a>Plantillas de grupo

Las plantillas de grupo admiten las funcionalidades de plantilla estándar de parámetros y variables. También admiten la siguiente construcción de nivel superior:

-   **Referencias de paquetes**

    -   Opcionalmente, permite que el software se copie en nodos de grupo mediante el uso de administradores de paquetes. Se especifican el administrador y el identificador de paquete. Al declarar uno o varios paquetes, evita la necesidad de crear un script que obtenga los paquetes necesarios, instalarlo y ejecutarlo en cada nodo de grupo.

El siguiente es un ejemplo de una plantilla que crea un grupo de máquinas virtuales de Linux con ffmpeg instalado. Para usarlo, proporcione solo una cadena de identificador de grupo y el número de máquinas virtuales del grupo:

```json
{
    "parameters": {
        "nodeCount": {
            "type": "int",
            "metadata": {
                "description": "The number of pool nodes"
            }
        },
        "poolId": {
            "type": "string",
            "metadata": {
                "description": "The pool ID "
            }
        }
    },
    "pool": {
        "type": "Microsoft.Batch/batchAccounts/pools",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "[parameters('poolId')]",
            "virtualMachineConfiguration": {
                "imageReference": {
                    "publisher": "Canonical",
                    "offer": "UbuntuServer",
                    "sku": "16.04-LTS",
                    "version": "latest"
                },
                "nodeAgentSKUId": "batch.node.ubuntu 16.04"
            },
            "vmSize": "STANDARD_D3_V2",
            "targetDedicatedNodes": "[parameters('nodeCount')]",
            "enableAutoScale": false,
            "maxTasksPerNode": 1,
            "packageReferences": [
                {
                    "type": "aptPackage",
                    "id": "ffmpeg"
                }
            ]
        }
    }
}
```

Si el archivo de plantilla se denominara _pool-ffmpeg.json_, invoque la plantilla del modo siguiente:

```azurecli
az batch pool create --template pool-ffmpeg.json
```

La CLI le solicita que proporcione valores para los parámetros `poolId` y `nodeCount`. También puede proporcionar los parámetros en un archivo JSON. Por ejemplo:

```json
{
  "poolId": {
    "value": "mypool"
  },
  "nodeCount": {
    "value": 2
  }
}
```

Si el archivo JSON de parámetros se llamaba *pool-parameters.json*, invoque entonces la plantilla de la manera siguiente:

```azurecli
az batch pool create --template pool-ffmpeg.json --parameters pool-parameters.json
```

### <a name="job-templates"></a>Plantillas de trabajo

Las plantillas de trabajos admiten las funcionalidades de plantilla estándar de parámetros y variables. También admiten la siguiente construcción de nivel superior:

-   **Fábrica de tareas**

    -   Crea varias tareas para un trabajo a partir de una definición de tarea. Se admiten tres tipos de fábricas de tareas: barrido paramétrico, tarea por archivo y colección de tareas.

El siguiente es un ejemplo de una plantilla que crea un trabajo para transcodificar archivos de video MP4 con ffmpeg a una de las dos resoluciones más bajas. Crea una tarea por cada archivo de vídeo de origen: Para más información sobre los grupos de archivos para la entrada y salida de trabajos, consulte [Grupos de archivos y transferencia de archivos](#file-groups-and-file-transfer).

```json
{
    "parameters": {
        "poolId": {
            "type": "string",
            "metadata": {
                "description": "The name of Azure Batch pool which runs the job"
            }
        },
        "jobId": {
            "type": "string",
            "metadata": {
                "description": "The name of Azure Batch job"
            }
        },
        "resolution": {
            "type": "string",
            "defaultValue": "428x240",
            "allowedValues": [
                "428x240",
                "854x480"
            ],
            "metadata": {
                "description": "Target video resolution"
            }
        }
    },
    "job": {
        "type": "Microsoft.Batch/batchAccounts/jobs",
        "apiVersion": "2016-12-01",
        "properties": {
            "id": "[parameters('jobId')]",
            "constraints": {
                "maxWallClockTime": "PT5H",
                "maxTaskRetryCount": 1
            },
            "poolInfo": {
                "poolId": "[parameters('poolId')]"
            },
            "taskFactory": {
                "type": "taskPerFile",
                "source": { 
                    "fileGroup": "ffmpeg-input"
                },
                "repeatTask": {
                    "commandLine": "ffmpeg -i {fileName} -y -s [parameters('resolution')] -strict -2 {fileNameWithoutExtension}_[parameters('resolution')].mp4",
                    "resourceFiles": [
                        {
                            "blobSource": "{url}",
                            "filePath": "{fileName}"
                        }
                    ],
                    "outputFiles": [
                        {
                            "filePattern": "{fileNameWithoutExtension}_[parameters('resolution')].mp4",
                            "destination": {
                                "autoStorage": {
                                    "path": "{fileNameWithoutExtension}_[parameters('resolution')].mp4",
                                    "fileGroup": "ffmpeg-output"
                                }
                            },
                            "uploadOptions": {
                                "uploadCondition": "TaskSuccess"
                            }
                        }
                    ]
                }
            },
            "onAllTasksComplete": "terminatejob"
        }
    }
}
```

Si el archivo de plantilla se denominara _job-ffmpeg.json_, invoque la plantilla del modo siguiente:

```azurecli
az batch job create --template job-ffmpeg.json
```

Como antes, la CLI le pide que proporcione valores para los parámetros. También puede proporcionar los parámetros en un archivo JSON.

### <a name="use-templates-in-batch-explorer"></a>Uso de plantillas en Batch Explorer

Puede cargar una plantilla de la CLI de Batch en la aplicación de escritorio [Batch Explorer](https://github.com/Azure/BatchExplorer) (anteriormente llamado BatchLabs) para crear un grupo o trabajo de Batch. También puede seleccionar entre las plantillas predefinidas de grupo y trabajo de la Galería de Batch Explorer.

Para cargar una plantilla:

1. En Batch Explorer, seleccione **Gallery** > **Local templates** (Galería > Plantillas locales).

2. Seleccione, o arrastre y coloque, una plantilla local de grupo o trabajo.

3. Seleccione **Use this template** (Usar esta plantilla), y siga las indicaciones en pantalla.

## <a name="file-groups-and-file-transfer"></a>Grupos de archivos y transferencia de archivos

La mayoría de los trabajos y tareas requieren archivos de entrada y generan archivos de salida. Normalmente, tanto los archivos de entrada como los de salida tienen que transferirse, desde el cliente al nodo, o bien desde el nodo al cliente. La extensión de la CLI de Azure Batch abstrae la transferencia de archivos y utiliza la cuenta de almacenamiento que puede asociar con cada cuenta de Batch.

Un grupo de archivos equivale a un contenedor que se crea en la cuenta de Azure Storage. Dicho grupo puede tener subcarpetas.

La extensión de la CLI de Batch proporciona comandos para la carga de archivos desde el cliente a un grupo de archivos especificado y la descarga de archivos desde el grupo de archivos especificado a un cliente.

```azurecli
az batch file upload --local-path c:\source_videos\*.mp4 
    --file-group ffmpeg-input

az batch file download --file-group ffmpeg-output --local-path
    c:\output_lowres_videos
```

Las plantillas de grupo y de trabajo permiten que se especifiquen los archivos almacenados en grupos de archivos para la copia en nodos de grupo o para su extracción de los nodos de grupo a un grupo de archivos. Por ejemplo, en la plantilla de trabajo especificada anteriormente, el grupo de archivos *ffmpeg-input* se especifica para el generador de tareas como la ubicación de los archivos de vídeo de origen copiados hasta el nodo para su transcodificación. El grupo de archivos *ffmpeg-output* es la ubicación donde se copian los archivos de salida transcodificados desde el nodo que ejecuta cada tarea.

## <a name="summary"></a>Resumen

Actualmente, solo se ha agregado la compatibilidad de transferencia de plantillas y archivos a la CLI de Azure. El objetivo es expandir la audiencia que pueda usar Batch a los usuarios que no necesitan desarrollar código usando las API de Batch, como los investigadores y los usuarios de TI. Sin programación, los usuarios con conocimientos de Azure, Batch y las aplicaciones que ejecuta Batch, pueden crear plantillas para la creación de grupos y trabajos. Con los parámetros de plantilla, los usuarios sin un conocimiento detallado de Batch ni de las aplicaciones pueden usar las plantillas.

Pruebe la extensión Batch para la CLI de Azure y proporcione sus comentarios y sugerencias, ya sea en los comentarios de este artículo o a través del [repositorio para la comunidad de Batch](https://github.com/Azure/Batch).

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar documentación detallada de instalación y el uso, ejemplos y el código fuente en el [repositorio de GitHub de Azure](https://github.com/Azure/azure-batch-cli-extensions).

- Más información sobre el uso de [Batch Explorer](https://github.com/Azure/BatchExplorer) para crear y administrar recursos de Batch.
