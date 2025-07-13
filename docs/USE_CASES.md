# Use Cases & Anwendungsfälle

**Dieses Dokument beschreibt die wichtigsten Anwendungsfälle (Use Cases) für Kunden und Administratoren der Honigschätze E-Commerce-Anwendung für die Familienimkerei.**

## 🛒 **Kunden-Workflows**

### 1. **Einkaufsprozess**
- **Produktbrowsing**: Shop durchsuchen, filtern, nach Honigsorten suchen (mobile-optimiert)
- **Warenkorb**: Produkte hinzufügen, Mengen anpassen, Rabattlogik (touch-optimiert)
- **Checkout**: Kundendaten eingeben, Zahlungs- und Versandart wählen (mobile-Formulare)
- **Validierung**: Umfassende Eingabevalidierung (E-Mail, Telefon, Adresse)
- **Bestellung**: Automatische Bestellbestätigung per E-Mail
- **Manuelle Zahlungsabwicklung**: Zahlung wird nach Bestellung manuell abgewickelt
- **Tracking**: Bestellübersicht mit E-Mail-Verifikation
- **Mobile-Erlebnis**: Touch-optimierte Navigation, responsive Design, iOS-Zoom-Verhinderung

### 2. **Kontakt & Support**
- **Kontaktformular**: Nachrichten an den Verkäufer senden (mobile-optimiert)
- **Validierung**: Eingabevalidierung und Spam-Schutz
- **Bestätigung**: Automatische Bestätigung an Kunde und Verkäufer
- **Mobile-Formulare**: Touch-freundliche Eingabefelder, größere Buttons

### 3. **Mehrsprachigkeit**
- **Sprachauswahl**: Dynamische Umschaltung zwischen Deutsch/Ungarisch (mobile-optimiert)
- **Lokalisierung**: Alle Inhalte, Formulare und E-Mails übersetzt (Deutsch vollständig, Ungarisch technisch vorbereitet)
- **SEO-optimiert**: Sprachspezifische URLs und Meta-Tags
- **Mobile-Navigation**: Touch-freundliche Sprachumschaltung im Header

## 🔧 **Admin-Workflows**

### 4. **Produktverwaltung**
- **Honigsorten**: Neue Sorten anlegen, Preise und Lagerbestand verwalten
- **Eimer-Größen**: Verschiedene Verpackungsgrößen konfigurieren
- **Kategorien**: Produktkategorien organisieren und verwalten
- **Automatisierung**: Produkte werden automatisch für alle Eimer-Größen erstellt

### 5. **Bestellungsverwaltung**
- **Bestellübersicht**: Alle Bestellungen mit Status und Details
- **Status-Management**: Bestellungen bearbeiten und Status ändern
- **Kundendaten**: Kundenprofile und Bestellhistorie einsehen
- **Lagerverwaltung**: Automatische Bestandsführung und Warnungen

### 6. **System-Monitoring**
- **Aktivitäts-Logging**: Benutzeraktivitäten und Systemereignisse
- **Django-Logging**: Error-Logging und User-Activity-Logging
- **Fehler-Tracking**: Django-Fehler und Exception-Handling
- **Manuelle Backups**: Einfache Datenbank-Backups bei Bedarf

## 🎯 **Besondere Features**

### 7. **Rabattlogik**
- **Mengenrabatt**: Automatischer 1€ Rabatt pro kg Honig ab 7kg
- **Preisberechnung**: Intelligente Preisberechnung basierend auf Eimer-Größe
- **Transparenz**: Rabatte werden in Warenkorb, Bestellung und E-Mails angezeigt

### 8. **Lagerverwaltung**
- **Automatische Bestandsführung**: Honig und Eimer werden automatisch abgezogen
- **Verfügbarkeitsprüfung**: Produkte sind nur verfügbar, wenn Honig vorrätig (Eimer-Mangel wird protokolliert, verhindert aber nicht die Bestellung)
- **Eimer-Mangel-Benachrichtigung**: Automatische E-Mail bei Eimer-Engpässen
- **Transaktionssicherheit**: Lagerbestand wird atomar aktualisiert

### 9. **E-Mail-System**
- **Bestellbestätigungen**: Automatische E-Mails an Kunde und Verkäufer
- **HTML-Templates**: Professionelle E-Mail-Templates
- **Eimer-Mangel-Benachrichtigungen**: Automatische E-Mails bei Eimer-Engpässen

### 10. **Mobile-Optimierung**
- **Responsive Design**: Mobile-First CSS mit Media Queries für alle Bildschirmgrößen
- **Touch-Interaktionen**: Größere Touch-Targets (44px), Touch-Feedback, doppelte Klicks verhindert
- **Performance**: Lazy Loading, debounced Events, iOS-Zoom-Verhinderung
- **Formulare**: Touch-freundliche Eingabefelder, automatisches Scrollen zu Fehlern
- **Navigation**: Mobile-optimierte Header, responsive Produktkarten, Touch-optimierte Buttons