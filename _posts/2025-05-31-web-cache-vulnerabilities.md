---
title: Web Cache Deception & Web Cache Poisoning
author: d0x
date: 2025-05-31
categories: [Vulnerabilities, Notes]
tags: [Web Cache Vulnerabilities]
---

# Web cache deception

Esta vulnerabilidad puede hacer un disclosure de informacion sensible.
Para testear esto, hago el laboratorio de portswigger en el cual en uno de los ejercicios podemos ver lo siguiente en una de las respuestas.

![image](https://github.com/user-attachments/assets/d857cf95-2096-474c-84b2-83926f5dd1b7)

que significa esto, por ejemplo si yo hago una peticion por GET a un archivo, esa respuesta no esta cacheada aun, por lo tanto la respuesta va a tener un miss, ya que va al origin a buscarlo, y en la proxima peticion que realice, en lugar de un miss veremos un hit, ya q esa peticion ahora si esta cacheada.
Por ejemplo

```
GET /my-account/asdf.js HTTP/2
Host: 0a2c0025046d4738c9ae262b00e70081.web-security-academy.net
Cookie: session=3oYCUnROBQeiI0UHEcnh23M4TdoNdyhL
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a2c0025046d4738c9ae262b00e70081.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
```

Esta peticion tendra un miss en la response pero luego cambiara por un hit. Otra cosa importante a tener en cuenta, es cuanto tiempo va a durar estando cacheado, para eso vemos el Cache-Control, lo que nos esta diciendo ahi, es que como maximo va a durar 30 segundos, por lo que, pasados esos 30 segundos, pasara de hit a miss nuevamente.

En el primer lab, vemos como al pasar esta peticion, la peticion muestra un 200 y me muestra la response, pero no asi en otros. Ya que muchas veces hay que hacer uso de delimitadores, para se puede fuzzear con el intruder, pasandole como payload una lista de caracteres, y en el que arroje un 200, sabremos con certeza que ese delimitador esta permitido. Como muestro a continuacion.

![image](https://github.com/user-attachments/assets/0895ab3d-0693-42c5-bd8b-051c92d9aa04)

Como se puede observar en la imagen, tanto el ? como el ; arrojaron un 200, asique ahora podemos probar en la peticion con alguno de esos dos, ya que al contrario del primer lab, la peticion mostrada anteriormente no mostraba un 200 sino un 404, ya que el .js no esta siendo encontrado, por lo tanto seria un indicio tambien para determinar que deberiamos usar un delimitador.

![image](https://github.com/user-attachments/assets/a9a22b17-9f3f-4c50-92ce-38c636267520)

Al probar con el ? vemos que la respuesta nos devuelve 200 pero no esta mostrando el cache, puede deberse a que el ? tambien esta siendo cacheado. Quedaria probar con ;

![image](https://github.com/user-attachments/assets/0021ebbb-0b80-480b-9283-5a905b8e3830)

Y ahora si estamos viendo el cache en la respuesta, en este laboratorio particularmente solo habria que ahora mandarle un script a la victima, mediante el uso de exploit server proporcionado por portswigger, en el cual mandariamos la url completa haciendo uso de -> document.location="{{ URL }}" para que al hacer click en esa url el usuario victima quedaria cacheada ahora la respuesta, asique cuando yo realice ahora la peticion veria filtrada la api key del user en cuestion, obviamente solo porque estaba filtrada en el body.


# Web Cache Poisoning

Funciona igual que el web cache deception, pero en este tipo de vulnerabilidad no se trata de exfiltrar informacion sensible, sino que es mas probable poder inyectar por ejemplo un XSS.

![image](https://github.com/user-attachments/assets/c431a22d-7a33-476b-8f0c-ab134c0ab2c2)

aca vemos en la response los headers, ahrora bien, en caso de una prueba real, lo conveniente seria que si estas buscando por un web cache poisoning no hagas las pruebas en el home, ya que de lograr inyectar un xss por ejemplo estarias afectando la usabilidad de la pagina, ya que por esos 30 segundos cada usuario que quiera visitar la pagina estaria viendo un pop up, o bien podias alterar tambien el contenido visible de la web. Para esto se usa tambien un cache buster.

en este vemos que el fehost se esta reflejando en la response

![image](https://github.com/user-attachments/assets/3adcc030-a3ef-4c17-aead-30f5f4e44b6c)

Por lo tanto, lo primero que podrias probar es ver si yo modifico el value y agrego script tag puedo inyectar un alert.

![image](https://github.com/user-attachments/assets/6def62ec-bf3f-4780-a28a-a2f9a0cd815e)
ahi veo que se esta reflejando correctamente en la response, si ahora hacemos un get mediante la url + el parametro ?cb=test, deberiamos ver el pop up.

![image](https://github.com/user-attachments/assets/0de6021c-7c72-45db-8dd5-933ee07cdf10)

En este laboratorio era en la cookie, pero tambien puede ser en los headers, por lo que podemos fuzzear mediante param miner con el burp, para encontrar algun header en el que podamos inyectar codigo js tambien.


