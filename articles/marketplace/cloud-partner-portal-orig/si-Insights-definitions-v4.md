---
title: Definiciones de Seller Insights
description: Proporciona definiciones para muchos de los términos que encontrará en perspectivas de vendedor
services: Azure, Marketplace, Cloud Partner Portal,
author: v-miclar
ms.service: marketplace
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: pabutler
ms.openlocfilehash: 8b3447b8094eb95bd2f61baed7371bf718fdf730
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606911"
---
<a name="seller-insights-definitions"></a>Definiciones de Seller Insights
=======================

En la siguiente tabla se incluyen definiciones de muchos de los términos usados en Seller Insights.

|  **Término**                 |  **Definición**                                                                                                                              |
|  --------------------------------------------    |  ---------------------------------------------------------------------------------------------------------------------------------           |
| **Tipo de licencia de Azure**                               | Tipo de contrato de licencia que los clientes utilizan para comprar Azure.  También conocido como Canal.                                                  |
| **Tipo de licencia de Azure: Proveedor de soluciones en la nube**      | El cliente final adquiere Azure y su oferta de Marketplace a través de su proveedor de soluciones en la nube, que actúa como su revendedor.                 |
| **Tipo de licencia de Azure: Enterprise**                   | El cliente final adquiere Azure y su oferta de Marketplace a través de un contrato Enterprise, firmado directamente con Microsoft.                  |
| **Tipo de licencia de Azure: Enterprise a través de distribuidor**  | El cliente final adquiere Azure y su oferta de Marketplace a través de un distribuidor que facilita su contrato Enterprise con Microsoft.     |
| **Tipo de licencia de Azure: Pago por uso**                | El cliente final adquiere Azure y su oferta de Marketplace a través de un contrato de "Pago por uso", firmado directamente con Microsoft.                |
| **Cargo (CC)**                              | La cantidad que se cobra al cliente, en el *divisa cliente* (CC) para la facturación.                                                                 |
| **Cargo (PC)**                               | La cantidad que se cobra al cliente, en el *moneda de pago* (PC).                                                                      |
| **Fecha del cargo**                                      | La fecha que cargo del cliente calcula (normalmente inmediatamente después del punto de uso).                                             |
| **Nombre de instancia en la nube**                              | Microsoft Cloud donde se produjo la implementación de una VM.                                   |
| **Nombre de instancia en la nube: Azure Global**                | La nube pública de Microsoft global.                           |
| **Nombre de instancia en la nube: Azure Government**        | Nubes de Microsoft específicos de Government para uno de los gobiernos siguientes: China, Alemania o los Estados Unidos de América.                           |
| **Estado de recopilación**  | Para un cargo específico, el estado más reciente del ciclo de vida de la facturación y la colección.  Por ejemplo, el estado podría ser la colección en curso, recopilan, o reembolso.                  |
| **Tamaño de núcleo**  | El número de núcleos virtuales presentadas por la máquina virtual en que se está ejecutando la oferta.               |
| **Ciudad del cliente**  | El nombre de la población proporcionado por el cliente. Esto podría ser diferente de la ciudad en la suscripción de Azure de un cliente.  |
| **Idioma de comunicación de cliente**        | Idioma preferido por el cliente para la comunicación.                                  |
| **Nombre de la empresa del cliente**                  | El nombre de la empresa proporcionado por el cliente. Esto podría ser diferente de la ciudad en la suscripción de Azure de un cliente.                                  |
| **País del cliente**                       | El nombre de país o región proporcionado por el cliente.  Esto podría ser diferente que el país o región en la suscripción de Azure de un cliente.                               |
| **Moneda (CC) del cliente**                 | Moneda preferida del cliente para precios y facturación.                            |
| **Correo electrónico de cliente**                         | La dirección de correo electrónico proporcionada por el cliente final.  Esto podría ser distinta de la dirección de correo electrónico en la suscripción de Azure de un cliente.                   |
| **Nombre del cliente**                              | El nombre proporcionado por el cliente.  Esto podría ser diferente del nombre proporcionado en la suscripción de Azure de un cliente.                    |
| **Apellido del cliente**                               | El último nombre proporcionado por el cliente.  Esto podría ser diferente del nombre proporcionado en la suscripción de Azure de un cliente.                    |
| **Id. de cliente**                               | El identificador único asignado a un cliente.  Un cliente puede tener cero o más suscripciones de Marketplace de Azure.                    |
| **Tipo de pago del cliente**                            | Tipo de instrumento de pago utilizado por el cliente.  Por ejemplo, esto podría ser verificación electrónica, factura o tarjeta.                             |
| **Código Postal del cliente**                             | El código postal proporcionado por el cliente.  Esto podría ser diferente que el código postal proporcionado en la suscripción de Azure de un cliente.                                                                 |
| **Estado de cliente**                                   | El estado (dirección) proporcionado por el cliente.  Esto podría ser distinto del último nombre proporcionado en la suscripción de Azure de un cliente.                                                                 |
| **Fecha de adquisición**                                    | La primera fecha en el cliente compró cualquier oferta que haya publicado.                 |
| **Fecha perdido**                                        | La última fecha el cliente canceló el último de todas las ofertas de adquirido anteriormente.                                                         |
| **Cargo total estimado (CC)**                   | Cargo total estimado de la cantidad de unidades de uso de una SKU determinada (en la moneda del cliente). Este valor puede no ser exacto debido a errores de truncamiento o redondeo.                            |
| **Cargo total estimado (PC)**                   | Se calcula extendidos cargo por la cantidad de unidades de uso para una SKU determinada, en función de conversión de monedas en el uso de la fecha estimada (en la moneda del publicador). Este valor puede no ser exacto debido a errores de truncamiento o redondeo.                  |
| **Pago estimado (PC)**                            | El pago estimado para la cantidad de unidades de uso de una SKU determinada, en función de conversión de monedas en la fecha que se calcula el uso (en la moneda del publicador). Este valor puede no ser exacto debido a errores de truncamiento o redondeo.                        |
| **Precio estimado (PC)**                             | Se calcula el precio estimado para una unidad de uso para una SKU determinada, en función de conversión de monedas en el uso de la fecha (en la moneda del publicador). Este valor puede no ser exacto debido a errores de truncamiento o redondeo.                                             |
| **Es el nuevo cliente**                                  | El valor le permitirá saber si es un nuevo cliente consume o comprar uno o varios de la plataforma Marketplace ofrece por primera vez (o no).  El valor será "Sí" si es el mismo mes del calendario para "Adquirir la fecha".  El valor será "No" si el cliente ha comprado cualquiera de las ofertas antes del mes de calendario notificados.                                                       |
| **Is Preview SKU**                                  | El valor le permitirá saber si han etiquetado la SKU como "preview". El valor será "Sí" si se ha etiquetado la SKU en consecuencia y suscripciones de Azure solo autorizadas por el usuario pueden implementar y usar esta imagen.  El valor será "No" si la SKU no se ha identificado como "preview".                                          |
| **Contacto promocional para utilizarlo**                   | El valor le permitirá saber si el cliente proactivamente optado por contacto promocional de publicadores. En este momento, no presentamos la opción a los clientes, por lo que hemos indicado "No" de forma general. Una vez implementada esta característica, comenzaremos a actualizarla en consecuencia.                                          |
| **Tipo de licencia de Marketplace**                         | Método de facturación de la oferta de Marketplace.                             |
| **Tipo de licencia de Marketplace: Factura a través de Azure**   | Microsoft es su agente para esta oferta de Marketplace y factura a los clientes en su nombre. (Tarjeta de crédito de pago por uso o factura de Enterprise)       |
| **Tipo de licencia de Marketplace: Traiga su propia licencia** | La máquina virtual requiere una clave de licencia proporcionada por el cliente para implementar. Microsoft no factura a los clientes por su uso de esta oferta de Marketplace. |
| **Tipo de licencia de Marketplace: gratis**                   | La oferta de Marketplace está configurada para ser gratis para todos los usuarios. Microsoft no factura a los clientes por su uso de esta oferta de Marketplace.    |
| **Tipo de licencia de Marketplace: Microsoft como revendedor**  | Microsoft es su revendedor para esta oferta de Marketplace.                               |
| **Nombre de la compañía de la suscripción de Marketplace**  | El nombre de la empresa proporcionado por el cliente en su suscripción de Azure.                           |
| **Id. de suscripción de Marketplace**            | El identificador único asociado con la suscripción de Azure, el cliente usó para comprar su oferta de Marketplace.  Antes era el GUID de suscripción de Azure.  En algunos casos, como un pedido de SAAS, no se necesita ninguna suscripción.           |
| **Microsoft Fee (CC)**                               | Tarifa de Microsoft para la transacción en la moneda del cliente.                              |
| **Nombre de la oferta**                                | El nombre de la oferta de Marketplace.                                                             |
| **Tipo de oferta**                                | El tipo de oferta de Marketplace de Microsoft.  Por ejemplo, podría ser una aplicación administrada, las licencias de máquina virtual o contenedor.                           |
| **Fecha de cancelación de pedido**                         | Fecha en que se canceló el pedido de Marketplace.                                                       |
| **Id. de pedido**                                  | El identificador único del pedido del cliente para el servicio de Marketplace.  Ofertas según el uso de máquinas virtuales no están asociadas con un pedido.                 |
| **Fecha de compra de pedido**                       | La fecha en que se creó el pedido de Marketplace.                                                        |
| **Cantidad de pedido**                       | La cantidad de pedidos de la misma SKU asociado a una suscripción de Azure.  Para los pedidos SAAS, que representa el número de puestos asociado con esta instancia de un pedido.                                            |
| **Estado del pedido**                              | Estado de un pedido de Marketplace en el momento en que se actualizaron por última vez los datos.  Por ejemplo, el orden podría ser activo o cancelado.             |
| **Estado del pedido: Active**                           | El cliente ha comprado un pedido y no ha cancelado su orden.   |
| **Estado del pedido: Cancelado**                        | El cliente compró anteriormente un pedido y su orden canceló posteriormente.        |
| **Importe de pago (PC)**                             | El importe de pago, en la moneda de pago preferido (PC).                                       |
| **Moneda de pago (PC)**                           | Moneda utilizada para sus pagos.                                                                |
| **Fecha de pago**                                    | La fecha se envió la solicitud de pago de Microsoft para tu institución financiera elegido.                                                                         |
| **Estado de pago**                                  | El estado de los pagos en el momento en que se actualizaron por última vez los datos.                                   |
| **Estado de pago: En curso**               | La transacción no está lista para el pago. (Para obtener más información, vea el estado de la colección)                             |
| **Estado de pago: Pagado**                          | La transacción se incluyó en un cálculo de pago anterior. Los valores positivos están pagados y los negativos se compensan en el importe total a pagar. |
| **Estado de pago: Próximo pago**                   | La transacción está preparada para el pago y se incluirá en el siguiente cálculo de pago disponible.                      |
| **Precio (CC)**                                   | Precio de una unidad de uso para una SKU determinada (en la moneda del cliente).                                                                  |
| **Correo electrónico del proveedor**                                 | La dirección de correo electrónico del proveedor implicado en la relación entre Microsoft y el cliente final. Si el cliente es una empresa a través de distribuidor, será el distribuidor.  Si está implicado un proveedor de soluciones en la nube (CSP), será el CSP.                                                                 |
| **Nombre del proveedor**                                 | El nombre del proveedor implicado en la relación entre el cliente de Microsoft y de finalización. Si el cliente es una empresa a través de distribuidor, será el distribuidor.  Si está implicado un proveedor de soluciones en la nube (CSP), será el CSP.                                                                 |
| **URI de recurso**                                  | La ruta de acceso donde se implementó el artefacto de la implementación.                                            |
| **SKU**                                           | Nombre de SKU tal como se define durante la publicación. Una oferta puede tener muchas de las SKU, pero solo se puede asociar con una única oferta una SKU.              |
| **Tipo de SKU de facturación**                                 | El método de facturación de la SKU.                                                                                                             |
| **Tipo de SKU de facturación: Traiga su propia licencia**         | La máquina virtual o servicio requiere una clave de licencia proporcionada por el cliente para implementar. Microsoft no factura a los clientes para su uso de estas ofertas de Marketplace.   |
| **Tipo de SKU de facturación: Microsoft As Reseller**          | Microsoft es su revendedor para esta SKU.   |
| **Tipo de SKU de facturación: gratis**                           | La SKU está configurada para ser gratis para todos los usuarios. Microsoft no factura a los clientes para su uso de esta SKU.                           |
| **Tipo de SKU de facturación: Pagado**                           | Microsoft es el agente para esta SKU clientes y les factura en su nombre. (Tarjeta de crédito de pago por uso o factura de Enterprise)                   |
| **Tipo de SKU de facturación: versión de evaluación**                          | El cliente está en su período de prueba y se convertirá en cliente de pago si no cancela o elimina la suscripción.                                         |
| **Importe de impuestos (CC)**                                  | El importe de impuestos que se aplica a la factura del cliente en la moneda de cliente (CC).                                                               |
| **Fecha de transacción**                                 | Fecha de la transacción registrada en los informes de pago.                                                                           |
| **Tipo de transacción**                               | Tipo de transacción del que se está generando el informe.  Por ejemplo, esto podría ser un cargo, reembolso o ajuste de pago.                                                                                        |
| **Fecha de fin de la prueba**                                 | La fecha del período de prueba de este pedido va a finalizar o ha finalizado.                                                                           |
| **Fecha de uso**                                       | Fecha en que se produjo el uso por parte del cliente.                                                                                                             |
| **Cantidad de uso**                                       | El uso de informado los clientes de la SKU. Para las imágenes de máquina virtual, los registros de uso representan el uso para el período notificado para ese SKU y el tamaño de máquina virtual.                                               |
| **Referencia de uso**                                       | El identificador de uno o varios días de uso del cliente para una SKU determinada asociada con una entrada del informe de pagos.                                               |
| **Tipo de uso**                                       | Descripción del uso que se está midiendo. (Uso normalizado o uso sin procesar)                                                              |
| **Tipo de uso: Uso normalizado**                | Horas de uso normalizadas para contabilizarse en el número de núcleos de VM implicados en el uso: [número de núcleos de VM] x [horas de uso sin procesar]. Las máquinas virtuales designadas como "SHAREDCORE" usan 1/6 (o 0,1666) como multiplicador de [número de núcleos de VM].                                        |
| **Tipo de uso: Uso sin procesar**                        | Horas de uso de la oferta de Marketplace.  Se presenta en términos de horas "calendar".  Es el número de máquinas virtuales de tiempo se han ejecutado.                           |
| **Unidades de uso**                                   | Unidad de medida del uso indicado. Las máquinas virtuales siempre se miden con unidades de media horarias.                               |
| **Tamaño de VM**                                       | Representa el tamaño del hardware de máquina virtual alineado con la oferta de Azure. Algunos ejemplos son `Basic_A0`, `Standard_A11`, `Standard_D12` y `Standard_G4`.   |
|  |  |


