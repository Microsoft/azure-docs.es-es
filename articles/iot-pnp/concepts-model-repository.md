---
title: Descripción de los conceptos del repositorio de modelos de Azure IoT | Microsoft Docs
description: La finalidad de este artículo es que los profesionales de TI o los desarrolladores de soluciones conozcan los conceptos básicos del repositorio de modelos de Azure IoT.
author: prashmo
ms.author: prashmo
ms.date: 07/24/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 7d736721e2676a42da90aead3144f8016329f730
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475505"
---
# <a name="azure-iot-model-repository"></a>Repositorio de modelos de Azure IoT

El repositorio de modelos de Azure IoT permite a los creadores de dispositivos administrar y compartir los modelos de dispositivos IoT Plug and Play. Los modelos de dispositivo son documentos LD JSON definidos mediante el [lenguaje de creación de modelos de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). Los modelos almacenados en el servicio del repositorio de modelos se pueden compartir con los desarrolladores de soluciones de forma privada mediante el control de acceso, o públicamente, sin necesidad de autenticación para integrar y desarrollar la solución de nube de IoT Plug and Play.

El acceso al repositorio de modelos se puede realizar por estos medios:

- Portal del [repositorio de modelos de Azure IoT](https://aka.ms/iotmodelrepo)
- [API REST del repositorio de modelos de Azure IoT](https://docs.microsoft.com/rest/api/iothub/digitaltwinmodelrepositoryservice/getmodelasync/getmodelasync)
- [Comandos del repositorio de modelos de la CLI de Azure](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/pnp?view=azure-cli-latest)

## <a name="public-models"></a>Modelos públicos

Los modelos de gemelos digitales públicos almacenados en el repositorio de modelos están disponibles para que todos los usuarios los consuman e integren en su aplicación sin necesidad de autenticación. Además, los modelos públicos permiten un ecosistema abierto donde creadores de dispositivos y desarrolladores de soluciones puedan compartir y reutilizar sus modelos de dispositivo IoT Plug and Play.

Consulte la sección [Publicación de modelos](#publish-a-model) en **Modelos de empresa** para obtener instrucciones sobre cómo publicar un modelo en el repositorio de modelos para que sea público.

Para ver un modelo público mediante el portal del repositorio de modelos:

1. Vaya al [portal del repositorio de modelos de Azure IoT](https://aka.ms/iotmodelrepo).

1. Seleccione **View public models** (Ver modelos públicos).

    ![Ver modelos públicos](./media/concepts-model-repository/public-models.png)

Para ver un modelo público mediante programación con la API REST, consulte la documentación de la [API REST de obtención de modelos](https://docs.microsoft.com/rest/api/iothub/digitaltwinmodelrepositoryservice/getmodelasync/getmodelasync).

```csharp
var httpClient = new HttpClient();
httpClient.BaseAddress = new Uri("https://repo.azureiotrepository.com");

var modelId = "dtmi:com:mxchip:model;1";
var response = await httpClient.GetAsync($"/models/{modelId}?api-version=2020-05-01-preview").ConfigureAwait(false);
```

Para ver un modelo público mediante la CLI, consulte el comando [Obtener modelo](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/pnp/model?view=azure-cli-latest#ext-azure-iot-az-iot-pnp-model-show) de la CLI de Azure.

## <a name="company-models"></a>Modelos de empresa

El repositorio de modelos de empresa es un inquilino del repositorio de modelos de Azure IoT de una organización que permite crear y administrar modelos de gemelos digitales creados por los usuarios de una empresa u organización. Los modelos de empresa solo están disponibles para los usuarios autenticados de la empresa u organización. Un administrador de inquilinos del repositorio de modelos puede asignar permisos y controlar el acceso de otros usuarios de la empresa u organización a los modelos del repositorio de modelos de empresa.

### <a name="set-up-your-company-model-repository"></a>Configuración del repositorio de modelos de empresa

Use la *cuenta profesional o educativa de Azure Active Directory (Azure AD)* para acceder al repositorio de modelos. Si la organización ya tiene un inquilino de Azure AD, puede usar las cuentas de usuario y las entidades de servicio de dicho inquilino.

Para aprender a configurar un inquilino de Azure AD y crear un usuario o una entidad de servicio en él, consulte la sección [Información adicional](#additional-information).

- Al primer usuario de la organización que accede al repositorio de modelos o que inicia sesión en el portal se le concede el rol **Tenant Administrator** (Administrador de inquilinos). Este rol permite asignar roles a otros usuarios del inquilino del repositorio de la organización.

- Un **administrador de inquilinos** puede asignarle otros roles, como **Read Models** (Leer modelos) o **Create Models** (Crear modelos).

### <a name="understand-access-management"></a>Administración del acceso

En la tabla siguiente se resumen las funcionalidades admitidas en el repositorio de modelos de empresa y sus permisos asociados:

| Capacidad  | Permiso| Descripción|
|-------------|-----------|------------|
|Leer modelos|Leer modelos|De forma predeterminada, todos los usuarios del inquilino de la empresa pueden ver sus modelos de empresa. Además, el usuario también puede ver los modelos privados compartidos por otras empresas.|
|Administración del acceso|Administración del acceso|Administre la asignación de roles de usuario (agregar o quitar) para otros usuarios de la organización.|
|Crear modelos|Crear modelos|Cree modelos en el repositorio de modelos de empresa.|
|Publicar modelos|Publicar modelos|Publique modelos para que sean públicos y cualquiera pueda verlos.|

En la tabla siguiente se resumen los roles admitidos y sus funcionalidades en el repositorio de modelos que se pueden utilizar para la administración del acceso.

|Role|Capacidad|
|----|----------|
|Administrador de inquilinos|Administrar el acceso, leer modelos|
|Creador|Crear modelos, leer modelos|
|Publicador|Publicar modelos, leer modelos|

#### <a name="passing-a-security-token-when-accessing-company-models-with-a-rest-api"></a>Paso de un token de seguridad al acceder a los modelos de empresa con una API REST

Cuando llame a las API REST para administrar los modelos de empresa privados o compartidos, debe proporcionar un token de autorización para el usuario o la entidad de servicio en formato JWT. Consulte la sección [Información adicional](#additional-information) para saber cómo obtener un token JWT para un usuario o una entidad de servicio.

El token de JWT se debe pasar en el encabezado HTTP de autorización de la API cuando el destino sean los modelos de empresa o los modelos compartidos. El token JWT no es necesario cuando el destino sean los modelos públicos.

```csharp
// sample token
var authorizationToken = "eyJhbGciOiJIUzI1NiIsInR5cCTI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c";
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authorizationToken);
```

### <a name="view-company-or-shared-models"></a>Visualización de los modelos de empresa o compartidos

Debe ser miembro del rol *Reader* (Lector) del inquilino del repositorio o el modelo debe compartirse con usted para leer un modelo. Puede ver una lista de modelos no publicados y publicados que se han compartido con usted. De forma predeterminada, los usuarios pueden leer los modelos de su empresa, los modelos que otras empresas han compartido con ellos y todos los modelos públicos.

Para ver un modelo de empresa o compartido mediante el portal:

1. Inicie sesión en el [portal del repositorio de modelos de Azure IoT](https://aka.ms/iotmodelrepo).

1. Expanda **Company Models** – **Unpublished** (Modelos de empresa - No publicados) en el panel izquierdo

    ![Ver modelos de empresa](./media/concepts-model-repository/view-company-models.png)

1. Expanda **Shared models – Unpublished** (Modelos compartidos - No publicados) en el panel izquierdo.

    ![Ver modelos compartidos](./media/concepts-model-repository/view-shared-models.png)

Para ver un modelo de empresa o compartido con la API REST, consulte la documentación de la [API REST de obtención de modelos](https://docs.microsoft.com/rest/api/iothub/digitaltwinmodelrepositoryservice/getmodelasync/getmodelasync). Para información sobre cómo pasar un encabezado de autorización JWT en la solicitud HTTP; consulte [Paso de un token de seguridad al acceder a los modelos de empresa con una API REST](#passing-a-security-token-when-accessing-company-models-with-a-rest-api).

```csharp
var modelId = "dtmi:com:mxchip:model;1";
var response = await httpClient.GetAsync($"/models/{modelId}?api-version=2020-05-01-preview").ConfigureAwait(false);
```

Para ver un modelo de empresa o compartido mediante la CLI, consulte el comando [Get Model](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/pnp/model?view=azure-cli-latest#ext-azure-iot-az-iot-pnp-model-show) (Obtener modelo) de la CLI de Azure.

### <a name="manage-roles"></a>Administrar roles

El administrador de inquilinos puede asignar roles a los usuarios del inquilino del repositorio para que puedan crear modelos privados para la empresa u organización, publicar modelos o administrar roles de otros usuarios.

Para agregar un usuario a un rol de inquilino del repositorio de modelos mediante el portal:

1. Inicie sesión en el [portal del repositorio de modelos de Azure IoT](https://aka.ms/iotmodelrepo).

1. Seleccione **Access management** (Administración de acceso) en el panel izquierdo y, luego, elija **+Add** (+Agregar). En el panel **Add Permission** (Agregar permiso), escriba la dirección de trabajo del usuario al que quiere agregar el rol:

    ![Agregar dirección del trabajo](./media/concepts-model-repository/add-user.png)

1. En la lista desplegable **Role** (Rol), elija el rol que quiere agregar al usuario. Después, seleccione **Save** (Guardar):

    ![Elegir rol](./media/concepts-model-repository/choose-role.png)

### <a name="upload-a-model"></a>Carga de un modelo

Para cargar un modelo en el repositorio de modelos de empresa, debe ser miembro del rol **Creator** (Creador) del inquilino del repositorio.

Estos modelos no se publican, y solo los usuarios de la organización pueden acceder a ellos de forma predeterminada. También puede compartir uno o varios modelos no publicados con usuarios externos.

Los modelos cargados son inmutables.

Los identificadores de modelo de estos modelos deben ser únicos globalmente en todos los inquilinos del repositorio de todos los modelos cargados.

Para cargar un modelo mediante el portal:

1. Inicie sesión en el [portal del repositorio de modelos de Azure IoT](https://aka.ms/iotmodelrepo).

1. Expanda **Company Models** (Modelos de empresa) en el panel izquierdo y seleccione **Create model** (Crear modelo). Luego, seleccione **Import Json** (Importar JSON).

    ![Crear modelo](./media/concepts-model-repository/create-model.png)

1. Seleccione el archivo que quiere cargar. Si el portal valida correctamente el modelo, seleccione **Save** (Guardar).

Para cargar un modelo mediante la API REST, consulte la [API de creación de modelos](https://docs.microsoft.com/rest/api/iothub/digitaltwinmodelrepositoryservice/createorupdateasync/createorupdateasync). Para información sobre cómo pasar un encabezado de autorización JWT en la solicitud HTTP; consulte [Paso de un token de seguridad al acceder a los modelos de empresa con una API REST](#passing-a-security-token-when-accessing-company-models-with-a-rest-api).

```csharp
var httpContent = new StringContent(jsonLdModel, Encoding.UTF8, "application/json");
var modelId = "dtmi:com:mxchip:model;1";
var response = await httpClient.PutAsync($"/models/{modelId}?api-version=2020-05-01-preview", httpContent).ConfigureAwait(false);
```

Para cargar un modelo mediante la CLI, consulte el comando [Create a Model](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/pnp/model?view=azure-cli-latest#ext-azure-iot-az-iot-pnp-model-create) (Crear un modelo) de la CLI de Azure.

### <a name="publish-a-model"></a>Publicación de modelos

Para publicar un modelo, deben cumplirse los siguientes requisitos:

1. La organización debe ser miembro de [Microsoft Partner Network](https://docs.microsoft.com/partner-center/) para publicar un modelo. Para crear una cuenta del Centro de partners, consulte [Creación de una cuenta del Centro de partners](https://docs.microsoft.com/partner-center/mpn-create-a-partner-center-account). Una vez aprobada su cuenta, puede publicar los modelos. Para más información, consulte [Preguntas frecuentes sobre el Centro de partners](https://support.microsoft.com/help/4340639/partner-center-account-faqs).

2. El usuario debe ser miembro del rol *Publisher* (Editor) del inquilino del repositorio.

Los modelos creados y publicados por los usuarios de la organización son visibles como *modelos publicados*. Estos modelos son públicos y los puede encontrar cualquier usuario en **Public Models** (Modelos públicos).

Para publicar un modelo mediante el portal:

1. Inicie sesión en el [portal del repositorio de modelos de Azure IoT](https://aka.ms/iotmodelrepo).

2. Expanda **Company Models** (Modelos de empresa) en el panel izquierdo y seleccione el modelo que quiere publicar. Después, seleccione **Publicar**.

    ![Publicar modelo](./media/concepts-model-repository/publish-model.png)

> [!NOTE]
> Si recibe una notificación que indica que no tiene un identificador de Microsoft Partner (MPN), siga los pasos de registro de la notificación. Para más información, consulte los requisitos al principio de esta sección.

Para publicar un modelo mediante la API REST, consulte la documentación de la [API REST de publicación de modelos](https://docs.microsoft.com/rest/api/iothub/digitaltwinmodelrepositoryservice/createorupdateasync/createorupdateasync). Proporcione el parámetro de cadena de consulta `update-metadata=true` para publicar un modelo mediante la API REST. Para información sobre cómo pasar un encabezado de autorización JWT en la solicitud HTTP; consulte [Paso de un token de seguridad al acceder a los modelos de empresa con una API REST](#passing-a-security-token-when-accessing-company-models-with-a-rest-api).

Para publicar un modelo mediante la CLI, consulte el comando [Publish a Model](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/pnp/model?view=azure-cli-latest#ext-azure-iot-az-iot-pnp-model-publish) (Publicar un modelo) de la CLI de Azure.

### <a name="share-a-model"></a>Compartir un modelo

Puede compartir los modelos de empresa que ha creado con usuarios de organizaciones externas. Así, puede permitir que los colaboradores vean y desarrollen soluciones con los modelos de empresa privados.

Por ejemplo, pude que un fabricante de dispositivos quiera que los modelos sean privados para la empresa u organización. Quizás tengan clientes que exijan que las funcionalidades de sus dispositivos tengan carácter confidencial.

Compartir modelos entre compañías u organizaciones permite el acceso seguro a modelos que no son públicos.

Para compartir un modelo de empresa mediante el portal:

- Si es el creador de un modelo, los botones **Share** (Compartir) y **Shared with** (Compartido con) están activos cuando se ve el modelo en la sección **Company models** (Modelos de empresa).

    ![Compartir el modelo](./media/concepts-model-repository/share-model.png)

- Para compartir el modelo con un usuario externo, seleccione **Share** (Compartir). En el panel **Share model** (Compartir modelo), escriba la dirección de correo electrónico del usuario externo y seleccione **Save** (Guardar).

- Para ver los usuarios con los que ha compartido el modelo, seleccione **Shared with** (Compartido con).

- Para dejar de compartir el modelo con un usuario específico, seleccione el usuario de la lista de usuarios del panel **Shared with** (Compartido con). Luego, seleccione **Remove** (Quitar) y, cuando se le pida, confirme su elección.

    ![Detener uso compartido](./media/concepts-model-repository/stop-sharing.png)

## <a name="additional-information"></a>Información adicional

Al trabajar con Azure AD, puede que le resulten útiles los temas siguientes:

- Para crear un inquilino de Azure AD, consulte [Creación de un inquilino en Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant). La mayoría de las organizaciones ya tendrán inquilinos de Azure AD.

- Para agregar usuarios o usuarios invitados a un inquilino de Azure AD, consulte [Incorporación o eliminación de usuarios mediante Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/add-users-azure-active-directory).

- Para agregar una entidad de servicio a un inquilino de Azure AD, consulte [Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal).

- Para información sobre cómo obtener un token JWT de Azure AD para usarlo al llamar a las API REST, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-app).

## <a name="next-steps"></a>Pasos siguientes

El paso siguiente sugerido es revisar la [arquitectura de IoT Plug and Play](concepts-architecture.md).