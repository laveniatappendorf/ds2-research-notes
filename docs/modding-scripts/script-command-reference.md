# Event and talk script commands: what we proved (PC Scholar 1.0.3.0, 2026-11-17)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

> Dark Souls II: Scholar of the First Sin, PC, Steam 1.0.3.0. Written 17 November 2026 from the Boss Arena project's tests.

## Summary

- This is the lookup table behind the other pages: every event-script and talk-script command or function the Boss Arena used or tried, with its developer name from the game's `.edd` debug files, the arguments as we used them, what it did live, the test id, and how often FromSoftware's own scripts use it. Each row points to the page that tells its story.
- Event scripts and talk scripts share one table of names (325 commands, 131 functions in the PC build). Commands (`1:130101`) do things. Functions (`f130421()`) read state. **The two id spaces overlap**: `f130441` is not `1:130441`.
- The mod's scripts rest on a small kit, all [live]: `1:130101` flags, `1:132125` warps, `1:131721` enemy removal, `1:140402` + `f140001` ("wait until the player is in this map"), `1:130334` player effects, Talk-script windows (`1:130441`, `1:130443`, `1:130481`), `1:131402` item lots, and `1:1` + `f1` dice.
- Proven not to work: an event-script YES/NO opened without an action-button press (B20c), object-state requests that restore an object (B25), a stored death count compared with the live one (B50 to B50c), and a direct call of the reallocation window from a Talk question (B30). No named command lights a bonfire or refills Estus.
- **A correction.** Earlier project notes, and some other pages of this wiki, say the four "set HP" commands, `130311` to `130314`, are "not implemented in the exe". Reading the exe again, the character-event handler has real cases for them. They were never called live. See [How the exe finds a command](#how-the-exe-finds-a-command).
- The trap when reading the exe: a command that lands in one handler's *default* case is not necessarily unimplemented. That handler passes it up a chain.

## How it works

A script state runs commands when it is entered, and its conditions read functions. The game looks each command id up in a chain of handlers in the exe (the last section of this part). The tables below are what we learnt about the ids, in nine groups.

### How to read the tables

- **Forms.** `1:132125(...)` is a command in bank 1. `5:130101(cond, flag, value)` is the conditional form (the condition comes first; only some commands have it). `6:2147483635(text)` calls a library state machine. `f130421()` is a function used inside a condition. [data]
- **Names.** Developer names are the `.edd` labels, read with `edd.py`. The English glosses are ours.
- **Orig.** How many times FromSoftware's original scripts use it, as **E** (28 event scripts) and **T** (25 talk scripts). Bank 1 and the conditional bank 5 are added together. Counted by walking every state, condition and command of the original scripts (the mod's own additions excluded), 17 Nov 2026. [data]
- **Proof tags.** `[live B9]` seen in the game; `[code]` read in the exe; `[data]` read in the game's files; `[inference]`; `[untested]`. The test ids point to the project's test log.
- **Not tried** means the originals use it (or its name exists) but we never ran it.

### 1. Event groups

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:130001()` | 【システム】終了通知, "end notification" | FromSoftware's groups call it in their last state. **None of the groups we added does**, and they ran in every test. [Event script groups](event-script-groups.md) | [live B9 onward] | E 1806 · T 96 |
| `1:130002()` | 【システム】再実行通知, "re-run notification" | Not used by us | [data] | E 594 |
| `1:130003(event, ...)` | 【システム】他のイベントを実行, "execute another event" | Not used by us. A rare route (3 uses in all) | [data] | E 2 · T 1 |
| `f130031(group)` | 【システム】イベント終了判定, "has the event ended?" | Gilligan's list script waits on his row's group with it; we use flags instead | [data] | E 62 · T 4 |

### 2. Warps, the respawn record, the map

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:132125(0, 0, map, location, 0)` | 【マップ】プレイヤーをポリ劇ワープ, "[map] warp the player (polygon-drama warp)" | A **full map reload** into `map` (the 8-digit id: 10040000 is m10_04) at EventLocation row `location`. We always pass `0, 0, ..., 0`. The originals pass the last argument 1 once (Things Betwixt) and 2 once (a warp inside m50_37); the meaning of the three constant arguments is unknown. **Leaves the respawn record alone.** [Warping](../modding-world/warping-by-script.md), [Respawn point](../modding-world/respawn-point.md) | [live B8, B9, B34, B38, B39, B52]; the originals' in-map form seen reloading the same blocks [live A8] | E 25 |
| `1:132121(location)` | 【マップ】プレイヤーをエリア内でワープ, "warp the player within the area" | One argument, an EventLocation row. Moves the player **without a load**; FromSoftware's boss libraries and cutscenes use it, so a cutscene is not a load. Also exists as `5:132121(cond, location)`. We never call it | [data] (B38 reasons from it) | E 19 |
| `1:132141(location)` | 【マップ】リトライポイント設定, "set the retry point" | Writes the respawn record `{the map the script runs in, kind 1, location}`. We run it before warps to Majula's landing (200000) and in arrival guards. The original's one use is the Throne of Want's own retry point. [Respawn point](../modding-world/respawn-point.md) | [live B19, B38, B44] | E 1 |
| `1:132132(region, 2)` | 【マップ】ナビメッシュ属性削除, "delete a navmesh attribute" | In our Belfry gate group as `(400000, 2)`, copied from the bell-lever library. The group worked as a whole; this call's own effect was not isolated. | [live B19, not isolated] | E 338 |

### 3. Flags and variables

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:130101(flag, value)` | 【フラグ】フラグを設定, "set a flag" | `value` 0 or 1. Reaches global flags (blocks 10 and 20) and the current map's blocks; the game's setter refuses a flag whose block is not loaded. Works from Talk scripts too. Our builds hold thousands of calls. [Event flags](event-flags-and-free-ids.md) | [live A1, B8, B9, B16, B20d] | E 1004 · T 1132 (206 of them `5:`) |
| `1:130131(first, last, 0)` | 【フラグ】パラメータに設定されている指定範囲のフラグをまとめて設定, "set a range of flags from the param, together" | Resets EventFlagListParam rows `first` to `last` (e.g. a pardon menu's `3500, 3999`). Not used by us. [Event flags](event-flags-and-free-ids.md) | [data] | E 26 · T 2 |
| `1:130122(var, value)`, `1:130124(var, n)`, `f130122(var)` | 【フラグ】グローバル変数設定 / 加算 / 取得, "set / add / get a global variable" | We stored the death count in variable 236 and compared it later: the comparison fired with no death and looped the player back into a fight. The notes infer that script global variables do not survive a map load; **that is not proven**. [Detecting a death](detecting-death-in-scripts.md) | [live B50, B50b, B50c: failed] | set E 9 · T 22; add E 11; get E 24 · T 114 |
| `1:130121` / `1:130123`, `f130121(var)` | 【フラグ】エリア変数設定 / 加算 / 取得, "set / add / get an area variable" | The area-variable twins of the above. Never tried | [data] | set E 56 · T 27; add E 26 · T 6; get E 66 · T 130 |
| `f6(flag)` | フラグの取得, "get a flag" | The flag read in every condition we wrote | [live all builds] | E 495 · T 1952 |
| `f3()` | ステート経過時間の取得, "get the time in this state" | Seconds since the state began: `f3() > 1`. Every timed step in our groups | [live all builds] | E 209 · T 66 |
| `1:140402(slot, 1, 0)` then `f140001(slot)` | 【グループ条件：マップ】プレイヤーがマップ内にいる判定, "group condition (map): is the player inside the map"; 【グループ条件】グループ条件を満たす, "group condition is met" | `140402(0, 1, 0)` registers the condition in slot 0. `f140001(0)` is true once the player is in **this** map. The second argument is 1 in 202 of the 247 original uses, 0 in 45; the third is always 0. Our load groups wait for it before deciding, because a neighbouring map that streams in runs its own scripts. [Warping](../modding-world/warping-by-script.md), [Event script groups](event-script-groups.md) | [live B39c] | 140402 E 247; `f140001` E 3819 |

### 4. The player: status, effects, covenant

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:130334(effect, 19, 4)` | 【ステータス】プレイヤーの特殊効果を発動, "trigger a special effect on the player" | `effect` is a SpEffect id: 110000010 (bonfire rest: HP, Estus, spells), 100000020 (HP, spells, repair, un-hollow; **no Estus**), 60410000 (Repair Powder), 100000010 (return to human). `19, 4` copied from the originals, meaning unknown. A duration-0 effect acts once. [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md) | [live B21, B56, B57] | E 3 |
| `1:130336(effect)` | 【ステータス】プレイヤーの特殊効果を解除, "cancel a special effect on the player" | Never exercised (B56 step 4 had nothing to cancel). No original uses it | [untested] | 0 |
| `1:130333(chr, effect, 19, 4)`, `1:130335(chr, effect)`, `1:140182` | 【ステータス】指定の敵の特殊効果を発動 / 解除, "trigger / cancel a special effect on a chosen enemy"; 【グループ条件：キャラ】プレイヤーの特殊効果判定, "group condition: does the player have an effect" | The enemy forms of the two above, and a test for a player effect. Not tried. In the enemy form the third argument is 19 in 37 of the 41 uses and 10 in 4, so `19` is a selector, not a constant | [data] | E 41; E 32; 0 |
| `1:130303(1)` / `(0)` | 【ステータス】プレイヤー無敵設定, "player invincibility setting" | No damage **and no hit reactions**; a load resets it. [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md), [Difficulty levers](../modding-world/difficulty-levers.md) | [live B56 step 5] | E 28 |
| `1:130301()` | 【ステータス】現在のステートにいる間だけ無敵状態にする, "invincible only while in the current state" | Not used by us | [data] | E 3 |
| `1:130307()` | 【ステータス】プレイヤーの落下ダメージを一度だけ無効化, "cancel the player's fall damage once" | Not tried (do not confuse with `f130307`) | [data] | E 1 |
| `1:130311(hp)`, `130312(ratio)`, `130313(min)`, `130314(ratio)` | 【ステータス】HPを直接設定 / HPを割合で設定 / 最低HPを直接設定 / 最低HPを割合で設定, "set HP / HP by ratio / minimum HP / minimum HP by ratio" | **Never used by FromSoftware and never called by us.** The exe has real cases for them (see below): they write the HP at +0x168, and the minimum at +0x16C, of the object at the handler's +0xD8. +0x16C is the field the cheat table writes for "Can't die". Whether an event script reaches that code, and what the object is there, is unknown | [code]; [untested] | 0 |
| `1:130317(generator, ratio)` | 【ステータス】ジェネレータ指定でHP割合を設定, "set the HP ratio, by generator" | Used once by FromSoftware, on Freja's generator 803, so it reaches a boss. Has its own case in the exe. Not tried by us. [Difficulty levers](../modding-world/difficulty-levers.md) | [data, code] | E 1 |
| `1:130381(covenant)` | 【ステータス】プレイヤーの誓約を変更, "change the player's covenant" | `(8)` joins the Company of Champions (enemy damage ×1.5, no enemy HP change); `(0)` leaves. Runs from our Majula groups. [Difficulty levers](../modding-world/difficulty-levers.md) | [live B56 step 6, B57] | T 48 |
| `f130361()` | 【ステータス】プレイヤーの誓約を取得, "get the player's covenant" | Our Hard-mode row is shown from `f130361() == 8` | [live B57] | E 6 · T 258 |
| `f130312()` | 【ステータス】プレイヤーのHP取得, "get the player's HP" | Current HP. **Negative on an overkill death** (-592, -41). `f130312() < 1`, once HP above 0 has been seen in this load, was the death test that worked. Not the command `1:130312`. [Detecting a death](detecting-death-in-scripts.md) | [live B50c, B57] | 0 |
| `f130308(0)` | 【ステータス】プレイヤーの霊体タイプ別死亡回数を取得, "get the player's death count by spirit type" | The argument 0 is what the Herald's talk passes. Stored in variable 236 and compared: failed (above) | [live B50 to B50c] | T 1 |
| `f130307()` | 【ステータス】プレイヤーの生者亡者タイプを取得, "get the player's living / hollow type" | Read 0 before and after a return to human; the values' meaning is unsettled | [live B21, read only] | T 13 |
| `f130301()`, `f130302()`, `f130303()`, `f130321()` | HP取得 / HP割合取得 / 死亡判定 / プレイヤーを殺した, "get HP / HP ratio / death judgement / killed the player" | In the name table, never tried by us. The first two are used a lot in talk scripts | [data] | T 371 / T 22 / 0 / T 366 |
| `1:130613(1)` | 【ネット】マルチプレイ禁止, "forbid multiplayer" | We run `130613(1)` before every warp into the Darklurker's chasm, copying the castle's circle. It ran with the fight list and the rush; whether it stopped anything was not checked | [live B50 onward, effect not observed] | E 31 |

### 5. Enemies and generators

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:131721(generator, 0)` | 【ジェネレータ】敵の強制削除, "force-delete an enemy" | In a load-time group the enemy **never exists** (no CharacterCtrl in memory). Second argument 0 in 9 of the 13 original uses, 1 in 2; ours is always 0. The roll-out holds 133 of them. [Removing enemies](../modding-world/removing-enemies.md) | [live B7, B8, B9, B10 to B19] | E 13 |
| `1:131731(generator, 1)` | 【ジェネレータ】指定の敵の一時停止, "pause the specified enemy" | Same visible result as delete in B7 (the enemy gone, none in memory). `0` resumes, as the group form's uses show. Our later builds delete instead | [live B7, B8] | E 2 |
| `1:131722(group, 0)`, `1:131732(group, 1 or 0)` | …ジェネレータグループ指定で敵の強制削除 / 一時停止, "force-delete / pause enemies by generator group" | Majula's script uses `131722` 70 times for its NPC groups. Never tried by us | [data] | E 70 · T 2; E 4 |

### 6. Windows and menus

The stories are on [YES/NO windows](../modding-menus/yes-no-windows.md), [Adding dialogue options](../modding-menus/adding-dialogue-options.md) and [NPC menus](../modding-menus/npc-menus.md).

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:130441(text, 0, -1, 0, 0, 0)` | 【FE】YES／NO選択ウィンドウの表示, "show a YES/NO window" | From Talk, through the library group 2147483635, it works. The event form `(text, 0, 190, 0, 0, 0)` opened and was **cancelled at once**: the game cancels a script-opened YES/NO unless an action-button press led to it. | [live B16, B20d, B43]; [live B20 to B20c: failed] | E 32 · T 29 |
| `f130422()`, `f130421()` | 【FE】YES／NO選択ウィンドウ表示判定 / 結果取得, "is it shown / get its result" | Wait for `f130422() != 0`, then read `f130421()`: 1 YES, 2 NO, 3 cancelled | [live B20d, B41 run 3] | f130422 E 49 · T 54; f130421 E 129 · T 86 |
| `1:130443(text, 0, 0, -1, 0, 0, 0)`, `f130431()` | 【FE】YESのみウィンドウの表示 / 表示判定, "show a YES-only window / is it shown" | The "OK" windows, through library 2147483628 | [live B20d] | E 43 · T 20; f E 86 · T 40 |
| `f130432()` | 【FE】イベントメッセージウィンドウ表示判定, "is an event message window shown" | Probed in B20c when the first question was due: 0, so the area-name banner was not what cancelled it | [live B20c] | E 13 |
| `1:130442()` | 【FE】YES／NO選択ウィンドウを閉じる, "close the YES/NO window" | We run it before opening an N-choice list. Acts only when the talk tracker is in state 1 | [live B36c] | 0 |
| `1:130444()` | 【FE】YESのみウィンドウを閉じる, "close the YES-only window" | Not used by anyone | [data] | 0 |
| `1:130445`, `1:130446` | the two windows "with an object instance id" | Not tried | [data] | E 30, E 17 |
| `1:130408(34 or 9, n)`, `1:130407()` | 【FE】キーガイド領域の作成 / を削除, "create / delete a key-guide area" | The "press the action button here" prompt. The first argument is 34 in all 38 event uses and 9 in all 84 talk uses; the rest is unknown. Our event YES/NO opened without one and failed | [data]; absence [live B20b, B20c] | 408: E 38 · T 84; 407: E 62 · T 248 |
| `1:130461(a, b, c, row)` | 【FE】NPCウィンドウを開く, "open the NPC window" | `row` is an NpcEventParam row. Run through library 2147483634. Talk only | [live B31, B36c] | T 19 |
| `1:130462()`, `130463()`, `130464()`, `130465()` | …ウィンドウを閉じる / 結果をクリア / 「何を選択したか」をクリア / 「何かを実行したか」をクリア, "close the NPC window / clear its result / clear 'what was chosen' / clear 'did something'" | `130462; 130464` are two of the four closes before a list | [live B36c] | T 131 / 248 / 116 / 1 |
| `f130451()`, `f130452()`, `f130453()` | 【FE】NPCウィンドウ表示判定 / の結果取得 / の使用結果取得, "is it shown / its result / its use result" | `f130452()`: 16 Talk, 19 cancelled, 23 a list has just closed | [live B36c] | T 147 / 362 / 25 |
| `1:130481(first, last, 0, title, key guide)`, `1:130482()` | 【FE】N択ウィンドウの表示 / を閉じる, "show / close the N-choice window" | The list over PluralSelectMenuParam rows (12 rows at most). PC ignores the title and key-guide arguments. Refused while the NPC menu is open, silently | [live B36c, B37, B50, B51, B57] | 481: T 1; 482: T 3 |
| `f130423()`, `f130433()` | 【FE】N択ウィンドウの結果取得 / 表示判定, "get its result / is it shown" | `f130423()`: 1 a pick, 3 cancelled; it does not say which row | [live B36c] | T 1; 0 |
| `1:130455(0, 220, 0)`, `f130434()` | 【FE】ステータス再設定ウィンドウを開く / 表示判定, "open the status-reset window / is it shown" | Works after the NPC menu returns Reallocate (result 11), with `1:132301(9)` first and a wait for `f130434` to open and shut. **A direct call from a Talk question chain only played the animation.** The Herald's own level-up passes a third argument of 1 | [live B30 fail, B31 pass]; level-up detail [data] | T 2; f T 5 |

### 7. Items, souls, saving, player actions

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:131402(lot, 1)` | 【アイテム】アイテム追加, "add an item" | `lot` is an ItemLotParam2_Other row, not an item. The second argument is 1 in every original use. One pop-up per lot. [Giving items](../modding-menus/giving-items.md) | [live B20d, B26, B47] | E 18 · T 56 |
| `f130412()` | 【FE】アイテム取得ウィンドウ表示判定, "is the item-get pop-up shown" | Wait for `!= 0`, then `!= 1`; our own 3 s fallback covers a key item already held | [live B20d] | E 3 · T 95 |
| `1:130494(...)`, `f130413()` | 【FE】アイテム取得失敗ウィンドウを表示 / 表示判定, "show the item-get-failed window / is it shown" | Not used by us; the setup's 3 s fallback does the job | [data] | 494: E 7 · T 23; f E 14 · T 46 |
| `1:132241()` | 【その他】セーブ実行, "execute a save" | The last state of the setup. The result survived a quit and reload. Not isolated from the game's autosave (a command, `1:132242`, exists to disable it, so the autosave is a real mechanism) | [live B20d, consistent] | E 22 · T 46 |
| `1:132211(souls)` | 【その他】ソウル値加算, "add souls" | The originals pass a negative value to take souls. Not used by us | [data] | E 20 · T 3 |
| `1:132301(n)`, `1:132303()` | 【キャラアクション】プレイヤーにアクションリクエスト / アクション終了リクエスト, "action request / action-end request to the player" | `132301(9)` plus `130455` (above) is the reallocation sequence. Other numbers in the originals: 1 the Herald's level-up, 5 lighting a Primal Bonfire, 6 a grave or the altar of the living, 14 a duel request. `132303()` ends it | [live B31 for 9]; the rest [data] | 301: E 60 · T 33; 303: E 29 · T 91 |

### 8. Objects

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:131636(object, state)`, `f131653(object, state, 0)` | 【OBJ】OBJステート遷移リクエスト / OBJステートID比較, "request an object state transition / compare an object's state id (by instance id)" | **Worked**: the Belfry gate, state 10 (closed) to 70 (opening), then 20 (open). **Did not work**: restoring the Fume Knight's idols (33 / 34) or Mytha's pools (20 to 10). [Map object states](map-object-states.md) | [live B19]; [live B25: failed] | 636: E 729; f: E 739 · T 1 |
| `1:131638(object, 1)` | 【OBJ】OBJのキーガイドを無効にする, "disable an object's key guide" | We lock the Throne's two ending objects (`131636(obj, 10)` plus this) at every castle load. It ran in every castle test; not tested with the lock removed | [live B16 onward, not isolated] | E 266 |
| `1:131622(fog, 0)` | 【OBJ】白扉のキーガイドを無効にする, "disable a white door's key guide" | In the Belfry gate group; effect not isolated | [live B19, not isolated] | E 4 |
| `1:131631(object)`, `1:131632(object)` | 【OBJ】OBJをタッチする, "touch an object" | The only candidates for "touch a bonfire". No original script uses either. [Bonfire lit state](../modding-world/bonfire-lit-state.md) | [untested] | 0 |

### 9. Dice, fights, and what does not exist

| Command | Developer name | What we did and saw | Proof | Orig. |
|---|---|---|---|---|
| `1:1(slot, min, max)`, `f1(slot)` | 【乱数】乱数生成, "generate a random number"; 乱数値の取得, "get the random value" | `1:1(0, 1, 39)` rolls into slot 0; `f1(0) == k` then works **inside a condition**. Both ends of the range came up (fight 1 and fight 39). [Randomness](randomness-in-scripts.md) | [live B52] | E 39 · T 64 · f E 68 · T 318 |
| `1:140531(...)` | 【グループ条件：イベント】乱数比較, "group condition (event): random comparison" | The game's own percentage roll. A group rolls `1:1(0, 0, 99)`, registers `1:140531(0, 0, N, 5)` and tests `f140001(0)`; the arguments are the slot, the dice slot (0 in every call), N and a comparison. Comparison 5 reads as `<=`, so the chance is N + 1 %. 23 calls in 20 groups: the Forlorn's invasion draw (N = 14, 15 %) and six other groups. Never run. [Randomness](randomness-in-scripts.md) | [data]; the reading of the arguments [inference] | E 23 |
| `1:140301(slot, flag, ...)` | 【グループ条件：フラグ】イベントフラグ比較, "group condition (flag): compare an event flag" | Registers an event-flag test into a group-condition slot. The Forlorn's hit-test group registers `1:140301(0, 109990, 1)` into the slot that holds its roll; a debug flag that forces a hit only works if the slot ORs its entries. Never run. [Randomness](randomness-in-scripts.md) | [data]; [inference] | not counted |
| `f4(min, max)` | ステート経過時間用の乱数値の取得, "get a random value for the state's elapsed time" | Talk scripts wait on it: `f3() > f4(0.1, 3)` is a random wait of 0.1 to 3 s. Never tried. [Randomness](randomness-in-scripts.md) | [data] | T 36 |
| `1:140511(slot, min, comparison, max)` | 【グループ条件：イベント】ステート経過時間比較, "group condition (event): compare the state's elapsed time" | The same `(min, max)` pair. Mostly `min == max`, a plain timer. The Forlorn's spawn wait passes a range (0 to 5 s, the Bastille 0 to 10): a random wait. Never tried. [Randomness](randomness-in-scripts.md) | [data]; [inference] | E 300 |
| `1:131705(...)` | 【ジェネレータ】初期開始位置を指定ポイントからランダムで選択して上書き, "pick the start position at random from the given points" | The Forlorn's random spawn point. Never tried. [Randomness](randomness-in-scripts.md) | [data] | E 14 |
| `1:131713(...)` | 【ジェネレータ】指定ジェネレータ内のキャラをランダム強制生成, "force-spawn a random character in the generator" | Not traced. [Randomness](randomness-in-scripts.md) | [data] | E 2 |
| `f140002(slot)` | 【グループ条件】ホスト調停した結果がグループ条件を満たす, "the host-arbitrated result meets the group condition" | The host-decided twin of `f140001`. Every random compare we read uses `f140001`, the local one. [Randomness](randomness-in-scripts.md) | [data] | E 77 |
| `1:132201(row)` | 【その他】ボス戦開始, "start a boss battle" | The argument is a boss-battle row (2010000 for the Giant Lord). The game's own region triggers call it; arriving at the centre of a fight-start region starts a fight. We never call it. Its partners in the originals: `132202` 【その他】ボス戦終了, "end the boss battle" (E 36), `132203` HPゲージ切り替え, "switch the HP gauge" (E 11), `132204` ボス戦キャンセル, "cancel the boss battle" (E 2) | [data]; the region [live B19] | E 38 |
| `1:132221` | 【その他】イベント血文字有効化, "enable an event bloodstain message" | No script uses it; the developer messages are read some other way | [untested] | 0 |
| `f132243(bonfire)`, `f132242()` | 【その他】篝火周回数取得 / 点火済み篝火数取得, "get the bonfire cycle count / the number of lit bonfires" | Counts only. **No named command lights a bonfire** (325 event and 324 talk names, one shared table, checked). The *commands* with these two numbers are different things: `1:132242` オートセーブ無効化, "disable autosave" (E 12), and `1:132243` プレイヤー操作禁止, "forbid player control" (E 5). [Bonfire lit state](../modding-world/bonfire-lit-state.md) | [data] | E 1; 0 |
| `f132251(npc, n)`, `f132252(..)`, `f132262(..)` | 【その他】NPC情報汎用値取得, "get a generic value from the NPC info"; NPC情報のキャラテキストID取得, "get the NPC info's character text id"; お墓ソウル量取得, "get the grave's soul amount" | Talk scripts pass an NPC's talk group id and a small index (0 to 3); a library counts a white phantom's summons. `f132252` and `f132262` fill the `%s` and `%d` of the grave's YES/NO text. Not used by us. (The command `1:132251` is something else: it starts the credits.) | [data] | 251: E 12 · T 42; 252: E 20; 262: E 60 |
| *no such command* | | No command refills Estus or repairs gear (`1:131421` upgrades the flask, T 1; `1:131401` consumes an item, E 29 · T 17). A player effect does it | [data] + [live B56] | |

### How the exe finds a command

The game runs a script command through a **chain of handlers**. Each handler is a tree of compares ending in a byte map and a jump table, then a default case that hands the command to the next handler. All three below are slot 1 of a vtable named in the exe's RTTI. [code, read 17 Nov 2026; image base 0x140000000, addresses are RVAs]

| Handler | RTTI class, handler address | Which ids it covers | Default goes to |
|---|---|---|---|
| A | `EventEzStateChrCtrl`, `+0x457C90` | 130408 has its own case (`+0x4583AB`). Otherwise `id - 130125` in 0 to 0xEC, byte map `+0x458B28`, 14 jump-table entries at `+0x458AF0`. Ids above 130408 go to `+0x458436` (not read) | B |
| B | `EventEzStateCtrl`, `+0x461F20` | up to 130303: `id - 130001` in 0 to 0x82. 130303 has its own case (`+0x462578`). 130304 to 130611: `id - 130307` in 0 to 0xCC, byte map `+0x46DC34`, 45 entries at `+0x46DB80`. 130611 has its own case. Above that: `+0x46386E` (not read) | C |
| C | `CommonEzStateCtrl`, `+0x4910B0` | ids 1 to 4 and 90 (the small ids; the dice command is id 1) | the stub at `+0xB1F370`, which reads the id and returns |

- The jump-table entries are 32-bit RVAs. The byte map says which entry an id uses. Entry 44 of B's table is the **default** (`+0x46DA81`, a call into C). Entry 36 (`+0x46DA8F`) is a **do nothing**: it jumps straight to the function's end. B itself does nothing with 130466, 130467 and 130511 (A's branch above 130408 was not read, so they may be handled there).
- In the range B's map covers (130307 to 130511), every named command that B sends to its default is implemented in A: 130311 to 130314, 130351 (reset the damage count), 130361 (change equipment) and 130408. So "lands in the default" never meant "not implemented".
- Event scripts must reach A [inference]: FromSoftware's event scripts use 130301, 130305, 130306 and 130408, and only A implements them. Which handler a *talk* script starts at was not read.
- A's four HP cases each fetch their one argument with a virtual call (an int or a float is accepted, anything else reads as 0), clamp it, and store it in the object at the handler's +0xD8: HP at +0x168 for 130311 and 130312, the minimum at +0x16C for 130313 and 130314, with the maximum at +0x170 as the upper limit. The two ratio forms scale the maximum. All four were read on 17 Nov. [code]
- B's own cases between 130307 and 130511: 130307 to 130310, 130317, 130331 to 130336, 130341 to 130343, 130381, 130382, 130407, 130431, 130432, 130441 to 130448, 130451 to 130455, 130461 to 130465, 130471, 130481, 130482, 130491 to 130494. [code]

## How to do it: check a command before you rely on it

1. **Get the name.** `edd.py` prints the commands, functions and state-machine names of an `.edd`. Event and talk tables are the same except one extra event command (140741).
2. **See how FromSoftware uses it.** Parse the original scripts with `esd.py` and collect the argument shapes. Copy one that exists. An argument shape no original uses is a risk: our event YES/NO form (kind 0 with 190) was one (B20c).
3. **Check the script type.** Look at the E and T columns. A command only the talk scripts use (`130461`) may not behave the same in an event script.
4. **Put it in a probe group** whose first state holds no commands (B9), one command per step, with a read-only watcher on the memory it should change. That is how B56 worked.
5. **If nothing happens, read the exe.** Find the handler for the script type (table above), compute `id - base`, read the byte map entry, and look at the case. Check the **next** handler before saying a command is not implemented.
6. **Write down what you could not isolate.** Several rows above say "not isolated": a command ran inside a group that worked, and nothing showed its own effect.

## Pitfalls and what failed

- **An event-script YES/NO is cancelled unless a button press led to it** (B20, B20b, B20c). The window opened, then closed at once with a result that was neither YES nor NO. Moving the question into the Herald's Talk worked (B20d). The `!= 1` wait copied in B20 was not the cause. [YES/NO windows](../modding-menus/yes-no-windows.md)
- **Commands in a group's first state did not take** (B9): the win flow ran but the removals did not. FromSoftware's groups have none there. Put work in the second state. [Event script groups](event-script-groups.md)
- **Same number, two meanings.** `f130312()` is the player's HP; `1:130312` is "set HP by ratio". `f130441` is a character-making window test; `1:130441` is the YES/NO window. `f132241` is the new-game-plus cycle; `1:132241` saves. `f132242` counts lit bonfires; `1:132242` disables autosave. Read the `.edd` name for the right table.
- **Object-state requests that restore did not take** (B25). The idols stayed destroyed and the pools stayed drained, whichever setting. Opening the Belfry gate by request worked (B19). [inference] A request that follows the object's own forward transition works; a request that sends it back does not. [Map object states](map-object-states.md)
- **A direct reallocation call only played the animation** (B30). The same two commands, run from the NPC menu's own Reallocate result, opened the screen (B31). [NPC menus](../modding-menus/npc-menus.md)
- **The N-choice list was refused silently** while the NPC menu was open (B36, B36b). Four closes before the list fixed it (B36c). [Adding dialogue options](../modding-menus/adding-dialogue-options.md)
- **A stored death count looped the player back into a fight** (B50, B50b). B50c's log shows the cause was the comparison, not the Aged Feather: the "count rose" marker came on one second after arriving, with no death. Why a stored copy compared as lower within a second is **not settled**. [Detecting a death](detecting-death-in-scripts.md)
- **"Not implemented" was too strong for 130311 to 130314.** The B56 write-up and the notes built on it read B's byte map, saw the default entry, and stopped. A has real cases. Nothing was called live, so "no script can't-die" really means "none tried".
- **"649 commands" is a double count.** An early note said it checked 649 commands for one that lights a bonfire. That is 325 event names plus 324 talk names: one table counted twice.

## Evidence

| Claim | How we know | Test or source |
|---|---|---|
| `1:132125` reloads the map and leaves the respawn alone | The whole 39-fight rush warped by script; Majula's record stayed | [live B38, B39] |
| `1:132141` writes `{running map, 1, location}` | Respawn record read in memory | [live B19, B38, B44] |
| A script group runs only with an EventParam row; its first state's commands do not take | Win flow ran, removals did not | [live B9] |
| `1:131721` removes an enemy so it never exists | No CharacterCtrl for the row; control row had 4 | [live B7] |
| Wait for `f140001` after `1:140402` stops a neighbour map's scripts acting | The rush went on past Najka after the fix | [live B39c] |
| Event YES/NO without a button press is cancelled | Debug flags: stale, opened, cancel | [live B20c] |
| Talk-script windows work; Talk and event scripts reload at the map's next load | Setup passed; same process, no restart | [live B20d] |
| `1:130334(110000010, 19, 4)` refills HP and Estus | Flask byte 2 to 4, HP to full | [live B56 step 2, B57 step 5] |
| `1:130303` stops all damage and hit reactions; a load resets it | HP flat through a fight; damage after the warp | [live B56 step 5] |
| `1:130381(8)` gives ×1.5 enemy damage and no HP change | Hits 398 to 597, 267 to 401 | [live B56 step 6] |
| `f130312() < 1` after HP above 0 is a working death test | Three damage deaths in two arenas | [live B50c] |
| `1:1` plus `f1` works inside conditions | Random picks, no repeats, both ends drawn | [live B52] |
| Object-state restore requests failed; the gate request worked | Idols, pools unchanged; gate opened | [live B25], [live B19] |
| A direct `1:130455` call fails; the NPC-menu route works | Animation only; screen opened | [live B30, B31] |
| No named command lights a bonfire or refills Estus | 325 + 324 names, one table | [data, 17 Nov] |
| 130311 to 130314 have cases in handler A | Disassembly of the cases | [code, 17 Nov] |
| The counts in the Orig. column | Walk over the 53 original scripts | [data, 17 Nov] |

## Tools

- `edd.py`, `esd.py`: read the developer names and parse a script into groups, states, conditions and commands. They live in the project's earlier research tools, not in `tools/`.
- `tools/esdwrite.py`: writes new groups and states; every new script must re-write byte for byte before it is installed.
- `tools/arenamod.py`, `tools/bossrush.py`, `tools/specials.py`, `tools/choices.py`, `tools/options.py`, `tools/fightlist.py`: build the groups that use the commands above.
- `tools/probe56.py`, `tools/b56.py`: the one-command-per-step probe (retired after B56).
- `tools/fltrace.py`: read-only trace of the front-end window state while a conversation runs.
- `tools/offline_tests.py`: checks the built scripts' command shapes before any game test.

## Open questions

- Do `1:130311` to `1:130314` work from an event script, and on whom? The target object is the handler's +0xD8. If it is the player, `1:130313(1)` would be a script "can't die" writing the field the cheat table writes. If it is null, the call may crash. [untested]
- What do the constant arguments mean: `19, 4` in `130334`; `0, 0` and the last argument of `132125`; `190` and `220` in the event windows; the arguments of `130408`; the second argument of `131721`?
- Does `1:130336` cancel an effect?
- Which handler do talk scripts start at, and what do A's cases above 130408 and B's above 130611 hold? Those branches were not read.
- Do `1:132121`, the group forms `131722` / `131732`, `130445` / `130446`, `131631` / `131632`, `132221`, `130131` and the area variables work for us? [untested]
- Does `1:132241` force a save, or would the autosave have kept the setup? [untested]
- Do script global variables really not survive a map load? Area variables were never tried.
- Does `1:140531` give a usable percentage roll? [untested]

## Related pages

- [Event script groups](event-script-groups.md): groups, EventParam rows, load-time groups
- [Warping the player by script](../modding-world/warping-by-script.md): `132125`, `132121`, arrival points
- [The respawn point](../modding-world/respawn-point.md): `132141` and the record
- [Event flags and safe ids](event-flags-and-free-ids.md): `130101`, flag ranges
- [Detecting a player death](detecting-death-in-scripts.md): `f130312`, `f130308`, variables
- [Restoring HP and Estus](../modding-menus/restoring-hp-and-estus.md): `130334`, `130303`, `130311` to `130314`
- [Difficulty levers](../modding-world/difficulty-levers.md): `130381`, `130317`
- [Removing enemies](../modding-world/removing-enemies.md): `131721`, `131731`
- [YES/NO windows](../modding-menus/yes-no-windows.md): `130441`, `130443`, key-guide areas
- [Adding dialogue options](../modding-menus/adding-dialogue-options.md): `130481`, the four closes
- [NPC menus](../modding-menus/npc-menus.md): `130461` to `130465`, `130455`, `132301`
- [Giving items](../modding-menus/giving-items.md): `131402`
- [Map object states](map-object-states.md): `131636`, `131638`
- [Bonfire lit state](../modding-world/bonfire-lit-state.md): why no command lights a bonfire
- [Randomness in scripts](randomness-in-scripts.md): `1:1`, `f1`, `140531`
- [Writing ESD files](writing-esd-files.md): how the mod writes these scripts
