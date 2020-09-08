---
title: 'Tutorial: Creación de un conjunto de réplicas en Azure AD Domain Services | Microsoft Docs'
description: Aprenda a crear y usar conjuntos de réplicas en Azure Portal para mejorar la resistencia de los servicios con Azure AD Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: iainfou
ms.openlocfilehash: 6f166cdcb5f3764d7b264fdb4ebc082ece4c798b
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245101"
---
# <a name="tutorial-create-and-use-replica-sets-for-resiliency-or-geolocation-in-azure-active-directory-domain-services-preview"></a>Tutorial: Creación y uso de conjuntos de réplicas para soluciones de resistencia o ubicación geográfica en Azure Active Directory Domain Services (versión preliminar)

Para mejorar la resistencia de un dominio administrado por Azure Active Directory Domain Services (Azure AD DS), o para realizar la implementación en ubicaciones geográficas adicionales próximas a las aplicaciones, puede usar *conjuntos de réplicas*. Cada espacio de nombres de dominio administrado por Azure AD DS, como, por ejemplo, *aaddscontoso.com*, contiene un conjunto de réplicas inicial. La posibilidad de crear conjuntos de réplicas adicionales en otras regiones de Azure proporciona resistencia geográfica a los dominios administrados.

Puede agregar un conjunto de réplicas a cualquier red virtual emparejada de cualquier región de Azure que admita Azure AD DS.

Los conjuntos de réplicas son una característica en versión preliminar pública de Azure AD Domain Services. Tenga en cuenta las diferencias de compatibilidad que existen con las características que aún están en versión preliminar. Para más información sobre las versiones preliminares, consulte el [Acuerdo de Nivel de Servicio de la versión preliminar de Azure Active Directory](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprender los requisitos de la red virtual
> * Crear un conjunto de réplicas
> * Eliminar un conjunto de réplicas

Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services creado mediante un modelo de implementación de Azure Resource Manager y configurado en su inquilino de Azure AD.
    * Si es necesario, [cree y configure un dominio administrado de Azure Active Directory Domain Services][tutorial-create-instance].

    > [!IMPORTANT]
    > Los dominios administrados creados con el modelo de implementación clásica no pueden utilizar conjuntos de réplicas. También debe usar como mínimo la SKU *Enterprise* para el dominio administrado. Si es necesario, [cambie la SKU de un dominio administrado][howto-change-sku].

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

En este tutorial, creará y administrará conjuntos de réplicas mediante Azure Portal. Para empezar a trabajar, primero inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="networking-considerations"></a>Consideraciones sobre redes

Las redes virtuales que hospedan conjuntos de réplicas deben ser capaces de comunicarse entre sí. Las aplicaciones y servicios que dependen de Azure AD DS también necesitan conectividad de red a las redes virtuales que hospedan los conjuntos de réplicas. El emparejamiento de red virtual de Azure debe configurarse entre todas las redes virtuales para crear una red totalmente de malla. Estos emparejamientos permiten la replicación dentro del sitio eficaz entre conjuntos de réplicas.

Antes de usar conjuntos de réplicas en Azure AD DS, revise los siguientes requisitos de red virtual de Azure:

* Evite que los espacios de direcciones IP se solapen a fin de permitir el emparejamiento y el enrutamiento de redes virtuales.
* Cree subredes con suficientes direcciones IP para admitir su escenario.
* Asegúrese de que Azure AD DS tenga su propia subred. No comparta esta subred de red virtual con servicios y máquinas virtuales de aplicaciones.
* Las redes virtuales emparejadas no son transitivas.

> [!TIP]
> Cuando se crea un conjunto de réplicas en Azure Portal, se crean automáticamente los emparejamientos entre las redes virtuales.
>
> Si es necesario, puede crear una red virtual y una subred al agregar un conjunto de réplicas en Azure Portal. También, puede elegir los recursos de red virtual existentes en la región de destino para un conjunto de réplicas y dejar que los emparejamientos se creen automáticamente si aún no existen.

## <a name="create-a-replica-set"></a>Creación de un conjunto de réplicas

Al crear un dominio administrado, como *aaddscontoso.com*, se crea un conjunto de réplicas inicial. Los demás conjuntos de réplicas compartirán el mismo espacio de nombres y la misma configuración. Los cambios en Azure AD DS, como los de configuración, identidad del usuario, credenciales, grupos, objetos de directiva de grupo, objetos de equipo y otros cambios, se aplican a todos los conjuntos de réplicas del dominio administrado mediante la replicación de AD DS.

En este tutorial, creará un conjunto de réplicas adicional en una región de Azure diferente a la del conjunto de réplicas inicial de Azure AD DS.

Para crear un conjunto de réplicas adicional, haga lo siguiente:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**.
1. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. En el lado izquierdo, seleccione **Conjuntos de réplicas (versión preliminar)** . Cada dominio administrado incluye un conjunto de réplicas inicial en la región seleccionada, tal como se muestra en la siguiente captura de pantalla de ejemplo:

    ![Captura de pantalla de ejemplo para ver y agregar un conjunto de réplicas en Azure Portal](./media/tutorial-create-replica-set/replica-set-list.png)

    Para crear un conjunto de réplicas adicional, seleccione **+ Agregar**.

1. En la ventana *Add a replica set* (Agregar un conjunto de réplicas), seleccione la región de destino, por ejemplo, *Este de EE. UU.* .

    Seleccione una red virtual de la región de destino, como *vnet-eastus* y, luego, elija una subred, como *aadds-subnet*. Si es necesario, elija **Crear nuevo** para agregar una red virtual en la región de destino y, luego, **Administrar** para crear una subred para Azure AD DS.

    Si aún no existen, los emparejamientos de redes virtuales de Azure se crean automáticamente entre la red virtual del dominio administrado existente y la red virtual de destino.

    En la captura de pantalla de ejemplo siguiente se muestra el proceso para crear un conjunto de réplicas en la región *Este de EE. UU.* :

    ![Captura de pantalla de ejemplo para crear un conjunto de réplicas en Azure Portal](./media/tutorial-create-replica-set/create-replica-set.png)

1. Cuando esté preparado, seleccione **Guardar**.

El proceso para crear el conjunto de réplicas tarda algún tiempo a medida que los recursos se crean en la región de destino. Después, el dominio administrado propiamente dicho se replica mediante la replicación de AD DS.

El conjunto de réplicas se presenta como *Aprovisionando* a medida que la implementación continúa, tal como se muestra en la siguiente captura de pantalla de ejemplo. Una vez finalizada, el conjunto de réplicas se muestra como *En ejecución*.

![Captura de pantalla de ejemplo del estado de implementación del conjunto de réplicas en Azure Portal](./media/tutorial-create-replica-set/replica-set-provisioning.png)

## <a name="delete-a-replica-set"></a>Eliminar un conjunto de réplicas

Un dominio administrado está actualmente limitado a cuatro réplicas: el conjunto de réplicas inicial y tres conjuntos de réplicas adicionales. Si ya no necesita un conjunto de réplicas, o si quiere crear un conjunto de réplicas en otra región, puede eliminar los conjuntos de réplicas innecesarios.

> [!IMPORTANT]
> Sin embargo, no se puede eliminar el último conjunto de réplicas de un dominio administrado.

Para eliminar una red virtual, haga lo siguiente:

1. En Azure Portal, busque y seleccione **Azure AD Domain Services**.
1. Elija el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. En el lado izquierdo, seleccione **Conjuntos de réplicas (versión preliminar)** . En la lista de conjuntos de réplicas, seleccione el menú contextual **...** situado junto al conjunto de réplicas que quiere eliminar.
1. Seleccione **Eliminar** en el menú contextual y, luego, confirme que quiere eliminar el conjunto de réplicas.

> [!NOTE]
> La eliminación del conjunto de réplicas puede ser una operación lenta.

Si ya no necesita la red virtual o el emparejamiento que usa el conjunto de réplicas, también puede eliminar esos recursos. Sin embargo, antes, asegúrese de que ningún otro recurso de aplicación de la otra región necesite las conexiones de red.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Configuración del emparejamiento de red virtual
> * Crear un conjunto de réplicas en una región geográfica diferente
> * Eliminar un conjunto de réplicas

Para más información conceptual, aprenda cómo funcionan los conjuntos de réplicas en Azure AD DS.

> [!div class="nextstepaction"]
> [Conceptos y características de los conjuntos de réplicas][concepts-replica-sets]

<!-- INTERNAL LINKS -->
[replica-sets]: concepts-replica-sets.md
[tutorial-create-instance]: tutorial-create-instance-advanced.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[howto-change-sku]: change-sku.md
[concepts-replica-sets]: concepts-replica-sets.md
