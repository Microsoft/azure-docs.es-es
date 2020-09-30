---
title: Protección de un clúster en Windows mediante certificados
description: Se describe cómo proteger la comunicación en el clúster independiente o local de Azure Service Fabric, así como entre los clientes y el clúster.
author: dkkapur
ms.topic: conceptual
ms.date: 10/15/2017
ms.author: dekapur
ms.openlocfilehash: 477a8e75aef3eb676d17c045f16a5c3f4ecf1b81
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299707"
---
# <a name="secure-a-standalone-cluster-on-windows-by-using-x509-certificates"></a>Protección de un clúster independiente en Windows mediante certificados X.509
En este artículo se describe cómo proteger la comunicación entre los diversos nodos de un clúster de Windows independiente. También se describe cómo autenticar a los clientes que se conectan a este clúster mediante certificados X.509. Esta autenticación garantiza que solo los usuarios autorizados pueden tener acceso al clúster y a las aplicaciones implementadas, así como realizar tareas de administración. La seguridad basada en certificados se debe haber habilitado en el clúster al crearlo.  

Para obtener más información sobre la seguridad de clúster, como la seguridad de nodo a nodo, la seguridad de cliente a nodo y el control de acceso basado en rol, consulte [Escenarios de seguridad de los clústeres de Service Fabric](service-fabric-cluster-security.md).

## <a name="which-certificates-do-you-need"></a>¿Qué certificados necesita?
Para empezar, [descargue Service Fabric para el paquete de clúster independiente](service-fabric-cluster-creation-for-windows-server.md#download-the-service-fabric-for-windows-server-package) en uno de los nodos del clúster. En el paquete descargado, encontrará el archivo ClusterConfig.X509.MultiMachine.json. Abra el archivo y revise la sección relativa a la seguridad en la sección properties:

```JSON
"security": {
    "metadata": "The Credential type X509 indicates this cluster is secured by using X509 certificates. The thumbprint format is d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
    "ClusterCredentialType": "X509",
    "ServerCredentialType": "X509",
    "CertificateInformation": {
        "ClusterCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },        
        "ClusterCertificateCommonNames": {
            "CommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]"
            }
            ],
            "X509StoreName": "My"
        },
        "ClusterCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames" : "Root"
            }
        ],
        "ServerCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },
        "ServerCertificateCommonNames": {
            "CommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]"
            }
            ],
            "X509StoreName": "My"
        },
        "ServerCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames" : "Root"
            }
        ],
        "ClientCertificateThumbprints": [
            {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": false
            },
            {
                "CertificateThumbprint": "[Thumbprint]",
                "IsAdmin": true
            }
        ],
        "ClientCertificateCommonNames": [
            {
                "CertificateCommonName": "[CertificateCommonName]",
                "CertificateIssuerThumbprint": "[Thumbprint1,Thumbprint2,Thumbprint3,...]",
                "IsAdmin": true
            }
        ],
        "ClientCertificateIssuerStores": [
            {
                "IssuerCommonName": "[IssuerCommonName]",
                "X509StoreNames": "Root"
            }
        ]
        "ReverseProxyCertificate": {
            "Thumbprint": "[Thumbprint]",
            "ThumbprintSecondary": "[Thumbprint]",
            "X509StoreName": "My"
        },
        "ReverseProxyCertificateCommonNames": {
            "CommonNames": [
                {
                "CertificateCommonName": "[CertificateCommonName]"
                }
            ],
            "X509StoreName": "My"
        }
    }
},
```

En esta sección se describen los certificados que necesita para proteger el clúster de Windows independiente. Si va a especificar un certificado de clúster, establezca el valor de ClusterCredentialType en _X509_. Para especificar un certificado de servidor para conexiones externas, establezca la propiedad ServerCredentialType en _X509_. Si bien no es obligatorio, se recomienda tener ambos certificados para contar con un clúster protegido correctamente. Si establece estos valores en *X509*, también debe especificar los certificados correspondientes. De lo contrario, Service Fabric generará una excepción. En algunos escenarios, es posible que solo desee especificar _ClientCertificateThumbprints_ o _ReverseProxyCertificate_. En esos escenarios, no es necesario establecer _ClusterCredentialType_ o _ServerCredentialType_ en _X509_.


> [!NOTE]
> Una [huella digital](https://en.wikipedia.org/wiki/Public_key_fingerprint) es la identidad principal de un certificado. Consulte [Retrieve a thumbprint of a certificate](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate) (Recuperación de la huella digital de un certificado) para averiguar la huella digital de los certificados que cree.
> 
> 

En la siguiente tabla se enumeran los certificados que va a necesitar en su instalación del clúster:

| **Valor de CertificateInformation** | **Descripción** |
| --- | --- |
| ClusterCertificate |Se recomienda para un entorno de prueba. Este certificado es necesario para proteger la comunicación entre los nodos de un clúster. Puede utilizar dos certificados diferentes, uno principal y otro secundario para la actualización. Establezca la huella digital del certificado principal en la sección Thumbprint y la del secundario en la variable ThumbprintSecondary. |
| ClusterCertificateCommonNames |Se recomienda para un entorno de producción. Este certificado es necesario para proteger la comunicación entre los nodos de un clúster. Puede utilizar uno o dos nombres comunes del certificado de clúster. CertificateIssuerThumbprint corresponde a la huella digital del emisor del certificado. Si se está usando más de 1 certificado con el mismo nombre común, puede especificar varias huellas digitales de emisor.|
| ClusterCertificateIssuerStores |Se recomienda para un entorno de producción. Este certificado corresponde al emisor del certificado de clúster. Puede proporcionar el nombre común del emisor y el nombre de almacén correspondiente en esta sección en lugar de especificar la huella digital del emisor en ClusterCertificateCommonNames.  Esto facilita la sustitución de los certificados del emisor de clúster. Se pueden especificar varios emisores si se usa más de un certificado de clúster. Un valor IssuerCommonName vacío permite todos los certificados de los almacenes correspondientes especificados en X509StoreNames.|
| ServerCertificate |Se recomienda para un entorno de prueba. Este certificado se presenta al cliente cuando intenta conectarse a este clúster. Para mayor comodidad, puede utilizar el mismo certificado para ClusterCertificate y ServerCertificate. Puede utilizar dos certificados de servidor diferentes, uno principal y otro secundario para la actualización. Establezca la huella digital del certificado principal en la sección Thumbprint y la del secundario en la variable ThumbprintSecondary. |
| ServerCertificateCommonNames |Se recomienda para un entorno de producción. Este certificado se presenta al cliente cuando intenta conectarse a este clúster. CertificateIssuerThumbprint corresponde a la huella digital del emisor del certificado. Si se está usando más de 1 certificado con el mismo nombre común, puede especificar varias huellas digitales de emisor. Por comodidad, puede utilizar el mismo certificado para ClusterCertificateCommonNames y ServerCertificateCommonNames. Puede utilizar uno o dos nombres comunes de certificado de servidor. |
| ServerCertificateIssuerStores |Se recomienda para un entorno de producción. Este certificado corresponde al emisor del certificado de servidor. Puede proporcionar el nombre común del emisor y el nombre de almacén correspondiente en esta sección en lugar de especificar la huella digital del emisor en ServerCertificateCommonNames.  Esto facilita la sustitución de los certificados del emisor de servidor. Se pueden especificar varios emisores si se usa más de un certificado de servidor. Un valor IssuerCommonName vacío permite todos los certificados de los almacenes correspondientes especificados en X509StoreNames.|
| ClientCertificateThumbprints |Instale este conjunto de certificados en los clientes autenticados. Puede tener varios certificados de cliente diferentes instalados en los equipos a los que desea permitir el acceso al clúster. Establece la huella digital de cada certificado en la variable CertificateThumbprint. Si establece IsAdmin en *True*, el cliente con este certificado instalado puede realizar actividades de administración en el clúster. Si IsAdmin es *false*, el cliente con este certificado solo puede realizar las acciones permitidas para los derechos de acceso de usuario, normalmente de solo lectura. Para obtener más información sobre roles, consulte [Control de acceso basado en rol (RBAC)](service-fabric-cluster-security.md#role-based-access-control-rbac). |
| ClientCertificateCommonNames |Establezca el nombre común del primer certificado de cliente para CertificateCommonName. CertificateIssuerThumbprint es la huella digital del emisor de este certificado. Consulte [Trabajar con certificados](/dotnet/framework/wcf/feature-details/working-with-certificates) para obtener más información sobre los nombres comunes y el emisor. |
| ClientCertificateIssuerStores |Se recomienda para un entorno de producción. Este certificado corresponde al emisor del certificado de cliente (roles de administrador y de otro tipo). Puede proporcionar el nombre común del emisor y el nombre de almacén correspondiente en esta sección en lugar de especificar la huella digital del emisor en ClientCertificateCommonNames.  Esto facilita la sustitución de los certificados del emisor de cliente. Se pueden especificar varios emisores si se usa más de un certificado de cliente. Un valor IssuerCommonName vacío permite todos los certificados de los almacenes correspondientes especificados en X509StoreNames.|
| ReverseProxyCertificate |Se recomienda para un entorno de prueba. Se puede especificar este certificado opcional si desea proteger el [proxy inverso](service-fabric-reverseproxy.md). Asegúrese de que reverseProxyEndpointPort está establecido en nodeTypes si usa este certificado. |
| ReverseProxyCertificateCommonNames |Se recomienda para un entorno de producción. Se puede especificar este certificado opcional si desea proteger el [proxy inverso](service-fabric-reverseproxy.md). Asegúrese de que reverseProxyEndpointPort está establecido en nodeTypes si usa este certificado. |

Este es un ejemplo de configuración del clúster en el que se han proporcionado los certificados de clúster, servidor y cliente. En lo que respecta a los certificados de clúster, servidor y reverseProxy, no se permite configurar conjuntamente huellas digitales y nombres comunes para el mismo tipo de certificado.

 ```JSON
 {
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "10-2017",
    "nodes": [{
        "nodeName": "vm0",
        "metadata": "Replace the localhost below with valid IP address or FQDN",
        "iPAddress": "10.7.0.5",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD0"
    }, {
        "nodeName": "vm1",
        "metadata": "Replace the localhost with valid IP address or FQDN",
        "iPAddress": "10.7.0.4",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r1",
        "upgradeDomain": "UD1"
    }, {
        "nodeName": "vm2",
        "iPAddress": "10.7.0.6",
        "metadata": "Replace the localhost with valid IP address or FQDN",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r2",
        "upgradeDomain": "UD2"
    }],
    "properties": {
        "diagnosticsStore": {
        "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
        "dataDeletionAgeInDays": "7",
        "storeType": "FileShare",
        "IsEncrypted": "false",
        "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
        },
        "security": {
            "metadata": "The Credential type X509 indicates this cluster is secured by using X509 certificates. The thumbprint format is d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
            "ClusterCredentialType": "X509",
            "ServerCredentialType": "X509",
            "CertificateInformation": {
                "ClusterCertificateCommonNames": {
                  "CommonNames": [
                    {
                      "CertificateCommonName": "myClusterCertCommonName"
                    }
                  ],
                  "X509StoreName": "My"
                },
                "ClusterCertificateIssuerStores": [
                    {
                        "IssuerCommonName": "ClusterIssuer1",
                        "X509StoreNames" : "Root"
                    },
                    {
                        "IssuerCommonName": "ClusterIssuer2",
                        "X509StoreNames" : "Root"
                    }
                ],
                "ServerCertificateCommonNames": {
                  "CommonNames": [
                    {
                      "CertificateCommonName": "myServerCertCommonName",
                      "CertificateIssuerThumbprint": "7c fc 91 97 13 16 8d ff a8 ee 71 2b a2 f4 62 62 00 03 49 0d"
                    }
                  ],
                  "X509StoreName": "My"
                },
                "ClientCertificateThumbprints": [{
                    "CertificateThumbprint": "c4 c18 8e aa a8 58 77 98 65 f8 61 4a 0d da 4c 13 c5 a1 37 6e",
                    "IsAdmin": false
                }, {
                    "CertificateThumbprint": "71 de 04 46 7c 9e d0 54 4d 02 10 98 bc d4 4c 71 e1 83 41 4e",
                    "IsAdmin": true
                }]
            }
        },
        "reliabilityLevel": "Bronze",
        "nodeTypes": [{
            "name": "NodeType0",
            "clientConnectionEndpointPort": "19000",
            "clusterConnectionEndpointPort": "19001",
            "leaseDriverEndpointPort": "19002",
            "serviceConnectionEndpointPort": "19003",
            "httpGatewayEndpointPort": "19080",
            "applicationPorts": {
                "startPort": "20001",
                "endPort": "20031"
            },
            "ephemeralPorts": {
                "startPort": "20032",
                "endPort": "20062"
            },
            "isPrimary": true
        }
         ],
        "fabricSettings": [{
            "name": "Setup",
            "parameters": [{
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
            }, {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
            }]
        }]
    }
}
 ```

## <a name="certificate-rollover"></a>Sustitución del certificado
Al utilizar el nombre común del certificado en lugar de la huella digital, el proceso de sustitución de certificados no precisa actualizar la configuración de clúster. Para las actualizaciones de huella digital de emisor, asegúrese de que la nueva lista de huella tenga intersección con la lista anterior. Primero, tiene que realizar una actualización de configuración con las huellas digitales del emisor nuevo y, a continuación, instalar los nuevos certificados (certificados de clúster o servidor, y de emisor) en el almacén. Mantenga el certificado de emisor antiguo en el almacén de certificados durante al menos dos horas después de instalar el nuevo certificado de emisor.
Si usa almacenes de emisor, no es necesario realizar ninguna actualización de configuración para la sustitución del certificado de emisor. Instale el nuevo certificado de emisor con una fecha de expiración de este último en el almacén de certificados correspondiente y quite el antiguo certificado de emisor tras unas horas.

## <a name="acquire-the-x509-certificates"></a>Adquisición de certificados X.509
Para proteger la comunicación en el clúster, primero debe obtener certificados X.509 para los nodos del clúster. Además, para limitar la conexión a este clúster a los equipos o usuarios autorizados, debe obtener e instalar certificados para los equipos cliente.

Para los clústeres que ejecutan cargas de trabajo de producción, use un certificado X.509 firmado por una [entidad de certificación (CA)](https://en.wikipedia.org/wiki/Certificate_authority) con el fin de proteger el clúster. Para obtener más información sobre cómo obtener estos certificados, consulte [Cómo obtener un certificado](/dotnet/framework/wcf/feature-details/how-to-obtain-a-certificate-wcf). 

Hay una serie de propiedades que el certificado debe tener para funcionar correctamente:

* El proveedor del certificado debe ser el **proveedor de servicios criptográficos RSA y AES mejorado de Microsoft**.

* Al crear una clave RSA, asegúrese de que la clave es de **2048 bits**.

* La extensión uso de clave tiene un valor de **Firma digital, cifrado de clave (a0)**

* La extensión uso mejorado de clave tiene valores de **autenticación de servidor** (OID: 1.3.6.1.5.5.7.3.1) y **Autenticación de cliente** (OID: (1.3.6.1.5.5.7.3.2)

En los clústeres que se usan con fines de prueba, puede usar un certificado autofirmado.

Para obtener más preguntas, consulte las [preguntas más frecuentes sobre los certificados](./cluster-security-certificate-management.md#troubleshooting-and-frequently-asked-questions).

## <a name="optional-create-a-self-signed-certificate"></a>Opcional: Creación de un certificado autofirmado
Una forma de crear un certificado autofirmado que se puede proteger correctamente es usar el script CertSetup.ps1 de la carpeta del SDK de Service Fabric en el directorio C:\Archivos de programa\Microsoft SDKs\Service Fabric\ClusterSetup\Secure. Edite este archivo para cambiar el nombre predeterminado del certificado. (Busque el valor CN = CN=ServiceFabricDevClusterCert). Ejecute este script como `.\CertSetup.ps1 -Install`.

Ahora exporte el certificado a un archivo PFX con una contraseña protegida. En primer lugar, obtenga la huella digital del certificado. 
1. Desde el menú **Inicio**, ejecute **Administrar certificados de equipo**. 

2. Vaya a la carpeta **Equipo local\Personal** y busque el certificado creado. 

3. Haga doble clic en el certificado para abrirlo, seleccione la pestaña **Detalles** y desplácese hacia abajo hasta el campo **Huella digital**. 

4. Quite los espacios y copie el valor de la huella digital en el comando de PowerShell siguiente. 

5. Cambie el valor `String` por una contraseña segura adecuada para protegerlo y ejecute el siguiente comando en PowerShell:

   ```powershell   
   $pswd = ConvertTo-SecureString -String "1234" -Force –AsPlainText
   Get-ChildItem -Path cert:\localMachine\my\<Thumbprint> | Export-PfxCertificate -FilePath C:\mypfx.pfx -Password $pswd
   ```

6. Para ver los detalles de un certificado instalado en el equipo, ejecute el siguiente comando de PowerShell:

   ```powershell
   $cert = Get-Item Cert:\LocalMachine\My\<Thumbprint>
   Write-Host $cert.ToString($true)
   ```

O bien, si tiene una suscripción a Azure, siga los pasos que se describen en [Creación de un clúster de Service Fabric con Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).

## <a name="install-the-certificates"></a>Instalación de los certificados
Cuando tenga los certificados, puede instalarlos en los nodos del clúster. Los nodos deben tener instalada la versión más reciente de Windows PowerShell 3.x. Repita estos pasos en cada nodo, para los certificados del clúster y el servidor y cualquier certificado secundario.

1. Copie los archivos .pfx en el nodo.

2. Abra una ventana de PowerShell como administrador y escriba los siguientes comandos. Reemplace *$pswd* por la contraseña que usó para crear este certificado. Reemplace *$PfxFilePath* por la ruta de acceso completa del archivo .pfx copiado en este nodo.
   
    ```powershell
    $pswd = "1234"
    $PfxFilePath ="C:\mypfx.pfx"
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\LocalMachine\My -FilePath $PfxFilePath -Password (ConvertTo-SecureString -String $pswd -AsPlainText -Force)
    ```
3. Ahora debe establecer el control de acceso de este certificado para que el proceso Service Fabric, que se ejecuta en la cuenta de servicio de red, pueda usarlo al ejecutar el script siguiente. Proporcione la huella digital del certificado y especifique **SERVICIO DE RED** para la cuenta de servicio. Para comprobar que las ACL del certificado son correctas, abra el certificado en **Inicio** > **Administrar certificados de equipo** y consulte **Todas las tareas** > **Administrar claves privadas**.
   
    ```powershell
    param
    (
    [Parameter(Position=1, Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [string]$pfxThumbPrint,
   
    [Parameter(Position=2, Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [string]$serviceAccount
    )
   
    $cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object -FilterScript { $PSItem.ThumbPrint -eq $pfxThumbPrint; }
   
    # Specify the user, the permissions, and the permission type
    $permission = "$($serviceAccount)","FullControl","Allow" # "NT AUTHORITY\NetworkService" is the service account
    $accessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $permission
   
    # Location of the machine-related keys
    $keyPath = Join-Path -Path $env:ProgramData -ChildPath "\Microsoft\Crypto\RSA\MachineKeys"
    $keyName = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
    $keyFullPath = Join-Path -Path $keyPath -ChildPath $keyName
   
    # Get the current ACL of the private key
    $acl = (Get-Item $keyFullPath).GetAccessControl('Access')
   
    # Add the new ACE to the ACL of the private key
    $acl.SetAccessRule($accessRule)
   
    # Write back the new ACL
    Set-Acl -Path $keyFullPath -AclObject $acl -ErrorAction Stop
   
    # Observe the access rights currently assigned to this certificate
    get-acl $keyFullPath| fl
    ```
4. Repita los pasos anteriores para cada certificado de servidor. También puede usar estos pasos para instalar los certificados de cliente en las máquinas a las que desea permitir el acceso al clúster.

## <a name="create-the-secure-cluster"></a>Creación del clúster seguro
Después de configurar la sección security del archivo ClusterConfig.X509.MultiMachine.json, puede continuar con la sección [Creación del clúster](service-fabric-cluster-creation-for-windows-server.md#create-the-cluster) para configurar los nodos y crear el clúster independiente. Recuerde usar el archivo ClusterConfig.X509.MultiMachine.json al crear el clúster. Por ejemplo, el comando podría ser similar al siguiente:

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json
```

Una vez que el clúster de Windows independiente seguro se ejecuta correctamente y que ha configurado los clientes autenticados para conectarse a él, siga los pasos de la sección [Conexión a un clúster con PowerShell](service-fabric-connect-to-secure-cluster.md#connect-to-a-cluster-using-powershell) para conectarse a él. Por ejemplo:

```powershell
$ConnectArgs = @{  ConnectionEndpoint = '10.7.0.5:19000';  X509Credential = $True;  StoreLocation = 'LocalMachine';  StoreName = "MY";  ServerCertThumbprint = "057b9544a6f2733e0c8d3a60013a58948213f551";  FindType = 'FindByThumbprint';  FindValue = "057b9544a6f2733e0c8d3a60013a58948213f551"   }
Connect-ServiceFabricCluster $ConnectArgs
```

Luego puede ejecutar otros comandos de PowerShell para trabajar con este clúster. Por ejemplo, puede ejecutar [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) para mostrar una lista de los nodos en este clúster protegido.


Para quitar el clúster, conéctese al nodo del clúster en el que descargó el paquete de Service Fabric, abra una línea de comandos y vaya a la carpeta del paquete. Ahora ejecute el comando siguiente:

```powershell
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.X509.MultiMachine.json
```

> [!NOTE]
> Una configuración incorrecta de un certificado puede impedir que el clúster se muestre durante la implementación. Para realizar un autodiagnóstico de los problemas de seguridad, consulte en el grupo del Visor de eventos **Registros de aplicaciones y servicios** > **Microsoft Service Fabric**.
> 
> 
