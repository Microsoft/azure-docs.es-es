---
title: Uso y administración de una instancia de App Service Environment
description: Aprenda a crear, publicar y escalar aplicaciones en Azure App Service Environment. Busque todas las tareas comunes de este artículo.
author: ccompy
ms.assetid: a22450c4-9b8b-41d4-9568-c4646f4cf66b
ms.topic: article
ms.date: 5/10/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 5e3cb07730aafed7d1c339f543e7fb09fe956cab
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961914"
---
# <a name="use-an-app-service-environment"></a>Uso de una instancia de App Service Environment

Un entorno App Service Environment (ASE) es una implementación de Azure App Service en una subred de una instancia de Azure Virtual Network (VNet) de un cliente. Una instancia de ASE se compone de:

- **Servidores front-end**: donde las solicitudes HTTP o HTTPS finalizan en App Service Environment.
- **Trabajos**: los recursos que hospedan las aplicaciones.
- **Base de datos**: contiene información que define el entorno.
- **Almacenamiento**: se usa para hospedar las aplicaciones publicadas del cliente.

Puede implementar una instancia de ASE con una IP virtual (VIP) externa o interna que permita el acceso de las aplicaciones. Una implementación con una VIP externa suele llamarse *ASE externo*. Una implementación con una VIP interna se llama *ASE con un ILB* porque usa un equilibrador de carga interno (ILB). Para más información sobre ASE con un ILB, consulte [Creación y uso de un ASE con un ILB][MakeILBASE].

## <a name="create-an-app-in-an-ase"></a>Creación de una aplicación en un entorno ASE

Para crear una aplicación en un entorno ASE, se usa el mismo proceso que al crearla normalmente, pero con pequeñas diferencias. Al crear un nuevo plan de App Service:

- En lugar de elegir una ubicación geográfica en la que implementar la aplicación, elija un entorno ASE como su ubicación.
- Todos los planes de App Service que se creen en una instancia de ASE solamente deberán tener un plan de tarifa Aislado.

Si no tiene un entorno ASE, puede crear uno según las instrucciones que aparecen en [Creación de un entorno en App Service Environment][MakeExternalASE].

Para crear una aplicación en un entorno ASE:

1. Seleccione **Crear un recurso** > **Web y móvil** > **Aplicación web**.

1. Escriba un nombre para la aplicación. Si ya seleccionó un plan de App Service en ASE, el nombre de dominio de la aplicación refleja el nombre de dominio del entorno ASE:

    ![Selección del nombre de la aplicación][1]

1. Seleccione una suscripción.

1. Especifique un nombre para un grupo de recursos nuevo o seleccione **Usar existente** y seleccione uno en la lista desplegable.

1. Seleccione el sistema operativo.

1. Seleccione un plan de App Service existente en ASE o cree uno nuevo siguiendo estos pasos:

    a. En el menú de la izquierda de Azure Portal, seleccione **Crear un recurso > Aplicación web**.

    b. Seleccione la suscripción.

    c. Seleccione o cree el grupo de recursos.

    d. Escriba el nombre de la aplicación web.

    e. Seleccione **Código** o **DockerContainer**.

    f. Seleccione una pila del entorno en tiempo de ejecución.

    g. Seleccione **Linux** o **Windows**. 

    h. Seleccione la instancia de ASE en la lista desplegable **Región**. 

    i. Seleccione o cree un nuevo plan de App Service. Si va a crear un nuevo plan de App Service, seleccione el tamaño del plan de tarifa **Aislado** que corresponda.

    ![Planes de tarifa aislados][2]

    > [!NOTE]
    > Las aplicaciones Windows y Linux no pueden estar en el mismo plan de App Service, pero sí que pueden estar en el mismo entorno de App Service Environment.
    >

1. Seleccione **Revisar y crear**, asegúrese de que la información es correcta y luego seleccione **Crear**.

## <a name="how-scale-works"></a>Cómo funciona escalar

Cada aplicación de App Service se ejecuta en un plan de App Service. Las instancias de App Service Environment contienen planes de App Service, y estos incluyen aplicaciones. Cuando escala una aplicación, también escala el plan de App Service y todas las aplicaciones de ese mismo plan.

Cuando se escala horizontalmente un plan de App Service, se agrega la infraestructura necesaria de forma automática. Hay un retraso para escalar las operaciones mientras se agrega la infraestructura. Si realiza varias operaciones de escalado una detrás de otra, se atenderá a la primera solicitud de escalado de la infraestructura y las demás se podrán en cola. Cuando finaliza la primera operación de escalado, el resto de las solicitudes de infraestructura se realizarán a la vez. Cuando se agregue la infraestructura, los planes de App Service se asignarán según corresponda. La propia acción de crear un nuevo plan de App Service es ya una operación de escalado porque requiere hardware adicional.

En las instancias multiinquilino de App Service, el escalado es inmediato, ya que hay un grupo de destino disponible al instante para realizarlo. En un entorno ASE, no hay ningún búfer de este tipo y los recursos se asignan según sea necesario.

En ASE, un plan de App Service se puede escalar hasta en 100 instancias. Un entorno de ASE puede tener como máximo un total de 201 instancias entre todos los planes de App Service de ese entorno.

## <a name="ip-addresses"></a>Direcciones IP

App Service puede asignar una dirección IP dedicada a una aplicación. Esta funcionalidad está disponible después de configurar un SSL basado en dirección IP, tal como se describe en [Enlace de un certificado TLS/SSL personalizado a Azure App Service][ConfigureSSL]. En las instancias de ASE con ILB, no se pueden agregar más direcciones IP para usarlas con un protocolo SSL basado en IP.

Con un entorno ASE externo, puede configurar un protocolo SSL basado en IP para la aplicación de la misma manera que en una instancia de App Service con varios inquilinos. En ASE siempre hay una dirección de reserva, hasta un máximo de treinta direcciones IP. Cada vez que usa una, se agrega otra, para que siempre haya una dirección disponible lista para su uso. Se requiere un retraso de tiempo para asignar otra dirección IP. Ese retraso evita la incorporación de direcciones IP en una sucesión rápida.

## <a name="front-end-scaling"></a>Escalado de front-end

Cuando se escala un plan de App Service, se agregan automáticamente trabajos que contribuyen a la operación de escalado. Cada ASE se crea con dos servidores front-end. Los servidores front-end se escalan horizontalmente a un ritmo de un servidor front-end por cada grupo de quince instancias de planes de App Service. Por ejemplo, si tiene tres planes de App Service con cinco instancias cada uno, tendría un total de quince instancias y tres servidores front-end. Si escala a un total treinta instancias, tendrá cuatro servidores front-end. Este patrón continúa a medida que se escala horizontalmente.

El número de servidores front-end que se asignan de forma predeterminada es el adecuado para una carga moderada. Puede reducir la ratio de modo que sea tan pequeña como un servidor front-end por cada cinco instancias. También puede cambiar el tamaño de los servidores front-end. De forma predeterminada, tienen un solo núcleo. En Azure Portal, puede cambiar su tamaño a dos o cuatro núcleos.

Se aplican cargos por cambiar la ratio o el tamaño de los servidores front-end. Para más información, consulte [Precios de Azure App Service][Pricing]. Si quiere mejorar la capacidad de carga de la instancia de ASE, disfrutará de un mejor rendimiento si escala primero a servidores front-end de dos núcleos antes de ajustar ratio de escalado. Al cambiar el tamaño de núcleo de los servidores front-end, se actualizará la instancia de ASE, lo que debe hacerse fuera del horario comercial normal.

Los recursos de Front-End son el punto de conexión HTTP/HTTPS para el entorno ASE. Con la configuración predeterminada de Front-End, el uso de memoria por Front-End se sitúa sistemáticamente en torno al 60 %. La razón principal para escalar los servidores front-end es el uso de CPU, lo que normalmente viene determinado por el tráfico HTTPS.

## <a name="app-access"></a>Acceso a las aplicaciones

En las instancias externas de ASE, el sufijo del dominio que se utiliza para crear aplicaciones es *.&lt;nombreASE&gt;.p.azurewebsites.net*. Por ejemplo, si la instancia de ASE se llama _external-ase_ y la aplicación _contoso_ se hospeda en ese ASE, puede alcanzarla utilizando las siguientes direcciones URL:

- contoso.external-ase.p.azurewebsites.net
- contoso.scm.external-ase.p.azurewebsites.net

Para información sobre cómo crear un entorno ASE externo, consulte [Creación de una instancia externa de App Service Environment][MakeExternalASE].

En una instancia de ASE con ILB, el sufijo del dominio que se utiliza para crear aplicaciones es *.&lt;nombreASE&gt;.appserviceenvironment.net*. Si la instancia de ASE se llama _ilb-ase_ y la aplicación _contoso_ se hospeda en ese ASE, puede alcanzarla en las siguientes direcciones URL:

- contoso.ilb-ase.appserviceenvironment.net
- contoso.scm.ilb-ase.appserviceenvironment.net

Para información sobre cómo crear una instancia de ASE con un ILB, consulte [Creación y uso de una instancia de ASE con un ILB][MakeILBASE].

La dirección URL del SCM se utiliza para obtener acceso a la consola de Kudu o para publicar la aplicación mediante Web Deploy. Para información sobre la consola Kudu, consulte el artículo sobre la [consola Kudu para Azure App Service][Kudu]. La consola Kudu ofrece una UI web para depurar, cargar archivos, editar archivos y mucho más.

### <a name="dns-configuration"></a>Configuración de DNS 

Cuando se usa un ASE externo, las aplicaciones realizadas en el ASE se registran con Azure DNS. En un ASE externo, no hay pasos adicionales para que las aplicaciones estén disponibles públicamente. En un ASE con un ILB tiene que administrar su propio DNS. Puede hacer esto en su propio servidor DNS o con las zonas privadas de Azure DNS.

Para configurar DNS en su propio servidor DNS con el ASE de ILB:

1. Cree una zona para el &lt;nombre de ASE&gt;.appserviceenvironment.net.
1. Cree un registro D en esa zona que apunte * a la dirección IP de ILB.
1. Cree un registro D en esa zona que apunte a la dirección IP de ILB.
1. Cree una zona en el &lt;nombre de ASE&gt;.appserviceenvironment.net denominada SCM.
1. Cree un registro D en la zona SCM que apunte * a la dirección IP de ILB.

Para configurar DNS en las zonas privadas de Azure DNS:

1. Cree una zona privada de Azure DNS denominada &lt;ASE name&gt;.appserviceenvironment.net.
1. Cree un registro D en esa zona que apunte * a la dirección IP de ILB.
1. Cree un registro D en esa zona que apunte a la dirección IP de ILB.
1. cree un registro A en esa zona que apunte *.scm a la dirección IP de ILB.

La configuración de DNS para el sufijo de dominio predeterminado de ASE no restringe las aplicaciones para que solo puedan acceder a ellas esos nombres. Puede establecer un nombre de dominio personalizado sin ninguna validación en las aplicaciones de un ASE con un ILB. Si desea crear una zona denominada *contoso.net*, puede hacerlo y hacer que apunte a la dirección IP de ILB. El nombre de dominio personalizado funciona para las solicitudes de aplicación, pero no para el sitio SCM. El sitio SCM solo está disponible en *&lt;appname&gt;.scm.&lt;asename&gt;.appserviceenvironment.net*. 

La zona denominada *.&lt;asename&gt;.appserviceenvironment.net* es globalmente única. Antes de mayo de 2019, los clientes podían especificar el sufijo de dominio del ASE con ILB. Si quería usar *.contoso.com* como sufijo de dominio, podía hacerlo y eso incluiría el sitio SCM. Había problemas con ese modelo, entre los que se incluían la administración del certificado SSL predeterminado, la falta de inicio de sesión único con el sitio SCM y la obligatoriedad de usar un certificado comodín. El proceso de actualización de certificados predeterminados del ASE con ILB también se ha interrumpido y ha provocado el reinicio de la aplicación. Para solucionar estos problemas, el comportamiento del ASE con ILB se ha modificado para que use un sufijo de dominio basado en el nombre del ASE y un sufijo propiedad de Microsoft. El cambio en el comportamiento del ASE con ILB solo afecta a aquellos ASE creados después de mayo de 2019. Los ASE con ILB existentes anteriormente deben todavía administrar el certificado predeterminado del ASE y su configuración de DNS.

## <a name="publishing"></a>Publicación

En una instancia de ASE, al igual que lo que ocurre con App Service multiinquilino, puede publicar con estos métodos:

- Implementación web
- FTP
- Integración continua (CI)
- Operación de arrastrar y soltar en la consola Kudu
- Un IDE, como Visual Studio, Eclipse o IntelliJ IDEA

Con un entorno ASE externo, todas estas opciones de publicación trabajan del mismo modo. Para más información, consulte [Implementación en Azure App Service][AppDeploy].

Con un ASE con un ILB, los puntos de conexión de publicación solo están disponibles a través de ILB. El ILB está en una dirección IP privada en la subred de ASE en la red virtual. Si no tiene acceso de red al ILB, no puede publicar ninguna aplicación en ese entorno ASE. Tal como se indica en [Creación y uso de un ASE con un ILB][MakeILBASE], debe configurar DNS para las aplicaciones del sistema. Ese requisito incluye el punto de conexión de SCM. Si los puntos de conexión no se han definido correctamente, no puede llevar a cabo la publicación. Los IDE también deben tener acceso de red al ILB para publicar directamente en él.

Sin cambios adicionales, los sistemas de CI basados en Internet, como GitHub y Azure DevOps, no funcionan con una instancia de ASE con ILB, ya que el punto de conexión de publicación no es accesible desde Internet. Puede habilitar la publicación en una instancia de ASE con ILB desde Azure DevOps instalando un agente de versiones autohospedado en la red virtual que contiene la instancia de ASE con ILB. También puede utilizar un sistema de CI que use un modelo de extracción, como Dropbox.

Los puntos de conexión de publicación para las aplicaciones en un ASE con un ILB usan el dominio con el que se creó el ASE con un ILB. Puede verlo en el perfil de publicación de la aplicación y en el panel del portal de la aplicación (en **Información general** > **Información esencial** y también en **Propiedades**).

## <a name="storage"></a>Storage

Una instancia de ASE dispone de 1 TB de almacenamiento para todas las aplicaciones que contiene. El plan de App Service de la SKU del plan de tarifa Aislado tiene un límite de 250 GB. En un ASE, se agregan 250 GB de almacenamiento por plan de App Service hasta el límite de 1 TB. Puede tener más de cuatro planes de App Service, pero no se agregará más espacio de almacenamiento más allá del límite de 1 TB.

## <a name="logging"></a>Registro

Puede integrar la instancia de ASE con Azure Monitor para enviar registros sobre esta instancia de ASE a Azure Storage, Azure Event Hubs o Log Analytics. Estos elementos se registran hoy:

| Situación | Message |
|---------|----------|
| ASE no tiene un estado correcto | La instancia de ASE especificada no tiene un estado correcto porque la configuración de la red virtual no es válida. La instancia de ASE se suspenderá si se mantiene un estado que no es correcto. Asegúrese de que se siguen las instrucciones indicadas aquí: https://docs.microsoft.com/azure/app-service/environment/network-info. |
| La subred de ASE casi no tiene espacio | La instancia de ASE especificada está en una subred que casi no tiene espacio. Quedan {0} direcciones. Una vez que se agoten estas direcciones, la instancia de ASE no se podrá escalar.  |
| ASE está a punto de alcanzar el límite total de instancias | La instancia de ASE especificada está a punto de alcanzar al límite total de instancias. Actualmente contiene {0} instancias de planes de App Service, de un máximo de 201 instancias. |
| ASE no puede obtener acceso a una dependencia | La instancia de ASE especificada no puede obtener acceso a {0}.  Asegúrese de que se siguen las instrucciones indicadas aquí: https://docs.microsoft.com/azure/app-service/environment/network-info. |
| La instancia de ASE está suspendida | La instancia de ASE especificada está suspendida. Esta suspensión puede deberse a que hay pocas cuentas o a que una configuración de red virtual no es válida. Resuelva la causa principal del problema y reanude la instancia de ASE para que siga atendiendo el tráfico. |
| Se ha iniciado una actualización de ASE | Se ha iniciado una actualización de la plataforma a la instancia de ASE especificada. Es posible que se produzcan retrasos en las operaciones de escalado. |
| Se ha completado una actualización de ASE | Ha finalizado una actualización de plataforma de la instancia de ASE especificada. |
| Se han iniciado operaciones de escalado | Se ha comenzado a escalar un plan de App Service ({0}). Estado deseado: {1} I{2} trabajos.
| Se han completado las operaciones de escalado | Ha finalizado el escalado de un plan de App Service ({0}). Estado actual: {1} I{2} trabajos. |
| Se han producido errores en las operaciones de escalado | No se ha podido escalar un plan de App Service ({0}). Estado actual: {1} I{2} trabajos. |

Puede habilitar el registro en la instancia de ASE:

1. En el portal, vaya a **Configuración de diagnóstico**.
1. Seleccione **Agregar configuración de diagnóstico**.
1. Especifique un nombre para la integración de registros.
1. Seleccione y configure los destinos de registro que desee.
1. Seleccione **AppServiceEnvironmentPlatformLogs**.

![Configuración de registros de diagnóstico de ASE][4]

Si integra esta solución con Log Analytics, podrá ver los registros seleccionando **Registros** en el portal de ASE y creando una consulta en **AppServiceEnvironmentPlatformLogs**. Los registros solo se emiten cuando el ASE tiene un evento que lo desencadenará. Si el ASE no tiene este tipo de evento, no habrá ningún registro. Para ver rápidamente un ejemplo de registros en el área de trabajo de Log Analytics, realice una operación de escalado con uno de los planes de App Service del ASE. Después, puede ejecutar una consulta en **AppServiceEnvironmentPlatformLogs** para ver esos registros. 

**Creación de una alerta**

Para crear una alerta sobre los registros, siga las instrucciones de [Creación, visualización y administración de alertas de registro mediante Azure Monitor][logalerts]. En resumen:

* Abra la página Alertas en el portal de ASE.
* Seleccione **Nueva regla de alertas**.
* Seleccione el recurso para que sea el área de trabajo de Log Analytics.
* Establezca la condición con una búsqueda de registros personalizada que use una consulta como "AppServiceEnvironmentPlatformLogs | donde ResultDescription contiene "ha empezado el escalado" o cualquier cosa que desee. Establezca el umbral según corresponda. 
* Agregue o cree un grupo de acciones según sus preferencias. En el grupo de acciones se define la respuesta a la alerta como, por ejemplo, enviar un correo electrónico o un mensaje SMS.
* Asigne un nombre a la alerta y guárdela.

## <a name="upgrade-preference"></a>Preferencia de actualización

Si tiene varias instancias de ASE, es posible que prefiera que unas se actualicen antes que otras. En el objeto **HostingEnvironment del ASE de Resource Manager**, puede establecer un valor en **upgradePreference**. El valor de **upgradePreference** se puede configurar utilizando una plantilla, ARMClient o https://resources.azure.com. Estos son los tres valores posibles:

- **Ninguna**: Azure actualizará la instancia de ASE en alguno de los lotes. Este es el valor predeterminado.
- **Early**: la instancia de ASE se actualizará durante la primera mitad de las actualizaciones de App Service.
- **Late**: la instancia de ASE se actualizará durante la segunda mitad de las actualizaciones de App Service.

Si utiliza https://resources.azure.com, siga estos pasos para establecer el valor **upgradePreferences**:

1. Vaya a resources.azure.com e inicie sesión con su cuenta de Azure.
1. Recorra los recursos hasta subscriptions\/\[nombre de suscripción\]\/resourceGroups\/\[nombre del grupo de recursos\]\/providers\/Microsoft.Web\/hostingEnvironments\/\[nombre de ASE\].
1. Seleccione **Lectura y escritura** en la parte superior.
1. Seleccione **Editar**.
1. Establezca **upgradePreference** en cualquiera de los tres valores que desee.
1. Seleccione **Revisión**.

![Pantalla de recursos de azure.com][5]

Tiene más sentido utilizar la característica **upgradePreferences** cuando hay varias instancias de ASE, porque las que estén configuradas con el valor "Early" (Pronto) se actualizarán antes de las que tienen el valor "Late" (Tarde). Si tiene varias instancias de ASE, los entornos de ASE de desarrollo y pruebas deben establecerse en "Early", mientras que los entornos de ASE de producción deben establecerse en "Late".

## <a name="pricing"></a>Precios

La SKU de precios denominada *Aislado* solo puede utilizarse con instancias de ASE. Todos los planes de App Service que se hospedan en la instancia de ASE están en el plan de tarifa Aislado. Las tarifas del plan Aislado para los planes de App Service pueden variar según la región.

Además del precio de los planes de App Service, hay una tarifa fija para el propio entorno ASE. Esta tarifa fija no cambia con el tamaño del entorno ASE. Se aplica a la infraestructura de ASE con una velocidad de escalado predeterminada de un front-end adicional por cada 15 instancias del plan de App Service.

Si la velocidad de escalado predeterminada de un front-end por cada quince instancias del plan de App Service no es lo suficientemente rápida, puede ajustar la ratio con la que se agregan servidores front-end o el tamaño de estos. Al ajustar la proporción o el tamaño, paga por los núcleos de front-end que no se agregarían de forma predeterminada.

Por ejemplo, si ajusta el factor de escala en 10, se agrega un Front-End por cada 10 instancias en los planes de App Service. La tarifa fija cubre una escala de un Front-End por cada 15 instancias. Con un factor de escala de 10, paga una tarifa por el tercer servidor front-end que se agrega para las 10 instancias del plan de App Service. No es necesario que pague por él si llega a 15 instancias, ya que se agregó automáticamente.

Si ajusta el tamaño de los front-end a dos núcleos pero no se ajusta la ratio, pagará por los núcleos adicionales. Las instancias de ASE se crean con dos servidores front-end, por lo que incluso estando por debajo del umbral de escalado automático, pagaría por dos núcleos adicionales si aumentara el tamaño a servidores front-end con dos núcleos.

Para más información, consulte [Precios de Azure App Service][Pricing].

## <a name="delete-an-ase"></a>Eliminar un entorno ASE

Para eliminar un entorno ASE:

1. Seleccione **Eliminar** en la parte superior del panel **App Service Environment**.

1. Escriba el nombre de su ASE para confirmar que desea eliminarlo. Cuando se elimina un entorno ASE, también se elimina todo su contenido.

    ![Eliminación de ASE][3]

1. Seleccione **Aceptar**.

## <a name="ase-cli"></a>CLI de ASE

Hay funcionalidades de línea de comandos para administrar a un ASE.  Los comandos de tipo AZ CLI se indican a continuación.

```azurecli
C:\>az appservice ase --help

Group
    az appservice ase : Manage App Service Environments v2.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    create         : Create app service environment.
    delete         : Delete app service environment.
    list           : List app service environments.
    list-addresses : List VIPs associated with an app service environment.
    list-plans     : List app service plans associated with an app service environment.
    show           : Show details of an app service environment.
    update         : Update app service environment.

For more specific examples, use: az find "az appservice ase"
```



<!--Image references-->
[1]: ./media/using_an_app_service_environment/usingase-appcreate.png
[2]: ./media/using_an_app_service_environment/usingase-pricingtiers.png
[3]: ./media/using_an_app_service_environment/usingase-delete.png
[4]: ./media/using_an_app_service_environment/usingase-logsetup.png
[4]: ./media/using_an_app_service_environment/usingase-logs.png
[5]: ./media/using_an_app_service_environment/usingase-upgradepref.png

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
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[logalerts]: ../../azure-monitor/platform/alerts-log.md