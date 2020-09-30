---
title: Planes y precios de las ofertas del marketplace comercial
description: Obtenga información acerca de los planes de las ofertas del marketplace comercial en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: mingshen
ms.author: mingshen
ms.date: 09/02/2020
ms.openlocfilehash: ef860d57556e17d432988fd0fd43f861a31fde6c
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380506"
---
# <a name="plans-and-pricing-for-commercial-marketplace-offers"></a>Planes y precios de las ofertas del marketplace comercial

Los planes definen el ámbito y los límites de las ofertas, así como los precios asociados, cuando corresponda. Por ejemplo, en función del tipo de oferta, puede seleccionar mercados regionales y elegir si un plan puede verlo todo el público o solo una audiencia privada. Algunos tipos de oferta admiten suscripciones recurrentes, otros admiten precios basados en el uso, mientras que otros permiten a los clientes adquirir la oferta con una licencia que han comprado directamente al publicador. Esto le ofrece la flexibilidad necesaria para proporcionar a sus clientes distintas opciones técnicas y de precio, cuando proceda.

Puede crear un máximo de 100 planes para cada oferta que admita planes y hasta 45 de ellos pueden ser [privados](#plan-visibility). Puede crear planes de pago solo para ofertas de máquinas virtuales de Azure, ofertas de aplicaciones de Azure (aplicaciones administradas) y ofertas de software como servicio (SaaS). Si opta por vender cualquiera de estas ofertas a través de Microsoft (ofertas comercializables), debe crear al menos un plan. Puede crear planes para algunos de los otros tipos de oferta, pero esos planes no incluyen opciones de precios.

> [!TIP]
> Una oferta comercializable es aquella en la que Microsoft facilita el intercambio de dinero por una licencia de software en nombre del editor.

## <a name="plans-by-offer-type"></a>Planes por tipo de oferta

En la tabla siguiente se muestran las opciones de los distintos planes de cada tipo de oferta. En las secciones siguientes se proporcionará más información sobre estas opciones. 

| Tipo de oferta | Planes con opciones de precios | Planes sin opciones de precios | Opción de audiencia privada |
| ------------ | ------------- | ------------- | ------------- |
| Aplicación administrada por Azure | &#10004; | | &#10004; |
| Plantilla de solución de Azure | | &#10004; | &#10004; |
| Contenedor de Azure | | &#10004; (BYOL) | |
| Módulo IoT Edge |  | &#10004; |  |
| Servicio administrado |  | &#10004; (BYOL) | &#10004; |
| Software como servicio | &#10004; |  | &#10004; |
| Máquina virtual de Azure | &#10004; |  | &#10004; |
|||||

Los siguientes tipos de ofertas admiten planes:
-  Servicios de consultoría
-  Dynamics 365 Business Central
-  Dynamics 365 Customer Engagement & PowerApps
-  Dynamics 365 for Operations
-  Power BI app

## <a name="plan-information"></a>Información de los planes

Cada tipo de oferta requiere información diferente cuando se crea un plan. Puede encontrar vínculos a artículos específicos de cada oferta en [Guía de publicación por tipo de oferta](publisher-guide-by-offer-type.md). Una vez que se crea un plan en la página de **Información general del plan**, verá diferentes pestañas, como **Lista del plan** o **Precios y disponibilidad** para configurar los distintos detalles de cada plan. Cada una de estas pestaña mostrará un estado (incompleto o completo) cuando se pasa por los distintos campos.

![Ilustra la página con de la descripción del plan del Centro de partners. Los campos Nombre del plan, Resumen del plan y Descripción del plan están resaltados.](./media/commercial-marketplace-plans/plan-listing-tab.png)

Hay algunos detalles comunes que se deben completar en los planes nuevos:

-  **Identificador del plan**: cree un identificador único para cada uno de los planes de esta oferta. Use un máximo de 50 caracteres: solo minúsculas, caracteres alfanuméricos, guiones y caracteres de subrayado. Los clientes lo verán en la dirección URL del producto y en las plantillas de Azure Resource Manager (si procede). Una vez que se publique la oferta, este identificador no se podrá cambiar.
-  **Nombre del plan**: (llamada 1 de la imagen anterior) cree un nombre único para cada plan de esta oferta. Use un máximo de 50 caracteres. El nombre del plan se usa para diferenciar los planes de software que pueden formar parte de la misma oferta (por ejemplo, Nombre de la oferta: plan Estándar y plan Enterprise). Los clientes verán este nombre al decidir que plan van a seleccionar en su oferta.
-  **Resumen del plan**: (llamada 2 de la imagen anterior) este resumen aparece en los resultados de la búsqueda de Azure Marketplace y puede contener hasta 100 caracteres.
   > [!NOTE]
   > Este campo no se aplica a las ofertas de SaaS.
-  **Descripción del plan**: (llamada 3 de la imagen anterior) agregue una descripción del plan en la que se explique lo que hace que este plan sea diferente de los restantes planes de su oferta. Use un máximo de 500 caracteres. Este contenido se mostrará a los clientes en las páginas de la descripción de la oferta cuando las exploran para seleccionan un plan.

El nombre y la descripción del plan aparecen en la página de la descripción de la oferta en las tiendas en línea del marketplace comercial. En la captura de pantalla siguiente se muestran tres planes de una descripción de la oferta de SaaS en Azure Marketplace.

![Ilustra una página de la descripción de una oferta del Centro de partners. Se muestran tres planes.](./media/commercial-marketplace-plans/offer-listing-page.png)

Una vez que se crean los planes, en la página **Información general del plan** se muestra una lista del nombre, identificador, otros detalles, estado de publicación actual y todas las acciones disponibles en los planes. Estas acciones dependerán del estado del plan y pueden incluir:

-  Si el estado del plan es **Borrador**: eliminar borrador.
-  Si el estado del plan es **Activo**: detener venta del plan o sincronizar público privado.

En la captura de pantalla siguiente se muestran dos borradores de ofertas.

![Ilustra la página Información general del plan del Centro de partners. Se muestran dos planes.](./media/commercial-marketplace-plans/plan-overview-tab.png)

## <a name="pricing-and-availability"></a>Precios y disponibilidad

El marketplace comercial funciona según un modelo de agencia, en el que los editores establecen los precios y Microsoft factura a los clientes y paga los ingresos al editor, a la vez que retiene la cuota correspondiente a la agencia. Los mercados, la visibilidad y los precios de la oferta se definen (cuando proceda) en las pestañas **Precios y disponibilidad** o **Disponibilidad**.

-  **Mercados**: Todos los planes debe estar disponible al menos en un mercado.  Puede seleccionar solo aquellos países con "Impuestos remitidos", en los que Microsoft lleva a cabo las ventas y la retención de impuestos en su nombre.
-  **Precios**: Los modelos de precios solo se aplican a los planes de las ofertas de aplicaciones administradas por Azure, de SaaS y de máquinas virtuales de Azure. Todos los planes de la misma oferta deben usar el mismo modelo de precios.  
-  **Visibilidad del plan**: en función del tipo de oferta, se puede definir una audiencia privada u ocultar la oferta o el plan en Azure Marketplace. Esto se explicará con más detalle en el apartado [Visibilidad del plan](#plan-visibility) de este mismo artículo.

> [!TIP]
> Se recomienda crear planes que se ajusten perfectamente a los patrones de uso de la base de clientes de destino. Esto reduce la posibilidad de que los usuarios cambien de plan con frecuencia, cuando cambia el uso que hacen de los planes. Para ver un ejemplo de una oferta de SaaS con tres planes de facturación medidos, consulte el apartado [Oferta de ejemplo](./partner-center-portal/saas-metered-billing.md#sample-offer).

### <a name="pricing-models"></a>Modelos de precios

En los siguientes tipos de ofertas es preciso asociar un modelo de precios a cada plan. Cada uno de estos tipos de oferta tiene diferentes modelos de precios disponibles:

-  **Aplicación administrada por Azure**: tarifa plana (mensual) y precios basados en el uso (dimensiones de servicio de medición).
-  **Software como servicio**: tarifa plana (mensual o anual), por usuario y precios basados en el uso (dimensiones de servicio de medición). 
-  **Máquina virtual de Azure**: traiga su propia licencia (BYOL) y precios basados en el uso. En el caso del modelo de precios basado en el uso, se puede cobrar por núcleo, por tamaño de núcleo o por mercado y tamaño de núcleo. El modelo de licencia BYOL no permite cargos adicionales basados en el uso   (las ofertas de máquinas virtuales BYOL no requieren un modelo de precios).

Todos los planes de la misma oferta deben usar el mismo modelo de precios. Por ejemplo, una oferta SaaS no puede tener un plan de tarifa plana y otro plan por usuario. Para más información, consulte la documentación de la oferta específica.

Si ya se ha establecido los precios del plan en dólares de Estados Unidos (USD) y agrega otra ubicación del mercado, el precio del nuevo mercado se calculará en función de los tipos de cambio actuales. Tras guardar los cambios, verá el vínculo **Export prices (xlsx)** (Exportar precios) que puede usar para examinar y cambiar el precio de cada mercado antes de la publicación.

> [!IMPORTANT]
> Una vez que la oferta se publica, no se puede cambiar. el modelo de precios elegido.

Tanto las ofertas de SaaS de como las aplicaciones administradas de tarifa plana admiten la facturación medida mediante el servicio de medición de Marketplace. Este es un modelo de facturación basado en uso que le permite definir unidades no estándar, como el ancho de banda o los correos electrónicos, que los clientes pagarán en función de su consumo. Para más información acerca de la facturación medida en las [aplicaciones administradas](./partner-center-portal/azure-app-metered-billing.md) y en las [aplicaciones SaaS](./partner-center-portal/saas-metered-billing.md), consulte la documentación relacionada.

## <a name="custom-prices"></a>Precios personalizado

Para establecer precios personalizados en un mercado individual, exporte, modifique e importe la hoja de cálculo de precios. Usted es responsable de validar estos precios y de poseer estos valores.

Revise los precios cuidadosamente antes de la publicación, ya que hay algunas restricciones en lo que se puede cambiar una vez publicado un plan:

-  Una vez que un plan se publica, el modelo de precios no se puede cambiar.
-  Una vez que se publica el periodo de facturación de un plan, no se puede eliminar.
-  Después de que se publica un precio para un mercado en el plan, no se puede cambiar más adelante.

Los precios establecidos en dólares estadounidenses se convierten en la moneda local de todos los mercados seleccionados y usa los tipos de cambio actuales en el momento de guardarlos. Valide estos precios antes de realizar la publicación. Para ello, debe exportar la hoja de cálculo de precios y examinar el precio de cada mercado que haya seleccionado.

1.  Primero debe guardar los cambios de precios para habilitar la exportación de los datos de precios. Cerca de la parte inferior de la pestaña **Precios y disponibilidad**, seleccione **Guardar borrador**.
1.  En **Precio**, seleccione el vínculo **Export pricing data** (Exportar datos de precios).
1.  Abra el archivo exportedPrice.xlsx en Microsoft Excel.
1.  En la hoja de cálculo, realice las actualizaciones que desee en la información de los precios y, después, guarde el archivo .csv.
    > [!NOTE]
    > Para poder actualizar el archivo, es posible que tenga que habilitar la edición en Excel.
1.  En la pestaña **Precios y disponibilidad**, en **Precio**, seleccione el vínculo **Import pricing data** (Importar datos de precios).
1.  En el cuadro de diálogo que aparece, seleccione **Yes** (Sí).
1.  Seleccione el archivo exportedPrice.xlsx que ha actualizado y, después, seleccione **Abrir**.
















### <a name="plan-visibility"></a>Visibilidad del plan

Puede crear planes para algunos tipos de ofertas que estén disponibles públicamente o disponibles solo para una audiencia específica (privada). Las ofertas con planes privados se publicarán en Azure Portal. Más información sobre los planes privados en [Ofertas privadas en el marketplace comercial de Microsoft](private-offers.md).

Para definir una audiencia privada se usan identificadores de inquilino de Azure o identificadores de suscripción de Azure, en función del tipo de oferta. Puede escribir hasta 10 identificadores manualmente o importar hasta 10 000 identificadores de suscripción o 20 000 identificadores de inquilino (según corresponda) con un archivo. csv. También puede definir audiencias privadas para servicios de consultoría y ofertas de Dynamics 365 que no tengan planes.

Una vez que su oferta se publique con un plan privado, es posible actualizar el público o elegir que el plan esté disponible para todos los usuarios. Una vez que un plan se publica como visible para todos los usuarios, debe permanecer visible para todos y no se puede volver a publicarse como un plan privado.

> [!NOTE]
> Las audiencias privadas difieren de las audiencias en versión preliminar. En la página  **Disponibilidad** de algunos tipos de oferta, se puede definir una audiencia que pueda obtener una vista previa de la oferta antes de que la oferta se publique en directo en el marketplace comercial. Aunque la designación de una audiencia privada solo se aplica a un plan concreto, la audiencia de la versión preliminar puede ver todos los planes (privados o no), pero solo durante el período de versión preliminar, mientras el plan se prueba y valida.

También puede ocultar el plan completamente desde Azure Marketplace si la oferta se implementará como parte de una aplicación administrada o una plantilla de solución.

## <a name="free-trials"></a>Pruebas gratuitas

En los planes de las ofertas de SaaS y de máquinas virtuales de Azure comercializables se puede habilitar una evaluación gratuita.

> [!NOTE]
> En esta sección se explican las evaluaciones gratuitas de los planes de pago de las ofertas que se venden a través de Microsoft. Esto no es lo mismo que la descripción de la evaluación gratuita de los asociados que eligen procesar sus transacciones de manera independiente. Puede crear una descripción de la evaluación gratuita (que no esté asociada con un plan) para los siguientes tipos de ofertas:
> -  Máquina virtual de Azure 
> -  SaaS 
> -  Dynamics 365 Business Central
> -  Dynamics 365 for Customer Engagement y Power Apps
> -  Dynamics 365 for Operations
> 
> Para más información sobre las opciones de la descripción, consulte [Determinación de la opción de publicación](determine-your-listing-type.md).

Las evaluaciones gratuitas se admiten para todos los modelos de facturación, excepto los planes medidos. Los planes de SaaS permiten evaluaciones gratuita de un mes. Los planes de máquinas virtuales de Azure permiten evaluaciones gratuitas de 1, 3 o 6 meses.

Cuando un cliente selecciona una evaluación gratuita, recopilamos su información de facturación, pero no empezamos a facturar al cliente hasta que la evaluación se convierte en una suscripción de pago. Las evaluaciones gratuitas se convierten automáticamente en una suscripción de pago al finalizar la evaluación, a no ser que el cliente cancele la suscripción antes de que finalice el periodo de evaluación.

Durante el período de prueba, los clientes pueden evaluar cualquiera de los planes admitidos de la misma oferta que tenga una evaluación gratuita habilitada. Si cambian a otra evaluación diferente dentro de la misma oferta, el período de evaluación no se reinicia. Por ejemplo, si un cliente usa una evaluación gratuita durante 15 días y, después, cambia a otra que se encuentre en la misma oferta, el nuevo período de evaluación contabilizará los 15 días utilizados. La evaluación gratuita que el cliente esté evaluando cuando el período de evaluación finalice es el que se convierte automáticamente en una suscripción de pago.

Una vez que un cliente selecciona una evaluación gratuita de un plan, no puede convertirla en una suscripción de pago en ese plan hasta que finalice el período de evaluación. Si un cliente decide realizar la conversión a otro plan de la misma oferta que no tiene una evaluación gratuita, la conversión tendrá lugar, pero la evaluación gratuita finalizará inmediatamente y se perderán los datos.

> [!NOTE]
> Una vez que un cliente empieza a pagar un plan, no puede obtener una evaluación gratuita de la misma oferta, aunque cambie a otro plan que admita evaluaciones gratuitas.

Para obtener información sobre las suscripciones de clientes que actualmente participan en una evaluación gratuita, use la nueva propiedad de API `isFreeTrial`, que se marcará como true o false. Para más información, vea la [API para obtener suscripción de SaaS](./partner-center-portal/pc-saas-fulfillment-api-v2.md#get-subscription).

## <a name="next-steps"></a>Pasos siguientes

-  Para aprender a agregar o actualizar planes en una oferta existente, consulte [Actualización de una oferta existente en el marketplace comercial](./partner-center-portal/update-existing-offer.md).
-  Para más información sobre las opciones de transacción y los modelos de precios asociados, consulte [Funcionalidades de transacción de marketplace comercial](./marketplace-commercial-transaction-capabilities-and-considerations.md).
