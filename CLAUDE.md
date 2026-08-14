# Fleet Card — Calculadora de Ahorro

Contexto para Claude Code. Este proyecto es una calculadora de ahorro fiscal para clientes de **Clara Fleet Card** (tarjeta de flotilla para gasto en combustible, México).

## Qué es

Un calculador HTML de una sola página (`fleet-card-calculator.html`), self-contained (todo el CSS y JS inline, sin dependencias). Estima cuánto ahorra un cliente al mes al facturar sus consumos de combustible con Fleet Card.

**Audiencia:** cold traffic (self-serve web), dueños de flotilla en México. NO asumas que el usuario sabe términos fiscales.

**Idioma de la UI:** español (es-MX).

## La lógica del cálculo

El ahorro se compone de tres partes:

```
Ahorro neto = IVA que puedes acreditar  +  ISR que dejas de pagar  −  Comisión Fleet Card
```

1. **IVA acreditable** = `gasto × (16 / 116)`
   - El precio de gasolina en bomba YA incluye IVA. Se divide entre 116 (no 100) para separar el impuesto ya incluido.
   - Con CFDI 4.0 + complemento de combustibles, ese IVA se acredita contra el IVA que el cliente cobra a sus clientes.

2. **ISR (deducción)** = `(gasto − IVA acreditable) × tasa_ISR`
   - Se calcula sobre el gasto SIN IVA (el IVA ya se contó aparte, no doble-contar).
   - La gasolina es un gasto deducible que baja la utilidad gravable → menos ISR.
   - Asume que el cliente tiene utilidades gravables. Si tiene pérdidas ese año, el beneficio se pospone como pérdida fiscal (usable hasta 10 años).

3. **Comisión Fleet Card** = `gasto × 1.5%`
   - Se cobra sobre el 100% del gasto que pasa por la tarjeta.
   - 1.5% es fijo y confirmado por producto. No bajar de 1.5%.

## Tasas de ISR por régimen (VERIFICADAS contra LISR)

| Régimen | Tasa ISR | Base legal | Notas |
|---|---|---|---|
| **PMG** (Persona Moral General) | 30% | LISR Art. 9 | Tasa estándar. |
| **AGAPES** (Sector Primario) | ~21% efectivo | LISR Art. 74, párrafo 12 | 30% con reducción del 30% sobre ISR determinado, para ingresos entre 20 y 423 UMA anual. Se topa arriba de 4,230 UMA. |
| **Coordinados** (autotransporte) | 30% | LISR Arts. 72-73 → Art. 9 | Sin tasa preferencial en combustibles. Integrantes PM aplican 30%; integrantes PF aplican tarifa progresiva (hasta 35%). La facilidad del 8% "sin comprobantes" NO aplica a combustibles. |

**Hallazgo importante:** Coordinados NO tiene ventaja de tasa ISR sobre combustible vs. PMG. La diferenciación del pitch a un Coordinado es el acreditamiento de IVA y el flujo de efectivo, no la tasa.

## Decisiones de producto tomadas (no revertir sin discutir)

- **NO** incluir inputs de tamaño de flota, tipo de flota, ni ingresos del cliente — no afectan la matemática.
- **NO** incluir un slider de "% que ya facturas hoy" — se probó y confundía a los usuarios. Se eliminó. El cálculo ahora asume 0% facturado hoy (muestra ahorro máximo). Está documentado en las notas al pie.
- **Etiquetas de cara al cliente** usan lenguaje simple, no jerga:
  - "IVA que puedes acreditar" (término correcto — es *acreditar*, no *deducir*)
  - "ISR que dejas de pagar sobre tus utilidades" (es *utilidades*, no *ingresos*)
  - "Comisión Fleet Card (1.5%)"
- Los términos técnicos legales (acreditable, deducción, referencias a LISR) viven en las **notas al pie**, para el contador que revise.
- Todo el lenguaje debe ser de **AHORRO**, nunca de "profit/ganancia".

## Pendientes / cosas NO verificadas (resolver antes de lanzar a cold traffic)

1. **Default de AGAPES (21%)** — validar con el equipo fiscal de Clara los umbrales UMA exactos y la tasa efectiva real.
2. **Precio promedio de gasolina (placeholder $24/L)** — el cálculo de litros es solo referencial. Conseguir promedio actualizado (CRE publica semanal) o quitar el estimado de litros.
3. **Cobertura de estaciones afiliadas** — el disclaimer es genérico. Cuando haya un número real de cobertura al lanzamiento, meter un factor de ajuste o footnote con el % real.
4. **Estímulo fiscal al diésel (LIF Art. 16)** — acreditamiento adicional de IEPS para autotransporte. NO está en el cálculo. Solo aplica a diésel + autotransporte. Requiere un input de tipo de combustible si se quiere modelar.
5. **Default de gasto** — actualmente bajo para el ICP real de flotillas ($50k–$1M+/mes). Considerar subirlo a ~$50,000 para más impacto.
6. **Disclaimer de utilidades** — el cálculo de ISR asume que el cliente tiene utilidades gravables. Considerar un aviso explícito.

## Notas técnicas

- Archivo único, sin build step, sin dependencias externas (salvo la fuente DM Sans de Google Fonts).
- Usa tokens de color del design system de Clara (--primary #335894, etc.).
- Para probar localmente: `python3 -m http.server 8000` y abrir `http://localhost:8000/fleet-card-calculator.html`.

## Aviso legal

Los cálculos son referenciales y no constituyen asesoría fiscal. Todo supuesto fiscal debe ser validado por el equipo fiscal de Clara antes de publicar.
