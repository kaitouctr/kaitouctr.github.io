+++
title = 'A Quick Bit of Information on Twilight Menu RNG...'
date = 2024-11-02
draft = false
+++

I was bored so I just decided to check out the situation on Twilight Menu, and then post my findings here. 

It seems like the newer versions of nds-bootstrap (and probably Twilight Menu as well) seems to do a better job of stabilising the Timer0 and making the RNG more consistent (from my sample size of ~13, it looks like I am hitting around two Timer0s which is a major improvement from the old situation of ~10 Timer0s and 2 VCounts). The new Wood UI theme also seems to boot games faster than the rest but maybe my timing is just really bad.

I also tried out calibrating from a different version of nds-bootstrap and unfortunately I also discovered that if nds-bootstrap is updated (which is usually done via updating Twilight Menu as those two are bundled together), the DS parameters completely change with the Timer0, VCount, and VFrame being significantly different between the nds-bootstrap versions.

Below is the versions of Twilight Menu and nds-bootstrap for reference:
 - Twilight Menu v27.11.0
 - nds-bootstrap v2.1.0

Below is the settings I have set in Twilight Menu:
DS mode
ARM9 CPU Speed: 67mhz (NTR)
VRAM Mode: DS mode
Card Read DMA: On
DSi Splash Auto-skip: On
TWLMenu++ Splash Screen: Hide