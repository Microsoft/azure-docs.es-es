---
title: Configuración de un nombre de dominio personalizado en Cloud Services | Microsoft Docs
description: Aprenda a exponer su aplicación o sus datos de Azure en Internet en un dominio personalizado mediante la configuración de sus valores DNS.  Estos ejemplos usan el Portal de Azure.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/05/2017
ms.author: tagore
ms.openlocfilehash: 37189df6b1c9bf3f9fca185226f2ee3eeb3ddd7d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092735"
---
# <a name="configuring-a-custom-domain-name-for-an-azure-cloud-service"></a>Configuración de un nombre de dominio personalizado para un servicio en la nube de Azure
Cuando se crea un servicio en la nube, Azure lo asigna a un subdominio de **cloudapp.net**. Por ejemplo, si el nombre del servicio en la nube es "contoso", los usuarios podrán tener acceso a la aplicación en una dirección URL como `http://contoso.cloudapp.net`. Azure también asigna una dirección IP virtual.

Sin embargo, también puede exponer su aplicación en su propio nombre de dominio, como **contoso.com**. En este artículo se explica cómo reservar o configurar un nombre de dominio personalizado para los roles web de servicio en la nube.

¿Ya entiende qué son los registros CNAME y D? [Omita la explicación](#add-a-cname-record-for-your-custom-domain).

> [!NOTE]
> Los procedimientos de esta tarea se aplican a Azure Cloud Services. Para App Services, vea [Asignar un nombre DNS personalizado a Azure Web Apps](../app-service/app-service-web-tutorial-custom-domain.md). En el caso de las cuentas de almacenamiento clásicas, vea [Configurar un nombre de dominio personalizado para el punto de conexión de Blob Storage](../storage/blobs/storage-custom-domain-name.md).
> 
> 

<p/>

> [!TIP]
> Póngase en marcha más rápido: use el NUEVO [tutorial guiado](https://support.microsoft.com/kb/2990804)de Azure.  Con este tutorial, asociar un nombre de dominio personalizado y proteger las comunicaciones (TLS) con Azure Cloud Services o Azure Websites es facilísimo.
> 
> 

## <a name="understand-cname-and-a-records"></a>Descripción de los registros D y CNAME
Los registros D y CNAME (o registros de alias) le permiten asociar un nombre de dominio a un servidor específico (o, en este caso, servicio). Sin embargo, cada uno funciona de manera distinta. Existen también algunas consideraciones específicas al usar los registros D con los Servicios en la nube de Azure que debe considerar antes de decidir cuál usar.

### <a name="cname-or-alias-record"></a>Registro de CNAME o Alias
Un registro CNAME asigna un dominio *específico*, como **contoso.com** o **www\.contoso.com**, a un nombre de dominio canónico. En este caso, el nombre de dominio canónico es el nombre de dominio **[myapp].cloudapp.net** de su aplicación hospedada de Azure. Una vez creado, el CNAME crea un alias para **[myapp].cloudapp.net**. La entrada de CNAME se resolverá en la dirección IP del servicio de **[myapp].cloudapp.net** de manera automática, por lo que si la dirección IP del servicio en la nube cambia, no es preciso realizar ninguna acción.

> [!NOTE]
> Algunos registradores de dominio solo permiten asignar subdominios cuando se usa un registro CNAME, como www\.contoso.com, y no nombres raíz, como contoso.com. Para obtener más información acerca de los registros CNAME, consulte la documentación que proporciona el registrador, [la entrada de Wikipedia sobre el registro CNAME](https://en.wikipedia.org/wiki/CNAME_record) o el documento [Nombres de dominio IETF: implementación y especificación (en inglés)](https://tools.ietf.org/html/rfc1035).

### <a name="a-record"></a>Registro D
El registro *D* asigna un dominio, como **contoso.com** o **www\.contoso.como**, *o un nombre de dominio con comodín*, como **\*.contoso.com**, a una dirección IP. En el caso de un servicio en la nube de Azure, la IP virtual del servicio. Por lo tanto, el principal beneficio de un registro D en relación con un registro CNAME es que puede disponer de una entrada que utilice un carácter comodín, como \* **.contoso.com**, que administraría las solicitudes de varios subdominios como **mail.contoso.com**, **login.contoso.com** o **www\.contoso.com**.

> [!NOTE]
> Puesto que un registro D se asigna a una dirección IP estática, no puede resolver automáticamente cambios en la dirección IP de su servicio en la nube. La dirección IP que usa el Servicio en la nube se asigna la primera vez que se implementa en una ranura vacía (producción o ensayo). Si elimina la implementación para la ranura, Azure libera la dirección IP y a toda implementación posterior en la ranura se le podrá dar una dirección IP nueva.
> 
> De manera conveniente, la dirección IP de una ranura de implementación dada (producción o ensayo) se mantiene cuando se realiza un intercambio entre las implementaciones de ensayo y producción o se realiza una actualización local de una implementación existente. Para obtener más información sobre la realización de estas acciones, consulte [Administración de servicios en la nube](cloud-services-how-to-manage-portal.md).
> 
> 

## <a name="add-a-cname-record-for-your-custom-domain"></a>Incorporación de un CNAME al dominio personalizado
Para crear un registro CNAME, debe agregar una nueva entrada en la tabla DNS para el dominio personalizado mediante las herramientas proporcionadas por el registrador. Cada registrador dispone de un método similar pero ligeramente distinto de especificación de un registro CNAME. Sin embargo, los conceptos son los mismos.

1. Use uno de estos métodos para buscar el nombre de dominio **.cloudapp.net** asignado a su servicio en la nube.

   * Inicie sesión en [Azure Portal], seleccione el servicio en la nube, busque la sección **Información general** y, luego, busque la entrada **Dirección URL del sitio**.

       ![sección de vista rápida que muestra la dirección URL del sitio][csurl]

       **OR**
   * Instale y configure [Azure Powershell](/powershell/azure/)y, luego, use el siguiente comando:

       ```powershell
       Get-AzureDeployment -ServiceName yourservicename | Select Url
       ```

     Guarde el nombre de dominio que se usó en la URL devuelta por cualquier método, ya que lo necesitará al crear un registro CNAME.
2. Inicie sesión en el sitio web del registrador DNS y vaya a la página de administración de DNS. Busque los vínculos o áreas del sitio etiquetados como **Nombre de dominio**, **DNS** o **Administración del servidor de nombres**.
3. Ahora busque el lugar en el que puede seleccionar o especificar los registros CNAME. Es posible que tenga que seleccionar el tipo de registro de un menú desplegable o ir a una página de configuración avanzada. Debe buscar las palabras **CNAME**, **Alias** o **Subdominios**.
4. También debe proporcionar el alias del dominio o subdominio para el registro CNAME, como **www** si desea crear un alias para **www\.customdomain.com**. Si desea crear un alias para el dominio raíz, puede enumerarse como el símbolo " **\@** " en las herramientas de DNS del registrador.
5. A continuación, debe proporcionar un nombre de host canónico que, en este caso, es el dominio **cloudapp.net** de su aplicación.

Por ejemplo, el siguiente registro CNAME reenvía todo el tráfico de **www\.contoso.com** a **contoso.cloudapp.net**, el nombre de dominio personalizado de la aplicación implementada:

| Alias/Nombre de host/Subdominio | Dominio canónico |
| --- | --- |
| www |contoso.cloudapp.net |

> [!NOTE]
> Los visitantes de **www\.contoso.com** no verán nunca el verdadero host (contoso.cloudapp.net), por lo que el usuario final no percibirá el proceso de reenvío.
> 
> El ejemplo anterior solo se aplica al tráfico en el subdominio **www** . Puesto que no puede usar caracteres comodín con registros CNAME, debe crear un CNAME para cada dominio/subdominio. Si desea dirigir el tráfico desde subdominios, como *.contoso.com, a su dirección cloudapp.net, puede configurar una entrada **Redirección de URL** o **Desvío de URL en la configuración de DNS** o crear un registro D.

## <a name="add-an-a-record-for-your-custom-domain"></a>Agregar un registro D para el dominio personalizado
Para crear un registro D, primero debe buscar la dirección IP virtual de su servicio en la nube. A continuación, agregue una entrada en la tabla DNS para el dominio personalizado mediante las herramientas proporcionadas por el registrador. Cada registrador dispone de un método similar pero ligeramente distinto de especificación de un registro D. Sin embargo, los conceptos son los mismos.

1. Use uno de los siguientes métodos para obtener la dirección IP de su servicio en la nube.

   * Inicie sesión en [Azure Portal], seleccione el servicio en la nube, busque la sección **Información general** y, luego, busque la entrada **Direcciones IP públicas**.

       ![sección de vista rápida que muestra la IP virtual][vip]

       **OR**
   * Instale y configure [Azure Powershell](/powershell/azure/)y, luego, use el siguiente comando:

       ```powershell
       get-azurevm -servicename yourservicename | get-azureendpoint -VM {$_.VM} | select Vip
       ```

     Guarde la dirección IP, debido a que la necesitará cuando cree un registro D.
2. Inicie sesión en el sitio web del registrador DNS y vaya a la página de administración de DNS. Busque los vínculos o áreas del sitio etiquetados como **Nombre de dominio**, **DNS** o **Administración del servidor de nombres**.
3. Ahora busque el lugar en el que puede seleccionar o especificar los registros D. Es posible que tenga que seleccionar el tipo de registro de un menú desplegable o ir a una página de configuración avanzada.
4. Seleccione o especifique el dominio o subdominio que usará el registro D. Por ejemplo, seleccione **www** si desea crear un alias para **www\.customdomain.com**. Si desea crear una entrada de comodín para todos los subdominios, especifique '*****'. De esta forma, se incluirán todos los subdominios como **mail.customdomain.com**, **login.customdomain.com** y **www\.customdomain.com**.

    Si desea crear un registro D para el dominio raíz, puede enumerarse como el símbolo " **\@** " en las herramientas de DNS del registrador.
5. Especifique la dirección IP del servicio en la nube en el campo proporcionado. De esta forma, se asocia la entrada del dominio utilizada en el registro D a la dirección IP de la implementación del servicio en la nube.

Por ejemplo, el siguiente registro D desvía todo el tráfico de **contoso.com** a **137.135.70.239**, la dirección IP de su aplicación implementada:

| Nombre de host/Subdominio | Dirección IP |
| --- | --- |
| \@ |137.135.70.239 |

En este ejemplo se crea un registro D para el dominio raíz. Si desea crear una entrada de comodín para incluir todos los subdominios, debe especificar '*****' como subdominio.

> [!WARNING]
> Las direcciones IP en Azure son dinámicas de forma predeterminada. Probablemente querrá usar una [dirección IP reservada](../virtual-network/virtual-networks-reserved-public-ip.md) para asegurarse de que la dirección IP no cambia.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
* [Administración de Cloud Services](cloud-services-how-to-manage-portal.md)
* [Asignación del contenido de la red CDN a un dominio personalizado](../cdn/cdn-map-content-to-custom-domain.md)
* [Configuración general de su servicio en la nube](cloud-services-how-to-configure-portal.md).
* Obtenga información sobre cómo [implementar un servicio en la nube](cloud-services-how-to-create-deploy-portal.md).
* Configure los [certificados TLS/SSL](cloud-services-configure-ssl-certificate-portal.md).

[Expose Your Application on a Custom Domain]: #access-app
[Add a CNAME Record for Your Custom Domain]: #add-cname
[Expose Your Data on a Custom Domain]: #access-data
[VIP swaps]: cloud-services-how-to-manage-portal.md#how-to-swap-deployments-to-promote-a-staged-deployment-to-production
[Create a CNAME record that associates the subdomain with the storage account]: #create-cname
[Azure Portal]: https://portal.azure.com
[vip]: ./media/cloud-services-custom-domain-name-portal/csvip.png
[csurl]: ./media/cloud-services-custom-domain-name-portal/csurl.png



