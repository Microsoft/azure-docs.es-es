---
title: Modelado de relaciones en el diseño de almacenamiento en tablas de Azure | Microsoft Docs
description: Comprenda el proceso de modelado al diseñar la solución de almacenamiento de Azure Table. Lea acerca de las relaciones uno a varios, uno a uno y de herencia.
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: 3023b478ef7a4aaf6d9239e997bdf63282b56210
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271199"
---
# <a name="modeling-relationships"></a>Modelado de relaciones
En este artículo se explica el proceso de modelado para ayudarle a diseñar soluciones de almacenamiento en tablas de Azure.

La creación de modelos de dominio es un paso clave en el diseño de sistemas complejos. Normalmente, usará el proceso de modelado para identificar las entidades y las relaciones entre ellas como manera de entender el dominio de negocio e informar del diseño del sistema. Esta sección se centra en cómo puede convertir algunos de los tipos de relación comunes encontrados en los modelos de dominio en diseños para Table service. El proceso de asignación de un modelo de datos lógico a uno físico basado en NoSQL es muy diferente del que se usa cuando se diseña una base de datos relacional. El diseño de bases de datos relacionales normalmente supone un proceso de normalización de datos optimizado para minimizar la redundancia y una capacidad de consulta declarativa que abstrae el modo de funcionamiento de la implementación de la base de datos.  

## <a name="one-to-many-relationships"></a>Relaciones uno a varios
Las relaciones uno a varios entre los objetos de dominio de negocio se producen con mucha frecuencia: por ejemplo, cuando un departamento tiene muchos empleados. Hay varias formas de implementar relaciones uno a varios en Table service, cada una con los correspondientes pros y contras para el escenario en concreto.  

Considere el ejemplo de una gran empresa multinacional con decenas de miles de departamentos y entidades de empleado en las que cada departamento tiene muchos empleados y cada empleado está asociado a un departamento determinado. Un enfoque consiste en almacenar el departamento independiente y entidades de empleado como las siguientes:  


![Almacenamiento de entidades de departamento y empleado independientes](media/storage-table-design-guide/storage-table-design-IMAGE01.png)

En este ejemplo se muestra una relación de uno a varios implícita entre los tipos basados en el valor **PartitionKey** . Cada departamento puede tener muchos empleados.  

En este ejemplo también se muestra una entidad de departamento y sus entidades relacionadas de empleado relacionadas en la misma partición. Puede elegir usar distintas particiones, tablas o incluso cuentas de almacenamiento para los diferentes tipos de entidad.  

Un enfoque alternativo es desnormalizar los datos y almacenar solo las entidades de empleado con datos sin normalizar de departamentos tal como se muestra en el ejemplo siguiente. En este escenario concreto, este enfoque sin normalizar puede que no sea el mejor si tiene un requisito para poder cambiar los detalles de un administrador de departamento porque para ello deberá actualizar a todos los empleados del departamento.  

![Entidad de empleado](media/storage-table-design-guide/storage-table-design-IMAGE02.png)

Para obtener más información, consulte más adelante en esta guía el [Patrón de desnormalización](table-storage-design-patterns.md#denormalization-pattern) .  

En la tabla siguiente se resumen las ventajas y desventajas de cada uno de los métodos descritos anteriormente para almacenar entidades de departamento y empleado que tienen una relación uno a varios. También debe tener en cuenta la frecuencia con que espera realizar varias operaciones: puede ser aceptable tener un diseño que incluye una operación costosa si esa operación solo ocurre con poca frecuencia.  

<table>
<tr>
<th>Enfoque</th>
<th>Ventajas</th>
<th>Desventajas</th>
</tr>
<tr>
<td>Tipos de entidad independientes, la misma partición, la misma tabla</td>
<td>
<ul>
<li>Puede actualizar una entidad de departamento con una sola operación.</li>
<li>Puede usar una transacción de un grupo de entidades* (EGT) para mantener la coherencia si tiene un requisito para modificar una entidad department siempre que se actualice, inserte o elimine una entidad de empleado. Por ejemplo, si mantiene un recuento de empleados de departamento para cada departamento.</li>
</ul>
</td>
<td>
<ul>
<li>Puede que necesite recuperar una entidad de empleado y de departamento para algunas actividades de cliente.</li>
<li>Las operaciones de almacenamiento se producen en la misma partición. Con volúmenes de transacciones elevados, esto puede producir un punto de conflicto.</li>
<li>No se puede mover a un empleado a un nuevo departamento mediante un EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Tipos de entidad independientes, particiones o tablas distintas o cuentas de almacenamiento</td>
<td>
<ul>
<li>Puede actualizar una entidad de departamento o de empleado con una sola operación.</li>
<li>Con volúmenes de transacciones altos, esto puede ayudar a distribuir la carga entre más particiones.</li>
</ul>
</td>
<td>
<ul>
<li>Puede que necesite recuperar una entidad de empleado y de departamento para algunas actividades de cliente.</li>
<li>No se pueden usar EGT para mantener la coherencia cuando se actualiza, inserta y elimina un empleado y se actualiza un departamento. Por ejemplo, actualizando un recuento de empleados en una entidad de departamento.</li>
<li>No se puede mover a un empleado a un nuevo departamento mediante un EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Desnormalizar en un único tipo de entidad</td>
<td>
<ul>
<li>Puede recuperar toda la información que necesita con una única solicitud.</li>
</ul>
</td>
<td>
<ul>
<li>Puede ser costoso de mantener la coherencia si necesita actualizar la información de departamento (esto requerirá actualizar a todos los empleados de un departamento).</li>
</ul>
</td>
</tr>
</table>

*Para más información, consulte [Transacciones de grupos de entidades](table-storage-design.md#entity-group-transactions).  


Cómo elegir entre estas opciones y cuáles de las ventajas y desventajas son más importantes, depende de los escenarios de aplicación concretos. Por ejemplo, la frecuencia con que modifica las entidades de departamento; necesitan todas las consultas de empleados la información adicional del departamento; ¿cómo de cerca se encuentra de los límites de escalabilidad de sus particiones o de su cuenta de almacenamiento?  

## <a name="one-to-one-relationships"></a>Relaciones uno a uno
Es posible que los modelos de dominio incluyan relaciones uno a uno entre las entidades. Si necesita implementar una relación uno a uno en Table service, también debe elegir cómo vincular las dos entidades relacionadas cuando se necesita para recuperar ambas. Este vínculo puede ser implícito, en función de una convención en los valores de clave o explícito almacenando un vínculo en el formulario de los valores **PartitionKey** y **RowKey** de cada entidad con su entidad relacionada. Para obtener una explicación sobre si debe almacenar las entidades relacionadas en la misma partición, consulte la sección [Relaciones uno a varios](#one-to-many-relationships).  

También hay que considerar algunos aspectos sobre la implementación que podrían llevarle a implementar relaciones uno a uno en Table service:  

* Administración de entidades de gran tamaño (para obtener más información, consulte [Patrón de entidades de gran tamaño](table-storage-design-patterns.md#large-entities-pattern)).  
* Implementación de controles de acceso (para más información, consulte Control de acceso con firmas de acceso compartido).  

## <a name="join-in-the-client"></a>Únase al cliente
Aunque hay formas de modelar las relaciones en Table service, no debe olvidar que las dos razones principales para utilizar Table service son la escalabilidad y el rendimiento. Si observa que está modelando muchas relaciones que ponen en peligro el rendimiento y la escalabilidad de su solución, debe preguntarse si es necesario crear todas las relaciones de datos en el diseño de tabla. Es posible que pueda simplificar el diseño y mejorar la escalabilidad y el rendimiento de la solución si permite que la aplicación cliente realice las uniones necesarias.  

Por ejemplo, si tiene tablas pequeñas que contienen datos que no cambian muy a menudo, puede recuperar estos datos una vez y almacenarlos en la caché en el cliente. Esto puede evitar repetidas idas y vueltas para recuperar los mismos datos. En los ejemplos que hemos visto en esta guía, es probable que el conjunto de departamentos de una organización pequeña sea pequeño y cambie con poca frecuencia, lo que lo convierte en un buen candidato para los datos que la aplicación cliente puede descargar una vez y almacenar en la memoria caché como datos de búsqueda.  

## <a name="inheritance-relationships"></a>Relaciones de herencia
Si la aplicación cliente usa un conjunto de clases que forman parte de una relación de herencia para representar entidades empresariales, puede conservar fácilmente las entidades en Table service. Por ejemplo, el siguiente conjunto de clases puede estar definido en la aplicación cliente, donde **Person** es una clase abstracta.

![Clase abstracta Person](media/storage-table-design-guide/storage-table-design-IMAGE03.png)

Puede conservar las instancias de las dos clases concretas en Table service utilizando una sola tabla Person mediante entidades que presentan este aspecto:  

![Tabla Person](media/storage-table-design-guide/storage-table-design-IMAGE04.png)

Para más información acerca de cómo trabajar con varios tipos de entidad en la misma tabla en el código de cliente, consulte la sección Trabajo con tipos de entidad heterogéneos en esta misma guía. Esto proporciona ejemplos de cómo reconocer el tipo de entidad en el código de cliente.  


## <a name="next-steps"></a>Pasos siguientes

- [Patrones de diseño de tablas](table-storage-design-patterns.md)
- [Diseño de consulta](table-storage-design-for-query.md)
- [Cifrado de datos de tabla](table-storage-design-encrypt-data.md)
- [Diseño para la modificación de datos](table-storage-design-for-modification.md)
