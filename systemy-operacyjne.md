

# Procesy i Wątki

## Procesy

Proces jest abstrakcją działającego programu. Sam program jest niejako “przepisem na proces”. Procesor jest komponentem, który wykonuje program i udziela swojego kwantu czasu procesowi. Nie sposób wyobrazić sobie współczesnego systemu operacyjnego bez równobieżnych procesów. Procesor fizyczny przełącza się między procesami tworząc iluzję współbieżności. Procesy działające w tle innych procesów określane są jako “demony”. 

Procesy są tworzone w wyniku następujących zdarzeń:
- inicjalizacja systemu
- uruchomienie wywołania systemowego tworzącego proces przez inny proces
- żądanie użytkownika utworzenia nowego procesu
- inicjalizacja zadania wsadowego

Nowo utworzony proces dziedziczy wszystkie atrybuty od swojego “rodzica”

## Komunikacja między procesami

Procesy muszą komunikować się z innymi procesami. Dzieje się to z kilku powodów, między innymi mogą chcieć wymienić dane albo uzgodnić dostęp do zasobu. Jak rozwiązać komunikację? 

Rozważmy sytuację, której procesy nie komunikują się w tym kto ma dostęp do katalogu spoolera drukarki, w którym zapisywane są adresy plików do wydruku. Dwa procesy ścigają się o to który zapisze swój adres pliku w pierwszym wolnym indeksie. A następnie oba inkrementują wartość następnego wolnego indeksu. Skutkuje to tym, że jeden z plików zostanie pominięty i bardzo trudno będzie odtworzyć ten błąd aby znaleźć przyczynę.  

Aby podjąć próby rozwiązania sytuacji wyścigu należy określić pewne abstrakcyjne właściwości jakie dzieją się sytuacji wyścigu. Czasami proces musi skorzystać z pamięci współdzielonej lub z pliku, te obszary można nazwać sekcją krytyczną. Należy tak projektować operacje aby nigdy dwa procesy nie znalazły się w tych samych sekcjach krytycznych.

##### Określono cztery warunki, które będzie spełniało dobre rozwiązanie:

 - żadne dwa procesy nie mogą jednocześnie przebywać w swoich sekcjach krytycznych
 - nie można przyjmować założeń co do szybkości procesów
 - proces działający w swoim regionie krytycznym nie może blokować innych procesów
 - żaden proces nie powinien oczekiwać w nieskończoność na dostęp do swojego regionu krytycznego 

##### Blokowanie zmienną

Naiwnym rozwiązaniem będzie udostępnienie zmiennej która będzie określała dostępność zasobu. Spowoduje to tylko sytuację wyścigu do zmiany tej zmiennej.

##### Ścisła naprzemienność - blokada typu spin lock

O dostępie do zasobu decyduje zmienna turn, którą zmienia się po zwolnieniu dostępu do zasobu. Oba procesy mają zapisaną różną wartość zmiennej dla której mogą wejść do sekcji krytycznej. W ten sposób wymuszona jest ścisła naprzemienność dostępu do zasobu. Nie jest to rozwiązanie idealne bo często jeden proces potrzebuje skorzystać z zasobu wiele razy z rzędu. Ponadto ciągłe testowanie zmiennej (aktywne oczekiwanie), która decyduje o dostępie jest dodatkowym obciążeniem dla procesora.

##### Rozwiązanie Petersona

W 1981 roku Gary L. Peterson zaproponował rozwiązanie zawierające tabelę ‘zainteresowanych’ skorzystaniem z regionu krytycznego oraz zmienną przechowującą czyja jest teraz kolej. Rozwiązanie to wymusza oczekiwanie procesów na wejście do regionu krytycznego jednak nie eliminuje problemu ‘aktywnego oczekiwania’

##### Problem producent - konsument z sytuacją wyścigu.

Jest to prymityw komunikacji międzyprocesowej czyli operacja która kiedy proces nie może wejść do regionu krytycznego blokuje go lub usypia nie marnotrawiąc czasu procesora. 

Dwa procesy współdzielą bufor o określonym rozmiarze jeden dodaje (produkuje) a drugi odbiera (konsumuje) gdy producent chce umieścić element w buforze który jest zapełniony - zostaje uśpiony do czasu zwolnienia się miejsca a także kiedy konsument nie ma już elementów do pobrania - jest usypiany do czasu pojawienia się nowych danych.

Problemem tutaj jest nieograniczony dostęp do zmiennej przechowującej ilość elementów w buforze. 

 - Konsument odczytuje zmienną `count` i widzi wartość 0.
 - W tym czasie producent wstawił element do bufora i inkrementował wartość `count`
 - Konsument zasypia.
 - Producent zapełnia bufor i również zasypia.
 - W ten sposób oba proces śpią wiecznie. 

###### Semafory
Sposobem na ograniczenie dostępu do zmiennej `count` jest użycie zmiennej typu semafor int. Jest to do obiekt którego dostęp jest zarządzany przez instrukcje TSL (Test Set and Lock) lub XCHG, które to zapobiegają jednoczesnemu dostępowi przez kilka procesów na raz. 

W zaproponowanym rozwiązaniu użyto semaforów jako liczników wysyłanych sygnałów `wake up` do poszczególnych procesów. W ten sposób uniknięto uśpienia procesu na zawsze.

###### Muteksy
Gdy nie potrzeba zliczać sygnałów a jedynie blokować dostęp zamiast semaforów można użyć muteksów które przechowują  tylko wartości 1 i 0. 

###### Bariery
Bariery są mechanizmem wymuszającym synchronizację procesów lub wątków w celu koordynacji np obliczeń prowadzonych równolegle. Gdy proces wykona swoją pracę blokuje się do czasu aż pozostałe procesy z puli wykonają swoją pracę po czym następuje synchronizacja wyników i praca jest kontynuowana
  
###### Unikanie blokad - Karencja dostępu
Innym sposobem na ograniczanie dostępu jest pozwolenie jedynie na czasowy dostęp do zasobu. W ten sposób kilka procesów może korzystać z danej struktury np. Drzewa a w przypadku jego modyfikacji zachowuje się odstęp czasu ustalony na wykonanie operacji.

### Szeregowanie

Procesy możemy podzielić na zorientowane na obliczenia i zorientowane na operacje wejścia i wyjścia. To w jaki sposób procesor przydzieli swój czas obliczeniowy poszczególnym procesom określa algorytm szeregowania.
Istnieją dwie kategorie algorytmów szeregowania: wywłaszczające i bez wywłaszczenia.
Cele algorytmów szeregowania zależą od środowiska w jakim się znajdują:

Wszystkie systemy:
 - Sprawiedliwość - przydzielenie procesom należnego czasu
 - Wymuszanie strategii - sprawdzenie czy przestrzegana jest wybrana strategia
 - Równowaga - wszystkie części systemu powinny być zajęte
Systemy wsadowe
 - Przepustowość
 - Czas cyklu przetwarzania
Wykorzystanie procesora
 - Systemy interaktywne
 - Czas odpowiedzi
 - Proporcjonalność
Systemy czasu rzeczywistego
 - Dotrzymywanie terminów
 - Przewidywalność

#### Algorytmy szeregowania

##### Pierwszy zgłoszony pierwszy obsłużony
Każdy proces ustawia się w kolejkę i może działaś tak długo jak wypełni swoje zadanie. 
##### Najpierw najkrótsze zadanie
Algorytm optymalny gdy dużo zadań jest dostępnych jednocześnie. Odmianą tego algorytmu jest `Następny proces o najkrótszym czasie działania`, które działa podobnie tylko porównuje estymowany czas działania kolejno nadchodzących zadań.
##### Szeregowanie cykliczne
Procesor przydziela wybrany kwant czasu każdemu z zadań w kolejce. Im większy kwant czasu tym mniej tego czasu 'marnuje' się na przełączanie kontekstu pomiędzy procesami
##### Szeregowanie bazujące na priorytetach
Procesor daje więcej czasu procesom o wyższym priorytecie, przy czym priorytety mogą być zmienione tak, aby procesy o niższym priorytecie
##### Wielokrotne kolejki
Procesor udziela coraz większych kwantów czasu kolejnym cyklom danego procesu. W ten sposób nie traci się czasu na długie kwanty czasu a także nie traci się go zbyt dużo na częste zmiany kontekstu
##### Szeregowanie gwarantowane
Każdy proces otrzymuje 1/n mocy procesora gdzie n to liczba procesów
##### Szeregowanie loteryjne 
##### Szeregowanie sprawiedliwe
Szeregowanie sprawiedliwe uwzględnia także użytkownika w przydziale czasu procesora. Np. użytkownik 1 ma 10 procesów a użytkownik 2 ma 1 proces. Gdyby dzielić czas procesora szeregowaniem cyklicznym użytkownik 1 dostałby 90% czasu procesora. Zamiast tego należałoby częściej udzielać czasu procesowi użytkownika 2.
##### Problem ucztujących filozofów
##### Problem czytelników i pisarzy

# Systemy plików

## Pliki
  

Pliki zapisywane są na dysku w porcjach zwanych blokami. Zapis ten jest oderwany od struktury plików i katalogów reprezentowanych od strony użytkownika. Głównym problemem do zaimplementowania systemu plików jest rozpoznanie na jakich adresach bloków dysku są zapisane poszczególne pliki.

  

Potrzebne było stworzenie niezależnej struktury zwanej i-węzłem której zadaniem jest śledzenie informacji o adresach i atrybutach pliku

  

W przypadku plików współdzielonych stworzono dodatkową strukturę informującą o dowiązaniach do konkretnego pliku i o jego właścicielu.

  

Aby zapobiec błędom mogącym wystąpić w czasie awarii podczas operacji na systemie plików, stworzono mechanizmy księgujące operacje planowane przez system. W przypadku awarii system jest w stanie wrócić do miejsca i wykonać brakujące czynności.

  

Wirtualne systemy plików powstały jako adaptery łączące różne systemy plików w celu ujednolicenia interfejsu

## Zarządzanie systemem plików

Problem wyboru rozmiaru bloku na dysku ma wpływ na szybkość dostępu do bloku ale z drugiej strony na ilość traconego miejsca na dysku. Większy blok to więcej miejsca straconego na wyrównanie ale szybszy dostęp.

  

Śledzenie wolnych bloków na dysku może odbywać się za pomocą jednokierunkowych list z numerów wolnych bloków lub masek bitowych o określonej długości reprezentujących miejsca zajęte i wolnej.

## Kopie zapasowe.

  

Częste pełne kopie zapasowe mogą być nieefektywne z uwagi na ilość miejsca. Zamiast tego lepiej tworzyć kopie przyrostowe na bazie sporadycznej kopii pełnej

Istnieją kopie fizyczne i logiczne. Fizyczne odwzorowują dysk w całości a logiczne wybrane struktury katalogów i pliki. 

Podczas wykonywania kopii szczególnie ważne jest dokładne odwzorowanie i-węzłów plików i katalogów aby nie spowodować błędów prowadzących do nieprawidłowych lub utraconych powiązań. 

## Wydajność systemu plików

Technika buforowania

Gdy system otrzymuje żądanie odczytu pliku najpierw sprawdza czy ten plik znajduje się w buforowej pamięci podręcznej - cache. System przechowuje jednokierunkową listę skrótów plików umieszczonych w cache a następnie sprawdza czy żądany zasób znajduje się w tej liście i jeśli nie pobiera go z dysku i kopiuje do pamięci podręcznej. Ten sam sposób pliki mogą być zapisywane na dysk. Należy się liczyć z dwoma problemami związanymi z awariami i spójnością systemu plików.  gdy dojdzie do awarii w momencie gdy plik znajduje się w pamięci podręcznej zostanie on utracony. 

  

Technika czytania bloków zawczasu

Jedyne zastosowanie tej techniki to pliki o dostępie sekwencyjnym. Obecnie rzadko używane (w niektórych rodzajach formatów spakowanych).

  

Technika minimalizacji ruchu ramienia dysku

Pliki powinny być tak zapisywane w sąsiadującej grupie cylindrów. Obecnie większość komputerów posiada dyski SSD, których to zagadnienie nie dotyczy.

  

## Przykładowe systemy plików

MS DOS

Wpis w katalogu w systemie MS DOS (poniżej wielkość w bajtach i opis pola):

| 8 Nazwa | 3 Rozszerzenie | 1 Atrybuty | 10 Zarezerwowane | 2 Czas | 2 Data | 2 Numer pierwszego bloku | 4 Rozmiar

System MS DOS śledzi wolne bloki za pośrednictwem tablicy alokacji plików umieszczonej w pamięci głównej .

Maksymalna wielkość pliku jest zatem uzależniona od rozmiaru bloku dyskowego i od przyjętego wariantu systemu plików (FAT-12,16,32).

  

V7 systemu UNIX

Wpisy w systemie UNIX korzystają ze struktury i-węzłow, stąd ich prosta struktura wewnętrzna:

| 2 Numer węzła | 14 Nazwa |

Struktura i-węzłów tworzy strukturę drzewiastą zbudowaną z i-węzła, bloków pośrednich i adresów bloków dyskowych

  

# Input output

Urządzenia wejścia-wyjścia można omawiać pod względem ich budowy i parametrów fizycznych lub oprogramowania. Urządzenia powinny dostarczać sprzętowo niezależny interfejs dostępu. Dostęp do urządzeń może być blokowy lub znakowy. 

Urządzenia podłączone są z CPU za pomocą kontrolera znajdującego się na płycie głównej pełniącego funkcję adaptera. To on jest odpowiedzialny za przyjęcie danych do wewnętrznego bufora i przekazanie ich do systemu operacyjnego oraz za przetłumaczenie instrukcji i przekazanie jej do urządzenia.

Czasem dla urządzeń stosuje się DMA (Direct Memory Access), żeby ograniczyć czas procesora potrzebny na przesłanie danych do pamięci komputera. 

## Przerwania

Urządzenia Input-Output współpracują z procesorem asynchronicznie w strukturze przerwań. Procesor rozpoczyna transfer a gdy urządzenie ma dane dla procesora generuje odpowiedni typ przerwania, które procesor obsługuje natychmiast. Urządzeniem odpowiedzialnym za zbieranie i przekazywanie przerwań do procesora jest kontroler przerwań. 

  

Oprogramowanie wejścia-wyjścia zorganizowane jest w czterech warstwach:

- Oprogramowanie IO z poziomu użytkownika
    
- Oprogramowanie IO z poziomu  OS
    
- Sterowniki urządzeń
    
- Procedury przerwań
    

  

## Interfejs

Choć ideałem byłoby żeby urządzenia miały ten sam interfejs z punktu widzenia różnorodności zastosowań jest to niemożliwe. Postanowiono zatem podzielić urządzenia na typy choćby po to, żeby móc zastosować jeden sterownik do wybranego typu urządzeń ale także aby odpowiednio odwzorować urządzenie z punktu widzenia systemu operacyjnego i użytkownika

## Buforowanie

Ważnym elementem obsługi urządzeń znakowych są bufory. Należy zapewnić taką konfigurację aby nie utracić, żadnego znaku ale także utrzymać odpowiednią wydajność. Jednym z rozwiązań może być technika podwójnego buforowania dzięki, której naprzemiennie używa się dwóch buforów w przestrzeni jądra i naprzemiennie się je przesyła użytkownikowi. 

Ponadto buforowanie pełni ważną rolę w komunikacji sieciowej w której dane są wysyłane w pakietach

  
  

## Dyski i technologia RAID

Rozbieżność między szybkością procesorów a szybkością transmisji dysków rośnie. Potrzeba było usprawnić szybkość zapisu i odczytu danych z dysków. Kontroler RAID współpracuje z kilkoma napędami tak że z punktu widzenia te dyski stanowią jedno urządzenie ale zapis i odczyt jest dzielony pomiędzy nie dla zrównoleglenia czasu pracy.

Rozbicie żądania procesora na mniejsze części i rozdzielenie ich pomiędzy napędy to zdanie kontrolera RAID. 

Kod Hamminga - lokalizacja wadliwych bitów umieszczając bity kontrolne na przemian z bitami danych. 

## Formatowanie dysków

Ciągły odczyt danych wymaga pojemnego bufora. Jednak i on kiedyś się zapełni. Podczas gdy odbywa się transmisja z bufora do pamięci dysk czeka aż odpowiedni sektor znajdzie się pod głowicą. Gdyby sektory umieszczano jeden za drugim promieniście na dysku wówczas głowicą musiałaby czekać prawie pełny obrót aż kolejny sektor pojawi się ponownie. Dlatego stosuje się przeplot sektorów na talerzu dysku.

  

Algorytm windy - realizacja żądań obsługi cylindrów znajdujących się “po drodze” w jednym kierunku ruchu głowicy. Zastosowanie jedynie w dyskach HDD

## Zegar

Zegar pełni liczne funkcje w komputerze: przechowywanie aktualne godziny, niedopuszczanie aby procesy trwały zbyt długo, rozliczenia za wykorzystanie procesora, wywołanie systemowe ‘alarm’, liczniki nadzorujące, profilowanie monitorowanie i zbieranie statystyk.

## Interfejsy użytkowników: klawiatura, mysz, monitor

Użytkownicy wprowadzają dane wejściowe za pomocą urządzeń podłączonych od wyspecjalizowanych portów szeregowych. Np. w przypadku klawiatury naciśnięcie dowolnego klawisza generuje przerwanie. Monitor jest typowym urządzeniem wyjścia, chyba, że to monitor dotykowy. 

## GUI

Systemy Unix wywodzą swój graficzny interfejs z systemu X window którego podstawą jest komunikacja terminala z maszyną hosta. Wywodzi się to z czasów kiedy komputery najczęściej nie były osobiste i miały wielu użytkowników. Obecnie w systemach UNIXowych środowisko składa się z programów wysyłających polecenia serwerowi, który pisze lub rysuje na wyświetlaczu. GUI systemu Windows jest obsługiwane przez win32 API. Bazuje ono na modelach okien i zdarzeniach. Każdemu z okien odpowiada obiekt z klasy. Z każdym programem powiązana jest kolejka komunikatów i zbiór procedur obsługi. To całkowicie inny model od przyjętego modelu proceduralnego w systemach opartych na X window. 

## Cienkie klienty

Niewątpliwą zaletą komputerów współdzielonych była łatwa aktualizacja, konserwacja. Obecnie w czasach PC użytkownikom trudno jest utrzymać aktualność oprogramowania dobry stan maszyn. Dodatkowo prawie całą działalność użytkowników przeniosła się do sieci. Stąd powstał pomysł cienkich klientów czyli bardzo odchudzonych systemów operacyjnych opartych o założenie bycia cały czas online i dostępnym do aktualizacji (np ChromeOS)

# Zakleszczenia

  

Systemy operacyjne mogą przydzielać zasoby procesom. Muszą to robić w uporządkowany sposób aby nie dopuścić do używania jednego zasobu przez dwa procesy w tym samym czasie.

Zakleszczeniem można nazwać sytuację, w której dwa procesy zajęły zasoby których wzajemnie potrzebują lub bardziej ogólnie do zakleszczenia dochodzi wtedy gdy każdy proces z danego zbioru oczekuje na zdarzenie, które może spowodować tylko inny proces z tego zbioru.

Istnieją zasoby, które można wywłaszczyć od procesu oraz takie, których wywłaszczenie powodowałoby szkodę dla zasobu. 

  

typedef int semaphore;

semaphore pencil;

semaphore paper;

  

void process_A(void) {

down(&pencil);

down(&paper);

write_something();

up(&pencil);

up(&paper);

}

  

void process_B(void) {

down(&paper);

down(&pencil);

write_something();

up(&pencil);

up(&paper);

}

  

Jeśli oba procesy ruszą na raz może zdarzyć się sytuacja, w której proces A zablokuje zasób “pencil” a proces B zablokuje zasób “paper” a następnie będą wiecznie czekać na zwolnienie zasobów.

  

Określono pewne warunki konieczne jakie muszą zaistnieć na raz, żeby powstało zakleszczenie:

- Zasób może być przypisany do wyłącznie jednego procesu
    
- Procesy mogą żądać więcej niż jednego zasobu
    
- Brak możliwości wywłaszczenia zasobu
    
- Występuje cykliczny łańcuch oczekiwania na zasób
    

  

Potencjalne sytuacje zakleszczenia można także przedstawić graficznie za pomocą grafów skierowanych.

## Algorytm wykrywania zakleszczeń

przechodzenie po drzewie zasobów aby wykryć cykl. Algorytm powyższy sprawdza się tylko dla pojedynczych kopii zasobu dostępnych w systemie. Dla zasobów które występują w większej liczbie należy zastosować algorytm z listą istniejących zasobów i dostępnych zasobów oraz z macierzą bieżących alokacji i macierzą żądań. Algorytm sprawdza które procesy można przeprowadzić do końca i oznacza je. Pozostałe nieoznaczone procesy będą zakleszczone. 

## Usuwanie zakleszczeń

Poprzez wywłaszczenie. Czyli zasób jest odbierany procesowi w celu przekazania go innemu. 

Cofnięcie operacji. Systemy operacyjne okresowo rejestrują stan procesów i zasobów. Taki zapis nazywany jest punktem kontrolnym. Dzięki temu można ten stan cofnąć do pewnego momentu i inaczej rozdysponować zasoby.

Usuwanie zakleszczeń przez zabijanie procesów. Zabicie procesu, który posiada zasoby potrzebne dla innych procesów uwolni potrzebne zasoby. Należy jednak ostrożnie wybrać "ofiarę" tak aby nie doszło do uszkodzenia danych bądź nośników.

## Unikanie zakleszczeń

śledzenie trajektorii zasobów - wymaga wiedzy o tym kiedy jaki proces będzie używał danego zasobu i przez jaki czas. 

Zapobieganie zakleszczeniom.

atak na wzajemne wykluczenie - buforowanie wszystkiego

atak na wstrzymanie i oczekiwanie - żądanie wszystkich zasobów z góry

atak na brak wywłaszczenia - odbieranie zasobów

atak na cykliczne powtarzania - numeracja zasobów i uporządkowanie

  

## Blokowanie dwufazowe 

Proces najpierw blokuje wszystkie potrzebne zasoby a potem ich używa. Plusem jest to, że w fazie blokowania proces nie wykona żadnych działań a co za tym idzie nie dojdzie do uszkodzenia danych.

Zakleszczenia komunikacyjne. 

Timeouts

Przykładem zakleszczenia komunikacyjnego może być niemożność przesłania pakietów w cyklu routerów pomiędzy sobą z uwagi na przepełnienie bufora w każdym z nich.

## Uwięzienia

Uwięzienie to sytuacja w której procesy żądają równolegle w pętli zasobów podejmują próby ich przejęcia i rezygnację z powodu braku kompletu zasobów potrzebnych do ukończenia procesu.  

## Zagłodzenia

Procesy walczą o jeden zasób. Gdyby system przydzielał zasób w pierwszej kolejności procesom, które potrzebują go na najkrótszy czas. Długi proces mógłby nigdy by się nie doczekać swojej kolejki. Niekiedy stosuje się algorytm FCFS który po obsłużeniu najmniejszego procesu przydziela zasób procesowi, który czeka najdłużej.

  
  
**