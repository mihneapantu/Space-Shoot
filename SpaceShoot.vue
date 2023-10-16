<template>
    <v-toolbar style = "position : absolute; top : 0px; left : 0px; height: 7%">
            <v-btn @click = "backToMenu()">
                <v-icon> mdi-arrow-left-bold-circle</v-icon>
            </v-btn>
            <v-app-bar-title>{{name}}</v-app-bar-title>
            <v-btn @click = "togglePause()">
                <v-icon>mdi-pause-octagon</v-icon>
            </v-btn>
            <v-btn @click = "scores()">
                HighScores
            </v-btn>
        </v-toolbar>
        
</template>

<script>
    import * as THREE from 'three'
    import {CSS2DRenderer, CSS2DObject} from 'three/examples/jsm/renderers/CSS2DRenderer' 
    export default{
        name: "SpaceShoot",
        props: ['name','gameId', 'user', 'userId'],
        mounted () {     //cand se incarca pagina se apeleaza ce e in mounted/
            this.init()
        },
        data () {
            return{
                keyboard: {},
                playerSpeed: 0.4,
                spawnPoint: new THREE.Vector3(0, -0.45, 1),
                time: 0,
                time1: 0,
                playerState: 0,
                playerCounter: 0,
                request: 0,
                indicator: 0,
                isPaused: false,
                score: 0,
                coinCollected: false,
                score1: 0
            }
        },
        methods: {
            init () {
                //Event listener
                window.addEventListener('keydown', this.keyDown) //Cand e apasata tasta
                window.addEventListener('keyup', this.keyUp) //Cand nu e apasata
                //Create scene/
                this.scene = new THREE.Scene()

                //Clock
                this.clock = new THREE.Clock()

                //Create camera
                this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
                this.camera.position.y = 1.5
                this.camera.position.z = 5
                this.camera.lookAt(new THREE.Vector3(0, 1.5, 0))

                //Background
                var textureLoader = new THREE.TextureLoader()
                
                var backgroundTexture = textureLoader.load('/WEBDEV_POZE/space.jpg')
                backgroundTexture.wrapS = THREE.RepeatWrapping
                
                this.backgroundMesh = new THREE.Mesh(
                    new THREE.PlaneGeometry(20, 10, 100, 100),
                    new THREE.MeshBasicMaterial({
                        map: backgroundTexture
                    })
                )

                this.backgroundMesh.position.y = 2
                this.scene.add(this.backgroundMesh)  //Adaugam ce am facut in scena

                //Player
                //var playerTexture = textureLoader.load('/WEBDEV_POZE/rocket.png')
                this.playerMesh = new THREE.Mesh(
                        new THREE.PlaneGeometry(1, 1, 50, 50), //50 de triunghiuri pe lungime si latime
                        new THREE.MeshBasicMaterial({
                            map: textureLoader.load('/WEBDEV_POZE/Rocket-Transparent.png'),
                            transparent: true
                        })
                )

                this.playerMesh.position.z = 1
                this.playerMesh.position.x = -3.5
                this.playerMesh.position.y = -0.2
                this.scene.add(this.playerMesh)



                //Obstacle
                this.alien = new THREE.Mesh(
                    new THREE.PlaneGeometry(1, 1, 50, 50),
                    new THREE.MeshBasicMaterial({
                        map: textureLoader.load('/WEBDEV_POZE/alien4.png'),
                        transparent: true
                    })
                )
                this.alien.position.x = this.spawnPoint.x
                this.alien.position.y = this.spawnPoint.y
                this.alien.position.z = this.spawnPoint.z
                this.scene.add(this.alien)
                
                //coin
                
                this.coin = new THREE.Mesh(
                    new THREE.PlaneGeometry(1, 1, 50, 50),
                    new THREE.MeshBasicMaterial({
                        map: textureLoader.load('/WEBDEV_POZE/coin.png'),
                        transparent: true
                    })
                )
                this.coin.position.x = this.spawnPoint.x
                this.coin.position.y = this.spawnPoint.y + 2
                this.coin.position.z = this.spawnPoint.z
                this.scene.add(this.coin)

                
                //Text renderer
                this.timerDiv = document.createElement('div')
				this.timerDiv.id = 'timerDiv'
				this.timerDiv.textContent = 'Time: 0'
				this.timerDiv.style.backgroundColor = 'transparent'
                this.timerDiv.style.fontSize = '30px'
                this.timerDiv.style.color = 'white'
				const timerLabel = new CSS2DObject(this.timerDiv)
				timerLabel.position.set(0, 0, 0)
				this.scene.add(timerLabel)
				timerLabel.layers.set(0)

                this.scoreDiv = document.createElement('div')
				this.scoreDiv.id = 'scoreDiv'
				this.scoreDiv.textContent = 'Score: 0'
				this.scoreDiv.style.backgroundColor = 'transparent'
                this.scoreDiv.style.fontSize = '30px'
                this.scoreDiv.style.color = 'white'
				const scoreLabel = new CSS2DObject(this.scoreDiv)
				scoreLabel.position.set(0, 3, 0)
				this.scene.add(scoreLabel)
				scoreLabel.layers.set(0)

                this.labelRenderer = new CSS2DRenderer()
                this.labelRenderer.setSize(window.innerWidth * 15 / 100, window.innerHeight * 10 / 100)
                this.labelRenderer.domElement.id = "timerLabel"
                this.labelRenderer.domElement.style.position = "absolute"
                this.labelRenderer.domElement.style.left = "5%"
                this.labelRenderer.domElement.style.top = "9%"
                document.body.appendChild(this.labelRenderer.domElement)


                //Rednderer setup/
                this.renderer = new THREE.WebGL1Renderer()
                this.renderer.setSize(window.innerWidth * 90 / 100, window.innerHeight * 90 / 100)
                document.body.appendChild(this.renderer.domElement)
                //STABILIRE ECRAN:
                this.renderer.domElement.id = "canvas"
                this.renderer.domElement.style.position = "absolute"
                this.renderer.domElement.style.left = "5%"
                this.renderer.domElement.style.right = "5%"
                this.renderer.domElement.style.top = "10%"

                this.animate()
            }, 

            animate(){
                if(this.isPaused == false)
                {
                    this.request = requestAnimationFrame(this.animate)
                    this.delta = this.clock.getDelta()
                    this.backgroundMesh.material.map.offset.x += this.playerSpeed * this.delta
                    this.scoreDiv.textContent = "Score: "+ this.score.toFixed(0).toString()

                    this.time += this.delta
                    
                    this.playerCounter += this.delta
                    this.timerDiv.textContent = "Time: "+ this.time.toFixed(1).toString()

                    if(parseInt(this.time) == 45)
                {
                    console.log(this.score)
                    cancelAnimationFrame(this.request)
                }

                    //Player Update
                    // if (this.playerCounter >= 1)
                    // {
                    //     this.playerCounter = 0
                    //     this.playerState += 1
                    //     this.playerState %= 4
                    //     this.playerMesh.material.map = this.sprites[this.playerState]
                    // }

                    if(this.keyboard[39]){
                        if(this.playerSpeed < 0.6)
                        {
                        this.playerSpeed += 2 * this.delta
                        }
                    }else
                    {
                        //this.playerSpeed -= 5 * this.delta
                        if (this.playerSpeed > 0.1)
                        {
                            this.playerSpeed -= 1.5 * this.delta
                        }
                    }

                    if (this.keyboard[38]) { 
                        if (this.playerMesh.position.y < -0.1) { 
                            this.playerMesh.position.y += 1.95
                        }
                    }
                    if(this.keyboard[40]){
                        if(this.playerMesh.position.y > -0.1) {
                            this.playerMesh.position.y -= 1.95
                        }
                    }

                    //Obstacle movement
                    this.alien.position.x -= this.playerSpeed * this.delta * 10
                    this.coin.position.x -= this.playerSpeed * this.delta * 10


                    //Scapam de piatra dupa ce am trecut de ea ca sa nu umplem memoria
                    if(this.alien.position < -5){
                        this.alien.material.dispose()
                        this.alien.geometry.dispose()
                        this.alien.removeFromParent()
                    }

                    //Enemy collision
                    var playerBoundingBox = new THREE.Box3().setFromObject(this.playerMesh)

                    var alienBoundingBox = new THREE.Box3().setFromObject(this.alien)
                    var collision = playerBoundingBox.intersectsBox(alienBoundingBox)

                    //Coin collision

                    var coinBoundingBox = new THREE.Box3().setFromObject(this.coin)
                    var collision2 = playerBoundingBox.intersectsBox(coinBoundingBox)
                                    
                    

                    if(collision == true)
                    {
                       // this.isPaused = true
                        console.log("Game over")
                        //console.log(this.rock.position.x)
                        //this.firstRockPosition = this.rock.position.x
                        console.log(this.score.toFixed(0))
                        cancelAnimationFrame(this.request)
                        //this.playerSpeed = 0
                             
                        
                    }

                    if(collision2 == true && this.coinCollected == false)
                    {
                       // this.isPaused = true
                        console.log("Coin collected")
                        this.score = (this.score + 1)
                        this.coinCollected = true
                             
                    }
                    if(collision2 == false)
                    {
                        this.coinCollected = false
                    }
                    

                    

                    var i = this.generateRandomBinary()
                    //alien spawn
                    
                        if(this.spawnPoint.x - this.alien.position.x > 7)
                        {
                            this.alien.position.x = this.spawnPoint.x + 1
                            
                             if(i == 0)
                            {
                                this.alien.position.y = this.spawnPoint.y 
                            }
                            else{
                                this.alien.position.y = this.spawnPoint.y + 2
                            }

                            this.alien.position.z = this.spawnPoint.z
                            this.scene.add(this.alien) 
                        
                        }

                        //coin spawn

                        if(this.spawnPoint.x - this.coin.position.x > 7)
                        {
                            this.coin.position.x = this.spawnPoint.x + 1
                            if(i == 1)
                            {
                                this.coin.position.y = this.spawnPoint.y 
                            }
                            else{
                                this.coin.position.y = this.spawnPoint.y + 2
                            }
                            this.coin.position.z = this.spawnPoint.z
                            this.scene.add(this.alien) 
                        
                        }
                    
                    

                    this.renderer.render(this.scene, this.camera)
                    this.labelRenderer.render(this.scene, this.camera)

                }
                
            },

            

            keyDown(event){
                this.keyboard[event.keyCode] = true;
            },

            keyUp(event){
                this.keyboard[event.keyCode] = false;
            },

            backToMenu(){
                this.$emit("changeState", 1)
                document.getElementById('canvas').remove()
                document.getElementById('timerLabel').remove()
                document.getElementById('timerDiv').remove()

            },

            scores(){
                this.$emit("changeState", 4)
                document.getElementById('canvas').remove()
                document.getElementById('timerLabel').remove()
                document.getElementById('timerDiv').remove()
            },

            generateRandomBinary() {
                return Math.floor(Math.random() * 2);
            },

            // togglePause() {
            //     this.isPaused = !this.isPaused
            //     if (this.isPaused) {
            //         cancelAnimationFrame(this.request)
            //         this.time1 = this.time
            //         this.playerSpeed = 0
            //     } else {
            //         this.animate()
            //         this.playerSpeed = 0.1
            //         this.time = this.time1
            //     }
            //},
            mounted() {
                this.init();
                this.animate(); // Start the animation loop
            },

            generateRandomBinary() {
                return Math.floor(Math.random() * 2);
            },

            togglePause() {
                this.isPaused = !this.isPaused
                if (this.isPaused) {
                    cancelAnimationFrame(this.request)
                    this.time1 = this.time
                    this.score1 = this.score
                    this.playerSpeed = 0
                } else {
                    this.animate()
                    this.time = this.time1
                    this.score = this.score1
                    this.playerSpeed = 0.4
                }
            },
        }
    }
</script>