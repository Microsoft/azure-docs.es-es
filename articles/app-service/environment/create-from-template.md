---
title: Creación de un ASE con ARM
description: Aprenda a crear una instancia externa o con un ILB de App Service Environment mediante una plantilla de Azure Resource Manager.
author: ccompy
ms.assetid: 6eb7d43d-e820-4a47-818c-80ff7d3b6f8e
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: a6b9ba8c3f72b070ad1ec9ade9684a182e2dea08
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962407"
---
# <a name="create-an-ase-by-using-an-azure-resource-manager-template"></a>Creación de una instancia de ASE mediante el uso de una plantilla de Azure Resource Manager

## <a name="overview"></a>Información general

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Las instancias de Azure App Service Environment (ASE) pueden crearse con un punto de conexión accesible de Internet o un punto de conexión en una dirección interna en una red virtual de Azure (VNet). Cuando se crea con un punto de conexión interno, ese punto de conexión siempre lo proporciona un componente de Azure denominado equilibrador de carga interno (ILB). La instancia de ASE en una dirección IP interna se denomina ASE de ILB. La instancia de ASE con un punto de conexión público se denomina ASE externo. 

Un ASE puede crearse con Azure Portal o con una plantilla de Azure Resource Manager. Este artículo le guía por los pasos y la sintaxis que necesita para crear un ASE externo o uno con ILB con plantillas de Resource Manager. Para información sobre cómo crear una instancia de ASE en Azure Portal, consulte [Creación de una instancia externa de App Service Environment][MakeExternalASE] o [Creación de una instancia de Azure App Environment de LIB][MakeILBASE].

Cuando crea un ASE en Azure Portal, puede crear la red virtual al mismo tiempo o seleccionar una red virtual existente en donde implementarlo. Cuando crea un ASE desde una plantilla, debe comenzar por: 

* Una red virtual de Resource Manager.
* Una subred de esa red virtual. Se recomienda que el tamaño de la subred del ASE sea `/24` con 256 direcciones para adaptarse al crecimiento futuro. Una vez creado el ASE, no se puede cambiar el tamaño.
* El identificador de recurso de la red virtual. Puede obtener esta información en Azure Portal, en las propiedades de su red virtual.
* La suscripción en la que desea implementarlo.
* La ubicación en la que desea implementarlo.

Para automatizar la creación del ASE:

1. Cree el ASE a partir de una plantilla. Si crea un ASE externo, habrá terminado después de este paso. Si crea un ASE con ILB, hay que realizar algunas acciones adicionales.

2. Una vez creado el ASE con ILB, se carga un certificado TLS/SSL que coincide con su dominio de ASE con ILB.

3. El certificado TLS/SSL cargado se asigna al ASE con ILB como certificado TLS/SSL "predeterminado".  Este certificado se usa en el tráfico TLS/SSL que va a las aplicaciones del ASE con ILB cuando usan el dominio raíz común asignado al ASE (por ejemplo, `https://someapp.mycustomrootdomain.com`).


## <a name="create-the-ase"></a>Creación del ASE
En GitHub, encontrará [un ejemplo][quickstartasev2create] de plantilla de Resource Manager que crea una instancia de ASE y su archivo de parámetros asociado.

Si quiere crear una instancia de ASE con ILB, use estos [ejemplos][quickstartilbasecreate] de plantillas de Resource Manager. Son específicas a dicho caso de uso. La mayoría de los parámetros del archivo *azuredeploy.parameters.json* son comunes para la creación de los ASE con un ILB y de los ASE externos. La siguiente lista llama a los parámetros especiales o únicos cuando crea un ASE con ILB:

* *internalLoadBalancingMode*: en la mayoría de los casos se establece en 3, lo que significa que tanto el tráfico HTTP/HTTPS de los puertos 80 y 443 como los puertos de los canales de control o de datos a los que escucha el servicio FTP en el ASE estarán enlazados a una dirección de red virtual interna asignada al ILB. Si esta propiedad se establece en 2, solo los puertos relacionados con el servicio FTP (los canales de control y de datos) estarán enlazados a una dirección de ILB. El tráfico HTTP/HTTPS permanece en la VIP pública.
* *dnsSuffix*: este parámetro define el dominio raíz predeterminado que se asigna al ASE. En la variación pública de Azure App Service, el dominio raíz predeterminado de todas las aplicaciones web es *azurewebsites.net*. Dado que un ASE con ILB está dentro de la red virtual de un cliente, no tiene sentido utilizar el dominio raíz predeterminado del servicio público. En su lugar, un ASE de ILB debe tener un dominio raíz predeterminado que tenga sentido usar en la red virtual interna de una compañía. Por ejemplo, Contoso Corporation podría usar el dominio raíz predeterminado *interno contoso.com* para aquellas aplicaciones que se pretende que se puedan resolver en la red virtual de Contoso, y a las que solo se pueda acceder desde ella. 
* *ipSslAddressCount*: automáticamente, el valor predeterminado de este parámetro es 0 en el archivo *azuredeploy.json*, ya que los ASE con ILB solo tienen una dirección de ILB individual. No hay direcciones IP-SSL explícitas para un ASE con ILB. Por lo tanto, el grupo de direcciones IP-SSL para un ASE con ILB debe establecerse en cero. En caso contrario, se produce un error de aprovisionamiento. 

Después de que se llene el archivo *azuredeploy.parameters.json*, cree el ASE mediante el fragmento de código de PowerShell. Cambie las rutas del archivo para que coincidan con las ubicaciones en las que se encuentran los archivos de plantilla de Resource Manager. Recuerde especificar sus propios valores para el nombre de implementación y el nombre del grupo de recursos de Resource Manager:

```powershell
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

La creación de un ASE tarda aproximadamente una hora. Luego el ASE se muestra en el portal en la lista de ASE de la suscripción que desencadenó la implementación.

## <a name="upload-and-configure-the-default-tlsssl-certificate"></a>Carga y configuración del certificado TLS/SSL "predeterminado"
Es preciso asociar un certificado TLS/SSL "predeterminado" con el ASE para establecer conexiones TLS/SSL con las aplicaciones. Si el sufijo DNS predeterminado del ASE es *internal-contoso.com*, una conexión a `https://some-random-app.internal-contoso.com` requiere un certificado TLS/SSL que sea válido para * *.internal-contoso.com*. 

Se puede obtener un certificado TLS/SSL válido mediante autoridades de certificados internas, al adquirir un certificado de un emisor externo o al usar un certificado autofirmado. Independientemente del origen del certificado TLS/SSL, es necesario configurar correctamente los siguientes atributos del certificado:

* **Firmante**: Este atributo se debe establecer en * *.su-dominio-raíz.com*.
* **Nombre alternativo del firmante**: Este atributo debe incluir tanto * *.su-dominio-raíz.com* como * *.scm.su-dominio-raíz.com*. Las conexiones TLS con el sitio de SCM/Kudu asociadas a cada aplicación usan una dirección cuyo formato será *nombre-de-aplicación.scm.dominio-raíz.com*.

Con un certificado TLS/SSL válido, se necesitan dos pasos preparatorios adicionales. Convierta/guarde el certificado TLS/SSL como un archivo .pfx. Recuerde que el archivo .pfx debe incluir todos los certificados intermedios y raíz. Protéjalo con una contraseña.

El archivo .pfx debe convertirse en una cadena base64, ya que el certificado TLS/SSL se cargará mediante una plantilla de Resource Manager. Dado que las plantillas de Resource Manager son archivos de texto, el archivo .pfx debe convertirse en una cadena base64. De esta forma, se puede incluir como un parámetro de la plantilla.

Use el siguiente fragmento de código de PowerShell para:

* Generar un certificado autofirmado.
* Exportar el certificado como un archivo .pfx.
* Convierta el archivo .pfx en una cadena codificada en base64.
* Guarde la cadena codificada en base64 en un archivo independiente. 

El código de PowerShell para la codificación en base64 se adaptó del [blog de scripts de PowerShell][examplebase64encoding]:

```powershell
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

$fileContentBytes = get-content -encoding byte $fileName
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$fileContentEncoded | set-content ($fileName + ".b64")
```

Una vez que el certificado TLS/SSL se genera y convierte correctamente en una cadena con codificación en base64, use el ejemplo de la plantilla de Resource Manager [Configuración del certificado SSL predeterminado][quickstartconfiguressl] en GitHub. 

Los parámetros del archivo *azuredeploy.parameters.json* se enumeran aquí:

* *appServiceEnvironmentName*: el nombre del ASE de ILB que se configura.
* *existingAseLocation*: cadena de texto que contiene la región de Azure en que se implementó el ASE de ILB.  Por ejemplo: "Centro-sur de EE. UU."
* *pfxBlobString*: la representación de la cadena con codificación Base64 del archivo pfx. Use el fragmento de código mostrado anteriormente y copie la cadena incluida en "exportedcert.pfx.b64". Péguelo como el valor del atributo *pfxBlobString*.
* *password*: la contraseña que se usa para proteger el archivo pfx.
* *certificateThumbprint*: la huella digital del certificado. Si este valor se recupera de PowerShell (por ejemplo, *$certificate.Thumbprint* del fragmento de código anterior), se puede usar tal cual. Si copia el valor del cuadro de diálogo del certificado de Windows, no olvide eliminar los espacios superfluos. *certificateThumbprint* debería ser similar a AF3143EB61D43F6727842115BB7F17BBCECAECAE.
* *certificateName*: es el identificador de cadena fácil de usar que elija y que se usa para identificar el certificado. El nombre se usa como parte del identificador único de Resource Manager para la entidad *Microsoft.Web/certificates* que representa el certificado TLS/SSL. El nombre *debe* terminar con el sufijo siguiente: \__yourASENameHere_InternalLoadBalancingASE. Azure Portal utiliza este sufijo como un indicador de que el certificado se usa para asegurar un ASE habilitado para ILB.

Aquí se muestra un ejemplo abreviado de *azuredeploy.parameters.json* :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "appServiceEnvironmentName": {
      "value": "yourASENameHere"
    },
    "existingAseLocation": {
      "value": "East US 2"
    },
    "pfxBlobString": {
      "value": "MIIKcAIBAz...snip...snip...pkCAgfQ"
    },
    "password": {
      "value": "PASSWORDGOESHERE"
    },
    "certificateThumbprint": {
      "value": "AF3143EB61D43F6727842115BB7F17BBCECAECAE"
    },
    "certificateName": {
      "value": "DefaultCertificateFor_yourASENameHere_InternalLoadBalancingASE"
    }
  }
}
```

Después de que se llene el archivo *azuredeploy.parameters.json*, configure el certificado TLS/SSL mediante el fragmento de código de PowerShell. Cambie las rutas del archivo para que coincidan con la ubicación de la máquina en la que se encuentran los archivos de plantilla de Resource Manager. Recuerde especificar sus propios valores para el nombre de implementación y el nombre del grupo de recursos de Resource Manager:

```powershell
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

La aplicación del cambio se tarda aproximadamente 40 minutos por cada front-end de ASE. Por ejemplo, para un ASE de un tamaño predeterminado que usa dos front-ends, la plantilla tarda aproximadamente una hora y veinte minutos en completarse. Mientras la plantilla se ejecute, el ASE no se puede escalar.  

Una vez finalizada la plantilla, se puede tener acceso a aplicaciones en el ASE con ILB a través de HTTPS. Las conexiones se protegen mediante el certificado TLS/SSL predeterminado. El certificado TLS/SSL predeterminado se usa cuando las direcciones de las aplicaciones del ASE con ILB usan una combinación del nombre de la aplicación y el nombre de host predeterminado. Por ejemplo, `https://mycustomapp.internal-contoso.com` usa el certificado TLS/SSL predeterminado para * *.internal-contoso.com*.

Sin embargo, al igual que las aplicaciones que se ejecutan en el servicio multiinquilino público, los desarrolladores pueden configurar nombres de host personalizados para las aplicaciones individuales. También pueden configurar enlaces de certificado TLS/SSL SNI únicos para las aplicaciones individuales.

## <a name="app-service-environment-v1"></a>App Service Environment v1 ##
App Service Environment tiene dos versiones: ASEv1 y ASEv2. La información anterior se basa en ASEv2. En esta sección se muestran las diferencias entre ASEv1 y ASEv2.

En ASEv1, administra todos los recursos de forma manual. Esto incluye los front-end, los trabajos y las direcciones IP utilizadas para SSL basada en IP. Antes de escalar horizontalmente el plan de App Service, debe escalar horizontalmente el grupo de trabajo en el que desea hospedarlo.

ASEv1 utiliza un modelo de precios diferente al de ASEv2. En ASEv1, paga por cada vCPU asignada. Esto incluye las vCPU que se usan para los servidores front-end y trabajos que no hospedan ninguna carga de trabajo. En ASEv1, el tamaño de escala máxima predeterminada de una instancia de ASE es de 55 hosts en total. Esto incluye servidores front-end y trabajos. Una ventaja de ASEv1 es que se puede implementar en una red virtual clásica y en una red virtual de Resource Manager. Para más información sobre ASEv1, consulte [Introducción a App Service Environment v1][ASEv1Intro].

Para crear una instancia de ASEv1 mediante una plantilla de Resource Manager, consulte [Creación de una instancia de ASEv1 de ILB con una plantilla de Azure Resource Manager][ILBASEv1Template].


<!--Links-->
[quickstartilbasecreate]: https://azure.microsoft.com/documentation/templates/201-web-app-asev2-ilb-create
[quickstartasev2create]: https://azure.microsoft.com/documentation/templates/201-web-app-asev2-create
[quickstartconfiguressl]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl
[quickstartwebapponasev2create]: https://azure.microsoft.com/documentation/templates/201-web-app-asp-app-on-asev2-create
[examplebase64encoding]: https://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../../app-service/configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[ILBASEv1Template]: app-service-app-service-environment-create-ilb-ase-resourcemanager.md