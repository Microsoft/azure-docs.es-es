---
title: 'Azure AD Connect: Migración desde la federación a PTA para Azure AD'
description: Este artículo contiene información acerca de cómo mover un entorno de identidad híbrida de la federación a la autenticación de paso a través.
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
ms.openlocfilehash: 84b81e32f8c71ccf0c3369e137a24e90dc284e86
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276599"
---
# <a name="migrate-from-federation-to-pass-through-authentication-for-azure-active-directory"></a>Migración de la federación a la autenticación de paso a través en Azure Active Directory

En este artículo se describe cómo mover los dominios de la organización de Servicios de federación de Active Directory (AD FS) a la autenticación de paso a través.

> [!NOTE]
> El cambio del método de autenticación necesita planeación, pruebas y un posible tiempo de inactividad. El [lanzamiento preconfigurado](how-to-connect-staged-rollout.md) proporciona una forma alternativa de probar y migrar gradualmente la autenticación de la federación a la nube mediante la autenticación de paso a través.
> 
> Si planea usar el lanzamiento preconfigurado, recuerde desactivar las características del lanzamiento por fases una vez que haya terminado la migración.  Para obtener más información, vea [Migración a la autenticación en la nube mediante un lanzamiento preconfigurado](how-to-connect-staged-rollout.md).


## <a name="prerequisites-for-migrating-to-pass-through-authentication"></a>Requisitos previos para la migración a la autenticación de paso a través

Los siguientes requisitos previos son necesarios para migrar del uso de AD FS al uso de la sincronización de paso a través.

### <a name="update-azure-ad-connect"></a>Actualización de Azure AD Connect

Para completar correctamente los pasos necesarios para migrar a la autenticación de paso a través, debe tener [Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594) (Azure AD Connect) 1.1.819.0 o una versión posterior. En Azure AD Connect 1.1.819.0, el modo en que se realiza la conversión del inicio de sesión cambia significativamente. El tiempo total para migrar de AD FS a la autenticación en la nube en esta versión se ha reducido de posiblemente horas a minutos.

> [!IMPORTANT]
> Es posible que lea en documentación, herramientas y blogs obsoletos que se requiere la conversión de los usuarios al convertir dominios de identidad federada a identidad administrada. Pues bien, la *conversión de los usuarios* ya no es necesaria. Microsoft está trabajando para actualizar la documentación y las herramientas de forma que reflejen este cambio.

Para actualizar Azure AD Connect, realice los pasos que se describen en [Azure AD Connect: actualización a la versión más reciente](./how-to-upgrade-previous-version.md).

### <a name="plan-authentication-agent-number-and-placement"></a>Planeación de la colocación y del número de agente de autenticación

Para la autenticación de paso a través, es necesario implementar agentes ligeros en el servidor de Azure AD Connect y en el equipo local que ejecuta Windows Server. Para reducir la latencia, instale los agentes lo más cerca posible de los controladores de Active Directory.

La mayoría de los clientes solo necesitarán dos o tres agentes de autenticación para obtener alta disponibilidad y la capacidad necesaria. Un inquilino puede tener registrados hasta 12 agentes. El primer agente siempre se instala en el propio servidor de Azure AD Connect. Para conocer las limitaciones de agentes y las opciones de implementación de agentes, consulte [Autenticación de paso a través de Azure AD: limitaciones actuales](./how-to-connect-pta-current-limitations.md).

### <a name="plan-the-migration-method"></a>Planeamiento del método de migración

Puede elegir entre dos métodos para migrar de la administración de identidades federadas a la autenticación de paso a través y el inicio de sesión único (SSO) de conexión directa. El método que utilice dependerá de la forma en que la instancia de AD FS se haya configurado originalmente.

* **Azure AD Connect**. Si originalmente configuró AD FS con Azure AD Connect, *debe* cambiar a la autenticación de paso a través mediante el asistente de Azure AD Connect.

   ‎Azure AD Connect ejecuta automáticamente el cmdlet **Set-MsolDomainAuthentication** al cambiar el método de inicio de sesión de usuario. Azure AD Connect anula automáticamente la federación de todos los dominios federados comprobados en el inquilino de Azure AD.

   > [!NOTE]
   > En la actualidad, si usó originalmente Azure AD Connect para configurar AD FS, no podrá evitar anular la federación de todos los dominios del inquilino cuando se cambia el inicio de sesión del usuario a la autenticación de paso a través.
‎
* **Azure AD Connect con PowerShell**. Puede usar este método solo si no configuró originalmente AD FS con Azure AD Connect. Con esta opción, todavía debe cambiar el método de inicio de sesión de usuario mediante el asistente de Azure AD Connect. La diferencia principal con esta opción es que el Asistente no ejecuta automáticamente el cmdlet **Set-MsolDomainAuthentication**. Con esta opción, tiene control total sobre qué dominios se convierten y en qué orden.

Para entender qué método debe usar, siga los pasos de las próximas secciones.

#### <a name="verify-current-user-sign-in-settings"></a>Comprobación de la configuración del inicio de sesión del usuario actual

1. Inicie sesión en el [portal de Azure AD](https://aad.portal.azure.com/) con una cuenta de administrador global.
2. En la sección **Inicio de sesión de usuario**, compruebe la configuración siguiente:
   * **Federación** está establecido en **Habilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Deshabilitado**.
   * **Autenticación de paso a través** está establecido en **Deshabilitado**.

   ![Captura de pantalla de la configuración de la sección de inicio de sesión de usuario de Azure AD Connect](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image1.png)

#### <a name="verify-how-federation-was-configured"></a>Comprobación de cómo se ha configurado la federación

1. En el servidor de Azure AD Connect, abra Azure AD Connect. Seleccione **Configurar**.
2. En la página **Tareas adicionales**, seleccione **Ver configuración actual** y, después, seleccione **Siguiente**.<br />
 
   ![Captura de pantalla de la opción Ver configuración actual de la página Tareas adicionales](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image2.png)<br />
3. En **Additional Tasks > Manage Federation** (Tareas adicionales > Administrar federación), desplácese hasta **Active Directory Federation Services (AD FS)** (Servicios de federación de Active Directory [AD FS]).<br />

   * Si la configuración de AD FS aparece en esta sección, puede suponer con seguridad que AD FS se configuró originalmente con Azure AD Connect. Puede convertir los dominios de identidad federada a identidad administrada mediante la opción **Cambiar inicio de sesión de usuario** de Azure AD Connect. Para más información sobre este proceso, consulte la sección **Opción A: Configuración de la autenticación de paso a través mediante Azure AD Connect**.
   * Si AD FS no aparece en la configuración actual, debe convertir manualmente los dominios de identidad federada a identidad administrada mediante PowerShell. Para obtener más información sobre este proceso, consulte la sección **Opción B: Cambio de la federación a la autenticación de paso a través mediante Azure AD Connect y PowerShell**.

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
> Si el valor **SupportsMfa** está establecido en **True**, está usando una solución de autenticación multifactor local para insertar un segundo factor en el flujo de autenticación del usuario. Esta configuración ya no funciona en escenarios de autenticación de Azure AD. 
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

AD FS emite la notificación **InsideCorporateNetwork** si el usuario que realiza la autenticación está dentro de la red corporativa. Esta notificación se puede transmitir entonces a Azure AD. La notificación se utiliza para omitir la autenticación multifactor basada en la ubicación de red del usuario. Para aprender a determinar si esta funcionalidad actualmente está disponible en AD FS, consulte [Direcciones IP de confianza para usuarios federados](../authentication/howto-mfa-adfs.md).

La notificación **InsideCorporateNetwork** no está disponible cuando los dominios se conviertan a la autenticación de paso a través. Se pueden usar las [ubicaciones con nombre de Azure AD](../reports-monitoring/quickstart-configure-named-locations.md) para reemplazar esta funcionalidad.

Una vez que se han configurado las ubicaciones con nombre, debe actualizar todas las directivas de acceso condicional que se configuraron para incluir o excluir los valores de red **Todas las ubicaciones de confianza** o **IP de confianza de MFA** para reflejar las ubicaciones con nombre recién creadas.

Para más información sobre la condición **Ubicación** en el acceso condicional, consulte [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?](../conditional-access/location-condition.md).

#### <a name="hybrid-azure-ad-joined-devices"></a>Dispositivos unidos a Azure AD híbrido

Al unir un dispositivo a Azure AD, puede crear reglas de acceso condicional que obligan a que el acceso desde dispositivos cumpla con las normas de seguridad y cumplimiento. Además, los usuarios pueden iniciar sesión en un dispositivo mediante una cuenta profesional o educativa en lugar de una cuenta personal. Al usar dispositivos unidos a Azure AD híbrido, puede unir a Azure AD sus dispositivos unidos a un dominio de Active Directory. Es posible que un entorno federado se haya configurado para usar esta característica.

Para asegurarse de que la unión híbrida sigue funcionando en todos los dispositivos unidos al dominio una vez que los dominios se han convertido a la autenticación de paso a través, para clientes de Windows 10, debe usar Azure AD Connect para sincronizar las cuentas de equipos de Active Directory con Azure AD.

En el caso de las cuentas de equipos de Windows 8 y Windows 7, la unión híbrida usa SSO de conexión directa para registrar el equipo en Azure AD. No es necesario sincronizar cuentas de equipo de Windows 8 y Windows 7 como hace para dispositivos con Windows 10. Sin embargo, debe implementar una versión actualizada del archivo workplacejoin.exe (a través de un archivo .msi) en los clientes de Windows 8 y Windows 7 para que puedan registrarse mediante SSO de conexión directa. [Descargue el archivo .msi](https://www.microsoft.com/download/details.aspx?id=53554).

Para más información, consulte [Configuración de dispositivos unidos a Azure AD híbrido](../devices/hybrid-azuread-join-plan.md).

#### <a name="branding"></a>Personalización de marca

Si su organización [personalizó las páginas de inicio de sesión de AD FS](/windows-server/identity/ad-fs/operations/ad-fs-user-sign-in-customization) para mostrar información que es más pertinente para la organización, considere la posibilidad de establecer [personalizaciones similares en la página de inicio de sesión de Azure AD](../fundamentals/customize-branding.md).

Aunque hay personalizaciones similares disponibles, cabe esperar que haya algunos cambios visuales en las páginas de inicio de sesión tras la conversión. Es posible que desee proporcionar información sobre los cambios previstos en sus comunicaciones a los usuarios.

> [!NOTE]
> La personalización de marca de empresa solo está disponible si ha adquirido las licencias Premium o Básica de Azure Active Directory, o si tiene una licencia de Office 365.

## <a name="plan-for-smart-lockout"></a>Plan de bloqueo inteligente

El bloqueo inteligente de Azure AD protege contra los ataques de contraseña por fuerza bruta. El bloqueo inteligente impide que una cuenta de Active Directory local se bloquee cuando se usa la autenticación de paso a través y se establece una directiva de grupo de bloqueo de cuenta en Active Directory.

Para más información, consulte [Ediciones de Azure Active Directory](../authentication/howto-password-smart-lockout.md).

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

Una vez que se hayan implementado tanto la autenticación de paso a través como el SSO de conexión directa, cambia la experiencia de inicio de sesión del usuario para acceder a Office 365 y otros recursos autenticados mediante Azure AD. Los usuarios que están fuera de la red solo ven la página de inicio de sesión de Azure AD. Estos usuarios no son redirigidos a la página basada en formularios que se presenta por los servidores proxy de aplicación web de uso externo.

Incluya los siguientes elementos en su estrategia de comunicación:

* Informe a los usuarios de las funcionalidades próximas y publicadas por los medios siguientes:
   * Correo electrónico y otros canales de comunicación interna.
   * Objetos visuales, como pósteres.
   * Comunicaciones en directo, ejecutivas o de otro tipo.
* Determinación de quién va a personalizar las comunicaciones y quién va a enviarlas y cuándo.

## <a name="implement-your-solution"></a>Implementación de la solución

Ya ha planeado la solución. Ahora, ya puede implementarla. La implementación incluye los siguientes componentes:

* Preparación para el SSO de conexión directa.
* Cambio del método de inicio de sesión a la autenticación de paso a través y habilitación de SSO de conexión directa.

### <a name="step-1-prepare-for-seamless-sso"></a>Paso 1: Preparación para el SSO de conexión directa

Para que los dispositivos usen SSO de conexión directa, debe agregar una dirección URL de Azure AD a la configuración de la zona de intranet de los usuarios mediante una directiva de grupo de Active Directory.

De forma predeterminada, los exploradores web calculan automáticamente la zona correcta (Internet o intranet) a partir de una dirección URL. Por ejemplo, **http:\/\/contoso/** se asigna a la zona de intranet, mientras que **http:\/\/intranet.contoso.com** se asigna a la zona de Internet (porque la dirección URL contiene un punto). Los exploradores envían vales Kerberos a un punto de conexión en la nube, como la dirección URL de Azure AD, solo si agrega explícitamente la dirección URL a la zona de intranet del explorador.

Siga los pasos para [implementar](./how-to-connect-sso-quick-start.md) los cambios necesarios en los dispositivos.

> [!IMPORTANT]
> Hacer este cambio no modifica la forma en que los usuarios inician sesión en Azure AD. Sin embargo, es importante que aplique esta configuración a todos los dispositivos antes de continuar. Los usuarios que inicien sesión en dispositivos que no hayan recibido esta configuración solo tendrán que escribir un nombre de usuario y una contraseña para iniciar sesión Azure AD.

### <a name="step-2-change-the-sign-in-method-to-pass-through-authentication-and-enable-seamless-sso"></a>Paso 2: Cambio del método de inicio de sesión a la autenticación de paso a través y habilitación de SSO de conexión directa

Tiene dos opciones para cambiar el método de inicio de sesión a autenticación de paso a través y habilitación del SSO de conexión directa.

#### <a name="option-a-configure-pass-through-authentication-by-using-azure-ad-connect"></a>Opción A: Configuración de la autenticación de paso a través mediante Azure AD Connect

Use este método si ha configurado inicialmente su entorno de AD FS con Azure AD Connect. No puede usar este método si *no* configuró inicialmente su entorno de AD FS con Azure AD Connect.

> [!IMPORTANT]
> Después de completar los pasos siguientes, todos los dominios pasan de identidad federada a identidad administrada. Para más información, revise [Planeamiento del método de migración](#plan-the-migration-method).

En primer lugar, cambie el método de inicio de sesión:

1. En el servidor de Azure AD Connect, abra el asistente de Azure AD Connect.
2. Seleccione **Cambiar inicio de sesión de usuario** y, después, seleccione **Siguiente**. 
3. En la pantalla **Conectar a Azure AD**, indique el nombre de usuario y la contraseña de una cuenta de administrador global.
4. En la página **Inicio de sesión de usuario**, seleccione el botón **Autenticación de paso a través**, seleccione **Habilitar inicio de sesión único** y, luego, seleccione **Siguiente**.
5. En la página **Habilitar inicio de sesión único**, escriba las credenciales de la cuenta de administrador del dominio y, después, seleccione **Siguiente**.

   > [!NOTE]
   > Las credenciales de la cuenta de administrador de dominio son necesarias para habilitar el inicio de sesión único de conexión directa. Durante el proceso se realizan las acciones siguientes, que requieren estos permisos elevados. Las credenciales de la cuenta de administrador de dominio no se almacenan en Azure AD Connect ni en Azure AD. Las credenciales de la cuenta de administrador de dominio se usan solo para activar la característica. Cuando el proceso finaliza correctamente, estas credenciales se descartan.
   >
   > 1. Se crea una cuenta de equipo denominada AZUREADSSOACC (que representa a Azure AD) en la instancia local de Active Directory.
   > 2. La clave de descifrado de Kerberos de la cuenta de equipo se comparte de manera segura con Azure AD.
   > 3. Se crean dos nombres de entidad de seguridad de servicio (SPN) de Kerberos que representan las dos direcciones URL que se usan en el inicio de sesión de Azure AD.

6. En la página **Listo para configurar**, asegúrese de que la casilla **Inicie el proceso de sincronización cuando se complete la configuración** esté activada. A continuación, seleccione **Configurar**.<br />

   ![Captura de pantalla de la página Listo para configurar](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image8.png)<br />
7. Abra el portal de Azure AD, seleccione **Azure Active Directory** y, después, seleccione **Azure AD Connect**.
8. Compruebe la configuración de estos parámetros:
   * **Federación** está establecido en **Deshabilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Habilitado**.
   * **Autenticación de paso a través** está establecido en **Deshabilitado**.<br />

   ![Captura de pantalla que muestra la configuración de la sección de inicio de sesión de usuario](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image9.png)<br />

A continuación, implemente métodos de autenticación adicionales:

1. En Azure Portal, vaya a **Azure Active Directory** > **Azure AD Connect** y, luego, seleccione **Autenticación de paso a través**.
2. En la página **Autenticación de paso a través**, seleccione el botón **Descargar**.
3. En la página **Descargar agente**, seleccione **Accept terms and download** (Aceptar los términos y descargar).

   Los agentes de autenticación adicionales comienzan a descargarse. Instale el agente de autenticación secundario en un servidor unido a un dominio. 

   > [!NOTE]
   > El primer agente siempre se instala en el mismo servidor de Azure AD Connect como parte de los cambios de configuración realizados en la sección **Inicio de sesión de usuario** de la herramienta Azure AD Connect. Instale los agentes de autenticación adicionales en otro servidor. Se recomienda tener dos o tres agentes de autenticación adicionales disponibles. 

4. Ejecute la instalación del agente de autenticación. Durante la instalación, debe especificar las credenciales de una cuenta de administrador global.

   ![Captura de pantalla que muestra el botón Instalar en la página del paquete de agente de autenticación de Microsoft Azure AD Connect](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image11.png)

   ![Captura de pantalla que muestra la página de inicio de sesión](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image12.png)

5. Una vez instalado el agente de autenticación, puede volver a la página de mantenimiento del agente de autenticación de paso a través para comprobar el estado de los agentes adicionales.

Vaya a [Pruebas y pasos siguientes](#testing-and-next-steps).

> [!IMPORTANT]
> Omita la sección **Opción B: Cambio de la federación a la autenticación de paso a través mediante Azure AD Connect y PowerShell**. Los pasos de esa sección no se aplican si eligió la Opción A para cambiar el método de inicio de sesión a autenticación de paso a través y habilitación de SSO de conexión directa. 

#### <a name="option-b-switch-from-federation-to-pass-through-authentication-by-using-azure-ad-connect-and-powershell"></a>Opción B: Cambio de la federación a la autenticación de paso a través mediante Azure AD Connect y PowerShell

Use esta opción si no configuró inicialmente los dominios federados con Azure AD Connect.

Primero, habilite la autenticación de paso a través:

1. En el servidor de Azure AD Connect, abra el asistente de Azure AD Connect.
2. Seleccione **Cambiar inicio de sesión de usuario** y, después, seleccione **Siguiente**.
3. En la pantalla **Conectar a Azure AD**, indique el nombre de usuario y la contraseña de una cuenta de administrador global.
4. En la página **Inicio de sesión de usuario**, seleccione el botón **Autenticación de paso a través**. Seleccione **Habilitar inicio de sesión único** y, después, **Siguiente**.
5. En la página **Habilitar inicio de sesión único**, escriba las credenciales de la cuenta de administrador del dominio y, después, seleccione **Siguiente**.

   > [!NOTE]
   > Las credenciales de la cuenta de administrador de dominio son necesarias para habilitar el inicio de sesión único de conexión directa. Durante el proceso se realizan las acciones siguientes, que requieren estos permisos elevados. Las credenciales de la cuenta de administrador de dominio no se almacenan en Azure AD Connect ni en Azure AD. Las credenciales de la cuenta de administrador de dominio se usan solo para activar la característica. Cuando el proceso finaliza correctamente, estas credenciales se descartan.
   > 
   > 1. Se crea una cuenta de equipo denominada AZUREADSSOACC (que representa a Azure AD) en la instancia local de Active Directory.
   > 2. La clave de descifrado de Kerberos de la cuenta de equipo se comparte de manera segura con Azure AD.
   > 3. Se crean dos nombres de entidad de seguridad de servicio (SPN) de Kerberos que representan las dos direcciones URL que se usan en el inicio de sesión de Azure AD.

6. En la página **Listo para configurar**, asegúrese de que la casilla **Inicie el proceso de sincronización cuando se complete la configuración** esté activada. A continuación, seleccione **Configurar**.<br />

   ‎![Captura de pantalla que muestra la página Listo para configurar y el botón Configurar](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image18.png)<br />
   Al seleccionar **Configurar**, tienen lugar los siguientes pasos:

   1. Se instala el primer agente de autenticación de paso a través.
   2. Se habilita la característica de paso a través.
   3. Se habilita SSO de conexión directa.

7. Compruebe la configuración de estos parámetros:
   * **Federación** está establecido en **Habilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Habilitado**.
   * **Autenticación de paso a través** está establecido en **Deshabilitado**.
   
   ![Captura de pantalla que muestra la configuración de la sección de inicio de sesión de usuario](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image19.png)
8. Seleccione **Autenticación de paso a través** y compruebe que el estado es **Activo**.<br />
   
   Si el agente de autenticación no está activo, lleva a cabo algunos [pasos de solución de problemas](./tshoot-connect-pass-through-authentication.md) antes de continuar con el proceso de conversión de dominio en el paso siguiente. Si convierte los dominios antes de validar que los agentes de autenticación de paso a través están instalados correctamente y que su estado es **Activo** en Azure Portal, se arriesga a provocar una interrupción de la autenticación.

A continuación, implemente agentes de autenticación adicionales:

1. En Azure Portal, vaya a **Azure Active Directory** > **Azure AD Connect** y, luego, seleccione **Autenticación de paso a través**.
2. En la página **Autenticación de paso a través**, seleccione el botón **Descargar**. 
3. En la página **Descargar agente**, seleccione **Accept terms and download** (Aceptar los términos y descargar).
 
   El agente de autenticación comienza a descargarse. Instale el agente de autenticación secundario en un servidor unido a un dominio.

   > [!NOTE]
   > El primer agente siempre se instala en el mismo servidor de Azure AD Connect como parte de los cambios de configuración realizados en la sección **Inicio de sesión de usuario** de la herramienta Azure AD Connect. Instale los agentes de autenticación adicionales en otro servidor. Se recomienda tener dos o tres agentes de autenticación adicionales disponibles.
 
4. Ejecute la instalación del agente de autenticación. Durante la instalación, deberá proporcionar las credenciales de una cuenta de administrador global.<br />

   ![Captura de pantalla que muestra el botón Instalar en la página del paquete de agente de autenticación de Microsoft Azure AD Connect](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image23.png)<br />
   ![Captura de pantalla que muestra la página de inicio de sesión](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image24.png)<br />
5. Después de instalar el agente de autenticación, puede volver a la página de estado del agente de autenticación de paso a través para comprobar el estado de los agentes adicionales.

En este momento, autenticación federada sigue activa y operativa para los dominios. Para continuar con la implementación, debe convertir cada dominio de identidad federada a identidad administrada para que la autenticación de paso a través comience a atender las solicitudes de autenticación del dominio.

No tiene que convertir todos los dominios al mismo tiempo. Puede empezar con un dominio de prueba en el inquilino de producción o con el dominio que tenga el menor número de usuarios.

Realice la conversión mediante el módulo de PowerShell de Azure AD:

1. En PowerShell, inicie sesión en Azure AD con una cuenta de administrador global.
2. Para convertir el primero dominio, ejecute el comando siguiente:
 
   ``` PowerShell
   Set-MsolDomainAuthentication -Authentication Managed -DomainName <domain name>
   ```
 
3. En el portal de Azure AD, seleccione **Azure Active Directory** > **Azure AD Connect**.
4. Después de convertir todos los dominios federados, compruebe esta configuración:
   * **Federación** está establecido en **Deshabilitado**.
   * **Inicio de sesión único de conexión directa** está establecido en **Habilitado**.
   * **Autenticación de paso a través** está establecido en **Deshabilitado**.<br />

   ![Captura de pantalla que muestra la configuración de la sección de inicio de sesión de usuario](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image26.png)<br />

## <a name="testing-and-next-steps"></a>Pruebas y pasos siguientes

Realice las tareas siguientes para comprobar la autenticación de paso a través y finalizar el proceso de conversión.

### <a name="test-pass-through-authentication"></a>Prueba de la autenticación de paso a través 

Cuando el inquilino usaba la identidad federada, los usuarios eran redirigidos desde la página de inicio de sesión de Azure AD hasta el entorno de AD FS. Ahora que el inquilino está configurado para usar la autenticación de paso a través en lugar de la autenticación federada, los usuarios no se redirigen a AD FS. En su lugar, los usuarios inician sesión directamente en la página de inicio de sesión de Azure AD.

Para probar la autenticación de paso a través:

1. Abra Internet Explorer en modo InPrivate para que el inicio de sesión único de conexión directa no inicie su sesión automáticamente.
2. Vaya a la página de inicio de sesión de Office 365 ([https://portal.office.com](https://portal.office.com/)).
3. Escriba un nombre principal de usuario y, a continuación, seleccione **Siguiente**. Asegúrese de escribir el nombre principal de usuario de un usuario híbrido que se sincronizó desde la instancia de Active Directory local y que usaba anteriormente la autenticación federada. Aparece una página en la que se escribe el nombre de usuario y la contraseña:

   ![Captura de pantalla que muestra la página de inicio de sesión en la que se escribe un nombre de usuario](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image27.png)

   ![Captura de pantalla que muestra la página de inicio de sesión en la que se escribe una contraseña](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image28.png)

4. Después de escribir la contraseña y seleccionar **Iniciar sesión**, se le redirigirá al portal de Office 365.

   ![Captura de pantalla que muestra el portal de Office 365](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image29.png)

### <a name="test-seamless-sso"></a>Prueba del inicio de sesión único de conexión directa

Para probar el inicio de sesión único de conexión directa:

1. Inicie sesión en una máquina unida a un dominio que esté conectada a la red corporativa.
2. En Internet Explorer o Chrome, vaya a una de las siguientes direcciones URL (reemplace "contoso" por su dominio):

   * https:\/\/myapps.microsoft.com/contoso.com
   * https:\/\/myapps.microsoft.com/contoso.onmicrosoft.com

   Se redirige al usuario brevemente a la página de inicio de sesión de Azure AD que muestra el mensaje "Intentando iniciar sesión". No se le pedirá ni un nombre de usuario ni una contraseña.<br />

   ![Captura de pantalla que muestra la página de inicio de sesión de Azure AD y el mensaje](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image30.png)<br />
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

## <a name="roll-over-the-seamless-sso-kerberos-decryption-key"></a>Sustitución de la clave de descifrado de Kerberos de inicio de sesión único de conexión directa

Es importante implementar con frecuencia la clave de descifrado de Kerberos de la cuenta de equipo AZUREADSSOACC (que representa a Azure AD). La cuenta de equipo AZUREADSSOACC se crea en su bosque de Active Directory local. Se recomienda encarecidamente que sustituya la clave de descifrado de Kerberos al menos cada 30 días para que se corresponda con la forma en que los miembros del dominio de Active Directory envían los cambios de contraseña. Como no hay ningún dispositivo asociado conectado al objeto de la cuenta de equipo AZUREADSSOACC, la sustitución debe realizarse manualmente.

Inicie la sustitución de la clave de descifrado de Kerberos del inicio de sesión único de conexión directa en el servidor local que ejecuta Azure AD Connect.

Para más información, consulte [¿Cómo puedo implementar la clave de descifrado de Kerberos de la cuenta de equipo AZUREADSSOACC?](./how-to-connect-sso-faq.md).

## <a name="monitoring-and-logging"></a>Supervisión y registro

Supervise los servidores que ejecutan los agentes de autenticación para mantener la disponibilidad de la solución. Además de los contadores de rendimiento generales del servidor, los agentes de autenticación exponen objetos de rendimiento que pueden usarse para comprender las estadísticas de autenticación y los errores.

Los agentes de autenticación registran las operaciones en los registros de eventos de Windows en "Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin".

También puede activar el registro para solucionar problemas.

Para más información, lea [Solución de problemas de autenticación de paso a través de Azure Active Directory](./tshoot-connect-pass-through-authentication.md).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [conceptos de diseño de Azure AD Connect](plan-connect-design-concepts.md).
* Selección de la [autenticación adecuada](./choose-ad-authn.md).
* Más información sobre las [topologías admitidas](plan-connect-design-concepts.md).