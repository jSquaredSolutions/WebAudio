# WebAudio
  You need to start the note server with demo dot J's to get this to work 


  intervalId = setInterval(function(){
    console.log(panner.positionX.value);
    audioElement.play();
    panner.positionX.value = panner.positionX.value + 20;
    console.log(panner.positionX.value);
   }, 5000);

   // Draws a canvas and tracks mouse click/drags on the canvas.
function Field(canvas) {
  this.ANGLE_STEP = 0.2;
  this.canvas = canvas;
  this.isMouseInside = false;
  this.center = {x: canvas.width/2, y: canvas.height/2};
  this.angle = 0;
  this.point = null;

  var obj = this;
  // Setup mouse listeners.
  canvas.addEventListener('mouseover', function() {
    obj.handleMouseOver.apply(obj, arguments)
  });
  canvas.addEventListener('mouseout', function() {
    obj.handleMouseOut.apply(obj, arguments)
  });
  canvas.addEventListener('mousemove', function() {
    obj.handleMouseMove.apply(obj, arguments)
  });
  canvas.addEventListener('mousewheel', function() {
    obj.handleMouseWheel.apply(obj, arguments);
  });
  // Setup keyboard listener
  canvas.addEventListener('keydown', function() {
    obj.handleKeyDown.apply(obj, arguments);
  });

  this.manIcon = new Image();
  this.manIcon.src = 'res/man.svg';

  this.speakerIcon = new Image();
  this.speakerIcon.src = 'res/speaker.svg';

  // Render the scene when the icon has loaded.
  var ctx = this;
  this.manIcon.onload = function() {
    ctx.render();
  }
}

Field.prototype.render = function() {
  // Draw points onto the canvas element.
  var ctx = this.canvas.getContext('2d');
  ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

  ctx.drawImage(this.manIcon, this.center.x - this.manIcon.width/2,
                              this.center.y - this.manIcon.height/2);
  ctx.fill();

  if (this.point) {
    // Draw it rotated.
    ctx.save();
    ctx.translate(this.point.x, this.point.y);
    ctx.rotate(this.angle);
    ctx.translate(-this.speakerIcon.width/2, -this.speakerIcon.height/2);
    ctx.drawImage(this.speakerIcon, 0, 0);
    //ctx.drawImage(this.speakerIcon, this.point.x - this.speakerIcon.width/2,
    //                                this.point.y - this.speakerIcon.height/2);
    ctx.restore();
  }
  ctx.fill();
};

Field.prototype.handleMouseOver = function(e) {
  this.isMouseInside = true;
};

Field.prototype.handleMouseOut = function(e) {
  this.isMouseInside = false;
  if (this.callback) {
    this.callback(null);
  }
  this.point = null;
  this.render();
};

Field.prototype.handleMouseMove = function(e) {
  if (this.isMouseInside) {
    // Update the position.
    this.point = {x: e.offsetX, y: e.offsetY};
    // Re-render the canvas.
    this.render();
    // Callback.
    if (this.callback) {
      // Callback with -0.5 < x, y < 0.5
      this.callback({x: this.point.x - this.center.x,
                     y: this.point.y - this.center.y});
    }
  }
};

Field.prototype.handleKeyDown = function(e) {
  // If it's right or left arrow, change the angle.
  if (e.keyCode == 37) {
    this.changeAngleHelper(-this.ANGLE_STEP);
  } else if (e.keyCode == 39) {
    this.changeAngleHelper(this.ANGLE_STEP);
  }
};

Field.prototype.handleMouseWheel = function(e) {
  e.preventDefault();
  this.changeAngleHelper(e.wheelDelta/500);
};

Field.prototype.changeAngleHelper = function(delta) {
  this.angle += delta;
  if (this.angleCallback) {
    this.angleCallback(this.angle);
  }
  this.render();
}

Field.prototype.registerPointChanged = function(callback) {
  this.callback = callback;
};

Field.prototype.registerAngleChanged = function(callback) {
  this.angleCallback = callback;
};

// Super version: http://chromium.googlecode.com/svn/trunk/samples/audio/simple.html

function PositionSample(el, context) {
  var urls = ['sounds/position.wav'];
  var sample = this;
  this.isPlaying = false;
  this.size = {width: 400, height: 300};

  // Load the sample to pan around.
  var loader = new BufferLoader(context, urls, function(buffers) {
    sample.buffer = buffers[0];
  });
  loader.load();

  // Create a new canvas element.
  var canvas = document.createElement('canvas');
  canvas.setAttribute('width', this.size.width);
  canvas.setAttribute('height', this.size.height);
  el.appendChild(canvas);

  // Create a new Area.
  field = new Field(canvas);
  field.registerPointChanged(function() {
    sample.changePosition.apply(sample, arguments);
  });
  field.registerAngleChanged(function() {
    sample.changeAngle.apply(sample, arguments);
  });
}

PositionSample.prototype.play = function() {
  // Hook up the audio graph for this sample.
  var source = context.createBufferSource();
  source.buffer = this.buffer;
  source.loop = true;
  var panner = context.createPanner();
  panner.coneOuterGain = 0.1;
  panner.coneOuterAngle = 180;
  panner.coneInnerAngle = 0;
  // Set the panner node to be at the origin looking in the +x
  // direction.
  panner.connect(context.destination);
  source.connect(panner);
  source.start(0);
  // Position the listener at the origin.
  context.listener.setPosition(0, 0, 0);

  // Expose parts of the audio graph to other functions.
  this.source = source;
  this.panner = panner;
  this.isPlaying = true;
}

PositionSample.prototype.stop = function() {
  this.source.stop(0);
  this.isPlaying = false;
}

PositionSample.prototype.changePosition = function(position) {
  // Position coordinates are in normalized canvas coordinates
  // with -0.5 < x, y < 0.5
  if (position) {
    if (!this.isPlaying) {
      this.play();
    }
    var mul = 2;
    var x = position.x / this.size.width;
    var y = -position.y / this.size.height;
    this.panner.setPosition(x * mul, y * mul, -0.5);
  } else {
    this.stop();
  }
};

PositionSample.prototype.changeAngle = function(angle) {
//  console.log(angle);
  // Compute the vector for this angle.
  this.panner.setOrientation(Math.cos(angle), -Math.sin(angle), 1);
};

var createScene = function () {
    // This creates a basic Babylon Scene object (non-mesh)
    var scene = new BABYLON.Scene(engine);

    // Lights
    var light0 = new BABYLON.DirectionalLight("Omni", new BABYLON.Vector3(-2, -5, 2), scene);
    var light1 = new BABYLON.PointLight("Omni", new BABYLON.Vector3(2, -5, -2), scene);

    // Need a free camera for collisions
    var camera = new BABYLON.FreeCamera("FreeCamera", new BABYLON.Vector3(0, -8, -20), scene);
    camera.attachControl(canvas, true);

    //Ground
    var ground = BABYLON.Mesh.CreatePlane("ground", 400.0, scene);
    ground.material = new BABYLON.StandardMaterial("groundMat", scene);
    ground.material.diffuseColor = new BABYLON.Color3(1, 1, 1);
    ground.material.backFaceCulling = false;
    ground.position = new BABYLON.Vector3(5, -10, -15);
    ground.rotation = new BABYLON.Vector3(Math.PI / 2, 0, 0);

    //Simple crate
    var box = BABYLON.Mesh.CreateBox("crate", 2, scene);
    box.material = new BABYLON.StandardMaterial("Mat", scene);
    box.material.diffuseTexture = new BABYLON.Texture("textures/crate.png", scene);
    box.position = new BABYLON.Vector3(10, -9, 0);

    // Create and load the sound async
    var music = new BABYLON.Sound("Violons", "sounds/violons11.wav", scene, function () {
        // Call with the sound is ready to be played (loaded & decoded)
        // TODO: add your logic
        console.log("Sound ready to be played!");
    }, { loop: true, autoplay: true });

    // Sound will now follow the mesh position
    music.attachToMesh(box);

    //Set gravity for the scene (G force like, on Y-axis)
    scene.gravity = new BABYLON.Vector3(0, -0.9, 0);

    // Enable Collisions
    scene.collisionsEnabled = true;

    //Then apply collisions and gravity to the active camera
    camera.checkCollisions = true;
    camera.applyGravity = true;

    //Set the ellipsoid around the camera (e.g. your player's size)
    camera.ellipsoid = new BABYLON.Vector3(1, 1, 1);

    //finally, say which mesh will be collisionable
    ground.checkCollisions = true;

    var alpha = 0;

    scene.registerBeforeRender(function () {
        // Moving the box will automatically move the associated sound attached to it
        box.position = new BABYLON.Vector3(Math.cos(alpha) * 30, -9, Math.sin(alpha) * 30);
        alpha += 0.01;
    });

    return scene;
};