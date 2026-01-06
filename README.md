# hollow-graphic-designer
used to design through hollowraghic syle like ironman
// ------------------------ Final AI Holo Design Studio ------------------------
// Run: npm install express openai body-parser cors
// Then: node AiHoloFinal.js
// Open: http://localhost:3000

const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const { Configuration, OpenAIApi } = require('openai');

const app = express();
const PORT = 3000;

app.use(cors());
app.use(bodyParser.json());

// ----- Replace with your OpenAI API key -----
const configuration = new Configuration({
  apiKey: "YOUR_OPENAI_API_KEY"
});
const openai = new OpenAIApi(configuration);

// Serve frontend
app.get('/', (req, res) => {
  res.send(`<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Holo Design Studio</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body{background:#0a0e27;color:#00f3ff;font-family:'Segoe UI',sans-serif;overflow:hidden;height:100vh;}
#header{padding:20px;background: rgba(10,20,45,0.95); display:flex; justify-content:center;}
#ai-command-input{flex:1;padding:12px;border-radius:10px;border:2px solid #00f3ff;background:rgba(0,0,0,0.6);color:#00f3ff;font-size:16px;}
#generate-btn,#speak-btn{margin-left:10px;padding:12px 20px;background:linear-gradient(135deg,#00f3ff,#0066ff);border:none;border-radius:10px;cursor:pointer;font-weight:bold;}
#scene-wrapper{position:relative;width:100%;height:calc(100vh - 80px);background:radial-gradient(ellipse at center, rgba(0,50,100,0.3) 0%, rgba(10,14,39,0.95) 100%); overflow:hidden;}
#main-canvas{width:100%; height:100%; display:block; cursor:grab;}
#control-panel{position:absolute;right:20px;top:50%;transform:translateY(-50%);width:280px;background: rgba(10,20,45,0.95);border:2px solid #00f3ff;border-radius:12px;padding:15px;max-height:80vh;overflow-y:auto;}
#control-panel h3{color:#00f3ff;text-align:center;margin-bottom:15px;}
.control-group{margin-bottom:15px;}
.control-group label{display:block;margin-bottom:5px;font-size:12px;}
.control-group input[type=range]{width:100%;}
#loading{position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);background:rgba(10,20,45,0.95);padding:30px 50px;border-radius:15px;border:2px solid #00f3ff;display:none;z-index:1000;text-align:center;}
.loader{width:40px;height:40px;border:4px solid rgba(0,243,255,0.3);border-top:4px solid #00f3ff;border-radius:50%;animation:spin 1s linear infinite;margin:0 auto 15px;}
@keyframes spin{0%{transform:rotate(0deg);}100%{transform:rotate(360deg);}}
</style>
</head>
<body>

<div id="header">
  <input type="text" id="ai-command-input" placeholder="Type or speak: car, building, Iron Man suit...">
  <button id="generate-btn">Generate ✨</button>
  <button id="speak-btn">🎤 Speak</button>
</div>

<div id="scene-wrapper">
  <canvas id="main-canvas"></canvas>
  <div id="control-panel">
    <h3>Object Controls</h3>
    <div class="control-group">
      <label>Scale: <span id="scale-val">1.0</span></label>
      <input type="range" id="scale-control" min="0.1" max="5" step="0.1" value="1">
    </div>
    <div class="control-group">
      <label>Rotation X: <span id="rotx-val">0°</span></label>
      <input type="range" id="rotx-control" min="0" max="360" step="1" value="0">
    </div>
    <div class="control-group">
      <label>Rotation Y: <span id="roty-val">0°</span></label>
      <input type="range" id="roty-control" min="0" max="360" step="1" value="0">
    </div>
    <div class="control-group">
      <label>Rotation Z: <span id="rotz-val">0°</span></label>
      <input type="range" id="rotz-control" min="0" max="360" step="1" value="0">
    </div>
    <div class="control-group">
      <label>Color:</label>
      <input type="color" id="color-control" value="#00f3ff">
    </div>
  </div>
</div>

<div id="loading"><div class="loader"></div><p>Generating...</p></div>

<script>
let scene = new THREE.Scene();
let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
let renderer = new THREE.WebGLRenderer({canvas:document.getElementById('main-canvas'), antialias:true, alpha:true});
renderer.setSize(window.innerWidth, window.innerHeight-80);
camera.position.set(0,5,15);
let currentObject = null;
let controls={scale:1, rotx:0, roty:0, rotz:0, color:'#00f3ff'};

scene.add(new THREE.AmbientLight(0xffffff,0.5));
let dir = new THREE.DirectionalLight(0xffffff,1);
dir.position.set(10,20,10); scene.add(dir);
scene.add(new THREE.GridHelper(30,30,0x00f3ff,0x003344));

function animate(){ requestAnimationFrame(animate); if(currentObject){ currentObject.rotation.y+=0.005;} renderer.render(scene,camera);}
animate();

// UI sliders
document.getElementById('scale-control').addEventListener('input',(e)=>{controls.scale=parseFloat(e.target.value); document.getElementById('scale-val').textContent=controls.scale.toFixed(1); if(currentObject) currentObject.scale.setScalar(controls.scale);});
['x','y','z'].forEach(a=>{document.getElementById('rot'+a+'-control').addEventListener('input',(e)=>{let val=parseInt(e.target.value); document.getElementById('rot'+a+'-val').textContent=val+'°'; if(currentObject) currentObject.rotation[a]=THREE.MathUtils.degToRad(val);});});
document.getElementById('color-control').addEventListener('input',(e)=>{controls.color=e.target.value;if(currentObject) currentObject.traverse(c=>{if(c.isMesh)c.material.color.set(controls.color);});});

// AI call
async function callAI(prompt){
  try{
    const res = await fetch('/api/generate',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({prompt})});
    if(!res.ok) throw new Error('Network response was not ok');
    const data = await res.json();
    return data.components || [];
  }catch(err){console.error('AI request failed:',err); alert('AI generation failed.'); return [];}
}

async function generateObject(prompt){
  if(currentObject) scene.remove(currentObject);
  document.getElementById('loading').style.display='block';
  const components = await callAI(prompt);
  currentObject = new THREE.Group();
  components.forEach(c=>{
    let mat=new THREE.MeshStandardMaterial({color:c.color||0x00f3ff});
    let mesh;
    switch(c.shape){
      case 'cube': mesh=new THREE.Mesh(new THREE.BoxGeometry(1,1,1),mat); break;
      case 'sphere': mesh=new THREE.Mesh(new THREE.SphereGeometry(0.5,32,32),mat); break;
      case 'cylinder': mesh=new THREE.Mesh(new THREE.CylinderGeometry(0.5,0.5,1,32),mat); break;
      default: mesh=new THREE.Mesh(new THREE.BoxGeometry(1,1,1),mat); break;
    }
    mesh.position.set(c.position?.x||0,c.position?.y||0,c.position?.z||0);
    currentObject.add(mesh);
  });
  scene.add(currentObject);
  document.getElementById('loading').style.display='none';
}

// Button events
document.getElementById('generate-btn').addEventListener('click',()=>{let prompt=document.getElementById('ai-command-input').value.trim(); if(!prompt){alert('Type something!'); return;} generateObject(prompt);});
document.getElementById('speak-btn').addEventListener('click',()=>{
  const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
  recognition.lang = 'en-US';
  recognition.start();
  recognition.onresult = e=>{let text=e.results[0][0].transcript; document.getElementById('ai-command-input').value=text; generateObject(text);};
});

// ------------------------ Hand gesture controls ------------------------
const video=document.createElement('video'); video.style.display='none'; document.body.appendChild(video);
const hands = new Hands({locateFile:(file)=>\`https://cdn.jsdelivr.net/npm/@mediapipe/hands/\${file}\`});
hands.setOptions({maxNumHands:1,minDetectionConfidence:0.7,minTrackingConfidence:0.7});
hands.onResults(results=>{
  if(!currentObject || !results.multiHandLandmarks) return;
  const lm = results.multiHandLandmarks[0];
  const dx = lm[4].x-lm[8].x;
  const dy = lm[4].y-lm[8].y;
  let dist = Math.sqrt(dx*dx + dy*dy);
  currentObject.scale.setScalar(dist*10); 
  currentObject.position.x = (lm[0].x-0.5)*20;
  currentObject.position.y = (0.5-lm[0].y)*15;
  currentObject.rotation.y = lm[8].x*Math.PI*2;
  currentObject.rotation.x = lm[8].y*Math.PI;
});

const cameraHands = new Camera(video,{onFrame: async()=>{await hands.send({image:video});},width:640,height:480});
cameraHands.start();

window.addEventListener('resize',()=>{camera.aspect=window.innerWidth/window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight-80);});
</script>

</body>
</html>`);
});

// AI endpoint
app.post('/api/generate', async (req, res) => {
  const { prompt } = req.body;
  if(!prompt) return res.status(400).json({error:"Prompt required"});
  try {
    const response = await openai.chat.completions.create({
      model: "gpt-4",
      messages: [
        {role:"system", content:"You are an AI 3D designer that outputs JSON with objects: cube, sphere, cylinder, or custom mesh. Include color, shape, position {x,y,z}."},
        {role:"user", content:`Generate 3D components for: ${prompt}`} 
      ],
      temperature: 0.7,
      max_tokens: 600
    });
    let content = response.choices[0].message.content;
    let components;
    try{ components = JSON.parse(content); } catch { components = []; }
    res.json({components});
  } catch(err){console.error(err); res.status(500).json({error:"AI generation failed"});}
});

app.listen(PORT, ()=>console.log(\`AI Holo Design Studio running at http://localhost:\${PORT}\`));
