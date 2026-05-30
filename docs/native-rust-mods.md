# 네이티브 Rust 모드

네이티브 Rust 모드는 맞춤형 코드가 필요한 프로젝트를 위한 것입니다. 데이터 전용 챔피언이나 자산 덮어쓰기로는 부족할 때 사용하십시오.

일반적인 JSON, 이미지, 텍스트 또는 데이터 전용 챔피언 모드에는 이 경로가 필요하지 않습니다. 네이티브 Rust 모드에는 Mod SDK가 필요합니다.

## 시작하기 전에

네이티브 DLL은 그것이 빌드된 게임 버전과 SDK에 종속됩니다.

- 지원하려는 게임 버전에 맞는 Mod SDK로 빌드하십시오.
- 게임 업데이트로 SDK가 변경되면 DLL을 다시 빌드하십시오.
- DLL이 반환하는 모드 ID가 모드 폴더 이름과 일치하는지 확인하십시오.
- DLL이 호환되지 않으면 게임은 타이틀 화면에 진단 메시지를 표시합니다.

## 최소 DLL

```rust
use mod_api::*;

const MOD_ID: &str = "my_mod";

fn init(_ctx: &GameCtx) -> ModRegistration {
    ModRegistration::new(MOD_ID)
}

declare_mod!(init);
```

`declare_mod!` 매크로는 게임이 DLL을 불러오는 데 필요한 진입점을 내보냅니다.

## 콘텐츠 등록

대부분의 네이티브 모드는 `ModRegistration`을 생성하고, 여기에 콘텐츠를 추가한 뒤, 그것을 반환합니다:

```rust
fn init(_ctx: &GameCtx) -> ModRegistration {
    let mut reg = ModRegistration::new("my_mod");
    reg.add_champion(MyChampion);
    reg.add_item(MyItem::default());
    reg.add_draft_score_hook(MyDraftScoreHook);
    reg.add_player_input_ai(MyPlayerInputAi::default());
    reg.set_extension(MyExtension::default());
    reg.set_server_extension(MyServerExtension);
    reg
}
```

다음을 등록할 수 있습니다:

- `ModChampionInfo`: 맞춤형 런타임 로직을 가진 챔피언.
- `ModItemInfo`: 메타데이터와 런타임 콜백을 가진 아이템.
- `ModDraftScoreHook`: 밴/픽 AI 점수 조정 훅.
- `ModPlayerInputAi`: 최종 플레이어 입력 대체 훅.
- `ModExtension`: UI, 장면 및 자산 동작을 위한 생명주기 훅.
- `ModServerExtension`: 서버 측 관리 훅과 클라이언트 명령 처리.

AI 전용 예시는 [Native AI Hooks](native-ai-hooks.md)를 참조하십시오.
내보내진 전체 Rust API 범위는 [Native Mod API Reference](native-mod-api-reference.md)를 참조하십시오.
모드가 소유하는 저장 슬롯 데이터는 [Mod Save Data](mod-save-data.md)를 참조하십시오.

## 현재 저장 데이터 읽기

`Scene::InGame`은 확장 기능에 `ClientData` 핸들을 제공합니다. 이는 `Athlete`, `Team`, `MatchInfo`, `League`, `ChampionInfo`와 같은 게임 내부 자료형을 사용하여 현재 클라이언트 측 게임 데이터를 직접 노출합니다.

빠르게 ID 기반 조회를 하려면 `ClientData`의 도우미 메서드를 호출하십시오:

```rust
impl ModExtension for MyExtension {
    fn post_update(&self, scene: &mut Scene, _ui: &mut GameUI, _assets: &mut Assets, _dt: f32) {
        let Scene::InGame { data } = scene else {
            return;
        };

        let player_team_id = data.player_team_id();

        if let Some(team) = data.team(player_team_id) {
            println!("My team: {}", team.name);
        }

        for athlete_id in data.athlete_ids() {
            if let Some(athlete) = data.athlete(athlete_id) {
                println!("Athlete {}: {}", athlete.id, athlete.name);
            }
        }
    }
}
```

이 도우미들은 내부 데이터를 빌려서 반환합니다. DTO나 복사된 뷰 모델이 아닙니다. 빌림 형식은 클라이언트 내부 `RefCell`의 Rust 일반 `Ref<'_, T>` 가드이므로 `&T`처럼 사용하고, 다음 가변 빌림을 하기 전에 해제되도록 두십시오.

데이터베이스 빌림 하나를 유지한 채 여러 조회를 하고 싶다면 `data.db()`를 사용하십시오:

```rust
let db = data.db();

let team = db.team(team_id);
let athlete = db.athlete(athlete_id);
let match_info = db.normal_match(match_id);
let champion = db.champion_info("fighter");
```

사용 가능한 조회 도우미에는 다음이 포함됩니다:

- `player_team_id`, `player_team`, `team`, `team_ids`
- `athlete`, `athlete_ids`
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

## 챔피언 예시

`ModChampionInfo`는 챔피언의 정체성, 능력치, 행동, 그리고 선택적 패시브를 정의합니다:

```rust
#[derive(Debug)]
struct MyChampion;

impl ModChampionInfo for MyChampion {
    fn id(&self) -> &str { "my_mod_fire_mage" }
    fn name(&self) -> &str { "my_mod_fire_mage" }
    fn category(&self) -> ChampionCategory { ChampionCategory::Magician }
    fn tags(&self) -> Vec<ChampionTag> { vec![ChampionTag::AP, ChampionTag::Range] }

    fn stat(&self) -> EntityStat {
        EntityStat {
            attack: 40,
            magic_power: 65,
            hp: 620,
            defence: 20,
            magic_resistance: 30,
            move_speed: 1050,
            hp_regen: 2,
            stack: 0,
            crit_chance: 0,
        }
    }

    fn growth(&self) -> EntityStat {
        EntityStat {
            attack: 3,
            magic_power: 7,
            hp: 75,
            defence: 3,
            magic_resistance: 3,
            move_speed: 0,
            hp_regen: 1,
            stack: 0,
            crit_chance: 0,
        }
    }

    fn attack(&self) -> Box<dyn ModAction> { Box::new(MyAttack) }
    fn skill(&self) -> Box<dyn ModAction> { Box::new(MySkill) }
    fn skill2(&self) -> Box<dyn ModAction> { Box::new(MySkill2) }
}
```

각 행동은 `ModEffect`를 반환할 수 있습니다. 이 효과는 `GameCtx`를 사용해 게임 상태를 읽고 피해, 치유, 강화 효과, 군중 제어 또는 디버그 그리기를 적용합니다.

## 행동 및 효과 예시

```rust
#[derive(Clone, Debug)]
struct MySkill;

impl ModAction for MySkill {
    fn clone_box(&self) -> Box<dyn ModAction> { Box::new(self.clone()) }
    fn action_name(&self) -> &str { "skill" }
    fn duration(&self) -> usize { 18 }
    fn cooltime(&self, _stat: &EntityStat, _level: usize) -> usize { 240 }
    fn casting_target(&self) -> CastingTarget { CastingTarget::Enemy }

    fn effect(&self) -> Option<ModEffect> {
        Some(ModEffect {
            range: 65000,
            growth_range: 0,
            start_timing: 10,
            casting: CastingType::Targeting,
            target: CastingTarget::Enemy,
            attack_type: AttackType::Skill,
            effect_type: Box::new(MySkillEffect),
        })
    }
}

#[derive(Debug)]
struct MySkillEffect;

impl ModEffectType for MySkillEffect {
    fn apply(&self, ctx: &mut GameCtx, _rng_seed: u64, caster_id: usize, input: InputTarget) {
        let InputTarget::Target { target_id } = input else {
            return;
        };

        let damage = ctx.get_entity(caster_id)
            .map(|caster| 50 + caster.stat().magic_power)
            .unwrap_or(50);

        ctx.deal_damage(caster_id, target_id, 0, damage, AttackType::Skill);
    }

    fn expected_damage(&self, caster_stat: &EntityStat) -> (usize, usize) {
        (0, 50 + caster_stat.magic_power)
    }
}
```

게임에서 복제되는 동작, 패시브, 아이템에는 `clone_box`가 필요합니다. `Clone`을 derive하고 `Box::new(self.clone())`를 반환하는 것만으로도 대체로 충분합니다.

## 아이템 예시

`ModItemInfo`는 새 아이템과 그에 필요한 모든 콜백을 정의합니다:

```rust
#[derive(Clone, Debug, Default)]
struct MyItem {
    hit_count: usize,
}

impl ModItemInfo for MyItem {
    fn clone_box(&self) -> Box<dyn ModItemInfo> { Box::new(self.clone()) }
    fn key(&self) -> &str { "my_mod_claw" }
    fn icon(&self) -> &str { "t3_0" }
    fn price(&self) -> usize { 650 }
    fn tier(&self) -> usize { 2 }

    fn stat(&self) -> BuffState {
        BuffState {
            duration: BuffType::Permanent,
            attack: 40,
            attack_speed_mult: 10,
            ..Default::default()
        }
    }

    fn previous_tier(&self) -> Vec<String> {
        vec!["soldiers_longsword".to_string()]
    }

    fn next_tier(&self) -> Vec<String> {
        vec!["conquerors_greatsword".to_string()]
    }

    fn tags(&self) -> Vec<ItemTag> {
        vec![ItemTag::AD, ItemTag::AS]
    }

    fn category(&self) -> ItemCategory {
        ItemCategory::AD
    }
}
```

전체 아이템 설정 파일을 교체하지 않고도 기존 기본 아이템에서 해당 아이템에 도달할 수 있게 하려면 `previous_tier`를 사용하십시오.

## 네이티브 모드 간 런타임 서비스

네이티브 DLL 모드는 다른 네이티브 DLL 모드를 위해 작은 런타임 서비스를 노출할 수 있습니다. 다른 모드가 의존하는 재사용 가능한 개발자/라이브러리 모드를 배포하려는 경우 이를 사용하십시오.

이것은 직접적인 DLL 연결이 아닙니다. 게임이 서비스 레지스트리를 소유하고, 의존성을 먼저 불러온 뒤, 소비자 모드가 모드 id, 서비스 id, 버전 요구사항으로 제공자 모드를 조회할 수 있게 합니다.

의존성은 여전히 소비자 측 `mod.mod_info`에 포함되어야 합니다:

```json
{
  "dependencies": [
    {
      "mod_id": "base",
      "version": ">=0.1.0"
    },
    {
      "mod_id": "service_provider",
      "version": ">=1.0.0, <2.0.0"
    }
  ]
}
```

소비자가 활성화되면, 설치된 의존성이 자동으로 포함되어 가장 먼저 불러와집니다. 이는 로컬 폴더와 Workshop 설치 폴더 모두에 동일하게 적용되는데, `mod.mod_info`가 게임 측의 단일한 기준 정보이기 때문입니다.

해당 의존성 목록의 `base` 항목은 특별합니다: 기본 자산 패키지가 아니라 Teamfight Manager 2 게임 버전을 뜻합니다. 네이티브 DLL이 어떤 게임 버전 계열을 기준으로 빌드되고 테스트되었는지 선언할 때 사용하십시오.

제공자 예시:

```rust
use std::ffi::c_void;

use mod_api::*;

const MOD_ID: &str = "service_provider";
const SERVICE_ID: &str = "math.v1";

#[repr(C)]
pub struct MathServiceV1 {
    pub bonus: unsafe extern "C" fn(value: u32) -> u32,
}

unsafe extern "C" fn bonus(value: u32) -> u32 {
    value + 77
}

static MATH_SERVICE: MathServiceV1 = MathServiceV1 {
    bonus,
};

fn init(ctx: &GameCtx) -> ModRegistration {
    ctx.register_service(
        SERVICE_ID,
        ModServiceVersion::new(1, 0, 0),
        ModService::from_raw(
            std::ptr::null_mut(),
            &MATH_SERVICE as *const MathServiceV1 as *const c_void,
        ),
    );

    ModRegistration::new(MOD_ID)
}

declare_mod!(init);
```

소비자 예시:

```rust
use mod_api::*;

const MOD_ID: &str = "service_consumer";
const PROVIDER_MOD_ID: &str = "service_provider";
const SERVICE_ID: &str = "math.v1";

#[repr(C)]
struct MathServiceV1 {
    bonus: unsafe extern "C" fn(value: u32) -> u32,
}

fn init(ctx: &GameCtx) -> ModRegistration {
    let provider_bonus = ctx
        .query_service(PROVIDER_MOD_ID, SERVICE_ID, ">=1.0.0, <2.0.0")
        .and_then(|service| unsafe {
            service
                .vtable::<MathServiceV1>()
                .map(|vtable| (vtable.bonus)(23) as usize)
        })
        .unwrap_or(0);

    let mut reg = ModRegistration::new(MOD_ID);
    reg.add_item(MyItem::with_bonus(provider_bonus));
    reg
}

declare_mod!(init);
```

서비스 경계는 단순하게 유지하십시오:

- `#[repr(C)]` 서비스 구조체를 사용하십시오.
- 제공자와 소비자의 레이아웃을 정확히 일치시키십시오.
- 기본형 값, 불투명 핸들, 명시적인 함수 포인터를 우선 사용하십시오.
- 명확한 소유권 규칙도 함께 정의하지 않는 한 Rust 트레이트 객체, `String`, `Vec` 또는 제공자 소유 할당을 전달하지 마십시오.
- 제공자 DLL이 로드되어 있는 동안에는 제공자 서비스 데이터와 vtable이 유효하도록 유지하십시오.
- 서비스 레이아웃을 깨뜨리는 경우 서비스 id 또는 주 버전을 변경하십시오.

## 확장 훅

`ModExtension`은 네이티브 모드가 장면, UI 및 에셋 생명주기에 반응할 수 있게 합니다:

```rust
#[derive(Default)]
struct MyExtension;

impl ModExtension for MyExtension {
    fn on_init(&self, _scene: &mut Scene, _ui: &mut GameUI, _assets: &mut Assets) {}

    fn post_update(&self, _scene: &mut Scene, _ui: &mut GameUI, _assets: &mut Assets, _dt: f32) {
        // UI or scene logic here.
    }
}
```

이러한 훅은 강력하므로, 용도를 집중시키십시오:

- 변경하기 전에 UI 노드가 존재하는지 확인하십시오.
- 전체 기본 레이아웃을 교체하는 대신 작은 UI 요소를 추가하십시오.
- 이벤트 처리기는 한 번만 등록하거나, 설정이 반복 실행되지 않도록 보호 장치를 두십시오.

## 서버 확장 훅

`ModServerExtension`은 게임 서버 측에서 실행됩니다. 직접적인 UI 작업이 아니라, 권한 있는 처리가 필요한 관리/저장 로직에 사용하십시오.

```rust
use mod_api::*;

const MOD_ID: &str = "my_mod";

struct MyServerExtension;

impl ModServerExtension for MyServerExtension {
    fn on_server_start(&self, ctx: &mut ServerModContext) {
        // Runs once after the server initializes the save for this session.
        ctx.database.mod_save_data.set_string(MOD_ID, "server_started_at", ctx.database.time.to_string());
    }

    fn before_management_tick(&self, ctx: &mut ServerModContext) {
        // Runs before a server-side management time step.
        let current_time = ctx.database.time;
        ctx.database.mod_save_data.set_string(MOD_ID, "last_seen_before_tick", current_time.to_string());
    }

    fn after_management_tick(&self, ctx: &mut ServerModContext) {
        // Runs after the server-side management systems for a time step.
        ctx.emit_event("tick_finished", ctx.database.time.to_string().into_bytes());
    }
}
```

클라이언트/UI 확장과는 별도로 등록하십시오:

```rust
fn init(_ctx: &GameCtx) -> ModRegistration {
    let mut reg = ModRegistration::new(MOD_ID);
    reg.set_extension(MyClientExtension::default());
    reg.set_server_extension(MyServerExtension);
    reg
}
```

서버 컨텍스트는 서버가 사용하는 실제 `Database`와 `ServerState`를 드러냅니다. 여기에서 이루어진 변경은 권한 있는 변경입니다. 편집은 최소한으로 유지하고, 기존 게임 메서드가 있다면 이를 우선 사용하십시오. 서버 규칙을 우회하면 멀티플레이어 저장 데이터의 동기화가 어긋나거나 필요한 부수 효과가 누락될 수 있습니다.

## 모드 명령과 이벤트

클라이언트/UI 코드는 작은 명령 패킷을 서버로 보낼 수 있습니다:

```rust
impl ModExtension for MyClientExtension {
    fn post_update(&self, scene: &mut Scene, _ui: &mut GameUI, _assets: &mut Assets, _dt: f32) {
        let Scene::InGame { data } = scene else {
            return;
        };

        data.send_mod_command(MOD_ID, "mark_seen", b"hello".to_vec());

        for event in data.take_mod_events(MOD_ID) {
            if event.event == "mark_seen_done" {
                println!("server replied: {:?}", event.payload);
            }
        }
    }
}
```

대응하는 서버 확장은 그 명령을 처리하고 클라이언트에 이벤트를 다시 보낼 수 있습니다:

```rust
impl ModServerExtension for MyServerExtension {
    fn handle_command(&self, ctx: &mut ServerModContext, command: &ModServerCommand) -> ModServerCommandResult {
        if command.command != "mark_seen" {
            return ModServerCommandResult::Pass;
        }

        if let Some(team_id) = command.sender_team_id {
            ctx.database.mod_save_data.set_string(MOD_ID, "last_command_team", team_id.to_string());
        }
        ctx.emit_event_to_command_sender(command, "mark_seen_done", command.payload.clone());
        ModServerCommandResult::Handled
    }
}
```

명령과 이벤트는 모드 ID를 기준으로 구분됩니다. 이름은 128바이트로 제한되며, 페이로드는 1 MiB로 제한되고, 잘못된 패킷은 거부됩니다.

서버 이벤트는 서로 다른 대상으로 보낼 수 있습니다:

```rust
ctx.emit_event("broadcast_event", vec![]);
ctx.emit_event_to_player(PlayerId(0), "player_event", vec![]);
ctx.emit_event_to_team(team_id, "team_event", vec![]);
ctx.emit_event_to_command_sender(command, "reply_event", vec![]);
```

멀티플레이어에서 UI 응답이나 팀 전용 정보에는 대상 지정 이벤트를 사용하십시오. 연결된 모든 클라이언트가 받아도 되는 정보에만 전체 방송 이벤트를 사용하십시오.

## 저장별 모드 데이터

네이티브 확장은 장면이 `Scene::InGame`일 때 `ClientData::mod_save_*` 도우미를 통해 현재 저장 데이터에 사용자 지정 데이터를 저장할 수 있습니다.

```rust
impl ModExtension for MyExtension {
    fn post_update(&self, scene: &mut Scene, _ui: &mut GameUI, _assets: &mut Assets, _dt: f32) {
        let Scene::InGame { data } = scene else {
            return;
        };

        if data.mod_save_get_string("my_mod", "initialized").is_none() {
            data.mod_save_set_version("my_mod", 1);
            data.mod_save_set_string("my_mod", "initialized", "true");
        }
    }
}
```

데이터는 모드 아이디별로 구분되어 게임 데이터베이스와 함께 저장됩니다. 멀티플레이어 리그 쓰기는 호스트만 가능하며, 쓰기 도우미는 요청을 대기열에 넣을 수 없으면 `false`를 반환합니다. 제한 사항, 마이그레이션 방식, 전체 도우미 목록은 [모드 저장 데이터](mod-save-data.md)를 참조하십시오.

## 모드 SDK로 빌드하기

SDK에는 미리 빌드된 `mod-api` 파일과 빌드 도우미가 포함되어 있습니다:

```text
mod-sdk/
  deps/
  native/
  build_mod.bat
  build_mod_cargo.ps1
  rust-toolchain.toml
  toolchain_version.txt
  template/
    Cargo.toml
    src/lib.rs
```

간단한 단일 파일 모드의 경우 `src/lib.rs`만 있는 폴더도 여전히 사용할 수 있습니다. SDK는 `rustc`를 직접 호출하고 일치하는 `mod_api` 크레이트를 주입합니다:

```bat
cd mod-sdk
build_mod.bat path\to\your_mod\src\lib.rs
```

외부 Rust 크레이트가 필요한 모드의 경우, 모드 폴더 안에서 일반 Cargo 프로젝트를 사용하십시오:

```text
my_mod/
  mod.mod_info
  Cargo.toml
  src/
    lib.rs
  preview.png
```

`Cargo.toml` 예시:

```toml
[package]
name = "my_mod"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
rand = "0.8"
serde_json = "1.0"
```

공개 SDK 작업 흐름에서는 `[dependencies]`에 `mod-api`를 추가하지 마십시오. 업로더와 SDK 빌드 스크립트가 일치하는 미리 빌드된 `mod_api` 크레이트를 자동으로 주입하므로 DLL이 게임 SDK와 일치하게 됩니다.

SDK를 통한 수동 Cargo 빌드:

```bat
cd mod-sdk
build_mod.bat path\to\your_mod
```

빌드는 생성된 DLL을 모드 폴더 이름을 사용해 `my_mod.dll`로 모드 폴더에 다시 복사합니다. 폴더 이름과 `ModRegistration::new("my_mod")` 아이디를 일치시키십시오.

다른 Cargo 워크스페이스 안에서 개발 중인데 Cargo가 패키지가 예기치 않게 워크스페이스 안에 있다고 말하면, 모드의 `Cargo.toml`에 빈 `[workspace]` 테이블을 추가하거나 모드 폴더를 해당 워크스페이스 밖으로 옮기십시오.

## 네이티브 Rust 모드 업로드하기

SDK가 바로 옆에 설치되어 있으면 `TFM2ModUploader.exe`는 네이티브 Rust 모드를 빌드하고 업로드할 수 있습니다.

권장 배치:

```text
TeamfightManager2.exe
TFM2ModUploader.exe
steam_api64.dll
bundle.game_data
mod-sdk/
  deps/
  native/
  build_mod.bat
  build_mod_cargo.ps1
  rust-toolchain.toml
```

업로더는 선택한 모드 폴더를 다음 순서로 확인합니다:

1. `Cargo.toml`이 있으면 Cargo로 빌드합니다. 이는 crates.io 또는 그 밖의 일반적인 Cargo 의존성 원본의 외부 크레이트를 지원합니다.
2. `Cargo.toml`은 없지만 `src/lib.rs`가 있으면, 이전의 직접 `rustc` 빌드를 사용합니다.
3. 둘 다 없으면, 네이티브 빌드 단계는 표시되지 않습니다.

**업로드 전에 네이티브 Rust 코드 빌드**를 선택하면, 업로더는 SDK의 `mod-api` 파일을 사용해 먼저 DLL을 빌드합니다. 그다음 컴파일된 DLL과 나머지 실행용 모드 자산을 업로드합니다. `src/`, `target/`, `Cargo.toml`, `Cargo.lock` 같은 빌드 전용 파일과 소스 폴더는 업로드 중 건너뛰므로, Rust 소스 코드는 Workshop으로 전송되지 않습니다.

이미 직접 DLL을 빌드했다면, DLL을 모드 폴더에 그대로 두고 업로드 전에 빌드 옵션 선택을 해제하십시오.

SDK와 게임 버전은 일치해야 합니다. SDK가 변경되는 게임 갱신 후에는 DLL을 다시 빌드하고 새로운 Workshop 갱신본을 업로드하십시오.
## 일반적인 네이티브 불러오기 문제

- DLL이 모드 폴더에 없습니다.
- DLL이 다른 SDK로 빌드되었습니다.
- DLL이 폴더 이름과 일치하지 않는 모드 ID를 등록합니다.
- `declare_mod!`를 사용하지 않아 DLL이 예상된 진입점을 내보내지 않습니다.
- 오래된 SDK로 빌드되어 DLL이 예상된 API 버전 심볼을 내보내지 않습니다.
- Rust 도구 모음 또는 SDK 산출물이 대상 게임 버전과 일치하지 않습니다.
- 진입 함수에서 패닉이 발생했거나 잘못된 등록값을 반환했습니다.