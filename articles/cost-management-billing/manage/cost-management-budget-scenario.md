---
title: Escenario de presupuesto de la administración de los costos y la facturación de Azure
description: Obtenga información sobre cómo usar la automatización de Azure para apagar las máquinas virtuales según umbrales específicos de presupuesto.
author: bandersmsft
ms.reviewer: adwise
tags: billing
ms.service: cost-management-billing
ms.subservice: cost-management
ms.topic: how-to
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 7bed8cc55e0880d88df22ca32bc5886e22022cbc
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690194"
---
# <a name="manage-costs-with-azure-budgets"></a>Administración de costos con Azure Budgets

El control de costos es un componente esencial para maximizar el valor de su inversión en la nube. Hay varios escenarios donde la orquestación en función del costo, los informes y la visibilidad del costo resultan fundamentales para la continuidad de las operaciones empresariales. [Las API de Azure Cost Management](https://docs.microsoft.com/rest/api/consumption/) proporcionan un conjunto de API para admitir cada uno de estos escenarios. Las API proporcionan detalles de uso, lo que le permite ver los costos a un nivel de instancia pormenorizado.

Los presupuestos normalmente se usan como parte del control de costos. Se pueden seguir en Azure. Por ejemplo, podría restringir la vista de presupuesto en función de una colección de recursos, grupos de recursos o una suscripción. Además de utilizar la API de presupuestos para recibir notificaciones por correo electrónico cuando se alcanza un umbral de presupuesto, puede usar [grupos de acciones de Azure Monitor](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups) para desencadenar un conjunto organizado de acciones que resultan en un evento de presupuesto.

Un escenario habitual en el que un cliente que ejecuta una carga de trabajo que no resulta fundamental tiene en cuenta los presupuestos podría ser cuando se desea llevar a cabo la administración en función de uno concreto y también no superar un costo predecible al recibir la factura mensual. Este escenario requiere cierta organización en función del costo de los recursos que forman parte del entorno de Azure. En este escenario, se establece un presupuesto mensual de 1000 USD para la suscripción. Además, se establecen umbrales de notificación para desencadenar algunas orquestaciones. Este escenario comienza con un umbral de costo del 80 %, que, si se supera, implicará detener todas las máquinas virtuales en el grupo de recursos **Opcional**. A continuación, en el umbral de costo del 100 %, se detendrán todas las instancias de máquina virtual.

Para configurar este escenario, completará las siguientes acciones mediante los pasos proporcionados en cada sección de este tutorial.

Las acciones incluidas en este tutorial le permiten:

- Crear un Runbook de Azure Automation para detener las máquinas virtuales mediante el uso de webhooks.
- Crear una instancia de Azure Logic App para que se desencadene en función del valor de umbral de presupuesto y llame al Runbook con los parámetros correctos.
- Crear un grupo de acciones de Azure Monitor que se configurarán para desencadenar la instancia de Azure Logic App cuando se alcance el umbral de presupuesto.
- Crear el presupuesto de Azure con los umbrales deseados y conectarlo al grupo de acción.

## <a name="create-an-azure-automation-runbook"></a>Creación de un Runbook de Azure Automation

[Azure Automation](https://docs.microsoft.com/azure/automation/automation-intro) es un servicio que permite crear scripts para la mayoría de las tareas de administración de recursos y ejecutar esas tareas de forma programada o a petición. Como parte de este escenario, creará un [Runbook de Azure Automation](https://docs.microsoft.com/azure/automation/automation-runbook-types) que se usará para detener las máquinas virtuales. Usará el runbook gráfico [Stop Azure V2 VMs](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) de la [galería](https://docs.microsoft.com/azure/automation/automation-runbook-gallery) para crear este escenario. Al importar este runbook en su cuenta de Azure y publicarlo, podrá detener las máquinas virtuales cuando se alcance un umbral de presupuesto.

### <a name="create-an-azure-automation-account"></a>Creación de una cuenta de Azure Automation

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con sus credenciales de su cuenta de Azure.
2. Seleccione el botón **Crear un recurso** de la esquina superior izquierda de Azure.
3. Seleccione **Herramientas de administración** > **Automatización**.
   > [!NOTE]
   > Si no tiene una cuenta de Azure aún, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
4. Escriba la información de la cuenta. En **Crear cuenta de ejecución de Azure**, elija **Sí** para habilitar automáticamente la configuración necesaria a fin de simplificar la autenticación en Azure.
5. Cuando haya terminado, seleccione **Crear** para iniciar la implementación de la cuenta de Automation.

### <a name="import-the-stop-azure-v2-vms-runbook"></a>Importación del Runbook Stop Azure V2 VMs

Mediante un [Runbook de Azure Automation](https://docs.microsoft.com/azure/automation/automation-runbook-types), importe el Runbook gráfico [Stop Azure V2 VMs](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) desde la galería.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con sus credenciales de su cuenta de Azure.
1. Abra su cuenta de Automation seleccionando **Todos los servicios** > **Cuentas de Automation**. Después, seleccione su cuenta de Automation.
1. Seleccione **Galería de Runbooks** desde la sección **Automatización de procesos**.
1. Establezca el **Origen de la Galería** en **Centro de scripts** y seleccione **Aceptar**.
1. Busque y seleccione el elemento de la galería [Stop Azure V2 Vms](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) dentro de Azure Portal.
1. Seleccione **Importar** para mostrar el área **Importar** y seleccione **Aceptar**. Se mostrará la hoja de información general del runbook.
1. Una vez que el Runbook haya completado el proceso de importación, seleccione **Editar** para mostrar la opción de publicación y el editor del Runbook gráfico.  
    ![Azure: edición de runbook gráfico](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-01.png)
1. Seleccione **Publicar** para publicar el runbook y **Sí** cuando se le solicite. Al publicar un Runbook, reemplaza la versión publicada existente con la versión del borrador. En este caso, no tiene una versión publicada porque ha creado el runbook.
    Para más información sobre cómo publicar un Runbook, consulte [Mi primer Runbook gráfico](https://docs.microsoft.com/azure/automation/automation-first-runbook-graphical).

## <a name="create-webhooks-for-the-runbook"></a>Creación de webhooks para el Runbook

Mediante el runbook gráfico [Stop Azure V2 VMs](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b), creará dos webhooks para iniciar el runbook en Azure Automation mediante una sola solicitud HTTP. El primer webhook invoca el runbook en un umbral de presupuesto de 80 % con el nombre del grupo de recursos como un parámetro, lo que permite que las máquinas virtuales opcionales se detengan. A continuación, el segundo webhook invoca el runbook sin parámetros (al 100 %), lo que detiene las restantes instancias de máquina virtual.

1. En la página **Runbooks** de [Azure Portal](https://portal.azure.com/), seleccione el runbook **StopAzureV2Vm** que muestra el área de información general del runbook.
1. Haga clic en **Webhook** en la parte superior de la página para abrir el área **Agregar webhook**.
1. Haga clic en **Crear nuevo Webhook** para abrir la hoja **Crear un nuevo Webhook**.
1. Establezca el **nombre** del webhook en **Opcional**. El valor de la propiedad **Habilitada** debe ser **Sí**. No es necesario cambiar el valor de **Expira**. Para más información sobre las propiedades del webhook, consulte [Propiedades de un webhook](../../automation/automation-webhooks.md#webhook-properties).
1. Junto al valor de la dirección URL, seleccione el icono de copiar para copiar la dirección URL del webhook.
   > [!IMPORTANT]
   > Guarde la dirección URL del webhook denominado **Opcional** en un lugar seguro. La usará más adelante en este tutorial. Po motivos de seguridad, una vez creado el webhook, no se puede ver ni recuperar de nuevo la dirección URL.
1. Seleccione **Aceptar** para crear el nuevo webhook.
1. Seleccione **Configurar parámetros y ejecutar configuraciones** para ver los valores de los parámetros para el runbook.
   > [!NOTE]
   > Si el runbook tiene parámetros obligatorios, no puede crear el webhook, salvo que se especifiquen los valores.
1. Seleccione **Aceptar** para aceptar los valores de los parámetros del webhook.
1. Seleccione **Crear** para crear el webhook.
1. A continuación, siga los pasos anteriores para crear un segundo webhook denominado **Completo**.
    > [!IMPORTANT]
    > No olvide guardar ambas direcciones URL de webhook para usarlas más adelante en este tutorial. Po motivos de seguridad, una vez creado el webhook, no se puede ver ni recuperar de nuevo la dirección URL.

Ahora debería tener dos webhooks configurados que están disponibles con las direcciones URL que ha guardado.

![Webhooks: Opcional y Completo](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-02.png)

Ahora ha terminado con el programa de instalación de Azure Automation. Puede probar los webhooks con una prueba sencilla de Postman para asegurarse de que el webhook funciona. A continuación, debe crear la aplicación lógica para la orquestación.

## <a name="create-an-azure-logic-app-for-orchestration"></a>Crear una aplicación Azure Logic App para la orquestación

Logic Apps ayuda a crear, programar y automatizar procesos como flujos de trabajo, permitiéndole integrar aplicaciones, datos, sistemas y servicios en empresas u organizaciones. En este escenario, la [aplicación lógica](https://docs.microsoft.com/azure/logic-apps/) que crea hace más que simplemente llamar al webhook de automatización que creó.

Los presupuestos se pueden configurar para desencadenar una notificación cuando se alcanza un umbral especificado. Puede proporcionar varios umbrales en los que recibir una notificación y la aplicación lógica demostrará la capacidad de realizar distintas acciones según el umbral superado. En este ejemplo, configurará un escenario donde obtendrá un par de notificaciones: la primera, cuando se alcanza el 80 % del presupuesto y la segunda, en el 100 %. La aplicación lógica se usará para apagar todas las máquinas virtuales en el grupo de recursos. En primer lugar, al umbral **Opcional** se llega en el 80 %; entonces, el segundo umbral se alcanzará cuando todas las máquinas virtuales en la suscripción se cierren.

Las aplicaciones lógicas le permiten proporcionar un esquema de ejemplo para el desencadenador HTTP, pero requieren que se defina el encabezado **Content-Type**. Dado que el grupo de acciones no tiene encabezados personalizados para el webhook, debe analizar la carga en un paso independiente. Usará la acción **Analizar** y le proporcionará una carga de ejemplo.

### <a name="create-the-logic-app"></a>Creación de la aplicación lógica

La aplicación lógica llevará a cabo varias acciones. En la lista siguiente se proporciona un conjunto de acciones que la aplicación lógica llevará a cabo:

- Reconoce cuándo se recibe una solicitud HTTP
- Se analizan los datos JSON pasados para determinar el valor de umbral que se ha alcanzado
- Use una instrucción condicional para comprobar si la cantidad del umbral alcanzó el 80 % o más del intervalo de presupuesto, pero no un valor mayor o igual que el 100 %.
  - Si se ha alcanzado la cantidad de este umbral, envíe una solicitud HTTP POST con el webhook denominado **Opcional**. Esta acción apagará las máquinas virtuales en el grupo "Opcional".
- Use una instrucción condicional para comprobar si la cantidad de umbral ha alcanzado o superado el 100 % del valor de presupuesto.
  - Si se ha alcanzado la cantidad de este umbral, envíe una solicitud HTTP POST con el webhook denominado **Completo**. Así se apagará el resto de máquinas virtuales.

Los pasos siguientes son necesarios para crear la aplicación lógica que llevará a cabo los pasos anteriores:

1. En [Azure Portal](https://portal.azure.com/), seleccione **Crear un recurso** > **Integración** > **Logic App**.  
    ![Azure: selección del recurso de aplicación lógica](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-03.png)
1. En el área **Crear aplicación lógica**, proporcione los detalles que necesita para crear la aplicación lógica, seleccione **Anclar al panel** y seleccione **Crear**.  
    ![Azure: creación de una aplicación lógica](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-03a.png)

Una vez que Azure ha implementado la aplicación lógica, el **Diseñador de aplicaciones lógicas** se abre y muestra un área con un vídeo de introducción y desencadenadores utilizados frecuentemente.

### <a name="add-a-trigger"></a>Incorporación de un desencadenador

Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica que inicia y ejecuta el flujo de trabajo. Las acciones son todos los pasos que se producen después del desencadenador.

1. En **Plantillas** del área **Diseñador de aplicaciones lógicas**, elija **Aplicación lógica en blanco**.
1. Agregue un [desencadenador](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview#logic-app-concepts); para ello, escriba "solicitud http" en el cuadro de búsqueda del **Diseñador de aplicaciones lógicas** para buscar y seleccionar el desencadenador denominado **Solicitud: cuando se recibe una solicitud HTTP**.  
    ![Azure: aplicación lógica, desencadenador HTTP](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-04.png)
1. Seleccione **Nuevo paso** > **Agregar una acción**.  
    ![Azure: nuevo paso, agregar una acción](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-05.png)
1. Busque "análisis del archivo JSON" en el cuadro de búsqueda del **Diseñador de aplicaciones lógicas** para buscar y seleccionar la [acción](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview#logic-app-concepts) **Operaciones de datos: análisis del archivo JSON**.  
    ![Azure: aplicación lógica, agregar la acción de análisis del archivo JSON](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-06.png)
1. Escriba "Payload" (carga) como el nombre de **contenido** para la carga de análisis del archivo JSON o use la etiqueta "Body" (cuerpo) del contenido dinámico.
1. Seleccione la opción **Use sample payload to generate schema** (Usar una carga de ejemplo para generar el esquema) en el cuadro **Análisis del archivo JSON**.  
    ![Azure: aplicación lógica, usar datos de JSON de ejemplo para generar el esquema](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-07.png)
1. Pegue la siguiente carga de ejemplo de archivo JSON en el cuadro de texto: `{"schemaId":"AIP Budget Notification","data":{"SubscriptionName":"CCM - Microsoft Azure Enterprise - 1","SubscriptionId":"<GUID>","SpendingAmount":"100","BudgetStartDate":"6/1/2018","Budget":"50","Unit":"USD","BudgetCreator":"email@contoso.com","BudgetName":"BudgetName","BudgetType":"Cost","ResourceGroup":"","NotificationThresholdAmount":"0.8"}}`
   El cuadro de texto se mostrará como:  
    ![Azure: aplicación lógica, carga de archivo JSON de ejemplo](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-08.png)
1. Seleccione **Listo**.

### <a name="add-the-first-conditional-action"></a>Incorporación de la primera acción condicional

Use una instrucción condicional para comprobar si la cantidad del umbral alcanzó el 80 % o más del intervalo de presupuesto, pero no un valor mayor o igual que el 100 %. Si se ha alcanzado la cantidad de este umbral, envíe una solicitud HTTP POST con el webhook denominado **Opcional**. Esta acción apagará las máquinas virtuales en el grupo **Opcional**.

1. Seleccione **Nuevo paso** > **Agregar una condición**.  
    ![Azure: aplicación lógica, agregar una condición](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-09.png)
1. En el cuadro **Condición**, seleccione el cuadro de texto que contiene `Choose a value` para mostrar una lista de los valores disponibles.  
    ![Azure, aplicación lógica, cuadro Condición](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-10.png)
1. Seleccione **Expresión** en la parte superior de la lista y escriba la siguiente expresión en el editor de expresiones: `float()`.  
    ![Azure: aplicación lógica, expresión de punto flotante](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-11.png)
1. Seleccione **Contenido dinámico**, coloque el cursor dentro del paréntesis () y seleccione **NotificationThresholdAmount** en la lista para rellenar la expresión completa.
   La expresión será:<br>
    `float(body('Parse_JSON')?['data']?['NotificationThresholdAmount'])`
1. Seleccione **Aceptar** para establecer la expresión.
1. Seleccione **es mayor o igual que** en el cuadro de lista desplegable de la **Condición**.
1. En el cuadro **Elegir un valor** de la condición, escriba `.8`.  
    ![Azure : aplicación lógica, expresión de punto flotante con un valor](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-12.png)
1. Seleccione **Agregar** > **Agregar fila** dentro del cuadro Condición, para agregar una parte adicional de la condición.
1. En **Condición**, seleccione el cuadro de texto que contiene `Choose a value`.
1. Seleccione **Expresión** en la parte superior de la lista y escriba la siguiente expresión en el editor de expresiones: `float()`
1. Seleccione **Contenido dinámico**, coloque el cursor dentro del paréntesis () y seleccione **NotificationThresholdAmount** en la lista para rellenar la expresión completa.
1. Seleccione **Aceptar** para establecer la expresión.
1. Seleccione **es menor que** en el cuadro de lista desplegable de la **Condición**.
1. En el cuadro **Elegir un valor** de la condición, escriba `1`.  
    ![Azure: aplicación lógica, expresión de punto flotante con un valor](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-13.png)
1. En el cuadro **If true** (si es verdad), seleccione **Agregar una acción**. Agregará una acción HTTP POST que apagará las máquinas virtuales opcionales.  
    ![Azure: aplicación lógica, agregar una acción](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-14.png)
1. Escriba **HTTP** para buscar la acción HTTP y seleccione la acción **HTTP-HTTP**.  
    ![Azure: aplicación lógica, agregar una acción HTTP](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-15.png)
1. Seleccione **Post** como valor de **Método**.
1. Escriba la dirección URL del webhook denominado **Opcional** que creó anteriormente en este tutorial como el valor de **Uri**.  
    ![Azure: aplicación lógica, URI de una acción HTTP](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-16.png)
1. Seleccione **Agregar una acción** en el cuadro **If true**. Agregará una acción de correo electrónico que enviará un correo electrónico notificando al destinatario que las máquinas virtuales opcionales se han apagado.
1. Busque "enviar correo electrónico" y seleccione una acción *enviar correo electrónico* en función del servicio de correo electrónico que use.  
    ![Azure: aplicación lógica, acción de envío de correo electrónico](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-17.png)

    Para las cuentas de Microsoft personales, seleccione **Outlook.com**. Para las cuentas profesionales o educativas de Azure, seleccione **Office 365 Outlook**. Si no tiene ya una conexión, se le solicita que inicie sesión en su cuenta de correo electrónico. Logic Apps crea una conexión a la cuenta de correo electrónico.
   Deberá permitir que la aplicación lógica acceda a la información de correo electrónico.  
    ![Azure: aplicación lógica, aviso de acceso](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-18.png)
1. Agregue el texto de **Para**, **Asunto** y **Cuerpo** para el correo electrónico que informa al destinatario que las máquinas virtuales opcionales se han apagado. Use el contenido dinámico de **BudgetName** y **NotificationThresholdAmount** para rellenar los campos de asunto y cuerpo. 
    ![Azure: aplicación lógica, detalles de correo electrónico](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-19.png)

### <a name="add-the-second-conditional-action"></a>Incorporación de la segunda acción condicional

Use una instrucción condicional para comprobar si la cantidad de umbral ha alcanzado o superado el 100 % del valor de presupuesto. Si se ha alcanzado la cantidad de este umbral, envíe una solicitud HTTP POST con el webhook denominado **Completo**. Así se apagará el resto de máquinas virtuales.

1. Seleccione **Nuevo paso** > **Agregar una condición**.  
    ![Azure: aplicación lógica, agregar acción](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-20.png)
1. En el cuadro **Condición**, seleccione el cuadro de texto que contiene `Choose a value` para mostrar una lista de los valores disponibles.
1. Seleccione **Expresión** en la parte superior de la lista y escriba la siguiente expresión en el editor de expresiones: `float()`
1. Seleccione **Contenido dinámico**, coloque el cursor dentro del paréntesis () y seleccione **NotificationThresholdAmount** en la lista para rellenar la expresión completa.
   La expresión será similar a la siguiente:<br>
    `float(body('Parse_JSON')?['data']?['NotificationThresholdAmount'])`
1. Seleccione **Aceptar** para establecer la expresión.
1. Seleccione **es mayor o igual que** en el cuadro de lista desplegable de la **Condición**.
1. En el cuadro **Elegir un valor** de la condición, escriba `1`.  
    ![Azure: aplicación lógica, establecer valor de condición](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-21.png)
1. En el cuadro **If true** (si es verdad), seleccione **Agregar una acción**. Agregará una acción HTTP POST que apagará el resto de máquinas virtuales.  
    ![Azure: aplicación lógica, agregar una acción](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-22.png)
1. Escriba **HTTP** para buscar la acción HTTP y seleccione la acción **HTTP-HTTP**.
1. Seleccione **Post** como valor de **Método**.
1. Escriba la dirección URL del webhook denominado **Completo** que creó anteriormente en este tutorial como el valor de **Uri**.  
    ![Azure: aplicación lógica, agregar una acción](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-23.png)
1. Seleccione **Agregar una acción** en el cuadro **If true**. Agregará una acción de correo electrónico que enviará un correo electrónico notificando al destinatario que el resto de máquinas virtuales se han apagado.
1. Busque "enviar correo electrónico" y seleccione una acción *enviar correo electrónico* en función del servicio de correo electrónico que use.
1. Agregue el texto de **Para**, **Asunto** y **Cuerpo** para el correo electrónico que informa al destinatario que las máquinas virtuales opcionales se han apagado. Use el contenido dinámico de **BudgetName** y **NotificationThresholdAmount** para rellenar los campos de asunto y cuerpo.  
    ![Azure: aplicación lógica, envío de detalles por correo electrónico](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-24.png)
1. Seleccione **Guardar** en la parte superior del área **Diseñador de aplicaciones lógicas**.

### <a name="logic-app-summary"></a>Resumen de aplicación lógica

Aquí se muestra la apariencia que tendrá la aplicación lógica una vez que haya terminado. En el escenario más básico donde no necesita ninguna orquestación basada en el umbral, podría llamar directamente al script de automatización desde **Monitor** y omitir el paso de **Aplicación lógica**.

![Azure - Aplicación lógica: vista completa](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-25.png)

Cuando guardó la aplicación lógica, se generó una dirección URL a la que podrá llamar. La usará en la sección siguiente de este tutorial.

## <a name="create-an-azure-monitor-action-group"></a>Creación de un grupo de acciones de Azure Monitor

Un grupo de acciones es una colección de las preferencias de notificación que el usuario define. Cuando se desencadena una alerta, un grupo de acciones específico puede recibir la alerta si se le notifica. Una alerta de Azure genera una notificación de forma proactiva según condiciones específicas y ofrece la oportunidad de tomar medidas. Una alerta puede usar datos de varios orígenes, como las métricas y los registros.

Los grupos de acciones son el punto de conexión única que se integrará con su presupuesto. Puede configurar las notificaciones en una serie de canales, pero en este escenario se centrará en la aplicación lógica que creó anteriormente en este tutorial.

### <a name="create-an-action-group-in-azure-monitor"></a>Creación de un grupo de acciones en Azure Monitor

Cuando cree el grupo de acciones, apuntará a la aplicación lógica que creó anteriormente en este tutorial.

1. Si aún no ha iniciado sesión en [Azure Portal](https://portal.azure.com/), hágalo y seleccione **Todos los servicios** > **Monitor**.
1. Seleccione **Alertas** y **Administrar acciones**.
1. Seleccione **Agregar un grupo de acciones** desde el área **Grupos de acciones**.
1. Agregue y compruebe los siguientes elementos:
    - Nombre del grupo de acciones
    - Nombre corto
    - Subscription
    - Resource group  
    ![Azure - Aplicación lógica: agregar un grupo de acciones](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-26.png)
1. Dentro del panel **Agregar grupo de acciones**, agregue una acción LogicApp. Denomine la acción **Budget-BudgetLA**. En el panel **Aplicación lógica**, seleccione la **Suscripción** y **Grupo de recursos**. A continuación, seleccione la **Aplicación lógica** que creó anteriormente en este tutorial.
1. Seleccione **Aceptar** para establecer la aplicación lógica. Después, seleccione **Aceptar** en el panel **Agregar grupo de acciones** para crear el grupo de acciones.

Ya ha terminado con todos los componentes complementarios necesarios para organizar eficazmente su presupuesto. Ahora todo lo que tienen que hacer es crear el presupuesto y configurarlo para usar el grupo de acciones que ha creado.

## <a name="create-the-azure-budget"></a>Creación del presupuesto de Azure

Puede crear un presupuesto en Azure Portal mediante el [característica Presupuesto](../costs/tutorial-acm-create-budgets.md) en Cost Management. O bien, puede crear un presupuesto con las API REST, los cmdlets de Powershell o la CLI. El siguiente procedimiento usa la API REST. Antes de llamar a la API REST, necesitará un token de autorización. Para crear un token de autorización, puede usar el proyecto [ARMClient](https://github.com/projectkudu/ARMClient). **ARMClient** le permite autenticarse en Azure Resource Manager y obtener un token para llamar a las API.

### <a name="create-an-authentication-token"></a>Creación de un token de autenticación

1. Vaya al proyecto [ARMClient](https://github.com/projectkudu/ARMClient) en GitHub.
1. Clone el repositorio para obtener una copia local.
1. Abra el proyecto en Visual Studio y compílelo.
1. Una vez que la compilación es correcta, el archivo ejecutable debe estar en la carpeta *\bin\debug*.
1. Ejecute el proyecto ARMClient. Abra un símbolo del sistema y vaya a la carpet *\bin\debug* desde la raíz del proyecto.
1. Para iniciar sesión y llevar a cabo la autenticación, escriba el siguiente comando en el símbolo del sistema:<br>
    `ARMClient login prod`
1. Copie el **GUID de suscripción** desde la salida.
1. Para copiar un token de autorización en el Portapapeles, escriba el siguiente comando en el símbolo del sistema, pero asegúrese de usar el identificador de suscripción copiado en el paso anterior: <br>
    `ARMClient token <subscription GUID from previous step>`

    Una vez haya completado el paso anterior, verá lo siguiente:<br>
    **El token se copió correctamente en el Portapapeles.**
1. Guarde el token que se usará para los pasos descritos en la siguiente sección de este tutorial.

### <a name="create-the-budget"></a>Creación del presupuesto

Después, configurará **Postman** para crear un presupuesto mediante una llamada a las API REST de consumo de Azure. Postman es un entorno de desarrollo de API. Importará los archivos de entorno y colección en Postman. La colección contiene definiciones agrupadas de solicitudes HTTP que llaman a las API REST de consumo de Azure. El archivo de entorno contiene variables que la colección usa.

1. Descargue el [cliente de REST de Postman](https://www.getpostman.com/) y ábralo para ejecutar las API REST.
1. En Postman, cree una nueva solicitud.  
    ![Postman: crear una nueva solicitud](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-27.png)
1. Guarde la nueva solicitud como una colección, de modo que la nueva solicitud no tenga nada en ella.  
    ![Postman: guardar la nueva solicitud](./media/cost-management-budget-scenario/billing-cost-management-budget-scenario-28.png)
1. Cambie la solicitud desde `Get` a una acción `Put`.
1. Modifique la dirección URL siguiente reemplazando `{subscriptionId}` con el **Id. de suscripción** que usó en la sección anterior de este tutorial. Además, modifique la dirección URL para incluir "SampleBudget" como el valor de `{budgetName}`: `https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/budgets/{budgetName}?api-version=2018-03-31`
1. Seleccione la pestaña **Encabezados** dentro de Postman.
1. Agregue una nueva **Clave** llamada "Autorización".
1. Establezca el **Valor** en el token que se creó mediante ArmClient al final de la última sección.
1. Seleccione la pestaña **Cuerpo** dentro de Postman.
1. Seleccione el botón de opción **raw**.
1. En el cuadro de texto, pegue la definición del presupuesto de ejemplo siguiente; sin embargo, debe reemplazar los parámetros `subscriptionID`, `resourcegroupname` y `actiongroupname` por el identificación de la suscripción, un nombre único para su grupo de recursos y el nombre del grupo de acciones que creó tanto en la dirección URL como en el cuerpo de la solicitud:

    ```
        {
            "properties": {
                "category": "Cost",
                "amount": 100.00,
                "timeGrain": "Monthly",
                "timePeriod": {
                "startDate": "2018-06-01T00:00:00Z",
                "endDate": "2018-10-31T00:00:00Z"
                },
                "filters": {},
            "notifications": {
                "Actual_GreaterThan_80_Percent": {
                    "enabled": true,
                    "operator": "GreaterThan",
                    "threshold": 80,
                    "contactEmails": [],
                    "contactRoles": [],
                    "contactGroups": [
                        "/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/microsoft.insights/actionGroups/{actiongroupname}"
                    ]
                },
               "Actual_EqualTo_100_Percent": {
                       "operator": "EqualTo",
                       "threshold": 100,
                       "contactGroups": [
                           "/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/microsoft.insights/actionGroups/{actiongroupname}"
                       ]
                   }
               }
            }
        }
    ```
1. Presione **Enviar** para enviar la solicitud.

Ahora tiene todas las piezas que necesita para llamar a la [API de presupuestos](https://docs.microsoft.com/rest/api/consumption/budgets). La referencia de la API de presupuestos contiene detalles adicionales sobre solicitudes específicas, incluidas las siguientes:

- **budgetName**: se admiten varios presupuestos.  Los nombres de presupuesto deben ser únicos.
- **category**: debe ser **Cost** (Costo) o **Usage** (Uso). La API admite tanto los presupuestos de costo como los de uso.
- **timeGrain**: un presupuesto mensual, trimestral o anual. La cantidad se restablece al final del período.
- **filters**: los filtros permiten restringir el presupuesto a un conjunto específico de recursos dentro del ámbito seleccionado. Por ejemplo, un filtro podría ser una colección de grupos de recursos para un presupuesto de nivel de suscripción.
- **notifications**: determina los detalles de la notificación y los umbrales. Puede configurar varios umbrales y proporcionar una dirección de correo electrónico o un grupo de acciones para recibir una notificación.

## <a name="summary"></a>Resumen

En este tutorial, ha aprendido:

- Cómo crear un Runbook de Azure Automation para detener máquinas virtuales.
- Cómo crear una instancia de Azure Logic App que se desencadene en función de valores de umbral de presupuesto y llame al Runbook con los parámetros correctos.
- Cómo crear un grupo de acciones de Azure Monitor que se ha configurado para desencadenar la instancia de Azure Logic App cuando se alcance el umbral de presupuesto.
- Cómo crear el presupuesto de Azure con los umbrales deseados y conectarlo al grupo de acciones.

Ahora tiene un presupuesto completamente funcional para la suscripción que apagará las máquinas virtuales cuando se alcancen los umbrales de presupuesto configurados.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de los escenarios de facturación de Azure, consulte [Escenarios de automatización de administración de costos y facturación](cost-management-automation-scenarios.md).
