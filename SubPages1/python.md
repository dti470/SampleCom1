import base64
import json
import os
import requests
from google.cloud import translate_v2 as translate
import functions_framework
import google.auth
from googleapiclient.discovery import build
import datetime

# ここに自分の情報へ変更してね
SPREADSHEET_ID = "ここにスプレッドシートIDを設定してください"
RANGE_NAME = "シート1!A:I"

def update_spreadsheet(incident_id, status, timestamp, products, project, title, description):
    try:
        credentials, _ = google.auth.default(scopes=['https://www.googleapis.com/auth/spreadsheets'])
        service = build('sheets','v4', credentials=credentials)
        sheet = service.spreadsheets()

        result = sheet.values().get(spreadsheetId=SPREADSHEET_ID, range=RANGE_NAME).execute()
        values = result.get('values', [])

        row_index = -1
        for i, row in enumerate(values):
            if(len(row)) > 0 and row[0] == incident_id:
                row_index = i + 1
                break

        start_time_str = timestamp
        duration_str = "-"

        if row_index != -1:
            if len(values[row_index-1])> 2:
                start_time_str = values[row_index-1][2]

            try:
                fmt = "%Y-%m-%d %H:%M:%S"
                t_start = datetime.datetime.strptime(start_time_str, fmt)
                t_now = datetime.datetime.strptime(timestamp, fmt)
                diff = t_now - t_start
                duration_str = f"{diff.total_seconds() / 60:.1f}分"
            except:
                duration_str = "計算エラー"
        
        row_data = [incident_id, status, start_time_str, timestamp, products, project, title, description, duration_str]

        if row_index == -1:
            body = {'values': [row_data]}
            sheet.values().append(
                spreadsheetId=SPREADSHEET_ID, range=RANGE_NAME,
                valueInputOption="USER_ENTERED", body=body).execute()
            return "NEW"
        else:
            body = {'values': [row_data]}
            sheet.values().update(
                spreadsheetId=SPREADSHEET_ID, range=f"シート1!A{row_index}:I{row_index}",
                valueInputOption="USER_ENTERED", body=body).execute()
            return "UPDATE"
    except Exception as e:
        print(f"Spreadsheet Error: {e}")
        return "ERROR"

@functions_framework.cloud_event
def process_incident(cloud_event):
    # 1.Pub/Sub メッセージの取得とでコード
    if "message" in cloud_event.data and "data" in cloud_event.data["message"]:
        pubsub_message = base64.b64decode(cloud_event.data["message"]["data"]).decode('utf-8')
        try:
            log_data = json.loads(pubsub_message)
        except json.JSONDecodeError:
            print("Error: Invalid JSON format in Pub/Sub message.")
            return
    else:
        print("Error: No data found in Pub/Sub message.")
        return
    
    # 2.必要なデータの抽出（.get()で安全に取得）
    json_payload = log_data.get("jsonPayload", {})
    incident_name = json_payload.get("name", "Unknown-ID").split('/')[-1]
    title_en = json_payload.get("title", "No Title")
    desc_en = json_payload.get("detailedDescription", "No Description")
    state = json_payload.get("state", "UNKNOWN")

    resource = log_data.get("resource", {})
    project_id = resource.get("labels",{}).get("project_id", "Organization/Other")

    # 影響を受けるプロダクトをカンマ区切りの文字列にする
    products_list = json_payload.get("impactedProducts", ["Unknown Product"])
    products = ", ".join(products_list) if isinstance(products_list, list) else str(products_list)

    # 3. Cloud Translation API で日本語に翻訳
    translate_client = translate.Client()
    try:
        title_ja = translate_client.translate(title_en, target_language='ja')['translatedText']
        desc_ja = translate_client.translate(desc_en, target_language='ja')['translatedText']
    except Exception as e:
        print(f"Translate API Error: {e}")
        # 翻訳APIでエラーが起きた場合は、ログをロストさせないため英語のままフォールバック
        title_ja = title_en
        desc_ja = desc_en

    JST = datetime.timezone(datetime.timedelta(hours=+9), 'JST')
    now_str = datetime.datetime.now(JST).strftime("%Y-%m-%d %H:%M:%S")

    action = update_spreadsheet(incident_name, state, now_str, products, project_id, title_ja, desc_ja)

    if action == "NEW":
        status_label = "🔴 [新規発生]"
    elif state in ["CLOSED", "RESOLVED"]:
        status_label = "🟢 [復旧・クローズ]" 
    else:
        status_label = "🟡 [更新]"

    # 4. Google Chat への通知メッセージ成型
    webhook_url = os.environ.get("WEBHOOK_URL")
    if not webhook_url:
        print("Error: WEBHOOK_URL environment variable is not set.")
        return

    message_text= (
        f"{status_label} *GCP Service Health インシデント ({state})*\n"
        f"*[PJ]* {project_id}\n"
        f"*[影響サービス]* {products}\n"
        f"*[タイトル]* {title_ja}\n"
        f"*[詳細]*\n{desc_ja}"
    )

    chat_payload = {"text": message_text}

    #5 Webhook URL へ POST リクエスト
    try:
        response = requests.post(
            webhook_url,
            data=json.dumps(chat_payload),
            headers={'Content-Type': 'application/json; charset=UTF-8'}

        )
        response.raise_for_status() # HTTPエラー時に例外を発生させる
        print("Successfully sent message to Google Chat.") 
    except requests.exceptions.RequestException as e:
        print(f"Error sending to Google Chat: {e}")