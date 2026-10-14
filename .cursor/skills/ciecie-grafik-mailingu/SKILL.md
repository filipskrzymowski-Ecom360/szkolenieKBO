---
name: ciecie-grafik-mailingu
description: Slices mailing mockup graphics (JPG/PNG/PDF/PSD/WEBP) into email-ready PNG/JPG assets with preview.html, ZIP, manifest, and palette. Use when the user uploads a file to do-ciecia/, asks to pociąć or ciąć grafiki, slice a mailing, or attaches an email mockup. Always launch a Task subagent with Claude Opus High instead of slicing in the parent agent.
---

# Cięcie grafik z mailingu

## Auto-start (obowiązkowe)

Gdy w workspace pojawi się makieta do cięcia, **nie tnij w agencie nadrzędnym**.

Pierwsze wywołanie narzędzia:

- Tool: `Task`
- `subagent_type`: `generalPurpose`
- `model`: `claude-opus-5-thinking-high`
- Wklej do promptu tę skill + bezwzględną ścieżkę pliku źródłowego + katalog wyjścia

Po powrocie subagenta: pokaż `preview.html`, listę plików, `ciecia.zip` i zastrzeżenia jakości. Nie twórz szablonu Klaviyo, chyba że user o to poprosi osobno.

## Foldery

| Rola | Ścieżka |
|---|---|
| Wrzuć makietę tutaj | `do-ciecia/` |
| Wynik cięcia | `ciecia/<stem-pliku-zrodlowego>/` |

Akceptowane źródła: `.jpg` `.jpeg` `.png` `.webp` `.gif` `.tif` `.tiff` `.psd` `.pdf` `.svg`

Jeśli user załączy plik spoza `do-ciecia/` (np. Downloads), skopiuj go do `do-ciecia/` i tnij z kopii.

## Kontrakt wyjścia

W `ciecia/<stem>/` muszą powstać:

1. Wycinki `NN-opis-elementu.png` (lub `.jpg` dla zdjęć) — numeracja z góry na dół
2. `preview.html` — galeria, wymiary, waga, `alt`, przycisk **Pobierz** przy każdej grafice
3. `ciecia.zip` — same wycinki, bez źródła
4. `manifest.json` — box `[x,y,w,h]`, format, alt, `not_cut`
5. `palette.json` — kolory tła zmierzone z pikseli (hex + rgb)

Subagent ma sam napisać i uruchomić skrypt Python (Pillow + numpy), potem **wizualnie odczytać każdy wycinek** narzędziem Read i poprawić kadry.

Nie tnij bloków tekstowych. CTA zapisuj tylko jako referencję (`NN-cta-referencja.png`, `reference_only: true`).

---

# Cięcie grafik z mailingu — instrukcja i specyfikacja

---

## 1. Jak wygląda proces cięcia

1. **Analiza makiety.** Skanuję plik piksel po pikselu i wykrywam pasy tła (biel, błękit, kolor marki) oraz miejsca, w których zmienia się sekcja. Dzięki temu wiem, gdzie kończy się jeden blok, a zaczyna następny.
2. **Wykrycie krawędzi elementu.** Dla każdego elementu (logo, zdjęcie, przycisk, falka) szukam ostrego przejścia kontrastu — to daje dokładny prostokąt, bez ucinania liter i bez zbędnego tła dookoła.
3. **Margines bezpieczeństwa.** Elementy typograficzne (logo, ikony) dostają ok. 18–20 px zapasu tła z każdej strony, żeby nic nie było przycięte. Zdjęcia i pełne pasy tnę dokładnie po krawędzi.
4. **Zapis 1:1.** Każdy wycinek zapisuję w oryginalnej rozdzielczości jako PNG (bez ponownej kompresji), z nazwą opisową i numerem zgodnym z kolejnością w mailu.
5. **Kontrola i wydanie.** Sprawdzam wycinki wizualnie, a potem dostajesz stronę z podglądem i przyciskiem „Pobierz" przy każdej grafice oraz całość w paczce ZIP.

---

## 2. Co dostajesz na wyjściu

| Typ elementu | Format | Uwagi |
|---|---|---|
| Logo, ikony, falki | PNG (najlepiej SVG) | Z zapasem tła; SVG jeśli istnieje wektor |
| Zdjęcia hero i produktowe | PNG lub JPG 85% | Cięte dokładnie po krawędzi kadru |
| Przyciski CTA | PNG | Lepiej odtworzyć w HTML — klikalność i dostępność |
| Pasy / sekcje tła | PNG | Pełna szerokość maila |
| Bloki tekstowe | **nie tnę** | Tekst powinien być żywym HTML-em, nie obrazkiem |

Nazewnictwo plików: `NN-opis-elementu.png`, np. `03-hero-basen.png`, `09-produkt-bestsellery.png`. Numeracja idzie z góry na dół maila.

---

## 3. Specyfikacja pliku źródłowego (co mi przekazujesz)

**Idealnie:**

- **PNG @2x** — szerokość 2× szerokości maila (mail 600 px → plik 1200 px; mail 1250 px → plik 2500 px). Bez kompresji stratnej.
- Profil kolorów **sRGB**, 8 bit.
- Albo od razu: link do **Figmy / Canvy** lub plik **PSD** z warstwami — wtedy eksportuję każdy element osobno, idealnie ostry.
- **Logo i ikony jako SVG** — skalują się bez straty, ważą kilka kB.
- Oryginalne zdjęcia produktowe i hero osobno, zanim trafią do makiety.

**Czego unikać:**

- JPG jako źródła do cięcia — kompresja zostawia artefakty wokół liter i krawędzi, których nie da się cofnąć.
- Zrzutów ekranu i plików z komunikatorów (WhatsApp, Messenger) — są przeskalowane i skompresowane.
- WebP o niskiej jakości.
- Plików w CMYK.

---

## 4. Specyfikacja grafik do maila (co wraca do wdrożenia)

| Parametr | Wartość |
|---|---|
| Szerokość maila | 600 px (eksport @2x = 1200 px) |
| Format zdjęć | JPG, jakość 80–85% |
| Format grafik z tekstem / płaskich | PNG-24 |
| Logo i ikony | SVG, fallback PNG @2x |
| Waga pojedynczej grafiki | do ~200 kB |
| Łączna waga maila | do ~1 MB |
| Przestrzeń barw | sRGB |
| Przezroczystość | tylko PNG; tła w mailach lepiej wypełnione kolorem |
| `alt` | każdy obrazek musi mieć opis tekstowy |
| Szerokość w HTML | zawsze `width` + `max-width: 100%` |

---

## 5. Dobre praktyki

- Tekst w mailu = **żywy HTML**, nie obrazek. Klienci pocztowi często blokują grafiki, a tekst na obrazku nie jest czytany przez czytniki ekranu ani nie da się go przeszukać.
- Kluczowy przekaz (rabat, kod, CTA) nie może istnieć wyłącznie na grafice.
- Dziel długie sekcje na kilka mniejszych plików — szybciej się ładują i łatwiej podmienić jeden fragment.
- Falki i pasy dekoracyjne: jeden kafelek + powtarzanie w tle zamiast jednej szerokiej grafiki.

## Prompt subagenta (szablon)

```
You are slicing graphics from an email mailing mockup for Klaviyo implementation.

WORK ONLY in this local repo. Do not create a Klaviyo template. Do not upload to Klaviyo.

Source: <ABS_PATH>
Copy into do-ciecia/ if it is not already there.
Output directory: <REPO>/ciecia/<STEM>/

Follow the skill at:
<REPO>/.cursor/skills/ciecie-grafik-mailingu/SKILL.md

Write and run a Python/Pillow script. Visually QA every slice with the Read tool. Produce preview.html, ciecia.zip, manifest.json, palette.json.

Return: output directory, each file (name, box, px, bytes, alt), not_cut list, quality caveats.
```
