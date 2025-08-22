# Key-Naming-Konventionen für Redis

## Übersicht

Redis-Keys müssen eindeutig, vorhersagbar und hierarchisch navigierbar sein. Diese Konventionen stellen sicher, dass aus jedem Markdown-Text konsistente, kollisionsfreie Redis-Strukturen entstehen.

## Basis-Prinzipien

### 1. Namespace-Trennung
Alle Keys beginnen mit einem **Typ-Präfix** für klare Kategorisierung:

```
doc:       - Dokument-Level
ch:        - Chapter-Level (# Headers)
para:      - Paragraph-Level (## Headers)  
subpara:   - SubParagraph-Level (### Headers)
chunk:     - Content-Chunk-Level (Text ohne Header)
```

### 2. Hierarchische Struktur
```
Format: {prefix}:{clean_identifier}:{number}

Beispiele:
doc:portfolio_management_strategies:001
ch:risk_assessment_fundamentals:001
para:volatility_measurement_methods:001
subpara:standard_deviation_analysis:001
chunk:correlation_coefficient_explanation:001
```

### 3. Eindeutigkeit durch Nummerierung
- **3-stellige Nummerierung**: 001, 002, 003...
- **Pro Typ fortlaufend**: Nicht pro Parent, sondern global
- **Automatische Kollisions-Vermeidung**: Bei Duplikaten automatisch hochzählen

## Text-zu-Key-Transformation

### Schritt 1: Identifier-Extraktion

#### Aus Header-Texten:
```
"# Portfolio Management & Risk Assessment" → "portfolio_management_risk_assessment"
"## Diversifikation: Der Schlüssel zum Erfolg" → "diversifikation_der_schluessel_zum_erfolg"
"### Value at Risk (VaR) Berechnungen" → "value_at_risk_var_berechnungen"
```

#### Aus Content-Texten (erste 3-5 Wörter):
```
"Diversifikation ist der Grundstein erfolgreicher Investments." 
→ "diversifikation_ist_der_grundstein"

"Standard Deviation misst die Volatilität von Renditen."
→ "standard_deviation_misst_die"

"Korrelationskoeffizienten zwischen Assets bestimmen Diversifikationseffekte."
→ "korrelationskoeffizienten_zwischen_assets_bestimmen"
```

### Schritt 2: Text-Bereinigung

#### Reihenfolge der Transformationen:
1. **Lowercase**: Alle Großbuchstaben → klein
2. **Umlaut-Konversion**: ä→ae, ö→oe, ü→ue, ß→ss
3. **Sonderzeichen entfernen**: Alle nicht-alphanumerischen Zeichen
4. **Leerzeichen ersetzen**: Spaces → Unterstriche
5. **Mehrfach-Unterstriche bereinigen**: __ → _
6. **Längen-Begrenzung**: Maximal 30 Zeichen
7. **Trailing-Bereinigung**: Führende/abschließende Unterstriche entfernen

#### Bereinigung-Beispiele:
```
"Portfolio Management & Risk Assessment Strategies"
→ "portfolio management & risk assessment strategies"  (lowercase)
→ "portfolio management  risk assessment strategies"   (sonderzeichen)
→ "portfolio_management__risk_assessment_strategies"   (spaces)
→ "portfolio_management_risk_assessment_strategies"    (mehrfach-unterstriche)
→ "portfolio_management_risk_asse"                     (längen-begrenzung)
```

```
"Diversifikation: Der Schlüssel zum Erfolg"
→ "diversifikation: der schlüssel zum erfolg"         (lowercase)
→ "diversifikation der schluessel zum erfolg"         (umlaut + sonderzeichen)
→ "diversifikation_der_schluessel_zum_erfolg"         (spaces)
→ "diversifikation_der_schluessel"                    (längen-begrenzung)
```

### Schritt 3: Kollisions-Behandlung

#### Automatische Duplikat-Erkennung:
```python
def generate_unique_key(prefix, identifier, existing_keys):
    base_key = f"{prefix}:{identifier}"
    counter = 1
    
    while f"{base_key}:{counter:03d}" in existing_keys:
        counter += 1
    
    return f"{base_key}:{counter:03d}"

# Beispiel:
# "Risk Management" (1. Vorkommen) → ch:risk_management:001
# "Risk Management" (2. Vorkommen) → ch:risk_management:002
```

## Key-Typen Spezifikation

### Document-Keys
```
Format: doc:{document_identifier}:{number}

Input-Quellen:
1. YAML title (primär)
2. Dateiname ohne Extension (sekundär)
3. Erstes H1-Header (tertiär)

Beispiele:
doc:portfolio_management_complete:001
doc:risk_assessment_framework:001
doc:investment_strategies_guide:001
```

### Chapter-Keys (H1-Headers)
```
Format: ch:{chapter_identifier}:{number}

Input: Header-Text nach dem #

Beispiele:
ch:portfolio_theory_fundamentals:001
ch:risk_management_strategies:001
ch:practical_implementation:001
```

### Paragraph-Keys (H2-Headers)
```
Format: para:{paragraph_identifier}:{number}

Input: Header-Text nach dem ##

Beispiele:
para:diversification_strategies:001
para:volatility_measurement:001
para:correlation_analysis:001
```

### SubParagraph-Keys (H3-Headers)
```
Format: subpara:{subparagraph_identifier}:{number}

Input: Header-Text nach dem ###

Beispiele:
subpara:standard_deviation_calculation:001
subpara:value_at_risk_methodology:001
subpara:beta_coefficient_analysis:001
```

### Chunk-Keys (Content)
```
Format: chunk:{content_identifier}:{number}

Input: Erste 3-5 bedeutungsvolle Wörter des Inhalts

Beispiele:
chunk:diversifikation_ist_der_grundstein:001
chunk:standard_deviation_misst_volatilitaet:001
chunk:korrelationen_bestimmen_risikoreduktion:001
```

## Set-Keys für Beziehungen

### Hierarchie-Sets
```
{parent_key}:children → Direkte Kinder
{parent_key}:all_descendants → Alle Nachkommen (rekursiv)

Beispiele:
doc:portfolio_guide:001:children
ch:risk_management:001:children
para:volatility_analysis:001:children
```

### Sequenz-Sets
```
{scope_key}:sequence → Lese-Reihenfolge

Beispiele:
doc:portfolio_guide:001:sequence (gesamte Dokument-Sequenz)
ch:risk_management:001:sequence (Chapter-interne Sequenz)
para:volatility_analysis:001:sequence (Paragraph-interne Sequenz)
```

### Navigation-Sets
```
{element_key}:next → Nächstes Element
{element_key}:previous → Vorheriges Element
{element_key}:siblings → Geschwister-Elemente

Beispiele:
chunk:diversifikation_grundstein:001:next
chunk:volatilitaet_messung:002:previous
para:risk_assessment:001:siblings
```

### Kategorisierung-Sets
```
category:{category_name}:docs → Dokumente dieser Kategorie
author:{author_clean}:docs → Dokumente dieses Autors
tag:{tag_name}:chunks → Chunks mit diesem Tag
level:{level_name}:chunks → Alle Chunks dieser Hierarchie-Ebene

Beispiele:
category:investment:docs
author:oskar_sch:docs
tag:diversification:chunks
level:chunk:chunks
```

## Spezielle Bereinigungsregeln

### Stopp-Wörter (entfernen für Chunk-Keys)
```
Deutsch: der, die, das, und, oder, ist, sind, ein, eine, zu, zum, zur, von, mit, für, auf, in, an, bei, über, unter
Englisch: the, and, or, is, are, a, an, to, of, with, for, on, in, at, by, over, under

Beispiel:
"Die Korrelation zwischen den Assets ist wichtig"
→ "korrelation_zwischen_assets_wichtig" (ohne "die", "den", "ist")
```

### Zahlen und Abkürzungen
```
Zahlen: Ausschreiben wenn möglich
"5 Strategien" → "fuenf_strategien"
"2024 Analysis" → "zweitausendvierundzwanzig_analysis"

Abkürzungen: Vollform wenn erkennbar
"VaR Berechnung" → "value_at_risk_berechnung"
"ROI Analysis" → "return_on_investment_analysis"

Unbekannte Abkürzungen: Beibehalten
"API Documentation" → "api_documentation"
```

### Sonderzeichen-Mapping
```
& → and
% → prozent
€ → euro
$ → dollar
@ → at
# → hash (nur wenn nicht Header-Marker)
+ → plus
= → equals
< → less_than
> → greater_than
```

## Validierung und Qualitätssicherung

### Key-Format-Validation
```python
import re

def validate_key_format(key):
    pattern = r'^(doc|ch|para|subpara|chunk):[a-z][a-z0-9_]{1,29}:\d{3}$'
    return bool(re.match(pattern, key))

def validate_set_key_format(key):
    pattern = r'^[a-z][a-z0-9_:]{1,50}:(children|sequence|next|previous|siblings)$'
    return bool(re.match(pattern, key))

# Gültige Keys:
# ✅ doc:portfolio_strategies:001
# ✅ chunk:diversifikation_grundstein:001
# ❌ Doc:Portfolio_Strategies:001 (Großbuchstaben)
# ❌ doc:portfolio-strategies:001 (Bindestrich)
# ❌ doc:portfolio_strategies:1 (zu wenig Ziffern)
```

### Eindeutigkeits-Prüfung
```python
def check_key_uniqueness(generated_keys):
    """Prüft auf versehentliche Duplikate"""
    duplicates = []
    seen = set()
    
    for key in generated_keys:
        if key in seen:
            duplicates.append(key)
        seen.add(key)
    
    return duplicates

def check_cross_type_conflicts(doc_keys, ch_keys, para_keys, subpara_keys, chunk_keys):
    """Prüft auf Konflikte zwischen verschiedenen Key-Typen"""
    all_identifiers = []
    
    for key_list, prefix in [(doc_keys, 'doc'), (ch_keys, 'ch'), 
                            (para_keys, 'para'), (subpara_keys, 'subpara'), 
                            (chunk_keys, 'chunk')]:
        for key in key_list:
            identifier = key.split(':')[1]  # Extrahiert Identifier-Teil
            all_identifiers.append((identifier, prefix))
    
    # Suche nach identischen Identifiern über Typ-Grenzen hinweg
    identifier_map = {}
    for identifier, prefix in all_identifiers:
        if identifier not in identifier_map:
            identifier_map[identifier] = []
        identifier_map[identifier].append(prefix)
    
    conflicts = {k: v for k, v in identifier_map.items() if len(v) > 1}
    return conflicts
```

### Performance-Optimierung
```python
def optimize_key_length(identifier, max_length=30):
    """Intelligente Kürzung unter Erhaltung der Eindeutigkeit"""
    if len(identifier) <= max_length:
        return identifier
    
    # Strategie 1: Entferne Vokale (außer am Anfang)
    consonants_only = identifier[0] + ''.join(c for c in identifier[1:] if c not in 'aeiou_')
    if len(consonants_only) <= max_length:
        return consonants_only
    
    # Strategie 2: Nehme erste N Zeichen
    return identifier[:max_length]

# Beispiele:
# "portfolio_management_risk_assessment_strategies" (47 chars)
# → "prtfl_mngmnt_rsk_ssssmnt_strtgs" (32 chars, zu lang)
# → "portfolio_management_risk_asse" (30 chars, perfekt)
```

## Fehlerbehandlung

### Ungültige Input-Texte
```python
def handle_invalid_input(text):
    """Behandlung von problematischen Inputs"""
    if not text or text.isspace():
        return f"empty_content_{generate_timestamp()}"
    
    if len(text.strip()) < 3:
        return f"short_content_{generate_timestamp()}"
    
    # Nur Sonderzeichen
    cleaned = re.sub(r'[^a-zA-Z0-9äöüß\s]', '', text)
    if not cleaned.strip():
        return f"special_chars_only_{generate_timestamp()}"
    
    return text

def generate_timestamp():
    """Eindeutige Zeitstempel für Fallback-Keys"""
    import datetime
    return datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
```

### Kollisions-Recovery
```python
def resolve_key_collision(base_key, existing_keys, max_attempts=1000):
    """Robuste Kollisions-Auflösung"""
    prefix, identifier, _ = base_key.split(':')
    
    for attempt in range(1, max_attempts + 1):
        candidate_key = f"{prefix}:{identifier}:{attempt:03d}"
        if candidate_key not in existing_keys:
            return candidate_key
    
    # Fallback: Verwende Timestamp
    timestamp = generate_timestamp()
    fallback_key = f"{prefix}:{identifier}_{timestamp}:001"
    return fallback_key
```

## Best Practices

### Do's ✅
- Konsistente Kleinschreibung verwenden
- Unterstriche statt Bindestriche oder Leerzeichen
- Bedeutungsvolle Identifier wählen (nicht "text1", "content2")
- 3-stellige Nummerierung für Sortierfähigkeit
- Stopp-Wörter für bessere Chunk-Keys entfernen

### Don'ts ❌
- Großbuchstaben in Keys verwenden
- Sonderzeichen (außer : und _ an erlaubten Stellen)
- Sehr kurze oder nichtssagende Identifier
- Hardcodierte Nummern ohne System
- Key-Duplikate ignorieren

### Debugging-Tools
```python
def analyze_key_quality(keys):
    """Analysiert Qualität der generierten Keys"""
    stats = {
        'total_keys': len(keys),
        'average_length': sum(len(k) for k in keys) / len(keys),
        'format_violations': [],
        'length_violations': [],
        'duplicates': []
    }
    
    for key in keys:
        if not validate_key_format(key):
            stats['format_violations'].append(key)
        
        if len(key) > 50:  # Redis-Key-Limit beachten
            stats['length_violations'].append(key)
    
    stats['duplicates'] = check_key_uniqueness(keys)
    return stats
```