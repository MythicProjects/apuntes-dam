---
tags:
  - desarrollo-de-interfaces
  - DAM2
unidad: 1
tema: Arquitectura de software
---
# Arquitectura de software

La arquitectura soluciona muchos problemas, ya está probado y no se debe inventar. Existen arquitecturas conocidas y muy utilizadas.

// Es necesario un repaso de UML

## Arquitectura MVC
- El modelo es donde está el proceso de cálculos
- En la vista está toda la parte de presentación. Todas las clases de presentación van allí
- El controlador se encarga de controlar las otras dos V<--->C<--->M pero también V<---->M
- Esa conexión V<->M aporta flexibilidad pero no es muy estable y es una de las principales diferencias con MVP 

## Arquitectura MVP
- Añade reglas adicionales al MVC
- V <--> P <--> M
- El presentador se encarga de organizar la ejecución
- Se implementan interficies e inyección de dependencias
- Las interfaces son útiles aquí porque obligas a la clase a usar los métodos mínimos de dichas interfaces
- La inyección de dependencias (la clase se pasa por parámetro en vez de estar ligado)
- No ayuda al modelo a organizarse correctamente

Ejemplo de inyección de dependecias:
```java
interface Animal {
    void hacerSonido();
}

class Perro implements Animal {
    public void hacerSonido() {
        System.out.println("Guau");
    }
}

class Gato implements Animal {
    public void hacerSonido() {
        System.out.println("Miau");
    }
}

public class Main {
    // El parámetro es de tipo interfaz
    static void escuchar(Animal animal) {
        animal.hacerSonido();
    }

    public static void main(String[] args) {
        escuchar(new Perro()); // Guau
        escuchar(new Gato());  // Miau
    }
}
``` 

--- 
>[!info]
### Acceso a miembros a través de una referencia de tipo interfaz

Cuando un parámetro se declara con el tipo de una interfaz, el compilador solo permite acceder a los miembros definidos en dicha interfaz, con independencia de la clase concreta del objeto recibido. Esta restricción se debe a que la comprobación de tipos se realiza sobre el **tipo estático** (el declarado), no sobre el **tipo dinámico** (el del objeto en tiempo de ejecución).

```java
interface Animal {
    void hacerSonido();
}

class Perro implements Animal {
    public void hacerSonido() { System.out.println("Guau"); }
    public void traerPelota() { System.out.println("Pelota"); }
}

static void usarAnimal(Animal a) {
    a.hacerSonido();     // Válido: miembro de la interfaz
    // a.traerPelota();  // Error de compilación: no pertenece a Animal
}
```

### Consideraciones

1. **El objeto no se modifica.** La instancia conserva todos sus atributos y métodos; la referencia de tipo interfaz únicamente restringe cuáles son accesibles.
2. **Enlace dinámico.** La invocación `a.hacerSonido()` ejecuta la implementación de la clase concreta (`Perro`). Este comportamiento constituye el polimorfismo de inclusión.
3. **Conversión explícita.** Es posible acceder a los miembros específicos mediante una conversión descendente (_downcasting_), previa verificación del tipo (`instanceof` en Java, `is` en C#). Su uso frecuente suele indicar un diseño de interfaz inadecuado.

Este mecanismo favorece el **desacoplamiento**: el método depende de un contrato abstracto y no de una implementación concreta, lo que permite reutilizarlo con cualquier clase que implemente la interfaz.

**Síntesis:** el tipo estático de la referencia determina qué miembros son accesibles; el tipo dinámico del objeto determina qué implementación se ejecuta.

--- 


## Arquitectura Clean

## Arquitectura Hexagonal