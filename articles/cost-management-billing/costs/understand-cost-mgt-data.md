---
title: Información sobre los datos de Azure Cost Management
description: Este artículo le ayudará a comprender mejor qué datos se incluyen en Azure Cost Management y con qué frecuencia se procesan, recopilan, muestran y cierran.
author: bandersmsft
ms.author: banders
ms.date: 03/02/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: micflan
ms.openlocfilehash: 904ea7a50e4546d2721a9e701c78b6b77ed2d43a
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683188"
---
# <a name="understand-cost-management-data"></a>Descripción de los datos de Cost Management

Este artículo lo ayudará a comprender mejor los datos de costo y uso de Azure que se incluyen en Azure Cost Management. En él se explica con qué frecuencia se procesan, se recopilan, se muestran y se cierran los datos. Cada mes, se le factura el uso que haga de Azure. Si bien los ciclos de facturación son períodos mensuales, las fechas de inicio y finalización de los ciclos varían según el tipo de suscripción. La frecuencia con que Cost Management recibe datos de uso varía en función de diferentes factores. Por ejemplo, cuánto tarda el procesamiento de los datos y cada cuánto transmiten los servicios de Azure el uso al sistema de facturación.

Cost Management incluye todo el uso y todas las compras, incluidas reservas y ofertas de terceros para cuentas de Contrato Enterprise (EA). Las cuentas de Contrato de cliente de Microsoft y las suscripciones individuales con tarifas de pago por uso solo incluyen el uso de los servicios de Azure y Marketplace. No se incluyen los costos de soporte técnico ni otros costos. Los costos se calculan hasta que se genera una factura y no tienen en cuenta los créditos.

Si su suscripción es nueva, no podrá usar inmediatamente las características de Cost Management. Para poder hacerlo deberán transcurrir un máximo de 48 horas.

## <a name="supported-microsoft-azure-offers"></a>Ofertas compatibles de Microsoft Azure

La siguiente información muestra las [ofertas de Microsoft Azure](https://azure.microsoft.com/support/legal/offer-details/) compatibles actualmente con Azure Cost Management. Una oferta de Azure es el tipo de la suscripción a Azure que tiene. Los datos están disponibles en Cost Management a partir de la fecha **Datos disponibles desde**. Si una suscripción cambia las ofertas, los costos anteriores a la fecha de cambio de la oferta no están disponibles.

| **Categoría**  | **Nombre de la oferta** | **Identificador de la cuota** | **Número de la oferta** | **Datos disponibles desde** |
| --- | --- | --- | --- | --- |
| **Azure Government** | Azure Government Enterprise                                                         | EnterpriseAgreement_2014-09-01 | MS-AZR-USGOV-0017P | Mayo de 2014<sup>1</sup> |
| **Contrato Enterprise (EA)** | Desarrollo/pruebas - Enterprise                                                        | MSDNDevTest_2014-09-01 | MS-AZR-0148P | Mayo de 2014<sup>1</sup> |
| **Contrato Enterprise (EA)** | [Microsoft Azure Enterprise](https://azure.microsoft.com/offers/enterprise-agreement-support-upgrade) | EnterpriseAgreement_2014-09-01 | MS-AZR-0017P | Mayo de 2014<sup>1</sup> |
| **Contrato de cliente de Microsoft** | [Plan de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0017g) | EnterpriseAgreement_2014-09-01 | N/D | Marzo de 2019<sup>3</sup> |
| **Contrato de cliente de Microsoft** | [Plan de Microsoft Azure para Desarrollo/pruebas](https://azure.microsoft.com/offers/ms-azr-0148g) | MSDNDevTest_2014-09-01 | N/D | Marzo de 2019<sup>3</sup> |
| **Contrato de cliente de Microsoft admitido por los asociados** | Plan de Microsoft Azure | CSP_2015-05-01, CSP_MG_2017-12-01 y CSPDEVTEST_2018-05-01<br><br>El identificador de cuota se reutiliza para el Contrato de cliente de Microsoft y las suscripciones de CSP heredadas. Actualmente, solo se admiten las suscripciones de Contrato de cliente de Microsoft. | N/D | Octubre de 2019 |
| **Microsoft Developer Network (MSDN)** | [Plataformas de MSDN](https://azure.microsoft.com/offers/ms-azr-0062p)<sup>4</sup> | MSDN_2014-09-01 | MS-AZR-0062P | 2 de octubre de 2018<sup>2</sup> |
| **Pay-As-You-Go** | [Pay-As-You-Go](https://azure.microsoft.com/offers/ms-azr-0003p)                  | PayAsYouGo_2014-09-01 | MS-AZR-0003P | 2 de octubre de 2018<sup>2</sup> |
| **Pay-As-You-Go** | [Desarrollo/pruebas - Pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p)         | MSDNDevTest_2014-09-01 | MS-AZR-0023P | 2 de octubre de 2018<sup>2</sup> |
| **Pay-As-You-Go** | [Microsoft Partner Network](https://azure.microsoft.com/offers/ms-azr-0025p)      | MPN_2014-09-01 | MS-AZR-0025P | 2 de octubre de 2018<sup>2</sup> |
| **Pay-As-You-Go** | [Evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p)<sup>4</sup>         | FreeTrial_2014-09-01 | MS-AZR-0044P | 2 de octubre de 2018<sup>2</sup> |
| **Pay-As-You-Go** | [Azure bajo licencia Open](https://azure.microsoft.com/offers/ms-azr-0111p)<sup>4</sup>      | AzureInOpen_2014-09-01 | MS-AZR-0111P | 2 de octubre de 2018<sup>2</sup> |
| **Pay-As-You-Go** | Pase para Azure<sup>4</sup>                                                            | AzurePass_2014-09-01 | MS-AZR-0120P, MS-AZR-0122P - MS-AZR-0125P, MS-AZR-0128P - MS-AZR-0130P | 2 de octubre de 2018<sup>2</sup> |
| **Visual Studio** | [Visual Studio Enterprise – MPN](https://azure.microsoft.com/offers/ms-azr-0029p)<sup>4</sup>     | MPN_2014-09-01 | MS-AZR-0029P | 2 de octubre de 2018<sup>2</sup> |
| **Visual Studio** | [Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p)<sup>4</sup>         | MSDN_2014-09-01 | MS-AZR-0059P | 2 de octubre de 2018<sup>2</sup> |
| **Visual Studio** | [Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p)<sup>4</sup>    | MSDNDevTest_2014-09-01 | MS-AZR-0060P | 2 de octubre de 2018<sup>2</sup> |
| **Visual Studio** | [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p)<sup>4</sup>           | MSDN_2014-09-01 | MS-AZR-0063P | 2 de octubre de 2018<sup>2</sup> |
| **Visual Studio** | [Visual Studio Enterprise: BizSpark](https://azure.microsoft.com/offers/ms-azr-0064p)<sup>4</sup> | MSDN_2014-09-01 | MS-AZR-0064P | 2 de octubre de 2018<sup>2</sup> |

_<sup>**1**</sup> Para los datos antes de mayo de 2014, visite [Azure Enterprise Portal](https://ea.azure.com)._

_<sup>**2**</sup> Para los datos antes del 2 de octubre de 2018, visite el [Centro de cuentas de Azure](https://account.azure.com/subscriptions)._

_<sup>**3**</sup> Los contratos de cliente de Microsoft comenzaron en marzo de 2019 y no tienen ningún dato histórico antes de este punto._

_<sup>**4**</sup> Es posible que los datos históricos de las suscripciones basadas en crédito y pagadas por adelantado no coincidan con la factura. Consulte [Los datos históricos pueden no coincidir con la factura](#historical-data-might-not-match-invoice) a continuación._

Las siguientes ofertas todavía no se admiten:

| Category  | **Nombre de la oferta** | **Identificador de la cuota** | **Número de la oferta** |
| --- | --- | --- | --- |
| **Azure Alemania** | [Pago por uso de Azure Germany](https://azure.microsoft.com/offers/ms-azr-de-0003p) | PayAsYouGo_2014-09-01 | MS-AZR-DE-0003P |
| **Azure Government** | Pago por uso de Azure Government | PayAsYouGo_2014-09-01 | MS-AZR-USGOV-0003P |
| **Proveedor de soluciones en la nube (CSP)** | Microsoft Azure                                    | CSP_2015-05-01 | MS-AZR-0145P |
| **Proveedor de soluciones en la nube (CSP)** | Azure Government CSP                               | CSP_2015-05-01 | MS-AZR-USGOV-0145P |
| **Proveedor de soluciones en la nube (CSP)** | Azure Alemania en CSP para Microsoft Cloud Alemania   | CSP_2015-05-01 | MS-AZR-DE-0145P |
| **Pay-As-You-Go**                 | Paquete de inicio de Azure for Students | DreamSpark_2015-02-01 | MS-AZR-0144P |
| **Pay-As-You-Go** | [Azure for Students](https://azure.microsoft.com/offers/ms-azr-0170p)<sup>4</sup> | AzureForStudents_2018-01-01 | MS-AZR-0170P |
| **Pay-As-You-Go**                 | [Patrocinio de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0036p/) | Sponsored_2016-01-01 | MS-AZR-0036P |
| **Planes de soporte técnico** | Soporte técnico Standard                    | Default_2014-09-01 | MS-AZR-0041P |
| **Planes de soporte técnico** | Soporte técnico Professional Direct         | Default_2014-09-01 | MS-AZR-0042P |
| **Planes de soporte técnico** | Soporte técnico Developer                   | Default_2014-09-01 | MS-AZR-0043P |
| **Planes de soporte técnico** | Planes de soporte técnico de Alemania                | Default_2014-09-01 | MS-AZR-DE-0043P |
| **Planes de soporte técnico** | Soporte técnico Standard de Azure Government   | Default_2014-09-01 | MS-AZR-USGOV-0041P |
| **Planes de soporte técnico** | Soporte técnico Pro-Direct de Azure Government | Default_2014-09-01 | MS-AZR-USGOV-0042P |
| **Planes de soporte técnico** | Soporte técnico para desarrolladores de Azure Government  | Default_2014-09-01 | MS-AZR-USGOV-0043P |

### <a name="free-trial-to-pay-as-you-go-upgrade"></a>Actualización de evaluación gratuita a pago por uso

Para información sobre la disponibilidad de los servicios de nivel gratuito después de actualizar una evaluación gratuita a los precios de pago, consulte las [preguntas más frecuentes sobre las cuentas gratuitas de Azure](https://azure.microsoft.com/free/free-account-faq/).

### <a name="determine-your-offer-type"></a>Determinación del tipo de oferta

Si no ve los datos de una suscripción y desea determinar si tal suscripción se encuentra entre las ofertas admitidas, puede comprobar si realmente se admite. Para validar que se admite una suscripción a Azure, inicie sesión en [Azure Portal](https://portal.azure.com). Luego, seleccione **Todos los servicios** en el panel del menú izquierdo. En la lista de servicios, seleccione **Suscripciones**. En el menú de la lista de suscripciones, seleccione la suscripción que desee comprobar. La suscripción se muestra en la pestaña Información general y puede ver la **Oferta** y el **Id. de oferta**. En la imagen siguiente se muestra un ejemplo:

![Ejemplo de la pestaña Información general de la suscripción mostrando la Oferta y el Id. de oferta](./media/understand-cost-mgt-data/offer-and-offer-id.png)

## <a name="costs-included-in-cost-management"></a>Costos incluidos en Cost Management

Las siguientes tablas muestran los datos que se incluyen o no se incluyen en Cost Management. Todos los costos se calculan hasta que se genera una factura. Los costos que se muestran no incluyen los créditos gratuitos ni los pagados por adelantado.

| **Se incluye** | **No se incluye** |
| --- | --- |
| Uso del servicio de Azure<sup>5</sup>        | Cargos de soporte técnico - Para obtener más información, consulte [Explicación de los términos de facturación](../understand/understand-invoice.md). |
| Uso de la oferta de Marketplace<sup>6</sup> | Impuestos - Para obtener más información, consulte [Explicación de los términos de facturación](../understand/understand-invoice.md). |
| Compras de Marketplace<sup>6</sup>      | Créditos - Para obtener más información, consulte [Explicación de los términos de facturación](../understand/understand-invoice.md). |
| Compras de reserva<sup>7</sup>      |  |
| Amortización de las compras de reserva<sup>7</sup>      |  |

_<sup>**5**</sup> El uso del servicio de Azure se basa en la reserva y la negociación de los precios._

_<sup>**6**</sup> Las compras de Marketplace no están disponible para las ofertas de MSDN ni Visual Studio por el momento._

_<sup>**7**</sup> En este momento, las compras de reserva solo están disponibles para las cuentas de Contrato Enterprise (EA) y Contrato de cliente de Microsoft._

## <a name="how-tags-are-used-in-cost-and-usage-data"></a>Uso de las etiquetas en los datos de costo y uso

Azure Cost Management recibe etiquetas como parte de cada registro de uso enviado por los servicios individuales. Se aplican las restricciones siguientes a estas etiquetas:

- Las etiquetas deben aplicarse directamente a los recursos y no se heredan implícitamente del grupo de recursos primario.
- Las etiquetas de recursos solo se admiten para los recursos implementados en grupos de recursos.
- Es posible que algunos recursos implementados no admitan etiquetas o que no incluyan etiquetas en los datos de uso; consulte [Compatibilidad de etiquetas de los recursos de Azure](../../azure-resource-manager/tag-support.md).
- Las etiquetas de recursos solo se incluyen en los datos de uso mientras se aplica la etiqueta; las etiquetas no se aplican a los datos históricos.
- Las etiquetas de recursos solo están disponibles en Cost Management después de que se actualicen los datos; consulte [Retención y actualizaciones de datos de uso y costos](#cost-and-usage-data-updates-and-retention).
- Las etiquetas de recursos solo están disponibles en Cost Management cuando el recurso está activo o en ejecución y genera registros de uso (por ejemplo, no cuando una máquina virtual está desasignada).
- La administración de etiquetas requiere el acceso de colaborador a cada recurso.
- La administración de directivas de etiquetas requiere el acceso de propietario o de colaborador de directiva a un grupo de administración, suscripción o grupo de recursos.
    
Si no ve una etiqueta específica en Cost Management, tenga en cuenta lo siguiente:

- ¿Se aplicó la etiqueta directamente al recurso?
- ¿Se aplicó la etiqueta hace más de 24 horas? Consulte [Retención y actualizaciones de datos de uso y costos](#cost-and-usage-data-updates-and-retention)
- ¿Admite etiquetas el tipo de recurso? Los siguientes tipos de recursos no admiten etiquetas en los datos de uso a partir del 1 de diciembre de 2019. Consulte [Compatibilidad de etiquetas de los recursos de Azure](../../azure-resource-manager/tag-support.md) para ver la lista completa de lo que se admite.
    - Directorios de Azure Active Directory B2C
    - Azure Bastion
    - Instancias de Azure Firewall
    - Azure NetApp Files
    - Data Factory
    - Databricks
    - Equilibradores de carga
    - Network Watcher
    - Notification Hubs
    - Azure Service Bus
    - Time Series Insights
    
Estas son algunas sugerencias para trabajar con etiquetas:

- Planee por anticipado y defina una estrategia de etiquetado que le permita desglosar los costos por organización, aplicación, entorno, etc.
- Use Azure Policy para copiar las etiquetas de grupos de recursos en recursos individuales y aplicar la estrategia de etiquetado.
- Use Tag API junto con Query o UsageDetails para obtener todo el costo basado en las etiquetas actuales.


## <a name="cost-and-usage-data-updates-and-retention"></a>Retención y actualizaciones de datos de uso y costos

Los datos de uso y costos normalmente están disponibles en Administración de costos + facturación en Azure Portal y las [API de apoyo](../index.yml) en un plazo de 8 a 24 horas. Tenga en cuenta los puntos siguientes al revisar los costes:

- Cada servicio de Azure (como Storage, Compute y SQL) emite el uso en diferentes intervalos: puede ver los datos de algunos servicios antes que otros.
- La estimación de los cargos para el período de facturación actual se actualiza seis veces al día.
- La estimación de los cargos para el período de facturación actual puede cambiar según se incurre en un uso mayor.
- Cada actualización es acumulativa e incluye todos los elementos de línea y la información de la actualización anterior.
- Azure finaliza o _cierra_ el período de facturación actual hasta 72 horas (tres días naturales) después de finalizado el período de facturación.

Los ejemplos siguientes muestran cómo podrían terminar los períodos de facturación:

* Suscripciones del contrato Enterprise (EA): si el mes de facturación termina el 31 de marzo, la estimación de cargos se actualiza hasta 72 horas más tarde. En este ejemplo, a la medianoche (UTC) del 4 de abril.
* Suscripciones de pago por uso: si el mes de facturación termina el 15 de mayo, la estimación de cargos podría actualizarse hasta 72 horas más tarde. En este ejemplo, a la medianoche (UTC) del 19 de mayo.

Cuando los datos de uso y costos están disponibles en Administración de costos + facturación, se conservarán durante al menos siete años.

### <a name="rerated-data"></a>Nueva valoración de los datos

Si usa las [API de Cost Management](../index.yml), Power BI o Azure Portal para recuperar datos, es posible que los cargos del período de facturación actual se vuelvan a valorar y, consecuentemente, cambien hasta que se cierre la factura.

## <a name="cost-rounding"></a>Redondeo de los costos

Los costos que se muestran en Cost Management se redondean. Los costos devueltos por la API de consulta no se redondean. Por ejemplo:

- Análisis de costos en Azure Portal: los cargos se redondean mediante reglas de redondeo estándar: los valores de más de 0,5 se redondean al alza; de lo contrario, los costos se redondean a la baja. El redondeo solo se produce cuando se muestran valores. El redondeo no se produce durante el procesamiento y la agregación de datos. Por ejemplo, el análisis de costos agrega costos como se indica a continuación:
  -    Cargo 1: 0,004 USD
  - Carga 2: 0,004 USD
  -    Costo agregado representado: 0,004 + 0,004 = 0,008. El cargo mostrado es 0,01 USD.
- API de consulta: los cargos se muestran con ocho posiciones decimales y no se realiza el redondeo.

## <a name="historical-data-might-not-match-invoice"></a>Los datos históricos pueden no coincidir con la factura

Es posible que los datos históricos de las ofertas basadas en crédito y pagadas por adelantado no coincidan con la factura. Algunas ofertas de pago por uso, MSDN y Visual Studio para Azure pueden tener créditos de Azure y pagos por adelantado aplicados en la factura. Sin embargo, los datos históricos que se muestran en Cost Management se basan solo en los cargos de consumo calculados. Los datos históricos de Cost Management no incluyen pagos ni créditos. Por tanto, es posible que los datos históricos que se muestran para estas ofertas no coincidan exactamente con la factura.

- Azure for Students (MS-AZR-0170P)
- Azure bajo licencia Open (MS-AZR-0111P)
- Pase para Azure (MS-AZR-0120P, MS-AZR-0123P, MS-AZR-0125P, MS-AZR-0128P, MS-AZR-0129P)
- Evaluación gratuita (MS-AZR-0044P)
- MSDN (MS-AZR-0062P)
- Visual Studio (MS-AZR-0029P, MS-AZR-0059P, MS-AZR-0060P, MS-AZR-0063P, MS-AZR-0064P)

## <a name="see-also"></a>Consulte también

- Si aún no ha completado la primera guía rápida de Cost Management, léala en [Start analyzing costs](../../cost-management/quick-acm-cost-analysis.md) (Comenzar a analizar los costos).
