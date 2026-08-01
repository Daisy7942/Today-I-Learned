1. 텔레그램 설치 및 로그인

PC 또는 모바일에서 Telegram에 로그인합니다.

2. BotFather 열기

공식 BotFather를 검색하거나 아래를 엽니다.

BotFather

반드시 파란색 인증 배지(✔️) 가 있는 공식 BotFather를 사용하세요.

3. /start 입력

채팅창에 다음을 입력합니다.

/start
4. 새 봇 생성

다음을 입력합니다.

/newbot
5. 봇 이름 입력

예시

My Notification Bot

이 이름은 사용자에게 보이는 이름입니다.

6. 봇 Username 입력

예시

my_notification_bot

조건은 다음과 같습니다.

반드시 bot 또는 Bot으로 끝나야 합니다.
영문, 숫자, _만 사용 가능합니다.
이미 사용 중인 이름은 사용할 수 없습니다.
7. 토큰 발급

정상적으로 생성되면 BotFather가 다음과 같은 메시지를 보냅니다.

Done! Congratulations on your new bot.

Use this token to access the HTTP API:

1234567890:AAHxxxxxxxxxxxxxxxxxxxxxxxxxxxx

이 문자열이 Bot Token입니다.


 

5. 성공 시 메시지와 함께 HTTI API Token을 줍니다. 봇 토큰이니 따로 메모합니다.


토큰 정보는 반드사 외부에 노출되지 않도록 합니다

 

 

 

6. 텔레그램 검색창에 방금 생성한 @사용자 아이디를 검색합니다.



 

START 클릭

 

7. Chat ID를 확인하기 위해 해당 채팅방에서 아무 메시지나 입력합니다.


 

 

8. 브라우저 주소창에 아래 주소를 복사해서 붙여넣습니다. 토큰 부분은 아까 받은 API 토큰으로 넣습니다.

https://api.telegram.org/bot<토큰>/getUpdates
 

이때 화면에 다음과 같이 나온다면 채팅방에 다시 한번 아무 메시지나 보내고 브라우저 창을 새로고침합니다.

{"ok":true,"result":[]}
 

 


chat 객체 내 id가 chat_id입니다. 

 

 


```python
.env
TELEGRAM_BOT_TOKEN='여기에 발급토큰 입력'
TELEGRAM_CHAT_ID='여기에 Chat ID 입력'
```



---
코드
```python
import requests
load_dotenv()


# 텔레그램 알림 설정
TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
TELEGRAM_CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

# web_server.py의 대기질 등급 기준과 동일 (PM2.5 기준)
STATUS_BANDS = [
    (15, "좋음"),
    (35, "보통"),
    (75, "나쁨"),
    (float("inf"), "매우 나쁨"),
]

# web_server.py의 GUIDE_BY_STATUS와 동일한 행동요령 문구
GUIDE_BY_STATUS = {
    "좋음": "실외 활동하기 좋은 날이에요.",
    "보통": "대부분 활동은 괜찮아요. 민감군(어린이·노약자·호흡기질환자)은 장시간 실외활동을 주의하세요.",
    "나쁨": "장시간 실외활동을 자제하고, 외출 시 마스크 착용을 권장해요.",
    "매우 나쁨": "실외활동을 자제하고, 외출 시 마스크를 꼭 착용하세요.",
}
```
float("inf") 의 의미
파이썬에서 양의 무한대(Positive Infinity)를 나타내는 표현


[] 리스트 : 순서가 있는 데이터의 집합  
() 튜플 : 수정되지 않는(임계값, 상태명) 한 쌍을 묶어두는 자료형  
{} Set : 중복을 허용하지 않고, 순서가 없는 자료형인 집합


for문  
반환값 for 리스트안의 컬럼1,리스트안의 컬럼2인데 반복해서 나오고 싶은것  in 리스트인데 반복돌릴 대상 :
  
실제 예: for limit,status in STATUS_BANDS:  
예2: v for v in values if v is not None
 ->values의 내부를 반복문 돌릴건데 v가 none값이면 맨앞 v를 반환한다.

```python
def get_air_quality(value):
    # 구간별로 순회하며 수치가 기준값보다 작거나 같은지 확인
    for limit, status in STATUS_BANDS:
        if value <= limit:
            return status

# 테스트
current_pm25 = 45  # 미세먼지 수치 45

status = get_air_quality(current_pm25)  # "나쁨" 반환 (35 초과 75 이하)

# 경고 알림 여부 체크
if status in ALERT_STATUSES:
    print(f"경고: 현재 상태는 [{status}]입니다. 마스크를 착용하세요!")
else:
    print(f"현재 상태는 [{status}]입니다.")
```

```python
def status_for(pm25):
    """PM2.5 평균값으로 대기질 등급을 판정합니다 (web_server.py와 동일 기준)."""
    for threshold, label in STATUS_BANDS:
        if pm25 <= threshold:
            return label
    return "매우 나쁨"


def send_telegram_alert(text):
    """텔레그램 봇으로 알림 메시지를 전송합니다. 설정이 없으면 건너뜁니다."""
    if not TELEGRAM_BOT_TOKEN or not TELEGRAM_CHAT_ID:
        print("텔레그램 설정이 없어 알림을 건너뜁니다.")
        return

    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    response = requests.post(
        url, data={"chat_id": TELEGRAM_CHAT_ID, "text": text}, timeout=10
    )
    print("텔레그램 알림 전송 상태:", response.status_code)


def get_avg_fpm(dt):
    """특정 시간(dt)의 평균 초미세먼지(PM2.5) 값을 DB에서 조회합니다. 데이터가 없으면 None."""
    conn = get_connection()
    cursor = conn.cursor()

    cursor.execute("SELECT AVG(fpm) FROM air_quality_hourly WHERE msrmt_dt = %s", (dt,))
    avg_fpm = cursor.fetchone()[0]

    cursor.close()
    conn.close()

    return float(avg_fpm) if avg_fpm is not None else None


def check_and_alert(rows, request_dt):
    """직전 시간 대비 대기질 등급이 바뀌었을 때만 텔레그램 알림을 보냅니다."""
    values = [to_number(row.get("FPM")) for row in rows]
    values = [v for v in values if v is not None]

    if not values:
        return

    current_avg = sum(values) / len(values)
    current_status = status_for(current_avg)

    previous_avg = get_avg_fpm(request_dt - timedelta(hours=1))
    if previous_avg is None:
        return  # 비교할 직전 시간 데이터가 없으면 알림 판단 불가

    previous_status = status_for(previous_avg)

    if current_status == previous_status:
        return

    guide = GUIDE_BY_STATUS[current_status]
    message = f"[공기질 알림] 미세먼지가 {current_status} 수준입니다. {guide}"

    send_telegram_alert(message)
    log_telegram_alert(request_dt, previous_status, current_status, current_avg, message)


def log_telegram_alert(request_dt, previous_status, current_status, avg_fpm, message):
    """발송된 알림 이력을 DB에 기록합니다 (대시보드 표시용)."""
    conn = get_connection()
    cursor = conn.cursor()

    cursor.execute(
        """
        INSERT INTO telegram_alerts
            (msrmt_dt, previous_status, current_status, avg_fpm, message)
        VALUES (%s, %s, %s, %s, %s)
        """,
        (request_dt, previous_status, current_status, avg_fpm, message),
    )

    conn.commit()
    cursor.close()
    conn.close()

```

```python
# 직전 시간 대비 대기질 등급이 바뀌었으면 텔레그램 알림 전송
check_and_alert(rows, request_dt)
```
