---
layout: post
title:  "IntelliJ IDEA 단축키"
date:   2026-08-06 09:00:00 +0900
categories: SelfStudy
---

# IntelliJ IDEA 단축키 정리 (Windows / Mac)

스프링 공부하면서 실제로 손이 자주 가는 것만 추렸다.
Windows 기준이고, Mac은 `⌘`(Command) · `⌥`(Option) · `⌃`(Control) · `⇧`(Shift).

> 단축키가 안 먹으면 대부분 아래 셋 중 하나다.
> 1. 한글 입력 상태 — 한/영 키로 영문 전환
> 2. `File > Settings > Keymap` 이 `Windows`가 아님
> 3. 다른 프로그램(한글, Discord, GeForce Experience)이 전역 단축키를 먼저 가로챔

---

## 0. 이것 하나만 외운다면

| 기능 | Windows | Mac |
|---|---|---|
| **Find Action** — 모든 기능을 이름으로 검색해 실행 | `Ctrl + Shift + A` | `⌘ ⇧ A` |
| **Show Intention Actions** — 상황에 맞는 수정/생성 제안 | `Alt + Enter` | `⌥ Enter` |
| Search Everywhere — 파일·클래스·설정 전부 검색 | `Shift` 두 번 | `⇧` 두 번 |

`Alt + Enter`와 `Ctrl + Shift + A`만 알면 나머지 단축키를 몰라도 다 할 수 있다.
단축키가 기억 안 나면 `Ctrl + Shift + A`로 기능 이름을 검색하면 되고,
검색 결과 오른쪽에 그 기능의 실제 단축키가 같이 표시된다.

---

## 1. 실행 · 빌드

| 기능 | Windows | Mac |
|---|---|---|
| 직전 실행 다시 실행 | `Shift + F10` | `⌃ R` |
| 커서 위치의 것 실행 (테스트 1개, main 등) | `Ctrl + Shift + F10` | `⌃ ⇧ R` |
| 실행 대상 선택해서 실행 | `Alt + Shift + F10` | `⌃ ⌥ R` |
| 프로젝트 빌드 | `Ctrl + F9` | `⌘ F9` |
| 실행 중지 | `Ctrl + F2` | `⌘ F2` |

## 2. 코드 생성 · 자동완성

| 기능 | Windows | Mac |
|---|---|---|
| **Generate** — 생성자, getter/setter, toString, override | `Alt + Insert` | `⌘ N` |
| **Override Methods** | `Ctrl + O` | `⌃ O` |
| Implement Methods | `Ctrl + I` | `⌃ I` |
| 기본 자동완성 | `Ctrl + Space` | `⌃ Space` |
| 타입 기반 스마트 완성 | `Ctrl + Shift + Space` | `⌃ ⇧ Space` |
| 문장 자동 완성 (`;`, `{}` 채우기) | `Ctrl + Shift + Enter` | `⌘ ⇧ Enter` |
| Live Template 삽입 (`psvm`, `sout`, `fori`) | `Ctrl + J` | `⌘ J` |
| 코드를 `try/catch`·`if`로 감싸기 | `Ctrl + Alt + T` | `⌘ ⌥ T` |

**자주 쓰는 Live Template** — 입력 후 `Tab`

- `psvm` → `public static void main(String[] args) {}`
- `sout` → `System.out.println();`
- `soutv` → 변수명과 값을 같이 출력
- `fori` → 인덱스 for 문
- `iter` → 향상된 for 문

## 3. 테스트

| 기능 | Windows | Mac |
|---|---|---|
| **테스트 클래스 생성 / 테스트로 이동** | `Ctrl + Shift + T` | `⌘ ⇧ T` |
| 커서 위치 테스트 1개 실행 | `Ctrl + Shift + F10` | `⌃ ⇧ R` |
| 직전 테스트 재실행 | `Shift + F10` | `⌃ R` |
| 실패한 테스트만 재실행 | 실행창의 `Ctrl + Alt + R` 대신 툴바 버튼 사용 | 동일 |

테스트 껍데기 만드는 흐름:

1. 커서를 **클래스 이름 위**에 놓는다 (`public class MemberService`의 `MemberService`)
2. `Ctrl + Shift + T` → `Create New Test...`
3. Testing library `JUnit5` 선택, 테스트할 메서드 체크 → OK

`Ctrl + Shift + T`가 안 먹으면 클래스 이름 위에서 `Alt + Enter` → `Create Test`.
이미 테스트 파일이 있으면 새로 만들지 않고 그 파일로 **이동**한다. 같은 단축키로 코드 ↔ 테스트를 왔다 갔다 할 수 있다.

## 4. 리팩터링

| 기능 | Windows | Mac |
|---|---|---|
| 리팩터링 메뉴 전체 | `Ctrl + Alt + Shift + T` | `⌃ T` |
| **이름 변경** (참조하는 곳 전부 같이 변경) | `Shift + F6` | `⇧ F6` |
| 변수로 추출 | `Ctrl + Alt + V` | `⌘ ⌥ V` |
| 메서드로 추출 | `Ctrl + Alt + M` | `⌘ ⌥ M` |
| 필드로 추출 | `Ctrl + Alt + F` | `⌘ ⌥ F` |
| 파라미터로 추출 | `Ctrl + Alt + P` | `⌘ ⌥ P` |
| 인라인 (추출의 반대) | `Ctrl + Alt + N` | `⌘ ⌥ N` |
| 메서드 시그니처 변경 | `Ctrl + F6` | `⌘ F6` |

파일 이름을 직접 고치지 말고 `Shift + F6`을 쓴다. 클래스명·파일명·모든 참조가 한 번에 바뀐다.

## 5. 이동 (Navigation)

| 기능 | Windows | Mac |
|---|---|---|
| 선언부로 이동 | `Ctrl + B` 또는 `Ctrl + 클릭` | `⌘ B` |
| 구현체로 이동 (인터페이스 → 구현 클래스) | `Ctrl + Alt + B` | `⌘ ⌥ B` |
| 사용처 전부 찾기 | `Alt + F7` | `⌥ F7` |
| 뒤로 가기 / 앞으로 가기 | `Ctrl + Alt + ←` / `→` | `⌘ ⌥ ←` / `→` |
| 마지막 편집 위치로 | `Ctrl + Shift + Backspace` | `⌘ ⇧ ⌫` |
| 최근 열었던 파일 | `Ctrl + E` | `⌘ E` |
| 클래스로 이동 | `Ctrl + N` | `⌘ O` |
| 파일로 이동 | `Ctrl + Shift + N` | `⌘ ⇧ O` |
| 현재 파일의 메서드 목록 | `Ctrl + F12` | `⌘ F12` |
| 특정 줄 번호로 이동 | `Ctrl + G` | `⌘ L` |

인터페이스에서 `Ctrl + Alt + B`는 스프링 공부할 때 특히 많이 쓴다.
`MemberRepository`에서 눌러 `MemoryMemberRepository`로 바로 간다.

## 6. 검색

| 기능 | Windows | Mac |
|---|---|---|
| 현재 파일에서 찾기 | `Ctrl + F` | `⌘ F` |
| 현재 파일에서 바꾸기 | `Ctrl + R` | `⌘ R` |
| **전체 프로젝트에서 찾기** | `Ctrl + Shift + F` | `⌘ ⇧ F` |
| 전체 프로젝트에서 바꾸기 | `Ctrl + Shift + R` | `⌘ ⇧ R` |
| 다음 / 이전 검색 결과 | `F3` / `Shift + F3` | `⌘ G` / `⌘ ⇧ G` |

## 7. 편집

| 기능 | Windows | Mac |
|---|---|---|
| **코드 정렬(포맷)** | `Ctrl + Alt + L` | `⌘ ⌥ L` |
| import 정리 | `Ctrl + Alt + O` | `⌃ ⌥ O` |
| 줄 복제 | `Ctrl + D` | `⌘ D` |
| 줄 삭제 | `Ctrl + Y` | `⌘ ⌫` |
| 줄 이동 | `Alt + Shift + ↑` / `↓` | `⌥ ⇧ ↑` / `↓` |
| 주석 토글 (`//`) | `Ctrl + /` | `⌘ /` |
| 블록 주석 토글 (`/* */`) | `Ctrl + Shift + /` | `⌘ ⌥ /` |
| 선택 영역 넓히기 / 좁히기 | `Ctrl + W` / `Ctrl + Shift + W` | `⌥ ↑` / `⌥ ↓` |
| **같은 단어 다중 선택** | `Alt + J` | `⌃ G` |
| 커서 여러 개 놓기 | `Alt + 클릭` | `⌥ 클릭` |
| 클립보드 히스토리 | `Ctrl + Shift + V` | `⌘ ⇧ V` |

## 8. 디버깅

| 기능 | Windows | Mac |
|---|---|---|
| 디버그 실행 | `Shift + F9` | `⌃ D` |
| 브레이크포인트 토글 | `Ctrl + F8` | `⌘ F8` |
| 한 줄 실행 (Step Over) | `F8` | `F8` |
| 메서드 안으로 (Step Into) | `F7` | `F7` |
| 메서드 빠져나오기 (Step Out) | `Shift + F8` | `⇧ F8` |
| 다음 브레이크포인트까지 | `F9` | `⌘ ⌥ R` |
| 식 계산 (Evaluate Expression) | `Alt + F8` | `⌥ F8` |

## 9. 창 · 화면

| 기능 | Windows | Mac |
|---|---|---|
| 프로젝트 창 열기/포커스 | `Alt + 1` | `⌘ 1` |
| 터미널 | `Alt + F12` | `⌥ F12` |
| Git 창 | `Alt + 9` | `⌘ 9` |
| 실행 결과 창 | `Alt + 4` | `⌘ 4` |
| 모든 도구창 숨기고 코드만 | `Ctrl + Shift + F12` | `⌘ ⇧ F12` |
| 설정 열기 | `Ctrl + Alt + S` | `⌘ ,` |
| 탭 닫기 | `Ctrl + F4` | `⌘ W` |

## 10. 문서 · 정보 보기

| 기능 | Windows | Mac |
|---|---|---|
| 문서(Javadoc) 미리보기 | `Ctrl + Q` | `F1` |
| 메서드 파라미터 정보 | `Ctrl + P` | `⌘ P` |
| 정의 미리보기 (창 이동 없이) | `Ctrl + Shift + I` | `⌥ Space` |
| 에러/경고 설명 보기 | `Ctrl + F1` | `⌘ F1` |
| 다음 에러로 이동 | `F2` | `F2` |

---

## 단축키 확인 · 변경하는 법

`File > Settings > Keymap` (`Ctrl + Alt + S`)

- 검색창에 기능 이름(`Create Test`)을 치면 현재 배정된 키가 보인다
- 우클릭 → `Add Keyboard Shortcut`으로 직접 배정할 수 있다
- 상단 드롭다운에서 keymap 종류(Windows / Eclipse / Visual Studio)를 바꿀 수 있다.
  단축키가 통째로 다르게 동작하면 여기가 원인일 확률이 높다

`Help > Keyboard Shortcuts PDF`로 공식 치트시트 전체를 받을 수도 있다.
