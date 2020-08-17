---
title: Localización de Azure Media Player
description: Compatibilidad con varios idiomas para usuarios de configuraciones regionales que no estén en inglés.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 04/20/2020
ms.openlocfilehash: 8a459d1fba4353b4b3e092e83a759314cc455ead
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087295"
---
# <a name="azure-media-player-localization"></a>Localización de Azure Media Player #

La compatibilidad con varios idiomas permite a los usuarios de configuraciones regionales distintas del inglés interactuar de forma nativa con el reproductor. Azure Media Player creará una instancia con un diccionario global de idiomas, que localizará los mensajes de error basados en el idioma de la página. Un desarrollador también puede crear una instancia del reproductor con un idioma establecido. El idioma predeterminado es el inglés (en).

> [!NOTE]
> Esta característica está todavía en pruebas y, como tal, está sujeta a errores.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin" data-setup='{"language":"es"}'>...</video>
```

Actualmente, Azure Media Player admite los siguientes idiomas con sus códigos de idioma correspondientes:

| Idioma            | Código | Idioma                | Código   | Idioma                | Código         |
|---------------------|------|-------------------------|--------|-------------------------|--------------|
| Inglés {predeterminado}   | en   | Croata                | hr     | Rumano                | ro           |
| Árabe              | ar   | Húngaro               | hu     | Eslovaco                  | sk           |
| Búlgaro           | bg   | Indonesio              | id     | Esloveno                 | sl           |
| Catalán             | ca   | Islandés               | is     | Serbio (cirílico)      | sr-cyrl-cs   |
| Checo               | cs   | Italiano                 | it     | Serbio (latino)         | sr-latn-rs   |
| Danés              | da   | Japonés                | ja     | Ruso                 | ru           |
| Alemán              | de   | Kazajo                  | kk     | Sueco                 | sv           |
| Griego               | el   | Coreano                  | ko     | Tailandés                    | th           |
| Español             | es   | Lituano              | lt     | Tagalo                 | tl           |
| Estonio            | et   | Letón                 | lv     | Turco                 | tr           |
| Vasco              | eu   | Malayo               | ms     | Ucraniano               | uk           |
| Persa               | fa   | Noruego (bokmal)     | nb     | Urdu                    | ur           |
| Finés             | fi   | Neerlandés                   | nl     | Vietnamita              | vi           |
| Francés              | fr   | Noruego (Nynorsk)     | nn     | Chino (simplificado)    | zh-hans      |
| Gallego            | gl   | Polaco                  | pl     | Chino (tradicional)   | zh-hant      |
| Hebreo              | he   | Portugués (Brasil)     | pt-br  |                         |              |
| Hindi               | hi   | Portugués (Portugal)   | pt-pt  |                         |              |


> [!NOTE]
> Si no desea que se produzca ninguna localización, debe aplicar el idioma inglés.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)
