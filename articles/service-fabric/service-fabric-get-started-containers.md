---
title: Creación de una aplicación contenedora de Azure Service Fabric
description: Cree la primera aplicación contenedora en Windows en Azure Service Fabric. Compile una imagen de Docker con una aplicación de Python, inserte la imagen en un registro de contenedor y, luego, compile e implemente el contenedor en Azure Service Fabric.
ms.topic: conceptual
ms.date: 01/25/2019
ms.custom: devx-track-python
ms.openlocfilehash: 6303e37eaa8fa7ad45677d551b89337d20b1b604
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87844446"
---
# <a name="create-your-first-service-fabric-container-application-on-windows"></a>Cree la primera aplicación contenedora en Service Fabric en Windows

> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

Para ejecutar una aplicación que existe en un contenedor de Windows en un clúster de Service Fabric no hay que hacer ningún cambio en la aplicación. Este artículo le guía en la creación de una imagen de Docker que contiene una aplicación web [Flask](http://flask.pocoo.org/) de Python y su implementación en un clúster de Azure Service Fabric. También compartirá la aplicación en el contenedor mediante [Azure Container Registry](../container-registry/index.yml). Este artículo supone que el usuario tiene un conocimiento básico de Docker. Para obtener información acerca de Docker, lea la [introducción a Docker](https://docs.docker.com/engine/understanding-docker/).

> [!NOTE]
> Este artículo es aplicable a un entorno de desarrollo de Windows.  El runtime del clúster de Service Fabric y el runtime de Docker deben ejecutarse en el mismo sistema operativo.  No se pueden ejecutar contenedores de Windows en un clúster de Linux.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Un equipo de desarrollo en el que se ejecute:
  * Visual Studio 2015 o Visual Studio 2019.
  * [SDK y herramientas de Service Fabric](service-fabric-get-started.md).
  *  Docker para Windows. [Obtener Docker CE para Windows (estable)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description). Después de instalar e iniciar Docker, haga clic con el botón derecho en el icono de la bandeja y seleccione **Switch to Windows containers** (Conmutar a contenedores de Windows), Este paso es necesario para ejecutar imágenes de Docker basadas en Windows.

* Un clúster de Windows con tres o más nodos que se ejecute en Windows Server con Containers. 

  En este artículo, la versión (compilación) de Windows Server con Containers que se ejecuta en los nodos del clúster debe coincidir con la de la máquina de desarrollo. El motivo es que la imagen de Docker se crea en la máquina de desarrollo y existen restricciones de compatibilidad entre las versiones del sistema operativo del contenedor y el sistema operativo del host donde se implementa. Para más información, consulte [Compatibilidad entre el sistema operativo del contenedor y del host en Windows Server](#windows-server-container-os-and-host-os-compatibility). 
  
Para determinar la versión de Windows Server con Containers que necesita para el clúster, ejecute el comando `ver` desde un símbolo del sistema de Windows en la máquina de desarrollo:

* Si la versión contiene *x.x.14323.x*, seleccione *WindowsServer 2016-Datacenter-with-Containers* como sistema operativo al [crear un clúster](service-fabric-cluster-creation-via-portal.md).
  * Si la versión contiene *x.x.16299.x*, seleccione *WindowsServerSemiAnnual Datacenter-Core-1709-with-Containers* como sistema operativo al [crear un cluster](service-fabric-cluster-creation-via-portal.md).

* Un registro de Azure Container Registry: [cree un registro de contenedor](../container-registry/container-registry-get-started-portal.md) en la suscripción de Azure.

> [!NOTE]
> Se admite la implementación de contenedores en un clúster de Service Fabric que se ejecuta en Windows 10.  En [este artículo](service-fabric-how-to-debug-windows-containers.md) encontrará información acerca de cómo configurar Windows 10 para ejecutar contenedores de Windows.
>   

> [!NOTE]
> Las versiones 6.2 y posteriores de Service Fabric admiten la implementación de contenedores en clústeres que se ejecutan en la versión 1709 de Windows Server.  
> 

## <a name="define-the-docker-container"></a>Definición del contenedor de Docker

Compile una imagen basada en la [imagen de Python](https://hub.docker.com/_/python/) ubicada en Docker Hub.

Especifique el contenedor de Docker en un archivo Dockerfile. El archivo Dockerfile consiste en instrucciones para la configuración del entorno dentro del contenedor, la carga de la aplicación que desea ejecutar y la asignación de puertos. El Dockerfile es la entrada para el comando `docker build`, que crea la imagen.

Cree un directorio vacío y cree el archivo *Dockerfile* (sin extensión de archivo). Agregue lo siguiente al archivo *Dockerfile* y guarde los cambios:

```
# Use an official Python runtime as a base image
FROM python:2.7-windowsservercore

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Para más información, lea la [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="create-a-basic-web-application"></a>Creación de una aplicación web básica
Cree una aplicación web de Flask que escucha en el puerto 80 y devuelve `Hello World!`. En el mismo directorio, cree el archivo *requirements.txt*. Agregue lo siguiente y guarde los cambios:
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

<a id="Build-Containers"></a>
## <a name="build-the-image"></a>Compilación de la imagen
Ejecute el comando `docker build` para crear la imagen que ejecuta la aplicación web. Abra una ventana de PowerShell y navegue hasta el directorio que contiene el archivo Dockerfile. Ejecute el siguiente comando:

```
docker build -t helloworldapp .
```

Este comando crea la imagen nueva con las instrucciones del Dockerfile y asigna el nombre (etiqueta -t) `helloworldapp` a la imagen. Para crear una imagen de contenedor, primero se descarga la imagen base del concentrador de Docker al que se agrega la aplicación. 

Una vez que se complete el comando de compilación, ejecute el comando `docker images` para ver información sobre la nueva imagen:

```
$ docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              8ce25f5d6a79        2 minutes ago       10.4 GB
```

## <a name="run-the-application-locally"></a>Ejecución de la aplicación de forma local
Compruebe la imagen localmente antes de insertarla en el registro de contenedor. 

Ejecute la aplicación:

```
docker run -d --name my-web-site helloworldapp
```

*name* asigna un nombre al contenedor en ejecución (en lugar del identificador del contenedor).

Una vez que el contenedor se inicia, busque su dirección IP para que pueda conectarse a su contenedor en ejecución desde un explorador:
```
docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" my-web-site
```

Si este comando no devuelve nada, ejecute el siguiente e inspeccione la dirección IP del elemento **NetworkSettings**->**Networks**:
```
docker inspect my-web-site
```

Conéctese al contenedor en ejecución. Abra un explorador web que apunte a la dirección IP devuelta; por ejemplo, "http:\//172.31.194.61". Debería ver que el título "¡Hola mundo!" se muestra en el explorador.

Para detener el contenedor, ejecute:

```
docker stop my-web-site
```

Elimine el contenedor del equipo de desarrollo:

```
docker rm my-web-site
```

<a id="Push-Containers"></a>
## <a name="push-the-image-to-the-container-registry"></a>Inserción de la imagen en el registro de contenedor

Después de comprobar que el contenedor se ejecuta en el equipo de desarrollo, inserte la imagen en el registro de Azure Container Registry.

Ejecute ``docker login`` para iniciar sesión en el registro de contenedor con sus [credenciales de registro](../container-registry/container-registry-authentication.md).

En el ejemplo siguiente se pasa el identificador y la contraseña de una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md) de Azure Active Directory. Por ejemplo, puede que haya asignado una entidad de servicio al registro para ver un escenario de automatización. O bien, puede iniciar sesión con su nombre de usuario y contraseña del registro.

```
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

El siguiente comando crea una etiqueta, o alias, de la imagen, con una ruta de acceso completa al registro. Este ejemplo coloca la imagen en el espacio de nombres ```samples``` para evitar el desorden en la raíz del registro.

```
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

Inserte la imagen en el registro de contenedor:

```
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="create-the-containerized-service-in-visual-studio"></a>Creación del servicio en contenedor en Visual Studio
Las herramientas y el SDK de Service Fabric proporcionan una plantilla de servicio que le ayuda a crear una aplicación en contenedor.

1. Inicie Visual Studio. Seleccione **File (Archivo)**  > **New (Nuevo)**  > **Project (Proyecto)** .
2. Seleccione **Aplicación de Service Fabric**, asígnele el nombre "MyFirstContainer" y haga clic en **Aceptar**.
3. Seleccione **Contenedor** en la lista de **plantillas del servicio**.
4. En **Nombre de la imagen** escriba "myregistry.azurecr.io/samples/helloworldapp", la imagen que insertó en el repositorio de contenedor.
5. Asigne un nombre a su servicio y haga clic en **Aceptar**.

## <a name="configure-communication"></a>Configuración de la comunicación
El servicio en contenedor necesita un punto de conexión para la comunicación. Agregar un elemento `Endpoint` con el protocolo, el puerto y el tipo en el archivo ServiceManifest.xml. En este ejemplo, se usa un puerto fijo 8081. Si no se especifica ningún puerto, se elige un puerto aleatorio dentro del intervalo de puertos de la aplicación. 

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
  </Endpoints>
</Resources>
```
> [!NOTE]
> Pueden agregarse puntos de conexión adicionales para un servicio mediante la declaración de más elementos EndPoint con los valores de propiedad aplicables. Cada puerto solo puede declarar un valor de protocolo.

Al definir un punto de conexión, Service Fabric publica el punto de conexión en el servicio de nomenclatura. Otros servicios que se ejecuten en el clúster pueden resolver este contenedor. También puede realizar la comunicación de contenedor a contenedor utilizando el [proxy inverso](service-fabric-reverseproxy.md). La comunicación se establece al proporcionar el puerto de escucha HTTP del proxy inverso y el nombre de los servicios con los que desea comunicarse como variables de entorno.

El servicio escucha en un determinado puerto (8081 en este ejemplo). Cuando la aplicación se implementa en un clúster de Azure, el clúster y la aplicación se ejecutan detrás de un equilibrador de carga de Azure. El puerto de la aplicación debe estar abierto en el equilibrador de carga de Azure para que el tráfico entrante pueda pasar al servicio.  Puede abrir este puerto en el equilibrador de carga de Azure mediante un [script de PowerShell](./scripts/service-fabric-powershell-open-port-in-load-balancer.md) o en [Azure Portal](https://portal.azure.com).

## <a name="configure-and-set-environment-variables"></a>Configuración y establecimiento de variables de entorno
En el manifiesto de servicio, las variables de entorno se pueden especificar para cada paquete de código. Esta característica está disponible para todos los servicios, con independencia de si se implementan como contenedores, procesos o archivos ejecutables de invitado. Estos valores de variable de entorno se invalidan en el manifiesto de aplicación o se especifican durante la implementación como parámetros de la aplicación.

El siguiente fragmento de código XML del manifiesto de servicio muestra un ejemplo de cómo especificar variables de entorno para un paquete de código:
```xml
<CodePackage Name="Code" Version="1.0.0">
  ...
  <EnvironmentVariables>
    <EnvironmentVariable Name="HttpGatewayPort" Value=""/>    
  </EnvironmentVariables>
</CodePackage>
```

Estas variables de entorno se pueden invalidar en el manifiesto de aplicación:

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
  <EnvironmentOverrides CodePackageRef="FrontendService.Code">
    <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
  </EnvironmentOverrides>
  ...
</ServiceManifestImport>
```

## <a name="configure-container-port-to-host-port-mapping-and-container-to-container-discovery"></a>Configurar la asignación de puerto de puerto a host del contenedor y la detección de contenedor a contenedor
Configurar un puerto de host que se utiliza para comunicarse con el contenedor. El enlace de puerto asigna el puerto en el que el servicio está escuchando dentro del contenedor a un puerto en el host. Agregar un elemento `PortBinding` en el elemento `ContainerHostPolicies` del archivo ApplicationManifest.xml. Para este artículo, `ContainerPort` es 80 (el contenedor expone el puerto 80, como se especifica en el Dockerfile) y `EndpointRef` es "Guest1TypeEndpoint" (el punto de conexión definido previamente en el manifiesto de servicio). Las solicitudes entrantes para el servicio en el puerto 8081 se asignan al puerto 80 del contenedor.

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```
> [!NOTE]
> Pueden agregarse puertos de enlace adicionales para un servicio mediante la declaración de elementos PortBindings adicionales con los valores de propiedad aplicables.

## <a name="configure-container-repository-authentication"></a>Configuración la autenticación de repositorio del contenedor

Consulte [autenticación del repositorio de contenedores](configure-container-repository-credentials.md) para obtener información sobre cómo configurar diferentes tipos de autenticación para la descarga de imágenes de contenedor.

## <a name="configure-isolation-mode"></a>Configuración del modo de aislamiento
Windows admite dos modos de aislamiento para contenedores: de proceso y de Hyper-V. Con el modo de aislamiento de proceso, todos los contenedores que se ejecutan en la misma máquina host comparten el kernel con el host. Con el modo de aislamiento de Hyper-V, los kernels se aíslan entre los contenedores de Hyper-V y el host del contenedor. El modo de aislamiento se especifica en el elemento `ContainerHostPolicies` del archivo de manifiesto de aplicación. Los modos de aislamiento que se pueden especificar son `process`, `hyperv` y `default`. En los hosts de Windows Server, el valor predeterminado es el modo de aislamiento de proceso. En los hosts de Windows 10, solamente se admite el modo de aislamiento de Hyper-V, por lo que el contenedor se ejecutará en este modo independientemente de la configuración de modo de aislamiento establecida. El siguiente fragmento de código muestra cómo el modo de aislamiento se especifica en el archivo de manifiesto de aplicación.

```xml
<ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
```
   > [!NOTE]
   > El modo de aislamiento de HyperV está disponible en las SKU Ev3 y Dv3 de Azure que tienen compatibilidad con la virtualización anidada. 
   >
   >

## <a name="configure-resource-governance"></a>Configuración de la gobernanza de recursos
La [gobernanza de recursos](service-fabric-resource-governance.md) restringe los recursos que el contenedor puede usar en el host. El elemento `ResourceGovernancePolicy`, especificado en el manifiesto de la aplicación, se utiliza para declarar los límites de recursos para un paquete de código de servicio. Es posible establecer límites para los siguientes recursos: memoria, MemorySwap, CpuShares (peso relativo de CPU), MemoryReservationInMB, BlkioWeight (peso relativo de BlockIO). En este ejemplo, el paquete de servicio Guest1Pkg obtiene un núcleo en los nodos del clúster en los que es situado. Los límites de memoria son absolutos, por lo que el paquete de código está limitado a 1024 MB de memoria (con una reserva de garantía flexible de dicha capacidad). Los paquetes de código (contenedores o procesos) no pueden asignar más memoria de la que establece este límite; si se intenta, el resultado es una excepción de memoria insuficiente. Para que la aplicación del límite de recursos funcione, es necesario haber definido límites de memoria en todos los paquetes de código de un paquete de servicio.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
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

![HealthCheckHealthy][3]

![HealthCheckUnhealthyApp][4]

![HealthCheckUnhealthyDsp][5]

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

## <a name="deploy-the-container-application"></a>Implementación de la aplicación contenedora
Guarde todos los cambios y compile la aplicación. Para publicar la aplicación, haga clic con el botón derecho en **MyFirstContainer** en el Explorador de soluciones y seleccione **Publicar**.

En **Punto de conexión de la conexión**, escriba el punto de conexión de administración del clúster. Por ejemplo, `containercluster.westus2.cloudapp.azure.com:19000`. El punto de conexión del cliente se puede encontrar en la pestaña Información general del clúster en [Azure Portal](https://portal.azure.com).

Haga clic en **Publicar**.

[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) es una herramienta web para inspeccionar y administrar aplicaciones y nodos en un clúster de Service Fabric. Abra un explorador y vaya a `http://containercluster.westus2.cloudapp.azure.com:19080/Explorer/` y siga la implementación de aplicaciones. La aplicación se implementa, pero está en estado de error hasta que la imagen se descarga en los nodos del clúster (lo que pueden tardar un tiempo, según el tamaño de imagen): ![Error][1]

La aplicación está lista cuando está en el estado ```Ready```: ![Listo][2]

Abra un explorador y vaya a `http://containercluster.westus2.cloudapp.azure.com:8081`. Debería ver que el título "¡Hola mundo!" se muestra en el explorador.

## <a name="clean-up"></a>Limpieza

Mientras el clúster esté en ejecución seguirá generando cargos, así que debería considerar la posibilidad de [eliminar el clúster](./service-fabric-tutorial-delete-cluster.md).

Después de insertar la imagen en el registro de contenedor puede eliminar la imagen local del equipo de desarrollo:

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="windows-server-container-os-and-host-os-compatibility"></a>Compatibilidad entre el sistema operativo del contenedor y del host en Windows Server

Los contenedores de Windows Server no son compatibles con todas las versiones del sistema operativo de un host. Por ejemplo:
 
- Los contenedores de Windows Server creados con la versión 1709 de Windows Server no funcionarán en un host que tenga la versión 2016 de Windows Server. 
- Los contenedores de Windows Server creados con Windows Server 2016 solamente podrán utilizar el modo de aislamiento de Hyper-V en un host que tenga la versión 1709 de Windows Server. 
- En el caso de los contenedores de Windows Server creados con Windows Server 2016, tal vez sea necesario garantizar que la revisión del sistema operativo del contenedor y del sistema operativo del host es la misma cuando se utiliza el modo de aislamiento de proceso en un host con Windows Server 2016.
 
Para más información, consulte [Compatibilidad con versiones de contenedores de Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

Tenga en cuenta la compatibilidad de los sistemas operativos del host y del contenedor al compilar e implementar contenedores en el clúster de Service Fabric. Por ejemplo:

- Asegúrese de que implementa los contenedores utilizando un sistema operativo compatible con el sistema operativo de los nodos del clúster.
- Asegúrese de que el modo de aislamiento especificado en la aplicación de contenedor es coherente y compatible con el sistema operativo del contenedor que se ejecuta en el nodo donde se está implementando.
- Tenga en cuenta el modo en que las actualizaciones del sistema operativo de los contenedores y los nodos del clúster pueden afectar a la compatibilidad. 

Le recomendamos que aplique las prácticas siguientes para asegurarse de que los contenedores están correctamente implementados en el clúster de Service Fabric:

- Utilice un etiquetado de imagen explícito con las imágenes de Docker para especificar la versión del sistema operativo de Windows Server a partir de la que se crea un contenedor. 
- Utilice el [etiquetado del sistema operativo](#specify-os-build-specific-container-images) en el archivo de manifiesto de la aplicación para asegurarse de que la aplicación es compatible con diferentes versiones y actualizaciones de Windows Server.

> [!NOTE]
> Con Service Fabric 6.2 y versiones posteriores, puede implementar localmente contenedores basados en Windows Server 2016 en un host de Windows 10. En Windows 10, los contenedores se ejecutan en modo de aislamiento de Hyper-V independientemente del modo de aislamiento definido en el manifiesto de aplicación. Para más información, consulte [Configuración del modo de aislamiento](#configure-isolation-mode).   
>
 
## <a name="specify-os-build-specific-container-images"></a>Especificación de las imágenes de contenedores de compilación específica del sistema operativo 

Es posible que los contenedores Windows Server no sean compatibles con las distintas versiones del sistema operativo. Por ejemplo, los contenedores de Windows Server creados con Windows Server 2016 no funcionan con la versión 1709 de Windows Server en el modo de aislamiento de proceso. Por tanto, si los nodos de clúster se actualizan a la versión más reciente, pueden producirse errores en los servicios de contenedor compilados con versiones anteriores del sistema operativo. Para evitar esto en la versión 6.1 y posteriores del runtime, Service Fabric permite especificar varias imágenes del sistema operativo en cada contenedor y etiquetarlas con las versiones de compilación del sistema operativo en el manifiesto de aplicación. Puede obtener la versión de compilación del sistema operativo ejecutando `winver` en un símbolo del sistema de Windows. Actualice primero los manifiestos de aplicación y especifique los reemplazos de imágenes por versión del sistema operativo antes de actualizar este en los nodos. El fragmento de código siguiente muestra cómo especificar varias imágenes de contenedor en el manifiesto de aplicación, **ApplicationManifest.xml**:


```xml
      <ContainerHostPolicies> 
         <ImageOverrides> 
           <Image Name="myregistry.azurecr.io/samples/helloworldappDefault" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1701" Os="14393" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1709" Os="16299" /> 
         </ImageOverrides> 
      </ContainerHostPolicies> 
```
La versión de compilación de Windows Server 2016 es 14393 y la versión de compilación de la versión 1709 de Windows Server es 16299. El manifiesto de servicio sigue especificando solamente una imagen por cada servicio de contenedor como se muestra a continuación:

```xml
<ContainerHost>
    <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName> 
</ContainerHost>
```

   > [!NOTE]
   > Las características de etiquetado de la versión de compilación del sistema operativo solo están disponibles para Service Fabric en Windows
   >

Si el sistema operativo subyacente de la máquina virtual tiene la versión de compilación 16299 (versión 1709), Service Fabric seleccionará la imagen de contenedor que se corresponde con esa versión de Windows Server. Si se proporciona también una imagen de contenedor sin etiquetar junto con otras imágenes etiquetadas en el manifiesto de aplicación, Service Fabric considerará a la imagen sin etiquetar como una imagen que funciona en distintas versiones. Etiquete las imágenes de contenedor explícitamente para evitar problemas durante las actualizaciones.

La imagen del contenedor no etiquetado invalida la que se proporciona en el manifiesto ServiceManifest. Por lo tanto, la imagen "myregistry.azurecr.io/samples/helloworldappDefault" reemplazará la imagen "myregistry.azurecr.io/samples/helloworldapp" en ServiceManifest.

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Manifiestos de servicio y de aplicación de Service Fabric de ejemplo completos
Estos son los manifiestos de servicio y de aplicación completos que se usan en este artículo.

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Guest1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="Guest1Type" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->    
    <EnvironmentVariables>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
    </EnvironmentVariables>

  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to
           listen. Please note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="MyFirstContainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Guest1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
    <EnvironmentOverrides CodePackageRef="FrontendService.Code">
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
    </EnvironmentOverrides>
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="MIIB6QYJKoZIhvcNAQcDoIIB2jCCAdYCAQAxggFRMIIBTQIBADA1MCExHzAdBgNVBAMMFnJ5YW53aWRhdGFlbmNpcGhlcm1lbnQCEFfyjOX/17S6RIoSjA6UZ1QwDQYJKoZIhvcNAQEHMAAEg
gEAS7oqxvoz8i6+8zULhDzFpBpOTLU+c2mhBdqXpkLwVfcmWUNA82rEWG57Vl1jZXe7J9BkW9ly4xhU8BbARkZHLEuKqg0saTrTHsMBQ6KMQDotSdU8m8Y2BR5Y100wRjvVx3y5+iNYuy/JmM
gSrNyyMQ/45HfMuVb5B4rwnuP8PAkXNT9VLbPeqAfxsMkYg+vGCDEtd8m+bX/7Xgp/kfwxymOuUCrq/YmSwe9QTG3pBri7Hq1K3zEpX4FH/7W2Zb4o3fBAQ+FuxH4nFjFNoYG29inL0bKEcTX
yNZNKrvhdM3n1Uk/8W2Hr62FQ33HgeFR1yxQjLsUu800PrYcR5tLfyTB8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBybgM5NUV8BeetUbMR8mJhgFBrVSUsnp9B8RyebmtgU36dZiSObDsI
NtTvlzhk11LIlae/5kjPv95r3lw6DHmV4kXLwiCNlcWPYIWBGIuspwyG+28EWSrHmN7Dt2WqEWqeNQ==" PasswordEncrypted="true"/>
        <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
      </ContainerHostPolicies>
      <ServicePackageResourceGovernancePolicy CpuCores="1"/>
      <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this
         application type is created. You can also create one or more instances of service type using the
         ServiceFabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Guest1">
      <StatelessService ServiceTypeName="Guest1Type" InstanceCount="[Guest1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="configure-time-interval-before-container-is-force-terminated"></a>Configuración del intervalo de tiempo antes de hacer que se termine el contenedor

Puede configurar un intervalo de tiempo para que el entorno de ejecución espere antes de que el contenedor se quite una vez que la eliminación del servicio (o el movimiento a otro nodo) se haya iniciado. Si se configura el intervalo de tiempo, se envía el comando `docker stop <time in seconds>` al contenedor.  Para más detalles, consulte [docker stop](https://docs.docker.com/engine/reference/commandline/stop/). El intervalo de tiempo de espera se especifica en la sección `Hosting`. La sección `Hosting` puede agregarse al crear el clúster o posteriormente en una actualización de la configuración. El siguiente fragmento de manifiesto de clúster muestra cómo establecer el intervalo de espera:

```json
"fabricSettings": [
    ...,
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
]
```
El intervalo de tiempo predeterminado se establece en diez segundos. Puesto que esta configuración es dinámica, una actualización de solo configuración en el clúster actualiza el tiempo de espera. 


## <a name="configure-the-runtime-to-remove-unused-container-images"></a>Configuración del entorno de ejecución para quitar imágenes de contenedor sin usar

Puede configurar el clúster de Service Fabric para quitar del nodo las imágenes de contenedor sin usar. Esta configuración permite recuperar el espacio en disco si hay demasiadas imágenes de contenedor en el nodo. Para habilitar esta característica, actualice la sección [Hospedaje](service-fabric-cluster-fabric-settings.md#hosting) en el manifiesto de clúster, tal como se muestra en el fragmento siguiente: 


```json
"fabricSettings": [
    ...,
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
]
```

Puede especificar las imágenes que no se deben eliminar en el parámetro `ContainerImagesToSkip`.  


## <a name="configure-container-image-download-time"></a>Configuración del tiempo de descarga de la imagen de contenedor

El entorno de tiempo de ejecución de Service Fabric asigna 20 minutos para descargar y extraer las imágenes de contenedor, lo cual suele bastar en la mayoría de los casos. En el caso de imágenes grandes, o si la conexión de red es lenta, puede que sea necesario aumentar el tiempo de espera antes de anular la descarga y extracción de la imagen. El tiempo de espera se establece mediante el atributo **ContainerImageDownloadTimeout** de la sección de **hospedaje** del manifiesto de clúster, tal y como se muestra en el siguiente fragmento de código:

```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
    }
]
```


## <a name="set-container-retention-policy"></a>Establecimiento de la directiva de retención de contenedor

Para ayudar a diagnosticar los errores de inicio del contenedor, Service Fabric (versión 6.1 o posterior) admite la retención de contenedores que finalizaron o que produjeron un error en el inicio. Esta directiva se puede establecer en el archivo **ApplicationManifest.xml** tal y como se muestra en el siguiente fragmento de código:

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

El valor **ContainersRetentionCount** especifica el número de contenedores que se conservarán cuando se produzca un error en ellos. Si se especifica un valor negativo, se conservarán todos los contenedores con errores. Si no se especifica el atributo **ContainersRetentionCount**, no se conservará ningún contenedor. El atributo **ContainersRetentionCount** también admite parámetros de aplicación, por lo que los usuarios pueden especificar valores diferentes para los clústeres de prueba y de producción. Utilice restricciones de colocación para seleccionar un destino para el servicio de contenedor en un nodo concreto al usar estas características, para impedir que este se mueva a otros nodos. Los contenedores que se conserven mediante esta característica deben quitarse manualmente.

## <a name="start-the-docker-daemon-with-custom-arguments"></a>Inicio del demonio de Docker con argumentos personalizados

Con la versión 6.2 del entorno del tiempo de ejecución de Service Fabric y superior, puede iniciar el demonio de Docker con argumentos personalizados. Cuando se especifican argumentos personalizados, Service Fabric no pasa ningún otro argumento al motor de Docker excepto el argumento `--pidfile`. Por lo tanto, `--pidfile` no debe pasarse como argumento. Además, el argumento debería continuar haciendo que el demonio de Docker escuche en la canalización del nombre predeterminado en Windows (o en el socket de dominio Unix en Linux) para que Service Fabric se comunique con el demonio. Los argumentos personalizados se pasan en el manifiesto de clúster bajo la sección **Hospedaje** en **ContainerServiceArguments**  como se muestra en el siguiente fragmento de código: 
 

```json
"fabricSettings": [
    ...,
    { 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
    } 
]
```

## <a name="next-steps"></a>Pasos siguientes
* Más información acerca de cómo ejecutar [contenedores en Service Fabric](service-fabric-containers-overview.md).
* Consulte el tutorial [Implementación de una aplicación .NET en un contenedor](service-fabric-host-app-in-a-container.md).
* Más información acerca del [ciclo de vida de aplicaciones](service-fabric-application-lifecycle.md) de Service Fabric.
* Consulte [los ejemplos de código de contenedor de Service Fabric](https://github.com/Azure-Samples/service-fabric-containers) en GitHub.

[1]: ./media/service-fabric-get-started-containers/MyFirstContainerError.png
[2]: ./media/service-fabric-get-started-containers/MyFirstContainerReady.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[4]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[5]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png
