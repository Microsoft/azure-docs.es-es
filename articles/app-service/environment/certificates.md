---
title: Enlaces de certificados
description: Se explican los numerosos temas relacionados con los certificados en una instancia de App Service Environment. Obtenga información sobre cómo funcionan los enlaces de certificados en las aplicaciones de un solo inquilino de App Service Environment.
author: ccompy
ms.assetid: 9e21a7e4-2436-4e81-bb05-4a6ba70eeaf7
ms.topic: article
ms.date: 08/29/2018
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 306445e26e5b236b49273b9ab8888ecc610bc075
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962050"
---
# <a name="certificates-and-the-app-service-environment"></a>Certificados y App Service aislado 

Una instancia de App Service aislado es una implementación de Azure App Service que se ejecuta en su instancia de Azure Virtual Network (VNet). Se puede implementar con un punto de conexión de la aplicación accesible a Internet o un punto de conexión de la aplicación que se encuentra en la red virtual. Si App Service aislado se implementa con un punto de conexión accesible a Internet, dicha implementación se denomina App Service aislado externo. Si App Service aislado se implementa con un punto de conexión en la red virtual, dicha implementación se denomina App Service aislado con ILB. Puede obtener más información sobre App Service aislado con ILB en el documento [Creación y uso de App Service aislado con ILB](./create-ilb-ase.md).

App Service aislado es un sistema de inquilino único. Debido a esto, existen algunas características que solo están disponibles con App Service aislado que no está disponible en la instancia de App Service multiinquilino. 

## <a name="ilb-ase-certificates"></a>Certificados de App Service aislado con ILB 

Si usa una instancia de App Service aislado externo, se establece contacto con las aplicaciones en [appname].[asename].p.azurewebsites.net. De forma predeterminada, todas las instancias de App Service aislado, incluso las de App Service aislado con ILB, se crean con certificados que siguen ese formato. Cuando tiene una instancia de App Service aislado con ILB, se establece contacto con las aplicaciones según el nombre de dominio que especifique al crear App Service aislado con ILB. Para que las aplicaciones admitan TLS, debe cargar los certificados. La obtención de un certificado TLS/SSL válido se realiza mediante autoridades de certificados internas, la adquisición de un certificado de un emisor externo o el uso de un certificado autofirmado. 

Hay dos opciones para configurar certificados con App Service aislado con ILB.  Puede establecer un certificado predeterminado de carácter comodín para App Service aislado con ILB o bien certificados en las aplicaciones web individuales de App Service aislado.  Independientemente de la opción que escoja, es necesario configurar correctamente los siguientes atributos del certificado:

- **Firmante:** el valor de este atributo tiene que establecerse como *.[su-dominio-raíz] en el caso de un certificado de App Service Environment con ILB de carácter comodín. Si crea el certificado para la aplicación, debe ser [appname].[su-dominio-raíz].
- **Nombre alternativo del firmante**: este atributo tiene que incluir tanto *.[su-dominio-raíz] como *.scm.[su-dominio-raíz] para el certificado de App Service Environment con ILB de carácter comodín. Si crea el certificado para la aplicación, debe ser [appname].[su-dominio-raíz] y [appname].scm.[su-dominio-raíz].

Como tercera variante, puede crear un certificado de App Service aislado con ILB que incluya todos los nombres de las aplicaciones individuales del SAN del certificado en lugar de usar una referencia de carácter comodín. El problema con este método es que necesita conocer por adelantado los nombres de las aplicaciones que va a incluir en App Service aislado o mantener actualizado el certificado de App Service aislado con ILB.

### <a name="upload-certificate-to-ilb-ase"></a>Carga del certificado en App Service aislado con ILB 

Después de crear una instancia de App Service aislado con ILB en el portal, se debe establecer el certificado para dicha instancia. Hasta que no se establezca, App Service aislado mostrará un banner indicando que no se ha establecido el certificado.  

El certificado que cargue debe ser un archivo .pfx. Tras cargar el certificado, App Service aislado llevará a cabo una operación de escalado para establecer el certificado. 

No puede crear la instancia de App Service aislado ni cargar el certificado como una acción en el portal o en una plantilla. De forma alternativa, puede cargar el certificado mediante una plantilla según se describe en el documento [Creación de una instancia de App Service aislado mediante una plantilla](./create-from-template.md).  

Si quiere crear un certificado autofirmado rápidamente para realizar pruebas, puede usar el siguiente fragmento de PowerShell:

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password
```

Al crear un certificado autofirmado, deberá asegurar que el nombre del sujeto tiene el formato de CN = {ASE_NAME_HERE} _InternalLoadBalancingASE.

## <a name="application-certificates"></a>Certificados de aplicación 

Las aplicaciones hospedadas en una instancia de App Service aislado pueden usar las características de los certificados centradas en la aplicación que están disponibles en la instancia de App Service multiinquilino. Estas características son:  

- Certificados SNI 
- SSL basada en IP, que solo es compatible con una instancia de App Service aislado externo.  Una instancia de App Service aislado con ILB no es compatible con la SSL basada en IP.
- Certificados hospedados en Key Vault 

Las instrucciones para cargar y administrar dichos certificados están disponibles en [Adición de un certificado TLS/SSL en Azure App Service](../configure-ssl-certificate.md).  Si está configurando los certificados simplemente para que coincidan con un nombre de dominio personalizado que ha asignado a la aplicación web, esas instrucciones serán suficientes. Si va a cargar el certificado para una aplicación web de App Service aislado con ILB con el nombre de dominio predeterminado, a continuación, especifique el sitio de SCM en la SAN del certificado como se indicó anteriormente. 

## <a name="tls-settings"></a>Configuración de TLS 

Puede establecer la configuración de TLS en un nivel de aplicación.  

## <a name="private-client-certificate"></a>Certificado de cliente privado 

Es un caso de uso común consiste en configurar la aplicación como un cliente en un modelo cliente/servidor. Si protege el servidor con un certificado de entidad de certificación privado, deberá cargar el certificado de cliente en la aplicación.  Las instrucciones siguientes servirán para la carga de certificados en el almacén de confianza de los trabajadores en el que se ejecuta la aplicación. Si carga el certificado en una aplicación, se puede utilizar con el resto de aplicaciones en el mismo plan de App Service sin volver a cargar el certificado.

Para cargar el certificado en la aplicación de App Service aislado:

1. Genere un archivo *.cer* para el certificado. 
2. Vaya a la aplicación que necesita el certificado en Azure Portal.
3. Vaya a la configuración de SSL en la aplicación. Haga clic en Cargar certificado. Seleccione Público. Seleccione Máquina local. Proporcione un nombre. Busque y seleccione el archivo *.cer*. Seleccione Cargar. 
4. Copie la huella digital.
5. Vaya a Configuración de la aplicación. Cree una configuración de aplicación WEBSITE_LOAD_ROOT_CERTIFICATES con la huella digital como valor. Si tiene varios certificados, puede colocarlos en la misma configuración separados por comas y sin espacios en blanco, como se muestra a continuación: 

    84EC242A4EC7957817B8E48913E50953552DAFA6,6A5C65DC9247F762FE17BF8D4906E04FE6B31819

El certificado estará disponible para todas las aplicaciones del mismo plan de App Service que la aplicación que configuró dicho valor. Si necesita que esté disponible para las aplicaciones en otro plan de App Service, deberá repetir la operación Configuración de la aplicación en una aplicación en dicho plan de App Service. Para comprobar si el certificado está establecido, vaya a la consola de Kudu y emita este comando en la consola de depuración de PowerShell.

```azurepowershell-interactive
dir cert:\localmachine\root
```

Para realizar pruebas, puede crear un certificado autofirmado y generar un archivo *.cer* con el siguiente comando de PowerShell: 

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.cer"
export-certificate -Cert $certThumbprint -FilePath $fileName -Type CERT
```