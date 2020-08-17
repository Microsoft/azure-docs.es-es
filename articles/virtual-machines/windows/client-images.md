---
title: Uso de imágenes de cliente de Windows en Azure
description: Cómo usar las ventajas para la suscripción de Visual Studio a fin de implementar Windows 7, Windows 8 o Windows 10 en Azure para escenarios de desarrollo/pruebas
author: cynthn
ms.subservice: imaging
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: 26bfd9a65cd3e6d36b8190e87d382543cc7f0f60
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292060"
---
# <a name="use-windows-client-in-azure-for-devtest-scenarios"></a>Uso del cliente Windows en Azure para escenarios de desarrollo y pruebas
Puede usar Windows 7, Windows 8 o Windows 10 Enterprise (x64) en Azure para escenarios de desarrollo y pruebas siempre que tenga una suscripción adecuada a Visual Studio (anteriormente MSDN). En este artículo se describen los requisitos de idoneidad para ejecutar Windows 7, Windows 8.1 o Windows 10 Enterprise en Azure y usar las siguientes imágenes de la galería de Azure.

![Detalles de la imagen de Azure Portal](./media/client-images/windows-client-msdn-images.png) 

> [!NOTE]
> Para la imagen de Windows 10 Pro y Windows 10 Pro N en la galería de Azure, consulte [Implementación de Windows 10 en Azure con derechos de hospedaje multiinquilino](windows-desktop-multitenant-hosting-deployment.md)
>![Detalles de la imagen profesional de Azure Portal](./media/client-images/windows-client-pro-images.png) 
>

## <a name="subscription-eligibility"></a>Idoneidad de la suscripción
Los suscriptores activos de Visual Studio (es decir, las personas que adquirieron una licencia de suscripción de Visual Studio) pueden usar el cliente Windows para desarrollo y pruebas. El cliente Windows se puede usar en su propio hardware y en máquinas virtuales de Azure que se ejecutan en cualquier tipo de suscripción de Azure. El cliente Windows no se puede implementar ni usar en Azure para un uso de producción normal. Tampoco lo pueden usar quienes no son suscriptores activos de Visual Studio.

Para su comodidad, ciertas imágenes de Windows 10 están disponibles en la galería de Azure dentro de las [ofertas de desarrollo y pruebas a las que tiene acceso](#eligible-offers). Los suscriptores de Visual Studio dentro de cualquier tipo de oferta también pueden [preparar y crear apropiadamente](prepare-for-upload-vhd-image.md) una imagen de Windows 7, Windows 8 o Windows 10 de 64 bits y, luego, [cargarla en Azure](upload-generalized-managed.md). El uso sigue limitado a desarrollo y pruebas por parte de los suscriptores activos de Visual Studio.

## <a name="eligible-offers"></a>Ofertas a las que se puede tener acceso
La tabla siguiente muestra los detalles de los id. de oferta idóneos para implementar Windows 10 a través de la galería de Azure. Las imágenes de Windows 10 solo son visibles para las ofertas siguientes. Los suscriptores de Visual Studio que deban ejecutar el cliente Windows en un tipo de oferta distinto necesitan que [prepare y cree apropiadamente](prepare-for-upload-vhd-image.md) una imagen de Windows 7, Windows 8 o Windows 10 de 64 bits y, [luego, la cargue en Azure](upload-generalized-managed.md).

| Nombre de la oferta | Número de la oferta | Imágenes de cliente disponibles |
|:--- |:---:|:---:|
| [Desarrollo/pruebas - Pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/) |0023P |Windows 10 |
| [Suscriptores de Visual Studio Enterprise (MPN)](https://azure.microsoft.com/offers/ms-azr-0029p/) |0029P |Windows 10 |
| [Suscriptores de Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/) |0059P |Windows 10 |
| [Suscriptores de Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/) |0060P |Windows 10 |
| [Visual Studio Premium con MSDN (ventaja)](https://azure.microsoft.com/offers/ms-azr-0061p/) |0061P |Windows 10 |
| [Suscriptores de Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) |0063P |Windows 10 |
| [Suscriptores de Visual Studio Enterprise (BizSpark)](https://azure.microsoft.com/offers/ms-azr-0064p/) |0064P |Windows 10 |
| [Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/) |0148P |Windows 10 |

## <a name="check-your-azure-subscription"></a>Compruebe la suscripción a Azure
Si no conoce el identificador de su oferta, puede obtenerlo en Azure Portal de una de estas dos formas:  

- En la ventana *Suscripciones*:

  ![Detalles del id. de oferta desde el Portal de Azure](./media/client-images/offer-id-azure-portal.png) 

- O bien, haga clic en **Facturación** y, a continuación, haga clic en el identificador de suscripción. El identificador de oferta aparece en la ventana de *facturación*.

También puede ver el id. de la oferta en la pestaña ["Suscripciones"](https://account.windowsazure.com/Subscriptions) del portal de la cuenta de Azure:

![Detalles del id. de oferta desde el portal de la cuenta de Azure](./media/client-images/offer-id-azure-account-portal.png) 

## <a name="next-steps"></a>Pasos siguientes
Ahora puede implementar las máquinas virtuales con [PowerShell](quick-create-powershell.md), [plantillas de Resource Manager](ps-template.md) o [Visual Studio](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md).
