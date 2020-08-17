---
title: Características de las solicitudes de datos de clientes para dispositivos de Azure Digital Twins
titleSuffix: Azure Digital Twins
description: En este artículo se muestran los procesos para exportar y eliminar datos personales en Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 05/05/2020
ms.topic: conceptual
ms.service: digital-twins
services: digital-twins
ms.openlocfilehash: 31da6a6b5375571f53cd7b478e957cc350ef591c
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86522353"
---
# <a name="azure-digital-twins-customer-data-request-features"></a>Características de las solicitudes de datos de clientes para Azure Digital Twins

Azure Digital Twins es una plataforma de desarrollo para la creación de representaciones digitales seguras de un entorno empresarial. Las representaciones están controladas por datos de estado activos de orígenes de datos seleccionados por los usuarios.

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

Las representaciones digitales llamadas *gemelos digitales* en Azure Digital Twins representan entidades en entornos reales y se asocian con identificadores. Microsoft no retiene ninguna información y no tiene acceso a datos que permitirían que los identificadores estuvieran correlacionados con los usuarios. 

Muchos de los gemelos digitales de Azure Digital Twins no representan directamente entidades personales: los objetos típicos que se representan pueden ser una sala de reuniones de oficina o una planta de fábrica. Sin embargo, los clientes pueden considerar que algunas entidades se pueden identificar a nivel personal y, a su entera discreción, pueden mantener sus propios métodos de seguimiento de activos o de inventario que vinculan gemelos digitales a personas. Azure Digital Twins administra y almacena todos los datos asociados con gemelos digitales como si fueran datos personales.

Para ver, exportar y eliminar datos personales a los que se puede hacer referencia en una solicitud del titular de los datos, un administrador de Azure Digital Twins puede usar [**Azure Portal**](https://portal.azure.com/) para usuarios y roles, o la [**API REST de Azure Digital Twins**](how-to-use-apis-sdks.md) para gemelos digitales. Las API de Azure Portal y REST proporcionan diferentes métodos para que los usuarios puedan atender dichas solicitudes del titular de los datos.

## <a name="identifying-customer-data"></a>Identificación de los datos del cliente

Azure Digital Twins considera que los *datos personales* son datos asociados a sus administradores y usuarios. 

Azure Digital Twins almacena el [id. de objeto](../active-directory/fundamentals/active-directory-whatis.md) de *Azure Active Directory* de los usuarios con acceso al entorno. Azure Digital Twins en Azure Portal muestra las direcciones de correo electrónico del usuario, pero estas direcciones de correo electrónico no se almacenan en Azure Digital Twins. Se buscan dinámicamente en Azure Active Directory mediante el id. de objeto de Azure Active Directory.

## <a name="deleting-customer-data"></a>Eliminación de datos del cliente

Los administradores de Azure Digital Twins pueden usar Azure Portal para eliminar los datos relacionados con los usuarios. También es posible realizar operaciones de eliminación en gemelos digitales individuales mediante las API REST de Azure Digital Twins. Para obtener más información sobre las API disponibles, consulte la [documentación de las API REST de Azure Digital Twins](https://docs.microsoft.com/rest/api/azure-digitaltwins/).

## <a name="exporting-customer-data"></a>Exportación de datos del cliente

Azure Digital Twins almacena los datos relacionados con gemelos digitales. Los usuarios pueden recuperar y ver estos datos a través de las API REST y exportar estos datos mediante la funcionalidad de copiar y pegar. 

Los datos del cliente, incluidos los roles de usuario y las asignaciones de roles, se pueden seleccionar, copiar y pegar desde Azure Portal. 

## <a name="links-to-additional-documentation"></a>Vínculos a documentación adicional

Para obtener una lista completa de las API del servicio Azure Digital Twins, consulte la [documentación de las API REST de Azure Digital Twins](https://docs.microsoft.com/rest/api/azure-digitaltwins/).