
## **WhatsApp Business API 收費一覽**

WhatsApp Business API 需要收費，但不是按單次訊息收費，與 SMS 不同，所以整體成本不會太高。

### **商戶發起／用戶發起收費比較**

商戶發起訊息收費較貴，並會根據 3 種類型對話區分收費，並需要使用預設的 WhatsApp 訊息 template，而 SHOPLINE 已提供基本 template。用戶發起訊息較便宜，只有 1 種收費模式，而且沒有 template 限制。

![WhatsApp Business API HK fee](https://storage.googleapis.com/pos.shopline.hk/2023/06/whatsapp-business-api-hk-fee-2.png)

### **每 24 小時按訊息類型計算費用**

所有對話都以 24 小時為單位計算，從商戶第一條訊息發送 / 回覆起計一次收費，每個對話持續 24 小時，在一個對話有效期內裡發送同類型的訊息才是免費。但如果商戶在該訊息的 24 小時對話框內，發起另一對話訊息，此對話訊息需收費，並另外計算為一個新訊息的 24 小時對話框。

**情境一：**

由於第一條訊息由用戶發起，商戶從回覆時間（11:00）起計算一次「用戶發起訊息費用」。商戶在溝通過程中，給此顧客發送了「營銷推廣」類型的訊息模板（15:00），即使是 24 小時內，仍會計算行銷對話收費。及後商戶發送了「客服消息」類型的訊息模板（17:00），即使是 24 小時內，仍會計算客服對話收費。

![WhatsApp Business API conversation fee](https://storage.googleapis.com/pos.shopline.hk/2023/06/whatsapp-business-api-conversation1.png)

**情境二：**

由於第一條訊息由用戶發起，商戶從回覆時間（11:00）起計算一次「用戶發起訊息費用」。商戶在溝通過程中，給此顧客發送了「客服消息」類型的訊息模板（15:00），即使是 24 小時內，仍會計算客服對話收費。及後商戶再給此顧客發送了「客服消息」類型的訊息模板，由於 24 小時內再次發送「客服消息」類型的對話，所以不會額外收費。

![WhatsApp Business API conversation fee](https://storage.googleapis.com/pos.shopline.hk/2023/06/whatsapp-business-api-conversation2.png.png)

**情境三：**

由於第一條訊息由商戶發起，從發送時間（10:00）起計算一次「發送營銷訊息費用」。商戶於（11:00）回覆用戶發起的查詢，因商戶已開啟「營銷訊息」對話框，發送即時訊息不會額外收費。商戶在溝通過程中，給此顧客發送了「客服消息」類型的訊息模板（13:00），即使是 24 小時內，都會計算客服對話收費。及後商戶再給此顧客發送了「客服消息」類型的訊息模板，由於此「客服消息」類型仍於 24 小時對話框內，所以不會額外收費。

![WhatsApp Business API conversation fee](https://storage.googleapis.com/pos.shopline.hk/2023/06/whatsapp-business-api-conversation-3.png.png)

### **每個帳號每月可獲 1,000 次對話收費豁免**

每個 WhatsApp 商業帳戶每月首 1,000 次對話免費，只限「用戶發起的對話」，其後按對話量收費，並會根據帳戶時區每月更新限額。

### **免費訊息：由廣告引流 WhatsApp 展開對話**

由 FB 頁面 / Meta 廣告用戶點擊廣告 / Facebook 頁面上的 “Click-to-WhatsApp” (Call To Action) 按鈕，而引流到 WhatsApp 展開對話，訊息不會收費，但 72 小時後持續對話便需收費。