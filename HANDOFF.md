# Handoff — Fleet Card Calculadora de Ahorro

**Estado:** Funcional, en revisión antes de lanzar a cold traffic.
**Archivo:** `fleet-card-calculator.html` (single-file, sin build, sin dependencias salvo DM Sans de Google Fonts).
**Idioma UI:** español (es-MX). **Audiencia:** dueños de flotilla en México, self-serve.

---

## Cómo correrlo

El archivo es HTML self-contained, pero **necesita servirse por HTTP** para que corra el JavaScript. Si lo abres como archivo estático (doble clic o preview de archivo), el script no ejecuta y **todos los valores quedan en $0**.

```bash
cd /Users/clara/Downloads/fleet-calculator
python3 -m http.server 8000
```

Luego abre: `http://localhost:8000/fleet-card-calculator.html`

---

## Qué calcula

```
Ahorro neto = IVA acreditable  +  ISR que dejas de pagar  −  Comisión Fleet Card
```

| Componente | Fórmula | Ejemplo ($100,000/mes, PMG 30%) |
|---|---|---|
| IVA acreditable | `gasto × 16/116` | + $13,793 |
| ISR (deducción) | `(gasto − IVA) × tasa_ISR` | + $25,862 |
| Comisión Fleet Card | `gasto × 1.5%` (fija, no bajar) | − $1,500 |
| **Ahorro neto mensual** | | **$38,155** ($457,862/año) |

Verificado: la lógica coincide con la especificación en `CLAUDE.md`. Todos los componentes calculan correctamente y el dropdown de régimen + opciones avanzadas están conectados.

### Tasas de ISR por régimen (verificadas contra LISR)
- **PMG** (Persona Moral General): 30% — Art. 9 LISR.
- **AGAPES** (Sector Primario): ~21% efectivo — Art. 74, párrafo 12. ⚠️ Default por validar.
- **Coordinados** (autotransporte): 30% — Arts. 72-73 → Art. 9. Sin ventaja de tasa vs. PMG en combustible.

---

## Decisiones de producto (no revertir sin discutir)

- Sin inputs de tamaño de flota, tipo de flota, ni ingresos — no afectan la matemática.
- Sin slider de "% que ya facturas hoy" — se probó, confundía, se eliminó. El cálculo asume 0% facturado (ahorro máximo), documentado en notas al pie.
- Etiquetas de cara al cliente en lenguaje simple ("IVA que puedes acreditar", "ISR que dejas de pagar sobre tus utilidades"). Jerga legal (LISR, artículos) vive solo en las notas al pie.
- Todo el lenguaje es de **AHORRO**, nunca "ganancia/profit".

---

## Pendientes antes de lanzar a cold traffic

1. **Default de AGAPES (21%)** — validar umbrales UMA y tasa efectiva con el equipo fiscal de Clara.
2. **Precio de gasolina (placeholder $24/L)** — conseguir promedio actualizado (CRE, semanal) o quitar la estimación de litros.
3. **Cobertura de estaciones afiliadas** — el disclaimer es genérico; meter % real al lanzamiento.
4. **Estímulo fiscal al diésel (LIF Art. 16)** — acreditamiento adicional de IEPS para autotransporte, no modelado. Requiere input de tipo de combustible.
5. **Default de gasto** — actualmente $100,000/mes. El ICP real es $50k–$1M+/mes; considerar ajustar.
6. **Disclaimer de utilidades** — el cálculo de ISR asume utilidades gravables; considerar aviso explícito.

## Bug menor observado

En la primera carga, el campo de gasto mostró brevemente un número basura antes de estabilizarse — el autofill del navegador inyectando en el input numérico. Es artefacto del navegador, no del código. Mitigación sugerida: `autocomplete="off"` en el input `#fuelSpend`.

---

## Aviso legal

Los cálculos son referenciales y no constituyen asesoría fiscal. Todo supuesto debe ser validado por el equipo fiscal de Clara antes de publicar.
