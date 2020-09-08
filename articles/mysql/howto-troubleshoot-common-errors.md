---
title: 'Solución de errores comunes: Azure Database for MySQL'
description: Aprenda a solucionar errores comunes de migración que encuentran los usuarios nuevos en el servicio Azure Database for MySQL
author: savjani
ms.service: mysql
ms.author: pariks
ms.custom: mvc
ms.topic: overview
ms.date: 8/20/2020
ms.openlocfilehash: ebe9f936e3d0dfafec23842fcdbfd225995d546b
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719788"
---
# <a name="common-errors"></a>Errores comunes

Azure Database for MySQL es un servicio totalmente administrado que usa la tecnología de la versión de comunidad de MySQL. La experiencia de MySQL en un entorno de servicio administrado puede diferir de ejecutarlo en su propio entorno. En este artículo, verá algunos de los errores habituales que pueden encontrar los usuarios la primera vez que migran al servicio Azure Database for MySQL, o que desarrollan en dicho servicio.

## <a name="errors-due-to-lack-of-super-privilege-and-dba-role"></a>Errores debidos a la falta del privilegio SUPER y del rol DBA

El privilegio SUPER y el rol DBA no se admiten en el servicio. En consecuencia, pueden producirse algunos errores comunes que se enumeran a continuación:

#### <a name="error-1419-you-do-not-have-the-super-privilege-and-binary-logging-is-enabled-you-might-want-to-use-the-less-safe-log_bin_trust_function_creators-variable"></a>ERROR 1419: You do not have the SUPER privilege and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable) (No tiene el privilegio SUPER y el registro binario está habilitado [puede] usar la variable log_bin_trust_function_creators, pero es menos segura)

El error anterior puede aparecer al crear una función, al desencadenar como se muestra a continuación o al importar un esquema. Las instrucciones del DDL como CREATE FUNCTION o CREATE TRIGGER se escriben en el registro binario, por lo que la réplica secundaria puede ejecutarlas. El subproceso de réplica de SQL tiene privilegios totales, lo que se puede aprovechar para elevar los privilegios. Para protegerse frente a este riesgo en los servidores que tienen habilitado el registro binario, el motor de MySQL requiere que los creadores de funciones almacenadas tengan el privilegio SUPER, además del privilegio CREATE ROUTINE habitual que se requiere. 

```sql
CREATE FUNCTION f1(i INT)
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
  RETURN i;
END;
```

**Solución:**  para resolver el error, establezca log_bin_trust_function_creators en 1 en la hoja [Parámetros del servidor](howto-server-parameters.md) del portal, ejecute las instrucciones de DDL o importe el esquema para crear los objetos deseados y revertir el parámetro log_bin_trust_function_creators a su valor anterior después de la creación.

#### <a name="error-1227-42000-at-line-101-access-denied-you-need-at-least-one-of-the-super-privileges-for-this-operation-operation-failed-with-exitcode-1"></a>ERROR 1227 (42000) en la línea 101: Access denied; you need (at least one of) the SUPER privilege(s) for this operation [Acceso denegado; necesita (al menos uno de) los privilegios SUPER para esta operación]. Error en la operación, exitcode 1

El error anterior puede producirse tanto al importar un archivo de volcado como al crear un procedimiento que contenga [depuradores](https://dev.mysql.com/doc/refman/5.7/en/create-procedure.html). 

**Solución:**  para resolver este error, el usuario administrador puede conceder privilegios para crear o ejecutar procedimientos mediante la ejecución del comando GRANT, como en los ejemplos siguientes:

```sql
GRANT CREATE ROUTINE ON mydb.* TO 'someuser'@'somehost';
GRANT EXECUTE ON PROCEDURE mydb.myproc TO 'someuser'@'somehost';
```
Como alternativa, puede reemplazar los depuradores por el nombre del usuario administrador que ejecuta el proceso de importación, como se muestra a continuación.

```sql
DELIMITER;;
/*!50003 CREATE*/ /*!50017 DEFINER=`root`@`127.0.0.1`*/ /*!50003
DELIMITER;;

/* Modified to */

DELIMITER ;;
/*!50003 CREATE*/ /*!50017 DEFINER=`AdminUserName`@`ServerName`*/ /*!50003
DELIMITER ;
```

## <a name="next-steps"></a>Pasos siguientes
Si no ha encontrado la respuesta que buscaba, considere la posibilidad de realizar estas otras acciones:
- Publique su pregunta en la [página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-database-mysql.html) o en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-database-mysql).
- Envíe un correo electrónico al equipo de Azure Database for MySQL [@Ask Azure DB for MySQL](mailto:AskAzureDBforMySQL@service.microsoft.com). Esta dirección de correo electrónico no es un alias de soporte técnico.
- Póngase en contacto con el soporte técnico de Azure, [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.
- Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql).
