---
title: Conexiones híbridas
description: Aprenda a crear y usar conexiones híbridas en Azure App Service para acceder a recursos de redes distintas.
author: ccompy
ms.assetid: 66774bde-13f5-45d0-9a70-4e9536a4f619
ms.topic: article
ms.date: 06/08/2020
ms.author: ccompy
ms.custom: seodec18, fasttrack-edit
ms.openlocfilehash: 8c8b71e4be11a886da3e0889add0a3cdc19a56c3
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962424"
---
# <a name="azure-app-service-hybrid-connections"></a>Hybrid Connections de Azure App Service

Conexiones híbridas es tanto un servicio de Azure como una característica de Azure App Service. Como servicio tiene un uso y unas funcionalidades que van más allá de aquellas que se usan en App Service. Para más información sobre Conexiones híbridas y su uso fuera de App Service, consulte [Protocolo de Conexiones híbridas de Azure Relay][HCService].

En App Service, las Conexiones híbridas se pueden usar para acceder a recursos de aplicaciones en cualquier red que pueda realizar llamadas salientes a Azure a través del puerto 443. Las Conexiones híbridas proporcionan acceso desde la aplicación a un punto de conexión TCP y no habilitan una nueva forma de acceder a la aplicación. Dado que se utiliza en App Service, cada conexión híbrida se correlaciona con una combinación única de host y puerto TCP. Esto permite a las aplicaciones acceder a los recursos de cualquier sistema operativo, siempre que se trate de un punto de conexión TCP. La característica Conexiones híbridas no sabe lo que es el protocolo de aplicaciones ni sabe a qué se accede. Simplemente ofrece acceso a la red.  

## <a name="how-it-works"></a>Funcionamiento ##
Las Conexiones híbridas requieren la implementación de un agente de retransmisión desde donde se pueda acceder al punto de conexión deseado y a Azure. El agente de retransmisión, el Administrador de conexiones híbridas (HCM), llama a Azure Relay a través del puerto 443. Desde el sitio de la aplicación web, la infraestructura de App Service también se conecta a Azure Relay en nombre de la aplicación. Mediante las conexiones unidas, la aplicación puede acceder al punto de conexión deseado. La conexión usa TLS 1.2 para la seguridad y las claves de firma de acceso compartido (SAS) para la autenticación y la autorización.    

![Diagrama del flujo de alto nivel de conexión híbrida][1]

Cuando la aplicación realiza una solicitud DNS que coincide con un punto de conexión híbrida configurado, el tráfico TCP saliente se redirige por la conexión híbrida,  

> [!NOTE]
> lo que significa que debe intentar usar siempre un nombre DNS para la conexión híbrida. Cierto software cliente no lleva a cabo una búsqueda de DNS si el punto de conexión utiliza una dirección IP. 
>

### <a name="app-service-hybrid-connection-benefits"></a>Beneficios de usar las conexiones híbridas de App Service ###

El uso de la funcionalidad de conexiones híbridas reporta varios beneficios, entre los que se cuentan los siguientes:

- Las aplicaciones pueden acceder de forma segura a los servicios y sistemas locales.
- La característica no requiere ningún punto de conexión al que se pueda acceder a través de Internet.
- Se configura de forma rápida y sencilla. No se requieren puertas de enlace.
- Cada conexión híbrida coincide con una combinación única de host:puerto, lo que resulta útil para la seguridad.
- Normalmente no requiere vulnerabilidades de seguridad en el firewall, porque todas las conexiones salen a través de puertos web estándar.
- Dado que la característica está en el nivel de la red, da igual el lenguaje que use la aplicación y la tecnología que emplee el punto de conexión.
- Se puede utilizar para proporcionar acceso en varias redes desde una única aplicación. 
- Se admite en GA para las aplicaciones de Windows y se encuentra en versión preliminar para las aplicaciones de Linux.

### <a name="things-you-cannot-do-with-hybrid-connections"></a>Cosas que no se pueden hacer con las conexiones híbridas ###

Las cosas que no se pueden hacer con las conexiones híbridas incluyen:

- Montaje de una unidad.
- Uso de UDP.
- Acceso a servicios basados en TCP que usen puertos dinámicos, como el modo pasivo FTP o el modo pasivo extendido.
- Compatibilidad con LDAP, porque puede requerir UDP.
- Compatibilidad con Active Directory, porque un trabajo de App Service no se puede unir a un dominio. 

## <a name="add-and-create-hybrid-connections-in-your-app"></a>Incorporación y creación de Conexiones híbridas en la aplicación ##

Para crear una conexión híbrida, vaya a [Azure Portal][portal] y seleccione su aplicación. Seleccione **Redes** > **Configure los puntos de conexión de la conexión híbrida**. Aquí se pueden ver las conexiones híbridas que están configuradas para su aplicación.  

![Captura de pantalla de la lista de conexiones híbridas][2]

Para agregar una conexión híbrida nueva, seleccione **[+] Agregar conexión híbrida**.  Verá una lista de las conexiones híbridas que ya haya creado. Para agregar una o varias de ellas en la aplicación, seleccione las que desee y, luego, seleccione **Agregar conexión híbrida seleccionada**.  

![Captura de pantalla del portal de conexiones híbridas][3]

Si quiere crear una conexión híbrida, seleccione **Crear conexión híbrida nueva**. Especifique: 

- Nombre de la conexión híbrida.
- El nombre de host del punto de conexión.
- El puerto del punto de conexión.
- El espacio de nombres de Service Bus que desea usar.

![Captura de pantalla del cuadro de diálogo Crear conexión híbrida nueva][4]

Cada conexión híbrida está asociada a un espacio de nombres de Service Bus y cada uno de estos espacios se encuentra en una región de Azure. Es importante intentar usar un espacio de nombres de Service Bus que se encuentre en la misma región que la aplicación, para evitar una latencia de red excesiva.

Si desea quitar una conexión híbrida de una aplicación, haga clic con el botón derecho en ella y seleccione **Desconectar**.  

Cuando se agrega una conexión híbrida a la aplicación, puede seleccionarla para ver su información detallada. 

![Captura de pantalla de los detalles de las conexiones híbridas][5]

### <a name="create-a-hybrid-connection-in-the-azure-relay-portal"></a>Creación de una conexión híbrida en el portal de Azure Relay ###

Además de la experiencia del portal dentro de la aplicación, puede crear conexiones híbridas en el portal de Azure Relay. Para que App Service use una conexión híbrida, debe:

* Requerir autorización del cliente.
* Tener un elemento de metadatos llamado punto de conexión que contenga una combinación de host:puerto como valor.

## <a name="hybrid-connections-and-app-service-plans"></a>Conexiones híbridas y planes de App Service ##

Las conexiones híbridas de App Service solo están disponibles en las SKU de precios de nivel Básico, Estándar, Premium y Aislado. No hay límites asociados al plan de precios.  

| Plan de precios | Número de conexiones híbridas que se pueden usar en el plan |
|----|----|
| Básico | 5 por plan |
| Estándar | 25 por plan |
| PremiumV2 | 200 por aplicación |
| Aislado | 200 por aplicación |

La interfaz de usuario del plan de App Service muestra cuántas conexiones híbridas se usan y qué aplicaciones las usan.  

![Captura de pantalla de las propiedades del plan de App Service][6]

Seleccione la conexión híbrida para ver detalles. Puede consultar toda la información que vio en la vista de la aplicación. También puede ver cuántas otras aplicaciones en el mismo plan usan esa conexión híbrida.

Hay un límite en el número de puntos de conexión de las conexiones híbridas que se pueden usar en un plan de App Service. Sin embargo, cada conexión híbrida se puede usar en cualquier cantidad de aplicaciones de ese plan. Por ejemplo, una conexión híbrida única que se usa en cinco aplicaciones distintas en un plan de App Service cuenta como una conexión híbrida.

### <a name="pricing"></a>Precios ###

Además de un requisito de SKU del plan de App Service, hay un costo adicional por el uso de las conexiones híbridas. Hay un cargo por cada agente de escucha que una conexión híbrida utilice. El agente de escucha es el Administrador de conexiones híbridas. Si tiene cinco conexiones híbridas compatibles con dos Administradores de conexiones híbridas, esto sería un total de 10 agentes de escucha. Para más información, consulte [Precios de Service Bus][sbpricing].

## <a name="hybrid-connection-manager"></a>Hybrid Connection Manager ##

La característica Conexiones híbridas requiere un agente de retransmisión en la red que hospeda el punto de conexión de la conexión híbrida. Dicho agente de retransmisión se denomina Hybrid Connection Manager (HCM). Para descargar HCM, desde la aplicación en [Azure Portal][portal], seleccione **Redes** > **Configure los puntos de conexión de la conexión híbrida**.  

Esta herramienta se ejecuta en Windows Server 2012 y versiones posteriores. El Administrador de conexiones híbridas se ejecuta como un servicio y se conecta para la salida a Azure Relay en el puerto 443.  

Después de instalar HCM, puede ejecutar HybridConnectionManagerUi.exe para usar la interfaz de usuario de la herramienta. Este archivo se encuentra en el directorio de instalación del Administrador de conexiones híbridas. En Windows 10, también puede simplemente buscar la *interfaz de usuario del Administrador de conexiones híbridas* en el cuadro de búsqueda.  

![Captura de pantalla de Administrador de conexiones híbridas][7]

Cuando inicia la interfaz de usuario de HCM, lo primero que se ve es una tabla donde se muestran todas las conexiones híbridas configuradas con esta instancia de HCM. Si desea hacer algún cambio, primero debe autenticarse con Azure. 

Para agregar una o varias conexiones híbridas a HCM:

1. Inicie la interfaz de usuario de HCM.
2. Seleccione **Configure another Hybrid Connection** (Configurar otra conexión híbrida).
![Captura de pantalla de configuración de conexiones híbridas nuevas][8]

1. Inicie sesión con su cuenta de Azure para obtener las conexiones híbridas disponibles con sus suscripciones. El HCM no usará su cuenta de Azure aparte de para esto. 
1. Elija una suscripción.
1. Seleccione las conexiones híbridas que desea que retransmita el HCM.
![Captura de pantalla de las conexiones híbridas][9]

1. Seleccione **Guardar**.

Ahora puede ver las conexiones híbridas que agregó. También puede seleccionar la conexión híbrida configurada para ver detalles.

![Captura de pantalla de los detalles de la conexión híbrida][10]

Para que HCM pueda admitir las conexiones híbridas con las que se configuró, necesita:

- Acceso de TCP a Azure a través del puerto 443.
- Acceso de TCP al punto de conexión de la conexión híbrida.
- La capacidad de realizar búsquedas de DNS en el host del punto de conexión y el espacio de nombres de Service Bus.

> [!NOTE]
> Azure Relay depende de Web Sockets para la conectividad. Esta funcionalidad solo está disponible en Windows Server 2012 o versiones posteriores. Por este motivo, HCM no se admite en versiones anteriores de Windows Server 2012.
>

### <a name="redundancy"></a>Redundancia ###

Cada HCM puede admitir varias conexiones híbridas. Además, varios HCM pueden admitir cualquier conexión híbrida dada. El comportamiento predeterminado es enrutar el tráfico a través de los HCM configurados en cualquier punto de conexión dado. Si desea alta disponibilidad en las conexiones híbridas de la red, ejecute varias instancias de HCM en máquinas independientes. El algoritmo de distribución de carga usado por el servicio Relay para distribuir el tráfico a los Administradores de conexiones híbridas es de asignación aleatoria. 

### <a name="manually-add-a-hybrid-connection"></a>Incorporación manual de una conexión híbrida ###

Para permitir que alguien de fuera de su suscripción hospede una instancia de HCM para una conexión híbrida determinada, comparta con dicho usuario la cadena de conexión de puerta de enlace de la conexión híbrida. Puede ver la cadena de conexión de la puerta de enlace en las propiedades de la conexión híbrida en [Azure Portal][portal]. Para usar dicha cadena, seleccione **Enter Manually** (Especificar manualmente) en el HCM y pegue la cadena de conexión de puerta de enlace.

![Incorporación manual de una conexión híbrida][11]

### <a name="upgrade"></a>Actualizar ###

El Administrador de conexiones híbridas se actualiza periódicamente para solucionar problemas o proporcionar mejoras. Cuando se publique una actualización, se mostrará un elemento emergente en la interfaz de usuario del Administrador de conexiones híbridas. Al aplicar la actualización se aplicarán los cambios y se reiniciará el Administrador de conexiones híbridas. 

## <a name="adding-a-hybrid-connection-to-your-app-programmatically"></a>Incorporación de una conexión híbrida a su aplicación mediante programación ##

La CLI de Azure es compatible con las Conexiones híbridas. Los comandos proporcionados funcionan tanto a nivel de aplicación como del plan de App Service.  Los comandos a nivel de aplicación son:

```azurecli
az webapp hybrid-connection

Group
    az webapp hybrid-connection : Methods that list, add and remove hybrid-connections from webapps.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    add    : Add a hybrid-connection to a webapp.
    list   : List the hybrid-connections on a webapp.
    remove : Remove a hybrid-connection from a webapp.
```

Los comandos del plan de App Service permiten establecer qué clave usará una conexión híbrida determinada. Hay dos claves establecidas en cada conexión híbrida, una principal y una secundaria. Puede optar por usar la clave principal o secundaria con los siguientes comandos. Esto le permite cambiar las claves para cuando desee regenerarlas periódicamente. 

```azurecli
az appservice hybrid-connection --help

Group
    az appservice hybrid-connection : A method that sets the key a hybrid-connection uses.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    set-key : Set the key that all apps in an appservice plan use to connect to the hybrid-
                connections in that appservice plan.
```

## <a name="secure-your-hybrid-connections"></a>Proteja su Conexiones híbridas ##

Cualquier usuario que tenga permisos suficientes en la retransmisión de Azure Service Bus subyacente puede agregar una conexión híbrida existente a otros App Service Web Apps. Esto significa que, si debe evitar que otros usuarios puedan reutilizar la misma conexión híbrida (por ejemplo, cuando el recurso de destino es un servicio que no tiene ninguna medida de seguridad adicional para evitar el acceso no autorizado), debe bloquear el acceso a Azure Service Bus Relay.

Cualquier persona con `Reader` acceso a la retransmisión podrá _ver_ la conexión híbrida al intentar agregarla a su aplicación web en Azure Portal, pero no podrá _agregarla_ ya que carece de los permisos para recuperar la cadena de conexión que se usa para establecer la conexión de retransmisión. Para agregar correctamente la conexión híbrida, deben tener el permiso `listKeys` (`Microsoft.Relay/namespaces/hybridConnections/authorizationRules/listKeys/action`). El rol `Contributor` o cualquier otro rol que incluya este permiso en la Retransmisión permitirá a los usuarios usar la conexión híbrida y agregarla a su propia Web Apps.

## <a name="troubleshooting"></a>Solución de problemas ##

El estado "Conectado" significa que hay al menos una instancia de HCM configurada con esa conexión híbrida y es capaz de conectarse a Azure. Si en el estado de la conexión híbrida no indica **Conectado**, la conexión híbrida no está configurada en ningún HCM que tenga acceso a Azure.

La razón principal por la que los clientes no se pueden conectar a su punto de conexión es porque este se especificó mediante una dirección IP en lugar de un nombre DNS. Si la aplicación no puede acceder al punto de conexión deseado y ha usado una dirección IP, utilice un nombre DNS que sea válido en el host en el que se ejecuta HCM. Compruebe también que el nombre DNS se resuelve adecuadamente en el host donde se ejecuta el HCM. Confirme que existe conectividad desde el host donde se ejecuta el HCM al punto de conexión de la conexión híbrida.  

En App Service, es posible invocar la herramienta de la línea de comandos **tcpping** desde la consola de herramientas avanzadas (Kudu). Esta herramienta puede indicarle si tienen acceso a un punto de conexión TCP, pero no indica si tiene acceso a un punto de conexión híbrida. Cuando usa la herramienta de la consola en un punto de conexión de la conexión híbrida, simplemente confirma que usa una combinación de host:puerto.  

Si tiene un cliente de línea de comandos para el punto de conexión, puede probar la conectividad desde la consola de la aplicación. Por ejemplo, puede probar el acceso a los puntos de conexión de servidor web mediante cURL.


<!--Image references-->
[1]: ./media/app-service-hybrid-connections/hybridconn-connectiondiagram.png
[2]: ./media/app-service-hybrid-connections/hybridconn-portal.png
[3]: ./media/app-service-hybrid-connections/hybridconn-addhc.png
[4]: ./media/app-service-hybrid-connections/hybridconn-createhc.png
[5]: ./media/app-service-hybrid-connections/hybridconn-properties.png
[6]: ./media/app-service-hybrid-connections/hybridconn-aspproperties.png
[7]: ./media/app-service-hybrid-connections/hybridconn-hcm.png
[8]: ./media/app-service-hybrid-connections/hybridconn-hcmadd.png
[9]: ./media/app-service-hybrid-connections/hybridconn-hcmadded.png
[10]: ./media/app-service-hybrid-connections/hybridconn-hcmdetails.png
[11]: ./media/app-service-hybrid-connections/hybridconn-manual.png
[12]: ./media/app-service-hybrid-connections/hybridconn-bt.png

<!--Links-->
[HCService]: /azure/service-bus-relay/relay-hybrid-connections-protocol/
[portal]: https://portal.azure.com/
[oldhc]: /azure/biztalk-services/integration-hybrid-connection-overview/
[sbpricing]: https://azure.microsoft.com/pricing/details/service-bus/
[armclient]: https://github.com/projectkudu/ARMClient/