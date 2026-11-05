# Weather App — Product Specification

Especificação de produto gerada a partir de [discovery.md](./discovery.md).
Este documento é a fonte única da verdade para o plano técnico (`plans/`),
quebra de tarefas (`tasks/`), implementação (`src/`) e testes (`tests/`).

---

## Overview

O **Weather App** é uma aplicação web responsiva de previsão do tempo voltada para decisões rápidas do cotidiano, planejamento de viagens e organização de atividades ao ar livre, tanto em dispositivos móveis quanto em desktops.

### Objetivos do Produto
- Entregar informações meteorológicas imediatas ao abrir a aplicação, aproveitando a última cidade selecionada pelo usuário ou a localização do navegador (com redirecionamento claro para busca manual caso indisponível).
- Permitir busca rápida de cidades ao digitar, desambiguando homônimos no formato **`Nome da Cidade - Estado - País`**.
- Exibir um painel completo de **Clima Atual** e a **Previsão de 5 Dias (Hoje + 4 próximos dias)** respeitando o fuso horário local da cidade consultada.
- Permitir alternância instantânea entre **Celsius (°C)** e **Fahrenheit (°F)** com precisão de no máximo 1 casa decimal.
- Suportar internacionalização da interface em **Português (`pt-BR`, padrão)**, **Espanhol (`es`)** e **Inglês (`en`)**.

### Personas Atendidas
- **Mariana (29) — Rotina e deslocamento urbano:** consulta rápida no celular (< 10s) para decidir roupa e guarda-chuva antes de sair de casa.
- **Carlos (41) — Planejador de viagens e agenda semanal:** pesquisa cidades em diferentes estados/países, consulta os 5 dias (`Hoje + 4`) e alterna unidades (`°C/°F`) e idiomas (`pt-BR`/`es`/`en`).
- **Beatriz (34) — Atividades ao ar livre:** analisa indicadores detalhados (sensação térmica, vento, umidade, precipitação e probabilidade de chuva) no celular em ambientes externos.

---

## Functional Requirements

| ID | Requisito Funcional | Regras de Negócio e Comportamento Observável | Critérios de Aceite |
| --- | --- | --- | --- |
| **FR-01** | **Inicialização inteligente** | Ordem estrita ao iniciar: **(1)** se houver última cidade válida salva no `localStorage`, carregar o clima dessa cidade; **(2)** senão, se `navigator.geolocation` existir, solicitar coordenadas com timeout de `5s` e carregar o clima usando o rótulo localizado de localização atual (`"Sua localização atual"` / `"Tu ubicación actual"` / `"Your current location"`); **(3)** se indisponível, negado ou expirado, renderizar o estado vazio inicial e aplicar foco (`focus`) no textbox de busca. | `AC-01.1`, `AC-01.2`, `AC-01.3` |
| **FR-02** | **Busca de cidades ao digitar com desambiguação** | Textbox com `maxLength=100` que dispara busca automática (`debounce` de `300ms`) a partir de `2` caracteres úteis (`trim().length >= 2`), exibindo **até 5 sugestões** (`count=5`). Cada sugestão deve seguir o formato **`Nome da Cidade - Estado - País`** (ou **`Nome da Cidade - País`** quando `admin1` estiver ausente). | `AC-02.1`, `AC-02.2`, `AC-02.3` |
| **FR-03** | **Seleção de cidade, teclado e persistência local** | Permitir selecionar sugestão por clique ou teclado (`ArrowDown`, `ArrowUp`, `Enter`) e fechar a lista com `Escape` ou clique externo. Ao selecionar uma cidade na busca, fechar a lista, salvar a cidade no `localStorage` como última cidade selecionada e carregar seu clima. | `AC-03.1`, `AC-03.2`, `AC-03.3` |
| **FR-04** | **Exibição do Clima Atual (Pacote Completo)** | Exibir para a localidade ativa: **(1)** cabeçalho `Nome da Cidade - Estado - País` (ou rótulo de localização atual), **(2)** temperatura atual, **(3)** sensação térmica, **(4)** descrição da condição climática + ícone (mapeados pelo código WMO), **(5)** mínima e máxima de hoje (`D0`), **(6)** umidade relativa (`%`), **(7)** vento (`km/h`) e **(8)** precipitação (`mm`). | `AC-04.1`, `AC-04.2` |
| **FR-05** | **Previsão de 5 Dias (Hoje + 4 próximos dias)** | Exibir exatamente `5` dias consecutivos (`D0` a `D4`) no fuso horário da localidade consultada (`timezone=auto`). Cada dia deve exibir: **(1)** rótulo de dia (`D0` como `"Hoje"`/`"Hoy"`/`"Today"` e `D1..D4` com dia da semana abreviado + dia/mês no idioma ativo), **(2)** condição climática + ícone, **(3)** mínima, **(4)** máxima e **(5)** probabilidade máxima de chuva (`%`). | `AC-05.1`, `AC-05.2`, `AC-05.3` |
| **FR-06** | **Alternância de unidade (`°C` ↔ `°F`) no cliente** | Padrão inicial em **Celsius (`°C`)**. A troca para **Fahrenheit (`°F`)** (e retorno para `°C`) recalcula todas as temperaturas no cliente (`°F = °C × 1.8 + 32`) sem nova chamada HTTP. Arredondar para **no máximo 1 casa decimal** (`Math.round(v * 10) / 10`), omitindo decimal zero em inteiros (ex.: `24°C`, `23,5°C` / `23.5°F` conforme locale) e normalizando `-0` para `0`. | `AC-06.1`, `AC-06.2`, `AC-06.3` |
| **FR-07** | **Seletor de idioma (`pt-BR`, `es`, `en`)** | Padrão inicial **`pt-BR`**, com alternância imediata para **`es`** e **`en`**. Atualiza textos da UI, mensagens de estado, descrições WMO e formatação de datas/números sem recarregar a previsão, além de enviar `language` (`pt`, `es`, `en`) nas próximas buscas de cidades. | `AC-07.1`, `AC-07.2` |
| **FR-08** | **Estados de Loading, Vazio e Erro** | Exibir indicador de carregamento com `role="status"`, estado vazio quando nenhuma cidade estiver ativa ou quando a busca não retornar resultados, e estado de erro com botão **"Tentar novamente"** quando a consulta de previsão falhar. Falhas no geocoding não apagam o clima já exibido; falhas no forecast substituem a área de clima pelo estado de erro da localidade solicitada. | `AC-08.1`, `AC-08.2`, `AC-08.3` |

---

## User Stories

Todas as User Stories seguem o formato **"Como [persona], quero [ação] para [valor]"**, utilizando as personas definidas em [discovery.md](./discovery.md) (**Mariana**, **Carlos** e **Beatriz**) e mantendo conexão explícita 1:1 com os requisitos funcionais (`FR-01` a `FR-08`):

| ID | Requisito Funcional Conectado | Persona | User Story ("Como [persona], quero [ação] para [valor]") |
| --- | --- | --- | --- |
| **US-01** | **FR-01** (Inicialização inteligente) | **Mariana** *(29, rotina e deslocamento urbano)* | **Como** Mariana, **quero** que o aplicativo abra carregando a última cidade consultada ou tentando detectar minha localização pelo navegador (ou focando direto no campo de busca se não estiver disponível), **para** conferir o clima em menos de 10 segundos antes de sair de casa. |
| **US-02** | **FR-02** (Busca ao digitar com desambiguação) | **Carlos** *(41, planejador de viagens e agenda semanal)* | **Como** Carlos, **quero** digitar o nome de uma cidade no campo de busca e ver sugestões automáticas no formato `Nome da Cidade - Estado - País`, **para** identificar rapidamente a cidade correta sem confundir localidades homônimas em estados ou países diferentes. |
| **US-03** | **FR-03** (Seleção de cidade e persistência local) | **Mariana** *(29, rotina e deslocamento urbano)* | **Como** Mariana, **quero** selecionar uma cidade nas sugestões e tê-la salva automaticamente no meu navegador como última cidade consultada, **para** não precisar pesquisar minha cidade novamente toda vez que reabrir o aplicativo no celular. |
| **US-04** | **FR-04** (Exibição do Clima Atual — Pacote Completo) | **Beatriz** *(34, organizadora de atividades ao ar livre)* | **Como** Beatriz, **quero** visualizar no painel de clima atual a temperatura, sensação térmica, condição com ícone, mínima/máxima de hoje, umidade (`%`), vento (`km/h`) e precipitação (`mm`), **para** avaliar com segurança se posso manter, adaptar ou remarcar um treino ao ar livre. |
| **US-05** | **FR-05** (Previsão de 5 Dias — Hoje + 4) | **Carlos** *(41, planejador de viagens e agenda semanal)* | **Como** Carlos, **quero** consultar a previsão de 5 dias composta por hoje e os próximos 4 dias (`D0` a `D4`) com condição, mínima, máxima e probabilidade de chuva (`%`) no fuso da cidade pesquisada, **para** planejar minhas viagens e escolher os melhores dias para deslocamento. |
| **US-06** | **FR-06** (Alternância entre Celsius e Fahrenheit) | **Carlos** *(41, planejador de viagens e agenda semanal)* | **Como** Carlos, **quero** alternar instantaneamente entre Celsius (`°C`) e Fahrenheit (`°F`) com precisão de no máximo 1 casa decimal, **para** interpretar ou compartilhar temperaturas na unidade utilizada pelo meu destino ou cliente sem esperar novo carregamento. |
| **US-07** | **FR-07** (Seletor de idioma `pt-BR`, `es`, `en`) | **Carlos** *(41, planejador de viagens e agenda semanal)* | **Como** Carlos, **quero** utilizar a interface em Português (`pt-BR`) por padrão e poder alternar para Espanhol (`es`) ou Inglês (`en`), **para** navegar e consultar cidades, datas e condições climáticas no idioma mais adequado a cada viagem. |
| **US-08** | **FR-08** (Estados de Loading, Vazio e Erro) | **Beatriz** *(34, organizadora de atividades ao ar livre)* | **Como** Beatriz, **quero** ver avisos claros quando o clima estiver carregando, quando uma busca não tiver resultados ou quando ocorrer falha de rede com botão **"Tentar novamente"**, **para** recuperar a consulta rapidamente mesmo usando conexões móveis instáveis em ambientes externos. |

---

## Acceptance Criteria

Todos os critérios de aceite estão estruturados no formato explícito **Given / When / Then** (**Dado / Quando / Então**) e são diretamente verificáveis em testes unitários (Vitest + Testing Library) e testes E2E (Playwright).

### US-01 / FR-01 — Inicialização inteligente
- **AC-01.1 (Última cidade salva):**
  - **Given (Dado):** que existe uma última cidade válida (`name`, `country`, `latitude`, `longitude`) salva no `localStorage`
  - **When (Quando):** o usuário abre ou recarrega a aplicação
  - **Then (Então):** a aplicação consulta automaticamente a previsão para essas coordenadas e renderiza o Clima Atual e a Previsão de 5 Dias dessa cidade sem acionar `navigator.geolocation`.
- **AC-01.2 (Geolocalização no primeiro acesso):**
  - **Given (Dado):** que não há cidade salva no `localStorage` e `navigator.geolocation` está disponível
  - **When (Quando):** o usuário abre a aplicação e concede permissão em até `5s`
  - **Then (Então):** a aplicação obtém `latitude` e `longitude`, carrega o Clima Atual e a Previsão de 5 Dias e exibe o cabeçalho localizado de localização atual (`"Sua localização atual"` / `"Tu ubicación actual"` / `"Your current location"`).
- **AC-01.3 (Fallback para foco na busca):**
  - **Given (Dado):** que não há cidade salva no `localStorage` e a geolocalização não está disponível, foi negada ou excedeu `5s`
  - **When (Quando):** a tentativa de inicialização automática encerra
  - **Then (Então):** a aplicação exibe a mensagem orientativa de estado vazio inicial sem alerta de erro bloqueante e define o textbox de busca como elemento focado (`document.activeElement`).

### US-02 / FR-02 — Busca de cidades ao digitar com desambiguação
- **AC-02.1 (Busca automática ao digitar com debounce e limite de 5 itens):**
  - **Given (Dado):** que o textbox de busca está visível
  - **When (Quando):** o usuário digita pelo menos `2` caracteres úteis (`trim().length >= 2`) e aguarda `300ms`
  - **Then (Então):** a aplicação consulta o geocoding e exibe uma lista contendo no máximo `5` sugestões correspondentes sem exigir clique em botão de submissão.
- **AC-02.2 (Formato `Nome da Cidade - Estado - País`):**
  - **Given (Dado):** que o geocoding retorna cidades com `name`, `admin1` (opcional) e `country`
  - **When (Quando):** as sugestões são renderizadas
  - **Then (Então):** itens com `admin1` exibem exatamente `Nome da Cidade - Estado - País` e itens sem `admin1` exibem `Nome da Cidade - País`.
- **AC-02.3 (Busca sem resultados):**
  - **Given (Dado):** que o usuário digitou `2` ou mais caracteres úteis no textbox de busca
  - **When (Quando):** o serviço de geocoding retorna `0` resultados (ou omite o array `results`)
  - **Then (Então):** a interface exibe uma mensagem de estado vazio no idioma ativo informando que nenhuma cidade foi encontrada.

### US-03 / FR-03 — Seleção de cidade, teclado e persistência local
- **AC-03.1 (Seleção por clique ou teclado):**
  - **Given (Dado):** que a lista de sugestões está aberta com resultados
  - **When (Quando):** o usuário clica em uma sugestão ou navega com `ArrowDown`/`ArrowUp` e pressiona `Enter`
  - **Then (Então):** a lista de sugestões fecha e a aplicação inicia o carregamento do clima da cidade escolhida.
- **AC-03.2 (Fechamento da lista sem seleção):**
  - **Given (Dado):** que a lista de sugestões está aberta
  - **When (Quando):** o usuário pressiona `Escape` ou clica fora do componente de busca
  - **Then (Então):** a lista de sugestões fecha mantendo a cidade ativa atual inalterada.
- **AC-03.3 (Persistência da última cidade no `localStorage`):**
  - **Given (Dado):** que o usuário selecionou uma cidade na lista de sugestões
  - **When (Quando):** a seleção é confirmada
  - **Then (Então):** os metadados da cidade (`name`, `admin1`, `country`, `latitude`, `longitude`) são gravados no `localStorage` como última cidade selecionada.

### US-04 / FR-04 — Clima Atual (Pacote Completo)
- **AC-04.1 (Exibição dos 8 indicadores obrigatórios):**
  - **Given (Dado):** que os dados meteorológicos de uma localidade foram carregados com sucesso
  - **When (Quando):** o painel de Clima Atual é renderizado
  - **Then (Então):** estão visíveis: (1) cabeçalho `Nome da Cidade - Estado - País` (ou localização atual), (2) temperatura atual, (3) sensação térmica, (4) condição climática textual + ícone WMO, (5) mínima e máxima de hoje (`D0`), (6) umidade em `%`, (7) vento em `km/h` e (8) precipitação em `mm`.
- **AC-04.2 (Formatação numérica no Clima Atual):**
  - **Given (Dado):** os valores de temperatura atual, sensação térmica, mínima e máxima de hoje
  - **When (Quando):** são renderizados no painel de Clima Atual
  - **Then (Então):** todos apresentam o símbolo da unidade ativa (`°C` ou `°F`), no máximo `1` casa decimal e nenhum valor `-0`, `NaN` ou `undefined`.

### US-05 / FR-05 — Previsão de 5 Dias (Hoje + 4)
- **AC-05.1 (Janela exata de Hoje + 4 próximos dias):**
  - **Given (Dado):** que a previsão diária da localidade foi carregada
  - **When (Quando):** a seção de previsão de 5 dias é renderizada
  - **Then (Então):** são exibidos exatamente `5` cards ordenados cronologicamente correspondentes a **Hoje (`D0`) e aos 4 dias seguintes (`D1` a `D4`)**.
- **AC-05.2 (Campos obrigatórios por card diário):**
  - **Given (Dado):** cada um dos `5` cards diários (`D0` a `D4`)
  - **When (Quando):** o card é renderizado
  - **Then (Então):** exibe: (1) rótulo `"Hoje"`/`"Hoy"`/`"Today"` em `D0` e dia da semana abreviado + dia/mês em `D1..D4` no locale ativo, (2) condição climática + ícone WMO, (3) temperatura mínima na unidade ativa, (4) temperatura máxima na unidade ativa e (5) probabilidade máxima de chuva em `%`.
- **AC-05.3 (Preservação da data no fuso horário da cidade):**
  - **Given (Dado):** que a API retorna as datas diárias (`YYYY-MM-DD`) no fuso horário local da cidade consultada (`timezone=auto`)
  - **When (Quando):** os 5 dias são formatados na interface em qualquer fuso horário do navegador do usuário
  - **Then (Então):** cada card preserva exatamente a data civil retornada pela API, sem deslocar o dia em `-1` ou `+1`.

### US-06 / FR-06 — Alternância entre Celsius e Fahrenheit
- **AC-06.1 (Unidade padrão inicial em Celsius):**
  - **Given (Dado):** que o usuário abre a aplicação na sessão inicial
  - **When (Quando):** temperaturas são exibidas
  - **Then (Então):** a unidade ativa padrão é **Celsius (`°C`)**.
- **AC-06.2 (Conversão imediata no cliente sem chamada HTTP):**
  - **Given (Dado):** que o clima já está carregado na tela
  - **When (Quando):** o usuário alterna entre `°C` e `°F`
  - **Then (Então):** todas as temperaturas (atual, sensação térmica e mínimas/máximas de `D0..D4`) são recalculadas imediatamente no cliente (`°F = °C × 1.8 + 32`) com `0` novas requisições HTTP à API de forecast.
- **AC-06.3 (Regra de arredondamento de até 1 casa decimal):**
  - **Given (Dado):** uma temperatura original ou convertida (ex.: `24.0`, `23.46`, `-0.03`)
  - **When (Quando):** o valor é formatado para exibição
  - **Then (Então):** o número é arredondado para no máximo `1` casa decimal (`Math.round(v * 10) / 10`), valores inteiros não exibem decimal zero (`24°C`), valores fracionários exibem `1` casa decimal (`23,5°C` em `pt-BR`/`es` ou `23.5°C` em `en`) e `-0` é exibido como `0`.

### US-07 / FR-07 — Idioma da Interface (`pt-BR`, `es`, `en`)
- **AC-07.1 (Idioma padrão `pt-BR`):**
  - **Given (Dado):** que o usuário abre a aplicação na sessão inicial
  - **When (Quando):** a interface é renderizada
  - **Then (Então):** o idioma ativo padrão é **Português (`pt-BR`)**.
- **AC-07.2 (Alternância imediata entre `pt-BR`, `es` e `en`):**
  - **Given (Dado):** que a aplicação está aberta em qualquer estado (vazio, carregado ou erro)
  - **When (Quando):** o usuário seleciona `pt-BR`, `es` ou `en`
  - **Then (Então):** todos os rótulos fixos da UI, descrições WMO, rótulos de dias (`D0..D4`) e mensagens de estado mudam imediatamente no cliente sem nova requisição de forecast, e próximas buscas de geocoding enviam `language=pt`, `language=es` ou `language=en`.

### US-08 / FR-08 — Estados de Loading, Vazio e Erro
- **AC-08.1 (Indicador de carregamento acessível):**
  - **Given (Dado):** que uma consulta de geocoding ou de forecast está em andamento
  - **When (Quando):** a resposta ainda não retornou
  - **Then (Então):** a interface renderiza um indicador de carregamento com `role="status"` e rótulo acessível no idioma ativo.
- **AC-08.2 (Estado de erro de forecast com botão "Tentar novamente"):**
  - **Given (Dado):** que uma requisição de forecast falha por erro de rede, status HTTP não-2xx ou timeout (`> 8s`)
  - **When (Quando):** a falha é processada
  - **Then (Então):** o loading é encerrado, a área principal exibe mensagem de erro no idioma ativo e o clique no botão **"Tentar novamente"** repete a consulta para as mesmas coordenadas.
- **AC-08.3 (Isolamento de falha e proteção contra valores inválidos):**
  - **Given (Dado):** que uma busca de geocoding falha enquanto já existe uma cidade carregada, ou que o forecast retorna campos opcionais nulos
  - **When (Quando):** a interface é atualizada
  - **Then (Então):** falhas no geocoding não apagam o clima já carregado na tela, e nenhum componente renderiza `"NaN"`, `"undefined"` ou `"null"`.

---

## Non-Functional Requirements

| ID | Categoria | Requisito Não-Funcional e Meta Verificável |
| --- | --- | --- |
| **NFR-01** | **Responsividade (Mobile-First)** | A interface deve ser integralmente funcional e legível sem rolagem horizontal em larguras de viewport de **`320px` a `1440px+`**, garantindo que busca, seletores de unidade/idioma, clima atual e previsão de 5 dias funcionem em dispositivos móveis e desktop. |
| **NFR-02** | **Ergonomia de Toque** | Todos os controles interativos principais (campo de busca, itens de sugestão de cidade, alternador `°C/°F`, seletor de idioma e botão "Tentar novamente") devem possuir área clicável/tocável mínima recomendada para mobile (**`44×44px`** ou espaçamento equivalente seguro). |
| **NFR-03** | **Acessibilidade (WCAG 2.2 AA)** | Uso de elementos HTML semânticos, rótulos acessíveis (`label` / `aria-label`) em todos os campos e botões, navegação integral por teclado (`Tab`, `Shift+Tab`, `Enter`, `Escape`), indicador visível de foco (`focus-visible`), contraste mínimo de **4.5:1** para texto normal sobre os cards *dark glassmorphism* e anúncio de estados dinâmicos (loading/erro/resultados) via atributos ARIA apropriados. |
| **NFR-04** | **Performance e Uso de Rede** | A busca ao digitar deve aplicar *debounce* (ex.: `300ms`) para evitar requisições excessivas; requisições obsoletas em andamento devem ser canceladas ao iniciar nova busca/seleção; a alternância entre `°C` e `°F` deve ocorrer no cliente em **menos de 50ms** sem tráfego de rede. |
| **NFR-05** | **Disponibilidade e Resiliência** | Requisições externas devem possuir timeout definido (ex.: até `8–10s` para APIs e até `5s` para geolocalização inicial) para que a interface nunca fique presa indefinidamente em estado de loading. |
| **NFR-06** | **Consistência de Estado** | Em buscas ou trocas rápidas de cidade, a interface nunca deve exibir o cabeçalho de uma cidade combinado com os dados meteorológicos de outra (*prevenção de race conditions*). |
| **NFR-07** | **Privacidade e Segurança** | A aplicação não exige login, não coleta dados pessoais em servidor próprio e não armazena chaves secretas no cliente. Apenas dados estritamente necessários (última cidade selecionada e preferências de exibição) podem ser persistidos no `localStorage` do próprio navegador. |

---

## Edge Cases

| # | Categoria | Caso Extremo (Edge Case) | Comportamento Esperado da Aplicação |
| --- | --- | --- | --- |
| **EC-01** | **Input vazio** | **Campo vazio, apenas espaços em branco ou limpeza rápida do input (`Backspace`) com requisição em andamento** | Não disparar chamadas ao geocoding quando `trim().length === 0`; cancelar imediatamente (`AbortController`) qualquer busca em andamento para impedir que sugestões atrasadas apareçam com o campo já vazio; fechar o dropdown sem exibir erro. |
| **EC-02** | **Input curto / longo** | **Texto com 1 caractere útil ou colagem de texto muito longo (> 100 caracteres)** | Não disparar busca automática com menos de 2 caracteres úteis; limitar a entrada a `100` caracteres (`maxLength=100`) e normalizar múltiplos espaços consecutivos antes da consulta. |
| **EC-03** | **Caracteres especiais** | **Acentos, hífens, apóstrofos (ex.: `São Paulo`, `Xique-Xique`, `D'Ávila`) vs. símbolos/emojis isolados (`@#$`, `🌧️`)** | Preservar letras acentuadas, hífens e apóstrofos codificando a query com `encodeURIComponent`; para entradas compostas apenas por símbolos ou emojis sem correspondência, exibir graciosamente o estado vazio (*"Nenhuma cidade encontrada"*), sem quebrar a UI. |
| **EC-04** | **Cidade inexistente / Geocoding sem resultados** | **Termo sem correspondência ou resposta da Open-Meteo sem a propriedade `results`** *(a Open-Meteo omite o array `results` quando não há matches)* | Tratar a ausência de `results` (`results ?? []`) como lista vazia válida (e não como falha técnica); exibir mensagem clara de *"Nenhuma cidade encontrada"* no idioma ativo e ignorar a tecla `Enter` para não disparar forecast sem coordenadas. |
| **EC-05** | **Ambiguidade geográfica** | **Cidades homônimas (ex.: múltiplas `"Santa Maria"` ou `"São José"`) e localidades sem `admin1` (Estado)** | Listar as sugestões no formato `Nome da Cidade - Estado - País`; quando `admin1` não existir na resposta da API, exibir `Nome da Cidade - País` (nunca renderizar `undefined`, `null` ou hífens duplicados `Cidade - - País`). |
| **EC-06** | **Falha de API (Geocoding vs. Forecast)** | **Erro de rede, HTTP `4xx`/`5xx` ou `429 Too Many Requests` na busca de cidades ou na consulta de previsão** | **Falha no Geocoding:** exibir aviso discreto dentro do dropdown de busca mantendo visível o clima da cidade que já estava aberta na tela. **Falha no Forecast:** encerrar o loading, exibir estado de erro amigável no idioma ativo e apresentar o botão **"Tentar novamente"**. |
| **EC-07** | **Timeout** | **Lentidão extrema na Geolocalização inicial (`> 5s`), no Geocoding (`> 5s`) ou no Forecast (`> 8s`)** | Abortar a operação ao atingir o respectivo timeout; na geolocalização inicial, seguir silenciosamente para o estado vazio e focar o textbox de busca; no forecast, exibir mensagem de tempo esgotado com botão **"Tentar novamente"**. |
| **EC-08** | **Resposta parcial da API** | **Campos opcionais nulos/ausentes (ex.: `precipitation_probability_max` com `null`, código WMO não mapeado ou menos de 5 itens em `daily`)** | Substituir valores numéricos opcionais ausentes por fallback seguro (`0%` ou `--`), mapear códigos WMO desconhecidos para condição/ícone neutro padrão e renderizar até `5` dias disponíveis (`slice(0, 5)`) sem lançar erro nem exibir `NaN` ou `undefined`. |
| **EC-09** | **Geolocalização sem nome reverso** | **Permissão de localização concedida, mas serviço de nome reverso da localidade indisponível ou sem cidade próxima** | Exibir normalmente o Clima Atual e a Previsão de 5 Dias para as coordenadas (`latitude`/`longitude`) obtidas, usando um título localizado padrão (`"Sua localização atual"` / `"Tu ubicación actual"` / `"Your current location"`). |
| **EC-10** | **Concorrência / Race Condition** | **Digitação rápida no textbox ou seleção seguida de duas cidades diferentes** | Cancelar a requisição anterior via `AbortController` (ou descartar respostas fora de ordem), garantindo que o cabeçalho da cidade e os dados meteorológicos exibidos pertençam sempre à última seleção do usuário. |
| **EC-11** | **Temperaturas extremas e `-0`** | **Temperaturas negativas, zero absoluto da escala (`0°C` / `32°F`) e arredondamento de `-0.04°C`** | Normalizar `-0` para `0` antes de formatar (`0°C` em vez de `-0°C`), manter sinal negativo correto em temperaturas abaixo de zero e respeitar o limite de no máximo 1 casa decimal em `°C` e `°F`. |
| **EC-12** | **Persistência local corrompida ou bloqueada** | **`localStorage` desabilitado (modo privado restrito) ou JSON salvo inválido/incompleto** | Envolver leituras e escritas em `try/catch`; se o JSON salvo estiver inválido ou sem coordenadas numéricas finitas, remover a chave corrompida e seguir o fluxo normal de inicialização sem quebrar a aplicação. |

---

## Assumptions

- **AS-01:** A aplicação será entregue como uma Single-Page Application (SPA) web responsiva executada no navegador do usuário.
- **AS-02:** Os dados de geocoding e previsão meteorológica serão fornecidos pelos endpoints públicos da **Open-Meteo**, que não exigem chave de autenticação (API key) para uso não-comercial/treinamento.
- **AS-03:** O usuário possui conexão ativa com a internet no momento de buscar novas cidades ou atualizar dados meteorológicos.
- **AS-04:** Uma única cidade/localidade ativa é visualizada por vez na tela principal.
- **AS-05:** As velocidades de vento permanecem em `km/h` e a precipitação em `mm` independentemente da alternância de temperatura entre `°C` e `°F`, pois o requisito de alternância refere-se especificamente a Celsius e Fahrenheit.

---

## Risks

| ID | Risco | Probabilidade | Impacto | Mitigação na Especificação |
| --- | --- | --- | --- | --- |
| **RK-01** | Permissão de localização do navegador negada ou indisponível no primeiro acesso. | Alta | Médio | Definir prioridade para última cidade salva e fallback automático (`5s`) que foca o campo de busca sem exibir erro bloqueante (`FR-01`, `AC-01.3`, `EC-07`). |
| **RK-02** | Seleção de cidade errada por nomes homônimos em estados ou países diferentes. | Alta | Alto | Exigir exibição padronizada `Nome da Cidade - Estado - País` nas sugestões e no cabeçalho principal (`FR-02`, `FR-04`, `AC-02.2`, `EC-05`). |
| **RK-03** | Indisponibilidade, lentidão ou falha de rede ao consultar a Open-Meteo em rede móvel. | Média | Alto | Exigir estados explícitos de loading/erro, timeouts (`5s` geocoding / `8s` forecast) e botão **"Tentar novamente"** (`FR-08`, `NFR-05`, `EC-06`, `EC-07`). |
| **RK-04** | Deslocamento de datas (`-1` dia) ao exibir a previsão de 5 dias para cidades em outros fusos horários. | Média | Alto | Especificar que `D0..D4` (Hoje + 4) preserva a data civil local da cidade consultada (`FR-05`, `AC-05.3`). |
| **RK-05** | Condição de corrida (*race condition*) ao digitar rápido ou trocar de cidade repetidamente. | Média | Alto | Exigir cancelamento/descarte de respostas obsoletas via `AbortController` e *debounce* de `300ms` na busca (`NFR-04`, `NFR-06`, `EC-01`, `EC-10`). |
| **RK-06** | Sobrecarga visual em telas móveis pequenas (`320px–375px`) com todos os indicadores e seletores. | Média | Médio | Definir responsividade *mobile-first* sem rolagem horizontal, limite de `5` sugestões e alvos de toque mínimos de `44×44px` (`FR-02`, `NFR-01`, `NFR-02`). |
| **RK-07** | Divergência de tradução entre interface, datas e condições climáticas nos 3 idiomas (`pt-BR`, `es`, `en`). | Média | Médio | Exigir cobertura de i18n para rótulos de UI, condições WMO, dias da semana e mensagens de estado (`FR-07`, `AC-07.2`). |

---

## Out of Scope

Os seguintes itens estão **explicitamente fora do escopo** desta versão (MVP):

1. **Autenticação, contas de usuário e backend próprio** (login, cadastro, perfil ou banco de dados em servidor).
2. **Lista de múltiplas cidades favoritas, histórico de buscas recentes ou comparação lado a lado** entre várias cidades simultaneamente (apenas a última cidade selecionada é mantida no `localStorage`).
3. **Previsão hora a hora (24h), previsão estendida além de 5 dias (7/14 dias)** ou gráficos de séries históricas.
4. **Indicadores meteorológicos avançados fora do pacote acordado** (índice UV, qualidade do ar/AQI, pressão atmosférica, visibilidade, ponto de orvalho, direção do vento em graus/bússola, horários de nascer/pôr do sol e fases da lua).
5. **Alternância de unidades secundárias** (velocidade do vento permanece em `km/h` e precipitação em `mm`, sem conversão para `mph` ou `in`).
6. **Mapas meteorológicos interativos**, radares de nuvens ou imagens de satélite.
7. **Notificações push, alertas sonoros ou atualização automática em segundo plano (*auto-refresh polling*)**.
8. **Funcionamento offline completo (PWA instalável com sincronização em background)** ou aplicativos móveis nativos (iOS/Android).
9. **Compartilhamento nativo/social**, exportação de relatórios ou geração de imagens do card de clima.
10. **Alternância de tema claro/escuro** (a aplicação seguirá exclusivamente o tema visual *dark glassmorphism* definido para o projeto).
11. **Outros idiomas além de Português (`pt-BR`), Espanhol (`es`) e Inglês (`en`)**.

---

## Open Questions

Nenhuma pergunta bloqueante permanece em aberto. Os pontos abaixo foram consolidados com decisões padrão determinísticas para o Plano Técnico e Testes:

| ID | Ponto Consolidado | Regra Determinística Adotada |
| --- | --- | --- |
| **OQ-01** | Escopo exato da persistência em `localStorage` | Persistir obrigatoriamente a **última cidade selecionada pelo usuário** (`FR-03`, `AC-03.3`). Unidade (`°C`) e idioma (`pt-BR`) iniciam em seus valores padrão documentados (`AC-06.1`, `AC-07.1`). |
| **OQ-02** | Comportamento visual quando uma nova consulta falha após uma cidade já estar carregada | Falhas no **dropdown de busca (geocoding)** mantêm o clima atual visível na tela; falhas ao carregar o **forecast de uma nova cidade selecionada** exibem o estado de erro principal com o botão **"Tentar novamente"** para a cidade solicitada (`FR-08`, `AC-08.2`, `AC-08.3`, `EC-06`). |

---

## Traceability Matrix

Matriz de rastreabilidade conectando cada **User Story** ao seu **Requisito Funcional**, **Acceptance Criteria**, **Non-Functional Requirements**, **Edge Cases** e **Camada de Teste** recomendada:

| User Story | Persona | Functional Requirement | Acceptance Criteria (Given/When/Then) | Non-Functional Requirements Relevantes | Edge Cases Associados | Cobertura de Testes Alvo |
| --- | --- | --- | --- | --- | --- | --- |
| **US-01** — Inicialização inteligente (última cidade, geolocalização ou foco na busca) | Mariana | **FR-01** | `AC-01.1`, `AC-01.2`, `AC-01.3` | `NFR-03` (foco acessível), `NFR-05` (timeout `5s`), `NFR-07` (privacidade / `localStorage`) | `EC-07`, `EC-09`, `EC-12` | Unit (`useWeather` / storage / geo) + E2E (abertura inicial e foco no input) |
| **US-02** — Busca ao digitar com desambiguação `Nome da Cidade - Estado - País` | Carlos | **FR-02** | `AC-02.1`, `AC-02.2`, `AC-02.3` | `NFR-01` (mobile `320px+`), `NFR-02` (toque `44×44px`), `NFR-03` (combobox/labels), `NFR-04` (debounce `300ms`), `NFR-05` (timeout `5s`), `NFR-06` (race conditions) | `EC-01`, `EC-02`, `EC-03`, `EC-04`, `EC-05`, `EC-06`, `EC-10` | Unit (`geocodingService`, `SearchBar`, debounce) + E2E (busca e homônimos) |
| **US-03** — Seleção de cidade, navegação por teclado e persistência local | Mariana | **FR-03** | `AC-03.1`, `AC-03.2`, `AC-03.3` | `NFR-02` (toque `44×44px`), `NFR-03` (`ArrowDown`/`ArrowUp`/`Enter`/`Escape`), `NFR-06` (consistência de cidade ativa), `NFR-07` (`localStorage`) | `EC-05`, `EC-10`, `EC-12` | Unit (`SearchBar` teclado + `localStorage`) + E2E (seleção e reload da página) |
| **US-04** — Painel completo de Clima Atual (8 indicadores) | Beatriz | **FR-04** | `AC-04.1`, `AC-04.2` | `NFR-01` (layout responsivo), `NFR-03` (semântica e contraste 4.5:1), `NFR-06` (consistência de dados) | `EC-05`, `EC-08`, `EC-09`, `EC-11` | Unit (`CurrentWeather` + mapeamento WMO) + E2E (exibição dos indicadores) |
| **US-05** — Previsão de 5 Dias (`Hoje + 4` no fuso da cidade) | Carlos | **FR-05** | `AC-05.1`, `AC-05.2`, `AC-05.3` | `NFR-01` (cards responsivos mobile/desktop), `NFR-03` (leitura acessível), `NFR-06` (integridade temporal) | `EC-08`, `EC-11` | Unit (`ForecastList` + parser de datas sem shift de fuso) + E2E (5 cards `D0..D4`) |
| **US-06** — Alternância instantânea entre Celsius (`°C`) e Fahrenheit (`°F`) | Carlos | **FR-06** | `AC-06.1`, `AC-06.2`, `AC-06.3` | `NFR-02` (botão `44×44px`), `NFR-03` (estado ativo acessível), `NFR-04` (conversão no cliente `< 50ms` sem rede) | `EC-11` | Unit (função pura `°C ↔ °F`, arredondamento 1 casa e `-0`) + E2E (toggle `°C/°F`) |
| **US-07** — Seletor de idioma da interface (`pt-BR`, `es`, `en`) | Carlos | **FR-07** | `AC-07.1`, `AC-07.2` | `NFR-01` (sem quebra com textos traduzidos), `NFR-02` (alvo de toque), `NFR-03` (`lang` / labels acessíveis), `NFR-04` (troca imediata na UI) | `EC-03`, `EC-08`, `EC-09` | Unit (dicionário i18n + formatação de datas/WMO) + E2E (troca `pt-BR`/`es`/`en`) |
| **US-08** — Estados de Loading, Vazio e Erro com "Tentar novamente" | Beatriz | **FR-08** | `AC-08.1`, `AC-08.2`, `AC-08.3` | `NFR-02` (botão "Tentar novamente" `44×44px`), `NFR-03` (`role="status"` / `aria-live`), `NFR-05` (timeouts e recuperação), `NFR-06` (isolamento de erro) | `EC-04`, `EC-06`, `EC-07`, `EC-08`, `EC-10` | Unit (estados loading/empty/error + retry) + E2E (simulação de falha de API e retry) |

