# Chapter 2: CSS Styling

### Global Styles

- 이 프로젝트에서는 `/app/ui/global.css` 파일을 사용해 애플리케이션의 모든 라우트에 적용되는 CSS 규칙을 추가할 수 있다.
- CSS 리셋 규칙, 링크 같은 HTML 요소의 사이트 전체 스타일 등을 정의할 때 유용하다.
- 보통 최상위 컴포넌트인 루트 레이아웃(`/app/layout.tsx`)에서 import 한다.

```typescript
import '@/app/ui/global.css';
```

- `global.css` 파일 안에 `@tailwind` 지시문이 있으면 Tailwind CSS가 자동으로 적용된다.

### Tailwind

- Tailwind는 유틸리티 클래스를 React 코드에 직접 작성해 빠르게 개발할 수 있게 해주는 CSS 프레임워크다.
- 클래스 이름을 추가해 요소를 스타일링한다. 예를 들어 `"text-blue-500"`를 추가하면 텍스트가 파란색이 된다.

```tsx
<h1 className="text-blue-500">I'm blue!</h1>
```

- CSS 스타일은 전역으로 공유되지만, 각 클래스는 개별 요소에만 적용된다.
- 요소를 추가하거나 삭제해도 별도 스타일시트 유지, 스타일 충돌, CSS 번들 크기 증가를 걱정할 필요가 없다.
- `create-next-app`으로 프로젝트를 만들 때 Tailwind 사용 여부를 선택할 수 있다. 선택하면 필요한 패키지가 자동으로 설치되고 설정된다.

### CSS Modules

- CSS Modules는 자동으로 고유한 클래스 이름을 생성해 CSS를 컴포넌트에 스코프를 지정한다. 스타일 충돌을 걱정할 필요가 없다.
- 전통적인 CSS 규칙을 작성하거나 스타일을 JSX와 분리하고 싶을 때 좋은 대안이다.
- `.module.css` 확장자를 가진 파일을 만들고, 컴포넌트에서 import 해서 사용한다.

```css
/* home.module.css */
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

```tsx
import styles from '@/app/ui/home.module.css';

<div className={styles.shape} />
```

- Tailwind와 CSS Modules는 같은 애플리케이션에서 함께 사용할 수 있다. 선호도에 따라 선택하면 된다.

### clsx Library

- 상태나 다른 조건에 따라 요소를 조건부로 스타일링해야 할 때 사용한다.
- 클래스 이름을 쉽게 토글할 수 있게 해주는 라이브러리다.
- 예를 들어 `InvoiceStatus` 컴포넌트에서 `status`가 `'paid'`면 초록색, `'pending'`이면 회색으로 표시하고 싶을 때 사용한다.

```tsx
import clsx from 'clsx';

export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
      {/* ... */}
    </span>
  );
}
```

- 객체 형태로 조건부 클래스를 전달하면 조건이 `true`일 때만 해당 클래스가 적용된다.

---

## 참고 링크

- [Next.js 공식 문서 - CSS Styling](https://nextjs.org/learn/dashboard-app/css-styling)

