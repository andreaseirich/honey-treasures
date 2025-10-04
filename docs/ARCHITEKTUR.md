# Architekturübersicht

**Dieses Dokument gibt einen Überblick über die technische Architektur des Honigschätze E-Commerce-Systems für die Familienimkerei, die wichtigsten Komponenten, deren Zusammenspiel und besondere technische Lösungen.**

## Komponenten
- **Backend:** Django (Python), Model-View-Template (MVT)
- **Frontend:** HTML5, CSS3, JavaScript, Mobile-First Responsive Design
- **Templates:** Django Templates mit Blöcken
- **Internationalisierung:** Django ModelTranslation, Sprachumschaltung per URL
- **Datenbank:** SQLite für Familienbetrieb
- **E-Mail:** SMTP für Bestellbestätigungen
- **Zahlung:** Manuelle Zahlungsabwicklung (Überweisung, Barzahlung, PayPal)
- **Admin-System:** Erweiterte rollenbasierte Benutzerverwaltung mit Analytics Dashboard
- **Middleware:** Spezialisierte Middleware für Wartungsmodus und Berechtigungen
- **Mobile-Optimierung:** Touch-optimierte Benutzeroberfläche, Responsive CSS, Performance-Optimierungen
- **Logging:** Benutzeraktivitäten und Geräte-Labels werden ausschließlich in der Datenbank gespeichert (keine Logdateien)

## Systemübersicht

```mermaid
graph TD
    U[Kunde] -->|"Produkte ansehen"| S(Shop)
    S --> W(Warenkorb)
    W --> C(Checkout)
    C --> V{Validierung}
    V -- OK --> B[Bestellung speichern]
    B --> L[Lager aktualisieren]
    B --> E[E-Mail an Kunde & Admin]
    B --> O[Bestellübersicht]
    V -- Fehler --> C
    
    A[Admin] -->|"Verwaltung"| AS[Admin System]
    AS --> RB[Rollenbasierte Berechtigungen]
    AS --> AD[Analytics Dashboard]
    AS --> UM[Benutzerverwaltung]
    AS --> SE[System-Einstellungen]
    AS --> WM[Wartungsmodus]
    
    AS --> DB[(Datenbank)]
    S --> DB
    W --> DB
    C --> DB
    B --> DB
    L --> DB
    RB --> DB
    AD --> DB
```

## Besonderheiten
- Lagerverwaltung für Honig und Eimer
- Rabattlogik ab 7kg Honig
- E-Mail-Benachrichtigungen für Kunden und Familienbetrieb
- Mehrsprachigkeit (Deutsch vollständig, Ungarisch technisch vorbereitet)
- DSGVO-konforme Datenschutz- und Cookie-Hinweise
- Mobile-First Responsive Design für alle Bildschirmgrößen
- Touch-optimierte Benutzeroberfläche mit 44px Touch-Targets
- iOS-Zoom-Verhinderung und mobile Performance-Optimierungen
- Live-System unter https://honey-treasures.com
- **Erweiterte Admin-Architektur**: Rollenbasierte Benutzerverwaltung mit granularen Berechtigungen
- **Analytics Dashboard**: Echtzeit-Monitoring mit interaktiven Charts und Export-Funktionen
- **Middleware-Architektur**: Spezialisierte Middleware für Wartungsmodus und Berechtigungsprüfung
- **System-Einstellungen**: Zentrale Konfiguration aller System-Parameter
- **Benutzeraktivitäten und Geräte-Labels werden zur Sicherheit und Nachvollziehbarkeit ausschließlich in der Datenbank gespeichert**