---
title: Azure Marketplace
description: Describe de qué forma pueden usar Azure Marketplace los clientes de Contrato Enterprise
author: bandersmsft
ms.reviewer: baolcsva
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 09/03/2020
ms.author: banders
ms.openlocfilehash: 0f2d3c830f27eec9f521e6f79ac8dce3bce818e9
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442571"
---
# <a name="azure-marketplace"></a>Azure Marketplace

En este artículo de qué forma los clientes y asociados de Contrato Enterprise pueden ver los cargos de Marketplace y habilitar las compras en Azure Marketplace.

## <a name="azure-marketplace-for-ea-customers"></a>Azure Marketplace para clientes de EA

Para los clientes directos, los cargos de Azure Marketplace se pueden ver en Azure Enterprise Portal. Las compras y los consumos de Azure Marketplace se facturan fuera del prepago de Azure, en plazos mensuales o trimestrales a pago vencido.

Los clientes indirectos pueden encontrar sus suscripciones a Azure Marketplace en la página **Administrar suscripciones** de Azure Enterprise Portal, pero no se muestran los precios. Los clientes deben ponerse en contacto con su Proveedor de soluciones de licencia (LSP) para obtener información sobre los cargos en Azure Marketplace.

Las nuevas compras periódicas mensuales o anuales de Azure Marketplace se facturan por completo durante el período en que se compran los elementos de Azure Marketplace. Estos artículos se renovarán automáticamente en el siguiente período el mismo día de la compra original.

Los cargos periódicos mensuales existentes se seguirán renovando el primero de cada mes calendario. Los cargos anuales se renovarán en el aniversario de la fecha de compra.

Algunos servicios de revendedores de terceros disponibles en Azure Marketplace ahora consumen el saldo del prepago de Azure del Contrato Enterprise (EA). Antes, estos servicios se facturaban por separado, fuera del prepago de Azure de EA. El prepago de Azure de EA para estos servicios de Azure Marketplace simplifica las compras y la administración de los pagos para los clientes. Para ver una lista completa de los servicios que actualmente consumen prepago de Azure, consulte la [actualización del 6 de marzo de 2018 en el sitio web de Azure](https://azure.microsoft.com/updates/azure-marketplace-third-party-reseller-services-now-use-azure-monetary-commitment/).

### <a name="partners"></a>Asociados

Los LSP pueden descargar una lista de precios de Azure Marketplace en la página de la hoja de precios en Azure Enterprise Portal. Seleccione el vínculo **Lista de precios de Marketplace** en la esquina superior derecha. La lista de precios de Azure Marketplace muestra todos los servicios disponibles y sus precios.

Para descargar la hoja de precios:

1. En Azure Enterprise Portal, vaya a **Informes** > **Hoja de precios**.
1. En la esquina superior derecha, busque el vínculo a la lista de precios de Azure Marketplace debajo de su nombre de usuario.
1. Haga clic con el botón derecho en el vínculo y seleccione **Guardar destino como**.
1. En la ventana **Guardar**, cambie el título del documento a `AzureMarketplacePricelist.zip`, que cambiará el archivo de .xlsx a .zip.
1. Después de completada la descarga, tendrá un archivo ZIP con listas de precios específicas del país o región.
1. Los LSP deben hacer referencia al archivo individual del país para obtener los precios específicos del país o región. Los LSP pueden usar la pestaña **Notificaciones** para estar al tanto de las SKU que son nuevas o retiradas.
1. Los cambios de precio son poco frecuentes. Los LSP reciben notificaciones por correo electrónico de los aumentos de precios y las variaciones en los tipos de cambio (FX) con 30 días de antelación.
1. Los LSP reciben una factura por inscripción, por ISV y por trimestre.

### <a name="enabling-azure-marketplace-purchases"></a>Habilitación de compras en Azure Marketplace

Los administradores de empresa pueden deshabilitar o habilitar las compras de Azure Marketplace para todas las suscripciones de Azure en su inscripción. Si el administrador de empresa deshabilita las compras, y hay suscripciones de Azure que ya tienen suscripciones a Azure Marketplace, esas suscripciones a Azure Marketplace no se cancelarán ni se verán afectadas.

Aunque los clientes pueden convertir sus suscripciones de Azure directas en Azure EA si las asocian a su inscripción en Azure Enterprise Portal, esta acción no convierte automáticamente las suscripciones secundarias.

Para habilitar compras en Azure Marketplace:

1. Inicie sesión en Azure Enterprise Portal como administrador de empresa.
1. Vaya a **Administrar**.
1. En **Enrollment Detail** (Detalles de la inscripción), seleccione el icono de lápiz situado junto al elemento de línea **Azure Marketplace**.
1. Cambie entre **Habilitado/Deshabilitado** o **Solo SKU BYOL o libres**, según corresponda.
1. Seleccione **Guardar**.

> [!NOTE]
> Las opciones BYOL (traiga su propia licencia) y solo gratis limitan la compra y adquisición de SKU de Azure Marketplace a solo SKU BYOL y gratuitas.

### <a name="services-billed-hourly-for-azure-ea"></a>Servicios facturados por horas para Azure EA

Los siguientes servicios se facturan por horas en un Contrato Enterprise, en lugar de la tarifa mensual de MOSP:

- Red de entrega de aplicaciones
- Firewall de aplicaciones web

### <a name="azure-remoteapp"></a>Azure RemoteApp

Si tiene un Contrato Enterprise, paga por Azure RemoteApp según el nivel de precios del Contrato Enterprise. No existen cargos adicionales. El precio estándar incluye 40 horas iniciales. El precio ilimitado cubre 80 horas iniciales. RemoteApp deja de emitir el uso por encima de las 80 horas.

## <a name="azure-marketplace-faq"></a>P+F sobre Azure Marketplace

En esta sección se explica cómo se puede aplicar el prepago de Azure a algunos servicios de revendedores de terceros en Azure Marketplace.

### <a name="what-changed-with-azure-marketplace-services-and-azure-ea-prepayment"></a>¿Qué ha cambiado con los servicios de Azure Marketplace y el prepago de Azure de EA?

A partir del 1 de marzo de 2018, algunos servicios de revendedores de terceros consumen el prepago de Azure de EA. A excepción de las instancias reservadas (RI) de máquina virtual de Azure, antes los servicios se facturaban fuera del prepago de Azure de EA y por separado.

Ampliamos el uso del prepago de Azure para incluir algunos de los servicios de Azure Marketplace publicados por terceros que se compran con más frecuencia. El prepago de Azure de EA para estos servicios de Azure Marketplace simplifica las compras y la administración de los pagos.

### <a name="why-did-we-make-this-change"></a>¿Por qué hemos realizado este cambio?

Los clientes buscan continuamente otras formas de aprovechar el prepago de Azure. Los clientes solicitaron este cambio a menudo, y afectó a una gran parte de los clientes de Azure Marketplace.

### <a name="how-do-you-benefit"></a>¿En qué se beneficia?

Obtiene una experiencia de facturación más sencilla y podrá gastar mejor su prepago de Azure de EA. Dado que estos servicios se incluyen en el prepago de Azure de EA, este es más valioso.

### <a name="what-azure-marketplace-services-use-azure-ea-prepayment-and-how-do-i-know"></a>¿Qué servicios de Azure Marketplace usan el prepago de Azure de EA y cómo lo averiguo?

Cuando compra un servicio que usa el prepago de Azure, Azure Marketplace presenta una declinación de responsabilidades. Se admiten algunos servicios publicados por Red Hat, SUSE, Autodesk y Oracle. Actualmente, los servicios con el mismo nombre publicados por otros terceros no se deducen del prepago de Azure. Al final de esta sección de preguntas frecuentes encontrará una lista completa.

### <a name="what-if-my-azure-ea-prepayment-runs-out"></a>¿Qué ocurre si se agota el prepago de Azure de EA?

Si consume todo su prepago de Azure y pasa a uso por encima del límite, los cargos relacionados con estos servicios aparecerán en la próxima factura de uso por encima del límite, junto con cualquier otro servicio de consumo. Antes del cambio del 1 de marzo de 2018, estos cargos se facturaban con otros servicios de Azure Marketplace.

### <a name="why-dont-all-azure-marketplaces-consume-azure-ea-prepayment"></a>¿Por qué no todos los marketplaces de Azure consumen el prepago de Azure de EA?

Hacemos lo posible por ofrecer la mejor experiencia de cliente relacionada con el prepago de Azure de EA. Este cambio afectó un gran número de clientes y a una parte importante del gasto total en Azure Marketplace. Es posible que se agreguen otros servicios en el futuro.

### <a name="how-does-this-impact-indirect-enrollment-and-partners"></a>¿Cómo afecta esto a la inscripción indirecta y los asociados?

Nuestros asociados o clientes de inscripciones indirectas no se verán afectados. Estos servicios están sujetos a las mismas funcionalidades de marcado de asociados que otros servicios de consumo. El único cambio es que los cargos aparecen en una factura diferente, y el pago de los cargos queda fuera del prepago de Azure de EA del cliente.

### <a name="is-there-a-list-of-azure-marketplace-services-that-consume-azure-ea-prepayment"></a>¿Hay una lista de servicios de Azure Marketplace que consumen prepago de Azure de EA?

Algunas ofertas específicas de Azure Marketplace pueden usar fondos del prepago de Azure. Consulte los [servicios de terceros que usan prepago de Azure](https://azure.microsoft.com/updates/azure-marketplace-third-party-reseller-services-now-use-azure-monetary-commitment) para obtener una lista completa de productos que participan en este programa.


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información acerca de los [precios](ea-pricing-overview.md).