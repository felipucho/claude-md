# Mejores prácticas (2025-2026) para escribir prompts de investigación profunda dirigidos a Claude y otros LLMs avanzados

## TL;DR
- Un prompt de investigación profunda de alta calidad **no** es un párrafo largo; es un contrato estructurado con cinco bloques obligatorios: rol + objetivo, criterios de éxito medibles, plan de descomposición (depth-first vs breadth-first vs straightforward), reglas de fuente/evidencia con stop-conditions, y formato de salida con etiquetado de evidencia. Esto es lo que diferencia los prompts oficiales del cookbook de Anthropic (`research_lead_agent.md`) y las plantillas de OpenAI Deep Research de los prompts genéricos virales.
- Las técnicas con mayor evidencia empírica son: (a) **escalado explícito de esfuerzo** — reglas tipo "consulta simple → 1 agente, 3-10 tool calls; compleja → hasta 10 subagentes con 10-15 calls cada uno"; (b) **quote-first / extraer evidencia antes de sintetizar**; (c) **dar explícitamente la opción de "no sé"**; (d) **heurísticas de calidad de fuente** con whitelist (papers, docs oficiales, fuentes primarias) y blacklist (agregadores, SEO content farms, marketing copy, "experts say" sin nombre); (e) **instrucciones positivas** en lugar de negativas (Pink Elephant Problem). El equipo de Research de Anthropic reporta que su sistema multi-agente con estas reglas superó al single-agent Claude Opus 4 por **90.2 %** en su eval interna (blog del 13 de junio 2025, Hadfield et al.), a un coste de **~15× más tokens** que un chat estándar.
- Para el meta-prompt específico sobre CLAUDE.md: (1) inyectar URLs oficiales obligatorias como whitelist (`code.claude.com/docs/best-practices`, `docs.anthropic.com/en/docs/claude-code/memory`, repo `anthropics/anthropic-cookbook`); (2) imponer schema de salida con etiquetado `[DOC_OFICIAL] | [EVIDENCIA_EMPÍRICA] | [CONSENSO_COMUNIDAD] | [INFERENCIA]`; (3) añadir stop-conditions multi-criterio y cláusula explícita de *absence of evidence*; (4) reducir bullets paralelos y jerarquizar entregables; (5) reformular instrucciones negativas como mandatos positivos; (6) reordenar para colocar las instrucciones críticas al final del prompt.

---

## Key Findings

### Bloque A — Documentación oficial de Anthropic (2025-2026)

1. **XML tags como contenedores semánticos.** Claude fue entrenado para reconocer estructura XML; envolver instrucciones, contexto, ejemplos y entrada en tags (`<instructions>`, `<context>`, `<examples>`, `<input>`, `<thinking>`, `<answer>`) reduce mezcla de roles y mejora parseabilidad. Fuente: `docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags`. **[DOC_OFICIAL]**

2. **Chain-of-thought estructurado, no prescriptivo.** Para modelos con extended/adaptive thinking (Opus 4.5+, Sonnet 4.6+), Anthropic recomienda instrucciones de alto nivel ("piensa profundamente, considera múltiples enfoques, intenta diferentes métodos si el primero falla") en vez de pasos prescriptivos. Para modelos sin thinking activo, usar tags `<thinking>` y `<answer>` explícitos. **[DOC_OFICIAL]**

3. **Quote-first / evidence-first.** Para documentos largos (>20K tokens) o investigación, Anthropic recomienda explícitamente extraer citas textuales **antes** de razonar. El cookbook `anthropics/courses/08_Avoiding_Hallucinations.ipynb` formaliza el patrón: scratchpad con cita relevante + evaluación metacognitiva ("¿esta cita responde la pregunta o le falta detalle?") antes de generar respuesta. **[DOC_OFICIAL]** + **[EVIDENCIA_EMPÍRICA]**

4. **Permiso explícito de "no sé".** Documentado en `docs.claude.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations`: permitir a Claude decir "I don't know" o "I'm not certain" cuando carece de información fiable. La comunidad lo reporta como una de las tres palancas más efectivas para reducir confabulación. **[DOC_OFICIAL]**

5. **Verificación por citación obligatoria.** Patrón: cada claim debe llevar quote de soporte; si Claude no puede encontrar quote, debe retractar el claim. Especialmente potente combinado con "no sé". **[DOC_OFICIAL]**

6. **Restricción de conocimiento externo.** "Explicitly instruct Claude to only use information from provided documents and not its general knowledge." Bloquea el "relleno" desde memoria paramétrica. **[DOC_OFICIAL]**

7. **Define success criteria antes de prompt-engineer.** `docs.anthropic.com/en/docs/test-and-evaluate/develop-tests` exige criterios específicos, medibles, alcanzables, relevantes y temporales (SMART). Sin esto se optimiza a ciegas. **[DOC_OFICIAL]**

8. **Instrucciones positivas > negativas (Pink Elephant Problem).** Anthropic recomienda explícitamente reformular negaciones: en vez de "Do not use markdown" → "Your response should be composed of smoothly flowing prose paragraphs". La evidencia conductual: la negación introduce el concepto que se quería suprimir, manteniéndolo activo en la ventana de atención. DreamHost (2025) reporta que Claude 4.5 con Extended Thinking es **más** sensible a este efecto que modelos previos. **[DOC_OFICIAL]** + **[EVIDENCIA_EMPÍRICA]**

9. **Emphasis markers (`IMPORTANT`, `YOU MUST`) sí mueven la aguja.** Anthropic confirma en su documentación que estos marcadores aumentan la probabilidad de que el modelo trate la regla como restricción dura. Limitarlos a 1-2 por prompt; si todo es importante, nada lo es. **[DOC_OFICIAL]**

### Bloque B — Patrones del cookbook oficial de research multi-agente (Anthropic, 13 jun 2025)

Los prompts publicados en `github.com/anthropics/anthropic-cookbook/blob/main/patterns/agents/prompts/research_lead_agent.md` codifican heurísticas de investigación profesional:

10. **Clasificación obligatoria de tipo de query** *antes* de planificar:
    - *Depth-first*: múltiples perspectivas sobre un mismo tema (ej. "¿Qué causó la crisis financiera de 2008?")
    - *Breadth-first*: sub-preguntas independientes paralelizables (ej. "Compara los sistemas fiscales de los países nórdicos")
    - *Straightforward*: fact-finding directo (ej. "¿Cuál es la población de Tokio?")
    Cada tipo dispara un patrón distinto de delegación. **[DOC_OFICIAL]**

11. **Escalado de esfuerzo explícito.** Verbatim del blog de ingeniería: "Simple fact-finding requires 1 agent with 3-10 tool calls, direct comparisons might need 2-4 subagents with 10-15 calls each, and complex research might use more than 10 subagents." Tope duro: 20 subagentes. **[DOC_OFICIAL]**

12. **Stop-condition explícita.** El lead agent tiene la regla literal: *"when you have reached the point where further research has diminishing returns and you can give a good enough answer to the user, STOP FURTHER RESEARCH and do not create any new subagents."* Los subagentes: *"As soon as you have the necessary information, complete the task rather than wasting time by continuing research unnecessarily."* **[DOC_OFICIAL]**

13. **Reglas de calidad de fuente codificadas en el prompt.** El subagent prompt lista verbatim indicadores de baja calidad: *"news aggregators rather than original sources of the information, false authority, pairing of passive voice with nameless sources, general qualifiers without specifics, unconfirmed reports, marketing language for a product, spin language, speculation, or misleading and cherry-picked data."* Y la regla positiva: *"Prioritize original sources over news aggregators."* **[DOC_OFICIAL]**

14. **Reglas de manejo de conflictos.** Verbatim: *"When encountering conflicting information, prioritize based on recency, consistency with other facts, and use best judgment."* Los subagentes deben *"flag estas issues when returning your report to the lead researcher rather than blindly presenting all results as established facts."* **[DOC_OFICIAL]**

15. **Start wide, then narrow.** "Agents often default to overly long, specific queries that return few results. We counteracted this by prompting agents to start with short, broad queries, evaluate what's available, then progressively narrow focus." Replica cómo trabajan investigadores humanos expertos. **[DOC_OFICIAL]** + **[EVIDENCIA_EMPÍRICA]**

16. **Métricas reportadas por Anthropic.** Del blog `anthropic.com/engineering/multi-agent-research-system`, publicado el 13 de junio 2025 (Hadfield, Zhang, Lien, Scholz, Fox y Ford): *"a multi-agent system with Claude Opus 4 as the lead agent and Claude Sonnet 4 subagents outperformed single-agent Claude Opus 4 by 90.2% on our internal research eval."* Coste: *"multi-agent systems use about 15× more tokens than chats."* Y *"token usage by itself explains 80% of the variance"* en BrowseComp. **[EVIDENCIA_EMPÍRICA]**

17. **Think like your agent.** Consejo nº1 del equipo de Research: simular el prompt con las herramientas reales y ver dónde falla paso a paso. Iterar prompts sin observar trazas es desarrollo a ciegas. **[EVIDENCIA_EMPÍRICA]**

### Bloque C — Patrones de Deep Research (OpenAI, Perplexity) que transfieren a Claude

18. **Front-loaded clarification step.** El cookbook de OpenAI Deep Research recomienda usar un modelo más rápido (4o, Haiku) para clarificar y/o enriquecer el prompt antes de pasárselo al agente, porque "the model expects fully-formed prompts up front and will not ask for additional context". Si el usuario no clarifica, el meta-prompt debe codificar todas las restricciones por adelantado. **[DOC_OFICIAL OpenAI]**

19. **Prioridad explícita de fuentes según dominio.** OpenAI Deep Research cookbook: *"For academic or scientific queries, prefer linking directly to the original paper or official journal publication rather than survey papers or secondary summaries. For product and travel research, prefer linking directly to official or primary websites."* Aplicar la misma lógica a CLAUDE.md: priorizar `docs.claude.com` y `code.claude.com` sobre blogs aggregator. **[DOC_OFICIAL OpenAI]**

20. **Pedir tablas comparativas explícitamente.** Sin instrucción, los agentes raramente las generan. El cookbook de OpenAI lo lista como mandato. Útil para "qué incluir en CLAUDE.md" comparado con anti-patrones. **[DOC_OFICIAL OpenAI]**

21. **"Misinformation by omission" es el modo de fallo dominante.** Simon Willison (25 feb 2025, `simonwillison.net/2025/Feb/25/deep-research-system-card/`): *"the one thing that can't be easily spotted is misinformation by omission: it's very possible for the tool to miss out on crucial details because they didn't show up in the searches that it conducted."* Implicación práctica: el prompt debe forzar al agente a listar **qué buscó y qué NO encontró**, no solo a presentar lo encontrado. **[CONSENSO_COMUNIDAD experto]**

### Bloque D — Evidencia empírica sobre reducción de alucinaciones

22. **Factored verification (George & Stuhlmüller, WIESP/IJCNLP-AACL nov 2023, arXiv 2310.10627).** Al descomponer afirmaciones, verificarlas individualmente contra la fuente y revisar, las alucinaciones por resumen bajaron de **1.55 → 0.95** en Claude 2 (~39 % reducción). El método estableció SotA en HaluEval con **76.2 % accuracy** en detección de alucinaciones. Pre-Claude 4, pero la dirección del efecto está bien establecida. **[EVIDENCIA_EMPÍRICA]**

23. **Trade-off real entre citas y creatividad.** El paper arXiv 2307.02185 reporta que las restricciones de citación reducen la salida creativa. Implicación práctica documentada en r/ClaudeAI: usar modo "research" con citaciones forzadas solo para tareas de investigación, **no como default global**. **[EVIDENCIA_EMPÍRICA]** + **[CONSENSO_COMUNIDAD]**

24. **Sensibilidad a "trust the tool" en modelos nuevos.** DreamHost (2025) reporta que con Extended Thinking habilitado, instrucciones prescriptivas sobre *cómo* pensar empeoran resultados — el modelo razona mejor con prompts simplificados. Pink-elephant amplificado en Sonnet 4.5/Opus 4.5+. **[EVIDENCIA_EMPÍRICA semicontrolada]**

### Bloque E — Anti-patrones documentados

25. **Demasiadas instrucciones paralelas.** HumanLayer (basado en research interna, blog `humanlayer.dev/blog/writing-a-good-claude-md`): *"Frontier thinking LLMs can follow ~150-200 instructions with reasonable consistency. Smaller models can attend to fewer instructions than larger models, and non-thinking models can attend to fewer instructions than thinking models."* Listas de 30+ bullets paralelos diluyen la atención. **[CONSENSO_COMUNIDAD técnico]**

26. **Bullets sin jerarquía.** Cuando todo se presenta como bullets coordinados, el modelo no distingue restricción dura de preferencia. Solución: jerarquía visual (`# CRITICAL` / `## Should` / `### Nice-to-have`) + emphasis markers solo en lo critical. **[CONSENSO_COMUNIDAD]**

27. **Contradicciones acumuladas por edits incrementales.** Digital Applied (2026): *"Each edit is individually reasonable. Together, over six months, they convert a crisp instruction into a paragraph of overlapping, contradictory directives. The result is silent regression."* Diagnóstico operativo: contar requisitos en instrucciones vs. en ejemplos; >3 contradicciones predice regresión. **[CONSENSO_COMUNIDAD técnico]**

28. **CoT para tareas simples.** Añadir "think step by step" a clasificación binaria solo agrega tokens y latencia sin mejorar accuracy. Use CoT solo cuando un humano necesitaría pensar la respuesta. **[EVIDENCIA_EMPÍRICA]**

29. **Instrucciones negativas redundantes (DO NOT … DO NOT … DO NOT …).** Reddit r/ClaudeAI (atestiguado en XDA Developers 2026): "LLMs seem to produce worse output the more DO NOTs are included in the prompt." Mecanismo: ironic process / attention highlighting. **[CONSENSO_COMUNIDAD]**

30. **Ejemplos few-shot que contradicen las instrucciones.** Si el bloque de ejemplos viola una regla del bloque de instrucciones, los ejemplos ganan en el espacio de atención del modelo. Auditar coherencia instrucciones↔ejemplos antes de desplegar. **[EVIDENCIA_EMPÍRICA]**

31. **Auto-generar y olvidarse.** Builder.io y HumanLayer coinciden: `/init` produce un starting point con relleno y obviedades; borrarlo. Cada línea de un prompt largo debe justificar su existencia o consume contexto sin aportar señal. **[CONSENSO_COMUNIDAD]**

### Bloque F — Patrones de power-users y comunidad

32. **"Custom Deep Research" gist de XInTheDark (2025, `gist.github.com/XInTheDark/6fef041cb3edfe054b507813a03cb47d`).** Stop-condition multi-criterio verbatim: *"Only stop researching when ALL of these are met: 1. Minimum source requirement fulfilled 2. All major aspects of the topic thoroughly investigated 3. Conflicting information resolved or acknowledged 4. You have fully understood all information 5. You can answer the query accurately and comprehensively, with high confidence."* Y mínimo de fuentes explícito: *"a minimum of 10 unique, authoritative sources MUST be fetched and cited. Snippets from search results do NOT count as sources."* **[CONSENSO_COMUNIDAD]**

33. **Toggle "research vs. default".** Reddit u/ColdPlankton9273 (atestiguado por XDA Developers 2026): construir un sistema con dos modos — las tres instrucciones anti-alucinación de Anthropic ("permite no-sé + citas + quotes directos") se activan solo en research mode, no como default — por el trade-off creatividad/citas. **[CONSENSO_COMUNIDAD]**

34. **Niveles de thinking jerárquicos.** Doc oficial de Claude Code: `think < think hard < think harder < ultrathink`, asignando ~4K / ~10K / ~32K tokens de presupuesto interno respectivamente. Aplicable cuando se quiere forzar planificación profunda antes del research. Útil en la fase de PLAN. **[DOC_OFICIAL]**

### Bloque G — CLAUDE.md: contexto mínimo necesario (secundario para esta investigación)

CLAUDE.md es un archivo markdown leído al inicio de cada sesión de Claude Code; vive en la raíz del proyecto (también puede vivir en `~/.claude/CLAUDE.md`, `./.claude/CLAUDE.md`, `./CLAUDE.local.md` con precedencia jerárquica). Anthropic recomienda **<200 líneas**, alto contenido de señal, orientado a comandos, convenciones, arquitectura y gotchas — no a documentación abstracta. Cinco secciones de alto impacto convergen en la comunidad (Bijit Ghosh 2026, HumanLayer 2025, Builder.io 2025): (1) comandos build/test/lint, (2) arquitectura/mapa del repo, (3) convenciones de código, (4) gotchas/no-obvios, (5) reglas críticas con `IMPORTANT`/`YOU MUST` (máx ~15). CLAUDE.md es **advisory, no determinístico**: según Builder.io (`builder.io/blog/claude-code-tips-best-practices`), *"CLAUDE.md is advisory. Claude follows it about 80% of the time. Hooks are deterministic, 100%."* Para enforcement obligatorio, use hooks. **[DOC_OFICIAL]** + **[CONSENSO_COMUNIDAD]**

---

## Details — Plantilla recomendada para un prompt de investigación profunda óptimo

```
<role>
Eres un investigador senior especializado en [DOMINIO]. 
Tu output será usado para [DECISIÓN DOWNSTREAM CONCRETA].
</role>

<research_objective>
Pregunta principal: [...]
Sub-preguntas obligatorias (en orden de prioridad):
1. ...
2. ...
3. ...
Tipo de query: [depth-first | breadth-first | straightforward]
</research_objective>

<success_criteria>
- Cobertura: cada sub-pregunta debe tener ≥3 fuentes independientes citadas.
- Recencia: ≥70% de fuentes con fecha 2025-2026; material anterior solo si la técnica/dato sigue vigente, marcado.
- Verificabilidad: cada afirmación cuantitativa debe llevar URL + cita textual ≤30 palabras.
- Diversidad: no más del 40% de fuentes de un mismo dominio.
- Densidad: sin relleno; cada párrafo debe aportar dato o interpretación.
</success_criteria>

<source_quality_rules>
PRIORIZAR (whitelist):
- Documentación oficial: docs.claude.com, anthropic.com/engineering, anthropic-cookbook (GitHub)
- Papers peer-reviewed (arXiv ≥2024 preferido)
- Blogs técnicos con autor identificable (Simon Willison, latent.space, Hamel Husain, ...)
- Repos GitHub con >100 estrellas o commits 2025-2026

PENALIZAR / DESCARTAR si predominan:
- News aggregators, content farms SEO-optimizados
- Pasiva + fuente innominada ("se dice que...", "experts say")
- Marketing copy, spin language, qualifiers genéricos
- Cherry-picked data sin contexto
- Listicles sin autoría
</source_quality_rules>

<evidence_labeling>
Cada hallazgo debe etiquetarse:
- [DOC_OFICIAL] = doc primaria del vendor
- [EVIDENCIA_EMPÍRICA] = estudio/medición con número
- [CONSENSO_COMUNIDAD] = patrón repetido en ≥3 fuentes independientes
- [INFERENCIA] = deducción propia, marcada como tal

Conflictos entre fuentes confiables: mostrar AMBAS posturas con cita.
</evidence_labeling>

<process>
Fase 1 — PLAN (ultrathink): lista sub-preguntas, criterios de filtrado, tipo de query, 3 estrategias de búsqueda candidatas, elige la mejor.
Fase 2 — RESEARCH: empieza wide (queries cortas, broad), luego narrow. Ejecuta búsquedas en paralelo donde sea posible. Para cada fuente: extrae quote verbatim ≤30 palabras + URL + fecha ANTES de sintetizar.
Fase 3 — ANALYSIS: cruza quotes, identifica contradicciones, etiqueta evidencia.
Fase 4 — SYNTHESIS: redacta output siguiendo <output_format>. Antes de finalizar, para cada afirmación: verifica que tienes quote, o márcala [INFERENCIA].
</process>

<stop_conditions>
DETÉN la investigación cuando TODAS estas se cumplan:
1. Cada sub-pregunta tiene ≥3 fuentes citadas.
2. Conflictos identificados están resueltos o explícitamente marcados como abiertos.
3. Diminishing returns: las últimas 2 búsquedas no aportaron información nueva.

ABORTA y reporta si:
- Menos de N fuentes whitelist disponibles para una sub-pregunta crítica → reporta gap, no rellenes.
- El tópico requiere datos no públicos → declarar explícitamente.
</stop_conditions>

<absence_of_evidence>
Si una sub-pregunta no tiene evidencia suficiente, reporta literalmente:
"AUSENCIA_DE_EVIDENCIA: [sub-pregunta]. Búsquedas ejecutadas: [...]. No fabricar."
Es PREFERIBLE reportar gaps que rellenar con inferencias no marcadas.
</absence_of_evidence>

<output_format>
1. TL;DR (3-5 bullets, decision-ready)
2. Hallazgos clave (lista priorizada): nombre · mecanismo · ejemplo · fuente+URL · etiqueta
3. Tabla comparativa cuando aplique
4. Anti-patrones con justificación técnica
5. Recomendaciones accionables escalonadas (Now / Next / Later)
6. Caveats y limitaciones (incluyendo qué no se pudo verificar)
</output_format>

<anti_patterns_to_avoid>
- Texto genérico cuando falta evidencia → declarar AUSENCIA_DE_EVIDENCIA.
- Frases tipo "se dice que", "muchos expertos", "según se ha visto" sin nombre y URL.
- Mezclar inferencias propias con citas sin etiquetarlas.
- Bullets paralelos sin jerarquía.
- Afirmación cuantitativa sin cita verbatim → eliminarla o marcarla [INFERENCIA].
</anti_patterns_to_avoid>
```

---

## Recommendations — Cómo refinar el meta-prompt sobre CLAUDE.md

### Now (cambios de alta prioridad)
1. **Inyectar whitelist de URLs específicas** en el prompt: `https://code.claude.com/docs/en/best-practices`, `https://docs.anthropic.com/en/docs/claude-code/memory`, `https://www.anthropic.com/engineering/multi-agent-research-system`, repo `anthropics/anthropic-cookbook`, gist `XInTheDark/6fef041cb3edfe054b507813a03cb47d`. Sin esto el modelo gravita a blogs aggregator SEO-optimizados.
2. **Añadir bloque `<stop_conditions>` multi-criterio** (patrón de XInTheDark): mínimo de fuentes por sub-pregunta, criterio de diminishing returns, abort criterion explícito.
3. **Añadir bloque `<absence_of_evidence>`** con instrucción literal de reportar gaps en vez de inventar.
4. **Schema de output estricto** con etiquetado `[DOC_OFICIAL] | [EVIDENCIA_EMPÍRICA] | [CONSENSO_COMUNIDAD] | [INFERENCIA]` por hallazgo (ya está en tu prompt pero conviene consolidarlo en un solo `<evidence_labeling>` y no repartirlo).
5. **Fase de PLAN explícita con `ultrathink`** antes de empezar a buscar.
6. **Mandato explícito de tabla comparativa** (qué incluir en CLAUDE.md vs qué NO incluir vs qué mover a skills/hooks).

### Next (refactor estructural)
1. **Reagrupar las 5 áreas** en *Critical* (ingeniería del prompt de investigación) / *Should* (anti-patrones, patrones comunidad) / *Nice-to-have* (contexto sobre CLAUDE.md mismo). Hoy se presentan como bullets paralelos sin jerarquía, lo que dilata la atención del modelo.
2. **Consolidar instrucciones redundantes**. "Etiquetar cada hallazgo" aparece en dos sitios; consolidarlo en `<evidence_labeling>` único.
3. **Reformular las instrucciones negativas dispersas** ("no inventar URLs", "no fabricar", "sin relleno") en un único bloque `<anti_patterns>` con framing positivo donde sea posible: *"Cada URL debe haber aparecido en un resultado de búsqueda real"* (positivo) en lugar de *"no inventes URLs"* (negativo).
4. **Reordenar para colocar instrucciones críticas al final**. Anthropic recomienda en `extended-thinking-tips` colocar instrucciones específicas tras el contexto. (Nota: el cookbook de OpenAI también lo sugiere para inputs largos, aunque no aporta cifra específica de mejora — esto es **[CONSENSO_COMUNIDAD]**, no medición publicada.)
5. **Mover el contexto sobre CLAUDE.md (área 5 actual) a un `<background_context>` minimal** — es contexto, no objetivo de investigación.

### Later (madurez)
1. **Construir eval set** con 5-10 prompts de investigación de prueba y rúbrica LLM-as-judge (criterios: factual accuracy, citation accuracy, completeness, source quality, tool efficiency — los cinco que usa Anthropic en su sistema de Research).
2. **Iterar el meta-prompt en simulación**, observando trazas (think-like-your-agent).
3. **Versionar el meta-prompt** y hacer canary deployment (<10% del tráfico) con métricas de regresión antes de promover.

### Benchmarks que cambiarían las recomendaciones
- Si Anthropic publica nuevas cifras de su Research eval que contradigan la heurística "1 agente / 3-10 tool calls", recalibrar el escalado de esfuerzo.
- Si Claude 4.7+ documenta que los XML tags ya no son necesarios (por mejor parsing de markdown), simplificar la plantilla.
- Si el toggle research/default deja de ser necesario (citation constraint deja de penalizar creatividad), unificar como default.

---

## Caveats

- Las cifras del 90.2 % de mejora y 15× tokens son de Anthropic sobre su eval interna (blog del 13 jun 2025); generalización fuera de research multi-agente no está validada.
- La cifra 1.55 → 0.95 alucinaciones por resumen (Elicit / George & Stuhlmüller, WIESP nov 2023, arXiv 2310.10627) es sobre Claude 2; magnitud puede diferir en Claude 4.x, pero la dirección está bien establecida.
- Las tres "system prompts mágicos" (permite no-sé + citas + quotes directos) viralizados en r/ClaudeAI funcionan en research pero degradan creatividad — usar como toggle, no default global. Trade-off documentado en arXiv 2307.02185.
- "Ultrathink" tiene presupuestos de tokens documentados (~32K) pero el beneficio real depende del task: para tareas simples es desperdicio.
- Los modos Research / Deep Research de Claude/ChatGPT/Perplexity tienen system prompts no públicos; las recomendaciones se basan en docs públicos + cookbooks + ingeniería inversa parcial. Tratar como buenas prácticas, no como leyes.
- CLAUDE.md es advisory (~80 % adherencia según Builder.io). Para enforcement determinístico use hooks, no instrucciones en el .md.
- No pude verificar directamente el hilo Reddit original de u/ColdPlankton9273 sobre las tres palancas anti-alucinación; la atribución viene de XDA Developers y Tamiltech (reporting secundario). El **patrón** está respaldado por los docs oficiales de Anthropic; la **anécdota Reddit específica** es secondhand.
- La afirmación de que "instrucciones específicas al final mejoran la respuesta hasta ~30 %" no aparece verificada en el cookbook de OpenAI Deep Research; tratarla como **[CONSENSO_COMUNIDAD]** y no como medición publicada.

