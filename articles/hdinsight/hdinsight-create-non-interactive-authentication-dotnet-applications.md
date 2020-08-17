---
title: 'Aplicaciones .NET de autenticación no interactivas: Azure HDInsight'
description: Aprenda a crear aplicaciones Microsoft .NET de autenticación no interactivas en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/23/2019
ms.openlocfilehash: 5be217cd2afbb95c4c02a958c1299db599c349d0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074781"
---
# <a name="create-a-non-interactive-authentication-net-hdinsight-application"></a>Creación de aplicaciones .NET para HDInsight con autenticación no interactivas

Ejecute su aplicación Microsoft .NET para Azure HDInsight en la propia identidad de la aplicación (no interactiva) o en la identidad del usuario con sesión iniciada de la aplicación (interactiva). En este artículo se muestra cómo crear aplicaciones .NET de autenticación no interactivas para conectarse a Azure y administrar HDInsight. Para ver un ejemplo de una aplicación interactiva, consulte [Conexión a Azure HDInsight](hdinsight-administer-use-dotnet-sdk.md#connect-to-azure-hdinsight).

Desde su aplicación de .NET no interactiva, necesita:

* El identificador de inquilino de su suscripción a Azure (también conocido como *identificador de directorio*). Consulte [Obtención del identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
* El id. de cliente de la aplicación de Azure Active Directory (Azure AD). Consulte [Create an Azure Active Directory application](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) (Creación de una aplicación de Azure Active Directory) y el apartado sobre cómo [conseguir un id. de aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
* La clave secreta de la aplicación de Azure AD. Consulte [Obtención de la clave de autenticación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).

## <a name="prerequisites"></a>Prerequisites

Un clúster de HDInsight. Consulte el [tutorial de introducción](hadoop/apache-hadoop-linux-tutorial-get-started.md).

## <a name="assign-a-role-to-the-azure-ad-application"></a>Asignación de un rol a la aplicación de Azure AD

Asigne un [rol](../role-based-access-control/built-in-roles.md) a la aplicación de Azure AD para concederle permiso para realizar acciones. Puede establecer el ámbito en el nivel de suscripción, grupo de recursos o recurso. Los permisos se heredan en los niveles inferiores del ámbito. Por ejemplo, el hecho de agregar una aplicación al rol Lector para un grupo de recursos significa que esta puede leer el grupo de recursos y los recursos que contenga. En este artículo, establecerá el ámbito en el nivel del grupo de recursos. Para más información, consulte [Uso de asignaciones de roles para administrar el acceso a los recursos de la suscripción de Azure](../role-based-access-control/role-assignments-portal.md).

**Para agregar el rol de propietario a la aplicación de Azure AD**

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Navegue al grupo de recursos que tiene el clúster de HDInsight en el que se ejecutará su consulta de Hive posteriormente en este artículo. Si tiene un gran número de grupos de recursos, puede usar el filtro para buscar el que desee.
1. En el menú de grupo de recursos, haga clic en **Control de acceso (IAM)** .
1. Seleccione la pestaña **Asignaciones de roles** para ver las asignaciones de roles actuales.
1. En la parte superior de la página, seleccione **+ Agregar**.
1. Siga las instrucciones para agregar el rol Propietario a la aplicación de Azure AD. Después de agregar el rol correctamente, la aplicación aparece en el rol Propietario.

## <a name="develop-an-hdinsight-client-application"></a>Desarrollo de una aplicación de cliente de HDInsight

1. Cree una aplicación de consola de C#.
2. Agregue los siguientes paquetes [NuGet](https://www.nuget.org/):

    * `Install-Package Microsoft.Azure.Common.Authentication -Pre`
    * `Install-Package Microsoft.Azure.Management.HDInsight -Pre`
    * `Install-Package Microsoft.Azure.Management.Resources -Pre`

3. Ejecute el código siguiente:

    ```csharp
    using System;
    using System.Security;
    using Microsoft.Azure;
    using Microsoft.Azure.Common.Authentication;
    using Microsoft.Azure.Common.Authentication.Factories;
    using Microsoft.Azure.Common.Authentication.Models;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.HDInsight;
    
    namespace CreateHDICluster
    {
        internal class Program
        {
            private static HDInsightManagementClient _hdiManagementClient;
    
            private static Guid SubscriptionId = new Guid("<Enter your Azure subscription ID>");
            private static string tenantID = "<Enter your tenant ID (also called directory ID)>";
            private static string applicationID = "<Enter your application ID>";
            private static string secretKey = "<Enter the application secret key>";
    
            private static void Main(string[] args)
            {
                var key = new SecureString();
                foreach (char c in secretKey) { key.AppendChar(c); }
    
                var tokenCreds = GetTokenCloudCredentials(tenantID, applicationID, key);
                var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
    
                var resourceManagementClient = new ResourceManagementClient(subCloudCredentials);
                resourceManagementClient.Providers.Register("Microsoft.HDInsight");
    
                _hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);
    
                var results = _hdiManagementClient.Clusters.List();
                foreach (var name in results.Clusters)
                {
                    Console.WriteLine("Cluster Name: " + name.Name);
                    Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
                    Console.WriteLine("\t Cluster location: " + name.Location);
                    Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
                }
                Console.WriteLine("Press Enter to continue");
                Console.ReadLine();
            }
    
            /// Get the access token for a service principal and provided key.          
            public static TokenCloudCredentials GetTokenCloudCredentials(string tenantId, string clientId, SecureString secretKey)
            {
                var authFactory = new AuthenticationFactory();
                var account = new AzureAccount { Type = AzureAccount.AccountType.ServicePrincipal, Id = clientId };
                var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
                var accessToken =
                    authFactory.Authenticate(account, env, tenantId, secretKey, ShowDialog.Never).AccessToken;
    
                return new TokenCloudCredentials(accessToken);
            }
    
            public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(SubscriptionCloudCredentials creds, Guid subId)
            {
                return new TokenCloudCredentials(subId.ToString(), ((TokenCloudCredentials)creds).Token);
            }
        }
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una aplicación de Azure Active Directory y una entidad de servicio mediante Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md)
* Aprenda a [autenticar una entidad de servicio con Azure Resource Manager](../active-directory/develop/howto-authenticate-service-principal-powershell.md).
* Aprenda sobre el [control de acceso basado en rol (RBAC) de Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md).
