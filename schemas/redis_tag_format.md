# Redis-Tag-Format für strukturierte Text-Transformation

## Grundprinzipien

Das Redis-Tag-System transformiert hierarchische Markdown-Dokumente in Redis-kompatible Key-Value-Strukturen mit vollständiger Erhaltung der ursprünglichen Text-Hierarchie und -Sequenz.

### Design-Philosophie
- **Key-basierte Identifikation** statt UUID-System
- **Flache Redis-Strukturen** für optimale Performance
- **Explizite Sequenz-Erhaltung** durch position-Felder
- **Hierarchie-Bewahrung** durch parent-Referenzen
- **Volltext-Preservation** - niemals Text kürzen oder zusammenfassen

## Tag-Typen Übersicht

### Document-Level Tags
```
{RedisDoc: key={doc_key} ; title="{title}" ; author="{author}" ; created="{date}" ; total_chunks={count} ; category="{category}" ; language="{lang}"}
```

### Content-Level Tags  
```
{RedisChunk: key={chunk_key} ; parent={parent_key} ; text="{complete_text}" ; level="{hierarchy_level}" ; position={sequence_number} ; title="{heading_title}"}
```

### Relationship-Level Tags
```
{RedisSet: key={parent_key}:{relationship_type} ; members=[{child_key1}, {child_key2}, ...]}
```

## Document-Tags

### Format-Spezifikation
```
{RedisDoc: key=doc:{clean_title}:{number} ; title="{original_title}" ; author="{author_name}" ; created="{yyyy-mm-dd}" ; total_chunks={chunk_count} ; category="{category}" ; language="{lang_code}"}
```

### Beispiele
```
{RedisDoc: key=doc:portfolio_management_strategies:001 ; title="Portfolio Management Strategies" ; author="Oskar Sch." ; created="2024-12-27" ; total_chunks=47 ; category="investment" ; language="de"}

{RedisDoc: key=doc:communication_rules_complete:001 ; title="Communication Rules - Complete Guide" ; author="Oskar Sch." ; created="2024-12-27" ; total_chunks=156 ; category="communication" ; language="de"}

{RedisDoc: key=doc:risk_assessment_framework:001 ; title="Risk Assessment Framework" ; author="Team Lead" ; created="2024-12-27" ; total_chunks=89 ; category="risk_management" ; language="en"}
```

### Pflichtfelder
- **key**: Redis-Key im Format `doc:{clean_title}:{number}`
- **title**: Originaler Dokumententitel aus YAML Front Matter
- **author**: Autor aus YAML Front Matter
- **created**: Erstellungsdatum im Format YYYY-MM-DD
- **total_chunks**: Anzahl aller generierten Chunks (wird automatisch gezählt)

### Optionale Felder
- **category**: Thematische Kategorie aus YAML
- **language**: Sprach-Code (Standard: "de")
- **tags**: Array von Such-Tags (wird zu separatem Redis-Set)

## Content-Chunk-Tags

### Hierarchie-Ebenen
1. **chapter**: Hauptkapitel (# Headers)
2. **paragraph**: Unterkapitel (## Headers)  
3. **subparagraph**: Unter-Unterkapitel (### Headers)
4. **chunk**: Textblöcke ohne Header

### Chapter-Level-Tags
```
{RedisChunk: key=ch:{topic_keyword}:{number} ; parent=doc:{doc_id} ; text="{complete_chapter_text}" ; level="chapter" ; position={doc_position} ; title="{chapter_title}" ; chapter_number={ch_num}}
```

**Beispiel:**
```
{RedisChunk: key=ch:portfolio_theory_fundamentals:001 ; parent=doc:portfolio_management_strategies:001 ; text="# Portfolio Theory Fundamentals\n\nDiversifikation ist der Grundstein erfolgreicher Investments. Ein ausgewogenes Portfolio reduziert das Gesamtrisiko durch geschickte Verteilung der Investments auf verschiedene Anlageklassen.\n\nModerne Portfoliotheorie nach Markowitz zeigt mathematisch, dass durch optimale Kombination verschiedener Assets das Gesamtrisiko unter das Risiko der einzelnen Komponenten gesenkt werden kann." ; level="chapter" ; position=1 ; title="Portfolio Theory Fundamentals" ; chapter_number=1}
```

### Paragraph-Level-Tags
```
{RedisChunk: key=para:{topic_keyword}:{number} ; parent=ch:{chapter_id} ; text="{complete_paragraph_text}" ; level="paragraph" ; position={doc_position} ; title="{paragraph_title}" ; section_number="{section_num}"}
```

**Beispiel:**
```
{RedisChunk: key=para:risk_assessment_methods:001 ; parent=ch:portfolio_theory_fundamentals:001 ; text="## Risk Assessment Methods\n\nRisiko kann durch verschiedene Metriken gemessen werden. Die wichtigsten Kennzahlen umfassen Volatilität, Value at Risk und Korrelationsanalysen zwischen Assets." ; level="paragraph" ; position=2 ; title="Risk Assessment Methods" ; section_number="1.1"}
```

### SubParagraph-Level-Tags
```
{RedisChunk: key=subpara:{topic_keyword}:{number} ; parent=para:{paragraph_id} ; text="{complete_subparagraph_text}" ; level="subparagraph" ; position={doc_position} ; title="{subparagraph_title}" ; subsection_number="{subsection_num}"}
```

**Beispiel:**
```
{RedisChunk: key=subpara:volatility_measures:001 ; parent=para:risk_assessment_methods:001 ; text="### Volatility Measures\n\nVolatilität zeigt die Schwankungsbreite von Assets über einen bestimmten Zeitraum. Standard Deviation ist die häufigste Volatilitätsmessung." ; level="subparagraph" ; position=3 ; title="Volatility Measures" ; subsection_number="1.1.1"}
```

### Chunk-Level-Tags (Textblöcke)
```
{RedisChunk: key=chunk:{main_concept}:{number} ; parent={parent_id} ; text="{complete_sentence_or_paragraph}" ; level="chunk" ; position={doc_position} ; sequence_in_parent={parent_sequence}}
```

**Beispiele:**
```
{RedisChunk: key=chunk:standard_deviation_definition:001 ; parent=subpara:volatility_measures:001 ; text="Standard Deviation ist die häufigste Volatilitätsmessung und zeigt die durchschnittliche Abweichung der Renditen vom Mittelwert." ; level="chunk" ; position=4 ; sequence_in_parent=1}

{RedisChunk: key=chunk:value_at_risk_complement:002 ; parent=subpara:volatility_measures:001 ; text="Value at Risk (VaR) ergänzt die Standardabweichung, indem es das maximale Verlustpotential bei gegebener Wahrscheinlichkeit quantifiziert." ; level="chunk" ; position=5 ; sequence_in_parent=2}
```

## Relationship-Tags (Redis-Sets)

### Hierarchie-Sets
```
{RedisSet: key={parent_key}:children ; members=[{child_key1}, {child_key2}, ...]}
```

**Beispiele:**
```
{RedisSet: key=doc:portfolio_management_strategies:001:chapters ; members=[ch:portfolio_theory_fundamentals:001, ch:practical_implementation:002, ch:risk_management_advanced:003]}

{RedisSet: key=ch:portfolio_theory_fundamentals:001:children ; members=[para:risk_assessment_methods:001, para:modern_portfolio_theory:002, para:diversification_strategies:003]}

{RedisSet: key=para:risk_assessment_methods:001:children ; members=[subpara:volatility_measures:001, subpara:correlation_analysis:002]}

{RedisSet: key=subpara:volatility_measures:001:children ; members=[chunk:standard_deviation_definition:001, chunk:value_at_risk_complement:002]}
```

### Sequenz-Sets (Lese-Reihenfolge)
```
{RedisSet: key={scope_key}:sequence ; members=[{element1}, {element2}, {element3}, ...]}
```

**Beispiele:**
```
{RedisSet: key=doc:portfolio_management_strategies:001:sequence ; members=[ch:portfolio_theory_fundamentals:001, para:risk_assessment_methods:001, subpara:volatility_measures:001, chunk:standard_deviation_definition:001, chunk:value_at_risk_complement:002, subpara:correlation_analysis:002, para:modern_portfolio_theory:002, ch:practical_implementation:002]}

{RedisSet: key=ch:portfolio_theory_fundamentals:001:sequence ; members=[para:risk_assessment_methods:001, subpara:volatility_measures:001, chunk:standard_deviation_definition:001, chunk:value_at_risk_complement:002, subpara:correlation_analysis:002, para:modern_portfolio_theory:002]}
```

### Navigation-Sets
```
{RedisSet: key={element_key}:next ; members=[{next_element}]}
{RedisSet: key={element_key}:previous ; members=[{previous_element}]}
{RedisSet: key={element_key}:siblings ; members=[{sibling1}, {sibling2}, ...]}
```

**Beispiele:**
```
{RedisSet: key=chunk:standard_deviation_definition:001:next ; members=[chunk:value_at_risk_complement:002]}
{RedisSet: key=chunk:value_at_risk_complement:002:previous ; members=[chunk:standard_deviation_definition:001]}
{RedisSet: key=chunk:value_at_risk_complement:002:next ; members=[subpara:correlation_analysis:002]}

{RedisSet: key=subpara:volatility_measures:001:siblings ; members=[subpara:correlation_analysis:002]}
{RedisSet: key=ch:portfolio_theory_fundamentals:001:siblings ; members=[ch:practical_implementation:002, ch:risk_management_advanced:003]}
```

### Kategorisierung-Sets
```
{RedisSet: key=category:{category_name}:docs ; members=[{doc_key1}, {doc_key2}, ...]}
{RedisSet: key=author:{author_clean}:docs ; members=[{doc_key1}, {doc_key2}, ...]}
{RedisSet: key=tag:{tag_name}:chunks ; members=[{chunk_key1}, {chunk_key2}, ...]}
```

**Beispiele:**
```
{RedisSet: key=category:investment:docs ; members=[doc:portfolio_management_strategies:001, doc:risk_assessment_framework:001]}
{RedisSet: key=author:oskar_sch:docs ; members=[doc:portfolio_management_strategies:001, doc:communication_rules_complete:001]}
{RedisSet: key=tag:diversification:chunks ; members=[chunk:standard_deviation_definition:001, chunk:portfolio_balance:003]}
```

## Key-Naming-Konventionen

### Dokument-Keys
```
Format: doc:{clean_title}:{3-digit-number}
Beispiele: 
- doc:portfolio_management_strategies:001
- doc:complete_investment_guide:001  
- doc:risk_assessment_methods:001
```

### Content-Keys nach Hierarchie
```
Chapters:     ch:{main_topic}:{3-digit-number}
Paragraphs:   para:{subtopic}:{3-digit-number} 
SubParagraphs: subpara:{specific_topic}:{3-digit-number}
Chunks:       chunk:{main_concept}:{3-digit-number}
```

### Key-Bereinigungsregeln
1. **Lowercase-Transformation**: Alle Großbuchstaben → klein
2. **Space-Replacement**: Leerzeichen → Unterstriche
3. **Special-Character-Removal**: Sonderzeichen entfernen
4. **Umlaut-Conversion**: ä→ae, ö→oe, ü→ue, ß→ss
5. **Length-Limitation**: Maximal 30 Zeichen für Topic-Teil
6. **Uniqueness-Guarantee**: Automatische Nummerierung bei Duplikaten

### Bereinigung-Beispiele
```
"Portfolio Management & Risk Assessment" 
→ "portfolio_management_risk_assessment"
→ doc:portfolio_management_risk_assessment:001

"Diversifikation: Der Schlüssel zum Erfolg"
→ "diversifikation_der_schluessel_zum_erfolg"  
→ chunk:diversifikation_der_schluessel:001

"Value at Risk (VaR) - Mathematical Foundation"
→ "value_at_risk_var_mathematical_foundation"
→ chunk:value_at_risk_var_mathematical:001
```

## Volltext-Preservation-Regeln

### KRITISCHE REGEL: Niemals Text kürzen
```
{RedisChunk: text="{KOMPLETTER_ORIGINALTEXT_OHNE_JEDE_KÜRZUNG_ODER_ZUSAMMENFASSUNG}"}
```

### Beispiel für korrekte Volltext-Übernahme
**Markdown-Input:**
```markdown
### Diversifikation und Portfolio-Balance

Diversifikation ist der Grundstein erfolgreicher Investments. Ein ausgewogenes Portfolio reduziert das Gesamtrisiko durch geschickte Verteilung der Investments auf verschiedene Anlageklassen. Moderne Portfoliotheorie nach Markowitz zeigt mathematisch, dass durch optimale Kombination verschiedener Assets das Gesamtrisiko unter das Risiko der einzelnen Komponenten gesenkt werden kann.

Praktische Umsetzung erfordert systematische Analyse der Korrelationen zwischen verschiedenen Anlageklassen.
```

**Korrekter Redis-Tag:**
```
{RedisChunk: key=subpara:diversifikation_portfolio_balance:001 ; parent=para:portfolio_theory:001 ; text="### Diversifikation und Portfolio-Balance\n\nDiversifikation ist der Grundstein erfolgreicher Investments. Ein ausgewogenes Portfolio reduziert das Gesamtrisiko durch geschickte Verteilung der Investments auf verschiedene Anlageklassen. Moderne Portfoliotheorie nach Markowitz zeigt mathematisch, dass durch optimale Kombination verschiedener Assets das Gesamtrisiko unter das Risiko der einzelnen Komponenten gesenkt werden kann.\n\nPraktische Umsetzung erfordert systematische Analyse der Korrelationen zwischen verschiedenen Anlageklassen." ; level="subparagraph" ; position=5 ; title="Diversifikation und Portfolio-Balance"}
```

### Chunk-Granularität-Regeln
1. **Chapter-Level**: Komplettes Kapitel mit allen Unter-Inhalten
2. **Paragraph-Level**: Kompletter Paragraph mit allen Unter-Inhalten  
3. **SubParagraph-Level**: Kompletter SubParagraph mit allen Unter-Inhalten
4. **Chunk-Level**: Einzelne Absätze oder zusammenhängende Sätze

## Position-System für Sequenz-Erhaltung

### Dokumenten-weite Position-Nummerierung
Alle Elemente eines Dokuments erhalten aufsteigende position-Nummern:

```
position=1:  ch:portfolio_theory_fundamentals:001
position=2:  para:risk_assessment_methods:001  
position=3:  subpara:volatility_measures:001
position=4:  chunk:standard_deviation_definition:001
position=5:  chunk:value_at_risk_complement:002
position=6:  subpara:correlation_analysis:002
position=7:  chunk:correlation_benefits:003
position=8:  para:modern_portfolio_theory:002
position=9:  ch:practical_implementation:002
```

### Parent-interne Sequenz-Nummerierung
Zusätzlich erhalten Kinder sequentielle Nummern innerhalb ihres Parents:

```
subpara:volatility_measures:001:
  - chunk:standard_deviation_definition:001 (sequence_in_parent=1)
  - chunk:value_at_risk_complement:002 (sequence_in_parent=2)

para:risk_assessment_methods:001:
  - subpara:volatility_measures:001 (sequence_in_parent=1)
  - subpara:correlation_analysis:002 (sequence_in_parent=2)
```

Diese doppelte Nummerierung ermöglicht sowohl dokumenten-weite Navigation als auch lokale Parent-Child-Navigation für optimale RAG-Performance.
