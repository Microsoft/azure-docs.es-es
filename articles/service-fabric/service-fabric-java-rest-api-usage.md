---
title: API de cliente Java de Azure Service Fabric
description: Generación y uso de las API de cliente Java de Service Fabric con la especificación de la API de REST del cliente de Service Fabric
author: rapatchi
ms.topic: conceptual
ms.date: 11/27/2017
ms.custom: devx-track-java
ms.author: rapatchi
ms.openlocfilehash: 24ee4a3d8109626bf93e01fbd10c00140762c9c6
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324631"
---
# <a name="azure-service-fabric-java-client-apis"></a>API de cliente Java de Azure Service Fabric

Las API de cliente de Service Fabric permiten la implementación y administración de contenedores y aplicaciones basadas en microservicios en un clúster de Service Fabric en Azure, ya sea localmente o en la máquina de desarrollo local o en otra nube. Este artículo describe cómo generar y utilizar las API de cliente Java de Service Fabric sobre las API de REST de cliente de Service Fabric

## <a name="generate-the-client-code-using-autorest"></a>Generación del código de cliente mediante AutoRest

[AutoRest](https://github.com/Azure/autorest) es una herramienta que genera bibliotecas de cliente para tener acceso a servicios web RESTful. La entrada a AutoRest es una especificación que describe la API de REST con el formato de especificación OpenAPI. [Las API de REST de cliente de Service Fabric](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/servicefabric/data-plane) siguen esta especificación.

Siga los pasos que se mencionan a continuación para generar el código del cliente Java de Service Fabric mediante la herramienta AutoRest.

1. Instalación de nodejs y NPM en la máquina

    Si usa Linux:
    ```bash
    sudo apt-get install npm
    sudo apt install nodejs
    ```
    Si usa Mac OS X:
    ```bash
    brew install node
    ```

2. Instale AutoRest con NPM.
    ```bash
    npm install -g autorest
    ```

3. Bifurque y clone el repositorio de [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs) en su máquina local y vaya a la ubicación clonada desde el terminal de su máquina.


4. Vaya a la ubicación que se menciona más abajo en el repositorio clonado.
    ```bash
    cd specification\servicefabric\data-plane\Microsoft.ServiceFabric\stable\6.0
    ```

    > [!NOTE]
    > Si su versión de clúster no es la 6.0.*, haga clic en el directorio adecuado en la carpeta estable.
    >   

5. Ejecute el siguiente comando de AutoRest para generar el código de cliente Java.
    
    ```bash
    autorest --input-file= servicefabric.json --java --output-folder=[output-folder-name] --namespace=[namespace-of-generated-client]
    ```
   A continuación se muestra un ejemplo que ilustra el uso de AutoRest.
   
    ```bash
    autorest --input-file=servicefabric.json --java --output-folder=java-rest-api-code --namespace=servicefabricrest
    ```
   
   El siguiente comando toma el archivo de especificación ``servicefabric.json`` como entrada, genera el código de cliente Java en la carpeta ``java-rest-api-     code`` y agrega el código en el espacio de nombres ``servicefabricrest``. Después de este paso, encontraría dos carpetas ``models`` y ``implementation``, y dos archivos ``ServiceFabricClientAPIs.java`` y ``package-info.java`` generados en la carpeta ``java-rest-api-code``.


## <a name="include-and-use-the-generated-client-in-your-project"></a>Inclusión y uso del cliente generado en el proyecto

1. Agregue el código generado correctamente en el proyecto. Se recomienda crear una biblioteca con el código generado e incluirla en el proyecto.
2. Si va a crear una biblioteca, incluya la siguiente dependencia en su proyecto. Si está siguiendo un enfoque diferente, incluya la dependencia adecuadamente.

    ```
        GroupId:  com.microsoft.rest
        Artifactid: client-runtime
        Version: 1.2.1
    ```
    Por ejemplo, si usa un sistema de compilación de Maven, incluya lo siguiente en el archivo ``pom.xml``:

    ```xml
        <dependency>
          <groupId>com.microsoft.rest</groupId>
          <artifactId>client-runtime</artifactId>
          <version>1.2.1</version>
        </dependency>
    ```

3. Cree RestClient con el código siguiente:

    ```java
        RestClient simpleClient = new RestClient.Builder()
            .withBaseUrl("http://<cluster-ip or name:port>")
            .withResponseBuilderFactory(new ServiceResponseBuilder.Factory())
            .withSerializerAdapter(new JacksonAdapter())
            .build();
        ServiceFabricClientAPIs client = new ServiceFabricClientAPIsImpl(simpleClient);
    ```
4. Use el objeto de cliente y realice las llamadas adecuadas según sea necesario. Estos son algunos ejemplos que muestran el uso del objeto de cliente. Se supone que el paquete de aplicación se compila y se carga en el almacén de imágenes antes de usar las API siguientes.
    * Aprovisionamiento de una aplicación
    
        ```java
            ApplicationTypeImageStorePath imageStorePath = new ApplicationTypeImageStorePath();
            imageStorePath.withApplicationTypeBuildPath("<application-path-in-image-store>");
            client.provisionApplicationType(imageStorePath);
        ```
    * Crear una aplicación

        ```java
            ApplicationDescription applicationDescription = new ApplicationDescription();
            applicationDescription.withName("<application-uri>");
            applicationDescription.withTypeName("<application-type>");
            applicationDescription.withTypeVersion("<application-version>");
            client.createApplication(applicationDescription);
        ```

## <a name="understanding-the-generated-code"></a>Descripción del código generado
Para todas las API, encontrará cuatro sobrecargas de implementación. Si no hay parámetros opcionales, encontraría cuatro variantes más, incluidos esos. Por ejemplo, considere la API ``removeReplica``.
 1. **public void removeReplica(String nodeName, UUID partitionId, String replicaId, Boolean forceRemove, Long timeout)**
    * Se trata de la variante sincrónica de la llamada a la API removeReplica.
 2. **public ServiceFuture\<Void> removeReplicaAsync(String nodeName, UUID partitionId, String replicaId, Boolean forceRemove, Long timeout, final ServiceCallback\<Void> serviceCallback)**
    * Esta variante de llamada de la API puede usarse si desea utilizar futuras devoluciones de llamada y llamadas de programación asincrónica.
 3. **public Observable\<Void> removeReplicaAsync(String nodeName, UUID partitionId, String replicaId)**
    * Esta variante de llamada a la API puede usarse si desea utilizar la programación asincrónica reactiva.
 4. **public Observable\<ServiceResponse\<Void>> removeReplicaWithServiceResponseAsync(String nodeName, UUID partitionId, String replicaId)**
    * Esta variante de llamada a la API puede usarse si se desea utilizar una programación asincrónica reactiva y trabajar con la respuesta de la API de REST con formato RAW.

## <a name="next-steps"></a>Pasos siguientes
* Más información sobre las [API de REST de Service Fabric](/rest/api/servicefabric/).
