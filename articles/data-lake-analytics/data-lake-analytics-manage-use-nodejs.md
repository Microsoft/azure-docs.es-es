---
title: Administración de Análisis de Azure Data Lake mediante Azure SDK para Node.js
description: En este artículo se describe cómo usar el SDK de Azure para Node.js para administrar cuentas, orígenes de datos, usuarios y trabajos de Data Lake Analytics.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 9de1bcf4-b15b-4d0b-9284-8889ecf0c438
ms.topic: how-to
ms.date: 12/05/2016
ms.custom: devx-track-javascript
ms.openlocfilehash: 5347218c17af89e58ae6219b860a8db03e1bf89b
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87415013"
---
# <a name="manage-azure-data-lake-analytics-using-azure-sdk-for-nodejs"></a>Administración de Análisis de Azure Data Lake mediante Azure SDK para Node.js
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

En este artículo se describe cómo administrar cuentas, orígenes de datos, usuarios y trabajos de Azure Data Lake Analytics con una aplicación escrita utilizando el SDK de Azure para Node.js. 

Las versiones compatibles son las siguientes:
* **Versión de Node.js: 0.10.0 o superior**
* **Versión de API de REST para la cuenta: 2015-10-01-preview**
* **Versión de API de REST para el catálogo: 2015-10-01-preview**
* **Versión de API de REST para el trabajo: 2016-03-20-preview**

## <a name="features"></a>Características
* Administración de cuentas: crear, obtener, enumerar, actualizar y eliminar.
* Administración de trabajos: enviar, obtener, enumerar y cancelar.
* Administración de catálogos: obtener y lista.

## <a name="how-to-install"></a>Cómo instalarlo
```bash
npm install azure-arm-datalake-analytics
```

## <a name="authenticate-using-azure-active-directory"></a>Autenticación mediante Azure Active Directory
 ```javascript
 var msrestAzure = require('ms-rest-azure');
 //user authentication
 var credentials = new msRestAzure.UserTokenCredentials('your-client-id', 'your-domain', 'your-username', 'your-password', 'your-redirect-uri');
 //service principal authentication
 var credentials = new msRestAzure.ApplicationTokenCredentials('your-client-id', 'your-domain', 'your-secret');
 ```

## <a name="create-the-data-lake-analytics-client"></a>Creación del cliente de Análisis de Data Lake
```javascript
var adlaManagement = require("azure-arm-datalake-analytics");
var accountClient = new adlaManagement.DataLakeAnalyticsAccountClient(credentials, 'your-subscription-id');
var jobClient = new adlaManagement.DataLakeAnalyticsJobClient(credentials, 'azuredatalakeanalytics.net');
var catalogClient = new adlaManagement.DataLakeAnalyticsCatalogClient(credentials, 'azuredatalakeanalytics.net');
```

## <a name="create-a-data-lake-analytics-account"></a>Creación de una cuenta de Análisis de Data Lake
```javascript
var util = require('util');
var resourceGroupName = 'testrg';
var accountName = 'testadlaacct';
var location = 'eastus2';

// A Data Lake Store account must already have been created to create
// a Data Lake Analytics account. See the Data Lake Store readme for
// information on doing so. For now, we assume one exists already.
var datalakeStoreAccountName = 'existingadlsaccount';

// account object to create
var accountToCreate = {
  tags: {
    testtag1: 'testvalue1',
    testtag2: 'testvalue2'
  },
  name: accountName,
  location: location,
  properties: {
    defaultDataLakeStoreAccount: datalakeStoreAccountName,
    dataLakeStoreAccounts: [
      {
        name: datalakeStoreAccountName
      }
    ]
  }
};

client.account.create(resourceGroupName, accountName, accountToCreate, function (err, result, request, response) {
  if (err) {
    console.log(err);
    /*err has reference to the actual request and response, so you can see what was sent and received on the wire.
      The structure of err looks like this:
      err: {
        code: 'Error Code',
        message: 'Error Message',
        body: 'The response body if any',
        request: reference to a stripped version of http request
        response: reference to a stripped version of the response
      }
    */
  } else {
    console.log('result is: ' + util.inspect(result, {depth: null}));
  }
});
```

## <a name="get-a-list-of-jobs"></a>Obtención de una lista de dominios
```javascript
var util = require('util');
var accountName = 'testadlaacct';
jobClient.job.list(accountName, function (err, result, request, response) {
  if (err) {
    console.log(err);
  } else {
    console.log('result is: ' + util.inspect(result, {depth: null}));
  }
});
```

## <a name="get-a-list-of-databases-in-the-data-lake-analytics-catalog"></a>Obtención de una lista de bases de datos en el catálogo de Análisis de Data Lake
```javascript
var util = require('util');
var accountName = 'testadlaacct';
catalogClient.catalog.listDatabases(accountName, function (err, result, request, response) {
  if (err) {
    console.log(err);
  } else {
    console.log('result is: ' + util.inspect(result, {depth: null}));
  }
});
```

## <a name="see-also"></a>Consulte también
* [Microsoft Azure SDK for Node.js](https://github.com/azure/azure-sdk-for-node)
