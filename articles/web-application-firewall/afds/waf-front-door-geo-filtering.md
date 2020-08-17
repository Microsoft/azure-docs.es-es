---
title: Filtrado geográfico en un dominio para Azure Front Door Service
description: En este artículo, obtendrá información acerca de la directiva de filtrado geográfico para Azure Front Door Service
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: fcd7a0fe60639bbb17661a906d15136996b325e4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005453"
---
# <a name="what-is-geo-filtering-on-a-domain-for-azure-front-door-service"></a>¿Qué es el filtrado geográfico en un dominio para Azure Front Door Service?

De forma predeterminada, Azure Front Door Service responde a las solicitudes del usuario independientemente de la ubicación del usuario que realiza la solicitud. Sin embargo, puede que en algunos casos desee restringir el acceso a las aplicaciones web por país o región. El servicio de firewall de aplicaciones web (WAF) en Front Door le permite definir una directiva que usa reglas de acceso personalizadas en una ruta de acceso concreta de su punto de conexión para permitir o bloquear el acceso desde determinados países o regiones. 

Habitualmente, una directiva de WAF incluye un conjunto de reglas personalizadas. Las reglas constan de condiciones de coincidencia, acciones y prioridades. En las condiciones de coincidencia, se definen una variable de coincidencia, un operador y un valor de coincidencia.  En el caso de la regla de filtrado geográfico, la variable de coincidencia es REMOTE_ADDR, el operador es GeoMatch y el valor es el código de país o región de dos letras de interés. Puede combinar una condición GeoMatch y una condición de coincidencia de la cadena REQUEST_URI para crear una regla de filtrado geográfico basada en ruta de acceso.

Las directivas de filtrado geográfico para Front Door se pueden configurar mediante [Azure PowerShell](waf-front-door-tutorial-geo-filtering.md) o nuestra [plantilla de inicio rápido](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering).

## <a name="countryregion-code-reference"></a>Referencia de código de país o región

|Código de país o región | Nombre de país o región |
| ----- | ----- |
| AD | Andorra |
| AE | Emiratos Árabes Unidos|
| AF | Afganistán|
| AG | Antigua y Barbuda|
| AL | Albania|
| AM | Armenia|
| AO | Angola|
| AR | Argentina|
| AS | Samoa Americana|
| AT | Austria|
| AU | Australia|
| AZ | Azerbaiyán|
| BA | Bosnia y Herzegovina|
| BB | Barbados|
| BD | Bangladés|
| BE | Bélgica|
| BF | Burkina Faso|
| BG | Bulgaria|
| BH | Bahréin|
| BI | Burundi|
| BJ | Benín|
| BL | San Bartolomé|
| BN | Brunéi Darussalam|
| BO | Bolivia|
| BR | Brasil|
| BS | Bahamas|
| BT | Bután|
| BW | Botsuana|
| BY | Belarús|
| BZ | Belice|
| CA | Canadá|
| CD | República Democrática del Congo|
| CF | República Centroafricana|
| CH | Suiza|
| CI | República de Côte d’Ivoire|
| CL | Chile|
| CM | Camerún|
| CN | China|
| CO | Colombia|
| CR | Costa Rica|
| CU | Cuba|
| CV | Cabo Verde|
| CY | Chipre|
| CZ | República Checa|
| DE | Alemania|
| DK | Dinamarca|
| DO | República Dominicana|
| DZ | Argelia|
| EC | Ecuador|
| EE | Estonia|
| EG | Egipto|
| ES | España|
| ET | Etiopía|
| FI | Finlandia|
| FJ | Islas Fiji|
| FM | Micronesia, Estados Federados de|
| VF | Francia|
| GB | Reino Unido|
| GE | Georgia|
| GF | Guayana Francesa|
| GH | Ghana|
| GN | Guinea|
| GP | Guadalupe|
| GR | Grecia|
| GT | Guatemala|
| GY | Guayana|
| HK | RAE de Hong Kong|
| HN | Honduras|
| HR | Croacia|
| HT | Haití|
| HU | Hungría|
| id | Indonesia|
| IE | Irlanda|
| IL | Israel|
| IN | India|
| IQ | Iraq|
| IR | Irán, República Islámica de|
| IS | Islandia|
| IT | Italia|
| JM | Jamaica|
| JO | Jordania|
| JP | Japón|
| KE | Kenia|
| KG | Kirguistán|
| KH | Camboya|
| KI | Kiribati|
| KN | San Cristóbal y Nieves|
| KP | Corea, República Popular Democrática de|
| KR | Corea, República de|
| KW | Kuwait|
| KY | Islas Caimán|
| KZ | Kazajistán|
| Los Ángeles | República Democrática Popular de Laos|
| LB | Líbano|
| LI | Liechtenstein|
| LK | Sri Lanka|
| LR | Liberia|
| LS | Lesoto|
| LT | Lituania|
| LU | Luxemburgo|
| LV | Letonia|
| LY | Libia |
| MA | Marruecos|
| MD | Moldavia, República de|
| MG | Madagascar|
| MK | Macedonia del Norte|
| ML | Malí|
| MM | Myanmar|
| MN | Mongolia|
| MO | RAE de Macao|
| MQ | Martinica|
| MR | Mauritania|
| MT | Malta|
| MV | Maldivas|
| MW | Malaui|
| MX | México|
| MY | Malasia|
| MZ | Mozambique|
| N/D | Namibia|
| NE | Níger|
| NG | Nigeria|
| NI | Nicaragua|
| NL | Países Bajos|
| No | Noruega|
| NP | Nepal|
| NR | Nauru|
| NZ | Nueva Zelanda|
| OM | Omán|
| PA | Panamá|
| PE | Perú|
| PH | Filipinas|
| PK | Pakistán|
| PL | Polonia|
| PR | Puerto Rico|
| PT | Portugal|
| PW | Palaos|
| PY | Paraguay|
| QA | Qatar|
| RE | Reunión|
| RO | Rumanía|
| RS | Serbia|
| RU | Federación Rusa|
| RW | Ruanda|
| SA | Arabia Saudí|
| SD | Sudán|
| SE | Suecia|
| SG | Singapur|
| SI | Eslovenia|
| SK | Eslovaquia|
| SN | Senegal|
| SO | Somalia|
| SR | Surinam|
| SS | Sudán del Sur|
| SV | El Salvador|
| SY | República Árabe Siria|
| SZ | Suazilandia|
| TC | Islas Turcas y Caicos|
| TG | Togo|
| TH | Tailandia|
| TN | Túnez|
| TR | Turquía|
| TT | Trinidad y Tobago|
| TW | Taiwán|
| TZ | Tanzania, República Unida de|
| UA | Ucrania|
| UG | Uganda|
| US | Estados Unidos|
| UY | Uruguay|
| UZ | Uzbekistán|
| VC | San Vicente y las Granadinas|
| VE | Venezuela|
| VG | Islas Vírgenes Británicas|
| VI | Islas Vírgenes de EE. UU.|
| VN | Vietnam|
| ZA | Sudáfrica|
| ZM | Zambia|
| ZW | Zimbabue|

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de la [seguridad de la capa de aplicaciones con Front Door](../../frontdoor/front-door-application-security.md).
- Aprenda a [crear una instancia de Front Door](../../frontdoor/quickstart-create-front-door.md).
