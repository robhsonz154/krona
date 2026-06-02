<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>KRONA Analytics</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{font-family:system-ui,sans-serif;background:#060A13;color:#E2E8F0;max-width:480px;margin:0 auto;padding-bottom:90px}
.hdr{background:#0B1120;padding:14px 18px;border-bottom:1px solid #1E3050}
.brand{font-size:20px;font-weight:bold;color:#D4A84B}
.usr-info{text-align:right}
.usr-nm{font-size:14px;font-weight:bold}
.usr-xp{color:#D4A84B}
.lvl-pill{background:#D4A84B;color:#060A13;padding:2px 8px;border-radius:20px;font-size:10px}
.ctn{padding:14px}
.stats{display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-bottom:16px}
.st{background:#101B2E;border-radius:10px;padding:14px;text-align:center}
.st-v{font-size:22px;font-weight:bold;color:#D4A84B}
.sec-h{margin:20px 0 12px}
.sec-tt{color:#D4A84B;font-weight:bold}
.mod{background:#101B2E;border-radius:10px;margin-bottom:8px;overflow:hidden}
.mod-h{padding:14px 16px;display:flex;justify-content:space-between;cursor:pointer}
.mod-nm{font-weight:bold}
.mod-chev{transition:transform .3s}
.mod.open .mod-chev{transform:rotate(90deg)}
.mod-body{max-height:0;overflow:hidden;transition:max-height .3s}
.mod.open .mod-body{max-height:500px}
.mod-inner{padding:12px}
.tema{display:flex;justify-content:space-between;align-items:center;padding:12px;background:#0B1120;border-radius:8px;margin-top:6px;cursor:pointer}
.toast{position:fixed;bottom:80px;left:50%;transform:translateX(-50%);background:#152238;padding:10px 20px;border-radius:10px;color:#D4A84B;opacity:0;transition:opacity .3s}
.toast.show{opacity:1}
.bnav{position:fixed;bottom:0;left:0;right:0;max-width:480px;margin:0 auto;background:#0B1120;display:flex;justify-content:space-around;padding:10px}
.nav-i{text-align:center;padding:6px 14px;border-radius:10px;cursor:pointer}
.nav-i.act{color:#D4A84B}
.loading{text-align:center;padding:40px;color:#D4A84B}
</style>
</head>
<body>

<div class="hdr">
    <div style="display:flex;justify-content:space-between;align-items:center">
        <div class="brand">KRONA Analytics</div>
        <div class="usr-info">
            <div class="usr-nm" id="uName">Carregando...</div>
            <div><span class="usr-xp" id="uXP">0 XP</span> <span class="lvl-pill" id="uLvl">Lv.1</span></div>
        </div>
    </div>
    <div style="margin-top:10px">
        <div>Progresso: <span id="pPct">0%</span></div>
        <div style="background:#1E3050;height:6px;border-radius:3px;margin-top:5px"><div id="pFill" style="width:0%;height:100%;background:#D4A84B;border-radius:3px"></div></div>
        <div id="pInfo" style="font-size:10px;color:#64748B">0 de 15 temas</div>
    </div>
</div>

<div class="ctn">
    <div class="stats">
        <div class="st"><div>✅</div><div class="st-v" id="sHits">0</div><div>Acertos</div></div>
        <div class="st"><div>📝</div><div class="st-v" id="sTot">0</div><div>Questões</div></div>
        <div class="st"><div>🎯</div><div class="st-v" id="sPct">0%</div><div>Aproveit.</div></div>
    </div>

    <div class="sec-h"><span class="sec-tt">◆ MÓDULOS</span> <span id="temasCount">0/15</span></div>
    <div id="modulosContainer"><div class="loading">Carregando...</div></div>
</div>

<div class="bnav">
    <div class="nav-i act" id="nHome">🏠 Início</div>
    <div class="nav-i" id="nRank">🏆 Ranking</div>
    <div class="nav-i" id="nErr">📓 Erros</div>
    <div class="nav-i" id="nRefresh">🔄 Atualizar</div>
</div>
<div class="toast" id="toast"></div>

<script>
const tg = window.Telegram?.WebApp;
if(tg){tg.ready();tg.expand();}
const API_BASE = "https://krona-api.onrender.com";
let temasStats = [];

function showToast(msg){let t=document.getElementById('toast');t.textContent=msg;t.classList.add('show');setTimeout(()=>t.classList.remove('show'),2000);}
function getUserId(){return tg?.initDataUnsafe?.user?.id||localStorage.getItem('test_user_id')||12345;}
function getUserName(){return tg?.initDataUnsafe?.user?.first_name||'Aluno';}

async function carregarDados(){
    let userId=getUserId();
    document.getElementById('uName').textContent=getUserName();
    try{
        let res=await fetch(`${API_BASE}/api/user/${userId}`);
        if(!res.ok)throw new Error('Erro');
        let data=await res.json();
        document.getElementById('uXP').textContent=`${data.xp||0} XP`;
        document.getElementById('uLvl').textContent=`Lv.${data.level||1}`;
        document.getElementById('sHits').textContent=data.acertos||0;
        document.getElementById('sTot').textContent=data.total_questoes||0;
        let pct=data.percentual_geral||0;
        document.getElementById('sPct').textContent=`${pct}%`;
        document.getElementById('pFill').style.width=`${pct}%`;
        document.getElementById('pPct').textContent=`${pct}%`;
        document.getElementById('pInfo').textContent=`${data.temas_completos||0} de 15 temas`;
        document.getElementById('temasCount').textContent=`${data.temas_completos||0}/15`;
        if(data.temas_stats)temasStats=data.temas_stats;
        else criarStatsMock();
        renderModulos();
    }catch(e){console.error(e);showToast('❌ Erro ao conectar');criarStatsMock();renderModulos();}
}

function criarStatsMock(){
    let nomes=["Contextualização","Sistemas Sustentáveis","Botânica","Nutrição Mineral","Irrigação","Culturas","Produção Animal","Anatomia Animal","Nutrição Animal","Forragicultura","Melhoramento Genético","Monogástricos","Ruminantes","Agricultura Precisão","Simulado Bônus"];
    temasStats=[];
    for(let i=0;i<15;i++)temasStats.push({id:`TEMA_${i+1}`,nome:nomes[i],completado:false,acertos:0,total:15});
}

function renderModulos(){
    let container=document.getElementById('modulosContainer');
    let modulos=[
        {id:'A',nome:'Agronegócio',icon:'🌾',temas:[0,1]},{id:'B',nome:'Vegetal & Solo',icon:'🌿',temas:[2,3]},
        {id:'C',nome:'Recursos Hídricos',icon:'💧',temas:[4]},{id:'D',nome:'Culturas',icon:'🌽',temas:[5]},
        {id:'E',nome:'Produção Animal I',icon:'🐄',temas:[6,7,8]},{id:'F',nome:'Forragem & Genética',icon:'🌾',temas:[9,10]},
        {id:'G',nome:'Produção Animal II',icon:'🐔',temas:[11,12]},{id:'H',nome:'Tecnologia',icon:'📡',temas:[13]},
        {id:'X',nome:'Bônus',icon:'🎁',temas:[14]}
    ];
    let html='';
    for(let m of modulos){
        let temasMod=m.temas.map(i=>temasStats[i]);
        let completos=temasMod.filter(t=>t?.completado).length;
        html+=`<div class="mod" id="mod${m.id}">
            <div class="mod-h" onclick="toggleMod('${m.id}')"><span><span style="font-size:20px;margin-right:10px">${m.icon}</span>${m.nome}</span><span>${completos}/${m.temas.length} <span class="mod-chev">▸</span></span></div>
            <div class="mod-body"><div class="mod-inner">`;
        for(let t of temasMod){
            if(!t)continue;
            let num=t.id.split('_')[1];
            html+=`<div class="tema" onclick="startQuiz('${t.id}','${t.nome}')">
                <span>${m.icon} Tema ${num}</span>
                <span>${t.completado?'✅ Concluído':'📖 Iniciar'}</span>
            </div>`;
        }
        html+=`</div></div></div>`;
    }
    container.innerHTML=html;
}

function toggleMod(id){let el=document.getElementById('mod'+id);el.classList.toggle('open');}
function startQuiz(temaId,temaNome){tg?.showConfirm?tg.showConfirm(`Iniciar ${temaNome}?`,(ok)=>{if(ok)enviarDados(temaId);}):confirm(`Iniciar ${temaNome}?`)&&enviarDados(temaId);}
function enviarDados(temaId){showToast('Iniciando simulado...');tg?.sendData(JSON.stringify({action:'start_quiz',tema_id:temaId}));setTimeout(()=>tg?.close(),500);}
async function mostrarRanking(){
    try{
        let res=await fetch(`${API_BASE}/api/ranking`);
        let data=await res.json();
        let txt='🏆 RANKING\n━━━━━━━━━━━\n';
        if(!data.ranking?.length)txt+='Sem dados ainda. Complete um tema!';
        else for(let i=0;i<Math.min(data.ranking.length,5);i++)txt+=`${i+1}. ${data.ranking[i].user_name} - ${data.ranking[i].xp} XP\n`;
        tg?.showPopup?tg.showPopup({title:'Ranking',message:txt,buttons:[{type:'ok'}]}):alert(txt);
    }catch(e){showToast('Erro ao carregar ranking');}
}
async function mostrarErros(){
    let userId=getUserId();
    try{
        let res=await fetch(`${API_BASE}/api/errors/${userId}?limit=10`);
        let data=await res.json();
        if(!data.errors?.length){tg?.showPopup?tg.showPopup({title:'Erros',message:'Nenhum erro registrado!',buttons:[{type:'ok'}]}):alert('Nenhum erro!');return;}
        let txt=`📓 ERROS (${data.errors.length})\n`;
        for(let e of data.errors.slice(0,5))txt+=`❓ ${e.pergunta.substring(0,60)}...\n✅ ${e.correta}\n\n`;
        tg?.showPopup?tg.showPopup({title:'Caderno de Erros',message:txt,buttons:[{type:'ok'}]}):alert(txt);
    }catch(e){showToast('Erro ao carregar erros');}
}
function initNav(){
    document.getElementById('nHome')?.addEventListener('click',()=>{location.reload();});
    document.getElementById('nRank')?.addEventListener('click',()=>{mostrarRanking();});
    document.getElementById('nErr')?.addEventListener('click',()=>{mostrarErros();});
    document.getElementById('nRefresh')?.addEventListener('click',()=>{location.reload();});
}
window.addEventListener('load',()=>{initNav();carregarDados();});
window.toggleMod=toggleMod;window.startQuiz=startQuiz;
</script>
</body>
</html>
