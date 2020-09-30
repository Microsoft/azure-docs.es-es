---
title: Configuración de marcos de servidores front-end con Azure Static Web Apps versión preliminar
description: Configuración de los marcos de servidores front-end populares necesarios para Azure Static Web Apps
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 07/18/2020
ms.author: cshoe
ms.openlocfilehash: 4b1bc58b6b4a87cd6e5e09e83020a38261b8746f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905374"
---
# <a name="configure-front-end-frameworks-and-libraries-with-azure-static-web-apps-preview"></a>Configuración de marcos y bibliotecas de servidores front-end con Azure Static Web Apps versión preliminar

Azure Static Web Apps requiere que tenga los valores de configuración adecuados en el [archivo de configuración de compilación](github-actions-workflow.md) para el marco o la biblioteca del servidor front-end.

## <a name="configuration"></a>Configuración

En la siguiente tabla se muestra la configuración de varios marcos y bibliotecas<sup>1</sup>.

Los siguientes elementos explican la intención de las columnas de la tabla:

- **Ubicación del artefacto de la aplicación**: Muestra el valor de `app_artifact_location`, que es la [carpeta para las versiones compiladas de los archivos de aplicación](github-actions-workflow.md#build-and-deploy).

- **Comando de compilación personalizado**: Cuando el marco de trabajo requiere un comando distinto de `npm run build` o `npm run azure:build`, puede definir un [comando de compilación personalizado](github-actions-workflow.md#custom-build-commands).

| marco | Ubicación del artefacto de la aplicación | Comando de compilación personalizado |
|--|--|--|
| [Alpine.js](https://github.com/alpinejs/alpine/) | `/` | n/d <sup>2</sup> |
| [Angular](https://angular.io/) | `dist/<APP_NAME>` | `npm run build -- --prod` |
| [Angular Universal](https://angular.io/guide/universal) | `dist/<APP_NAME>/browser` | `npm run prerender` |
| [Aurelia](https://aurelia.io/) | `dist` | N/D |
| [Backbone.js](https://backbonejs.org/) | `/` | N/D |
| [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) | `wwwroot` | N/D |
| [Ember](https://emberjs.com/) | `dist` | N/D |
| [Flutter](https://flutter.dev/) | `build/web` | `flutter build web` |
| [Framework7](https://framework7.io/) | `www` | `npm run build-prod` |
| [Glimmer](https://glimmerjs.com/) | `dist` | N/D |
| [HTML](https://developer.mozilla.org/docs/Web/HTML) | `/` | N/D |
| [Hyperapp](https://hyperapp.dev/) | `/` | N/D |
| [JavaScript](https://developer.mozilla.org/docs/Web/javascript) | `/` | N/D |
| [jQuery](https://jquery.com/) | `/` | N/D |
| [KnockoutJS](https://knockoutjs.com/) | `dist` | N/D |
| [LitElement](https://lit-element.polymer-project.org/) | `dist` | N/D |
| [Marko](https://markojs.com/) | `public` | N/D |
| [Meteor](https://www.meteor.com/) | `bundle` | N/D |
| [Mithril](https://mithril.js.org/) | `dist` | N/D |
| [Polymer](https://www.polymer-project.org/) | `build/default` | N/D |
| [Preact](https://preactjs.com/) | `build` | N/D |
| [React](https://reactjs.org/) | `build` | N/D |
| [Stencil](https://stenciljs.com/) | `www` | N/D |
| [Svelte](https://svelte.dev/) | `public` | N/D |
| [Three.js](https://threejs.org/) | `/` | N/D |
| [TypeScript](https://www.typescriptlang.org/) | `dist` | N/D |
| [Vue.js](https://vuejs.org/) | `dist` | N/D |

<sup>1</sup> La tabla anterior no pretende ser una lista exhaustiva de marcos de trabajo y bibliotecas que funcionen con Azure Static Web Apps.

<sup>2</sup> No aplicable

## <a name="next-steps"></a>Pasos siguientes

- [Creación y configuración de flujos de trabajo](github-actions-workflow.md)
