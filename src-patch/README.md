# Patch: Bisektionsverfahren für die Energie→Temperatur-Umrechnung

`thermoI.H` ersetzt die originale Datei aus OpenFOAM 13:

```
$FOAM_INST_DIR/openfoam13/src/thermophysicalModels/specie/thermo/thermo/thermoI.H
```

**Änderung:** Die Newton-Methode in `T(...)` (original Zeilen 195–269) ist durch
das **Bisektionsverfahren** ersetzt. Wie in der Projektbeschreibung gefordert,
wird das Intervall `[T_low, T_high] = [50 K, 500 K]` schrittweise halbiert,
bis die Intervallbreite kleiner als `tol_ * T0` ist (`tol_` und `maxIter_`
aus `thermo.C`, Default `tol_ = 1e-4`, `maxIter_ = 100`). Bei überkritischem
Druck ist `Cp/Cv > 0`, d. h. `h(T)` bzw. `e(T)` streng monoton — die Bisektion
ist daher immer stabil, auch nahe am kritischen Punkt (Betriebspunkt B).
Signatur und Aufrufer (`THs`, `Tha`, `Tes`, `Tea`) sind unverändert.

## Anwendung

```sh
# 1. Original sichern und ersetzen
cd $FOAM_INST_DIR/openfoam13/src/thermophysicalModels/specie/thermo/thermo
cp thermoI.H thermoI.H.orig
cp <dieses-repos>/src-patch/thermoI.H thermoI.H

# 2. Thermophysikalische Bibliothek neu kompilieren
cd $FOAM_INST_DIR/openfoam13/src/thermophysicalModels
wmake -j4
```

Danach Betriebspunkt B starten (`operatingPointB/Allrun`). Zum Vergleich
Newton vs. Bisektion (Laufzeit, Betriebspunkt A) vorher mit der originalen
`thermoI.H.orig` zurückbauen und ebenfalls `wmake` ausführen.
