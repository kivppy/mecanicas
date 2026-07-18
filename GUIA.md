# 🌙 Guía del jugador — MOON BOT

Todo lo que podés hacer en el bot, explicado como si te lo contara un
amigo que ya lo tiene re jugado. Ordenado por tema, no por comando suelto.

---

## 💰 1. Lo básico: tus MOON COINS

Todo en el bot gira alrededor de una sola moneda: la **MOON COIN**.

- `/economia` — ver cuánta plata tenés.
- `/topmoon` — el ranking de los más ricos del server.
- `/transferir` — mandarle plata a otro usuario.
- `/prestamo`, `/deuda`, `/pagardeuda` — si te quedaste corto, podés pedir
  un préstamo... pero se paga con interés, ojo.

**¿Cómo se consigue plata?** Básicamente de 4 formas: trabajando,
apostando en el casino, teniendo negocios, o jugando con el mercado. Las
vemos todas abajo.

---

## 📈 2. El mercado (comprar/vender MOON COINS con USD)

El precio de la MOON COIN sube y baja solo, como una cripto. La idea es
comprar barato y vender caro.

- `/moonchart` — ver cómo viene moviéndose el precio últimamente.
- `/comprarmoon` — comprar MOON COINS con tu USD virtual.
- `/vendermoon` — vender tus MOON COINS al precio actual.
- `/guia_inversion` — si sos nuevo en esto, te explica cómo jugarla bien.
- De vez en cuando el bot tira alertas solo cuando el precio pega una
  subida o bajada fuerte, para que no te la pierdas.

💡 Si no querés estar pendiente todo el día, hay un **Autobot** que compra
y vende por vos automáticamente (lo vemos más abajo, en la sección 9).

---

## 👔 3. Trabajar

El comando principal es `/trabajar`. Ahí elegís QUÉ trabajo hacer, y hay
varias familias:

- 🟢 **Trabajos legales** — siempre te pagan, sin riesgo. Menos plata pero
  seguro.
- 🔴 **Trabajos ilegales** — pagan más, pero pueden salir mal (perdés
  plata si te agarran). Cuanto más "invertís" al hacerlos, menos perdés
  si fallan.
- 🌾 **Trabajos rurales** — necesitás tener (o craftear) una herramienta
  específica para cada uno. Sin la herramienta no podés trabajar de eso.
- 💊🌿 **Trabajos especiales — Farmacéutico y Cosechador de droga** —
  pagan bastante más que los normales, tardan más en recargarse (10 y 25
  minutos), pero a cambio te dan el DOBLE de ítems cuando te va bien, y
  encima tienen su propia chance de regalarte un fármaco o una droga
  (ver sección 6). El Cosechador es "ilegal" también: puede fallar.

Cada trabajo tiene su propio tiempo de espera (cooldown) antes de poder
volver a hacerlo — el bot te avisa cuánto te falta si lo intentás antes de
tiempo.

💎 Si sos **Server Booster**, tenés `/trabajo_booster` aparte: un trabajo
extra con recarga cortita (6 minutos) exclusivo para vos.

---

## 🎒 4. Ítems (los que dropean trabajando)

Cuando te va bien en un trabajo, además de plata podés recibir **ítems**
al azar. Hay 4 rarezas: común, especial, legendario y mítico (de más
fácil a más difícil de conseguir).

- `/moon_tienda_items` — mirá el catálogo completo: qué ítems existen, qué
  probabilidad tienen de salir, y cuánto valen si los querés comprar
  directo en vez de esperar a que te toquen.
- `/moon_inventario` — ver qué tenés guardado.
- `/moon_vender_item` — vender ítems que no uses por MOON COINS.
- `/moon_comprar_item` — comprarlos directo, sin depender de la suerte.

También existe un **mercado entre jugadores**: si querés vender algo tuyo
a un precio que vos elegís (en vez del precio fijo de la tienda), usás
`/vender_item_usuario`, y otros lo ven con `/mercado_items` y te lo
compran con `/comprar_item_usuario`.

---

## 💊 5. Fármacos y 🧬 Drogas (los boost temporales)

Estos son ítems especiales que, a diferencia del resto, no los vendés:
los **usás** para activar un efecto que dura un tiempo.

**¿Cómo los consigo?**
- Dropeando en cualquier trabajo (legal o ilegal), con más o menos chance
  según la rareza.
- Completando tareas (sección 7).
- En los trabajos especiales, que tienen su chance extra.
- O comprándolos directo en la tienda si no querés esperar.

**¿Cómo los uso?**
- `/sustancias usar` — elegís cuál de tu inventario querés activar.
- `/sustancias efectos` — ver qué tenés activo ahora mismo y cuánto te
  queda de tiempo.

**¿Qué hace cada uno?**

| | Común | Especial | Legendario | Mítico |
|---|---|---|---|---|
| 💊 **Fármaco** | +% al vender ítems | +% de plata en trabajos legales | -% tiempo de espera en trabajos legales | Lo de legendario + el doble de ítems en CUALQUIER trabajo |
| 🧬 **Droga** | -% tiempo de espera en trabajos ilegales | +% de chances de ganar apuestas | +% de chances de ítems raros | +% de plata ganada en apuestas |

Cuanto más rara la sustancia, más dura el efecto activo (desde media hora
las comunes hasta 24 horas las míticas).

⚠️ **Ojo con el límite**: solo podés tener **2 sustancias activas al mismo
tiempo**. La única excepción son los 2 boosts de apuestas (droga
especial y droga mítica), que van aparte y no cuentan para ese límite —
podés tenerlos siempre sumados a otros 2.

---

## 📋 6. Tareas diarias y de fin de semana

Además de trabajar, tenés misiones que te dan plata extra (y chance de
sustancias) por cumplirlas:

- `/tareas ver` — mirá cuáles ya cumpliste y cuáles te faltan.
- `/tareas completar` — elegís una tarea pendiente y la completás.

Hay **4 tareas diarias** (se reinician todos los días a la medianoche) y
**6 tareas de fin de semana** (solo se pueden hacer sábado o domingo, y se
reinician cada lunes). Cuanto más difícil la tarea, más paga y más chance
tenés de sacarte un fármaco o droga de yapa.

---

## ⚡ 7. Mejoras permanentes

Estas son distintas a los fármacos/drogas: **una vez que las comprás, son
para siempre**, no se vencen.

- `/mejoras` — ver el catálogo y cuáles ya tenés.
- `/comprar_mejora` — comprar una nueva.

Las mejoras suben cosas como cuánto ganás trabajando o en negocios. Y para
los que quieren ir más al fondo: cada mejora tiene una versión **ELITE**
mucho más fuerte, que no se compra con plata sino que se **craftea**
combinando materiales específicos (`/materiales_elite` te dice qué
necesitás, `/craftear_elite` la crea). Si conseguís las 4 mejoras ELITE,
las podés combinar en una sola **Mejora Masiva** todavía mejor
(`/craftear_mejora_masiva`).

---

## 🛍️ 8. Mini Shop (ítems de 500 a 10.000 MC)

Un catálogo aparte de ítems más baratos y accesibles, organizados por
categorías. La diferencia con las mejoras es que acá **solo podés tener
equipado 1 ítem por categoría a la vez** (si comprás otro de la misma
categoría, reemplaza al anterior).

- `/minishop` — ver el catálogo.
- `/comprar_item` — comprar (y equipar automáticamente).
- `/mis_items` — ver qué tenés equipado ahora.

---

## 🤖 9. Autobots de inversión

Si te cansa estar pendiente del mercado (sección 2), podés comprar un
Autobot que compra y vende MOON COINS solo, según reglas que vos le
configurás.

- `/autobots` — ver el catálogo.
- `/comprar_autobot` — comprar uno (suma usos, no reemplaza el que ya
  tenías).
- `/autobuy` — configurar cómo querés que opere.

Solo podés tener 1 Autobot activo a la vez.

---

## 🏢 10. Negocios

Comprás un negocio y te va generando plata solo con el paso del tiempo,
que después tenés que ir a cobrar.

- `/negocios` — ver el catálogo (hay legales e ilegales).
- `/comprar_negocio` — comprar uno.
- `/mis_negocios` — ver cuánto tenés pendiente de cobrar.
- `/cobrar_negocios` — cobrar todo de una.

Los negocios **ilegales** pagan más pero tienen riesgo de redada: si te
agarran al cobrar, perdés ese negocio. Al cobrar hay un pequeño impuesto
sobre lo que retirás.

**Dos negocios dan premios extra al cobrarlos:**
- **Fiscalía** (legal): de vez en cuando (chance mínima) te regala
  "Papeles legales" (te baja el impuesto al cobrar) o "Empleados
  contentos" (te duplica el % diario de tus negocios legales).
- **Casa de Gobierno** (ilegal): puede regalarte "Vista ciega" (te baja la
  chance de redada en tus negocios ilegales) o "Cronómetro" (duplica el %
  diario de tus negocios ilegales).

Estos 4 premios **no se compran ni se venden**: solo se consiguen así, y
apenas los tenés en el inventario ya están funcionando, no hace falta
"usarlos".

Si no querés estar pendiente de cobrar manualmente, existe el
**Auto Cobrador**: un ítem que cobra tus negocios solo cada cierto tiempo
y te avisa por mensaje privado. `/autocobrador`, `/comprar_autocobrador`,
`/configurar_autocobrador` (para elegir cada cuánto cobra).

---

## 🕶️ 11. Facciones: Mafia y Policía

- `/afiliarse_mafia`, `/afiliarse_policia`, `/desafiliarse`,
  `/traicionar` (cambiarte al bando contrario) — `/faccion` te dice en
  cuál estás.
- Estar en la **Mafia** te sube las chances de éxito en trabajos ilegales,
  y si además tenés el negocio de drogas, te habilita el **mercado
  negro**: `/mercado_drogas`, `/comprar_droga`, `/vender_droga` —
  commodities con precio que fluctúa, para comprar barato y revender caro.
- 💎 Los Server Boosters pueden afiliarse a las dos facciones a la vez con
  `/afiliarse_ambas`, sin tener que elegir.

---

## 🎰 12. Casino

Todo lo que es apostar plata:

- `/blackjack` — contra el bot.
- `/ruleta` — números, colores, docenas, lo de siempre.
- `/tragamonedas` — 3 símbolos, combinaciones y premios.
- `moon carrera_caballos` (se escribe como mensaje normal, no es comando
  con `/`) — elegís un caballo de una lista y apostás a que gane.
- `moon pelea_callejera` (también con `moon`, sin `/`) — 1 contra 1,
  elegís a quién le apostás.

Extras del casino:
- `/jackpot` — el Pozo Acumulado va sumando un poquito de cada apuesta que
  se hace en cualquier juego, y en algún momento alguien se lo puede
  llevar entero.
- `/ranking_ludopatia` — quién ganó más y quién perdió más jugando.

---

## 💎 13. VIP

- `/vip beneficios` — mostrá los distintos niveles VIP y qué te dan cada
  uno (bonus de trabajos, de éxito en ilegales, etc.).

---

## ⭐ 14. Niveles, por chatear

Simplemente por mandar mensajes en el server subís de nivel, y cada nivel
te puede dar una recompensa en MOON COINS.

- `/minivel` — tu nivel actual y cuánto te falta para el siguiente.
- `/topniveles` — el ranking de los que más chatearon.

---

## 🧑 15. Tu perfil

- `/setprofile` — configurá tu perfil personal del server (a través de un
  menú, por partes).
- `/myprofile` — mostrar tu propio perfil.
- `/seeprofile` — ver el perfil de otro usuario.

---

## 🤝 16. Invitaciones y logros

- `/invitaciones` — quién te invitó a vos, y a cuánta gente invitaste vos.
- `/ranking_invitaciones` — el top de invitadores del server.
- `/logros` — logros por juntar cierta cantidad de MOON COINS (o por
  invitar gente), con su progreso.

---

## 💎 17. Ventajas de ser Server Booster

Si boosteás el server tenés acceso a cosas extra: `/beneficios_booster`
te las lista todas (trabajo especial propio, afiliación doble a
facciones, un negocio exclusivo, campos extra en tu perfil, etc.).

---

## 🎲 18. Encuestas con apuesta

- `/moon_encuesta` — crea una encuesta donde la gente apuesta MOON COINS
  a la opción que crea que va a ganar.
- Cuando se cierra, si el resultado no estaba precargado, un mod carga el
  resultado con `/moon_resultado` y ahí se reparten los premios.

---

## 🎭 19. Roles y comunidad

- `/rolshop`, `/buyrol` — comprar roles del server con tus MOON COINS
  (si el server los configuró).
- `/opinasobre` — el bot te tira su "sincera opinión" sobre un usuario,
  del 1 al 10 (es solo joda, no te lo tomes muy en serio 😄).
- `/rank_my_server` — un ranking del server en la comunidad general de
  MOON BOT.
- `/start_count` — el clásico juego de contar en grupo sin repetir
  número ni que la misma persona cuente dos veces seguidas.

---

## ❓ 20. ¿Te perdiste?

- `/moon_help` — lista completa de comandos organizados por sección, con
  botones para navegar.
- `/moon_guias` — lo mismo pero con explicación detallada de cada
  mecánica, no solo el nombre del comando.

---

### 🔄 Cómo se combina todo (resumen rápido)

1. Trabajás (`/trabajar`) → ganás plata + de vez en cuando algún ítem, tal
   vez un fármaco o droga.
2. Completás tareas (`/tareas completar`) → más plata + más chance de
   fármacos/drogas.
3. Usás esos fármacos/drogas (`/sustancias usar`) → mientras estén
   activos, laburás más rápido, ganás más plata, o tenés más suerte en
   el casino.
4. Con la plata que juntás: comprás mejoras permanentes, ítems de la Mini
   Shop, negocios, o la jugás en el mercado/casino para multiplicarla.
5. Los negocios (sobre todo Fiscalía y Casa de Gobierno) te van tirando,
   de yapa, ítems pasivos que te hacen todavía más eficiente cobrando.

¡Y con eso ya tenés todo el panorama para armar tu estrategia! 🌙
