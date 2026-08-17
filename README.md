<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Raiz — Gestão Clínica & Prontuário Nutricional</title>
  
  <!-- Fontes do Google -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,500;0,9..144,600;0,9..144,700;1,9..144,400&family=Inter:wght@400;500;600;700&family=IBM+Plex+Mono:wght@500;600&display=swap" rel="stylesheet">
  
  <!-- Chart.js para Renderização dos Gráficos -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <!-- SDKs do Firebase (Compatibility mode) -->
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore-compat.js"></script>

  <style>
    /* ============================== DESIGN TOKENS & RESET ============================== */
    :root {
      --paper: #F2F4EE;
      --paperCard: #FFFFFF;
      --ink: #1B2B22;
      --inkSoft: #4B5A4F;
      --inkFaint: #8B9A8E;
      --line: #DCE2D6;
      --pine: #1F6F5C;
      --pineDark: #16302B;
      --pineTint: #E4F0EB;
      --amber: #C99A3C;
      --amberTint: #FBF3E1;
      --amberDark: #7A5A18;
      --red: #C1443B;
      --redTint: #FBEAE8;
      --redDark: #7A2C22;
      --green: #3F7A5C;
      --greenTint: #E1EFE6;
      --greenDark: #204E37;

      --font-display: 'Fraunces', Georgia, serif;
      --font-body: 'Inter', system-ui, -apple-system, sans-serif;
      --font-mono: 'IBM Plex Mono', monospace;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: var(--font-body);
      background-color: var(--paper);
      color: var(--ink);
      min-height: 100vh;
      line-height: 1.5;
    }

    button, input, select, textarea { font-family: inherit; font-size: inherit; }
    button { cursor: pointer; border: none; background: none; }

    /* Utilitários */
    .hidden { display: none !important; }
    .flex { display: flex; }
    .flex-col { flex-direction: column; }
    .items-center { align-items: center; }
    .justify-between { justify-content: space-between; }
    .justify-center { justify-content: center; }
    .flex-wrap { flex-wrap: wrap; }
    .flex-1 { flex: 1; }
    .grid { display: grid; }
    .grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
    .gap-1 { gap: 0.25rem; }
    .gap-2 { gap: 0.5rem; }
    .gap-3 { gap: 0.75rem; }
    .gap-4 { gap: 1rem; }
    .gap-5 { gap: 1.25rem; }
    .gap-6 { gap: 1.5rem; }

    @media (min-width: 768px) {
      .md\:grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .md\:grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
      .md\:grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
    }

    .container {
      max-width: 76rem;
      margin: 0 auto;
      padding: 1.5rem;
    }

    /* Componentes */
    .card {
      background-color: var(--paperCard);
      border: 1px solid var(--line);
      border-radius: 1rem;
      padding: 1.25rem;
    }

    .btn {
      padding: 0.5rem 1rem;
      border-radius: 0.5rem;
      font-size: 0.875rem;
      font-weight: 600;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 0.5rem;
      transition: all 0.2s;
    }
    .btn:hover { opacity: 0.9; transform: translateY(-1px); }
    .btn-pine { background-color: var(--pine); color: #FFF; }
    .btn-amber { background-color: var(--amber); color: #FFF; }
    .btn-dark { background-color: var(--ink); color: #FFF; }
    .btn-outline { border: 1px solid var(--line); color: var(--inkSoft); background: var(--paperCard); }

    .input-group { display: flex; flex-direction: column; gap: 0.25rem; }
    .input-label { font-size: 0.75rem; font-weight: 600; color: var(--inkSoft); }
    .form-control {
      border: 1px solid var(--line);
      border-radius: 0.5rem;
      padding: 0.5rem 0.75rem;
      font-size: 0.875rem;
      outline: none;
      background-color: var(--paperCard);
      color: var(--ink);
    }
    .form-control:focus { border-color: var(--pine); }

    /* Badges de Status */
    .badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      padding: 4px 10px;
      border-radius: 999px;
      font-size: 12px;
      font-weight: 600;
      white-space: nowrap;
    }
    .badge-dot { width: 7px; height: 7px; border-radius: 999px; }
    .badge-red { background: var(--redTint); color: var(--redDark); }
    .badge-red .badge-dot { background: var(--red); }
    .badge-amber { background: var(--amberTint); color: var(--amberDark); }
    .badge-amber .badge-dot { background: var(--amber); }
    .badge-green { background: var(--greenTint); color: var(--greenDark); }
    .badge-green .badge-dot { background: var(--green); }

    /* Trilha de Etapas */
    .stage-track { display: flex; align-items: center; gap: 3px; }
    .stage-dot {
      height: 5px; border-radius: 3px; background: var(--line);
      transition: all 0.2s;
    }

    .info-row {
      display: flex;
      flex-direction: column;
      padding: 0.45rem 0;
      border-bottom: 1px solid var(--line);
    }
    .info-row:last-child { border-bottom: none; }
    .info-label { font-size: 0.75rem; text-transform: uppercase; font-weight: 600; color: var(--inkFaint); letter-spacing: 0.05em; }
    .info-value { font-size: 0.875rem; color: var(--ink); font-weight: 500; }

    /* Modais */
    .modal-overlay {
      position: fixed; inset: 0; z-index: 50;
      background-color: rgba(22, 48, 43, 0.5);
      display: flex; align-items: center; justify-content: center;
      padding: 1rem;
    }
    .modal-content {
      background-color: var(--paper);
      border: 1px solid var(--line);
      border-radius: 1rem;
      width: 100%; max-width: 38rem; max-height: 90vh;
      overflow-y: auto;
    }
    .modal-content.wide { max-width: 52rem; }
    .modal-header {
      position: sticky; top: 0; background-color: var(--paper);
      border-bottom: 1px solid var(--line); padding: 1rem 1.5rem;
      display: flex; align-items: center; justify-content: space-between; z-index: 10;
    }

    .tab-btn {
      padding: 0.625rem 1rem; font-size: 0.875rem; font-weight: 600;
      color: var(--inkFaint); border-bottom: 2px solid transparent; cursor: pointer;
    }
    .tab-btn.active { color: var(--pine); border-bottom-color: var(--pine); }
  </style>
</head>
<body>

  <!-- ============================== TOPO / NAVBAR ============================== -->
  <header style="position: sticky; top:0; z-index: 40; background: var(--paperCard); border-bottom: 1px solid var(--line); padding: 0.75rem 1.5rem;">
    <div class="flex items-center justify-between">
      <div class="flex items-center gap-3">
        <div style="width: 34px; height: 34px; border-radius: 8px; background: var(--pine); display: flex; align-items: center; justify-content: center; color: white; font-weight: bold;">
          🚴
        </div>
        <div>
          <span style="font-family: var(--font-display); font-size: 1.125rem; font-weight: 700; color: var(--ink);">Raiz &amp; Performance</span>
          <p style="font-size: 0.75rem; color: var(--inkFaint);">Gestão Clínica &amp; Prontuário Nutricional</p>
        </div>
      </div>
      <div class="flex items-center gap-3">
        <button id="btnSyncFirebase" class="btn btn-outline" style="font-size: 0.75rem; padding: 0.35rem 0.75rem;">☁️ Sincronizar Nuvem</button>
        <span id="userEmailDisplay" style="font-size: 0.75rem; color: var(--inkSoft);">willianfreitas0054@gmail.com</span>
      </div>
    </div>
  </header>

  <main class="container">
    
    <!-- ============================== TELA 1: DASHBOARD & PIPELINE ============================== -->
    <section id="viewDashboard">
      
      <!-- Cards de Métricas Rápidas -->
      <div class="grid grid-cols-1 md:grid-cols-4 gap-3" style="margin-bottom: 1.5rem;">
        <div class="card flex justify-between items-center">
          <div>
            <span style="font-size: 0.75rem; font-weight: 600; color: var(--inkSoft);">Pacientes Ativos</span>
            <div id="metricTotal" style="font-family: var(--font-display); font-size: 1.75rem; font-weight: 700;">0</div>
          </div>
          <span style="font-size: 1.5rem;">👥</span>
        </div>
        <div class="card flex justify-between items-center">
          <div>
            <span style="font-size: 0.75rem; font-weight: 600; color: var(--inkSoft);">Devolutivas em Vídeo (48h)</span>
            <div id="metricVideos" style="font-family: var(--font-display); font-size: 1.75rem; font-weight: 700; color: var(--amberDark);">0</div>
          </div>
          <span style="font-size: 1.5rem;">📹</span>
        </div>
        <div class="card flex justify-between items-center">
          <div>
            <span style="font-size: 0.75rem; font-weight: 600; color: var(--inkSoft);">Consultas ao Vivo</span>
            <div id="metricCalls" style="font-family: var(--font-display); font-size: 1.75rem; font-weight: 700; color: var(--pine);">0</div>
          </div>
          <span style="font-size: 1.5rem;">🗓️</span>
        </div>
        <div class="card flex justify-between items-center">
          <div>
            <span style="font-size: 0.75rem; font-weight: 600; color: var(--inkSoft);">Formulários Atrasados</span>
            <div id="metricOverdue" style="font-family: var(--font-display); font-size: 1.75rem; font-weight: 700; color: var(--red);">0</div>
          </div>
          <span style="font-size: 1.5rem;">⚠️</span>
        </div>
      </div>

      <!-- Filtros e Ações -->
      <div class="flex items-center justify-between flex-wrap gap-3" style="margin-bottom: 1rem;">
        <div class="flex gap-2 flex-wrap" id="filterPills">
          <button class="btn btn-pine" data-filter="all">Todos</button>
          <button class="btn btn-outline" data-filter="yellow">🟡 Aguardando Minha Ação</button>
          <button class="btn btn-outline" data-filter="red">🔴 Atrasados</button>
          <button class="btn btn-outline" data-filter="green">🟢 Concluídos</button>
        </div>
        <div class="flex gap-2">
          <input type="text" id="inputSearch" placeholder="Buscar paciente..." class="form-control" style="width: 220px;">
          <button id="btnNewPatient" class="btn btn-pine">+ Novo Paciente</button>
        </div>
      </div>

      <!-- Tabela Central do Pipeline -->
      <div class="card" style="padding: 0; overflow-x: auto;">
        <table style="width: 100%; border-collapse: collapse; font-size: 0.875rem; text-align: left; min-width: 960px;">
          <thead>
            <tr style="background: var(--paper); border-bottom: 1px solid var(--line); color: var(--inkFaint); font-size: 0.75rem; text-transform: uppercase;">
              <th style="padding: 10px 14px;">Paciente</th>
              <th style="padding: 10px 14px;">Plano / Ciclo</th>
              <th style="padding: 10px 14px;">Início</th>
              <th style="padding: 10px 14px;">Formulário Atual</th>
              <th style="padding: 10px 14px;">Status</th>
              <th style="padding: 10px 14px;">Próxima Ação</th>
              <th style="padding: 10px 14px;">Prazo</th>
              <th style="padding: 10px 14px; text-align: center;">Concluir [x]</th>
              <th style="padding: 10px 14px; text-align: right;">Ações</th>
            </tr>
          </thead>
          <tbody id="tablePatientsBody"></tbody>
        </table>
      </div>

      <!-- Feed Simulado do Google Calendar -->
      <div style="margin-top: 1.5rem;">
        <h4 style="font-size: 0.75rem; font-weight: 700; color: var(--inkFaint); text-transform: uppercase; margin-bottom: 0.5rem;">Próximos Eventos &amp; Alertas (Google Agenda)</h4>
        <div id="calendarEventsList" class="flex flex-col gap-2"></div>
      </div>

    </section>

    <!-- ============================== TELA 2: PRONTUÁRIO & MEMÓRIA CLÍNICA ============================== -->
    <section id="viewDetail" class="hidden">
      <div class="flex items-center justify-between" style="margin-bottom: 1rem;">
        <button id="btnBackToDashboard" class="btn btn-outline" style="font-size: 0.875rem;">← Voltar ao Painel</button>
        <div class="flex gap-2">
          <a id="btnWhatsDetail" href="#" target="_blank" class="btn btn-pine">💬 WhatsApp</a>
          <button id="btnNovaEntrega" class="btn btn-amber">+ Registrar Entrega</button>
        </div>
      </div>

      <!-- Cabeçalho do Paciente -->
      <div class="card flex items-start justify-between flex-wrap gap-4" style="margin-bottom: 1.5rem;">
        <div>
          <h2 id="detNome" style="font-family: var(--font-display); font-size: 1.5rem; color: var(--ink);"></h2>
          <p id="detSub" style="font-size: 0.875rem; color: var(--inkSoft);"></p>
          <div style="margin-top: 0.5rem;" class="flex items-center gap-3">
            <span id="detPlanoBadge" class="badge badge-amber"></span>
            <div id="detStageTrack" class="stage-track"></div>
            <span id="detStageText" style="font-size: 0.75rem; color: var(--inkFaint);"></span>
          </div>
        </div>
        <div class="flex gap-3">
          <div style="background: var(--pineTint); padding: 0.5rem 1rem; border-radius: 0.75rem; text-align: center;">
            <span style="font-size: 10px; text-transform: uppercase; font-weight: 700; color: var(--pineDark); display: block;">Peso Atual</span>
            <span id="detUltimoPeso" style="font-family: var(--font-mono); font-size: 1.125rem; font-weight: 700; color: var(--pineDark);">—</span>
          </div>
          <div style="background: #EEF1EF; padding: 0.5rem 1rem; border-radius: 0.75rem; text-align: center;">
            <span style="font-size: 10px; text-transform: uppercase; font-weight: 700; color: var(--inkFaint); display: block;">Último Check-in</span>
            <span id="detUltimaData" style="font-size: 0.875rem; font-weight: 600; color: var(--ink);">—</span>
          </div>
        </div>
      </div>

      <!-- Abas de Navegação Clínica -->
      <div class="flex gap-2" style="border-bottom: 1px solid var(--line); margin-bottom: 1.5rem; overflow-x: auto;">
        <button class="tab-btn active" data-tab="tabAnamnese">Ficha &amp; Anamnese</button>
        <button class="tab-btn" data-tab="tabAcompanhamentos">Acompanhamentos (Quinzenal/Mensal)</button>
        <button class="tab-btn" data-tab="tabKcal">Plano de Kcal &amp; Refeições</button>
        <button class="tab-btn" data-tab="tabComparador">Evolução &amp; Gráficos</button>
      </div>

      <!-- Conteúdo das Abas -->
      <div id="tabAnamnese" class="tab-pane">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="card"><h4 style="color: var(--pine); font-size: 0.875rem; margin-bottom: 0.75rem;">Dados Pessoais</h4><div id="infoDados"></div></div>
          <div class="card"><h4 style="color: var(--pine); font-size: 0.875rem; margin-bottom: 0.75rem;">Saúde &amp; Hábitos</h4><div id="infoSaude"></div></div>
          <div class="card"><h4 style="color: var(--pine); font-size: 0.875rem; margin-bottom: 0.75rem;">Treino &amp; Rotina</h4><div id="infoTreino"></div></div>
          <div class="card"><h4 style="color: var(--pine); font-size: 0.875rem; margin-bottom: 0.75rem;">Preferências Alimentares</h4><div id="infoAlimentacao"></div></div>
        </div>
      </div>

      <div id="tabAcompanhamentos" class="tab-pane hidden">
        <div class="flex gap-2" style="margin-bottom: 1rem;">
          <button id="btnModalQuinzenal" class="btn btn-pine">+ Novo Quinzenal</button>
          <button id="btnModalMensal" class="btn btn-amber">+ Novo Mensal</button>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div><h4 style="color: var(--pine); font-size: 0.875rem; margin-bottom: 0.5rem;">Histórico Quinzenal</h4><div id="listQuinzenal" class="flex flex-col gap-3"></div></div>
          <div><h4 style="color: var(--amberDark); font-size: 0.875rem; margin-bottom: 0.5rem;">Histórico Mensal</h4><div id="listMensal" class="flex flex-col gap-3"></div></div>
        </div>
      </div>

      <div id="tabKcal" class="tab-pane hidden">
        <button id="btnModalKcal" class="btn btn-pine" style="margin-bottom: 1rem;">+ Novo Plano de Kcal</button>
        <div id="listKcalPlans" class="flex flex-col gap-4"></div>
      </div>

      <div id="tabComparador" class="tab-pane hidden">
        <div class="card" style="margin-bottom: 1.5rem;">
          <h4 style="font-size: 0.875rem; color: var(--inkSoft); margin-bottom: 1rem;">Curva de Evolução Temporal</h4>
          <div style="height: 260px;"><canvas id="chartEvolucao"></canvas></div>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-3" style="margin-bottom: 1rem;">
          <div class="input-group"><label class="input-label">Registro A (Inicial/Anterior)</label><select id="selCompA" class="form-control"></select></div>
          <div class="input-group"><label class="input-label">Registro B (Recente)</label><select id="selCompB" class="form-control"></select></div>
        </div>
        <div class="card" style="padding: 0; overflow: hidden;"><div id="tableComparativa"></div></div>
      </div>
    </section>

  </main>

  <!-- ============================== MODAIS DE CADASTRO ============================== -->
  <div id="modalContainer" class="modal-overlay hidden">
    
    <!-- Modal: Anamnese / Novo Paciente -->
    <div id="modalAnamnese" class="modal-content wide hidden">
      <div class="modal-header">
        <h3 style="font-family: var(--font-display);">Cadastro de Paciente &amp; Anamnese</h3>
        <button class="btnCloseModal" style="font-size: 1.5rem;">&times;</button>
      </div>
      <form id="formAnamnese" style="padding: 1.5rem;" class="flex flex-col gap-4">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
          <div class="input-group"><label class="input-label">Nome Completo</label><input type="text" id="ana_nome" class="form-control" required></div>
          <div class="input-group"><label class="input-label">E-mail</label><input type="email" id="ana_email" class="form-control" required></div>
          <div class="input-group"><label class="input-label">WhatsApp (com DDD)</label><input type="text" id="ana_whatsapp" class="form-control" required></div>
          <div class="input-group">
            <label class="input-label">Plano Contratado</label>
            <select id="ana_plano" class="form-control">
              <option value="Mensal Start">Mensal Start (1 Mês)</option>
              <option value="Trimestral Foco">Trimestral Foco (3 Meses)</option>
              <option value="Semestral Híbrido">Semestral Híbrido (6 Meses)</option>
              <option value="Anual Performance">Anual Performance (12 Meses)</option>
            </select>
          </div>
          <div class="input-group"><label class="input-label">Data de Início</label><input type="date" id="ana_dataInicio" class="form-control" required></div>
          <div class="input-group"><label class="input-label">Peso Inicial (kg)</label><input type="number" step="0.1" id="ana_pesoInicial" class="form-control" value="70"></div>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
          <div class="input-group"><label class="input-label">Principal Objetivo</label><input type="text" id="ana_objetivo" class="form-control" placeholder="Ex: Hipertrofia, Ciclismo, Emagrecimento"></div>
          <div class="input-group"><label class="input-label">Modalidade de Treino / Frequência</label><input type="text" id="ana_treino" class="form-control" placeholder="Ex: Ciclismo 4x + Musculação 2x"></div>
        </div>
        <div class="input-group"><label class="input-label">Alergias, Intolerâncias ou Restrições</label><textarea id="ana_restricoes" rows="2" class="form-control"></textarea></div>
        <div class="flex justify-end gap-2"><button type="button" class="btn btn-outline btnCloseModal">Cancelar</button><button type="submit" class="btn btn-pine">Salvar Paciente</button></div>
      </form>
    </div>

    <!-- Modal: Quinzenal -->
    <div id="modalQuinzenal" class="modal-content hidden">
      <div class="modal-header"><h3 style="font-family: var(--font-display);">Check-in Quinzenal</h3><button class="btnCloseModal" style="font-size: 1.5rem;">&times;</button></div>
      <form id="formQuinzenal" style="padding: 1.5rem;" class="flex flex-col gap-3">
        <div class="grid grid-cols-2 gap-3">
          <div class="input-group"><label class="input-label">Data</label><input type="date" id="qui_data" class="form-control" required></div>
          <div class="input-group"><label class="input-label">Peso Atual (kg)</label><input type="number" step="0.1" id="qui_peso" class="form-control" required></div>
          <div class="input-group"><label class="input-label">Adesão à Dieta (1-5)</label><input type="range" min="1" max="5" id="qui_adesao" value="4" class="form-control"></div>
          <div class="input-group"><label class="input-label">Nível de Estresse (1-10)</label><input type="range" min="1" max="10" id="qui_estresse" value="5" class="form-control"></div>
        </div>
        <div class="input-group"><label class="input-label">Sintomas Digestivos / Intestino</label><input type="text" id="qui_digestao" class="form-control" placeholder="Ex: Regular, gases após refeição X"></div>
        <div class="input-group"><label class="input-label">Relato / Dificuldades</label><textarea id="qui_notas" rows="2" class="form-control"></textarea></div>
        <div class="flex justify-end gap-2"><button type="button" class="btn btn-outline btnCloseModal">Cancelar</button><button type="submit" class="btn btn-pine">Salvar Check-in</button></div>
      </form>
    </div>

    <!-- Modal: Mensal -->
    <div id="modalMensal" class="modal-content hidden">
      <div class="modal-header"><h3 style="font-family: var(--font-display);">Reavaliação Mensal</h3><button class="btnCloseModal" style="font-size: 1.5rem;">&times;</button></div>
      <form id="formMensal" style="padding: 1.5rem;" class="flex flex-col gap-3">
        <div class="grid grid-cols-2 gap-3">
          <div class="input-group"><label class="input-label">Data</label><input type="date" id="men_data" class="form-control" required></div>
          <div class="input-group"><label class="input-label">Peso em Jejum (kg)</label><input type="number" step="0.1" id="men_peso" class="form-control" required></div>
        </div>
        <div class="input-group"><label class="input-label">Evolução Geral</label><input type="text" id="men_evolucao" class="form-control" placeholder="Ex: Redução de dobras, melhora na disposição"></div>
        <div class="input-group"><label class="input-label">Meta para o Próximo Mês</label><textarea id="men_meta" rows="2" class="form-control"></textarea></div>
        <div class="flex justify-end gap-2"><button type="button" class="btn btn-outline btnCloseModal">Cancelar</button><button type="submit" class="btn btn-amber">Salvar Reavaliação</button></div>
      </form>
    </div>

    <!-- Modal: Plano de Kcal -->
    <div id="modalKcal" class="modal-content wide hidden">
      <div class="modal-header"><h3 style="font-family: var(--font-display);">Novo Plano de Kcal</h3><button class="btnCloseModal" style="font-size: 1.5rem;">&times;</button></div>
      <form id="formKcal" style="padding: 1.5rem;" class="flex flex-col gap-4">
        <div class="grid grid-cols-3 gap-3">
          <div class="input-group"><label class="input-label">Data do Plano</label><input type="date" id="kc_data" class="form-control" required></div>
          <div class="input-group"><label class="input-label">Kcal Meta</label><input type="number" id="kc_meta" value="2200" class="form-control"></div>
          <div class="input-group"><label class="input-label">% Ajuste (Déficit/Superávit)</label><input type="number" id="kc_ajuste" value="15" class="form-control"></div>
        </div>
        <div id="containerRefeicoes" class="flex flex-col gap-2"></div>
        <div class="flex justify-end gap-2"><button type="button" class="btn btn-outline btnCloseModal">Cancelar</button><button type="submit" class="btn btn-pine">Salvar Plano Kcal</button></div>
      </form>
    </div>

  </div>

  <!-- ============================== LÓGICA JAVASCRIPT ============================== -->
  <script>
    /* 1. CONFIGURAÇÃO DO FIREBASE */
    const firebaseConfig = {
      apiKey: "AIzaSyBvJJktelKoZYoGT2hw2BBk1F3_CfZ5ZIk",
      authDomain: "prontuario-nutri.firebaseapp.com",
      projectId: "prontuario-nutri",
      storageBucket: "prontuario-nutri.firebasestorage.app",
      messagingSenderId: "1058166886511",
      appId: "1:1058166886511:web:1d985c296ee2b6cc5b3d9a"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    /* 2. REGRAS DE CICLO DOS PLANOS */
    const PLAN_CYCLES = {
      "Mensal Start": [
        { period: "Semana 1", formType: "[Inicial / Anamnese]", delivery: "AO_VIVO", label: "Consulta ao Vivo #1 (60 min)", dueDays: 2 },
        { period: "Semana 2/3", formType: "[Intermediário / Check-in]", delivery: "VIDEO", label: "Devolutiva em Vídeo", dueDays: 2 },
        { period: "Semana 4", formType: "[Reavaliação]", delivery: "RENOVA", label: "Fim do ciclo — Renovação", dueDays: 3 }
      ],
      "Trimestral Foco": [
        { period: "Mês 1", formType: "[Inicial / Anamnese]", delivery: "AO_VIVO", label: "Consulta ao Vivo #1", dueDays: 2 },
        { period: "Mês 2", formType: "[Intermediário / Check-in]", delivery: "VIDEO", label: "Devolutiva em Vídeo #1", dueDays: 2 },
        { period: "Mês 3", formType: "[Reavaliação]", delivery: "AO_VIVO", label: "Consulta ao Vivo #2", dueDays: 2 }
      ],
      "Semestral Híbrido": [1, 2, 3, 4, 5, 6].map(m => ({
        period: `Mês ${m}`,
        formType: m % 2 === 1 ? "[Reavaliação]" : "[Intermediário / Check-in]",
        delivery: m % 2 === 1 ? "AO_VIVO" : "VIDEO",
        label: m % 2 === 1 ? `Consulta ao Vivo #${Math.ceil(m/2)}` : `Devolutiva em Vídeo #${m/2}`,
        dueDays: 2
      })),
      "Anual Performance": Array.from({ length: 12 }, (_, i) => i + 1).map(m => ({
        period: `Mês ${m}`,
        formType: m % 2 === 1 ? "[Reavaliação]" : "[Intermediário / Check-in]",
        delivery: m % 2 === 1 ? "AO_VIVO" : "VIDEO",
        label: m % 2 === 1 ? `Consulta ao Vivo #${Math.ceil(m/2)} + Periodização` : `Devolutiva em Vídeo #${m/2}`,
        dueDays: 2
      }))
    };

    /* 3. ESTADO DA APLICAÇÃO */
    let currentNutriEmail = "willianfreitas0054@gmail.com";
    let patients = [];
    let currentPatient = null;
    let chartInstance = null;
    let calendarEvents = [];
    let currentFilter = "all";

    // Estrutura padrão de refeições Kcal
    let defaultRefeicoes = [
      { nome: "Café da Manhã", horario: "07:00", kcal: 350 },
      { nome: "Lanche da Manhã", horario: "10:00", kcal: 180 },
      { nome: "Almoço", horario: "12:30", kcal: 600 },
      { nome: "Lanche da Tarde", horario: "16:00", kcal: 250 },
      { nome: "Jantar", horario: "19:30", kcal: 500 },
      { nome: "Ceia", horario: "22:00", kcal: 120 }
    ];

    /* 4. SINCRONIZAÇÃO FIREBASE */
    async function loadPatients() {
      try {
        const snapshot = await db.collection("nutricionistas").doc(currentNutriEmail).collection("pacientes").get();
        if (!snapshot.empty) {
          patients = snapshot.docs.map(doc => doc.data());
        } else {
          // Dados iniciais de demonstração baseados no seu histórico
          patients = [
            {
              id: "p_arao",
              nome: "Arão Freitas Alves de Lima",
              email: "arao.freitas@hotmail.com",
              whatsapp: "(11) 95880-2837",
              plano: "Trimestral Foco",
              dataInicio: "2026-07-21",
              stageIndex: 1,
              status: "FORM_ANSWERED",
              nextAction: "Gravar Devolutiva em Vídeo (Prazo: 48h)",
              nextDueDate: "2026-08-19",
              anamnese: { objetivo: "Hipertrofia & Rendimento", pesoInicial: 69, treino: "Musculação 5x" },
              quinzenal: [{ data: "2026-08-07", peso: 67.8, adesao: 4, estresse: 7, notas: "Consistente, ajustes no pré-treino." }],
              mensal: [],
              kcalPlans: [{ data: "2026-07-22", kcalMeta: 2400, ajustePct: 10, refeicoes: defaultRefeicoes }]
            },
            {
              id: "p_luiz",
              nome: "Luiz Eduardo Reis Tavares Silva",
              email: "le564705@gmail.com",
              whatsapp: "(11) 94473-8704",
              plano: "Mensal Start",
              dataInicio: "2026-07-21",
              stageIndex: 1,
              status: "FORM_ANSWERED",
              nextAction: "Gravar Devolutiva em Vídeo (Prazo: 48h)",
              nextDueDate: "2026-08-19",
              anamnese: { objetivo: "Hipertrofia & Lutas", pesoInicial: 104.3, treino: "Muay Thai + Jiu Jitsu 5x" },
              quinzenal: [{ data: "2026-08-07", peso: 103, adesao: 5, estresse: 7, notas: "Excelente adesão." }],
              mensal: [],
              kcalPlans: []
            }
          ];
        }
        renderDashboard();
      } catch (err) {
        console.error("Erro ao carregar Firebase:", err);
      }
    }

    async function savePatient(p) {
      try {
        await db.collection("nutricionistas").doc(currentNutriEmail).collection("pacientes").doc(p.id).set(p);
      } catch (e) {
        console.error("Erro salvando paciente:", e);
      }
    }

    /* 5. RENDERIZAÇÃO DO DASHBOARD */
    function renderDashboard() {
      const search = document.getElementById("inputSearch").value.toLowerCase();
      const today = new Date().toISOString().slice(0, 10);

      const filtered = patients.filter(p => {
        const matchSearch = p.nome.toLowerCase().includes(search) || p.email.toLowerCase().includes(search);
        const overdue = p.nextDueDate < today && p.status !== "COMPLETED";
        if (!matchSearch) return false;
        if (currentFilter === "yellow") return p.status === "FORM_ANSWERED";
        if (currentFilter === "red") return overdue;
        if (currentFilter === "green") return p.status === "COMPLETED";
        return true;
      });

      // Métricas
      document.getElementById("metricTotal").innerText = patients.length;
      document.getElementById("metricVideos").innerText = patients.filter(p => p.status === "FORM_ANSWERED" && p.nextAction.includes("Vídeo")).length;
      document.getElementById("metricCalls").innerText = patients.filter(p => p.status === "FORM_ANSWERED" && p.nextAction.includes("Consulta")).length;
      document.getElementById("metricOverdue").innerText = patients.filter(p => p.nextDueDate < today && p.status !== "COMPLETED").length;

      const tbody = document.getElementById("tablePatientsBody");
      tbody.innerHTML = "";

      filtered.forEach(p => {
        const cycle = PLAN_CYCLES[p.plano] || PLAN_CYCLES["Mensal Start"];
        const stage = cycle[p.stageIndex] || cycle[0];
        const overdue = p.nextDueDate < today && p.status !== "COMPLETED";

        const badgeClass = overdue ? "badge-red" : p.status === "FORM_ANSWERED" ? "badge-amber" : p.status === "COMPLETED" ? "badge-green" : "badge-red";
        const badgeLabel = overdue ? "Atrasado" : p.status === "FORM_ANSWERED" ? "Formulário Respondido" : p.status === "COMPLETED" ? "Concluído" : "Pendente Paciente";

        const tr = document.createElement("tr");
        tr.style.borderBottom = "1px solid var(--line)";
        tr.innerHTML = `
          <td style="padding: 12px 14px;">
            <a href="javascript:void(0)" onclick="openDetail('${p.id}')" style="font-weight: 700; color: var(--ink); text-decoration: none;">${p.nome}</a>
            <div style="font-size: 0.75rem; color: var(--inkFaint);">${p.email}</div>
          </td>
          <td style="padding: 12px 14px;">
            <span style="font-size: 0.75rem; font-weight: 600; color: var(--pine);">${p.plano}</span>
            <div style="font-size: 0.75rem; color: var(--inkSoft);">${stage.period}</div>
          </td>
          <td style="padding: 12px 14px; font-size: 0.75rem; color: var(--inkSoft);">${p.dataInicio.split('-').reverse().join('/')}</td>
          <td style="padding: 12px 14px; font-size: 0.75rem; color: var(--inkSoft); font-weight: 500;">${stage.formType}</td>
          <td style="padding: 12px 14px;"><span class="badge ${badgeClass}"><span class="badge-dot"></span>${badgeLabel}</span></td>
          <td style="padding: 12px 14px; font-size: 0.75rem; color: var(--inkSoft); max-width: 200px;">${p.nextAction}</td>
          <td style="padding: 12px 14px; font-size: 0.75rem; font-weight: 600; color: ${overdue ? 'var(--red)' : 'var(--inkSoft)'};">${p.nextDueDate.split('-').reverse().join('/')}</td>
          <td style="padding: 12px 14px; text-align: center;">
            <input type="checkbox" ${p.status === 'COMPLETED' ? 'checked disabled' : ''} onchange="handleAdvanceStage('${p.id}')" style="width: 18px; height: 18px; cursor: pointer;">
          </td>
          <td style="padding: 12px 14px; text-align: right;">
            <button class="btn btn-outline" style="padding: 4px 8px; font-size: 0.75rem;" onclick="openDetail('${p.id}')">Prontuário →</button>
          </td>
        `;
        tbody.appendChild(tr);
      });

      renderCalendarFeed();
    }

    /* 6. AVANÇO DE ETAPAS & CALENDAR */
    async function handleAdvanceStage(id) {
      const p = patients.find(x => x.id === id);
      if (!p) return;

      const cycle = PLAN_CYCLES[p.plano] || PLAN_CYCLES["Mensal Start"];
      if (p.stageIndex + 1 < cycle.length) {
        p.stageIndex += 1;
        const nextStage = cycle[p.stageIndex];
        const nextDate = new Date();
        nextDate.setDate(nextDate.getDate() + nextStage.dueDays);
        
        p.status = "PENDING_PATIENT";
        p.nextAction = `Aguardando envio: ${nextStage.formType}`;
        p.nextDueDate = nextDate.toISOString().slice(0, 10);

        calendarEvents.unshift({
          date: p.nextDueDate,
          title: `📤 Enviar ${nextStage.formType} — ${p.nome}`
        });
      } else {
        p.status = "COMPLETED";
        p.nextAction = "Plano Finalizado — Oferecer Renovação";
      }

      await savePatient(p);
      renderDashboard();
    }

    function renderCalendarFeed() {
      const cont = document.getElementById("calendarEventsList");
      cont.innerHTML = "";
      if (calendarEvents.length === 0) {
        calendarEvents = [
          { date: "2026-08-19", title: "📹 Gravar Devolutiva em Vídeo — Arão Freitas" },
          { date: "2026-08-19", title: "📹 Gravar Devolutiva em Vídeo — Luiz Eduardo" }
        ];
      }
      calendarEvents.slice(0, 4).forEach(e => {
        const item = document.createElement("div");
        item.style.cssText = "background: var(--paperCard); border: 1px solid var(--line); padding: 8px 12px; border-radius: 8px; font-size: 0.8rem; display: flex; gap: 8px; align-items: center;";
        item.innerHTML = `<span>🗓️</span> <b>${e.date.split('-').reverse().join('/')}</b> — <span>${e.title}</span>`;
        cont.appendChild(item);
      });
    }

    /* 7. PRONTUÁRIO DETALHADO (VIEW DETAIL) */
    function openDetail(id) {
      currentPatient = patients.find(x => x.id === id);
      if (!currentPatient) return;

      document.getElementById("viewDashboard").classList.add("hidden");
      document.getElementById("viewDetail").classList.remove("hidden");

      document.getElementById("detNome").innerText = currentPatient.nome;
      document.getElementById("detSub").innerText = `${currentPatient.email} · ${currentPatient.whatsapp}`;
      document.getElementById("detPlanoBadge").innerText = currentPatient.plano;
      document.getElementById("btnWhatsDetail").href = `https://wa.me/55${currentPatient.whatsapp.replace(/\D/g, '')}`;

      const cycle = PLAN_CYCLES[currentPatient.plano] || PLAN_CYCLES["Mensal Start"];
      document.getElementById("detStageText").innerText = `${cycle[currentPatient.stageIndex]?.period || 'Final'} de ${cycle.length}`;

      // Preenche dados da Anamnese
      const a = currentPatient.anamnese || {};
      const dDados = document.getElementById("infoDados"); dDados.innerHTML = "";
      addInfo(dDados, "Nome", currentPatient.nome);
      addInfo(dDados, "E-mail", currentPatient.email);
      addInfo(dDados, "WhatsApp", currentPatient.whatsapp);
      addInfo(dDados, "Início", currentPatient.dataInicio.split('-').reverse().join('/'));

      const dSaude = document.getElementById("infoSaude"); dSaude.innerHTML = "";
      addInfo(dSaude, "Objetivo Principal", a.objetivo);
      addInfo(dSaude, "Peso Inicial", a.pesoInicial ? `${a.pesoInicial} kg` : "—");
      addInfo(dSaude, "Restrições", a.restricoes || "Nenhuma");

      const dTreino = document.getElementById("infoTreino"); dTreino.innerHTML = "";
      addInfo(dTreino, "Modalidade & Rotina", a.treino || "Musculação");

      const dAlim = document.getElementById("infoAlimentacao"); dAlim.innerHTML = "";
      addInfo(dAlim, "Preferências", a.preferencias || "Flexível / Marmitas");

      renderAcompanhamentosView();
      renderKcalView();
      setupComparador();
    }

    function addInfo(container, label, val) {
      const row = document.createElement("div");
      row.className = "info-row";
      row.innerHTML = `<span class="info-label">${label}</span><span class="info-value">${val || "—"}</span>`;
      container.appendChild(row);
    }

    function renderAcompanhamentosView() {
      const qList = document.getElementById("listQuinzenal"); qList.innerHTML = "";
      (currentPatient.quinzenal || []).forEach(q => {
        const card = document.createElement("div");
        card.className = "card";
        card.innerHTML = `
          <div class="flex justify-between items-center" style="margin-bottom: 4px;">
            <b style="font-size: 0.8rem; color: var(--inkSoft);">${q.data.split('-').reverse().join('/')}</b>
            <span class="badge badge-green">${q.peso} kg</span>
          </div>
          <p style="font-size: 0.75rem; color: var(--ink);"><b>Adesão:</b> ${q.adesao || 4}/5 · <b>Estresse:</b> ${q.estresse || 5}/10</p>
          <p style="font-size: 0.75rem; color: var(--inkSoft); margin-top: 4px;">${q.notas || ''}</p>
        `;
        qList.appendChild(card);
      });

      const mList = document.getElementById("listMensal"); mList.innerHTML = "";
      (currentPatient.mensal || []).forEach(m => {
        const card = document.createElement("div");
        card.className = "card";
        card.innerHTML = `
          <div class="flex justify-between items-center" style="margin-bottom: 4px;">
            <b style="font-size: 0.8rem; color: var(--amberDark);">${m.data.split('-').reverse().join('/')}</b>
            <span class="badge badge-amber">${m.peso} kg</span>
          </div>
          <p style="font-size: 0.75rem; color: var(--ink);"><b>Evolução:</b> ${m.evolucao || 'Estável'}</p>
          <p style="font-size: 0.75rem; color: var(--inkSoft); margin-top: 4px;"><b>Meta:</b> ${m.meta || ''}</p>
        `;
        mList.appendChild(card);
      });

      // Atualiza peso no topo
      const todosPesos = [...(currentPatient.quinzenal || []), ...(currentPatient.mensal || [])].sort((a,b) => new Date(a.data) - new Date(b.data));
      if (todosPesos.length > 0) {
        const ult = todosPesos[todosPesos.length - 1];
        document.getElementById("detUltimoPeso").innerText = `${ult.peso} kg`;
        document.getElementById("detUltimaData").innerText = ult.data.split('-').reverse().join('/');
      }
    }

    function renderKcalView() {
      const cont = document.getElementById("listKcalPlans"); cont.innerHTML = "";
      (currentPatient.kcalPlans || []).forEach(k => {
        const card = document.createElement("div");
        card.className = "card";
        const desc = k.kcalMeta * (k.ajustePct / 100);
        const liquida = k.kcalMeta - desc;
        card.innerHTML = `
          <div class="flex justify-between items-center" style="margin-bottom: 8px;">
            <b style="font-family: var(--font-display);">Plano Kcal — ${k.data.split('-').reverse().join('/')}</b>
            <span class="badge badge-green">Líquida: ${Math.round(liquida)} kcal</span>
          </div>
          <div class="grid grid-cols-3 gap-2" style="background: var(--pineTint); padding: 8px; border-radius: 8px; font-size: 0.75rem; margin-bottom: 8px;">
            <div>Meta: <b>${k.kcalMeta} kcal</b></div>
            <div>Ajuste: <b>${k.ajustePct}%</b></div>
            <div>Déficit: <b>-${Math.round(desc)} kcal</b></div>
          </div>
          <table style="width: 100%; font-size: 0.75rem; border-collapse: collapse;">
            ${(k.refeicoes || defaultRefeicoes).map(r => `
              <tr style="border-top: 1px solid var(--line);">
                <td style="padding: 4px 0;"><b>${r.nome}</b> (${r.horario})</td>
                <td style="padding: 4px 0; text-align: right; font-family: var(--font-mono);">${r.kcal} kcal</td>
              </tr>
            `).join('')}
          </table>
        `;
        cont.appendChild(card);
      });
    }

    function setupComparador() {
      const evs = [];
      if (currentPatient.anamnese?.pesoInicial) {
        evs.push({ label: `Início (${currentPatient.dataInicio.split('-').reverse().join('/')})`, peso: currentPatient.anamnese.pesoInicial });
      }
      (currentPatient.quinzenal || []).forEach(q => evs.push({ label: `Quinzenal (${q.data.split('-').reverse().join('/')})`, peso: q.peso }));
      (currentPatient.mensal || []).forEach(m => evs.push({ label: `Mensal (${m.data.split('-').reverse().join('/')})`, peso: m.peso }));

      // Renderiza Gráfico
      const ctx = document.getElementById("chartEvolucao").getContext("2d");
      if (chartInstance) chartInstance.destroy();
      chartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels: evs.map(e => e.label),
          datasets: [{
            label: 'Peso Corporal (kg)',
            data: evs.map(e => e.peso),
            borderColor: '#1F6F5C',
            backgroundColor: 'rgba(31, 111, 92, 0.1)',
            borderWidth: 3,
            fill: true,
            tension: 0.2
          }]
        },
        options: { responsive: true, maintainAspectRatio: false }
      });
    }

    /* 8. EVENTOS DE INTERFACE & MODAIS */
    document.addEventListener("DOMContentLoaded", () => {
      loadPatients();

      document.getElementById("btnBackToDashboard").addEventListener("click", () => {
        document.getElementById("viewDetail").classList.add("hidden");
        document.getElementById("viewDashboard").classList.remove("hidden");
        renderDashboard();
      });

      // Filtros
      document.querySelectorAll("#filterPills button").forEach(btn => {
        btn.addEventListener("click", (e) => {
          document.querySelectorAll("#filterPills button").forEach(b => { b.className = "btn btn-outline"; });
          btn.className = "btn btn-pine";
          currentFilter = btn.dataset.filter;
          renderDashboard();
        });
      });

      document.getElementById("inputSearch").addEventListener("input", renderDashboard);

      // Controle de Abas do Prontuário
      document.querySelectorAll(".tab-btn").forEach(btn => {
        btn.addEventListener("click", (e) => {
          document.querySelectorAll(".tab-btn").forEach(b => b.classList.remove("active"));
          document.querySelectorAll(".tab-pane").forEach(p => p.classList.add("hidden"));
          btn.classList.add("active");
          document.getElementById(btn.dataset.tab).classList.remove("hidden");
        });
      });

      // Abertura de Modais
      document.getElementById("btnNewPatient").addEventListener("click", () => openModal("modalAnamnese"));
      document.getElementById("btnModalQuinzenal").addEventListener("click", () => openModal("modalQuinzenal"));
      document.getElementById("btnModalMensal").addEventListener("click", () => openModal("modalMensal"));
      document.getElementById("btnModalKcal").addEventListener("click", () => openModal("modalKcal"));

      document.querySelectorAll(".btnCloseModal").forEach(b => b.addEventListener("click", closeModal));

      // Submit: Anamnese
      document.getElementById("formAnamnese").addEventListener("submit", async (e) => {
        e.preventDefault();
        const novo = {
          id: "p_" + Date.now(),
          nome: document.getElementById("ana_nome").value,
          email: document.getElementById("ana_email").value,
          whatsapp: document.getElementById("ana_whatsapp").value,
          plano: document.getElementById("ana_plano").value,
          dataInicio: document.getElementById("ana_dataInicio").value,
          stageIndex: 0,
          status: "FORM_ANSWERED",
          nextAction: "Agendar Consulta ao Vivo #1",
          nextDueDate: document.getElementById("ana_dataInicio").value,
          anamnese: {
            objetivo: document.getElementById("ana_objetivo").value,
            pesoInicial: Number(document.getElementById("ana_pesoInicial").value),
            treino: document.getElementById("ana_treino").value,
            restricoes: document.getElementById("ana_restricoes").value
          },
          quinzenal: [], mensal: [], kcalPlans: []
        };
        patients.unshift(novo);
        await savePatient(novo);
        closeModal();
        renderDashboard();
      });

      // Submit: Quinzenal
      document.getElementById("formQuinzenal").addEventListener("submit", async (e) => {
        e.preventDefault();
        if (!currentPatient) return;
        currentPatient.quinzenal = currentPatient.quinzenal || [];
        currentPatient.quinzenal.push({
          data: document.getElementById("qui_data").value,
          peso: Number(document.getElementById("qui_peso").value),
          adesao: Number(document.getElementById("qui_adesao").value),
          estresse: Number(document.getElementById("qui_estresse").value),
          notas: document.getElementById("qui_notas").value
        });
        currentPatient.status = "FORM_ANSWERED";
        currentPatient.nextAction = "Gravar Devolutiva em Vídeo (Prazo: 48h)";
        await savePatient(currentPatient);
        closeModal();
        openDetail(currentPatient.id);
      });

      // Submit: Mensal
      document.getElementById("formMensal").addEventListener("submit", async (e) => {
        e.preventDefault();
        if (!currentPatient) return;
        currentPatient.mensal = currentPatient.mensal || [];
        currentPatient.mensal.push({
          data: document.getElementById("men_data").value,
          peso: Number(document.getElementById("men_peso").value),
          evolucao: document.getElementById("men_evolucao").value,
          meta: document.getElementById("men_meta").value
        });
        currentPatient.status = "FORM_ANSWERED";
        currentPatient.nextAction = "Agendar Consulta de Reavaliação";
        await savePatient(currentPatient);
        closeModal();
        openDetail(currentPatient.id);
      });
    });

    function openModal(id) {
      document.getElementById("modalContainer").classList.remove("hidden");
      document.querySelectorAll(".modal-content").forEach(m => m.classList.add("hidden"));
      document.getElementById(id).classList.remove("hidden");
    }

    function closeModal() {
      document.getElementById("modalContainer").classList.add("hidden");
    }
  </script>
</body>
</html>
