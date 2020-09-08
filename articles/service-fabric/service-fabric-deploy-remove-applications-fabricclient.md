---
title: Implementación de Azure Service Fabric con FabricClient
description: Use las API de FabricClient para implementar y quitar aplicaciones de Service Fabric.
ms.topic: conceptual
ms.date: 01/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 565e6b8f23f159a5c231295694830917217a3d19
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89009308"
---
# <a name="deploy-and-remove-applications-using-fabricclient"></a>Implementación y eliminación de aplicaciones mediante FabricClient
> [!div class="op_single_selector"]
> * [Resource Manager](service-fabric-application-arm-resource.md)
> * [PowerShell](service-fabric-deploy-remove-applications.md)
> * [CLI de Service Fabric](service-fabric-application-lifecycle-sfctl.md)
> * [API de FabricClient](service-fabric-deploy-remove-applications-fabricclient.md)
> 
> 

<br/>

Una vez que un [tipo de aplicación se ha empaquetado][10], está listo para la implementación en un clúster de Azure Service Fabric. La implementación implica los tres pasos siguientes:

1. Carga del paquete de aplicación en el almacén de imágenes
2. Cargar el tipo de aplicación
3. Quitar el paquete de aplicación del almacén de imágenes
4. Crear la instancia de aplicación

Después de implementar una aplicación y ejecutar una instancia en el clúster, puede eliminar la instancia de aplicación y su tipo de aplicación. Para quitar completamente una aplicación del clúster, realice los siguientes pasos:

1. Quitar (o eliminar) la instancia de la aplicación en ejecución
2. Anular el registro del tipo de aplicación si ya no lo necesita

Si usa Visual Studio para implementar y depurar aplicaciones en el clúster de desarrollo local, todos los pasos anteriores se controlan automáticamente mediante un script de PowerShell.  que se encuentra en la carpeta *Scripts* del proyecto de la aplicación. En este artículo se ofrece información sobre lo que hace ese script para que pueda realizar las mismas operaciones fuera de Visual Studio. 
 
## <a name="connect-to-the-cluster"></a>Conectarse al clúster
Para conectarse al clúster, cree una instancia de [FabricClient](/dotnet/api/system.fabric.fabricclient) antes de ejecutar cualquiera de los ejemplos de código de este artículo. Para obtener ejemplos de conexión a un clúster de desarrollo local, a un clúster remoto o a un cluster protegido con Azure Active Directory, certificados X509 o Windows Active Directory, consulte [Conexión a un clúster seguro](service-fabric-connect-to-secure-cluster.md#connect-to-a-cluster-using-the-fabricclient-apis). Para conectarse al clúster de desarrollo local, ejecute el ejemplo siguiente:

```csharp
// Connect to the local cluster.
FabricClient fabricClient = new FabricClient();
```

## <a name="upload-the-application-package"></a>Cargar el paquete de la aplicación
Imagine que compila y empaqueta una aplicación denominada *MyApplication* en Visual Studio. De forma predeterminada, el nombre del tipo de aplicación que aparece en ApplicationManifest.xml es "MyApplicationType".  El paquete de aplicación, que contiene el manifiesto de aplicación necesario, así como los manifiestos de servicio y los paquetes code/config/data, se encuentra en *C:\Users\&username&gt;\Documents\Visual Studio 2019\Projects\MyApplication\MyApplication\pkg\Debug*.

Cargar el paquete de aplicación lo pone en una ubicación a la que pueden tener acceso los componentes internos de Service Fabric. Service Fabric comprueba el paquete de aplicación durante su registro. Sin embargo, si quiere comprobar el paquete de aplicación de forma local (es decir, antes de cargarlo), use el cmdlet [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps).

La API [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) carga el paquete de aplicación en el almacén de imágenes del clúster. 

Si el paquete de aplicación es grande o tiene muchos archivos, puede [comprimirlo](service-fabric-package-apps.md#compress-a-package) y copiarlo en el almacén de imágenes mediante PowerShell. La compresión reduce el tamaño y el número de archivos.

Consulte [Descripción del valor ImageStoreConnectionString](service-fabric-image-store-connection-string.md) para más información sobre el almacén de imágenes y su cadena de conexión.

## <a name="register-the-application-package"></a>Registrar el paquete de la aplicación
El tipo y la versión de la aplicación declarados en el manifiesto de aplicación estarán disponibles para usarse cuando se registre el paquete de aplicación. El sistema leerá el paquete cargado en el paso anterior, comprobará dicho paquete, procesará su contenido y copiará el paquete procesado en una ubicación del sistema interno.  

La API [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) registra el tipo de aplicación en el clúster y hace que esté disponible para su implementación.

La API [GetApplicationTypeListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationtypelistasync) proporciona información acerca de todos los tipos de aplicaciones registradas correctamente. Puede usar esta API para determinar cuándo se realiza el registro.

## <a name="remove-an-application-package-from-the-image-store"></a>Eliminación de un paquete de aplicación del almacén de imágenes
Se recomienda que quite el paquete de aplicación después de que la aplicación se haya registrado correctamente.  Al eliminar los paquetes de aplicación del almacén de imágenes, se liberan recursos del sistema.  Mantener los paquetes de aplicación sin usar consume almacenamiento en disco y conduce a problemas de rendimiento de la aplicación. Elimine el paquete de aplicación del almacén de imágenes mediante la API [RemoveApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.removeapplicationpackage).

## <a name="create-an-application-instance"></a>Creación de una instancia de aplicación
Se pueden crear instancias de una aplicación a partir de cualquier tipo de aplicación que se haya registrado correctamente mediante la API [CreateApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.createapplicationasync). El nombre de cada aplicación debe empezar con el esquema *"fabric:"* y ser único para cada instancia de la aplicación (dentro de un clúster). También se crean los servicios predeterminados que se hayan definido en el manifiesto de aplicación del tipo de aplicación de destino.

Pueden crearse varias instancias de aplicación para cualquier versión concreta de un tipo de aplicación registrado. Cada instancia de la aplicación se ejecuta de forma aislada, con su propio directorio de trabajo y conjunto de procesos.

Para ver qué aplicaciones y servicios con nombre se ejecutan en el clúster, ejecute las API [GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync) y [GetServiceListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync).

## <a name="create-a-service-instance"></a>Creación de una instancia de servicio
Puede crear una instancia de un servicio a partir de un tipo de servicio mediante la API [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync).  Si el servicio se declara como servicio predeterminado en el manifiesto de aplicación, se crea una instancia de este con la aplicación.  La llamada a la API [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) de un servicio del que ya se ha creado una instancia devolverá una excepción de tipo FabricException. La excepción contendrá un código de error con un valor de FabricErrorCode.ServiceAlreadyExists.

## <a name="remove-a-service-instance"></a>Eliminación de una instancia de servicio
Cuando una instancia de servicio deja de ser necesaria, para quitarla de la instancia de aplicación en funcionamiento, hay que llamar a la API [DeleteServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync).  

> [!WARNING]
> Esta operación no se puede revertir y el estado del servicio no se puede recuperar.

## <a name="remove-an-application-instance"></a>Eliminación de una instancia de aplicación
Cuando una instancia de aplicación deja de ser necesaria, puede quitarla de manera permanente por el nombre mediante la API [DeleteApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.deleteapplicationasync). [DeleteApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.deleteapplicationasync) quita automáticamente todos los servicios que pertenecen a la aplicación, con lo que se eliminan de forma permanente todos los estados de servicio.

> [!WARNING]
> No se puede deshacer esta operación y no se puede recuperar el estado de la aplicación.

## <a name="unregister-an-application-type"></a>Anulación de un registro del tipo de aplicación
Cuando ya no se necesita una versión concreta de un tipo de aplicación, debe anularse el registro de esa versión en particular del tipo de aplicación mediante la API [Unregister-ServiceFabricApplicationType](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.unprovisionapplicationasync). La anulación del registro de versiones no usadas de tipos de aplicación permite liberar espacio de almacenamiento usado en el almacén de imágenes. El registro de una versión de un tipo de aplicación se puede anular mientras no haya ninguna aplicación con instancias de esa versión de tipo de aplicación. Además, el tipo de aplicación no puede tener ninguna actualización de la aplicación pendiente que haga referencia a esa versión del tipo de aplicación.

## <a name="troubleshooting"></a>Solución de problemas
### <a name="copy-servicefabricapplicationpackage-asks-for-an-imagestoreconnectionstring"></a>Copy-ServiceFabricApplicationPackage pide una ImageStoreConnectionString
El entorno del SDK de Service Fabric ya debe tener configurados los valores predeterminados correctos. Pero si es necesario, ImageStoreConnectionString para todos los comandos debe coincidir con el valor que usa el clúster de Service Fabric. Puede encontrar ImageStoreConnectionString en el manifiesto del clúster, recuperado mediante los comandos [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest?view=azureservicefabricps) y Get-ImageStoreConnectionStringFromClusterManifest:

```powershell
PS C:\> Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)
```

El cmdlet **Get-ImageStoreConnectionStringFromClusterManifest** , que forma parte del módulo de PowerShell correspondiente al SDK de Service Fabric, se utiliza para obtener la cadena de conexión del almacén de imágenes.  Para importar el módulo de SDK, ejecute el siguiente código:

```powershell
Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
```


ImageStoreConnectionString se encuentra en el manifiesto de clúster:

```xml
<ClusterManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Name="Server-Default-SingleNode" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

    [...]

    <Section Name="Management">
      <Parameter Name="ImageStoreConnectionString" Value="file:D:\ServiceFabric\Data\ImageStore" />
    </Section>

    [...]
```

Consulte [Descripción del valor ImageStoreConnectionString](service-fabric-image-store-connection-string.md) para más información sobre el almacén de imágenes y su cadena de conexión.

### <a name="deploy-large-application-package"></a>Implementación de un paquete de aplicación grande
Problema: la API [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) agota el tiempo de espera para un paquete de aplicación grande (del orden de GB).
Pruebe lo siguiente:
- Especifique un tiempo de espera mayor para el método [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) con el parámetro `timeout`. De forma predeterminada, el tiempo de espera es de 30 minutos.
- Compruebe la conexión de red entre la máquina de origen y el clúster. Si la conexión es lenta, considere la posibilidad de usar una máquina con una conexión de red mejor.
Si la máquina cliente se encuentra en otra región diferente al clúster, considere el uso de una máquina cliente que se encuentre en una región más cercana o en la misma región que el clúster.
- Compruebe si está alcanzando la limitación externa. Por ejemplo, cuando el almacén de imágenes está configurado para usar Azure Storage, se puede limitar carga.

Problema: la carga del paquete se completó correctamente, pero la API [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) agota el tiempo de espera. Pruebe lo siguiente:
- [Comprima el paquete](service-fabric-package-apps.md#compress-a-package) antes de realizar la copia en el almacén de imágenes.
La compresión reduce el tamaño y el número de archivos lo que, a su vez, reduce la cantidad de tráfico y trabajo que Service Fabric debe realizar. La operación de carga puede ser más lenta (especialmente si incluye el tiempo de compresión), pero el registro y la anulación del registro del tipo de aplicación son más rápidos.
- Especifique un tiempo de espera mayor para la API [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) con el parámetro `timeout`.

### <a name="deploy-application-package-with-many-files"></a>Implementación del paquete de aplicación con muchos archivos
Problema: [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) agota el tiempo de espera para un paquete de aplicación con muchos archivos (del orden de miles).
Pruebe lo siguiente:
- [Comprima el paquete](service-fabric-package-apps.md#compress-a-package) antes de realizar la copia en el almacén de imágenes. La compresión reduce el número de archivos.
- Especifique un tiempo de espera mayor para [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) con el parámetro `timeout`.

## <a name="code-example"></a>Ejemplo de código
En el ejemplo siguiente se copia un paquete de aplicación en el almacén de imágenes y se aprovisiona el tipo de aplicación. A continuación, en el ejemplo se crea una instancia de la aplicación y una instancia del servicio. Finalmente, en el ejemplo se elimina la instancia de la aplicación, se desaprovisiona el tipo de aplicación y se elimina el paquete de aplicación del almacén de imágenes.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Reflection;
using System.Threading.Tasks;

using System.Fabric;
using System.Fabric.Description;
using System.Threading;

namespace ServiceFabricAppLifecycle
{
class Program
{
static void Main(string[] args)
{

    string clusterConnection = "localhost:19000";
    string appName = "fabric:/MyApplication";
    string appType = "MyApplicationType";
    string appVersion = "1.0.0";
    string serviceName = "fabric:/MyApplication/Stateless1";
    string imageStoreConnectionString = "file:C:\\SfDevCluster\\Data\\ImageStoreShare";
    string packagePathInImageStore = "MyApplication";
    string packagePath = "C:\\Users\\username\\Documents\\Visual Studio 2019\\Projects\\MyApplication\\MyApplication\\pkg\\Debug";
    string serviceType = "Stateless1Type";

    // Connect to the cluster.
    FabricClient fabricClient = new FabricClient(clusterConnection);

    // Copy the application package to a location in the image store
    try
    {
        fabricClient.ApplicationManager.CopyApplicationPackage(imageStoreConnectionString, packagePath, packagePathInImageStore);
        Console.WriteLine("Application package copied to {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Application package copy to Image Store failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Provision the application.  "MyApplicationV1" is the folder in the image store where the application package is located. 
    // The application type with name "MyApplicationType" and version "1.0.0" (both are found in the application manifest) 
    // is now registered in the cluster.            
    try
    {
        fabricClient.ApplicationManager.ProvisionApplicationAsync(packagePathInImageStore).Wait();

        Console.WriteLine("Provisioned application type {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Provision Application Type failed:");

        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete the application package from a location in the image store.
    try
    {
        fabricClient.ApplicationManager.RemoveApplicationPackage(imageStoreConnectionString, packagePathInImageStore);
        Console.WriteLine("Application package removed from {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Application package removal from Image Store failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    //  Create the application instance.
    try
    {
        ApplicationDescription appDesc = new ApplicationDescription(new Uri(appName), appType, appVersion);
        fabricClient.ApplicationManager.CreateApplicationAsync(appDesc).Wait();
        Console.WriteLine("Created application instance of type {0}, version {1}", appType, appVersion);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("CreateApplication failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Create the stateless service description.  For stateful services, use a StatefulServiceDescription object.
    StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
    serviceDescription.ApplicationName = new Uri(appName);
    serviceDescription.InstanceCount = 1;
    serviceDescription.PartitionSchemeDescription = new SingletonPartitionSchemeDescription();
    serviceDescription.ServiceName = new Uri(serviceName);
    serviceDescription.ServiceTypeName = serviceType;

    // Create the service instance.  If the service is declared as a default service in the ApplicationManifest.xml,
    // the service instance is already running and this call will fail.
    try
    {
        fabricClient.ServiceManager.CreateServiceAsync(serviceDescription).Wait();
        Console.WriteLine("Created service instance {0}", serviceName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("CreateService failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete a service instance.
    try
    {
        DeleteServiceDescription deleteServiceDescription = new DeleteServiceDescription(new Uri(serviceName));

        fabricClient.ServiceManager.DeleteServiceAsync(deleteServiceDescription);
        Console.WriteLine("Deleted service instance {0}", serviceName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("DeleteService failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete an application instance from the application type.
    try
    {
        DeleteApplicationDescription deleteApplicationDescription = new DeleteApplicationDescription(new Uri(appName));

        fabricClient.ApplicationManager.DeleteApplicationAsync(deleteApplicationDescription).Wait();
        Console.WriteLine("Deleted application instance {0}", appName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("DeleteApplication failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Un-provision the application type.
    try
    {
        fabricClient.ApplicationManager.UnprovisionApplicationAsync(appType, appVersion).Wait();
        Console.WriteLine("Un-provisioned application type {0}, version {1}", appType, appVersion);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Un-provision application type failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    Console.WriteLine("Hit enter...");
    Console.Read();
}        
}
}

```

## <a name="next-steps"></a>Pasos siguientes
[Actualización de la aplicación de Service Fabric](service-fabric-application-upgrade.md)

[Introducción al estado de Service Fabric](service-fabric-health-introduction.md)

[Diagnosticar y solucionar problemas de un servicio de Service Fabric](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Modelar una aplicación en Service Fabric](service-fabric-application-model.md)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-package-apps.md
[11]: service-fabric-application-upgrade.md
