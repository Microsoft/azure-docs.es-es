---
title: Incorporación de una audiencia preliminar a la oferta de SaaS en el marketplace comercial de Microsoft
description: Incorporación de una audiencia preliminar a la oferta de software como servicio (SaaS) en el Centro de partners de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: ed0c7ef98e70774350c9a3ff12b0cc3f4186e1bb
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380533"
---
# <a name="how-to-add-a-preview-audience-for-your-saas-offer"></a>Incorporación de una audiencia preliminar a la oferta de SaaS

Cuando se crea una oferta de software como servicio (SaaS) en el Centro de partners, es necesario definir una audiencia preliminar que pueda revisar la descripción de la oferta antes de publicarla. En este artículo se explica cómo configurar una audiencia preliminar.

> [!NOTE]
> Si elige procesar las transacciones de forma independiente, no verá esta opción. En lugar de ello, vaya a [Comercialización de una oferta de SaaS](create-new-saas-offer-marketing.md).

## <a name="define-a-preview-audience"></a>Definición de una audiencia preliminar

En la pestaña **Vista previa de la audiencia**, puede definir una audiencia limitada que revise la oferta de SaaS antes de publicarla para la audiencia más amplia del marketplace. Puede enviar invitaciones a direcciones de correo electrónico de cuentas de Microsoft (MSA) o de Azure Active Directory (Azure AD). Agregue un mínimo de 1 y un máximo de 10 direcciones de correo electrónico manualmente o importe hasta 20 con un archivo .csv. La lista de audiencias preliminares se puede actualizar en cualquier momento.

> [!NOTE]
> Un público preliminar no es el mismo que un público privado. A una audiencia preliminar se le permite el acceso a la oferta antes de que se publique en las tiendas en línea. También puede elegir crear un plan y ponerlo a disposición únicamente de un público privado. Los planes privados se explican en [Creación de planes para la oferta de SaaS](create-new-saas-offer-plans.md).

### <a name="add-email-addresses-manually"></a>Agregar direcciones de correo electrónico manualmente

1. En la página **Audiencia preliminar**, agregue una sola dirección de correo electrónico de Azure AD o MSA y una descripción opcional en los cuadros proporcionados.
1. Para agregar otra dirección de correo electrónico, seleccione el vínculo **Add another email** (Agregar otro correo electrónico).
1. Seleccione **Guardar borrador** antes de continuar con la siguiente pestaña: **Configuración técnica**.
1. Continúe con [Incorporación de detalles técnicos a la oferta de SaaS](create-new-saas-offer-technical.md).

### <a name="add-email-addresses-using-the-csv-file"></a>Agregar direcciones de correo electrónico mediante el archivo CSV

1. En la página **Audiencia preliminar**, seleccione el vínculo **Exportar público (CSV)** .
1. Abra el archivo CSV en una aplicación, como Microsoft Excel.
1. En el archivo CSV, en la columna **Id.** , escriba las direcciones de correo electrónico que quiere agregar a la audiencia preliminar.
1. En la columna **Descripción**, tiene la opción de agregar una descripción para cada dirección de correo electrónico.
1. En la columna **Tipo**, agregue **MixedAadMsaId** a cada fila que tenga una dirección de correo electrónico.
1. Guarde el archivo como un archivo CSV.
1. En la página **Audiencia preliminar**, seleccione el vínculo **Import Audience (csv)** (Importar audiencia [csv]).
1. En el cuadro de diálogo **Confirmar**, seleccione **Sí**.
1. Seleccione el archivo CSV y, después, elija **Abrir**.
1. Seleccione **Guardar borrador** antes de continuar con la siguiente pestaña: **Configuración técnica**.

## <a name="next-steps"></a>Pasos siguientes

- [Incorporación de detalles técnicos a la oferta de SaaS](create-new-saas-offer-technical.md)
