# Chapter 3: Optimizing Fonts and Images

## 핵심 요약

- **폰트 최적화가 필요한 이유**
  - 웹폰트 로딩 중 시스템 폰트로 먼저 그렸다가 사용자 정의 폰트로 교체되면 Cumulative Layout Shift(CLS)가 발생한다.
  - CLS는 텍스트의 크기·자간·행간 변화로 주변 레이아웃을 밀어 UI가 흔들리는 체감이 크다.
  - `next/font`는 빌드 타임에 폰트 파일을 내려받아 정적 자산으로 함께 서빙해, 런타임 추가 네트워크 요청을 줄이고 안정적 렌더링을 돕는다.

- **Next.js에서 font 변경 방법**
  - `app/ui/fonts.ts`에서 `next/font/google` 모듈을 사용해 폰트를 선언한다.

    ```ts
    import { Inter, Lusitana } from 'next/font/google';
    
    export const inter = Inter({ subsets: ['latin'] });
    export const lusitana = Lusitana({ subsets: ['latin'], weight: ['400', '700'] });
    ```

  - `app/layout.tsx`의 `<body>`에 기본 폰트를 적용한다. 필요 시 Tailwind의 `antialiased`로 가독성을 보완한다.

    ```tsx
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

  - 보조 폰트(예: `Lusitana`)는 특정 요소에만 선택적으로 적용해 혼합 타이포그래피를 구성한다.

- **이미지 최적화가 필요한 이유**
  - 단순 `<img>` 사용 시 반응형 처리, 디바이스별 크기 지정, 로딩 중 레이아웃 안정성, 뷰포트 밖 지연 로딩 등 수작업이 많고 실수 여지가 크다.
  - 비최적화 원본을 그대로 전송하면 불필요하게 큰 용량·대역폭 낭비와 느린 로딩이 발생한다.

- **Next Image 컴포넌트의 이점**
  - `<Image>`는 로딩 중 레이아웃 이동을 자동 방지(가로·세로 비율 기반)한다.
  - 디바이스·뷰포트에 맞춰 이미지를 리사이즈해 과대 전송을 줄인다.
  - 기본 지연 로딩(lazy loading)과 최신 포맷(WebP, AVIF) 서빙을 지원한다.

- **사용자 정의 폰트와 크기가 지정되지 않은 이미지가 레이아웃을 바꾸는 일반적 원인**
  - 폰트: 처음엔 시스템 폰트로 그렸다가 웹폰트 적용 시 글자 너비·줄바꿈이 바뀌어 주변 요소가 이동한다.
  - 이미지: `width`/`height`(또는 비율)가 없으면 로딩 중 컨테이너 크기가 확정되지 않아 콘텐츠가 밀린다.
  - 과도한 원본 크기/비율 불일치도 공간 재계산을 유발해 레이아웃 흔들림을 키운다.

- **실전 예시: 히어로 이미지**
  - 데스크톱과 모바일을 다른 자산으로 구분하고, 각기 적절한 `width`/`height`를 제공해 CLS를 방지한다.

    ```tsx
    import Image from 'next/image';
    
    // Desktop 전용
    <Image
      src="/hero-desktop.png"
      width={1000}
      height={760}
      className="hidden md:block"
      alt="Screenshots of the dashboard project showing desktop version"
    />
    
    // Mobile 전용
    <Image
      src="/hero-mobile.png"
      width={560}
      height={620}
      className="block md:hidden"
      alt="Screenshots of the dashboard project showing mobile version"
    />
    ```

## 참고 링크

- [Next.js 공식 문서 - Optimizing Fonts and Images](https://nextjs.org/learn/dashboard-app/optimizing-fonts-images)


