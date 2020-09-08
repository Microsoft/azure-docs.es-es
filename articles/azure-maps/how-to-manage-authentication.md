---
title: Administración de la autenticación
titleSuffix: Azure Maps
description: Familiarícese con la autenticación de Azure Maps. Vea qué enfoque funciona mejor en cada escenario. Aprenda a usar el portal para ver la configuración de autenticación.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 8fc27ea4de7e81f1279f68d2e4a1a7fde1fcf41f
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037344"
---
# <a name="manage-authentication-in-azure-maps"></a>Administración de la autenticación en Azure Maps

Después de crear una cuenta de Azure Maps, se crean las claves y el identificador de cliente para admitir la autenticación de clave compartida y la autenticación de Azure Active Directory (Azure AD).

## <a name="view-authentication-details"></a>Visualización de los detalles de la autenticación

Después de crear la cuenta de Azure Maps, se generan las claves principal y secundaria. Es recomendable que use la clave principal como clave de suscripción cuando [llame a Azure Maps mediante la autenticación de clave compartida](https://docs.microsoft.com/azure/azure-maps/azure-maps-authentication#shared-key-authentication). Puede usar la clave secundaria, por ejemplo, para realizar cambios de clave graduales. Para más información, consulte [Autenticación con Azure Maps](https://aka.ms/amauth).

Puede ver los datos de autenticación en Azure Portal. Una vez allí, vaya a su cuenta y seleccione **Autenticación** en el menú **Configuración**.

> [!div class="mx-imgBorder"]
> ![Detalles de la autenticación](./media/how-to-manage-authentication/how-to-view-auth.png)

## <a name="discover-category-and-scenario"></a>Detección de categoría y escenario

Dependiendo de las necesidades de la aplicación, hay rutas específicas para protegerla. Azure AD define categorías para admitir una amplia gama de flujos de autenticación. Consulte las [categorías de aplicación](https://docs.microsoft.com/azure/active-directory/develop/authentication-flows-app-scenarios#application-categories) para saber qué categoría se adapta a la aplicación.

> [!NOTE]
> Incluso si usa la autenticación de clave compartida, la descripción de las categorías y los escenarios le ayudará a proteger la aplicación.

## <a name="determine-authentication-and-authorization"></a>Definición de la autenticación y autorización

En la tabla siguiente se describen los escenarios comunes de autenticación y autorización en Azure Maps. La tabla proporciona una comparación de los tipos de protección que ofrece cada escenario.

> [!IMPORTANT]
> Microsoft recomienda implementar Azure Active Directory (Azure AD) con el control de acceso basado en rol (RBAC) para las aplicaciones de producción.

| Escenario                                                                                    | Authentication | Authorization | Esfuerzo para desarrollo | Esfuerzo operativo |
| ------------------------------------------------------------------------------------------- | -------------- | ------------- | ------------------ | ------------------ |
| [Demonio de confianza o aplicación cliente no interactiva](./how-to-secure-daemon-app.md)        | Clave compartida     | N/D           | Media             | Alto               |
| [Demonio de confianza o aplicación cliente no interactiva](./how-to-secure-daemon-app.md)        | Azure AD       | Alto          | Bajo                | Media             |
| [Aplicación de una sola página web con inicio de sesión único interactivo](./how-to-secure-spa-users.md) | Azure AD       | Alto          | Media             | Media             |
| [Aplicación de una sola página web con inicio de sesión no interactivo](./how-to-secure-spa-app.md)      | Azure AD       | Alto          | Media             | Media             |
| [Aplicación web con inicio de sesión único interactivo](./how-to-secure-webapp-users.md)          | Azure AD       | Alto          | Alto               | Media             |
| [Dispositivo IoT/dispositivo restringido de entrada](./how-to-secure-device-code.md)                     | Azure AD       | Alto          | Media             | Media             |

Los vínculos de la tabla muestran información de configuración detallada para cada escenario.

## <a name="view-role-definitions"></a>Visualización de definiciones de roles

Para ver los roles de Azure que están disponibles en Azure Maps, vaya al **Control de acceso (IAM)** . Seleccione **Roles** y busque los roles que comienzan con *Azure Maps*. Esos son los roles de Azure Maps a los que puede conceder acceso.

> [!div class="mx-imgBorder"]
> ![Visualización de roles disponibles](./media/how-to-manage-authentication/how-to-view-avail-roles.png)

## <a name="view-role-assignments"></a>Visualización de asignaciones de roles

Para ver los usuarios y las aplicaciones a los que se han concedido roles RBAC para Azure Maps, vaya a **Control de acceso (IAM)** . Una vez allí, seleccione **Asignaciones de rol** y filtre los resultados por **Azure Maps**.

> [!div class="mx-imgBorder"]
> ![Visualización de los usuarios y aplicaciones a los que se les han concedido roles RBAC](./media/how-to-manage-authentication/how-to-view-amrbac.png)

## <a name="request-tokens-for-azure-maps"></a>Solicitud de tokens de Azure Maps

Solicite un token del punto de conexión de token de Azure AD. Use los siguientes datos en la solicitud de Azure AD:

| Entorno de Azure      | Punto de conexión del token de Azure AD             | Identificador de recurso de Azure              |
| ---------------------- | ----------------------------------- | ------------------------------ |
| Nube pública de Azure     | `https://login.microsoftonline.com` | `https://atlas.microsoft.com/` |
| Nube de Azure Government | `https://login.microsoftonline.us`  | `https://atlas.microsoft.com/` |

Para obtener más información sobre la solicitud de tokens de acceso de Azure AD para usuarios y entidades de servicio, consulte [Escenarios de autenticación de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/authentication-scenarios) y consulte los escenarios específicos en la tabla [Escenarios](./how-to-manage-authentication.md#determine-authentication-and-authorization).

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte el artículo sobre el [SDK web de Azure AD y Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-use-map-control).

Descubra las métricas de uso de API de la cuenta de Azure Maps:
> [!div class="nextstepaction"]
> [Visualización de métricas de uso](how-to-view-api-usage.md)

Explore ejemplos acerca de cómo integrar Azure AD con Azure Maps:

> [!div class="nextstepaction"]
> [Muestras de autenticación de Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples)
