# 🌙 MOON BOT — README

**Versión:** V 2.5.9
**Estado:** ⚠️ Bot en fase **BETA** — sujeto a cambios, apagones o reinicios sin aviso previo.

MOON BOT es un bot de Discord todo-en-uno pensado como una **economía RPG completa**: tiene su propia moneda (**MOON COIN**), un mercado con precio flotante, casino, trabajos, facciones tipo Mafia/Policía, negocios, pandillas, mercado negro, un sistema de mejoras y crafteo, autobots de trading, mercado entre usuarios, encuestas con apuestas, niveles por actividad, logros, sistema de invitaciones, beneficios para Server Boosters, utilidades de administración (logs, roles) y hasta consultas de anime.

Este documento es una guía **100% orientada al uso** del bot (comandos, configuración y mecánicas). No contiene código ni detalles de implementación.

---

## 📑 Índice

1. [Primeros pasos y configuración](#1-primeros-pasos-y-configuración)
2. [Roles y niveles de permisos](#2-roles-y-niveles-de-permisos)
3. [Lista completa de comandos](#3-lista-completa-de-comandos)
4. [Ejemplos de uso](#4-ejemplos-de-uso)
5. [Mecánicas del bot](#5-mecánicas-del-bot)
6. [Cosas extra del bot](#6-cosas-extra-del-bot)
7. [Notas finales](#7-notas-finales)

---

## 1. Primeros pasos y configuración

Cuando MOON BOT se une a un servidor nuevo, envía automáticamente un **mensaje de bienvenida** en un canal de texto (prioriza el canal del sistema del servidor) presentando sus funciones principales y avisando que está en fase BETA.

A partir de ahí, hay una serie de canales que **el Staff o los administradores** deben configurar una sola vez para que ciertas funciones empiecen a funcionar. Ningún sistema deja de andar si no se configura: simplemente no manda avisos a ningún canal hasta que se le indique uno.

### Checklist de configuración recomendada

| Sistema | Comando de configuración | Quién puede usarlo |
|---|---|---|
| Logs del servidor | `/configurar_logs` | Administrador |
| Registro de mensajes (on/off) | `/setlogs` | Staff |
| Alertas de logros | `/configurar_logros` | Staff |
| Alertas de invitaciones | `/configurar_invitaciones` | Staff |
| Mensaje visual de invitaciones | `/configurar_mensaje_invitaciones` | Staff |
| Alertas de subida de nivel | `/configurar_canal_niveles` | Staff |
| Mensaje visual de subida de nivel | `/configurar_mensaje_nivel` | Staff |
| Recompensas por nivel | `/configurar_recompensas_nivel` | Staff |
| Rol automático de Booster | `/configurar_rol_booster` | Staff |
| Canal de Moon Encuestas | `/encuestas_config` | Dueño / Admin |
| **Alertas de inversiones (USD)** | `/configurar_alertas_inversiones` | Staff |
| Tienda de roles | `/configshop` / `/eliminarol` | Dueño / Admin |

> 💡 Todas estas configuraciones son **por servidor**: cada servidor donde esté MOON BOT tiene su propia economía, mercado, niveles, negocios, etc. Nada se comparte entre servidores.

---

## 2. Roles y niveles de permisos

MOON BOT reconoce distintos niveles de acceso:

- **👤 Usuario común:** puede usar todos los comandos de juego, economía, casino, trabajos, etc.
- **🛡️ Staff:** cualquier miembro con un rol llamado **Staff, Admin o Moderador** (configurable por quien aloja el bot), o con permiso de Administrador, o el dueño del servidor. Puede usar los comandos de configuración de canales y mensajes.
- **👑 Dueño del servidor / Administrador:** algunos comandos sensibles (tienda de roles, cargar MOON COINS a mano, encuestas) están restringidos exclusivamente a quien tiene permiso de **Administrador** o es el dueño del servidor, sin aceptar el rol genérico de Staff.
- **🔧 Dueño del bot:** comandos de mantenimiento (`/sync`, `/resync`) y el anuncio global (`/moon_anuncio`) están reservados exclusivamente a quien aloja/administra el bot a nivel técnico, no a los administradores de cada servidor.

Si un usuario sin permisos intenta usar un comando restringido, el bot responde con un aviso de error explicando qué falta.

---

## 3. Lista completa de comandos

> Los parámetros entre `< >` son obligatorios y entre `[ ]` son opcionales.

### 📌 General

| Comando | Descripción |
|---|---|
| `/moon_help` | Panel con todos los comandos de MOON BOT ordenados por sección, navegable con botones. |
| `/moon_guias` | Igual que el anterior, pero con una explicación detallada de cómo funciona cada comando y mecánica. |
| `/help_moon` | Versión clásica del panel de ayuda, navegable con flechas ⬅️➡️. |
| `/informacion` | Muestra datos generales del servidor: miembros, canales, fecha de creación, etc. |
| `/getmyinfo` | Genera un archivo `.txt` con tu información dentro del servidor (roles, fechas, estadísticas). |
| `/newsrol <usuario> <nombre>` | Crea un rol personalizado y lo asigna al usuario indicado. *(Staff)* |
| `/setlogs` | Activa o desactiva el registro de mensajes del servidor. *(Staff)* |
| `/configurar_logs <canal>` | Define el canal donde se envían los logs del servidor. *(Administrador)* |

### 🎮 Casino

| Comando | Descripción |
|---|---|
| `/blackjack <apuesta>` | Partida de blackjack 1 jugador contra el bot. |
| `/blackjack_1v1 <usuario> <apuesta>` | Duelo de blackjack contra otro usuario; el pozo es la apuesta ×2 y se lo lleva el ganador. |
| `/ruleta <apuesta> <tipo>` | Apostá a número, color o mitad; el pago varía según el tipo de apuesta. |
| `/tragamonedas <apuesta>` | Tragamonedas de 3 slots; ciertas combinaciones pagan distintos multiplicadores. |
| `/ranking_ludopatia` | Ranking de quiénes más ganaron y más perdieron jugando en el casino. |

### 💰 Economía

| Comando | Descripción |
|---|---|
| `/economia` | Muestra tu balance de MOON COINS, tu wallet en USD virtuales y tu valor total estimado. |
| `/prestamo <cantidad>` | Pedí un préstamo de entre 200 y 100.000 MOON COINS. |
| `/deuda` | Consultá tu deuda activa. |
| `/pagardeuda <cantidad>` | Pagá parte o toda tu deuda manualmente. |
| `/vendermoon <cantidad>` | Vendé MOON COINS por tu wallet en USD, al precio actual de mercado. |
| `/comprarmoon <cantidad>` | Comprá MOON COINS con tu wallet en USD, al precio actual de mercado. |
| `/moonchart` | Gráfico de la evolución reciente del precio del MOON COIN. |
| `/topmoon` | Ranking de los usuarios con más MOON COINS del servidor. |
| `/chargemoon <usuario> <monto>` | Acredita MOON COINS manualmente a un usuario. *(Dueño / Admin)* |

### 💵 Alertas de Inversiones

| Comando | Descripción |
|---|---|
| `/configurar_alertas_inversiones <canal>` | Define el canal donde el bot manda, en momentos aleatorios, alertas de subas y caídas del precio del USD que **mueven de verdad** el precio del MOON COIN. *(Staff)* |

### 🧉 Trabajo

| Comando | Descripción |
|---|---|
| `/trabajar <trabajo> [inversion]` | Trabajá para ganar MOON COINS. Elegís entre trabajo legal o ilegal. |
| `/trabajo_booster` | Trabajo exclusivo para Server Boosters, con cooldown corto. |

### 🎒 Ítems de trabajo

| Comando | Descripción |
|---|---|
| `/moon_tienda_items` | Catálogo de ítems que se pueden obtener trabajando, con probabilidad y precio, ordenado por rareza. |
| `/moon_inventario [usuario]` | Muestra tu inventario de ítems (o el de otro usuario). |
| `/moon_vender_item <item> <cantidad>` | Vendé ítems de tu inventario por MOON COINS. |
| `/moon_item_personalizado <...>` | Crea o edita un ítem personalizado del sistema de trabajos (nombre, emoji, rareza, probabilidad, precio, etc.). *(Admin)* |

### 🕵️ Facciones

| Comando | Descripción |
|---|---|
| `/faccion` | Consultá tu facción actual y sus beneficios activos. |
| `/afiliarse_mafia` | Unite a la facción Mafia. |
| `/afiliarse_policia` | Unite a la facción Policía. |
| `/afiliarse_ambas` | Afiliate a Mafia y Policía al mismo tiempo. *(Solo Boosters)* |
| `/desafiliarse` | Salí de tu facción actual (quedás como civil). |
| `/traicionar` | Cambiá directamente de Mafia a Policía o viceversa. |

### 🌑 Mercado negro

| Comando | Descripción |
|---|---|
| `/mercado_drogas` | Precios actuales del mercado negro (requiere ser Mafia + tener el Negocio de Drogas). |
| `/comprar_droga <droga> <cantidad>` | Comprá unidades de una droga al precio actual. |
| `/vender_droga <droga> <cantidad>` | Vendé unidades de una droga al precio actual. |

### 🏢 Negocios

| Comando | Descripción |
|---|---|
| `/negocios` | Catálogo de negocios legales e ilegales disponibles para comprar. |
| `/comprar_negocio <negocio>` | Comprá un negocio, que empieza a generar ganancias pasivas con el tiempo. |
| `/mis_negocios` | Muestra tus negocios y las ganancias pendientes de cobro. |
| `/cobrar_negocios` | Cobrá las ganancias pendientes de todos tus negocios. |

### 🔫 Pandillas

| Comando | Descripción |
|---|---|
| `/crear_pandilla <nombre>` | Creá tu propia pandilla (cuesta 500.000 MOON COINS). |
| `/invitar_pandilla <usuario>` | Invitá a un usuario a tu pandilla. |
| `/pandilla [usuario]` | Info de tu pandilla o la de otro usuario. |
| `/mision_pandilla` | Iniciá una misión grupal (2 a 8 integrantes). |

### 🛒 Tiendas

| Comando | Descripción |
|---|---|
| `/rolshop` | Roles disponibles para comprar con MOON COINS. |
| `/buyrol <rol>` | Comprá un rol de la tienda. |
| `/configshop <rol> <precio>` | Agregá o actualizá un rol en la tienda. *(Dueño / Admin)* |
| `/eliminarol <rol>` | Sacá un rol de la tienda. *(Dueño / Admin)* |
| `/minishop` | Catálogo de la Mini Shop (ítems entre 500 y 10.000 MOON COINS). |
| `/comprar_item <item>` | Comprá y equipá un ítem de la Mini Shop. |
| `/mis_items` | Muestra tu build actual de ítems equipados. |

### 🚀 Mejoras y Boosters

| Comando | Descripción |
|---|---|
| `/mejoras` | Catálogo de mejoras permanentes y cuáles ya comprás. |
| `/comprar_mejora <mejora>` | Comprá una mejora permanente. |
| `/beneficios_booster` | Lista de beneficios exclusivos para Server Boosters. |
| `/configurar_rol_booster <rol>` | Define qué rol se entrega automáticamente a los Boosters. *(Staff)* |

### 🔧 Crafteo ELITE

| Comando | Descripción |
|---|---|
| `/materiales_elite` | Ver qué te falta para craftear cada mejora ELITE. |
| `/craftear_elite <mejora>` | Crafteá la versión ELITE de una mejora. |
| `/craftear_mejora_masiva` | Combiná las 4 mejoras ELITE en la Mejora Masiva. |

### 🤖 Autobots de inversión

| Comando | Descripción |
|---|---|
| `/autobots` | Catálogo de Autobots disponibles y tu estado actual. |
| `/comprar_autobot <autobot>` | Comprá un Autobot (suma usos a tu bot de trading). |
| `/autobuy <precio_compra> <precio_venta>` | Configurá tu orden automática de compra/venta de MOON COINS. |

### 🛒 Mercado de usuarios

| Comando | Descripción |
|---|---|
| `/vender_item_usuario <item> <cantidad> <precio>` | Publicá un ítem tuyo a la venta a otros usuarios. |
| `/mercado_items` | Lista de publicaciones activas en el servidor. |
| `/comprar_item_usuario <id_publicacion>` | Comprale un ítem a otro usuario. |
| `/cancelar_venta_item <id_publicacion>` | Cancelá tu publicación y recuperá el ítem. |

### 📊 Encuestas

| Comando | Descripción |
|---|---|
| `/moon_encuesta` | Abre un panel para crear una Moon Encuesta con apuestas en MOON COINS. |
| `/encuestas_config <canal>` | Define el canal donde se publican las Moon Encuestas. *(Dueño / Admin)* |
| `/moon_resultado <encuesta> <opcion_ganadora>` | Carga el resultado de una encuesta cerrada sin resultado preconfigurado. *(Dueño / Admin)* |

### 📈 Niveles, Logros e Invitaciones

| Comando | Descripción |
|---|---|
| `/minivel` | Tu nivel actual, tu XP y cuánto te falta para subir. |
| `/topniveles` | Ranking de usuarios con más nivel/XP. |
| `/logros` | Tus logros de MOON COINS e invitaciones y tu progreso. |
| `/invitaciones` | Quién te invitó y cuántos invitaste vos. |
| `/ranking_invitaciones` | Ranking de quiénes más invitaron gente al servidor. |
| `/configurar_canal_niveles <canal>` | Canal donde se avisa cuando alguien sube de nivel. *(Staff)* |
| `/configurar_recompensas_nivel <nivel> <recompensa>` | Configurá cuántos MOON COINS se entregan al llegar a cada nivel. *(Staff)* |
| `/configurar_logros <canal>` | Canal de alertas de logros. *(Staff)* |
| `/configurar_invitaciones <canal>` | Canal de alertas de invitaciones. *(Staff)* |

### 🎬 Anime

| Comando | Descripción |
|---|---|
| `/getmyanime <nombre>` | Busca info de un anime en AnimeFLV (sinopsis, estado, episodios, géneros). |
| `/rankinganime <nombre>` | Muestra el rating/ranking de un anime en AniDB. |

### 🔧 Administración / Owner del bot

| Comando | Descripción |
|---|---|
| `/moon_anuncio` | Abre un panel para configurar un embed y enviarlo a todos los servidores donde está el bot. *(Solo el dueño del bot)* |
| `/sync` | Sincroniza los slash commands con Discord. *(Solo el dueño del bot)* |
| `/resync` | Borra comandos viejos/fantasma (global y del servidor de desarrollo) y vuelve a registrar todo desde cero. *(Solo el dueño del bot)* |

---

## 4. Ejemplos de uso

### 💰 Economía básica
```
/economia
→ Balance: 🪙 3.250 MOON COINS | Wallet: 12.40 USD | Valor total estimado: ...

/trabajar trabajo:Legal
→ Ganaste 🪙 180 MOON COINS trabajando.

/vendermoon cantidad:1000
→ Vendiste 1.000 MOON COINS a 20.5 USD c/u (comisión 15%) → recibiste 17.42 USD.

/prestamo cantidad:5000
→ Recibiste 🪙 5.000. Deuda total: 🪙 5.500 (10% de interés). Todo ingreso futuro se descuenta automáticamente.
```

### 🎮 Casino
```
/ruleta apuesta:500 tipo:rojo
/blackjack apuesta:1000
/blackjack_1v1 usuario:@Fede apuesta:2000
/tragamonedas apuesta:250
```

### 🕵️ Facciones y mercado negro
```
/afiliarse_mafia
→ Ahora sos parte de la MAFIA.

/comprar_negocio negocio:"Negocio de Drogas"
/mercado_drogas
→ Muestra los precios actuales (requiere ser Mafia + tener ese negocio).

/comprar_droga droga:Cocaína cantidad:10
/vender_droga droga:Cocaína cantidad:10
```

### 🏢 Negocios y pandillas
```
/negocios
/comprar_negocio negocio:"Lavadero de Autos"
/cobrar_negocios
→ Cobraste 🪙 1.240 de ganancias pendientes.

/crear_pandilla nombre:"Los Lunáticos"
/invitar_pandilla usuario:@Fede
/mision_pandilla
→ Espera a que se sumen entre 2 y 8 integrantes (90 segundos) y luego arranca la misión.
```

### 🚀 Mejoras y crafteo ELITE
```
/mejoras
/comprar_mejora mejora:"Boost Trabajo Legal"
/materiales_elite
/craftear_elite mejora:"Boost Trabajo Legal"
/craftear_mejora_masiva
```

### 🤖 Autobots de inversión
```
/autobots
/comprar_autobot autobot:"Autobot Básico"
/autobuy precio_compra:18.5 precio_venta:22.0
→ El bot comprará solo cuando el precio baje a 18.5 USD y venderá cuando suba a 22.0 USD,
   avisándote por MD en cada operación hasta agotar los usos.
```

### 💵 Configurar y usar las alertas de inversiones
```
/configurar_alertas_inversiones canal:#alertas-usd
→ 📊 Alertas de Inversiones configuradas. Van a llegar en momentos aleatorios,
   entre 10 minutos y 1 hora, moviendo de verdad el precio del MOON COIN.

[El bot manda solo, sin que nadie escriba nada:]
📉 ALERTA DE INVERSIONES — El USD se desploma
Pánico en el mercado: los inversores institucionales huyen en masa.
El precio del MOON COIN se desplomó un 18.42% de golpe.
💵 Precio anterior: 21.30 USD  →  💵 Precio actual: 17.38 USD
```

### 🛒 Mercado entre usuarios
```
/vender_item_usuario item:"Prisma" cantidad:1 precio:5000
/mercado_items
/comprar_item_usuario id_publicacion:12
/cancelar_venta_item id_publicacion:12
```

### 📊 Encuestas con apuestas
```
/encuestas_config canal:#encuestas
/moon_encuesta
→ Se abre un panel para poner título, opciones e imagen antes de publicarla.
   Los usuarios apuestan MOON COINS a la opción que crean ganadora.
/moon_resultado encuesta:"¿Quién gana la final?" opcion_ganadora:"Equipo A"
→ Quienes apostaron a la opción ganadora cobran el doble de lo apostado.
```

### 📈 Configuración de niveles (Staff)
```
/configurar_canal_niveles canal:#subidas-de-nivel
/configurar_mensaje_nivel
→ Abre un panel visual para personalizar título, descripción e imagen del aviso.
/configurar_recompensas_nivel nivel:10 recompensa:2000
→ Al llegar a nivel 10, el usuario recibe 🪙 2.000 automáticamente.
```

### 🔧 Configuración general (Staff/Admin)
```
/configurar_logs canal:#logs
/setlogs
/configurar_invitaciones canal:#invitaciones
/configshop rol:@VIP precio:10000
/rolshop
/buyrol rol:VIP
```

---

## 5. Mecánicas del bot

### 🪙 MOON COIN y el mercado
- El MOON COIN es la moneda principal del servidor. Tiene un **precio en USD virtual** que arranca en 20 USD y **fluctúa automáticamente cada 5 minutos** (±5% de volatilidad).
- Comprar (`/comprarmoon`) y vender (`/vendermoon`) se hace siempre al precio actual de mercado, con una **comisión base del 15%**, reducible con ciertos ítems de la Mini Shop.
- `/moonchart` muestra un gráfico (sparkline) con la tendencia reciente del precio.
- Las **Alertas de Inversiones** son un sistema adicional que, en momentos aleatorios (10 minutos a 1 hora), simula una noticia de mercado y **mueve el precio real** entre un 3% y un 35% de golpe, hacia arriba o hacia abajo — el mismo precio que usan `/moonchart`, `/economia` y los autobots.

### 💸 Préstamos y deuda
- Se puede pedir un préstamo de entre 200 y 100.000 MOON COINS.
- Al pedirlo, se genera una deuda del **110%** de lo solicitado (10% de interés).
- Solo se puede tener **un préstamo activo a la vez**.
- Cualquier ingreso posterior de MOON COINS (trabajo, casino, negocios, etc.) descuenta automáticamente **entre un 10% y un 40%** (al azar) hacia el pago de la deuda, hasta saldarla.

### 🧉 Trabajo
- **Trabajo legal:** cooldown fijo de 5 minutos, menor riesgo.
- **Trabajo ilegal:** cooldown aleatorio de 10 a 20 minutos, mayor riesgo/recompensa y puede fallar.
- Ambos pueden dropear ítems (ver sistema de ítems de trabajo) y se ven afectados por la facción del usuario y sus mejoras compradas.
- Los **Server Boosters** tienen un trabajo exclusivo (`/trabajo_booster`) que paga entre 100 y 1.000 MOON COINS con un cooldown corto de 6 minutos.

### 🎒 Ítems de trabajo
- Los trabajos dropean ítems agrupados por rareza: Comunes, Especiales, Legendarios y Míticos, cada uno con su propia probabilidad, cantidad y precio de venta.
- Los administradores pueden crear ítems personalizados, incluso **ítems volátiles** cuyo precio cambia solo con el tiempo.

### 🕵️ Facciones: Mafia vs. Policía
- Un usuario puede afiliarse a **Mafia**, **Policía** o quedar como civil.
- Cambiar de facción (afiliarse, desafiliarse o traicionar) tiene un **cooldown de 5 minutos**.
- Si un usuario cambia de facción **3 veces o más en 10 minutos**, una facción al azar le quita 500 MOON COINS como penalización por inestabilidad.
- Ser de la **Mafia** es requisito, junto con tener el Negocio de Drogas, para operar en el mercado negro.
- Los **Server Boosters** pueden estar afiliados a **ambas facciones a la vez** (`/afiliarse_ambas`), acumulando los beneficios de las dos.

### 🌑 Mercado negro (drogas)
- Requiere ser de la Mafia **y** tener comprado el Negocio de Drogas.
- Los precios de cada droga fluctúan cada 5 minutos, cotizados en USD virtuales — funciona como un mini-mercado independiente dentro del mercado negro.

### 🏢 Negocios
- Hay negocios **legales** e **ilegales**, cada uno con su propio costo y tasa de ganancia pasiva.
- Las ganancias se acumulan solas con el tiempo, con un **tope de acumulación de 3 días** (si no se cobra, deja de sumar).
- Al cobrar un negocio **ilegal**, existe riesgo de "redada": el usuario puede perder el negocio junto con lo pendiente de cobro.

### 🔫 Pandillas
- Crear una pandilla cuesta **500.000 MOON COINS** y convierte al creador en líder.
- Las misiones grupales (`/mision_pandilla`) requieren entre **2 y 8 integrantes**, con una ventana de **90 segundos** para sumarse antes de arrancar.
- Cada misión tiene un **10% de probabilidad de fallar** (perdiendo entre un 2% y un 10% del botín) y, si sale bien, otorga un bonus extra de **20% o 30%**.

### 🚀 Mejoras permanentes y crafteo ELITE
- Las mejoras (`/mejoras`) son boosts permanentes que se compran una vez y quedan activos para siempre, mejorando trabajos y/o negocios.
- Cada mejora tiene una versión **ELITE**, que se craftea combinando el boost normal + 1 chip exclusivo (obtenido trabajando) + 1 ítem mítico (por ejemplo, un prisma). La versión ELITE suma un **+50% adicional** sobre el boost normal.
- Al craftear las **4 mejoras ELITE**, se desbloquea la **Mejora Masiva**, que suma un **+30% adicional** en trabajos legales, ilegales, negocios legales e ilegales, todo al mismo tiempo.

### 🤖 Autobots de inversión
- Hay **3 Autobots** disponibles, cada uno con un precio y una cantidad de usos: uno básico (1.000 MOON COINS / 3 usos), uno intermedio (5.000 / 8 usos) y uno avanzado (15.000 / 20 usos). Los usos se acumulan si se compran varios.
- Con `/autobuy` se configura un precio de compra y uno de venta; el bot opera automáticamente usando la wallet USD y las MOON COINS del usuario cada vez que el precio de mercado cumple la condición, avisando por mensaje directo en cada operación hasta agotar los usos.
- Las **Alertas de Inversiones** mueven el precio de golpe, así que también pueden disparar operaciones automáticas de los autobots configurados.

### 🛒 Tiendas y Mini Shop
- La **tienda de roles** (`/rolshop`) permite comprar roles de Discord con MOON COINS; solo los admins pueden dar de alta o quitar roles de la tienda.
- La **Mini Shop** (`/minishop`) vende ítems cosméticos y de utilidad entre 500 y 10.000 MOON COINS, algunos con descuento de comisión de trading.
- El **mercado entre usuarios** permite publicar ítems propios a un precio total en MOON COINS; el bot retiene el ítem hasta que se vende o se cancela la publicación.

### 📊 Encuestas con apuestas (Moon Encuestas)
- Se crean con un panel visual donde se configura título, opciones e imagen.
- Los usuarios apuestan MOON COINS a la opción que creen ganadora.
- Al cerrarse (manual o preconfigurado), quienes apostaron a la opción ganadora **cobran el doble** de lo apostado.

### 📈 Niveles y XP
- Se gana XP por mensaje, con un **cooldown anti-spam de 60 segundos** entre mensajes que otorgan XP.
- Cada nivel puede tener una **recompensa en MOON COINS** configurable por el Staff.
- Al subir de nivel, se puede enviar un aviso personalizable (título, descripción e imagen) a un canal configurado.

### 🏆 Logros e invitaciones
- Hay logros vinculados tanto a MOON COINS acumuladas como a cantidad de invitaciones logradas.
- El bot trackea quién invitó a quién, con rankings y alertas configurables por canal.

### 💎 Server Boosters
- Beneficios exclusivos: trabajo especial (`/trabajo_booster`), doble afiliación de facción (`/afiliarse_ambas`), y un rol automático configurable que se entrega al boostear.

---

## 6. Cosas extra del bot

- **🎨 Embeds temáticos con GIFs random:** muchas respuestas del bot (afirmaciones, avisos de problemas, subidas de nivel, apuestas ganadas/perdidas, negocios, mensajes aleatorios) incluyen un GIF elegido al azar de una colección propia, para darle más vida a las respuestas.
- **👋 Mensaje de bienvenida automático:** al sumarse a un servidor nuevo, MOON BOT manda un embed de presentación con sus funciones principales y el aviso de fase BETA.
- **📜 Sistema de logs:** los administradores pueden definir un canal de logs del servidor y activar/desactivar el registro de mensajes.
- **🗂️ Exportación de datos personales:** `/getmyinfo` genera un archivo `.txt` descargable con la información del usuario en el servidor.
- **🎛️ Paneles interactivos con botones:** el sistema de ayuda (`/moon_help`, `/moon_guias`, `/help_moon`), la configuración de mensajes de nivel/invitaciones y la creación de Moon Encuestas usan menús con botones en vez de comandos largos con muchos parámetros.
- **🌐 Anuncios globales:** el dueño del bot puede mandar un embed configurable a **todos los servidores** donde esté MOON BOT con un solo comando (`/moon_anuncio`).
- **🎬 Integración con AnimeFLV y AniDB:** `/getmyanime` y `/rankinganime` permiten buscar información y ratings de anime sin salir de Discord.
- **🛠️ Comandos de mantenimiento:** `/sync` y `/resync` permiten al dueño del bot sincronizar o limpiar comandos "fantasma" con Discord sin tener que reiniciar el bot.
- **🔒 Economía aislada por servidor:** cada servidor tiene su propio precio de MOON COIN, su propio mercado negro, su propio ranking, etc. — nada se mezcla entre comunidades distintas.

---

## 7. Notas finales

- ⚠️ **MOON BOT está en fase BETA.** Puede haber cambios de balance, nuevas mecánicas, interrupciones temporales o reinicios sin aviso previo mientras se sigue puliendo.
- Todos los valores en USD mencionados a lo largo de este documento son **moneda virtual interna del bot**, sin ningún vínculo con dinero real.
- Si algo no funciona como se espera, lo más rápido es reportarlo al Staff del servidor o a quien administra el bot.

---

*Generado para MOON BOT V 2.5.*
