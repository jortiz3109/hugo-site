---
title: "Complejidad ciclomática"
date: 2022-07-28
tags: [quality, software, metrics]
categories: ["software quality"]
draft: false
showTableOfContents: true
summary: La Complejidad Ciclomática (en inglés, Cyclomatic Complexity) es una métrica del software en ingeniería del software que proporciona una medición cuantitativa de la complejidad lógica de un programa.
---
## Complejidad ciclomática
La complejidad ciclomática es la métrica de calidad de software de mayor aceptación, principalmente por su independencia del lenguaje de programación.

El objetivo principal de esta métrica no es contar la cantidad de ciclos y estructuras de control, sino contar la cantidad de caminos diferentes que existen en un fragmento de código.

## Historia
Thomas McGabe, propone la complejidad ciclomática en su artículo *[A Complexity Measure](http://www.literateprogramming.com/mccabe.pdf)* publicado en 1976 cómo respuesta a la siguiente pregunta:
***"¿Cómo modularizar un sistema de software para que los módulos resultantes sean testeables y mantenibles?"***.

McGabe basaría su propuesta en un diagrama de flujo, el cual obtenía utilizando las estructuras de control en el código como insumo para calcular una medida cuantitativa de la dificultad para crear pruebas automatizadas, así como la confiabilidad del mismo.

## Para que sirve
Calcular la complejidad ciclomática sirve para resolver varios retos a los cuales nos enfrentamos al momento de escribir una nueva funcionalidad:

 * Permite obtener el número de caminos independientes dentro de un fragmento de código por los cuales se puede resolver una petición (determina cuan complejo es un fragmento de código).
 * Sirve como insumo para determinar el número de pruebas necesarias para asegurar que se ejecuta cada camino al menos una vez.
 * Ayudar a estimar el riesgo y costo de mantener un sistema de software.
 * Sirve como medida para determinar que tan bien escrito está el código que compone un sistema de software.

## Cómo se calcula

Para calcular la complejidad ciclomática se utiliza la siguiente fórmula:
```
M = E − N + 2
```

Donde:
**M** = Complejidad ciclomática.
**E** = Número de aristas del grafo.
**N** = Número de nodos del grafo correspondientes a sentencias del programa.

Tomemos como ejemplo la siguiente función:
```php
public function compareNumbers(int $num1, int $num2): string
{
    if ($num1 > $num2) {
        return "{$num1} is greather than {$num2}";
    } else if ($num2 > $num1) {
        return "{$num1} is less than {$num2}";
    } else {
        return "{$num1} and {$num2} are equals";
    }
}
```

Construimos el grafo de flujo de la función:
{{<
    figure src="ciclomatic-complexity-graph.jpg"
    alt="Grafo de la función"
    >}}

Aplicamos la fórmula para obtener la complejidad ciclomática:
```
M = E − N + 2
```

**E** = 8

**N** = 7

```
M = 8 − 7 + 2
M = 3
```

La complejidad ciclomática de la función compareNumbers es de **3**.

## Cómo se interpreta
Una vez calculamos la complejidad ciclomática, podemos determinar el riesgo de un fragmento de código usando la siguiente tabla:

| Complejidad ciclomática | Nivel de riesgo               |
|--------------------------|-------------------------------|
| 1-10                     | Simple, poco riesgo           |
| 11-20                    | Más complejo, riesgo moderado |
| 21-50                    | Complejo, alto riesgo         |
| 50                       | No testeable, Muy alto riesgo |

## Consejos
¿Cómo puedo reducir la complejidad ciclomática? aquí te doy algunos consejos:

 * Respetar el principio de responsabilidad única.
 * Evitar el uso de if anidados.
 * Evitar el uso de switch.
 * Mantener un número bajo de líneas de código.
 * Extraer grandes bloques de código a sus propias funciones.

## Conclusiones

 * Algunos estudios indican que existe una correlación entre una alta complejidad ciclomática y un alto número de defectos en un sistema de software.
 * Mantener una complejidad ciclomática alta puede aumentar significativamente el tiempo requerido para encontrar y corregir errores en nuestro código.
 * Mantener una complejidad ciclomática alta hace mas difícil entender, probar, mantener y refactorizar el código.
 * Una complejidad ciclomática alta hace que sea mas fácil introducir errores en el código.

