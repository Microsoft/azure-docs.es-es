---
title: Configuración de ASE v1
description: Configuración, administración y supervisión de App Service Environment v1. Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: ccompy
ms.assetid: b5a1da49-4cab-460d-b5d2-edd086ec32f4
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 598e43d07c213cfeb25f0ecbc7bd02b6ec54b7ed
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962594"
---
# <a name="configuring-an-app-service-environment-v1"></a>Configuración de una instancia de App Service Environment v1

> [!NOTE]
> Este artículo trata sobre App Service Environment v1.  Hay una versión más reciente de App Service Environment que resulta más fácil de usar y se ejecuta en una infraestructura más eficaz. Para aprender más sobre la nueva versión, consulte [Introducción a App Service Environment](intro.md).
> 

## <a name="overview"></a>Información general
A grandes rasgos, un entorno de Azure App Service Environment consta de varios componentes principales:

* Los recursos de proceso que se ejecutan en el servicio hospedado de la instancia de App Service Environment
* Storage
* Una base de datos
* Azure Virtual Network implementada según el modelo clásico (V1) o según el modelo de Resource Manager (V2) 
* Una subred con el servicio hospedado del entorno de App Service que se ejecuta en él

### <a name="compute-resources"></a>Recursos de proceso
Utilice los recursos de proceso para los cuatro grupos de recursos.  Cada entorno de App Service (ASE) tiene un conjunto de servidores front-end y tres posibles grupos de trabajo. No es necesario usar los tres grupos de trabajo y, si lo desea, puede usar solo uno o dos.

Los host de los grupos de recursos, los servidores front-end y los trabajos no son accesibles directamente para los inquilinos. No se puede usar el protocolo de escritorio remoto (RDP) para conectarse a ellos, cambiar su aprovisionamiento o actuar como un administrador en ellos.

Puede establecer el tamaño y la cantidad de grupos de recursos. En un ASE tiene cuatro opciones de tamaño cuyos nombres van de P1 a P4. Para obtener detalles sobre los tamaños y sus precios, consulte [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/).
Cambiar el tamaño o la cantidad se llama operación de escalado.  Solo puede haber una operación de escalado en curso en un momento determinado.

**Servidores front-end**: Los servidores front-end son los puntos de conexión HTTP/HTTPS para las aplicaciones que se mantienen en la instancia de ASE. No se ejecutan cargas de trabajo en los servidores front-end.

* Un ASE comienza con dos P2, lo cual es suficiente para cargas de trabajo de desarrollo y pruebas y cargas de trabajo de producción de bajo nivel. Es muy recomendable que utilice los P3 para cargas de trabajo de producción de moderadas a intensas.
* Para las cargas de trabajo de moderadas a intensas, se recomienda contar con al menos cuatro P3 para garantizar que haya suficientes servidores front-end en ejecución cuando se produzca el mantenimiento programado. Las actividades de mantenimiento programado desactivan un front-end cada vez. Esto reduce la capacidad global de los servidores front-end disponibles durante las actividades de mantenimiento.
* Los servidores front-end pueden tardar hasta una hora en aprovisionarse. 
* Para afinar más en el escalado, debe supervisar el porcentaje de CPU, el porcentaje de memoria y las métricas de solicitudes activas para el grupo de servidores front-end. Si los porcentajes de CPU o memoria están por encima del 70 % cuando se ejecutan los P3, agregue más servidores front-end. Si los valores de solicitudes activas están en promedio entre 15 000 y 20 000 solicitudes por front-end, debe agregar también más servidores front-end. El objetivo general es mantener los porcentajes de CPU y memoria por debajo del 70 % y el promedio de solicitudes activas por debajo de 15 000 solicitudes por front-end al ejecutar P3.  

**Trabajos**: Los trabajos se encuentran donde se ejecutan realmente sus aplicaciones. Al escalar verticalmente sus planes de App Service se utiliza el trabajo del grupo de trabajo asociado.

* No es posible agregar trabajo de manera instantánea. Pueden tardar hasta una hora en aprovisionarse.
* Escalar el tamaño de un recurso de proceso para cualquier grupo tardará menos de una hora por cada dominio de actualización. Hay 20 dominios de actualización en un ASE. El escalado del tamaño de proceso de un grupo de trabajo con diez instancias puede tardar hasta diez horas.
* Si cambia el tamaño de los recursos de proceso que se usan en un grupo de trabajo, provocará arranques en frío de las aplicaciones que se ejecutan en ese grupo de trabajo.

La forma más rápida de cambiar el tamaño de recursos de proceso de un grupo de trabajo que no está ejecutando aplicaciones es el siguiente:

* Reduzca la cantidad de trabajos a 2.  El tamaño mínimo de la reducción en el portal es 2. El proceso de cancelación de la asignación de instancias tardará algunos minutos. 
* Seleccione el nuevo tamaño de proceso y el número de instancias. Desde este punto, el proceso tardará hasta 2 horas en completarse.

Si las aplicaciones requieren un mayor tamaño de recursos de proceso, no le servirán las indicaciones anteriores. En lugar de cambiar el tamaño del grupo de trabajo que hospeda esas aplicaciones, puede rellenar otro grupo de trabajo con instancias del tamaño deseado y trasladar sus aplicaciones a ese grupo.

* Cree las instancias adicionales del tamaño de proceso necesario en otro grupo de trabajo. Desde este punto, el proceso tardará hasta una hora en completarse.
* Vuelva a asignar sus planes de App Service que hospedan las aplicaciones que necesitan un tamaño mayor al grupo de trabajo recién configurado. Se trata de una operación rápida que debería tardar menos de un minuto en completarse.  
* Reduzca verticalmente el primer grupo de trabajo si ya no necesita esas instancias no utilizadas. La operación tarda unos minutos.

**Escalabilidad automática**: Una de las herramientas que le pueden ayudar a administrar el consumo de recursos de proceso es la escalabilidad automática. Este se puede aplicar a grupos de trabajo o servidores front-end. Puede, por ejemplo, aumentar las instancias de cualquier tipo de grupo por la mañana y reducirlo por la noche. O puede agregar instancias cuando la cantidad de trabajo disponible en un grupo de trabajo caiga por debajo de un umbral determinado.

Si desea establecer reglas de escalado automático alrededor de métricas del grupo de recursos de proceso, tenga en cuenta el tiempo necesario para el aprovisionamiento. Para más información sobre la escalabilidad automática de las instancias de App Service Environment, consulte [Configuración de la escalabilidad automática en una instancia de App Service Environment][ASEAutoscale].

### <a name="storage"></a>Storage
Cada ASE se configura con 500 GB de almacenamiento. Este espacio se usa en todas las aplicaciones del ASE. Este espacio de almacenamiento es parte del ASE y actualmente no se puede cambiar para usar su espacio de almacenamiento. Si va a realizar ajustes en el enrutamiento o la seguridad de la red virtual, debe seguir permitiendo el acceso a Azure Storage o, de lo contrario, ASE no funcionará.

### <a name="database"></a>Base de datos
La base de datos contiene la información que define el entorno, así como los detalles acerca de las aplicaciones que se ejecutan en él. Esto también forma parte de la suscripción mantenida en Azure. No es algo que tenga una capacidad directa para manipular. Si va a realizar ajustes en el enrutamiento o la seguridad de la red virtual, debe seguir permitiendo el acceso a SQL Azure o, de lo contrario, ASE no funcionará.

### <a name="network"></a>Red
La red virtual que se usa con el ASE puede ser una de las que hizo al crear el ASE o alguna que ya tuviera de antes. Si crea la subred durante la creación del ASE, obligará al ASE a permanecer en el mismo grupo de recursos que la red virtual. Si necesita el grupo de recursos que usa su ASE sea diferente de la red virtual, debe crear su ASE utilizando una plantilla de Resource Manager.

Existen algunas restricciones en la red virtual que se aplican a un ASE:

* La red virtual debe ser una red virtual regional.
* Es necesario que haya una subred con 8 o más direcciones donde se implementa el ASE.
* Después de utilizar una subred para hospedar un ASE, no se puede cambiar el intervalo de direcciones de la subred. Por este motivo, se recomienda que la subred contenga al menos 64 direcciones para tener en cuenta el crecimiento futuro del ASE.
* Puede que no haya nada más en la subred excepto el ASE.

A diferencia del servicio hospedado que contiene el ASE, el usuario controla la [red virtual][virtualnetwork] y la subred.  Puede administrar la red virtual a través de la interfaz de usuario de la red virtual o de Powershell.  Un ASE se puede implementar en una red virtual según el modelo clásico o según el modelo de Resource Manager.  Las experiencias del portal y de la API son ligeramente diferentes para las redes virtuales implementadas según el modelo clásico y aquellas implementadas según el modelo de Resource Manager pero la experiencia del ASE es la misma.

La red virtual que se utiliza para hospedar un ASE puede usar direcciones IP RFC1918 privadas o puede usar direcciones IP públicas.  Si desea usar un intervalo IP que no está cubierto por RFC1918 (10.0.0.0/8, 172.16.0.0/12 y 192.168.0.0/16), deberá crear la red virtual y la subred que va a usar el ASE antes de la creación de este.

Dado que esta funcionalidad coloca Azure App Service en la red virtual, significa que las aplicaciones hospedadas en el ASE ahora pueden tener acceso a los recursos disponibles a través de ExpressRoute o redes privadas virtuales de sitio a sitio directamente. Las aplicaciones que están dentro de los entornos de App Service no requieren características de red adicionales para tener acceso a los recursos disponibles de la red virtual que hospeda el entorno de App Service. Esto significa que no es necesario usar la integración de red virtual o conexiones híbridas para acceder a los recursos o para conectarse a la red virtual. Sin embargo, puede usar ambas características para acceder a los recursos en redes que no están conectadas a la red virtual.

Por ejemplo, puede usar la integración de red virtual para integrarse con una red virtual que está en su suscripción, pero que no está conectada a la red virtual en la que se encuentra el ASE. También puede usar las conexiones híbridas para tener acceso a los recursos de otras redes, de la manera habitual.  

Si la red virtual está configurada con una VPN de ExpressRoute, debe tener en cuenta las necesidades de enrutamiento que tenga un ASE. Existen algunas configuraciones de rutas definidas por el usuario (UDR) que son incompatibles con un ASE. Para más información respecto a la ejecución de una instancia de ASE en una red virtual con ExpressRoute, consulte el artículo [Ejecución de una instancia de App Service Environment en una red virtual con ExpressRoute][ExpressRoute].

#### <a name="securing-inbound-traffic"></a>Protección del tráfico de entrada
Hay dos métodos principales para controlar el tráfico entrante en el ASE.  Puede utilizar grupos de seguridad de red (NSG) para controlar qué direcciones IP pueden tener acceso al ASE como se describe aquí [Cómo controlar el tráfico de entrada a un entorno de App Service](app-service-app-service-environment-control-inbound-traffic.md) y también puede configurar el ASE con un equilibrador de carga interno (ILB).  Estas características también pueden utilizarse a la vez si desea restringir el acceso mediante los NSG al ASE del ILB.

Cuando se crea un ASE, este creará a una dirección VIP de la red virtual.  Hay dos tipos de direcciones VIP, internas y externas.  Cuando se crea un ASE con una dirección VIP externa, se podrá acceder a las aplicaciones del ASE a través de una dirección IP enrutable de Internet. Si selecciona una dirección VIP interna, el ASE se configurara con un ILB y no podrá acceder directamente a ella a través de Internet.  Un ASE del ILB seguirá necesitando una dirección VIP externa pero solo se utilizará para la administración de Azure y el acceso de mantenimiento.  

Durante la creación del ASE del ILB deberá proporcionar el subdominio utilizado por este y tendrá que administrar su propio DNS para el subdominio que especifique.  Puesto que ha establecido el nombre del subdominio deberá administrar también el certificado usado para el acceso HTTPS.  Después de la creación del ASE se le pedirá que proporcione el certificado.  Para más información sobre la creación y el uso de una instancia de ASE de ILB, lea [Uso de un equilibrador de carga interno con una instancia de App Service Environment][ILBASE]. 

## <a name="portal"></a>Portal
Puede administrar y supervisar el entorno de App Service mediante la interfaz de usuario que está disponible en Azure Portal. Si tiene un ASE, es probable que vea el símbolo de App Service en la barra lateral. Este símbolo se usa para representar entornos de App Service en Azure Portal:

![Símbolo del entorno de App Service][1]

Para abrir la interfaz de usuario que muestra todos los entornos de App Service, puede utilizar el icono o seleccionar el botón de contenido adicional (símbolo ">") en la parte inferior de la barra lateral para seleccionar los entornos de App Service. Al seleccionar uno de los ASE mostrados, se abre la interfaz de usuario usada para supervisarlo y administrarlo.

![Interfaz de usuario para supervisar y administrar el entorno de App Service][2]

La primera hoja muestra algunas de las propiedades del ASE junto con un gráfico de métrica por grupo de recursos. Algunas de las propiedades que aparecen en el bloque **Essentials** también son hipervínculos que abrirán la hoja asociada. Por ejemplo puede seleccionar el nombre de la **red virtual** para abrir la interfaz de usuario asociada a la red virtual en la que se ejecuta el ASE. Los **planes de App Service** y las **aplicaciones** abren hojas que muestran los elementos que se encuentran en el ASE.  

### <a name="monitoring"></a>Supervisión
Los gráficos le permiten ver una variedad de estadísticas de rendimiento de cada grupo de recursos. Para el grupo de servidores front-end, puede supervisar la CPU y la memoria promedio. Para grupos de trabajo, puede supervisar la cantidad que se usa y la cantidad que está disponible.

Varios planes de App Service pueden usar los trabajos en un grupo de trabajo. La carga de trabajo no se distribuye de la misma manera que en los servidores front-end; por tanto, el uso de CPU y memoria no proporciona una gran cantidad de información útil. Es más importante realizar el seguimiento de cuántos trabajos usó y están disponibles, sobre todo si va a administrar este sistema para que otros lo usen.  

También puede utilizar todas las métricas que se pueden seguir en los gráficos para configurar alertas. La configuración de alertas funciona aquí de la misma forma que en cualquier otro lugar de App Service. Puede establecer una alerta de la parte de la interfaz de usuario de **alertas** o a partir de la exploración en profundidad de la interfaz de usuario de métricas y de hacer clic en **Agregar alerta**.

![Interfaz de usuario de métricas][3]

Las métricas que acabamos de analizar son las métricas el entorno de App Service. También hay métricas disponibles en el nivel del plan de App Service. Aquí es dónde la supervisión de la CPU y de la memoria tiene más sentido.

En un ASE, todos los planes de App Service son planes dedicados. Esto significa que las únicas aplicaciones que se ejecutan en los hosts asignados a dicho plan de App Service son las aplicaciones de este plan. Para ver los detalles del plan de App Service, muéstrelo desde cualquiera de las listas de la interfaz de usuario del ASE o examine los **planes de App Service** que enumeran todos ellos.   

### <a name="settings"></a>Configuración
En la hoja del ASE hay una sección de **configuración** que contiene varias funcionalidades importantes:

**Configuración** > **Propiedades**: La hoja **Configuración** se abrirá automáticamente cuando acceda a la hoja de la instancia de ASE. En la parte superior se encuentra **Propiedades**. Hay una serie de elementos aquí que son redundantes con lo que ve en **Essentials**, aunque lo que es muy útil es la **dirección IP virtual**, así como la **dirección IP saliente**.

![Hoja de configuración y propiedades][4]

**Configuración** > **Direcciones IP**: Al crear una aplicación de capa de sockets seguros (SSL) de IP en la instancia de ASE, necesita una dirección SSL de IP. Para obtener una, el ASE debe tener algunas direcciones SSL de IP que se puedan asignar. Cuando se crea un ASE, este tiene una dirección SSL de IP para este propósito, pero puede agregar más. Las direcciones SSL de IP adicionales conllevan un cargo, como se muestra en los [precios de App Service][AppServicePricing], en la sección sobre las conexiones SSL. El precio adicional es el precio de SSL de IP.

Grupos de trabajo**Configuración de** > **Grupo de servidores front-end** / **Grupos de trabajo**: Cada una de estas hojas del grupo de recursos ofrece la posibilidad de ver información únicamente acerca de ese grupo de recursos, además de proporcionar controles para escalar completamente ese grupo de recursos.  

La hoja base de cada grupo de recursos proporciona un gráfico con métricas de dicho grupo de recursos. Al igual que con los gráficos de la hoja de ASE, puede acceder al gráfico y configurar las alertas como desee. Configurar una alerta desde la hoja del ASE para un grupo de recursos específico es igual que hacerlo desde el grupo de recursos. Desde la hoja de **configuración** del grupo de trabajo tendrá acceso a la lista de todas las aplicaciones o planes de App Service que se ejecutan en este grupo de trabajo.

![Interfaz de usuario de configuración de grupo de trabajo][5]

### <a name="portal-scale-capabilities"></a>Funcionalidad de escalado del portal
Hay tres operaciones de escala:

* Cambio del número de direcciones IP en el ASE que están disponibles para el uso de SSL de IP.
* Cambio de tamaño del recurso de proceso usado en un grupo de recursos.
* Cambio de la cantidad de recursos de proceso que se usa en un grupo de recursos, bien manualmente o mediante el escalado automático.

En el portal existen tres formas de controlar el número de servidores de los grupos de recursos:

* Una operación de escalado desde la hoja de ASE principal de la parte superior. Puede realizar varios cambios en la configuración de escalado de los grupos de servidores front-end y trabajo. Todas se aplican como una única operación.
* Una operación de escalado manual desde la hoja **Escala** de grupos de recursos individuales que se encuentran en **Configuración**.
* Una operación de escalado automático que se configura desde la hoja **Escala** de grupos de recursos individuales.

Para usar la operación de escalado en la hoja de ASE, arrastre el control deslizante a la cantidad que desea y guarde. Esta interfaz de usuario también admite el cambio del tamaño.  

![Interfaz de usuario de escalado][6]

Para usar las funcionalidades de escalado manual o automático en un grupo de recursos específico, vaya a **Configuración** > **Grupo de servidores front-end** / **Grupos de trabajo** según corresponda. Abra el grupo que desea cambiar. Vaya a **Configuración** > **Escalar horizontalmente** o **Configuración** > **Escalar verticalmente**. La hoja **Escalar horizontalmente** le permite controlar la cantidad de la instancia. **Escalar verticalmente** le permite controlar el tamaño del recurso.  

![Interfaz de usuario de configuración de escalado][7]

## <a name="fault-tolerance-considerations"></a>Consideraciones de tolerancia a errores
Puede configurar un entorno de App Service para usar hasta 55 recursos de proceso en total. De esos 55 recursos de proceso, solamente 50 se pueden usar para hospedar cargas de trabajo. El motivo es de esto es doble. Hay un mínimo de 2 recursos de proceso front-end.  Esto deja hasta 53 para admitir la asignación de grupos de trabajo. Para proporcionar tolerancia a errores, necesita tener un recurso de proceso adicional asignado según las reglas siguientes:

* Cada grupo de trabajo necesita al menos un recurso de proceso adicional al que no se puede asignar cargas de trabajo.
* Cuando la cantidad de recursos de proceso de un grupo de trabajo supera un determinado valor, se necesita otro recurso de proceso para tolerancia a errores. Esto no ocurre en el grupo de servidores front-end.

Dentro de cualquier grupo de trabajo los requisitos de tolerancia a errores son que para un determinado valor de X recursos asignados a un grupo de trabajo:

* Si el valor de X está comprendido entre 2 y 20, la cantidad de recursos de proceso utilizables que puede usar para cargas de trabajo es X-1.
* Si el valor de X está comprendido entre 21 y 40, la cantidad de recursos de proceso utilizables que puede usar para cargas de trabajo es X-2.
* Si el valor de X está comprendido entre 41 y 53, la cantidad de recursos de proceso utilizables que puede usar para cargas de trabajo es X-3.

La superficie mínima tiene dos servidores front-end y dos trabajos.  Con las instrucciones anteriores, se muestran a continuación algunos ejemplos que sirven de aclaración:  

* Si tiene 30 trabajos en un solo grupo, 28 de ellos pueden usarse para hospedar las cargas de trabajo.
* Si tiene dos trabajos en un solo grupo, en este caso solo se puede usar uno para hospedar las cargas de trabajo.
* Si tiene 20 trabajos en un solo grupo, en este caso solo se pueden usar 19 para hospedar las cargas de trabajo.  
* Si tiene 21 trabajos en un solo grupo, en este caso solo se pueden usar 19 para hospedar las cargas de trabajo.  

El aspecto de la tolerancia a errores es importante, pero debe tenerlo en cuenta cuando la escala supera determinados umbrales. Si desea agregar más capacidad a partir de 20 instancias, en este caso elija 22 o un valor superior ya que 21 no agregará más capacidad. Lo mismo puede decirse por encima de 40, donde el número siguiente que agrega capacidad es 42.  

## <a name="deleting-an-app-service-environment"></a>Eliminación de un entorno de App Service
Si desea eliminar un entorno de App Service, simplemente utilice la acción **Eliminar** que se encuentra en la parte superior de la hoja Entorno de App Service. Al hacerlo, se le pedirá que escriba el nombre del entorno de App Service para confirmar que realmente quiere hacerlo. Tenga en cuenta que cuando se elimina un entorno de App Service, se elimina también todo su contenido.  

![Eliminación de la interfaz de usuario de un entorno de App Service][9]  

## <a name="getting-started"></a>Introducción
Para empezar a trabajar con los entornos de App Service, consulte [Creación de un entorno de App Service](app-service-web-how-to-create-an-app-service-environment.md).

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-web-configure-an-app-service-environment/ase-icon.png
[2]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-aseblade.png
[3]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolchart.png
[4]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-properties.png
[5]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolblade.png
[6]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-scalecommand.png
[7]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-poolscale.png
[8]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-pricingtiers.png
[9]: ./media/app-service-web-configure-an-app-service-environment/aseconfig-deletease.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[Appserviceplans]: ../overview-hosting-plans.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoScale]: app-service-web-scale-a-web-app-in-an-app-service-environment.md
[ControlInbound]: app-service-app-service-environment-control-inbound-traffic.md
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[AppServicePricing]: https://azure.microsoft.com/pricing/details/app-service/
[ASEAutoscale]: app-service-environment-auto-scale.md
[ExpressRoute]: app-service-app-service-environment-network-configuration-expressroute.md
[ILBASE]: app-service-environment-with-internal-load-balancer.md