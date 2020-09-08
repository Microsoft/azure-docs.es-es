---
title: Protección de Azure Remote Rendering y el almacenamiento de modelos
description: Protección del contenido de una aplicación de Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 403a5b68e3320700e275c744210f480be2c88e84
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021330"
---
# <a name="tutorial-securing-azure-remote-rendering-and-model-storage"></a>Tutorial: Protección de Azure Remote Rendering y el almacenamiento de modelos

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Proteger instancias de Azure Blob Storage que contienen modelos de Azure Remote Rendering.
> * Autenticarse con Azure AD para acceder a la instancia de Azure Remote Rendering.
> * Usar credenciales de Azure para la autenticación de Azure Remote Rendering.

## <a name="prerequisites"></a>Requisitos previos

* Este tutorial se basa en el [Tutorial: Refinamiento de materiales, iluminación y efectos](..\materials-lighting-effects\materials-lighting-effects.md).

## <a name="why-additional-security-is-needed"></a>Por qué se necesita seguridad adicional

El estado actual de la aplicación y su acceso a los recursos de Azure se parecen a esta imagen:

![Seguridad inicial](./media/security-one.png)

Tanto "AccountID + AccountKey" como "URL + Token de SAS" almacenan básicamente un nombre de usuario y una contraseña juntos. Por ejemplo, si se expusieran "AccountID + AccountKey", un atacante no tendría ningún problema para usar los recursos de ARR sin su permiso a su costa.

## <a name="securing-your-content-in-azure-blob-storage"></a>Protección del contenido de Azure Blob Storage

Azure Remote Rendering puede acceder de forma segura al contenido de la instancia de Azure Blob Storage con la configuración correcta. Consulte [ Vinculación de cuentas de almacenamiento](../../../how-tos/create-an-account.md#link-storage-accounts) para configurar la instancia de Azure Remote Rendering con las cuentas de almacenamiento de blobs.

Cuando se usa un almacenamiento de blobs vinculado, se emplearán métodos ligeramente diferentes para cargar los modelos:

```csharp
var loadModelParams = new LoadModelFromSASParams(modelPath, modelEntity);
var loadModelAsync = ARRSessionService.CurrentActiveSession.Actions.LoadModelFromSASAsync(loadModelParams);
```

En las líneas anteriores se usa la versión `FromSAS` de los parámetros y de la acción de sesión. Estos deben convertirse a las versiones que no sean SAS:

```csharp
var loadModelParams = new LoadModelParams(storageAccountPath, blobContainerName, modelPath, modelEntity);
var loadModelAsync = ARRSessionService.CurrentActiveSession.Actions.LoadModelAsync(loadModelParams);
```

Vamos a modificar **RemoteRenderingCoordinator** para cargar un modelo personalizado, desde una cuenta de blobs vinculada.

1. Si aún no lo ha hecho, realice el tutorial [Procedimientos: Vinculación de las cuentas de almacenamiento](../../../how-tos/create-an-account.md#link-storage-accounts) para conceder a la instancia de ARR permiso de acceso a la instancia de Blob Storage.
1. Agregue el siguiente método modificado **LoadModel** a **RemoteRenderingCoordinator** justo debajo del método **LoadModel** actual:

    ```csharp
    /// <summary>
    /// Loads a model from blob storage that has been linked to the ARR instance
    /// </summary>
    /// <param name="storageAccountName">The storage account name, this contains the blob containers </param>
    /// <param name="blobContainerName">The blob container name, i.e. arroutput</param>
    /// <param name="modelPath">The relative path inside the container to the model, i.e. test/MyCustomModel.arrAsset</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <returns></returns>
    public async Task<Entity> LoadModel(string storageAccountName, string blobContainerName, string modelPath, Transform parent = null, ProgressHandler progress = null)
    {
        //Create a root object to parent a loaded model to
        var modelEntity = ARRSessionService.CurrentActiveSession.Actions.CreateEntity();

        //Get the game object representation of this entity
        var modelGameObject = modelEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

        //Ensure the entity will sync its transform with the server
        var sync = modelGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;

        //Parent the new object under the defined parent
        if (parent != null)
        {
            modelGameObject.transform.SetParent(parent, false);
            modelGameObject.name = parent.name + "_Entity";
        }

    #if UNITY_WSA
        //Anchor the model in the world, prefer anchoring parent if there is one
        if (parent != null)
        {
            parent.gameObject.AddComponent<WorldAnchor>();
        }
        else
        {
            modelGameObject.AddComponent<WorldAnchor>();
        }
    #endif

        //Load a model that will be parented to the entity
        var loadModelParams = new LoadModelParams($"{storageAccountName}.blob.core.windows.net", blobContainerName, modelPath, modelEntity);
        var loadModelAsync = ARRSessionService.CurrentActiveSession.Actions.LoadModelAsync(loadModelParams);
        if (progress != null)
            loadModelAsync.ProgressUpdated += progress;
        var result = await loadModelAsync.AsTask();
        return modelEntity;
    }
    ```

    En su mayor parte, este código es idéntico al método `LoadModel` original; sin embargo, se ha reemplazado la versión SAS de las llamadas de método por las versiones que no son SAS.

    También se han agregado las entradas adicionales `storageAccountName` y `blobContainerName` a los argumentos. Llamaremos a este nuevo método **LoadModel** desde otro método parecido al primer método **LoadTestModel** que hemos creado en el primer tutorial.

1. Agregue el método siguiente a **RemoteRenderingCoordinator** justo después de **LoadTestModel**

    ```csharp
    private bool loadingLinkedCustomModel = false;

    [SerializeField]
    private string storageAccountName;
    public string StorageAccountName {
        get => storageAccountName.Trim();
        set => storageAccountName = value;
    }

    [SerializeField]
    private string blobContainerName;
    public string BlobContainerName {
        get => blobContainerName.Trim();
        set => blobContainerName = value;
    }

    [SerializeField]
    private string modelPath;
    public string ModelPath {
        get => modelPath.Trim();
        set => modelPath = value;
    }

    [ContextMenu("Load Linked Custom Model")]
    public async void LoadLinkedCustomModel()
    {
        if (CurrentCoordinatorState != RemoteRenderingState.RuntimeConnected)
        {
            Debug.LogError("Please wait for the runtime to connect before loading the test model. Try again later.");
            return;
        }
        if (loadingLinkedCustomModel)
        {
            Debug.Log("Linked Test model already loading or loaded!");
            return;
        }
        loadingLinkedCustomModel = true;

        // Create a parent object to use for positioning
        GameObject testParent = new GameObject("LinkedCustomModel");
        testParent.transform.position = new Vector3(0f, 0f, 3f);

        await LoadModel(StorageAccountName, BlobContainerName, ModelPath, testParent.transform, (progressValue) => Debug.Log($"Loading Test Model progress: {Math.Round(progressValue * 100, 2)}%"));
    }
    ```

    Este código agrega tres variables de cadena adicionales al componente **RemoteRenderingCoordinator**.
    ![Modelo vinculado](./media/storage-account-linked-model.png)

1. Agregue los valores al componente **RemoteRenderingCoordinator**. Tras haber seguido el [inicio rápido para la conversión de modelos](../../../quickstarts/convert-model.md), los valores serán:

    * **Nombre de cuenta de almacenamiento**: el nombre de la cuenta de almacenamiento, el nombre único global que elija para la cuenta de almacenamiento. En el inicio rápido fue *arrtutorialstorage*; su valor será diferente.
    * **Blob Container Name** (Nombre de contenedor de blobs): arroutput, el contenedor de Blob Storage.
    * **Model Path** (Ruta de acceso del modelo): la combinación de "outputFolderPath" y "outputAssetFileName" definida en el archivo *arrconfig.json*. En el inicio rápido, era "outputFolderPath":"converted/robot", "outputAssetFileName": "robot.arrAsset". El resultado sería un valor de la ruta de acceso del modelo de "converted/robot/robot.arrAsset", pero el suyo será diferente.

    >[!TIP]
    > Si [ejecuta el script **Conversion.ps1**](../../../quickstarts/convert-model.md#run-the-conversion), sin el argumento "-UseContainerSas", el script generará todos los valores anteriores automáticamente en lugar del token de SAS. ![Modelo vinculado](./media/converted-output.png)
1. Mientras tanto, quite o deshabilite el elemento GameObject **TestModel** para dejar espacio para la carga del modelo personalizado.
1. Reproduzca la escena y conéctese a una sesión remota.
1. Haga clic con el botón derecho en **RemoteRenderingCoordinator** y seleccione **Load Linked Custom Model** (Cargar modelo personalizado vinculado).
    ![Cargar modelo vinculado](./media/load-linked-model.png)

Estos pasos han aumentado la seguridad de la aplicación al eliminar el token de SAS de la aplicación local.

Ahora, el estado actual de la aplicación y su acceso a los recursos de Azure se parecen a esta imagen:

![Mayor seguridad](./media/security-two.png)

Tenemos que quitar una "contraseña" más de la aplicación local, AccountKey. Para ello se puede usar la autenticación de Azure Active Directory (AAD).

## <a name="azure-active-directory-azure-ad-authentication"></a>Autenticación de Azure Active Directory (Azure AD)

La autenticación de AAD le permitirá determinar qué individuos o grupos usan ARR de una manera más controlada. ARR integra compatibilidad con [tokens de acceso](https://docs.microsoft.com/azure/active-directory/develop/access-tokens) en lugar de usar una clave de cuenta. Los tokens de acceso pueden considerarse claves específicas del usuario de duración limitada que solo desbloquean determinadas partes del recurso específico para el que se solicitaron.

El script **RemoteRenderingCoordinator** tiene un delegado llamado **ARRCredentialGetter**, que contiene un método que devuelve un objeto **AzureFrontendAccountInfo** que se usa para configurar la administración de sesiones remotas. Podemos asignar un método diferente a **ARRCredentialGetter**, lo que nos permite usar un flujo de inicio de sesión de Azure y generar un objeto de **AzureFrontendAccountInfo** que contenga un token de acceso de Azure. Este token de acceso será específico para el usuario que inicia sesión.

1. Siga el tutorial [Procedimiento: Configuración de la autenticación: autenticación para aplicaciones implementadas](../../../how-tos/authentication.md#authentication-for-deployed-applications); en concreto, seguirá las instrucciones que se indican en [Autenticación de usuarios de Azure AD](https://docs.microsoft.com/azure/spatial-anchors/concepts/authentication?tabs=csharp#azure-ad-user-authentication) de la documentación de Azure Spatial Anchors. Esto implica registrar una nueva aplicación de Azure Active Directory y configurar el acceso a la instancia de ARR.
1. Después de configurar la nueva aplicación de AAD, compruebe que tenga el aspecto que se muestra en las siguientes imágenes:

    **Aplicación AAD -> Autenticación** ![Autenticación de aplicación](./media/app-authentication-public.png)

    **Aplicación de AAD -> Permisos de API** ![API de aplicaciones](./media/request-api-permissions-step-five.png)

1. Después de configurar la cuenta de Remote Rendering, compruebe que la configuración se parece a la de la siguiente imagen:

    **AAR-> AccessControl (IAM)** ![Rol de ARR](./media/azure-remote-rendering-role-assignment-complete.png)

    >[!NOTE]
    > El rol *Propietario* no es suficiente para administrar sesiones mediante la aplicación cliente. A cada usuario al que quiera conceder la capacidad de administrar sesiones le debe proporcionar el rol **Cliente de Remote Rendering**. A cada usuario que quiera que administre las sesiones y convierta los modelos le debe proporcionar el rol **Administrador de Remote Rendering**.

Ahora que todo está en su sitio en Azure, es necesario modificar cómo el código conecta con el servicio AAR. Para ello, se implementa una instancia de **BaseARRAuthentication**, que devolverá un nuevo objeto **AzureFrontendAccountInfo**. En este caso, la información de la cuenta se configurará con el token de acceso de Azure.

1. Cree un script llamado **AADAuthentication** y reemplace su código por el siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Identity.Client;
    using System;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using UnityEngine;

    public class AADAuthentication : BaseARRAuthentication
    {
        [SerializeField]
        private string accountDomain;
        public string AccountDomain
        {
            get => accountDomain.Trim();
            set => accountDomain = value;
        }

        [SerializeField]
        private string activeDirectoryApplicationClientID;
        public string ActiveDirectoryApplicationClientID
        {
            get => activeDirectoryApplicationClientID.Trim();
            set => activeDirectoryApplicationClientID = value;
        }

        [SerializeField]
        private string azureTenantID;
        public string AzureTenantID
        {
            get => azureTenantID.Trim();
            set => azureTenantID = value;
        }

        [SerializeField]
        private string azureRemoteRenderingAccountID;
        public string AzureRemoteRenderingAccountID
        {
            get => azureRemoteRenderingAccountID.Trim();
            set => azureRemoteRenderingAccountID = value;
        }

        public override event Action<string> AuthenticationInstructions;

        string authority => "https://login.microsoftonline.com/" + AzureTenantID;

        string redirect_uri = "https://login.microsoftonline.com/common/oauth2/nativeclient";

        string[] scopes => new string[] { "https://sts.mixedreality.azure.com/mixedreality.signin" };

        public void OnEnable()
        {
            RemoteRenderingCoordinator.ARRCredentialGetter = GetAARCredentials;
            this.gameObject.AddComponent<ExecuteOnUnityThread>();
        }

        public async override Task<AzureFrontendAccountInfo> GetAARCredentials()
        {
            var result = await TryLogin();
            if (result != null)
            {
                Debug.Log("Account signin successful " + result.Account.Username);

                var AD_Token = result.AccessToken;

                return await Task.FromResult(new AzureFrontendAccountInfo(AccountDomain, AzureRemoteRenderingAccountID, "", AD_Token, ""));
            }
            else
            {
                Debug.LogError("Error logging in");
            }
            return default;
        }

        private Task DeviceCodeReturned(DeviceCodeResult deviceCodeDetails)
        {
            //Since everything in this task can happen on a different thread, invoke responses on the main Unity thread
            ExecuteOnUnityThread.Enqueue(() =>
            {
                // Display instructions to the user for how to authenticate in the browser
                Debug.Log(deviceCodeDetails.Message);
                AuthenticationInstructions?.Invoke(deviceCodeDetails.Message);
            });

            return Task.FromResult(0);
        }

        public override async Task<AuthenticationResult> TryLogin()
        {
            var clientApplication = PublicClientApplicationBuilder.Create(ActiveDirectoryApplicationClientID).WithAuthority(authority).WithRedirectUri(redirect_uri).Build();
            AuthenticationResult result = null;
            try
            {
                var accounts = await clientApplication.GetAccountsAsync();

                if (accounts.Any())
                {
                    result = await clientApplication.AcquireTokenSilent(scopes, accounts.First()).ExecuteAsync();

                    return result;
                }
                else
                {
                    try
                    {
                        result = await clientApplication.AcquireTokenWithDeviceCode(scopes, DeviceCodeReturned).ExecuteAsync(CancellationToken.None);
                        return result;
                    }
                    catch (MsalUiRequiredException ex)
                    {
                        Debug.LogError("MsalUiRequiredException");
                        Debug.LogException(ex);
                    }
                    catch (MsalServiceException ex)
                    {
                        Debug.LogError("MsalServiceException");
                        Debug.LogException(ex);
                    }
                    catch (MsalClientException ex)
                    {
                        Debug.LogError("MsalClientException");
                        Debug.LogException(ex);
                        // Mitigation: Use interactive authentication
                    }
                    catch (Exception ex)
                    {
                        Debug.LogError("Exception");
                        Debug.LogException(ex);
                    }
                }
            }
            catch (Exception ex)
            {
                Debug.LogError("GetAccountsAsync");
                Debug.LogException(ex);
            }

            return null;
        }
    }
    ```

>[!NOTE]
> Este código no está completo y no está listo para uso comercial. Por ejemplo, como mínimo querrá tener también la posibilidad de cerrar la sesión. Para ello se puede usar el método `Task RemoveAsync(IAccount account)` proporcionado por la aplicación cliente. Este código está concebido para su uso en el tutorial; la implementación de cada usuario será específica de su aplicación.

Primero, el código intenta obtener el token de forma silenciosa mediante **AquireTokenSilent**. Esta acción se realizará correctamente si el usuario ha autenticado anteriormente esta aplicación. Si no, pase a una estrategia en la que intervenga más el usuario.

En este código, se usa el [flujo de código de dispositivo](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-device-code) para obtener un token de acceso. Este flujo permite que el usuario inicie sesión en su cuenta de Azure en un equipo o dispositivo móvil y que se envíe el token resultante de vuelta a la aplicación de HoloLens.

Desde la perspectiva de ARR, la parte más importante de esta clase es esta línea:

```csharp
return await Task.FromResult(new AzureFrontendAccountInfo(AccountDomain, AzureRemoteRenderingAccountID, "", AD_Token, ""));
```

Aquí, creamos un objeto **AzureFrontendAccountInfo** con el dominio de la cuenta, el identificador de la cuenta y el token de acceso. Luego, el servicio ARR usa este token para consultar, crear y combinar sesiones de representación remota siempre y cuando el usuario esté autorizado en función de los permisos basados en rol configurados anteriormente.

Con este cambio, el estado actual de la aplicación y su acceso a los recursos de Azure se asemejan a esta imagen:

![Mayor seguridad si cabe](./media/security-three.png)

Puesto que las credenciales de usuario no se almacenan en el dispositivo (o, en este caso, ni siquiera se insertan en el dispositivo), el riesgo de exposición es muy bajo. Ahora, el dispositivo usa un token de acceso específico del usuario de tiempo limitado para acceder a ARR, que usa el control de acceso (IAM) para acceder a Blob Storage. En estos dos pasos se han eliminado por completo las "contraseñas" del código fuente y ha aumentado la seguridad de manera considerable. Sin embargo, esta no es la mejor seguridad disponible; trasladar el modelo y la administración de sesiones a un servicio web mejorará aún más la seguridad. En el capítulo [Preparación comercial](../commercial-ready/commercial-ready.md) se analizan otras cuestiones de seguridad.

### <a name="testing-aad-auth"></a>Prueba de la autenticación de AAD

En el editor de Unity, cuando la autenticación de AAD esté activa, tendrá que autenticarse cada vez que inicie la aplicación. En el dispositivo, el paso de autenticación se realizará la primera vez y solo se volverá a requerir cuando el token expire o se invalide.

1. Agregue el componente **AADAuthentication** al elemento GameObject **RemoteRenderingCoordinator**.

    ![Componente de autenticación de AAD](./media/azure-active-directory-auth-component.png)

1. Rellene los valores de identificador de cliente e identificador de inquilino. Estos valores se pueden encontrar en la página de información general del registro de la aplicación:

    * **Account Domain** (Dominio de la cuenta): es el mismo dominio que ha estado usando en el dominio de la cuenta de **RemoteRenderingCoordinator**.
    * **Active Directory Application Client ID** (Id. de cliente de la aplicación de Active Directory) es el valor de *Application (client) ID* (Id. de aplicación [cliente]) que se encuentra en el registro de la aplicación de AAD (consulte la imagen a continuación).
    * El valor de **Azure Tenant ID** (Id. de inquilino de Azure) es el valor de *Directory (tenant) ID* (Id. de directorio [inquilino]) que se encuentra en el registro de la aplicación de AAD (consulte la imagen a continuación).
    * El valor de **Azure Remote Rendering Account ID** (Id. de la cuenta de Azure Remote Rendering) es el mismo valor de **Account ID** (Id. de cuenta) que ha estado usando para **RemoteRenderingCoordinator**.

    ![Componente de autenticación de AAD](./media/app-overview-data.png)

1. Presione el botón de reproducción en el editor de Unity y dé su consentimiento a la ejecución de una sesión.
    Puesto que el componente **AADAuthentication** tiene un controlador de vista, se enlaza automáticamente para mostrar un aviso después del panel modal de autorización de la sesión.
1. Siga las instrucciones que se encuentran en el panel a la derecha de **AppMenu**.
    Verá algo parecido a esto: ![Componente de autenticación de AAD](./media/device-flow-instructions.png) Después de escribir el código proporcionado en el dispositivo secundario (o en el explorador del mismo dispositivo) e iniciar sesión con sus credenciales, se devolverá un token de acceso a la aplicación que realiza la solicitud, en este caso, el editor de Unity.
1. Llegados a este punto, todas las fases de la aplicación proseguirán normalmente. Si no es así, compruebe la consola de Unity para ver si hay errores.

## <a name="build-to-device"></a>Compilación en el dispositivo

Si va a compilar una aplicación con MSAL en el dispositivo, debe incluir un archivo en la carpeta **Assets**(Recursos) del proyecto. De esta manera, el compilador podrá compilar la aplicación correctamente mediante el archivo *Microsoft.Identity.Client.dll* incluido en los **recursos del tutorial**.

1. Agregue un nuevo archivo en **Assets** (Recursos) llamado **link.xml**.
1. Agregue al archivo lo siguiente:

    ```xml
    <linker>
        <assembly fullname="Microsoft.Identity.Client" preserve="all"/>
        <assembly fullname="System.Runtime.Serialization" preserve="all"/>
        <assembly fullname="System.Core">
            <type fullname="System.Linq.Expressions.Interpreter.LightLambda" preserve="all" />
        </assembly>
    </linker>
    ```

1. Guarde los cambios.

Siga los pasos que se describen en el [Inicio rápido: Implementación del ejemplo de Unity en HoloLens: compilación del proyecto de ejemplo](../../../quickstarts/deploy-to-hololens.md#build-the-sample-project), para realizar la compilación en HoloLens.

## <a name="next-steps"></a>Pasos siguientes

El resto de este conjunto de tutoriales contiene temas conceptuales para crear una aplicación lista para producción que usa Azure Remote Rendering.

> [!div class="nextstepaction"]
> [Siguiente: Preparación comercial](../commercial-ready/commercial-ready.md)
