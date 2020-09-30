---
title: 'Cómo garantizar la validación de Azure Database for PostgreSQL: cifrado de datos'
description: Obtenga información acerca de cómo validar el cifrado de datos de Azure Database for PostgreSQL mediante la clave administrada de los clientes.
author: kummanish
ms.author: manishku
ms.service: postgresql
ms.topic: how-to
ms.date: 04/28/2020
ms.openlocfilehash: 0a1b8eaee0a483fb4ce09a33d3c7ca9fe6f980e1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90884336"
---
# <a name="validating-data-encryption-for-azure-database-for-postgresql"></a>Validación del cifrado de datos Azure Database for PostgreSQL

Este artículo le ayuda a validar que el cifrado de datos mediante la clave administrada por el cliente para Azure Database for PostgreSQL funciona según lo previsto.

## <a name="check-the-encryption-status"></a>Comprobación del estado de cifrado

### <a name="from-portal"></a>Desde el portal

1. Si quiere comprobar que la clave del cliente se usa para el cifrado, siga estos pasos:

    * En Azure Portal, vaya a **Azure Key Vault** -> **Claves**.
    * Seleccione la clave que se usa para el cifrado del servidor.
    * Establezca el estado de la clave **Habilitado** en **No**.
  
       Después de cierto tiempo (**unos 15 minutos**), el valor de **Estado** del servidor de Azure Database for PostgreSQL debería ser **Inaccesible**. Cualquier operación de E/S realizada en el servidor generará un error, lo que confirma que el servidor está cifrado realmente con la clave del cliente y que la clave no es válida actualmente.
    
        Para que el estado del servidor sea **Disponible**, puede volver a validar la clave. 
    
    * Establezca el estado de la clave en Key Vault en **Sí**.
    * En el campo **Cifrado de datos** del servidor, seleccione **Volver a validar la clave**.
    * Cuando la revalidación de la clave es correcta, el valor de **Estado** del servidor cambia a **Disponible**

2. En Azure Portal, si puede garantizar que se ha establecido la clave de cifrado, los datos se cifran con la clave del cliente usada en Azure Portal.

  :::image type="content" source="media/concepts-data-access-and-security-data-encryption/byok-validate.png" alt-text="Introducción a la directiva de acceso":::

### <a name="from-cli"></a>Desde la CLI

1. Se puede usar el comando *az CLI* para validar los recursos clave que se usan para el servidor de Azure Database for PostgreSQL.

    ```azurecli-interactive
   az postgres server key list --name  '<server_name>'  -g '<resource_group_name>'
    ```

    En el caso de un servidor sin el cifrado de datos establecido, este comando generará un conjunto vacío [].

### <a name="azure-audit-reports"></a>Informes de auditoría de Azure

También se puede revisar si los [informes de auditoría](https://servicetrust.microsoft.com) proporcionan información sobre el cumplimiento de los estándares de protección de datos y los requisitos normativos.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el cifrado de datos, consulte [Cifrado de datos de un solo servidor de Azure Database for PostgreSQL con la clave administrada por el cliente](concepts-data-encryption-postgresql.md).