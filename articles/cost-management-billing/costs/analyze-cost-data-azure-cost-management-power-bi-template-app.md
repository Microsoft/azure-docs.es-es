---
title: Análisis de los costos de Azure con la aplicación para Power BI
description: En este artículo se explica cómo instalar y usar la aplicación Azure Cost Management para Power BI.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.reviewer: benshy
ms.openlocfilehash: 388fbcf8dde2e01bf7f4e6c8b4ecbe344fd036d2
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290906"
---
# <a name="analyze-cost-with-the-azure-cost-management-power-bi-app-for-enterprise-agreements-ea"></a>Análisis del costo con la aplicación Azure Cost Management para Power BI para Contrato Enterprise (EA)

En este artículo se explica cómo instalar y usar la aplicación Azure Cost Management para Power BI. La aplicación le ayuda a analizar y administrar los costos de Azure en Power BI. Puede usar la aplicación para supervisar costos, tendencias de uso e identificar opciones de optimización de costos para reducir los gastos.

Puede usar la aplicación tal cual, o bien modificarla para ampliar los filtros, vistas y visualizaciones predeterminados para personalizarlos según sus necesidades. A continuación, úselo para combinar datos adicionales y crear informes personalizados para obtener vistas holísticas del costo total de la empresa.

Actualmente, la aplicación Azure Cost Management para Power BI solo admite clientes con un [Contrato Enterprise](https://azure.microsoft.com/pricing/enterprise-agreement/).

> [!NOTE]
> Las aplicaciones de plantilla de Power BI no admiten la descarga del archivo PBIX.

## <a name="prerequisites"></a>Requisitos previos

- Una [licencia de Power BI Pro](/power-bi/service-self-service-signup-for-power-bi) para instalar y usar la aplicación.
- Para conectarse a los datos, debe usar una cuenta de [administrador de empresa](../manage/understand-ea-roles.md).

## <a name="installation-steps"></a>Pasos de instalación

Para instalar la aplicación

1. Abra la [aplicación Azure Cost Management para Power BI](https://aka.ms/costmgmt/ACMApp).
2. En la página AppSource de Power BI, seleccione **Obtenerla ahora**.
3. Seleccione **Continuar** para aceptar las condiciones de uso y la directiva de privacidad.
4. En el cuadro **¿Quiere instalar esta aplicación de Power BI?** , seleccione **Instalar**.
5. Si es necesario, cree un área de trabajo y seleccione **Continuar**.
6. Cuando finalice la instalación, aparecerá una notificación que indica que la nueva aplicación está lista.
7. Seleccione **Ir a la aplicación**.
8. En **Empezar a trabajar con la nueva aplicación**, en **Conectar los datos**, seleccione **Conectar**.  
  ![Empezar a trabajar con la nueva aplicación: Conectar](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/connect-data2.png)
9. En el cuadro de diálogo que aparece, escriba el número de inscripción de Contrato Enterprise para **BillingProfileIdOrEnrollmentNumber**. Especifique el número de meses de datos que se van a obtener. Deje el valor de **Ámbito** predeterminado de **Número de inscripción** y, a continuación, seleccione **Siguiente**.  
  ![Introducción de la información de inscripción de EA](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ea-number.png)  
10. El siguiente cuadro de diálogo se conecta a Azure y obtiene los datos necesarios para las recomendaciones de instancia reservada. *Deje los valores predeterminados tal como están configurados* y seleccione **Iniciar sesión**.  
  ![Conexión a Azure](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/autofit.png)  
11. El paso de instalación final se conecta a su inscripción de Contrato Enterprise y requiere una cuenta de [administrador de empresa](../manage/understand-ea-roles.md). Seleccione **Iniciar sesión** para autenticarse con su inscripción de Contrato Enterprise. Este paso también inicia una acción de actualización de datos en Power BI.  
  ![Conexión a la inscripción de Contrato Enterprise](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ea-auth.png)  
    > [!NOTE]
    > El proceso de actualización de datos puede tardar bastante tiempo en completarse. La duración depende del número de meses especificado y de la cantidad de datos que haya que sincronizar.
12. Para comprobar el estado de la actualización de datos, seleccione la pestaña **Conjuntos de datos** en el área de trabajo. Mire junto a la marca de tiempo Actualizado. Si todavía se está actualizando, verá un indicador que muestra que la actualización está en curso.  
  ![Actualización de datos](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/data-refresh2.png)

Una vez finalizada la actualización de datos, seleccione la aplicación Azure Cost Management para ver los informes creados previamente.

## <a name="reports-available-with-the-app"></a>Informes disponibles con la aplicación

Están disponibles los siguientes informes en la aplicación.

**Getting Started** (Introducción): proporciona vínculos útiles a documentación y vínculos para proporcionar comentarios.

**Account overview** (Información general de la cuenta): el informe muestra un resumen mensual de la información, incluidos:

- Cargos de créditos
- Nuevas compras
- Cargos de Azure Marketplace
- Uso por encima del límite y cargos totales

**Usage by Subscriptions and Resource Groups** (Uso por suscripciones y grupos de recursos): proporciona una vista del costo en el tiempo y gráficos que muestran el costo por suscripción y grupo de recursos.

**Usage by Services** (Uso por servicios): proporciona una vista con el tiempo de uso por MeterCategory. Puede realizar un seguimiento de los datos de uso y profundizar en las anomalías para comprender los cambios de uso abruptos e interrupciones.

**Top 5 Usage drivers** (Principales 5 impulsores del uso): el informe muestra un resumen del costo filtrado por los cinco principales valores de MeterCategory y los valores de MeterName correspondientes.

**Windows Server AHB Usage** (Uso de AHB de Windows Server): el informe muestra el número de máquinas virtuales con la Ventaja híbrida de Azure habilitada. También muestra el número de núcleos o vCPU usados por las máquinas virtuales.

![Informe completo de Ventaja híbrida de Azure](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ahb-report-full.png)

El informe también identifica las máquinas virtuales de Windows en las que la ventaja híbrida está **habilitada** pero hay _menos de_ ocho vCPU. También muestra dónde **no está habilitada** la ventaja híbrida que tiene ocho _o más_ vCPU. Esta información le ayudará a usar completamente su ventaja híbrida. Aplique la ventaja a las máquinas virtuales más costosas para maximizar el posible ahorro.

![Ventajas híbridas de Azure: menos de ocho vCPU y vCPU no habilitados](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ahb-report.png)

**RI Chargeback** (Contracargo de RI): el informe le ayuda a comprender dónde y qué parte de una ventaja de instancia reservada (RI) se aplica por región, suscripción, grupo de recursos o recurso. El informe utiliza datos de uso amortizados para mostrar la vista.

Puede aplicar un filtro en _chargetype_ para ver los datos de sobreutilización de instancias reservadas.

Para más información sobre los datos amortizados, consulte [Obtención del uso y los costos de reservas de Contrato Enterprise](/azure/cost-management-billing/reservations/understand-reserved-instance-usage-ea).

**RI Savings** (Ahorro de RI): el informe muestra el ahorro acumulado por las reservas de suscripción, grupo de recursos y nivel de recurso. Muestra lo siguiente:

- Costo con reserva
- Costo estimado a petición si la reserva no se ha aplicado al uso
- Ahorro de costos acumulado en la reserva

 El informe resta el costo de los residuos de reserva infrautilizados del ahorro total. Los residuos no se producirán sin una reserva.

Puede usar los datos de uso amortizados para compilar sobre los datos.

<a name="shared-recommendation"></a>
**VM RI Coverage (shared recommendation)** [Cobertura de RI de máquina virtual (recomendación compartida)]: el informe se divide entre el uso de máquina virtual a petición y el uso de instancias reservadas de la máquina virtual durante el período seleccionado. Proporciona recomendaciones para las compras de instancias reservadas de máquinas virtuales en un ámbito compartido.

Para usar el informe, seleccione el filtro de exploración en profundidad.

![Informe de cobertura de instancias reservadas de máquinas virtuales: seleccionar Explorar en profundidad](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ri-drill-down2.png)

Seleccione la región que quiere analizar. A continuación, seleccione el grupo de flexibilidad de tamaño de instancia, etc.

Para cada nivel de exploración en profundidad, se aplican los siguientes filtros al informe:

- Los datos de cobertura de la derecha son el filtro que muestra la cantidad de uso que se cobra mediante la tarifa a petición frente a la cantidad que cubre la reserva.
- También se filtran las recomendaciones.

La tabla de recomendaciones proporciona recomendaciones para la compra de la reserva, en función de los tamaños de máquina virtual usados.

Los valores de _Normalized Size_ (Tamaño normalizado) y _Recommended Quantity Normalized_ (Cantidad recomendada normalizada) le ayudan a normalizar la compra al menor tamaño de un grupo de flexibilidad de tamaño de instancia. La información es útil si tiene previsto comprar solo una reserva para todos los tamaños del grupo de flexibilidad de tamaño de instancia.

![Recomendaciones de instancias reservadas](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ri-recomendations.png)

**VM RI Coverage (single recommendation)** [Cobertura de RI de máquina virtual (recomendación única)]: el informe se divide entre el uso de máquina virtual a petición y el uso de instancias reservadas de máquinas virtuales durante el período de tiempo seleccionado. Proporciona recomendaciones para las compras de instancias reservadas de máquinas virtuales en el ámbito de una suscripción.

Para más información sobre cómo usar el informe, consulte la sección sobre la [cobertura de RI de máquina virtual (recomendación compartida)](#shared-recommendation).

**RI purchases** (Compras de RI): el informe muestra las compras de instancias reservadas durante el período especificado.

**Price sheet** (Hoja de precios): este informe muestra una lista detallada de precios específicos de una cuenta de facturación o de la inscripción de Contrato Enterprise.

## <a name="troubleshoot-problems"></a>Solucionar problemas

Si tiene problemas con la aplicación Power BI, es posible que la siguiente información sea útil para solucionar problemas.

### <a name="error-processing-the-data-in-the-dataset"></a>Error al procesar los datos del conjunto de datos

Puede que reciba el siguiente error:

```
There was an error when processing the data in the dataset.
Data source error: {"error":{"code":"ModelRefresh_ShortMessage_ProcessingError","pbi.error":{"code":"ModelRefresh_ShortMessage_ProcessingError","parameters":{},"details":[{"code":"Message","detail":{"type":1,"value":"We cannot convert the value \"Required Field: 'Enr...\" to type List."}}],"exceptionCulprit":1}}} Table: <TableName>.
```

En lugar de `<TableName>` aparecería el nombre de una tabla.

#### <a name="cause"></a>Causa

El valor predeterminado de **Scope** (Ámbito) de `Enrollment Number` cambió en la conexión a Cost Management.

#### <a name="solution"></a>Solución

Vuelva a conectarse a Cost Management y establezca el valor de **Scope** (Ámbito) en `Enrollment Number`. No escriba el número de inscripción de su organización. En su lugar, escriba `Enrollment Number` exactamente como aparece en la siguiente imagen.

![Especificación de la información de inscripción de EA](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/ea-number.png)  

### <a name="budgetamount-error"></a>Error de BudgetAmount

Puede que reciba el siguiente error:

```
Something went wrong
There was an error when processing the data in the dataset.
Please try again later or contact support. If you contact support, please provide these details.
Data source error: The 'budgetAmount' column does not exist in the rowset. Table: Budgets.
```

#### <a name="cause"></a>Causa

Este error se produce debido a un error con los metadatos subyacentes. El problema se produce porque no hay ningún presupuesto disponible en **Cost Management > Presupuesto** en Azure Portal. La corrección del error está en proceso de implementación en Power BI Desktop y en el servicio Power BI. 

#### <a name="solution"></a>Solución

- Hasta que se solucione el error, puede darle una solución alternativa mediante la incorporación de un presupuesto de prueba a Azure Portal a nivel de cuenta de facturación o inscripción en EA. Este presupuesto de prueba desbloquea la conexión con Power BI. Para más información sobre la creación de presupuestos, consulte [Tutorial: Creación y administración de presupuestos en Azure](tutorial-acm-create-budgets.md).


### <a name="invalid-credentials-for-azureblob-error"></a>Error de credenciales no válidas para AzureBlob

Puede que reciba el siguiente error:

```
Failed to update data source credentials: The credentials provided for the AzureBlobs source are invalid.
```

#### <a name="cause"></a>Causa

Este error se produce si se cambia el método de autenticación de la conexión de blobs de AutoFitComboMeter.

#### <a name="solution"></a>Solución

1. Conexión a los datos.
1. Después de que especifique la inscripción a EA y el número de meses, asegúrese de que deja el valor predeterminado **Anonymous** (Anónimo) en Authentication method (Método de autenticación) y **None** (Ninguno) en Privacy level setting (Configuración del nivel de privacidad).  
  ![Conexión a Azure](./media/analyze-cost-data-azure-cost-management-power-bi-template-app/autofit-troubleshoot.png)  
1. En la siguiente página, seleccione **OAuth2** en Authentication method (Método de autenticación) y **None** (Ninguno) en Privacy level (Nivel de privacidad). Luego, inicie sesión para autenticarse con su inscripción. Este paso también inicia una actualización de los datos de Power BI.


## <a name="data-reference"></a>Referencia de datos

La información siguiente resume los datos disponibles en la aplicación. También hay vínculos a las API que proporcionan información detallada de los campos de datos y los valores.

| **Referencia de tabla** | **Descripción** |
| --- | --- |
| **AutoFitComboMeter** | Datos incluidos en la aplicación para normalizar la recomendación de instancia reservada y el uso al menor tamaño en el grupo de familias de instancias. |
| [**Balance summary**](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary#response) (Resumen del saldo) | Resumen del saldo de las instancias de Contrato Enterprise. |
| [**Budgets**](/rest/api/consumption/budgets/get#definitions) (Presupuestos) | Detalles del presupuesto para ver los costos reales o el uso de los objetivos de presupuesto existentes. |
| [**Pricesheets**](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet#see-also) (Hojas de precios) | Tasas de los medidores aplicables para el perfil de facturación proporcionado o la inscripción a Contrato Enterprise. |
| [**RI charges**](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges#response) (Cargos de RI) | Cargos asociados a las instancias reservadas en los últimos 24 meses. |
| [**RI recommendations (shared)** ](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#response) [Recomendaciones de RI (compartidas)] | Recomendaciones de compras de instancias reservadas basadas en todas las tendencias de uso de las suscripciones durante los últimos 7, 30 o 60 días. |
| [**RI recommendations (single)** ](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#response-1) [Recomendaciones de RI (únicas)] | Recomendaciones de compra de instancias reservadas basadas en las tendencias de uso de las suscripciones únicas durante los últimos 7, 30 o 60 días. |
| [**RI usage details**](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#response) (Detalles de uso de RI) | Detalles de consumo de las instancias reservadas existentes durante el último mes. |
| [**RI usage summary**](/rest/api/consumption/reservationssummaries/list) (Resumen de uso de RI) | Porcentaje diario de uso de reservas de Azure. |
| [**Usage details**](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#usage-details-field-definitions) (Detalles de uso) | Un desglose de cantidades consumidas y de cargos estimados para el perfil de facturación determinado en la inscripción a Contrato Enterprise. |
| [**Usage details amortized**](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#usage-details-field-definitions) (Detalles de uso amortizados) | Un desglose de cantidades consumidas y de cargos amortizados estimados para el perfil de facturación determinado en la inscripción a Contrato Enterprise. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la configuración de datos, la actualización, el uso compartido de informes y la personalización de informes adicional, consulte los siguientes artículos:

- [Configuración de actualización programada](/power-bi/refresh-scheduled-refresh)
- [Uso compartido de informes y paneles de Power BI con compañeros y otros usuarios](/power-bi/service-share-dashboards)
- [Suscripción personal y de otros usuarios a informes y paneles en el servicio Power BI](/power-bi/service-report-subscribe)
- [Descarga de un informe desde el servicio Power BI a Power BI Desktop (versión preliminar)](/power-bi/service-export-to-pbix)
- [Almacenamiento de un informe en el servicio Power BI y en Power BI Desktop](/power-bi/service-report-save)
- [Creación de un informe en el servicio Power BI mediante la importación de un conjunto de datos](/power-bi/service-report-create-new)
