# Native Mod API 참고 문서

이 페이지에는 네이티브 DLL 모드를 위해 `mod_api` 크레이트에서 내보내는 Rust API가 나열되어 있습니다.
대부분의 모드는 다음과 같이 시작합니다:

```rust
use mod_api::*;
```

여기에서 설명하는 API 버전은 `API_VERSION == (0, 7)`입니다. 네이티브 DLL은 SDK 버전에 종속되므로 게임 SDK가 변경되면 DLL을 다시 빌드해야 합니다.

## 최상위 내보내기 맵

`mod_api`는 다음 모듈을 다시 내보냅니다:

- `registration`의 등록 및 진입점.
- `traits`의 네이티브 콘텐츠 트레이트.
- `game_ctx`의 시뮬레이션 접근 래퍼.
- `handles`의 불투명 핸들.
- `extension`의 클라이언트 생명주기 훅 및 UI/장면 형식.
- `game-core`의 일부 게임 값 형식.

`Team`, `Athlete`, `MatchInfo`, `ChampionInfoSheet`와 같은 대규모 게임 데이터 구조의 경우, 내보내는 형식은 일치하는 SDK의 실제 게임 형식입니다. 대상 게임과 일치하는 SDK 버전을 사용하십시오.

## 등록

### 상수 및 형식

| 항목 | 시그니처 |
| --- | --- |
| `API_VERSION` | `(u32, u32)`, 현재 `(0, 7)` |
| `API_VERSION_ENCODED` | `major << 32 | minor`로 인코딩된 `u64` |
| `MOD_ENTRY_SYMBOL` | `"tfm2_mod_entry"` |
| `MOD_API_VERSION_SYMBOL` | `"tfm2_mod_api_version"` |
| `ModEntryFn` | `unsafe extern "C" fn(api: *const GameCtx) -> *mut ModRegistration` |
| `ModApiVersionFn` | `unsafe extern "C" fn() -> u64` |
| `decode_api_version(raw)` | `fn decode_api_version(raw: u64) -> (u32, u32)` |

`declare_mod!(init_fn)`을 사용하여 두 DLL 심볼을 모두 내보냅니다:

```rust
fn init(ctx: &GameCtx) -> ModRegistration {
    let mut reg = ModRegistration::new("my_mod");
    reg.add_item(MyItem::default());
    reg
}

declare_mod!(init);
```

### `ModRegistration`

항목:

| 필드 | 형식 |
| --- | --- |
| `mod_id` | `String` |
| `api_version` | `(u32, u32)` |
| `champions` | `Vec<Box<dyn ModChampionInfo>>` |
| `items` | `Vec<Box<dyn ModItemInfo>>` |
| `draft_score_hooks` | `Vec<Box<dyn ModDraftScoreHook>>` |
| `player_input_ai` | `Vec<Box<dyn ModPlayerInputAi>>` |
| `extension` | `Option<Box<dyn ModExtension>>` |
| `server_extension` | `Option<Box<dyn ModServerExtension>>` |

메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `new` | `fn new(mod_id: impl Into<String>) -> Self` |
| `add_champion` | `fn add_champion(&mut self, info: impl ModChampionInfo + 'static)` |
| `add_item` | `fn add_item(&mut self, info: impl ModItemInfo + 'static)` |
| `add_draft_score_hook` | `fn add_draft_score_hook(&mut self, hook: impl ModDraftScoreHook + 'static)` |
| `add_player_input_ai` | `fn add_player_input_ai(&mut self, ai: impl ModPlayerInputAi + 'static)` |
| `set_extension` | `fn set_extension(&mut self, ext: impl ModExtension + 'static)` |
| `set_server_extension` | `fn set_server_extension(&mut self, ext: impl ModServerExtension + 'static)` |

## 챔피언 및 전투 특성

### `ModChampionInfo`

하나의 챔피언을 정의합니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `id` | `fn id(&self) -> &str` | 필수 |
| `name` | `fn name(&self) -> &str` | `self.id()` |
| `skill_icon` | `fn skill_icon(&self, skill_index: usize) -> (String, String)` | `skill_icon` 시트 및 `{id}_{index}` 태그 |
| `category` | `fn category(&self) -> ChampionCategory` | 필수 |
| `tags` | `fn tags(&self) -> Vec<ChampionTag>` | 필수 |
| `stat` | `fn stat(&self) -> EntityStat` | 필수 |
| `growth` | `fn growth(&self) -> EntityStat` | 필수 |
| `attack` | `fn attack(&self) -> Box<dyn ModAction>` | 필수 |
| `skill` | `fn skill(&self) -> Box<dyn ModAction>` | 필수 |
| `skill2` | `fn skill2(&self) -> Box<dyn ModAction>` | 필수 |
| `ult` | `fn ult(&self) -> Option<Box<dyn ModAction>>` | `None` |
| `passive` | `fn passive(&self) -> Option<Box<dyn ModPassive>>` | `None` |

### `ModAction`

챔피언이 사용하는 행동을 정의합니다: attack, skill, skill2 또는 ult.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `clone_box` | `fn clone_box(&self) -> Box<dyn ModAction>` | 필수 |
| `action_name` | `fn action_name(&self) -> &str` | `"attack"` |
| `duration` | `fn duration(&self) -> usize` | 필수 |
| `cancelable` | `fn cancelable(&self) -> bool` | `false` |
| `cooltime` | `fn cooltime(&self, caster_stat: &EntityStat, caster_level: usize) -> usize` | 필수 |
| `casting_target` | `fn casting_target(&self) -> CastingTarget` | 필수 |
| `effect` | `fn effect(&self) -> Option<ModEffect>` | 필수 |
| `cooltime_use_count` | `fn cooltime_use_count(&self, caster_stat: &EntityStat) -> usize` | `1` |
| `can_use_with_move` | `fn can_use_with_move(&self) -> bool` | `false` |
| `description` | `fn description(&self) -> String` | 없음 |

### `ModEffect`

`ModAction::effect()`가 반환합니다.

| 필드 | 형식 |
| --- | --- |
| `range` | `u64` |
| `growth_range` | `u64` |
| `start_timing` | `usize` |
| `casting` | `CastingType` |
| `target` | `CastingTarget` |
| `attack_type` | `AttackType` |
| `effect_type` | `Box<dyn ModEffectType>` |

### `ModEffectType`

효과가 발동할 때 무엇을 수행하는지 정의합니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `apply` | `fn apply(&self, ctx: &mut GameCtx, rng_seed: u64, caster_id: usize, input: InputTarget)` | 필수 |
| `expected_damage` | `fn expected_damage(&self, caster_stat: &EntityStat) -> (usize, usize)` | `(0, 0)` |
| `expected_heal` | `fn expected_heal(&self, caster_stat: &EntityStat) -> usize` | `0` |
| `expected_shield` | `fn expected_shield(&self, caster_stat: &EntityStat) -> usize` | `0` |
| `expected_cc_time` | `fn expected_cc_time(&self) -> Option<usize>` | `None` |
| `expected_buff` | `fn expected_buff(&self, caster_stat: &EntityStat) -> Option<BuffState>` | `None` |
| `expected_move_distance` | `fn expected_move_distance(&self) -> Option<(usize, u64)>` | `None` |
| `expected_rush_effect` | `fn expected_rush_effect(&self) -> bool` | `false` |
| `auto_target` | `fn auto_target(&self) -> bool` | `false` |
| `on_caster` | `fn on_caster(&self) -> bool` | `false` |
| `can_move` | `fn can_move(&self) -> bool` | `false` |
| `linear_move_speed` | `fn linear_move_speed(&self) -> Option<usize>` | `None` |

### `ModPassive`

선택형 챔피언 패시브입니다. 엔진이 런타임 인스턴스를 복제하므로 `clone_box`를 구현해야 합니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `clone_box` | `fn clone_box(&self) -> Box<dyn ModPassive>` | 필수 |
| `on_spawn` | `fn on_spawn(&mut self, ctx: &mut GameCtx, player: usize, entity: usize)` | 아무 동작 안 함 |
| `on_attack` | `fn on_attack(&mut self, ctx: &mut GameCtx, player: usize, entity: usize, target: usize, damage: &mut usize)` | 아무 동작 안 함 |
| `on_damaged` | `fn on_damaged(&mut self, ctx: &mut GameCtx, player: usize, entity: usize, attacker: usize, damage: usize)` | 동작 없음 |
| `on_kill` | `fn on_kill(&mut self, ctx: &mut GameCtx, player: usize, entity: usize)` | 동작 없음 |
| `on_update` | `fn on_update(&mut self, ctx: &mut GameCtx, rng_seed: u64, player: usize, entity: usize)` | 동작 없음 |
| `on_base_attack` | `fn on_base_attack(&mut self, ctx: &mut GameCtx, rng_seed: u64, player: usize, entity: usize)` | 동작 없음 |
| `on_assist` | `fn on_assist(&mut self, ctx: &mut GameCtx, player: usize, entity: usize)` | 동작 없음 |
| `on_dead` | `fn on_dead(&mut self, ctx: &mut GameCtx, player: usize)` | 동작 없음 |

### `ModEffectBuff`

사용자 지정 지속형 버프 로직 유형입니다. API 표면의 일부이지만, `ModRegistration`을 통해 직접 등록되지는 않습니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `on_damaged` | `fn on_damaged(&self, ctx: &mut GameCtx, attacker: usize, target: usize, damage: &mut usize, attack_type: AttackType)` | 동작 없음 |
| `update` | `fn update(&mut self, ctx: &mut GameCtx, rng_seed: u64)` | 필수 |
| `is_end` | `fn is_end(&self, ctx: &GameCtx) -> bool` | 필수 |

## 아이템 트레이트

### `ModItemInfo`

아이템과 해당 런타임 콜백을 정의합니다. 게임이 서로 독립적인 런타임 인스턴스를 생성하므로 `clone_box`를 구현해야 합니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `clone_box` | `fn clone_box(&self) -> Box<dyn ModItemInfo>` | 필수 |
| `key` | `fn key(&self) -> &str` | 필수 |
| `icon` | `fn icon(&self) -> &str` | `self.key()` |
| `price` | `fn price(&self) -> usize` | 필수 |
| `tier` | `fn tier(&self) -> usize` | 필수 |
| `stat` | `fn stat(&self) -> BuffState` | 필수 |
| `next_tier` | `fn next_tier(&self) -> Vec<String>` | 없음 |
| `previous_tier` | `fn previous_tier(&self) -> Vec<String>` | 없음 |
| `tags` | `fn tags(&self) -> Vec<ItemTag>` | 없음 |
| `category` | `fn category(&self) -> ItemCategory` | `ItemCategory::default()` |
| `on_attack` | `fn on_attack(&mut self, ctx: &mut GameCtx, caster: usize, target: usize, damage: &mut usize, damage_type: DamageType)` | 동작 없음 |
| `update` | `fn update(&mut self, ctx: &mut GameCtx, rng_seed: u64, player: usize)` | 동작 없음 |
| `on_spawn` | `fn on_spawn(&mut self, ctx: &mut GameCtx, player: usize)` | 동작 없음 |
| `on_healed` | `fn on_healed(&mut self, ctx: &mut GameCtx, caster: Option<usize>, entity: usize, heal: usize)` | 동작 없음 |
| `on_damaged` | `fn on_damaged(&mut self, ctx: &mut GameCtx, player: usize, entity: usize, attacker: usize, damage: usize)` | 동작 없음 |
| `on_kill` | `fn on_kill(&mut self, ctx: &mut GameCtx, rng_seed: u64, player: usize, entity: usize)` | 동작 없음 |
| `on_skill_hit` | `fn on_skill_hit(&mut self, ctx: &mut GameCtx, rng_seed: u64, caster: usize, target: usize)` | 동작 없음 |

## 시뮬레이션 맥락

### 핸들

이는 불투명 포인터 래퍼입니다. 핸들은 획득된 콜백 동안에만 유효합니다. 저장하지 마십시오.

| 유형 | 메서드 |
| --- | --- |
| `EntityHandle` | `null()`, `is_null()`, `unsafe from_ptr(ptr)`, `as_ptr()` |
| `PlayerHandle` | `null()`, `is_null()`, `unsafe from_ptr(ptr)`, `as_ptr()` |
| `ProjectileHandle` | `null()`, `is_null()`, `unsafe from_ptr(ptr)`, `as_ptr()` |

### `GameCtx`

`GameCtx`는 네이티브 콜백에 전달되는 주된 핸들입니다.

실행 환경 서비스 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `register_service` | `fn register_service(&self, service_id: &str, version: ModServiceVersion, service: ModService) -> bool` |
| `query_service` | `fn query_service(&self, provider_mod_id: &str, service_id: &str, version_req: &str) -> Option<ModService>` |

시뮬레이션 조회 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `tick` | `fn tick(&self) -> usize` |
| `seed` | `fn seed(&self) -> u64` |
| `score_diff` | `fn score_diff(&self, team: usize) -> i32` |
| `is_end` | `fn is_end(&self) -> bool` |
| `get_entity` | `fn get_entity(&self, id: usize) -> Option<EntityRef<'_>>` |
| `entity_count` | `fn entity_count(&self) -> usize` |
| `entity_at` | `fn entity_at(&self, index: usize) -> Option<EntityRef<'_>>` |
| `get_player` | `fn get_player(&self, id: usize) -> Option<PlayerRef<'_>>` |
| `champion_count` | `fn champion_count(&self) -> usize` |
| `champion_id_at` | `fn champion_id_at(&self, index: usize) -> usize` |
| `tower_count` | `fn tower_count(&self) -> usize` |
| `tower_id_at` | `fn tower_id_at(&self, index: usize) -> usize` |
| `player_count` | `fn player_count(&self) -> usize` |
| `player_at` | `fn player_at(&self, index: usize) -> Option<PlayerRef<'_>>` |
| `projectile_count` | `fn projectile_count(&self) -> usize` |
| `projectile_at` | `fn projectile_at(&self, index: usize) -> Option<ProjectileRef<'_>>` |
| `kill_log_count` | `fn kill_log_count(&self) -> usize` |
| `kill_log_at` | `fn kill_log_at(&self, index: usize) -> KillLogEntry` |
| `distance_sq` | `fn distance_sq(&self, id1: usize, id2: usize) -> u64` |
| `is_visible` | `fn is_visible(&self, team: usize, id: usize) -> bool` |

시뮬레이션 변경 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `deal_damage` | `fn deal_damage(&mut self, attacker: usize, target: usize, ad: usize, ap: usize, attack_type: AttackType)` |
| `heal` | `fn heal(&mut self, caster: usize, target: usize, amount: usize)` |
| `add_buff` | `fn add_buff(&mut self, target: usize, buff: BuffState)` |
| `apply_cc` | `fn apply_cc(&mut self, target: usize, cc: CCState)` |

디버그 그리기 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `debug_draw_line` | `fn debug_draw_line(&mut self, x1: u64, y1: u64, x2: u64, y2: u64, color: u32)` |
| `debug_draw_circle` | `fn debug_draw_circle(&mut self, x: u64, y: u64, r: u64, color: u32)` |

색상은 압축된 `u32` 값입니다. 기존 예시에서는 `0xff66ccff`와 같은 값을 사용합니다.

### `EntityRef`

| 메서드 | 시그니처 |
| --- | --- |
| `handle` | `fn handle(&self) -> EntityHandle` |
| `id` | `fn id(&self) -> usize` |
| `stat` | `fn stat(&self) -> EntityStat` |
| `pos` | `fn pos(&self) -> EntityPos` |
| `hp` | `fn hp(&self) -> EntityHp` |
| `team` | `fn team(&self) -> usize` |
| `level` | `fn level(&self) -> usize` |
| `is_alive` | `fn is_alive(&self) -> bool` |
| `is_champion` | `fn is_champion(&self) -> bool` |
| `is_tower` | `fn is_tower(&self) -> bool` |
| `is_minion` | `fn is_minion(&self) -> bool` |
| `shield` | `fn shield(&self) -> usize` |
| `buff_count` | `fn buff_count(&self) -> usize` |
| `buff_at` | `fn buff_at(&self, index: usize) -> BuffState` |
| `cc_count` | `fn cc_count(&self) -> usize` |
| `cc_at` | `fn cc_at(&self, index: usize) -> CCInfo` |
| `radius` | `fn radius(&self) -> usize` |
| `is_targetable` | `fn is_targetable(&self) -> bool` |

### `PlayerRef`

| 메서드 | 시그니처 |
| --- | --- |
| `handle` | `fn handle(&self) -> PlayerHandle` |
| `champion` | `fn champion(&self) -> Option<EntityRef<'_>>` |
| `level` | `fn level(&self) -> usize` |
| `gold` | `fn gold(&self) -> usize` |
| `position` | `fn position(&self) -> Position` |
| `team` | `fn team(&self) -> usize` |
| `is_alive` | `fn is_alive(&self) -> bool` |
| `respawn_time` | `fn respawn_time(&self) -> usize` |
| `kills` | `fn kills(&self) -> usize` |
| `deaths` | `fn deaths(&self) -> usize` |
| `assists` | `fn assists(&self) -> usize` |
| `cs` | `fn cs(&self) -> usize` |

### `ProjectileRef`

| 메서드 | 시그니처 |
| --- | --- |
| `handle` | `fn handle(&self) -> ProjectileHandle` |
| `info` | `fn info(&self) -> ProjectileInfo` |

### 반환 구조체

| 유형 | 필드 |
| --- | --- |
| `EntityPos` | `x: u64`, `y: u64` |
| `EntityHp` | `current: usize`, `max: usize` |
| `CCInfo` | `cc_type: u32`, `tick: u64` |
| `ProjectileInfo` | `x: u64`, `y: u64`, `caster_id: usize`, `team: usize`, `is_end: bool` |
| `KillLogEntry` | `tick: usize`, `killer_team: usize`, `killer_position: u32`, `killed_position: u32`, `assist_count: u32`, `assist_positions: [u32; 4]` |

`CCInfo::cc_type`는 현재 다음을 사용합니다: `0=Airborne`, `1=Stun`, `2=Bind`, `3=BlockAttack`, `4=BlockSkill`, `5=BlockMoveSkill`, `6=ForceMove`, `7=Taunt`, `8=Fear`, `9=Animation`, `255=Invalid`.

## 런타임 서비스

런타임 서비스는 한 네이티브 모드가 불투명한 vtable을 게시하고 다른 네이티브 모드가 이를 조회할 수 있게 합니다.

### `ModServiceVersion`

필드: `major: u32`, `minor: u32`, `patch: u32`.

메서드:

```rust
pub const fn new(major: u32, minor: u32, patch: u32) -> Self
```

### `ModService`

항목:

| 필드 | 형식 |
| --- | --- |
| `data` | `*mut c_void` |
| `vtable` | `*const c_void` |

메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `null` | `pub const fn null() -> Self` |
| `from_raw` | `pub const fn from_raw(data: *mut c_void, vtable: *const c_void) -> Self` |
| `is_null` | `pub fn is_null(&self) -> bool` |
| `vtable` | `pub unsafe fn vtable<T>(&self) -> Option<&T>` |

공급자 모드와 소비자 모드 사이에서 공유 `#[repr(C)]` vtable 계약을 사용합니다. `query_service`는 `">=1.0.0, <2.0.0"`와 같은 semver 요구사항 문자열을 받습니다.

## AI 훅

### `ModDraftScoreHook`

기본 드래프트 AI가 ban/pick 후보 점수를 매긴 뒤, 그 점수를 조정합니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `id` | `fn id(&self) -> &str` | 필수 |
| `priority` | `fn priority(&self) -> i32` | `0` |
| `score_ban` | `fn score_ban(&self, ctx: &DraftScoreContext, candidate: usize, base_score: f32) -> DraftScoreDecision` | `Pass` |
| `score_pick` | `fn score_pick(&self, ctx: &DraftScoreContext, candidate: usize, base_score: f32) -> DraftScoreDecision` | `Pass` |

낮은 우선순위가 먼저 실행되며, 높은 우선순위는 나중에 실행되어 앞선 훅이 반영한 점수를 확인합니다.

### `DraftScoreContext`

필드:

| 필드 | 형식 |
| --- | --- |
| `phase` | `DraftScorePhase` |
| `available_champions` | `&[usize]` |
| `ally_ban` | `&[usize]` |
| `enemy_ban` | `&[usize]` |
| `ally_pick` | `&[usize]` |
| `enemy_pick` | `&[usize]` |
| `is_explore` | `bool` |
| `difficulty` | `Difficulty` |

관련 enum:

```rust
pub enum DraftScorePhase {
    Ban,
    Pick,
}

pub enum DraftScoreDecision {
    Pass,
    Add(f32),
    Replace(f32),
}
```

### `ModPlayerInputAi`

기본 제공 플레이어 AI가 tick마다 생성하는 최종 `Input`을 대체합니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `clone_box` | `fn clone_box(&self) -> Box<dyn ModPlayerInputAi>` | 필수 |
| `id` | `fn id(&self) -> &str` | 필수 |
| `priority` | `fn priority(&self) -> i32` | `0` |
| `matches` | `fn matches(&self, ctx: &PlayerAiInitContext) -> bool` | `true` |
| `think` | `fn think(&mut self, ctx: &mut PlayerAiContext<'_, '_, '_>, base_input: Option<Input>) -> PlayerInputDecision` | 필수 |

`PlayerInputDecision`:

```rust
pub enum PlayerInputDecision {
    Pass,
    Replace(Input),
}
```

### `PlayerAiInitContext`

팔드:

| 필드 | 형식 |
| --- | --- |
| `player_id` | `usize` |
| `athlete_id` | `usize` |
| `team` | `usize` |
| `position` | `Position` |
| `champion_name` | `String` |

### `PlayerAiContext`

| 메서드 | 시그니처 |
| --- | --- |
| `player_id` | `fn player_id(&self) -> usize` |
| `athlete_id` | `fn athlete_id(&self) -> usize` |
| `team` | `fn team(&self) -> usize` |
| `position` | `fn position(&self) -> Position` |
| `champion_name` | `fn champion_name(&self) -> &str` |
| `tick` | `fn tick(&self) -> usize` |
| `hp` | `fn hp(&self) -> Option<usize>` |
| `max_hp` | `fn max_hp(&self) -> Option<usize>` |
| `hp_ratio_percent` | `fn hp_ratio_percent(&self) -> Option<usize>` |
| `is_hp_below_percent` | `fn is_hp_below_percent(&self, threshold: usize) -> bool` |
| `is_valid_input` | `fn is_valid_input(&self, input: &Input) -> bool` |
| `get_run_away_input` | `fn get_run_away_input(&mut self) -> Option<Input>` |
| `get_run_away_without_skill_input` | `fn get_run_away_without_skill_input(&mut self) -> Option<Input>` |
| `get_recall_input` | `fn get_recall_input(&mut self) -> Option<Input>` |
| `is_safe_to_recall` | `fn is_safe_to_recall(&mut self) -> bool` |

## 클라이언트 확장 API

### 다시 내보낸 클라이언트 및 UI 형식

`mod_api` 다시 내보내기:

- `Assets`
- `RenderState`
- `Node`
- `NodeTemplate`
- `UI`
- `UIEvent`
- `UIEventHandlerContext`
- `Scene`
- `UIOutEvent`
- `ClientData`
- `ClientDatabase`

별칭:

```rust
pub type GameUI = UI<(), UIOutEvent>;
```

### `ModExtension`

클라이언트/게임 루프 측에서 호출되는 생명주기 훅.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `on_init` | `fn on_init(&self, scene: &mut Scene, ui: &mut GameUI, assets: &mut Assets)` | 동작 없음 |
| `pre_update` | `fn pre_update(&self, scene: &mut Scene, ui: &mut GameUI, assets: &mut Assets, dt: f32)` | 동작 없음 |
| `post_update` | `fn post_update(&self, scene: &mut Scene, ui: &mut GameUI, assets: &mut Assets, dt: f32)` | 동작 없음 |
| `pre_render` | `fn pre_render(&self, scene: &Scene, ui: &GameUI, assets: &Assets, state: &mut RenderState)` | 동작 없음 |
| `post_render` | `fn post_render(&self, scene: &Scene, ui: &GameUI, assets: &Assets, state: &mut RenderState)` | 동작 없음 |
| `on_end` | `fn on_end(&self, assets: &Assets)` | 동작 없음 |

일반 운영 플레이 중 `ClientData`에 접근하려면 `Scene::InGame { data }`를 사용하십시오.

### `ClientData`

필드:

| 필드 | 형식 |
| --- | --- |
| `db` | `Rc<RefCell<ClientDatabase>>` |
| `main_tutorial` | `Option<MainTutorial>` |

대여 도우미:

| 메서드 | 시그니처 |
| --- | --- |
| `db` | `fn db(&self) -> Ref<'_, ClientDatabase>` |
| `db_mut` | `fn db_mut(&self) -> RefMut<'_, ClientDatabase>` |

공통 읽기 도우미:

| 메서드 | 시그니처 |
| --- | --- |
| `player_team_id` | `fn player_team_id(&self) -> usize` |
| `player_team` | `fn player_team(&self) -> Option<Ref<'_, Team>>` |
| `team` | `fn team(&self, team_id: usize) -> Option<Ref<'_, Team>>` |
| `team_ids` | `fn team_ids(&self) -> Vec<usize>` |
| `athlete` | `fn athlete(&self, athlete_id: usize) -> Option<Ref<'_, Athlete>>` |
| `athlete_ids` | `fn athlete_ids(&self) -> Vec<usize>` |
| `staff` | `fn staff(&self, staff_id: usize) -> Option<Ref<'_, Staff>>` |
| `staff_ids` | `fn staff_ids(&self) -> Vec<usize>` |
| `knowledge_base` | `fn knowledge_base(&self, team_id: usize) -> Option<Ref<'_, KnowledgeBase>>` |
| `league` | `fn league(&self, league_id: usize) -> Option<Ref<'_, League>>` |
| `league_ids` | `fn league_ids(&self) -> Vec<usize>` |
| `tournament` | `fn tournament(&self, tournament_id: usize) -> Option<Ref<'_, Tournament>>` |
| `tournament_ids` | `fn tournament_ids(&self) -> Vec<usize>` |
| `match_info` | `fn match_info(&self, match_type: MatchType) -> Option<Ref<'_, MatchInfo>>` |
| `normal_match` | `fn normal_match(&self, match_id: usize) -> Option<Ref<'_, MatchInfo>>` |
| `practice_match` | `fn practice_match(&self, match_id: usize) -> Option<Ref<'_, MatchInfo>>` |
| `tutorial_match` | `fn tutorial_match(&self, match_id: usize) -> Option<Ref<'_, MatchInfo>>` |
| `solo_rank_match_info` | `fn solo_rank_match_info(&self, match_id: usize) -> Option<Ref<'_, MatchInfo>>` |
| `match_replay` | `fn match_replay(&self, match_id: usize) -> Option<Ref<'_, MatchReplayData>>` |
| `match_replay_ids` | `fn match_replay_ids(&self) -> Vec<usize>` |
| `league_competition` | `fn league_competition(&self, competition_id: usize) -> Option<Ref<'_, LeagueCompetition>>` |
| `league_competition_ids` | `fn league_competition_ids(&self) -> Vec<usize>` |
| `tournament_competition` | `fn tournament_competition(&self, competition_id: usize) -> Option<Ref<'_, TournamentCompetition>>` |
| `tournament_competition_ids` | `fn tournament_competition_ids(&self) -> Vec<usize>` |
| `solo_rank_match` | `fn solo_rank_match(&self, match_id: usize) -> Option<Ref<'_, SoloRankMatch>>` |
| `solo_rank_match_ids` | `fn solo_rank_match_ids(&self) -> Vec<usize>` |
| `champion_info` | `fn champion_info(&self, champion_name: &str) -> Option<Arc<dyn ChampionInfo>>` |

클라이언트/서버 모드 메시지:

| 메서드 | 시그니처 |
| --- | --- |
| `send_mod_command` | `fn send_mod_command(&self, mod_id: &str, command: &str, payload: impl Into<Vec<u8>>) -> bool` |
| `mod_events` | `fn mod_events(&self, mod_id: &str) -> Vec<ModClientEvent>` |
| `take_mod_events` | `fn take_mod_events(&self, mod_id: &str) -> Vec<ModClientEvent>` |

저장 데이터 도우미:

| 메서드 | 시그니처 |
| --- | --- |
| `can_write_mod_save` | `fn can_write_mod_save(&self) -> bool` |
| `mod_save_version` | `fn mod_save_version(&self, mod_id: &str) -> usize` |
| `mod_save_set_version` | `fn mod_save_set_version(&self, mod_id: &str, version: usize) -> bool` |
| `mod_save_keys` | `fn mod_save_keys(&self, mod_id: &str) -> Vec<String>` |
| `mod_save_contains_key` | `fn mod_save_contains_key(&self, mod_id: &str, key: &str) -> bool` |
| `mod_save_get_bytes` | `fn mod_save_get_bytes(&self, mod_id: &str, key: &str) -> Option<Vec<u8>>` |
| `mod_save_set_bytes` | `fn mod_save_set_bytes(&self, mod_id: &str, key: &str, value: impl Into<Vec<u8>>) -> bool` |
| `mod_save_get_string` | `fn mod_save_get_string(&self, mod_id: &str, key: &str) -> Option<String>` |
| `mod_save_set_string` | `fn mod_save_set_string(&self, mod_id: &str, key: &str, value: impl Into<String>) -> bool` |
| `mod_save_remove_key` | `fn mod_save_remove_key(&self, mod_id: &str, key: &str) -> bool` |
| `mod_save_clear_namespace` | `fn mod_save_clear_namespace(&self, mod_id: &str) -> bool` |

### `ClientDatabase`

`ClientDatabase`는 광범위한 클라이언트 측 저장/게임 스냅샷입니다. 공개 필드는 다음을 포함합니다:

```rust
pub scene: ClientScene,
pub id: usize,
pub time: NaiveDateTime,
pub teams: HashMap<usize, Team>,
pub knowledge_bases: HashMap<usize, KnowledgeBase>,
pub matches: HashMap<MatchType, MatchInfo>,
pub match_replays: HashMap<usize, MatchReplayData>,
pub leagues: HashMap<usize, League>,
pub tournaments: HashMap<usize, Tournament>,
pub athletes: HashMap<usize, Athlete>,
pub staffs: HashMap<usize, Staff>,
pub league_competitions: HashMap<usize, LeagueCompetition>,
pub tournament_competitions: HashMap<usize, TournamentCompetition>,
pub solo_rank_matches: Vec<SoloRankMatch>,
pub champion_info_sheet: ChampionInfoSheet,
pub game_setting: GameSetting,
pub map_setting: MapSetting,
pub item_setting: ItemSetting,
pub mod_ai_registry: ModAiRegistry,
pub mod_save_data: ModSaveData,
pub mod_events: Vec<ModClientEvent>,
pub pre_patch_data: HashMap<String, GamePatchState>,
pub server_mode: ServerMode,
pub training_plan: TeamTrainingPlan,
pub research_data: TeamResearchData,
pub champion_patch_statistics: HashMap<String, ChampionPatchStatistics>,
pub available_champions: Vec<String>,
pub recruit_done_athletes: Vec<RecruitDoneAthlete>,
pub scout_dispatch: Option<ScoutDispatchInfo>,
pub champion_positions: HashMap<String, Vec<usize>>,
pub head_to_head: HashMap<usize, (usize, usize)>,
```

공통 `ClientDatabase` 도우미 메서드는 `ClientData` 읽기 도우미와 대응되지만, `Ref` 가드 대신 직접 참조를 반환합니다:

- `player_team_id`, `try_player_team`, `team`, `team_ids`
- `athlete`, `athlete_ids`, `athlete_current_region_id`, `athlete_has_visible_solo_rank_in_region`, `visible_solo_rank_athletes`
- `staff`, `staff_ids`
- `knowledge_base`
- `league`, `league_ids`
- `tournament`, `tournament_ids`
- `match_info`, `normal_match`, `practice_match`, `tutorial_match`, `solo_rank_match_info`
- `match_replay`, `match_replay_ids`
- `league_competition`, `league_competition_ids`
- `tournament_competition`, `tournament_competition_ids`
- `solo_rank_match`, `solo_rank_match_ids`
- `champion_info`
- `mod_events`
- `team_display_name`, `multiplayer_chat_sender_display`
- `team_salary_total`, `player_team`, `can_pause_save`, `is_player_vs_player_match`, `match_has_player_team`, `due_player_match`
- `current_intl_break_target_date`, `version_at_date`, `get_historical_sheet`, `get_historical_game_setting`

## 서버 확장 API

### `ModServerExtension`

서버/관리 측에서 실행됩니다.

| 메서드 | 시그니처 | 기본값 |
| --- | --- | --- |
| `on_server_start` | `fn on_server_start(&self, ctx: &mut ServerModContext)` | 동작 없음 |
| `before_management_tick` | `fn before_management_tick(&self, ctx: &mut ServerModContext)` | 동작 없음 |
| `after_management_tick` | `fn after_management_tick(&self, ctx: &mut ServerModContext)` | 동작 없음 |
| `handle_command` | `fn handle_command(&self, ctx: &mut ServerModContext, command: &ModServerCommand) -> ModServerCommandResult` | `통과` |

### `ServerModContext`

필드:

| 필드 | 형식 |
| --- | --- |
| `mod_id` | `&str` |
| `database` | `&mut Database` |
| `server_state` | `&mut ServerState` |

메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `emit_event` | `fn emit_event(&mut self, event: impl Into<String>, payload: impl Into<Vec<u8>>) -> bool` |
| `emit_event_to_player` | `fn emit_event_to_player(&mut self, player_id: PlayerId, event: impl Into<String>, payload: impl Into<Vec<u8>>) -> bool` |
| `emit_event_to_team` | `fn emit_event_to_team(&mut self, team_id: usize, event: impl Into<String>, payload: impl Into<Vec<u8>>) -> bool` |
| `emit_event_to_command_sender` | `fn emit_event_to_command_sender(&mut self, command: &ModServerCommand, event: impl Into<String>, payload: impl Into<Vec<u8>>) -> bool` |
| `player_team_id` | `fn player_team_id(&self, player_id: PlayerId) -> Option<usize>` |
| `team_player_ids` | `fn team_player_ids(&self, team_id: usize) -> Vec<PlayerId>` |

### 클라이언트/서버 메시지 유형

`ModClientEvent` 필드:

| 필드 | 형식 |
| --- | --- |
| `mod_id` | `String` |
| `event` | `String` |
| `payload` | `Vec<u8>` |

`ModClientEvent` 상수 및 메서드:

| 항목 | 시그니처 |
| --- | --- |
| `MAX_MOD_ID_LEN` | `usize = 128` |
| `MAX_NAME_LEN` | `usize = 128` |
| `MAX_PAYLOAD_LEN` | `usize = 1024 * 1024` |
| `new` | `fn new(mod_id: impl Into<String>, event: impl Into<String>, payload: impl Into<Vec<u8>>) -> Option<Self>` |
| `is_valid` | `fn is_valid(&self) -> bool` |

`ModClientEventTarget`:

```rust
pub enum ModClientEventTarget {
    Broadcast,
    Player(PlayerId),
    Team(usize),
}
```

`ModServerCommand` 필드:

| 필드 | 형식 |
| --- | --- |
| `mod_id` | `String` |
| `command` | `String` |
| `payload` | `Vec<u8>` |
| `sender_player_id` | `Option<PlayerId>` |
| `sender_team_id` | `Option<usize>` |

`ModServerCommand` 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `new` | `fn new(mod_id: impl Into<String>, command: impl Into<String>, payload: impl Into<Vec<u8>>, sender_player_id: Option<PlayerId>, sender_team_id: Option<usize>) -> Option<Self>` |
| `is_valid` | `fn is_valid(&self) -> bool` |

`ModServerCommandResult`:

```rust
pub enum ModServerCommandResult {
    Pass,
    Handled,
}
```

## Mod 저장 데이터

`ModSaveData`는 mod id 네임스페이스가 소유한 세이브별 바이트를 저장합니다.

제한:

| 상수 | 값 |
| --- | --- |
| `MAX_ID_LEN` | `128` |
| `MAX_KEY_LEN` | `128` |
| `MAX_VALUE_LEN` | `1024 * 1024` |

`ModSaveData` 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `namespace_count` | `fn namespace_count(&self) -> usize` |
| `namespace_ids` | `fn namespace_ids(&self) -> Vec<String>` |
| `has_namespace` | `fn has_namespace(&self, mod_id: &str) -> bool` |
| `namespace` | `fn namespace(&self, mod_id: &str) -> Option<&ModSaveNamespace>` |
| `save_version` | `fn save_version(&self, mod_id: &str) -> usize` |
| `set_version` | `fn set_version(&mut self, mod_id: &str, version: usize) -> bool` |
| `keys` | `fn keys(&self, mod_id: &str) -> Vec<String>` |
| `contains_key` | `fn contains_key(&self, mod_id: &str, key: &str) -> bool` |
| `get_bytes` | `fn get_bytes(&self, mod_id: &str, key: &str) -> Option<Vec<u8>>` |
| `set_bytes` | `fn set_bytes(&mut self, mod_id: &str, key: &str, value: Vec<u8>) -> bool` |
| `get_string` | `fn get_string(&self, mod_id: &str, key: &str) -> Option<String>` |
| `set_string` | `fn set_string(&mut self, mod_id: &str, key: &str, value: impl Into<String>) -> bool` |
| `remove_key` | `fn remove_key(&mut self, mod_id: &str, key: &str) -> bool` |
| `clear_namespace` | `fn clear_namespace(&mut self, mod_id: &str) -> bool` |

`ModSaveNamespace` 메서드:

| 메서드 | 시그니처 |
| --- | --- |
| `save_version` | `fn save_version(&self) -> usize` |
| `keys` | `fn keys(&self) -> Vec<String>` |

`ClientData`를 통해 호출하면, 쓰기가 허용될 때 이 도우미는 서버용으로 일치하는 세이브 변경 패킷도 대기열에 추가합니다.

## 핵심 값 형식

### 데이터 및 설정 형식

이는 `game-core`에서 직접 다시 내보낸 것입니다:

- `Athlete`
- `Team`
- `Staff`
- `League`
- `Tournament`
- `LeagueCompetition`
- `TournamentCompetition`
- `KnowledgeBase`
- `MatchInfo`
- `MatchType`
- `MatchReplayData`
- `SoloRankMatch`
- `ChampionInfo`
- `ChampionInfoSheet`
- `GameSetting`
- `MapSetting`
- `ItemSetting`
- `TeamTrainingPlan`
- `TeamResearchData`
- `ChampionPatchStatistics`
- `GamePatchState`
- `ServerMode`
- `ScoutDispatchInfo`
- `RecruitDoneAthlete`

### 시뮬레이션 및 콘텐츠 유형

또한 `game-core`에서 다시 내보냄:

- `EntityStat`
- `ChampionCategory`
- `ChampionSubCategory`
- `ChampionTag`
- `AttackType`
- `DamageType`
- `CastingType`
- `CastingTarget`
- `Position`
- `Input`
- `InputTarget`
- `ItemTag`
- `ItemCategory`
- `BuffState`
- `BuffType`
- `CCState`

공통 필드 및 변형 참조:

```rust
pub struct EntityStat {
    pub attack: usize,
    pub magic_power: usize,
    pub hp: usize,
    pub defence: usize,
    pub magic_resistance: usize,
    pub move_speed: usize,
    pub hp_regen: usize,
    pub stack: usize,
    pub crit_chance: usize,
}

pub enum ChampionCategory {
    Melee,
    Range,
    Magician,
    Util,
    Assassin,
}

pub enum ChampionSubCategory {
    Rush,
    Sub,
    CC,
    Tank,
    Single,
    Range,
    Poking,
    Assassin,
}

pub enum ChampionTag {
    AD,
    AP,
    Heal,
    Shield,
    Dot,
    CC,
    Range,
    Melee,
    Tank,
    Magic,
}

pub enum ItemTag {
    AD,
    AP,
    AS,
    Defense,
    MagicResistance,
    HP,
    DefensePenetration, //방어구관통
    Vamp, //흡혈
    HealReduce, //치유감소
    ShieldBreak, //보호막파괴
    MoveSpeed,
    AttackRange,
    Shield,
    HpPercentDamage,
    ASDebuff, //공격속도감소
    ReflectDamage, //피해반사
    Toughness, //강인함
    MRDebuff, //마법저항감소
    RangeDamage,
    MRPenetration,
    CooltimeReduce,
    DotDamage,
    HPRegen,
    MyHpPercentDamage,
    ShareDamage, //피해분담
    Range,
}

pub enum ItemCategory {
    AD,
    AttackSpeed,
    Defense,
    MagicResistance,
    Magic,
    Hp,
}

pub enum DamageType {
    AD,
    AP,
    Fixed,
}

pub enum AttackType {
    BaseAttack,
    Skill,
    Dot,
    DotIgnoreShield,
    Item,
    Well,
}

pub enum CastingType {
    Targeting,
    Position,
    Direction,
    None,
}

pub enum CastingTarget {
    Ally, //아군
    AllyChampion, //아군영웅
    AllyChampionInCC, //군중제어상태의아군영웅
    AllyNotSelf, //자신이아닌아군
    AllyOnlySelf, //자신만
    Enemy, //적군
    EnemyWithoutTower, //포탑제외적군
    EnemyChampion, //적군영웅
    EnemyChampionInCC, //군중제어상태의적군영웅
    EnemyChampionRecentlyAttacked, //최근공격한적군영웅
    Both, //모두
    BothWithoutTower, //포탑제외모두
    BothChampion, //양측영웅
    None, //없음
}

pub enum Position {
    Top,
    Jungle,
    Mid,
    Bottom,
    Support,
}

pub enum Input {
    Move { x: u64, y: u64 },
    Return,
    Attack { target: InputTarget },
    Skill { target: InputTarget },
    Skill2 { target: InputTarget },
    Ult { target: InputTarget },
}

pub enum InputTarget {
    Target { target_id: usize },
    Dir { dir_x: i64, dir_y: i64 },
    Pos { x: u64, y: u64 },
    None,
}

pub enum BuffType {
    Permanent, //영구
    Time { tick: usize },
    WithShield, //방패 장착
}

pub enum CCState {
    Airborne { tick: u64 }, //공중에 뜸
    Stun { tick: u64 }, //기절
    Bind { tick: u64 }, //속박
    BlockAttack { tick: usize }, //공격 차단
    BlockSkill { tick: usize }, //기술 차단
    BlockMoveSkill { tick: usize }, //이동 기술 차단
    ForceMove { tick: u64, dx: i64, dy: i64, speed: u64 }, //강제 이동
    Taunt { tick: u64, target: usize }, //도발
    Fear { tick: u64, dx: i64, dy: i64 }, //공포
    Charm { tick: u64, dx: i64, dy: i64 }, //매혹
    Animation { name: String, tick: u64 }, //동작
}
```

`BuffState` 필드:

```rust
pub struct BuffState {
    pub name: ArrayString<64>,
    pub duration: BuffType,
    pub attack: i32,
    pub attack_mult: i32,
    pub magic_power: i32,
    pub magic_power_mult: i32,
    pub defence: i32,
    pub defence_mult: i32,
    pub hp: i32,
    pub hp_regen: i32,
    pub magic_resistance: i32,
    pub magic_resistance_mult: i32,
    pub vamp: i32,
    pub hp_mult: i32,
    pub move_speed_mult: i32,
    pub attack_speed_mult: i32,
    pub skill_cooldown_mult: i32,
    pub damage_reflect: usize,
    pub damaged_amplify: usize,
    pub defence_penetration: usize,
    pub magic_resistance_penetration: usize,
    pub toughness: usize,
    pub heal_reduce: usize,
    pub range: usize,
    pub base_attack_enemy_max_hp_damage: usize,
    pub self_max_hp_damage: usize,
    pub skill_enemy_max_hp_damage: usize,
    pub damaged_reduce: usize,
    pub dot_amplify: usize,
    pub cc_immune: bool,
    pub ult_cooldown_mult: i32,
    pub radius_mult: i32,
    pub crit_chance: i32,
    pub base_attack_damaged_reduce: usize,
    pub skill_damaged_reduce: usize,
    pub undying: bool,
    pub ignore_wall: bool,
}
```

이러한 기본 형식에 대한 공통 도우미 메서드:

| 형식 | 메서드 |
| --- | --- |
| `EntityStat` | `zero() -> Self`, `add_stat(&mut self, added: EntityStat)` |
| `ChampionCategory` | `to_text_key(&self) -> String` |
| `ChampionSubCategory` | `to_text_key(&self) -> String` |
| `AttackType` | `is_dot(&self) -> bool` |
| `CastingType` | `is_nontarget(&self) -> bool`, `convert(&self, caster: &Entity, entity: &Entity) -> InputTarget` |
| `CastingTarget` | `to_enemy(&self) -> bool`, `can_use_to_jungle(&self) -> bool`, `check(&self, caster: &Entity, target: &Entity) -> bool`, `check_projectile(&self, projectile: &Projectile, target: &Entity) -> bool` |
| `Position` | `as_index(&self) -> usize`, `as_index_with_rule(self, rule: &GameRule) -> usize`, `from_index(index: usize) -> Position`, `from_index_with_rule(index: usize, rule: &GameRule) -> Position`, `to_string(&self) -> String`, `line_priority(&self, line_type: LineType) -> usize` |
| `Input` | `is_act(&self) -> bool` |
| `InputTarget` | `adjust(&self, from_x: u64, from_y: u64, range: u64) -> Self` |
| `BuffType` | `is_end(&self, has_shield: bool) -> bool` |
| `BuffState` | `merge(&mut self, other: &BuffState)`, `apply(&self, stat: EntityStat) -> EntityStat` |
| `CCState` | `block_move(&self) -> bool`, `block_input(&self) -> bool`, `is_cc(&self) -> bool`, `tick(&self) -> u64` |

일부 헬퍼 메서드는 `Entity`, `Projectile`, `GameRule`, `LineType`와 같은 내부 게임 형식을 언급합니다. 이들은 대응하는 SDK에서 내보낸 기본형에 본래 속한 메서드이기 때문에 여기에 나열되어 있지만, 일반적인 네이티브 모드는 보통 다른 내보낸 객체를 통해 이러한 내부 값을 이미 가지고 있는 경우가 아니라면 안전한 `GameCtx`/`EntityRef` 래퍼를 우선 사용하는 것이 좋습니다.

### AI, 메시징 및 저장 형식

이들 또한 `game-core`에서 내보내지며 위에 문서화되어 있습니다:

- `DraftScoreContext`
- `DraftScoreDecision`
- `PlayerAiContext`
- `PlayerAiInitContext`
- `PlayerInputDecision`
- `ModClientEvent`
- `ModClientEventTarget`
- `ModServerCommand`
- `ModServerCommandResult`
- `ModServerExtension`
- `ServerModContext`
- `ModSaveData`
- `ModSaveNamespace`

## 저수준 ABI 형식

이 구조체들은 로더와 SDK에 ABI 경계가 필요하기 때문에 공개되어 있습니다. 일반적인 모드는 `GameCtx`, `EntityRef`, `PlayerRef`, `ProjectileRef`의 안전한 래퍼를 우선 사용하는 것이 좋습니다.

- `SimulationVtable`
- `FrameVtable`
- `ModServiceVtable`

`FrameVtable`에는 저수준 `debug_text` 함수 포인터도 포함되어 있습니다. API 버전 `(0, 7)`에는 안전한 `GameCtx::debug_text` 래퍼가 없습니다.

## 관련 가이드

- [네이티브 Rust 모드](native-rust-mods.md)
- [네이티브 AI 훅](native-ai-hooks.md)
- [모드 저장 데이터](mod-save-data.md)