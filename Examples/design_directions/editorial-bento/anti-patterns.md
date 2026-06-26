# Editorial Bento — anti-patterns

## ❌ Symmetrisk grid med like-store kort
```tsx
<div className="grid grid-cols-3 gap-4">
  {items.map(i => <Card key={i.id} />)}
</div>
```
Editorial Bento KREVER asymmetri. Hvis alle kort er like = ikke bento.

**RIKTIG:**
```tsx
<div className="grid grid-cols-12 grid-rows-6 gap-2">
  <Card className="col-span-7 row-span-4" featured />     {/* dominerende */}
  <Card className="col-span-5 row-span-2" />
  <Card className="col-span-3 row-span-4" />              {/* høyt smalt */}
  <Card className="col-span-2 row-span-2" />
  <Card className="col-span-5 row-span-2" />
</div>
```

## ❌ Bare sans-serif overalt
Editorial Bento HAR serif som display-font. Hvis hele siden er Inter eller Geist = ikke editorial — det er bare en grid med sans.

## ❌ Sentrert tekst overalt
Editorial = redaksjonell rytme. Tekst er venstre-justert som i en avis, ikke sentrert som i en presentasjon.

## ❌ Soft shadows på kort
Editorial bruker BORDERS for separasjon, ikke shadows. `shadow-lg` på kort = ikke editorial.

## ❌ Manglende hvit-rom mellom seksjoner
Editorial puster. Hvis sider er tett-pakket fra topp til bunn = ikke editorial — det er en landingsside.

**RIKTIG:** `py-32 md:py-48` mellom store seksjoner.

## ❌ Fargerike accent-knapper
Editorial er disiplinert. Én accent-farge brukes SJELDEN (kun for «featured»-merking), ikke som default-CTA-farge. CTA er ofte bare en tekst-lenke med underline.

## ❌ Ikoner istedenfor typografi
Hvis du erstatter «Les mer» med pil-ikon → ikke editorial. Editorial bruker ORD som visuelt anker.

## ❌ Mangler typografisk hierarki innen samme seksjon
Editorial = scale-kontrast. Hvis hero har H1 i text-4xl og body i text-base, mangler du den dramatiske kontrasten. Editorial-hero bør være text-7xl/8xl/9xl med body i text-base — minst 5x forhold.
