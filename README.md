# RV32I-CIAA

Procesador RISC-V **RV32I de ciclo único**, escrito en Verilog e implementado sobre la FPGA de la placa **EDU-CIAA-FPGA** (Lattice iCE40), usando el toolchain open-source [Apio](https://github.com/FPGAwars/apio) (Yosys + nextpnr + IcePack).

Sigue el datapath clásico de ciclo único de *Digital Design and Computer Architecture: RISC-V Edition* (Harris & Harris), con unidad de control, banco de registros y memorias de instrucciones/datos como módulos independientes.

## Características

- Núcleo RV32I de **ciclo único** (un ciclo de reloj por instrucción).
- Subconjunto de instrucciones soportado:
  - Tipo R: `add`, `sub`, `and`, `or`, `slt`
  - Tipo I: `addi`, `lw`
  - Tipo S: `sw`
  - Tipo B: `beq`
  - Tipo J: `jal`
- Banco de registros de 32x32 bits con salidas expuestas para debug (`x5`, `x6`, `x7`, `x8`, `x9`, `x18`).
- ROM de instrucciones precargada con un programa de prueba (operaciones aritmético-lógicas, bucle `while`, bucle `for` y acceso a memoria de datos).
- Testbenches individuales por módulo (`PC`, `ALU`, `ALU Decoder`, `Branch`, `Data Memory`, `Instruction Memory`, `Mux2x1`, `Sign Extension`) y testbenches de integración (`datapath`, `mainDeco`, `UC`, `rv32i`).

## Arquitectura

```
                 ┌────────────┐
   reset ───────▶│            │
   clk   ───────▶│    PC      │──┐
                 └────────────┘  │
                                  ▼
┌─────────┐   instr   ┌─────────────────────┐   aluResult   ┌─────────┐
│   ROM   │──────────▶│                      │──────────────▶│   DM    │
│ (IM)    │           │      DataPath        │   writeData   │         │
└─────────┘           │  (BR, SE, Mux, ALU)  │◀───────────────│         │
                       │                      │   readData    └─────────┘
                       └──────────┬───────────┘
                                  │ op / func3 / func7 / zero
                                  ▼
                       ┌─────────────────────┐
                       │ Control Unit (UC)   │
                       │  mainDeco + aluDeco │
                       └─────────────────────┘
```

## Estructura del repositorio

```
.
├── apio.ini                  # Configuración de build para la EDU-CIAA-FPGA
├── src/
│   ├── modules/               # Módulos del procesador (PC, ALU, UC, BR, DM, ROM, etc.)
│   │   ├── rv32i.v             # Top-level: conecta ROM, DM y DataPath
│   │   ├── datapath.v          # Camino de datos
│   │   ├── uc.v                # Unidad de control (mainDeco + aluDeco)
│   │   ├── mainDeco.v / aluDeco.v
│   │   ├── pc.v / br.v / alu.v / dm.v / rom.v / se.v / mux2x1.v
│   │   └── *_tb.v              # Testbenches de integración
│   └── testbenches/            # Testbenches unitarios por módulo
└── hardware.json / hardware.out # Artefactos de síntesis (Yosys)
```

## Requisitos

- [Apio](https://github.com/FPGAwars/apio) (incluye Yosys, nextpnr-ice40 e IcePack)
- [Icarus Verilog](http://iverilog.icarus.com/) para simulación (`iverilog` + `vvp`)
- [GTKWave](http://gtkwave.sourceforge.net/) (opcional, para visualizar los `.vcd`)
- Placa **EDU-CIAA-FPGA** y su programador para cargar el bitstream

## Build y programación de la FPGA

```bash
apio build       # Sintetiza el diseño (Yosys + nextpnr + IcePack)
apio upload       # Programa la EDU-CIAA-FPGA con el bitstream generado
```

## Simulación

Cada módulo cuenta con su propio testbench. Por ejemplo, para simular el procesador completo:

```bash
cd src/modules
iverilog -o rv32i_tb.out rv32i_tb.v rv32i.v pc.v br.v alu.v aluDeco.v mainDeco.v uc.v dm.v rom.v se.v mux2x1.v
vvp rv32i_tb.out
gtkwave rv32i_tb.vcd
```

El programa precargado en la ROM (`rom.v`) ejecuta operaciones aritmético-lógicas, un bucle `while`, un bucle `for` y accesos a memoria de datos (`sw`/`lw`), útil para verificar el funcionamiento del datapath completo.

## Créditos

Diseño basado en el datapath de ciclo único de *Digital Design and Computer Architecture: RISC-V Edition* (Sarah L. Harris & David Money Harris).
