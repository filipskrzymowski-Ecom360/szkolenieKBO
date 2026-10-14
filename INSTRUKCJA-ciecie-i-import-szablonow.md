# Cięcie makiety i import szablonu Klaviyo

Uniwersalna instrukcja dla Claude. Prowadź użytkownika krok po kroku, od pustego Cursora do gotowego szablonu drag-and-drop. Jeden krok na wiadomość. Napisz, co ma kliknąć, poczekaj na odpowiedź i dopiero wtedy idź dalej. Nie zakładaj, że MCP, klucz API, makieta albo linki już są.

Efekt końcowy: szablon `SYSTEM_DRAGGABLE` w bibliotece Klaviyo, z żywym tekstem i przyciskami oraz z grafikami wgranymi przez REST API na to konto, które użytkownik potwierdził.

Grafiki wgrywaj wyłącznie przez REST (`POST /api/image-upload`). Szablon twórz tym samym prywatnym kluczem. MCP służy do zalogowania konta i odczytu. Token OAuth z MCP nie zastępuje klucza API.

Klucza prywatnego nie zapisuj w repozytorium, w `mcp.json`, w markdownie ani w historii plików projektu. Trzymaj go w zmiennej środowiska albo w pliku tymczasowym poza repozytorium (`umask 077`) i usuń go po zakończeniu.

Cięcie grafik: skill `.cursor/skills/ciecie-grafik-mailingu/SKILL.md`.

---

## Jak prowadzić

Na początku napisz użytkownikowi, dokąd idziecie, w czterech zdaniach:

1. Podłączymy Klaviyo w Cursorze (MCP) i sprawdzimy, czy to właściwe konto.
2. Utworzymy prywatny klucz API do wgrania obrazków i szablonu.
3. Potniemy makietę i pokażemy podgląd do akceptacji.
4. Wgramy grafiki przez API i złożymy szablon. Na końcu dostanie link do edytora. Nic nie zostanie wysłane.

Potem wykonuj kroki po kolei. Jeśli użytkownik utknie, zostań przy tym kroku i podaj dokładniejszą ścieżkę kliknięć. Nie przeskakuj do cięcia ani uploadu.

---

## Krok 1. Podłącz MCP Klaviyo

Sprawdź, czy w tej sesji jest serwer MCP Klaviyo i czy da się wywołać odczyt konta (`get_account_details` albo równoważne narzędzie).

Gdy połączenie działa, odczytaj nazwę konta i napisz: „Jestem połączony z kontem: **{nazwa}**. Czy to właściwe konto?” Czekaj na „tak”. Przy „nie” przeprowadź ponowne logowanie na właściwe konto i zapytaj jeszcze raz.

Gdy MCP nie ma albo wymaga logowania, daj użytkownikowi tę instrukcję i zatrzymaj się do czasu, aż napisze, że skończył:

1. Zaloguj się w przeglądarce na konto Klaviyo, którego ma dotyczyć mail. Rola na koncie: Owner, Admin albo Manager.
2. W Cursorze: **Settings → Cursor Settings → MCP → New MCP Server**.
3. Wklej konfigurację. Adres bez ukośnika na końcu:

```json
{
  "mcpServers": {
    "klaviyo": {
      "url": "https://mcp.klaviyo.com/mcp"
    }
  }
}
```

4. Zapisz. Wejdź w **Settings → Cursor Settings → Tools & Integrations**. Przy serwerze Klaviyo kliknij **Needs login**.
5. W oknie Klaviyo zatwierdź dostęp (**Allow**). Wybierz to samo konto, na które jesteś zalogowany.
6. Napisz w czacie „gotowe”.

Gdy logowanie się nie uda: w MCP kliknij Logout i zaloguj się ponownie. Gdy to nie pomoże, wyloguj się z Cursora i zaloguj jeszcze raz, potem powtórz punkt 4.

Kilka kont Klaviyo: osobny wpis na każde konto, z nazwą w kluczu i parametrem `company` równym nazwie konta podanej przez użytkownika:

```json
{
  "mcpServers": {
    "klaviyo-nazwa-konta": {
      "url": "https://mcp.klaviyo.com/mcp?company=nazwa-konta"
    }
  }
}
```

W jednej rozmowie włączone jest tylko to konto, które użytkownik właśnie potwierdził.

Klucza prywatnego do tego kroku nie potrzeba. Nie wpisuj klucza do `mcp.json`.

---

## Krok 2. Prywatny klucz API

Napisz użytkownikowi, po co jest klucz: obrazki i szablon wchodzą osobnym requestem REST. Login MCP tego nie robi.

Daj mu ścieżkę i czekaj, aż wklei klucz w czacie:

1. Zostań zalogowany na konto potwierdzone w kroku 1.
2. W Klaviyo: **Settings** (ikona konta, lewy dół) → **API keys**.
3. **Create Private API Key**.
4. Nazwa dowolna, na przykład `Cursor — szablony`.
5. Uprawnienia, reszta wyłączona:
   - **Accounts** — Read
   - **Images** — Full
   - **Templates** — Full
6. Utwórz klucz i wklej go tutaj. Klucz zaczyna się od `pk_`. Po wklejeniu nie powtarzaj go w odpowiedzi.

Gdy klucz przyjdzie:

1. Odłóż go poza repozytorium.
2. Sprawdź konto:

```bash
curl -sS "https://a.klaviyo.com/api/accounts/" \
  -H "Authorization: Klaviyo-API-Key $KEY" \
  -H "revision: 2026-07-15" \
  -H "Accept: application/vnd.api+json"
```

3. Porównaj nazwę konta z nazwą z MCP. Zgadzają się — napisz użytkownikowi obie nazwy i idź dalej. Różnią się — przerwij, powiedz którą nazwę zwraca klucz, i poproś o klucz z konta z kroku 1. Nic nie wgrywaj.

---

## Krok 3. Makieta

Poproś o plik makiety (przeciągnięcie do czatu albo ścieżka na dysku).

Powiedz, jaki plik daje ostre cięcie: PNG sRGB, szerokość 2× maila (mail 600 px → plik 1200 px), albo PSD z warstwami, albo PDF. JPG ze zrzutu ekranu i plik z komunikatora da artefakty przy literach — wspomnij o tym, jeśli takie źródło przyjdzie, i tnij dalej dopiero po zgodzie użytkownika.

Skopiuj plik do `do-ciecia/`, jeśli jeszcze tam nie leży. Potwierdź nazwę pliku.

Akceptowane rozszerzenia: `.jpg` `.jpeg` `.png` `.webp` `.gif` `.tif` `.tiff` `.psd` `.pdf` `.svg`.

Gdy użytkownik prosi wyłącznie o cięcie, po kroku 4 zakończ. Szablon powstaje, gdy prosi o wgranie do Klaviyo.

---

## Krok 4. Cięcie

Nie tnij w agencie nadrzędnym. Odpal subagenta:

- Tool: `Task`
- `subagent_type`: `generalPurpose`
- `model`: `claude-opus-5-thinking-high`
- W prompcie: treść `.cursor/skills/ciecie-grafik-mailingu/SKILL.md`, bezwzględna ścieżka pliku, katalog wyjścia `ciecia/<stem>/`

### Co ma powstać w `ciecia/<stem>/`

1. Wycinki `NN-opis-elementu.png` (zdjęcia mogą być `.jpg`), numeracja z góry na dół, nazwy ASCII.
2. `preview.html` — galeria, wymiary, waga, `alt`.
3. `ciecia.zip` — same wycinki, bez pliku źródłowego.
4. `manifest.json` — `box` `[x, y, w, h]`, format, `alt`, `width_at_600`, lista `not_cut`.
5. `palette.json` — kolory tła zmierzone z pikseli (hex i rgb). W szablonie używaj tych wartości.

Skrypt: Python, Pillow i numpy. Po zapisie każdy wycinek otwórz narzędziem Read i popraw kadr, jeśli ucina znak albo zostawia obcy pas.

### Co wycinać

| Element | Plik | W szablonie |
|---|---|---|
| Logo, ikona, falka, pas dekoracyjny | PNG-24, zapas tła ok. 18–20 px | blok `image` |
| Zdjęcie | JPG 80–85% albo PNG, jeśli mniejszy; cięcie po krawędzi kadru | blok `image` |
| Przycisk | `NN-cta-referencja.png`, `reference_only: true` | żywy przycisk, nie ten plik |
| Tekst, ceny, nagłówki, regulamin | nie tnij | bloki `text` |

Mail ma 600 px szerokości. Wycinki zapisuj 1:1, bez drugiego skalowania. Szerokość bloku w szablonie bierz z `width_at_600`. Pojedyncza grafika do około 200 kB.

PDF renderuj wektorowo (PyMuPDF) od razu na 1200 px szerokości, potem tylko kadruj.

Po cięciu pokaż użytkownikowi `preview.html`, listę plików i zastrzeżenia jakości. Napisz: „Obejrzyj podgląd. Jak wycinki są dobre, napisz »wgrywaj«. Jak któryś kadr jest zły, wskaż numer pliku.” Czekaj.

---

## Krok 5. Linki i nazwa

Zanim powstanie szablon, zbierz od użytkownika to, czego nie ma w pikselach:

- nazwa szablonu w bibliotece Klaviyo,
- adres strony głównej (logo),
- adresy przycisków i zdjęć produktowych,
- adresy social mediów w stopce,
- przy mailu koszyka: adres koszyka użyty, gdy w evencie nie ma URL-a checkoutu.

Nie wymyślaj adresów. Brakujący link wypisz wprost i zapytaj.

---

## Krok 6. Import obrazków przez API

Wgrywaj dopiero po „wgrywaj” z kroku 4. Pomiń pliki `reference_only`.

Każdy wycinek:

```bash
curl -sS -X POST "https://a.klaviyo.com/api/image-upload" \
  -H "Authorization: Klaviyo-API-Key $KEY" \
  -H "revision: 2025-07-15" \
  -H "Accept: application/vnd.api+json" \
  -F "file=@ciecia/<stem>/01-logo.png;type=image/png" \
  -F "name=01-logo" \
  -F "hidden=false"
```

Dla JPEG ustaw `type=image/jpeg`. Z odpowiedzi zapisz:

- `data.id` — później `asset_id`,
- `data.attributes.image_url` — później `src`,
- `data.attributes.name`.

Mapę zapisz w `tools/<nazwa-maila>/urls.json`. W tym pliku nie ma klucza API.

Z pierwszego `image_url` odczytaj fragment `/company/{id}/`. Każdy kolejny plik musi mieć ten sam `{id}`. Inny identyfikator oznacza inny klucz — przerwij i wróć do kroku 2.

Ten sam plik, który ma już URL z tego konta, wgraj ponownie tylko na prośbę użytkownika. Kolejny upload dostaje nowy rekord w bibliotece.

Nie wysyłaj plików przez MCP `upload_image_from_url`, nie koduj ich jako `data:image/...;base64` i nie wystawiaj ich na zewnętrznym hostingu. Adres z dysku lokalnego Klaviyo nie pobierze.

Po serii napisz użytkownikowi: ile plików weszło i że adresy są na potwierdzonym koncie. Potem złóż szablon.

---

## Krok 7. Szablon drag-and-drop

Body tworzenia:

```json
{
  "data": {
    "type": "template",
    "attributes": {
      "name": "NAZWA_Z_KROKU_5",
      "editor_type": "SYSTEM_DRAGGABLE",
      "text": "Wersja tekstowa: nagłówki, treść, adresy przycisków, stopka z wypisem.",
      "definition": {
        "styles": [],
        "body": {
          "properties": { "css_class": "root-container" },
          "styles": { "background_color": "#KOLOR_Z_PALETY", "width": 600 },
          "sections": []
        }
      }
    }
  }
}
```

`text` to pełna wersja maila bez HTML, łącznie z `{% unsubscribe_url %}`.

Kolory bierz z `palette.json`. Szerokość maila to 600. Odstęp karty od krawędzi odczytaj z `canvas` w manifeście.

Hierarchia: `section` → `row` → `column` → `blocks`.

Układy kolumn:

- `1-column-full-width`
- `2-columns-equal-width`
- `3-columns-equal-width`

W `definition.styles` ustaw style bazowe, tekstu, nagłówków i linków (font, rozmiar, kolor, wyrównanie) według makiety. Wartości liczbowe bierz z pomiaru wycinków, nie z pamięci.

### Obrazek

W bloku są oba pola: `src` i `asset_id`. Sam adres bez `asset_id` nie podpina pliku w edytorze.

```json
{
  "content_type": "block",
  "type": "image",
  "data": {
    "properties": {
      "dynamic": false,
      "alt_text": "opis z manifestu",
      "asset_id": "ID_Z_ODPOWIEDZI_UPLOADU",
      "href": "URL_Z_KROKU_5",
      "src": "URL_Z_CDN_KONTA"
    },
    "display_options": { "show_on": "all" },
    "styles": {
      "align": "center",
      "block_background_color": "#KOLOR_Z_PALETY",
      "full_width_mobile": true,
      "width": 600,
      "max_width": 600
    }
  }
}
```

`width` to `width_at_600` z manifestu. Zdjęcie na szerokość karty: `full_width_mobile: true`. Logo: węższe i wyśrodkowane. `alt_text` z manifestu.

### Tekst i przycisk

Tekst, cena, nagłówek, regulamin i wypis to bloki `text`. Treść w `data.content` jako HTML.

Przycisk o mocno zaokrąglonym kształcie zrób blokiem `html`: tabela prezentacyjna, `bgcolor` na komórce, link z paddingiem. Prosty prostokąt może być natywnym blokiem `button`. Pliku `*-cta-referencja.png` do maila nie wstawiaj.

W stopce: `{% unsubscribe 'Wypisz się' %}` albo `{% unsubscribe_url %}`. Preheader linkuje `{{ web_view }}`.

### Utworzenie

```bash
curl -sS -X POST "https://a.klaviyo.com/api/templates/" \
  -H "Authorization: Klaviyo-API-Key $KEY" \
  -H "revision: 2026-07-15" \
  -H "Content-Type: application/vnd.api+json" \
  -H "Accept: application/vnd.api+json" \
  --data-binary @tools/<nazwa-maila>/szablon.json
```

Odczyt z definicją:

```bash
curl -sS -G "https://a.klaviyo.com/api/templates/ID/" \
  --data-urlencode "additional-fields[template]=definition" \
  -H "Authorization: Klaviyo-API-Key $KEY" \
  -H "revision: 2026-07-15" \
  -H "Accept: application/vnd.api+json"
```

Poprawka to `PATCH` na ten sam adres:

```json
{
  "data": {
    "type": "template",
    "id": "ID",
    "attributes": {
      "name": "NAZWA_Z_KROKU_5",
      "definition": {}
    }
  }
}
```

Zmianę układu rób na kopii: odczytaj `definition` i zrób `POST` nowego szablonu z inną nazwą. Pierwszy szablon zostaw.

Podgląd:

```bash
curl -sS -X POST "https://a.klaviyo.com/api/template-render/" \
  -H "Authorization: Klaviyo-API-Key $KEY" \
  -H "revision: 2026-07-15" \
  -H "Content-Type: application/vnd.api+json" \
  -H "Accept: application/vnd.api+json" \
  --data-binary @render.json
```

`render.json` zawiera `data.id` szablonu i `attributes.context`, gdy mail korzysta z eventu. W HTML wyniku mają być adresy CDN tego konta. Surowe `{{ }}` albo błąd filtra oznacza, że tag trzeba uprościć i wgrać definicję jeszcze raz.

Na końcu podaj nazwę, id i link:

`https://www.klaviyo.com/email-template-editor/{id}`

Napisz: „Otwórz edytor i napisz, co poprawić.” Kampanii nie wysyłaj i nie przypinaj szablonu do flow, dopóki użytkownik o to nie poprosi. Klucz API usuń z pliku tymczasowego.

---

## Krok 8. Koszyk — tylko gdy makieta ma pozycje z zamówienia

Produkty z eventu to natywna tabela, nie jeden obrazek i nie pętla `{% for %}` w bloku HTML. Każdy produkt narysowany na makiecie na stałe to osobna komórka: osobny obrazek i osobny tekst.

Zanim wpiszesz tagi, weź z konta przykład prawdziwego eventu (porzucony koszyk albo zamówienie) i odczytaj nazwy pól. Typowy kształt ze sklepu na WooCommerce:

| Co | Tag, gdy event ma taki kształt |
|---|---|
| Zdjęcie | `{{ item.Images.0.URL }}` |
| Nazwa | `{{ item.Name }}` |
| Wariant | ścieżka do konkretnego atrybutu, odczytana z eventu |
| Cena | `{{ item.LineTotal }}` |
| URL produktu | `{{ item.ProductURL }}` |
| Powrót do koszyka | `{% if event.extra.CheckoutURL %}{{ event.extra.CheckoutURL }}{% else %}URL_Z_KROKU_5{% endif %}` |

Wypisanie całego obiektu wariantu pokazuje słownik. Użyj ścieżki do jednego atrybutu. Pole zdjęcia bierz z eventu; nie zakładaj nazwy, której w przykładzie nie ma.

Szkielet tabeli:

```json
{
  "content_type": "block",
  "type": "table",
  "data": {
    "properties": {
      "dynamic": true,
      "num_columns": 2,
      "show_headers": false,
      "show_fallback_content": true,
      "show_static_row": false,
      "row_content_repeat": {
        "repeat_for": "event.extra.Items",
        "item_alias": "item"
      }
    },
    "display_options": { "show_on": "all" },
    "styles": {
      "cell_horizontal_border_style": "none",
      "cell_vertical_border_style": "none",
      "cell_vertical_align": "middle"
    }
  },
  "subblocks": []
}
```

Wiersz dynamiczny: komórka zdjęcia i komórka tekstu. API przyjmuje jeden fallback. Drugi zestaw komórek fallback odrzuca. Przykładowy produkt z makiety daj jako jeden blok HTML pod tabelą.

Zaokrąglone rogi zdjęcia: komórka `table_html` z `border-radius` na `img`. Komórka `table_image` rogów nie zaokrągla.

W komórkach tabeli trzymaj tagi krótkie. Po każdej zmianie tagów wywołaj `template-render` z przykładowym `event.extra.Items` w kształcie z tego konta.

---

## Zasady przez całą pracę

- Obrazki: `POST https://a.klaviyo.com/api/image-upload`, nagłówek `Authorization: Klaviyo-API-Key`, rewizja `2025-07-15`.
- Szablony: `POST` i `PATCH https://a.klaviyo.com/api/templates/`, rewizja `2026-07-15`, `Content-Type: application/vnd.api+json`.
- Klucz i obrazy należą do jednego, potwierdzonego konta.
- Tekst, ceny i przyciski są edytowalne w szablonie. Do wysyłki nie idą jako wycinki.
- Kolory pochodzą z `palette.json`.
- Klucz `pk_` znika z dysku po oddaniu linku do edytora.
