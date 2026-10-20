# JaquesKB (Jkb) — Firmware ZMK

Repositório de configuração e documentação do firmware **ZMK (Zephyr Mechanical Keyboard)** para o **JaquesKB (`Jkb`)**, um teclado mecânico ergonômico *split* (dividido em duas metades) sem fio, projetado sob medida com layout adaptado para **ABNT2 (Português - Brasil)**.

---

## 1. Visão Geral do Projeto

O **JaquesKB (`Jkb`)** é um teclado mecânico dividido de **60 teclas** (30 teclas no lado esquerdo + 30 teclas no lado direito), construído a partir da estrutura base de um shield *Corne*, porém expandido para uma matriz de **5 linhas por 13 colunas**.

### Especificações Principais
- **Firmware:** [ZMK Firmware](https://zmk.dev/) (baseado no RTOS Zephyr)
- **Controladoras:** 2x `nice!nano` (nRF52840) — comunicação sem fio via Bluetooth (BLE)
- **Topologia Split:**
  - **Lado Esquerdo (`jkb_left`):** Central (Mestre) — conecta-se ao computador via USB ou Bluetooth e recebe os sinais do lado direito.
  - **Lado Direito (`jkb_right`):** Periférico (Escravo) — envia os pressionamentos de tecla via BLE para o lado esquerdo.
- **Total de Teclas:** 60 teclas (30 esquerda + 30 direita)
- **Direção dos Diodos:** `col2row` (Coluna para Linha)
- **Gerenciamento de Energia:** Deep Sleep habilitado e potência de transmissão Bluetooth aumentada para `+8 dBm` (`CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`) para maior estabilidade entre as metades.

---

## 2. Estrutura do Repositório

```text
zmk-config-JaquesKB/
├── .github/workflows/
│   └── build.yml                          # Pipeline do GitHub Actions para compilar o firmware (.uf2)
├── config/
│   ├── boards/shields/corne/              # Definições de hardware do shield customizado (Jkb)
│   │   ├── Kconfig.defconfig              # Define nome padrão "Jkb" e papel Central no lado esquerdo
│   │   ├── Kconfig.shield                 # Registra os shields jkb_left e jkb_right
│   │   ├── jkb.conf                       # Configurações globais (Sleep, Timeout, TX Power, HID)
│   │   ├── jkb.dtsi                       # Matriz lógica (5x13) e transformação de coordenadas RC(row,col)
│   │   ├── jkb_left.conf                  # Configurações específicas da metade esquerda
│   │   ├── jkb_left.overlay               # Pinagem (GPIO) das 6 colunas e 5 linhas da esquerda
│   │   ├── jkb_right.conf                 # Configurações específicas da metade direita
│   │   └── jkb_right.overlay              # Pinagem (GPIO) das 7 colunas e 5 linhas da direita + offset
│   ├── jkb.keymap                         # Mapa de teclas (Layers 0, 1 e 2) adaptado para ABNT2
│   └── west.yml                           # Manifesto do West apontando para o repositório oficial do ZMK
├── zephyr/
│   └── module.yml                         # Declaração de módulo Zephyr para encontrar os shields locais
└── build.yaml                             # Matriz de compilação (jkb_left, jkb_right e settings_reset)
```

---

## 3. Funcionamento do Hardware e Matriz de Pinos

A leitura das teclas utiliza o driver `zmk,kscan-gpio-matrix` com diodos no sentido `col2row` (as colunas são acionadas em nível alto `GPIO_ACTIVE_HIGH` e as linhas possuem *pull-down* interno `GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN`).

### 3.1. Pinagem do Lado Esquerdo (`jkb_left.overlay`)
O lado esquerdo possui **6 colunas x 5 linhas = 30 teclas**, mapeadas nas colunas lógicas `0` a `5`:

| Função na Matriz | Índice Lógico | Pino `pro_micro` | Configuração GPIO |
| :--- | :---: | :---: | :--- |
| **Colunas (0 a 5)** | `Col 0` → `Col 5` | `7`, `6`, `5`, `4`, `3`, `2` | `GPIO_ACTIVE_HIGH` |
| **Linhas (0 a 4)** | `Row 0` → `Row 4` | `20`, `19`, `18`, `15`, `14` | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |

### 3.2. Pinagem do Lado Direito (`jkb_right.overlay`)
O lado direito aplica um deslocamento de 6 colunas (`col-offset = <6>`) e utiliza **7 colunas x 5 linhas**, ocupando as colunas lógicas `6` a `12` para totalizar **30 teclas**:
- As linhas `0` e `1` usam 6 teclas cada (`Col 7` a `Col 12`).
- As linhas `2` e `3` usam 7 teclas cada (`Col 6` a `Col 12`, adicionando 2 teclas internas extras próximas ao centro).
- A linha `4` (inferior/polegar) usa 4 teclas (`Col 7` a `Col 10`).

| Função na Matriz | Índice Lógico | Pino `pro_micro` | Configuração GPIO |
| :--- | :---: | :---: | :--- |
| **Colunas (6 a 12)** | `Col 6` → `Col 12` | `9`, `8`, `7`, `6`, `21`, `20`, `19` | `GPIO_ACTIVE_HIGH` |
| **Linhas (0 a 4)** | `Row 0` → `Row 4` | `18`, `15`, `14`, `16`, `10` | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |

### 3.3. Mapa Físico de Coordenadas (`jkb.dtsi`)

```text
LADO ESQUERDO (30 teclas)                                       LADO DIREITO (30 teclas)
┌───────┬───────┬───────┬───────┬───────┬───────┐               ┌───────┬───────┬───────┬───────┬───────┬───────┐
│RC(0,0)│RC(0,1)│RC(0,2)│RC(0,3)│RC(0,4)│RC(0,5)│               │RC(0,7)│RC(0,8)│RC(0,9)│RC(0,10│RC(0,11│RC(0,12│
├───────┼───────┼───────┼───────┼───────┼───────┤               ├───────┼───────┼───────┼───────┼───────┼───────┤
│RC(1,0)│RC(1,1)│RC(1,2)│RC(1,3)│RC(1,4)│RC(1,5)│               │RC(1,7)│RC(1,8)│RC(1,9)│RC(1,10│RC(1,11│RC(1,12│
├───────┼───────┼───────┼───────┼───────┼───────┤       ┌───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│RC(2,0)│RC(2,1)│RC(2,2)│RC(2,3)│RC(2,4)│RC(2,5)│       │RC(2,6)│RC(2,7)│RC(2,8)│RC(2,9)│RC(2,10│RC(2,11│RC(2,12│
├───────┼───────┼───────┼───────┼───────┼───────┤       ├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│RC(3,0)│RC(3,1)│RC(3,2)│RC(3,3)│RC(3,4)│RC(3,5)│       │RC(3,6)│RC(3,7)│RC(3,8)│RC(3,9)│RC(3,10│RC(3,11│RC(3,12│
├───────┼───────┼───────┼───────┼───────┼───────┤       └───────┼───────┼───────┼───────┼───────┼───────┴───────┘
│RC(4,0)│RC(4,1)│RC(4,2)│RC(4,3)│RC(4,4)│RC(4,5)│               │RC(4,7)│RC(4,8)│RC(4,9)│RC(4,10│
└───────┴───────┴───────┴───────┴───────┴───────┘               └───────┴───────┴───────┴───────┘
```

---

## 4. Funcionamento do Keymap (`config/jkb.keymap`)

O keymap utiliza os códigos padrão do ZMK combinados com o layout **ABNT2** configurado no sistema operacional. Abaixo está a tabela de equivalência dos códigos especiais para ABNT2 usados na camada base:

| Código no ZMK | Caractere no S.O. (ABNT2) | Posição no Teclado |
| :--- | :---: | :--- |
| `&kp SEMI` | **`Ç`** | Ao lado da tecla `L` |
| `&kp LBKT` | **`´` / `` ` ``** (Acento agudo / Crase) | Extrema direita da linha do `A S D F` |
| `&kp SQT` | **`~` / `^`** (Til / Circunflexo) | Extrema direita da linha do `Z X C V` |
| `&kp BSLH` | **`]` / `}`** | Tecla interna extra superior direita `RC(2,6)` |
| `&kp RBKT` | **`[` / `{`** | Última tecla inferior esquerda `RC(4,5)` |
| `&kp NUBS` | **`\` / `\|`** | Ao lado da tecla `V` na esquerda `RC(3,5)` |
| `&kp FSLH` | **`;` / `:`** | Ao lado da tecla `M` |
| `&kp INT1` | **`/` / `?`** | Ao lado de `; :` na linha inferior alfabética |

---

### Layer 0: Base (`default_layer`)
Camada principal de digitação. Possui acesso momentâneo para a **Layer 1** (`&mo 1`, no polegar esquerdo) e para a **Layer 2** (`&mo 2`, na tecla interna inferior do lado direito).

```text
LADO ESQUERDO                                                   LADO DIREITO
┌───────┬───────┬───────┬───────┬───────┬───────┐               ┌───────┬───────┬───────┬───────┬───────┬───────┐
│  ESC  │   1   │   2   │   3   │   4   │   5   │               │   6   │   7   │   8   │   9   │   0   │ BSPC  │
├───────┼───────┼───────┼───────┼───────┼───────┤               ├───────┼───────┼───────┼───────┼───────┼───────┤
│  TAB  │   Q   │   W   │   E   │   R   │   T   │               │   Y   │   U   │   I   │   O   │   P   │  DEL  │
├───────┼───────┼───────┼───────┼───────┼───────┤       ┌───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│LSHIFT │   A   │   S   │   D   │   F   │   G   │       │  ] }  │   H   │   J   │   K   │   L   │   Ç   │  ´ `  │
├───────┼───────┼───────┼───────┼───────┼───────┤       ├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│ LCTRL │   Z   │   X   │   C   │   V   │  \ |  │       │ MO(2) │   B   │   N   │   M   │  ; :  │  / ?  │  ~ ^  │
├───────┼───────┼───────┼───────┼───────┼───────┤       └───────┼───────┼───────┼───────┼───────┼───────┴───────┘
│ RWIN  │ LALT  │ CAPS  │ SPACE │ MO(1) │  [ {  │               │ ENTER │ RALT  │   ,   │   .   │
└───────┴───────┴───────┴───────┴───────┴───────┘               └───────┴───────┴───────┴───────┘
```

---

### Layer 1: Navegação e Numpad (`lower_layer`)
Ativada enquanto segura **`MO(1)`**. Transforma o `W A S D` em setas direcionais na mão esquerda e ativa um **Teclado Numérico (Numpad)** completo na mão direita.

```text
LADO ESQUERDO                                                   LADO DIREITO
┌───────┬───────┬───────┬───────┬───────┬───────┐               ┌───────┬───────┬───────┬───────┬───────┬───────┐
│       │       │       │       │       │       │               │       │       │  KP / │  KP * │       │       │
├───────┼───────┼───────┼───────┼───────┼───────┤               ├───────┼───────┼───────┼───────┼───────┼───────┤
│       │       │  UP   │       │       │       │               │       │ KP 7  │ KP 8  │ KP 9  │  KP - │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       ┌───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│       │ LEFT  │ DOWN  │ RIGHT │       │       │       │       │       │ KP 4  │ KP 5  │ KP 6  │  KP + │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       ├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│       │       │       │       │       │       │       │ MO(2) │       │ KP 1  │ KP 2  │ KP 3  │ KP .  │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       └───────┼───────┼───────┼───────┼───────┼───────┴───────┘
│       │       │       │       │ MO(1) │       │               │       │ KP 0  │ KP ,  │KP ENT │
└───────┴───────┴───────┴───────┴───────┴───────┘               └───────┴───────┴───────┴───────┘
(Células em branco = &trans, herdam o comportamento da Layer 0)
```

---

### Layer 2: Bluetooth e Mídia (`raise_layer`)
Ativada enquanto segura **`MO(2)`**. No estado atual, a estrutura da camada já está declarada no código, mas todas as teclas estão como `&trans` (aguardando mapeamento dos controles de Bluetooth e mídia).

---

## 5.Ponto Atual do Projeto (Status)

### O que já está pronto e funcional:
- [x] Definição completa da matriz de 60 teclas (`5x13`) e mapeamento de GPIOs de ambas as metades (`jkb_left` e `jkb_right`).
- [x] Pipeline de CI/CD no GitHub Actions gerando automaticamente os 3 arquivos `.uf2`: `jkb_left`, `jkb_right` e `settings_reset`.
- [x] Renomeação dos artefatos e do dispositivo Bluetooth de `Corne` para `Jkb`.
- [x] Camada Base (Layer 0) ajustada e funcional para digitação no padrão brasileiro **ABNT2** (incluindo `Ç`, acentos, `/ ?`, `\ |`, colchetes e chaves).
- [x] Camada Secundária (Layer 1) funcional com setas no `WASD` e bloco numérico (Numpad) completo no lado direito.
- [x] Potência do rádio Bluetooth amplificada (`+8 dBm`) e suporte a modo de economia de energia (`CONFIG_ZMK_SLEEP=y`).

### O que falta finalizar / Próximos passos planejados:
- [ ] Popular a **Layer 2 (`raise_layer`)** com comandos de gerenciamento de Bluetooth (`BT_CLR`, troca de perfis `BT_SEL`), teclas de função (`F1`–`F12`) e controles de mídia/volume.

---

## 6. Como Compilar e Gravar o Firmware

1. **Gerar o Firmware:**
   - A cada *push* no GitHub (ou executando manualmente na aba **Actions** → *Build ZMK firmware*), o GitHub compila e disponibiliza um arquivo `firmware.zip` nos *Artifacts* da execução.
2. **Gravar nas Controladoras (`nice!nano`):**
   - Conecte uma das metades ao PC via cabo USB e pressione duas vezes rapidamente o botão **RST** (Reset).
   - Uma unidade USB chamada `NICENANO` aparecerá no computador.
   - Copie o arquivo `jkb_left-nice_nano-zmk.uf2` para a controladora esquerda e `jkb_right-nice_nano-zmk.uf2` para a direita.
3. **Caso as metades percam a sincronia (Pareamento Split):**
   - Grave primeiro o arquivo `settings_reset-nice_nano-zmk.uf2` em **ambas** as controladoras para limpar a memória flash Bluetooth.
   - Em seguida, grave novamente o `jkb_left` e o `jkb_right` e ligue as duas metades próximas uma da outra.

---

## 7. Avaliação Técnica do Código e Lista de Melhorias Sugeridas

> **Nota:** Nenhum arquivo de código foi alterado. Abaixo estão os pontos levantados na revisão técnica do código atual para que você possa avaliar se e quando deseja aplicá-los.

### 1. Espaço após a vírgula em `Kconfig.shield` (Importante)
- **Onde:** `config/boards/shields/corne/Kconfig.shield`
- **Como está hoje:**
  ```kconfig
  config SHIELD_JKB_LEFT
      def_bool $(shields_list_contains, jkb_left)

  config SHIELD_JKB_RIGHT
      def_bool $(shields_list_contains, jkb_right)
  ```
- **O que acontece:** No pre-processador Kconfig do Zephyr, o espaço após a vírgula faz com que a macro procure pelo texto `" jkb_left"` (com espaço no início) em vez de `"jkb_left"`. Isso pode fazer com que `SHIELD_JKB_LEFT` e `SHIELD_JKB_RIGHT` retornem falso durante o build, fazendo o `Kconfig.defconfig` ser ignorado silenciosamente.
- **Sugestão:** Remover o espaço após a vírgula: `$(shields_list_contains,jkb_left)` e `$(shields_list_contains,jkb_right)`.

### 2. Configurações de `jkb.conf` vs `jkb_left.conf` / `jkb_right.conf`
- **Onde:** `config/boards/shields/corne/jkb.conf`, `jkb_left.conf` e `jkb_right.conf`
- **Como está hoje:** Os tempos de inatividade (`CONFIG_ZMK_IDLE_TIMEOUT=30000` e `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000`) estão declarados apenas em `jkb.conf`, mas não estão em `jkb_left.conf` nem em `jkb_right.conf`.
- **O que acontece:** No ZMK, quando você compila o shield `jkb_left`, o sistema busca automaticamente por `<nome_do_shield>.conf` (ou seja, `jkb_left.conf`). O arquivo `jkb.conf` só é carregado automaticamente se oKconfig ou o nome base casar, ou se você centralizar tudo em `config/jkb.conf` (na mesma pasta do `jkb.keymap`).
- **Sugestão:** Mover o `jkb.conf` para `config/jkb.conf` (junto ao `jkb.keymap`) ou copiar os parâmetros de `IDLE_TIMEOUT` e `IDLE_SLEEP_TIMEOUT` para `jkb_left.conf` e `jkb_right.conf`.

### 3. Preenchimento da Layer 2 (`raise_layer`) em `config/jkb.keymap`
- **Onde:** `config/jkb.keymap` (linhas 38 a 47)
- **Como está hoje:** O header `#include <dt-bindings/zmk/bt.h>` já está incluído no topo do arquivo, mas a `raise_layer` inteira está preenchida apenas com `&trans`.
- **Sugestão:** Adicionar na `raise_layer`:
  - **Bluetooth:** `&bt BT_CLR` (limpar perfil atual) e `&bt BT_SEL 0`, `&bt BT_SEL 1`, `&bt BT_SEL 2` (alternar entre até 3 computadores/dispositivos pareados).
  - **Teclas de Função:** `&kp F1` até `&kp F12` (por exemplo, na primeira linha onde ficam os números na Layer 0).
  - **Mídia:** `&kp C_VOL_DN`, `&kp C_VOL_UP`, `&kp C_MUTE`, `&kp C_PP` (Play/Pause).
  - **Bootloader via Teclado (Opcional, mas muito útil):** `&bootloader` em ambos os lados para entrar em modo de gravação `.uf2` sem precisar apertar duas vezes o botão físico da placa.

### 4. Versão da placa no `build.yaml` (`nice_nano` vs `nice_nano_v2`)
- **Onde:** `build.yaml`
- **Como está hoje:** `board: nice_nano`
- **O que avaliar:** No ZMK, `nice_nano` refere-se à versão 1.0 original da placa, enquanto a maioria das placas comercializadas atualmente (incluindo clones nRF52840 Pro Micro) usa a pinagem de controle de bateria/energia da `nice_nano_v2`. Se já está funcionando perfeitamente para o seu hardware atual, não precisa mudar; mas caso note algum comportamento estranho na medição de bateria ou alimentação de periféricos futuros, vale testar `nice_nano_v2`.

### 5. Organização do nome da pasta do Shield (Estético)
- **Onde:** `config/boards/shields/corne/`
- **Como está hoje:** Todos os arquivos já foram renomeados para `jkb*`, mas a pasta pai ainda se chama `corne`.
- **Sugestão:** Como o Zephyr lê todas as subpastas de `shields/` independentemente do nome do diretório, renomear a pasta `config/boards/shields/corne/` para `config/boards/shields/jkb/` não afeta o funcionamento do código e deixa a organização do repositório 100% consistente com o nome do projeto.

### 6. Posicionamento da tecla `B` e do `&mo 2` (Ergonomia / Layout)
- **Onde:** `config/jkb.keymap` (linha 21)
- **Como está hoje:** Na linha do `Z X C V`, a 6ª tecla da mão esquerda (`RC(3,5)`) está mapeada como `&kp NUBS` (`\ |`), enquanto a letra `B` foi deslocada para a mão direita (`RC(3,7)`), logo depois da tecla `&mo 2` (`RC(3,6)`). Além disso, o comentário da Layer 2 diz `/* LAYER 2: BLUETOOTH E MEDIA (Polegar Esquerda) */`, porém o atalho `&mo 2` está fisicamente no lado direito (`RC(3,6)`).
- **Sugestão:** Se você já se acostumou com o `B` na mão direita, ótimo! Caso contrário, pode avaliar trocar a posição do `B` de volta para a mão esquerda ou ajustar os comentários das layers para refletirem a posição real dos botões.
