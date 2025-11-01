# dining-philosophers..
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Dining Philosophers Auto Simulator</title>
  <style>
    body{
      font-family:Inter,system-ui,Segoe UI,Roboto,Arial;
      max-width:1000px;
      margin:20px auto;
      padding:10px;
      background-color:#000; /* black */
      color:white;
    }
    h1{font-size:22px;margin-bottom:10px;color:white}
    .controls{display:flex;gap:10px;flex-wrap:wrap;margin:12px 0}
    input[type=number]{width:80px}
    .container{display:flex;gap:20px;align-items:flex-start}
    .philos{display:flex;flex-wrap:wrap;gap:10px;margin-top:20px;flex:1}
    .phil{width:120px;border-radius:8px;padding:10px;border:1px solid #444;box-shadow:0 2px 6px rgba(255,255,255,0.1);background:#111;color:white;text-align:center}
    .phil .title{font-weight:600}
    .status{margin-top:8px}
    .eat{color:lime;font-weight:700}
    .hungry{color:orange}
    .idle{color:#999}
    .log{width:360px;background:#111;border-radius:8px;padding:10px;height:420px;overflow:auto;color:white;border:1px solid #333}
    .log h3{margin-top:0;color:white}
    .log-entry{margin-bottom:8px;padding:6px;border-radius:6px;background:rgba(255,255,255,0.08)}
    .controls button{padding:8px 12px;border-radius:6px;border:1px solid rgba(255,255,255,0.3);background:rgba(255,255,255,0.1);color:white}
  </style>
</head>
<body>
  <h1>Dining Philosophers — Stepwise Continuous Simulation</h1>
  <p>Enter number of philosophers and hungry count. The simulator will run continuously until hungry count = 0 and log each step (who eats and when).</p>

  <div class="controls">
    <label>Number of philosophers: <input id="tph" type="number" min="1" max="20" value="5"></label>
    <label>Hungry count: <input id="howhung" type="number" min="0" max="20" value="2"></label>
    <button id="start">Start Simulation</button>
    <button id="stop">Stop</button>
    <button id="clearLog">Clear Log</button>
  </div>

  <div class="container">
    <div class="philos" id="philos"></div>
    <div class="log" id="log">
      <h3>Simulation Log</h3>
      <div id="entries"></div>
    </div>
  </div>

  <script>
    const tphInput = document.getElementById('tph');
    const howhungInput = document.getElementById('howhung');
    const startBtn = document.getElementById('start');
    const stopBtn = document.getElementById('stop');
    const clearLogBtn = document.getElementById('clearLog');
    const philosDiv = document.getElementById('philos');
    const entriesDiv = document.getElementById('entries');

    let tph = 0;
    let philname = [];
    let status = [];
    let hu = [];
    let interval = null;
    let stepCount = 0;

    function appendLog(text){
      stepCount++;
      const el = document.createElement('div');
      el.className='log-entry';
      const time = new Date().toLocaleTimeString();
      el.innerHTML = `<strong>Step ${stepCount} — ${time}:</strong> ${text}`;
      entriesDiv.prepend(el);
    }

    function initPhilosophers(){
      if(interval) clearInterval(interval);
      stepCount = 0;
      entriesDiv.innerHTML='';

      tph = parseInt(tphInput.value) || 5;
      const howhung = Math.max(0, Math.min(parseInt(howhungInput.value)||0, tph));

      philname = [];
      status = [];
      hu = [];

      for(let i=0;i<tph;i++){
        philname[i] = i + 1;
        status[i] = 1; // idle initially
      }

      // Randomly assign hungry philosophers
      let chosen = [];
      while(chosen.length < howhung && chosen.length < tph){
        let rand = Math.floor(Math.random() * tph);
        if(!chosen.includes(rand)){
          chosen.push(rand);
          status[rand] = 2; // hungry
          hu.push(rand);
        }
      }

      appendLog(`Initial hungry philosophers: ${hu.map(i=> 'P'+philname[i]).join(', ') || 'none'}`);
      render();

      interval = setInterval(()=>{
        stepOnce();
      },1800);
    }

    function stopSimulation(){
      if(interval) clearInterval(interval);
      interval = null;
      appendLog('Simulation stopped by user');
    }

    function stepOnce(){
      updateHungryList();
      if(hu.length===0){
        appendLog('All hungry philosophers have eaten — simulation complete.');
        clearInterval(interval);
        interval = null;
        return;
      }

      // clear previous eaters
      for(let i=0;i<tph;i++) if(status[i]===3) status[i]=1;

      // choose who eats this step
      let toEat = [];
      toEat.push(hu[0]);
      if(hu.length>1) toEat.push(hu[1]);

      appendLog('Selected to eat: ' + toEat.map(i=>'P'+philname[i]).join(' and '));

      toEat.forEach(idx=>{
        status[idx]=3; // eating
      });

      render();

      // after eating, remove them from hungry list (they become idle/full)
      setTimeout(()=>{
        toEat.forEach(idx=>{
          if(status[idx]===3) status[idx]=1;
        });
        updateHungryList();
        hu = hu.filter(i => status[i]===2);
        appendLog('Finished eating: ' + toEat.map(i=>'P'+philname[i]).join(' and '));
        render();
      },1000);
    }

    function render(){
      philosDiv.innerHTML = '';
      for(let i=0;i<tph;i++){
        const box = document.createElement('div');
        box.className='phil';
        const title = document.createElement('div');
        title.className='title';
        title.textContent = 'Philosopher ' + philname[i];
        const st = document.createElement('div');
        st.className='status';
        if(status[i]===1) st.innerHTML = '<span class="idle">Idle</span>';
        else if(status[i]===2) st.innerHTML = '<span class="hungry">Hungry</span>';
        else if(status[i]===3) st.innerHTML = '<span class="eat">Eating</span>';
        box.appendChild(title);
        box.appendChild(st);
        philosDiv.appendChild(box);
      }
    }

    function updateHungryList(){
      hu = [];
      for(let i=0;i<tph;i++) if(status[i]===2) hu.push(i);
    }

    startBtn.onclick = initPhilosophers;
    stopBtn.onclick = stopSimulation;
    clearLogBtn.onclick = ()=>{ entriesDiv.innerHTML=''; stepCount=0; };
  </script>
</body>
</html>
