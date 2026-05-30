# Steam 창작마당 업로드

Teamfight Manager 2에는 모드와 데이터베이스 팩을 Steam 창작마당에 게시하기 위한 `TFM2ModUploader.exe`가 포함되어 있습니다.

패키지 폴더를 준비한 뒤 업로더를 열고, 폴더를 선택하고, 세부 정보를 확인한 다음 게시하십시오.

## 업로드하기 전에

Steam이 실행 중이며 창작마당 항목을 소유할 계정으로 로그인되어 있는지 확인하십시오.

일반 게임 모드 폴더는 `mod.mod_info`를 사용합니다:

```text
my_mod/
  mod.mod_info
  thumbnail.png
```

데이터베이스 팩 폴더는 패키지 폴더 안에 바로 `database_pack.info`를 사용합니다:

```text
my_database_pack/
  database_pack.info
  league_2026.tfm2db
  fantasy_rosters.tfm2db
  thumbnail.png
```

데이터베이스 팩 폴더 자체가 업로드 패키지입니다. 다운로드 후 자체 가져오기 기능이 해당 구조를 요구하지 않는 한, 필요한 내부 `packs/` 폴더를 추가로 넣지 마십시오.

`mod.mod_info`와 `database_pack.info`는 업로더에 표시되는 제목, 작성자, 버전, 설명을 제공합니다. 별도의 `preview.png`를 제공하지 않으면 `thumbnail.png`를 창작마당 미리보기 이미지로 사용할 수도 있습니다.

더 나은 창작마당 페이지를 위해 다음을 추가하십시오:

```text
my_mod/
  preview.png
```

`preview.jpg`와 `thumbnail.jpg`도 사용할 수 있습니다.

## 업로더 열기

게임 폴더에서 다음을 실행하십시오:

```text
TFM2ModUploader.exe
```

업로더는 같은 게임 앱을 통해 Steam을 사용하므로, 게임 실행 파일과 게임 패키지의 Steam DLL 옆에 두십시오.

## 기본 게임 데이터 보기

패키지된 게임 빌드는 게임 실행 파일과 업로더 옆의 `bundle.game_data`에 기본 게임 자산을 보관합니다.

모드를 제작하면서 해당 파일들을 살펴보려면 업로더의 **Base Bundle** 구역을 사용하십시오:

1. 게임 폴더에서 `TFM2ModUploader.exe`를 실행하십시오.
2. `bundle.game_data`가 업로더와 같은 폴더에 있는지 확인하십시오.
3. **Unpack Base Bundle**을 클릭하십시오.

이 도구는 다음을 생성합니다:

```text
mods/base_unpacked/
```

압축 해제된 폴더는 게임의 기본 자산 파일과 같은 구조를 사용합니다. 이는 `mod.mod_info`의 `base` 버전 의존성과는 별개입니다. 예를 들어, 묶음 자산 `asset/base/text/ui`는 다음과 같이 작성됩니다:

```text
mods/base_unpacked/text/ui.i18n
```

- UI 배치 파일

- text/i18n 파일

- 썸네일 및 미리보기 이미지
데이터베이스 팩의 경우 `database_pack.info`와 해당 폴더 안의 일반 파일(하나 이상의 사용자 지정 데이터베이스 팩 파일과 미리보기 이미지 등)이 포함됩니다.
워크숍 다운로드에 포함되어서는 안 되는 파일은 제외됩니다:
   - `mod.workshop_id`
   - **친구만**: Steam 친구에게만 표시됩니다.
   - **비공개**: 오직 자신에게만 표시됩니다.
   - **목록 비공개**: 링크로는 접근할 수 있지만 공개 목록에는 표시되지 않습니다.
4. `최초 업로드`와 같은 짧은 변경 사항 메모를 작성합니다.
5. **Steam Workshop에 게시**를 클릭합니다.

처음 업로드할 때 Steam은 새 Workshop 항목을 생성합니다. 그런 다음 업로더는 패키지 폴더 안에 id 파일을 저장합니다:

- 모드는 `mod.workshop_id`를 사용합니다.
- 데이터베이스 팩은 `database_pack.workshop_id`를 사용합니다.

그 파일은 보관하십시오. 업로더가 다음에 어떤 Workshop 항목을 업데이트해야 하는지 알 수 있게 해 주는 파일입니다.

## 기존 패키지 업데이트

처음 업로드한 뒤에는 같은 패키지 폴더에 해당 Workshop id 파일이 들어 있어야 합니다:

```text
my_mod/
  mod.workshop_id
```

```text
my_database_pack/
database_pack.workshop_id
```

`TFM2ModUploader.exe`를 열고 같은 폴더를 선택한 뒤, 변경 사항 메모를 작성하고 **Workshop 항목 업데이트**를 클릭합니다.

패키지를 새 Workshop 항목으로 게시하려는 의도가 아니라면 Workshop id 파일을 삭제하지 마십시오. 이 파일을 잃어버리면 Workshop 페이지 URL에서 항목 id를 찾아 파일을 다시 만들어야 할 수 있습니다.

예시:

```json
{
"published_file_id": 3725617184
}
```

## 업로드되는 항목

업로더는 패키지의 임시 복사본을 준비한 다음 그 복사본을 Steam에 업로드합니다.

모드의 경우 일반적인 모드 파일이 포함됩니다:

- `mod.mod_info`
- `mod.override_info`
- 컴파일된 DLL
- JSON 및 데이터 파일
- PNG/JPG 이미지
- Aseprite 파일
- 수동 스프라이트 시트 파일
- UI 레이아웃
- text/i18n 파일
- 썸네일 및 미리보기 이미지

데이터베이스 팩의 경우 `database_pack.info`와 해당 폴더의 일반 파일(예: 하나 이상의 사용자 지정 데이터베이스 팩 파일과 미리보기 이미지)이 포함됩니다.

Workshop 다운로드에 포함되면 안 되는 파일은 제외됩니다:

- `mod.workshop_id`
- `database_pack.workshop_id`
- 일반 모드 패키지의 `src/`

이는 네이티브 Rust 소스 코드는 업로드되지 않는다는 뜻입니다. 컴파일된 DLL만 업로드됩니다.

## 데이터베이스 팩

데이터베이스 팩은 Workshop에서 다운로드하고 공유하기 위한 것입니다. 다운로드되었다고 해서 게임 내 모드 메뉴를 통해 자동으로 활성화되지는 않습니다.

작업장 항목 하나에 하나 이상의 데이터베이스 파일을 담으려면 이 구조를 사용하세요:

```text
my_database_pack/
database_pack.info
league_2026.tfm2db
fantasy_rosters.tfm2db
preview.png
```

`database_pack.info`는 `mod.mod_info`와 동일한 기본 필드를 사용합니다:

```json
{
"name": "2026 리그 데이터베이스 팩",
"author": "당신의 이름",
"version": "1.0.0",
"description": "2026 리그 구성을 위한 사용자 지정 데이터베이스."
}
```

업로더는 `database_pack.info`를 감지하고, 패키지 유형을 **데이터베이스 팩**으로 표시하며, 해당 폴더에 대한 작업장 항목을 생성하거나 갱신합니다. 첫 업로드 후에는 이후 업로드가 동일한 항목을 갱신하도록 `database_pack.workshop_id`를 유지하세요.

## 네이티브 Rust 모드와 SDK

대부분의 모드는 Mod SDK가 필요하지 않습니다. JSON, 이미지, Aseprite, 텍스트, UI, 그리고 데이터 전용 챔피언 모드는 `TFM2ModUploader.exe`만으로 업로드할 수 있습니다.

네이티브 Rust 모드는 다릅니다. 업로더는 선택한 폴더에 다음 중 하나가 있으면 해당 모드를 코드 모드로 처리합니다:

```text
Cargo.toml
```

또는:

```text
src/lib.rs
```

**업로드 전에 네이티브 Rust 코드 빌드**가 체크되어 있으면, 업로드 전에 DLL을 빌드하려고 시도합니다.

그 빌드 단계에는 다음이 필요합니다:

- 일치하는 Teamfight Manager 2 Mod SDK
- `cargo`와 `rustc`가 작동하는 Rust 도구 모음
- `TFM2ModUploader.exe` 옆에 배치된 SDK 폴더
- Cargo 모드가 crates.io 패키지에 의존하는 경우 인터넷 접속 또는 채워진 Cargo 캐시

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

빌드 동작:

- `Cargo.toml`이 있는 경우: 업로더는 Cargo 릴리스 빌드를 실행하고, SDK의 사전 빌드된 `mod_api` 크레이트를 주입하며, 일반 Cargo 의존성 소스의 외부 크레이트를 지원합니다.
- `src/lib.rs`만 있는 경우: 업로더는 `mod_api`와 `std`만 사용하는 구식 직접 `rustc` 빌드를 사용합니다.

Cargo 모드의 경우 일반 라이브러리 크레이트를 사용하세요:

```toml
[package]
name = "my_mod"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
rand = "0.8"
```

공개 SDK 작업 흐름에서는 `[dependencies]`에 `mod-api`를 추가하지 마세요. 업로더가 일치하는 SDK 빌드를 자동으로 주입합니다.

SDK가 있으면 업로더는 당신의 모드 DLL을 선택한 모드 폴더에 빌드한 뒤, 완성된 런타임 파일을 업로드합니다. `src/`, `target/`, `Cargo.toml`, `Cargo.lock` 같은 빌드 전용/소스 경로는 건너뜁니다. 이미 DLL을 직접 빌드했다면 빌드 옵션의 선택을 해제하고 기존 파일을 업로드할 수 있습니다.

SDK는 대상으로 삼는 게임 버전과 일치해야 합니다. 새 SDK가 배포되어 게임이 갱신되면 네이티브 모드를 다시 빌드하십시오.
## 워크숍 항목 관리

각 워크숍 항목마다 로컬 패키지 폴더 하나를 기준 원본으로 사용하십시오.

좋은 습관:

- 첫 업로드 후에는 작업 사본에 `mod.workshop_id` 또는 `database_pack.workshop_id`를 유지하십시오.
- 출시할 때는 `mod.mod_info` 또는 `database_pack.info`의 `version` 필드를 갱신하십시오.
- 구독자가 무엇이 바뀌었는지 알 수 있도록 변경 사항 메모를 명확히 작성하십시오.
- 갱신본을 게시하기 전에 모드 또는 데이터베이스 팩을 로컬에서 시험하십시오.
- 패키지 폴더의 백업 또는 소스 저장소를 유지하십시오.

다른 사람이 복사할 수 있도록 서식을 게시하는 경우에는 당신의 워크숍 id 파일을 포함하지 마십시오. 그렇지 않으면 그들의 업로드가 당신의 워크숍 항목을 갱신하려 할 수 있습니다.

## Steam 법적 동의

Steam은 첫 업로드를 완료하기 전에 워크숍 법적 동의에 대한 수락을 요구할 수 있습니다.

열기:

```text
https://steamcommunity.com/sharedfiles/workshoplegalagreement
```

수락한 뒤 다시 업로드를 실행하십시오.