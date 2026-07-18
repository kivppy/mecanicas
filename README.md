# 🌙 LUNA V 1.0 — Bot de Discord

## Instalación

```bash
pip install -r requirements.txt
cp .env.example .env
# editar .env y poner tu DISCORD_TOKEN
python3 main.py
```

⚠️ Este zip **no incluye `cogs/roullete.py` tal como lo tenías vos si era distinto al que armé acá** — si ya tenías uno propio, este lo reemplaza por una versión nueva con animación de giro. Si preferís tu versión original, no sobreescribas ese archivo al descomprimir.

## Configuración

En `.env`:
- `DISCORD_TOKEN`: token del bot.
- `STAFF_ROLES`: nombres de roles considerados Staff, separados por coma.
- `DEV_GUILD_ID`: **ya no se usa.** Los slash commands se sincronizan solos (ver "Notas técnicas").

Intents requeridos en el Developer Portal: `SERVER MEMBERS INTENT` y `MESSAGE CONTENT INTENT`.

## Comandos

### Staff / Admin
- `/help_moon`, `/informacion`, `/setlogs`, `/getmyinfo`, `/newsrol`
- `/welcome_channel_set canal` — define el canal donde se publican las bienvenidas a nuevos miembros.
- `/welcome_message_set` — abre un panel visual (con vista previa en vivo) para editar el título, la descripción y la imagen/GIF del mensaje de bienvenida. Soporta los placeholders `{usuario}`, `{usuario_nombre}`, `{servidor}` y `{conteo}`.

### Juegos
- `/blackjack [apuesta]`
- `/ruleta monto tipo valor` — número (0-36, x36), color rojo/negro (x2), par/impar (x2), mitad 1-18/19-36 (x2), docena 1/2/3 (x3).
- `/start_count` — arranca un juego grupal de conteo en el canal: los usuarios van mandando números en orden (1, 2, 3...) alternándose entre sí. Si alguien manda el número equivocado, cuenta dos veces seguidas, o si llegan al límite de 1000, el juego termina y **solo los participantes** se reparten MOON COINS (0.1 por cada número correcto contado entre todos).

### Comunidad
- `/opinasobre @usuario` — MOON BOT tira una opinión random del 1 al 10 sobre ese usuario, con GIF incluido.
- `/moon_invite` — te tira el link para invitar a MOON BOT a otro servidor.
- `/rank_my_server` — analiza de verdad tu servidor (canales, categorías, roles, miembros, ícono, descripción, boosts, emojis) y le pone un puntaje del 1 al 10 en base a qué tan completo está.
- `/votar_user @usuario` — le da un voto único a ese usuario para el ranking de más queridos del server (no podés votar dos veces al mismo usuario, ni votarte a vos mismo, ni votar a un bot).
- `/votes_rank` — muestra el top 5 de usuarios más votados/queridos del servidor.

### Economía
- `/chargemoon usuario monto` — solo dueño/admin.
- `/economia` — balance, deuda e historial.
- `/prestamo cantidad` — pedí entre 200 y 100.000 🌙 (10% de interés).
- `/deuda` — consultá tu deuda activa.
- `/topmoon` — ranking de riqueza.
- `/moonchart` — evolución del precio de la MOON COIN.
- `/trabajar trabajo [inversion]` — legales (1-25 🌙, cooldown 10 min) e ilegales (25-50 🌙 + bonus, requieren inversión, cooldown 20-35 min).

### Moon Encuestas (apuestas grupales)
- `/encuestas_config canal` — admin, define el canal de publicación.
- `/moon_encuesta` — admin, abre un modal para crear la encuesta (nombre, opciones, monto mínimo, duración, texto extra, imagen y resultado ganador opcionales).
- Los usuarios apuestan clickeando los botones de la encuesta publicada (piden monto vía modal). Sin MOON COINS no se puede participar.
- Si el admin no fijó un resultado ganador al crearla, al cerrar debe cargarlo con `/moon_resultado`.
- Los ganadores reciben el doble de lo apostado.

## Notas técnicas

- Todo persiste en JSON dentro de `data/` (por servidor), sin base de datos externa.
- **Precio de la MOON COIN:** ya no fluctúa por porcentajes. Cada tick (cada 5 minutos, y también en cada Alerta de Inversiones) suma o resta un número entero al azar entre 1 y 500 al precio actual. El mercado se mueve por "episodios de tendencia": rachas de varios ticks seguidos donde el precio solo sube, seguidas de rachas donde solo baja, sorteadas 50/50. El techo es 100.000 (al tocarlo arranca automáticamente una racha bajista) y el piso es 1 (al tocarlo arranca automáticamente una racha alcista), para que el precio nunca quede pegado en un extremo.
- Las Moon Encuestas se cierran automáticamente vía tarea en segundo plano cada 60 segundos.
- **Bienvenidas:** al unirse un usuario nuevo (no bots), si hay un canal configurado con `/welcome_channel_set`, se le envía automáticamente el mensaje configurado con `/welcome_message_set` (o uno por defecto si no se configuró nada).
- **Sincronización de slash commands:** es automática, ya no existen comandos manuales `/sync` ni `/resync`. Al arrancar, el bot compara un "fingerprint" de la lista de comandos actual contra la del último arranque guardado en `data/sync_state.json`; si no cambió nada, no llama a la API de Discord (para no gastar rate limit en cada reinicio). Si hay cambios, sincroniza solo. También hace, una única vez en la vida del bot, una limpieza de comandos viejos registrados a nivel de servidor (restos de configuraciones anteriores), dejando como única fuente de verdad los comandos globales.
