---
title: Azure Active Directory y ofertas de SaaS comercializable en el marketplace comercial
description: Obtenga información sobre cómo funciona Azure Active Directory y ofertas de SaaS comercializable en el marketplace comercial de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 07/10/2020
ms.openlocfilehash: 0d8e2ee684bc08ec23e052229d50b7e9d62c0ecb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87328093"
---
# <a name="azure-ad-and-transactable-saas-offers-in-the-commercial-marketplace"></a>Azure AD y ofertas de SaaS comercializable en el marketplace comercial

El servicio de administración de identidades y acceso basado en la nube de Microsoft, [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis) (Azure AD), ayuda a los usuarios a iniciar sesión y acceder a recursos internos y externos. En el marketplace commercial de Microsoft Commercial, Azure AD hace que las ofertas de SaaS comercializables sean más fáciles y seguras para todos, incluidos los editores, los compradores y los usuarios. Con Azure AD, los editores pueden automatizar el aprovisionamiento de los usuarios en sus aplicaciones de software como servicio (SaaS), y los compradores pueden administrar estos usuarios aprovisionados ellos mismos. 

Además, el [inicio de sesión único de Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on) (SSO) proporciona seguridad y comodidad cuando los usuarios inician sesión en las aplicaciones de Azure AD. Una interacción más rápida y las experiencias optimizadas también inspiran confianza a los compradores y usuarios desde la primera interacción con la aplicación de SaaS de un editor. Esto proporciona una impresión positiva que crea visibilidad y fomenta la repetición de la empresa.

Seguir las instrucciones de este artículo le ayudará a certificar su oferta de SaaS en el marketplace comercial. Para obtener más información acerca de la certificación, lea las [directivas de certificación detalladas del marketplace comercial](https://aka.ms/commercial-marketplace-certification-policies#100-general), incluidas las [específicas de SaaS](https://aka.ms/commercial-marketplace-certification-policies#1000-software-as-a-service-saas).

## <a name="before-you-begin"></a>Antes de empezar

Al [crear la oferta de SaaS](./partner-center-portal/create-new-saas-offer.md) en el Centro de partners, elige entre un conjunto de opciones específicas de llamada a la acción que se mostrarán en la lista de ofertas. Su elección determina cómo se realiza la transacción de la oferta en el marketplace comercial. Las ofertas vendidas a través de Microsoft se denominan ofertas comercializables. Facturamos al cliente en su nombre por todas las ofertas comercializables. Si opta por vender a través de Microsoft y que realicemos las transacciones en su nombre (la opción **Sí**), ha elegido crear una oferta comercializable y este artículo es para usted. Le recomendamos que lo lea en su totalidad.

Si decide solo incluir la oferta en el marketplace comercial y procesar las transacciones de forma independiente (la opción **No**), tiene tres opciones para la forma en que los clientes potenciales accederán a su oferta: Obténgalo ahora (de forma gratuita), Evaluación gratuita y Ponerse en contacto conmigo. Si selecciona **Obténgalo ahora (de forma gratuita)** o **Evaluación gratuita**, este artículo no es para usted. En su lugar, consulte [Creación de la página de aterrizaje de su oferta de SaaS gratuita o de prueba en el marketplace comercial](./azure-ad-free-or-trial-landing-page.md) para obtener más información. Si selecciona **Ponerse en contacto conmigo**, no hay responsabilidades directas del editor. Siga creando su oferta en el Centro de partners.

## <a name="how-azure-ad-works-with-the-commercial-marketplace-for-saas-offers"></a>Cómo funciona Azure AD con el marketplace comercial para las ofertas de SaaS

Azure AD permite la adquisición, el cumplimiento y la administración sin problemas de las soluciones del marketplace comercial. En la figura 1 se muestra cómo el editor, el comprador y el usuario interactúan para comprar y activar una suscripción. También se muestra cómo los clientes usan y administran las aplicaciones de SaaS que obtienen del marketplace comercial. Para los fines de esta ilustración, el comprador es el usuario de la aplicación de SaaS que inicia una compra desde el marketplace comercial.

Como se muestra en la figura 1, cuando un comprador selecciona su oferta, inicia una cadena de flujos de trabajo que incluyen la compra, la suscripción y la administración de usuarios. Dentro de esta cadena, usted como editor es responsable de ciertos requisitos y Microsoft proporciona soporte técnico en los puntos clave.

***Figura 1: Uso de Azure AD para ofertas de SaaS en el marketplace comercial***

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow.png" alt-text="Muestra los pasos del proceso de administración de compras, administración de suscripciones y administración de usuarios opcionales.":::

En las secciones siguientes se proporcionan detalles sobre los requisitos de cada paso del proceso.

## <a name="process-steps-for-purchase-management"></a>Pasos del proceso de administración de compras

En esta ilustración se muestran los cuatro pasos del proceso para la administración de compras.

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow-1-4.png" alt-text="Muestra los cuatro pasos del proceso para la administración de compras.":::

En esta tabla se proporcionan los detalles de los pasos del proceso de administración de compras.

| Paso del proceso | Acción del editor | Recomendado o requerido para los editores |
| ------------ | ------------- | ------------- |
| 1. El comprador inicia sesión en el marketplace comercial con su identidad de Id. de Azure y selecciona una oferta de SaaS. | No se requiere ninguna acción por parte del editor. | No aplicable |
| 2. Después de la compra, el comprador selecciona **Configurar cuenta** en Azure Marketplace o **Configurar ahora** en AppSource, que dirige al comprador a la página de aterrizaje del anunciante para esta oferta. El comprador debe ser poder iniciar sesión en la aplicación de SaaS del editor con el SSO de Azure AD y solo se le debe solicitar el consentimiento mínimo que no requiera la aprobación del administrador Azure AD. | Diseñe una [página de aterrizaje](azure-ad-transactable-saas-landing-page.md) para la oferta, de modo que reciba un usuario con su identidad de Azure AD o cuenta de Microsoft (MSA) y facilite el aprovisionamiento o la configuración adicional necesarios. | Obligatorio |
| 3. El editor solicita los detalles de la compra a la API de cumplimiento de SaaS. | Mediante un [token de acceso](./partner-center-portal/pc-saas-registration.md) generado a partir del identificador de aplicación de la página de aterrizaje, [llama al punto de conexión de resolución](./partner-center-portal/pc-saas-fulfillment-api-v2.md#resolve-a-purchased-subscription) para recuperar los detalles de la compra. | Obligatorio |
| 4. A través de Azure AD y de la API de Microsoft Graph, el editor recopila los detalles de la empresa y del usuario necesarios para aprovisionar al comprador en la aplicación de SaaS del editor.  | Descomponga el token de usuario de Azure AD para buscar el nombre y el correo electrónico, o [llame a la API de Microsoft Graph](https://docs.microsoft.com/graph/use-the-api) y use permisos delegados para [recuperar información](https://docs.microsoft.com/graph/api/user-get) sobre el usuario que ha iniciado sesión. | Obligatorio |
||||

## <a name="process-steps-for-subscription-management"></a>Pasos del proceso de la administración de suscripciones

En esta ilustración se muestran los dos pasos del proceso para la administración de suscripciones.

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow-5-6.png" alt-text="Muestra los dos pasos del proceso para la administración de suscripciones.":::

En esta tabla se describen los detalles de los pasos del proceso de administración de suscripciones.

| Paso del proceso | Acción del editor | Recomendado o requerido para los editores |
| ------------ | ------------- | ------------- |
| 5. El editor administra la suscripción a la aplicación de SaaS a través de la API de cumplimiento de SaaS. | Gestione los cambios de suscripción y otras tareas de administración a través de las [API de cumplimiento de SaaS](./partner-center-portal/pc-saas-fulfillment-api-v2.md).<br><br>Este paso requiere un token de acceso, como se describe en el paso 3 del proceso. | Obligatorio |
| 6. Cuando se usan precios medidos, el editor emite eventos de uso a la API del servicio de medición. | Si su aplicación de SaaS incluye la facturación basada en el uso, envíe notificaciones de uso a través de las [API del servicio de medición de Marketplace](./partner-center-portal/marketplace-metering-service-apis.md).<br><br>Este paso requiere un token de acceso, como se describe en el paso 3. | Necesario para la medición |
||||

## <a name="process-steps-for-user-management"></a>Pasos del proceso de administración de usuarios

En esta ilustración se muestran los tres pasos del proceso para la administración de usuarios.

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow-7-9.png" alt-text="Muestra los tres pasos del proceso opcionales para la administración de usuarios.":::

Los pasos del 7 al 9 son pasos opcionales del proceso de administración de usuarios. Proporcionan ventajas adicionales para los editores que admiten el inicio de sesión único (SSO) de Azure AD. En esta tabla se describen los detalles de los pasos del proceso de administración de usuarios.

| Paso del proceso | Acción del editor | Recomendado o requerido para los editores |
| ------------ | ------------- | ------------- |
| 7. Los administradores de Azure AD de la empresa del comprador pueden administrar opcionalmente el acceso de usuarios y grupos a través de Azure AD. | No se requiere ninguna acción por parte del editor para habilitar esto si el SSO de Azure AD está configurado para los usuarios (paso 9). | No aplicable |
| 8. El servicio de aprovisionamiento de Azure AD comunica los cambios entre Azure AD y la aplicación de SaaS del editor. | [Implemente un punto de conexión SCIM](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups) para recibir actualizaciones de Azure AD cuando se agreguen y quiten usuarios. | Recomendado |
| 9. Una vez que la aplicación se ha autorizado y aprovisionado, los usuarios de la empresa del comprador pueden usar el SSO de Azure AD para iniciar sesión en la aplicación de SaaS del editor. | [Use el SSO de Azure AD](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on) para permitir que los usuarios inicien sesión una vez con una cuenta en la aplicación de SaaS del editor. | Recomendado |
||||

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de SaaS en el marketplace comercial](./partner-center-portal/create-new-saas-offer.md)
- [Creación de la página de aterrizaje de su oferta de SaaS comercializable en el marketplace comercial](./azure-ad-transactable-saas-landing-page.md)
