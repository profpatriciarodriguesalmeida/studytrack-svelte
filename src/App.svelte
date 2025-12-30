<script>
  // Disciplinas iniciais
  let newSubjectName = "";
  let subjects = [
    "Português",
    "Inglês",
    "História e Geografia de Portugal",
    "Cidadania e Desenvolvimento",
    "Matemática",
    "Ciências Naturais",
    "Educação Visual",
    "Educação Tecnológica",
    "Educação Musical",
    "TIC",
    "Educação Física",
    "Educação Moral e Religiosa"
  ];

  // Tabela de resultados: para cada disciplina, 10 avaliações (A1..A10)
  function initGradeTable(subjectList) {
    const obj = {};
    for (const s of subjectList) {
      obj[s] = {
        A1: "",
        A2: "",
        A3: "",
        A4: "",
        A5: "",
        A6: "",
        A7: "",
        A8: "",
        A9: "",
        A10: ""
      };
    }
    return obj;
  }

  let gradeTable = initGradeTable(subjects);

  // Formulário de tarefas
  let subject = subjects[0];
  let type = "Teste";
  let date = "";
  let description = "";

  let tasks = [];
  let nextTaskId = 1;

  function addSubject() {
    const trimmed = newSubjectName.trim();
    if (!trimmed) {
      alert("Escreve o nome da disciplina.");
      return;
    }
    if (subjects.includes(trimmed)) {
      alert("Essa disciplina já existe na lista.");
      return;
    }
    subjects = [...subjects, trimmed];

    // adicionar disciplina à tabela de resultados
    gradeTable = {
      ...gradeTable,
      [trimmed]: {
        A1: "",
        A2: "",
        A3: "",
        A4: "",
        A5: "",
        A6: "",
        A7: "",
        A8: "",
        A9: "",
        A10: ""
      }
    };

    newSubjectName = "";
    if (!subject) {
      subject = trimmed;
    }
  }

  function removeSubject(name) {
    if (subjects.length === 1) {
      alert("Tem de existir pelo menos uma disciplina.");
      return;
    }

    subjects = subjects.filter((s) => s !== name);

    const updatedTable = { ...gradeTable };
    delete updatedTable[name];
    gradeTable = updatedTable;

    if (subject === name) {
      subject = subjects[0] ?? "";
    }
  }

  function addTask() {
    if (!subject || !date) {
      alert("Escolhe a disciplina e a data.");
      return;
    }

    tasks = [
      ...tasks,
      {
        id: nextTaskId++,
        subject,
        type,
        date,
        description,
        done: false,
        tempGrade: "",
        gradeSlotKey: null,
        gradeSlotSubject: null
      }
    ];

    type = "Teste";
    date = "";
    description = "";
  }

  function removeTask(id) {
    tasks = tasks.filter((task) => task.id !== id);
  }

  function markAsDone(taskId) {
    tasks = tasks.map((task) => {
      if (task.id === taskId) {
        return { ...task, done: true };
      }
      return task;
    });
  }

  // Guardar / editar nota
  function saveGrade(taskId) {
    const task = tasks.find((t) => t.id === taskId);
    if (!task) return;
    if (!task.tempGrade) {
      alert("Escreve a nota antes de guardar.");
      return;
    }

    const subj = task.subject;

    // garantir disciplina na tabela
    if (!gradeTable[subj]) {
      gradeTable = {
        ...gradeTable,
        [subj]: {
          A1: "",
          A2: "",
          A3: "",
          A4: "",
          A5: "",
          A6: "",
          A7: "",
          A8: "",
          A9: "",
          A10: ""
        }
      };
    }

    // Já tem slot => EDITAR nota
    if (task.gradeSlotKey && task.gradeSlotSubject) {
      const entry = { ...gradeTable[task.gradeSlotSubject] };
      entry[task.gradeSlotKey] = task.tempGrade; // substitui a nota
      gradeTable = {
        ...gradeTable,
        [task.gradeSlotSubject]: entry
      };
      return;
    }

    // Ainda não tem slot => GUARDAR pela primeira vez
    const entry = { ...gradeTable[subj] };
    const keys = Object.keys(entry); // A1..A10
    let placedKey = null;

    for (const key of keys) {
      if (!entry[key]) {
        entry[key] = task.tempGrade; // só a nota
        placedKey = key;
        break;
      }
    }

    if (!placedKey) {
      alert("Já usaste as 10 colunas de avaliações para esta disciplina.");
      return;
    }

    gradeTable = {
      ...gradeTable,
      [subj]: entry
    };

    // guardar em que slot ficou a avaliação
    tasks = tasks.map((t) =>
      t.id === taskId
        ? {
            ...t,
            gradeSlotKey: placedKey,
            gradeSlotSubject: subj
          }
        : t
    );
  }

  // Apagar avaliação
  function deleteGrade(taskId) {
    const task = tasks.find((t) => t.id === taskId);
    if (!task) return;

    // Se não há slot, limpa só o campo local
    if (!task.gradeSlotKey || !task.gradeSlotSubject) {
      tasks = tasks.map((t) =>
        t.id === taskId
          ? { ...t, tempGrade: "" }
          : t
      );
      return;
    }

    const subj = task.gradeSlotSubject;
    const key = task.gradeSlotKey;

    if (gradeTable[subj]) {
      const entry = { ...gradeTable[subj] };
      entry[key] = ""; // limpa a célula
      gradeTable = {
        ...gradeTable,
        [subj]: entry
      };
    }

    // limpar info na tarefa
    tasks = tasks.map((t) =>
      t.id === taskId
        ? {
            ...t,
            tempGrade: "",
            gradeSlotKey: null,
            gradeSlotSubject: null
          }
        : t
    );
  }

  function formatDate(value) {
    if (!value) return "";
    const [y, m, d] = value.split("-");
    return `${d}/${m}`;
  }

  // Tabs: pendentes / realizados
  let activeTab = "pendentes";

  $: pendingTasks = tasks
    .filter((t) => !t.done)
    .sort((a, b) => (a.date < b.date ? -1 : a.date > b.date ? 1 : 0));

  $: doneTasks = tasks
    .filter((t) => t.done)
    .sort((a, b) => (a.date < b.date ? -1 : a.date > b.date ? 1 : 0));

  const gradeColumns = ["A1","A2","A3","A4","A5","A6","A7","A8","A9","A10"];
</script>

<svelte:head>
  <title>Gestão da Avaliação — Organiza os teus testes</title>
  <meta
    name="description"
    content="Landing page para alunos do 2.º ciclo registarem testes, trabalhos e resultados por disciplina."
  />
</svelte:head>

<main class="page">
  <!-- HEADER -->
  <header class="header">
    <div class="logo">Gestão da Avaliação</div>
    <nav class="nav">
      <a href="#como-funciona">Como funciona</a>
      <a href="#registo">Registar</a>
      <a href="#lista">Testes e trabalhos</a>
      <a href="#resultados">Resultados</a>
      <a href="#disciplinas">Disciplinas</a>
    </nav>
  </header>

  <!-- HERO -->
  <section class="hero" id="topo">
    <div class="hero-text">
      <h1>Não te esqueças dos testes e trabalhos.</h1>
      <p>
        Esta página ajuda‑te a apontar o que tens para fazer, a ver os próximos testes
        e a registar as notas de cada disciplina.
      </p>
      <p class="hero-note">
        Pensado para alunos do 2.º ciclo que querem organizar melhor o seu estudo.
      </p>
      <a class="btn-primary" href="#registo">Começar a registar</a>
    </div>
  </section>

  <!-- COMO FUNCIONA -->
  <section class="section" id="como-funciona">
    <h2>Como funciona?</h2>
    <div class="steps">
      <article class="step">
        <h3>1. Regista testes e trabalhos</h3>
        <p>
          Sempre que o professor marcar algo, regista a disciplina, o tipo e a data para não te esqueceres.
        </p>
      </article>
      <article class="step">
        <h3>2. Marca o que já fizeste</h3>
        <p>
          Quando terminares, usa o botão “Já fiz” para marcar o trabalho como concluído e apontar a nota.
        </p>
      </article>
      <article class="step">
        <h3>3. Acompanha as notas por disciplina</h3>
        <p>
          A tabela de resultados mostra, por disciplina, as avaliações que já fizeste ao longo do ano.
        </p>
      </article>
    </div>
  </section>

  <!-- REGISTAR TESTES E TRABALHOS -->
  <section class="section section-alt" id="registo">
    <h2>Registar testes e trabalhos</h2>
    <p class="section-intro">
      Preenche este formulário sempre que tiveres um novo teste, trabalho ou questão‑aula.
    </p>

    <form class="form" on:submit|preventDefault={addTask}>
      <div class="form-group">
        <label for="subject">Disciplina</label>
        <select id="subject" bind:value={subject}>
          {#each subjects as s}
            <option value={s}>{s}</option>
          {/each}
        </select>
      </div>

      <div class="form-group">
        <label for="type">Tipo</label>
        <select id="type" bind:value={type}>
          <option>Teste</option>
          <option>Trabalho</option>
          <option>Questão‑Aula</option>
          <option>Outro</option>
        </select>
      </div>

      <div class="form-group">
        <label for="date">Data</label>
        <input id="date" type="date" bind:value={date} required />
      </div>

      <div class="form-group form-description">
        <label for="description">Descrição (opcional)</label>
        <textarea
          id="description"
          rows="2"
          bind:value={description}
          placeholder="Ex.: Capítulo 3 do manual, páginas 45–52"
        ></textarea>
      </div>

      <div class="form-group form-button">
        <label>&nbsp;</label>
        <button type="submit" class="btn-primary">Adicionar à lista</button>
      </div>
    </form>
  </section>

  <!-- TESTES E TRABALHOS (TABS) -->
  <section class="section" id="lista">
    <h2>Testes e trabalhos</h2>

    <div class="tabs">
      <button
        type="button"
        class:selected-tab={activeTab === "pendentes"}
        on:click={() => (activeTab = "pendentes")}
      >
        Pendentes
      </button>
      <button
        type="button"
        class:selected-tab={activeTab === "realizados"}
        on:click={() => (activeTab = "realizados")}
      >
        Realizados
      </button>
    </div>

    {#if activeTab === "pendentes"}
      <h3>Próximos testes e trabalhos</h3>
      {#if pendingTasks.length === 0}
        <p>Não tens tarefas pendentes neste momento.</p>
      {:else}
        <ul class="task-list">
          {#each pendingTasks as task}
            <li class="task-item">
              <div class="task-main">
                <span class="task-subject">{task.subject}</span>
                <span class="task-type">{task.type}</span>
                <span class="task-date">{formatDate(task.date)}</span>
              </div>
              {#if task.description}
                <p class="task-description">{task.description}</p>
              {/if}
              <div class="task-actions">
                <button class="btn-remove" on:click={() => removeTask(task.id)}>
                  Apagar
                </button>
                <button class="btn-secondary" on:click={() => markAsDone(task.id)}>
                  Já fiz
                </button>
              </div>
            </li>
          {/each}
        </ul>
      {/if}
    {:else}
      <h3>Testes e trabalhos realizados</h3>
      {#if doneTasks.length === 0}
        <p>Ainda não marcaste nenhum teste ou trabalho como realizado.</p>
      {:else}
        <ul class="task-list">
          {#each doneTasks as task}
            <li class="task-item">
              <div class="task-main">
                <span class="task-subject">{task.subject}</span>
                <span class="task-type">
                  {task.type} (concluído)
                </span>
                <span class="task-date">{formatDate(task.date)}</span>
              </div>
              {#if task.description}
                <p class="task-description">{task.description}</p>
              {/if}
              <div class="task-grade">
                <label>
                  Nota:
                  <input
                    type="text"
                    bind:value={task.tempGrade}
                    placeholder="Ex.: 4, 80%, 16 valores"
                  />
                </label>

                {#if task.gradeSlotKey}
                  <!-- Já existe avaliação: EDITAR / APAGAR -->
                  <button
                    class="btn-secondary-small"
                    on:click={() => saveGrade(task.id)}
                  >
                    Editar nota
                  </button>
                  <button
                    class="btn-remove"
                    on:click={() => deleteGrade(task.id)}
                  >
                    Apagar avaliação
                  </button>
                {:else}
                  <!-- Ainda não existe avaliação: GUARDAR -->
                  <button
                    class="btn-secondary-small"
                    on:click={() => saveGrade(task.id)}
                  >
                    Guardar nota
                  </button>
                {/if}
              </div>
              {#if task.tempGrade}
                <p class="task-grade-view">
                  Nota registada: <strong>{task.tempGrade}</strong>
                </p>
              {/if}
            </li>
          {/each}
        </ul>
      {/if}
    {/if}
  </section>

  <!-- RESULTADOS POR DISCIPLINA -->
  <section class="section" id="resultados">
    <h2>Resultados por disciplina</h2>
    <p class="section-intro">
      Esta tabela mostra, para cada disciplina, até dez avaliações (testes, trabalhos ou questões‑aula)
      que foste registando.
    </p>

    <div class="results-table-wrapper">
      <table class="results-table">
        <thead>
          <tr>
            <th>Disciplina</th>
            {#each gradeColumns as col, i}
              <th>A{ i + 1 }</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each subjects as s}
            <tr>
              <td>{s}</td>
              {#each gradeColumns as col}
                <td>{gradeTable[s]?.[col]}</td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </section>

  <!-- GESTÃO DE DISCIPLINAS -->
  <section class="section" id="disciplinas">
    <h2>Gerir as Disciplinas</h2>
    <p class="section-intro">
      Aqui podes ver e ajustar as disciplinas que usas. Se a tua turma tiver outra disciplina,
      podes adicioná‑la à lista.
    </p>

    <form class="subject-form" on:submit|preventDefault={addSubject}>
      <div class="subject-input-group">
        <label for="newSubject">Adicionar disciplina</label>
        <input
          id="newSubject"
          type="text"
          bind:value={newSubjectName}
          placeholder="Ex.: Clube de Ciências"
        />
      </div>
      <button type="submit" class="btn-secondary">Adicionar</button>
    </form>

    <ul class="subject-list">
      {#each subjects as s}
        <li>
          <span>{s}</span>
          <button
            type="button"
            class="btn-circle-x"
            on:click={() => removeSubject(s)}
            aria-label="Remover disciplina"
          >
            ×
          </button>
        </li>
      {/each}
    </ul>
  </section>

  <!-- FOOTER -->
  <footer class="footer">
    <p>Gestão da Avaliação · Página de demonstração para alunos do 2.º ciclo</p>
  </footer>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    color: #111827;
    background: #f9fafb;
  }

  .page {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  .header {
    position: sticky;
    top: 0;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 5vw;
    background: rgba(249, 250, 251, 0.94);
    backdrop-filter: blur(8px);
    border-bottom: 1px solid #e5e7eb;
  }

  .logo {
    font-weight: 700;
    font-size: 1.2rem;
    color: #2563eb;
  }

  .nav {
    display: flex;
    gap: 1rem;
    align-items: center;
  }

  .nav a {
    text-decoration: none;
    font-size: 0.95rem;
    color: #374151;
  }

  .nav a:hover {
    color: #111827;
  }

  .hero {
    display: block;
    padding: 3rem 5vw;
  }

  .hero-text h1 {
    font-size: clamp(2rem, 3vw, 2.6rem);
    margin-bottom: 1rem;
  }

  .hero-text p {
    font-size: 1rem;
    line-height: 1.5;
  }

  .hero-note {
    margin-top: 0.75rem;
    font-size: 0.9rem;
    color: #6b7280;
  }

  .btn-primary {
    display: inline-block;
    margin-top: 1.5rem;
    padding: 0.75rem 1.5rem;
    border-radius: 999px;
    border: none;
    cursor: pointer;
    background: #2563eb;
    color: white;
    font-weight: 600;
    font-size: 0.95rem;
    text-decoration: none;
    text-align: center;
  }

  .btn-primary:hover {
    background: #1d4ed8;
  }

  .btn-secondary {
    padding: 0.6rem 1.2rem;
    border-radius: 999px;
    border: none;
    cursor: pointer;
    background: #10b981;
    color: white;
    font-weight: 600;
    font-size: 0.9rem;
  }

  .btn-secondary:hover {
    background: #059669;
  }

  .btn-secondary-small {
    padding: 0.4rem 0.9rem;
    border-radius: 999px;
    border: none;
    cursor: pointer;
    background: #10b981;
    color: white;
    font-weight: 500;
    font-size: 0.85rem;
  }

  .section {
    padding: 3rem 5vw;
  }

  .section-alt {
    background: #eef2ff;
  }

  .section h2 {
    font-size: 1.5rem;
    margin-bottom: 1rem;
  }

  .section-intro {
    max-width: 40rem;
    font-size: 1rem;
    color: #4b5563;
    margin: 0 auto;
    text-align: center;
  }

  /* Formulário */
  .form {
    margin-top: 2rem;
    display: grid;
    gap: 1rem;
  }

  @media (min-width: 768px) {
    .form {
      grid-template-columns: repeat(3, minmax(0, 1fr));
      align-items: end;
    }
  }

  .form-group {
    display: flex;
    flex-direction: column;
    gap: 0.3rem;
  }

  @media (min-width: 768px) {
    .form-description {
      grid-column: 1 / 3;
    }

    .form-button {
      grid-column: 3 / 4;
    }
  }

  label {
    font-size: 0.9rem;
    color: #374151;
  }

  input,
  select,
  textarea {
    border-radius: 0.5rem;
    border: 1px solid #d1d5db;
    padding: 0.5rem 0.75rem;
    font-size: 0.95rem;
    font-family: inherit;
  }

  input:focus,
  select:focus,
  textarea:focus {
    outline: 2px solid #2563eb;
    outline-offset: 1px;
    border-color: #2563eb;
  }

  /* Tabs */
  .tabs {
    display: inline-flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
    border-radius: 999px;
    padding: 0.2rem;
    background: #e5e7eb;
  }

  .tabs button {
    border: none;
    background: transparent;
    padding: 0.4rem 0.9rem;
    border-radius: 999px;
    font-size: 0.9rem;
    cursor: pointer;
    color: #4b5563;
  }

  .tabs button.selected-tab {
    background: white;
    color: #111827;
    font-weight: 600;
  }

  /* Como funciona */
  .steps {
    margin-top: 2rem;
    display: grid;
    gap: 1.5rem;
  }

  @media (min-width: 768px) {
    .steps {
      grid-template-columns: repeat(3, minmax(0, 1fr));
    }
  }

  .step {
    background: white;
    border-radius: 0.75rem;
    padding: 1.25rem;
    box-shadow: 0 8px 20px rgba(15, 23, 42, 0.04);
  }

  .step h3 {
    font-size: 1.05rem;
    margin-bottom: 0.5rem;
  }

  /* Disciplinas */
  .subject-form {
    margin-top: 1.5rem;
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    align-items: flex-end;
  }

  .subject-input-group {
    display: flex;
    flex-direction: column;
    gap: 0.3rem;
    min-width: min(260px, 100%);
  }

  .subject-list {
    margin-top: 1.5rem;
    list-style: none;
    padding: 0;
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .subject-list li {
    padding: 0.35rem 0.75rem;
    border-radius: 999px;
    background: white;
    border: 1px solid #e5e7eb;
    font-size: 0.9rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .btn-circle-x {
    width: 22px;
    height: 22px;
    border-radius: 999px;
    border: none;
    background: #ef4444;
    color: white;
    font-size: 0.9rem;
    line-height: 1;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    padding: 0;
  }

  .btn-circle-x:hover {
    background: #dc2626;
  }

  /* Lista de tarefas */
  .task-list {
    margin-top: 1.5rem;
    list-style: none;
    padding: 0;
  }

  .task-item {
    background: white;
    border-radius: 0.75rem;
    padding: 1rem 1.25rem;
    box-shadow: 0 8px 20px rgba(15, 23, 42, 0.04);
    margin-bottom: 1rem;
  }

  .task-main {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem 1rem;
    align-items: baseline;
  }

  .task-subject {
    font-weight: 600;
  }

  .task-type {
    font-size: 0.9rem;
    color: #4b5563;
  }

  .task-date {
    margin-left: auto;
    font-weight: 500;
    color: #2563eb;
  }

  .task-description {
    margin: 0.5rem 0 0.75rem;
    font-size: 0.9rem;
    color: #4b5563;
  }

  .task-actions {
    margin-top: 0.75rem;
    display: flex;
    gap: 0.5rem;
  }

  .task-grade {
    margin-top: 0.75rem;
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    align-items: center;
  }

  .task-grade input {
    max-width: 180px;
  }

  .task-grade-view {
    margin-top: 0.3rem;
    font-size: 0.9rem;
    color: #111827;
  }

  .btn-remove {
    border: none;
    background: #ef4444;
    color: white;
    padding: 0.4rem 0.75rem;
    border-radius: 999px;
    font-size: 0.85rem;
    cursor: pointer;
  }

  .btn-remove:hover {
    background: #dc2626;
  }

  /* Resultados */
  .results-table-wrapper {
    margin-top: 1.5rem;
    overflow-x: auto;
  }

  .results-table {
    width: 100%;
    border-collapse: collapse;
    background: white;
    border-radius: 0.75rem;
    overflow: hidden;
  }

  .results-table th,
  .results-table td {
    padding: 0.6rem 0.75rem;
    border-bottom: 1px solid #e5e7eb;
    font-size: 0.9rem;
    text-align: left;
    white-space: nowrap;
  }

  .results-table th {
    background: #f3f4f6;
    font-weight: 600;
  }

  .footer {
    margin-top: auto;
    padding: 1.5rem 5vw;
    text-align: center;
    font-size: 0.85rem;
    color: #6b7280;
    border-top: 1px solid #e5e7eb;
    background: #f9fafb;
  }

  @media (max-width: 768px) {
    .nav {
      display: none;
    }
  }
</style>
