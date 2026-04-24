# PokeDex — Despliegue en la Nube

## ¿Qué es esto?

Es una aplicación web que te permite explorar diferentes Pokémon, ver su información, habilidades y características. El código fue desarrollado por PumasLab y mi trabajo fue desplegarlo en la nube de forma segura.

- **Repositorio:** https://github.com/victordmarrugo/pokedex
- **App en vivo:** https://polite-ground-0d1b20a0f.7.azurestaticapps.net

---

## Cómo creé la cuenta en Azure

Esto lo hice el viernes 18 de abril en clase. Nunca había usado Azure antes así que fue todo nuevo para mí.

1. Entré a [https://azure.microsoft.com](https://azure.microsoft.com) y le di clic a **"Iniciar gratis"**
2. Inicié sesión con mi cuenta personal de Microsoft
3. Llené el formulario con mis datos: nombre, país, número de teléfono
4. Me pidió una tarjeta de crédito para verificar identidad, pero aclara que no cobra nada en el plan gratuito
5. Acepté los términos y me llegó un correo de confirmación
6. Entré a [https://portal.azure.com](https://portal.azure.com) y ya tenía acceso

Fue más sencillo de lo que esperaba, aunque el portal al principio se ve bastante abrumador porque tiene muchísimas opciones.

---

## Reflexión personal

### ¿Para qué sirven los encabezados de seguridad que configuré?

Antes de este proyecto honestamente no sabía que existían. Son instrucciones que el servidor le manda al navegador para decirle cómo comportarse:

- **Content-Security-Policy** — le dice al navegador de dónde puede cargar cosas. Evita que alguien inyecte código malicioso en la página.
- **Strict-Transport-Security** — obliga a que la conexión siempre sea HTTPS y no HTTP.
- **X-Content-Type-Options** — evita que el navegador "adivine" qué tipo de archivo es algo, lo cual puede ser peligroso.
- **X-Frame-Options** — impide que alguien meta tu página dentro de otra en un iframe, una técnica de ataque bastante conocida.
- **Referrer-Policy** — controla qué información se comparte cuando alguien hace clic en un enlace de tu app.
- **Permissions-Policy** — bloquea acceso a cámara, micrófono y ubicación. La PokeDex no necesita nada de eso.

### ¿Qué aprendí?

Que desplegar una app no es solo subirla y ya. Hay que pensar en cómo está configurada y qué tan expuesta queda. Con un solo archivo de configuración pude pasar de una app sin protección a una con calificación A en seguridad. Eso me pareció bastante poderoso para ser algo tan simple de hacer.

### Los problemas que tuve

Fueron varios y me quitaron bastante tiempo, pero los fui resolviendo. Los detallo paso a paso en `Despliegue.md`.

---

*Víctor D. Marrugo — Abril 2026*
