---
title: Seguridad de datos de Log Analytics | Microsoft Docs
description: Obtenga información sobre cómo Log Analytics protege su privacidad y sus datos.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/04/2019
ms.openlocfilehash: ef34dbfd3af326dbf2d82e09a4c5c8c8e4a91a84
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319803"
---
# <a name="log-analytics-data-security"></a>Seguridad de datos de Log Analytics
Este documento está diseñado para proporcionar información específica sobre Log Analytics, que es una característica de Azure Monitor, para complementar la información que se halla en el [Centro de confianza de Azure](https://www.microsoft.com/en-us/trust-center?rtc=1).  

En este artículo, se explica cómo Log Analytics recopila, procesa y protege los datos. Puede usar a agentes para conectarse al servicio web, utilizar System Center Operations Manager con el fin de recopilar datos operativos o recuperar datos de Diagnósticos de Azure para que Log Analytics los utilice. 

El servicio Log Analytics administra sus datos basados en la nube de forma segura mediante los siguientes métodos:

* Segregación de datos
* Retención de datos
* Seguridad física
* Administración de incidentes
* Cumplimiento normativo
* Certificaciones de estándares de seguridad

Póngase en contacto con nosotros si tiene preguntas, sugerencias o problemas acerca de la siguiente información, incluidas nuestras directivas de seguridad en [Opciones de soporte técnico de Azure](https://azure.microsoft.com/support/options/).

## <a name="sending-data-securely-using-tls-12"></a>Envío de datos de forma segura con TLS 1.2 

Para garantizar la seguridad de los datos en tránsito a Log Analytics, se recomienda encarecidamente configurar el agente para que use al menos Seguridad de la capa de transporte (TLS) 1.2. Las versiones anteriores de TLS/Capa de sockets seguros (SSL) han demostrado ser vulnerables y, si bien todavía funcionan para permitir la compatibilidad con versiones anteriores, **no se recomiendan**, y el sector está cambiando rápidamente para abandonar la compatibilidad de estos protocolos más antiguos. 

[PCI Security Standards Council](https://www.pcisecuritystandards.org/) ha establecido una [fecha límite de 30 de junio de 2018](https://www.pcisecuritystandards.org/pdfs/PCI_SSC_Migrating_from_SSL_and_Early_TLS_Resource_Guide.pdf) para deshabilitar las versiones anteriores de TLS/SSL y actualizar a protocolos más seguros. Una vez que Azure quite la compatibilidad heredada, si los agentes no pueden comunicarse a través de TLS 1.2 como mínimo, podría no ser capaz de enviar datos a Log Analytics. 

No se recomienda establecer explícitamente el agente para que solo use TLS 1.2, a menos que sea absolutamente necesario, ya que esto puede interrumpir las características de seguridad a nivel de la plataforma que le permiten detectar y aprovechar automáticamente las ventajas de los protocolos más seguros más recientes a medida que estén disponibles, como TLS 1.3. 

### <a name="platform-specific-guidance"></a>Guía específica para la plataforma

|Plataforma/lenguaje | Soporte técnico | Más información |
| --- | --- | --- |
|Linux | Las distribuciones de Linux tienden a basarse en [OpenSSL](https://www.openssl.org) para la compatibilidad con TLS 1.2.  | Compruebe el [registro de cambios de OpenSSL](https://www.openssl.org/news/changelog.html) para confirmar si su versión de OpenSSL es compatible.|
| Windows 8.0 a 10 | Compatible y habilitado de manera predeterminada. | Para confirmar que aún usa la [configuración predeterminada](/windows-server/security/tls/tls-registry-settings).  |
| Windows Server 2012 a 2016 | Compatible y habilitado de manera predeterminada. | Para confirmar que aún usa la [configuración predeterminada](/windows-server/security/tls/tls-registry-settings) |
| Windows 7 SP1 y Windows Server 2008 R2 SP1 | Compatible, pero no habilitado de manera predeterminada. | Consulte la página [Configuración del registro de TLS](/windows-server/security/tls/tls-registry-settings) para obtener más información sobre cómo se habilita.  |

## <a name="data-segregation"></a>Segregación de datos
Una vez que el servicio Log Analytics ha ingerido los datos, estos se mantienen de forma independiente en cada componente del servicio. Todos los datos se etiquetan por área de trabajo. Este etiquetado persiste a lo largo del ciclo de vida de los datos y se aplica en cada nivel del servicio. Los datos se almacenan en una base de datos dedicada en el clúster de almacenamiento de la región que ha seleccionado.

## <a name="data-retention"></a>Retención de datos
Los datos indexados de búsqueda de registros se almacenan y se conservan conforme al plan de precios. Para obtener más información, consulte [Precios de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).

Como parte del [acuerdo de suscripción](https://azure.microsoft.com/support/legal/subscription-agreement/), Microsoft conservará sus datos conforme a los términos del contrato.  Cuando se eliminan los datos de cliente, no se destruyen unidades físicas.  

En la tabla siguiente se muestran algunas de las soluciones disponibles y se proporcionan ejemplos de los tipos de datos que recopilan.

| **Solución** | **Tipos de datos** |
| --- | --- |
| Capacidad y rendimiento |Datos de rendimiento y metadatos |
| Administración de actualizaciones |Metadatos y datos de estado |
| Administración de registros |Registros IIS, de eventos de Windows o de eventos definidos por el usuario |
| Seguimiento de cambios |Inventario de software, servicio de Windows y metadatos de demonio de Linux, y metadatos de archivo de Windows o Linux |
| Evaluación de SQL y Active Directory |Datos de WMI, datos de registro, datos de rendimiento y resultados de la vista de administración dinámica de SQL Server |

La tabla siguiente muestra ejemplos de los tipos de datos:

| **Tipo de datos** | **Fields** |
| --- | --- |
| Alerta |Alert Name, Alert Description, BaseManagedEntityId, Problem ID, IsMonitorAlert, RuleId, ResolutionState, Priority, Severity, Category, Owner, ResolvedBy, TimeRaised, TimeAdded, LastModified, LastModifiedBy, LastModifiedExceptRepeatCount, TimeResolved, TimeResolutionStateLastModified, TimeResolutionStateLastModifiedInDB, RepeatCount |
| Configuración |CustomerID, AgentID, EntityID, ManagedTypeID, ManagedTypePropertyID, CurrentValue, ChangeDate |
| Evento |EventId, EventOriginalID, BaseManagedEntityInternalId, RuleId, PublisherId, PublisherName, FullNumber, Number, Category, ChannelLevel, LoggingComputer, EventData, EventParameters, TimeGenerated, TimeAdded <br>**Nota:** Cuando se escriben eventos con campos personalizados en el registro de eventos de Windows, Log Analytics los recopila. |
| Metadatos |BaseManagedEntityId, ObjectStatus, OrganizationalUnit, ActiveDirectoryObjectSid, PhysicalProcessors, NetworkName, IPAddress, ForestDNSName, NetbiosComputerName, VirtualMachineName, LastInventoryDate, HostServerNameIsVirtualMachine, IP Address, NetbiosDomainName, LogicalProcessors, DNSName, DisplayName, DomainDnsName, ActiveDirectorySite, PrincipalName, OffsetInMinuteFromGreenwichTime |
| Rendimiento |ObjectName, CounterName, PerfmonInstanceName, PerformanceDataId, PerformanceSourceInternalID, SampleValue, TimeSampled, TimeAdded |
| State |StateChangeEventId, StateId, NewHealthState, OldHealthState, Context, TimeGenerated, TimeAdded, StateId2, BaseManagedEntityId, MonitorId, HealthState, LastModified, LastGreenAlertGenerated, DatabaseTimeModified |

## <a name="physical-security"></a>Seguridad física
El servicio Log Analytics está a cargo de personal de Microsoft y todas las actividades se registran y se pueden auditar. Log Analytics funciona como un servicio de Azure y cumple con todos los requisitos de cumplimiento y seguridad de Azure. Puede ver detalles acerca de la seguridad física de los recursos de Azure en la página 18 de la [Información general de la seguridad de Microsoft Azure](https://download.microsoft.com/download/6/0/2/6028B1AE-4AEE-46CE-9187-641DA97FC1EE/Windows%20Azure%20Security%20Overview%20v1.01.pdf). Los derechos de acceso físico para proteger áreas de las personas que ya no tienen ninguna responsabilidad en el servicio de Log Analytics se cambian en un plazo de un día hábil, incluidas la transferencia y la finalización. Puede consultar información sobre la infraestructura física global que se utiliza en los [centros de datos de Microsoft](https://azure.microsoft.com/global-infrastructure/).

## <a name="incident-management"></a>Administración de incidentes
Log Analytics incluye un proceso de administración de incidentes que cumplen todos los servicios de Microsoft. Resumiendo:

* Usamos un modelo de responsabilidad compartida donde una parte de la responsabilidad de seguridad reside en Microsoft y otra parte pertenece al cliente.
* Administramos los incidentes de seguridad de Azure:
  * Iniciamos una investigación tras la detección de un incidente.
  * Un miembro de guardia del equipo de respuesta a incidentes evalúa el impacto y la gravedad de un incidente. En función de las pruebas, la evaluación puede dar lugar a que el incidente se remita al equipo de respuesta de seguridad.
  * Diagnostique un incidente gracias a los expertos de respuesta de seguridad para realizar la investigación forense o técnica, e identificar las estrategias de contención, mitigación y solución. Si el equipo de seguridad considera que los datos de cliente pueden quedar expuestos a una persona no autorizada, la ejecución en paralelo del proceso de notificación de incidentes de cliente.  
  * Estabilice y recupere la infraestructura después del incidente. El equipo de respuesta a incidentes crea un plan de recuperación para mitigar el problema. Pueden llevarse a cabo procesos de gestión de crisis, como poner en cuarentena sistemas afectados, de inmediato y en paralelo al diagnóstico. Pueden planearse más mitigaciones a largo plazo para realizarse después de que ha pasado el riesgo inmediato.  
  * Cierre el incidente y realice un análisis final. El equipo de respuesta a incidentes lleva a cabo un análisis final donde se describen los detalles del incidente con el objetivo de revisar las directivas, los procedimientos y los procesos para evitar que se vuelva a producir el evento.
* Notificamos a los clientes los incidentes de seguridad:
  * Determine el alcance de los incidentes y notifique de la forma más detallada posible a todos las personas afectadas.
  * Cree un aviso para los clientes con la información detallada suficiente para que puedan realizar una investigación y cumplir los compromisos con sus usuarios finales sin retrasar indebidamente el proceso de notificación.
  * Confirme y declare el incidente según sea necesario.
  * Envíe a los clientes una notificación de incidentes puntual y de acuerdo con cualquier compromiso contractual o jurídico. La notificación de incidentes de seguridad se entrega a uno o varios de los administradores de un cliente por los medios que Microsoft elija, entre ellos el correo electrónico.
* Formamos y preparamos al equipo:
  * El personal de Microsoft tiene que finalizar el curso sobre seguridad y concienciación, para que puedan identificar y notificar los posibles problemas de seguridad.  
  * Los operadores que trabajan en el servicio de Microsoft Azure tienen más obligaciones de aprendizaje relacionadas con el acceso a los sistemas confidenciales que hospedan los datos de cliente.
  * El personal de respuesta de seguridad de Microsoft recibe cursos especializados para los roles que desempeñan.

En caso de pérdida de datos de cualquier cliente, se notificará a cada cliente en el plazo de un día. Sin embargo, con este servicio nunca se han perdido datos de cliente. 

Para obtener más información sobre cómo Microsoft responde a los incidentes de seguridad, consulte [Microsoft Azure Security Response in the Cloud](https://gallery.technet.microsoft.com/Azure-Security-Response-in-dd18c678/file/150826/4/Microsoft%20Azure%20Security%20Response%20in%20the%20cloud.pdf) (Respuesta de seguridad de Microsoft Azure en la nube).

## <a name="compliance"></a>Cumplimiento normativo
El programa de gobernanza y seguridad de la información del equipo de servicio y desarrollo del software de Log Analytics respalda sus requisitos empresariales y cumple las leyes y los reglamentos, tal y como se describe en el [Centro de confianza de Microsoft Azure](https://azure.microsoft.com/support/trust-center/) y en [Microsoft Trust Center Compliance](https://www.microsoft.com/en-us/trustcenter/compliance/default.aspx). En estas páginas también se describe cómo Log Analytics establece los requisitos de seguridad, identifica los controles de seguridad, y administra y supervisa los riesgos. Cada año, revisamos las directivas, los estándares, los procedimientos y las instrucciones.

Cada miembro del equipo de desarrollo recibe cursos formales sobre seguridad de aplicaciones. Internamente, utilizamos un sistema de control de versiones para el desarrollo de software. Cada proyecto de software se protege mediante el sistema de control de versiones.

Microsoft cuenta con un equipo de seguridad y cumplimiento normativo que supervisa y evalúa todos los servicios de Microsoft. Los responsables de seguridad de la información conforman el equipo y no están vinculados con los equipos de ingeniería que desarrollan Log Analytics. Los responsables de seguridad tienen su propia cadena de administración y llevan a cabo evaluaciones independientes de los productos y servicios para garantizar la seguridad y el cumplimiento normativo.

Al comité de dirección de Microsoft se le notifica mediante informes anuales sobre todos los programas de información de seguridad de Microsoft.

Los equipos de servicio y desarrollo de software de Log Analytics trabajan de manera activa con los equipos de cumplimiento y asuntos jurídicos de Microsoft, así como con otros asociados del sector, para adquirir diversas certificaciones.

## <a name="certifications-and-attestations"></a>Certificaciones y atestaciones
Azure Log Analytics cumple los requisitos siguientes:

* [ISO/IEC 27001](https://www.iso.org/iso/home/standards/management-standards/iso27001.htm)
* [ISO/IEC 27018:2014](https://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=61498)
* [ISO 22301](https://azure.microsoft.com/blog/iso22301/)
* [Conformidad con la norma Data Security Standard de PCI (PCI DSS)](https://www.microsoft.com/en-us/TrustCenter/Compliance/PCI) del PCI Security Standards Council.
* [Conformidad con Service Organization Control (SOC) 1, tipo 1 y SOC 2, tipo 1](https://www.microsoft.com/en-us/TrustCenter/Compliance/SOC1-and-2)
* [HIPAA y HITECH](https://www.microsoft.com/en-us/TrustCenter/Compliance/hipaa) para aquellas empresas que posean un contrato de asociación comercial según las normas HIPAA.
* Criterios de ingeniería comunes de Windows
* Microsoft Trustworthy Computing
* Como se trata de un servicio de Azure, los componentes que usa Log Analytics cumplen los requisitos de cumplimiento normativo de Azure. Puede obtener más información en el artículo [Microsoft Trust Center Compliance](https://www.microsoft.com/en-us/trustcenter/compliance/default.aspx) (Cumplimiento normativo del Centro de confianza de Microsoft Azure).

> [!NOTE]
> En algunas certificaciones o atestaciones, Log Analytics aparece con su nombre anterior, *Operational Insights*.
>
>

## <a name="cloud-computing-security-data-flow"></a>Flujo de datos de seguridad de informática en la nube
En el diagrama siguiente se muestra una arquitectura de seguridad en la nube como el flujo de información proveniente de la empresa y cómo se protege a medida que se mueve al servicio Log Analytics y al que, en última instancia, ve usted en Azure Portal. Después del diagrama aparece más información acerca de cada paso.

![Imagen de recopilación de datos de Log Analytics y seguridad](./media/data-security/log-analytics-data-security-diagram.png)

## <a name="1-sign-up-for-log-analytics-and-collect-data"></a>1. Suscripción a Log Analytics y recopilación de datos
Para que su organización envíe datos a Log Analytics, puede configurar un agente de Windows o Linux que se ejecute en máquinas virtuales de Azure, o en equipos físicos o virtuales de su entorno o de otro proveedor en la nube.  Si usa Operations Manager, desde el grupo de administración puede configurar el agente de Operations Manager. Los usuarios (que pueden ser usted, otros usuarios individuales o un grupo de personas) deben crear una o más áreas de trabajo de Log Analytics y registrar los agentes mediante una de las siguientes cuentas:

* [Id. de organización](../../active-directory/fundamentals/sign-up-organization.md)
* [Cuenta de Microsoft: Outlook, Office Live, MSN](https://account.microsoft.com/account)

Un área de trabajo de Log Analytics es donde se recopilan, agregan, analizan y presentan los datos. Se usa principalmente como un medio para realizar la partición de datos y cada uno de ellos es único. Por ejemplo, recomendamos que los datos de producción se administren con un área de trabajo de Log Analytics, y los datos de prueba, con otro diferente. Los espacios de trabajo también ayudan a un administrador a controlar el acceso de los usuarios a los datos. Cada área de trabajo puede tener, a su vez, varias cuentas de usuario asociadas y viceversa: cada cuenta de usuario puede tener varias áreas de trabajo de Log Analytics. Las áreas de trabajo se crean en función de la región del centro de datos.

En lo que respecta a Operations Manager, el grupo de administración de este establece una conexión con el servicio Log Analytics. Después, puede configurar qué sistemas administrados por un agente del grupo de administración pueden recopilar y enviar datos al servicio. Dependiendo de la solución habilitada, los datos de estas soluciones se envían directamente desde un servidor de administración de Operations Manager al servicio Log Analytics o bien, debido al volumen de los datos recopilados en el sistema administrado por agentes, se envían directamente desde el agente al servicio. En el caso de los sistemas que no supervisa Operations Manager, cada uno se conecta directamente y de forma segura al servicio Log Analytics.

Toda la comunicación se cifra entre sistemas conectados y el servicio Log Analytics. El protocolo TLS (HTTPS) se utiliza para el cifrado.  Se sigue el proceso del SDL de Microsoft para garantizar que el servicio Log Analytics está actualizado con los últimos avances en protocolos criptográficos.

Cada tipo de agente recopila datos para Log Analytics. El tipo de datos recopilados depende de los tipos de soluciones usadas. Puede ver un resumen del proceso de recolección de datos en el artículo [Incorporación de soluciones de Log Analytics desde la galería de soluciones](../insights/solutions.md). Además, hay información más detallada disponible sobre este tema para la mayoría de las soluciones. Una solución es una agrupación de vistas predefinidas, consultas de búsqueda de registros, reglas de recopilación de datos y lógica de procesamiento. Solo los administradores pueden usar Log Analytics para importar una solución. Una vez importada, la solución se transfiere a los servidores de administración de Operations Manager (en caso de utilizarse) y, después, a los agentes que haya elegido. Posteriormente, los agentes recopilan los datos.

## <a name="2-send-data-from-agents"></a>2. Envío de datos desde agentes
El registro de todos los tipos de agentes se efectúa con una clave de inscripción y se establece una conexión segura entre el agente en cuestión y el servicio Log Analytics mediante la autenticación basada en certificados y TLS con el puerto 443. Log Analytics utiliza un almacén secreto para generar y mantener las claves. Las claves privadas se alternan cada 90 días, y se almacenan en Azure y administran a través de las operaciones de Azure que siguen las estrictas prácticas regulatorias y de cumplimiento normativo.

Con Operations Manager, el grupo de administración registrado con un área de trabajo de Log Analytics establece una conexión HTTPS segura con un servidor de administración de Operations Manager.

En lo que respecta a los agentes de Windows o de Linux que se ejecutan en máquinas virtuales de Azure, se utiliza una clave de almacenamiento de solo lectura para leer los eventos de diagnóstico en las tablas de Azure.  

En el caso de agentes que envían notificaciones a un grupo de administración de Operations Manager que está integrado con Log Analytics, si el servidor de administración no se puede comunicar con el servicio por cualquier motivo, los datos recopilados se almacenan localmente en una memoria caché temporal del servidor de administración.   Intentarán volver a enviar los datos cada ocho minutos durante dos horas.  Para los datos que eluden el servidor de administración y se envían directamente a Log Analytics, el comportamiento es coherente con el agente de Windows.  

Los datos almacenados en la memoria caché del agente del servidor de administración o de Windows se protegen mediante el almacén de credenciales del sistema operativo. Si el servicio no puede procesar los datos después de dos horas, los agentes pondrán en cola los datos. Si la cola se llena, el agente empieza a quitar tipos de datos, empezando por los datos de rendimiento. El límite de cola de los agentes es una clave de registro que puede modificar si es necesario. Los datos recopilados se comprimen y se envían al servicio, omitiendo las bases de datos del grupo de administración de Operations Manager, por lo que no se les agrega ninguna carga. Después de enviar los datos recopilados, se quitan de la memoria caché.

Como se describió anteriormente, los datos del servidor de administración o de los agentes conectados directamente se envían por TLS a los centros de datos de Microsoft Azure. También puede usar ExpressRoute para proporcionar más seguridad a los datos. Con ExpressRoute puede conectarse directamente a Azure desde una red WAN, como una VPN de conmutación por etiquetas multiprotocolo (MPLS) que proporcione un proveedor de servicios de red. Para obtener más información, consulte el artículo sobre [ExpressRoute](https://azure.microsoft.com/services/expressroute/).

## <a name="3-the-log-analytics-service-receives-and-processes-data"></a>3. Recepción y procesamiento de los datos por parte del servicio Log Analytics
El servicio Log Analytics asegura que los datos entrantes provienen de una fuente de confianza mediante la validación de certificados y la integridad de los datos con la autenticación de Azure. Los datos sin procesar se almacenarán posteriormente en una instancia de Azure Event Hubs en la región en que finalmente se almacenarán los datos en reposo. El tipo de datos que se almacena depende de los tipos de soluciones que se importaron y se usaron para recopilar datos. Después, el servicio Log Analytics procesa los datos sin procesar y los ingiere en la base de datos.

El período de retención de los datos recopilados que se almacenan en la base de datos depende del plan de precios seleccionado. Para el nivel *Gratis*, los datos recopilados están disponibles durante siete días. Para el nivel de *pago*, los datos recopilados están disponibles durante 31 días de forma predeterminada, pero se puede ampliar a 730 días. Los datos se almacenan cifrados en reposo en Azure Storage, para garantizar la confidencialidad de los datos, y los datos se replican dentro de la región local con almacenamiento con redundancia local (LRS). Las dos últimas semanas de datos también se almacenan en caché basada en SSD y esta caché está cifrada.

## <a name="4-use-log-analytics-to-access-the-data"></a>4. Uso de Log Analytics para acceder a los datos
Para acceder al área de trabajo de Log Analytics puede iniciar sesión en Azure Portal mediante la cuenta de la organización o la cuenta Microsoft configurada anteriormente. Todo el tráfico que circula entre el portal y el servicio Log Analytics se envía por un canal seguro HTTPS. Al usar el portal, se genera un identificador de sesión en el cliente del usuario (explorador web) y los datos se almacenan en una caché local hasta que finalice la sesión. Cuando termina, se elimina la memoria caché. Las cookies del cliente, que no contienen información de identificación personal, no se quitan automáticamente. Las de sesión se marcan con HTTPOnly y se protegen. Después de un periodo de inactividad predeterminado, se termina la sesión de Azure Portal.

## <a name="next-steps"></a>Pasos siguientes
* Aprenda a recopilar datos con Log Analytics para las máquinas virtuales de Azure con la [guía de inicio rápido de máquinas virtuales de Azure](../learn/quick-collect-azurevm.md).  

*  Si desea recopilar datos desde equipos físicos o virtuales de Windows o Linux del entorno, consulte la [guía de inicio rápido para equipos con Linux](../learn/quick-collect-linux-computer.md) o la [guía de inicio rápido para equipos con Windows](../learn/quick-collect-windows-computer.md)

