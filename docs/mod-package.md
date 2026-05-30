# 모드 패키지 구조

모드는 폴더입니다. 폴더 이름이 모드 ID입니다.

더 큰 예시는 다음과 같습니다:

```text
mods/example/
  mod.mod_info
  mod.override_info
  thumbnail.png
  preview.png
  example.dll
  champion/
    eagle.data_champion
  text/
    champion.i18n
    item.i18n
    ui.i18n
  icons/
    eagle_skill.png
    eagle_skill2.png
    eagle_ult.png
  sprite/
    eagle.png
    eagle.aseprite
    spell_icons#sheet.png
    spell_icons#data.sprite_sheet
  ui/
    layout/
      test_popup.ui
```

Mods 메뉴에 모드가 표시되려면 `mod.mod_info`만 있으면 됩니다. 다른 파일은 모드에 필요한 만큼 추가하십시오.

## 에셋 경로

모드 폴더 안의 파일은 다음과 같이 참조됩니다:

```text
asset/<mod_id>/<확장자를 제외한_상대_경로>
```

예시:

```text
mods/example/text/champion.i18n      -> asset/example/text/champion
mods/example/icons/eagle_skill.png       -> asset/example/icons/eagle_skill
mods/example/ui/layout/test_popup.ui -> asset/example/ui/layout/test_popup
```

Aseprite 파일과 수동 스프라이트 시트는 `#` 접미사를 통해 관련 에셋을 노출할 수 있습니다:

```text
asset/example/aseprite_resources/champions/ghoul_king#sheet
asset/example/aseprite_resources/champions/ghoul_king#anim
asset/example/sprite/spell_icons#sheet
asset/example/sprite/spell_icons#data
```

`#sheet`는 이미지 아틀라스이고, `#anim`은 애니메이션 데이터이며, `#data`는 정지 이미지 태그 데이터입니다. 사용자 지정 애니메이션 스프라이트 또는 아이콘 시트를 만들기 전에 [에셋 및 스프라이트 시트](assets-and-sprite-sheets.md)를 참조하십시오.

## `mod.mod_info`

`mod.mod_info`는 모드를 설명합니다:

```json
{
  "name": "예시 모드",
  "author": "TeamSamoyed",
  "version": "1.1.0",
  "description": "예시 챔피언, 아이템, 텍스트 및 UI를 추가합니다.",
  "last_updated": "2026-05-13",
  "dependencies": [
    {
      "mod_id": "base",
      "version": ">=0.1.0"
    }
  ]
}
```

항목:

- `name`: Mods 메뉴에 표시되는 이름입니다.
- `author`: 제작자 이름입니다.
- `version`: `0.1.0`과 같은 모드 버전입니다.
- `description`: 플레이어에게 표시되는 짧은 설명입니다.
- `last_updated`: 최근 갱신 날짜 또는 짧은 갱신 내용.
- `dependencies`: 이 모드가 불러와지기 위해 충족되어야 하는 다른 모드 또는 게임 버전 요구사항.

## `base` 게임 버전 의존성

`base`는 Teamfight Manager 2 게임 버전을 뜻하는 특별한 의존성 ID입니다. 기본 게임의 asset 폴더를 뜻하지 않으며, 당신의 모드가 `asset/base/...` 파일에 의존한다는 의미도 아닙니다.

플레이어는 타이틀 화면 오른쪽 아래에서 설치된 게임 버전을 확인할 수 있습니다. 당신의 모드가 지원하는 게임 버전을 선언하려면 `base` 의존성을 사용하십시오:

```json
{
  "mod_id": "base",
  "version": ">=0.1.0"
}
```

모드 의존성 버전 요구사항은 시맨틱 버저닝을 따릅니다. 특별한 `base` 의존성의 경우, `major.minor.patch` 각 부분의 의미는 다음과 같습니다:

- `major`: 얼리 액세스 기간에는 `0`, 정식 출시 계열에는 `1`.
- `minor`: 기능, 게임 콘텐츠, 데이터 형식 또는 호환성에 영향을 주는 동작이 변경될 때 증가합니다.
- `patch`: 버그 수정만 있는 갱신 때 증가합니다.

대부분의 모드는 `base` 요구사항을 포함해야 합니다. 그래야 플레이어가 지원되지 않는 게임 버전에서 모드를 불러오려 할 때 명확한 진단 메시지를 받을 수 있습니다.

다른 의존성은 일반적인 모드 의존성입니다. 이것들은 불러오기 순서에도 사용됩니다. 플레이어가 모드를 활성화하면 설치된 의존성은 자동으로 함께 포함되어 먼저 불러와집니다. 필요한 의존성이 없거나 그 버전이 맞지 않으면, 해당 의존 모드는 비활성화되고 게임은 진단 메시지를 표시합니다.

재사용 가능한 네이티브 서비스 모드에도 의존성을 사용하십시오. 예를 들어, 제공자의 런타임 서비스를 호출하는 네이티브 DLL 소비자는 `mod.mod_info`에서 제공자를 선언해야 합니다:

```json
{
  "mod_id": "service_provider",
  "version": ">=1.0.0, <2.0.0"
}
```

## 데이터베이스 팩 창작마당 패키지

데이터베이스 팩 패키지는 일반적인 게임 내 모드가 아니라 창작마당 공유 폴더입니다. `mod.mod_info` 대신 `database_pack.info`를 사용하므로 Mods 메뉴에 자동으로 표시되지 않습니다.

Steam 창작마당을 통해 하나 이상의 데이터베이스 파일을 공유하려면 이 구조를 사용하십시오:

```text
mods/my_database_pack/
  database_pack.info
  league_2026.tfm2db
  fantasy_rosters.tfm2db
  thumbnail.png
  preview.png
```

폴더 이름이 패키지 ID입니다. 데이터베이스 파일은 그 폴더에 직접 넣으십시오. 다운로드 후 당신의 가져오기 도구가 정확히 그 배치를 요구하는 경우가 아니라면, 추가로 필수 내부 폴더를 만들지 마십시오.

`database_pack.info`는 `mod.mod_info`와 동일한 기본 표시 필드를 사용합니다:

```json
{
  "name": "2026 리그 데이터베이스 팩",
  "author": "당신의 이름",
  "version": "1.0.0",
  "description": "2026 리그 설정을 위한 사용자 지정 데이터베이스."
}
```

`TFM2ModUploader.exe`는 `database_pack.info`를 감지하고, 패키지 유형을 **데이터베이스 팩**으로 표시하며, 해당 폴더의 파일을 올리고, 첫 번째 업로드 후 `database_pack.workshop_id`를 기록합니다.

## 선택 파일

- `thumbnail.png`: 게임 내 모드 메뉴에 표시되는 이미지이며, 워크숍 미리보기 대체 이미지로도 사용됩니다.
- `preview.png` 또는 `preview.jpg`: 스팀 워크숍 미리보기로 사용되는 이미지입니다. 이 파일들이 없으면, 업로더는 `thumbnail.png` 또는 `thumbnail.jpg`를 대신 사용할 수 있습니다.
- `mod.override_info`: 일반 모드용 자산 병합 및 덮어쓰기 규칙.
- `<mod_id>.dll`: 코드 기반 모드용 네이티브 Rust DLL.
- `mod.workshop_id`: 첫 번째 일반 모드 업로드 후 워크숍 업로더가 생성합니다.
- `database_pack.workshop_id`: 첫 번째 데이터베이스 팩 업로드 후 워크숍 업로더가 생성합니다.

네이티브 DLL 모드의 경우, DLL에 등록된 모드 ID는 폴더 이름과 일치해야 합니다. 이렇게 하면 잘못된 모드 폴더의 DLL이 실수로 불러와지는 일을 막을 수 있습니다.

Workshop id 파일은 로컬 작업 복사본에 속합니다. 나중에 `TFM2ModUploader.exe`가 동일한 스팀 워크숍 항목을 갱신하도록 하려면 이 파일들을 유지하십시오. 다른 제작자가 복사해서 쓰도록 배포하는 공개 템플릿에는 포함하지 마십시오.

## 업로드 패키징

워크숍에 업로드할 때 `TFM2ModUploader.exe`는 패키지를 임시 업로드 폴더로 복사합니다.

일반 모드의 경우, 다음 항목은 건너뜁니다:

- `src/`
- `mod.workshop_id`
- `database_pack.workshop_id`

데이터베이스 팩의 경우, 다음 항목은 건너뜁니다:

- `mod.workshop_id`
- `database_pack.workshop_id`

선택한 폴더에 남아 있는 컴파일된 DLL, JSON 파일, 이미지, Aseprite 파일, UI 배치, 썸네일, 미리보기 이미지, 데이터베이스 팩 파일은 포함됩니다.

네이티브 Rust 모드의 경우, 업로드 전에 도구가 DLL을 빌드하게 하려면 업로더 옆에 일치하는 Mod SDK를 설치하십시오. SDK가 없어도 업로더는 이미 빌드된 DLL을 올릴 수 있습니다.

## 이름 지정 팁

- ASCII 소문자 폴더 이름을 사용하십시오: `my_mod`, `new_champions`, `balance_pack`, `league_database_pack`.
- 출시 후에도 콘텐츠 ID는 안정적으로 유지하십시오. 저장 데이터와 패치가 해당 ID를 참조할 수 있습니다.
- 다른 모드와 충돌할 가능성이 높은 기본 게임 ID와 이름은 피하십시오.
- 가독성에 도움이 된다면 새 ID 앞에 모드 ID를 접두사로 붙이십시오. 예: `my_mod_fire_mage`.