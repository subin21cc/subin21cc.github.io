# SUBIN.dev

[![Deploy](https://github.com/subin21cc/subin21cc.github.io/actions/workflows/pages-deploy.yml/badge.svg)](https://github.com/subin21cc/subin21cc.github.io/actions/workflows/pages-deploy.yml)
[![Jekyll](https://img.shields.io/badge/Jekyll-Chirpy%207.5-CC0000?logo=jekyll&logoColor=white)](https://github.com/cotes2020/jekyll-theme-chirpy)

> 프로젝트 개발 과정과 기술 학습을 기록하는 개인 기술 블로그

**🔗 [subin21cc.github.io](https://subin21cc.github.io)**

## Overview

- **Static Site** — Jekyll 기반 정적 사이트, Chirpy 테마 커스터마이징
- **CI/CD** — `main` 브랜치 push 시 GitHub Actions로 빌드·검증 후 GitHub Pages 자동 배포
- **SEO** — sitemap, Open Graph 메타데이터, Google Search Console 등록
- **Analytics** — GoatCounter 기반 방문 통계

## Project Structure

```text
.
├── _posts/          # 블로그 글 (Markdown)
├── _tabs/           # About · Projects 등 고정 페이지
├── _layouts/        # 홈 · 카테고리 · 아카이브 커스텀 레이아웃
├── _includes/       # 메타데이터 · 푸터 등 공통 조각
├── _data/           # 연락처 · 공유 버튼 · UI 문구 설정
├── assets/          # 이미지 · 스타일
└── _config.yml      # 사이트 전역 설정
```

## Getting Started

**Requirements:** Ruby 3.x, Bundler

```bash
bundle install
bash tools/run.sh     # 로컬 서버 실행 → http://127.0.0.1:4000
bash tools/test.sh    # 프로덕션 빌드 및 HTML 링크 검사
```

## License

- 게시글 및 이미지: © Shin-SuBin. All rights reserved.
- 테마 및 소스 코드: [MIT License](LICENSE)
