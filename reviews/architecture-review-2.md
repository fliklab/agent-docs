# Architecture Review v2

## Source
- Based on: `architecture-review.md`
- Focus: JSON result + actionable insights

## Structured JSON
```json
{
  "verdict": "A solid, modular foundation for a text/pixel-based 2D game, but needs critical performance optimizations in its core loops (event queue and rendering) before it can scale to complex games.",
  "score": 7,
  "pros": [
    "Clean modular architecture with clear separation of concerns (Store, Renderer, EventQueue, Transition Manager).",
    "Predictable state management using a Redux-like unidirectional data flow.",
    "Robust Event Queue implementation featuring priority levels, TTL, retries, and a dead-letter queue.",
    "Strongly typed with TypeScript, reducing runtime errors."
  ],
  "cons": [
    "Performance bottleneck in EventQueue: uses `Array.prototype.shift()` which is O(n) and will cause frame drops if the queue grows large.",
    "Inefficient rendering: `PixelRenderer` relies heavily on string manipulation, array mapping, and regex (`replace(/./g, '·')`) which generate significant garbage collection overhead per frame.",
    "Lack of advanced rendering capabilities (currently just outputs arrays of strings)."
  ],
  "suggestions": [
    "Replace arrays in `GlobalEventQueue` with a Ring Buffer or Linked List to achieve O(1) dequeue performance.",
    "Optimize `PixelRenderer` by using a pre-allocated 1D array (or TypedArray) and manipulating characters in-place instead of creating new strings and arrays every frame.",
    "Ensure `Store` listeners are strictly managed to prevent memory leaks if components mount/unmount frequently."
  ],
  "rating": "7/10"
}
```

## Insights

### 1) Immediate priority: hot-path performance
- **Event dequeue (`shift`) 교체가 최우선**입니다. 프레임 루프에 가까운 경로에서 O(n) 연산은 부하 시 프레임 드랍을 유발합니다.
- 최소 변경으로는 내부 큐 구현만 교체하고 외부 API는 유지하는 방식이 안전합니다.

### 2) Rendering cost control
- 현재 렌더링 파이프라인은 프레임마다 문자열/배열 재생성이 많아 GC 부담이 커질 수 있습니다.
- **사전 할당 버퍼(1D 배열/TypedArray)** + in-place 갱신 전략을 도입하면 프레임 안정성이 개선됩니다.

### 3) Reliability already strong, observability gap remains
- TTL, retries, dead-letter 등 신뢰성 장치는 좋은 편입니다.
- 다만 TTL 드롭/핸들러 미등록 이벤트에 대한 계측이 부족하면 운영 중 원인 분석이 어려워집니다.

### 4) Architectural direction
- 모듈 분리와 단방향 상태 흐름은 유지 가치가 높습니다.
- 다음 단계는 기능 확장보다 **코어 루프 성능 최적화 + 진단 지표 추가**가 ROI가 높습니다.

## Recommended next actions
1. Queue 내부 구현을 Ring Buffer(또는 Deque)로 교체하고 동일 테스트 스위트로 회귀 검증
2. Renderer에 preallocated buffer 경로를 추가하고 프레임당 할당량 측정 지표 도입
3. 이벤트 드롭/미처리 카운터(원인 코드 포함) 메트릭 추가
4. 리스너 lifecycle(등록/해제) 가이드와 테스트 케이스 보강
