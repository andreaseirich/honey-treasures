# Datenmodell

**Dieses Dokument erläutert die wichtigsten Datenmodelle, deren Beziehungen und Besonderheiten für das Honigschätze E-Commerce-System der Familienimkerei. Ein ER-Diagramm visualisiert die Zusammenhänge.**

## Hauptmodelle

- **Honigsorte**: Name, Preis pro 500g, Lagerbestand (kg)
- **Eimer**: Größe (kg), Lagerbestand
- **Produkt**: Name, Bild, Kategorie, Preis, Beschreibung, Verfügbarkeit, Honigsorte, Eimer
- **Kategorie**: Name, Beschreibung
- **Kunde**: Vorname, Nachname, E-Mail, Telefon, Benutzer (optional)
- **Bestellung**: Kunde, Adresse, Gesamtpreis, Zahlungsart, Versandart, Status, Produkte
- **BestellungProdukt**: Bestellung, Produkt, Menge, Einzelpreis
- **Adresse**: Straße, Hausnummer, PLZ, Stadt, Land, Kunde
- **Benutzeraktivität**: IP-Adresse, User-Agent, URL, Zeitstempel
- **UserActivityLog**: Zeit, User-Label, Gerät, IP, Ort, User-Agent, Pfad, Aktion
- **DeviceUserLabel**: Gerät, User-Label, First Seen

## Beziehungen (ER-Diagramm)

```mermaid
erDiagram
    Kunde ||--o{ Bestellung : "hat"
    Bestellung ||--|{ BestellungProdukt : "enthält"
    Produkt ||--o{ BestellungProdukt : "kommt vor in"
    Produkt }o--|| Honigsorte : "ist von"
    Produkt }o--|| Eimer : "wird abgefüllt in"
    Produkt }o--|| Kategorie : "gehört zu"
    Bestellung }o--|| Adresse : "wird geliefert an"
    Kunde ||--o{ Adresse : "hat"
    Benutzeraktivität }o--|| Kunde : "optional zugeordnet"
    UserActivityLog }o--|| DeviceUserLabel : "bezieht sich auf"
```

## Besonderheiten
- Verfügbarkeit eines Produkts ergibt sich aus Honigbestand (Eimer-Mangel wird protokolliert, verhindert aber nicht die Bestellung)
- Rabattlogik ab 7kg Honig
- Bestellungen speichern IP-Adresse und Status
- Übersetzbare Felder für Produktnamen, Beschreibungen, Kategorienamen
- Benutzeraktivitäten und Geräte-Labels werden zur Sicherheit und Nachvollziehbarkeit in der Datenbank gespeichert 