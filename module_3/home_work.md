# Moduł 3 - praca domowa

Skoro już wiesz jak działa przekazywanie konfiguracji do aplikacji w K8s, zastanów się i opisz (najlepiej na Slack), co w swojej aplikacji byś gdzie umieścił. Do dyspozycji masz:

- Zmienne środowiskowe
- ConfigMap
- Secrets
- Zewnętrzne narzędzia integrujące się z K8s

Aplikacja jest obecnie konfigurowalna jedynie przez zmienne środowiskowe oraz przekazywany jest szereg parametrów które można podzielić na następujące kategorie:

## Dane dostępowe do baz danych
Dane dostępowe do każdej bazy danych wykorzystywanej przez aplikacje:
- PostgreSQL
- Cassandra
- Redis

Konfiguracja pobierania z: `HashiCorp Vault`

## Konfiguracja monitoringu
Przykładowe konfiguracje:
- hosty monitora celery per środowisko
- Sentry DNS dla każdego środowiska

Konfiguracja pobierania z: `ConfigMap`

## Konfiguracja Celery
Przykładowe konfiguracje:
- ilość pamięci dla workera
- tryb pobierania zadań z kolejki
- definicje dostępnych kolejek oraz routing
- polityka wznawiania zadań
- ttl

Konfiguracja pobierania z: `ConfigMap`

## Dane autoryzacyjne zewnętrznych usług
Przykładowe dostępy do:
- AWS
- Azure
- Firebase

Konfiguracja pobierania z: `HashiCorp Vault`

## Tokeny dostępowe wewnętrznych usług
Aplikacja posiada tokeny JWT dla wewnętrznych usług które mogą komunikować się przez REST API z aplikacją i np. synchronizować/pobierać dane.

Konfiguracja pobierania z: `Secrets`

## Typowa konfiguracja aplikacji
Przykładowe konfiguracje:
- ścieżki do katalogów
- feature flagi per środowisko
- konfiguracja CORS
- kanału cache w aplikacji (mapowanie kanałów redisa per cache)
- tryb debug
- konfiguracja email (host, mail, auth, subject)

Konfiguracja pobierania z: `ConfigMap` (autoryzacja do konta mailowego z `Secrets`)