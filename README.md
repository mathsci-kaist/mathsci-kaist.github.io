# KAIST 수리과학과 학계 동문 사이트

이 저장소는 KAIST 수리과학과 학계 동문 정보를 정적 사이트로 제공하는 프로젝트입니다.

## 동문 데이터 관리 규칙

`people.csv` 업데이트 규칙은 아래 문서를 반드시 참고하세요.

- [PEOPLE_CSV_RULES.md](./PEOPLE_CSV_RULES.md)

핵심 규칙:

- 데이터 원본 파일: `assets/data/people.csv`
- 신규 박사학위자 추가 시 `handle` 페이지에서 `thesis` 제목 일치 검증 필수
- `debut`는 박사학위 연도(`phd`)와 동일하게 입력
- `scholar`, `mrauthid`, `mathgen`은 확인된 값만 입력 (미확정 시 공란)

## GitHub Pages 배포 설정

- GitHub Pages `Source`는 반드시 **GitHub Actions**로 설정하세요.
- 이 저장소는 `.github/workflows/hugo.yml`에서 `hugo --minify`로 빌드 후 `public/`를 배포합니다.
- `Deploy from a branch`(root)로 설정하면 README가 첫 화면으로 보일 수 있습니다.
