---
title: Revisión de la factura del Contrato Enterprise de Azure
description: Aprenda a leer y a comprender el uso y la facturación de los contratos Enterprise de Azure.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 5340d6124ae4fdc669a30458b9e58cf4c86d6746
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684667"
---
# <a name="understand-your-azure-enterprise-agreement-bill"></a>Descripción de la factura de Azure Enterprise

Los clientes de Azure con un Contrato Enterprise reciben una factura cuando superan el crédito de la organización o usan servicios que no están incluidos en ese crédito.

El crédito de su organización incluye su compromiso monetario. El compromiso monetario es la cantidad que su organización paga por adelantado por el uso de los servicios de Azure. Para agregar fondos al compromiso monetario de su Contrato Enterprise, póngase en contacto con el administrador de la cuenta Microsoft o el distribuidor.

Este tutorial se aplica solo a los clientes de Azure con un Contrato Enterprise de Azure.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Revisar los cargos facturados
> * Revisar los cargos por uso del servicio por encima del límite
> * Revisar la factura de Marketplace

## <a name="prerequisites"></a>Prerrequisitos

Para revisar y comprobar los gastos en su factura, debe ser un administrador de empresa. Para más información, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](../manage/understand-ea-roles.md). Si no sabe quién es el administrador de la empresa para su organización, [póngase en contacto con el equipo de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="review-invoiced-charges-for-most-customers"></a>Revisión de los cargos facturados para la mayoría de los clientes

Esta sección no se aplica a los clientes de Azure en Australia, Japón ni Singapur.

Recibe una factura de Azure cuando se produce alguno de estos eventos durante el ciclo de facturación:

- **Uso por encima del límite del servicio**: los gastos por uso de la organización superan el saldo de crédito.
- **Gastos facturados por separado**: los servicios que ha usado la organización no están cubiertos por el crédito. Se le facturará por los siguientes servicios a pesar del saldo de crédito:
    - Canonical
    - Citrix XenApp Essentials
    - Citrix XenDesktop
    - Usuario registrado
    - Openlogic
    - Usuario registrado de Remote Access Rights XenApp Essentials
    - Ubuntu Advantage
    - Visual Studio Enterprise (mensual)
    - Visual Studio Enterprise (anual)
    - Visual Studio Professional (mensual)
    - Visual Studio Professional (anual)
- **Cargos de Marketplace**: las compras y el uso de Azure Marketplace no están incluidos en el crédito de la organización. Por lo tanto, los cargos de Marketplace se le facturan a pesar del saldo de crédito. En Enterprise Portal, un administrador de Enterprise puede habilitar y deshabilitar las compras de Marketplace.

La factura mostrará los cargos de uso de Azure con los costos asociados en primer lugar, seguidos de los cargos de Marketplace. Si tiene un saldo de crédito, se aplica al uso de Azure y la factura mostrará dicho uso y el uso de Marketplace sin ningún costo último.

Compare el importe total combinado que se muestra en Enterprise Portal en **Informes** > **Resumen de uso** con la factura de Azure. Los importes que aparecen en **Resumen de uso** no incluyen impuestos.

Inicie sesión en el [portal del Contrato Enterprise de Azure](https://ea.azure.com). Seleccione **Informes**. En la esquina superior derecha de la pestaña, cambie la vista de **M** a **C** y haga que coincida con el período de la factura.  

![Captura de pantalla que muestra la opción M + C en Resumen de uso.](./media/review-enterprise-agreement-bill/ea-portal-usage-sumary-cm-option.png)

El importe combinado del **uso total** y **Azure Marketplace** debe coincidir con el **importe total extendido** en la factura. Para más detalles sobre los cargos, vaya a **Descarga del uso**.  

![Captura de pantalla que muestra la pestaña Descargar uso](./media/review-enterprise-agreement-bill/ea-portal-download-usage.png)

## <a name="review-invoiced-charges-for-other-customers"></a>Revisión de cargos facturados para otros clientes

Esta sección solo se aplica a los clientes de Azure en Australia, Japón o Singapur.

Recibirá una o más facturas de Azure cuando se produzcan los siguientes eventos:

- **Uso por encima del límite del servicio**: los gastos por uso de la organización superan el saldo de crédito.
- **Gastos facturados por separado**: los servicios que ha usado la organización no están cubiertos por el crédito. Se le facturarán los siguientes servicios:
    - Canonical
    - Citrix XenApp Essentials
    - Citrix XenDesktop
    - Usuario registrado
    - Openlogic
    - Usuario registrado de Remote Access Rights XenApp Essentials
    - Ubuntu Advantage
    - Visual Studio Enterprise (mensual)
    - Visual Studio Enterprise (anual)
    - Visual Studio Professional (mensual)
    - Visual Studio Professional (anual)
- **Cargos de Marketplace**: las compras y el uso de Azure Marketplace no están incluidos en el crédito de la organización y se facturan por separado. En Enterprise Portal, un administrador de Enterprise puede habilitar y deshabilitar las compras de Marketplace.

Cuando tenga cargos por usos por encima del límite del servicio y cargos que se facturan por separado durante el período de facturación, recibirá una factura. Esta incluye ambos tipos de cargos. Los gastos de Marketplace siempre se facturan de forma independiente.

## <a name="review-service-overage-charges-for-other-customers"></a>Revisión de los cargos por uso por encima del límite del servicio para otros clientes

Esta sección solo se aplica si está en Australia, Japón o Singapur.

Compare el importe total por uso en Enterprise Portal en **Informes** > **Resumen de uso** con su factura por uso por encima del límite del servicio. La factura por uso por encima del límite del servicio incluye el uso que ha superado el crédito de la organización y los servicios que no están incluidos en ese crédito. Los importes que aparecen en **Resumen de uso** no incluyen impuestos.

Inicie sesión en el [portal del Contrato Enterprise de Azure](https://ea.azure.com) y seleccione **Informes**. En la esquina superior derecha de la pestaña, cambie la vista de **M** a **C** y haga que coincida con el período de la factura.  

![Captura de pantalla que muestra la opción M + C en Resumen de uso.](./media/review-enterprise-agreement-bill/ea-portal-usage-sumary-cm-option.png)

El importe de **Uso total** debe coincidir con el **Importe total extendido** en la factura por un uso por encima del límite del servicio. Para más información sobre los gastos, vaya a **Descargar uso** > **Descargar informe avanzado**. El informe no incluye los impuestos ni los gastos por reservas o de Marketplace.  

![Captura de pantalla que muestra Descargar informe avanzado en la pestaña Descargar uso.](./media/review-enterprise-agreement-bill/ea-portal-download-usage-advanced.png)

La tabla siguiente enumera los términos y las descripciones que se muestran en la factura y en el **Resumen de uso** de Enterprise Portal:

|Término de facturación|Término de Resumen de uso|Descripción|
|---|---|---|
|Importe total extendido|Uso total|El gasto por uso total antes de impuestos durante el período específico anterior a la aplicación del crédito.|
|Uso de compromiso|Uso de compromiso|El crédito que se aplica durante ese período específico.|
|Total venta|Uso por encima del límite total|El gasto por uso por encima del límite total que supera el importe del crédito. Este importe no incluye impuestos.|
|Importe de impuestos|No aplicable|Impuestos que se aplican al importe total de venta durante el período específico.|
|Importe total|No aplicable|El importe a pagar de la factura una vez aplicado el crédito y agregado los impuestos.|

### <a name="review-marketplace-invoice"></a>Revisar la factura de Marketplace

Esta sección solo se aplica si está en Australia, Japón o Singapur.

Compare el total de Azure Marketplace en **Informes** > **Resumen de uso** en Enterprise Portal con la factura de Marketplace. La factura de Marketplace es solo para el uso y las compras de Azure Marketplace. Los importes en el **Resumen de uso** ya incluyen un impuesto determinado por el publicador.

Inicie sesión en el [portal de Enterprise](https://ea.azure.com) y seleccione **Informes**. En la esquina superior derecha de la pestaña, cambie la vista de **M** a **C** y haga que coincida con el período de la factura.  

![Captura de pantalla que muestra la opción M + C en Resumen de uso.](./media/review-enterprise-agreement-bill/ea-portal-usage-sumary-cm-option.png)  

El importe total de **Azure Marketplace** debe coincidir con el importe **Total venta** en su factura de Marketplace. Para más información sobre los gastos basados en el uso, vaya a **Descargar uso**. En **Gastos de Marketplace**, seleccione **Descargar**. El precio de Marketplace incluye un impuesto determinado por el editor. Los clientes no recibirán ninguna factura independiente del editor para el cobro de impuestos por la transacción.

![Captura de pantalla que muestra la opción Descargar en Gastos de Marketplace.](./media/review-enterprise-agreement-bill/ea-portal-download-usage-marketplace.png)

## <a name="view-price-sheet-information"></a>Visualización de la información de la hoja de precios

Los administradores de empresa pueden ver la lista de precios asociada a su inscripción para los servicios de Azure.

Para ver la hoja de precios actual:

1. En Azure Enterprise Portal, seleccione **Informes** y, a continuación, seleccione **Hoja de precios**.
2. Vea la hoja de precios o seleccione **Descargar**.

![Ejemplo que muestra la información de la hoja de precios](./media/review-enterprise-agreement-bill/ea-create-view-price-sheet-information.png)

Para descargar una lista de precios histórica:

1. En Azure Enterprise Portal, seleccione **Informes** y, a continuación, seleccione **Descargar uso**.
2. Descargue la hoja de precios.

![Ejemplo que muestra dónde descargar una hoja de precios anterior](./media/review-enterprise-agreement-bill/create-ea-view-price-sheet-download-historical-price-list.png)

Algunos motivos para las diferencias de precios:

- Los precios pueden haber cambiado entre la inscripción anterior y la nueva inscripción. Los cambios de precio pueden producirse porque el precio es contractual para la inscripción específica desde la fecha de inicio hasta la fecha de finalización de un contrato.
- Cuando se transfiere a una nueva inscripción, los precios cambian con el nuevo contrato. Los precios se definen por la hoja de precios, que podrían ser más altos en la nueva inscripción.
- Si una inscripción entra en un período extendido, los precios también cambian. Los precios cambian a las tarifas de pago por uso.

## <a name="request-detailed-usage-information"></a>Solicitud de información de uso detallada

Los administradores de empresa pueden ver un resumen de sus datos de uso, el compromiso monetario consumido y los cargos asociados con el uso adicional en Azure Enterprise Portal. Los cargos se presentan en el nivel de resumen en todas las cuentas y suscripciones.

Para ver el uso detallado en cuentas específicas, descargue el informe de detalles de uso en **Informes** > **Descargar uso**.

> [!NOTE]
> El informe de detalles de uso no incluye ningún impuesto aplicable. Puede haber una latencia de hasta ocho horas desde el momento en que se incurrió en el uso hasta el momento en que este se ve reflejado en el informe.

En el caso de inscripciones indirectas, el asociado debe habilitar la función de incremento antes de que pueda ver cualquier información relacionada con el costo.

## <a name="reports"></a>Informes

Los administradores de empresa pueden ver un resumen de sus datos de uso, el compromiso monetario consumido y los cargos asociados con el uso adicional en Azure Enterprise Portal. Los cargos se presentan en el nivel de resumen en todas las cuentas y suscripciones.

### <a name="azure-enterprise-reports"></a>Informes de Azure Enterprise

- Resumen de uso y gráficos
- Informe de uso de servicio
- Informe de saldo y cargos
- Informe de detalles de uso
- Informe de cargos de Azure Marketplace
- Hoja de precios
- Descarga de informes avanzada
- Formato de informe CSV

### <a name="to-view-the-usage-summary-reports-and-graphs"></a>Para ver los informes y gráficos del resumen de uso:

1. Vaya a Azure Enterprise Portal.
1. Seleccione **Informes** en el panel izquierdo.
1. Seleccione la pestaña **Resumen de uso**.
1. Seleccione el período de compromiso en el menú de rangos de fechas en la parte superior izquierda.
1. Seleccione un periodo o el mes en el gráfico para ver detalles adicionales.
1. En esta pestaña, puede:
   - Ver un gráfico del uso mes a mes con un desglose del uso, el cargo extra por los servicios, los cargos facturados por separado y cargos de Azure Marketplace.
   - Filtrar por departamentos, cuentas y suscripciones debajo del gráfico.
   - Cambiar entre los desgloses **Charge by Services** (Cargo por servicios) y **Charge by Hierarchy** (Cargo por jerarquía).
   - Ver los detalles de los servicios de Azure, los cargos facturados por separado y los cargos de Azure Marketplace.

## <a name="service-usage-report"></a>Informe de uso de servicio

La página de informe de uso de los servicios permite a los administradores de empresa ver un resumen de sus datos de uso de servicio. El uso se presenta en el nivel de resumen en todas las cuentas y suscripciones. Para ver el uso detallado, puede filtrar el informe por cuentas o suscripciones.

> [!NOTE]
> Puede haber una latencia de hasta cinco días entre la fecha en que se realizó el uso y el momento en que el uso se muestra en este informe.

### <a name="to-view-the-report"></a>Para ver el informe:

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Informes** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Resumen de uso**.
1. Seleccione el rango de fechas.
1. Elija qué cuentas o suscripciones desea ver.
1. Como alternativa, puede:
   - Cambiar la vista entre **Charge by Services** (Cargo por servicios) y **Charge by Hierarchy** (Cargo por jerarquía) para mostrar distintos desgloses.
   - Ver los detalles, incluidos el nombre del servicio, la unidad de medida, las unidades consumidas, la tasa efectiva y el costo ampliado.

## <a name="download-csv-reports"></a>Descarga de informes CSV

La página de descarga de informes mensuales permite a los administradores de empresa descargar varios informes como archivos CSV. Entre los informes que se pueden descargar se incluyen:

- Informe de saldo y cargos
- Informe de detalles de uso
- Informe de cargos de Azure Marketplace
- Hoja de precios

### <a name="to-download-reports"></a>Para descargar informes:

1. En Azure Enterprise Portal, seleccione **Informes**.
1. Seleccione **Usage Download** (Descarga de uso) en la cinta de opciones superior.
1. Seleccione **Download** (Descargar) junto al informe del mes correspondiente.

### <a name="csv-report-formatting-issues"></a>Problemas de formato de los informes CSV

Los clientes que ven los informes CSV de Azure Enterprise Portal en euros pueden tener problemas de formato relacionados con las comas y los puntos.

Por ejemplo, es posible que vea lo siguiente:

| **ServiceResource** | **ResourceQtyConsumed** | **ResourceRate** | **ExtendedCost** |
| --- | --- | --- | --- |
| Horas | 24 | 0,0535960591133005 | 12,863,054,187,192,100,000,000 |

Debería ver lo siguiente:

| ServiceResource | ResourceQtyConsumed | ResourceRate | ExtendedCost |
| --- | --- | --- | --- |
| Horas | 24 | 0,0535960591133005 | 1,2863054187192120000000 |

Este problema de formato se debe a la configuración predeterminada de la funcionalidad de importación de Excel. Excel importa todos los campos como texto "General" y supone que un número está separado con el estándar matemático. Por ejemplo: "1,000.00".

Si una moneda europea use un punto (.) como separador de miles y una coma (,) como separador de decimales, se mostrará incorrectamente. Por ejemplo: "1.000,00". Los resultados de la importación pueden variar en función de la configuración regional del idioma.

### <a name="to-import-the-csv-file-without-formatting-issues"></a>Para importar el archivo CSV sin problemas de formato:

1. En Microsoft Excel, vaya a **Archivo** > **Abrir**.
   Aparecerá el Asistente para importar texto.
1. En **Tipo de los datos originales**, elija **Delimitado**.  El valor predeterminado es **De ancho fijo**.
1. Seleccione **Next** (Siguiente).
1. En Delimitadores, active la casilla de **Coma**. Desactive **Tabulación** si está seleccionada.
1. Seleccione **Next** (Siguiente).
1. Desplácese a las columnas **Tasa de recursos** y **Coste ampliado**.
1. Seleccione la columna **Tasa de recursos**. Aparece resaltada en negro.
1. En la sección **Formato de los datos en columnas**, seleccione **Texto** en lugar de **General**. El encabezado de columna cambiará de **General** a **Texto**.
1. Repita los pasos 8 y 9 en la columna **Coste ampliado** y seleccione **Finalizar**.

> [!TIP]
> Si ha establecido que los archivos CSV se abran automáticamente en Excel, debe usar la función **Abrir** en Excel en su lugar. En Excel, vaya a **Archivo** > **Abrir**.

## <a name="balance-and-charge-report"></a>Informe de saldo y cargos

El informe de saldo y cargos ofrece un resumen mensual de información sobre saldos, nuevas compras, gastos de servicios en Azure Marketplace, ajustes y gastos por de uso por encima del límite.

Todas las filas de la tabla de resumen Compromiso de servicio de Azure siguen siendo estáticas mes a mes. Encontrará detalles de todas las compras y los ajustes en las secciones **New Purchase Details** (Detalles de nuevas compras) y **Adjustment Details** (Detalles de ajustes).

### <a name="download-the-balance-and-charge-report"></a>Descarga del informe de saldo y cargos

1. Inicie sesión en Azure Enterprise Portal como administrador de empresa.
1. Seleccione **Informes** en el panel izquierdo.
1. Seleccione la pestaña **Descargar informe**.
1. Seleccione el mes correspondiente en la columna **Balance and Charge** (Saldo y cargos) y seleccione para descargar el informe.

## <a name="usage-detail-report"></a>Informe de detalles de uso

El informe de detalles de uso ofrece un resumen mensual del consumo de servicios y cantidades de una inscripción. Incluye información sobre los nombres de medidores, los tipos de medidores y las cantidades consumidas.

### <a name="download-the-usage-detail-report"></a>Descarga del informe de detalles de uso.

1. Inicie sesión en Azure Enterprise Portal como administrador de empresa.
1. Seleccione **Informes** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Descargar uso**.
1. Seleccione el mes correspondiente en la columna **Usage Detail** (Detalles de uso) y seleccione para descargar el informe.

## <a name="azure-marketplace-charges-in-azure-enterprise-portal-reports"></a>Cargos de Azure Marketplace en informes de Azure Enterprise Portal

Existen dos tipos de cargos de Azure Marketplace:

- **Basado en uso:** productos medidos en unidades transaccionales.  Por ejemplo, las máquinas virtuales se cobran por hora, mientras que las búsquedas de Bing API se cobran según el número de búsquedas.
- **Precio mensual**: Se factura mensualmente según el uso o el acceso.

Para más información sobre los cargos de Azure Marketplace, consulte las [preguntas más frecuentes sobre Azure Marketplace](https://azure.microsoft.com/marketplace/faq/).

Para ver los distintos cargos en Azure Enterprise Portal:

- **Usage summary report** (Informe de resumen de uso): Muestra **tanto** los cargos de Azure Marketplace de tarifa mensual como los basados en el uso.
- **Marketplace charges report** (Informe de cargos de Marketplace): Muestra **solo** los cargos de Azure Marketplace basados en el uso.  No se muestran las tarifas puntuales.

> [!NOTE]
> Azure Marketplace no está disponible para inscripciones de MPSA.

## <a name="advanced-report-download"></a>Descarga de informes avanzada

Para los informes de cuentas o rangos de fechas específicos, puede usar la descarga de informes avanzados. A partir del 30 de agosto de 2016, el formato del archivo de salida es CSV para alojar conjuntos de registros mayores.

1. En Azure Enterprise Portal, seleccione **Descarga de informes avanzada**.
1. Seleccione **Appropriate Date Range** (Intervalo de fechas adecuado).
1. Seleccione **Appropriate Accounts** (Cuentas adecuadas).
1. Seleccione **Request Usage Data** (Solicitar datos de uso).
1. Seleccione el botón **Actualizar** hasta que el estado del informe se actualice a **Descargar**.
1. Descargue el informe.

## <a name="reporting-for-non-enterprise-administrators"></a>Informes para administradores que no son de empresa

Los administradores de empresa pueden conceder a los administradores de departamento (DA) y propietarios de cuenta (AO) permisos para ver los cargos en una inscripción. Los propietarios de cuenta con acceso pueden descargar los informes CSV específicos de su cuenta y sus suscripciones. También pueden ver la información de forma visual en la sección de informes de Azure Enterprise Portal.

### <a name="to-enable-access"></a>Para habilitar el acceso:

 1. Inicie sesión en Azure Enterprise Portal como administrador de empresa.
 1. Seleccione **Administrar** en el panel de navegación izquierdo.
 1. Seleccione la pestaña **Inscripción**.
 1. En la sección **Enrollment Detail** (Detalles de inscripción), seleccione el icono de lápiz situado junto a **DA View Charges** (El administrador del departamento ve los cargos) o **AO View Charges** (El propietario de la cuenta ve los cargos).
 1. Seleccione **Habilitado**.
 1. Seleccione **Guardar**.

### <a name="to-view-reports"></a>Para ver informes:

1. Inicie sesión en Azure Enterprise Portal como administrador de departamento o como propietario de cuenta.
1. Seleccione **Informes** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Usage Summary** (Resumen de uso) para ver la información de la cuenta y la suscripción visualmente.
1. Seleccione **Usage Download** (Descarga de uso) para ver los informes CSV.

Los administradores de departamento y los propietarios de cuenta no pueden ver los cargos al usar la función **Advanced Report Download** (Descarga de informes avanzada). Los costos se muestran como $0.

Los permisos del propietario de la cuenta para ver los cargos se extienden a los propietarios de cuenta y a todos los usuarios que tienen permisos en las suscripciones asociadas. Si es un cliente indirecto, el partner de canal debe habilitar las características de costos.

## <a name="power-bi-reporting"></a>Informes de Power BI

Los informes de Power BI están disponibles para clientes directos, asociados e indirectos de Contrato Enterprise que tengan acceso a la información de facturación.

### <a name="power-bi-pro"></a>Power BI Pro

Power BI Pro está disponible para los clientes de EA. Con Power BI Pro, puede generar y compartir informes para administrar de forma eficaz datos de costos. También tiene características adicionales de colaboración y de actualización de datos. Power BI Pro ofrece mayor capacidad de datos y límites de streaming de datos.

<!--We plan to add new cost management features for Azure Enterprise customers.

Current Power BI (free) users who use the Microsoft Azure Consumption Insights content pack can get a 60-day free trial of Power BI Pro. After the trial is over, you can continue using Power BI Pro by adding a license.

To sign up for the free Power BI Pro trial:

1. From the gear icon in Power BI, select **Manage personal storage**.
1. Select **Try Pro for free** on the right.

See [Power BI self-service sign up](https://powerbi.microsoft.com/documentation/powerbi-service-self-service-signup-for-power-bi/#power-bi-pro-60-day-trial) for more information on the Power BI Pro free trial.

### Azure EA Power BI Pro trial terms

- **General purpose**: The extended Power BI Pro for the "Microsoft Azure Enterprise" content pack trial offer (the "Offer") is available to existing qualified users during the term of the Offer, to allow them to access certain insights related to their Azure consumption through the use of a specific Power BI content pack.
- **Eligibility**: Users under an Enterprise Agreement (EA) can participate in the Offer if they have a function related to their organization's Azure billing, service, or cost management.
- **Exclusions**:
  - Users already participating in the Extended Power BI Pro trial will continue to qualify under the pre-existing offer and can't enter into the Azure EA Power BI Pro trial offer.
  - Users participating in the Offer can only use Power BI Pro with the Microsoft Azure Enterprise content pack. Any other use of Power BI Pro is prohibited.
  - Term: The Offer began on June 1, 2017 and ended on May 31, 2018.  Acceptance can occur at any time during the 12-month period, though the offer will terminate on May 31, 2018 for all users regardless of when they accepted the Offer.
  -->

### <a name="to-access-microsoft-azure-consumption-insights"></a>Para acceder a Microsoft Azure Consumption Insights:

1. Vaya a [Microsoft Azure Consumption Insights](https://app.powerbi.com/getdata/services/azureconsumption?cpcode=MicrosoftAzureConsumptionInsights&amp;getDataForceConnect=true&amp;WT.mc_id=azurebg_email_Trans_33675_1378_Service_Notice_EA_Customer_Power_BI_EA_Content_Pack_Apr26).
1. Seleccione **Obtenerla ahora**.
1. Proporcione un número de inscripción y el número de meses y, a continuación, seleccione **Siguiente**.
1. Proporcione la clave de acceso de la API para conectarse. Puede encontrar la clave para la inscripción en [Enterprise Portal](https://ea.azure.com/?WT.mc_id=azurebg_email_Trans_33675_1378_Service_Notice_EA_Customer_Power_BI_EA_Content_Pack_Apr26).
1. Seleccione **Iniciar sesión** para iniciar automáticamente el proceso de importación.
1. Cuando haya finalizado, aparecerán un nuevo panel de información, un informe y un modelo en el panel de navegación. Seleccione el panel de información para ver los datos importados.

> [!TIP]
>
> - Para obtener más información sobre cómo generar la clave de API para su inscripción, consulte el archivo de ayuda de API Reports en [Enterprise Portal](https://ea.azure.com/?WT.mc_id=azurebg_email_Trans_33675_1378_Service_Notice_EA_Customer_Power_BI_EA_Content_Pack_Apr26).
> - Para obtener más información sobre cómo conectar Power BI al consumo de Azure, consulte [Microsoft Azure Consumption Insights](/power-bi/desktop-connect-azure-cost-management).

### <a name="to-access-the-legacy-power-bi-ea-content-pack"></a>Para acceder al paquete de contenido de EA de Power BI heredado:

1. Vaya al [sitio web de Power BI](https://app.powerbi.com/getdata/services/azure-enterprise).
1. Inicie sesión con una cuenta profesional o educativa válida.

   La cuenta profesional o educativa puede ser la misma o distinta de la que usa para tener acceso a la inscripción mediante Azure Enterprise Portal.
1. En el panel de servicios, seleccione **Microsoft Azure Enterprise** y, luego, **Conectar**.
1. En la pantalla **Conectar con Azure Enterprise**, rellene los campos:
    - Dirección URL del entorno de Azure: [https://ea.azure.com](https://ea.azure.com/)
    - Número de meses: de 1 a 36
    - Número de inscripción: el número de inscripción
1. Seleccione **Next** (Siguiente).
1. En el cuadro **Clave de autenticación**, escriba la clave de API.

    Puede obtener la clave de API en Azure Enterprise Portal, en la pestaña **Descargar uso**. Seleccione **API Access Key** (Clave de acceso de la API) y, a continuación, pegue la clave en el cuadro **Clave de cuenta**.
1. Los datos tardarán aproximadamente de 5 a 30 minutos en cargarse en Power BI, según el tamaño de los conjuntos de datos.

## <a name="reports-faq"></a>Preguntas más frecuentes sobre informes

En esta sección se tratan las preguntas comunes sobre los informes.

### <a name="why-is-my-cost-showing-as-0"></a>¿Por qué se muestra el costo como $0?

Por los clientes de **inscripción directa**, los administradores de empresa pueden proporcionar a los propietarios de la cuenta y administradores de departamento acceso a la información de costos y precios en los informes de uso. Siga estos pasos:

1. En Azure Enterprise Portal, seleccione **Administrar** en el panel de navegación izquierdo.
1. Seleccione el lápiz azul que hay junto a DA View Charges (El administrador del departamento ve los cargos).
1. Seleccione **Enabled** (Habilitado) y guarde.
1. Seleccione el lápiz azul que hay junto a AO View Charges (El propietario de la cuenta ve los cargos).
1. Seleccione **Enabled** (Habilitado) y guarde.

> [!NOTE]
> Si es propietario de la cuenta o administrador de departamento, póngase en contacto con su administrador de empresa para habilitar la característica de precios.

Para los clientes de **inscripción indirecta**, consulte con su partner para comprobar que haya habilitado la característica de precios por usted. Esto solo lo puede realizar el partner. Una vez habilitado, puede ver los costos y los precios de su inscripción como administrador de empresa.

Si un partner desea habilitar la característica para ver los cargos para un administrador de la cuenta o administrador de departamento, debe seguir los pasos descritos en **inscripción directa**.

### <a name="why-is-there-no-sku-information-on-my-usage-detail-report"></a>¿Por qué no hay información de SKU en el informe de detalles de uso?

El informe de detalles de uso no contiene información de SKU. Sin embargo, el informe sí contiene información de uso para que pueda descargar el informe de la hoja de precios para obtener la información de las SKU.

### <a name="why-doesnt-the-total-amount-on-azure-marketplace-match-the-reports-for-usage-summary-and-detail"></a>¿Por qué la cantidad total en Azure Marketplace no coincide con los informes de resumen y detalles de uso?

El informe de cargos de Azure Marketplace muestra solo los cargos basados en el uso. No se muestran las tarifas puntuales. Consulte la página de resumen de uso para ver los cargos más actualizados basados en el uso y las tarifas puntuales.

### <a name="why-is-there-no-information-on-my-api-report"></a>¿Por qué no hay información en el informe de la API?

Las claves de API expiran cada seis meses. Si tiene algún problema, un administrador de empresa debe generar una nueva clave de API. Recuerde seguir los pasos descritos en las preguntas más frecuentes sobre informes de API.

### <a name="why-isnt-my-power-bi-report-working"></a>¿Por qué mi informe de Power BI no funciona?

Si tiene problemas con Power BI, registre un vale en el [equipo de Soporte técnico de Power BI](https://support.powerbi.com).

### <a name="why-dont-my-resource-tags-show-on-my-reports"></a>¿Por qué las etiquetas de recursos no aparecen en mis informes?

Las etiquetas de recursos se administran en Azure Portal. Puede ponerse en contacto con el equipo de suscripciones de Azure en [Azure Portal](https://portal.azure.com). Siga los pasos descritos en el artículo [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).

### <a name="why-does-my-resource-rate-change-every-day"></a>¿Por qué cambia la tasa de recursos cada día?

La tasa de recursos que se muestra en el informe de detalles de uso es un valor calculado. Representa la tasa mensual media que se cargó por el servicio. Esta tasa de recursos se calcula a partir del promedio del compromiso mensual y los cargos mensuales de uso por encima del límite de una unidad de servicio. La parte del uso que se aplica a las tasas de uso por encima del límite y al compromiso cambia hasta el día en que se cierre el mes. Por lo tanto, la tasa de recursos que se enumera también cambia durante el mes. La tasa de recursos se bloquea al quinto día después del final del mes.

### <a name="glossary-of-processes-for-calculating-the-resource-rate"></a>Glosario de procesos para calcular la tasa de recursos

- **Total de unidades sin procesar:** Cantidad consumida en el informe de uso detallado.
- **Recurso de MOCP por unidad:** Sistema de uso ascendente que emite los usos de cada servicio en distintas unidades. (Valor preestablecido)
- **Consumo por unidad:** Unidad de medida de Azure Enterprise. (Valor preestablecido)
- **Precio:** Precio unitario de Azure Enterprise Portal.
- **Costo total:** Costo ampliado del informe de uso detallado o del uso del compromiso más el uso por encima del límite de Azure Enterprise Portal.

### <a name="charges-calculations"></a>Cálculos de los cargos

- **Convertir a recursos de MOCP consumidos** = `ROUND(Total RAW Units * MOCP Resource Per Unit,4)`
- **Convertir a unidades consumidas** = `Consumed MOCP Resources / Consumption per Unit`
- **Calcular costo total** = `Consumed Units * Price`

### <a name="logic-in-the-usage-calculation-logic"></a>Lógica del cálculo de uso

**Tasa de recursos** = `Total Cost /(Total RAW Units / MOCP Resource Per Unit)`

La tasa de recursos se obtiene en función de los cargos. Podría no coincidir con el precio unitario real que aparece en la hoja de precios.

En el informe de descarga de datos de uso, puede ver el uso de recursos sin procesar hasta seis lugares decimales. Estos datos se usan para el cálculo de los cargos de uso por encima del límite. Sin embargo, los datos de uso que se muestran en Azure Enterprise Portal se redondean hasta cuatro posiciones decimales para las unidades de compromiso y se truncan en cero decimales para las unidades de uso por encima del límite. En Azure Enterprise Portal, todo el uso por encima del límite se cobra por las unidades completas únicamente. Puede que vea una gran diferencia entre el precio unitario y la tasa de recursos para el uso que se cobra como uso por encima del límite o en meses mixtos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Revisar los cargos facturados
> * Revisar los cargos por uso del servicio por encima del límite
> * Revisar la factura de Marketplace

Continúe con el siguiente artículo para más información con el uso del portal del Contrato Enterprise de Azure.

> [!div class="nextstepaction"]
> [Introducción a Azure Enterprise Portal](../manage/ea-portal-get-started.md)
