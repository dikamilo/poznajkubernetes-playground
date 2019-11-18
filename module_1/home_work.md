# Moduł 1 - praca domowa

Aplikacja produkcyjna w Pythonie + SPA stack:
- django
- celery
- postgresql
- cassandra
- redis
- rabbitmq
- elasticsearch
- reactjs
- nginx

Dodatkowo monitoring oraz raw logi self-hosted:
- elk
- prometheus
- grafana

Aplikacja jest zintegrowana również z Sentry (na każdym środowisku).

Obecnie aplikacja jest wdrożona na VM'kach, ale jesteśmy w trakcie przygotowań przenosin do Azure K8s.


## Jakie praktyki z 12Factor mój projekt aktualnie spełnia?

### Codebase
Projekt przechowywany w kilku repozytoriach na bitbucket: 

- backend
- frontend
- docker conf
- ansible conf pod deploy/provision)

### Zależności
Zależności zgromadzone w jednym miejscu z wymuszoną wersją.

Na frontendzie:
- `package.json` (dependencies oraz devDependencies)

Na backendzie:
- `requirements.txt` - zależności projektowe
- `requirements.dev.txt` - zależności dodatkowe potrzebne do testów oraz lokalnego 
developmentu (framework testowy, mocki, fabryki, lintery itp.)

### Konfiguracja
Konfiguracja z env'a + konfiguracja pod środowiska wdrożeniowa bezpośrednio w Ansible (wrażliwe dane zaszyfrowane 
ansible vault).

### Usługi wspierające
Produkcyjne używany AWS3, na środowiskach testowych S3 nie jest używane, pliki są przechowywane lokalnie na serwerach. 
Na środowisku local dev używamy Localstack'a do emulowania usług AWS.

### Buduj, publikuj, uruchamiaj.
- Budowanie razem z testami w Continuous Integration
- Publikowanie i uruchamianie przez ansible

### Procesy
Aplikacje są uruchamiane jako usługi systemd. Backend jest uruchamiany przez Uwsgi który sam zarządza procesami. 
Podobnie Celery samo zarządza ilością procesów w zależności od konfiguracji oraz ilości obsługiwanych kolejek zadań.

### Przydzielanie portów
Wszystkie aplikacje są udostępnianie przez porty, konfiguracja jest zdefiniowana w ansible.

### Współbieżność
Współbieżność jest zapewniona. Skalowany jest backend (kilka maszyn, osobne instancje, mapowanie w nginx w 
proxy pass i upstream) oraz celery (kilka niezależnych maszyn).

### Zbywalność
Celery może być zrestartowane, tylko jeżeli przetworzy swoje zadania, backend może być restartowany niezależnie, jest 
replikowany na kilku serwerach. Praktycznie każda część aplikacji może być zrestartowana/zatrzymana oraz 
uruchomiona przez Systemd.

### Jednolitość środowisk
Środowiska są niemal 1:1, wyjątkiem jest wykorzystanie S3 tylko na produkcji oraz emulacja go na środowisku local dev 
przez Localstack'a. Moglibyśmy popracować nad ujednoliceniem środowisk, aby S3 było wszędzie.

### Logi
Logujemy logi produkcyjne do ELK-a. Aplikacja jest zintegrowana również z Sentry (na każdym środowisku, backend 
oraz frontend), gdzie logujemy wszystkie wyjątki.

### Zarządzanie aplikacją
Jest zastosowane, zarządzanie głównie przez Ansible. Wyjątkiem jest jedynie przywracanie backupu bazy na 
środowiskach, co jest wykonywane ręcznie przez SSH.

## Czy jest sens spełniać wszystkie praktyki 12Factor w aktualnym projekcie?
Tak

## Czy architektura rozwiązania umożliwi konteneryzację?
Tak. Mimo tego, że produkcyjnie aplikacja nie jest oparta o kontenery, lokalne środowisko developerskie jest 
operate w pełni o Docker'a i docker-compose.

## Dlaczego tak?
Elementy aplikacji łatwo przenieść na kontenery oraz konfiguracja jest z env'a co jest również pomocne 
przy konteneryzacji.
