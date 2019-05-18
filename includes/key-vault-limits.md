---
author: rothja
ms.service: billing
ms.topic: include
ms.date: 11/09/2018
ms.author: jroth
ms.openlocfilehash: 0b9d87fd7929607da8407ae5bbfb2f6dd6d69dab
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65212044"
---
#### <a name="key-transactions-maximum-transactions-allowed-in-10-seconds-per-vault-per-regionsup1sup"></a>Transacciones clave (número máximo de transacciones permitido en 10 segundos, por almacén y región<sup>1</sup>):

|Tipo de clave|Clave de HSM<br>Crear clave|Clave de HSM<br>Las restantes transacciones|Clave de software<br>Crear clave|Clave de software<br>Las restantes transacciones|
|:---|---:|---:|---:|---:|
|RSA de 2048 bits|5|1000|10|2.000|
|3.072 bits de RSA|5|250|10|500|
|RSA de 4096 bits|5|125|10|250|
|ECC P-256|5|1000|10|2.000|
|ECC P-384|5|1000|10|2.000|
|ECC P-521|5|1000|10|2.000|
|ECC SECP256K1|5|1000|10|2.000|

> [!NOTE]
> En la tabla anterior, vemos que para las claves de software de RSA 2048 bits, se permiten 2.000 transacciones GET cada 10 segundos. RSA 2048 bits de las claves HSM, se permiten los 1000 transacciones GET cada 10 segundos.
>
> Los umbrales de limitación se ponderan y el cumplimiento es en su suma. Por ejemplo, tal como se muestra en la tabla anterior, al realizar las operaciones GET en las claves RSA HSM, es ocho veces más costoso de usar claves de 4.096 bits en comparación con las claves de 2048 bits. Eso es porque 1000/125 = 8.
>
> En un determinado intervalo de 10 segundos, puede hacer un cliente de Azure Key Vault *sola* de las siguientes operaciones antes de encontrar un `429` limitación código de estado HTTP:
> - 2.000 transacciones GET de clave de software de RSA 2048 bits
> - 1000 transacciones de RSA 2048 bits clave de HSM GET
> - 125 transacciones RSA 4096 bits clave de HSM GET
> - Las transacciones de clave de HSM RSA 4096 bits GET 124 y 8 RSA 2048 bits clave de HSM GET

#### <a name="secrets-managed-storage-account-keys-and-vault-transactions"></a>Los secretos, claves de cuenta de almacenamiento administrado y transacciones de almacén:
| Tipo de transacciones | Número máximo de transacciones permitido en 10 segundos, por almacén y región<sup>1</sup> |
| --- | --- |
| Todas las transacciones |2.000 |

Para obtener información sobre cómo controlar la limitación cuando se superan estos límites, consulte [guía sobre la limitación de Azure Key Vault](../articles/key-vault/key-vault-ovw-throttling.md).

<sup>1</sup> un límite de toda la suscripción para todos los tipos de transacción es cinco veces por límite de almacén de claves. Por ejemplo, HSM-otras transacciones por cada suscripción se limita a 5000 transacciones en 10 segundos por suscripción.
