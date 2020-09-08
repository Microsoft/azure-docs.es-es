---
title: Introducción a Azure Enterprise Portal
description: En este artículo se explica cómo los clientes de Contrato Enterprise de Azure (Azure EA) usan Azure Enterprise Portal.
author: bandersmsft
ms.author: banders
ms.date: 08/20/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.openlocfilehash: 8997cf5e86eb79b527c0b37ef0b85556469aea98
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687644"
---
# <a name="get-started-with-the-azure-enterprise-portal"></a>Introducción a Azure Enterprise Portal

Este artículo ayuda a los clientes directos e indirectos de Contrato Enterprise de Azure (Azure EA) a empezar a usar [Azure Enterprise Portal](https://ea.azure.com). Obtenga información básica sobre:

- La estructura de Azure Enterprise Portal.
- Los roles usados en Azure Enterprise Portal.
- La creación de suscripciones.
- El análisis de costos en Azure Enterprise Portal y Azure Portal.

Consulte este vídeo para ver una sesión completa de incorporación de Azure Enterprise Portal:

> [!VIDEO https://www.youtube.com/embed/OiZ1GdBpo-I]

## <a name="azure-enterprise-portal-hierarchy"></a>Jerarquía de Azure Enterprise Portal

La jerarquía de Azure Enterprise Portal consta de:

- **Azure Enterprise Portal**: Portal de administración en línea que ayuda a administrar los costos de los servicios de Azure EA. Puede:

  - Crear una jerarquía de Azure EA con departamentos, cuentas y suscripciones.
  - Conciliar los costos de los servicios consumidos, descargar informes de uso y ver listas de precios.
  - Crear claves de API para la inscripción.

- Los **departamentos** ayudan a segmentar los costos en agrupaciones lógicas. Los departamentos le permiten establecer un presupuesto o una cuota a nivel de departamento.

- Las **cuentas** son unidades organizativas en Azure Enterprise Portal. Puede usar las cuentas para administrar las suscripciones y obtener acceso a los informes.

- Las **suscripciones** son la unidad más pequeña Azure Enterprise Portal. Son contenedores para los servicios de Azure que administra el administrador de servicios.

En el siguiente diagrama se ilustran las jerarquías simples de Azure EA.

![Diagrama de jerarquías simples de Azure EA](./media/ea-portal-get-started/ea-hierarchies.png)

## <a name="enterprise-user-roles"></a>Roles de los usuarios de empresa

Los siguientes roles de usuario administrativo forman parte de la inscripción empresarial:

- Administrador de empresa
- Administrador de departamentos
- Propietario de cuenta
- Administrador de servicios
- Contacto para notificaciones

Los roles funcionan en dos portales distintos para completar tareas. Use [Azure Enterprise Portal](https://ea.azure.com) para administrar la facturación y los costos, y [Azure Portal](https://portal.azure.com) para administrar los servicios de Azure.

Los roles de usuario están asociados a una cuenta de usuario. Para validar la autenticidad de los usuarios, todos ellos deben tener una cuenta profesional, educativa o de Microsoft válida. Asegúrese de que cada cuenta está asociada a una dirección de correo electrónico que se supervisa activamente. Las notificaciones de las cuentas se envían a la dirección de correo electrónico.

Al configurar usuarios, puede asignar varias cuentas al rol Administrador de empresa. Sin embargo, solo una cuenta puede contener el rol Propietario de la cuenta. Además, puede asignar tanto el rol Administrador de empresa como el rol Propietario de cuenta a una sola cuenta.

### <a name="enterprise-administrator"></a>Administrador de empresa

Los usuarios con este rol tienen el nivel de acceso más alto. Pueden realizar las acciones siguientes:

- Administrar cuentas y propietarios de cuentas.
- Administrar otros administradores de empresa.
- Administrar administradores de departamento.
- Administrar contactos de notificación.
- Ver el uso en todas las cuentas.
- Ver los cargos no facturados en todas las cuentas.

Puede tener varios administradores de empresa en una inscripción empresarial. Puede conceder acceso de solo lectura a los administradores de empresa. Estos heredan el rol de administrador de departamento.

### <a name="department-administrator"></a>Administrador de departamentos

Los usuarios con este rol pueden:

- Crear y administrar departamentos.
- Crear nuevos propietarios de cuentas.
- Ver detalles de uso de los departamentos que administran.
- Ver los costos, si tienen los permisos necesarios.

Puede tener varios administradores de departamento en cada inscripción empresarial.

Puede conceder a los administradores de departamento acceso de solo lectura al editar o crear un nuevo administrador de departamento. Establezca la opción de solo lectura en **Sí**.

### <a name="account-owner"></a>Propietario de cuenta

Los usuarios con este rol pueden:

- Crear y administrar suscripciones.
- Administrar administradores de servicios.
- Ver el uso de las suscripciones.

Todas las cuentas requieren una cuenta profesional, educativa o de Microsoft única. Para obtener más información sobre los roles administrativos de Azure Enterprise Portal, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](understand-ea-roles.md).

### <a name="service-administrator"></a>Administrador de servicios

El rol Administrador de servicios tiene permisos para administrar servicios en Azure Portal y asignar a los usuarios el rol de coadministrador.

### <a name="notification-contact"></a>Contacto para notificaciones

El contacto para notificaciones recibe las notificaciones de uso relacionadas con la inscripción.

## <a name="activate-your-enrollment"></a>Activación de la inscripción

Para activar el servicio, el administrador de empresa inicial abre [Azure Enterprise Portal](https://ea.azure.com) e inicia sesión con la dirección del correo electrónico de invitación.

Si le han asignado el rol de administrador de empresa, no es preciso que reciba el correo electrónico de activación. Vaya a [Azure Enterprise Portal](https://ea.azure.com) e inicie sesión con la dirección de correo electrónico de su cuenta profesional, educativa o de Microsoft y su contraseña.

Si tiene más de una inscripción, elija la que desea activar. De forma predeterminada, solo se muestran las inscripciones activas. Para ver el historial de inscripciones, desactive la opción **Activa** en la parte superior derecha de Azure Enterprise Portal.

En **Inscripción**, aparece el estado **Activa**.

![Ejemplo en el que se muestra una inscripción activa](./media/ea-portal-get-started/ea-enrollment-status.png)

Los administradores de empresa de Azure existentes son los únicos que pueden crear otros administradores de empresa.

### <a name="create-another-enterprise-administrator"></a>Creación de otro administrador de empresa

Para agregar otro administrador de empresa:

1. Inicie sesión en [Azure Enterprise Portal](https://ea.azure.com).
1. Vaya a **Administrar** > **Enrollment Detail** (Detalles de inscripción).
1. En la parte superior derecha, seleccione **Agregar administrador**.

Asegúrese de tener la dirección de correo electrónico del usuario y el método de autenticación preferido, como cuenta profesional, educativa o de Microsoft.

Si no es administrador de empresa, póngase en contacto con uno para pedirle que le agregue a una inscripción. Una vez que se haya agregado a una inscripción, recibirá un correo electrónico de activación.

Si el administrador de empresa no puede ayudarle, cree una [solicitud de soporte técnico en Azure Enterprise Portal](https://support.microsoft.com/supportrequestform/cf791efa-485b-95a3-6fad-3daf9cd4027c). Proporcione la siguiente información:

- Número de inscripción
- Dirección de correo electrónico que se debe agregar y tipo de autenticación (cuenta profesional, educativa o de Microsoft)
- Aprobación por correo electrónico por parte de un administrador de empresa existente
  - Si el administrador de empresa existente no está disponible, póngase en contacto con su partner o asesor de software para solicitarle que cambie los detalles del contacto a través de la herramienta Centro de servicios de licencias por volumen (VLSC).

Para más información acerca de los roles administrativos de la empresa, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](understand-ea-roles.md).

## <a name="create-an-azure-enterprise-department"></a>Creación de un departamento de Azure Enterprise

Los administradores de empresa y los administradores de departamento usan departamentos para organizar e informar no solo acerca de los servicios empresariales de Azure sino también de su uso por departamento y centro de costos. El administrador de empresa puede:

- Agregar o quitar departamentos.
- Asociar una cuenta a un departamento.
- Crear administradores de departamento.
- Permitir a los administradores de departamento ver el precio y los costos.

Un administrador de departamento puede agregar cuentas a sus departamentos. Puede quitar cuentas de sus departamentos, pero no de la inscripción.

Para agregar un departamento:

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Administrar** en el panel izquierdo.
1. Seleccione la pestaña **Departamento** y, a continuación, seleccione **+ Add Department** (+ Agregar departamento).
1. Escriba la información.
   El nombre del departamento es el único campo obligatorio. Debe tener tres caracteres como mínimo.
1. Cuando haya terminado, seleccione **Agregar**.

## <a name="add-a-department-administrator"></a>Adición de un administrador de departamento

Una vez creado un departamento, el administrador de empresa puede agregar administradores de departamento y asociar cada uno de ellos a un departamento. Los administradores de departamento pueden realizar las siguientes acciones para sus departamentos:

- Crear otros administradores de departamento
- Ver y editar propiedades de departamento, como el nombre o el centro de costos
- Adición de cuentas
- Quitar cuentas
- Descargar detalles de uso
- Ver el uso y los cargos mensuales <sup>1</sup>

> <sup>1</sup> Un administrador de empresa debe conceder estos permisos. Si tiene permiso para ver el uso y los cargos mensuales del departamento, pero no puede verlos, póngase en contacto con su partner.

### <a name="to-add-a-department-administrator"></a>Para agregar un administrador de departamento

Como administrador de empresa:

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Administrar** en el panel izquierdo.
1. Seleccione la pestaña **Departamento** y, a continuación, seleccione el departamento.
1. Haga clic en **+ Agregar administrador** y agregue la información necesaria.
1. Para que el acceso sea de solo lectura, establezca la opción **Solo lectura** en **Sí** y seleccione **Agregar**.

![Ejemplo que muestra el cuadro de diálogo Add Department Administrator (Agregar administrador de departamento)](./media/ea-portal-get-started/ea-create-add-department-admin.png)

### <a name="to-set-read-only-access"></a>Para establecer acceso de solo lectura

Puede conceder acceso de solo lectura a los administradores de departamento.

- Cuando cree un administrador de departamento, establezca la opción de solo lectura en **Sí**.

- Para editar un administrador de departamento existente:
   1. Seleccione un departamento y, después, seleccione el símbolo del lápiz situado junto al **administrador de departamento** que desea editar.
   1. Establezca la opción de solo lectura en **Sí**y, a continuación, seleccione **Guardar**.

Los administradores de empresa obtienen automáticamente permisos de administrador de departamento.

## <a name="add-an-account"></a>Agregar una cuenta

La estructura de las cuentas y suscripciones afecta al modo en que se administran y aparecen en las facturas e informes. Entre los ejemplos de estructura típica de una organización se incluye las divisiones empresariales, los equipos funcionales y las ubicaciones geográficas.

Para agregar una cuenta:

1. En Azure Enterprise Portal, seleccione **Administrar** en el área de navegación de la izquierda.
1. Seleccione la pestaña **Cuenta**. En la página **Cuenta**, haga clic en **+Agregar cuenta**.
1. Seleccione un departamento o déjelo como sin asignar y, después, seleccione el tipo de autenticación que desea.
1. Escriba el nombre descriptivo para identificar la cuenta en los informes.
1. Escriba la dirección de **correo electrónico del propietario de la cuenta** que se va a asociar a la nueva cuenta.
1. Confirme la dirección de correo electrónico y seleccione **Agregar**.

![Ejemplo que muestra la lista de cuentas y la opción + Add Account (+ Agregar cuenta)](./media/ea-portal-get-started/create-ea-add-an-account.png)

Para agregar otra cuenta, seleccione **Add Another Account** (Agregar otra cuenta), o bien puede seleccionar **Agregar** en la esquina inferior derecha de la barra de herramientas izquierda.

Para confirmar la propiedad de la cuenta:

1. Inicie sesión en Azure Enterprise Portal.
1. Vea el estado.

   El estado debe cambiar de **Pending** (Pendiente) a **Start/End date** (Fecha de inicio y de finalización). La fecha de inicio y de finalización es la fecha en que el usuario inició sesión por primera vez y la fecha de finalización del contrato.
1. Cuando aparezca el mensaje **Advertencia**, el propietario de la cuenta debe seleccionar **Continuar** para activar la cuenta la primera vez que inicie sesión en Azure Enterprise Portal.

## <a name="change-account-owner"></a>Cambiar el propietario de la cuenta

Los administradores de empresa pueden usar Azure Enterprise Portal para transferir la propiedad de la cuenta de suscripción en una inscripción. La acción mueve todas las suscripciones de una cuenta de usuario de origen a una cuenta de usuario de destino.

Tenga en cuenta esta información importante al transferir cuentas:

- Puede realizar estas transferencias:
  - De una cuenta profesional o educativa a otra cuenta profesional o educativa.
  - De una cuenta de Microsoft a una cuenta profesional o educativa.
  - De una cuenta de Microsoft a otra cuenta de Microsoft.

    La cuenta de destino debe ser una cuenta de Azure Commerce válida para ser un destino válido para las transferencias. En las nuevas cuentas, se le pide que cree una cuenta de Azure Commerce al iniciar sesión en Azure Enterprise Portal. En el caso de las cuentas existentes, para que la cuenta sea apta es preciso crear antes una suscripción de Azure.

- No puede realizar una transferencia de una cuenta profesional o educativa a una cuenta de Microsoft.

- Cuando se completa una transferencia de suscripción, Microsoft actualiza el propietario de la cuenta.

Comprenda estas directivas de control de acceso basado en rol (RBAC):

- Cuando se realizan transferencias de suscripciones entre dos identificadores de organización que se encuentran en el mismo inquilino, se conservan tanto las directivas de RBAC como los roles de administrador y coadministrador de servicios existentes.
- Otras transferencias de suscripciones dan como resultado la pérdida de las directivas de RBAC y las asignaciones de roles.
- Las directivas y los roles de administrador no se transfieren entre los distintos directorios. Los administradores de servicios se actualizan al propietario de la cuenta de destino.

Antes de cambiar el propietario de una cuenta:

1. En Azure Enterprise Portal, consulte la pestaña **Cuenta** e identifique la cuenta de origen. La cuenta de origen debe estar activa.
1. Identifique la cuenta de destino y asegúrese de que esté activa.

Para transferir la propiedad de la cuenta para todas las suscripciones:

1. Inicie sesión en Azure Enterprise Portal.
1. En el área de navegación izquierda, seleccione **Administrar**.
1. Seleccione la pestaña **Cuenta** y mantenga el puntero sobre una cuenta.
1. Seleccione el icono de cambio de propietario de la cuenta a la derecha. El icono se parece a una persona.
1. Elija una cuenta válida y, después, seleccione **Siguiente**.
1. Confirme la transferencia y seleccione **Enviar**.

![Imagen que muestra el símbolo Cambiar propietario de cuenta.](./media/ea-portal-get-started/create-ea-create-sub-transfer-account-ownership-of-sub.png)

Para transferir la propiedad de la cuenta para una sola suscripción:

1. Inicie sesión en Azure Enterprise Portal.
1. En el área de navegación izquierda, seleccione **Administrar**.
1. Seleccione la pestaña **Cuenta** y mantenga el puntero sobre una cuenta.
1. Seleccione el icono de transferir suscripciones a la derecha. El icono se parece a una página.
1. Elija una suscripción válida y seleccione **Siguiente**.
1. Confirme la transferencia y seleccione **Enviar**.

![Imagen que muestra el símbolo Transferir suscripciones](./media/ea-portal-get-started/ea-transfer-subscriptions.png)

Consulte este vídeo para ver la administración de usuarios de Azure Enterprise Portal:

> [!VIDEO https://www.youtube.com/embed/621jVkvmwm8]

## <a name="create-a-subscription"></a>una suscripción

Los propietarios de cuentas pueden ver y administrar suscripciones. Las suscripciones se pueden usar para dar acceso a los equipos de la organización a proyectos y entornos de desarrollo. Por ejemplo: prueba, producción, desarrollo y almacenamiento provisional.

La creación de distintas suscripciones para cada entorno de aplicación ayuda a proteger cada entorno.

- También se puede asignar una cuenta de administrador de servicios diferente a cada suscripción.
- Se pueden asociar las suscripciones a cualquier número de servicios.
- El propietario de la cuenta crea suscripciones y asigna una cuenta de administrador de servicios a cada una de las suscripciones de su cuenta.

### <a name="add-a-subscription"></a>Agregar una suscripción

Use la siguiente información para agregar una suscripción.

La primera vez que agrega una suscripción a la cuenta, se le pide que acepte el Contrato Microsoft Online Subscription (MOSA) y un plan de tarifas. Aunque ni MOSA ni el plan de tarifas se aplican a los clientes con Contrato Enterprise, son necesarios para crear la suscripción. La inscripción del Contrato Enterprise de Microsoft Azure sustituye a los elementos anteriores y la relación contractual no cambia. Cuando se le solicite, seleccione la casilla que indica que acepta los términos.

_Microsoft Azure Enterprise_ es el nombre predeterminado cuando se crea una suscripción. Puede cambiar el nombre para diferenciarla de las demás suscripciones de la inscripción y para asegurarse de reconocerla en los informes de empresa.

Para agregar una suscripción:

1. En Azure Enterprise Portal, inicie sesión con la cuenta.
1. Seleccione la pestaña **Administración** y, después, seleccione **Suscripción** en la parte superior de la página.
1. Compruebe que ha iniciado sesión como propietario de la cuenta.
1. Seleccione **+Agregar suscripción** y luego seleccione **Compra**.

   La primera vez que se agrega una suscripción a una cuenta, es preciso especificar la información de contacto. Cuando agrega suscripciones adicionales, su información de contacto se agrega automáticamente.

1. Seleccione **Suscripciones** y, después, seleccione la suscripción que ha creado.
1. Seleccione **Editar detalles de suscripción**.
1. Edite **Nombre de la suscripción** y **Administrador de servicios** y, después, seleccione la marca de verificación.

   El nombre de la suscripción aparece en los informes. Se trata del nombre del proyecto asociado a la suscripción en el portal de desarrollo.

Las nuevas suscripciones pueden tardar hasta 24 horas en aparecer en la lista de suscripciones. Después de crear una suscripción, puede:

- [Editar los detalles de la suscripción](https://account.azure.com/Subscriptions)
- [Administrar los servicios de la suscripción](https://portal.azure.com/#home)

## <a name="transfer-an-enterprise-subscription-to-a-pay-as-you-go-subscription"></a>Transferencia de una suscripción de Enterprise a una suscripción de pago por uso

Para transferir una suscripción de Enterprise a una suscripción individual con tarifas de pago por uso, debe crear una nueva solicitud de soporte técnico en Azure Enterprise Portal. Para crear una solicitud de soporte técnico, seleccione **+ Nueva solicitud de soporte técnico** en el área **Ayuda y soporte técnico**.

## <a name="associate-an-existing-account-with-your-pay-as-you-go-subscription"></a>Asociación de una cuenta existente con la suscripción de pago por uso

Si ya tiene una cuenta de Microsoft Azure existente en Azure Portal, escriba la cuenta profesional, educativa o de Microsoft asociada con el fin de asociarla a la inscripción del Contrato Enterprise.

### <a name="associate-an-existing-account"></a>Asociación de una cuenta existente

1. En Azure Enterprise Portal, seleccione **Administrar**.
1. Seleccione la pestaña **Cuenta**.
1. Haga clic en **+Agregar cuenta**.
1. Escriba la cuenta profesional, educativa o de Microsoft asociada a la cuenta de Azure existente.
1. Confirme la cuenta asociada a la cuenta de Azure existente.
1. Proporcione el nombre que desea usar para identificar esta cuenta en los informes.
1. Seleccione **Agregar**.
1. Para agregar una cuenta adicional, puede seleccionar de nuevo la opción **+Agregar cuenta** o volver a la página principal; para ello, seleccione el botón **Administración**.
1. Si ve la página **Cuenta**, la cuenta recién agregada aparecerá con estado **Pendiente**.

### <a name="confirm-account-ownership"></a>Confirmación de la propiedad de la cuenta

1. Inicie sesión en la cuenta de correo electrónico asociada a la cuenta profesional, educativa o de Microsoft que ha proporcionado.
1. Abra la notificación de correo electrónico titulada _"Invitation to Activate your Account on the Microsoft Azure Service from Microsoft Volume Licensing"_ (Invitación para activar su cuenta en el servicio Microsoft Azure de licencias por volumen de Microsoft).
1. Seleccione el vínculo **Log into the Microsoft Azure Enterprise Portal** (Iniciar sesión en Microsoft Azure Enterprise Portal) de la invitación.
1. Seleccione **Iniciar sesión**.
1. Escriba la cuenta profesional, educativa o de Microsoft y la contraseña para iniciar sesión y confirmar la propiedad de la cuenta.

### <a name="azure-marketplace"></a>Azure Marketplace

Aunque la mayoría de las suscripciones pueden convertirse del entorno de pago por uso a Contrato Enterprise de Azure, los servicios de Azure Marketplace no. Para tener una vista única de todas las suscripciones y cargos, se recomienda agregar los servicios de Azure Marketplace a Azure Enterprise Portal.

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Administrar** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Inscripción**.
1. Observe la sección **Enrollment Detail** (Detalles de la inscripción).
1. A la derecha del campo Azure Marketplace, seleccione el icono de lápiz para habilitarlo. Seleccione **Guardar**.

El propietario de la cuenta ahora puede comprar cualquier servicio de Azure Marketplace que haya adquirido anteriormente en la suscripción de pago por uso.

Después de que se activen las nuevas suscripciones de Azure Marketplace en su inscripción de Azure EA, cancele los servicios de Azure Marketplace que se crearon en el entorno de pago por uso. Este paso es fundamental para que las suscripciones de Azure Marketplace no entren en un estado incorrecto cuando expire el instrumento de pago por uso.

### <a name="msdn"></a>MSDN

Las suscripciones a MSDN se convierten automáticamente a Desarrollo/pruebas de MSDN, y la oferta de Azure EA perderá cualquier crédito monetario existente.

### <a name="azure-in-open"></a>Azure bajo licencia Open

Si asocia una suscripción de Azure bajo licencia Open con un Contrato Enterprise, perderá todos los créditos no consumidos de Azure bajo licencia Open. Por lo tanto, se recomienda usar todo el crédito en una suscripción de Azure bajo licencia Open antes de agregar la cuenta al Contrato Enterprise.  

### <a name="accounts-with-support-subscriptions"></a>Cuentas con suscripciones de soporte técnico

Si el Contrato Enterprise no tiene una suscripción de soporte técnico, y usted agrega una cuenta existente con una suscripción de soporte técnico a Azure Enterprise Portal, la suscripción de soporte técnico de MOSA no se transferirá automáticamente. Deberá volver a comprar una suscripción de soporte técnico en Azure EA durante el período de gracia, al final del mes siguiente.

## <a name="view-usage-summary-and-download-reports"></a>Visualización de resumen de uso e informes de las descargas

Los administradores de empresa pueden ver un resumen de sus datos de uso, el pago por adelantado de Azure consumido y los cargos asociados con el uso adicional en Azure Enterprise Portal. Los cargos se presentan en el nivel de resumen en todas las cuentas y suscripciones.

Para ver el uso detallado para cuentas específicas, descargue el informe de detalles de uso:

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Informes**.
1. Seleccione la pestaña **Descargar uso**.
1. En la lista de informes, seleccione **Descargar** en el informe mensual que desee obtener.

   > [!NOTE]
   > El informe de detalles de uso no incluye ningún impuesto aplicable.
   >
   > Puede haber una latencia de hasta ocho horas desde el momento en que se realizó el uso hasta el momento en que se refleja en el informe.

Para ver los informes y gráficos del resumen de uso:

1. Inicie sesión en Azure Enterprise Portal.

1. Seleccione un plazo de pago por adelantado.

   Para cambiar el rango de fechas de **Resumen de uso**, puede alternar entre **M** (mensual) y **C** (personalizado) en la parte superior derecha de la página y, a continuación, especificar fechas de inicio y de finalización personalizadas.

   ![Crear y ver el resumen de uso y descargar informes en la vista personalizada](./media/ea-portal-get-started/create-ea-view-usage-summary-and-download-reports-custom-view.png)
1. Para ver detalles adicionales, puede seleccionar un periodo o un mes en el gráfico.

   - El gráfico muestra el uso mes a mes con un desglose del uso realizado, el cargo extra por los servicios, los cargos facturados por separado y los cargos por Azure Marketplace.
   - En el mes seleccionado, puede usar los campos debajo del gráfico para filtrar por departamentos, cuentas y suscripciones.
   - Puede alternar entre **Charge by Services** (Cargo por servicios) y **Charge by Hierarchy** (Cargo por jerarquía).
   - Expanda las secciones pertinentes para ver los detalles de **Servicio de Azure**, **Gastos facturados por separado** y **Azure Marketplace**.

Consulte este vídeo para conocer cómo ver el uso:

> [!VIDEO https://www.youtube.com/embed/Cv2IZ9QCn9E]

### <a name="download-csv-reports"></a>Descarga de informes CSV

Los administradores de empresa usan la página de descarga de informes mensuales para descargar los informes siguientes como archivos CSV:

- Saldo y cargo
- Detalles de uso
- Cargos de Azure Marketplace
- Hoja de precios

Para descargar informes:

1. En Azure Enterprise Portal, seleccione **Informes**.
2. Seleccione **Descargar uso** en la parte superior de la página.
3. Seleccione **Download** (Descargar) junto al informe del mes.

   > [!NOTE]
   > Puede haber una latencia de hasta cinco días entre la fecha en que se realizó el uso y el momento en que el uso se muestra en los informes.
   >
   > Los usuarios que descarguen archivos CSV con Safari en Excel pueden encontrarse con errores de formato. Para evitar errores, abra el archivo con un editor de texto.

![Ejemplo que muestra la página Descargar uso](./media/ea-portal-get-started/create-ea-download-csv-reports.png)

Vea este vídeo para ver cómo descargar la información de uso:

> [!VIDEO https://www.youtube.com/embed/eY797htT1qg]

### <a name="advanced-report-download"></a>Descarga de informes avanzada

Puede usar la descarga de informes avanzados para obtener informes que cubran cuentas o rangos de fechas específicos. El archivo de salida está en formato CSV para admitir grandes conjuntos de registros.

1. En Azure Enterprise Portal, seleccione **Descarga de informes avanzada**.
1. Seleccione un rango de fechas adecuado y las cuentas adecuadas.
1. Seleccione **Request Usage Data** (Solicitar datos de uso).
1. Seleccione el botón **Actualizar** hasta que el estado del informe se actualice a **Descargar**.
1. Descargue el informe.

### <a name="download-usage-reports-and-billing-information-for-a-prior-enrollment"></a>Descargar informes de uso e información de facturación de una inscripción anterior

Puede descargar los informes de uso y la información de facturación para una inscripción anterior después de que haya tenido lugar la transferencia de una inscripción. Los informes históricos están disponibles en Azure Enterprise Portal y en Cost Management.

Azure Enterprise Portal filtra las inscripciones inactivas para excluirlas. Tendrá que desactivar la casilla **Activa** para ver las inscripciones transferidas inactivas.  

![Al desactivar la casilla activa, el usuario puede ver las inscripciones inactivas.](./media/ea-portal-get-started/unchecked-active-box.png)

## <a name="azure-ea-term-glossary"></a>Glosario de términos de Azure EA

- **Cuenta**: Unidad organizativa en Azure Enterprise Portal. Se utiliza para administrar las suscripciones y para los informes.
- **Propietario de la cuenta**: Persona que administra las suscripciones y los administradores de servicios en Azure. Pueden ver los datos de uso de esta cuenta y sus suscripciones asociadas.
- **Suscripción de enmienda**: Suscripción de un año o coincidente de la enmienda de la inscripción.
- **Pago por adelantado**: pago por adelantado de una cantidad monetaria anual por los servicios de Azure con una tasa de prepago con descuento por el uso de este pago por adelantado.
- **Administrador de departamento**: Persona que administra departamentos, crea cuentas nuevas y propietarios de cuentas, ve los detalles de uso de los departamentos que administra y ve los costos cuando se le conceden permisos.
- **Número de inscripción**: Identificador único proporcionado por Microsoft para identificar la inscripción específica asociada a un Contrato Enterprise.
- **Administrador de empresa**: Persona que administra departamentos, propietarios de departamentos, cuentas y propietarios de cuentas en Azure. Tienen la capacidad de administrar los administradores de empresa, así como ver los datos de uso, las cantidades facturadas y los cargos no facturados en todas las cuentas y suscripciones asociadas a la inscripción de la empresa.
- **Contrato Enterprise**: un contrato de licencias de Microsoft para clientes con adquisición centralizada que desean estandarizar toda la organización en la tecnología de Microsoft y mantener una infraestructura de tecnología de la información en un estándar de software de Microsoft.
- **Inscripción de Contrato Enterprise**: Inscripción en el programa de Contrato Enterprise que proporciona productos de Microsoft en volumen a tarifas con descuento.
- **Cuenta de Microsoft**: Servicio basado en web que permite a los sitios participantes autenticar a un usuario con un único conjunto de credenciales.
- **Enmienda de inscripción de Microsoft Azure Enterprise (enmienda de inscripción)** : Enmienda firmada por una empresa, que proporciona acceso a Azure como parte de la inscripción empresarial.
- **Azure Enterprise Portal**: Portal que usan los clientes empresariales para administrar sus cuentas de Azure y las suscripciones relacionadas.
- **Cantidad de recursos consumidos**: Cantidad de un servicio individual de Azure que se usó en un mes.
- **Administrador del servicio**: Persona que accede y administra suscripciones y proyectos de desarrollo en Azure Enterprise Portal.
- **Suscripción**: Representa una suscripción de Azure Enterprise Portal y es un contenedor de servicios de Azure administrados por el mismo administrador de servicios.
- **Cuenta profesional o educativa**: Para organizaciones que han configurado Active Directory con Federación en la nube y todas las cuentas están en un solo inquilino.

### <a name="enrollment-statuses"></a>Estados de inscripción

- **Nuevo**: Este estado se asigna a una inscripción que se creó hace menos de 24 horas y que se actualizará a un estado pendiente en un plazo máximo de 24 horas.
- **Pending**: El administrador de la inscripción debe iniciar sesión en Azure Enterprise Portal. Una vez que inicia sesión, la inscripción cambiará a estado Activa.
- **Activa**: La inscripción está activa, y se pueden crear cuentas y suscripciones en Azure Enterprise Portal. La inscripción permanecerá activa hasta la fecha de finalización del Contrato Enterprise.
- **Período extendido indefinido**: Un período extendido indefinido tiene lugar después de que se haya alcanzado la fecha de finalización del Contrato Enterprise. Permite a los clientes de Azure EA que participan en el período extendido seguir usando los servicios de Azure indefinidamente al final de su Contrato Enterprise.

   Antes de que la inscripción de Azure EA alcance la fecha de finalización del Contrato Enterprise, el administrador de inscripciones debe decidir entre las opciones siguientes:

  - Renovar la inscripción mediante la adición de un pago por adelantado adicional.
  - Transferirla a una nueva inscripción.
  - Migrar al Programa de suscripción en línea de Microsoft (MOSP).
  - Confirmar la deshabilitación de todos los servicios asociados con la inscripción.
- **Expirada**: El cliente de Azure EA no participa en el período extendido, y la inscripción de Azure EA ha alcanzado la fecha de finalización del Contrato Enterprise. La inscripción expirará, y se deshabilitarán todos los servicios asociados.
- **Transferida**: Las inscripciones en las que se hayan transferido a una nueva inscripción todas las cuentas y servicios asociados aparecen con estado Transferida.
  >[!NOTE]
  > Las inscripciones no se transfieren automáticamente si se genera un nuevo número de inscripción en el momento de la renovación. Debe incluir el número de inscripción anterior en la solicitud de renovación para facilitar la transferencia automática.

## <a name="get-started-on-azure-ea---faq"></a>Introducción a Azure EA: preguntas más frecuentes

En esta sección se proporcionan detalles sobre las preguntas típicas que han formulado los clientes durante el proceso de incorporación.  

### <a name="can-i-associate-my-existing-azure-account-to-azure-ea-enrollment"></a>¿Puedo asociar mi cuenta de Azure existente con la inscripción de Azure EA?

Sí. Todas las suscripciones de Azure en las que usted es el propietario de la cuenta se convertirán al Contrato Enterprise. Se incluyen las suscripciones que usan el crédito mensual, como Visual Studio, AzurePass, MPN, BizSpark, etc. Perderá el crédito mensual al convertir dichas suscripciones.

### <a name="i-accidentally-associated-my-existing-azure-account-with-azure-ea-enrollment-as-a-result-i-lost-my-monthly-credit-can-i-get-my-monthly-credit-back"></a>He asociado por accidente mi cuenta de Azure existente con una inscripción de Azure EA. Como resultado, perdí mi crédito mensual. ¿Puedo recuperar el crédito mensual?

Si ha iniciado sesión como propietario de la cuenta de Azure EA con las mismas credenciales que la suscripción a Visual Studio, puede recuperar la ventaja de Azure en su suscripción a Visual Studio individual mediante una de las siguientes acciones:

- Elimine el propietario de la cuenta de Azure Enterprise Portal después de quitar o mover las suscripciones de Azure asociadas. A continuación, regístrese para obtener las ventajas individuales de Azure en Visual Studio nuevamente.
- Elimine el suscriptor de Visual Studio del sitio de administración en VLSC y reasigne la suscripción a una cuenta, esta vez, con otras credenciales. A continuación, regístrese para obtener las ventajas individuales de Azure en Visual Studio nuevamente.

### <a name="what-type-of-subscription-should-i-create"></a>¿Qué tipo de suscripción debo crear?

Azure Enterprise Portal ofrece dos tipos de suscripciones para clientes empresariales:

- Microsoft Azure Enterprise, que es ideal para:
  - Todos los usos de producción
  - Mejores precios según el gasto en infraestructura

  Para obtener más información, [póngase en contacto con el personal de ventas de Azure](https://azure.microsoft.com/pricing/enterprise-agreement/).

- Desarrollo/pruebas - Enterprise, que es ideal para:
  - Todas las cargas de trabajo de desarrollo y pruebas del equipo
  - Cargas de trabajo de medianas a pesadas para desarrollo y pruebas individuales
  - Acceso a imágenes especiales de MSDN y tarifas de servicio preferentes

  Para más información, consulte la [oferta de Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/).

### <a name="is-it-possible-to-transfer-subscription-ownership-to-another-account"></a>¿Es posible transferir la propiedad de la suscripción a otra cuenta?

Sí, puede transferir la propiedad de la suscripción a una cuenta diferente. Por ejemplo, si la cuenta A tiene tres suscripciones, el administrador de empresa puede transferir una suscripción a la cuenta B, una a la cuenta C y otra a la cuenta D. O bien, puede transferir todas las suscripciones a la cuenta E.

Para transferir suscripciones:

1. En Azure Enterprise Portal, seleccione **Administrar** > **Cuenta**.
1. Mantenga el mouse sobre **Cuenta** en el extremo derecho para ver las opciones **Transferir propiedad** (icono de persona) y **Transferir suscripción** (icono de lista). Estas opciones solo son visibles para las cuentas activas.

### <a name="my-subscription-name-is-the-same-as-the-offer-name-should-i-change-the-subscription-name-to-something-meaningful-to-my-organization"></a>El nombre de mi suscripción es el mismo que el nombre de la oferta. ¿Debería cambiar el nombre de la suscripción a algo que sea significativo para mi organización?

Al crear una suscripción, el nombre toma como valor predeterminado el tipo de oferta que elija. Se recomienda cambiar el nombre de la suscripción a algo que facilite el seguimiento de la suscripción.

Para cambiar el nombre:

1. Inicie sesión en [https://account.windowsazure.com](https://account.windowsazure.com).
1. Seleccione la lista de suscripciones.
1. Seleccione la suscripción que quiere editar.
1. Seleccione el icono **Administrar suscripción**.
1. Edite los detalles de la suscripción.

### <a name="how-can-i-track-costs-incurred-by-a-cost-center"></a>¿Cómo puedo realizar un seguimiento de los costos en los que incurre un centro de costos?

Para realizar un seguimiento de los costos por centro de costos, debe definir el centro de costos en cualquiera de los siguientes niveles:

- department
- Cuenta
- Suscripción

En función de sus necesidades, puede usar el mismo centro de costos para realizar un seguimiento del uso y los costos asociados a un centro de costos en particular.

Por ejemplo, para realizar un seguimiento de los costos de un proyecto especial en el que participan varios departamentos, puede que desee definir el centro de costos en un nivel de suscripción para realizar el seguimiento del uso y de los costos.

No puede definir un centro de costos en el nivel de servicio. Si desea realizar un seguimiento del uso en el nivel de servicio, puede usar la característica _Etiqueta_ disponible en el nivel de servicio.

### <a name="how-do-i-track-usage-and-spend-by-different-departments-in-my-organization"></a>¿Cómo puedo realizar un seguimiento del uso y el gasto de los distintos departamentos de mi organización?

Puede crear tantos departamentos como necesite en su inscripción de Azure EA. Para realizar el seguimiento del uso correctamente, asegúrese de no compartir las suscripciones entre departamentos.

Después de haber creado los departamentos y las suscripciones, puede ver los datos en el informe de uso. Esta información puede ayudarle a realizar un seguimiento del uso y a administrar los costos y el gasto a nivel de departamento.

También puede tener acceso a los datos de uso a través de la API de informes. Puede encontrar información detallada y ejemplos de código en [API REST de Azure Enterprise](https://docs.microsoft.com/azure/cost-management-billing/manage/ea-portal-rest-apis).

### <a name="can-i-set-a-spending-quota-and-get-alerts-as-i-approach-my-limit"></a>¿Puedo establecer una cuota de gastos y recibir alertas a medida que me acerque al límite?

Puede establecer una cuota de gastos a nivel de departamento, y el sistema le notificará automáticamente a medida que los límites de gastos lleguen al 50 %, 75 %, 90 % y 100 % de la cuota que defina.

Para definir la cuota de gastos, seleccione un departamento y, a continuación, seleccione el icono de edición. Después de editar los detalles del límite de gasto, seleccione **Guardar**.

### <a name="i-used-resource-groups-to-implement-rbac-and-track-usage-how-can-i-view-the-associated-usage-details"></a>Usé grupos de recursos para implementar RBAC y realizar un seguimiento del uso. ¿Cómo puedo ver los detalles de uso asociados?

Si usa _grupos de recursos_ y _etiquetas_, se realiza un seguimiento de esta información a nivel de servicio, y puede acceder a ella en el archivo de descarga de uso detallado (CSV). Consulte el [informe de uso de descarga](https://ea.azure.com/report/downloadusage) en Azure Enterprise Portal.

También puede tener acceso al uso a través de la API. Puede encontrar información detallada y ejemplos de código en [API REST de Azure Enterprise](https://docs.microsoft.com/azure/cost-management-billing/manage/ea-portal-rest-apis).

> [!NOTE]
> Solo puede aplicar etiquetas a recursos que admiten operaciones de Azure Resource Manager. Si creó una máquina virtual, una red virtual o un almacenamiento mediante el modelo de implementación clásica (por ejemplo, a través del portal clásico), no podrá aplicar una etiqueta a ese recurso. Debe volver a implementar estos recursos mediante Resource Manager para admitir el etiquetado. Todos los demás recursos admiten el etiquetado.

### <a name="can-i-perform-analyses-using-power-bi"></a>¿Puedo realizar análisis mediante Power BI?

Sí. Con el paquete de contenido de Microsoft Azure Enterprise para Power BI, puede:

- Importar y analizar rápidamente el consumo de Azure para la inscripción de empresa.
- Averiguar qué departamento, cuenta o suscripción consumió el mayor uso.
- Obtener información sobre qué servicio usó más su organización.
- Realizar un seguimiento de las tendencias de gasto y uso.

Para usar Power BI:

1. Vaya al sitio web de Power BI.
1. Inicie sesión con una cuenta profesional o educativa válida.

   La cuenta profesional o educativa puede ser la misma o distinta de la que se usa para tener acceso a la inscripción mediante Azure Enterprise Portal.
1. En el panel de servicios, seleccione el elemento Microsoft Azure Enterprise y seleccione **Conectar**.
1. En la pantalla **Conectar con Azure Enterprise**, escriba:
    - Dirección URL del entorno de Azure: [https://ea.azure.com](https://ea.azure.com)
    - Número de meses: entre 1 y 36
    - Número de inscripción: el número de inscripción
1. Seleccione **Next** (Siguiente).
1. Escriba la clave de la API en el cuadro **Clave de cuenta**.

   Puede encontrar la clave de la API en Azure Enterprise Portal. Mire en la pestaña **Descargar uso** y, después, seleccione **API Access Key** (Clave de acceso de la API). Cópiela y, a continuación, pegue la clave en el cuadro **Clave de cuenta** en Power BI.

En función del tamaño del conjunto de datos, los datos pueden tardar entre 5 y 30 minutos en cargarse en Power BI.

Los informes de Power BI están disponibles para clientes directos, asociados e indirectos de Azure EA que puedan ver la información de facturación.

## <a name="next-steps"></a>Pasos siguientes

- Los administradores de Azure Enterprise Portal deben leer [Administración del portal del Contrato Enterprise de Azure](ea-portal-administration.md) para obtener información sobre las tareas administrativas comunes.
- Si necesita ayuda para solucionar problemas con Azure Enterprise Portal, consulte [Solución de problemas de acceso al portal de Azure EA](ea-portal-troubleshoot.md).
- Para obtener una guía de incorporación de Azure EA, consulte la [Guía de incorporación de Azure EA](https://ea.azure.com/api/v3Help/v2AzureEAOnboardingGuide).
