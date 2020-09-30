---
title: Garantía de validación del cifrado de datos de Azure Database for MySQL
description: Aprenda a validar el cifrado de datos de Azure Database for MySQL mediante la clave administrada por los clientes.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 04/28/2020
ms.openlocfilehash: e7b747bd2babeeccb210bb30c3eb28f22d4befe7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905803"
---
# <a name="validating-data-encryption-for-azure-database-for-mysql"></a>Validación del cifrado de datos de Azure Database for MySQL

La información de este artículo le ayuda a validar que el cifrado de datos con la clave administrada por el cliente para Azure Database for MySQL funciona según lo previsto.

## <a name="check-the-encryption-status"></a>Comprobación del estado de cifrado

### <a name="from-portal"></a>Desde el portal

1. Si quiere comprobar que la clave del cliente se usa para el cifrado, siga estos pasos:

    * En Azure Portal, vaya a **Azure Key Vault** -> **Claves**.
    * Seleccione la clave que se usa para el cifrado del servidor.
    * Establezca el estado de la clave **Habilitado** en **No**.
  
       Al cabo de un tiempo (**unos 15 minutos**), el **estado** del servidor de Azure Database for MySQL debe ser **Inaccesible**. Cualquier operación de E/S realizada en el servidor generará un error, lo que confirma que el servidor está cifrado realmente con la clave del cliente y que la clave no es válida actualmente.
    
       Para que el estado del servidor sea **Disponible**, puede volver a validar la clave. 
    
    * Establezca el estado de la clave en Key Vault en **Sí**.
    * En el campo **Cifrado de datos** del servidor, seleccione **Volver a validar la clave**.
    * Cuando la revalidación de la clave sea correcta, el valor de **Estado** del servidor cambiará a **Disponible**

2. En Azure Portal, si puede garantizar que se ha establecido la clave de cifrado, los datos se cifran con la clave del cliente usada en Azure Portal.

  :::image type="content" source="media/concepts-data-access-and-security-data-encryption/byok-validate.png" alt-text="Introducción a la directiva de acceso":::

### <a name="from-cli"></a>Desde la CLI

1. Se puede usar el comando *az CLI* para validar los recursos de clave que se usan para el servidor de Azure Database for MySQL.

    ```azurecli-interactive
   az mysql server key list --name  '<server_name>'  -g '<resource_group_name>'
    ```

    Cuando un servidor no tiene establecido el cifrado de datos, este comando produce un conjunto vacío [].

### <a name="azure-audit-reports"></a>Informes de auditoría de Azure

También se puede revisar si los [informes de auditoría](https://servicetrust.microsoft.com) proporcionan información sobre el cumplimiento de los estándares de protección de datos y los requisitos normativos.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el cifrado de datos, consulte [Cifrado de datos de Azure Database for MySQL con clave administrada por el cliente](concepts-data-encryption-mysql.md).