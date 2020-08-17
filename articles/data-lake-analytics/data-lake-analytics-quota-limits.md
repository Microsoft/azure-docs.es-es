---
title: Ajustar las cuotas y los límites en Azure Data Lake Analytics
description: Obtenga información sobre cómo ajustar y aumentar las cuotas y los límites en cuentas de Azure Data Lake Analytics (ADLA).
services: data-lake-analytics
ms.service: data-lake-analytics
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.assetid: 49416f38-fcc7-476f-a55e-d67f3f9c1d34
ms.topic: how-to
ms.date: 03/15/2018
ms.openlocfilehash: 0025e35f516543c8fe703daa647ca29ed3fb87e6
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87127594"
---
# <a name="adjust-quotas-and-limits-in-azure-data-lake-analytics"></a>Ajustar las cuotas y los límites en Azure Data Lake Analytics

Obtenga información sobre cómo ajustar y aumentar la cuota y los límites en cuentas de Azure Data Lake Analytics (ADLA). Conocer estos límites le ayudará a comprender el comportamiento de los trabajos de U-SQL. Todos los límites de cuota son flexibles, y siempre se puede poner en contacto con el servicio de soporte técnico de Azure para aumentar los límites máximos.

## <a name="azure-subscriptions-limits"></a>Límites de las suscripciones de Azure

**Número máximo de cuentas de ADLA por suscripción y región:**  5

Al intentar crear la sexta cuenta de ADLA, se muestra el error "Ha alcanzado el número máximo de cuentas de Data Lake Analytics permitido (5) en {región} con la suscripción {nombre}".

Si desea superar este límite, puede probar estas opciones:
* Si es posible, seleccione otra región.
* Póngase en contacto con el servicio de soporte técnico de Azure y [abra una incidencia de soporte](#increase-maximum-quota-limits) para solicitar un aumento de la cuota.

## <a name="default-adla-account-limits"></a>Límites predeterminados de cuentas de ADLA

**Número máximo de unidades de análisis (UA) por cuenta:** 250, predeterminado 32

Es el número máximo de unidades de análisis que se pueden ejecutar de forma simultánea en su cuenta. Si el total de unidades de análisis en ejecución entre todos los trabajos supera este límite, los trabajos nuevos se ponen en cola automáticamente. Por ejemplo:

* Si solo tiene un trabajo en ejecución con 32 unidades de análisis, cuando envíe un segundo trabajo, este permanecerá en la cola de trabajos hasta que el primero se complete.
* Si ya tiene cuatro trabajos en ejecución y cada uno usa ocho unidades de análisis, al enviar un quinto trabajo que necesita ocho unidades de análisis, este permanecerá en la cola de trabajos hasta que haya ocho unidades de análisis disponibles.

    ![Página de límites y cuotas de Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/adjust-quota-limits.png)

**Número máximo de unidades de análisis (UA) por trabajo:** 250, predeterminado 32

Este es el número máximo de unidades de análisis que se puede asignar a cada trabajo individual en la cuenta. Los trabajos a los que se asigne más de este límite se rechazarán, a menos que el remitente se vea afectado por una directiva de proceso (límite de envío de trabajos) que le otorgue más unidades de análisis por trabajo. El límite superior de este valor es el límite de unidades de análisis de la cuenta.

**Número máximo de trabajos de U-SQL simultáneos por cuenta:** 20

Es el número máximo de trabajos que se pueden ejecutar de forma simultánea en su cuenta. Por encima de este valor, los últimos trabajos permanecen en la cola automáticamente.

## <a name="adjust-adla-account-limits"></a>Ajuste de los límites de cuentas de ADLA

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Elija una cuenta de ADLA existente.
3. Haga clic en **Propiedades**.
4. Ajuste los valores de **Número de AU máximo**, **Número máximo de trabajos en ejecución** y **Límites del envío de trabajos** para satisfacer sus necesidades.

## <a name="increase-maximum-quota-limits"></a>Aumento de los límites de cuota máximos

Puede encontrar más información sobre los límites de Azure en la [documentación sobre los límites específicos de los servicios de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#data-lake-analytics-limits).

1. Cree una solicitud de soporte técnico en Azure Portal.

    ![Página del portal de Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-help-support.png)

    ![Página del portal de Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request.png)
2. Seleccione el tipo de problema **Cuota**.
3. Seleccione su **Suscripción** (asegúrese de que no sea una suscripción de "prueba").
4. Seleccione el tipo de cuota **Data Lake Analytics**.

    ![Página del portal de Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request-basics.png)

5. En la página del problema, explique el límite de aumento que quiere pedir y los **detalles** de por qué necesita esta capacidad adicional.

    ![Página del portal de Azure Data Lake Analytics](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request-details.png)

6. Verifique su información de contacto y cree la petición de soporte técnico.

Microsoft revisa la solicitud y se intenta dar cabida a sus necesidades empresariales tan pronto como sea posible.

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Administración de Análisis de Azure Data Lake mediante Azure Powershell](data-lake-analytics-manage-use-powershell.md)
* [Supervisión y solución de problemas de trabajos de Azure Data Lake Analytics con Azure Portal](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
