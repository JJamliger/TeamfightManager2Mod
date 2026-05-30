# 모드 저장 데이터

기본 Rust 모드는 게임의 저장 파일 안에 소규모 사용자 지정 데이터를 저장할 수 있습니다.

모드 튜토리얼 표시, 사용자 지정 진행도, 생성된 상태, 또는 저장된 리그와 함께 이동해야 하는 설정처럼 하나의 저장 슬롯에 속하는 데이터에 이것을 사용하십시오. 대용량 자산, 기록, 또는 모든 저장에 적용되어야 하는 전역 환경설정에는 사용하지 마십시오.

모드 저장 데이터는 `ClientData::mod_save_*` 도우미를 통해 사용할 수 있습니다. 가장 일반적으로 접근하는 위치는 장면이 `Scene::InGame`일 때의 `ModExtension`입니다.

서버 확장도 `ModServerExtension` 훅 또는 명령 처리기에서 `ctx.database.mod_save_data`를 통해 권한 있는 저장 데이터를 기록할 수 있습니다.

## 기본 예시

```rust
use mod_api::*;

const MOD_ID: &str = "my_mod";

#[derive(Default)]
struct MyExtension;

impl ModExtension for MyExtension {
    fn post_update(&self, scene: &mut Scene, _ui: &mut GameUI, _assets: &mut Assets, _dt: f32) {
        let Scene::InGame { data } = scene else {
            return;
        };

        if data.mod_save_get_string(MOD_ID, "initialized").is_none() {
            data.mod_save_set_version(MOD_ID, 1);
            data.mod_save_set_string(MOD_ID, "initialized", "true");
        }
    }
}

fn init(_ctx: &GameCtx) -> ModRegistration {
    let mut reg = ModRegistration::new(MOD_ID);
    reg.set_extension(MyExtension::default());
    reg
}

declare_mod!(init);
```

이것은 `MOD_ID`로 이름 붙은 네임스페이스에 기록합니다. 직접 만든 모드 id를 사용하고, 그것을 모드 폴더 이름 및 `ModRegistration::new(...)` id와 같게 유지하십시오.

## 사용 가능한 도우미

```rust
data.can_write_mod_save();

data.mod_save_version(MOD_ID);
data.mod_save_set_version(MOD_ID, 1);

data.mod_save_keys(MOD_ID);
data.mod_save_contains_key(MOD_ID, "key");

data.mod_save_get_bytes(MOD_ID, "key");
data.mod_save_set_bytes(MOD_ID, "key", vec![1, 2, 3]);

data.mod_save_get_string(MOD_ID, "key");
data.mod_save_set_string(MOD_ID, "key", "value");

data.mod_save_remove_key(MOD_ID, "key");
data.mod_save_clear_namespace(MOD_ID);
```

기록 도우미는 로컬 요청이 수락되어 대기열에 들어가면 `true`를 반환합니다. 모드 id, 키, 값 또는 멀티플레이어 기록 권한이 유효하지 않으면 `false`를 반환합니다.

## 버전 및 마이그레이션

각 모드 네임스페이스에는 버전 번호가 있습니다. 이를 사용해 자신의 저장 데이터를 마이그레이션하십시오:

```rust
let version = data.mod_save_version(MOD_ID);

if version < 1 {
    data.mod_save_set_string(MOD_ID, "initialized", "true");
    data.mod_save_set_version(MOD_ID, 1);
}
```

네임스페이스 버전은 `mod.mod_info`의 패키지 버전과는 별개입니다. 이는 오직 자신의 저장 데이터 형식을 위한 것입니다.

## 제한

현재 제한:

- 모드 ID: 비어 있지 않아야 하며, 최대 128바이트, NUL 바이트는 허용되지 않습니다.
- 키: 비어 있지 않아야 하며, 최대 128바이트, NUL 바이트는 허용되지 않습니다.
- 값: 키당 최대 1 MiB.
- 문자열 도우미는 UTF-8 바이트를 저장합니다.

값을 간결하게 사용하십시오. 구조화된 데이터가 필요하다면, 자신의 JSON 또는 바이너리 블롭을 바이트나 문자열로 직렬화하십시오.

## 멀티플레이어 동작

싱글플레이어 저장에서는 모드 저장 데이터를 정상적으로 기록할 수 있습니다.

멀티플레이어 리그 저장에서는 호스트만 모드 저장 데이터를 기록할 수 있습니다. 비호스트 클라이언트는 `can_write_mod_save()`를 확인하거나, 기록 도우미에서 `false` 반환값을 처리해야 합니다.

서버가 최종 저장 데이터를 소유합니다. 서버는 기록을 검증하고, 저장 데이터베이스를 갱신하며, 변경 사항을 다른 클라이언트에 전파합니다.

`ModServerExtension`이 `ctx.database.mod_save_data`에 직접 기록할 때, 그 변경은 이미 권한 있는 서버 데이터베이스에 반영된 상태입니다. 클라이언트 UI가 즉시 반응해야 한다면 `ctx.emit_event(...)`를 사용하고, 즉각적인 UI 반응이 필요 없다면 다음 일반 데이터 동기화에 맡기십시오.

## 실용적인 조언

- 매 프레임마다 같은 값을 기록하지 마십시오. 먼저 값이 없거나 변경되었는지 확인하십시오.
- 하나의 네임스페이스를 사용하십시오: 자신의 `MOD_ID`.
- 플래그와 카운터에는 작은 문자열 값을 우선 사용하십시오.
- 오래된 키의 이름을 즉시 바꾸기보다 마이그레이션에는 네임스페이스 버전을 사용하십시오.
- 모든 저장에 적용되어야 하는 전역 모드 설정은 저장 데이터 밖에 유지하십시오.