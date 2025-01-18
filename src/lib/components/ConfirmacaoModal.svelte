<script>
  import { fade } from 'svelte/transition';
  export let mensagem = 'Tem certeza que deseja excluir este contato?';
  export let onConfirmar;
  export let onCancelar;
</script>

<div class="modal" transition:fade>
  <div class="modal-content">
    <button class="fechar" on:click={onCancelar}>
      <i class="fas fa-times"></i>
    </button>
    
    <h2>Confirmação</h2>
    <p>{mensagem}</p>
    
    <div class="botoes">
      <button class="cancelar" on:click={onCancelar}>
        Cancelar
      </button>
      <button class="confirmar" on:click={onConfirmar}>
        Confirmar
      </button>
    </div>
  </div>
</div>

<style>
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
  
  h2 {
    margin: 0 0 1rem 0;
    color: #333;
  }
  
  p {
    margin-bottom: 2rem;
    color: #666;
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
  
  .botoes {
    display: flex;
    gap: 1rem;
    justify-content: flex-end;
  }
  
  button {
    padding: 0.8rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
  }
  
  .confirmar {
    background-color: #f44336;
    color: white;
  }
  
  .cancelar {
    background-color: #9e9e9e;
    color: white;
  }
</style> 