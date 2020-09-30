---
title: 'Inicio rápido: Registro y exposición de una API web | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido se registra una API web con la plataforma de identidad de Microsoft y se configuran sus ámbitos mediante la exposición a los clientes para el acceso basado en permisos a los recursos de la API.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/03/2020
ms.author: marsma
ms.custom: aaddev, contperfq1
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 72d66bd4c738ed60bbaefc123daae90ecc0db163
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442170"
---
# <a name="quickstart-configure-an-application-to-expose-a-web-api"></a>Inicio rápido: Configuración de una aplicación para exponer una API web

En este inicio rápido se registra una API web en la plataforma de identidad de Microsoft y se agrega un ámbito de ejemplo para exponerla a las aplicaciones cliente. Al registrar la API web y exponerla mediante ámbitos, puede proporcionar acceso basado en permisos a sus recursos a los usuarios autorizados y a las aplicaciones cliente que accedan a la API.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Finalización de [Inicio rápido: Configuración de un inquilino](quickstart-create-new-tenant.md).

## <a name="register-the-web-api"></a>Registro de la API web

Para proporcionar acceso con ámbito a los recursos de la API web, primero debe registrar la API en la plataforma de identidad de Microsoft.

1. Siga los pasos de la sección **Registro de una aplicación** en [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](quickstart-register-app.md).
1. Omita las secciones **Incorporación de un URI de redireccionamiento** y **Configuración de la plataforma**. No es necesario configurar un URI de redireccionamiento para las API web, ya que ningún usuario inicia sesión de forma interactiva.
1. Omita la sección **Incorporación de las credenciales** por ahora. La API necesitaría sus propias credenciales solo si accede a otra de nivel inferior, pero este escenario no se trata en este artículo.

Una vez registrada la API web, está listo para agregar los ámbitos que el código de la API puede usar para proporcionar permisos pormenorizados a los consumidores de la API.

## <a name="add-a-scope"></a>Agregar un ámbito

El código de las aplicaciones cliente solicita permiso para realizar operaciones definidas por la API web; para ello, pasa un token de acceso junto con sus solicitudes al recurso protegido (la API web). A continuación, la API web realiza la operación solicitada solo si el token de acceso que recibe contiene los ámbitos necesarios para la operación.

En primer lugar, siga estos pasos para crear un ámbito de ejemplo denominado `Employees.Read.All`:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/quickstart-configure-app-expose-web-apis/portal-01-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino que contiene el registro de la aplicación cliente.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones** y el registro de la aplicación de API.
1. Seleccione **Exponer una API** > **Agregar un ámbito**.

    :::image type="content" source="media/quickstart-configure-app-expose-web-apis/portal-02-expose-api.png" alt-text="Panel de exposición de una API durante el registro de una aplicación en Azure Portal":::

1. Se le pedirá que establezca un valor de **URI de id. de aplicación** si aún no ha configurado uno.

   El URI de identificación de aplicación actúa como prefijo de los ámbitos a los que se hará referencia en el código de la API y debe ser único globalmente. Puede usar el valor predeterminado proporcionado, que tiene el formato `api://<application-client-id>`, o especificar uno más legible, como `https://contoso.com/api`.

1. A continuación, especifique los atributos del ámbito en el panel **Agregar un ámbito**. Para este tutorial, puede usar los valores de ejemplo o especificar los suyos propios.

    | Campo | Descripción | Ejemplo |
    |-------|-------------|---------|
    | **Nombre de ámbito** | Nombre del ámbito. Una convención de nomenclatura típica es `resource.operation.constraint`. | `Employees.Read.All` |
    | **¿Quién puede dar el consentimiento?** | Si los usuarios pueden dar su consentimiento a este ámbito o si se requiere el consentimiento del administrador. Seleccione **Solo administradores** para permisos con más privilegios. | **Administradores y usuarios** |
    | **Nombre para mostrar del consentimiento del administrador** | Breve descripción del propósito del ámbito que solo verán los administradores. | `Read-only access to Employee records` |
    | **Descripción del consentimiento del administrador** | Descripción más detallada del permiso concedido por el ámbito que solo verán los administradores. | `Allow the application to have read-only access to all Employee data.` |
    | **Nombre para mostrar del consentimiento del usuario** | Descripción breve del propósito del ámbito. Se muestra a los usuarios solo si establece **Quién puede dar el consentimiento** en **Administradores y usuarios**. | `Read-only access to your Employee records` |
    | **Descripción del consentimiento del usuario** | Descripción más detallada del permiso concedido por el ámbito. Se muestra a los usuarios solo si establece **Quién puede dar el consentimiento** en **Administradores y usuarios**. | `Allow the application to have read-only access to your Employee data.` |

1. Establezca **Estado** en **Habilitado** y seleccione **Agregar ámbito**.

1. (Opcional) Para eliminar la solicitud de consentimiento por parte de los usuarios de su aplicación en los ámbitos que ha definido, puede *preautorizar* a la aplicación cliente para que acceda a su API web. Debe preautorizar *únicamente* a las aplicaciones cliente en las que confíe, ya que los usuarios no tendrán la oportunidad de rechazar el consentimiento.
    1. En **Aplicaciones cliente autorizadas**, seleccione **Agregar una aplicación cliente**
    1. Escriba el valor de **Id. de aplicación (cliente)** de la aplicación cliente que desea preautorizar. Por ejemplo, el de una aplicación web que ha registrado previamente.
    1. En **Ámbitos autorizados**, seleccione los ámbitos en que desea suprimir la solicitud de consentimiento y, después, seleccione **Agregar aplicación**.

    Si ha seguido este paso opcional, la aplicación cliente será ya una aplicación cliente preautorizada (PCA) y no se pedirá a los usuarios su consentimiento al iniciar sesión en ella.

## <a name="add-a-scope-requiring-admin-consent"></a>Incorporación de un ámbito que requiera el consentimiento del administrador

A continuación, agregue otro ámbito de ejemplo denominado `Employees.Write.All` para el que solo los administradores puedan dar su consentimiento. Los ámbitos que requieren el consentimiento del administrador se suelen usar para proporcionar acceso a operaciones de mayor privilegio y, a menudo, los usan aplicaciones cliente que se ejecutan como servicios back-end o demonios que no inician sesión de forma interactiva.

Para agregar el ámbito de ejemplo `Employees.Write.All`, siga los pasos de la sección [Incorporación de un ámbito](#add-a-scope) y especifique estos valores en el panel **Agregar un ámbito**:

| Campo                          | Valor de ejemplo                                                      |
|--------------------------------|--------------------------------------------------------------------|
| **Nombre de ámbito**                 | `Employees.Write.All`                                              |
| **¿Quién puede dar el consentimiento?**            | **Solo administradores**                                                    |
| **Nombre para mostrar del consentimiento del administrador** | `Write access to Employee records`                                 |
| **Descripción del consentimiento del administrador**  | `Allow the application to have write access to all Employee data.` |
| **Nombre para mostrar del consentimiento del usuario**  | *Ninguno (deje este campo en blanco)*                                               |
| **Descripción del consentimiento del usuario**   | *Ninguno (deje este campo en blanco)*                                               |

## <a name="verify-the-exposed-scopes"></a>Verificación de los ámbitos expuestos

Si ha agregado correctamente los dos ámbitos de ejemplo descritos en las secciones anteriores, aparecerán en el panel **Exponer una API** del registro de la aplicación de API web, de forma similar a esta imagen:

:::image type="content" source="media/quickstart-configure-app-expose-web-apis/portal-03-scopes-list.png" alt-text="Captura de pantalla del panel Exponer una API que muestra dos ámbitos expuestos.":::

Como se muestra en la imagen, la cadena completa de un ámbito es la concatenación de los valores de **URI de id. de aplicación** de la API web y **Nombre de ámbito** del ámbito.

Por ejemplo, si el identificador URI de aplicación de la API web es `https://contoso.com/api` y el nombre del ámbito es `Employees.Read.All`, el ámbito completo es:

`https://contoso.com/api/Employees.Read.All`

## <a name="using-the-exposed-scopes"></a>Uso de los ámbitos expuestos

En el siguiente artículo de la serie configurará el registro de una aplicación cliente con acceso a su API web y los ámbitos que haya definido mediante los pasos de este artículo.

Una vez que el registro de una aplicación cliente tenga permiso para acceder a la API web, se puede emitir un token de acceso de OAuth 2.0 para el cliente mediante la plataforma de identidad de Microsoft. Cuando el cliente llama a la API web, presenta un token de acceso cuya demanda de ámbito (`scp`) se establece en los permisos especificados en el registro de la aplicación cliente.

Puede exponer ámbitos adicionales posteriormente si es necesario. Tenga en cuenta que la API web puede exponer varios ámbitos asociados a varias operaciones. El recurso puede controlar el acceso a la API web en tiempo de ejecución, mediante la evaluación de las notificaciones de ámbito (`scp`) del token de acceso de OAuth 2.0 recibido.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha expuesto su API web mediante la configuración de sus ámbitos, configure el permiso para acceder a esos ámbitos en el registro de la aplicación cliente.

> [!div class="nextstepaction"]
> [Configuración de una aplicación para acceder a una API web](quickstart-configure-app-access-web-apis.md)

<!-- REF LINKS -->
[ms-graph-application]: /graph/api/resources/application
