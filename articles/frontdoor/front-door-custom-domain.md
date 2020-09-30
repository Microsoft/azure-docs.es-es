---
title: 'Tutorial: Adición de un dominio personalizado a la configuración de Azure Front Door'
description: En este tutorial, aprenderá a incorporar un dominio personalizado a Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: ee9a883cbd69826e30d6f2416d588792a8c17b1c
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89648802"
---
# <a name="tutorial-add-a-custom-domain-to-your-front-door"></a>Tutorial: Incorporación de un dominio personalizado a Front Door
En este tutorial se muestra cómo agregar un dominio personalizado a Front Door. Si usa Azure Front Door para la entrega de aplicaciones, es necesario un dominio personalizado si desea que su nombre de dominio se vea en la solicitud de usuario final. El hecho de tener un nombre de dominio visible puede ser cómodo para sus clientes y útil con fines de personalización de marca.

Después de crear una instancia de Front Door, el host de front-end predeterminado, que es un subdominio de `azurefd.net`, se incluye en la dirección URL para entregar el contenido de Front Door desde el back-end de forma predeterminada (por ejemplo, https:\//contoso.azurefd.net/activeusers.htm). Para su comodidad, Azure Front Door permite asociar un dominio personalizado al host predeterminado. Con esta opción, entrega el contenido con un dominio personalizado en la dirección URL, en lugar de un nombre de dominio propiedad de Front Door (por ejemplo, https:\//www.contoso.com/photo.png). 

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Crear un registro DNS de CNAME
> - Asociar el dominio personalizado a Front Door.
> - Comprobar el dominio personalizado

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Front Door **no**admiten dominios personalizados con caracteres [Punycode](https://en.wikipedia.org/wiki/Punycode). 

## <a name="prerequisites"></a>Prerrequisitos

* Para poder completar los pasos de este tutorial, primero debe crear una instancia de Front Door. Para más información, consulte [Inicio rápido: Cree una instancia de Front Door](quickstart-create-front-door.md).

* Si no dispone ya de un dominio personalizado, primero debe adquirir uno con un proveedor de dominios. Por ejemplo, vea [Compra de un nombre de dominio personalizado](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain).

* Si usa Azure para hospedar sus [dominios DNS](https://docs.microsoft.com/azure/dns/dns-overview), debe delegar el sistema de nombres de dominio (DNS) del proveedor de dominios en una instancia de Azure DNS. Para más información, vea [Delegación de un dominio en DNS de Azure](https://docs.microsoft.com/azure/dns/dns-delegate-domain-azure-dns). Por el contrario, si utiliza un proveedor de dominios para controlar su dominio DNS, diríjase a [Creación de un registro DNS de CNAME](#create-a-cname-dns-record).


## <a name="create-a-cname-dns-record"></a>Creación de un registro DNS de CNAME

Para poder usar un dominio personalizado con Front Door, antes hay que crear un registro de nombre canónico (CNAME) con su proveedor de dominios para señalar a su host de front-end predeterminado de Front Door (como por ejemplo, contoso.azurefd.net). Un registro CNAME es un tipo de registro de DNS que asigna un nombre de dominio de origen a un nombre de dominio de destino. En el caso de Azure Front Door, el nombre del dominio de origen es su nombre de dominio personalizado, mientras que el nombre del dominio de destino es el nombre de host predeterminado de Front Door. Una vez que Front Door compruebe el registro CNAME que cree, el tráfico dirigido al dominio personalizado de origen (como www\.contoso.com) se enruta al host del front-end predeterminado de la instancia de Front Door del destino especificado (como contoso.azurefd.net). 

Un dominio personalizado y su subdominio no se pueden asociar con más de una instancia de Front Door a la vez. Sin embargo, se pueden utilizar diferentes subdominios del mismo dominio personalizado para distintas instancias de Front Door. Para ello es preciso usar varios registros CNAME. También se puede asignar un dominio personalizado con diferentes subdominios a la misma instancia de Front Door.


## <a name="map-the-temporary-afdverify-subdomain"></a>Asignar el subdominio afdverify temporal

Al asignar un dominio existente que esté en producción, existen consideraciones especiales. Mientras se registra un dominio personalizado en Azure Portal, se puede producir un breve período de inactividad en el dominio. Para evitar la interrupción del tráfico web, en primer lugar es preciso asignar el dominio personalizado al host de front-end predeterminado de Front Door con el subdominio afdverify de Azure para crear una asignación de CNAME temporal. Con este método, los usuarios pueden acceder a un dominio sin interrupción mientras se realiza la asignación de DNS.

Por el contrario, si es la primera vez que usa un dominio personalizado y no se ejecuta tráfico de producción en él, puede asignar directamente el dominio personalizado a Front Door. Diríjase a [Asignación del dominio personalizado permanente](#map-the-permanent-custom-domain).

Para crear un registro CNAME con el subdominio afdverify:

1. Inicie sesión en el sitio web del proveedor de dominios de su dominio personalizado.

2. Busque la página para administrar registros DNS mediante la consulta de la documentación del proveedor o busque las áreas del sitio web con las etiquetas **Nombre de dominio**, **DNS** o **Name Server Management** (Administración del servidor de nombres). 

3. Cree una entrada de registro CNAME para el dominio personalizado y rellene los campos como se muestra en la tabla siguiente (los nombres de campo pueden variar):

    | Source                    | Tipo  | Destination                     |
    |---------------------------|-------|---------------------------------|
    | afdverify. www.contoso.com | CNAME | afdverify.contoso.azurefd.net |

    - Origen: escriba un nombre de dominio personalizado, incluido el subdominio afdverify, con el siguiente formato: afdverify. _&lt;nombre de dominio personalizado&gt;_ . Por ejemplo, afdverify. www.contoso.com.

    - Escriba:  Escriba *CNAME*.

    - Destino: especifique el host de front-end de Front Door predeterminado, incluido el subdominio afdverify, con el siguiente formato: afdverify. _&lt;nombre de punto de conexión&gt;_ .azurefd.net. Por ejemplo, afdverify.contoso.azurefd.net.

4. Guarde los cambios.

Por ejemplo, este es el procedimiento para el registrador de dominios GoDaddy:

1. Inicie sesión y seleccione el dominio personalizado que desea usar.

2. En la sección Domains (Dominio)º, seleccione **Manage all** (Administrar todos) y, después, seleccione **DNS** | **Manage Zones** (Administrar Zonas).

3. En **Domain Name** (Nombre de dominio), escriba el dominio personalizado y seleccione **Search** (Buscar).

4. En la página **DNS Management** (Administración de SNS), seleccione **Add** (Agregar) y, después, seleccione **CNAME** en la lista **Type** (Tipo).

5. Complete los siguientes campos de la entrada de CNAME:

    - Escriba:  Deje seleccionado *CNAME*.

    - Host: escriba el subdominio del dominio personalizado que va a usar, incluido el nombre de subdominio afdverify. Por ejemplo, afdverify. www.

    - Points to (Apunta a): escriba el nombre del host de front-end de Front Door predeterminado, incluido el nombre de subdominio afdverify. Por ejemplo, afdverify.contoso.azurefd.net. 

    - TTL: Deje seleccionada la opción *una hora*.

6. Seleccione **Guardar**.
 
    La entrada de CNAME se agrega a la tabla de registros DNS.


## <a name="associate-the-custom-domain-with-your-front-door"></a>Asociación del dominio personalizado a Front Door

Una vez que haya registrado un dominio personalizado, puede agregarlo a Front Door.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya a la instancia de Front Door que contenga el host de front-end que desea asignar a un dominio personalizado.
    
2. En la página **Diseñador de Front Door**, haga clic en "+" para agregar un dominio personalizado.
    
3. Especifique **Dominio personalizado**. 

4. En **Host de front-end**, el host de front-end que se usa como dominio de destino de su registro CNAME se rellena previamente y se deriva de Front Door: *&lt;nombre de host predeterminado&gt;* .azurefd.NET. No se puede modificar.

5. Para **Nombre de host personalizado**, escriba el dominio personalizado, incluido el subdominio, que se usará como el dominio de origen del registro CNAME. Por ejemplo, www\.contoso.com o cdn.contoso.com. No utilice el nombre de subdominio afdverify.

6. Seleccione **Agregar**.

   Azure comprueba que el registro CNAME existe para el nombre de dominio personalizado que ha especificado. Si el registro CNAME es correcto, el dominio personalizado se valida.

>[!WARNING]
> **Debe** asegurarse de que todos los hosts de front-end (incluidos los dominios personalizados) de Front Door tienen una regla de enrutamiento con una ruta de acceso predeterminada ('/\*') asociada. Es decir, entre todas las reglas de enrutamientos debe haber al menos una para cada uno de los hosts de front-end que se definen en la ruta de acceso predeterminada ('/\*'). Si no la hay, puede darse el caso de que el tráfico del usuario final no se enrute correctamente.

## <a name="verify-the-custom-domain"></a>Comprobación del dominio personalizado

Después de haber completado el registro del dominio personalizado, compruebe que hace referencia al host de front-end de Front Door personalizado.
 
En el explorador, vaya a la dirección del archivo usando el dominio personalizado. Por ejemplo, si el dominio personalizado es robotics.contoso.com, la dirección URL al archivo almacenado en la caché sería similar a la siguiente: http:\//robotics.contoso.com/my-public-container/my-file.jpg. Compruebe que el resultado es el mismo que cuando accede a Front Door directamente en *&lt;host de Front Door&gt;* .azurefd.net.


## <a name="map-the-permanent-custom-domain"></a>Asignación de un dominio personalizado permanente

Si ha comprobado que el subdominio afdverify se ha asignado correctamente a Front Door (o si utiliza un nuevo dominio personalizado que no esté en producción), puede asignar el dominio personalizado directamente al host de Front Door predeterminado.

Para crear un registro CNAME para un dominio personalizado:

1. Inicie sesión en el sitio web del proveedor de dominios de su dominio personalizado.

2. Para encontrar la página de administración de los registros DNS, consulte la documentación del proveedor o busque las áreas del sitio web con la etiqueta **Nombre de dominio**, **DNS** o **Name Server Management** (Administración del servidor de nombres). 

3. Cree una entrada de registro CNAME para el dominio personalizado y rellene los campos como se muestra en la tabla siguiente (los nombres de campo pueden variar):

    | Source          | Tipo  | Destination           |
    |-----------------|-------|-----------------------|
    | <www.contoso.com> | CNAME | contoso.azurefd.net |

   - Origen: escriba un nombre de dominio personalizado (por ejemplo, www\.contoso.com).

   - Escriba:  Escriba *CNAME*.

   - Destino: especifique el host de front-end de Front Door predeterminado. Debe tener el siguiente formato: _&lt;nombre de host&gt;_ .azurefd.net. Por ejemplo, contoso.azurefd.net.

4. Guarde los cambios.

5. Si previamente ha creado un registro CNAME de un subdominio afdverify temporal, elimínelo. 

6. La primera vez que use este dominio personalizado en producción, siga los pasos de [Asociación del dominio personalizado a Front Door](#associate-the-custom-domain-with-your-front-door) y [Comprobación del dominio personalizado](#verify-the-custom-domain).

Por ejemplo, este es el procedimiento para el registrador de dominios GoDaddy:

1. Inicie sesión y seleccione el dominio personalizado que desea usar.

2. En la sección Domains (Dominio)º, seleccione **Manage all** (Administrar todos) y, después, seleccione **DNS** | **Manage Zones** (Administrar Zonas).

3. En **Domain Name** (Nombre de dominio), escriba el dominio personalizado y seleccione **Search** (Buscar).

4. En la página **DNS Management** (Administración de SNS), seleccione **Add** (Agregar) y, después, seleccione **CNAME** en la lista **Type** (Tipo).

5. Complete los campos de la entrada de CNAME:

    - Escriba:  Deje seleccionado *CNAME*.

    - Host: escriba el subdominio del dominio personalizado que va a usar. Por ejemplo, www o profile.

    - Points to (Apunta a): escriba el nombre de host predeterminado de Front Door. Por ejemplo, contoso.azurefd.net. 

    - TTL: Deje seleccionada la opción *una hora*.

6. Seleccione **Guardar**.
 
    La entrada de CNAME se agrega a la tabla de registros DNS.

7. Si tiene un registro CNAME de afdverify, seleccione el icono de lápiz que encontrará al lado y, después, seleccione el icono de la papelera.

8. Seleccione **Delete** (Eliminar) para eliminar el registro CNAME.


## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, ha agregado un dominio personalizado a Front Door. Si desea que Front Door deje de estar asociado a un dominio personalizado, siga estos pasos para quitar el dominio personalizado:
 
1. En el diseñador de Front Door, seleccione el punto de conexión con el dominio personalizado que desee quitar.

2. Haga clic en Eliminar en el menú contextual del dominio personalizado.  

   El dominio personalizado se desasocia del punto de conexión.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

* Crear un registro DNS de CNAME
* Asociar el dominio personalizado a Front Door.
* Comprobar el dominio personalizado

Para obtener información sobre cómo habilitar HTTPS para el dominio personalizado, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Habilitar HTTPS para un dominio personalizado](front-door-custom-domain.md)
