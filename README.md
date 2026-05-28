[index.html..html](https://github.com/user-attachments/files/28347788/index.html.html)
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Controle de Ferramentas QR Code - Almoxarifado</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body { margin: 0; font-family: system-ui, -apple-system, sans-serif; }
    .active-scale:active { transform: scale(0.95); }
  </style>
</head>
<body class="bg-slate-100">
  <div id="root"></div>

  <script type="text/babel">
    function ControleFerramentasQRCode() {
      const [ferramentas, setFerramentas] = React.useState([]);
      const [historico, setHistorico] = React.useState([]);
      const [mostrarCadastro, setMostrarCadastro] = React.useState(false);
      const [mostrarMovRapid, setMostrarMovRapid] = React.useState(false);
      const [mostrarHistorico, setMostrarHistorico] = React.useState(false);
      const [historicoId, setHistoricoId] = React.useState("");

      // Formulário de cadastro
      const [novoId, setNovoId] = React.useState("");
      const [novoNome, setNovoNome] = React.useState("");
      const [novoSetor, setNovoSetor] = React.useState("");
      const [novoStatus, setNovoStatus] = React.useState("Disponível");

      // Movimentação rápida
      const [movCodigo, setMovCodigo] = React.useState("");
      const [movUsuario, setMovUsuario] = React.useState("");
      const [movTipo, setMovTipo] = React.useState("Retirada");
      const [movObservacoes, setMovObservacoes] = React.useState("");

      // Carregar dados do localStorage ao iniciar
      React.useEffect(() => {
        const salvosFerramentas = localStorage.getItem('ferramentas_qr');
        const salvosHistorico = localStorage.getItem('historico_qr');
        if (salvosFerramentas) {
          try {
            setFerramentas(JSON.parse(salvosFerramentas));
          } catch(e) {
            setFerramentas([]);
          }
        }
        if (salvosHistorico) {
          try {
            setHistorico(JSON.parse(salvosHistorico));
          } catch(e) {
            setHistorico([]);
          }
        }
      }, []);

      // Salvar sempre que ferramentas ou historico mudarem
      React.useEffect(() => {
        localStorage.setItem('ferramentas_qr', JSON.stringify(ferramentas));
      }, [ferramentas]);

      React.useEffect(() => {
        localStorage.setItem('historico_qr', JSON.stringify(historico));
      }, [historico]);

      const gerarQRCode = (texto) => {
        return `https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=${encodeURIComponent(texto)}`;
      };

      const cadastrarFerramenta = (e) => {
        e.preventDefault();
        if (!novoId || !novoNome || !novoSetor) {
          alert("Preencha todos os campos!");
          return;
        }
        const idUpper = novoId.trim().toUpperCase();
        if (ferramentas.some(f => f.id === idUpper)) {
          alert("ID já existe!");
          return;
        }
        const nova = {
          id: idUpper,
          nome: novoNome.trim(),
          setor: novoSetor.trim(),
          status: novoStatus,
          responsavel: novoStatus === "Disponível" ? "-" : "Em Uso",
          observacoes: ""
        };
        setFerramentas([...ferramentas, nova]);
        alert("Ferramenta cadastrada com sucesso!");
        setNovoId("");
        setNovoNome("");
        setNovoSetor("");
        setNovoStatus("Disponível");
        setMostrarCadastro(false);
      };

      const registrarMovimentacao = () => {
        if (!movCodigo || !movUsuario) {
          alert("Preencha código e nome do colaborador!");
          return;
        }
        const idx = ferramentas.findIndex(f => f.id === movCodigo.toUpperCase());
        if (idx === -1) {
          alert("Ferramenta não encontrada!");
          return;
        }
        const novaLista = [...ferramentas];
        const agora = new Date().toLocaleString('pt-BR');

        const registroHistorico = {
          data: agora,
          tipo: movTipo,
          codigo: movCodigo.toUpperCase(),
          nome: novaLista[idx].nome,
          usuario: movUsuario.trim(),
          setor: novaLista[idx].setor,
          observacoes: movObservacoes.trim(),
          idFerramenta: novaLista[idx].id
        };

        if (movTipo === "Retirada") {
          if (novaLista[idx].status !== "Disponível") {
            alert("Ferramenta não está disponível!");
            return;
          }
          novaLista[idx].status = "Em Uso";
          novaLista[idx].responsavel = movUsuario.trim();
        } else {
          if (novaLista[idx].status !== "Em Uso") {
            alert("Ferramenta não está em uso!");
            return;
          }
          novaLista[idx].status = "Disponível";
          novaLista[idx].responsavel = "-";
          novaLista[idx].observacoes = movObservacoes.trim();
        }

        setFerramentas(novaLista);
        setHistorico([registroHistorico, ...historico]);
        setMovCodigo("");
        setMovUsuario("");
        setMovObservacoes("");
        alert("Movimentação registrada com sucesso!");
      };

      const deletarFerramenta = (id) => {
        if (!confirm("Excluir esta ferramenta?")) return;
        setFerramentas(ferramentas.filter(f => f.id !== id));
      };

      const verHistoricoFerramenta = (id, nome) => {
        setHistoricoId(id);
        setMostrarHistorico(true);
      };

      const historicoFiltrado = historico.filter(h => h.idFerramenta === historicoId);

      return (
        <div className="min-h-screen bg-slate-100 p-3 md:p-4">
          <div className="max-w-5xl mx-auto">
            {/* Cabeçalho */}
            <div className="bg-gradient-to-r from-blue-900 to-blue-700 text-white rounded-[30px] p-5 shadow-xl mb-4">
              <div className="flex items-center justify-between gap-4 flex-wrap">
                <div>
                  <h1 className="text-3xl md:text-4xl font-bold">Controle de Ferramentas</h1>
                  <p className="text-blue-100 text-base mt-1">Sistema com QR Code para Almoxarifado</p>
                </div>
                <div className="flex gap-2 flex-wrap">
                  <button 
                    onClick={() => { setMostrarCadastro(!mostrarCadastro); setMostrarMovRapid(false); setMostrarHistorico(false); }}
                    className="bg-white text-blue-900 px-5 py-3 rounded-2xl font-bold shadow-md hover:bg-blue-50 transition"
                  >
                    + Ferramenta
                  </button>
                  <button 
                    onClick={() => { setMostrarMovRapid(!mostrarMovRapid); setMostrarCadastro(false); setMostrarHistorico(false); }}
                    className="bg-blue-800 text-white px-5 py-3 rounded-2xl font-bold shadow-md hover:bg-blue-900 transition"
                  >
                    Movimentação
                  </button>
                </div>
              </div>
            </div>

            {/* Cards de resumo */}
            <div className="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4">
              <div className="bg-white rounded-3xl p-4 shadow-md text-center">
                <p className="text-gray-500 text-sm">Total</p>
                <h2 className="text-3xl font-bold text-slate-800 mt-1">{ferramentas.length}</h2>
              </div>
              <div className="bg-white rounded-3xl p-4 shadow-md text-center">
                <p className="text-gray-500 text-sm">Disponíveis</p>
                <h2 className="text-3xl font-bold text-green-600 mt-1">
                  {ferramentas.filter(f => f.status === "Disponível").length}
                </h2>
              </div>
              <div className="bg-white rounded-3xl p-4 shadow-md text-center">
                <p className="text-gray-500 text-sm">Em Uso</p>
                <h2 className="text-3xl font-bold text-orange-500 mt-1">
                  {ferramentas.filter(f => f.status === "Em Uso").length}
                </h2>
              </div>
            </div>

            {/* Formulário de Cadastro */}
            {mostrarCadastro && (
              <div className="bg-white rounded-[30px] shadow-xl p-5 mb-4 border-2 border-blue-200">
                <h2 className="text-2xl font-bold text-slate-800 mb-4">Cadastrar Nova Ferramenta</h2>
                <form onSubmit={cadastrarFerramenta} className="space-y-4">
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">ID da Ferramenta</label>
                    <input
                      type="text"
                      placeholder="Ex: FT-001"
                      value={novoId}
                      onChange={(e) => setNovoId(e.target.value.toUpperCase())}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                      required
                    />
                  </div>
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Nome</label>
                    <input
                      type="text"
                      placeholder="Ex: Furadeira Bosch"
                      value={novoNome}
                      onChange={(e) => setNovoNome(e.target.value)}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                      required
                    />
                  </div>
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Setor</label>
                    <input
                      type="text"
                      placeholder="Ex: Manutenção, Produção, Solda"
                      value={novoSetor}
                      onChange={(e) => setNovoSetor(e.target.value)}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                      required
                    />
                  </div>
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Status Inicial</label>
                    <select
                      value={novoStatus}
                      onChange={(e) => setNovoStatus(e.target.value)}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                    >
                      <option>Disponível</option>
                      <option>Em Uso</option>
                      <option>Manutenção</option>
                    </select>
                  </div>
                  <div className="flex gap-3">
                    <button type="submit" className="flex-1 bg-green-600 text-white py-4 rounded-2xl font-bold text-xl shadow-lg active-scale transition">
                      Salvar Ferramenta
                    </button>
                    <button type="button" onClick={() => setMostrarCadastro(false)} className="px-6 py-4 bg-gray-400 text-white rounded-2xl font-bold text-xl active-scale transition">
                      Cancelar
                    </button>
                  </div>
                </form>
              </div>
            )}

            {/* Movimentação Rápida */}
            {mostrarMovRapid && (
              <div className="bg-white rounded-[30px] shadow-xl p-5 mb-4 border-2 border-blue-200">
                <h2 className="text-2xl font-bold text-slate-800 mb-4">Movimentação Rápida</h2>
                <div className="space-y-4">
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Código da Ferramenta</label>
                    <input
                      type="text"
                      placeholder="Ex: FT-001 (ou escanear QR)"
                      value={movCodigo}
                      onChange={(e) => setMovCodigo(e.target.value.toUpperCase())}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                    />
                  </div>
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Nome do Colaborador</label>
                    <input
                      type="text"
                      placeholder="Ex: João Silva"
                      value={movUsuario}
                      onChange={(e) => setMovUsuario(e.target.value)}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                    />
                  </div>
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Tipo de Movimentação</label>
                    <select
                      value={movTipo}
                      onChange={(e) => setMovTipo(e.target.value)}
                      className="w-full border-2 border-slate-200 rounded-2xl p-4 text-lg"
                    >
                      <option>Retirada</option>
                      <option>Devolução</option>
                    </select>
                  </div>
                  <div>
                    <label className="block text-gray-700 font-bold mb-2">Observações (avarias na devolução)</label>
                    <textarea
                      placeholder="Descreva avarias, se houver (apenas devolução)"
                      value={movObservacoes}
                      onChange={(e) => setMovObservacoes(e.target.value)}
                      className="w-full border-2 border-slate-200 rounded-2xl p-3 text-lg"
                      rows="3"
                    />
                  </div>
                  <button
                    onClick={registrarMovimentacao}
                    className="w-full bg-blue-900 text-white py-5 rounded-2xl font-bold text-xl shadow-lg active-scale transition"
                  >
                    Registrar Movimentação
                  </button>
                  <button
                    onClick={() => { setMostrarMovRapid(false); setMovObservacoes(""); }}
                    className="w-full bg-gray-400 text-white py-3 rounded-2xl font-bold text-lg active-scale transition"
                  >
                    Fechar
                  </button>
                </div>
              </div>
            )}

            {/* Modal Histórico */}
            {mostrarHistorico && (
              <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
                <div className="bg-white rounded-3xl shadow-2xl p-6 max-w-2xl w-full max-h-[80vh] overflow-y-auto">
                  <div className="flex justify-between items-center mb-4">
                    <h2 className="text-2xl font-bold text-slate-800">Histórico - {historicoId}</h2>
                    <button 
                      onClick={() => setMostrarHistorico(false)}
                      className="bg-red-600 text-white px-4 py-2 rounded-xl font-bold"
                    >
                      Fechar
                    </button>
                  </div>
                  {historicoFiltrado.length === 0 ? (
                    <p className="text-gray-500 text-center py-8">Nenhum histórico registrado para esta ferramenta.</p>
                  ) : (
                    <div className="space-y-3">
                      {historicoFiltrado.map((h, idx) => (
                        <div key={idx} className="bg-slate-100 rounded-2xl p-4">
                          <div className="flex justify-between items-start mb-2">
                            <span className={`px-3 py-1 rounded-full text-sm font-bold ${
                              h.tipo === "Retirada" ? "bg-green-100 text-green-700" : "bg-orange-100 text-orange-700"
                            }`}>
                              {h.tipo}
                            </span>
                            <span className="text-gray-500 text-sm">{h.data}</span>
                          </div>
                          <p className="text-slate-800 font-bold text-lg">Usuário: {h.usuario}</p>
                          <p className="text-gray-600">Setor: {h.setor}</p>
                          {h.observacoes && (
                            <p className="text-red-600 mt-2 font-semibold">Observações: {h.observacoes}</p>
                          )}
                        </div>
                      ))}
                    </div>
                  )}
                </div>
              </div>
            )}

            {/* Mensagem quando não há ferramentas */}
            {ferramentas.length === 0 && !mostrarCadastro && (
              <div className="bg-white rounded-[30px] shadow-lg p-8 text-center mb-4">
                <p className="text-2xl text-gray-500 mb-4">Nenhuma ferramenta cadastrada</p>
                <button
                  onClick={() => setMostrarCadastro(true)}
                  className="bg-blue-900 text-white px-8 py-4 rounded-2xl font-bold text-xl shadow-lg active-scale transition"
                >
                  Cadastrar Primeira Ferramenta
                </button>
              </div>
            )}

            {/* Lista de ferramentas */}
            <div className="space-y-4">
              {ferramentas.map((item, index) => (
                <div key={index} className="bg-white rounded-[30px] shadow-lg p-4 border border-slate-200">
                  <div className="flex gap-4">
                    <div className="flex flex-col items-center min-w-[120px]">
                      <img
                        src={gerarQRCode(item.id)}
                        alt="QR Code"
                        className="w-28 h-28 rounded-2xl border"
                      />
                      <p className="font-bold text-slate-700 mt-2 text-lg">{item.id}</p>
                      <button
                        onClick={() => deletarFerramenta(item.id)}
                        className="text-red-600 text-sm mt-2 hover:underline"
                      >
                        Excluir
                      </button>
                    </div>

                    <div className="flex-1">
                      <div className="flex items-start justify-between gap-2">
                        <div>
                          <h2 className="text-2xl font-bold text-slate-800">{item.nome}</h2>
                          <p className="text-gray-500 mt-1 text-lg">Setor: {item.setor}</p>
                        </div>
                        <span className={`px-4 py-2 rounded-full text-sm font-bold ${
                          item.status === "Disponível"
                            ? "bg-green-100 text-green-700"
                            : item.status === "Em Uso"
                            ? "bg-orange-100 text-orange-700"
                            : "bg-red-100 text-red-700"
                        }`}>
                          {item.status}
                        </span>
                      </div>

                      {item.observacoes && (
                        <div className="mt-2 bg-red-50 border border-red-200 rounded-xl p-2">
                          <p className="text-red-700 text-sm font-semibold">Observações:</p>
                          <p className="text-red-600 text-sm">{item.observacoes}</p>
                        </div>
                      )}

                      <div className="mt-4 bg-slate-100 rounded-2xl p-3">
                        <p className="text-gray-500 text-sm">Responsável</p>
                        <p className="text-xl font-semibold text-slate-800">{item.responsavel}</p>
                      </div>

                      <div className="grid grid-cols-2 md:grid-cols-3 gap-3 mt-4">
                        <button
                          className={`py-4 rounded-2xl font-bold text-lg active-scale transition ${
                            item.status === "Disponível" 
                              ? "bg-green-600 text-white" 
                              : "bg-gray-400 text-gray-200 cursor-not-allowed"
                          }`}
                          disabled={item.status !== "Disponível"}
                          onClick={() => {
                            setMovCodigo(item.id);
                            setMovTipo("Retirada");
                            setMovObservacoes("");
                            setMostrarMovRapid(true);
                          }}
                        >
                          Retirar
                        </button>
                        <button
                          className={`py-4 rounded-2xl font-bold text-lg active-scale transition ${
                            item.status === "Em Uso" 
                              ? "bg-blue-700 text-white" 
                              : "bg-gray-400 text-gray-200 cursor-not-allowed"
                          }`}
                          disabled={item.status !== "Em Uso"}
                          onClick={() => {
                            setMovCodigo(item.id);
                            setMovTipo("Devolução");
                            setMovObservacoes(item.observacoes || "");
                            setMostrarMovRapid(true);
                          }}
                        >
                          Devolver
                        </button>
                        <button 
                          className="bg-slate-700 text-white py-4 rounded-2xl font-bold text-lg active-scale transition"
                          onClick={() => verHistoricoFerramenta(item.id, item.nome)}
                        >
                          Histórico
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<ControleFerramentasQRCode />);
  </script>
</body>
</html>
