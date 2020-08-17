---
title: 'Inicio rápido: Su primera consulta de Ruby'
description: En este inicio rápido, dará los pasos necesarios para habilitar la gema de Resource Graph para Ruby y ejecutará la primera consulta.
ms.date: 07/12/2020
ms.topic: quickstart
ms.openlocfilehash: 9763e2e9ec49d6fb4ea37fac12578ab23b7b5363
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87101143"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-ruby"></a>Inicio rápido: Ejecución de la primera consulta de Resource Graph con Ruby

El primer paso para usar Azure Resource Graph es comprobar que están instaladas las gemas necesarias para Ruby. Este inicio rápido lo guiará a través del proceso de agregar las gemas a la instalación de Ruby.

Cuando finalice, habrá agregado las gemas a la instalación de Ruby y ejecutado la primera consulta de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Una entidad de servicio de Azure, que incluya _clientId_ y _clientSecret_.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-graph-project"></a>Creación del proyecto de Resource Graph

Para que Ruby pueda consultar Azure Resource Graph, se debe agregar la gema a `Gemfile`. Esta gema funciona siempre que se pueda usar Ruby, lo cual incluye el uso con [Azure Cloud Shell](https://shell.azure.com), [bash en Windows 10](/windows/wsl/install-win10) o instalado localmente.

1. Compruebe que está instalada la versión más reciente de Ruby (al menos la **2.7.1**). Si aún no está instalada, descárguela en [Ruby-Lang.org](https://www.ruby-lang.org/en/downloads/).

1. En el entorno de Ruby que prefiera, inicialice una agrupación en una nueva carpeta de proyecto:

   ```bash
   # Initialize a bundle to create a new Gemfile
   bundle init
   ```

1. Actualice su `Gemfile` con las gemas necesarias de Azure Resource Graph. El archivo actualizado debe ser parecido al del ejemplo siguiente:

   ```file
   # frozen_string_literal: true

   source "https://rubygems.org"

   git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

   # gem "rails"
   gem 'azure_mgmt_resourcegraph', '~> 0.17.2'
   ```

1. En la carpeta del proyecto, ejecute `bundle install`. Confirme que las gemas se instalaron con `bundle list`.

1. En la misma carpeta de proyecto, cree `argQuery.rb` con el código siguiente y guarde el archivo actualizado:

   ```ruby
   #!/usr/bin/env ruby

   require 'azure_mgmt_resourcegraph'
   ARG = Azure::ResourceGraph::Profiles::Latest::Mgmt

   # Get arguments and set options
   options = {
       tenant_id: ARGV[0],
       client_id: ARGV[1],
       client_secret: ARGV[2],
       subscription_id: ARGV[3]
   }

   # Create Resource Graph client from options
   argClient = ARG::Client.new(options)

   # Create Resource Graph QueryRequest for subscription with query
   query_request = ARGModels::QueryRequest.new.tap do |qr|
       qr.subscriptions = [ARGV[3]]
       qr.query = ARGV[4]
   end

   # Get the resources from Resource Graph
   response = argClient.resources(query_request)

   # Convert data to JSON and output
   puts response.data.to_json
   ```

## <a name="run-your-first-resource-graph-query"></a>Ejecutar la primera consulta de Resource Graph

Con el script de Ruby guardado y listo para usar, es el momento de probar una consulta simple de Resource Graph. La consulta devolverá los cinco primeros recursos de Azure con el **nombre** y el **tipo de recurso** de cada recurso.

En cada llamada a `argQuery`, hay variables que se usan que debe reemplazar por sus propios valores:

- `{tenantId}`: reemplácelo por su identificador de inquilino
- `{clientId}`: reemplácelo por el identificador de cliente de la entidad de servicio
- `{clientSecret}`: reemplácelo por el secreto de cliente de la entidad de servicio
- `{subscriptionId}`: reemplácelo por el identificador de suscripción

1. Cambie los directorios a la carpeta de proyecto en la que creó los archivos `Gemfile` y `argClient.rb`.

1. Ejecute la primera consulta de Azure Resource Graph mediante las gemas y el método `resources`:

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Cambie el parámetro final a `argQuery.rb` y cambie la consulta a `order by` en la propiedad **Nombre**:

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Cambie el parámetro final a `argQuery.rb` y cambie la consulta para realizar la operación `order by` primero en la propiedad **Nombre** y, luego, para realizar la operación `limit` a los cinco primeros resultados:

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | order by name asc | limit 5"
   ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar las gemas instaladas del entorno de Ruby, puede hacerlo mediante el comando siguiente:

```bash
# Remove the installed gems from the Ruby environment
gem uninstall azure_mgmt_resourcegraph
```

> [!NOTE]
> La gema `azure_mgmt_resourcegraph` tiene dependencias como `ms_rest` y `ms_rest_azure` que se pueden haber instalado también según su entorno. También puede desinstalar estas gemas si ya no las necesita.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha agregado las gemas de Resource Graph al entorno de Ruby y ha ejecutado la primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)
