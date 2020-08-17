---
title: Conceptos sobre los conjuntos de réplicas de Azure AD Domain Services | Microsoft Docs
description: Conozca qué conjuntos de réplicas se encuentran en Azure Active Directory Domain Services y cómo proporcionan redundancia a las aplicaciones que necesitan servicios de identidad.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: iainfou
ms.openlocfilehash: 698009ee8a57ed5d30e01376b4f2c63b0a27ead8
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505540"
---
# <a name="replica-sets-concepts-and-features-for-azure-active-directory-domain-services-preview"></a>Conceptos y características de los conjuntos de réplicas de Azure Active Directory Domain Services (versión preliminar)

Cuando se crea un dominio administrado en Azure Active Directory Domain Services (Azure AD DS), se define un espacio de nombres único. Este espacio de nombres es el nombre de dominio (por ejemplo, *aaddscontoso.com*). A continuación, se implementan dos controladores de dominio (DC) en la región de Azure seleccionada. Esta implementación de controladores de dominio se conoce como "conjunto de réplicas".

Puede ampliar un dominio administrado para que tenga más de un conjunto de réplicas por cada inquilino de Azure AD. Los conjuntos de réplicas pueden agregarse a cualquier red virtual emparejada en cualquier región de Azure que admita Azure AD DS. Contar con conjuntos de réplicas adicionales en diferentes regiones de Azure facilita la recuperación geográfica ante desastres de las aplicaciones heredadas si una región de Azure se queda sin conexión.

Actualmente, los conjuntos de réplicas están en versión preliminar.

> [!NOTE]
> Los conjuntos de réplicas no permiten que se implementen varios dominios administrados únicos en el mismo inquilino de Azure. Cada conjunto de réplicas contiene los mismos datos.

## <a name="how-replica-sets-work"></a>Funcionamiento de los conjuntos de réplicas

Cuando se crea un dominio administrado (por ejemplo, *aaddscontoso.com*), se genera un conjunto de réplicas inicial. Los demás conjuntos de réplicas compartirán el mismo espacio de nombres y la misma configuración. Los cambios que se realicen en Azure AD DS, como los de configuración, identidad del usuario, credenciales, grupos, objetos de directiva de grupo, objetos de equipo y otros, se aplican a todos los conjuntos de réplicas del dominio administrado mediante la replicación de AD DS.

Cada conjunto de réplicas se crea en una red virtual. Cada red virtual debe estar emparejada con todas las demás redes virtuales que hospeden conjuntos de réplicas de un dominio administrado. Esta configuración crea una topología de red de malla que permite la replicación de directorios. Una red virtual puede admitir varios conjuntos de réplicas, siempre que cada uno de ellos esté en una subred virtual diferente.

Todos los conjuntos de réplicas se colocan en el mismo sitio de Active Directory. Como resultado, todos los cambios se propagan utilizando la replicación entre sitios, lo que permite una convergencia rápida.

> [!NOTE]
> No se pueden definir sitios diferentes y establecer la configuración de replicación entre conjuntos de réplicas.

En el diagrama siguiente, se muestra un dominio administrado con dos conjuntos de réplicas. El primer conjunto de réplicas se crea con el espacio de nombres del dominio. Después, se crea un segundo conjunto de réplicas:

![Diagrama de un dominio administrado de ejemplo con dos conjuntos de réplicas](./media/concepts-replica-sets/two-replica-set-example.png)

> [!NOTE]
> Los conjuntos de réplicas garantizan la disponibilidad de los servicios de autenticación en aquellas regiones en las que hay un conjunto de réplicas configurado. Para que una aplicación tenga redundancia geográfica si se produce una interrupción regional, la plataforma de aplicaciones que se basa en el dominio administrado también debe residir en la otra región.
>
> La resistencia de otros servicios necesarios para el funcionamiento de la aplicación, como las máquinas virtuales de Azure o las instancias de Azure App Services, no depende de los conjuntos de réplicas. Al diseñar la disponibilidad de otros componentes de la aplicación, deben tenerse en cuenta las características de resistencia de los servicios que conforman la aplicación.

En el ejemplo siguiente, se muestra un dominio administrado con tres conjuntos de réplicas para proporcionar resistencia y garantizar la disponibilidad de los servicios de autenticación. En ambos ejemplos, las cargas de trabajo de la aplicación se encuentran en la misma región que el conjunto de réplicas del dominio administrado:

![Diagrama de un dominio administrado de ejemplo con tres conjuntos de réplicas](./media/concepts-replica-sets/three-replica-set-example.png)

## <a name="deployment-considerations"></a>Consideraciones de la implementación

La SKU predeterminada de los dominios administrados es *Enterprise*, que admite varios conjuntos de réplicas. Si ha cambiado a la SKU *Estándar* y desea crear otros conjuntos de réplicas adicionales, [actualice el dominio administrado](change-sku.md) a *Enterprise* o *Premium*.

El número máximo de conjuntos de réplicas admitidos durante la versión preliminar es cuatro, incluida la primera réplica que se crea al generar el dominio administrado.

La facturación de cada conjunto de réplicas dependerá de la SKU de configuración del dominio. Por ejemplo, si tiene un dominio administrado que usa la SKU *Enterprise* y tiene tres conjuntos de réplicas, la suscripción se facturará por hora en cada uno de los tres conjuntos de réplicas.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="can-i-use-my-production-managed-domain-with-this-preview"></a>¿Puedo usar mi dominio administrado de producción con esta versión preliminar?

Los conjuntos de réplicas son una característica de Azure AD Domain Services que se encuentra en versión preliminar pública. Puede usar un dominio administrado de producción, pero debe tener en cuenta las diferencias de compatibilidad que existen con las características que aún están en versión preliminar. Para más información sobre las versiones preliminares, consulte el [Acuerdo de Nivel de Servicio de la versión preliminar de Azure Active Directory](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="can-i-create-a-replica-set-in-subscription-different-from-my-managed-domain"></a>¿Puedo crear un conjunto de réplicas en una suscripción diferente a la del dominio administrado?

No. Los conjuntos de réplicas deben estar en la misma suscripción que el dominio administrado.

### <a name="how-many-replica-sets-can-i-create"></a>¿Cuántos conjuntos de réplicas puedo crear?

La versión preliminar permite crear un máximo de cuatro conjuntos de réplicas: el conjunto de réplicas inicial del dominio administrado y tres conjuntos de réplicas adicionales.

### <a name="how-does-user-and-group-information-get-synchronized-to-my-replica-sets"></a>¿Cómo se sincroniza la información de los usuarios y los grupos con los conjuntos de réplicas?

Todos los conjuntos de réplicas están conectados entre sí mediante un emparejamiento de red virtual de malla. Cada conjunto de réplicas recibe actualizaciones de los usuarios y los grupos de Azure AD. Estos cambios se replican después en los otros conjuntos de réplicas utilizando la replicación de AD DS dentro del sitio a través de la red emparejada.

Al igual que con las instancias locales de AD DS, una desconexión prolongada podría producir una interrupción en la replicación. Como las redes virtuales emparejadas no son transitivas, los requisitos de diseño de los conjuntos de réplicas requieren una topología de red que tenga totalmente forma de malla.

### <a name="how-do-i-make-changes-in-my-managed-domain-after-i-have-replica-sets"></a>¿Cómo puedo realizar cambios en el dominio administrado si ya tengo conjuntos de réplicas?

Los cambios en el dominio administrado funcionan igual que antes. [Hay que crear y usar una máquina virtual de administración con las herramientas de RSAT que esté unida al dominio administrado](tutorial-create-management-vm.md). Puede unir al dominio administrado tantas máquinas virtuales de administración como desee.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con los conjuntos de réplicas, [cree y configure un dominio administrado de Azure AD DS][tutorial-create-advanced]. Cuando realice la implementación, [cree y utilice otros conjuntos de réplicas adicionales][create-replica-set].

<!-- LINKS - INTERNAL -->
[tutorial-create-advanced]: tutorial-create-instance-advanced.md
[create-replica-set]: tutorial-create-replica-set.md
