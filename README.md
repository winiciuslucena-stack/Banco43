<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MedQuest | Plataforma de Questões e Caderno de Erros</title>
  <!-- Tailwind CSS via CDN para design moderno e responsivo -->
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
  <style>
    @media print {
      body * { visibility: hidden; }
      #printArea, #printArea * { visibility: visible; }
      #printArea { position: absolute; left: 0; top: 0; width: 100%; }
      .no-print { display: none !important; }
    }
    .custom-scroll::-webkit-scrollbar { width: 6px; }
    .custom-scroll::-webkit-scrollbar-track { background: #f1f5f9; }
    .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
  </style>
</head>
<body class="bg-slate-50 text-slate-800 font-sans antialiased min-h-screen flex flex-col">

  <!-- NAVBAR -->
  <header class="bg-indigo-900 text-white shadow-md sticky top-0 z-50">
    <div class="max-w-7xl mx-auto px-4 h-16 flex items-center justify-between">
      <div class="flex items-center gap-3 cursor-pointer" onclick="navegarPara('dashboard')">
        <i class="fa-solid fa-stethoscope text-2xl text-emerald-400"></i>
        <div>
          <h1 class="text-lg font-bold tracking-tight">MedQuest</h1>
          <p class="text-xs text-indigo-200 -mt-1">Banca de Questões & Caderno de Erros</p>
        </div>
      </div>
      
      <div class="flex items-center gap-4">
        <button onclick="navegarPara('dashboard')" class="text-sm font-medium hover:text-emerald-300 transition flex items-center gap-1.5">
          <i class="fa-solid fa-layer-group"></i> Provas
        </button>
        <button onclick="abrirModalNovaProva()" class="text-sm bg-indigo-700 hover:bg-indigo-600 px-3 py-1.5 rounded-lg border border-indigo-500 font-medium transition flex items-center gap-1.5">
          <i class="fa-solid fa-plus text-xs"></i> Adicionar Prova
        </button>
      </div>
    </div>
  </header>

  <!-- CONTAINER PRINCIPAL -->
  <main class="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6">

    <!-- ========================================== -->
    <!-- 1. TELA: DASHBOARD / SELETOR DE PROVAS     -->
    <!-- ========================================== -->
    <section id="view-dashboard" class="space-y-6">
      <!-- Banner de Boas-Vindas & Estatísticas Rápidas -->
      <div class="bg-gradient-to-r from-indigo-900 via-indigo-800 to-slate-900 rounded-2xl p-6 text-white shadow-lg">
        <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
          <div>
            <h2 class="text-2xl font-black mb-1">Painel de Simulados</h2>
            <p class="text-indigo-200 text-sm">Pratique com resolução comentada instantânea e consolidação de pontos-chave.</p>
          </div>
          <div class="flex gap-4">
            <div class="bg-white/10 backdrop-blur px-4 py-2.5 rounded-xl border border-white/10 text-center">
              <span class="block text-2xl font-bold text-emerald-400" id="stat-provas-feitas">0</span>
              <span class="text-xs text-slate-300">Concluídas</span>
            </div>
            <div class="bg-white/10 backdrop-blur px-4 py-2.5 rounded-xl border border-white/10 text-center">
              <span class="block text-2xl font-bold text-indigo-300" id="stat-taxa-geral">0%</span>
              <span class="text-xs text-slate-300">Aproveitamento</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Grade de Provas Disponíveis -->
      <div>
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-bold text-slate-800 flex items-center gap-2">
            <i class="fa-solid fa-book-medical text-indigo-600"></i> Provas Cadastradas
          </h3>
          <span class="text-xs bg-slate-200 text-slate-700 px-2.5 py-1 rounded-full font-medium" id="total-provas-contador">1 Prova</span>
        </div>

        <div id="grid-provas" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <!-- Renderizado via JS -->
        </div>
      </div>
    </section>

    <!-- ========================================== -->
    <!-- 2. TELA: RESOLUÇÃO INTERATIVA (ENGINE)     -->
    <!-- ========================================== -->
    <section id="view-quiz" class="hidden space-y-6">
      <!-- Barra Superior do Quiz -->
      <div class="bg-white border border-slate-200 rounded-xl p-4 shadow-sm flex flex-wrap justify-between items-center gap-3">
        <div>
          <button onclick="confirmarSaida()" class="text-xs text-slate-500 hover:text-slate-800 flex items-center gap-1 mb-1">
            <i class="fa-solid fa-arrow-left"></i> Voltar ao Painel
          </button>
          <h2 class="text-base font-bold text-slate-800" id="quiz-titulo">Título da Prova</h2>
        </div>

        <!-- Indicador de Questão e Barra de Progresso -->
        <div class="flex items-center gap-4">
          <div class="text-right">
            <span class="text-xs text-slate-400 block">Progresso</span>
            <span class="text-sm font-bold text-indigo-600" id="quiz-indice-contador">Questão 1 de 12</span>
          </div>
          <div class="w-32 bg-slate-200 rounded-full h-2.5 overflow-hidden">
            <div id="quiz-progresso-bar" class="bg-indigo-600 h-2.5 rounded-full transition-all duration-300" style="width: 8%"></div>
          </div>
          <button onclick="finalizarProva()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold px-3 py-2 rounded-lg transition shadow-sm">
            Finalizar Prova
          </button>
        </div>
      </div>

      <!-- Card da Questão Ativa -->
      <div class="bg-white border border-slate-200 rounded-2xl shadow-sm p-6 space-y-6">
        <!-- Cabeçalho da Questão -->
        <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-100 pb-3">
          <span class="text-xs font-bold uppercase tracking-wider px-3 py-1 rounded-md bg-indigo-50 text-indigo-700" id="q-tema">
            Tema da Questão
          </span>
          <span class="text-xs text-slate-400 font-medium" id="q-subtopico">Subtópico</span>
        </div>

        <!-- Texto Base / Situação Clínica (se houver) -->
        <div id="q-contexto-box" class="hidden bg-slate-50 border-l-4 border-indigo-600 p-4 rounded-r-lg text-sm text-slate-700 leading-relaxed">
          <p id="q-contexto"></p>
        </div>

        <!-- Enunciado Principal -->
        <div class="text-base font-semibold text-slate-900 leading-relaxed" id="q-enunciado">
          Enunciado da questão...
        </div>

        <!-- Imagens de Apoio (se houver) -->
        <div id="q-imagem-box" class="hidden">
          <img id="q-imagem" src="" alt="Imagem complementar" class="max-h-72 rounded-lg border border-slate-200 shadow-sm mx-auto object-contain bg-slate-900" />
        </div>

        <!-- Alternativas -->
        <div class="space-y-3 pt-2" id="q-alternativas-container">
          <!-- Renderizado via JS -->
        </div>

        <!-- Box de Gabarito e Comentário (Aparece Imediatamente após clicar) -->
        <div id="q-feedback-box" class="hidden rounded-xl border p-5 space-y-4 transition-all duration-300">
          <div class="flex items-center gap-2 font-bold text-sm" id="q-feedback-status">
            <!-- Ícone e status via JS -->
          </div>
          <div>
            <h4 class="text-xs font-bold text-slate-500 uppercase tracking-wider mb-1">Comentário Oficial:</h4>
            <p class="text-sm text-slate-700 leading-relaxed" id="q-comentario"></p>
          </div>
          <!-- Ideia Principal para Fixação -->
          <div class="bg-amber-50 border border-amber-200 rounded-lg p-3.5 flex items-start gap-3">
            <i class="fa-solid fa-lightbulb text-amber-600 mt-0.5 text-base"></i>
            <div>
              <span class="block text-xs font-bold text-amber-900 uppercase tracking-wider">Ideia Principal para Fixação:</span>
              <p class="text-sm text-amber-950 font-medium mt-0.5" id="q-fixacao"></p>
            </div>
          </div>
        </div>

        <!-- Navegação entre Questões -->
        <div class="flex justify-between items-center pt-4 border-t border-slate-100">
          <button id="btn-ant" onclick="mudarQuestao(-1)" class="px-4 py-2 border border-slate-300 text-slate-600 text-sm font-semibold rounded-lg hover:bg-slate-50 transition flex items-center gap-2">
            <i class="fa-solid fa-chevron-left text-xs"></i> Anterior
          </button>
          <div class="flex gap-1 overflow-x-auto max-w-[200px] md:max-w-md py-1 custom-scroll" id="painel-bolinhas">
            <!-- Marcadores numéricos via JS -->
          </div>
          <button id="btn-prox" onclick="mudarQuestao(1)" class="px-4 py-2 bg-indigo-600 text-white text-sm font-semibold rounded-lg hover:bg-indigo-700 transition flex items-center gap-2">
            Próxima <i class="fa-solid fa-chevron-right text-xs"></i>
          </button>
        </div>
      </div>
    </section>

    <!-- ========================================== -->
    <!-- 3. TELA: CADERNO DE ERROS & RESULTADOS     -->
    <!-- ========================================== -->
    <section id="view-resultado" class="hidden space-y-6">
      <div id="printArea" class="space-y-6">
        <!-- Card de Resumo de Desempenho -->
        <div class="bg-white border border-slate-200 rounded-2xl p-6 shadow-sm flex flex-col md:flex-row justify-between items-center gap-6">
          <div class="space-y-1 text-center md:text-left">
            <span class="text-xs font-bold text-indigo-600 uppercase tracking-wider">Resultado da Avaliação</span>
            <h2 class="text-2xl font-black text-slate-900" id="res-prova-nome">Nome da Prova</h2>
            <p class="text-sm text-slate-500">Confira abaixo os acertos e revise os pontos-chave de cada questão errada.</p>
          </div>

          <div class="flex items-center gap-6">
            <div class="text-center">
              <span class="text-3xl font-black text-indigo-600 block" id="res-score">0/12</span>
              <span class="text-xs text-slate-400 font-medium">Acertos</span>
            </div>
            <div class="h-10 w-[1px] bg-slate-200"></div>
            <div class="text-center">
              <span class="text-3xl font-black text-emerald-600 block" id="res-porcentagem">0%</span>
              <span class="text-xs text-slate-400 font-medium">Aproveitamento</span>
            </div>
          </div>

          <div class="flex gap-2 no-print">
            <button onclick="window.print()" class="px-4 py-2 border border-slate-300 text-slate-700 rounded-lg text-sm font-semibold hover:bg-slate-100 transition flex items-center gap-2">
              <i class="fa-solid fa-print"></i> Imprimir / PDF
            </button>
            <button onclick="navegarPara('dashboard')" class="px-4 py-2 bg-indigo-600 text-white rounded-lg text-sm font-semibold hover:bg-indigo-700 transition">
              Novo Simulado
            </button>
          </div>
        </div>

        <!-- Seção: Caderno de Erros -->
        <div class="space-y-4">
          <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3">
            <h3 class="text-xl font-bold text-slate-900 flex items-center gap-2">
              <i class="fa-solid fa-book-bookmark text-rose-500"></i> Caderno de Erros & Fixação
            </h3>
            
            <div class="flex items-center gap-2 no-print">
              <button onclick="filtrarCaderno('todos')" id="filtro-todos" class="text-xs font-bold px-3 py-1.5 rounded-lg bg-slate-200 text-slate-700">
                Todas (12)
              </button>
              <button onclick="filtrarCaderno('erros')" id="filtro-erros" class="text-xs font-bold px-3 py-1.5 rounded-lg bg-rose-100 text-rose-700">
                Apenas Erros
              </button>
            </div>
          </div>

          <!-- Lista do Caderno de Erros -->
          <div id="lista-caderno-erros" class="space-y-4">
            <!-- Cards de Erro/Revisão gerados via JS -->
          </div>
        </div>
      </div>
    </section>

  </main>

  <!-- ========================================== -->
  <!-- MODAL: CADASTRAR NOVA PROVA (JSON)         -->
  <!-- ========================================== -->
  <div id="modal-nova-prova" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-white rounded-2xl shadow-xl max-w-2xl w-full p-6 space-y-4 max-h-[90vh] flex flex-col">
      <div class="flex justify-between items-center border-b pb-3">
        <h3 class="text-lg font-bold text-slate-800 flex items-center gap-2">
          <i class="fa-solid fa-code text-indigo-600"></i> Importar Nova Prova
        </h3>
        <button onclick="fecharModalNovaProva()" class="text-slate-400 hover:text-slate-700 text-lg">
          <i class="fa-solid fa-xmark"></i>
        </button>
      </div>
      
      <p class="text-xs text-slate-500">
        Cole abaixo a estrutura em JSON de uma nova prova. O formato aceita situação clínica, tema, ideia de fixação e comentário completo para cada questão.
      </p>

      <div class="flex-1 min-h-[220px]">
        <textarea id="json-input" class="w-full h-full p-3 font-mono text-xs bg-slate-900 text-emerald-400 rounded-lg border border-slate-700 focus:outline-none focus:border-indigo-500 custom-scroll" placeholder='Cole seu JSON aqui...'></textarea>
      </div>

      <div class="flex justify-between items-center pt-2">
        <button onclick="carregarExemploTemplate()" class="text-xs text-indigo-600 hover:underline font-medium">
          Ver Modelo de Exemplo
        </button>
        <div class="flex gap-2">
          <button onclick="fecharModalNovaProva()" class="px-4 py-2 border rounded-lg text-xs font-bold text-slate-600 hover:bg-slate-50">Cancelar</button>
          <button onclick="salvarNovaProvaJSON()" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-lg text-xs font-bold">Salvar e Adicionar</button>
        </div>
      </div>
    </div>
  </div>

  <!-- ========================================================== -->
  <!-- SCRIPTS & MOTOR DA PLATAFORMA (LOGIC & DATABASE)          -->
  <!-- ========================================================== -->
  <script>
    // ------------------------------------------------------------------
    // BANCO DE DADOS DE PROVAS (ARMAZENA A PROVA DE SOI III DA UNINOVAFAPI)
    // ------------------------------------------------------------------
    const PROVAS_INICIAIS = [
      {
        id: "soi3-lab-1",
        titulo: "SOI III - 1ª Avaliação Multiest ações",
        instituicao: "Medicina - UNINOVAFAPI",
        descricao: "Cardiopatias (HAS, Chagas, IAM), Aterosclerose, TVP, Doença Renal e Farmacologia dos Anti-hipertensivos.",
        questoes: [
          {
            id: 1,
            tema: "Hipertensão & Repercussão Cardíaca/Vascular",
            subtopico: "Ecocardiograma e Ultrassonografia",
            contexto: "Homem de 58 anos, hipertenso há 12 anos, em uso irregular de losartana, comparece por cefaleia matinal e edema em MMII. PA=168/102 mmHg, sopro sistólico mitral. Ecocardiograma: hipertrofia concêntrica do VE e moderada obstrução da artéria abdominal com placa aterosclerótica.",
            enunciado: "Considerando os achados cardiovasculares, bem como a fisiopatologia associada, qual é a principal correlação clínica que explica esses resultados de imagem?",
            alternativas: {
              A: "Hipertensão arterial sistêmica de longa duração com repercussão cardíaca e vascular.",
              B: "Estenose aórtica grave como principal fator causal da hipertrofia e da obstrução arterial.",
              C: "Doença coronariana crônica como explicação primária para ambos os achados.",
              D: "Insuficiência cardíaca diastólica causada exclusivamente pela obstrução abdominal."
            },
            correta: "A",
            comentario: "A hipertrofia concêntrica do VE é adaptação clássica ao aumento crônico da pós-carga por HAS mantida. Esse remodelamento aumenta a espessura parietal sem dilatação cavitária. Paralelamente, o estresse hemodinâmico e a disfunção endotelial hipertensiva aceleram a aterosclerose em grandes artérias (como a aorta/artéria abdominal).",
            fixacao: "HAS crônica sustentada = Sobrecarga pressórica (Pós-carga elevada) gerando Hipertrofia Ventricular Concêntrica + aceleração da Aterosclerose vascular sistêmica."
          },
          {
            id: 2,
            tema: "Nefropatia Hipertensiva & Propedêutica Laboratorial",
            subtopico: "Avaliação Precoce de Lesão de Órgão-Alvo",
            contexto: "No contexto da hipertensão arterial sistêmica (HAS), a avaliação de lesão de órgão-alvo é essencial para estratificação de risco.",
            enunciado: "Entre os exames laboratoriais abaixo, qual é o mais sensível e específico para detecção precoce de lesão renal em pacientes hipertensos?",
            alternativas: {
              A: "Creatinina sérica, que avalia a função renal global, que costuma se elevar apenas quando já ocorreu perda significativa da taxa de filtração glomerular.",
              B: "Microalbuminúria de 24 horas (ou relação albumina/creatinina), que permite identificar a excreção mínima de proteínas na urina.",
              C: "Potássio sérico, útil para monitorar distúrbios hidroeletrolíticos.",
              D: "Hemograma completo, auxiliando na investigação de anemias e outras alterações hematológicas."
            },
            correta: "B",
            comentario: "A microalbuminúria (30-300 mg/24h) reflete dano endotelial e glomerular incipiente. A creatinina sérica só sofre elevação quando há perda de 50% ou mais da função renal glomerular, não sendo precoce.",
            fixacao: "Marcador mais precoce de lesão renal na HAS: Microalbuminúria urinária (a creatinina sérica é tardia e pouco sensível na fase inicial)."
          },
          {
            id: 3,
            tema: "Anatomopatologia Vascular",
            subtopico: "Doença Arterial Obstrutiva",
            contexto: "No exame de imagem do paciente hipertenso de 58 anos, foi evidenciada moderada obstrução interna da luz arterial por formação parietal intraluminal.",
            enunciado: "Qual patologia é mais compatível com esse achado, considerando os demais fatores de risco cardiovascular?",
            alternativas: {
              A: "Aterosclerose.",
              B: "Aneurisma verdadeiro.",
              C: "Aneurisma falso.",
              D: "Dissecção."
            },
            correta: "A",
            comentario: "As placas de ateroma crescem na túnica íntima e se projetam para o lúmen vascular, provocando obstrução interna. Aneurismas (dilatações) e dissecções (delaminação da camada média elástica) expandem-se para fora do vaso.",
            fixacao: "Aterosclerose projeta-se intraluminalmente (estenose da luz). Aneurisma é dilatação sacular/fusiforme excêntrica para fora da parede."
          },
          {
            id: 4,
            tema: "Radiologia Cardiotorácica",
            subtopico: "Radiografia de Tórax na Cardiopatia Hipertensiva",
            contexto: "Paciente masculino de 62 anos, hipertenso há 15 anos com dispneia e cansaço. Radiografia de tórax evidencia cardiomegalia e discreto alargamento do mediastino/botão aórtico.",
            enunciado: "Com base nesses achados, qual é a interpretação mais adequada das alterações radiográficas e sua correlação clínica?",
            alternativas: {
              A: "O alargamento do mediastino observado na radiografia sugere aneurisma dissecante de aorta agudo, principal causa da hipertrofia concêntrica ventricular.",
              B: "As alterações da silhueta cardíaca são mais compatíveis com derrame pericárdico crônico, frequentemente confundido com hipertrofia ventricular esquerda.",
              C: "O alargamento do mediastino e a cardiomegalia na radiografia indicam repercussões crônicas da hipertensão arterial sistêmica, compatíveis com hipertrofia ventricular e aortopatia hipertensiva.",
              D: "A radiografia demonstra sinais típicos de doença pulmonar obstrutiva crônica (DPOC), responsáveis pelas alterações cardíacas associadas."
            },
            correta: "C",
            comentario: "A cardiomegalia e o alongamento da aorta torácica (ectasia/aortopatia hipertensiva) são os sinais radiográficos típicos da sobrecarga pressórica de longa data.",
            fixacao: "Na HAS crônica: RX de tórax evidencia aumento da área cardíaca (hipertrofia/dilatação de VE) associado a alongamento e proeminência do arco aórtico (aortopatia hipertensiva)."
          },
          {
            id: 5,
            tema: "Farmacologia Cardiovascular",
            subtopico: "Inibidores da ECA & Efeitos Adversos",
            contexto: "Paciente hipertenso relata tosse seca e irritativa persistente, iniciada poucas semanas após introdução de novo fármaco anti-hipertensivo.",
            enunciado: "Qual é o mecanismo de ação do medicamento mais provavelmente associado a esse efeito colateral?",
            alternativas: {
              A: "Bloqueio dos canais de cálcio, reduzindo a entrada de cálcio nas células musculares lisas vasculares e miocárdio.",
              B: "Inibição da enzima conversora de angiotensina (ECA), reduzindo a formação de angiotensina II e aumentando os níveis de bradicinina.",
              C: "Bloqueio dos receptores beta-adrenérgicos cardíacos e renais, diminuindo frequência cardíaca e secreção de renina.",
              D: "Aumento da excreção de sódio e água pelo túbulo distal renal, promovendo redução do volume intravascular."
            },
            correta: "B",
            comentario: "A ECA é idêntica à cininase II, enzima que cliva a bradicinina. A inibição da ECA (enalapril, captopril, lisinopril) impede a degradação da bradicinina e substância P nas vias aéreas, causando tosse seca e risco de angioedema.",
            fixacao: "Tosse seca por iECA = Acúmulo de Bradicinina e Substância P (devido ao bloqueio da cininase II). Conduta: trocar por BRA (losartana)."
          },
          {
            id: 6,
            tema: "Patologia Geral & Celular",
            subtopico: "Hipertrofia Miocárdica",
            contexto: "Analise as seguintes assertivas: I - O paciente hipertenso sofreu hipertrofia ventricular esquerda excêntrica. II - Histologicamente, a fibra cardíaca contrátil desse paciente teve um aumento em seu volume celular. III - Devido a uma falha na função renal, houve um aumento do trabalho cardíaco e consequente hipertrofia ventricular.",
            enunciado: "Marque a alternativa que contenha apenas as assertivas verdadeiras:",
            alternativas: {
              A: "Apenas II.",
              B: "Apenas III.",
              C: "Apenas I e III.",
              D: "Apenas II e III."
            },
            correta: "D",
            comentario: "A assertiva I é falsa porque a sobrecarga pressórica da HAS gera hipertrofia CONCÊNTRICA (em paralelo), não excêntrica (em série). A II é verdadeira (miócitos adultos não dividem por mitose; aumentam de volume por síntese proteica). A III é verdadeira (a disfunção renal aumenta a volemia e resistência, ampliando a pós-carga).",
            fixacao: "Miócitos cardíacos são células permanentes que sofrem hipertrofia (aumento de volume celular, sem hiperplasia). Sobrecarga de pressão gera padrão CONCÊNTRICO."
          },
          {
            id: 7,
            tema: "Cardiopatia Chagásica & Eletrocardiograma",
            subtopico: "Arritmias e Bloqueios",
            contexto: "Homem de 54 anos, agricultor de Minas Gerais, com síncopes, FC=38 bpm e cardiomegalia. ECG evidencia Bloqueio Atrioventricular Total (BAVT) associado a extrassístoles ventriculares polimórficas frequentes; fração de ejeção de 30%.",
            enunciado: "Considerando os achados eletrocardiográficos, qual é a principal implicação clínica e terapêutica?",
            alternativas: {
              A: "As extrassístoles ventriculares frequentes são alterações benignas, sem repercussão clínica significativa, devendo ser apenas acompanhadas.",
              B: "O bloqueio atrioventricular total, associado às arritmias ventriculares, indica alto risco de morte súbita e necessidade de implante de marca-passo definitivo, além de tratamento da insuficiência cardíaca.",
              C: "O bloqueio atrioventricular total, em pacientes com cardiopatia chagásica, pode ser revertido com tratamento medicamentoso específico, sem necessidade de marca-passo.",
              D: "O achado de bloqueio atrioventricular total sugere apenas sobrecarga atrial, sendo o tratamento focado exclusivamente na anticoagulação profilática."
            },
            correta: "B",
            comentario: "O BAVT na doença de Chagas destrói o sistema His-Purkinje por fibrose inflamatória. Causa síncopes (Stokes-Adams) e morte súbita. A indicação mandatória é o marca-passo definitivo, aliado à otimização da IC com FE reduzida.",
            fixacao: "BAVT em Chagas = Destruição do sistema de condução cardíaco -> Risco altíssimo de morte súbita -> Indicação de Marca-Passo Definitivo."
          },
          {
            id: 8,
            tema: "Doença de Chagas & Diagnóstico Laboratorial",
            subtopico: "Imunodiagnóstico da Fase Crônica",
            contexto: "Suspeita de cardiopatia chagásica crônica em paciente de zona endêmica com cardiomegalia e bloqueio atrioventricular.",
            enunciado: "Qual é o método laboratorial mais adequado para confirmar a suspeita etiológica de cardiopatia chagásica crônica?",
            alternativas: {
              A: "Hemocultura direta, indicada principalmente para a fase aguda, pois a parasitemia é elevada.",
              B: "Sorologia para detecção de anticorpos anti-Trypanosoma cruzi (ELISA, Imunofluorescência ou HAI), recomendada para o diagnóstico na fase crônica.",
              C: "Pesquisa direta do parasito em esfregaço de sangue periférico por microscopia óptica.",
              D: "Dosagem de creatinina sérica isolada."
            },
            correta: "B",
            comentario: "Na fase crônica de Chagas, a parasitemia é mínima ou indetectável. Portanto, testes parasitológicos diretos têm baixa sensibilidade. O diagnóstico é sorológico (presença de IgG anti-T. cruzi por dois métodos sorológicos distintos).",
            fixacao: "Diagnóstico de Chagas Crônico: Sorologia (ELISA / IFI / HAI - dois testes positivos). Exames diretos (esfregaço) só têm sensibilidade na fase AGUDA."
          },
          {
            id: 9,
            tema: "Anatomopatologia da Doença de Chagas",
            subtopico: "Imunopatogenia e Lesão Miocárdica Crônica",
            contexto: "Na fase crônica da infecção pelo Trypanosoma cruzi, ocorre remodelamento ventricular progressivo com fibrose e aneurisma apical.",
            enunciado: "Com base nos achados anatomopatológicos e imunológicos da fase crônica, assinale a alternativa correta:",
            alternativas: {
              A: "O achado característico é o afinamento da ponta do VE associado à presença massiva de ninhos de tripomastigotas intracelulares intactos.",
              B: "A resposta imunológica envolve principalmente linfócitos T CD8+ e CD4+, com produção de citocinas pró-inflamatórias que contribuem para a destruição miocárdica e fibrose crônica.",
              C: "A histopatologia mostra necrose transmural extensa com abundância de tripomastigotas em todos os cardiomiócitos.",
              D: "A fibrose miocárdica é estritamente limitada ao septo interventricular, sem acometer a parede livre do ventrículo."
            },
            correta: "B",
            comentario: "Na fase crônica, a agressão ao miocárdio decorre de imunidade celular crônica mediada por linfócitos T (principalmente CD8+) que liberam citocinas (IFN-gama, TNF-alfa), perpetuando inflamação e fibrose. Ninhos de amastigotas (não tripomastigotas) são raros nessa fase.",
            fixacao: "Lesão na fase crônica de Chagas = Agressão imunomediada (Linfócitos T CD8+/CD4+ e citocinas pró-inflamatórias) gerando fibrose intersticial difusa e afinamento apical."
          },
          {
            id: 10,
            tema: "Trombose Venosa Profunda (TVP)",
            subtopico: "Diagnóstico Ultrassonográfico & Terapêutica",
            contexto: "Avaliação de edema assimétrico agudo em membro inferior com dor à palpação de trajeto venoso profundo.",
            enunciado: "Qual alternativa descreve corretamente os achados característicos relacionados à Trombose Venosa Profunda Aguda e sua conduta inicial?",
            alternativas: {
              A: "Presença de refluxo venoso ao Doppler Modo B, falência valvular e hipertensão venosa crônica; manejo apenas com flebotômicos orais.",
              B: "Doppler venoso mostrando refluxo com degeneração valvular crônica; tratamento focado em meias de compressão elástica sem anticoagulação.",
              C: "Doppler mostrando espessamento restrito à safena magna com trombo hiperecoico; tratamento com repouso e analgésicos sem anticoagulantes.",
              D: "Doppler venoso mostra veia não compressível com trombo hipoecoico e ausência de fluxo; trombo vermelho rico em fibrina/hemácias; tratamento com heparina (inibe fator Xa via antitrombina III)."
            },
            correta: "D",
            comentario: "No Doppler da TVP aguda, o critério diagnóstico essencial é a perda de compressibilidade venosa à manobra do transdutor. O trombo recente é hipoecoico. Na histologia, é um trombo vermelho (estase). A anticoagulação plena imediata com HBPM potencializa a antitrombina III inibindo o fator Xa e trombina.",
            fixacao: "Critério definitivo de TVP aguda no Doppler: INCOMPRESSIBILIDADE da veia + trombo hipoecoico. Tratamento: Anticoagulação imediata (ex: Enoxaparina / HBPM)."
          },
          {
            id: 11,
            tema: "Patologia Pericárdica",
            subtopico: "Pericardite e Derrame Pericárdico",
            contexto: "Considere as seguintes assertivas sobre pericardite e patologia do saco pericárdico: I - A radiografia de tórax pode mostrar aumento da área cardíaca (silhueta em moringa) em derrames volumosos. II - A citologia do líquido pericárdico auxilia na investigação etiológica de neoplasias. III - A pericardiocentese é isenta de riscos, dispensando ecocardiograma ou monitorização.",
            enunciado: "Assinale a alternativa correta:",
            alternativas: {
              A: "Apenas I é verdadeira.",
              B: "Apenas II é verdadeira.",
              C: "Apenas I e II são verdadeiras.",
              D: "Apenas II e III são verdadeiras."
            },
            correta: "C",
            comentario: "A afirmativa III é incorreta porque a pericardiocentese é um procedimento invasivo de risco considerável (risco de perfuração de câmara cardíaca, coronárias e pneumotórax), devendo ser guiada preferencialmente por ecocardiografia.",
            fixacao: "Derrame volumoso dá aspecto em 'moringa' no RX; citologia analisa malignidade. Punção pericárdica NUNCA é isenta de risco (exige guia ultrassonográfico)."
          },
          {
            id: 12,
            tema: "Síndrome Coronariana Aguda (IAM)",
            subtopico: "Diagnóstico Clínico, Eletrocardiográfico e Histopatológico",
            contexto: "Sobre o Infarto Agudo do Miocárdio: I - O supradesnivelamento do segmento ST é compatível com oclusão coronariana total aguda. II - A troponina cardíaca é um marcador bioquímico sensível e específico de necrose miocárdica. III - A necrose de coagulação das fibras miocárdicas é a marca histológica clássica do infarto.",
            enunciado: "Assinale a alternativa que apresenta o julgamento correto das afirmativas:",
            alternativas: {
              A: "Apenas I é verdadeira.",
              B: "Apenas I e II são verdadeiras.",
              C: "Apenas II e III são verdadeiras.",
              D: "I, II e III são verdadeiras."
            },
            correta: "D",
            comentario: "Todas as assertivas estão corretas: o IAM com supra reflete oclusão transmural aguda; a troponina é padrão-ouro para detectar necrose miocárdica; e o tipo de necrose tecidual típica do infarto isquêmico do miocárdio é a necrose de coagulação.",
            fixacao: "IAM com Supra = Oclusão coronariana total aguda. Biomarcador padrão = Troponina. Padrão histopatológico = Necrose de coagulação com infiltrado inflamatório."
          }
        ]
      }
    ];

    // ------------------------------------------------------------------
    // GERENCIAMENTO DE ESTADO DA APLICAÇÃO
    // ------------------------------------------------------------------
    let bancoProvas = [];
    let provaAtual = null;
    let questaoIndex = 0;
    let respostasUsuario = {}; // { questaoId: 'A' | 'B' | ... }
    let filtroAtualCaderno = 'todos';

    // Inicialização
    window.addEventListener('DOMContentLoaded', () => {
      carregarBanco();
      renderizarDashboard();
      atualizarEstatisticasGlobais();
    });

    function carregarBanco() {
      const salvo = localStorage.getItem('medquest_provas');
      if (salvo) {
        try {
          bancoProvas = JSON.parse(salvo);
        } catch (e) {
          bancoProvas = PROVAS_INICIAIS;
        }
      } else {
        bancoProvas = PROVAS_INICIAIS;
        salvarBanco();
      }
    }

    function salvarBanco() {
      localStorage.setItem('medquest_provas', JSON.stringify(bancoProvas));
    }

    // Navegação entre Telas
    function navegarPara(tela) {
      document.getElementById('view-dashboard').classList.add('hidden');
      document.getElementById('view-quiz').classList.add('hidden');
      document.getElementById('view-resultado').classList.add('hidden');

      if (tela === 'dashboard') {
        renderizarDashboard();
        atualizarEstatisticasGlobais();
        document.getElementById('view-dashboard').classList.remove('hidden');
      } else if (tela === 'quiz') {
        document.getElementById('view-quiz').classList.remove('hidden');
      } else if (tela === 'resultado') {
        document.getElementById('view-resultado').classList.remove('hidden');
      }
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    // ------------------------------------------------------------------
    // RENDERIZADOR DO DASHBOARD
    // ------------------------------------------------------------------
    function renderizarDashboard() {
      const grid = document.getElementById('grid-provas');
      const contador = document.getElementById('total-provas-contador');
      grid.innerHTML = '';
      contador.textContent = `${bancoProvas.length} ${bancoProvas.length === 1 ? 'Prova' : 'Provas'}`;

      bancoProvas.forEach((prova) => {
        const historico = JSON.parse(localStorage.getItem(`hist_${prova.id}`) || 'null');
        
        const card = document.createElement('div');
        card.className = "bg-white border border-slate-200 rounded-xl p-5 shadow-sm hover:shadow-md transition flex flex-col justify-between space-y-4";
        
        card.innerHTML = `
          <div class="space-y-2">
            <div class="flex justify-between items-start gap-2">
              <span class="text-xs font-bold text-indigo-600 bg-indigo-50 px-2.5 py-0.5 rounded">${prova.instituicao || 'Medicina'}</span>
              <span class="text-xs text-slate-400 font-medium">${prova.questoes.length} Questões</span>
            </div>
            <h4 class="font-bold text-slate-900 text-base leading-snug">${prova.titulo}</h4>
            <p class="text-xs text-slate-500 line-clamp-2">${prova.descricao || 'Sem descrição'}</p>
          </div>

          <div class="space-y-3 pt-2 border-t border-slate-100">
            ${historico ? `
              <div class="flex justify-between items-center text-xs">
                <span class="text-slate-400">Última pontuação:</span>
                <span class="font-bold ${historico.porcentagem >= 70 ? 'text-emerald-600' : 'text-amber-600'}">
                  ${historico.acertos}/${historico.total} (${historico.porcentagem}%)
                </span>
              </div>
            ` : `
              <span class="text-xs text-slate-400 italic">Ainda não respondida</span>
            `}

            <button onclick="iniciarProva('${prova.id}')" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-xs py-2.5 rounded-lg transition flex items-center justify-center gap-2">
              <i class="fa-solid fa-play text-[10px]"></i> Iniciar Simulado
            </button>
          </div>
        `;
        grid.appendChild(card);
      });
    }

    function atualizarEstatisticasGlobais() {
      let totalConcluidas = 0;
      let totalAcertos = 0;
      let totalRespondidas = 0;

      bancoProvas.forEach(prova => {
        const hist = JSON.parse(localStorage.getItem(`hist_${prova.id}`) || 'null');
        if (hist) {
          totalConcluidas++;
          totalAcertos += hist.acertos;
          totalRespondidas += hist.total;
        }
      });

      document.getElementById('stat-provas-feitas').textContent = totalConcluidas;
      const taxa = totalRespondidas > 0 ? Math.round((totalAcertos / totalRespondidas) * 100) : 0;
      document.getElementById('stat-taxa-geral').textContent = `${taxa}%`;
    }

    // ------------------------------------------------------------------
    // MOTOR DO QUIZ (ENGINE DE RESOLUÇÃO COM FEEDBACK IMEDIATO)
    // ------------------------------------------------------------------
    function iniciarProva(provaId) {
      provaAtual = bancoProvas.find(p => p.id === provaId);
      if (!provaAtual) return;

      questaoIndex = 0;
      respostasUsuario = {};
      document.getElementById('quiz-titulo').textContent = provaAtual.titulo;

      navegarPara('quiz');
      renderizarQuestao();
    }

    function renderizarQuestao() {
      const q = provaAtual.questoes[questaoIndex];
      const totalQ = provaAtual.questoes.length;

      // Atualiza Navegador e Progresso
      document.getElementById('quiz-indice-contador').textContent = `Questão ${questaoIndex + 1} de ${totalQ}`;
      document.getElementById('quiz-progresso-bar').style.width = `${((questaoIndex + 1) / totalQ) * 100}%`;

      document.getElementById('q-tema').textContent = q.tema || "Clínica Médica";
      document.getElementById('q-subtopico').textContent = q.subtopico || "";

      // Contexto Clínico
      const contextoBox = document.getElementById('q-contexto-box');
      if (q.contexto) {
        document.getElementById('q-contexto').textContent = q.contexto;
        contextoBox.classList.remove('hidden');
      } else {
        contextoBox.classList.add('hidden');
      }

      // Enunciado
      document.getElementById('q-enunciado').textContent = q.enunciado;

      // Imagem
      const imgBox = document.getElementById('q-imagem-box');
      if (q.imagem) {
        document.getElementById('q-imagem').src = q.imagem;
        imgBox.classList.remove('hidden');
      } else {
        imgBox.classList.add('hidden');
      }

      // Alternativas
      const altContainer = document.getElementById('q-alternativas-container');
      altContainer.innerHTML = '';

      const respondida = respostasUsuario[q.id] !== undefined;
      const letraEscolhida = respostasUsuario[q.id];

      ['A', 'B', 'C', 'D'].forEach(letra => {
        if (!q.alternativas[letra]) return;

        const btn = document.createElement('button');
        btn.disabled = respondida;
        btn.className = `w-full text-left p-3.5 rounded-xl border text-sm font-medium transition flex items-start gap-3 `;

        if (!respondida) {
          btn.className += "border-slate-200 hover:border-indigo-400 hover:bg-indigo-50/50 bg-white text-slate-800";
        } else {
          // Feedback de Cores
          if (letra === q.correta) {
            btn.className += "border-emerald-500 bg-emerald-50 text-emerald-900 font-semibold ring-2 ring-emerald-500/20";
          } else if (letra === letraEscolhida) {
            btn.className += "border-rose-400 bg-rose-50 text-rose-900 line-through ring-2 ring-rose-400/20";
          } else {
            btn.className += "border-slate-200 bg-slate-50 text-slate-400 opacity-60";
          }
        }

        btn.onclick = () => selecionarResposta(q.id, letra);

        btn.innerHTML = `
          <span class="w-6 h-6 rounded-full flex items-center justify-center text-xs font-bold shrink-0 ${
            respondida && letra === q.correta ? 'bg-emerald-600 text-white' : 
            respondida && letra === letraEscolhida ? 'bg-rose-500 text-white' : 
            'bg-slate-100 text-slate-700'
          }">${letra}</span>
          <span class="leading-relaxed mt-0.5">${q.alternativas[letra]}</span>
        `;
        altContainer.appendChild(btn);
      });

      // Feedback Box
      const feedbackBox = document.getElementById('q-feedback-box');
      if (respondida) {
        feedbackBox.classList.remove('hidden');
        const acertou = letraEscolhida === q.correta;
        const statusEl = document.getElementById('q-feedback-status');
        
        if (acertou) {
          statusEl.innerHTML = `<i class="fa-solid fa-circle-check text-emerald-600 text-base"></i> <span class="text-emerald-700">Parabéns! Você acertou. Alternativa ${q.correta}.</span>`;
          feedbackBox.className = "rounded-xl border border-emerald-200 bg-emerald-50/40 p-5 space-y-4";
        } else {
          statusEl.innerHTML = `<i class="fa-solid fa-circle-xmark text-rose-600 text-base"></i> <span class="text-rose-700">Resposta incorreta. O gabarito oficial é a alternativa ${q.correta}.</span>`;
          feedbackBox.className = "rounded-xl border border-rose-200 bg-rose-50/40 p-5 space-y-4";
        }

        document.getElementById('q-comentario').textContent = q.comentario || "Sem comentário cadastrado.";
        document.getElementById('q-fixacao').textContent = q.fixacao || "Revise o conceito fundamental da questão.";
      } else {
        feedbackBox.classList.add('hidden');
      }

      // Botões Anterior / Próxima
      document.getElementById('btn-ant').disabled = questaoIndex === 0;
      document.getElementById('btn-ant').classList.toggle('opacity-50', questaoIndex === 0);
      
      const ehUltima = questaoIndex === totalQ - 1;
      document.getElementById('btn-prox').innerHTML = ehUltima ? 
        `Ver Resultados <i class="fa-solid fa-flag-checkered text-xs"></i>` : 
        `Próxima <i class="fa-solid fa-chevron-right text-xs"></i>`;

      renderizarPainelBolinhas();
    }

    function renderizarPainelBolinhas() {
      const container = document.getElementById('painel-bolinhas');
      container.innerHTML = '';
      provaAtual.questoes.forEach((q, idx) => {
        const b = document.createElement('button');
        b.onclick = () => { questaoIndex = idx; renderizarQuestao(); };
        b.className = `w-7 h-7 rounded-lg text-xs font-bold transition flex items-center justify-center shrink-0 `;

        const resp = respostasUsuario[q.id];
        if (idx === questaoIndex) {
          b.className += "ring-2 ring-indigo-600 font-black ";
        }

        if (resp === undefined) {
          b.className += "bg-slate-100 text-slate-600 hover:bg-slate-200";
        } else if (resp === q.correta) {
          b.className += "bg-emerald-500 text-white";
        } else {
          b.className += "bg-rose-500 text-white";
        }
        b.textContent = idx + 1;
        container.appendChild(b);
      });
    }

    function selecionarResposta(questaoId, letra) {
      respostasUsuario[questaoId] = letra;
      renderizarQuestao();
    }

    function mudarQuestao(delta) {
      const novoIdx = questaoIndex + delta;
      if (novoIdx >= 0 && novoIdx < provaAtual.questoes.length) {
        questaoIndex = novoIdx;
        renderizarQuestao();
      } else if (novoIdx >= provaAtual.questoes.length) {
        finalizarProva();
      }
    }

    function confirmarSaida() {
      if (confirm("Deseja sair deste simulado e voltar para o menu principal? Seu progresso nesta sessão será perdido.")) {
        navegarPara('dashboard');
      }
    }

    // ------------------------------------------------------------------
    // FINALIZAÇÃO & CADERNO DE ERROS
    // ------------------------------------------------------------------
    function finalizarProva() {
      const total = provaAtual.questoes.length;
      let acertos = 0;

      provaAtual.questoes.forEach(q => {
        if (respostasUsuario[q.id] === q.correta) {
          acertos++;
        }
      });

      const porcentagem = Math.round((acertos / total) * 100);

      // Salva histórico
      localStorage.setItem(`hist_${provaAtual.id}`, JSON.stringify({
        acertos,
        total,
        porcentagem,
        data: new Date().toLocaleDateString('pt-BR')
      }));

      // Preenche Tela de Resultados
      document.getElementById('res-prova-nome').textContent = provaAtual.titulo;
      document.getElementById('res-score').textContent = `${acertos}/${total}`;
      document.getElementById('res-porcentagem').textContent = `${porcentagem}%`;

      filtroAtualCaderno = 'todos';
      renderizarCadernoErros();
      navegarPara('resultado');
    }

    function filtrarCaderno(tipo) {
      filtroAtualCaderno = tipo;
      document.getElementById('filtro-todos').className = tipo === 'todos' ? 
        'text-xs font-bold px-3 py-1.5 rounded-lg bg-indigo-600 text-white' : 
        'text-xs font-bold px-3 py-1.5 rounded-lg bg-slate-200 text-slate-700';
      document.getElementById('filtro-erros').className = tipo === 'erros' ? 
        'text-xs font-bold px-3 py-1.5 rounded-lg bg-rose-600 text-white' : 
        'text-xs font-bold px-3 py-1.5 rounded-lg bg-slate-200 text-slate-700';
      renderizarCadernoErros();
    }

    function renderizarCadernoErros() {
      const container = document.getElementById('lista-caderno-erros');
      container.innerHTML = '';

      let questoesExibidas = provaAtual.questoes.filter(q => {
        const marcada = respostasUsuario[q.id];
        const errou = marcada !== q.correta;
        return filtroAtualCaderno === 'erros' ? errou : true;
      });

      if (questoesExibidas.length === 0) {
        container.innerHTML = `
          <div class="bg-white border rounded-xl p-8 text-center text-slate-500">
            <i class="fa-solid fa-circle-check text-4xl text-emerald-500 mb-2"></i>
            <p class="font-bold text-slate-800">Nenhum erro registrado com este filtro!</p>
            <p class="text-xs text-slate-400">Você dominou todas as questões desta seção.</p>
          </div>
        `;
        return;
      }

      questoesExibidas.forEach((q) => {
        const marcada = respostasUsuario[q.id] || "Não respondida";
        const errou = marcada !== q.correta;

        const card = document.createElement('div');
        card.className = `bg-white border rounded-xl p-5 shadow-sm space-y-4 ${errou ? 'border-l-4 border-l-rose-500' : 'border-l-4 border-l-emerald-500'}`;

        card.innerHTML = `
          <div class="flex flex-wrap justify-between items-center gap-2 border-b border-slate-100 pb-2">
            <div>
              <span class="text-xs font-bold uppercase tracking-wider text-indigo-700 bg-indigo-50 px-2 py-0.5 rounded">Questão ${q.id}</span>
              <span class="text-xs font-semibold text-slate-600 ml-2">${q.tema}</span>
            </div>
            <div class="text-xs font-bold">
              ${errou ? 
                `<span class="text-rose-600 bg-rose-50 px-2.5 py-1 rounded-md">Marcou: ${marcada} \vert{} Correta:${q.correta}</span>` : 
                `<span class="text-emerald-700 bg-emerald-50 px-2.5 py-1 rounded-md"><i class="fa-solid fa-check"></i> Acertou (Letra ${q.correta})</span>`
              }
            </div>
          </div>

          <p class="text-sm font-semibold text-slate-800">${q.enunciado}</p>

          <!-- IDEIA DE FIXAÇÃO EM DESTAQUE -->
          <div class="bg-amber-50 border border-amber-200 rounded-lg p-3">
            <span class="text-xs font-bold text-amber-900 block flex items-center gap-1.5 uppercase">
              <i class="fa-solid fa-bullseye text-amber-600"></i> Ideia Principal para Fixação:
            </span>
            <p class="text-xs text-amber-950 font-medium mt-1 leading-relaxed">${q.fixacao}</p>
          </div>

          <div class="text-xs text-slate-600 bg-slate-50 p-3 rounded-lg border border-slate-100 leading-relaxed">
            <span class="font-bold text-slate-700 block mb-0.5">Justificativa Resumida:</span>
            ${q.comentario}
          </div>
        `;

        container.appendChild(card);
      });
    }

    // ------------------------------------------------------------------
    // MODAL & ADIÇÃO DE NOVAS PROVAS
    // ------------------------------------------------------------------
    function abrirModalNovaProva() {
      document.getElementById('modal-nova-prova').classList.remove('hidden');
    }

    function fecharModalNovaProva() {
      document.getElementById('modal-nova-prova').classList.add('hidden');
    }

    function carregarExemploTemplate() {
      const template = {
        id: "nova-prova-" + Date.now(),
        titulo: "Farmacologia & Cardiologia - Teste",
        instituicao: "Medicina",
        descricao: "Simulado rápido de fixação sobre drogas vasoativas e choque.",
        questoes: [
          {
            id: 1,
            tema: "Choque Circulatório",
            subtopico: "Drogas Vasoativas",
            contexto: "Paciente em choque séptico refratário a volume.",
            enunciado: "Qual é a droga vasoativa de primeira linha indicada para restaurar a PAM?",
            alternativas: {
              A: "Noradrenalina",
              B: "Dobutamina",
              C: "Dopamina",
              D: "Adrenalina"
            },
            correta: "A",
            comentario: "A noradrenalina é o vasopressor de escolha por sua potente ação alfa-1 agonista, aumentando a resistência vascular sistêmica com menor incidência de arritmias.",
            fixacao: "Primeira linha no Choque Séptico = Noradrenalina (alvo PAM >= 65 mmHg)."
          }
        ]
      };
      document.getElementById('json-input').value = JSON.stringify(template, null, 2);
    }

    function salvarNovaProvaJSON() {
      const txt = document.getElementById('json-input').value.trim();
      if (!txt) {
        alert("Por favor, cole um JSON válido.");
        return;
      }

      try {
        const nova = JSON.parse(txt);
        if (!nova.id || !nova.titulo || !nova.questoes || !Array.isArray(nova.questoes)) {
          alert("O JSON deve conter pelo menos 'id', 'titulo' e uma lista de 'questoes'.");
          return;
        }

        bancoProvas.push(nova);
        salvarBanco();
        fecharModalNovaProva();
        renderizarDashboard();
        alert("Prova cadastrada com sucesso!");
      } catch (e) {
        alert("Erro de sintaxe no JSON: " + e.message);
      }
    }
  </script>
</body>
</html>
