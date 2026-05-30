# 데이터 챔피언 스키마

이 폴더는 `.data_champion` 파일의 전체 참고 문서입니다.

작은 작동 예제를 보려면 먼저 [데이터 전용 챔피언](../data-champion.md)부터 시작하십시오. 현재 데이터 챔피언 불러오기 기능이 허용하는 모든 필드, enum 값, 효과 유형, 시각 바인딩이 필요할 때 이 참고 문서를 사용하십시오.

## 페이지

- [챔피언 파일](champion-file.md): 최상위 `.data_champion` 필드, 능력치, 아이콘, 필수 행동 슬롯.
- [행동](actions.md): `attack`, `skill`, `skill2`, `ult` 행동 스키마.
- [효과](effects.md): 모든 `effect.type` 값과 그 JSON 형태.
- [버프 및 능력치](buffs-and-stats.md): `stat`, `growth`, `buff_state`, 버프 지속시간 스키마.
- [시각 바인딩](visual-bindings.md): `view_effects`, `view_projectiles`, `view_buffs`, 그리고 이름이 효과와 연결되는 방식.
- [열거형](enums.md): 허용되는 enum 문자열과 외부 태그형 enum 객체.
- [패치 가능 필드](patchable-fields.md): 패치/밸런스 계통으로 조정할 수 있는 필드.
- [조합법](recipes.md): 복사하여 응용할 수 있는 일반적인 패턴.

## JSON 관례

대부분의 데이터 챔피언 효과는 내부 태그형 형식을 사용합니다:

```json
{
  "type": "LinearProjectile", //직선투사체
  "name": "fire_skill",
  "speed": 4200,
  "range": 65000
}
```

일부 보조 enum은 Rust/serde의 외부 태그형 형식을 사용합니다. 단일 값은 문자열입니다:

```json
"Targeting"
```

필드가 있는 enum 값은 객체입니다:

```json
{ "Time": { "tick": 180 } }
```

어떤 페이지에서 필드가 필수라고 명시하지 않는 한, 기본값 사용 가능으로 표시된 필드는 생략할 수 있습니다. 명확성을 위해 예시에는 보통 `casting_target` 및 `applied_target` 같은 중요한 기본값이 포함되는데, 이는 Rust 기본값이 항상 모드 제작자가 원하는 것과 일치하지는 않기 때문입니다.