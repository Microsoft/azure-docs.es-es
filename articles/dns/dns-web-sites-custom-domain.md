---
title: 'Tutorial: Creación de registros de Azure DNS personalizados para una aplicación web'
description: En este tutorial, va a crear registros DNS de dominios personalizados para aplicaciones web mediante Azure DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 3/11/2019
ms.author: rohink
ms.openlocfilehash: 7a250eaeb8ac4d0c8cacfb1748855700c9986bf5
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079491"
---
# <a name="tutorial-create-dns-records-in-a-custom-domain-for-a-web-app"></a>Tutorial: Creación de registros DNS en un dominio personalizado para una aplicación web 

Puede configurar Azure DNS para hospedar un dominio personalizado para aplicaciones web. Por ejemplo, puede crear una aplicación web de Azure y hacer que los usuarios accedan a ella mediante www\.contoso.com o contoso.com como un nombre de dominio completo (FQDN).

> [!NOTE]
> Contoso.com se utiliza como ejemplo en todo este tutorial. Sustituya su propio nombre de dominio por contoso.com.

Para ello, deberá crear tres registros:

* Un registro raíz "A" que apunta a contoso.com
* Un registro raíz "TXT" para la comprobación
* Un registro "CNAME" para el nombre www que apunta al registro A

Tenga en cuenta que si crea un registro A para una aplicación web en Azure, este debe actualizarse manualmente si la dirección IP subyacente de la aplicación web cambia.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un registro D y TXT para el dominio personalizado
> * Creación de un registro CNAME para el dominio personalizado
> * Prueba de los nuevos registros
> * Adición de nombres de host personalizado a la aplicación web
> * Prueba de nombres de host personalizados


Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* Tiene que tener un nombre de dominio disponible que pueda hospedar en Azure DNS para realizar las pruebas. Debe tener control total de este dominio. El control total incluye la capacidad de establecer los registros de nombre de servidor (NS) para el dominio.
* [Cree una aplicación de App Service](../app-service/quickstart-html.md) o use alguna aplicación que haya creado para otro tutorial.

* Antes de comenzar, debe crear una zona DNS en Azure DNS y delegar la zona del registrador a Azure DNS.

   1. Para crear una zona DNS, siga los pasos descritos en [Introducción a DNS de Azure](dns-getstarted-create-dnszone.md).
   2. Para delegar su zona a Azure DNS, siga los pasos descritos en [Delegación de un dominio en Azure DNS](dns-delegate-domain-azure-dns.md).

Después de crear una zona y delegarla a DNS de Azure, podrá crear registros para el dominio personalizado.

## <a name="create-an-a-record-and-txt-record"></a>Creación de un registro D y de un registro TXT

Un registro A se usa para asignar un nombre a su dirección IP. En el ejemplo siguiente, asigne "\@" como un registro A mediante la dirección IPv4 de la aplicación web. \@ normalmente representa el dominio raíz.

### <a name="get-the-ipv4-address"></a>Obtención de la dirección IPv4

En el panel de navegación izquierdo de la página App Services en Azure Portal, seleccione **Dominios personalizados**. 

![Menú Dominio personalizado](../app-service/./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

En la página **Dominios personalizados**, copie la dirección IPv4 de la aplicación:

![Navegación en el portal a la aplicación de Azure](../app-service/./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### <a name="create-the-a-record"></a>Crear el registro D

```azurepowershell
New-AzDnsRecordSet -Name "@" -RecordType "A" -ZoneName "contoso.com" `
 -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 `
 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "<your web app IP address>")
```

### <a name="create-the-txt-record"></a>Crear el registro TXT

App Services usa este registro solo durante la configuración para comprobar que se posee el dominio personalizado. Puede eliminar este registro TXT después de validar el dominio personalizado y de configurarlo en App Service.

> [!NOTE]
> Si desea comprobar el nombre de dominio, pero sin enrutar el tráfico de producción a la aplicación web, solo tendrá que especificar el registro TXT para el paso de comprobación.  La comprobación de no requiere un registro A o CNAME además el registro TXT.

```azurepowershell
New-AzDnsRecordSet -ZoneName contoso.com -ResourceGroupName MyAzureResourceGroup `
 -Name "@" -RecordType "txt" -Ttl 600 `
 -DnsRecords (New-AzDnsRecordConfig -Value  "contoso.azurewebsites.net")
```

## <a name="create-the-cname-record"></a>Crear un registro CNAME

Si el dominio ya lo administra Azure DNS (consulte [Delegación de dominios DNS](dns-domain-delegation.md)), puede utilizar el ejemplo siguiente para crear un registro CNAME para contoso.azurewebsites.net.

Abra Azure PowerShell y cree un nuevo registro CNAME. En este ejemplo se crea un conjunto de registros de tipo CNAME con un "período de vida" de 600 segundos en la zona DNS denominada "contoso.com" con el alias de la aplicación web contoso.azurewebsites.net.

### <a name="create-the-record"></a>Creación del registro

```azurepowershell
New-AzDnsRecordSet -ZoneName contoso.com -ResourceGroupName "MyAzureResourceGroup" `
 -Name "www" -RecordType "CNAME" -Ttl 600 `
 -DnsRecords (New-AzDnsRecordConfig -cname "contoso.azurewebsites.net")
```

El ejemplo siguiente es la respuesta:

```
    Name              : www
    ZoneName          : contoso.com
    ResourceGroupName : myresourcegroup
    Ttl               : 600
    Etag              : 8baceeb9-4c2c-4608-a22c-229923ee185
    RecordType        : CNAME
    Records           : {contoso.azurewebsites.net}
    Tags              : {}
```

## <a name="test-the-new-records"></a>Prueba de los nuevos registros

Para asegurarse de que los registros se hayan creado correctamente, consulte “www.contoso.com” y "contoso.com" con nslookup, como se muestra a continuación:

```
PS C:\> nslookup
Default Server:  Default
Address:  192.168.0.1

> www.contoso.com
Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
Name:    <instance of web app service>.cloudapp.net
Address:  <ip of web app service>
Aliases:  www.contoso.com
contoso.azurewebsites.net
<instance of web app service>.vip.azurewebsites.windows.net

> contoso.com
Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
Name:    contoso.com
Address:  <ip of web app service>

> set type=txt
> contoso.com

Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
contoso.com text =

        "contoso.azurewebsites.net"
```
## <a name="add-custom-host-names"></a>Adición de nombres de host personalizados

Ahora puede agregar los nombres de host personalizados a la aplicación web:

```azurepowershell
set-AzWebApp `
 -Name contoso `
 -ResourceGroupName MyAzureResourceGroup `
 -HostNames @("contoso.com","www.contoso.com","contoso.azurewebsites.net")
```
## <a name="test-the-custom-host-names"></a>Prueba de nombres de host personalizados

Abra un explorador y vaya a `http://www.<your domainname>` y `http://<you domain name>`.

> [!NOTE]
> Asegúrese de incluir el prefijo `http://`; en caso contrario, el explorador puede intentar predecir por su cuenta una dirección URL.

Debería ver la misma página para ambas direcciones URL. Por ejemplo:

![Servicio de aplicación de Contoso](media/dns-web-sites-custom-domain/contoso-app-svc.png)


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos creados en este tutorial, puede eliminar el grupo de recursos **myresourcegroup**.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a crear las zonas privadas de Azure DNS.

> [!div class="nextstepaction"]
> [Introducción al uso de PowerShell con las zonas privadas de Azure DNS](private-dns-getstarted-powershell.md)
