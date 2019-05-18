---
title: Uso de Graph API de Azure AD
description: Graph API de Azure Active Directory (Azure AD) proporciona acceso mediante programación a Azure AD a través de los puntos de conexión de la API REST OData. Las aplicaciones pueden usar Graph API de Azure AD para ejecutar operaciones de creación, lectura, actualización y eliminación (CRUD) en objetos y datos de directorio.
services: active-directory
documentationcenter: n/a
author: rwike77
manager: CelesteDG
editor: ''
tags: ''
ms.assetid: 9dc268a9-32e8-402c-a43f-02b183c295c5
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: sureshja
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 05b3d749eb3e2bc767889831b4c1ff01833a3f39
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544676"
---
# <a name="how-to-use-the-azure-ad-graph-api"></a>Procedimientos para: Usar Graph API de Azure AD

Graph API de Azure Active Directory (Azure AD) proporciona acceso mediante programación a Azure AD a través de los puntos de conexión de la API REST OData. Las aplicaciones pueden usar Graph API de Azure AD para ejecutar operaciones de creación, lectura, actualización y eliminación (CRUD) en objetos y datos de directorio. Por ejemplo, Graph API de Azure AD se puede usar para crear un nuevo usuario, ver o actualizar las propiedades de un usuario, cambiar la contraseña de un usuario, comprobar la pertenencia al grupo para el acceso basado en roles y deshabilitar o eliminar el usuario. Para más información sobre los escenarios de aplicaciones y características de Graph API de Azure AD, consulte [Graph API de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog) y los [requisitos previos de Graph API de Azure AD](https://msdn.microsoft.com/library/hh974476.aspx).

Este artículo se aplica a Azure AD Graph API. Para obtener información similar relacionada con Microsoft Graph API, consulte [Usar la API de Microsoft Graph](https://developer.microsoft.com/graph/docs/concepts/use_the_api).

> [!IMPORTANT]
> Se recomienda encarecidamente que utilice [Microsoft Graph](https://developer.microsoft.com/graph) en lugar de Graph API de Azure AD para tener acceso a recursos de Azure Active Directory. Nuestros esfuerzos de desarrollo ahora se centran en Microsoft Graph y no están previstas realizar mejoras adicionales para Graph API de Azure AD. Hay un número muy limitado de escenarios para los que Azure AD Graph API todavía podría ser adecuado. Para obtener más información, consulte la entrada de blog [Microsoft Graph o Azure AD Graph](https://dev.office.com/blogs/microsoft-graph-or-azure-ad-graph) del centro de desarrollo de Office.

## <a name="how-to-construct-a-graph-api-url"></a>Construcción de una dirección URL de Graph API

En Graph API, para tener acceso a los datos y objetos de los directorios (en otras palabras, a los recursos o entidades) con los que desee realizar operaciones CRUD, puede usar direcciones URL basadas en el protocolo Open Data (OData) Protocol. Las direcciones URL que se usan en Graph API constan de cuatro partes principales: raíz del servicio, identificador de inquilino, ruta de acceso a recursos y opciones de cadena de consulta: `https://graph.windows.net/{tenant-identifier}/{resource-path}?[query-parameters]`. Tome como ejemplo la siguiente dirección URL: `https://graph.windows.net/contoso.com/groups?api-version=1.6`.

* **Raíz del servicio**: en Graph API de Azure AD, la raíz del servicio es siempre https://graph.windows.net.
* **Identificador de inquilino**: esta sección puede ser un nombre de dominio (registrado) comprobado; en el ejemplo anterior, contoso.com. También puede ser un identificador de objeto de inquilino o los alias "myorganization" o "me". Para más información, consulte [Información general de las operaciones | Conceptos de Graph API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-operations-overview).
* **Ruta de acceso del recurso**: esta sección de una dirección URL identifica el recurso con el que se va a interactuar (usuarios, grupos, un usuario concreto o un grupo determinado, etc.) En el ejemplo anterior, son los "grupos" de nivel superior a los que se dirige este conjunto de recursos. También se puede dirigir una entidad concreta, como por ejemplo, “users/{objectId}” o “users/userPrincipalName”.
* **Parámetros de consulta**: un signo de interrogación (?) separa la sección de ruta de acceso a recursos de la sección de parámetros de consulta. Todas las solicitudes de Graph API de Azure AD requieren el parámetro de consulta "api-version". Graph API de Azure AD también admite las siguientes opciones de consulta de OData: **$filter**, **$orderby**, **$expand**, **$top** y **$format**. Sin embargo, las siguientes opciones de consulta no están admitidas actualmente: **$count**, **$inlinecount** y **$skip**. Para obtener más información, consulte [Consultas admitidas, filtros y opciones de paginación en Graph API de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-supported-queries-filters-and-paging-options).

## <a name="graph-api-versions"></a>Versiones de Graph API

La versión de las solicitudes de Graph API se especifica en el parámetro de consulta "api-version". En el caso de la versión 1.5 y posterior, use un valor de versión numérico; api-version=1.6. En el caso de versiones anteriores, use una cadena de fecha que se ajuste al formato AAAA-MM-DD; por ejemplo, api-version=2013-11-08. Para las características de vista previa, use la cadena "beta"; por ejemplo, api-version = beta. Para más información sobre las diferencias entre las distintas versiones de Graph API, consulte [Control de versiones de Graph API de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-versioning).

## <a name="graph-api-metadata"></a>Metadatos de Graph API

Para devolver el archivo de metadatos de Graph API de Azure AD, agregue el segmento "$metadata" después del identificador de inquilino en la dirección URL. Por ejemplo, la dirección URL siguiente devuelve los metadatos de la compañía de demostración: `https://graph.windows.net/GraphDir1.OnMicrosoft.com/$metadata?api-version=1.6`. Puede escribir esta dirección URL en la barra de direcciones de un explorador web para ver los metadatos. El documento de metadatos CSDL devuelto describe las entidades y los tipos complejos, sus propiedades, y las funciones y acciones expuestas por la versión de Graph API que ha solicitado. Si se omite el parámetro api-version se devolverán los metadatos de la versión más reciente.

## <a name="common-queries"></a>Consultas comunes

[Consultas comunes de Graph API de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-supported-queries-filters-and-paging-options#CommonQueries): se enumeran las consultas comunes que pueden usarse con Azure AD Graph, incluidas las consultas que pueden usarse para obtener acceso a recursos de nivel superior del directorio y las consultas para realizar operaciones en el directorio.

Por ejemplo, `https://graph.windows.net/contoso.com/tenantDetails?api-version=1.6` devuelve la información de la compañía en el directorio contoso.com.

O, `https://graph.windows.net/contoso.com/users?api-version=1.6` enumera todos los objetos de usuario del directorio de contoso.com.

## <a name="using-the-azure-ad-graph-explorer"></a>Uso del Probador de Graph de Azure AD
El Probador de Graph de Azure AD se puede usar para que Graph API de Azure AD consulte los datos de directorio al compilar la aplicación.

En la captura de pantalla siguiente se muestra el resultado que vería si fuera al Probador de Graph de Azure AD, iniciara y escribiera `https://graph.windows.net/GraphDir1.OnMicrosoft.com/users?api-version=1.6` para mostrar todos los usuarios del directorio del usuario que ha iniciado sesión:

![Azure AD graph api explorador](./media/active-directory-graph-api-quickstart/graph_explorer.png)

**Carga de Azure AD Graph Explorer**: para cargar la herramienta, vaya a [https://graphexplorer.azurewebsites.net/ ](https://graphexplorer.azurewebsites.net/). Haga clic en **Iniciar sesión** e inicie sesión con las credenciales de su cuenta de Azure AD para ejecutar el Probador de Graph de Azure AD con el inquilino. Si ejecuta el Probador de Graph de Azure AD con su propio inquilino, usted o el administrador tendrán que dar su consentimiento durante el inicio de sesión. Si dispone de una suscripción a Office 365, tendrá automáticamente un inquilino de Azure AD. De hecho, las credenciales que usa para iniciar sesión en Office 365 son cuentas de Azure AD y puede usarlas con el Probador de Graph de Azure AD.

**Ejecutar una consulta**: para ejecutar una consulta, escríbala en el cuadro de texto de la solicitud y haga clic en **GET** o en la tecla **Entrar**. Los resultados se muestran en el cuadro de respuesta. Por ejemplo, `https://graph.windows.net/myorganization/groups?api-version=1.6` enumera todos los objetos de grupo del directorio del usuario que ha iniciado sesión.

Tenga en cuenta las siguientes características y limitaciones del Probador de Graph de Azure AD:

* Funcionalidad Autocompletar en conjuntos de recursos. Para ver esta funcionalidad, haga clic en el cuadro de texto de la solicitud (donde aparece la dirección URL de la compañía). Puede seleccionar un conjunto de recursos en la lista desplegable.
* Historial de solicitudes.
* Admite los alias de direccionamiento “me” y “myorganization”. Por ejemplo, puede usar `https://graph.windows.net/me?api-version=1.6` para devolver el objeto de usuario del usuario con sesión iniciada o `https://graph.windows.net/myorganization/users?api-version=1.6` para devolver todos los usuarios del directorio del usuario con sesión iniciada.
* Admite operaciones de creación, lectura, actualización y eliminación (CRUD) completas en su propio directorio con `POST`, `GET`, `PATCH` y `DELETE`.
* Una sección de encabezados de respuesta. Esta sección se puede usar como ayuda para solucionar los problemas que se producen al ejecutar consultas.
* Un visor JSON para la respuesta con capacidades de expansión y contracción.
* No permite mostrar o cargar fotos en miniatura.

## <a name="using-fiddler-to-write-to-the-directory"></a>Uso de Fiddler para escribir en el directorio

Para seguir esta guía de inicio rápido, puede usar el depurador web Fiddler para practicar la realización de operaciones de "escritura" con su directorio de Azure AD. Por ejemplo, puede obtener y cargar la foto del perfil de un usuario (lo cual no es posible con el Probador de Graph de Azure AD). Para obtener más información y para instalar Fiddler, consulte [https://www.telerik.com/fiddler](https://www.telerik.com/fiddler).

En el ejemplo siguiente, usará el depurador web Fiddler para crear un nuevo grupo de seguridad, "MyTestGroup", en el directorio de Azure AD.

**Obtener un token de acceso**: para obtener acceso a Azure AD Graph, los clientes deben autenticarse primero correctamente en Azure AD. Para obtener más información, consulte [Escenarios de autenticación en Azure AD](authentication-scenarios.md).

**Redactar y ejecutar una consulta**: Complete los siguientes pasos:

1. Abra el depurador web Fiddler y cambie a la pestaña **Composer** (Compositor).
2. Puesto que desea crear un nuevo grupo de seguridad, seleccione **Post** como método de HTTP en el menú desplegable. Para más información sobre las operaciones y los permisos de los objetos de grupo, consulte la sección sobre los [grupos](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#group-entity) en la [referencia sobre la API de REST Graph de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog).
3. En el campo que se encuentra junto a **Post**, escriba la siguiente dirección URL de solicitud: `https://graph.windows.net/{mytenantdomain}/groups?api-version=1.6`.
   
   > [!NOTE]
   > Debe sustituir {mytenantdomain} por el nombre de dominio de su directorio de Azure AD.

4. En el campo que se encuentra inmediatamente debajo de Post, escriba el siguiente encabezado HTTP:
   
    ```
   Host: graph.windows.net
   Authorization: Bearer <your access token>
   Content-Type: application/json
   ```
   
   > [!NOTE]
   > Sustituya &lt;su token de acceso&gt; por el del directorio de Azure AD.

5. En el campo **Cuerpo de la solicitud** escriba el siguiente JSON:
   
    ```
        {
            "displayName":"MyTestGroup",
            "mailNickname":"MyTestGroup",
            "mailEnabled":"false",
            "securityEnabled": true
        }
   ```
   
    Para más información sobre la creación de grupos, consulte [Crear un grupo](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/groups-operations#CreateGroup).

Para más información sobre las entidades y los tipos de Azure AD expuestos por Graph, así como sobre las operaciones que se pueden realizar en ellos con Graph, consulte [Referencia de la API Graph de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de [Graph API de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)
* Más información sobre [los ámbitos de permiso de Graph API de Azure AD](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)
