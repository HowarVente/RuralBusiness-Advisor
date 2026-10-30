# RuralBusiness Advisor - Agente en Cloudflare Workers AI

Agente estratega de modelos de negocio rurales, bioeconomía, AgTech y financiación pública (PAC, LEADER, FEADER, PERTE) alojado en la red global de Cloudflare.

## 1. Arquitectura Técnica
- **Alojamiento:** Cloudflare Workers con TypeScript y Hono.
- **Inferencia (Workers AI):**
  - Modelo Principal (Estrategia y redacción): `@cf/meta/llama-3.3-70b-instruct-fp8-fast`
  - Modelo de Razonamiento (Análisis comparativo y financiero): `@cf/deepseek-ai/deepseek-r1-distill-qwen-32b`
  - Modelo de Traducción: `@cf/meta/m2m100-1.2b`
  - Modelo de Resumen: `@cf/meta/llama-3.2-3b-instruct`
- **Almacenamiento:**
  - **D1 (`ruralbusiness-advisor-db`):** Sesiones de chat, caché persistente de consultas y metadatos de convocatorias.
  - **R2 (`ruralbusiness-advisor-storage`):** Almacén de JSONs de convocatorias completas e informes exportados.
  - **KV (`RURALBUSINESS_KV`):** Caché rápida en edge para respuestas de APIs.
- **Servidor MCP Remoto:** Endpoint `/mcp` con protocolo JSON-RPC 2.0 (Streamable HTTP) para conectar Antigravity y otros clientes.
- **API REST / OpenAPI:** Endpoints `/api/chat`, `/api/convocatorias` y `/openapi.json` para Custom GPT Actions.
- **Interfaz Web:** Servida desde Workers Assets (`/public`).

---

## 2. Decisiones Técnicas Justificadas
1. **FAOSTAT:** Se implementa mediante llamada `fetch()` nativa en TypeScript dentro del Worker utilizando `FAOSTAT_TOKEN`. El paquete Python no es soportado en el runtime V8 de Workers y un servidor MCP separado añadiría latencia de red innecesaria. Si el token caduca o falta, la herramienta se desactiva elegantemente sin bloquear la inferencia.
2. **APITube:** Se emplea `fetch()` nativo en TypeScript para consultar noticias y señales de mercado, evitando dependencias de SDKs pesados.
3. **Scraping en Cascada:**
   - Nivel 1: Cloudflare Browser Rendering `/crawl` (nativo en la cuenta).
   - Nivel 2: Apify Actors mediante API REST (`APIFY_TOKEN`).
   - Nivel 3: Respaldo de extracción de texto y metadatos limpios.

---

## 3. Configuración de Secretos

Para activar las herramientas de inteligencia opcionales en producción, ejecuta:

```bash
# Token obligatorio para proteger /mcp y /api
npx wrangler secret put MCP_AUTH_TOKEN

# APIs externas opcionales (si no se configuran, la herramienta queda desactivada limpiamente)
npx wrangler secret put EXA_API_KEY
npx wrangler secret put PERPLEXITY_API_KEY
npx wrangler secret put APITUBE_API_KEY
npx wrangler secret put PATSNAP_API_KEY
npx wrangler secret put APIFY_TOKEN
npx wrangler secret put FAOSTAT_TOKEN
```

---

## 4. Despliegue en Cloudflare

```bash
# Comprobar tipos TypeScript
npm run build

# Desplegar en producción
npx wrangler deploy
```

---

## 5. Pruebas y Validación

El agente cuenta con la suite de pruebas unitarias que valida:
1. El caso de prueba de los snacks de garbanzo deshidratado (generando 3 variantes y 3 ideas).
2. El funcionamiento de la caché D1/KV.
3. El esquema formal de convocatorias (`schemas/condiciones.schema.json`).
4. El servidor MCP y el endpoint `/openapi.json`.

