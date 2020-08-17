---
title: 'Almacenamiento inmutable de blob: Azure Storage'
description: Azure Storage ofrece compatibilidad WORM (escribir una vez, leer muchas) con el almacenamiento de Blob (objeto), lo que permite a los usuarios almacenar datos en un estado no modificable durante un intervalo de tiempo especificado.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 11/18/2019
ms.author: tamram
ms.reviewer: hux
ms.subservice: blobs
ms.openlocfilehash: 54014a0d76130b82788a1ae432e42baec28df2c2
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448337"
---
# <a name="store-business-critical-blob-data-with-immutable-storage"></a>Almacenamiento de datos de blobs críticos para la empresa con almacenamiento inmutable

El almacenamiento inmutable para Azure Blob Storage permite a los usuarios almacenar objetos de datos críticos para la empresa en un estado WORM (escribir una vez, leer muchas). En este estado, los usuarios no pueden borrar ni modificar los datos durante el intervalo de tiempo especificado por el usuario. Mientras dure el intervalo de retención, se pueden crear y leer blobs, pero no modificar ni eliminar. El almacenamiento inmutable está disponible en las cuentas de uso general v1, de uso general v2, BlobStorage y BlockBlobStorage de todas las regiones de Azure.

Para obtener información sobre cómo establecer y borrar las suspensiones legales o crear una directiva de retención basada en el tiempo mediante Azure Portal, PowerShell o la CLI de Azure, vea [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](storage-blob-immutability-policies-manage.md).

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="about-immutable-blob-storage"></a>Acerca del almacenamiento de blobs inmutable

El almacenamiento inmutable ayuda a los centros de atención médica, las instituciones financieras y los sectores relacionados &mdash;sobre todo las organizaciones de agente-distribuidor&mdash; a almacenar los datos de forma segura. También puede usarse en cualquier escenario para proteger datos críticos e impedir que se eliminen o modifiquen.

Las aplicaciones típicas son:

- **Cumplimiento normativo**: el almacenamiento inmutable para Azure Blob Storage ayuda a las organizaciones a cumplir SEC 17a-4(f), CFTC 1.31(d), FINRA y otras regulaciones. En una nota técnica de producto de Cohasset Associates se detalla cómo el almacenamiento inmutable aborda estos requisitos reguladores. Este documento se puede descargar en el [portal de confianza de servicios de Microsoft](https://aka.ms/AzureWormStorage). El [Centro de confianza de Azure](https://www.microsoft.com/trustcenter/compliance/compliance-overview) contiene información detallada sobre nuestras certificaciones de cumplimiento.

- **Retención segura de documentos**: el almacenamiento inmutable de Azure Blob Storage garantiza que ningún usuario puede modificar ni eliminar datos, incluidos los usuarios que tienen privilegios administrativos en la cuenta.

- **Suspensión legal**: el almacenamiento inmutable de Azure Blob Storage permite a los usuarios almacenar información confidencial crítica para procedimientos judiciales o uso empresarial en un estado a prueba de manipulación durante el tiempo deseado hasta que se elimina la suspensión. Esta característica no se limita solo a los casos de uso legales, sino que también puede considerarse una suspensión basada en eventos o un bloqueo de la empresa, donde es necesario proteger los datos basados en desencadenadores de eventos o directivas corporativas.

El almacenamiento inmutable admite las características siguientes:

- **[Compatibilidad con directivas de retención con duración definida](#time-based-retention-policies)** : los usuarios pueden establecer directivas para almacenar datos durante un intervalo especificado. Cuando se establece una directiva de retención con duración definida, se pueden crear y leer blobs, pero no se pueden modificar ni eliminar. Una vez transcurrido el período de retención, los blobs se pueden eliminar pero no sobrescribir.

- **[Compatibilidad con directivas de suspensión legal](#legal-holds)** : si el intervalo de retención no se conoce, los usuarios pueden establecer suspensiones legales para almacenar los datos inmutables hasta que estas desaparezcan.  Cuando se establece una directiva de suspensión legal, se pueden crear y leer blobs, pero no se pueden modificar ni eliminar. Cada suspensión legal está asociada a una etiqueta alfanumérica definida por el usuario que se usa como una cadena de identificación (por ejemplo, un identificador de caso, un nombre de evento, etc.). 

- **Compatibilidad con todos los niveles de blobs:** las directivas WORM son independientes del nivel de Azure Blob Storage y se aplican a todos los niveles: de archivo, frecuente y esporádico. Los usuarios pueden transferir sus datos al nivel que les ofrezca la mayor optimización de costos de acuerdo con sus cargas de trabajo sin alterar la inmutabilidad de los datos.

- **Configuración en el nivel de contenedor**: los usuarios pueden configurar las directivas de retención con duración definida y las etiquetas de suspensión legal a nivel del contenedor. Mediante valores de configuración sencillos en el nivel de contenedor, los usuarios pueden crear y bloquear las directivas de retención con duración definida, ampliar los intervalos de retención, establecer y eliminar suspensiones legales, etc. Estas directivas se aplican a todos los blobs del contenedor, tanto a los nuevos como a los existentes.

- **Compatibilidad con el registro de auditoría**: todos los contenedores incluyen un registro de auditoría de directiva. En él se muestran hasta siete comandos de retención con duración definida para las directivas de retención con duración definida bloqueadas y contiene el identificador de usuario, el tipo de comando, las marcas de tiempo y el intervalo de retención. En el caso de las suspensiones legales, el registro contiene el identificador del usuario, el tipo de comando, las marcas de tiempo y las etiquetas de suspensión legal. Este registro se conserva mientras dure la directiva, de acuerdo con las directrices de regulación SEC 17a-4(f). El [registro de actividad de Azure](../../azure-monitor/platform/platform-logs-overview.md) muestra un registro más completo de todas las actividades del plano de control, mientras que al habilitar los [registros de recursos de Azure](../../azure-monitor/platform/platform-logs-overview.md) se conservan y se muestran las operaciones del plano de datos. Es responsabilidad del usuario almacenar dichos registros de forma persistente, ya que podría ser obligatorio por ley o por otros fines.

## <a name="how-it-works"></a>Funcionamiento

Almacenamiento inmutable para Azure Blob Storage admite dos tipos de directivas inmutables o WORM: retención con duración definida y suspensiones legales. Cuando se aplica una directiva de retención con duración definida o una suspensión legal a un contenedor, todos los blobs existentes pasan al estado WORM inmutable en menos de 30 segundos. Todos los nuevos blobs que se carguen en ese contenedor protegido mediante directiva también pasarán a un estado inmutable. Una vez que todos los blobs estén en un estado inmutable, la directiva inmutable se confirma y no se permite ninguna operación de sobrescritura o eliminación en el contenedor inmutable.

Tampoco se permite la eliminación de la cuenta de almacenamiento y el contenedor si hay blobs en un contenedor protegidos mediante una directiva de suspensión legal o de duración definida bloqueada. Una directiva de suspensión legal protegerá contra la eliminación del blob, el contenedor y la cuenta de almacenamiento. Las directivas de duración definida bloqueadas y desbloqueadas protegerán contra la eliminación de blobs durante el tiempo especificado. Las directivas de duración definida bloqueadas y desbloqueadas protegerán contra la eliminación del contenedor solo si existe al menos un blob en el contenedor. Solo un contenedor con una directiva de duración definida *bloqueada* protegerá contra eliminaciones de la cuenta de almacenamiento; los contenedores con directivas de duración definida desbloqueadas no ofrecen protección contra la eliminación de la cuenta de almacenamiento ni cumplimiento.

Para obtener más información sobre cómo establecer y bloquear las directivas de retención con duración definida, vea [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](storage-blob-immutability-policies-manage.md).

## <a name="time-based-retention-policies"></a>Directivas de retención con duración definida

> [!IMPORTANT]
> Una directiva de retención con duración definida debe estar *bloqueada* para que el blob esté en estado inmutable (protegido frente a escritura y eliminación) y, por consiguiente, se cumplan SEC 17a-4(f) y otras regulaciones. Se recomienda bloquear la directiva en un período razonable, normalmente antes de 24 horas. El estado inicial de una directiva de retención con duración definida es *desbloqueada*, lo que permite probar la característica y realizar cambios en la directiva antes de bloquearla. Aunque el estado *desbloqueada* proporciona protección de inmutabilidad, no se recomienda su uso para otros fines que no sean las evaluaciones de la característica a corto plazo. 

Cuando se aplica una directiva de retención con duración definida a un contenedor, todos los blobs de este permanecen en estado inmutable durante el período de retención *efectivo*. El período de retención efectivo para los blobs es igual a la diferencia entre la **hora de creación** del blob y el intervalo de retención especificado por el usuario. Como los usuarios pueden ampliar el intervalo de retención, el almacenamiento inmutable utiliza el valor más reciente del intervalo de retención especificado por el usuario para calcular el período de retención efectivo.

Por ejemplo, supongamos que un usuario crea una directiva de retención de duración definida con un intervalo de retención de cinco años. Un blob existente en ese contenedor, _testblob1_, se ha creado hace un año; por tanto, el período de retención efectivo para _testblob1_ es de cuatro años. Cuando se carga un nuevo blob, _testblob2_, en el contenedor, el período de retención efectivo de _testblob2_ es de cinco años a partir de su creación.

Se recomienda una directiva de retención con duración definida desbloqueada solo para probar la característica; no obstante, una directiva debe estar bloqueada para el cumplimiento de SEC 17a-4(f) y otras normas. Una vez que se ha bloqueado una directiva de retención con duración definida, no se puede quitar la directiva y se permite un máximo de cinco aumentos al período de retención efectivo.

Los límites siguientes se aplican a las directivas de retención:

- En el caso de una cuenta de almacenamiento, el número máximo de contenedores con directivas inmutables de duración definida bloqueadas es 10 000.
- El intervalo de retención mínimo es un día. El máximo es 146 000 días (400 años).
- Para un contenedor, el número máximo de modificaciones para ampliar un intervalo de retención para las directivas inmutables con duración definida bloqueadas es 5.
- En el caso de un contenedor, se retiene un máximo de siete registros de auditoría de la directiva de retención con duración definida para una directiva bloqueada.

### <a name="allow-protected-append-blobs-writes"></a>Permitir escrituras de blobs en anexos protegidos

Los blobs en anexos se componen de bloques de datos y se optimizan para las operaciones de anexión de datos necesarias en escenarios de auditoría y registro. Por diseño, los blobs en anexos solo permiten agregar nuevos bloques al final del blob. Con independencia de la inmutabilidad, la modificación o eliminación de los bloques existentes en un blob en anexos básicamente no se permite. Para más información sobre los blobs en anexos, vea [Acerca de los blobs en anexos](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-append-blobs).

Solo las directivas de retención de duración definida tienen una configuración `allowProtectedAppendWrites` que permite escribir nuevos bloques en un blob en anexos al tiempo que se mantiene la protección y el cumplimiento de inmutabilidad. Si se habilita este valor, puede crear un blob en anexos directamente en el contenedor protegido mediante la directiva y continuar agregando nuevos bloques de datos al final de los blobs en anexos existentes mediante la API *AppendBlock*. Solo se pueden agregar nuevos bloques y los bloques existentes no se pueden modificar ni eliminar. Se sigue aplicando la protección de inmutabilidad de retención de tiempo, lo que impide la eliminación del blob en anexos hasta que haya transcurrido el período de retención vigente. La habilitación de esta configuración no afecta al comportamiento de inmutabilidad de los blobs en bloques ni los blobs en páginas.

Como esta configuración forma parte de una directiva de retención de duración definida, los blobs en anexos todavía pueden permanecer en el estado inmutable durante el período de retención *efectivo*. Como se pueden anexar nuevos datos más allá de la creación inicial del blob en anexos, hay una ligera diferencia en el modo de determinar el período de retención. El período de retención efectivo es la diferencia entre la **hora de última modificación** del blob en anexos y el intervalo de retención especificado por el usuario. Como sucede al ampliar el intervalo de retención, el almacenamiento inmutable usa el valor más reciente del intervalo de retención especificado por el usuario para calcular el período de retención efectivo.

Por ejemplo, imagine que un usuario crea una directiva de retención de duración definida con `allowProtectedAppendWrites` habilitado y un intervalo de retención de 90 días. En el día de hoy, se crea un blob en anexos, _logblob1_, en el contenedor y se siguen agregando nuevos registros al blob en anexos durante los próximos 10 días; por tanto, el período de retención efectivo de _logblob1_ es de 100 días a partir de hoy (la hora de la última anexión + 90 días).

Las directivas de retención de duración definida desbloqueadas permiten habilitar y deshabilitar la configuración `allowProtectedAppendWrites` en cualquier momento. Una vez que se ha bloqueado la directiva de retención de duración definida, no se puede cambiar el valor de `allowProtectedAppendWrites`.

Las directivas de retención legal no pueden habilitar `allowProtectedAppendWrites` y cualquier retención legal anulará la propiedad 'allowProtectedAppendWrites'. Si se aplica la suspensión legal a una directiva de retención de duración definida con `allowProtectedAppendWrites` habilitada, se producirá un error en la API *AppendBlock* hasta que se levante la suspensión legal.

## <a name="legal-holds"></a>Retenciones legales

Las suspensiones legales son suspensiones temporales que se pueden usar con fines de investigación legal o directivas de protección general. Cada directiva de suspensión legal debe estar asociada a una o varias etiquetas. Las etiquetas se usan como un identificador con nombre, como un identificador de caso o un evento, para categorizar y describir el propósito de la suspensión.

Un contenedor puede tener una suspensión legal y una directiva de retención con duración definida al mismo tiempo. Todos los blobs de ese contenedor permanecen en estado inmutable hasta que se eliminen todas las suspensiones legales, aunque haya expirado el período de retención efectivo. Por el contrario, un blob permanece en estado inmutable hasta que expire el período de retención efectivo, aunque se hayan eliminado todas las suspensiones legales.

Los límites siguientes se aplican a las suspensiones legales:

- En el caso de una cuenta de almacenamiento, el número máximo de contenedores con un valor de suspensión legal es 10 000.
- En un contenedor, el número máximo de etiquetas de suspensión legal es 10.
- La longitud mínima de una etiqueta de suspensión legal es de tres caracteres alfanuméricos. La longitud máxima es de 23 caracteres alfanuméricos.
- Para un contenedor, se conservan hasta 10 registros de auditoría de directiva de suspensión legal lo que dura la directiva.

## <a name="scenarios"></a>Escenarios
La tabla siguiente muestra los tipos de operaciones de almacenamiento de blobs que se deshabilitan para los diferentes escenarios de inmutabilidad. Para obtener más información, vea la documentación de la [API REST de servicios Azure Blob](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api).

|Escenario  |Estado del blob  |Operaciones de blob denegadas  |Protección de contenedores y cuentas
|---------|---------|---------|---------|
|El intervalo de retención efectivo del blob no ha expirado todavía o se ha establecido una retención legal     |Inmutable: protegido frente a eliminación y escritura         | Put Blob<sup>1</sup>, Put Block<sup>1</sup>, Put Block List<sup>1</sup>, Delete Container, Delete Blob, Set Blob Metadata, Put Page, Set Blob Properties,  Snapshot Blob, Incremental Copy Blob, Append Block<sup>2</sup>         |Eliminación de contenedor denegada; Eliminación de la cuenta de almacenamiento denegada         |
|El intervalo de retención efectivo del blob ha expirado y no se ha establecido una suspensión legal    |Protegido contra escritura únicamente (están permitidas las operaciones de eliminación)         |Put Blob<sup>1</sup>, Put Block<sup>1</sup>, Put Block List<sup>1</sup>, Set Blob Metadata, Put Page, Set Blob Properties,  Snapshot Blob, Incremental Copy Blob, Append Block<sup>2</sup>         |Eliminación de contenedor denegada si existe al menos un blob en el contenedor protegido; Eliminación de la cuenta de almacenamiento denegada solo para las directivas de duración definida *bloqueadas*         |
|No se aplica ninguna directiva WORM (sin retención de duración definida ni etiqueta de suspensión legal)     |Mutable         |None         |None         |

<sup>1</sup> El servicio de blob permite que estas operaciones creen un blob una vez. Todas las sucesivas operaciones de sobrescritura en una ruta de acceso de blob existente en un contenedor inmutable no se permiten.

<sup>2</sup> Solo se permite Anexar bloque para las directivas de retención de duración definida con la propiedad `allowProtectedAppendWrites` habilitada. Para más información, vea la sección [Permitir la escritura de blobs en anexos protegidos](#allow-protected-append-blobs-writes).

## <a name="pricing"></a>Precios

El uso de esta característica no tiene costo adicional. El precio de los datos inmutables es el mismo que el de los datos mutables. Para obtener información detallada sobre los precios de almacenamiento de blobs de Azure, vea la [página de precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="faq"></a>Preguntas más frecuentes

**¿Se puede proporcionar documentación de cumplimiento WORM?**

Sí. Para documentar el cumplimiento, Microsoft ha usado una de las principales empresas de valoración independiente especializada en la administración de registros y la gobernanza de la información, Cohasset Associates. Dicha empresa se ha encargado de evaluar el almacenamiento de blobs inmutable y el cumplimiento con los requisitos específicos del sector de servicios financieros. Cohasset ha validado que el almacenamiento de blobs inmutable, cuando se ha usado para conservar los blobs con duración definida en un estado WORM, cumplía los requisitos de almacenamiento pertinentes de las reglas CFTC 1.31(c)-(d), FINRA 4511 y SEC 17a-4. Microsoft se centró en este conjunto de reglas, ya que representan la guía más prescriptiva globalmente para la retención de registros en las instituciones financieras. El informe de Cohasset está disponible en el [Centro de confianza de servicios de Microsoft](https://aka.ms/AzureWormStorage). Para solicitar una carta de certificación de Microsoft con respecto al cumplimiento de inmutabilidad de WORM, póngase en contacto con el Soporte técnico de Azure.

**¿La característica se aplica solo a los blobs en bloques y en anexos, o también a los blobs en páginas?**

El almacenamiento inmutable se puede usar con cualquier tipo de blob cuando se establece en el nivel de contenedor, pero se recomienda usar WORM para los contenedores que almacenan principalmente blobs en bloques y blobs en anexos. Los blobs existentes de un contenedor se protegerán mediante una directiva WORM recién establecida. Pero los blobs en páginas se tendrán que crear fuera del contenedor WORM y después copiarse en él. Una vez copiado en un contenedor WORM, no se permiten más cambios en un blob en páginas. No se recomienda establecer una directiva de WORM en un contenedor que almacene discos duros virtuales (blobs en páginas) para cualquier máquina virtual activa, ya que bloqueará el disco de la máquina virtual. Se recomienda revisar exhaustivamente la documentación y probar los escenarios antes de bloquear las directivas de duración definida.

**¿Es necesario crear una cuenta de almacenamiento para usar esta característica?**

No, puede usar el almacenamiento inmutable con cualquier cuenta de uso general v1, uso general v2, BlobStorage o BlockBlobStorage existente o recién creada. Se admiten las cuentas de almacenamiento de uso general v1, pero se recomienda actualizar a las de uso general v2 para poder aprovechar otras características. Para obtener información sobre la actualización de una cuenta de almacenamiento de uso general v1 existente, vea [Actualización de una cuenta de almacenamiento](../common/storage-account-upgrade.md).

**¿Se puede aplicar una suspensión legal y una directiva de retención con duración definida a la vez?**

Sí, un contenedor puede tener una retención legal y una directiva de retención basada en tiempo al mismo tiempo; sin embargo, el valor 'allowProtectedAppendWrites' no se aplicará hasta que se elimine la retención legal. Todos los blobs de ese contenedor permanecen en estado inmutable hasta que se eliminen todas las suspensiones legales, aunque haya expirado el período de retención efectivo. Por el contrario, un blob permanece en estado inmutable hasta que expire el período de retención efectivo, aunque se hayan eliminado todas las suspensiones legales. 

**¿Las directivas de suspensión legal son solo para procedimientos judiciales o pueden emplearse en otros casos?**

No, la suspensión legal es simplemente el término general que se usa con una directiva de retención sin duración definida. Su uso no se aplica solamente a procedimientos judiciales. También es útil para deshabilitar la sobrescritura y las eliminaciones con el fin de proteger los datos WORM importantes de la empresa, cuando el período de retención se desconoce. Se puede usar como una directiva de empresa para proteger las cargas de trabajo WORM críticas o como una directiva de almacenamiento provisional antes de que un desencadenador de eventos personalizado requiera el uso de una directiva de retención con duración definida. 

**¿Se puede quitar una suspensión legal o una directiva de retención con duración definida _bloqueada_?**

Solo las directivas de retención con duración definida desbloqueadas se pueden quitar de un contenedor. Una vez que se ha bloqueado una directiva de retención con duración definida, no se puede quitar; solo se permiten extensiones del período de retención efectivo. Se pueden eliminar las etiquetas de suspensión legal. Cuando se eliminan todas las etiquetas legales, se quita la suspensión legal.

**¿Qué ocurre si intento eliminar un contenedor con una directiva de retención de duración definida o de suspensión legal?**

La operación Eliminar contenedor producirá un error si hay al menos un blob en el contenedor con una directiva de retención de duración definida bloqueada o desbloqueada, o si el contenedor tiene suspensión legal. La operación Eliminar contenedor solo se ejecutará correctamente si no existe ningún blob en el contenedor y no hay suspensiones legales. 

**¿Qué ocurre si intento eliminar una cuenta de almacenamiento con un contenedor que tiene una directiva de retención de duración definida o una suspensión legal?**

Se producirá un error en la eliminación de la cuenta de almacenamiento si hay al menos un contenedor con una suspensión legal establecida o una directiva de duración definida **bloqueada**. Un contenedor con una directiva de duración definida desbloqueada no protege contra la eliminación de la cuenta de almacenamiento. Tendrá que eliminar todas las suspensiones legales y todos los contenedores **bloqueados** antes de poder eliminar la cuenta de almacenamiento. Para obtener información acerca de la eliminación de contenedores, vea la pregunta anterior. También puede aplicar más protecciones de eliminación para la cuenta de almacenamiento con [bloqueos de Azure Resource Manager](../../azure-resource-manager/management/lock-resources.md).

**¿Puedo mover los datos entre distintos niveles de blob (de acceso frecuente, esporádico, de archivo) cuando el blob está en el estado inmutable?**

Sí, puede usar el comando Set Blob Tier para mover datos entre los niveles de blob y mantenerlos al mismo tiempo en estado inmutable compatible. El almacenamiento inmutable solamente se admite en los niveles de blob de acceso frecuente, de acceso esporádico y de archivo.

**¿Qué ocurre si dejo de pagar y no ha expirado el intervalo de retención?**

En caso de impago, las directivas de retención de datos normales se aplicarán como estipulado en los términos y condiciones de su contrato con Microsoft. Para obtener información general, consulte [Administración de datos en Microsoft](https://www.microsoft.com/en-us/trust-center/privacy/data-management). 

**¿Se ofrece una evaluación o un período de gracia para probar la característica?**

Sí. Cuando se crea por primera vez una directiva de retención con duración definida, tiene el estado *desbloqueada*. En este estado, puede realizar los cambios que desee en el intervalo de retención, como por ejemplo, aumentarlo o disminuirlo, e incluso eliminar la directiva. Una vez bloqueada la directiva, permanece en este estado hasta que expira el intervalo de retención. Esta directiva bloqueada evita la eliminación y la modificación del intervalo de retención. Se recomienda encarecidamente que utilice el estado *desbloqueado* únicamente para fines de prueba y que bloquee la directiva en un periodo de 24 horas. Estas prácticas ayudan a cumplir SEC 17a-4(f) y otras normativas.

**¿Se puede usar la eliminación temporal junto con las directivas de blob inmutables?**

Sí, si los requisitos de cumplimiento permiten habilitar la eliminación temporal. La [eliminación temporal para Azure Blob Storage](storage-blob-soft-delete.md) se aplica a todos los contenedores dentro de una cuenta de almacenamiento, con independencia de que exista una directiva de retención con duración definida o una suspensión legal. Se recomienda habilitar la eliminación temporal para mayor protección antes de aplicar y confirmar las directivas WORM inmutables.

## <a name="next-steps"></a>Pasos siguientes

- [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](storage-blob-immutability-policies-manage.md)
- [Establecimiento de reglas para organizar y eliminar automáticamente datos de blob con administración del ciclo de vida](storage-lifecycle-management-concepts.md)
- [Eliminación temporal de blobs de Azure Storage](../blobs/storage-blob-soft-delete.md)
- [Protección de suscripciones, grupos de recursos y recursos con bloqueos de Azure Resource Manager](../../azure-resource-manager/management/lock-resources.md).
