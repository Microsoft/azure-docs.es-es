---
title: 'Uso compartido de datos y consentimiento de LinkedIn: Azure Active Directory | Microsoft Docs'
description: Se explica cómo la integración de LinkedIn comparte datos a través de aplicaciones de Microsoft en Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 11/08/2019
ms.author: curtand
ms.reviewer: beengen
ms.custom: it-pro;seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4eaf0ab6cc412b7f5a02f898682e90fdde9de3a1
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542368"
---
# <a name="linkedin-account-connections-data-sharing-and-consent"></a>Consentimiento y uso compartido de datos de conexiones de cuentas de LinkedIn

Puede permitir que los usuarios de su organización de Active Directory (Azure AD) den su consentimiento para conectar su cuenta profesional o educativa de Microsoft con su cuenta de LinkedIn. Cuando los usuarios conectan sus cuentas, la información y los aspectos destacados de LinkedIn están disponibles en algunas aplicaciones y servicios de Microsoft. Los usuarios también pueden esperar que su experiencia de creación de contactos en LinkedIn mejore y se enriquezca con la información de Microsoft.

Para ver información de LinkedIn en aplicaciones y servicios de Microsoft, los usuarios deben dar su consentimiento para conectar sus propias cuentas de Microsoft y LinkedIn. Se pedirá a los usuarios que conecten sus cuentas la primera vez que hagan clic para ver la información de LinkedIn de otro usuario en una tarjeta de perfil de Outlook, OneDrive o SharePoint Online. Las conexiones de cuentas de LinkedIn no estarán habilitadas por completo para los usuarios hasta que den su consentimiento para la experiencia y para la conexión de sus cuentas.

[!INCLUDE [active-directory-gdpr-note](../../../includes/gdpr-hybrid-note.md)]

## <a name="benefits-of-sharing-linkedin-information"></a>Ventajas de compartir información de LinkedIn

El acceso a la información de LinkedIn desde los servicios y aplicaciones de Microsoft facilita a los usuarios conectarse, ponerse en contacto y crear relaciones profesionales con colegas, clientes y socios dentro y fuera de su organización. Los nuevos usuarios pueden ganar productividad más rápido al conectarse con sus colegas, aprender más sobre ellos y acceder fácilmente a más información. Este es un ejemplo de cómo aparece la información de LinkedIn en la tarjeta de perfil de las aplicaciones de Microsoft:

![Habilitación de la integración de LinkedIn en su organización](./media/linkedin-user-consent/display-example.png)

## <a name="enable-and-announce-linkedin-integration"></a>Habilitación y anuncio de la integración de LinkedIn

Debe ser un administrador de Azure Active Directory para administrar la opción de configuración de su organización. Puede habilitar la opción para todos los usuarios o para un conjunto específico de usuarios.

1. Para habilitar o deshabilitar la integración, siga los pasos que se indican en [Consentimiento a la integración de LinkedIn en la organización de Azure AD](linkedin-integration.md).
2. Al anunciar la integración de LinkedIn en su organización, dirija a los usuarios a las preguntas más frecuentes sobre [información de LinkedIn en las aplicaciones y servicios de Microsoft](https://support.office.com/article/about-linkedin-information-and-features-in-microsoft-apps-and-services-dc81cc70-4d64-4755-9f1c-b9536e34d381). En este artículo, se describe la información de LinkedIn que se muestra, [la privacidad y el uso compartido de datos](https://support.microsoft.com/office/your-data-ae9c08a7-4d06-45b5-a065-320a97bc1400), [cómo conectarse a las cuentas](https://support.microsoft.com/office/connect-your-linkedin-and-work-or-school-accounts-c7c245f2-fa56-4c9b-ba20-3fceb23c5772), etc.

Debe anunciar la integración de LinkedIn a los usuarios y proporcionarles toda la información relacionada con la [privacidad y el uso compartido de datos en la integración de LinkedIn](https://support.microsoft.com/office/your-data-ae9c08a7-4d06-45b5-a065-320a97bc1400). 

## <a name="user-consent-for-data-access-in-microsoft-and-linkedin"></a>Consentimiento del usuario para el acceso a datos en Microsoft y LinkedIn

Los datos de LinkedIn a los que se tiene acceso no se almacenan de forma permanente en los servicios de Microsoft. Los datos de Microsoft a los que se tiene acceso no se almacenan de forma permanente con LinkedIn, a excepción de los contactos. Los contactos de Microsoft se almacenan en LinkedIn hasta que los usuarios los eliminan, tal y como se describe en [eliminar contactos importados de LinkedIn](https://www.linkedin.com/help/linkedin/answer/43377).

Cuando los usuarios conectan sus cuentas, la información y los aspectos destacados de LinkedIn están disponibles en algunas aplicaciones de Microsoft, como la tarjeta de perfil. Los usuarios también pueden esperar que su experiencia de creación de contactos en LinkedIn mejore y se enriquezca con la información de Microsoft.
Cuando los usuarios de su organización conectan sus cuentas profesionales o educativas de LinkedIn y Microsoft, tienen dos opciones:

* Otorgar permisos para que ambas cuentas puedan acceder a los datos. Esto significa otorgar permiso a su cuenta profesional o de Microsoft para acceder a los datos de su cuenta de LinkedIn, y otorgar permiso a [su cuenta de LinkedIn para acceder a los datos de su cuenta profesional o educativa de Microsoft](https://www.linkedin.com/help/linkedin/answer/84077).
* Otorgar permisos únicamente para que su cuenta profesional o educativa de Microsoft pueda acceder a los datos de LinkedIn.

Los usuarios pueden desconectar las cuentas y eliminar los permisos de acceso a los datos en cualquier momento, y [pueden controlar el modo de visualización de su perfil de LinkedIn](https://www.linkedin.com/help/linkedin/answer/83), por ejemplo, si su perfil se puede ver en las aplicaciones de Microsoft.

### <a name="linkedin-account-data"></a>Datos de la cuenta de LinkedIn

Cuando se conectan las cuentas de Microsoft y LinkedIn, LinkedIn obtiene los permisos para proporcionar los siguientes datos a Microsoft:

* Datos del perfil: incluye la identidad e información de contacto de LinkedIn, así como la información que comparte con otros usuarios en su [perfil de LinkedIn](https://www.linkedin.com/help/linkedin/answer/15493).
* Datos sobre intereses: incluye sus intereses en LinkedIn, como las personas y temas que sigue, los cursos, los grupos y el contenido que le gusta y comparte.
* Datos de suscripciones: incluye las suscripciones a las aplicaciones y servicios de LinkedIn, junto con los datos asociados. 
* Los datos de conexiones: incluye su [red de LinkedIn](https://www.linkedin.com/help/linkedin/answer/110), con los perfiles y la información de contacto de sus conexiones de primer grado.

Los datos de LinkedIn a los que se tiene acceso no se almacenan de forma permanente en los servicios de Microsoft. Para obtener más información sobre el uso que da Microsoft a los datos personales, consulte la [declaración de privacidad de Microsoft](https://privacy.microsoft.com/privacystatement/).

### <a name="microsoft-work-or-school-account-data"></a>Datos de las cuentas profesionales o educativas de Microsoft

Cuando se conectan las cuentas de Microsoft y LinkedIn, Microsoft obtiene los permisos para proporcionar los siguientes datos a LinkedIn:

* Datos de perfil: incluye información como su nombre, apellido, foto de perfil, dirección de correo electrónico, administrador y las personas a las que administra.
* Datos del calendario: incluye las reuniones en sus calendarios, así como la hora, ubicación e información de contacto de los asistentes. En los datos del calendario no se incluye información acerca de la reunión, como la agenda, el contenido o el título.
* Datos de intereses: incluye los intereses asociados con su cuenta según su uso de los servicios de Microsoft, como Cortana y Bing para empresas.
* Datos de suscripciones: incluye las suscripciones proporcionadas por la organización a las aplicaciones y servicios de Microsoft, como Office 365.
* Datos de contactos: incluye listas de contactos en Outlook, Skype y otros servicios de la cuenta de Microsoft, como la información de contacto de las personas con las que se comunica o colabora frecuentemente. LinkedIn importa, almacena y usa de forma periódica los contactos, por ejemplo, para sugerir conexiones y para ayudar a organizar a los contactos y mostrar sus actualizaciones.

Los datos de Microsoft a los que se tiene acceso no se almacenan de forma permanente con LinkedIn, a excepción de los contactos. Los contactos de Microsoft se almacenan en LinkedIn hasta que los usuarios los eliminan. Obtenga más información sobre cómo [eliminar contactos importados de LinkedIn](https://www.linkedin.com/help/linkedin/answer/43377).

Para obtener más información sobre el uso que da LinkedIn a los datos personales, consulte la [política de privacidad de LinkedIn](https://www.linkedin.com/legal/privacy-policy). Con los servicios, la transferencia de datos y el almacenamiento de LinkedIn, los datos pueden fluir de la Unión Europea a los Estados Unidos y viceversa, y se protege su privacidad tal y como se describe en [Transferencias de datos de la UE](https://www.linkedin.com/help/linkedin/answer/62533).

## <a name="next-steps"></a>Pasos siguientes

* [LinkedIn en las aplicaciones de Microsoft con tu cuenta de la universidad o el trabajo](https://www.linkedin.com/help/linkedin/answer/84077)
