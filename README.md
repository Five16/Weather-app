# Weather
## For security reasons, the API keys (for OpenWeatherMap and Google Maps) are managed through environment variables rather than being directly embedded in the code. Please register with OpenWeatherMap and Google Cloud, obtain your personal API keys, and set them as `OPENWEATHER_API_KEY` and `GOOGLE_MAPS_API_KEY`, respectively.


```python
import functions_framework
import requests
import os
from datetime import datetime

# 環境変数からAPIキーを取得
API_KEY = os.getenv("OPENWEATHER_API_KEY")
GOOGLE_MAPS_API_KEY = os.getenv("GOOGLE_MAPS_API_KEY")

@functions_framework.http
def get_weather(request):
    city = request.args.get("city", "Tokyo")

    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric&lang=ja"
    res = requests.get(url)
    data = res.json()

    if data.get("cod") != 200:
        return f"<p>天気情報を取得できませんでした: {data.get('message')}</p>", 200, {'Content-Type': 'text/html'}

    main_weather = data["weather"][0]["main"].lower()
    description = data["weather"][0]["description"]
    icon = data["weather"][0]["icon"]
    temp = data["main"]["temp"]
    humidity = data["main"]["humidity"]
    lat = data["coord"]["lat"]
    lon = data["coord"]["lon"]
    now = datetime.now().strftime("%Y-%m-%d %H:%M")

    # 背景グラデーション（画像より安定）
    backgrounds = {
        "clear": "linear-gradient(to top, #a8edea, #fed6e3)",
        "clouds": "linear-gradient(to right, #d7d2cc, #304352)",
        "rain": "linear-gradient(to bottom, #4e54c8, #8f94fb)",
        "snow": "linear-gradient(to bottom, #e0eafc, #cfdef3)",
        "thunderstorm": "linear-gradient(to right, #0f2027, #203a43, #2c5364)",
        "drizzle": "linear-gradient(to right, #74ebd5, #ACB6E5)",
        "mist": "linear-gradient(to right, #bdc3c7, #2c3e50)"
    }
    background = backgrounds.get(main_weather, "#ffffff")

    # 天気に応じたメッセージ
    messages = {
        "clear": "今日は洗濯日和です！外に出かけましょう☀️",
        "clouds": "ちょっと肌寒いかも。羽織るものをお忘れなく☁️",
        "rain": "傘を忘れずに☔ 交通機関にも注意を！",
        "snow": "足元に気をつけて！暖かくして出かけよう⛄",
        "thunderstorm": "落雷注意⚡️今日は屋内で過ごすのが安全かも！",
        "drizzle": "小雨が降ってます☂️ 傘があると安心だね。",
        "mist": "視界が悪いかも…運転には気をつけて！"
    }
    message_text = messages.get(main_weather, "良い一日を！")

   # 服装のアドバイス
    if temp >= 30:
        fashion_advice = "とても暑いので、涼しい服装＆水分補給を！🧢"
    elif temp >= 20:
        fashion_advice = "半袖や薄手の服が快適です。🌤"
    elif temp >= 10:
        fashion_advice = "長袖や羽織りものがあると安心です。🧥"
    else:
        fashion_advice = "コートや手袋が必要かも！暖かくしてね🧣"

    html = f"""
    <!DOCTYPE html>
    <html lang='ja'>
    <head>
        <meta charset='UTF-8'>
        <title>{city} の天気</title>
    </head>
    <body style="margin:0;padding:0;font-family:sans-serif;background: {background};">
	<!-- アイコン画像-->
        <img src="https://i.imgur.com/LYFy7L9.jpeg" alt="MyIcon" style="width: 80px; position: absolute; top: 100px; left: 34%; transform: translateX(-50%); border-radius: 12px;">
	<!-- メインのカード-->
        <div style="max-width: 600px; margin: 50px auto; padding: 20px; background: rgba(255,255,255,0.85); border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.2); text-align: center;">
        <!-- タイトル-->
        <h2 style="margin-top: 20px;">☁️ <strong>{city}</strong> の現在の天気</h2>

	<!-- 検索フォーム-->
            <form method="GET">
                <input type="text" name="city" value="{city}" placeholder="都市名" required>
                <button type="submit">検索</button>
            </form>

	<!--天気情報 -->
            <img src="http://openweathermap.org/img/wn/{icon}@4x.png" alt="{description}" style="width: 100px;">
            <p><strong>{description}</strong></p>
            <p style="color: #e74c3c;">🌡 気温: {temp}℃</p>
            <p style="color: #3498db;">💧 湿度: {humidity}%</p>
            <p style="margin-top: 20px; font-size: 1.1em;"><strong>{message_text}</strong></p>

            <iframe
              width="100%"
              height="300"
              style="border:0; border-radius: 12px;"
              loading="lazy"
              allowfullscreen
              referrerpolicy="no-referrer-when-downgrade"
             src="https://www.google.com/maps/embed/v1/place?key={GOOGLE_MAPS_API_KEY}&q={lat},{lon}"
            </iframe>

            <p style="font-size: 0.9em; color: #555; margin-top: 20px;">📅 {now}</p>
	    <p style="margin-top: 10px; font-size: 1.1em;"><strong>{fashion_advice}</strong></p>
            <p style="font-size: 0.9em; color: #555;">Created by <strong>KOICHIRO</strong></p>
        </div>
    </body>
    </html>
    """

    return html, 200, {'Content-Type': 'text/html'}
