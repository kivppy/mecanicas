# 🌙 MOON BOT V 2.7.9 — Documentación Técnica de Mecánicas

Documento de referencia interno: explica **cómo funciona cada sistema por dentro** (fórmulas, constantes, probabilidades, listeners y flujo de datos), no solo qué comando existe. Basado 1:1 en el código de `luna_bot.py`.

---

## 📁 1. Arquitectura de persistencia

No hay base de datos: todo se guarda en **JSON plano** dentro de `data/<colección>.json`, uno por dominio (`economy.json`, `market.json`, `businesses.json`, etc.).

- `storage_load(name, default)` / `storage_save(name, data)`: leen/escriben el archivo completo cada vez (no hay índices ni queries parciales).
- Concurrencia: `_lock_for(path)` mantiene un diccionario global `asyncio.Lock` por ruta de archivo, para que dos operaciones simultáneas sobre el mismo JSON no se pisen (se serializan, pero **no** hay locking entre distintos JSON relacionados — por eso funciones como `economy_add_coins` y `achievements_check_and_unlock` se llaman en secuencia, no atómicamente).
- Claves siempre como `str(guild_id)` / `str(user_id)` porque JSON no admite claves int.
- No hay migraciones ni versión de esquema: si cambia la forma de un dict, el código usa `.get()` con default para tolerar registros viejos.

---

## 💰 2. Economía (`EconomyCog`, utils `economy_*`)

### 2.1 Modelo de usuario económico
Cada usuario tiene, por servidor:
```
{ "moon_coins": float, "usd_wallet": float, "debt": float, "faction": str, "history": [...] }
```
`history` guarda como máximo los **últimos 50** movimientos (`user["history"][-50:]`), cada uno con `t` (timestamp), `type`, `amount`, y opcionalmente `debt_paid` / `by`.

### 2.2 `economy_add_coins` — el corazón del sistema
Toda ganancia pasa por acá. Orden de operaciones:
1. Si el usuario tiene deuda activa, se aplica **corte automático de deuda** (`_debt_cut`): se descuenta entre **10% y 40%** (aleatorio uniforme) del monto entrante, tope = mínimo entre el corte, el monto y la deuda restante.
2. El resto (`amount - debt_paid`) se acredita al balance.
3. Se registra en el historial.
4. Si el neto es positivo y el motivo no es `"logro_moon_coins"` (para evitar recursión), se dispara `achievements_check_and_unlock` — es decir, **cualquier ingreso de MOON COINS puede destapar un logro de milestone**, en cualquier comando del bot.

### 2.3 Préstamos y deuda
- `/prestamo <cantidad>`: entre 200 y 100.000. Solo se puede tener **un préstamo activo por vez**.
- Deuda generada: `deuda_total = int(cantidad * 1.10)` → interés fijo del **10%**.
- La deuda **no se cobra manualmente**: se descuenta automáticamente de cada ingreso futuro vía `_debt_cut` (10-40% aleatorio por transacción), o se puede saldar manualmente con `/pagardeuda`.

### 2.4 Mercado de MOON COINS — sistema de "episodios de tendencia"
- Precio inicial: **20.0 USD**. Piso absoluto `PRICE_FLOOR = 1`, techo absoluto `PRICE_CEILING = 100.000`.
- `price_task` (loop cada `PRICE_UPDATE_MINUTES = 5`) llama `market_tick_price` por cada guild. **Ya no se mueve por porcentaje**: cada tick suma o resta un **entero aleatorio fijo** entre `PRICE_STEP_MIN = 1` y `PRICE_STEP_MAX = 500` (nunca un %).
- **Episodios**: en vez de sortear el signo (sube/baja) en cada tick de forma independiente, el mercado vive en rachas ("episodios") de varios ticks seguidos donde el precio **solo sube** o **solo baja**. Cada episodio dura entre `PRICE_EPISODE_MIN_TICKS = 3` y `PRICE_EPISODE_MAX_TICKS = 15` ticks; al terminar, se sortea de nuevo la dirección (50/50) y la duración del próximo episodio.
- **Rebote en los topes**: si el precio toca el techo (100.000), el episodio se corta ahí mismo y arranca uno nuevo **obligatoriamente bajista**. Si toca el piso (1), arranca uno nuevo **obligatoriamente alcista** — así el mercado nunca queda pegado en un extremo para siempre.
- Historial de precios: hasta `PRICE_HISTORY_LIMIT = 500` puntos, usado por `/moonchart` (sparkline Unicode con 8 niveles `▂▃▄▅▆▇█` normalizados min-max de los últimos 15 puntos).
- **Compra/venta** (`/comprarmoon`, `/vendermoon`): comisión base `MINISHOP_BASE_TRADE_FEE = 15%`, reducible por ítems de la Mini Shop (categoría *trading*, ver §7). La fórmula:
  - Venta: `usd = cantidad * precio * (1 - fee)`
  - Compra: `coins = (usd * (1 - fee)) / precio`
  - Las MOON COINS obtenidas al comprar también pasan por `_debt_cut` si hay deuda activa.

### 2.5 Alertas aleatorias de inversiones (`InvestmentAlertsCog`)
- `/configurar_alertas_inversiones <canal>` (staff): define el canal donde el bot manda alertas espontáneas simulando subas/caídas del mercado.
- Cada guild tiene programada su próxima alerta en un timestamp aleatorio (`next_alert_at`), separado entre `INVESTMENT_ALERT_MIN_SECONDS = 10 min` y `INVESTMENT_ALERT_MAX_SECONDS = 60 min`. Un `tasks.loop` revisa cada `INVESTMENT_ALERT_CHECK_SECONDS = 30s` si ya se cumplió el plazo.
- Al dispararse, la alerta **ejecuta un tick real** del mismo motor de `market_tick_price` (no es un salto aparte) para que la dirección sea siempre coherente con el episodio de tendencia vigente, dispara también `autobuy_process_guild` (para que los Autobots reaccionen, ver §7.1), y muestra un evento de sabor aleatorio (`INVESTMENT_CRASH_EVENTS` / `INVESTMENT_RISE_EVENTS`) junto al precio anterior/nuevo.
- **Este mismo canal se reutiliza** para las alertas de cambio de precio del ítem Auto Cobrador (ver §9.1).

### 2.6 Transferencias entre usuarios (`/transferir`)
- `/transferir <usuario> <cantidad>`: le manda MOON COINS a otro miembro del servidor, directo de tu balance al suyo (sin comisión).
- Validaciones: cantidad > 0, no podés transferirte a vos mismo, no se le puede transferir a un bot, y necesitás el balance completo disponible (si no alcanza, se rechaza la operación entera, no se hace una transferencia parcial).
- Internamente usa `economy_remove_coins` (motivo `transfer_out`) sobre el emisor y `economy_add_coins` (motivo `transfer_in`) sobre el receptor — por eso el receptor también puede destapar logros de milestone (§14) si el ingreso lo cruza.
- El receptor recibe un aviso por MD (si tiene los DMs abiertos) y el movimiento queda registrado en el canal de logs (`economy_log`) si está configurado.

### 2.7 Ranking y consulta
- `/topmoon`: top 10 por `moon_coins`.
- `/economia`: balance, valor en USD (`balance * price`), wallet USD, deuda (si > 0), checklist de mejoras (`BOOST_DEFS`) y últimos 5 movimientos.

---

## 🎰 3. Casino

### 3.1 Blackjack (`/blackjack`, `/blackjack_1v1`)
- Baraja estándar de 52 cartas (`SUITS × RANKS`), barajada con `random.shuffle`.
- `hand_value`: as vale 11, se recalcula a 1 mientras la mano pase de 21 y queden ases "blandos".
- Modo solitario contra el bot:
  - Blackjack natural (21 con 2 cartas) al repartir → pago **x1.5** automático, sin turno.
  - El dealer pide carta mientras su valor sea `< 17` (regla estándar de casino).
  - Resultados: `win` (paga 1:1), `blackjack` (paga 1.5:1), `lose` (pierde la apuesta), `push` (empate, no hay intercambio).
- Modo 1v1 (`/blackjack_1v1 usuario apuesta`):
  - `ChallengeView`: el retador deposita la apuesta al lanzar el desafío; si el oponente acepta, ambos apuestan a un pozo común (`apuesta * 2`).
  - Turnos alternados (`DuelView`), con manos ocultas hasta que le toca el turno a cada jugador.
  - Empate si ambos se pasan de 21, o si tienen igual valor → se devuelve la apuesta a cada uno.
  - Si alguien no juega su turno en 90s (`timeout`), pierde por abandono y el pozo completo va al rival.

### 3.2 Ruleta (`/ruleta`)
- Números 0-36. Rojos: set fijo de 18 números (`RED_NUMBERS`); 0 es verde.
- Tipos de apuesta y pago:
  | Tipo | Pago | Condición |
  |---|---|---|
  | `numero` (1 a 10 números a la vez) | `36 / cantidad_numeros`, piso x2 | Bola cae en alguno de los elegidos |
  | `color` | x2 | rojo/negro exacto |
  | `paridad` | x2 | par/impar (0 no cuenta) |
  | `mitad` | x2 | 1-18 / 19-36 |
  | `docena` | x3 | 1ª (1-12) / 2ª (13-24) / 3ª (25-36) |
- Si se eligen **varios números**, el costo total es `monto × cantidad_de_números` (cada número "cuesta" el monto por separado), pero el pago se calcula sobre el monto base por número acertado.
- **Bonus Booster**: si la apuesta por número/tipo es ≥ `400` (`BOOSTER_ROULETTE_MIN_BET`) y el usuario es Server Booster (`member.premium_since is not None`) y gana, el multiplicador se **fija en x5** en lugar del multiplicador normal.
- Bonus de ítem de Mini Shop (`bet_win_bonus`, ver §7) se suma como porcentaje extra sobre la ganancia.
- Animación: 6 frames falsos (`SPIN_FRAMES`) con 0.6s de delay antes de mostrar el resultado real.
- Todo resultado (ganancia/pérdida neta) se registra vía `casino_track_result` para el ranking de ludopatía.

### 3.3 Tragamonedas (`/tragamonedas`)
- 7 símbolos con peso relativo y multiplicador si caen los 3 iguales:
  | Símbolo | Peso | Multiplicador (triple) |
  |---|---|---|
  | 🍒 | 30 | x2 |
  | 🍋 | 25 | x3 |
  | 🍇 | 20 | x4 |
  | 🔔 | 12 | x8 |
  | ⭐ | 8 | x15 |
  | 💎 | 4 | x30 |
  | 🪙 (mooncoin) | 1 | x100 |
- Par de símbolos iguales (2 de 3, cualquier posición) paga **x1.5** de consuelo.
- El ítem `bet_win_bonus` de Mini Shop también aplica acá.
- 4 frames de animación falsa antes del giro real.

### 3.4 Ranking de ludopatía (`/ranking_ludopatia`)
- `casino_track_result` acumula por usuario `total_ganado` / `total_perdido` / `jugadas` en `casino_stats.json`, alimentado por blackjack (implícito en pagos), ruleta y tragamonedas.
- Top 10 de cada columna, con medallas 🥇🥈🥉 para el top 3.

---

## 💼 4. Trabajos y facciones (`JobsCog`)

### 4.1 Facciones
- Un usuario tiene **una facción principal** (`mafia`, `policia`, `ninguna`), guardada en `economy.json`.
- Los **Server Boosters** pueden usar `/afiliarse_ambas` para tener MAFIA como principal y POLICÍA como "facción extra" (`extra_factions`), acumulando beneficios de ambas sin las restricciones normales de exclusividad.
- `user_has_faction(gid, uid, faction)` chequea tanto la principal como las extras — es la función que usan los sistemas de mercado negro, ruleta ilegal, etc.

### 4.2 Trabajos legales (`/trabajar`, cooldown 5 min)
6 tipos (`moderar`, `vender`, `programar`, `reparto`, `streaming`, `onlyfans`), cada uno con 3 mensajes de sabor aleatorios. Recompensa base: `random.randint(10, 30)`, multiplicada por:
- `boost_multiplier` (Boost Trabajo Legal, +50% si comprado, ver §6)
- `(1 + job_legal_bonus)` de ítems de Mini Shop equipados.

### 4.3 Trabajos ilegales (5 tipos, cooldown aleatorio 10-20 min)
| Trabajo | Chance base de éxito |
|---|---|
| Robar | 55% |
| Hackear | 50% |
| Contrabando | 45% |
| Estafa | 40% |
| Narcotráfico | 35% |

- Ser de la **MAFIA** suma **+20 puntos porcentuales** a la chance de éxito.
- Ítems de Mini Shop pueden sumar `job_illegal_chance` extra.
- Éxito → botín `random.randint(75, 150)`, multiplicado por boost (+50%) e ítems (`job_illegal_bonus`).
- Fallo → penalización = `inversion` (parámetro opcional del comando) si es ≥ `ILLEGAL_MIN_INVEST = 5`, si no, penalización fija de 10. Ser mafia resta 10 a la penalización (piso 0). Ítems pueden reducir la penalización un % adicional (`job_illegal_penalty_reduction`).

### 4.4 Trabajo Booster (`/trabajo_booster`)
Exclusivo Server Boosters, cooldown de solo **6 minutos**, recompensa `random.randint(100, 1000)` — pensado como fuente de ingresos rápida y repetible para boosters.

### 4.5 Drops de ítems en trabajos
Cada trabajo exitoso (legal o ilegal) tira `roll_job_items`: recorre **todos** los ítems de la tienda de ítems (base + personalizados del server) que apliquen a ese tipo de trabajo, y por cada uno hace un roll independiente `random.uniform(0,100) < chance%`. Es decir, **un solo trabajo puede darte múltiples ítems a la vez** si tenés suerte en varios rolls simultáneos.

---

## 🎒 5. Tienda de ítems de trabajo (`ItemShopCog`)

Ítems base predefinidos (`SHOP_ITEMS`), agrupados por rareza:

| Rareza | Ítems | Chance | Precio venta |
|---|---|---|---|
| Común | Piedra común (80%), Piedra poco común (90%) | alta | 200 / 300 |
| Especial | Hierro (60%), Oro (50%) | media | 500 / 670 |
| Legendario | Diamante (20%), Esmeralda (10%) | baja | 800 / 3.000 |
| Mítico (solo ilegales) | Prisma (0.05%), Archivos Epstein (0.6%), Bigote gracioso (1%) | ultra baja | 3.560.000 / 1.429.000 / 831.000 |

- `/moon_vender_item` vende del inventario al precio actual (fijo, salvo que sea un ítem personalizado volátil).
- **Ítems personalizados** (`/moon_item_personalizado`, admin): un servidor puede crear ítems propios con rareza, chance, precio, rango de cantidad y si pertenece a trabajos legales/ilegales/ambos. Pueden marcarse `volatile=True` con `tick_minutes`: su precio fluctúa `±8%` (`ITEM_VOLATILITY_DEFAULT`) cada intervalo configurado, similar al mercado de MOON COINS pero por ítem.
- El inventario (`item_inventory.json`) es independiente del inventario de drogas y del de Mini Shop.

### 5.1 Mercado entre usuarios / P2P (`ItemTradeCog`)
Compra-venta directa de ítems de la tienda de trabajos (§5) entre usuarios, con MOON COINS de por medio y el bot como intermediario/custodio:

- `/vender_item_usuario <item> <cantidad> <precio>`: publica una cantidad de un ítem del inventario propio a un precio **total** (no unitario) en MOON COINS. El ítem se descuenta del inventario del vendedor de inmediato y **queda retenido por el bot** (no se puede gastar/perder mientras la publicación esté activa).
- `/mercado_items`: lista todas las publicaciones activas del servidor (ID, ítem, cantidad, precio, vendedor).
- `/comprar_item_usuario <id>`: le descuenta el precio en MOON COINS al comprador, se lo acredita al vendedor y le transfiere el ítem retenido. El vendedor recibe una notificación por MD.
- `/cancelar_venta_item <id>`: el vendedor puede cancelar su propia publicación, lo que le devuelve el ítem retenido a su inventario.

---

## 🚀 6. Mejoras permanentes (`BoostsCog`, `BOOST_DEFS`)

Compra única por usuario, para siempre. Todas dan **+50% (`bonus: 0.5`)** al área correspondiente:

| Clave | Precio | Efecto |
|---|---|---|
| `trabajo_legal` | 450.000 | +50% ganancia en trabajos legales |
| `trabajo_ilegal` | 500.000 | +50% botín en trabajos ilegales exitosos |
| `negocio_legal` | 700.000 | +50% rendimiento diario negocios legales |
| `negocio_ilegal` | 1.000.000 | +50% rendimiento diario negocios ilegales |

`boost_multiplier(owned, key)` devuelve `1.5` si está comprado, `1.0` si no — se multiplica directo sobre la ganancia base.

### 6.1 Crafteo de mejoras ELITE y Mejora Masiva (`CraftingCog`)
Las 4 mejoras normales tienen una versión **🌟 ELITE**, que **no se compra con MOON COINS**, se craftea combinando materiales:

- Requisitos por mejora ELITE: tener ya el boost normal de esa categoría + **1x chip exclusivo** (`chip_trabajo_legal`, `chip_trabajo_ilegal`, `chip_negocio_legal`, `chip_negocio_ilegal` — dropean de trabajos, ver §4.5) + **1x cualquier ítem mítico** (`MYTHIC_ITEM_IDS`: Prisma, Archivos Epstein o Bigote Gracioso — los drops más raros, solo de trabajos ilegales, ver §5).
- `/materiales_elite`: muestra, por cada mejora, si tenés el boost normal, si ya crafteaste la ELITE, y cuántos chips del material exclusivo tenés.
- `/craftear_elite <mejora>`: consume 1 chip + 1 ítem mítico (cualquiera de los 3) y otorga **+50% adicional** (`bonus: 0.5`) sobre el boost normal de esa categoría — es decir, el total se vuelve +100% combinando boost normal + ELITE.
- `/craftear_mejora_masiva`: una vez crafteadas las **4 ELITE**, combina todo en la **Mejora Masiva** (`MEGA_BOOST_BONUS = 0.30`): **+30% adicional** en trabajos legales, ilegales, negocios legales e ilegales, todo a la vez, encima de los bonos anteriores.

---

## 🛍️ 7. Mini Shop (`MiniShopCog`, `MINISHOP_ITEMS`)

Ítems de **500 a 10.000** MOON COINS, organizados en **4 categorías = 4 slots**. Regla clave: **solo se puede tener 1 ítem equipado por categoría** — comprar otro de la misma categoría **reemplaza automáticamente** al anterior (no se acumulan, no se pueden mezclar 2 de la misma categoría), pero sí se pueden combinar 4 ítems distintos (uno por categoría) para armar un build.

| Categoría | Ítems (de menor a mayor tier) | Efecto |
|---|---|---|
| 💱 Trading | Auto-Cambista (1.500) → Corredor Bursátil (6.000) → Algoritmo HFT (10.000) | -3% / -7% / -12% comisión en compra/venta de MOON COINS |
| 💼 Trabajo Legal | Café Energizante (500) → Currículum Pro (2.500) → Networking VIP (8.000) | +10% / +20% / +35% ganancia en trabajos legales |
| 🕶️ Trabajo Ilegal | Pasamontañas (500) → Contactos del Bajo Mundo (3.000) → Informante Policial (9.000) | +botín (10/15/25%), algunos también +% éxito y/o -% pérdida al fallar |
| 🎰 Apuestas | Amuleto de la Suerte (800) → Dados Cargados (4.000) → As Bajo la Manga (10.000) | +8% / +18% / +30% extra de ganancia al ganar en ruleta o tragamonedas |

`minishop_get_effect(owned, effect_key)` suma el valor de un `effect_key` dado de **todos** los ítems equipados — en la práctica solo hay 1 ítem por categoría, así que normalmente es un único valor salvo overlaps de efecto entre categorías distintas.

---

## 🤖 7.1 Autobots de inversión (`AutobotsCog`, `AUTOBOT_ITEMS`)

Distinto a la Mini Shop: no se "equipan" por categoría, sino que dan **usos** (cargas), y **solo se puede tener 1 Autobot a la vez** (no se puede comprar otro hasta agotar todos los usos del actual).

| Ítem | Precio | Usos |
|---|---|---|
| Autobot Básico | 1.000 | 3 |
| Autobot Intermedio | 5.000 | 8 |
| Autobot Avanzado | 15.000 | 20 |

- `/comprar_autobot`: suma usos al autobot del usuario (se acumulan entre compras).
- `/autobuy <precio_de_compra> <precio_de_venta>`: configura la orden. El bot queda esperando en fase `"esperando_compra"`: cuando el precio de MOON COIN (§2.4) cae a `precio_de_compra` o menos, compra automáticamente con la wallet USD del usuario (reserva las coins) y pasa a fase `"esperando_venta"`; cuando el precio sube a `precio_de_venta` o más, vende esas coins reservadas. Cada operación (compra o venta) consume **1 uso**.
- `autobuy_process_guild` se ejecuta en cada tick del mercado (`price_task`, cada 5 min) **y** cada vez que dispara una alerta de inversiones (§2.5), para que el autobot pueda reaccionar a saltos de precio fuera del tick normal.
- El descuento de comisión de trading de la Mini Shop (§7, categoría *trading*) también aplica a las operaciones del autobot.
- Notifica por MD (`_autobuy_notify`) cada operación ejecutada, y avisa cuando se agotan los usos o cuando la orden se reinicia.

---

## 💎 8. Server Boosters

Sistema transversal que da beneficios cruzados en casi todos los otros sistemas:

- **Detección**: listener `on_member_update` compara `premium_since` antes/después; el instante en que pasa de `None` a un valor dispara `_grant_booster_rewards`.
- **Recompensa de bienvenida** (una sola vez por usuario, `booster_claims.json` evita re-pagar en re-boosts): **500.000 MOON COINS** + **5.000 USD virtuales**.
- **Rol automático**: si el server configuró un rol vía `/configurar_rol_booster`, se asigna automáticamente al boostear.
- Beneficios exclusivos ya cubiertos en otras secciones: doble afiliación de facción, trabajo de 6 min, bonus x5 fijo en ruleta desde apuesta 400+, negocio exclusivo (Fábrica de Chocolates), y en el mensaje de alerta se listan todos con `/beneficios_booster`.

---

## 🏢 9. Negocios (`BusinessCog`, `BUSINESS_DEFS`)

Se compran una sola vez por tipo (no se puede tener 2 del mismo), generan rendimiento **pasivo** que se acumula con el tiempo y se cobra manualmente.

| Negocio | Categoría | Precio | Tasa diaria | Riesgo redada |
|---|---|---|---|---|
| Kiosco 24hs | Legal | 120.000 | 3.5% | — |
| Fábrica Textil | Legal | 400.000 | 5% | — |
| Casa de Taxis | Legal | 850.000 | 6% | — |
| Puesto de Drogas | Ilegal | 220.000 | 7% | 8% |
| Casino Ilegal | Ilegal | 650.000 | 7.5% | 10% |
| Burdel | Ilegal | 1.300.000 | 8% | 12% |
| Manager de VTubers | Legal (exclusivo) | 4.000.000 | 12.5% | — |
| Negocio de Armas | Ilegal (exclusivo) | 10.000.000 | 10% | 15% |
| Negocio de Drogas | Ilegal (exclusivo) | 15.000.000 | 20% | 18% |
| Fábrica de Chocolates | Legal (**solo Boosters**) | 2.000.000 | 10% | — |

- **Rendimiento diario** = `precio × tasa_diaria` (multiplicado por boost `negocio_legal`/`negocio_ilegal` si aplica, +50%).
- **Acumulación**: `biz_compute_pending` calcula el pendiente proporcional al tiempo transcurrido desde el último cobro, con un **tope de acumulación de 3 días** (`COLLECT_CAP_DAYS`) — si no cobrás en mucho tiempo, se pierde lo que exceda ese tope (no se acumula infinito).
- **Cobro** (`/cobrar_negocios`): cobra todos los negocios de una. Por cada negocio **ilegal**, hay un roll independiente contra su `risk`: si sale mal, el negocio se **pierde por completo** (redada) y esa parte del pendiente no se cobra; si sale bien, se acredita normalmente.
- **Negocio de Drogas**: además de generar rendimiento pasivo, es la llave (junto con ser MAFIA) para desbloquear el mercado negro (§11).

### 9.1 Auto Cobrador (`AutoCobradorCog`) — ítem nuevo
Ítem que cobra automáticamente las ganancias pendientes de **todos** los negocios del usuario, sin necesidad de ejecutar `/cobrar_negocios` a mano.

- **Precio fluctuante**: arranca en un valor random entre `AUTOCOBRADOR_PRICE_MIN = 25.000` y `AUTOCOBRADOR_PRICE_MAX = 100.000`. Un `tasks.loop` (cada `PRICE_UPDATE_MINUTES = 5`, igual ritmo que el mercado de MOON COIN) mueve el precio un paso aleatorio entero entre `AUTOCOBRADOR_PRICE_STEP_MIN = 2.000` y `AUTOCOBRADOR_PRICE_STEP_MAX = 12.000` en la dirección vigente (`"up"`/`"down"`).
- **Rebote en los topes**: a diferencia del mercado de MOON COIN (que arma episodios de varios ticks), acá el precio directamente **rebota**: al tocar el techo (100.000) o el piso (25.000), queda clavado exactamente ahí e invierte la dirección hacia el otro extremo en el siguiente tick.
- **Alertas**: cada cambio de precio se avisa en el mismo canal configurado con `/configurar_alertas_inversiones` (§2.5), con un mensaje especial cuando el precio toca un tope.
- `/autocobrador`: muestra el precio actual y, si ya lo tenés, cada cuántos minutos cobra.
- `/comprar_autocobrador [intervalo_minutos]`: lo compra al precio del momento y define el intervalo de cobro automático — **mínimo `AUTOCOBRADOR_MIN_INTERVAL_MINUTES = 60` minutos (1 hora)**, sin techo. Solo se puede tener 1 por usuario.
- `/configurar_autocobrador <intervalo_minutos>`: cambia el intervalo después de comprado (mismo mínimo de 60 min).
- **Cobro automático**: un `tasks.loop` corre cada `AUTOCOBRADOR_CHECK_SECONDS = 60s` y revisa, para cada usuario con el ítem, si ya pasó su intervalo configurado desde el último cobro (`last_collect`). Si corresponde, ejecuta el mismo cobro que `/cobrar_negocios` (función compartida `business_collect_all`, con el mismo riesgo de redada en negocios ilegales) y le manda un MD al usuario con el detalle de lo cobrado (o de las redadas sufridas).

---

## 🔫 10. Pandillas (`GangCog`)

- `/crear_pandilla <nombre>`: cuesta **500.000** (`GANG_CREATION_PRICE`). Crea 2 roles de Discord reales (rol de pandilla + rol de dueño) y asigna ambos al creador. Un usuario solo puede estar en **una pandilla a la vez**.
- `/invitar_pandilla`: solo el dueño puede invitar; el invitado no puede estar ya en otra pandilla.
- **Misiones grupales** (`/mision_pandilla`): 6 misiones con costo y ganancia base por miembro:

  | Misión | Costo/miembro | Ganancia base/miembro |
  |---|---|---|
  | Asaltar el banco | 800.000 | 1.000.000 |
  | Robo millonario | 1.000.000 | 2.000.000 |
  | Contrabando en masa | 1.000.000 | 3.000.000 |
  | Trabajos gubernamentales | 2.500.000 | 4.000.000 |
  | Robo a joyería | 1.000.000 | 5.000.000 |
  | Robo a empresa | 2.000.000 | 6.000.000 |

  - `GangMissionJoinView`: ventana de reclutamiento de **90 segundos** donde otros miembros de la pandilla pueden sumarse (mínimo 2, máximo 8 participantes), antes de que el iniciador confirme el lanzamiento.
  - Al confirmar, se cobra el costo por miembro a cada participante.
  - Resolución (`gang_resolve_mission`): **10% de probabilidad de fallo** (`GANG_MISSION_FAIL_CHANCE`). Si falla, se pierde adicionalmente entre 2% y 10% del botín potencial (repartido entre miembros), sin devolver lo invertido. Si tiene éxito, se paga el botín base (`reward_per_user × num_miembros`) más un **bono aleatorio del 20% o 30%**, repartido en partes iguales entre todos los participantes.

---

## 💊 11. Mercado negro / Drogas (`DrugsCog`)

**Acceso restringido**: requiere ser **MAFIA** *y* tener comprado el negocio "Negocio de Drogas" (`negocio_drogas`) — chequeado por `drugs_check_access`.

3 drogas, cada una con su propia banda de precio en USD virtuales y volatilidad:

| Droga | Precio inicial | Rango | Volatilidad por tick |
|---|---|---|---|
| 🌿 Marihuana | 12.000 | 8.000 – 40.000 | ±6% |
| ❄️ Cocaína | 55.000 | 30.000 – 95.000 | ±10% |
| 💉 Fentanilo | 110.000 | 80.000 – 150.000 | ±15% |

- Igual que el mercado de MOON COINS, un `tasks.loop(minutes=5)` mueve el precio de cada droga aleatoriamente dentro de su rango (piso/techo hard-clamp), guardando hasta 500 puntos de historial.
- Compra/venta se paga en **wallet USD virtual** (no en MOON COINS), a precio de mercado del momento, sin comisión.

---

## 📊 12. Niveles por actividad (`LevelsCog`)

- **Ganancia de XP**: cada mensaje válido (no bot, en un servidor) otorga entre **15 y 25 XP** (`LEVEL_XP_MIN`/`MAX`), con **cooldown anti-spam de 60 segundos** entre mensajes que otorgan XP (`LEVEL_MESSAGE_COOLDOWN`).
- **Curva de nivel**: `LEVEL_BASE_XP = 100` XP para el nivel 1, y cada nivel siguiente requiere `req_anterior × 1.25` (`LEVEL_GROWTH`), redondeado. Es decir, la curva es **geométrica**, no lineal — subir de nivel se vuelve progresivamente más caro.
  - `level_xp_required(n)`: XP acumulada total necesaria para llegar al nivel n.
  - `level_from_xp(xp)`: recorre la progresión para encontrar el nivel actual.
  - Barra de progreso visual: 20 bloques (🟩/⬛) proporcionales al avance dentro del nivel actual.
- **Recompensas por nivel**: preset por defecto en niveles 5, 10, 15, 20, 25, 30, 40, 50, 75, 100 (de 5.000 a 1.500.000 MOON COINS), totalmente configurable/reemplazable por servidor vía panel visual (`/configurar_recompensas_nivel`).
- Si un usuario sube **varios niveles de una sola vez** (por acumulación de XP entre chequeos), se suman las recompensas de **todos** los niveles cruzados, aunque el embed de alerta solo muestra el nivel final.
- `/topniveles`: ranking por XP total acumulada.

---

## 📨 13. Sistema de invitaciones (`InvitesCog`)

- Cachea el estado de usos de cada invite (`invites_cache_guild`) en memoria (`_invite_uses_cache`) para poder diferenciar, en `on_member_join`, cuál invite subió su contador de usos y así identificar al invitador (`invites_find_used`). Se actualiza en `on_invite_create`/`on_invite_delete` también.
- **Limitación conocida y documentada en el propio código**: si la invitación usada es de un solo uso, Discord la borra automáticamente al usarse y no aparece en la comparación → en ese caso el invitador queda como "no rastreable" y no se paga a nadie.
- **Anti-abuso de rejoin**: se guarda un registro histórico (`invites_seen_members`) de todo usuario que alguna vez se unió al server. Si alguien vuelve a entrar (kick/leave y reingreso), **no se paga ninguna recompensa de nuevo**, aunque se sigue registrando el evento.
- Recompensas: **5.000** MOON COINS para quien invita, **2.000** para quien fue invitado (montos fijos, `INVITE_REWARD_INVITER`/`INVITED`).
- **Hitos de invitador** (acumulativos, se pagan una sola vez cada uno): 10 invitaciones → 5.000, 50 → 100.000, 100 → 250.000.
- Mensaje de bienvenida totalmente personalizable por panel visual (`/configurar_mensaje_invitaciones`), con placeholders `{invitado}`, `{invitador}`, `{recompensa_invitador}`, `{recompensa_invitado}`, `{total_invites}`.

---

## 🏆 14. Logros de MOON COINS (`AchievementsCog`)

- Umbrales de balance total con recompensa fija, pagada **una sola vez** por umbral cruzado: 100K→20K, 200K→50K, 500K→100K, 1M→300K, 10M→1M, 100M→5M, 1B→4M.
- Se evalúan automáticamente en **cada** `economy_add_coins` con neto positivo (salvo pago de otros logros, para evitar loop), sin importar de qué comando venga la ganancia — trabajar, casino, negocios, ventas, cargas de admin, etc. todo dispara el chequeo.
- Si el nuevo balance cruza **varios umbrales a la vez**, se pagan y anuncian todos en la misma pasada.

---

## 🎭 15. Roles

### 15.1 Rol personalizado (`/newsrol`, staff)
Crea un rol de Discord real con nombre y color hex (`#RRGGBB` validado por regex) y lo asigna directamente a un usuario elegido.

### 15.2 Tienda de roles (`ShopCog`)
- `/configshop` (admin): pone en venta un rol existente a un precio fijo. Valida que el rol del bot esté **por encima** en la jerarquía, o avisa que no podrá asignarlo.
- `/buyrol`: compra con autocompletado de roles disponibles. Si falla la asignación por permisos, **reembolsa automáticamente** las MOON COINS gastadas.
- `/eliminarol` (admin): saca un rol de la tienda (no lo borra del server, solo de la lista de compra).

---

## 📋 16. Encuestas con apuestas — "Moon Encuestas" (`PollsCog`)

- Creación vía modal (`/moon_encuesta`): nombre, 2-5 opciones separadas por coma, apuesta mínima, duración (formato `30m`/`2h`/`1d`), texto extra opcional, y un panel de configuración posterior con botones para imagen y para **precargar el resultado ganador** (opcional).
- **Apuestas**: cada jugador apuesta MOON COINS a una opción vía modal (mínimo el `min_bet` configurado). Se descuenta al apostar.
- **Pago fijo, no pool proporcional**: los ganadores reciben exactamente `monto_apostado × 2`, sin importar cuánta gente apostó a cada lado ni el tamaño total del pozo (no hay redistribución entre opciones).
- Un `tasks.loop` (`check_polls`) revisa periódicamente las encuestas activas; al vencer el tiempo:
  - Si tenía un resultado precargado (`preset_winner`), se resuelve automáticamente y se pagan los ganadores.
  - Si no, pasa a estado `"ended"` (cerrada, esperando que staff cargue el resultado con `/moon_resultado`, que despliega un selector de encuesta + opción ganadora).

---

## 🎌 17. Integraciones externas de anime

### 17.1 AnimeFLV (`/getmyanime`)
No usa una librería Python: **ejecuta un subproceso de Node.js** (`animeflv_helper.js`, no incluido en este archivo, requiere `npm install animeflv-api` en la misma carpeta). El bot llama al proceso, le pasa la query por argv, espera stdout con timeout de 20s y parsea el JSON de respuesta. Si Node.js no está instalado en el sistema, el comando devuelve un error explicativo en vez de fallar silenciosamente.

### 17.2 AniDB (`/rankinganime`)
Integración HTTP directa (API XML de AniDB), sin Node:
- Requiere credenciales de cliente registradas en AniDB (`ANIDB_CLIENT`/`ANIDB_CLIENTVER` en `.env`); si faltan, el comando avisa que hay que configurarlas.
- Descarga y cachea localmente (`data/cache/anidb_titles.xml.gz`) el dump completo de títulos de AniDB, con **TTL de 24 horas** — evita re-descargar el archivo pesado en cada búsqueda.
- Búsqueda de título: primero coincidencia exacta (case-insensitive), si no, primera coincidencia parcial (substring) sobre el diccionario de títulos cacheado.
- **Rate limiting propio**: `ANIDB_MIN_REQUEST_INTERVAL = 2.2s` con un lock global (`_anidb_lock`) que fuerza una espera mínima entre requests sucesivos a la API — respeta la política de AniDB de no más de 1 request cada ~2 segundos.

---

## 📝 18. Logging del servidor

### 18.1 Logs de comandos y economía (`LoggerCog`)
Cualquier ejecución (exitosa o fallida) de un slash command dispara un log embed automático en el canal configurado con `/configurar_logs` (requiere `administrator=True` de Discord nativo, no del sistema de staff del bot). Los movimientos económicos relevantes también pueden loguearse con `economy_log`.

### 18.2 Logs de mensajes (`LogsCog`)
`/setlogs` (staff) activa/desactiva por servidor. Si está activo, cada mensaje de usuario (no bot) se **anexa a un archivo de texto plano** en `data/logs/<guild_id>.txt`, con timestamp UTC, canal, autor y contenido — es un log de auditoría en disco, no en Discord.

---

## 🙋 19. Perfiles, minijuegos sociales y comunidad

### Perfil de usuario (`ProfileCog`)
- `/setprofile`: abre un panel efímero con botones para completar el perfil personal del usuario **por servidor** (bio, datos configurables vía `ProfileConfigView`).
- `/myprofile` / `/seeprofile <usuario>`: muestran el embed del perfil propio o de otro miembro.

### Juego de conteo grupal (`CountGameCog`, `/start_count`)
- Juego colaborativo en un canal: hay que contar en orden desde el 1, **sin que el mismo usuario cuente dos veces seguidas**. Un número equivocado o un usuario repitiendo turno **rompe la cadena** y termina el juego.
- Límite superior `COUNT_LIMIT = 1000`; si se llega, el juego termina por "conteo perfecto".
- Se autocierra por inactividad tras `COUNT_INACTIVITY_TIMEOUT = 15 min` sin un número válido (`_timeout_watch`, se reinicia con cada número correcto).
- Recompensa: cada número correcto suma `COUNT_REWARD_PER_NUMBER = 0.1` MOON COINS a un pozo común, que al terminar el juego se reparte **en partes iguales solo entre quienes participaron** (no entre todos los miembros del canal).

### Comunidad (`CommunityCog`)
- `/opinasobre <usuario>`: le pone a un usuario un puntaje aleatorio 1-10 con comentario de sabor, sin persistencia (solo entretenimiento).
- `/rank_my_server`: analiza la estructura del server actual (canales, categorías, roles, boost, cantidad de humanos/bots) y calcula un puntaje de organización con detalle punto por punto.
- `/moon_invite`: devuelve el link de invitación público del bot (top.gg).

### Ranking de popularidad / votos (`VotesCog`)
- `/votar_user <usuario>`: voto único e irrepetible por par (votante, votado) — un mismo usuario no puede votar dos veces a la misma persona, ni votarse a sí mismo, ni votar a un bot (`votes_has_voted`).
- `/votes_rank`: top 5 usuarios con más votos del servidor, con medallas para el top 3.

### Bienvenidas (`WelcomeCog`)
- `on_member_join` dispara automáticamente el mensaje de bienvenida configurado.
- `/welcome_channel_set` (staff): define el canal de bienvenidas.
- `/welcome_message_set` (staff): panel visual (`WelcomeMessageConfigView`) para personalizar título, descripción e imagen del embed de bienvenida.

### Anuncios globales (`BroadcastCog`)
- `/moon_anuncio` (solo el owner del bot): arma un embed visual con un panel de configuración y lo manda a **todos los servidores** donde está instalado el bot — pensado para avisos oficiales (mantenimiento, nuevas versiones, etc.).

---

## 🆔 20. Utilidades varias

- `/informacion`: metadata del servidor (ID, dueño, fecha de creación, miembros humanos/bots, canales por tipo, nivel de boost, emojis, nivel de verificación).
- `/getmyinfo`: genera un `.txt` descargable (efímero) con datos personales del usuario en el server, incluyendo un contador de mensajes propio (`UserInfoCog` cuenta cada mensaje no-bot en `message_counts.json`, independiente del sistema de niveles).

---

## 🔧 21. Sincronización de comandos

Deliberadamente **no automática** al iniciar — el owner debe correr `/sync` o `/resync` manualmente:
- `/sync`: sincroniza el árbol de comandos actual (global, o al guild de `DEV_GUILD_ID` si está seteado, para propagación instantánea en desarrollo).
- `/resync`: además de sincronizar, **limpia primero** el scope (global y/o guild de dev) antes de repoblarlo, eliminando comandos "fantasma" que quedaron registrados en Discord pero ya no existen en el código.
- Ambos tienen timeout de 60s contra la API de Discord y manejan explícitamente `CommandSyncFailure`, `Forbidden` (falta scope `applications.commands`) y `HTTPException`.
- Manejador global `on_app_command_error`: intercepta `CheckFailure` (permisos), `TransformerError` (opciones de menú desactualizadas en el cliente de Discord) y cualquier excepción no controlada, respondiendo siempre de forma efímera y logueando en consola.

---

## 🧩 22. Resumen de todas las constantes económicas clave

| Constante | Valor |
|---|---|
| Precio inicial MOON COIN | 20.0 USD |
| Rango del mercado de MOON COIN | piso 1 / techo 100.000 |
| Paso de mercado por tick (5 min) | entero fijo entre 1 y 500 (no %), en episodios de 3-15 ticks seguidos en la misma dirección |
| Alertas aleatorias de inversión | cada 10-60 min (chequeo cada 30s), ejecutan un tick real |
| Comisión base de trading | 15% (reducible con ítems) |
| Interés de préstamo | +10% fijo |
| Corte automático de deuda por ingreso | 10%–40% aleatorio |
| XP por mensaje | 15–25, cooldown 60s |
| Crecimiento de curva de nivel | ×1.25 por nivel |
| Cooldown trabajo legal | 5 min |
| Cooldown trabajo ilegal | 10–20 min (aleatorio) |
| Cooldown trabajo Booster | 6 min |
| Tope de acumulación de negocios | 3 días |
| Precio del Auto Cobrador | fluctúa 25.000–100.000, rebota en los topes, cambia cada 5 min |
| Intervalo mínimo del Auto Cobrador | 60 min (1 hora) |
| Bonus ELITE por mejora | +50% adicional sobre el boost normal |
| Bonus Mejora Masiva (4 ELITE) | +30% adicional en las 4 categorías |
| Fallo de misión de pandilla | 10% de probabilidad |
| Bono de misión de pandilla exitosa | +20% o +30% aleatorio |
| Pago de encuestas | x2 fijo sobre lo apostado |
| Recompensa por número en juego de conteo | 0.1 MOON COINS, repartido solo entre participantes |
