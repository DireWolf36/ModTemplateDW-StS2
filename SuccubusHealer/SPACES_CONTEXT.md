# 📖 Copilot Spaces Context — Succubus Healer Mod

**Complete reference for any Copilot instance working on this project**

---

## 🎯 Project Summary

**Mod Name:** Succubus Healer  
**Game:** Slay the Spire 2 (Godot 4 .NET)  
**Status:** Code framework complete, files pending import  
**Scope:** 50 cards + 8 powers + 10 relics + custom character + custom energy type  
**Private:** Yes (development only)  
**Repository:** https://github.com/DireWolf36/ModTemplateDW-StS2  
**Branch:** `feature/succubus-healer-mod` (when available)

---

## 📝 What This Mod Does

**Succubus Healer** is a custom character for Slay the Spire 2 designed as a **multiplayer healer**:

- **HP:** 70 (fragile, needs resource management)
- **New Energy:** **Seed** (persistent orbs, like Regent's Stars)
- **New Currency:** **Fluid** (tracked via Power, used for healing)
- **Core Mechanics:**
  - **Consumption** — Drain enemies to gain Seed
  - **Expulsion** — Spend Seed for powerful effects
  - **Commitment** — Lock Seed away for N turns, generate Fluid/turn, return Seed when expired

- **Card Types:** 50 unique cards across 4 rarities
  - 4 Starter/Basic
  - 18 Common
  - 16 Uncommon
  - 12 Rare (many with Exhaust keyword for resurrection effects)

- **Relics:** 10 custom relics supporting Seed/Fluid generation and multiplication
- **Powers:** 8 core power effects (Regrowth, Life Link, Resurrection, etc.)
- **Multiplayer Focus:** Many cards heal allies, enable co-op playstyles

---

## 🔧 Project Structure

```
SuccubusHealer/
├── Source/                                  ← All C# source code
│   ├── SuccubusHealerInit.cs               ← Mod entry point [READY]
│   ├── Cards/                              ← 50 card files [PENDING IMPORT]
│   │   ├── SuccubusCard.cs                 ← Base card class [READY]
│   │   ├── Starter/, Common/, Uncommon/, Rare/
│   ├── Powers/                             ← 8 power files [PENDING IMPORT]
│   │   ├── SuccubusPower.cs                ← Base power class [READY]
│   ├── Relics/                             ← 10 relic files [PENDING IMPORT]
│   │   ├── SuccubusRelic.cs                ← Base relic class [READY]
│   ├── Character/                          ← Character & pools [PENDING IMPORT]
│   └── Energy/                             ← Seed energy type [PENDING IMPORT]
├── content/
│   ├── images/                             ← 93 PNGs (all placeholder) [PENDING IMPORT]
│   │   ├── cards/, powers/, relics/, character/, animations/
│   └── localization/eng/                   ← 5 JSON files (en only) [PENDING IMPORT]
├── .github/workflows/
│   └── build.yml                           ← GitHub Actions CI/CD [READY] ✅
├── SuccubusHealer.csproj                   ← .NET 9.0 project [READY]
├── SuccubusHealer.json                     ← Mod manifest [READY]
├── Directory.Build.props                   ← Godot/STS2 path config [READY]
├── project.godot                           ← Godot project [READY]
├── README.md                               ← Quick start [READY]
├── SETUP.md                                ← Build guide [READY]
├── SOURCE_ORGANIZATION.md                  ← File structure reference [READY]
├── THINK_CHAIN.md                          ← Complete dev history & API patterns [READY]
└── .gitignore                              ← C# & Godot exclusions [READY]
```

**[READY]** = File exists, configured, tested  
**[PENDING IMPORT]** = Awaiting user's AI-generated files

---

## 🏗️ File Statistics

| Category | Count | Status |
|----------|-------|--------|
| C# Source Files | 77 | Framework ready, code pending |
| PNG Images | 93 | All placeholders, needs real art |
| JSON Localization | 5 | Framework ready, translations pending |
| Config/Doc Files | 8 | Complete & configured |
| **Total** | **183** | **42% complete** |

---

## 🎓 Critical API Patterns

**These are the CORRECT patterns for STS2 modding with BaseLib.** Any code review/generation must follow these:

### ✅ Card Pattern (Correct)
```csharp
[CardLocalization(name: "CardName", description: "...")]
public sealed class MyCard() : SuccubusCard(cost, type, rarity, target)
{
    protected override IEnumerable<DynamicVar> CanonicalVars => [new DamageVar(6, ValueProp.Move)];
    
    protected override async Task OnPlay(PlayerChoiceContext choiceContext, CardPlay play)
    {
        await CommonActions.CardAttack(this, play).Execute(choiceContext);
    }
    
    protected override void OnUpgrade()
    {
        DynamicVars.Damage.UpgradeValueBy(3m); // decimal!
    }
}
```

**Key Points:**
- Primary constructor: `CustomCardModel(cost, type, rarity, target)` ✅
- **NOT** property overrides ❌
- `async Task OnPlay(...)` — **NOT** `IEnumerable<GameCommand>` ❌
- Use `CommonActions.*` for damage/block/heal ✅
- `DynamicVars.X.UpgradeValueBy(decimal)` for upgrades ✅
- Rarity: `CardRarity.Basic` for starters (NOT `Starter`) ✅

### ✅ Power Pattern (Correct)
```csharp
public sealed class MyPower : SuccubusPower
{
    public override PowerType Type => Buff;
    public override PowerStackType StackType => Counter;
    
    public override async Task BeforeHandDraw()
    {
        Flash();
        await CreatureCmd.Heal(Owner, Amount);
    }
}
```

**Key Points:**
- Inherit from `CustomPowerModel` via `SuccubusPower` ✅
- All hooks are `async Task` ✅
- Use `base.Owner`, `base.Amount`, `Flash()` ✅
- Icon path: `CustomPackedIconPath` (NOT `IconPath`) ✅

### ✅ Relic Pattern (Correct)
```csharp
[Pool(typeof(SuccubusHealerRelicPool))]
public sealed class MyRelic : SuccubusRelic
{
    public override RelicRarity Rarity => Common;
    
    public override async Task AfterCombatStart()
    {
        Flash();
    }
}
```

**Key Points:**
- `[Pool(typeof(...))]` attribute ✅
- Inherit from `CustomRelicModel` via `SuccubusRelic` ✅
- All hooks are `async Task` ✅
- Icon path: `PackedIconPath` (NOT `IconPath`) ✅

### ✅ Character Pattern (Correct)
```csharp
public class SuccubusHealerCharacter : PlaceholderCharacterModel
{
    public override int StartingHp => 70;
    public override IEnumerable<CardModel> StartingDeck => [ModelDb.Card<StrikeSuccubus>(), ...];
    public override CardPoolModel CardPool => ModelDb.CardPool<SuccubusHealerCardPool>();
}
```

**Key Points:**
- Inherit from `PlaceholderCharacterModel` ✅
- Use `ModelDb.Card<T>()` for card references ✅
- Use `ModelDb.CardPool<T>()` for pools ✅

### ✅ Init Pattern (Correct)
```csharp
[ModInitializer(nameof(ModLoaded))]
public partial class SuccubusHealerInit : Node
{
    public static void ModLoaded()
    {
        var assembly = System.Reflection.Assembly.GetExecutingAssembly();
        Godot.Bridge.ScriptManagerBridge.LookupScriptsInAssembly(assembly);
    }
}
```

**Key Points:**
- `[ModInitializer]` attribute ✅
- Call `ScriptManagerBridge.LookupScriptsInAssembly()` ✅
- **NO explicit registrations** — BaseLib auto-discovers ✅

---

## ❌ Common Mistakes (What NOT to Do)

| Mistake | Wrong | Right | Impact |
|---------|-------|-------|--------|
| Card lifecycle | `IEnumerable<GameCommand> OnPlay()` | `async Task OnPlay()` | **Compilation error** |
| Constructor | Property overrides | Primary constructor | **Runtime crash** |
| Damage logic | `yield return DamageCmd` | `await CommonActions.CardAttack()` | **Won't execute** |
| Rarity | `CardRarity.Starter` | `CardRarity.Basic` | **Won't display** |
| Upgrades | `BaseDamage += 3` | `DynamicVars.Damage.UpgradeValueBy(3m)` | **Wrong values in text** |
| Power base | `CustomPower` | `CustomPowerModel` | **Compilation error** |
| Icons | `IconPath = "..."` | `CustomPackedIconPath` property | **No icon in UI** |
| Registration | Manual `RegisterCharacter<T>()` | Just inherit + `LookupScriptsInAssembly()` | **Won't load** |

---

## 🚀 GitHub Actions CI/CD

**Automated build pipeline included:**

**File:** `.github/workflows/build.yml`

**Triggers:**
- Every push to `feature/*` branches
- Manual trigger via Actions tab

**Jobs:**
1. **Build Debug** — Full compilation with debug symbols
2. **Build Release** — Optimized build
3. **Upload Artifacts** — Compiled DLL ready for testing

**How to Use:**
1. Push code to `feature/succubus-healer-mod`
2. Go to: https://github.com/DireWolf36/ModTemplateDW-StS2/actions
3. Check build status
4. Download artifacts if successful
5. Test in game: `mods/SuccubusHealer/SuccubusHealer.dll`

---

## 📋 Workflow for Code Review & Iteration

### When User Uploads Files:

1. **Import** — Copy files into correct folders per `SOURCE_ORGANIZATION.md`
2. **Verify** — Check API patterns against this guide
3. **Flag Issues** — List any:
   - Wrong async/await patterns
   - Incorrect property names
   - Missing attributes
   - Rarity/enum mismatches
4. **Fix** — Batch corrections via automated tools
5. **Compile** — GitHub Actions builds automatically
6. **Debug** — Check logs for remaining errors
7. **Iterate** — Refactor, rebalance, add features

### Code Review Checklist:

- [ ] All cards use `async Task OnPlay()`?
- [ ] All powers/relics use `async Task` hooks?
- [ ] Card rarities are `CardRarity.Basic` (not `Starter`)?
- [ ] Upgrades use `UpgradeValueBy(3m)` decimal syntax?
- [ ] All base classes correct (`CustomCardModel`, `CustomPowerModel`, etc.)?
- [ ] Icon paths use extension methods (`.PowerImagePath()`, `.RelicImagePath()`)?
- [ ] `[Pool(typeof(...))]` attributes on relics/cards?
- [ ] Character uses `ModelDb.Card<T>()` for starting deck?
- [ ] Init uses only `LookupScriptsInAssembly()` (no manual registration)?
- [ ] Build compiles without errors?

---

## 📚 Reference Documents

All in `SuccubusHealer/`:

1. **`THINK_CHAIN.md`** — Complete development history, why decisions were made, all API patterns reference
2. **`README.md`** — Character overview, card/power/relic lists
3. **`SETUP.md`** — Build, install, troubleshooting (10+ sections)
4. **`SOURCE_ORGANIZATION.md`** — All 77+ files documented with purpose
5. **`SPACES_CONTEXT.md`** — This document

---

## 🔗 External References

**Required Knowledge:**
- BaseLib — https://github.com/Alchyr/BaseLib-StS2
- TheCursedMod2 (reference implementation) — https://github.com/jhp109/TheCursedMod2
- STS2 Modding Handbook — https://fresh-milkshake.github.io/Modding-Tutorial/
- ModSmith Framework — https://cpimhoff.github.io/Sts2-ModSmith/

**Dependencies:**
- .NET 9.0 SDK — https://dotnet.microsoft.com/en-us/download/dotnet/9.0
- Godot 4.5.1 Mono — https://godotengine.org/download/archive/

---

## 🎮 Testing & Deployment

### Local Testing:
1. Build locally: `dotnet build`
2. Copy `bin/Debug/net9.0/SuccubusHealer.dll` to:  
   `C:\Program Files (x86)\Steam\steamapps\common\Slay the Spire 2\mods\SuccubusHealer\`
3. Ensure `SuccubusHealer.json` is in same folder
4. Launch STS2, check mods menu

### GitHub Actions Testing:
1. Push to `feature/*`
2. Actions runs auto-build
3. Download artifact
4. Test as above

---

## ✅ Current Status

| Component | Status | Notes |
|-----------|--------|-------|
| Project structure | ✅ Complete | Ready for code import |
| Base classes | ✅ Ready | `SuccubusCard`, `SuccubusPower`, `SuccubusRelic` stubs exist |
| Entry point | ✅ Ready | `SuccubusHealerInit.cs` configured |
| Build config | ✅ Ready | `.csproj`, `Directory.Build.props`, Godot config |
| CI/CD pipeline | ✅ Ready | GitHub Actions auto-builds on push |
| Documentation | ✅ Complete | All guides & references written |
| Code files (77) | ⏳ Pending | Awaiting user's AI-generated files |
| Assets (93 PNGs) | ⏳ Pending | Placeholders in place, needs real art |
| Localization (5) | ⏳ Pending | Framework ready, content pending |

---

## 🎯 Next Steps

1. **User uploads AI-generated files** to `/SuccubusHealer/`
2. **Copilot reviews code** against API patterns
3. **GitHub Actions auto-builds** → Check for errors
4. **Fix compilation issues** → Iterate with user
5. **Test in-game** → Gameplay balance & bug fixes
6. **Add real art** → Replace placeholder PNGs
7. **Finalize & release** → Ready for Steam Workshop

---

**Created:** 2026-05-18  
**Repository:** https://github.com/DireWolf36/ModTemplateDW-StS2  
**For:** DireWolf36 via Copilot Spaces
