---
title: Empaquetado de una aplicación de Azure Service Fabric
description: Obtenga información sobre cómo empaquetar una aplicación de Azure Service Fabric y cómo prepararse para la implementación en un clúster.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 11a3fdd5dbaef53af321342952f786ed8119689c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012405"
---
# <a name="package-an-application"></a>Empaquetar una aplicación

En este artículo se describe cómo empaquetar una aplicación de Service Fabric y prepararla para su implementación.

## <a name="package-layout"></a>Diseño del paquete

El manifiesto de aplicación, uno o varios manifiestos de servicio y otros archivos de paquete necesarios deben organizarse en un diseño específico para la implementación en un clúster de Service Fabric. Los manifiestos de ejemplo en este artículo deberían organizarse con la siguiente estructura de directorios:

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

Las carpetas reciben un nombre para coincidir con los atributos **Nombre** de cada elemento correspondiente. Por ejemplo, si el manifiesto de servicio contenía dos paquetes de código con los nombres **MiCódigoA** y **MiCódigoB**, dos carpetas con los mismos nombres incluirán los archivos binarios necesarios para cada paquete de código.

## <a name="use-setupentrypoint"></a>Usar SetupEntrypoint

Los escenarios de uso de **SetupEntryPoint** habituales corresponden a cuando se necesita ejecutar un archivo ejecutable antes de iniciar el servicio o cuando necesita realizar una operación con privilegios elevados. Por ejemplo:

* configuración e inicialización de las variables de entorno que el ejecutable del servicio necesita. No se limita tan solo a los archivos ejecutables escritos a través de los modelos de programación de Service Fabric. Por ejemplo, npm.exe necesita algunas variables de entorno configuradas para implementar una aplicación node.js.
* Configuración del control de acceso mediante la instalación de certificados de seguridad.

Para obtener más información sobre cómo configurar **SetupEntryPoint**, vea [Configuración de la directiva para un punto de entrada del programa de instalación del servicio](service-fabric-application-runas-security.md).

<a id="Package-App"></a>

## <a name="configure"></a>Configuración

### <a name="build-a-package-by-using-visual-studio"></a>Crear un paquete mediante Visual Studio

Si usó Visual Studio para crear su aplicación, puede utilizar el comando *Empaquetar* para crear automáticamente un paquete que coincida con el diseño descrito anteriormente.

Para crear un paquete, haga clic con el botón derecho en el proyecto de aplicación en el *Explorador de soluciones* y elija el comando **Empaquetar**:

![Empaquetado de una aplicación con Visual Studio][vs-package-command]

Una vez completado el empaquetado, busque la ubicación del paquete en la ventana **Salida**. El paso de empaquetado se produce automáticamente al implementar o depurar la aplicación en Visual Studio.

### <a name="build-a-package-by-command-line"></a>Creación de un paquete mediante la línea de comandos

También es posible empaquetar la aplicación mediante programación usando `msbuild.exe`. En el fondo esto es lo que ejecuta Visual Studio, por lo que la salida es la misma.

```shell
D:\Temp> msbuild HelloWorld.sfproj /t:Package
```

## <a name="test-the-package"></a>Probar el paquete

Puede comprobar la estructura del paquete localmente a través de PowerShell mediante el comando [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) .
Este comando comprueba si existen problemas de análisis de manifiestos y verifica todas las referencias. Este comando solo comprueba la corrección estructural de los directorios y archivos del paquete.
No comprobará ningún contenido de código o de paquete de datos y se limitará a comprobar que todos los archivos necesarios están presentes.

```powershell
Test-ServiceFabricApplicationPackage .\MyApplicationType
```

```Output
False
Test-ServiceFabricApplicationPackage : The EntryPoint MySetup.bat is not found.
FileName: C:\Users\servicefabric\AppData\Local\Temp\TestApplicationPackage_7195781181\nrri205a.e2h\MyApplicationType\MyServiceManifest\ServiceManifest.xml
```

Este error muestra que el archivo *MySetup.bat* al que se hace referencia en el manifiesto de servicio **SetupEntryPoint** no está en el paquete de código. Después de agregar el archivo que falta, se comprueba la aplicación:

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

```powershell
Test-ServiceFabricApplicationPackage .\MyApplicationType
```

```Output
True
```

Si la aplicación tiene [parámetros de aplicación](service-fabric-manage-multiple-environment-app-configuration.md) definidos, puede pasarlos en [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) para realizar una validación adecuada.

Si conoce el clúster donde se implementará la aplicación, se recomienda pasar el parámetro `ImageStoreConnectionString`. En este caso, el paquete también se valida con las versiones anteriores de la aplicación que ya se está ejecutando en el clúster. Por ejemplo, la validación puede detectar si ya se implementó un paquete con la misma versión pero distinto contenido.  

Una vez que la aplicación se empaquete correctamente y pase el proceso de validación, considere la posibilidad de comprimir el paquete para realizar las operaciones de implementación más rápidamente.

## <a name="compress-a-package"></a>Compresión de un paquete

Cuando un paquete es grande o tiene muchos archivos, puede comprimirlo para acelerar la implementación. La compresión reduce el número de archivos y el tamaño del paquete.
Para un paquete de aplicación comprimido, la [carga del paquete de aplicación](service-fabric-deploy-remove-applications.md#upload-the-application-package) puede tardar más en comparación a la carga del paquete sin comprimir, especialmente si la compresión se realiza como parte de la copia. Con la compresión, [el registro](service-fabric-deploy-remove-applications.md#register-the-application-package) y [la anulación de un registro del tipo de aplicación](service-fabric-deploy-remove-applications.md#unregister-an-application-type) son más rápidos.

El mecanismo de implementación es el mismo para los paquetes comprimidos y sin comprimir. Si el paquete se comprime, se almacena como tal en el almacén de imágenes de clúster y se descomprime en el nodo antes de ejecutar la aplicación.
La compresión reemplaza el paquete de Service Fabric válido por la versión comprimida. La carpeta debe permitir permisos de escritura. La ejecución de la compresión en un paquete ya comprimido no produce ningún cambio.

Puede comprimir un paquete ejecutando el comando de Powershell [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) con el modificador `CompressPackage`. Puede descomprimir el paquete con el mismo comando, mediante el modificador `UncompressPackage`.

El siguiente comando comprime el paquete sin copiarlo en el almacén de imágenes. Puede copiar un paquete comprimido en uno o varios clústeres de Service Fabric, según sea necesario, con [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) sin la marca `SkipCopy`.
El paquete ahora incluye los archivos comprimidos para los paquetes `code`, `config` y `data`. El manifiesto de la aplicación y los manifiestos del servicio no se comprimen, ya que son necesarios para muchas operaciones internas. Por ejemplo, el uso compartido del paquete, el nombre del tipo de aplicación y la extracción de la versión para determinadas validaciones necesitan tener acceso a los manifiestos. La compresión de los manifiestos haría que estas operaciones resultaran ineficaces.

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -CompressPackage -SkipCopy
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
       ServiceManifest.xml
       MyCode.zip
       MyConfig.zip
       MyData.zip

```

Como alternativa, puede comprimir y copiar el paquete con [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) en un solo paso.
Si el paquete es grande, proporcione un tiempo de espera lo suficientemente alto como para dar tiempo tanto para la compresión del paquete como para la carga en el clúster.

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -ApplicationPackagePathInImageStore MyApplicationType -ImageStoreConnectionString fabric:ImageStore -CompressPackage -TimeoutSec 5400
```

Internamente, Service Fabric calcula las sumas de comprobación para los paquetes de aplicación para la validación. Cuando se utilice la compresión, las sumas de comprobación se calculan en las versiones comprimidas de cada paquete. Generar un nuevo archivo ZIP desde el mismo paquete de aplicación crea sumas de comprobación diferentes. Para evitar errores de validación, utilice el [aprovisionamiento de diferencias](service-fabric-application-upgrade-advanced.md). Con esta opción, no incluya los paquetes sin cambios en la nueva versión. En su lugar, acceda a ellos directamente desde el nuevo manifiesto del servicio.

Si el aprovisionamiento de diferencias no es una opción y debe incluir los paquetes, genere nuevas versiones para los paquetes `code`, `config` y `data` para evitar errores de coherencia en la suma de comprobación. Es necesario generar nuevas versiones de paquetes sin cambios cuando se utiliza un paquete comprimido, independientemente de si la versión anterior utiliza la compresión o no.

El paquete se empaqueta correctamente, validado y comprimido (si es necesario), por lo que está listo para la [implementación](service-fabric-deploy-remove-applications.md) en uno o varios clústeres de Service Fabric.

### <a name="compress-packages-when-deploying-using-visual-studio"></a>Compresión de paquetes al implementar con Visual Studio

Puede indicar a Visual Studio que comprima paquetes en la implementación, agregando el elemento `CopyPackageParameters` al perfil de publicación, y establezca el atributo `CompressPackage` en `true`.

``` xml
    <PublishProfile xmlns="http://schemas.microsoft.com/2015/05/fabrictools">
        <ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com" />
        <ApplicationParameterFile Path="..\ApplicationParameters\Cloud.xml" />
        <CopyPackageParameters CompressPackage="true"/>
    </PublishProfile>
```

## <a name="create-an-sfpkg"></a>Crear una archivo sfpkg

A partir de la versión 6.1, Service Fabric permite el aprovisionamiento desde un almacén externo.
Con esta opción, el paquete de aplicación no tiene que copiarse al almacén de imágenes. En su lugar, puede crear un archivo `sfpkg` y cargarlo a un almacén externo. A continuación, debe proporcionar el URI de descarga a Service Fabric durante el aprovisionamiento. El mismo paquete puede aprovisionarse a varios clústeres. El aprovisionamiento desde el almacén externo ahorra el tiempo que se necesita para copiar el paquete a cada clúster.

El archivo `sfpkg` es un archivo ZIP que contiene el paquete de aplicación inicial y tiene la extensión ".sfpkg".
En el archivo ZIP, el paquete de aplicación puede estar comprimido o no. La compresión del paquete de aplicación en el archivo ZIP se realiza en el nivel de paquete de código, configuración y datos, como [se ha mencionado anteriormente](service-fabric-package-apps.md#compress-a-package).

Para crear un archivo `sfpkg`, comience con una carpeta que contenga el paquete de aplicación original, comprimido o no. A continuación, utilice cualquier utilidad que comprima la carpeta con la extensión ".sfpkg". Por ejemplo, utilice [ZipFile.CreateFromDirectory](/dotnet/api/system.io.compression.zipfile.createfromdirectory?view=netcore-3.1#System_IO_Compression_ZipFile_CreateFromDirectory_System_String_System_String_System_IO_Compression_CompressionLevel_System_Boolean_).

```csharp
ZipFile.CreateFromDirectory(appPackageDirectoryPath, sfpkgFilePath);
```

El archivo `sfpkg` debe cargarse al almacén externo fuera de banda, fuera de Service Fabric. El almacén externo puede ser cualquier almacén que exponga un punto de conexión http o https de REST. Durante el aprovisionamiento, Service Fabric ejecuta una operación GET para descargar el paquete de aplicación `sfpkg`, por lo que el almacén debe permitir el acceso de lectura para el paquete.

Para aprovisionar el paquete, utilice el aprovisionamiento externo, lo que requiere el URI de descarga y la información del tipo de aplicación.

>[!NOTE]
> El aprovisionamiento basado en la ruta relativa al almacén de imágenes no es compatible con archivos `sfpkg` en estos momentos. Por lo tanto, el archivo `sfpkg` no debe copiarse al almacén de imágenes.

## <a name="next-steps"></a>Pasos siguientes

[Implementación y eliminación de aplicaciones][10] describe cómo usar PowerShell para administrar las instancias de aplicaciones.

[Administración de los parámetros de la aplicación en varios entornos][11] describe cómo configurar parámetros y variables de entorno para diferentes instancias de aplicación.

[Configuración de directivas de seguridad para la aplicación][12] describe cómo ejecutar los servicios a través de directivas de seguridad para restringir el acceso.

<!--Image references-->
[vs-package-command]: ./media/service-fabric-package-apps/vs-package-command.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-manage-multiple-environment-app-configuration.md
[12]: service-fabric-application-runas-security.md
