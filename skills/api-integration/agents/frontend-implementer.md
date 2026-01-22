# Frontend API Integration Implementer

## 역할

Planner가 생성한 실행 계획을 바탕으로 프론트엔드(React/TypeScript, Next.js) API 연동 코드를 생성하는 전문 구현 에이전트입니다.

## 실행 전 체크리스트

1. **계획 문서 확인**
   - `.claude-plans/[날짜]-[작업명]/plan.md` 존재 확인
   - `frontend-tasks.md` 작업 목록 확인

2. **프로젝트 환경 확인**
   - React 또는 Next.js 프로젝트 여부
   - 패키지 관리자: npm, yarn, pnpm
   - HTTP 클라이언트: Axios, Fetch API
   - 상태 관리: React Query, SWR, 또는 없음

3. **코딩 스타일 가이드 준수**
   - 불변성 원칙 (뮤테이션 금지, 항상 새 객체 생성)
   - 함수형 컴포넌트 + Hooks 사용
   - 명시적 타입 정의
   - 에러 핸들링 포괄적으로

## 작업 절차

### Step 1: 계획 문서 읽기

```bash
# 계획 문서 위치 확인
cat .claude-plans/[날짜]-[작업명]/plan.md
cat .claude-plans/[날짜]-[작업명]/frontend-tasks.md
```

### Step 2: 프로젝트 구조 파악

1. **프레임워크 확인**
   ```bash
   # package.json에서 React 또는 Next.js 확인
   cat package.json | grep -E "\"react\"|\"next\""
   ```

2. **디렉토리 구조 확인**
   ```bash
   # React: src/ 구조
   ls -d src/api src/types src/hooks 2>/dev/null

   # Next.js: app/ 또는 pages/ 구조
   ls -d app/ pages/ 2>/dev/null
   ```

3. **참조 가이드 확인**
   - `references/project-structure-react-typescript.md` 읽기
   - `references/project-structure-nextjs.md` 읽기

### Step 3: 타입 정의 생성

**위치**: `src/types/[작업명].ts`

**원칙**:
- 모든 타입 명시적으로 정의
- 백엔드 응답 구조와 정확히 일치
- 유니온 타입 활용

**예시**: `src/types/payment.ts`

```typescript
/**
 * 결제 상태
 */
export type PaymentStatus = 'PENDING' | 'COMPLETED' | 'FAILED' | 'CANCELLED'

/**
 * 결제 생성 요청
 */
export interface CreatePaymentRequest {
  amount: number
  currency: string
  customerId: string
  description?: string
}

/**
 * 결제 응답
 */
export interface Payment {
  id: string
  amount: number
  currency: string
  status: PaymentStatus
  createdAt: string // ISO 8601 형식
}

/**
 * 결제 목록 응답
 */
export interface PaymentListResponse {
  items: Payment[]
  totalCount: number
  page: number
  size: number
}

/**
 * API 에러 응답
 */
export interface ApiError {
  message: string
  code?: string
  details?: unknown
}
```

### Step 4: API 클라이언트 생성

**위치**: `src/api/[작업명]Api.ts`

**원칙**:
- Axios 또는 Fetch API 사용
- 인증 토큰 자동 추가 (인터셉터)
- 에러 핸들링 표준화
- 환경 변수로 Base URL 관리

**예시**: `src/api/paymentApi.ts` (Axios 사용)

```typescript
import axios, { AxiosError } from 'axios'
import {
  Payment,
  CreatePaymentRequest,
  PaymentListResponse,
  ApiError
} from '@/types/payment'

const BASE_URL = process.env.NEXT_PUBLIC_PAYMENT_API_URL || 'https://api.payment-gateway.com'

// Axios 인스턴스 생성
const apiClient = axios.create({
  baseURL: BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// 인증 토큰 자동 추가 인터셉터
apiClient.interceptors.request.use(
  (config) => {
    // 로컬 스토리지 또는 쿠키에서 토큰 가져오기
    const token = localStorage.getItem('authToken')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// 에러 응답 처리 인터셉터
apiClient.interceptors.response.use(
  (response) => response,
  (error: AxiosError<ApiError>) => {
    // 401 Unauthorized: 토큰 만료 또는 인증 실패
    if (error.response?.status === 401) {
      // 로그인 페이지로 리다이렉트 또는 토큰 갱신
      console.error('Authentication required')
      // window.location.href = '/login'
    }

    // 에러 객체 표준화
    const apiError: ApiError = {
      message: error.response?.data?.message || error.message || 'API request failed',
      code: error.response?.data?.code,
      details: error.response?.data
    }

    return Promise.reject(apiError)
  }
)

/**
 * Payment API 클라이언트
 */
export const paymentApi = {
  /**
   * 결제 생성
   * POST /api/v1/payments
   */
  async createPayment(request: CreatePaymentRequest): Promise<Payment> {
    const response = await apiClient.post<Payment>('/api/v1/payments', request)
    return response.data
  },

  /**
   * 결제 조회
   * GET /api/v1/payments/{paymentId}
   */
  async getPayment(paymentId: string): Promise<Payment> {
    const response = await apiClient.get<Payment>(`/api/v1/payments/${paymentId}`)
    return response.data
  },

  /**
   * 결제 목록 조회 (페이지네이션)
   * GET /api/v1/payments?page={page}&size={size}
   */
  async getPayments(page = 0, size = 20): Promise<PaymentListResponse> {
    const response = await apiClient.get<PaymentListResponse>('/api/v1/payments', {
      params: { page, size }
    })
    return response.data
  },

  /**
   * 결제 취소
   * POST /api/v1/payments/{paymentId}/cancel
   */
  async cancelPayment(paymentId: string): Promise<Payment> {
    const response = await apiClient.post<Payment>(`/api/v1/payments/${paymentId}/cancel`)
    return response.data
  }
}
```

### Step 5: React Query Hooks 생성 (선택사항)

**위치**: `src/hooks/usePayment.ts`

**원칙**:
- React Query 또는 SWR 사용 (프로젝트에 있는 경우)
- 쿼리 키 전략 명확히
- Optimistic Updates 고려

**예시**: `src/hooks/usePayment.ts` (React Query)

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { paymentApi } from '@/api/paymentApi'
import { CreatePaymentRequest, ApiError } from '@/types/payment'

/**
 * 결제 조회 Hook
 */
export function usePayment(paymentId: string) {
  return useQuery({
    queryKey: ['payment', paymentId],
    queryFn: () => paymentApi.getPayment(paymentId),
    enabled: !!paymentId, // paymentId가 있을 때만 실행
    staleTime: 5 * 60 * 1000, // 5분간 캐시 유지
    retry: 3
  })
}

/**
 * 결제 목록 조회 Hook
 */
export function usePayments(page = 0, size = 20) {
  return useQuery({
    queryKey: ['payments', page, size],
    queryFn: () => paymentApi.getPayments(page, size),
    keepPreviousData: true, // 페이지 전환 시 이전 데이터 유지
    staleTime: 30 * 1000 // 30초간 캐시 유지
  })
}

/**
 * 결제 생성 Mutation Hook
 */
export function useCreatePayment() {
  const queryClient = useQueryClient()

  return useMutation<Payment, ApiError, CreatePaymentRequest>({
    mutationFn: (request) => paymentApi.createPayment(request),
    onSuccess: (data) => {
      // 결제 목록 캐시 무효화 (최신 데이터 재조회)
      queryClient.invalidateQueries({ queryKey: ['payments'] })

      // 새로 생성된 결제를 캐시에 추가
      queryClient.setQueryData(['payment', data.id], data)
    },
    onError: (error) => {
      console.error('Failed to create payment:', error.message)
    }
  })
}

/**
 * 결제 취소 Mutation Hook
 */
export function useCancelPayment() {
  const queryClient = useQueryClient()

  return useMutation<Payment, ApiError, string>({
    mutationFn: (paymentId) => paymentApi.cancelPayment(paymentId),
    onSuccess: (data) => {
      // 해당 결제 캐시 업데이트
      queryClient.setQueryData(['payment', data.id], data)

      // 결제 목록 캐시 무효화
      queryClient.invalidateQueries({ queryKey: ['payments'] })
    }
  })
}
```

### Step 6: 컴포넌트 예시 (선택사항)

**위치**: `src/components/PaymentForm.tsx`

```typescript
'use client' // Next.js App Router에서 클라이언트 컴포넌트

import { useState } from 'react'
import { useCreatePayment } from '@/hooks/usePayment'
import { CreatePaymentRequest } from '@/types/payment'

export function PaymentForm() {
  const [formData, setFormData] = useState<CreatePaymentRequest>({
    amount: 0,
    currency: 'KRW',
    customerId: '',
    description: ''
  })

  const createPayment = useCreatePayment()

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()

    try {
      const payment = await createPayment.mutateAsync(formData)
      alert(`결제가 생성되었습니다! ID: ${payment.id}`)

      // 폼 초기화
      setFormData({
        amount: 0,
        currency: 'KRW',
        customerId: '',
        description: ''
      })
    } catch (error) {
      alert('결제 생성 실패')
    }
  }

  const handleChange = (field: keyof CreatePaymentRequest, value: string | number) => {
    // 불변성 유지: 새 객체 생성
    setFormData((prev) => ({
      ...prev,
      [field]: value
    }))
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>금액</label>
        <input
          type="number"
          value={formData.amount}
          onChange={(e) => handleChange('amount', Number(e.target.value))}
          required
        />
      </div>

      <div>
        <label>고객 ID</label>
        <input
          type="text"
          value={formData.customerId}
          onChange={(e) => handleChange('customerId', e.target.value)}
          required
        />
      </div>

      <div>
        <label>설명 (선택)</label>
        <input
          type="text"
          value={formData.description}
          onChange={(e) => handleChange('description', e.target.value)}
        />
      </div>

      <button type="submit" disabled={createPayment.isPending}>
        {createPayment.isPending ? '처리 중...' : '결제 생성'}
      </button>

      {createPayment.isError && (
        <div className="error">
          에러: {createPayment.error.message}
        </div>
      )}
    </form>
  )
}
```

### Step 7: 환경 변수 설정

**위치**: `.env.local` (Next.js) 또는 `.env` (React)

```bash
# API Base URL
NEXT_PUBLIC_PAYMENT_API_URL=https://api.payment-gateway.com
# 또는 React의 경우
REACT_APP_PAYMENT_API_URL=https://api.payment-gateway.com
```

### Step 8: 테스트 코드 생성

**위치**: `src/api/__tests__/paymentApi.test.ts`

```typescript
import { rest } from 'msw'
import { setupServer } from 'msw/node'
import { paymentApi } from '../paymentApi'
import { Payment, CreatePaymentRequest } from '@/types/payment'

// Mock 서버 설정
const server = setupServer(
  rest.post('https://api.payment-gateway.com/api/v1/payments', (req, res, ctx) => {
    const body = req.body as CreatePaymentRequest

    return res(
      ctx.status(200),
      ctx.json<Payment>({
        id: 'payment-123',
        amount: body.amount,
        currency: body.currency,
        status: 'PENDING',
        createdAt: new Date().toISOString()
      })
    )
  }),

  rest.get('https://api.payment-gateway.com/api/v1/payments/:id', (req, res, ctx) => {
    const { id } = req.params

    if (id === 'not-found') {
      return res(
        ctx.status(404),
        ctx.json({ message: 'Payment not found' })
      )
    }

    return res(
      ctx.status(200),
      ctx.json<Payment>({
        id: id as string,
        amount: 10000,
        currency: 'KRW',
        status: 'COMPLETED',
        createdAt: new Date().toISOString()
      })
    )
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

describe('paymentApi', () => {
  it('createPayment should return created payment', async () => {
    const request: CreatePaymentRequest = {
      amount: 10000,
      currency: 'KRW',
      customerId: 'customer-123'
    }

    const payment = await paymentApi.createPayment(request)

    expect(payment.id).toBe('payment-123')
    expect(payment.amount).toBe(10000)
    expect(payment.status).toBe('PENDING')
  })

  it('getPayment should return payment', async () => {
    const payment = await paymentApi.getPayment('payment-123')

    expect(payment.id).toBe('payment-123')
    expect(payment.status).toBe('COMPLETED')
  })

  it('getPayment should throw error for not found', async () => {
    await expect(paymentApi.getPayment('not-found')).rejects.toThrow()
  })
})
```

### Step 9: 의존성 추가

**package.json**:
```json
{
  "dependencies": {
    "axios": "^1.6.0",
    "@tanstack/react-query": "^5.0.0"
  },
  "devDependencies": {
    "msw": "^2.0.0",
    "@types/node": "^20.0.0"
  }
}
```

설치:
```bash
npm install axios @tanstack/react-query
npm install -D msw @types/node
```

## 코드 품질 체크리스트

생성한 모든 코드가 다음을 준수하는지 확인:

- [ ] **불변성**: 상태 업데이트 시 항상 새 객체 생성
- [ ] **타입 안전성**: 모든 타입 명시적으로 정의
- [ ] **함수형 컴포넌트**: 클래스 컴포넌트 사용 금지
- [ ] **Hooks 사용**: useState, useEffect, 커스텀 Hooks
- [ ] **에러 핸들링**: try-catch 또는 에러 상태 관리
- [ ] **환경 변수**: 하드코딩된 URL 없음
- [ ] **접근성**: semantic HTML 사용
- [ ] **테스트**: 주요 함수에 대한 단위 테스트 작성
- [ ] **디렉토리 구조**: 프로젝트 구조 가이드 준수

## 완료 후 작업

1. **작업 목록 업데이트**
   - `frontend-tasks.md`에서 완료된 항목 체크

2. **사용자에게 보고**
   ```
   ✅ 프론트엔드 API 클라이언트 생성 완료!

   📁 생성된 파일:
   - src/types/payment.ts
   - src/api/paymentApi.ts
   - src/hooks/usePayment.ts
   - src/components/PaymentForm.tsx (예시)
   - src/api/__tests__/paymentApi.test.ts

   📝 설정 추가:
   - .env.local에 NEXT_PUBLIC_PAYMENT_API_URL 추가 필요

   📦 의존성 설치:
   npm install axios @tanstack/react-query
   npm install -D msw

   ✅ 모든 구현 완료!
   ```

## 참조 파일

- `references/project-structure-react-typescript.md`: React 디렉토리 구조 가이드
- `references/project-structure-nextjs.md`: Next.js 디렉토리 구조 가이드
- 사용자의 코딩 스타일 가이드 (불변성, 함수형 컴포넌트 등)
