# 열거형

## `ChampionCategory`

```text
Melee //근접
Range //원거리
Magician //마법사
Util //지원
Assassin //암살자
```

기본값: `근접`.

## `ChampionTag`

```text
AD
AP
Heal //치유
Shield //방패
Dot //지속 피해
CC //군중 제어
Range //원거리
Melee //근접
Tank //탱커
Magic //마법
```

## `CastingType`

```text
Targeting //대상 지정
Position //위치
Direction //방향
None //없음
```

기본값: `대상 지정`.

- `대상 지정`: 대상 개체 입력.
- `위치`: 위치 입력.
- `방향`: 시전자로부터의 방향 입력.
- `없음`: 대상 입력 없음.

## `CastingTarget`

```text
Ally //아군
AllyChampion //아군 챔피언
AllyChampionInCC //군중 제어 상태의 아군 챔피언
AllyNotSelf //자신 제외 아군
AllyOnlySelf //자기 자신만
Enemy //적군
EnemyWithoutTower //포탑 제외 적군
EnemyChampion //적군 챔피언
EnemyChampionInCC //군중 제어 상태의 적군 챔피언
EnemyChampionRecentlyAttacked //최근 공격한 적군 챔피언
Both //양측
BothWithoutTower //포탑 제외 양측
BothChampion //양측 챔피언
None //없음
```

기본값: `아군`.

기본값이 `아군`이므로, 피해 및 투사체 효과는 보통 `casting_target`, `applied_target`, 또는 `target`을 명시적으로 설정해야 합니다.

## `AttackType`

```text
BaseAttack //기본 공격
skill
Dot //지속 피해
DotIgnoreShield //보호막 무시 지속 피해
Item //아이템
Well //
```

기본값: `기본 공격`.

의도적으로 다른 분류가 필요하지 않은 한, `attack` 슬롯에는 `기본 공격`을, `skill`, `skill2`, `ult`에는 `스킬`을 사용하십시오.

## `ProjectileShape`

```json
{ "Circle": { "radius": 10000 } }
```

```json
{ "Line": { "width": 8000, "from_x": 0, "from_y": 0, "to_x": 50000, "to_y": 0 } }
```

```json
{ "Rect": { "width": 30000, "height": 16000 } }
```

```json
{ "DirDot": { "radius": 8000, "range": 700 } }
```

기본값: `{ "Circle": { "radius": 10000 } }`.

## `DataAttackEffectType`

```json
"Target"
```

```json
"EnemyTarget"
```

```json
{ "EnemyAll": { "range": 40000 } }
```

기본값: `대상`.

## `DataHealType`

```json
"Caster"
```

```json
"Any"
```

```json
"Ally"
```

```json
{ "AllyAll": { "range": 40000 } }
```

기본값: `임의`.

## `DataRangeApplyType`

```json
"AroundCaster" //시전자 주변
```

```json
{ "Forward": { "offset": 24000 } }
```

기본값: `시전자 주변`.

`전방`은 시전자를 기준으로 오프셋을 두고, 행동 입력 방향에 영역을 배치합니다.


## `DataCastedType`

```json
"Bleed" //방패 포함
```

```json
"Poison" //중독
```

```json
"Fire" //화염
```

```json
"Heal" //치유
```

기본값: `화염`.

`AddCasted`에서 지속 효과 범주/아이콘과 공격 유형 동작을 선택하는 데 사용됩니다. `Poison`은 방패를 무시하는 DOT 공격 유형을 사용하며, 그 밖의 피해를 주는 캐스트 유형은 일반 DOT 동작을 사용합니다.

## `BuffType`

```json
"Permanent" //영구
```

```json
{ "Time": { "tick": 180 } }
```

```json
"WithShield" //방패 포함
```

기본값: `영구`.