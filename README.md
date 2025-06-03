% Cavalo preto ('c')
movimento_valido(c,X,Y,NX,NY) :-
    member((DX,DY), [(2,1),(1,2),(-1,2),(-2,1),(-2,-1),(-1,-2),(1,-2),(2,-1)]),
    NX is X + DX,
    NY is Y + DY.

% Peão preto ('p') – só pode capturar nas diagonais à frente
movimento_valido(p,X,Y,NX,NY) :-
    NX is X + 1,
    (NY is Y + 1; NY is Y - 1).

% Rei preto ('r') – 1 casa em qualquer direção
movimento_valido(r,X,Y,NX,NY) :-
    member((DX,DY), [(-1,-1),(-1,0),(-1,1),(0,-1),(0,1),(1,-1),(1,0),(1,1)]),
    NX is X + DX,
    NY is Y + DY.

% Torre preta ('t') – linha ou coluna
movimento_longo(t, DX, 0) :- member(DX,[-1,1]).
movimento_longo(t, 0, DY) :- member(DY,[-1,1]).

% Bispo preto ('b') – diagonais
movimento_longo(b, DX, DY) :- member((DX,DY),[(-1,-1),(-1,1),(1,-1),(1,1)]).

% Dama preta ('d') – torre + bispo
movimento_longo(d, DX, DY) :- movimento_longo(t,DX,DY).
movimento_longo(d, DX, DY) :- movimento_longo(b,DX,DY).

% Encontra a peça na posição (Linha, Coluna)
get_peca(Tabuleiro,X,Y,Peca) :-
    nth0(X,Tabuleiro,Linha),
    nth0(Y,Linha,Peca).

% Verifica se uma peça preta ameaça o rei branco
ameaca_rei(Tab,X,Y,Peca) :-
    (movimento_valido(Peca,X,Y,NX,NY),
     dentro_do_tabuleiro(NX,NY),
     get_peca(Tab,NX,NY,'R')).

% Peças de longo alcance (torre, bispo, dama)
ameaca_rei(Tab,X,Y,Peca) :-
    movimento_longo(Peca,DX,DY),
    anda_em_direcao(Tab,X,Y,DX,DY).

% Anda até encontrar o rei branco ou parar
anda_em_direcao(Tab,X,Y,DX,DY) :-
    NX is X + DX,
    NY is Y + DY,
    dentro_do_tabuleiro(NX,NY),
    get_peca(Tab,NX,NY,Peca),
    ( Peca = 'R';
      Peca \= 8, !, fail;  % peça no caminho -> para
      anda_em_direcao(Tab,NX,NY,DX,DY)
    ).

% Confirma se coordenada está no tabuleiro
dentro_do_tabuleiro(X,Y) :-
    X >= 0, X < 8,
    Y >= 0, Y < 8.

% Procura o rei branco
posicao_rei(Tab,X,Y) :-
    nth0(X, Tab, Linha),
    nth0(Y, Linha, 'R').

% Procura peças pretas e testa ameaça
rei_em_xeque(Tab) :-
    posicao_rei(Tab,RX,RY),
    nth0(X,Tab, Linha),
    nth0(Y,Linha,Peca),
    atom(Peca), char_type(Peca,lower), % peça preta
    ameaca_rei(Tab,X,Y,Peca),
    RX = RX, RY = RY.

% Interface principal
esta_em_xeque(Tab,true) :- rei_em_xeque(Tab), !.
esta_em_xeque(_,false).
