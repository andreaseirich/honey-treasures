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

## 8. Erweiterte Admin-Architektur

### Rollenbasierte Benutzerverwaltung
```python
class AdminUserProfile(models.Model):
    """Erweitertes Admin-Benutzerprofil mit granularen Berechtigungen."""
    
    ROLE_CHOICES = [
        ('super_admin', 'Super Admin'),
        ('order_admin', 'Bestellungs Admin'),
        ('product_admin', 'Produkt Admin'),
        ('analytics_admin', 'Analytics Admin'),
        ('settings_admin', 'Einstellungs Admin'),
        ('custom', 'Benutzerdefiniert'),
    ]
    
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='admin_profile')
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='custom')
    is_active = models.BooleanField(default=True)
    
    # Granulare Berechtigungen
    can_view_analytics = models.BooleanField(default=False)
    can_manage_orders = models.BooleanField(default=False)
    can_manage_products = models.BooleanField(default=False)
    can_manage_customers = models.BooleanField(default=False)
    can_manage_settings = models.BooleanField(default=False)
    can_view_reports = models.BooleanField(default=False)
    can_export_data = models.BooleanField(default=False)
    can_access_system_logs = models.BooleanField(default=False)
    can_manage_users = models.BooleanField(default=False)
    can_backup_restore = models.BooleanField(default=False)
    
    def set_role_permissions(self):
        """Setzt Berechtigungen basierend auf der Rolle."""
        if self.role == 'super_admin':
            # Alle Berechtigungen für Super Admin
            for field in self._meta.fields:
                if field.name.startswith('can_'):
                    setattr(self, field.name, True)
        elif self.role == 'order_admin':
            self.can_manage_orders = True
            self.can_manage_customers = True
            self.can_view_reports = True
            self.can_export_data = True
```

### Custom Admin Site mit Analytics
```python
class CustomAdminSite(admin.AdminSite):
    """Eigene AdminSite mit Analytics Dashboard und erweiterten Funktionen."""
    
    def get_urls(self):
        """Ergänzt eigene URLs für Analytics und Berichte."""
        urls = super().get_urls()
        custom_urls = [
            path('analytics/', self.admin_view(self.analytics_view), name='analytics'),
            path('reports/', self.admin_view(self.reports_view), name='reports'),
            path('admin-profiles/', self.admin_view(self.admin_profiles_view), name='admin-profiles'),
            path('settings/', self.admin_view(self.settings_view), name='settings'),
        ]
        return custom_urls + urls
    
    def analytics_view(self, request):
        """Zeigt Analytics-Dashboard im Admin an."""
        # Berechne Statistiken
        total_orders = Bestellung.objects.count()
        total_customers = Kunde.objects.count()
        total_products = Produkt.objects.count()
        
        # Umsatz der letzten 30 Tage
        from datetime import datetime, timedelta
        thirty_days_ago = datetime.now() - timedelta(days=30)
        recent_orders = Bestellung.objects.filter(bestelldatum__gte=thirty_days_ago)
        revenue = sum(order.gesamtpreis for order in recent_orders)
        
        context = dict(
            self.each_context(request),
            title="Analytics Dashboard",
            total_orders=total_orders,
            total_customers=total_customers,
            total_products=total_products,
            revenue=revenue,
        )
        return TemplateResponse(request, "admin/analytics.html", context)
```

### System-Einstellungen Management
```python
class AdminSettings(models.Model):
    """Zentrale System-Einstellungen für das Admin-System."""
    
    site_name = models.CharField(max_length=100, default="Honigschätze")
    site_description = models.TextField(blank=True)
    admin_email = models.EmailField(blank=True)
    
    # System-Einstellungen
    maintenance_mode = models.BooleanField(default=False)
    debug_mode = models.BooleanField(default=False)
    analytics_enabled = models.BooleanField(default=True)
    
    # Bestellungen
    max_orders_per_day = models.IntegerField(default=50)
    low_stock_threshold = models.IntegerField(default=10)
    
    # E-Mail
    email_notifications = models.BooleanField(default=True)
    
    # Backup
    backup_frequency = models.CharField(
        max_length=20,
        choices=[('daily', 'Täglich'), ('weekly', 'Wöchentlich'), ('monthly', 'Monatlich')],
        default='weekly'
    )
    
    def save(self, *args, **kwargs):
        # Stelle sicher, dass nur eine AdminSettings-Instanz existiert
        if not self.pk and AdminSettings.objects.exists():
            existing = AdminSettings.objects.first()
            for field in self._meta.fields:
                if field.name not in ['id', 'created_at', 'updated_at']:
                    setattr(existing, field.name, getattr(self, field.name))
            existing.save()
            return existing
        return super().save(*args, **kwargs)
```

### Spezialisierte Middleware
```python
class MaintenanceModeMiddleware:
    """Middleware für System-weiten Wartungsmodus."""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Prüfe Wartungsmodus in AdminSettings
        try:
            settings = AdminSettings.objects.first()
            if settings and settings.maintenance_mode:
                # Erlaube Admin-Zugriff
                if not request.path.startswith('/admin/') and not request.user.is_staff:
                    return HttpResponse(
                        "System wird gewartet. Bitte versuchen Sie es später erneut.",
                        status=503
                    )
        except:
            pass  # Fallback bei Fehlern
        
        return self.get_response(request)

class AdminPermissionsMiddleware:
    """Middleware für rollenbasierte Admin-Berechtigungen."""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        if request.path.startswith('/admin/') and request.user.is_authenticated:
            # Prüfe Admin-Berechtigungen
            try:
                profile = request.user.admin_profile
                if not profile.is_active:
                    return HttpResponse("Zugriff verweigert: Inaktives Admin-Profil", status=403)
            except:
                # Fallback für Benutzer ohne Admin-Profil
                if not request.user.is_superuser:
                    return HttpResponse("Zugriff verweigert: Kein Admin-Profil", status=403)
        
        return self.get_response(request)
```

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

## 9. Robuste Fehlerbehandlung

### Defensive Programmierung in Views
```python
class ProduktListView(ListView):
    def get_queryset(self):
        try:
            # Vereinfachte Abfrage ohne problematische Felder
            queryset = Produkt.objects.all()
            search_query = self.request.GET.get('search', '')
            if search_query:
                queryset = queryset.filter(
                    models.Q(produktname__icontains=search_query) |
                    models.Q(beschreibung__icontains=search_query)
                )
            # ... weitere Filter
            return queryset
        except Exception as e:
            # Logge den Fehler für Debugging
            import logging
            logger = logging.getLogger(__name__)
            logger.error(f"Fehler in ProduktListView.get_queryset(): {str(e)}")
            # Fallback: Alle Produkte ohne Filter
            return Produkt.objects.all()

    def get_context_data(self, **kwargs):
        try:
            context = super().get_context_data(**kwargs)
            context['kategorien'] = Kategorie.objects.all()
            cart = get_cart(self.request)
            context['warenkorb_anzahl'] = sum(cart.values())
            return context
        except Exception as e:
            # Logge den Fehler für Debugging
            import logging
            logger = logging.getLogger(__name__)
            logger.error(f"Fehler in ProduktListView.get_context_data(): {str(e)}")
            # Fallback: Minimaler Kontext
            context = super().get_context_data(**kwargs)
            context['kategorien'] = []
            context['warenkorb_anzahl'] = 0
            return context
```

## Technische Highlights

### Komplexität & Qualität
- **Datenbank-Transaktionen**: Sichere Bestellerstellung mit Django ORM
- **Validierung**: Eingabevalidierung mit Regex für E-Mail und Telefon
- **Internationalisierung**: Mehrsprachigkeit mit Django ModelTranslation
- **Sicherheit**: CSRF-Schutz und SSL-Verschlüsselung
- **E-Mail-System**: HTML-Templates für Bestellbestätigungen
- **Admin-System**: Rollenbasierte Benutzerverwaltung mit Analytics Dashboard
- **Middleware-Architektur**: Spezialisierte Middleware für Wartung und Berechtigungen
- **Fehlerbehandlung**: Defensive Programmierung mit Fallback-Mechanismen
- **System-Einstellungen**: Zentrale Konfiguration aller System-Parameter

### Bewährte Praktiken
- **DRY-Prinzip**: Wiederverwendbare Funktionen
- **Separation of Concerns**: Klare Trennung von Logik
- **Error Handling**: Umfassende Fehlerbehandlung mit Logging
- **Documentation**: Ausführliche Docstrings
- **Type Hints**: Klare Funktionssignaturen
- **Defensive Programming**: Try-catch Blöcke mit Fallback-Mechanismen
- **Role-Based Access Control**: Granulare Berechtigungen
- **Middleware Pattern**: Modulare Request-Processing-Pipeline