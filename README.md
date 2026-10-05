<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contas Jr.</title>
  <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>💸</text></svg>">
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body { background-color: #0f172a; color: #f8fafc; font-family: sans-serif; }
    .conta-item { border-left: 4px solid #334155; transition: all 0.3s ease; }
    .conta-paga { border-left-color: #10b981; opacity: 0.7; }
    .conta-atrasada { border-left-color: #ef4444; }
    .conta-pendente { border-left-color: #f59e0b; }
    .hidden { display: none !important; }
  </style>
</head>
<body class="p-4 pb-20">

  <!-- Cabeçalho -->
  <header class="flex justify-center items-center mb-6 mt-2">
    <h1 class="text-3xl font-bold text-emerald-500">Contas Jr.</h1>
  </header>

  <!-- Dashboard 3.0 (Total / Pendente / Pago) -->
  <section class="grid grid-cols-3 gap-2 mb-6">
    <div class="bg-slate-800 p-3 rounded-xl shadow-md border border-slate-700/50 text-center">
      <p class="text-[10px] uppercase font-bold text-slate-400">Total</p>
      <p class="text-sm font-bold text-white mt-1" id="dash-total">R$ 0,00</p>
    </div>
    <div class="bg-slate-800 p-3 rounded-xl shadow-md border border-slate-700/50 text-center">
      <p class="text-[10px] uppercase font-bold text-slate-400">Pendente</p>
      <p class="text-sm font-bold text-amber-500 mt-1" id="dash-pendente">R$ 0,00</p>
    </div>
    <div class="bg-slate-800 p-3 rounded-xl shadow-md border border-slate-700/50 text-center">
      <p class="text-[10px] uppercase font-bold text-slate-400">Pago</p>
      <p class="text-sm font-bold text-emerald-500 mt-1" id="dash-pago">R$ 0,00</p>
    </div>
  </section>

  <!-- Lista de Contas -->
  <main>
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-lg font-semibold">Suas Contas</h2>
    </div>
    <div id="lista-contas" class="space-y-3">
      <!-- Contas carregadas aqui -->
    </div>
  </main>

  <!-- Botão Flutuante (+) -->
  <button onclick="abrirModal()" class="fixed bottom-6 right-6 bg-emerald-600 hover:bg-emerald-500 text-white w-14 h-14 rounded-full text-3xl shadow-lg flex items-center justify-center pb-1 z-40 transition transform active:scale-95">
    +
  </button>

  <!-- MODAL DE NOVA CONTA -->
  <div id="modal-add" class="hidden fixed inset-0 bg-slate-900/95 z-50 flex items-center justify-center p-4 transition-opacity">
    <div class="bg-slate-800 p-6 rounded-2xl w-full max-w-sm border border-slate-600 shadow-2xl">
      <h3 class="text-xl font-bold text-emerald-500 mb-4">Nova Conta</h3>
      <div class="space-y-4">
        <div>
          <label class="block text-xs font-semibold text-slate-400 uppercase mb-1">Descrição</label>
          <input type="text" id="nova-nome" placeholder="Ex: Conta de Luz" class="w-full bg-slate-900 border border-slate-600 rounded-lg p-3 text-white focus:outline-none focus:border-emerald-500">
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-400 uppercase mb-1">Valor (R$)</label>
          <input type="number" step="0.01" id="nova-valor" placeholder="Ex: 150.50" class="w-full bg-slate-900 border border-slate-600 rounded-lg p-3 text-white focus:outline-none focus:border-emerald-500">
        </div>
        <div>
          <label class="block text-xs font-semibold text-slate-400 uppercase mb-1">Vencimento</label>
          <input type="date" id="nova-data" class="w-full bg-slate-900 border border-slate-600 rounded-lg p-3 text-white focus:outline-none focus:border-emerald-500">
        </div>
      </div>
      <div class="flex gap-3 mt-6">
        <button onclick="fecharModal()" class="flex-1 bg-slate-700 hover:bg-slate-600 text-white p-3 rounded-lg font-bold transition">Cancelar</button>
        <button onclick="salvarNovaConta()" class="flex-1 bg-emerald-600 hover:bg-emerald-500 text-white p-3 rounded-lg font-bold transition">Salvar</button>
      </div>
    </div>
  </div>

  <!-- SCRIPTS E MOTOR DO SISTEMA -->
  <script>
    // O banco de dados volta a ser direto no aparelho, sem contas de usuário
    let contas = JSON.parse(localStorage.getItem('contasJr_db_v2')) || [];

    function formatarMoeda(valor) {
      return Number(valor).toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
    }

    function abrirModal() {
      document.getElementById('nova-data').value = new Date().toISOString().split('T')[0];
      document.getElementById('modal-add').classList.remove('hidden');
    }
    
    function fecharModal() {
      document.getElementById('modal-add').classList.add('hidden');
      document.getElementById('nova-nome').value = '';
      document.getElementById('nova-valor').value = '';
    }

    function salvarNovaConta() {
      const nome = document.getElementById('nova-nome').value;
      const valor = document.getElementById('nova-valor').value;
      const vencimento = document.getElementById('nova-data').value;

      if(!nome || !valor || !vencimento) return alert("Preencha todos os campos!");

      contas.push({ nome, valor: Number(valor), vencimento, paga: false });
      salvarDados();
      fecharModal();
      renderizarContas();
    }

    function atualizarDashboard() {
      const total = contas.reduce((acc, c) => acc + Number(c.valor), 0);
      const pendente = contas.filter(c => !c.paga).reduce((acc, c) => acc + Number(c.valor), 0);
      const pago = contas.filter(c => c.paga).reduce((acc, c) => acc + Number(c.valor), 0);
      
      document.getElementById('dash-total').innerText = formatarMoeda(total);
      document.getElementById('dash-pendente').innerText = formatarMoeda(pendente);
      document.getElementById('dash-pago').innerText = formatarMoeda(pago);
    }

    function renderizarContas() {
      const lista = document.getElementById('lista-contas');
      lista.innerHTML = '';

      if (contas.length === 0) {
        lista.innerHTML = '<p class="text-slate-500 text-center mt-10">Nenhuma conta. Toque no + para começar.</p>';
      }

      // Ordena por data de vencimento mais próxima
      contas.sort((a, b) => new Date(a.vencimento) - new Date(b.vencimento));

      contas.forEach((conta, index) => {
        const dataHoje = new Date().toISOString().split('T')[0];
        let statusClass = 'conta-pendente';
        let statusText = 'Pendente';
        let statusColor = 'text-amber-500';

        if (conta.paga) {
          statusClass = 'conta-paga';
          statusText = 'Pago';
          statusColor = 'text-emerald-500';
        } else if (conta.vencimento < dataHoje) {
          statusClass = 'conta-atrasada';
          statusText = 'Atrasado';
          statusColor = 'text-rose-500';
        }

        const partesData = conta.vencimento.split('-');
        const dataExibicao = partesData.length === 3 ? `${partesData[2]}/${partesData[1]}` : conta.vencimento;

        lista.innerHTML += `
          <div class="bg-slate-800 p-4 rounded-xl shadow flex justify-between items-center conta-item ${statusClass}">
            <div class="flex-1">
              <p class="font-bold text-white text-sm">${conta.nome}</p>
              <p class="text-[11px] text-slate-400 mt-1">Vence: <span class="text-slate-300">${dataExibicao}</span></p>
            </div>
            <div class="text-right flex flex-col items-end gap-1">
              <p class="font-bold text-slate-100 text-sm">${formatarMoeda(conta.valor)}</p>
              <div class="flex gap-4 items-center mt-2">
                <button onclick="excluirConta(${index})" class="text-[10px] uppercase font-bold text-slate-500 tracking-wider hover:text-rose-400">Excluir</button>
                <button onclick="alternarPagamento(${index})" class="text-[10px] font-bold ${statusColor} uppercase tracking-wider bg-slate-900 px-2 py-1 rounded border border-slate-700">
                  ${statusText}
                </button>
              </div>
            </div>
          </div>
        `;
      });
      atualizarDashboard();
    }

    function alternarPagamento(index) {
      contas[index].paga = !contas[index].paga;
      salvarDados();
      renderizarContas();
    }

    function excluirConta(index) {
      if (confirm("Excluir esta conta?")) {
        contas.splice(index, 1);
        salvarDados();
        renderizarContas();
      }
    }

    function salvarDados() {
      localStorage.setItem('contasJr_db_v2', JSON.stringify(contas));
    }

    // Inicializa a tela ao abrir o app
    renderizarContas();
  </script>
</body>
</html>

