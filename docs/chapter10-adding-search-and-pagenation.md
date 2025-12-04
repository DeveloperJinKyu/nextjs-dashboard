# Chapter 10: Adding Search and Pagination

### next.js의 hook useSearchParams이란?

- **URL 쿼리 파라미터를 읽는 Next.js 전용 React Hook**이다.  
  - 예: `/dashboard/invoices?page=1&query=pending` → `{ page: '1', query: 'pending' }` 형태로 접근 가능하다.  
  - `useSearchParams()`가 반환하는 값에서 `get`, `has` 등을 사용해 현재 URL의 검색 조건을 읽는다.
- **현재 라우터 상태와 연동되는 “반응형(read-only) 쿼리 객체”**를 제공한다.  
  - 사용자가 검색어를 입력해 URL이 바뀌면, 이 훅을 사용하는 컴포넌트가 다시 렌더링된다.  
  - React 훅이므로 Client Component(파일 상단에 `'use client';`가 선언된 컴포넌트)에서만 사용할 수 있다.
- **서버에서 검색/페이지 데이터를 가져올 때의 진입점**으로도 쓰인다.  
  - 클라이언트에서 URL 쿼리가 바뀌면, 서버 컴포넌트가 해당 쿼리를 기준으로 데이터를 다시 fetch 하고, 서버 렌더링 결과가 클라이언트에 스트리밍된다.  
  - 이렇게 하면 검색 상태를 URL에 올려두면서도 서버 사이드 렌더링(SSR)의 이점을 유지할 수 있다.

### useSearchParams과 URLSearchParams의 차이

- **역할과 출처가 다르다.**  
  - `useSearchParams`는 **Next.js(react + next/navigation)의 Hook**이고,  
  - `URLSearchParams`는 **브라우저 Web API**로, 쿼리 문자열을 다루기 위한 표준 객체이다.
- **useSearchParams → 읽기 전용(Immutable) / URLSearchParams → 수정 가능(Mutable)**  
  - `useSearchParams()`가 반환하는 것은 `ReadonlyURLSearchParams`와 유사한 읽기 전용 객체라 직접 `set`, `delete`를 호출할 수 없다.  
  - 그래서 다음과 같이 **복사본을 만들어 수정 가능한 인스턴스를 생성**한다:

```ts
const searchParams = useSearchParams();

const params = new URLSearchParams(searchParams); // 기존 쿼리 복사 + mutable
params.set('query', term);
params.set('page', '1');
```

- **기존 쿼리 보존 vs 새 쿼리 생성**  
  - `new URLSearchParams(searchParams)`는 **현재 URL의 모든 쿼리 파라미터를 복사**한 뒤, 일부만 수정한다.  
    - 예: `?page=2&status=paid&query=apple` → `page`, `status`는 유지하고 `query`만 변경.  
  - `new URLSearchParams()`처럼 인자를 비우면 **완전히 빈 쿼리에서 시작**하므로, 기존 쿼리가 모두 사라지고 새로 설정한 값만 남는다.
- **최종적으로는 문자열 쿼리로 변환해 라우터로 전달**  
  - `params.toString()`으로 `page=1&query=pending` 같은 쿼리 문자열을 만들고,  
  - `useRouter().replace(\`\${pathname}?\${params.toString()}\`)`로 URL을 갱신해 검색/페이지 상태를 라우팅에 반영한다.  
  - 이 패턴을 통해, 복잡한 문자열 조합 없이 안전하게 쿼리 문자열을 생성할 수 있다.

### 클라이언트 사이드 방식, 서버 사이드 방식 검색 및 페이지 매김 기능 차이와 이점

- **클라이언트 사이드 검색/페이지네이션의 특징**  
  - 보통 `useState`, `useEffect` 등으로 검색어/페이지 상태를 관리하고, 한번 가져온 전체 리스트를 클라이언트에서 필터링·슬라이싱한다.  
  - 장점: 즉각적인 UI 반응, 서버 왕복이 적을 수 있다(데이터가 작을 때).  
  - 단점: 대량 데이터일수록 초기 전송량이 크고, 클라이언트에서의 연산 비용이 증가하며, URL에 상태가 잘 드러나지 않으면 **새로고침/공유/북마크에 약하다.**

- **서버 사이드 검색/페이지네이션(이 챕터의 패턴)의 특징**  
  - 검색어(`query`), 페이지 번호(`page`) 등을 **URL search params에 저장**한다.  
  - 클라이언트에서는 `useSearchParams`, `usePathname`, `useRouter`로 URL을 업데이트만 하고,  
  - 실제 데이터 필터링/페이지 계산은 서버 컴포넌트에서 수행한다:

```ts
// /dashboard/invoices/page.tsx (Server Component)
export default async function Page(props: {
  searchParams?: Promise<{ query?: string; page?: string }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;

  const totalPages = await fetchInvoicesPages(query);
  // ...
}
```

- **서버 사이드 방식의 주요 이점**  
  - **URL 기반 상태 관리**  
    - 검색어와 페이지가 `/dashboard/invoices?page=2&query=pending`처럼 URL에 포함되므로,  
      - 북마크/링크 공유 시 동일 상태로 바로 진입 가능  
      - 새로고침해도 동일 상태 유지  
      - 분석/로그에서 쿼리값을 그대로 활용 가능하다.
  - **SSR + 데이터 보안**  
    - 데이터 fetch가 서버에서 이뤄지기 때문에, DB 쿼리나 비밀 키를 노출하지 않고도 안전하게 데이터를 가져올 수 있다.  
    - 클라이언트는 결과 UI만 전달받기 때문에, API 레이어 없이도 서버 액션/서버 컴포넌트 조합으로 구조를 단순화할 수 있다.
  - **대량 데이터에 유리한 성능**  
    - 필터링/페이지 계산을 서버에서 수행하고, 필요한 데이터만 클라이언트로 보내므로 네트워크 비용과 클라이언트 연산 비용을 줄인다.  
    - 검색어 타이핑에 대해서는 `use-debounce`의 `useDebouncedCallback`을 사용해 요청 빈도를 줄여, DB 부담을 완화한다.

- **클라이언트/서버 협업 패턴의 정리**  
  - 클라이언트:  
    - 사용자의 입력 이벤트를 처리하고  
    - `useSearchParams` + `URLSearchParams` + `useRouter`로 **URL만 업데이트**한다.  
  - 서버:  
    - URL에 담긴 `searchParams`를 읽어  
    - `fetchFilteredInvoices`, `fetchInvoicesPages` 등으로 데이터를 가져오고  
    - 검색 및 페이지네이션이 반영된 UI를 렌더링해 클라이언트로 스트리밍한다.  
  - 이 구조 덕분에, **“상태는 URL에, 데이터 처리는 서버에, 인터랙션은 클라이언트에”**라는 역할 분리가 명확해진다.

## 참고 링크

- [Next.js Learn – Adding Search and Pagination](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination)
