---
title: Cómo integrar con Azure Active Directory | Microsoft Docs
description: Descubra las ventajas de integrar su aplicación con Azure Active Directory y obtenga recursos para características como la administración simplificada de inicio de sesión de identidad, autenticación multifactor y control de acceso.
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: d13bba54-96bd-4b81-bee9-c8025ffa1648
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/08/2019
ms.author: ryanwi
ms.reviewer: bryanla
ms.custom: aaddev, seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: c8804988387ba4a6cdc8dff5d93077de42ea2051
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546191"
---
# <a name="integrating-with-azure-active-directory"></a>Integración con Azure Active Directory

[!INCLUDE [active-directory-devguide](../../../includes/active-directory-devguide.md)]

En este artículo, obtenga información sobre las ventajas de integrar su aplicación con Azure Active Directory (Azure AD) y obtenga recursos para la integración. Azure AD proporciona las organizaciones con administración de identidades de nivel empresarial para aplicaciones en la nube. La integración de Azure AD ofrece a los usuarios una experiencia de inicio de sesión simplificada y ayuda a que la aplicación se ajuste a la directiva de TI.

## <a name="how-to-integrate"></a>Cómo integrar

Hay varias maneras de integrar la aplicación con Azure AD. Aprovechar muchos o algunos de estos escenarios es apropiado para su aplicación.

### <a name="support-azure-ad-as-a-way-to-sign-in-to-your-application"></a>Soporte técnico de Azure AD como una manera de iniciar sesión su aplicación

**Reduzca la fricción de inicio de sesión y reduzca los costes de soporte técnico.**  Mediante el uso de Azure AD para iniciar sesión en la aplicación, los usuarios no tendrán un nombre y una contraseña más que recordar. Como desarrollador, tendrá una contraseña menos que almacenar y proteger. No tener que administrar restablecimientos de contraseñas olvidadas puede considerarse en sí un ahorro notable. Azure AD acciona el inicio de sesión para algunas aplicaciones en la nube más populares del mundo, incluido Office 365 y Microsoft Azure. Con cientos de millones de usuarios de millones de organizaciones, lo más probable es que el usuario ya haya iniciado sesión en Azure AD. Más información sobre la [adición de compatibilidad para el inicio de sesión de Azure AD](authentication-scenarios.md).

**Simplifique el registro de la aplicación.**   Durante el registro de la aplicación, Azure AD puede enviar información esencial acerca de un usuario para que pueda rellenar previamente el formulario de registro o eliminarlo completamente. Los usuarios pueden registrarse en la aplicación con su cuenta de Azure AD a través de una experiencia familiar de consentimiento familiar similar a las que se encuentran en las redes sociales y en las aplicaciones móviles. Cualquier usuario puede registrarse e iniciar sesión en una aplicación que se integra con Azure AD sin necesidad de la participación de TI. Más información sobre el [registro de la aplicación para el inicio de sesión con la cuenta de Azure AD](../../app-service/configure-authentication-provider-aad.md).

### <a name="browse-for-users-manage-user-provisioning-and-control-access-to-your-application"></a>Búsqueda de usuarios, administrar el aprovisionamiento de usuarios y controlar el acceso a la aplicación

**Busque usuarios en el directorio.**   Utilice Graph API para ayudar a los usuarios a buscar y examinar otras personas en su organización cuando inviten a otras o concedan acceso, en lugar de solicitarles que escriban direcciones de correo electrónico. Los usuarios pueden examinar con una interfaz de estilo agenda familiar, incluida la visualización de información de la jerarquía organizacional. Más información acerca de [Graph API](active-directory-graph-api.md).

**Vuelva a usar las listas de distribución y grupos de Active Directory que el cliente ya está administrando.**   Azure AD contiene los grupos que el cliente ya está usando para la distribución de correo electrónico y la administración de acceso. Utilice Graph API para volver a utilizar estos grupos en lugar de solicitar al cliente que cree y administre un conjunto independiente de grupos en su aplicación. La información de grupo también puede enviarse a la aplicación en tokens de inicio de sesión. Más información acerca de [Graph API](active-directory-graph-api.md).

**Use Azure AD para controlar quién tiene acceso a la aplicación.**   Los administradores y propietarios de aplicaciones en Azure AD pueden asignar acceso a las aplicaciones para usuarios y grupos específicos. Con Graph API, puede leer esta lista y usarla para controlar el aprovisionamiento y la cancelación de aprovisionamiento de recursos y obtener acceso dentro de la aplicación.

**Utilice Azure AD para roles basados en el control de acceso.**   Los administradores y propietarios de aplicaciones pueden asignar usuarios y grupos a los roles que se definen al registrar la aplicación en Azure AD. La información del rol se envía a la aplicación en los tokens de inicio de sesión y también se puede leer utilizando Graph API. Obtenga más información sobre [uso de Azure AD para la autorización](https://cloudblogs.microsoft.com/enterprisemobility/2014/12/18/azure-active-directory-now-with-group-claims-and-application-roles/).

### <a name="get-access-to-users-profile-calendar-email-contacts-files-and-more"></a>Obtener acceso del usuario perfil, calendario, correo electrónico, contactos, archivos etc.

**Azure AD es el servidor de autorización para Office 365 y otros servicios empresariales de Microsoft.**   Si admite Azure AD para el inicio de sesión en la aplicación o admite la vinculación de cuentas de usuario actuales con cuentas de usuario de Azure AD mediante OAuth 2.0, puede solicitar acceso de lectura y escritura a un perfil de usuario, al calendario, al correo electrónico, a archivos y a otra información. Puede escribir sin problemas eventos para el calendario de usuario y leer o escribir  archivos en OneDrive. Obtenga más información sobre el [acceso a la API de Office 365](https://msdn.microsoft.com/office/office365/howto/platform-development-overview).

### <a name="promote-your-application-in-the-azure-and-office-365-marketplaces"></a>Promueva su aplicación en Azure y catálogos de soluciones de Office 365

**Promueva la aplicación en millones de organizaciones que ya están utilizando Azure AD.**   Los usuarios que buscan y examinan estos catálogos de soluciones ya están usando uno o más servicios en la nube, lo que los convierte en clientes de servicio en la nube cualificados. Más información acerca de la promoción de la aplicación en [Azure Marketplace](https://azure.microsoft.com/marketplace/partner-program/).

**Cuando los usuarios registrar la aplicación, aparecerá en el panel de acceso de Azure AD y en el iniciador de aplicaciones de Office 365.**   Los usuarios podrán volver de forma rápida y sencilla a la aplicación más tarde y mejorar la afiliación del usuario. Más información sobre el [panel de acceso de Azure AD](../user-help/active-directory-saas-access-panel-introduction.md).

### <a name="secure-device-to-service-and-service-to-service-communication"></a>Proteger la comunicación de servicio de dispositivos y servicios:

**El uso de Azure AD para la administración de identidades de servicios y dispositivos reduce el código que tiene que escribir y permite a la TI administrar el acceso.**   Los servicios y dispositivos pueden obtener tokens de  Azure AD con OAuth y usar esos tokens para obtener acceso a API web. Con Azure AD puede evitar escribir código de autenticación complejo. Dado que las identidades de los servicios y los dispositivos se almacenan en Azure AD, la TI puede administrar las claves y la revocación en un solo lugar en lugar de tener que hacerlo por separado en la aplicación.

## <a name="benefits-of-integration"></a>Ventajas de la integración

La integración con Azure AD conlleva beneficios que no requieren que escriba código adicional.

### <a name="integration-with-enterprise-identity-management"></a>Integración con administración de identidades empresariales

**Ayude a su aplicación a cumplir las políticas de TI.**   Las organizaciones integran sus sistemas de administración de identidades empresariales con Azure AD, por lo que cuando una persona deja una organización,  perderá automáticamente el acceso a la aplicación sin necesidad de realizar pasos adicionales en la TI. La TI puede administrar quién puede tener acceso a la aplicación y determinar qué directivas de acceso son necesarias, para Multi-factor Authentication por ejemplo,  reduce la necesidad de escribir código para cumplir con las directivas corporativas complejas. Azure AD proporciona a los administradores un registro detallado de auditoría de quién inicia sesión en su aplicación, por lo que la TI puede realizar un seguimiento del uso.

**Azure AD amplía Active Directory a la nube para que la aplicación pueda integrarse con AD.**   Muchas organizaciones en todo el mundo utilizan Active Directory como su sistema de administración de identidades y de inicio de sesión de entidad de seguridad y requiere que sus aplicaciones trabajen con AD. Integración con Azure AD para la integración de la aplicación con Active Directory

### <a name="advanced-security-features"></a>Características de seguridad avanzadas

**Multi-factor authentication.**   Azure AD proporciona Multi-factor Authentication nativa. Los administradores de TI pueden requerir Multi-factor Authentication para tener acceso a la aplicación, por lo que no tiene que codificar este soporte técnico usted mismo. Obtenga más información sobre [Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/).

**Detección de inicio de sesión erróneo.**   Azure AD procesa más de mil millones de inicios de sesión al día, y además usa algoritmos de aprendizaje automático para detectar actividades sospechosas e informar a los administradores de TI de posibles problemas. Al admitir el inicio de sesión de Azure AD, la aplicación obtiene la ventaja de esta protección. Más información sobre la [visualización del informe de acceso de Azure Active Directory](../active-directory-view-access-usage-reports.md).

**Acceso condicional.**   Además de Multi-factor Authentication, los administradores pueden solicitar que se cumplan condiciones específicas para que los usuarios puedan iniciar sesión en la aplicación. Las condiciones que se pueden establecer incluyen el intervalo de direcciones IP de los dispositivos cliente, la pertenencia a grupos especificados y el estado del dispositivo que se utiliza para el acceso. Más información sobre el [acceso condicional de Azure Active Directory](../active-directory-conditional-access-azure-portal.md).

### <a name="easy-development"></a>Desarrollo sencillo

**Protocolos estándar del sector.**   Microsoft se ha comprometido a admitir los estándares del sector. La plataforma Microsoft identity es compatible con los protocolos de OAuth 2.0 y OpenID Connect 1.0 estándar del sector. Obtenga más información sobre [protocolos de autenticación de plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

**Abra las bibliotecas de código abierto.**   Microsoft proporciona bibliotecas de código abierto totalmente compatibles para plataformas y lenguajes conocidos para acelerar el desarrollo. El código fuente tiene una licencia de Apache 2.0 y puede realizar la bifurcación y contribución de nuevo a los proyectos. Obtenga más información sobre [biblioteca de autenticación de Microsoft (MSAL)](reference-v2-libraries.md).

### <a name="worldwide-presence-and-high-availability"></a>Alta disponibilidad y presencia en todo el mundo

**Azure AD se implementa en centros de datos en todo el mundo y se administra y supervisa continuamente.**   Azure AD es el sistema de administración de identidades de Microsoft Azure y Office 365 y se implementa en 28 centros de datos en todo el mundo. Se garantiza que los datos de directorio se replican en al menos tres centros de datos. Los equilibradores de carga global garantizan el acceso de los usuarios a la copia más cercana de Azure AD que contiene sus datos y vuelve a dirigir automáticamente las solicitudes a otros centros de datos si se detecta un problema.

## <a name="next-steps"></a>Pasos siguientes

[Inicio de escritura de código](v2-overview.md#getting-started).

[Inicio de sesión a los usuarios con la plataforma Microsoft identity](authentication-scenarios.md)

