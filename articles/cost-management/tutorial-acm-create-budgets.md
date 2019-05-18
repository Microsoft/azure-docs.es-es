---
title: 'Tutorial: Creación y administración de presupuestos de Azure | Microsoft Docs'
description: Este tutorial le ayuda a planear y tener en cuenta los costos de los servicios de Azure que usted consume.
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 05/14/2019
ms.topic: conceptual
ms.service: cost-management
manager: dougeby
ms.custom: seodec18
ms.openlocfilehash: eab45948b5f931377396d93d93e8955ba0f3e767
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792852"
---
# <a name="tutorial-create-and-manage-azure-budgets"></a>Tutorial: Creación y administración de presupuestos de Azure

Los presupuestos en Cost Management le ayudan a planear y dirigir la presentación de cuentas de la organización. Con presupuestos, puede tener en cuenta los servicios de Azure que consume o a los que se suscribe durante un período específico. Le ayudan a informar a otros usuarios sobre sus gastos a fin de administrar de manera proactiva los costos y supervisar cómo avanza el gasto a lo largo del tiempo. Cuando se superan los umbrales presupuestarios que ha creado, solo se desencadenan las notificaciones. Ninguno de los recursos se ve afectado y no se detiene el consumo. Puede usar los presupuestos para comparar y realizar un seguimiento de gastos para analizar los costos.

Los presupuestos mensuales se evalúan en función del gasto cada cuatro horas. Sin embargo, las notificaciones y datos de los recursos consumidos están disponibles pasadas ocho horas.  

Los presupuestos se restablecen automáticamente al final de un período (mensual, trimestral o anualmente) para el mismo importe presupuestario al seleccionar una fecha de expiración futura. Dado que se restablecen con el mismo importe presupuestario, deberá crear presupuestos independientes cuando los importes presupuestarios en moneda difieran para períodos futuros.

Los ejemplos de este tutorial le guiarán a través de la creación y edición de un presupuesto para una suscripción de Contrato Enterprise (EA) de Azure.

Inspección del [cómo crear un presupuesto para supervisar los gastos con Azure Cost Management](https://www.youtube.com/watch?v=ExIVG_Gr45A) vídeo para ver cómo puede crear los presupuestos de Azure para supervisar los gastos.


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un presupuesto en Azure Portal
> * Editar un presupuesto

## <a name="prerequisites"></a>Requisitos previos

Los presupuestos se admiten en varios tipos de cuenta de Azure. Para ver la lista completa de tipos de cuenta compatibles, consulte [Understand Cost Management data](understand-cost-mgt-data.md) (Información sobre los datos de Cost Management). Para ver los presupuestos, se necesita al menos acceso de lectura en la cuenta de Azure.

 En el caso de las suscripciones con contrato Enterprise de Azure, debe tener acceso de lectura para ver los presupuestos. Para crear y administrar presupuestos, debe tener permiso de colaborador. Puede crear presupuestos individuales para las suscripciones de EA y los grupos de recursos. Sin embargo, no se pueden crear presupuestos para cuentas de facturación de EA.

Los siguientes permisos de Azure, o ámbitos, se admiten por suscripción para los presupuestos por usuario y grupo. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

- Propietario: puede crear, modificar o eliminar los presupuestos para una suscripción.
- Colaborador y Colaborador de Cost Management: puede crear, modificar o eliminar sus propios presupuestos. Puede modificar el importe presupuestario para los presupuestos creados por otros usuarios.
- Lector y Lector de Cost Management: puede ver los presupuestos para los que tiene permiso.

Para más información sobre cómo asignar permisos a los datos de Cost Management, consulte [Asignación del acceso a los datos de Cost Management](assign-access-acm-data.md).

## <a name="sign-in-to-azure"></a>Iniciar sesión en Azure

- Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-budget-in-the-azure-portal"></a>Crear un presupuesto en Azure Portal

Puede crear un presupuesto de suscripción de Azure durante un período mensual, trimestral o anual. El contenido de navegación en el portal de Azure determina si crea un presupuesto para una suscripción o para un grupo de administración.

Para crear o ver un presupuesto, abra el ámbito deseado en Azure portal y seleccione **presupuestos** en el menú. Por ejemplo, vaya a **suscripciones**, seleccione una suscripción en la lista y, a continuación, seleccione **presupuestos** en el menú. Use la **ámbito** pastilla para cambiar a un ámbito diferente, como un grupo de administración, en los presupuestos. Para más información sobre los ámbitos, consulte [Descripción y uso de ámbitos](understand-work-scopes.md).

Después de crear los presupuestos, muestran a un lado una vista sencilla de su gasto actual.

Haga clic en **Agregar**.

![Presupuestos de Cost Management que se muestran en Azure Portal](./media/tutorial-acm-create-budgets/budgets01.png)

En la ventana **Crear presupuesto**, escriba un nombre de presupuesto y el importe presupuestario. A continuación, elija un período mensual, trimestral o anual. A continuación, seleccione una fecha de finalización. Los presupuestos requieren al menos un umbral de costos (% del presupuesto) y una dirección de correo electrónico correspondiente. De manera opcional, puede incluir hasta cinco umbrales y cinco direcciones de correo electrónico en un único presupuesto. Cuando se alcanza un umbral de presupuesto, las notificaciones por correo electrónico se reciben normalmente en menos de ocho horas. Para más información acerca de las notificaciones, consulte [Use cost alerts](cost-mgt-alerts-monitor-usage-spending.md) (Uso de alertas de costos).

Si tiene una suscripción de pago por uso, MSDN o Visual Studio, es posible que el período de facturación no esté alineado con el mes natural. Para esos tipos de suscripciones y grupos de recursos, puede crear un presupuesto que está alineado al período de facturación o meses naturales. Para crear un presupuesto alineado en el período de facturación, seleccione un período de restablecimiento de mes de facturación, facturación trimestre o año de facturación. Para crear un presupuesto alineado con el mes natural, seleccione un período de restablecimiento de mensual, trimestral o anualmente.

Este es un ejemplo de creación de un presupuesto mensual para 4500 USD. Se genera una alerta por correo electrónico cuando se alcanza el 90 % del presupuesto.

![Información de ejemplo que se muestra en el cuadro para crear un presupuesto](./media/tutorial-acm-create-budgets/monthly-budget01.png)

Cuando crea un presupuesto trimestral, funciona de la misma manera que un presupuesto mensual. La diferencia es que el importe presupuestario para el trimestre se divide de manera uniforme entre los tres meses del trimestre. Como cabría esperar, un importe presupuestario anual se divide de manera uniforme entre los 12 meses del año natural.

El gasto actual respecto a presupuestos se actualiza cada vez que Cost Management recibe datos actualizados de facturación. Normalmente, esto sucede a diario.

![Información de ejemplo que muestra el gasto actual con respecto a los presupuestos](./media/tutorial-acm-create-budgets/budgets-current-spending.png)

Después de crear un presupuesto, se muestra en el análisis de costos. Ver el presupuesto en relación con la tendencia del gasto es uno de los primeros pasos cuando empieza a [analizar los costos y los gastos](quick-acm-cost-analysis.md).

![Presupuesto y gasto de ejemplo que se muestra en el análisis de costos](./media/tutorial-acm-create-budgets/cost-analysis.png)

En el ejemplo anterior, creó un presupuesto para una suscripción. Sin embargo, también puede crear un presupuesto para un grupo de recursos. Si quiere crear un presupuesto para un grupo de recursos, vaya a **Administración de costos + facturación** &gt; **Suscripciones** &gt; seleccione una suscripción > **Grupos de recursos** > seleccione un grupo de recursos > **Presupuestos** > y, luego, **Agregar** un presupuesto.

## <a name="edit-a-budget"></a>Editar un presupuesto

Según el nivel de acceso que tenga, puede editar un presupuesto para cambiar sus propiedades. En el ejemplo siguiente, algunas de las propiedades son de solo lectura porque el usuario solo tiene permiso de colaborador en la suscripción. Actualmente, la **Fecha de expiración** está deshabilitada y no puede modificarse una vez establecida.

![Ejemplo de edición de un presupuesto para cambiar varias propiedades](./media/tutorial-acm-create-budgets/edit-budget.png)

## <a name="trigger-an-action-group"></a>Un grupo de acciones del desencadenador

Al crear o editar un presupuesto para una suscripción o el ámbito del grupo de recursos, puede configurar para llamar a un grupo de acciones. El grupo de acciones puede realizar una serie de acciones diferentes cuando se alcanza el umbral de presupuesto. Para obtener más información acerca de los grupos de acción, consulte [crear y administrar grupos de acciones en el portal de Azure](../azure-monitor/platform/action-groups.md). Para obtener más información sobre el uso de la automatización basada en el presupuesto con grupos de acciones, vea [administrar los costos con Azure presupuestos](../billing/billing-cost-management-budget-scenario.md).

Para crear o actualizar grupos de acciones, haga clic en **administrar grupos de acciones** mientras se crea o edita un presupuesto.

![Ejemplo de creación de un presupuesto para mostrar de la administración de grupos de acción](./media/tutorial-acm-create-budgets/manage-action-groups01.png)

A continuación, haga clic en **Agregar grupo de acciones** y crear el grupo de acciones.


![Imagen del cuadro de grupo de acción de agregar](./media/tutorial-acm-create-budgets/manage-action-groups02.png)

Después de la acción se crea el grupo, cierre el cuadro para volver a su presupuesto.

Configure su presupuesto para usar el grupo de acciones cuando se alcanza un umbral individual. Se admiten hasta cinco diferentes umbrales.

![Ejemplo que muestra la selección de grupos de acción para una condición de alerta](./media/tutorial-acm-create-budgets/manage-action-groups03.png)

El ejemplo siguiente muestra los umbrales de presupuesto establecidos en 50%, 75% y 100%. Cada uno está configurado para desencadenar las acciones especificadas en el grupo de acciones designadas.

![Ejemplo que muestra las condiciones de alerta configuradas con varios grupos de acciones y los tipos de acciones](./media/tutorial-acm-create-budgets/manage-action-groups04.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió lo siguiente:

> [!div class="checklist"]
> * Crear un presupuesto en Azure Portal
> * Editar un presupuesto

Pase al siguiente tutorial para crear una exportación periódica de los datos de administración de costos.

> [!div class="nextstepaction"]
> [Creación y administración de datos exportados](tutorial-export-acm-data.md)
