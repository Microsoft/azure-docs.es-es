---
title: Adición de un punto de conexión HTTPS mediante Kestrel
description: En este tutorial, aprenderá a agregar un punto de conexión HTTPS a un servicio web de front-end de ASP.NET Core que usa Kestrel y a implementar la aplicación en un clúster.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: b309a13288c8ea95f453c1e80549a979e3f89921
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441534"
---
# <a name="tutorial-add-an-https-endpoint-to-an-aspnet-core-web-api-front-end-service-using-kestrel"></a>Tutorial: Incorporación de un punto de conexión HTTPS a un servicio de front-end de API Web de ASP.NET Core mediante Kestrel

Este tutorial es la tercera parte de una serie.  Aprenderá a habilitar HTTPS en un servicio de ASP.NET Core que se ejecuta en Service Fabric. Una vez que haya terminado, tendrá una aplicación de votación con un front-end web de ASP.NET Core habilitado para HTTPS que escucha en el puerto 443. Si no desea crear manualmente la aplicación de votación descrita en [Crear una aplicación de .NET Service Fabric](service-fabric-tutorial-deploy-app-to-party-cluster.md), puede [descargar el código fuente](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/) de la aplicación terminada.

En la tercera parte de la serie, se aprende a:

> [!div class="checklist"]
> * Definir un punto de conexión HTTPS en el servicio
> * Configurar Kestrel para que use HTTPS
> * Instalar el certificado TLS/SSL en los nodos del clúster remoto
> * Conceder a NETWORK SERVICE acceso a la clave privada del certificado
> * Abrir el puerto 443 en Azure Load Balancer
> * Implementar la aplicación en un clúster remoto

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * [Crear una aplicación de .NET Service Fabric](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [Implementar la aplicación en un clúster remoto](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * Agregar un punto de conexión HTTPS a un servicio de front-end de ASP.NET Core
> * [Configuración de CI/CD con Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Configurar la supervisión y el diagnóstico para la aplicación](service-fabric-tutorial-monitoring-aspnet.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar este tutorial:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Instale Visual Studio 2019](https://www.visualstudio.com/) versión 16.5 o posterior con las cargas de trabajo **Desarrollo de Azure** y **Desarrollo de ASP.NET y web**.
* [Instale el SDK de Service Fabric](service-fabric-get-started.md)

## <a name="obtain-a-certificate-or-create-a-self-signed-development-certificate"></a>Obtención de un certificado o creación de un certificado de desarrollo autofirmado

Para aplicaciones de producción, use un certificado de una [entidad de certificación (CA)](https://wikipedia.org/wiki/Certificate_authority). Con fines de desarrollo y pruebas, puede crear y usar un certificado autofirmado. El SDK de Service Fabric proporciona el script *CertSetup.ps1*, que crea un certificado autofirmado y lo importa en el almacén de certificados `Cert:\LocalMachine\My`. Abra un símbolo del sistema como administrador y ejecute el siguiente comando para crear un certificado con el asunto "CN=mytestcert":

```powershell
PS C:\program files\microsoft sdks\service fabric\clustersetup\secure> .\CertSetup.ps1 -Install -CertSubjectName CN=mytestcert
```

Si ya tiene un archivo PFX de certificado, ejecute lo siguiente para importar el certificado en el almacén de certificados `Cert:\LocalMachine\My`:

```powershell

PS C:\mycertificates> Import-PfxCertificate -FilePath .\mysslcertificate.pfx -CertStoreLocation Cert:\LocalMachine\My -Password (ConvertTo-SecureString "!Passw0rd321" -AsPlainText -Force)


   PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\My

Thumbprint                                Subject
----------                                -------
3B138D84C077C292579BA35E4410634E164075CD  CN=zwin7fh14scd.westus.cloudapp.azure.com
```

## <a name="define-an-https-endpoint-in-the-service-manifest"></a>Definición de un punto de conexión HTTPS en el manifiesto de servicio

Inicie Visual Studio como **administrador** y abra la solución de votación. En el Explorador de soluciones, abra *VotingWeb/PackageRoot/ServiceManifest.xml*. El manifiesto del servicio define los puntos de conexión del servicio.  Busque la sección **Puntos de conexión** y edite el punto de conexión "ServiceEndpoint" existente.  Cambie el nombre a "EndpointHttps", establezca el protocolo en *https*, el tipo en *Entrada*y el puerto en *443*.  Guarde los cambios.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="configure-kestrel-to-use-https"></a>Configurar Kestrel para que use HTTPS

En el Explorador de soluciones, abra el archivo *VotingWeb/VotingWeb.cs*.  Configure Kestrel para que use HTTPS y busque el certificado en el almacén `Cert:\LocalMachine\My`. Agregue las siguientes instrucciones using:

```csharp
using System.Net;
using Microsoft.Extensions.Configuration;
using System.Security.Cryptography.X509Certificates;
```

Actualice `ServiceInstanceListener` para que use el nuevo punto de conexión *EndpointHttps* y escuche en el puerto 443. Al configurar el host de web para usar el servidor Kestrel, debe configurar Kestrel para escuchar las direcciones IPv6 en todas las interfaces de red: `opt.Listen(IPAddress.IPv6Any, port, listenOptions => {...}`.

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(FindMatchingCertificateBySubject());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

                .ConfigureServices(
                    services => services
                        .AddSingleton<HttpClient>(new HttpClient())
                        .AddSingleton<FabricClient>(new FabricClient())
                        .AddSingleton<StatelessServiceContext>(serviceContext))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                .UseUrls(url)
                .Build();
        }))
```

Agregue también el método siguiente para que Kestrel pueda encontrar el certificado en el almacén `Cert:\LocalMachine\My` mediante el asunto.  

Reemplace "&lt;your_CN_value&gt;" por "mytestcert" si ha creado un certificado autofirmado con el comando de PowerShell anterior o use el nombre CN del certificado.
Tenga en cuenta que, en el caso de la implementación local en `localhost`, es preferible usar "CN=localhost" para evitar las excepciones de autenticación.

```csharp
private X509Certificate2 FindMatchingCertificateBySubject(string subjectCommonName)
{
    using (var store = new X509Store(StoreName.My, StoreLocation.LocalMachine))
    {
        store.Open(OpenFlags.OpenExistingOnly | OpenFlags.ReadOnly);
        var certCollection = store.Certificates;
        var matchingCerts = new X509Certificate2Collection();
    
    foreach (var enumeratedCert in certCollection)
    {
      if (StringComparer.OrdinalIgnoreCase.Equals(subjectCommonName, enumeratedCert.GetNameInfo(X509NameType.SimpleName, forIssuer: false))
        && DateTime.Now < enumeratedCert.NotAfter
        && DateTime.Now >= enumeratedCert.NotBefore)
        {
          matchingCerts.Add(enumeratedCert);
        }
    }

        if (matchingCerts.Count == 0)
    {
        throw new Exception($"Could not find a match for a certificate with subject 'CN={subjectCommonName}'.");
    }
        
        return matchingCerts[0];
    }
}


```

## <a name="grant-network-service-access-to-the-certificates-private-key"></a>Conceder a NETWORK SERVICE acceso a la clave privada del certificado

En un paso anterior, se importa el certificado del almacén `Cert:\LocalMachine\My` en el equipo de desarrollo.  Ahora, proporcione explícitamente a la cuenta que ejecuta el servicio (NETWORK SERVICE de forma predeterminada) acceso a la clave privada del certificado. Puede hacerlo manualmente (mediante la herramienta certlm.msc), pero es mejor ejecutar automáticamente un script de PowerShell realizando la [configuración de un script de inicio](service-fabric-run-script-at-service-startup.md) en el punto **SetupEntryPoint** del manifiesto de servicio.

>[!NOTE]
> Service Fabric admite la declaración de certificados de punto de conexión por huella digital o nombre común del firmante. En ese caso, el entorno de ejecución configurará el enlace e incluirá en la ACL la clave privada del certificado para la identidad en la que se ejecuta el servicio. El entorno de ejecución también supervisará el certificado en busca de cambios y renovaciones y volverá a incluir en la ACL la clave privada correspondiente según corresponda.

### <a name="configure-the-service-setup-entry-point"></a>Configuración del punto de entrada de instalación del servicio

En el Explorador de soluciones, abra *VotingWeb/PackageRoot/ServiceManifest.xml*.  En la sección **CodePackage**, agregue el nodo **SetupEntryPoint** y, a continuación, un nodo **ExeHost**.  En **ExeHost**, establezca **Program** en "Setup.bat" y **WorkingFolder** en "CodePackage".  Cuando se inicia el servicio VotingWeb, el script Setup.bat se ejecuta en la carpeta CodePackage antes de que se inicie VotingWeb.exe.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>Setup.bat</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </SetupEntryPoint>

    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

### <a name="add-the-batch-and-powershell-setup-scripts"></a>Incorporación del lote y de los scripts de configuración de PowerShell

Para ejecutar PowerShell desde el punto **SetupEntryPoint**, puede ejecutar PowerShell.exe en un archivo por lotes que apunte al archivo de PowerShell. En primer lugar, agregue el archivo por lotes al proyecto de servicio.  En el Explorador de soluciones, haga clic con el botón derecho en **VotingWeb** y seleccione **Agregar**->**Nuevo elemento** y agregue un nuevo archivo denominado "Setup.bat".  Edite el archivo *Setup.bat* y agregue el siguiente comando:

```cmd
powershell.exe -ExecutionPolicy Bypass -Command ".\SetCertAccess.ps1"
```

Modifique las propiedades del archivo *Setup.bat* para establecer la opción **Copiar en el directorio de salida** en "Copiar si es posterior".

![Establecimiento de las propiedades del archivo][image1]

En el Explorador de soluciones, haga clic con el botón derecho en **VotingWeb** y seleccione **Agregar**->**Nuevo elemento** y agregue un nuevo archivo denominado "SetCertAccess.ps1".  Edite el archivo *SetCertAccess.ps1* y agregue el siguiente script:

```powershell
$subject="mytestcert"
$userGroup="NETWORK SERVICE"

Write-Host "Checking permissions to certificate $subject.." -ForegroundColor DarkCyan

$cert = (gci Cert:\LocalMachine\My\ | where { $_.Subject.Contains($subject) })[-1]

if ($cert -eq $null)
{
    $message="Certificate with subject:"+$subject+" does not exist at Cert:\LocalMachine\My\"
    Write-Host $message -ForegroundColor Red
    exit 1;
}elseif($cert.HasPrivateKey -eq $false){
    $message="Certificate with subject:"+$subject+" does not have a private key"
    Write-Host $message -ForegroundColor Red
    exit 1;
}else
{
    $keyName=$cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName

    $keyPath = "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\"
    $fullPath=$keyPath+$keyName
    $acl=(Get-Item $fullPath).GetAccessControl('Access')


    $hasPermissionsAlready = ($acl.Access | where {$_.IdentityReference.Value.Contains($userGroup.ToUpperInvariant()) -and $_.FileSystemRights -eq [System.Security.AccessControl.FileSystemRights]::FullControl}).Count -eq 1

    if ($hasPermissionsAlready){
        Write-Host "Account $userGroup already has permissions to certificate '$subject'." -ForegroundColor Green
        return $false;
    } else {
        Write-Host "Need add permissions to '$subject' certificate..." -ForegroundColor DarkYellow

        $permission=$userGroup,"Full","Allow"
        $accessRule=new-object System.Security.AccessControl.FileSystemAccessRule $permission
        $acl.AddAccessRule($accessRule)
        Set-Acl $fullPath $acl

        Write-Output "Permissions were added"

        return $true;
    }
}

```

Modifique las propiedades del archivo *Setup.bat* para establecer **Copiar en el directorio de salida** en "Copiar si es posterior".

### <a name="run-the-setup-script-as-a-local-administrator"></a>Ejecución del script de instalación como administrador local

De forma predeterminada, el ejecutable de punto de entrada del programa de instalación del servicio se ejecuta con las mismas credenciales que Service Fabric (normalmente la cuenta NetworkService). El archivo *SetCertAccess.ps1* requiere privilegios de administrador. En el manifiesto de aplicación, puede cambiar los permisos de seguridad para ejecutar el script de inicio con una cuenta de administrador local.

En el Explorador de soluciones, abra *Voting/ApplicationPackageRoot/ApplicationManifest.xml*. En primer lugar, cree una sección **Principals** y agregue un nuevo usuario (por ejemplo, "SetupAdminUser"). Agregue la cuenta de usuario SetupAdminUser al grupo de administradores del sistema.
A continuación, en la sección **ServiceManifestImport** de VotingWebPkg, configure una directiva **RunAsPolicy** para aplicar la entidad de seguridad SetupAdminUser al punto de entrada de la instalación. Esta directiva indica a Service Fabric que el archivo Setup.bat se debe ejecutar como SetupAdminUser (con privilegios de administrador).

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="run-the-application-locally"></a>Ejecución de la aplicación de forma local

En el Explorador de soluciones, seleccione la aplicación **Voting** y establezca la propiedad **Application URL** en "https:\//localhost:443".

Guarde todos los archivos y presione la tecla F5 para ejecutar la aplicación localmente.  Después de que se implementa la aplicación, se abre un explorador web en https:\//localhost:443. Si va a usar un certificado autofirmado, aparecerá una advertencia que le indica que el PC no confía en la seguridad de este sitio web.  Continúe en la página web.

![Aplicación de votación][image2]

## <a name="install-certificate-on-cluster-nodes"></a>Instalación del certificado en los nodos de clúster

Antes de implementar la aplicación en Azure, instale el certificado en el almacén `Cert:\LocalMachine\My` de todos los nodos de clúster remotos.  Los servicios se pueden mover a diferentes nodos del clúster.  Cuando se inicia el servicio web de front-end en un nodo de clúster, el script de inicio busca el certificado y configura los permisos de acceso.

Primero, exporte el certificado a un archivo PFX. Abra la aplicación certlm.msc y vaya a **Personal**>**Certificados**.  Haga clic con el botón derecho en el certificado *mytestcert* y seleccione **Todas las tareas**>**Exportar**.

![Exportación de certificado][image4]

En el asistente para exportación, elija **Exportar la clave privada** y elija el formato de intercambio de información personal (PFX).  Exporte el archivo a *C:\Users\sfuser\votingappcert.pfx*.

A continuación, instale el certificado en el clúster remoto mediante [estos scripts de PowerShell proporcionados](./scripts/service-fabric-powershell-add-application-certificate.md).

> [!Warning]
> Un certificado autofirmado es suficiente para desarrollar y probar aplicaciones. Para aplicaciones de producción, use un certificado de una [entidad de certificación (CA)](https://wikipedia.org/wiki/Certificate_authority) en vez de un certificado autofirmado.

## <a name="open-port-443-in-the-azure-load-balancer"></a>Abrir el puerto 443 en Azure Load Balancer

Abra el puerto 443 en el equilibrador de carga si aún no está abierto.

```powershell
$probename = "AppPortProbe6"
$rulename="AppPortLBRule6"
$RGname="voting_RG"
$port=443

# Get the load balancer resource
$resource = Get-AzResource | Where {$_.ResourceGroupName –eq $RGname -and $_.ResourceType -eq "Microsoft.Network/loadBalancers"}
$slb = Get-AzLoadBalancer -Name $resource.Name -ResourceGroupName $RGname

# Add a new probe configuration to the load balancer
$slb | Add-AzLoadBalancerProbeConfig -Name $probename -Protocol Tcp -Port $port -IntervalInSeconds 15 -ProbeCount 2

# Add rule configuration to the load balancer
$probe = Get-AzLoadBalancerProbeConfig -Name $probename -LoadBalancer $slb
$slb | Add-AzLoadBalancerRuleConfig -Name $rulename -BackendAddressPool $slb.BackendAddressPools[0] -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -Probe $probe -Protocol Tcp -FrontendPort $port -BackendPort $port

# Set the goal state for the load balancer
$slb | Set-AzLoadBalancer
```

## <a name="deploy-the-application-to-azure"></a>Implementación de la aplicación en Azure

Guarde todos los archivos, cambie de depurar a liberar y presione F6 para recompilar.  En el Explorador de soluciones, haga clic con el botón derecho en **Voting** y seleccione **Publicar**. Seleccione el punto de conexión del clúster creado en [Implementar una aplicación en un clúster.](service-fabric-tutorial-deploy-app-to-party-cluster.md), o seleccione otro clúster.  Haga clic en **Publicar** para publicar la aplicación en el clúster remoto.

Cuando se implemente la aplicación, abra un explorador web y vaya a `https://mycluster.region.cloudapp.azure.com:443` (actualice la dirección URL con el punto de conexión del clúster). Si va a usar un certificado autofirmado, aparecerá una advertencia que le indica que el PC no confía en la seguridad de este sitio web.  Continúe en la página web.

![Aplicación de votación][image3]

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:

> [!div class="checklist"]
> * Definir un punto de conexión HTTPS en el servicio
> * Configurar Kestrel para que use HTTPS
> * Instalar el certificado TLS/SSL en los nodos del clúster remoto
> * Conceder a NETWORK SERVICE acceso a la clave privada del certificado
> * Abrir el puerto 443 en Azure Load Balancer
> * Implementar la aplicación en un clúster remoto

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Configuración de CI/CD con Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)

[image1]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/SetupBatProperties.png
[image2]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppLocal.png
[image3]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppAzure.png
[image4]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/ExportCert.png
