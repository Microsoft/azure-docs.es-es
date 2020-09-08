---
title: Planeamiento de la implementación del acceso condicional de Azure Active Directory
description: Obtenga información sobre cómo diseñar directivas de acceso condicional e implementarlas de forma eficaz en la organización.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 06/09/2020
ms.author: baselden
author: BarbaraSelden
manager: daveba
ms.reviewer: joflore
ms.collection: M365-identity-device-management
ms.openlocfilehash: 14612247d241aed420a76ff40e19d04164cbe150
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89049934"
---
# <a name="plan-a-conditional-access-deployment"></a>Planeamiento de la implementación del acceso condicional

El planeamiento de la implementación del acceso condicional es fundamental para conseguir la estrategia de acceso para las aplicaciones y los recursos de su organización.

En un mundo que da prioridad a los dispositivos móviles y la nube, los usuarios tienen acceso a los recursos de su organización desde cualquier parte mediante diversos dispositivos y aplicaciones. Como resultado, ya no es suficiente con centrarse en quién puede acceder a un recurso. También debe tener en cuenta dónde se encuentra el usuario, el dispositivo en uso, el recurso al que se tiene acceso, etc. 

El acceso condicional (CA) de Azure Active Directory (Azure AD) analiza señales como el usuario, el dispositivo y la ubicación para automatizar las decisiones y aplicar las directivas de acceso de la organización para el recurso. Puede usar directivas de CA para aplicar controles de acceso como Multi-Factor Authentication (MFA). Las directivas de CA le permiten solicitar a los usuarios MFA cuando sea necesario para la seguridad y dejarles cuando no lo sea.

![Introducción al acceso condicional](./media/plan-conditional-access/conditional-access-overview-how-it-works.png)

Microsoft proporciona directivas condicionales estándar llamadas [valores predeterminados de seguridad](../fundamentals/concept-fundamentals-security-defaults.md) que garantizan un nivel básico de seguridad. Sin embargo, su organización puede necesitar más flexibilidad de la que ofrecen los valores predeterminados de seguridad. Puede usar el acceso condicional para personalizar los valores predeterminados de seguridad con mayor granularidad y configurar nuevas directivas que cumplan sus requisitos.

## <a name="learn"></a>Obtener información

Antes de comenzar, asegúrese de que comprende cómo funciona el [acceso condicional](overview.md) y cuándo se debe usar.

### <a name="benefits"></a>Ventajas

Las ventajas de implementar el acceso condicional son:

* Aumento de la productividad. Interrumpa solo a los usuarios con una condición de inicio de sesión como MFA cuando una o varias señales lo garanticen. Las directivas de CA le permiten controlar cuándo se solicita a los usuarios MFA, cuándo se bloquea el acceso y cuándo deben usar un dispositivo de confianza.

* Administración del riesgo. La automatización de la evaluación de riesgos con condiciones de directiva significa que los inicios de sesión de riesgo se identifican y corrigen o bloquean. El acoplamiento del acceso condicional con [Identity Protection](../identity-protection/overview-identity-protection.md), que detecta anomalías y eventos sospechosos, le permite establecer un destino cuando el acceso a los recursos se bloquea o se valida. 

* Control de cumplimiento y gobernanza. El acceso condicional le permite auditar el acceso a las aplicaciones, presentar términos de uso para su consentimiento y restringir el acceso en función de las directivas de cumplimiento.

* Administración del coste. Mover las directivas de acceso a Azure AD reduce la dependencia de las soluciones personalizadas o locales para el acceso condicional y sus costos de infraestructura.

### <a name="license-requirements"></a>Requisitos de licencia

Consulte los [requisitos de licencia de acceso condicional](overview.md).

Si se necesitan otras características, también necesitará las licencias asociadas. Para más información, consulte los [precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="prerequisites"></a>Requisitos previos

* Un inquilino de Azure AD activo con Azure AD Premium o una licencia de prueba habilitada. Si es preciso, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Una cuenta con privilegios de administrador global de acceso condicional.

* Un usuario que no sea administrador con una contraseña que conozca, como usuarioDePrueba. Si necesita crear un usuario, consulte [Inicio rápido: Incorporación de nuevos usuarios a Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).

* Un grupo del que sea miembro el usuario que no es administrador. Si necesita crear un grupo, consulte [Creación de un grupo y adición de miembros en Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md).

### <a name="training-resources"></a>Recursos de aprendizaje

Los siguientes recursos pueden ser útiles a medida que obtiene información sobre el acceso condicional:


#### <a name="videos"></a>Vídeos

* [¿Qué es el acceso condicional?](https://youtu.be/ffMAw2IVO7A)
* [¿Cómo se implementa el acceso condicional?](https://youtu.be/c_izIRNJNuk)
* [¿Cómo se implementan las directivas de CA en los usuarios finales?](https://youtu.be/0_Fze7Zpyvc)
* [Acceso condicional con controles de dispositivos](https://youtu.be/NcONUf-jeS4)
* [Acceso condicional con Azure MFA](https://youtu.be/Tbc-SU97G-w)
* [Acceso condicional en Enterprise Mobility + Security](https://youtu.be/A7IrxAH87wc)


#### <a name="online-courses-on-pluralsight"></a>Cursos en línea en Pluralsight

* [Design Identity Management in Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-identity-management-design) (Diseño de la administración de identidades en Microsoft Azure)
* [Design Authentication for Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-authentication-design) (Diseño de la autenticación en Microsoft Azure)
* [Design Authorization for Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-authorization-design) (Diseño de la autorización en Microsoft Azure)

## <a name="plan-the-deployment-project"></a>Planeamiento del proyecto de implementación

Tenga en cuenta las necesidades de su organización al determinar la estrategia de esta implementación en su entorno.

### <a name="engage-the-right-stakeholders"></a>Interactuar con las partes interesadas adecuadas

Cuando fracasan los proyectos tecnológicos, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de que interactúa con las partes interesadas adecuadas](https://aka.ms/deploymentplans) y que los roles del proyecto están claros.

### <a name="plan-communications"></a>Planeamiento de las comunicaciones

La comunicación es fundamental para el éxito de cualquier servicio nuevo. Comunique de forma proactiva a los usuarios cómo y cuándo va a cambiar su experiencia, y cómo obtener soporte técnico si tienen cualquier problema.

### <a name="plan-a-pilot"></a>Planeamiento de un piloto

Cuando las nuevas directivas estén listas para su entorno, impleméntelas en fases en el entorno de producción. En primer lugar, aplique una directiva a un pequeño conjunto de usuarios en un entorno de prueba y compruebe si la directiva se comporta según lo previsto. Consulte [Procedimientos recomendados para un piloto](../fundamentals/active-directory-deployment-plans.md).

> [!NOTE]
> Para implementar nuevas directivas no específicas en los administradores, excluya todos los administradores. Esto garantiza que los administradores puedan seguir teniendo acceso a la directiva y que realicen cambios o la revoquen si hay un impacto significativo. Valide siempre la directiva con grupos de usuarios más pequeños antes de aplicarla a todos los usuarios.

## <a name="understand-ca-policy-components"></a>Descripción de los componentes de la directiva de CA
Las directivas de CA son instrucciones if-then: si se cumple una asignación, aplique estos controles de acceso.

Al configurar las directivas de CA, las condiciones se denominan *asignaciones*. Las directivas de CA permiten exigir controles de acceso en las aplicaciones de la organización según determinadas condiciones.


Para obtener más información, consulte [Creación de una directiva de acceso condicional](concept-conditional-access-policies.md).

![pantalla de la creación de la directiva](media/plan-conditional-access/create-policy.png)

Las [asignaciones](concept-conditional-access-policies.md#assignments) definen lo siguiente:

* Los [usuarios y grupos](concept-conditional-access-users-groups.md) que se van a ver afectados por la directiva

* Las [aplicaciones o acciones en la nube](concept-conditional-access-cloud-apps.md) a las que se les va a aplicar la directiva 

* Las [condiciones](concept-conditional-access-conditions.md) en las que se aplicará la directiva

La configuración de [controles de acceso](concept-conditional-access-policies.md) determina cómo aplicar una directiva:

* [Conceda o bloquee](concept-conditional-access-grant.md) el acceso a las aplicaciones en la nube.

* Los [controles de sesión](concept-conditional-access-session.md) permiten limitar las experiencias desde aplicaciones en la nube específicas.

### <a name="ask-the-right-questions-to-build-your-policies"></a>Formular las preguntas adecuadas para crear sus directivas

Las directivas responden a preguntas sobre quién debe tener acceso a los recursos, a qué recursos deben tener acceso y en qué condiciones. Las directivas se pueden diseñar para conceder acceso, o bien para bloquearlo. Asegúrese de formular las preguntas adecuadas sobre lo que está tratando de lograr su directiva. 

Documente las respuestas a las preguntas de cada directiva antes de su creación. 

#### <a name="common-questions-about-assignments"></a>Preguntas habituales sobre las asignaciones

[Usuarios y grupos](concept-conditional-access-users-groups.md)

* ¿Qué usuarios y grupos se incluirán o se excluirán de la directiva?

* ¿Incluye esta directiva a todos los usuarios, grupos específicos de usuarios, roles de directorio o usuarios externos?

[Aplicaciones o acciones en la nube](concept-conditional-access-cloud-apps.md)

* ¿A qué aplicaciones se aplicará la directiva?

* ¿Qué acciones del usuario estarán sujetas a esta directiva?

[Condiciones](concept-conditional-access-conditions.md)

* ¿Qué plataformas de dispositivo se incluirán o se excluirán de la directiva?

* ¿Cuáles son las ubicaciones de confianza de la organización?

* ¿Qué ubicaciones se incluirán o excluirán de la directiva?

* ¿Qué tipos de aplicación cliente (explorador, móvil, clientes de escritorio o aplicaciones con métodos de autenticación heredados) se incluirán o excluirán de la directiva?

* ¿Tiene directivas que impulsarían la exclusión de dispositivos unidos a Azure AD o dispositivos híbridos unidos a Azure AD de las directivas? 

* Si usa [Identity Protection](../identity-protection/concept-identity-protection-risks.md), ¿desea incorporar la protección frente a riesgos de inicio de sesión?

#### <a name="common-questions-about-access-controls"></a>Preguntas habituales sobre los controles de acceso

[Conceder o bloquear ](concept-conditional-access-grant.md) 

¿Desea conceder acceso a los recursos requiriendo una o varias de las siguientes condiciones?

* Requerir MFA

* Requerir que el dispositivo esté marcado como compatible

* Requerir un dispositivo unido a Azure AD híbrido

* Requerir aplicación cliente aprobada

* Requerir la directiva de protección de aplicaciones

[Control de sesión](concept-conditional-access-session.md)

¿Desea aplicar cualquiera de los siguientes controles de acceso en las aplicaciones en la nube?

* Utilizar permisos que exige la aplicación

* Utilizar el Control de aplicaciones de acceso condicional

* Aplicar la frecuencia de inicio de sesión

* Utilizar sesiones del explorador persistentes

### <a name="access-token-issuance"></a>Emisión de tokens de acceso

Es importante comprender cómo se emiten los tokens de acceso. 

![Diagrama de emisión de tokens de acceso](media/plan-conditional-access/CA-policy-token-issuance.png)

> [!NOTE]
> Si no se requiere ninguna asignación y no hay ninguna directiva de CA en vigor, el comportamiento predeterminado será emitir un token de acceso. 

Por ejemplo, considere una directiva en la que:

SI el usuario está en el Grupo 1, fuerce MFA para tener acceso a la Aplicación 1.

Si un usuario que no está en el Grupo 1 intenta tener acceso a la aplicación, no se cumplirá ninguna condición "if" y se emitirá un token. Para excluir a los usuarios del Grupo 1 se requiere una directiva independiente para bloquear a todos los demás usuarios.

## <a name="follow-best-practices"></a>Seguimiento de los procedimientos recomendados

El marco de trabajo de acceso condicional le proporciona una gran flexibilidad de configuración. Sin embargo, una gran flexibilidad también implica revisar cuidadosamente cada directiva de configuración antes de liberarla para evitar resultados no deseados.

### <a name="apply-ca-policies-to-every-app"></a>Aplicar directivas de CA a todas las aplicaciones

Los tokens de acceso se emiten de forma predeterminada si una condición de la directiva de CA no desencadena un control de acceso. Asegúrese de que todas las aplicaciones tienen aplicada al menos una directiva de acceso condicional.

> [!IMPORTANT]
> Tenga mucho cuidado al usar el bloque y todas las aplicaciones en una sola directiva. Esto podría bloquear a los administradores del portal de administración de Azure y las exclusiones no se pueden configurar para puntos de conexión importantes como Microsoft Graph.

### <a name="minimize-the-number-of-ca-policies"></a>Minimizar el número de directivas de CA

Crear una directiva para cada aplicación no es eficaz y conlleva una administración difícil. El acceso condicional solo aplicará las primeras 195 directivas por usuario. Se recomienda analizar las aplicaciones y agruparlas en aplicaciones que tengan los mismos requisitos de recursos para los mismos usuarios. Por ejemplo, si todas las aplicaciones de Microsoft 365 o todas las aplicaciones de recursos humanos tienen los mismos requisitos para los mismos usuarios, cree una sola directiva e incluya todas las aplicaciones a las que se aplique. 

### <a name="set-up-emergency-access-accounts"></a>Configurar cuentas de acceso de emergencia

Si configura mal una directiva, esto puede bloquear las organizaciones de Azure Portal. Mitigue el impacto de un bloqueo accidental del administrador mediante la creación de dos o más [cuentas de acceso de emergencia](../users-groups-roles/directory-emergency-access.md) en la organización.

* Cree una cuenta de usuario dedicada a la administración de directivas y excluida de todas las directivas.

* Situación excepcional para entornos híbridos:

  * Cree un grupo de seguridad local y sincronícelo con Azure AD. El grupo de seguridad debe contener su cuenta de administración de directivas dedicada. 

   * EXCLUYA este grupo de seguridad de todas las directivas de CA.

   * Cuando se produzca una interrupción del servicio, agregue los demás administradores al grupo local según corresponda y fuerce una sincronización. Esto anima su exención a las directivas de CA.

### <a name="set-up-report-only-mode"></a>Configurar el modo de solo informe

Puede ser difícil predecir el número y los nombres de los usuarios afectados por iniciativas de implementación comunes como, por ejemplo:

* bloqueo de la autenticación heredada
* requerimiento de MFA
* implementación de directivas de riesgo de inicio de sesión

El [modo de solo informe](concept-conditional-access-report-only.md) permite a los administradores evaluar el impacto de las directivas de CA antes de habilitarlas en su entorno.

Obtenga información sobre cómo [configurar el modo de solo informe en una directiva de acceso condicional](howto-conditional-access-insights-reporting.md).

### <a name="plan-for-disruption"></a>Planear la interrupción

Si se basa en un control de acceso único, como MFA o una ubicación de red, para proteger sus sistemas de TI, usted es susceptible a los errores de acceso si ese control de acceso único no está disponible o está mal configurado. Para reducir el riesgo de bloqueo durante las interrupciones imprevistas, [planee estrategias](../authentication/concept-resilient-controls.md) a fin de adoptarlas para su organización.

### <a name="set-naming-standards-for-your-policies"></a>Establecer pautas de nomenclatura para sus directivas

La pauta de nomenclatura ayuda a encontrar las directivas y a comprender su propósito sin tener que abrirlas en el portal de administrador de Azure. Se recomienda asignar un nombre a la directiva para mostrar:

* Un número de secuencia

* Las aplicaciones en la nube a las que se refiere

* La respuesta

* A quién se aplica

* Cuando se aplica (si procede)

![Pauta de nomenclatura](media/plan-conditional-access/11.png)

**Ejemplo**: Una directiva para requerir MFA para los usuarios de marketing con acceso a la aplicación de Dynamics CRP desde redes externas podría ser:

![Pauta de nomenclatura](media/plan-conditional-access/naming-example.png)

Un nombre descriptivo le ayuda a mantener una visión general de la implementación del acceso condicional. El número de secuencia es útil si necesita hacer referencia a una directiva en una conversación. Por ejemplo, si habla con un administrador por teléfono, puede pedirle que abra la directiva CA01 para resolver una incidencia.

#### <a name="naming-standards-for-emergency-access-controls"></a>Estándares de nomenclatura para controles de acceso de emergencia

Además de las directivas activas, implemente directivas deshabilitadas que actúen como [controles de acceso resistentes para escenarios de interrupción/emergencia](../authentication/concept-resilient-controls.md) secundarios. La pauta de nomenclatura para las directivas de contingencia debe incluir lo siguiente:
* HABILITAR EN CASO DE EMERGENCIA al principio para resaltar el nombre entre las otras directivas.

* El nombre de la interrupción a la que se debe aplicar.

* Un número de secuencia de ordenación para ayudar al administrador a saber en qué orden se deben habilitar las directivas.

**Ejemplo**

El siguiente nombre indica que esta directiva es la primera de cuatro directivas que debe habilitar en caso de una interrupción de MFA:

EM01 - HABILITAR EN CASO DE EMERGENCIA: Interrupción de MFA [1/4]: Exchange SharePoint: Requerir la unión a Azure AD híbrido para usuarios VIP.

### <a name="exclude-countries-from-which-you-never-expect-a-sign-in"></a>Excluya los países desde los que nunca espera un inicio de sesión.

Azure Active Directory permite crear [ubicaciones con nombre](location-condition.md). Cree una ubicación con nombre que incluya todos los países desde los que nunca esperaría que se produjera un inicio de sesión. Después, cree una directiva para todas las aplicaciones que bloquee el inicio de sesión desde esa ubicación con nombre. **Asegúrese de excluir a los administradores de esta directiva**.

### <a name="plan-your-policy-deployment"></a>Planear la implementación de la directiva

Cuando las nuevas directivas estén listas para su entorno, asegúrese de revisar cada una de ellas antes de publicarla para evitar resultados no deseados. Consulte la siguiente documentación para comprender información importante sobre cómo se aplican las directivas y cómo evitar incidencias.

* [Qué debería saber](best-practices.md)

* [Qué no debería hacer](best-practices.md)

## <a name="common-policies"></a>Directivas comunes

Al planear la solución de directiva de CA, determine si necesita crear directivas para lograr los siguientes resultados.

* [Requerir MFA](#require-mfa)
* [Respuesta ante las cuentas en posible riesgo](#respond-to-potentially-compromised-accounts)
* [Requerir dispositivos administrados](#require-managed-devices)
* [Requerir aplicaciones cliente aprobadas](#require-approved-client-apps)
* [Bloquear acceso](#block-access)

### <a name="require-mfa"></a>Requerir MFA

Casos de uso comunes para exigir el acceso a MFA:

* [De los administradores](howto-conditional-access-policy-admin-mfa.md)

* [A aplicaciones específicas](../authentication/tutorial-enable-azure-mfa.md)

* [Para todos los usuarios](howto-conditional-access-policy-all-users-mfa.md)

* [Desde ubicaciones de red en las que no se confía](untrusted-networks.md)

* [Para la administración de Azure](howto-conditional-access-policy-azure-management.md)

### <a name="respond-to-potentially-compromised-accounts"></a>Respuesta ante las cuentas en riesgo

Con las directivas de CA se pueden implementar respuestas automatizadas en inicios de sesión de identidades que pueden suponer un riesgo. La probabilidad de que una cuenta esté en riesgo se expresa mediante niveles de riesgo. En Identity Protection se calculan dos niveles de riesgo: riesgo de inicio de sesión o de usuario. Se pueden habilitar las tres directivas predeterminadas siguientes:

* [Exigir a todos los usuarios que se registren en MFA](howto-conditional-access-policy-risk.md)

* [Exigir un cambio de contraseña para los usuarios que son de alto riesgo](howto-conditional-access-policy-risk.md)

* [Exigir MFA para los usuarios con un riesgo de inicio de sesión medio o alto](howto-conditional-access-policy-risk.md)

### <a name="require-managed-devices"></a>Requerir dispositivos administrados

La proliferación de dispositivos compatibles para acceder a recursos en la nube ayuda a mejorar la productividad de los usuarios. Es posible que no desee que dispositivos con un nivel de protección desconocido tengan acceso a determinados recursos del entorno. Para estos recursos, [haga que los usuarios solo puedan acceder a ellos mediante un dispositivo administrado](require-managed-devices.md).

### <a name="require-approved-client-apps"></a>Requerir aplicaciones cliente aprobadas

Los empleados usan sus dispositivos móviles para tareas personales y profesionales. En el caso de los escenarios de BYOD, debe decidir si desea administrar todo el dispositivo o solo los datos que contenga. Si solo se administran datos y acceso, puede [requerir aplicaciones en la nube aprobadas](app-based-conditional-access.md) que puedan proteger los datos corporativos. Por ejemplo, puede requerir que solo se tenga acceso al correo electrónico a través de Outlook Mobile y no a través de un programa de correo genérico.

### <a name="block-access"></a>Bloquear acceso

La opción de [bloquear todo el acceso](howto-conditional-access-policy-block-access.md) resulta eficaz. Se puede usar, por ejemplo, cuando migra una aplicación a Azure AD, pero no está listo para que nadie inicie sesión en ella todavía. Bloquear acceso: 

* Altera el resto de asignaciones de un usuario

* Tiene la capacidad de bloquear el inicio de sesión en el inquilino a toda la organización

> [!IMPORTANT]
> Si crea una directiva para bloquear el acceso a todos los usuarios, asegúrese de excluir las cuentas de acceso de emergencia y considere la posibilidad de excluir a todos los administradores de la directiva.

Otros casos habituales en los que puede bloquear el acceso a los usuarios son:

* [Bloquee algunas ubicaciones de red](howto-conditional-access-policy-location.md) para tener acceso a sus aplicaciones en la nube. Puede usar esta directiva para bloquear determinados países de los que sabe que no procede el tráfico.

* Azure AD admite la autenticación heredada. Sin embargo, la autenticación heredada no admite MFA y muchos entornos la requieren para solucionar la seguridad de identidad. En este caso, puede [evitar que las aplicaciones que usan la autenticación heredada](block-legacy-authentication.md) accedan a los recursos del inquilino.

## <a name="build-and-test-policies"></a>Compilación y prueba de directivas

En cada fase de la implementación, asegúrese de que está evaluando que los resultados son los esperados. 

Cuando las nuevas directivas estén listas, impleméntelas en fases en el entorno de producción:

* Comunique los cambios internos a los usuarios finales.

* Empiece por un conjunto de usuarios pequeño y verifique que la directiva se comporta según lo previsto.

* Cuando expanda una directiva para incluir más usuarios, continúe con la exclusión de todos los administradores. La exclusión de los administradores garantiza que alguien tiene acceso a la directiva en caso de que se requieran cambios.

* Aplique una directiva a todos los usuarios solo después de que se haya probado exhaustivamente. Asegúrese de que tiene al menos una cuenta de administrador a la que no se aplica una directiva.

### <a name="create-test-users"></a>Creación de usuarios de prueba

Cree un conjunto de usuarios de prueba que reflejen los usuarios del entorno de producción. La creación de usuarios de prueba permite verificar que las directivas funcionan según lo previsto antes de que afecten a usuarios reales y puedan impedir su acceso a aplicaciones y recursos.

Algunas organizaciones tienen inquilinos de prueba para este propósito. Sin embargo, puede resultar difícil volver a crear todas las condiciones y las aplicaciones en un inquilino de prueba para probar completamente el resultado de una directiva.

### <a name="create-a-test-plan"></a>Creación de un plan de pruebas

El plan de pruebas es importante para tener una comparación entre los resultados previstos y los reales. Antes de probar algo debe tener siempre una previsión. En la siguiente tabla se muestran casos de prueba de ejemplo. Ajuste los escenarios y los resultados previstos en función de la configuración de sus directivas de acceso condicional.

| Directiva| Escenario| Resultado previsto |
| - | - | - |
| [Exigir la autenticación multifactor desde fuera de la oficina](untrusted-networks.md)| El usuario autorizado inicia sesión en la aplicación desde la oficina / una ubicación de confianza| Al usuario no se le solicita autenticación multifactor |
| [Exigir la autenticación multifactor desde fuera de la oficina](untrusted-networks.md)| El usuario autorizado inicia sesión en la aplicación desde otro lugar distinto a la oficina / una ubicación de confianza| Al usuario se le solicita autenticación multifactor para iniciar sesión |
| [Exigir autenticación multifactor (para administradores)](../fundamentals/concept-fundamentals-security-defaults.md)| El administrador global inicia sesión en la aplicación| Al administrador se le solicita autenticación multifactor |
| [Inicios de sesión no seguros](../identity-protection/howto-identity-protection-configure-risk-policies.md)| El usuario inicia sesión en la aplicación usando un explorador no aprobado| Al administrador se le solicita autenticación multifactor |
| [Administración de dispositivos](require-managed-devices.md)| El usuario autorizado intenta iniciar sesión desde un dispositivo autorizado| El acceso se le concede |
| [Administración de dispositivos](require-managed-devices.md)| El usuario autorizado intenta iniciar sesión desde un dispositivo no autorizado| El acceso se le bloquea |
| [Cambio de contraseña en caso de riesgo del usuario](../identity-protection/howto-identity-protection-configure-risk-policies.md)| El usuario autorizado intenta iniciar sesión con credenciales en riesgo (inicio de sesión de alto riesgo)| Al usuario se le solicita que cambie la contraseña o se le bloquea el acceso (de conformidad con la directiva) |


### <a name="configure-the-test-policy"></a>Configurar la directiva de prueba

En [Azure Portal](https://portal.azure.com/), configure las directivas de CA en Azure Active Directory > Seguridad > Acceso condicional.

Si desea obtener más información sobre cómo crear directivas de CA, consulte este ejemplo: [Directiva de CA para solicitar MFA cuando un usuario inicie sesión en Azure Portal](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json). Este artículo de inicio rápido le ayudará a:

* Familiarizarse con la interfaz de usuario

* Obtener una primera idea de cómo funciona el acceso condicional

### <a name="enable-the-policy-in-report-only-mode"></a>Habilitar la directiva en modo de solo informe

Para evaluar el impacto de la directiva, empiece por habilitarla en [modo de solo informe](concept-conditional-access-report-only.md). Las directivas de solo informe se evalúan durante el inicio de sesión, pero no se aplican controles de concesión ni de sesión. Una vez que guarde la directiva en modo de solo informe, podrá ver el impacto en los inicios de sesión en tiempo real de los registros de inicio de sesión. En los registros de inicio de sesión, seleccione un evento y vaya a la pestaña Solo informe para ver el resultado de cada directiva de solo informe.


![modo de solo informe ](media/plan-conditional-access/report-only-mode.png)

Al seleccionar la directiva, también puede ver cómo se evaluaron las asignaciones y los controles de acceso de la directiva mediante la pantalla Detalles de la directiva. Para que una directiva se aplique a un inicio de sesión, se deben satisfacer cada una de las asignaciones configuradas. 

### <a name="understand-the-impact-of-your-policies-using-the-insights-and-reporting-workbook"></a>Comprender el impacto de las directivas mediante el libro Conclusiones e informes

Puede ver el impacto agregado de las directivas de acceso condicional en el libro Conclusiones e informes. Para tener acceso al libro, necesita una suscripción de Azure Monitor y tendrá que [transmitir los registros de inicio de sesión a un área de trabajo de Log Analytics](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). 

### <a name="simulate-sign-ins-using-the-what-if-tool"></a>Simular inicios de sesión mediante la herramienta what-if

Otra manera de validar la directiva de acceso condicional es mediante la [herramienta what-if](troubleshoot-conditional-access-what-if.md), que simula las directivas que se aplicarían a un usuario que inicia sesión en circunstancias hipotéticas. Seleccione los atributos de inicio de sesión que desea probar (como usuario, aplicación, plataforma de dispositivo y ubicación) y consulte qué directivas se aplicarían.

> [!NOTE] 
> Aunque una ejecución simulada da una buena idea del efecto de una directiva de acceso condicional, no reemplaza a una serie de pruebas reales.

### <a name="test-your-policy"></a>Prueba de la directiva

Realice cada prueba del plan de pruebas con los usuarios de prueba.

**Asegúrese de que se prueban los criterios de exclusión de las directivas**. Por ejemplo, puede excluir a un usuario o grupo de una directiva que requiera MFA. Pruebe si a los usuarios excluidos se les solicita MFA, ya que la combinación de otras directivas puede hacer que se exija para esos usuarios.

### <a name="roll-back-policies"></a>Revertir las directivas

En caso de que necesite revertir las directivas recién implementadas, use una o varias de las siguientes opciones:

* **Deshabilite la directiva.** Deshabilitar una directiva garantiza que la directiva no se aplica cuando un usuario intente iniciar sesión. Si quiere que se use, siempre puede volver atrás y habilitar la directiva.

![imagen de la habilitación de la directiva](media/plan-conditional-access/enable-policy.png)

* **Excluya un usuario o grupo de una directiva.** Si un usuario no puede acceder a la aplicación, puede elegir excluirlo de la directiva.

![excluir usuarios y grupos](media/plan-conditional-access/exclude-users-groups.png)

> [!NOTE]
>  Esta opción debe usarse con moderación, solo en situaciones donde el usuario sea de confianza. El usuario debe agregarse de nuevo a la directiva o grupo n cuanto sea posible.

* **Elimine la directiva.** Si ya no es necesaria, [elimínela](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).

## <a name="manage-access-to-cloud-apps"></a>Administración del acceso a las aplicaciones en la nube

Use las siguientes opciones de administración para controlar y administrar las directivas de CA:

![administración-acceso](media/plan-conditional-access/manage-access.png)


### <a name="named-locations"></a>Ubicaciones con nombre

La condición de ubicación de una directiva de CA le permite asociar la configuración de los controles de acceso a las ubicaciones de red de los usuarios. Gracias a las [ubicaciones con nombre](location-condition.md), puede crear agrupaciones lógicas de rangos de direcciones IP o de países y regiones.

### <a name="custom-controls"></a>Controles personalizados

Los [controles personalizados](controls.md) redirigen a los usuarios a un servicio compatible para satisfacer requisitos de autenticación fuera de Azure AD. Para satisfacer este control, el explorador de un usuario se redirige al servicio externo, lleva a cabo todas las actividades de autenticación necesarias y luego se vuelve a redirigir a Azure AD. Azure AD comprueba la respuesta y, si el usuario se autenticó o validó correctamente, seguirá en el flujo de acceso condicional.

### <a name="terms-of-use"></a>Términos de uso

Antes de acceder a determinadas aplicaciones en la nube de su entorno, puede obtener el consentimiento de los usuarios mediante su aceptación de las condiciones de uso (CDU). Siga este [inicio rápido para crear las condiciones de uso](require-tou.md).

### <a name="classic-policies"></a>Directivas clásicas

En [Azure Portal](https://portal.azure.com/), encontrará las directivas de CA en Azure Active Directory > Seguridad > Acceso condicional. Es posible que su organización también tenga directivas de CA antiguas que no se hayan creado con esta página. Estas directivas se conocen como directivas clásicas. Se recomienda que [considere la posibilidad de migrar estas directivas clásicas en Azure Portal](best-practices.md).

## <a name="troubleshoot-conditional-access"></a>Solución de problemas de acceso condicional

Cuando un usuario tenga una incidencia con una directiva de CA, recopile la siguiente información para facilitar la solución de problemas.

* Nombre principal del usuario

* Nombre para mostrar del usuario

* Nombre del sistema operativo

* Marca de tiempo (aproximado es correcto)

* Aplicación de destino

* Tipo de aplicación cliente (explorador frente a cliente)

* Id. de correlación (este es único para el inicio de sesión)

Si el usuario ha recibido un mensaje con un vínculo Más detalles, podrá recopilar la mayor parte de esta información para usted.

![No se puede obtener el mensaje de error de la aplicación](media/plan-conditional-access/cant-get-to-app.png)

Una vez que haya recopilado la información, consulte los siguientes recursos:

* [Problemas de inicio de sesión con acceso condicional](troubleshoot-conditional-access.md): comprenda los resultados inesperados de inicio de sesión relacionados con el acceso condicional mediante mensajes de error y el registro de inicios de sesión de Azure AD.

* [Uso de la herramienta What-If](troubleshoot-conditional-access-what-if.md): comprenda por qué una directiva se ha aplicado o no a un usuario en una circunstancia específica, o bien si una directiva se aplicaría en un estado conocido.

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre Multi-Factor Authentication](../authentication/concept-mfa-howitworks.md)

[Más información sobre Identity Protection](../identity-protection/overview-identity-protection.md)

[Administración de directivas de CA con Microsoft Graph API](/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta.md)