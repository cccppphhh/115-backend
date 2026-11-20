
# Week 2：工具描述與分派

## 1. 實驗環境


- API Endpoint：https://integrate.api.nvidia.com/v1/chat/completions
- Model：minimaxai/minimax-m3.1-flash
- Tool：http_get
- 測試網址：https://example.com

## 2. 第一次 API 回應：取得 tool_calls

HTTP Status：200 OK

模型回傳：

```json
{
  "id": "call-6fa1c71e-b260-4931-a8e1-32cae37288ab",
  "type": "function",
  "function": {
    "name": "http_get",
    "arguments": "{\"url\":\"https://example.com\"}"
  }
}
```


## 3. 工具執行：Python 實際執行

模型建議：

```json
{"url":"https://example.com"}
```

程式執行結果：

```json
{
  "status_code": 200,
  "body_summary": "<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content=\"width=dev"
}
```


## 4. 第二次 API 請求：回填 Tool Message

Python 將工具執行結果加入 messages：

```json
{
  "role": "tool",
  "tool_call_id": "call-6fa1c71e-b260-4931-a8e1-32cae37288ab",
  "content": "{\"status_code\": 200, \"body_summary\": \"<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content=\\\"width=dev\"}"
}
```


## 5. 第二次 API 回應：取得最終 content

HTTP Status：200 OK

模型最終回答：

> 抓取成功（HTTP 200），我看到一個標準的 HTML 頁面，包含 `<head>` 中的字元編碼、favicon 與 viewport meta 標籤，看起來是 example.com 用來示範的簡單頁面。


## 6. 兩次 API 呼叫觀察紀錄

| 步驟 | 模型建議 | 程式實際執行 |
|---|---|---|
| 第一次 API 回應 | name: http_get，arguments: {"url":"https://example.com"} | 尚未執行，只收到工具建議 |
| 工具執行 | 無 | status_code: 200，body_summary 為 HTML 前 120 個字元 |
| 第二次 API 請求 | 無新的工具建議 | 回填 role: "tool"，tool_call_id 與第一次 id 相同 |
| 第二次 API 回應 | 無新的工具建議 | 取得 choices[0].message.content，產生最終回答 |

