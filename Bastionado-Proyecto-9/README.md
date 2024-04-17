
# Introducción

En la actualidad, la seguridad en línea es una preocupación fundamental para cualquier usuario de internet. Una de las principales herramientas para garantizar esta seguridad es el protocolo HTTPS, utilizado por la mayoría de las páginas web para cifrar la conexión entre el servidor y el cliente. Este protocolo se basa en el uso de certificados digitales, emitidos por Autoridades de Certificación confiables, que permiten la autenticación del sitio web y establecen una conexión segura.

El presente trabajo explora la importancia teórica y práctica de los certificados digitales en el contexto de la seguridad en línea. Se analiza cómo estos certificados son utilizados para asegurar la integridad y confidencialidad de la información transmitida, así como para prevenir ataques cibernéticos como el phishing y la suplantación de identidad. Además, se examina el impacto que tiene el uso de HTTPS en el posicionamiento de los sitios web en los motores de búsqueda.

En este proyecto vamos a poner en producción un servidor web(NGINX en mi caso), hosteado en Digital Ocean, usaré DuckDNS e instalaré el certificado con CertBot. Todo esto lo realizaré desde mi equipo personal que tiene como sistema operativo Ubuntu 22.04.4 LTS, y el despliegue del servidor web se realizará en un Ubuntu Server 20.

# Parte 1

En esta parte se presenta un gif que demuestra la correcta configuración de las claves ssh que me permiten iniciar sesión sin usar contraseña.
![](img/Parte%201.gif)

# Parte 2

Ahora que podemos iniciar sesión sin usar contraseña vamos a instalar el servidor web, crear nuestro dominio e instalar nuestro certificado SSL.

Comprobación de DNS:

![](Captura%20desde%202024-04-16%2014-25-46.png)

Comprobación de la instalación del servidor web:
![](Captura%20desde%202024-04-16%2014-27-56.png)

Proceso de configuración e instalación del certificado:

![](Captura%20desde%202024-04-16%2014-30-39.png)

![](Captura%20desde%202024-04-16%2014-32-05.png)

![](Captura%20desde%202024-04-16%2014-32-13.png)

Una pequeña simulación de renovación del certificado para confirmar la instalación del mismo:

![](Captura%20desde%202024-04-16%2014-33-09.png)

Comprobación de que el nuevo tráfico es HTTPS:

![](Captura%20desde%202024-04-16%2014-33-31.png)

## Comparación entre certificados

En las siguientes capturas presento información de los certificados de mi servidor y de otra elegido de forma arbitraria, en este caso genialy.io.

### Mi servidor

![](Captura%20desde%202024-04-16%2014-41-25.png)![](Captura%20desde%202024-04-16%2014-41-42.png)
### Servidor de genialy

![](Captura%20desde%202024-04-16%2014-43-35.png)
![](Captura%20desde%202024-04-16%2014-43-54.png)![](Captura%20desde%202024-04-16%2014-44-46.png)

Una de las principales diferencias que tenemos es el DNS CAA,  es un tipo de registro de recursos en el DNS que permite a los propietarios de dominios especificar qué autoridades de certificación (CAs) están autorizadas para emitir certificados digitales para ese dominio. Este registro ayuda a mejorar la seguridad en línea al limitar las CAs que pueden emitir certificados para un dominio específico, lo que reduce el riesgo de emisión indebida de certificados por parte de CAs no autorizadas. En el caso de mi dominio no tenemos ninguna y en el caso de Genialy tenemos varias, vemos algunos famosos como godaddy o amazon.

Otra diferencia la tenemos en el tipo de clave, mi servidor usa  mientras que Genialy usa claves RSA de 2048 bit(bastante más seguras). 

Vemos que el emitidor del certificado en el caso de Genialy es Amazon y en mi caso es R3 que referencia a el certificador que hemos usado "Let's encrypt", como se ve en el siguiente texto:

> Under normal circumstances, certificates issued by Let’s Encrypt will come from “R3”, an RSA intermediate. Currently, issuance from “E1”, an ECDSA intermediate, is possible only for ECDSA subscriber keys for [allowlisted accounts](https://community.letsencrypt.org/t/ecdsa-availability-in-production-environment/150679). In the future, issuance from “E1” will be available for everyone. 
> https://letsencrypt.org/certificates/#intermediate-certificates

Como es  obvio las fechas de creación y validez de ambos certificados no coinciden.

Otra de las principales diferencias que encuentro es la presencia de varios certificados adicionales por parte del servidor de Genialy como se ve en la tercera captura.

# Parte 3

Ahora vamos a comparar el servidor de Genialy con otros 3 certificados erróneos y entender porque pasa esto.

### Sobre el certificado de Genialy

La validez del certificado de Genialy se podría resumir en los siguientes puntos:

1. **Emisión por una CA confiable**: El certificado SSL está emitido por una CA reconocida y confiable, que sigue prácticas rigurosas de verificación de identidad y emisión de certificados.
2. **Coincidencia de dominio**: El certificado SSL está asociado con el dominio del sitio web al que se está accediendo.
3. **Fecha de vigencia activa**: Es muy importante que el certificado no sobrepase la fecha de vigencia del mismo.
4. **Firma digital válida**: Tiene firma digital válida, lo que demuestra que ha sido emitido por la CA indicada y no ha sido alterado desde su emisión.
5. **Cifrado adecuado**: El certificado SSL utiliza un algoritmo de cifrado bastante robusto.
6. **Certificados intermedios y de raíz confiables**: Los certificados SSL a menudo se emiten a través de una cadena de confianza que incluye certificados intermedios y de raíz. Estos certificados son confiables y reconocidos por los navegadores y sistemas operativos.
### Primer certificado erróneo(Firmado por el mismo)
![](2024-04-17_11-48.png)
![](2024-04-17_11-50.png)

No es de fiar porque carece de una validación independiente de una tercera parte de confianza, como una Autoridad de Certificación (CA, por sus siglas en inglés). Cuando un certificado SSL es emitido por una CA de confianza, significa que ha pasado por un proceso de verificación riguroso que confirma la identidad del propietario del dominio. Esto ayuda a garantizar a los usuarios que están interactuando con el sitio web legítimo y no con un sitio fraudulento.

En contraste, un certificado SSL autofirmado, es decir, firmado por el mismo dominio, no ha sido verificado por una CA externa. Por lo tanto, aunque el sitio web puede afirmar que tiene una conexión segura, los usuarios no tienen una garantía independiente de que la identidad del propietario del dominio sea auténtica.

### Segundo certificado erróneo(Revocado)

![](2024-04-17_12-17.png)
![](2024-04-17_11-50_2.png)
  
Un certificado SSL caducado no es seguro porque su fecha de vencimiento indica que la información de seguridad que proporciona ya no está actualizada ni garantizada. Los certificados SSL tienen una fecha de vencimiento establecida, después de la cual ya no se consideran válidos. Esto se debe a que la seguridad en línea es un campo en constante evolución, y las tecnologías y prácticas de seguridad pueden cambiar con el tiempo.

### Segundo certificado erróneo(Firmado por un proveedor no fiable)

![](2024-04-17_11-48_1.png)![](2024-04-17_11-50_1.png)

  
Los certificados SSL firmados por proveedores no fiables o no reconocidos presentan riesgos significativos para la seguridad en línea. Las Autoridades de Certificación (CA) son entidades confiables que emiten certificados SSL después de verificar la identidad del propietario del dominio. Sin embargo, los proveedores no fiables pueden no seguir los mismos estándares rigurosos de seguridad y autenticación.
