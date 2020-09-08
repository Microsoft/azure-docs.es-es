---
title: Solución de problemas de visualización de una cuenta de facturación en Azure Portal
description: Este artículo le ayuda a solucionar problemas al intentar ver una cuenta de facturación en Azure Portal.
author: amberbhargava
ms.reviewer: banders
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: f701e41c62336bcd7638360a27a0fb4c3ce3ec7d
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686673"
---
# <a name="troubleshoot-viewing-your-billing-account-in-the-azure-portal"></a>Solución de problemas de visualización de una cuenta de facturación en Azure Portal

Cuando se registra para usar Azure, se crea una cuenta de facturación. Use su cuenta de facturación para administrar las facturas, los pagos y hacer seguimiento de los costos. Puede tener acceso a varias cuentas de facturación. Por ejemplo, es posible que haya registrado Azure para su uso personal. También se puede tener acceso a Azure mediante el Contrato Enterprise de la organización o del contrato de cliente de Microsoft. Para cada uno de estos escenarios, se dispondría de una cuenta de facturación independiente. Este artículo le ayuda a solucionar problemas al intentar ver una cuenta de facturación en Azure Portal.

Puede ver sus cuentas de facturación en la página [Azure Cost Management + Facturación](https://portal.azure.com/#blade/Microsoft_Azure_GTM/ModernBillingMenuBlade).

Para más información sobre las cuentas de facturación y la identificación del tipo de cuenta de facturación, consulte [Visualización de las cuentas de facturación en Azure Portal](view-all-accounts.md).

Si no puede ver su cuenta de facturación en Azure Portal, pruebe las siguientes opciones:

## <a name="sign-in-to-a-different-tenant"></a>Inicio de sesión con otro inquilino

Su cuenta de facturación está asociada con un solo inquilino de Azure Active Directory. No verá su cuenta de facturación en la página Azure Cost Management + Facturación si ha iniciado sesión en un inquilino incorrecto. Use los siguientes pasos para cambiar a otro inquilino en Azure Portal y ver sus cuentas de facturación en ese inquilino.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione su perfil (dirección de correo electrónico) en la parte superior derecha de la página.
1. Seleccione **Cambiar directorio**.  
    ![Captura de pantalla que muestra la selección de Cambiar directorio en Azure Portal](./media/troubleshoot-account-not-found/select-switch-directory.png)
1. Seleccione un directorio en la sección **Todos los directorios**.  
    ![Captura de pantalla que muestra la selección de un directorio en Azure Portal](./media/troubleshoot-account-not-found/select-directory.png)

## <a name="sign-in-with-a-different-email-address"></a>Inicio de sesión con otra dirección de correo electrónico

Algunos usuarios tienen varias direcciones de correo electrónico para iniciar sesión en [Azure Portal](https://portal.azure.com). No todas las direcciones de correo electrónico tienen acceso a una cuenta de facturación. Si inicia sesión con una dirección de correo electrónico que tenga permisos para administrar recursos, pero no tiene permisos para ver una cuenta de facturación, no verá la cuenta de facturación en la página [Azure Cost Management + Facturación](https://portal.azure.com/#blade/Microsoft_Azure_GTM/ModernBillingMenuBlade) de Azure Portal.

Inicie sesión en Azure Portal con una dirección de correo electrónico que tenga permiso en la cuenta de facturación para acceder a su cuenta de facturación.

## <a name="sign-in-with-a-different-identity"></a>Inicio de sesión con otra identidad

Algunos usuarios tienen dos identidades con la misma dirección de correo electrónico: una cuenta profesional o educativa y una cuenta personal. Normalmente, solo una de sus identidades tiene permisos para ver una cuenta de facturación. Es posible que tenga dos identidades con una sola dirección de correo electrónico. Si inicia sesión con una identidad que no tiene permiso para ver una cuenta de facturación, no verá la cuenta de facturación en la página [Azure Cost Management + Facturación](https://portal.azure.com/#blade/Microsoft_Azure_GTM/ModernBillingMenuBlade). Siga estos pasos para cambiar la identidad:

1. Inicie sesión en [Azure Portal ](https://portal.azure.com) en una ventana InPrivate/incógnito.
1. Si su dirección de correo electrónico tiene dos identidades, verá una opción para seleccionar una cuenta personal o una cuenta profesional o educativa. Seleccione una de las cuentas.
1. Si no puede ver la cuenta de facturación en la página Azure Cost Management + Facturación de Azure Portal, repita los pasos 1 y 2, y seleccione la otra identidad.

## <a name="contact-us-for-help"></a>Póngase en contacto con nosotros para obtener ayuda

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Pasos siguientes

Lea los siguientes artículos sobre facturación y suscripción para solucionar problemas.

- [Tarjeta rechazada](https://docs.microsoft.com/azure/cost-management-billing/manage/troubleshoot-declined-card)
- [Problemas de inicio de sesión de suscripción](https://docs.microsoft.com/azure/cost-management-billing/manage/troubleshoot-sign-in-issue)
- [No se han encontrado suscripciones](https://docs.microsoft.com/azure/cost-management-billing/manage/no-subscriptions-found)
- [Se ha deshabilitado la vista de costos empresariales](https://docs.microsoft.com/azure/cost-management-billing/manage/enterprise-mgmt-grp-troubleshoot-cost-view)
