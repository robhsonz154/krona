#!/usr/bin/env python3
"""
KRONA Analytics — Bot Backend para Telegram Mini App
Versão COMPLETA com todas as funções sincronizadas
"""

import os
import json
import time
import sqlite3
import logging
from datetime import datetime
from flask import Flask, request, jsonify
from flask_cors import CORS
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, WebAppInfo
from telegram.ext import Application, CommandHandler, MessageHandler, CallbackQueryHandler, ContextTypes, filters

# ================== CONFIGURAÇÕES ==================
TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN")
WEBAPP_URL = os.environ.get("WEBAPP_URL", "https://robhsonz154.github.io/krona/")
PORT = int(os.environ.get("PORT", 8080))

# ================== LOGGING ==================
logging.basicConfig(format="%(asctime)s [%(levelname)s] %(message)s", level=logging.INFO)
log = logging.getLogger(__name__)

# ================== IMPORT QUESTÕES ==================
try:
    from questions_bank import QUESTIONS_DB
    log.info(f"📚 Banco carregado: {sum(len(v) for v in QUESTIONS_DB.values())} questões em {len(QUESTIONS_DB)} temas")
except ImportError:
    log.error("❌ questions_bank.py não encontrado!")
    QUESTIONS_DB = {}

# ================== NOMES DOS TEMAS ==================
TEMAS = {
    "TEMA_1": "🌾 Contextualização do Agronegócio",
    "TEMA_2": "🌱 Sistemas de Produção Sustentável",
    "TEMA_3": "🌿 Fundamentos de Botânica",
    "TEMA_4": "🧪 Nutrição Mineral e Fertilidade",
    "TEMA_5": "💧 Irrigação e Drenagem",
    "TEMA_6": "🌽 Principais Culturas",
    "TEMA_7": "🐄 Produção Animal",
    "TEMA_8": "🔬 Anatomia e Fisiologia Animal",
    "TEMA_9": "🍽️ Alimentação e Nutrição Animal",
    "TEMA_10": "🌾 Forragicultura e Pastagens",
    "TEMA_11": "🧬 Melhoramento Genético Animal",
    "TEMA_12": "🐔 Produção de Monogástricos",
    "TEMA_13": "🐮 Produção de Ruminantes",
    "TEMA_14": "📡 Agricultura de Precisão",
    "TEMA_15": "🎁 Simulado Bônus",
}

# ================== BANCO DE DADOS ==================
DB_FILE = "krona.db"

def get_db():
    conn = sqlite3.connect(DB_FILE)
    conn.row_factory = sqlite3.Row
    return conn

def init_db():
    try:
        conn = get_db()
        conn.executescript("""
        CREATE TABLE IF NOT EXISTS krona_users (
            user_id INTEGER PRIMARY KEY,
            user_name TEXT DEFAULT 'Aluno',
            xp INTEGER DEFAULT 0,
            level INTEGER DEFAULT 1,
            total_acertos INTEGER DEFAULT 0,
            total_questoes INTEGER DEFAULT 0,
            temas_completos TEXT DEFAULT '',
            lembrete INTEGER DEFAULT 0,
            chat_id INTEGER,
            criado_em TEXT DEFAULT (datetime('now')),
            atualizado_em TEXT DEFAULT (datetime('now'))
        );
        CREATE TABLE IF NOT EXISTS krona_results (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            tema_id TEXT NOT NULL,
            acertos INTEGER NOT NULL,
            total INTEGER NOT NULL,
            nota REAL NOT NULL,
            xp_ganho INTEGER DEFAULT 0,
            criado_em TEXT DEFAULT (datetime('now'))
        );
        CREATE TABLE IF NOT EXISTS krona_errors (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            tema_id TEXT NOT NULL,
            pergunta TEXT NOT NULL,
            correta TEXT NOT NULL,
            criado_em TEXT DEFAULT (datetime('now'))
        );
        """)
        conn.commit()
        conn.close()
        log.info("💾 SQLite inicializado")
        return True
    except Exception as e:
        log.error(f"Erro init_db: {e}")
        return False

def db_user(uid, name="Aluno"):
    try:
        conn = get_db()
        row = conn.execute("SELECT * FROM krona_users WHERE user_id=?", (uid,)).fetchone()
        if not row:
            conn.execute("INSERT INTO krona_users (user_id, user_name) VALUES (?,?)", (uid, name))
            conn.commit()
            row = conn.execute("SELECT * FROM krona_users WHERE user_id=?", (uid,)).fetchone()
        conn.close()
        return dict(row)
    except Exception as e:
        log.error(f"Erro db_user: {e}")
        return {
            "user_id": uid,
            "user_name": name,
            "xp": 0,
            "level": 1,
            "total_acertos": 0,
            "total_questoes": 0,
            "temas_completos": "",
            "lembrete": 0,
            "chat_id": None
        }

def db_save_result(uid, tema, ac, tot, xp_g):
    try:
        conn = get_db()
        nota = round((ac / max(tot, 1)) * 10, 1)
        conn.execute("INSERT INTO krona_results (user_id, tema_id, acertos, total, nota, xp_ganho) VALUES (?,?,?,?,?,?)", (uid, tema, ac, tot, nota, xp_g))
        u = conn.execute("SELECT * FROM krona_users WHERE user_id=?", (uid,)).fetchone()
        temas = [t for t in u["temas_completos"].split(",") if t]
        if tema not in temas:
            temas.append(tema)
        new_xp = u["xp"] + xp_g
        new_lvl = new_xp // 200 + 1
        conn.execute("UPDATE krona_users SET xp=?, level=?, total_acertos=total_acertos+?, total_questoes=total_questoes+?, temas_completos=?, atualizado_em=datetime('now') WHERE user_id=?", (new_xp, new_lvl, ac, tot, ",".join(temas), uid))
        conn.commit()
        conn.close()
        return nota, new_xp, new_lvl
    except Exception as e:
        log.error(f"Erro db_save_result: {e}")
        return 0, 0, 1

def db_save_error(uid, tema, perg, corr):
    try:
        conn = get_db()
        conn.execute("INSERT INTO krona_errors (user_id, tema_id, pergunta, correta) VALUES (?,?,?,?)", (uid, tema, perg[:200], corr))
        conn.commit()
        conn.close()
    except Exception as e:
        log.error(f"Erro db_save_error: {e}")

def db_ranking(limit=15):
    try:
        conn = get_db()
        rows = conn.execute("""
            SELECT user_id, user_name, xp, level, total_acertos, total_questoes, temas_completos 
            FROM krona_users 
            WHERE total_questoes > 0 
            ORDER BY xp DESC 
            LIMIT ?
        """, (limit,)).fetchall()
        conn.close()
        return [dict(r) for r in rows]
    except Exception as e:
        log.error(f"Erro db_ranking: {e}")
        return []

def db_errors(uid, limit=20):
    try:
        conn = get_db()
        rows = conn.execute("SELECT tema_id, pergunta, correta, criado_em FROM krona_errors WHERE user_id=? ORDER BY criado_em DESC LIMIT ?", (uid, limit)).fetchall()
        conn.close()
        return [dict(r) for r in rows]
    except Exception as e:
        log.error(f"Erro db_errors: {e}")
        return []

def db_user_results(uid):
    """Retorna todos os resultados do usuário por tema"""
    try:
        conn = get_db()
        rows = conn.execute("""
            SELECT tema_id, acertos, total, nota, criado_em 
            FROM krona_results 
            WHERE user_id=? 
            ORDER BY criado_em DESC
        """, (uid,)).fetchall()
        conn.close()
        return [dict(r) for r in rows]
    except Exception as e:
        log.error(f"Erro db_user_results: {e}")
        return []

def db_user_stats_by_tema(uid):
    """Retorna estatísticas por tema para o gráfico"""
    try:
        conn = get_db()
        rows = conn.execute("""
            SELECT tema_id, 
                   SUM(acertos) as total_acertos, 
                   SUM(total) as total_questoes,
                   AVG(nota) as media_nota
            FROM krona_results 
            WHERE user_id=? 
            GROUP BY tema_id
        """, (uid,)).fetchall()
        conn.close()
        return [dict(r) for r in rows]
    except Exception as e:
        log.error(f"Erro db_user_stats_by_tema: {e}")
        return []

# ================== FUNÇÕES AUXILIARES ==================
def barra(atual, total):
    n = int((atual / max(total, 1)) * 10)
    return "▓" * n + "░" * (10 - n)

def calc_xp(acertos, total):
    return acertos * 15 + (50 if acertos == total else 0)

# ================== SESSÕES ==================
sessions = {}

# ================== HANDLERS ==================
async def cmd_start(update: Update, ctx):
    u = update.effective_user
    db_user(u.id, u.first_name or "Aluno")
    kb = InlineKeyboardMarkup([
        [InlineKeyboardButton("◆ Abrir KRONA Analytics", web_app=WebAppInfo(url=WEBAPP_URL))],
        [InlineKeyboardButton("🏆 Ranking", callback_data="cb_ranking"), 
         InlineKeyboardButton("📊 Progresso", callback_data="cb_progresso")],
        [InlineKeyboardButton("📓 Meus Erros", callback_data="cb_erros"), 
         InlineKeyboardButton("🔔 Lembrete", callback_data="cb_lembrete")]
    ])
    await update.message.reply_text(
        f"◆ *KRONA Analytics*\n━━━━━━━━━━━━━━━━━━━━━━━━\n\n"
        f"Olá, *{u.first_name}*! 👋\n\n"
        f"Plataforma de simulados preparatórios para Técnico em Agronegócio — SENAR\n\n"
        f"📚 15 temas · 230+ questões\n"
        f"🎯 Estilo ENEM · Timer 5 min\n"
        f"🏆 Ranking · XP · Níveis\n\n"
        f"━━━━━━━━━━━━━━━━━━━━━━━━\n"
        f"Toque no botão abaixo para começar:",
        reply_markup=kb,
        parse_mode="Markdown"
    )

async def handle_webapp(update: Update, ctx):
    data = update.effective_message.web_app_data.data
    uid = update.effective_user.id
    try:
        payload = json.loads(data)
        if payload.get("action") == "start_quiz" and payload.get("tema_id"):
            await iniciar_quiz(update, ctx, payload["tema_id"])
    except:
        pass

async def iniciar_quiz(update, ctx, tema_id):
    uid = update.effective_user.id
    if tema_id not in QUESTIONS_DB:
        await update.effective_message.reply_text(f"❌ Tema não encontrado.")
        return
    sessions[uid] = {
        "tema": tema_id,
        "idx": 0,
        "score": 0,
        "total": len(QUESTIONS_DB[tema_id]),
        "questoes": QUESTIONS_DB[tema_id].copy(),
        "timer": time.time()
    }
    await enviar_questao(update.effective_message, uid, tema_id, 0)

async def enviar_questao(msg, uid, tema_id, idx):
    s = sessions.get(uid)
    if not s or idx >= len(s["questoes"]):
        await finalizar(msg, uid, tema_id)
        return
    q = s["questoes"][idx]
    texto = f"◆ *Questão {idx+1}/{s['total']}*\n\n{q['pergunta']}\n\n"
    for letra, opcao in q["opcoes"].items():
        texto += f"\n*{letra}.* {opcao}"
    texto += f"\n\n━━━━━━━━━━━━━━━━━━━━━━━━\n⏱️ *5 minutos para responder*"
    kb = InlineKeyboardMarkup([
        [InlineKeyboardButton(l, callback_data=f"R_{tema_id}_{idx}_{l}") for l in ["A","B","C","D","E"]],
        [InlineKeyboardButton("❌ Cancelar", callback_data="cb_cancelar")]
    ])
    await msg.reply_text(texto, reply_markup=kb, parse_mode="Markdown")

async def handle_cb(update: Update, ctx):
    q = update.callback_query
    await q.answer()
    uid = q.from_user.id
    data = q.data
    
    if data.startswith("R_"):
        parts = data.split("_")
        tema = f"{parts[1]}_{parts[2]}"
        idx = int(parts[3])
        resp = parts[4]
        s = sessions.get(uid)
        if s and idx < len(s["questoes"]):
            q_atual = s["questoes"][idx]
            if resp == q_atual["correta"]:
                s["score"] += 1
                txt = f"✅ *ACERTOU!* 🎉\n\n💡 *Explicação:*\n{q_atual['explicacao']}\n\n━━━━━━━━━━━━━━━━━━━━━\n📊 Placar: *{s['score']}/{idx+1}*"
            else:
                db_save_error(uid, tema, q_atual["pergunta"], f"{q_atual['correta']}) {q_atual['opcoes'][q_atual['correta']]}")
                txt = f"❌ *ERROU!*\n\n✅ *Correta:* {q_atual['correta']}) {q_atual['opcoes'][q_atual['correta']]}\n\n💡 *Explicação:*\n{q_atual['explicacao']}\n\n━━━━━━━━━━━━━━━━━━━━━\n📓 _Salvo no caderno de erros_"
            s["idx"] = idx + 1
            if idx + 1 >= s["total"]:
                kb = InlineKeyboardMarkup([[InlineKeyboardButton("📊 VER RESULTADO", callback_data=f"N_{tema}_{idx+1}")]])
            else:
                kb = InlineKeyboardMarkup([[InlineKeyboardButton("➡️ PRÓXIMA", callback_data=f"N_{tema}_{idx+1}")]])
            await q.edit_message_text(txt, reply_markup=kb, parse_mode="Markdown")
            
    elif data.startswith("N_"):
        parts = data.split("_")
        tema = f"{parts[1]}_{parts[2]}"
        idx = int(parts[3])
        await enviar_questao(q.message, uid, tema, idx)
        
    elif data == "cb_cancelar":
        if uid in sessions:
            del sessions[uid]
        await q.edit_message_text("❌ Simulado cancelado.\n/start")
        
    elif data == "cb_ranking":
        ranking = db_ranking(10)
        if not ranking:
            txt = "🏆 *RANKING*\n\nSem dados ainda. Complete um tema para aparecer no ranking!"
        else:
            medals = ["🥇", "🥈", "🥉"]
            txt = "🏆 *RANKING DA TURMA* 🏆\n━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n"
            for i, r in enumerate(ranking[:10]):
                medal = medals[i] if i < 3 else f"{i+1}."
                pct = round(r["total_acertos"] / max(r["total_questoes"], 1) * 100)
                temas_feitos = len([t for t in r["temas_completos"].split(",") if t])
                txt += f"{medal} *{r['user_name']}*\n   📊 {pct}% acertos · {r['xp']} XP · Lv.{r['level']} · {temas_feitos}/15 temas\n\n"
        await q.edit_message_text(txt, parse_mode="Markdown")
        
    elif data == "cb_progresso":
        u = db_user(uid)
        temas_feitos = len([t for t in u["temas_completos"].split(",") if t])
        pct = round(u["total_acertos"] / max(u["total_questoes"], 1) * 100)
        txt = f"📊 *SEU PROGRESSO*\n━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n"
        txt += f"👤 *{u['user_name']}*\n"
        txt += f"🎖️ Level: *{u['level']}*\n"
        txt += f"⭐ XP: *{u['xp']}*\n"
        txt += f"✅ Acertos: *{u['total_acertos']}/{u['total_questoes']}* ({pct}%)\n"
        txt += f"📚 Temas concluídos: *{temas_feitos}/15*\n"
        txt += f"🏆 Próximo nível: *{200 - (u['xp'] % 200)} XP*"
        await q.edit_message_text(txt, parse_mode="Markdown")
        
    elif data == "cb_erros":
        erros = db_errors(uid, 10)
        if not erros:
            txt = "📓 *CADERNO DE ERROS*\n\n✅ Parabéns! Você não tem erros registrados!\n\n_Continue assim!_"
        else:
            txt = f"📓 *CADERNO DE ERROS* ({len(erros)} registros)\n━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n"
            for i, e in enumerate(erros[:10], 1):
                tema_nome = TEMAS.get(e["tema_id"], e["tema_id"])
                txt += f"*{i}. {tema_nome}*\n"
                txt += f"❓ {e['pergunta'][:80]}...\n"
                txt += f"✅ {e['correta']}\n\n"
        await q.edit_message_text(txt, parse_mode="Markdown")
        
    elif data == "cb_lembrete":
        conn = get_db()
        r = conn.execute("SELECT lembrete FROM krona_users WHERE user_id=?", (uid,)).fetchone()
        novo = 0 if (r and r["lembrete"]) else 1
        conn.execute("UPDATE krona_users SET lembrete=?, chat_id=? WHERE user_id=?", (novo, q.message.chat_id, uid))
        conn.commit()
        conn.close()
        st = "ativado 🔔" if novo else "desativado 🔕"
        await q.edit_message_text(
            f"◆ Lembrete *{st}*!\n\n" + 
            ("Você receberá notificações diárias às 19h." if novo else "Notificações desativadas."),
            parse_mode="Markdown"
        )

async def finalizar(msg, uid, tema_id):
    s = sessions.get(uid)
    if not s:
        return
    ac, tot = s["score"], s["total"]
    xp_g = calc_xp(ac, tot)
    nota, new_xp, new_lvl = db_save_result(uid, tema_id, ac, tot, xp_g)
    
    # Feedback por nota
    if nota >= 9:
        emoji, frase = "🏆", "EXCELENTE! Domínio total do tema!"
    elif nota >= 7:
        emoji, frase = "🥇", "MUITO BOM! Continue assim!"
    elif nota >= 5:
        emoji, frase = "📚", "BOM! Revise os pontos que errou."
    else:
        emoji, frase = "📖", "REVEJA O CONTEÚDO antes de continuar."
    
    txt = f"{emoji} *RESULTADO FINAL* {emoji}\n━━━━━━━━━━━━━━━━━━━━━━━━\n\n"
    txt += f"📚 *{TEMAS.get(tema_id, tema_id)}*\n\n"
    txt += f"✅ *Acertos:* {ac}/{tot}\n"
    txt += f"📈 *Nota:* {nota:.1f}/10\n"
    txt += f"⭐ *XP ganho:* +{xp_g}\n"
    txt += f"🎖️ *XP total:* {new_xp} (Lv.{new_lvl})\n\n"
    txt += f"💬 *{frase}*"
    
    kb = InlineKeyboardMarkup([
        [InlineKeyboardButton("◆ Abrir KRONA", web_app=WebAppInfo(url=WEBAPP_URL))],
        [InlineKeyboardButton("🏆 Ranking", callback_data="cb_ranking"), 
         InlineKeyboardButton("📓 Erros", callback_data="cb_erros")]
    ])
    
    await msg.reply_text(txt, reply_markup=kb, parse_mode="Markdown")
    del sessions[uid]

# ================== API FLASK ==================
app = Flask(__name__)
CORS(app)

@app.route('/health', methods=['GET'])
def health():
    return "OK", 200

@app.route('/init-db', methods=['GET'])
def init_db_route():
    try:
        if init_db():
            return "Database initialized successfully!", 200
        return "Database initialization failed.", 500
    except Exception as e:
        return f"Error: {e}", 500

@app.route('/api/user/<int:user_id>', methods=['GET'])
def api_user(user_id):
    try:
        user = db_user(user_id)
        temas_completos = len([t for t in user["temas_completos"].split(",") if t]) if user["temas_completos"] else 0
        stats_by_tema = db_user_stats_by_tema(user_id)
        
        # Preparar dados por tema para o gráfico
        temas_stats = []
        for tema_id, tema_nome in TEMAS.items():
            tema_data = next((s for s in stats_by_tema if s["tema_id"] == tema_id), None)
            temas_stats.append({
                "id": tema_id,
                "nome": tema_nome,
                "completado": tema_id in user["temas_completos"].split(",") if user["temas_completos"] else False,
                "acertos": tema_data["total_acertos"] if tema_data else 0,
                "total": tema_data["total_questoes"] if tema_data else 0,
                "nota": round(tema_data["media_nota"], 1) if tema_data else None
            })
        
        return jsonify({
            "user_id": user["user_id"],
            "nome": user["user_name"],
            "xp": user["xp"],
            "level": user["level"],
            "acertos": user["total_acertos"],
            "total_questoes": user["total_questoes"],
            "temas_completos": temas_completos,
            "percentual_geral": round((user["total_acertos"] / max(user["total_questoes"], 1)) * 100, 1),
            "temas_stats": temas_stats
        })
    except Exception as e:
        log.error(f"Erro /api/user: {e}")
        return jsonify({"error": str(e)}), 500

@app.route('/api/ranking', methods=['GET'])
def api_ranking():
    try:
        ranking_data = db_ranking(15)
        return jsonify({"ranking": ranking_data})
    except Exception as e:
        log.error(f"Erro /api/ranking: {e}")
        return jsonify({"ranking": []}), 200

@app.route('/api/errors/<int:user_id>', methods=['GET'])
def api_errors(user_id):
    try:
        errors_data = db_errors(user_id, 20)
        # Formatar com nome do tema
        for e in errors_data:
            e["tema_nome"] = TEMAS.get(e["tema_id"], e["tema_id"])
        return jsonify({"errors": errors_data, "total": len(errors_data)})
    except Exception as e:
        log.error(f"Erro /api/errors: {e}")
        return jsonify({"errors": [], "total": 0}), 200

@app.route('/api/progress/<int:user_id>', methods=['GET'])
def api_progress(user_id):
    try:
        results = db_user_results(user_id)
        return jsonify({"results": results})
    except Exception as e:
        log.error(f"Erro /api/progress: {e}")
        return jsonify({"results": []}), 200

@app.route('/db-test', methods=['GET'])
def db_test():
    try:
        conn = get_db()
        conn.execute("SELECT 1 FROM krona_users LIMIT 1")
        conn.close()
        return "Database connection successful!", 200
    except Exception as e:
        return f"Database connection failed: {e}", 500

# ================== MAIN ==================
def main():
    init_db()
    
    from threading import Thread
    def run_flask():
        app.run(host='0.0.0.0', port=PORT)
    Thread(target=run_flask, daemon=True).start()
    
    if not TOKEN:
        log.error("❌ TOKEN não configurado!")
        return
    
    application = Application.builder().token(TOKEN).build()
    application.add_handler(CommandHandler("start", cmd_start))
    application.add_handler(MessageHandler(filters.StatusUpdate.WEB_APP_DATA, handle_webapp))
    application.add_handler(CallbackQueryHandler(handle_cb))
    
    log.info("=" * 42)
    log.info("◆ KRONA Analytics — COMPLETO E SINCRONIZADO")
    log.info(f"🌐 WebApp: {WEBAPP_URL}")
    log.info(f"🔌 API Porta: {PORT}")
    log.info("=" * 42)
    
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == "__main__":
    main()
