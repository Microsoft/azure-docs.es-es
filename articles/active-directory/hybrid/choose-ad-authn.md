---
title: Autenticación para soluciones de identidad híbrida de Azure AD
titleSuffix: Active Directory
description: Esta guía está pensada para ayudar a los Consejeros delegados (CEO), Directores de información (CIO), Directores de seguridad de la información (CISO), Arquitectos jefe de identidad, Arquitectos de empresa y responsables de la toma de decisiones de TI a elegir un método de autenticación para su solución de identidad híbrida de Azure AD en organizaciones de tamaño medio y grande.
keywords: ''
author: martincoetzer
ms.author: martinco
ms.date: 10/30/2019
ms.topic: article
ms.service: security
ms.subservice: security-fundamentals
ms.workload: identity
ms.openlocfilehash: beabe9440c37d16ca4a81efdbc9588739f4c7473
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279353"
---
# <a name="choose-the-right-authentication-method-for-your-azure-active-directory-hybrid-identity-solution"></a>Seleccione el método de autenticación adecuado para su solución de identidad híbrida de Azure Active Directory

La elección del método de autenticación correcto es la primera preocupación para las organizaciones que desean mover sus aplicaciones a la nube. Esta decisión no debe tomarse a la ligera por las razones siguientes:

1. Es la primera decisión de una organización que quiere trasladarse a la nube.

2. El método de autenticación es un componente esencial de la presencia de una organización en la nube. Esto es debido a que controla el acceso a todos los datos y recursos que hay en ella.

3. Es la base de todas las demás características avanzadas de seguridad y experiencia de usuario en Azure AD.

La identidad es el nuevo plano de control de la seguridad de TI, de modo que la autenticación se convierte en el guardián de acceso de una organización al nuevo mundo de la nube. Las organizaciones necesitan un plano de control de identidad que fortalezca su seguridad y mantenga las aplicaciones en la nube a salvo de intrusos.

> [!NOTE]
> Cambiar el método de autenticación requiere planeación, pruebas y un posible tiempo de inactividad. El [lanzamiento preconfigurado](./how-to-connect-staged-rollout.md) es una excelente forma de probar la migración de usuarios desde la federación hasta la autenticación en la nube.

### <a name="out-of-scope"></a>Fuera de ámbito
Las organizaciones que no tienen una superficie de directorio local existente no entran en el ámbito de este artículo. Por lo general, esas empresas crean identidades solo en la nube, lo que no requiere una solución de identidad híbrida. Las identidades que solo se usan en la nube existen únicamente en la nube y no están asociadas a sus identidades locales correspondientes.

## <a name="authentication-methods"></a>Métodos de autenticación
Cuando la solución de identidad híbrida de Azure AD sea el nuevo plano de control, la autenticación será la base del acceso a la nube. La elección del método de autenticación correcto es una primera decisión fundamental en la configuración de una solución de identidad híbrida de Azure AD. Implemente el método de autenticación que se configura mediante Azure AD Connect, y que también aprovisionará a los usuarios en la nube.

Para elegir un método de autenticación, es necesario tener en cuenta el tiempo, la infraestructura existente, la complejidad y el costo de implementar cada opción. Estos factores varían con cada organización y pueden cambiar con el tiempo.

>[!VIDEO https://www.youtube.com/embed/YtW2cmVqSEw]

Azure AD admite los siguientes métodos de autenticación en soluciones de identidad híbrida.

### <a name="cloud-authentication"></a>Autenticación en la nube
Cuando se elige este método de autenticación, Azure AD administra el proceso de inicio de sesión de los usuarios. Junto con el inicio de sesión único (SSO) completo, los usuarios pueden iniciar sesión en las aplicaciones en la nube sin tener que volver a escribir sus credenciales. Con la autenticación en la nube puede elegir entre dos opciones:

**Sincronización de hash de contraseñas de Azure AD**. Es la manera más sencilla de habilitar la autenticación para los objetos de directorio local en Azure AD. Gracias a ella, los usuarios pueden usar el mismo nombre de usuario y contraseña que en el entorno local sin tener que implementar ninguna infraestructura adicional. Algunas características premium de Azure AD, como Identity Protection y [Azure AD Domain Services](../../active-directory-domain-services/tutorial-create-instance.md), requieren la sincronización de hash de contraseñas con independencia del método de autenticación seleccionado.

> [!NOTE]
> Las contraseñas nunca se almacenan en texto no cifrado o cifradas con un algoritmo reversible en Azure AD. Para obtener más información sobre el proceso real de la sincronización de hash de contraseñas, consulte [Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

**Autenticación de paso a través de Azure AD**. Proporciona una validación de contraseñas simple para los servicios de autenticación de Azure AD mediante un agente de software que se ejecuta en uno o más servidores locales. Los servidores validan a los usuarios directamente con la instancia local de Active Directory, lo que garantiza que la validación de la contraseña no se realiza en la nube.

Las empresas con un requisito de seguridad para aplicar de inmediato los estados de cuenta de los usuarios locales, las directivas de contraseña y las horas de inicio de sesión pueden usar este método de autenticación. Para obtener más información sobre el proceso real de autenticación de paso a través, consulte [Inicio de sesión del usuario con la autenticación de paso a través de Azure AD](../../active-directory/hybrid/how-to-connect-pta.md).

### <a name="federated-authentication"></a>Autenticación federada
Cuando se elige este método de autenticación, Azure AD deja el proceso de autenticación en manos de un sistema de autenticación de confianza como, por ejemplo, una instancia local de Servicios de federación de Active Directory (AD FS) para validar la contraseña del usuario.

El sistema de autenticación puede proporcionar requisitos adicionales de autenticación avanzada. Algunos ejemplos son la autenticación basada en tarjetas inteligentes o la autenticación multifactor de terceros. Para más información, consulte [Implementación de Servicios de federación de Active Directory](/windows-server/identity/ad-fs/deployment/windows-server-2012-r2-ad-fs-deployment-guide).

En la siguiente sección encontrará un árbol de decisión que le ayudará a decidir qué método de autenticación es más adecuado en su caso. Gracias a este árbol podrá determinar si debe implementar la autenticación en la nube o la autenticación federada para la solución híbrida de Azure AD.

## <a name="decision-tree"></a>Árbol de decisión

![Árbol de decisión de autenticación de Azure AD](./media/choose-ad-authn/azure-ad-authn-image1.png)

Detalles sobre las preguntas de decisión:

1. Azure AD puede controlar el inicio de sesión de los usuarios sin tener que depender de los componentes locales para comprobar las contraseñas.
2. Azure AD puede entregar el inicio de sesión de usuario a un proveedor de autenticación de confianza como AD FS de Microsoft.
3. Si tiene que aplicar directivas de seguridad de Active Directory a nivel de usuario, como la cuenta expirada, cuenta deshabilitada, contraseña expirada, cuenta bloqueada y horas de inicio de sesión de cada usuario, Azure AD requiere algunos componentes locales.
4. Las características de inicio de sesión no son compatibles de forma nativa con Azure AD:
   * Inicio de sesión con tarjetas inteligentes o certificados.
   * Inicio de sesión con el servidor de MFA de forma local.
   * Inicio de sesión con una solución de autenticación de terceros.
   * Solución de autenticación local de varios sitios.
5. Azure AD Identity Protection requiere la sincronización del hash de contraseña, independientemente de qué método de inicio de sesión elija, para proporcionar el informe *Usuarios con credenciales filtradas*. Las organizaciones pueden conmutar por error a la sincronización del hash de contraseña si se produce un error en su método principal de inicio de sesión y se ha configurado antes del evento de error.

> [!NOTE]
> Azure AD Identity Protection requiere licencias de [Azure AD Premium P2](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="detailed-considerations"></a>Consideraciones detalladas

### <a name="cloud-authentication-password-hash-synchronization"></a>Autenticación en la nube: Sincronización de hash de contraseña

* **Esfuerzo**. La sincronización de hash de contraseñas requiere un esfuerzo mínimo en cuanto a la implementación, el mantenimiento y la infraestructura.  Este nivel de esfuerzo se aplica generalmente a organizaciones que solo necesitan que sus usuarios inicien sesión en Office 365, aplicaciones de SaaS y otros recursos basados en Azure AD. Cuando se habilita, la sincronización de hash de contraseñas forma parte del proceso de sincronización de Azure AD Connect y se ejecuta cada dos minutos.

* **Experiencia del usuario**. Para mejorar la experiencia de inicio de sesión de los usuarios, puede implementar un SSO de conexión directa gracias a la sincronización de hash de contraseñas. El SSO de conexión directa elimina las solicitudes innecesarias cuando los usuarios inician sesión.

* **Escenarios avanzados**. Si las organizaciones lo eligen, es posible usar información detallada de identidades con informes de Azure AD Identity Protection con Azure AD Premium P2. Por ejemplo, el informe de credenciales filtradas. Windows Hello para empresas tiene [requisitos específicos cuando se usa la sincronización de hash de contraseñas](/windows/access-protection/hello-for-business/hello-identity-verification). [Azure AD Domain Services](../../active-directory-domain-services/tutorial-create-instance.md) requiere sincronización de hash de contraseñas para aprovisionar a los usuarios con sus credenciales corporativas en el dominio administrado.

    Las organizaciones que requieren una autenticación multifactor con sincronización de hash de contraseñas tienen que usar Azure Multi-Factor Authentication o los [controles personalizados de acceso condicional](../../active-directory/conditional-access/controls.md#custom-controls-preview). Esas organizaciones no pueden usar métodos de autenticación multifactor de terceros o locales que se basen en la federación.

> [!NOTE]
> El acceso condicional de Azure AD requiere licencias de [Azure AD Premium P1](https://azure.microsoft.com/pricing/details/active-directory/).

* **Continuidad del negocio**. La sincronización de hash de contraseñas tiene una alta disponibilidad como servicio en la nube que se puede escalar a todos los centros de datos de Microsoft. Para asegurarse de que la sincronización de hash de contraseñas no deja de funcionar durante períodos prolongados, implemente un segundo servidor de Azure AD Connect de modo provisional en una configuración en espera.

* **Consideraciones**. Actualmente, la sincronización de hash de contraseñas no aplica inmediatamente los cambios en los estados de la cuenta local. En esta situación, un usuario tendrá acceso a las aplicaciones en la nube hasta que el estado de la cuenta del usuario se sincronice con Azure AD. Si las organizaciones quieren superar esta limitación, es recomendable ejecutar un nuevo ciclo de sincronización después de que los administradores realicen actualizaciones masivas en los estados de las cuentas de los usuarios locales. Por ejemplo, pueden deshabilitar las cuentas.

> [!NOTE]
> Los estados de contraseña expirada y cuenta bloqueada no se sincronizan actualmente con Azure AD mediante Azure AD Connect. Al cambiar una contraseña de usuario y establecer la marca *el usuario debe cambiar la contraseña en el siguiente inicio de sesión*, el hash de contraseña no se sincronizará con Azure AD a través de Azure AD Connect hasta que el usuario cambie su contraseña.

Para ver los pasos de implementación, consulte [Implementación de la sincronización de hash de contraseñas](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

### <a name="cloud-authentication-pass-through-authentication"></a>Autenticación en la nube: Autenticación de paso a través  

* **Esfuerzo**. Para realizar la autenticación de paso a través, necesita uno o más (recomendamos tres) agentes livianos instalados en los servidores existentes. Estos agentes deben tener acceso a la instancia local de Active Directory Domain Services, incluidos los controladores de dominio locales de AD. Estos agentes necesitan acceso saliente a Internet y tener acceso a los controladores de dominio. Por esta razón, no se pueden implementar los agentes en una red perimetral.

    Este tipo de autenticación de paso a través necesita que los controladores de dominio le proporcionen acceso a la red sin restricciones. Todo el tráfico de red se cifra y se limita a las solicitudes de autenticación. Para obtener más información sobre este proceso, consulte [Información de seguridad detallada sobre la autenticación de paso a través](../../active-directory/hybrid/how-to-connect-pta-security-deep-dive.md).

* **Experiencia del usuario**. Para mejorar la experiencia de inicio de sesión de los usuarios, puede implementar un SSO de conexión directa con la autenticación de paso a través. El SSO de conexión directa elimina las solicitudes innecesarias cuando los usuarios inician sesión.

* **Escenarios avanzados**. La autenticación de paso a través aplica la directiva de cuenta local al iniciar sesión. Por ejemplo, se deniega el acceso cuando el estado de la cuenta de un usuario local indica que está deshabilitada, bloqueada, con su [contraseña expirada](../../active-directory/hybrid/how-to-connect-pta-faq.md#what-happens-if-my-users-password-has-expired-and-they-try-to-sign-in-by-using-pass-through-authentication) o que el intento de inicio de sesión se encuentra fuera de las horas de inicio de sesión del usuario.

    Las organizaciones que requieren una autenticación multifactor con una autenticación de paso a través tienen que usar Azure Multi-Factor Authentication (MFA) o los [controles personalizados de acceso condicional](../../active-directory/conditional-access/controls.md#custom-controls-preview). Esas organizaciones no pueden usar métodos de autenticación multifactor de terceros o locales que se basen en la federación. Las características avanzadas requieren que se implemente la sincronización de hash de contraseñas sin importar si elige o no la autenticación de paso a través. Por ejemplo, esto sucede con el informe de credenciales filtradas de Identity Protection.

* **Continuidad del negocio**. Es recomendable que implemente dos agentes de autenticación de paso a través adicionales. Estos extras son adicionales al primer agente en el servidor de Azure AD Connect. Esta implementación adicional garantiza una alta disponibilidad de solicitudes de autenticación. Cuando hay tres agentes implementados, un agente puede dejar de funcionar cuando otro agente está inactivo por mantenimiento.

    Existe otro beneficio al implementar la sincronización de hash de contraseñas además de la autenticación de paso a través. Actúa como un método de autenticación de respaldo cuando el método de autenticación principal ya no está disponible.

* **Consideraciones**. Puede utilizar la sincronización de hash de contraseñas como método de autenticación de respaldo para la autenticación de paso a través, cuando los agentes no puedan asegurarse de las credenciales de un usuario debido a un error local importante. La conmutación por error de la sincronización de hash de contraseñas no sucede automáticamente y debe usar Azure AD Connect para cambiar el método de inicio de sesión manualmente.

    Para ver más detalles sobre la autenticación de paso a través, incluida la compatibilidad con identificadores alternativos, consulte las [preguntas más frecuentes](../../active-directory/hybrid/how-to-connect-pta-faq.md).

Para obtener información sobre los pasos de implementación, consulte [Implementación de la autenticación de paso a través](../../active-directory/hybrid/how-to-connect-pta.md).

### <a name="federated-authentication"></a>Autenticación federada

* **Esfuerzo**. El uso de un sistema de autenticación federada se basa en un sistema externo de confianza para autenticar a los usuarios. Algunas empresas desean reutilizar sus inversiones existentes en sistemas federados con su solución de identidad híbrida de Azure AD. El mantenimiento y la administración del sistema federado no entra en el control de Azure AD. Es responsabilidad de la organización que usa el sistema federado asegurarse de que se implementa de forma segura y de que puede administrar la carga de autenticación.

* **Experiencia del usuario**. La experiencia de usuario de la autenticación federada depende de la implementación de las características, la topología y la configuración de la granja de servidores de federación. Algunas organizaciones necesitan esta flexibilidad para adaptar y configurar el acceso a la granja de servidores de federación a fin de satisfacer sus requisitos de seguridad. Por ejemplo, es posible configurar usuarios y dispositivos conectados internamente para iniciar la sesión de los usuarios automáticamente, sin pedirles las credenciales. Esta configuración funciona porque ya han iniciado sesión en sus dispositivos. Por otro lado, si es necesario, algunas características de seguridad avanzadas pueden dificultar el proceso de inicio de sesión del usuario.

* **Escenarios avanzados**. Se requiere una solución de autenticación federada cuando los clientes tienen un requisito de autenticación que Azure AD no admite de forma nativa. Puede consultar información detallada que le ayudará a [elegir la opción de inicio de sesión correcta](/archive/blogs/samueld/choosing-the-right-sign-in-option-to-connect-to-azure-ad-office-365). Considere los siguientes requisitos comunes:

  * Un método de autenticación que requiere tarjetas inteligentes o certificados.
  * Servidores MFA locales o proveedores de autenticación multifactor de terceros que requieren un proveedor de identidades federadas.
  * Un método de autenticación con una solución de autenticación de terceros. Consulte la [lista de compatibilidad de federación de AD Azure](../../active-directory/hybrid/how-to-connect-fed-compatibility.md).
  * Un inicio de sesión que requiera un elemento sAMAccountName, como DOMAIN\username, en lugar de usar el nombre principal de usuario (UPN); por ejemplo, user@domain.com.

* **Continuidad del negocio**. Los sistemas federados generalmente requieren una matriz de servidores de carga equilibrada, conocida como granja de servidores. Esta granja de servidores está configurada en una red interna y en una topología de red perimetral para garantizar una alta disponibilidad de las solicitudes de autenticación.

    La sincronización de hash de contraseñas se puede implementar en combinación con la autenticación federada como método de autenticación de reserva cuando el método de autenticación principal ya no esté disponible. Por ejemplo, cuando los servidores locales no están disponibles. Algunas organizaciones empresariales de gran tamaño requieren una solución de federación para admitir varios puntos de entrada a Internet con DNS geográfico para solicitudes de autenticación de baja latencia.

* **Consideraciones**. Normalmente, los sistemas federados requieren una inversión más importante en infraestructura local. La mayoría de las organizaciones eligen esta opción si ya han invertido en una federación local. También la escogerán si es un requisito de negocios importante usar un proveedor de identidades únicas. Es más difícil usar la federación y solucionar sus problemas en comparación con las soluciones de autenticación en la nube.

Si hay un dominio no enrutable que no se puede verificar en Azure AD, debe realizar una configuración adicional para implementar el inicio de sesión de un id. de usuario. Este requisito se conoce como compatibilidad con identificadores de inicio de sesión alternativos. Para conocer los requisitos y las limitaciones, consulte [Configurar un identificador de inicio de sesión alternativo](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id). Si decide usar un proveedor de autenticación multifactor de terceros con federación, asegúrese de que el proveedor admite WS-Trust para permitir que los dispositivos se unan a Azure AD.

Para conocer los pasos de implementación, consulte [Deploying Federation Servers](/windows-server/identity/ad-fs/deployment/deploying-federation-servers) (Implementación de los servidores de federación).

> [!NOTE]
> Al implementar la solución de identidad híbrida de Azure AD, debe asegurarse de implementar una de las topologías admitidas de Azure AD Connect. Aprenda más sobre las configuraciones admitidas y no admitidas en [Topologías de Azure AD Connect](../../active-directory/hybrid/plan-connect-topologies.md).

## <a name="architecture-diagrams"></a>Diagramas de arquitectura

En los siguientes diagramas se describen los componentes de arquitectura de alto nivel que requiere cada método de autenticación que se puede usar con la solución de identidad híbrida de Azure AD. Proporcionan información general para comparar las diferencias entre las soluciones.

* Simplicidad de una solución de sincronización de hash de contraseñas:

    ![Identidad híbrida de Azure AD con sincronización de hash de contraseñas](./media/choose-ad-authn/azure-ad-authn-image2.png)

* Requisitos del agente para la autenticación de paso a través, con el uso de dos agentes para la redundancia:

    ![Identidad híbrida de Azure AD con autenticación de paso a través](./media/choose-ad-authn/azure-ad-authn-image3.png)

* Componentes necesarios para la federación en la red perimetral e interna de la organización:

    ![Identidad híbrida de Azure AD con autenticación federada](./media/choose-ad-authn/azure-ad-authn-image4.png)

## <a name="comparing-methods"></a>Comparación de métodos

|Consideración|Sincronización de hash de contraseñas + SSO de conexión directa|Autenticación de paso a través + SSO de conexión directa|Federación con AD FS|
|:-----|:-----|:-----|:-----|
|¿Dónde se realiza la autenticación?|En la nube|En la nube, después de un intercambio de comprobación de contraseña segura con el agente de autenticación local|Local|
|¿Cuáles son los requisitos de servidor local más allá del sistema de aprovisionamiento (Azure AD Connect)?|None|Un servidor para cada agente de autenticación adicional|Dos o más servidores de AD FS<br><br>Dos o más servidores WAP en la red perimetral o DMZ|
|¿Cuáles son los requisitos de redes e Internet locales más allá del sistema de aprovisionamiento?|None|[Acceso saliente a Internet](../../active-directory/hybrid/how-to-connect-pta-quick-start.md) desde los servidores que ejecutan los agentes de autenticación|[Acceso entrante a Internet](/windows-server/identity/ad-fs/overview/ad-fs-requirements) para los servidores WAP del perímetro<br><br>Acceso entrante de red para los servidores de AD FS desde los servidores WAP del perímetro<br><br>Equilibrio de carga de red|
|¿Hay algún requisito de certificado TLS/SSL?|No|No|Sí|
|¿Hay alguna solución de supervisión de estado?|No se requiere|El estado del agente lo proporciona el [Centro de administración de Azure Active Directory](../../active-directory/hybrid/tshoot-connect-pass-through-authentication.md)|[Azure AD Connect Health](../../active-directory/hybrid/how-to-connect-health-adfs.md)|
|¿Los usuarios realizan el inicio de sesión único en los recursos de nube desde dispositivos unidos a un dominio de la red de la empresa?|Sí, con [SSO de conexión directa](../../active-directory/hybrid/how-to-connect-sso.md)|Sí, con [SSO de conexión directa](../../active-directory/hybrid/how-to-connect-sso.md)|Sí|
|¿Qué tipos de inicio de sesión se admiten?|UserPrincipalName + contraseña<br><br>Autenticación integrada de Windows con [SSO de conexión directa](../../active-directory/hybrid/how-to-connect-sso.md)<br><br>[Identificador de inicio de sesión alternativo](../../active-directory/hybrid/how-to-connect-install-custom.md)|UserPrincipalName + contraseña<br><br>Autenticación integrada de Windows con [SSO de conexión directa](../../active-directory/hybrid/how-to-connect-sso.md)<br><br>[Identificador de inicio de sesión alternativo](../../active-directory/hybrid/how-to-connect-pta-faq.md)|UserPrincipalName + contraseña<br><br>sAMAccountName + contraseña<br><br>Autenticación integrada de Windows<br><br>[Autenticación de certificados y tarjetas inteligentes](/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[Identificador de inicio de sesión alternativo](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|¿Se admite Windows Hello para empresas?|[Modelo de confianza de clave](/windows/security/identity-protection/hello-for-business/hello-identity-verification)|[Modelo de confianza de clave](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br>*Requiere el nivel funcional de dominio de Windows Server 2016*|[Modelo de confianza de clave](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Modelo de confianza de certificado](/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|¿Cuáles son las opciones de autenticación multifactor?|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Controles personalizados con acceso condicional*](../../active-directory/conditional-access/controls.md)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Controles personalizados con acceso condicional*](../../active-directory/conditional-access/controls.md)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Servidor de Azure MFA](../../active-directory/authentication/howto-mfaserver-deploy.md)<br><br>[MFA de terceros](/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[Controles personalizados con acceso condicional*](../../active-directory/conditional-access/controls.md)|
|¿Qué estados de cuenta de usuario se admiten?|Cuentas deshabilitadas<br>(retraso de hasta 30 minutos)|Cuentas deshabilitadas<br><br>Cuenta bloqueada<br><br>Cuenta expirada<br><br>Contraseña expirada<br><br>Horas de inicio de sesión|Cuentas deshabilitadas<br><br>Cuenta bloqueada<br><br>Cuenta expirada<br><br>Contraseña expirada<br><br>Horas de inicio de sesión|
|¿Cuáles son las opciones de acceso condicional?|[Acceso condicional de Azure AD, con Azure AD Premium](../../active-directory/conditional-access/overview.md)|[Acceso condicional de Azure AD, con Azure AD Premium](../../active-directory/conditional-access/overview.md)|[Acceso condicional de Azure AD, con Azure AD Premium](../../active-directory/conditional-access/overview.md)<br><br>[Reglas de notificaciones de AD FS](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|¿Se admite el bloqueo de protocolos heredados?|[Sí](../../active-directory/conditional-access/overview.md)|[Sí](../../active-directory/conditional-access/overview.md)|[Sí](/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|¿Se puede personalizar el logotipo, la imagen y la descripción en las páginas de inicio de sesión?|[Sí, con Azure AD Premium](../../active-directory/fundamentals/customize-branding.md)|[Sí, con Azure AD Premium](../../active-directory/fundamentals/customize-branding.md)|[Sí](../../active-directory/hybrid/how-to-connect-fed-management.md)|
|¿Qué escenarios avanzados se admiten?|[Smart Password Lockout](../../active-directory/authentication/howto-password-smart-lockout.md) (Bloqueo inteligente de contraseñas)<br><br>[Informes de credenciales filtrados, con Azure AD Premium P2](../identity-protection/overview-identity-protection.md)|[Smart Password Lockout](../../active-directory/authentication/howto-password-smart-lockout.md) (Bloqueo inteligente de contraseñas)|Sistema de autenticación multisitio de baja latencia<br><br>[Bloqueo de extranet de AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[Integración con sistemas de identidad de terceros](../../active-directory/hybrid/how-to-connect-fed-compatibility.md)|

> [!NOTE]
> Actualmente, los controles personalizados del acceso condicional de Azure AD no admiten el registro de dispositivos.

## <a name="recommendations"></a>Recomendaciones
El sistema de identidad garantiza que los usuarios tengan acceso a aplicaciones en la nube y a aplicaciones de línea de negocio que se migran y están disponibles en la nube. La autenticación controla el acceso a las aplicaciones, a fin de mantener la productividad de los usuarios autorizados, y a los usuarios malintencionados alejados de los datos confidenciales de la organización.

Use o habilite la sincronización de hash de contraseñas con independencia del método de autenticación que elija, por las razones siguientes:

1. **Alta disponibilidad y recuperación ante desastres**. La autenticación de paso a través y la federación se basan en la infraestructura local. En la autenticación de paso a través, la superficie local incluye las redes y el hardware del servidor que requieren los agentes de autenticación de paso a través. En el caso de la federación, la superficie local es aún mayor. Esto es debido a que se requieren servidores en la red perimetral para las solicitudes de autenticación de proxy y los servidores de federación internos.

    Para evitar los únicos puntos de errores, implemente servidores redundantes. A continuación, las solicitudes de autenticación siempre se atenderán si se produce un error en alguno de los componentes. Tanto la autenticación de paso a través como la de federación están sujetas a los controladores de dominio para responder a las solicitudes de autenticación, que también pueden producir un error. Muchos de estos componentes deben mantenerse para permanecer en buen estado. Las interrupciones son más probables cuando el mantenimiento no se planifica ni se implementa correctamente. Para evitar interrupciones, se puede usar la sincronización de hash de contraseñas porque el servicio de autenticación en la nube de Microsoft Azure AD se escala globalmente y siempre está disponible.

2. **Subsistencia a interrupciones locales**.  Las consecuencias de una interrupción local debida a un ataque cibernético o a un desastre pueden ser de gran envergadura; desde daños en la reputación de la marca hasta la paralización de organizaciones que se ven incapaces de hacer frente al ataque. Últimamente muchas organizaciones han sido víctimas de ataques de malware, como ransomware dirigido, que provocaron que sus servidores locales se desconectaran. A la hora de ayudar a los clientes a tratar con estos tipos de ataques, Microsoft observó dos categorías de organizaciones:

   * Las organizaciones que también activaron previamente la sincronización de hash de contraseñas sobre la autenticación federada o la autenticación de paso a través, cambiaron su método de autenticación principal para usar la sincronización de hash de contraseñas. Gracias a esto, volvieron a estar en línea en cuestión de horas. Al usar el acceso al correo electrónico a través de Office 365, pudieron trabajar para resolver los problemas y tener acceso a otras cargas de trabajo basadas en la nube.

   * Las organizaciones que no habilitaron previamente la sincronización de hash de contraseñas, tuvieron que recurrir a sistemas de correo electrónico de consumidor externos que no son de confianza para las comunicaciones, con el fin de resolver los problemas. En esos casos, tardaron semanas en restaurar su infraestructura de identidad local, antes de que los usuarios pudieran volver a iniciar sesión en aplicaciones basadas en la nube.

3. **Identity Protection**. una de las mejores maneras de proteger a los usuarios de la nube es Azure AD Identity Protection con Azure AD Premium P2. Microsoft examina continuamente Internet en busca de listas de usuarios y contraseñas que los usuarios malintencionados venden y proporcionan en la Internet oscura. Azure AD puede usar esta información para comprobar si algunos de los nombres de usuario y contraseñas de su organización están en peligro. Por lo tanto, es fundamental habilitar la sincronización de hash de contraseñas, con independencia de qué método de autenticación se use, ya sea autenticación federada o de paso a través. Las credenciales filtradas se presentan como un informe. Use esta información para bloquear o exigir que los usuarios cambien sus contraseñas cuando intenten iniciar sesión con contraseñas filtradas.

## <a name="conclusion"></a>Conclusión

En este artículo se describen distintas opciones de autenticación que las organizaciones pueden configurar e implementar para admitir el acceso a las aplicaciones en la nube. Para satisfacer los diversos requisitos de negocios, técnicos y de seguridad, las organizaciones pueden elegir entre la sincronización de hash de contraseñas, la autenticación de paso a través y la federación.

Revise con atención cada método de autenticación interno. ¿Los esfuerzos para implementar la solución, así como la experiencia del usuario en el proceso de inicio de sesión responden a los requisitos de su negocio? También debe valorar si su organización necesita las características de escenarios avanzados y continuidad empresarial de cada método de autenticación. Por último, sopese los detalles de cada método de autenticación. ¿Alguno de ellos le impide implementar su elección?

## <a name="next-steps"></a>Pasos siguientes

En la actualidad, las amenazas están presentes 24 horas al día y proceden de cualquier lugar. La implementación del método de autenticación correcto le ayudará a mitigar los riesgos de seguridad y a proteger las identidades.

[Comience a trabajar](../fundamentals/active-directory-whatis.md) con Azure AD e implemente la solución de autenticación adecuada para su organización.

Si está pensando cambiar de la autenticación federada a la autenticación en la nube, consulte la información acerca de [cómo cambiar el método de inicio de sesión](../../active-directory/hybrid/plan-connect-user-signin.md). Para ayudarle a planear e implementar la migración, use [estos planes de implementación de proyectos](https://aka.ms/deploymentplans) o considere la posibilidad de usar la nueva característica de [lanzamiento por fases](../../active-directory/hybrid/how-to-connect-staged-rollout.md) para migrar usuarios federados al uso de la autenticación en la nube en un enfoque por fases.