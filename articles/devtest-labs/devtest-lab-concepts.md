---
title: Conceptos de DevTest Labs | Microsoft Docs
description: Aprenda los conceptos básicos de DevTest Labs y cómo puede facilitar la creación, la administración y la supervisión de máquinas virtuales de Azure
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 43d62d637686a785cafd29aa311ccf20cb942721
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283850"
---
# <a name="devtest-labs-concepts"></a>Conceptos de DevTest Labs
## <a name="overview"></a>Información general
La lista siguiente contiene las definiciones y los conceptos clave de DevTest Labs:

## <a name="labs"></a>Laboratorios
Un laboratorio es la infraestructura que abarca un grupo de recursos, como máquinas virtuales (VM), que permite una mejor administración de esos recursos especificando límites y cuotas.

## <a name="virtual-machine"></a>Máquina virtual
Una máquina virtual de Azure es uno de los distintos tipos de [recursos informáticos a petición y escalables](/azure/architecture/guide/technology-choices/compute-decision-tree) que ofrece Azure. Las máquinas virtuales de Azure proporcionan la flexibilidad de virtualización sin necesidad de comprar y mantener el hardware físico que la ejecuta, aunque todavía es necesario mantener la máquina virtual mediante la realización de ciertas tareas, como configurar, aplicar revisiones e instalar el software que se ejecuta en la máquina virtual.

[Información general sobre las máquinas virtuales Windows en Azure](../virtual-machines/windows/overview.md) proporciona información sobre lo que se debe considerar antes de crear una máquina virtual, cómo crearla y cómo administrarla.

## <a name="claimable-vm"></a>Creación de máquinas virtuales reclamables
Una máquina virtual reclamable de Azure Claimable es una que está disponible para que pueda usarla cualquier usuario de laboratorio con permisos. Un administrador de laboratorio puede preparar las máquinas virtuales con los artefactos y las imágenes base específicas y guardarlas en un grupo compartido. Luego, un usuario de laboratorio puede solicitar una máquina virtual de trabajo del grupo cuando necesite una con esa configuración concreta.

Una máquina virtual reclamable no se asigna inicialmente a ningún usuario concreto, pero se mostrará en la lista de todos los usuarios en "Claimable virtual machines" (Máquinas virtuales reclamables). Después de que un usuario reclame una máquina virtual, se mueve hasta su área "Mis máquinas virtuales" y ya no podrá reclamarla otro usuario.

## <a name="environment"></a>Entorno
En DevTest Labs, un entorno hace referencia a una colección de recursos de Azure en un laboratorio. [Esta entrada de blog](./devtest-lab-faq.md#blog-post) describe cómo crear entornos de varias máquinas virtuales a partir de las plantillas de Azure Resource Manager.

## <a name="base-images"></a>Imágenes base
Las imágenes base son imágenes de máquina virtual con todas las herramientas y los valores de configuración preinstalados y configurados para crear rápidamente una máquina virtual. Puede aprovisionar una máquina virtual seleccionando una base existente y agregando un artefacto para instalar su agente de prueba. Luego puede guardar la máquina virtual aprovisionada como base para que esta pueda utilizarse sin que sea necesario volver a instalar al agente de prueba para cada aprovisionamiento de la máquina virtual.

## <a name="artifacts"></a>Artefactos
Los artefactos se utilizan para implementar y configurar la aplicación después de aprovisionar una máquina virtual. Los artefactos pueden ser:

* Herramientas que desea instalar en la máquina virtual, como agentes, Fiddler y Visual Studio.
* Acciones que desea ejecutar en la máquina virtual, como la clonación de un repositorio.
* Aplicaciones que desea probar.

Los artefactos son archivos JSON basados de [Azure Resource Manager](../azure-resource-manager/management/overview.md) que contienen instrucciones para realizar la implementación y aplicar la configuración.

## <a name="artifact-repositories"></a>Repositorios de artefacto
Los repositorios de artefactos son repositorios de Git donde se insertan los artefactos. Es posible agregar los repositorios de artefactos a varios laboratorios de la organización a fin de reutilizarlos y compartirlos.

## <a name="formulas"></a>Fórmulas
Las fórmulas, además de las imágenes base, proporcionan un mecanismo para el aprovisionamiento rápido de la máquina virtual. Una fórmula en DevTest Labs es una lista de valores de propiedad predeterminados que se usan para crear una máquina virtual de laboratorio.
Mediante las fórmulas, las máquinas virtuales que tengan el mismo conjunto de propiedades, como la imagen base, el tamaño de máquina virtual, la red virtual y los artefactos, se pueden crear sin necesidad de especificar esas propiedades todas las veces. Al crear una máquina virtual a partir de una fórmula, los valores predeterminados pueden usarse tal como están o modificarlos.

## <a name="policies"></a>Directivas
Las directivas ayudan a controlar los costos en su laboratorio. Por ejemplo, puede crear una directiva para apagar automáticamente las máquinas virtuales según una programación definida.

## <a name="caps"></a>Límites
Los límites son un mecanismo que sirve para minimizar la pérdida en el laboratorio. Por ejemplo, puede establecer un límite para restringir el número de máquinas virtuales que se pueden crear por usuario o en un laboratorio.

## <a name="security-levels"></a>Niveles de seguridad
El acceso de seguridad lo determina el control de acceso basado en roles de Azure (Azure RBAC). Para entender cómo funciona el acceso, es importante comprender las diferencias entre un permiso, un rol y un ámbito según se define en RBAC.

* Permiso: un permiso es un acceso definido para una acción específica (por ejemplo, acceso de lectura a todas las máquinas virtuales).
* Rol: un rol es un conjunto de permisos que se pueden agrupar y asignar a un usuario. Por ejemplo, el rol de *propietario de la suscripción* tiene acceso a todos los recursos dentro de una suscripción.
* Ámbito: un ámbito es un nivel dentro de la jerarquía un recurso de Azure (por ejemplo, un grupo de recursos, un único laboratorio o toda la suscripción).

Dentro del ámbito de DevTest Labs, hay dos tipos de roles para definir los permisos de usuario: usuario de laboratorio y propietario de laboratorio.

* Propietario de laboratorio: el propietario de un laboratorio tiene acceso a todos los recursos del laboratorio. Por lo tanto, el propietario de un laboratorio puede modificar las directivas, leer y escribir en todas las máquinas virtuales, cambiar la red virtual, etc.
* Usuario de laboratorio: un usuario de laboratorio puede ver todos los recursos de laboratorio, como máquinas virtuales, directivas y redes virtuales, pero no puede modificar las directivas ni las máquinas virtuales creadas por otros usuarios.

Para ver cómo crear roles personalizados en DevTest Labs, consulte el artículo [Concesión de permisos de usuario a directivas específicas de laboratorio](devtest-lab-grant-user-permissions-to-specific-lab-policies.md).

Puesto que los ámbitos son jerárquicos, cuando un usuario tiene permisos en un ámbito determinado, también se le conceden automáticamente en cada ámbito de nivel inferior que engloba. Por ejemplo, si un usuario está asignado al rol de propietario de la suscripción, tiene acceso a todos los recursos de una suscripción, lo que incluye todas las máquinas virtuales, todas las redes virtuales y todos los laboratorios. Por lo tanto, el propietario de una suscripción hereda automáticamente el rol de propietario de laboratorio. Sin embargo, lo contrario no es cierto. El propietario de un laboratorio tiene acceso a un laboratorio, que es un ámbito inferior al del nivel de suscripción. Por lo tanto, el propietario de un laboratorio no podrá ver las máquinas virtuales, las redes virtuales o los recursos que se encuentren fuera del laboratorio.

## <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure
Todos los conceptos tratados en este artículo pueden configurarse mediante el uso de plantillas de Azure Resource Manager, que permiten definir la configuración y la infraestructura de la solución de Azure e implementarla varias veces en un estado coherente.

[Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md#template-format) describe la estructura de una plantilla de Azure Resource Manager y las propiedades que están disponibles en las diferentes secciones de ella.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Pasos siguientes
[Creación de un laboratorio en DevTest Labs](devtest-lab-create-lab.md)
