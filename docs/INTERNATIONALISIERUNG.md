# Internationalisierung (i18n)

**Dieses Dokument erklärt die umfassende Implementierung der Mehrsprachigkeit und Internationalisierung im Honigschätze-Projekt.**

## 🌍 **Unterstützte Sprachen**

### **Aktueller Status**
- **Deutsch (de)**: ✅ Vollständig implementiert und übersetzt
- **Ungarisch (hu)**: 🔄 Technisch vorbereitet, Übersetzung noch nicht implementiert

**Hinweis**: Die ungarische Version ist technisch vorbereitet, aber die Übersetzung ist noch nicht implementiert.

### **Erweiterbarkeit**
- **Modulare Architektur**: Einfache Hinzufügung weiterer Sprachen
- **Fallback-Mechanismus**: Automatischer Fallback auf Deutsch bei fehlenden Übersetzungen
- **SEO-Optimierung**: Sprachspezifische URLs und Meta-Tags

## 🛠 **Technische Umsetzung**

### **Django ModelTranslation**
**Übersetzbare Felder:**
- **Produkt**: produktname, beschreibung
- **Honigsorte**: name
- **Kategorie**: name

**Technische Umsetzung:**
- Django ModelTranslation für automatische Feldverwaltung
- Übersetzbare Felder sind in der Datenbank implementiert
- Admin-Interface unterstützt direkte Bearbeitung mehrsprachiger Inhalte
- Ungarische Version technisch vorbereitet, Übersetzung noch nicht vollständig

### **Template-Internationalisierung**
- **{% trans %}**: Einzelne Texte übersetzen
- **{% blocktrans %}**: Komplexe Texte mit Variablen
- **{% get_current_language %}**: Aktuelle Sprache abrufen
- **{% get_available_languages %}**: Verfügbare Sprachen anzeigen

### **URL-Struktur**
```
/de/          # Deutsche Version
/hu/          # Ungarische Version
/de/produkte/ # Deutsche Produktseite
/hu/produkte/ # Ungarische Produktseite
```

## 🎯 **Benutzerfreundlichkeit**

### **Sprachumschaltung**
- **Dropdown-Menü**: Intuitive Sprachauswahl im Header
- **URL-basiert**: Direkte Navigation zu Sprachversionen
- **JavaScript-basierte Weiterleitung**: Automatische URL-Anpassung bei Sprachwechsel

### **SEO-Optimierung**
- **Sprachspezifische URLs**: Klare URL-Struktur pro Sprache
- **HTML lang-Attribut**: Korrekte Sprachkennzeichnung im HTML-Tag
- **Basis-Sitemap**: Standard-Sitemap für Produkte und statische Seiten

## 📧 **E-Mail-Internationalisierung**

### **Template-System**
- **E-Mail-Templates**: Einsprachige E-Mail-Benachrichtigungen (Mehrsprachigkeit in Planung)

**Hinweis**: Die ungarische Version ist technisch vorbereitet, aber die Übersetzung ist noch nicht implementiert. E-Mail-Benachrichtigungen werden derzeit nur in Deutsch versendet.

## 🔧 **Entwickler-Features**

### **Übersetzungs-Management**
- **Django Admin**: Übersetzbare Felder direkt im Admin bearbeiten
- **Django i18n**: Standard Django-Übersetzungsfunktionen
- **Locale-Dateien**: Übersetzungen in .po/.mo Dateien

### **Manuelle Qualitätssicherung**
- **Browser-Tests**: Manuelle Überprüfung der Sprachversionen
- **Übersetzungs-Check**: Kontrolle der Übersetzungsvollständigkeit