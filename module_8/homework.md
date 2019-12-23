# Moduł 8 - praca domowa

Wracając do Twojej aplikacji, albo twojej „wirtualnej” aplikacji, zastanów się:

1. Do czego przydałby się CrobJob?

- cykliczne wykonywanie kopi zapasowej bazy danych (PostgreSQL, Cassandra, ElasticSearch)
- cykliczne czyszczenie cache w redis
- cyklicznie czyszczenie indeksów w ElasticSeach np. od logów albo metryk tak aby statystyki były np. tylko z tygodnia/miesiąca

Aplikacja posiada usługę zadań w tle wraz z planowaniem więc wszystkie cykliczne generowanie raportów itp. jest obsługiwane w tym narzędziu i nie ma potrzeby przenoszenia tego mechanizmy CronJobów w K8s.


2. Czy widzisz bezpośrednie (bez CronJob) użycie Job?

Tak. Zwykły `Job` może być użyty np. jako post/pre deployment command gdzie wykonują się pewne operacje w aplikacji, koniecznie do deployu, ale mające się wykonać tylko raz niezależnie od ilości replik podów np.

- generowanie plików z tłumaczeniami i aktualizacja ich na współdzielonym zasobie/chmurze
- zbieranie plików statycznych i kopiowanie ich do współdzielonego zasobu
- wykonywanie specyficznych komend po deplyu np. synchronizacja namespace w Cassandra