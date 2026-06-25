# 발표 본문 슬라이드 목업 (Canva 재현용)

Canva 빈 본문 템플릿에 채울 내용을 미리 디자인한 **목업 24장**입니다. 브라우저로 열어 보고, Canva에서 동일하게 재현합니다.
(타이틀·목차·섹션 divider는 이미 Canva에서 완성 — 여기서는 다루지 않음)

## 전체 보기
- **[index.html](index.html)** — 24장을 축소 그리드로 한눈에
- **[no-home-mockups.pptx](../no-home-mockups.pptx)** — 25장을 16:9 PPTX로 변환(이격 없이 full-bleed 이미지 복원, **편집 불가**)
- **[no-home-mockups-editable.pptx](../no-home-mockups-editable.pptx)** — 25장을 **요소별 객체(도형·텍스트박스·이미지)로 분리** — 개별 선택·편집 가능

## 두 가지 PPTX
| 파일 | 특성 | 용도 |
| --- | --- | --- |
| `no-home-mockups.pptx` | 슬라이드 1장 = 이미지 1장(픽셀 동일) | 시안 그대로 보기·공유 |
| `no-home-mockups-editable.pptx` | 요소별 도형·텍스트박스·이미지 분리 | PowerPoint/Canva에서 직접 편집 |

### 편집판 변환 방식 (`_build_editable_pptx.py`)
1. 미리보기(localhost:8123, docroot=docs)에서 `index.html`의 25개 iframe 내부 요소를 추출 → bbox·색·폰트·텍스트 → `_editable_data.json`
2. python-pptx로 박스=도형, 텍스트=텍스트박스, 스크린샷=이미지 재구성 (px→EMU ×9525, fs px→pt ×0.75)
3. 한계: 강조색은 단일화(inline `<b>` 색 손실), 폰트는 `Malgun Gothic` 임시 → **Canva/PowerPoint에서 210 다락방·나눔스퀘어로 교체 및 미세조정** 권장
4. 재생성: `python _build_editable_pptx.py` (입력 `_editable_data.json`)

## PPTX 변환 (이격 없이 그대로 복원)
HTML 렌더를 픽셀 그대로 PPTX로 추출합니다.
1. Chrome headless로 각 슬라이드 캡처: `--window-size=1328,768 --force-device-scale-factor=2`(슬라이드 1280×720 + body 여백 24px)
2. 슬라이드 영역만 crop `(48,48)~(2608,1488)` = 2560×1440(@2x)
3. `python _build_pptx.py` → 16:9(13.333×7.5") 슬라이드에 full-bleed(l=0,t=0) 삽입 → `../no-home-mockups.pptx`

> 재생성: 위 캡처 후 `python _build_pptx.py`. 중간 PNG(`_pptx_build/`)는 gitignore. 슬라이드는 이미지라 텍스트 편집 불가 — 수정은 HTML을 고쳐 재변환.

## 슬라이드 목록 (섹션 9 구조)

| 섹션 | 파일 |
| --- | --- |
| 1 기획 배경·목표 | [11-problem](11-problem.html) · [12-required](12-required.html) · [13-ai-intro](13-ai-intro.html) |
| 2 추진 계획 | [21-milestone](21-milestone.html) · [22-team](22-team.html) |
| 3 시장 분석 | [31-competitor](31-competitor.html) · [32-differentiation](32-differentiation.html) |
| 4 개발 결과 | [41-required-status](41-required-status.html) · [42-search](42-search.html) · [43-member](43-member.html) · [44-ai-assistant](44-ai-assistant.html) · [45-verification](45-verification.html) |
| 5 개발 환경·구조도 | [51-architecture](51-architecture.html) · [52-stack](52-stack.html) |
| 6 화면 흐름·시연 | [61-flow](61-flow.html) · [62-demo](62-demo.html) |
| 7 패턴·알고리즘 | [71-toolcalling](71-toolcalling.html) · [72-capability](72-capability.html) · [73-memory](73-memory.html) · [74-performance](74-performance.html) · [75-security](75-security.html) |
| 8 기대 효과 | [81-impact](81-impact.html) |
| 9 개발 후기 | [91-troubleshooting](91-troubleshooting.html) · [92-retro](92-retro.html) · [93-closing](93-closing.html) |

> 공통 스타일은 [shared.css](shared.css). 각 HTML 하단에 **Canva 재현 메모** 주석 포함. 1280×720(16:9).

## 디자인 토큰 (캡처 기준)

| 용도 | 값 |
| --- | --- |
| 배경 | `#F7F5EF` (크림) |
| 프레임 노랑(좌상 L) | `#F5C84C` |
| 프레임 하늘(우·하단) | `#82BEF5` |
| 파랑 강조·번호 | `#3E8EDE` (텍스트 위 `#1F5A92`) |
| 노랑 번호/강조 | `#F5C84C` |
| 텍스트 | `#2E2E3A` (본문 보조 `#5F5E5A`) |
| 점선 구분선 | `#C9C7BF` |
| 상태색 | ✓ 초록 `#1D9E75` · △ 주황 `#E0982B` · ✕ 회색 `#C9C7BF` |

- **번호 뱃지**: 원형, **파랑·노랑 교차**, 흰 숫자 / **프레임 두께** 14px(상단 좌 62% 노랑 / 우 38% 하늘)
- **폰트**: 제목 **210 다락방**, 본문 **나눔스퀘어** — 목업은 유사 웹폰트 **Do Hyeon / Gothic A1**로 렌더(실제 Canva는 지정 폰트)
- 카드·pill·강조띠는 둥근 모서리 + 부드러운 그림자(캐릭터 로고의 둥근 톤과 통일)

## 워크플로
1. `index.html`로 전체 확인 → 개별 HTML로 디테일 확인
2. Canva 빈 본문 템플릿에 동일 배치(텍스트박스·도형·아이콘·스크린샷) — 재현 메모 참조
3. 스크린샷은 `../../screens/`, 다이어그램은 `../../diagrams/` 자산 활용 (mockups 기준 상대경로 — `docs/`까지 두 단계 상위)

> 미리보기: 정적 서버 docroot를 `docs/`로 두고 `presentation/mockups/index.html`로 열어야 스크린샷(`/screens/`)이 보입니다.

## 팀이 채울 것
- 경쟁 비교(31) 타사 값 최종 확인 · 팀 사진/개인 회고(92) · 이모지→Canva 스티커 대체(선택)
