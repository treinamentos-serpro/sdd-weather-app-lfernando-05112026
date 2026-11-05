# Discovery — Weather App

Análise inicial do briefing. Os requisitos explícitos estão separados das
recomendações e suposições, que precisam de validação antes da especificação.

## Contexto

A empresa solicita uma aplicação de previsão do tempo que permita buscar cidades,
consultar o clima atual e a previsão de cinco dias, alternar entre Celsius e
Fahrenheit e utilizar essas capacidades em dispositivos móveis.

O objetivo de negócio proposto é facilitar decisões cotidianas com informações
meteorológicas compreensíveis para perfis de deslocamento urbano, viagens e
atividades ao ar livre (detalhados na seção **Personas**).

O projeto adota Spec-Driven Development: este discovery será entrada para a
especificação, seguida do plano técnico, das tarefas, da implementação, dos testes
e da revisão. Não constitui autorização para iniciar código de feature.

As orientações do repositório já estabelecem uma aplicação web com React,
TypeScript, Vite e Tailwind CSS, além da Open-Meteo para geocoding e forecast, sem
API key. Essas são restrições do projeto, não exigências extraídas do briefing.

## Personas

| Persona | Perfil e Objetivo Principal | Contexto de Uso | Métrica de Sucesso (visão do usuário) |
| --- | --- | --- | --- |
| **Mariana (29) — Rotina e deslocamento urbano** | Sair de casa sabendo temperatura atual, sensação térmica e se vai chover hoje para escolher roupa e guarda-chuva. | **Mobile (90%)** — consulta rápida com uma mão em conexão móvel, beneficiando-se da localização inicial do navegador ou da última cidade salva (`DEC-04`, `DEC-08`). | Abrir o app e decidir sobre roupa/guarda-chuva em **menos de 10 segundos** sem precisar digitar quando estiver na própria cidade. |
| **Carlos (41) — Planejador de viagens e agenda semanal** | Pesquisar cidades de destino (incluindo homônimas em estados/países diferentes) e analisar os **5 dias (Hoje + 4)** para planejar deslocamentos e reuniões. | **Híbrido (Desktop + Mobile)** — usa busca com `Cidade - Estado - País` (`DEC-06`, `DEC-09`) e alterna unidades (`°C/°F`, `DEC-03`) e idiomas (`pt-BR`/`es`/`en`, `DEC-05`). | Encontrar a cidade/estado exata na primeira tentativa e comparar mínima, máxima e probabilidade de chuva dos 5 dias sem ambiguidade. |
| **Beatriz (34) — Atividades ao ar livre** | Avaliar indicadores detalhados do clima atual (**vento em km/h, umidade, precipitação em mm e sensação térmica**, `DEC-07`) e probabilidade de chuva nos próximos dias para confirmar treinos externos. | **Mobile em ambiente externo** — exige alto contraste no tema *dark glassmorphism* e alvos de toque confortáveis (`44×44px`). | Visualizar vento, chuva e sensação térmica em uma única tela legível no celular, evitando cancelamentos de última hora. |

## Requisitos Funcionais

### Confirmados pelo briefing

| ID | Requisito | Pontos a esclarecer |
| --- | --- | --- |
| RF-01 | Permitir buscar cidades. | Abrangência geográfica, forma de busca e identificação de cidades homônimas. |
| RF-02 | Exibir o clima atual da cidade escolhida. | Variáveis exibidas, significado de “atual” e frequência de atualização. |
| RF-03 | Exibir a previsão de cinco dias da cidade escolhida. | Inclusão ou não de hoje, fuso horário e dados apresentados por dia. |
| RF-04 | Permitir alternar a exibição das temperaturas entre Celsius e Fahrenheit. | Unidade inicial, precisão, arredondamento e persistência da preferência. |

O uso em dispositivos móveis também é explícito no briefing e está detalhado
como requisito de qualidade em RNF-01.

### Derivados das convenções do projeto

| ID | Requisito | Comportamento a especificar |
| --- | --- | --- |
| RF-05 | Informar os estados de carregamento, erro e ausência de resultados. | Distinguir busca sem resultados, dados meteorológicos indisponíveis e falha de conexão; definir a ação de recuperação. |

### Recomendações a validar

- Permitir distinguir e selecionar cidades homônimas com região e país.
- Indicar a referência temporal dos dados e sinalizar informações desatualizadas.
- Oferecer uma ação explícita para tentar novamente após uma falha recuperável.

Essas recomendações não ampliam automaticamente o escopo aprovado.

## Requisitos Não-Funcionais

| ID | Dimensão | Requisito e status |
| --- | --- | --- |
| RNF-01 | Responsividade | Confirmado: busca, clima atual, previsão e troca de unidade devem ser utilizáveis em dispositivos móveis. Definir larguras mínimas, orientações e dispositivos de aceite. |
| RNF-02 | Acessibilidade | Convenção do projeto: usar controles semânticos, labels e suporte a teclado. Recomenda-se validar contraste, foco visível e leitores de tela, adotando WCAG 2.2 AA como alvo a aprovar. |
| RNF-03 | Desempenho | Proposto: manter busca e consulta responsivas em conexões móveis. Definir limites mensuráveis de tempo de resposta, perfil de rede e condições de medição. |
| RNF-04 | Disponibilidade e resiliência | Proposto: tornar falhas da API e da rede compreensíveis, sem apresentar dados ausentes como sucesso. Definir timeout, política de novas tentativas e eventual uso de dados em cache. |
| RNF-05 | Consistência dos dados | Proposto: manter cidade, datas e unidades coerentes em toda a interface; mudanças rápidas de busca não devem misturar resultados de cidades diferentes. Definir precisão e tolerância ao atraso dos dados. |
| RNF-06 | Compatibilidade | Proposto: funcionar nos navegadores móveis e desktop acordados. Definir navegadores e versões suportados antes dos testes de aceite. |
| RNF-07 | Privacidade | Proposto: minimizar dados coletados e esclarecer o envio de consultas a serviços externos. Geolocalização, histórico e analytics não estão autorizados pelo briefing. |
| RNF-08 | Manutenibilidade e testabilidade | Convenção do projeto: TypeScript strict, acesso a dados isolado em serviços e funções puras e testáveis. Validar critérios de aceite com testes unitários e E2E. |

As metas quantitativas ainda precisam ser acordadas; não há evidência suficiente
para prometer desempenho ou disponibilidade específicos.

## Riscos

Mapa consolidado de riscos técnicos e de produto, atualizado com base nas decisões do discovery (`DEC-01` a `DEC-08`):

| # | Categoria | Risco | Probabilidade | Impacto | Estratégia de Mitigação |
| --- | --- | --- | --- | --- | --- |
| **R-01** | Técnico / Permissão | **Indisponibilidade ou recusa da localização do navegador** (permissão negada, timeout ou navegador sem suporte). | **Alta** | **Médio** — não é possível carregar o clima local automaticamente na primeira abertura. | Se houver última cidade salva em `localStorage` (`DEC-04`), carregá-la; caso contrário, tentar `navigator.geolocation` com timeout curto e, se indisponível/negado, direcionar o foco visual e de teclado para o textbox de busca (`DEC-08`). |
| **R-02** | Técnico / API | **Indisponibilidade, lentidão ou *rate limit* da API externa (Open-Meteo)** em conexões móveis instáveis. | **Média** | **Alto** — o usuário fica sem dados meteorológicos ou com a interface presa em carregamento. | Isolar chamadas em `src/services/`, aplicar `debounce` (300ms) na busca ao digitar, cancelar requisições pendentes com `AbortController`, definir timeout explícito e exibir erro amigável com botão **"Tentar novamente"**. |
| **R-03** | Produto / Dados | **Seleção de cidade homônima incorreta** (ex.: múltiplas cidades chamadas "Santa Maria" ou "São José"). | **Alta** | **Alto** — exibição da previsão para outro local e perda de confiança no produto. | Exibir sempre **`Nome da Cidade - Estado - País`** nas sugestões do textbox de busca e no cabeçalho do clima atual (`DEC-06`, `DEC-09`). |
| **R-04** | Técnico / Estado | **Condição de corrida (*race condition*) ao digitar no textbox ou trocar cidades rapidamente** enquanto uma requisição anterior ainda está em andamento. | **Média** | **Alto** — a tela pode exibir sugestões ou dados meteorológicos de uma consulta anterior. | Cancelar requisições anteriores via `AbortController` e descartar respostas obsoletas antes de atualizar o estado no React. |
| **R-05** | Produto / Data-Hora | **Divergência de fuso horário nos "5 dias (Hoje + 4)"** ao consultar cidades em outros fusos próximos à meia-noite. | **Média** | **Alto** — "Hoje" e os dias da semana ficam deslocados em 1 dia em relação ao horário local da cidade consultada. | Consultar a Open-Meteo com `timezone=auto` e interpretar as datas diárias (`YYYY-MM-DD`) sem deslocamento indevido pelo fuso do navegador (`DEC-02`). |
| **R-06** | Produto / UX Mobile | **Sobrecarga visual em telas pequenas (320px–375px)** com pacote completo de clima atual + seletor de idioma (`pt-BR`/`es`/`en`) + alternância `°C`/`°F`. | **Média** | **Médio** — quebra de layout, truncamento de rótulos traduzidos ou alvos de toque pequenos. | Projetar layout *mobile-first* responsivo em Tailwind, manter alvos de toque mínimos de `44x44px` e validar viewports móveis nos testes E2E com Playwright. |
| **R-07** | Produto / i18n | **Inconsistência de idioma entre interface, busca e dados meteorológicos** ao alternar entre `pt-BR`, `es` e `en`. | **Média** | **Médio** — parte da tela traduzida e parte em outro idioma (descrições WMO ou dias da semana). | Centralizar dicionário i18n (`pt-BR`, `es`, `en`) para UI e códigos WMO, formatar datas com `Intl.DateTimeFormat` usando o locale ativo e enviar `language` na busca da Open-Meteo (`DEC-05`). |
| **R-08** | Técnico / Regra de Negócio | **Inconsistência na conversão `°C ↔ °F` e no arredondamento (máx. 1 casa decimal)** entre clima atual e previsão de 5 dias. | **Baixa** | **Médio** — diferença visual de temperatura para o mesmo dia entre componentes. | Converter no cliente a partir dos valores em Celsius usando função pura única (`máx. 1 casa decimal`) e cobrir casos-limite com testes unitários no Vitest (`DEC-03`). |
| **R-09** | Produto / Acessibilidade | **Baixo contraste no tema *dark glassmorphism*** ou falta de suporte completo a teclado nos controles de busca, unidade e idioma. | **Média** | **Médio** — dificuldade de leitura em ambientes externos e falha de acessibilidade. | Garantir contraste mínimo WCAG AA (4.5:1), foco visível (`focus-visible`), roles/labels semânticos e suporte completo a teclado (`Tab`, `Enter`, `Escape`, setas nas sugestões). |

## Perguntas em Aberto

| ID | Pergunta | Impacto de seguir sem resposta |
| --- | --- | --- |
| PA-01 | Quem é o público prioritário, qual problema cotidiano queremos resolver e como medir sucesso? | Dificuldade de priorizar informações e avaliar valor entregue. |
| PA-02 | A busca será global ou restrita a determinados países? Como tratar acentos, nomes alternativos e homônimos? | Resultados inesperados e seleção incorreta de cidades. |
| PA-03 | A busca será submetida explicitamente ou terá sugestões durante a digitação? | Afeta interação, acessibilidade e volume de chamadas à API. |
| PA-04 | Quais dados compõem o clima atual: temperatura, condição, sensação térmica, umidade, vento ou precipitação? | Escopo e contratos de dados permanecem indefinidos. |
| PA-05 | “Cinco dias” significa hoje mais quatro dias ou os cinco dias seguintes? Qual fuso determina as datas? | Implementação e critérios de aceite podem divergir. |
| PA-06 | Quais dados devem aparecer por dia: mínima, máxima, condição e probabilidade de chuva? | Não é possível fechar o conteúdo da previsão. |
| PA-07 | Qual é a unidade padrão? Quantas casas decimais usar? A preferência deve sobreviver ao recarregamento? | Retrabalho na apresentação e no armazenamento local. |
| PA-08 | Qual atraso dos dados é aceitável e quando atualizar: ao abrir, por ação manual ou periodicamente? | Possibilidade de exibir dados obsoletos ou gerar chamadas excessivas. |
| PA-09 | Qual será o idioma da interface e a convenção de datas? | Impacta textos, formatação e testes; pt-BR da documentação não decide o idioma da UI. |
| PA-10 | Quais navegadores, tamanhos de tela e nível de acessibilidade serão critérios de aceite? | “Funcionar em mobile” permanece subjetivo. |
| PA-11 | Quais metas de desempenho e disponibilidade são necessárias? Qual comportamento esperar sem conexão? | Não há critérios mensuráveis nem política de recuperação definida. |
| PA-12 | Login, favoritos, histórico, geolocalização, mapas, alertas ou compartilhamento fazem parte do escopo? | Expansão não controlada e possíveis impactos de privacidade. |
| PA-13 | Qual volume de uso e finalidade de publicação são esperados? Os termos da Open-Meteo atendem a esse cenário? | Limites, custos e condições de uso podem inviabilizar a operação pretendida. |

As decisões sobre conteúdo meteorológico, período de cinco dias, unidades e
abrangência da busca devem ser priorizadas para destravar a especificação.

## Suposições

Hipóteses de trabalho validadas e premissas para a especificação:

| ID | Suposição | Como validar |
| --- | --- | --- |
| SU-01 | A primeira entrega será uma aplicação web responsiva, não um aplicativo nativo. | Confirmado pela stack do repositório (React + Vite + Tailwind). |
| SU-02 | A consulta acontecerá para uma cidade selecionada por vez, mantendo a última cidade selecionada salva no navegador (`localStorage`). | Confirmado em `DEC-04`. |
| SU-03 | A localização inicial via navegador terá fallback direto para foco no campo de busca manual caso indisponível ou negada. | Confirmado em `DEC-08`. |
| SU-04 | Login, persistência de servidor, favoritos múltiplos, mapas e alertas não entrarão na primeira entrega. | Confirmado em `DEC-04` para o MVP. |
| SU-05 | Haverá conexão à internet para obter dados meteorológicos novos. | Definir mensagens claras de erro e tentativa em caso de falha de rede (`RF-05`). |
| SU-06 | A aplicação terá finalidade informativa cotidiana, sem garantia para decisões críticas. | Confirmado pelo contexto de uso e personas do produto. |

## Decisões

Decisões de produto e arquitetura validadas para destravar a especificação (`specs/weather-app-spec.md`):

| ID | Decisão | Justificativa (Trade-off) | Perguntas em aberto resolvidas |
| --- | --- | --- | --- |
| **DEC-01** | **Fonte de dados:** Open-Meteo (Geocoding + Forecast, sem API key). | Elimina gestão de segredos/chaves no frontend e acelera o desenvolvimento do MVP, aceitando a dependência de uma API pública gratuita. | Resolve **PA-13** |
| **DEC-02** | **Janela de "5 dias":** **Hoje + 4 próximos dias** (`D0` a `D4`), respeitando o fuso horário da cidade consultada (`timezone=auto` na API). | Conecta a condição do dia atual com os próximos 4 dias em uma linha contínua e elimina ambiguidade entre implementação e testes de aceite. | Resolve **PA-05** |
| **DEC-03** | **Unidade padrão, precisão e conversão no cliente:** **Celsius (°C)** por padrão, alternável para **Fahrenheit (°F)** com **conversão no cliente** e exibição com **no máximo 1 casa decimal**. | Evita chamadas extras de rede ao alternar `°C/°F` (troca instantânea) e mantém consistência numérica entre clima atual e previsão. | Resolve **PA-07** |
| **DEC-04** | **Sem autenticação/servidor, com persistência local da última cidade:** aplicação 100% client-side, mantendo a **última cidade selecionada pelo usuário** salva em `localStorage`. | Preserva a simplicidade de não ter backend enquanto melhora a experiência de retorno do usuário ao reabrir o app. | Resolve **PA-07** (persistência) e **PA-12** |
| **DEC-05** | **Idioma da UI:** padrão **Português (`pt-BR`)**, podendo ser alterado pelo usuário para **Espanhol (`es`)** ou **Inglês (`en`)**. | Garante experiência nativa em português por padrão e suporta usuários/viagens internacionais na UI, datas e busca localizada. | Resolve **PA-09** |
| **DEC-06** | **Desambiguação de cidades:** exibir sempre **`Nome da Cidade - Estado - País`** (usando `admin1` e `country` da Open-Meteo) nas sugestões de busca e no cabeçalho do clima. | Evita seleção incorreta em cidades homônimas com leitura imediata e padronizada. | Resolve **PA-02** |
| **DEC-07** | **Campos do Clima Atual e Previsão:** Clima Atual com pacote completo (temperatura, sensação térmica, condição + ícone, mín/máx do dia, umidade `%`, vento `km/h` e precipitação `mm`) e Previsão de 5 dias com pacote recomendado (dia/data, condição + ícone, mínima, máxima e probabilidade de chuva `%`). | Equilibra riqueza de informação no card principal com leitura compacta nos 5 cards diários em telas móveis. | Resolve **PA-04** e **PA-06** |
| **DEC-08** | **Estado inicial da aplicação:** tentar obter a localização do navegador (`navigator.geolocation`) quando não houver cidade previamente salva; se não estiver disponível ou for negada, **focar a atenção do usuário no campo de busca de cidade**. | Entrega conveniência automática quando permitida e garante um caminho imediato e claro (foco na busca) quando indisponível. | Resolve estado inicial e **SU-03** |
| **DEC-09** | **Mecânica da busca de cidades:** textbox que inicia a busca automaticamente conforme o usuário digita (*autocomplete* com `debounce`), listando sugestões no formato **`Nome da Cidade - Estado - País`**. | Reduz cliques em dispositivos móveis e permite escolher rapidamente a cidade/estado correta antes de carregar o clima. | Resolve **PA-02** e **PA-03** |
