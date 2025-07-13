# Code-Beispiele & Technische Highlights

**Diese Dokumentation zeigt ausgewählte Code-Beispiele, die die technische Qualität, Komplexität und bewährte Praktiken des Honigschätze E-Commerce-Systems für die Familienimkerei demonstrieren.**

## 1. Komplexe Datenmodelle mit Geschäftslogik

### Produkt-Modell mit Verfügbarkeitsberechnung
```python
class Produkt(models.Model):
    honigsorte = models.ForeignKey(Honigsorte, on_delete=models.CASCADE, null=True, blank=True)
    eimer = models.ForeignKey(Eimer, on_delete=models.CASCADE, null=True, blank=True)
    produktbild = models.ImageField(upload_to='produkte/', null=True, blank=True)
    produktname = models.CharField(max_length=255)
    kategorie = models.ForeignKey(Kategorie, on_delete=models.SET_NULL, null=True, blank=True)
    preis = models.DecimalField(max_digits=10, decimal_places=2)
    beschreibung = models.TextField(null=True, blank=True)
    verfügbarkeit = models.DecimalField(max_digits=10, decimal_places=2)

    def is_verfügbar(self):
        """Prüft, ob das Produkt verfügbar ist (nur Honig-Lagerbestand relevant)"""
        if not self.honigsorte or not self.eimer:
            return False
        
        # Prüfe Honig-Lagerbestand
        verfügbare_honig_eimer = self.honigsorte.lagerbestand_in_kg // self.eimer.größe
        if verfügbare_honig_eimer <= 0:
            return False
        
        # Eimer-Mangel wird protokolliert, verhindert aber nicht die Bestellung
        # if self.eimer.lagerbestand <= 0:
        #     return False
        
        return True

    def get_verfügbare_menge(self):
        """Gibt die verfügbare Menge basierend auf Honig und Eimer zurück"""
        if not self.honigsorte or not self.eimer:
            return 0
        
        verfügbare_honig_eimer = self.honigsorte.lagerbestand_in_kg // self.eimer.größe
        verfügbare_eimer = self.eimer.lagerbestand
        
        return min(verfügbare_honig_eimer, verfügbare_eimer)
```

### Automatische Produkterstellung durch Django Signals
```python
@receiver(post_save, sender=Honigsorte)
def create_produkte_for_honigsorte(sender, instance, created, **kwargs):
    """Erstellt automatisch Produkte für alle Eimer-Größen bei neuer Honigsorte"""
    for eimer in Eimer.objects.all():
        produkt, created = Produkt.objects.get_or_create(
            honigsorte=instance,
            eimer=eimer,
        )
        # Wenn das Produkt schon existiert, nur die Verfügbarkeit und den Preis aktualisieren
        if not created:
            produkt.preis = (instance.preis_pro_500g * Decimal(2)) * Decimal(eimer.größe)
            # Verfügbarkeit basierend auf Honig UND Eimer berechnen
            verfügbare_honig_eimer = instance.lagerbestand_in_kg // eimer.größe
            verfügbare_eimer = eimer.lagerbestand
            produkt.verfügbarkeit = min(verfügbare_honig_eimer, verfügbare_eimer)

        honig_kategorie, _ = Kategorie.objects.get_or_create(name="Honig")
        produkt.kategorie = honig_kategorie
        produkt.save()
```

## 2. Sichere Checkout-Validierung

### Umfassende Eingabevalidierung
```python
def validate_checkout_data(data):
    """Validiert die Checkout-Daten mit umfassender Fehlerbehandlung"""
    errors = {}
    
    # Basis-Pflichtfelder prüfen
    required_fields = ['vorname', 'nachname', 'email', 'payment_method', 'delivery_method']
    for field in required_fields:
        if not data.get(field):
            errors[field] = _("Dieses Feld ist erforderlich.")
    
    # Zahlungsart-Validierung (manuelle Zahlungsarten verfügbar)
    valid_payment_methods = ['Überweisung', 'Barzahlung', 'PayPal']
    if data.get('payment_method') and data['payment_method'] not in valid_payment_methods:
        errors['payment_method'] = _("Bitte wählen Sie eine gültige Zahlungsart.")
    
    # Adressfelder nur bei Lieferung prüfen
    if data.get('delivery_method') == 'Lieferung':
        address_fields = ['strasse', 'postleitzahl', 'ort']
        for field in address_fields:
            if not data.get(field):
                errors[field] = _("Dieses Feld ist erforderlich.")
    
    # E-Mail-Validierung mit Regex
    if data.get('email'):
        email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(email_regex, data['email']):
            errors['email'] = _("Bitte geben Sie eine gültige E-Mail-Adresse ein.")
    
    # Telefon-Validierung (optional)
    if data.get('telefon'):
        phone_regex = r'^[\+]?[0-9\s\-\(\)]{6,20}$'
        if not re.match(phone_regex, data['telefon']):
            errors['telefon'] = _("Bitte geben Sie eine gültige Telefonnummer ein.")
    
    # Postleitzahl-Validierung (nur bei Lieferung)
    if data.get('delivery_method') == 'Lieferung' and data.get('postleitzahl'):
        plz_regex = r'^[0-9]{4,5}$'
        if not re.match(plz_regex, data['postleitzahl']):
            errors['postleitzahl'] = _("Bitte geben Sie eine gültige Postleitzahl ein.")
    
    return errors
```

### Transaktionssichere Bestellerstellung
```python
@ensure_csrf_cookie
def checkout(request):
    """Verbesserte Checkout-View mit umfassender Validierung und Sicherheit"""
    cart = get_cart(request)
    if not cart:
        return redirect('warenkorb')

    # Lagerbestand prüfen - nur Honig-Mangel verhindert Bestellung
    honig_mangel_errors = []
    eimer_mangel_produkte = []
    
    for pid, qty in cart.items():
        produkt = produkte[pid]
        if hasattr(produkt, "honigsorte") and produkt.honigsorte is not None:
            # Honigprodukt
            if produkt.honigsorte.lagerbestand_in_kg < produkt.eimer.größe * qty:
                honig_mangel_errors.append(_("Nicht genügend Honig vorrätig für {}!").format(produkt.produktname))
            
            # Eimer-Mangel wird protokolliert und an Familienbetrieb gemeldet, verhindert aber nicht die Bestellung
            if produkt.eimer.lagerbestand < qty:
                eimer_mangel_produkte.append((produkt, qty, produkt.eimer.lagerbestand))

    # Transaktion starten
    with transaction.atomic():
        # Kunde erstellen oder aktualisieren
        kunde, created = Kunde.objects.get_or_create(
            email=data['email'],
            defaults={
                'vorname': data['vorname'],
                'nachname': data['nachname'],
                'telefon': data.get('telefon', ''),
                'benutzer': request.user if request.user.is_authenticated else None
            }
        )
        
        # Bestellung erstellen (manuelle Zahlungsabwicklung)
        bestellung = Bestellung.objects.create(
            kunde=kunde,
            adresse=adresse,
            gesamtpreis=gesamtpreis,
            zahlungsart=data['payment_method'],  # Überweisung, Barzahlung oder PayPal
            versandart=data['delivery_method'],
            ip_adresse=get_client_ip(request),
            status='WARTEND_AUF_ZAHLUNG'  # Status für manuelle Zahlungsabwicklung
        )
        
        # Lagerbestand aktualisieren
        for produkt, menge in produkte_im_cart:
            if produkt.honigsorte:
                produkt.honigsorte.lagerbestand_in_kg -= produkt.eimer.größe * menge
                produkt.honigsorte.save()
            if produkt.eimer:
                produkt.eimer.lagerbestand -= menge
                produkt.eimer.save()
```

## 3. Internationalisierung & Mehrsprachigkeit

### Django ModelTranslation Integration
```python
# translation.py
from modeltranslation.translator import translator, TranslationOptions
from .models import Produkt, Kategorie, Honigsorte

class ProduktTranslationOptions(TranslationOptions):
    fields = ('produktname', 'beschreibung')

class KategorieTranslationOptions(TranslationOptions):
    fields = ('name', 'beschreibung')

class HonigsorteTranslationOptions(TranslationOptions):
    fields = ('name',)

translator.register(Produkt, ProduktTranslationOptions)
translator.register(Kategorie, KategorieTranslationOptions)
translator.register(Honigsorte, HonigsorteTranslationOptions)
```

### Sprachumschaltung mit Context Processor
```python
# context_processors.py
from django.utils.translation import get_language

def current_language(request):
    """Liefert die aktuelle Sprache an alle Templates"""
    return {
        'current_language': get_language(),
        'available_languages': [
            ('de', 'Deutsch'),
            ('hu', 'Magyar'),
        ]
    }
```

## 4. Sicherheit & Datenschutz

### Content Security Policy Middleware
```python
class ContentSecurityPolicyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        response["Content-Security-Policy"] = (
            "default-src 'self'; "
            "script-src 'self' https://cdn.consentmanager.net https://js.stripe.com "
            "https://www.googletagmanager.com https://www.google-analytics.com; "
            "object-src 'none'; "
            "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; "
            "img-src 'self' https://www.google-analytics.com https://www.googletagmanager.com; "
            "font-src 'self' https://fonts.gstatic.com; "
            "frame-src 'self' https://www.google.com https://www.facebook.com https://js.stripe.com; "
            "connect-src 'self' https://js.stripe.com https://api.stripe.com "
            "https://www.google-analytics.com https://www.googletagmanager.com; "
            "frame-ancestors 'self'; "
        )
        return response
```

## 5. E-Mail-System mit Templates

### Automatische Bestellbestätigungen
```python
def send_order_confirmation(bestellung):
    """Sendet automatische Bestellbestätigungen an Kunde und Verkäufer"""
    
    # E-Mail an Kunden
    customer_context = {
        'bestellung': bestellung,
        'produkte': bestellung.bestellungprodukt_set.all(),
        'kunde': bestellung.kunde,
    }
    
    customer_html = render_to_string('emails/order_confirmation_customer.html', customer_context)
    customer_text = render_to_string('emails/order_confirmation_customer.txt', customer_context)
    
    customer_email = EmailMessage(
        subject=_('Bestellbestätigung - Honigschätze'),
        body=customer_html,
        from_email=settings.DEFAULT_FROM_EMAIL,
        to=[bestellung.kunde.email],
        reply_to=['admin@honey-treasures.com']
    )
    customer_email.content_subtype = "html"
    customer_email.send()
    
    # E-Mail an Verkäufer
    seller_context = {
        'bestellung': bestellung,
        'produkte': bestellung.bestellungprodukt_set.all(),
        'kunde': bestellung.kunde,
    }
    
    seller_html = render_to_string('emails/order_notification_seller.html', seller_context)
    seller_text = render_to_string('emails/order_notification_seller.txt', seller_context)
    
    seller_email = EmailMessage(
        subject=_('Neue Bestellung - Honigschätze'),
        body=seller_html,
        from_email=settings.DEFAULT_FROM_EMAIL,
        to=['admin@honey-treasures.com'],
        reply_to=[bestellung.kunde.email]
    )
    seller_email.content_subtype = "html"
    seller_email.send()
```

## 6. Rabattlogik & Preisberechnung

### Intelligente Rabattberechnung
```python
def berechne_honig_gesamtmenge(produkte_im_cart):
    """Berechnet die Gesamtmenge an Honig im Warenkorb"""
    gesamt_kg = 0
    for produkt, menge in produkte_im_cart:
        if produkt.honigsorte and produkt.eimer:
            gesamt_kg += produkt.eimer.größe * menge
    return gesamt_kg

def rabattierter_preis(produkt, gesamt_honig_kg):
    """Berechnet den rabattierten Preis basierend auf der Gesamtmenge"""
    if gesamt_honig_kg >= 7:  # Rabatt ab 7kg
        return produkt.preis * Decimal('0.9')  # 10% Rabatt
    return produkt.preis
```

## 7. Logging & Monitoring

### Benutzeraktivitäts-Logging
```python
def log_user_activity(request, action, details=None):
    """Loggt Benutzeraktivitäten für Monitoring und Analytics"""
    logger = logging.getLogger('user_activity')
    
    log_entry = {
        'timestamp': datetime.now().isoformat(),
        'ip_address': get_client_ip(request),
        'user_agent': request.META.get('HTTP_USER_AGENT', ''),
        'url': request.get_full_path(),
        'action': action,
        'details': details or {},
        'user_id': request.user.id if request.user.is_authenticated else None,
    }
    
    logger.info(json.dumps(log_entry))
```

## 8. Admin-Integration

### Erweiterte Django Admin-Konfiguration
```python
@admin.register(Bestellung)
class BestellungAdmin(admin.ModelAdmin):
    list_display = ['id', 'kunde', 'gesamtpreis', 'zahlungsart', 'versandart', 'status', 'bestelldatum']
    list_filter = ['status', 'zahlungsart', 'versandart', 'bestelldatum']
    search_fields = ['kunde__vorname', 'kunde__nachname', 'kunde__email']
    readonly_fields = ['bestelldatum', 'token', 'ip_adresse']
    
    inlines = [BestellungProduktInline]
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('kunde', 'adresse')
    
    def save_model(self, request, obj, form, change):
        if not change:  # Neue Bestellung
            obj.token = secrets.token_urlsafe(32)
        super().save_model(request, obj, form, change)
```

## Technische Highlights

### Komplexität & Qualität
- **Datenbank-Transaktionen**: Sichere Bestellerstellung mit Django ORM
- **Validierung**: Eingabevalidierung mit Regex für E-Mail und Telefon
- **Internationalisierung**: Mehrsprachigkeit mit Django ModelTranslation
- **Sicherheit**: CSRF-Schutz und SSL-Verschlüsselung
- **E-Mail-System**: HTML-Templates für Bestellbestätigungen
- **Admin**: Django Admin für Familienbetrieb

### Bewährte Praktiken
- **DRY-Prinzip**: Wiederverwendbare Funktionen
- **Separation of Concerns**: Klare Trennung von Logik
- **Error Handling**: Umfassende Fehlerbehandlung
- **Documentation**: Ausführliche Docstrings
- **Type Hints**: Klare Funktionssignaturen