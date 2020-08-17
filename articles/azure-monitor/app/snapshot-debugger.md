---
title: Depurador de instantáneas de Azure Application Insights para aplicaciones .NET
description: Depuración de las instantáneas que se recopilan automáticamente cuando se producen excepciones en aplicaciones de producción de .NET
ms.topic: conceptual
ms.date: 10/23/2019
ms.reviewer: cweining
ms.openlocfilehash: 58fbb0cee5e4f06c20d31b4b5011582957f6f6c3
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325651"
---
# <a name="debug-snapshots-on-exceptions-in-net-apps"></a>Depurar instantáneas cuando se producen excepciones en aplicaciones de .NET
Cuando se produce una excepción, puede recopilar automáticamente una instantánea de depuración desde la aplicación web activa. La instantánea muestra el estado del código fuente y las variables en el momento en que se produjo la excepción. En [Application Insights](./app-insights-overview.md), Snapshot Debugger supervisa los datos de telemetría de las excepciones de su aplicación web. Recopila instantáneas de las excepciones más importantes con el fin de que tenga la información necesaria para diagnosticar problemas en producción. Incluya el [paquete NuGet del recopilador de instantáneas](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) en la aplicación y, opcionalmente, configure los parámetros de recopilación en [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). Las instantáneas aparecen en [excepciones](./asp-net-exceptions.md) en el portal de Application Insights.

Puede ver las instantáneas de depuración en el portal para examinar la pila de llamadas e inspeccionar las variables en cada marco de pila de llamadas. Para obtener una experiencia de depuración más eficaz con el código fuente, abra las instantáneas con Visual Studio 2019 Enterprise. En Visual Studio también puede [establecer puntos de acoplamiento para tomar instantáneas de forma interactiva](https://aka.ms/snappoint) sin tener que esperar una excepción.

Las instantáneas de depuración se guardan durante 15 días. Esta directiva de retención se establece para cada aplicación. Si necesita aumentar este valor, puede solicitar un aumento abriendo una incidencia de soporte técnico en Azure Portal.

## <a name="enable-application-insights-snapshot-debugger-for-your-application"></a>Habilitación de Snapshot Debugger de Application Insights para la aplicación
La recopilación de instantáneas está disponible para:
* Aplicaciones de .NET Framework y ASP.NET que ejecuten .NET Framework 4.5 o posterior.
* Aplicaciones de .NET Core 2.0 y ASP.NET Core 2.0 que se ejecuten en Windows.

Se admiten los siguientes entornos:

* [Azure App Service](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) con la familia del SO 4 o posterior
* [Servicios de Azure Service Fabric](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) con Windows Server  2012 R2 o posterior
* [Azure Virtual Machines y conjuntos de escalado de máquinas virtuales](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) con Windows Server 2012 R2 o posterior
* [Máquinas virtuales o físicas locales](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) que ejecutan Windows Server 2012 R2 o posterior, o Windows 8.1 o posterior.

> [!NOTE]
> No se admiten las aplicaciones cliente (por ejemplo, WPF, Windows Forms o UWP).

Si ha habilitado Snapshot Debugger, pero no ve las instantáneas, consulte nuestra [Guía de solución de problemas](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

## <a name="grant-permissions"></a>Concesión de permisos

El acceso a las instantáneas está protegido por el control de acceso basado en rol (RBAC). Para poder inspeccionar una instantánea, el propietario de la suscripción debe agregarle primero al rol necesario.

> [!NOTE]
> Los propietarios y los colaboradores no tienen automáticamente este rol. Si desean ver las instantáneas, se deben agregar a sí mismos al rol.

Los propietarios de la suscripción deben asignar el rol `Application Insights Snapshot Debugger` a los usuarios que van a inspeccionar las instantáneas. Los propietarios de suscripción pueden asignar este rol a usuarios individuales o grupos en el recurso de Application Insights de destino o en su grupo de recursos o suscripción.

1. Vaya al recurso de Application Insights en Azure Portal.
1. Haga clic en **Control de acceso (IAM).**
1. Haga clic en el botón **+Add role assignment** (+ Agregar asignación de roles) botón.
1. Seleccione **Application Insights Snapshot Debugger** en la lista desplegable **Roles**.
1. Busque el usuario que quiere agregar y escriba un nombre.
1. Haga clic en el botón **Guardar** para agregar el usuario al rol.


> [!IMPORTANT]
> Las instantáneas pueden contener información personal y otra información confidencial en valores de variables y parámetros.

## <a name="view-snapshots-in-the-portal"></a>Visualización de instantáneas en el portal

Una vez que se haya producido una excepción en la aplicación y se haya creado una instantánea, debería tener instantáneas para visualizar. Pueden transcurrir entre cinco y diez minutos desde que se produce una excepción hasta que una instantánea está lista y puede visualizarse desde el portal. Para visualizar las instantáneas, en el panel **Error**, seleccione el botón **Operaciones** al ver la pestaña **Operaciones**, o bien seleccione el botón **Excepciones** al ver la pestaña **Excepciones**:

![Página de errores](./media/snapshot-debugger/failures-page.png)

Seleccione una operación o una excepción en el panel derecho para abrir el panel **Detalles de la transacción completa** y, luego, seleccione el evento de excepción. Si una instantánea está disponible para una excepción determinada, se mostrará el botón **Abrir instantánea de depuración** en el panel derecho con detalles de la [excepción](./asp-net-exceptions.md).

![Botón Abrir instantánea de depuración de la excepción](./media/snapshot-debugger/e2e-transaction-page.png)

En la vista de depuración instantánea, verá una pila de llamadas y un panel de variables. Al seleccionar marcos de la pila de llamadas en el panel de la pila de llamadas, podrá ver las variables locales y los parámetros para esa llamada de función en el panel de variables.

![Visualización de la instantánea de depuración en el portal](./media/snapshot-debugger/open-snapshot-portal.png)

Las instantáneas pueden incluir información confidencial y, de manera predeterminada, no están visibles. Para ver las instantáneas, debe tener asignado el rol `Application Insights Snapshot Debugger`.

## <a name="view-snapshots-in-visual-studio-2017-enterprise-or-above"></a>Visualización de instantáneas en Visual Studio 2017 Enterprise o versiones posteriores
1. Haga clic en el botón **Descargar la instantánea** para descargar un archivo `.diagsession`, que puede abrirse en Visual Studio Enterprise.

2. Para abrir el archivo `.diagsession`, debe tener instalado el componente Snapshot Debugger de Visual Studio. El componente Snapshot Debugger es un componente obligatorio de la carga de trabajo de ASP.NET en Visual Studio y se puede seleccionar de la lista de componentes individuales en el instalador de Visual Studio. Si usa una versión de Visual Studio anterior a Visual Studio 2017, versión 15.5, deberá instalar la extensión desde [Visual Studio Marketplace](https://aka.ms/snapshotdebugger).

3. Después de abrir el archivo de instantánea, aparece la página de depuración de minivolcado de Visual Studio. Haga clic en **Depurar código administrado** para empezar a depurar la instantánea. La instantánea se abre en la línea de código donde se produjo la excepción para que pueda depurar el estado actual del proceso.

    ![Visualización de la instantánea de depuración en Visual Studio](./media/snapshot-debugger/open-snapshot-visualstudio.png)

La instantánea descargada incluye los archivos de símbolos que se encontraron en el servidor de aplicaciones web. Estos archivos de símbolos son necesarios para asociar los datos de instantáneas con el código fuente. Para las aplicaciones de App Service, asegúrese de habilitar la implementación de símbolos cuando publique las aplicaciones web.

## <a name="how-snapshots-work"></a>Funcionamiento de las instantáneas

Snapshot Collector se implementa como un [procesador de telemetría de Application Insights](./configuration-with-applicationinsights-config.md#telemetry-processors-aspnet). Al ejecutar la aplicación, el procesador de telemetría de Snapshot Collector se agrega a la canalización de telemetría de la aplicación.
Cada vez que la aplicación llama a [TrackException](./asp-net-exceptions.md#exceptions), Snapshot Collector calcula un identificador del problema del tipo de excepción producida y el método de lanzamiento.
Cada vez que la aplicación llama a TrackException, se incrementa un contador para el identificador del problema adecuado. Cuando el contador alcanza el valor `ThresholdForSnapshotting`, el identificador del problema se agrega a un plan de recolección.

Snapshot Collector también supervisa las excepciones a medida que se producen si se suscribe al evento [AppDomain.CurrentDomain.FirstChanceException](/dotnet/api/system.appdomain.firstchanceexception). Cuando ese evento se desencadena, el identificador del problema de la excepción se calcula y se compara con los identificadores del problema del plan de recolección.
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

El período de retención de datos predeterminado es de 15 días. Para cada instancia de Application Insights, se permite un número máximo de 50 instantáneas por día.

### <a name="publish-symbols"></a>Publicación de símbolos
El Depurador de instantáneas requiere que los archivos de símbolos estén presentes en el servidor de producción para descodificar variables y proporcionar una experiencia de depuración en Visual Studio.
La versión 15.2 (o superior) de Visual Studio 2017 publica símbolos de compilaciones de versión de forma predeterminada al publicar en App Service. En las versiones anteriores, tiene que agregar la siguiente línea al archivo `.pubxml` de su perfil de publicación para que los símbolos se publiquen en modo de versión:

```xml
    <ExcludeGeneratedDebugSymbol>False</ExcludeGeneratedDebugSymbol>
```

Para Azure Compute y otros tipos, asegúrese de que los archivos de símbolos están en la misma carpeta del archivo .dll principal de la aplicación (normalmente, `wwwroot/bin`), o que están disponibles en la ruta de acceso actual.

> [!NOTE]
> Para más información acerca de las diferentes opciones de símbolos disponibles, consulte la [, documentación de Visual Studio](/visualstudio/ide/reference/advanced-build-settings-dialog-box-csharp?view=vs-2019#output
). Para obtener los mejores resultados posible, se recomienda usar "Full", "Portable" o "Embedded".

### <a name="optimized-builds"></a>Compilaciones optimizadas
En algunos casos, las variables locales no se pueden ver en las compilaciones de versión debido a las optimizaciones que aplica el compilador JIT.
Sin embargo, en Azure App Services, Snapshot Collector puede invalidar la optimización de los métodos de lanzamiento que forman parte del plan de recolección.

> [!TIP]
> Instale la extensión de sitio de Application Insights en su App Service para obtener soporte técnico para la invalidación de optimizaciones.

## <a name="next-steps"></a>Pasos siguientes
Habilitación de Snapshot Debugger de Application Insights para la aplicación:

* [Azure App Service](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Servicios de Azure Service Fabric](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Azure Virtual Machines y Virtual Machine Scale Sets](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Máquinas virtuales o físicas locales](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)

Más allá de Snapshot Debugger de Application Insights:
 
* [Establezca puntos de ajuste en el código](/visualstudio/debugger/debug-live-azure-applications) para obtener instantáneas sin tener que esperar una excepción.
* En el artículo sobre cómo [diagnosticar excepciones en aplicaciones web](./asp-net-exceptions.md) se explica cómo hacer más visibles las excepciones en Application Insights.
* [Detección inteligente](./proactive-diagnostics.md) detecta automáticamente las anomalías de rendimiento.

