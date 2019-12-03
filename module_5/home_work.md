# Moduł 5 - praca domowa

Bazując na wiedzy z poprzednich modułów możesz teraz przeanalizować w jaki sposób udostępniać aplikacje sieciowo w klastrze i po za klastrem. Jeżeli nie masz swojego systemu, to wykonaj mentalne ćwiczenie „jakie wartości fajnie by było mieć.

1. Czy będzie korzystać z Service Discovery? Jeśli tak to jakiego? Dlaczego?

Aplikacja w klastrze K8s będzie uruchomiona na trzech środowiskach:

- produkcja
- stage
- test/dev

Dla każdego środowiska zostanie utworzony osobny namespace w celu separacji usług.

Wykorzystany zostanie wewnętrzny Service Discovery K8s (etcd) ponieważ jest wystarczający oraz spełnia wszystkie wymagania aplikacji.

Każde środowisko będzie dostępne (z zewnątrz) przez główny load balancer.

2. Jakie typy serwisu wykorzysta do Services wykorzystasz? Dlaczego?

- Dostęp z zewnątrz klastra będzie obsługiwany przez `Ingress` wiec serwis typu `LoadBalancer` nie będzie używany.

- Wszystkie usługi aplikacji będą wystawione na serwisie z typem `ClusterIP` ponieważ nie ma potrzeby wystawiania ich po za klaster.

- Zostanie wykorzystany serwis typu `ExternalName` w celu mapowania usług zewnętrznych np. PostgreSQL lub Redis (as a Service) wewnątrz klastra aby nie było potrzeby przekazywania adresu/konfiguracji do każdego serwisu aplikacji.

3. Do publikacji aplikacji użyjesz NodePort czy LoadBalancer? Dlaczego?

Do publikacji aplikacji będzie użyty `Ingress` więc żaden z tych typów serwisów nie będzie wykorzystywany bezpośrednio. W przypadku braku ingressa, wykorzystany był by `LoadBalancer` w celu integracji z load balancerem w usłudze chmurowej (Azure).