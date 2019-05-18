---
title: Instalación del agente de Azure AD Connect Health | Microsoft Docs
description: Página de Azure AD Connect Health que describe la instalación del agente para AD FS y sincronización.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
editor: curtand
ms.assetid: 1cc8ae90-607d-4925-9c30-6770a4bd1b4e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/18/2017
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c3d1a8afdbad1878f4ce134edeeb95dad79e98a1
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784828"
---
# <a name="azure-ad-connect-health-agent-installation"></a>Instalación del agente de Azure AD Connect Health

Este documento le guiará en la instalación y configuración de los agentes de Azure AD Connect Health. Puede descargar los agentes [aquí](how-to-connect-install-roadmap.md#download-and-install-azure-ad-connect-health-agent).

## <a name="requirements"></a>Requisitos

En la tabla siguiente hay una lista de requisitos para utilizar Azure AD Connect Health.

| Requisito | DESCRIPCIÓN |
| --- | --- |
| Azure AD Premium |Azure AD Connect Health es una característica de Azure AD Premium y, por tanto, requiere Azure AD Premium. <br /><br />Para más información, consulte [Introducción a Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md). <br />Para iniciar una evaluación gratuita de 30 días, consulte cómo [iniciar una evaluación](https://azure.microsoft.com/trial/get-started-active-directory/). |
| Debe ser un administrador global de su entorno de Azure AD para empezar a trabajar con Azure AD Connect Health |De forma predeterminada, solo los administradores globales pueden instalar y configurar los agentes de mantenimiento para empezar, acceder al portal y realizar operaciones en Azure AD Connect Health. Para más información, consulte [Administración del directorio de Azure AD](../fundamentals/active-directory-administer.md). <br /><br /> El uso de control de acceso basado en rol puede permitir que otros usuarios de la organización accedan a Azure AD Connect Health. Para más información, consulte cómo usar el [control de acceso basado en rol para Azure AD Connect Health](how-to-connect-health-operations.md#manage-access-with-role-based-access-control). <br /><br />**Importante:** la cuenta que utilice al instalar los agentes debe ser una cuenta profesional o educativa. No puede ser una cuenta Microsoft. Para más información, consulte [Inicio de sesión en Azure como una organización](../fundamentals/sign-up-organization.md). |
| El agente de Azure AD Connect Health está instalado en cada servidor de destino. | Azure AD Connect Health requiere que se instalen y configuren agentes de mantenimiento en servidores de destino para recibir los datos y proporcionar las funcionalidades de supervisión y análisis. <br /><br />Por ejemplo, para obtener datos sobre su infraestructura de AD FS, el agente se debe instalar en los servidores proxy de AD FS y en los servidores proxy de aplicación web. De igual manera, para obtener datos sobre su infraestructura local de AD DS, el agente se debe instalar en los controladores de dominio. <br /><br /> |
| Conectividad saliente a los extremos del servicio de Azure | Durante la instalación y el tiempo de ejecución, el agente requiere conectividad a los puntos de conexión del servicio de Azure AD Connect Health. Si la conectividad de salida está bloqueada mediante firewalls, asegúrese de agregar los siguientes puntos de conexión a la lista de elementos permitidos: Vea la sección sobre [conectividad de salida (puntos de conexión)](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints) |
|Conectividad saliente basada en direcciones IP | Para el filtrado basado en direcciones IP en los firewalls, consulte los [intervalos de IP de Azure](https://www.microsoft.com/download/details.aspx?id=41653).|
| La inspección de SSL para el tráfico saliente se filtra o se deshabilita. | Las operaciones de carga de datos o de paso de registro de agente pueden producir un error si se produce la inspección de SSL o la finalización del tráfico saliente en el nivel de red. Más información acerca de [cómo configurar la inspección SSL](https://technet.microsoft.com/library/ee796230.aspx) |
| Puertos del firewall en el servidor que ejecuta el agente |El agente requiere que los siguientes puertos de firewall estén abiertos para poder comunicarse con los puntos de conexión de Azure AD Health.<br /><br /><li>Puerto TCP 443</li><li>Puerto TCP 5671</li> <br />Tenga en cuenta que el puerto 5671 ya no es necesario para la versión más reciente del agente. Actualice a la versión más reciente, para que solo el puerto 443 sea necesario. Más información sobre cómo [habilitar los puertos del firewall](https://technet.microsoft.com/library/ms345310(v=sql.100).aspx) |
| Permitir los siguientes sitios web si la seguridad mejorada de IE está habilitada |Si está habilitada la seguridad mejorada de Internet Explorer, los siguientes sitios web se deben permitir en el servidor en el que estará instalado el agente.<br /><br /><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com</li><li>https:\//login.windows.net</li><li>El servidor de federación de su organización en el que confía Azure Active Directory. Por ejemplo: https:\//sts.contoso.com</li> Más información sobre [cómo configurar Internet Explorer](https://support.microsoft.com/help/815141/internet-explorer-enhanced-security-configuration-changes-the-browsing) |
| Asegúrese de que PowerShell v4.0 o posterior está instalado | <li>Windows Server 2008 R2 se suministra con PowerShell 2.0, que no es suficiente para el agente. Actualice PowerShell como se explica a continuación en [Instalación del agente en servidores de Windows Server 2008 R2](#agent-installation-on-windows-server-2008-r2-servers).</li><li>Windows Server 2012 se suministra con PowerShell 3.0, que no es suficiente para el agente.  [Actualice](https://www.microsoft.com/download/details.aspx?id=40855) Windows Management Framework.</li><li>Windows Server 2012 R2 y las versiones posteriores se suministran con una versión lo suficientemente reciente de PowerShell.</li>|
|Deshabilitar FIPS|FIPS no es compatible con los agentes de Azure AD Connect Health.|

### <a name="outbound-connectivity-to-the-azure-service-endpoints"></a>Conectividad saliente a los extremos del servicio de Azure

 Durante la instalación y el tiempo de ejecución, el agente requiere conectividad a los puntos de conexión del servicio de Azure AD Connect Health. Si la conectividad de salida está bloqueada mediante firewalls, asegúrese de que las siguientes direcciones URL no están bloqueadas de forma predeterminada. No deshabilite la supervisión de seguridad ni la inspección de estas direcciones URL, pero permítalas como haría con otro tráfico de Internet. Estas direcciones permiten la comunicación con los puntos de conexión de servicio de Azure AD Connect Health. Obtenga información sobre cómo [comprobar la conectividad saliente con Test-AzureADConnectHealthConnectivity](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-health-agent-install#test-connectivity-to-azure-ad-connect-health-service).

| Entorno de dominio | Puntos de conexión del servicio de Azure necesarios |
| --- | --- |
| Público general | <li>&#42;.blob.core.windows.net </li><li>&#42;.aadconnecthealth.azure.com </li><li>&#42;.servicebus.windows.net - Puerto: 5671 </li><li>&#42;.adhybridhealth.azure.com/</li><li>https:\//management.azure.com </li><li>https:\//policykeyservice.dc.ad.msft.net/</li><li>https:\//login.windows.net</li><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com *este punto de conexión solo se utiliza para la detección durante el registro.</li> |
| Azure Alemania | <li>&#42;.blob.core.cloudapi.de </li><li>&#42;.servicebus.cloudapi.de </li> <li>&#42;.aadconnecthealth.microsoftazure.de </li><li>https:\//management.microsoftazure.de </li><li>https:\//policykeyservice.aadcdi.microsoftazure.de </li><li>https:\//login.microsoftonline.de </li><li>https:\//secure.aadcdn.microsoftonline-p.de </li><li>https:\//www.office.de *este punto de conexión solo se utiliza para la detección durante el registro.</li> |
| Azure Government | <li>&#42;.blob.core.usgovcloudapi.net </li> <li>&#42;.servicebus.usgovcloudapi.net </li> <li>&#42;.aadconnecthealth.microsoftazure.us </li> <li>https:\//management.usgovcloudapi.net </li><li>https:\//policykeyservice.aadcdi.azure.us </li><li>https:\//login.microsoftonline.us </li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com *este punto de conexión solo se utiliza para la detección durante el registro.</li> |


## <a name="download-and-install-the-azure-ad-connect-health-agent"></a>Descarga e instalación de un agente de Azure AD Connect Health

* Asegúrese de que [cumple los requisitos](how-to-connect-health-agent-install.md#requirements) de Azure AD Connect Health.
* Introducción a Azure AD Connect Health para AD FS
    * [Descargue el agente de Azure AD Connect Health para AD FS.](https://go.microsoft.com/fwlink/?LinkID=518973)
    * [Consulte las instrucciones de instalación](#installing-the-azure-ad-connect-health-agent-for-ad-fs).
* Introducción al uso de Azure AD Connect Health para la sincronización
    * [Descargue e instale la versión más reciente de Azure AD Connect](https://go.microsoft.com/fwlink/?linkid=615771). El agente de Health para la sincronización se instalará como parte de la instalación de Azure AD Connect (versión 1.0.9125.0 o superior).
* Introducción a Azure AD Connect Health para AD DS
    * [Descargue el agente de Azure AD Connect Health para AD DS](https://go.microsoft.com/fwlink/?LinkID=820540).
    * [Consulte las instrucciones de instalación](#installing-the-azure-ad-connect-health-agent-for-ad-ds).

## <a name="installing-the-azure-ad-connect-health-agent-for-ad-fs"></a>Instalación del agente de Azure AD Connect Health para AD FS

> [!NOTE]
> El servidor de AD FS no puede ser el mismo que el servidor de sincronización. No instale el agente de AD FS en su servidor de sincronización.
>

Antes de la instalación, asegúrese de que el nombre de host del servidor de AD FS es único y no está presente en el servicio AD FS.
Para iniciar la instalación del agente, haga doble clic en el archivo .exe que descargó. En la primera pantalla, haga clic en Instalar.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/install1.png)

Una vez que finalice la instalación, haga clic en Configurar ahora.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/install2.png)

Se muestra una ventana de PowerShell para iniciar el proceso de registro del agente. Cuando se le solicite, inicie sesión con una cuenta de Azure AD que tenga acceso para realizar el registro del agente. De forma predeterminada, tiene acceso la cuenta de administrador global.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/install3.png)

Después de iniciar sesión, PowerShell continuará. Una vez que termine, puede cerrar PowerShell y la configuración estará completa.

En este momento, se deben iniciar automáticamente los servicios del agente que le permiten cargar los datos necesarios en el servicio en la nube de forma segura.

Si no ha cumplido todos los requisitos previos descritos en las secciones anteriores, verá advertencias en la ventana de PowerShell. Asegúrese de completar todos los [requisitos](how-to-connect-health-agent-install.md#requirements) antes de instalar el agente. La captura de pantalla siguiente es un ejemplo de estos errores.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/install4.png)

Para comprobar que se instaló el agente, busque los siguientes servicios en el servidor. Si completó la configuración, estos servicios deben aparecer en ejecución. De lo contrario, estarán detenidos hasta que se complete la configuración.

* Servicio de diagnóstico de AD FS de Azure AD Connect Health
* Servicio de análisis de AD FS de Azure AD Connect Health
* Servicio de supervisión de AD FS de Azure AD Connect Health

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/install5.png)

### <a name="agent-installation-on-windows-server-2008-r2-servers"></a>Instalación del agente en servidores de Windows Server 2008 R2

En el caso de los servidores de Windows Server 2008 R2, realice los siguientes pasos:

1. Asegúrese de que se esté ejecutando el Service Pack 1 o posterior del servidor.
2. Desactive ESC de Internet Explorer para la instalación del agente:
3. Instale Windows PowerShell 4.0 en cada uno de los servidores antes de instalar el agente de AD Health. Para instalar Windows PowerShell 4.0:
   * Instale [Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=40779) mediante el vínculo siguiente para descargar el instalador sin conexión.
   * Instale PowerShell ISE (desde Características de Windows)
   * Instale [Windows Management Framework 4.0.](https://www.microsoft.com/download/details.aspx?id=40855)
   * Instale Internet Explorer versión 10 o posterior en el servidor. (Necesario para que el servicio de mantenimiento autentique su identidad con las credenciales de administrador de Azure).
4. Para más información acerca de cómo instalar Windows PowerShell 4.0 en Windows Server 2008 R2, consulte el artículo wiki [aquí](https://social.technet.microsoft.com/wiki/contents/articles/20623.step-by-step-upgrading-the-powershell-version-4-on-2008-r2.aspx).

### <a name="enable-auditing-for-ad-fs"></a>Habilitación de la auditoría de AD FS

> [!NOTE]
> Esta sección solo se aplica a los servidores de AD FS. No es necesario seguir estos pasos en los servidores proxy de aplicación web.
>

Para que la característica Análisis de uso pueda recopilar y analizar datos, el agente de Azure AD Connect Health necesita la información de los registros de auditoría de AD FS. que no están habilitados de forma predeterminada. Use los procedimientos siguientes para habilitar la auditoría de AD FS y localizar los registros de auditoría de AD FS en los servidores correspondientes.

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2008-r2"></a>Para habilitar la auditoría de AD FS en Windows Server 2008 R2

1. Haga clic en **Inicio**, seleccione **Programas**, **Herramientas administrativas** y luego haga clic en **Directiva de seguridad local**.
2. Navegue hasta la carpeta **Configuración de seguridad\Directivas locales\Asignación de derechos de usuario** y haga doble clic en **Generar auditorías de seguridad**.
3. En la pestaña **Configuración de seguridad local** , compruebe que aparezca la cuenta de servicio de AD FS 2.0. Si no aparece, haga clic en **Agregar usuario o grupo**, agréguela a la lista y luego haga clic en **Aceptar**.
4. Para habilitar la auditoría, abra un símbolo del sistema con privilegios elevados y ejecute el siguiente comando: <code>auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable</code>.
5. Cierre **Directiva de seguridad local**.
<br />   -- **Los pasos siguientes solo son necesarios en los servidores principales de AD FS.** -- <br />
6. Abra el complemento **Administración de AD FS**. Para abrir el complemento Administración de AD FS, haga clic en **Inicio**, seleccione **Programas**, **Herramientas administrativas** y luego haga clic en **Administración de AD FS 2.0**.
7. En el panel **Acciones**, haga clic en **Editar propiedades del servicio de federación**.
8. En el cuadro de diálogo **Propiedades del servicio de federación**, haga clic en la pestaña **Eventos**.
9. Active las casillas **Auditorías de aciertos** y **Auditorías de errores**.
10. Haga clic en **OK**.

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2012-r2"></a>Para habilitar la auditoría de AD FS en Windows Server 2012 R2

1. Abra **Directiva de seguridad local**; para ello, abra **Administrador del servidor** en la pantalla Inicio o Administrador del servidor en la barra de tareas del escritorio y luego haga clic en **Herramientas/Directiva de seguridad local**.
2. Navegue hasta la carpeta **Configuración de seguridad\Directivas locales\Asignación de derechos de usuario** y haga doble clic en **Generar auditorías de seguridad**.
3. En la pestaña **Configuración de seguridad local** , compruebe que aparezca la cuenta de servicio de AD FS. Si no aparece, haga clic en **Agregar usuario o grupo**, agréguela a la lista y luego haga clic en **Aceptar**.
4. Para habilitar la auditoría, abra un símbolo del sistema con privilegios elevados y ejecute el siguiente comando: ```auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable```
5. Cierre **Directiva de seguridad local**.
<br />   -- **Los pasos siguientes solo son necesarios en los servidores principales de AD FS.** -- <br />
6. Abra el complemento **Administración de AD FS** (en Administrador del servidor, haga clic en Herramientas y luego seleccione Administración de AD FS).
7. En el panel **Acciones**, haga clic en **Editar propiedades del servicio de federación**.
8. En el cuadro de diálogo **Propiedades del servicio de federación**, haga clic en la pestaña **Eventos**.
9. Active las casillas **Auditorías de aciertos y Auditorías de errores** y luego haga clic en **Aceptar**.

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2016"></a>Para habilitar la auditoría de AD FS en Windows Server 2016

1. Abra **Directiva de seguridad local**; para ello, abra **Administrador del servidor** en la pantalla Inicio o Administrador del servidor en la barra de tareas del escritorio y luego haga clic en **Herramientas/Directiva de seguridad local**.
2. Navegue hasta la carpeta **Configuración de seguridad\Directivas locales\Asignación de derechos de usuario** y haga doble clic en **Generar auditorías de seguridad**.
3. En la pestaña **Configuración de seguridad local** , compruebe que aparezca la cuenta de servicio de AD FS. Si no aparece, haga clic en **Agregar usuario o grupo**, agregue la cuenta de servicio de AD FS a la lista y después haga clic en **Aceptar**.
4. Para habilitar la auditoría, abra un símbolo del sistema con privilegios elevados y ejecute el siguiente comando: <code>auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable</code>
5. Cierre **Directiva de seguridad local**.
<br />   -- **Los pasos siguientes solo son necesarios en los servidores principales de AD FS.** -- <br />
6. Abra el complemento **Administración de AD FS** (en Administrador del servidor, haga clic en Herramientas y luego seleccione Administración de AD FS).
7. En el panel **Acciones**, haga clic en **Editar propiedades del servicio de federación**.
8. En el cuadro de diálogo **Propiedades del servicio de federación**, haga clic en la pestaña **Eventos**.
9. Active las casillas **Auditorías de aciertos y Auditorías de errores** y luego haga clic en **Aceptar**. Esto se debe habilitar de forma predeterminada.
10. Abra una ventana de PowerShell y ejecute el siguiente comando: ```Set-AdfsProperties -AuditLevel Verbose```.

Tenga en cuenta que el nivel de auditoría "básico" está habilitado de forma predeterminada. Obtenga más información en [Mejoras de auditoría de AD FS en Windows Server 2016](https://docs.microsoft.com/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server).


#### <a name="to-locate-the-ad-fs-audit-logs"></a>Para buscar los registros de auditoría de AD FS

1. Abra **Visor de eventos**.
2. Vaya a Registros de Windows y seleccione **Seguridad**.
3. A la derecha, haga clic en **Filtrar registros actuales**.
4. En Origen de evento, seleccione **Auditoría de AD FS**.

    Y [consulte las preguntas más frecuentes](reference-connect-health-faq.md#operations-questions) sobre registros de auditoría.

![Registros de auditoría de AD FS](./media/how-to-connect-health-agent-install/adfsaudit.png)

> [!WARNING]
> Una directiva de grupo puede deshabilitar la auditoría de AD FS. Si se deshabilita la auditoría de AD FS, no estará disponible el análisis de uso sobre las actividades de inicio de sesión. Asegúrese de no tener ninguna directiva de grupo de este tipo.>
>


## <a name="installing-the-azure-ad-connect-health-agent-for-sync"></a>Instalación del agente de Azure AD Connect Health para sincronización

El agente de Azure AD Connect Health para sincronización se instala automáticamente en la última compilación de Azure AD Connect. Para usar Azure AD Connect para sincronización, debe descargar la versión más reciente de Azure AD Connect e instalarla. Puede descargar la versión más reciente [aquí](https://www.microsoft.com/download/details.aspx?id=47594).

Para comprobar que se instaló el agente, busque los siguientes servicios en el servidor. Si completó la configuración, estos servicios deben aparecer en ejecución. De lo contrario, estarán detenidos hasta que se complete la configuración.

* Servicio de análisis de sincronización de Azure AD Connect Health
* Servicio de supervisión de sincronización de Azure AD Connect Health

![Comprobación de Azure AD Connect Health para sincronización](./media/how-to-connect-health-agent-install/services.png)

> [!NOTE]
> Recuerde que el uso de Azure AD Connect Health requiere Azure AD Premium. Si no tiene Azure AD Premium no podrá completar la configuración en Azure Portal. Para más información, consulte la [página de requisitos](how-to-connect-health-agent-install.md#requirements).
>
>

## <a name="manual-azure-ad-connect-health-for-sync-registration"></a>Registro manual de Azure AD Connect Health para sincronización

Si se produce un error del agente de registro de Azure AD Connect Health para sincronización después de instalar correctamente Azure AD Connect, puede usar el siguiente comando de PowerShell para registrar el agente manualmente.

> [!IMPORTANT]
> El uso de este comando de PowerShell solo es necesario si se produce un error en el registro del agente después de instalar Azure AD Connect.
>
>

El siguiente comando de PowerShell solo es necesario cuando se produce un error en el registro del agente de mantenimiento incluso después de una correcta instalación y configuración de Azure AD Connect. Los servicios de Azure AD Connect Health se iniciarán después de que el agente se registre correctamente.

Puede registrar manualmente el agente de Azure AD Connect Health para sincronización con el siguiente comando de PowerShell:

`Register-AzureADConnectHealthSyncAgent -AttributeFiltering $false -StagingMode $false`

El comando toma los siguientes parámetros:

* AttributeFiltering: $true (predeterminado) si Azure AD Connect no está sincronizando el conjunto de atributos predeterminado y se ha personalizado para usar un conjunto de atributos filtrados. $false en caso contrario.
* StagingMode: $false (predeterminado) si el servidor de Azure AD Connect no está en modo de almacenamiento provisional; $true si el servidor está configurado en modo de almacenamiento provisional.

Cuando se le solicite autenticación, debe usar la misma cuenta de administrador global (como admin@domain.onmicrosoft.com) que se usó para configurar Azure AD Connect.

## <a name="installing-the-azure-ad-connect-health-agent-for-ad-ds"></a>Instalación del agente de Azure AD Connect Health para AD DS

Para iniciar la instalación del agente, haga doble clic en el archivo .exe que descargó. En la primera pantalla, haga clic en Instalar.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install1.png)

Una vez que finalice la instalación, haga clic en Configurar ahora.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install2.png)

Con esto se iniciará un símbolo del sistema seguido de una instancia de PowerShell que ejecutará Register-AzureADConnectHealthADDSAgent. Cuando se le solicite que inicie sesión en Azure, proceda e inicie sesión.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install3.png)

Después de iniciar sesión, PowerShell continuará. Una vez que termine, puede cerrar PowerShell y la configuración estará completa.

En este punto, los servicios se deberían iniciar automáticamente permitiendo que el agente supervise y recopile datos. Si no ha cumplido todos los requisitos previos descritos en las secciones anteriores, verá advertencias en la ventana de PowerShell. Asegúrese de completar todos los [requisitos](how-to-connect-health-agent-install.md#requirements) antes de instalar el agente. La captura de pantalla siguiente es un ejemplo de estos errores.

![Comprobación de Azure AD Connect Health para AD DS](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install4.png)

Para comprobar que se instaló el agente, busque los siguientes servicios en el controlador de dominio.

* Servicio de análisis de AD DS de Azure AD Connect Health
* Servicio de supervisión de AD DS de Azure AD Connect Health

Si completó la configuración, estos servicios deben aparecer ya en ejecución. De lo contrario, estarán detenidos hasta que se complete la configuración.

![Comprobación de Azure AD Connect Health](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install5.png)

### <a name="quick-agent-installation-in-multiple-servers"></a>Instalación del agente rápido en varios servidores

1. Crear una cuenta de usuario en Azure AD con una contraseña.
2. Asignar el **propietario** rol para esta cuenta AAD local en Azure AD Connect Health a través del portal. Siga los pasos [aquí](how-to-connect-health-operations.md#manage-access-with-role-based-access-control). Asigne el rol para todas las instancias de servicio. 
3. Descargue el archivo MSI .exe en el controlador de dominio local para la instalación.
4. Ejecute el siguiente script para el registro. Reemplace los parámetros con la nueva cuenta de usuario creada y su contraseña. 

```powershell
AdHealthAddsAgentSetup.exe /quiet
Start-Sleep 30
$userName = "NEWUSER@DOMAIN"
$secpasswd = ConvertTo-SecureString "PASSWORD" -AsPlainText -Force
$myCreds = New-Object System.Management.Automation.PSCredential ($userName, $secpasswd)
import-module "C:\Program Files\Azure Ad Connect Health Adds Agent\PowerShell\AdHealthAdds"
 
Register-AzureADConnectHealthADDSAgent -UserPrincipalName $USERNAME -Credential $myCreds

```

1. Cuando haya terminado, puede quitar el acceso para la cuenta local, realice una o varias de las siguientes acciones: 
    * Quitar la asignación de roles para la cuenta local para AAD Connect Health
    * Rotar la contraseña para la cuenta local. 
    * Deshabilitar la cuenta local de AAD
    * Eliminar la cuenta local de AAD  

## <a name="agent-registration-using-powershell"></a>Registro del agente con PowerShell

Después de instalar el ejecutable setup.exe del agente adecuado, puede realizar el paso de registro del agente con los siguientes comandos de PowerShell en función del rol. Abra una ventana de PowerShell y ejecute el comando adecuado:

```powershell
    Register-AzureADConnectHealthADFSAgent
    Register-AzureADConnectHealthADDSAgent
    Register-AzureADConnectHealthSyncAgent

```

Estos comandos aceptan "Credential" como parámetro para completar el registro de forma no interactiva o en una máquina de Server Core.
* El elemento Credential puede capturarse en una variable de PowerShell que se pasa como un parámetro.
* Puede proporcionar cualquier identidad de Azure AD a la que tenga acceso para registrar los agentes y que no tenga MFA habilitado.
* De forma predeterminada los administradores globales tienen acceso para realizar el registro del agente. También puede permitir otras identidades con menos privilegios para realizar este paso. Más información sobre el [control de acceso basado en rol](how-to-connect-health-operations.md#manage-access-with-role-based-access-control).

```powershell
    $cred = Get-Credential
    Register-AzureADConnectHealthADFSAgent -Credential $cred

```

## <a name="configure-azure-ad-connect-health-agents-to-use-http-proxy"></a>Configuración de agentes de Azure AD Connect Health para usar el proxy HTTP

Puede configurar agentes de Azure AD Connect Health para trabajar con un proxy HTTP

> [!NOTE]
> * No se admite el uso de “Netsh WinHttp set ProxyServerAddress”, ya que el agente utiliza System.Net para realizar solicitudes web en lugar de los servicios HTTP de Microsoft Windows.
> * Se utilizará la dirección del proxy HTTP configurada para transmitir mensajes HTTPS cifrados.
> * No se admiten los servidores proxy autenticados (mediante HTTPBasic).
>
>

### <a name="change-health-agent-proxy-configuration"></a>Cambio de configuración del proxy del agente de estado

Tiene las siguientes opciones para configurar el agente de Azure AD Connect Health para utilizar a un proxy HTTP.

> [!NOTE]
> Debe reiniciar todos los servicios del agente de Azure AD Connect Health para que se actualice la configuración de proxy. Ejecute el siguiente comando:<br />
>  Restart-Service AdHealth*
>
>

#### <a name="import-existing-proxy-settings"></a>Importación de configuración de proxy existente

##### <a name="import-from-internet-explorer"></a>Importación desde Internet Explorer.

La configuración del proxy HTTP de Internet Explorer se puede importar para que los agentes de mantenimiento de Azure AD Connect Health la usen. En cada uno de los servidores que ejecutan al agente de mantenimiento, ejecute el siguiente comando de PowerShell:

    Set-AzureAdConnectHealthProxySettings -ImportFromInternetSettings

##### <a name="import-from-winhttp"></a>Importación desde WinHTTP

La configuración del proxy WinHTTP se puede importar para que los agentes de mantenimiento de Azure AD Connect Health la usen. En cada uno de los servidores que ejecutan al agente de mantenimiento, ejecute el siguiente comando de PowerShell:

    Set-AzureAdConnectHealthProxySettings -ImportFromWinHttp

#### <a name="specify-proxy-addresses-manually"></a>Especificación manual de direcciones de proxy

Puede especificar un servidor proxy manualmente en cada uno de los servidores en los que se ejecuta el agente de mantenimiento mediante el siguiente comando de PowerShell:

    Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress address:port

Ejemplo: *Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress myproxyserver: 443*

* "address" puede ser un nombre de servidor resoluble DNS o una dirección IPv4
* "port" se puede omitir. Si se omite, se elige 443 como puerto predeterminado.

#### <a name="clear-existing-proxy-configuration"></a>Borrado de la configuración de proxy existente

Puede borrar la configuración de proxy existente ejecutando el comando siguiente:

    Set-AzureAdConnectHealthProxySettings -NoProxy


### <a name="read-current-proxy-settings"></a>Lectura de la configuración de proxy actual

Puede ejecutar el comando siguiente para leer la configuración de proxy definida actualmente:

    Get-AzureAdConnectHealthProxySettings


## <a name="test-connectivity-to-azure-ad-connect-health-service"></a>Prueba de la conectividad al servicio Azure AD Connect Health

Pueden surgir problemas que hagan que el agente de Azure AD Connect Health pierda la conectividad con el servicio Azure AD Connect Health, por ejemplo, problemas de red, problemas de permisos o de otro tipo.

Si el agente no puede enviar datos al servicio Azure AD Connect Health durante más de dos horas, aparecerá la siguiente alerta en el portal: "Los datos del Servicio de mantenimiento no están actualizados". Puede confirmar si el agente de Azure AD Connect Health es capaz de cargar datos en el servicio de Azure AD Connect Health ejecutando el siguiente comando de PowerShell:

    Test-AzureADConnectHealthConnectivity -Role ADFS

El parámetro de rol tiene actualmente los siguientes valores:

* ADFS
* Sync
* ADDS

> [!NOTE]
> Para poder utilizar la herramienta de conectividad, primero debe completar el registro del agente. Si no puede completar el registro del agente, asegúrese de que cumple todos los [requisitos](how-to-connect-health-agent-install.md#requirements) para Azure AD Connect Health. Esta prueba de conectividad se realiza de forma predeterminada durante el registro del agente.
>
>

## <a name="related-links"></a>Vínculos relacionados

* [Azure AD Connect Health](whatis-hybrid-identity-health.md)
* [Operaciones de Azure AD Connect Health](how-to-connect-health-operations.md)
* [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md)
* [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-sync.md)
* [Uso de Azure AD Connect Health con AD DS](how-to-connect-health-adds.md)
* [Preguntas más frecuentes de Azure AD Connect Health](reference-connect-health-faq.md)
* [Historial de versiones de Azure AD Connect Health](reference-connect-health-version-history.md)
