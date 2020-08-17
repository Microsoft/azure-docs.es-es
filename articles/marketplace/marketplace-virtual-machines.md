---
title: Guía de publicación de ofertas de máquinas virtuales en Azure Marketplace
description: En este artículo se describen los requisitos para publicar una máquina virtual y una evaluación gratuita de software para su implementación desde Azure Marketplace.
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 04/15/2020
ms.openlocfilehash: dac67c6dae801d11866f7c4978a050a0cb2fa1dd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065641"
---
# <a name="publishing-guide-for-virtual-machine-offers"></a>Guía de publicación para ofertas de máquinas virtuales

La publicación de imágenes de máquina virtual (VM) es una de las formas principales de publicar una solución en Azure Marketplace. Use esta guía para comprender los requisitos para este tipo de oferta. 

Las ofertas de máquina virtual son ofertas de transacción que se implementan y facturan mediante Azure Marketplace. La llamada a la acción que el usuario ve es *Obtener ahora*.

## <a name="free-trial"></a>Evaluación gratuita 

Para que los usuarios puedan probar la oferta, acceda a licencias de software de tiempo limitado al usar el modelo de facturación traiga su propia licencia (BYOL). 

## <a name="test-drive"></a>Versión de prueba

Puede implementar una o varias máquinas virtuales mediante aplicaciones de infraestructura como servicio (IaaS) o software como servicio (SaaS). Una ventaja de la opción de publicación de *versión de prueba* es la configuración automatizada de una máquina virtual o una solución completa dirigida por una visita guiada hospedada por un asociado. Una versión de prueba permite a los clientes evaluar las VM sin ningún costo adicional. El cliente no tiene que ser un cliente existente de Azure para poder participar en la experiencia de prueba. 

Para empezar, póngase en contacto con nosotros mediante [amp-testdrive](mailto:amp-testdrive@microsoft.com). 

|Requisitos  |Detalles |
|---------|---------|
| Tiene una aplicación de Azure Marketplace   |  Una o varias máquinas virtuales mediante IaaS o SaaS.      |

## <a name="interactive-demo"></a>Demostración interactiva

Con esta oferta, proporciona a los clientes una experiencia guiada de su solución mediante una demostración interactiva. La ventaja de la opción de publicación de demostración interactiva es que puede ofrecer una experiencia de prueba sin tener que proporcionar una configuración complicada de la solución compleja. 

## <a name="virtual-machine-offer"></a>Oferta de máquina virtual

Use el tipo de oferta de *máquina virtual* cuando implemente un dispositivo virtual para la suscripción asociada con el cliente. Las VM se pueden comercializar íntegramente con los modelos de licencia de pago por uso o BYOL (traiga su propia licencia). Microsoft hospeda la transacción comercial y factura al cliente en su nombre. Usted obtiene la ventaja de usar la relación de pago preferida entre el cliente y Microsoft, incluidos los contratos empresariales existentes.

> [!NOTE]
> En este momento, los compromisos monetarios asociados con un Contrato Enterprise se pueden aplicar al uso que Azure hace de su VM, pero no a las tarifas de licencias de software.  
> 
> [!NOTE]
> Puede restringir la detección y la implementación de su VM para un conjunto de clientes específico. Para ello, publique la imagen y los precios como una oferta privada. Las ofertas privadas le ofrecen la posibilidad de crear ofertas exclusivas para sus clientes más próximos y proporcionan software y términos personalizados. Los términos personalizados le permiten resaltar una gran variedad de escenarios, incluidas las ofertas sobre el terreno con precios y términos especializados, así como acceso anticipado a software de versión limitada. Las ofertas privadas permiten ofrecer precios o productos específicos a un conjunto limitado de clientes mediante la creación de un nuevo plan con esos detalles.  
>
> Para más información, consulte [Ofertas privadas en Azure Marketplace](https://azure.microsoft.com/blog/private-offers-on-azure-marketplace).  

| Requisito | Detalles |  
|:--- |:--- | 
| Facturación y medición | Su VM debe admitir las opciones de facturación mensual BYOL o de pago por uso. |  
| Disco duro virtual (VHD) compatible con Azure | Las máquinas virtuales deben estar basadas en Windows o Linux. Para obtener más información sobre cómo crear un VHD, vea: <ul> <li>[Distribuciones de Linux aprobadas en Azure](../virtual-machines/linux/endorsed-distros.md) (para discos duros virtuales de Linux).</li> <li>[Creación de un VHD compatible con Azure](./partner-center-portal/azure-vm-create-offer.md) (para VHD con Windows).</li> </ul> |  

>[!Note]
>La participación en el canal de asociados de Proveedores de soluciones en la nube (CSP) ya está disponible. Para más información sobre cómo comercializar su oferta mediante los canales de asociados de CSP de Microsoft, consulte [Proveedores de soluciones en la nube](./cloud-solution-providers.md).

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, aprenda a [Ampliar su negocio en la nube con Azure Marketplace](https://azuremarketplace.microsoft.com/sell).

Para registrarse y empezar a trabajar en el Centro de partners, haga lo siguiente:

- [Inicie sesión en el Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear o completar la oferta.
- Para más información, consulte [Creación de una oferta de máquina virtual](./partner-center-portal/azure-vm-create-offer.md).
