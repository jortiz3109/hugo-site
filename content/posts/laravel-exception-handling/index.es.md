---
title: 'Manejo de excepciones en Laravel'
date: 2022-08-07
draft: false
tags: ['exceptions']
categories: ['PHP', 'Laravel']
showTableOfContents: true
---
Hoy vamos a conversar un poco sobre el manejo de excepciones en Laravel.
<!--more-->

## Excepciones
Un correcto manejo de excepciones permite que tu aplicación sea robusta, tolerante a fallos y amigable con el usuario final. Pero por el contrario: un incorrecto manejo puede hacer un infierno la gestión de errores en cualquier aplicación.

Antes de comenzar, es necesario hacer un poco de historia y recordar que es una excepción.

### ¿Qué es una excepción?
Podemos definir una excepción como la ocurrencia de un error durante la ejecución de un programa que impide su normal funcionamiento y cuya particularidad es que aparece con poca frecuencia.

Por ende y debido a la excepcionalidad con la que ocurren dichos errores; como equipo de desarrollo estamos en la obligación de gestionar correctamente las excepciones dentro de nuestra aplicación.

## Manejo de excepciones en Laravel

La [Wikipedia](https://es.wikipedia.org/wiki/Manejo_de_excepciones) define el manejo de excepciones como:

> Es una técnica de programación que permite al programador controlar los errores ocasionados durante la ejecución de un programa informático. Cuando ocurre cierto tipo de error, el sistema reacciona ejecutando un fragmento de código que resuelve la situación, por ejemplo retornando un mensaje de error o devolviendo un valor por defecto. 

### Importancia

Retomando: Del correcto manejo de excepciones depende que tus aplicaciones sean tolerantes a fallos, amigables con el usuario final y fáciles de debugar para el equipo de desarrollo.

### Cómo funcionan las excepciones en Laravel
[Laravel](https://www.laravel.com) viene preparado de serie con un excelente manejador de excepciones. Por defecto todas las excepciones que arroja el framework son manejadas mediante la clase ```App\Exceptions\Handler```. Ésta clase gestiona la escritura en logs y el renderizado de la respuesta que recibirá el usuario final.

#### Valores de configuración
La configuración relacionada con la gestión de excepciones se puede controlar mediante la opción ```debug``` en el archivo ```config/app.php```. Dicha opción se parametriza utilizando la variable de entorno ```APP_DEBUG```. Si en nuestro entorno (cualquiera que sea) configuramos dicha variable a ```true``` nuestras excepciones mostrarán la mayor cantidad de Información acerca del error. Es recomendable que en entornos de producción su valor sea siempre ```false```.

#### Reporte de excepciones
Antes de entrar en detalle, es pertinente aclarar algunos conceptos:
> Reporte de excepciones se refiere a la lógica de escribir los detalles del error a log y/o enviarlos a servicios externos de gestioń de errores *([Sentry](https://sentry.io/), [Bugsnag](https://www.bugsnag.com/), etc)*. Mientras renderizado de excepciones se refiere a la lógica responsable de mostrar un error amigable al usuario final.

Aunque la clase ```App\Exceptions\Handler``` contiene el método ```register()``` en el cual se puede personalizar la lógica del reporte y renderizado para las excepciones de nuestra aplicación; siempre es recomendable personalizar dicho comportamiento directamente en nuestras clases de excepción personalizadas.

{{< highlight php >}}
<?php
 
namespace App\Exceptions;
 
use Log;
use Exception;
use Illuminate\Http\Response;
use Illuminate\Http\Request;
 
class InvalidProductException extends Exception
{

    public function report(): ?bool
    {
        Log::info('Invalid product exception');
    }
 
    public function render(Request $request): Response
    {
        return response('The required product is not valid for this operation');
    }
}
{{< /highlight >}}


##### Ignorando excepciones
En ocasiones podrías requerir que en tu aplicación algunas excepciones sean ignoradas y nunca reportadas. Esto puede configurarse usando la propiedad ```$dontReport``` de la clase ```App\Exceptions\Handler``` así:

{{< highlight php >}}
<?php
 
namespace App\Exceptions;

use Psr\Log\LogLevel;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    protected $dontReport = [
        InvalidProductException::class,
    ];
}
{{< /highlight >}}

> Aunque técnicamente es viable, la recomendación es **nunca** ignorar excepciones. Esto podría ocultar información importante acerca del comportamiento de tu aplicación al equipo de desarrollo.

#### Información de contexto
La Información de contexto ayuda al equipo de desarrollo a identificar mucho más rápido la causa de un error en nuestra aplicación. Laravel automáticamente incluirá el ID del usuario autenticado al contexto de la excepción. Podemos agregar información de contexto de manera global modificando la clase ```App\Exceptions\Handler``` así:

{{< highlight php >}}
use Illuminate\Support\Arr;

protected function context(): array
{
    return Arr::add(parent::context(), 'foo', 'bar');
}
{{< /highlight >}}


En ocasiones podemos encontrarnos con el escenario en el que debemos personalizar la información de contexto en nuestras excepciones, esto podemos hacerlo sobreescribiendo el método ```context()```.

{{< highlight php >}}
<?php
 
namespace App\Exceptions;
 
use Exception;
use App\Models\Product;

class InvalidProductException extends Exception
{
    public Product $product;

    public function context(): array
    {
        return [
            'product' => $this->product->toArray()
        ];
    }
}
{{< /highlight >}}

#### Niveles de severidad

Los niveles de severidad toman especial relevancia cuando evidenciamos que no todas las excepciones tienen la misma severidad ni el mismo impacto en el flujo de nuestra aplicación.

> Laravel ofrece soporte para todos los niveles de severidad definidos en la especificación: [RFC 5424 - The Syslog Protocol](https://www.rfc-editor.org/rfc/rfc5424.html).
> **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, y **debug**.

Podemos indicar el nivel de severidad usando las constantes definidas en la clase ```Psr\Log\LogLevel``` para personalizar la propiedad ```$levels``` de la clase ```App\Exceptions\Handler``` así:

{{< highlight php >}}
<?php
 
namespace App\Exceptions;

use Psr\Log\LogLevel;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    protected $levels = [
        InvalidProductException::class => LogLevel::NOTICE,
    ];
}
{{< /highlight >}}

## Conclusiones

Laravel ofrece un simple, pero poderoso sistema de gestión de excepciones, que te permitirá controlar de manera sencilla cualquier error que ocurra en tu aplicación, agregar información de contexto y finalmente ofrecer una experiencia mas gratificante a los usuarios de tu aplicación.

## Recursos
Finalmente dejo a tu disposición algunos recursos que pueden serte de utilidad si quieres profundizar en la gestión de errores que ofrece Laravel.

* [Documentación oficial de Laravel](https://laravel.com/docs/9.x/errors)