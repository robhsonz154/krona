<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>KRONA Analytics — Simulado Agronegócio</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
<style>
*{margin:0;padding:0;box-sizing:border-box}
:root{
  --bg:#060A13;
  --bg2:#0B1120;
  --sf:#101B2E;
  --sf2:#152238;
  --sf3:#1A2B48;
  --bd:#1E3050;
  --bd2:#264070;
  --gold:#D4A84B;
  --gold2:#E8C46E;
  --gold3:#C49A3C;
  --goldG:rgba(212,168,75,.06);
  --goldG2:rgba(212,168,75,.12);
  --grn:#22C55E;
  --grnD:#16A34A;
  --grnG:rgba(34,197,94,.08);
  --red:#EF4444;
  --redG:rgba(239,68,68,.08);
  --blu:#3B82F6;
  --vio:#8B5CF6;
  --org:#F59E0B;
  --txt:#E2E8F0;
  --tx2:#94A3B8;
  --tx3:#64748B;
  --tx4:#475569;
  --r:14px;
  --rs:10px;
  --sh:0 2px 12px rgba(0,0,0,.3);
  --sh2:0 4px 24px rgba(0,0,0,.35);
  --tr:.25s cubic-bezier(.4,0,.2,1)
}
html{background:var(--bg)}
body{
  font-family:'Inter',system-ui,sans-serif;
  background:var(--bg);color:var(--txt);
  min-height:100vh;max-width:480px;
  margin:0 auto;overflow-x:hidden;
  -webkit-font-smoothing:antialiased;
  padding-bottom:90px;
}
.hdr{
  background:linear-gradient(170deg,#0D1525 0%,var(--bg2) 100%);
  position:sticky;top:0;z-index:100;
  border-bottom:1px solid var(--bd);
}
.hdr-top{
  display:flex;align-items:center;
  justify-content:space-between;
  padding:14px 18px 8px;
}
.brand{display:flex;align-items:center;gap:10px}
.brand-mark{
  width:36px;height:36px;border-radius:8px;
  background:linear-gradient(135deg,var(--gold),var(--gold2));
  display:flex;align-items:center;
  justify-content:center;
  box-shadow:0 2px 12px rgba(212,168,75,.25);
}
.brand-mark span{font-size:16px;font-weight:900;color:var(--bg);letter-spacing:-1px;}
.brand-txt{display:flex;flex-direction:column}
.brand-name{font-size:17px;font-weight:900;color:var(--gold);letter-spacing:2px;line-height:1;}
.brand-sub{font-size:8px;font-weight:600;color:var(--tx3);letter-spacing:2.5px;text-transform:uppercase;margin-top:2px;}
.usr{display:flex;align-items:center;gap:10px}
.usr-info{text-align:right}
.usr-lbl{font-size:8px;color:var(--tx3);font-weight:600;text-transform:uppercase;letter-spacing:.5px;}
.usr-nm{font-size:12px;color:var(--txt);font-weight:700;line-height:1.2;}
.usr-xp-row{display:flex;align-items:center;gap:5px;justify-content:flex-end;margin-top:2px;}
.usr-xp{font-size:11px;color:var(--gold);font-weight:700;}
.lvl-pill{background:linear-gradient(135deg,var(--gold),var(--gold2));color:var(--bg);font-size:8px;font-weight:900;padding:2px 7px;border-radius:20px;letter-spacing:.3px;}
.usr-av{width:34px;height:34px;border-radius:50%;background:linear-gradient(135deg,var(--sf2),var(--sf3));border:2px solid var(--bd);display:flex;align-items:center;justify-content:center;font-size:13px;font-weight:800;color:var(--gold);}
.prog-strip{padding:10px 18px 12px;background:rgba(0,0,0,.2);}
.prog-row{display:flex;justify-content:space-between;align-items:center;margin-bottom:5px;}
.prog-t{font-size:9px;color:var(--tx3);font-weight:600;text-transform:uppercase;letter-spacing:1.2px;}
.prog-p{font-size:13px;color:var(--gold);font-weight:800;}
.prog-track{background:var(--sf);border-radius:20px;height:6px;overflow:hidden;}
.prog-fill{height:100%;border-radius:20px;background:linear-gradient(90deg,var(--gold3),var(--gold),var(--gold2));transition:width .8s ease;}
.prog-info{text-align:center;margin-top:4px;font-size:9px;color:var(--tx4);font-weight:500;}
.ctn{padding:14px 14px 24px}
.stats{display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-bottom:16px;}
.st{background:var(--sf);border:1px solid var(--bd);border-radius:var(--rs);padding:14px 8px;text-align:center;position:relative;overflow:hidden;}
.st::before{content:'';position:absolute;top:0;left:0;right:0;height:2px;}
.st:nth-child(1)::before{background:linear-gradient(90deg,var(--grn),#4ADE80)}
.st:nth-child(2)::before{background:linear-gradient(90deg,var(--blu),#60A5FA)}
.st:nth-child(3)::before{background:linear-gradient(90deg,var(--gold),var(--gold2))}
.st-ic{font-size:18px;margin-bottom:4px}
.st-v{font-size:22px;font-weight:900;line-height:1}
.st:nth-child(1) .st-v{color:var(--grn)}
.st:nth-child(2) .st-v{color:var(--blu)}
.st:nth-child(3) .st-v{color:var(--gold)}
.st-l{font-size:8px;color:var(--tx3);font-weight:700;text-transform:uppercase;letter-spacing:.6px;margin-top:4px;}
.pnl{background:var(--sf);border:1px solid var(--bd);border-radius:var(--r);margin-bottom:16px;overflow:hidden;}
.pnl-h{padding:14px 16px 12px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid var(--bd);}
.pnl-tg{display:flex;align-items:center;gap:8px}
.pnl-ic{font-size:16px}
.pnl-tt{font-size:13px;font-weight:800;color:var(--txt);letter-spacing:-.2px;}
.pnl-badge{font-size:8px;font-weight:700;padding:3px 8px;border-radius:20px;text-transform:uppercase;letter-spacing:.5px;}
.bg-gold{background:var(--goldG2);color:var(--gold)}
.bg-grn{background:var(--grnG);color:var(--grn)}
.pnl-b{padding:14px 16px}
.chart{display:flex;align-items:flex-end;justify-content:center;gap:6px;height:90px;margin-bottom:14px;padding:0 6px;}
.cht-g{display:flex;flex-direction:column;align-items:center;gap:3px;flex:1;}
.cht-w{width:100%;height:72px;background:var(--bg2);border-radius:4px 4px 0 0;display:flex;align-items:flex-end;overflow:hidden;position:relative;}
.cht-b{width:100%;border-radius:4px 4px 0 0;transition:height .8s cubic-bezier(.34,1.56,.64,1);}
.cht-v{position:absolute;top:3px;left:50%;transform:translateX(-50%);font-size:8px;font-weight:800;color:var(--tx2);}
.cht-l{font-size:7px;color:var(--tx4);font-weight:700;text-transform:uppercase;letter-spacing:.2px;}
.met-grid{display:grid;grid-template-columns:1fr 1fr;gap:8px;}
.met{padding:12px;border-radius:var(--rs);background:var(--bg2);border:1px solid rgba(30,48,80,.5);}
.met-l{font-size:8px;color:var(--tx3);font-weight:700;text-transform:uppercase;letter-spacing:.6px;}
.met-r{display:flex;align-items:baseline;gap:3px;margin-top:4px;}
.met-v{font-size:20px;font-weight:900;color:var(--txt);}
.met-u{font-size:10px;color:var(--tx3);font-weight:600}
.met-s{font-size:9px;font-weight:600;margin-top:2px}
.met-s.pos{color:var(--grn)}
.met-s.neg{color:var(--red)}
.met-s.neu{color:var(--tx3)}
.sec-h{display:flex;align-items:center;justify-content:space-between;margin:20px 0 12px;padding:0 2px;}
.sec-l{display:flex;align-items:center;gap:6px}
.sec-tt{font-size:12px;font-weight:800;color:var(--gold);text-transform:uppercase;letter-spacing:1px;}
.sec-c{font-size:10px;color:var(--tx3);font-weight:600}
.mod{background:var(--sf);border:1px solid var(--bd);border-radius:var(--r);margin-bottom:8px;overflow:hidden;transition:border-color var(--tr);}
.mod.open{border-color:var(--bd2)}
.mod-h{padding:14px 16px;display:flex;align-items:center;justify-content:space-between;cursor:pointer;transition:background var(--tr);-webkit-tap-highlight-color:transparent;}
.mod-h:active{background:rgba(255,255,255,.02)}
.mod-left{display:flex;align-items:center;gap:12px;flex:1;min-width:0;}
.mod-icon{width:40px;height:40px;border-radius:10px;display:flex;align-items:center;justify-content:center;font-size:20px;flex-shrink:0;}
.mod-icon.agro{background:rgba(34,197,94,.08);box-shadow:inset 0 0 0 1px rgba(34,197,94,.15);}
.mod-icon.veg{background:rgba(16,185,129,.08);box-shadow:inset 0 0 0 1px rgba(16,185,129,.15);}
.mod-icon.agua{background:rgba(59,130,246,.08);box-shadow:inset 0 0 0 1px rgba(59,130,246,.15);}
.mod-icon.cult{background:rgba(245,158,11,.08);box-shadow:inset 0 0 0 1px rgba(245,158,11,.15);}
.mod-icon.anim{background:rgba(239,68,68,.08);box-shadow:inset 0 0 0 1px rgba(239,68,68,.15);}
.mod-icon.gen{background:rgba(139,92,246,.08);box-shadow:inset 0 0 0 1px rgba(139,92,246,.15);}
.mod-icon.tech{background:rgba(6,182,212,.08);box-shadow:inset 0 0 0 1px rgba(6,182,212,.15);}
.mod-icon.bonus{background:var(--goldG);box-shadow:inset 0 0 0 1px rgba(212,168,75,.15);}
.mod-det{flex:1;min-width:0}
.mod-nm{font-size:13px;font-weight:800;color:var(--txt);line-height:1.2;}
.mod-tutor{display:flex;align-items:center;gap:6px;margin-top:4px;}
.tutor-av{width:18px;height:18px;border-radius:50%;background:var(--sf3);display:flex;align-items:center;justify-content:center;font-size:7px;font-weight:800;color:var(--tx2);}
.tutor-nm{font-size:10px;color:var(--tx3);font-weight:600;}
.mod-right{display:flex;align-items:center;gap:10px;}
.mod-prog{font-size:10px;font-weight:700;color:var(--tx3);}
.mod-prog .done{color:var(--grn)}
.mod-chev{font-size:14px;color:var(--tx4);transition:transform .3s ease;line-height:1;}
.mod.open .mod-chev{transform:rotate(90deg)}
.mod-body{max-height:0;overflow:hidden;transition:max-height .4s ease;}
.mod.open .mod-body{max-height:600px}
.mod-inner{padding:0 12px 12px;border-top:1px solid var(--bd);}
.tema{display:flex;align-items:center;gap:12px;padding:12px;border-radius:var(--rs);cursor:pointer;margin-top:6px;transition:background var(--tr);-webkit-tap-highlight-color:transparent;}
.tema:active{background:rgba(255,255,255,.03)}
.tema-ic{width:36px;height:36px;border-radius:8px;background:var(--bg2);display:flex;align-items:center;justify-content:center;font-size:18px;flex-shrink:0;}
.tema-det{flex:1;min-width:0}
.tema-tag{font-size:8px;font-weight:800;text-transform:uppercase;letter-spacing:.8px;}
.tema-tag.agro{color:var(--grn)}
.tema-tag.veg{color:#10B981}
.tema-tag.agua{color:var(--blu)}
.tema-tag.cult{color:var(--org)}
.tema-tag.anim{color:var(--red)}
.tema-tag.gen{color:var(--vio)}
.tema-tag.tech{color:#06B6D4}
.tema-tag.bonus{color:var(--gold)}
.tema-nm{font-size:13px;font-weight:700;color:var(--txt);line-height:1.3;margin-top:1px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;}
.tema-ct{font-size:9px;color:var(--tx4);font-weight:500;margin-top:2px;}
.tema-st{flex-shrink:0}
.badge-sc{font-size:10px;font-weight:800;padding:3px 8px;border-radius:6px;}
.badge-sc.done{background:var(--grnG);color:var(--grn)}
.badge-sc.wait{background:rgba(100,116,139,.1);color:var(--tx3)}
.badge-sc.lock{background:var(--goldG);color:var(--gold3)}
.tema-arr{color:var(--tx4);font-size:14px;flex-shrink:0;transition:transform .15s ease;}
.tema:active .tema-arr{transform:translateX(2px)}
.bnav{position:fixed;bottom:0;left:0;right:0;max-width:480px;margin:0 auto;background:var(--bg2);border-top:1px solid var(--bd);display:flex;justify-content:space-around;padding:6px 0 max(10px,env(safe-area-inset-bottom));z-index:100;}
.nav-i{display:flex;flex-direction:column;align-items:center;gap:2px;padding:6px 14px;border-radius:10px;cursor:pointer;transition:all .15s ease;position:relative;-webkit-tap-highlight-color:transparent;}
.nav-i.act{color:var(--gold)}
.nav-i.act::before{content:'';position:absolute;top:-6px;width:16px;height:2px;border-radius:2px;background:var(--gold);}
.nav-i:not(.act){color:var(--tx4)}
.nav-ic{font-size:18px;line-height:1}
.nav-lb{font-size:8px;font-weight:700;text-transform:uppercase;letter-spacing:.3px;}
.nav-dot{position:absolute;top:3px;right:6px;width:14px;height:14px;border-radius:50%;background:var(--red);color:#fff;font-size:7px;font-weight:800;display:flex;align-items:center;justify-content:center;}
.toast{position:fixed;top:80px;left:50%;transform:translateX(-50%) translateY(-16px);background:var(--sf2);border:1px solid var(--bd2);color:var(--gold);padding:10px 22px;border-radius:10px;font-size:12px;font-weight:700;opacity:0;transition:all .3s ease;z-index:200;pointer-events:none;box-shadow:var(--sh2);}
.toast.show{opacity:1;transform:translateX(-50%) translateY(0);}
.st,.pnl,.mod{animation:fadeUp .5s ease both;}
@keyframes fadeUp{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
::-webkit-scrollbar{width:3px}
::-webkit-scrollbar-track{background:transparent}
::-webkit-scrollbar-thumb{background:var(--bd);border-radius:10px;}
.loading{text-align:center;padding:40px;color:var(--gold);font-size:14px;}
</style>
</head>
<body>

<div class="hdr">
  <div class="hdr-top">
    <div class="brand">
      <div class="brand-mark"><span>K</span></div>
      <div class="brand-txt">
        <span class="brand-name">KRONA</span>
        <span class="brand-sub">Analytics</span>
      </div>
    </div>
    <div class="usr">
      <div class="usr-info">
        <div class="usr-lbl">Aluno</div>
        <div class="usr-nm" id="uName">Carregando...</div>
        <div class="usr-xp-row">
          <span class="usr-xp" id="uXP">0 XP</span>
          <span class="lvl-pill" id="uLvl">Lv.1</span>
        </div>
      </div>
      <div class="usr-av" id="uAv">?</div>
    </div>
  </div>
  <div class="prog-strip">
    <div class="prog-row">
      <span class="prog-t">Progresso Geral</span>
      <span class="prog-p" id="pPct">0%</span>
    </div>
    <div class="prog-track">
      <div class="prog-fill" id="pFill" style="width:0%"></div>
    </div>
    <div class="prog-info" id="pInfo">0 de 15 módulos concluídos</div>
  </div>
</div>

<div class="ctn">
  <div class="stats">
    <div class="st">
      <div class="st-ic">✅</div>
      <div class="st-v" id="sHits">0</div>
      <div class="st-l">Acertos</div>
    </div>
    <div class="st">
      <div class="st-ic">📝</div>
      <div class="st-v" id="sTot">0</div>
      <div class="st-l">Questões</div>
    </div>
    <div class="st">
      <div class="st-ic">🎯</div>
      <div class="st-v" id="sPct">0%</div>
      <div class="st-l">Aproveit.</div>
    </div>
  </div>

  <div class="pnl">
    <div class="pnl-h">
      <div class="pnl-tg">
        <span class="pnl-ic">📊</span>
        <span class="pnl-tt">Desempenho por Área</span>
      </div>
      <span class="pnl-badge bg-gold" id="nivelBadge">Iniciante</span>
    </div>
    <div class="pnl-b">
      <div class="chart" id="chartContainer">
        <div class="loading">Carregando gráficos...</div>
      </div>
      <div class="met-grid">
        <div class="met">
          <div class="met-l">Melhor Nota</div>
          <div class="met-r">
            <span class="met-v" id="bestGrade">0</span>
            <span class="met-u">/10</span>
          </div>
          <div class="met-s pos" id="bestTema">-</div>
        </div>
        <div class="met">
          <div class="met-l">Temas</div>
          <div class="met-r">
            <span class="met-v" id="completedThemes">0</span>
            <span class="met-u">/15</span>
          </div>
          <div class="met-s pos" id="themesPercent">0%</div>
        </div>
        <div class="met">
          <div class="met-l">Erros</div>
          <div class="met-r">
            <span class="met-v" id="errorsCount">0</span>
            <span class="met-u">itens</span>
          </div>
          <div class="met-s neg" id="errorsMsg">Revisar ⚠</div>
        </div>
        <div class="met">
          <div class="met-l">XP Total</div>
          <div class="met-r">
            <span class="met-v" id="xpTotal">0</span>
            <span class="met-u">XP</span>
          </div>
          <div class="met-s neu" id="nextLevel">Próximo nível</div>
        </div>
      </div>
    </div>
  </div>

  <div class="sec-h">
    <div class="sec-l">
      <span class="sec-tt">◆ Módulos do Simulado</span>
    </div>
    <span class="sec-c" id="temasCount">0/15 temas</span>
  </div>

  <div id="modulosContainer">
    <div class="loading">Carregando módulos...</div>
  </div>
</div>

<div class="bnav">
  <div class="nav-i act" id="nHome">
    <span class="nav-ic">🏠</span>
    <span class="nav-lb">Início</span>
  </div>
  <div class="nav-i" id="nRank">
    <span class="nav-ic">🏆</span>
    <span class="nav-lb">Ranking</span>
  </div>
  <div class="nav-i" id="nErr">
    <span class="nav-ic">📓</span>
    <span class="nav-lb">Erros</span>
    <span class="nav-dot" id="errorDot" style="display:none">0</span>
  </div>
  <div class="nav-i" id="nRefresh">
    <span class="nav-ic">🔄</span>
    <span class="nav-lb">Atualizar</span>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
// ================== CONFIGURAÇÕES ==================
const tg = window.Telegram?.WebApp;
let userData = null;
let temasStats = [];

// === API Base URL ===
const API_BASE = "https://krona-api.onrender.com";

// ================== INICIALIZAÇÃO ==================
if (tg) {
  tg.ready();
  tg.expand();
  try { tg.setHeaderColor('#0B1120'); } catch(e) {}
  try { tg.setBackgroundColor('#060A13'); } catch(e) {}
}

// ================== FUNÇÕES AUXILIARES ==================
function showToast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2200);
}

function getUserId() {
  if (tg && tg.initDataUnsafe?.user) {
    return tg.initDataUnsafe.user.id;
  }
  return localStorage.getItem('test_user_id') || 12345;
}

function getUserName() {
  if (tg && tg.initDataUnsafe?.user) {
    return tg.initDataUnsafe.user.first_name || 'Aluno';
  }
  return 'Teste Local';
}

// ================== CARREGAR DADOS DO USUÁRIO ==================
async function carregarDadosUsuario() {
  const userId = getUserId();
  const userName = getUserName();
  
  document.getElementById('uName').textContent = userName;
  const initials = userName.substring(0, 2).toUpperCase();
  document.getElementById('uAv').textContent = initials;
  
  try {
    const response = await fetch(`${API_BASE}/api/user/${userId}`);
    if (!response.ok) throw new Error('Erro ao carregar dados');
    
    userData = await response.json();
    
    document.getElementById('uXP').textContent = `${userData.xp || 0} XP`;
    document.getElementById('uLvl').textContent = `Lv.${userData.level || 1}`;
    document.getElementById('sHits').textContent = userData.acertos || 0;
    document.getElementById('sTot').textContent = userData.total_questoes || 0;
    document.getElementById('sPct').textContent = `${userData.percentual_geral || 0}%`;
    
    const pct = userData.percentual_geral || 0;
    document.getElementById('pFill').style.width = `${pct}%`;
    document.getElementById('pPct').textContent = `${pct}%`;
    document.getElementById('pInfo').textContent = `${userData.temas_completos || 0} de 15 módulos concluídos`;
    document.getElementById('completedThemes').textContent = `${userData.temas_completos || 0}/15`;
    document.getElementById('themesPercent').textContent = `${Math.round(((userData.temas_completos || 0)/15)*100)}%`;
    document.getElementById('xpTotal').textContent = userData.xp || 0;
    
    const currentLevel = userData.level || 1;
    const nextLevelXP = currentLevel * 200;
    const xpToNext = nextLevelXP - (userData.xp || 0);
    document.getElementById('nextLevel').textContent = `${xpToNext > 0 ? xpToNext : 0} XP para Lv.${currentLevel + 1}`;
    
    const nivelBadge = document.getElementById('nivelBadge');
    if (userData.percentual_geral >= 90) {
      nivelBadge.textContent = '🏆 Mestre';
      nivelBadge.className = 'pnl-badge bg-gold';
    } else if (userData.percentual_geral >= 70) {
      nivelBadge.textContent = '🥇 Avançado';
      nivelBadge.className = 'pnl-badge bg-gold';
    } else if (userData.percentual_geral >= 50) {
      nivelBadge.textContent = '📚 Em Progresso';
      nivelBadge.className = 'pnl-badge bg-grn';
    } else {
      nivelBadge.textContent = '🌱 Iniciante';
      nivelBadge.className = 'pnl-badge bg-grn';
    }
    
    // Calcular melhor nota
    let bestNota = 0;
    let bestTemaNome = '-';
    if (userData.temas_stats) {
      userData.temas_stats.forEach(t => {
        if (t.nota && t.nota > bestNota) {
          bestNota = t.nota;
          bestTemaNome = t.nome;
        }
      });
    }
    document.getElementById('bestGrade').textContent = bestNota.toFixed(1);
    document.getElementById('bestTema').textContent = bestTemaNome;
    
    await carregarErros(userId);
    
    if (userData.temas_stats) {
      temasStats = userData.temas_stats;
    } else {
      criarTemasStatsMock();
    }
    
    renderModulos();
    renderChart();
    
  } catch (error) {
    console.error('Erro ao carregar dados:', error);
    showToast('❌ Erro ao conectar com o servidor');
    usarDadosMock();
  }
}

async function carregarErros(userId) {
  try {
    const response = await fetch(`${API_BASE}/api/errors/${userId}?limit=10`);
    if (response.ok) {
      const data = await response.json();
      const errorCount = data.total || data.errors?.length || 0;
      document.getElementById('errorsCount').textContent = errorCount;
      const errorDot = document.getElementById('errorDot');
      if (errorCount > 0) {
        errorDot.style.display = 'flex';
        errorDot.textContent = errorCount > 99 ? '99+' : errorCount;
        document.getElementById('errorsMsg').textContent = `${errorCount} erros para revisar`;
      } else {
        errorDot.style.display = 'none';
        document.getElementById('errorsMsg').textContent = 'Nenhum erro! 🎉';
      }
    }
  } catch (error) {
    console.error('Erro ao carregar erros:', error);
  }
}

function criarTemasStatsMock() {
  const temasNomes = [
    "🌾 Contextualização do Agronegócio", "🌱 Sistemas de Produção Sustentável",
    "🌿 Fundamentos de Botânica", "🧪 Nutrição Mineral e Fertilidade",
    "💧 Irrigação e Drenagem", "🌽 Principais Culturas",
    "🐄 Produção Animal", "🔬 Anatomia e Fisiologia Animal",
    "🍽️ Alimentação e Nutrição Animal", "🌾 Forragicultura e Pastagens",
    "🧬 Melhoramento Genético Animal", "🐔 Produção de Monogástricos",
    "🐮 Produção de Ruminantes", "📡 Agricultura de Precisão",
    "🎁 Simulado Bônus"
  ];
  temasStats = [];
  for (let i = 0; i < 15; i++) {
    temasStats.push({
      id: `TEMA_${i+1}`,
      nome: temasNomes[i],
      completado: false,
      nota: null,
      acertos: 0,
      total: 15
    });
  }
}

function usarDadosMock() {
  document.getElementById('uXP').textContent = '0 XP';
  document.getElementById('uLvl').textContent = 'Lv.1';
  document.getElementById('sHits').textContent = '0';
  document.getElementById('sTot').textContent = '0';
  document.getElementById('sPct').textContent = '0%';
  document.getElementById('pFill').style.width = '0%';
  document.getElementById('pPct').textContent = '0%';
  document.getElementById('pInfo').textContent = '0 de 15 módulos concluídos';
  document.getElementById('completedThemes').textContent = '0/15';
  document.getElementById('themesPercent').textContent = '0%';
  document.getElementById('xpTotal').textContent = '0';
  document.getElementById('nextLevel').textContent = '200 XP para Lv.2';
  document.getElementById('bestGrade').textContent = '0';
  document.getElementById('bestTema').textContent = '-';
  document.getElementById('errorsCount').textContent = '0';
  document.getElementById('errorsMsg').textContent = 'Nenhum erro! 🎉';
  criarTemasStatsMock();
  renderModulos();
  renderChart();
}

function renderModulos() {
  if (!temasStats || temasStats.length === 0) return;
  
  const container = document.getElementById('modulosContainer');
  if (!container) return;
  
  document.getElementById('temasCount').textContent = `${temasStats.filter(t => t.completado).length}/15 temas`;
  
  const modulos = [
    { id: 'A', nome: 'Agronegócio & Sustentabilidade', icon: '🌾', iconClass: 'agro', temas: [0, 1] },
    { id: 'B', nome: 'Ciências Vegetais & Solo', icon: '🌿', iconClass: 'veg', temas: [2, 3] },
    { id: 'C', nome: 'Recursos Hídricos', icon: '💧', iconClass: 'agua', temas: [4] },
    { id: 'D', nome: 'Culturas Agrícolas', icon: '🌽', iconClass: 'cult', temas: [5] },
    { id: 'E', nome: 'Produção Animal I', icon: '🐄', iconClass: 'anim', temas: [6, 7, 8] },
    { id: 'F', nome: 'Forragem & Genética', icon: '🌾', iconClass: 'gen', temas: [9, 10] },
    { id: 'G', nome: 'Produção Animal II', icon: '🐔', iconClass: 'anim', temas: [11, 12] },
    { id: 'H', nome: 'Tecnologia & Precisão', icon: '📡', iconClass: 'tech', temas: [13] },
    { id: 'X', nome: 'Simulado Bônus', icon: '🎁', iconClass: 'bonus', temas: [14] }
  ];
  
  let html = '';
  let modIndex = 0;
  
  for (const mod of modulos) {
    const temasMod = mod.temas.map(idx => temasStats[idx]).filter(t => t);
    const completosMod = temasMod.filter(t => t && t.completado).length;
    const totalMod = temasMod.length;
    const isOpen = modIndex === 0 ? 'open' : '';
    
    html += `
      <div class="mod ${isOpen}" id="mod${mod.id}">
        <div class="mod-h" onclick="toggleModulo('${mod.id}')">
          <div class="mod-left">
            <div class="mod-icon ${mod.iconClass}">${mod.icon}</div>
            <div class="mod-det">
              <div class="mod-nm">${mod.nome}</div>
              <div class="mod-tutor">
                <div class="tutor-av">KR</div>
                <span class="tutor-nm">Equipe KRONA Analytics</span>
              </div>
            </div>
          </div>
          <div class="mod-right">
            <div class="mod-prog"><span class="done">${completosMod}</span>/${totalMod}</div>
            <div class="mod-chev">▸</div>
          </div>
        </div>
        <div class="mod-body">
          <div class="mod-inner">
    `;
    
    for (const tema of temasMod) {
      if (!tema) continue;
      const temaNum = tema.id.split('_')[1];
      const completedClass = tema.completado ? 'done' : 'wait';
      const completedText = tema.completado ? (tema.nota ? tema.nota.toFixed(1) : 'Concluído') : 'Iniciar';
      const tagClass = mod.iconClass;
      
      html += `
        <div class="tema" onclick="startQuiz('${tema.id}', '${tema.nome}')">
          <div class="tema-ic">${mod.icon}</div>
          <div class="tema-det">
            <div class="tema-tag ${tagClass}">Tema ${temaNum}</div>
            <div class="tema-nm">${tema.nome}</div>
            <div class="tema-ct">${tema.total || 15} questões</div>
          </div>
          <div class="tema-st">
            <span class="badge-sc ${completedClass}">${completedText}</span>
          </div>
          <span class="tema-arr">›</span>
        </div>
      `;
    }
    
    html += `
          </div>
        </div>
      </div>
    `;
    modIndex++;
  }
  
  container.innerHTML = html;
}

function renderChart() {
  const chartContainer = document.getElementById('chartContainer');
  if (!chartContainer) return;
  
  if (!temasStats || temasStats.length === 0) {
    chartContainer.innerHTML = '<div class="loading">Carregando gráficos...</div>';
    return;
  }
  
  const temasParaGrafico = temasStats.slice(0, 6);
  const cores = ['#22C55E', '#10B981', '#3B82F6', '#F59E0B', '#EF4444', '#06B6D4'];
  
  let html = '';
  for (let i = 0; i < temasParaGrafico.length; i++) {
    const tema = temasParaGrafico[i];
    const percentual = tema.total > 0 ? Math.round((tema.acertos / tema.total) * 100) : 0;
    const nomeAbreviado = tema.nome.substring(0, 8);
    
    html += `
      <div class="cht-g">
        <div class="cht-w">
          <div class="cht-v">${percentual}%</div>
          <div class="cht-b" style="height:${percentual}%; background:linear-gradient(180deg, ${cores[i]}, ${cores[i]}aa)"></div>
        </div>
        <span class="cht-l">${nomeAbreviado}</span>
      </div>
    `;
  }
  
  chartContainer.innerHTML = html;
}

function toggleModulo(modId) {
  const el = document.getElementById('mod' + modId);
  if (!el) return;
  const wasOpen = el.classList.contains('open');
  
  document.querySelectorAll('.mod').forEach(m => {
    m.classList.remove('open');
  });
  
  if (!wasOpen) {
    el.classList.add('open');
    setTimeout(() => {
      el.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }, 100);
  }
  
  if (tg) tg.HapticFeedback?.impactOccurred('light');
}

function startQuiz(temaId, temaNome) {
  if (tg && tg.showConfirm) {
    tg.showConfirm(`Iniciar simulado:\n${temaNome}?`, function(ok) {
      if (ok) enviarDados(temaId);
    });
  } else {
    if (confirm(`Iniciar simulado: ${temaNome}?`)) {
      enviarDados(temaId);
    }
  }
}

function enviarDados(temaId) {
  showToast('◆ Iniciando simulado...');
  
  const payload = JSON.stringify({
    action: 'start_quiz',
    tema_id: temaId
  });
  
  if (tg && tg.sendData) {
    setTimeout(() => {
      tg.sendData(payload);
      tg.close();
    }, 600);
  } else {
    console.log('sendData:', payload);
    showToast(`Tema: ${temaId} (demo)`);
  }
}

async function mostrarRanking() {
  showToast('🏆 Carregando ranking...');
  try {
    const response = await fetch(`${API_BASE}/api/ranking`);
    if (!response.ok) throw new Error('Erro ao carregar ranking');
    const data = await response.json();
    
    let txt = '🏆 *RANKING DA TURMA* 🏆\n━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n';
    const medals = ['🥇', '🥈', '🥉'];
    
    if (!data.ranking || data.ranking.length === 0) {
      txt += 'Sem dados ainda. Complete um tema!';
    } else {
      for (let i = 0; i < Math.min(data.ranking.length, 10); i++) {
        const r = data.ranking[i];
        const medal = medals[i] || `${i+1}.`;
        const pct = r.total_questoes > 0 ? Math.round((r.total_acertos / r.total_questoes) * 100) : 0;
        txt += `${medal} *${r.user_name}*\n   ${pct}% · ${r.xp} XP · Lv.${r.level}\n\n`;
      }
    }
    
    if (tg && tg.showPopup) {
      tg.showPopup({ title: 'Ranking', message: txt, buttons: [{ type: 'ok' }] });
    } else {
      alert(txt);
    }
  } catch (error) {
    console.error('Erro ao carregar ranking:', error);
    showToast('❌ Erro ao carregar ranking');
  }
}

async function mostrarErros() {
  showToast('📓 Carregando erros...');
  const userId = getUserId();
  
  try {
    const response = await fetch(`${API_BASE}/api/errors/${userId}?limit=10`);
    if (!response.ok) throw new Error('Erro ao carregar erros');
    const data = await response.json();
    
    if (!data.errors || data.errors.length === 0) {
      if (tg && tg.showPopup) {
        tg.showPopup({ title: 'Caderno de Erros', message: '✅ Parabéns! Você não tem erros registrados!', buttons: [{ type: 'ok' }] });
      } else {
        alert('✅ Nenhum erro registrado!');
      }
      return;
    }
    
    let txt = `📓 *CADERNO DE ERROS* (${data.total || data.errors.length} registros)\n━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n`;
    for (let i = 0; i < Math.min(data.errors.length, 5); i++) {
      const e = data.errors[i];
      const temaNome = e.tema_nome || e.tema_id;
      txt += `${i+1}. *${temaNome}*\n❓ ${e.pergunta.substring(0, 80)}...\n✅ ${e.correta}\n\n`;
    }
    
    if (tg && tg.showPopup) {
      tg.showPopup({ title: 'Meus Erros', message: txt, buttons: [{ type: 'ok' }] });
    } else {
      alert(txt);
    }
  } catch (error) {
    console.error('Erro ao carregar erros:', error);
    showToast('❌ Erro ao carregar erros');
  }
}

function initNav() {
  document.getElementById('nHome').addEventListener('click', () => {
    document.querySelectorAll('.nav-i').forEach(n => n.classList.remove('act'));
    document.getElementById('nHome').classList.add('act');
    window.scrollTo({ top: 0, behavior: 'smooth' });
    if (tg) tg.HapticFeedback?.selectionChanged();
  });
  
  document.getElementById('nRank').addEventListener('click', () => {
    document.querySelectorAll('.nav-i').forEach(n => n.classList.remove('act'));
    document.getElementById('nRank').classList.add('act');
    mostrarRanking();
    if (tg) tg.HapticFeedback?.selectionChanged();
  });
  
  document.getElementById('nErr').addEventListener('click', () => {
    document.querySelectorAll('.nav-i').forEach(n => n.classList.remove('act'));
    document.getElementById('nErr').classList.add('act');
    mostrarErros();
    if (tg) tg.HapticFeedback?.selectionChanged();
  });
  
  document.getElementById('nRefresh').addEventListener('click', () => {
    location.reload();
  });
}

window.addEventListener('load', () => {
  initNav();
  carregarDadosUsuario();
});

window.toggleModulo = toggleModulo;
window.startQuiz = startQuiz;
</script>
</body>
</html>
