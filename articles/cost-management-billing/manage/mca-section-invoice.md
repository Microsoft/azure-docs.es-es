---
title: 'Organice su factura en función de sus necesidades: Azure'
description: Obtenga información acerca de cómo organizar los costos en su factura. Puede personalizar su cuenta de facturación al crear perfiles de facturación y secciones de factura.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 9f5e69a609931f4f8ead62f4d5c5ba2a1064a6ea
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683018"
---
# <a name="organize-costs-by-customizing-your-billing-account"></a>Organización de costos mediante la personalización de la cuenta de facturación

Su cuenta de facturación para el Contrato de cliente de Microsoft proporciona flexibilidad para organizar los costos en función de sus necesidades, ya sea por departamento, proyecto o entorno de desarrollo.

En este artículo se describe cómo usar Azure Portal para organizar sus costos. Se aplica a una cuenta de facturación para un Contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

## <a name="structure-your-account-with-billing-profiles-and-invoice-sections"></a>Estructure la cuenta mediante el uso de perfiles de facturación y secciones de factura.

En la cuenta de facturación de un Contrato de cliente de Microsoft, utilizará perfiles de facturación y secciones de factura para organizar sus costos.

![Captura de pantalla que muestra la jerarquía de facturación del Contrato de cliente de Microsoft](./media/mca-section-invoice/mca-hierarchy.png)

### <a name="billing-profile"></a>Perfil de facturación

Una perfil de facturación representa una factura y la información de facturación relacionada, como los métodos de pago y la dirección de facturación. Una factura mensual se genera al comienzo del mes para cada perfil de facturación de la cuenta. La factura incluye los cargos por uso de Azure y otras compras del mes anterior.

Al suscribirse a Azure, se crea automáticamente un perfil de facturación junto con su cuenta de facturación. Puede crear perfiles de facturación adicionales para organizar los costos en varias facturas mensuales.

> [!IMPORTANT]
>
> La creación de perfiles de facturación adicionales puede afectar al costo general. Para obtener más información, consulte [Aspectos a tener en cuenta al agregar nuevos perfiles de facturación](#things-to-consider-when-adding-new-billing-profiles).

### <a name="invoice-section"></a>Sección de factura

Una sección de factura representa una agrupación de costos en la factura. Esta se crea automáticamente para cada perfil de facturación de la cuenta. Puede crear secciones adicionales para organizar los costos en función de sus necesidades. Cada sección de factura se muestra en la factura con los cargos incurridos en ese mes.

En la imagen siguiente se muestra una factura con dos secciones de factura: Ingeniería y Marketing. En cada sección de la factura, se muestra un resumen de los cargos y los detalles de estos. Los precios que se muestran en la imagen son solo para fines de ejemplo y no representan los precios reales de los servicios de Azure.

![Imagen que muestra una factura con secciones](./media/mca-section-invoice/mca-invoice-with-sections.png)

## <a name="billing-account-structure-for-common-scenarios"></a>Estructura de la cuenta de facturación para escenarios comunes

En esta sección se describen escenarios comunes para la organización de los costos y las estructuras de las cuentas de facturación correspondientes:

|Escenario  |Estructura  |
|---------|---------|
|José se suscribe a Azure y necesita obtener una sola factura mensual. | Un perfil de facturación y una sección de factura. Esta estructura se configura automáticamente para José cuando se suscribe a Azure y no requiere ningún paso adicional. |

![Gráfico de información para un escenario de facturación simple](./media/mca-section-invoice/organize-billing-scenario1.png)

|Escenario  |Estructura  |
|---------|---------|
|Contoso es una pequeña organización que necesita una sola factura mensual, pero agrupa los costos por sus departamentos (de marketing e ingeniería).  | Un perfil de facturación para Contoso y una sección de factura para los departamentos de marketing e ingeniería. |

![Gráfico de información para un escenario de facturación simple](./media/mca-section-invoice/organize-billing-scenario2.png)

|Escenario  |Estructura  |
|---------|---------|
|Fabrikam es una organización mediana que necesita facturas independientes para sus departamentos de ingeniería y marketing. En cuanto al departamento de ingeniería, quiere agrupar los costos por los entornos de producción y desarrollo.  | Un perfil de facturación para los departamentos de marketing e ingeniería. Para el departamento de ingeniería, una sección de factura para el entorno de producción y desarrollo. |

![Gráfico de información para un escenario de facturación simple](./media/mca-section-invoice/organize-billing-scenario3.png)

## <a name="create-a-new-invoice-section"></a>Creación de una sección de factura

Para crear una sección de factura, debe ser **propietario del perfil de facturación** o **colaborador de perfil de facturación**. Para obtener más información, consulte [Administración de las secciones de factura para el perfil de facturación](understand-mca-roles.md#manage-invoice-sections-for-billing-profile).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/mca-section-invoice/search-cmb.png)

3. Seleccione **Perfiles de facturación** en el panel izquierdo. En la lista, seleccione un perfil de facturación. La nueva sección se mostrará en la factura del perfil de facturación seleccionado.

   [![Captura de pantalla que muestra la lista de perfiles de facturación](./media/mca-section-invoice/mca-select-profile.png)](./media/mca-section-invoice/mca-select-profile-zoomed-in.png#lightbox)

4. Seleccione **Secciones de factura** en el panel izquierdo y, a continuación, **Agregar** en la parte superior de la página.

   [![Captura de pantalla que muestra la adición de las secciones de factura](./media/mca-section-invoice/mca-list-invoice-sections.png)](./media/mca-section-invoice/mca-list-invoice-sections-zoomed-in.png#lightbox)

5. Escriba un nombre para la sección de factura.

   [![Captura de pantalla que muestra la página de creación de secciones de factura](./media/mca-section-invoice/mca-create-invoice-section.png)](./media/mca-section-invoice/mca-create-invoice-section-zoomed-in.png#lightbox)

6. Seleccione **Crear**.

## <a name="create-a-new-billing-profile"></a>Creación de un nuevo perfil de facturación

Para crear un perfil de facturación, debe ser **propietario de la cuenta de facturación** o **colaborador de la cuenta de facturación**. Para obtener más información, consulte [Administrar perfiles de facturación para la cuenta de facturación](understand-mca-roles.md#manage-billing-profiles-for-billing-account).

> [!IMPORTANT]
>
> La creación de perfiles de facturación adicionales puede afectar al costo general. Para obtener más información, consulte [Aspectos a tener en cuenta al agregar nuevos perfiles de facturación](#things-to-consider-when-adding-new-billing-profiles).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/mca-section-invoice/search-cmb.png)

3. Seleccione **Perfiles de facturación** en el panel izquierdo y, a continuación, **Agregar** en la parte superior de la página.

   [![Captura de pantalla que muestra la lista de perfiles de facturación](./media/mca-section-invoice/mca-list-profiles.png)](./media/mca-section-invoice/mca-list-profiles-zoomed-in.png#lightbox)

    > [!Note]
    >
    > Si no ve el botón Agregar en la página Perfil de facturación, significa que la característica no está disponible para su cuenta. Actualmente, solo está disponible para las cuentas que se han configurado mientras se trabaja con un representante de Microsoft.

4. Complete el formulario y haga clic en **Crear**.

   [![Captura de pantalla que muestra la página de creación del perfil de facturación](./media/mca-section-invoice/mca-add-profile.png)](./media/mca-section-invoice/mca-add-profile-zoomed-in.png#lightbox)

    |Campo  |Definición  |
    |---------|---------|
    |Nombre     | El nombre para mostrar ayuda a identificar fácilmente el perfil de facturación en Azure Portal.  |
    |Número de pedido de compra    | Número de pedido de compra opcional. El número de pedido de compra se mostrará en las facturas generadas para el perfil de facturación. |
    |Dirección de facturación   | La dirección de facturación se mostrará en las facturas generadas para el perfil de facturación. |
    |Enviar factura por correo electrónico   | Active la casilla Enviar factura por correo electrónico para recibir las facturas de este perfil de facturación por correo electrónico. Si no opta por recibirlas, podrá consultarlas y descargarlas en Azure Portal.|

5. Seleccione **Crear**.

## <a name="link-charges-to-invoice-sections-and-billing-profiles"></a>Vinculación de cargos a secciones de factura y perfiles de facturación

Una vez que haya personalizado su cuenta de facturación en función de sus necesidades, puede vincular las suscripciones y otros productos a la sección de factura y al perfil de facturación que quiera.

### <a name="link-a-new-subscription"></a>Vinculación de una nueva suscripción

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Suscripciones**.

   [![Captura de pantalla que muestra la búsqueda de suscripciones en el portal](./media/mca-section-invoice/search-subscriptions.png)](./media/mca-section-invoice/search-subscriptions.png#lightbox)

3. Seleccione **Agregar** en la parte superior de la página.

   ![Captura de pantalla que muestra el botón Agregar en la vista Suscripciones](./media/mca-section-invoice/subscription-add.png)

4. Si tiene acceso a varias cuentas de facturación, seleccione la cuenta de facturación de su Contrato de cliente de Microsoft.

   ![Captura de pantalla que muestra el botón Agregar en la vista Suscripciones](./media/mca-section-invoice/mca-create-azure-subscription.png)

5. Seleccione el perfil de facturación que se facturará por el uso de la suscripción. Los cargos por uso de Azure y otras compras de esta suscripción se facturarán a la factura del perfil de facturación seleccionado.

6. Seleccione la sección de factura a la que quiere vincular los cargos de la suscripción. Los cargos se mostrarán en esta sección de la factura del perfil de facturación.

7. Seleccione un plan de Azure y escriba un nombre descriptivo para la suscripción.

9. Haga clic en **Crear**.  

### <a name="link-existing-subscriptions-and-products"></a>Vinculación de suscripciones y productos existentes

Si tiene suscripciones a Azure existentes u otros productos, como Azure Marketplace y recursos de origen de la aplicación, puede moverlos de la sección de factura existente a otra para reorganizar los costos.

> [!IMPORTANT]
>
> Las suscripciones y otros productos solo pueden moverse entre las secciones de factura que pertenezcan al mismo perfil de facturación. No se admite la transferencia de suscripciones y productos entre las secciones de factura de distintos perfiles de facturación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda de suscripciones en el portal](./media/mca-section-invoice/search-cmb.png)

3. Para vincular una suscripción a una nueva sección de factura, seleccione **Suscripciones a Azure** en el lado izquierdo de la pantalla. En el caso de otros productos como Azure Marketplace y recursos de origen de la aplicación, seleccione **Cargos periódicos**.

   [![Captura de pantalla que muestra la opción para cambiar la sección de factura](./media/mca-section-invoice/mca-select-change-invoice-section.png)](./media/mca-section-invoice/mca-select-change-invoice-section.png#lightbox)

4. En la página, haga clic en los puntos suspensivos (tres puntos) de la suscripción o el producto que quiera vincular a una nueva sección de factura. Seleccione **Cambiar sección de factura**.

5. Seleccione la sección de factura nueva en el menú desplegable. La lista desplegable solo mostrará las secciones de factura asociadas al mismo perfil de facturación que la sección de factura existente.

    [![Captura de pantalla que muestra la selección de una sección de factura nueva](./media/mca-section-invoice/mca-select-new-invoice-section.png)](./media/mca-section-invoice/mca-select-new-invoice-section-zoomed-in.png#lightbox)

6. Seleccione **Guardar**.

## <a name="things-to-consider-when-adding-new-billing-profiles"></a>Aspectos a tener en cuenta al agregar nuevos perfiles de facturación

### <a name="azure-usage-charges-may-be-impacted"></a>Los cargos de uso de Azure pueden verse afectados

En la cuenta de facturación de un Contrato de cliente de Microsoft, cada mes se agrega el uso de Azure de cada perfil de facturación. Los precios de los recursos de Azure con precios por niveles se determinan en función del uso de cada perfil de facturación por separado. El uso no se agrega en los perfiles de facturación al calcular el precio. Esto puede afectar al costo general del uso de Azure de las cuentas con varios perfiles de facturación.

Echemos un vistazo a un ejemplo de cómo varían los costos en dos escenarios. Los precios que se usan en los escenarios son solo para fines de ejemplo y no representan los precios reales de los servicios de Azure.

#### <a name="you-only-have-one-billing-profile"></a>Solo tiene un perfil de facturación.

Supongamos que usa el almacenamiento de blobs en bloques de Azure, que cuesta 0,00184 USD por GB por los primeros 50 terabytes (TB) y 0,00177 USD por GB por los siguientes 450 terabytes (TB). Usó 100 TB en las suscripciones que se facturan a su perfil de facturación. A continuación, se indica cuánto se le cobrará.

|  Precios por niveles (USD) |Cantidad | Importe (USD)|
|---------|---------|---------|
|1,84 por TB para los primeros 50 TB/mes    | 50 TB        | 92,0   |
|1,77 por TB para los siguientes 450 TB/mes    |  50 TB         | 88,5   |
|Total     |     100 TB  | 180,5

Los cargos totales por usar 100 TB de datos en este escenario son de **180,5**.

#### <a name="you-have-multiple-billing-profiles"></a>Tiene varios perfiles de facturación.

Ahora, supongamos que creó otro perfil de facturación y usó 50 GB a través de suscripciones que se facturan en el primer perfil de facturación y 50 GB a través de suscripciones que se facturan en el segundo perfil de facturación. A continuación, se muestra cuánto se le cobrará.

`Charges for the first billing profile`

|  Precios por niveles (USD) |Cantidad | Importe (USD)|
|---------|---------|---------|
|1,84 por TB para los primeros 50 TB/mes    | 50 TB        | 92,0  |
|1,77 por TB para los siguientes 450 TB/mes    |  0 TB         | 0,0  |
|Total     |     50 TB  | 92,0

`Charges for the second billing profile`

|  Precios por niveles (USD) |Cantidad | Importe (USD)|
|---------|---------|---------|
|1,84 por TB para los primeros 50 TB/mes    | 50 TB        | 92,0  |
|1,77 por TB para los siguientes 450 TB/mes    |  0 TB         | 0,0  |
|Total     |     50 TB  | 92,0

Los cargos totales por usar 100 TB de datos en este escenario son de **184,0** (92,0 * 2).

### <a name="azure-reservation-benefits-might-not-apply-to-all-subscriptions"></a>Es posible que las ventajas de la reserva de Azure no se apliquen a todas las suscripciones

Las reservas de Azure con ámbito compartido se aplican a las suscripciones de un solo perfil de facturación y no se comparten entre los perfiles de facturación.

![Gráfico informativo de la aplicación de reserva para diferentes estructuras de cuentas de facturación](./media/mca-section-invoice/mca-reservations-benefits-by-bg.png)

En la imagen anterior, Contoso tiene dos suscripciones. La ventaja de la reserva de Azure se aplica de manera diferente en función de la estructura de la cuenta de facturación. En el escenario de la izquierda, la ventaja de reserva se aplica a las dos suscripciones que se facturan en el perfil de facturación de ingeniería. En el escenario de la derecha, la ventaja de reserva solo se aplica a la suscripción 1, ya que es la única que se factura en el perfil de facturación de ingeniería.

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una suscripción a Azure adicional para el contrato de cliente de Microsoft](create-subscription.md)
- [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal)
- [Obtención de la propiedad de la facturación de las suscripciones de usuarios de otras cuentas de facturación](mca-request-billing-ownership.md)
