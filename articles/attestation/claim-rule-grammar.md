---
title: Gramática de las reglas de notificaciones de Azure Attestation
description: Detalles sobre las notificaciones de directivas y las reglas de notificaciones de Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: a0f3e517e60037de6456bc3a549e072e58e2fa67
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89244745"
---
# <a name="claim-and-claim-rules"></a>Notificaciones y reglas de notificaciones

Para comprender la gramática de las reglas de notificación, debe comprender las notificaciones de la directiva de atestación.

## <a name="claim"></a>Notificación

Una notificación es un conjunto de propiedades agrupadas para proporcionar información pertinente. En el caso de Azure Attestation, contiene las propiedades siguientes:

- **type**: un valor de cadena que representa el tipo de notificación.
- **value**: un valor booleano, entero o de cadena que representa el valor de la notificación.
- **valueType**: el tipo de datos de la información almacenada en la propiedad value. Los tipos admitidos son String, Integer y Boolean. Si no se define, el valor predeterminado será String.
- **issuer**: información relacionada con el emisor de la notificación. El emisor será uno de los siguientes tipos:
  - **AttestationService**: Azure Attestation pone ciertas notificaciones a disposición del autor de la directiva. El autor de la directiva de atestación puede usarlas para elaborar la directiva adecuada.
  - **AttestationPolicy**: la directiva (tal como el administrador la define) puede agregar notificaciones a la evidencia entrante durante el procesamiento. En este caso, el emisor se establece en "AttestationPolicy".
  - **CustomClaim**: el atestador (cliente) también puede agregar notificaciones adicionales a la evidencia de atestación. En este caso, el emisor se establece en "CustomClaim".

Si no está definido, el valor predeterminado será "CustomClaim".

## <a name="claim-rule"></a>Regla de notificaciones

El motor de directivas utiliza el conjunto de notificaciones entrantes para calcular el resultado de la atestación. Una regla de notificación es un conjunto de condiciones que se usan para validar las notificaciones entrantes y realizar la acción definida.

```
Conditions list => Action (Claim)
```

La evaluación de Azure Attestation de una regla de notificaciones implica los siguientes pasos:

- Si la lista de condiciones no está presente, se ejecuta la acción con la notificación especificada. 
- De lo contrario, se evalúan las condiciones de la lista.
- Si la lista de condiciones se evalúa como false, se detiene. De lo contrario, se continúa.

Las condiciones de una regla de notificaciones se usan para determinar si es necesario ejecutar la acción. La lista de condiciones es una secuencia de condiciones separadas por el operador "&&".

La lista de condiciones se estructura como:

```
Condition && Condition && ...
```

La condición se estructura como:

```
Identifier:[ClaimPropertyCondition, ClaimPropertyCondition,…]
```

La condición se compone de condiciones individuales en varias propiedades de una notificación. Una condición puede tener un identificador opcional, que se puede usar para hacer referencia a las notificaciones que cumplan la condición. Esta referencia se puede usar en las demás condiciones o en la acción de la misma regla.

Por ejemplo

```
F1:[type=="OSName" , issuer=="CustomClaim"] && 
[type=="OSName" , issuer=="AttestationService", value== F1.value ] 
=> issueproperty(type="report_validity_in_minutes", value=1440);

F1:[type=="OSName" , issuer=="CustomClaim"] && 
C2:[type=="OSName" , issuer=="AttestationService", value== F1.value ] 
=> issue(claim = C2);
```

A continuación se muestran los operadores que se pueden usar para comprobar las condiciones:

| Valuetype | Operaciones admitidas |
|--|--|
| Entero | == (igual), \!= (distinto), <= (menor o igual que), < (menor que), >= (mayor o igual que), > (mayor que) |
| String | == (igual), \!= (distinto) |
| Boolean | == (igual), \!= (distinto) |

Evaluación de la lista de condiciones:

- La presencia del operador "&&" implica que una lista de condiciones solo se evalúe como true si todas ellas se evalúan como true.
- Una condición representa los criterios de filtro en el conjunto de notificaciones. Se dice que la propia condición se evalúa como true si se encuentra al menos una notificación que satisfaga la condición.
- Se dice que una notificación satisface el criterio de filtrado representado por la condición si cada una de sus propiedades satisface las condiciones de propiedad de notificaciones correspondientes presentes en la condición.  

A continuación se describe el conjunto de acciones que se permiten en una directiva.

| Verbo de acción | Descripción | Secciones de la directiva a las que se aplican |
|--|--|--|
| permit() | El conjunto de notificaciones entrantes se puede usar para calcular **issuancerules**. No toma ninguna notificación como parámetro | **authorizationrules** |
| deny() | El conjunto de notificaciones entrantes no debe usarse para calcular **issuancerules**. No toma ninguna notificación como parámetro. | **authorizationrules** |
| add(claim) | Agrega la notificación al conjunto de notificaciones entrantes. Cualquier notificación agregada al conjunto de notificaciones entrantes estará disponible para las reglas de notificación posteriores. |**authorizationrules**, **issuancerules** |
| issue(claim) | Agrega la notificación al conjunto de notificaciones entrantes y salientes | **issuancerules** |
| issueproperty(claim) | Agrega la notificación al conjunto de notificaciones entrantes y de propiedad | **issuancerules**

## <a name="next-steps"></a>Pasos siguientes

- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)

