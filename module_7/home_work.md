# Moduł 7 - praca domowa

Bazując na wiedzy z poprzednich modułów i pracach domowych możesz teraz przeanalizować, w jaki sposób udostępniać aplikacje. Jeżeli nie masz swojego systemu, to wykonaj mentalne ćwiczenie „jakie wartości fajnie by było mieć”.

1. Czy wykorzystasz Ingress zamiast wybranych metod publikacji z pracy domowej z modułu 5?

Tak, Ingress będzie wykorzystywany jako metoda publikacji oraz konfiguracji usług.

2. Czy będzie potrzebował różnych instalacji Ingress Controller?

Nie

3. Jaki reguły Ingress wykorzystasz?

Zostanie wykorzystany `virtual host` oraz `fanout`:

- `virtual host`, ponieważ aplikacja wykorzystuje kilka hostów
- `fanout`, ponieważ istnieje potrzeba mapowania niektórych ścieżek

4. Jakie typy certyfikatów wykorzystasz?

Aplikacja wykorzystuje certyfikacja wystawiony przez zewnętrzną organizację, więc certyfikat zostanie zaimportowany jako `secret`. Dla środowisk test/stage/dev zostanie wykorzystany certyfikat Let’s Encrypt zarządzany z pomocą `cert-managera`.