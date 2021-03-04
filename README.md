# Tu proyecto Laravel a Producción

_Con esta guía aprenderás a hacer Deploy de tu proyecto Laravel en un hosting compartido, ya sea por primera vez o para actualizar._

## Requisitos 📋️

Se asume desde ya que tu proyecto se encuentra listo para ser cargado, también que tienes instalado y conoces los manejadores [Composer](https://getcomposer.org/) y [NPM](https://www.npmjs.com/get-npm).

Esta guía se enfoca tanto en **cargar por primera vez como en actualizar un proyecto ya existente**, para estos casos, se implementará el comando ``` php artisan down ```, el cual se explicará más adelante.

Es necesario tener habilitado **acceso a la Terminal** en tu hosting. En cPanel puedes acceder en Avanzado -> Terminal.

#### No tengo acceso a la terminal ¿Qué hago? 📟️

En muchos casos **la terminal web no está habilitada por defecto**, por lo que debes pedir a tu proveedor de servicio que la habilite. Lo más sencillo es enviar un correo o abrir un nuevo Ticket de consulta.

Una vez habilitada se verá así (en cPanel):

![Terminal](https://mishorasweb.com/images/guia-prod/terminal.png)

_⚠️ **Importante:** Esta guía aplica SOLO a proyectos listos para producción._
```
Ninguna de las optimizaciones realizadas aquí debe ser aplicada durante la etapa de desarrollo.
```

## Comenzando 🚀️

Una vez terminado el desarrollo de tu web/app debemos ingresar los siguientes comandos en una terminal local dentro del proyecto:

_Para optimizar JS y CSS_
```
npm run production
```

Se debe activar si trabajas con **Frameworks JS y/o CSS** tales como:
* **JS** - React, Vue, Angular, Alpine.Js, etc
* **CSS** - Bootstrap, Tailwind, etc.

_Para optimizar paquetes en laravel_
```
composer install --optimize-autoloader -no-dev
```
Este comando de Composer permite optimizar toda la carga de clases y paquetes dentro de tu aplicación.

```
⚠️ Se debe ejecutar siempre y cuando se hayan modificado los paquetes o dependencias.
```

## Comprimiendo el proyecto 📦️

_Procedemos a crear el comprimido que contendrá todas los archivos del programa. Veremos como cargar por primera vez o por actualización._

**En ambos casos NO es necesario comprimir ni subir los siguientes archivos:**

* **.git** (carpeta / puede existir o no)
```
Contiene la información relacionado a la gestión de GIT.
```

* **/storage/logs/xxx.log** (contenido de la carpeta logs)
```
Los archivos existentes dentro de la carpeta Logs se generaron durante el desarrollo, es importante borrarlos para que existan solo los nuevos generados en producción.
```
📃️ Nota: En algunos sistemas es posible eliminar estas carpetas del comprimido sin necesidad de re-comprimir todo el paquete.

#### Comprimiendo por primera vez 🥇️

_Siga estos pasos si es la primera vez que carga el proyecto en un servidor web._

Comprima la carpeta raíz de su proyecto, incluyendo vendor y node_modules (contienen las dependencias php y js).

El resultado final es un comprimido (.zip, .tar, etc.) que contiene todas las carpetas y archivos: ``` app, database, vendor, artisan, webpack, etc. ```

⚠️ **Atención**: asegure que los archivos dot (.) se incluyan en el comprimido, por ejemplo ``` .htaccess en /public ``` o ``` .env en /raíz ```. En algunos casos estos no se incluyen al comprimir.

#### Comprimiendo para una actualización 📣️

_Con actualización nos referimos a aplicar cualquier cambio sobre un proyecto ya existente en el servidor._

Es **importante** que se sepa si las siguientes carpetas fueron modificadas:

* **vendor** - Contiene dependencias PHP
* **node_modules** - Contiene dependencias JS y TS

```
📃️ Nota: Por lo general estas 2 carpetas son las más pesadas.
```

Estas carpetas se actualizan al realizar los siguientes comandos:

**vendor**
```
composer update
```

**node_modules**
```
npm update
```

#### ¿Por qué es importante saber si se actualizaron?

💡️ Pensemos lo siguiente:

vendor(+30mb) + node_modules(+100mb) + tu app = peso aproximado superior a 100 mb.

¿Cargarías +100mb por cada actualización? O ¿Cargarías solo tus modificaciones?

```
Actualizar las dependencias requiere cargar vendor y/o node_modules de nuevo.
```

Actualizar las dependencias puede llevar a problemas de compatibilidad, se recomienda testear antes de pasar a producción. Al ser archivos pesados requieren más ancho de banda para cargarse y más espacio de almacenamiento.

_En caso de actualizar dependencias, incluya vendor y/o node_modules en el comprimido._


## Cargando los archivos 🗃️

_Veamos los procedimientos que debemos seguir para cargar nuestro proyecto en el servidor._

Este ejemplo está basado en interfaz de cPanel, el proceso no suele variar mucho entre distintos hostings:

* **Ir a Archivos**
* **Sección de carga de archivos**
* **Cargar comprimido en directorio raíz**
* **Descomprimir**

El resultado será algo parecido a esto:

```
🏠️ /home/tu-usuario/tu-proyecto-laravel
```

Felicidades tu aplicación ya está cargada! 🏆️🎉️

## Modo mantenimiento 🔧️

_Esta modo puede ser activado cuando el proyecto está cargado y descomprimido, revisaremos como actuar si es la primera vez que se carga o si es para actualizar._

El **Modo Mantenimiento inhabilita el acceso de los usuarios a nuestra web** para que podamos trabajar los cambios necesarios sin que haya conflictos. Al activarse e intentar ingresar se mostrará el mensaje "503" diciendo que el servicio está en mantenimiento.

**[¿Cómo modificar la pantalla de mantenimiento?](https://youtu.be/tFBfPKSBG4Y)** - Ver vídeo - _Simply UY_

```
⚠️ Importante: Activar el modo mantenimiento siempre que se quiera actualizar la web.
```

_**Para las cargas por primera vez no es necesario** ya que por lo general no se cuenta con usuarios que puedan recibir los posibles errores que surgen durante la carga._

_Accedemos a la terminal web e ingresamos en el directorio raíz de nuestro proyecto:_
```
php artisan down
```

_En este punto, probamos acceder a nuestra URL y comprobamos el mensaje "503"._

## Gestionando los archivos 🗂️

_Ahora es necesario realizar algunos ajustes antes de pasar a configuración más profunda. Revisaremos la seguridad general aplicando buenas practicas._

### Cambiar los permisos 🔒️

_Es común que desarrollemos nuestro proyecto como administradores (en windows) o con permisos 777 o sudo en linux o Mac, si no cambiamos esto en producción los **archivos pueden ser modificados por terceros**._

🛡️ _Por defecto Laravel/Symphony notifica que los archivos tienen permiso 777 estando en el servidor, lo cual permite que personas no deseadas puedan modificarlos, **para evitar esta vulnerabilidad** realizaremos lo siguiente:_

---

⚠️ _**Terminal requerida!** Los siguientes comandos se pueden ejecutar en la terminal web para más comodidad, de lo contrario se deben aplicar de manera local antes de generar el comprimido._

---

Será necesario cambiar los permisos y asignar nuevos:
      
**En Carpetas** - Asignamos de forma recursiva los permisos con el comando:
```
 find /home/tu-usuario/tu-proyecto-laravel -type d -exec chmod 755 {} \;
```
**En Archivos** - Asignamos de forma recursiva los permisos con el comando:
```
 find /home/tu-usuario/tu-proyecto-laravel -type f -exec chmod 664 {} \;
```
_La asignación de permisos evita que cualquiera pueda manipular los archivos, de esta forma asignamos solo lectura para todos los usuarios._

---

De todas formas **Laravel necesita leer/escribir** en la carpeta ``` /storage ``` por lo que también le asignaremos los siguientes permisos:

```
sudo chmod -R ug+rwx /home/tu-usuario/tu-proyecto-laravel/storage /home/tu-usuario/tu-proyecto-laravel/bootstrap/cache
```

🛡️ _Ejecutados estos comandos la aplicación se encuentra **segura frente a escrituras y ejecuciones maliciosas.**_


## Optimizar Laravel 🚀️

_Llegado a este punto nos enfocaremos en optimizar la carga y velocidad de nuestra página aplicando algunos comandos artisan que Laravel tiene para ofrecernos._

Es necesario limpiar el cache almacenado durante el desarrollo y reemplazarlo por nuevo estando ya en el servidor, esto agiliza los tiempos de carga y procesamiento, además de evitar errores.

#### Comandos ⌨️

⚠️ **Requiere el uso de la terminal web!** Aplicaremos estos comandos en la terminal web estando situados en la raíz de nuestro proyecto. Por ejemplo en ``` /home/tu-usuario/tu-proyecto-laravel/ ```.

🧐️ _Si no podemos utilizar los comandos quiere decir que estamos parados en la ruta incorrecta!_

_Limpiar el cache de las rutas_
```
php artisan route:clear
```
_Limpiar las configuraciones_
```
php artisan config:clear
```
_Limpieza de cache general_
```
php artisan cache:clear
```

_Esta linea realiza todo en una sola maniobra_ 📃️✂️
```
php artisan route:clear && php artisan config:clear && php artisan cache:clear
```

En este punto **se habrá eliminado todo el cache generado durante el desarrollo local.**

---

### Generaremos nuevo cache en el servidor 📔️

_Optimiza los archivos de configuración_
```
php artisan config:cache
```
_Optimiza las carga de rutas_
```
php artisan route:cache
```
_Compila archivos blade para no hacerlo en demanda_
```
php artisan view:cache
```

_Esta linea realiza todo en una sola maniobra_ 📃️✂️
```
php artisan route:cache && php artisan config:cache && php artisan view:cache
```

En este punto **se habrá generado nuevo cache con las directivas del servidor.**

#### ¿Por qué limpiar cache? 🤔️

Al desarrollar de manera local, las configuraciones y datos se almacenan con rutas similares a  ``` /var/www/html/tu-proyecto-laravel/ ```, si no eliminamos el cache, el servidor buscará trabajar con esa ruta y esto generará errores.

```
Los comandos anteriores deben ser ejecutados SI o SI ✔️
```

## Re-ubicando public 📌️

Como buena practica separaremos el proyecto en dos secciones, la carpeta "public" y el resto. Hacemos esto ya que **no es necesario tener todo el contenido del programa en la carpeta public_html** de nuestro hosting, con solo algunos archivos allí Laravel hará funcionar la aplicación.

Moveremos el contenido la carpeta "public" ``` /home/tu-usuario/tu-proyecto-laravel/public ``` hacia ``` /home/public_html ```, esta ruta es donde accede el usuario cuando visita la página. La carpeta public_html será similar a esta:

 ![public_html](https://mishorasweb.com/images/guia-prod/public.png)

_Las carpetas y archivos varían según cada proyecto pero la estructura es similar._

```
📃️ Nota: Queda a criterio de cada uno si eliminar o no la carpeta public (que ahora está vacía porque le sacamos el contenido).
```

### Actualizar contenido de public_html 🌐️

*En caso de querer actualizar un public_html ya existente no es necesario eliminar todo.*

Debemos identificar las carpetas que han sido modificadas, un caso habitual **si trabajamos con Frameworks JS** es que el archivo **app.js** ubicado en ``` /js/app.js ``` es modificado al guardar los cambios en nuestros archivos .js, .vue, etc.

También es común que se modifique **app.css** ubicado en ``` /jscss/app.css ```.

*Para trabajar correctamente public_html debemos **reemplazar solo los archivos que se hayan modificado** durante el desarrollo, los podemos cargar individualmente o en comprimido.*

### Modificando index.php ⚙️

_Este archivo se encarga de hacer funcionar nuestra aplicación, si lo examinamos vemos que invoca archivos de dos carpetas, entre otras varias cosas. Las archivos son ``` /vendor/autoload.php ``` y ``` /bootstrap/app.php ```._

La primera inclusión es: ```__DIR__.'/../vendor/autoload.php ```

![public_html](https://mishorasweb.com/images/guia-prod/vendor.png)
 
La segunda inclusión es: ```__DIR__.'/../bootstrap/app.php ```

![public_html](https://mishorasweb.com/images/guia-prod/bootstrap.png)

#### ¿Qué debemos cambiar?

Si lo analizamos, las inclusiones buscan los archivos vendor y bootstrap que se encuentren en esa ruta, pero estos no existen allí porque **los separamos de public en el paso anterior!**.

_Para logar acceder a estos archivos debemos modificar los "require"_.

Como public ahora está en public_html y vendor/bootsrap en la carpeta ``` /home/tu-usuario/tu-proyecto-laravel ``` debemos indicar la búsqueda de los archivos en una ruta anterior, para ello **cambiamos los require**:

_La nueva primera inclusión:_ ```__DIR__.'/../tu-proyecto-laravel/vendor/autoload.php ```

_La nueva segunda inclusión:_ ```__DIR__.'/../tu-proyecto-laravel/bootstrap/app.php ```

```
⚠️ Importante: Evitar modificar el archivo index.php cuando cargues nuevas versiones. De ser así, actualizalo de inmediato.
```

**De esta forma, nuestro proyecto ya puede cargar todo lo necesario!** 🏆️

## Cada vez falta menos 🏁️

_Es aquí donde te recomiendo repasar lo anterior y asegurarte que todo esté en orden_ 🤓️

_Aprovecho a comentar que puedes **encontrarme en Youtube**, soy [Simply UY](https://www.youtube.com/channel/UChhbijYNjlgiVuqPEIMXuzQ) y subo vídeos sobre programación en Laravel y Vue.js entre tantas cosas más._

#### 🛸️ Continuamos...

---







