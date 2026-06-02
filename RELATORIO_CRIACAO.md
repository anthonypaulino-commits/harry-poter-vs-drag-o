# 📚 Relatório de Criação: Batalha Mágica vs Dragão 🐉

## 📋 Visão Geral do Projeto

Este é um jogo 3D em primeira pessoa desenvolvido com **Three.js**, onde o jogador controla um mago em batalha contra um dragão ancestral em um ambiente de ruínas. O jogo foi implementado em HTML5 com JavaScript vanilla e gráficos 3D renderizados em tempo real.

---

## 🎮 Mecânicas Principais do Jogo

### Sistema de Combate
- **Magia do Mago**: Projéteis de feitiço (azul ciano) disparados por clique esquerdo
- **Ataque do Dragão**: Bolas de fogo que perseguem o jogador
- **Sistema de HP**: Ambos os personagens possuem barras de vida no HUD
  - Mago: 100 HP (verde)
  - Dragão: 300 HP (vermelho)

### Controles
```
W, A, S, D     → Movimento
Mouse         → Olhar ao redor
Espaço        → Pular
Clique Esquerdo → Disparar feitiço
ESC           → Pausar (unlock pointer)
```

### IA do Dragão

O dragão utiliza máquina de estados com 3 modos:

1. **Chase (Perseguição)**
   - Persegue o jogador quando distante (> 20 unidades)
   - Velocidade: 15 unidades por frame
   - Muda para "attack" quando próximo

2. **Attack (Ataque)**
   - Atira bolas de fogo a cada 1.5 segundos
   - 30% de chance de mudar de posição
   - Mantém altura de 15 unidades

3. **Reposition (Reposicionamento)**
   - Muda para altura de 25 unidades
   - Voa para posição aleatória ao redor do jogador (velocidade: 20)
   - Duração: 2 segundos

---

## 🌍 Ambiente do Jogo

### Componentes Cênicos

| Elemento | Descrição |
|----------|-----------|
| **Chão** | Plano 200x200 com textura de pedra (cor: #2e2e2e) |
| **Pilares** | 15 pilares de ruína (3x15x3) distribuídos aleatoriamente |
| **Paredes** | 4 muros de contenção (borders da arena) |
| **Iluminação** | Luz ambiente (azul), luz direcional (lua), luz pontual (verde central) |
| **Neblina** | FogExp2 para efeito de profundidade sinistra |

### Sistema de Iluminação
```javascript
- Ambient Light: #404060 (0.6 intensidade) - tom azulado
- Directional Light: #aaaaaa (0.5 intensidade) - efeito lunar
- Point Light: #00ff00 (0.5 intensidade, 50 raio) - luz verde central
```

---

## 👾 Entidades 3D

### Mago (Jogador)

**Varinha Mágica:**
- Handle: Cilindro marrom (#5c4033) - 0.6 unidades de comprimento
- Tip: Esfera azul ciano (#00ffff) com brilho
- Posição: Câmera + offset (0.3, -0.2, -0.5)
- Efeito de recoil ao disparar

**Projéteis:**
- Geometria: Esfera (0.2 raio)
- Velocidade: 60 unidades/s
- Vida útil: 2 segundos

### Dragão

**Estrutura Hierárquica:**

```
DragonGroup
├── Body (BoxGeometry 4x3x6)
├── DragonHead (Group)
│   ├── HeadBox (BoxGeometry 2x2x3)
│   ├── Eye Left (BoxGeometry 0.4x0.4x0.1)
│   ├── Eye Right (BoxGeometry 0.4x0.4x0.1)
│   └── Snout (BoxGeometry 1.8x1x2)
├── Tail (ConeGeometry)
├── LeftWing (Group com rotação dinâmica)
└── RightWing (Group com rotação dinâmica)
```

**Cores:**
- Corpo e cabeça: Vermelho escuro (#8b0000)
- Escamas: Preto (#222222)
- Olhos: Amarelo (#ffff00)

**Animações:**
- Asas batem continuamente (sine wave)
- Corpo flutua (oscilação vertical)
- Cabeça segue o jogador independentemente

**Ataque (Bolas de Fogo):**
- Geometria: Esfera (0.8 raio)
- Cor: Laranja fogo (#ff4500)
- Velocidade: 30 unidades/s
- Vida útil: 3 segundos
- Efeito visual: Pulsação de escala

---

## 🖥️ Tecnologias Utilizadas

### Bibliotecas
- **Three.js r128**: Renderização 3D WebGL
- **PointerLockControls**: Controle de câmera em primeira pessoa
- **CDN jsDelivr/cdnjs**: Distribuição de bibliotecas

### Recursos WebGL
```javascript
- Geometry: PlaneGeometry, BoxGeometry, SphereGeometry, CylinderGeometry, ConeGeometry
- Material: MeshStandardMaterial, MeshLambertMaterial, MeshBasicMaterial
- Lighting: AmbientLight, DirectionalLight, PointLight
- Effects: Fog, Shadow mapping (PCFSoftShadowMap)
```

---

## 🎨 Interface de Usuário (UI)

### Telas
1. **Menu Inicial**
   - Título em laranja com efeito de brilho
   - Instruções de controle
   - Botão "Clique para Jogar"

2. **HUD In-Game**
   - Barra de vida do dragão (topo)
   - Retícula de mira (centro)
   - Barra de vida do mago (inferior esquerdo)

3. **Game Over**
   - Mensagem de vitória/derrota
   - Botão "Jogar Novamente"

### Estilos CSS
- Cores temáticas: Azul (#2196F3), Verde (#4CAF50), Vermelho (#F44336), Laranja (#ff9800)
- Transições suaves em barras de vida
- Efeitos de hover em botões

---

## 🔄 Fluxo de Jogo

```
MENU INICIAL
    ↓
[Clique para Jogar]
    ↓
JOGO ATIVO
├─ Câmera travada (PointerLock)
├─ Jogador controla posição/visão
├─ Dragão executa IA (chase/attack/reposition)
├─ Sistema de detecção de colisão
└─ Atualização de HP e UI
    ↓
GAME OVER
├─ Vitória: "DRAGÃO DERROTADO!" (verde)
└─ Derrota: "VOCÊ FOI INCINERADO!" (vermelho)
    ↓
[Jogar Novamente] → MENU INICIAL
```

---

## 🐛 Sistema de Detecção de Colisão

### Hitbox Simples
- **Feitiço**: Distância < 5 unidades do dragão
- **Bola de Fogo**: Distância < 2 unidades da câmera
- **Chão**: Y < 0
- **Paredes da Arena**: Limites X/Z ±98

---

## 📊 Variáveis de Estado Importantes

```javascript
// Jogo
let gameActive          // Jogo rodando?
let playerHP = 100      // Vida do mago
let dragonHP = 300      // Vida do dragão

// Movimento
velocity                // Vetor de velocidade 3D
canJump                 // Pode pular?

// Dragão
dragonState = {
  mode: 'chase',        // Estado atual: 'chase', 'attack', 'reposition', 'dead'
  timer: 0,             // Contador para transição de estado
  hoverY: 0,            // Altura de flutuação
  time: 0,              // Tempo total decorrido
  targetPos: Vector3    // Posição alvo (reposition)
}

// Entidades
spells: [],             // Array de feitiços ativos
fireballs: []           // Array de bolas de fogo ativas
```

---

## ⚡ Performance e Otimizações

- **Shadow Mapping**: PCFSoftShadowMap para sombras suaves
- **Fog**: Reduz renderização de objetos distantes
- **Cleanup**: Remoção de projéteis após colisão ou expiração
- **Delta Time**: Frame rate independente via `deltaTime`

---

## 🎯 Próximas Melhorias Possíveis

1. Múltiplos tipos de feitiços com efeitos diferentes
2. Consumíveis (poções de vida, mana)
3. Diferentes arenas/fases
4. Sistema de pontuação
5. Efeitos sonoros e música
6. Modelos 3D importados
7. Partículas avançadas
8. Multiplayer online
9. Gerenciamento de recursos (mana system)
10. Boss secundários/minions

---

## 📝 Conclusão

"Batalha Mágica vs Dragão" é uma demonstração funcional de um jogo 3D em primeira pessoa que combina:
- Renderização 3D com Three.js
- Física básica (gravidade, velocidade)
- IA comportamental (máquina de estados)
- Detecção de colisão
- Interface responsiva
- Sistema de combate interativo

O código é modular e extensível, permitindo fácil adição de novos recursos e mecânicas.

---

**Data de Criação**: Junho 2026  
**Tecnologia Principal**: Three.js r128 + HTML5 + JavaScript  
**Tipo de Jogo**: FPS / Ação 3D  
**Status**: Funcional e Completo ✅