---
title: Administración de Protección contra DDoS de Azure estándar mediante Azure Portal
titlesuffix: Azure Virtual Network
description: Aprenda a usar la telemetría de Protección contra DDoS de Azure estándar en Azure Monitor para mitigar un ataque.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/17/2019
ms.author: kumud
ms.openlocfilehash: 73036ba1a72d657fd07a826bbee8651781f70e9b
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931971"
---
# <a name="manage-azure-ddos-protection-standard-using-the-azure-portal"></a>Administración de Protección contra DDoS de Azure estándar mediante Azure Portal

Obtenga información acerca de cómo habilitar y deshabilitar la protección contra denegación de servicio (DDoS) distribuido y usar la telemetría para mitigar un ataque DDoS con Protección contra DDoS de Azure estándar. Protección contra DDoS de Azure estándar protege los recursos de Azure, como las máquinas virtuales, los equilibradores de carga y las puertas de enlace de aplicaciones que disponen de una [dirección IP pública](virtual-network-public-ip-address.md) de Azure asignada. Para más información sobre el estándar de protección contra DDoS y sus funcionalidades, vea [Introducción a Protección contra DDoS de Azure estándar](ddos-protection-overview.md).

Antes de completar los pasos de este tutorial, inicie sesión en Azure Portal en https://portal.azure.com con una cuenta asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un[ rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones apropiadas en [Permisos](#permissions).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-ddos-protection-plan"></a>Creación de un plan de protección contra DDoS

Un plan de protección contra DDoS define un conjunto de redes virtuales que tiene habilitada la protección contra DDoS estándar en distintas suscripciones. Puede configurar un plan de protección contra DDoS para la organización y vincular redes virtuales de distintas suscripciones al mismo plan. El propio plan de protección contra DDoS también está asociado a una suscripción, la que selecciona durante la creación del plan. El plan de DDoS Protection funciona en todas las regiones y suscripciones. Ejemplo: puede crear el plan en la Región Este de EE. UU. y vincularlo a la suscripción número 1 en su inquilino. El mismo plan se puede vincular a las redes virtuales de otras suscripciones en diferentes regiones, a través de su inquilino. La suscripción a la que está asociado el plan incurre en la factura mensual recurrente del plan, además de cargos de uso por encima del límite, en caso de que la cantidad de direcciones IP públicas protegidas supere las 100. Para más información sobre los precios de DDoS, consulte los [detalles de precios](https://azure.microsoft.com/pricing/details/ddos-protection/).

En la mayoría de las organizaciones, no es necesario crear más de un plan. Un plan no puede moverse entre suscripciones. Si desea cambiar la suscripción en que se encuentra un plan, debe [eliminar el plan existente](#work-with-ddos-protection-plans) y crear uno nuevo.

1. Seleccione **Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Busque *DDoS*. Seleccione **Plan de protección contra DDoS** cuando aparezca en los resultados de la búsqueda.
3. Seleccione **Crear**.
4. Escriba o seleccione sus propios valores, o bien escriba o seleccione los valores de ejemplo siguientes y, luego, seleccione **Crear**:

    |Configuración        |Value                                              |
    |---------      |---------                                          |
    |Nombre           | myDdosProtectionPlan                              |
    |Subscription   | Seleccione su suscripción.                         |
    |Resource group | Seleccione **Crear nuevo** y escriba *myResourceGroup* |
    |Location       | Este de EE. UU.                                           |

## <a name="enable-ddos-for-a-new-virtual-network"></a>Habilitación de DDoS para una red virtual nueva

1. Seleccione **Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Redes** y **Red virtual**.
3. Escriba o seleccione sus propios valores, o bien escriba o seleccione los valores de ejemplo siguientes, acepte el resto de los valores predeterminados y, luego, seleccione **Crear**:

    | Configuración         | Value                                                        |
    | ---------       | ---------                                                    |
    | Nombre            | myVirtualNetwork                                             |
    | Subscription    | Seleccione su suscripción.                                    |
    | Resource group  | Seleccione **Usar existente** y, luego, seleccione **myResourceGroup** |
    | Location        | Este de EE. UU.                                                      |
    | DDoS Protection Standard | Seleccione **Habilitar**. El plan que selecciona puede estar en la misma suscripción que la red virtual, o una suscripción distinta, pero ambas suscripciones deben estar asociadas al mismo inquilino de Azure Active Directory.|

No puede mover una red virtual a otro grupo de recursos ni a otra suscripción si DDoS Standard está habilitado para la red virtual. Si tiene que mover una red virtual que tenga habilitado DDoS Standard, primero deshabilítelo, mueva la red virtual y, luego, habilite DDoS Standard. Después de eso, se restablecen los umbrales de directiva ajustados automáticamente para todas las direcciones IP públicas protegidas en la red virtual.

## <a name="enable-ddos-for-an-existing-virtual-network"></a>Habilitación de DDoS para una red virtual existente

1. Cree un plan de protección contra DDoS con los pasos descritos en [Creación de un plan de protección contra DDoS](#create-a-ddos-protection-plan), si no tiene un plan de protección de DDoS existente.
2. Seleccione **Crear un recurso** en la esquina superior izquierda de Azure Portal.
3. Escriba el nombre de la red virtual para la que desea habilitar DDoS Protection Standard en el cuadro **Buscar recursos, servicios y documentos** en la parte superior del portal. Seleccione el nombre de la red virtual cuando aparezca en los resultados de la búsqueda.
4. Seleccione **Protección DDoS** en **CONFIGURACIÓN**.
5. Seleccione **Estándar**. En **Plan de protección contra DDoS**, seleccione un plan de protección contra DDoS existente o el plan que creó en el paso 1 y, luego, seleccione **Guardar**. El plan que selecciona puede estar en la misma suscripción que la red virtual, o una suscripción distinta, pero ambas suscripciones deben estar asociadas al mismo inquilino de Azure Active Directory.

**Comandos** 
- CLI de Azure: [az network ddos-protection create](https://docs.microsoft.com/cli/azure/network/ddos-protection?view=azure-cli-latest#az-network-ddos-protection-create)
- Powershell: [New-AzDdosProtectionPlan](https://docs.microsoft.com/powershell/module/Az.Network/New-AzDdosProtectionPlan?view=azps-2.8.0)
 

## <a name="disable-ddos-for-a-virtual-network"></a>Deshabilitación de DDoS para una red virtual

1. Escriba el nombre de la red virtual para la que desea deshabilitar DDoS Protection Standard en el cuadro **Buscar recursos, servicios y documentos** en la parte superior del portal. Seleccione el nombre de la red virtual cuando aparezca en los resultados de la búsqueda.
2. Seleccione **En DDoS Protection estándar**, seleccione **Deshabilitar**.

**Comandos** 
- CLI de Azure: [az network ddos-protection delete](https://docs.microsoft.com/cli/azure/network/ddos-protection?view=azure-cli-latest#az-network-ddos-protection-delete)
- Powershell: [Remove-AzDdosProtectionPlan](https://docs.microsoft.com/powershell/module/az.network/remove-azddosprotectionplan?view=azps-3.2.0)

## <a name="work-with-ddos-protection-plans"></a>Trabajo con planes de protección contra DDoS

1. Seleccione **Todos los servicios** en la parte superior izquierda del portal.
2. Escriba *DDoS* en el cuadro **Filtrar**. Seleccione **Planes de protección contra DDoS** cuando aparezca en los resultados.
3. Seleccione el plan de protección que desea ver en la lista.
4. Aparecen todas las redes virtuales asociadas al plan.
5. Si desea eliminar un plan, primero debe desasociar todas las redes virtuales que tiene asociadas. Para desasociar un plan de una red virtual, consulte [Deshabilitación de DDoS para una red virtual](#disable-ddos-for-a-virtual-network).

## <a name="configure-alerts-for-ddos-protection-metrics"></a>Configuración de alertas para métricas de protección contra DDoS

Puede seleccionar cualquiera de las métricas de protección contra DDoS disponibles para que le avisen cuando hay una mitigación activa durante un ataque mediante la configuración de alertas de Azure Monitor. Cuando se cumplan las condiciones, recibirá un correo electrónico de alerta en la dirección especificada:

1. Seleccione **Todos los servicios** en la parte superior izquierda del portal.
2. Escriba *Monitor* en el cuadro **Filtrar**. Seleccione **Monitor** cuando aparezca en los resultados.
3. Seleccione **Métricas** en **SERVICIOS COMPARTIDOS**.
4. Escriba o seleccione sus propios valores, o bien escriba los valores de ejemplo siguientes, acepte el resto de los valores predeterminados y, luego, seleccione **Aceptar**:

    |Configuración                  |Value                                                                                               |
    |---------                |---------                                                                                           |
    |Nombre                     | myDdosAlert                                                                                        |
    |Subscription             | Seleccione la suscripción que contiene la dirección IP pública para la que desea recibir alertas.        |
    |Resource group           | Seleccione el grupo de recursos que contiene la dirección IP pública para la que desea recibir alertas.      |
    |Resource                 | Seleccione la dirección IP pública que contiene la dirección IP pública para la que desea recibir alertas. DDoS supervisa las direcciones IP públicas asignadas a los recursos dentro de una red virtual. Si no tiene ningún recurso con direcciones IP públicas en la red virtual, primero debe crear un recurso con una dirección IP pública. Puede supervisar la dirección IP pública de todos los recursos implementados a través de Resource Manager (no clásico) que aparecen en [Red virtual para servicios de Azure](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network), excepto para entornos de Azure App Service y Azure VPN Gateway. Para continuar con este tutorial, puede crear rápidamente una máquina virtual [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json).                   |
    |Métrica                   | Bajo ataque de DDoS o no                                                                            |
    |Umbral                | 1: un **1** significa que está siendo atacado. Un **0** significa que no está siendo atacado.                         |
    |Período                   | Seleccione el valor que prefiera.                                                                   |
    |Notificar por correo electrónico         | Active la casilla de verificación                                                                                  |
    |Administrador adicional | Escriba su dirección de correo electrónico si no es propietario del correo electrónico, colaborador o lector de la suscripción. |

    Dentro de unos pocos minutos después de la detección del ataque, recibirá un correo electrónico con métricas de Azure Monitor que tiene un aspecto similares a la imagen siguiente:

    ![Alerta de ataque](./media/manage-ddos-protection/ddos-alert.png)


Para simular un ataque de DDoS para validar la alerta, consulte [Validación de la detección de DDoS](#validate-ddos-detection).

Puede aprender más sobre [configurar webhooks](../azure-monitor/platform/alerts-webhooks.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [aplicaciones lógicas](../logic-apps/logic-apps-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para la creación de alertas.

## <a name="use-ddos-protection-telemetry"></a>Uso de telemetría de protección contra DDoS

La telemetría para un ataque se proporciona a través de Azure Monitor en tiempo real. La telemetría está disponible solo durante el tiempo en que una dirección IP pública está en proceso de mitigación. No ve ninguna telemetría antes o después de mitigar un ataque.

1. Seleccione **Todos los servicios** en la parte superior izquierda del portal.
2. Escriba *Monitor* en el cuadro **Filtrar**. Seleccione **Monitor** cuando aparezca en los resultados.
3. Seleccione **Métricas** en **SERVICIOS COMPARTIDOS**.
4. Seleccione la **suscripción** y el **grupo de recursos** que contienen la dirección IP pública para la que desea la telemetría.
5. Seleccione la **dirección IP pública** para el **tipo de recurso**, luego seleccione la dirección IP pública específica para la que desea la telemetría.
6. En el lado izquierdo de la pantalla se muestra una serie de **métricas disponibles**. Cuando se seleccionan estas métricas, se representan en el **gráfico de métricas de Azure Monitor**, en la pantalla de información general.
7. Seleccione el tipo de **agregación** como **Máximo**.

Los nombres de las métricas presentan distintos tipos de paquetes y bytes frente a paquetes, con una construcción básica de nombres de etiqueta en cada métrica, como se muestra aquí:

- **Nombre de etiqueta de eliminado** (por ejemplo, **Paquetes de entrada quitados por DDoS**): El número de paquetes que el sistema de protección contra DDoS elimina o limpia.
- **Nombre de etiqueta de reenviado** (por ejemplo, **Paquetes de entrada reenviados por DDoS**): el número de paquetes que reenvía el sistema DDoS a la dirección IP virtual de destino (tráfico que no se filtró).
- **Sin nombre de etiqueta** (por ejemplo **DDoS de paquetes de entrada**): el número total de paquetes que han llegado al sistema de limpieza (representa la suma de los paquetes eliminados y los reenviados).

Para simular un ataque de DDoS para validar la telemetría, consulte [Validación de la detección de DDoS](#validate-ddos-detection).

## <a name="view-ddos-mitigation-policies"></a>Visualización de las directivas de mitigación de DDoS

DDoS Protection Standard aplica tres directivas de mitigación de ajuste automático (TCP SYN, TCP y UDP) a cada dirección IP pública del recurso protegido, en la red virtual que tiene habilitado DDoS. Puede ver los umbrales de directiva si selecciona las métricas **Paquetes entrantes TCP para desencadenar la mitigación de DDoS** y **Paquetes entrantes UDP para desencadenar la mitigación de DDoS** con el tipo de **agregación** como "Máximo", tal como se muestra en la siguiente imagen:

![Vista de directivas de mitigación](./media/manage-ddos-protection/view-mitigation-policies.png)

Los umbrales de las directivas se configuran automáticamente con nuestro sistema de generación de perfiles de tráfico de red basado en aprendizaje automático de Azure. Solo cuando se supera el umbral de la directiva, la mitigación de DDoS se produce para la dirección IP que está siendo atacada.

## <a name="configure-ddos-attack-analytics"></a>Configuración del análisis de ataques DDoS
La versión estándar de Azure DDoS Protection proporciona información detallada y visualización de ataques con DDoS Attack Analytics. Los clientes que protegen sus redes virtuales frente a ataques DDoS disponen de visibilidad detallada sobre el tráfico de ataques y las acciones llevadas a cabo para mitigarlos mediante informes de mitigación de ataques y registros de flujo de mitigación. 

## <a name="configure-ddos-attack-mitigation-reports"></a>Configuración de informes de mitigación de ataques DDoS
Los informes de mitigación de ataques usan los datos del protocolo Netflow que se agrega para proporcionar información detallada sobre el ataque en el recurso. Cada vez que un recurso IP público es objeto de ataque, tan pronto se inicia la mitigación se inicia la generación de informes. Durante todo el período de mitigación, habrá un informe incremental que se genera cada cinco minutos y un informe posterior a la mitigación. Su finalidad es garantizar que en caso de que el ataque DDoS continúe durante mucho tiempo, podrá ver la instantánea más actual del informe de mitigación cada cinco minutos y un resumen completo una vez finalizada la mitigación del ataque. 

1. Seleccione **Todos los servicios** en la parte superior izquierda del portal.
2. Escriba *Monitor* en el cuadro **Filtrar**. Seleccione **Monitor** cuando aparezca en los resultados.
3. En **CONFIGURACIÓN**, seleccione **Configuración de diagnóstico**.
4. Seleccione la **suscripción** y el **grupo de recursos** que contienen la dirección IP pública que desea registrar.
5. Seleccione la **dirección IP pública** para el **tipo de recurso**, luego seleccione la dirección IP pública específica para la que desea registrar las métricas.
6. Seleccione **Turn on diagnostics to collect the DDoSMitigationReports log** (Activar diagnósticos para recopilar el registro DDoSMitigationReports) y, luego, seleccione tantas opciones de las siguientes como necesite:

    - **Archivar en una cuenta de almacenamiento**: los datos se escriben en una cuenta de Azure Storage. Para más información sobre esta opción, consulte [Archivo de registros de recursos](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
    - **Transmisión a un centro de eventos**: permite que un receptor de registros seleccione los registros mediante una instancia de Azure Event Hub. Los centros de eventos habilitan la integración con Splunk y otros sistemas SIEM. Para más información sobre esta opción, consulte [Transmisión de registros de recursos a un centro de eventos](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
    - **Enviar a Log Analytics**: Escribe registros en el servicio Azure Monitor. Para obtener más información sobre esta opción, consulte [Recopilación de registros para su uso en los registros de Azure Monitor](../azure-monitor/platform/collect-azure-metrics-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Tanto los informes de mitigación incremental como los posteriores a los ataques incluyen los siguientes campos:
- Vectores de ataque
- Estadísticas de tráfico
- Motivo de paquetes descartados
- Protocolos implicados
- 10 países o regiones de origen principales
- 10 ASN de origen principales

## <a name="configure-ddos-attack-mitigation-flow-logs"></a>Configuración de registros de flujo de mitigación de ataques DDoS
Los registros de flujo de mitigación de ataques le permiten revisar casi en tiempo real el tráfico descartado, el tráfico reenviado y otros puntos de datos interesantes durante un ataque DDoS activo. Puede ingerir el flujo constante de estos datos en sus sistemas SIEM mediante un centro de eventos para la supervisión casi en tiempo real, realizar acciones y solucionar la necesidad de sus operaciones de defensa. 

1. Seleccione **Todos los servicios** en la parte superior izquierda del portal.
2. Escriba *Monitor* en el cuadro **Filtrar**. Seleccione **Monitor** cuando aparezca en los resultados.
3. En **CONFIGURACIÓN**, seleccione **Configuración de diagnóstico**.
4. Seleccione la **suscripción** y el **grupo de recursos** que contienen la dirección IP pública que desea registrar.
5. Seleccione la **dirección IP pública** para el **tipo de recurso**, luego seleccione la dirección IP pública específica para la que desea registrar las métricas.
6. Seleccione **Turn on diagnostics to collect the DDoSMitigationFlowLogs log** (Activar diagnósticos para recopilar el registro DDoSMitigationFlowLogs) y, luego, seleccione tantas opciones de las siguientes como necesite:

    - **Archivar en una cuenta de almacenamiento**: los datos se escriben en una cuenta de Azure Storage. Para más información sobre esta opción, consulte [Archivo de registros de recursos](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
    - **Transmisión a un centro de eventos**: permite que un receptor de registros seleccione los registros mediante una instancia de Azure Event Hub. Los centros de eventos habilitan la integración con Splunk y otros sistemas SIEM. Para más información sobre esta opción, consulte [Transmisión de registros de recursos a un centro de eventos](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
    - **Enviar a Log Analytics**: Escribe registros en el servicio Azure Monitor. Para obtener más información sobre esta opción, consulte [Recopilación de registros para su uso en los registros de Azure Monitor](../azure-monitor/platform/collect-azure-metrics-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
1. Para ver los datos de registros de flujo en el panel de análisis de Azure, puede importar el panel de ejemplo desde https://github.com/Anupamvi/Azure-DDoS-Protection/raw/master/flowlogsbyip.zip.

Los registros de flujo tienen los siguientes campos: 
- IP de origen
- IP de destino
- Puerto de origen 
- Puerto de destino 
- Tipo de protocolo 
- Acción realizada durante la mitigación

El análisis de ataques solo funcionará si el estándar DDoS Protection está habilitado en la red virtual de la dirección IP pública. 

## <a name="validate-ddos-detection"></a>Validación de la detección de DDoS

Microsoft se ha asociado con [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) para crear una interfaz en la que pueda generar tráfico destinado a las direcciones IP públicas que tengan habilitado el servicio DDoS Protection con fines de simulación. La simulación de BreakPoint Cloud le permite:

- Comprobar en qué medida Microsoft Azure DDoS Protection protege sus recursos de Azure frente a ataques de DDoS.
- Optimizar el proceso de respuesta a incidentes durante el ataque de DDoS.
- Documentar el cumplimiento normativo de DDoS.
- Enseñar a los equipos de seguridad de red.

## <a name="view-ddos-protection-alerts-in-azure-security-center"></a>Ver las alertas de protección DDoS en Azure Security Center

Azure Security Center proporciona una lista de [alertas de seguridad](/azure/security-center/security-center-managing-and-responding-alerts), que cuentan con información para ayudarle a investigar y solucionar problemas. Gracias a esta característica, obtendrá una vista unificada de las alertas, incluyendo las alertas relacionadas con los ataques DDoS y las acciones que se tomaron para mitigar el ataque a corto plazo.
Hay dos alertas específicas que verá en cualquier mitigación y detección de ataques DDoS:

- **Ataque DDoS detectado para IP pública**: Esta alerta se genera cuando el servicio de protección contra DDoS detecta que una de sus direcciones IP públicas es el objetivo de un ataque DDoS.
- **Ataque DDoS mitigado para IP pública**: Esta alerta se genera cuando se ha mitigado un ataque a la dirección IP pública.
Para ver las alertas, abra el **Centro de seguridad** en Azure Portal. En **Protección contra amenazas**, seleccione **Alertas de seguridad**. La siguiente captura de pantalla muestra un ejemplo de las alertas de ataques DDoS.

![Alerta de ataque DDoS en Azure Security Center](./media/manage-ddos-protection/ddos-alert-asc.png)

Las alertas incluyen información general sobre la dirección IP pública que está bajo ataque, información de inteligencia geográfica y de amenazas y los pasos para solucionar el problema.

## <a name="permissions"></a>Permisos

Para trabajar con planes de protección contra DDoS, su cuenta debe estar asignada al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un rol [personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones adecuadas que se muestran en la tabla siguiente:

| Acción                                            | Nombre                                     |
| ---------                                         | -------------                            |
| Microsoft.Network/ddosProtectionPlans/read        | Obtención de un plan de protección contra DDoS              |
| Microsoft.Network/ddosProtectionPlans/write       | Creación o actualización de un plan de protección contra DDoS  |
| Microsoft.Network/ddosProtectionPlans/delete      | Eliminación de un plan de protección contra DDoS            |
| Microsoft.Network/ddosProtectionPlans/join/action | Unión a un plan de protección contra DDoS              |

Para habilitar la protección contra DDoS para una red virtual, su cuenta también debe estar asignada a las [acciones para redes virtuales](manage-virtual-network.md#permissions) adecuadas.

## <a name="next-steps"></a>Pasos siguientes

- Creación y asignación de [definiciones de Azure Policy](policy-samples.md) para redes virtuales
