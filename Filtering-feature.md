A quick log for shit and giggles: technical debt diff from the new filtering feature

-  collect filtering widgets #11066 : +7631 / -2113
-  Filename filter speedup #11550 : +65 / -85
-  collect: fix range selection and click #11636 : +55 / -57
-  range : allow shift-click to select a range #11665 : + 7 / -0
-  filtering : add grouping filter #11721 : + 817 / -29
-  range : reset bounds correctly when typing in entries #11807 : +7 / -2
-  filters : keep off/remove visible but dimmed if pinned #11811 : +8 / -9
-  filtering : fix an issue with conf value overwritting #11819 : +17 / -5
-  filters : update rating numbers correctly #11809 : +57 / - 37
-  filters : always show the pin icon #11808 : +44 / -53
  ----

Total: + 8708 / -2390

---
---

Same for the phweyland's version:

- Add text & color label filters #10694 : +502 / -87
- text filter: don't add wildcards on empty entry #11264 : +13 / -10
- Text filter: fix bug on query #11267 : +6 / -7
- color label filter: inverts icons symbols #11293 : +4 / -8
- filter (text & colors label): save their state between dt sessions #11303 : +134 / -94
  ---

Total: +659 / -206