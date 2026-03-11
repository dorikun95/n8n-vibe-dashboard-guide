# 🚀 n8n × 바이브 코딩 × Claude Code — 실시간 대시보드 구축 가이드

> **핵심 아이디어**: n8n이 데이터를 수집 → API로 노출 → Claude Code가 Next.js 대시보드 생성 → Vercel로 배포
> 코드 한 줄 안 짜도 됩니다. 말로 설명하면 Claude Code가 다 만들어줍니다.

---

## 🧠 전체 아키텍처 한눈에 보기

```
[데이터 소스들]
  소셜미디어 / RSS / API / DB
       ↓
   [n8n 워크플로우]
  수집 → 가공 → 저장
       ↓
  [Webhook / API 노출]
  n8n의 HTTP Response 노드
       ↓
  [Next.js 대시보드]  ← Claude Code가 만들어 줌
  실시간 차트 + 테이블
       ↓
   [Vercel 배포]
  전 세계 어디서나 접근
```

**왜 이 조합인가?**
| 도구 | 역할 | 핵심 장점 |
|------|------|----------|
| n8n | 데이터 파이프라인 | 노코드로 복잡한 수집 자동화 |
| Claude Code | 대시보드 개발 | 말로 설명하면 코드 완성 |
| Next.js | 프론트엔드 프레임워크 | API Route + 서버사이드 렌더링 내장 |
| Vercel | 배포 플랫폼 | GitHub push 만으로 자동 배포 |

---

## 1단계: n8n 워크플로우 — 데이터 수집 & API 만들기

### 1-1. 기본 워크플로우 패턴

```
[Webhook 트리거 또는 Cron 스케줄]
         ↓
   [데이터 수집 노드]
  (HTTP Request / RSS / DB 쿼리 등)
         ↓
   [데이터 가공 노드]
  (Function 노드로 JSON 정리)
         ↓
  [저장 노드 (선택)]
  (Google Sheets / DB / Airtable)
         ↓
  [HTTP Response 노드]
  → 대시보드가 여기서 데이터를 가져감
```

### 1-2. n8n Webhook으로 API 엔드포인트 만들기

**설정 방법:**
1. n8n에서 **Webhook 노드** 추가
2. HTTP Method: `GET`
3. Path: `/dashboard-data` (원하는 이름)
4. 마지막에 **Respond to Webhook 노드** 연결

**Function 노드에서 데이터 포맷 정리 (예시):**
```javascript
// n8n Function 노드 코드
const rawData = $input.all();

return [{
  json: {
    summary: {
      totalCount: rawData.length,
      lastUpdated: new Date().toISOString(),
    },
    // 차트용 시계열 데이터
    timeSeries: rawData.map(item => ({
      date: item.json.date,
      value: item.json.count,
      label: item.json.category
    })),
    // 테이블용 상세 데이터
    tableData: rawData.map(item => ({
      id: item.json.id,
      title: item.json.title,
      status: item.json.status,
      timestamp: item.json.timestamp
    }))
  }
}];
```

**최종 n8n Webhook URL 형태:**
```
https://your-n8n.com/webhook/dashboard-data
```

> 💡 **핵심 인사이트**: n8n의 Webhook 노드 하나가 곧 REST API입니다.
> 별도 백엔드 서버 없이 n8n 자체가 API 서버 역할을 합니다.

### 1-3. CORS 처리 (필수!)

n8n Webhook 노드의 **Response Headers** 설정:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST
Content-Type: application/json
```

---

## 2단계: Claude Code 설치 & 프로젝트 세팅

### 2-1. 환경 준비

```bash
# Node.js 설치 확인 (18 이상 필요)
node --version

# Claude Code 설치
npm install -g @anthropic/claude-code

# 새 Next.js 프로젝트 생성
npx create-next-app@latest my-dashboard \
  --typescript \
  --tailwind \
  --app \
  --no-src-dir

cd my-dashboard
```

### 2-2. Claude Code 실행

```bash
# 프로젝트 폴더 안에서
claude
```

터미널에 Claude Code 인터페이스가 열립니다. 이제 **말로** 지시하면 됩니다.

---

## 3단계: 바이브 코딩 — Claude Code에게 말로 시키기

### 🎯 바이브 코딩의 핵심 원칙

> **나쁜 프롬프트**: "대시보드 만들어줘"
> **좋은 프롬프트**: 목적 + 데이터 구조 + 원하는 시각화 + 기술 스택을 구체적으로

### 3-1. 초기 프롬프트 템플릿

Claude Code 터미널에 아래처럼 입력:

```
나는 n8n으로 수집한 데이터를 시각화하는 실시간 대시보드를 만들려고 해.

## 데이터 소스
- n8n Webhook URL: https://your-n8n.com/webhook/dashboard-data
- 응답 JSON 구조:
  {
    "summary": { "totalCount": 150, "lastUpdated": "2024-01-01T00:00:00Z" },
    "timeSeries": [{ "date": "2024-01-01", "value": 42, "label": "category" }],
    "tableData": [{ "id": 1, "title": "제목", "status": "active", "timestamp": "..." }]
  }

## 원하는 기능
1. 30초마다 자동 데이터 갱신 (실시간 느낌)
2. 상단: KPI 카드 3개 (총 건수, 오늘 건수, 증감률)
3. 중간: 라인 차트 (시계열 데이터 시각화)
4. 하단: 데이터 테이블 (검색, 정렬 기능 포함)
5. 마지막 업데이트 시간 표시

## 기술 요구사항
- Next.js App Router 사용
- Recharts 라이브러리로 차트 구현
- Tailwind CSS로 스타일링
- TypeScript 사용
- Vercel 배포 최적화

시작해줘.
```

### 3-2. 단계별 후속 프롬프트 예시

**레이아웃 수정:**
```
상단 KPI 카드를 더 눈에 띄게 만들어줘.
숫자가 크게 보이고, 증감은 초록/빨강 색상으로 표시해줘.
```

**에러 처리 추가:**
```
n8n API 호출 실패할 때 스켈레톤 로딩과 에러 메시지를 보여주는 
컴포넌트를 추가해줘.
```

**반응형 처리:**
```
모바일에서도 잘 보이게 반응형으로 수정해줘.
모바일은 카드를 세로로 쌓아줘.
```

**환경변수 처리:**
```
n8n Webhook URL을 환경변수 NEXT_PUBLIC_N8N_WEBHOOK_URL로 
분리해줘. .env.local 파일도 만들어줘.
```

---

## 4단계: 프로젝트 구조 (Claude Code가 만들어 줄 결과물)

```
my-dashboard/
├── app/
│   ├── layout.tsx          # 전체 레이아웃
│   ├── page.tsx            # 메인 대시보드 페이지
│   └── api/
│       └── data/
│           └── route.ts    # n8n 프록시 API (CORS 우회용)
├── components/
│   ├── KpiCard.tsx         # 지표 카드
│   ├── TimeSeriesChart.tsx # 라인 차트
│   ├── DataTable.tsx       # 데이터 테이블
│   └── LastUpdated.tsx     # 업데이트 시간 표시
├── hooks/
│   └── useDashboardData.ts # 데이터 페칭 + 자동 갱신 훅
├── types/
│   └── dashboard.ts        # TypeScript 타입 정의
├── .env.local              # 환경변수 (git 제외)
├── .env.example            # 환경변수 예시 (git 포함)
└── vercel.json             # Vercel 배포 설정
```

### 핵심 파일: 실시간 데이터 훅

```typescript
// hooks/useDashboardData.ts — Claude Code가 만들어 줄 코드 예시
import { useState, useEffect } from 'react';

export function useDashboardData(refreshInterval = 30000) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [lastUpdated, setLastUpdated] = useState(null);

  const fetchData = async () => {
    try {
      const res = await fetch('/api/data'); // Next.js API Route (n8n 프록시)
      if (!res.ok) throw new Error('데이터 로드 실패');
      const json = await res.json();
      setData(json);
      setLastUpdated(new Date());
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData(); // 최초 로드
    const interval = setInterval(fetchData, refreshInterval); // 자동 갱신
    return () => clearInterval(interval);
  }, [refreshInterval]);

  return { data, loading, error, lastUpdated, refresh: fetchData };
}
```

> 💡 **핵심 인사이트**: Next.js API Route를 중간에 두는 이유는
> n8n Webhook URL을 클라이언트에 노출하지 않기 위해서입니다.
> 보안 + CORS 문제를 한 번에 해결합니다.

---

## 5단계: Vercel 배포

### 5-1. GitHub 연동 배포 (권장)

```bash
# Git 초기화 및 GitHub 업로드
git init
git add .
git commit -m "초기 대시보드 구성"
git remote add origin https://github.com/your-id/my-dashboard.git
git push -u origin main
```

1. [vercel.com](https://vercel.com) 로그인
2. **Add New Project** → GitHub 레포 선택
3. Framework: **Next.js** (자동 감지됨)
4. **Environment Variables** 추가:
   ```
   N8N_WEBHOOK_URL = https://your-n8n.com/webhook/dashboard-data
   ```
5. **Deploy** 클릭 → 1-2분 후 완료

### 5-2. Vercel CLI 배포 (빠른 방법)

```bash
# Vercel CLI 설치
npm install -g vercel

# 배포 (터미널에서 질문에 답변)
vercel

# 환경변수 추가
vercel env add N8N_WEBHOOK_URL

# 프로덕션 배포
vercel --prod
```

### 5-3. vercel.json 설정 (선택)

```json
{
  "functions": {
    "app/api/**": {
      "maxDuration": 10
    }
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "no-store" }
      ]
    }
  ]
}
```

---

## 6단계: n8n과 Vercel 연동 — 실시간성 강화

### 방법 A: 폴링 방식 (기본, 구현 쉬움)
```
Vercel 대시보드 → 30초마다 → n8n Webhook 호출
```
- 장점: 구현 단순, 안정적
- 단점: 실시간이 아닌 근사 실시간 (최대 30초 지연)

### 방법 B: n8n이 Push하는 방식 (진짜 실시간)
```
n8n 워크플로우 완료 → Vercel API Route에 POST → 
Vercel이 데이터 캐시 업데이트 → 브라우저 즉시 반영
```

**구현 흐름:**
1. n8n 워크플로우 마지막에 **HTTP Request 노드** 추가
2. URL: `https://your-dashboard.vercel.app/api/webhook`
3. Method: `POST`
4. Body: 새 데이터 JSON

```typescript
// app/api/webhook/route.ts — n8n이 데이터를 밀어넣는 엔드포인트
import { NextRequest, NextResponse } from 'next/server';

// 간단한 인메모리 캐시 (실제 서비스는 Redis/KV 사용)
let cache: any = null;
let cacheTime: number = 0;

export async function POST(req: NextRequest) {
  // n8n 시크릿 검증
  const secret = req.headers.get('x-webhook-secret');
  if (secret !== process.env.WEBHOOK_SECRET) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  cache = await req.json();
  cacheTime = Date.now();
  
  return NextResponse.json({ ok: true });
}

export async function GET() {
  return NextResponse.json(cache || { empty: true });
}
```

> 💡 **핵심 인사이트**: 방법 B는 n8n → Vercel 방향으로 데이터를 "밀어넣기"하는 방식.
> n8n이 데이터를 수집한 즉시 대시보드에 반영되므로 진짜 실시간입니다.

---

## 7단계: 실전 바이브 코딩 팁

### ✅ Claude Code와 효과적으로 대화하는 법

| 상황 | 이렇게 말하세요 |
|------|---------------|
| 컴포넌트 추가 | "KPI 카드 위에 날짜 필터 드롭다운 추가해줘" |
| 버그 수정 | "차트가 모바일에서 잘려 보여. 수정해줘" |
| 스타일 변경 | "전체 테마를 다크 모드로 바꿔줘" |
| 기능 확장 | "CSV 다운로드 버튼 추가해줘" |
| 최적화 | "API 호출 결과를 5분 캐시해줘" |

### ✅ 자주 쓰는 Claude Code 명령어

```bash
# 특정 파일 수정 요청
claude "components/KpiCard.tsx 파일에 툴팁 추가해줘"

# 전체 프로젝트 컨텍스트로 질문
claude "현재 프로젝트 구조 확인하고 TypeScript 에러 있으면 다 고쳐줘"

# 파일 생성 요청
claude "다크모드 토글 컴포넌트 만들어서 헤더에 추가해줘"
```

### ✅ Vercel 배포 전 체크리스트

```bash
# 빌드 에러 확인
npm run build

# 환경변수 확인
cat .env.local

# .gitignore에 민감 정보 포함 확인
cat .gitignore | grep ".env"

# 타입 체크
npx tsc --noEmit
```

---

## 전체 요약 — 한 장으로 보기

```
1️⃣  n8n 워크플로우 구성
    └─ Webhook 노드로 GET API 만들기
    └─ 데이터 JSON 포맷 통일

2️⃣  Claude Code로 대시보드 생성
    └─ "말로" 요구사항 설명
    └─ Next.js + Recharts + Tailwind

3️⃣  로컬 테스트
    └─ npm run dev
    └─ n8n Webhook 연결 확인

4️⃣  Vercel 배포
    └─ GitHub push → 자동 배포
    └─ 환경변수 설정

5️⃣  실시간 확인
    └─ 30초 폴링 또는 n8n Push 방식
```

**예상 소요 시간**: 처음 해보는 경우 2~3시간, 익숙해지면 30분

---

## 추천 라이브러리

```bash
# 차트
npm install recharts

# 날짜 처리
npm install date-fns

# 테이블 (고급 정렬/필터 필요시)
npm install @tanstack/react-table

# 로딩 스켈레톤
npm install react-loading-skeleton

# 실시간 (WebSocket 필요시)
npm install socket.io-client
```

---

> **마지막 핵심 인사이트** 🎯
>
> 바이브 코딩의 본질은 "내가 원하는 것"을 명확히 말하는 것입니다.
> Claude Code는 코드를 짜고, n8n은 데이터를 모으고, Vercel은 배포합니다.
> 당신은 **아이디어와 방향**만 제시하면 됩니다.
> 이 스택의 핵심 가치는 **빠른 실험 → 빠른 배포 → 빠른 피드백** 사이클입니다.
