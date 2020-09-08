---
title: Cinco pasos para la integración de todas las aplicaciones en Azure AD
description: En esta guía se explica cómo integrar todas las aplicaciones con Azure AD. En cada paso, se explica el valor y se proporcionan vínculos a los recursos que explicarán los detalles técnicos.
services: active-directory
author: knicholasa
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: nichola
ms.openlocfilehash: bbe813b46b41bf0e2b8bf9c6b3fe3e3e843204ed
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89319753"
---
# <a name="five-steps-for-integrating-all-your-apps-with-azure-ad"></a>Cinco pasos para la integración de todas las aplicaciones en Azure AD

Azure Active Directory (Azure AD) es el servicio de administración de accesos e identidades basado en la nube de Microsoft. Azure AD proporciona soluciones seguras de autenticación y autorización para que los clientes, asociados y empleados puedan acceder a las aplicaciones que necesiten. Con Azure AD, el [acceso condicional](../conditional-access/overview.md), la [autenticación multifactor](../authentication/concept-mfa-howitworks.md), el [inicio de sesión único](../hybrid/how-to-connect-sso.md) y el [aprovisionamiento automático de usuarios](../app-provisioning/user-provisioning.md) facilitan y protegen la administración de accesos e identidades.

Si su empresa tiene una suscripción a Microsoft 365, probablemente [ya esté utilizando](/office365/enterprise/about-office-365-identity) Azure AD. Sin embargo, se puede usar Azure AD para todas las aplicaciones y, al [centralizar la administración de aplicaciones](../manage-apps/common-scenarios.md), puede utilizar las mismas características, herramientas y directivas de administración de identidades en toda la cartera de aplicaciones. Se proporcionará así una solución unificada que mejore la seguridad, reduzca los costos, aumente la productividad y permita garantizar la compatibilidad. Y tendrá acceso remoto seguro a aplicaciones locales.

En esta guía se explica cómo integrar todas las aplicaciones con Azure AD. En cada paso, se explica el valor y se proporcionan vínculos a los recursos que explicarán los detalles técnicos. Estos pasos se presentan en el orden recomendado. Sin embargo, puede ir directamente a cualquier parte del proceso para comenzar con lo que le suponga un mayor valor.

Otros recursos sobre este tema, incluidas las notas del producto detalladas de procesos de negocio, que se pueden encontrar en nuestra página [Recursos para migrar aplicaciones a Azure Active Directory](../manage-apps/migration-resources.md).

## <a name="1-use-azure-ad-for-new-applications"></a>1. Uso de Azure AD para aplicaciones nuevas

En primer lugar, céntrese en las aplicaciones que acaba de adquirir. Cuando su empresa comience a utilizar una aplicación nueva, [agréguela al inquilino de Azure AD](../manage-apps/add-application-portal.md) de inmediato. Configure una directiva de empresa para que la incorporación de nuevas aplicaciones a Azure AD sea una práctica estándar de su organización. Esto supone una interrupción mínima de los procesos de negocio existentes, y permite investigar y demostrar el valor que se obtiene de la integración de aplicaciones sin cambiar la forma en que las personas hacen negocios en su entorno actualmente.

Azure Active Directory (Azure AD) incluye una galería que contiene miles de aplicaciones previamente integradas para facilitar el inicio. Puede [agregar una aplicación de la galería a la organización de Azure AD](../manage-apps/add-application-portal.md) con los tutoriales [paso a paso](../saas-apps/tutorial-list.md) para integrar conocidas aplicaciones como:

- [ServiceNow](../saas-apps/servicenow-tutorial.md)
- [Workday](../saas-apps/workday-tutorial.md)
- [Salesforce](../saas-apps/salesforce-tutorial.md)
- [AWS](../saas-apps/amazon-web-service-tutorial.md)
- [Slack](../saas-apps/slack-tutorial.md)

Además, puede [integrar aplicaciones que no estén en la galería](../manage-apps/view-applications-portal.md), incluida cualquier aplicación que ya exista en su organización o cualquier aplicación de terceros de un proveedor que aún no forme parte de la galería de Azure AD. También puede [agregar la aplicación a la galería](../azuread-dev/howto-app-gallery-listing.md) si no está allí.

Por último, también puede integrar las aplicaciones que desarrolla de modo interno. Esto se trata en el paso cinco de esta guía.

## <a name="2-determine-existing-application-usage-and-prioritize-work"></a>2. Determinación del uso de la aplicación existente y priorización del trabajo

A continuación, detecte las aplicaciones que los empleados utilizan con frecuencia y priorice su trabajo para integrarlas con Azure AD.

Puede comenzar con las [herramientas de Cloud Discovery](/cloud-app-security/tutorial-shadow-it) de Microsoft Cloud App Security para detectar y administrar &quot;TI en la sombra&quot; en la red (es decir, las aplicaciones no administradas por el departamento de TI). Puede utilizar [Protección contra amenazas avanzada (ATP) de Microsoft Defender](/cloud-app-security/wdatp-integration) para simplificar y ampliar el proceso de detección.

Además, puede usar el [informe de actividad de la aplicación AD FS](../manage-apps/migrate-adfs-application-activity.md) en Azure Portal para detectar todas las aplicaciones de AD FS de su organización, el número de usuarios únicos que han iniciado sesión en ellas y la compatibilidad para integrarlas con Azure AD.

Una vez que haya descubierto el panorama existente, querrá [crear un plan](../manage-apps/migration-resources.md) y priorizar las aplicaciones con una prioridad más alta de integración. Algunas preguntas de ejemplo que puede realizar para guiar este proceso son las siguientes:

- ¿Qué aplicaciones son las más utilizadas?
- ¿Cuáles son las más arriesgadas?
- ¿Qué aplicaciones se retirarán en el futuro, lo que haría innecesario un traslado?
- ¿Qué aplicaciones deben seguir siendo locales y no se pueden mover a la nube?

Verá las mayores ventajas y el ahorro de costos una vez que todas las aplicaciones estén integradas y ya no dependa de varias soluciones de identidad. Sin embargo, podrá disfrutar de una administración de identidades más sencilla y una mayor seguridad a medida que avance escalonadamente hacia este objetivo. Quiere utilizar este tiempo para priorizar el trabajo y decidir lo que sea más significativo para su situación.

## <a name="3-integrate-apps-that-rely-on-other-identity-providers"></a>3. Integración de aplicaciones que dependen de otros proveedores de identidades

Durante el proceso de detección, es posible que haya encontrado aplicaciones sin seguimiento por parte del departamento de TI, lo que hace que los datos y recursos sean más vulnerables. También puede tener aplicaciones que usen soluciones de identidad alternativas, como Servicios de federación de Active Directory (AD FS) u otros proveedores de identidades. Piense en cómo puede consolidar la administración de identidades y accesos para ahorrar dinero y aumentar la seguridad. Al reducir el número de soluciones de identidad que tiene, podrá:

- Ahorrar dinero al eliminar la necesidad de aprovisionamiento y autenticación de usuarios locales, así como de las tarifas de licencias pagadas por otros proveedores de identidades en la nube para el mismo servicio.
- Reducir la sobrecarga administrativa y permitir una mayor seguridad con menos redundancias en el proceso de administración de identidades y accesos.
- Permitir a los empleados obtener acceso seguro de inicio de sesión único a TODAS las aplicaciones que necesiten mediante el portal [Mis aplicaciones](../manage-apps/access-panel-collections.md).
- Mejorar la inteligencia de los servicios relacionados con [protección de identidades](../identity-protection/overview-identity-protection.md) de Azure AD, como el acceso condicional mediante el aumento de la cantidad de datos que obtiene del uso de la aplicación, y ampliar las ventajas a las aplicaciones recién agregadas.

Hemos publicado una guía para administrar el proceso empresarial de integración de aplicaciones con Azure AD, incluido un [póster](https://aka.ms/AppOnePager) y una [presentación](https://aka.ms/AppGuideline) que puede utilizar para concienciar e interesar a los propietarios de las aplicaciones y del negocio. Puede modificar esos ejemplos con su propia personalización de marca y publicarlos en su organización en el portal de empresa, el boletín u otro medio a medida que vaya completando este proceso.

Un buen punto de partida es evaluar el uso de Servicios de federación de Active Directory (AD FS). Muchas organizaciones usan ADFS para la autenticación con aplicaciones SaaS, aplicaciones de línea de negocio personalizadas, así como aplicaciones basadas en Office 365 y Azure AD:

![En el diagrama se muestran aplicaciones locales, aplicaciones de línea de negocio, aplicaciones SaaS y, a través de Azure AD, Office 365, todos conectados por líneas de rayas a Active Directory y a AD FS.](\media\five-steps-to-full-application-integration-with-azure-ad\adfs-integration-1.png)

Puede actualizar esta configuración [reemplazando ADFS por Azure AD como centro](../manage-apps/migrate-adfs-apps-to-azure.md) de la solución de administración de identidades. Si lo hace, se habilita el inicio de sesión para todas las aplicaciones a las que los empleados deseen acceder y les facilita la búsqueda de cualquier aplicación empresarial que necesiten a través del portal [Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md), además de las otras ventajas mencionadas anteriormente.

![En el diagrama se muestran aplicaciones locales a través de Active Directory y AD FS, aplicaciones de línea de negocio, aplicaciones SaaS y Office 365, todos conectados por líneas de rayas a Azure Active Directory.](\media\five-steps-to-full-application-integration-with-azure-ad\adfs-integration-2.png)

Cuando Azure AD se convierte en el proveedor de identidades central, es posible que pueda cambiar desde ADFS totalmente, en lugar de utilizar una solución federada. Las aplicaciones que anteriormente usaban ADFS para la autenticación ahora pueden utilizar Azure AD independiente.

![En el diagrama se muestran aplicaciones locales, aplicaciones de línea de negocio, aplicaciones SaaS y Office 365, todos conectados por líneas de rayas a Azure Active Directory. Active Directory y AD FS no están presentes.](\media\five-steps-to-full-application-integration-with-azure-ad\adfs-integration-3.png)

También puede migrar aplicaciones que utilizan otro proveedor de identidades basado en la nube para Azure AD. Su organización puede tener varias soluciones de Administración de acceso a identidades (IAM). La migración a una infraestructura de Azure AD es una oportunidad para reducir las dependencias de las licencias de IAM (locales o en la nube) y los costos de infraestructura. En los casos en los que ya haya pagado Azure AD a través de licencias de M365, no hay razón para pagar el costo adicional de otra solución IAM.

## <a name="4-integrate-on-premises-applications"></a>4. Integración de aplicaciones locales

Tradicionalmente, las aplicaciones se mantenían seguras al permitir el acceso solo mientras estaban conectadas a la red corporativa. Sin embargo, en un mundo cada vez más conectado, queremos permitir el acceso a las aplicaciones a clientes, asociados o empleados, independientemente de dónde se encuentren en el mundo. [Azure AD Application Proxy](../manage-apps/what-is-application-proxy.md) (AppProxy) es una característica de Azure AD que conecta las aplicaciones locales existentes a Azure AD y no requiere mantener servidores perimetrales u otras infraestructuras adicionales para ello.

![Un diagrama muestra el servicio de Application Proxy en acción. Un usuario tiene acceso a "https://sales.contoso.com" y su solicitud se redirige mediante "https://sales-contoso.msappproxy.net" en Azure Active Directory a la dirección local "http://sales".](./media/five-steps-to-full-application-integration-with-azure-ad\app-proxy.png)

Puede usar el [Tutorial: Adición de una aplicación local para el acceso remoto mediante Application Proxy en Azure Active Directory](../manage-apps/application-proxy-add-on-premises-application.md) para habilitar Application Proxy y agregar una aplicación local a su inquilino de Azure AD.

Además, puede integrar controladores de entrega de aplicaciones como F5 BIG-IP APM o Zscaler Private Access. Al integrarlos con Azure AD, obtendrá la moderna autenticación y administración de identidades de Azure AD, junto con las características de seguridad y administración del tráfico del producto de los asociados. Llamamos a esta solución [Acceso híbrido seguro](../manage-apps/secure-hybrid-access.md). Si utiliza cualquiera de los siguientes servicios actualmente, tenemos tutoriales que le indicarán paso a paso cómo integrarlos con Azure AD.

- [Acceso de aplicaciones empresariales (EAA) de Akamai](../saas-apps/akamai-tutorial.md)
- [Citrix Application Deliver Controller (ADC)](../saas-apps/citrix-netscaler-tutorial.md) (anteriormente conocido como Citrix Netscaler)
- [APM BIG-IP de F5](../saas-apps/headerf5-tutorial.md)
- [Zscaler Private Access (ZPA)](../saas-apps/zscalerprivateaccess-tutorial.md)

## <a name="5-integrate-apps-your-developers-build"></a>5. Integración de las aplicaciones que compilan los desarrolladores

En el caso de las aplicaciones que se compilan en su empresa, los desarrolladores pueden utilizar la [plataforma de identidad de Microsoft](../develop/index.yml) para implementar la autenticación y la autorización. Las aplicaciones integradas con la plataforma [se registran con Azure AD](../develop/quickstart-register-app.md) y se administran igual que cualquier otra aplicación de su cartera.

Los desarrolladores pueden usar la plataforma para aplicaciones de uso interno y aplicaciones orientadas al cliente, y existen otras ventajas al utilizar la plataforma. Las [bibliotecas de autenticación de Microsoft (MSAL)](../develop/msal-overview.md), que forman parte de la plataforma, permiten a los desarrolladores habilitar experiencias modernas como autenticación multifactor y el uso de claves de seguridad para acceder a sus aplicaciones sin necesidad de implementarlas. Además, las aplicaciones integradas en la plataforma de identidad de Microsoft pueden acceder a [Microsoft Graph](../develop/microsoft-graph-intro.md): un punto de conexión de API unificado que proporciona los datos de Microsoft 365, que describen los patrones de productividad, identidad y seguridad de una organización. Los desarrolladores pueden utilizar esta información para implementar características que aumenten la productividad de los usuarios. Por ejemplo, mediante la identificación de las personas con las que el usuario ha estado interactuando recientemente y su exposición en la interfaz de usuario de la aplicación.

Disponemos de una [serie de vídeos](https://www.youtube.com/watch?v=zjezqZPPOfc&amp;list=PLLasX02E8BPBxGouWlJV-u-XZWOc2RkiX) que proporciona una introducción completa a la plataforma, así como [muchos ejemplos de código](../develop/sample-v2-code.md) en lenguajes y plataformas compatibles.

## <a name="next-steps"></a>Pasos siguientes

- [Recursos para migrar aplicaciones a Azure Active Directory](../manage-apps/migration-resources.md)