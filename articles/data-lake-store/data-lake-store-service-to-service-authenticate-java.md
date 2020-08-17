---
title: 'Autenticación entre servicios: Data Lake Storage Gen1 – SDK de Java'
description: Aprenda a realizar la autenticación de servicio a servicio con Azure Data Lake Storage Gen1 mediante Azure Active Directory con Java.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.custom: devx-track-java
ms.author: twooley
ms.openlocfilehash: 0e320557a7372af6a41038d9b3196db23d2496c3
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325056"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-java"></a>Autenticación de servicio a servicio con Azure Data Lake Storage Gen1 mediante Java

> [!div class="op_single_selector"]
> * [Uso de Java](data-lake-store-service-to-service-authenticate-java.md)
> * [Uso de SDK de .NET](data-lake-store-service-to-service-authenticate-net-sdk.md)
> * [Uso de Python](data-lake-store-service-to-service-authenticate-python.md)
> * [Uso de la API de REST](data-lake-store-service-to-service-authenticate-rest-api.md)
>
>  

En este artículo, aprenderá a usar el SDK de Java para realizar la autenticación de servicio a servicio con Azure Data Lake Storage Gen1. No se admite la autenticación de usuario final con Data Lake Storage Gen1 mediante el SDK de Java.

## <a name="prerequisites"></a>Requisitos previos

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cree una aplicación "web" de Azure Active Directory**. Debe haber completado los pasos descritos en [Service-to-service authentication with Data Lake Storage Gen1 using Azure Active Directory](data-lake-store-service-to-service-authenticate-using-active-directory.md) (Autenticación entre servicios con Data Lake Storage Gen1 mediante Azure Active Directory).

* [Maven](https://maven.apache.org/install.html). Este tutorial usa Maven para crear las dependencias de un proyecto. Aunque es posible generarlas sin utilizar un sistema como Maven o Gradle, estos sistemas facilitan mucho la administración de las dependencias.

* (Opcional) Un IDE como [IntelliJ IDEA](https://www.jetbrains.com/idea/download/), [Eclipse](https://www.eclipse.org/downloads/) o similar.

## <a name="service-to-service-authentication"></a>Autenticación entre servicios

1. Cree un proyecto de Maven mediante [mvn archetype](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) en la línea de comandos o con un entorno de desarrollo integrado. Para ver instrucciones sobre cómo crear un proyecto de Java mediante IntelliJ, consulte [este artículo](https://www.jetbrains.com/help/idea/2016.1/creating-and-running-your-first-java-application.html). Para ver instrucciones sobre cómo crear un proyecto con Eclipse, consulte [este artículo](https://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm).

2. Agregue las siguientes dependencias a su archivo **pom.xml** de Maven. Agregue el siguiente fragmento de código antes de la etiqueta **\</project>** :

    ```xml
    <dependencies>
      <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-data-lake-store-sdk</artifactId>
        <version>2.2.3</version>
      </dependency>
      <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-nop</artifactId>
        <version>1.7.21</version>
      </dependency>
    </dependencies>
    ```

    La primera dependencia es el uso del SDK de Data Lake Storage Gen1 (`azure-data-lake-store-sdk`) desde el repositorio de Maven. La segunda dependencia consiste en especificar qué plataforma de registro (`slf4j-nop`) se usará para esta aplicación. El SDK de Data Lake Storage Gen1 usa la fachada de registro [slf4j](https://www.slf4j.org/), que permite elegir entre una serie de plataformas de registro populares, como slf4j, registro de Java, Logback, etc., o no registrarse. En este ejemplo, se deshabilita el registro; por tanto, se usa el enlace **slf4j-nop**. Para usar otras opciones de registro en su aplicación, consulte [este artículo](https://www.slf4j.org/manual.html#projectDep).

3. Agregue las siguientes instrucciones de importación a la aplicación.

    ```java
    import com.microsoft.azure.datalake.store.ADLException;
    import com.microsoft.azure.datalake.store.ADLStoreClient;
    import com.microsoft.azure.datalake.store.DirectoryEntry;
    import com.microsoft.azure.datalake.store.IfExists;
    import com.microsoft.azure.datalake.store.oauth2.AccessTokenProvider;
    import com.microsoft.azure.datalake.store.oauth2.ClientCredsTokenProvider;
    ```

4. Utilice el siguiente fragmento de código de la aplicación de Java para obtener el token para la aplicación web de Active Directory que se ha creado anteriormente con una de las subclases de `AccessTokenProvider` (el siguiente ejemplo usa `ClientCredsTokenProvider`). El proveedor de tokens almacena en caché las credenciales que se utilizan para obtener el token en la memoria y renueva el token de forma automática si está a punto de expirar. Puede crear sus propias subclases de `AccessTokenProvider` para que los tokens se obtengan mediante su código de cliente. Por ahora vamos a usar simplemente el que se proporciona en el SDK.

    Reemplace **FILL-IN-HERE** por los valores reales de la aplicación web de Azure Active Directory.

    ```java
    private static String clientId = "FILL-IN-HERE";
    private static String authTokenEndpoint = "FILL-IN-HERE";
    private static String clientKey = "FILL-IN-HERE";

    AccessTokenProvider provider = new ClientCredsTokenProvider(authTokenEndpoint, clientId, clientKey);   
    ```

El SDK de Data Lake Storage Gen1 proporciona métodos útiles que permiten obtener los tokens de seguridad necesarios para comunicarse con la cuenta de Data Lake Storage Gen1. Sin embargo, el SDK no obliga a que se usen solo estos métodos. También puede usar cualquier otro medio de obtención del token, como el [SDK de Azure Active Directory](https://github.com/AzureAD/azure-activedirectory-library-for-java) o su propio código personalizado.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a usar la autenticación de usuario final para autenticarse en Data Lake Storage Gen1 mediante el SDK de Java. Ahora puede consultar los siguientes artículos, que tratan sobre cómo usar el SDK de Java con Data Lake Storage Gen1.

* [Operaciones de datos en Data Lake Storage Gen1 mediante el SDK de Java](data-lake-store-get-started-java-sdk.md)
