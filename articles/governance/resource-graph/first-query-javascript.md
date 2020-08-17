---
title: 'Inicio rápido: Su primera consulta de JavaScript'
description: En este inicio rápido, dará los pasos necesarios para habilitar la biblioteca de Resource Graph para JavaScript y ejecutará su primera consulta.
ms.date: 06/23/2020
ms.topic: quickstart
ms.custom: devx-track-javascript
ms.openlocfilehash: fc9809dbe615950b4f5c9e4dc66cefcd054f5ebf
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87415931"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-javascript"></a>Inicio rápido: Ejecución de la primera consulta de Resource Graph con JavaScript

Este inicio rápido le guiará a través del proceso de incorporación de las bibliotecas a la instalación de JavaScript. El primer paso para usar Azure Resource Graph es inicializar una aplicación de JavaScript con las bibliotecas necesarias.

Cuando finalice el proceso, habrá agregado las bibliotecas a la instalación de JavaScript y ejecutado su primera consulta de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

- **Suscripción de Azure**: si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- **Node.js**: se requiere [Node.js](https://nodejs.org/) versión 12 o posterior.

## <a name="application-setup"></a>Configuración de la aplicación

Para permitir que JavaScript consulte Azure Resource Graph, debe configurarse el entorno. Este programa de instalación funciona siempre que se pueda usar JavaScript, incluido [bash en Windows 10](/windows/wsl/install-win10).

1. Configure un nuevo proyecto de Node.js ejecutando el siguiente comando.

   ```bash
   npm init -y
   ```

1. Agregue una referencia al módulo yargs.

   ```bash
   npm install yargs
   ```

1. Agregue una referencia al módulo de Azure Resource Graph.

   ```bash
   npm install @azure/arm-resourcegraph
   ```

1. Agregue una referencia a la biblioteca de autenticación de Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Compruebe en _package.json_ que `@azure/arm-resourcegraph` utiliza la versión **2.0.0** o posterior y `@azure/ms-rest-nodeauth` la versión **3.0.3** o posterior.

## <a name="query-the-resource-graph"></a>Consulta de Resource Graph

1. Cree un nuevo archivo denominado _index.js_ y escriba el código siguiente.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const resourceGraph = require("@azure/arm-resourcegraph");

   if (argv.query && argv.subs) {
       const subscriptionList = argv.subs.split(",");

       const query = async () => {
          const credentials = await authenticator.interactiveLogin();
          const client = new resourceGraph.ResourceGraphClient(credentials);
          const result = await client.resources(
             {
                 query: argv.query,
                 subscriptions: subscriptionList,
             },
             { resultFormat: "table" }
          );
          console.log("Records: " + result.totalRecords);
          console.log(result.data);
       };

       query();
   }
   ```

1. En el terminal, escriba el siguiente comando:

   ```bash
   node index.js --query "Resources | project name, type | limit 5" --subs <YOUR_SUBSCRIPTION_ID_LIST>
   ```

   Asegúrese de reemplazar el marcador de posición `<YOUR_SUBSCRIPTION_ID_LIST>` por la lista separada por comas de identificadores de suscripción de Azure.

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Cambie el primer parámetro a `index.js` y cambie la consulta para realizar la operación `order by` en la propiedad **Nombre**. Reemplace `<YOUR_SUBSCRIPTION_ID_LIST>` por el identificador de suscripción:

   ```bash
   node index.js --query "Resources | project name, type | limit 5 | order by name asc" --subs "<YOUR_SUBSCRIPTION_ID_LIST>"
   ```

   A medida que el script intenta autenticarse, se muestra un mensaje similar al siguiente en el terminal:

   > Para iniciar sesión, use un explorador web para abrir la página https://microsoft.com/devicelogin y escriba el código FGB56WJUGK para autenticarse.

   Una vez que se autentica en el explorador, el script continúa ejecutándose.

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Cambie el primer parámetro a `index.js` y cambie la consulta para realizar la operación `order by` primero en la propiedad **Nombre** y, luego, para realizar la operación `limit` a los cinco primeros resultados. Reemplace `<YOUR_SUBSCRIPTION_ID_LIST>` por el identificador de suscripción:

   ```bash
   node index.js --query "Resources | project name, type | order by name asc | limit 5" --subs "<YOUR_SUBSCRIPTION_ID_LIST>"
   ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar las bibliotecas instaladas de la aplicación, ejecute el siguiente comando.

```bash
npm uninstall @azure/arm-resourcegraph @azure/ms-rest-nodeauth yargs
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha agregado las bibliotecas de Resource Graph al entorno de JavaScript y ha ejecutado su primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)