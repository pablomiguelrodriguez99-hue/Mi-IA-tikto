# Mi-IA-tikto
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tablero IA 3.0</title>
<style>
:root {
  --primary: #69C9D0;
  --secondary: #FF2D55;
  --dark: #1C1C1E;
  --light: #F2F2F7;
  --gray: #8E8E93;
}
body {
  margin: 0;
  padding: 20px;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(135deg, var(--dark) 0%, #0F0F11 100%);
  color: var(--light);
  min-height: 100vh;
}
header {
  text-align: center;
  margin-bottom: 30px;
}
h1 {
  font-size: 2.5rem;
  background: linear-gradient(45deg, var(--primary), var(--secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 10px;
}
.subtitle {
  color: var(--gray);
  font-size: 1.1rem;
}
.stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px,1fr));
  gap: 20px;
  margin-bottom: 20px;
}
.stat-card {
  background: rgba(255,255,255,0.05);
  padding: 15px;
  border-radius: 12px;
  text-align: center;
}
.stat-value {
  font-size: 2rem;
  font-weight: bold;
  margin-top: 5px;
  background: linear-gradient(45deg,var(--primary),var(--secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
.controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}
button {
  padding: 12px 25px;
  border-radius: 50px;
  border: none;
  cursor: pointer;
  font-weight: bold;
  font-size: 1rem;
  background: linear-gradient(45deg,var(--primary),#4EC6CD);
  color: var(--dark);
  transition: transform 0.2s;
}
button:hover { transform: translateY(-2px);}
button:disabled { opacity:0.5; cursor:not-allowed;}
.agents-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px,1fr));
  gap: 10px;
  margin-bottom: 20px;
}
.agent {
  background: rgba(28,28,30,0.7);
  border-radius: 8px;
  padding: 8px;
  text-align: center;
  transition: all 0.3s ease;
  animation: pulse 2s infinite;
}
.agent-id { font-size:0.8rem; color: var(--gray);}
.agent-state { font-weight: bold; margin-top:5px; }
.energy-bar {
  height: 4px;
  background: #333;
  border-radius: 2px;
  overflow: hidden;
  margin-top:5px;
}
.energy-fill {
  height: 100%;
  border-radius: 2px;
  transition: width 0.5s ease;
}
.energy-high { background: #69C9D0; }
.energy-medium { background: #FFD700; }
.energy-low { background: #FFA500; }
.energy-empty { background: var(--gray); }
.log {
  background: rgba(0,0,0,0.3);
  padding: 10px;
  border-radius: 8px;
  font-family: monospace;
  font-size: 12px;
  max-height:200px;
  overflow-y:auto;
}
@keyframes pulse {
  0% { opacity: 0.6;}
  50% { opacity:1;}
  100% {opacity:0.6;}
}
</style>
</head>
<body>
<header>
<h1>Tablero IA 3.0</h1>
<p class="subtitle">100 agentes controlando 200 cuentas (TikTok + YouTube)</p>
</header>

<div class="stats">
  <div class="stat-card">
    Agentes Activos
    <div class="stat-value" id="activeAgents">0</div>
  </div>
  <div class="stat-card">
    Día Real
    <div class="stat-value" id="currentDay">0</div>
  </div>
  <div class="stat-card">
    Tiempo Total
    <div class="stat-value" id="totalTime">0</div>
    Minutos
  </div>
  <div class="stat-card">
    Estado
    <div class="stat-value" id="status">Detenido</div>
  </div>
</div>

<div class="controls">
<button id="startBtn">▶️ Iniciar</button>
<button id="stopBtn" disabled>⏸️ Detener</button>
<button id="resetBtn">🔄 Reiniciar</button>
</div>

<div class="agents-grid" id="agentsGrid"></div>
<div class="log" id="logContent"></div>

<script>
class Agent {
  constructor(id, accounts) {
    this.id = id;
    this.state = "inactive";
    this.energy = Math.random()*0.5+0.5;
    this.accounts = accounts;
    this.totalActions = 0;
  }
  work() {
    if(this.energy<=0){ this.state="inactive"; return;}
    const account = this.accounts[Math.floor(Math.random()*this.accounts.length)];
    const action = Math.random()<0.5?"TikTok":"YouTube";
    this.energy -= 0.01+Math.random()*0.05;
    this.energy = Math.max(0,this.energy);
    this.state="active";
    this.totalActions++;
    return `${action} (${account})`;
  }
  recover() {
    this.energy = Math.min(1,this.energy+0.05);
    if(this.energy>0.2 && this.state==="inactive") this.state="inactive";
  }
}

class AIOrchestrator {
  constructor(){
    this.agents=[];
    this.day=0;
    this.totalTime=0;
    this.isRunning=false;
    this.interval=null;
    this.accounts=[];
    for(let i=1;i<=100;i++) this.accounts.push("TikTok_"+i);
    for(let i=1;i<=100;i++) this.accounts.push("YouTube_"+i);
  }
  init(){
    this.agents=[];
    for(let i=0;i<100;i++){
      this.agents.push(new Agent(i,this.accounts));
    }
    this.updateStats();
    this.renderAgents();
    this.log("✅ Sistema inicializado con 100 agentes y 200 cuentas");
  }
  runDay(){
    let total=0;
    this.agents.forEach(agent=>{
      const work=this.agents[agent.id].work();
      if(work) this.log(`Agente ${agent.id}: ${work}`);
      agent.recover();
      total++;
    });
    this.totalTime+=total;
    this.day++;
    this.updateStats();
    this.renderAgents();
  }
  start(){
    if(this.isRunning) return;
    this.isRunning=true;
    document.getElementById("startBtn").disabled=true;
    document.getElementById("stopBtn").disabled=false;
    this.interval=setInterval(()=>this.runDay(),1000);
    this.log("▶️ Simulación iniciada");
  }
  stop(){
    clearInterval(this.interval);
    this.isRunning=false;
    document.getElementById("startBtn").disabled=false;
    document.getElementById("stopBtn").disabled=true;
    this.log("⏸️ Simulación detenida");
  }
  reset(){
    this.stop();
    this.day=0;
    this.totalTime=0;
    this.init();
    this.log("♻️ Sistema reiniciado");
  }
  updateStats(){
    const activeCount=this.agents.filter(a=>a.state==="active").length;
    document.getElementById("activeAgents").textContent=activeCount;
    document.getElementById("currentDay").textContent=this.day;
    document.getElementById("totalTime").textContent=this.totalTime;
    document.getElementById("status").textContent=this.isRunning?"Ejecutando":"Detenido";
  }
  renderAgents(){
    const grid=document.getElementById("agentsGrid");
    grid.innerHTML="";
    this.agents.forEach(agent=>{
      const div=document.createElement("div");
      let energyClass="energy-empty";
      if(agent.energy>0.7) energyClass="energy-high";
      else if(agent.energy>0.4) energyClass="energy-medium";
      else if(agent.energy>0.1) energyClass="energy-low";
      div.className="agent";
      div.innerHTML=`<div class="agent-id">Agente ${agent.id}</div>
                     <div class="agent-state ${energyClass}">${agent.state}</div>
                     <div class="energy-bar"><div class="energy-fill ${energyClass}" style="width:${agent.energy*100}%"></div></div>`;
      grid.appendChild(div);
    });
  }
  log(msg){
    const logDiv=document.getElementById("logContent");
    const entry=document.createElement("div");
    entry.className="log-entry";
    entry.textContent=`[${new Date().toLocaleTimeString()}] ${msg}`;
    logDiv.appendChild(entry);
    logDiv.scrollTop=logDiv.scrollHeight;
  }
}

const orchestrator=new AIOrchestrator();
orchestrator.init();

document.getElementById("startBtn").addEventListener("click",()=>orchestrator.start());
document.getElementById("stopBtn").addEventListener("click",()=>orchestrator.stop());
document.getElementById("resetBtn").addEventListener("click",()=>orchestrator.reset());
</script>
</body>
</html>
