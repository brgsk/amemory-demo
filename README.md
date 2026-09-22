# Pamięć agenta od podstaw

Materiały z warsztatu **Budujemy pamięć agenta: od prostego magazynu do świadomych decyzji**, LangChain Community Meetup, Wrocław, 22 września 2026. Prowadzący: [Bartosz Roguski](https://brgsk.xyz).

Rozwijamy dwie funkcje — `save_memory` i `search_memory` — oraz sprawdzamy ich działanie w rozmowie z modelem. Każde rozszerzenie odpowiada na konkretne wymaganie. To jedna możliwa implementacja; nie każdy system potrzebuje wszystkich pięciu etapów.

## Materiały

- [Notebook](agent_memory_workshop.ipynb) — kod, objaśnienia i zapisane odpowiedzi modelu.
- [Zapis przebiegu HTML](agent_memory_workshop.html) — pobierz i otwórz w przeglądarce, bez Pythona i klucza API.
- [Slajdy wprowadzające](workshop_intro.html) — otwórz lokalnie w przeglądarce; nawigacja strzałkami.

## Uruchomienie

Potrzebujesz Pythona 3.12–3.14, [uv](https://docs.astral.sh/uv/getting-started/installation/) oraz klucza OpenAI z dostępem do API. Wykonanie notebooka korzysta z płatnego API; odczyt zapisanych wyników nie wymaga klucza.

```sh
git clone https://github.com/brgsk/amemory-demo.git
cd amemory-demo
uv sync --locked --extra openai
uv run --extra openai jupyter lab agent_memory_workshop.ipynb
```

W Jupyter wybierz **Restart Kernel and Run All Cells**. Komórka konfiguracji poprosi o klucz w ukrytym polu `getpass`. Klucz pozostaje w środowisku procesu kernela; notebook nie zapisuje go do pliku i nie odczytuje `.env`. Jeśli `OPENAI_API_KEY` jest już ustawiony, pytanie się nie pojawi.

Domyślny model to `gpt-4.1-2025-04-14`, embeddingi: `text-embedding-3-small`. Odpowiedzi i argumenty narzędzi mogą różnić się między uruchomieniami. Zmiana modelu przez `WORKSHOP_MODEL` może wymagać dostosowania promptu i ponownej weryfikacji.

Opcjonalnie możesz ustawić `WORKSHOP_PROVIDER=anthropic`, `WORKSHOP_MODEL` na model tego dostawcy i zainstalować `uv sync --locked --extra anthropic`. Konfiguracja poprosi wtedy również o `ANTHROPIC_API_KEY`. OpenAI nadal jest potrzebne do embeddingów.

Notebook jest samowystarczalny: zawiera kod, grafiki i panel. Nie importuje `workshop.py` ani lokalnych modułów projektu.

## Pięć etapów

| Etap | Co dodajemy | Wymaganie |
|---|---|---|
| 1 | Lista tekstów i wyszukiwanie semantyczne | Korzystanie z faktów w nowej rozmowie |
| 2 | Kategoria i filtr | Kontrola zakresu odczytu |
| 3 | Klucz identyfikujący fakt | Aktualizacja zamiast dopisywania sprzecznych wersji |
| 4 | `valid_from`, `valid_to`, `valid_on` | Odczyt według daty obowiązywania |
| 5 | `recorded_from`, `recorded_to`, `known_on` | Odtworzenie wiedzy sprzed spóźnionej korekty |

Model wyodrębnia fakty z naturalnej rozmowy, wybiera argumenty narzędzi i interpretuje wyniki. Kod pilnuje zapisu, wersjonowania i filtrów; nie wybiera za model środka transportu.

Wyszukiwanie stosuje filtry, a następnie zwraca do trzech kandydatów według podobieństwa embeddingów. Podobieństwo nie gwarantuje przydatności ani kompletności wyników.

## Panel pamięci i praca krok po kroku

W sekcji **Panel pamięci** uruchom komórkę i kliknij **Otwórz pamięć na żywo**. Panel pokazuje rekordy jako węzły; kliknięcie odsłania szczegóły. Kolor oznacza kategorię, przygaszenie dawną wersję wiedzy. Położenie nie przedstawia podobieństwa ani relacji.

Panel działa lokalnie, dopóki działa kernel. Po restarcie uruchom komórkę ponownie i otwórz nowy link. Zapisany HTML nie ma połączenia z żywym panelem.

Każdy etap resetuje magazyn i ponownie wprowadza fakty rozmową. Wykonuj definicje, a potem demonstracje danego etapu. Po zmianie funkcji utwórz agenta ponownie przez `make_agent`, żeby otrzymał aktualne schematy narzędzi. Nie uruchamiaj starego demo z definicjami późniejszego etapu.

## Czas i ograniczenia przykładu

Przedziały są półotwarte: `[od, do)`. „Do końca września” oznacza `valid_to="2026-10-01"`. `None` oznacza brak końca. Model odczytuje okres obowiązywania z rozmowy, a czas zapisu nadaje aplikacja. `TODAY` to zegar przesuwany na potrzeby demonstracji.

Dwie osie pozwalają zapytać o ten sam wyjazd według wiedzy dostępnej przed korektą i po niej. Pytania historyczne korzystają ze świeżych wątków: filtr pamięci nie usuwa późniejszych informacji z historii rozmowy.

To przykład dla jednego użytkownika, z magazynem w RAM. Nie zawiera trwałej bazy, kontroli uprawnień, weryfikacji źródeł ani transakcji. Klucz faktu wybiera model, więc błędna nazwa może utworzyć duplikat. Zegar ma rozdzielczość dnia. Zachowanie poprawnych przedziałów nie gwarantuje poprawnej ekstrakcji ani odpowiedzi modelu.

## Własne eksperymenty

Edytuj funkcje i wiadomości bezpośrednio w notebooku. Cała implementacja jest w jego komórkach; osobne źródła i generator nie są potrzebne. Przed zmianami zachowaj kopię notebooka z oryginalnymi wynikami.

## Dalsza lektura

- [Agent memory: an anatomy](https://brgsk.xyz/agent-memory-anatomy)
- [LangChain — Memory overview](https://docs.langchain.com/oss/python/concepts/memory)
- [LangGraph — pętla narzędzi](https://docs.langchain.com/oss/python/langgraph/quickstart)
- [Bitemporal History — Martin Fowler](https://martinfowler.com/articles/bitemporal-history.html)
- [Making Sense of Memory in AI Agents — Leonie Monigatti](https://www.leoniemonigatti.com/blog/memory-in-ai-agents.html)
- [Memory in Agents — Philipp Schmid](https://www.philschmid.de/memory-in-agents)
