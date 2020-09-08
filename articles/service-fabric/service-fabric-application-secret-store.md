---
title: Almacén central de secretos de Azure Service Fabric
description: En este artículo se indica cómo usar el almacén central de secretos en Azure Service Fabric.
ms.topic: conceptual
ms.date: 07/25/2019
ms.openlocfilehash: e9fd435803ad5354b0eb2d4f5de50009a8cbbfe2
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869762"
---
# <a name="central-secrets-store-in-azure-service-fabric"></a>Almacén central de secretos en Azure Service Fabric 
En este artículo se describe cómo usar el Almacén central de secretos (CSS) en Azure Service Fabric para crear secretos en aplicaciones de Service Fabric. CSS es una caché local de almacén de secretos que conserva datos confidenciales, como contraseñas, tokens y claves, cifrados en memoria.

  > [!NOTE] 
  > Cuando se activa CSS por primera vez antes de la versión 7.1.CU3 de SF se puede producir un error en la activación que deja CSS en un estado incorrecto de forma permanente si: CSS está activado en un clúster autenticado de Windows o si está activado en cualquier clúster pero `EncryptionCertificateThumbprint` se declara incorrectamente o si el certificado correspondiente no está instalado o no incluye ACL en los nodos. En el caso del clúster de autenticación de Windows, vaya a 7.1.CU3 antes de continuar. En el caso de otros clústeres, compruebe estas invariantes o vaya a 7.1. CU3.
  
## <a name="enable-central-secrets-store"></a>Habilitación del Almacén central de secretos
Agregue el script siguiente a la configuración del clúster en `fabricSettings` para habilitar CSS. Para CSS se recomienda usar un certificado que no sea un certificado de clúster. Asegúrese de que el certificado de cifrado esté instalado en todos los nodos y de que `NetworkService` tenga permiso de lectura en la clave privada del certificado.
  ```json
    "fabricSettings": 
    [
        ...
    {
        "name":  "CentralSecretService",
        "parameters":  [
                {
                    "name":  "IsEnabled",
                    "value":  "true"
                },
                {
                    "name":  "MinReplicaSetSize",
                    "value":  "1"
                },
                {
                    "name":  "TargetReplicaSetSize",
                    "value":  "3"
                },
                 {
                    "name" : "EncryptionCertificateThumbprint",
                    "value": "<thumbprint>"
                 }
                ,
                ],
            },
            ]
     }
        ...
     ]
```
## <a name="declare-a-secret-resource"></a>Declaración de un recurso como secreto
Puede crear un recurso secreto mediante la API REST.
  > [!NOTE] 
  > Si el clúster usa la autenticación de Windows, la solicitud REST se envía a través de un canal HTTP no seguro. La recomendación es usar un clúster basado en X509 con puntos de conexión seguros.

Para crear un recurso secreto `supersecret` mediante la API REST, realice una solicitud PUT a `https://<clusterfqdn>:19080/Resources/Secrets/supersecret?api-version=6.4-preview`. Necesita el certificado del clúster o el certificado de cliente de administrador para crear un recurso secreto.

```powershell
$json = '{"properties": {"kind": "inlinedValue", "contentType": "text/plain", "description": "supersecret"}}'
Invoke-WebRequest  -Uri https://<clusterfqdn>:19080/Resources/Secrets/supersecret?api-version=6.4-preview -Method PUT -CertificateThumbprint <CertThumbprint> -Body $json
```

## <a name="set-the-secret-value"></a>Establecimiento del valor secreto

Con este script puede usar la API REST para establecer el valor secreto.
```powershell
$Params = '{"properties": {"value": "mysecretpassword"}}'
Invoke-WebRequest -Uri https://<clusterfqdn>:19080/Resources/Secrets/supersecret/values/ver1?api-version=6.4-preview -Method PUT -Body $Params -CertificateThumbprint <ClusterCertThumbprint>
```
### <a name="examine-the-secret-value"></a>Examen del valor del secreto
```powershell
Invoke-WebRequest -CertificateThumbprint <ClusterCertThumbprint> -Method POST -Uri "https:<clusterfqdn>/Resources/Secrets/supersecret/values/ver1/list_value?api-version=6.4-preview"
```
## <a name="use-the-secret-in-your-application"></a>Uso del secreto en la aplicación

Siga estos pasos para usar el secreto en la aplicación Service Fabric.

1. Agregue una sección al archivo **settings.xml** con el fragmento de código siguiente. Tenga en cuenta que aquí el valor tiene el formato {`secretname:version`}.

   ```xml
     <Section Name="testsecrets">
      <Parameter Name="TopSecret" Type="SecretsStoreRef" Value="supersecret:ver1"/
     </Section>
   ```

1. Importe la sección en **ApplicationManifest.xml**.
   ```xml
     <ServiceManifestImport>
       <ServiceManifestRef ServiceManifestName="testservicePkg" ServiceManifestVersion="1.0.0" />
       <ConfigOverrides />
       <Policies>
         <ConfigPackagePolicies CodePackageRef="Code">
           <ConfigPackage Name="Config" SectionName="testsecrets" EnvironmentVariableName="SecretPath" />
           </ConfigPackagePolicies>
       </Policies>
     </ServiceManifestImport>
   ```

   La variable de entorno `SecretPath` apuntará al directorio donde se almacenan todos los secretos. Cada parámetro incluido en la sección `testsecrets` se almacena en un archivo independiente. Ahora la aplicación puede usar el secreto como se muestra aquí:
   ```C#
   secretValue = IO.ReadFile(Path.Join(Environment.GetEnvironmentVariable("SecretPath"),  "TopSecret"))
   ```
1. Monte los secretos en un contenedor. El único cambio necesario para que los secretos pasen a estar disponibles dentro del contenedor consiste en aplicar `specify` a un punto de montaje en `<ConfigPackage>`.
El fragmento de código siguiente es el archivo **ApplicationManifest.xml** modificado.  

   ```xml
   <ServiceManifestImport>
       <ServiceManifestRef ServiceManifestName="testservicePkg" ServiceManifestVersion="1.0.0" />
       <ConfigOverrides />
       <Policies>
         <ConfigPackagePolicies CodePackageRef="Code">
           <ConfigPackage Name="Config" SectionName="testsecrets" MountPoint="C:\secrets" EnvironmentVariableName="SecretPath" />
           <!-- Linux Container
            <ConfigPackage Name="Config" SectionName="testsecrets" MountPoint="/mnt/secrets" EnvironmentVariableName="SecretPath" />
           -->
         </ConfigPackagePolicies>
       </Policies>
     </ServiceManifestImport>
   ```
   Los secretos están disponibles en el punto de montaje dentro del contenedor.

1. Puede enlazar un secreto a una variable de entorno de proceso si especifica `Type='SecretsStoreRef`. El fragmento siguiente es un ejemplo de cómo enlazar la versión `ver1` de `supersecret` a la variable de entorno `MySuperSecret` de **ServiceManifest.xml**.

   ```xml
   <EnvironmentVariables>
     <EnvironmentVariable Name="MySuperSecret" Type="SecretsStoreRef" Value="supersecret:ver1"/>
   </EnvironmentVariables>
   ```

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [seguridad de aplicaciones y servicios](service-fabric-application-and-service-security.md).
