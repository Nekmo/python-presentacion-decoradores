.. title: Decoradores en Python

:data-transition-duration: 500
:css: css/presentation.css
:css: css/monokai.css

----

:id: titulo

#####################
Decoradores en Python
#####################

----

Si has trabajado en Python, es posible que los hayas visto...

.. code-block::

    # Builtin
    @property, @classmethod
    # Django, Flask
    @login_required, @app.route
  
----

.. code-block::

    @login_required
    def profile_view(request):
        # Sólo los usuarios conectados pueden acceder aquí
        ...

----

¿Pero qué son?
==============
Los decoradores son clases o funciones que envuelven a otras clases o funciones *(valga de redundancia)*.

----

¿Qué? ¿Y eso para qué?
======================
Los decoradores le permite, sobre sus funciones y clases:

* Sustituir o cambiar su comportamiento.
* Cambiar los argumentos y parámetros de entrada, o su salida.
* Registrarlos.
    
----

Cómo usarlos
============
Mediante el símbolo arroba (``@``), seguido del decorador, encima de la clase o función

.. code-block::

    @app.route("/")
    def hello():
        return "Hello World!"
        
*Flask, un framework web, para definir las vistas usa decoradores.*

----

Pero en realidad, esto es equivalente a:

.. code-block::

    def hello():
        return "Hello world!"
    hello = app.route("/")(hello)
    
*¿Parece complicado? Aquí viene la explicación poco a poco...*

----

Cómo crearlos
=============
Un decorador es una función que envuelve a otra. Necesitamos así pues:

**Una función a decorar**

.. code-block::

    def hello():
        print("Hello world!")
        
 ----
 
 Y una función (decorador) que envolverá nuestra otra función (hello):
 
.. code-block::
 
    def decorador(f):
        return f
        
 ----
 
 ``decorador`` es una función que devuelve el valor que se le pasa por argumento.
 
 .. code-block::
 
    num = 4
    num = decorador(num)
    # ¿Esto es cierto, no?
    assert num == 4
    
----

Así pues, pasará lo mismo al introducir una función.

.. code-block::

    hello = decorador(hello)
    hello()  # Esto sigue imprimiendo "Hello world!"
    
*No obstante, esto no resulta muy útil*
    
----

Uso impráctico #2: volver loco a tus compañeros
===============================================
Nuestro decorador ``crazy`` hará que, el 50% de las veces que **se ejecute el programa**,
la función **no** haga lo esperado...

----

.. code-block::

    from random import choice

    def crazy(f):
        if choice([True, False]):
            def inquisition():
                print("Nobody expects the spanish inquisition!")
            return inquisition
        else:
            return f
                
----

.. code-block::

    # El 50% de las veces que ejecutemos esto, obtendremos una
    # función más divertida
    hello = crazy(hello)
    # Cada vez que se ejecute, obtendremos "Hello World!" o
    # una frase inquisidora, pero siempre lo mismo (hasta que
    # reiniciemos el programa):
    hello()
    
----

O usado el decorador de la forma habitual:

.. code-block::

    @crazy
    def hello():
        print("Hello world!")
    # Hello o inquisión. Siempre lo mismo.
    hello()

----

Mejorando nuestra función inútil (closure)
==========================================
El decorador ``crazy`` cambia de forma constante el comportamiento de la función. Si se establece en un comienzo que la función ``hello`` debe devolver "Hello World!", lo devolverá siempre. Y lo mismo con *inquisición*.

Ahora queremos que cambie por **cada ejecución de la función** (``hello``).

----

¿Cómo se hace?
--------------
El decorador sólo se está interponiendo en **la definición** de la función, lo cual hace que sólo cambie su comportamiento cuando se establece, pero no **por cada ejecución**. Afectaremos también a **su ejecución**.

----

.. code-block::

    def crazy(f):
        def ejecucion():
            if choice([True, False]):
                def inquisicion():
                    print("Nobody expects the spanish inquisition!")
                return inquisicion()
            else:
                return f()
        return ejecucion
        
----

.. code-block::

    hello = crazy(hello)
    # El 50% de las veces que **se ejecuta** la función, será
    # más divertido
    hello()

----

