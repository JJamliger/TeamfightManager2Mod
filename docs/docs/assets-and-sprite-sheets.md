# 에셋과 스프라이트 시트

모드에서 발생하는 대부분의 이미지 문제는 작은 한 가지 세부 사항에서 비롯됩니다: 게임이 항상 이미지 파일을 직접 사용하는 것은 아닙니다. 스프라이트, 아이콘 세트, 애니메이션은 같은 기본 경로를 공유하는 서로 관련된 에셋 묶음인 경우가 많습니다.

예를 들어, 이 파일은:

```text
mods/example/champions/fire_mage.aseprite
```

다음과 같이 불러와집니다:

```text
asset/example/champions/fire_mage
```

게임이 해당 Aseprite 파일을 렌더링용으로 준비할 때, 그 옆에 추가 에셋이 생성될 수도 있습니다:

```text
asset/example/champions/fire_mage#sheet
asset/example/champions/fire_mage#anim
asset/example/champions/fire_mage#data
```

`#` 뒤의 부분은 파일 확장자가 아닙니다. 같은 에셋의 이름 붙은 구성 요소입니다.

## 일반적인 에셋 구성 요소

`#sheet`는 화면에 그려지는 이미지입니다. 보통 PNG 아틀라스입니다.

`#anim`은 이름이 붙은 애니메이션 목록입니다. 각 애니메이션 태그는 `#sheet` 안의 하나 이상의 사각형 영역을 가리키며, 각 프레임의 재생 시간도 함께 가집니다. 보통 챔피언 스프라이트와 스킬 효과가 이를 사용합니다.

`#data`는 `#sheet` 안에 있는 이름 붙은 정지 이미지 사각형 영역 목록입니다. UI 아이콘, 아이템 아이콘, 인라인 텍스트 아이콘, 단순 이미지 아틀라스가 보통 이를 사용합니다.

`#lanim`은 레이어형 애니메이션 데이터입니다. 더 복잡한 레이어형 애니메이션 렌더링에 사용되며, 보통 기본 게임의 레이어형 Aseprite 구성을 맞출 때만 필요합니다.

대부분의 모드 파일에는 `#sheet`, `#anim`, `#data`만 있으면 됩니다.

## 스프라이트 시트 태그의 의미

스프라이트 시트는 여러 개의 작은 이미지가 들어 있는 하나의 이미지입니다. 태그는 그 작은 이미지나 애니메이션 중 하나를 선택할 때 쓰는 이름입니다.

스킬 아이콘 시트의 경우, 태그는 다음과 같을 수 있습니다:

```text
fire_skill
fire_skill2
fire_ult
```

챔피언 애니메이션의 경우, 태그는 보통 다음과 같습니다:

```text
idle
run
attack
skill
skill2
ult
dead
```

챔피언 액션에서 `action_name: "skill"`이라고 쓰면, 게임은 해당 챔피언의 `#anim` 데이터에서 `skill`이라는 이름의 애니메이션 태그를 찾습니다.

`fire_skill` 같은 스킬 아이콘 태그를 쓰면, UI는 아이콘 시트의 `#data` 데이터에서 `fire_skill`이라는 이름의 사각형 영역을 찾습니다.

## Aseprite 파일

`.aseprite` 파일은 모드에 직접 넣을 수 있습니다. 게임은 파일명으로 Aseprite 파일의 사용 방식을 결정하지 않습니다. 파일 내부의 스프라이트 사용자 데이터를 읽고, 그 메타데이터를 바탕으로 어떤 관련 에셋을 준비할지 결정합니다.

Aseprite 파일에 인식 가능한 `sheet_type`이 없으면 Aseprite 원본 파일로 불러오지만, 렌더링용 `#sheet`, `#anim`, `#data`는 자동으로 제공되지 않습니다.

중요한 부분은 Aseprite 스프라이트 사용자 데이터입니다. Aseprite에서 스프라이트 속성을 열고 사용자 데이터 텍스트 필드에 JSON을 입력하십시오.

일반적인 애니메이션 챔피언이나 효과의 경우:

```json
{
  "sheet_type": "Animation",
  "layers": ["body", "weapon"],
  "anchor_x": 0.5,
  "anchor_y": 0.5
}
```

이렇게 생성됩니다:

```text
asset/example/champions/fire_mage#sheet
asset/example/champions/fire_mage#anim
```

Aseprite의 타임라인 태그는 `#anim`의 애니메이션 태그가 됩니다. Aseprite 타임라인에 `attack`이라는 이름의 태그가 있으면, 게임은 `action_name: "attack"`으로 그 애니메이션을 재생할 수 있습니다.

프레임 타이밍도 Aseprite에서 가져옵니다. Aseprite 타임라인에서 어떤 프레임이 더 오래 지속되면, 게임에서도 더 오래 지속됩니다.

`layers`는 어떤 Aseprite 레이어를 조합해 애니메이션 프레임으로 만들지 게임에 알려줍니다. 이름은 반드시 Aseprite의 레이어 이름과 정확히 같게 유지하십시오.

`anchor_x`와 `anchor_y`는 원본 캔버스 내부의 정규화된 위치입니다. 기본값은 중앙인 `0.5, 0.5`입니다. 각 프레임에서 투명한 공간이 잘려 나가도 캐릭터가 시각적으로 고정되어 보이도록 도와줍니다.

짧게 요약하면 다음과 같습니다:

| `sheet_type` | 용도 | 제공하는 에셋 |
| --- | --- | --- |
| `Animation` | 챔피언 스프라이트 및 애니메이션 효과 | `#sheet`, `#anim` |
| `Sheet` | 전체 캔버스 크기를 유지하는 정지 이미지 시트 | `#sheet`, `#data` |
| `PackedSheet` | 빈 투명 영역을 잘라내는 정지 이미지 시트 | `#sheet`, `#data` |
| `LayeredAnimation` | 레이어 인식 애니메이션 설정 | `#sheet`, `#data`, `#anim`, `#lanim` |

정지 이미지 시트에는 다음을 사용하십시오:

```json
{
  "sheet_type": "Sheet"
}
```

또는:

```json
{
  "sheet_type": "PackedSheet"
}
```

둘 다 다음을 생성합니다:

```text
asset/example/ui/icons#sheet
asset/example/ui/icons#data
```

`Sheet`는 각 레이어/프레임 이미지를 전체 캔버스 크기로 유지합니다. `PackedSheet`는 각 이미지를 아틀라스에 배치하기 전에 주변의 빈 투명 영역을 잘라냅니다.

시트 파일의 태그 이름은 Aseprite 레이어 이름과 프레임 번호로 만들어집니다:

```text
<layer_name>_<frame_number>
```

`fire_skill`이라는 이름의 레이어가 있으면, 프레임 `0`은 다음과 같습니다:

```text
fire_skill_0
```

레이어 애니메이션의 경우:

```json
{
  "sheet_type": "LayeredAnimation",
  "layers": ["body", "weapon"]
}
```

이렇게 하면 `#sheet`, `#data`, `#anim`, `#lanim`이 생성됩니다. 이는 레이어 렌더링 동작이 특별히 필요할 때만 사용하십시오.

## PNG 및 JSON 스프라이트 시트

Aseprite는 편리하지만, 스프라이트 시트를 만드는 유일한 방법은 아닙니다. PNG 아틀라스와 JSON 데이터도 직접 제공할 수 있습니다.

다음 명명 규칙을 사용하십시오:

```text
mods/example/icons/spells#sheet.png
mods/example/icons/spells#data.sprite_sheet
```

게임은 이를 다음과 같이 불러옵니다:

```text
asset/example/icons/spells#sheet
asset/example/icons/spells#data
```

그러면 공용 기본 경로를 다음과 같이 참조할 수 있습니다:

```text
asset/example/icons/spells
```

`.sprite_sheet` 파일은 시트 안에 이름이 지정된 사각형 영역을 저장합니다. 사각형 영역은 정규화된 UV 값이며, `x: 0, y: 0`은 이미지의 좌측 상단이고 `w: 1, h: 1`은 이미지 전체 크기입니다.

```json
{
  "images": {
    "fire_skill": { "x": 0.0, "y": 0.0, "w": 0.25, "h": 0.5 },
    "fire_skill2": { "x": 0.25, "y": 0.0, "w": 0.25, "h": 0.5 },
    "fire_ult": { "x": 0.5, "y": 0.0, "w": 0.25, "h": 0.5 }
  }
}
```

이는 기술 아이콘, 아이템 아이콘, UI 아이콘, 줄내 텍스트 아이콘에 유용합니다.

무장 기술 아이콘의 경우, 데이터 무장은 다음과 같이 시트를 사용할 수 있습니다:

```json
{
  "skill_icon": {
    "source": "asset/example/icons/spells",
    "tags": ["fire_skill", "fire_skill2", "fire_ult"]
  }
}
```

각 아이콘이 개별 PNG라면, 대신 `skill_icons`를 사용하십시오:

```json
{
  "skill_icons": [
    "asset/example/icons/fire_skill",
    "asset/example/icons/fire_skill2",
    "asset/example/icons/fire_ult"
  ]
}
```

## PNG 및 JSON 동작화

PNG와 `.fanim` 파일을 짝지어 Aseprite 없이도 동작화 시트를 직접 제공할 수 있습니다.

다음 명명 규칙을 사용하십시오:

```text
mods/example/champions/fire_mage#sheet.png
mods/example/champions/fire_mage#anim.fanim
```

`.fanim` 파일은 이름이 지정된 동작화를 저장합니다. `.sprite_sheet`와 달리, 이 사각형 영역들은 시트 이미지 안의 픽셀 좌표입니다.

```json
{
  "anims": {
    "idle": {
      "frames": [
        { "duration": 0.12, "data": { "x": 0, "y": 0, "w": 64, "h": 64 } },
        { "duration": 0.12, "data": { "x": 64, "y": 0, "w": 64, "h": 64 } }
      ]
    },
    "attack": {
      "frames": [
        { "duration": 0.08, "data": { "x": 0, "y": 64, "w": 64, "h": 64 } },
        { "duration": 0.08, "data": { "x": 64, "y": 64, "w": 64, "h": 64 } }
      ]
    }
  }
}
```

데이터 챔피언은 해당 쌍을 자신의 스프라이트로 사용할 수 있습니다:

```json
{
  "sprite": "asset/example/champions/fire_mage",
  "anim_prefix": ""
}
```

비어 있는 `anim_prefix`는 애니메이션 태그가 원본 그대로 복사된다는 뜻입니다. 애니메이션 파일에 `idle`, `attack`, `skill`이 있으면, 챔피언도 동일한 태그를 받습니다.

## 하나의 애니메이션 파일 재사용하기

때로는 하나의 Aseprite 파일에 여러 캐릭터나 변형이 들어 있습니다. 그런 경우 Aseprite에서 태그에 접두사를 붙이십시오:

```text
eagle_idle
eagle_run
eagle_attack
eagle_skill
```

그런 다음 챔피언 데이터에서 `anim_prefix`를 설정하십시오:

```json
{
  "sprite": "asset/base/aseprite_resources/champions/druid",
  "anim_prefix": "eagle_"
}
```

챔피언은 다음 태그를 받습니다:

```text
idle
run
attack
skill
```

해당 접두사가 없는 태그도 그대로 유지됩니다. 이로써 동일한 원본 파일 안에서 공통 효과나 행동 태그를 함께 공유할 수 있습니다.

## 정적 PNG 스프라이트

빠른 시험을 위해 `sprite`가 PNG를 직접 가리키게 할 수 있습니다:

```json
{
  "sprite": "asset/example/champions/fire_mage_idle"
}
```

이 경우 `anim_prefix`는 설정하지 마십시오. 게임은 전체 PNG를 1프레임 스프라이트로 사용하고, 일반적인 챔피언 태그에 대해 단순한 대체 애니메이션을 생성합니다.

이는 챔피언이 제대로 불러와지는지 확인하는 데 좋습니다. 완성된 애니메이션 챔피언에는 Aseprite 파일이나 `#sheet`와 `#anim` 한 쌍을 사용하십시오.