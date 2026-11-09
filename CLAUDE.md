# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 현황

고양이 공개 API를 사용해 보는 프로젝트입니다. git 저장소는 아닙니다.

`index.html` 한 파일에 화면과 스크립트가 모두 들어 있고, 빌드 과정 없이 브라우저로 바로 엽니다. Tailwind와 Lucide는 CDN으로 불러오므로 인터넷 연결이 필요합니다.

연습용 프로젝트라서 사용자의 결정에 따라 The Cat API 키를 `index.html`에 직접 넣었습니다. 원본 키는 `.env`의 `CAT_KEY`에 있습니다.

## 기술 스택

HTML, Tailwind, JavaScript, Lucide 아이콘으로 웹페이지를 만듭니다.

## 작업 규칙

- 사용자가 특정 API나 라이브러리를 사용하라고 명시하면 그것을 사용합니다.
- 작업 중 API나 라이브러리가 추가로 필요하면 사용하기 전에 사용자에게 확인을 받습니다.
- 작업 중 문제가 생기면 안 된다고만 하지 말고 해결 방법을 함께 제시합니다.
- 사용자와 한글로 대화합니다.

## 사용하는 API

2026-11-09에 직접 호출해 확인한 결과입니다. The Cat API의 자세한 사용법은 `thecatapi-usage.md`를 참고하세요.

| 엔드포인트 | API 키 | 응답 |
|---|---|---|
| `GET https://api.thecatapi.com/v1/images/search?limit=N` | 불필요 | `[{id, url, width, height}]` |
| `GET https://api.thecatapi.com/v1/breeds` | **필요** (없으면 403) | 키가 없어 확인하지 못함 |
| `GET https://catfact.ninja/fact` | 불필요 | `{fact, length}` (`fact`는 영어) |
