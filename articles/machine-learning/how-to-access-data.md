---
title: Conexión con los servicios de Azure Storage
titleSuffix: Azure Machine Learning
description: Aprenda a usar los almacenes de datos para conectarse de forma segura a los servicios de Azure Storage durante el entrenamiento con Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sihhu
author: MayMSFT
ms.reviewer: nibaccam
ms.date: 07/22/2020
ms.custom: how-to, contperfq1, devx-track-python
ms.openlocfilehash: c5200214946b52ce974a8b7557e38eb57481028a
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782998"
---
# <a name="connect-to-azure-storage-services"></a>Conexión con los servicios de Azure Storage
[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, aprenderá a **conectarse a los servicios de Azure Storage a través de almacenes de datos de Azure Machine Learning**. Los almacenes de datos se conectan de forma segura a su servicio Azure Storage sin poner en riesgo sus credenciales de autenticación ni la integridad de su origen de datos original. Almacenan información de conexión, como el identificador de suscripción y la autorización de token de su instancia de [Key Vault](https://azure.microsoft.com/services/key-vault/) asociada con el área de trabajo, para que pueda acceder de forma segura al almacenamiento sin tener que codificarlos de forma rígida en los scripts. Puede usar el [SDK de Python de Azure Machine Learning](#python) o [Azure Machine Learning Studio](#studio) para crear y registrar almacenes de datos.

Si prefiere crear y administrar almacenes de datos mediante la extensión Azure Machine Learning para VS Code, visite la [guía paso a paso sobre la administración de recursos de VS Code](how-to-manage-resources-vscode.md#datastores) para más información.

Puede crear almacenes de datos a partir de [estas soluciones de Azure Storage](#matrix). **En el caso de las soluciones de almacenamiento no compatibles** y para ahorrar el costo de salida durante los experimentos de ML, [mueva los datos](#move) a una solución de almacenamiento de Azure compatible.  

Para comprender el lugar de los almacenes de datos en el flujo de trabajo global de acceso a datos de Azure Machine Learning, consulte el artículo [Acceso seguro a los datos](concept-data.md#data-workflow).

## <a name="prerequisites"></a>Requisitos previos

Necesitará:
- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

- Una cuenta de Azure Storage con un [tipo de almacenamiento compatible](#matrix).

- [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) o acceso a [Azure Machine Learning Studio](https://ml.azure.com/).

- Un área de trabajo de Azure Machine Learning.
  
  [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) o use una existente mediante el SDK de Python. 

    Importe las clases `Workspace` y `Datastore`, y cargue la información de la suscripción desde el archivo `config.json` con la función `from_config()`. De forma predeterminada, busca el archivo JSON en el directorio actual, pero también puede especificar un parámetro de ruta de acceso para que apunte al archivo mediante `from_config(path="your/file/path")`.

   ```Python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

    Al crear un área de trabajo, se registran automáticamente un contenedor de blobs de Azure y un recurso compartido de archivos de Azure como almacenes de datos en el área de trabajo. Se denominan `workspaceblobstore` y `workspacefilestore`, respectivamente. `workspaceblobstore` se usa para almacenar los artefactos del área de trabajo y los registros del experimento de aprendizaje automático. También se establece como el **almacén de datos predeterminado** y no se puede eliminar del área de trabajo. `workspacefilestore` se usa para almacenar cuadernos y scripts de R autorizados a través de la [instancia de proceso](https://docs.microsoft.com/azure/machine-learning/concept-compute-instance#accessing-files).
    
    > [!NOTE]
    > El diseñador de Azure Machine Learning (versión preliminar) creará automáticamente un almacén de datos denominado **azureml_globaldatasets** al abrir un ejemplo en la página principal del diseñador. Este almacén de datos solo contiene conjuntos de datos de ejemplo. **No** use este almacén de datos para el acceso a datos confidenciales.

<a name="matrix"></a>

## <a name="supported-data-storage-service-types"></a>Tipos de servicio de almacenamiento de datos admitidos

Los almacenes de datos admiten actualmente el almacenamiento de la información de conexión a los servicios de almacenamiento que se enumeran en la siguiente matriz.

| Tipo de&nbsp;almacenamiento | Tipo de&nbsp;autenticación | [Azure&nbsp;Machine&nbsp;Learning Studio](https://ml.azure.com/) | SDL de Python de [Azure&nbsp;Machine&nbsp;Learning Studio&nbsp;](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) |  Cli de [Azure&nbsp;Machine&nbsp;Learning CLI](reference-azure-machine-learning-cli.md) | API Rest de [Azure&nbsp;Machine&nbsp;Learning&nbsp;](https://docs.microsoft.com/rest/api/azureml/) | Código de VS
---|---|---|---|---|---|---
[Azure&nbsp;Blob&nbsp;Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview)| Clave de cuenta <br> Token de SAS | ✓ | ✓ | ✓ |✓ |✓
[Recurso compartido de &nbsp;archivos de&nbsp;Azure](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)| Clave de cuenta <br> Token de SAS | ✓ | ✓ | ✓ |✓|✓
[Azure&nbsp;Data Lake&nbsp;Storage Gen&nbsp;1](https://docs.microsoft.com/azure/data-lake-store/)| Entidad de servicio| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Data Lake&nbsp;Storage Gen&nbsp;2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)| Entidad de servicio| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;SQL&nbsp;Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)| Autenticación SQL <br>Entidad de servicio| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview) | Autenticación SQL| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Database&nbsp;for&nbsp;MySQL](https://docs.microsoft.com/azure/mysql/overview) | Autenticación SQL|  | ✓* | ✓* |✓*|
[Sistema&nbsp; de archivos de &nbsp;Databricks](https://docs.microsoft.com/azure/databricks/data/databricks-file-system)| Sin autenticación | | ✓** | ✓ ** |✓** |

*MySQL solo se admite para la canalización [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?view=azure-ml-py). <br>
**Databricks solo se admite para la canalización [DatabricksStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep?view=azure-ml-py).

### <a name="storage-guidance"></a>Orientación sobre el almacenamiento

Le recomendamos que cree un almacén de datos para un [contenedor de blobs de Azure](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction). El almacenamiento Estándar y Premium están disponibles para blobs. Aunque el almacenamiento premium sea más costoso, su mayor velocidad de rendimiento puede mejorar la velocidad de las ejecuciones de entrenamiento, sobre todo si usa un gran conjunto de datos. Para obtener información sobre el costo de las cuentas de almacenamiento, vea la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/?service=machine-learning-service).

[Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction?toc=/azure/storage/blobs/toc.json) se basa en Azure Blob Storage y se ha diseñado para el análisis de macrodatos empresarial. Parte fundamental de Data Lake Storage Gen2 es la incorporación de un [espacio de nombres jerárquico](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-namespace) en Blob Storage. El espacio de nombres jerárquico organiza los objetos o archivos en una jerarquía de directorios para un acceso eficaz a los datos.

## <a name="storage-access-and-permissions"></a>Permisos y acceso a Storage

Para asegurarse de que se conecta de forma segura a su servicio Azure Storage, Azure Machine Learning requiere que tenga permiso para obtener acceso al contenedor de almacenamiento de datos correspondiente. Este acceso depende de las credenciales de autenticación usadas para registrar el almacén de datos. 

### <a name="virtual-network"></a>Virtual network 

Si la cuenta de almacenamiento de datos se encuentra en una **red virtual**, se necesitan pasos de configuración adicionales para garantizar que Azure Machine Learning tenga acceso a los datos. Consulte [Aislamiento de red y privacidad](how-to-enable-virtual-network.md#machine-learning-studio) para asegurarse de que se aplican los pasos de configuración adecuados al crear y registrar el almacén de datos.  

### <a name="access-validation"></a>Validación de acceso

**Como parte del proceso de creación y registro del almacén de datos inicial**, Azure Machine Learning valida automáticamente que el servicio de almacenamiento subyacente exista y que la entidad de seguridad proporcionada por el usuario (nombre de usuario, entidad de servicio o token de SAS) tenga acceso al almacenamiento especificado.

**Una vez creado el almacén de datos**, esta validación solo se realiza para los métodos que requieren acceso al contenedor de almacenamiento subyacente, y **no** cada vez que se recuperan objetos del almacén de datos. Por ejemplo, la validación se produce si quiere descargar archivos del almacén de archivos. Sin embargo, no se produce si solo quiere cambiar el almacén de datos predeterminado.

Para autenticar su acceso al servicio de almacenamiento subyacente, puede proporcionar su clave de cuenta, tokens de firmas de acceso compartido (SAS) o entidad de servicio en el método `register_azure_*()` correspondiente del tipo de almacén de datos que desea crear. La [matriz de tipo de almacenamiento](#matrix) muestra los tipos de autenticación admitidos que corresponden a cada tipo de almacén de datos.

Encontrará información sobre la clave de cuenta, el token de SAS y la entidad de servicio en [Azure Portal](https://portal.azure.com).

* Si tiene previsto usar una clave de cuenta o un token de SAS para la autenticación, seleccione **Cuentas de almacenamiento** en el panel izquierdo y elija la cuenta de almacenamiento que quiere registrar. 
  * La página **Información general** proporciona información como el nombre de la cuenta, el contenedor y el nombre del recurso compartido de archivos. 
      1. En el caso de las claves de cuenta, vaya a **Claves de acceso** en el panel **Configuración**. 
      1. En el caso de los tokens de SAS, vaya a **Firmas de acceso compartido** en el panel **Configuración**.

* Si tiene previsto usar una entidad de servicio para la autenticación, vaya a **Registros de aplicaciones** y seleccione la aplicación que quiere usar. 
    * Su página de **información general** correspondiente contendrá la información necesaria, como el id. de inquilino y de cliente.

> [!IMPORTANT]
> Por motivos de seguridad, puede que necesite cambiar las claves de acceso de una cuenta de Azure Storage (clave de cuenta o token de SAS). Al hacerlo, asegúrese de sincronizar las credenciales nuevas con el área de trabajo y los almacenes de datos conectados a ella. Obtenga información sobre cómo sincronizar las credenciales actualizadas con [estos pasos](how-to-change-storage-access-key.md). 

### <a name="permissions"></a>Permisos

En el caso del almacenamiento de Azure Data Lake Gen 2 y el contenedor de blobs de Azure, asegúrese de que sus credenciales de autenticación tengan acceso al **Lector de datos de Storage Blob**. Obtenga más información sobre el [Lector de datos de Storage Blob](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-reader). 

<a name="python"></a>

## <a name="create-and-register-datastores-via-the-sdk"></a>Creación y registro de almacenes de datos a través del SDK

Cuando se registra una solución de Azure Storage como almacén de datos, se crea y registra automáticamente ese almacén de datos en un área de trabajo específica. Revise la sección [Permisos y acceso a Storage](#storage-access-and-permissions) para saber dónde encontrar las credenciales de autenticación necesarias.

En esta sección encontrará ejemplos de cómo crear y registrar un almacén de datos a través del SDK de Python para los siguientes tipos de almacenamiento. Los parámetros proporcionados en estos ejemplos son los **parámetros obligatorios** para crear y registrar un almacén de datos.

* [Contenedor de blobs de Azure](#azure-blob-container)
* [Recurso compartido de archivos de Azure](#azure-file-share)
* [Azure Data Lake Storage Generation 2](#azure-data-lake-storage-generation-2)

 A fin de crear almacenes de datos para otros servicios de almacenamiento admitidos, consulte la [documentación de referencia de los métodos `register_azure_*` aplicables](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore.datastore?view=azure-ml-py#methods).

Si prefiere una experiencia de código bajo, consulte la sección sobre cómo [crear almacenes de datos en Azure Machine Learning Studio](#studio).

> [!NOTE]
> El nombre del almacén de datos solo puede contener letras minúsculas, dígitos y caracteres de subrayado. 

### <a name="azure-blob-container"></a>Contenedor de blobs de Azure

Para registrar un almacén de datos de un contenedor de blobs de Azure, use [`register_azure_blob_container()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#register-azure-blob-container-workspace--datastore-name--container-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false--blob-cache-timeout-none--grant-workspace-access-false--subscription-id-none--resource-group-none-).

En el código siguiente se crea y registra el almacén de datos `blob_datastore_name` en el área de trabajo `ws`. Este almacén de datos tiene acceso al contenedor de blobs `my-container-name` en la cuenta de almacenamiento `my-account-name` con la clave de acceso a la cuenta proporcionada.

```Python
blob_datastore_name='azblobsdk' # Name of the datastore to workspace
container_name=os.getenv("BLOB_CONTAINER", "<my-container-name>") # Name of Azure blob container
account_name=os.getenv("BLOB_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("BLOB_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

blob_datastore = Datastore.register_azure_blob_container(workspace=ws, 
                                                         datastore_name=blob_datastore_name, 
                                                         container_name=container_name, 
                                                         account_name=account_name,
                                                         account_key=account_key)
```

### <a name="azure-file-share"></a>Recurso compartido de archivos de Azure

Para registrar un recurso compartido de archivos de Azure como almacén de datos, use [`register_azure_file_share()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#register-azure-file-share-workspace--datastore-name--file-share-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false-). 

En el código siguiente se crea y registra el almacén de datos `file_datastore_name` en el área de trabajo `ws`. Este almacén de datos tiene acceso al recurso compartido de archivos `my-fileshare-name` en la cuenta de almacenamiento `my-account-name` con la clave de acceso a la cuenta proporcionada.

```Python
file_datastore_name='azfilesharesdk' # Name of the datastore to workspace
file_share_name=os.getenv("FILE_SHARE_CONTAINER", "<my-fileshare-name>") # Name of Azure file share container
account_name=os.getenv("FILE_SHARE_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("FILE_SHARE_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

file_datastore = Datastore.register_azure_file_share(workspace=ws,
                                                     datastore_name=file_datastore_name, 
                                                     file_share_name=file_share_name, 
                                                     account_name=account_name,
                                                     account_key=account_key)
```

### <a name="azure-data-lake-storage-generation-2"></a>Azure Data Lake Storage Generation 2

Para un almacén de datos de Azure Data Lake Storage Generation 2 (ADLS Gen 2), utilice [register_azure_data_lake_gen2()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore.datastore?view=azure-ml-py#register-azure-data-lake-gen2-workspace--datastore-name--filesystem--account-name--tenant-id--client-id--client-secret--resource-url-none--authority-url-none--protocol-none--endpoint-none--overwrite-false-) para registrar un almacén de datos de credenciales conectado a un almacenamiento Azure DataLake Gen 2 con [permisos de entidad de servicio](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal). 

Para poder usar la entidad de servicio, debe [registrar la aplicación](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals) y conceder el acceso **Lector de datos de Storage Blob** adecuado a la entidad de servicio. Aprenda más sobre la [configuración del control de acceso en ADLS Gen 2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control). 

En el código siguiente se crea y registra el almacén de datos `adlsgen2_datastore_name` en el área de trabajo `ws`. Este almacén de datos accede al sistema de archivos `test` en la cuenta de almacenamiento `account_name` con las credenciales de la entidad de servicio facilitadas.

```python 
adlsgen2_datastore_name = 'adlsgen2datastore'

subscription_id=os.getenv("ADL_SUBSCRIPTION", "<my_subscription_id>") # subscription id of ADLS account
resource_group=os.getenv("ADL_RESOURCE_GROUP", "<my_resource_group>") # resource group of ADLS account

account_name=os.getenv("ADLSGEN2_ACCOUNTNAME", "<my_account_name>") # ADLS Gen2 account name
tenant_id=os.getenv("ADLSGEN2_TENANT", "<my_tenant_id>") # tenant id of service principal
client_id=os.getenv("ADLSGEN2_CLIENTID", "<my_client_id>") # client id of service principal
client_secret=os.getenv("ADLSGEN2_CLIENT_SECRET", "<my_client_secret>") # the secret of service principal

adlsgen2_datastore = Datastore.register_azure_data_lake_gen2(workspace=ws,
                                                             datastore_name=adlsgen2_datastore_name,
                                                             account_name=account_name, # ADLS Gen2 account name
                                                             filesystem='test', # ADLS Gen2 filesystem
                                                             tenant_id=tenant_id, # tenant id of service principal
                                                             client_id=client_id, # client id of service principal
                                                             client_secret=client_secret) # the secret of service principal
```

<a name="studio"></a>


## <a name="create-datastores-in-the-studio"></a>Creación de almacenes de datos en Studio 


Cree un nuevo almacén de datos en unos cuantos pasos con Azure Machine Learning Studio.

> [!IMPORTANT]
> Si la cuenta de almacenamiento de datos se encuentra en una red virtual, se necesitan pasos de configuración adicionales para garantizar que Studio tenga acceso a los datos. Vea [Aislamiento de red y privacidad](how-to-enable-virtual-network.md#machine-learning-studio) para asegurarse de que se aplican los pasos de configuración adecuados. 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. Seleccione **Almacenes de datos** en el panel izquierdo en **Administrar**.
1. Seleccione **+ Nuevo almacén de datos**.
1. Complete el formulario para un nuevo almacén de datos. El formulario se actualiza de forma inteligente según las selecciones de tipo de Azure Storage y de autenticación. Consulte la [sección Permisos y acceso a Storage](#access-validation) para saber dónde encontrar las credenciales de autenticación que necesita para rellenar este formulario.

En el ejemplo siguiente se muestra el aspecto que tendría el formulario al crear un **almacén de Azure Blob**: 
    
![Formulario para un nuevo almacén de datos](media/how-to-access-data/new-datastore-form.png)

<a name="train"></a>
## <a name="use-data-in-your-datastores"></a>Uso de datos en los almacenes de datos

Una vez que cree un almacén de datos, [cree un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md) para interactuar con los datos. Los conjuntos de datos empaquetan sus datos en un objeto consumible evaluado de forma diferida para tareas de aprendizaje automático, como un curso. También proporcionan la capacidad de [descargar o montar](how-to-train-with-datasets.md#mount-vs-download) archivos de cualquier formato desde servicios Azure Storage como Azure Blob Storage y ADLS Gen 2. También puede usarlos para cargar datos tabulares en un DataFrame de Pandas o Spark.

<a name="get"></a>

## <a name="get-datastores-from-your-workspace"></a>Obtención de almacenes de almacenamiento del área de trabajo

Para obtener un almacén de datos específico registrado en el área de trabajo actual, utilice el método estático [`get()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.datastore(class)?view=azure-ml-py#get-workspace--datastore-name-) en la clase `Datastore`:

```Python
# Get a named datastore from the current workspace
datastore = Datastore.get(ws, datastore_name='your datastore name')
```
Para obtener la lista de almacenes de datos registrados con un área de trabajo determinada, puede utilizar la propiedad [`datastores`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace%28class%29?view=azure-ml-py#datastores) en un objeto del área de trabajo:

```Python
# List all datastores registered in the current workspace
datastores = ws.datastores
for name, datastore in datastores.items():
    print(name, datastore.datastore_type)
```

Para obtener el almacén de datos predeterminado del área de trabajo, use esta línea:

```Python
datastore = ws.get_default_datastore()
```
También puede cambiar el almacén de datos predeterminado con el código siguiente. Esta capacidad solo se admite a través del SDK. 

```Python
 ws.set_default_datastore(new_default_datastore)
```

## <a name="access-data-during-scoring"></a>Acceso a los datos durante la puntuación

Azure Machine Learning dispone de varios métodos para usar los modelos para puntuación. Algunos de estos métodos no proporcionan acceso a los almacenes de datos. Use la tabla siguiente para saber qué métodos le permiten acceder a los almacenes de datos durante la puntuación:

| Método | Acceso a almacén de datos | Descripción |
| ----- | :-----: | ----- |
| [Predicción por lotes](how-to-use-parallel-run-step.md) | ✔ | Realice predicciones sobre grandes cantidades de datos asincrónicamente. |
| [Servicio web](how-to-deploy-and-where.md) | &nbsp; | Implemente modelos como servicios web. |
| [Módulo de Azure IoT Edge](how-to-deploy-and-where.md) | &nbsp; | Implemente modelos en dispositivos IoT Edge. |

En situaciones en las que el SDK no proporciona acceso a los almacenes de datos, es posible que pueda crear código personalizado mediante el SDK de Azure correspondiente para obtener acceso a los datos. Por ejemplo, el [SDK de Azure Storage para Python](https://github.com/Azure/azure-storage-python) es una biblioteca cliente que puede usar para acceder a los datos almacenados en blobs o archivos.

<a name="move"></a>

## <a name="move-data-to-supported-azure-storage-solutions"></a>Movimiento de datos a soluciones de Azure Storage compatibles

Azure Machine Learning admite el acceso a datos desde Azure Blob Storage, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database y Azure Database for PostgreSQL. Si usa un almacenamiento que no es compatible, le recomendamos que mueva los datos a soluciones de Azure Storage compatibles mediante [Azure Data Factory y estos pasos](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-copy-data-tool). El movimiento de datos a un almacenamiento compatible puede ayudarle a ahorrar costos de salida de datos durante los experimentos de aprendizaje automático. 

Azure Data Factory proporciona una transferencia de datos eficaz y resistente con más de 80 conectores pregenerados sin costo adicional. Estos conectores incluyen servicios de datos de Azure, orígenes de datos locales, Amazon S3 y Redshift, y Google BigQuery.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md)
* [Entrenamiento de un modelo](how-to-train-ml-models.md)
* [Implementar un modelo](how-to-deploy-and-where.md)
