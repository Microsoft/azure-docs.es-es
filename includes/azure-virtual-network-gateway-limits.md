---
title: archivo de inclusión
description: archivo de inclusión
services: networking
author: anzaman
ms.service: VPN Gateway
ms.topic: include
ms.date: 08/25/2020
ms.author: alzam
ms.custom: include file
ms.openlocfilehash: 01796985a05f88b59786be7bbe0b06907cf0bc25
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854094"
---
| Resource                                | Límite        |
|-----------------------------------------|------------------------------|
| Prefijos de dirección de red virtual                   | 600 por puerta de enlace de VPN          |
| Agregar rutas BGP                    | 4000 por puerta de enlace de VPN        |
| Prefijos de direcciones de puerta de enlace de red local  | 1000 por puerta de enlace de red local               |
| de conexiones de sitio a sitio                         | [Depende de la SKU de puerta de enlace](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku)|
| Conexiones de punto a sitio                         | [Depende de la SKU de puerta de enlace](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku) |
| Límite de ruta de P2S: IKEv2                 | 256 si no es de Windows **/** 25 si es de Windows           |
| Límite de ruta de P2S: OpenVPN               | 1000                         |
| Máx. flows                              | 100 000 para VpnGw1/AZ  **/** 512 000 para VpnGw2-4/AZ|