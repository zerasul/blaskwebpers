---
date: 2020-02-24
title: Depurando Mega Drive usando VSCode | Victor Suarez
template: blog.html
author: Victor
category: articulo
tags: megadrive, sgdk
---

<section class="hero">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        Depurando Mega Drive con Visual Studio Code
      </h1>
      <h2 class="subtitle">
        Como depurar tu Mega Drive usando Vistual Studio Code.
      </h2>
    </div>
  </div>
</section>

A la hora de programar, es esencial, saber depurar un programa. Para poder detectar errores y conocer correctamente que esta haciendo nuestro programa en todo momento. Pero ¿Qué ocurre cuando queremos depurar un videojuego en una plataforma que no es un PC como tal, sino que por ejemplo es una consola como puede ser la antigua Mega Drive?

En este articulo, vamos a mostrar como podemos usar un depurador como [GDB](https://www.gnu.org/software/gdb/), para poder depurar los juegos que podemos desarrollar para la Sega Mega Drive. Normalmente, en sistemas como la Mega Drive, se utiliza el ensamblador (Assembly); pero en este caso, vamos a usar el lenguaje de programación C, a través de una serie de librerías y herramientas que nos provee el proyecto [SGDK](https://github.com/Stephane-D/SGDK).

Gracias a las herramientas que nos provee SGDK, podemos compilar un programa en C para la arquitectura que tiene la Mega Drive (en este caso se trata de un [Motorola 68000](https://es.wikipedia.org/wiki/Motorola_68000)). A este proceso de compilar en otra arquitectura se le llama compilación cruzada.

A la hora de programar, es necesario tener un editor que nos permita realizar distintas operaciones como puede ser el depurar o ayudarnos a la hora de crear el código. Aunque hay muchos como puede ser Eclipse, Clion o en este caso usaremos [Visual Studio Code](https://code.visualstudio.com/). Para este artículo, usaremos VScode (También podemos usar Visual Studio Codium).

Pero antes de entrar en materia, vamos a ver como funciona el depurador y como podemos usarlo con una compilación cruzada. Realmente, se trata de poder depurar el programa que se esta ejecutando en una máquina remota (o un emulador). Esto lo realizaremos usando el depurador GDB, como hemos comentando antes.

![DepuracionRemota](/static/img/depuracionremota.png)
<span><strong>Esquema de depuracion Remota</strong></span>

Como podemos ver en el esquema anterior, en nuestro equipo de desarrollo (host), tendremos el editor Visual Studio code que se comunicara con el depurador GDB y este a su vez, se conectara con el juego que este ejecutandose en este caso (aunque mostramos en el esquema una mega drive), se trata de un emulador. Vamos a ver  a continuación, cada una de estas partes para depurar nuestros juegos de mega drive que crearemos con SGDK.

Para empezar, vamos a ver un ejemplo de código C usando SGDK; haciendo un "Hola Mundo":

    :::C
    #include <genesis.h>

    int main(){

      VDP_drawText("Hello Sega!", 10, 13); 
      
      while(1){
         VDP_waitVSync();
      }
      return 0;
    }

Si compilamos el juego y lo ejecutamos, veremos como en un emulador (o en un hardware real), se muestra por pantalla con letras el texto 'Hello Sega!'. Para poder compilar este programa, normalmente utilizariamos un compilador de C como puede ser [GCC](https://gcc.gnu.org/), o usaremos la herramienta [make](https://www.gnu.org/software/make/), para poder crear el binario.

``` %GDK_WIN%\bin\make -f %GDK_WIN%\makefile.gen ```

**NOTA**: Este comando coresponde a la instalación SGDK para Windows.

Esto generara un binario, cuyo contenido esta formado por 0 y 1. El cual no podemos leerlo a simple vista (aunque podemos sacar el ensamblador y ver cada instrucción); por lo que necesitaremos poder traducir este binario para poder depurarlo posteriormente.