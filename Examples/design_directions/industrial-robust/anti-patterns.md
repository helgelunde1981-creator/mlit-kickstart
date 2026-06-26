# Industrial / Robust — anti-patterns AI MÅ unngå

Konkrete fail-mønstre observert i Container Consult AS (mai 2026). Hver bullet er noe Claude vision-evaluator vil score lavt på V3 (direction-fidelitet).

## ❌ Hero med mørk overlay som gjemmer bildet
```tsx
// FORBUDT — bildet underordnes teksten
<Image src={bg} className="opacity-55" />
<div className="absolute inset-0 bg-gradient-to-t from-black/80 via-black/40 to-black/30" />
```
**Hvorfor fail:** `opacity-55` gjør bildet matt. Full mørk gradient = bildet er bare «bakgrunns-mood», ikke ankeret. Claude scorer V1=4-5.

**RIKTIG:**
```tsx
<Image src={bg} className="object-cover" />  {/* full opacity */}
<div className="bg-gradient-to-r from-black/85 to-transparent" />  {/* kun venstre */}
```
Industrial: bildet ER produktet. Vis det.

## ❌ Inter_Tight / Geist / soft-tech-sans
```tsx
// FORBUDT — leser som SaaS-landingsside, ikke industri
<h1 className="font-display"> // = Inter_Tight extrabold
```
**RIKTIG:** Barlow Condensed Black, Bebas Neue, Anton, Oswald Heavy. Kondenserte tunge sans-fonts. Uppercase + font-black.

## ❌ Rounded-lg shadow-lg CTA-knapper
```tsx
// FORBUDT — default shadcn, ikke industri
<button className="rounded-lg bg-primary px-6 py-3 shadow-lg">
```
**RIKTIG:** Hard-shadow uten rounding:
```tsx
<button className="bg-primary px-8 py-4 shadow-[8px_8px_0_rgba(0,0,0,0.4)]">
```

## ❌ Uniform 3-kolonne grid med like-store kort
```tsx
// FORBUDT — flat hierarki, V2=3
<div className="grid grid-cols-3 gap-4">
  {services.map(s => <ServiceCard key={s.id} service={s} />)}
</div>
```
**RIKTIG:** Bento med ett dominerende kort:
```tsx
<div className="grid grid-cols-1 md:grid-cols-4 md:grid-rows-3 gap-4">
  <ServiceCard featured className="md:col-span-2 md:row-span-2" />
  <ServiceCard className="md:col-span-2" />
  {/* ... */}
</div>
```

## ❌ Sentrert hero med padding-symmetri
```tsx
// FORBUDT — leser som "presentasjon", ikke "industri-arbeid"
<div className="mx-auto max-w-4xl text-center py-32">
  <h1>{headline}</h1>
</div>
```
**RIKTIG:** Bunn-justert, venstre-aligned, MASSIV skala:
```tsx
<div className="flex min-h-[80vh] flex-col justify-end px-6 pb-20">
  <h1 className="text-7xl uppercase">{headline}</h1>
</div>
```

## ❌ Soft beige / krem bakgrunner
```css
/* FORBUDT — leser som "wellness", ikke industri */
--color-bg-primary: oklch(96% 0.01 80);  /* gulgrå krem */
```
**RIKTIG:** Industri-grå (nesten-hvit eller ren-grå, ingen varmt undertone):
```css
--color-bg-primary: oklch(98% 0 0);     /* nesten-hvit */
--color-bg-tertiary: oklch(20% 0 0);    /* mørk asfalt for industri-seksjoner */
```

## ❌ Generic Tailwind grays uten brand-tokens
```tsx
// FORBUDT — bryter K4 anti-template-grep
<div className="bg-slate-100 text-slate-900 border-slate-200">
```
**RIKTIG:**
```tsx
<div className="bg-bg-primary text-text-primary border-border">
```

## ❌ Manglende industri-signaler
Industrial-direction LEVER av småsignaler som ALDRI ses i SaaS-templates:
- Gul/sort sikkerhetsstriper på seksjon-overganger
- Mono-kicker med uppercase + letter-spacing
- Numeriske aksenter (50 år, 600 containere, 24t levering) i extra-stor display-font
- Tunge fotografi (ikke illustrasjoner, ikke ikoner alene)

Hvis hero ikke har MINST 2 av disse → V3 fail.

## ❌ Direction inkonsistens mellom hovedside og undersider
Forsiden ser Industrial ut, men `/kontakt` blir til standard skjema-side med rounded-lg + soft shadows. Claude vision vurderer V4 (side-variasjon) — det er IKKE «variasjon» det er «inkonsistens».

**RIKTIG:** Hver side beholder samme Industrial-signaler. Variasjon kommer fra LAYOUT, ikke fra at man dropper direction.

---

## Sjekk-script: V3 scorer dette automatisk

`scripts/check-visual-quality.ts` med `DESIGN_STYLE="Industrial / Robust"` ber Claude vision sammenligne screenshot mot disse karakteristikkene. Hvis sider har mørk overlay, rounded-lg, generic grays, eller Inter_Tight → V3 dropper til 2-4.
