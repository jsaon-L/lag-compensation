# lag-compensation


* https://zhuanlan.zhihu.com/p/1976699507237492478
* https://danieljimenezmorales.github.io/2023-10-29-the-art-of-hit-registration/
* https://www.riotgames.com/en/news/demolishing-wallhacks-valorants-fog-war?utm_source=chatgpt.com
* https://2xko.riotgames.com/en-us/news/dev/how-2xko-handles-online-play/?utm_source=chatgpt.com
* https://www.riotgames.com/en/news/valorants-128-tick-servers?utm_source=chatgpt.com
* https://www.riotgames.com/en/news/peeking-valorants-netcode
* https://playvalorant.com/en-us/news/dev/the-state-of-hit-registration/?utm_source=chatgpt.com


# 二、第一优先级：VALORANT 官方资料

如果你只看一套，我建议你先看 Riot 的。

## 1. VALORANT：The State of Hit Registration

[The State of Hit Registration — Riot / VALORANT](https://playvalorant.com/en-us/news/dev/the-state-of-hit-registration/?utm_source=chatgpt.com)

这是我认为**最适合你现在研究的问题**的一篇。

Riot 非常明确地描述了：

```text
Client Simulation
        ↓
Fire Input + Timestamp
        ↓
Server
        ↓
Rewind Simulation
        ↓
Hit Registration
        ↓
Server Result
        ↓
Client Hit VFX
```

而且它专门解释了一个很容易误解的问题：

> **Hit Registration Correctness ≠ Hit VFX Clarity**

例如：

```text
T0:
敌人在站立状态
玩家开枪
↓
服务器记录：Headshot

T0 + 100ms:
敌人已经蹲下

T0 + 100ms:
客户端收到 Hit Confirm
```

玩家看到的可能是：

```text
当前敌人：蹲着
Hit VFX：头部附近
```

于是玩家认为：

> “这不是身体吗？为什么是爆头？”

实际上服务器判断的是 **T0 的状态**。

这和你现在研究的问题高度相关。

Riot 明确说他们的 Hit Registration 是：

> server authoritative

而且客户端发送开枪时对应的 simulation timestamp，服务器再根据这个 timestamp 回退模拟状态。([英雄联盟][1])

---

# 三、第二篇：VALORANT 128 Tick Server

这个更偏工程实现。

[VALORANT's 128-Tick Servers — Riot Games](https://www.riotgames.com/en/news/valorants-128-tick-servers?utm_source=chatgpt.com)

这里有一个非常关键的信息：

### VALORANT 保存的不只是 Position

他们历史 Buffer 保存：

```text
Player Position
+
Animation State
```

收到射击包之后：

```text
Historical Buffer
        ↓
Rewind Position
        ↓
Rewind Animation
        ↓
Hit Detection
```

Riot 甚至明确提到：

> 为了准确进行射击判定，服务器需要运行玩家看到的相同 Animation。

因此他们把：

```text
Position
Animation State
```

一起放入历史数据。

([Riot Games][2])

---

# 四、这篇资料对你尤其重要：COD Black Ops III

这个我非常推荐你认真研究。

## Fighting Latency on Call of Duty: Black Ops III

[Fighting Latency on Call of Duty Black Ops III — GDC Slides](https://media.gdcvault.com/gdc2016/Presentations/Goyette_Benjamin_Fighting_Latency_COD.pdf?utm_source=chatgpt.com)

这里有一句对你现在的问题非常关键：

> **Snapshot Buffer originally did not store the animation state; only player position & orientation.**

也就是说，他们早期只保存：

```text
Position
Rotation
```

但是后来发现：

```text
Hit Detection
+
Current Animation State
```

会产生问题。

比如：

```text
T0:
玩家站立

T0 + 50ms:
玩家开始蹲下

T0 + 100ms:
服务器收到射击
```

如果你只 rewind：

```text
Position
Rotation
```

但是 Animation 还是当前状态：

```text
Historical Position
+
Current Animation
```

就可能得到错误 Hitbox。

所以后来需要：

```text
Historical Position
+
Historical Orientation
+
Historical Animation State
```

这和 VALORANT 的方案高度一致。([GDC Vault][3])

---

# 五、如果你要真正理解“射击游戏网络同步”，一定看 Halo Reach

## I Shot You First: Networking the Gameplay of HALO: REACH

[GDC — I Shot You First: Networking the Gameplay of HALO: REACH](https://www.gdcvault.com/play/1014345/IShotYouFirstNetworking?utm_source=chatgpt.com)

这是非常经典的一套 FPS 网络架构资料。

Bungie 的 David Aldridge 讲 Halo Reach 的：

* Network simulation
* Prediction
* Latency
* Hit detection
* Server authority
* Network inspection
* Lag compensation
* Gameplay synchronization

而且这个演讲被 Destiny 等 Bungie 后续网络架构资料多次引用。([GDC Vault][4])

如果你准备深入做：

> **竞技 FPS 网络战斗系统**

这套资料非常值得看。

---

# 六、Overwatch：你现在尤其应该研究

你之前问过我：

> 守望先锋 / Valorant 怎么做射击判定
> 源氏高速位移怎么办
> 无敌 / 增伤怎么和 Server Rewind 配合

所以我特别建议你看 Blizzard 的两套 GDC。

---

## 1. Overwatch Gameplay Architecture and Netcode

[Overwatch Gameplay Architecture and Netcode — GDC](https://gdcvault.com/play/1024001/-Overwatch-Gameplay-Architecture-and?utm_source=chatgpt.com)

这个非常重要。

它讨论：

```text
ECS
+
Deterministic Simulation
+
Networking
+
Prediction
+
Gameplay
```

而不是单纯讲“射线检测”。

Blizzard 的核心思想是：

> Gameplay Simulation 本身就是网络系统的一部分。

这对于你现在考虑的 GAS 架构非常有启发。

---

# 七、Overwatch 的 Weapons / Abilities 网络化

## Networking Scripted Weapons and Abilities in Overwatch

[Networking Scripted Weapons and Abilities in Overwatch — GDC](https://www.gdcvault.com/play/1024041/Networking-Scripted-Weapons-and-Abilities?utm_source=chatgpt.com)

这个甚至比上一个更贴近你的问题。

因为你关注的不只是：

```text
Bullet → Hitbox
```

而是：

```text
Bullet
  ↓
Hitbox
  ↓
Damage
  ↓
Shield
  ↓
Invulnerability
  ↓
Damage Boost
  ↓
Armor
  ↓
Death
```

这个演讲讨论的是：

* Weapons
* Abilities
* State Machine
* Prediction
* Replication
* Responsiveness
* Security
* Networked Gameplay

尤其值得你研究的是 **StateScript** 思路。

Overwatch 使用状态机来处理英雄技能和武器行为，并让网络同步成为 Gameplay State Machine 的一部分。([GDC Vault][5])

---

# 八、Valve Source：非常适合学习“最基础的 Rewind”

如果你想从最容易理解的实现开始：

## Source Multiplayer Networking

[Source Multiplayer Networking — Valve Developer Community](https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking?language=uk&utm_source=chatgpt.com)

Valve 把整个过程讲得非常清楚。

服务器维护最近一段时间的：

```text
Player Position History
```

玩家开枪：

```text
Client Fire
    ↓
UserCmd
    ↓
Server
    ↓
Calculate Command Time
    ↓
Rewind Players
    ↓
Hit Detection
    ↓
Restore Players
```

Source 的公式思想非常值得学习：

```text
Command Execution Time
=
Current Server Time
-
Packet Latency
-
Client View Interpolation
```

然后把玩家恢复到这个时间点。

([Valve Developer Community][6])

---

# 九、直接看 Valve 源码

这个特别推荐你收藏。

## Source SDK Lag Compensation

[Valve Source SDK 2013 — player_lagcompensation.cpp](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/game/server/player_lagcompensation.cpp?utm_source=chatgpt.com)

这是**真正的源码级资料**。

里面能看到：

```cpp
CLagCompensationManager
```

以及：

```cpp
StartLagCompensation()
```

具体怎么：

```text
计算 latency
↓
计算 interpolation
↓
计算 target tick
↓
rewind
↓
执行 hit detection
↓
restore
```

比如源码里明确存在：

```cpp
correct += nci->GetLatency(...)
correct += TICKS_TO_TIME(lerpTicks)
```

然后计算：

```cpp
targettick
```

再执行历史位置恢复。([GitHub][7])

---

# 十、一个你一定要看的资料：COD 的“动画回溯”

我甚至建议你把这个单独做成一个学习主题：

> **Historical Animation State**

因为很多初学者只想到：

```cpp
HistoricalTransform
```

但竞技 FPS 往往还需要：

```cpp
Historical Pose
Historical Hitbox
Historical Movement State
```

COD Black Ops III 的 GDC Slides 正好展示了这个问题。([GDC Vault][3])

这也是为什么：

```text
Server Rewind
```

不能简单理解成：

> “把 Actor Location 改回去。”

真正成熟的实现更接近：

```text
Historical World Query State
```

---

# 十一、然后就是你最关心的：Buff / Shield / Invulnerability

这一部分公开资料反而比较少。

商业游戏通常不会把：

```text
Damage Pipeline
+
Historical Gameplay State
```

完整公开。

但你可以从上面的资料反推出一个非常重要的架构。

---

## 不要设计成：

```cpp
RewindHitbox();

ApplyDamage(
    Target->GetCurrentDamageModifier()
);
```

这会产生严重 Bug。

例如：

```text
0ms
敌人：
Shield = 100
Invulnerable = false
DamageTaken = 100%

50ms
敌人获得无敌

100ms
服务器收到你的射击
```

你 rewind 到：

```text
0ms
```

发现：

```text
Hit = true
```

但是如果 Damage 系统读取：

```cpp
Target->IsInvulnerable()
```

拿到的是：

```text
true
```

于是：

```text
Hit happened at T0
but damage evaluated using T100
```

这就是典型的 **Temporal State Mismatch**。

---

# 十二、正确的思路应该是“Historical Combat State”

例如服务器每个 Tick 保存：

```cpp
struct FCombatHistory
{
    double ServerTime;

    // Spatial
    FTransform RootTransform;
    FHitBoxState HitBoxes;
    FAnimationState Animation;

    // Defensive
    float Health;
    float Shield;

    // Gameplay State
    bool bInvulnerable;
    bool bDamageImmune;

    // Modifiers
    float DamageTakenMultiplier;
    float DamageDealtMultiplier;

    // Status
    ECharacterState State;
    TArray<FActiveBuffSnapshot> Buffs;
};
```

于是：

```text
T0
│
├── Position
├── Hitbox
├── Animation
├── Shield
├── Invulnerability
├── Damage Modifier
└── Buffs
```

形成一个完整的：

> **Historical Combat Snapshot**

---

# 十三、我建议你把射击判定拆成 4 个阶段

这是我认为最适合你 UE5 项目的架构。

```text
                   Fire
                    │
                    ▼
          ┌──────────────────┐
          │ 1. Shot Validation│
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │ 2. Historical    │
          │    Rewind        │
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │ 3. Hit Resolution│
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │ 4. Damage        │
          │    Resolution    │
          └──────────────────┘
```

---

# 十四、第一层：Shot Validation

先判断这枪本身是否合法：

```text
Shooter
    │
    ├── Weapon exists?
    ├── Ammo?
    ├── Fire cooldown?
    ├── Fire rate?
    ├── Timestamp valid?
    ├── Rewind time within limit?
    └── Shooter state valid?
```

尤其：

```text
Client:
"I shot at T = 12345"
```

服务器不能无条件相信。

应该：

```text
Allowed Rewind Window

例如：
最大 200ms / 250ms / 300ms
```

VALORANT 也明确讨论了 rewind 上限，否则高延迟玩家可以利用回溯在自己已经躲到掩体之后仍然击杀目标。([Riot Games][8])

---

# 十五、第二层：Historical Rewind

只 rewind 与 Hit Detection 有关的东西。

例如：

```cpp
HistoricalState TargetState;

TargetState.Transform
TargetState.HitBoxes
TargetState.Pose
TargetState.MovementState
```

不要真的把整个世界：

```cpp
World->RollbackEverything();
```

成熟系统通常是：

```text
Query State
```

而不是：

```text
Whole World Rollback
```

---

# 十六、第三层：Hit Resolution

例如：

```text
Ray
 ↓
Head HitBox
 ↓
Body HitBox
 ↓
Arm HitBox
```

得到：

```cpp
FHitResult
{
    Target
    Bone
    Location
    Distance
    SurfaceType
}
```

此时还**不要马上 ApplyDamage**。

因为：

> Hit Detection ≠ Damage Resolution

这是你这个项目非常值得建立的概念。

---

# 十七、第四层：Damage Resolution

然后才：

```text
Historical Hit
        │
        ▼
Damage Context
        │
        ├── Shot Time
        ├── Hit Bone
        ├── Weapon
        ├── Base Damage
        ├── Attacker Historical State
        └── Target Historical State
                    │
                    ├── Shield
                    ├── Armor
                    ├── Invulnerable
                    ├── Damage Reduction
                    └── Buffs
```

最终：

```cpp
FinalDamage =
    BaseDamage
    * AttackerDamageMultiplier
    * TargetDamageTakenMultiplier
    * HitMultiplier;
```

---

# 十八、最重要的一个原则

我建议你把整个系统建立在一句话上：

> **A shot should be resolved against a coherent historical combat state.**

也就是：

```text
T_fire
```

这个时间点的：

```text
Position
Hitbox
Animation
Shield
Buff
Debuff
Invulnerability
Damage Modifier
```

必须属于**同一个时间切片**。

而不是：

```text
Hitbox       → T_fire
Animation    → T_fire
Shield       → T_now
Invincible   → T_now
Damage Buff  → T_now
```

---

# 十九、不过这里还有一个非常有意思的问题

你之前问过：

> “如果我回朔判定的时候命中了，但是现在已经获得无敌 Buff，怎么办？”

实际上存在两种设计哲学。

### A：Gameplay Event Time

```text
伤害发生时间 = T_fire
```

那么：

```text
T_fire 有无敌？
```

决定结果。

例如：

```text
T0  无敌结束
T1  玩家开枪
T2  玩家获得无敌
T3  Server 收到 Shot
```

则：

```text
Hit = true
Damage = true
```

因为攻击发生在 T1。

---

### B：Server Arrival Time

另一种设计：

```text
伤害事件正式产生
=
Server 收到请求的时间
```

那么：

```text
T3 有无敌
```

可能导致：

```text
Damage = 0
```

这种设计在某些特殊技能/机制中可能有意义。

---

# 二十、所以真正高级的系统不是简单地“所有 Buff 都 rewind”

而是给 Gameplay Effect 定义：

```text
Temporal Semantics
```

例如：

| 状态                   | 判断时间                  |
| -------------------- | --------------------- |
| Hitbox               | T_fire                |
| Animation            | T_fire                |
| Shield               | T_fire                |
| Armor                | T_fire                |
| Damage Boost         | T_fire                |
| Damage Taken         | T_fire                |
| Invulnerability      | 通常 T_fire             |
| Projectile Explosion | T_explosion           |
| DoT Tick             | T_tick                |
| Death                | Event Resolution Time |
| Respawn              | Server Timeline       |
| 后续触发效果               | Event Resolution Time |

这其实已经开始接近一个真正的：

> **Temporal Combat System**

---

# 二十一、UE5 项目可以直接借鉴的资料

你既然主要使用 UE5，我再给你两个可以直接看源码的。

### UE5 Server-Side Rewind Demo

[UE5 Server-Side Rewind — GitHub](https://github.com/marcohenning/ue5-server-side-rewind?utm_source=chatgpt.com)

这是一个比较干净的 UE5 C++ SSR Demo。

里面使用：

```cpp
FServerSideRewindSnapshot
```

保存：

```text
Hitbox Position
```

并维护：

```cpp
ServerSideRewindSnapshotHistory
```

然后在射击时：

```text
Find Historical Snapshot
↓
Rewind
↓
CheckForKill()
```

非常适合你先跑通第一版。([GitHub][9])

---

### UE5 Multiplayer Shooter

[UE5 C++ Multiplayer Shooter — GitHub](https://github.com/nbertoa/ue5-cpp-multiplayer-shooter?utm_source=chatgpt.com)

这个项目更加完整：

```text
Hit Scan
Projectile
Shotgun
Server Side Rewind
Weapon
Combat Component
Buff
Replication
```

它的 SSR 甚至维护了多个 HitBox 的历史记录。([GitHub][10])

---

# 二十二、如果你喜欢视频，这几个值得看

### ① Overwatch Gameplay Architecture and Netcode

[GDC — Overwatch Gameplay Architecture and Netcode](https://gdcvault.com/play/1024001/-Overwatch-Gameplay-Architecture-and?utm_source=chatgpt.com)

**优先级：★★★★★**

重点：

```text
Network Simulation
Determinism
Prediction
Gameplay Architecture
```

---

### ② Networking Scripted Weapons and Abilities in Overwatch

[GDC — Networking Scripted Weapons and Abilities in Overwatch](https://www.gdcvault.com/play/1024041/Networking-Scripted-Weapons-and-Abilities?utm_source=chatgpt.com)

**优先级：★★★★★**

你特别应该看：

```text
Weapon
Ability
State Machine
Replication
Prediction
```

---

### ③ I Shot You First: Halo Reach

[GDC — I Shot You First: Networking the Gameplay of HALO: REACH](https://www.gdcvault.com/play/1014345/IShotYouFirstNetworking?utm_source=chatgpt.com)

**优先级：★★★★★**

重点：

```text
FPS Network Model
Latency
Hit Detection
Prediction
Server Authority
```

---

### ④ COD Black Ops III — Fighting Latency

[Fighting Latency on Call of Duty Black Ops III](https://media.gdcvault.com/gdc2016/Presentations/Goyette_Benjamin_Fighting_Latency_COD.pdf?utm_source=chatgpt.com)

**优先级：★★★★★**

这个对你尤其重要：

```text
Snapshot Buffer
Hit Detection
Animation State
Historical State
```

---

# 二十三、还有一个非常好的现代资料：Rollback / Rewindable Entity System

## Knockout City's Parallel, Deterministic, and Rewindable Entity System

[GDC — Knockout City's Rewindable Entity System](https://gdcvault.com/play/1028073/-Knockout-City-s-Parallel?utm_source=chatgpt.com)

虽然不是 FPS，但是它讲的是：

```text
Entity
+
Deterministic
+
Parallel
+
Rewindable
```

而且不是只 rewind 一个位置，而是设计一个可以：

```text
Rewind World State
```

的 Entity System。

对于你以后做：

```text
Replay
Kill Cam
Server Rewind
Prediction
Rollback
Combat Simulation
```

都非常有参考价值。

---

# 二十四、我给你整理成一个学习路线

如果你的目标是：

> **最终自己在 UE5 + GAS 中实现类似 Valorant / Overwatch 的精准射击 + 状态判定**

我建议按这个顺序：

```text
                    FPS NETCODE
                         │
             ┌───────────┴───────────┐
             │                       │
       Client Prediction       Server Authority
             │                       │
             └───────────┬───────────┘
                         │
                  Lag Compensation
                         │
                         ▼
                  Server Rewind
                         │
              ┌──────────┴──────────┐
              │                     │
        Historical Hitbox     Historical State
              │                     │
       Position / Pose       Shield / Buff
       Animation             Invulnerability
       Movement              Damage Modifier
              │                     │
              └──────────┬──────────┘
                         │
                         ▼
                  Hit Resolution
                         │
                         ▼
                 Damage Resolution
                         │
                         ▼
                  Gameplay Event
```

---

# 二十五、你现在最应该重点研究的 5 个关键词

以后你搜索资料，不要只搜：

> `FPS hit detection`

而应该重点搜：

### 1. `Server-Side Rewind`

```text
server-side rewind
```

### 2. `Historical Hitbox`

```text
historical hitbox
historical pose
historical animation
```

### 3. `Lag Compensation`

```text
lag compensated hit detection
```

### 4. `Rewindable Gameplay State`

这是你现在最缺的一块：

```text
rewindable gameplay state
historical gameplay state
rollback gameplay state
```

### 5. `Temporal Gameplay State`

这个关键词非常值得你研究：

```text
temporal gameplay state
temporal combat system
time-aware gameplay state
```

因为你的真正问题已经不是：

> “怎么 rewind Hitbox？”

而是：

> **“一个发生在过去的 Gameplay Event，到底应该读取过去还是现在的 Gameplay State？”**

这是两个完全不同层次的问题。

---

## 最后，我特别建议你下一步这样做

如果你愿意，我可以继续帮你做一份更深入的：

### **《竞技 FPS 射击判定完整技术调研：Valorant / Overwatch / CS / CoD / Halo》**

我会重点把它拆成：

1. **Hitscan / Projectile / Shotgun 判定**
2. **Client Prediction**
3. **Server Rewind**
4. **Historical Hitbox**
5. **Historical Animation**
6. **移动、冲刺、闪现、瞬移的回溯**
7. **Shield / Armor**
8. **Damage Boost**
9. **Invulnerability**
10. **Damage Reduction**
11. **Buff / Debuff**
12. **GAS 如何接入**
13. **“攻击发生时间”和“服务器收到时间”冲突怎么解决**
14. **两个同时发生的事件如何排序**
15. **Kill / Death / Revive / Invulnerability 的时间线**
16. **UE5 C++ 数据结构**
17. **如何实现 Historical Combat Snapshot**
18. **如何避免真的 Rollback 整个 Actor**
19. **如何支持 128 Tick / 60 Tick**
20. **最后给你一套可以直接用于 UE5 项目的架构 + C++ 伪代码**

尤其是 **第 7～15 项**，我可以专门围绕你之前问的：

> **“源氏冲刺过程中被射击”**
> **“敌人刚获得无敌时收到一发旧子弹”**
> **“增伤 Buff 在开枪后、命中前发生变化”**
> **“护盾在开枪后被击破”**
> **“两个玩家同时开枪互杀”**

逐帧画时间线。

这部分会比单纯研究 `Server Rewind` 深很多，也是你现在做 UE5/GAS 射击游戏最值得建立的核心架构。

[1]: https://playvalorant.com/en-us/news/dev/the-state-of-hit-registration/?utm_source=chatgpt.com "The State of Hit Registration"
[2]: https://www.riotgames.com/en/news/valorants-128-tick-servers?utm_source=chatgpt.com "VALORANT's 128-Tick Servers | Riot Games"
[3]: https://media.gdcvault.com/gdc2016/Presentations/Goyette_Benjamin_Fighting_Latency_COD.pdf?utm_source=chatgpt.com "Microsoft PowerPoint - Fighting Latency on Call of Duty Black Ops III.pptx"
[4]: https://media.gdcvault.com/gdc2015/presentations/Truman_Justin_Shared_World_Shooter.pdf?utm_source=chatgpt.com "Slide 1"
[5]: https://www.gdcvault.com/play/1024041/Networking-Scripted-Weapons-and-Abilities?utm_source=chatgpt.com "GDC Vault - Networking Scripted Weapons and Abilities in 'Overwatch'"
[6]: https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking?language=uk&utm_source=chatgpt.com "Source Multiplayer Networking - Valve Developer Community"
[7]: https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/game/server/player_lagcompensation.cpp?utm_source=chatgpt.com "source-sdk-2013/src/game/server/player_lagcompensation.cpp at master · ValveSoftware/source-sdk-2013 · GitHub"
[8]: https://www.riotgames.com/en/news/peeking-valorants-netcode?utm_source=chatgpt.com "Peeking into VALORANT's Netcode | Riot Games"
[9]: https://github.com/marcohenning/ue5-server-side-rewind?utm_source=chatgpt.com "GitHub - marcohenning/ue5-server-side-rewind: A demo of lag compensation using server-side rewind in Unreal Engine 5 · GitHub"
[10]: https://github.com/nbertoa/ue5-cpp-multiplayer-shooter?utm_source=chatgpt.com "GitHub - nbertoa/ue5-cpp-multiplayer-shooter: Multiplayer FPS built in Unreal Engine 5 with C++. Features a custom Steam session plugin, server-side rewind lag compensation, hit-scan and projectile weapons, and a full match lifecycle system. · GitHub"
