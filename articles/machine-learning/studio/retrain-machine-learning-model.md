---
title: 'ML Studio (clásico): reciclaje de servicios web (Azure)'
description: Aprenda a actualizar un servicio web para usar un modelo de aprendizaje automático recién entrenado en Azure Machine Learning Studio (clásico).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 02/14/2019
ms.openlocfilehash: 9acb9db0024ea1601f82b96376f084536fbbeabb
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431867"
---
# <a name="retrain-and-deploy-a-machine-learning-model"></a>Nuevo entrenamiento e implementación de un modelo de aprendizaje automático

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Una forma de garantizar que los modelos de aprendizaje automático siguen siendo precisos y se basan en los datos más adecuados disponibles, es volver a entrenarlos. En este artículo se muestra cómo volver a entrenar e implementar un modelo de aprendizaje automático como un nuevo servicio web en Studio (clásico). Si está pensando en volver a entrenar un servicio web clásico, [vea este artículo de procedimientos.](retrain-classic-web-service.md)

En este artículo se da por supuesto que ya tiene implementado un servicio web predictivo. Si no es así, [aprenda a implementar aquí un servicio web de Studio (clásico).](deploy-a-machine-learning-web-service.md)

Siga estos pasos para volver a entrenar e implementar un nuevo servicio web de aprendizaje automático:

1. Implementar un **servicio web de nuevo entrenamiento**
1. Entrenar un modelo nuevo mediante el **servicio web de nuevo entrenamiento**
1. Actualizar el **experimento predictivo** existente para usar el nuevo modelo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="deploy-the-retraining-web-service"></a>Implementar el servicio web de reciclaje

Un servicio web de nuevo entrenamiento le permite volver a entrenar el modelo con un nuevo conjunto de parámetros, por ejemplo, nuevos datos, y guardarlo para más adelante. Cuando se conecta una **salida de servicio web** a un **modelo de entrenamiento**, el experimento de entrenamiento genera un nuevo modelo para usar.

Utilice los pasos siguientes para implementar un servicio web de nuevo entrenamiento:

1. Conecte un módulo de **entrada de servicio web** a la entrada de datos. Normalmente, quiere asegurarse de que los datos de entrada se procesen de la misma forma que los datos de entrenamiento original.
1. Conecte un módulo de **salida de servicio web** a la salida del **modelo de entrenamiento**.
1. Si tiene un módulo de **evaluación de modelo**, puede conectar un módulo de **salida de servicio web** para generar los resultados de evaluación.
1. Ejecute el experimento.

    Después de ejecutar el experimento, el flujo de trabajo resultante debe ser similar a la imagen siguiente:

    ![Flujo de trabajo resultante](media/retrain-machine-learning/machine-learning-retrain-models-programmatically-IMAGE04.png)

    Ahora, implementará el experimento de entrenamiento como un servicio web de nuevo entrenamiento que genera un modelo entrenado y los resultados de evaluación del modelo.

1. En la parte inferior del lienzo del experimento, haga clic en **Set Up Web Service** (Configurar servicio web).
1. Seleccione **Deploy Web Service [New]** (Implementar servicio web [Nuevo]). El portal de servicios web de Azure Machine Learning se abre en la página **Deploy Web service** (Implementar servicio web).
1. Escriba un nombre para el servicio web y elija un plan de pago.
1. Seleccione **Implementar**.

## <a name="retrain-the-model"></a>Nuevo entrenamiento del modelo

En este ejemplo, se utiliza C# para crear la aplicación de reciclado. También puede utilizar el código de ejemplo de Python o R para realizar esta tarea.

Use los pasos siguientes para llamar a las API de nuevo entrenamiento:

1. Cree una aplicación de consola en C# mediante Visual Studio: **Nuevo** > **Proyecto** > **Visual C#**  > **Escritorio clásico de Windows** > **Aplicación de consola (.NET Framework)** .
1. Inicie sesión en el portal de servicio web Machine Learning.
1. Haga clic en el servicio web con el que está trabajando.
1. Haga clic en **Consume**(Consumo).
1. En la parte inferior de la página **Consume** (Consumo), en la sección **Código de ejemplo**, haga clic en **Batch**.
1. Copie el código C# de ejemplo para la ejecución por lotes y péguelo en el archivo Program.cs. Asegúrese de que el espacio de nombres permanece intacto.

Agregue el paquete NuGet Microsoft.AspNet.WebApi.Client tal como se especifica en los comentarios. Para agregar la referencia a Microsoft.WindowsAzure.Storage.dll, puede que deba instalar la [biblioteca de cliente para servicios de Azure Storage](https://www.nuget.org/packages/WindowsAzure.Storage).

La siguiente captura de pantalla muestra la página **Consumo** en el portal de servicios de web de Azure Machine Learning.

![Página Consumo](media/retrain-machine-learning/machine-learning-retrain-models-consume-page.png)

### <a name="update-the-apikey-declaration"></a>Actualización de la declaración de apikey

Localice la declaración de **apikey**:

```csharp
const string apiKey = "abc123"; // Replace this with the API key for the web service
```

En la sección **Basic consumption info** (Información básica sobre consumo) de la página **Consume** (Consumo), localice la clave principal y cópiela en la declaración de **apikey**.

### <a name="update-the-azure-storage-information"></a>Actualización de la información de Azure Storage

El código de ejemplo de BES carga un archivo desde una unidad local (por ejemplo "C:\temp\CensusIpnput.csv") en Azure Storage, lo procesa y escribe los resultados de nuevo en Azure Storage.

1. Inicie sesión en Azure Portal.
1. En la columna de navegación izquierda, haga clic en **Más servicios**, busque **Cuentas de almacenamiento** y selecciónelo.
1. En la lista de cuentas de almacenamiento, seleccione una para almacenar el modelo reciclado.
1. En la columna de navegación izquierda, haga clic en **Claves de acceso**.
1. Copie y guarde el valor de **Primary Access Key**.
1. En la columna de navegación izquierda, haga clic en **Blobs**.
1. Seleccione un contenedor existente o cree uno nuevo y guarde el nombre.

Busque las declaraciones *StorageAccountName*, *StorageAccountKey* y *StorageContainerName* y actualice los valores guardados en el portal.

```csharp
const string StorageAccountName = "mystorageacct"; // Replace this with your Azure storage account name
const string StorageAccountKey = "a_storage_account_key"; // Replace this with your Azure Storage key
const string StorageContainerName = "mycontainer"; // Replace this with your Azure Storage container name
```

También deberá asegurarse de que el archivo de entrada está disponible en la ubicación que especifique en el código.

### <a name="specify-the-output-location"></a>Especifique la ubicación de salida.

Al especificar la ubicación de salida en la carga útil de solicitud, la extensión del archivo especificado en *RelativeLocation* debe especificarse como `ilearner`.

```csharp
Outputs = new Dictionary<string, AzureBlobDataReference>() {
    {
        "output1",
        new AzureBlobDataReference()
        {
            ConnectionString = storageConnectionString,
            RelativeLocation = string.Format("{0}/output1results.ilearner", StorageContainerName) /*Replace this with the location you want to use for your output file and a valid file extension (usually .csv for scoring results or .ilearner for trained models)*/
        }
    },
```

El siguiente es un ejemplo de salida de nuevo entrenamiento:

![Salida de reentrenamiento](media/retrain-machine-learning/machine-learning-retrain-models-programmatically-IMAGE06.png)

### <a name="evaluate-the-retraining-results"></a>Evaluación de los resultados de reciclaje

Al ejecutar la aplicación, la salida incluye la dirección URL y el token de firmas de acceso compartido que son necesarios para tener acceso a los resultados de evaluación.

Podrá ver los resultados de rendimiento del modelo entrenado de nuevo si combina *BaseLocation*, *RelativeLocation* y *SasBlobToken* de los resultados de salida de *output2* y pega la dirección URL completa en la barra de direcciones del explorador.

Examine los resultados para determinar si el modelo recién entrenado funciona mejor que el ya existente.

Guarde *BaseLocation*, *RelativeLocation* y *SasBlobToken* de los resultados de salida.

## <a name="update-the-predictive-experiment"></a>Actualización del experimento predictivo

### <a name="sign-in-to-azure-resource-manager"></a>Iniciar sesión en Azure Resource Manager

Primero, inicie sesión en la cuenta de Azure en el entorno de PowerShell mediante el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

### <a name="get-the-web-service-definition-object"></a>Obtener el objeto de definición del servicio web

A continuación, obtenga el objeto de definición de servicio web llamando al cmdlet [Get-AzMlWebService](https://docs.microsoft.com/powershell/module/az.machinelearning/get-azmlwebservice).

```azurepowershell
$wsd = Get-AzMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'
```

Para determinar el nombre del grupo de recursos de un servicio web existente, ejecute el cmdlet Get-AzMlWebService sin ningún parámetro para mostrar los servicios web en la suscripción. Busque el servicio web y luego examine su identificador de servicio web. El nombre del grupo de recursos es el cuarto elemento del identificador, justo después del elemento *resourceGroups* . En el ejemplo siguiente, el nombre del grupo de recursos es Default-MachineLearning-SouthCentralUS.

```azurepowershell
Properties : Microsoft.Azure.Management.MachineLearning.WebServices.Models.WebServicePropertiesForGraph
Id : /subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237
Name : RetrainSamplePre.2016.8.17.0.3.51.237
Location : South Central US
Type : Microsoft.MachineLearning/webServices
Tags : {}
```

Como alternativa, para determinar el nombre del grupo de recursos de un servicio web existente, inicie sesión en el portal de servicios web de Azure Machine Learning. Seleccione el servicio web. El nombre del grupo de recursos es el quinto elemento de la dirección URL del servicio web, justo después del elemento *resourceGroups* . En el ejemplo siguiente, el nombre del grupo de recursos es Default-MachineLearning-SouthCentralUS.

`https://services.azureml.net/subscriptions/<subscription ID>/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/RetrainSamplePre.2016.8.17.0.3.51.237`

### <a name="export-the-web-service-definition-object-as-json"></a>Exportar el objeto de definición de servicio web como JSON

Para modificar la definición del modelo entrenado a fin de usar el modelo recién entrenado, primero debe usar el cmdlet [Export-AzMlWebService](https://docs.microsoft.com/powershell/module/az.machinelearning/export-azmlwebservice) para exportar a un archivo con formato JSON.

```azurepowershell
Export-AzMlWebService -WebService $wsd -OutputFile "C:\temp\mlservice_export.json"
```

### <a name="update-the-reference-to-the-ilearner-blob"></a>Actualizar la referencia al blob ilearner

En los recursos, busque [modelo entrenado], actualice el valor de *uri* en el nodo *locationInfo* con el identificador URI del blob ilearner. El identificador URI se genera mediante la combinación de *BaseLocation* y *RelativeLocation* a partir del resultado de la llamada de reentrenamiento de BES.

```json
"asset3": {
    "name": "Retrain Sample [trained model]",
    "type": "Resource",
    "locationInfo": {
        "uri": "https://mltestaccount.blob.core.windows.net/azuremlassetscontainer/baca7bca650f46218633552c0bcbba0e.ilearner"
    },
    "outputPorts": {
        "Results dataset": {
            "type": "Dataset"
        }
    }
},
```

### <a name="import-the-json-into-a-web-service-definition-object"></a>Importar JSON en un objeto de definición de servicio web

Use el cmdlet [Import-AzMlWebService](https://docs.microsoft.com/powershell/module/az.machinelearning/import-azmlwebservice) para convertir el archivo JSON modificado en un objeto de definición de servicio web que puede usar para actualizar el experimento predictivo.

```azurepowershell
$wsd = Import-AzMlWebService -InputFile "C:\temp\mlservice_export.json"
```

### <a name="update-the-web-service"></a>Actualizar el servicio web

Por último, utilice el cmdlet [Update-AzMlWebService](https://docs.microsoft.com/powershell/module/az.machinelearning/update-azmlwebservice) para actualizar el experimento predictivo.

```azurepowershell
Update-AzMlWebService -Name 'RetrainSamplePre.2016.8.17.0.3.51.237' -ResourceGroupName 'Default-MachineLearning-SouthCentralUS'
```

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo administrar los servicios web o realizar un seguimiento de la ejecución de varios experimentos, consulte los artículos siguientes:

* [Exploración del portal de Web Services](manage-new-webservice.md)
* [Administración de iteraciones de experimentos](manage-experiment-iterations.md)
