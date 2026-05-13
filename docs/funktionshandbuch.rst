Ausführlicher Funktionsleitfaden
================================

Diese Seite dient als Einstieg in die vollständige FEMAGTools-Dokumentation.
Sie fasst die wichtigsten Funktionsbereiche zusammen, verweist auf bestehende
Beispiele im Repository und ergänzt die Benutzerdokumentation um eine
vollständige API-Referenz, die im PDF automatisch mit ausgegeben wird.

Ziel und Aufbau
---------------

Die Dokumentation besteht aus drei Ebenen:

* **Anwendungsorientierte Kapitel** in diesem Handbuch und in den bestehenden
  Benutzerseiten wie :doc:`intro`, :doc:`femag`, :doc:`models` oder
  :doc:`windings`.
* **Praktische Beispiele** aus ``examples/`` und ``notebooks/``.
* **Vollständige API-Referenz** für Module, Funktionen, Klassen und Methoden aus
  dem Python-Paket ``femagtools``.

Schnellstart
------------

Installation mit allen optionalen Abhängigkeiten::

   pip install 'femagtools[all]'

Typischer Einstieg in Python::

   import femagtools

   print(femagtools.__version__)

FEMAG-Simulation aus einem Arbeitsverzeichnis starten::

   import pathlib
   import femagtools

   workdir = pathlib.Path.home() / "femag"
   femag = femagtools.Femag(workdir)
   femag.run("femag.fsl")

Wichtige Funktionsbereiche
--------------------------

.. list-table::
   :header-rows: 1
   :widths: 25 30 25 20

   * - Bereich
     - Zentrale Module
     - Typische Aufgaben
     - Weiterführende Inhalte
   * - FEMAG aus Python steuern
     - ``femagtools``, ``femagtools.femag``, ``femagtools.fsl``, ``femagtools.model``
     - Modelle aufbauen, FSL erzeugen, Rechnungen lokal oder remote starten
     - :doc:`intro`, :doc:`femag`, ``notebooks/1-ModelsAndFemag.ipynb``
   * - Material- und Kennliniendaten
     - ``femagtools.mcv``, ``femagtools.tks``, ``femagtools.jhb``, ``femagtools.losscoeffs``
     - Magnetisierungskurven lesen, schreiben und auswerten
     - :doc:`models`, ``notebooks/2-MaterialHandling.ipynb``
   * - FEMAG-Ergebnisse lesen
     - ``femagtools.bch``, ``femagtools.erg``, ``femagtools.airgap``, ``femagtools.forcedens``, ``femagtools.isa7``, ``femagtools.nc``, ``femagtools.vtu``
     - BCH-, ERG-, Airgap-, PLT-, ISA7- und NetCDF-Dateien auswerten
     - :doc:`bchreader`, :doc:`ncisa`, :doc:`forcedens`, ``notebooks/4-BchFileReader.ipynb``
   * - Maschinenmodelle und Auslegung
     - ``femagtools.machine``, ``femagtools.machine.pm``, ``femagtools.machine.sm``, ``femagtools.machine.im``, ``femagtools.machine.afpm``, ``femagtools.machine.sizing``
     - Ersatzmodelle, Kennfelder, Wirkungsgradkarten und Vorauslegung
     - :doc:`sizing`, ``notebooks/5-MachineCharacteristicsEfficiencyMaps.ipynb``
   * - Wicklungen
     - ``femagtools.windings``
     - Wicklungen erzeugen, analysieren und visualisieren
     - :doc:`windings`, ``notebooks/3-Windings.ipynb``
   * - Variantenstudien und Optimierung
     - ``femagtools.parstudy``, ``femagtools.grid``, ``femagtools.opt``, ``femagtools.moo``, ``femagtools.dakota``
     - Parameterstudien, DoE und Mehrzieloptimierung
     - :doc:`dakota`, ``notebooks/7-ParameterVariationAndOptimization.ipynb``
   * - Geometrie-Konvertierung
     - ``femagtools.dxfsl``, ``femagtools.svgfsl``, ``femagtools.convert``, ``femagtools.gmsh``
     - DXF/SVG nach FSL übertragen, Netze und Austauschformate konvertieren
     - :doc:`models`, ``examples/model-creation/``
   * - Plotting und Nachbearbeitung
     - ``femagtools.plot``, ``femagtools.ts``, ``femagtools.amela``, ``femagtools.tspost``
     - Diagramme, Post-Processing, Verlust- und Temperaturauswertung
     - :doc:`amela`, :doc:`tspost`, ``notebooks/6-MagnetLosses.ipynb``
   * - Ausführungsumgebungen
     - ``femagtools.multiproc``, ``femagtools.zmq``, ``femagtools.condor``, ``femagtools.amazon``, ``femagtools.google``, ``femagtools.docker``
     - Lokale Parallelisierung, ZMQ, Cloud- und Cluster-Läufe
     - :doc:`engine`, ``examples/docker-zmq/README``

Typische Arbeitsabläufe mit Beispielen
--------------------------------------

BCH-Datei lesen und Kennwerte ausgeben::

   import femagtools

   bch = femagtools.read_bchfile("TEST_001.BCH")
   print(bch.machine["torque"])

FSL aus einem Maschinenmodell erzeugen::

   import femagtools

   machine = {
       "name": "PM 130 L4",
       "lfe": 0.1,
       "poles": 4,
       "outer_diam": 0.13,
       "bore_diam": 0.07,
       "inner_diam": 0.015,
       "airgap": 0.001,
       "stator": {
           "num_slots": 12,
           "mcvkey_yoke": "dummy",
           "rlength": 1.0,
           "stator1": {
               "slot_rf1": 0.057,
               "tip_rh1": 0.037,
               "tip_rh2": 0.037,
               "tooth_width": 0.009,
               "slot_width": 0.003,
           },
       },
       "magnet": {
           "mcvkey_shaft": "dummy",
           "mcvkey_yoke": "dummy",
           "magnetSector": {
               "magn_num": 1,
               "magn_width_pct": 0.8,
               "magn_height": 0.004,
               "magn_type": 1,
               "condshaft_r": 0.02,
               "magn_ori": 2,
               "magn_len": 1.0,
           },
       },
       "windings": {
           "num_phases": 3,
           "num_wires": 100,
           "coil_span": 3.0,
           "num_layers": 1,
       },
   }

   fsl = femagtools.create_fsl(machine)

Maschine analytisch auslegen::

   import femagtools

   machine = femagtools.machine.sizing.spm(
       p2=1.5e3,
       speed=1500/60,
       p=4,
       udc=550)

Parameterstudie vorbereiten::

   import pathlib
   import femagtools

   study = femagtools.parstudy.List(
       workdir=pathlib.Path("parstudy"))

Weitere Beispiele im Repository
-------------------------------

Für nahezu jeden Funktionsbereich gibt es bereits ausführbare Beispiele:

* ``examples/calculation/`` – FEMAG-Läufe mit Eingabedateien
* ``examples/model-creation/`` – Modell- und Geometrieerzeugung
* ``examples/parameter-variation/`` – Variantenstudien
* ``examples/optimization/`` – Optimierungsabläufe
* ``examples/bch-erg/`` – Ergebnisdateien einlesen
* ``examples/mcv/`` und ``examples/magnetcurves/`` – Materialdaten

Die Notebooks liefern zusätzlich schrittweise Erläuterungen:

* ``notebooks/0-Introduction.ipynb``
* ``notebooks/1-ModelsAndFemag.ipynb``
* ``notebooks/2-MaterialHandling.ipynb``
* ``notebooks/3-Windings.ipynb``
* ``notebooks/4-BchFileReader.ipynb``
* ``notebooks/5-MachineCharacteristicsEfficiencyMaps.ipynb``
* ``notebooks/6-MagnetLosses.ipynb``
* ``notebooks/7-ParameterVariationAndOptimization.ipynb``
* ``notebooks/8-IronLosses.ipynb``
* ``notebooks/9-Eccentricity-Analysis.ipynb``
* ``notebooks/VTK-Postprocessing.ipynb``

Konsolenskripte
---------------

Die folgenden Skripte stehen nach der Installation über ``pip`` zur Verfügung:

* ``femagtools-plot`` – grafischer Bericht aus BCH/BATCH-Dateien
* ``femagtools-convert`` – Konvertierung verschiedener Mesh-Formate
* ``femagtools-bchxml`` – BCH/BATCH nach XML umwandeln
* ``femagtools-dxfsl`` – DXF nach FSL konvertieren
* ``femagtools-svgfsl`` – SVG nach FSL konvertieren

Vollständige API-Referenz
-------------------------

Die PDF-Ausgabe enthält nach den Handbuchkapiteln eine automatisch erzeugte
Referenz für das gesamte Paket ``femagtools``. Dort sind alle im Paket
enthaltenen Module, Funktionen, Klassen und Methoden samt Docstrings
aufgelistet.
