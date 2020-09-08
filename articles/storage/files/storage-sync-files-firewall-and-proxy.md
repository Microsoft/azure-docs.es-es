---
title: Configuración del proxy y el firewall locales de Azure File Sync | Microsoft Docs
description: Comprenda la configuración del proxy y el firewall locales de Azure File Sync. Revise los detalles de configuración de puertos, redes y conexiones especiales con Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 06/24/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: e4f011d9286a0685f1b091b930155db969407423
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87903721"
---
# <a name="azure-file-sync-proxy-and-firewall-settings"></a>Configuración del proxy y el firewall de Azure File Sync
Azure File Sync conecta los servidores locales a Azure Files, lo que permite sincronizar las características de niveles de nube y de sincronización multisitio. Por lo tanto, un servidor local debe estar conectado a Internet. Un administrador de TI tiene que decidir cuál es la mejor ruta de acceso para que el servidor acceda a los servicios en la nube de Azure.

Este artículo ofrece un resumen de las opciones y los requisitos específicos disponibles para conectar el servidor a Azure File Sync de forma correcta y segura.

Se recomienda leer [Consideraciones de redes para Azure File Sync](storage-sync-files-networking-overview.md) antes de pasar a esta guía de procedimientos.

## <a name="overview"></a>Información general
Azure File Sync funciona como un servicio de orquestación entre el servidor Windows, el recurso compartido de archivos de Azure y otros servicios de Azure con el objetivo de sincronizar los datos tal y como se describe en el grupo de sincronización. Para que Azure File Sync funcione correctamente, debe configurar los servidores para que se comuniquen con los siguientes servicios de Azure:

- Azure Storage
- Azure File Sync
- Azure Resource Manager
- Servicios de autenticación

> [!Note]  
> El agente de Azure File Sync en Windows Server inicia todas las solicitudes a servicios en la nube, por lo que solo hay que tener en cuenta el tráfico saliente en el firewall. <br /> Ningún servicio de Azure inicia una conexión con el agente de Azure File Sync.

## <a name="ports"></a>Puertos
Azure File Sync mueve los metadatos y los datos de archivo exclusivamente a través de HTTPS y requiere que el puerto 443 esté abierto al tráfico saliente.
Como resultado, se cifra todo el tráfico.

## <a name="networks-and-special-connections-to-azure"></a>Redes y conexiones especiales a Azure
El agente de Azure File Sync no tiene ningún requisito en lo que respecta a canales especiales como [ExpressRoute](../../expressroute/expressroute-introduction.md), etc., en Azure.

Azure File Sync funciona con cualquier medio disponible que permita conectarse con Azure, de modo que se adapta automáticamente a diversas características de red, como el ancho de banda, la latencia y la posibilidad de usar el control de administración para realizar ajustes. No todas las características están disponibles en este momento. Si desea configurar un comportamiento específico, comuníquelo en [UserVoice de Azure Files](https://feedback.azure.com/forums/217298-storage?category_id=180670).

## <a name="proxy"></a>Proxy
Azure File Sync admite la configuración del proxy específico de aplicación en el nivel de máquina.

De este modo, se permite la **configuración de un proxy específico de la aplicación**, específicamente para el tráfico de Azure File Sync. La configuración de un proxy específico de la aplicación es compatible con la versión 4.0.1.0 o versiones posteriores, y se puede configurar durante la instalación del agente o mediante el cmdlet Set-StorageSyncProxyConfiguration de PowerShell.

Comandos de PowerShell para configurar el proxy específico de la aplicación:
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Set-StorageSyncProxyConfiguration -Address <url> -Port <port number> -ProxyCredential <credentials>
```
**Esta configuración de proxy en el nivel de máquina** es transparente para el agente de Azure File Sync, ya que todo el tráfico del servidor se enruta a través de este proxy.

Para configurar los valores de proxy en el nivel de máquina, siga estos pasos: 

1. Configuración de valores de proxy para aplicaciones de .NET 

   - Edite estos dos archivos:  
     C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config  
     C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config

   - Agregue la sección <system.net> en los archivos machine.config (debajo de la sección <system.serviceModel>).  Cambie 127.0.01:8888 a la dirección IP y puerto del servidor proxy. 
     ```
      <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
          <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
      </system.net>
     ```

2. Establecimiento de la configuración del proxy WinHTTP 

   - Ejecute el siguiente comando desde un símbolo del sistema con privilegios elevados o PowerShell para ver la configuración del proxy existente:   

     netsh winhttp show proxy

   - Ejecute el siguiente comando desde un símbolo del sistema con privilegios elevados o PowerShell para establecer la configuración de proxy (cambie 127.0.01:8888 a la dirección IP y puerto del servidor proxy):  

     netsh winhttp set proxy 127.0.0.1:8888

3. Reinicie el servicio del agente de sincronización de Azure Storage ejecutando el comando siguiente desde un símbolo del sistema con privilegios elevados o PowerShell: 

      net stop filesyncsvc

      Nota: El servicio del agente de sincronización de Storage (filesyncsvc) se iniciará automáticamente una vez detenido.

## <a name="firewall"></a>Firewall
Como se mencionó en una sección anterior, el puerto 443 tiene que abrirse al tráfico saliente. En función de las directivas del centro de datos, la rama o la región, puede ser recomendable u obligatorio restringir aún más el tráfico a través de este puerto a dominios concretos.

En la tabla siguiente se describen los dominios necesarios para la comunicación:

| Servicio | Punto de conexión de la nube pública | Punto de conexión de Azure Government | Uso |
|---------|----------------|---------------|------------------------------|
| **Azure Resource Manager** | `https://management.azure.com` | https://management.usgovcloudapi.net | Todas las llamadas de usuario (como PowerShell) van a esta URL, incluida la llamada de registro inicial del servidor. |
| **Azure Active Directory** | https://login.windows.net<br>`https://login.microsoftonline.com` | https://login.microsoftonline.us | Las llamadas de Azure Resource Manager deben realizarlas usuarios autenticados. Para que el proceso se complete correcta, esta URL se utiliza para la autenticación de usuarios. |
| **Azure Active Directory** | https://graph.microsoft.com/ | https://graph.microsoft.com/ | Como parte de la implementación de Azure File Sync, se crea una entidad de servicio en la instancia de Azure Active Directory de la suscripción. Esta URL se utiliza con ese fin. Esta entidad de seguridad se usa para delegar un conjunto mínimo de derechos en el servicio Azure File Sync. El usuario que realiza la configuración inicial de Azure File Sync debe ser un usuario autenticado con privilegios de propietario en la suscripción. |
| **Azure Active Directory** | https://secure.aadcdn.microsoftonline-p.com | Use la dirección URL del punto de conexión público. | La biblioteca de autenticación de Active Directory tiene acceso a esta dirección URL que usa la interfaz de usuario de registro del servidor de Azure File Sync para iniciar sesión en el administrador. |
| **Almacenamiento de Azure** | &ast;.core.windows.net | &ast;.core.usgovcloudapi.net | Cuando el servidor descarga un archivo, realiza el movimiento de datos de forma más eficaz al comunicarse directamente con el recurso compartido de archivos de Azure en la cuenta de almacenamiento. El servidor tiene una clave SAS que solo se permite el acceso al recurso compartido de archivos de destino. |
| **Azure File Sync** | &ast;.one.microsoft.com<br>&ast;.afs.azure.net | &ast;. afs.azure.us | Después del registro inicial del servidor, el servidor recibe una URL regional para la instancia de servicio de Azure File Sync en dicha región. El servidor puede utilizar la URL para comunicarse de forma directa y eficaz con la instancia que controla la sincronización. |
| **Microsoft PKI** | https://www.microsoft.com/pki/mscorp/cps<br><http://ocsp.msocsp.com> | https://www.microsoft.com/pki/mscorp/cps<br><http://ocsp.msocsp.com> | Una vez instalado el agente Azure File Sync, la dirección URL de PKI se utiliza para descargar los certificados intermedios necesarios para comunicarse con el servicio Azure File Sync y el recurso compartido de archivos de Azure. La dirección URL de OCSP se utiliza para comprobar el estado de un certificado. |

> [!Important]
> Si permite el tráfico a &ast;.afs.azure.net, el tráfico solo podrá enviarse al servicio sincronización. Ningún otro servicio de Microsoft usa este dominio.
> Al permitir el tráfico a &ast;.one.microsoft.com, es posible dirigir el tráfico del servidor a más recursos que simplemente el servicio de sincronización. Existen muchos otros servicios de Microsoft en subdominios.

Si &ast;.afs.azure.net o &ast;.one.microsoft.com es demasiado larga, puede limitar la comunicación del servidor al permitir solo la comunicación con instancias regionales explícitas del servicio Azure Files Sync. La elección de las instancias dependerá de la región del servicio de sincronización de almacenamiento en el que haya implementado y registrado el servidor. Esa región se denomina "Dirección URL del punto de conexión principal" en la tabla siguiente.

Por razones de continuidad empresarial y recuperación ante desastres (BCDR), es posible que haya especificado los recursos compartidos de archivos de Azure en una cuenta de almacenamiento globalmente redundante (GRS). Si es así, los recursos compartidos de archivos de Azure conmutarán por error a la región emparejada en caso de una interrupción regional duradera. Azure File Sync usa los mismos emparejamientos regionales que el almacenamiento. Por lo tanto, si usa cuentas de almacenamiento GRS, deberá habilitar direcciones URL adicionales para permitir que el servidor se comunique con la región emparejada para Azure File Sync. En la tabla siguiente, se denomina "Región emparejada". Además, hay una dirección URL del perfil de Traffic Manager que debe habilitarse también. Esto asegurará que el tráfico de red se pueda redirigir sin problemas a la región emparejada en caso de que se produzca una conmutación por error y se denomina "URL de detección" en la tabla siguiente.

| Nube  | Region | Dirección URL del punto de conexión principal | Región emparejada | Dirección URL de detección |
|--------|--------|----------------------|---------------|---------------|
| Público |Este de Australia | https:\//australiaeast01.afs.azure.net<br>https:\//kailani-aue.one.microsoft.com | Sudeste de Australia | https:\//tm-australiaeast01.afs.azure.net<br>https:\//tm-kailani-aue.one.microsoft.com |
| Público |Sudeste de Australia | https:\//australiasoutheast01.afs.azure.net<br>https:\//kailani-aus.one.microsoft.com | Este de Australia | https:\//tm-australiasoutheast01.afs.azure.net<br>https:\//tm-kailani-aus.one.microsoft.com |
| Público | Sur de Brasil | https:\//brazilsouth01.afs.azure.net | Centro-sur de EE. UU. | https:\//tm-brazilsouth01.afs.azure.net |
| Público | Centro de Canadá | https:\//canadacentral01.afs.azure.net<br>https:\//kailani-cac.one.microsoft.com | Este de Canadá | https:\//tm-canadacentral01.afs.azure.net<br>https:\//tm-kailani-cac.one.microsoft.com |
| Público | Este de Canadá | https:\//canadaeast01.afs.azure.net<br>https:\//kailani-cae.one.microsoft.com | Centro de Canadá | https:\//tm-canadaeast01.afs.azure.net<br>https:\//tm-kailani.cae.one.microsoft.com |
| Público | Centro de la India | https:\//centralindia01.afs.azure.net<br>https:\//kailani-cin.one.microsoft.com | Sur de la India | https:\//tm-centralindia01.afs.azure.net<br>https:\//tm-kailani-cin.one.microsoft.com |
| Público | Centro de EE. UU. | https:\//centralus01.afs.azure.net<br>https:\//kailani-cus.one.microsoft.com | Este de EE. UU. 2 | https:\//tm-centralus01.afs.azure.net<br>https:\//tm-kailani-cus.one.microsoft.com |
| Público | Este de Asia | https:\//eastasia01.afs.azure.net<br>https:\//kailani11.one.microsoft.com | Sudeste de Asia | https:\//tm-eastasia01.afs.azure.net<br>https:\//tm-kailani11.one.microsoft.com |
| Público | Este de EE. UU. | https:\//eastus01.afs.azure.net<br>https:\//kailani1.one.microsoft.com | Oeste de EE. UU. | https:\//tm-eastus01.afs.azure.net<br>https:\//tm-kailani1.one.microsoft.com |
| Público | Este de EE. UU. 2 | https:\//eastus201.afs.azure.net<br>https:\//kailani-ess.one.microsoft.com | Centro de EE. UU. | https:\//tm-eastus201.afs.azure.net<br>https:\//tm-kailani-ess.one.microsoft.com |
| Público | Japón Oriental | https:\//japaneast01.afs.azure.net | Japón Occidental | https:\//tm-japaneast01.afs.azure.net |
| Público | Japón Occidental | https:\//japanwest01.afs.azure.net | Japón Oriental | https:\//tm-japanwest01.afs.azure.net |
| Público | Centro de Corea del Sur | https:\//koreacentral01.afs.azure.net/ | Corea del Sur | https:\//tm-koreacentral01.afs.azure.net/ |
| Público | Corea del Sur | https:\//koreasouth01.afs.azure.net/ | Centro de Corea del Sur | https:\//tm-koreasouth01.afs.azure.net/ |
| Público | Centro-Norte de EE. UU | https:\//northcentralus01.afs.azure.net | Centro-sur de EE. UU. | https:\//tm-northcentralus01.afs.azure.net |
| Público | Norte de Europa | https:\//northeurope01.afs.azure.net<br>https:\//kailani7.one.microsoft.com | Oeste de Europa | https:\//tm-northeurope01.afs.azure.net<br>https:\//tm-kailani7.one.microsoft.com |
| Público | Centro-sur de EE. UU. | https:\//southcentralus01.afs.azure.net | Centro-Norte de EE. UU | https:\//tm-southcentralus01.afs.azure.net |
| Público | Sur de la India | https:\//southindia01.afs.azure.net<br>https:\//kailani-sin.one.microsoft.com | Centro de la India | https:\//tm-southindia01.afs.azure.net<br>https:\//tm-kailani-sin.one.microsoft.com |
| Público | Sudeste de Asia | https:\//southeastasia01.afs.azure.net<br>https:\//kailani10.one.microsoft.com | Este de Asia | https:\//tm-southeastasia01.afs.azure.net<br>https:\//tm-kailani10.one.microsoft.com |
| Público | Sur de Reino Unido | https:\//uksouth01.afs.azure.net<br>https:\//kailani-uks.one.microsoft.com | Oeste de Reino Unido | https:\//tm-uksouth01.afs.azure.net<br>https:\//tm-kailani-uks.one.microsoft.com |
| Público | Oeste de Reino Unido | https:\//ukwest01.afs.azure.net<br>https:\//kailani-ukw.one.microsoft.com | Sur de Reino Unido | https:\//tm-ukwest01.afs.azure.net<br>https:\//tm-kailani-ukw.one.microsoft.com |
| Público | Centro-Oeste de EE. UU. | https:\//westcentralus01.afs.azure.net | Oeste de EE. UU. 2 | https:\//tm-westcentralus01.afs.azure.net |
| Público | Oeste de Europa | https:\//westeurope01.afs.azure.net<br>https:\//kailani6.one.microsoft.com | Norte de Europa | https:\//tm-westeurope01.afs.azure.net<br>https:\//tm-kailani6.one.microsoft.com |
| Público | Oeste de EE. UU. | https:\//westus01.afs.azure.net<br>https:\//kailani.one.microsoft.com | Este de EE. UU. | https:\//tm-westus01.afs.azure.net<br>https:\//tm-kailani.one.microsoft.com |
| Público | Oeste de EE. UU. 2 | https:\//westus201.afs.azure.net | Centro-Oeste de EE. UU. | https:\//tm-westus201.afs.azure.net |
| Government | US Gov: Arizona | https:\//usgovarizona01.afs.azure.us | US Gov Texas | https:\//tm-usgovarizona01.afs.azure.us |
| Government | US Gov Texas | https:\//usgovtexas01.afs.azure.us | US Gov: Arizona | https:\//tm-usgovtexas01.afs.azure.us |

- Si utiliza cuentas de almacenamiento con redundancia local (LRS) o de almacenamiento con redundancia de zona (ZRS), solo tiene que habilitar la URL que aparece en "Dirección URL del punto de conexión principal".

- Si usa cuentas de almacenamiento con redundancia geográfica (GRS), habilite tres direcciones URL.

**Ejemplo**: Implemente un servicio de sincronización de almacenamiento en `"West US"` y registre el servidor allí. Las direcciones URL que permiten que el servidor se comunique en este caso son:

> - https:\//westus01.afs.azure.net (punto de conexión principal: Oeste de EE. UU.)
> - https:\//eastus01.afs.azure.net (región de conmutación por error emparejada: Este de EE. UU.)
> - https:\//tm-westus01.afs.azure.net (dirección URL de detección de la región primaria)

### <a name="allow-list-for-azure-file-sync-ip-addresses"></a>Lista de direcciones IP de Azure File Sync permitidas
Azure File Sync permite usar [etiquetas de servicio](../../virtual-network/service-tags-overview.md), que representan un grupo de prefijos de direcciones IP relativos a un servicio de Azure determinado. Se pueden usar etiquetas de servicio para crear reglas de firewall que permitan la comunicación con el servicio Azure File Sync. La etiqueta de servicio de Azure File Sync es `StorageSyncService`.

Si usa Azure File Sync en Azure, puede usar el nombre de la etiqueta de servicio directamente en el grupo de seguridad de red para permitir el tráfico. Para saber cómo, vea [Grupos de seguridad de red](../../virtual-network/security-overview.md).

Si usa Azure File Sync de forma local, puede usar la API de etiquetas de servicio para obtener intervalos de direcciones IP específicos para la lista de elementos permitidos del firewall. Existen dos formas de obtener esta información:

- La lista actual de intervalos de direcciones IP de todos los servicios de Azure que admiten etiquetas de servicio se publica semanalmente en el Centro de descarga de Microsoft en forma de documento JSON. Cada nube de Azure tiene su propio documento JSON con los intervalos de direcciones IP pertinentes para esa nube:
    - [Azure público](https://www.microsoft.com/download/details.aspx?id=56519)
    - [Azure US Government](https://www.microsoft.com/download/details.aspx?id=57063)
    - [Azure en China](https://www.microsoft.com/download/details.aspx?id=57062)
    - [Azure Alemania](https://www.microsoft.com/download/details.aspx?id=57064)
- La API de detección de etiquetas de servicio (versión preliminar) permite recuperar mediante programación la lista actual de etiquetas de servicio. En la versión preliminar, es posible que la API de detección de etiquetas de servicio devuelva información menos actual que la información incluida en los documentos JSON publicados en el Centro de descarga de Microsoft. Puede usar la superficie de API en función de su preferencia de automatización:
    - [REST API](https://docs.microsoft.com/rest/api/virtualnetwork/servicetags/list)
    - [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.network/Get-AzNetworkServiceTag)
    - [CLI de Azure](https://docs.microsoft.com/cli/azure/network#az-network-list-service-tags)

Dado que la API de detección de etiquetas de servicio no se actualiza con tanta frecuencia como los documentos JSON publicados en el Centro de descarga de Microsoft, se recomienda usar este último para actualizar la lista de elementos permitidos del firewall local. Se puede hacer de la forma siguiente:

```PowerShell
# The specific region to get the IP address ranges for. Replace westus2 with the desired region code 
# from Get-AzLocation.
$region = "westus2"

# The service tag for Azure File Sync. Do not change unless you're adapting this
# script for another service.
$serviceTag = "StorageSyncService"

# Download date is the string matching the JSON document on the Download Center. 
$possibleDownloadDates = 0..7 | `
    ForEach-Object { [System.DateTime]::Now.AddDays($_ * -1).ToString("yyyyMMdd") }

# Verify the provided region
$validRegions = Get-AzLocation | `
    Where-Object { $_.Providers -contains "Microsoft.StorageSync" } | `
    Select-Object -ExpandProperty Location

if ($validRegions -notcontains $region) {
    Write-Error `
            -Message "The specified region $region is not available. Either Azure File Sync is not deployed there or the region does not exist." `
            -ErrorAction Stop
}

# Get the Azure cloud. This should automatically based on the context of 
# your Az PowerShell login, however if you manually need to populate, you can find
# the correct values using Get-AzEnvironment.
$azureCloud = Get-AzContext | `
    Select-Object -ExpandProperty Environment | `
    Select-Object -ExpandProperty Name

# Build the download URI
$downloadUris = @()
switch($azureCloud) {
    "AzureCloud" { 
        $downloadUris = $possibleDownloadDates | ForEach-Object {  
            "https://download.microsoft.com/download/7/1/D/71D86715-5596-4529-9B13-DA13A5DE5B63/ServiceTags_Public_$_.json"
        }
    }

    "AzureUSGovernment" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/6/4/D/64DB03BF-895B-4173-A8B1-BA4AD5D4DF22/ServiceTags_AzureGovernment_$_.json"
        }
    }

    "AzureChinaCloud" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/9/D/0/9D03B7E2-4B80-4BF3-9B91-DA8C7D3EE9F9/ServiceTags_China_$_.json"
        }
    }

    "AzureGermanCloud" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/0/7/6/076274AB-4B0B-4246-A422-4BAF1E03F974/ServiceTags_AzureGermany_$_.json"
        }
    }

    default {
        Write-Error -Message "Unrecognized Azure Cloud: $_" -ErrorAction Stop
    }
}

# Find most recent file
$found = $false 
foreach($downloadUri in $downloadUris) {
    try { $response = Invoke-WebRequest -Uri $downloadUri -UseBasicParsing } catch { }
    if ($response.StatusCode -eq 200) {
        $found = $true
        break
    }
}

if ($found) {
    # Get the raw JSON 
    $content = [System.Text.Encoding]::UTF8.GetString($response.Content)

    # Parse the JSON
    $serviceTags = ConvertFrom-Json -InputObject $content -Depth 100

    # Get the specific $ipAddressRanges
    $ipAddressRanges = $serviceTags | `
        Select-Object -ExpandProperty values | `
        Where-Object { $_.id -eq "$serviceTag.$region" } | `
        Select-Object -ExpandProperty properties | `
        Select-Object -ExpandProperty addressPrefixes
} else {
    # If the file cannot be found, that means there hasn't been an update in
    # more than a week. Please verify the download URIs are still accurate
    # by checking https://docs.microsoft.com/azure/virtual-network/service-tags-overview
    Write-Verbose -Message "JSON service tag file not found."
    return
}
```

Después, puede usar los intervalos de direcciones IP en `$ipAddressRanges` para actualizar el firewall. Consulte el sitio web del dispositivo de firewall/red para obtener información sobre cómo actualizar el firewall.

## <a name="test-network-connectivity-to-service-endpoints"></a>Prueba de la conectividad de red con los puntos de conexión de servicio
Una vez que se registra un servidor con el servicio de Azure File Sync, se puede usar el cmdlet Test-StorageSyncNetworkConnectivity cmdlet y ServerRegistration.exe para probar la comunicación con todos los puntos de conexión (URL) específicos de este servidor. Este cmdlet puede ayudar a solucionar problemas si una comunicación incompleta impide que el servidor funcione totalmente con Azure File Sync, y se puede usar para ajustar las configuraciones de proxy y firewall.

Para ejecutar la prueba de conectividad de red, instale la versión 9.1 o posterior del agente de Azure File Sync y ejecute los siguientes comandos de PowerShell:
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Test-StorageSyncNetworkConnectivity
```

## <a name="summary-and-risk-limitation"></a>Resumen y limitación de riesgos
Las listas que aparecen anteriormente en este documento contienen las URL con las que actualmente se comunica Azure File Sync. Los firewalls deben poder permitir el tráfico de salida a estos dominios. Microsoft se esfuerza por mantener esta lista actualizada.

Configurar reglas de firewall que restrinjan dominios puede ser una medida para mejorar la seguridad. Si se usan estas configuraciones de firewall, hay que tener en cuenta que se agregarán direcciones URL que incluso podrían cambiar con el tiempo. Compruebe periódicamente este artículo.

## <a name="next-steps"></a>Pasos siguientes
- [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md)
- [Implementación de Azure File Sync](storage-sync-files-deployment-guide.md)
- [Supervisión de Azure File Sync](storage-sync-files-monitoring.md)
