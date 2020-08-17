---
title: Configuración de la integración de AWS con Azure Cost Management
description: En este artículo se le guiará a través de la instalación y configuración de la integración de informes de uso y costo de AWS con Azure Cost Management.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.reviewer: matrive
ms.openlocfilehash: 293fbe49572b8eacc95331de909ed5a2a00441b6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290842"
---
# <a name="set-up-and-configure-aws-cost-and-usage-report-integration"></a>Instalar y configurar la integración de informes de uso y costo de AWS

Con la integración de informes de uso y costo (CUR) de Amazon Web Services (AWS), supervisa y controla los gastos de AWS en Azure Cost Management. La integración permite tener una sola ubicación en Azure Portal donde supervisar y controlar el gasto de Azure y AWS. En este artículo se explica cómo instalar la integración y cómo configurarla para usar las características de Azure Cost Management para analizar los costos y revisar los presupuestos.

Cost Management procesa el informe de uso y costo de AWS almacenado en un bucket de S3 con sus credenciales de acceso de AWS para obtener las definiciones del informe y descargar archivos GZIP CSV del informe.

Vea el vídeo [Configuración de conectores para AWS en Cost Management](https://www.youtube.com/watch?v=Jg5KC1cx5cA) para más información sobre cómo configurar la integración de informes de AWS. Para ver otros vídeos, visite el [canal de YouTube de Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/Jg5KC1cx5cA]

## <a name="create-a-cost-and-usage-report-in-aws"></a>Crear un informe de uso y costo en AWS

Usar un informe de uso y costo es la manera recomendada de AWS para recopilar y procesar los costos de AWS. Para obtener más información, consulte la documentación de [AWS Cost and Usage Report](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-reports-costusage.html) (Informe de uso y costo de AWS).

Use la página **Informes de uso y costo** de la consola de Administración de costos y facturación de AWS para crear un informe de uso y costo con los pasos siguientes:

1. Inicie sesión en la consola de Administración de AWS y abra la [Consola de administración de costos y facturación](https://console.aws.amazon.com/billing).
2. Seleccione **Cost & Usage Reports** (Informes de costo y uso) en el panel de navegación.
3. Seleccione **Crear informe**.
4. En **Nombre del informe**, escriba un nombre para el informe.
5. En **Detalles de informes adicionales**, seleccione **Incluir ID de recurso**.
6. En **Configuración de actualización de datos**, seleccione si desea que el informe de uso y costo de AWS se actualice si AWS aplica reembolsos, créditos o honorarios de soporte técnico a su cuenta después de finalizar la factura. Cuando se actualiza un informe, se carga un nuevo informe a Amazon S3. Se recomienda dejar la opción seleccionada.
7. Seleccione **Next** (Siguiente).
8. En **Bucket de S3**, elija **Configurar**.
9. En el cuadro de diálogo Configurar bucket de S3, realice una de las siguientes tareas:
    1. Seleccione un bucket existente en la lista desplegable y elija **Siguiente**.
    2. Escriba un nombre de bucket y la región donde desea crear un nuevo bucket y elija **Siguiente**.
10.    Seleccione **Confirmo que esta política es correcta** y haga clic en **Guardar**.
11.    (Opcional) En Prefijo de la ruta del informe, escriba el prefijo de la ruta de acceso del informe que quiera antes del nombre del informe.
Si no especifica un prefijo, el prefijo predeterminado es el nombre que especificó para el informe. El intervalo de fechas tiene el formato `/report-name/date-range/`.
12. Para **Detalle del tiempo**, elija **Por hora**.
13.    En **Control de versiones de informe**, elija si desea que cada versión del informe sobrescriba la versión anterior o si desea informes nuevos adicionales.
14. En **Habilitar la integración de datos de informe para**, no se requiere ninguna selección.
15. Para **Tipo de compresión**, seleccione **GZIP**.
16. Seleccione **Next** (Siguiente).
17. Una vez que haya revisado la configuración para el informe, seleccione **Revisar y completar**.

    Anote el nombre del informe. Lo usará en pasos posteriores.

Pueden transcurrir hasta 24 horas para que AWS empiece a entregar informes en el bucket de Amazon S3. Cuando se inicia la entrega, AWS actualiza los archivos de informe de uso y costo de AWS al menos una vez al día. Puede seguir configurando un entorno de AWS sin tener que esperar la entrega para empezar.

## <a name="create-a-role-and-policy-in-aws"></a>Crear un rol y una directiva en AWS

Azure Cost Management tiene acceso al bucket de S3 donde se encuentra el informe de uso y costo varias veces al día. El servicio necesita acceso a las credenciales para verificar si hay nuevos datos. Se crea un rol y una directiva en AWS para permitir que Cost Management acceda a ellas.

Para habilitar el acceso basado en roles para una cuenta de AWS en Cost Management, se crea el rol en la consola de AWS. Debe tener el _Rol de ARN_ e _Id. externo_ de la consola AWS. Más adelante, se usan en la página **Crear un conector de AWS**  en Cost Management.

Use el Asistente para crear un nuevo rol:

1. Inicie sesión en la consola de AWS y seleccione **Servicios**.
2. En la lista de servicios, seleccione **IAM**.
3. Seleccione **Roles** y, a continuación, seleccione **Crear un rol**.
4. En la siguiente página, seleccione **Otra cuenta de AWS**.
5. En **ID de cuenta**, escriba **432263259397**.
6. En **Opciones**, seleccione **Requerir un ID externo (practica recomendada si un tercero va a adoptar este rol)** .
7. En **Id. externo**, escriba el identificador externo, que es un código de acceso compartido entre el rol de AWS y Azure Cost Management. También se usa el mismo id. externo en la página **Nuevo conector** de Cost Management. Microsoft recomienda que use una directiva de código de acceso segura al especificar el identificador externo.

    > [!NOTE]
    > No cambie la selección de **Require MFA (Requerir MFA)** . Debe permanecer desactivado.
8. Seleccione **Siguiente: Permisos**.
9. Seleccione **Crear una directiva**. Se abre una nueva pestaña del explorador. Es donde se crea una directiva.
10. Seleccione **Elegir un servicio**.

Configure el permiso para el informe de uso y costo:

1. Escriba **Cost and Usage Report** (Informe de uso y costo).
2. Seleccione **Nivel de acceso** > **Lectura** > **DescribeReportDefinitions**. Este paso permite a Cost Management leer qué informes CUR están definidos y determinar si coinciden con el requisito previo de definición de informe.
3. Seleccione **Añadir permisos adicionales**.

Configure el permiso para el bucket de S3 y objetos:

1. Seleccione **Elegir un servicio**.
2. Escriba **S3**.
3. Seleccione **Nivel de acceso** > **Enumeración** > **ListBucket**. Esta acción obtiene la lista de objetos en el bucket de S3.
4. Seleccione **Nivel de acceso** > **Lectura** > **GetObject**. Esta acción permite la descarga de archivos de facturación.
5. Seleccione **Recursos**.
6. Seleccione **bucket: Agregar ARN**.
7. En **Bucket name** (Nombre de bucket), escriba el bucket que se usa para almacenar los archivos CUR.
8. Seleccione **objeto: Agregar ARN**.
9. En **Bucket name** (Nombre de bucket), escriba el bucket que se usa para almacenar los archivos CUR.
10. En **Nombre de objeto**, seleccione **Cualquiera**.
11. Seleccione **Añadir permisos adicionales**.

Configure el permiso para Cost Explorer:

1. Seleccione **Elegir un servicio**.
2. Escriba **Cost Explorer Service**.
3. Seleccione **Todas las acciones de Cost Explorer Service (ce:\*)** . Esta acción valida que la colección es correcta.
4. Seleccione **Añadir permisos adicionales**.

Agregue el permiso para AWS Organizations:

1. Escriba **Organizations**.
2. Seleccione **Nivel de acceso** > **Enumeración** > **ListAccounts**. Esta acción obtiene los nombres de las cuentas.
3. En **Revisar la directiva**, escriba un nombre para la nueva directiva. Compruebe que escribió la información correcta y, a continuación, seleccione **Crear directiva**.
4. Vuelva a la pestaña anterior y actualice la página web de su explorador. Busque la nueva directiva en la barra de búsqueda.
5. Seleccione **Siguiente: Review** (Siguiente: revisar).
6. Escriba un nombre para el nuevo rol. Compruebe que escribió la información correcta y, a continuación, seleccione **Crear rol**.

    Anote el rol ARN y el id. externo usados en los pasos anteriores cuando creó el rol. Los usará más adelante al configurar el conector de Azure Cost Management.

El JSON de la directiva debe ser similar al ejemplo siguiente. Reemplace _bucketname_ por el nombre del bucket de S3.

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
"organizations:ListAccounts",
             "ce:*",
             "cur:DescribeReportDefinitions"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::bucketname",
                "arn:aws:s3:::bucketname/*"
            ]
        }
    ]
}
```

## <a name="set-up-a-new-aws-connector-in-azure"></a>Configurar un nuevo conector de AWS en Azure

Use la siguiente información para crear un conector de AWS y comenzar a supervisar los costos de AWS:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a **Administración de costos + facturación** > **Cost Management**.
3. En **Configuración**, seleccione **Conectores en la nube (versión preliminar)** .  
    ![Ejemplo que muestra la configuración de los conectores en la nube (versión preliminar)](./media/aws-integration-setup-configure/cloud-connectors-preview01.png).
4. Seleccione **+ Agregar** en la parte superior de la página para crear un conector.
5. En la página **Crear un conector de AWS**, en **Nombre para mostrar**, escriba un nombre para el conector.  
    ![Ejemplo de la página para crear un conector de AWS](./media/aws-integration-setup-configure/create-aws-connector01.png)
6. Opcionalmente, seleccione el grupo de administración predeterminado. Almacenará todas las cuentas vinculadas detectadas. Se puede configurar más adelante.
7. En la sección **Facturación**, seleccione **Cargar automáticamente el 1 % en disponibilidad general** si quiere garantizar el funcionamiento continuado cuando expire la versión preliminar. Si selecciona la opción automática, debe seleccionar una suscripción de facturación.
8. Para **Rol de ARN**, escriba el valor que usó al configurar el rol en AWS.
9. Para **Id. externo**, escriba el valor que usó al configurar el rol en AWS.
10. Para **Nombre del informe**, escriba el nombre que creó en AWS.
11. Seleccione **Siguiente** y después **Crear**.

Los nuevos ámbitos de AWS, la cuenta consolidada de AWS y las cuentas vinculadas de AWS y sus datos de costo pueden tardar unas horas en aparecer.

Después de crear el conector, se recomienda asignarle control de acceso. Se asignan a los usuarios permisos para los ámbitos recién detectados: la cuenta consolidada de AWS y las cuentas vinculadas de AWS. El usuario que crea el conector es el propietario del conector, la cuenta consolidada y todas las cuentas vinculadas.

Al asignar permisos de conector a los usuarios después de que se produzca la detección no se asignan permisos a los ámbitos de AWS existentes. En su lugar, solo se asignan permisos a las nuevas cuentas vinculadas.

## <a name="take-additional-steps"></a>Pasos adicionales

- [Configure grupos de administración](../../governance/management-groups/overview.md#initial-setup-of-management-groups), si no lo ha hecho ya.
- Compruebe que se agregan nuevos ámbitos para el selector de ámbitos. Seleccione **Actualizar** para ver los datos más recientes.
- En la página **Conectores de nube**, seleccione el conector y seleccione **Ir a la cuenta de facturación** para asignar la cuenta vinculada a grupos de administración.

## <a name="manage-cloud-connectors"></a>Administración de los conectores de nube

Cuando selecciona un conector en la página **Conectores de nube**, puede:

- Seleccionar **Ir a la cuenta de facturación** para ver información de la cuenta consolidada de AWS.
- Seleccione **Access Control** para administrar la asignación de roles para el conector.
- Seleccione **Editar** para actualizar el conector. No se puede cambiar el número de cuenta de AWS, ya que aparece en el rol de ARN. Pero se puede crear un nuevo conector.
- Seleccione **Comprobar** para volver a ejecutar la prueba de comprobación y asegurarse de que Cost Management podrá recopilar datos mediante la configuración del conector.

![Ejemplo de lista de conectores creados de AWS](./media/aws-integration-setup-configure/list-aws-connectors.png)

## <a name="set-up-azure-management-groups"></a>Configurar grupos de administración de Azure

Coloque sus suscripciones de Azure y cuentas vinculadas de AWS en el mismo grupo de administración para crear una ubicación única en la que pueda ver la información de proveedor de toda la nube. Si aún no ha configurado el entorno de Azure con grupos de administración, consulte [Initial setup of management groups](../../governance/management-groups/overview.md#initial-setup-of-management-groups) (Configuración inicial de los grupos de administración).

Si quiere separar los costos, puede crear un grupo de administración que contenga solo las cuentas vinculadas de AWS.

## <a name="set-up-an-aws-consolidated-account"></a>Configuración de la cuenta consolidada de AWS

La cuenta consolidada de AWS combina la facturación y pago para varias cuentas de AWS. También actúa como una cuenta vinculada de AWS.

![Ejemplo de detalles de una cuenta consolidada de AWS](./media/aws-integration-setup-configure/aws-consolidated-account01.png)

En la página, haga lo siguiente:

- Seleccione **Actualizar** para la actualización masiva de la asociación de cuentas vinculadas de AWS con un grupo de administración.
- Seleccione **Access Control** para establecer la asignación de roles para el ámbito.

### <a name="permissions-for-an-aws-consolidated-account"></a>Permisos para una cuenta consolidada de AWS

De forma predeterminada, los permisos para una cuenta consolidada de AWS se establecen durante la creación de la cuenta, en función de los permisos de conector de AWS. El creador del conector es el propietario.

El nivel de acceso se administra mediante la página **Nivel de acceso** de la cuenta consolidada de AWS. Sin embargo, las cuentas vinculadas de AWS no heredan permisos para la cuenta consolidada de AWS.

## <a name="set-up-an-aws-linked-account"></a>Configuración de una cuenta vinculada de AWS

La cuenta vinculada de AWS es donde se crean y administran los recursos de AWS. Una cuenta vinculada también actúa como límite de seguridad.

En esta página puede realizar las siguientes acciones:

- Seleccione **Actualizar** para la actualización de la asociación de una cuenta vinculada de AWS con un grupo de administración.
- Seleccione **Access Control** para establecer una asignación de roles para el ámbito.

![Ejemplo de página de la cuenta vinculada de AWS](./media/aws-integration-setup-configure/aws-linked-account01.png)

### <a name="permissions-for-an-aws-linked-account"></a>Permisos para una cuenta vinculada de AWS

De forma predeterminada, los permisos para una cuenta vinculada de AWS se establecen durante la creación de la cuenta, en función de los permisos de conector de AWS. El creador del conector es el propietario. El nivel de acceso se administra mediante la página **Nivel de acceso** de la cuenta vinculada de AWS. Las cuentas vinculadas de AWS no heredan permisos de la cuenta consolidada de AWS.

Las cuentas vinculadas de AWS siempre heredan los permisos del grupo de administración al que pertenecen.

## <a name="next-steps"></a>Pasos siguientes

- Ahora que ha instalado y configurado la integración de informes de uso y costo de AWS, continúe con [Administración de los costos y el uso de AWS en Azure](aws-integration-manage.md).
- Si no está familiarizado con el análisis de costos, consulte [Guía de inicio rápido: Explore y analice los costos con Análisis de costos](quick-acm-cost-analysis.md).
- Si no está familiarizado con los presupuestos en Azure, consulte [Creación y administración de presupuestos de Azure](tutorial-acm-create-budgets.md).
