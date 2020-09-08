---
title: Empaquetado e implementación de contenedores
description: En este tutorial, aprenderá a generar una definición de aplicación de Azure Service Fabric con Yeoman y a empaquetar la aplicación.
author: suhuruli
ms.topic: tutorial
ms.date: 07/22/2019
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: cc1d6e04b19d36f0ca8c7ed4b2bb3d62f5e8e15a
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "78252750"
---
# <a name="tutorial-package-and-deploy-containers-as-a-service-fabric-application-using-yeoman"></a>Tutorial: Empaquetamiento e implementación de contenedores como aplicación de Service Fabric mediante Yeoman

Este tutorial es la segunda parte de una serie. En este tutorial, se emplea una herramienta de generador de plantillas (Yeoman) para generar una definición de aplicación de Service Fabric. Por lo tanto, esta aplicación se puede usar para implementar contenedores en Service Fabric. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Instalación de Yeoman
> * Crear un paquete de aplicación con Yeoman.
> * Definir la configuración en el paquete de aplicación para su uso con contenedores
> * Compilar la aplicación
> * Implementar y ejecutar la aplicación
> * Limpiar la aplicación

## <a name="prerequisites"></a>Prerequisites

* Se usan las imágenes de contenedor insertadas en Azure Container Registry creadas en la [Parte 1](service-fabric-tutorial-create-container-images.md) de esta serie del tutorial.
* Se [configura](service-fabric-tutorial-create-container-images.md) el entorno de desarrollo de Linux.

## <a name="install-yeoman"></a>Instalación de Yeoman

Service Fabric proporciona herramientas de scaffolding para ayudarle a crear aplicaciones desde el terminal mediante el generador de plantillas Yeoman. Siga los pasos siguientes para asegurarse de que dispone del generador de plantillas Yeoman.

1. Instale nodejs y NPM en la máquina. Tenga en cuenta que los usuarios de Mac OSX tendrán que usar el administrador de paquetes Homebrew.

    ```bash
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
    nvm install node 
    ```
2. Instale el generador de plantillas Yeoman en la máquina desde NPM.

    ```bash
    npm install -g yo
    ```
3. Instale el generador de contenedor Yeoman de Service Fabric.

    ```bash 
    npm install -g generator-azuresfcontainer
    ```

## <a name="package-a-docker-image-container-with-yeoman"></a>Empaquetamiento de un contenedor de imagen de Docker con Yeoman

1. Para crear una aplicación de contenedor de Service Fabric, ejecute el siguiente comando en el directorio "tutorial de contenedor" del repositorio clonado.

    ```bash
    yo azuresfcontainer
    ```
2. Escriba en "TestContainer" para asignar un nombre a la aplicación
3. Escriba en "azurevotefront" para asignar un nombre al servicio de aplicación.
4. Proporcione la ruta de acceso a la imagen de contenedor en ACR para el repositorio de front-end: por ejemplo, "\<acrName>.azurecr.io/azure-vote-front:v1". El campo \<acrName> debe ser el mismo que el valor que se utilizó en el tutorial anterior.
5. Presione Entrar para dejar la sección Comandos vacía.
6. Especifique un número de instancias de "1".

A continuación se muestra la entrada y salida de la ejecución del comando yo:

```bash
? Name your application TestContainer
? Name of the application service: azurevotefront
? Input the Image Name: <acrName>.azurecr.io/azure-vote-front:v1
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/TestContainer/ApplicationManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/ServiceManifest.xml
   create TestContainer/TestContainer/azurevotefrontPkg/config/Settings.xml
   create TestContainer/TestContainer/azurevotefrontPkg/code/Dummy.txt
   create TestContainer/install.sh
   create TestContainer/uninstall.sh
```

Para agregar otro servicio de contenedor a una aplicación ya creada mediante Yeoman, realice los pasos siguientes:

1. Cambie el directorio un nivel hasta el directorio **TestContainer**; por ejemplo, *./TestContainer*
2. Ejecute `yo azuresfcontainer:AddService`:
3. Asigne el nombre "azurevoteback" al servicio
4. Proporcione la ruta de acceso de imagen de contenedor para Redis: "alpine:redis"
5. Presione Entrar para dejar la sección Comandos vacía
6. Especifique un número de instancias de "1".

Las entradas usadas para la adición del servicio son las que se muestran a continuación:

```bash
? Name of the application service: azurevoteback
? Input the Image Name: alpine:redis
? Commands:
? Number of instances of guest container application: 1
   create TestContainer/azurevotebackPkg/ServiceManifest.xml
   create TestContainer/azurevotebackPkg/config/Settings.xml
   create TestContainer/azurevotebackPkg/code/Dummy.txt
```

Para el resto de este tutorial, estamos trabajando en el directorio **TestContainer**. Por ejemplo, *./TestContainer/TestContainer*. El contenido de este directorio debería ser el siguiente.

```bash
$ ls
ApplicationManifest.xml azurevotefrontPkg azurevotebackPkg
```

## <a name="configure-the-application-manifest-with-credentials-for-azure-container-registry"></a>Configuración del manifiesto de aplicación con credenciales para Azure Container Registry

Para que Service Fabric extraiga las imágenes de contenedor de Azure Container Registry, necesitamos proporcionar las credenciales en **ApplicationManifest.xml**.

Inicie sesión en la instancia de ACR. Use el comando **az acr login** para completar la operación. Proporcione el nombre único que se especificó para el registro de contenedor cuando se creó.

```azurecli
az acr login --name <acrName>
```

Al finalizar, el comando devuelve un mensaje que indica que el **inicio de sesión se ha realizado correctamente**.

A continuación, ejecute el siguiente comando para obtener la contraseña del registro de contenedor. Service Fabric utiliza esta contraseña para realizar la autenticación con ACR y extraer las imágenes del contenedor.

```azurecli
az acr credential show -n <acrName> --query passwords[0].value
```

En **ApplicationManifest.xml**, agregue el fragmento de código en el elemento **ServiceManifestImport** para el servicio de front-end. Inserte **acrName** para el campo **AccountName** y la contraseña devuelta del comando anterior se usará para el campo **Contraseña**. Se proporciona un archivo **ApplicationManifest.xml** completo al final de este documento.

```xml
<Policies>
  <ContainerHostPolicies CodePackageRef="Code">
    <RepositoryCredentials AccountName="<acrName>" Password="<password>" PasswordEncrypted="false"/>
  </ContainerHostPolicies>
</Policies>
```

## <a name="configure-communication-and-container-port-to-host-port-mapping"></a>Configuración de la comunicación y asignación del puerto "puerto a host" del contenedor

### <a name="configure-communication-port"></a>Configuración del puerto de comunicación

Configure un punto de conexión HTTP para que los clientes puedan comunicarse con el servicio. Abra el archivo *./TestContainer/azurevotefrontPkg/ServiceManifest.xml* y declare un recurso de punto de conexión en el elemento **ServiceManifest**.  Agregue el protocolo, el puerto y el nombre. En este tutorial, el servicio realiza escuchas en el puerto 80. El siguiente fragmento de código se coloca en la etiqueta *ServiceManifest* del recurso.

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
  </Endpoints>
</Resources>

```

De forma similar, modifique el manifiesto de servicio para el servicio back-end. Abra *./TestContainer/azurevotebackPkg/ServiceManifest.xml* y declare un recurso de punto de conexión en el elemento **ServiceManifest**. Para este tutorial, se mantiene el valor predeterminado de Redis de 6379. El siguiente fragmento de código se coloca en la etiqueta *ServiceManifest* del recurso.

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
            listen. Please note that if your service is partitioned, this port is shared with 
            replicas of different partitions that are placed in your code. -->
    <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
  </Endpoints>
</Resources>
```

Si se especifica **UriScheme**, se registra automáticamente el punto de conexión del contenedor con el servicio de nombres de Service Fabric para facilitar la detección. Al final de este artículo se proporciona como ejemplo un archivo de ejemplo ServiceManifest.xml completo para el servicio back-end.

### <a name="map-container-ports-to-a-service"></a>Asignación de puertos de contenedor a un servicio

Para exponer los contenedores en el clúster, también es necesario crear un enlace de puerto en el archivo e "ApplicationManifest.xml". La directiva **PortBinding** hace referencia a los **puntos de conexión** que definimos en los archivos **ServiceManifest.xml**. Las solicitudes entrantes a estos puntos de conexión se asignan a los puertos del contenedor que se abren y enlazan aquí. En el archivo **ApplicationManifest.xml**, agregue el siguiente código para enlazar los puertos 80 y 6379 a los puntos de conexión. Hay un archivo **ApplicationManifest.xml** completo disponible al final de este documento.

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
</ContainerHostPolicies>
```

```xml
<ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
</ContainerHostPolicies>
```

### <a name="add-a-dns-name-to-the-backend-service"></a>Adición de un nombre DNS al servicio back-end

Para que Service Fabric asigne este nombre DNS al servicio back-end, el nombre tiene que especificarse en el archivo **ApplicationManifest.xml**. Agregue el atributo **ServiceDnsName** al elemento de **servicio** como se muestra:

```xml
<Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
  <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
    <SingletonPartition/>
  </StatelessService>
</Service>
```

El servicio front-end lee una variable de entorno para conocer el nombre DNS de la instancia de Redis. Esta variable de entorno ya está definida en el Dockerfile que se usó para generar la imagen de Docker y no es necesario realizar ninguna acción.

```dockerfile
ENV REDIS redisbackend.testapp
```

El fragmento de código siguiente muestra cómo el código de Python de front-end recoge la variable de entorno que se describe en el Dockerfile. No es necesario realizar ninguna acción.

```python
# Get DNS Name
redis_server = os.environ['REDIS']

# Connect to the Redis store
r = redis.StrictRedis(host=redis_server, port=6379, db=0)
```

En este punto del tutorial, la plantilla para una aplicación de paquete de servicio está disponible para su implementación en un clúster. En el tutorial posterior, se implementa esta aplicación y se ejecuta en un clúster de Service Fabric.

## <a name="create-a-service-fabric-cluster"></a>Creación de un clúster de Service Fabric

Para implementar la aplicación en Azure, se necesita un clúster de Service Fabric que ejecute la aplicación. Los siguientes comandos crean un clúster de cinco nodos en Azure.  Los comandos también crean un certificado autofirmado, lo agregan a un almacén de claves y los descargan en el sitio local como un archivo PEM. El nuevo certificado se usa para proteger el clúster al implementarlo y también para autenticar a los clientes.

```azurecli
#!/bin/bash

# Variables
ResourceGroupName="containertestcluster" 
ClusterName="containertestcluster" 
Location="eastus" 
Password="q6D7nN%6ck@6" 
Subject="containertestcluster.eastus.cloudapp.azure.com" 
VaultName="containertestvault" 
VmPassword="Mypa$$word!321"
VmUserName="sfadminuser"

# Login to Azure and set the subscription
az login

az account set --subscription <mySubscriptionID>

# Create resource group
az group create --name $ResourceGroupName --location $Location 

# Create secure five node Linux cluster. Creates a key vault in a resource group
# and creates a certficate in the key vault. The certificate's subject name must match 
# the domain that you use to access the Service Fabric cluster.  
# The certificate is downloaded locally as a PEM file.
az sf cluster create --resource-group $ResourceGroupName --location $Location \ 
--certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject \ 
--cluster-name $ClusterName --cluster-size 5 --os UbuntuServer1604 --vault-name $VaultName \ 
--vault-resource-group $ResourceGroupName --vm-password $VmPassword --vm-user-name $VmUserName
```

> [!Note]
> El servicio front-end web está configurado para escuchar en el puerto 80 el tráfico entrante. De forma predeterminada, el puerto 80 está abierto en las máquinas virtuales del el clúster y en el equilibrador de carga de Azure.
>

Para información sobre cómo crear su propio clúster, consulte [Creación de un clúster de Service Fabric en Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md).

## <a name="build-and-deploy-the-application-to-the-cluster"></a>Creación e implementación de la aplicación en el clúster

Puede implementar la aplicación en el clúster de Azure mediante la CLI de Service Fabric. Si la CLI de Service Fabric CLI no está instalada en su máquina, siga las instrucciones [aquí](service-fabric-get-started-linux.md#set-up-the-service-fabric-cli) para instalarla.

Conéctese al clúster de Service Fabric en Azure. Reemplace el punto de conexión del ejemplo por uno propio. El punto de conexión debe ser una dirección URL completa similar a la siguiente.  El archivo PEM es el certificado autofirmado que se creó anteriormente.

```bash
sfctl cluster select --endpoint https://containertestcluster.eastus.cloudapp.azure.com:19080 --pem containertestcluster22019013100.pem --no-verify
```

Use el script de instalación proporcionado en el directorio **TestContainer** para copiar el paquete de aplicación en el almacén de imágenes del clúster, registrar el tipo de aplicación y crear una instancia de la aplicación.

```bash
./install.sh
```

Abra un explorador y vaya a Service Fabric Explorer en http:\//containertestcluster.eastus.cloudapp.azure.com:19080/Explorer. Expanda el nodo Applications y observe que hay una entrada para su tipo de aplicación y otra para la instancia.

![Service Fabric Explorer][sfx]

Para conectarse a la aplicación en ejecución, abra un explorador web y vaya a la dirección URL del clúster, por ejemplo, http:\//containertestcluster.eastus.cloudapp.azure.com:80. Debería ver la aplicación de votación en la interfaz de usuario web.

![votingapp][votingapp]

## <a name="clean-up"></a>Limpieza

Para eliminar la instancia de aplicación del clúster y anular el registro del tipo de aplicación, utilice el script de desinstalación proporcionado en la plantilla. Este comando tarda algún tiempo en limpiar la instancia y no se puede ejecutar el comando "install'sh" inmediatamente después de este script.

```bash
./uninstall.sh
```

## <a name="examples-of-completed-manifests"></a>Ejemplos de manifiestos completados

### <a name="applicationmanifestxml"></a>ApplicationManifest.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ApplicationManifest ApplicationTypeName="TestContainerType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotefrontPkg" ServiceManifestVersion="1.0.0"/>
    <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myaccountname" Password="<password>" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="azurevotefrontTypeEndpoint"/>
    </ContainerHostPolicies>
        </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="azurevotebackPkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
          <PortBinding ContainerPort="6379" EndpointRef="azurevotebackTypeEndpoint"/>
        </ContainerHostPolicies>
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="azurevotefront">
      <StatelessService ServiceTypeName="azurevotefrontType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
    <Service Name="azurevoteback" ServiceDnsName="redisbackend.testapp">
      <StatelessService ServiceTypeName="azurevotebackType" InstanceCount="1">
        <SingletonPartition/>
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

### <a name="front-end-servicemanifestxml"></a>Front-end ServiceManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotefrontPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotefrontType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>acrName.azurecr.io/azure-vote-front:v1</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotefrontTypeEndpoint" UriScheme="http" Port="80" Protocol="http"/>
    </Endpoints>
  </Resources>

 </ServiceManifest>
```

### <a name="redis-servicemanifestxml"></a>Redis ServiceManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="azurevotebackPkg" Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" >

   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="azurevotebackType" UseImplicitHost="true">
   </StatelessServiceType>
   </ServiceTypes>

   <CodePackage Name="code" Version="1.0.0">
      <EntryPoint>
         <ContainerHost>
            <ImageName>alpine:redis</ImageName>
            <Commands></Commands>
         </ContainerHost>
      </EntryPoint>
      <EnvironmentVariables>
      </EnvironmentVariables>
   </CodePackage>
     <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="azurevotebackTypeEndpoint" Port="6379" Protocol="tcp"/>
    </Endpoints>
  </Resources>
 </ServiceManifest>
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se empaquetaron varios contenedores en una aplicación de Service Fabric mediante Yeoman. Esta aplicación se implementó y ejecutó en un clúster de Service Fabric. Se han completado los siguientes pasos:

> [!div class="checklist"]
> * Instalación de Yeoman
> * Crear un paquete de aplicación con Yeoman.
> * Definir la configuración en el paquete de aplicación para su uso con contenedores
> * Compilar la aplicación
> * Implementar y ejecutar la aplicación
> * Limpiar la aplicación

Vaya al siguiente tutorial para obtener información acerca de la conmutación por error y el escalado de la aplicación en Service Fabric.

> [!div class="nextstepaction"]
> [Más información sobre la conmutación por error y el escalado de aplicaciones](service-fabric-tutorial-containers-failover.md)

[votingapp]: ./media/service-fabric-tutorial-deploy-run-containers/votingapp.png
[sfx]: ./media/service-fabric-tutorial-deploy-run-containers/containerspackagetutorialsfx.png