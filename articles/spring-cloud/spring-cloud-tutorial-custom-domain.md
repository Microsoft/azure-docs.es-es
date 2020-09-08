---
title: 'Tutorial: Asignación de un dominio personalizado existente a Azure Spring Cloud'
description: Asignación a Azure Spring Cloud de un nombre existente personalizado del servicio de nombres distribuido (DNS)
author: MikeDodaro
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 05107d0246be2273c09e91573bd30a4108ac7795
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89290334"
---
# <a name="map-an-existing-custom-domain-to-azure-spring-cloud"></a>Asignación de un dominio personalizado existente a Azure Spring Cloud
El servicio de nombres de dominio (DNS) es una técnica para almacenar nombres de nodos de red en toda una red. En este tutorial se asigna un dominio, como www.contoso.com, mediante un registro CNAME. El dominio personalizado se protege con un certificado y se muestra cómo aplicar Seguridad de la capa de transporte (TLS), también conocida como Capa de sockets seguros (SSL). 

El tráfico web se cifra mediante certificados. Estos certificados TLS/SSL se pueden almacenar en Azure Key Vault. 

## <a name="prerequisites"></a>Prerrequisitos
* Una aplicación implementada en Azure Spring Cloud (consulte [Inicio rápido: Inicio de una aplicación de Azure Spring Cloud existente desde Azure Portal](spring-cloud-quickstart.md)) o puede usar una aplicación existente.
* Un nombre de dominio con acceso al registro DNS para el proveedor de dominios, como GoDaddy.
* Un certificado privado (es decir, el certificado autofirmado) de un proveedor de terceros. El certificado debe coincidir con el dominio.
* Una instancia implementada de [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview).

## <a name="import-certificate"></a>Importación de certificado 
Para importar un certificado, es necesario que el archivo codificado PEM o PFX esté en el disco y se debe tener la clave privada. 

Para cargar el certificado en el almacén de claves:
1. Vaya a la instancia de Key Vault.
1. En el panel de navegación izquierdo, haga clic en **Certificados**.
1. En el menú superior, haga clic en **Generar o importar**.
1. En el cuadro de diálogo **Crear un certificado** en **Método de creación de certificados**, seleccione `Import`.
1. En **Cargar archivo de certificado**, vaya a la ubicación del certificado y selecciónela.
1. En **Contraseña**, escriba la clave privada del certificado.
1. Haga clic en **Crear**.

    ![Importación del certificado 1](./media/custom-dns-tutorial/import-certificate-a.png)

Para conceder acceso a Azure Spring Cloud a su almacén de claves antes de importar el certificado:
1. Vaya a la instancia de Key Vault.
1. En el panel de navegación izquierdo, haga clic en **Directiva de acceso**.
1. En el menú superior, haga clic en **Agregar directiva de acceso**.
1. Rellene la información, haga clic en el botón **Agregar** y, a continuación, **guarde** la directiva de acceso.

| Permiso de secreto | Permiso de certificado | Selección de la entidad de seguridad |
|--|--|--|
| Get, List | Get, List | Administración de dominios en Azure Spring Cloud |

![Importación del certificado 2](./media/custom-dns-tutorial/import-certificate-b.png)

También puede usar la CLI de Azure para conceder acceso a Azure Spring Cloud al almacén de claves.

Obtenga el identificador de objeto mediante el siguiente comando.
```
az ad sp show --id 03b39d0f-4213-4864-a245-b1476ec03169 --query objectId
```

Conceda acceso de lectura a Azure Spring Cloud al almacén de claves y reemplace el identificador de objeto en el siguiente comando.
```
az keyvault set-policy -g <key vault resource group> -n <key vault name>  --object-id <object id> --certificate-permissions get list --secret-permissions get list
``` 

Para importar el certificado en Azure Spring Cloud:
1. Vaya a la instancia del servicio. 
1. En el panel de navegación izquierdo de la aplicación, seleccione **Configuración de TLS/SSL**.
1. Luego, haga clic en **Import Key Vault Certificate** (Importar certificado de Key Vault).

    ![Importación de certificado](./media/custom-dns-tutorial/import-certificate.png)

También puede usar la CLI de Azure para importar el certificado:

```
az spring-cloud certificate add --name <cert name> --vault-uri <key vault uri> --vault-certificate-name <key vault cert name>
```

> [!IMPORTANT] 
> Asegúrese de conceder acceso a Azure Spring Cloud a su almacén de claves antes de ejecutar el comando de importación de certificado anterior. Si no lo ha hecho, puede ejecutar el siguiente comando para conceder los derechos de acceso.

```
az keyvault set-policy -g <key vault resource group> -n <key vault name>  --object-id 938df8e2-2b9d-40b1-940c-c75c33494239 --certificate-permissions get list
``` 

Cuando haya importado correctamente el certificado, lo verá en la lista de **Certificados de clave privada**.

![Certificado de clave privada](./media/custom-dns-tutorial/key-certificates.png)

También puede usar la CLI de Azure para mostrar una lista de certificados:

```
az spring-cloud certificate list
```

> [!IMPORTANT] 
> Para proteger un dominio personalizado con este certificado, todavía debe enlazar el certificado a un dominio determinado. Siga los pasos de este documento que se indican en **Agregar enlace SSL**.

## <a name="add-custom-domain"></a>Adición de un dominio personalizado
Puede usar un registro CNAME para asignar un nombre DNS personalizado a Azure Spring Cloud. 

> [!NOTE] 
> No se admite el registro D. 

### <a name="create-the-cname-record"></a>Crear un registro CNAME
Vaya a su proveedor de DNS y agregue un registro CNAME para asignar el dominio a < service_name>.azuremicroservices.io. Aquí <service_name> es el nombre de la instancia de Azure Spring Cloud. Se admite el dominio y el subdominio con comodín. Después de agregar el registro CNAME, la página de registros DNS se parecerá al ejemplo siguiente: 

![Página de registros DNS](./media/custom-dns-tutorial/dns-records.png)

## <a name="map-your-custom-domain-to-azure-spring-cloud-app"></a>Asignación del dominio personalizado a la aplicación de Azure Spring Cloud
Si no tiene una aplicación en Azure Spring Cloud, siga las instrucciones que se indican en [Inicio rápido: Inicio de una aplicación de Azure Spring Cloud existente desde Azure Portal](https://review.docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-launch-app-portal?branch=master).

Vaya a la página de la aplicación.

1. Seleccione **Dominio personalizado**.
2. Luego, elija **Add Custom Domain** (Agregar dominio personalizado). 

    ![Dominio personalizado](./media/custom-dns-tutorial/custom-domain.png)

3. Escriba el nombre de dominio completo para el que ha agregado un registro CNAME, como www.contoso.com. Asegúrese de que el tipo de registro de nombre de host esté establecido en CNAME (<service_name>.azuremicroservices.io)
4. Haga clic en **Validar** para habilitar el botón **Agregar**.
5. Haga clic en **Agregar**.

    ![Adición de un dominio personalizado](./media/custom-dns-tutorial/add-custom-domain.png)

También puede usar la CLI de Azure para agregar un dominio personalizado:
```
az spring-cloud app custom-domain bind --domain-name <domain name> --app <app name> 
```

Una aplicación puede tener varios dominios, pero un dominio solo puede asignarse a una aplicación. Cuando haya asignado correctamente el dominio personalizado a la aplicación, lo verá en la tabla de dominios personalizados.

![Tabla de dominios personalizados](./media/custom-dns-tutorial/custom-domain-table.png)

También puede usar la CLI de Azure para mostrar una lista de dominios personalizados:
```
az spring-cloud app custom-domain list --app <app name> 
```

> [!NOTE]
> Una etiqueta **Not Secure** (No seguro) para el dominio personalizado significa que aún no está enlazado a un certificado SSL. Todas las solicitudes HTTPS a este dominio personalizado desde un explorador recibirán un error o una advertencia.

## <a name="add-ssl-binding"></a>Agregar enlace SSL
En la tabla de dominios personalizados, seleccione **Agregar enlace SSL** como se muestra en la ilustración anterior.  
1. Seleccione el **certificado** o impórtelo.
1. Haga clic en **Save**(Guardar).

    ![Agregar enlace SSL 1](./media/custom-dns-tutorial/add-ssl-binding.png)

También puede usar la CLI de Azure para **agregar un enlace SSL**:
```
az spring-cloud app custom-domain update --domain-name <domain name> --certificate <cert name> --app <app name> 
```

Después de agregar correctamente el enlace SSL, el estado del dominio será seguro: **Correcto**. 

![Agregar enlace SSL 2](./media/custom-dns-tutorial/secured-domain-state.png)

## <a name="enforce-https"></a>Aplicación de HTTPS
De forma predeterminada, los usuarios pueden seguir accediendo a la aplicación mediante HTTP, pero puede redirigir todas las solicitudes HTTP al puerto HTTPS.

En la página de la aplicación, en el panel de navegación izquierdo, seleccione **Dominio personalizado**. Luego, establezca **Solo HTTPS** en *True*.

![Agregar enlace SSL 3](./media/custom-dns-tutorial/enforce-http.png)

También puede usar la CLI de Azure para aplicar HTTPS:
```
az spring-cloud app update -name <app-name> --https-only <true|false> -g <resource group> --service <service-name>
```

Una vez finalizada la operación, vaya a cualquiera de las direcciones URL HTTP que apuntan a la aplicación. Observe que las direcciones URL HTTP no funcionan.

## <a name="see-also"></a>Consulte también
* [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
* [Importación de un certificado](https://docs.microsoft.com/azure/key-vault/certificate-scenarios#import-a-certificate)
* [Inicio de la aplicación Spring Cloud desde la CLI de Azure](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-launch-app-cli)

