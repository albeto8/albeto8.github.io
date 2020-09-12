---
layout: post
title:  "Dispatch groups"
author: "Mario Barragán"
date:   2020-09-12 12:47:28 -0500
categories: concurrency
---

# DispatchGroup en Swift

Hola amigos, en este artículo explicaremos un poco qué son los DispatchGroups, para qué funcionan y programaremos una pequeña app de ejemplo.


Los DispatchGroups son muy usados en iOS sobretodo en networking cuando queremos hacer múltiples llamadas a algún API. Dentro de un DispatchGroup podemos ejecutar varias tareas sincronas y/o asincronas y saber el momento exacto en el que todas estás tareas han terminado.

Para que quede más claro, vamos al código, primero emepzaremos creando un proyecto nuevo:

Lo primero que haremos es dentro de nuestro controller definir un arreglo de URLs de los sprites de pokemones del Api de pokemon pokeapi.co

{% highlight swift %}
    let urls = [
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/1.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/2.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/3.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/4.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/5.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/6.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/7.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/8.png")!,
        URL(string: "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/9.png")!
    ]
}
{% endhighlight %}

Lo siguiente que haremos sera crear una función que utilizaremos para descargas los sprites de pokemon 

{% highlight swift %}
func fetchPokemonStarters() {
    // Más código a continuación
}
{% endhighlight %}

Dentro de la función crearemos un DispatchGroup de la siguiente forma:

{% highlight swift %}
fetchPokemonStarters() {
    let group = DispatchGroup()
}
{% endhighlight %}

Ahora vamos a crear un for loop para iterar a través de nuestro arreglo de URLs, especial atención en el enumerated() lo usaremos para obtener el índice del arreglo

{% highlight swift %}
func fetchPokemonStarters() {
        let group = DispatchGroup()
        
        for (index, url) in urls.enumerated() {
        
        }
    }
{% endhighlight %}

Cada vez que entramos al for vamos a ejecutar group.enter() para notificar manualmente al grupo que una tarea ha comenzado.

{% highlight swift %}
func fetchPokemonStarters() {
        let group = DispatchGroup()
        
        for (index, url) in urls.enumerated() {
            group.enter()
            print("Entrando a dispatchGroup con indice: \(index)")
        }
    }
{% endhighlight %}

Vamos a comenzar a descargar las imágenes para ello usaremos URLSession.shared.dataTask(with: url), que no se nos vaya a olvidar de añadir el .resume() al final

{% highlight swift %}
func fetchPokemonStarters() {
        let group = DispatchGroup()
        
        for (index, url) in urls.enumerated() {
            group.enter()
            print("Entrando a dispatchGroup con indice: \(index)")
            
            URLSession.shared.dataTask(with: url) { (data, response, error) in
                
            }.resume()
        }
    }
{% endhighlight %}

Y ahora por simplicidad solo verificaremos si hubo error en la descarga

{% highlight swift %}
func fetchPokemonStarters() {
        let group = DispatchGroup()
        
        for (index, url) in urls.enumerated() {
            group.enter()
            print("Entrando a dispatchGroup con indice: \(index)")
            
            URLSession.shared.dataTask(with: url) { (data, response, error) in
                if let error = error {
                    print("Error al descargar \(error.localizedDescription)")
                }
            }.resume()
        }
 }
{% endhighlight %}



Con group.leave() podemos a indicar explícitamente que la tarea ha terminado.

El bloque de código dentro de defer será lo último que se ejecutará dentro del scope actual por lo que es un buen lugar para colocar group.leave().

{% highlight swift %}
func fetchPokemonStarters() {
        let group = DispatchGroup()
        
        for (index, url) in urls.enumerated() {
            group.enter()
            print("Entrando a dispatchGroup con indice: \(index)")
            
            URLSession.shared.dataTask(with: url) { (data, response, error) in
                defer {
                    group.leave()
                    print("Saliendo de dispatchGroup del con indice \(index)")
                }
                if let error = error {
                    print("Error al descargar \(error.localizedDescription)")
                }
            }.resume()
        }
 }
{% endhighlight %}

Con group.notify podemos saber el momento exacto en el que todas las descargas terminaron 


En nuestro caso como vamos a actualizar la UI queremos que notify se ejecute en el main thread 

{% highlight swift %}
group.notify(queue: .main)
{% endhighlight %}

{% highlight swift %}
func fetchPokemonStarters() {
        let group = DispatchGroup()
        
        for (index, url) in urls.enumerated() {
            group.enter()
            print("Entrando a dispatchGroup con indice: \(index)")
            
            URLSession.shared.dataTask(with: url) { (data, response, error) in
                defer {
                    group.leave()
                    print("Saliendo de dispatchGroup del con indice \(index)")
                }
                if let error = error {
                    print("Error al descargar \(error.localizedDescription)")
                }
            }.resume()
        }
        
        group.notify(queue: .main) {
            //  Update UI
            print("Todas las descargas terminaron")
            self.view.backgroundColor = .blue
        }
    }
{% endhighlight %}


Si ejecutamos la aplicación podemos ver que la pantalla vuelve azul al momento que todas las descargas terminan

![Image of simulator](/assets/dispatchGroups/simulator.png)

Si vemos nuestra consola:
![image](/assets/dispatchGroups/console.png)

Aquí podemos observar como las descargas comienzan ordenadas pero al momento de finalizar no está garantizado que terminaran en el mismo orden en el que comenzaron.


https://developer.apple.com/documentation/dispatch/dispatchgroup
