---
title: 'Inicio rápido: Implementación de una aplicación con el portal de LUIS'
description: En este inicio rápido se muestra cómo implementar una aplicación mediante la creación de un recurso de punto de conexión de predicción, la asignación del recurso, el entrenamiento y la publicación de la aplicación.
ms.topic: quickstart
ms.date: 05/06/2020
ms.openlocfilehash: 3d12fa2956f962fe7a05b9639c1d32a7069635f9
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87336383"
---
# <a name="quickstart-deploy-an-app-in-the-luis-portal"></a>Inicio rápido: Implementación de una aplicación en el portal de LUIS

Una vez que la aplicación de LUIS está lista para devolver predicciones de expresiones a una aplicación cliente (por ejemplo, un bot de chat), es preciso implementar la aplicación en el punto de conexión de la predicción.

En esta guía de inicio rápido, aprenderá a implementar una aplicación. Cree un recurso de punto de conexión de predicción, asigne el recurso a la aplicación, entrene la aplicación y publíquela.

## <a name="prerequisites"></a>Prerrequisitos

* Obtener una [suscripción de Azure](https://azure.microsoft.com/free).
* Completar el [inicio rápido del portal anterior](get-started-portal-build-app.md) o [descargar e importar la aplicación](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/portal-build-app.json).
* Si tiene aplicaciones anteriores a la autenticación de recursos de Azure, [migrar a un recurso de Azure](luis-migration-authoring.md). Algunas páginas del portal tienen un aspecto diferente cuando la autenticación de correo electrónico está en vigor.

<a name="create-the-endpoint-resource"></a>

[!INCLUDE [Create LUIS resource](includes/create-luis-resource.md)]

## <a name="assign-the-resource-key-to-the-luis-app-in-the-luis-portal"></a>Asignar la clave de recurso a la aplicación de LUIS en el portal de LUIS

Cada vez que crea un recurso de predicción de consultas o de creación para LUIS, es preciso asignarlo a la aplicación de LUIS. Una vez que lo asigne, no tendrá que volver a realizar este paso, a menos que cree otro recurso. Puede crear un recurso para expandir las regiones de la aplicación o para admitir un mayor número de consultas de predicción.

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai) y elija la aplicación **myEnglishApp** en la lista de aplicaciones.

1. Seleccione **Manage** (Administrar) en el menú superior derecho y, después, seleccione **Azure Resources** (Recursos de Azure).

1. Para agregar LUIS, seleccione **Add prediction resource** (Agregar recurso de predicción).

    ![Para agregar el recurso de predicción de LUIS, seleccione Add prediction resource (Agregar recurso de predicción).](./media/get-started-portal-deploy-app/azure-resources-add-prediction-resource.png)

1. Seleccione el inquilino, la suscripción y el nombre del recurso. Seleccione **Asignar recurso**.

   > [!div class="mx-imgBorder"]
   > ![Asignar un recurso a la aplicación](./media/get-started-portal-deploy-app/assign-resource.png)

1. Siga los mismos pasos para agregar la clave de creación a la aplicación.

1. Busque la nueva fila en la tabla correspondiente al nuevo recurso de predicción y copie la dirección URL del punto de conexión. Se construye correctamente para realizar una solicitud `HTTP GET` al punto de conexión de LUIS API para una predicción.

> [!TIP]
> Si tiene previsto usar el aprendizaje activo para mejorar una aplicación LUIS, seleccione **Cambiar parámetros de consulta** y seleccione **Guardar registros**. Esta acción cambia la dirección URL, ya que agrega el parámetro QueryString `log=true`. Copie y utilice la dirección URL de consulta de ejemplo modificada al realizar consultas de predicción en el punto de conexión del runtime.

## <a name="train-the-app"></a>Entrenamiento de la aplicación

Si cambió la aplicación desde que la entrenó por última vez, [entrene](get-started-portal-build-app.md) la aplicación.

## <a name="publish-the-app-to-the-prediction-endpoint"></a>Publicación de la aplicación en el punto de conexión de predicción

[!INCLUDE [LUIS How to Publish steps](includes/howto-publish.md)]

## <a name="prediction-endpoint-request"></a>Solicitud de punto de conexión de predicción

En el portal, `query=` al final de la dirección URL es donde la expresión del usuario se anexa a la solicitud GET. Después de `query=`, escriba la misma expresión del usuario que se utilizó al final del inicio rápido anterior:

```Is there a form named hrf-234098```

Asegúrese de que la cadena de consulta incluya los siguientes pares:

* `show-all-intents=true`
* `verbose=true`

El explorador muestra la respuesta:

```JSON
{
    "query": "Is there a form named hrf-234098",
    "prediction": {
        "topIntent": "FindForm",
        "intents": {
            "FindForm": {
                "score": 0.9768753
            },
            "None": {
                "score": 0.0216071177
            }
        },
        "entities": {
            "Human Resources Form Number": [
                "hrf-234098"
            ],
            "$instance": {
                "Human Resources Form Number": [
                    {
                        "type": "Human Resources Form Number",
                        "text": "hrf-234098",
                        "startIndex": 22,
                        "length": 10,
                        "modelTypeId": 8,
                        "modelType": "Regex Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    }
}
```

Para ver el mismo nivel de información en el panel de prueba, debe publicar la aplicación. Una vez publicada la aplicación, seleccione **Comparar con el publicado** en el panel de prueba. Use **Show JSON view** (Mostrar vista de JSON) en el panel de prueba publicado para ver el mismos JSON que en el paso anterior. De esta forma, puede comparar los cambios en la aplicación en la que trabaja actualmente con una aplicación publicada en el punto de conexión.

[![Comparar la versión que se está editando con la versión publicada de la aplicación](./media/get-started-portal-deploy-app/compare-test-pane.png)](./media/get-started-portal-deploy-app/compare-test-pane.png#lightbox)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con este inicio rápido, seleccione **Mis aplicaciones** en el menú de navegación superior. Seleccione la casilla de la aplicación en la lista y seleccione **Delete** (Eliminar) en la barra de herramientas de contexto que está encima de la lista.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Identificación de intenciones y entidades habituales](luis-tutorial-prebuilt-intents-entities.md)
