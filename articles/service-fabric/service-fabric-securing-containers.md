---
title: Importación de certificados en un contenedor
description: Aprenda ahora a importar los archivos de certificados a un servicio de contenedor de Service Fabric.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 426aa2ebbfb87fe2c80e0d1aff3eeecbe0e2472d
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050750"
---
# <a name="import-a-certificate-file-into-a-container-running-on-service-fabric"></a>Importación de un archivo de certificados a un contenedor que se ejecuta en Service Fabric

> [!NOTE]
> En el caso de los clústeres de Service Fabric que se ejecutan en Azure, se recomienda usar la [Identidad administrada de la aplicación de Service Fabric](https://docs.microsoft.com/azure/service-fabric/concepts-managed-identity) para aprovisionar certificados de aplicación desde dentro de un contenedor. La identidad administrada proporciona aislamiento de secretos y certificados en el nivel de servicio, y permite que el aprovisionamiento de certificados de la aplicación forme parte del flujo de trabajo de la aplicación, en lugar de ser parte del flujo de trabajo de la infraestructura. El mecanismo CertificateRef quedará en desuso en una versión futura.

Puede proteger los servicios de contenedor especificando un certificado. Service Fabric proporciona un mecanismo para los servicios dentro de un contenedor para acceder a un certificado que está instalado en los nodos de un clúster de Windows o Linux (versión 5.7 o superior). Este certificado debe instalarse en un almacén de certificados en LocalMachine en todos los nodos del clúster. La clave privada correspondiente al certificado debe estar disponible, ser accesible y, en Windows, exportable. La información del certificado se proporciona en el manifiesto de aplicación en la etiqueta `ContainerHostPolicies` como se muestra en el siguiente fragmento de código:

```xml
  <ContainerHostPolicies CodePackageRef="NodeContainerService.Code">
    <CertificateRef Name="MyCert1" X509StoreName="My" X509FindValue="[Thumbprint1]"/>
    <CertificateRef Name="MyCert2" X509FindValue="[Thumbprint2]"/>
 ```

Para los clústeres de Windows, al iniciar la aplicación, el tiempo de ejecución exporta cada certificado al que se hace referencia y su correspondiente clave privada en un archivo PFX protegido con una contraseña generada de forma aleatoria. Se puede acceder a los archivos de contraseña y de PFX, respectivamente, desde el contenedor mediante las siguientes variables de entorno: 

* Certificates_ServicePackageName_CodePackageName_CertName_PFX
* Certificates_ServicePackageName_CodePackageName_CertName_Password

Para clústeres Linux, los certificados (PEM) se copian del almacén que especifica X509StoreName al contenedor. Las variables de entorno correspondientes en Linux son:

* Certificates_ServicePackageName_CodePackageName_CertName_PEM
* Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey

Tenga en cuenta que tanto el archivo `PEM` como el archivo `PrivateKey` contienen el certificado y la clave privada sin cifrar.

Como alternativa, si ya tiene los certificados con el formato necesario y quiere poder acceder desde dentro del contenedor, puede crear un paquete de datos dentro del paquete de aplicación y especificar lo siguiente en el manifiesto de aplicación:

```xml
<ContainerHostPolicies CodePackageRef="NodeContainerService.Code">
  <CertificateRef Name="MyCert1" DataPackageRef="[DataPackageName]" DataPackageVersion="[Version]" RelativePath="[Relative Path to certificate inside DataPackage]" Password="[password]" IsPasswordEncrypted="[true/false]"/>
 ```

El proceso o el servicio de contenedor es el responsable de importar archivos de certificado en el contenedor. Para importar el certificado, puede usar scripts de `setupentrypoint.sh` o ejecutar código personalizado dentro del proceso de contenedor. A continuación se muestra código de ejemplo en C# para importar el archivo PFX:

```csharp
string certificateFilePath = Environment.GetEnvironmentVariable("Certificates_MyServicePackage_NodeContainerService.Code_MyCert1_PFX");
string passwordFilePath = Environment.GetEnvironmentVariable("Certificates_MyServicePackage_NodeContainerService.Code_MyCert1_Password");
X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
string password = File.ReadAllLines(passwordFilePath, Encoding.Default)[0];
password = password.Replace("\0", string.Empty);
X509Certificate2 cert = new X509Certificate2(certificateFilePath, password, X509KeyStorageFlags.MachineKeySet | X509KeyStorageFlags.PersistKeySet);
store.Open(OpenFlags.ReadWrite);
store.Add(cert);
store.Close();
```
Este certificado PFX puede usarse para autenticar la aplicación o el servicio, o para proteger la comunicación con otros servicios. De forma predeterminada, los archivos se agregan a listas de control de acceso solo en SYSTEM. Puede agregarlos a listas de control de acceso en otras cuentas, según lo requiera el servicio.

A continuación, lea los siguientes artículos:

* [Implementación de un contenedor de Windows en Service fabric en Windows Server 2016](service-fabric-get-started-containers.md)
* [Implementación de un contenedor de Docker en Service Fabric en Linux](service-fabric-get-started-containers-linux.md)
