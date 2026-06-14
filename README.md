# Opis projektu końcowego

## **Projekt uruchamia środowisko analizy jakości sekwencji FASTQ przy użyciu `FastQC` oraz udostępnia wyniki przez serwer `nginx`.**
---
Podczas pierwszego uruchomienia Docker Compose automatycznie pobiera obraz nginx:latest. Projekt opiera się na trzech głównych plikach konfiguracyjnych: docker-compose.yml, Dockerfile oraz nginx.conf.

1. Dockerfile definiuje obraz dla kontenera `FastQC` bazujący na `ubuntu 22.04`, w którym instalowane są niezbędne zależności takie jak Java 11, wget, unzip oraz perl. Pobierane jest narzędzie `FastQC` w wersji 0.12.1, które jest rozpakowywane, nadawane są mu uprawnienia wykonywania i instalowane w systemie kontenera. Plik wykonywalny jest dodawany poprzez link symboliczny. Ustawiane są również zmienne środowiskowe: JAVA_HOME oraz CLASSPATH. Entrypoint stanowi `fastqc` a domyślna komenda przyjmuje `--help`.
---
2. docker-compose.yml definiuje dwa kontenery (1) fastqc i (2) nginx. Kontener fastqc wykorzystuje zbudowany obraz z Dockerfile, montuje wolumen fastqc_results do katalogu /results oraz katalog input jako źródło danych FASTQ w trybie tylko do odczytu. Po uruchomieniu kontener wykonuje analizę pliku FASTQ wskazanego w komendzie i zapisuje wyniki do wolumenu. Z kolei kontener nginx wykorzystuje obraz `nginx:latest`. Mapuje ten sam wolumen fastqc_results do katalogu /usr/share/nginx/html i wystawia usługę na porcie 8080, dzięki czemu wyniki analizy mogą być przeglądane w przeglądarce pod adresem `localhost:8080`. Oba kontenery znajdują się w sieci `fastqc_network`. Wyniki analizy są przekazywane pomiędzy kontenerami za pomocą współdzielonego wolumenu fastqc_results.
---
3. Plik nginx.conf definiuje konfigurację dla `NGINX`, ustawiając katalog główny na `/usr/share/nginx/html` i określając index jako raport FastQC (html). Mechanizm try_files pozwala na poprawne wyświetlanie plików HTML raportu.
---
Aby uruchomić projekt, należy umieścić plik FASTQ (np. SRR8786200_1.fastq.gz) w katalogu input. W terminalu należy wykonać komendę `docker compose up --build`, co spowoduje zbudowanie obrazu `FastQC`, uruchomienie analizy i wygenerowanie raportu HTML dostępnego pod adresem localhost:8080 z poziomu przeglądarki. Raport HTML jest generowany przez FastQC i zapisywany we współdzielonym wolumenie, a następnie udostępniany przez serwer NGINX. Aby zakończyć działanie kontenerów wykonaj komendę `docker compose down`.
