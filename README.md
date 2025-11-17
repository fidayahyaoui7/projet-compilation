# Projet COMPILATION

##  Table des Matières

1. [Description Générale](#description-générale)
2. [Architecture du Projet](#architecture-du-projet)
3. [Partie I - Spécifications Lexicales](#partie-i---spécifications-lexicales)
4. [Partie II - Spécifications Syntaxiques](#partie-ii---spécifications-syntaxiques)
5. [Installation et Compilation](#installation-et-compilation)
6. [Tests et Validation](#tests-et-validation)
7. [Notes Techniques Importantes](#notes-techniques-importantes)

---

##  Description Générale

Ce projet implémente un **compilateur simplifié** comprenant :

- **Analyse Lexicale (AL)** : Automate à états finis (DFA) pour la tokenisation
- **Analyse Syntaxique (AS)** : Parser récursif descendant basé sur la grammaire G'
- **Table des Symboles** : Gestion des identificateurs avec vérification des déclarations

### Objectifs du Projet

 **Partie I** : Définir unités lexicales, grammaire G' sans récursion gauche ni ambiguïté  
**Partie II** : Implémenter DFA + Analyseur Lexical + Analyseur Syntaxique + Table des identificateurs  

---

##  Architecture du Projet
Projet
│
├── include/
│   └── tokens.h        
│
├── src/
│   ├── lexer.h           
│   ├── lexer.c          
│   ├── parser.h       
│   ├── parser.c        
│   ├── symtab.h          
│   ├── symtab.c          
│   └── main.c          
│
├── tests/
│   ├── test1.txt        
│   └── test2.txt       
│
├── Makefile         
└── README.md      
---

## Partie I - Spécifications Lexicales

### 1.1 Unités Lexicales (Tokens)

| **Catégorie** | **Tokens** | **Code** | **Exemple** |
|---------------|------------|----------|-------------|
| **Mots-clés** | program, var, Begin, End, Endif, then, int, readln, writeln, if | 1-10 | `program`, `Begin` |
| **Identificateurs** | Lettres suivies de lettres/chiffres | 11 | `x`, `var1`, `total` |
| **Nombres** | Suite de chiffres | 12 | `123`, `0`, `9999` |
| **Opérateurs relationnels** | `=`, `==`, `<>`, `<`, `<=`, `>`, `>=` | 20-25 | `a > 5` |
| **Opérateurs arithmétiques** | `+`, `*` | 30-31 | `x + 3`, `y * 2` |
| **Séparateurs** | `:=`, `,`, `:`, `;`, `(`, `)`, `.` | 40-46 | `x := 10;` |
| **Commentaires** | `(* ... *)` | Ignoré | `(* ceci est un commentaire *)` |
| **Fin de fichier** | EOF | 999 | - |

### 1.2 Automate à États Finis (DFA)
┌─────────────────────────────────────────────────────────────┐
│                    AUTOMATE LEXICAL (DFA)                   │
└─────────────────────────────────────────────────────────────┘
État Initial (S0)
│
├── [a-zA-Z] ──→ S_ID ──→ [a-zA-Z0-9]* ──→ [ID ou MOT-CLÉ]
│
├── [0-9] ──────→ S_NB ──→ [0-9]* ──────→ [NOMBRE]
│
├── '+' ────────────────────────────────→ [TOK_PLUS]
├── '' ────────────────────────────────→ [TOK_MULT]
├── ';' ────────────────────────────────→ [TOK_SEMI]
├── ',' ────────────────────────────────→ [TOK_COMMA]
├── '.' ────────────────────────────────→ [TOK_DOT]
├── '(' ──→ si '' ──→ COMMENTAIRE ──→ jusqu'à '*)'
│      └─→ sinon ──→ [TOK_LPAR]
├── ')' ────────────────────────────────→ [TOK_RPAR]
│
├── ':' ──→ si '=' ──→ [TOK_AFFECT :=]
│      └─→ sinon ──→ [TOK_COLON :]
│
├── '=' ──→ si '=' ──→ [TOK_OPREL_EQ ==]
│      └─→ sinon ──→ [TOK_OPREL_EQ =]
│
├── '<' ──→ si '=' ──→ [TOK_OPREL_LE <=]
│      ├─→ si '>' ──→ [TOK_OPREL_NE <>]
│      └─→ sinon ──→ [TOK_OPREL_LT <]
│
└── '>' ──→ si '=' ──→ [TOK_OPREL_GE >=]
└─→ sinon ──→ [TOK_OPREL_GT >]
États Finaux : ID, NB, tous les tokens d'opérateurs
### 1.3 Gestion des Commentaires

**Format** : `(* commentaire sur une ou plusieurs lignes *)`

**Algorithme** :     
SI caractère = '(' ALORS
lire caractère suivant
SI caractère = '' ALORS
prev ← caractère courant
TANT QUE NON (prev = '' ET caractère = ')') ET caractère ≠ EOF FAIRE
prev ← caractère
lire caractère suivant
FIN TANT QUE
continuer analyse
SINON
retourner TOK_LPAR
FIN SI
FIN SI
  ### 1.4 Reconnaissance des Mots-Clés

**Fonction** : `keyword_or_id(lexeme)`
```c
Entrée : chaîne de caractères
Sortie : code token

SI lexeme = "program" RETOURNER TOK_PROGRAM
SI lexeme = "var"     RETOURNER TOK_VAR
SI lexeme = "Begin"   RETOURNER TOK_BEGIN
SI lexeme = "End"     RETOURNER TOK_END
SI lexeme = "Endif"   RETOURNER TOK_ENDIF
SI lexeme = "then"    RETOURNER TOK_THEN
SI lexeme = "int"     RETOURNER TOK_INT
SI lexeme = "readln"  RETOURNER TOK_READLN
SI lexeme = "writeln" RETOURNER TOK_WRITELN
SI lexeme = "if"      RETOURNER TOK_IF
SINON RETOURNER TOK_ID
```

 **Important** : Les mots-clés sont **sensibles à la casse**  
-  Correct : `Begin`, `End`, `Endif`  
-  Incorrect : `begin`, `end`, `endif`

---

##  Partie II - Spécifications Syntaxiques

### 2.1 Grammaire G' (sans récursion gauche, non ambiguë)
P          → program id ; Dcl Begin Liste_inst End.
Dcl        → var Liste_id : int ;
Liste_id   → id (, id)*
Liste_inst → (I)*
I          → id := Exp ;
| writeln(id);
| readln(id);
| if C then Liste_inst Endif
C          → Exp Oprel Exp
Exp        → Term (+ Term)*
Term       → Factor (* Factor)*
Factor     → id
| nb
| (Exp)
Oprel      → = | == | <> | < | <= | > | >=

### 2.2 Analyse Récursive Descendante

**Principe** : Chaque non-terminal de la grammaire → une fonction récursive
```c
Fonction parse_program()
    match(TOK_PROGRAM)
    match(TOK_ID)
    match(TOK_SEMI)
    parse_decls()
    match(TOK_BEGIN)
    parse_insts()
    match(TOK_END)
    match(TOK_DOT)

Fonction parse_decls()
    match(TOK_VAR)
    parse_list_id()
    match(TOK_COLON)
    match(TOK_INT)
    match(TOK_SEMI)

Fonction parse_list_id()
    match(TOK_ID)  [+ insertion dans table symboles]
    TANT QUE token = TOK_COMMA FAIRE
        match(TOK_COMMA)
        match(TOK_ID)  [+ insertion dans table symboles]
    FIN TANT QUE

Fonction parse_insts()
    TANT QUE token ∈ {TOK_ID, TOK_WRITELN, TOK_READLN, TOK_IF} FAIRE
        parse_inst()
    FIN TANT QUE

Fonction parse_inst()
    CAS token DE
        TOK_ID: 
            match(TOK_ID)
            match(TOK_AFFECT)
            parse_exp()
            match(TOK_SEMI)
        
        TOK_WRITELN:
            match(TOK_WRITELN)
            match(TOK_LPAR)
            match(TOK_ID)
            match(TOK_RPAR)
            match(TOK_SEMI)
        
        TOK_READLN:
            match(TOK_READLN)
            match(TOK_LPAR)
            match(TOK_ID)
            match(TOK_RPAR)
            match(TOK_SEMI)
        
        TOK_IF:
            match(TOK_IF)
            parse_cond()
            match(TOK_THEN)
            parse_insts()
            match(TOK_ENDIF)
    FIN CAS

Fonction parse_cond()
    parse_exp()
    match(Opérateur_Relationnel)
    parse_exp()

Fonction parse_exp()
    parse_term()
    TANT QUE token = TOK_PLUS FAIRE
        match(TOK_PLUS)
        parse_term()
    FIN TANT QUE

Fonction parse_term()
    parse_factor()
    TANT QUE token = TOK_MULT FAIRE
        match(TOK_MULT)
        parse_factor()
    FIN TANT QUE

Fonction parse_factor()
    CAS token DE
        TOK_ID: match(TOK_ID)
        TOK_NB: match(TOK_NB)
        TOK_LPAR:
            match(TOK_LPAR)
            parse_exp()
            match(TOK_RPAR)
        SINON: erreur("factor attendu")
    FIN CAS
```

### 2.3 Table des Symboles

**Structure** :
```c
┌──────────┬────────────────┐
│  Index   │  Identificateur│
├──────────┼────────────────┤
│    0     │   p1           │  ← Nom du programme
│    1     │   x            │  ← Variables
│    2     │   y            │
│    3     │   total        │
└──────────┴────────────────┘
```

**Opérations** :

1. **Insertion** : `symtab_insert(id)`
   - Vérifie si `id` existe déjà
   - Si non, ajoute `id` et retourne son index
   - Si oui, retourne l'index existant

2. **Recherche** : `symtab_find(id)`
   - Retourne l'index si trouvé
   - Retourne -1 si non trouvé

3. **Affichage** : `symtab_print()`
   - Affiche tous les identificateurs avec leur index

**Moment d'insertion** :
- Nom du programme : après `program`
- Variables : lors de la déclaration dans `var Liste_id : int ;`

**Vérification** :
- Lors de l'utilisation d'un identificateur (affectation, lecture, écriture)
- Si `symtab_find(id) < 0` → Warning "variable non déclarée"

---

##  Installation et Compilation

### Étape 1 : Créer la Structure de Répertoires
```bash
mkdir -p Projet_Analyseur_PartieI_II/{include,src,tests}
cd Projet_Analyseur_PartieI_II
```

### Étape 2 : Copier les Fichiers

Copier tous les fichiers dans leurs répertoires respectifs selon la structure indiquée.

### Étape 3 : Compiler
```bash
make
```

**Sortie attendue** :
gcc -Iinclude -Wall -Wextra -std=c99 -c src/lexer.c -o src/lexer.o
gcc -Iinclude -Wall -Wextra -std=c99 -c src/parser.c -o src/parser.o
gcc -Iinclude -Wall -Wextra -std=c99 -c src/symtab.c -o src/symtab.o
gcc -Iinclude -Wall -Wextra -std=c99 -c src/main.c -o src/main.o
gcc -Iinclude -Wall -Wextra -std=c99 -o parser src/lexer.o src/parser.o src/symtab.o src/main.o

### Étape 4 : Nettoyer (optionnel)
```bash
make clean
```

---

##  Tests et Validation

### Test 1 : Programme Simple

**Fichier** : `tests/test1.txt`
```pascal
program p1;
var x, y : int;
Begin
  x := 2;
  y := x + 3 * (x + 1);
  writeln(x);
End.
```

**Exécution** :
```bash
./parser tests/test1.txt
```

**Sortie attendue** :
Analyse syntaxique OK.
Table des symboles (3 entrées):
0: p1
1: x
2: y

### Test 2 : Structure Conditionnelle

**Fichier** : `tests/test2.txt`
```pascal
program p2;
var a, b : int;
Begin
  a := 10;
  if a > 5 then
    b := a + 1;
    writeln(b);
  Endif
End.
```

**Exécution** :
```bash
./parser tests/test2.txt
```

**Sortie attendue** :
Analyse syntaxique OK.
Table des symboles (3 entrées):
0: p2
1: a
2: b

### Test 3 : Détection d'Erreurs

**Test avec variable non déclarée** :
```pascal
program test_error;
var x : int;
Begin
  y := 5;  (* y non déclaré *)
  writeln(y);
End.
```

**Sortie attendue** :
Warning: use of undeclared id 'y' (line 4)
Warning: writeln uses undeclared id 'y' (line 5)
Analyse syntaxique OK.
Table des symboles (2 entrées):
0: test_error
1: x

### Test 4 : Commentaires

**Programme avec commentaires** :
```pascal
program test_comments;
(* Déclaration des variables *)
var x, y : int;
Begin
  x := 10;  (* Initialisation de x *)
  (* Calcul complexe *)
  y := x + (* opération *) 5;
  writeln(y);
End.
```

**Résultat** : Les commentaires sont correctement ignorés.

---

## Notes Techniques Importantes

###  Limitations du Projet

1. **Pas de génération de code** : Ce projet s'arrête après l'analyse syntaxique
2. **Un seul type** : Uniquement `int` est supporté
3. **Opérateurs limités** : Seulement `+` et `*` (pas de `-`, `/`, `%`)
4. **Pas de else** : Structure `if-then-Endif` uniquement
5. **Pas de boucles** : while/for non implémentés

###  Respect des Spécifications

| **Critère** | **Statut** | **Détails** |
|-------------|------------|-------------|
| DFA complet | ✅ | Automate couvre tous les tokens |
| Grammaire G' | ✅ | Sans récursion gauche ni ambiguïté |
| Parser récursif | ✅ | Une fonction par non-terminal |
| Table symboles | ✅ | Insertion + recherche + affichage |
| Gestion commentaires | ✅ | Format `(* ... *)` |
| Sensibilité casse | ✅ | Mots-clés : Begin/End/Endif |

###  Points de Vigilance

1. **Mots-clés** : Respecter exactement la casse
   -  `Begin` `End` `Endif`
   - `begin` `end` `endif`

2. **Opérateurs** :
   - `:=` pour affectation (pas `=`)
   - `==` ou `=` pour égalité
   - `<>` pour différence

3. **Syntaxe** :
   - Toujours terminer par `.` après `End`
   - Point-virgule `;` après chaque instruction
   - Parenthèses obligatoires pour `readln` et `writeln`

4. **Déclarations** :
   - Toutes les variables doivent être déclarées dans `var`
   - Une seule section `var` au début du programme

###  Débogage

**Erreur** : "Syntax error at line X"
- Vérifier la syntaxe du fichier source
- Vérifier la casse des mots-clés
- Vérifier les symboles (`;`, `:=`, etc.)

**Warning** : "use of undeclared id"
- Variable utilisée avant déclaration
- Variable non déclarée dans la section `var`

**Erreur compilation** : "undefined reference"
- Vérifier que tous les fichiers .c sont compilés
- Nettoyer et recompiler : `make clean && make`

---

##  Références Théoriques

### Automates Finis

- **DFA** : Deterministic Finite Automaton
- **États** : Ensemble fini d'états avec un état initial et des états finaux
- **Transitions** : Fonction de transition δ(état, symbole) → état

### Analyse Récursive Descendante

- **Principe** : Exploration top-down de l'arbre syntaxique
- **Conditions** : Grammaire LL(1) sans récursion gauche
- **Avantages** : Simple à implémenter, facile à déboguer

### Grammaires LL(1)

- **LL** : Left-to-right, Leftmost derivation
- **1** : Un seul symbole de lookahead
- **Propriétés** : Non ambiguë, sans récursion gauche

---

##  Auteur et Contact

**Projet Académique** - Compilateur Simplifié  
**Objectif** : Implémentation d'analyseur lexical et syntaxique  
**Niveau** : Partie I & II 

Pour toute question ou amélioration, consulter la documentation du cours de compilation.

Instructions de Déploiement

Créer la structure de répertoires :

bash   mkdir -p Projet_Analyseur_PartieI_II/{include,src,tests}
   cd Projet_Analyseur_PartieI_II

Copier tous les fichiers dans leurs répertoires respectifs
Compiler :

bash   make

Tester :

bash   ./parser tests/test1.txt
   ./parser tests/test2.txt

 Résultat Attendu
Analyse syntaxique OK.

Table des symboles (4 entrées):
0: p1
1: x
2: y
3: a
