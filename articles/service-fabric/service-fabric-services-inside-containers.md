---
title: Inclusión de los servicios de Azure Service Fabric en contenedores en Windows
description: Aprenda a incluir los servicios de Reliable Services y Reliable Actors de Service Fabric en contenedores en Windows.
ms.topic: conceptual
ms.date: 5/23/2018
ms.author: anmola
ms.openlocfilehash: 186afb87b4668ad575208beeeb19643e6a11cedd
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87824552"
---
# <a name="containerize-your-service-fabric-reliable-services-and-reliable-actors-on-windows"></a>Inclusión de Reliable Services y Reliable Actors de Service Fabric en contenedores en Windows

Service Fabric admite la inclusión de microservicios de Service Fabric en contenedores (servicios basados en Reliable Services y Reliable Actors). Para más información, consulte [Service Fabric y los contenedores](service-fabric-containers-overview.md).

Este documento ofrece una guía para que el servicio se ejecute dentro de un contenedor de Windows.

> [!NOTE]
> Actualmente, esta característica solo funciona para Windows. Para ejecutar contenedores, el clúster debe ejecutarse en Windows Server 2016 con Containers.

## <a name="steps-to-containerize-your-service-fabric-application"></a>Pasos para incluir la aplicación de Service Fabric en contenedores

1. Abra la aplicación de Service Fabric en Visual Studio.

2. Agregue la clase [SFBinaryLoader.cs](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/code/SFBinaryLoaderForContainers/SFBinaryLoader.cs) al proyecto. El código de esta clase es un asistente para cargar correctamente los archivos binarios del entorno en tiempo de ejecución de Service Fabric dentro de la aplicación cuando se ejecuta dentro de un contenedor.

3. Para cada paquete de código que le gustaría incluir en contenedores, inicialice el cargador en el punto de entrada del programa. Agregue el constructor estático que se muestra en el siguiente fragmento de código al archivo de punto de entrada del programa.

   ```csharp
   namespace MyApplication
   {
      internal static class Program
      {
          static Program()
          {
              SFBinaryLoader.Initialize();
          }

          /// <summary>
          /// This is the entry point of the service host process.
          /// </summary>
          private static void Main()
          {
   ```

4. Compile y [empaquete](service-fabric-package-apps.md#Package-App) el proyecto. Para compilar y crear un paquete, haga clic con el botón derecho en el proyecto de aplicación en el Explorador de soluciones y elija el comando **Empaquetar**.

5. Para cada paquete de código que necesite incluir en contenedores, ejecute el script de PowerShell [CreateDockerPackage.ps1](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/scripts/CodePackageToDockerPackage/CreateDockerPackage.ps1). El uso es como sigue:

    .NET completo
      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $applicationExeName = 'Name of the Code package executable.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -ApplicationExeName $applicationExeName
      ```
    .NET Core
      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $dotnetCoreDllName = 'Name of the Code package dotnet Core Dll.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -DotnetCoreDllName $dotnetCoreDllName
      ```
      El script crea una carpeta con los artefactos de Docker en $dockerPackageOutputDirectoryPath. Modifique el Dockerfile generado para `expose` los puertos, ejecutar scripts de configuración, etc. según sus necesidades.

6. A continuación, necesita [compilar](service-fabric-get-started-containers.md#Build-Containers) e [insertar](service-fabric-get-started-containers.md#Push-Containers) el paquete de contenedor de Docker en el repositorio.

7. Modifique los archivos ApplicationManifest.xml y ServiceManifest.xml para agregar la imagen del contenedor, la información del repositorio, la autenticación del registro y la asignación de puerto al host. Para modificar los manifiestos, consulte [Cree la primera aplicación contenedora en Service Fabric en Windows](service-fabric-get-started-containers.md). La definición de paquete de código en el manifiesto de servicio debe reemplazarse por la imagen de contenedor correspondiente. Asegúrese de cambiar EntryPoint a un tipo ContainerHost.

   ```xml
   <!-- Code package is your service executable. -->
   <CodePackage Name="Code" Version="1.0.0">
   <EntryPoint>
    <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
    <ContainerHost>
      <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
    </ContainerHost>
   </EntryPoint>
   <!-- Pass environment variables to your container: -->
   </CodePackage>
   ```

8. Agregue la asignación de host a puerto para el replicador y el punto de conexión de servicio. Puesto que Service Fabric asigna ambos puertos en tiempo de ejecución, ContainerPort se establece en cero para usar el puerto asignado para la asignación.

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="0" EndpointRef="ServiceEndpoint"/>
    <PortBinding ContainerPort="0" EndpointRef="ReplicatorEndpoint"/>
   </ContainerHostPolicies>
   </Policies>
   ```

9. Para configurar el modo de aislamiento de contenedor, consulte [Configuración del modo de aislamiento]( ./service-fabric-get-started-containers.md#configure-isolation-mode). Windows admite dos modos de aislamiento para contenedores: de proceso y de Hyper-V. Los siguientes fragmentos de código muestran cómo el modo de aislamiento se especifica en el archivo de manifiesto de aplicación.

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
   ...
   </ContainerHostPolicies>
   </Policies>
   ```
   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
   ...
   </ContainerHostPolicies>
   </Policies>
   ```

> [!NOTE] 
> De manera predeterminada, las aplicaciones de Service Fabric tienen acceso al entorno de ejecución de Service Fabric en forma de punto de conexión que acepta solicitudes específicas de la aplicación. Considere la posibilidad de deshabilitar este acceso cuando la aplicación hospede código que no sea de confianza. Para obtener más información, consulte los [Procedimientos recomendados de seguridad de Service Fabric](service-fabric-best-practices-security.md#platform-isolation). Para deshabilitar el acceso al entorno de ejecución de Service Fabric, agregue la siguiente configuración en la sección Directivas del manifiesto de la aplicación correspondiente al manifiesto del servicio importado, como se muestra a continuación:
>
```xml
  <Policies>
      <ServiceFabricRuntimeAccessPolicy RemoveServiceFabricRuntimeAccess="true"/>
  </Policies>
```
>

10. Para probar esta aplicación, debe implementarla en un clúster que esté ejecutando la versión 5.7 o posterior. Para las versiones 6.1 o posteriores del entorno de ejecución, debe editar y actualizar la configuración del clúster para habilitar esta característica en vista previa. Siga los pasos de este [artículo](service-fabric-cluster-fabric-settings.md) para agregar la configuración que se muestra a continuación.
    ```
      {
        "name": "Hosting",
        "parameters": [
          {
            "name": "FabricContainerAppsEnabled",
            "value": "true"
          }
        ]
      }
    ```

11. Después, [implemente](service-fabric-deploy-remove-applications.md) el paquete de aplicación editado en este clúster.

Ahora debería tener una aplicación de Service Fabric en contenedores ejecutando el clúster.

## <a name="next-steps"></a>Pasos siguientes
* Más información acerca de cómo ejecutar [contenedores en Service Fabric](service-fabric-get-started-containers.md).
* Más información acerca del [ciclo de vida de aplicaciones](service-fabric-application-lifecycle.md) de Service Fabric.