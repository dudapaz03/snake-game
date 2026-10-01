//=============================
// CONFIGURAÇÕES
//=============================
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const tamanhoQuadrado = 20;
const quantidadeQuadrados = canvas.width / tamanhoQuadrado;

// vel = intervalo inicial (ms), min = intervalo mínimo (ms), mult = multiplicador de pontos
const DIFICULDADES = {
    facil:   { vel: 180, min: 90, mult: 1 },
    normal:  { vel: 140, min: 65, mult: 2 },
    dificil: { vel: 100, min: 45, mult: 3 }
};
const COMIDAS_POR_NIVEL = 5;
const DURACAO_ESPECIAL = 5000;   // ms
const JANELA_COMBO = 3000;       // ms

//=============================
// VARIÁVEIS DO JOGO
//=============================
let cobra, direcao, filaDirecoes, comida, especial;
let pontos, comidas, nivel, combo, ultimaComida, recorde;
let jogoAtivo = false, jogoPausado = false, velocidade, temporizador;

const el = id => document.getElementById(id);

//=============================
// ARMAZENAMENTO (com proteção contra erros)
//=============================
function chaveRecorde() {
    return 'recordeCobrinha_' + el('dificuldade').value + (el('atravessar').checked ? '_livre' : '');
}
function carregarRecorde() {
    try { recorde = Number(localStorage.getItem(chaveRecorde())) || 0; }
    catch (e) { recorde = 0; }
    el('recorde').textContent = recorde;
}
function salvarRecorde() {
    try { localStorage.setItem(chaveRecorde(), recorde); } catch (e) {}
}

//=============================
// SOM (gerado pelo navegador, sem arquivos)
//=============================
let audioCtx;
function beep(freq, dur = 0.1, tipo = 'square') {
    if (!el('som').checked) return;
    try {
        audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = tipo;
        osc.frequency.value = freq;
        gain.gain.setValueAtTime(0.08, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + dur);
    } catch (e) {}
}

//=============================
// PREPARAR / INICIAR
//=============================
function prepararJogo() {
    cobra = [{ x: 10, y: 10 }, { x: 9, y: 10 }, { x: 8, y: 10 }];
    direcao = 'direita';
    filaDirecoes = [];
    pontos = 0; comidas = 0; nivel = 1; combo = 1; ultimaComida = 0;
    especial = null;
    velocidade = DIFICULDADES[el('dificuldade').value].vel;
    gerarComida();
    atualizarPainel();
    carregarRecorde();
}

function iniciarJogo() {
    clearTimeout(temporizador);
    prepararJogo();
    jogoAtivo = true;
    jogoPausado = false;
    el('dificuldade').disabled = true;
    el('atravessar').disabled = true;
    desenhar();
    agendar();
    beep(520, 0.12);
}

// setTimeout em vez de setInterval: a velocidade pode mudar a cada passo
function agendar() {
    clearTimeout(temporizador);
    temporizador = setTimeout(atualizarJogo, velocidade);
}

//=============================
// LOOP PRINCIPAL
//=============================
function atualizarJogo() {
    if (!jogoAtivo || jogoPausado) return;

    if (filaDirecoes.length) direcao = filaDirecoes.shift();

    const cabeca = { ...cobra[0] };
    if (direcao === 'cima') cabeca.y--;
    else if (direcao === 'baixo') cabeca.y++;
    else if (direcao === 'esquerda') cabeca.x--;
    else if (direcao === 'direita') cabeca.x++;

    // Paredes: atravessar (modo livre) ou perder
    const fora = cabeca.x < 0 || cabeca.x >= quantidadeQuadrados ||
                 cabeca.y < 0 || cabeca.y >= quantidadeQuadrados;
    if (fora) {
        if (el('atravessar').checked) {
            cabeca.x = (cabeca.x + quantidadeQuadrados) % quantidadeQuadrados;
            cabeca.y = (cabeca.y + quantidadeQuadrados) % quantidadeQuadrados;
        } else {
            fimDeJogo(false);
            return;
        }
    }

    const agora = Date.now();
    if (especial && agora > especial.expira) especial = null;

    const comeuComida = cabeca.x === comida.x && cabeca.y === comida.y;
    const comeuEspecial = especial && cabeca.x === especial.x && cabeca.y === especial.y;

    // A ponta da cauda sai do lugar neste passo, então só conta como colisão se a cobra crescer
    const corpo = (comeuComida || comeuEspecial) ? cobra : cobra.slice(0, -1);
    if (corpo.some(p => p.x === cabeca.x && p.y === cabeca.y)) {
        fimDeJogo(false);
        return;
    }

    cobra.unshift(cabeca);

    if (comeuComida) {
        combo = (agora - ultimaComida < JANELA_COMBO) ? Math.min(combo + 1, 5) : 1;
        ultimaComida = agora;
        pontos += 10 * DIFICULDADES[el('dificuldade').value].mult * combo;
        comidas++;
        beep(660 + combo * 60, 0.1);

        if (comidas % COMIDAS_POR_NIVEL === 0) subirNivel();

        if (cobra.length >= quantidadeQuadrados * quantidadeQuadrados) {
            atualizarPainel();
            fimDeJogo(true);
            return;
        }
        gerarComida();
        if (!especial && Math.random() < 0.3) gerarEspecial();
        atualizarPainel();
    } else if (comeuEspecial) {
        pontos += 50 * DIFICULDADES[el('dificuldade').value].mult;
        especial = null;
        beep(880, 0.08); setTimeout(() => beep(1175, 0.15), 80);
        atualizarPainel();
    } else {
        cobra.pop();
    }

    desenhar();
    agendar();
}

function subirNivel() {
    const d = DIFICULDADES[el('dificuldade').value];
    nivel++;
    velocidade = Math.max(d.min, d.vel - (nivel - 1) * 8);
    beep(440, 0.1); setTimeout(() => beep(660, 0.1), 100); setTimeout(() => beep(880, 0.15), 200);
}

//=============================
// PAINEL E RECORDE
//=============================
function atualizarPainel() {
    el('pontos').textContent = pontos;
    el('nivel').textContent = nivel;
    el('combo').textContent = 'x' + combo;
    if (pontos > recorde) {
        recorde = pontos;           // atualiza a variável, não só o texto
        el('recorde').textContent = recorde;
        salvarRecorde();
    }
}

//=============================
// FIM DE JOGO
//=============================
function fimDeJogo(venceu) {
    jogoAtivo = false;
    clearTimeout(temporizador);
    el('dificuldade').disabled = false;
    el('atravessar').disabled = false;

    if (!venceu) {
        canvas.classList.remove('tremer');
        void canvas.offsetWidth;        // reinicia a animação
        canvas.classList.add('tremer');
        beep(200, 0.3, 'sawtooth');
    } else {
        beep(1000, 0.4);
    }
    desenhar();
    mensagem(venceu ? 'VOCÊ VENCEU!' : 'FIM DE JOGO', `Pontuação: ${pontos}  |  Nível: ${nivel}`,
             'Pressione Enter para jogar de novo');
}

//=============================
// DESENHO
//=============================
function retArredondado(x, y, w, h, r) {
    ctx.beginPath();
    if (ctx.roundRect) ctx.roundRect(x, y, w, h, r); else ctx.rect(x, y, w, h);
    ctx.fill();
}

function desenhar() {
    const T = tamanhoQuadrado;

    // Fundo em xadrez suave
    for (let x = 0; x < quantidadeQuadrados; x++) {
        for (let y = 0; y < quantidadeQuadrados; y++) {
            ctx.fillStyle = (x + y) % 2 ? '#1f2937' : '#222c3b';
            ctx.fillRect(x * T, y * T, T, T);
        }
    }

    // Maçã
    const cx = comida.x * T + T / 2, cy = comida.y * T + T / 2;
    ctx.fillStyle = '#e8384f';
    ctx.beginPath(); ctx.arc(cx, cy, T / 2 - 2, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#5ddc7a';
    ctx.fillRect(cx - 1, cy - T / 2 + 1, 2, 4);

    // Estrela dourada com anel que encolhe até sumir
    if (especial) {
        const ex = especial.x * T + T / 2, ey = especial.y * T + T / 2;
        const restante = Math.max(0, (especial.expira - Date.now()) / DURACAO_ESPECIAL);
        ctx.fillStyle = '#ffd34d';
        ctx.beginPath();
        for (let i = 0; i < 10; i++) {
            const r = i % 2 ? 4 : 9, a = -Math.PI / 2 + i * Math.PI / 5;
            ctx.lineTo(ex + Math.cos(a) * r, ey + Math.sin(a) * r);
        }
        ctx.closePath(); ctx.fill();
        ctx.strokeStyle = 'rgba(255, 211, 77, .6)';
        ctx.lineWidth = 2;
        ctx.beginPath(); ctx.arc(ex, ey, 10, -Math.PI / 2, -Math.PI / 2 + Math.PI * 2 * restante); ctx.stroke();
    }

    // Cobra: degradê da cabeça para a cauda
    cobra.forEach((p, i) => {
        const t = i / Math.max(1, cobra.length - 1);
        ctx.fillStyle = i === 0 ? '#9cf0ae' : `hsl(135, ${65 - t * 15}%, ${50 - t * 18}%)`;
        retArredondado(p.x * T + 1, p.y * T + 1, T - 2, T - 2, i === 0 ? 7 : 5);
    });

    // Olhos, conforme a direção
    const h = cobra[0];
    const olhos = {
        direita:  [[14, 5], [14, 14]], esquerda: [[5, 5], [5, 14]],
        cima:     [[5, 5], [14, 5]],   baixo:    [[5, 14], [14, 14]]
    }[direcao];
    ctx.fillStyle = '#10141b';
    olhos.forEach(([ox, oy]) => { ctx.beginPath(); ctx.arc(h.x * T + ox, h.y * T + oy, 2, 0, Math.PI * 2); ctx.fill(); });

    // Borda do modo "atravessar paredes"
    if (el('atravessar').checked) {
        ctx.strokeStyle = 'rgba(110, 168, 255, .5)';
        ctx.lineWidth = 2;
        ctx.setLineDash([6, 6]);
        ctx.strokeRect(1, 1, canvas.width - 2, canvas.height - 2);
        ctx.setLineDash([]);
    }
}

function mensagem(titulo, linha1, linha2) {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = 'white';
    ctx.textAlign = 'center';
    ctx.font = 'bold 36px Arial';
    ctx.fillText(titulo, canvas.width / 2, canvas.height / 2 - 10);
    ctx.font = '18px Arial';
    if (linha1) ctx.fillText(linha1, canvas.width / 2, canvas.height / 2 + 30);
    ctx.font = '14px Arial';
    ctx.fillStyle = '#b4c0d1';
    if (linha2) ctx.fillText(linha2, canvas.width / 2, canvas.height / 2 + 60);
}

//=============================
// COMIDA
//=============================
function posicaoLivre() {
    const livres = [];
    for (let x = 0; x < quantidadeQuadrados; x++) {
        for (let y = 0; y < quantidadeQuadrados; y++) {
            const ocupado = cobra.some(p => p.x === x && p.y === y) ||
                            (comida && comida.x === x && comida.y === y);
            if (!ocupado) livres.push({ x, y });
        }
    }
    return livres.length ? livres[Math.floor(Math.random() * livres.length)] : null;
}

function gerarComida() {
    comida = null;
    comida = posicaoLivre();
}

function gerarEspecial() {
    const pos = posicaoLivre();
    if (pos) especial = { ...pos, expira: Date.now() + DURACAO_ESPECIAL };
}

//=============================
// CONTROLES
//=============================
const OPOSTA = { cima: 'baixo', baixo: 'cima', esquerda: 'direita', direita: 'esquerda' };

function mudarDirecao(nova) {
    if (!jogoAtivo || jogoPausado) return;
    // compara com a última direção já enfileirada, para não virar 180° em dois toques rápidos
    const ultima = filaDirecoes.length ? filaDirecoes[filaDirecoes.length - 1] : direcao;
    if (nova === ultima || nova === OPOSTA[ultima]) return;
    if (filaDirecoes.length < 2) filaDirecoes.push(nova);
}

function pausarJogo() {
    if (!jogoAtivo) return;
    jogoPausado = !jogoPausado;
    if (jogoPausado) {
        clearTimeout(temporizador);
        mensagem('PAUSADO', '', 'Espaço ou P para continuar');
    } else {
        desenhar();
        agendar();
    }
}

el('btnIniciar').addEventListener('click', iniciarJogo);
el('btnPausar').addEventListener('click', pausarJogo);

document.querySelectorAll('.tecla[data-dir]').forEach(b => {
    b.addEventListener('pointerdown', e => { e.preventDefault(); mudarDirecao(b.dataset.dir); });
});

// Tira o foco dos botões para o Espaço não "clicar" neles sem querer
document.querySelectorAll('button, select, input').forEach(c => {
    c.addEventListener('change', () => c.blur());
    if (c.tagName === 'BUTTON') c.addEventListener('click', () => c.blur());
});

el('dificuldade').addEventListener('change', () => { prepararJogo(); desenhar(); });
el('atravessar').addEventListener('change', () => { carregarRecorde(); desenhar(); });

const TECLAS = {
    ArrowUp: 'cima', w: 'cima', W: 'cima',
    ArrowDown: 'baixo', s: 'baixo', S: 'baixo',
    ArrowLeft: 'esquerda', a: 'esquerda', A: 'esquerda',
    ArrowRight: 'direita', d: 'direita', D: 'direita'
};

document.addEventListener('keydown', e => {
    if (e.target.tagName === 'SELECT') return;
    if (TECLAS[e.key]) {
        e.preventDefault();
        mudarDirecao(TECLAS[e.key]);
    } else if (e.key === ' ' || e.key === 'p' || e.key === 'P') {
        e.preventDefault();
        pausarJogo();
    } else if (e.key === 'Enter' && !jogoAtivo) {
        e.preventDefault();
        iniciarJogo();
    }
});

// Deslizar o dedo no celular
let toqueInicio = null;
canvas.addEventListener('touchstart', e => {
    const t = e.touches[0];
    toqueInicio = { x: t.clientX, y: t.clientY };
}, { passive: true });
canvas.addEventListener('touchend', e => {
    if (!toqueInicio) return;
    const t = e.changedTouches[0];
    const dx = t.clientX - toqueInicio.x, dy = t.clientY - toqueInicio.y;
    toqueInicio = null;
    if (Math.max(Math.abs(dx), Math.abs(dy)) < 25) return;
    if (Math.abs(dx) > Math.abs(dy)) mudarDirecao(dx > 0 ? 'direita' : 'esquerda');
    else mudarDirecao(dy > 0 ? 'baixo' : 'cima');
}, { passive: true });

//=============================
// TELA INICIAL
//=============================
prepararJogo();
desenhar();
mensagem('COBRINHA', 'Escolha a dificuldade', 'Clique em Iniciar ou pressione Enter');