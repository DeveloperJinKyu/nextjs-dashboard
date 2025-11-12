# Chapter 6: Setting Up Your Database

- **프로젝트를 GitHub에 푸시**
  - 로컬 저장소를 GitHub 원격 저장소에 연결해 `main` 브랜치에 푸시한다.
  - 이후 Vercel과 연동 시 푸시마다 자동 배포가 이루어지고, PR마다 미리보기 URL이 생성된다.

- **Vercel 계정을 설정하고 GitHub 저장소를 연결하여 즉시 미리보기 및 배포**
  - Vercel에 가입 후 “Continue with GitHub”로 GitHub 계정을 연결한다.
  - Import Project에서 해당 리포지터리를 선택하고 “Deploy”를 클릭한다.
  - 연결 이후 `main` 푸시 시 자동 재배포, PR은 미리보기 배포로 오류를 조기에 확인 가능하다.

- **프로젝트를 생성하고 Postgres 데이터베이스에 연결**
  - Vercel 프로젝트 대시보드 → Storage → Create Database에서 Postgres 선택(Neon 또는 Supabase 등).
  - 권장 리전: Washington D.C (iad1). 생성 후 `.env.local` 탭에서 시크릿을 “Show secret” 후 “Copy Snippet”으로 복사한다.
  - 로컬에서 `.env.example`를 `.env`로 변경하고, 복사한 환경변수를 붙여넣는다.
  - `.gitignore`에 `.env`가 포함돼 있는지 확인한다.

    ```gitignore
    # 환경변수 파일은 절대 커밋 금지
    .env
    .env.local
    .env.*.local
    ```

- **초기 데이터로 데이터베이스를 시드**
  - 개발 서버를 실행한 뒤 `http://localhost:3000/seed`로 이동하면 시드 스크립트가 실행되어 테이블 생성 및 더미 데이터가 삽입된다.
  - 시딩은 애플리케이션을 빌드할 때 사용할 데이터가 필요할 때 유용하다.
  - 완료 메시지: “Database seeded successfully”가 브라우저에 표시된다.(생성된 파일은 완료 후 삭제해도 된다.)
  - 트러블슈팅
    - `.env`에 DB 시크릿이 올바르게 채워졌는지 확인(복사 전 “Show secret” 필수).
    - `bcrypt`가 환경에 맞지 않으면 `bcryptjs`로 대체 가능.
    - 재실행 필요 시 DB에서 관련 테이블을 `DROP TABLE ...`로 제거 후 다시 시도(예제 앱에서만 권장, 프로덕션 금지).

## 참고 링크

- [Next.js 공식 문서 - Setting Up Your Database (Seed your database)](https://nextjs.org/learn/dashboard-app/setting-up-your-database#seed-your-database)


