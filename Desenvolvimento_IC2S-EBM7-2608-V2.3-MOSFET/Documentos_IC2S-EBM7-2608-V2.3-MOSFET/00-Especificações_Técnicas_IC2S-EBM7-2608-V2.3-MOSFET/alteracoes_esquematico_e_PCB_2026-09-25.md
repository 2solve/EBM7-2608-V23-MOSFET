# Alteração 25-09-2026 — esquemático do sem_trilhas alinhado com a legacy + modo duplo reposto

Pedido de Javier Rivadineira (25-09-2026): levar ao esquemático do sem_trilhas as correcções da legacy e redimensionar o que o
MOSFET (selecção 0-10 V / 4-20 mA) exigir. PCB, biblioteca de footprints, regras e keepout ficam para a fase seguinte.

## Achado que motivou o redimensionamento

A decisão de 21-09 (carga de 110 R, só 4-20 mA) tinha sido aplicada também ao sem_trilhas: R27-R30 = 0 R e R31-R34 = 110 R.
Com isso o sem_trilhas tinha 110 R permanentes no OUT de cada TPS26613, em paralelo com o 100 R comutado pelo MOSFET:
em 0-10 V a entrada puxava 10 V / 110 Ω = 91 mA (o TPS26613 limita a 25-40 mA, SLVSFE3C p.5) — modo 0-10 V impossível.
O divisor original do modo duplo (nota 12 da folha 6, 2026-09-04) estava no backup `06_lacos.kicad_sch.antes_carga` (18-09):
R27-R30 = 40k2, R31-R34 = 10k.

## Alterações (folhas 01, 02, 06)

| Folha | Alteração |
|---|---|
| 01 | `#PWR0102` (P1.19) GND_24V → GND_ADC; nota 3 |
| 02 | `#PWR0202/0206/0209` GND_24V → GND_ADC (D200.2, C1.2, T7); **R200 removido** com os 2 fios; `#FLG0204` + `#PWR0210` + fio + 2 junções (PWR_FLAG redundante) removidos; T7 → `TP_GND_ADC_P1`; notas 6, 7, 12 e ERC |
| 06 | R27-R30 0R → **40k2 `RG1608P-4022-B-T5`**; R31-R34 110R → **10k `RG1608P-103-B-T5`**; C56 100 nF 0603 → **10 µF/100 V 1210 `GRM32EC72A106ME05L`** (CIN local do U15, como o C34 da legacy); texto "DECISAO 21-09" substituído por "DECISAO 25-09" (modo duplo); notas 14, 16, 17 |

MPN: `RG1608P-4022-B-T5` derivado do sistema de numeração Susumu (catálogo RG p.17: E-96 com 4 dígitos, P = ±25 ppm, B = 0,1 %);
substitui o `ERA-3AEB4022V` do backup, cujo datasheet não existe no projecto. `RG1608P-103-B-T5` e `GRM32EC72A106ME05L` já verificados.

## Dimensionamento (valores calculados)

| Grandeza | Valor | Limite / fonte | Veredicto |
|---|---|---|---|
| k do divisor | 10/50,2 = 0,1992 | — | — |
| Carga 4-20 (Q fechado) | 100 ‖ 50,2 k = 99,8 Ω | — | — |
| V(OUT) a 20 mA | 2,00 V | < 4,46 V (SLVSFE3C Tab. 8-2 nota 2, p.20) | ok |
| V(OUT) a IOL máx 40 mA | 3,99 V | < 4,46 V (IOL p.5) | ok |
| Tap 4-20 mA | 0,080-0,398 V | ganho > 1: ≥ AVSS−0,05 V (AD7124 rev.F p.7) | ok |
| Fim de escala ganho 4 | 31,4 mA | — | — |
| 0-10 V (Q aberto) | 50,2 kΩ; tap 1,99 V a 10 V | ganho 1 buffer: ≥ AVSS+0,1 V (p.7) → gama 0,50-12,5 V | 0-0,5 V fora de spec (já documentado, nota 13) |
| Filtro anti-aliasing | 8,0 k + 3k3, 100 nF → 141 Hz | — | — |
| Burden 20 mA | 40 mW | RG1608 0,1 W (Susumu p.18) | ok |
| Gate | VOH ≥ AVDD−0,6 = 2,7 V | AD7124 p.8 | depende do MOSFET |

## Verificação

- ERC sem_trilhas: 331 → 328 avisos, 0 erros; nenhum aviso novo (saem 3 `lib_symbol_issues` das peças removidas).
- Netlist: 169 → 168 componentes (sai R200); mudam só C56, R27-R34 e T7; pinos que mudam: C1.2, D200.2, P1.19, T7.1 (GND_24V → GND_ADC); a rede GND_24V deixa de existir.
- Texto renderizado (PDF do kicad-cli): movido 12 mm para a direita, já não sobrepõe o rótulo do C50.
- Backups: `01_conectores/02_entrada/06_lacos.kicad_sch.antes_sync_legacy`.
- Legacy: 2 linhas órfãs da nota 7 da folha 2 corrigidas (backup `02_entrada.kicad_sch.antes_notas_gnd`); ERC legacy 301, 0 erros.

## Pendentes

- **Q1-Q4 FDN337N e D26-D29 MMSZ4702T1G sem datasheet** no projecto (onsemi bloqueia descargas). As notas 13/14 decidiram
  AO3400A e BZT52H-C15, que têm PDF local. Falta confirmar RDS(on) a VGS 2,7 V, pinagem SOT-23 e fuga do zener a 12,3 V.
- **Opção a avaliar:** k = 0,1 (90k9/10k) com ganho 2 em 0-10 V e ganho 8 em 4-20 mA fecha a gama 0-0,5 V (ganho > 1 aceita
  AVSS−0,05 V). Muda o firmware.
- Fase PCB: footprint HVSSOP corrigido, C56 em 1210, zonas GND_24V, `.kicad_dru`, classes de rede, keepout do P1; a paridade
  vai acusar os valores e o footprint novos até se actualizar a PCB a partir do esquemático.

## Adenda — FDN337N e MMSZ4702T1G verificados (datasheets entregues pelo Javier)

`FDN337N-D.PDF` (onsemi Rev.5) e `MMSZ4678T1-D.PDF` (Rev.13) em `Documentos_de_Referência`.

| Verificação | Datasheet | Circuito | Veredicto |
|---|---|---|---|
| VDSS | 30 V (p.1) | dreno ≤ ~16,4 V em surto (zener) | ok, 1,8× |
| VGS máx | ±8 V (p.1) | GPIO do AD7124 ≤ AVDD 3,3 V | ok |
| VGS(th) | 0,4-1,0 V (p.2) | 0 V desligado (100k), ≥ 2,7 V ligado | ok |
| RDS(on) | ≤ 0,082 Ω a VGS 2,5 V (p.2); 125 °C ≈ ×1,7 | em série com 100 Ω | 0,08-0,14 %, vai na calibração |
| IDSS | ≤ 1 µA a 24 V; 10 µA a 55 °C (p.2) | só carrega a fonte 0-10 V | desprezável |
| Pinagem FDN337N | não numerada no PDF Rev.5 | footprint 1=G, 2=S, 3=D | ok, confirmada pelo Javier (25-09) |
| Land pattern | recomendação p.5 com pads maiores que os 0,6 × 1,475 do footprint | — | menor, fase PCB |
| VZ MMSZ4702 | 14,25-15,75 V (p.3) | OUT corta a 12,3 V | não conduz em serviço |
| IR | ≤ 0,05 µA a 11,4 V (p.3) | 0-10 V | desprezável |
| Polaridade | pino 1 = cátodo (p.1) | D26 pad 1 = BRD_AIN1, pad 2 = GND_ADC | ok |
| Surto | ZZT ≈ 2 Ω a 20 mA (Fig.5); Fig.4 ≈ 50 W a 0,3 ms | ~0,32 A, ~5 W de pico (48,4 V herdado da nota 13) | ok (extrapolação, critério D) |

Nota da folha 6 ("POR CONFIRMAR") reescrita com estes números; ERC 328, 0 erros. Backup `06_lacos.kicad_sch.antes_nota_mosfet`.
Incidente: a primeira escrita juntou as linhas com "n" em vez de `\n`; detectado no render, reposto do backup e reescrito.

## PCB do sem_trilhas importada do esquemático e colocada como na legacy (25-09-2026)

Pedido do Javier: se o esquemático estiver pronto, importar para a PCB e colocar os componentes como na legacy.
Pré-verificação: BOM do sem_trilhas = legacy + só selecção de modo (4 FDN337N, 4 MMSZ4702T1G, 4×100k, 4×100R, divisor
40k2/10k) + protecção de laços (4 MBR1H100SF, SMBJ51A em vez de SMBJ36A).

Correspondência pelo UUID do símbolo: os 148 footprints da legacy existem no sem_trilhas; contorno idêntico.
Script `st_pcb_sync.py` (cópia em scratch, verificação, depois cópia para o projecto):

| Passo | Resultado |
|---|---|
| Pistas/vias antigas | 114 apagadas (ficariam inválidas com a colocação nova) |
| R200 | removido (não está no esquemático) |
| 148 footprints comuns | posição, rotação, lado e campos iguais à legacy |
| U15, C56, P1, P2 | footprint copiado da legacy (pads TI do HVSSOP, C56 em 1210, courtyard ±1,52 dos conectores = biblioteca) |
| Valores/MPN | R27-R30 40k2 RG1608P-4022-B-T5, R31-R34 10k RG1608P-103-B-T5, T7 TP_GND_ADC_P1, U5 MPN do esquemático |
| Redes | 7 pads alterados; GND_24V desaparece |
| Zonas | as 11 da legacy (GND_ADC/DGND ×3, keepouts ISO7141, P1, Fid1-3) |
| 20 peças só do sem_trilhas | aparcadas fora do contorno, fila por canal: x 186 / 191,5 / 197 / 202,5 / 208 = 100R / FDN337N / MMSZ4702 / 100k / MBR1H100SF; y 86 / 96 / 106 / 116 = canal 1-4 |
| `.kicad_pro` | classe DIGSIDE; 90 padrões (por nome, por pads nos 6 nomes que mudam, por função nas redes novas: BRD_AINn e Net-(D32..35-A) em Loop, Net-(U12-OUT) em Loop) |
| `.kicad_dru` | novo: barreira_PLC_campo e U15_pads_NC (U12 na legacy) |
| Biblioteca | HVSSOP-8-1EP.kicad_mod da legacy (pads TI) |

Verificação independente (segundo método): 148/148 footprints e 444/444 pads na mesma posição da legacy, mesmos tamanhos e
camadas; 20 extras fora do contorno; zonas iguais (nome, vértices, rede).
No projecto: DRC 67 avisos de serigrafia, 0 erros, 336 por ligar (sem pistas), paridade 0; ERC 328 avisos, 0 erros.
Backups: `.kicad_pcb.antes_import_pcb`, `.kicad_pro.antes_import_pcb`, `footprints/HVSSOP-8-1EP.kicad_mod.antes_import_pcb`.

Pendente: colocar as 20 peças aparcadas (cada canal junto ao seu TPS26613 e ao seu fusível) e rotear; as 740 pistas da legacy
não foram copiadas (os laços são diferentes).

## Vias e pistas de massa copiadas da fixo_420 (25-09-2026)

Pedido do Javier: as zonas GND da configuravel não tinham o mesmo que a fixo_420. As zonas eram iguais (nome, vértices, rede)
e estavam cheias, mas sem nenhuma via/pista de GND_ADC e DGND (apagadas na importação). Copiadas as 233 da fixo_420
(78 vias: 69 GND_ADC + 9 DGND; 155 pistas), cada uma verificada contra os pads da configuravel: 0 tocam um pad de outra rede.
DRC no projecto: 0 erros, 67 avisos de serigrafia, paridade 0; por ligar 336 → 231; os 12 pendentes de GND_ADC são todos de
peças aparcadas fora da placa. Backup `.kicad_pcb.antes_gnd_vias`.

## Roteamento da fixo_420 copiado; ordem do H1 portada (25-09-2026)

Javier: "as áreas de GND estão erradas, copia como na 420". As zonas eram idênticas (definição linha a linha), mas o
enchimento diferia porque a configuravel não tinha o roteamento de sinal (furos das vias, pistas em B.Cu). Render das 3
camadas de massa lado a lado confirmou.

**Achado no caminho:** a folha 7 da fixo_420 teve duas revisões a 22-09 (`antes_ordem` 11:03, `antes_ordem2` 11:42) que
reordenaram o H1 — 1-2 +2V5_EXC, 3-4 +5V_ADC (omissão), 5-6 +3.3V_ANA — e que não tinham passado para a configuravel
(ficara 1-2 +5V_ADC, 3-4 +3.3V_ANA, 5-6 +2V5_EXC). Comparação pad a pad das duas placas: só 13 pads diferem — F3-F6.2 (díodo
em série, intencional), U6 pinos 5-10 (P1-P4 como GPIO do modo duplo, intencional) e H1.2/4/6 (esta falha). Portado à folha 7
(etiquetas, fios e nota 15 iguais à fixo_420; render comparado); ERC 328, 0 erros, só mudam H1.2/4/6. Backup
`07_celula.kicad_sch.antes_h1_ordem`.

Roteamento: 61 redes copiadas inteiras (550 itens) quando todos os pads caem numa só rede e nenhum item toca pad de outra;
as 10 redes que diferem (+24V_AIN1-4, AIN1-6_ADC) copiadas item a item e podadas pelo DRC (troço exacto por posição e
comprimento). Resultado: 690 pistas e 136 vias (fixo_420: 740 e 136). DRC 0 erros, 67 avisos de serigrafia, paridade 0;
por ligar 50 = 40 das 20 peças aparcadas + 6 fan-outs do U6 + 4 P2↔TVS (dependem de onde ficar o díodo). Backup
`.kicad_pcb.antes_rota_420`.

## PCB reconstruída a partir da fixo_420 (proposta do Javier, 25-09-2026)

"Copiar as posições e pistas dos componentes usados na 420 e na configurável; deixar fora, juntos numa área, os que só
existem na configurável." Método `st_rebase.py`: base = PCB da fixo_420 inteira. Nos 148 footprints comuns (mesmo UUID)
muda só a identidade (referência, valor, MPN e restantes campos, folha) e as redes dos pads (69), a partir da netlist da
configuravel. Pistas/vias: mantidas as 826 da fixo_420 cuja geometria já estava na configuravel depurada; removidas 50
(as que curto-circuitariam: saídas de F3-F6 antes do díodo, fan-out dos pinos GPIO do U6). As 20 peças só da configuravel:
dentro de um rectângulo em User.Drawings (182,5;81,5)-(211,5;117,0), fila = canal, com legenda.
Verificação: 148/148 footprints com conteúdo (geometria, serigrafia, textos) igual à fixo_420; zonas 11 = 11; vias 136 = 136.
DRC no projecto: 0 erros, 65 avisos (serigrafia), paridade 0, por ligar 50 (40 das peças por colocar + 10 a rotear).
Motivo: na versão anterior os textos ${REFERENCE} internos de 9 footprints ficavam os da configuravel (só os campos tinham
sido copiados). Backup `.kicad_pcb.antes_rebase`.

## Estrutura de pastas (25-09-2026)

Projecto antigo da raiz de `EBM7-2608-V23_configuravel` (placement de 16-17/09, 504 ficheiros) apagado a pedido do Javier;
cópia zip em scratch `out/configuravel_raiz_antiga_2026-09-25.zip`. O Javier subiu o conteúdo de `sem_trilhas` para a raiz e
apagou a subpasta. Conferido: mesma estrutura que a fixo_420 (projecto na raiz + footprints/ symbols/ .history/);
PCB e folhas = últimas versões (KiCad regravou: 1 padrão de netclass duplicado removido; retoque do Javier em +5V_ADC perto
de (133,8;114,6)). DRC 0 erros, 65 avisos de serigrafia, paridade 0, 50 por ligar; ERC 328, 0 erros.
