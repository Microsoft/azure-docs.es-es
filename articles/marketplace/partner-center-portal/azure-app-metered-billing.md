---
title: Facturación según uso para aplicaciones administradas con el servicio de medición de Marketplace | Azure Marketplace
description: Esta documentación es una guía para proveedores de software independientes que publican aplicaciones de Azure con modelos de facturación flexibles.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 04/22/2020
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: 07779769dae4c43148bbf4a8205feceea85ed363
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320058"
---
# <a name="managed-application-metered-billing"></a>Facturación según uso de aplicaciones administradas 

Con el servicio de medición de Marketplace, puede crear planes de aplicación administrada para ofertas de aplicaciones de Azure que se cobran según unidades no estándar. Antes de publicar esta oferta, debe definir las dimensiones de facturación, como el ancho de banda, los vales o los correos electrónicos procesados. Después, los clientes pagan según su consumo en estas dimensiones.  El sistema informará a Microsoft por medio de la API de servicio de medición de Marketplace sobre los eventos facturables a medida que se produzcan.

## <a name="prerequisites-for-metered-billing"></a>Requisitos previos para la facturación según uso

Para que un plan de aplicación administrada use la facturación según uso, debe:

* Cumplir todos los requisitos de la oferta, según se describen en [Creación de una oferta de aplicación de Azure](create-new-azure-apps-offer.md).
* Configurar **precios** para cobrar a los clientes el costo por mes del servicio. El precio puede ser cero si no desea cobrar una tarifa fija y prefiere depender exclusivamente de la facturación según uso.
* Establecer **dimensiones de facturación** para los eventos de medición que el cliente pagará a partir de la tarifa plana.
* Integrarse con las [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md) para informar a Microsoft de eventos facturables.

## <a name="how-metered-billing-fits-in-with-pricing"></a>Cómo encaja la facturación según uso con los precios

A la hora de definir la oferta junto con sus modelos de precios, es importante comprender la jerarquía de las ofertas.

* Cada oferta de aplicación de Azure puede tener planes de plantilla de solución o planes de aplicación administrada.
* La facturación según uso solo se implementa con los planes de aplicación administrada.
* Cada plan de aplicación administrada tiene un modelo de precios asociado. 
* El modelo de precios tiene una tarifa periódica mensual, que se puede establecer en 0 USD.
* Además de la tarifa periódica, el plan también puede incluir dimensiones opcionales que se usan para cobrar a los clientes por el uso no incluido en la tarifa plana. Cada dimensión representa una unidad facturable que el servicio comunicará a Microsoft mediante la [API del servicio de medición de Marketplace](marketplace-metering-service-apis.md).

## <a name="sample-offer"></a>Oferta de ejemplo

Por ejemplo, Contoso es un editor con un servicio de aplicaciones administradas denominado Contoso Analytics (CoA). CoA permite que los clientes analicen una gran cantidad de datos para la creación de informes y el almacenamiento de datos. Contoso está registrado como editor en el centro de asociados para que el programa comercial de Marketplace publique ofertas para clientes de Azure. Hay dos planes asociados a CoA, descritos a continuación:

* Plan base
    * Analizar 100 GB y generar 100 informes cuesta 0 USD/mes.
    * Más allá de 100 GB, se pagan 10 USD por cada GB.
    * Más allá de 100 informes, se paga 1 USD por cada informe.
* Plan Premium
    * Analizar 1000 GB y generar 1000 informes cuesta 350 USD/mes.
    * Más allá de 1000 GB, se paga 100 USD por cada TB.
    * Más allá de 1000 informes, se paga 0,5 USD por cada informe.

Un cliente de Azure que se suscribe al servicio de CoA puede analizar y generar informes al mes en función del plan seleccionado. Contoso mide el uso hasta la cantidad incluida sin enviar eventos de uso a Microsoft. Cuando los clientes consumen más de la cantidad incluida, no tienen que cambiar de plan ni hacer nada diferente. Contoso medirá el exceso sobre la cantidad incluida y comenzará a emitir eventos de uso a Microsoft para determinar el uso adicional mediante la [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md). Microsoft, a su vez, cobrará al cliente por el uso adicional, según ha especificado por el editor.

## <a name="billing-dimensions"></a>Dimensiones de facturación

Las dimensiones de facturación se utilizan para comunicar al cliente cómo se le facturará el uso del software.  Estas dimensiones también se usan para comunicar eventos de uso a Microsoft. Estas dimensiones se definen de la manera siguiente:

* **Identificador de dimensión**: el identificador inmutable al que se hace referencia al emitir eventos de uso.
* **Nombre de dimensión**: nombre para mostrar asociado a la dimensión; por ejemplo, "mensajes de texto enviados".
* **Unidad de medida**: descripción de la unidad de facturación; por ejemplo, "por mensaje de texto" o "por cada 100 correos electrónicos".
* **Precio por unidad**: el precio de una unidad de la dimensión.
* **Cantidad incluida para el período mensual**: la cantidad de dimensión incluida al mes para los clientes que pagan la cuota mensual periódica; debe ser un número entero.

Las dimensiones de facturación se comparten entre todos los planes de una oferta. Algunos atributos son aplicables a la dimensión en todos los planes y otros son específicos de un plan.

Los atributos, que definen la propia dimensión, se comparten entre todos los planes de una oferta. Antes de publicar la oferta, un cambio realizado en estos atributos desde el contexto de cualquier plan afectará a la definición de la dimensión en todos los planes. Una vez publicada la oferta, estos atributos ya no se podrán editar. Los atributos son los siguientes:

* Identificador
* Nombre
* Unidad de medida

Los demás atributos de una dimensión son específicos de cada plan y pueden tener valores diferentes de un plan a otro.  Antes de publicar el plan, puede editar estos valores y solo se verá afectado este plan. Una vez publicado el plan, estos atributos ya no se podrán editar. Los atributos son los siguientes:

* Precio unitario
* Cantidad incluida para clientes mensuales 
* Cantidad incluida para clientes anuales 

Las dimensiones también tienen dos conceptos especiales, "habilitado" e "infinito":

* **Habilitado** indica que este plan participa en esta dimensión.  Es posible que desee dejar esta opción desactivada si va a crear un nuevo plan que no enviará eventos de uso basados en esta dimensión. Además, las nuevas dimensiones que se agregan después de publicar un plan por primera vez se muestran como "no habilitadas" en el plan ya publicado.  Ahora se mostrará una dimensión deshabilitada en todas las listas de dimensiones de un plan visualizado por los clientes.
* **Infinito**, representado por el símbolo de infinito "∞", indica que este plan participa en esta dimensión, sin medir el uso respecto a ella. Permite indicar a los clientes que la funcionalidad representada por esta dimensión está incluida en el plan, sin límite de uso.  Una dimensión con un uso infinito se mostrará en las listas de dimensiones de un plan visualizado por los clientes.  Este plan nunca incurrirá en un cargo.

>[!Note] 
>Los siguientes escenarios se admiten explícitamente:  <br> - Puede agregar una nueva dimensión a un nuevo plan.  La nueva dimensión no se habilitará para los planes ya publicados. <br> - Puede publicar un plan con una tarifa mensual fija sin dimensiones y, a continuación, agregar un nuevo plan y configurar una nueva dimensión para ese plan. La nueva dimensión no se habilitará para los planes ya publicados.

## <a name="constraints"></a>Restricciones

### <a name="locking-behavior"></a>Comportamiento de bloqueo

Una dimensión que se usa con el servicio de medición de Marketplace representa un reconocimiento del modo en que el cliente pagará por el servicio.  Una vez publicada una oferta, ya no se podrá editar ningún detalle de una dimensión.  Por tanto, es importante que las dimensiones estén totalmente definidas antes de esa publicación.

Una vez que se publica una oferta con una dimensión, ya no se pueden cambiar los detalles de nivel de oferta de esa dimensión:

* Identificador
* Nombre
* Unidad de medida

Una vez que se publica un plan, los detalles de nivel de plan ya no se pueden cambiar:

* Precio unitario
* Cantidad incluida para el período mensual
* Si la dimensión está habilitada para el plan

>[!Note]
>Todavía no se admite la facturación según uso con el servicio de medición de Marketplace en la nube de Azure Government.

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
