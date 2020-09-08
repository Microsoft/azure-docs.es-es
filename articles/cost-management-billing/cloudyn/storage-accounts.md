---
title: Configuración de cuentas de almacenamiento para Cloudyn en Azure
description: En este artículo se describe cómo configurar cuentas de almacenamiento de Azure y cubos de almacenamiento de AWS para Cloudyn.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: secdec18
ROBOTS: NOINDEX
ms.openlocfilehash: 130542833f2f5e0d4ae63cfa7f681b390e46fa1e
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689667"
---
# <a name="configure-storage-accounts-for-cloudyn"></a>Configuración de cuentas de almacenamiento para Cloudyn

<!--- intent: As a Cloudyn user, I want to configure Cloudyn to use my cloud service provider storage account to store my reports. -->

Puede guardar los informes de Cloudyn en el portal de Cloudyn, en el almacenamiento de Azure o en cubos de almacenamiento de AWS. El almacenamiento de los informes en el portal de Cloudyn es gratuito. Sin embargo, guardar los informes en el almacenamiento del proveedor de servicios en la nube es opcional y puede conllevar costos adicionales. Este artículo le ayuda a configurar cuentas de almacenamiento de Azure y cubos de almacenamiento de Amazon Web Services (AWS) para almacenar los informes.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Debe tener una cuenta de almacenamiento de Azure o un cubo de almacenamiento de Amazon.

Si no tiene una cuenta de almacenamiento de Azure, debe crear una. Para más información sobre la creación de una cuenta de almacenamiento de Azure, consulte [Crear una cuenta de almacenamiento](../../storage/common/storage-account-create.md).

Si no tiene un cubo de servicio de almacenamiento simple (S3) de AWS, debe crear uno. Para más información sobre cómo crear un cubo S3, consulte [Crear un bucket](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html).

## <a name="configure-your-azure-storage-account"></a>Configuración de la cuenta de almacenamiento de Azure

La configuración del almacenamiento de Azure para que lo use Cloudyn es sencilla. Recopile detalles sobre la cuenta de almacenamiento y cópielos en el portal de Cloudyn.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. Haga clic en **Todos los servicios**, seleccione **Cuentas de almacenamiento**, desplácese a la cuenta de almacenamiento que desea usar y selecciónela.
3. En la página de la cuenta de almacenamiento, en **Configuración**, haga clic en **Claves de acceso**.
4. Copie la información de los cuadros **Nombre de la cuenta de almacenamiento** y **Cadena de conexión** en Key1.  
   ![Copia de la cadena de conexión y el nombre de la cuenta de almacenamiento](./media/storage-accounts/azure-storage-access-keys.png)  
5. Abra el portal de Cloudyn desde Azure Portal o vaya a [https://azure.cloudyn.com](https://azure.cloudyn.com) e inicie sesión.
6. Haga clic en el símbolo de engranaje y seleccione **Administración de informes de almacenamiento**.
7. Haga clic en **Agregar nuevo +** y asegúrese de que Microsoft Azure está seleccionado. Pegue el nombre de la cuenta de almacenamiento de Azure en el área **Nombre**. Pegue la **cadena de conexión** en el área correspondiente. Escriba un nombre de contenedor y haga clic en **Guardar**.  
   ![Pegado del nombre y la cadena de conexión de Azure Storage en el cuadro de texto Agregar un nuevo almacenamiento de informes](./media/storage-accounts/azure-cloudyn-storage.png)

   La nueva entrada de almacenamiento de informe de Azure aparece en la lista de cuentas de almacenamiento.  
    ![Nueva entrada de almacenamiento de informe de Azure en la lista](./media/storage-accounts/azure-storage-entry.png)


Ahora puede guardar informes en el almacenamiento de Azure. En cualquier informe, haga clic en **Actions** (Acciones) y, a continuación, seleccione **Schedule report** (Programar informe). Asigne un nombre al informe y luego agregue su propia dirección URL o use la dirección URL creada automáticamente. Seleccione **Save to storage** (Guardar en el almacenamiento) y elija la cuenta de almacenamiento. Escriba un prefijo que se anexe al nombre de archivo del informe. Seleccione el formato de archivo CSV o JSON y guarde el informe.

## <a name="configure-an-aws-storage-bucket"></a>Configuración de un cubo de almacenamiento de AWS

Cloudyn usa las credenciales de AWS existentes: Usuario o rol, para guardar los informes en el cubo. Para probar el acceso, Cloudyn intenta guardar un archivo de texto pequeño en el cubo con el nombre de archivo _check-bucket-permission.txt_.

Al cubo se le proporciona el rol o usuario de Cloudyn con el permiso PutObject. Luego, se usa un cubo existente o se crea uno nuevo para guardar los informes. Por último, decida cómo administrar la clase de almacenamiento, establezca reglas de ciclo de vida o quite los archivos innecesarios.

###  <a name="assign-permissions-to-your-aws-user-or-role"></a>Asignación de permisos a su rol o usuario de AWS

Al crear una nueva directiva, proporcione los permisos exactos necesarios para guardar un informe en un cubo S3.

1. Inicie sesión en la consola de AWS y seleccione **Services** (Servicios).
2. Seleccione **IAM** en la lista de servicios.
3. Seleccione **Policies** (Directivas) en el lado izquierdo de la consola y luego haga clic en **Create Policy** (Crear directiva).
4. Haga clic en la pestaña **JSON**.
5. La siguiente directiva permite guardar un informe en un cubo S3. Copie el siguiente ejemplo de directiva y péguelo en la pestaña **JSON**. Reemplace &lt;bucketname&gt; por el nombre del cubo.

   ```json
   {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid":  "CloudynSaveReport2S3",
        "Effect":      "Allow",
        "Action": [
          "s3:PutObject"
        ],
        "Resource": [
          "arn:aws:s3:::<bucketname>/*"
        ]
      }
    ]
   }
   ```

6. Haga clic en **Review policy** (Revisar directiva).  
    ![Directiva JSON de AWS que muestra información de ejemplo](./media/storage-accounts/aws-policy.png)  
7. En la página Review policy (Revisar directiva), escriba un nombre para la directiva. Por ejemplo, _CloudynSaveReport2S3_.
8. Haga clic en **Create policy** (Crear directiva).

### <a name="attach-the-policy-to-a-cloudyn-role-or-user-in-your-account"></a>Asociación de la directiva a un rol o usuario de Cloudyn en su cuenta

Para asociar la nueva directiva, abra la consola de AWS y edite el rol o usuario de Cloudyn.

1. Inicie sesión en la consola de AWS, seleccione **Services** (Servicios) y luego seleccione **IAM** en la lista de servicios.
2. Seleccione **Roles** o **Users** (Usuarios) en el lado izquierdo de la consola.

**Para roles:**

  1. Haga clic en el nombre de rol de Cloudyn.
  2. En la pestaña **Permissions** (Permisos), haga clic en **Attach Policy** (Asociar directiva).
  3. Busque la directiva que ha creado, selecciónela y luego haga clic en **Attach Policy** (Asociar directiva).
    ![Directiva de ejemplo conectada al rol de Cloudyn](./media/storage-accounts/aws-attach-policy-role.png)

**Para usuarios:**

1. Seleccione el usuario de Cloudyn.
2. En la pestaña **Permissions** (Permisos), haga clic en **Add permissions** (Agregar permisos).
3. En la sección **Grant Permission** (Conceder permiso), seleccione **Attach existing policies directly** (Asociar directivas existentes directamente).
4. Busque la directiva que ha creado, selecciónela y luego haga clic en **Next: Review** (Siguiente: revisar).
5. En la página Add permissions to role name (Agregar permisos al nombre de rol), haga clic en **Add permissions** (Agregar permisos).  
    ![Directiva de ejemplo conectada al usuario de Cloudyn](./media/storage-accounts/aws-attach-policy-user.png)


### <a name="optional-set-permission-with-bucket-policy"></a>Opcional: establecimiento de permisos con directiva de cubo

También puede establecer permisos para crear informes en el cubo S3 mediante una directiva de cubo. En la vista clásica de S3:

1. Cree o seleccione un cubo existente.
2. Seleccione la pestaña **Permissions** (Permisos) y haga clic en **Bucket policy** (Directiva de cubo).
3. Copie y pegue el siguiente ejemplo de directiva. Reemplace &lt;bucket\_name&gt; y &lt;Cloudyn\_principle&gt; por el ARN del cubo. Reemplace el ARN del rol o usuario que Cloudyn usa.

   ```
   {
   "Id": "Policy1485775646248",
   "Version": "2012-10-17",
   "Statement": [
    {
      "Sid": "SaveReport2S3",
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": "<bucket_name>/*",
      "Principal": {
        "AWS": [
          "<Cloudyn_principle>"
        ]
      }
    }
   ]
   }
   ```

4. En el editor de directivas de cubos, haga clic en **Save** (Guardar).

### <a name="add-aws-report-storage-to-cloudyn"></a>Incorporación de almacenamiento de informes de AWS a Cloudyn

1. Abra el portal de Cloudyn desde Azure Portal o vaya a [https://azure.cloudyn.com](https://azure.cloudyn.com) e inicie sesión.
2. Haga clic en el símbolo de engranaje y seleccione **Administración de informes de almacenamiento**.
3. Haga clic en **Agregar nuevo +** y asegúrese de que AWS está seleccionado.
4. Seleccione una cuenta y un cubo de almacenamiento. El nombre del cubo de almacenamiento de AWS se rellena automáticamente.  
    ![Información de ejemplo en el cuadro de texto Agregar un nuevo almacenamiento de informes](./media/storage-accounts/aws-cloudyn-storage.png)  
5. Haga clic en **Guardar** y luego en **Aceptar**.

    La nueva entrada de almacenamiento de informe de AWS aparece en la lista de cuentas de almacenamiento.  
    ![Nueva entrada de almacenamiento de informes de AWS en la lista de cuentas de almacenamiento](./media/storage-accounts/aws-storage-entry.png)


Ahora puede guardar informes en el almacenamiento de Azure. En cualquier informe, haga clic en **Actions** (Acciones) y, a continuación, seleccione **Schedule report** (Programar informe). Asigne un nombre al informe y luego agregue su propia dirección URL o use la dirección URL creada automáticamente. Seleccione **Save to storage** (Guardar en el almacenamiento) y elija la cuenta de almacenamiento. Escriba un prefijo que se anexe al nombre de archivo del informe. Seleccione el formato de archivo CSV o JSON y guarde el informe.

## <a name="next-steps"></a>Pasos siguientes

- Revise [Descripción de los informes de Cloudyn](understanding-cost-reports.md) para más información sobre la estructura básica y las funciones de los informes de Cloudyn.
