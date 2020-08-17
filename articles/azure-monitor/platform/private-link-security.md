---
title: Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura
description: Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura
author: nkiest
ms.author: nikiest
ms.topic: conceptual
ms.date: 05/20/2020
ms.subservice: ''
ms.openlocfilehash: 6045fa475b3bb112afee9ceacd8d6b136087feab
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077166"
---
# <a name="use-azure-private-link-to-securely-connect-networks-to-azure-monitor"></a>Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura

> [!IMPORTANT]
> Por el momento, debe **solicitar acceso** para poder usar esta función. Puede solicitar acceso mediante el [formulario de suscripción](https://aka.ms/AzMonPrivateLinkSignup).


[Azure Private Link](../../private-link/private-link-overview.md) le permite vincular de forma segura los servicios PaaS de Azure a la red virtual mediante puntos de conexión privados. Para muchos servicios, solo tiene que configurar un punto de conexión por recurso. Sin embargo, Azure Monitor es una constelación de diferentes servicios interconectados que funcionan conjuntamente para supervisar las cargas de trabajo. Como consecuencia de ello, hemos creado un recurso denominado Ámbito de Private Link de Azure Monitor (AMPLS) que le permite definir los límites de la red de supervisión y conectarse a la red virtual. En este artículo se explica cuándo usar y cómo configurar un Ámbito de Private Link de Azure Monitor.

## <a name="advantages"></a>Ventajas

Con Private Link puede:

- Conectarse de forma privada a Azure Monitor sin necesidad de abrir ningún acceso a la red pública
- Asegurarse de que solo se accede a los datos de supervisión a través de redes privadas autorizadas
- Impedir la filtración de datos de las redes privadas mediante la definición de recursos de Azure Monitor específicos que se conectan a través del punto de conexión privado
- Conectar de forma segura la red local privada a Azure Monitor mediante ExpressRoute y Private Link
- Mantener todo el tráfico dentro de la red troncal de Microsoft Azure

Para más información, consulte [Ventajas principales de Private Link](../../private-link/private-link-overview.md#key-benefits).

## <a name="how-it-works"></a>Funcionamiento

El Ámbito de Private Link de Azure Monitor es un recurso de agrupación para conectar uno o varios puntos de conexión privados (y, por tanto, las redes virtuales en las que se encuentran) a uno o más recursos de Azure Monitor. Estos recursos incluyen las áreas de trabajo de Log Analytics y los componentes de Application Insights.

![Diagrama de la topología de los recursos](./media/private-link-security/private-link-topology-1.png)

> [!NOTE]
> Un único recurso de Azure Monitor puede pertenecer a varios AMPLS, pero no puede conectar una sola red virtual a más de un AMPLS. 

## <a name="planning-based-on-your-network"></a>Planeación basada en la red

Antes de configurar los recursos de AMPLS, tenga en cuenta los requisitos de aislamiento de la red. Evalúe el acceso de las redes virtuales a Internet pública y las restricciones de acceso de cada uno de los recursos de Azure Monitor (es decir, los componentes de Application Insights y las áreas de trabajo de Log Analytics).

### <a name="evaluate-which-virtual-networks-should-connect-to-a-private-link"></a>Evaluación de qué redes virtuales deben conectarse a Private Link

Empiece por evaluar qué redes virtuales (Vnet) tienen acceso restringido a Internet. Es posible que las redes virtuales que tienen Internet gratis no requieran Private Link para acceder a los recursos de Azure Monitor. Los recursos de supervisión a los que se conectan las redes virtuales pueden restringir el tráfico entrante y requerir una conexión de Private Link (ya sea para la consulta o la ingesta de registros). En tales casos, incluso una red virtual que tenga acceso a Internet pública necesita conectarse a estos recursos a través de Private Link y de un AMPLS.

### <a name="evaluate-which-azure-monitor-resources-should-have-a-private-link"></a>Evaluación de qué recursos de Azure Monitor deben tener Private Link

Revisión de cada uno de los recursos de Azure Monitor:

- ¿Debería el recurso permitir la ingesta de registros de recursos ubicados solo en redes virtuales específicas?
- ¿Deben consultar el recurso solo los clientes ubicados en redes virtuales específicas?

Si la respuesta a alguna de estas preguntas es afirmativa, establezca las restricciones tal y como se explica en [Configuración de áreas de trabajo de Log Analytics](#configure-log-analytics) y [Configuración de los componentes de Application Insights](#configure-application-insights) y asocie estos recursos a uno o varios AMPLS. Las redes virtuales que deben acceder a estos recursos de supervisión deben tener un punto de conexión privado que se conecte al AMPLS pertinente.
Recuerde: puede conectar las mismas áreas de trabajo o la misma aplicación a varios AMPLS, para permitir que las diferentes redes puedan llegar a ellas.

### <a name="group-together-monitoring-resources-by-network-accessibility"></a>Agrupación de los recursos de supervisión por accesibilidad de red

Dado que cada red virtual se puede conectar a un solo recurso de AMPLS, debe agrupar los recursos de supervisión que deben ser accesibles a las mismas redes. La manera más sencilla de administrar esta agrupación es crear un AMPLS por red virtual y seleccionar los recursos para conectarse a esa red. Sin embargo, para reducir los recursos y mejorar la capacidad de administración, puede que desee reutilizar un AMPLS entre redes.

Por ejemplo, si las redes virtuales internas VNet1 y VNet2 deben conectarse a las áreas de trabajo Workspace1 y Workspace2 y al componente Application Insights 3 de Application Insights, asocie los tres recursos al mismo AMPLS. Si VNet3 solo accede a Workspace1, cree otro recurso de AMPLS, asocie Workspace1 a dicho recurso y conecte VNet3 tal y como se muestra en los diagramas siguientes:

![Diagrama de la topología de AMPLS A](./media/private-link-security/ampls-topology-a-1.png)

![Diagrama de la topología de AMPLS B](./media/private-link-security/ampls-topology-b-1.png)

### <a name="consider-limits"></a>Consideración de los límites

Hay una serie de límites que se deben tener en cuenta al planear la configuración de Private Link:

* Una red virtual solo puede conectarse a 1 objeto de AMPLS. Esto significa que el objeto de AMPLS debe proporcionar acceso a todos los recursos de Azure Monitor a los que debería tener acceso la red virtual.
* Un recurso de Azure Monitor (área de trabajo o componente de Application Insights) puede conectarse a 5 AMPLS como máximo.
* Un objeto de AMPLS puede conectarse a 20 recursos como máximo de Azure Monitor.
* Un objeto de AMPLS puede conectarse a 10 puntos de conexión privados como máximo.

En la topología siguiente:
* Cada red virtual se conecta a 1 objeto de AMPLS, por lo que no se puede conectar a otros AMPLS.
* AMPLS B se conecta a 2 redes virtuales: usa 2 de sus 10 conexiones posibles de punto de conexión privado.
* AMPLS A se conecta a 2 áreas de trabajo y a 1 componente de Application Insights: usa 3 de sus 20 recursos posibles de Azure Monitor.
* El área de trabajo 2 se conecta a AMPLS A y AMPLS B: usa 2/5 de sus posibles conexiones AMPLS.

![Diagrama de límites de AMPLS](./media/private-link-security/ampls-limits.png)

## <a name="example-connection"></a>Conexión de ejemplo

Empiece por crear un recurso de Ámbito de Private Link de Azure Monitor.

1. Vaya a **Crear un recurso** en Azure Portal y busque **Azure Monitor Private Link Scope** (Ámbito de Private Link de Azure Monitor).

   ![Búsqueda de ámbito de Private Link de Azure Monitor](./media/private-link-security/ampls-find-1c.png)

2. Haga clic en **create** (crear).
3. Seleccione una suscripción y un grupo de recursos.
4. Asigne un nombre al AMPLS. Es mejor usar un nombre que deje claro qué finalidad y límite de seguridad usará el ámbito para que alguien no rompa accidentalmente los límites de seguridad de la red. Por ejemplo, "AppServerProdTelem".
5. Haga clic en **Revisar y crear**. 

   ![Creación de ámbito de Private Link de Azure Monitor](./media/private-link-security/ampls-create-1d.png)

6. Deje que se supere la validación y luego haga clic en **Crear**.

## <a name="connect-azure-monitor-resources"></a>Conexión de recursos de Azure Monitor

Puede conectar su AMPLS primero a los puntos de conexión privados y luego a los recursos de Azure Monitor, o viceversa, pero el proceso de conexión es más rápido si comienza con los recursos de Azure Monitor. A continuación se describe cómo conectamos las áreas de trabajo de Log Analytics de Azure Monitor y los componentes de Application Insights a un AMPLS.

1. En el ámbito de Private Link de Azure Monitor, haga clic en **Azure Monitor Resources**  (Recursos de Azure Monitor) en el menú de la izquierda. Haga clic en el botón **Agregar**.
2. Agregue el área de trabajo o el componente. Al hacer clic en el botón **Agregar** aparece un cuadro de diálogo en el que puede seleccionar los recursos de Azure Monitor. Puede examinar las suscripciones y los grupos de recursos, o puede escribir su nombre para filtrarlos. Seleccione el área de trabajo o componente y haga clic en **Aplicar** para agregarlos al ámbito.

    ![Captura de pantalla de selección de una experiencia de usuario de ámbito](./media/private-link-security/ampls-select-2.png)

### <a name="connect-to-a-private-endpoint"></a>Conexión a un punto de conexión privado

Ahora que tiene recursos conectados a su AMPLS, cree un punto de conexión privado para conectar nuestra red. Puede realizar esta tarea en el [centro de Private Link de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/privateendpoints) o en el Ámbito de Private Link de Azure Monitor, como se hace en este ejemplo.

1. En el recurso de ámbito, haga clic en **Conexiones de punto de conexión privado** en el menú de recursos de la izquierda. Haga clic en **Punto de conexión privado** para iniciar el proceso de creación del punto de conexión. Aquí también puede aprobar las conexiones que se iniciaron en el centro de Private Link seleccionándolas y haciendo clic en **Aprobar**.

    ![Captura de pantalla de la experiencia de usuario de conexiones de punto de conexión privado](./media/private-link-security/ampls-select-private-endpoint-connect-3.png)

2. Seleccione la suscripción, el grupo de recursos y el nombre del punto de conexión, así como la región en la que debe residir. La región debe ser la misma región que la red virtual a la que la conectará.

3. Haga clic en **Siguiente: Resource** (Siguiente: Recurso). 

4. En la pantalla Recurso,

   a. En **Suscripción**, seleccione la suscripción que contiene el recurso de Ámbito de Private Link de Azure Monitor. 

   b. Para **Tipo de recurso**, elija **Microsoft.insights/privateLinkScopes**. 

   c. En la lista desplegable **Recurso**, elija el ámbito de Private Link que creó anteriormente. 

   d. Haga clic en **Siguiente: Configuración >** .
      ![Captura de pantalla de selección de creación de punto de conexión privado](./media/private-link-security/ampls-select-private-endpoint-create-4.png)

5. En el panel de configuración,

   a.    Elija el **red virtual** y la **subred** a las que desea conectar los recursos de Azure Monitor. 
 
   b.    Elija **Sí** para **Integrar con la zona DNS privada** y permita que cree automáticamente una nueva zona DNS privada. Las zonas DNS reales pueden ser distintas de las que se muestran en la siguiente captura de pantalla. 
 
   c.    Haga clic en **Revisar + crear**.
 
   d.    Permita que la validación se supere. 
 
   e.    Haga clic en **Crear**. 

    ![Captura de pantalla de seleccione de creación de punto de conexión privado 2](./media/private-link-security/ampls-select-private-endpoint-create-5.png)

Ahora ha creado un nuevo punto de conexión privado que está conectado a este Ámbito de Private Link de Azure Monitor.

## <a name="configure-log-analytics"></a>Configuración de Log Analytics

Vaya a Azure Portal. En el recurso del área de trabajo de Log Analytics, hay un elemento de menú **Network Isolation** (Aislamiento de red) en el lado izquierdo. En este menú puede controlar dos estados diferentes. 

![Aislamiento de red de Log Analytics](./media/private-link-security/ampls-log-analytics-lan-network-isolation-6.png)

En primer lugar, puede conectar este recurso de Log Analytics a cualquier ámbito de Private Link de Azure Monitor al que tenga acceso. Haga clic en **Agregar** y seleccione el Ámbito de Private Link de Azure Monitor.  Haga clic en **Aplicar** para conectarlo. Todos los ámbitos conectados aparecen en esta pantalla. Gracias a esta conexión, el tráfico de red de las redes virtuales conectadas llega a esta área de trabajo. La creación de la conexión tiene el mismo efecto realizar la conexión desde el ámbito que creamos en [Conexión de recursos de Azure Monitor](#connect-azure-monitor-resources).  

En segundo lugar, puede controlar cómo se puede acceder a este recurso desde fuera de los ámbitos de Private Link enumerados anteriormente. Si establece **Allow public network access for ingestion** (Permitir el acceso de la red pública para la ingesta) en **No**, las máquinas que se encuentren fuera de los ámbitos conectados no podrán cargar datos en esta área de trabajo. Si establece **Allow public network access for queries** (Permitir el acceso a la red pública para las consultas) en **No**, las máquinas que se encuentren fuera de los ámbitos no podrán acceder a los datos de esta área de trabajo. Estos datos incluyen el acceso a libros, paneles, experiencias de cliente basadas en la API de consulta, conclusiones en el Azure Portal, etc. Las experiencias que se ejecutan fuera de Azure Portal y que consultan los datos de Log Analytics también deben ejecutarse dentro de la red virtual vinculada privada.

Restringir el acceso de esta manera solo se aplica a los datos del área de trabajo. Los cambios de configuración, incluida la activación o desactivación de esta configuración de acceso, se administran mediante Azure Resource Manager. Restrinja el acceso a Resource Manager mediante las funciones, los permisos, los controles de red y la auditoría adecuados. Para más información, consulte [Roles, permisos y seguridad en Azure Monitor](roles-permissions-security.md).

> [!NOTE]
> Los registros y las métricas que se cargan en un área de trabajo a través de [Configuración de diagnóstico](diagnostic-settings.md) pasan por un canal de Microsoft privado seguro y no se controlan mediante esta configuración.

## <a name="configure-application-insights"></a>Configuración de Application Insights

Vaya a Azure Portal. En el recurso del componente de Application Insights de Azure Monitor se encuentra el elemento de menú **Aislamiento de red** en el lado izquierdo. En este menú puede controlar dos estados diferentes.

![Aislamiento de red de Application Insights](./media/private-link-security/ampls-application-insights-lan-network-isolation-6.png)

En primer lugar, puede conectar este recurso de Application Insights a ámbitos de Private Link de Azure Monitor a los que tenga acceso. Haga clic en **Agregar** y seleccione el **Ámbito de Private Link de Azure Monitor**. Haga clic en Aplicar para conectarlo. Todos los ámbitos conectados aparecen en esta pantalla. Gracias a esta conexión, el tráfico de red de las redes virtuales conectadas llega a este componente. La creación de la conexión tiene el mismo efecto realizar la conexión desde el ámbito que creamos en [Conexión de recursos de Azure Monitor](#connect-azure-monitor-resources). 

En segundo lugar, puede controlar cómo se puede acceder a este recurso desde fuera de los ámbitos de Private Link enumerados anteriormente. Si establece **Allow public network access for ingestion** (Permitir el acceso de la red pública para la ingesta) en **No**, las máquinas o los SDK que se encuentren fuera de los ámbitos conectados no podrán cargar datos en este componente. Si establece **Allow public network access for queries** (Permitir el acceso a la red pública para las consultas) en **No**, las máquinas que se encuentren fuera de los ámbitos no podrán acceder a los datos de este recurso de Application Insights. Estos datos incluyen el acceso a los registros de APM, las métricas y la transmisión de métricas en directo, así como experiencias basadas en ellos, como libros, paneles, experiencias de cliente basadas en API de consulta, conclusiones de Azure Portal, etc. 

Tenga en cuenta que las experiencias de consumo que no son del portal también tienen que ejecutarse dentro de la red virtual vinculada privada que incluye las cargas de trabajo supervisadas. 

Tendrá que agregar recursos que hospeden las cargas de trabajo supervisadas al vínculo privado. Aquí está la [documentación](../../app-service/networking/private-endpoint.md) sobre cómo hacer esto para App Services.

Restringir el acceso de esta manera solo se aplica a los datos del recurso de Application Insights. Los cambios de configuración, incluida la activación o desactivación de esta configuración de acceso, se administran mediante Azure Resource Manager. En su lugar, restrinja el acceso a Resource Manager mediante las funciones, los permisos, los controles de red y la auditoría adecuados. Para más información, consulte [Roles, permisos y seguridad en Azure Monitor](roles-permissions-security.md).

> [!NOTE]
> Para proteger completamente la instancia de Application Insights basada en el área de trabajo, debe bloquear tanto el acceso al recurso de Application Insights como el área de trabajo de Log Analytics subyacente.
>
> Los diagnósticos de nivel de código (Perfilador/Depurador) deben proporcionar su propia cuenta de almacenamiento para admitir el vínculo privado. Esta es [documentación](../app/profiler-bring-your-own-storage.md) sobre cómo hacerlo.

## <a name="use-apis-and-command-line"></a>Uso de API y de la línea de comandos

Puede automatizar el proceso descrito anteriormente mediante plantillas de Azure Resource Manager y las interfaces de la línea de comandos.

Para crear y administrar ámbitos de Private Link, use [az monitor private-link-scope](/cli/azure/monitor/private-link-scope?view=azure-cli-latest). Con este comando, puede crear ámbitos, asociar áreas de trabajo de Log Analytics y componentes de Application Insights, y agregar, quitar o aprobar puntos de conexión privados.

Para administrar el acceso a la red, utilice las marcas `[--ingestion-access {Disabled, Enabled}]` y `[--query-access {Disabled, Enabled}]`en [áreas de trabajo de Log Analytics](/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest) o [componentes de Application Insights](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest).

## <a name="collect-custom-logs-over-private-link"></a>Recopilación de registros personalizados a través de Private Link

Las cuentas de almacenamiento se usan en el proceso de ingesta de registros personalizados. De forma predeterminada, se usan cuentas de almacenamiento administradas por el servicio. Sin embargo, para ingerir registros personalizados en vínculos privados, debe usar sus propias cuentas de almacenamiento y asociarlas a áreas de trabajo de Log Analytics. Vea más detalles sobre cómo configurar tales cuentas mediante la [línea de comandos](/cli/azure/monitor/log-analytics/workspace/linked-storage?view=azure-cli-latest).

Para obtener más información sobre cómo traer su propia cuenta de almacenamiento, consulte [Cuentas de almacenamiento propiedad del cliente para la ingesta de registros](private-storage.md)

## <a name="restrictions-and-limitations"></a>Restricciones y limitaciones

### <a name="agents"></a>Agentes

Las versiones más recientes de los agentes de Windows y Linux deben usarse en redes privadas para permitir la ingesta de telemetría segura en áreas de trabajo de Log Analytics. Las versiones anteriores no pueden cargar datos de supervisión en una red privada.

**Agente de Windows de Log Analytics**

Use la versión 10.20.18038.0, o una posterior, del agente de Log Analytics.

**Agente de Linux de Log Analytics**

Use la versión 1.12.25, o una posterior, del agente. Si no puede, ejecute los siguientes comandos en la máquina virtual.

```cmd
$ sudo /opt/microsoft/omsagent/bin/omsadmin.sh -X
$ sudo /opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <workspace key>
```

### <a name="azure-portal"></a>Azure portal

Para usar las experiencias del portal de Azure Monitor, como Application Insights y Log Analytics, debe permitir que se pueda acceder a las extensiones de Azure Portal y Azure Monitor en las redes privadas. Agregue las [etiquetas de servicio](../../firewall/service-tags.md) **AzureActiveDirectory**, **AzureResourceManager**, **AzureFrontDoor.FirstParty** y **AzureFrontdoor.Frontend** al firewall.

### <a name="programmatic-access"></a>Acceso mediante programación

Para usar la API REST, la [CLI](/cli/azure/monitor?view=azure-cli-latest) o PowerShell con Azure Monitor en redes privadas, agregue las [etiquetas de servicio](../../virtual-network/service-tags-overview.md) **AzureActiveDirectory** y **AzureResourceManager** al firewall.

La incorporación de estas etiquetas permite realizar acciones como consultar datos de registro así como crear y administrar áreas de trabajo de Log Analytics y componentes de Application Insights.

### <a name="application-insights-sdk-downloads-from-a-content-delivery-network"></a>Descargas de SDK de Application Insights desde una red de entrega de contenido

Agrupe el código JavaScript en el script para que el explorador no intente descargar código desde una red CDN. En [GitHub](https://github.com/microsoft/ApplicationInsights-JS#npm-setup-ignore-if-using-snippet-setup) se proporciona un ejemplo.

### <a name="log-analytics-solution-download"></a>Descarga de soluciones de Log Analytics

Para permitir que el agente de Log Analytics descargue paquetes de soluciones, agregue los nombres de dominio completos correspondientes a la lista de permitidos del firewall. 


| Entorno en la nube | Recurso del agente | Puertos | Dirección |
|:--|:--|:--|:--|
|Azure Public     | scadvisorcontent.blob.core.windows.net         | 443 | Salida
|Azure Government | usbn1oicore.blob.core.usgovcloudapi.net | 443 |  Salida
|Azure China 21Vianet      | mceast2oicore.blob.core.chinacloudapi.cn| 443 | Salida

### <a name="browser-dns-settings"></a>Configuración de DNS del explorador

Si se va a conectar a los recursos de Azure Monitor a través de un vínculo privado, el tráfico a estos recursos debe pasar por el punto de conexión privado que está configurado en la red. Para habilitar el extremo privado, actualice la configuración de DNS tal como se explica en [Conectarse a un punto de conexión privado](#connect-to-a-private-endpoint). Algunos exploradores usan su propia configuración de DNS en lugar de las que se establecen. Es posible que el explorador intente conectarse a los puntos de conexión públicos de Azure Monitor y omitir por completo el vínculo privado. Compruebe que la configuración de su explorador no invalide ni guarde en caché la configuración de DNS anterior. 

## <a name="next-steps"></a>Pasos siguientes

- Obtener información sobre el [almacenamiento privado](private-storage.md).
