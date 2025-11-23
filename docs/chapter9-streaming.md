# Chapter 9: Streaming


- **Streaming이란?**
  - 한 페이지를 여러 청크로 나눠, 준비되는 대로 서버→클라이언트로 점진 전송하여 초기 체감 속도를 높이는 기법.
  - 느린 데이터 요청이 전체 페이지 렌더링을 막지 않도록 해 일부 UI를 먼저 보여준다.

- **적용 방법**
  - 페이지 단위: `loading.tsx` 사용(Next.js가 자동으로 `<Suspense>` 경계 생성).
  - 컴포넌트 단위: React `<Suspense>`로 개별/그룹 단위 제어.

- **로딩 스켈레톤**
  - `loading.tsx`나 `<Suspense fallback>`에 골격 UI를 배치해 사용자에게 진행 중임을 명확히 전달.

- **Route Groups**
  - `/(overview)`처럼 괄호 폴더로 URL에 영향 없이 파일을 묶는다.
  - 특정 섹션만 별도의 `loading.tsx`가 적용되도록 스코프를 분리.

## 실전 예시

- 페이지 스트리밍 시작하기: `/app/dashboard/loading.tsx`

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

- 스켈레톤으로 교체:

```tsx
// app/dashboard/loading.tsx
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  return <DashboardSkeleton />;
}
```

- Route Group으로 로딩 스코프 좁히기:
  - `app/dashboard/(overview)/page.tsx`와 `app/dashboard/(overview)/loading.tsx`로 이동하면, 개요 페이지에만 로딩 UI가 적용.

- 특정 컴포넌트만 스트리밍(느린 fetch를 컴포넌트로 이동):

```tsx
// app/dashboard/(overview)/page.tsx
import { Suspense } from 'react';
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
import { fetchLatestInvoices, fetchCardData } from '@/app/lib/data';
import { RevenueChartSkeleton, LatestInvoicesSkeleton, CardsSkeleton } from '@/app/ui/skeletons';

export default async function Page() {
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();

  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>Dashboard</h1>

      {/* 카드 그룹은 한 번에 나타나도록 그룹 경계 */}
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <Suspense fallback={<CardsSkeleton />}>
          <>
            <Card title="Collected" value={totalPaidInvoices} type="collected" />
            <Card title="Pending" value={totalPendingInvoices} type="pending" />
            <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
            <Card title="Total Customers" value={numberOfCustomers} type="customers" />
          </>
        </Suspense>
      </div>

      {/* RevenueChart만 개별 스트리밍 */}
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </Suspense>
        <Suspense fallback={<LatestInvoicesSkeleton />}>
          <LatestInvoices latestInvoices={latestInvoices} />
        </Suspense>
      </div>
    </main>
  );
}
```

```tsx
// app/ui/dashboard/revenue-chart.tsx
import { generateYAxis } from '@/app/lib/utils';
import { CalendarIcon } from '@heroicons/react/24/outline';
import { lusitana } from '@/app/ui/fonts';
import { fetchRevenue } from '@/app/lib/data';

export default async function RevenueChart() {
  const revenue = await fetchRevenue(); // 느린 요청을 컴포넌트 내부로 이동
  const chartHeight = 350;
  const { yAxisLabels, topLabel } = generateYAxis(revenue);

  if (!revenue || revenue.length === 0) {
    return <p className="mt-4 text-gray-400">No data available.</p>;
  }

  return (
    // ...차트 렌더링
    <div className="rounded-lg border p-4">...</div>
  );
}
```

## Suspense 경계 배치 가이드

- 전체 페이지 스트리밍: 가장 간단하지만 느린 컴포넌트 하나가 전체 초기 표시를 지연시킬 수 있음.
- 모든 컴포넌트 개별 스트리밍: 과도하면 UI가 튀는(popping) 체감. 스켈레톤/그룹화로 완화.
- 섹션 단위 스트리밍(권장): 사용자에게 우선 노출할 영역을 먼저 보여주고, 느린 영역은 별도 경계로 지연.
- 원칙: 데이터가 필요한 컴포넌트 가까이로 fetch를 내리고, 해당 컴포넌트를 `<Suspense>`로 감싸라.

## 참고 링크

- [Next.js 공식 문서 - Streaming](https://nextjs.org/learn/dashboard-app/streaming)


