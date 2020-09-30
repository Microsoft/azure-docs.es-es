---
title: Rotación de certificados para servidor único de Azure Database for PostgreSQL
description: Obtenga información sobre los próximos cambios del certificado raíz que afectarán al servidor único de Azure Database for PostgreSQL.
author: kummanish
ms.author: manishku
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 5b46dc0197022aa72b6ca4c206d1da4369234167
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90707686"
---
# <a name="understanding-the-changes-in-the-root-ca-change-for-azure-database-for-postgresql-single-server"></a>Descripción de los cambios en la CA raíz para servidor único de Azure Database for PostgreSQL

Azure Database for PostgreSQL cambiará el certificado raíz del controlador o la aplicación cliente habilitado con SSL, que se usa para [conectarse al servidor de base de datos](concepts-connectivity-architecture.md). El certificado raíz disponible actualmente está configurado para expirar el 26 de octubre de 2020 (26/10/2020) como parte de los procedimientos recomendados de seguridad y mantenimiento estándar. En este artículo se proporcionan más detalles sobre los próximos cambios, los recursos que se verán afectados y los pasos necesarios para asegurarse de que la aplicación mantenga la conectividad con el servidor de base de datos.

## <a name="what-update-is-going-to-happen"></a>¿Qué actualización va a producirse?

En algunos casos, las aplicaciones usan un archivo de certificado local generado a partir de un archivo de certificado de una entidad de certificación (CA) de confianza para conectarse de forma segura. Actualmente, los clientes solo pueden usar el certificado predefinido para conectarse a un servidor de Azure Database for PostgreSQL que se encuentra [aquí](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem). Sin embargo, [Certificate Authority (CA) Browser Forum](https://cabforum.org/) recientemente publicó informes relativos a que varios certificados emitidos por proveedores de CA no son compatibles.

Según los requisitos de cumplimiento del sector, los proveedores de CA comenzaron a revocar los certificados de CA para las CA no compatibles, lo que requiere que los servidores usen certificados emitidos por CA compatibles y firmados por certificados de CA de esas CA compatibles. Como actualmente Azure Database for PostgreSQL usa uno de estos certificados no compatibles, que las aplicaciones cliente usan para validar sus conexiones SSL, es necesario asegurarse de que se adopten las acciones adecuadas (descritas a continuación) para minimizar el posible impacto en los servidores de PostgreSQL.

El nuevo certificado se utilizará a partir del 26 de octubre de 2020 (26/10/2020). Si usa la validación de CA o la validación completa del certificado de servidor al conectarse desde un cliente de PostgreSQL (sslmode=verify-ca o sslmode=verify-full), debe actualizar la configuración de la aplicación antes del 26 de octubre de 2020 (26/10/2020).

## <a name="how-do-i-know-if-my-database-is-going-to-be-affected"></a>¿Cómo sé si mi base de datos se verá afectada?

Todas las aplicaciones que usan SSL/TLS y comprueban el certificado raíz tienen que actualizar el certificado raíz. Para saber si las conexiones comprueban el certificado raíz, puede revisar la cadena de conexión.
-   Si la cadena de conexión incluye `sslmode=verify-ca` o `sslmode=verify-full`, tiene que actualizar el certificado.
-   Si la cadena de conexión incluye `sslmode=disable`, `sslmode=allow`, `sslmode=prefer` o `sslmode=require`, no tiene que actualizar los certificados. 
-   Si la cadena de conexión no especifica sslmode, no tiene que actualizar los certificados.

Si utiliza un cliente que abstrae la cadena de conexión, revise la documentación del cliente para saber si comprueba los certificados.

Para entender sslmode de PostgreSQL, consulte las [descripciones del modo SSL](https://www.postgresql.org/docs/11/libpq-ssl.html#ssl-mode-descriptions) en la documentación de PostgreSQL.

Para evitar que se interrumpa la disponibilidad de la aplicación debido a la revocación inesperada de certificados, o para actualizar un certificado que se ha revocado, consulte la sección [ **"¿Qué tengo que hacer para mantener la conectividad?"** ](concepts-certificate-rotation.md#what-do-i-need-to-do-to-maintain-connectivity).

## <a name="what-do-i-need-to-do-to-maintain-connectivity"></a>¿Qué tengo que hacer para mantener la conectividad?

Para evitar que se interrumpa la disponibilidad de la aplicación debido a la revocación inesperada de certificados, o para actualizar un certificado que se ha revocado, siga los pasos a continuación. La idea es crear un nuevo archivo *.pem*, que combina el certificado actual y el nuevo y, durante la validación del certificado SSL, se usará uno de los valores permitidos. Consulte los pasos siguientes:

*   Descargue la CA raíz de BaltimoreCyberTrustRoot y DigiCertGlobalRootG2 de los vínculos siguientes:
    *   https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem
    *   https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem

*   Genere un almacén de certificados de CA combinado que incluya los certificados tanto de **BaltimoreCyberTrustRoot** como de **DigiCertGlobalRootG2**.
    *   Para los usuarios de Java (PostgreSQL JDBC) que usan DefaultJavaSSLFactory, se debe ejecute:

          ```azurecli-interactive
          keytool -importcert -alias PostgreSQLServerCACert  -file D:\BaltimoreCyberTrustRoot.crt.pem  -keystore truststore -storepass password -noprompt
          ```

          ```azurecli-interactive
          keytool -importcert -alias PostgreSQLServerCACert2  -file D:\DigiCertGlobalRootG2.crt.pem -keystore truststore -storepass password  -noprompt
          ```

          A continuación, reemplace el archivo de almacén de claves original por el nuevo generado:
        *   System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file"); 
        *   System.setProperty("javax.net.ssl.trustStorePassword","password");
        
    *   En el caso de un usuario de .NET (Npgsql) en Windows, debe asegurarse de que **Baltimore CyberTrustRoot** y **DigiCertGlobalRootG2** existan en el almacén de certificados de Windows, entidades de certificación raíz de confianza. Si no existe alguno de los certificados, importe el certificado que falta.

        ![Certificado .Net para Azure Database for PostgreSQL](media/overview/netconnecter-cert.png)

    *   En el caso de un usuario de .NET (Npgsql) en Linux con SSL_CERT_DIR, debe asegurarse de que **BaltimoreCyberTrustRoot** y **DigiCertGlobalRootG2** existan en el directorio indicado por SSL_CERT_DIR. Si no existe alguno de los certificados, cree el archivo de certificado que falta.

    *   Para otros usuarios de cliente de PostgreSQL, se pueden fusionar dos archivos de certificado de CA con el formato siguiente.

        </br>-----BEGIN CERTIFICATE-----
 </br>(Root CA1: BaltimoreCyberTrustRoot.crt.pem)
 </br>-----END CERTIFICATE-----
 </br>-----BEGIN CERTIFICATE-----
 </br>(Root CA2: DigiCertGlobalRootG2.crt.pem)
 </br>-----END CERTIFICATE-----

*   Reemplace el archivo PEM de la CA raíz original por el archivo de CA raíz combinado y reinicie la aplicación o el cliente.
*   En el futuro, después de que se implemente el nuevo certificado en el servidor, puede cambiar el archivo PEM de la CA a DigiCertGlobalRootG2.crt.pem.

## <a name="what-can-be-the-impact-of-not-updating-the-certificate"></a>¿Cuál puede ser el impacto de no actualizar el certificado?
Si usa el certificado de BaltimoreCyberTrustRoot para comprobar la conexión SSL a Azure Database for PostgreSQL tal y como se documenta aquí, la disponibilidad de la aplicación puede verse interrumpida, ya que no se podrá acceder a la base de datos. Según la aplicación, puede recibir distintos mensajes de error, entre los que se incluyen los siguientes:
*   Certificado no válido o certificado revocado
*   Se agotó el tiempo de espera de la conexión

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="1-if-i-am-not-using-ssltls-do-i-still-need-to-update-the-root-ca"></a>1. Si no utilizo SSL/TLS, ¿aún tengo que actualizar la CA raíz?
No se requiere ninguna acción si no se usa SSL/TLS. 

### <a name="2-if-i-am-using-ssltls-do-i-need-to-restart-my-database-server-to-update-the-root-ca"></a>2. Si uso SSL/TLS, ¿es necesario reiniciar el servidor de base de datos para actualizar la CA raíz?
No, no es necesario reiniciar el servidor de base de datos para empezar a usar el nuevo certificado. Se trata de un cambio en el lado cliente y las conexiones de cliente entrantes deben usar el nuevo certificado para asegurarse de que puedan conectarse al servidor de bases de datos.

### <a name="3-what-will-happen-if-i-do-not-update-the-root-certificate-before-october-26-2020-10262020"></a>3. ¿Qué sucederá si no actualizo el certificado raíz antes del 26 de octubre de 2020 (26/10/2020)?
Si no actualiza el certificado raíz antes del 26 de octubre de 2020, las aplicaciones que se conecten a través de SSL/TLS y comprueben el certificado raíz no podrán comunicarse con el servidor de base de datos PostgreSQL, y la aplicación experimentará problemas de conectividad con el servidor de base de datos PostgreSQL.

### <a name="4-what-is-the-impact-if-using-app-service-with-azure-database-for-postgresql"></a>4. ¿Qué consecuencia tendrá si se usa App Service con Azure Database for PostgreSQL?
En el caso de los servicios de aplicaciones de Azure que se conectan a Azure Database for PostgreSQL, podemos tener dos escenarios posibles y depende de cómo se use SSL con la aplicación.
*   Este nuevo certificado se ha agregado a App Service en el nivel de la plataforma. Si usa en la aplicación los certificados SSL incluidos en la plataforma de App Service, no es necesario realizar ninguna acción.
*   Si va a incluir explícitamente la ruta de acceso al archivo de certificado SSL en el código, debería descargar el nuevo certificado y actualizar el código para usar el nuevo certificado.

### <a name="5-what-is-the-impact-if-using-azure-kubernetes-services-aks-with-azure-database-for-postgresql"></a>5. ¿Qué consecuencia tendrá si se usa Azure Kubernetes Service (AKS) con Azure Database for PostgreSQL?
Si está intentando conectarse a Azure Database for PostgreSQL con Azure Kubernetes Services (AKS), es similar a acceder desde un entorno de host de clientes dedicados. Consulte los pasos [aquí](../aks/ingress-own-tls.md).

### <a name="6-what-is-the-impact-if-using-azure-data-factory-to-connect-to-azure-database-for-postgresql"></a>6. ¿Qué consecuencia tendrá si se usa Azure Data Factory para conectarse con Azure Database for PostgreSQL?
Para el conector que usa Azure Integration Runtime, el conector aprovecha los certificados del almacén de certificados de Windows en el entorno hospedado en Azure. Estos certificados ya son compatibles con los certificados recién aplicados y, por lo tanto, no es necesaria ninguna acción.

Para el conector que usa el entorno de ejecución de integración autohospedado donde se incluye explícitamente la ruta de acceso al archivo de certificado SSL en la cadena de conexión, deberá descargar el [nuevo certificado](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) y actualizar la cadena de conexión para utilizarlo.

### <a name="7-do-i-need-to-plan-a-database-server-maintenance-downtime-for-this-change"></a>7. ¿Es necesario planear un tiempo de inactividad de mantenimiento del servidor de base de datos para este cambio?
No. Dado que el cambio aquí solo está en el lado del cliente para conectarse al servidor de base de datos, este cambio no requiere ningún tiempo de inactividad de mantenimiento para el servidor de base de datos.

### <a name="8--what-if-i-cannot-get-a-scheduled-downtime-for-this-change-before-october-26-2020-10262020"></a>8.  ¿Qué ocurre si no puedo obtener un tiempo de inactividad programado para este cambio antes del 26 de octubre de 2020 (26/10/2020)?
Dado que los clientes usados para conectarse al servidor deben actualizar la información del certificado tal y como se describe en la sección de corrección [aquí](./concepts-certificate-rotation.md#what-do-i-need-to-do-to-maintain-connectivity), no es necesario un tiempo de inactividad para el servidor en este caso.

### <a name="9-if-i-create-a-new-server-after-october-26-2020-will-i-be-impacted"></a>9. Si creo un nuevo servidor después del 26 de octubre de 2020, ¿me veré afectado?
En el caso de los servidores creados después del 26 de octubre de 2020 (26/10/2020), puede usar el certificado recién emitido para que las aplicaciones se conecten mediante SSL.

### <a name="10-how-often-does-microsoft-update-their-certificates-or-what-is-the-expiry-policy"></a>10. ¿Con qué frecuencia actualiza Microsoft sus certificados o cuál es la directiva de expiración?
Los certificados utilizados por Azure Database for PostgreSQL provienen de entidades de certificación (CA) de confianza. Por lo tanto, la compatibilidad de estos certificados en Azure Database for PostgreSQL está ligada a la compatibilidad de estos certificados por parte de la CA. Sin embargo, como en este caso, puede haber errores imprevistos en estos certificados predefinidos, que deben corregirse lo antes posible.

### <a name="11-if-i-am-using-read-replicas-do-i-need-to-perform-this-update-only-on-master-server-or-the-read-replicas"></a>11. Si utilizo réplicas de lectura, ¿tengo que realizar esta actualización solo en el servidor maestro o en las réplicas de lectura?
Puesto que esta actualización es un cambio en el lado cliente, si el cliente leía los datos del servidor réplica, deberá aplicar también los cambios para esos clientes. 

### <a name="12-do-we-have-server-side-query-to-verify-if-ssl-is-being-used"></a>12. ¿Tenemos que consultar en el lado servidor para comprobar si se está utilizando SSL?
Para comprobar si está usando la conexión SSL para conectarse al servidor, consulte [Comprobación de la conexión SSL](concepts-ssl-connection-security.md#applications-that-require-certificate-verification-for-tls-connectivity).

### <a name="13-what-if-i-have-further-questions"></a>13. ¿Qué hago si tengo más preguntas?
Si tiene alguna pregunta, obtenga respuestas de expertos de la comunidad en [Microsoft Q&A](mailto:AzureDatabaseforPostgreSQL@service.microsoft.com). Si tiene un plan de soporte técnico y necesita ayuda técnica, [póngase en contacto con nosotros](mailto:AzureDatabaseforPostgreSQL@service.microsoft.com)