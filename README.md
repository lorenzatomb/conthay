# Conthay — MVP

Apoio de **registro** de refeições e macronutrientes para pessoas com diabetes.
Mostra carboidratos totais (métrica principal), proteínas, gorduras, fibras, calorias,
escolhas de carboidrato, **fonte** e **nível de confiança** de cada alimento.

> ⚠️ **Não** diagnostica, **não** prescreve, **não** calcula dose de insulina e **não**
> substitui orientação médica/nutricional. É uma ferramenta de registro.

---

## Como rodar / deploy (fluxo GitHub → Vercel)

É **um único arquivo** (`index.html`), 100% client-side. Não precisa build.

1. Crie um repositório no GitHub (ou use um existente).
2. Suba `index.html` na raiz (e, opcionalmente, este `README.md`).
3. No Vercel: **Add New → Project → importe o repo → Deploy**.
   - Framework Preset: **Other** (não é Next.js).
   - Root Directory: `/`. Build Command: vazio. Output: vazio.
4. Pronto. Sugestão de URL: `conthay.vercel.app`.

Rodar local: abra o `index.html` no navegador (ou `python3 -m http.server` e acesse `localhost:8000`).
No celular, dá pra "Adicionar à tela inicial" (PWA básico, standalone).

> Dica do teu fluxo: como é arquivo único, dá pra editar direto pela interface web do GitHub
> e o Vercel redeploya sozinho. Sem dor de extensão de arquivo se você criar como `index.html`.

---

## Variáveis de ambiente

Nenhuma é necessária para o MVP (tudo local, sem API paga).

Preparado para fase 2 (adaptadores isolados, com fallback manual):
- `USDA FoodData Central` — leria de `window.__USDA_KEY` (ou env no backend futuro).
- `TBCA/TACO` — importador CSV/JSON (estrutura pronta, sem dados embutidos).
- `Open Food Facts` (código de barras) — adaptador stub, marcado como confiança média/baixa.

---

## O que está implementado (MVP)

- **Onboarding** com consentimento + perfil (tipo de diabetes, insulina, unidade de glicemia, escolha de carbo).
- **Dashboard**: carbo total como herói, proteína/gordura/calorias, escolhas, refeições e glicemias do dia, recentes e favoritos.
- **Adicionar refeição**: busca, "digitar o que comi" (parser de linguagem natural por regex, com confirmação), entrada manual; folha de porção com prévia ao vivo de macros.
- **Detalhe da refeição**: itens, totais, fonte/confiança, registrar glicemia antes/2h depois, observações.
- **Histórico**: filtro 7/14/30/90d, gráfico de carbo/dia, refeições de maior carga, glicemias.
- **Relatórios**: resumo por período + exportação **CSV** e **JSON** (PDF via "Imprimir → Salvar como PDF").
- **Banco de alimentos**: cadastro/edição de alimento próprio, fonte e confiança; sementes de demonstração claramente marcadas.
- **Ajustes**: perfil, metas, unidade, escolha de carbo, carbo líquido (opcional), exportar tudo, apagar tudo.
- **Persistência local** (localStorage) atrás de uma camada trocável (`Store`).
- **Funções puras de cálculo** testadas (macros, totais, escolhas, net carbs, conversão de porção, parser NL).

### Regras de segurança clínica (embutidas)
- `ENABLE_BOLUS_CALCULATOR = false` (não calcula dose).
- Aviso de razão insulina/carbo prescrita quando o perfil usa insulina.
- Dado estimado (demonstração/foto/voz/código/parser) **exige confirmação** antes de salvar.
- Glicemia muito baixa/alta → mensagem segura, **sem conduta** ("procure atendimento se grave").
- Sem linguagem de diagnóstico, sem alimento "proibido/liberado", sem promessa de controle.
- Disclaimer persistente: *"Use este aplicativo como apoio de registro e converse com sua equipe de saúde para decisões de tratamento."*

### Privacidade
- Local-first, sem conta. Nada é enviado a terceiros.
- Glicemia, nome, observações e refeições **não** são logados no console.
- Exportar e apagar todos os dados disponíveis nos Ajustes.

---

## Limitações conhecidas

- **Sementes de alimentos são valores aproximados para demonstração** — confira na TACO/TBCA antes de uso real.
- Parser de linguagem natural é por regex (sem LLM): bom para frases simples, sempre pede confirmação.
- `localStorage` (não IndexedDB) no MVP — suficiente para o volume atual; ver "próximos passos".
- PWA básico (manifest inline, sem service worker) — instala e abre standalone, mas o offline depende do cache do navegador.
- Sem scanner de código de barras, OCR de rótulo, sincronização ou multiusuário (fase 2).

---

## Próximos passos (fase 2)

- Migrar `Store` de `localStorage` → **IndexedDB** e depois **Supabase/Postgres** (camada já isolada).
- Service worker dedicado para offline completo + ícones do PWA.
- Adaptadores reais: USDA, TBCA/TACO (CSV/JSON), Open Food Facts (código de barras), OCR de rótulo.
- Exportação PDF dedicada para o profissional de saúde.
- Análise de padrões refeição × glicemia; metas com progresso visual; backup criptografado.
- Se um dia houver calculadora de bolus: exige validação clínica, revisão regulatória, testes, auditoria e configuração por profissional habilitado — fora do escopo deste MVP.

---

## Testes

As funções puras de cálculo foram validadas (macros por item, totais ponderados, escolhas de
carbo, net carbs, conversão de porção). Como o deploy é single-file/sem build, não há suíte
Vitest no repo; para evoluir, basta extrair `window.Conthay` para um módulo e plugar Vitest.
