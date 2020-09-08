---
title: Definición de esquemas de la aplicación
description: La aplicación de LUIS se representa en formato `.json` o `.lu` e incluye todas las intenciones, las entidades, las expresiones de ejemplo, las características y la configuración.
ms.topic: reference
ms.date: 08/22/2020
ms.openlocfilehash: 816a6c50129f37a55ab3dba72319358e832a6b8b
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88756795"
---
# <a name="app-schema-definition"></a>Definición del esquema de la aplicación

La aplicación de LUIS se representa en formato `.json` o `.lu` e incluye todas las intenciones, las entidades, las expresiones de ejemplo, las características y la configuración.

## <a name="format"></a>Formato

Cuando importe y exporte la aplicación, elija `.json` o `.lu`.

|Formato|Information|
|--|--|
|`.json`| Interfaz de programación estándar|
|`.lu`|Compatible con las [herramientas de Bot Builder](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) de Bot Framework.|

## <a name="version-7x"></a>Version 7.x

* Al pasar a la versión 7.x, las entidades se representan como entidades anidadas de aprendizaje automático.
* Compatibilidad para la creación de entidades anidadas de aprendizaje automático con la propiedad `enableNestedChildren` en las siguientes API de creación:
    * [Agregar etiqueta](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c08)
    * [Agregar etiqueta de lote](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c09)
    * [Revisar etiquetas](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c0a)
    * [Sugerir consultas de punto de conexión para entidades](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2e)
    * [Sugerir consultas de punto de conexión para intenciones](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2d)

```json
{
  "luis_schema_version": "7.0.0",
  "intents": [
    {
      "name": "None",
      "features": []
    }
  ],
  "entities": [],
  "hierarchicals": [],
  "composites": [],
  "closedLists": [],
  "prebuiltEntities": [],
  "utterances": [],
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "patternAnyEntities": [],
  "regex_entities": [],
  "phraselists": [
  ],
  "regex_features": [],
  "patterns": [],
  "settings": []
}
```

| elemento                  | Comentario                              |
|--------------------------|--------------------------------------|
| "hierarchicals": [],     | En desuso, use [entidades de aprendizaje automático](luis-concept-entity-types.md).   |
| "composites": [],        | En desuso, use [entidades de aprendizaje automático](luis-concept-entity-types.md). Referencia de la [entidad compuesta](reference-entity-composite.md). |
| "closedLists": [],       | Referencia de la [entidad de lista](reference-entity-list.md), que se usa principalmente como característica para las entidades.    |
| "versionId": "0.1",      | Versión de una aplicación de LUIS.|
| "name": "example-app",   | Nombre de la aplicación de LUIS. |
| "desc": "",              | Descripción opcional de la aplicación de LUIS.  |
| "culture": "en-us",      | El [lenguaje](luis-language-support.md) de la aplicación afecta a las características subyacentes, como las entidades creadas previamente, el aprendizaje automático y el tokenizador.  |
| "tokenizerVersion": "1.0.0", | [Tokenizador](luis-language-support.md#tokenization)  |
| "patternAnyEntities": [],   | [Entidad Pattern.any](reference-entity-pattern-any.md)    |
| "regex_entities": [],    |  [Entidad de expresión regular](reference-entity-regular-expression.md)   |
| "phraselists": [],       |  [Listas de frases (característica)](luis-concept-feature.md#create-a-phrase-list-for-a-concept)   |
| "regex_features": [],    |  En desuso, use [entidades de aprendizaje automático](luis-concept-entity-types.md). |
| "patterns": [],          |  Los [patrones mejoran la precisión de las predicciones](luis-concept-patterns.md) con la [sintaxis del patrón](reference-pattern-syntax.md).   |
| "settings": []           | [Configuración de aplicación](luis-reference-application-settings.md)|

## <a name="version-6x"></a>Versión 6.x

* Al pasar a la versión 6.x, use la nueva [entidad de aprendizaje automático](reference-entity-machine-learned-entity.md) para representar las entidades.

```json
{
  "luis_schema_version": "6.0.0",
  "intents": [
    {
      "name": "None",
      "features": []
    }
  ],
  "entities": [],
  "hierarchicals": [],
  "composites": [],
  "closedLists": [],
  "prebuiltEntities": [],
  "utterances": [],
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "patternAnyEntities": [],
  "regex_entities": [],
  "phraselists": [],
  "regex_features": [],
  "patterns": [],
  "settings": []
}
```

## <a name="version-4x"></a>Version 4.x

```json
{
  "luis_schema_version": "4.0.0",
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "intents": [
    {
      "name": "None"
    }
  ],
  "entities": [],
  "composites": [],
  "closedLists": [],
  "patternAnyEntities": [],
  "regex_entities": [],
  "prebuiltEntities": [],
  "model_features": [],
  "regex_features": [],
  "patterns": [],
  "utterances": [],
  "settings": []
}
```

## <a name="next-steps"></a>Pasos siguientes

* Migración a las [API de creación V3](luis-migration-authoring-entities.md)