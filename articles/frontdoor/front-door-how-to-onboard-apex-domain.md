---
title: 'Incorporación de un dominio raíz o de vértice a una instancia existente de Front Door: Azure Portal'
description: Obtenga información sobre cómo incorporar un dominio raíz o de vértice a una instancia existente de Front Door mediante Azure Portal.
services: front-door
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 5/21/2019
ms.author: duau
ms.openlocfilehash: 05267ad43f6e7f89ec50b1765d2475a02fae1702
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399595"
---
# <a name="onboard-a-root-or-apex-domain-on-your-front-door"></a>Incorporación de un dominio raíz o de vértice a una instancia de Front Door
Azure Front Door usa registros CNAME a fin de validar la propiedad del dominio para la incorporación de dominios personalizados. Además, Front Door no expone la dirección IP de front-end asociada al perfil de Front Door, por lo que no podrá asignar su dominio de vértice a una dirección IP, si lo que pretende es incorporarlo a Azure Front Door.

El protocolo DNS evita la asignación de registros CNAME en el vértice de zona. Por ejemplo, si el dominio es `contoso.com`, puede crear registros CNAME para `somelabel.contoso.com`, pero no puede crear CNAME para `contoso.com`. Esta restricción presenta un problema para los propietarios de aplicaciones que tienen aplicaciones con equilibrio de carga detrás de Azure Front Door. Puesto que el uso de un perfil de Front Door requiere la creación de un registro CNAME, no es posible apuntar al perfil de Front Door desde el vértice de la zona.

Este problema se resuelve mediante los registros de alias en Azure DNS. A diferencia de los registros CNAME, los registros de alias se crean en el vértice de la zona y los propietarios de aplicaciones pueden usarlos para apuntar su registro de vértice de la zona a un perfil de Front Door que tiene puntos de conexión públicos. Los propietarios de aplicaciones pueden apuntar al mismo perfil de Front Door que se use para cualquier otro dominio dentro de su zona DNS. Por ejemplo, `contoso.com` y `www.contoso.com` pueden apuntar al mismo perfil de Front Door. 

Para asignar el dominio raíz o de vértice al perfil de Front Door, básicamente se requiere la eliminación del formato de CNAME o la búsqueda de DNS, que son un mecanismo mediante el cual el proveedor de DNS resuelve de forma recursiva la entrada de CNAME hasta llegar a una dirección IP. Esta funcionalidad es compatible con Azure DNS para puntos de conexión de Front Door. 

> [!NOTE]
> También hay otros proveedores de DNS que admiten la eliminación del formato de CNAME o la búsqueda de DNS, pero Azure Front Door recomienda a sus clientes el uso de Azure DNS para hospedar sus dominios.

Puede usar Azure Portal para incorporar un dominio de vértice en Front Door y habilitar HTTPS si lo asocia con un certificado para la terminación TSL. Los dominios de vértice también se conocen como dominios raíz o "desnudos".

En este artículo aprenderá a:

> [!div class="checklist"]
> * Crear un registro de alias que apunte al perfil de Front Door
> * Agregar el dominio raíz a Front Door
> * Configurar HTTPS en el dominio raíz

> [!NOTE]
> Para este tutorial, es necesario que ya haya creado un perfil de Front Door. Consulte otros tutoriales como [Inicio rápido: Creación de una instancia de Front Door](./quickstart-create-front-door.md) o [Creación de una instancia de Front Door con redireccionamiento de HTTP a HTTPS](./front-door-how-to-redirect-https.md) para empezar a trabajar.

## <a name="create-an-alias-record-for-zone-apex"></a>Creación de un registro de alias para el vértice de la zona

1. Abra la configuración de **Azure DNS** del dominio que se va a incorporar.
2. Cree o edite el registro del vértice de la zona.
3. Seleccione el **Tipo** de registro _A_ y haga clic en _Sí_ para el **Conjunto de registros del alias**. El **Tipo de alias** debe establecerse en _Recurso de Azure_.
4. Elija la suscripción de Azure donde se hospeda el perfil de Front Door y, después, seleccione el recurso de Front Door en la lista desplegable **Recurso de Azure**.
5. Haga clic en **Aceptar** para enviar los cambios.

    ![Registro de alias para el vértice de la zona](./media/front-door-apex-domain/front-door-apex-alias-record.png)

6. El paso anterior creará un registro de vértice de la zona que apunta al recurso de Front Door y también una asignación de registros CNAME "afdverify" (por ejemplo, `afdverify.contosonews.com`) a `afdverify.<name>.azurefd.net` que se usará para la incorporación del dominio en el perfil de Front Door.

## <a name="onboard-the-custom-domain-on-your-front-door"></a>Incorporación del dominio personalizado a la instancia de Front Door

1. En la pestaña del diseñador de Front Door, haga clic en el icono "+" en la sección Hosts de front-end para agregar un nuevo dominio personalizado.
2. Escriba el nombre de dominio raíz o de vértice en el campo Nombre de host personalizado, por ejemplo, `contosonews.com`.
3. Una vez validada la asignación de CNAME del dominio a la instancia de Front Door, haga clic en **Agregar** para agregar el dominio personalizado.
4. Haga clic en **Guardar** para enviar los cambios.

![Menú Dominio personalizado](./media/front-door-apex-domain/front-door-onboard-apex-domain.png)

## <a name="enable-https-on-your-custom-domain"></a>Habilitación de HTTPS en el dominio personalizado

1. Haga clic en el dominio personalizado que se ha agregado y, en la sección **Personalizar HTTPS de dominio**, cambie el estado a **Habilitado**.
2. En **Tipo de administración de certificados**, seleccione _Usar mi propio certificado_.

> [!WARNING]
> Actualmente no se admite el tipo de administración de certificados Front Door administrado para dominios raíz o de vértice. La única opción disponible para habilitar HTTPS en un dominio raíz o de vértice para Front Door consiste en usar su propio certificado TLS/SSL personalizado hospedado en Azure Key Vault.

3. Asegúrese de que ha configurado los permisos adecuados para que Front Door acceda a Key Vault, como se indica en la interfaz de usuario, antes de continuar con el paso siguiente.
4. Elija una **cuenta de Key Vault** desde su suscripción actual y, luego, seleccione el **Secreto** y la **Versión del secreto** adecuados para asignar el certificado correcto.
5. Haga clic en **Actualizar** para guardar la selección y, después, haga clic en **Guardar**.
6. Haga clic en **Actualizar** tras un par de minutos y, luego, haga clic en el dominio personalizado de nuevo para ver el progreso del aprovisionamiento de certificados. 

> [!WARNING]
> Asegúrese de que ha creado las reglas de enrutamiento adecuadas para su dominio de vértice o de que ha agregado el dominio a las reglas de enrutamiento existentes.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).