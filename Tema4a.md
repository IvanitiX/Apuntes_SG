# Tema 4a. Animación
---

## 1. Introducción
La animación transmite al usuario la ilusión de que algo en la escena que está observando cambia a lo largo del tiempo.
Esta ilusión se logra mostrando una secuencia de varios fotogramas cada segundo (FPS), de forma que el espectador percibe el movimiento gracias a la *persistencia de la visión* (que el cerebro las procese y lo vea como movimiento por el poco tiempo entre imágenes)
***Velocidades:*** **Cine**: 24 FPS, **PAL**: 25 FPS, **NTSC**:29'97 FPS

### 1.1 Clasificación
Se clasifica ateniendo a los medios para obtener los fotogramas
***Animación convencional***: Todo se hace a mano, en 2D, todos los dibujos tienen apariencia plana. Su ventaja es la alta expresividad, y animaciones de gran calidad artística.
***Animación asistida por ordenador***: Animación convencional donde en algunas fases del proceso interviene el PC
***Animación por computador***: El ordenador, en base a toda la información que se ha dado, genera la secuencia de imágenes que dan lugar a la animación. La sensación de 3D se da por cálculos de *shading*, luces, representación de materiales, transparencias, brillos, reflejos y refracciones... Que hechos a mano serían difíciles de hacer.
La desventaja es la poca expresividad, moviéndose como robots. Se solventa mediante **motion capture**(que calca la expresividad del rostro o movimientos humanos en un modelo 3D)

Los profesionales que intervienen son dos:
-*Animador*: Define un movimiento en base a unos dibujos de unas poses clave.
-*Intercalador*: A partir de esas poses clave y sabiendo el tiempo de duración, hace los dibujos intermedios para que la transición sea más suave.
En la animación por computador, este trabajo es sustituido por el ordenador, quien interpola el valor de los parámetros del modelo sobre los parámetros definidos por el animador en las poses clave.

### 1.2 Etapas en una animación por ordenador
Da igual la envergadura del proyecto, hay dos tareas fundamentales: el guión y el esquema de la historia (o *storyboard*)
#### 1.2.a Guión o script
Es un texto donde se define qué se hace, qué se quiere contar, qué idea se quiere transmitir al espectador... Y a partir de ahí ir concretando lo demás para llegar a ese fin.
#### 1.2.b Esquema de la historia (o *Storyboard*)
Es un resumen gráfico, en viñetas, de la historia. Una especie de cómic donde se ve la pelicula. Se hacen anotaciones sobre si se hace zoom, intensidad de la luz, sonidos de fondo... En general, es el documento que contiene el diseño de la película en muchos aspectos de esta.
#### 1.2.c Bobina Leyca
A partir del *Storyboard*, y una vez que se tienen los diálogos de los personajes, es la primera versión de la película superponiendo los diálogos sobre la imagen fija de cada viñeta. Permite decidir realizar modificaciones en una etapa temprana de la producción.

> Es importante tener en toda animación:
> 1. ¿Qué se quiere contar con esa animación?
> 2. ¿Qué se debe mover para transmitirlo? ¿Tengo que dividir la animación en otras más pequeñas?
> 3. ¿Cuándo tarda en hacerse?

### 1.3 Modos de implementar la animación
***Procedural***: SIn técnicas ni metodologías. Hay un método que se llama en cada frame que se encarga de modificar los parámetros en función del tiempo que ha transcurrido.
***Mediante escenas clave***:La metodología se basa en definir qué valores van a tener los parámetros en los instantes inicial y final de la animación, e indicar el tiempo que debe transcurrir. La interpolación se hace en el PC
***Mediante caminos***: Apropiado si se quiere que un elemento siga una trayectoria. Hay que definir la trayectoria y vincular la posición de la figura a esta.

## 2. Animación procedural
Cada clase que tiene elementos que se mueven usa el método `update()` para modificar los parámetros que orrespondan según el tiempo que ha transcurrido desde la última vez que se ejecutó.
Esta animación se podría actualizar frame a frame, o hacer quese controle a través del tiempo con `THREE.Clock`para anotar el inicio de la animación y al actualizar, que añada la diferencia de movimiento con respecto al tiempo transcurrido desde la última  vez. Podría comprimirse en una clase `Animation`, por ejemplo.
> El `update()` de cada clase tiene la responsabilidad de la animación en su ámbito. Deben saber en qué momento ocurre cada cosa y, no menos importante, a qué velocidad
### 2.1 Curvas de función
Para el control de la velocidad nos ayudamos de las curvas de una función que recibe como entrada un valor de tiempo y proporciona como salida el valor de un parámetro. La gráfica resultante se podría interpretar como indicativo de la velocidad con la que cambia un parámetro con respecto al tiempo.
Esa gráfica podría ser más o menos compleja, pero se basaría en 4 estados: *detenido*, *acelerado*, *velocidad constante* o *decelerando*.
Para implementar una animación en donde un parámetro varíe a velocidad no constante, definiremos variables auxiliares que se basen en el tiempo (que es constante) para obtener otras que, además de representar ese tiempo transcurrido, varié la velocidad a nuestro gusto.
A partir de esos tiempos se extrae $\lambda$, que estará entre 0 y 1 y representa el porcentaje en tanto por 1 de la animación.
 Si $\lambda=0$, estaremos al inicio de la animación; y si $\lambda=1$, estaremos al final.
El cálculo de *lambda* sería:
$$\lambda=\frac{t-t_0}{t_f-t_0}$$
Si la velocidad para variar los parámetros no es constante, definiremos $f(\lambda)$ como una función que devuelve ese porcentaje de la animación, pero que variará el valor de lambda en función de ella misma.
Entonces, un valor del parámetro se definiría como:
$$p=v_0+f(\lambda)(v_f-v_0)$$
> Si $f(\lambda)$=$\lambda$, $p=v_0+\lambda(v_f-v_0)$
### 2.2 Animaciones con velocidad independiente del PC
>Nunca debe ocurrir que la misma animación se vea más rápido en un PC menos cargado o más rápido; y más lento en un PC lento o saturado.

Ese tipo de problemas pueden ocurrir si la animación es del tipo `figura.position.x += 0.1`. Para evitarlo recurriremos a la fórmula de la cinemática ($e=v\cdot t$).
Al actualizar ese valor así sería como hacer $e += \Delta e$ (añadir más espacio al espacio de antes). Ese $\Delta e$ no es un valor fijo, sino $v\cdot\Delta t$, siendo $\Delta t$ el tiempo que ha pasado desde que se llamó por última vez a animarse.
> Así, en un PC rápido, las llamadas serán muchas y los incrementos, más pequeños. En un PC lento, las llamadas serán pocas, pero con incrementos más grandes.

## 3. Animación mediante escenas clave. `Tween.js`
La **animación mediante escenas clave** es el modo de implementar animaciones más usado. Deriva de la *animación convencional* donde un animador define momentos importantes de los movimientos mediante escenas clave, y un intercalador hace los demás movimientos intermedios. 
>En el caso de la **animación por ordenador**, es el mismo procedimiento, pero las escenas clave se definen en base a dar valores concretos a los parámetros correspondientes de la animación.

Si la animación es compleja, esta se descompone en otras más simples. En ese caso, cada subanimación está determinado por dos escenas clave: la escena inicial y la escena final. Habrá que saber el tiempo que tardará y se puede definir los cambios de velocidad de ese movimiento. La animación global se obtiene encadenando esas subanimaciones.

### 3.1 Implementación en `Three.js`. Uso de `Tween.js`
>Para implementar esa animación se usará `Tween.js`, que se llama en el HTML.
La figura se construye como siempre. Una vez que sepamos qué animación hacemos, definiremos dos variables de tipo diccionario con los parámetros que editaremos en la animación. Uno será el *origen* y el otro, el *destino*.
Tras ello se define el movimiento, se dan los cambios de movimiento e indicaciones para modificar los valores de los parámetros de la animación.
(Para modificar la figura se trata desde `that`en vez de desde `this`)

**¿Cómo creo la animación?**
Se crea una instancia de Tween:
```
this.movimiento = new TWEEN.Tween(<punto_origen>).to(<punto_destino>,<tiempo>)
```
**¿Cómo cambio la velocidad?**
Con el método `this.movimiento.easing(<Constante>)` y la constante `TWEEN.Easing.<Aceleración>.<Ámbito>`.
La velocidad se basará en el tiempo dado y la distancia recorrida. Este método permite dar aceleración o deceleración a ese movimiento.
La `<Aceleración>` puede ser `Quadratic`,`Cubic`,`Quartic`,`Quintic` o `Exponential`.
El `<Ámbito>` de la aceleración puede ser `In` (que acelere al inicio), `Out` (que decelere al final), o `InOut` (acelera al inicio, frena al final)
**¿Dónde actualizo los parámetros?**
En Tween se usa su propio método de actualización, llamado `this.movimiento.onUpdate(...)`. En la clase principal se llamará a `TWEEN.update()` para que actualice los parámetros de las animaciones hechas con la biblioteca.
**¿Cómo reseteo la animación?**
Con el método `this.movimiento.onComplete(...)`
**Y el resto de métodos interesantes**
`start()`: Inicia la animación.
`stop()`: Para la animación.
`yoyo(true)`:Hace que la animación sea de ida y vuelta.
`<animacion1>.chain(<animacion2>)`: Encadena dos animaciones. Las animaciones se pueden encadenar en cualquier orden.


## 4. Animación mediante caminos
Esta técnica se utiliza cuando una figura debe moverse siguiendo una trayectoria arbitraria. Consiste en definir esa trayectoria (normalmente mediante un *spline*) y hace que la orientación y la posición sigan esa trayectoria. Ese *spline* se define a partir de un `CatmullRomCurve3`pasando una lista de puntos `Vector3`. Si se quiere dibujar la línea hay que tener en cuenta que la línea se realiza aproximándola mediante un conjunto de segmentos rectos. Cuantos más segmentos, más parece una curva. Eso se denomina *resolución*.
SI se quiere usar ese *spline* para que una figura se mueva por esa trayectoria, se va evaluando la curva para obtener su posición y colocar la figura. Además se debe orientar la figura usando la tangente de la curva en esa posición.

La curva se recorre con `this.spline.getPointAt(t)`, siendo $0 <= t <= 1$.

## 5. Mezclando técnicas
En una aplicación, no todas las animaciones deben hacerse con la misma técnica. Habrá que usar unas u otras según interese. No son más que un conjunto de técnicas para usarlas adaptándolas y combinándolas según el problema concreto que haya que resolver en cada caso.


