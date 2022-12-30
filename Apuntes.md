# VIDEOJUEGO PESCADO

- Linkeo los archivos CSS y JS al index.html
- Creo un canvas en el body con el id canvas1
- Hago un reset del CSS
- Le digo que el body ocupe toda la pantalla y le pongo un display: flex. Lo centro 
- Seteo el canvas

~~~css
*{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}


body{
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

#canvas1{
    width: 800px;
    height: 500px;
    border: 4px solid black;
}
~~~

- Ahora le toca al script
- Para poner todo a funcionar se necesitan 5 cosas básicas
    - Canvas setup
    - Mouse interactivity
    - Player
    - Bubbles
    - Animation Loop

## Canvas Setup

- Capturo el canvas y creo el contexto
- Seteo el canvas
~~~js
const canvas = document.getElementById('canvas1')
const ctx = canvas.getContext('2d')

canvas.width = 800;
canvas.height= 500;
~~~

- Creo la variable score a 0
- También la variable gameFrame a 0
- Seteo la fuente

~~~js
let score = 0
let gameFrame = 0

ctx.font = '50px Georgia'
~~~

## Mouse interactivity

- Seteo las coordenadas del mouse para que empiecen en lamitad del canvas
- Creo un listener para el mouse down y mouse up y setear las coordenadas con mis movimientos de mouse
- En el listener seteo el cmouse.click en true

~~~js
const mouse ={
    x: canvas.width/2,
    y: canvas.height/2,
    click: false
}

canvas.addEventListener('mousedown', function(event){
    mouse.click = true;
    mouse.x= event.x;
    mouse.y = event.y
})
~~~

- Hay un problema, que las coordenadas 00 del canvas no corresponden al extremo superior izquierdo del rectángulo si no de la pantalla
- Esto se soluciona de la siguiente forma

~~~js
let canvasPosition = canvas.getBoundingClientRect();
~~~

- Si hago un console.log de canvasPosition me da la posición del canvas y las coordenadas del left, top, etc
- Le resto estas coordenadas al listener

~~~js
canvas.addEventListener('mousedown', function(event){
    mouse.click= true;
    mouse.x= event.x - canvasPosition.left;
    mouse.y = event.y - canvasPosition.top
})
~~~

- Esto hace que las coordenadas 00 sean el extremo izquierdo del canvas ( rectángulo )

## Player

- Creo la clase Player
- En el constructor le paso las coordenadas desde dónde va a partir el jugador antes de moverse
    - Corresponde a las coordenadas que he seteado el mouse
- Le añado el radius porque el personaje va aser representado por un círculo
- El ángulo será usado para rotar el personaje cuando este se mueva
- Quiero que mi personaje siempre mire en la dirección que nade
- El frame lo sitúo en 0
- El spritesheet mide de ancho 1982px y tiene 4 columnas 1982/4 === 498;
- De altura tiene 981px  y tengo 3 columnas 981/3 === 327;

~~~js
class Player {
    constructor(){
        this.x = canvas.width;
        this.y = canvas.height/2;
        this.radius= 50;
        this.angle = 0;
        this.frameX = 0;
        this.frameY = 0;
        this.frame= 0;
        this.spriteWidth = 498;
        this.SpriteHeight= 327;
    }
}
~~~

- Creo un método de update
- Si le pongo this.x --; solo podría mover a la izquierda en menos dirección del eje x. en cambio dx puede mover en las dos direcciones, por eso -=dx
- Lo divido entre 30 por la velocidad, porque si no no habría animación en los cambios

~~~js
     update(){

        const dx = this.x - mouse.x;
        const dy = this.y - mouse.y;

        if(mouse.x != this.x){
            this.x -= dx/30;
        }
        if(mouse.y != this.y){
            this.y -= dy/30;
        }
    }
~~~

- Necesitaré también un método draw
- Creo ( fuera de la clase ) el addEventListener para el mouseup y seteo el click en false

~~~js
canvas.addEventListener('mouseup', function(){
    mouse.click = false
})
~~~

- Hago la condición en el método draw 
    - Establezco una linea delgada
    - Llamo a beginPath
    - moveTo seteará un punto de partida de la linea. Le paso las coordenadas del jugador
    - lineTo será el punto final de la linea
    - stroke conectará estos dos puntos
- Ahora dibujaré un círculo que representará el jugador
    - Uso arc para el círculo. Le paso las coordenadas, el radius para el tamaño, start ángulo 0, end ángulo Math.PI
    - Llamo a fill para dibujar el círculo y closePath para cerrar

~~~js 
class Player {
    constructor(){
        this.x = canvas.width;
        this.y = canvas.height/2;
        this.radius= 50;
        this.angle = 0;
        this.frameX = 0;
        this.frameY = 0;
        this.frame= 0;
        this.spriteWidth = 498;
        this.SpriteHeight= 327;
    }

    update(){

        const dx = this.x - mouse.x;
        const dy = this.y - mouse.y;

        if(mouse.x != this.x){
            this.x -= dx/30;
        }
        if(mouse.y != this.y){
            this.y -= dy/30;
        }
    }

    draw(){
        if(mouse.click){
            ctx.lineWidth = 0.2;
            ctx.beginPath();
            ctx.moveTo(this.x, this.y);
            ctx.lineTo(mouse.x, mouse.y);
        }
        ctx.fillStyle= 'red';
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI*2)
        ctx.fill();
        ctx.closePath();
    }
}
~~~

- Ahora puedo instanciar la clase

~~~js
const player = new Player()
~~~

- Creo la función animate y llamo al método update de player
- Llamo al draw para dibujar la linea entre el jugador y el mouse y dibujar el círculo que representa al jugador
- Creo el loop con requestAnimationFrame pasándole la función
- Invoco la función

~~~js
const player = new Player()

function animate(){
    player.update();
    player.draw()
    requestAnimationFrame(animate)
}

animate()
~~~

- Aparece una animación de un circulo rojo dejando un rastro rojo
- Para arreglarlo llamo a clearRect y le paso las dimensiones del canvas, para limpiar el canvas entero de la animación de cada frame 

~~~js
function animate(){
    ctx.clearRect(0,0, canvas.width, canvas.height)
    player.update();
    player.draw()
    requestAnimationFrame(animate)
}
~~~

- Cómo en las coordenadas del mouse se puso /2, la animación corrige el círculo y lo situa en el medio del canvas
- Donde clico con el mouse, el círculo va
- el this.x de la clase Player es lo que determina de dónde sale el círculo cuando inicia. Si lo pongo a 0 saldrá de la derecha


## Bubbles

- Creo antes de la función animate un array de burbujas
- Creo la clase bubble
- Uso Math.random para que salga un numero aleatorio
- De velocidad será un número aleatorio del 1 al 6
- La distancia determinará la distáncia de entre cada burbuja y con el jugador también para hacer pop cuando esté cerca
- Creo el update method para mover las burbujas hacia arriba, en dirección negativa al eje y dependiendo de la velocidad
- Creo el método draw
    - Voy a hacer las burbujas azules

~~~js
const bubblesArray = []

class Bubble{
    constructor(){
        this.x = Math.random() * canvas.width;
        this.y = Math.random() * canvas.height;
        this.radius = 50;
        this.speed = Math.random() * 5 + 1;
        this.distance;
    }

    update(){
        this.y -= this.speed;
    }

    draw(){
        ctx.fillStyle = 'blue';
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI*2 );
        ctx.fill();
        ctx.closePath();
        ctx.stroke();
    }
}
~~~

- Al principio declaré la variable gameFrame a 0
- Hago un incremento de gameFrame en la función animate
- Ahora puedo usarlo para mover objetos
- Creo una función handleBubbles fuera de la clase
- Por cada frame divisible por 50 añadiré con el método push una burbuja al array

~~~js
function handleBubbles(){
    if( gameFrame % 50 == 0){
        bubblesArray.push(new Bubble())
    }
}
~~~

- Añado handleBubbles en la función animate
- hago un ciclo for y por cada iteración llamo al método update y draw

~~~js
function handleBubbles(){
    if( gameFrame % 50 == 0){
        bubblesArray.push(new Bubble())
    }
    for(let i = 0; i < bubblesArray.length; i++){
        bubblesArray[i].update()
        bubblesArray[i].draw()
    }
}
~~~

- Las burbujas aparecen en cualquier lado de la pantalla yedo hacia arriba, pero quiero que aparezcan desde abajo del todo
- Para ello cambio en el constructor el this.y y le añado el canvas.height

~~~js
this.y = canvas.height + Math.random() * canvas.height;
~~~

- Ahora el problema es que el array de burbujas crece sin medida. Para evitarlo usaré el método splice.
- El primer argumento es el índice y el segundo el número de items
- Le añado i--, porque todos los elementos al hacer el splice, se corren una posición abajo.
- i+1 se convierete en i, y los métodos update y draw no son llamados, por eso "blinkea" la burbuja
- Añadiendo i -- hace que el loop vuelva un paso atrás y dibuje el item que se ha movido a la posición i
- Le añado filltext en la función animate. Le paso el texto y la posición
- Ahora quiero añadir un punto en el score por cada colisión con una burbuja
- En la función handleBubbles
- Si la distancia es menor que el radio de la burbuja más el radio del jugador suma 1
- Le añado un score++ para que sume con cada colisión en la puntuación

~~~js

function handleBubbles(){
    if( gameFrame % 50 == 0){
        bubblesArray.push(new Bubble())
    }
    for(let i = 0; i < bubblesArray.length; i++){
        bubblesArray[i].update()
        bubblesArray[i].draw()
        if(bubblesArray[i] < 0){
            bubblesArray.splice(i, 1)
            i--;
        }
        if(bubblesArray[i].distance < bubblesArray[i].radius + player.radius){
            score++;
        }
    }
}
~~~

- He creado en el constructor this.distance pero por ahora no tiene un valor
- En el método update creo las variables dx y dy que sera la diferencia entre la burbuja y el jugador en el eje x e y
- Con estos dos valores, podemos imaginar un triángulo imaginario, uniendo el centro de un circulo en linea recta haciendo ángulo para unirse con otro centro.
    - La base ( que uniría los dos centros en diagonal ) es la hipotenusa. Para calcularla podemos usar el teorema de Pitágoras
    - Math.sqrt es la raíz cuadrada
- class Bubble

~~~js
    update(){
        this.y -= this.speed;
        const dx = this.x - player.x;
        const dy = this.y - player.y;

        this.distance = Math.sqrt(dx*dx + dy*dy)
    }
~~~

- Lo que pasa es que con una sola colisión, al pasar frame a frame suma varios puntos con una sola burbuja
- Para ello añado una nueva propiedad al constructor de Bubble llamado this.counted = false
- Vuelvo al if de colisión para meterlo dentro otro if
- Si el elemento es falso, puntuo. Una vez puntuado cambio el valor a true y con el splice lo elimino del array 

~~~js
function handleBubbles(){
    if( gameFrame % 50 == 0){
        bubblesArray.push(new Bubble())
    }
    for(let i = 0; i < bubblesArray.length; i++){
        bubblesArray[i].update()
        bubblesArray[i].draw()
        if(bubblesArray[i] < 0){
            bubblesArray.splice(i, 1)
            i--;
        }
        if(bubblesArray[i].distance < bubblesArray[i].radius + player.radius){

            if(!bubblesArray[i].counted){
                score++;
                bubblesArray[i].counted = true
                bubblesArray.splice(i,1)
            }

        }
    }
}
~~~

- En el constructor de Bubble creo una nueva propiedad para añadirle sonido a la colisión de la burbuja
- Tendrá dos sonidos. Para que elija uno de los dos aleatoriamente uso Math.random y un oprador ternario

~~~js
class Bubble{
    constructor(){
        this.x = Math.random() * canvas.width;
        this.y = canvas.height + Math.random() * canvas.height;
        this.radius = 50;
        this.speed = Math.random() * 5 + 1;
        this.distance;
        this.counted= false;
        this.sound= Math.random() <= 0.5 ? 'sound1': 'sound2'
    }
}
~~~

- Creo bubblePop1 y 2 fuera de la clase

~~~js
const bubblePop1 = document.createElement('audio')
bubblePop1.src= 'bubbles-single1.wav'
const bubblePop2 = document.createElement('audio')
bubblePop2.src= 'bubbles-single2.wav'
~~~

- Vuelvo al if de la colisión en handleBubbles
- Uso un if de nuevo y uso el método play()

~~~js
function handleBubbles(){
    if( gameFrame % 50 == 0){
        bubblesArray.push(new Bubble())
    }
    for(let i = 0; i < bubblesArray.length; i++){
        bubblesArray[i].update()
        bubblesArray[i].draw()
        if(bubblesArray[i] < 0){
            bubblesArray.splice(i, 1)
            i--;
        }
        if(bubblesArray[i].distance < bubblesArray[i].radius + player.radius){

            if(!bubblesArray[i].counted){
                if(bubblesArray[i].sound=== 'sound1'){
                    bubblePop1.play()
                }else{
                    bubblePop2.play()
                }
                score++;
                bubblesArray[i].counted = true
                bubblesArray.splice(i,1)
            }

        }
    }
}
~~~

- Uso png tools para cambiar del revés el pescado. Este método no siempre funciona. hay mejores maneras de hacerlo

> https://onlinepngtools.com/flip-png-vertically

- Lo guardo como fish-right

~~~js
const playerLeft = new Image()
playerLeft.src= 'fish-left.png'

const playerRight = new Image()
playerRight.src = 'fish-right.png'
~~~

- En el método draw de la clase Player uso drawImage que tiene 9 argumentos (puede tener menos)
    - La imagen que quiero dibujar
    - Los siguientes 4 indica el area que quiero recortar del sprite sheet
    - Los otros 4 definen en dónde del canvas lo voy a colocar

~~~js
    draw(){
        if(mouse.click){
            ctx.lineWidth = 0.2;
            ctx.beginPath();
            ctx.moveTo(this.x, this.y);
            ctx.lineTo(mouse.x, mouse.y);
        }
        ctx.fillStyle= 'red';
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI*2)
        ctx.fill();
        ctx.closePath();

        ctx.drawImage(playerLeft, this.frameX * this.spriteWidth, this.frameY * this.SpriteHeight, 
             this.spriteWidth, this.SpriteHeight, this.x, this.y, this.spriteWidth/4, this.SpriteHeight/4)
    }
~~~

- Quiero que el pez esté dentro del circulo rojo, para eso juego con los valores this.x, this.y

~~~js
  ctx.drawImage(playerLeft, this.frameX * this.spriteWidth, this.frameY * this.SpriteHeight, 
             this.spriteWidth, this.SpriteHeight, this.x -60, this.y -45, this.spriteWidth/4, this.SpriteHeight/4)
~~~

- Ahora quiero que cuando vaya a la derecha se gire el pez. Primero uso un if con las coordenadas del mouse.x

~~~js

    draw(){
        if(mouse.click){
            ctx.lineWidth = 0.2;
            ctx.beginPath();
            ctx.moveTo(this.x, this.y);
            ctx.lineTo(mouse.x, mouse.y);
        }
        ctx.fillStyle= 'red';
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI*2)
        ctx.fill();
        ctx.closePath();
        ctx.fillRect(this.x, this.y, this.radius, 10)

        

        if(this.x >= mouse.x){
             ctx.drawImage(playerLeft, this.frameX * this.spriteWidth, this.frameY * this.SpriteHeight, 
             this.spriteWidth, this.SpriteHeight, this.x -60, this.y -45, this.spriteWidth/4, this.SpriteHeight/4)
            }else {
                ctx.drawImage(playerRight, this.frameX * this.spriteWidth, this.frameY * this.SpriteHeight, 
                this.spriteWidth, this.SpriteHeight, this.x -60, this.y -45, this.spriteWidth/4, this.SpriteHeight/4)
        }

    }
~~~

- Ahora tengo que rotar el pez en el canvas para que mire a la derecha
- Uso el save()
- En el translate primero le paso las coordenadas de donde esta el jugador
- Ahora en el if de la dirección cambio el this.x y el this.y por 0 
- Llamo al this.rotate y le paso el this.angle
- Después de todo llamo al restore para que resetee todas las lalmadas al translate hasta el ctx.save()
- Si ahora cambio el valor de this.angle a 60 en el constructor puedo ver que gira de una manera, pero cómo puedo hacerla relativa entre la posición del jugador y el mouse?
- Con n valor llamado theta. Se calcula así
- en el método update de Player

~~~js
    update(){

        const dx = this.x - mouse.x;
        const dy = this.y - mouse.y;

        let theta = Math.atan2(dy,dx);
        this.angle = theta

        if(mouse.x != this.x){
            this.x -= dx/30;
        }
        if(mouse.y != this.y){
            this.y -= dy/30;
        }
    }
~~~

- de esta manera el ángulo será reclaculado cada Frame