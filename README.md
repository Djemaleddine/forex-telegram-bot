import yfinance as yf
import pandas as pd
import requests
import os
import time
import matplotlib.pyplot as plt
from datetime import datetime

# إعداد بيانات تليجرام
BOT_TOKEN = "8214626642:AAH47-RyHLn5lhzDf-28rruIsvOqytJGgJc"
CHAT_ID = "828158876"

def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    payload = {"chat_id": CHAT_ID, "text": message, "parse_mode": "Markdown"}
    requests.post(url, data=payload)

# إعداد مجلد الحفظ
folder_name = "forex_data"
os.makedirs(folder_name, exist_ok=True)

# الأزواج
pairs = {
    "EURUSD": "EURUSD=X",
    "GBPUSD": "GBPUSD=X",
    "USDJPY": "JPY=X",
    "USDCHF": "CHF=X",
    "USDCAD": "CAD=X",
    "AUDUSD": "AUDUSD=X",
    "NZDUSD": "NZDUSD=X"
}

# 🔁 تكرار دائم
while True:
    report = ["📊 **تحديث جديد لأسعار الفوركس**\n"]
    
    for name, symbol in pairs.items():
        try:
            data = yf.download(symbol, period="1d", interval="1h")
            if isinstance(data.columns, pd.MultiIndex):
                data.columns = [col[0] for col in data.columns]
            data = data.reset_index()
            latest_price = round(data['Close'].iloc[-1], 5)
            
            # حفظ CSV
            csv_file = os.path.join(folder_name, f"{name}.csv")
            data.to_csv(csv_file, index=False)
            
            # رسم بياني
            plt.figure(figsize=(6,3))
            plt.plot(data['Datetime'], data['Close'], label=name, linewidth=2)
            plt.title(f"{name} - آخر الأسعار")
            plt.xlabel("الوقت")
            plt.ylabel("السعر")
            plt.grid(True)
            chart_path = os.path.join(folder_name, f"{name}.png")
            plt.savefig(chart_path)
            plt.close()
            
            report.append(f"{name}: {latest_price}")
            
            # إرسال الصورة
            files = {'photo': open(chart_path, 'rb')}
            url_photo = f"https://api.telegram.org/bot{BOT_TOKEN}/sendPhoto"
            payload = {"chat_id": CHAT_ID, "caption": f"{name} - آخر تحديث {datetime.now().strftime('%H:%M:%S')}"}
            requests.post(url_photo, data=payload, files=files)
        
        except Exception as e:
            print(f"❌ خطأ في {name}: {e}")
    
    report_text = "\n".join(report)
    report_text += f"\n\n⏰ التوقيت: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
    send_telegram_message(report_text)
    
    print("✅ تم الإرسال إلى تليجرام بنجاح.")
    time.sleep(1800)  # ← كل نصف ساعة
