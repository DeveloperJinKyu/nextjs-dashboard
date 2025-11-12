# Chapter 5: Navigating Between Pages

## 핵심 요약

- **next/link**
  - 내부 페이지 간 이동은 `<a>` 대신 `next/link`의 `<Link>`를 사용해 클라이언트 사이드 내비게이션을 수행한다.
  - 페이지 전환 시 전체 페이지 새로고침이 발생하지 않아 네이티브 앱과 유사한 매끄러운 UX를 제공한다.
  - 프로덕션에서 뷰포트에 등장한 Link 태그는 자동 사전로딩(prefetch)되어 클릭 시 즉시 전환된다.

    ```tsx
    // app/ui/dashboard/nav-links.tsx (예시)
    import Link from 'next/link';
    
    export default function NavLinks() {
      return (
        <nav className="flex gap-2">
          <Link href="/dashboard" className="px-3 py-2 rounded-md">Home</Link>
          <Link href="/dashboard/invoices" className="px-3 py-2 rounded-md">Invoices</Link>
          <Link href="/dashboard/customers" className="px-3 py-2 rounded-md">Customers</Link>
        </nav>
      );
    }
    ```

- **usePathname**
  - `usePathname()` React hook은 현재 URL 경로를 반환하여 활성 링크 스타일링에 유용하다.
  - hook 사용 파일은 클라이언트 컴포넌트여야 하므로 `"use client"` 지시문을 추가한다.
  - `clsx` 라이브러리 등으로 조건부 클래스를 적용해 활성 상태를 표시한다.

    ```tsx
    'use client';
    import Link from 'next/link';
    import { usePathname } from 'next/navigation';
    import clsx from 'clsx';
    
    const links = [
      { name: 'Home', href: '/dashboard' },
      { name: 'Invoices', href: '/dashboard/invoices' },
      { name: 'Customers', href: '/dashboard/customers' },
    ];
    
    export default function NavLinks() {
      const pathname = usePathname();
      return (
        <nav className="flex gap-2">
          {links.map((link) => (
            <Link
              key={link.name}
              href={link.href}
              className={clsx(
                'px-3 py-2 rounded-md hover:bg-sky-100 hover:text-blue-600',
                { 'bg-sky-100 text-blue-600': pathname === link.href }
              )}
            >
              {link.name}
            </Link>
          ))}
        </nav>
      );
    }
    ```

- **Next.js에서 navigation이 작동하는 방식**
  - 라우트 세그먼트 단위 자동 코드 분할(code-splitting)로, 초기 로드 크기를 줄이고 페이지 간 오류 격리 효과가 있다.(특정 페이지에서 오류가 발생하더라도 나머지 애플리케이션은 계속 작동)
  - 브라우저가 파싱해야 할 코드도 줄어들어 애플리케이션 속도가 향상된다.
  - 브라우저가 초기 페이지 로드 시 모든 애플리케이션 코드를 로드하는 기존 React SPA와 다른 방식이다.
  - `<Link>`가 뷰포트에 나타나면 대상 라우트의 번들을 백그라운드에서 prefetch한다(프로덕션).
  - 내비게이션 시 레이아웃은 유지되고 페이지 영역만 교체되는 부분 렌더링으로 상태 보존과 빠른 전환을 제공한다.
  - 내부 링크는 `<Link>`를, 외부 링크나 파일 다운로드 등 전체 새로고침이 필요한 경우에만 `<a>`를 사용한다.

## 참고 링크

- [Next.js 공식 문서 - Navigating Between Pages](https://nextjs.org/learn/dashboard-app/navigating-between-pages)


