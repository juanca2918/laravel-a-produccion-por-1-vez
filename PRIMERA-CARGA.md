# Tu proyecto Laravel a Producción por primera vez

_Con esta guía aprenderás a hacer Deploy de Tu Proyecto Laravel en un hosting compartido Por Primara Vez, veremos en detalle todos los conceptos necesarios para lograr esto._

## Requisitos 📋️

Se asume desde ya que tu proyecto se encuentra listo para ser cargado, también que tienes instalado y conoces los manejadores [Composer](https://getcomposer.org/) y [NPM](https://www.npmjs.com/get-npm).

**Es recomendable pero NO necesario tener habilitado acceso a la Terminal en tu hosting**. En cPanel puedes acceder en Avanzado -> Terminal.

#### Quiero usar la terminal pero no la tengo ¿Qué hago? 📟️

En muchos casos **la terminal web no está habilitada por defecto**, por lo que debes pedir a tu proveedor de servicio que la habilite. Lo más sencillo es enviar un correo o abrir un nuevo Ticket de consulta.

Una vez habilitada se verá así (en cPanel):

![Terminal](https://mishorasweb.com/images/guia-prod/terminal.png)

❗️ **Importante:** Esta guía aplica SOLO a proyectos listos para producción.
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
❗️ Se debe ejecutar siempre y cuando se hayan modificado los paquetes o dependencias ❗️
```

_Para limpiar todo el cache local_
```
php artisan route:clear && php artisan config:clear && php artisan cache:clear
```

## Comprimiendo el proyecto 📦️

_Procedemos a crear el comprimido que contendrá todas los archivos del programa._

**NO es necesario comprimir ni subir los siguientes archivos:**

* **.git** (carpeta / puede existir o no)
```
Contiene la información relacionado a la gestión de GIT.
```

* **/storage/logs/xxx.log** (contenido de la carpeta logs)
```
Los archivos existentes dentro de la carpeta Logs se generaron durante el desarrollo, es importante borrarlos para que existan solo los nuevos generados en producción.
```

📃️ Nota: En algunos sistemas es posible eliminar estas carpetas del comprimido sin necesidad de re-comprimir todo el paquete.

_Comprima la carpeta raíz de su proyecto, incluyendo vendor y node_modules (contienen las dependencias php y js)._

El resultado final es un comprimido (.zip, .tar, etc.) que contiene todas las carpetas y archivos: ``` app, database, vendor, artisan, webpack, etc. ```

❗️ **Atención**: asegure que los archivos dot (.) se incluyan en el comprimido, por ejemplo ``` .htaccess en /public ``` o ``` .env en /raíz ```. En algunos casos estos no se incluyen al comprimir.


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

## Gestionando los archivos 🗂️

_Ahora es necesario realizar algunos ajustes antes de pasar a configuración más profunda. Revisaremos la seguridad general aplicando buenas practicas._

### Cambiar los permisos 🔒️

_Es común que desarrollemos nuestro proyecto como administradores (en windows) o con permisos 777 o sudo en linux o Mac, si no cambiamos esto en producción los **archivos pueden ser modificados por terceros**._

🛡️ _Por defecto Laravel/Symphony notifica que los archivos tienen permiso 777 estando en el servidor, lo cual permite que personas no deseadas puedan modificarlos, **para evitar esta vulnerabilidad** realizaremos lo siguiente:_

---

❗️ _**Terminal requerida!** Los siguientes comandos se pueden ejecutar en la terminal web para más comodidad, de lo contrario se deben aplicar de manera local antes de generar el comprimido._

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

## Re-ubicando public 📌️

Como buena practica separaremos el proyecto en dos secciones, la carpeta "public" y el resto. Hacemos esto ya que **no es necesario tener todo el contenido del programa en la carpeta public_html** de nuestro hosting, con solo algunos archivos allí Laravel hará funcionar la aplicación.

Moveremos el contenido la carpeta "public" ``` /home/tu-usuario/tu-proyecto-laravel/public ``` hacia ``` /home/public_html ```, esta ruta es donde accede el usuario cuando visita la página. La carpeta public_html será similar a esta:

 ![public_html](https://mishorasweb.com/images/guia-prod/public.png)

_Las carpetas y archivos varían según cada proyecto pero la estructura es similar._

```
📃️ Nota: Queda a criterio de cada uno si eliminar o no la carpeta public (que ahora está vacía porque le sacamos el contenido).
```

### Modificando index.php ⚙️

_Este archivo se encarga de hacer funcionar nuestra aplicación, si lo examinamos vemos que invoca archivos de dos carpetas, entre otras varias cosas. Las archivos son ``` /vendor/autoload.php ``` y ``` /bootstrap/app.php ```._

La primera inclusión es: ```__DIR__.'/../vendor/autoload.php ```

![vendor](https://mishorasweb.com/images/guia-prod/vendor.png)
 
La segunda inclusión es: ```__DIR__.'/../bootstrap/app.php ```

![bootstrap](https://mishorasweb.com/images/guia-prod/bootstrap.png)

#### ¿Qué debemos cambiar?

Si lo analizamos, las inclusiones buscan los archivos vendor y bootstrap que se encuentren en esa ruta, pero estos no existen allí porque **los separamos de public en el paso anterior!**.

_Para logar acceder a estos archivos debemos modificar los "require"_.

Como public ahora está en public_html y vendor/bootsrap en la carpeta ``` /home/tu-usuario/tu-proyecto-laravel ``` debemos indicar la búsqueda de los archivos en una ruta anterior, para ello **cambiamos los require**:

_La nueva primera inclusión:_ ```__DIR__.'/../tu-proyecto-laravel/vendor/autoload.php ```

_La nueva segunda inclusión:_ ```__DIR__.'/../tu-proyecto-laravel/bootstrap/app.php ```

```
❗️ Importante: Evitar modificar el archivo index.php cuando cargues nuevas versiones. De ser así, actualizalo de inmediato.
```

**De esta forma, nuestro proyecto ya puede cargar todo lo necesario!** 🏆️

## Cada vez falta menos 🏁️

_Es aquí donde te recomiendo repasar lo anterior y asegurarte que todo esté en orden_ 🤓️

_Aprovecho a comentar que puedes **encontrarme en Youtube**, soy [Simply UY](https://www.youtube.com/channel/UChhbijYNjlgiVuqPEIMXuzQ) y subo vídeos sobre programación en Laravel y Vue.js entre tantas cosas más._

## Conexión a Base de Datos 🗃️

_Realizaremos la conexión completa con la base de datos de producción._

❗️ Los siguientes ejemplos se basan en cPanel, los procesos no suelen variar mucho entre los distintos proveedores.

Recuerda copiar los datos que generaremos ahora: ``` usuario, contraseña y base de datos. ```

### Configurando desde 0

_Crearemos todo mediante la interfaz web._

Databases -> MySQL Databases
_Aquí podremos crear una nueva base para nuestro proyecto._

![db-crear](https://mishorasweb.com/images/guia-prod/db-crear.png)

_Creamos el usuario que utilizará la base de datos._

![db-crear](https://mishorasweb.com/images/guia-prod/db-user.png)

_Agregamos el usuario a la base de datos para que la pueda manipular._

![db-crear](https://mishorasweb.com/images/guia-prod/db-add-user.png)

#### Permisos de usuarios 🔓️

_Evitemos errores y revisemos que los permisos del usuario sean correctos para esa base._

Esto permite que el usuario pueda ejecutar las sentencias ```SELECT, INSERT, UPDATE, DELETE, etc.``` de SQL.

#### PhpMyAdmin

_Si tenemos acceso a un manejador de BD podemos comprobar que la base se haya configurado de manera correcta._

**En este punto tendremos nuestra Base de Datos pronta para usar 👏️**

## Ajustando variables de entorno 🔧️

_Ahora trabajaremos con el archivo .env de tu proyecto._

Nos aseguramos que **.env** se haya respaldado en el comprimido, de no ser así, creamos un nuevo .env en la raíz de nuestro proyecto y copiamos dentro todos los datos.

_Cambio de variables: Revisaremos 3 secciones de nuestro .env_

```
Los datos pueden variar según cada proyecto.
```

**Variables APP_**

![env_data](https://mishorasweb.com/images/guia-prod/env-data.png)

* ENV = ```local``` por ```production```

_Con esto Laravel sabe que el entorno ahora es de Producción._

* KEY = ```base64:xxx...``` por ```key:generate```

_Key es un aributo de seguridad de tu proyecto, para actualizarlo en producción crearemos una nueva clave estando en la terminal local._

Ingresamos comando: ```php artisan key:generate```

Nos creará una nueva key en **.env local** la cual copiamos y **pegamos en el .env de producción**.

* DEBUG = ```true``` por ```false```

_Debug define si mostrar errores estando en Producción, es Importante establecerlo FALSE para que el usario no vea información sensible en caso de error._

* URL = ```http://localhost``` por ```https://mi-dominio.com```

_Cambiamos la url local por nuestro dominio._

**Variables DB_**

❗️ Es aquí cuando utilizamos los datos ```usuario, contraseña y base de datos``` que habíamos guardado antes.

![env_db](https://mishorasweb.com/images/guia-prod/env-bd.png)

* DATABASE = ```tu-base-local``` por ```tu-base-web```

_La base local será remplazada por la base web._

* USERNAME = ``` user-local ``` por ``` user-web ```

_Ingresamos el usuario con acceso a la DB de nuestro hosting._

* PASSWORD = ``` pass-local ``` por ``` pass-web ```

_Cambiamos por la contraseña de ese usuario._

**Variables MAIL_**

❗️ Esta configuración **NO es necesaria** si NO pensamos utilizar los servicios de **correo de nuestro dominio o de terceros**.

En este ejemplo se ve el uso de [Mailtrap](https://mailtrap.io/) para el testing de correos durante desarrollo, vamos a cambiar esto.

![env_mail](https://mishorasweb.com/images/guia-prod/env-mail.png)

_Necesitamos contactar con nuestro proveedor para obtener algunos datos:_ ``` MAILER, PORT nos lo brinda nuestro proveedor ```.

* HOST = ```smtp.mailtrap.io``` por ```tu-dominio.com```

_Cambiamos el host de prueba (si existe) por nuestro dominio web._

📨️ Para estos datos debemos de tener un correo existente bajo nuestro dominio.

* USERNAME = ```test-user``` por ```real-mail-user```

_Ingresamos el correo completo **example@tu-dominio**._

* PASSWORD = ```test-pass``` por ```real-mail-pass```

_La contraseña de ese correo._

* FROM_ADDRESS = ```quien-envia@tu-dominio.com```

**Otras variables**

_Recuerden además cambiar las variables que cada uno haya modificado en su proyecto._

**Nuestro .env ya está configurado y listo para funcionar! 🎉️**

## Optimizar Laravel 🚀️

_Llegado a este punto nos enfocaremos en optimizar la carga y velocidad de nuestra página aplicando algunos comandos artisan que Laravel tiene para ofrecernos._

Es necesario limpiar el cache almacenado durante el desarrollo y reemplazarlo por nuevo estando ya en el servidor, esto agiliza los tiempos de carga y procesamiento, además de evitar errores.

### Comandos ⌨️

🗳️ **Recordemos que ya limpiamos el cache al inicio de la guía!**

❗️ **Requiere el uso de la terminal web!** Aplicaremos estos comandos **SOLO** si tenemos la terminal web.

Nos situamos en la raíz de nuestro proyecto. Por ejemplo en ``` /home/tu-usuario/tu-proyecto-laravel/ ```.

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

### Generar nuevo cache en el servidor 📔️

❗️ **NO utilizar los siguientes comandos si no tenemos acceso a la Terminal Web!**

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

❗️ **NO utilizar los comandos anteriores si no tenemos acceso a la Terminal Web!**

#### ¿Por qué limpiar cache? 🤔️

Al desarrollar de manera local, las configuraciones y datos se almacenan con rutas similares a  ``` /var/www/html/tu-proyecto-laravel/ ```, si no eliminamos el cache, el servidor buscará trabajar con esa ruta y esto generará errores.

_Limpiar cache_
```
La limpieza de cache debe ser ejecutada SI o SI ✔️
```
_Generar nuevo cache_
```
SOLO debemos generar nuevo cache si tenemos acceso a La Terminal Web ❗️
```


