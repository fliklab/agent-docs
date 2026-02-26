# Code Style & Working Plan — 2D Pixel Game Framework

이 문서는 `2d-pixel-game-framework` 저장소의 코딩 스타일/아키텍처/작업 절차 규칙을 정리한다.

## 1) Coding / Working Style
- TypeScript 우선, 명시적 타입 사용 (`any` 최소화).
- 작은 단위 커밋, 작은 단위 함수/모듈.
- 파일/폴더/심볼 네이밍 일관성 유지.
- 신규 코드에는 최소한의 사용 예시 또는 주석 추가.
- 기존 동작 변경 시 변경 이유를 PR/커밋 메시지에 명확히 기록.

## 2) 아키텍처: FSD 기반
Feature-Sliced Design(FSD) 계층을 기본으로 사용한다.

권장 디렉토리(예시):
- `src/app` : 앱 부트스트랩, 전역 설정
- `src/processes` : 런타임 프로세스(필요 시)
- `src/pages` : 화면 단위 구성
- `src/widgets` : 복합 UI 블록
- `src/features` : 사용자 액션/게임 기능 단위
- `src/entities` : 도메인 엔티티(플레이어, 맵, 아이템 등)
- `src/shared` : 공통 유틸, 공통 UI, 상수, 타입

규칙:
- 상위 레이어는 하위 레이어를 참조 가능, 역참조 금지.
- `shared`는 도메인 지식 최소화.
- 엔티티 간 직접 결합 대신 인터페이스/이벤트를 통한 연결 우선.

## 3) Low Coupling 원칙
- 모듈 간 결합도를 낮추고 응집도를 높인다.
- 외부 의존성은 어댑터/포트 패턴으로 격리.
- 상태 접근은 공개 API를 통해서만 수행.
- 사이드이펙트는 경계 레이어(예: app/processes)로 제한.

## 4) util 작성 규칙
- 범용 로직은 `src/shared/lib` 또는 `src/shared/utils`에 배치.
- 게임 도메인 특화 유틸은 해당 `entities/*/lib` 또는 `features/*/lib`에 배치.
- util은 가능하면 순수 함수로 작성하고 테스트 가능하게 유지.
- 재사용 가능성이 낮은 로직은 util로 승격하지 않는다.

## 5) 작업 절차
1. `requirements.md` 확인
2. 작업 단위를 잘게 분리
3. 설계/구현
4. 검증(타입체크/테스트/실행 확인)
5. 문서 업데이트
6. 커밋
