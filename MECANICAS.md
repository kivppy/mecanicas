# 🌙 MOON BOT — Documentación Técnica de Mecánicas (Actualizada)

Documento de referencia interno: explica **cómo funciona cada sistema por dentro** (flujo, integración y lógica real), alineado con el estado actual del bot.

---

## 📁 1. Arquitectura de persistencia

* Persistencia en **JSON plano** dentro de `data/`
* Acceso mediante:

  * `storage_load`
  * `storage_save`
* Locks por archivo (`asyncio.Lock`) para evitar corrupción
* No hay base de datos ni migraciones

---

## 💰 2. Economía

### Modelo de usuario

```
{ "moon_coins": float, "usd_wallet": float, "debt": float, "history": [...] }
```

### Flujo principal (`economy_add_coins`)

1. Se aplica corte de deuda (10%–40%)
2. Se acredita el resto
3. Se registra en historial
4. Se dispara sistema de logros

---

### Préstamos

* Rango: 200 – 100.000
* Interés: +10%
* Pago automático por ingresos futuros

---

### Mercado MOON COIN

* Precio dinámico con sistema de **episodios**
* Cambios cada 5 min (±1 a ±500)
* Rebote en piso (1) y techo (100.000)

---

## 🎰 3. Casino (Sistema Principal)

El bot ahora gira alrededor del **casino como núcleo económico**.

### Integración global

Todas las apuestas:

* Usan cooldown antispam
* Aportan al jackpot (`casino_jackpot_add`)
* Se registran en ranking (`casino_track_result`)
* Aplican bonus (`bet_win_bonus`)

---

## 🃏 3.1 Blackjack

* Pago:

  * Win → x1
  * Blackjack → x1.5
  * Push → 0 (pero cuenta como partida)
* Dealer pide hasta 17
* 1v1:

  * Pozo compartido
  * Timeout = derrota automática
  * Ahora SIEMPRE registra resultado y aporta al jackpot

---

## 🎡 3.2 Ruleta

* Tipos: número, color, paridad, mitad, docena
* Multiplicadores clásicos
* Booster:

  * Apuesta ≥ 400 → x5 fijo
* Animación previa al resultado

---

## 🎰 3.3 Tragamonedas

* Sistema por pesos (raridad de símbolos)
* Triple → multiplicador alto
* Doble → x1.5
* Integrado a jackpot y ranking

---

## 🆕 3.4 Carrera de caballos

* 6 caballos:

  * Probabilidad distinta
  * Multiplicador distinto
* Animación en vivo
* Balanceado (EV controlado, ningún caballo roto)

---

## 🆕 3.5 Pelea callejera

* 2 peleadores aleatorios
* 50/50 → x1.9
* Narración dinámica antes del resultado

---

## 💰 3.6 Jackpot (`/jackpot`)

* Pozo global acumulado
* Alimentado por rake de todas las apuestas
* Visible en tiempo real

---

## 📊 3.7 Ranking Ludopatía

* Registra:

  * Total ganado
  * Total perdido
  * Cantidad de jugadas
* Incluye:

  * Blackjack (incluyendo empates)
  * Ruleta
  * Tragamonedas
  * Nuevos juegos

---

## ⚙️ 3.8 Correcciones clave

* Blackjack 1v1 ahora SIEMPRE impacta ranking y jackpot
* Empates cuentan como jugadas
* Sistema sin huecos ni casos perdidos

---

## 💼 4. Trabajos

Se mantienen como sistema secundario:

### Legales

* Ganancia baja/moderada
* Cooldown corto

### Ilegales

* Riesgo vs recompensa
* Penalización por fallo

---

## 🎒 5. Ítems

### Ítems de trabajo

* Drops por probabilidad
* Venta directa

### Mini Shop

Categorías:

* Trading
* Trabajo legal
* Trabajo ilegal
* Apuestas

👉 Solo 1 ítem por categoría equipado

---

## 🎰 5.1 Ítems de apuestas

Afectan directamente el casino:

* Aumentan ganancia al ganar
* No afectan probabilidad

---

## 🤖 6. Autobots

* Ejecutan compra/venta automática
* Usan ticks del mercado
* Consumen usos por operación

---

## 🏢 7. Negocios

* Generan ingresos pasivos
* Cobro manual o automático

---

## 🆕 7.1 Auto Cobrador

* Cobra negocios automáticamente
* Precio dinámico
* Intervalo configurable (mínimo 60 min)

---

## ❌ 8. Sistemas eliminados

Los siguientes sistemas fueron removidos completamente:

* Pandillas
* Misiones grupales
* Golpes
* Restricciones de apuestas

👉 Ya no forman parte del flujo del bot

---

## 📊 9. Balance general

* Casino = núcleo principal
* Economía = soporte
* Trabajos / negocios = secundarios
* Sistema unificado y consistente

---

## ✔️ Estado actual

* Sistema estable
* Sin inconsistencias internas
* Integración completa entre:

  * apuestas
  * ranking
  * jackpot
  * economía

---

## 🧠 Filosofía del sistema

Menos sistemas dispersos
Más enfoque en un núcleo fuerte

👉 Todo gira alrededor del riesgo, recompensa y progresión económica
