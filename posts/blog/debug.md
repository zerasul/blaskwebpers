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

A la hora de programar, es necesario tener un editor que nos permita realizar distintas operaciones como puede ser el depurar o ayudarnos a la hora de crear el código. Aunque hay muchos como puede ser Eclipse, Clion o en este caso usaremos [Visual Studio Code](https://code.visualstudio.com/). 

Pero antes de entrar en materia, vamos a ver como funciona el depurador y como podemos usarlo con una compilación cruzada. Realmente, se trata de poder depurar el programa que se esta ejecutando en una máquina remota (o un emulador). Esto lo realizaremos usando el depurador GDB, como hemos comentando antes.


<div>
  <figure class="image">
   <img src="/static/img/depuracionremota.png" alt="DepuracionRemota">
   <span><strong>Esquema de depuracion Remota</strong></span>
   </figure>
</div>



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

Esto generara un binario, cuyo contenido esta formado por 0 y 1. El cual no podemos leerlo a simple vista (aunque podemos sacar el ensamblador y ver cada instrucción); por lo que necesitaremos poder traducir este binario para poder depurarlo posteriormente. Sin embargo, no podemos hacer eso desde el propio binario ya que no tenemos información; por ejemplo de las variables y funciones usadas; lo cual nos puede dificultar la depuración. Es por ello, que vamos a compilar nuestra ROM en modo depuración, para así generar la tabla de Simbolos.

``` %GDK_WIN%\bin\make -f %GDK_WIN%\makefile.gen debug```

Esto nos va a generar la ROM con todo lo que necesitaremos para poder depurar nuestra ROM. Como la tabla de simbolos que podemos encontrar en _out/simbols.txt_. Contiene toda la información de variables y funciones de nuestro juego.

Otro aspecto a tener en cuenta es como vamos a poder depurar el juego y ejecutarlo; aunque existen cartuchos como los [everdrive](https://www.amazon.es/Everdrive-Megadrive-genesis-Flash-Drive/dp/B00I121A1M); que permiten usando una tarjeta SD, añadir ROMS para poder jugarlos en nuestra Mega Drive, aunque no permiten realizar una depuración; aunque existen [proyectos](https://github.com/jdesiloniz/ssp16asm) para poder crear un cartucho que pueda ser usado como depurador. Para nuestro caso, usaremos un emulador para poder realizar la depuración.

En este artículo hemos probado con el emulador [Gens](https://segaretro.org/Gens_KMod) con la modificación Kmod. Se ha intentado con otro emulador como [Blastem](https://www.retrodev.com/blastem/), pero no tiene implementado completamente el poder depurar usando GDB.


<div>
  <figure class="image resp-image">
   <img src="/static/img/gens.png" alt="gens Kmod">
   </figure>
   <span><strong>Emulador Gens KMod</strong></span>
</div>


Este emulador nos permite depurar un juego, usando de forma remota GDB. Para ello se puede configurar el puerto de este; en este artículo solo depuraremos el procesador principal (Motorola 68000), aunque también se puede depurar el Z80 (usado para controlar sonido o modo Retrocompatibilidad de Master System). las opciones pueden encontrarse en el menú Option->Debug. Hay que seleccionar la opción de use GDB (Tendremos que reiniciar el emulador para que tenga efecto) y poner el puerto que queramos usar; usaremos el que trae por defecto 6868.

Tras tener el emulador y activar las opciones de depuración, vamos a usar GDB para poder parar el programa y depurarlo. Para ello usaremos el programa GDB que trae incluido el directorio bin de la carpeta donde tengamos instalado el SGDK. Usaremos el siguiente comando para entrar en gdb para depurar nuestro juego:

``` %GDK%\bin\gdb out\rom.out ```

El fichero _rom.out_ es uno de los generados por la compilación en modo depuración, el cual contiene información acerca del binario nuestra ROM.

Una vez dentro del depurador de GDB y hemos abierto la rom (fichero rom.bin) en el emulador, el cual nos mostrará la siguiente información:

    :::bash
    GNU gdb (GDB) 7.12.1
    Copyright (C) 2017 Free Software Foundation, Inc.    
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "--host=i686-pc-mingw32 --target=m68k-elf".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.
    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    Reading symbols from out\rom.out...done.
    (gdb) 


Como vemos se ha leido la tabla de simbolos y esta a la espera de poder conectarse con nuestro emulador; escribimos el siguiente comando:

``` (gdb) target remote:6868 ```

Y observaremos que el emulador ha parado el juego (puede incluso que se vea en azul); y podemos observar en que punto se ha parado o en que función se encuentra:

    :::bash
    Remote debugging using :6868
    0x00007f28 in partition_s16 (data=0x48e70038, p=4, r=28672) at D:/Apps/SGDK/src/tools.c:1115
    1115    D:/Apps/SGDK/src/tools.c: No such file or directory.

Nos indica en que linea se encuentra (el error de que no encuentra el fichero o directorio, es correcto ya que no hemos configurado la ruta donde se encuentra el código fuente de SGDK, ni el nuestro); si añadimos el comando _bt_ veremos la pila de programa y podremos ver las funciones, y además el comando _n_ nos permite seguir avanzando.

**NOTA:**: Es Recomendable que se deshabilite el sonido ya que al parar la ejecución puede ser algo molesto.

Ya hemos conseguido conectar nuestro emulador y GDB; pero aunque ya podemos obtener más información, nuestro objetivo es poder depurarlo usando Visual Studio Code. Para ello, visual studio code permite crear configuraciones para poder depurar usando GDB o cualquier otro depurador; de esta forma, vamos a poder enlazar VSCode con GDB y depurar nuestro juego.

Para poder configurar la depuración en VSCode, vamos a añadir un fichero con las tareas que podemos realizar para poder lanzar el depurador. Para crear este fichero podemos hacerlo en el menú _run->Add configuration..._ nos aparecerá una lista y seleccionaremos la opción de depurar con GDB. Luego solo tenemos que añadir al fichero que se nos ha creado _launch.json_, la siguiente configuración.


    :::json
      {
        "version": "0.2.0",
        "configurations": [    
          {
              "name": "Debug with gdb remote",
              "request": "launch",
              "type": "cppdbg",
              "program": "${workspaceRoot}\\out\\rom.out",
              "miDebuggerServerAddress": "localhost:6868",
              "sourceFileMap": {
                  "d:\\apps\\sgdk\\src\\": "F:\\SGDK2\\src\\",
              },
              "args": [],
              "stopAtEntry": true,
              "cwd": "${workspaceFolder}",
              "environment": [],
              "externalConsole": false,
              "MIMode": "gdb",
              "launchCompleteCommand": "exec-continue",
              "miDebuggerPath": "F:\\SGDK2\\bin\\gdb.exe",
              "setupCommands": [
                  {
                      "text": "set directories '${workspaceFolder};$cwd;$cdir'"
                  }
              ],
          }
      ]
    }

Como podemos ver en la configuración anterior, se ha añadido una nueva configuración para usar GDB. Seguidamente vamos a mostrar algunas de las propiedades de esta configuración para poder comprender como se realizará la depuración.

<div class="content">
<ul>
  <li> <strong>name</strong>: Nombre de la configuración que podremos ver en VSCODE.</li>
  <li> <strong>program</strong>: Indica el nombre del binario que usará gdb para iniciar la depuración; se trata del fichero _rom.out_ que se genera al compilar en modo depuración.</li>
  <li> <strong>miDebuggerServerAddress</strong>: Indica la dirección y puerto donde se conectará gdb para hacer la depuración remota. Debe coincidir con el puerto del emulador.</li>
  <li><strong>sourceFileMap</strong>: Esta propiedad es importante ya que GDB tiene establecidas unas rutas con las que se compilo y se configuró (Concretamente las del proyecto SGDK); por lo tanto se debe de mapear a nuestra carpeta de fuentes de SGDK.</li>
  <li><strong>cwd</strong>: Indica el directorio de trabajo.</li>
  <li><strong>MIMode</strong>: Indica el modo de depurador en este caso se trata de gdb.</li>
  <li> <strong>miDebuggerPath</strong>: Ruta donde se encuentra GDB; en este caso se usa el integrado en SGDK. Puede definirse otro.</li>
  <li> <strong>setupCommands</strong>: Indica los comandos y configuraciones que se pasará a GDB. Entre ellas se establece el directorio de los fuentes.</li>
</ul>
</div>

Una vez que finalizamos la configuración, solo nos queda probar y ver si podemos depurar nuestro programa. Para ello, pulsamos en la pestaña de depuración de VSCode y arrancamos la configuración que acabamos de crear. Obviamente no hay que olvidar poner antes el emulador con el juego que estamos depurando. Si todo va bien, debería de mostrarse una pantalla similar a esta.

![vscodedebug](/static/img/vscodedebug.png)
<span><strong>Visual Studio Code depurando una rom</strong></span>

Por último, con la depuración ya conseguida, podremos depurar nuestros juegos para mega Drive usando este depurador ya que muchas veces puede ser un quebradero de cabeza encontrar donde esta el error. 

<h3 class="title is-3">Referencias </h3>

<div class="content">
  <ul>
      <li><a href="https://github.com/Stephane-D/SGDK">https://github.com/Stephane-D/SGDK</a></li>
      <li><a href="https://www.gnu.org/software/gdb/">https://www.gnu.org/software/gdb/</a></li>
      <li><a href="https://gcc.gnu.org/">https://gcc.gnu.org/</a></li>
      <li><a href="https://segaretro.org/Gens_KMod">https://segaretro.org/Gens_KMod</a></li>
      <li><a href="https://github.com/jdesiloniz/ssp16asm">https://github.com/jdesiloniz/ssp16asm</a></li>
      <li><a href="https://code.visualstudio.com/docs/cpp/cpp-debug">https://code.visualstudio.com/docs/cpp/cpp-debug</a></li>
      <li><a href="https://marketplace.visualstudio.com/items?itemName=zerasul.genesis-code">https://marketplace.visualstudio.com/items?itemName=zerasul.genesis-code</a></li>
      <li>Grupo de Desarrolladores Mega Drive Dev de telegram</li>
  </ul>
</div>

<hr/>