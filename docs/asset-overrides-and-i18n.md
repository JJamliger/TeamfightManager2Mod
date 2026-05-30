# 에셋 재정의 및 i18n

모드는 자체 에셋을 가져올 수 있으며, 선택한 기본 게임 에셋도 변경할 수 있습니다.

다음과 같은 경우에 사용합니다:

- 챔피언 이름과 기술 설명 추가,
- UI 문구 추가,
- 스프라이트 교체,
- 전체 파일을 교체하지 않고 몇 개의 JSON 키를 병합.

## 에셋 경로

모드 폴더 아래의 모든 파일은 파일 확장자를 제외한 에셋 경로를 가집니다:

```text
mods/example/text/ui.i18n -> asset/example/text/ui
mods/example/icons/q.png  -> asset/example/icons/q
```

이 경로를 데이터 파일, 네이티브 코드, 재정의 규칙에서 사용하십시오.

## `mod.override_info`

기존 에셋을 병합하거나 교체하려면 모드 루트에 `mod.override_info`를 생성하십시오:

```json
{
  "asset/base/text/ui": {
    "remapping": "asset/example/text/ui",
    "type": "merge"
  },
  "asset/base/aseprite_resources/champions/example_ghoul": {
    "remapping": "asset/example/aseprite_resources/champions/ghoul_king",
    "type": "override"
  }
}
```

객체 키는 변경하려는 기본 에셋입니다. `remapping`은 교체하거나 덧씌울 에셋을 가리킵니다.

유형:

- `merge`: JSON 객체를 병합합니다. i18n과 소규모 JSON 추가에 가장 적합합니다.
- `override`: 대상 에셋 경로 전체를 당신의 에셋으로 교체합니다.

이 파일에 유효하지 않은 JSON이 들어 있으면 게임은 모드를 비활성화하고 진단 팝업에 오류를 표시합니다.

## i18n 파일

i18n 파일은 언어별로 묶인 JSON입니다:

```json
{
  "en": {
    "description": {
      "my_mod_fire_mage": {
        "name": "화염 마법사",
        "skill": "Launches a fire bolt.",
        "skill2": "Focuses flame energy.",
        "ult": "Burns all nearby enemies."
      }
    }
  },
  "ko": {
    "description": {
      "my_mod_fire_mage": {
        "name": "화염 마법사",
        "skill": "Launches a fire bolt.",
        "skill2": "Focuses flame energy.",
        "ult": "Burns all nearby enemies."
      }
    }
  }
}
```

이를 기본 챔피언 텍스트 표에 추가하려면 `asset/base/text/champion`에 병합하십시오:

```json
{
  "asset/base/text/champion": {
    "remapping": "asset/my_mod/text/champion",
    "type": "merge"
  }
}
```

그러면 당신의 챔피언 데이터는 다음을 참조할 수 있습니다:

```text
#asset/base/text/champion?description.my_mod_fire_mage.skill
```

## 텍스트 마크업

설명에는 기본 게임과 동일한 인라인 마크업을 사용할 수 있습니다:

```text
<#ff9028ff>60<> + <i#asset/base/ui/banpick/champion_stat_icon:ad_0><#ff9028ff>80% AD<>의 물리 피해를 입힙니다.
```

일반적인 형식:

- `<#rrggbbaa>text<>`는 글자에 색을 입힙니다.
- `<iasset_path:tag>`는 태그된 스프라이트 시트에서 아이콘을 삽입합니다.

인라인 아이콘의 경우 `asset_path`는 `#sheet` 경로가 아니라 공용 시트 경로입니다. 게임은 `asset_path#data`에서 사각형 정보를 읽고 `asset_path#sheet`에서 그립니다. 시트 형식은 [에셋과 스프라이트 시트](assets-and-sprite-sheets.md)를 참고하십시오.

확실하지 않을 때는 기본 텍스트 파일을 보고 기존 챔피언이나 아이템 설명의 양식을 그대로 따르십시오.

## 호환성 요령

- 텍스트와 JSON 파일에는 `merge`를 우선 사용하십시오.
- 전체 에셋을 정말로 교체하려는 경우에만 `override`를 사용하십시오.
- 작은 변경 때문에 전체 화면 UI 배치를 교체하지 마십시오. 버튼 하나나 팝업 하나를 추가하는 네이티브 확장이 보통 호환성을 유지하기 더 쉽습니다.
- 여러 모드가 같은 키를 수정하면 활성화된 모드 순서가 최종 결과에 영향을 줄 수 있습니다.