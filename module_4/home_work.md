# Moduł 4 - praca domowa

Bazując na swoim systemie, zastanów nad listą rzeczy poniżej i podziel się z nami przemyśleniami na Slacku. Jeżeli nie masz swojego systemu, to wykonaj mentalne ćwiczenie „jakie wartości fajnie by było mieć”.


1. Na jakie i ile etykiet można by rozsądnie podzielić Twój system i dlaczego tak

- etykiety określające środowisko: `env` = `production`, `stage`, `test` aby rozróżniać które Pod'y są do którego środowiska

- etykiety określające "warstwy/tier": `tier` = `frontend`, `backend`, `database`, `cache`, `monitoring`, `documentation` itp.

- etykiety określające jaki typ kolejek obsługują workery: `queue` = `notifications`, `permissions`, `reports` itp. aby w łatwy sposób filtrować Pody z workerami które obsługują dane kolejki

2. Jakie etykiety fajnie by było mieć by móc lepiej i sprawniej zarządzać grupą zasobów?

Najsensowniejszymi etykietami wydaje się etykieta `env` określająca środowisko oraz `queue` która pozwala grupować typy obsługiwanych kolejek przez workery. 

3. Jakie adnotacje warto by wprowadzić i co one powinny zawierać?


- `description` opisująca za co jest odpowiedzialny dany Pod
- `repository` url do repozytorium Git'a z kodem z którego zbudowany jest używany obraz
- `contact` adres email lub imię i nazwisko osoby odpowiedzialnej ze dany zasób

W przypadku workerów:
- adnotacje konfigurujące Celery danego workera
- adnotacje określające kolejkę lub kolejki obsługiwane przez danego workera

W przypadku ingressa:
- adnotacje konfigurujące nginxa

W przypadku monitoringu:
- adnotacje konfigurujące prometheus'a