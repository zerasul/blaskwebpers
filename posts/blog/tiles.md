---
date: 2020-05-14
title: Creando niveles de Juegos Con Tiled y Mega Drive | Victor Suarez
template: blog.html
author: Victor
category: articulo
tags: megadrive,sgdk, Tiled,tilemap,tileset
---

<section class="hero">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        Creando niveles de Juegos Con Tiled y Mega Drive
      </h1>
      <h2 class="subtitle">
        Crear tus propios niveles para Mega Drive
      </h2>
    </div>
  </div>
</section>

A la hora de desarrollar un videojuego, es esencial poder tener distintas herramientas; no solo de programación, sino por ejemplo de diseño de niveles o mapas. Como puede ser el caso de la herramienta [Tiled](https://www.mapeditor.org/); esta herramienta permite crear niveles a partir de una paleta de elementos que conformaran nuestro juego.

Este tipo de herramientas se basa en el uso del comunmente conocido _"Tileset"_. Un Tileset (tambien llamado [teselas](https://es.wikipedia.org/wiki/Tesela)) es un conjunto de elementos comunmente llamados Tiles (o Tesela); que componen todos los elementos gráficos que pueden aparecer en nuestro juego (o una parte de ellos). Por ejemplo:

<p align="center">
<img src="/static/img/sonic1tileset.png" alt="Sonic 1 Tileset" />
<span><bold>Parte del Tileset de Sonic 1</bold></span>
</p>

Como vemos en la anterior imagen, vemos cada una de las partes que puede componer el primer nivel del más que conocido videojuego _Sonic the hedgehog_. Vemos que se compone de una serie de elementos que pueden contener este primer nivel. Con Tiled, podemos generar un mapa a partir de estos elementos. Como por ejemplo:

<p align="center">
<img src="/static/img/tiled.png" alt="Editor Tiled" />
<span><bold>Editor Tiled</bold></span>
</p>

En este anterior ejemplo, vemos como se genera el mapa a partir del Tileset que se ha utilizado creando parte de la _Green Hill zone_; por lo que podriamos utilizar este mapa para generar nuestro primer nivel de nuestro videojuego. Pero ¿Cómo podemos utilizar esta herramienta para exportar estos mapas para usarlo en nuestros juegos para Mega Drive?

Para ello, vamos primero a guardar nuestro mapa. Tiled por defecto, tiene su propio formato llamado TMX para almacenar la información tanto del TileSet como de nuestro mapa; aunque permite exportar este formato a otros como puede ser a JSON. Si abrimos un fichero _.tmx_ con un editor texto, veremos un fichero en formato XML con la información parecida a la siguiente:

    :::xml
    <map version="1.2" tiledversion="1.3.4" orientation="orthogonal" renderorder="right-down" width="40" height="28"
    tilewidth="16" tileheight="16" infinite="0" nextlayerid="2" nextobjectid="1">
    <tileset firstgid="1" source="Green Hill TileSet 1.tsx"/>
    <layer id="1" name="Capa de patrones 1" width="40" height="28">
    <data encoding="csv">
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
    ...
    65,66,67,66,67,66,67,66,67,66,67,66,67,58,59,78,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,23,
    85,86,87,86,87,86,87,86,87,86,87,86,87,78,79,23,23,23,23,23,23,23,23,23,23,23,23,
    ...

Podemos observar algunos atributos dentro de este xml, como el ancho y alto (en tiles), el ancho y alto de cada tile (16x16), y además vemos una etiqueta data que tiene una serie de numeros separados por comas (en formato csv). Estos números corresponden al índice de cada elemento en nuestro tileset; de esta forma cada Tile en nuestro mapa esta identificado por el índice de cada elemento dentro del tileset.

Gracias a esto, podemos hacer niveles dinámicos y mucho mayores ya que ahorramos mucha memoria y procesamiento si tuviesemos que cargar todos los elementos de un mapa. En este artículo, vamos a ver como utilizar esta técnica, para poder pasar la información de un TMX a un videojuego de Mega Drive usando el kit de Desarrollo [SGDK](https://github.com/Stephane-D/SGDK).

Vamos a ver como realizar esto, usando un ejemplo; Utilizaremos el siguiente TileSet:

<p align="center">
<img src="/static/img/naturetileset.png" alt="Ejemplo Tileset" />
<span><bold>Ejemplo Tileset</bold></span>
</p>

Con este TileSet, vamos a crear un mapa utilizando Tiled; sin embargo, para poder utilizarlo para Mega Drive, tenemos que tener en cuenta las siguientes restricciones:

1. No puede tener transparencias; sino que se debe definir un color transparente
2. Debe ser una imagen indexada con una paleta de 16 colores. El color transparente siempre será el primer color de la paleta.
3. Cada Tile medira 8x8 pixels.

Podemos utilizar herramientas como [GIMP](https://www.gimp.org/); para pasar la imagen a indexada y definir el color transparente. Quedando el TileSet de la siguiente forma:

<p align="center">
<img src="/static/img/naturetlsetmd.png" alt="Tileset Modificado" />
<span><bold>Tileset Modificado</bold></span>
</p>

**NOTA:** Dejaremos un enlace de descarga en las referencias con el fichero modificado.

Una vez tenemos nuestro tileset preparado, es hora de crear nuestro mapa; de tal forma que utilizaremos este tileset para crear un fichero tmx con nuestro mapa. Debemos definir en el mapa y al importar el Tileset que nuestro el tamaño de cada Tile es de 8x8 pixels y además podemos definir el color transparente (en este caso es verde). Crearemos el siguiente mapa:

<p align="center">
<img src="/static/img/ejemplomapa.png" alt="Mapa Ejemplo" />
<span><bold>Mapa Ejemplo</bold></span>
</p>

Una vez generado el mapa, vamos a pasar a la programación de nuestro juego. Como hemos comentado, vamos a utilizar el kit de Desarrollo SGDK, para crear nuestro juego para Mega Drive. Si queréis aprender como hacerlo os dejamos un enlace a un [tutorial](https://zerasul.github.io/taller-megadrive/), donde se explica como crear videojuegos de Mega Drive con SGDK. Además de utilizar una extensión para Visual Studio Code llamada [Genesis Code](https://marketplace.visualstudio.com/items?itemName=zerasul.genesis-code).

En este caso, vamos a crear un nuevo proyecto e importar 2 recursos; el primero, será el propio TileSet (no el mapa) que hemos generado y el segundo cargaremos una imagen que será el fondo. Para ello, en nuestro proyecto recien creado, usaremos un fichero ```.res```, para definir estos recursos dentro de la carpeta ```res```.

    :::bash
    TILESET tileset "tilesetOpenGame.png" 0
    PALETTE tileset_pal "tilesetOpenGame.png" 0
    IMAGE bg_b "bga.bmp" 0

Como vemos hemos definido 3 recursos; el primero es el propio TileSet; que definimos con el nombre ```tileset``` (el flag 0 indica que no usaremos compresion); el segundo es la paleta asociada al tileset que definimos con el nombre ```tileset_pal``` y por último una imagen que será el fondo.

Aunque ya tenemos definidos los recursos, aun nos falta importar a nuestro proyecto la información del fichero TMX a nuestro juego; en este caso y solo como demostración, se realizará de forma manual esta importación; en este caso, vamos a crear un fichero ```.h``` que almacenará toda la información de nuestro mapa. En este caso solo necesitaremos para este ejemplo los elementos que estan dentro de la etiqueta ```<data>``` en el fichero TMX. Copiaremos el contenido de todos los numeros separados por comas a un nuevo fichero .h. Teniendo el siguiente formato.

    :::c
    #ifndef _INC_TILEMAP_H_
    #define _INC_TILEMAP_H_

    const u16 tilemap_tiles[1120] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1... };
    #endif

El número de elementos en el array es 1120 que corresponde al numero de datos almacenados en el TMX. Este fichero lo guardaremos en la carpeta ```inc``` de nuestro proyecto de SGDK (aunque también puede usarse la carpeta ```res```). Una vez almacenada la información del mapa ya podemos crear nuestro juego creando el siguiente código:

    :::c
    #include <genesis.h>
    #include "gfx.h"
    #include "tilemap.h"

    int main()
    {
        u8 i, j;
        u16 ind = TILE_USERINDEX;
        VDP_setScreenWidth320();
        VDP_loadTileSet(&tileset, ind, DMA);
        VDP_setPalette(PAL2, tileset_pal.data);

        for (i = 0; i < 40; i++)
        {
            for (j = 0; j < 28; j++)
            {
                VDP_setTileMapXY(BG_A, TILE_ATTR_FULL(PAL2, FALSE, FALSE, FALSE, ind + tilemap_tiles[i + 40 * j]), i, j);
            }
        }
        ind += tileset.numTile;
        VDP_drawImageEx(BG_B, &bg_b, TILE_ATTR_FULL(PAL1, FALSE, FALSE, FALSE, ind), 0, 0, TRUE, TRUE);
        ind += bg_b.tileset->numTile;
        while (1)
        {
            VDP_waitVSync();
        }
        return (0);
    }

Veamos con detalle este código; en primer lugar, añadimos la información de la librería de SGDK ```genesis.h``` y seguidamente importamos tanto nuestros recursos con el fichero ```gfx.h``` (este fichero se genera con la información definida en el fichero ```.res```) y el fichero ```tilemap.h```que contiene la información del mapa.

Tras tener todos los recursos y el mapa includos, vamos a inicializar el sistema; por lo que vamos a cargar el _TileSet_ y la paleta de este. Para ello se utilizan las funciones ```VDP_loadTileSet``` y ```VDP_setPalette``` que la primera carga el tileset en la memoria del VDP (Chip encargado de mostrar los graficos por pantalla); y la segunda carga la paleta en la ```PAL_2``` (Se permite cargar hasta 4 paletas de 16 colores).

Antes de continuar explicando el código, quiero hablar de como almacena y se muestran las imagenes en el VDP; el VDP almacena y muestra por pantalla en Tiles de 8x8 píxeles; por lo que tenemos que trabajar en esta unidad; muchos emuladores como _Gens KMod_ permiten ver la memoria del VDP y ver los tiles que hay cargados. Cada tile tiene asociado un índice que puede ser referenciado para cargar y mostrar por pantalla. Este indice lo podemos ver en nuestro código (variable ```ind```).

<p align="center">
<img src="/static/img/townquest.png" alt="GensKmod VDP" />
<span><bold>Gens Kmod mostrando los Tiles en el VDP</bold></span>
</p>

Una vez visto como cargar tanto el TileSet como la paleta, vamos a mostrar por pantalla nuestro mapa; usando cada uno de los elementos de este como índice de cada Tile. Usaremos la función ```VDP_setTileMapXY``` para establecer cada Tile en Pantalla. Podemos observar que se usa una constante llamada ```BG_A```(en versiones anteriores a SGDK 1.50 se define como ```PLAN_A```); esto indica el plano o fondo a utilizar ya que Mega Drive permite mostrar 2 fondos a la vez.

Veamos en más detalle la utilización de esta función:

    :::c
    for (i = 0; i < 40; i++)
        {
            for (j = 0; j < 28; j++)
            {
                VDP_setTileMapXY(BG_A, TILE_ATTR_FULL(PAL2, FALSE, FALSE, FALSE, ind + tilemap_tiles[i + 40 * j]), i, j);
            }
        }

Vemos los dos bucles for con limite de 40 y 28 esto indica el número de Tiles que se muestran por pantalla (La Mega Drive en formato PAL tiene una resolución de 320x224). Seguidamente se usa la función ```VDP_setTileMapXY``` para ir mostrando cada Tile en su posición; que usaremos el fondo A. Después se usa la macro ```TILE_ATTR_FULL``` para indicar en cada caso que Tile va a mostrarse. Veamos los parametros de esta Macro:

1. El primer parametro indica la paleta a utilizar; usaremos la ```PAL_2``` que es la que hemos cargado la información de la paleta del TileSet.
2. El segundo parametro indica si se dibujará con prioridad.
3. El tercero indica si se volteara Verticalmente.
4. El cuarto indica si se Voletara Horizontalmente.
5. Indica el indice del Tile a mostrar por pantalla. En este caso usaremos el indice base (Variable ```ìnd```) y el valor del array que corresponde a esta posicion (definido por i+ 40(tiles por linea)*j).

Por último, cargaremos en el Fondo B, la imagen de fondo usando la función ```VDP_drawImageEx```. El fondo B se dibuja en baja prioridad detras del A; el cual al tener definido un color transparente permite ver este fondo; dando la sensación de profundidad.

Si compilamos nuestro juego y lo ejecutamos en un emulador, lo podemos ver más o menos de la siguiente forma (algunas partes no se ven correctamente debido a algunos indices incorrectos).

<p align="center">
<img src="/static/img/tileejemplo.png" alt="Ejemplo con Tiles" />
<span><bold>Ejemplo Final de nuestro Mapa</bold></span>
</p>

**NOTA:** Se esta trabajando en automatizar la importación del fichero TMX al fichero .h usando el plugin Genesis Code. Añadiremos más información, cuando este listo.

<h3 class="title is-3">Referencias </h3>

<div class="content">
  <ul>
      <li><a href="https://github.com/Stephane-D/SGDK">https://github.com/Stephane-D/SGDK</a></li>
      <li><a href="https://opengameart.org/content/sonic-style-tiles">Ejemplo Sonic TileSet</a></li>
      <li><a href="https://opengameart.org/content/nature-tileset">Ejemplo TileSet. Creditos Al Autor</a></li>
      <li><a href="/static/img/naturetlsetmd.png" download>Descarga TileSet modificado para Mega Drive</a></li>
      <li><a href="https://www.mapeditor.org/">Tiled Página Web</a></li>
      <li><a href="https://www.gimp.org/">Gimp Página Web</a></li>
      <li><a href="https://segaretro.org/Gens_KMod">https://segaretro.org/Gens_KMod</a></li>
      <li><a href="https://marketplace.visualstudio.com/items?itemName=zerasul.genesis-code">https://marketplace.visualstudio.com/items?itemName=zerasul.genesis-code</a></li>
      <li>Grupo de Desarrolladores Mega Drive Dev de telegram</li>
  </ul>
</div>

<hr/>