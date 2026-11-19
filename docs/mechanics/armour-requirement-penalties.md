# Penalties for unmet stat requirements (PlayerLackOfStatsParam)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

2026-10-31. Row names: the developers' Japanese names in the pre-release PC regulation
(`DS2 Vanilla PC\2014-03-31`). Values are identical in PC Scholar 1.0.3.0. Code: PS3 retail v1.00 `EBOOT.elf`
(static, with the scratchpad tools ppc.py / xref.py / vcall.py; Ghidra was not connected).

Columns: stats min / max (applied when the shortfall `lack` > 0: min + (max - min) x lack, lack 0..1) and
stamina min / max (applied when stamina is short). Results are percentages.

| row | developer name | meaning | stats % | stamina % | armour shortfall in code? |
|---:|---|---|---|---|---|
| 0 | 攻撃力 減少割合 | attack power down | 30-80 | 0-10 | (weapon group, not traced) |
| 1 | スタミナ奪取量 減少割合 | stamina damage dealt down | 30-80 | 0-20 | |
| 2 | 耐久削り値 減少割合 | durability damage dealt down | 30-80 | 0-20 | |
| 3 | 気絶値 減少割合 | stun (poise damage) dealt down | 30-80 | 0-20 | |
| 4 | 受け能力 減少割合 | guard ability (stability) down | 30-80 | 0 | |
| 5 | カット率 減少割合 | block damage cut down | 30-80 | 0 | |
| 6 | ガード時スタミナ回復量 減少割合 | stamina regen while guarding down | 30-80 | 0 | |
| 7 | 攻撃アニメ速度 開始部 減少割合 | attack wind-up slower | 10-40 | 0-30 | |
| 8 | 攻撃アニメ速度 終了部 減少割合 | attack recovery slower | 10-30 | 0-20 | |
| 9 | ガードアニメ速度 構え部 減少割合 | raising guard slower | 10-30 | 0 | |
| 10 | スペル詠唱時間 増加割合 | casting time up | 50-100 | 0-50 | |
| 11 | 攻撃消費スタミナ 増加割合 | attack stamina cost up | 30-50 | 0 | |
| 12 | 武器耐久値消耗 増加割合 | weapon durability wear up | 50-100 | 0-100 | |
| 13 | 攻撃軌跡ブレ ブレンド率 | swing wobble blend | 40-60 | 40-60 | |
| 14 | 攻撃後よれ ブレンド率 | post-attack stagger blend | 60-90 | 60-90 | |
| 15 | 防御力 減少割合 | defence down (physical, probably) | 30-50 | 0 | not traced (li 15/16/17 cluster at 0x70ab88 / 0x70b148, receiver class unknown) |
| 16 | 抵抗力 減少割合 | resistance down (elemental, probably) | 30-50 | 0 | not traced |
| 17 | 耐性値 減少割合 | status resistance down | 30-50 | 0 | not traced |
| 18 | 回避無敵期間 減少割合 | dodge i-frames down | 30-50 | 0-50 | yes: 0x849a10, mode 2 ([roll i-frames and Agility](agility-iframes.md)) |
| 19 | 回避消費スタミナ 増加割合 | dodge stamina cost up | 30-50 | 0-50 | called from the roll / backstep cost functions (0x83a7b4, 0x83a914) with lack 0.0 and short 0: **no effect** |
| 20 | ローリング距離 減少割合 | roll distance down | 10-15 | 0-15 | yes: vslot +0x118 (0x7a57fc) -> +0x120 (0x84aac8, mode 2); call 0x7a41dc |
| 21 | スタミナ切れローリング距離 減少割合 | out-of-stamina roll distance down | 10-15 | 0-15 | yes, call 0x7a40b8 |
| 22 | ステップ距離 減少割合 | backstep distance down | 30-50 | 0-30 | yes, call 0x7a3fcc |
| 23 | スタミナ切れステップ距離 減少割合 | out-of-stamina backstep distance down | 10-25 | 0-20 | yes, call 0x7a3ecc |
| 24 | ジャンプ速度 減少割合 | jump speed down | 0-30 | 0 | yes: 0x84a528 -> vslot +0x120 |
| 25 | 防具耐久値消耗 増加割合 | armour durability wear up | 50-100 | 0 | yes, per piece: 0x5ab310, lack read from a per-piece table, [chr+0x2e4] vslot +0x2c |
| 26 | アイテム使用アニメ速度 減少割合 | item-use animation slower | 0-20 | 0 | not traced (only a 19/25/26 cluster at 0xb0429c) |
| 27 | 攻撃時 強靭度 減少割合 | poise while attacking down | 30-80 | 0-30 | not traced (0x45ec48, other class) |
| 28 | 強靭度 減少割合 | poise down | 30-80 | 0-30 | yes, per piece: 0x849b50 / ba4 / bf4 / c44, lack = piece +0x1b8, vslot +0x30 |

Rows 15-17 wording, from the PS3 v1.00 Japanese menu text (`text\jp_menu`): the UI uses 防御力 for physical and
elemental defence alike (物理 / 魔法 / 炎 / 雷 / 闇防御力) and 耐性 for poison / bleed / petrify / curse (and equipment
break). 抵抗力 appears only in flavour text, always as resistance to magic or fire (Black / Straid / pyromancer sets).
With three separate rows the likely split is physical / elemental / status; not confirmed in code.

Mechanics: `FUN_007a4b28(out, obj, mode)` is the only shortfall getter: mode 2 = equipment +0x214 (armour),
mode 0 = +0x218 (no direct caller found). Its two callers both use mode 2. Class vtable 0x1c2abe0 ([chr+0x2e4]):
+0x2c / +0x30 = penalty for row r4 with the caller's lack in f1 (stamina term 0), +0x34 = compute from {row, lack,
short}, +0x38 = 1 + penalty, +0x3c = 1 - penalty, +0x118 = movement helper (row in r9), +0x120 = penalty for row r4
with the armour lack and current stamina shortfall.

Open: rows 15-17, 26, 27 and the whole weapon group ([how weapon shortfall is fed in](sl1-naked-damage.md#under-requirement)). PC Scholar code not checked.
