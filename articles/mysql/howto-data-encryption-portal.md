---
title: Cifrado de datos de Azure Database for MySQL en Azure Portal
description: Aprenda a configurar y administrar el cifrado de datos de Azure Database for MySQL mediante Azure Portal.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 01/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 097e07f3f39fc647d311af5e4eface4a7c6b3675
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498487"
---
# <a name="data-encryption-for-azure-database-for-mysql-by-using-the-azure-portal"></a>Cifrado de datos para Azure Database for MySQL mediante Azure Portal

Aprenda a usar Azure Portal para configurar y administrar el cifrado de datos de Azure Database for MySQL.

## <a name="prerequisites-for-azure-cli"></a>Requisitos previos de la CLI de Azure

* Debe tener una suscripción de Azure y ser un administrador en esa suscripción.
* En Azure Key Vault, cree un almacén de claves y una clave que se usará como clave administrada por el cliente.
* Para ello, el almacén de claves debe tener las siguientes propiedades:
  * [Eliminación temporal](../key-vault/general/soft-delete-overview.md)

    ```azurecli-interactive
    az resource update --id $(az keyvault show --name \ <key_vault_name> -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
    ```

  * [Protegido contra purgas](../key-vault/general/soft-delete-overview.md#purge-protection)

    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
    ```

* La clave debe tener los siguientes atributos para que se pueda usar como clave administrada por el cliente:
  * Sin fecha de expiración
  * No deshabilitado
  * Poder realizar operaciones get, wrap key, unwrap key

## <a name="set-the-right-permissions-for-key-operations"></a>Establecer los permisos adecuados para las operaciones de clave

1. En Key Vault, seleccione **Directivas de acceso** > **Agregar directiva de acceso**.

   ![Captura de pantalla de Key Vault, con las opciones Directivas de acceso y Agregar directiva de acceso resaltadas](media/concepts-data-access-and-security-data-encryption/show-access-policy-overview.png)

2. En **Permisos clave** seleccione **Get** (Obtener), **Wrap** (Encapsular), **Unwrap** (Desencapsular) y **Principal** (Entidad de seguridad), que es el nombre del servidor MySQL. Si no se encuentra la entidad de seguridad del servidor en la lista de entidades de seguridad existentes, debe registrarla. Se le pedirá que registre la entidad de seguridad del servidor cuando intente configurar el cifrado de datos por primera vez y se produzca un error.

   ![Introducción a la directiva de acceso](media/concepts-data-access-and-security-data-encryption/access-policy-wrap-unwrap.png)

3. Seleccione **Guardar**.

## <a name="set-data-encryption-for-azure-database-for-mysql"></a>Establecimiento del cifrado de datos para Azure Database for MySQL

1. En Azure Database for MySQL, seleccione **Cifrado de datos** para configurar la clave administrada por el cliente.

   ![Captura de pantalla de Azure Database for MySQL, con Cifrado de datos resaltado](media/concepts-data-access-and-security-data-encryption/data-encryption-overview.png)

2. Puede seleccionar un almacén de claves y un par de claves o escribir un identificador de clave.

   ![Captura de pantalla de Azure Database for MySQL, con las opciones de cifrado de datos resaltadas](media/concepts-data-access-and-security-data-encryption/setting-data-encryption.png)

3. Seleccione **Guardar**.

4. Para asegurarse de que todos los archivos (incluidos los archivos temporales) están totalmente cifrados, reinicie el servidor.

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>Uso del cifrado de datos con servidores de restauración o réplica

Después de cifrar Azure Database for MySQL con la clave administrada de un cliente almacenada en Key Vault, también se cifra cualquier copia recién creada del servidor. Puede crear esta nueva copia mediante una operación de restauración local o geográfica, o por medio de una operación de réplica (local o entre regiones). De modo que, con un servidor MySQL cifrado, puede usar los siguientes pasos para crear un servidor restaurado cifrado.

1. En el servidor, seleccione **Información general** > **Restaurar**.

   ![Captura de pantalla de Azure Database for MySQL, las opciones información general y Restaurar resaltadas](media/concepts-data-access-and-security-data-encryption/show-restore.png)

   O bien, en el caso de un servidor habilitado para la replicación, en el encabezado **Configuración**, seleccione **Replicación**.

   ![Captura de pantalla de Azure Database for MySQL, con la opción Replicación resaltada](media/concepts-data-access-and-security-data-encryption/mysql-replica.png)

2. Una vez completada la operación de restauración, el nuevo servidor creado se cifra con la clave del servidor principal. Sin embargo, las características y opciones del servidor están deshabilitadas y el servidor está inaccesible. Esto evita la manipulación de los datos, ya que la identidad del nuevo servidor todavía no tiene permiso para acceder al almacén de claves.

   ![Captura de pantalla de Azure Database for MySQL, con estado Inaccesible resaltado](media/concepts-data-access-and-security-data-encryption/show-restore-data-encryption.png)

3. Para que el servidor sea accesible, vuelva a validar la clave en el servidor restaurado. Seleccione **Cifrado de datos** > **Revalidate key** (Volver a validar la clave).

   > [!NOTE]
   > Se producirá un error en el primer intento de volver a realizar la validación, ya que la entidad de servicio debe tener acceso al almacén de claves. Para generar la entidad de servicio, seleccione **Revalidate key** (Volver a validar la clave). Se muestra un error, pero se genera la entidad de servicio. A partir de ese momento, consulte [estos pasos](#set-the-right-permissions-for-key-operations) anteriormente en este artículo.

   ![Captura de pantalla de Azure Database for MySQL, con el paso de nueva validación resaltado](media/concepts-data-access-and-security-data-encryption/show-revalidate-data-encryption.png)

   Tendrá que conceder al almacén de claves acceso al nuevo servidor.

4. Después de registrar la entidad de servicio, vuelva a validar la clave otra vez; el servidor reanudará su funcionalidad normal.

   ![Captura de pantalla de Azure Database for MySQL, que muestra la funcionalidad restaurada](media/concepts-data-access-and-security-data-encryption/restore-successful.png)

## <a name="next-steps"></a>Pasos siguientes

 Para más información sobre el cifrado de datos, consulte [Cifrado de datos de Azure Database for MySQL con clave administrada por el cliente](concepts-data-encryption-mysql.md).
