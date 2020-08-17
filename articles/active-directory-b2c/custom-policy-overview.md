---
title: Directivas personalizadas de Azure Active Directory B2C | Microsoft Docs
description: Información acerca de las directivas personalizadas de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 06/06/2019
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 043cf83d804803e1b5b47d3ac51bbccaa06e4e87
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116426"
---
# <a name="custom-policies-in-azure-active-directory-b2c"></a>Directivas personalizadas de Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Las directivas personalizadas son archivos de configuración que definen el comportamiento del inquilino de Azure Active Directory B2C (Azure AD B2C). Los flujos de usuario están predefinidos en el portal de Azure AD B2C para las tareas de identidad más comunes. Un desarrollador de identidades puede editar completamente las directivas personalizadas para completar muchas tareas distintas.

## <a name="comparing-user-flows-and-custom-policies"></a>Comparación de las directivas personalizadas y los flujos de usuario

| Context | Flujos de usuario | Directivas personalizadas |
|-|-------------------|-----------------|
| Usuarios de destino | Todos los desarrolladores de aplicaciones con o sin conocimientos sobre la identidad. | Profesionales de la identidad, integradores de sistemas, consultores y equipos internos de identidad. Se sienten cómodos con los flujos de OpenID Connect y comprenden a los proveedores de identidades y la autenticación basada en notificaciones. |
| Método de configuración | Azure Portal con una interfaz de usuario (UI) fácil de usar. | Edición directa de archivos XML y posterior carga en Azure Portal. |
| Personalización de la interfaz de usuario | Personalización completa de la interfaz de usuario, incluido HTML, CSS y JavaScript.<br><br>Compatibilidad multilingüe con cadenas personalizadas. | Iguales |
| Personalización de atributos | Atributos estándar y personalizados. | Iguales |
| Administración de tokens y sesiones | Varias opciones de sesiones y tokens personalizados. | Iguales |
| Proveedores de identidades | Proveedor local o social predefinido y la mayoría de los proveedores de identidades de OIDC, como la federación con inquilinos de Azure Active Directory. | OIDC, OAUTH y SAML basados en estándares.  También es posible la autenticación mediante la integración con API REST. |
| Tareas de identidad | Registro o inicio de sesión con muchas cuentas locales y de redes sociales.<br><br>Restablecimiento de la contraseña de autoservicio.<br><br>Edición de perfil.<br><br>Autenticación multifactor.<br><br>Sesiones y tokens personalizados.<br><br>Flujos de token de acceso. | Completar las mismas tareas que con los flujos de usuario usando proveedores de identidades personalizados o usar ámbitos personalizados.<br><br>Aprovisionar una cuenta de usuario en otro sistema en el momento del registro.<br><br>Enviar un mensaje de bienvenida con su propio proveedor de servicios de correo electrónico.<br><br>Usar un almacén de usuario externo a Azure AD B2C.<br><br>Validar la información proporcionada por el usuario con un sistema de confianza mediante el uso de una API. |

## <a name="policy-files"></a>Archivos de directivas

Se usan estos tres tipos de archivos de directivas:

- **Archivo base**: contiene la mayoría de las definiciones. Se recomienda que haga un mínimo de cambios en este archivo para facilitar la solución de problemas y el mantenimiento a largo plazo de las directivas.
- **Archivo de extensiones**: contiene los cambios de configuración únicos para el inquilino.
- **Un archivo de usuario de confianza (RP)** : el archivo centrado en una tarea única que la aplicación o el servicio (también conocido como usuario de confianza) invoca directamente. Cada tarea única requiere su propio usuario de confianza y, en función de los requisitos de marca, el número podría ser "total de aplicaciones x número total de casos de uso".

Los flujos de usuario de Azure AD B2C siguen el patrón de archivos mencionado, pero el desarrollador solo ve el archivo de usuario de confianza, mientras que Azure Portal hace cambios en el archivo de extensiones en segundo plano.

Aunque hay tres tipos de archivos de directiva, no está restringido a usar solo tres archivos. Puede tener varios archivos de cada tipo. Por ejemplo, si no quiere realizar cambios en el archivo Extensions, puede crear un archivo Extensions2 para ampliar aún más el archivo Extensions.

## <a name="custom-policy-core-concepts"></a>Conceptos básicos de directivas personalizadas

El servicio de administración de identidades y acceso de cliente (CIAM) de Azure incluye:

- Un directorio de usuarios accesible a través de Microsoft Graph y que contiene los datos de usuario tanto para cuentas locales como para cuentas federadas.
- Acceso al **Marco de experiencia de identidad** que orquesta la confianza entre usuarios y entidades, y transmite notificaciones entre ellos para completar una tarea de administración de identidades o acceso.
- Un servicio de token de seguridad (STS) que emite tokens de identificación, tokens de actualización y tokens de acceso (y aserciones SAML equivalentes) y los valida con el fin de proteger los recursos.

Azure AD B2C interactúa en secuencia con proveedores de identidades, usuarios, otros sistemas y con el directorio local de usuario para completar una tarea de identidad. Por ejemplo, iniciar sesión en un cliente, registrar un nuevo usuario o restablecer una contraseña. El marco de experiencia de identidad y una directiva (que también se conoce como una directiva de marco de confianza o de recorrido del usuario) crea confianza entre varios usuarios y define explícitamente los actores, las acciones, los protocolos y la secuencia de pasos que se van a completar.

Identity Experience Framework es una plataforma de Azure basada en la nube, controlada por directivas y completamente configurable que orquesta la confianza entre entidades en formatos de protocolo estándar como OpenID Connect, OAuth, SAML y algunos no estándar (como intercambios de notificaciones sistema a sistema basados en API de REST, por ejemplo). El marco crea experiencias propias fáciles de usar que admiten HTML y CSS.

Una directiva personalizada se representa como uno o varios archivos con formato XML que se hacen referencia entre sí en una cadena jerárquica. Los elementos XML definen el esquema de notificaciones, las transformaciones de notificaciones, las definiciones de contenido, los proveedores de notificaciones, los perfiles técnicos y los pasos de orquestación del recorrido del usuario, entre otros elementos. Una directiva personalizada es accesible como uno o varios archivos XML que el marco de experiencia de identidad ejecuta cuando lo invoca un usuario de confianza. Los desarrolladores que configuran las directivas personalizadas deben definir las relaciones de confianza con mucho detalle para incluir puntos de conexión de metadatos, definiciones exactas de intercambio de notificaciones, además de configurar secretos, claves y certificados según lo necesite cada proveedor de identidades.

### <a name="inheritance-model"></a>Modelo de herencia

Cuando una aplicación llama al archivo de directiva de usuario de confianza, la instancia del marco de experiencia de identidad en Azure AD B2C agrega a todos los elementos desde el archivo base, el archivo de extensiones y, después, desde el archivo de directiva de usuario de confianza para ensamblar la directiva actual en vigor.  Los elementos del mismo tipo y nombre en el archivo de usuario de confianza reemplazarán los existentes en el de extensiones, y el de extensiones reemplazará al archivo base.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a las directivas personalizadas](custom-policy-get-started.md)
