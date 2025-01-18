<script>
  import ContatoForm from '$lib/components/ContatoForm.svelte';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import ConfirmacaoModal from '$lib/components/ConfirmacaoModal.svelte';
  import { db } from '$lib/services/db';
  
  let contatos = [];
  let contatoSelecionado = null;
  let mostrarFormulario = false;
  let filtro = '';
  let contatoParaExcluir = null;
  let menuConfigAberto = false;
  let deferredPrompt;
  let podeInstalar = false;
  
  onMount(async () => {
    try {
      await db.init();
      contatos = await db.getAll();
      
      console.log('Verificando suporte a PWA...'); // Debug
      
      // Adiciona o listener para instalação do app
      window.addEventListener('beforeinstallprompt', (e) => {
        console.log('beforeinstallprompt disparado!', e); // Debug detalhado
        e.preventDefault();
        deferredPrompt = e;
        podeInstalar = true;
      });
      
      // Verifica se já está instalado
      window.addEventListener('appinstalled', () => {
        console.log('App instalado com sucesso!');
        podeInstalar = false;
        deferredPrompt = null;
      });
      
      // Verifica service worker
      if ('serviceWorker' in navigator) {
        console.log('Service Worker suportado');
        navigator.serviceWorker.ready.then(() => {
          console.log('Service Worker está ativo');
        });
      }
      
    } catch (error) {
      console.error('Erro ao carregar contatos:', error);
    }
  });
  
  async function adicionarContato(event) {
    try {
      const novoContato = event.detail;
      if (!novoContato.id) {
        novoContato.id = Date.now().toString();
        await db.add(novoContato);
        contatos = [...contatos, novoContato];
      } else {
        await db.update(novoContato);
        contatos = contatos.map(c => 
          c.id === novoContato.id ? novoContato : c
        );
      }
      mostrarFormulario = false;
    } catch (error) {
      console.error('Erro ao salvar contato:', error);
    }
  }
  
  function editarContato(contato) {
    contatoSelecionado = { ...contato };
    mostrarFormulario = true;
  }
  
  function excluirContato(contato) {
    contatoParaExcluir = contato;
  }
  
  async function confirmarExclusao() {
    try {
      await db.delete(contatoParaExcluir.id);
      contatos = contatos.filter(c => c.id !== contatoParaExcluir.id);
      contatoParaExcluir = null;
    } catch (error) {
      console.error('Erro ao excluir contato:', error);
    }
  }
  
  function exportarCSV() {
    const headers = 'Nome;Telefone;Email;Endereco\n';
    const csvContent = headers + contatos.map(c => 
      `${c.nome};${c.telefone};${c.email};${c.endereco}`
    ).join('\n');
    
    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'contatos.csv';
    link.click();
  }
  
  async function importarCSV(event) {
    const file = event.target.files[0];
    const reader = new FileReader();
    
    reader.onload = async (e) => {
      const text = e.target.result;
      const linhas = text.split('\n');
      linhas.shift(); // Remove o cabeçalho
      
      const novosContatos = linhas
        .filter(linha => linha.trim())
        .map(linha => {
          const [nome, telefone, email, endereco] = linha.split(';');
          return {
            id: Date.now().toString() + Math.random(),
            nome: nome?.trim() || '',
            telefone: telefone?.trim() || '',
            email: email?.trim() || '',
            endereco: endereco?.trim() || ''
          };
        });
      
      // Adiciona cada contato no banco de dados
      for (const contato of novosContatos) {
        await db.add(contato);
      }
      
      contatos = [...contatos, ...novosContatos];
      menuConfigAberto = false;
    };
    
    reader.readAsText(file);
  }
  
  function toggleMenuConfig() {
    menuConfigAberto = !menuConfigAberto;
  }
  
  // Fecha o menu quando clicar fora
  function handleClickFora(event) {
    const configMenu = document.querySelector('.config-menu');
    if (configMenu && !configMenu.contains(event.target)) {
      menuConfigAberto = false;
    }
  }
  
  $: contatosFiltrados = contatos.filter(contato =>
    contato.nome.toLowerCase().includes(filtro.toLowerCase()) ||
    contato.email.toLowerCase().includes(filtro.toLowerCase()) ||
    contato.telefone.includes(filtro)
  );
  
  async function instalarApp() {
    if (!deferredPrompt) return;
    
    deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    
    if (outcome === 'accepted') {
      console.log('App instalado');
      podeInstalar = false;
    }
    
    deferredPrompt = null;
  }
</script>

<svelte:window on:click={handleClickFora}/>

<main>
  <h1>POC CRUD Agenda</h1>
  
  <div class="barra-superior">
    <div class="busca">
      <input
        type="text"
        placeholder="Buscar contatos..."
        bind:value={filtro}
      >
    </div>
    <div class="acoes">
      <button class="novo" on:click={() => {
        contatoSelecionado = null;
        mostrarFormulario = true;
      }}>
        <i class="fas fa-plus"></i> Novo Contato
      </button>
      <div class="config-menu">
        <button class="config" type="button" on:click|stopPropagation={toggleMenuConfig}>
          <i class="fas fa-cog"></i>
          <i class="fas fa-chevron-down"></i>
        </button>
        {#if menuConfigAberto}
          <div class="menu-opcoes" transition:fade>
            <button on:click={exportarCSV}>
              <i class="fas fa-download"></i> Exportar CSV
            </button>
            <label class="importar">
              <i class="fas fa-upload"></i> Importar CSV
              <input
                type="file"
                accept=".csv"
                on:change={importarCSV}
                style="display: none"
              >
            </label>
            {#if podeInstalar}
              <button on:click={instalarApp}>
                <i class="fas fa-mobile-alt"></i> Instalar Aplicativo
              </button>
            {/if}
          </div>
        {/if}
      </div>
    </div>
  </div>
  
  {#if mostrarFormulario}
    <div class="modal" transition:fade>
      <div class="modal-content">
        <button class="fechar" on:click={() => mostrarFormulario = false}>
          <i class="fas fa-times"></i>
        </button>
        <ContatoForm
          contato={contatoSelecionado || {
            id: '',
            nome: '',
            telefone: '',
            email: '',
            endereco: ''
          }}
          on:salvar={adicionarContato}
          on:cancelar={() => mostrarFormulario = false}
        />
      </div>
    </div>
  {/if}
  
  <div class="grid-contatos">
    {#each contatosFiltrados as contato}
      <div class="card-contato" transition:fade>
        <div class="acoes-card">
          <button 
            class="editar"
            on:click={() => editarContato(contato)}
          >
            <i class="fas fa-edit"></i>
          </button>
          <button 
            class="excluir"
            on:click={() => excluirContato(contato)}
          >
            <i class="fas fa-trash"></i>
          </button>
        </div>
        <div class="info-contato">
          <h3>{contato.nome}</h3>
          <p><i class="fas fa-phone"></i> {contato.telefone}</p>
          <p><i class="fas fa-envelope"></i> {contato.email}</p>
          <p><i class="fas fa-map-marker-alt"></i> {contato.endereco}</p>
        </div>
      </div>
    {/each}
  </div>
  
  {#if contatoParaExcluir}
    <ConfirmacaoModal
      mensagem={`Tem certeza que deseja excluir o contato "${contatoParaExcluir.nome}"?`}
      onConfirmar={confirmarExclusao}
      onCancelar={() => contatoParaExcluir = null}
    />
  {/if}
</main>

<style>
  @import 'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css';
  
  main {
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }
  
  h1 {
    text-align: center;
    color: #333;
    margin-bottom: 2rem;
  }
  
  .barra-superior {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
    gap: 1rem;
  }
  
  .busca {
    flex: 1;
  }
  
  .busca input {
    width: 100%;
    height: 2.8rem;
    padding: 0 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
  }
  
  .acoes {
    display: flex;
    gap: 1rem;
    align-items: center;
  }
  
  .grid-contatos {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 1.5rem;
  }
  
  .card-contato {
    position: relative;
    background: white;
    border-radius: 8px;
    padding: 1.5rem;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    transition: transform 0.2s;
  }
  
  .card-contato:hover {
    transform: translateY(-5px);
  }
  
  .acoes-card {
    position: absolute;
    top: 1rem;
    right: 1rem;
    display: flex;
    gap: 0.5rem;
  }
  
  .info-contato h3 {
    margin: 0 0 1rem 0;
    color: #333;
  }
  
  .info-contato p {
    margin: 0.5rem 0;
    color: #666;
  }
  
  .info-contato i {
    width: 20px;
    color: #2196F3;
  }
  
  button {
    padding: 0.8rem 1.2rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 1rem;
  }
  
  .novo, .config {
    height: 2.8rem;
    padding: 0 1.2rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 1rem;
    white-space: nowrap;
  }
  
  .novo {
    background-color: #4CAF50;
    color: white;
  }
  
  .config {
    background-color: #2196F3;
    color: white;
    padding: 0 1rem;
  }
  
  .config i {
    font-size: 1rem;
  }
  
  .config i.fa-chevron-down {
    font-size: 0.8rem;
    margin-left: 0.2rem;
  }
  
  .editar, .excluir {
    padding: 0.5rem;
    border-radius: 50%;
  }
  
  .editar {
    background-color: #2196F3;
    color: white;
  }
  
  .excluir {
    background-color: #f44336;
    color: white;
  }
  
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }
  
  .modal-content {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    position: relative;
    width: 90%;
    max-width: 500px;
  }
  
  .fechar {
    position: absolute;
    top: 1rem;
    right: 1rem;
    background: none;
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    color: #666;
  }
  
  .config-menu {
    position: relative;
  }
  
  .menu-opcoes {
    position: absolute;
    top: 100%;
    right: 0;
    background: white;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    padding: 0.5rem;
    min-width: 180px;
    z-index: 100;
    margin-top: 0.5rem;
  }
  
  .menu-opcoes button,
  .menu-opcoes label {
    background: none;
    color: #333;
    padding: 0.8rem 1rem;
    cursor: pointer;
    text-align: left;
    white-space: nowrap;
    width: 100%;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    border: none;
    border-radius: 4px;
  }
  
  .menu-opcoes button:hover,
  .menu-opcoes label:hover {
    background: #f5f5f5;
  }
  
  @media (max-width: 768px) {
    .barra-superior {
      flex-direction: column;
    }
    
    .grid-contatos {
      grid-template-columns: 1fr;
    }
  }
</style>
