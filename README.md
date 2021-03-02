# Proyecto Laravel a Producción

_Con esta guía aprenderás a hacer Deploy de tu proyecto Laravel en un hosting compartido, ya sea por primera vez o para actualizar._

---

## Pre-requisitos 📋️

Se asume desde ya que tu proyecto se encuentra listo para ser cargado, también que tienes instalado y conoces los manejadores [Composer](https://getcomposer.org/) y [NPM](https://www.npmjs.com/get-npm).

**Esta guía aplica SOLO a proyectos listos para producción.**
```
Ninguna de las optimizacionas realizadas aquí debe ser aplicada durante la etapa de desarrollo.
```

---

## Comenzando 🚀️

Una vez terminado el desarrollo de tu web/app debemos ingresar los siguientes comandos:

_Para optimizar JS y CSS_
```
npm run production
```

Se debe activar si trabajas con **Frameworks JS y/o CSS** tales como:
* **JS** -> React, Vue, Angular, Alpine.Js, etc
* **CSS** -> Bootstrap, Tailwind, etc.

_Para optimizar paquetes en laravel_
```
composer install --optimize-autoloader -no-dev
```
Este comando de Composer permite optimizar toda la carga de clases y paquetes dentro de tu apicacción.

⚠️ _Se debe ejecutar con cada deploy y cuando se hayan modificado los paquetes y dependencias._

---

## Comprimiendo el proyecto 📦️

_Procedemos a crear el comprimido que contendrá todas los archivos del programa._

_Veremos como cargar por primera vez o por actualización._

**En ambos casos NO es necesario comprimir ni subir los siguientes archivos:**

* **.git** (carpeta / puede existir o no)
```
Contiene la información relacionado a la gestión de GIT.
```

* **/storage/logs/xxx.log** (contenido de la carpeta logs)
```
Los archivos existentes dentro de la carpeta Logs se generaron durante el desarrollo, es importante borrarlos para que existan solo los nuevos generados en producción.
```
En algunos sistemas es posible eliminar estas carpetas del comprimido sin necesidad de re-comprimir todo el paquete.

#### Comprimiendo por primera vez 🥇️

_Siga estos pasos si es la primera vez que carga el proyecto en un servidor web._

Comprima la carpeta raíz de su proyecto, incluyendo vendor y node_modules (contienen las dependencias php y js).

El resultado final es un comprimido (.zip, .tar, etc) que contiene todas las carpetas y archivos: ``` app, database, vendor, artisan, webpack, etc. ```

⚠️ **Atención**: asegure que los archivos dot (.) se incluyan en el comprimido, por ejemplo que se inculya **.htaccess** en public o **.env** en raíz. _En algunos formatos estos archivos no se incluyen al comprimir._

#### Comprimiendo para una actualización 📣️

_Con actualización nos referimos a aplicar cualquier cambio sobre un proyecto ya exisitente en el servidor._

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

#### ¿Porque es importante saber si se actualizaron?

💡️ Pensemos lo siguiente:

vendor(+30mb) + node_modules(+100mb) + tu app tienen un peso aproximado superior a 100 mb.

¿Cargarías +100mb por cada actualización? ¿o solo cargarías tus modificaciones?

```
Actualizar las dependencias requiere cargar vendor y/o node_modules de nuevo.
```

Actualizar las dependecias puede llevar a problemas de compatibilidad, se recomienda testear antes de pasar a producción. Al ser archivos pesados requieren más ancho de banda para cargarse y más espacio de almacenamiento.

_En caso de actualizar, incluya vendor y/o node_modules en el comprimido._

---

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

Llegado a este punto nuestra aplicación ya está cargada! 🏆️🎉️

---


## Organizar los archivos

_Ahora es necesario realizar algúnos ajustes en nuestra aplicación antes de pasar a configurarla._

mover public, ajustar index y .env
