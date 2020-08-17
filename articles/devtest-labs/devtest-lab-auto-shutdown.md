---
title: Administración de directivas de apagado automático en Azure DevTest Labs | Microsoft Docs
description: Aprenda a establecer una directiva de apagado automático para un laboratorio, y que así que las máquinas virtuales se apaguen automáticamente cuando no se estén usando.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a30070470f9a75ec5c56d448cd09ca82dd0cbce7
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287546"
---
# <a name="configure-autoshutdown-for-lab-and-compute-virtual-machines-in-azure-devtest-labs"></a>Configuración del apagado automático de máquinas virtuales de laboratorio y proceso en Azure DevTest Labs

En este artículo se explica cómo configurar las opciones de apagado automático de las máquinas virtuales de laboratorio y proceso de DevTest Labs. 

## <a name="configure-autoshutdown-for-lab-vms-devtest-labs"></a>Configuración del apagado automático de las máquinas virtuales de laboratorio (DevTest Labs)
Azure DevTest Labs permite controlar los costos y desperdiciar lo mínimo posible en sus laboratorios gracias a la posibilidad de administrar políticas (configuración) en cada uno de ellos. En este artículo le mostramos cómo configurar la directiva de apagado automático para una cuenta de laboratorio y cómo configurar el apagado automático para un laboratorio de la cuenta de laboratorio. Para ver cómo establecer cada una de las directivas de laboratorio, vea [Definición de directivas de laboratorio en Azure DevTest Labs](devtest-lab-set-lab-policy.md).  

### <a name="set-auto-shutdown-policy-for-a-lab"></a>Establecer la directiva de apagado automático para un laboratorio
Como propietario del laboratorio, puede configurar una programación de apagado para todas las máquinas virtuales de su laboratorio. Al hacerlo, puede ahorrar costos derivados de la ejecución de máquinas que no se usan (inactivas). Puede aplicar una directiva de apagado en todas las máquinas del laboratorio a nivel central, pero también ahorrarles a los usuarios del laboratorio el trabajo de configurar una programación para sus máquinas individuales. Esta característica le permite definir la directiva en la programación del laboratorio: desde no ofrecer ningún control a ofrecer control completo a los usuarios del laboratorio. Como propietario del laboratorio, puede configurar esta directiva mediante los pasos siguientes:

1. En la página principal del laboratorio, seleccione **Configuration and policies** (Configuración y directivas).
2. Seleccione **Auto shutdown policy** (Directiva de apagado automático) en la sección **Schedules** (Programaciones) del menú izquierdo.
3. Seleccione una de las opciones. En las siguientes secciones, se proporcionan más detalles de estas opciones: La directiva establecida se aplica solo a las nuevas máquinas virtuales creadas en el laboratorio y no a las máquinas virtuales ya existentes. 

    ![Opciones de la directiva de apagado automático](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-options.png)

### <a name="configure-auto-shutdown-settings"></a>Configuración de opciones de apagado automático
La directiva de apagado automático le ayuda a minimizar la pérdida del laboratorio, ya que permite especificar la hora de apagado de la máquina virtual de este laboratorio.

Para ver (y cambiar) las directivas de un laboratorio, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
3. En la lista de laboratorios, seleccione el laboratorio que desee.   
4. Seleccione **Configuration and policies** (Directivas y configuración).

    ![Panel de configuración de directivas](./media/devtest-lab-set-lab-policy/policies-menu.png)
5. En el panel **Configuration and policies** (Configuración y directivas) del laboratorio, seleccione **Auto shutdown** (Apagado automático) en **Schedules** (Programaciones).
   
    ![Apagado automático](./media/devtest-lab-set-lab-policy/auto-shutdown.png)
6. Seleccione **Activado** para habilitar esta directiva, y **Desactivado** para deshabilitarla.
7. Si habilita esta directiva, especifique la hora local (y la zona horaria) para apagar todas las máquinas virtuales del laboratorio actual.
8. Especifique **Sí** o **No** en la opción de enviar una notificación 30 minutos antes de la hora especificada para el apagado automático. Si elige **Sí**, escriba un punto de conexión de URL del webhook o una dirección de correo electrónico para especificar dónde quiere que se publique o se envíe la notificación. El usuario recibe la notificación y se le ofrece la opción de retrasar el apagado. Para obtener más información, consulte la sección [Notificaciones](#notifications). 
9. Seleccione **Guardar**.

    De manera predeterminada, una vez que se habilite, esta directiva se aplica a todas las máquinas virtuales del laboratorio actual. Para quitar esta configuración de una máquina virtual específica, abra el panel de administración de la máquina virtual y cambie la configuración del **apagado automático**.
    
> [!NOTE]
> Si actualiza la programación de apagado automático del laboratorio o de una máquina virtual de laboratorio específica en un plazo de 30 minutos a partir de la hora programada, se aplicará la hora de apagado actualizada en la programación del día siguiente. 

### <a name="user-sets-a-schedule-and-can-opt-out"></a>El usuario establece una programación y puede excluirse de ella
Si establece esta directiva para el laboratorio, los usuarios del laboratorio pueden excluirse de la programación del laboratorio o invalidarla. Esta opción concede a los usuarios del laboratorio control completo sobre la programación del apagado automático de sus máquinas virtuales. Los usuarios del laboratorio no verán ningún cambio en su página de programación de apagado de la máquina virtual.

![Opciones de la directiva de apagado automático: 1](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-1.png)

### <a name="user-sets-a-schedule-and-cannot-opt-out"></a>El usuario establece una programación y no puede excluirse de ella
Si establece esta directiva para el laboratorio, los usuarios del laboratorio pueden invalidar la programación del laboratorio. Sin embargo, no se pueden excluir de la directiva de apagado automático. Esta opción garantiza que todas las máquinas del laboratorio están bajo una programación de apagado automático. Los usuarios del laboratorio pueden actualizar la programación de apagado automático de sus máquinas virtuales y configurar notificaciones de apagado.

![Opciones de la directiva de apagado automático: 2](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-2.png)

### <a name="user-has-no-control-over-the-schedule-set-by-lab-admin"></a>El usuario no tiene control sobre la programación establecida por el administrador del laboratorio
Si establece esta directiva para el laboratorio, los usuarios del laboratorio no pueden excluirse de la programación del laboratorio ni invalidarla. Esta opción ofrece al administrador del laboratorio control total sobre la programación de cada máquina del laboratorio. Los usuarios del laboratorio solo pueden configurar notificaciones de apagado automático para sus máquinas virtuales.

![Opciones de la directiva de apagado automático: 3](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-3.png)

### <a name="notifications"></a>Notificaciones
Una vez que el propietario del laboratorio haya configurado el apagado automático, las notificaciones se enviarán a los usuarios del laboratorio 30 minutos antes de que se active el apagado automático si alguna de sus VM se ve afectada. Esta opción proporciona a los usuarios del laboratorio la oportunidad de guardar su trabajo antes del apagado. La notificación también les proporciona vínculos para cada VM para que puedan realizar las siguientes acciones:

- Omitir el apagado automático de ese momento
- Posponer el apagado automático durante una o dos horas, para que puedan seguir trabajando en la VM.

La notificación se envía a través del punto de conexión del webhook configurado o mediante una dirección de correo electrónico que especificaron los propietarios del laboratorio en la configuración del apagado automático. Los webhooks le permiten crear o configurar integraciones que se suscriben a ciertos eventos. Cuando se activa uno de esos eventos, DevTest Labs enviará una carga HTTP POST a la dirección URL configurada del webhook. Para obtener más información sobre los webhooks, consulte [Creación de un webhook o una función de API de Azure](../azure-functions/functions-bindings-http-webhook.md). 

Le recomendamos que use webhooks porque son ampliamente compatibles con varias aplicaciones (por ejemplo, Slack, Azure Logic Apps, etc.) y le permiten implementar su propio método para enviar notificaciones. A modo de ejemplo, este artículo le indicará cómo obtener notificaciones de apagado automático de los correos electrónicos, mediante Azure Logic Apps. Primero, veamos rápidamente los pasos básicos para habilitar la notificación de apagado automático en su laboratorio.   

### <a name="create-a-logic-app-that-receives-email-notifications"></a>Crear una aplicación lógica que reciba notificaciones por correo electrónico
[Azure Logic Apps](../logic-apps/logic-apps-overview.md) le ofrece muchos conectores integrados que facilitan la integración de un servicio con otros clientes, como Office 365 y Twitter. En el nivel alto, los pasos para configurar una aplicación lógica para las notificaciones por correo electrónico se pueden dividir en cuatro fases: 

- Cree una aplicación lógica. 
- Configure la plantilla integrada.
- Intégrela con su cliente de correo electrónico.
- Copie la dirección URL del webhook.

### <a name="create-a-logic-app"></a>Creación de una aplicación lógica
Para comenzar, cree una aplicación lógica en su suscripción de Azure siguiendo estos pasos:

1. Seleccione **+ Create a resource** (+ Crear un recurso) en el menú de la izquierda, seleccione **Integration** (Integración) y seleccione **Logic App** (Aplicación lógica). 

    ![Nuevo menú de aplicaciones lógicas](./media/devtest-lab-auto-shutdown/new-logic-app.png)
2. En la página **Aplicación lógica: crear**, siga estos pasos: 
    1. Escriba un **nombre** para la aplicación lógica.
    2. Selección la **suscripción**de Azure.
    3. Cree un **grupo de recursos** nuevo o seleccione uno existente. 
    4. Seleccione una **ubicación** para la aplicación lógica. 

        ![Aplicación lógica nueva: configuración](./media/devtest-lab-auto-shutdown/new-logic-app-page.png)
3. En las **notificaciones**, seleccione **Go to resource** (Ir al recurso) en la notificación. 

    ![Ir al recurso](./media/devtest-lab-auto-shutdown/go-to-resource.png)
4. Seleccione **Logic app designer** (Diseñador de aplicaciones lógicas) en la categoría **Deployment Tools** (Herramientas de implementación).

    ![Seleccionar solicitud o respuesta HTTP](./media/devtest-lab-auto-shutdown/select-http-request-response-option.png)
5. En la página **HTTP Request-Response** (Solicitud-respuesta HTTP), seleccione **Use this template** (Usar esta plantilla). 

    ![Seleccione Usar esta opción de plantilla](./media/devtest-lab-auto-shutdown/select-use-this-template.png)
6. Copie el siguiente JSON en la sección **Request Body JSON Schema** (Esquema de JSON del cuerpo de la solicitud): 

    ```json
    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "properties": {
            "delayUrl120": {
                "type": "string"
            },
            "delayUrl60": {
                "type": "string"
            },
            "eventType": {
                "type": "string"
            },
            "guid": {
                "type": "string"
            },
            "labName": {
                "type": "string"
            },
            "owner": {
                "type": "string"
            },
            "resourceGroupName": {
                "type": "string"
            },
            "skipUrl": {
                "type": "string"
            },
            "subscriptionId": {
                "type": "string"
            },
            "text": {
                "type": "string"
            },
            "vmName": {
                "type": "string"
            }
        },
        "required": [
            "skipUrl",
            "delayUrl60",
            "delayUrl120",
            "vmName",
            "guid",
            "owner",
            "eventType",
            "text",
            "subscriptionId",
            "resourceGroupName",
            "labName"
        ],
        "type": "object"
    }
    ```
    
    ![Esquema JSON del cuerpo de la solicitud](./media/devtest-lab-auto-shutdown/request-json.png)
7. Seleccione **+ New step** (+ Nuevo paso) en el diseñador y siga estos pasos:
    1. Busque **Office 365 Outlook - Send an email** (Office 365 Outlook: enviar un correo electrónico). 
    2. Seleccione **Send an email** (Enviar un correo electrónico) desde **Actions** (Acciones). 
    
        ![Opción de enviar un correo electrónico](./media/devtest-lab-auto-shutdown/select-send-email.png)
    3. Seleccione **Sign in** (Iniciar sesión) para iniciar sesión en su cuenta de correo electrónico. 
    4. Seleccione el campo **TO** (PARA) y elija el propietario.
    5. Seleccione **SUBJECT** (ASUNTO) y escriba el asunto de la notificación por correo electrónico. Por ejemplo: "Apagado de la máquina vmName para el laboratorio: labName".
    6. Seleccione **BODY** (CUERPO) y defina el contenido del cuerpo para la notificación por correo electrónico. Por ejemplo: "vmName está programado para apagarse en 15 minutos. Para omitir este apagado, haga clic en: dirección URL. Retrasar el apagado durante una hora: delayUrl60. Retrasar el apagado durante dos hora: delayUrl120".

        ![Esquema JSON del cuerpo de la solicitud](./media/devtest-lab-auto-shutdown/email-options.png)
1. Seleccione **Guardar** en la barra de herramientas. Ya puede copiar la **dirección URL de HTTP POST**. Seleccione el botón para copiar la dirección URL en el Portapapeles. 

    ![Dirección URL de Webhook](./media/devtest-lab-auto-shutdown/webhook-url.png)

## <a name="configure-autoshutdown-for-compute-vms"></a>Configuración del apagado automático de máquinas virtuales de proceso

1. En la página **Máquina virtual**, seleccione **Apagado automático** en el menú de la izquierda de la sección **Operaciones**. 
2. En la página **Apagado automático**, seleccione **Activado** para habilitar esta directiva y **Desactivado** para deshabilitarla.
3. Si habilita esta directiva, especifique la **hora** (y la **zona horaria**) a la que se deben apagar las máquinas virtuales.
4. Especifique **Sí** o **No** en la opción de enviar una notificación 30 minutos antes de la hora especificada para el apagado automático. Si elige **Sí**, escriba un punto de conexión de URL del webhook o una dirección de correo electrónico para especificar dónde quiere que se publique o se envíe la notificación. El usuario recibe la notificación y se le ofrece la opción de retrasar el apagado. Para obtener más información, consulte la sección [Notificaciones](#notifications). 
9. Seleccione **Guardar**.

    ![Configuración del apagado automático de una máquina virtual de proceso](./media/devtest-lab-auto-shutdown/comnpute-auto-shutdown.png)

### <a name="view-activity-logs-for-auto-shutdown-updates"></a>Visualización de registros de actividad para las actualizaciones del apagado automático
Al actualizar la configuración del apagado automático, verá la actividad registrada en el registro de actividad de la máquina virtual. 

1. En [Azure Portal](https://portal.azure.com), vaya a la página principal de la máquina virtual.
2. Seleccione **Registro de actividad** en el menú de la izquierda. 
3. Quite **Recurso: mycomputevm** de los filtros.
3. Confirme que ve la operación **Agregar o modificar programaciones** en el registro de actividad. Si no lo ve, espere unos minutos y actualice el registro de actividad.

    ![Entrada de registro de actividad](./media/devtest-lab-auto-shutdown/activity-log-entry.png)
4. Seleccione la operación **Agregar o modificar programaciones** para ver la información siguiente en la página **Resumen**:

    - Nombre de la operación (Agregar o modificar programaciones)
    - Fecha y hora en que se actualizó la configuración del apagado automático.
    - La dirección de correo electrónico del usuario que actualizó la configuración. 

        ![Resumen de entradas del registro de actividad](./media/devtest-lab-auto-shutdown/activity-log-entry-summary.png)
5. Cambie a la pestaña **Historial de cambios** de la página **Agregar o modificar programaciones** y verá el historial de cambios de la configuración. En el ejemplo siguiente, la hora de apagado se cambió de 7 p.m. a 6 p.m. el 10 de abril de 2020 a las 15:18:47 EST. Y la configuración se deshabilitó a las 15:25:09 EST. 

    ![Registro de actividad - historial de cambios](./media/devtest-lab-auto-shutdown/activity-log-entry-change-history.png)
6. Para ver más detalles sobre la operación, cambie a la pestaña **JSON** de la página **Agregar o modificar programaciones**.

## <a name="next-steps"></a>Pasos siguientes
Para saber cómo configurar todas las directivas, consulte [Administración de las directivas de un laboratorio de Azure DevTest Labs](devtest-lab-set-lab-policy.md).
