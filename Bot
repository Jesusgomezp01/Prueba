import requests
import re
import csv
from bs4 import BeautifulSoup
from io import StringIO

from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

TOKEN = "8006124044:AAEmqvOdIAvNJd-ktS27nPCyyzXQl75tCtw"

HEADERS = {"User-Agent": "Mozilla/5.0"}

# ===== DETECCIÓN =====
def detect(html):
    data = html.lower()

    result = {
        "platform": "Unknown",
        "gateway": "Unknown"
    }

    # Plataforma
    if "shopify" in data:
        result["platform"] = "Shopify"
    elif "woocommerce" in data:
        result["platform"] = "WooCommerce"

    # Gateway
    if "stripe" in data:
        result["gateway"] = "Stripe"
    elif "paypal" in data:
        result["gateway"] = "PayPal"
    elif "adyen" in data:
        result["gateway"] = "Adyen"

    return result

# ===== ANALIZAR =====
def analyze(url):
    try:
        res = requests.get(url, headers=HEADERS, timeout=10)
        html = res.text

        soup = BeautifulSoup(html, "html.parser")

        scripts = [s.get("src", "") for s in soup.find_all("script") if s.get("src")]
        full = html + " ".join(scripts)

        detected = detect(full)

        return {
            "url": url,
            "platform": detected["platform"],
            "gateway": detected["gateway"]
        }

    except:
        return {
            "url": url,
            "platform": "Error",
            "gateway": "Error"
        }

# ===== BUSCAR SITIOS =====
def search_sites(query):
    urls = []

    try:
        search_url = f"https://duckduckgo.com/html/?q={query}"
        res = requests.get(search_url, headers=HEADERS)
        soup = BeautifulSoup(res.text, "html.parser")

        for a in soup.select("a.result__a"):
            href = a.get("href")
            if href and "http" in href:
                urls.append(href)

            if len(urls) >= 5:
                break

    except:
        pass

    return list(set(urls))

# ===== BOT =====
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🤖 Bot listo\n\nUsa:\n/search accessories shop"
    )

async def search(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = " ".join(context.args)

    if not query:
        await update.message.reply_text("❌ Usa: /search accessories shop")
        return

    await update.message.reply_text(f"🔍 Buscando: {query}")

    urls = search_sites(query)

    results = []

    for url in urls:
        await update.message.reply_text(f"🌐 Analizando:\n{url}")
        r = analyze(url)
        results.append(r)

    # CSV
    output = StringIO()
    writer = csv.DictWriter(output, fieldnames=["url", "platform", "gateway"])
    writer.writeheader()
    writer.writerows(results)
    output.seek(0)

    # Mensaje
    text = "\n\n".join([
        f"{r['url']}\n🏪 {r['platform']} | 💳 {r['gateway']}"
        for r in results
    ])

    await update.message.reply_text(f"✅ Resultados:\n\n{text}")
    await update.message.reply_document(document=output, filename="results.csv")

# ===== RUN =====
app = ApplicationBuilder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(CommandHandler("search", search))

app.run_polling()
