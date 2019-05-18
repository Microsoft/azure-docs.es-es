---
title: 'Restringir el acceso: Azure App Service | Microsoft Docs'
description: Cómo usar restricciones de acceso con Azure App Service
author: ccompy
manager: stefsch
editor: ''
services: app-service\web
documentationcenter: ''
ms.assetid: 3be1f4bd-8a81-4565-8a56-528c037b24bd
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/22/2018
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: de898a7ebb9611f469f42bb23774b3b0a0c2410d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541682"
---
# <a name="azure-app-service-access-restrictions"></a>Restricciones de acceso de Azure App Service #

Restricciones de acceso le permiten definir una lista ordenada de permitir o denegar prioridad que controla el acceso de red a la aplicación. La lista puede incluir direcciones IP o subredes de red Virtual de Azure. Cuando hay una o varias entradas, hay a continuación, implícita "Denegar todo" que existe al final de la lista.

La capacidad de restricciones de acceso funciona con todos los App Service carga de trabajo hospedado incluidos; aplicaciones Web, aplicaciones de API, aplicaciones de Linux, aplicaciones de contenedor de Linux y funciones.

Cuando se realiza una solicitud a la aplicación, la dirección FROM se evalúa con las reglas de la dirección IP en la lista de restricciones de acceso. Si la dirección está en una subred que se configura con los extremos de servicio Microsoft.Web, a continuación, la subred de origen se compara con las reglas de red virtual en la lista de restricciones de acceso. Si la dirección no tiene permitido el acceso según las reglas de la lista, el servicio responde con un código de estado [HTTP 403](https://en.wikipedia.org/wiki/HTTP_403).

La funcionalidad de las restricciones de acceso se implementa en los roles de front-end de App Service, que son de nivel superior de los hosts de trabajo donde se ejecuta el código. Por lo tanto, las restricciones de acceso son eficazmente las ACL de red.

Se llama a la capacidad para restringir el acceso a la aplicación web de Azure Virtual Network (VNet) [los extremos del servicio][serviceendpoints]. Los puntos de conexión de servicio le permiten restringir el acceso a un servicio multiempresa de subredes seleccionadas. Debe habilitarse en el lado de la red así como el servicio que se habilita con. No funciona para restringir el tráfico a las aplicaciones que se hospedan en un entorno de App Service.  Si está en un entorno de App Service, puede controlar el acceso a la aplicación con reglas de direcciones IP.

![flujo de las restricciones de acceso](media/app-service-ip-restrictions/access-restrictions-flow.png)

## <a name="adding-and-editing-access-restriction-rules-in-the-portal"></a>Adición y edición de reglas de restricción de acceso en el portal ##

Para agregar una regla de restricción de acceso a la aplicación, use el menú para abrir **red**>**restricciones de acceso** y haga clic en **configurar restricciones de acceso**

![Opciones de red de App Service](media/app-service-ip-restrictions/access-restrictions.png)  

Desde la interfaz de usuario de las restricciones de acceso, puede revisar la lista de reglas de restricción de acceso definido para la aplicación.

![restricciones de acceso de la lista](media/app-service-ip-restrictions/access-restrictions-browse.png)

La lista mostrará todas las restricciones actuales que están en la aplicación. Si tiene una restricción de red virtual en la aplicación, la tabla mostrará si los extremos de servicio están habilitados para Microsoft.Web. Cuando no hay ninguna restricción definida en la aplicación, la aplicación será accesible desde cualquier lugar.  

Puede hacer clic en **[+] agregar** para agregar una nueva regla de restricción de acceso. Una vez que agregue una regla, entrará en vigor de inmediato. Las reglas se aplican en orden de prioridad, empezando por el número más bajo y en ascenso. Hay una denegación implícita de todo lo que esté en vigor una vez que agregue incluso una sola regla.

![Agregar una regla de restricción de acceso IP](media/app-service-ip-restrictions/access-restrictions-ip-add.png)

Al crear una regla, debe seleccionar Permitir/Denegar y también el tipo de regla. También deben proporcionar el valor de prioridad y está restringiendo el acceso a.  Opcionalmente, puede agregar un nombre y una descripción para la regla.  

Para establecer una dirección IP basada en reglas, seleccione un tipo de IPv4 o IPv6. La notación de dirección IP debe ser CIDR tanto para direcciones IPv4 como IPv6. Para especificar una dirección exacta, puede usar un formato como 1.2.3.4/32, donde los cuatro primeros octetos representan la dirección IP y /32 es la máscara. La notación CIDR IPv4 para todas las direcciones es 0.0.0.0/0. Para más información acerca de la notación CIDR, puede leer [Classless Inter-Domain Routing](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) (Enrutamiento interdominios sin clases). 

![Agregar una regla de restricción de acceso de red virtual](media/app-service-ip-restrictions/access-restrictions-vnet-add.png)

Para restringir el acceso a las subredes seleccionadas, seleccione un tipo de red Virtual. A continuación podrá elegir la suscripción, red virtual y subred que desea permitir o denegar el acceso con. Si los extremos de servicio no ya están habilitados con Microsoft.Web para la subred que seleccionó, automáticamente se habilitará automáticamente a menos que se active la casilla que se pregunta si no desea hacerlo. La situación donde desearía para habilitarlo en la aplicación pero no la subred está estrechamente relacionada si tiene los permisos para habilitar los puntos de conexión de servicio en la subred o no. Si necesita obtener nadie más para habilitar puntos de conexión de servicio en la subred, puede que la aplicación configurada para puntos de conexión de servicio en previsión de la misma que se va a habilitar más adelante en la subred y Active la casilla. 

Los puntos de conexión de servicio no se puede usar para restringir el acceso a las aplicaciones que se ejecutan en un entorno de App Service. Cuando la aplicación está en un entorno de App Service, puede controlar el acceso a la aplicación con las reglas de acceso IP. 

Puede hacer clic en cualquier fila para editar una regla de restricción de acceso existente. Las modificaciones son efectivas inmediatamente, incluidos los cambios en el orden de prioridad.

![Editar una regla de restricción de acceso](media/app-service-ip-restrictions/access-restrictions-ip-edit.png)

Al editar una regla, no se puede cambiar el tipo de una regla de dirección IP y una regla de red Virtual. 

![Editar una regla de restricción de acceso](media/app-service-ip-restrictions/access-restrictions-vnet-edit.png)

Para eliminar una regla, haga clic en los puntos suspensivos **...**  en la regla y, a continuación, haga clic en **quitar**.

![Eliminar regla de restricción de acceso](media/app-service-ip-restrictions/access-restrictions-delete.png)

### <a name="scm-site"></a>Sitio SCM 

Además de ser capaz de controlar el acceso a la aplicación, también puede restringir el acceso al sitio de scm utilizado por la aplicación. El sitio scm es WebDeploy extremo y también en la consola de Kudu. Puede asignar restricciones de acceso al sitio de scm desde la aplicación por separado o usar el mismo conjunto de la aplicación y el sitio de scm. Cuando se activa la casilla para tener las mismas restricciones que la aplicación, todo lo que está en blanco. Si desactiva la casilla, se aplican las configuraciones que tenía anteriormente en el sitio de scm. 

![restricciones de acceso de la lista](media/app-service-ip-restrictions/access-restrictions-scm-browse.png)

## <a name="programmatic-manipulation-of-access-restriction-rules"></a>Manipulación de las reglas de restricción de acceso mediante programación ##

Actualmente no hay ningún CLI o PowerShell para la nueva funcionalidad de las restricciones de acceso, pero se pueden establecer manualmente los valores con una operación PUT en la configuración de la aplicación en el Administrador de recursos. Por ejemplo, puede usar resources.azure.com y editar el bloque ipSecurityRestrictions para agregar el código JSON requerido.

La ubicación de esta información en Resource Manager es:

management.azure.com/subscriptions/**Id. de suscripción**/resourceGroups/**resource groups**/providers/Microsoft.Web/sites/**nombre de aplicación web**/config/web?api-version=2018-02-01

La sintaxis JSON para el ejemplo anterior es:

    "ipSecurityRestrictions": [
      {
        "ipAddress": "131.107.159.0/24",
        "action": "Allow",
        "tag": "Default",
        "priority": 100,
        "name": "allowed access"
      }
    ],

## <a name="function-app-ip-restrictions"></a>Restricciones de IP de la aplicación de función

Las restricciones de IP están disponibles para ambas aplicaciones de función con la misma funcionalidad que los planes de App Service. Habilitación de restricciones de IP, se deshabilitará al editor de código de portal para las direcciones IP no permitido.

[Más información aquí](../azure-functions/functions-networking-options.md#inbound-ip-restrictions).


<!--Links-->
[serviceendpoints]: https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview
