---
title: Visualización de las cuentas de facturación en Azure Portal
description: Obtenga información sobre cómo ver las cuentas de facturación en Azure Portal. Consulte la información de ámbito de los contratos Enterprise, los contratos de clientes de Microsoft y Microsoft Partner Agreement.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: banders
ms.openlocfilehash: 7763340831f1ffbc5ae33b5cc7bff5696d7a9b96
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461704"
---
# <a name="billing-accounts-and-scopes-in-the-azure-portal"></a>Cuentas de facturación y ámbitos en Azure Portal

Cuando se registra para usar Azure, se crea una cuenta de facturación. Use su cuenta de facturación para administrar las facturas, los pagos y hacer seguimiento de los costos. Puede tener acceso a varias cuentas de facturación. Por ejemplo, es posible que te hayas registrado en Azure para tus proyectos personales. También se puede tener acceso a Azure a través del Contrato Enterprise de la organización o del contrato de cliente de Microsoft. Para cada uno de estos escenarios, se dispondría de una cuenta de facturación independiente.

Azure Portal admite actualmente el tipo siguiente de cuentas de facturación:

- **Microsoft Online Services Program**: Se crea una cuenta de facturación para Microsoft Online Services Programa cuando se registra en el sitio web de Azure. Por ejemplo, cuando se registra para obtener una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), como una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o como un [suscriptor de Visual Studio](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/).

- **Contrato Enterprise**: Se crea una cuenta de facturación para un Contrato Enterprise cuando la organización firma este [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) para utilizar Azure. En un Contrato Enterprise no puede tener más de 2000 suscripciones.

- **Contrato de cliente de Microsoft**: Se crea una cuenta de facturación para un contrato de cliente de Microsoft cuando una organización trabaja con un representante de Microsoft para firmar un contrato de cliente de Microsoft. Algunos clientes en regiones seleccionadas, que se registran en el sitio web de Azure para una [cuenta con tarifas de Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), también pueden tener una cuenta de facturación para un Contrato de cliente de Microsoft. Puede tener un máximo de 20 suscripciones en un Contrato de cliente de Microsoft. Para obtener más información, consulte [Empezar a trabajar con la cuenta de facturación para un contrato de cliente de Microsoft](../understand/mca-overview.md).

- **Microsoft Partner Agreement**: Para los asociados del programa Proveedor de soluciones en la nube (CSP), se crea una cuenta de facturación para un acuerdo de Microsoft Partner Agreement con el fin de administrar sus clientes en la nueva experiencia comercial. Los asociados deben tener al menos un cliente con un [plan de Azure](https://docs.microsoft.com/partner-center/purchase-azure-plan) para administrar su cuenta de facturación en Azure Portal. Para obtener más información, consulte [Introducción a la cuenta de facturación para un contrato Microsoft Partner Agreement](../understand/mpa-overview.md).

Para determinar el tipo de la cuenta de facturación, consulte [Comprobación del tipo de la cuenta de facturación](#check-the-type-of-your-account).

## <a name="scopes-for-billing-accounts"></a>Ámbitos para las cuentas de facturación
Un ámbito es un nodo dentro de una cuenta de facturación que los usuarios utilizan para ver y administrar la facturación. Es donde se administran los datos de facturación, los pagos, las facturas y realizar la administración de cuentas generales.

### <a name="microsoft-online-services-program"></a>Microsoft Online Services Program

![Captura de pantalla que muestra la jerarquía de MOSP](./media/view-all-accounts/mosp-hierarchy.png)

|Ámbito  |Definición  |
|---------|---------|
|Cuenta de facturación     | Representa un contrato que un cliente acepta para usar Azure. Contiene una o varias suscripciones.  |
|Suscripción     |  Representa una agrupación de los recursos de Azure. La factura se genera en este ámbito. Otra información de facturación, como los métodos de pago y la dirección de uso, está asociada a este ámbito.|

### <a name="enterprise-agreement"></a>Contrato Enterprise

![Captura de pantalla que muestra la jerarquía de EA](./media/view-all-accounts/ea-hierarchy.png)

|Ámbito  |Definición  |
|---------|---------|
|Cuenta de facturación    | Representa una inscripción de Contrato Enterprise. Contiene uno o varios departamentos y cuentas. La factura se genera en este ámbito. |
|department     |  Agrupación opcional de cuentas para segmentar los costos en agrupaciones lógicas y establecer el presupuesto.     |
|Cuenta     |  Representa al propietario de una cuenta única. Los propietarios de la cuenta tienen permisos para crear y administrar suscripciones de Azure que se facturan a la inscripción. |

### <a name="microsoft-customer-agreement"></a>Contrato de cliente de Microsoft

![Captura de pantalla que muestra la jerarquía de MCA](./media/view-all-accounts/mca-hierarchy.png)

|Ámbito  |Tareas  |
|---------|---------|
|Cuenta de facturación     |   Representa un contrato de cliente para los productos y servicios de Microsoft. Contiene uno o varios perfiles de facturación. |
|Perfil de facturación     |   Representa una factura y la información de facturación relacionada, como los métodos de pago y la dirección de facturación. Contiene una o varias secciones de factura. |
|Sección de factura     |   Representa una agrupación de costos en una factura. Las suscripciones de Azure y otras compras, como Azure Marketplace y productos de origen de la aplicación, están asociadas a este ámbito.    |

### <a name="microsoft-partner-agreement"></a>Microsoft Partner Agreement

![Captura de pantalla que muestra la jerarquía de MPA](./media/view-all-accounts/mpa-hierarchy.png)

|Ámbito  |Tareas  |
|---------|---------|
|Cuenta de facturación     |   Representa un contrato del asociado para administrar los productos y servicios de Microsoft de los clientes en la nueva experiencia comercial. Contiene uno o varios perfiles de facturación y clientes.   |
|Perfil de facturación     |   Representa una factura para una moneda.     |
|Customer    |   Representa un cliente para un asociado del programa Proveedor de soluciones en la nube (CSP).  Las suscripciones de Azure y otras compras, como Azure Marketplace y productos de origen de la aplicación, están asociadas a este ámbito.  |
|Vendedor    |   Revendedor que proporciona servicios a un cliente. Este es un campo opcional para una suscripción y solo se aplica a los proveedores indirectos en el modelo de dos niveles de CSP.     |

## <a name="switch-billing-scope-in-the-azure-portal"></a>Cambio del ámbito de facturación en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/view-all-accounts/billing-search-cost-management-billing.png)

3. En la página de información general, seleccione **Switch scope** (Cambiar ámbito).

   ![Captura de pantalla que muestra los ámbitos de facturación](./media/view-all-accounts/overview-select-scopes.png)

   > [!Note]
    >
    > No verá Swich scope (Cambiar ámbito) si solo tiene acceso a un ámbito.

4. Seleccione un ámbito para ver los detalles.

   ![Captura de pantalla que muestra los ámbitos de facturación](./media/view-all-accounts/list-of-scopes.png)

## <a name="check-the-type-of-your-account"></a>Comprobación del tipo de la cuenta
[!INCLUDE [billing-check-mca](../../../includes/billing-check-account-type.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes
- Sepa cómo empezar a [analizar los costos](../costs/quick-acm-cost-analysis.md).
