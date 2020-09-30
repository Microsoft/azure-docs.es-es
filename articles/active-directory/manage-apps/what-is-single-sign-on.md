---
title: ¿Qué es el inicio de sesión único de Azure?
description: Obtenga información sobre cómo funciona el inicio de sesión único (SSO) con Azure Active Directory. Use el inicio de sesión único para que los usuarios no tengan que recordar contraseñas para todas las aplicaciones. Use también el inicio de sesión único para simplificar la administración de la administración de cuentas.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: overview
ms.date: 12/03/2019
ms.author: kenwith
ms.reviewer: arvindh, japere
ms.custom: contperfq1
ms.openlocfilehash: 1eaef57f46bf6373fdd2a73575bb028904ef108b
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90561657"
---
# <a name="what-is-single-sign-on-sso"></a>¿Qué es el inicio de sesión único (SSO)?

El inicio de sesión único permite que un usuario no tenga que iniciar sesión en cada aplicación que use. El usuario inicia sesión una vez y esa credencial también se utiliza para otras aplicaciones.

Si es un usuario final, es probable que no le interesen mucho los detalles del inicio de sesión único. Solo deseará usar las aplicaciones que le permitan ser productivo sin tener que escribir su contraseña a menudo. Puede encontrar sus aplicaciones en https://myapps.microsoft.com.
 
Si es administrador o profesional de TI, siga leyendo para conocer mejor el inicio de sesión único y cómo se implementa en Azure.

## <a name="single-sign-on-basics"></a>Aspectos básicos del inicio de sesión único
El inicio de sesión único proporciona un gran avance en el modo en que los usuarios inician sesión y usan las aplicaciones. Los sistemas de autenticación basados en el inicio de sesión único se suelen denominar de "autenticación moderna". La autenticación moderna y el inicio de sesión único pertenecen a una categoría de computación denominada Administración de identidades y acceso (IAM). Para comprender lo que hace posible el inicio de sesión único, consulte este vídeo.

Aspectos básicos de la autenticación: Aspectos básicos | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/fbSVgC8nGz4]

## <a name="single-sign-on-with-web-applications"></a>Inicio de sesión único con aplicaciones web
Las aplicaciones web son conocidas por todos. Varias empresas las hospedan y permiten que estén disponibles como servicio. Algunos ejemplos conocidos de aplicaciones web son Microsoft 365, GitHub y Salesforce, pero hay muchas otras. Las personas acceden a las aplicaciones web mediante un explorador web en su equipo. El inicio de sesión único permite a los usuarios desplazarse entre las distintas aplicaciones web sin tener que iniciar sesión varias veces.

Para información sobre cómo funciona el inicio de sesión único con las aplicaciones web, consulte estos dos vídeos.

Aspectos básicos de la autenticación: Aplicaciones web | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/tCNcG1lcCHY]

Aspectos básicos de la autenticación: Inicio de sesión único web | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/51B-jSOBF8U]

## <a name="cloud-versus-on-premises-hosted-apps"></a>Aplicaciones hospedadas de forma local y en la nube
La forma de implementar el inicio de sesión único depende de dónde se hospede la aplicación. El hospedaje es importante debido a la manera en que el tráfico de red se enruta para acceder a la aplicación. Si una aplicación se hospeda en la red local y se accede a ella a través de la red local, lo que se conoce como "aplicación en el entorno local", no es necesario que los usuarios accedan a Internet para usarla. Si la aplicación se hospeda en otra parte y, por tanto, se conoce como "hospedada en la nube", los usuarios tendrán que acceder a Internet para poder usarla.

> [!TIP]
> Las aplicaciones hospedadas en la nube también se denominan aplicaciones de software como servicio (SaaS). 

El inicio de sesión único en las aplicaciones hospedadas en la nube es sencillo. El usuario permite que el proveedor de identidades sepa que la aplicación se está usando. Luego, la aplicación se configura para que confíe en el proveedor de identidades. Para aprender a usar Azure AD como proveedor de identidades de una aplicación, consulte la [serie de inicios rápidos sobre la administración de aplicaciones](add-application-portal.md).

> [!TIP]
> Los términos "nube" e "Internet" suelen usarse indistintamente. El motivo tiene que ver con los diagramas de red. Es habitual denotar redes de equipos de gran tamaño con una forma de nube en un diagrama porque no es factible dibujar todos los componentes. Internet es la red más conocida y, por lo tanto, es fácil usar los términos indistintamente. Sin embargo, cualquier red de equipos puede usar la nube.

También puede usar el inicio de sesión único en aplicaciones locales. La tecnología para realizar el inicio de sesión único local se denomina proxy de aplicación. Para más información al respecto, consulte [Opciones de inicio de sesión único](sso-options.md).

## <a name="multiple-identity-providers"></a>Varios proveedores de identidades
A la acción de configurar el inicio de sesión único para que funcione entre varios proveedores de identidades se le conoce como federación. Para saber cómo funciona la federación, consulte este vídeo.

Aspectos básicos de la autenticación: Federación | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/CjarTgjKcX8]


## <a name="next-steps"></a>Pasos siguientes
* [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
* [Opciones de inicio de sesión único](sso-options.md)
