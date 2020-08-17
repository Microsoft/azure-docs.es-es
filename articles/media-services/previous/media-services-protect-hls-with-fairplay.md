---
title: Protección del contenido HLS con Microsoft PlayReady o Apple FairPlay | Microsoft Docs
description: Este tema ofrece información general y muestra cómo usar Azure Media Services para cifrar dinámicamente el contenido HTTP Live Streaming (HLS) con Apple FairPlay. También muestra cómo utilizar el servicio de entrega de licencias de Media Services para entregar licencias de FairPlay a los clientes.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 7f4df99ccd960a6569cd293a86f21359368c139f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000580"
---
# <a name="protect-your-hls-content-with-apple-fairplay-or-microsoft-playready"></a>Protección del contenido HLS con Apple FairPlay o Microsoft PlayReady

> [!NOTE]
> Para completar este tutorial, deberá tener una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).   > No se van a agregar características o funcionalidades nuevas a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).
>

Azure Media Services permite cifrar dinámicamente el contenido HTTP Live Streaming (HLS) usando los siguientes formatos:  

* **Clave sin cifrado de sobre AES-128**

    El fragmento completo se cifra con el modo **AES-128 CBC**. El reproductor de OS X e iOS admite el descifrado de la transmisión de forma nativa. Para más información, consulte [Uso del cifrado dinámico AES-128 y del servicio de entrega de claves](media-services-protect-with-aes128.md).
* **Apple FairPlay**

    Las muestras de audio y vídeo individuales se cifran con el modo **AES-128 CBC**. **FairPlay Streaming** (FPS) se integra en los sistemas operativos de dispositivos, con compatibilidad nativa en iOS y TV de Apple. Safari en OS X habilita FPS utilizando la compatibilidad con la interfaz Encrypted Media Extensions (EME).
* **Microsoft PlayReady**

La siguiente imagen muestra el flujo de trabajo del **cifrado dinámico de HLS más FairPlay o PlayReady**.

![Diagrama de flujo de trabajo de cifrado dinámico](./media/media-services-content-protection-overview/media-services-content-protection-with-FairPlay.png)

En este artículo se muestra cómo usar Media Services para cifrar dinámicamente el contenido HLS con FairPlay de Apple. También muestra cómo utilizar el servicio de entrega de licencias de Media Services para entregar licencias de FairPlay a los clientes.

> [!NOTE]
> Si también quiere cifrar el contenido HLS con PlayReady, debe crear una clave de contenido común y asociarla con el recurso. También tiene que configurar la directiva de autorización de la clave de contenido, como se describe en el tema [Uso de cifrado dinámico común de PlayReady](media-services-protect-with-playready-widevine.md).
>
>

## <a name="requirements-and-considerations"></a>Requisitos y consideraciones

Cuando se usa Media Services para proporcionar HLS cifrado con FairPlay y entregar licencias de FairPlay se necesita lo siguiente:

  * Una cuenta de Azure. Para más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F).
  * Una cuenta de Media Services. Para crear una, consulte [Creación de una cuenta de Azure Media Services mediante Azure Portal](media-services-portal-create-account.md).
  * Suscríbase al [programa de desarrollo de Apple](https://developer.apple.com/).
  * En Apple es obligatorio que el propietario del contenido obtenga el [paquete de implementación](https://developer.apple.com/contact/fps/). Indique que ya ha implementado el módulo principal de seguridad (KSM) con Media Services y que está solicitando el paquete FPS final. Hay instrucciones que aparecen en el paquete FPS final para generar certificados y obtener la clave secreta de la aplicación (ASK). Utilice la ASK para configurar FairPlay.
  * Versión **3.6.0** del SDK .NET de Azure Media Services.

En la entrega de claves de Media Services se debe establecer lo siguiente:

  * **Certificado de aplicación (CA)** : Se trata de un archivo .pfx que contiene la clave privada. Puede crear este archivo y cifrarlo con una contraseña.

       Cuando configure la directiva de entrega de claves, tiene que proporcionar esa contraseña y el archivo .pfx en formato Base64.

      En los pasos siguientes se describe cómo generar un certificado pfx para FairPlay:

    1. Instale OpenSSL desde https://slproweb.com/products/Win32OpenSSL.html.

        Vaya a la carpeta donde están los certificados FairPlay y otros archivos cuyo emisor sea Apple.
    2. Ejecute el siguiente comando desde la línea de comando. Esto convierte el archivo .cer en un archivo .pem.

        "C:\OpenSSL-Win32\bin\openssl.exe" x509 -inform der -in FairPlay.cer -out FairPlay-out.pem
    3. Ejecute el siguiente comando desde la línea de comando. Esto convierte el archivo .pem a un archivo .pfx con la clave privada. A continuación, OpenSSL pide la contraseña para el archivo .pfx.

        "C:\OpenSSL-Win32\bin\openssl.exe" pkcs12 -export -out FairPlay-out.pfx -inkey privatekey.pem -in FairPlay-out.pem -passin file:privatekey-pem-pass.txt
  * **Contraseña de certificado de la aplicación**: La contraseña para crear el archivo .pfx.
  * **Identificador de la contraseña de certificado de la aplicación**: Tiene que cargar la contraseña de forma similar a como carga otras claves de Media Services. Use el valor de enumeración **ContentKeyType.FairPlayPfxPassword** para obtener el identificador de Media Services. Esto es lo que se necesita usar dentro de la opción de directiva de entrega de claves.
  * **iv**: Se trata de un valor aleatorio de 16 bytes. Tiene que coincidir con el iv de la directiva de entrega de recursos. El cliente genera el iv y lo pone en ambos lugares: en la directiva de entrega de recursos y en la opción de directiva de entrega de claves.
  * **ASK**: Esta clave se recibe cuando se genera la certificación mediante el portal para desarrolladores de Apple (Apple Developer). Cada equipo de desarrollo recibe una única ASK. Guarde una copia de la ASK y almacénela en un lugar seguro. Necesita configurar la ASK como FairPlayAsk en Media Services más adelante.
  * **Identificador de ASK**: Este identificador se obtiene cuando se carga la ASK en Media Services. Tiene que cargar la ASK mediante el valor de enumeración **ContentKeyType.FairPlayAsk**. Como resultado, se obtiene el identificador de Media Services, que debe utilizarse al establecer la opción de directiva de entrega de claves.

En el lado cliente FPS se debe establecer lo siguiente:

  * **Certificado de aplicación (CA)** : Se trata de un archivo .cer/.der que contiene la clave pública que el sistema operativo usa para cifrar alguna carga útil. Media Services necesita tener información sobre él porque lo necesita el reproductor. El servicio de entrega de claves lo descifra utilizando la clave privada correspondiente.

Para reproducir una transmisión cifrada FairPlay, obtenga primero la ASK real y luego genere un certificado real. Este proceso crea todas las tres partes:

  * archivo .der
  * archivo .pfx
  * la contraseña para el archivo .pfx

Los siguientes clientes son compatibles con HLS con cifrado **AES-128 CBC**: Safari en OS X, TV Apple e iOS.

## <a name="configure-fairplay-dynamic-encryption-and-license-delivery-services"></a>Configuración del cifrado dinámico y los servicios de entrega de licencias de FairPlay
Estos son los pasos generales para proteger los recursos con FairPlay, mediante el servicio de entrega de licencias de Media Services y también mediante cifrado dinámico.

1. Creación de un recurso y carga de los archivos en el recurso.
2. Codificación del recurso que contiene el archivo para el conjunto de MP4 de velocidad de bits adaptable.
3. Creación de una clave de contenido y su asociación con el recurso codificado.  
4. Configuración de la directiva de autorización de la clave de contenido. Especifique lo siguiente:

   * Método de entrega (en este caso, FairPlay).
   * Configuración de opciones de directiva de FairPlay. Para más detalles sobre cómo configurar FairPlay, consulte el método **ConfigureFairPlayPolicyOptions()** en el ejemplo más adelante.

     > [!NOTE]
     > En general, lo más seguro es que configure las opciones de directiva de FairPlay solo una vez, ya que solo tendrá un conjunto de certificados y una ASK.
     >
     >
   * Restricciones (abiertas o token).
   * La información específica del tipo de entrega de claves que define cómo se entrega la clave al cliente.
5. Configuración de directivas de entrega de recursos. La configuración de directiva de entrega incluye:

   * Protocolo de entrega (HLS).
   * El tipo de cifrado dinámico (cifrado CBC común).
   * La dirección URL de adquisición de licencias.

     > [!NOTE]
     > Si desea entregar una transmisión que se cifra con FairPlay + otro sistema de administración de derechos digitales (DRM), tiene que configurar directivas de entrega independientes:
     >
     > * Una IAssetDeliveryPolicy para configurar Dynamic Adaptive Streaming sobre HTTP (DASH) con Common Encryption (CENC) (PlayReady + Widevine), y Smooth con PlayReady
     > * Otra IAssetDeliveryPolicy para configurar FairPlay para HLS
     >
     >
6. Creación de un localizador a petición para obtener una URL de streaming.

## <a name="use-fairplay-key-delivery-by-player-apps"></a>Uso de la entrega de la clave FairPlay por aplicaciones reproductor
Puede desarrollar aplicaciones de reproductor usando el SDK de iOS. Para poder reproducir el contenido de FairPlay, tiene que implementar el protocolo de intercambio de licencias. Apple no especifica este protocolo. Depende de cada aplicación decidir cómo enviar las solicitudes de entrega de claves. El servicio de entrega de claves de FairPlay de Media Services espera que SPC funcione como mensaje de publicación codificado www-form-url con el formato siguiente:

`spc=<Base64 encoded SPC>`

> [!NOTE]
> Azure Media Player admite la reproducción de FairPlay. Consulte la [documentación de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) para más información.
>
>

## <a name="streaming-urls"></a>Direcciones URL de streaming
Si el recurso se cifró con más de un DRM, debe usar una etiqueta de cifrado en la dirección URL de streaming: (formato = 'm3u8-aapl', cifrado = 'xxx').

Se aplican las siguientes consideraciones:

* Se puede especificar solo un tipo de cifrado o ninguno.
* El tipo de cifrado no tiene que especificarse en la dirección URL si solo se aplicó un cifrado al recurso.
* El tipo de cifrado distingue mayúsculas de minúsculas.
* Se pueden especificar los siguientes tipos de cifrado:  
  * **cenc**:  Cifrado común (PlayReady o Widevine)
  * **cbcs-aapl**: FairPlay
  * **cbc**: Cifrado de sobre AES

## <a name="create-and-configure-a-visual-studio-project"></a>Creación y configuración de un proyecto de Visual Studio

1. Configure el entorno de desarrollo y rellene el archivo app.config con la información de la conexión, como se describe en [Desarrollo de Media Services con .NET](media-services-dotnet-how-to-use.md). 
2. Agregue los siguientes elementos al elemento **appSettings** definido en el archivo app.config:

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

## <a name="example"></a>Ejemplo

El ejemplo siguiente muestra la capacidad para usar Media Services para entregar el contenido cifrado con FairPlay. Esta funcionalidad se introdujo en el SDK de Azure Media Services para .NET versión 3.6.0. 

Sobrescriba el código del archivo Program.cs con el código mostrado en esta sección.

>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este](media-services-dotnet-manage-entities.md#limit-access-policies) artículo.

Asegúrese de actualizar las variables para que apunten a las carpetas donde se encuentran los archivos de entrada.

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
using Microsoft.WindowsAzure.MediaServices.Client.FairPlay;
using Newtonsoft.Json;
using System.Security.Cryptography.X509Certificates;

namespace DynamicEncryptionWithFairPlay
{
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static readonly Uri _sampleIssuer =
            new Uri(ConfigurationManager.AppSettings["Issuer"]);
        private static readonly Uri _sampleAudience =
            new Uri(ConfigurationManager.AppSettings["Audience"]);

        // Field for service context.
        private static CloudMediaContext _context = null;

        private static readonly string _mediaFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _singleMP4File =
            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            bool tokenRestriction = false;
            string tokenTemplateString = null;

            IAsset asset = UploadFileAndCreateAsset(_singleMP4File);
            Console.WriteLine("Uploaded asset: {0}", asset.Id);

            IAsset encodedAsset = EncodeToAdaptiveBitrateMP4Set(asset);
            Console.WriteLine("Encoded asset: {0}", encodedAsset.Id);

            IContentKey key = CreateCommonCBCTypeContentKey(encodedAsset);
            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
            Console.WriteLine("FairPlay License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay));
            Console.WriteLine();

            if (tokenRestriction)
                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
            else
                AddOpenAuthorizationPolicy(key);

            Console.WriteLine("Added authorization policy: {0}", key.AuthorizationPolicyId);
            Console.WriteLine();

            CreateAssetDeliveryPolicy(encodedAsset, key);
            Console.WriteLine("Created asset delivery policy. \n");
            Console.WriteLine();

            if (tokenRestriction && !String.IsNullOrEmpty(tokenTemplateString))
            {
                // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
                // back into a TokenRestrictionTemplate class instance.
                TokenRestrictionTemplate tokenTemplate =
                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

                // Generate a test token based on the data in the given TokenRestrictionTemplate.
                // Note, you need to pass the key id Guid because we specified
                // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey,
                                            DateTime.UtcNow.AddDays(365));
                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
                Console.WriteLine();
            }

            string url = GetStreamingOriginLocator(encodedAsset);
            Console.WriteLine("Encrypted HLS URL: {0}/manifest(format=m3u8-aapl)", url);

            Console.ReadLine();
        }

        static public IAsset UploadFileAndCreateAsset(string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset inputAsset)
        {
            var encodingPreset = "Adaptive Streaming";

            IJob job = _context.Jobs.Create(String.Format("Encoding {0}", inputAsset.Name));

            var mediaProcessors =
            _context.MediaProcessors.Where(p => p.Name.Contains("Media Encoder Standard")).ToList();

            var latestMediaProcessor =
            mediaProcessors.OrderBy(mp => new Version(mp.Version)).LastOrDefault();

            ITask encodeTask = job.Tasks.AddNew("Encoding", latestMediaProcessor, encodingPreset, TaskOptions.None);
            encodeTask.InputAssets.Add(inputAsset);
            encodeTask.OutputAssets.AddNew(String.Format("{0} as {1}", inputAsset.Name, encodingPreset), AssetCreationOptions.StorageEncrypted);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        static public IContentKey CreateCommonCBCTypeContentKey(IAsset asset)
        {
            // Create HLS SAMPLE AES encryption content key
            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                        keyId,
                        contentKey,
                        "ContentKey",
                        ContentKeyType.CommonEncryptionCbcs);

            // Associate the key with the asset.
            asset.ContentKeys.Add(key);

            return key;
        }


        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
        {
            // Create ContentKeyAuthorizationPolicy with Open restrictions
            // and create authorization policy          

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                    {
                    new ContentKeyAuthorizationPolicyRestriction
                    {
                        Name = "Open",
                        KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                        Requirements = null
                    }
                    };


            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();

            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
            ContentKeyDeliveryType.FairPlay,
            restrictions,
            FairPlayConfiguration);


            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with no restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key.
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;
        }

        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
        {
            string tokenTemplateString = GenerateTokenRequirements();

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                    {
                    new ContentKeyAuthorizationPolicyRestriction
                    {
                        Name = "Token Authorization Policy",
                        KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                        Requirements = tokenTemplateString,
                    }
                    };

            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();


            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                   ContentKeyDeliveryType.FairPlay,
                   restrictions,
                   FairPlayConfiguration);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with token restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;

            return tokenTemplateString;
        }

        private static string ConfigureFairPlayPolicyOptions()
        {
            // For testing you can provide all zeroes for ASK bytes together with the cert from Apple FPS SDK.
            // However, for production you must use a real ASK from Apple bound to a real prod certificate.
            byte[] askBytes = Guid.NewGuid().ToByteArray();
            var askId = Guid.NewGuid();
            // Key delivery retrieves askKey by askId and uses this key to generate the response.
            IContentKey askKey = _context.ContentKeys.Create(
                        askId,
                        askBytes,
                        "askKey",
                        ContentKeyType.FairPlayASk);

            //Customer password for creating the .pfx file.
            string pfxPassword = "<customer password for creating the .pfx file>";
            // Key delivery retrieves pfxPasswordKey by pfxPasswordId and uses this key to generate the response.
            var pfxPasswordId = Guid.NewGuid();
            byte[] pfxPasswordBytes = System.Text.Encoding.UTF8.GetBytes(pfxPassword);
            IContentKey pfxPasswordKey = _context.ContentKeys.Create(
                        pfxPasswordId,
                        pfxPasswordBytes,
                        "pfxPasswordKey",
                        ContentKeyType.FairPlayPfxPassword);

            // iv - 16 bytes random value, must match the iv in the asset delivery policy.
            byte[] iv = Guid.NewGuid().ToByteArray();

            //Specify the .pfx file created by the customer.
            var appCert = new X509Certificate2("path to the .pfx file created by the customer", pfxPassword, X509KeyStorageFlags.Exportable);

            string FairPlayConfiguration =
            Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
                appCert,
                pfxPassword,
                pfxPasswordId,
                askId,
                iv);

            return FairPlayConfiguration;
        }

        static private string GenerateTokenRequirements()
        {
            TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

            template.PrimaryVerificationKey = new SymmetricVerificationKey();
            template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();
            template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

            return TokenRestrictionTemplateSerializer.Serialize(template);
        }

        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
        {
            var kdPolicy = _context.ContentKeyAuthorizationPolicies.Where(p => p.Id == key.AuthorizationPolicyId).Single();

            var kdOption = kdPolicy.Options.Single(o => o.KeyDeliveryType == ContentKeyDeliveryType.FairPlay);

            FairPlayConfiguration configFP = JsonConvert.DeserializeObject<FairPlayConfiguration>(kdOption.KeyDeliveryConfiguration);

            // Get the FairPlay license service URL.
            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay);

            // The reason the below code replaces "https://" with "skd://" is because
            // in the IOS player sample code which you obtained in Apple developer account,
            // the player only recognizes a Key URL that starts with skd://.
            // However, if you are using a customized player,
            // you can choose whatever protocol you want.
            // For example, "https".

            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                    {AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl, acquisitionUrl.ToString().Replace("https://", "skd://")},
                    {AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs, configFP.ContentEncryptionIV}
            };

            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            assetDeliveryPolicyConfiguration);

            // Add AssetDelivery Policy to the asset
            asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        }


        /// <summary>
        /// Gets the streaming origin locator.
        /// </summary>
        /// <param name="assets"></param>
        /// <returns></returns>
        static public string GetStreamingOriginLocator(IAsset asset)
        {

            // Get a reference to the streaming manifest file from the  
            // collection of files in the asset.

            var assetFile = asset.AssetFiles.LoList().Where(f => f.Name.ToLower().
                         EndsWith(".ism")).
                         FirstOrDefault();

            // Create a 30-day readonly access policy.
            IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

            // Create a locator to the streaming content on an origin.
            ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

            // Create a URL to the manifest file.
            return originLocator.Path + assetFile.Name;
        }

        static private void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine(string.Format("{0}\n  State: {1}\n  Time: {2}\n\n",
            ((IJob)sender).Name,
            e.CurrentState,
            DateTime.UtcNow.ToString(@"yyyy_M_d__hh_mm_ss")));
        }

        static private byte[] GetRandomBuffer(int length)
        {
            var returnValue = new byte[length];

            using (var rng =
            new System.Security.Cryptography.RNGCryptoServiceProvider())
            {
                rng.GetBytes(returnValue);
            }

            return returnValue;
        }
    }
}
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps-media-services-learning-paths"></a>Pasos siguientes: Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
