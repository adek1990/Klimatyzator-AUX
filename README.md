# Moduły Wi-Fi do klimatyzatorów AUX — instrukcje

![Platforma](https://img.shields.io/badge/platforma-ESP32--C3%20%7C%20ESP8266-0e6b6b)
![Firmware](https://img.shields.io/badge/firmware-ESPHome%20%2B%20aux__ac-b45f1b)
![Magistrala](https://img.shields.io/badge/UART-4800%208E1-6d7783)
![Dokumenty](https://img.shields.io/badge/format-PDF-a8261c)

Dokumentacja użytkownika modułów Wi-Fi, które wpina się w gniazdo fabrycznego modułu
sieciowego jednostki wewnętrznej klimatyzatora AUX. Moduł udostępnia klimatyzator w systemie
automatyki domowej przez ESPHome — bez chmury producenta i bez aplikacji mobilnej.

---

## Instrukcje

| Dokument | Moduł | Złącze | Zamiana TX/RX |
| --- | --- | --- | --- |
| [Moduł Wi-Fi AUX / ESP32-C3](Instrukcja%20modu%C5%82u%20klimatyzatora%20WiFi%20ESP32-C3.pdf) | ESP32-C3 | wtyk USB-A | zworki 2,54 mm |
| [Moduł Wi-Fi AUX / ESP8266](Instrukcja%20modulu%20klimatyzatora%20WiFi%20ESP8266%20%28JST-SM%29.pdf) | ESP8266 (ESP-12E/F) | JST-SM 5-pin | przeróbka wiązki |

Instrukcje są samodzielnymi plikami PDF — nie wymagają niczego poza czytnikiem.

## Który moduł mam?

Rozpoznasz go po zakończeniu wiązki:

- **Wtyk USB-A** — wersja na ESP32-C3. Ma zewnętrzną antenę na złączu gwintowanym, okno
  zworek wyboru kierunku linii TX/RX oraz listwę goldpinów do programowania i rozbudowy.
  Kształt wtyku nie oznacza zgodności z USB — na jego stykach jest magistrala klimatyzatora.
- **Wtyk JST-SM 5-pin** — wersja na ESP8266. Antena jest wykonana na płytce, obudowa ma tylko
  wyjście wiązki, a firmware aktualizuje się wyłącznie przez OTA.

## Co znajdziesz w instrukcji

Oba dokumenty mają ten sam układ:

1. **Bezpieczeństwo** — praca przy jednostce zasilanej z sieci 230 V
2. **Opis modułu** — budowa, funkcje, ułożenie w jednostce wewnętrznej
3. **Złącze klimatyzatora** — rozkład styków i przypisanie sygnałów
4. **Zasilanie** — parametry i próby na stanowisku
5. **Podłączenie magistrali UART** — kolejność czynności krok po kroku
6. **Zamiana linii TX/RX** — gdy płyta ma odwrotny opis gniazda
7. **Konfiguracja ESPHome** — gotowy plik YAML
8. **Uruchomienie i diagnostyka** — listy kontrolne i tabela *objaw → przyczyna → działanie*
9. **Dane techniczne**
10. **Ograniczenie odpowiedzialności**

## Zanim zaczniesz

> [!WARNING]
> Płyta główna jednostki wewnętrznej pracuje pod napięciem **230 V AC**. Wszystkie czynności
> montażowe wykonuj przy zasilaniu odłączonym w rozdzielnicy — nie pilotem i nie wyłącznikiem
> urządzenia — zabezpieczonym przed przypadkowym załączeniem, po odczekaniu co najmniej 5 minut.

> [!CAUTION]
> Moduł nie ma separacji galwanicznej: jego masa jest wspólna z masą płyty sterującej.
> Nie podłączaj komputera ani programatora do modułu wpiętego w klimatyzator.

Montaż jest przeznaczony dla osób z wiedzą z zakresu elektroniki i instalacji elektrycznych.
Ingerencja w instalację klimatyzatora może oznaczać utratę gwarancji producenta.

## Firmware

Moduły pracują pod ESPHome z komponentem `aux_ac`, obsługującym zastrzeżony protokół
szeregowy klimatyzatorów AUX (4800 bit/s, 8E1):

- [GrKoR/esphome_aux_ac_component](https://github.com/GrKoR/esphome_aux_ac_component) — komponent
- [GrKoR/AUX_HVAC_Protocol](https://github.com/GrKoR/AUX_HVAC_Protocol) — opis protokołu

Gotowe konfiguracje YAML dla obu wariantów sprzętowych są w instrukcjach, w sekcji 7.

## Zastrzeżenie

Moduły nie są wyrobami certyfikowanymi do współpracy z konkretnym modelem klimatyzatora ani
produktami autoryzowanymi przez producenta urządzenia klimatyzacyjnego. Montaż, podłączenie
i eksploatacja odbywają się na wyłączną odpowiedzialność użytkownika. Firmware oparty jest na
oprogramowaniu open source, dostarczanym „tak jak jest", bez gwarancji przydatności do
określonego celu.

Oznaczenia na płytce i na złączu mają pierwszeństwo przed tabelami w dokumentach.
