<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/addons/p5.sound.min.js"></script>

    <link rel="stylesheet" type="text/css" href="style.css">
    <meta charset="utf-8" />

  </head>
  <body style="background-color:black;">
    <main>
    </main>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.12.0/matter.js"></script>
    <script>
    var Engine = Matter.Engine;
var World = Matter.World;
var Bodies = Matter.Bodies;
var Render = Matter.Render;
var Constraint = Matter.Constraint
var Mouse = Matter.Mouse
var MouseConstraint = Matter.MouseConstraint
var Composites = Matter.Composites
var Composite = Matter.Composite

var engine;
var world;
var temp;
var constraint1;
var render;
var circle;

/*document.addEventListener("keydown", function(event) {
    if (event.key === "e" || event.key === "E") {
        
    }
});*/

function drawConstraintZigZag(ctx, constraint, options = {}) {
    const {
        color = "blue",
        segments = 10,
        amplitude = 10
    } = options;
    const startX = constraint.pointA.x;
    const startY = constraint.pointA.y;
    const endX = constraint.bodyB.position.x;
    const endY = constraint.bodyB.position.y;
    const dx = endX - startX;
    const dy = endY - startY;
    const length = Math.sqrt(dx*dx + dy*dy);
    const ux = dx / length;
    const uy = dy / length;
    const px = -uy;
    const py = ux;
    ctx.beginPath();
    ctx.moveTo(startX, startY);

    for (let i = 1; i <= segments; i++) {
        const t = i / segments;
        const cx = startX + dx * t + px * ((i % 2 === 0 ? 1 : -1) * amplitude);
        const cy = startY + dy * t + py * ((i % 2 === 0 ? 1 : -1) * amplitude);
        ctx.lineTo(cx, cy);
    }

    ctx.strokeStyle = color;
    ctx.lineWidth = 2;
    ctx.stroke();
}

function drawForce(ctx, x, y, fx, fy, txt, scale = 1, color = "red") {
    const endX = x + fx * scale;
    const endY = y + fy * scale;

    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(endX, endY);
    ctx.strokeStyle = color;
    ctx.lineWidth = 3;
    ctx.stroke();
    const angle = Math.atan2(endY - y, endX - x);
    const headLength = 10;
    ctx.beginPath();
    ctx.moveTo(endX, endY);
  
  if(txt!= "")
    {
      ctx.font = "24px Arial";
      ctx.fillStyle = "white";
      
      ctx.fillText(txt, endX+10, endY);
    }
  
    ctx.lineTo(
        endX - headLength * Math.cos(angle - Math.PI / 6),
        endY - headLength * Math.sin(angle - Math.PI / 6)
    );
    ctx.lineTo(
        endX - headLength * Math.cos(angle + Math.PI / 6),
        endY - headLength * Math.sin(angle + Math.PI / 6)
    );
    ctx.lineTo(endX, endY);
    ctx.fillStyle = color;
    ctx.fill();
}

function getCurrentLength(constraint) {
    const ax = constraint.pointA.x;
    const ay = constraint.pointA.y;
    const bx = constraint.bodyB.position.x;
    const by = constraint.bodyB.position.y;
    return Math.sqrt((bx - ax) * (bx - ax) +(by - ay) * (by - ay));
}

function setup() {
  engine = Engine.create();
  world = engine.world;
  
  circle = Bodies.circle(400, 300, 30);
  World.add(world, circle);
  
   constraint1 = Constraint.create({
        pointA: { x: 400, y: 100 },
        bodyB: circle,
        stiffness: 0.01,
       render: {visible: false}
    });
  Composite.add(world, [constraint1]);
  
  
  var wall1 = Bodies.rectangle(400, 0, 800, 10)
  var wall2 = Bodies.rectangle(400, 600, 800, 10)
  var wall3 = Bodies.rectangle(800, 300, 10, 600)
  var wall4 = Bodies.rectangle(0, 300, 10, 600)
  
  render = Render.create({
    element: document.body,
    engine: engine,
    options: {
      width: 800,
      height: 600,
      wireframes: false
    }
  });
  Render.run(render);
  
  var walls = [wall1, wall2, wall3, wall4]
  for (var wall of walls)
  {
    Matter.Body.setStatic(wall, true);
  }
  World.add(world, walls)
  
  var mouse = Mouse.create(render.canvas),
        mouseConstraint = MouseConstraint.create(engine, {
            mouse: mouse,
            constraint: {
                angularStiffness: 0,
                render: {
                    visible: false
                }
            }
        });
  render.mouse = mouse;
  World.add(world, mouseConstraint);


  Engine.run(engine);
  Render.run(render);
  
  Matter.Events.on(render, "afterRender", () => {
    document.getElementById("delta").innerHTML = "Î”x = " + (Math.round(getCurrentLength(constraint1)-constraint1.length)/10).toString() + "m";
    var force = Math.round((getCurrentLength(constraint1)*constraint1.stiffness)*100)/10;
      document.getElementById("force").innerHTML = "F = " + force.toString() + "N";
    var sina = (constraint1.bodyB.position.y - constraint1.pointA.y)/Math.sqrt(Math.pow(constraint1.bodyB.position.x - constraint1.pointA.x, 2) + Math.pow(constraint1.bodyB.position.y - constraint1.pointA.y, 2));
var cosa = (constraint1.bodyB.position.x - constraint1.pointA.x) /Math.sqrt(Math.pow(constraint1.bodyB.position.x - constraint1.pointA.x, 2) +Math.pow(constraint1.bodyB.position.y - constraint1.pointA.y, 2));

    drawConstraintZigZag(render.context, constraint1, { color: "orange", segments: 20, amplitude: 10});
    drawForce(render.context, constraint1.bodyB.position.x, constraint1.bodyB.position.y, -force*cosa, -force*sina, "F", 5, "red");
    drawForce(render.context, constraint1.bodyB.position.x, constraint1.bodyB.position.y, 0, circle.mass*8.172, "G", 5, "blue");
  });
}
    </script>
    <style>
      h1 {color:white;}
    </style>
    <h1 id="delta">Î”x = </h1>
    <h1 id="force">F = </h1>
  </body>
</html>
