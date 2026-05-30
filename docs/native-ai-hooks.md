# 네이티브 AI 훅

네이티브 Rust 모드는 두 가지 AI 표면을 맞춤화할 수 있습니다:

- 금지/선택 후보 점수 매기기.
- 게임 내 플레이어 AI가 생성하는 최종 플레이어 입력.

이 훅은 게임의 내부 AI 계획 유형이 아니라 공개된 문맥과 입력 유형을 노출합니다. 전략, 작전, 소행동 계획과 같은 내부 유형은 구현 세부 사항이며, 모드 API의 일부가 되지 않은 채 변경될 수 있습니다. 모드에서 내부 소행동과 일치하는 동작이 필요할 경우, `PlayerAiContext`의 도우미 메서드를 사용하여 공개 `Input`을 요청하십시오.

## AI 훅 등록

네이티브 모드 진입 함수에서 AI 훅을 등록하십시오:

```rust
use mod_api::*;

const MOD_ID: &str = "my_ai_mod";

fn init(_ctx: &GameCtx) -> ModRegistration {
    let mut reg = ModRegistration::new(MOD_ID);
    reg.add_draft_score_hook(InvertDraftScoreHook);
    reg.add_player_input_ai(LowHpRecallInputAi::default());
    reg
}

declare_mod!(init);
```

여러 모드가 훅을 등록하면 `priority()` 값이 낮은 것이 먼저 실행됩니다. 우선순위가 높은 훅은 나중에 실행되며 앞선 훅이 생성한 결과를 대체할 수 있습니다.

기본 제공되는 예시 네이티브 모드는 두 훅을 모두 보여 줍니다: 드래프트 점수를 반전시키고, HP가 낮은 플레이어 입력을 도주 또는 귀환 입력으로 대체합니다.

## 금지/선택 점수 훅

드래프트 AI가 금지 또는 선택 후보의 점수를 매기는 방식을 조정하려면 `ModDraftScoreHook`를 구현하십시오.

게임은 먼저 기본 점수를 계산합니다. 그다음 이 훅이 현재 점수를 받아 `DraftScoreDecision`을 반환합니다:

- `DraftScoreDecision::Pass`: 점수를 변경하지 않고 유지합니다.
- `DraftScoreDecision::Add(delta)`: 점수 증감값을 더합니다.
- `DraftScoreDecision::Replace(score)`: 점수를 대체합니다.

드래프트 AI는 여전히 최종 점수가 가장 높은 대상을 선택합니다.

예시: 모든 금지 및 선택 점수를 반전시킵니다.

```rust
#[derive(Debug)]
struct InvertDraftScoreHook;

impl ModDraftScoreHook for InvertDraftScoreHook {
    fn id(&self) -> &str {
        "my_ai_mod:invert_draft_score"
    }

    fn score_ban(
        &self,
        _ctx: &DraftScoreContext,
        _candidate: usize,
        current_score: f32,
    ) -> DraftScoreDecision {
        DraftScoreDecision::Replace(-current_score)
    }

    fn score_pick(
        &self,
        _ctx: &DraftScoreContext,
        _candidate: usize,
        current_score: f32,
    ) -> DraftScoreDecision {
        DraftScoreDecision::Replace(-current_score)
    }
}
```

`DraftScoreContext`에는 단계, 기존 선택 및 금지, 사용 가능한 후보, 난이도, 그리고 게임이 드래프트 대안을 탐색 중인지와 같은 드래프트 측 정보가 포함됩니다. 후보 값은 게임 드래프트 후보 목록에서 사용하는 챔피언 인덱스입니다.

## 플레이어 입력 AI 훅

플레이어에게 최종 선택된 `Input`을 교체하려면 `ModPlayerInputAi`를 구현하십시오.

이 훅은 게임의 일반 플레이어 AI가 기본 입력을 생성한 뒤에 실행됩니다. 반환값:

- `PlayerInputDecision::Pass`: 기본 입력을 유지합니다.
- `PlayerInputDecision::Replace(input)`: 대신 당신의 입력을 사용합니다.

교체 입력이 현재 프레임에 유효하지 않으면, 게임은 이를 무시하고 일반 동작을 유지합니다. 입력을 수동으로 만들 때는 `ctx.is_valid_input(&input)`을 호출하십시오.

예시: HP가 50% 미만이면, 귀환이 안전해질 때까지 도망치고, 그다음 귀환합니다.

```rust
#[derive(Clone, Debug, Default)]
struct LowHpRecallInputAi;

impl ModPlayerInputAi for LowHpRecallInputAi {
    fn clone_box(&self) -> Box<dyn ModPlayerInputAi> {
        Box::new(self.clone())
    }

    fn id(&self) -> &str {
        "my_ai_mod:low_hp_recall"
    }

    fn matches(&self, _ctx: &PlayerAiInitContext) -> bool {
        true
    }

    fn think(
        &mut self,
        ctx: &mut PlayerAiContext<'_, '_, '_>,
        _base_input: Option<Input>,
    ) -> PlayerInputDecision {
        if !ctx.is_hp_below_percent(50) {
            return PlayerInputDecision::Pass;
        }

        if ctx.is_safe_to_recall() {
            if let Some(input) = ctx.get_recall_input() {
                return PlayerInputDecision::Replace(input);
            }
        }

        if let Some(input) = ctx.get_run_away_input() {
            return PlayerInputDecision::Replace(input);
        }

        PlayerInputDecision::Pass
    }
}
```

위의 도우미 호출은 내부의 세부 행동 객체를 노출하지 않습니다. 예를 들어 `get_run_away_input()`은 게임에 해당 내부 도주 동작을 생성하도록 요청하고, 그 결과인 공개 `Input`만 반환합니다.

## 플레이어 AI 컨텍스트

`PlayerAiInitContext`는 AI 인스턴스가 플레이어에 연결될 때 `matches()`에 전달됩니다. 이를 사용해 훅을 팀, 위치, 선수 또는 챔피언으로 제한하십시오.

`PlayerAiContext`는 각 시뮬레이션 업데이트마다 전달됩니다. 이는 안정적인 읽기 전용 상태와 도우미 함수를 제공하며, 다음을 포함합니다:

- `player_id()`, `athlete_id()`, `team()`, `position()`, 및 `champion_name()`.
- `tick()`, `hp()`, `max_hp()`, `hp_ratio_percent()`, 및 `is_hp_below_percent(percent)`.
- `is_safe_to_recall()`.
- `get_recall_input()`.
- `get_run_away_input()`.
- `get_run_away_without_skill_input()`.
- `is_valid_input(&input)`.

`is_safe_to_recall()`은 귀환 결정을 위한 게임 제공 유틸리티입니다. 이는 동작 선택을 위한 것이며, 영구적인 밸런스 계약으로 의도된 것은 아닙니다. 정확한 휴리스틱은 기본 AI와 함께 변경될 수 있습니다.

## 호환성 참고사항

- 게임 SDK가 변경될 때마다 네이티브 DLL을 다시 빌드하십시오.
- 경계로 `Input` 타입과 컨텍스트 도우미를 사용하십시오. 내부 AI 구현 세부사항에 의존하지 마십시오.
- 후크는 작고 결정론적으로 유지하십시오. 이들은 경기 시뮬레이션 내부에서 실행됩니다.
- 후크에서 패닉이 발생하면 게임은 오류를 기록하고 가능한 경우 대체 처리로 돌아가지만, 후크는 여전히 자체적인 예외 상황을 처리해야 합니다.
- 릴리스 게임 패키지에는 릴리스 빌드된 DLL을 사용하십시오.