---
title: Conectar datos de eventos de seguridad de Windows a Azure Sentinel| Microsoft Docs
description: Aprenda a usar el conector de eventos de seguridad para transmitir todos los eventos de seguridad de sus sistemas Windows al área de trabajo de Azure Sentinel. 
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: d51d2e09-a073-41c8-b396-91d60b057e6a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: a16afcafa03ef2ab8642316db560e30a473a526b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883706"
---
# <a name="connect-windows-security-events"></a>Conexión de eventos de seguridad de Windows 

El conector de eventos de seguridad permite transmitir todos los eventos de seguridad de los sistemas Windows (servidores y estaciones de trabajo, físicos y virtuales) al área de trabajo de Azure Sentinel. Esto le permite ver los eventos de seguridad de Windows en los paneles. De este modo, puede usarlos en la creación de alertas personalizadas y basarse en ellos para mejorar las investigaciones, lo que le proporciona más información sobre la red de su organización y amplía las funciones de las operaciones de seguridad. Puede seleccionar los eventos que se transmitirán de entre los siguientes conjuntos: <a name="event-sets"></a>

- **Todos los eventos**: todos los eventos de seguridad de Windows y AppLocker.
- **Común**: conjunto estándar de eventos con fines de auditoría. En este conjunto se proporciona una pista de auditoría de usuario completa. Por ejemplo, contiene los eventos de inicio de sesión y cierre de sesión de usuario (identificadores de evento 4624, 4634). También hay acciones de auditoría como los cambios en los grupos de seguridad, las operaciones de Kerberos en el controlador de dominio de clave y otros tipos de eventos que siguen los procedimientos recomendados aceptados.

    El conjunto de eventos **común** puede contener algunos tipos de eventos que no son tan comunes.  Esto se debe a que el argumento principal del conjunto **común** es reducir el volumen de eventos a un nivel más fácil de administrar, a la vez que se mantiene una capacidad de registro de auditoría completa.

- **Mínimo**: conjunto reducido de eventos que pueden señalar posibles amenazas. Este conjunto no contiene un registro de auditoría completo. Abarca solamente los eventos que podrían indicar una brecha correcta, así como otros eventos importantes que tienen tasas de aparición muy bajas. Por ejemplo, contiene un inicio de sesión de usuario correcto y uno erróneo (identificadores de evento 4624 y 4625), pero no contiene información del cierre de sesión (4634), que es importante para la auditoría pero no lo es para la detección de brechas y tiene un volumen relativamente alto. La mayor parte del volumen de datos de este conjunto está conformado por los eventos de inicio de sesión y de creación de proceso (identificador de evento 4688).

- **Ninguno**: no se transmite ningún evento de seguridad o de AppLocker. (Esta configuración se usa para deshabilitar el conector).

    En la lista siguiente se muestra un desglose completo de los identificadores de evento de seguridad y de AppLocker para cada conjunto:

    | Conjunto de eventos | Identificadores de eventos recopilados |
    | --- | --- |
    | **Mínimo** | 1102, 4624, 4625, 4657, 4663, 4688, 4700, 4702, 4719, 4720, 4722, 4723, 4724, 4727, 4728, 4732, 4735, 4737, 4739, 4740, 4754, 4755, 4756, 4767, 4799, 4825, 4946, 4948, 4956, 5024, 5033, 8001, 8002, 8003, 8004, 8005, 8006, 8007, 8222 |
    | **Común** | 1, 299, 300, 324, 340, 403, 404, 410, 411, 412, 413, 431, 500, 501, 1100, 1102, 1107, 1108, 4608, 4610, 4611, 4614, 4622, 4624, 4625, 4634, 4647, 4648, 4649, 4657, 4661, 4662, 4663, 4665, 4666, 4667, 4688, 4670, 4672, 4673, 4674, 4675, 4689, 4697, 4700, 4702, 4704, 4705, 4716, 4717, 4718, 4719, 4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4733, 4732, 4735, 4737, 4738, 4739, 4740, 4742, 4744, 4745, 4746, 4750, 4751, 4752, 4754, 4755, 4756, 4757, 4760, 4761, 4762, 4764, 4767, 4768, 4771, 4774, 4778, 4779, 4781, 4793, 4797, 4798, 4799, 4800, 4801, 4802, 4803, 4825, 4826, 4870, 4886, 4887, 4888, 4893, 4898, 4902, 4904, 4905, 4907, 4931, 4932, 4933, 4946, 4948, 4956, 4985, 5024, 5033, 5059, 5136, 5137, 5140, 5145, 5632, 6144, 6145, 6272, 6273, 6278, 6416, 6423, 6424, 8001, 8002, 8003, 8004, 8005, 8006, 8007, 8222, 26401, 30004 |

> [!NOTE]
> La recopilación de eventos de seguridad dentro del contexto de un área de trabajo única se puede configurar desde Azure Security Center o desde Azure Sentinel, pero no con ambas herramientas. Si va a incorporar Azure Sentinel a un área de trabajo que ya recibe alertas de Azure Defender desde Azure Security Center y está configurado para recopilar eventos de seguridad, tiene dos opciones:
> - Deje la recopilación de eventos de seguridad en Azure Security Center tal y como está. Podrá consultar y analizar estos eventos tanto en Azure Sentinel como en Azure Defender. Sin embargo, no podrá supervisar el estado de conectividad del conector ni cambiar su configuración en Azure Sentinel. Si esta acción le parece importante, considere la segunda opción.
>
> - [Deshabilite la recopilación de eventos de seguridad](../security-center/security-center-enable-data-collection.md) en Azure Security Center y, a continuación, agregue el conector de eventos de seguridad en Azure Sentinel. Al igual que con la primera opción, podrá consultar y analizar eventos en Azure Sentinel y Azure Defender (o ASC), pero ahora podrá supervisar el estado de conectividad del conector o cambiar su configuración únicamente en Azure Sentinel.

## <a name="set-up-the-windows-security-events-connector"></a>Configuración del conector de eventos de seguridad de Windows

Para recopilar los eventos de seguridad de Windows en Azure Sentinel, haga lo siguiente:

1. En el menú de navegación de Azure Sentinel, seleccione **Data connectors** (Conectores de datos). En la lista de conectores, haga clic en **Eventos de seguridad** y, a continuación, en el botón **Open connector page** (Abrir página del conector) en la parte inferior derecha. A continuación, siga las instrucciones en pantalla debajo de la pestaña **Instrucciones**, tal como se describe en el resto de esta sección.

1. Compruebe que tiene los permisos adecuados, tal como se describe en **Requisitos previos**.

1. Descargue e instale el [agente de Log Analytics](../azure-monitor/platform/log-analytics-agent.md) (también conocido como Microsoft Monitoring Agent o MMA) en los equipos desde los que quiere transmitir eventos de seguridad a Azure Sentinel.

    Para las máquinas virtuales de Azure:
    
    1. Haga clic en **Install agent on Azure Windows Virtual Machine** (Instalar el agente en la máquina virtual de Windows de Azure) y, después, en el vínculo que aparece a continuación.
    1. Haga clic en el nombre de cada máquina virtual que quiera conectar en la lista que aparece a la derecha y, a continuación, haga clic en **Conectar**.

    Para las máquinas Windows que no son de Azure (físicas, virtuales locales o virtuales en otra nube):

    1. Haga clic en **Install agent on non-Azure Windows Machine** (Instalar el agente en una máquina de Windows que no es de Azure) y, después, en el vínculo que aparece a continuación.
    1. Haga clic en los vínculos de descarga correspondientes que aparecen a la derecha, debajo de **Equipos Windows**.
    1. Con el archivo ejecutable descargado, instale el agente en los sistemas Windows que quiera y configúrelo con el **Id. y las claves de área de trabajo** que aparecen debajo de los vínculos de descarga mencionados anteriormente.

    > [!NOTE]
    >
    > Para que los sistemas Windows sin la conectividad a Internet necesaria puedan seguir transmitiendo eventos a Azure Sentinel, use el vínculo de la esquina inferior derecha para descargar e instalar la **puerta de enlace de OMS** en una máquina independiente, que actuará como proxy.  Tendrá que instalar el agente de Log Analytics en todos los sistemas Windows cuyos eventos quiera recopilar.
    >
    > Para obtener más información sobre este escenario, consulte la [documentación de la **puerta de enlace de Log Analytics** ](../azure-monitor/platform/gateway.md).

    Para obtener más información sobre las opciones de instalación adicionales y otros detalles, consulte la [documentación del **agente de Log Analytics**](../azure-monitor/platform/agent-windows.md).

1. Seleccione el conjunto de eventos ([todos, común o mínimo](#event-sets)) que quiere transmitir.

1. Haga clic en **Update**(Actualizar).

1. Para usar el esquema correspondiente en Log Analytics para encontrar los eventos de seguridad de Windows, escriba `SecurityEvent` en la ventana de consulta.

## <a name="validate-connectivity"></a>Validar conectividad

Hasta que los registros empiecen a aparecer en Log Analytics, pueden transcurrir unos 20 minutos. 

### <a name="configure-the-security-events-connector-for-anomalous-rdp-login-detection"></a>Configuración del conector de eventos de seguridad para la detección de inicios de sesión de RDP anómalos

> [!IMPORTANT]
> La detección de inicios de sesión de RDP anómalos se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Sentinel puede aplicar aprendizaje automático (ML) a los datos de eventos de seguridad para identificar una actividad de inicio de sesión de protocolo de escritorio remoto (RDP) anómala. Los escenarios incluyen:

- **IP inusual**: la dirección IP no se ha observado nunca o casi nunca en los últimos 30 días

- **Ubicación geográfica inusual**: la dirección IP, la ciudad, el país y el ASN no se han observado nunca o casi nunca en los últimos 30 días

- **Nuevo usuario**: un nuevo usuario inicia sesión desde una dirección IP o una ubicación geográfica que no se esperaba ver de acuerdo con los datos de los 30 días anteriores.

**Instrucciones de configuración**

1. Debe recopilar los datos de inicio de sesión de RDP (identificador de evento 4624) mediante el conector de datos de **eventos de seguridad**. Asegúrese de haber seleccionado un [conjunto de eventos](#event-sets) distinto de "Ninguno" para transmitirlo a Azure Sentinel.

1. En el portal de Azure Sentinel, haga clic en **Analytics** (Análisis) y, después, haga clic en la pestaña **Rule templates** (Plantillas de reglas). Seleccione la regla **(Preview) Anomalous RDP Login Detection** [(Versión preliminar) Detección de inicio de sesión RDP anómalo] y mueva el regulador **Status** (Estado) a **Enabled** (Habilitado).

    > [!NOTE]
    > Dado que el algoritmo de aprendizaje automático requiere datos de 30 días para crear un perfil de base de referencia del comportamiento de los usuarios, debe permitir que se recopilen los datos de eventos de seguridad de 30 días antes de que se puedan detectar incidentes.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar eventos de seguridad de Windows a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a detectar amenazas con Azure Sentinel mediante las reglas [integradas](tutorial-detect-threats-built-in.md) o [personalizadas](tutorial-detect-threats-custom.md).

