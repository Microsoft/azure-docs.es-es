---
title: Migración de cuentas de Cloud Partner Portal a Marketplace comercial de Microsoft
description: Aprenda a migrar su cuenta de Cloud Partner Portal al Centro de partners en Marketplace comercial de Microsoft para Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: parthpandyaMSFT
ms.author: parthp
ms.date: 09/23/2019
ms.openlocfilehash: fbf76f846395d6bf0b02828ab1d6ad8b70c97784
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289340"
---
# <a name="how-to-migrate-your-account-from-cloud-partner-portal-to-partner-center"></a>Cómo migrar su cuenta del Cloud Partner Portal al Centro de partners

Si tiene una cuenta de Cloud Partner Portal (CPP) existente, la configuración de la cuenta debe migrarse al Centro de partners.

## <a name="account-migration-process"></a>Proceso de migración de cuentas

Si es un usuario con el rol de propietario en CPP para una cuenta determinada, se muestra un banner en amarillo en la página Perfil de publicación. Puede que su caso se encuadre en uno de estos:

- Su cuenta ya se ha migrado y está listo para administrar su configuración de cuenta en el Centro de partners.
- Su cuenta no se ha migrado al Centro de partners y es necesario que tome medidas adicionales.

### <a name="your-account-has-been-migrated-to-partner-center"></a>Su cuenta se ha migrado al Centro de partners.

Administre ahora su cuenta en el Centro de partners. Los cambios, como la adición o la eliminación de usuarios, se volverán a sincronizar con CPP.

### <a name="you-have-not-yet-migrated-your-account-to-partner-center"></a>Todavía no ha migrado su cuenta al Centro de partners.

Haga clic en el banner para iniciar el proceso de migración de la cuenta. Deberá escribir los siguientes elementos:

1. Dirección de correo electrónico del trabajo: <br> <br> La mayoría de las veces se inicia sesión con la dirección de correo electrónico que se usa para iniciar sesión en CPP. En algunos casos, se debe usar una dirección de correo electrónico diferente:

    * Cuenta de Microsoft: si la cuenta de CPP es una cuenta Microsoft, especifique una dirección de correo electrónico del trabajo válida asociada al inquilino para el que está registrado el identificador de Microsoft Partner Network (MPN). Para más información, consulte [Registro en el programa Microsoft Partner Network](#sign-up-for-microsoft-partner-network-program).

    * El inquilino no coincide: si la dirección de correo electrónico profesional no pertenece al inquilino asociado al identificador de Microsoft Partner Network de la cuenta de CPP, se produce un error. Para saltarse este error, escriba una dirección de correo electrónico asociada con el inquilino. En un mensaje de error se proporcionará el nombre del inquilino.

2. Registro en el programa Microsoft Partner Network

    En caso de que la cuenta de CPP no tenga un identificador de Microsoft Partner Network o si lo tiene, pero no es válido, debe registrarse en el programa Microsoft Partner Network como parte del proceso de activación.

## <a name="publishers-moving-from-cpp"></a>Publicadores que se trasladan desde CPP

Si su cuenta se migró desde CPP, no es necesario que cree una nueva cuenta del Centro de partners. Debe haber recibido un vínculo personalizado a su nueva cuenta del Centro de partners en el correo electrónico y en una notificación de banner tras iniciar sesión en su cuenta de CPP existente.

Una vez haya habilitado la nueva cuenta de Centro de partners con la visita a este vínculo personalizado, puede volver a su cuenta mediante el [panel de información de Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) en Centro de partners.

El contrato de publicación y la información de perfil de empresa se migrarán a la nueva cuenta de Centro de partners, junto con cualquier información de perfil de pago de cuenta, permisos y cuentas de usuario configurados previamente, así como ofertas activas asociadas con su cuenta de CPP.

Una vez que la información de la cuenta se haya migrado de CPP al Centro de partners, ya no usará CPP para hacer actualizaciones de cuentas o administrar usuarios, permisos y facturación. Durante un tiempo limitado, las actualizaciones de cuenta que haga en el Centro de partners se actualizarán automáticamente con su cuenta CPP de solo lectura hasta que el portal CPP entre finalmente en desuso.

## <a name="sign-up-for-microsoft-partner-network-program"></a>Registro en el programa Microsoft Partner Network

Las empresas que deseen asociarse a Microsoft deben unirse a Microsoft Partner Network (MPN) y obtener un identificador de MPN. Si ya es miembro de Microsoft Partner Network y tiene un identificador de MPN, tenga a mano la información, ya que va a necesitarla durante el proceso de activación de la cuenta.  

Si no es miembro de Microsoft Partner Network, puede [unirse aquí](https://signup.microsoft.com/signup?sku=StoreForBusinessIW&origin=partnerdashboard&culture=en-us&ru=https://partner.microsoft.com/dashboard/account/v3/xpu/onboard?ru=/dashboard/account/v3/enrollment/companyprofile/basicpartnernetwork/new) para obtener un identificador de MPN. Anote el identificador de MPN, ya que debe escribirlo durante el proceso de activación de la cuenta.

Para obtener más información sobre Microsoft Partner Network, consulte [Únase a Microsoft Partner Network](https://partner.microsoft.com/membership) en el sitio web del asociado. Para obtener más información sobre las ventajas de ISV en Microsoft Partner Network, consulte el [centro de recursos de ISV](https://partner.microsoft.com/isv-resource-hub).  

## <a name="move-dynamics-365-and-powerapps-offers-to-partner-center"></a>Traslado de ofertas de Dynamics 365 y PowerApps al Centro de partners

Para optimizar la administración de cuentas y ofertas para Dynamics 365 Customer Engagement, PowerApps y Dynamics 365 Operations, las ofertas se han trasladado al [Centro de partners](https://partner.microsoft.com/). El traslado garantiza que el mismo contenido esté disponible tanto para los catálogos públicos como para los catálogos de vendedores.

Para obtener información sobre lo que debe hacerse antes del **15 de octubre de 2019** para sus ofertas de Dynamics 365 Customer Engagement, PowerApps y Dynamics 365 Operations, siga las instrucciones a continuación.

> [!NOTE]
> Esto no se aplica a las ofertas de Dynamics 365 Business Central.  

1. Si la cuenta de pertenencia de MPN se creó originalmente en Partner Membership Center (PMC), inicie sesión en el [Centro de partners](https://partner.microsoft.com/pcv/accountsettings/connectedpartnerprofile) para confirmar que la cuenta se ha migrado. Si ve una pantalla de perfil con el identificador de MPN, significa que ya está listo para continuar. Si no es así, debe iniciar la migración de la cuenta siguiendo las indicaciones de [Partner Membership Center](https://partners.microsoft.com/partnerprogram/Welcome.aspx). Consulte [Soporte técnico](https://partner.microsoft.com/support?issueid=100-0077) si necesita ayuda.
2. Vaya a [Introducción al portal de Marketplace comercial en el Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview). Si ve "Marketplace comercial" en el panel de navegación izquierdo, significa que está inscrito y debe continuar con el paso siguiente. Si no es así, [inscríbase ahora en el Marketplace comercial](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership).
3. Confirme que sus ofertas están en AppSource. Para ello, [busque sus ofertas](https://appsource.microsoft.com/). Si las ofertas ya están en AppSource, continúe con el paso siguiente. En el caso de las ofertas que no estén en AppSource, cree una [nueva oferta de Dynamics 365 Customer Engagement](create-new-customer-engagement-offer.md) o una [nueva oferta de Dynamics 365 Operations](create-new-operations-offer.md).
4. En la [página de acuerdos](https://partner.microsoft.com/dashboard/account/agreements) del Centro de partners, asegúrese de haber revisado y aceptado el **anexo de ISV de Business Applications**.
5. En la [configuración de cuenta](https://partner.microsoft.com/dashboard/account/v3/accountsettings/billingprofile) del Centro de partners, asegúrese de que su información de facturación está completa.
6. Envíe cada oferta nueva y existente de certificación y publicación, incluso si las ofertas se han certificado previamente.
    * Complete las pantallas de información, incluido el suministro de su aplicación para la certificación, así como la información de marketing. Seleccione **Enviar** (esquina superior derecha de la pantalla) antes del **15 de octubre de 2019**. Estos pasos deben completarse para que la disponibilidad de la oferta no se vea afectada.
    * Si es apto, puede solicitar participar en el nivel premium durante este proceso.
    * La certificación o la recertificación requieren que la aplicación sea compatible con la versión más reciente de nuestra plataforma Business Applications.
    * Una vez aprobada la aplicación, recibe un correo electrónico para volver a la oferta y seleccionar "publicar" a fin de permitir que la oferta se publique en Microsoft AppSource.

## <a name="additional-resources"></a>Recursos adicionales

Obtenga ayuda de expertos y compañeros en los foros y descubra blogs, seminarios web, vídeos, eventos, etc. en [Microsoft Dynamics CRM](https://community.dynamics.com/crm?wa=wsignin1.0).

Si necesita ayuda para publicar, certificar o administrar sus ofertas de Marketplace, [envíe una incidencia de soporte técnico](https://aka.ms/MarketplacePublisherSupport).

## <a name="next-step"></a>Paso siguiente

- [Administración de la cuenta de Marketplace comercial en el Centro de partners](./manage-account.md)
