---
title: Uso de claves únicas en Azure Cosmos DB
description: Aprenda a usar las claves únicas en la base de datos de Azure Cosmos DB
author: rimman
ms.author: rimman
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/08/2019
ms.reviewer: sngun
ms.openlocfilehash: c3524f27cc14050647c457fdb5e87b2b83dbdfc0
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595868"
---
# <a name="unique-key-constraints-in-azure-cosmos-db"></a>Restricciones de clave únicas de Azure Cosmos DB

Las claves únicas agregan una capa de integridad de datos a un contenedor de Azure Cosmos. Cree una directiva de clave única cuando cree un contenedor de Azure Cosmos. Con las claves únicas, se garantiza que uno o varios valores dentro de una partición lógica son únicos. También puede garantizar la exclusividad por [clave de partición](partition-data.md). 

Después de crear un contenedor con una directiva de clave única, se impide la creación de una nueva o una actualización de un elemento existente, lo que provocará un duplicado dentro de una partición lógica, tal y como especifica la restricción de clave única. La clave de partición combinada con la clave única garantiza la exclusividad de un elemento dentro del ámbito del contenedor.

Por ejemplo, considere un contenedor de Azure Cosmos con la dirección de correo electrónico como restricción de clave única y `CompanyID` como clave de partición. Cuando configura la dirección de correo electrónico del usuario con una clave única, cada elemento tiene una dirección de correo electrónico única dentro de un ámbito `CompanyID` determinado. No se pueden crear dos elementos con direcciones de correo electrónico duplicadas y con el mismo valor de clave de partición. 

Para crear elementos con la misma dirección de correo electrónico, pero no con el mismo nombre, apellidos y dirección de correo electrónico, agregue otras rutas de acceso a la directiva de claves únicas. En lugar de crear una clave única basada en la dirección de correo electrónico solo, también puede crear una clave única con una combinación del nombre, apellidos y dirección de correo electrónico. Esta clave se conoce como clave única compuesta. En este caso, se permite cada combinación única de los tres valores dentro de un ámbito `CompanyID` determinado. 

Por ejemplo, el contenedor puede contener elementos con los valores siguientes, donde cada elemento respeta la restricción de clave única.

|CompanyID|Nombre|Apellido|Dirección de correo electrónico|
|---|---|---|---|
|Contoso|Gaby|Duperre|gaby@contoso.com |
|Contoso|Gaby|Duperre|gaby@fabrikam.com|
|Fabrikam|Gaby|Duperre|gaby@fabrikam.com|
|Fabrikam|Ivan|Duperre|gaby@fabrikam.com|
|Fabrikam|   |Duperre|gaby@fabraikam.com|
|Fabrikam|   |   |gaby@fabraikam.com|

Si intenta insertar otro elemento con las combinaciones que aparecen en la tabla anterior, recibirá un error. El error indica que no se ha cumplido con la restricción de clave única. Recibir cualquiera `Resource with specified ID or name already exists` o `Resource with specified ID, name, or unique index already exists` como un mensaje de vuelta. 

## <a name="define-a-unique-key"></a>Definición de una clave única

Puede definir claves únicas solo cuando se crea un contenedor de Azure Cosmos. Una clave única se limita a una partición lógica. En el ejemplo anterior, si crea particiones del contenedor según el código postal, al final tendrá elementos duplicados en cada partición lógica. Tenga en cuenta las siguientes propiedades al crear claves únicas:

* No puede actualizar un contenedor existente para que use una clave única distinta. En otras palabras, una vez creado un contenedor con una directiva de clave única, la directiva no se puede cambiar.

* Para establecer una clave única para un contenedor existente, cree un nuevo contenedor con la restricción de clave única. Utilice la herramienta de migración de datos adecuada para mover los datos desde el contenedor existente al nuevo contenedor. Para los contenedores de SQL, use la [Herramienta de migración de datos](import-data.md) para mover los datos. Para los contenedores de MongoDB, utilice [mongoimport.exe o mongorestore.exe](mongodb-migrate.md) para mover los datos.

* Una directiva de clave única puede tener un máximo de 16 valores de ruta de acceso. Por ejemplo, los valores pueden ser `/firstName`, `/lastName`, y `/address/zipCode`. Cada directiva de clave única puede tener un máximo de 10 restricciones o combinaciones de clave única. Las rutas de acceso combinadas para cada restricción de índice única no deben superar los 60 bytes. En el ejemplo anterior, el nombre, el apellido y la dirección de correo electrónico de forma conjunta constituyen solo una restricción. Esta restricción utiliza tres de las dieciséis posibles rutas de acceso.

* Cuando un contenedor tiene una directiva de clave única, [unidad de solicitud (RU)](request-units.md) cargos para crear, actualizar y eliminar un elemento son ligeramente superiores.

* No se admiten las claves únicas dispersas. Si faltan algunos valores de ruta de acceso única, se tratan como valores NULL, que participan en la restricción de exclusividad. Por este motivo, solo puede haber un único elemento con un valor NULL para cumplir esta restricción.

* Los nombres de clave única distinguen mayúsculas de minúsculas. Por ejemplo, considere la posibilidad de un contenedor con la restricción de clave única establecida en `/address/zipcode`. Si los datos tienen un campo denominado `ZipCode`, Azure Cosmos DB inserta "null" como clave única porque `zipcode` no es igual a `ZipCode`. Debido a esta distinción entre mayúsculas y minúsculas, todos los demás registros con "Código Postal" no se podrán insertar porque un valor "null" duplicado infringirá la restricción de clave única.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las [particiones lógicas](partition-data.md)
* Explorar [cómo definir las claves únicas](how-to-define-unique-keys.md) al crear un contenedor
