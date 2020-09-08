---
title: Creación de un ASE externo
description: Aprenda a crear un entorno de App Service con una aplicación en él o a crear una instancia de App Service Environment independiente (vacía).
author: ccompy
ms.assetid: 94dd0222-b960-469c-85da-7fcb98654241
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 03bc41a0159253359b4f87fd35d4de055b294cc4
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961795"
---
# <a name="create-an-external-app-service-environment"></a>Creación de una instancia externa de App Service Environment

Azure App Service Environment es una implementación de Azure App Service en una subred de Azure Virtual Network (VNet).

> [!NOTE]
> Cada instancia de App Service Environment tiene una IP virtual (VIP) que puede usarse para ponerse en contacto con App Service Environment.

Hay dos maneras de implementar una instancia de App Service Environment (ASE):

- Con una dirección VIP en una dirección IP externa, a la que se suele hacer referencia como instancia externa de ASE.
- Con la dirección VIP en una dirección IP interna, llamada a menudo instancia de ASE con un ILB porque el punto de conexión interno es un equilibrador de carga interno (ILB).

En este artículo se muestra cómo crear un ASE externo. Para información general sobre el ASE, consulte [Introducción a App Service Environment][Intro]. Para información sobre cómo crear una instancia de ASE con un ILB, consulte [Creación y uso de una instancia de ASE con un ILB][MakeILBASE].

## <a name="before-you-create-your-ase"></a>Antes de crear su ASE

Después de crear la instancia de ASE, no puede cambiar los siguientes parámetros:

- Location
- Subscription
- Resource group
- Red virtual usada
- La subred usada
- Tamaño de la subred

> [!NOTE]
> Al seleccionar una VNet y especificar una subred, asegúrese de que es lo suficientemente grande para adaptarse al crecimiento futuro y a las necesidades de escalado. Se recomienda un tamaño de `/24` con doscientas cincuenta y seis direcciones.
>

## <a name="three-ways-to-create-an-ase"></a>Tres formas de crear un ASE

Hay tres formas de crear una instancia de ASE:

- **Al crear un plan de App Service**. Este método crea la instancia de ASE y el plan de App Service en un solo paso.
- **Como una acción independiente**. Este método crea una instancia de ASE independiente, es decir, que no contiene nada. Se trata de un proceso más avanzado de creación de una instancia de ASE. Úselo para crear una instancia de ASE con un ILB.
- **A partir de una plantilla de Azure Resource Manager**. Este método es para usuarios avanzados. Para más información, consulte [Creación de una instancia de ASE mediante el uso de una plantilla][MakeASEfromTemplate].

Una instancia externa de ASE tiene una dirección VIP pública, lo que significa que todo el tráfico HTTP/HTTPS dirigido a las aplicaciones de ASE visita una dirección IP a la que se puede acceder a través de Internet. Una instancia de ASE con un ILB tiene una dirección IP de la subred utilizada por el entorno de ASE. Las aplicaciones hospedadas en una instancia de ASE con un ILB no se exponen directamente a Internet.

## <a name="create-an-ase-and-an-app-service-plan-together"></a>Creación de un ASE y de un plan de App Service juntos

El plan de App Service es un contenedor de aplicaciones. Cuando se crea una aplicación en App Service, seleccione o cree un plan de App Service. Las instancias de App Service Environment contienen planes de App Service, y estos incluyen aplicaciones.

Para compilar una instancia de ASE al crear un plan de App Service:

1. En [Azure Portal](https://portal.azure.com/), seleccione **Crear un recurso** > **Web y móvil** > **Aplicación web**.

    ![Creación de la aplicación web][1]

2. Seleccione su suscripción. La aplicación y la instancia de ASE se crean en las mismas suscripciones.

3. Seleccione o cree un grupo de recursos. Con los grupos de recursos, puede administrar recursos de Azure relacionados como una unidad. Los grupos de recursos también son útiles cuando establece reglas de control de acceso basado en rol para las aplicaciones. Para más información, consulte [Información general de Azure Resource Manager][ARMOverview].

4. Seleccione el sistema operativo (Windows, Linux o Docker). 

5. Seleccione el plan de App Service y después seleccione **Crear nuevo**. Las aplicaciones web Windows y Linux no pueden estar en el mismo plan de App Service, pero sí que pueden estar en el mismo entorno de App Service. 

    ![Nuevo plan de App Service][2]

6. En la lista desplegable **Ubicación**, seleccione la región donde desea crear la instancia de ASE. Si selecciona una instancia de ASE existente, entonces no se crea ninguna. El plan de App Service se crea en la instancia de ASE seleccionada. 

7. Seleccione **Plan de tarifa** y elija una de las SKU de precios **aisladas**. Si elige una tarjeta de SKU **aislada** y una ubicación que no sea una instancia de ASE, significa que se va a crear una instancia de ASE en dicha ubicación. Para iniciar el proceso de creación de una instancia de ASE, haga clic en **Seleccionar**. La SKU **aislada** está disponible solo junto con una instancia de ASE. Tampoco puede utilizar cualquier otra SKU de precios en una instancia de ASE distinta de la **aislada**. 

    ![Selección del plan de tarifa][3]

8. Escriba el nombre del ASE. Este nombre se utiliza en el nombre direccionable de las aplicaciones. Si el nombre de la instancia de ASE es _appsvcenvdemo_, el nombre de dominio es *.appsvcenvdemo.p.azurewebsites.net*. Si crea una aplicación con nombre *mytestapp*, esta es direccionable a mytestapp.appsvcenvdemo.p.azurewebsites.net. No puede usar espacios en blanco en el nombre. Si utiliza caracteres en mayúsculas, el nombre de dominio es la versión en minúsculas total de ese nombre.

    ![Nombre del nuevo plan de App Service][4]

9. Especifique los detalles de redes virtuales de Azure. Seleccione **Crear nuevo** o **Seleccionar existente**. La opción para seleccionar una red virtual existente está disponible solo si tiene una red virtual en la región seleccionada. Si selecciona **Crear nuevo**, escriba un nombre para la red virtual. Se crea una red virtual de Resource Manager con dicho nombre. Utiliza el espacio de direcciones `192.168.250.0/23` en la región seleccionada. Si selecciona **Seleccionar existente**, tiene que:

    a. Seleccionar el bloque de direcciones de red virtual si tiene más de uno.

    b. Escribir un nuevo nombre de subred.

    c. Seleccionar el tamaño de la subred. *Recuerde seleccionar un tamaño de subred suficientemente grande para hospedar el crecimiento futuro de la instancia de ASE.* Se recomienda un tamaño de `/24`, que tiene ciento veintiocho direcciones y puede controlar una instancia de ASE con un tamaño máximo. No se recomienda el tamaño de `/28`, por ejemplo, porque solo tiene dieciséis direcciones disponibles. La infraestructura utiliza al menos siete direcciones, mientras que Redes de Azure utiliza otras cinco. En una subred `/28`, se quedará con un escalado máximo de cuatro instancias de planes de App Service para un ASE externo y solo tres instancias de planes de App Service para un ASE de ILB.

    d. Seleccione el intervalo IP de subred.

10. Seleccione **Crear** para crear la instancia de ASE. Con este proceso también se crea la aplicación y el plan de App Service. Tanto la instancia de ASE como el plan de App Service y la aplicación se encuentran en la misma suscripción y también en el mismo grupo de recursos. Si la instancia de ASE necesita un grupo de recursos independiente, o si necesita una instancia de ASE con un ILB, siga los pasos para crear un ASE por sí mismo.

## <a name="create-an-ase-and-a-linux-web-app-using-a-custom-docker-image-together"></a>Creación de un ASE y una aplicación web Linux con una imagen personalizada de Docker

1. En [Azure Portal](https://portal.azure.com/), **Crear un recurso** > **Web y móvil** > **Web App for Containers.** 

    ![Creación de la aplicación web][7]

1. Seleccione su suscripción. La aplicación y la instancia de ASE se crean en las mismas suscripciones.

1. Seleccione o cree un grupo de recursos. Con los grupos de recursos, puede administrar recursos de Azure relacionados como una unidad. Los grupos de recursos también son útiles cuando establece reglas de control de acceso basado en rol para las aplicaciones. Para más información, consulte [Información general de Azure Resource Manager][ARMOverview].

1. Seleccione el plan de App Service y después seleccione **Crear nuevo**. Las aplicaciones web Windows y Linux no pueden estar en el mismo plan de App Service, pero sí que pueden estar en el mismo entorno de App Service. 

    ![Nuevo plan de App Service][8]

1. En la lista desplegable **Ubicación**, seleccione la región donde desea crear la instancia de ASE. Si selecciona una instancia de ASE existente, entonces no se crea ninguna. El plan de App Service se crea en la instancia de ASE seleccionada. 

1. Seleccione **Plan de tarifa** y elija una de las SKU de precios **aisladas**. Si elige una tarjeta de SKU **aislada** y una ubicación que no sea una instancia de ASE, significa que se va a crear una instancia de ASE en dicha ubicación. Para iniciar el proceso de creación de una instancia de ASE, haga clic en **Seleccionar**. La SKU **aislada** está disponible solo junto con una instancia de ASE. Tampoco puede utilizar cualquier otra SKU de precios en una instancia de ASE distinta de la **aislada**. 

    ![Selección del plan de tarifa][3]

1. Escriba el nombre del ASE. Este nombre se utiliza en el nombre direccionable de las aplicaciones. Si el nombre de la instancia de ASE es _appsvcenvdemo_, el nombre de dominio es *.appsvcenvdemo.p.azurewebsites.net*. Si crea una aplicación con nombre *mytestapp*, esta es direccionable a mytestapp.appsvcenvdemo.p.azurewebsites.net. No puede usar espacios en blanco en el nombre. Si utiliza caracteres en mayúsculas, el nombre de dominio es la versión en minúsculas total de ese nombre.

    ![Nombre del nuevo plan de App Service][4]

1. Especifique los detalles de redes virtuales de Azure. Seleccione **Crear nuevo** o **Seleccionar existente**. La opción para seleccionar una red virtual existente está disponible solo si tiene una red virtual en la región seleccionada. Si selecciona **Crear nuevo**, escriba un nombre para la red virtual. Se crea una red virtual de Resource Manager con dicho nombre. Utiliza el espacio de direcciones `192.168.250.0/23` en la región seleccionada. Si selecciona **Seleccionar existente**, tiene que:

    a. Seleccionar el bloque de direcciones de red virtual si tiene más de uno.

    b. Escribir un nuevo nombre de subred.

    c. Seleccionar el tamaño de la subred. *Recuerde seleccionar un tamaño de subred suficientemente grande para hospedar el crecimiento futuro de la instancia de ASE.* Se recomienda un tamaño de `/24`, que tiene ciento veintiocho direcciones y puede controlar una instancia de ASE con un tamaño máximo. No se recomienda el tamaño de `/28`, por ejemplo, porque solo tiene dieciséis direcciones disponibles. La infraestructura utiliza al menos siete direcciones, mientras que Redes de Azure utiliza otras cinco. En una subred `/28`, se quedará con un escalado máximo de cuatro instancias de planes de App Service para un ASE externo y solo tres instancias de planes de App Service para un ASE de ILB.

    d. Seleccione el intervalo IP de subred.

1.  Seleccione "Configurar contenedor".
    * Escriba el nombre de la imagen personalizada (puede usar Azure Container Registry, Docker Hub y su propio registro privado). Si no desea utilizar su propio contenedor personalizado, puede traer su código y utilizar una imagen integrada con App Service en Linux, utilizando las instrucciones anteriores. 

    ![Configuración del contenedor][9]

1. Seleccione **Crear** para crear la instancia de ASE. Con este proceso también se crea la aplicación y el plan de App Service. Tanto la instancia de ASE como el plan de App Service y la aplicación se encuentran en la misma suscripción y también en el mismo grupo de recursos. Si la instancia de ASE necesita un grupo de recursos independiente, o si necesita una instancia de ASE con un ILB, siga los pasos para crear un ASE por sí mismo.


## <a name="create-an-ase-by-itself"></a>Creación de un ASE por sí mismo

Si crea una instancia de ASE independiente, esta no contendrá nada. Aunque la instancia de ASE esté vacía, seguirá generando un cargo mensual para la infraestructura. Siga estos pasos para crear una instancia de ASE con un ILB o crear una instancia de ASE en su propio grupo de recursos. Después de crear la instancia de ASE, puede crear aplicaciones en ella con el método habitual. Seleccione la nueva instancia de ASE como ubicación.

1. Busque **App Service Environment** en Azure Marketplace o seleccione **Crear un recurso** > **Web y móvil** > **App Service Environment**. 

1. Escriba el nombre de la instancia de ASE. Este nombre se utiliza para las aplicaciones que se crean en la instancia de ASE. Si el nombre es *mynewdemoase*, el nombre del subdominio es *.mynewdemoase.p.azurewebsites.net*. Si crea una aplicación con nombre *mytestapp*, esta es direccionable a mytestapp.mynewdemoase.p.azurewebsites.net. No puede usar espacios en blanco en el nombre. Si utiliza caracteres en mayúsculas, el nombre de dominio es la versión en minúsculas total del nombre. Si usa un ILB, el nombre la instancia de ASE no se usa en su subdominio pero, en su lugar, se indica explícitamente durante la creación de la instancia de ASE.

    ![Nomenclatura de ASE][5]

1. Seleccione su suscripción. Esta suscripción también es la que usan todas las aplicaciones incluidas en la instancia de ASE. No se puede colocar la instancia de ASE en una red virtual que se encuentra en otra suscripción.

1. Seleccione o especifique un nuevo grupo de recursos. El grupo de recursos usado para la instancia de ASE debe ser el mismo que se utiliza para la red virtual. Si selecciona una red virtual existente, la selección del grupo de recursos para su instancia de ASE se actualiza para reflejar la red virtual. *Puede crear una instancia de ASE con un grupo de recursos distinto del grupo de recursos de la red virtual si usa una plantilla de Resource Manager.* Para crear una instancia de ASE a partir de una plantilla, vea [Creación de una instancia de ASE mediante el uso de una plantilla][MakeASEfromTemplate].

    ![Selección de grupo de recursos][6]

1. Seleccione la red virtual y la ubicación. Puede crear una red virtual o seleccionar una existente: 

    * Si selecciona una red virtual nueva, puede especificar un nombre y una ubicación. 
    
    * La nueva red virtual tiene el intervalo de direcciones 192.168.250.0/23 y una subred con nombre predeterminado. La subred se define como 192.168.250.0/24. Solo se puede seleccionar una red virtual de Resource Manager. La selección del **tipo de VIP** determina si es posible acceder directamente a la instancia de ASE desde Internet (externa) o si usa un ILB. Para más información sobre estas opciones, vea [Creación y uso de un entorno de una instancia de Azure App Service Environment de Load Balancer][MakeILBASE]. 

      * Si selecciona **Externo** en **Tipo de VIP**, puede seleccionar la cantidad de direcciones IP externas que crea el sistema con fines de SSL basados en IP. 
    
      * Si selecciona **Interno** en **Tipo de VIP**, debe especificar el dominio que la instancia de ASE utiliza. Puede implementar una instancia de ASE en una red virtual que utiliza intervalos de direcciones públicas o privadas. Para usar una red virtual con un intervalo de direcciones públicas, necesita crear la red virtual con antelación. 
    
    * Si selecciona una red virtual existente, se crea una subred al crear la instancia de ASE. *No puede usar una subred creada previamente en el portal. Puede crear una instancia de ASE con una subred existente si usa una plantilla de Resource Manager.* Para crear una instancia de ASE a partir de una plantilla, vea [Creación de una instancia de ASE mediante el uso de una plantilla][MakeASEfromTemplate].

## <a name="app-service-environment-v1"></a>App Service Environment v1

Todavía puede crear instancias de la primera versión de App Service Environment (ASEv1). Para iniciar ese proceso, busque **App Service Environment v1** en Marketplace. Cree la instancia de ASE de la misma forma que ha creado la instancia de ASE independiente. Cuando termine, la instancia de ASEv1 tendrá dos servidores front-end y dos trabajos. Con ASEv1 necesita administrar los servidores front-end y los trabajos. No se agregan de forma automática al crear los planes de App Service. Los front-end actúan como los puntos de conexión HTTP/HTTPS y envían tráfico a los trabajos. Los trabajos son roles que hospedan las aplicaciones. Puede ajustar la cantidad de servidores front-end y de trabajos después de crear la instancia de ASE. 

Para más información sobre ASEv1, consulte [Introducción a App Service Environment v1][ASEv1Intro]. Para más información sobre el escalado, la administración y la supervisión de ASEv1, consulte [Configuración de App Service Environment][ConfigureASEv1].

<!--Image references-->
[1]: ./media/how_to_create_an_external_app_service_environment/createexternalase-create.png
[2]: ./media/how_to_create_an_external_app_service_environment/createexternalase-aspcreate.png
[3]: ./media/how_to_create_an_external_app_service_environment/createexternalase-pricing.png
[4]: ./media/how_to_create_an_external_app_service_environment/createexternalase-embeddedcreate.png
[5]: ./media/how_to_create_an_external_app_service_environment/createexternalase-standalonecreate.png
[6]: ./media/how_to_create_an_external_app_service_environment/createexternalase-network.png
[7]: ./media/how_to_create_an_external_app_service_environment/createexternalase-createwafc.png
[8]: ./media/how_to_create_an_external_app_service_environment/createexternalase-aspcreatewafc.png
[9]: ./media/how_to_create_an_external_app_service_environment/createexternalase-configurecontainer.png



<!--Links-->
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
[webapps]: ../overview.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md