# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

Jekyll 기반 개인 개발 블로그. **minima** 테마를 사용하며 GitHub Pages로 배포.

- 사이트 제목: 이수빈의 개발 블로그
- Base URL: `/blog`
- 테마: minima (~2.5)
- Jekyll 버전: ~4.4.1

## 주요 명령어

```bash
# 의존성 설치
bundle install

# 로컬 개발 서버 실행 (http://localhost:4000/blog/)
bundle exec jekyll serve

# 사이트 빌드 (_site/ 디렉토리에 출력)
bundle exec jekyll build
```

참고: `_config.yml` 변경 시 서버를 재시작해야 반영됨.

## 블로그 포스트 작성 규칙

포스트는 `_posts/` 디렉토리에 `YYYY-MM-DD-<슬러그>.markdown` 형식으로 작성.

프론트매터 형식:
```yaml
---
layout: post
title:  "포스트 제목"
date:   YYYY-MM-DD HH:MM:SS +0900
categories: 카테고리명
---
```

기존 카테고리: `ECC`, `Database`. 포스트는 한국어 마크다운으로 작성.
