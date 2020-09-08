---
title: 'Azure AD Connect: Inicio de sesión de usuario | Microsoft Docs'
description: Inicio de sesión de usuarios de Azure AD Connect para la configuración personalizada
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 547b118e-7282-4c7f-be87-c035561001df
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/31/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: cd1d03edad2e13d3460fb2e7a635bb198ac5c1bf
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279625"
---
# <a name="azure-ad-connect-user-sign-in-options"></a>Opciones para el inicio de sesión de los usuarios en Azure AD Connect
Azure Active Directory (Azure AD) Connect permite que los usuarios inicien sesión en los recursos en la nube y locales con las mismas contraseñas. En este artículo se describen los conceptos clave para cada modelo de identidad para ayudarlo a elegir la identidad que desea utilizar con el fin de iniciar sesión en Azure AD.

Si ya está familiarizado con el modelo de identidad de Azure AD y desea obtener más información sobre un método específico, haga clic más abajo en el vínculo que corresponda:

* [Sincronización de hash de contraseña](#password-hash-synchronization) con [Inicio de sesión único de conexión directa](how-to-connect-sso.md)
* [Autenticación de paso a través](how-to-connect-pta.md) con [Inicio de sesión único de conexión directa](how-to-connect-sso.md)
* [SSO federado (con Active Directory Federation Services [AD FS])](#federation-that-uses-a-new-or-existing-farm-with-ad-fs-in-windows-server-2012-r2)
* [Federación con PingFederate](#federation-with-pingfederate)

> [!NOTE] 
> Es importante recordar que, al configurar la federación de Azure AD, establece confianza entre su inquilino de Azure AD y los dominios federados. Con este dominio federado de confianza, los usuarios tendrán acceso a los recursos de nube de Azure AD en el inquilino.  
>

## <a name="choosing-the-user-sign-in-method-for-your-organization"></a>Elección del método de inicio de sesión de usuario para su organización
La primera decisión de la implementación de Azure AD Connect consiste en elegir el método de autenticación que los usuarios utilizarán para iniciar sesión. Es importante asegurarse de que elige el método adecuado que cumpla los requisitos avanzados y de seguridad de la organización. La autenticación es fundamental, ya que validará las identidades del usuario para obtener acceso a aplicaciones y datos en la nube. Para elegir el método de autenticación adecuado, es necesario tener en cuenta el tiempo, la infraestructura existente, la complejidad y el costo de implementación de cada opción. Estos factores varían con cada organización y pueden cambiar con el tiempo.

Azure AD admite los siguientes métodos de autenticación: 

* **Autenticación en la nube**: cuando se elige este método de autenticación, Azure AD administra el proceso de autenticación del inicio de sesión de los usuarios. Con la autenticación en la nube puede elegir entre dos opciones: 
   * **Sincronización de hash de contraseñas (PHS)** : la sincronización de hash de contraseñas permite a los usuarios usar el mismo nombre de usuario y contraseña que en el entorno local sin tener que implementar infraestructura adicional, además de Azure AD Connect. 
   * **Autenticación de paso a través (PTA)** : esta opción es similar a la sincronización de hash de contraseñas, pero proporciona una validación simple de contraseñas con agentes de software en el entorno local para las organizaciones con directivas estrictas de seguridad y cumplimiento.
* **Autenticación federada**: cuando se elige este método de autenticación, Azure AD deja el proceso de autenticación en manos de un sistema de autenticación de confianza como, por ejemplo, una instancia de AD FS o un sistema de federación de terceros, para validar el inicio de sesión del usuario. 

Para la mayoría de las organizaciones que simplemente quieren habilitar el inicio de sesión de usuarios en Office 365, aplicaciones SaaS y otros recursos basados en Azure AD, se recomienda la opción de sincronización de hash de contraseña predeterminada.
 
Para obtener información detallada sobre cómo elegir un método de autenticación, vea [Elegir el método de autenticación adecuado para su solución de identidad híbrida de Azure Active Directory](./choose-ad-authn.md)

### <a name="password-hash-synchronization"></a>Sincronización de hash de contraseña
Con la sincronización de hash de contraseña, se sincronizan los valores hash de las contraseñas de los usuarios de Active Directory local con Azure AD. Cuando las contraseñas se cambian o se restablecen de forma local, los nuevos hash de contraseña se sincronizan de inmediato con Azure AD para que los usuarios puedan usar siempre en los recursos en la nube la misma contraseña que usan de manera local. Las contraseñas nunca se envían a Azure AD ni se almacenan en Azure AD en texto no cifrado. La sincronización de hash de contraseña puede usarse junto con la escritura diferida de contraseñas para habilitar el autoservicio de restablecimiento de contraseña en Azure AD.

También puede habilitar el [inicio de sesión único de conexión directa](how-to-connect-sso.md) para usuarios en máquinas unidas a un dominio que se encuentran en la red corporativa. Con el inicio de sesión único, los usuarios habilitados solo necesitan escribir un nombre de usuario para acceder a los recursos de nube de manera segura.

![Sincronización de hash de contraseña](./media/plan-connect-user-signin/passwordhash.png)

Para más información, vea el artículo [Implementación de la sincronización de contraseña mediante la sincronización de Azure AD Connect](how-to-connect-password-hash-synchronization.md).

### <a name="pass-through-authentication"></a>Autenticación de paso a través
Con la autenticación de paso a través, la contraseña del usuario se valida con el controlador de Active Directory local, y la contraseña no necesita estar presente en Azure AD de ninguna forma. Esto permite evaluar las directivas locales, como las restricciones en la hora de inicio de sesión, durante la autenticación en Cloud Services.

La autenticación de paso a través utiliza un agente sencillo en una máquina unida a un dominio de Windows Server 2012 R2 en el entorno local, que escucha las solicitudes de validación de contraseñas. Este agente no requiere que ningún puerto de entrada esté abierto a Internet.

También puede habilitar el inicio de sesión único para usuarios en máquinas unidas a un dominio que se encuentran en la red corporativa. Con el inicio de sesión único, los usuarios habilitados solo necesitan escribir un nombre de usuario para acceder a los recursos de nube de manera segura.
![Autenticación de paso a través](./media/plan-connect-user-signin/pta.png)

Para más información, consulte:
- [Autenticación de paso a través](how-to-connect-pta.md)
- [Inicio de sesión único](how-to-connect-sso.md)

### <a name="federation-that-uses-a-new-or-existing-farm-with-ad-fs-in-windows-server-2012-r2"></a>Federación con AD FS nuevo o existente en granja de Windows Server 2012 R2
Con el inicio de sesión federado, los usuarios pueden iniciar sesión en servicios basados en Azure con sus contraseñas locales. Aunque se encuentren en la red corporativa, no tendrán que escribir sus contraseñas. La opción de federación con AD FS permite implementar una granja nueva o existente con AD FS en Windows Server 2012 R2. Si decide especificar una granja existente, Azure AD Connect configurará la confianza entre la granja y Azure AD para que los usuarios puedan iniciar sesión.

<center>

![Federación con AD FS en Windows Server 2012 R2](./media/plan-connect-user-signin/federatedsignin.png)</center>

#### <a name="deploy-federation-with-ad-fs-in-windows-server-2012-r2"></a>Implementación de la federación con AD FS en Windows Server 2012 R2

Si va a implementar una nueva granja, necesita lo siguiente:

* Un servidor Windows Server 2012 R2 para el servidor de federación.
* Un servidor Windows Server 2012 R2 para el Proxy de aplicación web.
* Un archivo .pfx con un certificado TLS/SSL para el nombre de servicio de federación previsto. Por ejemplo: fs.contoso.com.

Si va a implementar una nueva granja o va a utilizar una existente, necesita esto:

* Credenciales de administrador local en los servidores de federación.
* Credenciales de administrador local en cualquier servidor de grupo de trabajo (no unido a dominio) en el que pretenda implementar el rol Proxy de aplicación web.
* La máquina en la que se ejecuta el Asistente debe poder conectarse a otras máquinas en las que va a instalar AD FS o el Proxy de aplicación web a través de Administración remota de Windows.

Para obtener más información, consulte [Configuración de SSO con AD FS](how-to-connect-install-custom.md#configuring-federation-with-ad-fs).

### <a name="federation-with-pingfederate"></a>Federación con PingFederate
Con el inicio de sesión federado, los usuarios pueden iniciar sesión en servicios basados en Azure con sus contraseñas locales. Aunque se encuentren en la red corporativa, no tendrán que escribir sus contraseñas.

Para más información acerca de cómo configurar PingFederate para su uso con Azure Active Directory, consulte [PingFederate Integration with Azure Active Directory and Office 365](https://www.pingidentity.com/AzureADConnect) (Integración de PingFederate con Azure Active Directory y Office 365)

Para más información sobre cómo configurar Azure AD Connect con PingFederate, consulte [Instalación personalizada de Azure AD Connect](how-to-connect-install-custom.md#configuring-federation-with-pingfederate)

#### <a name="sign-in-by-using-an-earlier-version-of-ad-fs-or-a-third-party-solution"></a>Inicio de sesión con una versión anterior de AD FS o una solución de terceros
Si ya ha configurado el inicio de sesión en la nube con una versión anterior de AD FS (por ejemplo, AD FS 2.0) o un proveedor de federación de terceros, puede optar por omitir la configuración del inicio de sesión de usuarios a través de Azure AD Connect. De este modo, podrá obtener la sincronización más reciente y otras funcionalidades de Azure AD Connect mientras sigue usando la solución existente para el inicio de sesión.

Para obtener más información, consulte [Lista de compatibilidad de federación de terceros de AD Azure](how-to-connect-fed-compatibility.md).


## <a name="user-sign-in-and-user-principal-name"></a>Inicio de sesión de usuario y nombre principal de usuario (UPN)
### <a name="understanding-user-principal-name"></a>Descripción del nombre principal de usuario
En Active Directory, el sufijo UPN predeterminado es el nombre DNS del dominio en el que se creó la cuenta de usuario. En la mayoría de los casos, se trata del nombre de dominio registrado como dominio de empresa en Internet. Sin embargo, puede agregar más sufijos UPN usando Dominios y confianzas de Active Directory.

El UPN del usuario tiene el formato username@domain. Por ejemplo, para un dominio de Active Directory denominado "contoso.com", un usuario llamado Héctor puede tener el UPN john@contoso.com. El UPN del usuario se basa en la especificación RFC 822. Aunque el UPN y el correo electrónico comparten el mismo formato, el valor de UPN de un usuario podría ser o no el mismo que la dirección de correo electrónico del usuario.

### <a name="user-principal-name-in-azure-ad"></a>Nombre principal de usuario en Azure AD
El Asistente para Azure AD Connect usará el atributo userPrincipalName o le permiten especificar el atributo (en la instalación personalizada) para su uso local como nombre principal de usuario en Azure AD. Este es el valor que se usará para iniciar sesión en Azure AD. Si el valor del atributo userPrincipalName no corresponde a un dominio comprobado en Azure AD, a continuación, Azure AD lo reemplazará con un valor predeterminado ".onmicrosoft.com".

Todos los directorios de Azure Active Directory vienen con un nombre de dominio integrado con el formato "contoso.onmicrosoft.com", lo que le permite empezar a utilizar Azure u otros servicios de Microsoft. Puede mejorar y simplificar la experiencia de inicio de sesión utilizando dominios personalizados. Para informarse sobre los nombres de dominio personalizado en Azure AD y cómo comprobar un dominio, consulte [Incorporación de su nombre de dominio personalizado a Azure Active Directory](../fundamentals/add-custom-domain.md).

## <a name="azure-ad-sign-in-configuration"></a>Configuración de inicio de sesión de Azure AD
### <a name="azure-ad-sign-in-configuration-with-azure-ad-connect"></a>Configuración de inicio de sesión de Azure AD con Azure AD Connect
La experiencia de inicio de sesión de Azure AD depende de si este puede hacer coincidir el sufijo del nombre principal del usuario que se esté sincronizando con uno de los dominios personalizados comprobados en el directorio de Azure AD. Azure AD Connect proporciona ayuda durante la configuración del inicio de sesión de Azure AD, para que la experiencia de inicio de sesión de usuario en la nube sea similar a la experiencia local.

Azure AD Connect muestra los sufijos UPN definidos para los dominios y trata de hacerlos coincidir con un dominio personalizado de Azure AD. Después, puede ayudarlo con la acción pertinente que necesite realizar.
La página de inicio de sesión de Azure AD enumera los sufijos UPN definidos para Active Directory local y muestra el estado correspondiente en cada sufijo. Los valores de estado pueden ser uno de los siguientes:

| State | Descripción | Se requiere acción |
|:--- |:--- |:--- |
| Verified |Azure AD Connect ha encontrado un dominio comprobado coincidente en Azure AD. Todos los usuarios de este dominio pueden iniciar sesión con sus credenciales locales. |No se requiere ninguna acción. |
| Not verified (Sin comprobar) |Azure AD Connect encontró una coincidencia de dominio personalizado en Azure AD, pero no está comprobada. Se cambiará el sufijo UPN de los usuarios de este dominio al sufijo predeterminado .onmicrosoft.com después de la sincronización si el dominio no se comprueba. | [Compruebe el dominio personalizado en Azure AD.](../fundamentals/add-custom-domain.md#verify-your-custom-domain-name) |
| Not added (Sin agregar) |Azure AD Connect no encontró un dominio personalizado que se correspondiera con el sufijo UPN. Se cambiará el sufijo UPN de los usuarios de este dominio al sufijo predeterminado .onmicrosoft.com después de la sincronización si el dominio no se agrega y comprueba en Azure. | [Agregue y compruebe un dominio personalizado que se corresponda con el sufijo UPN.](../fundamentals/add-custom-domain.md) |

La página de inicio de sesión de AD Azure muestra los sufijos UPN definidos para Active Directory local y el dominio personalizado correspondiente en Azure AD con el estado actual de comprobación. En una instalación personalizada, ahora puede seleccionar el atributo de nombre principal de usuario en la página de **inicio de sesión en Azure AD** .

![Página de inicio de sesión de AD Azure](./media/plan-connect-user-signin/custom_azure_sign_in.png)

Puede hacer clic en el botón de actualización para volver a capturar el estado más reciente de los dominios personalizados de Azure AD.

### <a name="selecting-the-attribute-for-the-user-principal-name-in-azure-ad"></a>Seleccione el atributo de nombre principal de usuario en Azure AD
userPrincipalName es el atributo que los usuarios utilizan al iniciar sesión en Azure AD y Office 365. Los dominios utilizados, también conocidos como "sufijo UPN", deben comprobarse en Azure AD antes de que se sincronicen los usuarios.

Se recomienda encarecidamente mantener el atributo userPrincipalName predeterminado. Si este atributo no es enrutable y no se puede comprobar, se puede seleccionar otro atributo, por ejemplo, email, como que contiene el identificador de inicio de sesión. Esto se conoce como un "identificador alternativo". El valor del atributo Alternate ID debe seguir el estándar RFC822. Un identificador alternativo puede usarse como solución de inicio de sesión tanto con el SSO de contraseña como con el SSO de federación.

> [!NOTE]
> El uso de un identificador alternativo no es compatible con todas las cargas de trabajo de Office 365. Para obtener más información, consulte [Configuración del identificador de inicio de sesión alternativo](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id).
>
>

#### <a name="different-custom-domain-states-and-their-effect-on-the-azure-sign-in-experience"></a>Diferentes estados de los dominios personalizados y su efecto en el inicio de sesión en Azure
Es muy importante comprender la relación entre los estados de un dominio personalizado en el directorio de Azure AD y los sufijos UPN definidos de forma local. Vamos a revisar las distintas experiencias de inicio de sesión de Azure cuando se configura la sincronización usando Azure AD Connect.

Para la siguiente información, supongamos que estamos interesados en el sufijo UPN "contoso.com" que se usa en el directorio local como parte del UPN, por ejemplo, user@contoso.com.

###### <a name="express-settingspassword-hash-synchronization"></a>Configuración rápida y sincronización de hash de contraseña

| State | Efecto en la experiencia de usuario sobre el inicio de sesión de Azure |
|:---:|:--- |
| Not added (Sin agregar) |En este caso no se ha agregado ningún dominio personalizado para "contoso.com" en el directorio de Azure AD. Los usuarios que tienen el UPN local con el sufijo @contoso.com, no podrán usar su UPN local para iniciar sesión en Azure. En su lugar, tendrán que utilizar un UPN nuevo que les proporcionará Azure AD, agregando el sufijo para el directorio predeterminado de Azure AD. Por ejemplo, si la sincronización de los usuarios es con el directorio de Azure AD azurecontoso.onmicrosoft.com, al usuario local user@contoso.com se le dará un UPN user@azurecontoso.onmicrosoft.com. |
| Not verified (Sin comprobar) |En este caso tenemos un dominio personalizado de "contoso.com" agregado en el directorio de Azure AD, pero que no se ha comprobado. Sin embargo, aún no se ha comprobado. Si continúa con la sincronización de los usuarios sin comprobar el dominio, Azure AD asignará a los usuarios un UPN nuevo igual que lo hizo en el escenario "Sin agregar". |
| Verified |En este caso, tenemos un dominio personalizado de "contoso.com" que ya está agregado y comprobado en Azure AD para el sufijo UPN. Los usuarios podrán utilizar su nombre principal de usuario local, por ejemplo, user@contoso.com, para iniciar sesión en Azure una vez que se hayan sincronizado con Azure AD. |

###### <a name="ad-fs-federation"></a>Federación de AD FS
En Azure AD no se puede crear una federación con el dominio predeterminado ".onmicrosoft.com", ni un dominio personalizado no comprobado. Cuando se ejecuta el Asistente para Azure AD Connect, si selecciona un dominio no comprobado para crear una federación, Azure AD Connect le pedirá que cree los registros necesarios en donde esté hospedado el DNS para el dominio. Para obtener más información, consulte [Comprobación del dominio de Azure AD seleccionado para la federación](how-to-connect-install-custom.md#verify-the-azure-ad-domain-selected-for-federation).

Si seleccionó la opción de inicio de sesión de usuario como **Federación con AD FS**, tiene que tener un dominio personalizado para continuar con la creación de una federación en Azure AD. Para el caso que estamos tratando, esto significa que deberíamos tener un dominio personalizado "contoso.com" agregado en el directorio de Azure AD.

| State | Efecto en la experiencia de usuario sobre el inicio de sesión de Azure |
|:---:|:--- |
| Not added (Sin agregar) |En este caso Azure AD Connect no pudo encontrar un dominio personalizado coincidente para el sufijo UPN "contoso.com" en el directorio de Azure AD. Tiene que agregar un dominio personalizado "contoso.com" si los usuarios necesitan iniciar sesión usando AD FS con su UPN local como user@contoso.com. |
| Not verified (Sin comprobar) |En este caso, Azure AD Connect le indicará los detalles sobre cómo puede comprobar el dominio en una etapa posterior. |
| Verified |En este caso, puede proseguir con la configuración sin realizar ninguna acción. |

## <a name="changing-the-user-sign-in-method"></a>Cambio del método de inicio de sesión de usuario
Puede cambiar el método de inicio de sesión de federación a sincronización de hash de contraseña o a autenticación de paso a través usando las tareas que estarán disponibles en Azure AD Connect después de la configuración inicial de Azure AD Connect realizada con el asistente. Vuelva a ejecutar al Asistente para Azure AD Connect y verá una lista de tareas que puede realizar. Seleccione **Cambiar inicio de sesión de usuario** de la lista de tareas.

![Cambiar inicio de sesión de usuario](./media/plan-connect-user-signin/changeusersignin.png)

En la página siguiente, deberá proporcionar las credenciales de Azure AD.

![Conectarse a Azure](./media/plan-connect-user-signin/changeusersignin2.png)

En la página **Inicio de sesión de usuario**, seleccione el inicio de sesión de usuario deseado.

![Conectarse a Azure](./media/plan-connect-user-signin/changeusersignin2a.png)

> [!NOTE]
> Si el cambio a sincronización de hash de contraseña es solo temporal, active la casilla **No convertir las cuentas de usuario**. Si no lo hace, se realizará la conversión de cada usuario a federado lo que puede llevar varias horas.
>
>

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre la [integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).
- Más información sobre los [conceptos de diseño de Azure AD Connect](plan-connect-design-concepts.md)