---
title: Consideraciones de implementación para el autoservicio de restablecimiento de contraseña de Azure Active Directory
description: Obtenga información sobre las consideraciones y la estrategia para la implementación correcta del autoservicio de restablecimiento de contraseña de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: baselden
author: barbaraselden
manager: daveba
ms.reviewer: sahenry
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c9f59c7bb9b3977bdff7e3fd1ab78ed6fa2e412
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717751"
---
# <a name="plan-an-azure-active-directory-self-service-password-reset-deployment"></a>Plan de implementación de autoservicio de restablecimiento de contraseña de Azure Active Directory

> [!IMPORTANT]
> Este plan de implementación ofrece instrucciones y procedimientos recomendados para implementar el autoservicio de restablecimiento de contraseña (SSPR) de Azure AD.
>
> **Si es un usuario final y necesita volver a su cuenta, vaya a [https://aka.ms/sspr](https://aka.ms/sspr)** .

El [autoservicio de restablecimiento de contraseña (SSPR)](https://www.youtube.com/watch?v=tnb2Qf4hTP8) es una característica de Azure Active Directory (AD) que permite a los usuarios restablecer sus contraseñas sin ponerse en contacto con el personal de TI. Los usuarios se pueden desbloquear rápidamente ellos mismos y continuar trabajando con independencia de dónde se encuentren o la hora del día. Al permitir que los empleados se desbloqueen ellos mismos, su organización puede reducir el tiempo no productivo y los altos costes de soporte técnico para los problemas más comunes relacionados con las contraseñas.

SSPR ofrece las siguientes funcionalidades clave:

* El autoservicio permite a los usuarios finales restablecer sus contraseñas expiradas o no expiradas sin necesidad de ponerse en contacto con un administrador o el departamento de soporte técnico.
* La [escritura diferida de contraseñas](./concept-sspr-writeback.md) permite la administración de contraseñas locales y la resolución de bloqueos de cuenta a través de la nube.
* Los informes de actividad de administración de contraseñas proporcionan a los administradores información sobre una actividad de registro y restablecimiento de contraseña en su organización.

En esta guía de implementación se muestra cómo planear y probar una implementación de SSPR.

Para ver rápidamente SSPR en acción y, a continuación, volver para conocer las consideraciones de implementación adicionales:

> [!div class="nextstepaction"]
> [Habilitar el autoservicio de restablecimiento de contraseña (SSPR)](tutorial-enable-sspr.md)

## <a name="learn-about-sspr"></a>Más información acerca de SSPR

Más información acerca de SSPR. Consulte [Funcionamiento: Autoservicio de restablecimiento de contraseña de Azure AD](./concept-sspr-howitworks.md).

### <a name="key-benefits"></a>Ventajas principales

Las ventajas clave de habilitar SSPR son:

* **Administración del coste**. SSPR reduce los costes de soporte técnico de TI al permitir que los usuarios restablezcan las contraseñas por sí mismos. También reduce el coste de tiempo perdido debido a la pérdida de contraseñas y bloqueos. 

* **Experiencia de usuario intuitiva**. Proporciona un proceso intuitivo único de registro de usuarios que permite a los usuarios restablecer contraseñas y desbloquear cuentas a petición desde cualquier dispositivo o ubicación. SSPR permite a los usuarios volver a trabajar más rápidamente y ser más productivos.

* **Flexibilidad y seguridad**. SSPR permite a las empresas acceder a la seguridad y la flexibilidad que proporciona una plataforma en la nube. Los administradores pueden cambiar la configuración para adaptarse a los nuevos requisitos de seguridad y aplicar los cambios a los usuarios sin interrumpir su inicio de sesión.

* **Auditoría y seguimiento del uso sólidos**. Una organización puede asegurarse de que los sistemas empresariales permanecen seguros mientras sus usuarios restablecen sus propias contraseñas. Los registros de auditoría sólidos incluyen información de cada paso del proceso de restablecimiento de contraseña. Estos registros también están disponibles desde una API y permiten al usuario importar estos datos en el sistema de Administración de eventos e información de seguridad (SIEM) de su elección.

### <a name="licensing"></a>Licencias

Las licencias de Azure Active Directory se otorgan por usuario; es decir, cada usuario requiere una licencia adecuada para las características que usa. Se recomiendan las licencias basadas en grupos para SSPR. 

Para comparar las ediciones y las características y habilitar las licencias basadas en grupos o usuarios, consulte [Requisitos de concesión de licencias del autoservicio de restablecimiento de contraseña de Azure AD](./concept-sspr-licensing.md).

Para obtener más información sobre los precios, consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="prerequisites"></a>Requisitos previos

* Un inquilino de Azure AD activo con al menos una licencia de prueba habilitada. Si es preciso, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Una cuenta con privilegios de administrador global.


### <a name="training-resources"></a>Recursos de aprendizaje

| Recursos| Vínculo y descripción |
| - | - |
| Vídeos| [Capacite a sus usuarios con una mejor escalabilidad de TI](https://youtu.be/g9RpRnylxS8) 
| |[¿Qué es el autoservicio de restablecimiento de contraseña?](https://youtu.be/hc97Yx5PJiM)|
| |[Implementación del autoservicio de restablecimiento de contraseña](https://www.youtube.com/watch?v=Pa0eyqjEjvQ&index=18&list=PLLasX02E8BPBm1xNMRdvP6GtA6otQUqp0)|
| |[¿Cómo se configura el autoservicio de restablecimiento de contraseña para los usuarios de Azure AD?](https://azure.microsoft.com/resources/videos/self-service-password-reset-azure-ad/) |
| |[Cómo [preparar a los usuarios para] registrar [su] información de seguridad para Azure Active Directory](https://youtu.be/gXuh0XS18wA) |
| Cursos en línea|[Administración de identidades en Microsoft Azure Active Directory](https://www.pluralsight.com/courses/microsoft-azure-active-directory-managing-identities) Use SSPR para ofrecer a los usuarios una experiencia moderna y protegida. Consulte especialmente el módulo "[Administración de usuarios y grupos de Azure Active Directory](https://app.pluralsight.com/library/courses/microsoft-azure-active-directory-managing-identities/table-of-contents)". |
|Cursos de pago de Pluralsight |[Problemas de la administración de identidades y acceso](https://www.pluralsight.com/courses/identity-access-management-issues) Obtenga información IAM y los problemas de seguridad que deben tenerse en cuenta en su organización. Consulte especialmente el módulo "Otros métodos de autenticación".|
| |[Introducción a Microsoft Enterprise Mobility Suite](https://www.pluralsight.com/courses/microsoft-enterprise-mobility-suite-getting-started) Conozca los procedimientos recomendados para ampliar los recursos locales en la nube de forma que se permita la autenticación, la autorización, el cifrado y una experiencia móvil segura. Consulte especialmente el módulo "Configuración de características avanzadas de Microsoft Azure Active Directory Premium".
|Tutoriales |[Realización de una implementación piloto del autoservicio de restablecimiento de contraseñas en Azure AD](./tutorial-enable-sspr.md) |
| |[Habilitación de la escritura diferida de contraseñas](./tutorial-enable-sspr-writeback.md) |
| |[Restablecimiento de la contraseña de Azure AD desde la pantalla de inicio de sesión de Windows 10](./howto-sspr-windows.md) |
| Preguntas más frecuentes|[Preguntas más frecuentes sobre la administración de contraseñas](./active-directory-passwords-faq.md) |


### <a name="solution-architecture"></a>Arquitectura de la solución

En el ejemplo siguiente se describe la arquitectura de la solución de restablecimiento de contraseña para entornos híbridos habituales.

![diagrama de la arquitectura de la solución](./media/howto-sspr-deployment//solutions-architecture.png)

Descripción del flujo de trabajo

Para restablecer la contraseña, los usuarios van al [portal de restablecimiento de contraseña](https://aka.ms/sspr). Deben verificar su método o métodos de autenticación registrados anteriormente para demostrar su identidad. Si restablecen correctamente la contraseña, comenzará el proceso de restablecimiento.

* En el caso de los usuarios que solo están en la nube, SSPR almacena la nueva contraseña en Azure AD. 

* Para los usuarios híbridos, SSPR escribe la contraseña en diferido en la instancia local de Active Directory a través del servicio de Azure AD Connect. 

Nota: En el caso de los usuarios que tienen la [sincronización de hash de contraseñas (PHS)](../hybrid/whatis-phs.md) deshabilitada, SSPR solo almacena las contraseñas localmente en Active Directory.

### <a name="best-practices"></a>Procedimientos recomendados

Puede ayudar a los usuarios a registrarse rápidamente mediante la implementación de SSPR junto con otra aplicación o servicio popular en la organización. Esta acción generará un gran número de inicios de sesión e impulsará el registro.

Antes de implementar SSPR, puede optar por determinar el número y el coste medio de cada llamada de restablecimiento de contraseña. Puede usar estos datos después de la implementación para mostrar el valor que SSPR aporta a su organización.

#### <a name="enable-combined-registration-for-sspr-and-mfa"></a>Habilitar el registro combinado en SSPR y MFA

Microsoft recomienda que las organizaciones habiliten la experiencia de registro combinado para SSPR y autenticación multifactor. Al habilitar esta experiencia de registro combinado, los usuarios solo tienen que seleccionar su información de registro una vez para habilitar ambas características.

La experiencia de registro combinado no requiere que las organizaciones habiliten tanto SSPR como Azure Multi-factor Authentication. El registro combinado proporciona a las organizaciones una mejor experiencia de usuario. Para más información, consulte [Registro de información de seguridad combinado](concept-registration-mfa-sspr-combined.md).

## <a name="plan-the-deployment-project"></a>Planeamiento del proyecto de implementación

Tenga en cuenta las necesidades de su organización al determinar la estrategia de esta implementación en su entorno.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de involucrar a las partes interesadas correctas](https://aka.ms/deploymentplans) y que los roles de las partes interesadas en el proyecto se entiendan bien; para ello, documente las partes interesadas y sus aportes y responsabilidades en el proyecto.

#### <a name="required-administrator-roles"></a>Roles de administrador necesarios


| Rol de negocio| Rol de Azure AD (si es necesario) |
| - | - |
| Departamento de soporte técnico de nivel 1| Administrador de contraseñas |
| Departamento de soporte técnico de nivel 2| Administrador de usuarios |
| Administrador de SSPR| Administrador global |


### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Debería comunicar de forma proactiva a los usuarios cómo y cuándo va a cambiar su experiencia, y cómo obtener soporte técnico si tienen cualquier problema. Revise los [materiales de lanzamiento del autoservicio de restablecimiento de contraseña en el Centro de descargas de Microsoft](https://www.microsoft.com/download/details.aspx?id=56768) para obtener ideas sobre cómo planear su estrategia de comunicación con los usuarios finales.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Se recomienda que la configuración inicial de SSPR esté en un entorno de prueba. Comience con un grupo piloto habilitando SSPR para un subconjunto de usuarios de la organización. Consulte [Procedimientos recomendados para un piloto](../fundamentals/active-directory-deployment-plans.md).

Para crear un grupo, consulte [Creación de un grupo y adición de miembros en Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md). 

## <a name="plan-configuration"></a>Planeamiento de la configuración

La siguiente configuración es necesaria para habilitar SSPR junto con los valores recomendados.

| Área | Configuración | Value |
| --- | --- | --- |
| **Propiedades de SSPR** | Se habilitó el autoservicio de restablecimiento de contraseña | Grupo **Seleccionado** para piloto/**Todos** para producción |
| **Métodos de autenticación** | Authentication methods required to register (Métodos de autenticación requeridos para registrarse) | Siempre 1 más de los necesarios para el restablecimiento |
|   | Authentication methods required to reset (Métodos de autenticación requeridos para restablecer) | Uno o dos |
| **Registro** | Exigir a los usuarios que se registren al iniciar sesión | Sí |
|   | Número de días que pasan hasta que se pide a los usuarios que vuelvan a confirmar su información de autenticación | De 90 a 180 días |
| **Notificaciones** | ¿Quiere notificar a los usuarios los restablecimientos de contraseña? | Sí |
|   | ¿Quiere notificar a todos los administradores cuando otros administradores restablezcan su contraseña? | Sí |
| **Personalización** | Personalizar el vínculo del departamento de soporte técnico | Sí |
|   | Dirección URL o correo electrónico del departamento de soporte técnico personalizados | Dirección de correo electrónico o sitio de soporte técnico |
| **Integración local** | Escritura diferida de contraseñas al Active Directory local | Sí |
|   | Allow users to unlock account without resetting password (Permitir a los usuarios desbloquear las cuentas sin restablecer la contraseña) | Sí |

### <a name="sspr-properties"></a>Propiedades de SSPR

Al habilitar SSPR, elija un grupo de seguridad adecuado en el entorno piloto.

* Para exigir el registro de SSPR a todos los usuarios, se recomienda usar la opción **Todos**.
* En caso contrario, seleccione el grupo de seguridad de Azure AD o AD adecuado.

### <a name="authentication-methods"></a>Métodos de autenticación

Cuando SSPR está habilitado, los usuarios solo pueden restablecer su contraseña si tienen datos en los métodos de autenticación que el administrador haya habilitado. Los métodos incluyen el teléfono, una notificación de la aplicación Authenticator, preguntas de seguridad, etc. Para más información, consulte [¿Qué son los métodos de autenticación?](./concept-authentication-methods.md).

Se recomienda la siguiente configuración de método de autenticación:

* Establezca **Authentication methods required to register** (Métodos de autenticación requeridos para registrarse) en al menos uno más que el número necesario para el restablecimiento. Permitir varias autenticaciones ofrece a los usuarios flexibilidad cuando necesiten restablecer la contraseña.

* Establezca **Número de métodos requeridos para el restablecimiento** en un nivel apropiado para su organización. Uno requiere la menor fricción, mientras que Dos puede aumentar la posición de seguridad. 

Nota: El usuario debe tener configurados los métodos de autenticación en las [Restricciones y directivas de contraseñas en Azure Active Directory](./concept-sspr-policy.md).

### <a name="registration-settings"></a>Configuración de registro

Establezca **Exigir a los usuarios que se registren al iniciar sesión** en **Sí**. Esta configuración requiere que los usuarios se registren cuando inician sesión, lo que garantiza que todos los usuarios estén protegidos.

Establezca **Número de días que pasan hasta que se pide a los usuarios que vuelvan a confirmar su información de autenticación** entre **90** y **180** días, a menos que su organización tenga una necesidad de negocio para un plazo de tiempo más corto.

### <a name="notifications-settings"></a>Configuración de notificaciones

Configure tanto **¿Quiere notificar a los usuarios los restablecimientos de contraseña?** como **¿Quiere notificar a todos los administradores cuando otros administradores restablezcan su contraseña?** en **Sí**. Al seleccionar **Sí** en ambos se aumenta la seguridad asegurándose de que los usuarios son conscientes de que se restablece la contraseña. También garantiza que todos los administradores son conscientes de que un administrador cambia una contraseña. Si los usuarios o administradores reciben una notificación y no han comenzado el cambio, pueden denunciar de inmediato un problema de seguridad potencial.

### <a name="customization-settings"></a>Configuración de personalización

Es fundamental personalizar el correo electrónico o la dirección URL del departamento de soporte técnico para asegurarse de que los usuarios que tengan problemas puedan obtener ayuda inmediatamente. Establezca esta opción en una dirección de correo electrónico o página web de soporte técnico comunes con los que los usuarios estén familiarizados. 

Para más información, consulte [Personalizar la funcionalidad del autoservicio de restablecimiento de contraseña de Azure AD](./howto-sspr-customization.md).

### <a name="password-writeback"></a>Escritura diferida de contraseñas

La [escritura diferida de contraseñas](../hybrid/whatis-hybrid-identity.md) se habilita con **Azure AD Connect** y escribe en diferido los restablecimientos de contraseña en la nube en un directorio local existente en tiempo real. Para más información, consulte [¿Qué es la escritura diferida de contraseñas?](./concept-sspr-writeback.md)

Se recomienda la configuración siguiente:

* Asegúrese de que **Escritura diferida de contraseñas al Active Directory local** esté establecido en **Sí**. 
* Establezca **Allow users to unlock account without resetting password** (Permitir a los usuarios desbloquear las cuentas sin restablecer la contraseña) en **Sí**.

De forma predeterminada, Azure AD desbloquea las cuentas cuando se realiza un restablecimiento de contraseña.

### <a name="administrator-password-setting"></a>Configuración de la contraseña de administrador

Las cuentas de administrador tienen permisos elevados. Los administradores empresariales o de dominios locales no pueden restablecer su contraseña mediante SSPR. Las cuentas de administrador locales tienen las siguientes restricciones:

* Solo pueden cambiar su contraseña en su entorno local.
* Nunca pueden usar las preguntas y respuestas secretas como método para restablecer su contraseña.

Se recomienda no sincronizar las cuentas de administrador locales de Active Directory con Azure AD.

### <a name="environments-with-multiple-identity-management-systems"></a>Entornos con varios sistemas de administración de identidades

Algunos entornos tienen varios sistemas de administración de identidades. Los administradores de identidades locales, como Oracle AM y SiteMinder, requieren la sincronización con AD para las contraseñas. Para ello, puede usar una herramienta como el servicio de notificación de cambio de contraseña (PCNS) con Microsoft Identity Manager (MIM). Para información sobre el escenario más complejo, consulte el artículo [Implementación del servicio de notificación de cambio de contraseña de MIM en un controlador de dominio](/microsoft-identity-manager/deploying-mim-password-change-notification-service-on-domain-controller).

## <a name="plan-testing-and-support"></a>Planeamiento de pruebas y soporte técnico

En cada fase de la implementación, desde los grupos piloto iniciales hasta toda la organización, asegúrese de que los resultados son los esperados.

### <a name="plan-testing"></a>Planeamiento de pruebas

Para asegurarse de que la implementación funciona según lo previsto, planee un conjunto de casos de prueba para validar la implementación. Para evaluar los casos de prueba, necesita un usuario de prueba que no sea administrador con una contraseña. Si necesita crear un usuario, consulte [Adición de nuevos usuarios a Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).

En la tabla siguiente se incluyen escenarios de prueba útiles que puede usar para documentar los resultados esperados por la organización según las directivas.
<br>


| Oportunidad de negocio| Resultados esperados |
| - | - |
| Se puede acceder al portal de SSPR desde la red corporativa| Determinado por la organización |
| Se puede acceder al portal de SSPR desde fuera de la red corporativa| Determinado por la organización |
| Restablecer la contraseña del usuario desde el explorador cuando el usuario no está habilitado para el restablecimiento de contraseña| El usuario no puede acceder al flujo de restablecimiento de contraseña |
| Restablecer la contraseña del usuario desde el explorador cuando el usuario no se ha registrado para el restablecimiento de contraseña| El usuario no puede acceder al flujo de restablecimiento de contraseña |
| El usuario inicia sesión cuando se exige el registro de restablecimiento de contraseña| Se solicita al usuario que registre información de seguridad |
| El usuario inicia sesión cuando se completa el registro de restablecimiento de contraseña| Se solicita al usuario que registre información de seguridad |
| Se puede acceder al portal de SSPR cuando el usuario no tiene una licencia| Es accesible |
| Restablecer la contraseña de usuario desde la pantalla de bloqueo de un dispositivo Windows 10 unido a Azure AD o Azure AD híbrido| El usuario puede restablecer la contraseña |
| Los datos de uso y de registro de SSPR están disponibles para los administradores casi en tiempo real| Están disponibles a través de los registros de auditoría |

También puede consultar [Realización de una implementación piloto del autoservicio de restablecimiento de contraseñas en Azure AD](./tutorial-enable-sspr.md). En este tutorial, habilitará una implementación piloto de SSPR de Azure AD en la organización y la probará con una cuenta sin privilegios de administrador.

### <a name="plan-support"></a>Planeamiento de soporte técnico

Aunque SSPR no suele crear problemas de usuario, es importante preparar al personal de soporte técnico para afrontar los problemas que puedan surgir. Si bien un administrador puede restablecer la contraseña para los usuarios finales a través del portal de Azure AD, es mejor ayudar a resolver el problema a través de un proceso de soporte técnico de autoservicio.

Para allanar el camino del equipo de soporte técnico, puede crear una página de P+F basada en las preguntas que reciba de los usuarios. Estos son algunos ejemplos:

| Escenarios| Descripción |
| - | - |
| El usuario no tiene disponible ningún método de autenticación registrado.| Un usuario intenta restablecer la contraseña, pero no tiene disponible ninguno de los métodos de autenticación que ha registrado (ejemplo: se ha dejado el teléfono móvil en casa y no puede acceder al correo electrónico). |
| El usuario no recibe mensajes de texto ni llamadas en su teléfono móvil o laboral.| Un usuario intenta verificar su identidad mediante mensaje de texto o una llamada, pero no recibe ni un mensaje de texto ni una llamada. |
| El usuario no puede acceder al portal de restablecimiento de contraseñas.| Un usuario quiere restablecer su contraseña, pero no está habilitado para restablecer la contraseña y no puede acceder a la página para actualizar las contraseñas. |
| El usuario no puede establecer una contraseña nueva.| Un usuario finaliza la verificación durante el flujo de restablecimiento de contraseña, pero no puede establecer una contraseña nueva. |
| El usuario no ve un vínculo de restablecimiento de contraseña en un dispositivo Windows 10.| Un usuario intenta restablecer la contraseña desde la pantalla de bloqueo de Windows 10, pero el dispositivo no está unido a Azure AD o no está habilitada la directiva de dispositivos de Intune. |

### <a name="plan-rollback"></a>Planeamiento de la reversión

Para revertir la implementación:

* Para un solo usuario, quite el usuario del grupo de seguridad. 

* Para un grupo, quite el grupo de la configuración de SSPR.

* Para todos los usuarios, deshabilite SSPR para el inquilino de Azure AD.

## <a name="deploy-sspr"></a>Implementación de SSPR

Antes de la implementación, asegúrese de haber realizado lo siguiente:

1. Creó y comenzó a ejecutar el [plan de comunicación](#plan-communications).

1. Determinó los [parámetros de configuración](#plan-configuration) adecuados.

1. Identificó los usuarios y los grupos para los entornos [piloto](#plan-a-pilot) y de producción.

1. [Determinó los parámetros de configuración](#plan-configuration) para el registro y el autoservicio.

1. [Configuró la escritura diferida de contraseñas](#password-writeback) si tiene un entorno híbrido.


**¡Ahora ya puede realizar la implementación de SSPR!**

Consulte [Habilitar el autoservicio de restablecimiento de contraseña](./tutorial-enable-sspr.md#enable-self-service-password-reset) para obtener instrucciones paso a paso sobre la configuración de las áreas siguientes.

1. [Métodos de autenticación](./concept-authentication-methods.md)

1. [Configuración de registro](./concept-registration-mfa-sspr-combined.md)

1. [Configuración de notificaciones](#notifications-settings)

1. [Configuración de personalización](./howto-sspr-customization.md)

1. [Integración local](./tutorial-enable-sspr-writeback.md)

### <a name="enable-sspr-in-windows"></a>Habilitar SSPR en Windows
En el caso de los equipos que ejecutan Windows 7, 8, 8.1 y 10, puede [permitir que los usuarios restablezcan su contraseña en la pantalla de inicio de sesión de Windows](./howto-sspr-windows.md).

## <a name="manage-sspr"></a>Administrar SSPR

Azure AD puede proporcionar información adicional sobre el rendimiento de SSPR a través de auditorías e informes.

### <a name="password-management-activity-reports"></a>Informes de actividad de administración de contraseñas 

Puede usar informes pregenerados en Azure Portal para medir el rendimiento de SSPR. Si está debidamente protegido por licencia, también puede crear consultas personalizadas. Para más información, consulte [Opciones de creación de informes para la administración de contraseñas de Azure AD](./howto-sspr-reporting.md).

> [!NOTE]
>  Debe ser [administrador global](../users-groups-roles/directory-assign-admin-roles.md) y habilitar la opción para que estos datos se recopilen para su organización. Para habilitarla, debe visitar la pestaña Informes o los registros de auditoría en Azure Portal como mínimo una vez. Hasta entonces, los datos no se recopilan para su organización.

Los registros de auditoría para el registro y el restablecimiento de contraseña están disponibles durante 30 días. Si la auditoría de seguridad dentro su una empresa requiere una retención más prolongada, los registros deben exportarse y consumirse en una herramienta SIEM, como [Azure Sentinel](../../sentinel/connect-azure-active-directory.md), Splunk o ArcSight.

![Captura de pantalla de informes de SSPR](./media/howto-sspr-deployment/sspr-reporting.png)

### <a name="authentication-methods--usage-and-insights"></a>Métodos de autenticación: uso y conclusiones

[El uso y las conclusiones](./howto-authentication-methods-usage-insights.md) le permiten comprender cómo funcionan en su organización los métodos de autenticación para características como Azure MFA y SSPR. Esta funcionalidad de informes proporciona a la organización los medios para comprender qué métodos se registran y cómo usarlos.

### <a name="troubleshoot"></a>Solución de problemas

* Consulte [Solución de problemas del autoservicio de restablecimiento de contraseñas](./active-directory-passwords-troubleshoot.md) 

* Siga [Preguntas más frecuentes sobre la administración de contraseñas](./active-directory-passwords-faq.md) 

### <a name="helpful-documentation"></a>Documentación útil

* [¿Qué son los métodos de autenticación?](./concept-authentication-methods.md)

* [Funcionamiento: Autoservicio de restablecimiento de contraseña de Azure AD](./concept-sspr-howitworks.md)

* [Personalizar la funcionalidad del autoservicio de restablecimiento de contraseña de Azure AD](./howto-sspr-customization.md)

* [Restricciones y directivas de contraseñas en Azure Active Directory](./concept-sspr-policy.md)

* [¿Qué es la escritura diferida de contraseñas?](./concept-sspr-writeback.md)

## <a name="next-steps"></a>Pasos siguientes

* Para empezar con la implementación de SSPR, consulte [Habilitación del autoservicio de restablecimiento de contraseña de Azure AD](tutorial-enable-sspr.md)

* [Implementar la protección de contraseña de Azure AD](./concept-password-ban-bad.md)

* [Implementar el bloqueo inteligente de Azure AD](./howto-password-smart-lockout.md)