<style>
 blockquote{
     border-left-color: #00bbff;
 }
 .vscode-light h1, .vscode-light hr, .vscode-light table > tbody > tr + tr > td{
     border-color: #00bbff;
 }
</style>
# 4c. Introducción a los Motores de Física

## 1. Introducción
En una escena gráﬁca que represente una escena real, se quiere poder representar las características que tiene el mundo real.

> Para implementar todos esos fenónmenos sería necesario representar las magnitudes físicas involucradas con dichos fenómenos,incorporarlas en los objetos y en cada frame, aplicar las ecuaciones físicas correspondientes para determinar la nueva posición y orientación de cada objeto según las fuerzas que actúan sobre él y su inercia lineal y angular. Eso es lo que hace un **motor de física**, una biblioteca que representa y gestiona tales magnitudes y los efectos que conllevan.

En nuestro caso, usaremos la biblioteca `Physijs`, pero *aviso*: Para usar esa biblioteca hay que utilizar la versión de `Three.js` que la acompaña. La bilioteca no es un motor de físicas de por sí, sino un *wrapper* que facilita el uso de la biblioteca `ammo.js`, quien de verdad implementa el motor.

### Una escena básica
Además de descargarse la biblioteca y añadir el archivo `physi.js` en el `html` y modiﬁcar el `html` para usar la versión de `Three.js` que trae `Physijs`, es necesario indicarle a `Physijs` cuál es el *web worker* que se va a usar para gestionar las hebras e indicar igualmente que se va a usar ammo como motor de física.

Por lo demás, una escena `Physijs` es similar a una escena `Three` pero usando versiones físicas de algunos elementos. En particular:
- La escena es **física** (clase `Physijs.Scene()`)
- Se establece una **gravedad** con el método `setGravity()`.
- Los materiales, aunque se basen en materiales de `Three`, **deben ser físicos**.
- Las mallas, basados en geometrías de `Three`, también **deben ser físicos**.
- Los elementos que se desee sean tenidos en cuenta por el motor de física deben colgar **DIRECTAMENTE** del nodo raiz de la escena. 
- Se indica en el `update()` principal que se debe simular las físicas con el método `simulate()`

No todos los elementos en una escena física deben ser físicos, solo aquellos que se desee que sean tenidos en cuenta por el motor de física.

## 2. Magnitudes físicas

 Una de las magnitudes que se representa es la fuerza de la gravedad, conﬁgurándola en la escena física con `setGravity()`.

 >En cuanto a magnitudes físicas aplicables a los objetos se tiene la **masa** del objeto, un **coeﬁciente de rozamiento** y un **coeﬁciente de rebote**.Vayamos por partes:

 - Al crear un ***material físico*** se pueden establecer los valores de **Rozamiento** y **Rebote**. Para crear un Material Físico se usa:
 ```javascript
 var matFisico = Physijs.createMaterial(<Material_de_Three>,<Rozamiento>,<Rebote>);
 ```

 donde los coeficientes están determinados entre 0 y 1. Cuanto mayor el rozamiento, más costará arrastrarlo. Cuanto mayor el rebote, más afectado estará ante colisiones.

 - Al crear una ***malla física*** se puede establecer la **masa**. Una malla física se crea así:
  ```javascript
  var suelo = new Physijs.BoxMesh ( <geometriaThree> , <materialFisico> , <masa>);
  ```

  Si se pone masa 0, el objeto no sufrirá los efectos dee la gravedad. Habrá que tener en cuenta la masa en las colisiones, pues los objetos ligeros se afectan más que los pesados.

  ## 3. Tipos de figuras físicas

  >La biblioteca Physijs dispone de varias clases para crear Mesh físicos. La geometría concreta que se visualizará de la ﬁgura será la geometría Three que se haya usado para construir el Mesh físico. Pero el motor de física realizará los cálculos considerando que la geometría de la ﬁgura es la que se corresponde con la clase Physijs que se ha usado para construirla.

  Así, por ejemplo, se podría crear una esfera que no rueda si ponemos que la geometría sea un `Three.SphereGeometry` pero las físicas sean de un `Physijs.BoxMesh` . Por ello habrá que coger las figuras en base a la geometría que usará el Mesh Físico. Esas mallas pueden ser:

  - Una caja: `Physijs.BoxMesh()`
  - Una esfera: `Physijs.SphereMesh()`
  - Un cilindro: `Physijs.CylinderMesh()`
  - Un cono: `Physijs.ConeMesh()`
  - Para el resto de geometrías, o aquellas queno encajen con las anteriores: `Physijs.ConvexMesh()`. **Aviso**: Su procesamiento es más lento, por lo que debería evitarse.

La idea sería coger la más idónea o la menos mala de esta lista.

### ¿Y si el objeto físico es compuesto?
Habrá que tener en cuenta estas restricciones:
- Debe haber una jerarquía entre ellas
- La raíz debe ser una figura física
- Se debe hacer la jerarquía antes de añadir la raíz de la jerarquía a la raíz de la escena.

### Otras formas especiales
Hay otro Mesh físico que sirve para crear terrenos o suelos ondulados. Se trata de la clase `Physijs.HeightfieldMesh`, que parte de una geometría de Three que es un plano con una determinada resolución, y donde la coordenada Z de cada vértice representa la altura en dicho punto. Se ponen en dichas Z las alturas deseadas y ﬁnalmente se construye el Mesh físico que representa ese terreno con ondulaciones.

## 4. Interacción con las figuras
Normalmente , en el motor de física, la posición y rotación de los objetos varía por las colisiones y magnitudes físicas, pero se puede seguir interactuando manualmente con la escena. Vayamos punto por punto:

### Poner una posición u orientación manualmente
Se puede cambiar los valores de position y/o rotation como hasta ahora, pero hay que indicarle al motor de física que se han cambiado con los booleanos `_dirtyPosition` y/o `_dirtyRotation`.

### Establecer una velocidad manualmente
Las figuras físicas tienen dos métodos para establecer velocidades. Estos recogen un vector 3D que indica su dirección, del cual su módulo establece la cantidad de velocidad. También tienen sus respectivos getters.

- Para la velocidad lineal, `setLinearVelocity(vector3)`
- Para la velocidad angular, `setAngularVelocity(vector3)`

### Empujar objetos
También se puede interactuar con los objetos aplicando impulsos. La fuerza que se aplica es un vector 3D donde se idica la dirección y cuyo módulo es la cantidad de fuerza. Si se quisiera independizar la cantidad y la dirección del impulso, se podría normalizar la dirección y multiplicar escalarmente por la cantidad.

Para aplicar el impulso se usa el método `applyCentralImpulse(vector3)`.

## 5. Procesado de colisiones
Una de las ventajas de usar un motor de física es que se detectan y procesan las colisiones. Se evalúa cuándo dos ﬁguras colisionan, sus respectivas velocidades, geometrías, masas, coeﬁcientes de restitución (rebote), etc. y calcula las nuevas velocidades para cada ﬁgura.

Cada ﬁgura sale despedida de esa colisión en la dirección calculada, y a la velocidad lineal y angular que se haya calculado.

Además, podemos programar un listener para cuando dos objetos colisionen, que obtiene toda la info de la colisión (velocidades, normal de contacto, la otra figura...), aunque en videojuegos se use más para el estado del juego, como quitar vidas o anotar puntos.

>Las figuras físicas que tengan un listener asociado deben colgar directamente de la raíz de la escena

## 6. Restricciones
Un motor de físicas puede aumentar su usabilidad si se ponen limitaciones en los movimientos a las figuras físicas.<br>¡Pero cuidado! Las restricciones tienen un orden para realizar las acciones fijo. Si se pusieran enotro orden puede que no surtan el efecto deseado.

>1. Los objetos que intervienen en la restricción se añaden a la escena
>2. Se define la restricción referenciando los objetos involucrados.
>3. Se añade la restricción a la escena con `addConstraint(<Restricción>)`
>4. De ser necesario, se configura la restricción.

Ahora, veamos algunas restricciones:

### 6.1 Restricción punto a punto
Restricción más básica. Restringe la posición de un objeto respecto a otro, haciendo que un objeto se mueva con otro manteniendo una distancia y orientación relativas. Se hace con `Physijs.PointConstraint`.
### 6.2 Restricción de bisagra
Con esta restricción un objeto gira con respecto a un eje determinado. Se hace con `Physijs.HingeConstraint`. Además, se puede activar y desactivar un motor (que mueva el objeto sujeto a la restricción) con esta restricción.
### 6.3 Restricción de deslizamiento
Un objeto con esta restricción sólo se puede mover a lo largo de una recta. Esta restricción se crea indicando la dirección de desplazamiento. Se pueden estabecer límites en el desplazamiento y se puede activar/desactivar un motor.

La dirección de desplazamiento se indica con un Vector3 (el propio Francisco desconoce el por qué) con unas coordenadas extrañas que se resumen en esta tabla:

|Desplazamiento por eje... | Usa el vector...|
|--------------------------|-----------------|
|X                         | $(0,1,0)$       |
|Y                         | $(0,0,\Pi/2)$   |
|Z                         | $(\Pi/2,0,0)$   |

Se hace con `Physijs.SliderConstraint`

### 6.4 Restricción tipo péndulo
Hace que la figura oscile como si de un péndulo se tratara. Se pueden establecer ángulos límite en cada eje y poder activar/desactivar un motor.
Se hace con la clase `Physijs.ConeTwistConstraint`

### 6.5 Restricciones con otros objetos
Otra forma de restringir movimientos es conotras figuras que impidan movimientos no deseados.

Por ejemplo, imagina que se está haciendo una juego de billar y se desea evitar que las bolas salten y puedan caerse de la mesa. Podría hacerse poniendo una tapa transparente por encima de la mesa de manera que si debido a una colisión entre bolas alguna saltara, tropezara con esa tapa que le está haciendo de límite vertical.

### 6.6 Restricción genérica de grados de libertad
Se trata de una restricción que es bastante conﬁgurable ,permitiendo controlar de manera exacta los movimientos de un objeto. Se pueden deﬁnir límites en los grados de libertad, modiﬁcar los límites cuando sea necesario, mover los elementos a conveniencia activando algún motor, y por supuesto, desactivar los motores que se hayan activado.

Esta restricción se deﬁne con la clase `Physijs.DOFConstraint` *(Degree Of Freedom Constraint)*

*Podéis ver un ejemplo a partir de la pag.16 de los apuntes de SG*









