---
title: Uso de Data Lake Storage Gen1 con Hadoop en Azure HDInsight
description: Aprenda a consultar datos desde Azure Data Lake Storage Gen1 y a almacenar los resultados del análisis.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/10/2019
ms.openlocfilehash: 890cd7080447649396855bfbe051dca4470a4564
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546295"
---
# <a name="use-data-lake-storage-gen1-with-azure-hdinsight-clusters"></a>Uso de Data Lake Storage Gen1 con clústeres de Azure HDInsight

> [!Note] 
> Implemente nuevos clústeres mediante [Azure Data Lake Storage Gen2](hdinsight-hadoop-use-data-lake-storage-gen2.md) para mejorar el rendimiento y descubrir nuevas características.

Para analizar datos del clúster de HDInsight, puede almacenar los datos en [Azure Storage](../storage/common/storage-introduction.md), [Azure Data Lake Storage Gen 1](../data-lake-store/data-lake-store-overview.md) o [Azure Data Lake Storage Gen 2](../storage/blobs/data-lake-storage-introduction.md). Todas las opciones de almacenamiento permiten eliminar de forma segura clústeres de HDInsight que se usan para el cálculo sin perder datos del usuario.

En este artículo, aprenderá cómo funciona Data Lake Storage Gen1 con clústeres de HDInsight. Para más información sobre cómo funciona Azure Storage con clústeres de HDInsight, consulte [Uso de Azure Storage con clústeres de Azure HDInsight](hdinsight-hadoop-use-blob-storage.md). Consulte [Creación de clústeres de Apache Hadoop en HDInsight](hdinsight-hadoop-provision-linux-clusters.md) para obtener información sobre la creación de un clúster de HDInsight.

> [!NOTE]  
> Para acceder a Data Lake Storage Gen1 siempre se usa un canal seguro, así que no hay ningún nombre de esquema de sistema de archivos `adls`. Usa siempre `adl`.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="availability-for-hdinsight-clusters"></a>Disponibilidad de los clústeres de HDInsight

Apache Hadoop admite una noción del sistema de archivos predeterminado. El sistema de archivos predeterminado implica una autoridad y un esquema predeterminados. También se puede usar para resolver rutas de acceso relativas. Durante el proceso de creación del clúster de HDInsight, puede especificar un contenedor de blobs en Azure Storage como sistema de archivos predeterminado; también, con HDInsight 3.5 y versiones más recientes, puede seleccionar Azure Storage o Azure Data Lake Storage Gen1 como sistema de archivos predeterminado con algunas excepciones. 

Los clústeres de HDInsight pueden usar Data Lake Storage Gen1 de dos maneras:

* Como almacenamiento predeterminado
* Como almacenamiento adicional, con Azure Storage Blob como almacenamiento predeterminado.

A partir de ahora, solo algunos tipos o versiones de clústeres de HDInsight admiten el uso de Data Lake Storage Gen1 como cuentas de almacenamiento predeterminado y adicional:

| Tipo de clúster de HDInsight | Data Lake Storage Gen1 como almacenamiento predeterminado | Data Lake Storage Gen1 como almacenamiento adicional| Notas |
|------------------------|------------------------------------|---------------------------------------|------|
| HDInsight versión 4.0 | No | No |Gen1 ADLS no es compatible con HDInsight 4.0 |
| HDInsight versión 3.6 | Sí | Sí | Con la excepción de HBase|
| Versión de HDInsight 3.5 | Sí | Sí | Con la excepción de HBase|
| Versión de HDInsight 3.4 | No | Sí | |
| HDInsight versión 3.3 | No | No | |
| HDInsight versión 3.2 | No | Sí | |
| Storm | | |Data Lake Storage Gen1 se puede usar para escribir datos de una topología de Storm. Puede usar Data Lake Storage para datos de referencia que luego puede leer una topología de Storm.|

> [!WARNING]  
> HDInsight HBase no es compatible con Azure Data Lake Storage Gen 1

El uso de Data Lake Storage Gen1 como cuenta de almacenamiento adicional no afecta al rendimiento ni la capacidad de lectura o escritura en Azure Storage desde el clúster.

## <a name="use-data-lake-storage-gen1-as-default-storage"></a>Uso de Data Lake Storage Gen1 como almacenamiento predeterminado

Cuando se implementa HDInsight con Data Lake Storage Gen1 como almacenamiento predeterminado, los archivos relacionados con el clúster se almacenan en `adl://mydatalakestore/<cluster_root_path>/`, donde `<cluster_root_path>` es el nombre de una carpeta que crea en Data Lake Storage. Al especificar una ruta de acceso raíz para cada clúster, puede usar la misma cuenta de Data Lake Storage con más de un clúster. Por lo tanto, puede tener una configuración donde:

* Cluster1 puede usar la ruta de acceso `adl://mydatalakestore/cluster1storage`
* Cluster2 puede usar la ruta de acceso `adl://mydatalakestore/cluster2storage`

Observe que ambos clústeres usan la misma cuenta de Data Lake Storage Gen1, **mydatalakestore**. Cada clúster tiene acceso a su propio sistema de archivos raíz en Data Lake Storage. La experiencia de implementación de Azure Portal en particular le pide que use un nombre de carpeta como **/clusters/\<clustername >** para la ruta de acceso raíz.

Para poder usar Data Lake Storage Gen1 como almacenamiento predeterminado, debe conceder a la entidad de servicio acceso a las siguientes rutas:

- La raíz de la cuenta de Data Lake Storage Gen1.  Por ejemplo: adl://mydatalakestore/.
- La carpeta para todas las carpetas del clúster.  Por ejemplo: adl://mydatalakestore/clusters.
- La carpeta para el clúster.  Por ejemplo: adl://mydatalakestore/clusters/cluster1storage.

Para más información sobre cómo crear la entidad de servicio y concederle acceso, consulte Configuración del acceso a Data Lake Storage.

### <a name="extracting-a-certificate-from-azure-keyvault-for-use-in-cluster-creation"></a>Extracción de un certificado de Azure Key Vault para usarlo en la creación del clúster

Si quiere configurar Azure Data Lake Storage Gen1 como almacenamiento predeterminado para un nuevo clúster y el certificado para la entidad de servicio se almacena en Azure Key Vault, hay algunos pasos adicionales necesarios para convertir el certificado al formato correcto. Los siguientes fragmentos de código muestran cómo llevar a cabo la conversión.

En primer lugar, descargue el certificado de Key Vault y extraiga el valor de `SecretValueText`.

```powershell
$certPassword = Read-Host "Enter Certificate Password"
$cert = (Get-AzureKeyVaultSecret -VaultName 'MY-KEY-VAULT' -Name 'MY-SECRET-NAME')
$certValue = [System.Convert]::FromBase64String($cert.SecretValueText)
```

A continuación, convierta el valor de `SecretValueText` en un certificado.

```powershell
$certObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certValue,$null,"Exportable, PersistKeySet"
$certBytes = $certObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword.SecretValueText);
$identityCertificate = [System.Convert]::ToBase64String($certBytes)
```

Puede usar el `$identityCertificate` para implementar un clúster nuevo como se muestra en el siguiente fragmento de código:

```powershell
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $pathToArmTemplate `
    -identityCertificate $identityCertificate `
    -identityCertificatePassword $certPassword.SecretValueText `
    -clusterName  $clusterName `
    -clusterLoginPassword $SSHpassword `
    -sshPassword $SSHpassword `
    -servicePrincipalApplicationId $application.ApplicationId
```

## <a name="use-data-lake-storage-gen1-as-additional-storage"></a>Uso de Data Lake Storage Gen1 como almacenamiento adicional

También puede usar Data Lake Storage Gen1 como almacenamiento adicional para el clúster. En tales casos, el almacenamiento predeterminado del clúster puede ser una cuenta de Azure Storage Blob o de Data Lake Storage. Si va a ejecutar trabajos de HDInsight con los datos almacenados en Data Lake Storage como almacenamiento adicional, debe usar la ruta de acceso completa a los archivos. Por ejemplo:

    adl://mydatalakestore.azuredatalakestore.net/<file_path>

Tenga en cuenta que ahora no hay ningún elemento **cluster_root_path** en la dirección URL. Esto se debe a que, en este caso, Data Lake Storage no es un almacenamiento predeterminado, así que todo lo que debe hacer es proporcionar la ruta de acceso a los archivos.

Para poder usar Data Lake Storage Gen1 como almacenamiento adicional, solo debe conceder a la entidad de servicio acceso a las rutas en las que se almacenan los archivos.  Por ejemplo:

    adl://mydatalakestore.azuredatalakestore.net/<file_path>

Para más información sobre cómo crear la entidad de servicio y concederle acceso, consulte Configuración del acceso a Data Lake Storage.


## <a name="use-more-than-one-data-lake-storage-accounts"></a>Uso de más de una cuenta de Data Lake Storage

La incorporación de una cuenta de Data Lake Storage como adicional y la incorporación de más de una cuenta de Data Lake Storage se lleva a cabo mediante la concesión de permisos al clúster de HDInsight sobre los datos de una o varias cuentas de Data Lake Storage. Consulte Configuración del acceso de Data Lake Storage.

## <a name="configure-data-lake-storage-access"></a>Configuración del acceso de Data Lake Storage

Para configurar el acceso a Data Lake Storage desde el clúster de HDInsight, debe tener una entidad de servicio de Azure Active Directory (Azure AD). Solo un administrador de Azure AD puede crear una entidad de servicio. La entidad de servicio debe crearse con un certificado. Para más información, consulte [Guía de inicio rápido: Configuración de clústeres en HDInsight](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md) y [Creación de una entidad de servicio con un certificado autofirmado](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate).

> [!NOTE]  
> Si va a usar Azure Data Lake Storage Gen1 como almacenamiento adicional para un clúster de HDInsight, se recomienda encarecidamente hacerlo al crear el clúster, como se describe en este artículo. La incorporación de Azure Data Lake Storage Gen1 como almacenamiento adicional a un clúster de HDInsight existente no es un escenario admitido.
>

## <a name="access-files-from-the-cluster"></a>Acceso a los archivos desde el clúster

Existen varias maneras de acceder a los archivos de Data Lake Storage desde un clúster de HDInsight.

* **Con el nombre completo**. Con este enfoque, proporciona la ruta de acceso completa al archivo al que quiere acceder.

        adl://mydatalakestore.azuredatalakestore.net/<cluster_root_path>/<file_path>

* **Con el formato abreviado de la ruta de acceso**. Con este enfoque, reemplaza la ruta de acceso hasta la raíz del clúster con adl:///. Por lo tanto, en el ejemplo anterior, puede reemplazar `adl://mydatalakestore.azuredatalakestore.net/<cluster_root_path>/` por `adl:///`.

        adl:///<file path>

* **Con la ruta de acceso relativa**. Con este enfoque, solo proporciona la ruta de acceso relativa al archivo al que quiere acceder. Por ejemplo, si la ruta de acceso completa al archivo es:

        adl://mydatalakestore.azuredatalakestore.net/<cluster_root_path>/example/data/sample.log

    Puede acceder al mismo archivo sample.log mediante esta ruta de acceso relativa.

        /example/data/sample.log

## <a name="create-hdinsight-clusters-with-access-to-data-lake-storage-gen1"></a>Creación de clústeres de HDInsight con acceso a Data Lake Storage Gen1

Use los vínculos siguientes para obtener instrucciones detalladas sobre cómo crear clústeres de HDInsight con acceso a Data Lake Storage Gen1.

* [Uso del portal](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)
* [Uso de PowerShell (con Data Lake Storage Gen1 como almacenamiento predeterminado)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
* [Uso de PowerShell (con Data Lake Storage Gen1 como almacenamiento adicional)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [Uso de plantillas de Azure](../data-lake-store/data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

## <a name="refresh-the-hdinsight-certificate-for-data-lake-storage-gen1-access"></a>Actualización del certificado de HDInsight para el acceso a Data Lake Storage Gen1

El siguiente código de PowerShell de ejemplo lee un certificado de un archivo local o de Azure Key Vault, y actualiza el clúster de HDInsight con el nuevo certificado para acceder a Azure Data Lake Storage Gen1. Especifique su propio nombre de clúster de HDInsight, el nombre del grupo de recursos, el identificador de la suscripción, el identificador de la aplicación y la ruta de acceso local al certificado. Escriba la contraseña cuando se le solicite.

```powershell-interactive
$clusterName = '<clustername>'
$resourceGroupName = '<resourcegroupname>'
$subscriptionId = '01234567-8a6c-43bc-83d3-6b318c6c7305'
$appId = '01234567-e100-4118-8ba6-c25834f4e938'
$addNewCertKeyCredential = $true
$certFilePath = 'C:\localfolder\adls.pfx'
$KeyVaultName = "my-key-vault-name"
$KeyVaultSecretName = "my-key-vault-secret-name"
$certPassword = Read-Host "Enter Certificate Password"
# certSource
# 0 - create self signed cert
# 1 - read cert from file path
# 2 - read cert from key vault
$certSource = 0

Login-AzAccount
Select-AzSubscription -SubscriptionId $subscriptionId

if($certSource -eq 0)
{
    Write-Host "Generating new SelfSigned certificate"

    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
elseif($certSource -eq 1)
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}
elseif($certSource -eq 2)
{

    Write-Host "Reading the cert file from Azure Key Vault $KeyVaultName"

    $cert = (Get-AzureKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName)
    $certValue = [System.Convert]::FromBase64String($cert.SecretValueText)
    $certObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certValue, $null,"Exportable, PersistKeySet"

    $certBytes = $certObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword.SecretValueText);

    $certString =[System.Convert]::ToBase64String($certBytes)
}

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    New-AzADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
    Write-Host "Waiting for 7 minutes for the permissions to get propagated"
    Start-Sleep -s 420 #7 minutes
}

Write-Host "Updating the certificate on HDInsight cluster..."

Invoke-AzResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId; Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force
```

## <a name="next-steps"></a>Pasos siguientes
En este artículo, aprendió a usar Azure Data Lake Storage Gen1 compatible con HDFS con HDInsight. Esto le permite crear soluciones de adquisición de datos de archivado escalables y a largo plazo y usar HDInsight para desbloquear la información que hay dentro de los datos estructurados y no estructurados almacenados.

Para más información, consulte:

* [Introducción a Azure HDInsight][hdinsight-get-started]
* [Guía de inicio rápido: Set up clusters in HDInsight](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md) (Guía de inicio rápido: Configuración de clústeres en HDInsight).
* [Uso de Azure PowerShell para crear clústeres de HDInsight con Azure Data Lake Storage Gen1 (como almacenamiento adicional)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [Carga de datos en HDInsight][hdinsight-upload-data]
* [Uso de Apache Hive con HDInsight][hdinsight-use-hive]
* [Uso de Apache Pig con HDInsight][hdinsight-use-pig]
* [Uso de firmas de acceso compartido de Azure Storage para restringir el acceso a datos con HDInsight][hdinsight-use-sas]

[hdinsight-use-sas]: hdinsight-storage-sharedaccesssignature-permissions.md
[powershell-install]: /powershell/azureps-cmdlets-docs
[hdinsight-creation]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md

[blob-storage-restAPI]: https://msdn.microsoft.com/library/windowsazure/dd135733.aspx
[azure-storage-create]:../storage/common/storage-create-storage-account.md

[img-hdi-powershell-blobcommands]: ./media/hdinsight-hadoop-use-blob-storage/HDI.PowerShell.BlobCommands.png
[img-hdi-quick-create]: ./media/hdinsight-hadoop-use-blob-storage/HDI.QuickCreateCluster.png
[img-hdi-custom-create-storage-account]: ./media/hdinsight-hadoop-use-blob-storage/HDI.CustomCreateStorageAccount.png  
