---
title: Implementación de estaciones de trabajo administradas por Azure con Azure Active Directory
description: Obtenga información sobre cómo implementar estaciones de trabajo seguras administradas por Azure para reducir el riesgo de que se produzcan brechas debido a una configuración incorrecta o una situación de peligro.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 11/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: frasim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 841bc3ae4fbddb376ea4da8141bf4df3f895c4dc
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269563"
---
# <a name="deploy-a-secure-azure-managed-workstation"></a>Implementación de una estación de trabajo segura administrada por Azure

Ahora que ya sabe en qué consisten las [estaciones de trabajo seguras](concept-azure-managed-workstation.md), es el momento de comenzar el proceso de implementación. Con esta guía, usará perfiles definidos para crear una estación de trabajo que sea más segura desde el principio.

![Implementación de una estación de trabajo segura](./media/howto-azure-managed-workstation/deploying-secure-workstations.png)

Seleccione un perfil antes de implementar la solución. Puede usar varios perfiles de forma simultánea en una implementación y asignarles etiquetas o grupos.

> [!NOTE]
> Aplique cualquiera de los perfiles según sea necesario para sus requisitos. Puede cambiar a otro perfil mediante su asignación en Microsoft Intune.

| Perfil | Bajo | Mejorada | Alto | Especializada | Protegido | Aislado |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Usuario de Azure AD | Sí | Sí | Sí | Sí | Sí | Sí |
| Administrado por Intune | Sí | Sí | Sí | Sí | Sí | Sí |
| Dispositivo registrado en Azure AD | Sí |  |  |  |  | |   |
| Dispositivo unido a Azure AD |   | Sí | Sí | Sí | Sí | Sí |
| Base de referencia de seguridad de Intune aplicada |   | Sí <br> (mejorado) | Sí <br> (HighSecurity) | Sí <br> (NCSC) | Sí <br> (protegido) | N/D |
| El hardware cumple los estándares seguros de Windows 10 |   | Sí | Sí | Sí | Sí | Sí |
| Protección contra amenazas avanzada de Microsoft Defender habilitada |   | Sí  | Sí | Sí | Sí | Sí |
| Eliminación de derechos de administrador |   |   | Sí  | Sí | Sí | Sí |
| Implementación mediante Microsoft Autopilot |   |   | Sí  | Sí | Sí | Sí |
| Aplicaciones instaladas solo mediante Intune |   |   |   | Sí | Sí |Sí |
| Direcciones URL restringidas a la lista aprobada |   |   |   | Sí | Sí |Sí |
| Internet bloqueada (entrante o saliente) |   |   |   |  |  |Sí |

> [!NOTE]
> En la guía de estaciones de trabajo seguras, los **dispositivos** se asignarán con perfiles y directivas. A los usuarios no se les aplican las directivas directamente, lo que permite que el uso compartido de dispositivos (dispositivos compartidos) esté en vigor. Si no se comparte una estación de trabajo segura en su implementación o se necesitan directivas de usuario individuales, la asignación de los perfiles de directiva de usuario se puede asignar al usuario y dispositivo. 

## <a name="license-requirements"></a>Requisitos de licencia

En los conceptos que se tratan en esta guía se da por supuesto que tiene Microsoft 365 Enterprise E5 o una SKU equivalente. Algunas de las recomendaciones de la guía se pueden implementar con versiones de SKU inferiores. Para obtener más información, consulte las [licencias de Microsoft 365 Enterprise](https://www.microsoft.com/licensing/product-licensing/microsoft-365-enterprise).

Para automatizar el aprovisionamiento de licencias, considere la posibilidad de usar [licencias basadas en grupos](../users-groups-roles/licensing-groups-assign.md) para los usuarios.

## <a name="azure-active-directory-configuration"></a>Configuración de Azure Active Directory

Azure Active Directory (Azure AD) administra usuarios, grupos y dispositivos para las estaciones de trabajo de administrador. Habilite las características y los servicios de identidad con una [cuenta de administrador](../users-groups-roles/directory-assign-admin-roles.md).

Cuando se crea la cuenta de administrador de la estación de trabajo protegida, la cuenta se expone a la estación de trabajo actual. Asegúrese de usar un dispositivo seguro conocido para realizar la configuración inicial y toda la configuración global. Para reducir el riesgo de ataques en la experiencia inicial, tenga en cuenta los siguientes [consejos para evitar infecciones de malware](/windows/security/threat-protection/intelligence/prevent-malware-infection).

Exija la autenticación multifactor, al menos a los administradores. Consulte más información sobre la [implementación de MFA basada en la nube](../authentication/howto-mfa-getstarted.md).

### <a name="azure-ad-users-and-groups"></a>Usuarios y grupos de Azure AD

1. En Azure Portal, vaya a **Azure Active Directory** > **Usuarios** > **Nuevo usuario**.
1. Cree el administrador del dispositivo con los pasos descritos en el [tutorial para crear un usuario](/Intune/quickstart-create-user).
1. Especifique:

   * **Nombre**: Administrador de la estación de trabajo segura.
   * **Nombre de usuario** - `secure-ws-admin@identityitpro.com`.
   * **Rol de directorio** - **Administrador limitado** y seleccione el rol **Administrador de Intune**.

1. Seleccione **Crear**.

Después, cree dos grupos: usuarios de la estación de trabajo y dispositivos de la estación de trabajo.

En Azure Portal, vaya a **Azure Active Directory** > **Grupos** > **Nuevo grupo**.

1. Para el grupo de usuarios de la estación de trabajo, es posible que le interese configurar [licencias basadas en grupos](../users-groups-roles/licensing-groups-assign.md) para automatizar el aprovisionamiento de las licencias a los usuarios.
1. Para el grupo de usuarios de la estación de trabajo, especifique lo siguiente:

   * **Tipo de grupo**: Seguridad.
   * **Nombre del grupo**: Usuarios de la estación de trabajo segura.
   * **Tipo de pertenencia**: Asignado.

1. Agregue el usuario del administrador de la estación de trabajo segura: `secure-ws-admin@identityitpro.com`.
1. Puede agregar otros usuarios que administrarán las estaciones de trabajo seguras.
1. Seleccione **Crear**.
1. Para el grupo de dispositivos de la estación de trabajo, especifique lo siguiente:

   * **Tipo de grupo**: Seguridad.
   * **Nombre del grupo**: Estaciones de trabajo seguras.
   * **Tipo de pertenencia**: Asignado.

1. Seleccione **Crear**.

### <a name="azure-ad-device-configuration"></a>Configuración de dispositivos de Azure AD

#### <a name="specify-who-can-join-devices-to-azure-ad"></a>Especificación de quién puede unir dispositivos a Azure AD

Configure la opción de los dispositivos en Active Directory para que el grupo de seguridad administrativa pueda unir dispositivos a su dominio. Para configurar esta opción desde Azure Portal:

1. Vaya a **Azure Active Directory** > **Dispositivos** > **Configuración de dispositivo**.
1. Elija **Seleccionados** en **Los usuarios pueden inscribir dispositivos en Azure AD** y, luego, haga clic en el grupo "Usuarios de la estación de trabajo segura".

#### <a name="removal-of-local-admin-rights"></a>Eliminación de derechos de administrador local

Este método requiere que los usuarios de las estaciones de trabajo de nivel seguro, VIP y DevOps no tengan derechos de administrador en los equipos. Para configurar esta opción desde Azure Portal:

1. Vaya a **Azure Active Directory** > **Dispositivos** > **Configuración de dispositivo**.
1. Seleccione **Ninguno** en **Administradores locales adicionales en dispositivos unidos a Azure AD**.

#### <a name="require-multi-factor-authentication-to-join-devices"></a>Exigencia de autenticación multifactor para unir dispositivos

Para reforzar aún más el proceso de unión de dispositivos a Azure AD:

1. Vaya a **Azure Active Directory** > **Dispositivos** > **Configuración de dispositivo**.
1. Seleccione **Sí** en **Requerir Multi-factor Authentication para conectar dispositivos**.
1. Seleccione **Guardar**.

#### <a name="configure-mobile-device-management"></a>Configuración de la administración de dispositivos móviles

En el Portal de Azure:

1. Vaya a **Azure Active Directory** > **Movilidad (MDM y MAM)**  > **Microsoft Intune**.
1. Cambie el valor de **Ámbito de usuario de MDM** a **Todos**.
1. Seleccione **Guardar**.

Estos pasos le permiten administrar cualquier dispositivo con Intune. Para obtener más información, consulte [Inicio rápido de Intune: Configurar la inscripción automática para dispositivos Windows 10](/Intune/quickstart-setup-auto-enrollment). Creará directivas de cumplimiento y configuración de Intune en un paso posterior.

#### <a name="azure-ad-conditional-access"></a>Acceso condicional de Azure AD

El acceso condicional de Azure AD puede ayudarle a restringir tareas administrativas con privilegios a dispositivos compatibles. Los miembros predefinidos del grupo **Usuarios de la estación de trabajo segura** deben superar una autenticación multifactor para iniciar sesión en aplicaciones en la nube. Se recomienda excluir de la directiva las cuentas de acceso de emergencia. Para obtener más información, vea [Administración de cuentas de acceso de emergencia en Azure AD](../users-groups-roles/directory-emergency-access.md).

## <a name="intune-configuration"></a>Configuración de Intune

### <a name="configure-enrollment-status"></a>Configuración del estado de la inscripción

Es importante asegurarse de que la estación de trabajo segura es un dispositivo limpio de confianza. Al comprar dispositivos nuevos, puede insistir en que se establezcan en la configuración de fábrica de [Windows 10 Pro en modo S](/Windows/deployment/Windows-10-pro-in-s-mode), lo que limita la exposición a las vulnerabilidades durante la administración de la cadena de suministro. Después de recibir un dispositivo de su proveedor, puede usar Autopilot para cambiarlo del modo S. A continuación se explica cómo aplicar el proceso de transformación.

Para asegurarse de que los dispositivos estén completamente configurados antes de su uso, Intune proporciona un medio de **bloquear el uso del dispositivo hasta que todos los perfiles y aplicaciones estén instalados**.

Desde **Azure Portal**:

1. Vaya a **Microsoft Intune** > **Inscripción de dispositivos** > **Inscripción de Windows** > **Página de estado de la inscripción** > **Predeterminado** > **Configuración**.
1. Establezca **Mostrar el progreso de la instalación de la aplicación y el perfil** en **Sí**.
1. Establezca **Bloquear el uso del dispositivo hasta que todos los perfiles y aplicaciones estén instalados** en **Sí**.

### <a name="create-an-autopilot-deployment-profile"></a>Creación de un perfil de implementación de Autopilot

Después de crear un grupo de dispositivos, debe crear un perfil de implementación para configurar los dispositivos Autopilot.

En Intune en Azure Portal:

1. Seleccione **Inscripción de dispositivos** > **Inscripción de Windows** > **Perfiles de implementación** > **Crear perfil** .
1. Especifique:

   * Nombre: **Perfil de implementación de estación de trabajo segura**.
   * Descripción: **Implementación de estaciones de trabajo seguras**.
   * Establezca **Convertir todos los dispositivos de destino a Autopilot** en **Sí**. Esta configuración garantiza que todos los dispositivos de la lista se registren con el servicio de implementación de Autopilot. Espere 48 horas a que se procese el registro.

1. Seleccione **Next** (Siguiente).

   * Para el **Modo de implementación**, elija **Implementación automática (versión preliminar)** . Los dispositivos con este perfil se asocian al usuario que inscribe el dispositivo. Se necesitan credenciales de usuario para inscribir el dispositivo. Es esencial tener en cuenta que la implementación de un dispositivo en el modo **Implementación automática**  le permitirá implementar equipos portátiles en un modelo compartido. No se producirá ninguna asignación de usuario hasta que el dispositivo se asigne a un usuario por primera vez. Por consiguiente, las directivas de usuario como BitLocker no se habilitarán hasta que se complete una asignación de usuario. Para obtener más información sobre cómo iniciar sesión en un dispositivo protegido, vea la información relacionada con los [perfiles seleccionados](/intune/device-profile-assign).
   * En el cuadro **Unirse a Azure AD como** debe aparecer **Unido a Azure AD** atenuado.
   * Seleccione el Idioma (región) y el tipo de cuenta de usuario **Estándar**. 

1. Seleccione **Next** (Siguiente).

   * Seleccione una etiqueta de ámbito si ha preconfigurado una.

1. Seleccione **Next** (Siguiente).
1. Elija **Asignaciones** > **Asignar a** > **Grupos seleccionados**. En **Seleccionar grupos para incluir**, elija **Estaciones de trabajo seguras**.
1. Seleccione **Next** (Siguiente).
1. Seleccione **Crear** para crear el perfil. El perfil de implementación de Autopilot ya está disponible para asignarse a dispositivos.

La inscripción de dispositivos en AutoPilot proporciona una experiencia de usuario diferente en función del tipo de dispositivo y el rol. En nuestro ejemplo de implementación, se muestra un modelo en el que los dispositivos protegidos se implementan de forma masiva y se pueden compartir, pero cuando se usan por primera vez, los dispositivos se asignan a un usuario. Para más información, consulte [¿Qué es la inscripción de dispositivos?](/intune/device-enrollment).

### <a name="configure-windows-update"></a>Configuración de Windows Update

Una de las cosas más importantes que puede hacer es mantener Windows 10 actualizado. Para mantener Windows en un estado seguro, debe implementar un anillo de actualización con el fin de administrar el ritmo al que se aplican las actualizaciones a las estaciones de trabajo. 

En esta guía le recomendamos que cree un anillo de actualización y cambie los valores predeterminados siguientes:

En Azure Portal:

1. Vaya a **Microsoft Intune** > **Actualizaciones de software** > **Anillos de actualización de Windows 10**.
1. Especifique:

   * Nombre: **Actualizaciones de la estación de trabajo administrada por Azure**.
   * Canal de mantenimiento: **Windows Insider: Rápida**.
   * Período de aplazamiento de actualizaciones de calidad (días): **3**.
   * Período de aplazamiento de actualizaciones de características (días): **3**.
   * Comportamiento de actualización automática: **Instalar y reiniciar automáticamente sin control del usuario final**.
   * Impedir al usuario pausar las actualizaciones de Windows: **Bloquear**.
   * Exigir la aprobación del usuario para reiniciar fuera de las horas de trabajo: **Requerido**.
   * Permitir al usuario que reinicie (reinicio establecido): **Requerido**.
   * Hacer la transición de los usuarios al reinicio comprometido después de un reinicio automático (días): **3**.
   * Posponer el recordatorio de reinicio establecido (días): **3**.
   * Establecer fecha límite para reinicios pendientes (días): **3**.

1. Seleccione **Crear**.
1. En la pestaña **Asignaciones**, agregue el grupo **Estaciones de trabajo seguras**.

Para obtener más información sobre las directivas de Windows Update, vea [Directiva CSP: actualización](/windows/client-management/mdm/policy-csp-update).

### <a name="windows-defender-atp-intune-integration"></a>Integración de Intune con ATP de Windows Defender

ATP de Windows Defender y Microsoft Intune funcionan conjuntamente para ayudar a impedir las infracciones de seguridad. Además, pueden limitar el impacto de las infracciones. Las capacidades de ATP ofrecen un servicio de detección de amenazas en tiempo real y permiten realizar una auditoría y registro amplios de los dispositivos de punto de conexión.

Para configurar la integración de ATP de Windows Defender e Intune, vaya a Azure Portal.

1. Vaya a **Microsoft Intune** > **Conformidad de dispositivos** > **ATP de Windows Defender**.
1. En el paso 1 de la **configuración de ATP de Windows Defender**, seleccione **Conectar ATP de Windows Defender a Microsoft Intune en el Centro de seguridad de Windows Defender**.
1. En el Centro de seguridad de Windows Defender:

   1. Seleccione **Configuración** > **Características avanzadas**.
   1. Para **Conexión de Microsoft Intune**, elija **Activada**.
   1. Seleccione **Guardar preferencias**.

1. Después de establecer una conexión, vuelva a Intune y seleccione **Actualizar** en la parte superior.
1. Establezca **Conectar dispositivos Windows de la versión 10.0.15063 y posteriores a ATP de Windows Defender** en **Activado**.
1. Seleccione **Guardar**.

Para obtener más información, consulte [Protección contra amenazas avanzada de Windows Defender](/Windows/security/threat-protection/windows-defender-atp/windows-defender-advanced-threat-protection).

### <a name="finish-workstation-profile-hardening"></a>Finalización de la protección del perfil de la estación de trabajo

Para completar correctamente la protección de la solución, descargue y ejecute el script adecuado. Encontrará aquí los vínculos de descarga del **nivel de perfil** deseado:

| Perfil | Ubicación de descarga | Nombre de archivo |
| --- | --- | --- |
| Seguridad baja | N/D | N/D |
| Seguridad mejorada | https://aka.ms/securedworkstationgit | Enhanced-Workstation-Windows10-(1809).ps1 |
| Seguridad alta | https://aka.ms/securedworkstationgit | HighSecurityWorkstation-Windows10-(1809).ps1 |
| Especializada | https://github.com/pelarsen/IntunePowerShellAutomation | DeviceConfiguration_NCSC - Windows10 (1803) SecurityBaseline.ps1 |
| Cumplimiento especializado* | https://aka.ms/securedworkstationgit | DeviceCompliance_NCSC-Windows10(1803).ps1 |
| Protegido | https://aka.ms/securedworkstationgit | Secure-Workstation-Windows10-(1809)-SecurityBaseline.ps1 |

\* El cumplimiento especializado es un script que aplica la configuración especializada que se proporciona en NCSC Windows10 SecurityBaseline.

Cuando el script se ejecute correctamente, podrá actualizar los perfiles y las directivas en Intune. Los scripts para los perfiles Mejorado y Protegido crean directivas y perfiles automáticamente, pero se debe asignar la directiva al grupo de dispositivos **Estaciones de trabajo seguras**.

* Aquí encontrará los perfiles de configuración de dispositivos de Intune creados por los scripts: **Azure Portal** > **Microsoft Intune** > **Configuración del dispositivo** > **Perfiles**.
* Aquí encontrará las directivas de cumplimiento de dispositivos de Intune creadas por los scripts: **Azure Portal** > **Microsoft Intune** > **Configuración del dispositivo** > **Directivas**.

Para revisar los cambios realizados por los scripts, puede exportar los perfiles. De este modo, se podrá determinar si es necesaria una protección adicional, de conformidad con la [documentación de SECCON](/windows/security/threat-protection/windows-security-configuration-framework/windows-security-configuration-framework).

Ejecute el script de exportación de datos de Intune `DeviceConfiguration_Export.ps1` desde el [repositorio de GitHub DeviceConfiguration](https://github.com/microsoftgraph/powershell-intune-samples/tree/master/DeviceConfiguration) para exportar todos los perfiles de Intune actuales.

## <a name="additional-configurations-and-hardening-to-consider"></a>Protección y configuraciones adicionales que se deben tener en cuenta

Con las instrucciones indicadas aquí, ha implementado una estación de trabajo segura. Aun así, también debería considerar la posibilidad de incluir otros controles. Por ejemplo:

* Limitar el acceso a exploradores alternativos
* Permitir el tráfico HTTP saliente
* Bloquear sitios web específicos
* Establecer permisos para ejecutar scripts de PowerShell personalizados

### <a name="set-rules-in-the-firewall-configuration-service-provider-csp"></a>Establecimiento de reglas en el proveedor de servicios de configuración (CSP) del firewall

Puede realizar cambios adicionales en la administración de las reglas de entrada y de salida según sea necesario para los puntos de conexión permitidos y bloqueados. Al mismo tiempo que aumenta la protección de la estación de trabajo segura, puede reducir la restricción que deniega todo el tráfico entrante y saliente. Existe la posibilidad de agregar sitios de salida permitidos para incluir sitios web comunes y de confianza. Para obtener más información, consulte [CSP de firewall](/Windows/client-management/mdm/firewall-csp).

La administración restrictiva de tráfico de dirección URL incluye lo siguiente:

* Denegación de todo el tráfico entrante: establezca el script de perfil Protegido de la estación de trabajo.
* Denegación de todo el tráfico excepto los servicios de Azure y Microsoft seleccionados, incluido Azure Cloud Shell y la capacidad de permitir el restablecimiento de contraseña de Azure.

```
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings]
"ProxyEnable"=dword:00000001
"ProxyServer"="127.0.0.1:80"
"ProxyOverride"="*.azure.com;*.azure.net;*.microsoft.com;*.windowsupdate.com;*.microsoftonline.com;*.microsoftonline.cn;*.windows.net;*.windowsazure.com;*.windowsazure.cn;*.azure.cn;*.loganalytics.io;*.applicationinsights.io;*.vsassets.io;*.azure-automation.net;*.visualstudio.com,portal.office.com;*.aspnetcdn.com;*.sharepointonline.com;*.msecnd.net;*.msocdn.com;*.webtrends.com"
"AutoDetect"=dword:00000000
```

Si se busca proporcionar más granularidad a las reglas de bloqueo, se puede hacer referencia al proyecto Spamhaus que mantiene [la Lista de bloques de dominio (DBL)](https://www.spamhaus.org/dbl/): un buen recurso para usar como un conjunto avanzado de reglas para la implementación de sitios de bloqueo.

### <a name="manage-local-applications"></a>Administración de aplicaciones locales

La estación de trabajo segura pasa a un estado totalmente protegido cuando se quitan las aplicaciones locales, incluidas las aplicaciones de productividad. En este caso, agregará Chrome como explorador predeterminado y usará Intune para limitar la capacidad de un usuario de modificar el explorador y sus complementos.

#### <a name="deploy-applications-using-intune"></a>Implementación de aplicaciones con Intune

En algunas situaciones, se necesitan aplicaciones como el explorador Google Chrome en la estación de trabajo protegida. El ejemplo siguiente contiene instrucciones para instalar Chrome en dispositivos del grupo de seguridad **Estaciones de trabajo seguras**.

1. Descargue el instalador sin conexión [Paquete de Chrome para Windows de 64 bits](https://cloud.google.com/chrome-enterprise/browser/download/).
1. Extraiga los archivos y tome nota de la ubicación del archivo `GoogleChromeStandaloneEnterprise64.msi`.
1. En **Azure Portal**, vaya a **Microsoft Intune** > **Aplicaciones cliente** > **Aplicaciones** > **Agregar**.
1. En **Tipo de aplicación**, seleccione **Línea de negocio**.
1. En **Archivo del paquete de aplicaciones**, seleccione el archivo `GoogleChromeStandaloneEnterprise64.msi` en la ubicación donde se ha extraído y seleccione **Aceptar**.
1. En **Información de la aplicación**, proporcione una descripción y un publicador. Seleccione **Aceptar**.
1. Seleccione **Agregar**.
1. En la pestaña **Asignaciones**, seleccione **Disponible para dispositivos inscritos** en **Tipo de asignación**.
1. En **Grupos incluidos**, agregue el grupo **Estaciones de trabajo seguras**.
1. Seleccione **Aceptar** y después **Guardar**.

Para obtener más información sobre cómo configurar las opciones de Chrome, vea [Administración del explorador Chrome con Microsoft Intune](https://support.google.com/chrome/a/answer/9102677).

#### <a name="configuring-the-company-portal-for-custom-apps"></a>Configuración del portal de empresa para aplicaciones personalizadas

En modo seguro, la instalación de aplicaciones está restringida al portal de empresa de Intune. Aun así, es necesario acceder a Microsoft Store para instalar el portal. En la solución protegida, puede hacer que el portal de empresa esté disponible para todos los dispositivos mediante un modo sin conexión.

Una copia administrada por Intune del [Portal de empresa](/Intune/store-apps-company-portal-app) le proporciona acceso a petición a otras herramientas que puede insertar en los usuarios de las estaciones de trabajo protegidas.

Es posible que necesite instalar aplicaciones de Windows 32 bits u otras aplicaciones cuya implementación requieran una preparación especial. En tales casos, la [Herramienta de preparación de contenido de Win32 de Microsoft](https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool) puede proporcionar un archivo de formato `.intunewin` listo para usar para la instalación.

### <a name="conditional-access-only-allowing-secured-workstation-ability-to-access-azure-portal"></a>Acceso condicional solo permite a la estación de trabajo protegida tener acceso a Azure Portal

Azure AD ofrece la capacidad de administrar y restringir quién y qué puede tener acceso al portal de administración en la nube de Azure. Al habilitar [Acceso condicional](../conditional-access/overview.md) se asegurará de que únicamente su estación de trabajo segura puede administrar o cambiar los recursos. Es fundamental que, al implementar esta característica, tenga en cuenta si la función de [acceso de emergencia](../users-groups-roles/directory-emergency-access.md) puede o debe usarse únicamente para casos extremos y la cuenta administrarse a través de la directiva.

> [!NOTE]
> Tendrá que crear un grupo de usuarios e incluir el usuario de emergencia que puede omitir la directiva de Acceso condicional. En nuestro ejemplo tenemos un grupo de seguridad denominado **BreakGlass de emergencia**.

1. Vaya a **Azure Portal** > **Microsoft Intune** > **Acceso condicional: directivas** > **Nueva directiva**.
1. Indique un **Nombre** para la directiva.
1. Seleccione **Usuarios y grupos** > **Seleccionar usuarios y grupos**. 
1. Seleccione **Incluir** > **Roles del directorio** Elija los roles > Administrador global, Administrador de roles con privilegios, Administrador de autenticación con privilegios, Administrador de seguridad, Administrador de cumplimiento, Administrador de acceso condicional, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Administrador de servicios de Intune
1. Seleccione **Excluir** > Elija **Usuarios y grupos** > Seleccione **Seleccionar usuarios excluidos** > Seleccione el grupo **BreakGlass de emergencia**.
1. Seleccione **Aplicaciones o acciones en la nube** > Seleccione **Todas las aplicaciones en la nube**.
1. Seleccione **Condiciones** > Seleccione **Plataformas de dispositivos** > Elija la configuración **Sí** > Seleccione **Seleccionar plataformas de dispositivos** Elija **Windows**.
1. Seleccione **Controles de acceso** > Seleccione **Conceder acceso** **Sí** > Elija **Requerir que el dispositivo esté marcado como compatible**. 
1. Seleccione **Habilitar directiva** > **Activar**
 
Este conjunto de directivas garantizará que los administradores deban usar un dispositivo Windows compatible, que se establece mediante Intune y WDATP. 

Las organizaciones también deben considerar la posibilidad de bloquear los protocolos de autenticación heredados en sus entornos. Hay varias maneras de realizar esta tarea. Para obtener más información sobre el bloqueo de protocolos de autenticación heredados, vea el artículo [Cómo: Bloquear la autenticación heredada en Azure AD con Acceso condicional](../conditional-access/block-legacy-authentication.md).

### <a name="use-powershell-to-create-custom-settings"></a>Uso de PowerShell para crear configuraciones personalizadas

También puede usar PowerShell para ampliar las funcionalidades de administración del host. Este script de ejemplo configura un fondo predeterminado en el host. Se trata de una funcionalidad que también está disponible en Azure Portal y los perfiles. El script de ejemplo solo sirve para ilustrar la funcionalidad de PowerShell.

Es posible que deba configurar algunos controles y valores personalizados en las estaciones de trabajo seguras. En este ejemplo se cambia el fondo de la estación de trabajo mediante el uso de la funcionalidad de PowerShell para identificar fácilmente el dispositivo como una estación de trabajo segura y lista para usar.

El script [SetDesktopBackground.ps1](https://gallery.technet.microsoft.com/scriptcenter/Set-Desktop-Image-using-5430c9fb/) del Centro de scripting de Microsoft permite que Windows cargue esta [imagen de fondo genérica y gratuita](https://i.imgur.com/OAJ28zO.png) al iniciarse.

1. Descargue el script en un dispositivo local.
1. Actualice customerXXXX y la ubicación de la descarga de la imagen de fondo. En nuestro ejemplo, reemplazamos customerXXXX por los fondos.
1. Vaya a **Azure Portal** > **Microsoft Intune** > **Configuración del dispositivo** > **Scripts de PowerShell** > **Agregar**.
1. Proporcione un **Nombre** para el script y especifique la **Ubicación del script**.
1. Seleccione **Configurar**.
   1. Establezca **Ejecutar este script con las credenciales de inicio de sesión** en **Sí**.
   1. Seleccione **Aceptar**.
1. Seleccione **Crear**.
1. Haga clic en **Asignaciones** > **Seleccionar grupos**.
   1. Agregue el grupo de seguridad **Estaciones de trabajo seguras**.
   1. Seleccione **Guardar**.

## <a name="enroll-and-validate-your-first-device"></a>Inscripción y validación del primer dispositivo

1. Para inscribir el dispositivo, necesita la información siguiente:
   * **Número de serie**: se encuentra en el chasis del dispositivo.
   * **Id. de producto de Windows**: se encuentra en **Sistema** > **Acerca de** en el menú de configuración de Windows.
   * Puede ejecutar [Get-WindowsAutoPilotInfo](https://aka.ms/Autopilotshell) para obtener un archivo hash CSV con toda la información necesaria para la inscripción del dispositivo.
   
     Ejecute `Get-WindowsAutoPilotInfo – outputfile device1.csv` para generar la información como un archivo CSV que se puede importar en Intune.

     > [!NOTE]
     > El script requiere privilegios elevados y se ejecuta como si fuera un script remoto firmado. El comando `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned` permite que el script se ejecute correctamente.

   * Puede recopilar esta información si inicia sesión en Windows 10 versión 1809 o en un dispositivo con una versión superior. El distribuidor de hardware también puede proporcionarle esta información.
1. En **Azure Portal**, vaya a **Microsoft Intune** > **Inscripción de dispositivos** > **Inscripción de Windows** > **Dispositivos: Administrar dispositivos de Windows Autopilot**.
1. Seleccione **Importar** y elija el archivo CSV.
1. Agregue el dispositivo al grupo de seguridad **Estaciones de trabajo seguras**.
1. En el dispositivo Windows 10 que quiere configurar, vaya a **Configuración de Windows** > **Actualización y seguridad** > **Recuperación**.
   1. Seleccione **Primeros pasos** en **Restablecer este PC**.
   1. Siga las indicaciones para restablecer el dispositivo y volver a configurarlo con las directivas de cumplimiento y el perfil configurados.

Una vez que haya configurado el dispositivo, realice una revisión y compruebe la configuración. Confirme que el primer dispositivo está configurado correctamente antes de continuar con la implementación.

## <a name="assign-devices"></a>Asignación de dispositivos

Para asignar usuarios y dispositivos, deberá asignar los [perfiles seleccionados](/intune/device-profile-assign) al grupo de seguridad. Todos los usuarios nuevos que requieran permisos para el servicio deberán agregarse también al grupo de seguridad.

## <a name="using-sentinel-and-windows-defender-atp-to-monitor-and-respond-to-security-incidents"></a>Uso de Sentinel y ATP de Windows Defender para supervisar y responder a incidentes de seguridad

La supervisión de la implementación de la estación de trabajo segura puede realizarse mediante la habilitación de [Sentinel] y el uso de la [Administración de amenazas y vulnerabilidades](/windows/security/threat-protection/microsoft-defender-atp/next-gen-threat-and-vuln-mgt). La guía no proporcionará una búsqueda exhaustiva de amenazas, sino que proporcionará esfuerzos buenos de sentido común para supervisar y responder a posibles incidentes de seguridad.

Se usará **Azure Sentinel** para lo siguiente: 

* Recopilar datos de Intune, Azure Portal y Azure AD para la supervisión de usuarios y dispositivos.
* Ayudar a investigar la actividad sospechosa de configuración del usuario y dispositivo.
* Impulsar la capacidad de explorar las investigaciones de seguridad mediante WDATP.

La supervisión de Sentinel requiere que se configuren los conectores a los orígenes de datos, como Azure AD.

1. En **Azure Portal**, vaya a **Azure Sentinel (versión preliminar)** > Seleccione **Agregar**.
1. En **Elegir un área de trabajo para agregarla a Azure Sentinel**, seleccione **Crear una nueva área de trabajo**.
1. Especifique:
   * **Área de trabajo de Log Analytics**: "supervisión segura de la estación de trabajo".
   * **Suscripción**: seleccione la suscripción activa.
   * **Grupo de recursos**: seleccione > **Crear nuevo** > Grupo de recursos de la estación de trabajo segura > **Aceptar**.
   * **Ubicación**: seleccione la ubicación que mejor se adapte geográficamente a su implementación.
   * **Nivel de precios**: seleccione **Por GB (2018)** .
1. Seleccione **Aceptar**.

Después, conectaremos los orígenes de datos de estaciones de trabajo seguras disponibles a la supervisión.

1. En **Azure Portal**, vaya a **Área de trabajo de Azure Sentinel** > Seleccione el área de trabajo de **Supervisión de la estación de trabajo segura**.
1. Seleccione **Conectores de datos**.
1. Elija **Azure Active Directory** > Abrir página de conector > Después de revisar los requisitos previos. Continúe con la configuración y seleccione **Conectar** para los registros de inicio de sesión de Azure AD y para los Registros de auditoría de Azure AD.
1. Elija **Actividad de Azure** > Abrir página de conector > Después de revisar los requisitos previos. Continúe con la configuración de los registros de actividad de Azure > Seleccione su suscripción > Seleccione **Conectar**.

A medida que los datos se recopilan en Sentinel, podrá observar la actividad seleccionando **Azure Portal**. Vaya a **Información general de Azure Sentinel**. 

Usaremos **ATP de Windows Defender (WDATP)** para lo siguiente:

* Observar y supervisar continuamente vulnerabilidades y errores de configuración.
* Usar WDATP para clasificar por orden de prioridad las amenazas dinámicas de libre circulación.
* Impulsar la correlación de vulnerabilidades con alertas de Detección y respuesta de puntos de conexión (EDR).
* Usar el panel para identificar vulnerabilidades de nivel de máquina durante las investigaciones.
* Extraer correcciones a Intune.

Configure el [Panel de ATP de Defender](https://securitycenter.windows.com/machines). Uso de la guía en [Información general del panel de Administración de amenazas y vulnerabilidades](/windows/security/threat-protection/microsoft-defender-atp/tvm-dashboard-insights).

## <a name="monitoring-application-activity-using-microsoft-monitoring-agent-mma"></a>Supervisión de la actividad de la aplicación mediante Microsoft Monitoring Agent (MMA)
A partir de la estación de trabajo especializada, el bloqueador de aplicaciones está habilitado para la supervisión de la actividad de la aplicación en una estación de trabajo. Para que la supervisión se integre en el área de trabajo de Log Analytics, se debe seguir una configuración y un agente de MMA. 

> [!NOTE]
> El perfil de estación de trabajo especializada contiene las directivas de supervisión de AppLocker. La implementación de las directivas es necesaria para la supervisión de la actividad de la aplicación en un cliente.

Implementación del agente de MMA con el script de PowerShell de Intune

1. Descargue el programa de instalación [script en un dispositivo local](https://aka.ms/securedworkstationgit).
1. Actualice los parámetros **$WorkSpaceID** y **$WorkSpaceKey**.
1. Vaya a **Azure Portal** > **Microsoft Intune** > **Configuración del dispositivo** > **Scripts de PowerShell** > **Agregar**.
1. Proporcione un **Nombre** para el script y especifique la **Ubicación del script**.
1. Seleccione **Configurar**.
   1. Establezca **Ejecutar este script con las credenciales de inicio de sesión** en **Sí**.
   1. Seleccione **Aceptar**.
1. Seleccione **Crear**.
1. Haga clic en **Asignaciones** > **Seleccionar grupos**.
   1. Agregue el grupo de seguridad **Estaciones de trabajo seguras**.
   1. Seleccione **Guardar**.

Luego debe configurar Log Analytics para recibir los registros nuevos.
1. En **Azure Portal**, vaya a **Área de trabajo de Log Analytics** > Seleccione: "Supervisión de la estación de trabajo segura".
1. Seleccione **Configuración avanzada** > **Datos** > **Registros de eventos de Windows**.
1. En **Recopilar eventos de los registros de eventos siguientes**, 
1. Especifique:
   * "Microsoft-Windows-AppLocker/EXE y DLL" > Anule la selección de **Informativo**.
   * "Microsoft-Windows-AppLocker/MSI y Script" > Anule la selección de **Informativo**.
   * "Microsoft-Windows-AppLocker/Implementación de la aplicación empaquetada" > Anule la selección de **Informativo**.
   * "Microsoft-Windows-AppLocker/Ejecución de la aplicación empaquetada" > Anule la selección de **Informativo**.
1. Seleccione **Guardar**.

El registro de aplicaciones estará disponible en el área de trabajo seleccionada de Log Analytics.

## <a name="monitoring"></a>Supervisión

* Obtención de información sobre cómo [Detectar amenazas con Azure Sentinel](../../sentinel/tutorial-detect-threats-built-in.md)
* [Investigación de incidentes con Azure Sentinel](../../sentinel/tutorial-investigate-cases.md)
* [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](../../sentinel/tutorial-respond-threats-playbook.md)
* Descripción de la revisión de la [Puntuación de exposición](/windows/security/threat-protection/microsoft-defender-atp/tvm-exposure-score)
* Revisión de las [Recomendaciones de seguridad](/windows/security/threat-protection/microsoft-defender-atp/tvm-security-recommendation)
* Administración de las [Correcciones](/windows/security/threat-protection/microsoft-defender-atp/tvm-remediation) de seguridad
* Administración de [Detección y respuesta de puntos de conexión](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)
* Supervisión de los perfiles con la [Supervisión de perfiles de Intune](/intune/device-profile-monitor)

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [Microsoft Intune](/intune/index).
* Descripción de [Azure AD](../index.yml).
* Trabaje con [Advanced Threat Protection de Microsoft Defender](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection).
* Descubra [Azure Sentinel](../../sentinel/index.yml).