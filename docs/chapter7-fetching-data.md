# Chapter 7: Fetching Data

- **데이터 패칭 접근 선택지**
  - API 레이어: 클라이언트에서 데이터를 가져와야 하거나 서드파티 API 사용 시 유용하다. 서버에서 실행되어 DB 시크릿을 노출하지 않는다.
  - DB 쿼리: 서버에서 직접 SQL 또는 ORM으로 질의한다. App Router의 Server Components와 궁합이 좋다.

- **Server Components로 안전하게 패칭**
  - 서버에서 실행되므로 `useEffect` 없이 `async/await`로 데이터를 가져올 수 있다.
  - 비싼 연산·DB 접근을 서버에 유지하고, 결과만 클라이언트로 전송해 보안·성능을 개선한다.
  - 별도 API 계층 없이 DB를 직접 질의할 수 있어 코드량이 줄어든다.

- **Using SQL (postgres.js)**
  - SQL은 관계형 DB의 표준이며, ORMs도 내부적으로 SQL을 생성한다.
  - SQL을 이해하면 RDB의 기본기를 익힐 수 있고, 다양한 도구로 지식을 이식하기 쉽다.
  - `postgres.js`는 SQL 인젝션 방어를 제공한다.

    ```ts
    // app/lib/data.ts (예시)
    import postgres from 'postgres';
    const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
    
    // 예: 최근 5개의 송장과 고객 정보
    const data = await sql`
      SELECT invoices.amount, customers.name, customers.image_url, customers.email
      FROM invoices
      JOIN customers ON invoices.customer_id = customers.id
      ORDER BY invoices.date DESC
      LIMIT 5
    `;
    ```

- **대시보드용 데이터 패칭 패턴**
  - 페이지 컴포넌트를 `async` 서버 컴포넌트로 선언해 직접 데이터 패칭:

    ```tsx
    // app/dashboard/page.tsx
    export default async function Page() {
      // const revenue = await fetchRevenue();
      // const latestInvoices = await fetchLatestInvoices();
      return <main>{/* 컴포넌트에 데이터 전달 */}</main>;
    }
    ```

- **요청 워터폴 vs 병렬 패칭**
  - 워터폴(순차 실행)은 각 요청이 이전 요청 완료를 기다려 전체 시간이 늘어난다.
  - 병렬 패칭: 모든 요청을 동시에 시작해 총 소요 시간을 단축한다.

    ```ts
    // 병렬 실행 패턴
    const revenuePromise = fetchRevenue();
    const latestInvoicesPromise = fetchLatestInvoices();
    const cardDataPromise = fetchCardData();
    
    const [revenue, latestInvoices, { numberOfInvoices, numberOfCustomers, totalPaidInvoices, totalPendingInvoices }]
      = await Promise.all([revenuePromise, latestInvoicesPromise, cardDataPromise]);
    ```

---

## API 레이어: 설명과 프로젝트 예시

- **API 레이어란?**
  - 브라우저에서 호출 가능한 서버 엔드포인트(Route Handler)이다. JSON 응답을 반환하며, 클라이언트에서 DB 시크릿을 직접 다루지 않게 해준다.
  - 예시 1: 시드 API `GET /seed` — DB 테이블 생성 후 메시지를 JSON으로 반환.
  
    ```ts
    // 경로: app/seed/route.ts
    export async function GET() {
      try {
        await sql.begin((sql) => [seedUsers(), seedCustomers(), seedInvoices(), seedRevenue()]);
        return Response.json({ message: 'Database seeded successfully' });
      } catch (error) {
        return Response.json({ error }, { status: 500 });
      }
    }
    ```
  
  - 예시 2: 쿼리 API `GET /query` — SQL을 실행하고 결과를 JSON으로 반환.
  
    ```ts
    // app/query/route.ts
    async function listInvoices() {
      const data = await sql`
        SELECT invoices.amount, customers.name
        FROM invoices
        JOIN customers ON invoices.customer_id = customers.id
        WHERE invoices.amount = 666;
      `;
      return data;
    }
    
    export async function GET() {
      try {
        return Response.json(await listInvoices());
      } catch (error) {
        return Response.json({ error }, { status: 500 });
      }
    }
    ```

## DB 쿼리: 설명과 프로젝트 예시

- **DB 쿼리란?**
  - 서버에서 `postgres.js`의 `sql` 템플릿으로 DB에 직접 질의하는 코드이다. 이 프로젝트는 `app/lib/data.ts`에 모아 두었다.
  
    ```ts
    // app/lib/data.ts (발췌)
    const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
    
    export async function fetchRevenue() {
      const data = await sql`SELECT * FROM revenue`;
      return data;
    }
    ```

---

## 요청 워터폴: 설명과 사용 시점

- **의존성이 있을 때**: 이전 요청 결과가 다음 요청의 입력이 될 경우 워터폴(순차 실행)이 필수.
  - 예: 사용자 ID를 먼저 받아온 뒤(요청 1) 해당 ID로 친구 목록을 조회(요청 2).
  - 예: 액세스 토큰을 발급받고(요청 1) 그 토큰으로 보호된 API 호출(요청 2).
- **이 프로젝트에서의 예시(순차 실행 패턴)**:
  - 현재 `app/dashboard/page.tsx`는 아래처럼 `await`를 순차로 사용해 워터폴이 발생할 수 있다.
  
    ```ts
    // app/dashboard/page.tsx
    const revenue = await fetchRevenue();
    const latestInvoices = await fetchLatestInvoices();
    const { totalPaidInvoices, totalPendingInvoices, numberOfInvoices, numberOfCustomers } = await fetchCardData();
    ```

---

## 병렬 패칭: 설명과 프로젝트 예시

- **이미 사용 중인 곳**: `app/lib/data.ts`의 `fetchCardData()`는 `Promise.all()`로 서로 독립적인 쿼리를 병렬로 수행한다.
  
  ```ts
  // app/lib/data.ts
  export async function fetchCardData() {
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`
      SELECT
        SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
        SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
      FROM invoices
    `;
  
    const [invoiceCount, customerCount, status] = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
  
    // ...후처리
  }
  ```

- **페이지 레벨 예시(권장 패턴)**: 서로 독립적인 요청은 동시에 시작한 뒤 `Promise.all`로 합친다.
  
  ```ts
  // app/dashboard/page.tsx (예시)
  const revenuePromise = fetchRevenue();
  const latestInvoicesPromise = fetchLatestInvoices();
  const cardDataPromise = fetchCardData();
  
  const [revenue, latestInvoices, { totalPaidInvoices, totalPendingInvoices, numberOfInvoices, numberOfCustomers }] =
    await Promise.all([revenuePromise, latestInvoicesPromise, cardDataPromise]);
  ```

## 요약

- 외부 API가 없을 땐
  - Next.js 안에서 서버 사이드로 API를 만들거나(Route Handler = API 레이어)
  - 서버 컴포넌트에서 직접 DB를 질의한다.(DB 쿼리)
  - 클라이언트 컴포넌트에서 DB 커넥션, 시크릿, 환경변수를 절대 노출 금지

## 참고 링크

- [Next.js 공식 문서 - Fetching Data (Using SQL)](https://nextjs.org/learn/dashboard-app/fetching-data#using-sql)


