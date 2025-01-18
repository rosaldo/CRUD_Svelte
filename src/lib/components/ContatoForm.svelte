<script>
  import { createEventDispatcher } from 'svelte';
  
  export let contato = {
    id: '',
    nome: '',
    telefone: '',
    email: '',
    endereco: ''
  };
  
  const dispatch = createEventDispatcher();
  
  let erros = {
    nome: '',
    telefone: '',
    email: ''
  };
  
  function validarForm() {
    let valido = true;
    erros = {
      nome: '',
      telefone: '',
      email: ''
    };
    
    if (!contato.nome || contato.nome.length < 3) {
      erros.nome = 'Nome deve ter pelo menos 3 caracteres';
      valido = false;
    }
    
    const telRegex = /^\([1-9]{2}\) (?:(?:9[1-9]\d{3})|(?:[2-8]\d{3}))-\d{4}$/;
    if (!telRegex.test(contato.telefone)) {
      erros.telefone = 'Telefone inválido. Use o formato (99) 9999-9999 ou (99) 99999-9999';
      valido = false;
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(contato.email)) {
      erros.email = 'Email inválido';
      valido = false;
    }
    
    return valido;
  }
  
  function handleSubmit() {
    if (validarForm()) {
      dispatch('salvar', contato);
    }
  }
  
  function handleCancelar() {
    dispatch('cancelar');
  }
  
  function formatarTelefone(valor) {
    // Remove tudo que não for número
    let numeros = valor.replace(/\D/g, '');
    
    // Se não tiver DDD ainda, apenas retorna os números (limitado a 2 dígitos)
    if (numeros.length <= 2) {
      return numeros.slice(0, 2);
    }
    
    // Pega o DDD e o resto dos números
    const ddd = numeros.slice(0, 2);
    let resto = numeros.slice(2);
    
    // Verifica se é celular (começa com 9)
    const eCelular = resto.startsWith('9');
    
    // Limita o tamanho do resto baseado no tipo
    if (eCelular) {
      resto = resto.slice(0, 9); // 9 dígitos para celular
    } else {
      resto = resto.slice(0, 8); // 8 dígitos para fixo
      // Se começar com 9, remove o 9
      if (resto.startsWith('9')) {
        resto = resto.slice(1);
      }
    }
    
    // Remonta o número com DDD
    numeros = ddd + resto;
    
    // Formata o número
    let telefoneFormatado = `(${ddd})`;
    
    if (eCelular) {
      // Formato celular: (99) 99999-9999
      if (resto.length > 5) {
        telefoneFormatado += ` ${resto.slice(0, 5)}-${resto.slice(5)}`;
      } else {
        telefoneFormatado += ` ${resto}`;
      }
    } else {
      // Formato fixo: (99) 9999-9999
      if (resto.length > 4) {
        telefoneFormatado += ` ${resto.slice(0, 4)}-${resto.slice(4)}`;
      } else {
        telefoneFormatado += ` ${resto}`;
      }
    }
    
    return telefoneFormatado;
  }
  
  function handleTelefoneInput(event) {
    const novoValor = formatarTelefone(event.target.value);
    contato.telefone = novoValor;
  }
</script>

<form on:submit|preventDefault={handleSubmit} class="formulario">
  <div class="campo">
    <label for="nome">Nome:</label>
    <input 
      type="text" 
      id="nome" 
      bind:value={contato.nome}
      class={erros.nome ? 'erro' : ''}
    >
    {#if erros.nome}
      <span class="erro-msg">{erros.nome}</span>
    {/if}
  </div>
  
  <div class="campo">
    <label for="telefone">Telefone:</label>
    <input 
      type="tel" 
      id="telefone" 
      bind:value={contato.telefone}
      on:input={handleTelefoneInput}
      class={erros.telefone ? 'erro' : ''}
    >
    {#if erros.telefone}
      <span class="erro-msg">{erros.telefone}</span>
    {/if}
  </div>
  
  <div class="campo">
    <label for="email">Email:</label>
    <input 
      type="email" 
      id="email" 
      bind:value={contato.email}
      class={erros.email ? 'erro' : ''}
    >
    {#if erros.email}
      <span class="erro-msg">{erros.email}</span>
    {/if}
  </div>
  
  <div class="campo">
    <label for="endereco">Endereço:</label>
    <input type="text" id="endereco" bind:value={contato.endereco}>
  </div>
  
  <div class="botoes">
    <button type="submit" class="salvar">
      {contato.id ? 'Atualizar' : 'Adicionar'} Contato
    </button>
    <button type="button" class="cancelar" on:click={handleCancelar}>
      Cancelar
    </button>
  </div>
</form>

<style>
  .formulario {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  
  .campo {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  
  label {
    font-weight: 500;
    color: #333;
  }
  
  input {
    padding: 0.8rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
  }
  
  input.erro {
    border-color: #f44336;
  }
  
  .erro-msg {
    color: #f44336;
    font-size: 0.875rem;
  }
  
  .botoes {
    display: flex;
    gap: 1rem;
    margin-top: 1rem;
  }
  
  button {
    flex: 1;
    padding: 0.8rem;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
  }
  
  .salvar {
    background-color: #4CAF50;
    color: white;
  }
  
  .cancelar {
    background-color: #f44336;
    color: white;
  }
  
  button:hover {
    opacity: 0.9;
  }
</style> 