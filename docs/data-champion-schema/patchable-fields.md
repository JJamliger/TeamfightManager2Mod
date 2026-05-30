# 패치 가능 필드

데이터 챔피언 행동은 게임의 `PatchableFields` 인터페이스를 구현합니다. 이는 일부 값을 패치/밸런스 계통에서 읽고 조정할 수 있음을 의미합니다.

## 행동 단계 필드

모든 `DataActionDef`는 다음을 노출합니다:

```text
cooltime //재사용 대기시간
range //원거리
duration //지속시간
start_timing //시작 시점
```

## 효과 단계 필드

해당 행동은 또한 그 효과의 일부 필드도 노출합니다.

| 효과 | 패치 가능 필드 |
| --- | --- |
| `Attack` | `damage`, `attack_ratio`, `hp_ratio`, `target_hp_ratio` |
| `ApAttack` | `damage`, `attack_ratio`, `hp_ratio` |
| `FixedAttack` | `damage`, `attack_ratio`, `hp_ratio`, `target_hp_ratio` |
| `Heal` | `amount`, `attack_ratio`, `ap_ratio` |
| `Shield` | `amount`, `attack_ratio`, `ap_ratio`, `tick` |
| `Stun` | `duration` |
| `Airborne` | `duration` |
| `Knockback` | `speed`, `tick` |
| `Fear` | `tick` |
| `Charm` | `tick` |
| `Bind` | `duration` |
| `Taunt` | `duration` |
| `Rush` | `speed`, `range` |
| `LinearProjectile` | `speed`, `range` |
| `Combine` | 첫 번째 하위 효과의 필드만 |

여기에 나열되지 않은 효과도 게임플레이에서는 여전히 작동할 수 있지만, 현재 이 인터페이스를 통해 패치 가능한 숫자 필드를 노출하지는 않습니다.

## 패치 유형 이름

데이터 행동의 패치 유형 이름은 해당 `action_name`입니다. 안정적인 이름을 우선 사용하십시오:

```text
attack
skill
skill2
ult
```

출시 후 `action_name`을 변경하면 패치/밸런스 참조 및 애니메이션 조회에 영향을 줄 수 있으므로, 안정적인 id처럼 취급하십시오.