---
title: 'Tutorial: Personalizar la interfaz del usuario'
titleSuffix: Azure AD B2C
description: Aprenda a personalizar la interfaz de usuario (IU) de las aplicaciones en Azure Active Directory B2C mediante Azure Portal.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 29a03f1d1b1007845c1df516192da22d74705af0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87481740"
---
# <a name="tutorial-customize-the-interface-of-user-experiences-in-azure-active-directory-b2c"></a>Tutorial: Personalización de la interfaz de experiencias de usuario en Azure Active Directory B2C

Para experiencias de usuario más comunes, como registrarse, iniciar sesión y editar perfiles, puede usar los [flujos de usuario](user-flow-overview.md) de Azure Active Directory B2C (Azure AD B2C). La información de este tutorial le ayuda a obtener información acerca de cómo [personalizar la interfaz de usuario (IU)](customize-ui-overview.md) de estas experiencias mediante sus propios archivos HTML y CSS.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Crear archivos de personalización de la interfaz de usuario
> * Actualizar el flujo de usuario para que use los archivos
> * Probar la interfaz de usuario personalizada

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

[Cree un flujo de usuario](tutorial-create-user-flows.md) para permitir a los usuarios registrarse e iniciar sesión en la aplicación.

## <a name="create-customization-files"></a>Crear archivos de personalización

Cree una cuenta de almacenamiento de Azure y el contenedor y, después, coloque los archivos básicos de HTML y CSS en el contenedor.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Aunque se pueden almacenar los archivos de muchas maneras, para este tutorial, almacénelos en el [almacenamiento de blobs de Azure](../storage/blobs/storage-blobs-introduction.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Asegúrese de que está usando el directorio que contiene su suscripción de Azure. Seleccione el filtro **Directorio y suscripciones** en el menú superior y elija el directorio que contiene su suscripción. Este directorio es diferente que el de la carpeta que contiene el inquilino de Azure B2C.
3. Elija Todos los servicios en la esquina superior izquierda de Azure Portal, busque y seleccione **Cuentas de almacenamiento**.
4. Seleccione **Agregar**.
5. En **Grupo de recursos**, seleccione **Crear nuevo** y, después, proporcione un nombre válido para el nuevo grupo de recursos y haga clic en **Aceptar**.
6. Escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure, tener entre tres y 24 caracteres y solo pueden contener números y letras minúsculas.
7. Seleccione la ubicación de la cuenta de almacenamiento o acepte la ubicación predeterminada.
8. Acepte los demás valores predeterminados, seleccione **Revisar y crear**y, después, haga clic en **Crear**.
9. Una vez creada la cuenta de almacenamiento, seleccione **Ir al recurso**.

### <a name="create-a-container"></a>Crear un contenedor

1. En la página de información general de la cuenta de almacenamiento, seleccione **Blobs**.
2. Seleccione **Contenedor**, escriba un nombre para el contenedor, elija **Blob (acceso de lectura anónimo solo para blobs)** y, después, haga clic en **Aceptar**.

### <a name="enable-cors"></a>Habilitación de CORS

 El código de Azure AD B2C en un explorador utiliza un enfoque moderno y estándar para cargar contenido personalizado desde una dirección URL que especifique en un flujo de usuario. Uso compartido de recursos entre orígenes (CORS) permite que los recursos restringidos en una página web se soliciten desde otros dominios.

1. En el menú, seleccione **CORS**.
2. En **Orígenes permitidos**, escriba `https://your-tenant-name.b2clogin.com`. Reemplace `your-tenant-name` por el nombre del inquilino de Azure AD B2C. Por ejemplo, `https://fabrikam.b2clogin.com`. Al escribir su nombre de inquilino, use solo minúsculas.
3. En **Métodos permitidos**, seleccione `GET`, `PUT` y `OPTIONS`.
4. En **Encabezados permitidos**, escriba un asterisco (*).
5. En **Encabezados expuestos**, escriba un asterisco (*).
6. Para **Antigüedad máxima**, introduzca 200.

    ![Página de configuración de CORS en Azure Blob Storage en Azure Portal](./media/tutorial-customize-ui/enable-cors.png)

5. Haga clic en **Save**(Guardar).

### <a name="create-the-customization-files"></a>Crear archivos de personalización

Para personalizar la interfaz de usuario de la experiencia de registro, primero debe crear un archivo sencillo HTML y CSS. Puede configurar su archivo HTML como desee, pero debe tener un elemento **div** con un identificador de `api`. Por ejemplo, `<div id="api"></div>`. Azure AD B2C inserta elementos en el contenedor `api` cuando se muestra la página.

1. En una carpeta local, cree el archivo siguiente y asegúrese de que cambia `your-storage-account` por el nombre de la cuenta de almacenamiento y `your-container` por el nombre del contenedor que ha creado. Por ejemplo, `https://store1.blob.core.windows.net/b2c/style.css`.

    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <title>My B2C Application</title>
        <link rel="stylesheet" href="https://your-storage-account.blob.core.windows.net/your-container/style.css">
      </head>
      <body>
        <h1>My B2C Application</h1>
        <div id="api"></div>
      </body>
    </html>
    ```

    La página se puede diseñar como quiera, pero el elemento div **api** es necesario para cualquier archivo de personalización HTML que cree.

3. Guarde el archivo como *custom-ui.html*.
4. Cree el siguiente archivo CSS sencillo que centra todos los elementos en la página de registro o de inicio de sesión, incluidos los elementos que Azure AD B2C inserta.

    ```css
    h1 {
      color: blue;
      text-align: center;
    }
    .intro h2 {
      text-align: center;
    }
    .entry {
      width: 300px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    .divider h2 {
      text-align: center;
    }
    .create {
      width: 300px ;
      margin-left: auto ;
      margin-right: auto ;
    }
    ```

5. Guarde el archivo como *style.css*.

### <a name="upload-the-customization-files"></a>Cargar archivos de personalización

En este tutorial, almacenará los archivos que ha creado en la cuenta de almacenamiento para que Azure AD B2C puede tener acceso a ellos.

1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Cuentas de almacenamiento**.
2. Seleccione la cuenta de almacenamiento que creó, seleccione **Blobs** y, después, seleccione el contenedor que ha creado.
3. Seleccione **Cargar**, vaya al archivo*custom-ui.html* y selecciónelo y, después, haga clic en **Cargar**.

    ![Página Cargar blob en el portal con el botón Cargar y Archivos resaltados](./media/tutorial-customize-ui/upload-blob.png)

4. Copie la dirección URL del archivo que cargó para usarla más adelante en el tutorial.
5. Repita el paso 3 y 4 para el archivo *style.css*.

## <a name="update-the-user-flow"></a>Actualización del flujo de usuario

1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
2. Seleccione **Flujos de usuario** y, luego, elija el flujo de usuario *B2C_1_signupsignin1*.
3. Seleccione **Diseños de página** y, a continuación, en **Página unificada de inicio de sesión o de registro**, haga clic en **Sí** para **Usar contenido de la página personalizada**.
4. En **URI de página personalizado**, escriba el identificador URI del archivo *custom-ui.html* que anotó anteriormente.
5. En la parte superior de la página, seleccione **Guardar**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario** y seleccione el flujo *B2C_1_signupsignin1*.
2. En la parte superior de la página, haga clic en **Ejecutar flujo de usuario**.
3. Haga clic en el botón **Ejecutar flujo de usuario**.

    ![Página Ejecutar flujo de usuario para el flujo de usuario de registro o de inicio de sesión](./media/tutorial-customize-ui/run-user-flow.png)

    Debería ver una página similar al ejemplo siguiente con los elementos centrados basándose en el archivo CSS que ha creado:

    ![Explorador web que muestra la página de registro o inicio de sesión con elementos de la interfaz de usuario personalizados](./media/tutorial-customize-ui/run-now.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo:

> [!div class="checklist"]
> * Crear archivos de personalización de la interfaz de usuario
> * Actualizar el flujo de usuario para que use los archivos
> * Probar la interfaz de usuario personalizada

> [!div class="nextstepaction"]
> [Personalización de la interfaz de usuario en Azure Active Directory B2C](customize-ui-overview.md)
