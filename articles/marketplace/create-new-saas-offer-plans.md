---
title: Creación de planes para su oferta de SaaS en el Centro de partners de Microsoft
description: Procedimiento para crear planes para una oferta de software como servicio (SaaS) mediante el portal de marketplace comercial de Microsoft en el Centro de partners
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: 63629f32c97611fa8bc57ecdac968552375125f4
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380538"
---
# <a name="how-to-create-plans-for-your-saas-offer"></a>Creación de planes para la oferta de SaaS

Las ofertas vendidas mediante el marketplace comercial de Microsoft deben tener al menos un plan. Puede crear una variedad de planes con diferentes opciones dentro de la misma oferta. Estos planes (a veces denominados SKU) pueden diferir en lo tocante a versión, monetización o niveles de servicio. Para obtener instrucciones detalladas sobre los planes, consulte [Planes y precios de las ofertas del marketplace comercial](plans-pricing.md).

> [!NOTE]
> Si elige procesar las transacciones de forma independiente, no verá esta opción. En lugar de ello, vaya a [Comercialización de una oferta de SaaS](create-new-saas-offer-marketing.md).

## <a name="create-a-plan"></a>Creación de un plan

1. Cerca de la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**.

1. En el cuadro de diálogo que aparece, en el cuadro **Id. de plan**, escriba un identificador de plan único. Use solo hasta 50 caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. No se puede modificar el identificador del plan después de seleccionar **Crear**.

1. En el cuadro **Nombre del plan**, escriba un nombre para este plan. Use un máximo de 50 caracteres.

1. Seleccione **Crear**.

## <a name="define-the-plan-listing"></a>Definición de la lista de planes

En la pestaña **Lista del plan**, puede definir el nombre y la descripción del plan como desee que aparezcan en el marketplace comercial.

1. En el cuadro **Nombre del plan**, se muestra el nombre que proporcionó anteriormente para este plan. Puede cambiarlo en cualquier momento. Este nombre aparecerá en el marketplace comercial como el título del plan de software de la oferta.

1. En el cuadro **Descripción del plan**, explique lo que hace que este plan de software sea único y las diferencias con respecto a otros planes de software de su oferta. Esta descripción puede contener hasta 500 caracteres.
1. Seleccione **Guardar borrador** antes de continuar con la siguiente pestaña: **Precios y disponibilidad**

## <a name="define-markets-pricing-and-availability"></a>Definición de mercados, precios y disponibilidad

Todos los planes debe estar disponible al menos en un mercado. En la pestaña **Precios y disponibilidad**, puede configurar los mercados en los que estará disponible este plan, el modelo de monetización deseado, el precio y el período de facturación. Además, puede indicar si desea que el plan sea visible a todos los usuarios o solo a clientes específicos (también llamado un plan privado).

1. En **Mercados**, seleccione el vínculo **Editar mercados**.
1. En el cuadro de diálogo que aparece, seleccione las ubicaciones de mercado en las que desea que el plan esté disponible. Debe seleccionar un mínimo de uno y un máximo de 141 mercados.

   > [!NOTE]
   > En este cuadro de diálogo se incluyen un cuadro de búsqueda y una opción para filtrar los países con "Impuesto remitido", en los que Microsoft remite de los impuestos de uso y ventas en su nombre.

1. Seleccione **Guardar** para cerrar el cuadro de diálogo.

## <a name="define-a-pricing-model"></a>Definición de un modelo de precios

Debe asociar un modelo de precios a cada plan: _tarifa plana_ o _por usuario_. Todos los planes de la misma oferta deben usar el mismo modelo de precios. Por ejemplo, una oferta no puede tener un plan de tarifa plana y otro plan por usuario. Para más información, consulte [Modelos de precios de SaaS](plan-saas-offer.md#saas-pricing-models).

> [!IMPORTANT]
> Una vez publicada la oferta, no se puede cambiar el modelo de precios. Además, todos los planes de la misma oferta deben compartir el mismo modelo de precios.

### <a name="configure-flat-rate-pricing"></a>Configuración de precios de tarifa plana

1. En la pestaña **Precios y disponibilidad**, en **Precios**, seleccione **Tarifa plana**.
1. Active la casilla **Mensual** o **Anual**, o ambas, y, después, especifique el precio.

### <a name="add-a-custom-meter-dimension"></a>Adición de una dimensión de medidor personalizada

Esta opción solo está disponible si se han seleccionado precios de tarifa plana. Para más información, consulte [Facturación según uso con el servicio de medición de marketplace](./partner-center-portal/saas-metered-billing.md).

1. En **Dimensiones del servicio de medición de Marketplace**, seleccione el vínculo **Agregar una dimensión de medidor personalizado (máx. 30)** .
1. En el cuadro **Id.** , escriba la referencia inmutable del identificador mientras emite eventos de uso.
1. En el cuadro **Nombre para mostrar**, escriba el nombre para mostrar asociado a la dimensión. Por ejemplo, "mensajes de texto enviados".
1. En el cuadro **Unidad de medida**, escriba la descripción de la unidad de facturación. Por ejemplo, "por mensaje de texto" o "por 100 correos electrónicos".
1. En el cuadro **Precio por unidad en USD**, indique el precio de una unidad de la dimensión.
1. En el cuadro **Cantidad mensual incluida en base**, escriba la cantidad (como entero) de la dimensión que se incluye cada mes para los clientes que pagan la cuota mensual recurrente. Para establecer una cantidad ilimitada, active la casilla en su lugar.
1. En el cuadro **Cantidad anual incluida en base**, escriba la cantidad de la dimensión (como un entero) que se incluye cada mes para los clientes que pagan la cuota anual periódica. Para establecer una cantidad ilimitada, active la casilla en su lugar.
1. Para agregar otra dimensión de medidor personalizada, seleccione el vínculo **Agregar otra dimensión** y, a continuación, repita los pasos del 1 al 7.

### <a name="configure-per-user-pricing"></a>Configuración de precios por usuario

1. En la pestaña **Precios y disponibilidad**, en **Precios**, seleccione **Por usuario**.
2. Si procede, en **Límites de usuario**, especifique el número mínimo y máximo de usuarios para este plan.
3. En **Período de facturación**, especifique un precio mensual, un precio anual o ambos.

### <a name="validate-custom-prices"></a>Validación de precios personalizados

Para establecer precios personalizados en un mercado individual, exporte, modifique e importe la hoja de cálculo de precios. El usuario es responsable de validar estos precios y de poseer estos valores. Para obtener información detallada, consulte [Precios personalizados](plans-pricing.md#custom-prices).

1. Primero debe guardar los cambios de precios para habilitar la exportación de los datos de precios. Cerca de la parte inferior de la pestaña **Precios y disponibilidad**, seleccione **Guardar borrador**.
1. En **Precios**, seleccione el vínculo **Export pricing data** (Exportar datos de precios).
1. Abra el archivo exportedPrice.xlsx en Microsoft Excel.
1. En la hoja de cálculo, realice las actualizaciones que desee en la información de los precios y, después, guarde el archivo .CSV.<br> Para poder actualizar el archivo, es posible que tenga que habilitar la edición en Excel.
2. En la pestaña **Precios y disponibilidad**, en **Precios**, seleccione el vínculo **Import pricing data** (Importar datos de precios).
3. En el cuadro de diálogo que aparece, haga clic en **Sí**.
4. Seleccione el archivo exportedPrice.xlsx que ha actualizado y, después, haga clic en **Abrir**.

### <a name="enable-a-free-trial"></a>Habilitación de una evaluación gratuita

Puede configurar una evaluación gratuita para cada plan de su oferta. Seleccione la casilla para permitir una evaluación gratuita de un mes. Esta casilla no está disponible para los planes que usan el servicio de medición del marketplace. Para más información, consulte cómo [obtener una evaluación gratuita](plans-pricing.md#free-trials).

> [!IMPORTANT]
> Después de haber publicado la oferta de transacciones con una evaluación gratuita, no se puede deshabilitar para ese plan. Asegúrese de que esta configuración sea correcta para la primera publicación para evitar tener que volver a crear el plan.

- En **Evaluación gratuita**, active la casilla **Permitir una prueba gratuita de un mes**.

## <a name="choose-who-can-see-your-plan"></a>Selección de quién puede ver el plan

Tiene la opción de configurar cada plan para que lo pueda ver todo el mundo o solo un público concreto. Puede conceder acceso a un plan privado mediante identificadores de inquilino con la opción de incluir una descripción de cada identificador de inquilino asignado. Puede agregar un máximo de 10 identificadores de inquilino de forma manual o hasta 20 000 identificadores de inquilino mediante un archivo .CSV.

> [!NOTE]
> Si publica un plan privado, puede cambiar su visibilidad a público más adelante. Sin embargo, una vez que se publica un plan público, no se puede cambiar su visibilidad a privado.

### <a name="make-your-plan-public"></a>Conversión del plan en público

1. En **Plan visibility** (Visibilidad del plan), seleccione el cuadro **Público**.
1. Seleccione **Guardar borrador** y, a continuación, en la parte superior izquierda de la pestaña, seleccione **Información general del plan** para volver a la pestaña **Información general del plan**.
1. Para crear otro plan para esta oferta, cerca de la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**. A continuación, repita los pasos descritos en la sección [Creación de un plan](#create-a-plan). En caso contrario, vaya a [Visualización de los planes](#view-your-plans).

### <a name="manually-add-tenant-ids-for-a-private-plan"></a>Adición manual de los identificadores de inquilino para un plan privado 

1. En **Plan visibility** (Visibilidad del plan), active la casilla **This is a private plan** (Ese es un plan privado).
1. En el cuadro **Id. de inquilino** que aparece, escriba el identificador del inquilino de Azure AD del público al que desea conceder acceso a este plan privado. Se requiere un mínimo de un identificador de inquilino.
1. (Opcional) Escriba una descripción de este público en el cuadro **Descripción**.
1. Para agregar otro identificador de inquilino, repita los pasos 2 y 3.
1. Cuando haya terminado de agregar los identificadores de inquilino, seleccione **Guardar borrador** y, a continuación, en la parte superior izquierda de la pestaña, seleccione **Información general del plan** para volver a la pestaña **Información general del plan**.
1. Para crear otro plan para esta oferta, cerca de la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**. A continuación, repita los pasos descritos en la sección [Creación de un plan](#create-a-plan). En caso contrario, vaya a [Visualización de los planes](#view-your-plans).

### <a name="use-a-csv-file-for-a-private-plan"></a>Uso de un archivo .CSV para un plan privado

1. En **Plan visibility** (Visibilidad del plan), active la casilla **This is a private plan** (Ese es un plan privado).
2. Seleccione el vínculo **Exportar público (CSV)** .
3. Abra el archivo .CSV y agregue los identificadores de Azure a los que desea conceder acceso a la oferta privada a la columna **Id.**
4. Opcionalmente, escriba una descripción para cada público en la columna **Descripción**.
5. Agregue "TenantID" en la columna **Tipo** para cada fila con un identificador de Azure.
6. Guarde el archivo .CSV.
7. En la pestaña **Precios y disponibilidad**, en **Plan visibility** (Visibilidad del plan), seleccione el vínculo **Importar público (CSV)** .
8. En el cuadro de diálogo que aparece, seleccione **Sí**.
9. Seleccione el archivo .CSV y, después, elija **Abrir**.
10. Seleccione **Guardar borrador** y, a continuación, en la parte superior izquierda de la pestaña, seleccione **Información general del plan** para volver a la pestaña **Información general del plan**.
11. Para crear otro plan para esta oferta, en la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**. A continuación, repita los pasos descritos en la sección [Creación de un plan](#create-a-plan). En caso contrario, si ha terminado de crear planes, vaya a la sección siguiente: **Visualización de los planes**.

## <a name="view-your-plans"></a>Visualización de los planes

Después de crear uno o varios planes, verá el nombre del plan, el identificador del plan, el modelo de precios, la disponibilidad (pública o privada), el estado de publicación actual y las acciones disponibles en la pestaña **Información general del plan**.

Las acciones que están disponibles en la columna **Acción** de la pestaña **Información general del plan** varían en función del estado del plan y pueden incluir lo siguiente:

- Si el estado del plan es **Borrador**, el vínculo de la columna **Acción** indicará **Eliminar borrador**.
- Si el estado del plan está **activo**, el vínculo de la columna **Acción** será **Stop sell plan** (Detener el plan de venta) o **Sync private audience** (Sincronizar público privado). El vínculo **Sync private audience** (Sincronizar público privado) publicará solo los cambios para el público privado, sin publicar ninguna otra actualización que haya podido realizar en la oferta.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [comercializar su oferta de SaaS](create-new-saas-offer-marketing.md) mediante los programas de **venta conjunta con Microsoft** y de **reventa mediante los CSP**.
- [Prueba y publicación de una oferta de SaaS en el marketplace comercial](test-publish-saas-offer.md)
