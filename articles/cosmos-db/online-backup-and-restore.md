---
title: Copias de seguridad en línea y restauración de datos a petición en Azure Cosmos DB
description: En este artículo se describe cómo funciona la copia de seguridad automática y la restauración de datos a petición y cómo se configura el intervalo de copia de seguridad y la retención en Azure Cosmos DB.
author: kanshiG
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: 54bbd5d45e14c1d345570eea9dc5469f77694154
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88853932"
---
# <a name="online-backup-and-on-demand-data-restore-in-azure-cosmos-db"></a>Copias de seguridad en línea y restauración de datos a petición en Azure Cosmos DB

Azure Cosmos DB crea automáticamente copias de seguridad de los datos a intervalos regulares. Las copias de seguridad automáticas se crean sin afectar al rendimiento o a la disponibilidad de las operaciones de las bases de datos. Todas las copias de seguridad se almacenan por separado en un servicio de almacenamiento y se replican globalmente para lograr resistencia frente a desastres regionales. Las copias de seguridad automáticas son útiles en escenarios en los se elimina accidentalmente o se actualiza su cuenta, base de datos o contenedor de Azure Cosmos y más tarde se requiere la recuperación de datos.

## <a name="automatic-and-online-backups"></a>Copias de seguridad automáticas y en línea

Con Azure Cosmos DB no solo los datos, sino también las copias de seguridad de los mismos, son altamente redundantes y resistentes ante los desastres regionales. Los pasos siguientes muestran cómo Azure Cosmos DB realiza la copia de seguridad de datos:

* Azure Cosmos DB toma automáticamente una copia de seguridad de la base de datos cada 4 horas y en cualquier momento, solo se almacenan las últimas 2 de manera predeterminada. Si los intervalos predeterminados no son suficientes para sus cargas de trabajo, puede cambiar el intervalo de copias de seguridad y el período de retención en Azure Portal. Puede cambiar la configuración de copia de seguridad durante la creación de la cuenta de Azure Cosmos o con posterioridad. Si se elimina el contenedor o la base de datos, Azure Cosmos DB conserva las instantáneas existentes de un contenedor o base de datos dado durante 30 días.

* Azure Cosmos DB almacena estas copias de seguridad en Azure Blob Storage, mientras que los datos reales residen localmente dentro de Azure Cosmos DB.

* Para garantizar la baja latencia, la instantánea de la copia de seguridad se guarda en Azure Blob Storage en la misma región que la región de escritura actual (o **una** de las regiones de escritura, si tiene una configuración de arquitectura multimaestro). Para lograr resistencia frente a desastres regionales, cada instantánea de la copia de seguridad de los datos en Azure Blob Storage se vuelve a replicar en otra región mediante almacenamiento con redundancia geográfica (GRS). La región a la que se replica la copia de seguridad se basa en la región de origen y el par regional asociado con la región de origen. Para más información, consulte el artículo [lista de parejas regionales de Azure para redundancia geográfica](../best-practices-availability-paired-regions.md). No se puede acceder directamente a esta copia de seguridad. El equipo de Azure Cosmos DB restaurará la copia de seguridad cuando lo pida mediante una solicitud de soporte técnico.

   La imagen siguiente muestra cómo se crea una copia de seguridad de un contenedor de Azure Cosmos con las tres particiones físicas principales en el Oeste de EE. UU. en una cuenta de Azure Blob Storage remota en el Oeste de EE. UU. y, luego, se replica al Este de EE. UU.:

  :::image type="content" source="./media/online-backup-and-restore/automatic-backup.png" alt-text="Copias de seguridad completas y periódicas de todas las entidades de Cosmos DB en Azure Storage de GRS" border="false":::

* Las copias de seguridad se crean sin afectar el rendimiento ni la disponibilidad de la aplicación. Azure Cosmos DB realiza la copia de seguridad de datos en segundo plano y no consume rendimiento aprovisionado (RU) aprovisionadas, tampoco afecta al rendimiento ni a la disponibilidad de su base de datos.

## <a name="options-to-manage-your-own-backups"></a>Opciones para gestionar sus propias copias de seguridad

Con las cuentas de la API de SQL de Azure Cosmos DB, también puede mantener sus propias copias de seguridad mediante uno de los métodos siguientes:

* Con [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) para mover datos periódicamente a un almacenamiento de su elección.

* Con la [fuente de cambios](change-feed.md) de Azure Cosmos DB para leer datos periódicamente para copias de seguridad completas o para cambios incrementales, y almacenarlo todo en su propio almacenamiento.

## <a name="modify-the-backup-interval-and-retention-period"></a>Modificación del intervalo de copias de seguridad y el período de retención

Azure Cosmos DB toma automáticamente una copia de seguridad de los datos cada 4 horas y en cualquier momento, se almacenan las últimas 2. Esta configuración es la opción predeterminada y se ofrece sin ningún costo adicional. Puede cambiar el intervalo de copias de seguridad y el período de retención predeterminados durante la creación de la cuenta de Azure Cosmos o con posterioridad. La configuración de copia de seguridad se establece en el nivel de cuenta de Azure Cosmos y debe ajustarla en cada cuenta. Después de configurar las opciones de copia de seguridad de una cuenta, se aplica a todos los contenedores dentro de esa cuenta. Actualmente, solo puede cambiar las opciones de copia de seguridad en Azure Portal.

Si eliminó o dañó accidentalmente los datos, **antes de crear una solicitud de soporte técnico para restaurar los datos, asegúrese de aumentar la retención de copias de seguridad de la cuenta al menos a siete días. Es mejor aumentar la retención en un plazo de 8 horas después de este evento.** De esta manera, el equipo de Azure Cosmos DB tiene tiempo suficiente para restaurar la cuenta.

Siga estos pasos para cambiar las opciones de copia de seguridad predeterminadas para una cuenta de Azure Cosmos existente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Vaya a la cuenta de Azure Cosmos y abra el panel **Copia de seguridad y restauración**. Actualice el intervalo de copia de seguridad y el período de retención de copia de seguridad según sea necesario.

   * **Intervalo de copia de seguridad**: es el intervalo en el que Azure Cosmos DB intenta realizar una copia de seguridad de los datos. La copia de seguridad tarda una cantidad de tiempo distinta de cero y, en algunos casos, podría producir un error debido a las dependencias de nivel inferior. Azure Cosmos DB intenta realizar una copia de seguridad en el intervalo configurado; sin embargo, no garantiza que la copia de seguridad se complete dentro de ese intervalo de tiempo. Este valor se puede configurar en horas o minutos. El intervalo de copia de seguridad no puede ser menor que 1 hora ni mayor que 24 horas. Al cambiar este intervalo, el nuevo intervalo entra en vigor a partir de la hora en que se realizó la última copia de seguridad.

   * **Retención de copia de seguridad**: representa el período en el que se conserva cada copia de seguridad. Puede configurarlo en horas o días. El período de retención mínimo no puede ser inferior a dos veces el intervalo de copia de seguridad (en horas) y no puede ser superior a 720 horas.

   * **Copias de los datos conservadas**: de manera predeterminada, se ofrecen dos copias de seguridad de los datos de forma gratuita. Si necesita copias adicionales, debe crear una solicitud de soporte técnico a través de Azure Portal y se cobrarán las copias adicionales. Consulte la sección Almacenamiento consumido en la [página de precios ](https://azure.microsoft.com/pricing/details/cosmos-db/) para conocer el precio exacto de las copias adicionales.

   :::image type="content" source="./media/online-backup-and-restore/configure-backup-interval-retention.png" alt-text="Configuración del intervalo de copia de seguridad y la retención de una cuenta de Azure Cosmos existente" border="true":::

Si configura las opciones de copia de seguridad durante la creación de la cuenta, puede configurar la **directiva de copia de seguridad**, que puede ser **periódica** o **continua**. La directiva periódica le permite configurar el intervalo y la retención de las copias de seguridad. La directiva continua está actualmente disponible solo bajo registro. El equipo de Azure Cosmos DB evaluará la carga de trabajo y aprobará la solicitud.

:::image type="content" source="./media/online-backup-and-restore/configure-periodic-continuous-backup-policy.png" alt-text="Configuración de una directiva de copia de seguridad periódica o continua para nuevas cuentas de Azure Cosmos" border="true":::

## <a name="restore-data-from-an-online-backup"></a>Restauración de datos desde una copia de seguridad en línea

Puede eliminar o modificar accidentalmente los datos en uno de los siguientes escenarios:  

* Se elimina toda la cuenta de Azure Cosmos.

* Se elimina una o varias bases de datos de Azure Cosmos.

* Se elimina uno o varios contenedores de Azure Cosmos.

* Se eliminan o modifican elementos de Azure Cosmos (por ejemplo, documentos) dentro de un contenedor. Este caso concreto normalmente se conoce como "daño de datos".

* Se elimina o se daña una base de datos de oferta compartida o los contenedores dentro de una base de datos de oferta compartida.

Azure Cosmos DB puede restaurar los datos en todos los escenarios anteriores. Al restaurar, se crea una nueva cuenta de Azure Cosmos para almacenar los datos restaurados. El nombre de la nueva cuenta, si no se especifica, tendrá el formato `<Azure_Cosmos_account_original_name>-restored1`. Si se realizan varios intentos de restauración, el último dígito se va incrementando. No se pueden restaurar los datos en una cuenta de Azure Cosmos creada previamente.

Cuando elimina accidentalmente una cuenta de Azure Cosmos, podemos restaurar los datos en una cuenta con el mismo nombre, siempre que el nombre de cuenta no esté en uso. Por lo tanto, se recomienda no volver a crear la cuenta después de eliminarla, porque ello no solo impide que los datos restaurados usen el mismo nombre, sino que además dificulta la detección de la cuenta correcta que se quiere restaurar.

Cuando elimina una base de datos de Azure Cosmos, podemos restaurar la base de datos completa o un subconjunto de los contenedores dentro de esta. También es posible seleccionar contenedores específicos en las bases de datos y restaurarlos en una nueva cuenta de Azure Cosmos.

Cuando elimina o modifica accidentalmente uno o varios elementos dentro de un contenedor (el caso del "daño de datos"), tendrá que especificar el momento al que se restaurará. La hora es importante si hay daños en los datos. Puesto que el contenedor está activo, las copias de seguridad se siguen ejecutando, por lo que si espera más allá del período de retención (el valor predeterminado es de ocho horas) se sobrescribirán las copias de seguridad. **Con el fin de evitar que se sobrescriba la copia de seguridad, aumente la retención de copia de seguridad de su cuenta en al menos 7 días. Es mejor aumentar la retención en un plazo de 8 horas a partir del daño en los datos.**

Si accidentalmente ha eliminado o dañado los datos, debe ponerse en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) en un plazo de ocho horas para que el equipo de Azure Cosmos DB pueda ayudarle a restaurar los datos a partir de las copias de seguridad. De este modo, el equipo de soporte técnico de Azure Cosmos DB tendrá suficiente tiempo para restaurar la cuenta.

Si aprovisiona el rendimiento en el nivel de base de datos, el proceso de copia de seguridad y restauración en este caso se produce a nivel de base de datos completa y no a nivel de contenedores individuales. En tales casos, no puede seleccionar un subconjunto de contenedores para restaurar.

## <a name="migrate-data-to-the-original-account"></a>Migración de datos a la cuenta original

El objetivo principal de la restauración de datos es recuperar los datos que ha eliminado o modificado accidentalmente. Por lo tanto, se recomienda que primero inspeccione el contenido de los datos recuperados para garantizar que contiene lo que esperaba. Más adelante, podrá volver a migrar los datos a la cuenta principal. Aunque se puede usar la cuenta restaurada como nueva cuenta activa, no es una opción recomendada si tiene cargas de trabajo de producción.  

Las siguientes son las diferentes formas para migrar datos a la cuenta de Azure Cosmos original:

* Use la [herramienta de migración de datos de Azure Cosmos DB](import-data.md).
* Use [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).
* Use la [fuente de cambios](change-feed.md) en Azure Cosmos DB.
* Puede escribir su propio código personalizado.

Asegúrese de eliminar las cuentas restauradas en cuanto haya migrado los datos, ya que generarán cargos.

## <a name="next-steps"></a>Pasos siguientes

A continuación puede aprender cómo restaurar los datos de una cuenta de Azure Cosmos o cómo migrar datos a una cuenta de Azure Cosmos

* Para hacer una solicitud de restauración, póngase en contacto con el servicio de soporte técnico de Azure y [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
* [Restauración de los datos desde una cuenta de Azure Cosmos](how-to-backup-and-restore.md)
* [Uso de la fuente de cambios en Cosmos DB](change-feed.md) para mover datos a Azure Cosmos DB.
* [Uso de Azure Data Factory](../data-factory/connector-azure-cosmos-db.md) para mover datos a Azure Cosmos DB.

