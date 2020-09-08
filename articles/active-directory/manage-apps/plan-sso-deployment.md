---
title: Planificación de una implementación de inicio de sesión único de Azure Active Directory
description: Guía para ayudarle a planear, implementar y administrar el inicio de sesión único en la organización.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 06/10/2020
ms.author: baselden
ms.reviewer: jeedes
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: e8aa5515bf8ae8e842a98656167f4198ac652981
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642391"
---
# <a name="plan-a-single-sign-on-deployment"></a>Planeación de una implementación de inicio de sesión único

El Inicio de sesión único (SSO) significa acceder a todas las aplicaciones y los recursos que necesita un usuario iniciando sesión una sola vez con una única cuenta de usuario. Con SSO, los usuarios pueden acceder a todas las aplicaciones necesarias sin tener que autenticarse una segunda vez.

## <a name="benefits-of-sso"></a>Ventajas del inicio de sesión único

El inicio de sesión único (SSO) agrega seguridad y comodidad cuando los usuarios inician sesión en aplicaciones en Azure Active Directory (Azure AD). 

Muchas organizaciones confían en las aplicaciones de software como servicio (SaaS), como Office 365, Box y Salesforce, para la productividad del usuario final. Tradicionalmente, el personal de TI tenía que crear y actualizar individualmente las cuentas de usuario en cada aplicación SaaS, y los usuarios tenían que recordar una contraseña para cada una.

Azure Marketplace tiene más de 3000 aplicaciones con conexiones de inicio de sesión único previamente integradas, lo que facilita su integración en el inquilino.

## <a name="licensing"></a>Licencias

- **Licencias de Azure AD**: el inicio de sesión único para las aplicaciones SaaS preintegradas es gratuito. Pero es posible que el número de objetos en el directorio y las características que quiere implementar requieran licencias adicionales. Para obtener una lista completa de los requisitos de licencia, vea [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).
- **Licencias de aplicaciones**: necesitará las licencias adecuadas para las aplicaciones SaaS con el fin de satisfacer las necesidades empresariales. Trabaje con el propietario de la aplicación para determinar si los usuarios asignados a la aplicación tienen las licencias adecuadas para sus roles dentro de ella. Si Azure AD administra el aprovisionamiento automático en función de los roles, los roles asignados en Azure AD se deben alinear con el número de licencias que posee dentro de la aplicación. Un número incorrecto de las licencias que se poseen en la aplicación puede dar lugar a errores durante el aprovisionamiento o la actualización de un usuario.

## <a name="plan-your-sso-team"></a>Planificación del equipo de inicio de sesión único

- **Interactuar con las partes interesadas adecuadas**: cuando los proyectos tecnológicos fracasan, normalmente se debe a expectativas incorrectas relacionadas con el impacto, los resultados y las responsabilidades. Para evitar estos problemas, [asegúrese de que interactúa con las partes interesadas adecuadas](https://aka.ms/deploymentplans) y que las partes interesadas comprendan sus roles.
- **Planificación de las comunicaciones**: la comunicación es fundamental para el éxito de cualquier servicio nuevo. Comunique de forma proactiva a los usuarios cómo y cuándo va a cambiar su experiencia y cómo obtener soporte técnico si experimentan problemas. Revise las opciones sobre [cómo accederán los usuarios finales a sus aplicaciones habilitadas para el inicio de sesión único](end-user-experiences.md) y elabore las comunicaciones de forma que coincidan con la selección. 

## <a name="plan-your-sso-protocol"></a>Planificación del protocolo de inicio de sesión único

Una implementación de SSO basada en protocolos de federación mejora la seguridad, la confiabilidad y las experiencias de los usuarios finales, y es más fácil de implementar. En Azure AD, muchas aplicaciones están preintegradas con [guías paso a paso disponibles](../saas-apps/tutorial-list.md). Puede encontrarlas en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/). En el artículo [Inicio de sesión único en aplicaciones de Azure Active Directory](what-is-single-sign-on.md) encontrará información detallada sobre cada método de inicio de sesión único.

Hay dos formas principales de permitir a los usuarios el inicio de sesión único a las aplicaciones:

- **Con el inicio de sesión único federado**, Azure AD autentica al usuario en la aplicación mediante su cuenta de Azure AD. Este método se admite para las aplicaciones compatibles con protocolos como SAML 2.0, WS-Federation u OpenID Connect, y es el modo de inicio de sesión único más completo. Se recomienda usar el inicio de sesión único federado con Azure AD cuando una aplicación lo admita, en lugar del inicio de sesión único basado en contraseña y ADFS.

- **Con el inicio de sesión único basado en contraseña**, los usuarios finales inician sesión en la aplicación con un nombre de usuario y una contraseña la primera vez que acceden a ella. Después del primer inicio de sesión, Azure AD proporciona el nombre de usuario y la contraseña a la aplicación. El inicio de sesión único basado en contraseña permite el almacenamiento seguro de contraseñas de las aplicaciones y la reproducción mediante una extensión de explorador web o aplicación móvil. Esta opción aprovecha el proceso de inicio de sesión existente proporcionado por la aplicación, permite que un administrador administre las contraseñas y no requiere que el usuario conozca la contraseña.

### <a name="considerations-for-federation-based-sso"></a>Consideraciones para el inicio de sesión único basado en federación

- **Uso de OpenID Connect y OAuth**: si la aplicación a la que se va a conectar lo admite, use el método OIDC/OAuth 2.0 para habilitar el inicio de sesión único en esa aplicación. Este método requiere menos configuración y permite una experiencia del usuario más completa. Para más información, vea [OAuth 2.0](../develop/v2-oauth2-auth-code-flow.md), [OpenID Connect 1.0](../develop/v2-protocols-oidc.md) y la [Guía para desarrolladores de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-developers-guide).
- **Configuraciones de punto de conexión para el inicio de sesión único basado en SAML**: si usa SAML, los desarrolladores necesitarán información específica antes de configurar la aplicación. Para más información, consulte [Configuración del inicio de sesión único basado en SAML](configure-saml-single-sign-on.md).
- **Administración de certificados para el inicio de sesión único basado en SAML**: al habilitar el inicio de sesión único federado para la aplicación, Azure AD crea un certificado con una validez predeterminada de tres años. Si es necesario, puede personalizar la fecha de expiración de ese certificado. Asegúrese de que tiene procesos implantados para renovar los certificados antes de que caduquen. Para más información, vea [Certificados de administración de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-sso-certs).

### <a name="considerations-for-password-based-sso"></a>Consideraciones para el inicio de sesión único basado en contraseña

El uso de Azure AD para el inicio de sesión único basado en contraseña requiere implementar una extensión del explorador que recuperará de forma segura las credenciales y rellenará los formularios de inicio de sesión. Defina un mecanismo para implementar la extensión a escala con los [exploradores compatibles](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction). Las opciones incluyen:

- [Directiva de grupo para Internet Explorer](https://azure.microsoft.com/documentation/articles/active-directory-saas-ie-group-policy/)
- [Configuration Manager para Internet Explorer](https://docs.microsoft.com/configmgr/core/clients/deploy/deploy-clients-to-windows-computers)
- [Descarga y configuración de Chrome, Firefox, Microsoft Edge o IE controlada por el usuario](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)

Para más información, vea [Configuración del inicio de sesión único con contraseña](https://docs.microsoft.com/azure/active-directory/application-config-sso-how-to-configure-password-sso-non-gallery).

#### <a name="capturing-login-forms-metadata-for-applications-that-arent-in-the-gallery"></a>Captura de metadatos de formularios de inicio de sesión para las aplicaciones que no están en la galería

Microsoft admite la captura de metadatos en una aplicación web para el almacenamiento de contraseñas (la captura de los campos de nombre de usuario y contraseña). Vaya a la dirección URL de inicio de sesión durante el proceso de configuración de la aplicación para capturar los metadatos de formularios. Solicite al propietario de la aplicación la dirección URL de inicio de sesión exacta. Esta información se usa durante el proceso de inicio de sesión, la asignación de credenciales de Azure AD a la aplicación durante el inicio de sesión.

Para más información, vea [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure AD?: inicio de sesión único basado en contraseña](https://azure.microsoft.com/documentation/articles/active-directory-appssoaccess-whatis/).

#### <a name="indications-that-metadata-in-forms-needs-to-be-recaptured"></a>Indicaciones de que los metadatos de los formularios se deben volver a capturar

Cuando las aplicaciones cambian su diseño HTML, es posible que tenga que volver a capturar los metadatos para ajustar los cambios. Entre los síntomas comunes que indican que el diseño HTML ha cambiado se incluyen los siguientes:

- Los usuarios notifican que al hacer clic en la aplicación se "bloquea" en la página de inicio de sesión
- Los usuarios notifican que el nombre de usuario o la contraseña no se rellenan

#### <a name="shared-accounts"></a>Cuentas compartidas

Desde la perspectiva del inicio de sesión, las aplicaciones con cuentas compartidas no son diferentes de una aplicación de la galería en la que se usa el inicio de sesión único con contraseña para usuarios individuales. Pero hay varios pasos adicionales necesarios al planear y configurar una aplicación diseñada para usar cuentas compartidas:

1. Trabaje con los usuarios profesionales de la aplicación para documentar lo siguiente:
   1. El conjunto de usuarios de la organización que van a usar la aplicación
   1. El conjunto existente de credenciales en la aplicación asociadas con el conjunto de usuarios 
1. Para cada combinación de conjunto de usuarios y credenciales, cree un grupo de seguridad local en la nube según los requisitos.
1. Restablezca las credenciales compartidas. Una vez que se haya implementado la aplicación en Azure AD, los usuarios no necesitan la contraseña de la cuenta compartida. Como la contraseña se va a almacenar en Azure AD, considere la posibilidad de establecer una muy larga y compleja. 
1. Si la aplicación lo admite, configure la sustitución automática de la contraseña. De ese modo, ni siquiera el administrador que ha realizado la instalación inicial conocerá la contraseña de la cuenta compartida. 

## <a name="plan-your-authentication-method"></a>Planificación del método de autenticación

La elección del método de autenticación correcto es una primera decisión fundamental en la configuración de una solución de identidad híbrida de Azure AD. Implemente el método de autenticación que se configura mediante Azure AD Connect, y que también aprovisionará a los usuarios en la nube.

Para elegir un método de autenticación, es necesario tener en cuenta el tiempo, la infraestructura existente, la complejidad y el costo de implementar cada opción. Estos factores varían con cada organización y pueden cambiar con el tiempo. Debe elegir el que mejor coincida con el escenario concreto. Para más información, vea [Selección del método de autenticación adecuado para la solución de identidad híbrida de Azure Active Directory](https://docs.microsoft.com/azure/security/fundamentals/choose-ad-authn).

## <a name="plan-your-security-and-governance"></a>Planificación de la seguridad y la gobernanza 

La identidad es el nuevo eje principal de la atención y las inversiones en materia de seguridad porque los perímetros de red se han vuelto cada vez más porosos y menos eficaces debido a la explosión de los dispositivos BYOD y las aplicaciones en la nube. 

### <a name="plan-access-reviews"></a>Planificación de las revisiones de acceso

Las [revisiones de acceso](https://docs.microsoft.com/azure/active-directory/governance/create-access-review) permiten a las organizaciones administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. Debe planear la revisión del acceso de los usuarios de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Algunos de los temas clave para planear la configuración de revisiones de acceso incluyen los siguientes:

1. Identificación de una cadencia de las revisiones de acceso que se adapte a las necesidades empresariales. Esto puede ser tan frecuente como una vez por semana, mensualmente, anualmente o como un ejercicio a petición.

1. Creación de grupos que representen los revisores de los informes de acceso de la aplicación. Tendrá que asegurarse de que las partes interesadas más familiarizadas con la aplicación y sus usuarios de destino y casos de uso participen en las revisiones de acceso

1. La finalización de una revisión de acceso incluye la eliminación de los permisos de acceso a la aplicación de los usuarios que ya no los necesiten. Planifique el control de las posibles solicitudes de soporte técnico de los usuarios denegados. Un usuario eliminado permanecerá eliminado en Azure AD durante 30 días, tiempo durante el cual un administrador puede restaurarlo si es necesario. Después de 30 días, ese usuario se eliminará definitivamente. Mediante el portal de Azure Active Directory, un administrador global puede eliminar permanentemente un usuario eliminado recientemente de manera explícita antes de que se alcance ese período de tiempo.

### <a name="plan-auditing"></a>Planificación de auditorías

Azure AD proporciona [informes que contienen datos técnicos y empresariales](https://azure.microsoft.com/documentation/articles/active-directory-view-access-usage-reports/). 

Hay informes de seguridad y actividad disponibles. En los informes de seguridad se muestran los usuarios marcados en riesgo y los inicios de sesión no seguros. Para ayudar a comprender el comportamiento de los usuarios de la organización, los informes de actividad detallan la actividad de inicio de sesión y proporcionan registros de auditoría de todos los inicios de sesión. Puede usar los informes para administrar los riesgos, aumentar la productividad y supervisar el cumplimiento.

| Tipo de informe | Revisión de acceso | Informes de seguridad | Informe de inicio de sesión |
|-------------|---------------|------------------|----------------|
| Se usa para revisar | Permisos y uso de la aplicación. | Cuentas potencialmente comprometidas | Quién accede a las aplicaciones |
| Acciones posibles | Auditar el acceso; revocar los permisos | Revocar el acceso; forzar el restablecimiento de la seguridad | Revocar el acceso |

Azure AD conserva la mayoría de los datos de auditoría durante 30 días y hace que estén disponibles a través del portal de administración de Azure o una API para que pueda descargarlos a los sistemas de análisis.

### <a name="consider-using-microsoft-cloud-application-security"></a>Posibilidad de usar Microsoft Cloud Application Security

Microsoft Cloud App Security (MCAS) es una solución de agente de seguridad de acceso a la nube (CASB). Ofrece visibilidad sobre las aplicaciones y servicios en la nube, proporciona análisis sofisticados para identificar y combatir las ciberamenazas, y permite controlar cómo viajan los datos.

La implementación de MCAS permite lo siguiente:

- El uso de Cloud Discovery para asignar e identificar el entorno de la nube y las aplicaciones en la nube que usa la organización.
- La prohibición y desautorización de aplicaciones en la nube.
- El uso de conectores de aplicación fáciles de implementar que aprovechan las ventajas de las API de proveedor, para ganar visibilidad y gobernanza de las aplicaciones a las que se conecta.
- Uso de la protección Control de aplicaciones de acceso condicional para obtener visibilidad en tiempo real y control sobre el acceso y las actividades dentro de las aplicaciones de nube.
- Ayuda para mantener el control constante mediante la configuración y posterior ajuste permanente de las directivas.

El control de sesiones de Microsoft Cloud Application Security (MCAS) está disponible para cualquier explorador de cualquier plataforma principal en cualquier sistema operativo. Las aplicaciones móviles y de escritorio también se pueden bloquear o permitir. Mediante la integración nativa con Azure AD, se puede admitir cualquier aplicación configurada con SAML, o bien aplicaciones Open ID Connect con el inicio de sesión único de Azure AD, incluidas [varias aplicaciones destacadas](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad).

Para obtener información sobre MCAS, vea la [introducción a Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/what-is-cloud-app-security). MCAS es un servicio de suscripción basado en el usuario. Puede revisar los detalles de licencia en la [hoja de datos de licencias de MCAS](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE2NXYO).

### <a name="use-conditional-access"></a>Uso del acceso condicional

Con el acceso condicional, puede automatizar las decisiones de control de acceso basadas en criterios para las aplicaciones en la nube.

Las directivas de acceso condicional se aplican una vez que se completa la autenticación en una fase. Por tanto, el acceso condicional no está pensado como primera línea de defensa en escenarios como los ataques por denegación de servicio (DoS), pero puede usar señales de estos eventos para determinar el acceso. Por ejemplo, se puede usar el nivel de riesgo de inicio de sesión, la ubicación de la solicitud, etc. Para más información sobre el acceso condicional, vea [la introducción](https://docs.microsoft.com/azure/active-directory/conditional-access/plan-conditional-access) y el [plan de implementación](https://docs.microsoft.com/azure/active-directory/conditional-access/plan-conditional-access).

## <a name="azure-sso-technical-requirements"></a>Requisitos técnicos de inicio de sesión único de Azure

En la sección siguiente se detallan los requisitos para configurar la aplicación específica, incluidos los entornos necesarios, los puntos de conexión, la asignación de notificaciones, los atributos necesarios, los certificados y los protocolos que se usan. Necesitará esta información para configurar el inicio de sesión único en el [portal de Azure AD](https://portal.azure.com/).

### <a name="authentication-mechanism-details"></a>Detalles del mecanismo de autenticación

Para todas las aplicaciones SaaS preintegradas, Microsoft proporciona un tutorial y no necesita esta información. Si la aplicación no está en nuestro Marketplace o galería de aplicaciones, es posible que tenga que recopilar los elementos de datos siguientes:

- **El proveedor de identidades actual que usa la aplicación para el inicio de sesión único, si corresponde**; por ejemplo: AD FS, PingFederate, Okta
- **Los protocolos admitidos por la aplicación de destino**: por ejemplo, SAML 2.0, OpenID Connect, OAuth, autenticación basada en formularios, WS-Fed, WS-Trust
- **El protocolo que se ha configurado con Azure AD**: por ejemplo, SAML 2.0 o 1.1, OpenID Connect, OAuth, basado en formularios, WS-Fed

### <a name="attribute-requirements"></a>Requisitos de los atributos

Hay un conjunto preconfigurado de atributos y asignaciones de atributos entre los objetos de usuario de Azure AD y los objetos de usuario de cada aplicación SaaS. Algunas aplicaciones administran otros tipos de objetos, como grupos. Planifique la asignación de atributos de usuario desde Azure AD a la aplicación y [personalice las asignaciones de atributos predeterminados](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes) según las necesidades de la empresa.

### <a name="certificate-requirements"></a>Requisitos de certificados

El certificado de la aplicación debe estar actualizado o habrá riesgo de que los usuarios no puedan acceder a la aplicación. La mayoría de los certificados de aplicación SaaS son válidos durante 36 meses. Puede cambiar esa duración del certificado en la hoja de la aplicación. Asegúrese de documentar la expiración y saber cómo administrará la renovación de certificados. 

Hay dos maneras de administrar los certificados. 

- **Sustitución automática de certificados**: Microsoft admite la [sustitución de claves de firma en Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-signing-key-rollover). Aunque este es el método preferido para administrar certificados, no todos los ISV lo admiten.

- **Actualización manual**: cada aplicación tiene su propio certificado que expira en función de cómo se haya definido. Antes de que expire el certificado de la aplicación, cree uno nuevo y envíelo al ISV. Esta información se puede extraer de los metadatos de federación. [Más información sobre los metadatos de federación aquí.](https://docs.microsoft.com/azure/active-directory/develop/active-directory-federation-metadata)

## <a name="implement-sso"></a>Implementación del inicio de sesión único

Use las fases siguientes para planear e implementar la solución en la organización:

### <a name="user-configuration-for-sso"></a>Configuración de usuarios para inicio de sesión único

- **Identificación de los usuarios de prueba**

   Póngase en contacto con el propietario de la aplicación y pídale que cree un mínimo de tres usuarios de prueba dentro de la aplicación. Asegúrese de la información que va a usar como identificador principal está rellenada de forma correcta y coincide con un atributo que está disponible en Azure AD. En la mayoría de los casos, esto se asignará a "NameID" para las aplicaciones basadas en SAML. Para los tokens JWT, es "preferred_username".
   
   Cree el usuario en Azure AD ya sea de forma manual como un usuario basado en la nube, o bien sincronice el usuario desde el entorno local mediante el motor de sincronización de Azure AD Connect. Asegúrese de que la información coincide con las notificaciones que se envían a la aplicación.

- **Configuración del inicio de sesión único**

   En la [lista de aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list), busque y abra el tutorial de inicio de sesión único para la aplicación y después siga los pasos del tutorial de configurar correctamente la aplicación SaaS.

   Si no se puede encontrar la aplicación, vea la [documentación de la aplicación personalizada](https://docs.microsoft.com/azure/active-directory/application-config-sso-how-to-configure-federated-sso-non-gallery). Esto le guiará por el proceso de agregar una aplicación que no se encuentra en la galería de Azure AD.

   Opcionalmente, puede usar las notificaciones emitidas en el token SAML para aplicaciones empresariales mediante la [documentación de las instrucciones de Microsoft](https://docs.microsoft.com/azure/active-directory/active-directory-claims-mapping). Asegúrese de que esto se asigna a lo que espera recibir en la respuesta SAML para la aplicación. Si encuentra problemas durante la configuración, use las instrucciones sobre [cómo depurar la integración del inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-debugging).

### <a name="provide-sso-change-communications-to-end-users"></a>Comunicación de cambios del inicio de sesión único a los usuarios finales

Implemente el plan de comunicación. Asegúrese de que permite a los usuarios finales saber que se va a producir un cambio, cuándo se ha producido, qué hacer ahora y cómo buscar ayuda.

### <a name="verify-end-user-scenarios-for-sso"></a>Comprobación de escenarios de usuario final para el inicio de sesión único

Puede usar los casos de prueba siguientes para realizar pruebas en los dispositivos propiedad de la empresa y personales para asegurarse de que las configuraciones de inicio de sesión único funcionan según lo esperado. En los escenarios siguientes se asume que un usuario navega a una dirección URL de la aplicación y pasa por un flujo de autenticación iniciado por el proveedor de servicios.

| Escenario | Resultado esperado por el usuario en el flujo de autenticación iniciado por el proveedor de servicios |
|----------|---------------------------------------------------|
| Inicio de sesión en la aplicación con Internet Explorer mientras se encuentra en la red corporativa. | Se produce la autenticación integrada de Windows (IWA) sin mensajes adicionales. |
| Inicio de sesión en la aplicación con Internet Explorer desde fuera de la red corporativa con un nuevo intento de inicio de sesión. | Mensaje basado en formularios en el servidor de AD FS. El usuario inicia sesión correctamente y el explorador solicita MFA. |
| Inicio de sesión en la aplicación con Internet Explorer desde fuera de la red corporativa con una sesión actual y nunca se ha realizado MFA. | El usuario no recibe un mensaje para el primer factor. El usuario recibe un mensaje para MFA. |
| Inicio de sesión en la aplicación con Internet Explorer desde fuera de la red corporativa con una sesión actual y en esta sesión ya se ha realizado MFA. | El usuario no recibe un mensaje para el primer factor. El usuario no recibe MFA. Inicio de sesión único del usuario en la aplicación. |
| Inicio de sesión en la aplicación con Internet Chrome, Firefox o Safari desde fuera de la red corporativa con una sesión actual y en esta sesión ya se ha realizado MFA. | El usuario no recibe un mensaje para el primer factor. El usuario no recibe MFA. Inicio de sesión único del usuario en la aplicación. |
| Inicio de sesión en la aplicación con Chrome, Firefox o Safari desde fuera de la red corporativa con un nuevo intento de inicio de sesión. | Mensaje basado en formularios en el servidor de AD FS. El usuario inicia sesión correctamente y el explorador solicita MFA. |
| Inicio de sesión en la aplicación como Chrome o Firefox mientras se encuentra en la red corporativa con una sesión actual. | El usuario no recibe un mensaje para el primer factor. El usuario no recibe MFA. Inicio de sesión único del usuario en la aplicación. |
| Inicio de sesión en la aplicación con la aplicación móvil con un nuevo intento de inicio de sesión. | Mensaje basado en formularios en el servidor de AD FS. El usuario inicia sesión correctamente y el cliente ADAL solicita MFA. |
| Un usuario no autorizado intenta iniciar sesión en la aplicación con la dirección URL de inicio de sesión. | Mensaje basado en formularios en el servidor de AD FS. El usuario no puede iniciar sesión con el primer factor. |
| El usuario autorizado intenta iniciar sesión pero escribe una contraseña incorrecta. | El usuario navega a la dirección URL de la aplicación y recibe un error de nombre de usuario o contraseña incorrectos. |
| El usuario autorizado hace clic en el vínculo de un correo electrónico y ya está autenticado. | El usuario hace clic en la dirección URL e inicia sesión en la aplicación sin solicitudes adicionales. |
| El usuario autorizado hace clic en el vínculo de un correo electrónico y todavía no está autenticado. | El usuario hace clic en la dirección URL y se le solicita autenticarse con el primer factor. |
| El usuario autorizado inicia sesión en la aplicación con la aplicación móvil de la aplicación (iniciada por el proveedor de servicios) con un nuevo intento de inicio de sesión. | Mensaje basado en formularios en el servidor de AD FS. El usuario inicia sesión correctamente y el cliente ADAL solicita MFA. |
| Un usuario no autorizado intenta iniciar sesión en la aplicación con la dirección URL de inicio de sesión (iniciada por el proveedor de servicios). | Mensaje basado en formularios en el servidor de AD FS. El usuario no puede iniciar sesión con el primer factor. |
| El usuario autorizado intenta iniciar sesión pero escribe una contraseña incorrecta.| El usuario navega a la dirección URL de la aplicación y recibe un error de nombre de usuario o contraseña incorrectos. |
| El usuario autorizado cierra la sesión y después vuelve a iniciar sesión. | Si se configura la dirección URL de cierre de sesión, se cierra la sesión del usuario en todos los servicios y se le solicita que se autentique. |
| El usuario autorizado cierra la sesión y después vuelve a iniciar sesión. | Si no se configura la dirección URL de cierre de sesión, el usuario volverá a iniciar sesión de forma automática con un token existente de la sesión del explorador de Azure AD existente. |
| El usuario autorizado hace clic en el vínculo de un correo electrónico y ya está autenticado. | El usuario hace clic en la dirección URL e inicia sesión en la aplicación sin solicitudes adicionales. |
| El usuario autorizado hace clic en el vínculo de un correo electrónico y todavía no está autenticado. | El usuario hace clic en la dirección URL y se le solicita autenticarse con el primer factor. |

## <a name="manage-sso"></a>Administración del inicio de sesión único

En esta sección se describen los requisitos y las recomendaciones para administrar el inicio de sesión único de forma correcta.

### <a name="required-administrative-roles"></a>Roles administrativos requeridos

Use siempre el rol con los permisos mínimos disponibles para realizar la tarea requerida en Azure Active Directory. Microsoft recomienda [revisar los distintos roles disponibles](https://docs.microsoft.com/azure/active-directory/active-directory-assign-admin-roles-azure-portal) y elegir el correcto para resolver las necesidades para cada rol de esta aplicación. Es posible que algunos roles se tengan que aplicar temporalmente y quitar una vez completada la implementación.

| Persona| Roles | Rol de Azure AD (si es necesario) |
|--------|-------|-----------------------------|
| Administrador del departamento de soporte técnico | Soporte técnico de nivel 1 | None |
| Administración de identidades | Configurar y depurar cuando haya problemas que afectan a Azure AD | Administrador global |
| Administrador de aplicaciones | Atestación de usuarios en la aplicación, configuración en usuarios con permisos | None |
| Administradores de infraestructura | Propietario de la sustitución de certificados | Administrador global |
| Propietario de la empresa o parte interesada | Atestación de usuarios en la aplicación, configuración en usuarios con permisos | None |

Se recomienda usar [Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-configure) (PIM) para administrar los roles con el fin de proporcionar auditoría, control y revisión de acceso adicionales para los usuarios con permisos de directorio.

### <a name="sso-certificate-lifecycle-management"></a>Administración del ciclo de vida de los certificados de inicio de sesión único

Es importante identificar los roles adecuados y las listas de distribución de correo electrónico encargadas de administrar el ciclo de vida del certificado de firma entre Azure AD y la aplicación que se va a configurar con el inicio de sesión único. Estos son algunos de los roles clave que se recomienda tener:

- Propietario para actualizar las propiedades de usuario en la aplicación
- Propietario de guardia para soporte técnico de interrupciones o correcciones de la aplicación
- Lista de distribución de correo electrónico supervisada con detalle para notificaciones de cambios relacionados con el certificado

La duración máxima de un certificado es de tres años. Se recomienda establecer un proceso sobre cómo controlar un cambio de certificado entre Azure AD y la aplicación. Esto puede ayudar a evitar o minimizar una interrupción debido a la expiración del certificado o forzar la sustitución del certificado.

### <a name="rollback-process"></a>Proceso de reversión

Después de completar las pruebas en función de los casos de prueba, es el momento de pasar a producción con la aplicación. Pasar a producción significa que se implementan las configuraciones planeadas y probadas en el inquilino de producción y se aplican a los usuarios. Pero es importante planear qué hacer en caso de que la implementación no funcione según lo previsto. Si se produce un error en la configuración del inicio de sesión único durante la implementación, debe saber cómo mitigar cualquier interrupción y reducir el impacto para los usuarios.

La disponibilidad de métodos de autenticación dentro de la aplicación determinará la mejor estrategia. Asegúrese de que siempre dispone de documentación detallada para los propietarios de la aplicación sobre cómo recuperar de forma exacta el estado de configuración de inicio de sesión original en caso de que se produzcan problemas en la implementación.

- **Si la aplicación admite varios proveedores de identidades**, por ejemplo, LDAP y AD FS y Ping, no elimine la configuración de inicio de sesión único existente durante la implementación. En su lugar, deshabilítela durante la migración en caso de que deba volver a activarla más tarde. 

- **Si la aplicación no admite varios IDP** pero permite a los usuarios iniciar sesión con la autenticación basada en formularios (nombre de usuario y contraseña), asegúrese de que pueden revertir a este enfoque en caso de que se produzca un error en la nueva implementación de la configuración de inicio de sesión único.

### <a name="access-management"></a>Administración de acceso

Se recomienda elegir un enfoque escalado para administrar el acceso a los recursos. Los enfoques comunes incluyen el uso de grupos locales mediante la sincronización a través de Azure AD Connect, [la creación de grupos dinámicos en Azure AD en función de atributos del usuario](https://docs.microsoft.com/azure/active-directory/active-directory-groups-dynamic-membership-azure-portal), o bien la creación de [grupos de autoservicio](https://docs.microsoft.com/azure/active-directory/active-directory-accessmanagement-self-service-group-management) en Azure AD administrados por un propietario del recurso.

### <a name="monitor-security"></a>Supervisión de la seguridad

Se recomienda configurar una cadencia periódica en la que se revisen los diferentes aspectos de seguridad de las aplicaciones SaaS y se realicen las acciones correctoras necesarias.

### <a name="troubleshooting"></a>Solución de problemas

Los vínculos siguientes presentan escenarios de solución de problemas. Es posible que quiera crear una guía específica para el personal de soporte técnico que incorpore estos escenarios y los pasos para corregirlos.

#### <a name="consent-issues"></a>Problemas de consentimiento

- [Error de consentimiento inesperado](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-unexpected-user-consent-prompt)

- [Error de consentimiento de usuario](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-unexpected-user-consent-error)

#### <a name="sign-in-issues"></a>Problemas de inicio de sesión

- [Problemas al iniciar sesión desde un portal personalizado](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-other-problem-deeplink)

- [Problemas al iniciar sesión desde Aplicaciones](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-other-problem-access-panel)

- [Error en la página de inicio de sesión de la aplicación](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-problem-application-error)

- [Problema al iniciar sesión en una aplicación de Microsoft](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-problem-first-party-microsoft)

#### <a name="sso-issues-for-applications-listed-in-the-azure-application-gallery"></a>Problemas de inicio de sesión único para aplicaciones de la Galería de aplicaciones de Azure

- [Problema con el inicio de sesión único con contraseña para aplicaciones de la Galería de aplicaciones de Azure](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-problem-password-sso-gallery) 

- [Problema con el inicio de sesión federado para aplicaciones de la Galería de aplicaciones de Azure](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-problem-federated-sso-gallery)   

#### <a name="sso-issues-for-applications-not-listed-in-the-azure-application-gallery"></a>Problemas de inicio de sesión único para aplicaciones NO incluidas en la Galería de aplicaciones de Azure

- [Problema con el inicio de sesión único con contraseña para aplicaciones NO incluidas en la Galería de aplicaciones de Azure](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-problem-password-sso-non-gallery) 

- [Problema con el inicio de sesión federado para aplicaciones NO incluidas en la Galería de aplicaciones de Azure](https://docs.microsoft.com/azure/active-directory/manage-apps/application-sign-in-problem-federated-sso-non-gallery)

## <a name="next-steps"></a>Pasos siguientes

[Depuración de SSO basado en SAML](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-debugging)

[Asignación de notificaciones para las aplicaciones a través de PowerShell](https://docs.microsoft.com/azure/active-directory/active-directory-claims-mapping)

[Personalización de notificaciones emitidas en el token SAML](https://docs.microsoft.com/azure/active-directory/develop/active-directory-saml-claims-customization)

[Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference)

[Protocolo SAML de cierre de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-out-protocol-reference)

[Azure AD B2B](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b) (para usuarios externos como socios y proveedores)

[Acceso condicional de Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal)

[Azure Active Directory Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection)

[Acceso con SSO](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

[Tutorial de inicio de sesión único de aplicaciones](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list)

[Notas del producto para descargar: Plan de implementación de inicio de sesión único](https://aka.ms/SSODeploymentPlan)