---
title: Creación de una aplicación web en ASE v1
description: Aprenda a crear aplicaciones web en una instancia de App Service Environment v1 Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: ccompy
ms.assetid: 983ba055-e9e4-495a-9342-fd3708dcc9ac
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: d0ac8e0cf187d1a6a329f2c7332c9ec37018d4cf
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962526"
---
# <a name="create-a-web-app-in-an-app-service-environment-v1"></a>Creación de una aplicación web en una instancia de App Service Environment v1

> [!NOTE]
> Este artículo trata sobre App Service Environment v1.  Hay una versión más reciente de App Service Environment que resulta más fácil de usar y se ejecuta en una infraestructura más eficaz. Para aprender más sobre la nueva versión, consulte [Introducción a App Service Environment](intro.md).
> 

## <a name="overview"></a>Información general
Este tutorial muestra cómo crear aplicaciones web y planes de App Service en una instancia de [App Service Environment v1](app-service-app-service-environment-intro.md) (ASE). 

> [!NOTE]
> Si quiere obtener información sobre cómo crear una aplicación web pero no necesita hacerlo en un entorno de App Service, consulte [Creación de una aplicación web de .NET](../quickstart-dotnetcore.md) o uno de los tutoriales relacionados para otros lenguajes y marcos.
> 
> 

## <a name="prerequisites"></a>Requisitos previos
En este tutorial se supone que ha creado un entorno de App Service. Si no es así, consulte [Creación de un entorno de App Service](app-service-web-how-to-create-an-app-service-environment.md). 

## <a name="create-a-web-app"></a>Creación de una aplicación web
1. En [Azure Portal](https://portal.azure.com/), haga clic en **Crear un recurso > Web y móvil > Aplicación web**. 
   
    ![Captura de pantalla que muestra dónde crear una aplicación web en Azure Portal.][1]
2. Seleccione su suscripción.  
   
    Si tiene varias suscripciones, tenga en cuenta que para crear una aplicación en el entorno de App Service, debe usar la misma suscripción que usó para crear el entorno. 
3. Seleccione o cree un grupo de recursos.
   
    *Los grupos de recursos* le permiten administrar los recursos de Azure relacionados como una unidad y resultan útiles al establecer las *reglas del control de acceso basado en rol (RBAC de Azure)* para las aplicaciones. Para más información, consulte [Información general de Azure Resource Manager][ResourceGroups]. 
4. Seleccione o cree un plan de App Service.
   
    Los *planes de App Service* son conjuntos administrados de aplicaciones web.  Normalmente, cuando se selecciona el precio, el precio que se cobra se aplica al plan de App Service y no a las aplicaciones individuales. En un ASE, paga por las instancias de proceso que se han asignado al ASE en lugar de lo que ha enumerado con su ASP.  Para escalar verticalmente el número de instancias de una aplicación web, escale verticalmente las instancias de su plan de App Service. Esto afecta a todas las aplicaciones web de ese plan.  Algunas características como las ranuras de sitio o la integración de la red virtual también tienen restricciones de cantidad dentro del plan.  Para obtener más información, consulte [Información general sobre los planes de Azure App Service](../overview-hosting-plans.md)
   
    Puede identificar los planes de App Service de su ASE con solo mirar la ubicación que se indica bajo el nombre del plan.  
   
    ![Captura de pantalla que muestra dónde ver los planes de App Service en el ASE.][5]
   
    Si quiere usar un plan de App Service que ya existe en el entorno de App Service, seleccione ese plan. Si quiere crear un nuevo plan de App Service, consulte la sección siguiente de este tutorial, [Creación de un plan de App Service en un entorno de App Service](#createplan).
5. Escriba el nombre de la aplicación web y luego haga clic en **Crear**. 
   
    Si su ASE usa una VIP externa, la dirección URL de una aplicación de un ASE es: [*nombre del sitio*].[*nombre de App Service Environment*].p.azurewebsites.net en lugar de [*nombre del sitio*].azurewebsites.net
   
    Si su ASE utiliza una VIP interna, en la dirección URL de una aplicación en ASE está: [*sitename*]. [*subdominio especificado durante la creación de ASE*]   
    Después de seleccionar ASP durante la creación de ASE, verá la actualización del subdominio debajo de **Nombre**

## <a name="create-an-app-service-plan"></a><a name="createplan"></a> Creación de un plan de App Service
Cuando crea un plan de App Service en un entorno de App Service, sus opciones de trabajo son diferentes dado que no hay trabajos compartidos en un ASE.  Los trabajos que tiene que usar son los que el administrador ha asignado al ASE.  Esto significa que para crear un nuevo plan, el número de trabajos asignados al grupo de trabajo del ASE debe ser superior al número total de instancias en todos los planes que ya existen en ese grupo de trabajo.  Si no tiene suficientes trabajos en su grupo de trabajo del ASE para crear su plan, deberá trabajar con el administrador del ASE para agregarlos.

Otra diferencia con los planes de App Service hospedados en un entorno de App Service es la ausencia de selección de precios.  Cuando tiene un entorno de App Service, paga por los recursos de proceso que usa el sistema y no se le cobra adicionalmente por los planes de ese entorno.  Normalmente, cuando crea un plan de App Service, selecciona el plan de precios que determina su facturación.  Un entorno de App Service es esencialmente una ubicación privada donde puede crear contenido.  Se paga por el entorno y no pro hospedar el contenido.

Las siguientes instrucciones muestran cómo crear un plan de App Service mientras crea una aplicación web, como se explica en la sección anterior de este tutorial.

1. Haga clic en **Crear nuevo** en la interfaz de usuario de selección del plan y proporcione un nombre para el plan, tal como haría normalmente fuera de un ASE.
2. Seleccione el ASE que quiere usar con su selector de ubicación.
   
    Como un entorno de App Service es básicamente una ubicación de implementación privada, se muestra en Ubicación. 
   
    ![Captura de pantalla del selector de ubicación para seleccionar el ASE que desea.][2]
   
    Después de seleccionar un ASE en el selector de ubicación, se actualiza la interfaz de usuario de creación del plan de App Service.  La ubicación muestra ahora el nombre del sistema ASE y la región en la que está, y el selector de plan de precios se sustituye por un selector de grupos de trabajo.  
   
    ![Captura de pantalla de los detalles del sistema ASE después de seleccionar el ASE en el selector de ubicación.][3]

### <a name="selecting-a-worker-pool"></a>Selección de un grupo de trabajo
Normalmente, en Azure App Service y fuera de un entorno de App Service, hay tres tamaños de proceso disponibles con la selección de un plan de precios dedicado.  De igual forma, puede definir hasta tres grupos de trabajo para un ASE y especificar el tamaño de proceso que se usa para ese grupo de trabajo.  Para los inquilinos del ASE, eso significa que en lugar de seleccionar un plan de precios con el tamaño de proceso de su plan de App Service, se selecciona lo que se conoce como un *grupo de trabajo*.  

La interfaz de usuario de selección del grupo de trabajo muestra el tamaño de proceso usado para ese grupo de trabajo debajo del nombre.  La cantidad disponible se refiere al número de instancias de proceso que están disponibles para usarse en ese grupo.  El grupo total puede tener en realidad un número de instancias superior a este, pero este valor se refiere simplemente al número que no está en uso.  Si necesita ajustar el entorno de App Service para agregar más recursos de proceso, consulte [Configuración del entorno de App Service](app-service-web-configure-an-app-service-environment.md).

![Captura de pantalla del panel grupo de trabajo, donde se pueden seleccionar grupos de trabajo para el ASE.][4]

En este ejemplo puede ver que solo hay dos grupos de trabajo disponibles. Eso se debe a que el administrador de ASE solamente asigna hosts en esos dos grupos de trabajo.  El tercero se mostraría cuando haya máquinas virtuales asignadas a él.  

## <a name="after-web-app-creation"></a>Después de la creación de la aplicación web
Existen algunas consideraciones que se deben tener en cuenta para ejecutar aplicaciones web y administrar planes de App Service en un ASE.  

Como se indicó anteriormente, el propietario del ASE es responsable del tamaño del sistema y, por tanto, también es responsable de garantizar que haya capacidad suficiente para hospedar los planes de App Service deseado. Si no hay trabajos disponibles, no podrá crear su plan de App Service.  Esto también se cumple para escalar verticalmente la aplicación web.  Si necesita más instancias, el administrador del entorno de App Service debe agregar más trabajos.

Después de crear la aplicación web y el plan de App Service, es una buena idea para escalarlos verticalmente.  En un ASE siempre debe haber al menos dos instancias del plan de App Service para proporcionar tolerancia a errores en las aplicaciones.  El escalado de un plan de App Service en un ASE es igual que mediante la interfaz de usuario.  Para obtener más información sobre el escalado, consulte [Escalado de una aplicación web en un entorno de App Service](app-service-web-scale-a-web-app-in-an-app-service-environment.md)

<!--Image references-->
[1]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createaspnewwebapp.png
[2]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createasplocation.png
[3]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createaspselected.png
[4]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createaspworkerpool.png
[5]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/selectaspinase.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[Appserviceplans]: ../overview-hosting-plans.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoScale]: app-service-web-scale-a-web-app-in-an-app-service-environment.md
[HowtoConfigureASE]: app-service-web-configure-an-app-service-environment.md
[ResourceGroups]: ../../azure-resource-manager/management/overview.md
[AzurePowershell]: /powershell/azure/?view=azps-3.8.0