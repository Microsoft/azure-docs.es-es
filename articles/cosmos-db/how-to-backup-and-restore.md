---
title: Restauración de los datos de Azure Cosmos DB desde una copia de seguridad
description: En este artículo se describe cómo restaurar los datos de Azure Cosmos DB desde una copia de seguridad, cómo ponerse en contacto con soporte técnico de Azure para restaurar los datos y los pasos que debe dar después de restaurar los datos.
author: kanshiG
ms.service: cosmos-db
ms.topic: how-to
ms.date: 08/24/2020
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 9956ca0ca9c0957557e7ee74883a75c074ff22f8
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797979"
---
# <a name="restore-data-from-a-backup-in-azure-cosmos-db"></a>Restauración de los datos desde una copia de seguridad en Azure Cosmos DB

En caso de que elimine accidentalmente la base de datos o un contenedor, puede [presentar una incidencia de soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) o [llamar al servicio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) para restaurar los datos a partir de copias de seguridad en línea automáticas. El Soporte técnico de Azure está disponible solo para algunos planes, por ejemplo, **Estándar** o **Desarrollador**, y planes superiores. El soporte técnico de Azure no está disponible con el plan **Básico**. Para más información sobre los diferentes planes de soporte técnico, consulte la página de [planes de soporte técnico de Azure](https://azure.microsoft.com/support/plans/).

Para restaurar una instantánea específica de la copia de seguridad, Azure Cosmos DB requiere que los datos estén disponibles durante el ciclo de copia de seguridad de esa instantánea.

## <a name="request-a-restore"></a>Solicitud de una restauración

Antes de solicitar una restauración, debe tener los siguientes detalles:

* Tenga listo el identificador de su suscripción.

* En función de cómo se hayan eliminado o modificado por accidente los datos, debe prepararse para tener información adicional. Se recomienda tener disponible la información con antelación para minimizar las idas y venidas que pueden ser perjudiciales en algunos casos donde el tiempo es fundamental.

* Si se elimina toda la cuenta de Azure Cosmos DB, deberá proporcionar el nombre de la cuenta eliminada. Si crea otra cuenta con el mismo nombre que la cuenta eliminada, indique esto al equipo de soporte técnico, ya que esto ayuda a establecer la cuenta adecuada que debe elegirse. Se recomienda presentar distintas incidencias de soporte técnico para cada cuenta eliminada, ya que esto minimiza la confusión del estado de la restauración.

* Si se han eliminado una o varias bases de datos, debe proporcionar la cuenta de Azure Cosmos, así como los nombres de las bases de datos de Azure Cosmos y especificar si existe una nueva base de datos con el mismo nombre.

* Si se han eliminado uno o varios contenedores, debe proporcionar el nombre de cuenta de Azure Cosmos, los nombres de las bases de datos y los nombres de los contenedores. Además, especifique si existe un contenedor con el mismo nombre.

* Si accidentalmente ha eliminado o dañado los datos, debe ponerse en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) en un plazo de ocho horas para que el equipo de Azure Cosmos DB pueda ayudarle a restaurar los datos a partir de las copias de seguridad. **Antes de crear una solicitud de soporte técnico para restaurar los datos, asegúrese de [aumentar la retención de copias de seguridad](online-backup-and-restore.md) de la cuenta al menos a siete días. Es mejor aumentar la retención en un plazo de 8 horas después de este evento.** De este modo, el equipo de soporte técnico de Azure Cosmos DB tendrá suficiente tiempo para restaurar la cuenta.

Además del nombre de la cuenta de Azure Cosmos, los nombres de las bases de datos y los nombres de los contenedores, debe especificar el momento en el que se pueden restaurar los datos. Es importante ser lo más preciso posible para ayudarnos a determinar las mejores copias de seguridad disponibles en ese momento. **También es importante especificar la hora en UTC.**

En la captura de pantalla siguiente se muestra cómo crear una solicitud de soporte técnico para un contenedor (colección/grafo/tabla) para restaurar los datos mediante Azure Portal. Proporcione detalles adicionales, como el tipo de datos, el propósito de la restauración, la hora a la que se eliminaron los datos para ayudarnos a dar prioridad a la solicitud.

:::image type="content" source="./media/how-to-backup-and-restore/backup-support-request-portal.png" alt-text="Creación de una solicitud de soporte técnico de copia de seguridad con Azure Portal":::

## <a name="post-restore-actions"></a>Acciones posteriores a la restauración

Después de restaurar los datos, recibirá una notificación sobre el nombre de la nueva cuenta (normalmente se encuentra en el formato `<original-name>-restored1`) y la hora a la que se ha restaurado la cuenta. La cuenta restaurada tendrá el mismo rendimiento aprovisionado, directivas de indexación y estará en la misma región que la cuenta original. Un usuario que sea administrador de suscripciones o administrador conjunto puede ver la cuenta restaurada.

Después de restaurar los datos, debe inspeccionar y validar los datos de la cuenta restaurada y asegurarse de que contiene la versión que esperaba. Si todo es correcto, debe migrar los datos nuevamente a la cuenta original usando la [fuente de cambios de Azure Cosmos DB](change-feed.md) o [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).

Se recomienda eliminar el contenedor o la base de datos inmediatamente después de migrar los datos. Si no elimina las bases de datos o los contenedores restaurados, supondrá un costo para las unidades de solicitud, el almacenamiento y la salida.

## <a name="next-steps"></a>Pasos siguientes

Luego puede obtener información sobre cómo migrar los datos nuevamente a la cuenta original siguiendo los artículos a continuación:

* Para hacer una solicitud de restauración, póngase en contacto con el servicio de soporte técnico de Azure y [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
* [Uso de la fuente de cambios en Cosmos DB](change-feed.md) para mover datos a Azure Cosmos DB.

* [Uso de Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) para mover datos a Azure Cosmos DB.
