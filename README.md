# ca-a_atomos
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Caça Átomos — Recreio Arcade</title>
<style>
/* ======================================================================
   Caça Átomos  —  Recreio Arcade
   Labirinto arcade (inspirado no Pac-Man) com desafios de Química.
   Tudo (HTML + CSS + JavaScript) está neste único arquivo.
   ====================================================================== */

:root{
  --fundo:#0a0f1e;
  --parede:#3a49ff;
  --azul:#4de2ff;
  --limao:#b6ff3d;
  --amarelo:#ffe066;
  --caixa:#151d3a;
  --erro:#ff5a5a;
}

*{box-sizing:border-box;}
html,body{height:100%;}

body{
  margin:0;
  padding:16px env(safe-area-inset-right,0) calc(16px + env(safe-area-inset-bottom,0)) env(safe-area-inset-left,0);
  padding-top:calc(16px + env(safe-area-inset-top,0));
  background:radial-gradient(circle at 50% 0%, #17224d 0%, #070a15 70%);
  color:#e8f2ff;
  font-family:"Courier New",Courier,monospace;
  display:flex;
  justify-content:center;
  align-items:flex-start;
}

.gabinete{
  width:100%;
  max-width:780px;
  background:var(--fundo);
  border:4px solid var(--azul);
  border-radius:10px;
  box-shadow:0 0 0 4px #050812, 0 18px 40px rgba(0,0,0,.6);
  padding:12px;
}

/* ---------- Placar ---------- */
.hud{display:flex; flex-wrap:wrap; gap:8px; margin-bottom:10px;}
.hud div{
  flex:1 1 110px;
  background:var(--caixa);
  border:2px solid #2b3a7a;
  padding:6px 8px;
  font-size:13px;
  letter-spacing:1px;
}
.hud b{display:block; font-size:17px; color:var(--amarelo); letter-spacing:2px;}
#hudEnergia{color:var(--limao);}

/* ---------- Tela ---------- */
.tela{
  position:relative;
  width:100%;
  aspect-ratio:504/312;
  background:#04060f;
  border:3px solid #2b3a7a;
  overflow:hidden;
}
canvas{display:block; width:100%; height:100%; image-rendering:pixelated;}
.scanlines{
  position:absolute; inset:0; pointer-events:none;
  background:repeating-linear-gradient(to bottom, rgba(0,0,0,.26) 0 2px, rgba(0,0,0,0) 2px 4px);
}

/* ---------- Painéis sobrepostos ---------- */
.painel{
  position:absolute; inset:0;
  display:none;
  flex-direction:column; justify-content:center; align-items:center;
  gap:10px; padding:14px; text-align:center;
  background:rgba(4,6,15,.95);
  overflow:auto;
}
.painel.ativo{display:flex;}
.painel h1{
  margin:0;
  font-size:clamp(19px,4.2vw,32px);
  color:var(--limao);
  letter-spacing:3px;
  text-shadow:3px 3px 0 var(--parede);
}
.painel p{margin:0; font-size:clamp(12px,2.5vw,15px); line-height:1.6; max-width:52ch;}

.pergunta-titulo{margin:0; color:var(--amarelo); font-size:clamp(13px,2.7vw,17px); line-height:1.5; max-width:56ch;}
.alternativas{display:grid; grid-template-columns:1fr; gap:8px; width:100%; max-width:540px;}
@media(min-width:620px){ .alternativas{grid-template-columns:1fr 1fr;} }

button{
  font-family:inherit; font-size:14px; letter-spacing:1px;
  color:#e8f2ff; background:var(--caixa);
  border:2px solid var(--azul); padding:10px 12px;
  cursor:pointer; text-align:left;
}
button:hover{background:#22305f;}
button:focus-visible{outline:3px solid var(--amarelo); outline-offset:2px;}
button:disabled{cursor:default; opacity:.85;}
button.certa{background:#1d5c33; border-color:var(--limao);}
button.errada{background:#5d1c28; border-color:var(--erro);}
button.selecionada{outline:3px solid var(--amarelo); outline-offset:2px;}

.botao-grande{
  text-align:center; background:var(--limao); border-color:#fff;
  color:#0a0f1e; font-weight:bold; padding:12px 26px;
}
.botao-grande:hover{background:#c9ff6b;}

.explicacao{
  background:var(--caixa); border-left:5px solid var(--azul);
  padding:10px 12px; font-size:13px; line-height:1.6;
  text-align:left; max-width:540px;
}

/* ---------- Direcional de toque ---------- */
.dpad{
  display:grid;
  grid-template-columns:repeat(3,64px);
  grid-template-rows:repeat(2,50px);
  gap:6px;
  justify-content:center;
  margin-top:10px;
}
.dpad button{font-size:20px; text-align:center; padding:0; border-radius:8px; user-select:none;}
.dpad .up{grid-column:2; grid-row:1;}
.dpad .left{grid-column:1; grid-row:2;}
.dpad .down{grid-column:2; grid-row:2;}
.dpad .right{grid-column:3; grid-row:2;}

.rodape{margin-top:8px; font-size:11px; color:#8ea2d6; text-align:center; letter-spacing:1px;}
@media(prefers-reduced-motion:reduce){ *{transition:none !important;} }
</style>
</head>
<body>

<div class="gabinete">

  <div class="hud">
    <div>Pontos <b id="hudPontos">0</b></div>
    <div>Energia <b id="hudEnergia">⚡⚡⚡</b></div>
    <div>Fase <b id="hudProgresso">1</b></div>
    <div>Átomos <b id="hudPelotas">0</b></div>
  </div>

  <div class="tela">
    <canvas id="tela" width="504" height="312"></canvas>
    <div class="scanlines"></div>

    <div class="painel ativo" id="painelInicio">
      <h1>Elemento Relâmpago</h1>
      <p>Percorra o laboratório recolhendo átomos e as 8 cápsulas brilhantes.
         Cada cápsula abre um desafio de química. Fuja dos íons instáveis: encostar neles custa energia, assim como errar uma resposta. Pegue a bolinha branca (energizador) e os íons ficam azuis: aí é você quem caça! Limpe os átomos para subir de fase.</p>
      <p>Use as setas para mover. Nas cápsulas, setas ↑ ↓ escolhem a alternativa e Enter ou Espaço confirmam. No celular, use o direcional.</p>
      <button class="botao-grande" id="btComecar">Entrar no laboratório</button>
    </div>

    <div class="painel" id="painelPergunta">
      <p class="pergunta-titulo" id="textoPergunta"></p>
      <div class="alternativas" id="listaAlternativas"></div>
      <div class="explicacao" id="explicacao" hidden></div>
      <button class="botao-grande" id="btContinuar" hidden>Voltar ao labirinto</button>
    </div>

    <div class="painel" id="painelPlot">
      <h1 id="plotTitulo">PLOT TWIST!</h1>
      <p id="plotTexto"></p>
      <button class="botao-grande" id="btPlot">Continuar</button>
    </div>

    <div class="painel" id="painelFim">
      <h1 id="tituloFim">Experimento concluído</h1>
      <p id="resumoFinal"></p>
      <button class="botao-grande" id="btReiniciar">Jogar de novo</button>
    </div>
  </div>

  <div class="dpad">
    <button class="up"    id="btCima"    aria-label="Mover para cima">▲</button>
    <button class="left"  id="btEsq"     aria-label="Mover para a esquerda">◀</button>
    <button class="down"  id="btBaixo"   aria-label="Mover para baixo">▼</button>
    <button class="right" id="btDir"     aria-label="Mover para a direita">▶</button>
  </div>

  <p class="rodape">Recreio Arcade · Química · Elementos e pH</p>
</div>

<script>
/* ======================================================================
   1. AS PERGUNTAS
   ====================================================================== */
const PERGUNTAS = [
  {
    enunciado:"Qual é o símbolo químico do sódio?",
    alternativas:["So","Na","S","Sd"],
    correta:1,
    explicacao:"O símbolo do sódio é Na, que vem do latim natrium. Vários símbolos guardam o nome antigo do elemento."
  },
  {
    enunciado:"O número atômico de um elemento indica a quantidade de:",
    alternativas:["prótons no núcleo","nêutrons no núcleo","elétrons na última camada","ligações que ele faz"],
    correta:0,
    explicacao:"O número atômico (Z) é o número de prótons. Ele identifica o elemento: todo átomo com Z = 6 é carbono."
  },
  {
    enunciado:"Uma solução tem pH igual a 3. Ela é:",
    alternativas:["neutra","básica","ácida","sem classificação"],
    correta:2,
    explicacao:"A escala de pH vai de 0 a 14. Abaixo de 7 a solução é ácida, acima de 7 é básica e exatamente 7 é neutra."
  },
  {
    enunciado:"Qual fórmula representa a molécula de água?",
    alternativas:["HO₂","H₂O","OH₂","H₂O₂"],
    correta:1,
    explicacao:"A água é H₂O: dois átomos de hidrogênio ligados a um de oxigênio. H₂O₂ é a água oxigenada."
  },
  {
    enunciado:"O símbolo Fe pertence a qual elemento?",
    alternativas:["Flúor","Férmio","Fósforo","Ferro"],
    correta:3,
    explicacao:"Fe é o ferro, do latim ferrum. O flúor é F e o fósforo é P."
  },
  {
    enunciado:"O composto NaCl é conhecido no dia a dia como:",
    alternativas:["sal de cozinha","bicarbonato","açúcar","cal virgem"],
    correta:0,
    explicacao:"NaCl é o cloreto de sódio, o sal de cozinha. É formado pela ligação iônica entre sódio e cloro."
  },
  {
    enunciado:"Gás mais abundante na atmosfera da Terra:",
    alternativas:["oxigênio (O₂)","gás carbônico (CO₂)","nitrogênio (N₂)","hidrogênio (H₂)"],
    correta:2,
    explicacao:"Cerca de 78% do ar é nitrogênio (N₂) e cerca de 21% é oxigênio (O₂). O CO₂ não chega a 0,1%."
  },
  {
    enunciado:"Na tabela periódica, o elemento de número atômico 1 é:",
    alternativas:["hélio","hidrogênio","lítio","carbono"],
    correta:1,
    explicacao:"O hidrogênio tem Z = 1: um único próton. É o elemento mais simples e mais abundante do universo."
  }
];

/* ======================================================================
   1B. PERGUNTAS INFINITAS E CURIOSIDADES
   As 7 perguntas acima saem primeiro (embaralhadas); depois o jogo GERA
   perguntas novas a partir da tabela de elementos, sem nunca acabar.
   ====================================================================== */
function embaralhar(a){for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];}return a;}
const R=(a,b)=>a+Math.floor(Math.random()*(b-a+1));
function montar(enun,certa,erradas,exp){
  const alts=embaralhar([certa,...[...new Set(erradas)].filter(x=>x!==certa)].slice(0,4));
  return {enunciado:enun,alternativas:alts,correta:alts.indexOf(certa),explicacao:exp};
}
const ELEM=[[1,"H","hidrogênio"],[2,"He","hélio"],[3,"Li","lítio"],[6,"C","carbono"],[7,"N","nitrogênio"],[8,"O","oxigênio"],[9,"F","flúor"],[11,"Na","sódio"],[12,"Mg","magnésio"],[13,"Al","alumínio"],[14,"Si","silício"],[15,"P","fósforo"],[16,"S","enxofre"],[17,"Cl","cloro"],[19,"K","potássio"],[20,"Ca","cálcio"],[26,"Fe","ferro"],[29,"Cu","cobre"],[47,"Ag","prata"],[79,"Au","ouro"]];
const sorteia=(n,exceto)=>embaralhar(ELEM.filter(e=>e!==exceto)).slice(0,n);
const GERADORES=[
  ()=>{const e=ELEM[R(0,ELEM.length-1)];return montar(`Qual é o símbolo químico de ${e[2]}?`,e[1],sorteia(3,e).map(x=>x[1]),`${e[2]} = ${e[1]}, número atômico ${e[0]}.`);},
  ()=>{const e=ELEM[R(0,ELEM.length-1)];return montar(`O símbolo ${e[1]} representa qual elemento?`,e[2],sorteia(3,e).map(x=>x[2]),`${e[1]} é o símbolo de ${e[2]} (Z = ${e[0]}).`);},
  ()=>{const pH=R(0,14);const c=pH<7?"ácida":pH>7?"básica":"neutra";
    return montar(`Uma solução tem pH ${pH}. Ela é:`,c,["ácida","básica","neutra","sem classificação"],`Abaixo de 7 é ácida, acima de 7 é básica, igual a 7 é neutra.`);},
  ()=>{const e=ELEM[R(0,ELEM.length-1)];return montar(`Quantos prótons tem um átomo de ${e[2]}?`,String(e[0]),sorteia(3,e).map(x=>String(x[0])),`O número de prótons é o número atômico: ${e[2]} tem Z = ${e[0]}.`);},
  ()=>{const i=[["Na",11,1],["K",19,1],["Li",3,1],["Cl",17,-1],["F",9,-1]][R(0,4)];const sinal=i[2]>0?"⁺":"⁻";const el=i[1]-i[2];
    return montar(`Quantos elétrons tem o íon ${i[0]}${sinal}? (Z = ${i[1]})`,String(el),[String(i[1]),String(el+2),String(el-2)],`${i[2]>0?"Cátion perdeu":"Ânion ganhou"} 1 elétron: ${i[1]} ${i[2]>0?"−":"+"} 1 = ${el}.`);}
];
let fila=[];
function proximaPergunta(fase){
  if(fila.length) return fila.shift();
  const usaveis=GERADORES.slice(0,Math.min(GERADORES.length,3+Math.floor(fase/2)));
  return usaveis[R(0,usaveis.length-1)]();
}
// "Plot twists": aparecem quando você captura um íon
const CURIOSIDADES=[
  "O sódio (Na) explode em contato com a água, mas o íon Na⁺ está no seu sal de cozinha. Perder um elétron muda tudo!",
  "Átomo vira ÍON ao ganhar ou perder elétrons: perdeu, vira cátion (+); ganhou, vira ânion (−).",
  "O ouro (Au) quase não reage com nada. Por isso tesouros de milhares de anos continuam brilhando.",
  "Você é feito de átomos forjados em estrelas: carbono, oxigênio e nitrogênio nasceram em explosões estelares.",
  "O hélio foi descoberto no Sol antes de ser achado na Terra. Seu nome vem de helios, que significa Sol.",
  "A água pura tem pH 7, mas a chuva normal é levemente ácida (pH ≈ 5,6) por causa do CO₂ dissolvido.",
  "O ferro (Fe) enferruja ao reagir com oxigênio e água: é uma reação de oxidação.",
  "O flúor (F) é o elemento mais eletronegativo: puxa elétrons como ninguém."
];
let filaPlot=[];
function proximaCuriosidade(){ if(!filaPlot.length) filaPlot=embaralhar(CURIOSIDADES.slice()); return filaPlot.shift(); }

/* ======================================================================
   2. O LABIRINTO
   #  parede     .  átomo (pontinho)     C  cápsula de pergunta
   E  energizador (deixa os íons azuis e comestíveis)
   P  início do jogador                  G  início de um íon
   ====================================================================== */
const MAPA = [
  "#####################",
  "#E...............E..#",
  "#.###.#####.#####.#.#",
  "#...................#",
  "#.###.#.#####.#.###.#",
  "#C..#.....G.....#..C#",
  "#.###.###.#.###.###.#",
  "#...................#",
  "#.###.#.#####.#.###.#",
  "#C..#.#...G...#.#..C#",
  "#.###.#.#.#.#.#.###.#",
  "#E........P........E#",
  "#####################"
];

const CELULA = 24;                  // tamanho de cada quadradinho, em pixels
const COLS = MAPA[0].length;        // 21
const LINS = MAPA.length;           // 13

const tela = document.getElementById("tela");
const ctx  = tela.getContext("2d");
ctx.imageSmoothingEnabled = false;

let grade, jogador, ions, jogo, ultimoTempo;

/* Copia o mapa para uma matriz que pode ser alterada durante a partida */
function montarLabirinto(){
  grade = [];
  const posIons = [];
  let posJogador = {col:1, lin:1};

  for(let l=0; l<LINS; l++){
    grade[l] = [];
    for(let c=0; c<COLS; c++){
      const simbolo = MAPA[l][c];
      if(simbolo === "#")      grade[l][c] = "parede";
      else if(simbolo === ".") grade[l][c] = "atomo";
      else if(simbolo === "C") grade[l][c] = "capsula";
      else if(simbolo === "E") grade[l][c] = "energia";
      else                     grade[l][c] = "vazio";

      if(simbolo === "P"){ posJogador = {col:c, lin:l}; grade[l][c] = "vazio"; }
      if(simbolo === "G"){ posIons.push({col:c, lin:l}); grade[l][c] = "vazio"; }
    }
  }
  return {posJogador, posIons};
}

/* Uma "criatura" anda de célula em célula; t vai de 0 a 1 entre duas células */
function criarCriatura(col, lin, duracao, cor){
  return {
    col, lin,
    antCol:col, antLin:lin,
    t:1,                       // 1 = parado exatamente em cima de uma célula
    dir:{c:0, l:0},
    proxDir:{c:0, l:0},
    duracao,                   // segundos para atravessar uma célula
    cor
  };
}

function ehParede(col, lin){
  if(lin < 0 || lin >= LINS || col < 0 || col >= COLS) return true;
  return grade[lin][col] === "parede";
}

const EXTRAS=[{col:10,lin:7},{col:5,lin:7}];       // íons extras que entram nas fases avançadas
const CORES_IONS=["#ff5a8a","#4de2ff","#ffa64d","#c77dff"];

/* Monta (ou remonta) o labirinto da fase atual. Fases altas = íons mais rápidos e mais numerosos */
function montarFase(){
  const inicio = montarLabirinto();
  jogo.restantes = grade.flat().filter(c => c === "atomo").length;
  jogador = criarCriatura(inicio.posJogador.col, inicio.posJogador.lin, 0.14, "#ffe066");
  jogador.ic = jogador.col; jogador.il = jogador.lin;
  const qtd = Math.min(4, 2 + Math.floor(jogo.fase/2));
  ions = inicio.posIons.concat(EXTRAS).slice(0, qtd).map((p,i)=>{
    const ion = criarCriatura(p.col, p.lin, Math.max(0.125, 0.21 - jogo.fase*0.012 + i*0.015), CORES_IONS[i]);
    ion.ic = p.col; ion.il = p.lin; ion.medo = false;
    return ion;
  });
}

function novoJogo(){
  jogo = {
    estado:"jogando",          // jogando | pergunta | plot | fim
    pontos:0, energia:3, fase:1,
    indice:0,                  // perguntas já respondidas
    acertos:0, errosResposta:0, atomos:0,
    invulneravel:1.5, piscar:0,
    medo:0,                    // segundos restantes de íons azuis
    combo:0,                   // íons comidos no mesmo energizador
    aviso:2, restantes:0, atual:null,
    inicio:Date.now()
  };
  fila = embaralhar(PERGUNTAS.slice());
  montarFase();
  atualizarHud();
}

function proximaFase(){
  jogo.fase++;
  jogo.energia = Math.min(5, jogo.energia + 1);
  jogo.pontos += 500 * (jogo.fase - 1);
  jogo.medo = 0; jogo.aviso = 2.2; jogo.invulneravel = 1.5;
  montarFase();
}

/* ======================================================================
   3. CONTROLES
   ====================================================================== */
function virar(c, l){
  if(!jogo || jogo.estado !== "jogando") return;
  jogador.proxDir = {c, l};
}

// Só o conjunto canônico chega ao jogo: setas, espaço, Enter (e Z/X, não usados aqui).
// event.repeat é ignorado para o auto-repeat do sistema não virar metralhadora de eventos.
document.addEventListener("keydown", (e)=>{
  const k = e.key;
  if(e.repeat) return;

  if(jogo && jogo.estado === "pergunta"){
    // Dentro da pergunta, as mesmas setas viram um cursor de menu
    if(k === "ArrowUp")   moverSelecao(-1);
    if(k === "ArrowDown") moverSelecao(1);
    if(k === "Enter" || k === " ") confirmarSelecao();
  }else if(jogo && jogo.estado === "plot"){
    if(k === "Enter" || k === " ") document.getElementById("btPlot").click();
  }else{
    if(k === "ArrowUp")    virar(0,-1);
    if(k === "ArrowDown")  virar(0, 1);
    if(k === "ArrowLeft")  virar(-1,0);
    if(k === "ArrowRight") virar( 1,0);
  }

  if(k === " " || k === "Enter"){
    if(painelInicio.classList.contains("ativo")) document.getElementById("btComecar").click();
    else if(painelFim.classList.contains("ativo")) document.getElementById("btReiniciar").click();
  }
  if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight"," "].includes(k)) e.preventDefault();
});

document.getElementById("btCima").addEventListener("click", ()=>virar(0,-1));
document.getElementById("btBaixo").addEventListener("click",()=>virar(0, 1));
document.getElementById("btEsq").addEventListener("click",  ()=>virar(-1,0));
document.getElementById("btDir").addEventListener("click",  ()=>virar( 1,0));

/* ======================================================================
   4. MOVIMENTO
   ====================================================================== */
function andar(criatura, dt, escolherDirecao){
  if(criatura.t < 1){                       // ainda está entre duas células
    criatura.t += dt / criatura.duracao;
    if(criatura.t < 1) return false;
    criatura.t = 1;
  }
  // chegou ao centro de uma célula: hora de decidir para onde ir
  escolherDirecao(criatura);

  const d = criatura.dir;
  if(d.c === 0 && d.l === 0) return true;
  if(ehParede(criatura.col + d.c, criatura.lin + d.l)){
    criatura.dir = {c:0, l:0};
    return true;
  }
  criatura.antCol = criatura.col;
  criatura.antLin = criatura.lin;
  criatura.col += d.c;
  criatura.lin += d.l;
  criatura.t = 0;                           // começa a travessia para a nova célula
  return true;
}

function direcaoDoJogador(j){
  const p = j.proxDir;
  // se a direção pedida pelo jogador é livre, vira; senão segue em frente
  if((p.c || p.l) && !ehParede(j.col + p.c, j.lin + p.l)) j.dir = {c:p.c, l:p.l};
}

function direcaoDoIon(ion){
  const opcoes = [{c:1,l:0},{c:-1,l:0},{c:0,l:1},{c:0,l:-1}]
    .filter(d => !ehParede(ion.col + d.c, ion.lin + d.l))
    .filter(d => !(d.c === -ion.dir.c && d.l === -ion.dir.l));   // não dá meia-volta

  const livres = opcoes.length ? opcoes : [{c:-ion.dir.c, l:-ion.dir.l}];

  if(ion.medo || Math.random() < 0.72){
    // persegue: escolhe a direção que mais aproxima do jogador
    livres.sort((a,b)=>{
      const da = Math.abs(ion.col+a.c - jogador.col) + Math.abs(ion.lin+a.l - jogador.lin);
      const db = Math.abs(ion.col+b.c - jogador.col) + Math.abs(ion.lin+b.l - jogador.lin);
      return ion.medo ? db - da : da - db;   // assustado foge do jogador
    });
    ion.dir = livres[0];
  }else{
    ion.dir = livres[Math.floor(Math.random()*livres.length)];
  }
}

function atualizar(dt){
  if(jogo.estado !== "jogando") return;
  if(jogo.invulneravel > 0) jogo.invulneravel -= dt;
  jogo.piscar += dt;
  if(jogo.aviso > 0) jogo.aviso -= dt;
  if(jogo.medo > 0){ jogo.medo -= dt; if(jogo.medo <= 0) ions.forEach(i => i.medo = false); }

  const chegou = andar(jogador, dt, direcaoDoJogador);
  if(chegou) recolherItem();

  for(const ion of ions) andar(ion, dt * (ion.medo ? 0.6 : 1), direcaoDoIon);   // azuis andam mais devagar

  verificarEncontro();
}

function recolherItem(){
  const casa = grade[jogador.lin][jogador.col];
  if(casa === "atomo"){
    grade[jogador.lin][jogador.col] = "vazio";
    jogo.pontos += 10;
    jogo.atomos++;
    jogo.restantes--;
    atualizarHud();
    if(jogo.restantes <= 0){ proximaFase(); return; }
  }
  if(casa === "energia"){
    grade[jogador.lin][jogador.col] = "vazio";
    jogo.pontos += 50;
    jogo.combo = 0;
    jogo.medo = Math.max(2.5, 8 - jogo.fase * 0.7);   // fases altas: azul dura menos
    ions.forEach(i => i.medo = true);
  }
  if(casa === "capsula"){
    grade[jogador.lin][jogador.col] = "vazio";
    abrirPergunta();
  }
}

function verificarEncontro(){
  for(const ion of ions){
    const mesmaCasa = ion.col === jogador.col && ion.lin === jogador.lin;
    const trocaram  = ion.col === jogador.antCol && ion.lin === jogador.antLin &&
                      ion.antCol === jogador.col && ion.antLin === jogador.lin;
    if(!(mesmaCasa || trocaram)) continue;
    if(ion.medo){ comerIon(ion); return; }                       // azul: você come!
    if(jogo.invulneravel <= 0){ levouChoque(); return; }        // normal: você perde energia
  }
}

/* Comeu um íon azul: pontos dobram a cada íon seguido (200, 400, 800, 1600) e aparece o plot twist */
function comerIon(ion){
  jogo.combo++;
  const ganho = 200 * Math.pow(2, Math.min(jogo.combo, 4) - 1);
  jogo.pontos += ganho;
  ion.medo = false;                                              // volta ao ponto de partida, normal
  ion.col = ion.antCol = ion.ic; ion.lin = ion.antLin = ion.il;
  ion.t = 1; ion.dir = {c:0, l:0};
  atualizarHud();
  jogo.estado = "plot";
  document.getElementById("plotTitulo").textContent = "PLOT TWIST! +" + ganho;
  document.getElementById("plotTexto").textContent = proximaCuriosidade();
  document.getElementById("painelPlot").classList.add("ativo");
  document.getElementById("btPlot").focus();
}
document.getElementById("btPlot").addEventListener("click", ()=>{
  document.getElementById("painelPlot").classList.remove("ativo");
  jogo.estado = "jogando";
});

function levouChoque(){
  jogo.energia--;
  jogo.invulneravel = 1.6;
  atualizarHud();
  if(jogo.energia <= 0){ terminar(false); return; }
  // devolve todo mundo para o ponto de partida
  jogador.col = jogador.antCol = jogador.ic;
  jogador.lin = jogador.antLin = jogador.il;
  jogador.t = 1; jogador.dir = {c:0,l:0}; jogador.proxDir = {c:0,l:0};
  ions.forEach(ion=>{
    ion.col = ion.antCol = ion.ic; ion.lin = ion.antLin = ion.il;
    ion.t = 1; ion.dir = {c:0,l:0};
  });
}

/* ======================================================================
   5. DESENHO
   ====================================================================== */
function posicao(cr){
  // mistura a célula anterior com a atual para o movimento ficar suave
  const x = (cr.antCol + (cr.col - cr.antCol) * cr.t) * CELULA;
  const y = (cr.antLin + (cr.lin - cr.antLin) * cr.t) * CELULA;
  return {x, y};
}

function desenhar(){
  ctx.fillStyle = "#04060f";
  ctx.fillRect(0,0,tela.width,tela.height);

  // labirinto
  for(let l=0;l<LINS;l++){
    for(let c=0;c<COLS;c++){
      const x = c*CELULA, y = l*CELULA, casa = grade[l][c];
      if(casa === "parede"){
        ctx.fillStyle = "#1b2570"; ctx.fillRect(x, y, CELULA, CELULA);
        ctx.fillStyle = "#3a49ff"; ctx.fillRect(x+2, y+2, CELULA-4, CELULA-4);
        ctx.fillStyle = "#6f7dff"; ctx.fillRect(x+2, y+2, CELULA-4, 3);
      }
      if(casa === "atomo"){
        ctx.fillStyle = "#9fb6ff";
        ctx.fillRect(x + CELULA/2 - 2, y + CELULA/2 - 2, 4, 4);
      }
      if(casa === "energia"){
        ctx.fillStyle = "#ffffff"; ctx.beginPath();
        ctx.arc(x + CELULA/2, y + CELULA/2, Math.sin(jogo.piscar*8) > 0 ? 7 : 5, 0, 6.3); ctx.fill();
      }
      if(casa === "capsula"){
        const brilho = Math.sin(jogo.piscar*7) > 0 ? "#b6ff3d" : "#e8ffb0";
        ctx.fillStyle = brilho;
        ctx.fillRect(x+7, y+5, 10, 14);
        ctx.fillStyle = "#04060f";
        ctx.fillRect(x+9, y+8, 6, 4);
      }
    }
  }

  // íons instáveis (fantasmas)
  for(const ion of ions){
    const p = posicao(ion);
    ctx.fillStyle = ion.medo ? ((jogo.medo < 2 && Math.floor(jogo.piscar*6) % 2) ? "#ffffff" : "#3a49ff") : ion.cor;
    ctx.beginPath();
    ctx.arc(p.x + CELULA/2, p.y + CELULA/2 - 1, 8, Math.PI, 0);
    ctx.rect(p.x + CELULA/2 - 8, p.y + CELULA/2 - 1, 16, 9);
    ctx.fill();
    ctx.fillStyle = "#04060f";                       // olhos
    ctx.fillRect(p.x + 7,  p.y + 8, 3, 4);
    ctx.fillRect(p.x + 14, p.y + 8, 3, 4);
  }

  // jogador: uma partícula com "boca" que aponta para onde ele anda
  const pisca = jogo.invulneravel > 0 && Math.floor(jogo.invulneravel*10) % 2 === 0;
  if(!pisca){
    const p = posicao(jogador);
    const cx = p.x + CELULA/2, cy = p.y + CELULA/2;
    let giro = 0;
    if(jogador.dir.c === -1) giro = Math.PI;
    if(jogador.dir.l === -1) giro = -Math.PI/2;
    if(jogador.dir.l ===  1) giro =  Math.PI/2;
    const abertura = (Math.abs(Math.sin(jogo.piscar*12)) * 0.35) + 0.08;

    ctx.fillStyle = "#ffe066";
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, 9, giro + abertura, giro - abertura);
    ctx.closePath();
    ctx.fill();
  }
}

/* ======================================================================
   6. PERGUNTAS
   ====================================================================== */
const painelInicio   = document.getElementById("painelInicio");
const painelPergunta = document.getElementById("painelPergunta");
const painelFim      = document.getElementById("painelFim");
const textoPergunta  = document.getElementById("textoPergunta");
const listaAlt       = document.getElementById("listaAlternativas");
const caixaExplic    = document.getElementById("explicacao");
const btContinuar    = document.getElementById("btContinuar");

let selecaoAtual = 0;   // índice da alternativa marcada pelo cursor (setas ↑ ↓)

function abrirPergunta(){
  jogo.estado = "pergunta";
  jogo.atual = proximaPergunta(jogo.fase);
  const p = jogo.atual;
  selecaoAtual = 0;

  textoPergunta.textContent = "Cápsula " + (jogo.indice+1) + " — " + p.enunciado;
  listaAlt.innerHTML = "";
  caixaExplic.hidden = true;
  btContinuar.hidden = true;

  p.alternativas.forEach((texto,i)=>{
    const b = document.createElement("button");
    b.textContent = (i+1) + ") " + texto;
    // O clique continua funcionando para toque/mouse; a resposta é a mesma função do teclado.
    b.addEventListener("click", ()=>{ selecaoAtual = i; confirmarSelecao(); });
    listaAlt.appendChild(b);
  });

  destacarSelecao();
  painelPergunta.classList.add("ativo");
}

function destacarSelecao(){
  const botoes = listaAlt.querySelectorAll("button");
  botoes.forEach((b, i) => b.classList.toggle("selecionada", i === selecaoAtual));
}

function moverSelecao(direcao){
  const total = listaAlt.querySelectorAll("button").length;
  if(!total) return;
  selecaoAtual = (selecaoAtual + direcao + total) % total;
  destacarSelecao();
}

function confirmarSelecao(){
  if(jogo.estado !== "pergunta") return;
  const botoes = listaAlt.querySelectorAll("button");
  const botao = botoes[selecaoAtual];
  if(!botao || botao.disabled) return;
  responder(selecaoAtual, botao);
}

function responder(escolha, botao){
  const p = jogo.atual;
  const botoes = listaAlt.querySelectorAll("button");
  botoes.forEach(b => { b.disabled = true; b.classList.remove("selecionada"); });
  botoes[p.correta].classList.add("certa");

  if(escolha === p.correta){
    jogo.acertos++;
    jogo.pontos += 200 * jogo.fase;
    caixaExplic.textContent = "Acertou! " + p.explicacao;
  }else{
    botao.classList.add("errada");
    jogo.energia--;                       // errar custa uma unidade de energia
    jogo.errosResposta++;
    caixaExplic.textContent = "Errou e perdeu energia. " + p.explicacao;
  }
  caixaExplic.hidden = false;
  btContinuar.hidden = false;
  btContinuar.focus();
  atualizarHud();
}

btContinuar.addEventListener("click", ()=>{
  painelPergunta.classList.remove("ativo");
  jogo.indice++;
  if(jogo.energia <= 0){ terminar(false); return; }
  jogo.estado = "jogando";
  jogo.invulneravel = 1.0;                // um respiro ao voltar para o labirinto
});

/* ======================================================================
   7. FIM DE JOGO E ENVIO DO PLACAR
   ====================================================================== */
function terminar(venceu){
  jogo.estado = "fim";
  const pontos = Math.max(0, jogo.pontos);
  const duracao_s = Math.round((Date.now() - jogo.inicio) / 1000);

  document.getElementById("tituloFim").textContent = "Energia esgotada — Fase " + jogo.fase;
  document.getElementById("resumoFinal").textContent =
    "Pontuação: " + pontos + " · Acertos: " + jogo.acertos + " de " + (jogo.acertos + jogo.errosResposta) +
    " · Átomos recolhidos: " + jogo.atomos;

  painelFim.classList.add("ativo");

  // Formato oficial do contrato Recreio Arcade (ScoreMessage): o jogo não sabe
  // o apelido do jogador nem o guarda — quem cuida disso é a plataforma local (G3).
  const placar = {
    tipo:"PLACAR",
    jogo:"elemento-relampago",
    versao:"1.0.0",
    pontos:pontos,
    duracao_s:duracao_s,
    acertos:jogo.acertos,
    erros:jogo.errosResposta,
    tema:"Química"
  };
  try{ window.parent.postMessage(placar, "*"); }catch(e){ console.log("Placar:", placar); }
}

/* ======================================================================
   8. PLACAR, BOTÕES E LAÇO PRINCIPAL
   ====================================================================== */
function atualizarHud(){
  document.getElementById("hudPontos").textContent     = jogo.pontos;
  document.getElementById("hudEnergia").textContent    = "⚡".repeat(Math.max(0, jogo.energia)) || "—";
  document.getElementById("hudProgresso").textContent  = jogo.fase;
  document.getElementById("hudPelotas").textContent    = jogo.atomos;
}

document.getElementById("btComecar").addEventListener("click", ()=>{
  painelInicio.classList.remove("ativo");
  novoJogo();
});
document.getElementById("btReiniciar").addEventListener("click", ()=>{
  painelFim.classList.remove("ativo");
  novoJogo();
});

function laco(tempo){
  if(!ultimoTempo) ultimoTempo = tempo;
  const dt = Math.min(0.05, (tempo - ultimoTempo)/1000);
  ultimoTempo = tempo;
  atualizar(dt);
  desenhar();
  if(jogo.aviso > 0){   // faixa "FASE N" no início de cada fase
    ctx.fillStyle = "#b6ff3d"; ctx.font = "bold 30px Courier New"; ctx.textAlign = "center";
    ctx.fillText("FASE " + jogo.fase, tela.width/2, tela.height/2 + 10); ctx.textAlign = "left";
  }
  requestAnimationFrame(laco);
}

novoJogo();
jogo.estado = "pergunta";     // congela o labirinto enquanto a tela inicial está aberta
requestAnimationFrame(laco);
</script>
</body>
</html>
