---
date: 2020-07-05
title: Creando juegos de Game Boy Advance con Rust | Victor Suarez
template: blog.html
author: Victor
category: articulo
tags: gba,retrodev,rust,crosscompiling, homebrew
---

<section class="hero">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        Creando juegos de Game Boy Advance con Rust
      </h1>
      <h2 class="subtitle">
        Usar el lenguaje Rust para poder crear Juegos para Game Boy Advance.
      </h2>
    </div>
  </div>
</section>

Para muchos que hemos utilizado librerías y Herramientas para crear contenido HomeBrew, normalmente se utilizaban lenguajes como C/C++, o ensamblador en algunos casos. Sin embargo, gracias a los avances de otros lenguajes como Rust, podemos utilizar las herramientas que trae para poder crear software homebrew para otras plataformas.

Muchos recordarán la consola portatil [Game Boy Advance de Nintendo](https://es.wikipedia.org/wiki/Game_Boy_Advance); una consola cuyo lanzamiento fue en 2001 hasta 2008. En este articulo, vamos a mostrar como poder utilizar el lenguaje Rust, para crear ROMS de nuestros desarrollos.

En primer lugar, vamos a explicar que es RUST y como podemos utilizar este lenguaje. Explicando las herramientas que utilizaremos y como podemos compilar para otras arquitecturas (como es el caso de la Game Boy Advance). Seguidamente, veremos un ejemplo de como aplicar este lenguaje en la creación de un juego para Game Boy Advance.

Muchos han oido hablar de Rust; un lenguaje compilado de proposito general que ha sido diseñado para ser un "lenguaje seguro, conciurrente y práctico". Este lenguaje esta siendo desarrollado por la fundación Mozilla y es un lenguaje de programación Multiparadigma.

Una de las principales caracteristicas principales de este lenguaje es que esta pensado para la eficiencia en el uso de la memoria ya que no permite por ejemplo punteros nulos o referencias vacias. Seguidamente vamos a mostrar el hola mundo en Rust:

    :::rust
    fn main(){
      println!("Hola Mundo");
    }

Como vemos, su sintaxis puede ser algo parecida a C/C++. Pero la potencia que tiene este lenguaje es que podemos usar librerias ya compiladas en C o C++ y usarlos desde programas Rust y viceversa (Usar una librería compilada con Rust desde C/C++). Además, el compilador de Rust, permite realizar compilación cruzada (Cross Compiling).Para poder instalar Rust, podemos hacerlo siguiendo las insterucciones de su [página oficial](https://www.rust-lang.org/).

Una vez visto que es Rust, vamos a entrar en materia. ¿Cómo podemos crear un juego en Game Boy Advance usando este lenguaje?. La respuesta es sencilla. Tenemos que "compilar" para esta plataforma; usando la llamada compilación cruzada. Esto quiere decir que el resultado de compilar nuestro código será un programa para una plataforma especifica. Normalmente cuando creamos y compilamos un programa, lo hacemos para la arquitectura donde estamos (normalmente x86, x86_64); pero para otros dispositivos como una Raspberry pi con una arquitectura ARM, ¿Cómo podemos hacerlo? Para ello utilizaremos el compilador de Rust especificando a que arquitectura compilar. Seguidamente mostraremos un esquema de los pasos a realizar.

<p align="center">
<img src="/static/img/esquemaxcompiling.png" alt="Esquemaxcompiling"/>
<span><bold>Esquema Compilacion Juego GBA</bold></span>
</p>

Como vemos en la imagen anterior, usaremos el compilador Rust para generar un binario para la arquitectura de la Game Boy Advance; la cual tiene un procesador ARM. Sin embargo, es necesario unas serie de herramientas extras para poder generar la Rom y ejecutar nuestro juego en un emulador o en un hardware real con un cartucho Flash. Seguidamente explicaremos cuales son estas herramientas.

Para poder generar Roms y disponer de una serie de librerias para poder compilar se utilizará el conjunto de herramientas de ```DevKitPro```; que se trata de un conjunto de herramientas y librerías, necesarias para crear software para plataformas como Nintendo DS, Game Boy Advance, Wii, etc...

Podemos ir a la [wiki de este proyecto](https://devkitpro.org/wiki/Getting_Started) para ver como instalarlo en nuestro equipo. Es importante que se instale las herramientas para GBA y se tengan las carpetas de los binarios dentro del PATH (en Windows, las carpetas ```C:\DevKitPro\DevKitARM\bin``` y ```C:\DevKitPro\tools\bin```). Una vez instalado ya podemos proceder a configurar nuestro sistema para usarlo en nuestros desarrollos.

Normalmente, cuando instalamos Rust por primera vez, nos instala la versión estable. Sin embargo, esta versión de Rust puede dar problemas por algunas funcionalidades que se necesitan para compilar. Por lo que necesitaremos activar la versión inestable (nigthly). y activar el componente ```rust-src```; para ello hay que ejecutar las siguientes ordenes:

    :::bash
    rustup default nightly
    rustup component add rust-src

Una vez activados en Rust los componentes necesarios para configurar nuestro sistema, vamos a instalar la herramienta que nos permitirá realizar compilaciones cruzadas ```xbuild``` para ello, utilizaremos el gestor de paquetes y construcción ```cargo```que viene con Rust:

    :::bash
    cargo install cargo-xbuild

Por último, para poder crear las compilaciones y ejecuciones de nuestro software de forma más fácil, utilizaremos la utilidad ```cargo-make``` para crear nuestros ficheros make para Rust.

    :::bash
    cargo install --force cargo-make

Una vez instaladas las herramientas necesarias, tanto DevKitPro como las herramientas para Rust, vamos a crear un pequeño ejemplo. Seguidamente vamos a ver como crear nuestro proyecto y compilarlo. En primer lugar, vamos a necesitar la librería propia de Rust para desarrollar para Game Boy Advance. Podéis encontrar el repositorio del proyecto con información [aquí](https://github.com/rust-console/gba).

Para comenzar nuestro proyecto necesitaremos 3 ficheros, de este proyecto mencionado:

* ```thumbv4-none-agb.json```: Este fichero describe como compilar para la arquitectura propia de la Game Boy Advance.
* ```crt0.s```: Fichero en ensablador con información inicial necesaria para compilar y crear la ROM.
* ```linker.ld```: Información necesaria para poder enlazar nuestro codigo con los distintos objetos generados y generar un único binario.

Una vez tenemos estos ficheros, vamos a generar un proyecto Rust Binario vacio; para ello utilizaremos ```cargo```.

    :::bash
    cargo init hellogba --bin

Esto generará un proyecto Rust básico con un fichero ```Cargo.toml```, con la información del proyecto y de las dependencias; este fichero lo modificaremos con la siguiente información:

    :::bash
    [package]
    name = "hellogba"
    version = "0.1.0"
    authors = ["myauthor <test@test.com>"]
    edition = "2018"

    # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

    [dependencies]
    #gba="0.3.2"
    gba={path="../gba"}

    [[bin]]

    name="hellogba"
    path="src/hellogba.rs"

Como vemos en el anterior fragmento, observareis que hay definida usando una ruta; esto puede realizarse para utilizar la ultima version del código de la libreria de GBA, sin necesidad de descargarlo del repositorio de Rust crate.io.

Una vez configurado nuestro proyecto, vamos a ver las instrucciones que hay que ejecutar para compilar y empaquetar la ROM. Pero no sin olvidar escribir nuestro código; os dejamos aquí un código de ejemplo para que podáis probarlo:

    :::rust
    #![no_std]
    #![feature(start)]
    #![forbid(unsafe_code)]

    use gba::{
      fatal,
      io::{
        display::{DisplayControlSetting, DisplayMode, DISPCNT, VBLANK_SCANLINE, VCOUNT},
        keypad::read_key_input,
      },
      vram::bitmap::Mode3,
      Color,
    };

    #[panic_handler]
    fn panic(info: &core::panic::PanicInfo) -> ! {
      // This kills the emulation with a message if we're running within mGBA.
      fatal!("{}", info);
      // If we're _not_ running within mGBA then we still need to not return, so
      // loop forever doing nothing.
      loop {}
    }

    /// Performs a busy loop until VBlank starts.
    ///
    /// This is very inefficient, and please keep following the lessons until we
    /// cover how interrupts work!
    pub fn spin_until_vblank() {
      while VCOUNT.read() < VBLANK_SCANLINE {}
    }

    /// Performs a busy loop until VDraw starts.
    ///
    /// This is very inefficient, and please keep following the lessons until we
    /// cover how interrupts work!
    pub fn spin_until_vdraw() {
      while VCOUNT.read() >= VBLANK_SCANLINE {}
    }

    #[start]
    fn main(_argc: isize, _argv: *const *const u8) -> isize {
      const SETTING: DisplayControlSetting =
        DisplayControlSetting::new().with_mode(DisplayMode::Mode3).with_bg2(true);
      DISPCNT.write(SETTING);

      let mut px = Mode3::WIDTH / 2;
      let mut py = Mode3::HEIGHT / 2;
      let mut color = Color::from_rgb(31, 0, 0);

      loop {
        // read our keys for this frame
        let this_frame_keys = read_key_input();

        // adjust game state and wait for vblank
        px = px.wrapping_add(2 * this_frame_keys.x_tribool() as usize);
        py = py.wrapping_add(2 * this_frame_keys.y_tribool() as usize);
        if this_frame_keys.l() {
          color = Color(color.0.rotate_left(5));
        }
        if this_frame_keys.r() {
          color = Color(color.0.rotate_right(5));
        }

        // now we wait
        spin_until_vblank();

        // draw the new game and wait until the next frame starts.
        if px >= Mode3::WIDTH || py >= Mode3::HEIGHT {
          // out of bounds, reset the screen and position.
          Mode3::dma_clear_to(Color::from_rgb(0, 0, 0));
          px = Mode3::WIDTH / 2;
          py = Mode3::HEIGHT / 2;
        } else {
          // draw the new part of the line
          Mode3::write(px, py, color);
          Mode3::write(px, py + 1, color);
          Mode3::write(px + 1, py, color);
          Mode3::write(px + 1, py + 1, color);
        }

        // now we wait again
        spin_until_vdraw();
      }
    }

Tras escribir nuestro código, pasaremos a generar nuestrea ROM. Primero, necesitaremos copiar y compilar el fichero de ensamblador para poder utilizarlo en nuestro desarrollo; para ello usaremos la herramienta ```arm-none-eabi-as```, para generar nuestro codigo objeto del ensamblador.

    :::bash
    arm-none-eabi-as crt0.s -o target/crt0.o

Una vez generado este objeto, ya podemos usar la herramienta de compilación cruzada ```xbuild``` especificando el target a utilizar:

    :::bash
    cargo xbuild --target thumbv4-none-agb.json [--release]

En este caso utilizaremos el parametro target para indicar el fichero json con toda la información; esto nos generará un binario ELF; el cual algunos emuladores ya podrían utilizarse para leer nuestra ROM. Pero si queremos utilizarlo en un Hadrware Real o nuestro emulador no dispone de soporte para binarios ELF, vamos a empaquetar nuestra ROM; por lo que ejecutaremos el anterior comando añadiendo el flag ```--release```.

Una vez generado el binario con la opción release, vamos a crear empaquetar la ROM; para ello utilizaremos el comando ```arm-none-eabi-objcopy``` que viene con DevKitPro para copiar los objetos necesarios y generar un fichero .gba.

    :::bash
    arm-none-eabi-objcopy -O binary target/thumbv4-none-agb/release/hellogba target/hellogba.gba

Con esto ya tenemos una ROM pero necesitaremos añadirle información a esta ROM para que funcione en el hadrware Real con un cartucho Flash. Para ello, utilizaremos otro comando de DevKitPro que nos permitirá solucionar este problema.

    :::bash
    gbafix target/hellogba.gba

Y por fin, ya tenemos nuestra ROM preparada y lista para ser ejecutada; la cual podemos ejecutar en un emulador como [Visual Boy Advance](http://www.emulator-zone.com/doc.php/gba/vboyadvance.html). Seguidamente dejamos una imagen de nuestro juego.

<p align="center">
<img src="https://raw.githubusercontent.com/zerasul/gba-rust/master/Hellogba.png" alt="rom"/>
<span><bold>Ejecución de nuestra ROM en un Emulador</bold></span>
</p>

Sin embargo, para poder ayudar con el desarrollo y poder automatizar el proceso, puede generarse un fichero ```Makefile.toml```, que nos permita generar tareas para crear nuestro juego. Os dejamos el fichero de ejemplo:

    :::
    [config]
    skip_core_tasks = true

    [tasks.prebuild]
    command = "cp"
    args =["crt0.s", "taget\\"]
    [tasks.build]
    dependencies = [
        "prebuildarm",
        "buildcargo",
        "copyelf",
        "fixsize"
    ]

    [tasks.prebuildarm]
    command = "arm-none-eabi-as"
    args = ["crt0.s","-o","target\\crt0.o"]

    [tasks.buildcargo]
    command="cargo"
    args=["xbuild", "--target", "thumbv4-none-agb.json", "--release"]

    [tasks.copyelf]
    command="arm-none-eabi-objcopy"
    args=["-O", "binary", "target/thumbv4-none-agb/release/hellogba", "target/hellogba.gba"]

    [tasks.fixsize]
    command="gbafix"
    args=["target/hellogba.gba"]


    [tasks.default]
    dependencies = [
        "prebuild",
        "build"
    ]

Para ejecutar este fichero, se utilizara la orden make seguida de la tarea.

    :::bash
    cargo make build

Os dejamos un enlace con [este ejemplo](https://github.com/zerasul/gba-rust) preparado para que podáis probarlo.

<h3 class="title is-3">Referencias </h3>

<div class="content">
  <ul>
      <li><a href="https://www.rust-lang.org/">https://www.rust-lang.org/</a></li>
      <li><a href="https://github.com/rust-console/gba">Librería GBA para Rust</a></li>
      <li><a href="https://devkitpro.org/wiki/Getting_Started">DevKitPro Wiki</a></li>
      <li><a href="https://github.com/sagiegurari/cargo-make">Cargo Make</a></li>
      <li><a href="https://github.com/zerasul/gba-rust">Ejemplo Rom</a></li>
  </ul>
</div>

<hr/>