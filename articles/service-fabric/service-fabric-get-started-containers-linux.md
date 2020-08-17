---
title: Creación de una aplicación contenedora de Azure Service Fabric en Linux
description: Cree la primera aplicación contenedora en Linux en Azure Service Fabric. Cree una imagen de Docker con la aplicación, inserte la imagen en un registro de contenedor y compile e implemente una aplicación contenedora en Service Fabric.
ms.topic: conceptual
ms.date: 1/4/2019
ms.custom: devx-track-python
ms.openlocfilehash: 35e96f1039dc71427a1a3d2745245eff5d012aaf
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87847540"
---
# <a name="create-your-first-service-fabric-container-application-on-linux"></a>Cree la primera aplicación contenedora de Service Fabric en Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

La ejecución de una aplicación existente en un contenedor de Linux en un clúster de Service Fabric no requiere que se realice ningún cambio en la aplicación. Este artículo le guiará por la creación de una imagen de Docker que contiene una aplicación web [Flask](http://flask.pocoo.org/) en Python y su implementación en un clúster de Service Fabric. También compartirá la aplicación en el contenedor mediante [Azure Container Registry](../container-registry/index.yml). Este artículo supone que el usuario tiene un conocimiento básico de Docker. Para obtener información acerca de Docker, lea la [introducción a Docker](https://docs.docker.com/engine/understanding-docker/).

> [!NOTE]
> Este artículo es aplicable a un entorno de desarrollo de Linux.  El runtime del clúster de Service Fabric y el runtime de Docker deben ejecutarse en el mismo sistema operativo.  No se pueden ejecutar contenedores de Linux en un clúster de Windows.

## <a name="prerequisites"></a>Prerrequisitos
* Un equipo de desarrollo en el que se ejecute:
  * [SDK y herramientas de Service Fabric](service-fabric-get-started-linux.md).
  * [Docker CE para Linux](https://docs.docker.com/engine/installation/#prior-releases). 
  * [CLI de Service Fabric](service-fabric-cli.md)

* Un clúster de Linux con tres o más nodos.

* Un registro de Azure Container Registry: [cree un registro de contenedor](../container-registry/container-registry-get-started-portal.md) en la suscripción de Azure. 

## <a name="define-the-docker-container"></a>Definición del contenedor de Docker
Compile una imagen basada en la [imagen de Python](https://hub.docker.com/_/python/) ubicada en Docker Hub. 

Especifique el contenedor de Docker en un archivo Dockerfile. El archivo Dockerfile consiste en instrucciones para la configuración del entorno dentro del contenedor, la carga de la aplicación que desea ejecutar y la asignación de puertos. El Dockerfile es la entrada para el comando `docker build`, que crea la imagen. 

Cree un directorio vacío y cree el archivo *Dockerfile* (sin extensión de archivo). Agregue lo siguiente al archivo *Dockerfile* y guarde los cambios:

```
# Use an official Python runtime as a base image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Para más información, lea la [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="create-a-basic-web-application"></a>Creación de una aplicación web básica
Cree una aplicación web de Flask que escucha en el puerto 80 y devuelve "Hola mundo". En el mismo directorio, cree el archivo *requirements.txt*. Agregue lo siguiente y guarde los cambios:
```
Flask
```

Cree también el archivo *app.py* y agregue el siguiente fragmento de código:

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def hello():

    return 'Hello World!'


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

## <a name="build-the-image"></a>Compilación de la imagen
Ejecute el comando `docker build` para crear la imagen que ejecuta la aplicación web. Abra una ventana de PowerShell y navegue hasta *c:\temp\helloworldapp*. Ejecute el siguiente comando:

```bash
docker build -t helloworldapp .
```

Este comando crea la imagen nueva con las instrucciones del Dockerfile y asigna el nombre (etiqueta -t) `helloworldapp` a la imagen. Para crear una imagen de contenedor, primero se descarga la imagen base del concentrador de Docker al que se agrega la aplicación. 

Una vez que se complete el comando de compilación, ejecute el comando `docker images` para ver información sobre la nueva imagen:

```bash
$ docker images
    
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              86838648aab6        2 minutes ago       194 MB
```

## <a name="run-the-application-locally"></a>Ejecución de la aplicación de forma local
Compruebe que la aplicación en contenedor se ejecuta localmente antes de insertarla en el registro de contenedor. 

Ejecute la aplicación, asignando el puerto 4000 del equipo al puerto 80 expuesto del contenedor:

```bash
docker run -d -p 4000:80 --name my-web-site helloworldapp
```

*name* asigna un nombre al contenedor en ejecución (en lugar del identificador del contenedor).

Conéctese al contenedor en ejecución. Abra un explorador web que apunte a la dirección IP que se devuelve en el puerto 4000; por ejemplo, "http:\//localhost:4000". Debería ver que el título "¡Hola mundo!" se muestra en el explorador.

!["¡Hola mundo!"][hello-world]

Para detener el contenedor, ejecute:

```bash
docker stop my-web-site
```

Elimine el contenedor del equipo de desarrollo:

```bash
docker rm my-web-site
```

## <a name="push-the-image-to-the-container-registry"></a>Inserción de la imagen en el registro de contenedor
Después de comprobar que la aplicación se ejecuta en Docker, inserte la imagen en el registro de Azure Container Registry.

Ejecute `docker login` para iniciar sesión en el registro de contenedor con sus [credenciales de registro](../container-registry/container-registry-authentication.md).

En el ejemplo siguiente se pasa el identificador y la contraseña de una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md) de Azure Active Directory. Por ejemplo, puede que haya asignado una entidad de servicio al registro para ver un escenario de automatización. O bien, puede iniciar sesión con su nombre de usuario y contraseña del registro.

```bash
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

El siguiente comando crea una etiqueta, o alias, de la imagen, con una ruta de acceso completa al registro. Este ejemplo coloca la imagen en el espacio de nombres `samples` para evitar el desorden en la raíz del registro.

```bash
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

Inserte la imagen en el registro de contenedor:

```bash
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="package-the-docker-image-with-yeoman"></a>Empaquetamiento de la imagen de Docker con Yeoman
El SDK de Service Fabric para Linux incluye un generador [Yeoman](https://yeoman.io/) que permite crear fácilmente la aplicación y agregar una imagen de contenedor. Vamos a usar Yeoman para crear una aplicación con un único contenedor de Docker denominado "*SimpleContainerApp*".

Para crear una aplicación contenedora de Service Fabric, abra una ventana de terminal y ejecute `yo azuresfcontainer`. 

Asigne un nombre a la aplicación (por ejemplo, "`mycontainer`") y otro nombre al servicio de aplicación (por ejemplo, "`myservice`").

Para el nombre de la imagen, especifique la dirección URL de la imagen de contenedor en un registro de contenedor (por ejemplo, "myregistry.azurecr.io/samples/helloworldapp"). 

Como dicha imagen tiene un punto de entrada de la carga de trabajo definido, no es preciso especificar explícitamente comandos de entrada (los comandos se ejecutarán dentro del contenedor, por lo que este seguirá en ejecución después del inicio). 

Especifique un número de instancias de "1".

Especifique la asignación de puertos con el formato adecuado. En este artículo, deberá proporcionar ```80:4000``` como asignación de puertos. Esto implica que todas las solicitudes entrantes que lleguen al puerto 4000 en el equipo host se redirijan al puerto 80 del contenedor.

![Generador Yeoman de Service Fabric para contenedores][sf-yeoman]

## <a name="configure-container-repository-authentication"></a>Configuración la autenticación de repositorio del contenedor

Consulte [autenticación del repositorio de contenedores](configure-container-repository-credentials.md) para obtener información sobre cómo configurar diferentes tipos de autenticación para la descarga de imágenes de contenedor.

## <a name="configure-isolation-mode"></a>Configuración del modo de aislamiento
Con la versión 6.3 del entorno de ejecución, se admite el aislamiento de la máquina virtual para los contenedores de Linux, con lo que se admiten dos modos de aislamiento para contenedores: proceso e Hyper-V. Con el modo de aislamiento de Hyper-V, los kernels se aíslan entre cada contenedor y el host del contenedor. El aislamiento de Hyper-V se implementa mediante la opción [Borrar contenedores](https://software.intel.com/en-us/articles/intel-clear-containers-2-using-clear-containers-with-docker). El modo de aislamiento se especifica para los clústeres de Linux en el elemento `ServicePackageContainerPolicy` del archivo de manifiesto de aplicación. Los modos de aislamiento que se pueden especificar son `process`, `hyperv` y `default`. El valor predeterminado es el modo de aislamiento de proceso. El siguiente fragmento de código muestra cómo el modo de aislamiento se especifica en el archivo de manifiesto de aplicación.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyServicePkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ServicePackageContainerPolicy Hostname="votefront" Isolation="hyperv">
          <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
        </ServicePackageContainerPolicy>
    </Policies>
  </ServiceManifestImport>
```


## <a name="configure-resource-governance"></a>Configuración de la gobernanza de recursos
La [gobernanza de recursos](service-fabric-resource-governance.md) restringe los recursos que el contenedor puede usar en el host. El elemento `ResourceGovernancePolicy`, especificado en el manifiesto de la aplicación, se utiliza para declarar los límites de recursos para un paquete de código de servicio. Es posible establecer límites para los siguientes recursos: memoria, MemorySwap, CpuShares (peso relativo de CPU), MemoryReservationInMB, BlkioWeight (peso relativo de BlockIO). En este ejemplo, el paquete de servicio Guest1Pkg obtiene un núcleo en los nodos del clúster en los que es situado. Los límites de memoria son absolutos, por lo que el paquete de código está limitado a 1024 MB de memoria (con una reserva de garantía flexible de dicha capacidad). Los paquetes de código (contenedores o procesos) no pueden asignar más memoria de la que establece este límite; si se intenta, el resultado es una excepción de memoria insuficiente. Para que la aplicación del límite de recursos funcione, es necesario haber definido límites de memoria en todos los paquetes de código de un paquete de servicio.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="MyServicePKg" ServiceManifestVersion="1.0.0" />
  <Policies>
    <ServicePackageResourceGovernancePolicy CpuCores="1"/>
    <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
  </Policies>
</ServiceManifestImport>
```




## <a name="configure-docker-healthcheck"></a>Configuración de la instrucción HEALTHCHECK de Docker 

A partir de la versión 6.1, Service Fabric integra automáticamente eventos de la [instrucción HEALTHCHECK de Docker](https://docs.docker.com/engine/reference/builder/#healthcheck) en su informe de mantenimiento del sistema. Esto significa que si el contenedor tiene habilitada la instrucción **HEALTHCHECK**, Service Fabric informará acerca del mantenimiento siempre que el estado de mantenimiento del contenedor cambie tal y como lo indique Docker. Aparecerá un informe de mantenimiento **correcto** en [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) siempre que *health_status* sea *correcto* y aparecerá **ADVERTENCIA** si *health_status* es *incorrecto*. 

A partir de la última versión de actualización, v6.4, tiene la opción de especificar que las evaluaciones de HEALTHCHECK de Docker deben notificarse como un error. Si se habilita esta opción, aparecerá un informe de estado **OK** cuando *health_status* sea *healthy* (correcto) y **ERROR** aparecerá cuando *health_status* sea *unhealthy* (incorrecto).

La instrucción **HEALTHCHECK** que apunta a la comprobación real que se lleva a cabo para supervisar el mantenimiento del contenedor debe estar presente en el archivo de Docker que se usa al generar la imagen de contenedor.

![HealthCheckHealthy][1]

![HealthCheckUnhealthyApp][2]

![HealthCheckUnhealthyDsp][3]

Puede configurar el comportamiento **HEALTHCHECK** en cada contenedor mediante la especificación de las opciones **HealthConfig** como parte de **ContainerHostPolicies** en ApplicationManifest.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="ContainerServicePkg" ServiceManifestVersion="2.0.0" />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <HealthConfig IncludeDockerHealthStatusInSystemHealthReport="true"
              RestartContainerOnUnhealthyDockerHealthStatus="false" 
              TreatContainerUnhealthyStatusAsError="false" />
      </ContainerHostPolicies>
    </Policies>
</ServiceManifestImport>
```
De forma predeterminada, se establece *IncludeDockerHealthStatusInSystemHealthReport* en **true**, *RestartContainerOnUnhealthyDockerHealthStatus* en **false** y *TreatContainerUnhealthyStatusAsError* en **false**. 

Si se establece *RestartContainerOnUnhealthyDockerHealthStatus* en **true**, se reiniciará un contenedor que constantemente informa de un error de mantenimiento (posiblemente en otros nodos).

Si *TreatContainerUnhealthyStatusAsError* se establece en **true**, el informe de mantenimiento con **ERROR** aparecerá cuando el elemento *health_status* del contenedor sea *unhealthy*.

Si quiere deshabilitar la integración de la instrucción **HEALTHCHECK** para todo el clúster de Service Fabric, deberá establecer [EnableDockerHealthCheckIntegration](service-fabric-cluster-fabric-settings.md) en **false**.

## <a name="deploy-the-application"></a>Implementación de la aplicación
Una vez compilada la aplicación, puede implementarla en el clúster local mediante la CLI de Service Fabric.

Conéctese al clúster de Service Fabric local.

```bash
sfctl cluster select --endpoint http://localhost:19080
```

Use el script de instalación proporcionado en la plantilla en https://github.com/Azure-Samples/service-fabric-containers/ para copiar el paquete de aplicación en el almacén de imágenes del clúster, registrar el tipo de aplicación y crear una instancia de la aplicación.


```bash
./install.sh
```

Abra un explorador y vaya a Service Fabric Explorer en http:\//localhost:19080/Explorer (reemplace localhost por la dirección IP privada de la máquina virtual si usa Vagrant en Mac OS X). Expanda el nodo Applications y observe que ahora hay una entrada para su tipo de aplicación y otra para la primera instancia de ese tipo.

Conéctese al contenedor en ejecución. Abra un explorador web que apunte a la dirección IP que se devuelve en el puerto 4000; por ejemplo, "http:\//localhost:4000". Debería ver que el título "¡Hola mundo!" se muestra en el explorador.

!["¡Hola mundo!"][hello-world]


## <a name="clean-up"></a>Limpieza
Para eliminar la instancia de aplicación del clúster de desarrollo local y anular el registro del tipo de aplicación, utilice el script de desinstalación proporcionado en la plantilla.

```bash
./uninstall.sh
```

Después de insertar la imagen en el registro de contenedor puede eliminar la imagen local del equipo de desarrollo:

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Manifiestos de servicio y de aplicación de Service Fabric de ejemplo completos
Estos son los manifiestos de servicio y de aplicación completos que se usan en este artículo.

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="myservicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="myserviceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying containers 
      to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->
    
    <EnvironmentVariables>
      <!--
      <EnvironmentVariable Name="VariableName" Value="VariableValue"/>
      -->
    </EnvironmentVariables>
    
  </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="myServiceTypeEndpoint" UriScheme="http" Port="4000" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="mycontainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="myservicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="myservice">
      <!-- On a local development cluster, set InstanceCount to 1. On a multi-node production 
      cluster, set InstanceCount to -1 for the container service to run on every node in 
      the cluster.
      -->
      <StatelessService ServiceTypeName="myserviceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
## <a name="adding-more-services-to-an-existing-application"></a>Incorporación de más servicios a una aplicación existente

Para agregar otro servicio de contenedor a una aplicación ya creada mediante yeoman, realice los pasos siguientes:

1. Cambie el directorio al directorio raíz de la aplicación existente. Por ejemplo, `cd ~/YeomanSamples/MyApplication`, si `MyApplication` es la aplicación creada por Yeoman.
2. Ejecute `yo azuresfcontainer:AddService`:

<a id="manually"></a>


## <a name="configure-time-interval-before-container-is-force-terminated"></a>Configuración del intervalo de tiempo antes de hacer que se termine el contenedor

Puede configurar un intervalo de tiempo para que el entorno de ejecución espere antes de que el contenedor se quite una vez que la eliminación del servicio (o el movimiento a otro nodo) se haya iniciado. Si se configura el intervalo de tiempo, se envía el comando `docker stop <time in seconds>` al contenedor.  Para más detalles, consulte [docker stop](https://docs.docker.com/engine/reference/commandline/stop/). El intervalo de tiempo de espera se especifica en la sección `Hosting`. El siguiente fragmento de manifiesto de clúster muestra cómo establecer el intervalo de espera:


```json
{
        "name": "Hosting",
        "parameters": [
          {
                "name": "ContainerDeactivationTimeout",
                "value" : "10"
          },
          ...
        ]
}
```

El intervalo de tiempo predeterminado se establece en diez segundos. Puesto que esta configuración es dinámica, una actualización de solo configuración en el clúster actualiza el tiempo de espera. 

## <a name="configure-the-runtime-to-remove-unused-container-images"></a>Configuración del entorno de ejecución para quitar imágenes de contenedor sin usar

Puede configurar el clúster de Service Fabric para quitar del nodo las imágenes de contenedor sin usar. Esta configuración permite recuperar el espacio en disco si hay demasiadas imágenes de contenedor en el nodo. Para habilitar esta característica, actualice la sección `Hosting` en el manifiesto de clúster, tal como se muestra en el fragmento siguiente: 


```json
{
        "name": "Hosting",
        "parameters": [
          {
                "name": "PruneContainerImages",
                "value": "True"
          },
          {
                "name": "ContainerImagesToSkip",
                "value": "microsoft/windowsservercore|microsoft/nanoserver|microsoft/dotnet-frameworku|..."
          }
          ...
          }
        ]
} 
```

Puede especificar las imágenes que no se deben eliminar en el parámetro `ContainerImagesToSkip`. 

## <a name="configure-container-image-download-time"></a>Configuración del tiempo de descarga de la imagen de contenedor

El entorno de tiempo de ejecución de Service Fabric asigna 20 minutos para descargar y extraer las imágenes de contenedor, lo cual suele bastar en la mayoría de los casos. En el caso de imágenes grandes, o si la conexión de red es lenta, puede que sea necesario aumentar el tiempo de espera antes de anular la descarga y extracción de la imagen. El tiempo de espera se establece mediante el atributo **ContainerImageDownloadTimeout** de la sección de **hospedaje** del manifiesto de clúster, tal y como se muestra en el siguiente fragmento de código:

```json
{
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
}
```


## <a name="set-container-retention-policy"></a>Establecimiento de la directiva de retención de contenedor

Para ayudar a diagnosticar los errores de inicio del contenedor, Service Fabric (versión 6.1 o posterior) admite la retención de contenedores que finalizaron o que produjeron un error en el inicio. Esta directiva se puede establecer en el archivo **ApplicationManifest.xml** tal y como se muestra en el siguiente fragmento de código:

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

El valor **ContainersRetentionCount** especifica el número de contenedores que se conservarán cuando se produzca un error en ellos. Si se especifica un valor negativo, se conservarán todos los contenedores con errores. Si no se especifica el atributo **ContainersRetentionCount**, no se conservará ningún contenedor. El atributo **ContainersRetentionCount** también admite parámetros de aplicación, por lo que los usuarios pueden especificar valores diferentes para los clústeres de prueba y de producción. Utilice restricciones de colocación para seleccionar un destino para el servicio de contenedor en un nodo concreto al usar estas características, para impedir que este se mueva a otros nodos. Los contenedores que se conserven mediante esta característica deben quitarse manualmente.

## <a name="start-the-docker-daemon-with-custom-arguments"></a>Inicio del demonio de Docker con argumentos personalizados

Con la versión 6.2 del entorno del tiempo de ejecución de Service Fabric y superior, puede iniciar el demonio de Docker con argumentos personalizados. Cuando se especifican argumentos personalizados, Service Fabric no pasa ningún otro argumento al motor de Docker excepto el argumento `--pidfile`. Por lo tanto, `--pidfile` no debe pasarse como argumento. Además, el argumento debería continuar haciendo que el demonio de Docker escuche en la canalización del nombre predeterminado en Windows (o en el socket de dominio Unix en Linux) para que Service Fabric se comunique con el demonio. Los argumentos personalizados se especifican en el manifiesto de clúster bajo la sección **Hospedaje** en **ContainerServiceArguments** . Un ejemplo se muestra en el fragmento de código siguiente: 
 

```json
{ 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
} 

```

## <a name="next-steps"></a>Pasos siguientes
* Más información acerca de cómo ejecutar [contenedores en Service Fabric](service-fabric-containers-overview.md).
* Consulte el tutorial [Implementación de una aplicación .NET en un contenedor](service-fabric-host-app-in-a-container.md).
* Más información acerca del [ciclo de vida de aplicaciones](service-fabric-application-lifecycle.md) de Service Fabric.
* Consulte [los ejemplos de código de contenedor de Service Fabric](https://github.com/Azure-Samples/service-fabric-containers) en GitHub.

[hello-world]: ./media/service-fabric-get-started-containers-linux/HelloWorld.png
[sf-yeoman]: ./media/service-fabric-get-started-containers-linux/YoSF.png

[1]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[2]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png
