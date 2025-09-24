Perfetto! Ti mostro come creare un **algoritmo in Kotlin** che calcola la **direzione da seguire (in gradi)** tra tre punti GPX e restituisce **una freccia** (es. con una rotazione) per indicare la **direzione del percorso**.

---

## 🎯 Obiettivo:

Dati 3 punti GPX:

- `A` = punto precedente
    
- `B` = punto attuale
    
- `C` = punto successivo
    

Vogliamo:

- Calcolare la **direzione** tra `B → C` (la direzione da seguire)
    
- Calcolare eventualmente l'**angolo rispetto a `A → B`** per mostrare una **freccia orientata**
    

---

## ✅ Algoritmo Kotlin

```kotlin
import kotlin.math.*

data class GpxPoint(val lat: Double, val lon: Double)

// Calcola bearing (azimut) tra due punti GPS (in gradi)
fun bearing(from: GpxPoint, to: GpxPoint): Double {
    val lat1 = Math.toRadians(from.lat)
    val lon1 = Math.toRadians(from.lon)
    val lat2 = Math.toRadians(to.lat)
    val lon2 = Math.toRadians(to.lon)

    val dLon = lon2 - lon1
    val y = sin(dLon) * cos(lat2)
    val x = cos(lat1) * sin(lat2) - sin(lat1) * cos(lat2) * cos(dLon)
    var brng = atan2(y, x)

    brng = Math.toDegrees(brng)
    return (brng + 360) % 360  // normalizza tra 0-360°
}

// Calcola differenza angolare tra due direzioni (utile per orientare freccia)
fun angleDifference(startBearing: Double, endBearing: Double): Double {
    val diff = (endBearing - startBearing + 540) % 360 - 180
    return diff  // Positivo = svolta a destra, negativo = sinistra
}
```

---

## 🧭 Esempio d’uso:

```kotlin
val puntoA = GpxPoint(45.123, 7.123)
val puntoB = GpxPoint(45.124, 7.124)
val puntoC = GpxPoint(45.125, 7.128)

val dirAB = bearing(puntoA, puntoB) // direzione attuale
val dirBC = bearing(puntoB, puntoC) // direzione da seguire
val svolta = angleDifference(dirAB, dirBC)

println("Direzione attuale: $dirAB°")
println("Direzione da seguire: $dirBC°")
println("Angolo di svolta: $svolta°")
```

---

## 🧭 Interpretazione:

- `svolta ≈ 0°`: andare dritto
    
- `svolta > 15°`: curva a destra
    
- `svolta < -15°`: curva a sinistra
    

---

## 🎯 Freccia grafica (concetto)

Nel tuo layout puoi ruotare una freccia (es. `ImageView`) con:

```kotlin
imageView.rotation = dirBC.toFloat()
```

Oppure, se stai disegnando una freccia su `Canvas`, ruoti il canvas o la path in base a `dirBC`.

---

Vuoi un esempio di `ImageView` con rotazione dinamica della freccia in base al bearing?