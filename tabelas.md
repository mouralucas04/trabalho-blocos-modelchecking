<br>
Alunos:
Antonio Lucas Rodrigues
Breno Augusto Pinheiro
Caio da Silva Martins
Gabryella Eduarda Mattos Reis
Lucas da Silva Moura
Luiz Felipe Nery
Rafael Emanuel Dantas Viana
Victor José Nunes Kossmann
## Tabela 1
<br>

| Conceito                                | STRIPS                             | Prolog Estendido                                                        | Proposta de Modelo NuSMV                    | Justificativa para projeto NuSMV                                                        |
| --------------------------------------- | ---------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Block Properties**                    | `block(X).` — blocos sem atributos | `block(X). size(X,W).` — cada bloco tem largura fixa                    | `DEFINE size_X := W;` (constantes)          | Dimensões são imutáveis; representá-las como `DEFINE` melhora eficiência.               |
| **Table / Workspace (expandido)**       | `place(N).` (lugares abstratos)    | `table_slot(N). table_width(M).` (slots discretos)                      | `VAR pos_X : {table0,table1,...,on_Y,...};` | Representar a mesa como grade evita “lugares mágicos” e permite verificar contiguidade. |
| **Position Representation (expandido)** | `on(X,Y).` (ambíguo)               | `pos(X,table(N)).` / `pos(X,on(Y)).`                                    | `VAR pos_X` + `next(pos_X)`                 | Captura unificação de posição horizontal + suporte.                                     |
| **Clear Property (expandido)**          | `clear(X).`                        | `clear(X).` usado em `can/2`                                            | `DEFINE clear_X := !(∃B : pos_B=on_X);`     | Em NuSMV, `clear` vira fórmula derivada (não variável).                                 |
| **Derived Knowledge (expandido)**       | —                                  | `is_free(Slot)`, `busy_slots(Block,Slots)`, `absolute_pos(Block,Coord)` | `DEFINE free_j := !(∃B: j∈busy_slots(B));`  | Mantém estado mínimo, computando ocupação sob demanda.                                  |
<br>

## Tabela 2
<br>

| Tipo de Restrição                      | Destino      | Regra em Linguagem Natural                                                          | Implementação NuSMV (Ex: `move(C,A)`)               | Implementação NuSMV (Ex: `move(C,table(2))`)              |
| -------------------------------------- | ------------ | ----------------------------------------------------------------------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| **Mobility**                           | ambos        | Um bloco só pode se mover se estiver livre (sem nada em cima).                      | `TRANS move_C_A -> clear_C & ...`                   | `TRANS move_C_table2 -> clear_C & ...`                    |
| **Target Accessibility**               | `on(Target)` | Só pode colocar sobre outro bloco se o destino estiver livre.                       | `TRANS move_C_A -> clear_A & ...`                   | — (não aplicável para mesa).                              |
| **Stability**                          | `on(Target)` | Um bloco só pode ser colocado sobre outro de largura ≥ à dele.                      | `TRANS move_C_A -> size_C <= size_A & ...`          | — (mesa sempre estável).                                  |
| **Spatial Occupancy**                  | `table(X)`   | Ao colocar na mesa, todos os slots necessários devem estar livres.                  | —                                                   | `TRANS move_C_table2 -> ∀j∈[2..2+size_C−1] free(j) & ...` |
| **Logical Validity**                   | ambos        | Um bloco não pode ser colocado sobre si mesmo.                                      | `TRANS move_C_A -> C != A & ...`                    | — (mesa não é bloco, logo não ocorre).                    |
| **Index Validity (expandido)**         | `table(X)`   | A coordenada X deve estar dentro da largura da mesa.                                | —                                                   | `TRANS move_C_table2 -> X+size_C−1 < table_width & ...`   |
| **Clear Propagation (expandido)**      | ambos        | Ao mover um bloco, o suporte antigo vira `clear`, e o destino deixa de ser `clear`. | `next(clear_oldSupport)=TRUE; next(clear_A)=FALSE;` | `next(clear_oldSupport)=TRUE;`                            |
| **Energy / Cost (expandido)**          | ambos        | Cada movimento tem custo acumulado (opcional).                                      | `next(cost)=cost+1;`                                | `next(cost)=cost+1;`                                      |
| **Rotation / Orientation (expandido)** | ambos        | Um bloco pode ser rotacionado mudando largura/profundidade.                         | `TRANS rotate_C -> size_C := swap(W,D);`            | `TRANS rotate_C -> DEFINE width_C’=depth_C;`              |
