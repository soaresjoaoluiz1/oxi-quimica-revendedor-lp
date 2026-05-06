# Oxiquímica Varginha — Landing Page de Captura de Revendedores

> 🌐 **Live:** https://revendedor.oxiquimicavarginha.com.br
> 🎯 **Objetivo:** captar revendedores qualificados para a Oxiquímica via tráfego pago (Meta Ads + Google Ads), filtrando intenção real e capacidade de investimento antes de gerar o lead no CRM.

---

## ✨ O que essa LP entrega

Uma landing page de **conversão dinâmica com qualificação de leads na entrada** — lead que cai aqui não vira CRM antes de provar:

1. **Que tem real interesse** em ser revendedor (não curiosidade)
2. **Que pode investir** o pedido mínimo de R$ 1.000

Resultado: pipeline de comercial limpo, taxa de fechamento mais alta, custo por lead qualificado mais baixo.

---

## 🧭 Fluxo do usuário (4 etapas em wizard)

```
┌──────────────────────────────────────────────────────────────────┐
│  HERO + Provas sociais + Benefícios + Produtos + Depoimentos     │
│                            ↓                                       │
│                     Card "Quero ser revendedor"                    │
└──────────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Etapa 1/4      │  │  Etapa 2/4      │  │  Etapa 3/4      │
│  Dados de       │→ │  Sua região?    │→ │  Tem CNPJ?      │
│  contato        │  │  (input texto)  │  │  (sim/não)      │
│                 │  │                 │  │  Não desqualifi │
└─────────────────┘  └─────────────────┘  └─────────────────┘
                                                    ↓
                                    ┌─────────────────────────┐
                                    │  Etapa 4/4              │
                                    │  Investe R$ 1.000?      │
                                    └─────────────────────────┘
                                       ↓               ↓
                             ┌──────SIM──────┐   ┌──NÃO───────────┐
                             │  envia lead    │   │  pedido-minimo │
                             │  → Pixel Lead  │   │  → fbq Custom  │
                             │  → GA4 Lead    │   │  LeadDisquali- │
                             │  → CRM webhook │   │  fied (excluir │
                             │  → obrigado    │   │  do remarketing│
                             │                │   │  no Meta Ads)  │
                             └────────────────┘   └────────────────┘
```

### Por que essa ordem?

- **Captura o nome/telefone primeiro** quando o engajamento ainda está alto.
- **Pergunta CNPJ** mas não desqualifica — apenas captura pra qualificação interna (Oxiquímica atende PF e PJ).
- **Investimento é o gate final.** Quem responde "não" não vira lead no CRM e cai numa página de "ainda não conseguimos atender" educada.
- **Lead de baixa qualidade vira evento custom no Meta Pixel** → audiência negativa que pode ser excluída de campanhas (economiza orçamento de mídia).

---

## 🎯 Tracking & Atribuição

### Meta Pixel (`387506053330342`)
- `PageView` em todas páginas
- `Lead` no envio do form (deduplicado entre browser e CAPI futura via `eventID`)
- **Custom event `LeadDisqualified`** na `pedido-minimo.html` com parâmetros `motivo: investimento_minimo` e `valor_minimo: 1000` — pra criar audiência personalizada e **excluir** dos conjuntos de anúncios

### Google Analytics 4 (`G-BTQ4TV7LH0`)
- `generate_lead` no submit do form e na `obrigado.html`

### UTMs + atribuição completa
A primeira visita salva em `sessionStorage`:
- `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`
- `gclid` (Google Ads click ID)
- `fbclid` (Meta Ads click ID)
- `referrer`, `first_landing` URL completa

Tudo enviado junto no payload pro Google Apps Script → Google Sheet → CRM. Você sabe exatamente qual campanha e anúncio trouxe cada lead.

### Pronto pra CAPI (Conversions API do Meta)
O payload já inclui:
- `event_id` (idêntico ao do browser, pra deduplicar)
- `_fbp` e `_fbc` (cookies do Meta)
- `user_agent`
- `page_url`

Quando ativar o CAPI no Apps Script, basta forwardar pra `graph.facebook.com/{PIXEL_ID}/events`. Recupera 30–50% dos eventos perdidos por adblocker/ITP.

---

## 🏗️ Páginas

| Arquivo | Função | Observações |
|---|---|---|
| `index.html` | Landing principal | Single-page com 13 seções: hero, trust bar, benefícios, banner produtos, grid de produtos, segmentos, como funciona, números, depoimentos, FAQ (9 perguntas com accordion), formulário em wizard 4 etapas, CTA final, footer |
| `obrigado.html` | Confirmação pós-envio | Dispara `Lead` (Pixel) e `generate_lead` (GA4). Ícone verde de sucesso, lista os próximos passos, link pro WhatsApp do consultor. `noindex` |
| `pedido-minimo.html` | "Ainda não é o momento" | Dispara `LeadDisqualified` no Pixel. Tom amigável: explica os 3 motivos do mínimo de R$ 1.000 (margem, frete grátis, variedade), dá uma dica de juntar recursos, único CTA "Voltar à página inicial". `noindex` |

---

## 🎨 Identidade visual

- **Cores:** navy `#0f1f4b` + blue `#2563eb` + green `#059669` + gold `#b8972a`
- **Tipografia:** Inter (Google Fonts) com pesos 400–900
- **Componentes:** sem framework JS, CSS puro com tokens em `:root` (radius, shadows, eases). ~550 linhas de CSS.
- **Mobile-first:** breakpoints em 640px e 1024px
- **Acessibilidade:** ARIA roles, `prefers-reduced-motion`, foco visível, touch targets ≥ 48px

---

## 🛠️ Stack & arquitetura

**Zero dependências de build.** É HTML/CSS/JS puro, hospedado estático em VPS Apache/cPanel.

- **HTML:** semântico, com `lang="pt-BR"`, `meta description`, Open Graph parcial
- **CSS:** inline `<style>` no head pra zero round-trip extra (LP rápida no first-paint)
- **JS:** vanilla — wizard de form (4 etapas), máscara de telefone, scroll-reveal via IntersectionObserver, FAQ accordion, captura de UTMs, fetch pro Apps Script
- **Backend do form:** Google Apps Script com modo `no-cors` (write-only, sem CORS issues)
- **Persistência:** Google Sheet → integrável com CRM, ZeroPaper, RD, qualquer ferramenta

### Estrutura

```
oxiquimica-lp/
├── index.html               # ★ Landing principal (1.4k linhas)
├── obrigado.html            # Pós-envio (Pixel/GA4 conversion)
├── pedido-minimo.html       # Desqualificação (Pixel custom event)
├── logo.png                 # Logo Oxiquímica
├── revendedor-oxi-quimica.png   # Hero
├── produtos-oxi-quimica.png # Banner de produtos
├── produtos-oxi.png         # Imagem de produtos individuais
├── oxi-quimica-bg.png       # Background hero (~7.7 MB — otimizar!)
├── oxi-quimica-bg-mobile.png # Background mobile (~5.5 MB — otimizar!)
└── icons/                   # 8 ícones PNG dos segmentos
    ├── agriculture.png
    ├── car.png
    ├── corporation.png
    ├── fast-food.png
    ├── hospital.png
    ├── house.png
    ├── raw-material.png
    └── workers.png
```

---

## 🚀 Deploy

Estática pura — sobe os arquivos por FTP/cPanel direto na pasta do subdomínio.

### Como atualizar
1. Edita arquivo local
2. Sobe via cPanel File Manager (apenas o arquivo modificado) ou FTP

### Hospedagem atual
- **Servidor:** mesma VPS dos outros sistemas Oxiquímica
- **Domínio:** `revendedor.oxiquimicavarginha.com.br` (subdomínio)
- **Web server:** Apache 2.4 + cPanel
- **SSL:** Let's Encrypt via cPanel AutoSSL

---

## 📊 Métricas de sucesso

Acompanhar no GA4 + Meta Events Manager:

| Métrica | Onde ver |
|---|---|
| Visitantes únicos | GA4 → Relatórios → Aquisição |
| Taxa de conversão (visitante → lead qualificado) | GA4 evento `generate_lead` / sessões |
| Lead disqualified rate | Meta Events Manager → eventos custom |
| Custo por lead qualificado | Meta Ads + Google Ads → divide gasto pelos `Lead` |
| Atribuição por canal | Google Sheet (UTMs) ou GA4 → Aquisição → Tráfego |

---

## 🛡️ Próximos passos

- [ ] Migração pra GTM (Google Tag Manager) pra centralizar todas as tags
- [ ] Ativar Meta CAPI server-side via Apps Script (token + endpoint)
- [ ] Adicionar Google Ads Conversion (`AW-XXX/label`) quando rodar campanhas no Google
- [ ] Otimizar imagens grandes (`oxi-quimica-bg.png` 7.7 MB, `oxi-quimica-bg-mobile.png` 5.5 MB) → WebP + dimensões corretas
- [ ] Open Graph tags completas pra compartilhamento social
- [ ] JSON-LD schema (Organization + FAQPage)

---

## 👥 Créditos

Desenvolvido por **[Dros Agência](https://drosagencia.com.br)** — agência de marketing digital orientada por dados. Estratégia, copy, design, dev e tracking integrados.

📞 Contato Oxiquímica: (35) 99714-8855 · contato@oxiquimicavarginha.com.br
📍 Av. Dr. Módena, 723 — Fátima, Varginha/MG
