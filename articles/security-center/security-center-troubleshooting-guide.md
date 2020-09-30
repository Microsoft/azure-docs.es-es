---
title: Guía de solución de problemas de Azure Security Center | Microsoft Docs
description: Esta guía está destina a profesionales de TI, analistas de seguridad y administradores de la nube para solucionar problemas relacionados con Azure Security Center.
services: security-center
author: v-miegge
manager: dcscontentpm
ms.service: security-center
ms.topic: conceptual
ms.date: 09/10/2019
ms.author: memildin
ms.openlocfilehash: e2922d19dbcad7da2808a86896e39d21420e73d5
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904749"
---
# <a name="azure-security-center-troubleshooting-guide"></a>Guía de solución de problemas de Azure Security Center

Esta guía está destinada a profesionales de tecnologías de la información (TI), analistas de seguridad de la información y administradores de la nube cuyas organizaciones utilizan Azure Security Center y necesitan solucionar problemas relacionados con Security Center.

Security Center usa el agente de Log Analytics para recopilar y almacenar datos. Consulte [Migración de la plataforma de Azure Security Center](security-center-platform-migration.md) para más información. La información de este artículo representa la funcionalidad de Security Center después de la transición al agente de Log Analytics.

## <a name="troubleshooting-guide"></a>Guía de solución de problemas

Esta guía explica cómo solucionar problemas relacionados con Security Center.

Tipos de alerta:

* Análisis del comportamiento de la máquina virtual (VMBA)
* Análisis de red
* Análisis de SQL Database y Azure Synapse Analytics (anteriormente SQL Data Warehouse)
* Información contextual

Dependiendo de los tipos de alerta, los clientes pueden obtener la información necesaria para investigarla mediante el uso de los siguientes recursos:

* Registros de seguridad en el visor de eventos de la máquina virtual (VM) en Windows
* AuditD en Linux
* Los registros de actividad de Azure y la opción de habilitar los registros de diagnóstico en el recurso del ataque.

Los clientes pueden compartir comentarios para la descripción de la alerta y la pertinencia. Navegue a la propia alerta, seleccione el botón **¿Le resultó útil?** , seleccione el motivo y escriba un comentario que sirva de ex. Supervisamos sistemáticamente este canal de comentarios para mejorar nuestras alertas.

## <a name="audit-log"></a>Registro de auditoría

La mayoría de las soluciones de problemas en Security Center se realizan examinando primero el [registro de auditoría](../azure-monitor/platform/platform-logs-overview.md) del componente afectado. A través de los registros de auditoría puede determinar:

* Qué operaciones se han llevado a cabo
* Quién inició la operación
* Cuándo tuvo lugar la operación
* El estado de la operación
* Los valores de otras propiedades que podrían ayudarle en la investigación de la operación

El registro de auditoría contiene todas las operaciones de escritura (PUT, POST, DELETE) realizadas en los recursos, pero no incluye las operaciones de lectura (GET).

## <a name="log-analytics-agent"></a>Agente de Log Analytics

Security Center usa el agente de Log Analytics, que es el mismo agente que usa el servicio Azure Monitor, para recopilar datos de seguridad de las máquinas virtuales de Azure. Una vez que se ha habilitado la recopilación de datos y se ha instalado correctamente el agente en la máquina de destino, los siguientes procesos deben estar en ejecución:

* HealthService.exe

Si abre la consola de administración de servicios (services.msc), también verá el servicio de agente de Log Analytics que se ejecuta como se muestra a continuación:

![Servicios](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig5.png)

Para ver qué versión del agente tiene, abra **Administrador de tareas**, en la pestaña **Procesos** busque el servicio **Agente de Log Analytics**, haga clic en él con el botón derecho y haga clic en **Propiedades**. En la pestaña **Detalles**, examine la versión del archivo tal y como se muestra a continuación:

![Archivo](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig6.png)

## <a name="log-analytics-agent-installation-scenarios"></a>Escenarios de instalación del agente de Log Analytics

Existen dos escenarios de instalación que pueden producir resultados diferentes al instalar el agente de Log Analytics en el equipo. Los escenarios admitidos son:

* **Agente instalado automáticamente por Security Center**: en este escenario podrá ver las alertas en ambas ubicaciones: Security Center y la búsqueda de registros. Recibirá notificaciones por correo electrónico a la dirección que configuró en la directiva de seguridad para la suscripción a la que pertenece el recurso.

* **Agente instalado manualmente en una VM ubicada en Azure**: en este escenario, si está usando los agentes descargados e instalados antes de febrero de 2017, puede ver las alertas en el portal de Security Center solo si filtra por la suscripción a la que pertenece el área de trabajo. Si filtra por la suscripción a la que pertenece el recurso, no verá ninguna alerta. Recibirá notificaciones por correo electrónico a la dirección que configuró en la directiva de seguridad para la suscripción a la que pertenece el área de trabajo.

> [!NOTE]
> Para evitar el comportamiento explicado en el segundo caso, asegúrese de descargar la versión más reciente del agente.

## <a name="monitoring-agent-health-issues"></a>Supervisión de problemas de Agent Health <a name="mon-agent"></a>

**Estado de supervisión** define el motivo por el que Security Center no puede supervisar correctamente las máquinas virtuales y los equipos inicializados para el aprovisionamiento automático. La tabla siguiente muestra los valores, las descripciones y los pasos de resolución de **Estado de supervisión**.

| Estado de supervisión | Descripción | Pasos de resolución |
|---|---|---|
| Instalación del agente pendiente | La instalación del agente de Log Analytics aún se está ejecutando.  La instalación puede tardar unas horas. | Espere hasta que finalice la instalación automática. |
| Estado de energía: desactivado | La VM está detenida.  El agente de Log Analytics solo puede instalarse en una máquina virtual que esté en ejecución. | Reinicie la VM. |
| Falta el agente de VM de Azure o no es válido | El agente de Log Analytics todavía no está instalado.  Es necesario un agente de VM de Azure válido para que Security Center instale la extensión. | Instale, reinstale o actualice el agente de VM de Azure en la VM. |
| Estado de máquina virtual no preparada para la instalación  | El agente de Log Analytics no está instalado todavía porque la máquina virtual no está lista para la instalación. La máquina virtual no está lista para la instalación debido a un problema con el agente de la máquina virtual o con el aprovisionamiento de máquinas virtuales. | Compruebe el estado de la máquina virtual. Vuelva a **Máquinas virtuales** en el portal y seleccione la máquina virtual para obtener información del estado. |
|Error de instalación: error general | El agente de Log Analytics se instaló, pero se produjo un error. | [Instale manualmente la extensión](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension) o desinstale la extensión para que Security Center intente instalarla de nuevo. |
| Error de instalación: agente local ya instalado | Error de instalación del agente de Log Analytics. Security Center ha identificado que un agente local (Log Analytics o System Center Operations Manager) ya está instalado en la máquina virtual. Para evitar la configuración de hospedaje múltiple, donde la máquina virtual informa a dos áreas de trabajo independientes, se detiene la instalación del agente de Log Analytics. | Hay dos maneras de resolverlo: [instalar manualmente la extensión](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension) y conectarla al área de trabajo deseada. O bien, establecer el área de trabajo deseada como el área de trabajo predeterminada y habilitar el aprovisionamiento automático del agente.  Consulte [Habilitación del aprovisionamiento automático](security-center-enable-data-collection.md). |
| El agente no puede conectarse al área de trabajo | El agente de Log Analytics se instaló, pero se produjo un error debido a la conectividad de red.  Compruebe que el sistema tiene acceso a Internet o que se ha configurado un servidor proxy HTTP válido para el agente. | Consulte los requisitos de red del agente de supervisión. |
| Agente conectado a un área de trabajo desconocida o no encontrada | Security Center identificó que el agente de Log Analytics instalado en la máquina virtual está conectado a un área de trabajo a la que no tiene acceso. | Esto puede ocurrir en dos casos. El área de trabajo se ha eliminado y ya no existe. Vuelva a instalar al agente con el área de trabajo correcta o desinstale el agente y permita que Security Center complete la instalación de aprovisionamiento automático. El segundo caso se da cuando el área de trabajo forma parte de una suscripción para la que Security Center no tiene permisos. Security Center requiere que las suscripciones permitan el acceso al proveedor de recursos de seguridad de Microsoft. Para habilitarlo, registre la suscripción en el proveedor de recursos de seguridad de Microsoft. Esto se puede hacer mediante una API, PowerShell, el portal o, simplemente, filtrando por la suscripción en el panel **Información general** de Security Center. Para más información, consulte [Proveedores de recursos y sus tipos](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal). |
| El agente no responde o falta el identificador | Security Center no puede recuperar los datos de seguridad escaneados de la máquina virtual, incluso aunque el agente está instalado. | El agente no notifica ningún dato, incluidos los latidos. El agente puede estar dañado o algo está bloqueando el tráfico. O bien, el agente está proporcionando datos, pero falta un identificador de recurso de Azure, por lo que es imposible relacionar los datos con la máquina virtual de Azure. Para solucionar problemas en Linux, vea la [Guía de solución de problemas del Agente de Log Analytics para Linux](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/Troubleshooting.md#im-not-seeing-any-linux-data-in-the-oms-portal). Para solucionar problemas de Windows, consulte [Solución de problemas con máquinas virtuales Windows](https://github.com/MicrosoftDocs/azure-docs/blob/8c53ac4371d482eda3d85819a4fb8dac09996a89/articles/log-analytics/log-analytics-azure-vm-extension.md#troubleshooting-windows-virtual-machines). |
| Agente no instalado | La colección de datos está deshabilitada. | Active la colección de datos en la directiva de seguridad o instale manualmente el agente de Log Analytics. |

## <a name="troubleshooting-monitoring-agent-network-requirements"></a>Solución de problemas relativos a los requisitos de red del agente de supervisión <a name="mon-network-req"></a>

Para que los agentes se puedan conectar a Security Center y registrarse ahí, deben tener acceso a los recursos de red, lo que incluye los números de puerto y las direcciones URL de dominio.

* Para los servidores proxy, debe asegurarse de que los recursos de servidor proxy adecuados están configurados en la configuración del agente. Lea este artículo para más información sobre [cómo cambiar la configuración del servidor proxy](https://docs.microsoft.com/azure/log-analytics/log-analytics-windows-agents).
* Si usa un firewall para restringir el acceso a Internet, debe configurarlo para permitir el acceso a Log Analytics. No es necesario realizar ninguna acción en la configuración del agente.

En la siguiente tabla se muestran los recursos necesarios para la comunicación.

| Recurso del agente | Puertos | Omitir inspección de HTTPS |
|---|---|---|
| *.ods.opinsights.azure.com | 443 | Sí |
| *.oms.opinsights.azure.com | 443 | Sí |
| *.blob.core.windows.net | 443 | Sí |
| *.azure-automation.net | 443 | Sí |

Si experimenta problemas con la incorporación del agente, asegúrese de leer el artículo [Solución de problemas de incorporación en Operations Management Suite](https://support.microsoft.com/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues).

## <a name="troubleshooting-endpoint-protection-not-working-properly"></a>La solución de problemas de Endpoint Protection no funciona correctamente

El agente invitado es el proceso primario de todo lo que hace la extensión [Microsoft Antimalware](../security/fundamentals/antimalware.md). Cuando se produce un error en el proceso del agente invitado, es posible que se produzca un error en la instancia de Microsoft Antimalware que se ejecuta como proceso secundario del agente invitado.  En escenarios como este se recomienda comprobar las opciones siguientes:

* Si la máquina virtual de destino es una imagen personalizada y el creador de la máquina virtual nunca instaló el agente invitado.
* Si el destino es una máquina virtual de Linux en lugar de una máquina virtual de Windows, se producirá un error en la instalación de la versión de Windows de la extensión del antimalware en una máquina virtual de Linux. El agente invitado Linux tiene requisitos específicos en términos de la versión de SO y los paquetes necesarios y si no se cumplen esos requisitos, el agente de máquina virtual no funcionará ahí tampoco.
* Si la máquina virtual se creó con una versión anterior del agente invitado. Si así fue, debe tener en cuenta que es posible que algunos agentes anteriores no realicen actualizaciones automáticas a la versión más recientes, lo que podría generar este problema. Use siempre la versión más reciente del agente invitado si crea sus propias imágenes.
* Algunos software de administración de terceros podrían deshabilitar el agente invitado o bloquear el acceso a ciertas ubicaciones de archivos. Si tiene instalado este tipo de software de terceros en la máquina virtual, asegúrese de que el agente esté en la lista de exclusiones.
* Ciertos ajustes del firewall o un grupo de seguridad de red (NSG) pueden bloquear el tráfico de red desde y hacia el agente invitado.
* Cierta lista de control de acceso (ACL) puede impedir el acceso al disco.
* La falta de espacio en disco puede impedir que el agente invitado funcione correctamente.

De manera predeterminada, la interfaz de usuario de Microsoft Antimalware está deshabilitada, lea [Enabling Microsoft Antimalware User Interface on Azure Resource Manager VMs Post Deployment](https://blogs.msdn.microsoft.com/azuresecurity/2016/03/09/enabling-microsoft-antimalware-user-interface-post-deployment/) (Habilitación de la interfaz de usuario de Microsoft Antimalware en máquinas virtuales de Azure Resource Manager tras la implementación) para más información sobre cómo habilitarla si necesita hacerlo.

## <a name="troubleshooting-problems-loading-the-dashboard"></a>Solución de problemas al cargar el panel

Si experimenta problemas al cargar el panel de Security Center, asegúrese de que el usuario que registra la suscripción a Security Center (es decir, el primer usuario que abrió Security Center con la suscripción) y el usuario que desearía activar la recopilación de datos son *propietarios* o *colaboradores* en la suscripción. A partir de ese momento, los usuarios *lectores* en la suscripción también podrán ver el panel, las alertas, las recomendaciones y las directivas.

## <a name="contacting-microsoft-support"></a>Contacto con el soporte técnico de Microsoft

Algunos problemas pueden identificarse mediante las instrucciones proporcionadas en este artículo; también puede encontrar otros en la [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-security-center.html) pública de Security Center. Sin embargo, si necesita más información para solucionar el problema, puede abrir una nueva solicitud de soporte técnico mediante **Azure Portal**, como se indica a continuación:

![Soporte técnico de Microsoft](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig2.png)

## <a name="see-also"></a>Consulte también

En este documento ha aprendido a configurar directivas de seguridad en Azure Security Center. Para obtener más información sobre Azure Security Center, consulte los siguientes recursos:

* [Guía de planeamiento y operaciones de Azure Security Center](security-center-planning-and-operations-guide.md): aprenda a planear y conozca las consideraciones de diseño necesarias para usar Azure Security Center.
* [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md): obtenga información sobre cómo supervisar el estado de los recursos de Azure.
* [Administración y respuesta a las alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md): obtenga información sobre cómo administrar y responder a alertas de seguridad.
* [Descripción de las alertas de seguridad en Azure Security Center](security-center-alerts-type.md)
* [Tutorial: Respuesta a incidentes de seguridad](tutorial-security-incident.md)
* [Validación de alertas en Azure Security Center](security-center-alert-validation.md)
* [Notificaciones de correo electrónico en Azure Security Center](security-center-provide-security-contact-details.md)
* [Control de incidentes de seguridad en Azure Security Center](security-center-incident.md)
* [Funcionalidades de detección de Azure Security Center](security-center-detection-capabilities.md)
* [Supervisión de las soluciones de asociados con Azure Security Center](security-center-partner-solutions.md): aprenda a supervisar el estado de mantenimiento de las soluciones de asociados.
* [Preguntas más frecuentes sobre Azure Security Center](faq-general.md): encuentre las preguntas más frecuentes sobre el uso del servicio.
* [Blog de seguridad de Azure](https://docs.microsoft.com/archive/blogs/azuresecurity/): encuentre entradas de blog sobre el cumplimiento y la seguridad de Azure.
