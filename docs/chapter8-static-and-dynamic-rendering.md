# Chapter 8: Static and Dynamic Rendering


- **정적 렌더링이란?**
  - 배포 시점(빌드 타임) 또는 리밸리데이트 시 서버에서 미리 렌더링한 결과를 캐시로 서빙하는 방식.
  - 이점: 전 세계 CDN 캐시로 더 빠른 응답, 서버 부하 감소, SEO 친화적.

- **정적 렌더링을 쓰는 경우**
  - 사용자별로 달라지지 않는 데이터, 자주 변하지 않는 콘텐츠.
  - 예: 마케팅 페이지, 블로그 포스트, 제품 상세(공용 데이터).

- **동적 렌더링이란?**
  - 요청 시점마다 서버에서 렌더링하는 방식(요청마다 새로 그려짐).
  - 이점: 실시간/자주 변경 데이터 반영, 사용자 맞춤(쿠키·세션·URL 파라미터 등 요청 시점 정보 활용).

- **동적 렌더링을 쓰는 경우**
  - 대시보드·프로필 등 개인화 화면, 빈번히 변하는 데이터, 요청 시 연산/검증이 필요한 화면.

- **느린 데이터 가져오기**
  - 동적 렌더링에서는 “가장 느린 데이터 패칭 속도”가 페이지 최초 표시 속도를 좌우한다.
  - 데모: `app/lib/data.ts`의 `fetchRevenue()`에 인위적 지연을 추가해 전체 로드 지연을 관찰.

    ```ts
    // app/lib/data.ts
    export async function fetchRevenue() {
      try {
        // 데모용 인위적 지연 (프로덕션에서는 사용 금지)
        console.log('Fetching revenue data...');
        await new Promise((resolve) => setTimeout(resolve, 3000));
    
        const data = await sql<Revenue[]>`SELECT * FROM revenue`;
    
        console.log('Data fetch completed after 3 seconds.');
        return data;
      } catch (error) {
        console.error('Database Error:', error);
        throw new Error('Failed to fetch revenue data.');
      }
    }
    ```

  - 브라우저에서 `http://localhost:3000/dashboard`를 열면 전체 페이지가 느린 요청만큼 지연되는 것을 확인할 수 있다.

## 참고 링크

- [Next.js 공식 문서 - Static and Dynamic Rendering](https://nextjs.org/learn/dashboard-app/static-and-dynamic-rendering)

---

## 추가 정리: SSR / SSG / CSR 비교

| 방식                              | 의미                                 | 데이터 패칭 시점               | 배포 형태                   |
| ------------------------------- | ---------------------------------- | ----------------------- | ----------------------- |
| **SSR (Dynamic Rendering)**     | 서버에서 요청마다 HTML 생성                  | **페이지 진입 시점**           | Node 서버 / Vercel        |
| **SSG (Static Rendering)**      | 빌드 시 HTML 생성 후 캐싱                  | **빌드 타임 또는 ISR 재생성 시점** | Vercel / Static hosting |
| **CSR (Client Side Rendering)** | 빈 HTML + JS 로딩 후 클라이언트에서 데이터 fetch | **사용자 접속 시점**           | S3 + CloudFront SPA     |

### S3 + CloudFront + Next.js Client Components

- Next.js를 사용해도 S3에 정적 파일로 배포하면 결과적으로 CSR 방식이다.
- HTML에는 데이터가 포함되지 않고, 클라이언트에서 API 호출로 최신 데이터를 로드한다.
- 빌드 시점 데이터를 포함하지 않으므로 Next.js 문서의 “정적 렌더링(SSG/ISR)”과는 다르다.

### SSR 사용 시 동작
- 사용자가 페이지에 접근할 때마다 서버에서 최신 데이터를 fetch하고, 데이터가 포함된 HTML을 전달한다.
- DB 값이 바뀌어도 브라우저는 자동 새로고침되지 않는다. 실시간 갱신을 원하면 Polling, SSE, WebSocket, SWR/React Query 등을 사용한다.

### 실시간 갱신 옵션(요약)
- Polling: 일정 주기로 API 재호출
- SWR/React Query: 포커스/네트워크 변화 시 자동 리패치
- WebSocket: 양방향 실시간 통신
- SSE(Server-Sent Events): 단방향 실시간 스트림


