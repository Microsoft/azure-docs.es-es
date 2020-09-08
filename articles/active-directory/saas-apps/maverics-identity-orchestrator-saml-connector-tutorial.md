---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Maverics Identity Orchestrator SAML Connector | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Maverics Identity Orchestrator SAML Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/12/2020
ms.author: jeedes
ms.openlocfilehash: 116ee7c8db3070a667c21a052bec739fd397a2dd
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88825569"
---
# <a name="tutorial-integrate-azure-ad-single-sign-on-with-maverics-identity-orchestrator-saml-connector"></a>Tutorial: Integración del inicio de sesión único de Azure AD con Maverics Identity Orchestrator SAML Connector

Strata proporciona una manera sencilla de integrar aplicaciones locales con Azure Active Directory (Azure AD) para la autenticación y el control de acceso.

En este artículo se describe cómo configurar Maverics Identity Orchestrator para:
* Migrar incrementalmente los usuarios de un sistema de identidad local a Azure AD durante el inicio de sesión en una aplicación local heredada.
* Enrutar las solicitudes de inicio de sesión de un producto de administración de acceso web heredado, como CA SiteMinder u Oracle Access Manager a Azure AD.
* Autenticar usuarios en aplicaciones locales que estén protegidas mediante encabezados HTTP o cookies de sesión propias después de autenticar el usuario con Azure AD.

Strata proporciona software que puede implementar en el entorno local o en la nube. Le ayuda a detectar, conectar y coordinar proveedores de identidades con el fin de crear una administración de identidades distribuida para empresas híbridas y de nube múltiple.

En este tutorial se muestra cómo migrar una aplicación web local que está protegida por un producto de administración de acceso web heredado (CA SiteMinder) para usar Azure AD para la autenticación y el control de acceso. Estos son los pasos básicos:
1. Instalar Maverics Identity Orchestrator.
2. Registrar su aplicación empresarial con Azure AD y configurarla para usar Maverics Azure AD SAML Zero Code Connector para el inicio de sesión único (SSO) basado en SAML.
3. Integrar Maverics con SiteMinder y el almacén de usuario del protocolo ligero de acceso a directorios (LDAP).
4. Configurar Azure Key Vault y configurar Maverics para usarlo como proveedor de administración de secretos.
5. Demostrar la migración de usuarios y la abstracción de sesión mediante Maverics para proporcionar acceso a una aplicación web Java local.

Para obtener instrucciones de instalación y configuración adicionales, visite el [sitio web de Strata](https://strata.io/docs).

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
- Una suscripción con el inicio de sesión único (SSO) de Maverics Identity Orchestrator SAML Connector habilitado. Para obtener el software de Maverics, póngase en contacto con el [equipo de ventas de Strata](mailto:sales@strata.io).

## <a name="install-maverics-identity-orchestrator"></a>Instalación de Maverics Identity Orchestrator

Para empezar a instalar Maverics Identity Orchestrator, consulte las [instrucciones de instalación](https://strata.io/docs).

### <a name="system-requirements"></a>Requisitos del sistema
* Sistemas operativos admitidos
  * RHEL 7+
  * CentOS 7+

* Dependencias
  * systemd

### <a name="installation"></a>Instalación

1. Obtenga el paquete Redhat Package Manager (RPM) de Maverics más reciente. Copie el paquete en el sistema en el que quiera instalar el software de Maverics.

2. Instale el paquete de Maverics; para ello, sustituya el nombre de archivo `maverics.rpm` por el suyo.

    `sudo rpm -Uvf maverics.rpm`

3. Después de instalar Maverics, se ejecutará como servicio en `systemd`. Use el siguiente comando para comprobar que el servicio se está ejecutando:

    `sudo systemctl status maverics`

De forma predeterminada, Maverics se instala en el directorio */usr/local/bin*.

Después de instalar Maverics, se crea el archivo predeterminado *maverics.yaml* en el directorio */etc/maverics*. Antes de editar la configuración para incluir `workflows` y `connectors`, el archivo de configuración tendrá el siguiente aspecto:

```yaml
# © Strata Identity Inc. 2020. All Rights Reserved. Patents Pending.

version: 0.1
listenAddress: ":7474"
```
## <a name="configuration-options"></a>Opciones de configuración
### <a name="version"></a>Versión
El campo `version` declara qué versión del archivo de configuración se está usando. Si no se especifica una, se usará la versión de configuración más reciente.

```yaml
version: 0.1
```
### <a name="listenaddress"></a>listenAddress
`listenAddress` declara qué dirección escuchará Orchestrator. Si la sección de host de la dirección está en blanco, Orchestrator escuchará todas las direcciones IP de unidifusión y multidifusión disponibles del sistema local. Si la sección de puerto de la dirección está en blanco, se elige automáticamente un número de puerto.

```yaml
listenAddress: ":453"
```
### <a name="tls"></a>TLS

El campo `tls` declara una asignación de objetos de seguridad de la capa de transporte (TLS). Los conectores y el servidor de Orchestrator pueden usar objetos TLS. Para ver todas las opciones de TLS disponibles, consulte la documentación del paquete `transport`.

Microsoft Azure requiere comunicación a través de TLS cuando se usa el inicio de sesión único basado en SAML. Para obtener información sobre la generación de certificados, vaya al [sitio web Let's Encrypt](https://letsencrypt.org/getting-started/).

La clave `maverics` está reservada para el servidor de Orchestrator. Todas las demás claves están disponibles y se pueden usar para insertar un objeto TLS en un conector determinado.

```yaml
tls:
  maverics:
    certFile: /etc/maverics/maverics.cert
    keyFile: /etc/maverics/maverics.key
```  
### <a name="include-files"></a>Archivos de inclusión

Puede definir `connectors` y `workflows` en sus propios archivos de configuración independientes y puede hacer referencia a ellos en el archivo *maverics.yaml* mediante `includeFiles` en el ejemplo siguiente:

```yaml
includeFiles:
  - workflow/sessionAbstraction.yaml
  - connector/AzureAD-saml.yaml
  - connector/siteminder.yaml
  ```

En este tutorial se usa un único archivo de configuración *maverics.yaml*.

## <a name="use-azure-key-vault-as-your-secrets-provider"></a>Uso de Azure Key Vault como proveedor de secretos

### <a name="manage-secrets"></a>Administración de secretos

Para cargar los secretos, Maverics puede integrarse con varias soluciones de administración de secretos. Las integraciones actuales incluyen un archivo, Hashicorp Vault y Azure Key Vault. Si no se especifica ninguna solución de administración de secretos, Maverics cargará secretos en texto sin formato del archivo *maverics.yaml* de forma predeterminada.

Para declarar un valor como secreto en un archivo de configuración *maverics.yaml*, encierre el secreto en corchetes angulares:

  ```yaml
  connectors:
  - name: AzureAD
    type: AzureAD
    apiToken: <AzureADAPIToken>
    oauthClientID: <AzureADOAuthClientID>
    oauthClientSecret: <AzureADOAuthClientSecret>
  ```

### <a name="load-secrets-from-a-file"></a>Carga de secretos desde un archivo

1. Para cargar los secretos de un archivo, agregue la variable de entorno `MAVERICS_SECRET_PROVIDER` en el archivo */etc/maverics/maverics.env* mediante:

   `MAVERICS_SECRET_PROVIDER=secretfile:///<PATH TO SECRETS FILE>`

2. Reinicie el servicio de Maverics; para ello, ejecute:

   `sudo systemctl restart maverics`

El contenido del archivo *secrets.yaml* se puede rellenar con cualquier número de `secrets`.

```yaml
secrets:
  AzureADAPIToken: aReallyGoodToken
  AzureADOAuthClientID: aReallyUniqueID
  AzureADOAuthClientSecret: aReallyGoodSecret
```
### <a name="set-up-an-azure-key-vault"></a>Configuración de un almacén de claves de Azure

Puede configurar un almacén de claves de Azure mediante Azure Portal o la CLI de Azure.

**Uso de Azure Portal**
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. [Cree una instancia de Key Vault](https://docs.microsoft.com/azure/key-vault/secrets/quick-create-portal#create-a-vault).
1. [Agregue los secretos al almacén de claves](https://docs.microsoft.com/azure/key-vault/secrets/quick-create-portal#add-a-secret-to-key-vault).
1. [Registre una aplicación con Azure AD](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application).
1. [Autorice a una aplicación para que use un secreto](https://docs.microsoft.com/azure/key-vault/secrets/quick-create-portal#add-a-secret-to-key-vault).

**Uso de la CLI de Azure**

1. Abra la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) y, luego, escriba el siguiente comando:

    ```shell
    az login
    ```

1. Cree un nuevo almacén de claves mediante el comando siguiente:
    ```shell
    az keyvault create --name "[VAULT_NAME]" --resource-group "[RESOURCE_GROUP]" --location "[REGION]"
    ```

1. Agregue los secretos al almacén de claves mediante el comando siguiente:
    ```shell
    az keyvault secret set --vault-name "[VAULT_NAME]" --name "[SECRET_NAME]" --value "[SECRET_VALUE]"
    ```

1. Registre una aplicación con Azure AD mediante el siguiente comando:
    ```shell
    az ad sp create-for-rbac -n "MavericsKeyVault" --skip-assignment > azure-credentials.json
    ```

1. Autorice a una aplicación para usar un secreto mediante el siguiente comando:
    ```shell
    az keyvault set-policy --name "[VAULT_NAME]" --spn [APPID] --secret-permissions list get
    #APPID can be found in the azure-credentials.json
    generated in the previous step
    ```

1. Para cargar secretos de su instancia de Azure Key Vault, establezca la variable de entorno `MAVERICS_SECRET_PROVIDER` en el archivo */etc/maverics/maverics.env* mediante las credenciales que se encuentran en el archivo *azure-credentials.json* en el siguiente formato:
 
   `MAVERICS_SECRET_PROVIDER='azurekeyvault://<KEYVAULT NAME>.vault.azure.net?clientID=<APPID>&clientSecret=<PASSWORD>&tenantID=<TENANT>'`

1. Reinicie el servicio de Maverics: `sudo systemctl restart maverics`

## <a name="configure-your-application-in-azure-ad-for-saml-based-sso"></a>Configuración de la aplicación en Azure AD para SSO basado en SAML

1. En el inquilino de Azure AD, vaya a **Aplicaciones empresariales**, busque **Maverics Identity Orchestrator SAML Connector** y, a continuación, selecciónelo.

1. En el panel **Propiedades** de Maverics Identity Orchestrator SAML Connector, establezca **¿Asignación de usuarios?** en **No** para permitir que la aplicación funcione para los usuarios migrados recientemente.

1. En el panel **Información general** de Maverics Identity Orchestrator SAML Connector, seleccione **Configurar inicio de sesión único** y, a continuación, seleccione **SAML**.

1. En el **Inicio de sesión basado en SAML** de Maverics Identity Orchestrator SAML Connector, edite **Configuración básica de SAML**; para ello, seleccione el botón **Editar** (icono de lápiz).

   ![Captura de pantalla del botón Editar de la "Configuración básica de SAML".](common/edit-urls.png)

1. Escriba un **Id. de entidad**; para ello, escriba una URL con el siguiente formato : `https://<SUBDOMAIN>.maverics.org`. El id. de entidad debe ser único en las aplicaciones del inquilino. Guarde el valor especificado aquí para incluirlo en la configuración de Maverics.

1. Escriba la **URL de respuesta** con el siguiente formato: `https://<AZURECOMPANY.COM>/<MY_APP>/`. 

1. Escriba la **URL de inicio de sesión** con el siguiente formato: `https://<AZURE-COMPANY.COM>/<MY_APP>/<LOGIN PAGE>`. 

1. Seleccione **Guardar**.

1. En la sección **Certificado de firma de SAML**, seleccione el botón **Copiar** para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdelo en su equipo.

    ![Captura de pantalla del botón Copiar de "Certificado de firma de SAML".](common/copy-metadataurl.png)

## <a name="configure-maverics-identity-orchestrator-azure-ad-saml-connector"></a>Configuración de Maverics Identity Orchestrator Azure AD SAML Connector

Maverics Identity Orchestrator Azure AD Connector es compatible con OpenID Connect y SAML Connect. Para configurar el conector, haga lo siguiente: 

1. Para habilitar el inicio de sesión único basado en SAML, establezca `authType: saml`.

1. Cree el valor de `samlMetadataURL` en el siguiente formato: `samlMetadataURL:https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>`.

1. Defina la dirección URL a la que se redirigirá Azure en la aplicación una vez que los usuarios hayan iniciado sesión con sus credenciales de Azure. Use el siguiente formato: `samlRedirectURL: https://<AZURECOMPANY.COM>/<MY_APP>`.

1. Copie el valor de EntityID configurado previamente: `samlEntityID: https://<SUBDOMAIN>.maverics.org`.

1. Copie el valor de la dirección URL de respuesta que usará Azure AD para publicar la respuesta SAML: `samlConsumerServiceURL: https://<AZURE-COMPANY.COM>/<MY_APP>`.

1. Genere una clave de firma JSON Web Token (JWT), que se usa para proteger la información de sesión de Maverics Identity Orchestrator, mediante la [herramienta OpenSSL](https://www.openssl.org/source/):

    ```shell 
    openssl rand 64 | base64
    ```
1. Copie la respuesta en la propiedad de configuración `jwtSigningKey`: `jwtSigningKey: TBHPvTtu6NUqU84H3Q45grcv9WDJLHgTioqRhB8QGiVzghKlu1mHgP1QHVTAZZjzLlTBmQwgsSoWxGHRcT4Bcw==`.

## <a name="attributes-and-attribute-mapping"></a>Atributos y asignación de atributos
La asignación de atributos se usa para definir la asignación de atributos de usuario desde un directorio de usuarios local de origen en un inquilino de Azure AD después de configurar al usuario.

Los atributos determinan los datos de usuario que podrían devolverse a una aplicación en una notificación, pasarse a las cookies de sesión o pasarse a la aplicación en variables de encabezado HTTP.

## <a name="configure-the-maverics-identity-orchestrator-azure-ad-saml-connector-yaml-file"></a>Configuración del archivo YAML de Maverics Identity Orchestrator Azure AD SAML Connector

La configuración de Maverics Identity Orchestrator Azure AD SAML Connector tendrá el siguiente aspecto:

```yaml
- name: AzureAD
  type: azure
  authType: saml
  samlMetadataURL: https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>
  samlRedirectURL: https://<AZURECOMPANY.COM>/<MY_APP>
  samlConsumerServiceURL: https://<AZURE-COMPANY.COM>/<MY_APP>
  jwtSigningKey: <SIGNING KEY>
  samlEntityID: https://<SUBDOMAIN>.maverics.org
  attributeMapping:
    displayName: username
    mailNickname: givenName
    givenName: givenName
    surname: sn
    userPrincipalName: mail
    password: password
```

## <a name="migrate-users-to-an-azure-ad-tenant"></a>Migración de usuarios a un inquilino de Azure AD

Siga esta configuración para migrar incrementalmente usuarios de un producto de administración de acceso web como CA SiteMinder, Oracle Access Manager o IBM Tivoli. También puede migrarlos desde un directorio de protocolo ligero de acceso a directorios (LDAP ) o una base de datos SQL.

### <a name="configure-your-application-permissions-in-azure-ad-to-create-users"></a>Configuración de los permisos de la aplicación en Azure AD para crear usuarios

1. En el inquilino de Azure AD, vaya a `App registrations` y seleccione la aplicación **Maverics Identity Orchestrator SAML Connector**.

1. En el panel **Maverics Identity Orchestrator SAML Connector | Certificados y secretos**, seleccione `New client secret` y, después, seleccione la opción de expiración. Seleccione el botón **Copiar** para copiar el secreto y guárdelo en su equipo.

1. En el panel **Maverics Identity Orchestrator SAML Connector | Permisos de la API**, seleccione **Agregar permiso** y, a continuación, en el panel **Solicitud de permisos de API**, seleccione **Microsoft Graph** y **Permisos de la aplicación**. 

1. En la siguiente pantalla, seleccione **User.ReadWrite.All** y, después, seleccione **Agregar permisos**. 

1. De vuelta en el panel **Permisos de API**, seleccione **Conceder consentimiento de administrador**.

### <a name="configure-the-maverics-identity-orchestrator-saml-connector-yaml-file-for-user-migration"></a>Configuración del archivo YAML de Maverics Identity Orchestrator SAML Connector para migración de usuarios

Para habilitar el flujo de trabajo de migración de los usuarios, agregue estas propiedades adicionales al archivo de configuración:
1. Escriba la **URL de Azure Graph** con el siguiente formato: `graphURL: https://graph.microsoft.com`.
1. Escriba la **URL de token de OAuth** con el siguiente formato: `oauthTokenURL: https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>`.
1. Escriba el secreto de cliente generado previamente con el siguiente formato: `oauthClientSecret: <CLIENT SECRET>`.


El archivo de configuración final de Maverics Identity Orchestrator Azure AD Connector tendrá el siguiente aspecto:

```yaml
- name: AzureAD
  type: azure
  authType: saml
  samlMetadataURL: https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>
  samlRedirectURL: https://<AZURECOMPANY.COM>/<MY_APP>
  samlConsumerServiceURL: https://<AZURE-COMPANY.COM>/<MY_APP>
  jwtSigningKey: TBHPvTtu6NUqU84H3Q45grcv9WDJLHgTioqRhB8QGiVzghKlu1mHgP1QHVTAZZjzLlTBmQwgsSoWxGHRcT4Bcw==
  samlEntityID: https://<SUBDOMAIN>.maverics.org
  graphURL: https://graph.microsoft.com
  oauthTokenURL: https://login.microsoftonline.com/<TENANT ID>/oauth2/v2.0/token
  oauthClientID: <APP ID>
  oauthClientSecret: <NEW CLIENT SECRET>
  attributeMapping:
    displayName: username
    mailNickname: givenName
    givenName: givenName
    surname: sn
    userPrincipalName: mail
    password: password
```

### <a name="configure-maverics-zero-code-connector-for-siteminder"></a>Configuración de Maverics Zero Code Connector para SiteMinder

El conector de SiteMinder se usa para migrar usuarios a un inquilino de Azure AD. Los usuarios se registran en aplicaciones locales heredadas que están protegidas por SiteMinder mediante las identidades y credenciales de Azure AD recién creadas.

En este tutorial, SiteMinder se ha configurado para proteger la aplicación heredada mediante la autenticación basada en formularios y el uso de la cookie `SMSESSION`. Para la integración con una aplicación que consume la autenticación y la información de sesión mediante encabezados HTTP, debe agregar la configuración de emulación de encabezados al conector.

En este ejemplo se asigna el atributo `username` al encabezado HTTP `SM_USER`:

```yaml
  headers:
    SM_USER: username
```

Establezca `proxyPass` en la ubicación a la que se redirigen mediante proxy las solicitudes. Normalmente, esta ubicación es el host de la aplicación protegida.

`loginPage` debe coincidir con la dirección URL del formulario de inicio de sesión que SiteMinder usa actualmente para redirigir a los usuarios para la autenticación.

```yaml
connectors:
- name: siteminder-login-form
  type: siteminder
  loginType: form
  loginPage: /siteminderagent/forms/login.fcc
  proxyPass: http://host.company.com
```

### <a name="configure-maverics-zero-code-connector-for-ldap"></a>Configuración de Maverics Zero Code Connector para LDAP

Cuando las aplicaciones están protegidas por un producto de administración de acceso web (WAM) como SiteMinder, las identidades y los atributos de usuario normalmente se almacenan en un directorio LDAP.

En esta configuración de conector se muestra cómo conectarse al directorio LDAP. El conector se configura como el almacén de usuario para SiteMinder, de modo que se pueda recopilar la información de perfil de usuario correcta durante el flujo de trabajo de migración y se pueda crear un usuario correspondiente en Azure AD.

* `baseDN` especifica la ubicación en el directorio en el que se realizará la búsqueda LDAP.

* `url` es la dirección y el puerto del servidor LDAP con el que se va a establecer la conexión.

* `serviceAccountUsername` es el nombre de usuario usado para conectarse al servidor LDAP; normalmente se expresa como un DN de enlace (por ejemplo, `CN=Directory Manager`).

* `serviceAccountPassword` es la contraseña que se usa para conectarse al servidor LDAP. Este valor se almacena en la instancia de Azure Key Vault configurada anteriormente.  

* `userAttributes` define la lista de atributos relacionados con el usuario que se van a consultar. Estos atributos se asignan posteriormente a los atributos de Azure AD correspondientes.

```yaml
- name: company-ldap
  type: ldap
  url: "ldap://ldap.company.com:389"
  baseDN: ou=People,o=company,c=US
  serviceAccountUsername: uid=admin,ou=Admins,o=company,c=US
  serviceAccountPassword: <vaulted-password>
  userAttributes:
    - uid
    - cn
    - givenName
    - sn
    - mail
    - mobile
```

### <a name="configure-the-migration-workflow"></a>Configuración del flujo de trabajo de migración

La configuración del flujo de trabajo de migración determina el modo en que Maverics migra usuarios de SiteMinder o LDAP a Azure AD.

Este flujo de trabajo:
- Usa el conector de SiteMinder para redirigir mediante proxy el inicio de sesión de SiteMinder. Las credenciales de usuario se validan mediante la autenticación SiteMinder y, a continuación, se pasan a los pasos posteriores del flujo de trabajo.
- Recupera los atributos de perfil de usuario del almacén de usuarios de SiteMinder.
- Realiza una solicitud a Microsoft Graph API para crear el usuario en el inquilino de Azure AD.

Para configurar el flujo de trabajo de migración, haga lo siguiente:

1. Asigne un nombre al flujo de trabajo (por ejemplo, **Migración de SiteMinder a Azure AD**).
1. Especifique `endpoint`, que es una ruta de acceso HTTP en la que se expone el flujo de trabajo, de modo que se desencadenan las `actions` de ese flujo de trabajo en respuesta a las solicitudes. Normalmente, `endpoint` corresponde a la aplicación que se redirige mediante proxy (por ejemplo, `/my_app`). El valor debe incluir las barras diagonales inicial y final.
1. Agregue el valor de `actions` adecuado al flujo de trabajo.

   a. Defina el método `login` para el conector de SiteMinder. El valor del conector debe coincidir con el valor del nombre de la configuración del conector.

   b. Defina el método `getprofile` para el conector LDAP.

   c.  Defina el método `createuser` para el conector de Azure AD.

    ```yaml
      workflows:
      - name: SiteMinder to Azure AD Migration
        endpoint: /my_app/
        actions:
        - connector: siteminder-login-form
          method: login
        - connector: company-ldap
          method: getprofile
        - connector: AzureAD
          method: createuser
    ```
### <a name="verify-the-migration-workflow"></a>Comprobación del flujo de trabajo de migración

1. Si el servicio Maverics aún no se está ejecutando, inícielo ejecutando el comando siguiente: . 

   `sudo systemctl start maverics`

1. Vaya a la dirección URL de inicio de sesión de proxy, `http://host.company.com/my_app`.
1. Proporcione las credenciales de usuario que se usan para iniciar sesión en la aplicación mientras está protegida por SiteMinder.
4. Vaya a **Inicio** > **Usuarios | Todos los usuarios** para comprobar que el usuario se ha creado en el inquilino de Azure AD.  

### <a name="configure-the-session-abstraction-workflow"></a>Configuración del flujo de trabajo de abstracción de sesión

El flujo de trabajo de abstracción de sesión traslada la autenticación y el control de acceso de la aplicación web local heredada al inquilino de Azure AD.

El conector de Azure usa el método `login` para redirigir al usuario a la dirección URL de inicio de sesión, suponiendo que no exista ninguna sesión.

Una vez autenticado, el token de sesión que se crea como resultado se pasa a Maverics. El método `emulate` del conector de SiteMinder se usa para emular la sesión basada en cookies o la sesión basada en el encabezado y, a continuación, agregar a la solicitud los atributos adicionales requeridos por la aplicación.

1. Asigne un nombre al flujo de trabajo (por ejemplo, **Abstracción de sesión de SiteMinder**).
1. Especifique `endpoint`, que corresponde a la aplicación que se va a redirigir mediante proxy. El valor debe incluir las barras diagonales inicial y final (por ejemplo, `/my_app/`).
1. Agregue el valor de `actions` adecuado al flujo de trabajo.

   a. Defina el método `login` para el conector de Azure. El valor de `connector` debe coincidir con el valor de `name` de la configuración del conector.

   b. Defina el método `emulate` para el conector de SiteMinder.

     ```yaml
      - name: SiteMinder Session Abstraction
        endpoint: /my_app/
        actions:
      - connector: azure
        method: login
      - connector: siteminder-login-form
        method: emulate
     ```
### <a name="verify-the-session-abstraction-workflow"></a>Comprobación del flujo de trabajo de abstracción de sesión

1. Vaya a la dirección URL de la aplicación de proxy, `https://<AZURECOMPANY.COM>/<MY_APP>`. 
    
    Se le redirigirá a la página de inicio de sesión de proxy.

1. Escriba las credenciales de usuario de Azure AD.

   Debe redirigirse a la aplicación como si se estuviera autenticando directamente con SiteMinder.
