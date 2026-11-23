# JaquesKB (Jkb) — Firmware ZMK

Repositório de configuração e documentação do firmware **ZMK (Zephyr Mechanical Keyboard)** para o **JaquesKB (`Jkb`)**, um teclado mecânico ergonômico *split* (dividido em duas metades) sem fio, projetado sob medida com layout adaptado e otimizado ergonomicamente para **ABNT2 (Português - Brasil)**.

---

## Sumário
1. [Visão Geral do Projeto](#1-visão-geral-do-projeto)
2. [Estrutura do Repositório e Links dos Arquivos](#2-estrutura-do-repositório-e-links-dos-arquivos)
3. [Funcionamento do Hardware e Matriz de Pinos](#3-funcionamento-do-hardware-e-matriz-de-pinos)
4. [Funcionamento do Keymap (Layers 0, 1 e 2)](#4-funcionamento-do-keymap-layers-0-1-e-2)
5. [Ponto Atual do Projeto (Status)](#5-ponto-atual-do-projeto-status)
6. [Como Compilar e Gravar o Firmware](#6-como-compilar-e-gravar-o-firmware)

---

## 1. Visão Geral do Projeto

O **JaquesKB (`Jkb`)** é um teclado mecânico dividido de **60 teclas** (30 teclas no lado esquerdo + 30 teclas no lado direito), construído com uma matriz lógica de **5 linhas por 13 colunas** ([`jkb.dtsi`](config/boards/shields/jkb/jkb.dtsi)).

### Especificações Principais
- **Firmware:** [ZMK Firmware](https://zmk.dev/) (baseado no RTOS Zephyr 4.1 / HWMv2, configurado em [`config/west.yml`](config/west.yml))
- **Controladoras:** 2x `nice_nano//zmk` (nice!nano v2 / nRF52840) — comunicação sem fio via Bluetooth (BLE), definidas em [`build.yaml`](build.yaml)
- **Topologia Split ([`Kconfig.defconfig`](config/boards/shields/jkb/Kconfig.defconfig)):**
  - **Lado Esquerdo (`jkb_left`):** Central (Mestre) — conecta-se ao computador via USB ou Bluetooth e recebe os sinais do lado direito.
  - **Lado Direito (`jkb_right`):** Periférico (Escravo) — envia os pressionamentos de tecla via BLE para o lado esquerdo.
- **Total de Teclas:** 60 teclas (30 esquerda + 30 direita)
- **Direção dos Diodos:** `col2row` (Coluna para Linha)
- **Gerenciamento de Energia ([`jkb_left.conf`](config/boards/shields/jkb/jkb_left.conf) / [`jkb_right.conf`](config/boards/shields/jkb/jkb_right.conf)):**
  - Tempo de inatividade (*Idle*): 30 segundos (`CONFIG_ZMK_IDLE_TIMEOUT=30000`)
  - Modo *Deep Sleep*: 15 minutos (`CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000`)
  - Potência de transmissão Bluetooth aumentada para `+8 dBm` (`CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`) para máxima estabilidade entre as metades.

---

## 2. Estrutura do Repositório e Links dos Arquivos

Clique em qualquer arquivo na tabela abaixo para abrir diretamente o código correspondente:

| Arquivo | Caminho no Repositório | Função no Projeto |
| :--- | :--- | :--- |
| **`build.yml`** | [`.github/workflows/build.yml`](.github/workflows/build.yml) | Pipeline do GitHub Actions para compilar automaticamente os arquivos `.uf2`. |
| **`build.yaml`** | [`build.yaml`](build.yaml) | Matriz de compilação (`nice_nano//zmk` + `jkb_left`, `jkb_right` e `settings_reset`). |
| **`module.yml`** | [`zephyr/module.yml`](zephyr/module.yml) | Declaração de módulo do Zephyr para localizar os shields customizados na raiz. |
| **`west.yml`** | [`config/west.yml`](config/west.yml) | Manifesto do West que importa a versão principal do ZMK Firmware. |
| **`jkb.keymap`** | [`config/jkb.keymap`](config/jkb.keymap) | Mapa de teclas completo (Layers 0, 1 e 2) adaptado e otimizado para ABNT2. |
| **`Kconfig.shield`** | [`config/boards/shields/jkb/Kconfig.shield`](config/boards/shields/jkb/Kconfig.shield) | Registra os identificadores dos shields `SHIELD_JKB_LEFT` e `SHIELD_JKB_RIGHT`. |
| **`Kconfig.defconfig`** | [`config/boards/shields/jkb/Kconfig.defconfig`](config/boards/shields/jkb/Kconfig.defconfig) | Define o nome `"Jkb"`, ativa `ZMK_SPLIT` e define o lado esquerdo como Central. |
| **`jkb.dtsi`** | [`config/boards/shields/jkb/jkb.dtsi`](config/boards/shields/jkb/jkb.dtsi) | Define a matriz lógica (`5x13`), direção dos diodos (`col2row`) e coordenadas `RC(row,col)`. |
| **`jkb_left.overlay`** | [`config/boards/shields/jkb/jkb_left.overlay`](config/boards/shields/jkb/jkb_left.overlay) | Pinagem física (GPIOs) das 6 colunas e 5 linhas da metade esquerda. |
| **`jkb_right.overlay`** | [`config/boards/shields/jkb/jkb_right.overlay`](config/boards/shields/jkb/jkb_right.overlay) | Pinagem física (GPIOs) das 7 colunas e 5 linhas da metade direita + `col-offset = <6>`. |
| **`jkb_left.conf`** | [`config/boards/shields/jkb/jkb_left.conf`](config/boards/shields/jkb/jkb_left.conf) | Configurações de energia, Bluetooth (`+8 dBm`) e HID da metade esquerda. |
| **`jkb_right.conf`** | [`config/boards/shields/jkb/jkb_right.conf`](config/boards/shields/jkb/jkb_right.conf) | Configurações de energia, Bluetooth (`+8 dBm`) e HID da metade direita. |
| **`jkb.conf`** | [`config/boards/shields/jkb/jkb.conf`](config/boards/shields/jkb/jkb.conf) | Arquivo de referência de configurações gerais e opções de RGB/Display comentadas. |

```text
zmk-config-JaquesKB/
├── .github/workflows/
│   └── build.yml
├── config/
│   ├── boards/shields/jkb/
│   │   ├── Kconfig.defconfig
│   │   ├── Kconfig.shield
│   │   ├── jkb.conf
│   │   ├── jkb.dtsi
│   │   ├── jkb_left.conf
│   │   ├── jkb_left.overlay
│   │   ├── jkb_right.conf
│   │   └── jkb_right.overlay
│   ├── jkb.keymap
│   └── west.yml
├── zephyr/
│   └── module.yml
├── build.yaml
└── README.md
```

---

## 3. Funcionamento do Hardware e Matriz de Pinos

A leitura das teclas é configurada em [`jkb.dtsi`](config/boards/shields/jkb/jkb.dtsi) utilizando o driver `zmk,kscan-gpio-matrix` com diodos no sentido `col2row` (as colunas são acionadas em nível alto `GPIO_ACTIVE_HIGH` e as linhas possuem *pull-down* interno `GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN`).

### 3.1. Pinagem do Lado Esquerdo ([`jkb_left.overlay`](config/boards/shields/jkb/jkb_left.overlay))
O lado esquerdo possui **6 colunas x 5 linhas = 30 teclas**, mapeadas nas colunas lógicas `0` a `5`:

| Função na Matriz | Índice Lógico | Pino `pro_micro` | Configuração GPIO |
| :--- | :---: | :---: | :--- |
| **Colunas (0 a 5)** | `Col 0` → `Col 5` | `7`, `6`, `5`, `4`, `3`, `2` | `GPIO_ACTIVE_HIGH` |
| **Linhas (0 a 4)** | `Row 0` → `Row 4` | `20`, `19`, `18`, `15`, `14` | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |

### 3.2. Pinagem do Lado Direito ([`jkb_right.overlay`](config/boards/shields/jkb/jkb_right.overlay))
O lado direito aplica um deslocamento de 6 colunas (`col-offset = <6>`) e utiliza **7 colunas x 5 linhas**, ocupando as colunas lógicas `6` a `12` para totalizar **30 teclas**:
- As linhas `0` e `1` usam 6 teclas cada (`Col 7` a `Col 12`).
- As linhas `2` e `3` usam 7 teclas cada (`Col 6` a `Col 12`, adicionando 2 teclas internas extras próximas ao centro).
- A linha `4` (inferior/polegar) usa 4 teclas (`Col 7` a `Col 10`).

| Função na Matriz | Índice Lógico | Pino `pro_micro` | Configuração GPIO |
| :--- | :---: | :---: | :--- |
| **Colunas (6 a 12)** | `Col 6` → `Col 12` | `9`, `8`, `7`, `6`, `21`, `20`, `19` | `GPIO_ACTIVE_HIGH` |
| **Linhas (0 a 4)** | `Row 0` → `Row 4` | `18`, `15`, `14`, `16`, `10` | `GPIO_ACTIVE_HIGH \| GPIO_PULL_DOWN` |

### 3.3. Mapa Físico de Coordenadas ([`jkb.dtsi`](config/boards/shields/jkb/jkb.dtsi))

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

## 4. Funcionamento do Keymap (Layers 0, 1 e 2)

Arquivo de definição: [`config/jkb.keymap`](config/jkb.keymap)

O keymap utiliza os códigos padrão do ZMK combinados com o layout **ABNT2** configurado no sistema operacional. Abaixo está a tabela de equivalência dos códigos especiais para ABNT2 usados na camada base:

| Código no ZMK | Caractere no S.O. (ABNT2) | Posição no Teclado |
| :--- | :---: | :--- |
| `&kp SEMI` | **`Ç`** | Ao lado da tecla `L` (`RC(2,11)`) |
| `&kp LBKT` | **`´` / `` ` ``** (Acento agudo / Crase) | Extrema direita da linha do `A S D F` (`RC(2,12)`) |
| `&kp SQT` | **`~` / `^`** (Til / Circunflexo) | Extrema direita da linha do `Z X C V` (`RC(3,12)`) |
| `&kp BSLH` | **`]` / `}`** | Tecla interna extra superior direita (`RC(2,6)`) |
| `&kp RBKT` | **`[` / `{`** | Última tecla inferior esquerda (`RC(4,5)`) |
| `&kp NUBS` | **`\` / `\|`** | Linha inferior esquerda ao lado de `LALT` (`RC(4,2)`) |
| `&kp FSLH` | **`;` / `:`** | Ao lado do ponto `.` na linha alfabética inferior (`RC(3,11)`) |
| `&kp INT1` | **`/` / `?`** | Linha inferior direita ao lado de `BSPC` (`RC(4,9)`) |

---

### Layer 0: Base ABNT2 Otimizada (`default_layer` em [`jkb.keymap`](config/jkb.keymap))
Camada principal de digitação otimizada ergonomicamente:
- **Alinhamento Alfabético e de Pontuação:** `B` posicionado na mão esquerda (`RC(3,5)`), mantendo `N`, `M`, `,`, `.` e `; :` alinhados na linha 3 da mão direita.
- **Duplo `Backspace`:** Disponível tanto no canto superior direito (`RC(0,12)`) quanto no cluster do polegar direito (`RC(4,8)`, ao lado do `ENTER`), reduzindo o esforço repetitivo do dedo mínimo direito.
- **Ativadores de Camada:** Acesso momentâneo para a **Layer 1** (`&mo 1`, no polegar esquerdo `RC(4,4)`) e para a **Layer 2** (`&mo 2`, na tecla interna do lado direito `RC(3,6)`).

```text
LADO ESQUERDO                                                   LADO DIREITO
┌───────┬───────┬───────┬───────┬───────┬───────┐               ┌───────┬───────┬───────┬───────┬───────┬───────┐
│  ESC  │   1   │   2   │   3   │   4   │   5   │               │   6   │   7   │   8   │   9   │   0   │ BSPC  │
├───────┼───────┼───────┼───────┼───────┼───────┤               ├───────┼───────┼───────┼───────┼───────┼───────┤
│  TAB  │   Q   │   W   │   E   │   R   │   T   │               │   Y   │   U   │   I   │   O   │   P   │  DEL  │
├───────┼───────┼───────┼───────┼───────┼───────┤       ┌───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│LSHIFT │   A   │   S   │   D   │   F   │   G   │       │  ] }  │   H   │   J   │   K   │   L   │   Ç   │  ´ `  │
├───────┼───────┼───────┼───────┼───────┼───────┤       ├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│ LCTRL │   Z   │   X   │   C   │   V   │   B   │       │ MO(2) │   N   │   M   │   ,   │   .   │  ; :  │  ~ ^  │
├───────┼───────┼───────┼───────┼───────┼───────┤       └───────┼───────┼───────┼───────┼───────┼───────┴───────┘
│ LWIN  │ LALT  │  \ |  │ SPACE │ MO(1) │  [ {  │               │ ENTER │ BSPC  │  / ?  │ RALT  │
└───────┴───────┴───────┴───────┴───────┴───────┘               └───────┴───────┴───────┴───────┘
```

---

### Layer 1: Navegação `ESDF` e Numpad (`lower_layer` em [`jkb.keymap`](config/jkb.keymap))
Ativada enquanto segura **`MO(1)`**:
- **Navegação `ESDF` na Mão Esquerda:** Setas direcionais alinhadas sobre a *Home Row* (`E` = Cima, `S` = Esquerda, `D` = Baixo, `F` = Direita), mantendo o indicador sobre a referência tátil da tecla `F`. Inclui `HOME` (`W`), `END` (`R`), `PG_UP` (`T`), `PG_DN` (`G`) e `CAPS` (`Q`).
- **Numpad Completo na Mão Direita:** Bloco numérico completo com operadores (`+`, `-`, `*`, `/`, `.`, `,` e `KP_ENTER`).

```text
LADO ESQUERDO                                                   LADO DIREITO
┌───────┬───────┬───────┬───────┬───────┬───────┐               ┌───────┬───────┬───────┬───────┬───────┬───────┐
│       │       │       │       │       │       │               │       │       │  KP / │  KP * │       │       │
├───────┼───────┼───────┼───────┼───────┼───────┤               ├───────┼───────┼───────┼───────┼───────┼───────┤
│       │ CAPS  │ HOME  │  UP   │  END  │ PG_UP │               │       │ KP 7  │ KP 8  │ KP 9  │  KP - │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       ┌───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│       │       │ LEFT  │ DOWN  │ RIGHT │ PG_DN │       │       │       │ KP 4  │ KP 5  │ KP 6  │  KP + │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       ├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│       │       │       │       │       │       │       │ MO(2) │       │ KP 1  │ KP 2  │ KP 3  │ KP .  │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       └───────┼───────┼───────┼───────┼───────┼───────┴───────┘
│       │       │       │       │ MO(1) │       │               │       │ KP 0  │ KP ,  │KP ENT │
└───────┴───────┴───────┴───────┴───────┴───────┘               └───────┴───────┴───────┴───────┘
(Células em branco = &trans, herdam o comportamento da Layer 0)
```

---

### Layer 2: Bluetooth, Mídia e Funções (`raise_layer` em [`jkb.keymap`](config/jkb.keymap))
Ativada enquanto segura **`MO(2)`**:
- **Linha Superior (`Row 0`):** Teclas de função **`F1` a `F6`** na mão esquerda e **`F7` a `F12`** na mão direita.
- **Gerenciamento Bluetooth (Mão Esquerda, `Row 1`):** `BT_CLR` (limpa o pareamento do perfil ativo) e `BT_SEL 0` a `BT_SEL 4` (alterna entre até 5 dispositivos pareados).
- **Mídia e Sistema (Mão Direita, `Row 1` e `Row 2`):** Faixa Anterior (`C_PREV`), Play/Pause (`C_PP`), Próxima Faixa (`C_NEXT`), Print Screen (`PSCRN`), Mudo (`C_MUTE`), Volume - (`C_VOL_DN`) e Volume + (`C_VOL_UP`).
- **Manutenção de Firmware (`Row 3`, cantos externos):** `&bootloader` (entra em modo de gravação UF2 pela própria tecla, sem precisar abrir o case para apertar o botão físico) e `&sys_reset` (reinicia a controladora daquele lado).

```text
LADO ESQUERDO                                                   LADO DIREITO
┌───────┬───────┬───────┬───────┬───────┬───────┐               ┌───────┬───────┬───────┬───────┬───────┬───────┐
│  F1   │  F2   │  F3   │  F4   │  F5   │  F6   │               │  F7   │  F8   │  F9   │  F10  │  F11  │  F12  │
├───────┼───────┼───────┼───────┼───────┼───────┤               ├───────┼───────┼───────┼───────┼───────┼───────┤
│BT_CLR │ BT 0  │ BT 1  │ BT 2  │ BT 3  │ BT 4  │               │       │ PREV  │PLAY/PS│ NEXT  │ PSCRN │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       ┌───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│       │       │       │       │       │       │       │       │       │ MUTE  │ VOL - │ VOL + │       │       │
├───────┼───────┼───────┼───────┼───────┼───────┤       ├───────┼───────┼───────┼───────┼───────┼───────┼───────┤
│ BOOT  │ RESET │       │       │       │       │       │ MO(2) │       │       │       │       │ RESET │ BOOT  │
├───────┼───────┼───────┼───────┼───────┼───────┤       └───────┼───────┼───────┼───────┼───────┼───────┴───────┘
│       │       │       │       │ MO(1) │       │               │       │       │       │       │
└───────┴───────┴───────┴───────┴───────┴───────┘               └───────┴───────┴───────┴───────┘
(Células em branco = &trans, herdam o comportamento da Layer 0)
```

---

## 5. Ponto Atual do Projeto (Status)

- [x] Definição completa da matriz de 60 teclas (`5x13`) em [`jkb.dtsi`](config/boards/shields/jkb/jkb.dtsi) e mapeamento de GPIOs em [`jkb_left.overlay`](config/boards/shields/jkb/jkb_left.overlay) e [`jkb_right.overlay`](config/boards/shields/jkb/jkb_right.overlay).
- [x] Estrutura do shield padronizada em [`config/boards/shields/jkb/`](config/boards/shields/jkb/) com [`Kconfig.shield`](config/boards/shields/jkb/Kconfig.shield) e [`Kconfig.defconfig`](config/boards/shields/jkb/Kconfig.defconfig) revisados.
- [x] Pipeline de CI/CD em [`.github/workflows/build.yml`](.github/workflows/build.yml) e [`build.yaml`](build.yaml) compilando para `nice_nano//zmk` (`jkb_left`, `jkb_right` e `settings_reset`).
- [x] **Layer 0 (Base)** otimizada ergonomicamente para **ABNT2** em [`config/jkb.keymap`](config/jkb.keymap) (alinhamento natural de `B`, `N`, `M`, `,`, `.`, `; :` e segundo `BSPC` no polegar direito).
- [x] **Layer 1 (Navegação e Numpad)** com setas ergonômicas em `ESDF` + `HOME`/`END`/`PG_UP`/`PG_DN`/`CAPS` na mão esquerda e bloco numérico completo na mão direita.
- [x] **Layer 2 (Bluetooth, Mídia e Funções)** preenchida com `F1`–`F12`, seleção/limpeza de perfis Bluetooth (`BT_SEL 0..4`, `BT_CLR`), controles de mídia/volume, `PSCRN`, `&bootloader` e `&sys_reset`.
- [x] Potência do rádio Bluetooth amplificada (`+8 dBm`) e timeouts de economia de energia sincronizados em [`jkb_left.conf`](config/boards/shields/jkb/jkb_left.conf) e [`jkb_right.conf`](config/boards/shields/jkb/jkb_right.conf).

---

## 6. Como Compilar e Gravar o Firmware

1. **Gerar o Firmware ([`build.yml`](.github/workflows/build.yml)):**
   - A cada *push* no GitHub (ou executando manualmente na aba **Actions** → *Build ZMK firmware*), o GitHub compila as configurações definidas em [`build.yaml`](build.yaml) e disponibiliza o pacote `firmware.zip` nos *Artifacts* da execução.
2. **Gravar nas Controladoras (`nice!nano v2`):**
   - Conecte uma das metades ao PC via cabo USB e entre no modo Bootloader (segurando `MO(2)` + tecla do canto inferior externo `BOOT`, ou pressionando duas vezes o botão físico **RST** na placa).
   - Uma unidade USB chamada `NICENANO` aparecerá no computador.
   - Copie o arquivo `.uf2` correspondente (`jkb_left` para a controladora esquerda e `jkb_right` para a direita).
3. **Caso as metades percam a sincronia (Pareamento Split):**
   - Grave primeiro o arquivo `settings_reset` em **ambas** as controladoras para limpar a memória flash Bluetooth.
   - Em seguida, grave novamente o `jkb_left` e o `jkb_right` e ligue as duas metades próximas uma da outra.
