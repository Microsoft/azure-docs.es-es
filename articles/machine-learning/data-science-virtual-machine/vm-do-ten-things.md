---
title: Exploración de los datos y el modelo en Windows
titleSuffix: Azure Data Science Virtual Machine
description: Realice tareas de exploración y modelado de datos en Windows Data Science Virtual Machine.
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 05/08/2020
ms.openlocfilehash: 82e7cd67da6b64788d597e84c723d797138ac608
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848543"
---
# <a name="ten-things-you-can-do-on-the-windows-data-science-virtual-machine"></a>Diez cosas que puede hacer en Windows Data Science Virtual Machine

Windows Data Science Virtual Machine (DSVM) es un entorno de desarrollo de ciencia de datos eficaz en el que puede realizar tareas de exploración y modelado de datos. El entorno incluye ya en su compilación varias herramientas populares de análisis de datos que le permitirán comenzar fácilmente su análisis de implementaciones locales, híbridas o en la nube. 

DSVM trabaja estrechamente con los servicios de Azure. Puede leer y procesar los datos que ya están almacenados en Azure, Azure SQL Data Warehouse, Azure Data Lake, Azure Storage o Azure Cosmos DB. También puede aprovechar otras herramientas de análisis como Azure Machine Learning y Azure Data Factory.

En este artículo, aprenderá a usar DSVM para realizar tareas de ciencia de datos e interactuar con otros servicios de Azure. Estas son algunas de las tareas que puede hacer en DSVM:

- Explorar datos y desarrollar modelos localmente en DSVM con Microsoft Machine Learning Server y Python.
- Usar un cuaderno de Jupyter Notebook para experimentar con los datos en un explorador mediante Python 2, Python 3 y Microsoft R. (Microsoft R es una versión de R preparada para las empresas y diseñada para ofrecer rendimiento).
- Implementar los modelos compilados mediante R y Python en Azure Machine Learning para que las aplicaciones cliente puedan acceder a ellos por medio de una sencilla interfaz de servicios web.
- Administrar los recursos de Azure mediante Azure Portal o PowerShell.
- Ampliar el espacio de almacenamiento y compartir código o conjuntos de datos de gran escala entre todo el equipo al crear un recurso compartido de Azure Files como una unidad que se puede montar en DSVM.
- Compartir código con su equipo mediante GitHub. Acceder al repositorio mediante los clientes de Git preinstalados: Git Bash y Git GUI.
- Acceder a servicios de análisis y datos de Azure, como Azure Blob Storage, Azure Data Lake, Azure Cosmos DB, Azure SQL Data Warehouse y Azure SQL Database.
- Crear informes y un panel con la instancia de Power BI Desktop que está preinstalada en DSVM e implementarlos en la nube.
- Escalar dinámicamente DSVM para satisfacer las necesidades del proyecto.
- Instalar herramientas adicionales en la máquina virtual.   

> [!NOTE]
> A muchos de los servicios de almacenamiento y análisis de datos que se enumeran en este artículo se les aplican cargos de uso adicionales. Para obtener información detallada, consulte la página de [precios de Azure](https://azure.microsoft.com/pricing/).
> 
> 

## <a name="prerequisites"></a>Prerrequisitos

* Necesita una suscripción de Azure. Puede [suscribirse a una evaluación gratuita](https://azure.microsoft.com/free/).
* Las instrucciones para el aprovisionamiento de Data Science Virtual Machine en Azure Portal están disponibles en [Creación de una máquina virtual](https://portal.azure.com/#create/microsoft-dsvm.dsvm-windowsserver-2016).


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="explore-data-and-develop-models-with-microsoft-machine-learning-server"></a>Exploración de datos y desarrollo de modelos con Microsoft Machine Learning Server
Puede utilizar lenguajes como R y Python para realizar el análisis de datos directamente en DSVM.

Para R, puede usar un IDE como RStudio que se puede encontrar en el menú Inicio o en el escritorio. O bien puede usar Herramientas de R para Visual Studio. Microsoft ha aportado bibliotecas adicionales además de la de CRAN R de código abierto para habilitar el análisis escalable y la capacidad de analizar datos mayores que el tamaño de memoria permitido en el análisis de fragmentos en paralelo. 

En el caso de Python, puede usar un IDE como Visual Studio Community Edition, que ya tiene preinstalada la extensión Herramientas de Python para Visual Studio (PTVS). De forma predeterminada, solo Python 3.6, el entorno raíz de Conda, se configura en PTVS. Siga estos pasos para habilitar Anaconda Python 2.7:

1. Cree entornos personalizados para cada versión; para ello, vaya a **Herramientas** > **Herramientas de Python** > **Entornos de Python** y seleccione **+ Personalizado** en Visual Studio Community Edition.
1. Incluya una descripción y establezca la ruta de acceso del prefijo del entorno como **c:\anaconda\envs\python2** para Anaconda Python 2.7.
1. Seleccione **Detectar automáticamente** > **Aplicar** para guardar el entorno.

Para consultar más detalles sobre cómo crear entornos de Python, vea la [documentación de PTVS](https://aka.ms/ptvsdocs).

Ya tiene todo configurado para crear un proyecto de Python. Vaya a **Archivo** > **Nuevo** > **Proyecto** > **Python** y seleccione el tipo de aplicación de Python que va a compilar. Puede establecer el entorno de Python del proyecto actual en la versión que quiera (Python 2.7 o 3.6) si hace clic con el botón derecho en **Entornos de Python** y selecciona **Agregar o quitar entornos de Python**. Puede encontrar más información sobre cómo trabajar con PTVS en la [documentación del producto](https://aka.ms/ptvsdocs).

## <a name="use-jupyter-notebooks"></a>Uso de Jupyter Notebooks
Jupyter Notebook proporciona un IDE basado en el explorador para explorar y modelar datos. En un cuaderno de Jupyter Notebook se pueden usar Python 2, Python 3 o R (tanto de código abierto como de Microsoft R Server).

Para iniciar el cuaderno de Jupyter Notebook, seleccione el icono **Jupyter Notebook** en el menú **Inicio** o en el escritorio. En el símbolo del sistema de DSVM, también puede ejecutar el comando ```jupyter notebook``` desde el directorio donde tiene cuadernos existentes o donde quiere crear cuadernos.  

Después de iniciar Jupyter, vaya al directorio `/notebooks` para ver los cuadernos de ejemplo que vienen incluidos en DSVM. Ahora puede:

* Seleccionar el cuaderno para ver el código.
* Seleccionar Mayús+Entrar para ejecutar cada celda.
* Seleccionar **Celda** > **Ejecutar** para ejecutar todo el cuaderno.
* Seleccionar el icono de Jupyter (esquina superior izquierda), después el botón **Nuevo** situado a la derecha y, finalmente, elegir el idioma del cuaderno (también conocido como kernels) para crear un cuaderno.   

> [!NOTE]
> Actualmente se admiten los kernels de Python 2.7, Python 3.6, R, Julia y PySpark en Jupyter. El kernel de R admite la programación en R de código abierto y en Microsoft R.   
> 
> 

Cuando esté en el cuaderno, podrá explorar los datos, así como compilar y probar el modelo con las bibliotecas que elija.

## <a name="train-and-deploy-models-by-using-azure-machine-learning"></a>Entrenamiento e implementación de modelos mediante Azure Machine Learning
Una vez compilado y validado el modelo, el paso siguiente suele ser implementarlo en producción. Con este paso, las aplicaciones cliente pueden invocar las predicciones del modelo en tiempo real o en el modo por lotes. Azure Machine Learning proporciona un mecanismo para hacer que un modelo creado en R o Python esté operativo.

Al poner en marcha el modelo en Azure Machine Learning, se expone un servicio web que permite a los clientes realizar llamadas REST que pasan parámetros de entrada y reciben predicciones del modelo como salidas.

### <a name="build-and-operationalize-python-models"></a>Compilación y puesta en marcha de modelos de Python
Este es un fragmento de código desarrollado en un cuaderno de Jupyter Notebook de Python que compila un modelo simple mediante la biblioteca Scikit-learn:

```python
# IRIS classification
from sklearn import datasets
from sklearn import svm
clf = svm.SVC()
iris = datasets.load_iris()
X, y = iris.data, iris.target
clf.fit(X, y)
```

El método que se usa para implementar los modelos de Python en Azure Machine Learning encapsula la predicción del modelo en una función y la decora con los atributos que proporciona la biblioteca de Python de Azure Machine Learning preinstalada. Los atributos denotan el identificador de área de trabajo de Azure Machine Learning, la clave de API y los parámetros de entrada y retorno.  

```python
from azureml import services
@services.publish(workspaceid, auth_token)
@services.types(sep_l=float, sep_w=float, pet_l=float, pet_w=float)
@services.returns(int)  # 0, or 1, or 2
def predictIris(sep_l, sep_w, pet_l, pet_w):
    inputArray = [sep_l, sep_w, pet_l, pet_w]


return clf.predict(inputArray)
```

Un cliente ahora puede realizar llamadas al servicio web. Para su comodidad, los contenedores construyen las solicitudes de la API REST. Aquí puede ver un código de ejemplo para consumir el servicio web:

```python
# Consume through web service URL and keys
from azureml import services
@services.service(url, api_key)
@services.types(sep_l = float, sep_w = float, pet_l=float, pet_w=float)
@services.returns(float)
def IrisPredictor(sep_l, sep_w, pet_l, pet_w):
pass

IrisPredictor(3,2,3,4)
```

### <a name="build-and-operationalize-r-models"></a>Compilación y puesta en marcha de modelos de R
Los modelos de R compilados en Data Science Virtual Machine, o en cualquier otro lugar, se pueden implementar en Azure Machine Learning de forma similar a como se hace para Python. He aquí los pasos:

1. Cree un archivo settings.json para proporcionar el identificador del área de trabajo y el token de autenticación. 
2. Escriba un contenedor para la función de predicción del modelo.
3. Llame a ```publishWebService``` en la biblioteca de Azure Machine Learning para pasar el contenedor de funciones.  

Use el procedimiento y los fragmentos de código que se indican a continuación para configurar, compilar, publicar y consumir un modelo como un servicio web en Azure Machine Learning.

#### <a name="set-up"></a>Configurar

Cree un archivo settings.json en un directorio denominado ```.azureml``` en el directorio particular. Escriba los parámetros de su área de trabajo de Azure Machine Learning.

Esta es la estructura del archivo settings.json:

```json
{"workspace":{
"id"                  : "ENTER YOUR AZUREML WORKSPACE ID",
"authorization_token" : "ENTER YOUR AZUREML AUTH TOKEN"
}}
```

#### <a name="build-a-model-in-r-and-publish-it-in-azure-machine-learning"></a>Creación de un modelo en R y posterior publicación en Azure Machine Learning

```r
library(AzureML)
ws <- workspace(config="~/.azureml/settings.json")

if(!require("lme4")) install.packages("lme4")
library(lme4)
set.seed(1)
train <- sleepstudy[sample(nrow(sleepstudy), 120),]
m <- lm(Reaction ~ Days + Subject, data = train)

# Define a prediction function to publish based on the model:
sleepyPredict <- function(newdata){
        predict(m, newdata=newdata)
}

ep <- publishWebService(ws, fun = sleepyPredict, name="sleepy lm", inputSchema = sleepstudy, data.frame=TRUE)
```

#### <a name="consume-the-model-deployed-in-azure-machine-learning"></a>Consumo del modelo implementado en Azure Machine Learning
Para consumir el modelo desde una aplicación cliente, use la biblioteca de Azure Machine Learning para buscar el servicio web publicado por su nombre. Use la llamada API `services` para determinar el punto de conexión. Luego basta con llamar a la función `consume` y pasar la trama de datos que se va a predecir.

Use el siguiente código para consumir el modelo publicado como un servicio web Azure Machine Learning:

```r
library(AzureML)
library(lme4)
ws <- workspace(config="~/.azureml/settings.json")

s <-  services(ws, name = "sleepy lm")
s <- tail(s, 1) # use the last published function, in case of duplicate function names

ep <- endpoints(ws, s)

# Try this out, and compare with raw data
ans = consume(ep, sleepstudy)$ans
```

Vea más información sobre los [paquetes de R en Machine Learning Studio](/azure/machine-learning/studio-module-reference/r-packages-supported-by-azure-machine-learning).

## <a name="manage-azure-resources"></a>Administración de recursos de Azure
DSVM no solo le permite compilar su solución de análisis de forma local en la máquina virtual, sino que también le permite acceder a los servicios de la plataforma en la nube de Azure. Azure proporciona varios servicios de proceso, almacenamiento, análisis de datos y de otra índole que puede administrar desde su DSVM y a los que puede acceder desde dicho entorno.

Para administrar los recursos de la nube y la suscripción de Azure, tiene dos opciones:
+ Vaya a [Azure Portal](https://portal.azure.com) en el explorador.

+ Use scripts de PowerShell. Ejecute Azure PowerShell desde un acceso directo del escritorio o desde el menú **Inicio**. Consulte la [documentación de Microsoft Azure PowerShell](../../powershell-azure-resource-manager.md) para obtener más información. 

## <a name="extend-storage-by-using-shared-file-systems"></a>Extensión del almacenamiento mediante sistemas de archivos compartidos
Los científicos de datos pueden compartir grandes conjuntos de datos, código u otros recursos dentro del equipo. DSVM tiene aproximadamente 45 GB de espacio disponible. Para ampliar el almacenamiento, puede usar Azure Files y montarlo en una o varias instancias de DSVM, o bien acceder a él mediante la API REST. También puede usar [Azure Portal](../../virtual-machines/windows/attach-managed-disk-portal.md) o [Azure PowerShell](../../virtual-machines/windows/attach-disk-ps.md) para agregar discos de datos dedicados adicionales. 

> [!NOTE]
> El espacio máximo en el recurso compartido de Azure Files es de 5 TB. El límite de tamaño de cada archivo es de 1 TB. 

Puede usar este script en Azure PowerShell para crear un recurso compartido de Azure Files:

```powershell
# Authenticate to Azure.
Connect-AzAccount
# Select your subscription
Get-AzSubscription –SubscriptionName "<your subscription name>" | Select-AzSubscription
# Create a new resource group.
New-AzResourceGroup -Name <dsvmdatarg>
# Create a new storage account. You can reuse existing storage account if you want.
New-AzStorageAccount -Name <mydatadisk> -ResourceGroupName <dsvmdatarg> -Location "<Azure Data Center Name For eg. South Central US>" -Type "Standard_LRS"
# Set your current working storage account
Set-AzCurrentStorageAccount –ResourceGroupName "<dsvmdatarg>" –StorageAccountName <mydatadisk>

# Create an Azure Files share
$s = New-AzStorageShare <<teamsharename>>
# Create a directory under the file share. You can give it any name
New-AzStorageDirectory -Share $s -Path <directory name>
# List the share to confirm that everything worked
Get-AzStorageFile -Share $s
```

Ahora que ha creado un recurso compartido de Azure Files, puede montarlo en cualquier máquina virtual de Azure. Le recomendamos que ponga la máquina virtual en el mismo centro de datos de Azure que la cuenta de almacenamiento para evitar la latencia y los cobros por la transferencia de datos. Estos son los comandos de Azure PowerShell para montar la unidad en DSVM:

```powershell
# Get the storage key of the storage account that has the Azure Files share from the Azure portal. Store it securely on the VM to avoid being prompted in the next command.
cmdkey /add:<<mydatadisk>>.file.core.windows.net /user:<<mydatadisk>> /pass:<storage key>

# Mount the Azure Files share as drive Z on the VM. You can choose another drive letter if you want.
net use z:  \\<mydatadisk>.file.core.windows.net\<<teamsharename>>
```

Ya puede acceder a esta unidad del mismo modo que a cualquier otra unidad normal de la máquina virtual.

## <a name="share-code-in-github"></a>Código compartido en GitHub
GitHub es un repositorio de código donde puede encontrar ejemplos de código y orígenes para diferentes herramientas mediante las tecnologías que comparte la comunidad de desarrolladores. Utiliza Git como tecnología para realizar un seguimiento de las versiones de los archivos de código y almacenarlas. Además, GitHub es una plataforma donde podrá crear su propio repositorio para almacenar el código compartido y la documentación de su equipo, implementar el control de versiones y controlar quién tiene acceso para ver el código y contribuir en él. 

Para más información sobre el uso de Git, visite las [páginas de ayuda de GitHub](https://help.github.com/). Puede usar GitHub como uno de los métodos para colaborar con su equipo, usar el código desarrollado por la comunidad y aportar código de vuelta a ella.

DSVM incluye herramientas de cliente en la línea de comandos y en la GUI para poder acceder al repositorio de GitHub. La herramienta de línea de comandos que funciona con Git y GitHub se denomina Git Bash. Visual Studio está instalado en DSVM y cuenta con las extensiones de Git. Puede encontrar los iconos de estas herramientas en el menú **Inicio** y en el escritorio.

Para descargar código de un repositorio de GitHub, debe usar el comando ```git clone```. Por ejemplo, para descargar el repositorio de ciencia de datos que publica Microsoft en el directorio actual, puede ejecutar el siguiente comando en Git Bash:

```bash
git clone https://github.com/Azure/DataScienceVM.git
```

En Visual Studio, puede realizar la misma operación de clonación. En la captura de pantalla siguiente, se muestra cómo acceder a las herramientas de Git y GitHub en Visual Studio:

![Captura de Visual Studio con la conexión de GitHub mostrada](./media/vm-do-ten-things/VSGit.PNG)

En los recursos disponibles en github.com, puede encontrar más información sobre cómo usar Git para trabajar con el repositorio de GitHub. La [hoja de referencia rápida](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf) puede resultar un recurso útil.

## <a name="access-azure-data-and-analytics-services"></a>Acceso a servicios de análisis y datos de Azure
### <a name="azure-blob-storage"></a>Azure Blob Storage
Azure Blob Storage es un servicio de almacenamiento en nube confiable y económico para muchos y pocos datos. En esta sección se describe cómo mover los datos a Blob Storage y cómo acceder a los datos almacenados en un blob de Azure.

#### <a name="prerequisites"></a>Prerrequisitos

* Cree una cuenta de Azure Blob Storage desde [Azure Portal](https://portal.azure.com).

   ![Captura de pantalla del proceso de creación de una cuenta de almacenamiento en Azure Portal](./media/vm-do-ten-things/create-azure-blob.png)

* Confirme que la herramienta de línea de comandos AzCopy se ha instalado previamente: ```C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy.exe```. El directorio que contiene el archivo azcopy.exe ya se encuentra en la variable de entorno PATH para que no tenga que escribir la ruta de acceso completa del comando al ejecutar esta herramienta. Para obtener más información sobre la herramienta AzCopy, consulte la [documentación de AzCopy](../../storage/common/storage-use-azcopy.md).
* Inicie el Explorador de Azure Storage desde aquí. Puede descargarlo desde la [página web del Explorador de Storage](https://storageexplorer.com/). 

   ![Captura de pantalla del Explorador de Azure Storage con acceso a una cuenta de almacenamiento](./media/vm-do-ten-things/AzureStorageExplorer_v4.png)

#### <a name="move-data-from-a-vm-to-an-azure-blob-azcopy"></a>Traslado de datos de una máquina virtual a un blob de Azure: AzCopy

Para mover datos entre los archivos locales y Blob Storage, puede usar AzCopy en la línea de comandos o en PowerShell:

```powershell
AzCopy /Source:C:\myfolder /Dest:https://<mystorageaccount>.blob.core.windows.net/<mycontainer> /DestKey:<storage account key> /Pattern:abc.txt
```

Reemplace **C:\myfolder** por la ruta de acceso al lugar donde está almacenado su archivo, **mystorageaccount** por el nombre de su cuenta de Blob Storage, **mycontainer** por el nombre del contenedor y **storage account key** por la clave de acceso a Blob Storage. Puede encontrar las credenciales de su cuenta de almacenamiento en [Azure Portal](https://portal.azure.com).

Ejecute el comando de AzCopy en PowerShell o desde un símbolo del sistema. Aquí puede ver algún ejemplo de uso del comando de AzCopy:

```powershell
# Copy *.sql from a local machine to an Azure blob
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:"c:\Aaqs\Data Science Scripts" /Dest:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /DestKey:[ENTER STORAGE KEY] /S /Pattern:*.sql

# Copy back all files from an Azure blob container to a local machine

"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Dest:"c:\Aaqs\Data Science Scripts\temp" /Source:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /SourceKey:[ENTER STORAGE KEY] /S
```

Después de ejecutar el comando de AzCopy para copiar en un blob de Azure, el archivo aparecerá en el Explorador de Azure Storage.

![Captura de pantalla de la cuenta de almacenamiento en la que se muestra el archivo .csv cargado](./media/vm-do-ten-things/AzCopy_run_finshed_Storage_Explorer_v3.png)

#### <a name="move-data-from-a-vm-to-an-azure-blob-azure-storage-explorer"></a>Traslado de datos de una máquina virtual a un blob de Azure: Explorador de Azure Storage

También puede cargar datos desde el archivo local en la máquina virtual mediante el Explorador de Azure Storage:

* Para cargar datos en un contenedor, elija el contenedor de destino y seleccione el botón **Cargar**.![Captura de pantalla del botón de carga en el Explorador de Azure Storage](./media/vm-do-ten-things/storage-accounts.png)
* Seleccione los puntos suspensivos ( **...** ) que se encuentran a la derecha del cuadro **Archivos**, elija uno o varios archivos que cargar del sistema de archivos y seleccione **Cargar** para empezar a cargarlos.![Captura de pantalla del cuadro de diálogo para cargar archivos](./media/vm-do-ten-things/upload-files-to-blob.png)

#### <a name="read-data-from-an-azure-blob-machine-learning-reader-module"></a>Lectura de datos desde un blob de Azure: módulo lector de Machine Learning

En Azure Machine Learning Studio puede usar el módulo Importar datos para leer datos del blob.

![Captura de pantalla del módulo de importación de datos en Machine Learning Studio](./media/vm-do-ten-things/AML_ReaderBlob_Module_v3.png)

#### <a name="read-data-from-an-azure-blob-python-odbc"></a>Lectura de datos desde un blob de Azure: ODBC de Python

Puede usar la biblioteca BlobService para leer datos directamente de un blob en un cuaderno de Jupyter Notebook o en un programa de Python.

En primer lugar, importe los paquetes necesarios:

```python
import pandas as pd
from pandas import Series, DataFrame
import numpy as np
import matplotlib.pyplot as plt
from time import time
import pyodbc
import os
from azure.storage.blob import BlobService
import tables
import time
import zipfile
import random
```

Después, especifique las credenciales de su cuenta de Blob Storage y lea los datos del blob:

```python
CONTAINERNAME = 'xxx'
STORAGEACCOUNTNAME = 'xxxx'
STORAGEACCOUNTKEY = 'xxxxxxxxxxxxxxxx'
BLOBNAME = 'nyctaxidataset/nyctaxitrip/trip_data_1.csv'
localfilename = 'trip_data_1.csv'
LOCALDIRECTORY = os.getcwd()
LOCALFILE =  os.path.join(LOCALDIRECTORY, localfilename)

#download from blob
t1 = time.time()
blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILE)
t2 = time.time()
print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))

#unzip downloaded files if needed
#with zipfile.ZipFile(ZIPPEDLOCALFILE, "r") as z:
#    z.extractall(LOCALDIRECTORY)

df1 = pd.read_csv(LOCALFILE, header=0)
df1.columns = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime','passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude']
print 'the size of the data is: %d rows and  %d columns' % df1.shape
```

Los datos se leen como una trama de datos:

![Captura de pantalla de las primeras 10 filas de datos](./media/vm-do-ten-things/IPNB_data_readin.PNG)

### <a name="azure-data-lake"></a>Azure Data Lake
Azure Data Lake Store es un repositorio de hiperescala para cargas de trabajo de análisis de macrodatos compatible con el sistema de archivos distribuido Hadoop (HDFS). Funciona con Hadoop, Spark y Azure Data Lake Analytics. En esta sección aprenderá a mover datos a Azure Data Lake Store y ejecutar análisis con Azure Data Lake Analytics.

#### <a name="prerequisites"></a>Prerrequisitos

* Cree una instancia de Azure Data Lake Analytics en [Azure Portal](https://portal.azure.com).

   ![Captura de pantalla de la creación de una instancia de Data Lake Analytics desde Azure Portal](./media/vm-do-ten-things/azure-data-lake-create-v3.png)

* El [complemento Herramientas de Azure Data Lake y Stream Analytics para Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504) ya está instalado en Visual Studio Community Edition en la máquina virtual. Después de iniciar Visual Studio e iniciar sesión en su suscripción de Azure, debería ver su cuenta de Azure Data Analytics y el almacenamiento en el panel izquierdo de Visual Studio.

   ![Captura de pantalla del complemento de las herramientas de Data Lake en Visual Studio](./media/vm-do-ten-things/Azure_Data_Lake_PlugIn_v2.PNG)

#### <a name="move-data-from-a-vm-to-data-lake-azure-data-lake-explorer"></a>Traslado de datos desde una máquina virtual a Data Lake: Explorador de Azure Data Lake

Puede usar el Explorador de Azure Data Lake para [cargar datos desde los archivos locales de la máquina virtual a Data Lake Storage](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal).

También puede crear una canalización de datos para implementar el movimiento de datos hacia o desde Azure Data Lake mediante [Azure Data Factory](https://azure.microsoft.com/services/data-factory/). En [este artículo](https://azure.microsoft.com/blog/creating-big-data-pipelines-using-azure-data-lake-and-azure-data-factory/) se le guiará por los pasos necesarios para generar las canalizaciones de datos.

#### <a name="read-data-from-an-azure-blob-to-data-lake-u-sql"></a>Lectura de datos desde un blob de Azure a Data Lake: U-SQL

Si los datos residen en Azure Blob Storage, puede leerlos directamente desde un blob de Azure en una consulta U-SQL. Antes de crear la consulta U-SQL, asegúrese de que la cuenta de Blob Storage esté vinculada a su instancia de Azure Data Lake. Vaya a Azure Portal, busque el panel de Azure Data Lake Analytics, seleccione **Agregar origen de datos**, seleccione el tipo de almacenamiento **Azure Storage** y escriba el nombre y la clave de su cuenta de Azure Storage. Después, podrá hacer referencia a los datos almacenados en la cuenta de almacenamiento.

![Captura de pantalla del cuadro de diálogo Agregar origen de datos](./media/vm-do-ten-things/Link_Blob_to_ADLA_v2.PNG)

En Visual Studio, puede leer datos de Blob Storage, manipular los datos, realizar ingeniería de características y enviar los datos resultantes a Azure Data Lake o Azure Blob Storage. Cuando haga referencia a los datos de Blob Storage, use **wasb://** . Cuando haga referencia a los datos de Azure Data Lake, use **swbhdfs://** .

Puede usar las siguientes consultas U-SQL en Visual Studio:

```usql
@a =
    EXTRACT medallion string,
            hack_license string,
            vendor_id string,
            rate_code string,
            store_and_fwd_flag string,
            pickup_datetime string,
            dropoff_datetime string,
            passenger_count int,
            trip_time_in_secs double,
            trip_distance double,
            pickup_longitude string,
            pickup_latitude string,
            dropoff_longitude string,
            dropoff_latitude string

    FROM "wasb://<Container name>@<Azure Blob Storage Account Name>.blob.core.windows.net/<Input Data File Name>"
    USING Extractors.Csv();

@b =
    SELECT vendor_id,
    COUNT(medallion) AS cnt_medallion,
    SUM(passenger_count) AS cnt_passenger,
    AVG(trip_distance) AS avg_trip_dist,
    MIN(trip_distance) AS min_trip_dist,
    MAX(trip_distance) AS max_trip_dist,
    AVG(trip_time_in_secs) AS avg_trip_time
    FROM @a
    GROUP BY vendor_id;

OUTPUT @b   
TO "swebhdfs://<Azure Data Lake Storage Account Name>.azuredatalakestore.net/<Folder Name>/<Output Data File Name>"
USING Outputters.Csv();

OUTPUT @b   
TO "wasb://<Container name>@<Azure Blob Storage Account Name>.blob.core.windows.net/<Output Data File Name>"
USING Outputters.Csv();
```

Después de enviar la consulta al servidor, aparecerá un diagrama con el estado del trabajo.

![Captura de pantalla del diagrama de estado del trabajo](./media/vm-do-ten-things/USQL_Job_Status.PNG)

#### <a name="query-data-in-data-lake-u-sql"></a>Consulta de datos en Data Lake: U-SQL

Después de que el conjunto de datos se haya introducido en Azure Data Lake, se puede usar el [lenguaje U-SQL](../../data-lake-analytics/data-lake-analytics-u-sql-get-started.md) para explorar los datos y realizar consultas en ellos. El lenguaje U-SQL es parecido a T-SQL, pero combina algunas características de C# para que los usuarios puedan escribir módulos personalizados y funciones definidas por el usuario. Puede utilizar los scripts del paso anterior.

Una vez enviada la consulta al servidor, aparece tripdata_summary.CSV en el Explorador de Azure Data Lake. Puede obtener una vista previa de los datos si hace clic con el botón derecho en el archivo.

![Captura de pantalla del archivo .csv en el Explorador de Data Lake](./media/vm-do-ten-things/USQL_create_summary.png)

Aparece la información del archivo:

![Captura de pantalla de la información de resumen de archivo](./media/vm-do-ten-things/USQL_tripdata_summary.png)

### <a name="azure-sql-data-warehouse-and-databases"></a>Azure SQL Data Warehouse y bases de datos
Azure SQL Data Warehouse es un almacén de datos elástico que funciona como un servicio con una experiencia de SQL Server empresarial.

Para aprovisionar Azure SQL Data Warehouse, siga las instrucciones que se proporcionan en [este artículo](../../sql-data-warehouse/sql-data-warehouse-get-started-provision.md). Una vez que aprovisione SQL Data Warehouse, puede usar [este tutorial](../team-data-science-process/sqldw-walkthrough.md) para cargar, explorar y modelar los datos mediante los datos de SQL Data Warehouse.

#### <a name="azure-cosmos-db"></a>Azure Cosmos DB
Azure Cosmos DB es una base de datos NoSQL en la nube. Puede usarlo para trabajar con documentos como JSON, así como para almacenar y consultar dichos documentos.

Siga estos pasos de requisitos previos para acceder a Azure Cosmos DB desde DSVM:

1. El SDK de Python de Azure Cosmos DB ya está instalado en DSVM. Para actualizarlo, ejecute ```pip install pydocumentdb --upgrade``` desde un símbolo del sistema.
2. Cree una base de datos y una cuenta de Azure Cosmos DB en [Azure Portal](https://portal.azure.com).
3. Descargue la herramienta de migración de datos de Azure Cosmos DB desde el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=53595) y extráigala en el directorio que quiera.
4. Importe los datos JSON (datos de volcanes) almacenados en un [blob público](https://data.humdata.org/dataset/a60ac839-920d-435a-bf7d-25855602699d/resource/7234d067-2d74-449a-9c61-22ae6d98d928/download/volcano.json) en Azure Cosmos DB con los siguientes parámetros de comando en la herramienta de migración. (Use dtui.exe desde el directorio en el que ha instalado la herramienta de migración de datos de Azure Cosmos DB). Especifique las ubicaciones de origen y destino con estos parámetros:
   
    `/s:JsonFile /s.Files:https://data.humdata.org/dataset/a60ac839-920d-435a-bf7d-25855602699d/resource/7234d067-2d74-449a-9c61-22ae6d98d928/download/volcano.json /t:DocumentDBBulk /t.ConnectionString:AccountEndpoint=https://[DocDBAccountName].documents.azure.com:443/;AccountKey=[[KEY];Database=volcano /t.Collection:volcano1`

Después de importar los datos, puede ir a Jupyter y abrir el cuaderno titulado *DocumentDBSample*. Contiene el código de Python para acceder a Azure Cosmos DB y realizar algunas consultas básicas. Para obtener más información sobre Azure Cosmos DB, visite la [página de documentación](https://docs.microsoft.com/azure/cosmos-db/) del servicio.

## <a name="use-power-bi-reports-and-dashboards"></a>Uso de informes y paneles de Power BI 
Puede visualizar el archivo JSON denominado Volcano del ejemplo de Azure Cosmos DB anterior en Power BI Desktop para obtener información visual sobre los datos. En este [artículo de Power BI](../../cosmos-db/powerbi-visualize.md)encontrará una explicación detallada de los pasos que se deben seguir. Los pasos generales son los siguientes:

1. Abra Power BI Desktop y seleccione **Obtener datos**. Especifique la URL como: `https://cahandson.blob.core.windows.net/samples/volcano.json`.
2. Debería ver los registros JSON importados como una lista. Convierta la lista en una tabla para que Power BI pueda trabajar con ella.
4. Seleccione el icono de expandir (flecha) para expandir las columnas.
5. Observe que la ubicación es un campo de **Registro**. Expanda el registro y seleccione solo las coordenadas. **Coordenada** es una columna de la lista.
6. Agregue una nueva columna para convertir la columna de coordenadas de la lista en una columna **LatLong** separada por comas. Concatene los dos elementos del campo de lista de coordenadas mediante la fórmula ```Text.From([coordinates]{1})&","&Text.From([coordinates]{0})```.
7. Convierta la columna **Elevación** en decimal y seleccione los botones **Cerrar** y **Aplicar**.

En lugar de los pasos anteriores, puede pegar el código siguiente. Incluye en el script los pasos usados en el editor avanzado de Power BI para escribir las transformaciones de datos en un lenguaje de consulta.

```pqfl
let
    Source = Json.Document(Web.Contents("https://cahandson.blob.core.windows.net/samples/volcano.json")),
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}, {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}),
    #"Expanded Location" = Table.ExpandRecordColumn(#"Expanded Column1", "Location", {"coordinates"}, {"coordinates"}),
    #"Added Custom" = Table.AddColumn(#"Expanded Location", "LatLong", each Text.From([coordinates]{1})&","&Text.From([coordinates]{0})),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Elevation", type number}})
in
    #"Changed Type"
```

Ya tiene los datos en el modelo de datos de Power BI. La instancia de Power BI Desktop debe aparecer de la siguiente forma:

![Power BI Desktop](./media/vm-do-ten-things/PowerBIVolcanoData.png)

Puede empezar a crear informes y visualizaciones con el modelo de datos. Para generar un informe, puede seguir los pasos de [este artículo de Power BI](../../cosmos-db/powerbi-visualize.md#build-the-reports).

## <a name="scale-the-dsvm-dynamically"></a>Escalado dinámico de DSVM 
Puede escalar y reducir verticalmente la máquina DSVM para satisfacer las necesidades del proyecto. Si no necesita usar la máquina virtual por la noche o durante los fines de semana, puede apagarla desde [Azure Portal](https://portal.azure.com).

> [!NOTE]
> Incurrirá en gastos de proceso si solo usa el botón de apagado del sistema operativo de la máquina virtual.  
> 
> 

Es posible que deba controlar algún análisis a gran escala y que necesite más capacidad de CPU, memoria o disco. De ser así, puede encontrar una variedad de tamaños de máquina virtual en términos de núcleos de CPU, instancias basadas en GPU para aprendizaje profundo, capacidad de memoria y tipos de disco (incluidas las unidades de estado sólido) que satisfarán sus necesidades presupuestarias y de proceso. En la página [Precios de Azure Virtual Machines](https://azure.microsoft.com/pricing/details/virtual-machines/) encontrará la lista completa de máquinas virtuales, además de sus precios por hora de proceso.

Del mismo modo, podría disminuir su necesidad de capacidad de procesamiento de máquinas virtuales. (Por ejemplo: ha migrado una carga de trabajo principal a un clúster de Hadoop o Spark). Después, puede reducir verticalmente el clúster desde [Azure Portal](https://portal.azure.com) e ir a la configuración de la instancia de máquina virtual. 

## <a name="add-more-tools"></a>Adición de más herramientas
Las herramientas precompiladas en DSVM pueden abordar muchas necesidades comunes de análisis de datos. Esto le ahorra tiempo, ya que no tiene que instalar y configurar los entornos uno por uno. También le ahorra dinero, ya que solo paga por los recursos que usa.

Para mejorar su entorno de análisis, puede usar otros servicios de datos y análisis de Azure que se han descrito en este artículo. En algunos casos, podría necesitar herramientas adicionales, incluidas algunas herramientas propias de asociados. Tiene acceso administrativo completo en la máquina virtual para instalar las nuevas herramientas que necesite. También puede instalar paquetes adicionales de Python y R que no estén instalados previamente. En el caso de Python, puede usar ```conda``` o ```pip```. En cuanto a R, puede usar ```install.packages()``` en la consola de R o recurrir al IDE y seleccionar **Paquetes** > **Instalar paquetes**.

## <a name="deep-learning"></a>Aprendizaje profundo

Además de los ejemplos basados en marcos, también obtiene un conjunto de tutoriales completos que se han validado en DSVM, con los que puede iniciar el desarrollo de aplicaciones de aprendizaje profundo en ámbitos como la comprensión de lenguajes o de imágenes y texto.   


- [Ejecución de redes neuronales en distintos marcos](https://github.com/ilkarman/DeepLearningFrameworks): en este tutorial se muestra cómo migrar código de un marco a otro. También se muestra cómo comparar el rendimiento de los modelos y el entorno de ejecución en varios marcos. 

- [Una guía paso a paso para compilar una solución integral que detecte productos dentro de las imágenes](https://github.com/Azure/cortana-intelligence-product-detection-from-images): la detección de imágenes es una técnica que puede localizar y clasificar objetos dentro de las imágenes. Esta tecnología tiene el potencial de aportar grandes ventajas a muchos ámbitos comerciales de uso real. Por ejemplo, los minoristas pueden aplicar esta técnica para determinar qué producto ha seleccionado un cliente de una estantería. A su vez, esta información permite que las tiendas administren el inventario de productos. 

- [Aprendizaje profundo para audio](https://blogs.technet.microsoft.com/machinelearning/2018/01/30/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure/): en este tutorial se muestra cómo entrenar un modelo de aprendizaje profundo para detectar eventos de audio en el [conjunto de datos de sonidos urbanos](https://serv.cusp.nyu.edu/projects/urbansounddataset/urbansound8k.html). También se proporciona información general sobre cómo trabajar con datos de audio.

- [Clasificación de documentos de texto](https://github.com/anargyri/lstm_han): en este tutorial se muestra cómo compilar y entrenar dos arquitecturas de redes neuronales: red de atención jerárquica y red de memoria a corto y largo plazo (LSTM). Estas redes neurales usan la API de Keras de aprendizaje profundo para clasificar documentos de texto. Keras es un front-end para tres de los marcos de aprendizaje profundo más populares: Microsoft Cognitive Toolkit, TensorFlow y Theano.

## <a name="summary"></a>Resumen
En este artículo se han descrito algunas de las cosas que puede hacer en Microsoft Data Science Virtual Machine, pero puede hacer muchas más para convertir DSVM en un entorno de análisis eficaz.

