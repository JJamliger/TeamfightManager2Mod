# 시작하기

이 안내서는 작은 로컬 모드 폴더를 만들고, 게임 내 모드 메뉴에서 사용할 수 있도록 준비합니다.

## 1. 모드 폴더 만들기

게임의 `mods` 디렉터리 아래에 폴더를 만드십시오:

```text
mods/my_mod/
```

개발 체크아웃에서는 저장소의 `mods/` 폴더를 사용하십시오. 패키지된 게임 빌드에서는 게임 실행 파일 옆에 있는 `mods/` 폴더를 사용하십시오. 아직 `mods` 폴더가 없다면 새로 만드십시오.

폴더 이름은 중요합니다. 모드 ID가 되기 때문입니다. `my_mod`, `new_champions`, `balance_pack`처럼 안정적인 소문자 이름을 사용하십시오.

## 2. `mod.mod_info` 추가

다음 파일을 만드십시오:

```text
mods/my_mod/mod.mod_info
```

예시:

```json
{
  "name": "내 모드",
  "author": "작성자 이름",
  "version": "0.1.0",
  "description": "새로운 챔피언을 추가합니다.",
  "last_updated": "2026-05-14",
  "dependencies": [
    {
      "mod_id": "base",
      "version": ">=0.1.0"
    }
  ]
}
```

`0.1.0`, `1.0.0`, `1.2.3` 같은 버전 번호를 사용하십시오.

모드 의존성 버전 요구사항은 시맨틱 버전 관리를 따릅니다. `base` 의존성은 모드가 호환되는 게임 버전을 뜻하며, 기본 게임 에셋 파일에 대한 의존성이 아닙니다. 플레이어는 타이틀 화면 오른쪽 아래에서 현재 게임 버전을 확인할 수 있습니다.

특수한 `base` 의존성에서 각 버전 부분의 의미는 다음과 같습니다:

- `major`: 앞서 해보기 기간에는 `0`, 정식 출시 계열에서는 `1`입니다.
- `minor`: 기능, 게임 콘텐츠 또는 호환성에 영향을 주는 동작이 바뀔 때 증가합니다.
- `patch`: 버그 수정만 있는 업데이트일 때 증가합니다.

예를 들어 `">=0.1.0"`은 이 모드가 Teamfight Manager 2 버전 `0.1.0` 이상을 요구한다는 뜻입니다.

## 3. 콘텐츠 추가

첫 번째 모드로 데이터 전용 챔피언을 추가하십시오:

```text
mods/my_mod/champion/my_champion.data_champion
```

적용 가능한 완전한 예시는 [데이터 전용 챔피언](data-champion.md)을 참조하십시오. 사용 가능한 모든 필드와 효과 유형이 필요하다면 [데이터 챔피언 스키마](data-champion-schema/index.md)를 사용하십시오.

챔피언 이름과 스킬 설명을 위해 다음도 추가하십시오:

```text
mods/my_mod/text/champion.i18n
```

대부분의 챔피언 UI는 기본 챔피언 텍스트 테이블에서 이름과 설명을 읽어 오므로, `mod.override_info`를 사용해 텍스트 파일을 `asset/base/text/champion`에도 병합해야 합니다:

```text
mods/my_mod/mod.override_info
```

```json
{
  "asset/base/text/champion": {
    "path": "asset/my_mod/text/champion",
    "type": "merge"
  }
}
```

이렇게 하면 `description.my_champion.name` 같은 키가 챔피언 목록, 밴/픽 화면, 툴팁, 스킬 설명에서 작동합니다. 전체 텍스트 병합 안내는 [에셋 오버라이드와 i18n](asset-overrides-and-i18n.md)을 참조하십시오.

아이콘이나 스프라이트의 경우, 모드 폴더 아래 아무 위치에나 PNG 또는 Aseprite 파일을 추가하고 `asset/my_mod/...`로 참조하십시오. 애니메이션 스프라이트나 묶음 아이콘 시트를 만드는 경우에는 파일 이름을 정하기 전에 [에셋과 스프라이트 시트](assets-and-sprite-sheets.md)를 읽으십시오.

패키지된 기본 게임 파일을 확인하려면 `TFM2ModUploader.exe`를 실행한 뒤 **Unpack Base Bundle**을 사용하십시오. 업로더 옆에 있는 로컬 `bundle.game_data` 파일을 읽어 `mods/base_unpacked`에 참조용 사본을 작성합니다. 자세한 내용은 [워크숍 업로드](workshop-upload.md#viewing-base-game-data)를 참조하십시오.

## 4. 모드 활성화

게임을 실행하고 타이틀 화면에서 Mods 메뉴를 열어 모드를 활성화한 뒤, 게임을 다시 시작하십시오.

모드 메타데이터, 네이티브 DLL, 그리고 많은 시작 에셋은 게임 시작 시 불러오므로, 변경 사항을 시험할 때는 다시 시작하는 것이 가장 안전합니다.

## 5. 진단 팝업 읽기

모드가 정상적으로 불러와지지 않으면, 게임은 타이틀 화면에 진단 팝업을 표시합니다. 여기에는 보통 잘못된 JSON, 누락된 의존성, 손상된 오버라이드, 네이티브 DLL 문제처럼 확인이 필요한 파일이 표시됩니다.

## 6. 게시하기

모드가 로컬에서 정상 작동하면, 게임 폴더에서 `TFM2ModUploader.exe`를 여십시오.

모드 폴더를 선택하고 `mod.mod_info`의 정보를 확인한 뒤, 공개 범위를 선택하고 변경 사항 메모를 작성하여 Steam Workshop에 게시하십시오. 업로더는 첫 업로드 후 `mod.workshop_id`를 생성하므로, 이후 업로드가 동일한 워크숍 항목을 갱신하도록 그 파일을 보관하십시오.

게임 내 모드 대신 사용자 지정 데이터베이스 파일을 공유하는 경우에는 `mod.mod_info` 대신 `database_pack.info`가 있는 폴더를 만드십시오. 업로더는 이를 **Database Pack**으로 인식하고 첫 업로드 후 `database_pack.workshop_id`를 생성합니다.

전체 게시 절차는 [워크숍 업로드](workshop-upload.md)를 참조하십시오.

## 좋은 첫 모드

간단하게 시작하십시오:

1. [Data-Only Champions](data-champion.md)에서 패키지 배치를 복사합니다.
2. 챔피언 `id`, 능력치, 텍스트 키를 변경합니다.
3. 처음에는 기본 챔피언 스프라이트를 재사용합니다.
4. 게임을 실행하고 챔피언이 나타나는지 확인합니다.
5. 기본 버전이 작동한 뒤 사용자 지정 아이콘과 시각 요소를 추가합니다.
6. 공유할 준비가 되면 `TFM2ModUploader.exe`로 업로드합니다.