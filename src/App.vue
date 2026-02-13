<script setup lang="ts">
import { ref } from 'vue';
import confetti from 'canvas-confetti';

interface RespostaConvidado {
  nome: string;
  vai: boolean;
}

const nomeDigitado = ref<string>('');
const mensagemStatus = ref<string>('');

const dispararConfetes = () => {
  const duration = 3000; 
  const animationEnd = Date.now() + duration;
  const defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 999 };

  const randomInRange = (min: number, max: number) => Math.random() * (max - min) + min;

  const interval: any = setInterval(function() {
    const timeLeft = animationEnd - Date.now();

    if (timeLeft <= 0) {
      return clearInterval(interval);
    }

    const particleCount = 50 * (timeLeft / duration);
    
    confetti({
      ...defaults,
      particleCount,
      origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 }
    });
    
    confetti({
      ...defaults,
      particleCount,
      origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 }
    });
  }, 250);
};

const enviarResposta = async (decisao: boolean) => {
  if (!nomeDigitado.value.trim()) {
    alert("Por favor, digite seu nome!");
    return;
  }

  const payload: RespostaConvidado = {
    nome: nomeDigitado.value,
    vai: decisao
  };

  try {
    const url = import.meta.env.VITE_BACK_URL
    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    if (response.ok) {
      mensagemStatus.value = decisao ? "Presença confirmada! 🎉" : "Agradecemos o aviso! 👍";
      
      if (decisao) {
        dispararConfetes();
      }
      
      nomeDigitado.value = ''; 
    } else {
      mensagemStatus.value = "Erro ao registrar. Tente novamente.";
    }
  } catch (error) {
    mensagemStatus.value = "Erro ao conectar com a API. Verifique o CORS.";
  }
};
</script>

<template>
  <div class="card-container"> 
    <div class="card">
      <h1>Confirme sua Presença</h1>
      <p>Digite seu nome e nos informe se você vem!</p>

      <input 
        v-model="nomeDigitado"
        type="text" 
        placeholder="Seu nome "
      />

      <div class="btn-group">
        <button @click="enviarResposta(true)" class="btn-confirm">SIM, VOU!</button>
        <button @click="enviarResposta(false)" class="btn-decline">NÃO POSSO</button>
      </div>

      <p v-if="mensagemStatus" class="status-msg">{{ mensagemStatus }}</p>
    </div>
  </div>
</template>