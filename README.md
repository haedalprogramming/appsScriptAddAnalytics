# appsScriptAddAnalytics
구글 앱스스크립트와 구글 애널리틱스 붙이기

Google Apps Script로 만든 홈페이지(웹 앱)에 Google Analytics를 추가하여 참가자를 추적하는 방법을 단계별로 정리해드릴게요.

---

## 1. **Google Analytics 계정 및 추적 코드 설정**
1. **Google Analytics 계정 생성**
   - [Google Analytics](https://analytics.google.com/)에 로그인하여 새 계정을 생성합니다.
   - **GA4 (Google Analytics 4) 속성**을 만듭니다.

2. **추적 ID 확인**
   - GA4의 **데이터 스트림**에서 **Measurement ID (G-XXXXXXX)**를 복사합니다.

---

## 2. **Google Apps Script 웹 앱에 Google Analytics 삽입**

### 방법 1: 클라이언트 측 GA 태그 추가 (HTML에 직접 삽입)
Apps Script로 만든 웹 앱이 HTML을 포함하는 경우, `<head>` 부분에 Google Analytics 스크립트를 추가할 수 있습니다.

```javascript
<!DOCTYPE html>
<html>
<head>
  <title>My Web App</title>
  <!-- Google Analytics -->
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXX"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-XXXXXXX');  // 추적 ID 입력
  </script>
</head>
<body>
  <h1>Welcome to My Web App</h1>
</body>
</html>
```
- `G-XXXXXXX` 부분을 본인의 Google Analytics Measurement ID로 변경하세요.

### 방법 2: 서버 측 이벤트 트래킹 (Google Apps Script 사용)
웹 앱이 동적으로 데이터를 처리하는 경우, 서버 측에서 Google Analytics에 데이터를 보낼 수도 있습니다.

#### **Google Analytics Measurement Protocol을 활용한 데이터 전송**
Google Apps Script에서 HTTP 요청을 사용해 직접 이벤트를 Google Analytics로 전송할 수 있습니다.

1. **Google Apps Script 코드에 서버 측 트래킹 추가**
```javascript
function logAnalyticsEvent(clientId, eventName) {
  var measurementId = "G-XXXXXXX"; // Google Analytics 측정 ID
  var apiSecret = "YOUR_API_SECRET"; // Google Analytics API 비밀 키

  var url = "https://www.google-analytics.com/mp/collect?measurement_id=" + measurementId + "&api_secret=" + apiSecret;

  var payload = {
    client_id: clientId, // 클라이언트 식별 ID (랜덤 생성 가능)
    events: [{
      name: eventName,
      params: {
        engagement_time_msec: 100
      }
    }]
  };

  var options = {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload)
  };

  UrlFetchApp.fetch(url, options);
}
```
- `YOUR_API_SECRET` 값은 GA4의 **Measurement Protocol API 설정**에서 생성 가능합니다.
- `clientId`는 사용자의 고유 식별자로, `SessionStorage` 또는 `Cookies`를 활용해 관리할 수 있습니다.

2. **웹 앱에서 사용자가 페이지를 방문할 때 실행**
```javascript
function doGet(e) {
  var htmlOutput = HtmlService.createHtmlOutputFromFile("index");
  
  // Google Analytics 이벤트 로깅
  logAnalyticsEvent("random-client-id", "page_view");
  
  return htmlOutput;
}
```
- 방문자 정보를 추적하고 싶은 이벤트에 `logAnalyticsEvent` 함수를 추가하면 됩니다.

---

## 3. **참가자 식별 및 트래킹 개선**
GA4는 기본적으로 익명 데이터를 수집합니다. 하지만 특정 참가자를 추적하려면 다음 방법을 사용할 수 있습니다.

### 1) **URL Parameter 활용**
각 참가자에게 고유한 `?user_id=12345` 같은 URL 파라미터를 할당하고, 이를 Analytics 이벤트와 함께 저장할 수 있습니다.
```javascript
var params = new URLSearchParams(window.location.search);
var userId = params.get("user_id");

gtag('config', 'G-XXXXXXX', {
  'user_id': userId
});
```

### 2) **쿠키를 활용한 사용자 식별**
Apps Script에서 `SessionStorage` 또는 `Cookies`를 활용하여 클라이언트를 구분할 수도 있습니다.

```javascript
document.cookie = "userId=12345; path=/";
```

이후 `document.cookie` 값을 읽어 Google Analytics에 전달하면 사용자별 분석이 가능합니다.

---

## 4. **실시간 트래킹 및 데이터 분석**
1. Google Analytics > 실시간 보고서에서 참가자가 접속했는지 확인
2. 이벤트 및 사용자 데이터를 맞춤 대시보드에서 분석

---

### ✅ **결론**
- Google Apps Script 웹 앱에서 Google Analytics를 활용하려면 **GA 태그를 HTML에 추가**하는 방법이 가장 간단함.
- 서버 측에서 **Measurement Protocol API**를 활용하면 좀 더 정밀한 추적이 가능함.
- 참가자별 데이터를 트래킹하려면 **URL Parameter, Cookies**를 활용하면 효과적임.

이 방법들을 조합하면 원하는 참가자 추적 시스템을 구축할 수 있습니다! 🚀
