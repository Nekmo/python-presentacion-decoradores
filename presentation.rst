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
 
 **Y una función (decorador)** que envolverá nuestra otra función (hello):
 
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
    # Imprime Hello o inquisión. Siempre lo mismo.
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
                return print("Nobody expects the spanish inquisition!")
            else:
                return f()
        return ejecucion
        
----

Esta función dentro de un decorador, se denomina ``closure``, y permite afectar al comportamiento de la función cuando **se ejecuta**.
        
----

.. code-block::

    hello = crazy(hello)
    # El 50% de las veces que **se ejecuta** la función, será
    # más divertido
    hello()

----

Ejemplo práctico: cambiar output
--------------------------------
El siguiente decorador hará que la salida de la función decoradora, se encuentre envuelta entre ``<em></em>``

.. code-block::

    def goodbye():
        return "Bye! Bye!"

    def em(f):
        def ejecucion(*args, **kwargs):
            output = f(*args, **kwargs)
            return '<em>{}</em>'.format(output)
        return ejecucion
        
    hello = em(hello)
    hello()
    # Devuelve: <em>Bye! Bye!</em>
    
----
    
Ejemplo práctico: cambiar entrada
---------------------------------
El siguiente ejemplo hace que la función decorada, siempre reciba números,
aunque estén como strings.

.. code-block::

    def parse_ints(f):
        def fn_wrap(*args):
            args = [int(a) for a in args]
            return f(**args)
        return fn_wrap
    
    my_fun = parse_ints(my_fun)
    my_fun('3', 4, '5')
    # my_fun recibe: 3, 4, 5.
    
----

Ejemplo práctico: condicionar función
-------------------------------------
Esta decorador restringe la ejecución del código en función a la entrada.
En este caso, que el usuario sea un moderador.

.. code-block::

    def is_moderator(f):
        def fn_wrap(request, *args, **kwargs):
            if request.user.is_moderator():
                return f(request, *args, **kwargs)
            else:
                raise PermissionDenied
        return fn_wrap
        
----

Sólo con esto, ya nos evitamos código como el siguiente en nuestras funciones::

.. code-block::

    def my_view(request):
        if not request.user.is_moderator():
            raise PermissionDenied
        # ....
        
Y además, podemos aplicarlo sobre funciones en las que no tenemos acceso al código.

----

Ejemplo práctico: sin afectar función
-------------------------------------

.. code-block::

    def ex_time(f):
        def fn_wrap(*args, **kwargs)
            t0 = time.time()
            output = f(*args, **kwargs)
            print("La función tardó {} segundos".format(time.time() - t0))
            return output
        return fn_wrap

----

Conclusión tipos decoradores
============================

* Los **decoradores simples** de nuestro primer decorador, afectan únicamente a la creación de la función.
* Los **decoradores con closure** (una función dentro), afectan a la ejecución de la función.

----

¿Qué uso pueden tener los simples?
==================================
Son menos empleados, pero permiten entre otras cosas *registrar* las funciones. Por ejemplo, para guardar las funciones de una API.

.. code-block::

    routes = set()
    
    def route(f)
        routes.add(f)
        return f
        
    @route
    def my_fun():
        ...
        
    # Ahora routes contiene my_fun.
    
----

Recibiendo parámetros en los decoradores
========================================

.. code-block::

    def tag(tag_name):
        def wrap(f):
            def wrapped(*args, **kwargs):
                return '<{0}>{1}</{0}>'.format(tag_name, f(*args, **kwargs))
            return wrapped
        return wrap

