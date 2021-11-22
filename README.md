# Cyclides


## Introduction

We present Mathematica code for the automatic verification
in the proof of Proposition 4 in the article "Translational and great Darboux cyclides".

For running the code copy paste the code presented below in [Mathematica](https://www.wolfram.com/mathematica/trial/).
The same code can be found in the Mathematica file [product.nb](https://github.com/niels-lubbes/ns_lattice/blob/master/ns_lattice/bin/product.nb).

__Initialization__
In the following code we use the notation and definitions from the article.
```Mathematica
Remove["Global`*"] (* clear all variables *)

(* classes in E(X) *)
e1 = {0, 0, 1, 0, 0, 0};e2 = {0, 0, 0, 1, 0, 0};e3 = {0, 0, 0, 0, 1, 0};e4 = {0, 0, 0, 0, 0, 1};e01 = {1, 0, -1, 0, 0, 0};e02 = {1, 0, 0, -1, 0, 0};e03 = {1, 0, 0, 0, -1, 0};e04 = {1, 0, 0, 0, 0, -1};e11 = {0, 1, -1, 0, 0, 0};e12 = {0, 1, 0, -1, 0, 0};e13 = {0, 1, 0, 0, -1, 0};e14 = {0, 1, 0, 0, 0, -1};ep1 = {1, 1, 0, -1, -1, -1};ep2 = {1, 1, -1, 0, -1, -1};ep3 = {1, 1, -1, -1, 0, -1};ep4 = {1, 1, -1, -1, -1, 0};

(* classes in G(X) *)
g0 = {1, 0, 0, 0, 0, 0};g1 = {0, 1, 0, 0, 0, 0};g2 = {2, 1, -1, -1, -1, -1};g3 = {1, 2, -1, -1, -1, -1};g12 = {1, 1, -1, -1, 0, 0};g34 = {1, 1, 0, 0, -1, -1};

(* classes in B(X) *)
b12 = {1, 0, -1, -1, 0, 0};b13 = {1, 0, -1, 0, -1, 0};b24 = {1, 0, 0, -1, 0, -1};b34 = {1, 0, 0, 0, -1, -1};bp12 = {0, 1, -1, -1, 0, 0};bp13 = {0, 1, -1, 0, -1, 0};bp14 = {0, 1, -1, 0, 0, -1};bp23 = {0, 1, 0, -1, -1, 0};bp24 = {0, 1, 0, -1, 0, -1};bp34 = {0, 1, 0, 0, -1, -1};b0 = {1, 1, -1, -1, -1, -1};b1 = {0, 0, 1, 0, -1, 0};b2 = {0, 0, 0, 1, 0, -1};


(* Converts class to string. *)
str[q_] := Module[{},If[q == e1, Return["e1"]];If[q == e2, Return["e2"]];If[q == e3, Return["e3"]];If[q == e4, Return["e4"]];If[q == e01, Return["e01"]];If[q == e02, Return["e02"]];If[q == e03, Return["e03"]];If[q == e04, Return["e04"]];If[q == e11, Return["e11"]];If[q == e12, Return["e12"]];If[q == e13, Return["e13"]];If[q == e14, Return["e14"]];If[q == ep1, Return["ep1"]];If[q == ep2, Return["ep2"]];If[q == ep3, Return["ep3"]];If[q == ep4, Return["ep4"]];If[q == g0, Return["g0"]];If[q == g1, Return["g1"]];If[q == g12, Return["g12"]];If[q == g34, Return["g34"]];If[q == g2, Return["g2"]];If[q == g3, Return["g3"]];Return[ToString[q]]];

(* Matrix for quadratic form defining intersection product between classes. *)
M = {{0, 1, 0, 0, 0, 0}, {1, 0, 0, 0, 0, 0}, {0, 0, -1, 0, 0, 0},{0, 0, 0, -1, 0, 0}, {0, 0, 0, 0, -1, 0}, {0, 0, 0, 0, 0, -1}};

(* Default real structure. We may overwrite this global method for different real structures. *)
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]};
```


__Auxiliary methods__

The following method is an implementation of the odot multiplication as defined in Section 5.
The inputs u and v are elements of E(X) or G(X). The input BB is a list consisting of the
elements in B(X).
We assume that B(X) is defined as one of the rows in
Table 2 and Table 3. We remark that if X is not EY cyclide or CY cyclide,
then a component in B(X) consists of at most 2 elements (see the sng X column in Table 2).
We use this fact to simplify the implementation of this method.

```Mathematica
odot[u_, v_, BB_] := Module[{W, i},
    If[u.M.v>0 , Return[1]];
    For[i=1, i<=Length[BB], i++,
        If[u.M.BB[[i]]>0 && v.M.BB[[i]]>0, Return[1]];
    ];
    W = Select[Subsets[BB, {2}], #[[1]].M.#[[2]]>0 &];
    For[i=1, i<=Length[W], i++,
        If[u.M.W[[i,1]]>0 && v.M.W[[i, 2]]>0, Return[1]];
    ];
    Return[0];
];
```

If the following methods returns True, then the four elements q1, q2, q3 and q4 in E(X) form a Clifford quartet w.r.t. elements in B(X).
Notice that if {q1,q2,q3,q4} is a Clifford quartet, then this method still may return False, as it assumes a certain ordering on the
elements. Thus we need to call this method with all permutations of {q1,q2,q3,q4}.

```Mathematica
isQuartet[q1_, q2_, q3_, q4_, BB_] :=
    If[inv[q1] == q2 && inv[q3] == q4 &&
    odot[q1, q2, BB] == 0 && odot[q3, q4, BB] == 0 &&
    odot[q1, q3, BB] == 1 && odot[q3, q2, BB] == 1 &&
    odot[q2, q4, BB] == 1 && odot[q4, q1, BB] == 1,
    Return[True],(* else *) Return[False]];
```


The following method checks for class e in E(X) and Clifford quartet A={a,b,c,d} whether
`e.M.a==1` and `e*b=e*c=e*d=0` where `*` stands for the odot product.

```Mathematica
isCross[e_, A_, a_, BB_] := Module[{j},
    If[MemberQ[A, e] || e.M.a != 1, Return[False]];
    For[j = 1, j <= 4, j++,
        If[A[[j]] != a && odot[e, A[[j]], BB] != 0, Return[False]];
    ];
    Return[True];
];
```

The input T in the following method consists of classes in G(X).
We verify whether there exists `g` in `T` such that `g.M.u!=0` for all `u` in `U`.

```Mathematica
check[T_, U_] := Module[{i},
    For[i = 1, i <= Length[T], i++,
        If[!MemberQ[Map[#.M.T[[i]]&, U], 0], Return[True]];
    ];
    Return[False];
];
```

__The main method__

Suppose that B(X), E(X) and G(X) corresponds to one of the 14 rows in Table 3.
We let BB=B(X), EE=E(X) and the list GT consist of all the tracing elements in G(X).
This method prints a list of all tuples (A,a,T,U) that satisfy the following property:
(A,a,g,U) is a Clifford data if and only if g in T.
For each such tuple we print True if and only if there exists g in T, such that the Clifford data (A,a,g,U) is
a certificate. The existence of a certificate implies that X satifies the Clifford criterion.

```Mathematica
comp[BB_, EE_, GT_] := Module[{Q, i, j, A, a, T, U},

    (* We compute a list of all Clifford quartets, using exhaustive search *)
    Q = DeleteDuplicatesBy[Sort]@Select[Permutations[EE, {4}],isQuartet[#[[1]], #[[2]], #[[3]], #[[4]], BB] &];
    If[Length[Q] == 0, Print["There exist no Clifford quartets."]];

    (* We compute all tuples (A,a,T,U) *)
    For[i = 1, i <= Length[Q], i++,
        For[j = 1, j <= 4, j++,
            A = Q[[i]]; a = Q[[i, j]];
            T = Select[GT, #.M.a > 0&];
            U = Select[EE, isCross[#, A, a, BB]&];
            Print[str /@ A, " ", str[a], " ", str /@ T, " ", str /@ U, " "," Certificate for Clifford criterion: " <> ToString[check[T, U]]];
        ];
    ];
];
```

__Blum cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]};   (* real structure of type 2A1 *)
BB = {};
EE = {e1, e2, e3, e4, e01, e02, e03, e04, e11, e12, e13, e14, ep1,
   ep2, ep3, ep4};
GT = {g0, g1, g12, g34, g2, g3};
comp[BB, EE, GT]
```

Output:

    {e1,e2,ep3,ep4} e1 {g12,g2,g3} {e01,e11,ep2}  Certificate for Clifford criterion: False
    {e1,e2,ep3,ep4} e2 {g12,g2,g3} {e02,e12,ep1}  Certificate for Clifford criterion: False
    {e1,e2,ep3,ep4} ep3 {g0,g1,g34} {e4,e03,e13}  Certificate for Clifford criterion: False
    {e1,e2,ep3,ep4} ep4 {g0,g1,g34} {e3,e04,e14}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} e3 {g34,g2,g3} {e03,e13,ep4}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} e4 {g34,g2,g3} {e04,e14,ep3}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} ep1 {g0,g1,g12} {e2,e01,e11}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} ep2 {g0,g1,g12} {e1,e02,e12}  Certificate for Clifford criterion: False
    {e01,e02,e13,e14} e01 {g1,g34,g3} {e1,e12,ep1}  Certificate for Clifford criterion: False
    {e01,e02,e13,e14} e02 {g1,g34,g3} {e2,e11,ep2}  Certificate for Clifford criterion: False
    {e01,e02,e13,e14} e13 {g0,g12,g2} {e3,e04,ep3}  Certificate for Clifford criterion: False
    {e01,e02,e13,e14} e14 {g0,g12,g2} {e4,e03,ep4}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e03 {g1,g12,g3} {e3,e14,ep3}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e04 {g1,g12,g3} {e4,e13,ep4}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e11 {g0,g34,g2} {e1,e02,ep1}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e12 {g0,g34,g2} {e2,e01,ep2}  Certificate for Clifford criterion: False


__Perseus cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b1, b2};
EE = {e3, e4, e01, e02, e11, e12, ep4, ep3};
GT = {g0, g1, g2, g3};
comp[BB, EE, GT]
```

Output:

    {e3,e4,ep4,ep3} e3 {g2,g3} {}  Certificate for Clifford criterion: True
    {e3,e4,ep4,ep3} e4 {g2,g3} {}  Certificate for Clifford criterion: True
    {e3,e4,ep4,ep3} ep4 {g0,g1} {}  Certificate for Clifford criterion: True
    {e3,e4,ep4,ep3} ep3 {g0,g1} {}  Certificate for Clifford criterion: True
    {e01,e02,e11,e12} e01 {g1,g3} {}  Certificate for Clifford criterion: True
    {e01,e02,e11,e12} e02 {g1,g3} {}  Certificate for Clifford criterion: True
    {e01,e02,e11,e12} e11 {g0,g2} {}  Certificate for Clifford criterion: True
    {e01,e02,e11,e12} e12 {g0,g2} {}  Certificate for Clifford criterion: True


__ring cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b13, b24, bp14, bp23};
EE = {e1, e2, e3, e4};
GT = {g12, g34};
{g0.M.bp14 != 0, g1.M.b13 !=0} (* g0 and g1 are not tracing elements of G(X)*)
comp[BB, EE, GT]
```

Output:

    {e1,e2,e3,e4} e1 {g12} {}  Certificate for Clifford criterion: True
    {e1,e2,e3,e4} e2 {g12} {}  Certificate for Clifford criterion: True
    {e1,e2,e3,e4} e3 {g34} {}  Certificate for Clifford criterion: True
    {e1,e2,e3,e4} e4 {g34} {}  Certificate for Clifford criterion: True

__EH1 cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b12};
EE = {e1, e2, e3, e4, e03, e04, e11, e12, e13, e14, ep1, ep2};
GT = {g0, g1, g34, g3};
comp[BB, EE, GT]
```

Output:

    {e3,e4,ep1,ep2}   e3  {g34,g3} {e03,e13} Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2}   e4  {g34,g3} {e04,e14} Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2}   ep1 {g0,g1}  {e2,e11}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2}   ep2 {g0,g1}  {e1,e12}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e03 {g1,g3}  {e3,e14}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e04 {g1,g3}  {e4,e13}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e11 {g0,g34} {e1,ep1}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e12 {g0,g34} {e2,ep2}  Certificate for Clifford criterion: False

__CH1 cyclide__


```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b13, b24, bp12};
EE = {e1, e2, e3, e4, e13, e14};
GT = {g0, g1, g34};
comp[BB, EE, GT]
```

Output:

    {e3,e4,e13,e14} e3 {g34} {}  Certificate for Clifford criterion: True
    {e3,e4,e13,e14} e4 {g34} {}  Certificate for Clifford criterion: True
    {e3,e4,e13,e14} e13 {g0} {}  Certificate for Clifford criterion: True
    {e3,e4,e13,e14} e14 {g0} {}  Certificate for Clifford criterion: True

__HP cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]};  (* real structure of type 2A1 *)
BB = {b12, bp34};
EE = {e1, e2, e3, e4, e03, e04, e11, e12};
GT = {g0, g1};
comp[BB, EE, GT]
```

Output:

    {e03,e04,e11,e12} e03 {g1} {e3}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e04 {g1} {e4}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e11 {g0} {e1}  Certificate for Clifford criterion: False
    {e03,e04,e11,e12} e12 {g0} {e2}  Certificate for Clifford criterion: False

__EY cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b12, b1, b2};
EE = {e3, e4, e11, e12};
GT = {g0, g1, g3};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets.

__CY cyclide__


```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b12, b1, b2, bp13, bp24};
EE = {e3, e4, e11, e12};
GT = {g0, g1, g3};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets.

__EO cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b12, b34};
EE = {e1, e2, e3, e4, e11, e12, e13, e14};
GT = {g0, g1, g3};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets.

__CO cyclide__

```Mathematica
inv[q_] := {q[[1]], q[[2]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 2A1 *)
BB = {b12, b34, bp13, bp24};
EE = {e1, e2, e3, e4};
GT = {g0, g1};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets.

__EE/EH2 cyclide__

```Mathematica
inv[q_] := {q[[2]], q[[1]], q[[4]], q[[3]], q[[6]], q[[5]]};  (* real structure of type 3A1 *)
BB = {b0};
EE = {e1, e2, e3, e4, e01, e12, e02, e11, e03, e14, e04, e13};
GT = {g12, g34};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets.

__EP cyclide__

```Mathematica
inv[q_] := {q[[2]], q[[1]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 3A1 *)
BB = {b13, bp24};
EE = {e1, e2, e3, e4, e02, e11, e04, e13};
GT = {g12, g34};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets.

__S1 cyclide__

```Mathematica
inv[q_] := {q[[2]], q[[1]], q[[4]], q[[3]], q[[6]], q[[5]]}; (* real structure of type 3A1 *)
BB = {};
EE = {e1, e2, e3, e4, e01, e02, e03, e04, e11, e12, e13, e14, ep1, ep2, ep3, ep4};
GT = {g12, g34};
comp[BB, EE, GT]
```

Output:

    {e1,e2,ep3,ep4} e1  {g12} {e01,e11,ep2} Certificate for Clifford criterion: False
    {e1,e2,ep3,ep4} e2  {g12} {e02,e12,ep1} Certificate for Clifford criterion: False
    {e1,e2,ep3,ep4} ep3 {g34} {e4,e03,e13}  Certificate for Clifford criterion: False
    {e1,e2,ep3,ep4} ep4 {g34} {e3,e04,e14}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} e3  {g34} {e03,e13,ep4} Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} e4  {g34} {e04,e14,ep3} Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} ep1 {g12} {e2,e01,e11}  Certificate for Clifford criterion: False
    {e3,e4,ep1,ep2} ep2 {g12} {e1,e02,e12}  Certificate for Clifford criterion: False

__S2 cyclide__


```Mathematica
D4 = {{1, 0, 0, 0, 0, 0}, {2, 1, 1, 1, 1, 1}, {-1, 0, -1, 0, 0,0}, {-1, 0, 0, -1, 0, 0}, {-1, 0, 0, 0, -1, 0}, {-1, 0, 0, 0, 0, -1}};
inv[q_] := D4.q; (* real structure of type D4 *)
BB = {};
EE = {e1, e2, e3, e4, e01, e02, e03, e04, e11, e12, e13, e14, ep1, ep2, ep3, ep4};
GT = {g1, g2};
comp[BB, EE, GT]
```

Output:

    There exist no Clifford quartets

Notice that when the real structure is of type D4, then complex conjugate classes in EE intersect via M.

```Mathematica
{str /@ EE, str /@ inv /@ EE} // TableForm
```

Output:

    e1   e2   e3   e4   e01  e02  e03  e04  e11  e12  e13  e14  ep1  ep2  ep3  ep4
    e11  e12  e13  e14  ep1  ep2  ep3  ep4  e1   e2   e3   e4   e01  e02  e03  e04
