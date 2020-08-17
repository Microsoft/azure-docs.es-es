---
title: Administración de recursos de datos en Azure Data Catalog
description: En este artículo se resalta cómo controlar la visibilidad y la propiedad de los recursos de datos registrados en Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 9d035417b613c81487c493bde90cdcc461f04412
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081158"
---
# <a name="manage-data-assets-in-azure-data-catalog"></a>Administración de recursos de datos en Azure Data Catalog
## <a name="introduction"></a>Introducción
Azure Data Catalog está diseñado para la detección de orígenes de datos, lo que le permite detectar y comprender fácilmente los orígenes de datos que necesitan para realizar análisis y tomar decisiones. Estas funcionalidades de detección consiguen el mayor impacto cuando todos los usuarios puedan buscar y comprender la gama más amplia de orígenes de datos disponibles. Con estos elementos en mente, el comportamiento predeterminado de Data Catalog es que todos los orígenes de datos registrados sean visibles, y detectables, para todos los usuarios del catálogo.

Data Catalog no ofrece a los usuarios acceso a los mismos datos. El propietario del origen de datos es quien controla el acceso a los datos. Con Data Catalog se pueden detectar los orígenes de datos y ver los metadatos relacionados con los orígenes registrados en el catálogo.

Sin embargo, puede haber situaciones en las que los orígenes de datos solo deben ser visibles a usuarios específicos o a los miembros de grupos específicos. En estos escenarios, los usuarios pueden tomar posesión de los recursos de datos registrados en el catálogo y, después, controlar la visibilidad de los recursos que poseen.

> [!NOTE]
> La funcionalidad descrita en este artículo solo está disponible en la edición Estándar de Azure Data Catalog. La edición Gratis no ofrece funcionalidades para poseer y restringir la visibilidad de los recursos de datos.
>
>

## <a name="manage-ownership-of-data-assets"></a>Administración de la propiedad de los recursos de datos
De forma predeterminada, los activos de datos que están registrados en Data Catalog no tienen propietario. Cualquier usuario con permiso para acceder al catálogo puede detectar y anotar estos recursos. Los usuarios pueden tomar posesión de los recursos de datos sin propietario y limitar la visibilidad de los recursos que poseen.

Cuando un recurso de datos de Data Catalog tiene propietario, solo los usuarios autorizados por los propietarios pueden detectar el recurso y ver sus metadatos, y solo los propietarios pueden eliminar el recurso del catálogo.

> [!NOTE]
> La propiedad de Data Catalog solo afecta a los metadatos almacenados en él. La propiedad no concede ningún permiso en el origen de datos subyacente.
>
>

### <a name="take-ownership"></a>Toma de posesión
Los usuarios pueden tomar posesión de los recursos de datos mediante la selección de la opción **Tomar posesión** en Azure Data Catalog. No se necesitan permisos especiales para tomar posesión de un recurso de datos sin propietario. Cualquier usuario puede tomar posesión de un recurso de datos sin propietario.

### <a name="add-owners-and-co-owners"></a>Adición de propietarios y copropietarios
Si un recurso de datos ya tiene propietario, los demás usuarios simplemente no podrán tomar posesión del mismo. Un propietario ya existente debe agregarles como copropietarios. Cualquier propietario puede agregar más usuarios o grupos de seguridad como copropietarios.

> [!NOTE]
> Es aconsejable tener al menos dos personas como propietarios de cualquier recurso de datos con propietario.
>
>

### <a name="remove-owners"></a>Eliminación de propietarios
Al igual que un propietario de recursos puede agregar copropietarios, cualquier propietario de recursos puede quitar a un copropietario.

Un propietario de recursos que se quita a sí mismo como propietario ya no podrá administrar el recurso. Si el propietario de recursos se quita a sí mismo como propietario y no hay ningún otro copropietario, el recurso se revertirá a un estado de sin propietario.

## <a name="control-visibility"></a>Control de la visibilidad
Los propietarios de recursos de datos pueden controlar la visibilidad de los recursos de datos que poseen. Para restringir la visibilidad del valor predeterminado, según el cual todos los usuarios de Data Catalog pueden detectar y ver el recurso de datos, el propietario del recurso puede cambiar la configuración de visibilidad de **Todos** a **Propietarios y estos usuarios** en las propiedades del recurso. Los propietarios pueden después agregar usuarios y grupos de seguridad específicos.

> [!NOTE]
> Siempre que sea posible, los permisos de propiedad y la visibilidad de los recursos deben asignarse a grupos de seguridad y no a usuarios individuales.
>
>

## <a name="catalog-administrators"></a>Administradores del Catálogo
Los administradores de Data Catalog son implícitamente copropietarios de todos los recursos del catálogo. Los propietarios de los recursos no pueden quitar la visibilidad a los administradores y estos pueden administrar la propiedad y la visibilidad de todos los recursos de datos del catálogo.

## <a name="summary"></a>Resumen
El modelo de micromecenazgo de Data Catalog para la detección de recursos de datos y metadatos permite que todos los usuarios del catálogo colaboren en él y lo detecten. La edición Estándar de Data Catalog está diseñada para la propiedad y administración con el fin de limitar la visibilidad y el uso de los recursos de datos específicos.
