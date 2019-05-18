---
title: Descripción de las experiencias de consentimiento de la aplicación de Azure AD | Microsoft Docs
description: Más información sobre las experiencias de consentimiento de Azure AD para ver cómo se pueden usar al administrar y desarrollar aplicaciones en Azure AD
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/27/2019
ms.author: ryanwi
ms.reviewer: zachowd
ms.collection: M365-identity-device-management
ms.openlocfilehash: d71bfd5e560bb1509337ac371fbe101b4c6d63b5
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540641"
---
# <a name="understanding-azure-ad-application-consent-experiences"></a>Descripción de las experiencias de consentimiento de la aplicación de Azure AD

Aprenda más acerca de la experiencia del usuario para el consentimiento de la aplicación de Azure Active Directory (Azure AD). Por lo que puede administrar las aplicaciones para su organización o desarrollar aplicaciones de manera inteligente con una experiencia de consentimiento más sencilla.

## <a name="consent-and-permissions"></a>Consentimiento y permisos

El consentimiento es el proceso por el cual un usuario concede autorización para que una aplicación acceda a recursos protegidos en su nombre. Se puede solicitar el consentimiento a un administrador o usuario para que permita el acceso a los datos de la organización o de personas.

La experiencia real del usuario para otorgar el consentimiento variará en función de las directivas establecidas en el inquilino del usuario, el ámbito de autoridad del usuario (o su rol) y el tipo de [permisos](https://docs.microsoft.com/azure/active-directory/develop/active-directory-permissions) que solicita la aplicación cliente. Esto significa que los desarrolladores de aplicaciones y los administradores de inquilinos tienen cierto control sobre la experiencia de consentimiento. Los administradores tienen la flexibilidad de configurar y deshabilitar las directivas en una aplicación o inquilino para controlar la experiencia de consentimiento en su inquilino. Los desarrolladores de aplicaciones pueden dictar qué tipos de permiso se solicitan y si quieren guiar a los usuarios a través del flujo de consentimiento del usuario o el flujo de consentimiento del administrador.

- El **flujo de consentimiento del usuario** es cuando un desarrollador de aplicaciones dirige a los usuarios al punto de conexión de autorización con la intención de registrar el consentimiento solo para el usuario actual.
- El **flujo de consentimiento del administrador** es cuando un desarrollador de aplicaciones dirige a los usuarios al punto de conexión de consentimiento del administrador con la intención de registrar el consentimiento para todo el inquilino. Para asegurarse de que el flujo de consentimiento del administrador funcione correctamente, los desarrolladores de aplicaciones deben enumerar todos los permisos en la propiedad `RequiredResourceAccess` en el manifiesto de aplicación. Para más información, consulte [Manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest).

## <a name="building-blocks-of-the-consent-prompt"></a>Bloques de creación de la petición de consentimiento

La petición de consentimiento está diseñada para garantizar que los usuarios tengan suficiente información para determinar si confían en que la aplicación cliente acceda a recursos protegidos en su nombre. Comprender los bloques de creación ayudará a los usuarios que otorgan consentimientos a tomar decisiones más informadas y ayudará a los desarrolladores a crear mejores experiencias de usuario.

En el diagrama y la tabla siguientes se proporciona información sobre los bloques de creación de la petición de consentimiento.

![Bloques de creación de la petición de consentimiento](./media/application-consent-experience/consent_prompt.png)

| # | Componente | Propósito |
| ----- | ----- | ----- |
| 1 | Identificador de usuario | Este identificador representa al usuario en cuyo nombre la aplicación cliente solicita acceso a recursos protegidos. |
| 2 | Puesto | El título cambia en función de si los usuarios siguen el flujo de consentimiento del usuario o del administrador. En el flujo de consentimiento del usuario, el título será "Permisos solicitados", mientras que en el flujo de consentimiento del administrador, el título tendrá una línea adicional "Aceptar para su organización". |
| 3 | Logotipo de la aplicación | Esta imagen debe ayudar a los usuarios a tener una indicación visual de si esta aplicación es la aplicación a la que pretenden acceder. Los desarrolladores de aplicaciones proporcionan esta imagen, y no se valida la propiedad de esta imagen. |
| 4 | Nombre de la aplicación | Este valor debe informar a los usuarios qué aplicación solicita acceso a sus datos. Tenga en cuenta que los desarrolladores proporcionan este nombre, y no se valida la propiedad del nombre de esta aplicación. |
| 5 | Dominio de editor | Este valor debe proporcionar a los usuarios un dominio cuya confiabilidad puedan evaluar. Los desarrolladores proporcionan este dominio, y no se valida la propiedad de este dominio de editor. |
| 6 | Permisos | Esta lista contiene los permisos solicitados por la aplicación cliente. Los usuarios deben evaluar siempre los tipos de permisos que se solicitan a fin de comprender a qué datos estará autorizada a acceder en su nombre la aplicación cliente si aceptan. Como desarrollador de aplicaciones es mejor solicitar acceso a los permisos con los privilegios mínimos. |
| 7 | Descripción del permiso | El servicio que expone los permisos proporciona este valor. Para ver las descripciones de los permisos, debe activar o desactivar el botón de contenido adicional junto al permiso. |
| 8 | Términos de la aplicación | Estos términos contienen vínculos a los términos de servicio y la declaración de privacidad de la aplicación. El editor es responsable de detallar sus reglas en las condiciones de servicio. Además, el editor es responsable de divulgar la forma en que usará y compartirá los datos de usuario en su declaración de privacidad. Si el editor no proporciona vínculos a estos valores para las aplicaciones multiinquilino, habrá una advertencia en negrita en la petición de consentimiento. |
| 9 | https://myapps.microsoft.com | Este es el vínculo donde los usuarios pueden revisar y quitar las aplicaciones que no son de Microsoft que actualmente tienen acceso a sus datos. |

## <a name="common-consent-scenarios"></a>Escenarios comunes de consentimiento

Estas son las experiencias de consentimiento que un usuario puede ver en los escenarios de consentimiento comunes:

1. Personas que acceden a una aplicación que los dirige al flujo de consentimiento del usuario, a la vez que se requiere un conjunto de permisos que se encuentra dentro de su ámbito de autoridad.
    
    1. Los administradores verán un control adicional en la petición de consentimiento tradicional que les permitirá dar su consentimiento en nombre de todo el inquilino. El control estará desactivado de manera predeterminada, por lo que solo cuando los administradores marquen la casilla explícitamente se otorgará el consentimiento en nombre de todo el inquilino. En este momento, esta casilla solo se mostrará para el rol de administrador global, por lo que el administrador en la nube y el administrador de aplicaciones no verán esta casilla.

        ![Petición de consentimiento para el escenario 1a](./media/application-consent-experience/consent_prompt_1a.png)
    
    2. Los usuarios verán la petición de consentimiento tradicional.

        ![Petición de consentimiento para el escenario 1b](./media/application-consent-experience/consent_prompt_1b.png)

2. Personas que accedan a una aplicación que requiera al menos un permiso que está fuera de su ámbito de autoridad.
    1. Los administradores verán la misma petición que la 1.i anterior.
    2. Los usuarios no podrá otorgar consentimiento a la aplicación y se les indicará que pidan al administrador acceso a la aplicación. 
                
        ![Petición de consentimiento para el escenario 1b](./media/application-consent-experience/consent_prompt_2b.png)

3. Personas que navegan o se las dirige al flujo de consentimiento del administrador.
    1. Los usuarios administradores verán la petición de consentimiento del administrador. El título y las descripciones de permisos cambiaron en esta petición, los cambios destacan el hecho de que, al aceptar esta petición, se otorgará a la aplicación acceso a los datos solicitados en el nombre de todo el inquilino.
        
        ![Petición de consentimiento para el escenario 1b](./media/application-consent-experience/consent_prompt_3a.png)
        
    1. Los usuarios sin privilegios de administrador verán la misma pantalla que 2.ii anterior.

## <a name="next-steps"></a>Pasos siguientes
- Introducción paso a paso de [cómo el marco de consentimiento de Azure AD implementa el consentimiento](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications).
- Para más detalles, aprenda [cómo una aplicación multiinquilino puede utilizar el marco de consentimiento](active-directory-devhowto-multi-tenant-overview.md) para implementar el consentimiento de "usuario" y "administrador", compatible con patrones de aplicación de niveles múltiples más avanzados.
- Obtenga información sobre [cómo configurar el dominio del publicador de la aplicación](howto-configure-publisher-domain.md).
