---
title: Migración de un nombre de DNS activo
description: Obtenga información sobre cómo migrar un nombre de dominio DNS personalizado que ya esté asignado a un sitio activo a Azure App Service sin ningún tiempo de inactividad.
tags: top-support-issue
ms.assetid: 10da5b8a-1823-41a3-a2ff-a0717c2b5c2d
ms.topic: article
ms.date: 08/25/2020
ms.custom: seodec18
ms.openlocfilehash: e1b50675bef0f883ff617b3098a742d3491b3c13
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89484312"
---
# <a name="migrate-an-active-dns-name-to-azure-app-service"></a>Migración de un nombre de DNS activo a Azure App Service

En este artículo se explica cómo migrar un nombre DNS activo a [Azure App Service](../app-service/overview.md) sin experimentar ningún tiempo de inactividad.

Cuando se migra un sitio activo y su nombre de dominio DNS para App Service, ese nombre DNS ya está atendiendo a tráfico activo. Puede evitar tiempos de inactividad en la resolución DNS durante la migración enlazando el nombre DNS activo a la aplicación de App Service de forma preferente.

Si no le preocupa el tiempo de inactividad en la resolución DNS, vea [Asignar un nombre DNS personalizado a Azure App Service](app-service-web-tutorial-custom-domain.md).

## <a name="prerequisites"></a>Prerrequisitos

Para completar este procedimiento:

- [Asegúrese de que la aplicación de App Service no esté en el nivel GRATIS](app-service-web-tutorial-custom-domain.md#checkpricing).

## <a name="bind-the-domain-name-preemptively"></a>Enlace del nombre de dominio de forma preventiva

Al enlazar un dominio personalizado de forma preventiva, logrará lo siguiente antes de efectuar cualquier cambio en sus registros de DNS existentes:

- Comprobar la propiedad del dominio
- Habilitar el nombre de dominio para la aplicación

Cuando finalmente migre el nombre DNS personalizado del sitio antiguo a la aplicación de App Service, no habrá tiempo de espera en la resolución DNS.

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### <a name="get-domain-verification-id"></a>Obtención del identificador de comprobación de dominio

Para obtener el identificador de comprobación de dominio de la aplicación, siga los pasos descritos en [Obtención del identificador de comprobación de dominio](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id).

### <a name="create-domain-verification-record"></a>Creación de un registro de comprobación de dominio

Para comprobar la propiedad del dominio, agregue un registro TXT para la comprobación del dominio. El nombre de host del registro TXT depende del tipo de registro de DNS que desee asignar. Vea la tabla siguiente (`@` normalmente representa el dominio raíz):

| Ejemplo de registro DNS | Host TXT | Valor TXT |
| - | - | - |
| \@ (raíz) | _asuid_ | [Identificador de comprobación de dominio para la aplicación](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id) |
| www (sub) | _asuid.www_ | [Identificador de comprobación de dominio para la aplicación](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id) |
| \* (comodín) | _asuid_ | [Identificador de comprobación de dominio para la aplicación](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id) |

En la página de registros DNS, tenga en cuenta el tipo de registro del nombre DNS que desee migrar. App Service es compatible con las asignaciones de CNAME y registros A.

> [!NOTE]
> Los registros con el carácter comodín `*` no validarán los subdominios con un registro CNAME existente. Es posible que deba crear explícitamente un registro TXT para cada subdominio.

### <a name="enable-the-domain-for-your-app"></a>Habilitación del dominio para la aplicación

1. En [Azure Portal](https://portal.azure.com), en la navegación izquierda de la página de aplicaciones, seleccione **Dominios personalizados**. 

    ![Menú Dominio personalizado](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. En la página **Dominios personalizados**, seleccione **Agregar un dominio personalizado**.

    ![Agregar nombre de host](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Escriba el nombre de dominio completo que desea migrar, que corresponde al registro TXT que cree, como `contoso.com`, `www.contoso.com` o `*.contoso.com`. Seleccione **Validar**.

    El botón **Agregar dominio personalizado** está activado. 

1. Asegúrese de que el **tipo de registro de nombre de host** se establece en el tipo de registro DNS que desea migrar. Seleccione **Agregar nombre de host**.

    ![Agregar nombre DNS a la aplicación](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

    El nuevo nombre de host puede tardar un tiempo en reflejarse en la página **Dominios personalizados** de la aplicación. Intente actualizar el explorador para actualizar los datos.

    ![Registro CNAME agregado](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    El nombre DNS personalizado ya estará habilitado en la aplicación de Azure. 

## <a name="remap-the-active-dns-name"></a>Reasignación del nombre DNS activo

Lo único que queda por hacer es reasignar el registro DNS activo para que señale a App Service. En este momento todavía señala a su sitio antiguo.

<a name="info"></a>

### <a name="copy-the-apps-ip-address-a-record-only"></a>Copia de la dirección IP de la aplicación (solo registro A)

Si va a reasignar un registro CNAME, omita esta sección. 

Para reasignar un registro A, necesita la dirección IP externa de la aplicación de App Service, que se muestra en la página **Dominios personalizados**.

En la página **Dominios personalizados**, copie la dirección IP de la aplicación.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### <a name="update-the-dns-record"></a>Actualización del registro DNS

En la página de registros DNS del proveedor de dominios, seleccione el registro DNS que reasignar.

Para el ejemplo de dominio raíz de `contoso.com`, reasigne el registro A o CNAME como los ejemplos de la siguiente tabla: 

| Ejemplo de FQDN | Tipo de registro | Host | Value |
| - | - | - | - |
| contoso.com (raíz) | Un | `@` | D www Dirección IP de [Copiar la dirección IP de la aplicación](#info) |
| www\.contoso.com (sub) | CNAME | `www` | _&lt;nombreaplic&gt;.azurewebsites.net_ |
| \*.contoso.com (comodín) | CNAME | _\*_ | _&lt;nombreaplic&gt;.azurewebsites.net_ |

Guarde la configuración.

Las consultas DNS deben comenzar resolviéndose en la aplicación de App Service de inmediato después de que se produzca la propagación de DNS.

## <a name="migrate-domain-from-another-app"></a>Migración del dominio desde otra aplicación

Puede migrar un dominio personalizado activo en Azure, ya sea entre suscripciones o dentro de la misma suscripción. Esta migración sin tiempo de inactividad, sin embargo, requiere que en un momento determinado se asigne el mismo dominio personalizado a la aplicación de origen y la aplicación de destino. Por lo tanto, debe asegurarse de que las dos aplicaciones no se han implementado en la misma unidad de implementación (lo que se conoce internamente como espacio web). Un nombre de dominio solo se puede asignar a una aplicación de cada unidad de implementación.

Puede encontrar la unidad de implementación de la aplicación si examina el nombre de dominio de la dirección URL de FTP/S `<deployment-unit>.ftp.azurewebsites.windows.net`. Compruebe, asegurándose de ello, que la unidad de implementación es diferente entre la aplicación de origen y la aplicación de destino. La unidad de implementación de una aplicación viene determinada por el [plan de App Service](overview-hosting-plans.md) en el que se encuentra. Azure la selecciona aleatoriamente al crear el plan y no se puede cambiar. Azure solo se asegura de que dos planes estén en la misma unidad de implementación cuando se [crean en el mismo grupo de recursos *y* en la misma región](app-service-plan-manage.md#create-an-app-service-plan), pero no tiene ninguna lógica para asegurarse de que los planes se encuentran en unidades de implementación diferentes. La única forma de crear un plan en una unidad de implementación diferente es seguir creando planes en nuevos grupos de recursos o regiones hasta que se obtenga una unidad de implementación diferente.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo enlazar un certificado TLS/SSL a App Service.

> [!div class="nextstepaction"]
> [Protección de un nombre DNS personalizado con un enlace TLS en Azure App Service](configure-ssl-bindings.md)
