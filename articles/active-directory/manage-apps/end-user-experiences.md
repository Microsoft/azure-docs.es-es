---
title: 'Experiencias de usuario final para aplicaciones: Azure Active Directory'
description: Azure Active Directory (Azure AD) proporciona varias maneras personalizables para implementar aplicaciones para los usuarios finales de su organización.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 09/27/2019
ms.author: kenwith
ms.reviewer: arvindh
ms.openlocfilehash: d3b7dbbb9aab29e083795025ad5bf7381a4fa0ee
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231221"
---
# <a name="end-user-experiences-for-applications-in-azure-active-directory"></a>Experiencias de usuario final para aplicaciones en Azure Active Directory

Azure Active Directory (Azure AD) proporciona varias maneras personalizables para implementar aplicaciones para los usuarios finales de su organización:

* Aplicaciones de Azure AD
* Iniciador de aplicaciones de Office 365
* Inicio de sesión directo en aplicaciones federadas
* Vínculos profundos a aplicaciones federadas, con contraseña o existentes

Los métodos que elija implementar en su organización son criterio suyo.

## <a name="azure-ad-my-apps"></a>Aplicaciones de Azure AD

Mis aplicaciones de https://myapps.microsoft.com es un portal basado en web que permite que los usuarios finales que tengan cuenta organizativa en Azure Active Directory puedan ver e iniciar aplicaciones a las que el administrador de Azure AD les haya concedido acceso. Si es usuario final con [Azure Active Directory Premium](https://azure.microsoft.com/pricing/details/active-directory/), también puede usar las capacidades autoservicio de administración de grupos mediante Mis aplicaciones.

De forma predeterminada, todas las aplicaciones se muestran juntas en una sola página. Sin embargo, puede usar colecciones para agrupar aplicaciones relacionadas y presentarlas en una pestaña independiente, lo que facilita su búsqueda. Por ejemplo, puede usar colecciones para crear agrupaciones lógicas de aplicaciones para roles de trabajo específicos, tareas, proyectos, etc. Para obtener información, vea [Creación de colecciones en el portal Mis aplicaciones](access-panel-collections.md). 

Mis aplicaciones es independiente de Azure Portal y no requiere que los usuarios tengan una suscripción de Azure u Office 365.

Para obtener más información sobre Mis aplicaciones de Azure AD, vea [Introducción a Mis aplicaciones](../user-help/active-directory-saas-access-panel-introduction.md).

## <a name="office-365-application-launcher"></a>Iniciador de aplicaciones de Office 365

Para las organizaciones que han implementado Office 365, las aplicaciones asignadas a los usuarios mediante Azure AD también aparecerán en el portal de Office 365, en [https://portal.office.com/myapps](https://portal.office.com/myapps). Esto hace que resulte fácil y cómodo para los usuarios de una organización iniciar sus aplicaciones sin tener que utilizar un segundo portal y es la solución de inicio de aplicaciones recomendada para las organizaciones que usan Office 365.

Para obtener más información sobre el iniciador de aplicaciones de Office 365, consulte [Hacer que su aplicación aparezca en el iniciador de aplicaciones de Office 365](https://msdn.microsoft.com/office/office365/howto/connect-your-app-to-o365-app-launcher).

## <a name="direct-sign-on-to-federated-apps"></a>Inicio de sesión directo en aplicaciones federadas

La mayoría de las aplicaciones federadas que admiten SAML 2.0, WS-Federation u OpenID Connect también tienen la capacidad de que los usuarios inicien la sesión en la aplicación y, después, lo hagan a través Azure AD, ya sea mediante el redireccionamiento automático o haciendo clic en un vínculo para iniciar la sesión. Esto se conoce como inicio de sesión iniciado por el proveedor de servicios, y la mayoría de las aplicaciones federadas de la galería de aplicaciones de Azure AD lo admiten (vea la documentación vinculada desde el Asistente para la configuración de inicio de sesión único de la aplicación en Azure Portal para obtener detalles).

## <a name="direct-sign-on-links"></a>Vínculos directos de inicio de sesión

Azure AD también admite vínculos directos de inicio de sesión único a las aplicaciones individuales que admiten el inicio de sesión con contraseña, el inicio de sesión único vinculado y cualquier forma de inicio de sesión único federado.

Estos vínculos son direcciones URL especialmente diseñadas que envían al usuario al proceso de inicio de sesión de Azure AD de una aplicación específica sin necesidad de que la inicie desde Mis aplicaciones de Azure AD u Office 365. Estas **direcciones URL de acceso de usuario** pueden encontrarse en las propiedades de las aplicaciones empresariales disponibles. En Azure Portal, seleccione **Azure Active Directory** > **Aplicaciones empresariales**. Seleccione la aplicación y, después, **Propiedades**.

![Ejemplo de la dirección URL de acceso de usuario en las propiedades de Twitter](media/end-user-experiences/direct-sign-on-link.png)

Estos vínculos se pueden copiar y pegar en cualquier sitio donde que desee proporcionar un vínculo de inicio de sesión a la aplicación seleccionada. Podría ser en un mensaje de correo electrónico o en cualquier portal personalizado basado en web que haya configurado para el acceso de los usuarios a la aplicación. Este es un ejemplo de una AD de Azure único inicio de sesión URL directa de Twitter:

`https://myapps.microsoft.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced`

De forma similar a las direcciones URL específicas de la organización de Mis aplicaciones, se puede personalizar aún más esta dirección URL mediante la adición de uno de los dominios activos o comprobados del directorio tras el dominio *myapps.microsoft.com*. Esto garantiza que cualquier personalización de marca corporativa se carga inmediatamente en la página de inicio de sesión sin que el usuario tenga que escribir primero su ID de usuario:

`https://myapps.microsoft.com/contosobuild.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced`

Cuando un usuario autorizado hace clic en uno de estos vínculos específicos de una aplicación, primero ve la página de inicio de sesión de la organización (siempre que aún no haya iniciado sesión) y, después de iniciar sesión, se le redirige a la aplicación sin detenerse en Mis aplicaciones. Si el usuario no cumple los requisitos previos para tener acceso a la aplicación, como la extensión de explorador para el inicio de sesión único con contraseña, el vínculo le pedirá al usuario que instale la extensión que le falta. La dirección URL del vínculo también permanece constante si cambia la configuración de inicio de sesión única para la aplicación.

Estos vínculos usan los mismos mecanismos de control de acceso que Mis aplicaciones y Office 365, y solo los usuarios o grupos que se han asignado a la aplicación en Azure Portal pueden autenticarse correctamente. Pero cualquier usuario que no tenga autorización ve un mensaje que explica que no se le ha concedido acceso, y se le proporciona un vínculo para cargar Mis aplicaciones para ver las aplicaciones disponibles a las que tiene acceso.

## <a name="next-steps"></a>Pasos siguientes

* [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
* [¿Qué es el inicio de sesión único?](what-is-single-sign-on.md)
* [Guía de introducción a la integración de Azure Active Directory con las aplicaciones](plan-an-application-integration.md)
