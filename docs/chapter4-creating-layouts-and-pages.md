# Chapter 4: Creating Layouts and Pages

- **중첩 라우팅**
  - Next.js는 파일 시스템 라우팅을 사용한다. 폴더가 곧 라우트 세그먼트(route segment)이며 URL 세그먼트와 1:1로 매핑된다.
  - 각 라우트에 접근 가능하려면 해당 폴더 안에 `page.tsx`가 필요하다. 예: `/app/dashboard/page.tsx` → `/dashboard`.
  - 하위 폴더로 더 깊이 중첩해 `/dashboard/customers`, `/dashboard/invoices`처럼 세부 페이지를 만든다.
  - 관련 컴포넌트·테스트·유틸은 라우트와 같은 경로에 '공존(colocation)'시킬 수 있으며, 퍼블릭 라우팅은 `page.tsx`만 담당한다.

    ```tsx
    // app/dashboard/page.tsx
    export default function Page() {
      return <p>Dashboard Page</p>;
    }
    ```

- **Next에서 layout의 이점**
  - `layout.tsx`는 여러 페이지 간에 공유되는 UI(예: 사이드 내비게이션, 헤더)를 정의한다.
  - 레이아웃은 `children`을 받아 하위 라우트의 페이지 또는 레이아웃을 감싼다(중첩 가능).
  - 부분 렌더링(Partial Rendering): 네비게이션 시 페이지 영역만 교체되고 레이아웃은 재렌더링되지 않아 클라이언트 상태가 보존된다.

    ```tsx
    // app/dashboard/layout.tsx
    import SideNav from '@/app/ui/dashboard/sidenav';
    
    export default function Layout({ children }: { children: React.ReactNode }) {
      return (
        <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
          <div className="w-full flex-none md:w-64">
            <SideNav />
          </div>
          <div className="grow p-6 md:overflow-y-auto md:p-12">{children}</div>
        </div>
      );
    }
    ```

- **root layout**
  - `/app/layout.tsx`는 모든 Next.js 앱에 필수인 루트 레이아웃으로, 전체 앱에 공통 적용되는 `<html>`/`<body>`·글로벌 스타일·폰트·메타데이터를 정의한다.
  - 대시보드 고유의 UI는 루트가 아닌 `/app/dashboard/layout.tsx`처럼 해당 라우트 전용 레이아웃에 둔다.

    ```tsx
    // app/layout.tsx
    import '@/app/ui/global.css';
    import { inter } from '@/app/ui/fonts';
    
    export default function RootLayout({ children }: { children: React.ReactNode }) {
      return (
        <html lang="en">
          <body className={`${inter.className} antialiased`}>{children}</body>
        </html>
      );
    }
    ```

## 참고 링크

- [Next.js 공식 문서 - Creating Layouts and Pages](https://nextjs.org/learn/dashboard-app/creating-layouts-and-pages)


