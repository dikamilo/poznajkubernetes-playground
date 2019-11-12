# Moduł 2 - praca domowa

## Czy będzie wymagać użycia kontenerów typu Init? jeśli tak to dlaczego?

Prawdopodobnie tak, do wykonywania pre/post deploymentu:
- zastosowanie migracji na bazie danych bazy danych
- generowanie plików językowych dla i18n oraz kopiowanie ich do współdzielonego wolumenu
- `collectstatic` (zbieranie nowych plików statycznych wykorzystywanych po stronie backendu - szablony maili, pliki statyczne panelu admina itp. -  i kopiowanie ich do współdzielonego wolumenu)
- uruchamianie innych komend wymaganych dla nowej wersji aplikacji

Jednak nie jestem pewny czy kontenery Init się sprawdzą gdy Pod będzie skalowany na wiele instancji, ponieważ wymienione wyżej kroki muszą się wykonać jedynie raz w trakcie deploymentu nowej wersji.

## Czy będzie wymagać użycia wolumenów emptyDir?

Nie, ponieważ aplikacja potrzebuje wolumenów które będą współdzielone między różnymi procesami (Pod'ami) oraz zapisywane dane muszą być trwałe. 

## Czy posiada endpointy mogące służyć jako probe dla Health checks? Jeśli nie to czy można je zaimplementować bez problemu?

Aplikacja backendowa posiada endpoint `/health` który może być wykorzystany jako `liveness probe`. Endpoint dla `readiness probe` nie istnieje ale może być dodany.

W przypadku Celery (task queue/workers) nie ma endpointów dla `liveness probe` oraz `readiness probe`, ale zamiast tego można wykorzystać wewnętrzne API Celery i odwołać się przez `command` aby sprawdzić status danego workera (wymaga to przekazania nazwy hosta workera w komendzie):

```
celery inspect ping -A apps -d celery@$HOSTNAME
```

## Jaka klasa QoS będzie do niej pasować najlepiej? Dlaczego?

Zarówno w przypadku backendu jak i Celery nie potrzebujemy klasy `Guaranteed`. `Burstable` jest wystarczające a w przypadku Celery wręcz porządne. Workery przetwarzające zadania potrafią alokować pamięć na potrzeby pojedyńczego zadania a następnie ją zwalniać więc `Burstable` sprawdzi się tutaj idealnie. Podobnie w przypadku backendu uruchomionym przez `uwsgi` które potrafi uruchamiać tyle procesów oraz wątków ile jest potrzebne dla danej ilości połączeń. Nie potrzebujemy alokować "sztywnych" zasobów.

## Czy będą potrzebne postStart i preStop hook? Jeśli tak to dlaczego?

Obecnie nie widzę potrzeby aby `postStart` był potrzebny w aplikacji. Hook `preStop` jest potrzebny do poprawnego zamknięcia workera Celery z racji tego że Celery poprawnie zakończy swoją pracę gdy przetworzy obecnie obsługiwane zadanie (co jest możliwe przez wykonanie polecenia `stopwait` na workerze). Czysty `SIG_TERM` spowoduje że obecnie przetwarzane zadanie zostanie przerwane i utracone.

## Czy aplikacja obsługuje `SIG_TERM` poprawnie? Jeśli nie to jak można to obejść?

Tak, aplikacja (backend) poprawnie obsługuje `SIG_TERM`.

## Czy domyślny czas na zamknięcie aplikacji w Pod jest wystarczający?

W przypadku backendu domyślna konfiguracja jest wystarczająca. W przypadku Celery wykorzystany zostanie dodatkowo hook `preStop` jednak domyślny czas musi być zwiększony ponieważ wywołanie `stopwait` może trwać kilka sekund ale również kilka minut w zależności od zadania (co może przekroczyć wartość ustawioną w `terminationGracePeriodSeconds`). Potrzebne jest oszacowanie czasu jaki jest wymagany dla workera lub ustawienie odpowiednio dużego limitu odgórnie.