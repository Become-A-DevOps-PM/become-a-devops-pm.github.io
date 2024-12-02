
+++
title = "Vad är Persistens?"
type = "slide"
date = 2024-12-02
draft = false
hidden = true

theme = "sky"
[revealOptions]
controls= true
progress= true
history= true
center= true
+++

## Vad är Persistens?
- Ett systems förmåga att behålla data över tid.
- Viktigt för applikationer som kräver datalagring och hämtning.

---

## Typer av Persistens
1. **Volatil Lagring**:
   - Temporär lagring (t.ex. RAM).
   - Data förloras när strömmen stängs av.

2. **Icke-Volatil Lagring**:
   - Permanent lagring (t.ex. HDD, SSD).
   - Data finns kvar även efter strömavbrott.

---

## Persistens inom IT
- **Databaser**: Lagrar strukturerad data för enkel åtkomst och manipulation.
- **Filsystem**: Hanterar filer på disk för långvarig lagring.
- **Nyckel-Värde Lagring**: Ger snabb åtkomst till data med hjälp av nycklar (t.ex. Redis).

---

## Vikt av Persistens
- Säkerställer datakontinuitet och tillförlitlighet.
- Möjliggör återhämtning vid fel.
- Stöder affärsverksamhet och användarupplevelse.
