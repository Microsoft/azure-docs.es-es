---
title: Depurador de instantáneas de Azure Application Insights para aplicaciones de .NET | Microsoft Docs
description: Depuración de las instantáneas que se recopilan automáticamente cuando se producen excepciones en aplicaciones de producción de .NET
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.reviewer: brahmnes
ms.date: 03/07/2019
ms.author: mbullwin
ms.openlocfilehash: 4157285e8af67acd1dc3627bebc12076d7fe072c
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595571"
---
# <a name="debug-snapshots-on-exceptions-in-net-apps"></a>Depurar instantáneas cuando se producen excepciones en aplicaciones de .NET
Cuando se produce una excepción, puede recopilar automáticamente una instantánea de depuración desde la aplicación web activa. La instantánea muestra el estado del código fuente y las variables en el momento en que se produjo la excepción. El depurador de instantáneas (versión preliminar) de [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) supervisa la telemetría de excepciones de su aplicación web. Recopila instantáneas de las excepciones más importantes con el fin de que tenga la información necesaria para diagnosticar problemas en producción. Incluya el [paquete NuGet del recopilador de instantáneas](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) en la aplicación y, opcionalmente, configure los parámetros de recopilación en [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md). Las instantáneas aparecen en [excepciones](../../azure-monitor/app/asp-net-exceptions.md) en el portal de Application Insights.

Puede ver las instantáneas de depuración en el portal para examinar la pila de llamadas e inspeccionar las variables en cada marco de pila de llamadas. Para obtener una experiencia de depuración más eficaz con el código fuente, abra instantáneas con Visual Studio Enterprise de 2019. En Visual Studio también puede [establecer puntos de acoplamiento para tomar instantáneas de forma interactiva](https://aka.ms/snappoint) sin tener que esperar una excepción.

Las instantáneas de depuración se guardan durante siete días. Esta directiva de retención se establece para cada aplicación. Si necesita aumentar este valor, puede solicitar un aumento abriendo una incidencia de soporte técnico en Azure Portal.

## <a name="enable-application-insights-snapshot-debugger-for-your-application"></a>Habilitar a Application Insights Snapshot Debugger para la aplicación
La recopilación de instantáneas está disponible para:
* Aplicaciones de .NET Framework y ASP.NET que ejecuten .NET Framework 4.5 o posterior.
* Aplicaciones de .NET Core 2.0 y ASP.NET Core 2.0 que se ejecuten en Windows.

Se admiten los siguientes entornos:

* [Azure App Service](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) ejecutar la familia del SO 4 o posterior
* [Servicios de Azure Service Fabric](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) con Windows Server 2012 R2 o posterior
* [Conjuntos de escalado de máquinas virtuales y Azure Virtual Machines](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) ejecuta Windows Server 2012 R2 o posterior
* [Máquinas virtuales o físicas locales](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) ejecuta Windows Server 2012 R2 o posterior

> [!NOTE]
> No se admiten las aplicaciones cliente (por ejemplo, WPF, Windows Forms o UWP).

Si ha habilitado el depurador de instantáneas pero no ve las instantáneas, consulte nuestra [Guía de solución de problemas](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

## <a name="grant-permissions"></a>Conceder permisos

El acceso a las instantáneas está protegido por el control de acceso basado en rol (RBAC). Para poder inspeccionar una instantánea, el propietario de la suscripción debe agregarle primero al rol necesario.

> [!NOTE]
> Los propietarios y los colaboradores no tienen automáticamente este rol. Si desean ver las instantáneas, se deben agregar a sí mismos al rol.

Los propietarios de la suscripción deben asignar el rol `Application Insights Snapshot Debugger` a los usuarios que van a inspeccionar las instantáneas. Los propietarios de suscripción pueden asignar este rol a usuarios individuales o grupos en el recurso de Application Insights de destino o en su grupo de recursos o suscripción.

1. Vaya al recurso de Application Insights en Azure Portal.
1. Haga clic en **Control de acceso (IAM)**.
1. Haga clic en el botón **+Add role assignment** (+ Agregar asignación de roles) botón.
1. Seleccione **Application Insights Snapshot Debugger** en la lista desplegable **Roles**.
1. Busque el usuario que quiere agregar y escriba un nombre.
1. Haga clic en el botón **Guardar** para agregar el usuario al rol.


> [!IMPORTANT]
> Las instantáneas pueden contener información personal y otra información confidencial en valores de variables y parámetros.

## <a name="view-snapshots-in-the-portal"></a>Ver las instantáneas en el Portal

Una vez que se ha producido una excepción en la aplicación y se ha creado una instantánea, debe tener instantáneas que desea ver. Puede tardar 5 a 10 minutos de una excepción que se producen en una instantánea lista y hacerlo visible desde el portal. Para ver las instantáneas, en el **error** panel, seleccione el **operaciones** botón al ver el **operaciones** pestaña o seleccione el **excepciones**botón al ver el **excepciones** pestaña:

![Página de errores](./media/snapshot-debugger/failures-page.png)

Seleccione una operación o una excepción en el panel derecho para abrir el **detalles to-End de la transacción** panel y, a continuación, seleccione el evento de excepción. Si una instantánea está disponible para la excepción dada, un **Abrir instantánea de depuración** botón aparece en el panel derecho con los detalles de la [excepción](../../azure-monitor/app/asp-net-exceptions.md).

![Botón Abrir instantánea de depuración de la excepción](./media/snapshot-debugger/e2e-transaction-page.png)

En la vista de depuración instantánea, verá una pila de llamadas y un panel de variables. Al seleccionar marcos de la pila de llamadas en el panel de la pila de llamadas, podrá ver las variables locales y los parámetros para esa llamada de función en el panel de variables.

![Visualización de la instantánea de depuración en el portal](./media/snapshot-debugger/open-snapshot-portal.png)

Las instantáneas pueden incluir información confidencial y, de manera predeterminada, no están visibles. Para ver las instantáneas, debe tener asignado el rol `Application Insights Snapshot Debugger`.

## <a name="view-snapshots-in-visual-studio-2017-enterprise-or-above"></a>Ver las instantáneas en Visual Studio Enterprise 2017 o versiones posteriores
1. Haga clic en el **descargar instantánea** botón para descargar un `.diagsession` archivo, que puede abrirse en Visual Studio Enterprise.

2. Para abrir el `.diagsession` archivo, debe tener instalado el componente de Visual Studio de depurador de instantáneas. El componente del depurador de instantáneas es un componente necesario de la carga de trabajo de ASP.net en Visual Studio y se puede seleccionar de la lista de componentes individuales en el instalador de Visual Studio. Si usa una versión de Visual Studio anteriores a Visual Studio 2017 versión 15.5, deberá instalar la extensión de la [Visual Studio Marketplace](https://aka.ms/snapshotdebugger).

3. Después de abrir el archivo de instantánea, aparece la página de depuración de minivolcado de Visual Studio. Haga clic en **Depurar código administrado** para empezar a depurar la instantánea. La instantánea se abre en la línea de código donde se produjo la excepción para que pueda depurar el estado actual del proceso.

    ![Visualización de la instantánea de depuración en Visual Studio](./media/snapshot-debugger/open-snapshot-visualstudio.png)

La instantánea descargada incluye los archivos de símbolos que se encontraron en el servidor de aplicaciones web. Estos archivos de símbolos son necesarios para asociar los datos de instantáneas con el código fuente. Para las aplicaciones de App Service, asegúrese de habilitar la implementación de símbolos cuando publique las aplicaciones web.

## <a name="how-snapshots-work"></a>Funcionamiento de las instantáneas

Snapshot Collector se implementa como un [procesador de telemetría de Application Insights](../../azure-monitor/app/configuration-with-applicationinsights-config.md#telemetry-processors-aspnet). Al ejecutar la aplicación, el procesador de telemetría de Snapshot Collector se agrega a la canalización de telemetría de la aplicación.
Cada vez que la aplicación llama a [TrackException](../../azure-monitor/app/asp-net-exceptions.md#exceptions), Snapshot Collector calcula un identificador del problema del tipo de excepción producida y el método de lanzamiento.
Cada vez que la aplicación llama a TrackException, se incrementa un contador para el identificador del problema adecuado. Cuando el contador alcanza el valor `ThresholdForSnapshotting`, el identificador del problema se agrega a un plan de recolección.

Snapshot Collector también supervisa las excepciones a medida que se producen si se suscribe al evento [AppDomain.CurrentDomain.FirstChanceException](https://docs.microsoft.com/dotnet/api/system.appdomain.firstchanceexception). Cuando ese evento se desencadena, el identificador del problema de la excepción se calcula y se compara con los identificadores del problema del plan de recolección.
Si se encuentra una coincidencia, se crea una instantánea del proceso en ejecución. Se asigna un identificador único a la instantánea y la excepción se marca con ese identificador. Tras la devolución del controlador de FirstChanceException, la excepción producida se procesa con normalidad. Finalmente, la excepción alcanza el método TrackException de nuevo que, junto con el identificador de instantáneas, se notifica a Application Insights.

El proceso principal sigue ejecutándose y ofrece tráfico a los usuarios con poca interrupción. Mientras tanto, la instantánea se entrega al proceso del cargador de instantáneas. El cargador de instantáneas crea un minivolcado y lo carga en Application Insights junto con los archivos de símbolos (.pdb) pertinentes.

> [!TIP]
> - Una instantánea de proceso es un clon suspendido del proceso en ejecución.
> - La creación de la instantánea tarda aproximadamente de 10 a 20 milisegundos.
> - El valor predeterminado de `ThresholdForSnapshotting` es 1. También es el valor mínimo. Por lo tanto, la aplicación debe desencadenar la misma excepción **dos veces** antes de crear una instantánea.
> - Establezca `IsEnabledInDeveloperMode` en true si quiere generar instantáneas durante la depuración en Visual Studio.
> - La velocidad de creación de instantáneas está limitada por la configuración de `SnapshotsPerTenMinutesLimit`. De manera predeterminada, el límite es una instantánea cada diez minutos.
> - No se pueden cargar más de 50 instantáneas al día.

## <a name="limitations"></a>Limitaciones

El período de retención de datos predeterminado es 15 días. Para cada instancia de Application Insights, se permite un número máximo de 50 instantáneas por día.

### <a name="publish-symbols"></a>Publicación de símbolos
El Depurador de instantáneas requiere que los archivos de símbolos estén presentes en el servidor de producción para descodificar variables y proporcionar una experiencia de depuración en Visual Studio.
La versión 15.2 (o superior) de Visual Studio 2017 publica símbolos de compilaciones de versión de forma predeterminada al publicar en App Service. En las versiones anteriores, tiene que agregar la siguiente línea al archivo `.pubxml` de su perfil de publicación para que los símbolos se publiquen en modo de versión:

```xml
    <ExcludeGeneratedDebugSymbol>False</ExcludeGeneratedDebugSymbol>
```

Para Azure Compute y otros tipos, asegúrese de que los archivos de símbolos están en la misma carpeta del archivo .dll principal de la aplicación (normalmente, `wwwroot/bin`), o que están disponibles en la ruta de acceso actual.

### <a name="optimized-builds"></a>Compilaciones optimizadas
En algunos casos, las variables locales no se pueden ver en las compilaciones de versión debido a las optimizaciones que aplica el compilador JIT.
Sin embargo, en Azure App Services, Snapshot Collector puede invalidar la optimización de los métodos de lanzamiento que forman parte del plan de recolección.

> [!TIP]
> Instale la extensión de sitio de Application Insights en su App Service para obtener soporte técnico para la invalidación de optimizaciones.

## <a name="next-steps"></a>Pasos siguientes
Habilitar a Application Insights Snapshot Debugger para la aplicación:

* [Azure App Service](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Servicios de Azure Service Fabric](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Azure Virtual Machines y Virtual Machine Scale Sets](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Máquinas virtuales o físicas locales](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)

Más allá de Application Insights Snapshot Debugger:
 
* [Establezca puntos de ajuste en el código](https://docs.microsoft.com/visualstudio/debugger/debug-live-azure-applications) para obtener instantáneas sin tener que esperar una excepción.
* En el artículo sobre cómo [diagnosticar excepciones en aplicaciones web](../../azure-monitor/app/asp-net-exceptions.md) se explica cómo hacer más visibles las excepciones en Application Insights.
* [Detección inteligente](../../azure-monitor/app/proactive-diagnostics.md) detecta automáticamente las anomalías de rendimiento.
