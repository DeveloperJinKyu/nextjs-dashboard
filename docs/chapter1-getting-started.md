# Chapter 1: 시작하기

### pnpm이 npm보다 빠르고 효율적이다

- Next.js 튜토리얼에서는 `pnpm`을 패키지 매니저로 권장한다.
- `npm`이나 `yarn`보다 빠르고 효율적이어서 개발 속도가 향상된다.
- 전역 설치 후 프로젝트 생성 시 `--use-pnpm` 플래그를 사용하면 된다.

```bash
npm install -g pnpm
npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm
```

- 프로젝트 패키지 설치와 개발 서버 실행도 `pnpm` 명령어를 사용한다.
  - `pnpm i`: 패키지 설치
  - `pnpm dev`: 개발 서버 실행 (포트 3000)

### 폴더구조

- **`/app`**: 라우트, 컴포넌트, 로직이 모두 들어있다. App Router를 사용하며 대부분의 작업을 여기서 한다.
- **`/app/lib`**: 재사용 가능한 유틸리티 함수와 데이터 페칭 함수가 있다.
- **`/app/ui`**: 카드, 테이블, 폼 등 UI 컴포넌트가 있다. 시간 절약을 위해 미리 스타일링되어 있다.
- **`/public`**: 이미지 같은 정적 자산을 저장한다.
- **Config Files**: 루트에 `next.config.ts` 같은 설정 파일이 있다. `create-next-app`으로 프로젝트 생성 시 자동으로 만들어지고 미리 구성된다.

이 튜토리얼은 처음부터 코드를 작성하는 방식이 아니라, 대부분의 코드가 이미 작성되어 있어 실제 개발 환경과 비슷하다.

### 플레이스홀더 데이터

- 데이터베이스나 API가 준비되지 않았을 때 UI 개발에 사용하는 테스트용 데이터다.
- `app/lib/placeholder-data.ts` 파일에 제공되어 있다.
- 각 JavaScript 객체는 데이터베이스의 테이블을 나타낸다.

```typescript
const invoices = [
  {
    customer_id: customers[0].id,
    amount: 15795,
    status: 'pending',
    date: '2022-12-06',
  },
  // ...
];
```

- JSON 형식이나 JavaScript 객체로 직접 만들 수도 있고, mockAPI 같은 서드파티 서비스를 쓸 수도 있다.
- 나중에 데이터베이스 설정 챕터에서 이 데이터로 데이터베이스를 시드(seed)할 때 사용한다.

---

## 참고 링크

- [Next.js 공식 문서 - Getting Started](https://nextjs.org/learn/dashboard-app/getting-started)
