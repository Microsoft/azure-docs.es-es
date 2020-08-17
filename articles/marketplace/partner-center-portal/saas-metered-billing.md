---
title: Facturación según uso con el servicio de medición de Marketplace | Azure Marketplace
description: Esta documentación es una guía para fabricantes de software independientes que publican ofertas de SaaS con modelos de facturación flexibles.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/08/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: d6c46575ed497c5067b3ffc5c745e79f814bc212
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87304384"
---
# <a name="metered-billing-using-the-marketplace-metering-service"></a>Facturación según uso con el servicio de medición de Marketplace

Con el servicio de medición de Marketplace, puede crear ofertas de software como servicio (SaaS) que se cobran por unidades no estándar.  Antes de publicar esta oferta, debe definir las dimensiones de facturación, como el ancho de banda, los vales o los correos electrónicos procesados.  A continuación, los clientes pagan en función del consumo que realicen de estas dimensiones, al tiempo que su sistema informa a Microsoft mediante la API del servicio de medición de Marketplace de los eventos facturables cuando se producen.  

## <a name="prerequisites-for-metered-billing"></a>Requisitos previos para la facturación según uso

Para que una oferta de SaaS utilice la facturación según uso, primero tiene que:

* Cumplir todos los requisitos de [una oferta de venta mediante Microsoft](./create-new-saas-offer.md#sell-through-microsoft), tal y como se describe en [Creación de una oferta SaaS](./create-new-saas-offer.md).
* Integrarse con las [API de suministro de SaaS](./pc-saas-fulfillment-api-v2.md) para que los clientes puedan aprovisionar y conectarse a su oferta.  
* Configurarse para el modelo de precios de **tarifa plana** al cobrar a los clientes por su servicio.  Las dimensiones son una extensión opcional del modelo de precios de tarifa plana. 
* Integrarse con las [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md) para informar a Microsoft de eventos facturables.

Después, la oferta SaaS ya se puede integrar con las [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md) para notificar a Microsoft los eventos facturables.

>[!Note]
>El servicio de medición de Marketplace solo está disponible para el modelo de facturación de tarifa plana y no se aplica al modelo de facturación de usuario.

## <a name="how-metered-billing-fits-in-with-pricing"></a>Cómo encaja la facturación según uso con los precios

Es importante comprender la jerarquía de las ofertas a la hora de definir la oferta junto con sus modelos de precios.

* Cada oferta SaaS está configurada para venderse mediante Microsoft o no.  Una vez publicada una oferta, esta opción no se puede modificar.
* Las ofertas SaaS configuradas para venderse en Microsoft pueden tener uno o más planes.  Un usuario se suscribe a la oferta SaaS, pero esta se adquiere en Microsoft en el contexto de un plan.
* Cada plan tiene un modelo de precios asociado: **tarifa plana** o **de usuario**. Todos los planes de una oferta deben estar asociados al mismo modelo de precios. Por ejemplo, no puede haber una oferta en la que haya planes que tengan el modelo de precios de tarifa plana y otros que tengan el modelo de precios por usuario.
* Dentro de cada plan configurado para un modelo de facturación de tarifa plana, se incluye al menos una cuota periódica (que puede ser de 0 USD):
    * Cuota **mensual** periódica: cuota mensual de tarifa plana que se paga por adelantado con una periodicidad mensual cuando el usuario compra el plan.
    * Cuota **anual** periódica: cuota anual de tarifa plana que se paga por adelantado con una periodicidad anual cuando el usuario compra el plan.
* Además de las tarifas periódicas, un plan de tarifa plana también puede incluir dimensiones opcionales personalizadas que se utilizan para cobrar a los clientes por el uso por encima del límite que no esté incluido en la tarifa plana.  Cada dimensión representa una unidad facturable que el servicio comunicará a Microsoft mediante la [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md).

## <a name="sample-offer"></a>Oferta de ejemplo

Por ejemplo, Contoso es un editor con un servicio SaaS llamado Contoso Notification Services (CNS). CNS permite que sus clientes envíen notificaciones por correo electrónico o por mensaje de texto. Contoso está registrado como editor en el Centro de partners para que el programa comercial de Marketplace publique ofertas de SaaS para clientes de Azure.  Hay dos planes asociados a CNS, descritos a continuación:

* Plan Básico
    * Envío de 10 000 correos electrónicos y 1000 mensajes de texto por 0 USD al mes (tarifa plana mensual).
    * Por encima de 10 000 correos electrónicos, pague 1 USD por cada 100 correos electrónicos
    * Por encima de 1000 mensajes de texto, pague 0,02 USD por cada mensaje de texto

    [![Precios del plan básico](./media/saas-basic-pricing.png "Haga clic para una vista ampliada")](./media/saas-basic-pricing.png)

* Plan Premium
    * Envío de 50 000 correos electrónicos y 10 000 mensajes de texto por 350 USD al mes o 5 millones de mensajes de correo electrónico y 1 millón de mensajes de texto por 3500 USD al año.
    * Por encima de 50 000 correos electrónicos, pague 0,5 USD por cada 100 correos electrónicos
    * Por encima de 10 000 mensajes de texto, pague 0,01 USD por cada mensaje de texto

    [![Precios del plan Premium](./media/saas-premium-pricing.png "Haga clic para una vista ampliada")](./media/saas-premium-pricing.png)

* Plan Enterprise
    * Envío de un número ilimitado de mensajes de correo electrónico y 50 000 mensajes de texto por 400 USD al mes.
    * Por encima de los 50 000 mensajes de texto, pague 0,005 USD por cada mensaje de texto.

    [![Precios del plan Enterprise](./media/saas-enterprise-pricing.png "Haga clic para una vista ampliada")](./media/saas-enterprise-pricing.png)

En función del plan seleccionado, un cliente de Azure que compre una suscripción a una oferta de SaaS de CNS podrá enviar la cantidad de mensajes de texto y correos electrónicos incluida en el período de suscripción (mes o año, tal y como aparece en los detalles de la suscripción: startDate y endDate).  Contoso mide el uso hasta la cantidad incluida en base sin enviar eventos de uso a Microsoft. Cuando los clientes consumen más de la cantidad incluida, no tienen que cambiar de plan ni hacer nada diferente.  Contoso medirá el uso por encima del límite de la cantidad incluida y comenzará a emitir eventos de uso a Microsoft para cobrar el uso adicional mediante la [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md).  Microsoft, a su vez, cobrará al cliente el uso por encima del límite que especifique el editor en las dimensiones personalizadas. La facturación del uso por encima del límite se realiza en el siguiente ciclo de facturación (mensualmente, pero puede ser trimestral o antes para algunos clientes).  En el caso de un plan de tarifa plana mensual, se realizará la facturación por uso por encima del límite para cada mes en el que se haya producido dicho exceso de uso.  En el caso de un plan de tarifa plana anual, una vez que se consume la cantidad incluida en base por año, todo el uso adicional emitido por el medidor personalizado se facturará como uso por encima del límite durante cada ciclo de facturación (mensualmente) hasta el final del plazo anual de la suscripción.

## <a name="billing-dimensions"></a>Dimensiones de facturación

Cada dimensión de facturación define una unidad personalizada por la que el ISV puede emitir eventos de uso.  Las dimensiones de facturación se utilizan también para comunicar al cliente cómo se le facturará el uso del software.  Estas dimensiones se definen de la manera siguiente:

* **Identificador**: el identificador de dimensión inmutable al que se hace referencia al emitir eventos de uso.
* **Nombre para mostrar**: el nombre para mostrar asociado a la dimensión; por ejemplo, "mensajes de texto enviados".
* **Unidad de medida**: descripción de la unidad de facturación; por ejemplo, "por mensaje de texto" o "por cada 100 correos electrónicos".
* **Precio por unidad en USD**: el precio de una unidad de la dimensión.  Puede ser 0. 
* **Cantidad mensual incluida en base**: la cantidad de dimensión incluida por mes para los clientes que pagan la cuota mensual periódica; tiene que ser un número entero. Puede ser 0 o ilimitado.
* **Cantidad anual incluida en base**: la cantidad de dimensión incluida por cada año para los clientes que pagan la cuota anual periódica; tiene que ser un número entero. Puede ser 0 o ilimitado.

Las dimensiones de facturación se comparten entre todos los planes de una oferta.  Algunos atributos son aplicables a la dimensión en todos los planes y otros son específicos de un plan.

Los atributos, que definen la propia dimensión, se comparten entre todos los planes de una oferta.  Antes de publicar la oferta, un cambio realizado en estos atributos desde el contexto de cualquier plan afectará a la definición de la dimensión en todos los planes.  Una vez publicada la oferta, estos atributos ya no se podrán editar.  Estos atributos son los siguientes:

* ID
* Display Name (Nombre para mostrar)
* Unidad de medida

Los demás atributos de una dimensión son específicos de cada plan y pueden tener valores diferentes de un plan a otro.  Antes de publicar el plan, puede editar estos valores y solo se verá afectado este plan.  Una vez publicado el plan, estos atributos ya no se podrán editar.  Estos atributos son los siguientes:

* Precio por unidad en USD
* Cantidad mensual incluida en base  
* Cantidad anual incluida en base  

Las dimensiones también tienen dos conceptos especiales, "habilitado" e "infinito":

* **Habilitado** indica que este plan participa en esta dimensión.  Si va a crear un nuevo plan que no envía eventos de uso basados en esta dimensión, es posible que desee dejar esta opción desactivada.  Además, las nuevas dimensiones que se agregan después de publicar un plan por primera vez se muestran como "no habilitadas" en el plan ya publicado.  Ahora se mostrará una dimensión deshabilitada en todas las listas de dimensiones de un plan visualizado por los clientes.
* **Infinito**, representado por el símbolo de infinito "∞", indica que este plan participa en esta dimensión, pero que no emite ningún uso de esta dimensión.  Permite indicar a los clientes que la funcionalidad representada por esta dimensión está incluida en el plan, sin límite de uso.  Una dimensión con uso infinito se mostrará en las listas de dimensiones de un plan visto por los clientes con una indicación de que nunca incurrirá en un cargo por este plan.

>[!Note] 
>Los siguientes escenarios se admiten explícitamente: <br> - Puede agregar una nueva dimensión a un nuevo plan.  La nueva dimensión no se habilitará para los planes ya publicados. <br> - Puede publicar un plan de **tarifa plana** sin dimensiones y, a continuación, agregar un nuevo plan y configurar una nueva dimensión para ese plan. La nueva dimensión no se habilitará para los planes ya publicados.

### <a name="setting-dimension-price-per-unit-per-supported-market"></a>Configuración del precio de dimensión por unidad por mercado compatible

Al igual que los precios de tarifa plana, los precios de dimensión de facturación se pueden establecer por país o región admitidos. El editor tiene que usar la característica de importación y exportación de datos de precios del Centro de partners.

1. En primer lugar, defina las dimensiones deseadas y marque qué mercados se admiten. 
1. A continuación, exporte estos datos a un archivo.
1. Agregue los precios correctos por país o región e importe el archivo en el Centro de partners.

La interfaz de usuario del medidor cambiará para reflejar que los precios de la dimensión solo pueden verse en el archivo.

[![Dimensiones del servicio de medición de Marketplace](./media/metering-service-dimentions.png "Haga clic para una vista ampliada")](./media/metering-service-dimentions.png)


### <a name="private-plan"></a>Plan privado

Al igual que los planes de tarifa plana, un plan con dimensiones se puede establecer como plan privado, accesible únicamente para la audiencia definida por el plan.

## <a name="constraints"></a>Restricciones

### <a name="trial-behavior"></a>Comportamiento de prueba

La facturación según uso con el servicio de medición de Marketplace no es compatible con una oferta de evaluación gratuita.  No es posible configurar un plan para usar la facturación según uso y una evaluación gratuita.

### <a name="locking-behavior"></a>Comportamiento de bloqueo

Dado que una dimensión que se usa con el servicio de medición de Marketplace representa un reconocimiento del modo en que el cliente pagará por el servicio, los detalles de una dimensión no se podrán editar una vez publicada.  Es importante que las dimensiones estén totalmente definidas para un plan antes de la publicación.

Una vez que se publica una oferta con una dimensión, ya no se pueden cambiar los detalles de nivel de oferta de esa dimensión:

* ID
* Display Name (Nombre para mostrar)
* Unidad de medida

Una vez que se publica un plan, los detalles de nivel de plan ya no se pueden cambiar:

* Precio por unidad en USD
* Cantidad mensual incluida en base
* Cantidad anual incluida en base
* Si la dimensión está habilitada para el plan o no

### <a name="upper-limits"></a>Límites superiores

El máximo de dimensiones que se pueden configurar para una única oferta es de 18 dimensiones únicas.

## <a name="get-support"></a>Obtención de soporte técnico

Puede abrir una incidencia de soporte técnico si tiene cualquiera de los problemas siguientes:

* Incidencias técnicas de la API del servicio de medición de Marketplace.
* Una incidencia que debe escalarse debido a un error por su parte (por ejemplo, un evento de uso incorrecto).
* Cualquier otra incidencia relacionada con la facturación según uso.

Siga las instrucciones de [Soporte técnico para el programa de Marketplace comercial en el Centro de partners](./support.md) para conocer las opciones de soporte técnico para editores y abrir una incidencia de soporte técnico con Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Marketplace metering service APIs](./marketplace-metering-service-apis.md) (API del servicio de medición de Marketplace) para más información.
