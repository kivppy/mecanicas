[README.md](https://github.com/user-attachments/files/30155145/README.md)
# MOON BOT — README técnico

Bot de Discord de economía/casino/rol para servidores, escrito en un solo
archivo (`luna_bot.py`, ~10.800 líneas) con `discord.py` (app commands +
algunos comandos de texto plano). Este documento explica el código
**ordenado por mecánica de juego**, no por orden de aparición en el archivo,
para que sea fácil ubicar dónde vive cada sistema.

> Convención del propio código: todo el archivo está dividido con bloques
> `# ===...===` seguidos de un título (`# UTILS: ...`, `# COG: ...`,
> `# SISTEMA: ...`). Los nombres de sección de abajo son los mismos que vas
> a encontrar con Ctrl+F en `luna_bot.py`.

---

## 0. Arquitectura general

- **Un solo Bot** (`LunaBot`, subclase de `commands.Bot`) que carga todos los
  `Cog` listados en la tupla `COGS` dentro de `setup_hook()`. Si un Cog
  falla al cargar, el bot sigue arrancando sin él y lo loggea.
- **Persistencia**: `storage_load(name)` / `storage_save(name, data)`
  (sección `# UTILS: storage`). Cada "tabla" es un archivo/colección lógica
  identificada por nombre (`"economy"`, `"businesses"`, `"tasks_progress"`,
  `"active_effects"`, etc.), indexada como `data[guild_id][user_id] = {...}`.
  No hay base de datos relacional: todo es JSON anidado por servidor y
  usuario.
- **Slash commands vs. comandos de texto**: la inmensa mayoría son slash
  commands (`@app_commands.command`). Un puñado de comandos administrativos
  peligrosos (reset de economía, limpieza de negocios) son comandos de
  **texto plano** con prefijo `moon` (ver sección 15) para no gastar cupo
  del límite de 100 comandos globales de Discord.
- **Límite de 100 comandos globales**: Discord no permite más de 100 slash
  commands top-level por aplicación. Varios sistemas usan
  `app_commands.Group` (ej. `/vip`, `/tareas`, `/sustancias`) para meter
  varios subcomandos bajo 1 solo slot del límite.
- **Ajedrez** (`ChessCog`) se importa desde un módulo externo
  (`moon_chess`, no incluido en este archivo) para evitar imports
  circulares; `luna_bot.py` le inyecta funciones (`storage_load`,
  `economy_add_coins`, etc.) como atributos de `bot` al final del archivo.

---

## 1. Economía base (MOON COINS)

**Sección:** `# UTILS: economy` · **Cog:** `EconomyCog`

- Moneda única del bot: **MOON COINS**. `economy_add_coins(gid, uid, amount, reason, ...)`
  es la función central por la que pasa CUALQUIER ingreso/egreso de saldo
  (trabajos, negocios, casino, tareas, ventas, etc.) — todo lo demás del
  bot llama a esta función, nunca edita el balance a mano.
- Comandos: `/economia` (ver tu balance), `/topmoon` (ranking de riqueza),
  `/transferir` (mandarle plata a otro usuario), `/chargemoon` (admin/dueño,
  carga manual).
- **Préstamos**: `/prestamo`, `/deuda`, `/pagardeuda` — sistema de deuda con
  interés, separado del balance normal.
- **Facciones** (Mafia / Policía): `/afiliarse_mafia`, `/afiliarse_policia`,
  `/afiliarse_ambas` (exclusivo Boosters), `/desafiliarse`, `/traicionar`,
  `/faccion`. Pertenecer a una facción da bonus/penalizaciones en otros
  sistemas (ej. mafia sube el % de éxito en trabajos ilegales y habilita el
  mercado negro de drogas — sección 8).
- Cambiar de facción tiene cooldown y penalización por abuso (sección
  `# UTILS: control de cambios de facción`).

## 2. Mercado MOON COIN ↔ USD virtual

**Secciones:** `# CONFIG` (precio inicial/techo/piso), `# UTILS: market`,
`# UTILS: alertas aleatorias de inversiones` · **Cogs:** `MarketCog`,
`InvestmentAlertsCog`

- El precio del MOON COIN fluctúa solo con un modelo de "tendencias"
  (arranca una racha de subida o bajada y se mantiene un rato, no es
  puramente aleatorio tick a tick) entre un piso y un techo (`PRICE_FLOOR`
  / `PRICE_CEILING`).
- `/vendermoon`, `/comprarmoon`: cambiás MOON COINS por USD virtuales (tu
  "wallet USD") y viceversa, al precio del momento.
- `/moonchart`: gráfico de la evolución reciente del precio.
- `/guia_inversion`: guía en texto de cómo jugar el mercado.
- `InvestmentAlertsCog` manda alertas random a un canal configurable cuando
  el precio pega una subida/bajada fuerte.

## 3. Trabajos (`/trabajar`)

**Secciones:** `# COG: Jobs / Trabajos y facciones`, `# SISTEMA: Trabajos
rurales`, `# SISTEMA: Trabajos especiales`

Todo vive bajo el mismo comando `/trabajar trabajo:<opción>`, que ramifica
según a qué diccionario pertenece la clave elegida (`JOB_CHOICES` es la
unión de las 4 categorías):

| Categoría | Diccionario | Cooldown | Notas |
|---|---|---|---|
| 🟢 Legal | `LEGAL_JOBS` | `LEGAL_COOLDOWN` (5 min, reducible con fármacos) | Siempre exitoso, dropea ítems `"legal"` |
| 🔴 Ilegal | `ILLEGAL_JOBS` | random entre `ILLEGAL_COOLDOWN_MIN/MAX` (10-20 min, reducible con droga_comun) | Puede fallar (`chance` de éxito), penalización si falla |
| 🌾 Rural | `RURAL_JOBS` | `RURAL_JOB_COOLDOWN` (5 min) | Requiere una herramienta (`tool_id`) crafteada o comprada; se consume 1 por uso |
| 💊🌿 Especiales | `SPECIAL_JOBS` (`farmaceutico`, `cosechador_droga`) | 10 min / 25 min | Pagan más, dropean el DOBLE de ítems normales (ver `roll_job_items(..., extra_rolls=1)`) y tienen su propia chance de dropear un fármaco/droga |

- `/trabajo_booster`: trabajo aparte, exclusivo Server Boosters, cooldown
  corto (6 min) fijo — no pasa por `/trabajar`.
- Bonos que se aplican en cascada sobre la recompensa: mejoras permanentes
  (`boost_multiplier`), ítems de la Mini Shop (`minishop_get_effect`),
  beneficios VIP (`vip_get_benefit_value`) y efectos de fármacos/drogas
  activos (`effects_get_value`, sección 6).
- El Cosechador de droga (ilegal) puede fallar como cualquier trabajo
  ilegal: `inversion` (parámetro del comando) reduce la pérdida si falla.

## 4. Ítems de trabajo y tienda (`SHOP_ITEMS`)

**Secciones:** `# SISTEMA: Ítems de trabajo` · **Cog:** `ItemShopCog`

- Catálogo único `SHOP_ITEMS`: cada ítem tiene rareza (`comun` / `especial`
  / `legendario` / `mitico`), `chance` de drop, `price`, `min_qty`/`max_qty`,
  y una lista `jobs` que dice en qué trabajos puede dropear (`"legal"`,
  `"illegal"`, ambas, o `[]` si NO dropea nunca de un trabajo — ej.
  herramientas rurales que solo se compran/craftean).
- `roll_job_items(gid, uid, job_type, extra_rolls=0)`: es la función que
  tira los dados de drop cada vez que un trabajo sale bien. La llaman
  `/trabajar`, `/trabajo_booster` y `/trabajo_especial` (dentro de
  `/trabajar`). Acepta `extra_rolls` para dropear ítems por partida doble
  (trabajos especiales) y lee automáticamente los efectos activos del
  usuario para aplicar boosts de rareza (`droga_legendario`) o tiradas
  extra (`farmaco_mitico`).
- Comandos: `/moon_tienda_items` (ver catálogo por rareza, con botones),
  `/moon_inventario`, `/moon_vender_item`, `/moon_comprar_item`,
  `/moon_item_personalizado` (admin: crear ítems custom por servidor).
- Precios pueden ser **volátiles** (fluctúan solo, como el MOON COIN) según
  el flag `volatile` del ítem — lo procesa `ItemShopCog.item_price_task`
  (loop cada 1 min).
- **Crafteo** (`# UTILS: Crafteo de mejoras ELITE y Mejora Masiva`,
  `# UTILS: Ítems de trabajo` → `format_recipe`): algunos ítems (ej.
  herramientas rurales) se craftean combinando otros ítems del inventario
  en vez de comprarse.
- **Mercado entre usuarios (P2P)** (`# UTILS`/`# COG: Mercado entre
  usuarios`): `/vender_item_usuario`, `/mercado_items`,
  `/comprar_item_usuario`, `/cancelar_venta_item` — un usuario publica un
  ítem de su inventario a un precio fijo y otro se lo compra directo.

## 5. Fármacos y Drogas (consumibles con efecto temporal)

**Sección:** `# SISTEMA: Fármacos y Drogas` · **Cog:** `SustanciasCog`
(comando `/sustancias`, grupo)

8 ítems nuevos dentro de `SHOP_ITEMS` (4 fármacos + 4 drogas, uno por
rareza). Se consiguen de **4 formas**: dropeando en cualquier trabajo
legal/ilegal (`jobs: ["legal","illegal"]`, chance 25%/20%/15%/10% según
rareza — común → mítico), en los trabajos especiales (chance extra propia),
completando tareas (sección 7), o comprándolos directo en la tienda.

- `CONSUMABLE_EFFECTS`: diccionario que define, por ítem, cuánto dura el
  efecto (según rareza: 30min / 2h / 6h / 24h) y qué hace:

  | Ítem | Efecto | Aplica en |
  |---|---|---|
  | `farmaco_comun` | +15% al vender ítems | `/moon_vender_item` |
  | `farmaco_especial` | +25% ganancia en trabajos legales | `/trabajar` (legal y especial legal) |
  | `farmaco_legendario` | -30% cooldown en trabajos legales | `/trabajar` (legal) |
  | `farmaco_mitico` | -30% cooldown legal **+** doble de ítems en cualquier trabajo | `roll_job_items` (global) |
  | `droga_comun` | -30% cooldown en trabajos ilegales | `/trabajar` (ilegal) |
  | `droga_especial` | +15% chance de ganar apuestas (exento del límite de 2) | ruleta, carrera, pelea callejera |
  | `droga_legendario` | +50% chance de ítems legendarios/míticos | `roll_job_items` (global) |
  | `droga_mitico` | +20% de plata ganada en apuestas (exento del límite de 2) | ruleta, tragamonedas, carrera, pelea callejera |

- **Activación**: `/sustancias usar` (consume 1 unidad del inventario,
  autocompletado solo muestra lo que tenés). `/sustancias efectos` muestra
  tus efectos activos y cuánto les queda.
- **Límite**: máximo 2 efectos activos simultáneos por usuario, salvo
  `droga_especial` y `droga_mitico` (marcados `limit_exempt: True`), que no
  cuentan para ese límite — están pensados para acumularse con los otros 2.
- Toda la lógica de activar/consultar efectos vive en 3 funciones
  reutilizables: `effects_get_active`, `effects_get_value`,
  `effects_activate` (más los helpers de apuestas
  `effects_roll_bet_win` / `effects_get_bet_payout_boost`).
- **Nota de diseño**: en tragamonedas solo aplica el bonus de plata
  (`droga_mitico`), no el de "ganar más" (`droga_especial`), porque ahí el
  resultado depende de los símbolos que salen, no de un booleano ganar/
  perder — no había forma sana de "forzar" un giro ganador sin regalar
  plata directamente.

## 6. Trabajos especiales (Farmacéutico / Cosechador de droga)

Ver sección 3 — viven adentro de `/trabajar`, no son comando aparte (se
fusionaron para no gastar un slot extra del límite de 100 comandos). Su
diferencial es que además de pagar más y dropear el doble de ítems
normales, tienen su propia chance (`consumable_chance`, ~6%) de regalar un
fármaco o droga de rareza aleatoria (`roll_consumable_drop`, pesada hacia
rarezas comunes: 60% común / 30% especial / 9% legendario / 1% mítico).

## 7. Tareas diarias y de fin de semana

**Sección:** `# SISTEMA: Tareas diarias y de fin de semana` · **Cog:**
`TasksCog` (comando `/tareas`, grupo)

- `DAILY_TASKS` (4 tareas, se reinician a las 00:00 UTC) y `WEEKEND_TASKS`
  (6 tareas, solo completables sábado/domingo, se reinician cada lunes por
  ISO-week). Cada tarea tiene dificultad (`facil`/`media`/`dificil`), rango
  de recompensa en MOON COINS y una `consumable_chance` de regalar un
  fármaco o droga al completarla.
- `tasks_get_state(gid, uid)`: lee el progreso del usuario y hace el reset
  automático comparando la fecha/semana guardada contra la actual.
- `/tareas ver`: progreso actual (✅/⬜ por tarea). `/tareas completar
  tarea:<opción>`: marca la tarea como hecha, paga la recompensa y tira el
  roll de consumible.

## 8. Mejoras permanentes, ELITE y Mejora Masiva

**Secciones:** `# UTILS: mejoras`, `# UTILS: mejoras ELITE` · **Cogs:**
`BoostsCog`, `CraftingCog`

- `BOOST_DEFS`: mejoras normales, se compran 1 vez por usuario con MOON
  COINS y quedan para siempre (`boost_multiplier(owned, key)` da el
  multiplicador acumulado). Cubren trabajos legales/ilegales y negocios.
- `ELITE_BOOST_DEFS`: versión mejorada de cada boost, NO se compra con
  plata — se craftea combinando un chip exclusivo + 1 ítem mítico
  (`/materiales_elite`, `/craftear_elite`).
- **Mejora Masiva**: combina las 4 mejoras ELITE en una sola mejora
  superior (`/craftear_mejora_masiva`).
- `/mejoras`, `/comprar_mejora`: catálogo y compra de las mejoras normales.

## 9. Mini Shop (500 – 10.000 MC)

**Sección:** `# UTILS: Mini Shop` · **Cog:** `MiniShopCog`

Ítems baratos organizados por **categoría/slot** (solo podés tener 1
equipado por categoría a la vez, a diferencia de las mejoras que se
acumulan todas). Dan efectos como bonus de ganancia en trabajos, reducción
de penalización en ilegales, bonus de apuestas, etc.
(`minishop_get_effect(owned, effect_key)` es como se leen en el resto del
código). Comandos: `/minishop`, `/comprar_item` (ítem de Mini Shop, no
confundir con `/moon_comprar_item` de la tienda de trabajos), `/mis_items`.

## 10. Autobots de inversión

**Sección:** `# UTILS: Autobots de inversión` · **Cog:** `AutobotsCog`

Ítems que automatizan la compra/venta de MOON COINS según reglas que
configurás (`/autobuy`). Límite: 1 Autobot activo por usuario; comprar uno
nuevo suma usos, no lo reemplaza. Comandos: `/autobots`, `/comprar_autobot`,
`/autobuy`.

## 11. Negocios (`/negocios`)

**Sección:** `# COG: Negocios` · **Cog:** `BusinessCog` +
**Auto Cobrador**: `# UTILS`/`# COG: Auto Cobrador` · **Cog:**
`AutoCobradorCog`

- `BUSINESS_DEFS`: catálogo de negocios legales e ilegales, cada uno con
  precio, `daily_rate` (% diario de rendimiento) y, si es ilegal, `risk`
  (probabilidad de redada al cobrar → perdés el negocio).
- `business_collect_all(guild_id, uid)`: función central que cobra TODOS
  los negocios de un usuario de una. Aplica, en orden: multiplicador de
  mejoras (`boost_multiplier`), boosts pasivos de negocio (ver abajo),
  chance de redada (reducida por `vista_ciega`), y finalmente el
  **Impuesto de logística** (`BUSINESS_WITHDRAWAL_TAX`, reducido por
  `papeles_legales`).
- **Negocios con drop de ítems pasivos** (`SPECIAL_BIZ_DROPS`): al cobrar
  **Fiscalía** (legal) hay una chance mínima (3%) de conseguir `papeles_legales`
  o `empleados_contentos`; al cobrar **Casa de Gobierno** (ilegal) la misma
  chance de conseguir `vista_ciega` o `cronometro`. Estos 4 ítems son
  `no_trade` (no se compran/venden, solo se consiguen así) y su efecto es
  **pasivo**: con solo tenerlos en el inventario ya aplican, no se gastan
  ni se "usan" con `/sustancias usar`.

  | Ítem | Efecto |
  |---|---|
  | `papeles_legales` | -50% del Impuesto de logística al cobrar |
  | `empleados_contentos` | +100% al % diario de negocios **legales** |
  | `vista_ciega` | -50% de probabilidad de redada en negocios ilegales |
  | `cronometro` | +100% al % diario de negocios **ilegales** |

- Comandos: `/negocios`, `/comprar_negocio`, `/mis_negocios`,
  `/cobrar_negocios`.
- **Auto Cobrador**: ítem de precio fluctuante que cobra los negocios del
  usuario en automático cada tanto (configurable, mínimo 1h) y le manda un
  DM con el resumen. `/autocobrador`, `/comprar_autocobrador`,
  `/configurar_autocobrador`.

## 12. Mercado negro de drogas (Mafia)

**Sección:** `# UTILS: mercado negro (drogas)` · **Cog:** `DrugsCog`

Sistema aparte de "drogas" (commodities de precio fluctuante en USD, NO
confundir con los consumibles `droga_*` de la sección 5) que solo pueden
operar usuarios de la **Mafia** con el negocio ilegal `negocio_drogas`
comprado. `/mercado_drogas`, `/comprar_droga`, `/vender_droga`.

## 13. Casino

**Cogs:** `BlackjackCog`, `RouletteCog`, `SlotsCog`, `GamblingRankCog` ·
Texto plano: `CasinoTextCommandsCog` (`# CARRERA DE CABALLOS`,
`# PELEA CALLEJERA`)

| Juego | Comando | Notas |
|---|---|---|
| Blackjack | `/blackjack` | Contra el bot |
| Ruleta | `/ruleta` | Números/colores/docenas, etc. |
| Tragamonedas | `/tragamonedas` | Símbolos con peso; solo el bonus de plata de `droga_mitico` aplica acá |
| Carrera de caballos | texto: `moon carrera_caballos` | Menú con Select para elegir caballo |
| Pelea callejera | texto: `moon pelea_callejera` | 1v1, paga ~1.9x |

- `bet_force_loss(gano)`: función de anti-abuso/margen de la casa que
  aplica a TODOS los juegos antes de calcular el pago.
- `effects_roll_bet_win` / `effects_get_bet_payout_boost` (sección 5) se
  enganchan justo después de `bet_force_loss` en ruleta, carrera y pelea
  callejera.
- **Pozo Acumulado / Jackpot** (`# UTILS: Pozo Acumulado`): 1.5% de cada
  apuesta en cualquier juego se acumula ahí. `/jackpot` para consultarlo.
- **Ranking de ludopatía** (`# COG: Ranking de ludopatía`):
  `/ranking_ludopatia` — quién ganó y quién perdió más en el casino.

## 14. Sistema VIP

**Sección:** `# SISTEMA VIP` · **Cog:** `VipCog` (comando `/vip`, grupo)

Tiers pagos (`VIP_TIERS`) con beneficios que se leen desde otros sistemas
vía `vip_get_benefit_value(gid, uid, benefit_key)` (bonus de trabajos,
bonus de éxito en ilegales, etc.). `/vip beneficios` muestra los tiers;
`/vip agregar` es de administración (otorgar/actualizar VIP a un usuario).

## 15. Niveles por mensaje

**Sección:** `# UTILS: Sistema de niveles` · **Cog:** `LevelsCog`

XP por mandar mensajes, con recompensas en MOON COINS configurables por
nivel. `/minivel`, `/topniveles`, y comandos de admin para configurar canal
de alertas, el embed de subida de nivel y las recompensas por nivel.

## 16. Perfiles, info y roles

- **Perfiles** (`# COG: Perfiles de usuario`): `/setprofile` (modals por
  partes), `/myprofile`, `/seeprofile`. Guardados por servidor, con campos
  extra exclusivos para Boosters.
- **Info** (`# COG: Info del servidor`, `# COG: User info`):
  `/informacion`, `/getmyinfo`.
- **Roles**: `/newsrol` (crear rol personalizado), y **Shop de roles**
  (`# COG: Shop de roles`): `/configshop`, `/rolshop`, `/buyrol`,
  `/eliminarol` — comprar roles con MOON COINS.

## 17. Invitaciones, Boosters, Logros

- **Invitaciones** (`# UTILS`/`# COG: Sistema de invitaciones`): trackea
  quién invitó a quién, con ranking y alertas configurables.
  `/invitaciones`, `/ranking_invitaciones`,
  `/configurar_mensaje_invitaciones`.
- **Recompensas Server Boosters** (`# COG: Recompensas para Server
  Boosters`): rol automático + beneficios exclusivos listados en
  `/beneficios_booster` (afiliación doble a facciones, trabajo booster,
  fábrica de chocolates en negocios, etc.).
- **Logros** (`# COG: Logros de MOON COINS`): umbrales de plata acumulada
  con recompensa y alerta a un canal. `/logros`.

## 18. Encuestas, comunidad y misceláneos

- **Moon Encuestas** (`# COG: Polls`): encuestas con apuestas de MOON
  COINS sobre el resultado. `/moon_encuesta`, `/moon_resultado`,
  `/encuestas_config`.
- **Comunidad** (`# COG: Comunidad`): `/opinasobre` (el bot opina de un
  usuario del 1 al 10), `/moon_invite`, `/rank_my_server`.
- **Votos de popularidad** (`# UTILS`/`# COG: Votos`): ranking de usuarios
  más "queridos" del servidor.
- **Bienvenidas** (`# COG: Bienvenidas`): mensaje de bienvenida
  configurable por menú visual.
- **Juego de conteo grupal** (`# COG: Juego de conteo`): `/start_count`.
- **Ajedrez**: módulo externo (`moon_chess`), ver sección 0.

## 19. Moderación, logs y administración

- **Logs** (`# COG: Logs de mensajes`): registro de mensajes
  editados/borrados en un canal. `/configurar_logs`, `/setlogs`.
- **Ayuda** (`# COG: Help`, `# COG: Moon Help`): `/moon_help` (lista por
  secciones con botones), `/moon_guias` (guía detallada por comando).
- **Broadcast** (`# COG: Anuncio global`): anuncio a todos los servidores,
  exclusivo del dueño real del bot (`BROADCAST_OWNER_ID`).
- **Comandos de texto plano** (`# COG: Comandos de texto`, prefijo
  `moon`): `moon resetear_economia`, `moon limpiar_negocios` (ambos
  exclusivos del dueño), además de `moon carrera_caballos` y
  `moon pelea_callejera` (casino, sección 13).

---

## Cómo se conectan los sistemas nuevos (fármacos/drogas/tareas/negocios especiales)

Para que quede claro el "cableado" entre mecánicas, así es el flujo típico:

```
/trabajar (legal/ilegal/especial)
    ├─ paga MOON COINS (boosteado por mejoras + Mini Shop + VIP + fármaco activo)
    ├─ roll_job_items(...) → puede dropear cualquier SHOP_ITEMS, incluidos
    │       farmaco_*/droga_* (25%/20%/15%/10% según rareza)
    └─ si es trabajo especial → roll extra de consumible propio

/tareas completar → paga MOON COINS + chance de farmaco_*/droga_*

/cobrar_negocios (fiscalia/casa_gobierno) → chance de ítem pasivo de negocio
    (papeles_legales / empleados_contentos / vista_ciega / cronometro)

/sustancias usar → activa el efecto de un farmaco_*/droga_* del inventario
    → el efecto queda guardado en storage "active_effects" con su
      timestamp de vencimiento, y lo leen: /trabajar, /moon_vender_item,
      ruleta, tragamonedas, carrera de caballos, pelea callejera y
      roll_job_items
```

## Storages usados (para debug/inspección)

| Storage | Qué guarda |
|---|---|
| `economy` | Balance de MOON COINS y wallet USD por usuario |
| `job_cooldowns` | Cooldowns de trabajar (legal/ilegal/rural/booster/especiales) |
| `iteminv` | Inventario de `SHOP_ITEMS` por usuario |
| `active_effects` | Fármacos/drogas activos y su timestamp de vencimiento |
| `tasks_progress` | Progreso de tareas diarias/finde por usuario |
| `businesses` | Negocios comprados por usuario y su `last_collect` |
| `boosts` | Mejoras permanentes/ELITE compradas por usuario |
| `minishop` | Ítems equipados de la Mini Shop por categoría |
| `vip` | Tier VIP y vencimiento por usuario |
| `profiles` | Perfiles de usuario por servidor |

---

*Generado a partir de `luna_bot.py`. Si agregás una mecánica nueva, sumala
acá en la sección que corresponda (o creá una nueva) para que este
documento no quede desactualizado.*
