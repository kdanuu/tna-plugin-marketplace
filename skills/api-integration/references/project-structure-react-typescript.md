# React/TypeScript 프로젝트 구조 가이드

## 표준 디렉토리 레이아웃

```
src/
├── api/                          # API 클라이언트
│   ├── [feature]Api.ts
│   ├── client.ts                 # Axios 인스턴스 설정
│   └── __tests__/
│       └── [feature]Api.test.ts
│
├── types/                        # TypeScript 타입 정의
│   ├── [feature].ts
│   └── common.ts
│
├── hooks/                        # 커스텀 Hooks
│   ├── use[Feature].ts
│   ├── use[Feature]Mutation.ts
│   └── __tests__/
│       └── use[Feature].test.ts
│
├── components/                   # 공통 컴포넌트
│   ├── common/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   └── Button.test.tsx
│   │   └── Input/
│   │       ├── Input.tsx
│   │       └── Input.test.tsx
│   └── [feature]/               # 기능별 컴포넌트
│       ├── [Component].tsx
│       └── [Component].test.tsx
│
├── utils/                        # 유틸리티 함수
│   ├── formatters.ts
│   └── validators.ts
│
└── lib/                          # 라이브러리 설정
    ├── queryClient.ts            # React Query 설정
    └── constants.ts
```

## 파일 배치 규칙

### API 클라이언트 (api/)
**위치**: `src/api/[feature]Api.ts`

**예시**:
- `paymentApi.ts` → `src/api/paymentApi.ts`
- `userApi.ts` → `src/api/userApi.ts`

### 타입 정의 (types/)
**위치**: `src/types/[feature].ts`

**예시**:
- `payment.ts` → `src/types/payment.ts`
- `user.ts` → `src/types/user.ts`

### 커스텀 Hooks (hooks/)
**위치**: `src/hooks/use[Feature].ts`

**예시**:
- `usePayment.ts` → `src/hooks/usePayment.ts`
- `usePayments.ts` → `src/hooks/usePayments.ts`
- `useCreatePayment.ts` → `src/hooks/useCreatePayment.ts` (Mutation)

### 컴포넌트 (components/)
**위치**: `src/components/[feature]/[Component].tsx`

**예시**:
- `PaymentForm.tsx` → `src/components/payment/PaymentForm.tsx`
- `PaymentList.tsx` → `src/components/payment/PaymentList.tsx`

## 명명 규칙

| 종류 | 패턴 | 예시 |
|-----|------|------|
| API 클라이언트 | `{feature}Api.ts` | `paymentApi.ts` |
| 타입 파일 | `{feature}.ts` | `payment.ts` |
| Query Hook | `use{Feature}.ts` | `usePayment.ts`, `usePayments.ts` |
| Mutation Hook | `use{Action}{Feature}.ts` | `useCreatePayment.ts` |
| 컴포넌트 | `{Feature}{Type}.tsx` | `PaymentForm.tsx` |
| 유틸리티 | `{기능명}.ts` | `formatters.ts` |

## 환경 설정 파일

### .env (React)
**위치**: `.env` 또는 `.env.local`

```bash
REACT_APP_API_URL=https://api.example.com
REACT_APP_TIMEOUT=30000
```

### .env.local (Next.js)
**위치**: `.env.local`

```bash
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_TIMEOUT=30000
```

## API 클라이언트 예시 구조

### client.ts (Axios 설정)
**위치**: `src/api/client.ts`

```typescript
import axios from 'axios'

export const apiClient = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// 인증 토큰 인터셉터
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('authToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

### [feature]Api.ts
**위치**: `src/api/paymentApi.ts`

```typescript
import { apiClient } from './client'
import { Payment, CreatePaymentRequest } from '@/types/payment'

export const paymentApi = {
  async createPayment(request: CreatePaymentRequest): Promise<Payment> {
    const response = await apiClient.post<Payment>('/api/v1/payments', request)
    return response.data
  },

  async getPayment(paymentId: string): Promise<Payment> {
    const response = await apiClient.get<Payment>(`/api/v1/payments/${paymentId}`)
    return response.data
  }
}
```

## React Query Hooks 구조

### usePayment.ts (Query)
**위치**: `src/hooks/usePayment.ts`

```typescript
import { useQuery } from '@tanstack/react-query'
import { paymentApi } from '@/api/paymentApi'

export function usePayment(paymentId: string) {
  return useQuery({
    queryKey: ['payment', paymentId],
    queryFn: () => paymentApi.getPayment(paymentId),
    enabled: !!paymentId
  })
}
```

### useCreatePayment.ts (Mutation)
**위치**: `src/hooks/useCreatePayment.ts`

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { paymentApi } from '@/api/paymentApi'

export function useCreatePayment() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: paymentApi.createPayment,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['payments'] })
      queryClient.setQueryData(['payment', data.id], data)
    }
  })
}
```

## 의존성 방향

```
[Components] ─────> [Hooks] ─────> [API] ─────> [Types]
                                      ↑
                                  [Client]
```
