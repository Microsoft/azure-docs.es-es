---
title: Eliminación temporal de Azure Key Vault | Microsoft Docs
description: La eliminación temporal de Azure Key Vault permite la recuperación de almacenes de claves y objetos de almacenes de claves eliminados, como claves, secretos y certificados.
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 03/19/2019
ms.openlocfilehash: 1affa396407ba9804261c799b559e40928b9b1fa
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87388332"
---
# <a name="azure-key-vault-soft-delete-overview"></a>Información general sobre la eliminación temporal de Azure Key Vault

> [!IMPORTANT]
> Debe habilitar inmediatamente la eliminación temporal en los almacenes de claves. La posibilidad de rechazar la eliminación temporal quedará en desuso a finales de año y la protección contra la eliminación temporal se activará automáticamente en todos los almacenes de claves.  Consulte los detalles completos [aquí](soft-delete-change.md).

La característica de eliminación temporal de Key Vault permite la recuperación de los almacenes y los objetos de Key Vault eliminados (por ejemplo, claves, secretos, certificados), lo que se conoce como eliminación temporal. En concreto, se tratan los siguientes escenarios:  Las protecciones que se ofrecen son las siguientes:

- Una vez que se elimina un secreto, una clave, un certificado o un almacén de claves, no se podrá recuperar durante un período configurable de 7 a 90 días naturales. Si no se especifica ninguna configuración, el período de recuperación predeterminado se establecerá en 90 días. De esta forma, los usuarios tendrán tiempo suficiente para darse cuenta de la eliminación accidental de un secreto y responder a ella.
- Para eliminar de forma permanente un secreto, se deben realizar dos operaciones. En primer lugar, el usuario debe eliminar el objeto, que lo coloca en estado de eliminación temporal. En segundo lugar, el usuario debe purgar el objeto en estado de eliminación temporal. La operación de purga requiere permisos adicionales de la directiva de acceso. Estas protecciones adicionales reducen el riesgo de que un usuario elimine un secreto o un almacén de claves por accidente o de forma malintencionada.  
- Para purgar un secreto en estado de eliminación temporal, se le debe conceder a una entidad de servicio un permiso adicional de la directiva de acceso, el permiso de purga. El permiso de purga de la directiva de acceso no se concede de manera predeterminada a ninguna entidad de servicio, ni siquiera a los propietarios de la suscripción y el almacén de claves, y se debe establecer de forma deliberada. Puesto que se exige un permiso elevado de la directiva de acceso para purgar un secreto eliminado temporalmente, se reduce la probabilidad de eliminar un secreto por accidente.

## <a name="supporting-interfaces"></a>Interfaces admitidas

La característica de eliminación temporal está disponible inicialmente mediante las interfaces de [REST](/rest/api/keyvault/), [CLI](soft-delete-cli.md), [PowerShell](soft-delete-powershell.md) y [.NET/C#](/dotnet/api/microsoft.azure.keyvault?view=azure-dotnet), así como [plantillas de ARM](https://docs.microsoft.com/azure/templates/microsoft.keyvault/2019-09-01/vaults).

## <a name="scenarios"></a>Escenarios

Los almacenes Azure Key Vault son recursos controlados que se administran por medio de Azure Resource Manager. Azure Resource Manager especifica además un comportamiento bien definido para su eliminación, lo que conlleva que una operación DELETE ejecutada correctamente debería traducirse en que ese recurso deje de estar accesible. La característica de eliminación temporal aborda la recuperación del objeto eliminado, con independencia de que la eliminación haya sido voluntaria o involuntaria.

1. En un escenario típico, un usuario puede haber eliminado por accidente una instancia o un objeto de Key Vault; si esa instancia o ese objeto de Key Vault fueran recuperables durante un período predeterminado, el usuario podría deshacer la eliminación y recuperar sus datos.

2. En un escenario diferente, un usuario malintencionado podría intentar eliminar una instancia o un objeto de Key Vault, como una clave dentro de un almacén, para provocar una interrupción de la actividad empresarial. El hecho de que al eliminar la instancia o el objeto de Key Vault no se eliminen los datos subyacentes puede resultar útil como medida de seguridad ya que, por ejemplo, se pueden limitar los permisos sobre la eliminación de datos a un rol diferente que sea de confianza. Este enfoque requiere un cuórum efectivo para una operación que, en caso contrario, podría provocar una pérdida de datos inmediata.

### <a name="soft-delete-behavior"></a>Comportamiento de eliminación temporal

Cuando está habilitada la eliminación temporal, los recursos marcados como recursos eliminados se conservan durante un período especificado (de forma predeterminada, 90 días). El servicio proporciona además un mecanismo para recuperar el objeto eliminado, básicamente deshaciendo la eliminación.

Al crear un almacén de claves, la eliminación temporal está activada de forma predeterminada. Puede crear un almacén de claves sin eliminación temporal a través de la [CLI de Azure](soft-delete-cli.md) o de [Azure PowerShell](soft-delete-powershell.md). Una vez que la eliminación temporal está habilitada en un almacén de claves, no se puede deshabilitar.

El período de retención predeterminado es de 90 días, durante la creación del almacén de claves, pero es posible establecer el intervalo de la directiva de retención en un valor de 7 a 90 días en Azure Portal. La directiva de retención de protección de purgas usa el mismo intervalo. Una vez establecido, el intervalo de la directiva de retención no se puede cambiar.

No se puede volver a usar el nombre de un almacén de claves que se ha eliminado temporalmente hasta que haya transcurrido el período de retención.

### <a name="purge-protection"></a>Protección de purgas 

La protección contra purgas es un comportamiento opcional de Key Vault y **no está habilitado de forma predeterminada**. La protección de purga solo se puede habilitar una vez habilitada la eliminación temporal.  Se puede activar a través de la [CLI](soft-delete-cli.md#enabling-purge-protection) o [Powershell](soft-delete-powershell.md#enabling-purge-protection).

Cuando la protección de purgas está activada, un almacén o un objeto en estado eliminado no se pueden purgar hasta que haya transcurrido el período de retención. Los objetos y los almacenes de eliminación temporal todavía se pueden recuperar, lo que garantiza que se seguirá la directiva de retención. 

El período de retención predeterminado es de 90 días, pero es posible establecer el intervalo de la directiva de retención en un valor de 7 a 90 días en Azure Portal. Una vez que el intervalo de la directiva de retención se establece y se guarda, no se puede cambiar en ese almacén. 

### <a name="permitted-purge"></a>Purga permitida

La purga permanente de una instancia de Key Vault es posible a través de una operación POST en el recurso de proxy y requiere privilegios especiales. Por lo general, el propietario de la suscripción es el único que puede purgar una instancia de Key Vault. La operación POST activa la eliminación inmediata y no recuperable de esa instancia. 

Las excepciones son estas:
- Cuando la suscripción de Azure se marcó como *no eliminable*. En este caso, solo el servicio puede realizar la eliminación real, y lo hace como un proceso programado. 
- Cuando `--enable-purge-protection flag` se habilita en el propio almacén. En este caso, Key Vault espera durante 90 días a partir de que el objeto de secreto original se marcara para eliminación para eliminar permanentemente el objeto.

### <a name="key-vault-recovery"></a>Recuperación de Key Vault

Tras eliminar una instancia de Key Vault, el servicio crea un recurso de proxy en la suscripción, agregando metadatos suficientes para la recuperación. El recurso de proxy es un objeto almacenado, disponible en la misma ubicación que la instancia de Key Vault eliminada. 

### <a name="key-vault-object-recovery"></a>Recuperación de un objeto de Key Vault

Cuando se elimina un objeto de Key Vault, como una clave, el servicio pondrá el objeto en estado "eliminado", de manera que no será accesible para ninguna operación de recuperación. Mientras esté en ese estado, el objeto de Key Vault solo se puede colocar en una lista, recuperar o eliminar por la fuerza/permanentemente. Para ver los objetos, use el comando `az keyvault key list-deleted` de la CLI de Azure (como se documenta en [Uso de la eliminación temporal de Key Vault con la CLI](soft-delete-cli.md)) o el parámetro `-InRemovedState` de Azure PowerShell (como se describe en [Uso de la eliminación temporal de Key Vault con PowerShell](soft-delete-powershell.md#secrets)).  

Al mismo tiempo, Key Vault programará la eliminación de los datos subyacentes correspondientes al objeto o la instancia de Key Vault eliminados para su ejecución tras un intervalo de retención predeterminado. El registro DNS correspondiente al almacén también se conserva durante la duración del intervalo de retención.

### <a name="soft-delete-retention-period"></a>Período de retención de la eliminación temporal

Los recursos eliminados temporalmente se conservan durante un período de tiempo determinado: 90 días. Durante el intervalo de retención de eliminación temporal, se dan las circunstancias siguientes:

- Puede confeccionar un listado con todos los objetos e instancias de Key Vault que se encuentran en estado de eliminación temporal en su suscripción, así como tener acceso a la información de eliminación y recuperación sobre ellos.
    - Solo los usuarios con permisos especiales pueden consultar los almacenes eliminados. Se recomienda que los usuarios creen un rol personalizado con estos permisos especiales para tratar los almacenes eliminados.
- No es posible crear una instancia de Key Vault con el mismo nombre en la misma ubicación; en consecuencia, no se puede crear un objeto de Key Vault en un almacén determinado si esa instancia de Key Vault contiene un objeto con el mismo nombre y está en estado eliminado. 
- Solo un usuario con privilegios determinados puede restaurar una instancia o un objeto de Key Vault mediante la emisión de un comando de recuperación en el recurso de proxy correspondiente.
    - El usuario, miembro del rol personalizado, que tiene privilegios para crear una instancia de Key Vault en el grupo de recursos puede restaurar el almacén.
- Solo un usuario con privilegios determinados puede eliminar por la fuerza una instancia o un objeto de Key Vault mediante la emisión de un comando de eliminación en el recurso de proxy correspondiente.

Una vez que finalice el intervalo de retención, el servicio lleva a cabo una purga de la instancia o el objeto de Key Vault eliminados temporalmente y de su contenido, a menos que se hayan recuperado. La eliminación de recursos no se puede volver a programar.

### <a name="billing-implications"></a>Implicaciones de facturación

En general, cuando un objeto (un almacén de claves, una clave o un secreto) está en estado eliminado, solo existen dos operaciones posibles: "purgarlo" y "recuperarlo". Se producirá un error en el resto de operaciones. Por consiguiente, aunque el objeto exista, no se puede realizar ninguna operación y, por tanto, no se producirá ningún uso ni ninguna factura. Sin embargo, existen las siguientes excepciones:

- las acciones "purgar" y "recuperar" contarán para las operaciones normales del almacén de claves y se facturarán.
- Si el objeto es una clave HSM, se aplicará el cargo de "Clave HSM protegida" por versión de clave al mes si se ha utilizado una versión de la clave en los últimos 30 días. Después, como el objeto está en estado eliminado, no podrá realizarse ninguna operación en él, por lo que no se aplicará ningún cargo.

## <a name="next-steps"></a>Pasos siguientes

Las dos guías siguientes ofrecen los escenarios de uso principal para uso de la eliminación temporal.

- [Uso de la eliminación temporal de Key Vault con PowerShell](soft-delete-powershell.md) 
- [Uso de la eliminación temporal de Key Vault con la CLI](soft-delete-cli.md)

