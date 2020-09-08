---
title: 'Inicio rápido: Biblioteca cliente de Administración de Azure para Java'
description: En este inicio rápido, comenzará a usar la biblioteca cliente de Administración de Azure para Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/01/2020
ms.author: pafarley
ms.openlocfilehash: 346854d5990ac6861bd4eb93914bb1745b90bfa5
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321606"
---
[Documentación de referencia](https://docs.microsoft.com/java/api/com.microsoft.azure.management.cognitiveservices?view=azure-java-stable) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/mgmt-v2017_04_18/src/main/java/com/microsoft/azure/management/cognitiveservices/v2017_04_18) | [Paquete (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure/azure-mgmt-cognitiveservices)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure válida: [cree una de manera gratuita](https://azure.microsoft.com/free/).
* La versión actual de [Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.


[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-java-application"></a>Creación de una aplicación Java

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `gradle init` desde el directorio de trabajo. Este comando creará archivos de compilación esenciales para Gradle, como *build.gradle.kts*, que se usa en el runtime para crear y configurar la aplicación.

```console
gradle init --type basic
```

Cuando se le solicite que elija un **DSL**, seleccione **Kotlin**.

En el directorio de trabajo, ejecute el siguiente comando:

```console
mkdir -p src/main/java
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

En este inicio rápido se usa el administrador de dependencias Gradle. Puede encontrar la biblioteca de cliente y la información de otros administradores de dependencias en el [repositorio central de Maven](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer).

En el archivo *build.gradle.kts* del proyecto, incluya la biblioteca cliente como una instrucción `implementation`, junto con los complementos y la configuración necesarios.

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.microsoft.azure", name = "azure-mgmt-cognitiveservices", version = "1.10.0-beta")
}
```

### <a name="import-libraries"></a>Importación de bibliotecas

Vaya a la nueva carpeta **src/main/java** y cree un archivo denominado *Management.java*. Ábralo en el editor o el IDE que prefiera y agregue las siguientes instrucciones `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Autenticar el cliente

Agregue una clase en *Management.java* y, luego, agregue dentro de ella los siguientes campos y sus valores. Rellene sus valores mediante la entidad de servicio que creó y la otra información de la cuenta de Azure.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_constants)]

Luego, en el método **main**, use estos valores para construir un objeto **CognitiveServicesManager**. Este objeto es necesario para todas las operaciones de Administración de Azure.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_auth)]

## <a name="call-management-methods"></a>Métodos de administración de llamada

Agregue el código siguiente al método **Main** para enumerar los recursos disponibles, crear un recurso de ejemplo, enumerar los recursos que posee y, después, eliminar el recurso de ejemplo. Estos métodos los definirá en los siguientes pasos.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_calls)]

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

### <a name="choose-a-service-and-pricing-tier"></a>Elección de un servicio y un plan de tarifa

Al crear un recurso, necesitará conocer el "tipo" de servicio que desea usar y el [plan de tarifa](https://azure.microsoft.com/pricing/details/cognitive-services/) (o la SKU) que desee. Usará esta y otra información como parámetros al crear el recurso. Puede encontrar una lista de los "tipos" de servicios cognitivos disponibles llamando al siguiente método:

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

Para crear un recurso de Cognitive Services y suscribirse a él, use el método **create**. Este método agrega un nuevo recurso facturable al grupo de recursos que se pasa. Cuando cree el recurso, deberá conocer el "tipo" de servicio que desea usar, junto con su plan de tarifa (o SKU) y una ubicación de Azure. El siguiente método usa todos estos valores como argumentos y crea un recurso.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_create)]

## <a name="view-your-resources"></a>Visualización de los recursos

Para ver todos los recursos de su cuenta de Azure (de todos los grupos de recursos), use el siguiente método:

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_list)]

## <a name="delete-a-resource"></a>Eliminación de un recurso

El método siguiente elimina el recurso especificado del grupo de recursos dado.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_delete)]

## <a name="see-also"></a>Consulte también

* [Documentación de referencia del SDK de Administración de Azure](https://docs.microsoft.com/java/api/com.microsoft.azure.management.cognitiveservices?view=azure-java-stable)
* [¿Qué es Azure Cognitive Services?](../../Welcome.md)
* [Autenticación de solicitudes en Azure Cognitive Services](../../authentication.md)
* [Creación de un recurso mediante Azure Portal](../../cognitive-services-apis-create-account.md)