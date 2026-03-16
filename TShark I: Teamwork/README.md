
---

## Tshark Challenge 1 – Teamwork (CTF Report)

# 1. Cel zadania

Celem zadania było przeanalizowanie przechwyconego ruchu sieciowego w pliku **pcap** i znalezienie podejrzanych artefaktów, które mogą wskazywać na złośliwą aktywność.

Scenariusz zakładał, że zespół bezpieczeństwa wykrył podejrzaną domenę i przekazał sprawę do analizy. Zadaniem było sprawdzenie ruchu sieciowego oraz znalezienie informacji takich jak:

* podejrzana domena
* adres IP
* podszywana usługa
* adres email użyty w ataku

Do analizy użyto narzędzia **TShark**, które jest wersją CLI programu **Wireshark**.
Plik do analizy:

```
teamwork.pcap
```

---

# 2. Wstępna analiza pliku PCAP

Najpierw otworzyłam plik pcap w TShark, aby zobaczyć jakie pakiety znajdują się w przechwyconym ruchu.

```bash
tshark -r teamwork.pcap --color
```

### Wyjaśnienie komendy

* `tshark` – uruchamia program do analizy ruchu sieciowego
* `-r` – oznacza **read**, czyli wczytanie pliku PCAP
* `teamwork.pcap` – plik z przechwyconym ruchem
* `--color` – koloruje output w terminalu, dzięki czemu łatwiej odróżnić protokoły

Po uruchomieniu komendy zobaczyłam, że plik zawiera **793 pakiety**, więc ręczna analiza wszystkich byłaby trudna.

---

# 3. Sprawdzenie statystyk protokołów

Następnie sprawdziłam jakie protokoły występują w ruchu sieciowym.

```bash
tshark -r teamwork.pcap -z io,phs -q
```

### Wyjaśnienie komendy

* `-z io,phs` – pokazuje **Protocol Hierarchy Statistics**, czyli statystyki protokołów
* `-q` – tryb quiet (wyłącza zbędne informacje)

Dzięki temu można zobaczyć np.:

* ile pakietów DNS
* ile HTTP
* ile TCP

W tym przypadku zauważyłam, że w ruchu znajduje się **kilkadziesiąt zapytań DNS**, więc warto było je przeanalizować dokładniej. ([Jasper Alblas][1])

---

# 4. Analiza zapytań DNS

Zapytań DNS można użyć do znalezienia podejrzanych domen.

```bash
tshark -r teamwork.pcap -Y 'dns.qry.type == 1'
```

### Wyjaśnienie komendy

* `-Y` – filtr wyświetlania (display filter)
* `dns.qry.type == 1` – filtr pokazujący **zapytania DNS typu A**, czyli zapytania o adres IP domeny

Dzięki temu można zobaczyć wszystkie domeny, do których komputer próbował się połączyć.

Podczas analizy zauważyłam bardzo dziwnie wyglądającą domenę:

```
www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com
```

Wygląda ona jak **próba podszycia się pod PayPal**, co jest typowe dla phishingu.

---

# 5. Znalezienie domen DNS w pliku

Aby zebrać wszystkie domeny z ruchu DNS użyłam polecenia:

```bash
tshark -r teamwork.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r
```

### Wyjaśnienie komendy

**tshark część**

* `-T fields` – pokazuje tylko wybrane pola zamiast całego pakietu
* `-e dns.qry.name` – wyświetla nazwę domeny z zapytania DNS

**pipe w Linux (`|`)**

Przekazuje wynik jednej komendy do kolejnej.

**pozostałe polecenia**

* `awk NF` – usuwa puste linie
* `sort -r` – sortuje wyniki
* `uniq -c` – liczy wystąpienia
* `sort -r` – sortuje od największej liczby

Dzięki temu widać, która domena pojawia się najczęściej.

Najczęściej pojawiała się domena:

```
www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com
```

---

# 6. Sprawdzenie domeny w VirusTotal

Podejrzaną domenę sprawdziłam w serwisie **VirusTotal**, który analizuje adresy URL i pliki pod kątem malware.

Wynik:

* domena została oznaczona jako **malicious**
* pierwsze zgłoszenie:

```
2017-04-17 22:52:53 UTC
```

Domena podszywała się pod usługę:

```
PayPal
```

---

# 7. Defang URL

W raportach bezpieczeństwa często używa się **defang**, aby link nie był klikalny.

Przykład:

```
www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com
```

po defang:

```
www[.]paypal[.]com4uswebappsresetaccountrecovery[.]timeseaways[.]com
```

---

# 8. Znalezienie adresu email

Następnie próbowałam znaleźć dane przesyłane przez HTTP.

```bash
tshark -r teamwork.pcap -Y "http contains gmail.com"
```

### Wyjaśnienie

* `http contains gmail.com` – filtr pokazujący pakiety HTTP zawierające tekst gmail.com

Aby zobaczyć szczegóły pakietu użyłam:

```bash
tshark -r teamwork.pcap -Y "frame contains login.php" -V
```

### Wyjaśnienie

* `frame contains` – szuka tekstu w pakiecie
* `login.php` – wskazuje formularz logowania
* `-V` – pokazuje **pełne szczegóły pakietu (verbose)** ([Jasper Alblas][2])

W danych pakietu znalazłam adres email użyty w formularzu logowania.

Email (defang):

```
johnny5alive[at]gmail[.]com
```

---

# 9. Podsumowanie

Podczas analizy ruchu sieciowego udało się znaleźć kilka artefaktów wskazujących na phishing.

### Znalezione informacje

| Element                      | Wartość                                                              |
| ---------------------------- | -------------------------------------------------------------------- |
| Podejrzana domena            | www[.]paypal[.]com4uswebappsresetaccountrecovery[.]timeseaways[.]com |
| Podszywana usługa            | PayPal                                                               |
| Data zgłoszenia w VirusTotal | 2017-04-17                                                           |
| Email użyty w formularzu     | johnny5alive[at]gmail[.]com                                          |

### Wnioski

Ruch sieciowy wskazuje na próbę **phishingu podszywającego się pod PayPal**. Użytkownik prawdopodobnie został przekierowany na fałszywą stronę logowania, gdzie jego dane mogły zostać przechwycone.

---




