# 🍯 Honigschätze – E-Commerce Websystem

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://python.org)
[![Django](https://img.shields.io/badge/Django-5.x-green.svg)](https://djangoproject.com)
[![E-Commerce](https://img.shields.io/badge/E--Commerce-Complete-orange.svg)]()
[![Multilingual](https://img.shields.io/badge/Multilingual-DE%2FHU-purple.svg)]()
[![Status](https://img.shields.io/badge/Status-Complete-success.svg)]()


## 🚀 **Projekt-Status**

**Live & Produktiv** - Vollständig funktionsfähige E-Commerce-Anwendung

**Aktueller Status:**
- ✅ **Online verfügbar** - https://honey-treasures.com
- ✅ **Deutsche Version** - Vollständig implementiert und übersetzt
- 🔄 **Ungarische Version** - In Entwicklung (Grundstruktur vorhanden)
- ✅ **Admin-Backend** - Umfassende Verwaltung für Familienbetrieb
- ✅ **Sicherheit & DSGVO** - Vollständig konform
- ✅ **Performance** - Optimiert und getestet

## 📋 **Projektüberblick**

**Honigschätze** ist ein vollwertiges E-Commerce-Websystem für die Familienimkerei meines Vaters. Die Anwendung ermöglicht es Kunden, Honigprodukte online zu bestellen und bietet eine umfassende Verwaltung für das Familienunternehmen. Das System ist bereits online verfügbar und wird aktiv genutzt.

---

## Inhaltsverzeichnis
- [Features](#features)
- [Architektur](#architektur)
- [Technologien](#technologien)
- [Internationalisierung](#internationalisierung)
- [Sicherheit & Datenschutz](#sicherheit--datenschutz)
- [Admin-Backend](#admin-backend)
- [Bestellprozess](#bestellprozess)
- [Datenmodell](#datenmodell)
- [Code-Qualität & Testing](#code-qualität--testing)
- [Performance & Skalierbarkeit](#performance--skalierbarkeit)
- [Deployment & DevOps](#deployment--devops)
- [Erweiterte Dokumentation](#erweiterte-dokumentation)

---

## 🛍️ **E-Commerce Features**

### **Kunden-Erlebnis**
- **Produktkatalog**: Umfassende Honigprodukte der Familienimkerei mit detaillierten Beschreibungen
- **Intelligenter Warenkorb**: Mengenrabatte ab 7kg, automatische Preisberechnung
- **Bestellprozess**: Online-Bestellung mit manueller Zahlungsabwicklung
- **Zahlungsarten**: Überweisung, Barzahlung, PayPal (alle manuell abgewickelt)
- **Bestellverfolgung**: E-Mail-Bestätigungen und Bestellübersicht
- **Mehrsprachigkeit**: Vollständige deutsche Version, ungarische Version in Entwicklung
- **Mobile-Optimierung**: Touch-optimierte Benutzeroberfläche für alle Bildschirmgrößen

### **Admin-Funktionen**
- **Produktverwaltung**: Honigsorten, Eimer-Größen, Kategorien für Familienbetrieb verwalten
- **Lagerverwaltung**: Automatische Bestandsführung mit Engpass-Warnungen
- **Bestellungsverwaltung**: Umfassende Übersicht und Status-Management für Familienbetrieb
- **Kundenverwaltung**: Kundenprofile und Bestellhistorie
- **E-Mail-System**: Automatische Benachrichtigungen an Kunden und Familienbetrieb

---

## Architektur

- **Backend**: Django (Python), Model-View-Template (MVT)
- **Frontend**: HTML5, CSS3, JavaScript, Mobile-First Responsive Design-System
- **Templates**: Modular, mit Blöcken für Wiederverwendbarkeit
- **Internationalisierung**: Django ModelTranslation, i18n-Tags in Templates, Sprachumschaltung per URL und Dropdown
- **Datenbank**: SQLite (entwicklungsnah, leicht auf PostgreSQL/MySQL portierbar)
- **E-Mail**: SMTP für Bestell- und Kontaktmails
- **Zahlung**: Manuelle Zahlungsabwicklung (Überweisung, Barzahlung, PayPal), Stripe in Entwicklung

---

## 🛠️ **Technologie-Stack**

### **Backend**
- **Python 3.11+** - Moderne Python-Version (automatisch aktualisiert)
- **Django 5.x** - Web-Framework mit MVT-Architektur (automatisch aktualisiert)
- **django-modeltranslation** - Mehrsprachige Datenmodelle
- **SQLite** - Datenbank für Familienbetrieb

### **Frontend**
- **HTML5, CSS3, JavaScript** - Moderne Web-Standards
- **Mobile-First CSS** - Responsive Design mit Media Queries für alle Bildschirmgrößen
- **Touch-Optimierungen** - Größere Touch-Targets, Touch-Feedback, iOS-Zoom-Verhinderung
- **Vanilla JavaScript** - Interaktive Features ohne Frameworks, mobile Performance-Optimierungen

### **Zahlungen & APIs**
- **Manuelle Zahlungsabwicklung** - Überweisung, Barzahlung und PayPal
- **Stripe Integration** - In Entwicklung für automatische Kartenzahlung
- **SMTP** - E-Mail-Versand mit HTML-Templates

### **DevOps & Deployment**
- **Nginx + Gunicorn** - Production-Server-Setup auf Raspberry Pi
- **Let's Encrypt** - SSL/TLS-Zertifikate

---

## 🌍 **Internationalisierung**

- **Deutsch (de)**: ✅ Vollständig implementiert und übersetzt
- **Ungarisch (hu)**: 🔄 Technisch vorbereitet, Übersetzung noch nicht implementiert
- **Django ModelTranslation**: Übersetzbare Felder für alle Inhalte
- **SEO-optimiert**: Sprachspezifische URLs und Meta-Tags
- **E-Mail-Templates**: Einsprachige E-Mail-Benachrichtigungen (Mehrsprachigkeit in Planung)

**Hinweis**: Die ungarische Version ist technisch vorbereitet, aber die Übersetzung ist noch nicht implementiert. E-Mail-Benachrichtigungen werden derzeit nur in Deutsch versendet.

---

## 🔒 **Sicherheit & Datenschutz**

- **Web-Sicherheit**: CSRF, XSS-Schutz, SQL-Injection-Schutz
- **SSL/TLS**: Vollständige HTTPS-Verschlüsselung mit Let's Encrypt
- **DSGVO-Compliance**: Umfassende Datenschutzerklärung und Cookie-Hinweise
- **Zahlungssicherheit**: Manuelle Zahlungsabwicklung ohne sensible Daten im System
- **Monitoring**: Umfassendes Logging und Sicherheits-Tracking

---

## ⚙️ **Admin-Backend**

- **Django Admin**: Umfassende Verwaltung aller Daten (Honigsorten, Produkte, Bestellungen, Kunden)
- **Übersetzbare Felder**: Direkte Bearbeitung mehrsprachiger Inhalte im Admin
- **Erweiterte Funktionen**: Filter, Suche, Inline-Bearbeitung
- **Benutzerfreundlich**: Intuitive Oberfläche für Nicht-Entwickler

---

## 🛒 **Bestellprozess**

1. **Produktauswahl**: Kunde wählt Honigprodukte und Mengen
2. **Warenkorb**: Intelligente Rabattlogik ab 7kg Honig
3. **Checkout**: Umfassende Eingabevalidierung (E-Mail, Telefon, Adresse)
4. **Zahlungsauswahl**: Überweisung, Barzahlung oder PayPal (manuelle Abwicklung)
5. **Bestellbestätigung**: Automatische E-Mails an Kunde und Familienbetrieb
6. **Manuelle Zahlungsabwicklung**: Zahlung wird nach Bestellung manuell abgewickelt
7. **Lagerverwaltung**: Atomare Aktualisierung der Bestände mit Eimer-Mangel-Benachrichtigung
8. **Bestellverfolgung**: Kunden können Bestellungen online einsehen

---

## 🗄️ **Datenmodell**

### **Kern-Entitäten**
- **Honigsorte**: Name, Preis pro 500g, Lagerbestand (kg)
- **Eimer**: Größe (kg), Lagerbestand
- **Produkt**: Kombination aus Honigsorte + Eimer mit Verfügbarkeitsberechnung
- **Kategorie**: Produktkategorisierung und -organisation

### **Geschäftsprozesse**
- **Kunde**: Profil mit Bestellhistorie und Präferenzen
- **Bestellung**: Vollständige Bestelldaten mit Status-Tracking
- **BestellungProdukt**: Verknüpfung zwischen Bestellung und Produkten
- **Adresse**: Liefer- und Rechnungsadressen

### **Besonderheiten**
- **Automatische Produkterstellung**: Neue Honigsorten werden automatisch für alle Eimer-Größen erstellt
- **Verfügbarkeitsberechnung**: Produkte sind nur verfügbar, wenn Honig vorrätig (Eimer-Mangel wird protokolliert, verhindert aber nicht die Bestellung)
- **Übersetzbare Felder**: Produktnamen und -beschreibungen mehrsprachig

---

## 🧪 **Code-Qualität & Testing**

### **Test-Strategie**
- **Manuelle Tests**: Funktionsprüfung durch Entwickler
- **Browser-Tests**: Manuelle Überprüfung der Hauptfunktionen
- **Sicherheit**: Grundlegende Django-Sicherheitsfeatures aktiviert

### **Qualitätssicherung**
- **Code Review**: Entwickler-Review bei Änderungen
- **Manuelle Validierung**: Funktionsprüfung vor Deployment

---

## ⚡ **Performance & Skalierbarkeit**

### **Performance & Stabilität**
- **Live-Status**: Online verfügbar unter https://honey-treasures.com
- **Stabilität**: System läuft stabil in Produktion
- **Response Time**: Gute Performance für Familienbetrieb

### **Optimierungen**
- **Django ORM**: Effiziente Datenbankabfragen
- **Static Files**: Standard Nginx-Konfiguration
- **Mobile Performance**: Touch-Optimierungen, Lazy Loading, debounced Events
- **Code-Struktur**: Übersichtliche Django-App-Struktur

### **Skalierbarkeit**
- **Datenbank**: SQLite für Familienbetrieb ausreichend
- **Architektur**: Einfache Erweiterbarkeit durch Django-Framework

---

## 🚀 **Deployment & DevOps**

### **Production-Setup**
- **Nginx + Gunicorn**: Professionelle Server-Konfiguration
- **SSL/TLS**: Let's Encrypt Integration mit automatischer Erneuerung
- **Security**: Firewall, CSP, Secure Headers, HSTS
- **Backup Strategy**: Automatisierte tägliche Backups
- **Live-Website**: https://honey-treasures.com

### **DevOps-Pipeline**
- **Manuelles Deployment**: Direkte Bereitstellung auf Server
- **Git Version Control**: Code-Versionskontrolle
- **Basis-Monitoring**: Server-Logs und Error-Tracking
- **Backup-Strategie**: Regelmäßige Datenbank-Backups

---

## 🌐 **Live-System**

### **Online verfügbar**
- **Website**: https://honey-treasures.com
- **Admin-Backend**: Verfügbar für Familienbetrieb
- **Deutsche Version**: Vollständig funktionsfähig
- **Ungarische Version**: Technisch vorbereitet, Übersetzung noch nicht implementiert

---

## Erweiterte Dokumentation

Für eine vollständige technische und fachliche Dokumentation siehe den Ordner [`/docs/`](./docs/):

### **Architektur & Design**
- **ARCHITEKTUR.md**: Systemarchitektur und Komponenten
- **DATENMODELL.md**: Datenmodell und Beziehungen
- **USE_CASES.md**: Nutzer- und Admin-Szenarien

### **Technische Implementierung**
- **CODE_EXAMPLES.md**: Ausgewählte Code-Beispiele mit komplexer Geschäftslogik
- **PERFORMANCE.md**: Performance-Optimierungen und Skalierbarkeitsmaßnahmen
- **MOBILE_OPTIMIZATION.md**: Mobile-Optimierungen und Touch-Interaktionen

### **Sicherheit & Deployment**
- **SECURITY.md**: Sicherheit & Datenschutz (DSGVO)
- **DEPLOYMENT.md**: Deployment-Prozess und Produktivumgebung
- **INTERNATIONALISIERUNG.md**: Mehrsprachigkeit, technische Umsetzung