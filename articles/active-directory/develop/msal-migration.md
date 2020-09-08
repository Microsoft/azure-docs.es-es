---
title: Migración a la Biblioteca de autenticación de Microsoft (MSAL)
titleSuffix: Microsoft identity platform
description: Conozca las diferencias entre la Biblioteca de autenticación de Microsoft (MSAL) y la Biblioteca de Autenticación de Azure AD (ADAL) y cómo migrar a MSAL.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/07/2020
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: ac9264ea8d6cc71d19d2c9bbd23b2123bdf1f924
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88224363"
---
# <a name="migrate-applications-to-microsoft-authentication-library-msal"></a>Migración de aplicaciones a la Biblioteca de autenticación de Microsoft (MSAL)

Muchos desarrolladores han creado e implementado aplicaciones mediante la Biblioteca de autenticación de Azure Active Directory (ADAL). Ahora se recomienda usar la Biblioteca de autenticación de Microsoft (MSAL) para la autenticación y autorización de entidades Azure AD.

Al usar MSAL en lugar de ADAL:

- Puede autenticar un conjunto más amplio de identidades:
  - Identidades de Azure AD
  - Cuentas de Microsoft
  - Cuentas locales y sociales mediante Azure AD B2C
- Los usuarios obtendrán la mejor experiencia de inicio de sesión único.
- La aplicación puede habilitar el consentimiento incremental.
- Es más fácil admitir el acceso condicional.
- Se beneficia de la innovación. Dado que todos los esfuerzos de desarrollo de Microsoft están ahora centrados en MSAL, no se implementarán nuevas características en ADAL.

**MSAL es ahora la biblioteca de autenticación recomendada para la plataforma de identidad de Microsoft**.

## <a name="migration-guidance"></a>Guía de migración

Los siguientes artículos pueden ayudarle a migrar a MSAL:

- [Migración a MSAL.Android](migrate-android-adal-msal.md)
- [Migración a MSAL.iOS o macOS](migrate-objc-adal-msal.md)
- [Migración a MSAL para Java](migrate-adal-msal-java.md)
- [Migración a MSAL.js](msal-compare-msal-js-and-adal-js.md)
- [Migración a MSAL.NET](msal-net-migration.md)
- [Migración a MSAL para Python](migrate-python-adal-msal.md)
- [Migración de aplicaciones de Xamarin mediante agentes a MSAL.NET](msal-net-migration-ios-broker.md)

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

__P: ¿ADAL se ha quedado en desuso?__  
A. Sí. A partir del 30 de junio de 2020, ya no se agregarán nuevas características a ADAL. Seguiremos agregando correcciones de seguridad críticas a ADAL hasta el 30 de junio de 2022. Después de esta fecha, las aplicaciones que usan ADAL seguirán funcionando, pero se recomienda actualizar a MSAL para aprovechar las características más recientes y mantener la seguridad.

__P: ¿Las aplicaciones de ADAL actuales seguirán funcionando?__  
A. No. Las aplicaciones existentes seguirán funcionando sin modificaciones. Si tiene previsto mantenerlas más allá del 30 de junio de 2022, considere la posibilidad de actualizarlas a MSAL para mantenerlas seguras, aunque no es necesario migrarlas a MSAL para mantener la funcionalidad existente.

__P: ¿Cómo puedo saber qué aplicaciones están usando ADAL?__  
A. Si tiene el código fuente de la aplicación, puede consultar las guías de migración anteriores para obtener ayuda para la determinación de la biblioteca que usa la aplicación y su migración a MSAL. Si se ha asociado con un ISV, le recomendamos que se ponga en contacto con ellos directamente para comprender el proceso de migración a MSAL.

__P: ¿Por qué debo invertir en la migración a MSAL?__  
A. MSAL contiene nuevas características que no se incluyen en ADAL, como el consentimiento incremental, el inicio de sesión único y la administración de la caché de tokens. Además, a diferencia de ADAL, MSAL seguirá recibiendo parches de seguridad después del 30 de junio de 2022. [Más información](msal-overview.md).

__P: ¿Tiene previsto Microsoft actualizar sus aplicaciones a MSAL?__  
Sí. Microsoft está migrando sus aplicaciones a MSAL y tiene previsto concluir el proceso para la fecha de finalización del soporte técnico. De este modo, las aplicaciones se beneficiarán de las mejoras continuas de las características y la seguridad de MSAL.

__P: ¿Se va a publicar una herramienta que me ayude a migrar las aplicaciones de ADAL a MSAL?__  
A. No. Las diferencias entre las bibliotecas requerirían dedicar recursos al desarrollo y mantenimiento de la herramienta que preferimos invertir en mejorar MSAL. No obstante, ponemos a disposición de los usuarios el conjunto anterior de guías de migración como ayuda para realizar los cambios necesarios en la aplicación.

__P: ¿Cómo funciona MSAL con AD FS?__  
A. MSAL.NET admite ciertos escenarios para autenticarse en AD FS 2019. Si la aplicación necesita adquirir tokens directamente desde una versión anterior de AD FS, debe permanecer en ADAL. [Más información](msal-net-adfs-support.md).

__P: ¿Cómo puedo obtener ayuda para migrar mi aplicación?__  
A. Consulte la sección [Guía de migración](#migration-guidance) de este artículo. Si después de leer la guía de la plataforma de la aplicación le quedan preguntas, puede publicar en Stack Overflow con la etiqueta `[adal-deprecation]` o abrir una incidencia en el repositorio de GitHub de la biblioteca. Vea la sección [Lenguajes y plataformas](msal-overview.md#languages-and-frameworks) del artículo información general de MSAL para obtener vínculos al repositorio de cada biblioteca.

## <a name="next-steps"></a>Pasos siguientes

- [Actualización de las aplicaciones para usar la Biblioteca de autenticación de Microsoft y Microsoft Graph API](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363)
- [Información general sobre la Plataforma de identidad de Microsoft](v2-overview.md)
- [Revise nuestros ejemplos de código de MSAL.](sample-v2-code.md)
