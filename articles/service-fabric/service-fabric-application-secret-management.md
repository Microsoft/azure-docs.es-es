---
title: Administración de secretos en aplicaciones de Azure Service Fabric
description: Aprenda a proteger los valores de secreto en una aplicación de Service Fabric (independiente de plataforma).
ms.topic: conceptual
ms.date: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: a11869c3b606ed9e74ce4f598109139fa1bb4164
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012830"
---
# <a name="manage-encrypted-secrets-in-service-fabric-applications"></a>Administración de secretos cifrados en aplicaciones de Service Fabric
Esta guía le lleva por los pasos para administrar secretos en una aplicación de Service Fabric. Los secretos pueden ser cualquier información confidencial, como cadenas de conexión de almacenamiento, contraseñas u otros valores que no se deben administrar en texto sin formato.

Para usar los secretos cifrados en una aplicación de Service Fabric hay que seguir tres pasos:
* Configurar un certificado de cifrado y cifrar los secretos
* Especificar secretos cifrados en una aplicación
* Descifrar secretos desde el código del servicio

## <a name="set-up-an-encryption-certificate-and-encrypt-secrets"></a>Configuración de un certificado de cifrado y cifrado de secretos
La configuración de un certificado de cifrado y su uso para cifrar los secretos varían entre Windows y Linux.
* [Configuración de un certificado de cifrado y cifrado de secretos en clústeres Windows.][secret-management-windows-specific-link]
* [Configuración de un certificado de cifrado y cifrado de secretos en Linux.][secret-management-linux-specific-link]

## <a name="specify-encrypted-secrets-in-an-application"></a>Especificación de los secretos cifrados en una aplicación
En el paso anterior se describe cómo cifrar un secreto con un certificado y generar una cadena codificada en base 64 para usarla en una aplicación. Esta cadena codificada en base 64 se puede especificar como un [parámetro][parameters-link] cifrado en un archivo Settings.xml del servicio o como una [variable de entorno][environment-variables-link] cifrada en un archivo ServiceManifest.xml del servicio.

Especifique un [parámetro][parameters-link] cifrado en el archivo de configuración Settings.xml del servicio con el atributo `IsEncrypted` establecido en `true`:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MySettings">
    <Parameter Name="MySecret" IsEncrypted="true" Value="I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=" />
  </Section>
</Settings>
```
Especifique una [variable de entorno][environment-variables-link] cifrada en el archivo ServiceManifest.xml del servicio con el atributo `Type` establecido en `Encrypted`:
```xml
<CodePackage Name="Code" Version="1.0.0">
  <EnvironmentVariables>
    <EnvironmentVariable Name="MyEnvVariable" Type="Encrypted" Value="I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=" />
  </EnvironmentVariables>
</CodePackage>
```

Los secretos también se deben incluir en la aplicación Service Fabric mediante la especificación de un certificado en el manifiesto de aplicación. Agregue un elemento **SecretsCertificate** a **ApplicationManifest.xml** e incluya la huella digital del certificado que desee.

```xml
<ApplicationManifest … >
  ...
  <Certificates>
    <SecretsCertificate Name="MyCert" X509FindType="FindByThumbprint" X509FindValue="[YourCertThumbrint]"/>
  </Certificates>
</ApplicationManifest>
```
> [!NOTE]
> Al activar una aplicación que especifica un elemento SecretsCertificate, Service Fabric encontrará el certificado coincidente y concederá a la identidad, bajo la que se ejecuta la aplicación, permisos completos para la clave privada del certificado. Service Fabric también supervisará los cambios del certificado y volverá a aplicar los permisos en consecuencia. Para detectar los cambios en los certificados declarados por nombre común, Service Fabric ejecuta una tarea periódica que encuentra todos los certificados coincidentes y los compara con una lista en caché de huellas digitales. Cuando se detecta una nueva huella digital, significa que ese sujeto ha renovado un certificado. La tarea se ejecuta una vez por minuto en cada nodo del clúster.
>
> Aunque el elemento SecretsCertificate permite declaraciones basadas en sujeto, tenga en cuenta que la configuración cifrada está asociada al par de claves que se usó para cifrar la configuración en el cliente. Debe asegurarse de que el certificado de cifrado original (o equivalente) coincide con la declaración basada en sujeto y que está instalado, incluida su clave privada correspondiente, en todos los nodos del clúster que pueden hospedar la aplicación. Todos los certificados válidos en el tiempo que coinciden con la declaración basada en sujeto y creados a partir del mismo par de claves que el certificado de cifrado original se consideran equivalentes.
>

### <a name="inject-application-secrets-into-application-instances"></a>Inserción de secretos de aplicación en instancias de aplicación
Lo más conveniente es que la implementación en entornos diferentes sea lo más automatizada posible. Para lograr esto, se puede realizar el cifrado del secreto en un entorno de compilación y proporcionar los secretos cifrados como parámetros al crear instancias de aplicación.

#### <a name="use-overridable-parameters-in-settingsxml"></a>Uso de parámetros reemplazables en Settings.xml
El archivo de configuración Settings.xml permite parámetros reemplazables que se pueden proporcionar en el momento de creación de una aplicación. En lugar de proporcionar un valor para el parámetro, use el atributo `MustOverride` :

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MySettings">
    <Parameter Name="MySecret" IsEncrypted="true" Value="" MustOverride="true" />
  </Section>
</Settings>
```

Para reemplazar los valores en el archivo Settings.xml, declare un parámetro de reemplazo para el servicio en ApplicationManifest.xml:

```xml
<ApplicationManifest ... >
  <Parameters>
    <Parameter Name="MySecret" DefaultValue="" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides>
      <ConfigOverride Name="Config">
        <Settings>
          <Section Name="MySettings">
            <Parameter Name="MySecret" Value="[MySecret]" IsEncrypted="true" />
          </Section>
        </Settings>
      </ConfigOverride>
    </ConfigOverrides>
  </ServiceManifestImport>
 ```

Ahora, el valor se puede especificar como un *parámetro de aplicación* al crear una instancia de la aplicación. La creación de una instancia de aplicación se puede generar mediante un script con PowerShell, o escribirse en C#, para facilitar la integración en un proceso de compilación.

Mediante PowerShell, el parámetro se proporciona al comando `New-ServiceFabricApplication` como una [tabla hash](/previous-versions/windows/it-pro/windows-powershell-1.0/ee692803(v=technet.10)):

```powershell
New-ServiceFabricApplication -ApplicationName fabric:/MyApp -ApplicationTypeName MyAppType -ApplicationTypeVersion 1.0.0 -ApplicationParameter @{"MySecret" = "I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM="}
```

Con C#, los parámetros de aplicación se especifican en `ApplicationDescription` como `NameValueCollection`:

```csharp
FabricClient fabricClient = new FabricClient();

NameValueCollection applicationParameters = new NameValueCollection();
applicationParameters["MySecret"] = "I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=";

ApplicationDescription applicationDescription = new ApplicationDescription(
    applicationName: new Uri("fabric:/MyApp"),
    applicationTypeName: "MyAppType",
    applicationTypeVersion: "1.0.0",
    applicationParameters: applicationParameters)
);

await fabricClient.ApplicationManager.CreateApplicationAsync(applicationDescription);
```

## <a name="decrypt-encrypted-secrets-from-service-code"></a>Descifrado de secretos desde el código del servicio
Las API para acceder a los [parámetros][parameters-link] y a las [variables de entorno][environment-variables-link] permiten cifrar fácilmente valores descifrados. Dado que la cadena cifrada contiene información sobre el certificado usado para el cifrado, no es necesario especificar manualmente el certificado. Simplemente se debe instalar en el nodo en el que se ejecuta el servicio.

```csharp
// Access decrypted parameters from Settings.xml
ConfigurationPackage configPackage = FabricRuntime.GetActivationContext().GetConfigurationPackageObject("Config");
bool MySecretIsEncrypted = configPackage.Settings.Sections["MySettings"].Parameters["MySecret"].IsEncrypted;
if (MySecretIsEncrypted)
{
    SecureString MySecretDecryptedValue = configPackage.Settings.Sections["MySettings"].Parameters["MySecret"].DecryptValue();
}

// Access decrypted environment variables from ServiceManifest.xml
// Note: you do not have to call any explicit API to decrypt the environment variable.
string MyEnvVariable = Environment.GetEnvironmentVariable("MyEnvVariable");
```

## <a name="next-steps"></a>Pasos siguientes
* [Almacén de secretos](service-fabric-application-secret-store.md) de Service Fabric 
* Más información sobre la [seguridad de aplicaciones y servicios](service-fabric-application-and-service-security.md)

<!-- Links -->
[parameters-link]:service-fabric-how-to-parameterize-configuration-files.md
[environment-variables-link]: service-fabric-how-to-specify-environment-variables.md
[secret-management-windows-specific-link]: service-fabric-application-secret-management-windows.md
[secret-management-linux-specific-link]: service-fabric-application-secret-management-linux.md
[service fabric secrets store]: service-fabric-application-secret-store.md
