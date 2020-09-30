---
title: Integración con la Plataforma de identidad de Microsoft
description: Descubra las ventajas de integrar su aplicación con la Plataforma de identidad de Microsoft y obtenga recursos para características como el inicio de sesión simplificado, la administración de identidades, la autenticación multifactor y el control de acceso.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/08/2019
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev, seoapril2019
ms.openlocfilehash: bdeb27f9e761fefc52c9c97b28d15770e22221f4
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90706173"
---
# <a name="integrating-with-microsoft-identity-platform"></a>Integración con la Plataforma de identidad de Microsoft

[!INCLUDE [active-directory-devguide](../../../includes/active-directory-devguide.md)]

En este artículo obtendrá información sobre las ventajas de integrar la aplicación con la Plataforma de identidad de Microsoft y obtendrá recursos para la integración. La Plataforma de identidad de Microsoft y Azure Active Directory (AD) proporcionan a las organizaciones administración de identidades de clase empresarial para las aplicaciones en la nube. La integración con la Plataforma de identidad de Microsoft ofrece a los usuarios una experiencia de inicio de sesión simplificada y ayuda a que la aplicación se ajuste a la directiva de TI.

## <a name="how-to-integrate"></a>Integración

Hay varias maneras de integrar la aplicación con la Plataforma de identidad de Microsoft. Aprovechar muchos o algunos de estos escenarios es apropiado para su aplicación.

### <a name="support-microsoft-identity-platform-as-a-way-to-sign-in-to-your-application"></a>Compatibilidad con la Plataforma de identidad de Microsoft como forma de iniciar sesión en la aplicación

**Reduzca la fricción de inicio de sesión y reduzca los costes de soporte técnico.** Al usar la Plataforma de identidad de Microsoft para iniciar sesión en la aplicación, los usuarios no tendrán un nombre y una contraseña más que recordar. Como desarrollador, tendrá una contraseña menos que almacenar y proteger. No tener que administrar restablecimientos de contraseñas olvidadas puede considerarse en sí un ahorro notable. La Plataforma de identidad de Microsoft se utiliza en el inicio de sesión de algunas de las aplicaciones en la nube más populares del mundo, incluido Microsoft 365 y Microsoft Azure. Con cientos de millones de usuarios de millones de organizaciones, lo más probable es que el usuario ya haya iniciado sesión en la Plataforma de identidad de Microsoft. Obtenga información sobre cómo [agregar compatibilidad con el inicio de sesión de la Plataforma de identidad de Microsoft](./authentication-vs-authorization.md).

**Simplifique el registro de la aplicación.**  Durante el registro de la aplicación, la Plataforma de identidad de Microsoft puede enviar información esencial acerca de un usuario para que pueda rellenar previamente el formulario de registro o eliminarlo completamente. Los usuarios pueden registrarse en la aplicación con su cuenta de Azure AD a través de una experiencia familiar de consentimiento familiar similar a las que se encuentran en las redes sociales y en las aplicaciones móviles. Cualquier usuario puede registrarse e iniciar sesión en una aplicación que está integrada con la Plataforma de identidad de Microsoft sin necesidad de la participación del departamento de TI. Más información sobre el [registro de la aplicación para el inicio de sesión con la cuenta de Azure AD](../../app-service/configure-authentication-provider-aad.md).

### <a name="browse-for-users-manage-user-provisioning-and-control-access-to-your-application"></a>Búsqueda de usuarios, administración del aprovisionamiento de usuarios y control del acceso a la aplicación

**Busque usuarios en el directorio.**  Use Microsoft Graph API para ayudar a los usuarios a buscar y examinar a otras personas de su organización cuando inviten a otras o concedan acceso, en lugar de solicitarles que escriban direcciones de correo electrónico. Los usuarios pueden examinar con una interfaz de estilo agenda familiar, incluida la visualización de información de la jerarquía organizacional. Obtenga más información acerca de [Microsoft Graph API](/graph/overview).

**Vuelva a usar las listas de distribución y grupos de Active Directory que el cliente ya está administrando.**  Azure AD contiene los grupos que el cliente ya está usando para la distribución de correo electrónico y la administración de acceso. Use Microsoft Graph API para volver a usar estos grupos en lugar de solicitar al cliente que cree y administre un conjunto independiente de grupos en su aplicación. La información de grupo también puede enviarse a la aplicación en tokens de inicio de sesión. Obtenga más información acerca de [Microsoft Graph API](/graph/overview).

**Use la Plataforma de identidad de Microsoft para controlar quién tiene acceso a la aplicación.**  Los administradores y propietarios de aplicaciones en Azure AD pueden asignar acceso a las aplicaciones para usuarios y grupos específicos. Con Microsoft Graph API, puede leer esta lista y usarla para controlar el aprovisionamiento y la cancelación de aprovisionamiento de recursos y accesos dentro de la aplicación.

**Use la Plataforma de identidad de Microsoft para el control de acceso basado en roles.**  Los administradores y propietarios de aplicaciones pueden asignar usuarios y grupos a los roles que se definen al registrar la aplicación en la Plataforma de identidad de Microsoft. La información del rol se envía a la aplicación en los tokens de inicio de sesión y también se puede leer a través de Microsoft Graph API. Más información sobre el [Uso de la Plataforma de identidad de Microsoft para la autorización](https://cloudblogs.microsoft.com/enterprisemobility/2014/12/18/azure-active-directory-now-with-group-claims-and-application-roles/).

### <a name="get-access-to-users-profile-calendar-email-contacts-files-and-more"></a>Obtenga acceso al perfil del usuario, el calendario, el correo electrónico, los contactos, los archivos, etc.

**La Plataforma de identidad de Microsoft es el servidor de autorización para Microsoft 365 y otros servicios empresariales de Microsoft.**  Si admite la Plataforma de identidad de Microsoft para el inicio de sesión en la aplicación o admite la vinculación de las cuentas de usuario actuales con cuentas de usuario de Azure AD mediante OAuth 2.0, puede solicitar acceso de lectura y escritura al perfil, el calendario, el correo electrónico, los contactos, los archivos y otra información del usuario. Puede escribir sin problemas eventos para el calendario de usuario y leer o escribir  archivos en OneDrive. Obtenga más información sobre el [acceso a la API de Microsoft 365](/previous-versions/office/office-365-api/).

### <a name="promote-your-application-in-the-azure-and-microsoft-365-marketplaces"></a>Promueva su aplicación en Azure Marketplace y el Catálogo de soluciones de Microsoft 365

**Promueva la aplicación en millones de organizaciones que ya están utilizando Azure AD.**  Los usuarios que buscan y examinan estos catálogos de soluciones ya están usando uno o más servicios en la nube, lo que los convierte en clientes de servicio en la nube cualificados. Más información acerca de la promoción de la aplicación en [Azure Marketplace](https://azure.microsoft.com/marketplace/partner-program/).

**Cuando los usuarios registrar la aplicación, aparecerá en el panel de acceso de Azure AD y en el iniciador de aplicaciones de Microsoft 365.**  Los usuarios podrán volver de forma rápida y sencilla a la aplicación más tarde y mejorar la afiliación del usuario. Más información sobre el [panel de acceso de Azure AD](../user-help/my-apps-portal-end-user-access.md).

### <a name="secure-device-to-service-and-service-to-service-communication"></a>Comunicación segura de dispositivo a servicio y de servicio a servicio

**El uso de la Plataforma de identidad de Microsoft para la administración de identidades de servicios y dispositivos reduce el código que es necesario escribir y permite al departamento de TI administrar el acceso.**  Los servicios y los dispositivos pueden obtener tokens de la Plataforma de identidad de Microsoft con OAuth y usar esos tokens para obtener acceso a las API web. Con la Plataforma de identidad de Microsoft puede evitar escribir código de autenticación complejo. Dado que las identidades de los servicios y los dispositivos se almacenan en Azure AD, la TI puede administrar las claves y la revocación en un solo lugar en lugar de tener que hacerlo por separado en la aplicación.

## <a name="benefits-of-integration"></a>Ventajas de la integración

La integración con la Plataforma de identidad de Microsoft conlleva ventajas que no requieren que se escriba código adicional.

### <a name="integration-with-enterprise-identity-management"></a>Integración con administración de identidades empresariales

**Ayude a su aplicación a cumplir las políticas de TI.**  Las organizaciones integran sus sistemas de administración de identidades empresariales con la Plataforma de identidad de Microsoft, por lo que cuando una persona deja una organización, perderá automáticamente el acceso a la aplicación sin necesidad de que el departamento de TI realice pasos adicionales. La TI puede administrar quién puede tener acceso a la aplicación y determinar qué directivas de acceso son necesarias, para Multi-factor Authentication por ejemplo,  reduce la necesidad de escribir código para cumplir con las directivas corporativas complejas. Azure AD proporciona a los administradores un registro detallado de auditoría de quién inicia sesión en su aplicación, por lo que la TI puede realizar un seguimiento del uso.

**Azure AD amplía Active Directory a la nube para que la aplicación pueda integrarse con AD.**  Muchas organizaciones en todo el mundo utilizan Active Directory como su sistema de administración de identidades y de inicio de sesión de entidad de seguridad y requiere que sus aplicaciones trabajen con AD. Integración con Azure AD para la integración de la aplicación con Active Directory

### <a name="advanced-security-features"></a>Características de seguridad avanzadas

**Multi-factor authentication.**  La Plataforma de identidad de Microsoft proporciona autenticación multifactor nativa. Los administradores de TI pueden requerir Multi-factor Authentication para tener acceso a la aplicación, por lo que no tiene que codificar este soporte técnico usted mismo. Obtenga más información sobre [Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/).

**Detección de inicio de sesión erróneo.**  La Plataforma de identidad de Microsoft procesa más de mil millones de inicios de sesión al día y además usa algoritmos de aprendizaje automático para detectar actividades sospechosas y notificar a los administradores de TI los posibles problemas. Al admitir el inicio de sesión de la Plataforma de identidad de Microsoft, la aplicación obtiene las ventajas de esta protección. Más información sobre la [visualización del informe de acceso de Azure Active Directory](../reports-monitoring/overview-reports.md).

**Acceso condicional.**  Además de Multi-factor Authentication, los administradores pueden solicitar que se cumplan condiciones específicas para que los usuarios puedan iniciar sesión en la aplicación. Las condiciones que se pueden establecer incluyen el intervalo de direcciones IP de los dispositivos cliente, la pertenencia a grupos especificados y el estado del dispositivo que se utiliza para el acceso. Más información sobre el [acceso condicional de Azure Active Directory](../conditional-access/overview.md).

### <a name="easy-development"></a>Desarrollo sencillo

**Protocolos estándar del sector.**  Microsoft se ha comprometido a admitir los estándares del sector. La Plataforma de identidad de Microsoft admite los protocolos OAuth 2.0 y OpenID Connect 1.0 estándar del sector. Obtenga más información sobre los [protocolos de autenticación de la Plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

**Abra las bibliotecas de código abierto.**  Microsoft proporciona bibliotecas de código abierto totalmente compatibles para plataformas y lenguajes conocidos para acelerar el desarrollo. El código fuente tiene una licencia de Apache 2.0 y puede realizar la bifurcación y contribución de nuevo a los proyectos. Obtenga más información sobre la [Biblioteca de autenticación de Microsoft (MSAL)](reference-v2-libraries.md).

### <a name="worldwide-presence-and-high-availability"></a>Alta disponibilidad y presencia en todo el mundo

**Azure AD se implementa en centros de datos en todo el mundo y se administra y supervisa continuamente.**  Azure AD es el sistema de administración de identidades de Microsoft Azure y Microsoft 365 y se implementa en 28 centros de datos en todo el mundo. Se garantiza que los datos de directorio se replican en al menos tres centros de datos. Los equilibradores de carga global garantizan el acceso de los usuarios a la copia más cercana de Azure AD que contiene sus datos y vuelve a dirigir automáticamente las solicitudes a otros centros de datos si se detecta un problema.

## <a name="next-steps"></a>Pasos siguientes

[Inicio de escritura de código](v2-overview.md#getting-started).

[Inicio de sesión de usuarios con la Plataforma de identidad de Microsoft](./authentication-vs-authorization.md)