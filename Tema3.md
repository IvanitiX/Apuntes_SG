# Tema 3. Creación de geometrías
---

## 3.1 Introducción: Modelos geométricos vs volumétricos
>Un **modelo** es la representación de lo más relevante de una entidad.

Los **modelos geométricos** representan la geometría de un objeto, es decir, la forma de su frontera. *Si esa frontera encierra un espacio, éste no es relevante, es decir, no se vería*. Sería como ver solo la carcasa. 
Esos modelos se consideran elementos con coordenadas 3D, pero se puede hablar aún así de elementos *unidimensionales*, *bidimensionales* y *tridimensionales*.

Los **modelos volumétricos** representan propiedades especialmente localizadas. El interior es heterogéneo en cuanto a su material o propiedades.

## 3.2 Modelado de sólidos
### 3.2.a Pero... ¿qué es un sólido?
La definición de sólido se puede dar desde dos puntos de vista.
>#### El topológico
> Un solido es una porción del espacio ($S \subset \mathbb{R}³$) acotado, cerrado, rígido y regular.

>#### El algebraico
> El sólido se define mediante su **superficie frontera**, que es cerrada, orientada, completa y de forma que no se corte a sí misma.

Un sólido se representará a través de una colección finita de símbolos de un alfabeto finito que lo describa. Es decir, una fórmula.
Las propiedades deseables del **esquema de representación de un sólido** (la relación entre los sólidos y sus descripciones) son el dominio, la no ambigüedad, poca ocupacion de memeoria, facilidad de creación y edición, etcétera.

### 3.2.b Esquemas de representación
#### Barrido
El sólido se representa mediante una superficie plana y una trayectoria. Es cómo para crear sólidos pero tiene un dominio reducido.
#### Instanciación de primitivas
Aquí se tiene un conjunto de sólidos básicos, y un sólido se representa mediante la instanciación correcta de uno de ellos. Presenta las mismas ventajas e inconvenientes que el barrido.
#### Geometría de Sólidos Constructiva (CSG)
Se tiene un conjunto de primitivas básicas (que normalmente son semiplanos) y se usan operaciones booleanas binarias entre sólidos.
**Unión**: Se toma toda la materia de los operandos
**Intersección**: Se toma solo la zona común de los operandos
**Diferencia**:Se toma la materia del primer operando menos la del segundo. ¡No es conmutativa!

Un sólido CSG se representa mediante el árbol de operaciones que lo definen.
Las operaciones booleanas son regularizadas para que el resultado siga siendo un sólido válido. La regularización se definiría como la clausura del interior de un cuerpo y se denota con un asterisco.

###### Las ventajas
+ Tiene un dominio más amplio, dependiendo del conjunto de primitivas.
+ Permite crear sólidos complejos de una manera intuitiva.
###### La desventaja
Sin embargo, para visualización, requiere ser convertido a **B**oundary **Rep**resentation, o usar Ray Casting y evaluar el árbol en cada rayo.

#### Descomposición
El espacio se descompone en elementos disjuntos, normalmente cubos. El sólido se representa enumerando cuáles de esos elementos están ocupados.
###### Las ventajas
+ De manera aproximada, puede representar cualquier sólido.
+ Las operaciones booleanas se pueden implementar fácilmente.
+ El cálculo de propiedades volumétricas es muy sencillo
###### La desventaja
El dominio de objetos representables de manera exacta es mínimo.

#### B-Rep: Boundary Representation
Un sólido es representado describiendo la frontera que lo delimita. La frontera es representada mediante una malla de polígonos, normalmente triángulos.
###### Las ventajas
+ De manera aproximada, puede representar cualquier sólido.
+ La visualización es sencilla, simulando representaciones exactas de cualquier sólido.
###### Las desventajas
+ El dominio de representaciones exactas se reduce a los poliedros.
+ Son operaciones complejas la creación, las operaciones booleanas y el cálculo de propiedades volumétricas

>Por ello, un modelador de sólidos suele usar varios esquemas, uno primario y otros secundarios, beneficiándose de las ventajas de ellos.

### 3.2.c La representación B-Rep
#### Características
+ Es exacta para figuras poliédricas
+ Es aproximada en las fronteras curvas, aunque mediante técnicas de shading se puede suplir.
+ Es robusta e independiente de su complejidad geométrica.
#### Información a almacenar
Obligatoriamente usará una *geometría* basada en los vértices y sus coordenadas; y una *topología*, es decir, las conexiones entre vértices para formar las caras.
Adicionalmente, en cada vértice se puede asignar un vector normal para el cálculode la luz; y unas coordenadas de textura para poder aplicarla.
#### Geometría indexada vs no indexada
| |Geometría no indexada|Geometría indexada|
|-|-------------------|------------|
|Estructura|Cada vértice aparece varias veces, uno por cara. Las caras se definen a través de los índices de cada vértice. Se ponen en orden anthorario. | Un vértice se define una sola vez. Las caras se definen a través de los índices de cada vértice. Se ponen en orden anthorario.|
|Ventajas|La visualización es más rápida.<br>Permite tener un mismo vértice con varias normales y coordenadas de textura, según la cara desde la que se use dicho vértice | Se ahorra espacio de almacenamiento. La modificación de un vértice es más rápida; Operaciones como saber si dos caras comparten un mismo vértice son más rápidas|
|Desventajas| Se pierde espacio de almacenamiento. La modificación de un vértice es más lenta; Operaciones como saber si dos caras comparten un mismo vértice son más lentas| La visualización es más lenta.<br>No permite tener un mismo vértice con varias normales y coordenadas de textura|

## 3.3 Creación de geometría
La geometría se puede crear de varias maneras:
#### Manualmente
Se define en el código toda la información de caras y vértices.
#### Proceduralmente
A partir de unos parámetros, un método genera la información, generando las listas de vértices y caras
##### En `Three.js`...
Para crear la geometría de las dos anteriores maneras se usa la clase `PolyhedronGeometry`, donde se definen o calculan los arrays de coordenadas e índices y se le pasan como parámetros al constructor.
El constructor se define como `new THREE.PolyhedronGeometry(<Vértices de lista de vértices>,<Índices para lista de caras>)` De la primera lista, cada 3 valores obtienes un vértice; y de la segunda, cada 3 valores, una cara.

#### Operando con primitivas básicas
Se dispone de una geometría básica ya generada y se construye una geometría compleja operando con la que se tiene en cada momento.

>###### La clase Shape
> Se usa para crear geometría 2D y extrusionarla por extrusión o barrido.
Se dibuja el contorno con estas órdenes:
`moveTo(x,y)`: Movimiento sin dibujar hasta la posición (x,y)
`lineTo(x,y)`: Línea recta desde la posición actual hasta la posición (x,y)
`quadraticCurveTo(PCx,PCy,x,y)`:Bezier cuadrática
`bezierCurveTo(PC1x,PC1y,PC2x,PC2y,x,y)`:Bezier cúbica
`splineThru(pts)`: Spline por los puntos indicados. `pts` es un array de `THREE.Vector2`
`absarc(x,y,radius,aStartAngle,aEndAngle)`: Dibuja un segmento de círculo
`absellipse(x,y,xRadius,yRadius, aStartAngle,aEndAngle)`: Dibuja un segmento de elipse
`holes.push`: Se añaden los agujeros a prtir de un shape.
A partir de un Shape se puede construir una geometría 2D (`ShapeGeometry`) o una 3D (`ExtrudeGeometry(shape,opciones)`)

#### Geometría por extrusión/barrido
La clase `ExtrudeGeometry` puede hacer extrusiones y barridos a partir de un `Shape` con el contorno y unas opciones que se detallan:
`amount`: Cantidad de extrusión.
`bevelEnabled`: Flag para añadir bisel.
`bevelThickness`: Anchura del bisel.
`bevelSize`: Tamaño del bisel.
`bevelSegments`: Segmantos para suavizar el bisel.
`curveSegments`: Segmentos para suvaizar las curvas del `Shape` extrusionado.
`steps`: Segmentos en los que dividir la parte extruida.
`extrudePath`: Es el camino por el que se quiere hacer el barrido. Por defecto es el eje Z.
Su constructor es `new THREE.ExtrudeGeometry(<Shape>,{<Opciones>})`

#### Geometría por revolución
La clase `LatheGeometry`se define a través de una serie de puntos para definir el perfil y unos parámetros opcionales. La revolución se hace en el eje Y.
`segments`: Segmentos para dividir el cuerpo de revolución.
`phiStart`: Ángulo de inicio.
`phiLength`: Ángulo de recorrido.

#### Primitivas básicas
Cubo: `CubeGeometry(width,height,depth,segmentosWidth,segmentosHeight,segmentosDepth)`
Esfera: `SphereGeometry(radius, widthSegments, heightSegments,phiStart, phiLength, thetaStart, thetaLength)`
Cilindro: `CylinderGeometry(radiusTop, radiusBottom, height,segmentsX, segmentsY, openEnded)`
Toro: `TorusGeometry(radius, tube, radialSegments, tubularSegments, arc)`
Poliedros regulares: `TetrahedronGeometry/OctahedronGeometry/IcosahedronGeometry/DodecahedronGeometry(radius)`

#### Operaciones booleanas
Para este tipo de operaciones, se hace uso de la biblioteca `ThreeBSP`
>###### Operando con ThreeBSP
>1. Se crean las geometrías por los medios vistos
>2. Se posicionan y orientan donde deban estar. Se usan los siguientes métodos:
>Traslación: `translate(x,y,z)`
>Escala:`scale(x,y,z)`
>Rotación:`rotate<X/Y/Z>(angulo)`
>>**¡Cuidado!** Esas transformaciones sí se aplican a la geometría en el mismo orden en el que se escriben en el código.
>3. Las geometrías se convierten a nodos ThreeBSP
>4. Se opera entre nodos ThreeBSP , el resultado es un nodo ThreeBSP
>+ Unión, a través del método `union()`
>+ Intersección, a través del método `intersect()`
>+ Diferencia, a través del método `subtract()`
>5. El resultado final se convierte a un Mesh de Three,a través del cual se calculan los vectores normales.

#### Interactivamente
Al usuario se le dan las herramientas para construir sus figuras *“a golpe de ratón”*
La herramienta requiere procesar adecuadamente los eventos del ratón según el contexto. A partir de ahí se van completando las listas de vértices e índices necesarias para construir la geometría. Es importante ir mostrando el resultado parcial mientras se construye ya que el usuario necesita una realimentación.

De esta forma, a partir de una forma simple,  se pueden ir separando  los polígonos para ir desplazándolos y refinándolos, haciendo una geometría más compleja.

#### Cargándola desde un archivo
Se genera la geometría a partir de archivos, ideal para modelos más complejos. En nuestro caso, los archivos son de formato `.obj` para los modelos y de `.mtl` para las texturas. Este formato fue creado por *Wavefront Technologies*.
El archivo `.obj` puede contener vértices, caras, coordenadas de textura y normales.
El archivo `.mtl` contiene definiciones de materiales.
Para cargar los archivos se hace uso de las librerías `OBJLoader` y `MTLLoader`, y a través de ellos se carga **primero el modelo** y después el objeto.

El constructor de un modelo que se carga es: 
```javascript
constructor(){
        super();

        var that = this ;
        var materialLoader = new THREE.MTLLoader();
        var objectLoader = new THREE.OBJLoader();

        materialLoader.load('<Ruta de texturas>',
            function(material){
                objectLoader.setMaterials(material);
                objectLoader.load('<Ruta del objeto>',
                function(objeto){
                    var modelo = objeto ;
                    that.add(modelo);
                },
                <Función de progreso de carga>, <Función de error>
            });
    }
```

#### Cargar geometrías en la GPU. Versiones `BufferGeometry`
Las instrucciones vistas crean la geometría en CPU, pero es más eficiente tenerla en la GPU. Algunas instrucciones tienen versión para GPU, con la coletilla `BufferGeometry`
Sin embargo, no hay operadores para geometría en GPU. Afortunadamente, se puede crear una geometría en GPU a partir de cualquier geometría de CPU con el parámetro `fromGeometry`

>#### Creación de figuras
> Una figura se compone de una malla y un material. Para crearlo se hace una instancia de `THREE.Mesh`con esos parámetros.

## 3.4 Modelos jerárquicos
En un modelo se combinan geometrías y transformaciones. Sus objetivos son crear figuras compuestas a partir de figuras simples, fáciles de editar y mantener; y tener objetos articulados fáciles de animar y mantener.
### 3.4.a Transformaciones geométricas
Las transformaciones más comunes son las traslaciones (mover items de una posición a otra), los escalados (agrandar un item respecto al origen de coordenadas) y las rotaciones (giros sobre el origen de coordenadas).
Las transformaciones pueden unirse conmutativamente (si combinas escalados y rotaciones; o haces varias traslaciones) o no conmutativamente(si haces traslaciones y rotaciones/escalados)
### 3.4.b El grafo del modelo
Al igual que los grafos de escena, un grafo de modelo jerárquico es un árbol (grafo dirigido acíclico) donde la raíz respresenta la figura completa, los nodos internos representan las transformaciones (una transformación por nodo que afecta a los hijos que le siguen), y los nodos hoja que representan las geometrías (transformadas cada vez más por nodos anteriores).
>En un buen diseno, un movimiento en un grado de libertad del objetodebería implicar cambios en un solo nodo. A veces puede que no sea posible.
Para diseñar un grafo sin errores se deben tener en cuenta los grados de libertad.
### 3.4.c Aplicación de transformaciones
#### A geometrías
Para poder aplicarlas:
+ No pueden depender de variables que cambien con el tiempo.
+ Se conectan directamente a Geometrías
+ No pueden tener otros hijos

Se aplican una sola vez en la construcción en el orden correcto antes de construir el Mesh.

Los métodos para ello en `Three.js` son:
Escalado: `scale(x,y,z)`
Rotación: `rotateX/Y/Z(ángulo en radianes)`
Traslación: `translate(x,y,z)`
#### A mallas
La clase `Mesh`, además de tener geometrías y materiales, puede incluir un nodo de transformación que se aplica en cada visualización. Se usa para mover el objeto con la interacción con el usuario; y para realizar animaciones.
Los `Mesh` pueden tener hijos, los cuales se conectan por dicha transformación.
Se pueden configurar mediante los atributos `position`, `rotation` y  `scale`.
> Las transformaciones siguen un orden:
>1. Escalado
>2. Rotaciones:
 2.1. Rotación en Z
 2.2. Rotación en Y
 2.3. Rotación en X
>3. Traslación
#### A Objetos
Un objeto usa la clase `THREE.Object3D` como nodo. Usa varios atributos que lo identifican y permiten acceder a sus nodos, y unos métodos para montar la jerarquía. Además, puede realizar alguna transformación con ámbito de la figura entera.
###### Atributos
`name` Un nombre opcional y no único del objeto
`parent` Referencia a su nodo padre
`children` Un array de nodos hijo
###### Métodos
`add(objeto)` Añade el objeto a `children`
`remove(objeto)` Elimina el objeto de `children`
`getObjectByName(string)` Devuelve un objeto con ese nombre
###### Transformaciones
`translateOnAxis(dirección,distancia)` Traslada todo el objeto o Mesh una distancia en una dirección concreta. La dirección es un `Vector3`normalizado, y acumula lo que se cambió en `position`
`rotateOnAxis(eje,ángulo)` Rota todo el objeto sobre un eje. El eje es un `Vector3`normalizado, y se acumula a los cambios en `rotation`.
