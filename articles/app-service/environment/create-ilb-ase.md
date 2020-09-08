---
title: Creación de un ASE de ILB con ARM
description: Aprenda a crear un entorno de App Service con un equilibrador de carga interno (ILB ASE) mediante las plantillas de Azure Resource Manager. Aísle completamente las aplicaciones de Internet.
author: ccompy
ms.assetid: 0f4c1fa4-e344-46e7-8d24-a25e247ae138
ms.topic: quickstart
ms.date: 08/05/2019
ms.author: ccompy
ms.custom: mvc, seodec18
ms.openlocfilehash: f2124dd77e3e5d9828ea457a6bccdf7d1bc05405
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961778"
---
# <a name="create-and-use-an-internal-load-balancer-app-service-environment"></a>Creación y uso de un entorno de una instancia de Azure App Service Environment de Load Balancer 

Azure App Service Environment es una implementación de Azure App Service en una subred de Azure Virtual Network (VNet). Hay dos maneras de implementar una instancia de App Service Environment (ASE): 

- Con una dirección VIP en una dirección IP externa, a la que se suele hacer referencia como instancia externa de ASE.
- Con una dirección VIP en una dirección IP interna, llamada a menudo un ASE con un ILB porque el punto de conexión interno es un equilibrador de carga interno (ILB). 

Este artículo muestra cómo crear un ASE con un ILB. Para obtener información general sobre el ASE, consulte [Introducción a App Service Environment][Intro]. Para obtener información sobre cómo crear un ASE externo, consulte [Creación de un ASE externo][MakeExternalASE].

## <a name="overview"></a>Información general 

Puede implementar un ASE con un punto de conexión accesible por Internet o con una dirección IP en la red virtual. Para establecer la dirección IP en una dirección de red virtual, el ASE debe implementarse con un ILB. Al implementar el ASE con un ILB, debe proporcionar el nombre del ASE. El nombre de la instancia de ASE se usa en el sufijo de dominio para las aplicaciones de ASE.  El sufijo de dominio de ASE de ILB es &lt;nombre de ASE&gt;.appserviceenvironment.net. Las aplicaciones que se crean en una instancia de ASE de ILB no se colocan en el DNS público. 

Con las versiones anteriores de la instancia de ASE de ILB era necesario proporcionar un sufijo de dominio y un certificado predeterminado para las conexiones HTTPS. El sufijo de dominio ya no se recopila durante la creación de la instancia de ASE de ILB ni tampoco se recopila ya un certificado predeterminado. Ahora, cuando se crea una instancia de ASE de ILB, Microsoft proporciona el certificado predeterminado y el explorador confía en él. Todavía puede establecer nombres de dominio personalizados en aplicaciones de la instancia de ASE y establecer certificados en esos nombres de dominio personalizados. 

Con una instancia de ASE de ILB, puede hacer cosas como las siguientes:

-   Hospedar aplicaciones de intranet de forma segura en la nube a las que se accede a través de una conexión de sitio a sitio o ExpressRoute
-   Proteger aplicaciones con un dispositivo WAF
-   Hospedar aplicaciones en la nube que no se muestran en los servidores DNS públicos.
-   Crear aplicaciones de back-end aisladas de Internet con las que pueden integrarse de forma segura sus aplicaciones de front-end.

### <a name="disabled-functionality"></a>Funcionalidad deshabilitada ###

Sin embargo, cuando utilice un ASE con un ILB, no podrá realizar algunas operaciones:

-   Use SSL basada en IP.
-   Asigne direcciones IP a aplicaciones específicas.
-   Compre y use un certificado con una aplicación a través de Azure Portal. Puede obtener certificados directamente en una entidad de certificación y usarlos con sus aplicaciones. No puede obtenerlas a través de Azure Portal.

## <a name="create-an-ilb-ase"></a>Creación de un ASE con un ILB ##

Pasos para crear un ASE con un ILB:

1. En Azure Portal, seleccione **Crear un recurso** > **Web** > **App Service Environment**.

2. Seleccione su suscripción.

3. Seleccione o cree un grupo de recursos.

4. Escriba el nombre de su instancia de App Service Environment.

5. Seleccione el tipo de IP virtual Interno.

    ![Creación de ASE](media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase.png)

> [!NOTE]
> El nombre de App Service Environment no debe tener más de 37 caracteres.

6. Selección de redes

7. Seleccione o cree una red Virtual. Si crea una red virtual aquí, se definirá con un intervalo de direcciones de 192.168.250.0/23. Para crear una red virtual con un intervalo de direcciones diferente o en otro grupo de recursos distinto de ASE, use el portal de creación de Azure Virtual Network. 

8. Seleccione o cree una subred vacía. Si quiere seleccionar una subred, debe estar vacía y no debe ser delegada. El tamaño de la subred no se puede cambiar una vez creada la instancia de ASE. Recomendamos un tamaño de `/24`, que tiene 256 direcciones y puede controlar los ASE de tamaño máximo y cualquier escala necesaria. 

    ![Redes de ASE][1]

7. Seleccione **Revisar y crear** y, luego, **Crear**.


## <a name="create-an-app-in-an-ilb-ase"></a>Creación de una aplicación en un ASE con un ILB ##

Crea una aplicación en un ASE con un ILB del mismo modo que crea una aplicación en un ASE normalmente.

1. En Azure Portal, seleccione **Create un recurso** > **Web** > **Web App**.

1. Escriba el nombre de la aplicación.

1. Seleccione la suscripción.

1. Seleccione o cree un grupo de recursos.

1. Seleccione la publicación, la pila del entorno en tiempo de ejecución y el sistema operativo.

1. Seleccione una ubicación que sea una instancia de ASE de ILB existente.  También puede crear una instancia de ASE durante la creación de la aplicación mediante la selección de un plan de App Service aislado. Si quiere crear una instancia de ASE, seleccione la región en la que quiere crearla.

1. Seleccione o cree un plan de App Service. 

1. Seleccione **Revisar y crear** y, cuando esté listo, seleccione **Crear**.

### <a name="web-jobs-functions-and-the-ilb-ase"></a>Trabajos web, Functions y ASE de ILB 

Functions y los trabajos web se admiten en un ASE de ILB, pero para que el portal funcione con ellos, debe tener acceso a la red para el sitio de SCM.  Esto significa que el explorador debe estar en un host que se encuentre en la red virtual o conectado a ella. Si la instancia de ASE de ILB tiene un nombre de dominio que no finaliza en *appserviceenvironment.net*, deberá conseguir que el explorador confíe en el certificado HTTPS que se usa en el sitio SCM.

## <a name="dns-configuration"></a>Configuración de DNS 

Cuando se utiliza una dirección VIP externa, Azure se encarga de administrar el DNS. Todas las aplicaciones que cree en el ASE se agregan automáticamente a DNS de Azure, que es un DNS público. En un ASE con un ILB tiene que administrar su propio DNS. El sufijo de dominio usado con una instancia de ASE de LIB depende del nombre de dicha instancia. El sufijo de dominio es *&lt;nombre de ASE&gt;.appserviceenvironment.net*. La dirección IP de ILB se encuentra en el portal, en **Direcciones IP**. 

Para configurar el DNS:

- Cree una zona para *&lt;nombre de ASE&gt;.appserviceenvironment.net*.
- Cree un registro D en esa zona que apunte * a la dirección IP de ILB.
- Cree un registro D en esa zona que apunte a la dirección IP de ILB.
- Cree una zona en *&lt;nombre de ASE&gt;.appserviceenvironment.net* llamada SCM.
- Cree un registro D en la zona SCM que apunte * a la dirección IP de ILB.

## <a name="publish-with-an-ilb-ase"></a>Publicación con un ASE con un ILB

Para cada aplicación que se cree, hay dos puntos de conexión. En una instancia de ASE de ILB, tiene *&lt;nombre de la aplicación&gt;.&lt;dominio de ASE de ILB&gt;* y *&lt;nombre de la aplicación&gt;.scm.&lt;dominio de ASE de ILB&gt;* . 

El nombre del sitio SCM le lleva a la consola de Kudu, denominada **Advanced Portal** dentro de Azure Portal. La consola de Kudu le permite visualizar las variables de entorno, explorar el disco, utilizar una consola y mucho más. Para más información, consulte [Consola de Kudu para Azure App Service][Kudu]. 

Los sistemas de CI basados en Internet, como GitHub y Azure DevOps, seguirán funcionando con un ASE de ILB si se puede acceder al agente de compilación a través de Internet y este está en la misma red que ASE de ILB. Por tanto, en el caso de Azure DevOps, si el agente de compilación se crea en la misma red virtual que el ASE de ILB (también puede usar una red virtual diferente), podrá extraer el código del repositorio Git de Azure DevOps e implementarlo en el ASE de ILB. Si no desea crear su propio agente de compilación, deberá usar un sistema de CI que use un modelo de extracción como Dropbox.

Los puntos de conexión de publicación para las aplicaciones en un ASE con un ILB usan el dominio con el que se creó el ASE con un ILB. Este dominio aparece en el perfil de publicación de la aplicación y en la hoja del portal de la aplicación (en **Información general** > **Información esencial** y también en **Propiedades**). Si tiene una instancia de ASE de ILB con el sufijo de dominio *&lt;nombre de ASE&gt;.appserviceenvironment.net* y una aplicación llamada *mytest*, use *mytest.&lt;nombre de ASE&gt;.appserviceenvironment.net* para FTP y *mytest.scm.contoso.net* para la implementación web.

## <a name="configure-an-ilb-ase-with-a-waf-device"></a>Configuración de una instancia de ASE de ILB con un dispositivo WAF ##

Puede combinar un dispositivo de firewall de aplicaciones web (WAF) con la instancia de ASE de ILB para exponer solo las aplicaciones que quiera a Internet y el resto mantenerlas accesibles desde la red virtual. Esta estrategia le permite crear aplicaciones seguras de varios niveles entre otras cosas.

Para más información sobre cómo configurar la instancia de ASE de ILB con un dispositivo WAF, consulte [Configuración de un firewall de aplicaciones web para App Service Environment][ASEWAF]. En este artículo se muestra cómo usar una aplicación virtual Barracuda con su ASE. Otra opción es utilizar Azure Application Gateway. Application Gateway utiliza las reglas de núcleo de OWASP para proteger cualquier aplicación que se encuentre detrás de él. Para más información sobre Application Gateway, consulte [Introducción al firewall de aplicaciones web de Azure][AppGW].

## <a name="ilb-ases-made-before-may-2019"></a>Instancias de ASE de LIB creadas antes de mayo de 2019

Con las instancias de ASE de ILB que se crearon antes de mayo de 2019, había que establecer el sufijo de dominio durante la creación de ASE. También era necesario cargar un certificado predeterminado que se basaba en dicho sufijo de dominio. Además, con una instancia antigua de ASE de ILB, no puede realizar el inicio de sesión único en la consola de Kudu con las aplicaciones de dicha instancia. Al configurar DNS para una instancia de este tipo, deberá establecer el registro D comodín en una zona que coincida con el sufijo de dominio. 

## <a name="get-started"></a>Introducción ##

* Para empezar a trabajar con instancias de App Service Environment, consulte [Introducción a App Service Environment][Intro]. 

<!--Image references-->
[1]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-network.png
[2]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-webapp.png
[5]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-ipaddresses.png

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
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[customdomain]: ../app-service-web-tutorial-custom-domain.md
[linuxapp]: ../overview.md#app-service-on-linux