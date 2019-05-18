---
title: 'Instalación de una puerta de enlace de datos local: Azure Logic Apps | Microsoft Docs'
description: Antes de poder acceder a datos locales desde Azure Logic Apps, descargue e instale la puerta de enlace de datos local
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: arthii, LADocs
ms.topic: article
ms.date: 10/01/2018
ms.openlocfilehash: 10a6e5c33f6a3c23d98e6eb3380de0d6dc6ac216
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544477"
---
# <a name="install-on-premises-data-gateway-for-azure-logic-apps"></a>Instalación de una puerta de enlace de datos local para Azure Logic Apps

Para poder conectarse a orígenes de datos locales desde Azure Logic Apps, descargue e instale la puerta de enlace de datos local en un equipo local. La puerta de enlace funciona como un puente que permite la transferencia rápida de datos y el cifrado entre orígenes de datos en el entorno local (no en la nube) y las aplicaciones lógicas. En este artículo se muestra cómo descargar, instalar y configurar la puerta de enlace de datos local. 

Puede usar la misma instalación de puerta de enlace con otros servicios, tales como Power BI, Microsoft Flow, PowerApps y Azure Analysis Services. Más información sobre [cómo funciona la puerta de enlace de datos](#gateway-cloud-service).

<a name="supported-connections"></a>

La puerta de enlace admite [conectores locales](../connectors/apis-list.md#on-premises-connectors) en Azure Logic Apps para estos orígenes de datos:

*   BizTalk Server 2016
*   Sistema de archivos
*   IBM DB2  
*   IBM Informix
*   IBM MQ
*   MySQL
*   Oracle Database
*   PostgreSQL
*   Servidor de aplicaciones de SAP 
*   Servidor de mensajes de SAP
*   SharePoint Server
*   SQL Server
*   Teradata

Para información sobre cómo usar la puerta de enlace con otros servicios, consulte estos artículos:

* [Puerta de enlace de datos local de Microsoft Power BI](https://powerbi.microsoft.com/documentation/powerbi-gateway-onprem/)
* [Administración de una puerta de enlace de datos local en Microsoft PowerApps](https://powerapps.microsoft.com/tutorials/gateway-management/)
* [Puerta de enlace de datos local de Microsoft Flow](https://flow.microsoft.com/documentation/gateway-manage/)
* [Puerta de enlace de datos local de Azure Analysis Services](../analysis-services/analysis-services-gateway.md)

<a name="requirements"></a>

## <a name="prerequisites"></a>Requisitos previos

* Una [cuenta profesional o educativa](../active-directory/fundamentals/sign-up-organization.md) con una [suscripción de Azure](https://docs.microsoft.com/azure/architecture/cloud-adoption/governance/resource-consistency/azure-resource-access) 

  Durante la instalación de puerta de enlace, inicie sesión en esta cuenta para poder asociarla a su suscripción de Azure. 
  Más adelante, usará también la misma cuenta al crear un recurso de Azure para la instalación de puerta de enlace en Azure Portal. 
  Si aún no tiene ninguna suscripción de Azure, <a href="https://azure.microsoft.com/free/" target="_blank">regístrese para obtener una cuenta gratuita de Azure</a>.

* Estos son los requisitos para el equipo local:

  **Requisitos mínimos**

  * .NET Framework 4.5.2
  * versión de 64 bits de Windows 7 o Windows Server 2008 R2 (o posterior)

  **Requisitos recomendados**

  * CPU de 8 núcleos
  * 8 GB de memoria
  * Versión de 64 bits de Windows Server 2012 R2 (o posterior)

* **Consideraciones importantes**

  * Solo puede instalar la puerta de enlace de datos local en un equipo local, no en un controlador de dominio. Aunque no tiene que instalar la puerta de enlace en el mismo equipo que el origen de datos. Además, solo necesita una puerta de enlace para todos los orígenes de datos, por lo que no es necesario instalar la puerta de enlace para cada origen de datos.

    > [!TIP]
    > Para minimizar la latencia, puede instalar la puerta de enlace lo más cerca posible del origen de datos o en el mismo equipo, suponiendo que tiene los permisos necesarios.

  * Instale la puerta de enlace en un equipo que esté conectado a Internet, siempre activado y *no* entre al modo de suspensión. En caso contrario, no se podrá ejecutar la puerta de enlace. 
  Además, el rendimiento podría verse afectado en una red inalámbrica.

  * Durante la instalación, solo puede iniciar sesión con una [cuenta profesional o educativa](../active-directory/sign-up-organization.md) que administre Azure Active Directory (Azure AD), como por ejemplo, @contoso.onmicrosoft.com, y no con una cuenta de Azure B2B (invitado) ni con una cuenta personal de Microsoft, como @hotmail.com o @outlook.com. 
  Asegúrese de utilizar la misma cuenta de inicio de sesión cuando registre la instalación de su puerta de enlace en Azure Portal creando un recurso de puerta de enlace. 
  Luego puede seleccionar este recurso de puerta de enlace al crear la conexión de la aplicación lógica con el origen de datos local. 
  [¿Por qué debo usar una cuenta profesional o educativa de Azure AD?](#why-azure-work-school-account)

  > [!TIP]
  > Si se suscribió a una oferta de Office 365 y no indicó el correo electrónico real del trabajo, puede que tenga una dirección de inicio de sesión similar a la de este ejemplo: `username@domain.onmicrosoft.com` 
  >
  > Para usar una cuenta de Microsoft que tenga una [suscripción a Visual Studio Standard](https://visualstudio.microsoft.com/vs/pricing/), [cree primero un directorio (inquilino) en Azure Active Directory](../active-directory/develop/quickstart-create-new-tenant.md) o use el directorio predeterminado con su cuenta de Microsoft. 
  > Agregue un usuario con una contraseña al directorio y, a continuación, conceda acceso a la suscripción a ese usuario. 
  > A continuación, puede iniciar sesión durante la instalación de la puerta de enlace con este nombre de usuario y contraseña.

  * La región que seleccione para la instalación de puerta de enlace determinará la ubicación en la que más adelante registrará la puerta de enlace en Azure mediante la creación de un recurso de Azure. 
  Al crear este recurso de puerta de enlace en Azure, debe seleccionar la *misma* ubicación de la instalación de la puerta de enlace. La región predeterminada es la misma ubicación del inquilino de Azure AD, que administra La cuenta de Azure, pero la ubicación se puede cambiar durante la instalación de puerta de enlace.

  * Si ya tiene una puerta de enlace que configuró con un instalador cuya versión es anterior a 14.16.6317.4, no puede cambiar la ubicación de la puerta de enlace ejecutando el instalador más reciente. Pero puede usar el instalador más reciente para instalar una nueva puerta de enlace con la ubicación que desee.
  
    Si tiene un instalador de puerta de enlace cuya versión es anterior a 14.16.6317.4, pero no ha instalado todavía la puerta de enlace, puede descargar y usar el instalador más reciente.

## <a name="high-availability-support"></a>Alta disponibilidad

La puerta de enlace de datos local admite alta disponibilidad cuando se tienen varias instalaciones de puerta de enlace y se configuran como clústeres. Si ya hay una puerta de enlace existente cuando vaya a crear otra puerta de enlace, tiene la posibilidad de crear clústeres de alta disponibilidad. Estos clústeres organizan las puertas de enlace en grupos que pueden ayudar a evitar únicos puntos de error. Además, todos los conectores de la puerta de enlace de datos local admiten ahora alta disponibilidad.

Para utilizar la puerta de enlace de datos local, revise estos requisitos y consideraciones:

* Debe tener al menos una instalación de puerta de enlace en la misma suscripción de Azure que la puerta de enlace principal y la clave de recuperación para esa instalación. 

* La puerta de enlace principal debe ejecutar la actualización de puerta de enlace de noviembre de 2017 o posterior.

Después de cumplir estos requisitos, cuando cree su próxima puerta de enlace, seleccione **Add to an existing gateway cluster** (Agregar a un clúster de puerta de enlace existente), seleccione la puerta de enlace principal para su clúster e indique la clave de recuperación de esa puerta de enlace principal.
Para obtener más información, consulte [Clústeres de alta disponibilidad para puerta de enlace a datos local](https://docs.microsoft.com/power-bi/service-gateway-high-availability-clusters).

<a name="install-gateway"></a>

## <a name="install-data-gateway"></a>Instalación de la puerta de enlace de datos

1. [Descargue, guarde y ejecute el instalador de puerta de enlace en un equipo local](https://aka.ms/on-premises-data-gateway-installer).

2. Acepte la ruta de instalación predeterminada o especifique la ubicación en el equipo donde quiere instalar la puerta de enlace.

3. Revise y acepte los términos de uso y la declaración de privacidad y luego elija **Instalar**.

   ![Aceptación de los términos de uso y la declaración de privacidad](./media/logic-apps-gateway-install/accept-terms.png)

4. Cuando la puerta de enlace esté correctamente instalada, indique la dirección de correo electrónico de su cuenta profesional o educativa y elija **Iniciar sesión**.

   ![Inicio de sesión con una cuenta profesional o educativa](./media/logic-apps-gateway-install/sign-in-gateway-install.png)

5. Elija **Registrar una nueva puerta de enlace en este equipo** > **Siguiente**, lo que registra la instalación de puerta de enlace en el [servicio en la nube de la puerta de enlace](#gateway-cloud-service). 

   ![Registro de la puerta de enlace](./media/logic-apps-gateway-install/register-new-gateway.png)

6. Facilite esta información para la instalación de puerta de enlace:

   * El nombre que quiere para la instalación 

   * La clave de recuperación que quiere crear, que debe tener al menos ocho caracteres

     > [!IMPORTANT]
     > Guarde y conserve la clave de recuperación en un lugar seguro. Necesitará esta clave cuando cambie la ubicación de la puerta de enlace o cuando migre, recupere o asuma el control de una puerta de enlace existente.

   * Confirmación de la clave de recuperación 

     ![Configuración de la puerta de enlace](./media/logic-apps-gateway-install/set-up-gateway.png)

7. Busque la región seleccionada para el servicio en la nube de la puerta de enlace y la instancia de Azure Service Bus que se usa en la instalación de puerta de enlace. 

   ![Comprobación de región](./media/logic-apps-gateway-install/check-region.png)

   > [!IMPORTANT]
   > Para cambiar esta región después de instalar la puerta de enlace, necesitará la clave de recuperación de la instalación de esa puerta de enlace. Además, debe desinstalar y volver a instalar la puerta de enlace. Para más información, vea [Instalación de la puerta de enlace de datos local para Azure Logic Apps](#update-gateway-installation).

   *¿Por qué cambiar la región de la instalación de puerta de enlace?* 

   Por ejemplo, para reducir la latencia, puede cambiar la región de la puerta de enlace a la misma región que la aplicación lógica. 
   O bien, puede seleccionar la región más cercana al origen de datos local. 
   El *recurso de puerta de enlace de Azure* y la aplicación lógica pueden tener ubicaciones distintas.

8. Para aceptar la región predeterminada, elija **Configurar**. O bien, para cambiar la región predeterminada, siga estos pasos:

   1. Junto a la región actual, seleccione **Cambiar la región**. 

      ![Cambio de región](./media/logic-apps-gateway-install/change-region.png)

   2. En la siguiente página, abra la lista **Seleccionar región**, seleccione la región que quiera y luego elija **Listo**.

      ![Seleccione otra región.](./media/logic-apps-gateway-install/select-region-gateway-install.png)

9. Cuando aparezca la página de confirmación, elija **Cerrar**. 

   El instalador confirma que la puerta de enlace está ahora conectada y lista para su uso.

   ![Puerta de enlace finalizada](./media/logic-apps-gateway-install/finished-gateway-default-location.png)

10. Registre ahora la puerta de enlace en Azure mediante la [creación de un recurso de Azure para la instalación de puerta de enlace](../logic-apps/logic-apps-gateway-connection.md). 

<a name="update-gateway-installation"></a>

## <a name="change-location-migrate-restore-or-take-over-existing-gateway"></a>Cambio de ubicación, migración, restauración o control de una puerta de enlace existente

Si tiene que cambiar la ubicación de la puerta de enlace, trasladar la instalación de puerta de enlace a otro equipo, recuperar una puerta de enlace dañada o asumir la propiedad de una puerta de enlace existente, necesitará la clave de recuperación que se le facilitó durante la instalación de puerta de enlace. Esta acción desconecta la puerta de enlace anterior.

1. Desde el **Panel de control** de su equipo, vaya a **Programas y características**. En la lista de programas, seleccione **Puerta de enlace de datos local** y luego elija **Desinstalar**.

2. [Reinstale la puerta de enlace de datos local](https://aka.ms/on-premises-data-gateway-installer).

3. Cuando el instalador se abra, inicie sesión con la misma cuenta profesional o educativa que se usó anteriormente para instalar la puerta de enlace.

4. Seleccione **Migrate, restore, or takeover an existing gateway** (Migrar, restaurar o asumir el control de una puerta de enlace existente) y luego elija **Siguiente**.

   ![Seleccione "Migrate, restore, or takeover an existing gateway" (Migrar, restaurar o asumir el control de una puerta de enlace existente).](./media/logic-apps-gateway-install/migrate-recover-take-over-gateway.png)

5. En **Puertas de enlace disponibles** o **Clústeres de puerta de enlace disponibles**, seleccione la instalación de puerta de enlace que quiere cambiar. Escriba la clave de recuperación de la instalación de puerta de enlace. 

   ![Selección de la puerta de enlace principal](./media/logic-apps-gateway-install/select-existing-gateway.png)

6. Para cambiar la región, seleccione **Cambiar la región** y luego elija la nueva región.

7. Cuando termine, seleccione, **Configurar**.

## <a name="configure-proxy-or-firewall"></a>Configuración de proxy o firewall

La puerta de enlace de datos local crea una conexión de salida con [Azure Service Bus](https://azure.microsoft.com/services/service-bus/). Si su entorno de trabajo requiere que el tráfico pase a través de un proxy para tener acceso a internet, esta restricción podría impedir la conexión de la puerta de enlace de datos al servicio en la nube de la puerta de enlace. Para determinar si la red usa un proxy, consulte este artículo en superuser.com: 

[¿Cómo sé cuál es el servidor proxy que uso? (SuperUser.com)](https://superuser.com/questions/346372/how-do-i-know-what-proxy-server-im-using) 

Para proporcionar información del proxy para la puerta de enlace, consulte [Configuración de proxy](https://docs.microsoft.com/power-bi/service-gateway-proxy). Para comprobar si el proxy o firewall puede bloquear conexiones, confirme que la máquina puede conectarse realmente a Internet y a [Azure Service Bus](https://azure.microsoft.com/services/service-bus/). Desde un símbolo del sistema de PowerShell, ejecute este comando:

`Test-NetConnection -ComputerName watchdog.servicebus.windows.net -Port 9350`

> [!NOTE]
> Este comando solo comprueba la conectividad de red y la conectividad a Azure Service Bus. El comando no realiza nada con la puerta de enlace ni con el servicio en la nube de la puerta de enlace que cifra y almacena las credenciales y los detalles de la puerta de enlace. 
>
> Además, este comando solo está disponible en Windows Server 2012 R2 o posterior y Windows 8.1 o posterior. En versiones anteriores del sistema operativo, puede usar Telnet para probar la conectividad. Obtenga más información sobre [Azure Service Bus y soluciones híbridas](../service-bus-messaging/service-bus-messaging-overview.md).

Los resultados deben ser similares a este ejemplo donde **TcpTestSucceeded** está establecido en **True**:

```text
ComputerName           : watchdog.servicebus.windows.net
RemoteAddress          : 70.37.104.240
RemotePort             : 5672
InterfaceAlias         : vEthernet (Broadcom NetXtreme Gigabit Ethernet - Virtual Switch)
SourceAddress          : 10.120.60.105
PingSucceeded          : False
PingReplyDetails (RTT) : 0 ms
TcpTestSucceeded       : True
```

Si el valor de **TcpTestSucceeded** no es **True**, la puerta de enlace podría estar bloqueada por un firewall. Si quiere ser detallado, sustituya los valores de **ComputerName** y **Port** por los que aparecen en [Configuración de los puertos](#configure-ports) en este artículo.

El firewall también podría estar bloqueando las conexiones de Azure Service Bus con los centros de datos de Azure. Si se produce este escenario, apruebe (desbloquee) todas las direcciones IP de esos centros de datos en su región. Para esas direcciones IP, puede [obtener la lista de direcciones IP de Azure aquí](https://www.microsoft.com/download/details.aspx?id=41653).

## <a name="configure-ports"></a>Configuración de los puertos

La puerta de enlace crea una conexión de salida a [Azure Service Bus](https://azure.microsoft.com/services/service-bus/) y se comunica en los puertos de salida: TCP 443 (predeterminado), 5671, 5672 y del 9350 al 9354. La puerta de enlace no requiere puertos de entrada. Obtenga más información sobre [Azure Service Bus y soluciones híbridas](../service-bus-messaging/service-bus-messaging-overview.md).

La puerta de enlace utiliza estos nombres de dominio completo:

| Nombres de dominio | Puertos de salida | DESCRIPCIÓN | 
| ------------ | -------------- | ----------- | 
| *. analysis.windows.net | 443 | HTTPS | 
| *.core.windows.net | 443 | HTTPS | 
| *.frontend.clouddatahub.net | 443 | HTTPS | 
| *.login.windows.net | 443 | HTTPS | 
| *.microsoftonline-p.com | 443 | Se utiliza para la autenticación dependiendo de la configuración. | 
| *.msftncsi.com | 443 | Se utiliza para probar la conectividad a Internet cuando el servicio Power BI no puede acceder a la puerta de enlace. | 
| *.servicebus.windows.net | 443, 9350-9354 | Agentes de escucha de Service Bus Relay sobre TCP (requiere 443 para la adquisición del token de Access Control) | 
| *.servicebus.windows.net | 5671-5672 | Advanced Message Queuing Protocol (AMQP) | 
| login.microsoftonline.com | 443 | HTTPS | 
||||

En algunos casos, las conexiones a Azure Service Bus se realizarán con direcciones IP y no con nombres de dominio completos. Por lo que tal vez quiera crear una lista blanca de direcciones IP de la región de datos en el firewall. Para crear una lista blanca de direcciones IP en lugar de dominios, puede descargar y usar la [lista de intervalos IP de centros de datos de Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653). Las direcciones IP de esta lista están en la notación [Classless Inter-Domain Routing (CIDR)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

### <a name="force-https-communication-with-azure-service-bus"></a>Aplicación forzosa de comunicación HTTPS con Azure Service Bus

Algunos servidores proxy solo permiten el tráfico a los puertos 80 y 443. De forma predeterminada, la comunicación con Azure Service Bus se produce en puertos distintos al 443.
Puede hacer que la puerta de enlace se comunique con Azure Service Bus mediante HTTPS en lugar de TCP directo, pero esto puede reducir en gran medida el rendimiento. Para realizar esta tarea, siga estos pasos:

1. Vaya a la ubicación del cliente de puerta de enlace de datos local, que normalmente se encuentra aquí: ```C:\Program Files\On-premises data gateway\Microsoft.PowerBI.EnterpriseGateway.exe```

   En caso contrario, para buscar la ubicación del cliente, abra la consola de servicios en el mismo equipo, busque **On-premises data gateway service** (Servicio de puerta de enlace de datos local) y vea la propiedad **Path to executable** (Ruta de acceso al archivo ejecutable).

2. Abra este archivo de *configuración*: **Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config**

3. Cambie el valor de **ServiceBusSystemConnectivityModeString** de **AutoDetect** a **Https**:

   ```html
   <setting name="ServiceBusSystemConnectivityModeString" serializeAs="String">
      <value>Https</value>
   </setting>
   ```

<a name="windows-service-account"></a>

## <a name="windows-service-account"></a>Cuenta de servicio de Windows

En el equipo donde se instala la puerta de enlace de datos local, esta se ejecuta como una cuenta de servicio de Windows denominada "Servicio de puerta de enlace de datos local". Sin embargo, la puerta de enlace usa el nombre de "NT SERVICE\PBIEgwService" para las credenciales de su cuenta "Iniciar sesión como". De forma predeterminada, la puerta de enlace tiene permisos "Iniciar sesión como servicio" en el equipo donde se instala la puerta de enlace. Esta cuenta de servicio de Windows de la puerta de enlace no es la misma que se usa para conectarse a orígenes de datos locales, ni la cuenta profesional o educativa que se usa para iniciar sesión en servicios en la nube.

Para crear y mantener la puerta de enlace en Azure Portal, la cuenta de servicio de Windows debe tener al menos permisos de **Colaborador**. Para comprobar estos permisos, consulte [Administración del acceso mediante RBAC y Azure Portal](../role-based-access-control/role-assignments-portal.md). 

<a name="restart-gateway"></a>

## <a name="restart-gateway"></a>Reinicio de la puerta de enlace

La puerta de enlace de datos se ejecuta como servicio de Windows, así que, al igual que con cualquier otro servicio de Windows, puede iniciar y detener la puerta de enlace de varias maneras. Por ejemplo, puede abrir un símbolo del sistema con permisos elevados en el equipo en el que se ejecuta la puerta de enlace y ejecutar cualquiera de estos comandos:

* Para detener el servicio, ejecute este comando:
  
  `net stop PBIEgwService`

* Para iniciar el servicio, ejecute este comando:
  
  `net start PBIEgwService`

## <a name="tenant-level-administration"></a>Administración en el nivel de inquilino 

En este momento no hay un único lugar donde los administradores de inquilinos puedan administrar todas las puertas de enlace que otros usuarios han instalado y configurado. Si es administrador de inquilinos, tal vez quiera que los usuarios de la organización le agreguen como administrador para cada puerta de enlace que instalen. Así, puede administrar todas las puertas de enlace de la organización a través de la página de configuración de la puerta de enlace o mediante [comandos de PowerShell](https://docs.microsoft.com/power-bi/service-gateway-high-availability-clusters#powershell-support-for-gateway-clusters). 

<a name="gateway-cloud-service"></a>

## <a name="how-does-the-gateway-work"></a>¿Cómo funciona la puerta de enlace?

La puerta de enlace de datos facilita la comunicación rápida y segura entre la aplicación lógica, el servicio en la nube de la puerta de enlace y el origen de datos local. El servicio en la nube de la puerta de enlace cifra y almacena los detalles de la puerta de enlace y las credenciales del origen de datos. El servicio también enruta las consultas y sus resultados entre la aplicación lógica, la puerta de enlace de datos local y el origen de datos local. 

La puerta de enlace funciona con firewalls y solo usa conexiones de salida. Todo el tráfico se origina como tráfico de salida seguro desde el agente de puerta de enlace. La puerta de enlace retransmite datos desde orígenes locales en canales cifrados hasta Azure Service Bus. Service Bus crea un canal entre la puerta de enlace y el servicio que realiza la llamada, pero no almacena ningún dato. Todos los datos que pasan por la puerta de enlace se cifran.

![diagram-for-on-premises-data-gateway-flow](./media/logic-apps-gateway-install/how-on-premises-data-gateway-works-flow-diagram.png)

Estos pasos describen lo que sucede cuando el usuario en la nube interactúa con un elemento conectado a un origen de datos local:

1. El servicio en la nube de la puerta de enlace crea una consulta, junto con las credenciales cifradas del origen de datos y la envía a la cola de la puerta de enlace por procesar.

2. El servicio en la nube de la puerta de enlace analiza la consulta e inserta la solicitud en la instancia de Azure Service Bus.

3. La puerta de enlace de datos local sondea Azure Service Bus en busca de solicitudes pendientes.

4. La puerta de enlace obtiene la consulta, descifra las credenciales y se conecta al origen de datos con ellas.

5. La puerta de enlace envía la consulta al origen de datos para su ejecución.

6. Los resultados se envían del origen de datos a la puerta de enlace y, luego, al servicio en la nube de la puerta de enlace. Seguidamente, el servicio en la nube de la puerta de enlace usa los resultados.

<a name="faq"></a>

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="general"></a>General

**P**: ¿Necesito una puerta de enlace para orígenes de datos en la nube como, por ejemplo, Azure SQL Database? <br/>
**R.**: No, las puertas de enlace se conectan únicamente a orígenes de datos locales.

**P.**: ¿La puerta de enlace debe estar instalada en la misma máquina que el origen de datos? <br/>
**R.**: No, la puerta de enlace se conecta al origen de datos con la información de conexión que se proporcionó. Considere la puerta de enlace como una aplicación cliente en este sentido. La puerta de enlace solo necesita la funcionalidad para conectarse al nombre de servidor que se proporcionó.

<a name="why-azure-work-school-account"></a>

**P.**: ¿Por qué hay que usar una cuenta profesional o educativa para iniciar sesión? <br/>
**R.**: Solo puede usar una cuenta profesional o educativa al instalar la puerta de enlace de datos local. Su cuenta de inicio de sesión se almacena en un inquilino administrado por Azure Active Directory (Azure AD). Por lo general, el nombre principal de usuario (UPN) de la cuenta de Azure AD coincide con la dirección de correo electrónico.

**P**: ¿Dónde se almacenan mis credenciales? <br/>
**R.**: Las credenciales que especifique para un origen de datos se cifran y almacenan en el servicio en la nube de la puerta de enlace. Las credenciales se descifran en la puerta de enlace de datos local.

**P**: ¿Hay algún requisito con respecto al ancho de banda de red? <br/>
**R.**: Compruebe que la conexión de red tiene buen rendimiento. Cada entorno es diferente, y la cantidad de datos que se envían puede incidir en los resultados. Para garantizar un nivel de rendimiento entre el origen de datos local y los centros de datos de Azure, pruebe [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/). Para ayudar a medir su rendimiento, pruebe una herramienta externa como, por ejemplo, Azure Speed Test.

**P.**: ¿cuál es la latencia para la ejecución de consultas en un origen de datos desde la puerta de enlace? ¿Cuál es la mejor arquitectura? <br/>
**R**: Para reducir la latencia de red, instale la puerta de enlace lo más cerca posible del origen de datos. Si puede instalar la puerta de enlace en el propio origen de datos, esta proximidad minimizará la latencia introducida. Además, tenga en cuenta la proximidad a los centros de datos de Azure. Por ejemplo, si el servicio usa el centro de datos Oeste de EE. UU. y tiene SQL Server hospedado en una máquina virtual de Azure, tal vez quiera que esta se encuentre también en la región Oeste de EE. UU. Esta proximidad minimiza la latencia y evita los cargos de salida en la máquina virtual de Azure.

**P**: ¿Cómo se devuelven los resultados a la nube? <br/>
**R.**: Los resultados se envían a través de Azure Service Bus.

**P.**: ¿Hay alguna conexión de entrada a la puerta de enlace desde la nube? <br/>
**R.**: No, la puerta de enlace usa conexiones de salida con Azure Service Bus.

**P.**: ¿Qué sucede si bloqueo las conexiones de salida? ¿Qué tengo que abrir? <br/>
**R**: Consulte los puertos y los hosts que usa la puerta de enlace.

**P**: ¿Cómo se llama el servicio real de Windows? <br/>
**R.**: En la pestaña Servicios del Administrador de tareas, el nombre del servicio es "PBIEgwService" o servicio Power BI Gateway - Enterprise. En la consola de servicios, el nombre del servicio es "On-premises data gateway service" (Servicio de puerta de enlace de datos local). El servicio de Windows utiliza "NT SERVICE\PBIEgwService" como SID del servicio (SSID).

**P.**: ¿Se puede ejecutar el servicio de Windows de puerta de enlace con una cuenta de Azure Active Directory? <br/>
**R.**: No, el servicio de Windows debe disponer de una cuenta de Windows válida.

### <a name="disaster-recovery"></a>Recuperación ante desastres

**P.**: ¿Cuáles son las opciones de recuperación ante desastres disponibles? <br/>
**R**: Puede usar la clave de recuperación para restaurar o mover una puerta de enlace. Cuando instale la puerta de enlace, especifique la clave de recuperación.

**P**: ¿Qué ventaja aporta la clave de recuperación? <br/>
**R**: La clave de recuperación proporciona una forma de migrar o recuperar la configuración de la puerta de enlace después de un desastre.

## <a name="troubleshooting"></a>solución de problemas

En esta sección se tratan algunos problemas comunes que pueden surgir al configurar y usar la puerta de enlace de datos local.

**P.**: ¿Por qué no se pudo realizar la instalación de puerta de enlace? <br/>
**R.**: Este problema puede suceder si el software antivirus del equipo de destino no está actualizado. Puede actualizar el software antivirus o deshabilitarlo (solo durante la instalación de puerta de enlace) y luego volver a habilitarlo.

**P.**: ¿Por qué no puedo ver mi instalación de puerta de enlace al crear el recurso de puerta de enlace en Azure? <br/>
**R.**: Este problema puede deberse a estos motivos:

* La instalación de puerta de enlace está registrada y ha sido reclamada por otro recurso de puerta de enlace en Azure. Las instalaciones de puerta de enlace no aparecen en la lista de instancias después de crear los recursos de puerta de enlace para ellas.
Para comprobar los registros de puerta de enlace en Azure Portal, revise todos los recursos de Azure de tipo **Puertas de enlace de datos local** para *todas* las suscripciones de Azure. 

* La identidad de Azure AD para la persona que instaló la puerta de enlace difiere de la persona que inició sesión en Azure Portal. Compruebe que ha iniciado sesión con la misma identidad que instaló la puerta de enlace.

[!INCLUDE [existing-gateway-location-changed](../../includes/logic-apps-existing-gateway-location-changed.md)]

**P.**: ¿Dónde están los registros de la puerta de enlace? <br/>
**R.**: Vea la sección [**Registros**](#logs) más adelante en este artículo.

**P.**: ¿Cómo se pueden ver las consultas que se envían al origen de datos local? <br/>
**R**: Puede habilitar el seguimiento de consultas, que incluye las consultas que se envían. No olvide devolver el seguimiento de consultas al valor original cuando haya terminado de solucionar problemas. Si lo deja activado, crea registros de mayor tamaño.

También puede examinar las herramientas de que dispone su origen de datos para el seguimiento de consultas. Por ejemplo, puede utilizar Eventos extendidos o SQL Profiler en SQL Server y Analysis Services.

### <a name="outdated-gateway-version"></a>Versión obsoleta de la puerta de enlace

Pueden surgir muchos problemas cuando la versión de la puerta de enlace deja de estar actualizada. En general, es conveniente asegurarse de utilizar la versión más reciente. Si no ha actualizado la puerta de enlace durante un mes o más, debería plantearse instalar su versión más reciente y comprobar si puede reproducir el problema.

### <a name="error-failed-to-add-user-to-group--2147463168-pbiegwservice-performance-log-users"></a>Error: No se pudo agregar el usuario al grupo. (Usuarios del registro de rendimiento de-2147463168 PBIEgwService)

Es posible que reciba este error si intenta instalar la puerta de enlace en un controlador de dominio, lo cual no se admite. Asegúrese de implementar la puerta de enlace en una máquina que no sea un controlador de dominio.

<a name="logs"></a>

### <a name="logs"></a>Registros

Para ayudarle a solucionar problemas, empiece por recopilar y revisar los registros de puerta de enlace. Existen varias formas de recopilar los registros, pero la opción más sencilla después de instalar la puerta de enlace es a través de la interfaz de usuario del instalador de puerta de enlace. 

1. En el equipo, abra al instalador de puerta de enlace de datos local.
2. En el menú de la izquierda, seleccione **Diagnósticos**.
3. En **Registros de la puerta de enlace**, seleccione **Exportar registros**.

   ![Exportación de registros del instalador de puerta de enlace](./media/logic-apps-gateway-install/export-logs.png)

Estas son otras ubicaciones donde puede encontrar diversos registros:

| Tipo de registro | Ubicación | 
|----------|----------| 
| **Registros del instalador** | %localappdata%\Temp\On-premises_data_gateway_<*yyyymmdd*>.<*number*>.log | 
| **Registros de configuración** | C:\Users\<*nombreDeUsuario*>\AppData\Local\Microsoft\On-premises data gateway\GatewayConfigurator<*AAAAMMDD*>.<*número*>.log | 
| **Registros del servicio Enterprise Gateway** | C:\Users\PBIEgwService\AppData\Local\Microsoft\On-premises data gateway\Gateway<*AAAAMMDD*>.<*número*>.log | 
||| 

**Registros de eventos**

Para encontrar los registros de eventos de la puerta de enlace, siga estos pasos:

1. En el equipo con la instalación de puerta de enlace, abra el **Visor de eventos**. 
2. Expanda **Visor de eventos (local)** > **Applications and Services Logs** (Registros de aplicaciones y servicios). 
3. Seleccione **On-premises data gateway service** (Servicio de puerta de enlace de datos local).

   ![Visualización de registros de eventos de la puerta de enlace](./media/logic-apps-gateway-install/event-viewer.png)

### <a name="review-slow-query-performance"></a>Revisión del rendimiento lento de las consultas

Si le parece que las consultas se ejecutan con lentitud a través de la puerta de enlace, puede activar un registro adicional que devuelva las consultas y sus duraciones. Estos registros pueden ayudarle a encontrar las consultas que son lentas o de larga ejecución. Para optimizar el rendimiento de las consultas, es posible que tenga que modificar el origen de datos, por ejemplo, ajustar los índices para las consultas de SQL Server.

Para determinar la duración de una consulta, siga estos pasos:

1. Vaya a la ubicación del cliente de puerta de enlace, que normalmente se encuentra aquí: ```C:\Program Files\On-premises data gateway```

   En caso contrario, para buscar la ubicación del cliente, abra la consola de servicios en el mismo equipo, busque **On-premises data gateway service** (Servicio de puerta de enlace de datos local) y vea la propiedad **Path to executable** (Ruta de acceso al archivo ejecutable).

2. Abra y edite estos archivos de configuración tal y como se describe:

   * **Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config**

     En este archivo, cambie el valor de **EmitQueryTraces** de **false** a **true** para que la puerta de enlace pueda registrar las consultas enviadas desde la puerta de enlace a un origen de datos:

     ```html
     <setting name="EmitQueryTraces" serializeAs="String">
        <value>true</value>
     </setting>
     ```

     > [!IMPORTANT]
     > La activación del valor de EmitQueryTraces podría aumentar significativamente el tamaño del registro según el uso de la puerta de enlace. Cuando termine de revisar los registros, asegúrese de restablecer EmitQueryTraces a **false** de nuevo, en lugar de dejar esta configuración activada a largo plazo.

   * **Microsoft.PowerBI.DataMovement.Pipeline.Diagnostics.dll.config**

     Para que la puerta de enlace registre entradas detalladas, incluidas las entradas que muestran la duración, cambie el valor de **TracingVerbosity** de **4** a **5** realizando uno de los pasos: 

     * En este archivo de configuración, cambie el valor de **TracingVerbosity** de **4** a **5**. 

       ```html
       <setting name="TracingVerbosity" serializeAs="String">
          <value>5</value>
       </setting>
       ```

     * Abra el instalador de puerta de enlace, seleccione **Diagnósticos**, active **Registro adicional** y luego elija **Aplicar**:

       ![Activación del registro adicional](./media/logic-apps-gateway-install/turn-on-additional-logging.png)

     > [!IMPORTANT]
     > La activación del valor de TracingVerbosity podría aumentar significativamente el tamaño del registro según el uso de la puerta de enlace. Cuando termine de revisar los registros, asegúrese de desactivar la opción **Registro adicional** en el instalador de puerta de enlace o restablezca TracingVerbosity a **4** de nuevo en el archivo de configuración, en lugar de dejar esta configuración activada a largo plazo.

3. Para buscar la duración de una consulta, siga estos pasos:

   1. [Exporte](#logs) y abra el registro de puerta de enlace.

   2. Para buscar una consulta, busque un tipo de actividad, por ejemplo: 

      | Tipo de actividad | DESCRIPCIÓN | 
      |---------------|-------------| 
      | MGEQ | Consultas que se ejecutan sobre ADO.NET. | 
      | MGEO | Consultas que se ejecutan sobre OLEDB. | 
      | MGEM | Consultas que se ejecutan desde el motor de Mashup. | 
      ||| 

   3. Observe el segundo GUID, que corresponde al identificador de la solicitud.

   4. Continúe buscando el tipo de actividad hasta que encuentre una entrada denominada "FireActivityCompletedSuccessfullyEvent" que tiene una duración en milisegundos. 
   Confirme que la entrada tiene el mismo identificador de la solicitud, por ejemplo:

      ```text 
      DM.EnterpriseGateway Verbose: 0 : 2016-09-26T23:08:56.7940067Z DM.EnterpriseGateway    baf40f21-2eb4-4af1-9c59-0950ef11ec4a    5f99f566-106d-c8ac-c864-c0808c41a606    MGEQ    21f96cc4-7496-bfdd-748c-b4915cb4b70c    B8DFCF12 [DM.Pipeline.Common.TracingTelemetryService] Event: FireActivityCompletedSuccessfullyEvent (duration=5004)
      ```

      > [!NOTE] 
      > "FireActivityCompletedSuccessfullyEvent" es una entrada detallada que no se registra a menos que sea el valor de "TracingVerbosity" en el nivel 5.

### <a name="trace-traffic-with-fiddler"></a>Seguimiento del tráfico con Fiddler

[Fiddler](https://www.telerik.com/fiddler) es una herramienta gratuita de Telerik que supervisa el tráfico HTTP. Puede ver este tráfico con el servicio Power BI desde la máquina cliente. Este servicio puede mostrar errores y otra información relacionada.

## <a name="next-steps"></a>Pasos siguientes
    
* [Conexión a datos locales desde aplicaciones lógicas](../logic-apps/logic-apps-gateway-connection.md)
* [Características de Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [Conectores para Azure Logic Apps](../connectors/apis-list.md)
