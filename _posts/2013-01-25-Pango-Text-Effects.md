---
layout: post
title:  Efectos de texto con Pango
author: Ismael
---

Pango es una libreria de renderizado y diagramación de texto. Esta
disponible para Linux, Windows y Mac, opcionalmente puede usar las APIs 
nativas de cada plataforma, o su propio motor de diagramación.
Suele usarse en conjunto con la librería de rasterización Cairo.

## Creando el contexto de Cairo

Creamos un superficie cairo donde vamos a dibujar nuestro resultado,
y obtenemos un contexto que vamos a usar para dibujar.

```c++
// superficie donde vamos a dibujar
cairo_surface_t *surface;
surface = cairo_image_surface_create(CAIRO_FORMAT_ARGB32, 640, 480);
// contexto de cairo usado para dibujar
cairo_t *context;
context = cairo_create(surface);
```

## Inicializando el _layout_ de pango

Ahora creamos un _layout_ de pango en nuestro contexto y establecemos
el texto a renderear, la fuente a usar y el tamaño.

```c++
PangoLayout *layout;
layout = pango_cairo_create_layout(context);

// texto a renderear, tiene que estar en utf-8
pango_layout_set_text(layout, "Hola mundo!", -1);
```

Usamos una cadena de texto para describir la fuente que queremos usar.
El formato es "Fuente Estilo Tamaño".

```c++
// fuente a usar con el texto
PangoFontDescription *desc;
desc = pango_font_description_from_string("Arial Bold 36");
pango_layout_set_font_description(layout, desc);
```

Establecemos el ancho del cuadro de texto, y actualizamos 

```c++
// el parámetro esta en unidades de PANGO_SCALE
pango_layout_set_width(layout, 600 * PANGO_SCALE);

// actualizamos el layout para que este listo para renderear
pango_cairo_update_layout(context, layout);
```

## Renderizado del _layout_

Para renderizar primero indicamos a cairo que vamos a crear un 
_path_ nuevo y ubicamos el comienzo del mismo

```c++
cairo_new_path(context);
cairo_move_to(context, 15.0, 10.0);
```

Ahora podemos renderizar el _layout_ de pango

```c++
pango_cairo_layout_path(context, layout);
```

Rellenamos el _path_ creado para que el texto sea visible

```c++
cairo_set_source_rgb(context, 0.9, 0.1, 0.8);
cairo_fill(context);
```

## Agregando borde al texto

Para agregar borde al texto, antes de pintar el relleno,
dibujamos una línea con ancho igual al borde que deseamos.

```c++
cairo_set_line_width(context, 6.0);   // ancho del borde
cairo_set_miter_limit(context, 2.0);

cairo_set_source_rgb(context, 0.0, 0.3, 0.9);
cairo_stroke_preserve(context);
```

Usamos `cairo_stroke_preserve` para dibujar la línea y
conservar el _path_ en el contexto de cairo. De ese modo podemos
volver a usar el _path_ para pintar el relleno.

## Efecto de sombra

Una forma sencilla de obtener el efecto sombra es pintar el texto
en color negro en un nuevo grupo del contexto cairo, y luego 
pintar dicho grupo con un alfa y un dezplazamiento para obtener
la sombra.

```c++
cairo_push_group(context);

// dibujamos el texto igual que antes

cairo_pop_group_to_source(context);
cairo_paint_with_alpha(context, 0.6);
```
