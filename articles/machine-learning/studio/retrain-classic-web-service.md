---
title: 'ML Studio (clásico): reentrenamiento de servicios web clásicos (Azure)'
description: Aprenda a entrenar de nuevo un modelo y a actualizar el servicio web clásico para que utilice este modelo en Azure Machine Learning Studio (clásico).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: peterclu
ms.author: peterlu
ms.custom: seodec18, previous-ms.author=yahajiza, previous-author=YasinMSFT, devx-track-csharp
ms.date: 02/14/2019
ms.openlocfilehash: dfc959984147d89ca5bfe68ed103a89b22f75501
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020695"
---
# <a name="retrain-and-deploy-a-classic-studio-classic-web-service"></a>Repetición del entrenamiento e implementación de un servicio web de Studio (clásico)

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico) ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Una forma de garantizar que los modelos de Machine Learning siguen siendo precisos y se basan en los datos más adecuados disponibles, es volver a entrenarlos. En este artículo se muestra cómo volver a entrenar un servicio web de Studio clásico. Para más información acerca de cómo volver a entrenar un nuevo servicio web de Studio (clásico), [consulte este artículo de procedimientos](retrain-machine-learning-model.md).

## <a name="prerequisites"></a>Requisitos previos

En este artículo se asume que tiene un experimento de reciclaje y un experimento de predicción. Estos pasos se explican en [Reciclaje de un modelo de Azure Machine Learning Studio](/azure/machine-learning/studio/retrain-machine-learning-model). Sin embargo, en lugar de implementar el modelo de Machine Learning como un servicio web nuevo, implementará el experimento de predicción como un servicio web clásico.
     
## <a name="add-a-new-endpoint"></a>Adición de un punto de conexión nuevo

El servicio web de predicción que ha implementado contiene un punto de conexión de puntuación predeterminado que se mantiene sincronizado con el entrenamiento original y el modelo entrenado de experimentos de puntuación. Para actualizar el servicio web a un nuevo modelo entrenado, debe crear un nuevo punto de conexión para la puntuación.

Hay dos formas en que puede agregar un nuevo punto de conexión a un servicio web:

* De manera programática
* Mediante el portal de Azure Web Services

> [!NOTE]
> Asegúrese de agregar el punto de conexión al servicio web predictivo y no al de entrenamiento. Si ha implementado correctamente un servicio web predictivo y otro de entrenamiento, debería ver dos servicios web independientes. El servicio web predictivo debe terminar con "[predictive exp.]".
>

### <a name="programmatically-add-an-endpoint"></a>Incorporación de un punto de conexión mediante programación

También puede agregar puntos de conexión de puntuación mediante el código de ejemplo proporcionado en este [repositorio de GitHub](https://github.com/hning86/azuremlps#add-amlwebserviceendpoint).

### <a name="use-the-azure-web-services-portal-to-add-an-endpoint"></a>Uso del portal de Azure Web Services para agregar un punto de conexión

1. En Machine Learning Studio (clásico), en la columna de navegación izquierda, haga clic en Servicios web.
1. En la parte inferior del panel de servicios web, haga clic en **Manage endpoints preview**(Administrar versión preliminar de puntos de conexión).
1. Haga clic en **Agregar**.
1. Escriba un nombre y una descripción para el nuevo punto de conexión. Seleccione el nivel de registro y si los datos de ejemplo están habilitados. Para más información sobre los registros, vea [Habilitar el registro para los servicios web de Machine Learning](web-services-logging.md).

## <a name="update-the-added-endpoints-trained-model"></a>Actualización del modelo entrenado del punto de conexión agregado

### <a name="retrieve-patch-url"></a>Recuperación de URL de PATCH

Para obtener el valor correcto de URL de PATCH mediante el portal web, siga estos pasos:

1. Inicie sesión en el portal [Servicios web Azure Machine Learning](https://services.azureml.net/).
1. Haga clic en **Web Services** (Servicios web) o **Classic Web Services** (Servicios web clásicos) en la parte superior.
1. Haga clic en el servicio web de puntuación con el que trabaja (si no ha modificado el nombre predeterminado del servicio web, terminará en "[Scoring Exp.]").
1. Haga clic en **+ NUEVO**.
1. Una vez agregado el punto de conexión, haga clic en su nombre.
1. En la dirección URL de **Revisión**, haga clic en **API Help** (Ayuda de API) para abrir la página de ayuda de la aplicación de revisiones.

> [!NOTE]
> Si ha agregado el punto de conexión al servicio web de entrenamiento en lugar de al de predicción, verá el siguiente mensaje de error al hacer clic en el vínculo **Update Resource** (Actualizar recurso): "Sorry, but this feature is not supported or available in this context." (Lo sentimos, pero esta característica no se admite ni está disponible en este contexto). Este servicio web no tiene ningún recurso actualizable. Sentimos las molestias. Estamos trabajando en mejorar este flujo de trabajo."
>

La página de ayuda sobre PATCH contiene el valor de PATCH URL que debe usar y proporciona un código de ejemplo que puede usar para realizar la llamada.

![PATCH URL.](./media/retrain-classic/ml-help-page-patch-url.png)

### <a name="update-the-endpoint"></a>Actualización del punto de conexión

Ahora puede usar el modelo entrenado para actualizar el punto de conexión de puntuación que creó anteriormente.

El código de ejemplo siguiente muestra cómo utilizar *BaseLocation*, *RelativeLocation*, *SasBlobToken* y el valor de PATCH URL para actualizar el punto de conexión.

```csharp
private async Task OverwriteModel()
{
    var resourceLocations = new
    {
        Resources = new[]
        {
            new
            {
                Name = "Census Model [trained model]",
                Location = new AzureBlobDataReference()
                {
                    BaseLocation = "https://esintussouthsus.blob.core.windows.net/",
                    RelativeLocation = "your endpoint relative location", //from the output, for example: "experimentoutput/8946abfd-79d6-4438-89a9-3e5d109183/8946abfd-79d6-4438-89a9-3e5d109183.ilearner"
                    SasBlobToken = "your endpoint SAS blob token" //from the output, for example: "?sv=2013-08-15&sr=c&sig=37lTTfngRwxCcf94%3D&st=2015-01-30T22%3A53%3A06Z&se=2015-01-31T22%3A58%3A06Z&sp=rl"
                }
            }
        }
    };

    using (var client = new HttpClient())
    {
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

        using (var request = new HttpRequestMessage(new HttpMethod("PATCH"), endpointUrl))
        {
            request.Content = new StringContent(JsonConvert.SerializeObject(resourceLocations), System.Text.Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.SendAsync(request);

            if (!response.IsSuccessStatusCode)
            {
                await WriteFailedResponse(response);
            }

            // Do what you want with a successful response here.
        }
    }
}
```

Se puede obtener *apiKey* y *endpointUrl* para la llamada desde el panel del punto de conexión.

El valor del parámetro *Name* de *Resources* debe coincidir con el nombre del recurso del modelo entrenado guardado en el experimento predictivo. Para obtener el nombre del recurso:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Machine Learning**en el menú izquierdo.
1. En Nombre, haga clic en el área de trabajo y, a continuación, haga clic en **Servicios web**.
1. En Nombre, haga clic en **Census Model [predictive exp.]** (Modelo de censo [exp. predictivo]).
1. Haga clic en el nuevo punto de conexión que ha agregado.
1. En el panel del punto de conexión, haga clic en **Actualizar recurso**.
1. En la página de documentación de la API de actualización de recurso para el servicio web, encontrará el **nombre del recurso** en **Updatable Resources** (Recursos actualizables).

Si el token de SAS expira antes de que termine de actualizar el punto de conexión, debe realizar una operación GET con el Id. del trabajo para obtener un nuevo token.

Si el código se ha ejecutado correctamente, el nuevo punto de conexión debería comenzar a utilizar el modelo reentrenado en aproximadamente 30 segundos.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo administrar los servicios web o realizar un seguimiento de la ejecución de varios experimentos, consulte los artículos siguientes:

* [Exploración del portal de Web Services](manage-new-webservice.md)
* [Administración de iteraciones de experimentos](manage-experiment-iterations.md)