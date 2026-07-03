# 基礎資訊 (Base Info)
Base URL: https://script.google.com/macros/s/AKfycbyrt8QKSreE23wA32wjfj2lRGI6tgGgARdzLk2Q5OkCgrEEX0hjvWTpWiJ6hF3bLt94/exec

# 資料來源: Google Sheets 工作表 "2026"

# 驗證機制: 免驗證 (由 GAS 網頁應用程式代管，權限設為 Anyone)

# 跨網域處理 (CORS): 後端已統一封裝為 ContentService.MimeType.JSON 回傳，無 Preflight 問題。

# 取得歷史戰績與統計數據 (GET)
讀取試算表中的所有對局資料。後端會自動以 A 欄為日期，並從 B 欄起自動過濾掉欄位名稱包含「東」或「驗證」的格子，將其餘留存欄位定義為有效玩家（如 p1 ~ p7 等，數量隨 Google Sheet 欄位動態增減）。
URL: /exec
Method: GET
Headers: Accept: application/json
請求範例 (Request Example)
HTTP
GET /macros/s/AKfycbyrt8QKSreE23wA32wjfj2lRGI6tgGgARdzLk2Q5OkCgrEEX0hjvWTpWiJ6hF3bLt94/exec HTTP/1.1
Host: script.google.com
回傳回應 (Response)Content-Type: application/json
成功回應 (200 OK)
後端會直接完成資料聚合（Data Aggregation），回傳整合成歷史對局（依時間倒序）以及當季排行榜所需的累積數據。
JSON
{
  "status": "success",
  "dashboard": {
    "p1": {
      "name": "林小明",
      "total": 3250,
      "trend": [300, -100, 500, 1200, -200] 
    },
    "p2": {
      "name": "陳大同",
      "total": -1150,
      "trend": [-200, 0, -150, 400, -100]
    },
    "p3": {
      "name": "張三",
      "total": 800,
      "trend": [0, 200, -100, -500, 300]
    },
    "p4": {
      "name": "李四",
      "total": -2900,
      "trend": [-100, -100, -250, -1100, 0]
    }
  },
  "history": [
    {
      "date": "2026/07/03",
      "p1": 300,
      "p2": -100,
      "p3": -150,
      "p4": -50
    },
    {
      "date": "2026/06/28",
      "p1": 1200,
      "p2": 400,
      "p3": -500,
      "p4": -1100
    }
  ]
}
註：dashboard.[pKey].trend 陣列固定僅切出該玩家最後 5 場的對局分數，供前端繪製 Sparkline 波動圖使用。異常回應JSON{
  "status": "error",
  "message": "找不到名為 '2026' 的工作表"
}
# 新增單場對局記錄 (POST)
將一筆新的對局結果寫入試算表末端。寫入時會自動鎖定 A 欄寫入日期，其餘分數依據 p1 ~ pN 代號精準對齊 Google Sheet 的玩家欄位，並自動將「驗證」欄位補 0。
URL: /exec
Method: POST
Content-Type: application/x-www-form-urlencoded(注意：請務必以此 Content-Type 送出，若採 application/json 會觸發瀏覽器 CORS Preflight 阻擋)

請求參數 (Parameters)
參數名稱 | 類型 | 必填 | 說明 | 範例
date | String | 否 | 對局日期，格式為 YYYY/MM/DD。若留空後端自動抓取當前系統今日日期。 | 2026/07/03
p1 | Number | 否 | 試算表內解析出的第 1 位玩家得分（未填預設為 0）| 500 | 
p2 | Number | 否 | 試算表內解析出的第 2 位玩家得分（未填預設為 0）| 200 |
p3 | Number | 否 | 試算表內解析出的第 3 位玩家得分（未填預設為 0）| 200 |
p4 | Number | 否 | 試算表內解析出的第 4 位玩家得分（未填預設為 0）| 100 |
後端支援動態擴充，參數可一併帶入 p5、p6、p7 等。當局沒打牌的玩家分數請傳 0 或不傳。
核心業務邏輯驗證 (Business Logic Validation)零和守恆定律： 後端接收到參數後，會將所有 p1 到 pN 的分數進行加總。總和必須絕對等於 0。若不等於 0，後端將拒絕寫入資料庫並回傳 validation_error。
回傳回應 (Response)
成功回應 (200 OK)
JSON
{
  "status": "success",
  "message": "記錄已成功寫入 2026 工作表"
}
資料驗證失敗回應 (400 Bad Request / Custom Status)JSON{
  "status": "validation_error",
  "message": "四家分數相加必須等於 0！目前總和：100"
}
後端開發調試 Tips測試工具選擇： 因 GAS 的部署特性，進行 POST 測試時，工具（如 Postman 或 cURL）必須啟用 Follow Redirects 功能（cURL 請加 -L），因為 Google 會將 POST 請求經由 302 重導向 移至暫存節點執行。欄位動態性： 在撰寫前端或進行系統對接時，玩家名字不需要寫死在 Code 裡，直接在 GET 請求後讀取 dashboard.pX.name 即可，這能保持整套系統未來更換牌友時的擴充彈性。



# 畫面的風格
金融看盤風 (Dark Mode Dashboard)
這個風格偏向數位資產或 ETF 的看盤軟體，強調數字的對比度與數據的專業感。打麻將本質上就是一種資金的流動，用這種風格會有一種在操作投資組合的趣味感。
主色調： 深灰黑底色 (如 #121212)，降低長時間看螢幕的刺眼感。
強調色： 嚴格遵循台股的紅綠配色。贏錢顯示鮮紅色，輸錢顯示草綠色，零則為低調的灰色。
排版特性： 資訊密度高但整齊。每個玩家的戰績會像一個個「幣種」或「標的」卡片，點擊卡片直接進入該玩家的獨立結算畫面。
視覺元素： 扁平化設計，邊界清晰，搭配無襯線字體，呈現冷靜、俐落的科技感。

# 功能
1.帳號登入(需驗證)
2.新增記錄功能
    新增記錄時，要檢核是否正確(參賽者+場地費的總合要為0)
3.排行顯示
4.歷史波動圖

# 部署至 GitHub Pages
這部分就是標準的 Git 流程：
將上述的 index.html 放入一個全新的 GitHub Repository 中。
進入 Repo 的 Settings -> Pages。
在 Build and deployment 的 Branch 選擇 main (或 master)，資料夾選擇 / (root)，然後 Save。
等待幾分鐘，GitHub Pages 的 URL 產生後，你就能隨時用手機開網頁來記帳了。