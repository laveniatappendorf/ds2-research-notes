# Seven weapon copies added after launch (2026-11-09)

!!! info "Status: From the data"
    Read straight from game files or compared between builds. What the files contain is checked; why they are that way (developer intent) is interpretation. See [status labels](../status-labels.md).

The author noticed a red Caestus and a golden Greataxe in the Boss Arena mod's "all weapons" kit. They belong to a block
of seven weapons, 5600000-5660000, that a post-launch patch added:

| copy | ordinary weapon | new model |
|---|---|---|
| 5600000 "Longsword" | 1220000 Longsword | WP_5600 (green-tinted blade) |
| 5610000 "Murakumo" | 5000000 Murakumo | WP_5610 |
| 5620000 "Blacksteel Katana" | 1730000 Blacksteel Katana | WP_5620 |
| 5630000 "Rapier" | 1500000 Rapier | WP_5630 (dark) |
| 5640000 "Greataxe" | 2210000 Greataxe | WP_5640 (gold) |
| 5650000 "Great Club" | 2620000 Great Club | WP_5650 |
| 5660000 "Caestus" | 3500000 Caestus | WP_5660 (red) |

Image: `patch-1.08-weapon-copies.png` (ordinary icon left, copy right).

- **Exact copies apart from the look.** Each WeaponParam row equals its original except for the item id and the model.
  The ItemParam row differs only in its icon, its sort position and the SpEffect field (-1 -> 0). The name and
  description are the original's word for word.
- **When.** Not in the Network Test, PS3 v1.00-1.03 or PC up to 1.03 (June 2014). First in PS3 **1.08**, the oldest
  later patch we have (1.04-1.07 can't be downloaded), and still in 1.09, 1.10, PS3 Scholar, vanilla PC 1.12 and PC
  Scholar.
- **Nothing gives them out.** In PC Scholar no param (item lots, shops, NPC loadouts, starting classes, server-event
  lots) and none of 597 event, talk-script and map files holds any of the seven ids. They can only be had by editing
  the params or the save, which is why an "every weapon" list picks them up.
- The community wiki lists three of them (Caestus, Greataxe, Rapier) as "Unused ... reskin" (`wiki-unused-content-crossref.md`).

## What they were for: the Majula server chest

The author's information: they were a time-limited bonus, handed out through a chest in Majula. The data fits:

- **The chest's lot is a live-event slot.** The server-event item-lot table (ItemLotParam2_SvrEvent) has one chest
  row, **10045500**. Its developer name in the pre-release PC params is 【サーバー宝箱】 "server treasure chest", and 1004
  is Majula's map number (m10_04). Every other row in the table is a covenant or co-op reward labelled 運営イベント用,
  "for live events".
- **Shipped contents: one Torch.** In every build we have (PS3 v1.00, 1.08, 1.10, vanilla PC from the pre-release to
  1.12, PC Scholar), row 10045500 gives one Torch (60420000). The Network Test has no such table.
- **No shipped regulation ever lists the seven** (PS3 1.08, 1.09, 1.10, vanilla PC 1.12, PS3 Scholar, PC Scholar: no
  param row holds their ids). So the bonus must have come from a regulation pushed by the server, which swapped
  the chest's lot for a while and was later withdrawn. That's why the weapons sit in the files with nothing giving
  them out.

Not established: when the bonus ran, which weapon each player got and how the server chose, and whether the event
weapons are the only time that slot was changed. None of that is in shipped data.

## The community record (online search, 2026-11-16)

Fan wikis and 2014-15 forum posts (sources below) answer most of that. They agree with each other; no official
FromSoftware / Bandai Namco page was found (the Japanese fan pages cite the official site but don't link it).

- **The chest:** the metal chest in the Majula Mansion basement (House Key, Cale's room, stairs down). It normally
  holds one **Soul Vessel**. Event items appeared only when playing online, once per account per event; offline
  players got the Soul Vessel. Wikidot notes event items could arrive by update without the version number on the
  title menu changing, which fits a server-pushed regulation.
- **2014 "thank-you" campaign** (JP ダークソウル２感謝アイテム配布キャンペーン): weekly items from 5 June 2014, ending
  22 July 2014 (Crown of the Sunken King launch) with a "secret item", the **blue-tinted Murakumo (22-31 July 2014)**.
- **Blacksteel Katana copy:** 26 August - 7 September 2014 (Old Iron King DLC).
- **Longsword copy (blue blade):** 30 September - 13 October 2014 (Ivory King DLC).
- **5-12 February 2015:** all four again (Murakumo, Blacksteel Katana, Longsword) plus the **Great Club copy** (first
  and only time), ahead of Scholar. A Japanese forum post mentions another all-four notice for 25 November - 5
  December (year not given); unconfirmed.
- **The red Caestus, gold Greataxe and black Rapier were never handed out.** Every source lists them as unreleased;
  2014 threads only show them from datamined images. So the wiki's "Unused" label for those three is right, and the
  answer given on 16 Nov ("very likely the giveaway weapons") needs this correction: they were made for it, but
  only four of the seven were used.
- **Patch:** wikidot says the files came with "Patch 1.06"; our data has them first in PS3 1.08, the Crown of the
  Sunken King patch (PC 1.04, 18 July 2014). The first weapon event began on that DLC's launch day.

Still not established from data: how the chest switches between its Soul Vessel lot and the server-event lot
10045500 (whose shipped default is a Torch).

Sources: darksouls2.wiki.gg/wiki/Weapon_Reskins; darksouls2.wikidot.com/weapon-reskins and /majula-mansion;
darksouls2.wiki.fextralife.com/Events; Steam DS2 discussions 35219681524692938 (31 July 2014) and 38596747648310706
(23-24 July 2014); game9.blog.jp/archives/1004031644.html (JP campaign schedule). The Nexus pages (mod 1093
"Restore Timed-Exclusive Content", article 94) returned 403 and weren't read. tcrf.net's page returned text aimed at
AI tools instead of content and wasn't used.
