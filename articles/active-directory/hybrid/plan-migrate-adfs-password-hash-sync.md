---
title: 'Azure AD Connect: Migración de Federación a PHS para Azure AD | Microsoft Docs'
description: Este artículo contiene información acerca de cómo mover un entorno de identidades híbrido a la sincronización de hash de contraseña.
services: active-directory
author: billmath
manager: daveba
ms.reviewer: martincoetzer
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/29/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5e463644820866607eee1deac115dc1381f463a7
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276565"
---
# <a name="migrate-from-federation-to-password-hash-synchronization-for-azure-active-directory"></a>Migración de la federación a la sincronización de hash de contraseña para Azure Active Directory

En este artículo se describe cómo mover los dominios de la organización de Servicios de federación de Active Directory (AD FS) a la sincronización de hash de contraseña.

> [!NOTE]
> Cambiar el método de autenticación requiere planificación, pruebas y un posible tiempo de inactividad. El [lanzamiento preconfigurado](how-to-connect-staged-rollout.md) proporciona una forma alternativa de probar y migrar gradualmente la autenticación de la federación a la nube mediante la sincronización de hash de contraseña.
>
> Si planea usar el lanzamiento por fases, recuerde desactivar las características del lanzamiento por fases una vez que haya terminado la migración.  Para obtener más información, vea [Migración a la autenticación en la nube mediante un lanzamiento por fases](how-to-connect-staged-rollout.md)


## <a name="prerequisites-for-migrating-to-password-hash-synchronization"></a>Requisitos previos para migrar a la sincronización de hash de contraseña

Se deben cumplir los siguientes requisitos previos para migrar del uso de AD FS al uso de la sincronización de hash de contraseña.


### <a name="update-azure-ad-connect"></a>Actualización de Azure AD Connect

Para realizar correctamente los pasos necesarios para migrar a la sincronización de hash de contraseña, es preciso tener, como mínimo, [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) 1.1.819.0. Esta versión contiene cambios importantes en la forma en que se realiza la conversión del inicio de sesión y reduce el tiempo total para migrar de la federación a la autenticación en la nube, que pasa de durar potencialmente horas a minutos.


> [!IMPORTANT]
> Es posible que lea en documentación, herramientas y blogs obsoletos que se requiere la conversión de los usuarios al convertir dominios de identidad federada a identidad administrada. Pues bien, la *conversión de los usuarios* ya no es necesaria. Microsoft está trabajando para actualizar la documentación y las herramientas de forma que reflejen este cambio.

Para actualizar Azure AD Connect, realice los pasos que se describen en [Azure AD Connect: actualización a la versión más reciente](./how-to-upgrade-previous-version.md).

### <a name="password-hash-synchronization-required-permissions"></a>Permisos necesarios para la sincronización del hash de contraseñas

Puede configurar Azure AD Connect mediante la configuración rápida o una instalación personalizada. Si utilizó la opción de instalación personalizada, es posible que no encuentre los [permisos necesarios](./reference-connect-accounts-permissions.md) para la sincronización de hash de contraseña.

La cuenta del servicio Active Directory Domain Services de Azure AD Connect necesita los siguientes permisos para sincronizar los hashes de las contraseñas:

* Replicación de cambios de directorio
* Replicación de todos los cambios de directorio

Este es un buen momento para comprobar que estos permisos están en vigor en todos los dominios del bosque.

### <a name="plan-the-migration-method"></a>Planeamiento del método de migración

Puede elegir entre dos métodos para migrar de la administración de identidades federadas a la sincronización de hash de contraseña y el inicio de sesión único de conexión directa. El método que utilice dependerá de la forma en que la instancia de AD FS se haya configurado originalmente.

* **Azure AD Connect**. Si originalmente configuró AD FS con Azure AD Connect, *debe* cambiar a la sincronización de hash de contraseña mediante el Asistente de Azure AD Connect.

   ‎Azure AD Connect ejecuta automáticamente el cmdlet **Set-MsolDomainAuthentication** al cambiar el método de inicio de sesión de usuario. Azure AD Connect anula automáticamente la federación de todos los dominios federados comprobados en el inquilino de Azure AD.

   > [!NOTE]
   > En la actualidad, si utilizó originalmente Azure AD Connect para configurar AD FS, no puede evitar que se anule la federación de todos los dominios del inquilino cuando se cambia el inicio de sesión del usuario a la sincronización de hash de contraseña. ‎
* **Azure AD Connect con PowerShell**. Puede usar este método solo si no configuró originalmente AD FS con Azure AD Connect. Con esta opción, todavía debe cambiar el método de inicio de sesión de usuario mediante el asistente de Azure AD Connect. La diferencia principal con esta opción es que el Asistente no ejecuta automáticamente el cmdlet **Set-MsolDomainAuthentication**. Con esta opción, tiene control total sobre qué dominios se convierten y en qué orden.

Para entender qué método debe usar, siga los pasos de las próximas secciones.

#### <a name="verify-current-user-sign-in-settings"></a>Comprobación de la configuración del inicio de sesión del usuario actual

Para comprobar la configuración del inicio de sesión del usuario actual:

1. Inicie sesión en el [portal de Azure AD](https://aad.portal.azure.com/) con una cuenta de administrador global.
2. En la sección **Inicio de sesión de usuario**, compruebe la configuración siguiente:
   * **Federación** está establecido en **Habilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Deshabilitado**.
   * **Autenticación de paso a través** está establecido en **Deshabilitado**.

   ![Captura de pantalla de la configuración de la sección de inicio de sesión de usuario de Azure AD Connect](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image1.png)

#### <a name="verify-the-azure-ad-connect-configuration"></a>Comprobación de la configuración de Azure AD Connect

1. En el servidor de Azure AD Connect, abra Azure AD Connect. Seleccione **Configurar**.
2. En la página **Tareas adicionales**, seleccione **Ver configuración actual** y, después, seleccione **Siguiente**.<br />

   ![Captura de pantalla de la opción Ver configuración actual seleccionada en la página Tareas adicionales](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image2.png)<br />
3. En la pantalla **Revisar su solución**, tome nota del estado de **Sincronización de hash de contraseña**.<br /> 

   * Si **Sincronización de hash de contraseña** está establecido en **Deshabilitado**, complete los pasos de este artículo para habilitarlo.
   * Si **Sincronización de hash de contraseña** está establecido en **Habilitado**, puede omitir la sección **Paso 1: Habilitación de la sincronización de hash de contraseñas** de este artículo.
4. En la página **Revisar su solución**, desplácese hasta **Servicios de federación de Active Directory (AD FS)** .<br />

   * ‎Si la configuración de AD FS aparece en esta sección, puede suponer con seguridad que AD FS se configuró originalmente con Azure AD Connect. Puede convertir los dominios de identidad federada a identidad administrada mediante la opción **Cambiar inicio de sesión de usuario** de Azure AD Connect. El proceso se detalla en la sección **Opción A: Cambio de la federación a la sincronización de hash de contraseña mediante Azure AD Connect**.
   * Si AD FS no aparece en la configuración actual, debe convertir manualmente los dominios de identidad federada a identidad administrada mediante PowerShell. Para obtener más información sobre este proceso, consulte la sección **Opción B: Cambio de la federación a la sincronización de hash de contraseña mediante Azure AD Connect y PowerShell**.

### <a name="document-current-federation-settings"></a>Documentación de la configuración actual de la federación

Para buscar la configuración actual de la federación, ejecute el cmdlet **Get-MsolDomainFederationSettings**.

``` PowerShell
Get-MsolDomainFederationSettings -DomainName YourDomain.extention | fl *
```

Ejemplo:

``` PowerShell
Get-MsolDomainFederationSettings -DomainName Contoso.com | fl *
```

Compruebe los valores de configuración que podrían haberse personalizado para su documentación de diseño e implementación de federación. En concreto, busque las personalizaciones en **PreferredAuthenticationProtocol**, **SupportsMfa** y **PromptLoginBehavior**.

Para obtener más información, consulte estos artículos:

* [Compatibilidad con el parámetro prompt=login de AD FS](/windows-server/identity/ad-fs/operations/ad-fs-prompt-login)
* [Set-MsolDomainAuthentication](/powershell/module/msonline/set-msoldomainauthentication?view=azureadps-1.0)

> [!NOTE]
> Si el valor **SupportsMfa** está establecido en **True**, está usando una solución de autenticación multifactor local para insertar un segundo factor en el flujo de autenticación del usuario. Esta configuración ya no funciona para los escenarios de autenticación de Azure AD después de convertir este dominio de la autenticación federada a la administrada. Después de deshabilitar la federación, interrumpe la relación con la federación en el entorno local y esto incluye los adaptadores MFA en el entorno local. 
>
> Use en su lugar el servicio en la nube de Azure Multi-factor Authentication para realizar la misma función. Valore detenidamente los requisitos de la autenticación multifactor antes de continuar. Antes de convertir los dominios, asegúrese de que sabe cómo usar Azure Multi-factor Authentication, las implicaciones en cuanto a licencias y el proceso de registro del usuario.

#### <a name="back-up-federation-settings"></a>Copia de seguridad de la configuración de federación

Aunque durante los procesos descritos en este artículo no se realizan cambios en otros usuarios de confianza de la granja de servidores de AD FS, es aconsejable que disponga de una copia de seguridad válida actual de la granja de servidores de AD FS desde la que se pueda restaurar. Puede crear una copia de seguridad válida actual mediante la [herramienta de restauración de AD FS rápida](/windows-server/identity/ad-fs/operations/ad-fs-rapid-restore-tool) de Microsoft, que es gratuita. Puede usar la herramienta para hacer una copia de seguridad de AD FS y restaurar una granja existente o crear otra.

Si decide no utilizar dicha herramienta, como mínimo, debe exportar la relación de confianza para usuario autenticado de la Plataforma de identidad de Microsoft Office 365 y todas las reglas de notificación personalizadas asociadas que haya agregado. Puede exportar la relación de confianza y las reglas de notificación asociadas mediante el siguiente ejemplo de PowerShell:

``` PowerShell
(Get-AdfsRelyingPartyTrust -Name "Microsoft Office 365 Identity Platform") | Export-CliXML "C:\temp\O365-RelyingPartyTrust.xml"
```

## <a name="deployment-considerations-and-using-ad-fs"></a>Consideraciones sobre la implementación y uso de AD FS

En esta sección se describen las consideraciones de implementación y los detalles sobre el uso de AD FS.

### <a name="current-ad-fs-use"></a>Uso actual de AD FS

Antes de realizar la conversión de identidad federada a identidad administrada, examine detenidamente cómo se usa actualmente AD FS para Azure AD u Office 365 y otras aplicaciones (relaciones de confianza para usuario autenticado). En concreto, tenga en cuenta los escenarios que se describen en la tabla siguiente:

| Si | Entonces |
|-|-|
| Tiene previsto seguir usando AD FS con otras aplicaciones (que no sean Azure AD y Office 365). | Después de convertir los dominios, deberá usar AD FS y Azure AD. Tenga en cuenta la experiencia del usuario. En algunos escenarios, es posible que los usuarios tengan que autenticarse dos veces: una vez en Azure AD (donde obtendrán acceso de inicio de sesión único para otras aplicaciones, como Office 365) y otra para todas las aplicaciones que aún están enlazadas a AD FS como una relación de confianza para usuario autenticado. |
| La instancia de AD FS está muy personalizada y depende de valores de configuración concretos del archivo onload.js (por ejemplo, ha cambiado la forma en que se inicia sesión para que los usuarios solo especifiquen un formato **SamAccountName** para su nombre de usuario, en lugar de un nombre principal de usuario, o su organización ha personalizado con marca la experiencia de inicio de sesión). El archivo onload.js no se puede duplicar en Azure AD. | Antes de continuar, debe comprobar que Azure AD puede cumplir los requisitos de personalización actuales. Para más información e instrucciones, consulte las secciones sobre personalización de marca de AD FS y personalización de AD FS.|
| Usará AD FS para bloquear las versiones anteriores de clientes de autenticación.| Considere la posibilidad de reemplazar los controles de AD FS que bloquean las versiones anteriores de clientes de autenticación mediante una combinación de [controles de acceso condicionales](../conditional-access/concept-conditional-access-conditions.md) y [reglas de acceso de cliente de Exchange Online](https://aka.ms/EXOCAR). |
| Necesita que los usuarios realicen la autenticación multifactor en una solución de servidor de autenticación multifactor local cuando los usuarios se autentican en AD FS.| En un dominio de identidad administrada, no puede insertar un desafío de autenticación multifactor a través de la solución de autenticación multifactor local en el flujo de autenticación. Sin embargo, puede usar el servicio Azure Multi-factor Authentication para la autenticación multifactor después de convertir el dominio.<br /><br /> Si los usuarios no usan actualmente Azure Multi-factor Authentication, es necesario un paso de registro puntual del usuario. Debe prepararse para el registro planeado y comunicárselo a los usuarios. |
| Actualmente usa directivas de control de acceso (reglas de AuthZ) en AD FS para controlar el acceso a Office 365.| Considere la posibilidad de reemplazarlas por las [directivas de acceso condicional](../conditional-access/overview.md) de Azure AD equivalentes y las [reglas de acceso de cliente de Exchange Online](https://aka.ms/EXOCAR).|

### <a name="common-ad-fs-customizations"></a>Personalizaciones de AD FS comunes

En esta sección se describen las personalizaciones de AD FS comunes.

#### <a name="insidecorporatenetwork-claim"></a>Notificación InsideCorporateNetwork

AD FS emite la notificación **InsideCorporateNetwork** si el usuario que realiza la autenticación está dentro de la red corporativa. Esta notificación se puede transmitir entonces a Azure AD. La notificación se utiliza para omitir la autenticación multifactor basada en la ubicación de red del usuario. Para obtener información sobre cómo determinar si esta funcionalidad actualmente está habilitada en AD FS, consulte [Direcciones IP de confianza para usuarios federados](../authentication/howto-mfa-adfs.md).

La notificación **InsideCorporateNetwork** dejará de estar estará disponible cuando los dominios se conviertan a la sincronización de hash de contraseña. Se pueden usar las [ubicaciones con nombre de Azure AD](../reports-monitoring/quickstart-configure-named-locations.md) para reemplazar esta funcionalidad.

Una vez que se han configurado las ubicaciones con nombre, debe actualizar todas las directivas de acceso condicional que se configuraron para incluir o excluir los valores de red **Todas las ubicaciones de confianza** o **IP de confianza de MFA** para reflejar las ubicaciones con nombre recién creadas.

Para más información sobre la condición **Ubicación** en el acceso condicional, consulte [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?](../conditional-access/location-condition.md).

#### <a name="hybrid-azure-ad-joined-devices"></a>Dispositivos unidos a Azure AD híbrido

Al unir un dispositivo a Azure AD, puede crear reglas de acceso condicional que obligan a que el acceso desde dispositivos cumpla con las normas de seguridad y cumplimiento. Además, los usuarios pueden iniciar sesión en un dispositivo mediante una cuenta profesional o educativa en lugar de una cuenta personal. Al usar dispositivos unidos a Azure AD híbrido, puede unir a Azure AD sus dispositivos unidos a un dominio de Active Directory. Es posible que un entorno federado se haya configurado para usar esta característica.

Para asegurarse de que la unión híbrida sigue funcionando en todos los dispositivos unidos al dominio una vez que los dominios se han convertido a la sincronización de hash de contraseña, para clientes de Windows 10, debe usar las opciones de dispositivo de Azure AD Connect para sincronizar las cuentas de equipos de Active Directory con Azure AD. 

En el caso de las cuentas de equipos de Windows 8 y Windows 7, la unión híbrida usa SSO de conexión directa para registrar el equipo en Azure AD. No es necesario sincronizar cuentas de equipo de Windows 8 y Windows 7 como hace para dispositivos con Windows 10. Sin embargo, debe implementar una versión actualizada del archivo workplacejoin.exe (a través de un archivo .msi) en los clientes de Windows 8 y Windows 7 para que puedan registrarse mediante SSO de conexión directa. [Descargue el archivo .msi](https://www.microsoft.com/download/details.aspx?id=53554).

Para más información, consulte [Configuración de dispositivos unidos a Azure AD híbrido](../devices/hybrid-azuread-join-plan.md).

#### <a name="branding"></a>Personalización de marca

Si su organización [personalizó las páginas de inicio de sesión de AD FS](/windows-server/identity/ad-fs/operations/ad-fs-user-sign-in-customization) para mostrar información que es más pertinente para la organización, considere la posibilidad de establecer [personalizaciones similares en la página de inicio de sesión de Azure AD](../fundamentals/customize-branding.md).

Aunque hay personalizaciones similares disponibles, cabe esperar que haya algunos cambios visuales en las páginas de inicio de sesión tras la conversión. Es posible que desee proporcionar información sobre los cambios previstos en sus comunicaciones a los usuarios.

> [!NOTE]
> La personalización de marca de empresa solo está disponible si ha adquirido las licencias Premium o Básica de Azure Active Directory, o si tiene una licencia de Office 365.

## <a name="plan-deployment-and-support"></a>Planeación de la implementación y el soporte técnico

Los pasos de esta sección le ayudarán a planear la implementación y el soporte técnico.

### <a name="plan-the-maintenance-window"></a>Planeación de la ventana de mantenimiento

Aunque el proceso de conversión de dominios es relativamente rápido, Azure AD puede seguir enviando solicitudes de autenticación a sus servidores de AD FS durante un período máximo de cuatro horas una vez finalizada la conversión del dominio. Durante este período de cuatro horas, y en función de varias memorias cachés del lado servicio, es posible que Azure AD no acepte estas autenticaciones. Los usuarios pueden recibir un error. El usuario aún puede autenticarse correctamente en AD FS, pero Azure AD ya no acepta el token emitido por el usuario porque se ha quitado la confianza de la federación.

Esta situación solo afecta a los usuarios que accedan a los servicios mediante un explorador web durante el periodo posterior a esta conversión hasta que se borre la caché del lado servicio. No se espera que los clientes heredados (Exchange ActiveSync, Outlook 2010/2013) se vean afectados porque Exchange Online mantiene una caché de sus credenciales durante un período de tiempo establecido. La caché se usa para autenticar al usuario en silencio. El usuario no tiene que volver a AD FS. Las credenciales almacenadas en el dispositivo para estos clientes se utilizan para volver a autenticarse en modo silencioso una vez que se borre esta caché. No se espera que los usuarios reciban mensajes de contraseña como resultado del proceso de conversión de dominio. 

Los clientes de autenticación moderna (Office 2016 y Office 2013, las aplicaciones iOS y Android) usan un token de actualización válido para obtener nuevos tokens de acceso para un acceso continuado a los recursos en lugar de volver a AD FS. Estos clientes son inmunes a los avisos de contraseña resultantes del proceso de conversión de dominio. Los clientes seguirán funcionando sin ninguna configuración adicional.

> [!IMPORTANT]
> No apague el entorno de AD FS ni quite la relación de confianza para usuario autenticado de Office 365 hasta que haya comprobado que todos los usuarios se pueden autenticar correctamente mediante la autenticación en la nube.

### <a name="plan-for-rollback"></a>Planeación para la reversión

Si surge algún problema importante que no se pueda resolver rápidamente, puede decidir revertir la solución a la opción de federación. Es importante planear qué hacer si la implementación no se aplica según lo previsto. Si se produce un error en la conversión del dominio o de los usuarios durante la implementación, o si necesita revertir a la federación, debe saber cómo mitigar las posibles interrupciones y reducir el efecto sobre los usuarios.

#### <a name="to-roll-back"></a>Para realizar la reversión

Para planear la reversión, consulte la documentación sobre el diseño y la implementación de la federación para ver los detalles de su implementación concreta. El proceso debe incluir estas tareas:

* Conversión de dominios administrados a dominios federados mediante el cmdlet **Convert-MSOLDomainToFederated**.
* Si fuera necesario, configurar reglas de notificaciones adicionales.

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

Una parte importante del planeamiento de la implementación y el soporte técnico es garantizar que a los usuarios finales se les informe con antelación de los próximos cambios. Los usuarios deben saber de antemano lo que podrían experimentar y qué se necesita de ellos. 

Una vez que se hayan implementado tanto la sincronización de hash de contraseña como el inicio de sesión único de conexión directa, cambia la experiencia de inicio de sesión del usuario para acceder a Office 365 y otros recursos autenticados mediante Azure AD. Los usuarios que están fuera de la red solo ven la página de inicio de sesión de Azure AD. Estos usuarios no son redirigidos a la página basada en formularios que se presenta por los servidores proxy de aplicación web de uso externo.

Incluya los siguientes elementos en su estrategia de comunicación:

* Informe a los usuarios de las funcionalidades próximas y publicadas por los medios siguientes:
   * Correo electrónico y otros canales de comunicación interna.
   * Objetos visuales, como pósteres.
   * Comunicaciones en directo, ejecutivas o de otro tipo.
* Determinación de quién va a personalizar las comunicaciones y quién va a enviarlas y cuándo.

## <a name="implement-your-solution"></a>Implementación de la solución

Ya ha planeado la solución. Ahora, ya puede implementarla. La implementación incluye los siguientes componentes:

* Habilitación de la sincronización de hash de contraseña.
* Preparación para el SSO de conexión directa.
* Cambio del método de inicio de sesión a sincronización de hash de contraseñas y habilitación del inicio de sesión único de conexión directa.

### <a name="step-1-enable-password-hash-synchronization"></a>Paso 1: Habilitación de la sincronización de hash de contraseñas

El primer paso para implementar esta solución es habilitar la sincronización de hash de contraseña en el Asistente de Azure AD Connect. La sincronización de hash de contraseña es una característica opcional que se puede habilitar en entornos que usan la federación. No surte ningún efecto en el flujo de autenticación. En este caso, Azure AD Connect iniciará la sincronización de los hashes de contraseña sin que ello afecte a los usuarios que inician sesión mediante federación.

Ese es el motivo por el que se recomienda realizar este paso como una tarea de preparación antes de cambiar el método de inicio de sesión de dominios. A continuación, tendrá tiempo suficiente para comprobar que la sincronización de hash de contraseña funciona correctamente.

Para habilitar la sincronización de hash de contraseña:

1. En el servidor de Azure AD Connect, abra el Asistente de Azure AD Connect y seleccione **Configurar**.
2. Seleccione **Personalizar las opciones de sincronización** y, después, seleccione **Siguiente**.
3. En la pantalla **Conectar a Azure AD**, indique el nombre de usuario y la contraseña de una cuenta de administrador global.
4. En la página **Conectar sus directorios**, haga clic en **Siguiente**.
5. En la pantalla **Filtrado de dominios y unidades organizativas**, seleccione **Siguiente**.
6. En la pantalla **Características opcionales**, seleccione **Sincronización de contraseña** y seleccione **Siguiente**.
 
   ![Captura de pantalla de la opción Sincronización de contraseña seleccionada en la página Características opcionales](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image6.png)<br />
7. Seleccione **Siguiente** en las páginas restantes. En la última página, seleccione **Configurar**.
8. Azure AD Connect empieza a sincronizar los hashes de contraseña en la próxima sincronización.

Una vez que se haya habilitado la sincronización de hash de contraseña, los hashes de contraseña de todos los usuarios del ámbito de sincronización de Azure AD Connect se vuelven a crear y escribir en Azure AD. En función del número de usuarios, esta operación puede durar entre varios minutos y varias horas.

En los planeamientos, debe calcular que en una hora se pueden procesar aproximadamente 20 000 usuarios.

Para validar que la sincronización de hash de contraseña funciona correctamente, complete la tarea **Solución de problemas** en el Asistente de Azure AD Connect:

1. Abra una nueva sesión de Windows PowerShell en el servidor de Azure AD Connect mediante la opción Ejecutar como administrador.
2. Ejecute `Set-ExecutionPolicy RemoteSigned` o `Set-ExecutionPolicy Unrestricted`.
3. Inicie el asistente de Azure AD Connect.
4. Vaya a la página **Tareas adicionales**, seleccione **Solucionar problemas** y haga clic en **Siguiente**.
5. En la página de **solución de problemas**, haga clic en **Iniciar** para iniciar el menú de solución de problemas en PowerShell.
6. En el menú principal, seleccione **Solución de problemas de sincronización de contraseñas**.
7. En el submenú, seleccione **Password Synchronization does not work at all** (La sincronización de hash de contraseñas no funciona en absoluto).

Para obtener información sobre la solución de problemas, consulte [Solución de problemas de sincronización de hash de contraseña con la sincronización de Azure AD Connect](./tshoot-connect-password-hash-synchronization.md).

### <a name="step-2-prepare-for-seamless-sso"></a>Paso 2: Preparación para el SSO de conexión directa

Para que los dispositivos usen SSO de conexión directa, debe agregar una dirección URL de Azure AD a la configuración de la zona de intranet de los usuarios mediante una directiva de grupo de Active Directory.

De forma predeterminada, los exploradores web calculan automáticamente la zona correcta (Internet o intranet) a partir de una dirección URL. Por ejemplo, **http:\/\/contoso/** se asigna a la zona de intranet, mientras que **http:\/\/intranet.contoso.com** se asigna a la zona de Internet (porque la dirección URL contiene un punto). Los exploradores envían vales Kerberos a un punto de conexión en la nube, como la dirección URL de Azure AD, solo si agrega explícitamente la dirección URL a la zona de intranet del explorador.

Siga los pasos para [implementar](./how-to-connect-sso-quick-start.md) los cambios necesarios en los dispositivos.

> [!IMPORTANT]
> Hacer este cambio no modifica la forma en que los usuarios inician sesión en Azure AD. Sin embargo, es importante que aplique esta configuración a todos los dispositivos antes de continuar. Los usuarios que inicien sesión en dispositivos que no hayan recibido esta configuración solo tendrán que escribir un nombre de usuario y una contraseña para iniciar sesión Azure AD.

### <a name="step-3-change-the-sign-in-method-to-password-hash-synchronization-and-enable-seamless-sso"></a>Paso 3: Cambio del método de inicio de sesión a la sincronización de hash de contraseña y habilitación de SSO de conexión directa

Tiene dos opciones para cambiar el método de inicio de sesión a sincronización de hash de contraseña y habilitación del inicio de sesión único de conexión directa.

#### <a name="option-a-switch-from-federation-to-password-hash-synchronization-by-using-azure-ad-connect"></a>Opción A: Cambio de la federación a la sincronización de hash de contraseña mediante Azure AD Connect

Use este método si ha configurado inicialmente su entorno de AD FS con Azure AD Connect. No puede usar este método si *no* configuró inicialmente su entorno de AD FS con Azure AD Connect.

En primer lugar, cambie el método de inicio de sesión:

1. En el servidor de Azure AD Connect, abra el asistente de Azure AD Connect.
2. Seleccione **Cambiar inicio de sesión de usuario** y, después, seleccione **Siguiente**. 

   ![Captura de pantalla de la opción Cambiar inicio de sesión de usuario en la página Tareas adicionales](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image7.png)<br />
3. En la pantalla **Conectar a Azure AD**, indique el nombre de usuario y la contraseña de una cuenta de administrador global.
4. En la página **Inicio de sesión de usuario**, seleccione el **botón de sincronización de hash de contraseña**. Asegúrese de activar la casilla **No convertir las cuentas de usuario**. La opción está en desuso. Seleccione **Habilitar inicio de sesión único** y, después, **Siguiente**.

   ![Captura de pantalla de la página Habilitar el inicio de sesión único](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image8.png)<br />

   > [!NOTE]
   > A partir de la versión 1.1.880.0 de Azure AD Connect, la casilla **Inicio de sesión único de conexión directa** está activa de forma predeterminada.

   > [!IMPORTANT]
   > Puede ignorar sin riesgo alguno las advertencias que indican que la conversión del usuario y la sincronización de hash de contraseña total son pasos obligatorios para realizar la conversión de federación a autenticación en la nube. Tenga en cuenta que estos pasos ya no son necesarios. Si aún ve dichas advertencias, compruebe que ejecuta la versión más reciente de Azure AD Connect y que usa la versión más reciente de esta guía. Para más información, consulte la sección [Actualización de Azure AD Connect](#update-azure-ad-connect).

5. En la página **Habilitar el inicio de sesión único**, escriba las credenciales de la cuenta del administrador del dominio y, después, seleccione **Siguiente**.

   ![Captura de pantalla de la página Habilitar el inicio de sesión único](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image9.png)<br />

   > [!NOTE]
   > Las credenciales de la cuenta de administrador de dominio son necesarias para habilitar el inicio de sesión único de conexión directa. Durante el proceso se realizan las acciones siguientes, que requieren estos permisos elevados. Las credenciales de la cuenta de administrador de dominio no se almacenan en Azure AD Connect ni en Azure AD. Las credenciales de la cuenta de administrador de dominio se usan solo para activar la característica. Cuando el proceso finaliza correctamente, estas credenciales se descartan.
   >
   > 1. Se crea una cuenta de equipo denominada AZUREADSSOACC (que representa a Azure AD) en la instancia local de Active Directory.
   > 2. La clave de descifrado de Kerberos de la cuenta de equipo se comparte de manera segura con Azure AD.
   > 3. Se crean dos nombres de entidad de seguridad de servicio (SPN) de Kerberos que representan las dos direcciones URL que se usan en el inicio de sesión de Azure AD.

6. En la página **Listo para configurar**, asegúrese de que la casilla **Inicie el proceso de sincronización cuando se complete la configuración** esté activada. A continuación, seleccione **Configurar**.

      ![Captura de pantalla de la página Listo para configurar](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image10.png)<br />

   > [!IMPORTANT]
   > En este momento, todos los dominios federados cambiarán a la autenticación administrada. La sincronización de hash de contraseña es el nuevo método de autenticación.

7. En el portal de Azure AD, seleccione **Azure Active Directory** > **Azure AD Connect**.
8. Compruebe la configuración de estos parámetros:
   * **Federación** está establecido en **Deshabilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Habilitado**.
   * **Sincronización de contraseñas** está establecido en **Habilitado**.<br /> 

   ![Captura de pantalla que muestra la configuración de la sección de inicio de sesión de usuario](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image11.png)<br />

Vaya a [Pruebas y pasos siguientes](#testing-and-next-steps).

   > [!IMPORTANT]
   > Omita la sección **Opción B: Cambio de la federación a la sincronización de hash de contraseña mediante Azure AD Connect y PowerShell**. Los pasos de esa sección no se aplican si eligió la Opción A para cambiar el método de inicio de sesión a sincronización de hash de contraseña y habilitar el inicio de sesión único de conexión directa.

#### <a name="option-b-switch-from-federation-to-password-hash-synchronization-using-azure-ad-connect-and-powershell"></a>Opción B: Cambio de la federación a la sincronización de hash de contraseña mediante Azure AD Connect y PowerShell

Use esta opción si no configuró inicialmente los dominios federados con Azure AD Connect. Como parte de este proceso, ha habilitado el inicio de sesión único de conexión directa y ha cambiado los dominios de federados a administrados.

1. En el servidor de Azure AD Connect, abra el asistente de Azure AD Connect.
2. Seleccione **Cambiar inicio de sesión de usuario** y, después, seleccione **Siguiente**.
3. En la pantalla **Conectar a Azure AD**, indique el nombre de usuario y la contraseña de una cuenta de administrador global.
4. En la página **Inicio de sesión de usuario**, seleccione el botón de **sincronización de hash de contraseña**. Seleccione **Habilitar inicio de sesión único** y, después, **Siguiente**.

   Antes de habilitar la sincronización de hash de contraseña: ![Captura de pantalla que muestra la opción No configurar en la página Inicio de sesión de usuario](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image12.png)<br />

   Después de habilitar la sincronización de hash de contraseña: ![Captura de pantalla que muestra las nuevas opciones en la página Inicio de sesión de usuario](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image13.png)<br />
   
   > [!NOTE]
   > A partir de la versión 1.1.880.0 de Azure AD Connect, la casilla **Inicio de sesión único de conexión directa** está activa de forma predeterminada.

5. En la página **Habilitar el inicio de sesión único**, escriba las credenciales de la cuenta del administrador del dominio y, después, seleccione **Siguiente**.

   > [!NOTE]
   > Las credenciales de la cuenta de administrador de dominio son necesarias para habilitar el inicio de sesión único de conexión directa. Durante el proceso se realizan las acciones siguientes, que requieren estos permisos elevados. Las credenciales de la cuenta de administrador de dominio no se almacenan en Azure AD Connect ni en Azure AD. Las credenciales de la cuenta de administrador de dominio se usan solo para activar la característica. Cuando el proceso finaliza correctamente, estas credenciales se descartan.
   >
   > 1. Se crea una cuenta de equipo denominada AZUREADSSOACC (que representa a Azure AD) en la instancia local de Active Directory.
   > 2. La clave de descifrado de Kerberos de la cuenta de equipo se comparte de manera segura con Azure AD.
   > 3. Se crean dos nombres de entidad de seguridad de servicio (SPN) de Kerberos que representan las dos direcciones URL que se usan en el inicio de sesión de Azure AD.

6. En la página **Listo para configurar**, asegúrese de que la casilla **Inicie el proceso de sincronización cuando se complete la configuración** esté activada. A continuación, seleccione **Configurar**.

   ![Captura de pantalla que muestra el botón Configurar en la página Listo para configurar](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image15.png)<br />
   Cuando se selecciona el botón **Configurar**, el inicio de sesión único de conexión directa se configura como se indica en el paso anterior. No se modifica la configuración de la sincronización de hash de contraseña porque se ha habilitado anteriormente.

   > [!IMPORTANT]
   > No se realiza ningún cambio en la forma en que los usuarios inician sesión en este momento.

7. En el portal de Azure AD, compruebe esta configuración:
   * **Federación** está establecido en **Habilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Habilitado**.
   * **Sincronización de contraseñas** está establecido en **Habilitado**.

   ![Captura de pantalla que muestra la configuración de la sección de inicio de sesión de usuario](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image16.png)

#### <a name="convert-domains-from-federated-to-managed"></a>Convertir dominios de federado a administrado

En este momento, la federación sigue habilitada y operativa para los dominios. Para continuar con la implementación, todos los dominios deben convertirse de federados a administrados para forzar la autenticación del usuario a través de Sincronización de hash de contraseña.

> [!IMPORTANT]
> No tiene que convertir todos los dominios al mismo tiempo. Puede empezar con un dominio de prueba en el inquilino de producción o con el dominio que tenga el menor número de usuarios.

Realice la conversión mediante el módulo de PowerShell de Azure AD:

1. En PowerShell, inicie sesión en Azure AD con una cuenta de administrador global.
2. Para convertir el primero dominio, ejecute el comando siguiente:

   ``` PowerShell
   Set-MsolDomainAuthentication -Authentication Managed -DomainName <domain name>
   ```

3. En el portal de Azure AD, seleccione **Azure Active Directory** > **Azure AD Connect**.
4. Compruebe que el dominio se ha convertido a administrado, para lo que debe ejecutar el siguiente comando:

   ``` PowerShell
   Get-MsolDomain -DomainName <domain name>
   ```

## <a name="testing-and-next-steps"></a>Pruebas y pasos siguientes

Complete las tareas siguientes para comprobar la sincronización de hash de contraseña y finalizar el proceso de conversión.

### <a name="test-authentication-by-using-password-hash-synchronization"></a>Autenticación de prueba mediante la sincronización de hash de contraseña 

Cuando el inquilino usaba la identidad federada, los usuarios eran redirigidos desde la página de inicio de sesión de Azure AD hasta el entorno de AD FS. Ahora que el inquilino está configurado para usar la sincronización de hash de contraseña en lugar de autenticación federada, los usuarios no se redirigen a AD FS. En su lugar, los usuarios inician sesión directamente en la página de inicio de sesión de Azure AD.

Para probar la sincronización de hash de contraseña:

1. Abra Internet Explorer en modo InPrivate para que el inicio de sesión único de conexión directa no inicie su sesión automáticamente.
2. Vaya a la página de inicio de sesión de Office 365 ([https://portal.office.com](https://portal.office.com/)).
3. Escriba un nombre principal de usuario y, a continuación, seleccione **Siguiente**. Asegúrese de escribir el nombre principal de usuario de un usuario híbrido que se sincronizó desde la instancia de Active Directory local y que usaba anteriormente la autenticación federada. Aparece una página en la que se escribe el nombre de usuario y la contraseña:

   ![Captura de pantalla que muestra la página de inicio de sesión en la que se escribe un nombre de usuario](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image18.png)

   ![Captura de pantalla que muestra la página de inicio de sesión en la que se escribe una contraseña](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image19.png)

4. Después de escribir la contraseña y seleccionar **Iniciar sesión**, se le redirigirá al portal de Office 365.

   ![Captura de pantalla que muestra el portal de Office 365](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image20.png)


### <a name="test-seamless-sso"></a>Prueba del inicio de sesión único de conexión directa

1. Inicie sesión en una máquina unida a un dominio que esté conectada a la red corporativa.
2. En Internet Explorer o Chrome, vaya a una de las siguientes direcciones URL (reemplace "contoso" por su dominio):

   * https:\/\/myapps.microsoft.com/contoso.com
   * https:\/\/myapps.microsoft.com/contoso.onmicrosoft.com

   Se redirige al usuario brevemente a la página de inicio de sesión de Azure AD que muestra el mensaje "Intentando iniciar sesión". No se le pedirá ni un nombre de usuario ni una contraseña.<br />

   ![Captura de pantalla que muestra la página de inicio de sesión de Azure AD y el mensaje](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image21.png)<br />
3. Se redirige al usuario, quien inicia sesión correctamente en el panel de acceso:

   > [!NOTE]
   > El inicio de sesión único de conexión directa funciona en los servicios de Office 365 que admiten sugerencias de dominio (por ejemplo, myapps.microsoft.com/contoso.com). Actualmente, el portal de Office 365 (portal.office.com) no es compatible con las sugerencias de dominio. Los usuarios deben escribir un nombre principal de usuario. Después de especificar un nombre principal de usuario, el inicio de sesión único de conexión directa recupera el vale de Kerberos en nombre del usuario. El usuario inicia sesión sin escribir ninguna contraseña.

   > [!TIP]
   > Considere la posibilidad de implementar la [unión híbrida de Azure AD en Windows 10](../devices/overview.md) para mejorar la experiencia de inicio de sesión único.

### <a name="remove-the-relying-party-trust"></a>Eliminación de la relación de confianza para usuario autenticado

Cuando haya comprobado que todos los usuarios y clientes se están autenticando correctamente mediante Azure AD, es seguro quitar la relación de confianza para usuario autenticado en Office 365.

Si no usa AD FS para otros fines (es decir, para otras relaciones de confianza), es seguro retirar AD FS en este momento.

### <a name="rollback"></a>Reversión

Si surge algún problema importante que no se pueda resolver rápidamente, puede decidir revertir la solución a la opción de federación.

Consulte la documentación acerca del diseño y la implementación de la federación para ver los detalles de su implementación concreta. El proceso debe incluir estas tareas:

* Conversión de dominios administrados a autenticación federada mediante el cmdlet **Convert-MSOLDomainToFederated**.
* Si fuera necesario, configure reglas de notificaciones adicionales.

### <a name="sync-userprincipalname-updates"></a>Sincronización de actualizaciones de userPrincipalName

Históricamente, las actualizaciones para el atributo **UserPrincipalName**, que usa el servicio de sincronización desde el entorno local, han estado bloqueadas, a menos que se cumplieran las siguientes condiciones:

* El usuario está en un dominio de identidad administrada (no federada).
* Al usuario no se le ha asignado una licencia.

Para saber cómo comprobar o activar esta característica, consulte [Sincronización de actualizaciones de userPrincipalName](./how-to-connect-syncservice-features.md).

### <a name="troubleshooting"></a>Solución de problemas

El equipo de soporte técnico debe saber solucionar cualquier problema de autenticación que surja durante el cambio de federación a administrado, o después. Use la siguiente documentación de solución de problemas para ayudar al equipo de soporte técnico a familiarizarse con los pasos comunes para la solución de problemas comunes y las acciones adecuadas que pueden ayudar a aislar y resolver el problema.

[Solución de problemas de sincronización de hash de contraseña en Azure Active Directory](./tshoot-connect-password-hash-synchronization.md)

[Solución de problemas de Inicio de sesión único de conexión directa en Azure Active Directory](./tshoot-connect-sso.md)

## <a name="roll-over-the-seamless-sso-kerberos-decryption-key"></a>Sustitución de la clave de descifrado de Kerberos de inicio de sesión único de conexión directa

Es importante implementar con frecuencia la clave de descifrado de Kerberos de la cuenta de equipo AZUREADSSOACC (que representa a Azure AD). La cuenta de equipo AZUREADSSOACC se crea en su bosque de Active Directory local. Se recomienda encarecidamente que sustituya la clave de descifrado de Kerberos al menos cada 30 días para que se corresponda con la forma en que los miembros del dominio de Active Directory envían los cambios de contraseña. Como no hay ningún dispositivo asociado conectado al objeto de la cuenta de equipo AZUREADSSOACC, la sustitución debe realizarse manualmente.

Inicie la sustitución de la clave de descifrado de Kerberos del inicio de sesión único de conexión directa en el servidor local que ejecuta Azure AD Connect.

Para más información, consulte [¿Cómo puedo implementar la clave de descifrado de Kerberos de la cuenta de equipo AZUREADSSOACC?](./how-to-connect-sso-faq.md).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [conceptos de diseño de Azure AD Connect](plan-connect-design-concepts.md).
* Selección de la [autenticación adecuada](./choose-ad-authn.md).
* Más información sobre las [topologías admitidas](plan-connect-design-concepts.md).