# Checkpoint 4 — Sincronización del Cerebro Agéntico con Ecosistemas de Negocio

**Proyecto:** Marimoon (comunidad freemium de anime/otaku para Latinoamérica)
**Autor:** David Toyama
**Archivo del workflow:** `checkpoint4_david_toyama.json`

## Qué hace este workflow

Automatiza el triage de la casilla de soporte al cliente de Marimoon, interconectando tres herramientas reales del negocio:

- **Gmail** — casilla de soporte (trigger de entrada + creación de borradores de respuesta)
- **HubSpot** — CRM, registro/actualización de contactos que escriben a soporte
- **Slack** — canal `#ops-marimur`, notificación al equipo de operaciones

## Los 4 nodos clave que evalúa la rúbrica

1. **① IF anti-auto-reply** — justo después del Gmail Trigger, descarta correos con asuntos/remitentes de tipo Auto-reply, Out of Office, Undeliverable o no-reply/noreply, cortando el bucle infinito de auto-respuestas.
2. **② Look up antes del Create en HubSpot** — busca si el contacto ya existe antes de crear uno nuevo, evitando el Error 409 de duplicados.
3. **③ Create Draft en Gmail (Human-in-the-loop)** — la respuesta generada por el Agente de IA nunca se envía sola: siempre queda como borrador, pendiente de revisión y aprobación humana.
4. **④ Set de limpieza y validación de payload** — deja solo los campos necesarios (email, nombre, asunto, thread, respuesta) y valida que el email del remitente no esté vacío antes de continuar, evitando errores 400.

## Autenticación de los conectores

- **Gmail**: OAuth2, scope mínimo (`gmail.modify` — leer, componer y enviar/crear borradores), sin el scope de borrado total.
- **Slack**: OAuth2, Bot Token Scopes acotados a `chat:write` y `channels:read`.
- **HubSpot**: ⚠️ **Desviación declarada respecto a la consigna.** La consigna pide OAuth2 para los tres conectores. Para HubSpot no fue posible completar ese flujo por dos motivos fuera de nuestro control:
  1. HubSpot deshabilitó la creación de nuevas "Legacy Public Apps" (el flujo OAuth2 clásico de 3 patas) para cuentas nuevas.
  2. El camino alternativo — generar la app vía HubSpot CLI (`hs project create`) — falló de forma reproducible por una incompatibilidad entre la CLI y Node.js v25 (bleeding edge, aún no soportado oficialmente por la herramienta).

  Como solución pragmática, se usó una **Service Key** de HubSpot (credencial de cuenta única, no OAuth2 de 3 patas), con scopes acotados exclusivamente a `crm.objects.contacts.read` y `crm.objects.contacts.write` — cumpliendo igualmente el principio de mínimo privilegio, aunque no el protocolo OAuth2 exacto que pide la consigna.

## Regression testing

Cada nodo fue probado individualmente (Test step) y el workflow completo fue ejecutado de punta a punta más de una vez con correos reales, confirmando ambas ramas del IF anti-auto-reply (descarte y procesamiento completo) y la creación/actualización de contactos en HubSpot sin duplicados.

## Importar en n8n

1. Descargar `checkpoint4_david_toyama.json` de este repositorio.
2. En n8n: **Workflow → Import from File** (o arrastrar el archivo al canvas).
3. Reconectar las credenciales de Gmail OAuth2, HubSpot Service Key y Slack OAuth2 con las propias del entorno donde se importe (las credenciales no se exportan por seguridad).
