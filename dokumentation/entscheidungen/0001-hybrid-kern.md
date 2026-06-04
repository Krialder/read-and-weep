# 0001: Zentraler Kern, vorbereitet auf Föderation

**Status:** akzeptiert

## Kontext

Ein zentraler Betreiber schützt Metadaten am besten, weil alles durch eine Hand läuft. Ein föderiertes Netz aus vielen Servern ist offener und unabhängiger, gibt dafür über jede Servergrenze hinweg Metadaten preis und ist im Betrieb deutlich komplexer. Die Frage war, womit man anfängt, ohne sich den anderen Weg ganz zu verbauen.

## Entscheidung

Wir bauen einen zentralen Kern und benennen Identitäten intern als `handle@home-server`, auch bei nur einem Server. Eine Server-zu-Server-Schnittstelle ist grob umrissen, aber abgeschaltet.

## Konsequenzen

Der Betrieb bleibt am Anfang beherrschbar, und der Metadatenschutz ist so stark, wie er nur bei einem Betreiber sein kann.

Ehrlich zur Reichweite dieser Vorbereitung: Sie de-riskt die Benennung, mehr nicht. Echte Föderation ist keine Erweiterung, sondern eine andere Vertrauenstopologie. Sie braucht serverübergreifende Key Transparency, ein Server-Discovery-Protokoll, S2S-Authentifizierung, und sie weicht den Metadatenschutz auf. Der frühere Eindruck, das wäre später "nur ein Schalter", stimmt nicht. Deshalb bleibt Föderation eine bewusste Option für später und ist standardmäßig aus.
