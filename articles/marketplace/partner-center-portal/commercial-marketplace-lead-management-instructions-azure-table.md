---
title: 'Administración de clientes potenciales con Azure Table Storage: marketplace comercial de Microsoft'
description: Aprenda a usar Azure Table Storage para administrar clientes potenciales de Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 08/25/2020
ms.openlocfilehash: db2bae9d9e1c9658937e725a04d919743ff9999e
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855779"
---
# <a name="use-azure-table-storage-to-manage-commercial-marketplace-leads"></a>Uso de Azure Table Storage para administrar clientes potenciales de marketplace comercial

Si el sistema de administración de relaciones con clientes (CRM) no es una opción admitida en el Centro de partners para recibir clientes potenciales de Azure Marketplace y Microsoft AppSource, puede usar Azure Table Storage para controlar estos clientes potenciales. Después, puede exportar los datos e importarlos en el sistema CRM. En este artículo se explica cómo crear una cuenta de almacenamiento de Azure y una tabla en dicha cuenta. Además, puede crear un flujo mediante Power Automate para enviar una notificación por correo electrónico cuando su oferta reciba un cliente potencial.

## <a name="configure-an-azure-storage-account"></a>Configuración de una cuenta de almacenamiento de Azure

1. Si no tiene una cuenta de Azure, puede [crear una cuenta de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
1. Cuando la cuenta de Azure esté activa, inicie sesión en [Azure Portal](https://portal.azure.com).
1. En Azure Portal, cree una cuenta de almacenamiento mediante el siguiente procedimiento:

    1. En la barra de menús de la izquierda, seleccione **+ Crear un recurso**. Aparece el panel **Nuevo** a la derecha.
    1. Seleccione **Almacenamiento** en el panel **Nuevo**. Aparece una lista **Destacados** a la derecha.
    1. Seleccione **Cuenta de almacenamiento** para comenzar a crear la cuenta. Siga las instrucciones que se indican en [Creación de una cuenta de almacenamiento](../../storage/common/storage-quickstart-create-account.md?tabs=azure-portal).

        :::image type="content" source="media/commercial-marketplace-lead-management-instructions-azure-table/azure-storage-create.png" alt-text="Pasos para crear una cuenta de Azure Storage.":::

        Para más información sobre las cuentas de almacenamiento, consulte el [tutorial de inicio rápido](../../storage/index.yml). Para más información sobre los precios de almacenamiento, consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

1. Espere hasta que se aprovisione la cuenta de almacenamiento. Este proceso acostumbra a tardar unos minutos.

## <a name="create-a-table-in-your-storage-account"></a>Creación de una tabla en la cuenta de almacenamiento

1. En la página **Inicio** de Azure Portal, seleccione **Ver todos los recursos** para obtener acceso a la cuenta de almacenamiento. También puede seleccionar **Todos los recursos** en la barra de menús izquierda de Azure Portal.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-azure-table/azure-storage-access.png" alt-text="Acceso a la cuenta de Azure Storage.":::

1. En el panel de la cuenta de almacenamiento, seleccione **Claves de acceso** y copie el valor de la **Cadena de conexión** de la clave. Guarde este valor, ya que es el valor de **Cadena de conexión de cuenta de almacenamiento** que tendrá que proporcionar en el portal de publicación para recibir clientes potenciales de su oferta de Marketplace.

    Este es un ejemplo de una cadena de conexión.

    ```sql
    DefaultEndpointsProtocol=https;AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=core.screens.net
    ```

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-azure-table/azure-storage-keys.png" alt-text="Clave de Azure Storage.":::


1. En el panel de la cuenta de almacenamiento, seleccione **Tablas** y **+Tabla** para crear una tabla. Escriba el nombre de la tabla y seleccione **Aceptar**. Guarde este valor, ya que lo necesitará si quiere configurar un flujo para recibir notificaciones por correo electrónico cuando reciba clientes potenciales.

    ![Tablas de Azure](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-tables.png)

    Puede usar el [Explorador de Azure Storage](https://archive.codeplex.com/?p=azurestorageexplorer) o cualquier otra herramienta para ver los datos de la tabla de almacenamiento. También puede exportar los datos en la tabla de Azure.

## <a name="optional-use-power-automate-to-get-lead-notifications"></a>(Opcional) Uso de Power Automate para recibir notificaciones de clientes potenciales

Puede usar [Power Automate](https://docs.microsoft.com/flow/) para automatizar las notificaciones cada vez que un cliente potencial se agregue a la tabla de Azure Storage. Si aún no tiene una cuenta, puede [registrarse para obtener una gratuita](https://flow.microsoft.com/).

### <a name="lead-notification-example"></a>Ejemplo de notificación de cliente potencial

En este ejemplo se crea un flujo que envía automáticamente una notificación por correo electrónico cuando se agrega un nuevo cliente potencial a Azure Table Storage. En este ejemplo se configura una periodicidad para enviar información de clientes potenciales cada hora si se actualiza el almacenamiento en la tabla.

1. Inicie sesión en su cuenta de Power Automate.
1. En la barra izquierda, seleccione **Mis flujos**.
1. En la barra superior, seleccione **+ Nuevo**.
1. En la lista desplegable, seleccione **+ Programado: desde cero**.

   ![Mis flujos + Programado: desde cero](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-scheduled-from-blank.png)

1. En la ventana **Crear un flujo programado** en **Repetir cada** seleccione **1** para el intervalo y **Hora** para la frecuencia. Además, asigne un nombre al flujo si lo desea. Seleccione **Crear**.

   >[!NOTE]
   >Aunque en este ejemplo se usa un intervalo de 1 hora, puede seleccionar el intervalo y la frecuencia más apropiada para sus necesidades empresariales.

   ![Creación de un flujo programado](./media/commercial-marketplace-lead-management-instructions-azure-table/build-scheduled-flow.png)

1. Seleccione **+ New step**(+ Nuevo paso).
1. En la ventana **Elegir una acción**, busque **Obtener la hora pasada**. A continuación, en **Acciones**, seleccione **Obtener la hora pasada**.

   ![Elegir una acción](./media/commercial-marketplace-lead-management-instructions-azure-table/choose-an-action.png)

1. En la ventana **Obtener la hora pasada**, establezca el valor de **Intervalo** en **1**. En la lista desplegable **Unidad de tiempo**, seleccione **Hora**.

    >[!IMPORTANT]
    >Asegúrese de que el intervalo y la unidad de tiempo que ha enviado en el paso 8 coincidan con el intervalo y la frecuencia que ha configurado para la periodicidad en el paso 5.

    ![Establecimiento del intervalo de Obtener la hora pasada](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-getpast-time.png)

   >[!TIP]
   >Puede comprobar el flujo en cualquier momento para comprobar que cada paso está configurado correctamente. Para comprobar el flujo, seleccione **Comprobador de flujo** en la barra de menús **Flujo**.

   En el siguiente conjunto de pasos, se conectará a la tabla y configurará la lógica de procesamiento para controlar nuevos clientes potenciales.

1. Seleccione **+ New step**(+ Nuevo paso). En la ventana **Elegir una acción**, busque **Obtener entidades**.
1. En **Acciones**,seleccione **Obtener entidades (Almacenamiento de tablas de Azure)** .
1. En la ventana **Azure Table Storage**, proporcione información en los cuadros siguientes y seleccione **Crear**:

    * **Nombre de la conexión**: proporcione un nombre descriptivo para la conexión que va a establecer entre este flujo y la tabla.
    * **Nombre de cuenta de almacenamiento**: proporcione el nombre de la cuenta de almacenamiento para la tabla. Puede encontrarlo en la página **Claves de acceso** de la cuenta de almacenamiento.
    * **Clave de almacenamiento compartida**: proporcione el valor de clave de la cuenta de almacenamiento de la tabla. Puede encontrar este valor en la página **Claves de acceso** de la cuenta de almacenamiento.

      ![Ventana de Azure Table Storage](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-storage.png)

   Después de seleccionar **Crear**, aparece la ventana **Obtener entidades**. Aquí, seleccione **Mostrar opciones avanzadas** y proporcione información en los cuadros siguientes:

   * **Tabla**: seleccione el nombre de la tabla (desde [Crear una tabla](#create-a-table-in-your-storage-account)). En la captura de pantalla siguiente se muestra el símbolo del sistema cuando se selecciona la tabla `marketplaceleads` en este ejemplo.

     ![Ventana Obtener entidades](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-get-entities.png)

   * **Consulta de filtro**: seleccione este cuadro y pegue esta función en el cuadro: `Timestamp gt datetime'@{body('Get_past_time')}'`.

     ![Obtener entidades, cuadro Consulta de filtro](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-get-entities-filter-query.png)

1. Ahora que ha completado la configuración de la conexión a la tabla de Azure, seleccione **Nuevo paso** para agregar una condición para examinar la tabla de Azure para los nuevos clientes potenciales.

1. En la ventana **Elegir una acción**, seleccione **Acciones**. Luego, elija **Control de condiciones**.

    ![Ventana Elegir una acción](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-choose-an-action.png)

1. En la ventana **Condición**, seleccione **Elegir un valor**. Luego, seleccione **Expresión** en la ventana emergente.

1. Pegue `length(body('Get_entities')?['value'])` en el cuadro **fx**. Seleccione **Aceptar** para agregar esta función.

1. Para finalizar la configuración de la condición:
    1. Seleccione **es mayor que** en la lista desplegable.
    2. Escriba **0** como valor.

        ![Ventana Condición](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition.png)

   En los pasos siguientes, va a configurar la acción que se realizará según el resultado de la condición:

   * Si la condición se resuelve como **En caso negativo**, no haga nada.
   * Si la condición se resuelve como **En caso positivo**, desencadene una acción que se conecte a su cuenta de Office 365 para enviar un correo electrónico.

1. Seleccione **Agregar una acción** en **En caso positivo**.

    ![Ventana Condición, si es así, agregar una acción](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition-if-yes.png)

1. Seleccione **Enviar un correo electrónico (Office 365 Outlook)** .

    ![Ventana Condición, si es sí, enviar un correo electrónico](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition-if-yes-send-email.png)

    >[!NOTE]
    >Para usar otro proveedor de correo electrónico, busque y seleccione en su lugar **Enviar una notificación por correo electrónico (Correo)** como acción. Aunque en las instrucciones se muestra cómo realizar la configuración con Office 365 Outlook, las correspondientes a un proveedor diferente son muy similares.

1. En la ventana de Office 365 Outlook, proporcione información en los cuadros siguientes:

    1. **Para**: escriba una dirección de correo electrónico para todos los usuarios que reciban esta notificación.
    1. **Firmante**: proporcione un asunto para el correo electrónico. Por ejemplo, **Nuevos clientes potenciales**.
    1. **Cuerpo**: agregue el texto que quiera incluir en cada correo electrónico (opcional) y, luego, péguelo en `body('Get_entities')?['value']`.

    >[!NOTE]
    >Puede insertar puntos de datos estáticos o dinámicos adicionales en el cuerpo de este correo electrónico.

    ![Ventana Condición, si es sí, ventana de Office 365 Outlook](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-condition-if-yes-outlook.png)

1. Seleccione **Guardar** para guardar el flujo. Power Automate comprueba automáticamente el flujo en busca de errores. Si no hay errores, el flujo comienza a ejecutarse después de guardarlo.

La imagen siguiente muestra un ejemplo de cómo debe ser el último flujo.

![Un ejemplo del flujo final.](./media/commercial-marketplace-lead-management-instructions-azure-table/final-flow.png)

### <a name="manage-your-flow"></a>Administración del flujo

Es fácil administrar el flujo una vez se está ejecutando. Tiene control completo sobre el flujo. Por ejemplo, puede detenerlo, editarlo, ver un historial de ejecución y obtener el análisis. En la siguiente imagen se muestran las opciones disponibles para administrar un flujo.

 ![Administración de un flujo](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-manage-completed.png)

El flujo seguirá ejecutándose hasta que lo detenga mediante la opción **Desactivar el flujo**.

Si no recibe notificaciones de correo electrónico de clientes potenciales, significa que no se han agregado nuevos clientes potenciales a la tabla de Azure. Si se produce un error en el flujo, recibirá un correo electrónico similar al del ejemplo siguiente.

 ![Notificación por correo electrónico de los errores del flujo](./media/commercial-marketplace-lead-management-instructions-azure-table/ms-flow-failure-note.png)

## <a name="configure-your-offer-to-send-leads-to-the-azure-table"></a>Configuración de la oferta para enviar clientes potenciales a la tabla de Azure

Cuando esté listo para configurar la información de administración de clientes potenciales de la oferta en el portal de publicación, siga estos pasos:

1. Vaya a la página **Configuración de la oferta** de su oferta.

1. En la sección **Clientes potenciales**, seleccione **Conectar**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-azure-table/customer-leads.png" alt-text="Clientes potenciales":::

1. En la ventana emergente **Detalles de la conexión**, seleccione **Azure Table** como **destino de clientes potenciales**. 
     ![Administración de clientes potenciales, detalles de la conexión](./media/commercial-marketplace-lead-management-instructions-azure-table/connection-details.png)

1. En el cuadro **Cadena de conexión de cuenta de almacenamiento**, pegue la cadena de conexión de la cuenta de almacenamiento de Azure que creó siguiendo los pasos anteriores.
     ![Administración de clientes potenciales, cuenta de almacenamiento de detalles de conexión](./media/commercial-marketplace-lead-management-instructions-azure-table/azure-table-connection-details.png)

1. **Dirección de correo electrónico de contacto**: proporcione los correos electrónicos de las personas de su empresa que deban recibir notificaciones por correo electrónico cuando se reciba un nuevo cliente potencial. Puede proporcionar varias direcciones de correo electrónico separadas con punto y coma.

1. Seleccione **Aceptar**.

Para asegurarse de que se ha conectado correctamente a un destino de clientes potenciales, seleccione el botón **Validar**. Si la validación es correcta, tendrá un cliente potencial de prueba en el destino de clientes potenciales.

>[!NOTE]
>Debe terminar de configurar el resto de las partes de la oferta y publicarla para poder recibir clientes potenciales para la oferta.

Cuando se generan los clientes potenciales, Microsoft los envía a la tabla de Azure. Si ha configurado un flujo, también se enviará un correo electrónico a la dirección de correo electrónico configurada.
