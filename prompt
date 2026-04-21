import asyncio
import yfinance as yf
from telegram import Bot
from openpyxl import Workbook

# 建议：生产环境下使用环境变量存储 Token
TELEGRAM_TOKEN = "8580840811:AAEigte-tDQqavaY6cF9exXoce2aJFdOg4U"
CHAT_ID = "8273033205"

# 修正了字典格式，键为 Ticker，值为公司名称
WATCHLIST = {
    "AAPL": "Apple",
    "MSFT": "Microsoft",
    "NVDA": "NVIDIA"
}

def fetch_data(ticker):
    try:
        s = yf.Ticker(ticker)
        # yfinance 的 info 获取有时较慢，这里做了基础兼容处理
        info = s.info if s.info else {}
        return {
            "price": info.get("currentPrice", 0),
            "pe": info.get("trailingPE", 0)
        }
    except Exception as e:
        print(f"获取 {ticker} 数据失败: {e}")
        return {"price": 0, "pe": 0}

def generate_excel(results):
    wb = Workbook()
    ws = wb.active
    ws.append(["公司名称", "股票代码", "现价", "市盈率"]) # 添加表头
    
    for r in results:
        ws.append([
            r["name"],
            r["ticker"],
            r["price"],
            r["pe"]
        ])

    file_path = "report.xlsx"
    wb.save(file_path)
    return file_path

async def main():
    bot = Bot(token=TELEGRAM_TOKEN)
    results = []

    print("正在获取数据...")
    for ticker, name in WATCHLIST.items():
        data = fetch_data(ticker)
        results.append({
            "name": name,
            "ticker": ticker,
            "price": data["price"],
            "pe": data["pe"]
        })

    # 构造消息内容
    msg = "今日美股行情摘要：\n"
    for r in results:
        msg += f"{r['name']} ({r['ticker']}): ${r['price']}\n"

    # 发送文本
    await bot.send_message(chat_id=CHAT_ID, text=msg)

    # 生成并发送文件
    file_path = generate_excel(results)
    with open(file_path, "rb") as f:
        await bot.send_document(chat_id=CHAT_ID, document=f)
    print("报告已发送至 Telegram。")

if __name__ == "__main__":
    asyncio.run(main())
