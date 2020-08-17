---
title: Integración de su oferta de marketplace comercial de Microsoft con Azure Active Directory
description: Use Azure Active Directory para autenticar sus ofertas de Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 07/24/2020
ms.openlocfilehash: 4c700a61de80968b17585faf92e268fef8d86f0e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323271"
---
# <a name="integrate-your-commercial-marketplace-listing-with-azure-active-directory"></a>Integre su listado de marketplace comercial con Azure Active Directory

 En este artículo se explican los requisitos para la integración de un listado u oferta de marketplace comercial con Azure Active Directory (Azure AD). Azure AD es un servicio de identidad en la nube que usa marcos estándar del sector para habilitar la autenticación con un cuenta de Microsoft. [Más información acerca de Azure Active Directory](https://azure.microsoft.com/services/active-directory).

## <a name="azure-ad-benefits"></a>Ventajas de Azure AD

Los clientes de Microsoft AppSource y Azure Marketplace usan las experiencias del producto para buscar los catálogos de listados de escaparate. Estas acciones requieren que los clientes inicien sesión en el producto. La integración de Azure AD proporciona las ventajas siguientes:

- Involucración más rápida y una experiencia de cliente optimizada
- Inicio de sesión único (SSO) para millones de usuarios de empresas
- Experiencia de inicio de sesión coherente en todas las aplicaciones que hayan publicado diferentes partners
- Opción de autenticación escalable y multiplataforma para sus aplicaciones móviles y en la nube

## <a name="offers-that-require-azure-ad"></a>Ofertas que requieren Azure AD

Las distintas [opciones de listado y tipos de ofertas](determine-your-listing-type.md) de marketplace comercial tienen requisitos distintos para implementarse en Azure AD. Consulte la tabla siguiente para más detalles.

| Tipo de oferta    | ¿Es necesario el SSO de Azure AD para ponerse en contacto conmigo?  | ¿Se requiere el SSO de Azure AD para la evaluación gratuita? | ¿Se requiere el SSO de Azure AD para la versión de prueba?  | El SSO de Azure AD se requiere para Transact |
| :------------------- | :-------------------|:-------------------|:-------------------|:-------------------|
| Máquina virtual | N/D | No | No | No |
| Azure Apps (plantilla de solución)  | N/D | N/D | N/D | N/D |
| Aplicaciones administradas  | N/D | N/D | N/D | No |
| SaaS  | No | Sí | Sí | Sí |
| Contenedores  | N/D | N/D | N/D | No |
| Servicios de consultoría  | No | N/D | N/D | N/D |

Para más información sobre los requisitos técnicos de SaaS, consulte [Azure AD y ofertas de SaaS comercializable en el marketplace comercial](./azure-ad-saas.md).

## <a name="azure-ad-integration"></a>Integración de Azure AD

- Para obtener más información sobre cómo integrar Azure AD para las ofertas de software como servicio (SaaS) comercializables, consulte [Azure AD y ofertas de SaaS comercializable en el marketplace comercial](./azure-ad-saas.md).
- Para obtener información sobre cómo habilitar el inicio de sesión único mediante la integración de Azure AD en su descripción, consulte [Azure Active Directory para desarrolladores](../active-directory/develop/index.yml).
- Para conocer más detalles sobre el inicio de sesión único de Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../active-directory/manage-apps/what-is-single-sign-on.md).

## <a name="enable-a-trial-listing"></a>Habilitar una descripción de prueba

Automatizar la instalación de cliente puede aumentar la probabilidad de conversión. Cuando el cliente selecciona la descripción de prueba y se redirecciona al entorno de prueba, usted puede configurar el cliente directamente, sin necesidad de pasos de inicio de sesión adicionales.

Durante la autenticación, Azure AD envía un token a la aplicación u oferta. La información de usuario proporcionada por el token permite crear una cuenta de usuario en la aplicación u oferta. Para obtener más información, consulte los [tokens de ejemplo](../active-directory/develop/id-tokens.md).

Cuando usa Azure AD para habilitar la autenticación con un solo clic en la aplicación o en la descripción de prueba, obtiene las siguientes ventajas:

- Optimice la experiencia del cliente de Marketplace a la descripción de prueba.
- Mantenga la sensación de una "experiencia dentro del producto" incluso cuando se redirige al usuario desde Marketplace al dominio o entorno de evaluación.
- Reduzca la probabilidad de abandono cuando redireccione a los usuarios al no haber pasos adicionales de inicio de sesión.
- Reduzca las barreras de implementación para la gran población de usuarios de Azure AD.

## <a name="verify-azure-ad-integration"></a>Comprobación de la integración de Azure AD

### <a name="multitenant-solutions"></a>Soluciones multiinquilino

Use Azure AD para admitir las acciones siguientes:

- Registre la aplicación en uno de los escaparates de Marketplace. Consulte el vínculo sobre el [registro de una aplicación](../active-directory/develop/quickstart-register-app.md) o la [obtención de un certificado de AppSource](../active-directory/azuread-dev/howto-get-appsource-certified.md) para obtener más información.
- Habilite la característica de compatibilidad multiinquilino en Azure AD para obtener una experiencia de evaluación con un clic.

Si no está familiarizado con el uso del inicio de sesión único federado de Azure AD, siga estos pasos:

1. Registre la aplicación en Marketplace.
1. Desarrolle el SSO con Azure AD mediante el uso de [OAuth 2.0](../active-directory/azuread-dev/v1-protocols-oauth-code.md) u [OpenID Connect](../active-directory/azuread-dev/v1-protocols-openid-connect-code.md).
1. Habilite la característica de compatibilidad multiinquilino en Azure AD para proporcionar una experiencia de evaluación con un clic.

### <a name="single-tenant-solutions"></a>Soluciones de un solo inquilino

Use Azure AD para admitir alguna de las acciones siguientes:

- Agregue usuarios al directorio como invitados con [B2B de Azure AD](../active-directory/b2b/what-is-b2b.md).
- Configure manualmente las versiones de evaluación de los clientes mediante la opción de publicación **Ponerse en contacto conmigo**.
- Desarrolle una versión de prueba por cliente.
- Cree una aplicación de demostración de ejemplo para varios inquilinos que use el inicio de sesión único.

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, 

- [Obtenga información](https://azuremarketplace.microsoft.com/sell) sobre Marketplace.

Para registrarse en el Centro de partner y empezar a crear una nueva oferta o trabajar en una existente, haga lo siguiente:

- [Inicie sesión en el Centro de Partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear o completar la oferta.
