# 구글 시트 연동 설정 가이드

발행된 모든 견적서가 구글 시트에 자동으로 기록되도록 설정하는 방법입니다.

---

## 1단계: 구글 시트 생성

1. [Google Sheets](https://sheets.google.com) 접속
2. **새 스프레드시트** 생성
3. 시트 이름을 `견적서발행내역` 등으로 변경 (선택)

---

## 2단계: Apps Script 배포

1. 시트에서 **확장 프로그램** → **Apps Script** 클릭
2. `SPREADSHEET_ID`를 본인 시트 ID로 변경 (URL의 `/d/시트ID/` 부분)
3. `SHEET_TAB_NAME`을 데이터를 받을 탭 이름으로 변경 (예: 청구서모음)
4. 기존 코드를 **전부 삭제**하고 아래 코드 붙여넣기:

```javascript
// 시트 ID: 구글 시트 URL의 /d/시트ID/ 부분
var SPREADSHEET_ID = '1-Yt5kFtVJa-uBHDQY1p0fbr9-xb2-lW8cqCH3ojSKbo';
// 탭 이름: 데이터를 받을 시트 탭 이름 (예: 청구서모음)
var SHEET_TAB_NAME = '청구서모음';

function doGet(e) {
  return ContentService.createTextOutput('구글 시트 연동 정상. 견적서 앱의 시트테스트 버튼으로 데이터를 전송하세요.').setMimeType(ContentService.MimeType.TEXT);
}

function doPost(e) {
  try {
    var raw = (e.postData && e.postData.contents) ? e.postData.contents : (e.parameter && e.parameter.data) ? e.parameter.data : '{}';
    var data = JSON.parse(raw);
    var ss = SpreadsheetApp.openById(SPREADSHEET_ID);
    var sheet = ss.getSheetByName(SHEET_TAB_NAME) || ss.getSheets()[0];
    
    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        '수신자', '수신자 연락처', '이메일', '담당 변리사', '견적 생성일', '견적 유효기간',
        '견적세부정보', '제안서제목', '관납료', '대리인수수료', '총금액', '견적서 링크 주소'
      ]);
      sheet.getRange(1, 1, 1, 12).setFontWeight('bold');
    }
    
    var rows = data.rows || [];
    rows.forEach(function(r) {
      sheet.appendRow([
        r.recipient || '',
        r.phone || '',
        r.email || '',
        r.attorney || '',
        r.quoteDate || '',
        r.expiry || '',
        r.quoteInfo || '',
        r.title || '',
        r.govFee || 0,
        r.agentFee || 0,
        r.total || 0,
        r.link || ''
      ]);
    });
    
    return ContentService.createTextOutput(JSON.stringify({ success: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. **저장** (Ctrl+S) — 반드시 저장
4. **배포** → **새 배포** 클릭 (기존 배포 수정이 아님)
5. 유형: **웹 앱** 선택
6. **설명**: (비워두거나 아무거나)
7. **실행할 앱**: **나** (실행 사용자: 나)
8. **액세스 권한**: **모든 사용자** (또는 "링크가 있는 모든 사용자")
9. **배포** 클릭
10. **웹 앱 URL** 복사

---

## 3단계: index.html에 URL 설정

`index.html`을 열어 상단 스크립트 영역에서 `GOOGLE_SHEETS_WEBHOOK` 값을 위에서 복사한 URL로 변경:

```javascript
const GOOGLE_SHEETS_WEBHOOK = 'https://script.google.com/macros/s/여기에복사한URL/exec';
```

- **Netlify 배포** 시 `netlify/functions/sheet-log.js`가 자동으로 프록시로 동작합니다 (CORS 회피).
- URL을 빈 문자열 `''`로 두면 구글 시트 연동을 사용하지 않습니다.

---

## 시트 컬럼 구성 (순서대로)

| # | 컬럼 | 설명 |
|---|------|------|
| 1 | 수신자 | 견적서 수신자 |
| 2 | 수신자 연락처 | 수신자 전화번호 |
| 3 | 이메일 | 수신자 이메일 |
| 4 | 담당 변리사 | 발신 담당 변리사 |
| 5 | 견적 생성일 | 견적서 작성일 |
| 6 | 견적 유효기간 | 견적 유효일 |
| 7 | 견적세부정보 | 견적 상세 정보 |
| 8 | 제안서제목 | 견적서 제목 |
| 9 | 관납료 | 비용세부항목 - 관납료 |
| 10 | 대리인수수료 | 비용세부항목 - 대리인수수료 |
| 11 | 총금액 | VAT 포함 총액 |
| 12 | 견적서 링크 주소 | 고객 공유용 URL |

### 비용세부항목 행 분리

견적서에서 비용세부항목을 2개 이상 추가한 경우, **각 항목마다 별도 행**으로 기록됩니다.  
예: 항목 3개 → 시트에 3행 추가 (수신자·담당변리사·총금액 등 공통 정보는 각 행에 동일하게 반복)

---

## 파일 구조

```
teheran-bill/
├── index.html
├── netlify/
│   └── functions/
│       └── sheet-log.js   ← 구글 시트 프록시 (Netlify 자동 배포)
└── GOOGLE_SHEETS_SETUP.md
```

## 중요: Netlify Function 필요

구글 시트 연동은 **Netlify Function**을 통해 동작합니다.  
`netlify/functions/sheet-log.js` 파일이 포함된 상태로 Netlify에 배포해야 합니다.  
(브라우저 → Netlify Function → Google Apps Script 순으로 전달)

---

## 문제 해결 (데이터가 안 들어올 때)

1. **Netlify 배포 확인**  
   `netlify/functions/sheet-log.js`가 포함되어 배포되었는지 확인하세요.

2. **Apps Script 새 버전 배포**  
   코드 수정 후 반드시 **배포 → 배포 관리 → 수정 → 새 버전 → 배포** 해야 변경사항이 적용됩니다.

3. **액세스 권한 확인**  
   배포 시 **액세스 권한: 모든 사용자** (또는 "링크가 있는 사용자")로 설정되어 있어야 합니다. "나"만으로 설정하면 외부에서 전송 시 실패합니다.

4. **시트 테스트 버튼**  
   견적 마스터 상단의 **시트테스트** 버튼으로 현재 입력값을 구글 시트에 바로 전송해볼 수 있습니다. 구글 시트를 새로고침해 데이터가 들어왔는지 확인하세요.

5. **SPREADSHEET_ID, SHEET_TAB_NAME 확인**  
   - `SPREADSHEET_ID`: 구글 시트 URL의 `/d/시트ID/` 부분  
   - `SHEET_TAB_NAME`: 데이터를 받을 탭 이름 (예: 청구서모음) — 시트 하단 탭 이름과 정확히 일치해야 합니다.

---

## 참고

- 구글 시트 연동 실패 시에도 Supabase 저장은 정상 동작합니다.
- URL을 비워두면 구글 시트 연동을 사용하지 않습니다.
