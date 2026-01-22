# Next.js 프로젝트 구조 가이드

## App Router 디렉토리 레이아웃

```
src/
├── app/                          # Next.js App Router
│   ├── api/                      # API Routes
│   │   └── [feature]/
│   │       └── route.ts
│   ├── [feature]/                # 기능별 페이지
│   │   ├── page.tsx              # 서버 컴포넌트
│   │   └── loading.tsx
│   └── layout.tsx
│
├── lib/                          # 라이브러리 설정
│   ├── api/                      # API 클라이언트
│   │   ├── [feature]Api.ts
│   │   └── client.ts
│   ├── types/                    # TypeScript 타입
│   │   └── [feature].ts
│   └── utils/                    # 유틸리티
│       └── formatters.ts
│
└── components/                   # 컴포넌트
    ├── ui/                       # 공통 UI 컴포넌트
    │   ├── button.tsx
    │   └── input.tsx
    └── [feature]/                # 기능별 컴포넌트
        ├── [component].tsx
        └── [component].test.tsx
```

## 파일 배치 규칙

### API Routes (app/api/)
**위치**: `src/app/api/[feature]/route.ts`

**예시**:
- `src/app/api/payments/route.ts` → GET/POST /api/payments
- `src/app/api/payments/[id]/route.ts` → GET/PUT/DELETE /api/payments/[id]

### 페이지 (app/)
**위치**: `src/app/[feature]/page.tsx`

**예시**:
- `src/app/payments/page.tsx` → /payments 페이지
- `src/app/payments/[id]/page.tsx` → /payments/[id] 상세 페이지

### API 클라이언트 (lib/api/)
**위치**: `src/lib/api/[feature]Api.ts`

**예시**:
- `paymentApi.ts` → `src/lib/api/paymentApi.ts`

### 타입 정의 (lib/types/)
**위치**: `src/lib/types/[feature].ts`

**예시**:
- `payment.ts` → `src/lib/types/payment.ts`

### 클라이언트 컴포넌트 (components/)
**위치**: `src/components/[feature]/[Component].tsx`

**예시**:
- `PaymentForm.tsx` → `src/components/payment/PaymentForm.tsx`

## 서버 vs 클라이언트 컴포넌트

### 서버 컴포넌트 (기본값)
**위치**: `src/app/[feature]/page.tsx`

```typescript
// 서버 컴포넌트 (최상단에 'use client' 없음)
import { paymentApi } from '@/lib/api/paymentApi'

export default async function PaymentsPage() {
  const payments = await paymentApi.getPayments()

  return <div>{/* ... */}</div>
}
```

### 클라이언트 컴포넌트
**위치**: `src/components/payment/PaymentForm.tsx`

```typescript
'use client' // 클라이언트 컴포넌트 표시

import { useState } from 'react'

export function PaymentForm() {
  const [data, setData] = useState({})

  return <form>{/* ... */}</form>
}
```

## 명명 규칙

| 종류 | 패턴 | 예시 |
|-----|------|------|
| API Route | `route.ts` | `app/api/payments/route.ts` |
| 페이지 | `page.tsx` | `app/payments/page.tsx` |
| 레이아웃 | `layout.tsx` | `app/layout.tsx` |
| 로딩 | `loading.tsx` | `app/payments/loading.tsx` |
| 에러 | `error.tsx` | `app/payments/error.tsx` |
| API 클라이언트 | `{feature}Api.ts` | `lib/api/paymentApi.ts` |
| 타입 | `{feature}.ts` | `lib/types/payment.ts` |

## 환경 설정 파일

### .env.local
**위치**: `.env.local`

```bash
# 서버 사이드에서만 접근 가능
DATABASE_URL=postgresql://...
API_SECRET_KEY=...

# 클라이언트에서도 접근 가능 (NEXT_PUBLIC_ 접두사)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_TIMEOUT=30000
```

## API Client 구조 (서버 & 클라이언트 공용)

### client.ts
**위치**: `src/lib/api/client.ts`

```typescript
import axios from 'axios'

// 서버 사이드용 (환경 변수 직접 접근)
export const serverApiClient = axios.create({
  baseURL: process.env.API_URL || 'https://api.example.com',
  timeout: 30000
})

// 클라이언트 사이드용 (NEXT_PUBLIC_)
export const clientApiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 30000
})
```

### paymentApi.ts
**위치**: `src/lib/api/paymentApi.ts`

```typescript
import { serverApiClient, clientApiClient } from './client'
import { Payment } from '@/lib/types/payment'

// 서버/클라이언트 자동 감지
const apiClient = typeof window === 'undefined' ? serverApiClient : clientApiClient

export const paymentApi = {
  async getPayments(): Promise<Payment[]> {
    const response = await apiClient.get<Payment[]>('/api/v1/payments')
    return response.data
  }
}
```

## API Route 예시

### route.ts (GET)
**위치**: `src/app/api/payments/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { paymentApi } from '@/lib/api/paymentApi'

export async function GET(request: NextRequest) {
  try {
    const payments = await paymentApi.getPayments()
    return NextResponse.json(payments)
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch payments' },
      { status: 500 }
    )
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const payment = await paymentApi.createPayment(body)
    return NextResponse.json(payment, { status: 201 })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create payment' },
      { status: 500 }
    )
  }
}
```

## 의존성 방향

```
[App Router] ────> [Lib/API] ────> [Types]
     ↑                 ↑
[Components] ────> [Client]
```

## 중요 사항

1. **서버 컴포넌트 우선**: 가능한 서버 컴포넌트 사용 (데이터 페칭 성능)
2. **클라이언트 컴포넌트**: 상태, 이벤트, Hooks 필요 시에만
3. **환경 변수**: 클라이언트 노출 시 `NEXT_PUBLIC_` 접두사 필수
4. **API Routes**: 외부 API 프록시 또는 서버 로직용
