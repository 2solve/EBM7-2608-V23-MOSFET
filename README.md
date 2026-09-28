# IC2S EBM7-2608 V2.3 — variante configurável 0-10 V / 4-20 mA (MOSFET)

Placa de extensão da família IC2S (Extension Board Model 7) com **quatro entradas analógicas configuráveis por software**
(cada uma em 0-10 V **ou** 4-20 mA), **uma entrada de célula de carga** e **um NTC de junta fria**, medidos por um ADC
sigma-delta de 24 bits (**AD7124-8**). Encaixa por baixo na base board: **P1** é o barramento (24 V, 5 V, SPI) e **P2** leva os
sinais de campo.

É a irmã da variante fixa 4-20 mA (repositório `2solve/EBM7-2608-V3`): mesma placa, mesmo layout de base, mais um MOSFET por
canal para escolher o modo e uma protecção completa da alimentação dos sensores.

Projecto KiCad 10 (testado em 10.0.5). Abrir
`Desenvolvimento_IC2S-EBM7-2608-V2.3-MOSFET/Projeto_IC2S-EBM7-2608-V2.3-MOSFET/IC2S-EBM7-2608-V2.3-MOSFET.kicad_pro`.

Identificador desta versão: **`IC2S-EBM7-2608-V2.3-MOSFET`**.

---

## Estado (28-09-2026)

| Verificação | Resultado |
|---|---|
| ERC | 0 erros |
| DRC (`--severity-all --schematic-parity`) | **0 erros**, 67 avisos (serigrafia) |
| Ligações por fazer | **48** |
| Paridade esquemático ↔ PCB | **1** (C35, ver [Pendentes](#pendentes)) |

**O esquemático está fechado; a placa não.** A PCB foi construída sobre a da variante fixa: as 148 peças comuns estão na mesma
posição e com as mesmas pistas; as **20 peças que só existem nesta variante** estão juntas num rectângulo à direita da placa,
à espera de serem colocadas e roteadas. Por isso as pastas de fabricação estão vazias.

---

## Estrutura do repositório

Padrão de pastas dos produtos da 2Solve (o mesmo de `Produtos/Axcel_Sensor`):

```
Desenvolvimento_IC2S-EBM7-2608-V2.3-MOSFET/
├── Projeto_IC2S-EBM7-2608-V2.3-MOSFET/
│   ├── IC2S-EBM7-2608-V2.3-MOSFET.kicad_pro / .kicad_sch / .kicad_pcb / .kicad_dru
│   ├── 01_conectores … 08_ntc.kicad_sch          (as oito folhas)
│   ├── footprints/EBM7_V23.pretty, symbols/       (bibliotecas locais, por ${KIPRJMOD})
│   ├── Documentos_de_Referência-…/                (33 datasheets + catálogo TDK de MLCC)
│   └── Outputs/drc_report.json
├── Documentos_IC2S-EBM7-2608-V2.3-MOSFET/
│   ├── 00-Especificações_Técnicas_…/   registos das alterações (25-09 e 28-09)
│   ├── 01-Esquemáticos_…/              Schematic-….pdf, ….step, …-Top_View.pdf, …-Bottom_View.pdf
│   ├── 02-Pinout_…/                    (vazio)
│   └── 03-Diagrama_Blocos_…/           (vazio)
└── Arquivos_Fabricação_IC2S-EBM7-2608-V2.3-MOSFET/
    ├── 00-Gerbers_…/  01-BOM_List_…/  02-Pick-and-Place_…/  03-Stencil_…/   (vazios até a placa estar roteada)
```

**Onde está cada decisão:** cada folha tem um bloco «DECISOES DESTA FOLHA» com notas numeradas; aqui cita-se *folha N, nota M*.

---

## O que é igual à variante fixa

Os blocos seguintes são os mesmos da fixa (lá estão explicados em detalhe); aqui só as referências, que mudam entre projectos:

| Bloco | Nesta placa |
|---|---|
| Entrada 24 V | **ramo único** (28-09-2026, folha 2, nota 13): TVS **D200** SMA6J33A-Q (clamp real 53,3 V) → fusível **F1** CC12H750MA → díodo **D5** MBR1H100SF → +24V_ADC; **C13** 10 µF/100 V na entrada do U3 (Recom p.4 pede 3,3 µF/100 V se Vin > 50 V) |
| Massas | GND_24V fundido em GND_ADC (folha 2, nota 7); DGND é a massa do processador |
| Alimentação | **U3** R-78HB5.0 (+24V_ADC → 5 V), **U4** SPX3819 (+3.3V_ANA), **U8** SPX3819 (+3.3V do lado PLC; era MCP1824 até 28-09), **U14** TL431 que grampeia o +3.3V_ANA a ~3,6 V (folha 3, nota 14) |
| Barreira | **U5** ISO7141: separação de ruído, não isolamento (folha 4, nota 1); regra de 1,0 mm e keepouts no PCB |
| ADC | **U6** AD7124-8, referência **U7** ADR4525 2,5 V no REFIN2 (folha 5, nota 1) |
| Protector dos laços | **U9-U12** TPS26613, +Vs a 12 V de **U15** TPS7A4001 (folha 6, notas 9 e 16) |
| Célula | **H1**: 1-2 = 2,5 V, 3-4 = 5 V (omissão), 5-6 = 3,3 V; **D31** Schottky RB162MM-60; **R5** 68 Ω 1 W (folha 7, notas 15 e 16) |
| NTC | junta fria para termopar (folha 8) |

---

## O que muda: modo duplo por canal

Decisão da folha 6: caixa **«DECISAO 25-09-2026 - MODO DUPLO»** e nota 12. Firmware: folha 5, nota 10.

Cada canal tem **dois caminhos** no P2:

- **Alimentação do sensor** (pino par, `+24V_AINn`): +24V_ADC → fusível **F3-F6** (125 mA) → díodo **D32-D35**
  (MBR1H100SF) → borne, com o TVS **D13/D16/D19/D22** (SMBJ51A) junto ao conector.
- **Sinal** (pino ímpar, `AINn`): borne → TVS 824500301 → **TPS26613** → divisor **40k2 / 10k** (sempre ligado) e, em
  paralelo, **burden 100 Ω** comutado pelo MOSFET → ferrite → RC → clamps BAV199 → ADC.

| Modo | MOSFET (Q1-Q4, FDN337N) | O que o sinal vê | No ADC |
|---|---|---|---|
| **4-20 mA** | fechado (GPIO = 1) | 100 Ω ‖ 50,2 kΩ = 99,8 Ω → 2,00 V a 20 mA | tap 0,080-0,398 V, ganho 4, fim de escala 31,4 mA |
| **0-10 V** | aberto (GPIO = 0) | 50,2 kΩ de entrada | tap 1,99 V a 10 V, ganho 1, gama garantida 0,5-12,5 V |

- A porta de cada MOSFET é comandada por uma **saída digital do próprio AD7124** (P1-P4 = pinos 6-9), com 100 kΩ a GND. No
  arranque a porta está em 0: o canal começa em 0-10 V; com um transmissor 4-20 mA ligado há hiccup até o firmware
  configurar o modo (folha 6, nota 13).
- **D26-D29** (MMSZ4702, zener de 15 V) protegem o dreno do MOSFET contra os surtos do campo.
- Verificações (na caixa da folha 6): nota (2) da Tabela 8-2 do TPS26613 cumprida mesmo a 40 mA (3,99 V < 4,46 V); FDN337N
  com RDS(on) ≤ 0,082 Ω a 2,5 V (0,08 % da carga, vai na calibração); zener não conduz em serviço (corte do OUT a 12,3 V).
- **Limitação:** abaixo de 0,5 V, em 0-10 V, o ADC com buffer e ganho 1 sai da especificação (AVSS + 0,1 V). Alternativa
  avaliada e **não aplicada** (muda o firmware): divisor 90k9/10k com ganho 2 em 0-10 V e ganho 8 em 4-20 mA.

**Mapa de canais** (da netlist):

| Canal de campo (P2) | Entrada do ADC | Saída que comanda o modo |
|---|---|---|
| AIN1 (P2.7) | AIN0 (pino 4) | P4 (pino 9) → MODO_AIN1 → Q1 |
| AIN2 (P2.9) | AIN1 (pino 5) | P3 (pino 8) → MODO_AIN2 → Q2 |
| AIN3 (P2.11) | AIN7 (pino 11) | P2 (pino 7) → MODO_AIN3 → Q3 |
| AIN4 (P2.13) | AIN6 (pino 10) | P1 (pino 6) → MODO_AIN4 → Q4 |
| Célula +/− | AIN10 / AIN11 | — |

## O que muda: protecção da alimentação dos sensores

Folha 6, nota 1, e folha 2, nota 9. Na variante fixa o fusível fica entre dois sumidouros de corrente (os condensadores do
bus e o TVS do borne) e um surto forte abre-o. Aqui:

- o **díodo em série** (D32-D35) impede que um surto do campo volte para o bus;
- o **SMBJ51A** (V_BR ≥ 56,7 V) não conduz nos surtos do bus, porque o TVS de entrada D200 grampeia a ≤ 53,3 V;
- assim a corrente de clamp **nunca passa pelo fusível**. Custo: ~0,5 V de queda na alimentação do sensor.

## Alterações de 28-09-2026 (reunião)

| O quê | Porquê |
|---|---|
| **Ramo único de 24 V**: saem F2, D1 e C1; F1 + D5 alimentam o U3 e o +24V_ADC; +24V_REG deixa de existir | reunião: é tudo alimentado pelos mesmos 24 V, não eram canais independentes nem isolados; ganha-se espaço |
| 100 nF fora do 24 V → TDK C1608X7R1H104K080AA (50 V), 24 peças; o C6 (+24V_ADC) fica GRM188R72A104KA35D (100 V) | 100 V estava sobredimensionado; pior caso fora do 24 V é o +12V_TPS a 31,7 V, por isso 50 V e não 25 V |
| 1 µF → TDK C1608X7R1H105K080AB (50 V) nos quatro | havia dois MPN para a mesma função |
| U8 MCP1824ST (SOT-223) → SPX3819 (SOT-23-5); faltam rotear 6 ligações do U8 | mesmo MPN do U4 (`alteracao_LDO_3V3_lado_PLC_2026-09-28.md`) |

Condensadores: folha 2, nota 14. O catálogo TDK usado está em `Documentos_de_Referência-…/TDK_mlcc_commercial_general_en.pdf`
(p.34 para o 100 nF, p.35 para o 1 µF).

## Outras diferenças

- **C56 = 10 µF / 100 V em 1210** (GRM32EC72A106ME05L): condensador de entrada do U15 junto ao pino (SBVS162B pede > 1 µF
  efectivo).
- Footprint do **U15** conforme o land pattern da TI (SBVS162B p.22), com excepção dos pads NC no `.kicad_dru`.

---

## Placa (PCB)

- 60 × 60 mm, 4 camadas: F.Cu sinal · In1/In2 planos de massa partidos (DGND | GND_ADC) · B.Cu sinal e massa.
- Colocação, pistas, vias, zonas e keepouts das 148 peças comuns copiados da variante fixa (25-09-2026). 50 pistas da fixa
  **não** foram copiadas porque aqui dariam curto (saídas de F3-F6 antes do díodo; pinos do ADC que passaram a ser GPIO).

---

## Notas do esquemático desactualizadas

- **Folha 6, notas 13 e 14:** dizem AO3400A e BZT52H-C15; as peças montadas são **FDN337N** e **MMSZ4702T1G** (verificadas
  contra os datasheets na caixa «DECISAO 25-09»).
- **Folha 3, nota 1:** diz U3 = K78U05; a peça é o **R-78HB5.0-0.5L**.
- **Folha 7, nota 11:** diz TPS7A2025 para o U13; a peça montada é o **LP5907MFX-2.5**.

---

## Documentos

Em `Documentos_…/00-Especificações_Técnicas_IC2S-EBM7-2608-V2.3-MOSFET/`:

| Ficheiro | Conteúdo |
|---|---|
| `alteracoes_esquematico_e_PCB_2026-09-25.md` | registo das alterações de 25-09: modo duplo reposto, correcções vindas da fixa, verificação do FDN337N/MMSZ4702, importação e colocação da PCB |
| `alteracoes_reuniao_2026-09-28.md` | revisão da reunião de 28-09: ramo único de 24 V, entrada do R-78HB, tensão dos 100 nF, MPN do 1 µF |

---

## Pendentes

- **Paridade C35:** o pad 2 do C35 está ligado por pista a +3.3V_ANA (via em 162,25; 84,65 → pad), mas no esquemático é
  GND_ADC (é o condensador de filtro de AIN5 para a massa). Corrigir a pista na PCB.
- **Colocar as 20 peças** do rectângulo «SO CONFIGURAVEL» (fila = canal) junto de cada TPS26613 e do seu fusível, e **rotear
  as 48 ligações** que faltam.
- Serigrafia (67 avisos).
- Gerar Gerbers, BOM, pick-and-place e stencil quando a placa estiver roteada.
- Datasheets em falta: SMBJ51A-13-F (Diodes), MBR1H100SFT3G (onsemi), BLM18PG471SN1D (Murata), LED KG EELP41.22,
  resistências genéricas.
- Revisão independente do roteamento (`2shw-pcb:check-roteamento`) antes de fabricar.
- O mecanismo da avaria em campo da V2.2 não foi medido (G8).
