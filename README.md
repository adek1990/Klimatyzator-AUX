# Moduły Wi-Fi do klimatyzatorów AUX - instrukcje

![Platforma](https://img.shields.io/badge/platforma-ESP32--C3%20%7C%20ESP8266-0e6b6b)
![Firmware](https://img.shields.io/badge/firmware-ESPHome%20%2B%20aux__ac-b45f1b)
![Magistrala](https://img.shields.io/badge/UART-4800%208E1-6d7783)
![Dokumenty](https://img.shields.io/badge/format-PDF-a8261c)

Dokumentacja użytkownika modułów Wi-Fi, które wpina się w gniazdo fabrycznego modułu
sieciowego jednostki wewnętrznej klimatyzatora AUX. Moduł udostępnia klimatyzator w systemie
automatyki domowej przez ESPHome - bez chmury producenta i bez aplikacji mobilnej.

---

## Instrukcje

| Dokument | Moduł | Złącze | Zamiana TX/RX |
| --- | --- | --- | --- |
| [Moduł Wi-Fi AUX / ESP32-C3](Instrukcja%20modu%C5%82u%20klimatyzatora%20WiFi%20ESP32-C3.pdf) | ESP32-C3 | wtyk USB-A | zworki 2,54 mm |
| [Moduł Wi-Fi AUX / ESP8266](Instrukcja%20modulu%20klimatyzatora%20WiFi%20ESP8266%20%28JST-SM%29.pdf) | ESP8266 (ESP-12E/F) | JST-SM 5-pin | przeróbka wiązki |

Instrukcje są samodzielnymi plikami PDF - nie wymagają niczego poza czytnikiem.

## Który moduł mam?

Rozpoznasz go po zakończeniu wiązki:

- **Wtyk USB-A** - wersja na ESP32-C3. Ma zewnętrzną antenę na złączu gwintowanym, okno
  zworek wyboru kierunku linii TX/RX oraz listwę goldpinów do programowania i rozbudowy.
  Kształt wtyku nie oznacza zgodności z USB - na jego stykach jest magistrala klimatyzatora.
- **Wtyk JST-SM 5-pin** - wersja na ESP8266. Antena jest wykonana na płytce, obudowa ma tylko
  wyjście wiązki, a firmware aktualizuje się wyłącznie przez OTA.

## Co znajdziesz w instrukcji

Oba dokumenty mają ten sam układ:

1. **Bezpieczeństwo** - praca przy jednostce zasilanej z sieci 230 V
2. **Opis modułu** - budowa, funkcje, ułożenie w jednostce wewnętrznej
3. **Złącze klimatyzatora** - rozkład styków i przypisanie sygnałów
4. **Zasilanie** - parametry i próby na stanowisku
5. **Podłączenie magistrali UART** - kolejność czynności krok po kroku
6. **Zamiana linii TX/RX** - gdy płyta ma odwrotny opis gniazda
7. **Konfiguracja ESPHome** - gotowy plik YAML
8. **Uruchomienie i diagnostyka** - listy kontrolne i tabela *objaw → przyczyna → działanie*
9. **Dane techniczne**
10. **Ograniczenie odpowiedzialności**

## Zanim zaczniesz

> [!WARNING]
> Płyta główna jednostki wewnętrznej pracuje pod napięciem **230 V AC**. Wszystkie czynności
> montażowe wykonuj przy zasilaniu odłączonym w rozdzielnicy - nie pilotem i nie wyłącznikiem
> urządzenia - zabezpieczonym przed przypadkowym załączeniem, po odczekaniu co najmniej 5 minut.

> [!CAUTION]
> Moduł nie ma separacji galwanicznej: jego masa jest wspólna z masą płyty sterującej.
> Nie podłączaj komputera ani programatora do modułu wpiętego w klimatyzator.

Montaż jest przeznaczony dla osób z wiedzą z zakresu elektroniki i instalacji elektrycznych.
Ingerencja w instalację klimatyzatora może oznaczać utratę gwarancji producenta.

## Firmware

Moduły pracują pod ESPHome z komponentem `aux_ac`, obsługującym zastrzeżony protokół
szeregowy klimatyzatorów AUX (4800 bit/s, 8E1):

- [GrKoR/esphome_aux_ac_component](https://github.com/GrKoR/esphome_aux_ac_component) - komponent
- [GrKoR/AUX_HVAC_Protocol](https://github.com/GrKoR/AUX_HVAC_Protocol) - opis protokołu

Gotowe konfiguracje YAML dla obu wariantów sprzętowych są w instrukcjach, w sekcji 7.

### Gotowy firmware ESP32-C3

Szablon [esphome/klimatyzator-esp32c3.yaml](esphome/klimatyzator-esp32c3.yaml) i skompilowane
binarki w [Releases](https://github.com/adek1990/Klimatyzator-AUX/releases/latest). Firmware nie zawiera
danych sieci ani haseł - jeden plik pasuje do każdego modułu.

| Plik | Zastosowanie |
| --- | --- |
| `klimatyzator-c3-firmware.factory.bin` | pierwsze wgranie przewodowo (web.esphome.io albo esptool od adresu `0x0`) |
| `klimatyzator-c3-firmware.ota.bin` | aktualizacja OTA z ESPHome / Home Assistant |

1. Wgraj `factory.bin` przez listwę goldpinów (D+, D−, VDD 5V, GND), przy module wyjętym z klimatyzatora.
2. Połącz się z otwartą siecią **`Klimatyzator-ESPC3`** - captive portal (192.168.4.1) pozwala wybrać
   Wi-Fi. Alternatywnie Wi-Fi przez USB (Improv Serial) z web.esphome.io.
3. Home Assistant wykryje urządzenie `klimatyzator-c3-xxxxxx` (końcówka z adresu MAC).
   API bez szyfrowania, OTA bez hasła - po przejęciu w ESPHome dodaj własne.

UART: TX = GPIO21, RX = GPIO20 (4800 8E1), log przez USB-Serial/JTAG, więc UART0 zostaje wolny
dla klimatyzatora. Kierunek linii ustawiają zworki TX/RX (instrukcja, sekcja 5).

## Dodatek: termometr pokojowy

Klimatyzator mierzy temperaturę w swojej obudowie, pod sufitem. Przy grzaniu trafia tam najcieplejsze
powietrze, więc jednostka wyłącza się, zanim w pokoju zrobi się ciepło; przy chłodzeniu bywa odwrotnie.
Protokół AUX nie pozwala podać jednostce innej temperatury (funkcja iFeel działa tylko z pilota), dlatego
moduł robi to sam: **regulator PI przesuwa nastawę klimatyzatora tak, żeby w pokoju - tam, gdzie stoi
termometr - była zadana temperatura**. Klimatyzator dalej sam płynnie steruje sprężarką.

Termometr może być:

| Źródło | Co potrzeba | Zalety |
| --- | --- | --- |
| **DS18B20 na kablu** | czujnik DS18B20 wpięty w GPIO4 albo GPIO5 | działa bez Home Assistant i bez Bluetooth |
| **Termometr BLE** | np. Xiaomi LYWSD03MMC z firmware pvvx | bez kabli, na baterii ~rok, wilgotność gratis |
| **Dowolny czujnik z HA** | czujnik temperatury już obecny w Home Assistant | bez dodatkowego sprzętu |

### Ustawienia w Home Assistant (urządzenie modułu)

| Encja | Znaczenie |
| --- | --- |
| Źródło temperatury pokoju | `Czujnik klimatyzatora` (regulacja wyłączona), `Termometr z Home Assistant`, `DS18B20 na GPIO4`, `DS18B20 na GPIO5` |
| Regulacja w trybach | `Grzanie i chłodzenie`, `Tylko grzanie`, `Tylko chłodzenie` |
| Temperatura zadana pokoju - grzanie / - chłodzenie | cel dla pokoju, osobno dla każdego trybu |
| Stan regulacji | co regulator teraz robi, np. `pokój 20.6 / cel 21.0 -> nastawa 23.0` |
| Regulator Kp / Ki | nastawy regulatora PI (domyślnie 1,0 / 1,0) - wpisuje je autostrojenie, można zmienić ręcznie |
| Autostrojenie - start / - przerwij | dobranie Kp i Ki do konkretnego pokoju i klimatyzatora |

Gdy regulacja działa, nastawa widoczna na karcie klimatyzatora to wartość wyliczona przez regulator
(maks. ±4 °C od celu). Zmiana nastawy z pilota albo karty jest traktowana jak nowy cel pokoju dla
bieżącego trybu. Bez odczytu z termometru (czujnik odłączony, termometr z HA milczy ponad 15 min)
moduł ustawia nastawę równą celowi - klimatyzator pracuje wtedy jak zwykle, według swojego czujnika.

### Autostrojenie

Każdy pokój nagrzewa się i chłodzi inaczej, więc domyślne nastawy regulatora to tylko punkt wyjścia.
Autostrojenie dobiera je metodą przekaźnikową: moduł przełącza nastawę klimatyzatora o ±2 °C wokół
punktu pracy za każdym razem, gdy temperatura w pokoju przekroczy cel, i mierzy okres oraz wielkość
powstałych wahań. Po dwóch pełnych okresach wylicza Kp i Ki (łagodne reguły Tyreusa-Luybena, bez
przeregulowań) i zapisuje je w module.

- Uruchom, gdy regulacja według termometru działa już od kilku godzin i warunki są typowe:
  zamknięte okna i drzwi, bez gotowania i bez słońca wpadającego prosto na termometr.
- Trwa zwykle **2-5 godzin**; w tym czasie temperatura w pokoju waha się o około ±0,5-1 °C wokół celu.
  Postęp widać w `Stan regulacji` (`przełączeń 3/5`). Nocny restart modułu jest wtedy wstrzymany.
- Strojenie przerywa: zmiana nastawy z pilota lub karty, zmiana trybu, wyłączenie klimatyzatora,
  brak odczytu z termometru, brak reakcji pokoju przez 3 h albo łącznie ponad 8 h. Nastawy zostają wtedy bez zmian.
- Grzanie i chłodzenie zachowują się inaczej - najlepiej stroić w trybie, w którym klimatyzator pracuje
  najczęściej. Wynik obowiązuje dla obu trybów.

### DS18B20 na kablu

- Połączenie: VDD - 3,3 V, GND - GND, DATA - GPIO4 albo GPIO5. Jeden czujnik na pin.
- Rezystor **4,7 kΩ między DATA a 3,3 V** - przy krótkim kablu wystarczy wbudowany podciąg ESP,
  przy dłuższym (powyżej ~1 m) rezystor jest konieczny.
- Czujnik jest wykrywany przy starcie modułu - po podłączeniu zrestartuj moduł (przycisk `Force Reboot`).
- Odczyt widać w encji `Temperatura DS18B20 GPIO4` / `GPIO5`; potem wybierz to źródło w module.

### Termometr BLE (pvvx)

Moduł działa jako **Bluetooth proxy** - Home Assistant sam wykrywa termometry BLE w jego zasięgu.

1. Kup termometr obsługiwany przez firmware pvvx, np. Xiaomi **LYWSD03MMC** (także MHO-C401, CGG1 i inne -
   lista w [pvvx/ATC_MiThermometer](https://github.com/pvvx/ATC_MiThermometer)).
2. W Chrome lub Edge otwórz [TelinkMiFlasher](https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html),
   połącz się z termometrem, wgraj firmware pvvx (`Custom Firmware` -> `Start Flashing`).
3. Po ponownym połączeniu ustaw **Advertising type: BTHome v2** i zapisz (`Send Config`).
4. W HA: **Ustawienia -> Urządzenia i usługi** - pojawi się wykryte urządzenie **BTHome**, dodaj je.
5. Zaimportuj blueprint:
   [![Importuj blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fadek1990%2FKlimatyzator-AUX%2Fblob%2Fmain%2Fblueprints%2Fklimatyzator-termometr.yaml)
   ([blueprints/klimatyzator-termometr.yaml](blueprints/klimatyzator-termometr.yaml)).
6. Utwórz z niego automatyzację: wybierz termometr z listy czujników temperatury i encję
   `Temperatura z termometru` modułu.
7. W module ustaw `Źródło temperatury pokoju` na `Termometr z Home Assistant`.

Ten sam blueprint działa z każdym czujnikiem temperatury w HA (Zigbee, Wi-Fi, inny moduł).

### Gdzie postawić termometr

Na wysokości 1-1,5 m, tam gdzie się przebywa, z dala od strumienia powietrza z klimatyzatora,
słońca, grzejnika i drzwi. Termometr w strumieniu nawiewu da regulację skokową.

## Zastrzeżenie

Moduły nie są wyrobami certyfikowanymi do współpracy z konkretnym modelem klimatyzatora ani
produktami autoryzowanymi przez producenta urządzenia klimatyzacyjnego. Montaż, podłączenie
i eksploatacja odbywają się na wyłączną odpowiedzialność użytkownika. Firmware oparty jest na
oprogramowaniu open source, dostarczanym „tak jak jest", bez gwarancji przydatności do
określonego celu.

Oznaczenia na płytce i na złączu mają pierwszeństwo przed tabelami w dokumentach.
