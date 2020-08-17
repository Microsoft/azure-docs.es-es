---
title: Implementación de la herramienta de diagnósticos para Windows Virtual Desktop (clásico) - Azure
description: Procedimiento para implementar la herramienta de diagnósticos de experiencia de usuario para Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c456f7e82bb605909f31b35cdd838145900396e2
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009195"
---
# <a name="deploy-the-windows-virtual-desktop-classic-diagnostics-tool"></a>Implementación de la herramienta de diagnósticos para Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager.

>[!IMPORTANT]
>A partir del 16 de marzo de 2020, se deshabilitaron temporalmente las consultas de diagnóstico que afectan a la experiencia del usuario debido al aumento de demanda en el servicio. Por este motivo, la herramienta dejará de funcionar, ya que depende de dichas consultas. Este artículo se actualizará cuando las consultas de diagnóstico vuelvan a estar disponibles.
>
>Hasta entonces, se recomienda [usar Log Analytics](diagnostics-log-analytics-2019.md) para la supervisión continuada.

Esto es lo que la herramienta de diagnósticos para Windows Virtual Desktop puede hacer:

- Buscar actividades de diagnóstico (administración, conexión o fuente) para un solo usuario durante un período de una semana.
- Recopilar información del host de sesión para las actividades de conexión desde el área de trabajo de Log Analytics.
- Revisar los detalles de rendimiento de la máquina virtual para un host determinado.
- Ver los usuarios que han iniciado sesión en el host de sesión.
- Enviar un mensaje a los usuarios activos en un host de sesión específico.
- Cerrar la sesión de los usuarios de un host de sesión.

## <a name="prerequisites"></a>Requisitos previos

Debe crear un registro de aplicaciones de Azure Active Directory y un área de trabajo de Log Analytics antes de implementar la plantilla de Azure Resource Manager para la herramienta. El usuario o el administrador necesitan estos permisos para hacerlo:

- Propietario de la suscripción de Azure
- Permiso para crear recursos en la suscripción de Azure
- Permiso para crear una aplicación de Azure AD
- Derechos de propietario o colaborador de RDS

También debe instalar estos dos módulos de PowerShell antes de empezar:

- [Módulo de Azure PowerShell](/powershell/azure/install-az-ps?view=azps-2.4.0/)
- [Módulo de Azure AD](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0/)

Asegúrese de que tiene el identificador de suscripción preparado para cuando inicie sesión.

Cuando ya lo tenga todo en orden, puede crear el registro de la aplicación de Azure AD.

## <a name="create-an-azure-active-directory-app-registration"></a>Creación de un registro de aplicaciones de Azure Active Directory

En esta sección se muestra cómo usar PowerShell para crear la aplicación de Azure Active Directory con una entidad de servicio y obtener permisos de API para ella.

>[!NOTE]
>Los permisos de API son de Windows Virtual Desktop, los permisos de Log Analytics y Microsoft Graph API se agregan a la aplicación de Azure Active Directory.

1. Abra PowerShell como administrador.
2. Inicie sesión en Azure con una cuenta que tenga permisos de propietario o de colaborador en la suscripción de Azure que desea utilizar para la herramienta de diagnóstico:
   ```powershell
   Login-AzAccount
   ```
3. Inicie sesión en Azure AD con la misma cuenta:
   ```powershell
   Connect-AzureAD
   ```
4. Vaya al [repositorio de GitHub de plantillas de RDS](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/diagnostics-sample/deploy/scripts) y ejecute el script **CreateADAppRegistrationforDiagnostics.ps1** en PowerShell.
5.  Cuando el script le pida que asigne un nombre a la aplicación, escriba un nombre de aplicación único.


Cuando el script se ejecuta correctamente, debería mostrar lo siguiente en la salida:

-  Un mensaje que confirma que la aplicación ahora tiene asignado un rol de entidad de servicio.
-  El identificador de cliente y la clave secreta de cliente que necesitará al implementar la herramienta de diagnóstico.

Ahora que ha registrado la aplicación, es el momento de configurar el área de trabajo de Log Analytics.

## <a name="configure-your-log-analytics-workspace"></a>Configuración del área de trabajo de Log Analytics

Para obtener la mejor experiencia posible, se recomienda configurar el área de trabajo de Log Analytics con los siguientes contadores de rendimiento que le permiten derivar instrucciones de la experiencia del usuario en una sesión remota. Para obtener una lista de los contadores recomendados con umbrales sugeridos, consulte [Umbrales de los contadores de rendimiento de Windows](deploy-diagnostics.md#windows-performance-counter-thresholds).

### <a name="create-an-azure-log-analytics-workspace-using-powershell"></a>Creación de un área de trabajo de Azure Log Analytics mediante PowerShell

Puede ejecutar un script de PowerShell para crear un área de trabajo de Log Analytics y configurar los contadores de rendimiento de Windows recomendados para supervisar la experiencia del usuario y el rendimiento de la aplicación.

>[!NOTE]
>Si ya tiene un área de trabajo de Log Analytics existente creada sin el script de PowerShell que quiere usar, vaya directamente a [validar los resultados del script en Azure Portal](#validate-the-script-results-in-the-azure-portal).

Para ejecutar el script de PowerShell:

1.  Abra PowerShell como administrador.
2.  Vaya al [repositorio de GitHub de plantillas de RDS](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/diagnostics-sample/deploy/scripts) y ejecute el script **CreateLogAnalyticsWorkspaceforDiagnostics.ps1** en PowerShell.
3. Escriba los siguientes valores para los parámetros:

    - En **ResourceGroupName**, escriba el nombre del grupo de recursos.
    - En **LogAnalyticsWorkspaceName**, escriba un nombre único para el área de trabajo de Log Analytics.
    - En **Ubicación**, especifique la región de Azure que está usando.
    - Escriba el **Id. de suscripción de Azure**, que puede encontrar en Azure Portal, en **Suscripciones**.

4. Escriba las credenciales de un usuario con acceso de administrador delegado.
5. Inicie sesión en Azure Portal con las credenciales del mismo usuario.
6. Anote o memorice el identificador de LogAnalyticsWorkspace para más adelante.
7. Si configura el área de trabajo de Log Analytics con el script de PowerShell, los contadores de rendimiento ya deben estar configurados y puede ir directamente a [Validación de los resultados del script en Azure Portal](#validate-the-script-results-in-the-azure-portal). De lo contrario, continúe con la sección siguiente.

### <a name="configure-windows-performance-counters-in-your-existing-log-analytics-workspace"></a>Configuración de contadores de rendimiento de Windows en el área de trabajo de Log Analytics existente

Esta sección es para aquellos usuarios que deseen usar un área de trabajo de Azure Log Analytics existente creada sin el script de PowerShell en la sección anterior. Si no ha usado el script, debe configurar manualmente los contadores de rendimiento de Windows recomendados.

Aquí se muestra cómo configurar manualmente los contadores de rendimiento recomendados:

1. Abra el explorador de Internet e inicie sesión en [Azure Portal](https://portal.azure.com/) con la cuenta administrativa.
2. A continuación, vaya a las **áreas de trabajo de Log Analytics** para revisar los contadores de rendimiento de Windows configurados.
3. En la sección **Configuración**, seleccione **Configuración avanzada**.
4. Después, vaya a **Datos** > **Contadores de rendimiento de Windows** y agregue los siguientes contadores:

    -   Disco lógico(\*)\\% espacio disponible
    -   Disco lógico (C:)\\Longitud Longitud de la cola de disco
    -   Memoria(\*)\\Mbytes disponibles
    -   Información del procesador(\*)\\Tiempo de procesador
    -   Retraso de entrada de usuario por sesión(\*)\\Retraso máximo de entrada

Obtenga más información sobre los contadores de rendimiento en [Orígenes de datos de rendimiento de Windows y Linux en Azure Monitor](/azure/azure-monitor/platform/data-sources-performance-counters).

>[!NOTE]
>Los contadores adicionales que configure no aparecerán en la herramienta de diagnóstico. Para que aparezca en la herramienta de diagnóstico, debe preparar el archivo de configuración de la herramienta. Las instrucciones sobre cómo hacerlo con administración avanzada estarán disponibles en GitHub en un futuro.

## <a name="validate-the-script-results-in-the-azure-portal"></a>Validación de los resultados del script en Azure Portal

Antes de continuar con la implementación de la herramienta de diagnóstico, se recomienda comprobar que la aplicación de Azure Active Directory tiene permisos de API y que el área de trabajo de Log Analytics tiene los contadores de rendimiento de Windows preconfigurados.

### <a name="review-your-app-registration"></a>Revisión del registro de aplicaciones

Para asegurarse de que el registro de la aplicación tiene permisos de API:

1. Abra un explorador y conéctese a [Azure Portal](https://portal.azure.com/) con su cuenta de administrador.
2. Vaya a **Azure Active Directory**.
3. Vaya a **Registros de aplicaciones** y seleccione **Todas las aplicaciones**.
4. Busque el registro de la aplicación Azure AD con el mismo nombre de aplicación que especificó en el paso 5 de [Creación de un registro de aplicaciones de Azure Active Directory](deploy-diagnostics.md#create-an-azure-active-directory-app-registration).

### <a name="review-your-log-analytics-workspace"></a>Revisión del área de trabajo de Log Analytics

Para asegurarse de que el área de trabajo de Log Analytics tiene los contadores de rendimiento de Windows preconfigurados:

1. En [Azure Portal](https://portal.azure.com/), vaya a **Áreas de trabajo de Log Analytics** para revisar los contadores de rendimiento de Windows configurados.
2. En **Configuración**, seleccione **Configuración avanzada**.
3. Después, vaya a **Datos** > **Contadores de rendimiento de Windows**.
4. Asegúrese de que los siguientes contadores están preconfigurados:

   - Disco lógico(\*)\\% espacio disponible: muestra el porcentaje de espacio libre que se puede usar del espacio total del disco.
   - Disco lógico (C:)\\Longitud promedio de la cola de disco: duración de la solicitud de transferencia de disco para la unidad C. El valor no debe ser superior a 2 durante más de un breve período.
   - Memoria(\*)\\Mbytes disponibles: memoria disponible para el sistema en megabytes.
   - Información de procesador(\*)\\Tiempo de procesador: porcentaje de tiempo transcurrido que el procesador invierte en ejecutar un subproceso no inactivo.
   - Retraso de entrada de usuario por sesión(\*)\\Retraso máximo de entrada

### <a name="connect-to-vms-in-your-log-analytics-workspace"></a>Conexión a las máquinas virtuales del área de trabajo de Log Analytics

Para poder ver el estado de las máquinas virtuales, debe habilitar la conexión de Log Analytics. Siga estos pasos para conectar las máquinas virtuales:

1. Abra un explorador e inicie sesión en [Azure Portal](https://portal.azure.com/) con su cuenta de administrador.
2. Vaya al área de trabajo de Log Analytics.
3. En el panel izquierdo, en Orígenes de datos del área de trabajo, seleccione **Máquinas virtuales**.
4. Seleccione el nombre de la máquina virtual a la que desea conectarse.
5. Seleccione **Conectar**.

## <a name="deploy-the-diagnostics-tool"></a>Implementación de la herramienta de diagnósticos

Para implementar la plantilla de Azure Resource Manager, siga las instrucciones:

1.  Vaya a la [página RDS-Templates de Azure de GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/diagnostics-sample/deploy).
2.  Implemente la plantilla en Azure y siga las instrucciones de la plantilla. Asegúrese de que tiene la siguiente información disponible:

    -   Client-Id
    -   Client-Secret
    -   Identificador del área de trabajo de Log Analytics

3.  Una vez que se proporcionan los parámetros de entrada, acepte los términos y condiciones y, después, seleccione **Comprar**.

La implementación tardará de dos a tres minutos. Cuando la implementación se ha realizado correctamente, vaya al grupo de recursos y asegúrese de que los recursos de la aplicación web y del plan de App Service están ahí.

Después, debe establecer el URI de redirección.

### <a name="set-the-redirect-uri"></a>Establecimiento del URI de redirección

Para establecer el URI de redirección:

1.  En [Azure Portal](https://portal.azure.com/), vaya a **App Services** y busque la aplicación que ha creado.
2.  Vaya a la página de información general y copie la dirección URL que encuentre allí.
3.  Vaya a **Registros de aplicaciones** y seleccione la aplicación que desea implementar.
4.  En el panel izquierdo, en la sección Administrar, seleccione **Autenticación**.
5.  Escriba el URI de redireccionamiento deseado en el cuadro de texto **URI de redirección** y, a continuación, seleccione **Guardar** en la esquina superior izquierda del menú.
6. Seleccione **Web** en el menú desplegable debajo de Tipo.
7. Escriba la dirección URL de la página de información general de la aplicación y agregue **/security/signin-callback** al final. Por ejemplo: `https://<yourappname>.azurewebsites.net/security/signin-callback`.

   > [!div class="mx-imgBorder"]
   > ![Página de redirección de URI](../media/redirect-uri-page.png)

8. Ahora, vaya a los recursos de Azure, seleccione el recurso de Azure App Services con el nombre proporcionado en la plantilla y vaya a la dirección URL asociada a él. (Por ejemplo, si el nombre de la aplicación que usó en la plantilla era `contosoapp45`, la dirección URL asociada es <https://contosoapp45.azurewebsites.net>).
9. Inicie sesión con la cuenta de usuario de Azure Active Directory adecuada.
10.   Seleccione **Aceptar**.

## <a name="distribute-the-diagnostics-tool"></a>Distribución de la herramienta de diagnósticos

Antes de que la herramienta de diagnóstico esté disponible para los usuarios, asegúrese de que tienen los siguientes permisos:

- Los usuarios necesitan acceso de lectura para Log Analytics. Para más información, consulte [Introducción a roles, permisos y seguridad con Azure Monitor](/azure/azure-monitor/platform/roles-permissions-security).
-  Los usuarios también necesitan acceso de lectura para el inquilino de Windows Virtual Desktop (rol de lector de RDS). Para más información, consulte [Acceso delegado en Windows Virtual Desktop](delegated-access-virtual-desktop-2019.md).

También debe proporcionar a los usuarios la siguiente información:

- La dirección URL de la aplicación
- Los nombres de los inquilinos individuales del grupo de inquilinos a los que pueden acceder.

## <a name="use-the-diagnostics-tool"></a>Uso de la herramienta de diagnósticos

Cuando haya iniciado sesión en la cuenta con la información que ha recibido de la organización, tenga el UPN preparado para el usuario para el que desea consultar las actividades. Una búsqueda le proporcionará todas las actividades en el tipo de actividad especificado que se produjeron en la última semana.

### <a name="how-to-read-activity-search-results"></a>Procedimiento para leer los resultados de la búsqueda de actividades

Las actividades se ordenan por marca de tiempo, con la actividad más reciente en primer lugar. Si los resultados devuelven un error, primero compruebe si se trata de un error de servicio. Para los errores de servicio, cree una incidencia de soporte técnico con la información de la actividad para ayudarnos a depurar el problema. Normalmente, el usuario o el administrador pueden resolver el resto de tipos de errores. Para obtener una lista de los escenarios de error más comunes y cómo resolverlos, consulte [Identificación y diagnóstico de problemas](diagnostics-role-service-2019.md#common-error-scenarios).

>[!NOTE]
>Los errores de servicio se denominan "errores externos" en la documentación vinculada. Se cambiará cuando se actualice la referencia de PowerShell.

Las actividades de conexión podrían tener más de un error. Puede expandir el tipo de actividad para ver cualquier otro error que haya incluido el usuario. Seleccione el nombre del código de error para abrir un cuadro de diálogo y ver más información sobre él.

### <a name="investigate-the-session-host"></a>Investigación del host de sesión

En los resultados de la búsqueda, busque y seleccione el host de sesión sobre el que desea obtener información.

Puede analizar el estado del host de sesión:

- En función de un umbral predefinido, puede recuperar la información de estado del host de sesión que consulta Log Analytics.
- Cuando no hay ninguna actividad o el host de sesión no está conectado a Log Analytics, la información no estará disponible.

También puede interactuar con usuarios en el host de sesión:

- Puede cerrar la sesión o enviar un mensaje a los usuarios que han iniciado sesión.
- El usuario que buscó originalmente está seleccionado de forma predeterminada, pero también puede seleccionar usuarios adicionales para enviar mensajes o cerrar la sesión de varios usuarios a la vez.

### <a name="windows-performance-counter-thresholds"></a>Umbrales del contador de rendimiento de Windows

- Disco lógico(\*)\\% espacio disponible:

    - muestra el porcentaje del total de espacio disponible en el disco lógico que es gratis.
    - Umbral: Un valor inferior al 20 % se marca como incorrecto.

- Disco lógico (C:)\\Longitud promedio de la cola de disco:

    - representa las condiciones del sistema de almacenamiento.
    - Umbral: Un valor superior a 5 se marca como incorrecto.

- Memoria(\*)\\Mbytes disponibles:

    - memoria disponible para el sistema.
    - Umbral: Un valor inferior a 500 megabytes se marca como incorrectos.

- Información del procesador(\*)\\Tiempo de procesador:

    - Umbral: Un valor superior al 80 % se marca como incorrecto.

- [Retraso de entrada de usuario por sesión(\*)\\Retraso máximo de entrada](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/):

    - Umbral: Un valor superior a 2000 ms se marca como incorrecto.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo supervisar los registros de actividad en [Uso de diagnósticos con Log Analytics](diagnostics-log-analytics-2019.md).
- Obtenga información sobre los escenarios de error más comunes y cómo corregirlos en [Identificación y diagnóstico de problemas](diagnostics-role-service-2019.md).
