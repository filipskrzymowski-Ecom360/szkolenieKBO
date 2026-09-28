# Instrukcja dla Claude — strona gotowa do wgrania jako szablon Elementora

Ten plik wklejasz do Claude razem z briefem strony (teksty, kolory, fonty, układ, linki, zdjęcia). Claude ma oddać **jeden plik `.json`**, który WordPress zaimportuje jako szablon Elementora i który da się dalej edytować w panelu — bez grzebania w HTML.

Działa na **Elementorze 3.6+ i 4.x** (kontenery Flexbox). Nie wymaga Elementor Pro.

---

## Jak tego użyć

1. Otwórz nowy czat w Claude.
2. Wklej **cały ten plik**.
3. Pod spodem dopisz brief: co to za strona, sekcje, teksty, kolory, fonty, adresy linków, zdjęcia.
4. Poproś o plik `nazwa-strony.json`.
5. Wgraj go w Elementorze według sekcji [Jak wgrać szablon](#jak-wgrać-szablon-w-wordpressie) na końcu.

Jeśli brief jest niekompletny, Claude ma dopytać. Nie ma zgadywać marki, tekstów ani oferty.

---

## Zadanie

Zbuduj stronę jako **natywny szablon Elementora** (kontenery Flexbox + zwykłe widgety). Wynik to jeden poprawny plik JSON do importu.

Szablon zawiera **tylko treść strony**. Nie dodawaj nagłówka ani stopki witryny — te biorą się z motywu albo Theme Buildera.

Bierz kolory, fonty, teksty, linki i zdjęcia **wyłącznie z briefu**. Brakuje treści → zapytaj. Nie wypełniaj lorem ipsum. Placeholder wolno wstawić tylko gdy użytkownik wyraźnie o to poprosi, i wtedy oznacz go `[DO UZUPEŁNIENIA]`.

---

## Plik wynikowy

- Nazwa: `nazwa-strony.json` (małe litery, myślniki, bez spacji).
- Czysty JSON. Zero komentarzy, zero końcowych przecinków, zero markdowna wewnątrz pliku.
- Kodowanie UTF-8. Polskie znaki zostaw jako znaki, nie jako `\uXXXX`, o ile narzędzie na to pozwala. JSON i tak musi być poprawny.
- Ładnie sformatowany (wcięcia 2 spacje), żeby dało się go czytać.

Korzeń pliku:

```json
{
  "content": [],
  "page_settings": {},
  "version": "0.4",
  "title": "Nazwa szablonu",
  "type": "page"
}
```

`content` to tablica kontenerów najwyższego poziomu (`isInner: false`). `version` zostaw `"0.4"` — to wersja formatu Elementora, nie numer wtyczki.

Każdy element ma unikalne `id`: 8 znaków hex (`a-f`, `0-9`), bez powtórzeń w całym pliku.

Kontener:

```json
{
  "id": "a1b2c3d4",
  "elType": "container",
  "isInner": false,
  "settings": {},
  "elements": []
}
```

Widget (zawsze pusty `elements`):

```json
{
  "id": "e5f6a7b8",
  "elType": "widget",
  "widgetType": "heading",
  "settings": {},
  "elements": []
}
```

W Navigatorze Elementora sekcje nazywaj przez `_title` w `settings` kontenera, np. `"_title": "Hero"`.

---

## Zasady layoutu

1. Tylko `elType: "container"` i `elType: "widget"`. Zero klasycznych sekcji (`section`) i kolumn (`column`).
2. Jedno drzewo na wszystkie urządzenia. Wartości desktop bez sufiksu, potem te same kontrolki z `_tablet` i `_mobile`. Nie rób drugiej, ukrytej kopii sekcji „na mobile”.
3. Składanie kolumn: rodzic zostaje `flex_direction: "row"` i `flex_wrap: "wrap"`. Dzieci dostają `width`, `width_tablet`, `width_mobile`. Nie przełączaj wiersza na `column`, żeby go złożyć.
4. Procenty, które sumują się do 100, **nie zawiną się same**. `33 + 33 + 33` zawsze mieści się w rzędzie. Szerokość tabletu i telefonu trzeba ustawić jawnie.
5. Styl (kolor, font, padding, tło, obramowanie) siedzi w `settings` widgetu. Nie buduj layoutu widgetem HTML, własnym CSS ani shortcode’em.
6. `custom_css` jest w Pro i łatwo psuje import u osób bez Pro. Nie używaj go.
7. Do klas w CSS używaj `css_classes`. `css_id` w Elementorze 4 często nie ląduje w HTML jako `id`. Kotwice w stronie rób widgetem `menu-anchor`.

### Breakpointy (tylko te)

| Urządzenie | Szerokość | Klucz w JSON |
|---|---|---|
| Desktop | ≥ 1025 px | `padding`, `width`, `typography_font_size` |
| Tablet | ≤ 1024 px | `padding_tablet`, `width_tablet`, `typography_font_size_tablet` |
| Mobile | ≤ 767 px | `padding_mobile`, `width_mobile`, `typography_font_size_mobile` |

Nie zakładaj dodatkowych breakpointów (laptop, widescreen, mobile extra).

### Rytm, gdy brief nie podaje liczb

| | Desktop | Tablet | Mobile |
|---|---|---|---|
| Sekcja, góra / dół | 80–120 px | 64–80 px | 48–56 px |
| Sekcja, boki | 32 px | 32 px | 16 px |
| Szerokość treści | boxed ~1140 px | 100% | 100% |
| Odstęp w kolumnie widgetów | 24–32 px | 20 px | 16 px |
| Gap pary przycisków | 12 px | 12 px | 12 px |

Na telefonie boki nie schodzą poniżej 16 px.

Dwa równoważne zapisy sekcji:

- padding na kontenerze zewnętrznym: boki i pion razem;
- zewnętrzny kontener `content_width: "full"` tylko z pionowym paddingiem, a w środku kontener `content_width: "boxed"` z bokami.

Hero i pas zdjęć na pełną szerokość: `content_width: "full"`, bez wąskiego boxa.

### Kolumny

| Układ | Desktop | Tablet | Mobile |
|---|---|---|---|
| Tekst + zdjęcie | 50/50 albo proporcja z briefu | 100% + 100% | 100% |
| Dwie równe karty | 50% / 50% | 50% / 50% | 100% |
| Trzy kolumny | 33% | 50% | 100% |
| Cztery kolumny | 25% | 50% | 100% |

Szerokość dziecka:

```json
"width": { "unit": "%", "size": 50, "sizes": [] },
"width_tablet": { "unit": "%", "size": 100, "sizes": [] },
"width_mobile": { "unit": "%", "size": 100, "sizes": [] }
```

Gap wiersza 50/50 ustaw na `0`, jeśli kolumny mają się stykać. Gap przycisków zostaw 12 px. Nie zeruj go.

Wiersz, który na desktopie ma zostać w jednej linii (hero 50/50): `flex_wrap: "nowrap"`, a kolumny i tak mają `width_tablet` / `width_mobile` 100%.

---

## Kształty wartości

Liczba (font, szerokość, min-height):

```json
{ "unit": "px", "size": 36, "sizes": [] }
```

Padding / margin / border-width / radius:

```json
{ "unit": "px", "top": "80", "right": "32", "bottom": "80", "left": "32", "isLinked": false }
```

Gap kontenera (`flex_gap`):

```json
{ "column": "24", "row": "24", "isLinked": true, "unit": "px", "size": 24 }
```

`column` to odstęp w poziomie, `row` w pionie.

Link:

```json
{ "url": "https://example.com", "is_external": true, "nofollow": false }
```

Link wewnętrzny (`/kontakt`, `#cennik`): `is_external: false`.

Tło kolorem:

```json
"background_background": "classic",
"background_color": "#111111"
```

Tło zdjęciem:

```json
"background_background": "classic",
"background_image": {
  "url": "https://…",
  "id": "",
  "size": "",
  "alt": "",
  "source": "url"
},
"background_position": "center center",
"background_size": "cover",
"background_repeat": "no-repeat"
```

---

## Widgety, których wolno użyć

Zostań przy widgetach z darmowego Elementora. Widgety Pro (formularz, nav menu, loop, slajdy, WooCommerce, posts) wstawiaj tylko wtedy, gdy użytkownik napisze, że ma Elementor Pro — i nazwij je w podsumowaniu.

### Nagłówek — `heading`

```json
{
  "title": "Tytuł sekcji",
  "header_size": "h2",
  "align": "left",
  "title_color": "#111111",
  "typography_typography": "custom",
  "typography_font_family": "Nazwa fontu z briefu",
  "typography_font_size": { "unit": "px", "size": 40, "sizes": [] },
  "typography_font_size_tablet": { "unit": "px", "size": 32, "sizes": [] },
  "typography_font_size_mobile": { "unit": "px", "size": 28, "sizes": [] },
  "typography_font_weight": "400",
  "typography_line_height": { "unit": "em", "size": 1.15, "sizes": [] }
}
```

`header_size`: `h1`–`h6`, `div`, `span`, `p`. Jeden `h1` na stronę. Kursywę i łamanie linii w tytule rób przez `<em>` i `<br>` w polu `title`.

### Tekst — `text-editor`

```json
{
  "editor": "<p>Akapit z briefu.</p>",
  "align": "left",
  "text_color": "#444444",
  "typography_typography": "custom",
  "typography_font_family": "Nazwa fontu z briefu",
  "typography_font_size": { "unit": "px", "size": 16, "sizes": [] },
  "typography_font_weight": "400",
  "typography_line_height": { "unit": "em", "size": 1.6, "sizes": [] }
}
```

W `editor` tylko proste znaczniki: `p`, `br`, `strong`, `em`, `a`, `ul`, `ol`, `li`. Bez `<style>`, bez `<script>`, bez layoutowego HTML.

### Przycisk — `button`

Elementor 4 **ignoruje `button_padding`**. Używaj `text_padding`.

Para przycisków obok siebie: oba mają `size: "md"`. Bez tego drugi potrafi zjechać do `sm` i mają różną wysokość.

```json
{
  "text": "Umów rozmowę",
  "link": { "url": "/kontakt", "is_external": false, "nofollow": false },
  "align": "left",
  "size": "md",
  "background_color": "#111111",
  "button_text_color": "#FFFFFF",
  "border_border": "none",
  "border_radius": { "unit": "px", "top": "0", "right": "0", "bottom": "0", "left": "0", "isLinked": true },
  "text_padding": { "unit": "px", "top": "16", "right": "32", "bottom": "16", "left": "32", "isLinked": false },
  "typography_typography": "custom",
  "typography_font_family": "Nazwa fontu z briefu",
  "typography_font_size": { "unit": "px", "size": 14, "sizes": [] },
  "typography_font_weight": "500"
}
```

Przycisk outline: `background_color: "transparent"`, `border_border: "solid"`, `border_width` 1 px, `border_color` z briefu.

Parę włóż w kontener: `flex_direction: "row"`, `flex_align_items: "center"`, `flex_wrap: "wrap"`, `flex_gap` 12 px. Nie dawaj marginu na samych przyciskach.

### Obraz — `image`

```json
{
  "image": {
    "url": "https://adres-z-briefu/zdjecie.jpg",
    "id": "",
    "size": "",
    "alt": "Konkretny opis zdjęcia",
    "source": "url"
  },
  "image_size": "full",
  "align": "center",
  "width": { "unit": "%", "size": 100, "sizes": [] }
}
```

`id` zostaw puste — plik nie jest jeszcze w Bibliotece mediów. W podsumowaniu wypisz, które obrazki użytkownik ma podmienić po imporcie.

Nie ma URL-a w briefie i użytkownik nie kazał stosować placeholderów → zapytaj. Nie wstawiaj losowych zdjęć stockowych jako „finalnych”.

### Kotwica — `menu-anchor`

```json
{ "anchor": "cennik" }
```

Id bez `#`. Link do niej: `"url": "#cennik"`.

### Lista ikon — `icon-list`

```json
{
  "icon_list": [
    {
      "_id": "i1a2b3c4",
      "text": "Punkt z briefu",
      "selected_icon": { "value": "fas fa-check", "library": "fa-solid" },
      "link": { "url": "", "is_external": false, "nofollow": false }
    }
  ],
  "icon_color": "#111111",
  "text_color": "#444444"
}
```

Ikony tylko z Font Awesome wbudowanego w Elementora (`fas`, `far`). Każdy punkt ma własne `_id`.

### Akordeon — `accordion`

```json
{
  "tabs": [
    { "_id": "q1a2b3c4", "tab_title": "Pytanie", "tab_content": "<p>Odpowiedź.</p>" }
  ]
}
```

### Odstęp i linia

- `spacer` → `"space": { "unit": "px", "size": 24, "sizes": [] }`
- `divider` → `color`, `weight`, `gap`

### Czego nie używać jako layoutu

`html`, `shortcode`, własne `<section>` w tekście, CSS grid, media queries. Widget `html` tylko gdy brief ma osadzenie, którego Elementor nie ma (np. mapę spoza widgetu), i wtedy tylko ten jeden fragment — nie całą sekcję.

---

## Typografia i kolor

- Fonty bierz z briefu. Nazwa w `typography_font_family` musi być dokładna (np. `Montserrat`, `Playfair Display`) — Elementor dociąga Google Fonts po tej nazwie. Font zainstalowany tylko na komputerze użytkownika na stronie się nie pojawi.
- Brief nie podaje fontu → zapytaj. Nie podstawiaj „ładnego” fontu po cichu.
- Kolory hex z briefu, zawsze z `#`. Tekst na ciemnym tle dawaj pełnym, jasnym kolorem z briefu. Nie zmniejszaj czytelności przez przezroczysty kolor tekstu. Półprzezroczystość zostaw na ramkach i dekoracji.
- Skala: jeden wyraźny H1, H2 mniejsze, tekst body czytelny (zwykle 16 px, line-height około 1.5–1.7). Eyebrow / etykieta może być mniejsza i z letter-spacingiem — tylko jeśli brief tak ma.
- Rozmiary nagłówków podaj na desktop, tablet i mobile. Drobny tekst (etykieta, przycisk) może zostać w jednym rozmiarze, jeśli na telefonie nadal da się go czytać.

---

## Szkielet sekcji

Wzorzec, od którego wychodzisz. Kolory i teksty podmień na brief.

```json
{
  "id": "aa11bb22",
  "elType": "container",
  "isInner": false,
  "settings": {
    "_title": "O nas",
    "content_width": "full",
    "flex_direction": "column",
    "background_background": "classic",
    "background_color": "#FFFFFF",
    "padding": { "unit": "px", "top": "96", "right": "0", "bottom": "96", "left": "0", "isLinked": false },
    "padding_tablet": { "unit": "px", "top": "72", "right": "0", "bottom": "72", "left": "0", "isLinked": false },
    "padding_mobile": { "unit": "px", "top": "48", "right": "0", "bottom": "48", "left": "0", "isLinked": false }
  },
  "elements": [
    {
      "id": "cc33dd44",
      "elType": "container",
      "isInner": true,
      "settings": {
        "content_width": "boxed",
        "boxed_width": { "unit": "px", "size": 1140, "sizes": [] },
        "flex_direction": "row",
        "flex_wrap": "wrap",
        "flex_align_items": "center",
        "flex_gap": { "column": "0", "row": "32", "isLinked": false, "unit": "px", "size": 0 },
        "padding": { "unit": "px", "top": "0", "right": "32", "bottom": "0", "left": "32", "isLinked": false },
        "padding_mobile": { "unit": "px", "top": "0", "right": "16", "bottom": "0", "left": "16", "isLinked": false }
      },
      "elements": []
    }
  ]
}
```

---

## Checklista przed oddaniem pliku

Sprawdź i popraw, zanim oddasz JSON:

1. Plik parsuje się jako JSON. Korzeń ma `content`, `page_settings`, `version: "0.4"`, `title`, `type: "page"`.
2. Każde `id` i każde `_id` jest unikalne i ma 8 znaków hex.
3. Nie ma `elType` `section` ani `column`. Widgety mają `"elements": []`.
4. Kontenery zewnętrzne mają `isInner: false`, zagnieżdżone `isInner: true`.
5. Każdy wiersz wielokolumnowy ma `width_tablet` albo `width_mobile`, więc składa się na węższym ekranie.
6. Przyciski używają `text_padding` i `size: "md"`. Nie ma klucza `button_padding`.
7. Teksty, kolory i fonty pochodzą z briefu. Nie ma wymyślonej oferty.
8. Obrazy mają `source: "url"`, puste `id` i sensowny `alt`.
9. Nie ma `<script>`, `<style>`, widgetu HTML jako layoutu ani `custom_css`.
10. Nagłówek i stopka witryny nie są w szablonie.

W odpowiedzi daj:

- plik `.json` do pobrania,
- listę sekcji po `_title`,
- listę obrazków do podmiany w Bibliotece mediów,
- czego z briefu brakowało i co zostało dopytane albo pominięte.

---

## Jak wgrać szablon w WordPressie

1. Zainstalowany i aktywny Elementor. W **Elementor → Ustawienia → Funkcje** kontenery Flexbox są włączone (w 4.x są domyślne).
2. Wejdź w edycję strony: **Strony → Dodaj nową → Edytuj w Elementorze**. Szablon strony zostaw taki, który pokazuje nagłówek i stopkę motywu. Canvas wybierz tylko wtedy, gdy strona ma być zupełnie goła.
3. W panelu Elementora kliknij ikonę folderu, potem **Importuj szablon**, i wskaż plik `.json`.
4. Ten sam plik można wgrać z **Szablony → Zapisane szablony → Importuj szablony**.
5. Po imporcie: folder → **Moje szablony** → **Wstaw**.
6. Podmień obrazki: kliknij widget obrazu albo tło kontenera i wybierz plik z Biblioteki mediów. Zewnętrzny URL po imporcie tylko podgląda zdjęcie.
7. Przejdź podgląd responsywny w Elementorze: ikona monitora, tabletu i telefonu. Popraw tylko to, co brief miał inaczej — struktura jest już jedna.

Gdy import zgłosi błąd pliku, prawie zawsze winny jest JSON (przecinek, komentarz, ucięty nawias) albo nieznany `widgetType`. Wklej komunikat z powrotem do Claude i poproś o poprawkę tego pliku, bez przebudowy układu.
