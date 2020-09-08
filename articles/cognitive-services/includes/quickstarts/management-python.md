---
title: 'Inicio rápido: Biblioteca cliente de Administración de Azure para Python'
description: En este inicio rápido, comience con la biblioteca cliente de Administración de Azure para Python.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: 7cc76ab7c9ce2191a54d5bd61282267467603694
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321591"
---
[Documentación de referencia](https://docs.microsoft.com/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices) | [Paquete (PyPi)](https://pypi.org/project/azure-mgmt-cognitiveservices/) | [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-mgmt-cognitiveservices/tests)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure válida: [cree una de manera gratuita](https://azure.microsoft.com/free/).
* [Python 3.x](https://www.python.org/)

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree una aplicación de Python en el editor o IDE que prefiera y vaya al proyecto en una ventana de consola.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Puede instalar la biblioteca cliente con lo siguiente:

```console
pip install azure-mgmt-cognitiveservices
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.

### <a name="import-libraries"></a>Importación de bibliotecas

Abra el script de Python e importe las siguientes bibliotecas.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Agregue los campos siguientes a la raíz del script y rellene sus valores, para lo que debe usar la entidad de servicio que creó y la información de su cuenta de Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_constants)]

Luego, agregue el siguiente código para construir un objeto **CognitiveServicesManagementClient**. Este objeto es necesario para todas las operaciones de Administración de Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_auth)]

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

### <a name="choose-a-service-and-pricing-tier"></a>Elección de un servicio y un plan de tarifa

Al crear un recurso, necesitará conocer el "tipo" de servicio que desea usar y el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) (o la SKU) que desee. Usará esta y otra información como parámetros al crear el recurso. La siguiente función enumera los "tipos" de Cognitive Services disponibles.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

Para crear un recurso de Cognitive Services y suscribirse a él, use la función **Create**. Esta función agrega un nuevo recurso facturable al grupo de recursos que se pasa. Cuando cree el recurso, deberá conocer el "tipo" de servicio que desea usar, el plan de tarifa (o SKU) y la ubicación de Azure. La siguiente función usa todos estos valores como argumentos y crea un recurso.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_create)]

## <a name="view-your-resources"></a>Visualización de los recursos

Para ver todos los recursos de su cuenta de Azure (de todos los grupos de recursos), use la siguiente función:

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_list)]

## <a name="delete-a-resource"></a>Eliminar un recurso

La función siguiente elimina el recurso especificado del grupo de recursos dado.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_delete)]

## <a name="call-management-functions"></a>Funciones de administración de llamadas

Agregue el siguiente código al final del script para llamar a las funciones anteriores. En este código se muestran los recursos disponibles, se crea un recurso de ejemplo, se enumeran los recursos que posee y, después, se elimina el recurso de ejemplo.

[!code-python[](~/cognitive-services-quickstart-code/python/azure_management_service/create_delete_resource.py?name=snippet_calls)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde la línea de comandos con el comando `python`.

```console
python <your-script-name>.py
```

## <a name="see-also"></a>Consulte también

* [Documentación de referencia del SDK de Administración de Azure](https://docs.microsoft.com/python/api/azure-mgmt-cognitiveservices/azure.mgmt.cognitiveservices?view=azure-python)
* [¿Qué es Azure Cognitive Services?](../../Welcome.md)
* [Autenticación de solicitudes en Azure Cognitive Services](../../authentication.md)
* [Creación de un recurso mediante Azure Portal](../../cognitive-services-apis-create-account.md)